# Anti-Slop Discipline for Concept Grids

Reused unmodified from `skills/brand-identity-sheet/references/anti-slop-discipline.md`.
The content below is the same universal-ban list, guardrail-transcription rule, and
pre-generation gate that skill uses — nothing here is concept-grid-specific except
this note. Apply it exactly as written, with one addition already called out in
`SKILL.md` step 3.5: score **each tile individually**, not just the prompt as a whole
— a grid has N times the surface area for a banned cliché that a single mark does.

---

An identity sheet has more surface area for slop than a single logo mark — six to
eight icon panels, a background treatment, a character panel — and every one of them
is a chance for Ideogram to reach for its own AI-startup priors instead of the
brand's. This file is the gate you run **before** calling `generate_image`, not a
style suggestion to eyeball after.

Adapted from Hallmark's slop-test discipline (`hallmark/references/slop-test.md`),
which exists for the same reason on the web-design side: a model's unprompted default
is the generic AI-anything look, and the only fix is naming the specific things to
avoid, in the model's own vocabulary, before it generates.

## Why the brand's own guardrails go missing

The single most common way an identity-sheet prompt still comes out generic even
though the brand doc explicitly lists things to avoid: the writer reads the avoid-list,
mentally absorbs the *intent* ("okay, don't make it look like a generic AI product"),
and then writes the negative-space clause in their own words instead of the brand
doc's words. "No overused tech clichés" is not the same instruction to an image model
as "no neural-network nodes, no glowing AI orb, no headset icon, no purple-to-pink
gradient" — the model can act on nouns it can picture, not on a paraphrase of a vibe.

**The fix is mechanical, not stylistic: if a brand doc (`brand.md`/`design.md`) has an
explicit "avoid" / "anti-slop" / "don't do this" section, copy those specific banned
nouns into the prompt's negative-space clause verbatim.** Don't summarize them, don't
soften them into "no generic elements," don't assume the generic list below already
covers a brand-specific ban the brand doc called out by name. Quote it.

## Universal bans (apply to every sheet, brand-specific list or not)

Unless the brand truth explicitly calls for one of these as a deliberate, named
exception (a security product's mark can be a padlock; a comms app's mark can suggest
a signal), the icon grid, character panel, and background must not include:

- **Neural-network nodes / circuit-brain diagrams** — the default "this is AI" visual,
  reached for regardless of whether the brand is actually AI-adjacent.
- **A glowing orb or floating sphere as the hero icon** — a contained "glow/beacon"
  icon *panel* is a legitimate material sample (see `sheet-anatomy.md`'s app-icon
  grid); a generic glowing ball standing in for "technology" or "intelligence" with no
  other motivation is not the same thing.
- **Headset / phone-receiver icon** for "support," "communication," or "contact" —
  reach for the brand's actual product surface instead, or drop the icon.
- **Padlock / shield** for "security" unless the product genuinely is a security
  product — otherwise it's decoration standing in for a claim, not the claim itself.
- **Lightbulb** for "ideas," "innovation," or "insight."
- **Soundwave bars / abstract waveform** as generic "tech" filler with no audio
  product behind it.
- **Circuit-board texture** as a stand-in for "technical" when the brand's own
  material vocabulary (metallic, woven, brushed, mineral, paper-grain, whatever
  design.md actually names) already gives you something more specific.
- **A purple-to-pink or cyan-to-magenta gradient wash across the whole sheet or
  background** — this is the single most recognizable "AI startup" tell. A brand
  whose own palette calls for a "glow" as one deliberate, contained icon (see
  Technauts' beacon panel) is fine; a gradient bleeding across every panel, or used as
  the sheet's overall background treatment, is not — that's the model defaulting to
  its own priors instead of the brand's locked palette.
- **Photorealistic stock-photo textures** (hands on a laptop, generic office scenes,
  smiling-team stock imagery) — an identity sheet is a designed system, not a
  marketing photo.
- **Startup-cliché placeholder names** (Acme, Nexus, Seamless, Unleash) anywhere text
  is faked in a UI-card panel — use the real brand name, or a plausible in-world
  supporting label (an edition tag, a version number), never a generic filler brand.

## Icon-grid specificity check

Six icons that are all "glowing orb, different color" isn't range, it's repetition —
`sheet-anatomy.md` already says this. The slop version of that failure is subtler:
six icons that are individually varied in *form* (one gradient, one mesh, one metal)
but generic in *material* — a mesh texture that could belong to any brand, a metallic
sphere that isn't tied to anything the brand actually stands for. Before generating,
check each icon in the planned grid against one question: **could this exact icon
description be pasted unchanged into a different brand's prompt and still make
sense?** If yes for more than one or two icons, the material vocabulary isn't doing
its job — go back to the brand doc's style keywords and reference brands and pull
something more specific (not "brushed metal," but the specific alloy/finish/context
the brand's own language points to).

## Pre-generation gate — run this before calling `generate_image`

Score the drafted paragraph prompt 1-5 on each axis below. Anything under 3 on any
axis means revise the prompt before generating — a second draft is cheap, a
regenerated image is not.

| Axis | What you're checking |
|---|---|
| **Brand-specificity** | Could this prompt's icon/material vocabulary be pasted onto a different brand unchanged? If the words don't name this brand's specific style keywords, it's generic. |
| **Guardrail transcription** | If the brand doc has its own avoid-list, does the prompt's negative-space clause quote those specific nouns — not a paraphrase, not "avoid AI clichés" — verbatim? |
| **Universal-ban sweep** | Read the drafted prompt sentence by sentence against the "Universal bans" list above. Is every banned noun absent, or present only as a brand-justified, explicitly-named exception? |
| **Glow containment** | If a gradient/glow appears anywhere, is it scoped to exactly one named icon panel (per `sheet-anatomy.md`), not the background or the whole composition? |
| **Restraint** | Is every panel earning its place, or is there a decorative element with no tie to the brand's actual product or material vocabulary? |

Do not skip this because the brand-truth read (step 1) already named colors and
typography correctly — palette accuracy and slop-freedom are two different failure
modes, and a prompt can nail the former while still defaulting to generic icon
clichés in the latter.
