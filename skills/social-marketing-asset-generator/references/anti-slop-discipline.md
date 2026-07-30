# Anti-Slop Discipline for Social Marketing Asset Generator

This skill adapts the universal anti-slop bans shared with `brand-identity-sheet`, `logo-prompting`, `moodboard-generator`, and `character-model-sheet`, plus ad-specific clichés from the marketing-creative register that are distinct from brand-identity work. It adds one failure mode unique to this skill: garbled or illegible rendered text, Ideogram's known weakness on dense in-image copy, which must be caught at the legibility-review step and regenerated against rather than accepted as a finished asset.

This file is the gate you run **before** calling `generate_image`, not a style suggestion to eyeball after. A composed JSON caption that fails the pre-generation gate gets revised and re-scored, not sent as-is.

## Universal bans (same list as the sibling skills)

Unless the brand truth explicitly calls for one of these as a deliberate, named exception, no ad composition should include:

- **Neural-network nodes / circuit-brain diagrams** as a stand-in for "modern" or "tech-forward" when the brand hasn't actually asked for that register.
- **A glowing orb or floating sphere as a hero element** in the background/scene or product/subject layers — a contained glow tied to an actual brand reference (bioluminescence, a specific light quality named in the creative brief) is fine; an unmotivated generic AI-glow is not.
- **Headset / phone-receiver icon** in the background or as a visual accent for "support" or "contact" unless the brand's product actually is a headset, phone, or communication device.
- **Padlock / shield** in the background or as a visual accent for "security" unless the product is genuinely a security product or service.
- **Lightbulb** anywhere for "ideas" or "innovation."
- **Soundwave bars / abstract waveform** as generic "tech" filler with no audio product behind it.
- **Circuit-board texture** in the background/scene layer as a stand-in for "technical" when the brand brief already names something more specific.
- **A purple-to-pink or cyan-to-magenta gradient wash** across the background or as a full overlay — the single most recognizable "AI startup" tell, and the fastest way an ad collapses into looking like every other AI-tool ad instead of this brand's.
- **Photorealistic stock-photo clichés** in the background or product/subject elements (generic smiling-team stock imagery, overly posed "lifestyle" shots) unless the brand's actual references explicitly call for that register.

## Ad-specific bans

These additions apply only to marketing creative and are distinct from the brand-identity register:

- **No generic "SALE!" starburst or sunburst badge.** A badge with bursting rays and an exclamation mark reads as generic stock-ad clip art rather than this brand's own promotional treatment. If a sale or offer needs visual emphasis, use the brand's own color, typography hierarchy, or a deliberate, named visual mark instead.
- **No stock-photo hands-on-laptop.** The single most recognizable generic-SaaS-ad tell, on par with the gradient wash for identity work. If the composition calls for a lifestyle or workstation shot, either source it from the brand's own reference imagery or describe a genuine, specific setting (a real product being used, not a generic "person at desk" pose).
- **No fake star-rating clutter.** Five golden stars with no real review data behind them is a fabricated claim in pixel form. This skill's literal-copy-only rule already forbids invented claims and pricing; call it out explicitly here since star ratings are a visual element, not a text string, and easy to miss in a copy-only review.
- **No arrow pointing at the CTA** unless the brand's own reference imagery or design system explicitly calls for it as a deliberate, named visual cue. An arrow is a crutch that substitutes for actual visual hierarchy — headline size, color, placement, contrast — doing the real work of guiding attention.
- **No illegible or gibberish rendered text.** Ideogram's known failure mode on dense in-image copy is garbled or unreadable text. This ban is caught at the legibility-review workflow step (not the pre-generation gate, since garbling can only be seen post-render), but it is named here as the failure mode it is so regeneration attempts happen immediately rather than accepting a broken render.
- **No low-contrast text-on-busy-background that would fail at thumbnail or feed-scroll size.** Most social placements are viewed at a fraction of full resolution — a headline that's legible in a full-size preview but vanishes when scrolled past at 1/4 size on mobile is a real failure, not a nitpick. Every text element's `bbox` and color pairing must survive the actual viewing context it will appear in.

## Pre-generation gate — run this before calling `generate_image`

Score the drafted JSON caption 1-5 on each axis below. Any axis scoring under 3 means revise the caption before generating. Do not skip this gate because the brief seemed clear or the copy is already locked — even locked copy can be placed in a clichéd or illegible context.

| Axis | What you're checking |
|---|---|
| **Cliché avoidance** | Does the drafted caption contain any of the Universal or Ad-specific bans listed above? Read the `background`, `product/subject`, and all `text` element descriptions sentence by sentence against both lists. Any banned noun present without an explicit, named exception from the brand brief? Score under 3. |
| **Message hierarchy** | Does the caption enforce exactly one dominant text element (the `headline`) with `subhead/tagline` and `CTA` clearly subordinate in size, weight, and placement, per `typography-baking-discipline.md`'s one-message-hierarchy-per-asset rule? A caption with two competing headlines or a CTA that's as prominent as the headline scores under 3. |
| **Copy fidelity** | Does every `text` element's `text` field match a literal, user-supplied copy string exactly, with no invented words, punctuation, offers, prices, or claims? A caption that writes "Save 50%" when the user said "on sale" scores under 3. |
| **Safe-zone placement** | Is every text element's `bbox` inside the resolved platform's safe zone per `platform-dimension-map.md`, not overlapping profile-photo areas, caption-bar overlays, sponsored-label zones, or other platform UI chrome? A `headline` bbox that extends into Instagram's caption-bar area scores under 3. |

Revise the caption against whichever axis(es) scored under 3, then re-score all four axes. Only send to `generate_image` when all four axes score 3 or higher.
