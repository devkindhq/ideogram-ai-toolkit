---
name: marketing-ui-mockup-hero-generator
description: Generates finished raster marketing visuals — landing-page hero shots, device-in-context product mockups, and dashboard screenshots — and actually renders them via the ideogram MCP tools, not just drafts a prompt. Use whenever the user asks for a "hero image," "landing page hero," "device mockup," "product mockup," "dashboard screenshot," "App Store screenshot," "pitch deck visual," or "pitch deck screenshot." This skill produces finished images, not functional HTML/CSS/React — a request for working code, a live component, or a responsive layout should go to a code-output tool instead, never get pulled into this skill just because it also involves UI. Sibling to `brand-identity-sheet`, `character-model-sheet`, and `moodboard-generator` structurally (same prompt-composition pattern, same paragraph-prompt + compositional-deconstruction pair of artifacts), but targets marketing-visual deliverables — a single hero shot, a device-in-context product photo, a dashboard screen — rather than a whole brand system or a character sheet.
---

# Marketing UI Mockup / Hero Generator

This skill produces one of three fixed format types, each a finished raster image
rendered via Ideogram, not a mockup described in prose or built as working code:

- **Landing-page hero shot** — a full-bleed marketing hero with headline, subheadline,
  and CTA button copy baked in via Ideogram's text rendering, typically showing the
  product in a browser/device frame or as a standalone visual. The shot a visitor sees
  before scrolling.
- **Device-in-context product mockup** — a product's UI shown inside a physical device
  shell (laptop, phone, tablet, browser chrome) placed in an environmental or lifestyle
  context (desk surface, hand, studio backdrop). The format for pitch-deck slides and
  "see it in the real world" shots.
- **Dashboard screenshot** — a single stylized app/dashboard screen (nav, cards,
  charts, sidebar) rendered to look polished and "real" for App Store listings, pitch
  decks, or marketing pages — not a literal render of any actual live codebase.

See `references/mockup-anatomy.md` for each format's full compositional-layer
breakdown; read it before drafting a prompt (workflow step 2 below), not
reverse-engineered only after generation.

## The two artifacts this skill produces

1. **A generation prompt** — a paragraph, Ideogram-legible, naming every assigned
   compositional layer in reading order with literal copy in quotes.
2. **A compositional deconstruction** — the structured JSON breakdown per
   `references/composition-spec-format.md`, tagging each element with which
   compositional layer it belongs to so a later pass can reference "the CTA element
   from the hero shot" without re-describing it.

Always produce both. The paragraph prompt is what you feed the image model; the
compositional deconstruction is what you save back to the vault so the mockup's
structure survives past the single image.

This skill is a sibling to `brand-identity-sheet`, `character-model-sheet`, and
`moodboard-generator` structurally — same paragraph-prompt-plus-JSON-deconstruction
pattern — but targets marketing-visual deliverables (hero shots, device mockups,
dashboard screenshots) rather than brand systems or characters.

## Workflow

### 1. Gather the brief

Ask for, or pull from context:

- **Format type** — hero / device mockup / dashboard screenshot. If the user's
  phrasing is ambiguous ("make a mockup"), use `references/mockup-anatomy.md`'s
  definitions to clarify which of the three they mean rather than guessing (see Error
  handling below).
- **Real headline/subhead/CTA/nav copy** — ask directly if it's missing rather than
  filling with placeholder text.
- **Brand truth** from the project's `brand.md`/`design.md` if one exists — palette,
  typography, adjectives. Same pull-don't-reinvent pattern `brand-identity-sheet`'s
  step 1 uses.
- **Target placements** — landing page only, or also a social-square variant, or also
  an App Store portrait crop. This determines whether workflow step 6 (reframe) runs.

### 2. Assign compositional layers

Using `references/mockup-anatomy.md`'s per-format layer table, decide which layers
this format needs and what each one contains. This is the step that directly
implements the UI-specific compositional-deconstruction differentiation angle — done
before drafting the prompt, not reverse-engineered only after.

### 3. Draft the paragraph prompt

One Ideogram-legible paragraph naming each assigned layer in reading order with its
literal copy in quotes. Apply the same "name real things, not generic placeholders"
discipline `brand-identity-sheet` step 3 uses for its material vocabulary, here applied
to UI copy and product specifics: a real nav-item label, not "Home / About / Contact";
a real chart metric, not "a chart."

### 4. Run the anti-slop gate

Score the drafted prompt against `references/anti-slop-discipline.md`'s
pre-generation table before calling `generate_image`. Anything scoring under 3 on any
axis means revise now, not after generating.

### 5. Generate the base image

Call `mcp__ideogram__generate_image` with the paragraph prompt.

- **`style_type`**: `"DESIGN"` for UI-chrome-dominant formats (dashboard screenshots,
  hero shots where the UI panel is the focal element), `"REALISTIC"` for photographic
  device-in-context shots. This mapping is documented guidance to sanity-check against
  the actual result, not a verified Ideogram model guarantee.
- **`rendering_speed`**: `"QUALITY"` always.
- **`aspect_ratio`**: pick for the primary target placement — e.g. `"16x9"` for a
  landing-page hero, `"1x1"` for a dashboard screenshot destined for a square social
  post, `"9x16"` for an App Store portrait screenshot.

### 6. Reframe for additional formats, if requested

For each additional placement gathered in step 1, call `mcp__ideogram__reframe_image`
with the base generation's `response_id` and the target `aspect_ratio`, rather than
re-running `generate_image` from scratch.

Crop-risk caveat: before reframing to a significantly different aspect ratio, check
whether the base composition's Hero Copy Zone/CTA elements sit safely away from the
frame edges (from the compositional deconstruction produced in step 8, or by
inspection if step 8 hasn't run yet for this composition). If the layout is
aspect-ratio-sensitive, tell the user a fresh `generate_image` composed for that target
ratio will look better than a reframe.

### 7. Targeted fixes via edit_image, not full regeneration

If a specific element comes back wrong after generation — a garbled CTA label, a
misspelled nav item, a headline needing a word changed — call
`mcp__ideogram__edit_image` with `image_response_ids` and a prompt describing only the
fix. Do not fall back to a fresh `generate_image` or `remix_image` call to "get a
similar but fixed" result — that silently discards a composition that was otherwise
correct.

### 8. Reverse-engineer into a compositional deconstruction

Produce the structured JSON breakdown per `references/composition-spec-format.md`,
tagging each element with which compositional layer it is (Nav Bar, Hero Copy Zone,
CTA, Device Frame/Chrome, Supporting UI Chrome, Background/Environment) so a later
pass can reference "the CTA element from the hero shot" without re-describing it.

### 9. Save what was made

Write the paragraph prompt, the compositional deconstruction JSON, and the resulting
image URL(s) (base + any reframed variants) to
`04-projects/<project>/marketing-assets/` (or wherever the project already keeps
marketing visuals — check for an existing folder first, matching the pattern
`brand-identity-sheet`/`character-model-sheet` use for `logo-explorations/`/
`branding/`). Nothing generated here should live only in the conversation.

## Error handling

- **User actually wants working code / a functional prototype** — say plainly this
  skill produces finished raster images, not editable/responsive code, and redirect to
  an HTML/CSS-output tool. Never fake interactivity in a static image, never silently
  produce an image when code was requested.
- **User wants a pixel-accurate recreation of a real, existing live product screen** —
  flag before generating that Ideogram's diffusion output is a stylized
  representation, not a pixel-accurate screenshot tool; set that expectation
  explicitly.
- **Garbled, misspelled, or illegible on-screen text after generation** — treat as a
  defect, attempt a targeted `edit_image` fix (workflow step 7), and if that doesn't
  resolve it after a reasonable attempt, report the defect honestly rather than
  shipping or describing broken text as acceptable.
- **`reframe_image` cropping into important copy** — per workflow step 6's caveat,
  check hero-copy/CTA placement safety before reframing to a significantly different
  aspect ratio; prefer a fresh `generate_image` over a reframe that would clip copy.
- **Missing real copy** (no headline/CTA/nav text supplied, no brand.md) — ask for it
  directly rather than inventing final marketing copy wholesale; a placeholder is
  acceptable only when the user explicitly says this is a rough concept exploration,
  not a deliverable.
- **Ambiguous format type** (user says "make a mockup" without specifying which of the
  three) — ask which of the three named format types they mean, using
  `references/mockup-anatomy.md`'s definitions, rather than guessing and producing the
  wrong compositional layout.

## Reference files

- `references/mockup-anatomy.md` — the three format types (landing-page hero,
  device-in-context mockup, dashboard screenshot) and the compositional layers each is
  built from, with a per-format table stating which layers apply.
- `references/composition-spec-format.md` — the full JSON schema for the
  compositional deconstruction (`high_level_description`, `elements[]` with
  `type`/`bbox`/`desc`, text elements with `text`), the layer-tagging convention, and a
  worked Technauts landing-hero example.
- `references/anti-slop-discipline.md` — the UI-mockup-specific failure modes
  (meaningless charts, decorative dead-end sidebars, illegible micro-text-as-texture,
  stock-SaaS gradient washes, AI-glow screens with no real content), the real-copy
  requirement gate, and the pre-generation gate to run before every `generate_image`
  call.
