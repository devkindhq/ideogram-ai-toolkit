# Compositional Deconstruction Format

This is the structured JSON breakdown of a character model sheet — the artifact that
makes a generated (or reference) image reusable as a spec, not just a picture to look
at. Think of it as "what a vision model would say if asked to precisely annotate every
panel of this image" — a `high_level_description` one-liner, then an `elements` array
where every view, close-up, text block, and swatch gets its own bounding box and
description.

This schema is identical to `brand-identity-sheet`'s composition-spec format — the
annotation problem (precisely locating panels/text in a multi-panel design sheet) is
the same regardless of whether the sheet is showing a brand system or a character.

## Schema

```json
{
  "high_level_description": "One sentence: what the whole sheet is and what character it's showing.",
  "compositional_deconstruction": {
    "background": "One paragraph describing the ground/grid the panels sit on.",
    "elements": [
      {
        "type": "obj",
        "bbox": [x_min, y_min, x_max, y_max],
        "desc": "What this view/panel/prop is — pose, angle, silhouette, color, and role."
      },
      {
        "type": "text",
        "bbox": [x_min, y_min, x_max, y_max],
        "text": "The literal text string in this panel.",
        "desc": "Typography treatment: weight, case, color, size relative to the sheet."
      }
    ]
  }
}
```

**Field notes:**

- `bbox` is `[x_min, y_min, x_max, y_max]` in whatever coordinate space you're working
  in — pixel coordinates if you have the source resolution, or a normalized 0-1000
  grid if you're estimating from a downscaled view (state which one you're using at the
  top of the file). Precision matters less than *relative* placement being right — a
  reader should be able to tell "the front view is upper-left, the back view is
  directly below it" from the boxes alone.
- `type: "obj"` covers any non-text visual element — a full-body view, a face close-up,
  a hand/prop detail, a color swatch bar, an interior-diagram illustration.
- `type: "text"` is for anything with literal legible characters — the character name
  headline, panel labels (FRONT / SIDE / BACK), overview/personality copy, height-scale
  numbers, callout labels on a technical diagram. Always include the exact string in
  `text`, separate from the visual description in `desc`.
- Order elements roughly by reading order (top-left to bottom-right, background first)
  — it makes the file scannable and mirrors how a human would describe the sheet aloud.
- Don't fabricate precision you don't have. If you're annotating a reference image by
  eye rather than one you composed yourself, coarse-but-honest boxes beat
  suspiciously-exact ones — a rough quadrant estimate labeled as such is more useful
  than fake precision that misleads whoever reads the spec next.

## Why this exists (not just decoration)

A loose paragraph description of a character sheet ("it has some views and a face
close-up") can't be handed to another agent as a spec — they'd have to look at the
image themselves and re-derive the layout. A bbox-level breakdown *is* the spec: it
tells a downstream agent (an animator, a 3D modeler, a second illustrator matching the
design) exactly which panel is the back view, which close-up is the signature prop,
which callout label belongs to which internal component, without re-interpreting the
image from scratch.

## Worked example (abbreviated — full version in `examples/bell-boy-character-sheet.md`)

```json
{
  "high_level_description": "A comprehensive character design sheet for Bell-Boy, a 3D orange bell-shaped AI assistant character, including a full body turnaround, head and detail views, color palettes, and a technical diagram of internal components.",
  "compositional_deconstruction": {
    "background": "A multi-panel character design sheet with a clean white background and light gray grid lines, organized into sections with thin black borders.",
    "elements": [
      {
        "type": "obj",
        "bbox": [81, 226, 307, 461],
        "desc": "Bell-Boy character in a full-body front view — orange bell-shaped body, smiling face, light blue over-ear headphones, glowing white Wi-Fi symbol on the chest."
      },
      {
        "type": "text",
        "bbox": [15, 15, 65, 172],
        "text": "BELL-BOY",
        "desc": "Large, bold, black sans-serif headline at the top left."
      },
      {
        "type": "text",
        "bbox": [334, 321, 384, 371],
        "text": "FRONT",
        "desc": "Small, bold, black sans-serif label below the front character view."
      }
    ]
  }
}
```

See the full example file for every panel (all four turnaround views, the head & detail
sheet, the color palette, and the accessories/interior diagram) at the same level of
detail.
