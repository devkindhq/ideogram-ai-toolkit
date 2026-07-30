# Worked Example — Technauts Identity Sheet

The real job this skill was built from. Two artifacts: the paragraph prompt used to
generate the sheet, and the compositional deconstruction produced afterward to make
its layout reusable.

## 1. The paragraph prompt (what generated the sheet)

```
A dynamic moodboard layout featuring multiple visual varieties for a brand or team
called "Technauts," displayed in a sleek, modern design grid. The composition includes
swatches of deep space navy, electric cyan, and metallic silver, alongside futuristic
typography samples, icon concepts, and texture tiles. Overlaid throughout the board,
the word "TECHNAUTS" appears in bold, angular, tech-forward lettering as a recurring
branding anchor. The overall aesthetic blends dark sci-fi atmosphere with clean digital
minimalism, evoking exploration, innovation, and cutting-edge technology, lit with cool
blue neon accents against a deep charcoal background.
```

Note what this prompt does right by this skill's step-3 discipline: it names the exact
3-color palette (navy/cyan/silver) with role words ("swatches," "recurring branding
anchor"), names the material vocabulary ("texture tiles," "metallic silver"), and gives
one mood sentence with concrete anchors ("dark sci-fi," "clean digital minimalism")
rather than adjective soup.

## 2. The compositional deconstruction (what came out of it)

Coordinate space: pixel-estimated from the generated image, `[x_min, y_min, x_max,
y_max]`.

```json
{
  "high_level_description": "A polished brand identity sheet for 'Technauts' showcasing a bold geometric logo, a cohesive navy-cyan-gray color palette, futuristic UI components including pill buttons and app icons, and a stylized cosmic astronaut character, all arranged in a structured guide layout.",
  "compositional_deconstruction": {
    "background": "A crisp, light silver-gray background overlaid with a large-scale geometric grid of faint dark-gray lines forming intersecting diagonal and orthogonal shapes, evoking a technical blueprint aesthetic. The layout is divided into clearly delineated sections: a left column of UI cards and icon grids, a dominant central logo zone, and a right column of additional UI panels and a character showcase.",
    "elements": [
      { "type": "obj", "bbox": [39, 41, 149, 283], "desc": "A white rectangular UI card with softly rounded corners presenting brand identity information. Bold dark-navy 'TECHNAUTS' header at the top, a large stylized 'Technauts' wordmark centered, small supporting text at the bottom, clean sans-serif typography on a pure white surface." },
      { "type": "obj", "bbox": [39, 381, 201, 517], "desc": "A tall vertical app icon with generously rounded corners depicting a deep cosmic scene. Dark purple-to-midnight-blue gradient background, glowing neon-purple and hot-pink geometric crystalline structure radiating light from the center." },
      { "type": "obj", "bbox": [39, 537, 110, 610], "desc": "A square app icon with rounded corners featuring a vivid blue and white abstract organic pattern — luminous tendrils of electric blue light curling and blooming against a deep navy background, resembling bioluminescent coral or plasma filaments." },
      { "type": "obj", "bbox": [39, 626, 110, 696], "desc": "A square app icon with a hyper-realistic metallic silver sphere — smooth mirror-like surface with subtle environmental reflections and a soft shadow beneath it." },
      { "type": "obj", "bbox": [126, 537, 201, 610], "desc": "A square app icon showing a light gray textured surface composed of a fine, precise grid of intersecting diagonal lines — a technical mesh or carbon-fiber weave pattern in monochrome silver tones." },
      { "type": "obj", "bbox": [126, 626, 201, 696], "desc": "A square app icon featuring a vertical brushed-metal texture in polished silver — parallel fine striations top to bottom, sleek industrial sheen like machined aluminum." },
      { "type": "obj", "bbox": [39, 722, 110, 796], "desc": "A square app icon depicting a detailed metallic silver gear/cog with a dark charcoal handle — precisely rendered teeth, reflective surface, suggesting a mechanical tool or settings utility." },
      { "type": "obj", "bbox": [126, 722, 201, 796], "desc": "A square app icon showing a dark charcoal textured surface with a small, intensely glowing light source at its center, blooming vivid purple and electric blue in a lens-flare halo." },
      { "type": "obj", "bbox": [126, 883, 201, 957], "desc": "A square app icon featuring a dark purple/midnight-blue abstract organic pattern — glowing neon-blue and violet tendrils swirling against the dark background." },
      { "type": "obj", "bbox": [168, 41, 219, 114], "desc": "A sleek pill-shaped button, fully rounded ends, filled with deep rich navy blue — smooth matte surface, solid confident presence." },
      { "type": "obj", "bbox": [168, 126, 219, 200], "desc": "A vibrant pill-shaped button, fully rounded ends, bright saturated cyan — the primary call-to-action color, clean electric luminosity." },
      { "type": "obj", "bbox": [168, 211, 219, 285], "desc": "A soft pill-shaped button, fully rounded ends, neutral light gray — a secondary/disabled-state UI control." },
      { "type": "obj", "bbox": [816, 274, 960, 417], "desc": "A square app icon with a dark navy-blue background — a detailed astronaut helmet rendered in a glowing illustrative style, cyan-gradient visor, small metallic antenna, retro-futuristic space-explorer aesthetic." },
      { "type": "text", "bbox": [48, 53, 98, 107], "text": "TECHNAUTS", "desc": "Small, crisp dark-navy sans-serif uppercase brand-label header at the top of the first UI card." },
      { "type": "text", "bbox": [48, 250, 98, 300], "text": "Edition", "desc": "Small, medium-gray lightweight sans-serif secondary label within the first UI card." },
      { "type": "text", "bbox": [94, 110, 144, 217], "text": "Technauts", "desc": "Large, bold dark-navy sans-serif wordmark — geometric letterforms, strong stroke contrast, tight spacing. The primary logo text." },
      { "type": "text", "bbox": [128, 53, 178, 103], "text": "Offroad edition", "desc": "Very small, light-gray condensed sub-edition label." },
      { "type": "text", "bbox": [128, 258, 178, 308], "text": "$30", "desc": "Very small, light-gray price label, clean minimal style." },
      { "type": "text", "bbox": [48, 816, 98, 866], "text": "Cosmogear", "desc": "Small, medium-gray sub-brand name label." },
      { "type": "text", "bbox": [48, 897, 98, 947], "text": "TECHNAUTS", "desc": "Small, bold dark-navy uppercase brand identifier, matching the primary brand typography." },
      { "type": "text", "bbox": [180, 816, 230, 866], "text": "Offroad edition", "desc": "Very small, light-gray condensed sub-edition label, second instance." },
      { "type": "text", "bbox": [419, 38, 563, 961], "text": "technauts", "desc": "Massive headline-scale geometric wordmark dominating the central zone — bold, futuristic, tightly spaced, deep navy blue, sharp angles, the composition's architectural anchor." },
      { "type": "text", "bbox": [763, 48, 813, 120], "text": "TECHNAUTS", "desc": "Small bold dark-navy uppercase text inside a white pill-shaped button — branded CTA label, high contrast on white." },
      { "type": "text", "bbox": [763, 226, 813, 276], "text": "Setup", "desc": "Small crisp white text inside a dark navy pill-shaped button — clear action label, strong contrast." },
      { "type": "text", "bbox": [828, 53, 878, 146], "text": "● 6GB — 5x/week/10/day", "desc": "Small light-gray technical-spec text (data size + frequency), clean condensed style within a UI info panel." },
      { "type": "text", "bbox": [846, 53, 896, 103], "text": "Character", "desc": "Small medium-gray section/category label, lightweight modern typeface." },
      { "type": "text", "bbox": [828, 287, 878, 341], "text": "TECHNAUTS", "desc": "Small bold white uppercase text overlaid on the astronaut icon — brand identifier, legible against the dark cosmic background." },
      { "type": "text", "bbox": [839, 574, 895, 915], "text": "technauts", "desc": "Medium-sized geometric wordmark, bold futuristic dark-navy letterforms, followed by a four-pointed star icon — the secondary brand lockup." },
      { "type": "obj", "bbox": [831, 917, 881, 967], "desc": "A crisp four-pointed star icon in solid dark navy blue, perfectly symmetrical elongated diamond points — the decorative brand glyph accompanying the secondary lockup." }
    ]
  }
}
```

## What to take from this example

- The paragraph prompt is short (5 sentences) and every color/material/mood claim in
  it shows up literally in the resulting elements — nothing in the prompt was wasted,
  and nothing in the sheet came from nowhere.
- The elements array mixes `obj` (icons, cards, buttons) and `text` (every wordmark
  instance, every label) — nine distinct app-icon materials, three pill states, two
  wordmark scales (card-size and hero-size), and one secondary glyph lockup. That's the
  panel variety `references/sheet-anatomy.md` describes.
- This is annotating a sheet generated from a loose brief (no locked brand.md existed
  yet for Technauts at this point) — the palette in the sheet (navy/cyan/silver) is
  what later informed the brand's real Cobalt, then Ascent, theme decisions. A sheet
  like this is often the *input* to locking a palette, not just an output of one.
