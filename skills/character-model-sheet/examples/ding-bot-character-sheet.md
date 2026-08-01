# Worked Example — Ding-Bot Character Model Sheet

A real reference-image reverse-engineering example this skill was seeded from. This is
a full compositional deconstruction (not a generation prompt paired with its output) —
useful as the target level of detail when annotating any character sheet, generated or
supplied as a reference.

Coordinate space: pixel-estimated from a ~1000x1000 source image,
`[x_min, y_min, x_max, y_max]`.

```json
{
  "high_level_description": "A comprehensive character design sheet for Ding-Bot, a 3D orange bell-shaped AI assistant character with light blue headphones and a glowing Wi-Fi emblem. The sheet includes a full body turnaround, head and detail views, color palettes, and a technical diagram of the internal components and accessories.",
  "compositional_deconstruction": {
    "background": "A multi-panel character design sheet with a clean, white background and light gray grid lines. The layout is organized into sections with thin black borders separating different areas of the document.",
    "elements": [
      { "type": "obj", "bbox": [81, 226, 307, 461], "desc": "Ding-Bot character in a full-body front view. An orange bell-shaped character with a smiling face, large black eyes, and a small mouth. It wears light blue over-ear headphones with a microphone extending from the right ear. A glowing white Wi-Fi symbol is on its chest. It has small orange arms and legs with dark blue shoes." },
      { "type": "obj", "bbox": [81, 501, 307, 693], "desc": "Ding-Bot character in a full-body 3/4 view from the right. The character is oriented towards the left, showing its right side profile. It maintains the same orange bell shape, smiling face, and light blue headphones with a microphone. The glowing white Wi-Fi symbol is visible on its chest." },
      { "type": "obj", "bbox": [394, 245, 626, 449], "desc": "Ding-Bot character in a full-body side view. The character is facing left, showing its profile. The orange bell shape, smiling face, and light blue headphones are clearly visible. The glowing white Wi-Fi symbol is on its chest." },
      { "type": "obj", "bbox": [394, 492, 626, 719], "desc": "Ding-Bot character in a full-body back view. The character is facing away from the viewer, showing the rear of the orange bell shape and the back of the light blue headphones. The glowing white Wi-Fi symbol is on its chest." },
      { "type": "obj", "bbox": [36, 775, 248, 984], "desc": "Close-up front face of Ding-Bot. The character has a large, friendly smile, large black eyes with white highlights, and thick black eyebrows. The light blue headphones and microphone are visible on its head." },
      { "type": "obj", "bbox": [273, 775, 483, 984], "desc": "Close-up side-view headset of Ding-Bot. It shows the light blue over-ear headphones, the microphone extending from the right ear, and the orange bell-shaped head." },
      { "type": "obj", "bbox": [534, 800, 720, 984], "desc": "Close-up of Ding-Bot's waving hand. The hand is orange with three fingers extended and the thumb and pinky curled, showing a simple, rounded design." },
      { "type": "obj", "bbox": [719, 45, 960, 339], "desc": "Detailed view of the light blue over-ear headphones. The headphones are shown from a front perspective, highlighting the smooth, rounded design and the light blue color." },
      { "type": "obj", "bbox": [709, 458, 971, 666], "desc": "Cross-section diagram of the Ding-Bot character's interior. It shows the internal components including a blue circuit board with a central chip, an 'Emission core & symbol projection mechanism' at the bottom, and a 'Wireless receiver unit' on the right side." },
      { "type": "text", "bbox": [15, 15, 65, 172], "text": "DING-BOT", "desc": "Large, bold, black sans-serif headline at the top left." },
      { "type": "text", "bbox": [51, 15, 101, 178], "text": "Character Model Sheet", "desc": "Medium-sized, black sans-serif sub-headline below the main title." },
      { "type": "text", "bbox": [78, 15, 150, 196], "text": "CHARACTER OVERVIEW:\nName: Ding-Bot\nAlias: Chime-E\nRole: Communication AI Assistant\nArchtype: Cheerful Guide.", "desc": "Small, black sans-serif text block in a light blue box on the left side." },
      { "type": "text", "bbox": [168, 15, 240, 211], "text": "PERSONALITY & TRAITS\nHelpful, Enthusiastic, High-Pitched Voice\nCore Theme: Support & Connection\nBehavior: Constantly monitoring\nsignals, quick to respond", "desc": "Small, black sans-serif text block in a light blue box on the left side." },
      { "type": "text", "bbox": [258, 15, 308, 196], "text": "WARDROBE / ACCESSORIES\nIntegrated Headphones (light blue),\nInternal Wi-Fi Emblem (glowing)", "desc": "Small, black sans-serif text block in a light blue box on the left side." },
      { "type": "text", "bbox": [326, 15, 448, 206], "text": "COLOR PALETTE\nPrimary: Orange (Bell).\nSecondary: Light Blue (Headphones,\ninternal details).\nAccent: Gold/Brass (Base metal),\nDark Blue (Clapper base),\nWhite (Emblem, teeth).", "desc": "Small, black sans-serif text block in a light blue box on the left side." },
      { "type": "text", "bbox": [471, 15, 521, 120], "text": "COLOR PALETTE\nPRIMARY:", "desc": "Small, bold, black sans-serif text above a color swatch bar." },
      { "type": "obj", "bbox": [516, 15, 566, 159], "desc": "Color swatch bar showing orange, light blue, and dark blue." },
      { "type": "text", "bbox": [558, 15, 608, 72], "text": "ACCENT:", "desc": "Small, bold, black sans-serif text above a color swatch bar." },
      { "type": "obj", "bbox": [581, 15, 631, 159], "desc": "Color swatch bar showing dark blue, gold, and white." },
      { "type": "text", "bbox": [15, 401, 65, 584], "text": "FULL BODY TURNAROUND", "desc": "Medium-sized, bold, black sans-serif headline at the top center." },
      { "type": "text", "bbox": [334, 321, 384, 371], "text": "FRONT", "desc": "Small, bold, black sans-serif label below the front character view." },
      { "type": "text", "bbox": [334, 576, 384, 638], "text": "3/4 VIEW", "desc": "Small, bold, black sans-serif label below the 3/4 character view." },
      { "type": "text", "bbox": [643, 331, 693, 381], "text": "SIDE", "desc": "Small, bold, black sans-serif label below the side character view." },
      { "type": "text", "bbox": [643, 591, 693, 641], "text": "BACK", "desc": "Small, bold, black sans-serif label below the back character view." },
      { "type": "text", "bbox": [15, 804, 65, 959], "text": "HEAD & DETAIL SHEET", "desc": "Medium-sized, bold, black sans-serif headline at the top right." },
      { "type": "text", "bbox": [250, 836, 300, 921], "text": "FRONT FACE", "desc": "Small, bold, black sans-serif label below the front face view." },
      { "type": "text", "bbox": [486, 811, 536, 946], "text": "SIDE-VIEW HEADSET", "desc": "Small, bold, black sans-serif label below the side headset view." },
      { "type": "text", "bbox": [725, 804, 775, 954], "text": "WAVING HAND DETAIL", "desc": "Small, bold, black sans-serif label below the waving hand view." },
      { "type": "text", "bbox": [950, 792, 1000, 966], "text": "BASE & EMISSION SYMBOL", "desc": "Small, bold, black sans-serif label below the base and symbol view." },
      { "type": "text", "bbox": [679, 305, 729, 488], "text": "ACCESSORIES & INTERIOR", "desc": "Medium-sized, bold, black sans-serif headline in the bottom section." },
      { "type": "text", "bbox": [734, 375, 784, 442], "text": "Integrated\nheadphone\nstructure", "desc": "Small, black sans-serif label with a line pointing to the headphone diagram." },
      { "type": "text", "bbox": [798, 363, 852, 447], "text": "Emission core\n& symbol\nprojection\nmechanism", "desc": "Small, black sans-serif label with a line pointing to the internal mechanism." },
      { "type": "text", "bbox": [816, 663, 866, 740], "text": "Wireless\nreceiver unit", "desc": "Small, black sans-serif label with a line pointing to the internal receiver unit." },
      { "type": "text", "bbox": [31, 715, 310, 765], "text": "HEIGHT\ninches\n120\n110\n100\n90\n80\n70\n60\n50\n40\n30\n20\n10\n0", "desc": "Small, black sans-serif height scale on the right side of the front and 3/4 views." },
      { "type": "text", "bbox": [371, 715, 626, 765], "text": "HEIGHT\n(in cm)\n170\n160\n150\n140\n130\n120\n110\n100\n90\n80\n70\n60\n50\n40\n30\n20\n10\n0", "desc": "Small, black sans-serif height scale on the right side of the side and back views." }
    ]
  }
}
```

## What to take from this example

- The header/overview block carries four distinct text boxes (character overview,
  personality & traits, wardrobe/accessories, color palette copy) plus two swatch bars
  (primary, accent) — that's the "spec sheet" framing `sheet-anatomy.md` describes, and
  it's what keeps this reading as a production document rather than a portrait.
- The full-body turnaround holds all four views (front, 3/4, side, back) to the same
  silhouette, pose, and exact colors — orange bell body, light-blue headphones, white
  glowing Wi-Fi emblem — repeated identically in every `desc`. That repetition in the
  annotation is a sign the source image actually held the character on-model; if you're
  annotating a generated sheet and find yourself writing different colors or
  proportions per view, that's the off-model drift `anti-slop-discipline.md` flags.
- Both height-scale rulers (inches alongside the front/3-4 views, cm alongside the
  side/back views) are captured as their own text elements — a detail worth preserving
  since it's what makes the sheet a proportion-lock reference, not just four pretty
  poses.
- The accessories & interior panel uses three separate labeled leader-line callouts
  (headphone structure, emission core, wireless receiver) rather than one generic
  "internals" caption — that level of specificity is what `sheet-anatomy.md`'s
  accessories/interior panel description calls for, and only makes sense because
  Ding-Bot is a gadget-driven character with real internal mechanisms to diagram.
