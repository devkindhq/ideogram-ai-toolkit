# Anti-Slop Discipline for Moodboards

A moodboard has more surface area for slop than any sibling skill — nine panels
instead of five or six, and unlike `brand-identity-sheet` and `logo-prompting`, this
skill's own framing deliberately invites "moodboard energy" (the exploratory, looser
feel) rather than fighting it. That makes the anti-slop discipline *more* important
here, not less — it's easy to mistake "loose and exploratory" for "generic," and they
are not the same thing. A loose sketch of a brand-specific motif is exploration; a
generic AI-startup gradient wash is slop, regardless of how "moodboard-y" it looks.

Adapted from `brand-identity-sheet/references/anti-slop-discipline.md`, which is itself
adapted from Hallmark's slop-test discipline. This file is the gate you run **before**
calling `generate_image`, not a style suggestion to eyeball after.

## The exploration-vs-collage line

This is the one place this skill's guidance differs meaningfully from its siblings:
`brand-identity-sheet` avoids the word "moodboard" entirely because it wants the
model to reach for a structured system-sheet look. This skill *wants* the word
"moodboard" and the exploratory energy that comes with it — but only up to the point
where "exploratory" tips into "unstructured Pinterest dump with no throughline." The
test: every panel should still trace back to one of the six template variables (see
`panel-anatomy.md`'s closing section) and the board should still render as nine
distinct, labeled, gridded panels — never as nine images bleeding into a collage with
no borders.

## Universal bans (same list as the sibling skills, apply to every panel)

Unless the brand truth explicitly calls for one of these as a deliberate, named
exception, no panel should include:

- **Neural-network nodes / circuit-brain diagrams** as a stand-in for "modern" or
  "tech-forward" when the brand hasn't actually asked for that register.
- **A glowing orb or floating sphere as a hero element** in the Abstract Patterns or
  Mood Imagery panels — a contained glow tied to an actual brand reference (bioluminescence,
  a specific light quality named in `{Imagery & Texture}`) is fine; an unmotivated
  generic AI-glow is not.
- **Headset / phone-receiver icon** in Iconography Style for "support" or "contact"
  unless the brand's product actually involves that.
- **Padlock / shield** in Iconography Style for "security" unless the product is
  genuinely a security product.
- **Lightbulb** anywhere for "ideas" or "innovation."
- **Soundwave bars / abstract waveform** as generic "tech" filler with no audio
  product behind it.
- **Circuit-board texture** in Material Samples as a stand-in for "technical" when
  `{Imagery & Texture}` already names something more specific.
- **A purple-to-pink or cyan-to-magenta gradient wash** across the whole board or
  background — the single most recognizable "AI startup" tell, and the fastest way a
  moodboard collapses into looking like every other AI-tool moodboard instead of this
  brand's.
- **Photorealistic stock-photo clichés** in Photography & Graphics (hands on a laptop,
  generic smiling-team stock imagery) unless the brand's actual references call for
  that register.
- **Startup-cliché placeholder names** (Acme, Nexus, Seamless) in the Application
  Mockup panel — use the real brand name if known, or a plausible in-world label if
  the name itself is still undecided (never a generic filler brand).

## Why "exploratory" can't mean "vague"

The failure mode specific to this skill: because the brief is often fuzzy (three
adjectives and maybe a color, nothing else), it's tempting to let every panel default
to generic "nice design" filler on the theory that the brand isn't locked yet anyway.
That's backwards — a fuzzy brief is exactly when the moodboard needs to work *harder*
to be specific, because it's the artifact meant to sharpen the fuzziness into real
decisions. "Warm, unhurried, grounded" adjectives should produce a genuinely different
board than "sharp, urgent, precise" ones — if two moodboards built from different
adjective sets would look interchangeable, the prompt isn't doing its job.

## Pre-generation gate — run this before calling `generate_image`

Score the drafted paragraph prompt 1-5 on each axis below. Anything under 3 on any
axis means revise the prompt before generating.

| Axis | What you're checking |
|---|---|
| **Adjective specificity** | Does the board's material/color/imagery vocabulary actually differ based on the three stated adjectives, or would this prompt produce roughly the same board for any brand? |
| **Universal-ban sweep** | Read the drafted prompt sentence by sentence against the "Universal bans" list above. Is every banned noun absent, or present only as a brand-justified, explicitly-named exception? |
| **Exploration vs. collage** | Does the prompt still name all nine panels explicitly, with a grid/border/caption structure — or has "moodboard energy" drifted the description toward an unstructured collage with no panel boundaries? |
| **Logo-panel restraint** | Is the Logo Exploration panel framed as rough sketches/variations (2-3 small marks), not accidentally written as if it's the one finished, final logo? |
| **Application-mockup coherence** | Does the final panel actually show the other eight panels' choices working together, or is it an unrelated generic mockup bolted on at the end? |

Do not skip this because the brief was fuzzy going in — a vague brief is more likely to
produce a generic board, not less, so the gate matters more here, not less.
