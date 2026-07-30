# Compositional Deconstruction Format

This is the structured JSON breakdown of a brand identity sheet — the artifact that
makes a generated (or reference) image reusable as a spec, not just a picture to look
at. Think of it as "what a vision model would say if asked to precisely annotate every
panel of this image" — a `high_level_description` one-liner, then an `elements` array
where every panel, icon, button, and piece of text gets its own bounding box and
description.

## Schema

```json
{
  "high_level_description": "One sentence: what the whole sheet is and what it's showing.",
  "compositional_deconstruction": {
    "background": "One paragraph describing the ground/grid the panels sit on.",
    "elements": [
      {
        "type": "obj",
        "bbox": [x_min, y_min, x_max, y_max],
        "desc": "What this panel/icon/button is — its shape, material, color, and role."
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
  reader should be able to tell "this panel is upper-left, this one is dead center and
  twice the size" from the boxes alone.
- `type: "obj"` covers any non-text visual element — a UI card, an icon, a button, a
  texture tile, a character illustration.
- `type: "text"` is for anything with literal legible characters — wordmark, labels,
  button copy, price tags. Always include the exact string in `text`, separate from the
  visual description in `desc`.
- Order elements roughly by reading order (top-left to bottom-right, background first)
  — it makes the file scannable and mirrors how a human would describe the sheet aloud.
- Don't fabricate precision you don't have. If you're annotating a reference image by
  eye rather than one you composed yourself, coarse-but-honest boxes beat
  suspiciously-exact ones — a rough quadrant estimate labeled as such is more useful
  than fake precision that misleads whoever reads the spec next.

## Why this exists (not just decoration)

A loose paragraph description of a brand sheet ("it has some icons and a big logo in
the middle") can't be handed to another agent as a spec — they'd have to look at the
image themselves and re-derive the layout. A bbox-level breakdown *is* the spec: it
tells a downstream agent (a hallmark/rebrand agent, a future logo pass, a website-build
agent) exactly which panel is the primary wordmark treatment, which icon is the
character mark, which button shows the accent color, without re-interpreting the image
from scratch. That's what let `technauts-logo-prompting-space`'s output feed directly
into the site rebrand without a second round of "what did you mean by this panel."

## Worked example (abbreviated — full version in `examples/technauts-identity-sheet.md`)

```json
{
  "high_level_description": "A polished brand identity sheet for 'Technauts' showcasing a bold geometric logo, a cohesive navy-cyan-gray color palette, futuristic UI components including pill buttons and app icons, and a stylized cosmic astronaut character, all arranged in a structured guide layout.",
  "compositional_deconstruction": {
    "background": "A crisp, light silver-gray background overlaid with a large-scale geometric grid of faint dark-gray lines forming intersecting diagonal and orthogonal shapes, evoking a technical blueprint aesthetic.",
    "elements": [
      {
        "type": "obj",
        "bbox": [39, 41, 149, 283],
        "desc": "A white rectangular UI card with softly rounded corners presenting brand identity information — bold dark-navy 'TECHNAUTS' header, large stylized wordmark centered, small supporting text at the bottom."
      },
      {
        "type": "text",
        "bbox": [94, 110, 144, 217],
        "text": "Technauts",
        "desc": "Large, bold dark-navy sans-serif wordmark, geometric letterforms, strong stroke contrast, tight spacing."
      },
      {
        "type": "obj",
        "bbox": [168, 126, 219, 200],
        "desc": "A vibrant pill-shaped button, fully rounded ends, bright saturated cyan — the primary call-to-action color."
      }
    ]
  }
}
```

See the full example file for every panel (icon grid, gear icon, astronaut character
panel, secondary star lockup) at the same level of detail.
