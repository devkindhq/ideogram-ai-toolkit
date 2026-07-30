# Photographic Icon Set Generator Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship `skills/photographic-icon-set-generator/` so Claude Code can generate a
*consistent set* of photographic/textured/3D-material icons (3D-rendered, claymation,
hand-painted, isometric-with-material-texture) via the connected Ideogram MCP, holding a
locked style block identical across every icon in the batch, reviewing the finished set
for cross-icon drift, and correcting/extending it later — with the same structural rigor
(`SKILL.md` + `references/` + a real worked example + `evals/`) as the sibling skills in
this repo (`collections-management`, `character-model-sheet`, `moodboard-generator`).

**Architecture:** One unified, hybrid skill combining prompt-composition discipline (like
`logo-prompting`/`character-model-sheet`) with a sequential multi-call orchestration
workflow (like `collections-management`): draft a style block once, draft N per-icon
prompts that repeat it byte-identical, run an icon-specific anti-slop gate, call
`generate_images_bulk` once for the whole batch, review the result *as a set* (not
icon-by-icon), and correct outliers or extend the set later via `edit_image`/`remix_image`
rather than a fresh batch call. A new "set style spec" JSON artifact (not a reuse of
`composition-spec-format.md`'s bbox schema — this skill's output is N separate
single-subject images, not one multi-panel sheet to annotate) records the locked style
block, per-icon prompts, and resulting identifiers so a later session can extend the set
without re-deriving the recipe.

**Tech Stack:** Markdown skill files (frontmatter + prose), live Ideogram MCP tool calls
(`mcp__ideogram__generate_images_bulk`, `mcp__ideogram__remix_image`,
`mcp__ideogram__edit_image`, `mcp__ideogram__get_generation_status`), JSON eval schema
matching `collections-management/evals/evals.json`'s shape (`id`/`eval_name`/`prompt`/
`expected_output`/`files`, no `assertions` field at this pass).

## Global Constraints

- Trigger only on requests for a **set** of photographic/textured/3D/material-rendered
  icons (3D-rendered, claymation, hand-painted, isometric-with-material-texture, or a
  described custom raster style) — never on a single one-off icon request (that's
  `generate_image` via `ideogram-prompt`) and never on flat/line/glyph/generic
  "vector-style" icon-set requests (state the scoping boundary plainly and stop; don't
  generate a raster substitute).
- On Ideogram's default v4 pipeline (no `custom_model_uri` set — confirmed directly from
  `generate_images_bulk`'s own tool-schema description), `style_type`, `negative_prompt`,
  `magic_prompt_option`, and `seed` are all ignored. Leave them unset by default; only use
  them if the escalation path (`custom_model_uri` via `custom-model-training`) is
  explicitly in play.
- Cross-icon consistency must be carried entirely by a repeated, held-byte-identical
  "style block" of prompt text across every entry in the `prompts[]` array — never by
  generation parameters alone.
- Default `aspect_ratio` is `"1x1"` unless the user specifies otherwise; default
  `rendering_speed` is `"QUALITY"`, matching sibling-skill precedent for detail-dense,
  consistency-sensitive renders.
- Confirmed live tool surface only: `generate_images_bulk`, `remix_image`, `edit_image`,
  `get_generation_status` (plus `upload_image` if local reference files are needed). Do
  not invent parameters not present on the live MCP schema (verified directly via
  tool-schema inspection during planning: `generate_images_bulk` takes `prompts`,
  `aspect_ratio`, `resolution`, `rendering_speed`, `style_type`, `negative_prompt`,
  `magic_prompt_option`, `seed`, `custom_model_uri`, `collection_id`, `private`;
  `edit_image` takes `prompt`, `image_response_ids`, `image_upload_ids`, `aspect_ratio`,
  `magic_prompt`, `seed`, `collection_id`, `private`; `remix_image` takes `prompt`,
  `image_response_id`, `image_upload_id`, `image_weight`, `num_images`, `seed`,
  `collection_id`, `private`).
- Because `negative_prompt` is dead weight on the default pipeline, every ban in
  `references/icon-anti-slop-discipline.md` must be phrased as a positive instruction to
  embed in the prompt text, never as a negative-prompt string.
- New artifact type: a "set style spec" JSON (locked style block verbatim, per-icon
  subject list, resulting `response_id`/image URL per icon). Do not reuse or extend
  `composition-spec-format.md`'s bbox/`elements[]` schema for this skill — that schema is
  for multi-panel sheets, this skill's deliverable is N separate single-subject images.
- `custom-model-training` escalation path (documented, not auto-invoked): only raise it in
  `references/set-consistency-workflow.md` and in step 8 of the workflow; never trigger it
  automatically. (Note: `custom-model-training` does not exist as a built skill in this
  worktree yet — only its design spec does, at
  `docs/superpowers/specs/2026-07-30-custom-model-training-design.md` — so any reference to
  it must be phrased as "if/when that skill exists" rather than assuming a live
  cross-reference.)
- `remove_background` is out of scope for v1 — icons render on a deliberately styled,
  consistent plain/staged ground, not a transparent background.
- No artificial cap imposed below `generate_images_bulk`'s own 1-500 `prompts[]` limit,
  but sanity-check unusually large batch requests with the user first (Pro-subscription
  and cost implications) rather than submitting a large job silently.
- Every real identifier produced while building the worked example (`response_id`s, image
  URLs, any `request_id` from `get_generation_status`) must be written to
  `examples/<name>/RESULT.md` — per "No Context Lost" / "Save First, Synthesize Later,"
  nothing generated while building the example may live only in this plan or a chat
  transcript.
- Do not modify any file under `skills/` other than the five new files/directories this
  plan creates under `skills/photographic-icon-set-generator/`.
- No placeholder content anywhere in the shipped skill — every workflow step,
  error-handling rule, style-block example, anti-slop entry, and eval prompt must have
  real, specific text, matching the sibling skills' style exactly.

---

## File structure

| File | Responsibility |
|---|---|
| `skills/photographic-icon-set-generator/SKILL.md` | Frontmatter trigger description (including the explicit flat/vector non-trigger) + the 9-step workflow body (gather → lock style block → draft prompts → anti-slop gate → generate batch → review as a set → correct outliers → extend later → save), pointing to the three reference files. |
| `skills/photographic-icon-set-generator/references/style-lock-recipe.md` | The style-block discipline: the fixed axes to name once and repeat identically (render/material technique, lighting, camera angle/perspective, background/ground treatment, color story), plus one worked example style block for each of the four named styles (3D-rendered, claymation, hand-painted, isometric-with-material-texture). |
| `skills/photographic-icon-set-generator/references/icon-anti-slop-discipline.md` | Icon-specific failure modes (glossy-app-icon-store clichés, drop shadows/gradient bubbles/bevel-emboss defaults, drift into "mini scene" territory, camera-angle/material drift between icons in the same batch) phrased as positive prompt instructions (since `negative_prompt` is dead weight), plus the pre-generation gate table. |
| `skills/photographic-icon-set-generator/references/set-consistency-workflow.md` | The batch-generate → review-as-a-set → correct-outliers → extend-later workflow in full, the `remix_image` vs. `edit_image` decision, the `custom-model-training` escalation path, and the honest unverified-facts callout on shared `seed` across varying prompts. |
| `skills/photographic-icon-set-generator/examples/<worked-example-name>/` | One real worked run: draft a style block + a small icon set's prompts, run the anti-slop gate, call `generate_images_bulk` for real, review the result as a set, run at least one real correction or extension call (`edit_image` or `remix_image`), and a `RESULT.md` documenting real identifiers and outcomes. |
| `skills/photographic-icon-set-generator/evals/evals.json` | 3 eval prompts (`id`, `eval_name`, `prompt`, `expected_output`, `files`) matching `collections-management/evals/evals.json`'s schema, covering fresh-set-generation, extend-existing-set, and flat/vector-out-of-scope. |

---

### Task 1: Write SKILL.md

**Files:**
- Create: `skills/photographic-icon-set-generator/SKILL.md`

**Interfaces:**
- Consumes: nothing (first file written)
- Produces: `skills/photographic-icon-set-generator/SKILL.md` — the entry point every
  later task's cross-references point back to (the three `references/*.md` links, the
  9-step workflow numbering Task 5's worked example follows, the tool names Task 6's
  evals exercise)

- [ ] **Step 1: Write frontmatter.** `name: photographic-icon-set-generator`.
  `description:` must state, in the description text itself (not buried in the body, so
  the routing decision is visible at the trigger-matching layer): (a) what the skill does
  — generates a *consistent set* of photographic/textured/3D-material-rendered icons
  (3D-rendered, claymation, hand-painted, isometric-with-material-texture) via Ideogram,
  holding a locked style recipe across every icon in the batch; (b) explicit trigger
  phrases from the spec's Components section — "3D icon set," "claymation icons,"
  "textured/material icon pack," "isometric icon set with material texture,"
  "photographic icon set," "match this icon style," "add an icon to my [style] set"; (c)
  the explicit non-trigger — flat, line, glyph, minimal, or generic "vector-style" icon
  set requests are out of scope for this skill (different mechanism: deterministic SVG
  code, not diffusion output) and it should say so rather than generating a raster
  substitute; (d) the set-vs-single distinction — a one-off "generate me a 3D camera icon"
  request is `generate_image` via `ideogram-prompt`, not this skill; this skill exists
  specifically for the multi-icon consistency problem.
- [ ] **Step 2: Write the intro paragraph(s).** State the tool surface this skill
  orchestrates (`generate_images_bulk`, `remix_image`, `edit_image`,
  `get_generation_status`), that it's a hybrid skill — prompt-composition discipline
  (like `logo-prompting`/`character-model-sheet`) *plus* a multi-call orchestration
  workflow (like `collections-management`) — and the single load-bearing technical fact
  the rest of the skill hangs off: on the default v4 pipeline (no `custom_model_uri`),
  `style_type`, `negative_prompt`, `magic_prompt_option`, and `seed` are all ignored per
  `generate_images_bulk`'s own tool-schema description, so consistency has to be carried
  entirely by a repeated, held-byte-identical style block of prompt text across every
  `prompts[]` entry — not by generation parameters.
- [ ] **Step 3: Write a "Before you start" pointer to the three reference files**,
  mirroring `collections-management/SKILL.md`'s "Before you start" section — direct the
  reader to `references/style-lock-recipe.md` (the style-block discipline, read before
  drafting any prompt), `references/icon-anti-slop-discipline.md` (the pre-generation
  gate, read before calling `generate_images_bulk`), and
  `references/set-consistency-workflow.md` (the full batch → review → correct → extend
  loop, read before step 5 below).
- [ ] **Step 4: Write "## Workflow" with 9 numbered subsections**, each with real,
  specific content (no placeholders), matching the spec's Data flow section:
  - `### 1. Gather the icon list and style direction` — ask for the list of icon
    concepts/subjects, the target count, and one of the four named styles (or a described
    custom raster style); don't invent a default look silently, since the whole value
    proposition is a *deliberately locked* shared recipe. If the request is actually for
    flat/line/glyph/vector icons, say so now and stop (see Error handling) rather than
    proceeding with a raster substitute.
  - `### 2. Lock the style block` — draft it once following
    `references/style-lock-recipe.md`'s checklist (material/render technique, lighting,
    camera angle, background/ground, color story) as one continuous description written
    to be repeated byte-identical across every icon prompt, not re-described per icon.
  - `### 3. Draft one prompt per icon` — each entry in the eventual `prompts[]` array is
    `[style block, held identical] + [subject block, varies per icon] + [composition
    block: isolated single subject, centered, consistent margin, consistent ground/shadow
    treatment]`, in the icon list's order.
  - `### 4. Run the anti-slop gate` — score the drafted prompts against
    `references/icon-anti-slop-discipline.md`'s pre-generation table before generating;
    regenerating a full batch is more expensive than revising N prompt strings first.
  - `### 5. Generate the batch` — call `generate_images_bulk(prompts=[...])` with a
    shared `aspect_ratio` (default `"1x1"` unless the user wants something else),
    `rendering_speed: "QUALITY"`, and `resolution` only if the user needs exact pixel
    dimensions. Leave `style_type`, `negative_prompt`, `magic_prompt_option`, and `seed`
    unset by default — they're ignored on the default v4 path — unless `custom_model_uri`
    is set via the escalation path in step 8, in which case they become real levers.
    State plainly this is a background job (per the tool's own description); tell the
    user images render as they complete, and offer to poll `get_generation_status` if the
    client is text-only rather than auto-rendering a carousel.
  - `### 6. Review the finished set as a set, not icon-by-icon` — look at all N results
    together and check material, lighting, and camera-angle consistency across the whole
    batch; the entire reason this skill exists is catching drift a per-icon review would
    miss.
  - `### 7. Correct any outlier` — prefer `edit_image` with `image_response_ids` set to
    2-3 "anchor" icons from the set that show the target style well, plus a corrective
    prompt describing the fix; multi-reference conditioning pulls the result toward the
    group in a way a single reference can't as reliably. Use `remix_image` against one
    anchor image (`image_weight` toward the higher end for "match this closely") as the
    simpler fallback when only one strong anchor exists. Don't regenerate the whole batch
    for a partial-drift problem — a fresh `generate_images_bulk` call has no better odds
    of consistency without also fixing the underlying style-block text, and it discards
    the icons that already matched.
  - `### 8. Extend the set later` — same multi-reference `edit_image` pattern (2-3
    existing icons as `image_response_ids` + a prompt for the new subject) rather than a
    fresh `generate_images_bulk` call, since a new batch call has no memory of the earlier
    batch's exact style-block wording. If the project's icon count and consistency needs
    have grown past what prompt-text-only style-locking is holding up (10+ icons, or icons
    generated across multiple sessions), this is the point to raise the
    `custom-model-training` escalation path documented in
    `references/set-consistency-workflow.md` — phrased as an option to consider, not
    something this skill triggers on its own, and noting that `custom-model-training`
    doesn't exist as a built skill in this repo yet if that's still true at the time.
  - `### 9. Save what was made` — write the locked style block, the per-icon prompts, the
    resulting `response_id`/image URL per icon, and any correction/extension notes to a
    "set style spec" JSON plus a project markdown file (check for an existing
    `logo-explorations/`, `branding/`, or icon-specific folder first and match it, same
    convention as the sibling skills). Follow "No Context Lost" — nothing generated here
    should live only in the conversation.
- [ ] **Step 5: Write "## The 'set style spec' artifact"** as its own section explaining
  the new JSON shape this skill introduces (deliberately not a reuse of
  `composition-spec-format.md`'s bbox schema): top-level `style_block` (the exact,
  byte-identical text repeated across every prompt), `icons[]` array of
  `{subject, prompt, response_id, image_url}` entries in the icon list's order, and a
  `corrections[]`/`extensions[]` log recording any post-generation `edit_image`/
  `remix_image` calls with their `prompt`, the `image_response_ids`/`image_response_id`
  used as anchors, and the resulting `response_id`. State why this shape exists: it's
  what lets a later session add another icon to an existing set without re-deriving the
  recipe from scratch.
- [ ] **Step 6: Write "## Error handling"** as its own section (not buried inline),
  covering all six cases from the spec verbatim in spirit: flat/vector/line/glyph request
  → state plainly out of scope, don't generate a raster substitute; undecided style
  direction → ask for one of the four named styles or a described custom style, don't
  invent a default silently; `generate_images_bulk` requires a Pro subscription — if it
  errors on subscription tier, surface the real error and ask before doing anything else,
  don't silently fall back to looping individual `generate_image` calls; partial batch
  failure (some `prompts[]` entries fail while others succeed) → report exactly which
  icons failed and why, don't claim the whole set completed; style drift detected across
  the set → run the targeted correction loop (workflow step 7) first, don't regenerate
  the whole batch blindly; correcting an outlier the user actually preferred as-is →
  confirm which icon is the intended "anchor" for the set's look before conforming others
  to it, don't assume the majority of the batch is automatically the correct target.
- [ ] **Step 7: Write "## Save what you made"**, mirroring
  `collections-management/SKILL.md`'s section of the same name — after generating,
  correcting, or extending a set, save the "set style spec" JSON and a project markdown
  file to the project's existing output location (check for an existing
  `logo-explorations/`, `branding/`, or icon-specific folder first and match it, same
  convention as `brand-identity-sheet`/`character-model-sheet`/`moodboard-generator`).
- [ ] **Step 8: Write "## Reference files"** closing section pointing to all three
  `references/*.md` files with a one-line description of what each covers, matching the
  bullet-list format at the bottom of `collections-management/SKILL.md`.
- [ ] **Step 9: Self-review pass.** Re-read the finished file against the design spec's
  Data flow/workflow (9 steps), Error handling (6 rules), and Components sections —
  confirm every numbered item has a corresponding, specific paragraph in `SKILL.md`, not
  a summary or omission. Confirm tone/section-header style matches
  `collections-management/SKILL.md` and `character-model-sheet/SKILL.md` (imperative
  workflow steps, `###` numbered subsections under `## Workflow`, no marketing language).
- [ ] **Step 10: Commit.**

---

### Task 2: Write references/style-lock-recipe.md

**Files:**
- Create: `skills/photographic-icon-set-generator/references/style-lock-recipe.md`

**Interfaces:**
- Consumes: `skills/photographic-icon-set-generator/SKILL.md` (Task 1) — the "Before you
  start" pointer and workflow step 2 this file fulfills
- Produces: `references/style-lock-recipe.md`, linked from `SKILL.md`'s intro and Task
  7's cross-check

- [ ] **Step 1: Write the "why a style block, not per-icon description" section.**
  Explain the load-bearing fact from the spec's Architecture section: because
  `style_type`/`negative_prompt`/`magic_prompt_option`/`seed` are ignored on the default
  v4 pipeline, cross-icon consistency has no generation-parameter lever — it can only be
  bought by repeating the exact same descriptive text across every `prompts[]` entry.
  State the mechanical failure mode this guards against: describing the material/lighting
  slightly differently for icon #3 than for icon #1 (a different color word, a looser
  lighting phrase) produces visible drift in the rendered set even though each individual
  prompt reads fine in isolation, because Ideogram has no memory of "the icon I rendered
  earlier in this batch" — every entry in `prompts[]` is generated independently from the
  words actually present in that one string.
- [ ] **Step 2: Write the five fixed axes checklist.** For each axis, state what it
  covers and give one line of guidance on writing it so it holds up when repeated
  verbatim across many subjects:
  - **Render/material technique** — the literal rendering method (3D-rendered, claymation/
    clay-sculpt, hand-painted/gouache, isometric-with-material-texture) named explicitly,
    not implied by adjectives like "cute" or "modern."
  - **Lighting setup** — a specific light description (e.g., "soft three-point studio
    lighting, warm key light from upper-left, soft fill, subtle rim light") stated once,
    since "nice lighting" left unspecified is what invites Ideogram to invent a different
    mood per icon.
  - **Camera angle/perspective** — an exact viewing angle (e.g., "three-quarter view from
    slightly above, 30-degree camera tilt" or "straight-on isometric projection") named
    once so every icon in the set reads from the same virtual camera.
  - **Background/ground treatment** — the literal ground/backdrop the subject sits on or
    against (e.g., "seated on a matte pastel-blue rounded pedestal, soft contact shadow,
    plain light-gray backdrop"), since an unspecified background is one of the most common
    sources of visible inconsistency across a batch.
  - **Color story** — the palette's role-based description (primary material color,
    accent color, backdrop color), stated as named roles rather than an unweighted
    adjective list, matching `ideogram-prompt`'s guidance to prefer exact hex/named-role
    color direction over vague color adjectives.
- [ ] **Step 3: Write one worked example style block for 3D-rendered.** A full,
  copy-pasteable paragraph (not a template with blanks) covering all five axes, e.g.:
  "Rendered as a glossy 3D toy-figure icon with soft matte-plastic material and subtle
  ambient-occlusion shading; lit with soft three-point studio lighting, a warm key light
  from the upper-left, gentle fill, and a subtle cool rim light along the right edge;
  shown from a three-quarter view tilted slightly downward, as if photographed on a small
  turntable; seated on a rounded pastel-blue pedestal with a soft contact shadow beneath
  it, set against a plain, seamless light-gray studio backdrop; primary material color
  warm coral-orange, accent details in cream-white, pedestal in dusty pastel blue."
- [ ] **Step 4: Write one worked example style block for claymation.** A full,
  copy-pasteable paragraph, e.g.: "Rendered as a hand-sculpted claymation/stop-motion
  figure with visible fingerprint and tool-mark texture in the clay surface and a soft
  matte, slightly waxy clay sheen; lit with warm, diffused stop-motion set lighting, a
  soft key light from the front-left and gentle fill, minimal harsh shadow; shown from a
  gentle three-quarter angle at eye level, as if photographed on a miniature tabletop
  set; resting on a small rough-textured clay base with a soft grounded shadow, set
  against a plain warm off-white studio backdrop; primary clay color muted terracotta,
  accent details in cream and moss-green, base in warm gray clay."
- [ ] **Step 5: Write one worked example style block for hand-painted.** A full,
  copy-pasteable paragraph, e.g.: "Rendered as a hand-painted gouache illustration with
  visible brushstroke texture, soft paper grain showing through thin paint layers, and
  gently imperfect, organic edges; lit with soft, even natural-light-style illumination
  with minimal cast shadow, consistent with a flat painted study rather than a
  photographed scene; shown from a straight-on front three-quarter view; painted directly
  onto a warm cream paper ground with a soft painted shadow beneath the subject, no hard
  background edge; primary pigment color muted sage-green, accent details in warm ochre
  and soft rust-red, paper ground warm cream."
- [ ] **Step 6: Write one worked example style block for isometric-with-material-texture.**
  A full, copy-pasteable paragraph, e.g.: "Rendered as a detailed isometric 3D object with
  realistic material textures (brushed metal, matte plastic, or fabric weave as
  appropriate to the subject) rather than flat isometric color blocks; lit with clean,
  neutral studio lighting from directly above and slightly to the left, soft shadows, no
  dramatic contrast; shown in true isometric projection (equal 120-degree axes, no
  perspective convergence); floating slightly above a plain rounded platform tile in
  muted slate-gray with a soft drop shadow, set against a plain white background; primary
  material color deep navy-blue, accent details in warm brass/gold, platform tile in
  muted slate-gray."
- [ ] **Step 7: Write a short closing note on subject-block composition.** State the
  companion rule from the spec's workflow step 3: each per-icon prompt is `[style block,
  identical] + [subject block, varies] + [composition block: isolated single subject,
  centered, consistent margin, consistent ground/shadow treatment]`. Give one line of
  guidance on the subject block specifically: name the subject concretely (e.g. "a simple
  rounded house/home icon" rather than "home"), and keep it short — the style block is
  already carrying the bulk of the description, so an overloaded subject block risks
  pulling the icon away from the locked recipe.
- [ ] **Step 8: Self-review pass.** Re-read against the design spec's Components bullet
  for this file ("the fixed axes... plus one worked example style block for each of the
  four named styles") — confirm all five axes are covered and all four worked examples
  are complete, specific paragraphs with no bracketed placeholders.
- [ ] **Step 9: Commit.**

---

### Task 3: Write references/icon-anti-slop-discipline.md

**Files:**
- Create: `skills/photographic-icon-set-generator/references/icon-anti-slop-discipline.md`

**Interfaces:**
- Consumes: `skills/photographic-icon-set-generator/SKILL.md` (Task 1) — the "Before you
  start" pointer and workflow step 4 this file fulfills; `references/style-lock-recipe.md`
  (Task 2) — this file's gate is run against prompts already following that file's
  discipline
- Produces: `references/icon-anti-slop-discipline.md`, linked from `SKILL.md`'s intro and
  Task 7's cross-check

- [ ] **Step 1: Write the framing paragraph**, matching
  `character-model-sheet/references/anti-slop-discipline.md`'s register: state this file
  is the gate run **before** calling `generate_images_bulk`, not a style suggestion to
  eyeball after generating, and that because `negative_prompt` is ignored on the default
  v4 pipeline (per the Global Constraints), every ban below is phrased as a positive
  instruction to embed directly in the prompt text, never as a negative-prompt string.
- [ ] **Step 2: Write the "glossy-app-icon-store clichés" section.** Name the specific
  defaults Ideogram reaches for regardless of the requested material: floating drop
  shadows detached from the subject (instead of a grounded contact shadow matching the
  locked ground treatment), gradient bubble/circle backgrounds behind the subject
  (instead of the locked plain/staged ground), and a bevel-and-emboss glossy-plastic look
  applied even when the requested material is matte clay, painted gouache, or brushed
  metal. State the positive-instruction fix for each: name the exact shadow treatment
  from the style block explicitly ("soft grounded contact shadow, not a floating drop
  shadow"), name the exact background explicitly rather than leaving "background" unsaid,
  and restate the material's actual surface finish (matte, waxy, brushed, painted) since
  an unstated finish defaults to glossy plastic.
- [ ] **Step 3: Write the "mini scene/illustration drift" section.** State the failure
  mode: an icon that adds a second subject, an implied environment, or a narrative
  moment (e.g., a "home" icon that becomes a small illustrated house-in-a-yard scene
  instead of a single centered icon-object) stops reading as an icon in a set and breaks
  visual rhythm against the other N-1 icons that stayed single-subject. Fix: state
  explicitly in the composition block that the subject is a single, isolated
  icon-object — one clear form, no secondary props, no implied environment beyond the
  locked ground treatment — and that this constraint applies identically to every icon in
  the set regardless of how "busy" a particular subject concept might otherwise invite.
- [ ] **Step 4: Write the "camera-angle/material drift between icons" section.** State
  this is the failure mode specific to *sets* (not present in a single-icon anti-slop
  gate): even with a well-written style block, a batch can still drift — e.g., icon #4
  rendered from a slightly different angle, or icon #7 with a noticeably glossier
  material finish than the rest — usually because the subject block for that one icon
  implicitly pulled the render away from the locked recipe (e.g., a subject whose natural
  form invites a different camera framing, like a tall icon versus a wide one). Fix:
  after generating, this is specifically what workflow step 6 (review as a set) exists to
  catch, and workflow step 7 (correction via `edit_image`/`remix_image` against anchor
  icons) exists to fix — flag that a batch-level anti-slop gate can reduce the odds of
  this happening but can't fully prevent it, since the model still generates each
  `prompts[]` entry independently.
- [ ] **Step 5: Write the pre-generation gate table**, matching
  `character-model-sheet/references/anti-slop-discipline.md`'s table format (score 1-5,
  anything under 3 means revise before generating):

```markdown
| Axis | What you're checking |
|---|---|
| **Style-block fidelity** | Is the style block copy-pasted byte-identical into every `prompts[]` entry, not re-typed or paraphrased per icon? |
| **Subject-block restraint** | Does every subject block name one clear icon-object concretely, with no second subject, implied scene, or narrative moment? |
| **Ground/shadow explicitness** | Does every prompt name the exact grounded shadow and background treatment from the style block, rather than leaving "background" or "shadow" unstated? |
| **Material-finish explicitness** | Does every prompt restate the actual requested surface finish (matte, waxy, painted, brushed) rather than leaving finish to the model's glossy-plastic default? |
| **Set-level restraint check** | Scanning all N drafted prompts together (not one at a time): does any single icon's subject block risk pulling that one icon toward a different camera framing or material read than the rest? |
```

- [ ] **Step 6: Self-review pass.** Re-read against the design spec's Components bullet
  for this file — confirm it covers "generic glossy-app-icon-store clichés," "drift into
  mini scene/illustration territory," "camera-angle/material drift between icons in the
  same batch," every ban phrased as a positive instruction (grep the file for the string
  "negative_prompt" — it should only appear in the framing paragraph explaining why
  positive phrasing is used, never as an instruction to pass one), and the
  pre-generation gate table.
- [ ] **Step 7: Commit.**

---

### Task 4: Write references/set-consistency-workflow.md

**Files:**
- Create: `skills/photographic-icon-set-generator/references/set-consistency-workflow.md`

**Interfaces:**
- Consumes: `skills/photographic-icon-set-generator/SKILL.md` (Task 1) — workflow steps
  5-8 this file expands on
- Produces: `references/set-consistency-workflow.md`, linked from `SKILL.md`'s intro and
  Task 7's cross-check

- [ ] **Step 1: Write the batch-generate section.** Expand on `SKILL.md` workflow step 5:
  `generate_images_bulk` is a background job per the tool's own description — it doesn't
  block the conversation, and images render into a carousel as they complete in
  carousel-capable clients. In a text-only client, or when the user asks for status, call
  `get_generation_status` (optionally with the specific `request_id`) to get the
  markdown status table. State that `generate_images_bulk` requires a Pro subscription
  per the tool's own description, and that a subscription-tier error must be surfaced
  verbatim to the user rather than silently falling back to looping individual
  `generate_image` calls (which changes cost and behavior without telling the user).
- [ ] **Step 2: Write the review-as-a-set section.** Expand on workflow step 6: once all N
  images have rendered, look at the full set together (not one at a time in isolation)
  and check the three axes from `style-lock-recipe.md`'s checklist that are hardest to
  hold across a batch — material/finish consistency, lighting consistency, and
  camera-angle consistency. State this explicitly as the core value proposition of the
  skill: a per-icon review would approve each individual image (each one, alone, looks
  fine) and only a side-by-side set review catches the icon that's subtly off-angle or a
  shade glossier than its siblings.
- [ ] **Step 3: Write the correction decision section** (`remix_image` vs. `edit_image`).
  State the decision rule from the spec: prefer `edit_image` with `image_response_ids`
  set to 2-3 anchor icons (response_ids from the finished batch that clearly show the
  target style) plus a corrective prompt describing the fix, since multi-reference
  conditioning pulls the result toward the group in a way a single reference can't as
  reliably. Use `remix_image` against exactly one anchor image (pass its
  `image_response_id`, tune `image_weight` toward the higher end, e.g. 70-85, for "match
  this closely") as the simpler fallback when only one strong anchor icon exists in the
  set. State plainly (per both tools' own descriptions) that `remix_image` and
  `edit_image` fail loudly and explicitly rather than silently substituting a fresh
  `generate_image`/`generate_images_bulk` call if the requested operation can't proceed —
  surface that failure to the user verbatim and stop, don't fall back to a "similar"
  regeneration.
- [ ] **Step 4: Write the "don't regenerate the whole batch" rule with its reasoning.**
  State why: a fresh `generate_images_bulk` call has no better odds of consistency than
  the first attempt unless the style-block text itself is also fixed (since the model has
  no memory of the first batch), and a full regeneration discards the icons that already
  matched the target look, which is wasted work and wasted spend on a Pro-gated call.
- [ ] **Step 5: Write the extend-the-set section.** Expand on workflow step 8: use the
  same multi-reference `edit_image` pattern (2-3 existing icons' `image_response_ids` + a
  prompt for the new subject, following the exact subject-block phrasing style already
  used for the rest of the set) rather than a fresh `generate_images_bulk` call, since a
  new batch call has no memory of the earlier batch's exact style-block wording and is
  therefore a weaker consistency guarantee than conditioning directly on the existing
  images.
- [ ] **Step 6: Write the custom-model-training escalation path section.** State when to
  raise it: a project whose icon count and consistency needs have grown past what
  prompt-text-only style-locking is holding up (10+ icons, or icons generated across
  multiple sessions where drift has crept in). State the mechanism: train a small custom
  model on 5-10 already-locked icons via `custom-model-training` (if/when that skill is
  built in this repo — note plainly that as of this writing it exists only as a design
  spec at `docs/superpowers/specs/2026-07-30-custom-model-training-design.md`, not a
  shipped skill), then pass the resulting `custom_model_uri` into `generate_images_bulk`,
  which unlocks `style_type` and `seed` as real levers on that custom (v3) pipeline. State
  this is documented here as an option, never auto-invoked — matching
  `collections-management`'s "explicit trigger only" discipline for its own non-auto-wiring
  into sibling skills.
- [ ] **Step 7: Write the honest unverified-facts callout on shared `seed`.** State
  plainly, per the repo-wide rule against stating unverified third-party API behavior as
  fact: it's unconfirmed whether passing a shared `seed` value across `prompts[]` entries
  with genuinely different subjects meaningfully improves material/lighting consistency.
  Seed reproducibility is documented for identical-prompt reruns, not for a batch of
  varying subject prompts sharing one seed. Treat trying a shared `seed` (once
  `custom_model_uri` makes `seed` a live parameter) as worth comparing against a
  no-seed baseline on a real project, not as a guaranteed consistency lever to assume up
  front.
- [ ] **Step 8: Self-review pass.** Re-read against the design spec's Components bullet
  for this file — confirm all five pieces are present: the full batch → review →
  correct → extend workflow, the `remix_image` vs. `edit_image` decision, the
  custom-model-training escalation path, and the unverified-`seed` callout.
- [ ] **Step 9: Commit.**

---

### Task 5: Run the real worked example and write examples/<name>/RESULT.md

**Files:**
- Create: `skills/photographic-icon-set-generator/examples/<worked-example-name>/RESULT.md`
  (name TBD at execution time based on the icon subject actually generated — e.g.
  `claymation-recipe-app-icons/` if the recipe-app icon set from the spec's Testing
  section item 1 is used as the real run)
- Test: live MCP tool calls (`mcp__ideogram__generate_images_bulk`,
  `mcp__ideogram__get_generation_status`, and at least one of
  `mcp__ideogram__edit_image`/`mcp__ideogram__remix_image`) — the "test" for this task is
  that these calls actually run against the connected Ideogram MCP and produce real,
  non-fabricated identifiers, per the spec's Components bullet: "one worked example,
  produced by actually running the pipeline once."

**Interfaces:**
- Consumes: `skills/photographic-icon-set-generator/SKILL.md` (Task 1) — this task
  follows the workflow it documents, exercising steps 1-7 for real;
  `references/style-lock-recipe.md` (Task 2) — the style block used in the real run is
  drafted following this file's checklist; `references/icon-anti-slop-discipline.md`
  (Task 3) — the drafted prompts are scored against this file's gate before generating
- Produces: a real style block, real per-icon prompts, real `response_id`s, a real
  correction or extension call, and a documented `RESULT.md` that Task 6's evals and
  Task 7's cross-check can reference as proof the workflow works end-to-end

- [ ] **Step 1: Choose a small real icon set and draft the style block.** Pick a
  6-8-icon set with a clear, concrete use case (e.g., a small recipe-app icon set: home,
  search, favorites, profile, settings, camera) and one of the four named styles (e.g.,
  claymation). Draft the style block by hand-adapting one of Task 2's worked examples to
  this specific project rather than reusing it verbatim unchanged, so the example
  demonstrates real drafting, not copy-paste. Record the exact style-block text used.
- [ ] **Step 2: Draft the per-icon prompts.** For each of the 6-8 icons, write
  `[style block, identical] + [subject block] + [composition block]` following
  `style-lock-recipe.md`'s pattern. Record all prompts verbatim.
- [ ] **Step 3: Run the anti-slop gate.** Score the drafted prompts against
  `icon-anti-slop-discipline.md`'s pre-generation gate table (all five axes). Record the
  scores; if anything scores under 3, revise the relevant prompt(s) before generating and
  record the revision.
- [ ] **Step 4: Call `mcp__ideogram__generate_images_bulk`** with the drafted `prompts`
  array, `aspect_ratio: "1x1"`, and `rendering_speed: "QUALITY"`. Record the real
  `request_id`(s)/`response_ids` returned in the tool's structured content.
- [ ] **Step 5: Call `mcp__ideogram__get_generation_status`** (if the batch is still
  running, or to confirm completion) and record the real status table / structured
  content once all icons have rendered.
- [ ] **Step 6: Review the finished set as a set.** Look at all N generated icons
  together and note, honestly, whether any icon drifted in material, lighting, or camera
  angle from the rest — do not fabricate a drift finding if the set actually came out
  consistent; if it did come out fully consistent on the first pass, say so plainly and
  still proceed to Step 7 using the extend-the-set path instead of a correction, so the
  worked example still exercises a real `edit_image`/`remix_image` call end-to-end.
- [ ] **Step 7: Run one real correction or extension call.** Either (a) if an outlier was
  found in Step 6, call `mcp__ideogram__edit_image` with `image_response_ids` set to 2-3
  anchor icons' real `response_id`s plus a corrective prompt, or (b) if the set was
  already consistent, call the same `edit_image` pattern to add one new icon to the set
  (e.g., a "notifications" icon) using 2-3 existing icons as anchors. Record the real
  prompt used, the anchor `response_id`s passed, and the resulting `response_id`.
- [ ] **Step 8: Write `examples/<worked-example-name>/RESULT.md`**, matching
  `collections-management/examples/anchorpoint-logo-collection/RESULT.md`'s format: an
  "## Identifiers" section with the real `response_id`s and image URLs for every icon
  generated (both the original batch and the correction/extension call); the exact style
  block and per-icon prompts used; the anti-slop gate scores from Step 3; and a short
  narrative of what the set-level review in Step 6 found. No fabricated values — every
  identifier in this file must be a real value copied from a tool response captured in
  Steps 4-7.
- [ ] **Step 9: Save the set style spec JSON** for this example at
  `examples/<worked-example-name>/set-style-spec.json`, following the shape defined in
  `SKILL.md`'s "The 'set style spec' artifact" section (`style_block`, `icons[]`,
  `corrections[]`/`extensions[]`), populated with the real values from this run — this
  doubles as the first real proof the artifact shape is usable, not just described.
- [ ] **Step 10: Self-review pass.** Confirm every identifier, prompt, and score in
  `RESULT.md` and `set-style-spec.json` traces back to a real tool response or a real
  drafting decision captured during Steps 1-7, with no invented or "typical" values.
- [ ] **Step 11: Commit.**

---

### Task 6: Write evals/evals.json

**Files:**
- Create: `skills/photographic-icon-set-generator/evals/evals.json`

**Interfaces:**
- Consumes: `skills/photographic-icon-set-generator/SKILL.md` (Task 1) — the workflow and
  error-handling language these prompts are written to probe; the spec's Testing section's
  3 named scenarios
- Produces: `evals/evals.json`, referenced by Task 7's cross-check against the spec's
  Testing section

- [ ] **Step 1: Write the file's top-level shape** —
  `{"skill_name": "photographic-icon-set-generator", "evals": [...]}`, matching
  `collections-management/evals/evals.json`'s top-level keys exactly.
- [ ] **Step 2: Write eval 1 (fresh-set-generation)** — `id: 1`,
  `eval_name: "claymation-icon-set-shared-style-block"`, `prompt`: "Generate a set of 8
  photographic 3D-rendered claymation-style app icons for a recipe app: home, search,
  favorites, profile, settings, notifications, cart, camera" (from the spec's Testing item
  1), `expected_output`: describes the skill gathering the icon list and confirming the
  claymation style per workflow step 1, drafting a style block once per
  `style-lock-recipe.md`'s checklist and repeating it byte-identical across all 8
  `prompts[]` entries per workflow step 2-3, running the anti-slop gate before generating
  per workflow step 4, calling `generate_images_bulk` with a shared `aspect_ratio` and
  `rendering_speed: "QUALITY"`, and leaving `style_type`/`seed`/`negative_prompt`/
  `magic_prompt_option` unset since no `custom_model_uri` is in play. `files: []`.
- [ ] **Step 3: Write eval 2 (extend-existing-set)** — `id: 2`,
  `eval_name: "extend-existing-set-with-multi-reference-edit"`, `prompt`: "Add a 'dark
  mode' icon to my existing 3D claymation icon set to match the others" (from the spec's
  Testing item 2), `expected_output`: describes the skill recognizing this as workflow
  step 8 (extend the set) rather than step 1 (a fresh set), using `edit_image` with 2-3
  existing icons' `image_response_ids` as anchors plus a prompt for the new "dark mode"
  subject following the same subject-block phrasing style as the rest of the set (or
  `remix_image` against one anchor as the documented simpler fallback if only one strong
  anchor is available), rather than a fresh, memory-less `generate_images_bulk` call —
  explicitly calling out that a new batch call would have no memory of the original
  batch's exact style-block wording. `files: []`.
- [ ] **Step 4: Write eval 3 (flat-vector-out-of-scope)** — `id: 3`,
  `eval_name: "flat-line-icon-set-declines-out-of-scope"`, `prompt`: "I need a flat line
  icon set for my nav bar, 12 icons" (from the spec's Testing item 3), `expected_output`:
  describes the skill recognizing this as a flat/vector/line request per the frontmatter's
  explicit non-trigger and workflow step 1's scoping check, stating plainly that this
  skill is scoped to raster/photographic/material-textured icon styles and this request
  is out of scope, and not proceeding to draft a style block or call
  `generate_images_bulk` — explicitly not offering a raster substitute for what should be
  scalable vector output, per the Error handling rule. `files: []`.
- [ ] **Step 5: Do not add an `assertions` field to any eval entry.**
  `collections-management/evals/evals.json` shipped with `id`/`eval_name`/`prompt`/
  `expected_output`/`files` only at first pass. Match that starting shape for this skill;
  leave assertions for a later eval cycle.
- [ ] **Step 6: Validate JSON** — run the file through a JSON parser (e.g.,
  `python3 -m json.tool skills/photographic-icon-set-generator/evals/evals.json`) to
  confirm it's syntactically valid before committing.
- [ ] **Step 7: Self-review pass.** Confirm the 3 prompts match the spec's Testing
  section's 3 scenarios verbatim in intent (fresh-set-generation, extend-existing-set,
  flat-vector-out-of-scope), and that `expected_output` for each ties back to a specific
  rule in `SKILL.md`'s Error handling or Workflow sections rather than a generic
  description.
- [ ] **Step 8: Commit.**

---

### Task 7: Cross-check finished skill against the spec

**Files:**
- No new files created — this task only reviews and, if needed, patches
  `skills/photographic-icon-set-generator/SKILL.md` and the three
  `skills/photographic-icon-set-generator/references/*.md` files for gaps found during
  review.

**Interfaces:**
- Consumes: all files from Tasks 1-6
- Produces: a confirmed-complete `skills/photographic-icon-set-generator/` skill; nothing
  new for later tasks to consume (final task)

- [ ] **Step 1: Re-read `2026-07-30-photographic-icon-set-generator-design.md`'s "Error
  handling" section line by line** against `SKILL.md`'s Error handling section. Confirm
  all 6 rules (flat/vector request → decline; undecided style → ask, don't invent a
  default; Pro-subscription error → surface and ask, don't silently loop
  `generate_image`; partial batch failure → report exactly which icons failed; drift
  detected → targeted correction first, not a blind full regeneration; user-preferred
  outlier → confirm the intended anchor before conforming others) are present with
  specific, non-generic language. Patch `SKILL.md` if any rule is missing or vague.
- [ ] **Step 2: Re-read the spec's "Out of scope" section** and confirm the finished
  skill does not contain any of the four excluded items: no flat/line/glyph/vector
  generation logic anywhere in `SKILL.md` or the three reference files (only the explicit
  decline-and-stop language); no `remove_background` call described as part of the v1
  workflow; no automatic (non-user-initiated) invocation of `custom-model-training`
  anywhere — grep for "custom-model-training" and confirm every occurrence is phrased as
  an optional, user-raised escalation, never an automatic step; no icon-preview grid UI or
  export/compositing into favicons/app-icon bundles/platform manifests described as part
  of this skill's job.
- [ ] **Step 3: Re-read the spec's "Data flow / workflow" section's 9 steps** against
  `SKILL.md`'s `## Workflow` subsections one-by-one, confirming no step was compressed,
  skipped, or reordered.
- [ ] **Step 4: Re-read the spec's "Assumptions (autonomous run)" section** and confirm
  each assumption made during planning is reflected consistently in the shipped files:
  default `aspect_ratio: "1x1"` (grep `SKILL.md` and `set-consistency-workflow.md` for
  consistency), default `rendering_speed: "QUALITY"`, the "set style spec" JSON as a
  deliberately new (not reused) artifact type, `remove_background` excluded from v1 scope,
  and no artificial cap below `generate_images_bulk`'s 1-500 `prompts[]` limit paired with
  the "sanity-check unusually large batches with the user first" rule.
- [ ] **Step 5: Type/naming consistency scan.** Grep across `SKILL.md`, all three
  `references/*.md` files, `examples/<worked-example-name>/RESULT.md`,
  `examples/<worked-example-name>/set-style-spec.json`, and `evals/evals.json` for:
  `style_block`, `response_id`, `image_response_ids`, `image_response_id`,
  `image_weight`, `custom_model_uri`, `style_type`, `negative_prompt`,
  `magic_prompt_option`, `seed`, `aspect_ratio`, `rendering_speed`. Confirm every
  occurrence uses the exact same casing and name in every file — no drift like
  `styleBlock` vs. `style_block`, or `image_response_id` (singular) used where
  `image_response_ids` (plural, the `edit_image` parameter) is actually meant.
- [ ] **Step 6: Placeholder scan.** Grep all files under
  `skills/photographic-icon-set-generator/` for generic filler phrases ("add appropriate
  error handling," "TODO," "[insert...]," "e.g. some value") and confirm none exist —
  every sentence should state real, specific content.
- [ ] **Step 7: Commit any patches made in Steps 1-6** (skip if no patches were needed).

---

## Self-Review

**Spec coverage:**
- **Why / Scope** (raster/photographic-only, set-not-single-icon, explicit flat/vector
  non-trigger) — covered by Task 1 Step 1 (frontmatter) and enforced by Task 7 Step 2 and
  Task 6 Step 4 (the eval 3 scenario).
- **Architecture** (hybrid skill: prompt-composition + orchestration; the style-block
  load-bearing fact; the deliberate schema deviation for the "set style spec" artifact;
  the optional escalation path) — covered by Task 1 Step 2 (intro paragraph), Task 1 Step
  5 (the new artifact section), and Task 4 Step 6 (escalation path reference file
  section).
- **Components** (`SKILL.md`, `references/style-lock-recipe.md`,
  `references/icon-anti-slop-discipline.md`, `references/set-consistency-workflow.md`,
  `examples/`, `evals/evals.json` — each with the specific content described in the spec)
  — one task per component: Task 1 (`SKILL.md`), Task 2 (`style-lock-recipe.md`), Task 3
  (`icon-anti-slop-discipline.md`), Task 4 (`set-consistency-workflow.md`), Task 5
  (`examples/`), Task 6 (`evals.json`).
- **Data flow / workflow steps 1-9** (gather, lock style block, draft prompts, anti-slop
  gate, generate batch, review as a set, correct outlier, extend later, save) — covered
  by Task 1 Step 4's nine subsections, exercised for real (steps 1-7) in Task 5, and
  cross-checked in Task 7 Step 3.
- **Error handling** (6 rules) — covered by Task 1 Step 6, verified in Task 7 Step 1.
- **Testing** (3 eval prompts matching the spec's named scenarios) — covered by Task 6
  Steps 2-4, verified in Task 6 Step 7.
- **Out of scope** (no flat/vector generation, no `remove_background`, no automatic
  custom-model-training invocation, no preview-UI/export-compositing) — explicitly
  enforced and checked in Task 7 Step 2.
- **Assumptions (autonomous run)** (`1x1` default aspect ratio, `QUALITY` default
  rendering speed, the new artifact-type decision, `remove_background` exclusion, no
  artificial batch cap) — each assumption is threaded through the relevant task (Task 1
  Step 4 subsection 5 for the generation defaults, Task 1 Step 5 for the artifact
  decision) and re-verified as a group in Task 7 Step 4, so a reviewer can confirm the
  autonomous calls made during spec-writing actually landed in the shipped skill rather
  than drifting during implementation.

**Placeholder scan confirmation:** Every task step above specifies exact section names,
exact tool names, exact worked-example prompt text (drafted concretely in Task 2 and Task
5, not left as a template with blanks), and exact file paths — no task step says "add
appropriate X" or leaves content undetermined except Task 5's worked-example directory
name, which is explicitly flagged as "TBD at execution time" because it depends on which
concrete icon set is actually chosen and generated, not left vague by omission. Task 7
Step 6 adds a dedicated placeholder grep as a final gate.

**Type/naming consistency check:** `style_block`, `response_id`/`image_response_ids`,
`image_weight`, `custom_model_uri`, `style_type`, `negative_prompt`,
`magic_prompt_option`, `seed`, `aspect_ratio`, `rendering_speed` are used with identical
spelling and casing across Task 1 (`SKILL.md`), Tasks 2-4 (the three reference files),
Task 5 (`RESULT.md`/`set-style-spec.json`), and Task 6 (`evals.json`) in this plan —
pulled directly from the live MCP tool schemas confirmed during planning (see Global
Constraints) and the spec's Data flow section, rather than re-derived per task. Task 7
Step 5 re-verifies this against the actual shipped files, not just this plan document,
since drift could still be introduced during writing.

**Noted ambiguity resolved during planning (autonomous run, no human available):** the
spec's Components section describes `references/icon-anti-slop-discipline.md`'s gate
table as covering the icon-specific failure modes but doesn't specify the exact axis
names or scoring scale; this plan (Task 3 Step 5) follows
`character-model-sheet/references/anti-slop-discipline.md`'s established 1-5 scale and
"anything under 3 means revise" threshold for consistency with the repo's existing
pre-generation gate convention, rather than inventing a different scale.
