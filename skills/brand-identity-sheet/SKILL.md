---
name: brand-identity-sheet
description: Generates a single polished "brand identity sheet" image — a Dribbble-style board showing a UI card with the wordmark, a grid of textured app icons, pill-shaped buttons, a giant central wordmark treatment, and a secondary lockup with a brand glyph — and actually renders it via the ideogram MCP tools, not just drafts a prompt. Use whenever the user wants to see a brand's whole visual system in one image (not a single logo), asks for a "moodboard," "identity sheet," "brand board," or "style board," wants app-icon / UI-component exploration alongside the logo, or references a reference image/moodboard they want matched or extended. Distinct from logo-prompting's single-mark prompts — trigger this when the ask is bigger than one logo and wants the full palette/type/icon/UI system shown together. Also use when the user wants to reverse-engineer an existing brand image into a structured, bbox-level description (a "compositional deconstruction") so it can be remixed or handed to another agent as a precise spec.
---

# Brand Identity Sheet

A brand identity sheet is a single generated image that shows a brand's whole visual
system at once — not one mark, but a UI card carrying the wordmark, a grid of
textured/gradient app icons, pill-shaped buttons in the brand palette, a headline-scale
wordmark treatment, and a secondary lockup with a small brand glyph. It's the format
you reach for when the ask is "help me see the whole system," not "give me a logo."

This is a sibling to `logo-prompting`, not a replacement. Use `logo-prompting` when the
deliverable is one mark. Use this skill when the deliverable is a board that puts the
mark in context — alongside icons, buttons, and type at scale — the way a designer's
Dribbble-style brand-sheet shot looks.

## The two artifacts this skill produces

1. **A generation prompt** — a tight descriptive paragraph, Ideogram-legible, that
   produces the sheet in one shot.
2. **A compositional deconstruction** — a structured JSON breakdown of the resulting
   image (or of a reference image the user supplies) with a `high_level_description`
   and an `elements` array of `{type, bbox, desc}` / `{type: "text", bbox, text, desc}`
   entries. This is not decoration — it's what makes the sheet *reusable*: another
   agent (or a future you) can read the bbox spec and know exactly which panel is the
   wordmark card, which is the icon grid, which is the CTA button, without having to
   re-look at the image. See `references/composition-spec-format.md` for the full
   schema and a worked example.

Always produce both. The paragraph prompt is what you feed the image model; the
compositional deconstruction is what you save back to the vault so the sheet's
structure survives past the single image (feeding hallmark/website work later, the
same way `technauts-logo-prompting-space`'s output fed the site rebrand).

## Workflow

### 1. Pull the brand truth

Read the project's `brand.md` (Strategy + Voice + Visual layers) or `design.md` if the
project has moved past brand.md into a locked design system. You need, specifically:

- **Colors** — exact hex/OKLCH values and their *usage rule* (which is the dominant
  paper/ink, which is the one accent, which is reserved for a single state).
- **Typography** — display face, weight, case (roman vs. italic).
- **Style keywords + reference brands** — these tell you the texture/material
  vocabulary (e.g. "metallic," "carbon-fiber mesh," "brushed aluminum" for a technical
  brand; "warm paper grain" for an analog one).
- **The brand's own avoid-list, if it has one** — many brand docs carry an explicit
  "anti-slop" / "avoid" / "don't do this" section naming specific visual clichés this
  brand must never look like. Pull the exact nouns from that list now, before you
  start drafting — you'll need to quote them verbatim in the prompt later (step 3),
  and a guardrail you only half-remember from this read is a guardrail that gets
  paraphrased into something too vague for the image model to act on.

If there's no brand.md/design.md yet, ask for the equivalent before drafting — a sheet
built on invented colors is decoration, not identity, and will need to be redone the
moment a real brand.md exists.

### 2. Choose the panel set

A full sheet doesn't need every panel from `references/sheet-anatomy.md` every time —
pick the ones that earn their place for this brand. The canonical set, in rough
reading order:

- **Wordmark card** — a white or paper-toned UI card with a small brand-name label at
  top, the wordmark large in the middle, and a tiny supporting line (edition/price/tag)
  at the bottom. This is the panel that anchors "what does the logo look like on a
  clean surface."
- **App-icon grid** — 4-8 square (or one tall) rounded-corner icons, each a distinct
  texture/motif pulled from the brand's material vocabulary: a gradient/glow icon, a
  mesh/weave texture icon, a metallic or material-sample icon, one mechanical/utility
  icon (gear, tool) if the brand has a technical register, one character/mascot icon if
  the brand has one. Each icon is a *material sample*, not a redundant logo restate —
  the point is range, not repetition.
- **Pill buttons** — 2-3 stacked pill-shaped buttons cycling through the primary,
  accent, and neutral/secondary colors, showing how CTA hierarchy reads in the palette.
- **Giant central wordmark** — the same wordmark at headline/hero scale, tightly
  spaced, anchoring the composition the way a type specimen sheet would.
- **Secondary lockup** — wordmark + a small geometric brand glyph (a star, a dot, a
  simple mark distinct from the primary logo) — this is the "what goes in a footer or
  a stamp" variant, not a duplicate of the main logo.
- **Character/mascot panel** (only if the brand has one) — a single stylized character
  icon, rendered in the same material vocabulary as the rest of the sheet.

Naming the panels explicitly in the prompt (not just "a moodboard of stuff") is what
keeps Ideogram from defaulting to a generic swatch-and-Pinterest-board layout.

### 3. Write the paragraph prompt

Structure, in order — same discipline as `logo-prompting`, applied to a whole sheet
instead of one mark:

1. Frame it as a **brand identity guide / UI style sheet layout**, not a "moodboard" —
   the word "moodboard" alone nudges the model toward loose Pinterest-collage energy;
   "identity sheet" / "style guide" nudges toward the structured grid you actually want.
2. Name the exact panel set from step 2, in the order they should read.
3. Color: exact hex/OKLCH, stated as the dominant paper + one accent + one secondary —
   never as an unweighted list.
4. Typography: face + weight + case for the wordmark specifically, since it appears at
   three different scales across the sheet (small card label, giant hero, secondary
   lockup) and needs to read as the same face at all three.
5. Material/texture vocabulary for the icon grid, pulled straight from the brand's
   style keywords — this is the part that most differentiates one brand's sheet from
   another's, so don't leave it generic ("some icons") — name the actual materials.
6. Negative space / anti-slop: this is not a generic "keep it tasteful" line — see
   `references/anti-slop-discipline.md` for the full universal-ban list (no
   neural-network nodes, no glowing-orb-as-hero-icon, no headset/padlock/lightbulb
   clichés, no purple-to-pink gradient wash across the whole sheet, no stock-photo
   textures, no invented product screenshots unless the brand has a real product to
   show) and **name the brand doc's own avoid-list items verbatim** if it has one —
   quoting the brand's specific nouns, not paraphrasing them into your own words, is
   what makes this clause actually bind. A "glow" icon is still a legitimate single
   panel (per `sheet-anatomy.md`); a gradient bleeding across the whole composition or
   background is not — see the anti-slop file's "glow containment" check.
7. Background/composition note: state it's a clean, structured grid layout (blueprint
   grid lines, light or dark ground per the brand) — this is what keeps the panels
   legible as distinct cards rather than bleeding into a collage.

Keep it one paragraph. If you want to explore two structurally different sheets (say,
a light-paper version and a dark-mode version), write two separate prompts — same
"structural variety over palette-swaps" rule as `logo-prompting`: two sheets that only
differ by which hex is plugged into the same slots aren't two directions.

### 3.5. Run the anti-slop gate before generating

Before calling `generate_image`, score the drafted prompt against the pre-generation
gate table in `references/anti-slop-discipline.md` (brand-specificity, guardrail
transcription, universal-ban sweep, glow containment, restraint). Anything scoring
under 3 on any axis means revise the prompt now — regenerating a shipped image is
expensive; revising a paragraph before you generate is not. This step exists because
palette and typography accuracy (step 1) and slop-freedom (this step) are different
failure modes — a prompt can get the Ascent/Cobalt/whatever hex values exactly right
while still defaulting to a generic AI-icon vocabulary, and only checking for the
second catches it.

### 4. Generate it — don't just hand back a prompt

Call `mcp__ideogram__generate_image` with the paragraph prompt. Sheets like this
benefit from `style_type: "DESIGN"` and a squarish-to-portrait aspect ratio
(`"1x1"` or `"4x5"`) since they're laid out as a grid of cards, not a single wide scene.
Use `rendering_speed: "QUALITY"` — this is a composition-heavy image with a lot of
small text-bearing panels, and QUALITY holds up legibility better than FAST/FLASH at
that density.

If the user supplies a reference image they like (their own upload, or a prior
generation) and wants variations, use `mcp__ideogram__remix_image` against it instead
of a fresh `generate_image` call — pass `image_weight` around 40-60 for "same
structure, different brand" remixes, higher for closer variations.

### 5. Reverse-engineer into a compositional deconstruction

Once the sheet is generated (or if the user hands you an existing reference image to
work from), produce the structured JSON breakdown described in
`references/composition-spec-format.md`. Two ways to get there:

- **From a generated image**: you already know the exact panel layout you asked for in
  step 3 — write the JSON directly from that, since you control the composition.
- **From a reference image the user supplies**: call `mcp__ideogram__describe_image`
  first to get the model's own read of the image, then structure that description into
  the `high_level_description` + `elements` schema — add bbox estimates based on visual
  inspection of the image, not fabricated precision. If you can't confidently place a
  bbox, use a coarse quadrant estimate and say so rather than inventing exact pixels.

Save both artifacts (prompt + JSON) to the project's brand folder — see step 6.

### 6. Save what you made

Write the paragraph prompt, the JSON compositional deconstruction, and the resulting
image URL(s) to `04-projects/<project>/logo-explorations/identity-sheet-<label>.md` (or
wherever that project keeps its branding work — check for an existing
`logo-explorations/` or `branding/` folder first and match it). Follow the "No Context
Lost" rule: nothing generated in this skill should live only in the conversation.

If a panel or a color usage turns out to be a pattern worth reusing across brands (not
just this one project's specific palette), add it to
`references/sheet-anatomy.md` as a named panel type, the same way `logo-prompting`
promotes reusable techniques into its own reference files.

## Reference files

- `references/composition-spec-format.md` — the full JSON schema for the compositional
  deconstruction (`high_level_description`, `elements[]` with `type`/`bbox`/`desc`,
  text elements with `text`), plus the worked Technauts example this skill was seeded
  from.
- `references/sheet-anatomy.md` — the named panel types (wordmark card, app-icon grid,
  pill buttons, giant wordmark, secondary lockup, character panel) with what each one
  is for and when to include or drop it.
- `references/anti-slop-discipline.md` — the universal AI-image-cliché bans (glowing
  orbs, neural nodes, gradient washes, stock-photo textures, and the rest), why a
  brand's own avoid-list has to be quoted verbatim rather than paraphrased, and the
  pre-generation gate to run before every `generate_image` call. Read this every time,
  not just when a brand doc happens to list its own guardrails — the universal bans
  apply regardless.
- `examples/technauts-identity-sheet.md` — the real paragraph prompt + full JSON
  breakdown this skill was built from, as a worked reference for format and level of
  detail.
