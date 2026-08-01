---
name: character-model-sheet
description: Generates a single polished "character model sheet" image — a multi-panel character design board showing a full-body turnaround (front / 3-4 / side / back), a head & detail close-up sheet (face, hair/headwear, hands/props), a color palette, an overview text block (name, role, personality, wardrobe), and an optional accessories/interior technical diagram — and actually renders it via the ideogram MCP tools, not just drafts a prompt. Use whenever the user wants a mascot, game character, animation character, or brand character rendered as a full reference/turnaround sheet (not a single portrait), asks for a "character sheet," "turnaround sheet," "character reference sheet," or "model sheet," wants multiple angles/views of the same character shown together, or references a character-sheet image they want matched or extended. Distinct from logo-prompting (single mark) and brand-identity-sheet (brand system of icons/buttons/wordmark) — trigger this specifically when the subject is a character (a mascot, a person, a creature, a robot) that needs to be shown from multiple angles with consistent design. Also use when the user wants to reverse-engineer an existing character-sheet image into a structured, bbox-level description (a "compositional deconstruction") so it can be remixed or handed to another agent as a precise spec.
---

# Character Model Sheet

A character model sheet is a single generated image that shows a character's complete
design at once — not one pose, but a full-body turnaround (front, 3/4, side, back), a
head & detail close-up sheet, a color palette, and a short overview block naming the
character, its role, and its personality. It's the format riggers, illustrators, and
game/animation studios use to lock a character's design before it's animated, modeled,
or drawn by a second artist — the point is *consistency across angles*, not a single
nice pose.

This is a sibling to `brand-identity-sheet` (same two-artifact workflow, same Ideogram
generation pattern) but for characters instead of brand systems, and a sibling to
`logo-prompting`'s "single mark" discipline scaled up to "single character, many views."

## The two artifacts this skill produces

1. **A generation prompt** — a tight descriptive paragraph, Ideogram-legible, that
   produces the sheet in one shot.
2. **A compositional deconstruction** — a structured JSON breakdown of the resulting
   image (or of a reference image the user supplies) with a `high_level_description`
   and an `elements` array of `{type, bbox, desc}` / `{type: "text", bbox, text, desc}`
   entries. This is what makes the sheet *reusable*: another agent (an animator, a
   modeler, a second illustrator) can read the bbox spec and know exactly which panel is
   the front view, which is the face close-up, which is the color swatch bar, without
   re-looking at the image. See `references/composition-spec-format.md` for the full
   schema — it's the same schema `brand-identity-sheet` uses, unmodified, because the
   annotation problem (precisely locating panels/text in a multi-panel design sheet) is
   identical regardless of subject.

Always produce both. The paragraph prompt is what you feed the image model; the
compositional deconstruction is what you save back to the vault so the sheet's
structure survives past the single image (feeding animation, modeling, or a second
illustrator's work later).

## Workflow

### 1. Pull the character truth

Before drafting, gather (ask the user if any are missing rather than inventing them):

- **Name, alias/nickname, role/function, archetype** — the identity block that anchors
  the whole sheet (e.g. "Ding-Bot / Chime-E / Communication AI Assistant / Cheerful
  Guide").
- **Personality & behavior** — 2-4 traits plus one "core theme" line and one behavior
  note. This shapes expression, pose energy, and how "busy" the detail panels should
  feel.
- **Silhouette & body type** — the character's base shape (bell-shaped, humanoid,
  quadruped, boxy robot) since this is what has to read identically across all four
  turnaround views.
- **Wardrobe / accessories** — anything worn or carried (headphones, a hat, a tool, a
  cape) — these need their own detail panel if they're a defining feature.
- **Color palette** — primary, secondary, and accent colors, stated with role, not just
  a flat list (e.g. "primary: orange bell body; secondary: light-blue headphones and
  internal details; accent: gold/brass base metal, dark blue, white").
- **Material/render style** — 3D-rendered toy/vinyl-figure look, flat vector illustration,
  painterly concept-art, anime cel-shaded, etc. This has to be named explicitly and held
  constant across every panel, or Ideogram will drift style between the turnaround and
  the detail sheet.

If there's no existing character brief for this project, ask for the equivalent before
drafting — a sheet built on invented traits is decoration, not a locked design, and will
need to be redone the moment a real character brief exists.

### 2. Choose the panel set

Not every sheet needs every panel from `references/sheet-anatomy.md` — pick the ones
that earn their place for this character. The canonical set, in rough reading order:

- **Header/overview block** — small text panels naming the character, alias, role,
  archetype, personality/behavior, wardrobe, and color palette (with swatch bars). This
  is the "spec sheet" framing that keeps the whole board from reading as fan art.
- **Full body turnaround** — front, 3/4, side, and back views of the character in the
  same neutral pose, each labeled, ideally with a height-scale ruler running alongside
  (inches and/or cm) to lock proportions.
- **Head & detail sheet** — a close-up front face, a close-up of any signature
  head-worn/hair detail (headphones, hat, ears), and a close-up of a signature
  hand/gesture or held prop.
- **Accessories & interior/technical diagram** — only if the character has a mechanical,
  robotic, or gadget-driven identity: an exploded/cutaway diagram of internal components
  or accessories with labeled callout lines. Skip entirely for organic/human characters
  unless they carry a notable prop worth diagramming.
- **Color palette bar** — primary/secondary/accent swatches, usually docked near the
  header block, each labeled.

Naming the panels explicitly in the prompt (not just "character sheet with some views")
is what keeps Ideogram from defaulting to a single illustrated portrait or a loose
sketch-dump instead of the structured grid you want.

### 3. Write the paragraph prompt

Structure, in order:

1. Frame it as a **character model sheet / character turnaround sheet / character
   reference sheet for [name]**, not "an illustration of" or "art of" — the framing word
   is what nudges Ideogram toward the structured multi-view grid instead of a single
   hero pose.
2. Name the exact panel set from step 2, in the order they should read (turnaround
   first, then head/detail, then palette/overview, then technical diagram if included).
3. Describe the character itself once, in full (silhouette, proportions, face,
   wardrobe/accessories, colors with hex or named-role if known) — this single
   description is what has to hold consistent across every panel, so front-load it
   rather than re-describing the character differently per panel.
4. State the render/material style explicitly (e.g. "3D-rendered vinyl-toy style with
   soft studio lighting" or "flat cel-shaded vector illustration") and repeat that it
   must be held identical across every panel — this is the single most common failure
   mode (style drifting between the turnaround and the detail close-ups).
5. Color: exact hex/named values stated with role (primary body, secondary
   wardrobe/accent, trim/metal color) — never as an unweighted list.
6. Background/composition note: clean white or light-gray background with thin grid
   lines and thin black panel borders, each panel labeled with a small caption
   (FRONT / 3-4 VIEW / SIDE / BACK / FRONT FACE / etc.) — this is what keeps the sheet
   reading as a reference document, not a moodboard or collage.
7. Negative space / anti-slop: see `references/anti-slop-discipline.md` — no
   inconsistent proportions/colors between panels, no panels that drift off-model, no
   generic stock-mascot clichés unless the brief specifically calls for them, no
   invented text/labels beyond what's specified.

Keep it one paragraph (it can run longer than a logo prompt — a character sheet is
carrying more distinct panels — but stay a single continuous paragraph rather than a
bulleted prompt). If you want to explore genuinely different design directions (not
just palette swaps), write separate prompts per direction — same "structural variety
over palette-swaps" rule as `logo-prompting` and `brand-identity-sheet`.

### 3.5. Run the anti-slop / on-model gate before generating

Before calling `generate_image`, score the drafted prompt against the pre-generation
gate table in `references/anti-slop-discipline.md` (character-specificity, style-lock
transcription, off-model risk sweep, panel-labeling completeness, restraint). Anything
scoring under 3 on any axis means revise the prompt now — regenerating a shipped image
is expensive; revising a paragraph before you generate is not. On-model consistency
(step 4 above) is the character-sheet-specific failure mode that brand-identity-sheet
doesn't have to worry about as acutely, since a character's silhouette repeating
correctly across four different angles is harder for an image model to hold than a
wordmark repeating across two scales.

### 4. Generate it — don't just hand back a prompt

Call `mcp__ideogram__generate_image` with the paragraph prompt. Character model sheets
benefit from `style_type: "DESIGN"` or `"GENERAL"` (test both if the character has a
painterly/illustrative brief rather than a clean product-design one) and a
landscape-to-square aspect ratio (`"4x3"`, `"3x2"`, or `"1x1"`) since the canonical
layout (turnaround block + head/detail block side by side) reads wider than tall. Use
`rendering_speed: "QUALITY"` — this is a composition-heavy image with many small
text-bearing panels and a proportion-lock requirement across views, and QUALITY holds up
both legibility and consistency better than FAST/FLASH at that density.

If the user supplies a reference image they like (their own upload, or a prior
generation) and wants variations or a new pose/detail panel added, use
`mcp__ideogram__remix_image` against it instead of a fresh `generate_image` call — pass
`image_weight` around 60-80 for "keep this exact character, adjust the sheet layout,"
lower for looser reinterpretations.

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
  See `examples/ding-bot-character-sheet.md` for a full worked example at this level of
  detail (a real reverse-engineered spec, not a generated one, showing the target
  granularity: every view, every label, every scale marking captured as its own
  element).

Save both artifacts (prompt + JSON) to the project's character/branding folder — see
step 6.

### 6. Save what you made

Write the paragraph prompt, the JSON compositional deconstruction, and the resulting
image URL(s) to `04-projects/<project>/logo-explorations/character-sheet-<label>.md` (or
wherever that project keeps its branding/character work — check for an existing
`logo-explorations/`, `branding/`, or `characters/` folder first and match it). Follow
the "No Context Lost" rule: nothing generated in this skill should live only in the
conversation.

If a panel or a technique turns out to be a pattern worth reusing across characters (not
just this one character's specific execution of it), add it to
`references/sheet-anatomy.md` as a named panel type, the same way `brand-identity-sheet`
and `logo-prompting` grow their reference files from real jobs.

## Reference files

- `references/composition-spec-format.md` — the JSON schema for the compositional
  deconstruction (shared, unmodified, with `brand-identity-sheet`'s schema —
  `high_level_description`, `elements[]` with `type`/`bbox`/`desc`, text elements with
  `text`).
- `references/sheet-anatomy.md` — the named panel types (header/overview block, full
  body turnaround, head & detail sheet, accessories/interior diagram, color palette bar,
  height-scale ruler) with what each one is for and when to include or drop it.
- `references/anti-slop-discipline.md` — the character-sheet-specific failure modes
  (off-model drift between panels, style drift between the turnaround and detail
  close-ups, generic stock-mascot clichés, unlabeled/mislabeled panels) and the
  pre-generation gate to run before every `generate_image` call.
- `examples/ding-bot-character-sheet.md` — a full worked compositional-deconstruction
  example (the "Ding-Bot" AI-assistant mascot sheet) at the target level of detail,
  showing every turnaround view, detail panel, palette swatch, and technical-diagram
  callout captured as its own element.
