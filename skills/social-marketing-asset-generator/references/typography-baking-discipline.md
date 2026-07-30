# Typography Baking Discipline

This file specifies the legibility rules, message-hierarchy rules, and tool-selection decision logic that the Legibility Review and Touch-up Loop workflow steps in `SKILL.md` enforce during asset composition and refinement.

## Why baked-in beats generate-then-composite

Two competing strategies for ad-creative text rendering exist in the field:

1. **Composite-afterward approach** (as seen in `wachawo/claude-skills` `banner-design`): Generate a text-free art composition, then layer headline and CTA typography on top afterward via HTML/CSS and a Chrome DevTools screenshot export. The advantage is pixel-perfect, guaranteed text rendering. The disadvantage is a second rendering pipeline: the compositor must hold both tools (an image generator and a web renderer), coordinate two separate outputs, and manage the interface between them.

2. **Baked-in text approach** (this skill): Embed typography directly into the diffusion model's input via the structured JSON caption from `composition-spec-format.md`, letting Ideogram render both the scene and the text together in one call. This leans into Ideogram's specific strength: reliable, on-brand in-image text rendering with full control over placement, size, weight, and color via the `bbox` and `color_palette` fields.

The tradeoff is honest: baked-in text is not guaranteed letter-perfect on every render. Ideogram's known weakness on dense in-image copy is garbled or unreadable text. **But a garbled baked-in render is a known, catchable failure mode** with a clear fix: regenerate the image against the same caption and surface the failure immediately rather than accepting it. The composite-afterward approach sidesteps the problem by outsourcing text rendering to a different tool — it does not solve the problem, and it costs an extra rendering pipeline this repo does not need.

This skill catches baked-in text failures at the **legibility-review** step (not the pre-generation gate, since garbling can only be seen post-render) and regenerates immediately. Failures are surfaced, not hidden.

## Legibility rules

Every rendered asset must satisfy four concrete, independently checkable rules before it can be delivered.

### Rule 1: Max words per line

No more than 6-8 words on a single headline line. Longer copy gets split into multiple `text` elements, one per line (following the shared schema's "one text element per line" rule from `composition-spec-format.md`), not squeezed into a single element with an embedded line break. A long headline like "Introducing the new Fizzwright noise-cancelling desk headset in terracotta" should be split across two or three `text` elements with separate `bbox` coordinates:

```json
[
  { "type": "text", "text": "INTRODUCING", "bbox": [...] },
  { "type": "text", "text": "TERRACOTTA", "bbox": [...] }
]
```

Split headlines remain readable at thumbnail/feed-scroll scale. Squeezed single-element text does not.

### Rule 2: Text size relative to canvas

A headline `bbox` height should occupy roughly **8-15% of the canvas height** to remain legible at feed-scroll and thumbnail magnification (the actual viewing context for these assets). If a headline renders at 20%+ of canvas height, it dominates the composition and leaves no room for the product/subject or background. If it renders at under 6%, it vanishes at scroll scale.

A subhead or CTA `bbox` must be **visibly smaller than the headline `bbox`**, never equal or larger. A subhead that's the same rendered height as the headline fails to subordinate the message. The fix is shrinking the subhead's `bbox` height, not increasing the headline's size.

### Rule 3: Contrast-checked color pairing

Every text element's `color_palette` must contrast clearly against whatever is directly behind that element's `bbox`. This is not a general contrast-ratio rule (the WCAG formula for web accessibility); it is a legibility rule for social placements viewed at variable magnification and on small screens.

If a text element sits on top of a solid background (e.g., a deep navy background with white headline text), the contrast is straightforward: light text on dark background or dark text on light background.

If a text element sits on top of a busy background (a product photo, a textured surface, a gradient), the contrast check is stricter. Per the busy-background ban in `anti-slop-discipline.md`, low-contrast text on busy backgrounds fails at thumbnail scale. The fix is **either** move the text element's `bbox` to a cleaner, less busy area of the canvas, **or** add a scrim or semi-opaque panel `obj` element behind the text to create a clean backdrop for the text element to sit on.

### Rule 4: No more than three text elements competing for attention

A headline, one subhead/tagline, and one CTA is the ceiling. A fourth competing text block (e.g., a second CTA, a disclaimer line rendered at the same visual weight as the headline) violates the one-message-hierarchy rule detailed below. Remove the competing fourth element or demote it to a supporting annotation that doesn't claim visual space equal to the headline.

## One-message-hierarchy-per-asset rule

Exactly one dominant message per asset, with two subordinate supporting messages.

- **Dominant (primary)**: The headline. Renders first visually. Largest size, highest contrast, most prominent placement.
- **Secondary**: The subhead or tagline (if present). Renders second. Visibly smaller than headline, lower contrast or less saturated, positioned directly beneath the headline.
- **Tertiary**: The CTA (call-to-action). Renders third. Smallest of the three, distinct visual treatment (often a button or contrasting shape), positioned in the platform's safe zone per `platform-dimension-map.md`.

This hierarchy is the "Message hierarchy" axis checked by the pre-generation gate in `anti-slop-discipline.md`. It is enforced via the `bbox` height and `color_palette` fields in the JSON caption: a headline element and a subhead element rendered at the same pixel height and the same color with no size or weight difference reads as **two competing headlines**, not a headline-and-support pair. The fix is not to reword either string; the fix is to shrink the subhead's `bbox` and/or lower its color contrast (e.g., via opacity, desaturation, or a darker shade of the headline color).

## `edit_image`-vs-regenerate decision rule

The `edit_image` tool has a strict contract: it edits **this exact image**. It is not a remix tool, not a "generate something similar" tool, and not a fallback to `generate_image`. If `edit_image` fails or cannot safely proceed with a requested change, the failure must be surfaced verbatim to the user rather than silently falling back to `generate_image` or `remix_image`.

Use this decision table to route touch-up requests:

| Requested change | Route |
|---|---|
| Recolor the CTA button | `edit_image`, localized touch-up |
| Swap in a different brand-mark asset | `edit_image`, localized touch-up |
| Fix one word of copy in an otherwise-correct render | `edit_image`, localized touch-up (with the caveat that diffusion-based edits aren't guaranteed surgical text replacement — verify the result before delivering it) |
| Change the entire headline | Fresh `generate_image` from a corrected JSON caption, not `edit_image` |
| Change the overall layout or composition | Fresh `generate_image` from a corrected JSON caption, not `edit_image` |
| Change to a different platform placement or aspect ratio | Fresh `generate_image` (or `generate_images_bulk`) with the new resolved `aspect_ratio`, not `edit_image` |

**When a request lands in the "fresh `generate_image`" row**, the skill states this explicitly to the user rather than silently switching tools or attempting the change anyway. This matches the Error handling section of the design spec and prevents the user from being surprised by a regenerated asset when they asked for a quick touch-up.

## Pixel-exact existing logo limitation

A diffusion model cannot guarantee pixel-fidelity reproduction of an existing logo file the way a compositor can. If a brand's logo must appear exactly as-is — a legal trademark lockup requirement, a specific vector artwork, a proprietary symbol that cannot be redrawn — that is a limitation this skill's generation step cannot satisfy.

When the user requests an exact-logo placement, the skill should say so explicitly: "The diffusion model cannot guarantee pixel-fidelity reproduction of your exact logo. I recommend you composite your logo file into the final asset afterward with your own tooling (e.g., Figma, Photoshop, or your web compositor)." Attempt to regenerate the image without the logo, or with a text-based or simplified alternative if the brief allows, rather than silently attempting a diffusion-based logo reproduction and delivering an approximation as if it were exact.

## Self-review pass

Confirm that the `edit_image`-vs-regenerate decision table in the section above covers every example named in the design spec's Assumption 4 and Error handling sections:

- Recolor CTA (Assumption 4, Error handling): routed to `edit_image` ✓
- Swap logo (Assumption 4, Error handling): routed to `edit_image` ✓
- One-word copy fix (Assumption 4, Error handling): routed to `edit_image` ✓
- Full headline change: routed to fresh `generate_image` ✓
- Layout change: routed to fresh `generate_image` ✓
- Platform/aspect-ratio change: routed to fresh `generate_image` ✓

All examples covered, none omitted or contradicted. Self-review passes.
