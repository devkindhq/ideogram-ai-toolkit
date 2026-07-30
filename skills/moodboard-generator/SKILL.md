---
name: moodboard-generator
description: Generates a single polished "moodboard" image — a 3×3 grid brand-exploration board (Color Palette, Typography, Logo Exploration, Iconography Style, Photography & Graphics, Material Samples, Abstract Patterns, Mood Imagery, Application Mockup) — and actually renders it via the ideogram MCP tools, not just drafts a prompt. Use whenever the user wants to see a brand's early visual direction *before* a logo or full identity system is locked, asks for a "moodboard," "mood board," "vibe board," "vision board," or "brand exploration board," wants to explore adjectives/palette/imagery together as a first pass, or references a moodboard image they want matched or extended. This is the tool for the earliest, fuzziest stage — before there's even a full brand.md — so use it even when the user only has a few loose words to go on ("something warm and analog," "no adjectives yet, just a name and an industry"). Distinct from `brand-identity-sheet` (which renders a *locked* brand system — UI card, app-icon grid, wordmark at scale — and needs real hex/typography values already decided) and from `logo-prompting` (a single mark, not a 9-panel board). Trigger this one first when the brand truth doesn't exist yet or is still fuzzy; trigger `brand-identity-sheet` once it does. Also use when the user wants to reverse-engineer an existing moodboard image into a structured, bbox-level description (a "compositional deconstruction") so it can be remixed or handed to another agent as a precise spec.
---

# Moodboard Generator

A moodboard is a single generated image that explores a brand's *direction* before
anything is locked — a 3×3 grid of panels (color, type, logo sketches, icon style,
photography, materials, pattern, mood imagery, one application mockup) that lets
someone react to a feeling before committing to a hex code or a typeface. It's the
format you reach for when the ask is "help me get oriented," not "give me the system"
(`brand-identity-sheet`) or "give me the mark" (`logo-prompting`).

This skill is a sibling to `brand-identity-sheet` and `logo-prompting` — same
two-artifact workflow, same Ideogram generation pattern, same reference-file structure
— but it sits *earlier* in the process than both of them. Where those two skills expect
a brand.md with real decisions already made, a moodboard is often the thing that
produces the first draft of those decisions. Tolerate fuzzy, partial, or three-word
inputs here in a way the other two skills shouldn't.

## The two artifacts this skill produces

1. **A generation prompt** — a tight descriptive paragraph, Ideogram-legible, that
   produces the 3×3 board in one shot.
2. **A compositional deconstruction** — a structured JSON breakdown of the resulting
   image (or of a reference image the user supplies) with a `high_level_description`
   and an `elements` array of `{type, bbox, desc}` / `{type: "text", bbox, text, desc}`
   entries. Same schema `brand-identity-sheet` and `character-model-sheet` use,
   unmodified — see `references/composition-spec-format.md`. This is what lets a later
   `brand-identity-sheet` or `logo-prompting` pass point back at "the logo-exploration
   panel from the moodboard" as a starting reference instead of re-describing it from
   scratch.

Always produce both. The paragraph prompt is what you feed the image model; the JSON is
what you save back to the vault so the board's structure survives past the single image.

## Workflow

### 1. Pull whatever brand truth exists — don't block on missing pieces

Read the project's `brand.md` if one exists. But unlike `brand-identity-sheet`, a
missing or half-written brand.md is the *normal* case here, not a blocker — a moodboard
is frequently the first artifact in a project, run specifically to help someone figure
out what the brand.md should say. Gather what you can, invent nothing, and leave the
rest as an open exploration:

- **Adjectives** — three words for the brand's personality. If the user hasn't given
  these, ask for them directly; this is the one input worth pausing for, since without
  it every panel defaults to generic. If they truly don't have words yet, pull from
  whatever context exists (industry, name, one sentence about who it's for) and propose
  three candidates back to them rather than inventing silently.
- **Color focus** — a hex, a named hue, or even just "warm neutrals" / "cool and
  clinical." Open palette is fine — a moodboard's whole point is exploring this.
- **Logo concept** — a symbol or motif the user envisions, if any. Often blank at this
  stage, and that's fine — panel 3 (Logo Exploration) can sketch *possibilities* rather
  than lock one.
- **Imagery & texture** — preferred visual/material references (a name-brand aesthetic,
  a material like "brushed steel" or "paper grain," a mood photographer's work).
- **Emotional impact** — one feeling the whole board should evoke (calm, urgent,
  playful, trustworthy).
- **Notes** — anything else that doesn't fit the above (a competitor to avoid looking
  like, an industry constraint, a reference image).

These six map directly onto the template variables in
`references/panel-anatomy.md`. If a project already has a `brand.md`, pull adjectives
from its Voice layer and color/imagery from its Visual layer rather than re-asking —
same as `brand-identity-sheet` does — but don't require it to exist first.

### 2. Fill the 9-panel template

The panel set is fixed for this skill (unlike `brand-identity-sheet`'s pick-and-choose
panel list) — the whole point of a moodboard is breadth across all nine categories in
one board, not a curated subset. See `references/panel-anatomy.md` for what each panel
is actually testing and what makes it earn its place rather than feel decorative:

1. Color Palette
2. Typography
3. Logo Exploration
4. Iconography Style
5. Photography & Graphics
6. Material Samples
7. Abstract Patterns
8. Mood Imagery
9. Application Mockup

### 3. Write the paragraph prompt

Structure, in order — same discipline as the sibling skills, applied to a 3×3
exploration board instead of a locked system:

1. Frame it as a **brand moodboard / visual exploration board**, arranged as a 3×3
   grid — naming the grid explicitly and the word "exploration" is what keeps this
   reading as an intentional research board rather than a single confused collage, the
   opposite problem `brand-identity-sheet` solves by avoiding the word "moodboard"
   entirely (that skill wants the *locked-system* look; this skill wants the
   *exploration* look, so lean into it here).
2. Name all nine panels in reading order (left-to-right, top-to-bottom), each with a
   short label so Ideogram renders them as nine distinct cards, not nine blended scenes.
3. Adjectives: state the three words plainly — they're the thread that should be
   legible across every panel, not just the mood-imagery one.
4. Color: whatever's known — exact hex if locked, a named hue or temperature if still
   open ("warm terracotta and cream, still exploring the exact palette").
5. Logo concept: describe it as a *sketch* or *exploration*, not a finished mark — "a
   loose sketched exploration suggesting X," never "the final logo is X." This is the
   one place this skill deliberately contradicts `logo-prompting`'s "abstract, don't
   depict literally" rule less strictly — a moodboard's logo panel is allowed to be
   more literal and rougher, because its job is to spark a direction, not ship a mark.
6. Imagery & texture, material samples, abstract patterns: pull straight from whatever
   references the user gave — name real materials and real reference aesthetics rather
   than "some textures."
7. Emotional impact: one sentence naming the feeling the whole board should leave
   someone with.
8. Negative space / anti-slop: see `references/anti-slop-discipline.md` — a moodboard
   with nine panels has nine chances to drift into generic AI-image territory, more
   than either sibling skill.
9. Background/composition note: clean white background, thin grid lines separating the
   nine panels, each panel labeled with a small caption — this is what keeps it reading
   as a structured board rather than a Pinterest dump, even though "moodboard" energy is
   otherwise welcome here.

Keep it one paragraph. If the user wants to compare genuinely different directions
(not just a palette swap), write separate prompts — same "structural variety over
palette-swaps" rule as `logo-prompting` and `brand-identity-sheet`: two boards that only
swap which hex fills the same nine slots aren't two directions.

### 3.5. Run the anti-slop gate before generating

Before calling `generate_image`, score the drafted prompt against the pre-generation
gate table in `references/anti-slop-discipline.md`. Anything scoring under 3 on any axis
means revise the prompt now — a nine-panel image is expensive to regenerate, and a
loose "vibe board" that pulls Ideogram toward generic Pinterest-collage energy is the
single most common failure mode this skill exists to prevent.

### 4. Generate it — don't just hand back a prompt

Call `mcp__ideogram__generate_image` with the paragraph prompt. Use `style_type:
"DESIGN"` and `aspect_ratio: "1x1"` — a 3×3 grid reads cleanest as a square. Use
`rendering_speed: "QUALITY"` for the same reason the sibling skills do: nine
text-bearing panels at once is a composition-heavy, legibility-sensitive render.

If the user supplies a reference moodboard they like and wants variations, use
`mcp__ideogram__remix_image` instead of a fresh `generate_image` call — `image_weight`
around 30-50 for "same energy, different brand," since moodboards are meant to be
loosely interpreted rather than precisely reproduced (contrast with
`brand-identity-sheet`'s tighter 40-60 remix range for a locked system).

### 5. Reverse-engineer into a compositional deconstruction

Once the board is generated (or if the user hands you a reference moodboard to work
from), produce the structured JSON breakdown described in
`references/composition-spec-format.md` — same process as the sibling skills:

- **From a generated image**: write the JSON directly from the panel layout you
  specified in step 3.
- **From a reference image**: call `mcp__ideogram__describe_image` first, then
  structure that description into the schema, using coarse-but-honest bbox estimates
  where you can't confidently place exact coordinates.

### 6. Save what you made

Write the paragraph prompt, the JSON compositional deconstruction, and the resulting
image URL(s) to `04-projects/<project>/logo-explorations/moodboard-<label>.md` (or
wherever that project keeps its branding work — check for an existing
`logo-explorations/` or `branding/` folder first and match it). Follow the "No Context
Lost" rule: nothing generated in this skill should live only in the conversation.

If the user reacts strongly to a specific panel (loves the palette, hates the logo
sketch), capture that reaction in the saved file explicitly — it's the input the next
`brand-identity-sheet` or `logo-prompting` pass will need, and a moodboard's whole
purpose is feeding that decision forward.

## Reference files

- `references/panel-anatomy.md` — the nine named panels (Color Palette, Typography,
  Logo Exploration, Iconography Style, Photography & Graphics, Material Samples,
  Abstract Patterns, Mood Imagery, Application Mockup), what each one is testing, and
  what makes it earn its place rather than feel decorative.
- `references/composition-spec-format.md` — the JSON schema for the compositional
  deconstruction (shared, unmodified, with `brand-identity-sheet`'s and
  `character-model-sheet`'s schema).
- `references/anti-slop-discipline.md` — the moodboard-specific anti-slop guardrails
  (nine panels of surface area, the "exploration vs. collage" line, the pre-generation
  gate) adapted from `brand-identity-sheet`'s version.
- `examples/` — worked moodboard jobs go here as they happen; empty for now.
