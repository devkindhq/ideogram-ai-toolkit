# Compositional Deconstruction Format

This is the exact same `high_level_description` + `elements[]` `{type, bbox, desc}`
schema `moodboard-generator`, `brand-identity-sheet`, and `character-model-sheet` use,
unmodified at the field level — reused per the spec's Architecture section, which
explicitly calls this "a structural adaptation, not a schema change." See
`moodboard-generator/references/composition-spec-format.md` for the fuller rationale.

The one adaptation: here, `elements[]` entries describe **motif instances placed within
one continuous tile canvas**, not named UI panels the way `moodboard-generator`'s 3×3
grid does. A pattern tile has no panel grid to annotate — instead it has motif instances
(a floral cluster, a stripe, a vine) scattered or repeating across a single continuous
ground, some of which cross a tile edge and must reconnect with their counterpart on the
opposite edge for the tile to read as seamless. That edge-crossing behavior is the one
piece of information this schema captures beyond the base schema's field notes.

## Schema

```json
{
  "high_level_description": "One sentence: what the tile is and what repeat/material it represents.",
  "compositional_deconstruction": {
    "background": "One paragraph describing the ground the motif instances sit on — color, texture, any base pattern.",
    "elements": [
      {
        "type": "obj",
        "bbox": [x_min, y_min, x_max, y_max],
        "desc": "What this motif instance is, its position within the tile, and — if it crosses a tile edge — specifically how it crosses or reconnects at that edge, e.g. \"large floral cluster, stems connecting diagonally to the cluster at lower-right to preserve edge continuity.\""
      },
      {
        "type": "text",
        "bbox": [x_min, y_min, x_max, y_max],
        "text": "The literal text string in this motif instance, if any.",
        "desc": "Typography treatment: weight, case, color, size relative to the tile."
      }
    ]
  }
}
```

**Field notes:**

- `bbox` is `[x_min, y_min, x_max, y_max]` — pixel coordinates if you have the source
  resolution, or a normalized 0-1000 grid if estimating from a downscaled view (state
  which one you're using at the top of the file).
- Order elements in reading order across the tile (background first, then motif
  instances roughly left-to-right, top-to-bottom).
- Any motif instance touching a tile edge must have its `desc` state which edge(s) it
  touches and whether/how it reconnects with the instance at the opposite edge — this is
  the field this skill's schema adds beyond the unmodified base.
- Don't fabricate precision you don't have — coarse-but-honest boxes beat
  suspiciously-exact ones on a reference image you're annotating by eye.

## Worked example (abbreviated)

```json
{
  "high_level_description": "A seamless straight-repeat floral wallpaper tile in a warm terracotta-and-cream palette, with trailing vines connecting motif clusters across the tile edges.",
  "compositional_deconstruction": {
    "background": "Flat warm cream ground, no directional lighting, no vignette or border, edge-to-edge coverage.",
    "elements": [
      {
        "type": "obj",
        "bbox": [50, 60, 320, 340],
        "desc": "Large terracotta floral cluster centered in the upper-left quadrant, five overlapping bloom shapes with a charcoal center dot each, fully contained within the tile with no edge contact."
      },
      {
        "type": "obj",
        "bbox": [0, 400, 1000, 480],
        "desc": "Trailing vine motif running horizontally, exits the tile at the left edge (y 400-480) and the right edge (y 400-480) at matching heights and stem angle, reconnecting seamlessly when the tile repeats side-by-side."
      },
      {
        "type": "obj",
        "bbox": [700, 850, 1000, 1000],
        "desc": "Small rust-colored bud cluster in the lower-right corner, stems crossing the bottom edge and the right edge simultaneously, both cut ends matching the stem angle and thickness of the corresponding bud cluster fragment at the top-left corner so the four-way tile join reads as one continuous stem."
      }
    ]
  }
}
```

See `examples/` for full worked examples as real pattern-tile jobs are run.
