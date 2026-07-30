# Ad-creative composition specification format

This file instantiates the ad-creative element vocabulary from the shared schema documented in `ideogram-prompt/references/json-caption-schema.md`. It uses the same field names, key order, and `bbox` convention (`[y_min, x_min, y_max, x_max]`, normalized 0–1000, origin top-left) without modification — it does not redefine the schema, only names the specific element set an ad composition draws from.

## Ad-creative element set

An ad composition consists of up to six distinct element roles, each with a fixed type and semantic purpose:

- **`background/scene`** — `type: "obj"`. The scene, setting, or ground layer the composition sits on: studio background, location, flat color, gradient, textured surface, environmental backdrop. Required in most compositions; omit only for minimalist text-only layouts.

- **`product/subject`** — `type: "obj"`. The product shot, subject, or hero visual; the dominant foreground object or character. Present for product ads, service visualizations, lifestyle imagery. Omit entirely if the ad is copy-only (e.g., a pure announcement graphic with no product shot).

- **`headline`** — `type: "text"`. One `text` element per line (following the shared schema's "one text element per line" rule), rendering the dominant message. Renders in the size and weight hierarchy that reads first.

- **`subhead/tagline`** — `type: "text"`. Secondary supporting copy directly beneath the headline, present only if literal copy was supplied for it in the creative brief. Same element type; subordinate weight/size.

- **`CTA`** (call-to-action) — `type: "text"`. The action string (e.g., "Shop Now," "Learn More," "Sign Up"). Rendered as its own element so its `bbox` can be placed in a platform's safe, unobstructed zone clear of UI chrome (profile photo overlay, caption bar, sponsored-label area).

- **`brand-mark`** — `type: "obj"` (or `type: "text"` if the brand mark is a wordmark/logotype). Present only when the brand identity does not require pixel-exact logo file reproduction (per the pixel-exact-logo limitation in `typography-baking-discipline.md`). Omit if an existing logo file needs to be placed without modification.

## Placing bbox inside platform safe zones

Each text element's `bbox` must be placed inside the platform's UI-chrome safe zone documented per-row in `platform-dimension-map.md`. This differs from sibling skills' bbox placement in *purpose*: a moodboard or identity-sheet panel layout only needs to look good, but an ad's text `bbox` must also survive being partially covered by the platform's own UI chrome (Instagram's caption overlay, TikTok's bottom info bar, LinkedIn's profile widget). If a text element's `bbox` overlaps the platform's reserved chrome zone, the text will be obscured or wrapped in unexpected ways in production.

## Full worked example

The following JSON caption represents a complete single-placement Instagram feed ad for a fictional product, demonstrating use of five of the six element roles (brand-mark omitted in this example) and strict adherence to the shared schema's field names, key order, and `bbox` convention.

```json
{
  "high_level_description": "A square Instagram feed ad for Fizzwright's noise-cancelling desk headset, announcing a new colorway with a clear 'Shop Now' call to action.",
  "style_description": {
    "aesthetics": "clean, confident, product-forward",
    "lighting": "soft studio lighting, single key light with a soft fill",
    "photo": "50mm, f/2.8, shallow depth of field",
    "medium": "photograph",
    "color_palette": ["#0B1F3A", "#F4F1EA", "#FF6A3D"]
  },
  "compositional_deconstruction": {
    "background": "A solid deep navy (#0B1F3A) studio background with a soft radial vignette, no texture or gradient wash.",
    "elements": [
      {
        "type": "obj",
        "bbox": [280, 150, 780, 850],
        "desc": "The Fizzwright headset in the new terracotta colorway, three-quarter angle, resting on a low pedestal, catching a soft rim light along the headband.",
        "color_palette": ["#FF6A3D", "#0B1F3A"]
      },
      {
        "type": "text",
        "bbox": [60, 60, 160, 700],
        "text": "MEET TERRACOTTA",
        "desc": "Bold uppercase sans-serif headline, warm off-white (#F4F1EA), left-aligned in the top safe zone.",
        "color_palette": ["#F4F1EA"]
      },
      {
        "type": "text",
        "bbox": [170, 60, 230, 600],
        "text": "The new Fizzwright colorway, in stores now.",
        "desc": "Smaller regular-weight sans-serif subhead directly beneath the headline, same off-white, lower opacity than the headline for clear hierarchy.",
        "color_palette": ["#F4F1EA"]
      },
      {
        "type": "text",
        "bbox": [860, 350, 940, 650],
        "text": "SHOP NOW",
        "desc": "A solid terracotta (#FF6A3D) pill-shaped button with bold off-white uppercase text centered inside it, placed in the bottom safe zone clear of Instagram's caption-overlay area.",
        "color_palette": ["#FF6A3D", "#F4F1EA"]
      }
    ]
  }
}
```

See `examples/` for full worked examples as real jobs are run.
