# Showcase examples

Six asset categories generated with the connected Ideogram MCP tools, using the techniques documented in this skill. Every prompt below is the exact caption passed to `mcp__ideogram__generate_image`'s `prompt` field — copy one verbatim to see how it renders, or use it as a template.

All were generated on the default Ideogram 4.0 model (no `custom_model_uri`), `rendering_speed: QUALITY`.

## 1. Editorial product photo — precise `color_palette` control

Demonstrates the photographic path (`photo` + `medium: "photograph"`) and exact hex steering instead of color-adjectives.

![Editorial product photo](images/01-editorial-product-photo.png)

```json
{
  "high_level_description": "A ceramic pour-over coffee dripper and mug on a rustic wooden table beside a sunlit window, styled for a specialty coffee brand's lifestyle campaign.",
  "style_description": {
    "aesthetics": "clean, warm, editorial, minimal",
    "lighting": "soft natural window light with a warm highlight on the ceramic glaze and a cool shadow beneath the mug",
    "photo": "50mm lens, f/2.0, shallow depth of field, slightly overhead angle",
    "medium": "photograph",
    "color_palette": ["#8B5E3C", "#D9A066", "#F4E9DA", "#2E2A26"]
  },
  "compositional_deconstruction": {
    "background": "A softly blurred wooden tabletop and a hint of window light, muted off-white wall in the far background, no clutter or extra objects.",
    "elements": [
      {"type": "obj", "desc": "A white ceramic pour-over dripper sitting atop a matching mug, steam rising gently, positioned left-of-center.", "color_palette": ["#F4E9DA", "#8B5E3C"]}
    ]
  }
}
```

`aspect_ratio: "4x5"`

## 2. Vintage sci-fi poster — `art_style` + in-image text via bounding boxes

Demonstrates the non-photo path (`art_style` + `medium: "illustration"`) and rendering multiple literal text strings at specific positions via `compositional_deconstruction.elements`.

![Vintage sci-fi poster](images/02-vintage-scifi-poster.png)

```json
{
  "high_level_description": "A 1980s pulp sci-fi movie poster for a fictional film called 'Chrome Horizon', featuring a lone astronaut silhouetted against twin moons.",
  "style_description": {
    "aesthetics": "retro-futuristic, dramatic, pulp adventure",
    "lighting": "bold rim lighting from twin moons, deep shadow silhouette in foreground",
    "medium": "illustration",
    "art_style": "airbrushed gradients, halftone dot texture, heavy analog film grain, painterly brushstroke shading",
    "color_palette": ["#FF6B35", "#2E1A47", "#F2C14E", "#0B0C10"]
  },
  "compositional_deconstruction": {
    "background": "A rust-orange alien desert horizon beneath two glowing moons in a deep violet sky, faint starfield above.",
    "elements": [
      {"type": "obj", "bbox": [300, 300, 900, 700], "desc": "A lone astronaut in a weathered spacesuit, standing silhouetted, helmet reflecting the twin moons.", "color_palette": ["#0B0C10", "#F2C14E"]},
      {"type": "text", "bbox": [60, 100, 180, 900], "text": "CHROME HORIZON", "desc": "Bold condensed display type across the top third of the poster, outlined in warm orange with a subtle drop shadow."},
      {"type": "text", "bbox": [910, 250, 960, 750], "text": "THE SKY HAS TEETH", "desc": "Small italic tagline centered near the bottom of the poster."}
    ]
  }
}
```

`aspect_ratio: "2x3"`

## 3. Packaging label — `graphic_design` medium with multiple text elements

Demonstrates flat graphic design, several independent text elements (wordmark + subtitle), and the highlight/shadow lighting tip from the schema reference.

![Packaging label](images/03-packaging-label.png)

```json
{
  "high_level_description": "A minimalist packaging label design for a fictional herbal tea brand called 'Solace', wrapped around a cylindrical tin.",
  "style_description": {
    "aesthetics": "calm, botanical, premium, minimal",
    "lighting": "even, diffuse studio lighting with a soft highlight along the tin's curve and a subtle shadow on the lower edge",
    "medium": "graphic_design",
    "art_style": "flat vector illustration, generous whitespace, thin botanical line art, serif wordmark",
    "color_palette": ["#F7F3E9", "#4A6B4E", "#C97B4A", "#2B2B28"]
  },
  "compositional_deconstruction": {
    "background": "A solid sage-green cylindrical tin surface with a cream label wrapped around the front.",
    "elements": [
      {"type": "obj", "desc": "A thin single-line botanical illustration of a chamomile sprig, centered above the wordmark.", "color_palette": ["#4A6B4E"]},
      {"type": "text", "text": "SOLACE", "desc": "Large serif wordmark centered on the cream label, dark charcoal color."},
      {"type": "text", "text": "chamomile + lavender", "desc": "Small lowercase sans-serif subtitle directly beneath the wordmark, terracotta color."}
    ]
  }
}
```

`aspect_ratio: "1x1"`

## 4. Abstract 3D icon — `3d_render` medium

Demonstrates the `3d_render` medium value and a minimal, geometric single-subject composition — a different rendering register from the photographic and illustrative examples above.

![Abstract 3D icon](images/04-abstract-3d-icon.png)

```json
{
  "high_level_description": "An abstract app icon: a smooth glass sphere balanced on a floating coral-colored ring, minimal 3D render on a soft gradient background.",
  "style_description": {
    "aesthetics": "clean, modern, tactile, soft",
    "lighting": "soft studio three-point lighting with a bright highlight on the sphere's upper-left and a warm ambient occlusion shadow beneath the ring",
    "medium": "3d_render",
    "art_style": "smooth matte and glass materials, soft global illumination, subtle depth of field",
    "color_palette": ["#FFFFFF", "#FF8B6B", "#6BAFFF", "#1A1A1A"]
  },
  "compositional_deconstruction": {
    "background": "A soft diagonal gradient from pale blue to white, subtly blurred.",
    "elements": [
      {"type": "obj", "desc": "A translucent glass sphere resting atop a matte coral ring, centered, both floating slightly above the gradient background.", "color_palette": ["#6BAFFF", "#FF8B6B"]}
    ]
  }
}
```

`aspect_ratio: "1x1"`

## 5. Style extraction — reference image to new subject

Demonstrates the full loop from `references/style-extraction-workflow.md`: generate a reference image, run `describe_image` on it to extract the recipe (medium, palette, lighting, composition), then apply that recipe to an unrelated subject via a fresh `generate_image` call.

**Step 1 — reference image:**

![Style extraction reference](images/05a-style-extraction-reference.png)

```json
{
  "high_level_description": "A rain-soaked back alley at night lit by overlapping neon signs, cyberpunk city aesthetic.",
  "style_description": {
    "aesthetics": "moody, cinematic, neon-drenched, atmospheric",
    "lighting": "magenta and cyan neon glow reflecting off wet pavement, deep shadow in the alley's recesses",
    "photo": "35mm lens, f/1.8, slight low angle, long exposure reflections",
    "medium": "photograph",
    "color_palette": ["#FF2E92", "#2EE6FF", "#0A0A12", "#3A1E4D"]
  },
  "compositional_deconstruction": {
    "background": "A narrow alley between weathered concrete buildings, steam rising from a vent, neon signage reflected in puddles on the ground.",
    "elements": [
      {"type": "obj", "desc": "A row of overlapping neon signs mounted on the alley walls, magenta and cyan glow.", "color_palette": ["#FF2E92", "#2EE6FF"]}
    ]
  }
}
```

**Step 2 — `describe_image` extracted recipe:** moody/cinematic/neon-drenched aesthetic, magenta-and-cyan neon lighting reflected off wet surfaces, 35mm low-angle photograph, deep blues/reds/purples against muted concrete grays.

**Step 3 — recipe applied to a new subject (a house cat):**

![Style extraction applied to new subject](images/05b-style-extraction-applied.png)

```json
{
  "high_level_description": "A house cat sitting calmly in a rain-soaked neon-lit alley, cyberpunk city aesthetic matching the reference mood.",
  "style_description": {
    "aesthetics": "moody, cinematic, neon-drenched, atmospheric",
    "lighting": "magenta and cyan neon glow reflecting off wet fur and pavement, deep shadow behind the cat",
    "photo": "35mm lens, f/1.8, slight low angle, long exposure reflections",
    "medium": "photograph",
    "color_palette": ["#FF2E92", "#2EE6FF", "#0A0A12", "#3A1E4D"]
  },
  "compositional_deconstruction": {
    "background": "The same narrow neon-lit alley, wet pavement reflecting magenta and cyan signage, steam rising faintly in the background.",
    "elements": [
      {"type": "obj", "desc": "A calm house cat sitting upright and facing the camera, fur rim-lit by the neon glow, positioned center-frame.", "color_palette": ["#2EE6FF", "#0A0A12"]}
    ]
  }
}
```

`aspect_ratio: "3x2"` for both.

## 6. Brand identity moodboard — 3×3 panel grid

Demonstrates the `moodboard-generator` skill's template: a single paragraph prompt naming all nine panels explicitly (Color Palette, Typography, Logo Exploration, Iconography, Photography & Graphics, Material Samples, Abstract Pattern, Mood Imagery, Application Mockup), run through the anti-slop pre-generation gate before calling `generate_image`. Unlike examples 1-5, this skill's prompt is a plain descriptive paragraph rather than the structured JSON caption format — see `skills/moodboard-generator/examples/anchorpoint-moodboard.md` for the full worked example, including the compositional deconstruction.

![Brand identity moodboard](images/06-moodboard-anchorpoint.png)

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

`aspect_ratio: "1x1"`
