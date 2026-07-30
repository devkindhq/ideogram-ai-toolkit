# Worked Example — Kip (Hopcarry) Character Model Sheet

![Kip character model sheet](images/kip-hopcarry-character-sheet.webp)

A from-scratch fictional character invented to demonstrate this skill end to end: Kip,
a courier fox mascot for a fictional bike-delivery app called "Hopcarry," unrelated to
any client or real product. The paragraph prompt below was submitted to Ideogram
(`generate_image`, `style_type: "DESIGN"`, `aspect_ratio: "4x3"`,
`rendering_speed: "QUALITY"`), and the compositional deconstruction was authored
directly from that same prompt, since the panel layout was fully controlled at write
time — the same "From a generated image" shortcut this skill's own SKILL.md describes.

## 1. The paragraph prompt (what generated the sheet)

```
A character model sheet / character turnaround sheet for "Kip," a courier fox mascot
for a bike-delivery app called "Hopcarry," laid out on a clean white background with
thin light-gray grid lines and thin black panel borders, each panel labeled with a
small caption. Kip is a stylized anthropomorphic fox with rust-orange fur, a cream
chest and muzzle, tall alert ears, round amber goggles pushed up on the forehead, and a
small canvas messenger satchel in forest green slung across the body with a brass
buckle; Kip stands in a confident, forward-leaning ready-to-run pose. Rendered in a
flat cel-shaded vector illustration style, held identical across every panel. The sheet
includes: a header/overview block with a bold "KIP" headline, a "Character Model Sheet"
sub-headline, and small info boxes for character overview (Name: Kip, Alias: The Hop,
Role: Courier Mascot, Archetype: Speedy Helper), personality & traits (Quick, Loyal,
Cheerful, always slightly out of breath), wardrobe/accessories (goggles, messenger
satchel, brass buckle), and a color palette bar with primary rust-orange #C1622D,
secondary cream #F2E8D8, accent forest-green #3B5D42; a full body turnaround with FRONT
(facing directly toward camera), 3/4 VIEW (rotated roughly 45 degrees), SIDE (90 degree
profile), and BACK (facing directly away) views of Kip in the same pose, each labeled,
with a height-scale ruler in inches and centimeters running alongside; a head & detail
sheet with a FRONT FACE close-up showing the goggles and expression, a SATCHEL DETAIL
close-up showing the buckle and strap construction, and a WAVING HAND DETAIL close-up
of Kip's paw. No off-model color or proportion drift between panels, no generic
stock-mascot clichés, no invented text beyond the labels specified.
```

Note what this prompt does right by this skill's own discipline: it describes Kip once
in full (silhouette, fur colors, goggles, satchel, pose) and then names an explicit
consistency directive ("held identical across every panel"), names the exact panel set
(header/overview, four-view turnaround, head & detail sheet) rather than a generic
"character illustration," and closes with an explicit off-model-drift exclusion.

## 2. The compositional deconstruction (authored from the known prompt)

Coordinate space: pixel-estimated for a ~1200x900 canvas (4:3),
`[x_min, y_min, x_max, y_max]`.

```json
{
  "high_level_description": "A character model sheet for Kip, a rust-orange anthropomorphic fox courier mascot for the fictional bike-delivery app 'Hopcarry,' rendered in flat cel-shaded vector style. The sheet includes a header/overview block, a full body turnaround (front/3-4/side/back) with height-scale rulers, and a head & detail sheet with three close-ups.",
  "compositional_deconstruction": {
    "background": "A clean white background with thin light-gray grid lines and thin black panel borders dividing the sheet into a header row, a full-body-turnaround row, and a head-and-detail row beneath it.",
    "elements": [
      { "type": "text", "bbox": [20, 15, 260, 70], "text": "KIP", "desc": "Bold black sans-serif headline at the top left." },
      { "type": "text", "bbox": [20, 70, 320, 105], "text": "Character Model Sheet", "desc": "Medium black sans-serif sub-headline directly beneath the KIP headline." },
      { "type": "text", "bbox": [20, 115, 220, 220], "text": "CHARACTER OVERVIEW\nName: Kip\nAlias: The Hop\nRole: Courier Mascot\nArchetype: Speedy Helper", "desc": "Small black sans-serif text block in a light cream info box." },
      { "type": "text", "bbox": [230, 115, 430, 220], "text": "PERSONALITY & TRAITS\nQuick, Loyal, Cheerful,\nalways slightly out of breath", "desc": "Small black sans-serif text block in a light cream info box." },
      { "type": "text", "bbox": [440, 115, 620, 220], "text": "WARDROBE / ACCESSORIES\nGoggles, messenger satchel,\nbrass buckle", "desc": "Small black sans-serif text block in a light cream info box." },
      { "type": "obj", "bbox": [630, 115, 900, 160], "desc": "Color palette bar with three swatches: rust-orange #C1622D, cream #F2E8D8, forest-green #3B5D42." },
      { "type": "obj", "bbox": [40, 250, 260, 560], "desc": "Kip full body, FRONT view, facing directly toward camera: rust-orange fur, cream chest and muzzle, tall alert ears, round amber goggles pushed up on forehead, forest-green canvas messenger satchel with brass buckle slung across body, confident forward-leaning ready-to-run pose." },
      { "type": "text", "bbox": [110, 565, 190, 590], "text": "FRONT", "desc": "Small bold black label beneath the front view." },
      { "type": "obj", "bbox": [280, 250, 500, 560], "desc": "Kip full body, 3/4 VIEW, rotated roughly 45 degrees, same rust-orange fur, cream chest, goggles, forest-green satchel with brass buckle, identical pose and colors to the front view." },
      { "type": "text", "bbox": [350, 565, 430, 590], "text": "3/4 VIEW", "desc": "Small bold black label beneath the 3/4 view." },
      { "type": "obj", "bbox": [520, 250, 740, 560], "desc": "Kip full body, SIDE view, 90 degree profile, same rust-orange fur, cream chest, goggles pushed up on forehead, forest-green satchel with brass buckle, identical pose and colors." },
      { "type": "text", "bbox": [590, 565, 670, 590], "text": "SIDE", "desc": "Small bold black label beneath the side view." },
      { "type": "obj", "bbox": [760, 250, 980, 560], "desc": "Kip full body, BACK view, facing directly away from camera, same rust-orange fur, satchel strap visible crossing the back, brass buckle at the hip, identical pose and colors." },
      { "type": "text", "bbox": [830, 565, 910, 590], "text": "BACK", "desc": "Small bold black label beneath the back view." },
      { "type": "text", "bbox": [995, 250, 1030, 560], "text": "HEIGHT\n(inches)\n40\n35\n30\n25\n20\n15\n10\n5\n0", "desc": "Small black sans-serif height-scale ruler with tick marks running alongside the front and 3/4 views." },
      { "type": "text", "bbox": [1035, 250, 1070, 560], "text": "HEIGHT\n(cm)\n100\n90\n80\n70\n60\n50\n40\n30\n20\n10\n0", "desc": "Small black sans-serif height-scale ruler with tick marks running alongside the side and back views." },
      { "type": "obj", "bbox": [40, 620, 280, 860], "desc": "FRONT FACE close-up: Kip's face showing round amber goggles pushed up on the forehead, cream muzzle, alert expression, tall ears in frame." },
      { "type": "text", "bbox": [110, 865, 210, 890], "text": "FRONT FACE", "desc": "Small bold black label beneath the front-face close-up." },
      { "type": "obj", "bbox": [330, 620, 570, 860], "desc": "SATCHEL DETAIL close-up: the forest-green canvas messenger satchel strap and body, brass buckle construction shown clearly, stitched canvas texture." },
      { "type": "text", "bbox": [390, 865, 510, 890], "text": "SATCHEL DETAIL", "desc": "Small bold black label beneath the satchel-detail close-up." },
      { "type": "obj", "bbox": [620, 620, 860, 860], "desc": "WAVING HAND DETAIL close-up: Kip's rust-orange paw mid-wave, simple rounded finger shapes, cream inner-palm patch." },
      { "type": "text", "bbox": [680, 865, 800, 890], "text": "WAVING HAND DETAIL", "desc": "Small bold black label beneath the waving-hand close-up." }
    ]
  }
}
```

## What to take from this example

- Kip's silhouette, colors, goggles, and satchel are described once in the prompt and
  then repeated identically across all four turnaround-view `desc` entries and both
  head-and-detail close-ups — the annotation makes the "held identical across every
  panel" consistency directive checkable panel by panel, the same on-model discipline
  `bell-boy-character-sheet.md` demonstrates.
- Both height-scale rulers (inches alongside front/3-4, cm alongside side/back) are
  captured as their own `text` elements, matching the proportion-lock convention this
  skill's `sheet-anatomy.md` calls for.
- No accessories/interior mechanical diagram panel is included, because Kip is an
  organic mascot with no gadget internals to diagram — this skill's optional
  accessories/interior panel is correctly reserved for mechanical/gadget characters
  only, and this example demonstrates leaving it out rather than forcing one in.
- This sheet was authored directly from the known prompt rather than run back through
  `describe_image`, the correct shortcut whenever the composition was already fully
  specified before generation.
