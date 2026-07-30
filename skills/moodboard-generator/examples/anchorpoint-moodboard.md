# Worked Example — Anchorpoint Coffee Moodboard

Fictional brand: **Anchorpoint**, an analog-industrial coffee-equipment brand.
Template variables: `{Adjectives}` grounded, unhurried, honest — `{Color Focus}`
warm terracotta with brushed-steel and charcoal — `{Logo Concept}` a compass-rose
motif suggesting brew time — `{Imagery & Texture}` brushed steel, terracotta
ceramic, uncoated paper grain — `{Emotional Impact}` quiet confidence, like a
well-made tool — `{Notes}` none.

## 1. The paragraph prompt (what generated the board)

```
A brand identity moodboard laid out as a clean 3x3 grid on a pure white background
with thin gray grid lines, nine equal square panels each with a small uppercase
caption in its top-left corner, for a fictional analog-industrial coffee-equipment
brand called Anchorpoint that feels grounded, unhurried, and honest. Panel 1
"COLOR PALETTE": five labeled swatch chips — warm terracotta, warm cream, charcoal
ink, brushed-steel gray, one small rust accent — each with a hex code beneath it.
Panel 2 "TYPOGRAPHY": a serif display face paired with a monospace body face,
shown stacked at three sizes in charcoal ink on cream, evoking a mechanical parts
catalog. Panel 3 "LOGO EXPLORATION": three small rough hand-drawn sketches of a
compass-rose motif suggesting brew time, unfinished exploratory linework, not one
finished mark. Panel 4 "ICONOGRAPHY": a simple set of line-weight icons (dial,
spout, filter cone, timer) in consistent thin charcoal linework with rounded
corners. Panel 5 "PHOTOGRAPHY & GRAPHICS": a warm, low-contrast photograph of
brushed-steel coffee equipment on a wood counter, natural window light. Panel 6
"MATERIAL SAMPLES": close-up textures of brushed steel, matte terracotta ceramic,
and uncoated paper grain, arranged as physical swatches. Panel 7 "ABSTRACT
PATTERN": a repeating geometric pattern built from small compass-rose ticks in
charcoal on cream, subtle and structured, not a gradient. Panel 8 "MOOD IMAGERY":
a quiet, warm-toned photograph of steam rising from a terracotta cup on a wood
table, conveying unhurried calm. Panel 9 "APPLICATION MOCKUP": a coffee bag label
mockup on kraft paper using the Anchorpoint wordmark, terracotta and charcoal
palette, and the compass-rose glyph, showing the system working together. No
gradients, no neural-network or circuit-board textures, no glowing orb, no
purple-to-pink wash, no generic stock-photo clichés, no placeholder brand names
other than Anchorpoint.
```

Generated with `mcp__ideogram__generate_image`, `aspect_ratio: "1x1"`,
`rendering_speed: "QUALITY"`. `style_type` was intentionally omitted — no
`custom_model_uri` was supplied, and Ideogram's default v4 path ignores
`style_type` entirely (it only applies to a custom-model/v3 generation), so
setting it here would have been a silent no-op.

## 2. The compositional deconstruction (what came out of it)

Coordinate space: normalized 0-1000 grid, estimated from the generated 1024x1024
image (each of the nine panels occupies roughly a 333x333 cell).

```json
{
  "high_level_description": "A 3x3 brand moodboard for the fictional analog-industrial coffee brand Anchorpoint, exploring a terracotta-and-brushed-steel palette, a serif/monospace type pairing, and a compass-rose logo motif, on a cream ground.",
  "compositional_deconstruction": {
    "background": "Warm off-white/cream ground with thin gray grid lines dividing nine equal panels in a 3x3 layout, each panel with a small uppercase caption in its top-left corner.",
    "elements": [
      {
        "type": "obj",
        "bbox": [0, 0, 333, 333],
        "desc": "Color Palette panel — five swatch chips (terracotta #B5654A, cream #F2B00B, charcoal ink #2C6470, brushed steel #8A847C, rust accent #9C442E) each labeled with a name and hex value beneath it."
      },
      {
        "type": "text",
        "bbox": [15, 15, 130, 30],
        "text": "COLOR PALETTE",
        "desc": "Small uppercase caption, charcoal, sans-serif, top-left of the panel."
      },
      {
        "type": "obj",
        "bbox": [333, 0, 666, 333],
        "desc": "Typography panel — the Anchorpoint wordmark set large in a serif display face, a smaller serif lockup beneath it, and a fine-print line 'EST. MMXXIV · ANCHORPOINT COFFEE CO.' in a monospace-adjacent caps face, all charcoal ink on cream."
      },
      {
        "type": "obj",
        "bbox": [666, 0, 1000, 333],
        "desc": "Logo Exploration panel — three small rough compass-rose sketches, hand-drawn linework with construction guides visible, unfinished/exploratory feel, in charcoal ink."
      },
      {
        "type": "obj",
        "bbox": [0, 333, 333, 666],
        "desc": "Iconography panel — four rounded-corner line icons (dial, filter cone/vessel, funnel, clock/timer) in consistent thin charcoal linework, arranged in a 2x2 grid within the cell."
      },
      {
        "type": "obj",
        "bbox": [333, 333, 666, 666],
        "desc": "Photography & Graphics panel — a warm, low-contrast photograph of a brushed-steel gooseneck kettle on a wood counter with soft natural window light."
      },
      {
        "type": "obj",
        "bbox": [666, 333, 1000, 666],
        "desc": "Material Samples panel — three overlapping physical swatches: brushed steel, matte terracotta, and uncoated kraft-paper grain, shown as stacked squares."
      },
      {
        "type": "obj",
        "bbox": [0, 666, 333, 1000],
        "desc": "Abstract Pattern panel — a repeating grid of small plus-sign/compass-tick marks in charcoal on cream, structured and evenly spaced, not a gradient or collage."
      },
      {
        "type": "obj",
        "bbox": [333, 666, 666, 1000],
        "desc": "Mood Imagery panel — a warm-toned photograph of steam rising from a terracotta cup on a wood table against a muted sage-green background, conveying unhurried calm."
      },
      {
        "type": "obj",
        "bbox": [666, 666, 1000, 1000],
        "desc": "Application Mockup panel — a kraft-paper coffee bag with the Anchorpoint wordmark, a small compass/plus glyph, and supporting copy ('SINGLE ORIGIN · SLOW BREW', 'ROASTED IN SMALL BATCHES'), showing the palette, type, and mark working together."
      }
    ]
  }
}
```

## What worked

All nine panels held their own borders with no bleed into a Pinterest-style
collage — the "identity sheet" framing from `logo-prompting`'s discipline (not
saying "moodboard" in isolation, naming every panel explicitly) carried over
cleanly even though this skill deliberately keeps the word "moodboard" in the
prompt. The Logo Exploration panel stayed appropriately rough (three sketches,
visible construction lines) rather than rendering one finished mark, and the
Application Mockup panel actually reads as the other eight panels' choices
working together rather than a bolted-on generic label.
