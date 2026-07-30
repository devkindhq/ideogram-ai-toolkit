# Compositional Deconstruction Format

This is the structured JSON breakdown of a marketing-UI mockup — the artifact that
makes a generated (or reference) image reusable as a spec, not just a picture to look
at. Think of it as "what a vision model would say if asked to precisely annotate every
panel of this image" — a `high_level_description` one-liner, then an `elements` array
where every layer, button, and piece of text gets its own bounding box and
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

## Layer tagging

Every element's `desc` field should note which of `references/mockup-anatomy.md`'s
six compositional layers it belongs to — Nav Bar, Hero Copy Zone, CTA Placement,
Device Frame/Chrome, Supporting UI Chrome, Background/Environment — as the first
phrase of the description, e.g. `"desc": "Nav Bar — a slim horizontal bar with three
real nav-item labels..."`. This is new content specific to this skill, not part of the
schema itself, and it exists so a downstream reader can find "the CTA element" or "the
nav bar element" in the `elements` array without re-deriving which layer an
undifferentiated description belongs to.

## Worked example

A landing-page hero for the fictional brand "Technauts," broken into nav-bar,
hero-copy, CTA, and product-visual elements, using the layer-tagging convention above:

```json
{
  "high_level_description": "A landing-page hero image for Technauts showing a browser-chrome mockup of a satellite telemetry dashboard, with a headline, subheadline, and CTA button above it.",
  "compositional_deconstruction": {
    "background": "A deep ink-navy background with a faint diagonal grid line pattern, evoking a technical blueprint aesthetic consistent with the Technauts brand.",
    "elements": [
      {
        "type": "text",
        "bbox": [120, 80, 880, 180],
        "text": "Ship telemetry that trusts itself",
        "desc": "Hero Copy Zone — large bold white sans-serif headline, centered, tight letter-spacing, the dominant text element in the composition."
      },
      {
        "type": "text",
        "bbox": [180, 190, 820, 240],
        "text": "Real-time satellite health, without the guesswork",
        "desc": "Hero Copy Zone — subheadline directly below the headline, smaller weight, warm-gray color, centered."
      },
      {
        "type": "obj",
        "bbox": [420, 260, 580, 310],
        "desc": "CTA Placement — a cyan pill-shaped button directly below the subheadline, containing the label 'Start free trial' in bold dark-navy text."
      },
      {
        "type": "obj",
        "bbox": [100, 340, 900, 700],
        "desc": "Device Frame/Chrome — a browser-chrome window frame (address bar, three window-control dots) containing the Product Visual."
      },
      {
        "type": "obj",
        "bbox": [130, 400, 250, 460],
        "desc": "Nav Bar — a slim horizontal bar inside the browser frame with three real nav-item labels: 'Dashboard', 'Fleet', 'Alerts', in light-gray sans-serif text."
      },
      {
        "type": "obj",
        "bbox": [130, 480, 870, 680],
        "desc": "Product Visual — a stylized satellite-telemetry dashboard screen with a sidebar, three stat cards showing signal-strength percentages, and a line chart labeled 'Orbit Health' trending upward."
      }
    ]
  }
}
```
