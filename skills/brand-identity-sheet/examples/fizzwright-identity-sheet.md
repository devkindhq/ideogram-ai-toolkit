# Worked Example — Fizzwright Identity Sheet

![Fizzwright brand identity sheet](images/fizzwright-identity-sheet.webp)

A from-scratch fictional brand invented to demonstrate this skill end to end: a
craft-soda brand called "Fizzwright," unrelated to any client or real product. Both
artifacts below were produced through the real workflow — the paragraph prompt was
submitted to Ideogram (`generate_image`, `style_type: "DESIGN"`, `aspect_ratio: "1x1"`,
`rendering_speed: "QUALITY"`), and the compositional deconstruction below was authored
directly from that same prompt (per this skill's "you already know the exact panel
layout you asked for" guidance), since the composition was fully controlled at write
time.

## 1. The paragraph prompt (what generated the sheet)

```
A brand identity guide / UI style sheet layout for a craft-soda brand called
"Fizzwright," displayed in a clean structured grid over a warm cream background with
thin blueprint grid lines. The composition includes: a white rounded UI wordmark card
with a small "FIZZWRIGHT" label at top, the "Fizzwright" wordmark centered in a bold
rounded slab-serif, and a tiny "Small-batch soda co." tag at the bottom; a grid of six
rounded-corner app icons — one glossy glass-bottle icon glowing with condensation
droplets, one citrus-peel texture-weave icon, one brushed brass bottlecap icon, one
fizzing-bubble organic pattern icon, one cherry-and-stem mascot icon, one vintage-ticket
paper-grain icon; three stacked pill-shaped buttons in cherry red, bottle green, and
cream-neutral; a giant central "fizzwright" wordmark at hero scale in the same bold
rounded slab-serif; and a secondary lockup pairing a smaller "fizzwright" wordmark with
a small cherry-with-stem glyph. Primary color is cherry red #C0392B, secondary/base is
warm cream #F5EDE0, accent is deep bottle green #2F5233 reserved for one button and one
icon only. No gradients bleeding across the whole composition, no neural-network or
circuit-board motifs, no glowing orb as hero icon, no generic soda-can stock
photography — keep every texture a deliberate material sample.
```

Note what this prompt does right by this skill's own discipline: it names an exact
3-color palette with a usage rule (cherry red dominant, cream the base/ground, bottle
green reserved for exactly one button and one icon — never decorative elsewhere), gives
each of the six app icons its own named physical material (glass, citrus-peel weave,
brushed brass, fizzing-bubble pattern, mascot, paper-grain) rather than six generic
icon tiles, and closes with an explicit anti-slop exclusion list scoped to this brand's
specific risk (soda-can stock photography, gradient wash, glowing orb).

## 2. The compositional deconstruction (authored from the known prompt)

Coordinate space: pixel-estimated for a 1024x1024 square canvas,
`[x_min, y_min, x_max, y_max]`.

```json
{
  "high_level_description": "A brand identity guide / UI style sheet for 'Fizzwright,' a craft-soda brand, showing a cherry-red-and-cream wordmark card, a six-icon grid of distinct physical material samples, three pill-shaped buttons, a hero-scale central wordmark, and a secondary wordmark-plus-glyph lockup, arranged in a clean structured grid over a warm cream blueprint-grid background.",
  "compositional_deconstruction": {
    "background": "A warm cream #F5EDE0 ground overlaid with a faint, thin blueprint-style grid of light lines, dividing the composition into a left column of UI cards and icon tiles, a dominant central wordmark zone, and a smaller secondary-lockup panel.",
    "elements": [
      { "type": "obj", "bbox": [40, 40, 340, 300], "desc": "A white rounded-corner UI wordmark card sitting on the cream ground, clean drop-shadow-free surface, generous internal padding." },
      { "type": "text", "bbox": [60, 60, 320, 90], "text": "FIZZWRIGHT", "desc": "Small uppercase label in cherry red #C0392B, condensed sans-serif, sitting at the top of the wordmark card." },
      { "type": "text", "bbox": [60, 120, 320, 210], "text": "Fizzwright", "desc": "The 'Fizzwright' wordmark centered in the card, bold rounded slab-serif letterforms, cherry red #C0392B, friendly rounded terminals." },
      { "type": "text", "bbox": [90, 250, 290, 280], "text": "Small-batch soda co.", "desc": "Tiny supporting tagline in muted warm-gray, sitting at the bottom of the wordmark card beneath the wordmark." },
      { "type": "obj", "bbox": [40, 340, 200, 500], "desc": "A rounded-corner square app icon: a glossy glass soda-bottle silhouette in translucent cherry red, rendered with fine condensation droplets and a soft highlight streak — a deliberate glass material sample, no glow bloom around it." },
      { "type": "obj", "bbox": [220, 340, 380, 500], "desc": "A rounded-corner square app icon: a citrus-peel texture-weave pattern in warm orange and cream, tightly cross-hatched pith texture filling the tile edge to edge." },
      { "type": "obj", "bbox": [40, 520, 200, 680], "desc": "A rounded-corner square app icon: a brushed-brass bottlecap surface sample, fine concentric machine-brushed striations, warm metallic gold-brass tone, no gloss highlight exaggeration." },
      { "type": "obj", "bbox": [220, 520, 380, 680], "desc": "A rounded-corner square app icon: a fizzing-bubble organic pattern in cream and pale cherry-pink, small and large circular bubbles clustered unevenly like carbonation caught mid-rise." },
      { "type": "obj", "bbox": [40, 700, 200, 860], "desc": "A rounded-corner square app icon: a flat cherry-with-stem mascot glyph, solid deep bottle green #2F5233 stem and leaf, cherry red #C0392B fruit body, the sheet's single deliberate accent-color icon." },
      { "type": "obj", "bbox": [220, 700, 380, 860], "desc": "A rounded-corner square app icon: a vintage-ticket paper-grain texture sample, aged cream cardstock fiber texture with a faint perforated edge along one side." },
      { "type": "obj", "bbox": [40, 890, 380, 950], "desc": "A fully rounded pill-shaped button filled solid cherry red #C0392B, matte surface, no gradient." },
      { "type": "obj", "bbox": [40, 965, 380, 1025], "desc": "A fully rounded pill-shaped button filled solid deep bottle green #2F5233 — the sheet's one deliberate accent-color button." },
      { "type": "obj", "bbox": [40, 1040, 380, 1100], "desc": "A fully rounded pill-shaped button filled solid warm cream-neutral, a subtle one-shade-darker cream than the background so it still reads as a distinct control." },
      { "type": "text", "bbox": [430, 380, 980, 680], "text": "fizzwright", "desc": "The wordmark at giant hero scale dominating the central zone, lowercase, bold rounded slab-serif, cherry red #C0392B, tight letter spacing, the composition's anchor." },
      { "type": "text", "bbox": [430, 760, 700, 820], "text": "fizzwright", "desc": "A smaller secondary lockup instance of the wordmark, lowercase, same bold rounded slab-serif, cherry red #C0392B." },
      { "type": "obj", "bbox": [720, 760, 780, 820], "desc": "A small solid cherry-with-stem glyph beside the secondary wordmark — cherry red #C0392B fruit, deep bottle green #2F5233 stem — matching the mascot icon's glyph but standalone, no background tile." }
    ]
  }
}
```

## What to take from this example

- Every claim in the five-sentence prompt shows up literally in the elements array and
  nowhere else: six icons, three named-material buttons stacked in a fixed order, one
  wordmark card, one hero wordmark, one secondary lockup. Nothing was invented at
  annotation time that wasn't already locked in the prompt.
- The accent color (bottle green) is deliberately placed on exactly one icon and one
  button in both the prompt and the deconstruction — a live demonstration of the
  "which color is reserved for one state, not decorative everywhere" usage-rule
  discipline from `references/composition-spec-format.md` and `logo-prompting`'s
  Visual-layer reading guidance.
- This sheet was authored directly from the known prompt rather than run back through
  `describe_image` — the right call whenever you already control the exact composition
  you asked for, per this skill's own "From a generated image" guidance.
