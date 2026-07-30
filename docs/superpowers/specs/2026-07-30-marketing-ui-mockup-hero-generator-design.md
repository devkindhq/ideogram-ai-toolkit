# Marketing UI Mockup / Hero Generator Skill — Design

## Why

Gap-research (`05-knowledge/technical/ideogram-tier3-differentiation-research-2026-07-30.md`
in the Devkind vault, dated 2026-07-30, Gap 3) found the "UI mockup / landing-page hero"
competitive set is bifurcated: HTML/CSS code-output tools (`baoyu-design`,
`ShadmanSakibRahman`, `borghei`, `inbharatai` — the majority, produce real editable/
responsive code) versus one generic raster tool (`jezweb/claude-skills`'
`ai-image-generator`, Gemini/GPT-Image-backed, no UI-specific compositional discipline).
Neither camp does what this skill does: a raster image, generated with an Ideogram-native
methodology, where a finished visual — not editable code — is the actual deliverable
wanted (landing-page hero shots, product-in-context device mockups, dashboard screenshots
for App Store listings or pitch decks).

A real informal precedent exists (`reviewthestack.com`'s "How I wire up Ideogram and
Claude Code to generate every hero image on this site"), narrowly scoped to 16:9 blog
hero images with baked-in headline text, but with no structured JSON captions, no
compositional-deconstruction schema, and no UI-specific anatomy. An open Anthropic
feature request (`anthropics/claude-code#22618`, "Built-in image generation for UI
mockups, assets, and visual content") signals unmet demand for this category directly
inside Claude Code workflows.

The connected Ideogram MCP already exposes the tool surface needed: `generate_image`,
`edit_image`, `reframe_image` (confirmed live via direct tool-schema inspection). No new
skill in this repo has used `reframe_image` yet — this is the first, and it's the tool
that makes "one hero composition, several marketing formats" (landing page 16:9, social
square, App Store portrait) a distinct capability rather than a re-prompt-and-hope
exercise.

## Scope

One new skill: `skills/marketing-ui-mockup-hero-generator/`. Produces **finished raster
images** of three format types, chosen explicitly so the skill never drifts into
competing with the HTML/CSS tools on their own turf:

1. **Landing-page hero shot** — a full-bleed marketing hero image with headline,
   subheadline, and CTA button copy baked in via Ideogram's text rendering, typically
   showing the product in a browser/device frame or as a standalone visual.
2. **Device-in-context product mockup** — a product's UI shown inside a physical device
   shell (laptop, phone, tablet, browser chrome) placed in an environmental or lifestyle
   context (desk surface, hand, studio backdrop) — the kind of image used on a pricing
   page, a case study, or a sales deck slide.
3. **Dashboard screenshot** — a single stylized app/dashboard screen (nav, cards, charts,
   sidebar) rendered to look polished and "real" for App Store listings, pitch decks, or
   marketing pages — not a literal render of any actual live codebase.

Triggers only on explicit marketing-visual requests ("generate a hero image for the
landing page," "make a device mockup showing the app on a laptop," "I need a dashboard
screenshot for the pitch deck," "App Store screenshot for [product]"). Does not
auto-trigger for requests that are actually asking for working code, a real prototype, or
a literal 1:1 recreation of an existing live product screen — see Error handling.

## Assumptions

Documented here per the autonomous-run instruction, since no one was available to confirm
these mid-task:

- **Three format types, not an open-ended list.** The brief names "hero shots,
  product-in-context device mockups, dashboard screenshots" as the scope; this spec treats
  those as the fixed set (mirroring how `moodboard-generator`'s panel set and
  `brand-identity-sheet`'s panel menu are fixed vocabularies, not open-ended). A fourth
  format request (e.g. "mobile app onboarding flow screens") should be handled by
  composing from the same compositional-layer vocabulary in `references/mockup-anatomy.md`
  rather than the skill refusing — but the three named types are what `SKILL.md`'s
  frontmatter and workflow are written around.
- **`reframe_image` is for same-composition format variants, not new compositions.**
  The brief calls it out for "multi-format hero variants." This spec takes that to mean:
  generate one base hero, then `reframe_image` it to 1-3 additional aspect ratios for
  different placements (landing page vs. social square vs. App Store portrait) — not a
  general-purpose reframe-everything step. Full rationale and the crop-risk caveat is in
  Error handling.
- **Real copy, not lorem ipsum, is the default expectation.** Because Ideogram's
  text-rendering reliability is the differentiator, the skill should push for actual
  headline/CTA/nav copy from the user or the project's brand/marketing docs rather than
  filling panels with placeholder text — closer to `brand-identity-sheet`'s
  locked-truth-required posture than `moodboard-generator`'s fuzzy-tolerant one, since the
  deliverable here is meant to ship in a deck or App Store listing, not explore a
  direction.
- **`style_type` choice is per-format, not fixed.** `DESIGN` reads best for UI-chrome-heavy
  renders (dashboard screenshots, hero shots where the UI panel dominates); `REALISTIC`
  reads best for photographic device-in-context shots (a laptop on a desk, a phone in a
  hand). This is stated as skill guidance, not verified against Ideogram's actual model
  behavior — flagged as an assumption to sanity-check against real output once the skill
  is built and run, same spirit as `custom-model-training`'s honest-unknowns framing.

## Architecture

Structural pattern matches the three sibling **prompt-composition** skills
(`brand-identity-sheet`, `character-model-sheet`, `moodboard-generator`), not the two
**orchestration-workflow** skills (`custom-model-training`, `collections-management`):
`SKILL.md` + `references/` + `examples/` + `evals/evals.json`, producing the same
two-artifact output (a generation prompt + a compositional-deconstruction JSON) as those
three siblings, via the same shared JSON schema.

New reference file, `references/mockup-anatomy.md`, plays the role
`panel-anatomy.md`/`sheet-anatomy.md` play for their skills: it names the fixed
vocabulary — three format types, and the compositional layers within each (nav bar, hero
copy zone, CTA placement, device frame/chrome, supporting UI chrome, background/
environment) — that both the generation prompt and the compositional deconstruction are
built from. This is the direct implementation of the differentiation angle: modeling nav
bar / hero copy zone / CTA placement / visual hierarchy as separately-specified
compositional layers *before* generating, rather than one undifferentiated scene
description.

`references/composition-spec-format.md` reuses the exact schema `brand-identity-sheet`,
`character-model-sheet`, and `moodboard-generator` already share (`high_level_description`
+ `elements` array of `{type, bbox, desc}` / `{type: "text", bbox, text, desc}`),
unmodified — same rationale moodboard-generator's version states: the annotation problem
is identical regardless of what the sheet depicts.

`references/anti-slop-discipline.md` is UI-mockup-specific, not a copy of a sibling's file
— it targets a different failure mode (generic "fake AI dashboard" clichés: meaningless
charts, illegible micro-text, decorative UI chrome with no function implied, stock-SaaS
gradient washes) rather than brand-panel slop.

## Components

- **`SKILL.md`** — frontmatter written to trigger on: "hero image," "landing page hero,"
  "device mockup," "product mockup," "dashboard screenshot," "App Store screenshot,"
  "pitch deck visual" / "pitch deck screenshot." States explicitly, in the description
  itself (matching the pattern `moodboard-generator`'s frontmatter uses to distinguish
  itself from `brand-identity-sheet`), that this skill produces **finished images**, not
  functional HTML/CSS/React — so it doesn't get pulled into requests that actually want
  working code. Body documents the workflow below.
- **`references/mockup-anatomy.md`** — the three format types (landing-page hero,
  device-in-context mockup, dashboard screenshot), the compositional-layer vocabulary
  each draws from, and which layers apply to which format (e.g. a dashboard screenshot has
  no "device frame in an environment" layer; a device-in-context mockup does).
- **`references/composition-spec-format.md`** — the shared JSON schema, unmodified, plus a
  UI-mockup-specific worked example (a landing-page hero broken into nav-bar, hero-copy,
  CTA, and product-visual elements).
- **`references/anti-slop-discipline.md`** — the pre-generation gate for this skill's
  specific slop risks: generic "fake AI dashboard" visual clichés (meaningless bar charts
  with no legible axis labels, decorative sidebar icons implying navigation that goes
  nowhere, illegible or garbled micro-text used as texture rather than real copy,
  stock-SaaS purple-gradient hero backgrounds, an AI-glow product screen with no actual
  content on it).
- **`examples/`** — one worked example per format type once run for real (a hero shot, a
  device mockup, a dashboard screenshot), following the "actually render it" discipline
  the other visual-generation skills use — empty at spec time.
- **`evals/evals.json`** — 3 eval prompts in the existing schema (matching
  `brand-identity-sheet/evals/evals.json`'s format: `id`, `eval_name`, `prompt`,
  `expected_output`, `files`, `expectations`).

## Data flow / workflow

1. **Gather the brief** — format type (hero / device mockup / dashboard screenshot),
   real headline/subhead/CTA/nav copy (ask for it directly if missing rather than
   filling with placeholder text — see Assumptions), and brand truth if a `brand.md`
   exists for the project (palette, typography, adjectives — same pull-don't-reinvent
   pattern the sibling skills use). Ask which target placements the image needs to serve
   (landing page only, or also a social-square variant, or also an App Store portrait
   crop) — this determines whether step 6 (reframe) runs at all.
2. **Assign compositional layers** — using `references/mockup-anatomy.md`, decide which
   layers this format needs and what each one contains (e.g. for a hero shot: background/
   environment, device frame or browser chrome, nav bar with real nav-item text, hero copy
   zone with the real headline/subhead, CTA button with the real button label, product
   visual). This is the step that directly implements the "UI-specific compositional
   deconstruction" differentiation angle — done before drafting the prompt, not
   reverse-engineered only after.
3. **Draft the paragraph prompt** — one Ideogram-legible paragraph naming each assigned
   layer in reading order with its literal copy in quotes, so Ideogram renders real,
   specific on-screen text rather than approximating it. Follow the "name real things, not
   generic placeholders" discipline the sibling skills already use for palettes/materials,
   applied here to UI copy and product specifics.
4. **Run the anti-slop gate** — score the drafted prompt against
   `references/anti-slop-discipline.md`'s pre-generation table before calling
   `generate_image`. Anything scoring under 3 on any axis means revise now.
5. **Generate the base image** — `mcp__ideogram__generate_image` with the paragraph
   prompt. `style_type: "DESIGN"` for UI-chrome-dominant formats (dashboard screenshots,
   hero shots where the UI panel is the focal element), `style_type: "REALISTIC"` for
   photographic device-in-context shots. `rendering_speed: "QUALITY"` — every format in
   this skill's scope is legibility-sensitive (real nav/CTA/headline text has to render
   cleanly). Pick `aspect_ratio` for the primary target placement (e.g. `"16x9"` for a
   landing-page hero).
6. **Reframe for additional formats, if requested** — for each additional placement
   gathered in step 1, call `mcp__ideogram__reframe_image` with the base generation's
   `response_id` and the target `aspect_ratio`, rather than re-running `generate_image`
   from scratch. This is the multi-format-hero-variant capability named in the brief:
   one composition, several marketing surfaces, without re-drafting the prompt each time.
   See Error handling for the crop-risk caveat this step carries.
7. **Targeted fixes via `edit_image`, not full regeneration** — if a specific element
   comes back wrong (a garbled CTA label, a misspelled nav item, a headline that needs a
   word changed) after generation, use `mcp__ideogram__edit_image` with
   `image_response_ids` and a prompt describing only the fix, per that tool's strict
   "edit this image" contract. Do not fall back to a fresh `generate_image` or
   `remix_image` call to "get a similar but fixed" result — that silently discards a
   composition that was otherwise correct.
8. **Reverse-engineer into a compositional deconstruction** — produce the structured JSON
   breakdown per `references/composition-spec-format.md`, tagging each element with which
   compositional layer it is (Nav Bar, Hero Copy Zone, CTA, Device Frame/Chrome,
   Supporting UI Chrome, Background/Environment) so a later pass (a different format
   variant, a different skill) can reference "the CTA element from the hero shot" without
   re-describing it.
9. **Save what was made** — the paragraph prompt, the compositional deconstruction JSON,
   and the resulting image URL(s) (base + any reframed variants) to
   `04-projects/<project>/marketing-assets/` (or wherever the project already keeps
   marketing visuals — check for an existing folder first, matching the pattern the
   sibling skills use for `logo-explorations/`/`branding/`). Nothing generated here should
   live only in the conversation.

## Error handling

- **User actually wants working code / a functional prototype** ("make this clickable,"
  "I need real HTML I can deploy," "recreate this as an interactive Figma-like mockup") →
  say plainly this skill produces finished raster images, not editable/responsive code,
  and that an HTML/CSS-output tool is the right fit for that ask. Do not attempt to fake
  interactivity in an image or silently produce a static image when code was requested.
- **User wants a pixel-accurate recreation of a real, existing live product screen**
  ("recreate our exact production dashboard exactly as it looks today") → flag before
  generating that Ideogram's diffusion output is a stylized representation, not a
  pixel-accurate screenshot tool, and set that expectation explicitly rather than let the
  user assume fidelity the tool can't guarantee.
- **Garbled, misspelled, or illegible on-screen text after generation** (nav items,
  CTA label, headline) → this is the exact failure mode Ideogram's text-reliability
  strength is supposed to reduce but not eliminate; treat visibly broken UI text as a
  defect, attempt a targeted `edit_image` fix (step 7), and if that doesn't resolve it
  after a reasonable attempt, report the defect honestly to the user rather than shipping
  or describing broken text as acceptable.
- **`reframe_image` cropping into important copy** — going from a wide format (16:9 hero)
  to a much narrower/taller one (9:16 App Store portrait) risks clipping headline or CTA
  text that was composed near the original frame's edges. Before reframing to a
  significantly different aspect ratio, check whether the base composition's hero-copy/CTA
  elements sit safely away from the edges; if the layout is aspect-ratio-sensitive, tell
  the user a fresh `generate_image` composed for that target ratio will look better than a
  reframe, rather than reframing anyway and shipping clipped copy.
- **Missing real copy** (no headline/CTA/nav text supplied, no brand.md) → ask for it
  directly rather than inventing final marketing copy wholesale; a placeholder is
  acceptable only when the user explicitly says this is a rough concept exploration, not
  a deliverable — distinct from `moodboard-generator`'s default-fuzzy-tolerant posture,
  since this skill's default output is meant to be deck/App-Store-ready.
- **Ambiguous format type** (user says "make a mockup" without saying hero / device
  mockup / dashboard screenshot) → ask which of the three named format types they mean,
  using `references/mockup-anatomy.md`'s definitions, rather than guessing and producing
  the wrong compositional layout.

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts, one per format type:

1. "Generate a landing-page hero image for Technauts — headline 'Ship telemetry that
   trusts itself', subhead 'Real-time satellite health, without the guesswork', and a
   'Start free trial' button, with a browser-chrome mockup of the product behind it. I
   also need a square version for social." — hero generation + `reframe_image`
   multi-format path.
2. "I need a device mockup showing VoiceHive's app running on a laptop, sitting on a desk,
   for the pitch deck." — device-in-context mockup, `REALISTIC` style path, no reframe
   requested.
3. "Make an App Store screenshot showing the VoiceHive dashboard — sidebar nav, a couple
   of stat cards, a call-volume chart." — dashboard-screenshot format, verifying the
   anti-slop gate catches meaningless-chart/illegible-microtext failure modes before
   generation.

Assertions to be drafted following the skill-creator eval workflow once the skill exists,
matching the specificity level of `brand-identity-sheet/evals/evals.json`'s existing
assertions (skill actually calls a generation tool rather than only printing a prompt;
compositional deconstruction is bbox-structured, not a paragraph; real user-supplied copy
appears verbatim in the prompt rather than placeholder text; results are saved to a
project file rather than left only in the conversation).

## Out of scope (this spec)

- **Functional/interactive prototypes or HTML/CSS output** — the explicit competitive
  line drawn in the differentiation research; this skill declines and redirects rather
  than attempting it (see Error handling).
- **Pixel-perfect reproduction of an existing real product's live UI** — stylized
  representation only, never claimed as exact.
- **App-Store-compliance frameworks** (exact per-device bezel/dimension specs required by
  Apple/Google listing rules, localized screenshot sets, required-shot-count checklists) —
  a plausible later scope expansion, not built in v1. The skill produces a stylized
  App-Store-style screenshot; verifying it meets a specific store's current submission
  requirements is left to the user.
- **Icon-set or app-icon-grid generation** — already `brand-identity-sheet`'s app-icon
  panel; this skill doesn't duplicate it.
- **Proactive/automatic triggering from other skills** (e.g. `brand-identity-sheet`
  auto-offering to generate a hero shot once a brand system locks) — explicit-request
  trigger only, same stance `collections-management`'s spec takes toward the other five
  skills.
- **A fourth+ format type beyond the three named** (e.g. multi-screen onboarding flows,
  email-template mockups) — not built or named in `SKILL.md`'s trigger list in v1; the
  compositional-layer vocabulary in `references/mockup-anatomy.md` is written so it could
  extend to one later, but that extension isn't part of this spec.
