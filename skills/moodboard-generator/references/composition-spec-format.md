# Compositional Deconstruction Format

This is the structured JSON breakdown of a moodboard — the artifact that makes a
generated (or reference) image reusable as a spec, not just a picture to react to.
Think of it as "what a vision model would say if asked to precisely annotate every
panel of this board" — a `high_level_description` one-liner, then an `elements` array
where every panel and piece of text gets its own bounding box and description.

This is the exact same schema `brand-identity-sheet` and `character-model-sheet` use,
unmodified — the annotation problem (precisely locating panels/text in a multi-panel
design sheet) is identical regardless of what stage of the process the sheet belongs
to. See those skills' `composition-spec-format.md` for the fuller rationale; this file
covers the moodboard-specific worked example.

## Schema

```json
{
  "high_level_description": "One sentence: what the whole board is exploring and for what brand.",
  "compositional_deconstruction": {
    "background": "One paragraph describing the ground/grid the nine panels sit on.",
    "elements": [
      {
        "type": "obj",
        "bbox": [x_min, y_min, x_max, y_max],
        "desc": "What this panel is — which of the nine template panels it corresponds to, its content, materials, and role."
      },
      {
        "type": "text",
        "bbox": [x_min, y_min, x_max, y_max],
        "text": "The literal text string in this panel, if any.",
        "desc": "Typography treatment: weight, case, color, size relative to the board."
      }
    ]
  }
}
```

**Field notes:**

- `bbox` is `[x_min, y_min, x_max, y_max]` — pixel coordinates if you have the source
  resolution, or a normalized 0-1000 grid if estimating from a downscaled view (state
  which one you're using at the top of the file).
- Tag each `obj` element with which of the nine template panels it is (Color Palette,
  Typography, Logo Exploration, Iconography Style, Photography & Graphics, Material
  Samples, Abstract Patterns, Mood Imagery, Application Mockup) in the `desc` — this is
  what makes the JSON reusable later, e.g. a future `logo-prompting` pass pointing
  specifically at "the Logo Exploration panel from this moodboard" as its starting
  reference.
- Order elements in reading order (typically left-to-right, top-to-bottom across the
  3×3 grid, background first).
- Don't fabricate precision you don't have — coarse-but-honest boxes beat
  suspiciously-exact ones on a reference image you're annotating by eye.

## Worked example (abbreviated)

```json
{
  "high_level_description": "A 3x3 brand moodboard for a fictional analog-industrial coffee-equipment brand, exploring a warm terracotta-and-brushed-steel palette, a monospace-adjacent type direction, and a compass-rose logo motif, on a clean white ground.",
  "compositional_deconstruction": {
    "background": "Pure white background with thin gray grid lines separating nine equal panels in a 3x3 layout, each panel bearing a small uppercase caption in the top-left corner.",
    "elements": [
      {
        "type": "obj",
        "bbox": [0, 0, 333, 333],
        "desc": "Color Palette panel — five swatch chips (terracotta, warm cream, charcoal, brushed-steel gray, a single rust accent), each labeled with a hex value beneath it."
      },
      {
        "type": "text",
        "bbox": [10, 10, 120, 30],
        "text": "COLOR PALETTE",
        "desc": "Small uppercase caption, charcoal, sans-serif, top-left of the panel."
      },
      {
        "type": "obj",
        "bbox": [333, 0, 666, 333],
        "desc": "Typography panel — a serif display face paired with a monospace body face, shown at three sizes stacked vertically, in charcoal ink on cream."
      },
      {
        "type": "obj",
        "bbox": [666, 0, 1000, 333],
        "desc": "Logo Exploration panel — three small rough sketches of a compass-rose motif, hand-drawn linework, unfinished/exploratory feel, in charcoal ink."
      }
    ]
  }
}
```

See `examples/` for full worked examples as real moodboard jobs are run.
