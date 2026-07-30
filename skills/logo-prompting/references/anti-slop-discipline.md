# Anti-Slop Discipline for Logo Prompts

A single logo mark has less surface area than a moodboard or identity sheet, but
also less room to hide a generic choice — one wrong icon and the whole mark reads
as stock. This file is the gate you run **before** calling `generate_image`, not a
style suggestion to eyeball after.

Adapted from `brand-identity-sheet/references/anti-slop-discipline.md`, which is
itself adapted from Hallmark's slop-test discipline (`hallmark/references/slop-test.md`).
A model's unprompted default is the generic AI-anything look, and the only fix is
naming the specific things to avoid, in the model's own vocabulary, before it
generates.

## Why the brand's own guardrails go missing

The most common way a logo prompt still comes out generic even though the brand doc
lists things to avoid: the writer reads the avoid-list, absorbs the *intent* ("don't
look like a generic AI product"), then writes the negative-space clause in their own
words instead of the brand doc's words. "No overused tech clichés" is not the same
instruction to an image model as "no gradients, no neural-network nodes, no glowing
orb" — the model can act on nouns it can picture, not a paraphrase of a vibe.

**The fix is mechanical: if a brand doc has an explicit avoid-list, copy those banned
nouns into the prompt's negative-space clause verbatim.** Don't summarize them, don't
soften them into "no generic elements."

## Universal bans (apply to every mark, brand-specific list or not)

Unless the brand truth explicitly calls for one of these as a deliberate, named
exception (a security product's mark can be a padlock), the mark must not include:

- **Neural-network nodes / circuit-brain diagrams** as a stand-in for "modern" or
  "tech-forward" when the brand hasn't actually asked for that register.
- **A glowing orb or floating sphere** as the mark itself or a supporting motif.
- **Headset / phone-receiver icon** for "support," "communication," or "contact" —
  reach for the brand's actual product surface instead, or drop the icon.
- **Padlock / shield** for "security" unless the product genuinely is a security
  product.
- **Lightbulb** for "ideas," "innovation," or "insight."
- **Soundwave bars / abstract waveform** as generic "tech" filler with no audio
  product behind it.
- **A purple-to-pink or cyan-to-magenta gradient** anywhere in the mark or its
  background — the single most recognizable "AI startup" tell, and the fastest way
  a logo collapses into looking like every other AI-tool logo instead of this
  brand's.
- **Startup-cliché placeholder names** (Acme, Nexus, Seamless) if a fictional stand-in
  brand name is needed — use a plausible in-world name instead of a generic filler.

## Structural-variety check (multi-direction jobs only)

If the job calls for more than one direction, re-read each prompt against the
"structural variety over palette-swaps" rule in `SKILL.md`: does this direction have
its own form (wordmark / badge / mark-only / monospace-as-identity), or is it the
same form as another direction with only the accent color changed? A palette swap is
not a second direction.

## Pre-generation gate — run this before calling `generate_image`

Score the drafted prompt 1-5 on each axis below. Anything under 3 on any axis means
revise the prompt before generating — a second draft is cheap, a regenerated image is
not.

| Axis | What you're checking |
|---|---|
| **Brand-specificity** | Could this prompt's motif/type/color vocabulary be pasted onto a different brand unchanged? If so, it's generic. |
| **Guardrail transcription** | If the brand doc has its own avoid-list, does the prompt's negative-space clause quote those specific nouns verbatim? |
| **Universal-ban sweep** | Read the drafted prompt sentence by sentence against the "Universal bans" list above. Is every banned noun absent, or present only as a brand-justified, explicitly-named exception? |
| **Motif restraint** | Is the motif described as an abstract suggestion, or does it name a literal clichéd object (a phone for a calling app, a shield for security)? |
| **Structural distinctness** | If this is one of several directions, does it have its own form and token set, not just its own accent color? |

Do not skip this because the palette and typography were already locked correctly —
palette accuracy and slop-freedom are two different failure modes, and a prompt can
nail the former while still defaulting to generic motif clichés in the latter.
