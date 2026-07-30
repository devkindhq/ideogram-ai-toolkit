# Photographic Icon Set Generator Skill — Design

## Why

Gap-research (`[redacted-internal-path]`
in the internal notes, dated 2026-07-30, Gap 2) confirms `jezweb/claude-skills` ships
`icon-set-generator` — hand-written SVG code, no AI/raster image generation at all — and,
separately, a general-purpose `ai-image-generator` (Gemini/GPT-Image backend) that lists
"icons" among its supported output types but applies no cross-icon consistency discipline
and isn't Ideogram-backed. No competitor combines (a) a dedicated icon-**set** consistency
system — a shared style spec enforced across many icons, the way jezweb's vector skill
enforces identical viewBox/stroke/`currentColor` rules — with (b) raster/photographic
output and (c) this repo's structured-JSON-caption/anti-slop/compositional-deconstruction
discipline.

The research validates this as a real, narrow gap: **photographic or textured icon
styles — 3D-rendered, claymation, hand-painted, isometric-with-material-texture — are
structurally impossible for vector-code tools to represent** (materials, lighting, 3D
form, photographic texture don't exist in stroke/fill SVG). Where the requested style is
flat/line/glyph/vector, jezweb's mature, free, scalable, themeable tool remains the
strictly better choice, and this skill should say so rather than forcing a raster
substitute. That scoping boundary is load-bearing for this spec, not a footnote.

The connected Ideogram MCP exposes the tool surface needed, confirmed live via direct
tool-schema inspection (not assumed from docs): `generate_images_bulk` (primary —
batch generation from a `prompts` array, 1-500 entries, one shared `aspect_ratio` /
`resolution` / `rendering_speed` across the whole batch), `remix_image` (single-reference
variation, `image_weight`-tuned similarity), and `edit_image` (literal edit, supports
**multiple** `image_response_ids` conditioning one new image on all of them at once).

## Scope

One new skill: `skills/photographic-icon-set-generator/`. Triggers on requests for a
**set** of icons in a photographic/textured/3D/material-rendered style — "3D icon set,"
"claymation icons," "isometric icon pack with material texture," "hand-painted icon
set," or an explicit request to match/extend an existing photographic icon set. Does
**not** trigger on requests for flat, line, glyph, minimal, or generic "vector-style"
icon sets — those are a different mechanism (deterministic SVG code, not diffusion
output) and this skill should say so plainly rather than quietly generating a raster
substitute that's worse on every axis that matters for that use case (not scalable, not
themeable via `currentColor`, not code-native).

A **set**, not a single icon — the differentiator is cross-icon consistency (same
lighting, same material, same camera angle/perspective held across every icon in the
batch), not single-icon quality. A one-off "generate me a 3D camera icon" request is
just `generate_image` from `ideogram-prompt`; this skill exists for the multi-icon
consistency problem specifically.

## Architecture

Structural pattern matches the existing 6 skills: `SKILL.md` + `references/` +
`examples/` + `evals/`. This skill is a **hybrid** — no existing skill in this repo
needs both halves it needs simultaneously:

- Like the visual-generation skills (`logo-prompting`, `brand-identity-sheet`,
  `character-model-sheet`, `moodboard-generator`), it requires careful prompt
  composition and an anti-slop gate before generating.
- Like `collections-management` and `custom-model-training`, it is also an
  **orchestration workflow** across sequential MCP calls — generate the batch, review
  the result *as a set*, then correct outliers or extend the set via `remix_image` /
  `edit_image` in later turns. Prompt composition alone doesn't cover this; the
  consistency work happens across calls, not within one.

**Load-bearing technical fact, confirmed directly from `generate_images_bulk`'s own
tool-schema description, not inferred:** on Ideogram's default v4 pipeline (no
`custom_model_uri` set), the `style_type`, `negative_prompt`, `magic_prompt_option`, and
`seed` parameters are **all ignored** — they only take effect on a custom-model (v3)
generation. This means cross-icon consistency cannot be bought through generation
parameters beyond the batch-shared `aspect_ratio` / `resolution` / `rendering_speed`.
It must be carried entirely by a **repeated, held-byte-identical "style block" of prompt
text** across every entry in the `prompts[]` array — the same "front-load the constant
description, vary only the subject" discipline `character-model-sheet` uses across
panels *within* one image, applied here across *separate* images in one batch call. This
is the single design decision the rest of the skill hangs off, and it's the reason a
naive "just call `generate_images_bulk` with N different prompts" approach fails: without
a deliberately shared style block, the batch has no consistency mechanism at all.

**Deliberate deviation from the sibling schema pattern (flagged explicitly, not an
oversight):** the sibling visual-generation skills all produce a single multi-panel
sheet image and reverse-engineer it into `composition-spec-format.md`'s shared bbox JSON
(`high_level_description` + `elements[]` with per-panel bounding boxes). That schema
exists to make a multi-panel *layout* reusable. This skill's deliverable is the opposite
shape — N separate single-subject images, each just one icon on a plain ground, not one
image with panels to annotate. Bbox-annotating "the icon" against "the background" inside
one already-simple image doesn't carry the same reuse value panel-annotation carries for
a 9-panel moodboard. Instead this skill introduces a **new artifact type**: a "set style
spec" JSON recording the locked style block verbatim, the per-icon subject list, and the
resulting `response_id`/image URL per icon. That's the artifact with real reuse value
here — it's what lets a later session add icon #9 to an existing 8-icon set without
re-deriving the recipe from scratch, mirroring what jezweb's vector skill gets for free
from its `style-spec.json`.

**Optional escalation path, not required for v1:** because `style_type`/`seed` are dead
weight on the default pipeline, a project that needs very tight consistency across a
large set (10+ icons, or icons generated across multiple sessions) can chain into
`custom-model-training` — train a small custom model on 5-10 already-locked icons, then
pass `custom_model_uri` into `generate_images_bulk`, which unlocks `style_type`/`seed` as
real levers. This skill documents that escalation path in
`references/set-consistency-workflow.md` but doesn't invoke it automatically — same
"explicit trigger only" discipline `collections-management` uses for its own
non-auto-wiring into sibling skills.

## Components

- **`SKILL.md`** — frontmatter description triggers on "3D icon set," "claymation
  icons," "textured/material icon pack," "isometric icon set with material texture,"
  "photographic icon set," "match this icon style," "add an icon to my [style] set." States
  the flat/vector non-trigger explicitly in the description itself (not buried in the
  body) so the routing decision is visible at the trigger-matching layer. Body documents
  the workflow below.
- **`references/style-lock-recipe.md`** — the style-block discipline: the fixed axes
  that must be named once and repeated identically across every prompt in the batch
  (render/material technique, lighting setup, camera angle/perspective, background/ground
  treatment, color story), plus one worked example style block for each of the four named
  styles (3D-rendered, claymation, hand-painted, isometric-with-material-texture) so a
  session has a concrete starting point instead of inventing phrasing from nothing.
- **`references/icon-anti-slop-discipline.md`** — icon-specific failure modes distinct
  from the sibling skills' scene/panel-level slop: generic glossy-app-icon-store clichés
  (floating drop shadows, gradient bubble backgrounds, bevel-and-emboss plastic look
  regardless of the requested material), icons that drift into "mini scene/illustration"
  territory and stop reading as a single-subject icon, and camera-angle/material drift
  between icons in the same batch. Because `negative_prompt` is dead weight on the
  default pipeline (see Architecture), every ban in this file is phrased as a positive
  instruction to embed in the prompt, not a negative-prompt string. Includes the
  pre-generation gate table (score before calling `generate_images_bulk`).
- **`references/set-consistency-workflow.md`** — the batch-generate → review-as-a-set →
  correct-outliers → extend-later workflow (full detail in Data flow below), the
  `remix_image` vs. `edit_image` decision (single-reference similarity tuning vs.
  multi-reference conditioning against 2-3 anchor icons), the custom-model-training
  escalation path, and an honest unverified-facts callout: it's unconfirmed whether
  passing a shared `seed` across prompts with genuinely different subjects meaningfully
  improves material/lighting consistency (seed reproducibility is documented for
  identical-prompt reruns, not for varying-prompt batches) — treat it as worth trying and
  comparing, not a guaranteed lever, consistent with the repo-wide rule against stating
  unverified third-party API behavior as fact.
- **`examples/`** — one worked example, produced by actually running the pipeline once
  (generate a small set, deliberately review it for drift, run at least one correction or
  extension call) — empty until that run happens.
- **`evals/evals.json`** — 3 eval prompts in the existing schema (`id`, `eval_name`,
  `prompt`, `expected_output`, `files`), matching `collections-management/evals/evals.json`'s
  format.

## Data flow / workflow

1. **Gather the icon list and style direction.** Ask for: the list of icon
   concepts/subjects, the target count, and one of the four named styles (or a described
   custom raster style) — don't invent a default look silently, since the whole value
   proposition is a *deliberately locked* shared recipe, not "whatever Ideogram feels like
   rendering this time." If the request is actually for flat/line/glyph/vector icons,
   say so now and stop rather than proceeding with a raster substitute (see Error
   handling).
2. **Lock the style block.** Draft it once, following `references/style-lock-recipe.md`'s
   checklist (material/render technique, lighting, camera angle, background/ground,
   color story) as one continuous description written to be repeated byte-identical
   across every icon prompt — not re-described per icon.
3. **Draft one prompt per icon.** Each entry in the eventual `prompts[]` array =
   `[style block, held identical] + [subject block, varies per icon] + [composition
   block: isolated single subject, centered, consistent margin, consistent ground/shadow
   treatment]`, in the icon list's order.
4. **Run the anti-slop gate.** Score the drafted prompts against
   `references/icon-anti-slop-discipline.md`'s pre-generation table before generating —
   regenerating a full batch is more expensive than revising N prompt strings first.
5. **Generate the batch.** Call `mcp__ideogram__generate_images_bulk(prompts=[...])`
   with a shared `aspect_ratio` (default `"1x1"` for icons unless the user wants
   something else), `rendering_speed: "QUALITY"` (matching sibling-skill precedent for
   detail-dense, consistency-sensitive renders), and `resolution` only if the user needs
   exact pixel dimensions. Leave `style_type`, `negative_prompt`, `magic_prompt_option`,
   and `seed` unset by default — they're ignored on the default v4 path per the tool's
   own schema — unless `custom_model_uri` is set via the escalation path in step 8, in
   which case they become real and worth using. This is a background job (per the tool's
   own description); tell the user the images render as they complete, and offer to poll
   `mcp__ideogram__get_generation_status` if the client is text-only rather than
   auto-rendering a carousel.
6. **Review the finished set as a set, not icon-by-icon.** Look at all N results
   together and check material, lighting, and camera-angle consistency across the whole
   batch — the entire reason this skill exists is catching drift a per-icon review would
   miss.
7. **Correct any outlier.** For an icon that drifted from the rest: prefer
   `mcp__ideogram__edit_image` with `image_response_ids` set to 2-3 "anchor" icons from
   the set that show the target style well, plus a corrective prompt describing the fix
   — multi-reference conditioning pulls the result toward the group, which a single
   reference can't do as reliably. Use `mcp__ideogram__remix_image` against one anchor
   image (tune `image_weight` toward the higher end for "match this closely") as the
   simpler fallback when only one strong anchor exists. Don't regenerate the whole batch
   for a partial-drift problem — a fresh `generate_images_bulk` call has no better odds
   of consistency without also fixing the underlying style-block text, and it discards
   the icons that already matched.
8. **Extend the set later.** Same multi-reference `edit_image` pattern (2-3 existing
   icons as `image_response_ids` + a prompt for the new subject) rather than a fresh
   `generate_images_bulk` call — a new batch call has no memory of the earlier batch's
   exact style-block wording, so it's a weaker consistency guarantee than conditioning
   directly on the existing images. If the project's icon count and consistency needs
   have grown past what prompt-text-only style-locking is holding up, this is also the
   point to raise the `custom-model-training` escalation path from Architecture.
9. **Save what was made.** Write the locked style block, the per-icon prompts, the
   resulting `response_id`/image URL per icon, and any correction/extension notes to a
   "set style spec" JSON plus a project markdown file (check for an existing
   `logo-explorations/`, `branding/`, or icon-specific folder first and match it, same
   convention as the sibling skills). Follow "No Context Lost" — nothing generated here
   should live only in the conversation.

## Error handling

- **Flat/vector/line/glyph request** → state plainly this skill is scoped to raster,
  photographic, or material-textured icon styles and this request is out of scope for
  it; don't quietly generate a raster substitute for what should be scalable vector
  output.
- **Undecided style direction** → ask for one of the four named styles or a described
  custom raster style; don't invent a default silently, since an unlocked style means
  there's no recipe to hold constant across the batch.
- **`generate_images_bulk` requires a Pro subscription** (per the tool's own
  description) — if it errors on subscription tier, surface the real error and ask
  before doing anything else; don't silently fall back to looping individual
  `generate_image` calls, since that changes cost and behavior without telling the user.
- **Partial batch failure** (some entries in `prompts[]` fail while others succeed) →
  report exactly which icons failed and why; don't claim the whole set completed.
- **Style drift detected across the set** → run the targeted `remix_image`/`edit_image`
  correction loop (step 7) first; don't regenerate the whole batch blindly, since a fresh
  batch call has the same missing-consistency-lever problem the first attempt had unless
  the style-block text itself is also fixed.
- **Correcting an outlier that the user actually preferred as-is** → confirm which icon
  is the intended "anchor" for the set's look before conforming others to it; don't
  assume the majority of the batch is automatically the correct target style.

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts:
1. "Generate a set of 8 photographic 3D-rendered claymation-style app icons for a
   recipe app: home, search, favorites, profile, settings, notifications, cart, camera"
   — verifies the style block is drafted once and repeated identically across all 8
   `prompts[]` entries, the anti-slop gate runs before generating, a shared
   `aspect_ratio`/`rendering_speed` is used, and `style_type`/`seed`/`negative_prompt`/
   `magic_prompt_option` are left unset (no `custom_model_uri` in play).
2. "Add a 'dark mode' icon to my existing 3D claymation icon set to match the others" —
   verifies the extend-the-set path uses `edit_image` with multiple existing
   `image_response_ids` as reference (or `remix_image` as the documented simpler
   fallback) rather than a fresh, memory-less `generate_images_bulk` call.
3. "I need a flat line icon set for my nav bar, 12 icons" — verifies the skill
   recognizes this as a flat/vector request outside its scope and says so plainly,
   rather than forcing a raster/photographic substitute onto a request that explicitly
   asked for a different, and for this case strictly better-suited, mechanism.

## Out of scope (this spec)

- Flat, line, glyph, or generic "vector-style" icon sets — different mechanism
  (deterministic SVG code), not this skill's job; jezweb's `icon-set-generator` remains
  the better tool for that request.
- Transparent-background/alpha-channel icon output — the connected Ideogram MCP exposes
  a `remove_background` tool, but it wasn't part of this skill's confirmed-relevant tool
  surface for v1 (`generate_images_bulk` / `remix_image` / `edit_image` only). Icons
  render on a deliberately styled, consistent plain/staged ground instead. Worth
  revisiting as a v2 addition if a real need shows up.
- Automatic custom-model training — `custom-model-training` is an optional,
  user-initiated escalation this skill can point to (see Architecture / step 8), not
  something this skill triggers on its own.
- Any icon-preview grid UI, or export/compositing into favicons, app-icon bundles, or
  platform-specific icon manifests — that's a downstream compositing concern, out of
  scope for the generation skill itself.

## Assumptions (autonomous run — no clarification available)

Documented here per this being an unattended run, rather than left implicit:

- Default `aspect_ratio` for icons is `"1x1"` unless the user specifies otherwise —
  reasonable for an "icon" deliverable, not stated in the brief.
- `rendering_speed: "QUALITY"` is the default, matching the precedent set by every
  sibling visual-generation skill for detail-dense, consistency-sensitive renders.
- The "set style spec" JSON is a new artifact type, deliberately not a reuse of
  `composition-spec-format.md`'s bbox schema — reasoning given in full under
  Architecture, flagged there as an explicit pattern deviation rather than an oversight,
  per the project rule to expose pattern conflicts rather than average them.
- `remove_background` is excluded from v1 scope because it wasn't in the
  confirmed-relevant tool list given for this skill, not because it's unavailable on the
  connected MCP — a deliberate scope-narrowing choice, not a technical limitation.
- No artificial cap is imposed below `generate_images_bulk`'s own 1-500 `prompts[]`
  limit, but the skill should sanity-check unusually large batch requests with the user
  first given the Pro-subscription/cost implications, rather than submitting a large job
  silently.
