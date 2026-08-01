# Contamination Check (Step 6 — Optional, Risk-Flagged)

Step 6 of the pipeline asks a genuinely open question: is it viable to use the trained
custom model on *every* generation in this world, including prompts that describe no
character at all (a landmark, a map, a piece of art)? Running that check is optional —
many worlds are perfectly well served by the with-model / without-model split the rest
of the pipeline already uses — but if the team wants the answer, this file is how to get
it honestly rather than by assumption.

## The actual risk, stated plainly

A custom model trained on one character's likeness learned that character's silhouette,
proportions, and distinguishing features as a strong prior. There is a real, non-hypothetical
risk that prior bleeds into prompts that never asked for a character at all:

- A landmark tower that comes out subtly bell-shaped, echoing the mascot's body silhouette.
- A crest or heraldry design whose central motif resembles the character's face or a
  signature accessory (a headphone shape, an antenna, a distinctive emblem).
- A textile or currency pattern whose repeating unit is, on close inspection, a stylized
  version of the character's silhouette rather than a neutral geometric motif.
- A map border ornament that repeats a shape traceable back to the character design.

None of this shows up as an error, a safety-filter block, or anything the API surfaces —
the image generates successfully and looks fine at a glance. The contamination is a
design problem, not a technical failure, and it only shows up on close visual comparison.

## How to run the check without pretending it's a formality

1. Pick a representative sample from step 1's moodboard batch and/or step 5's
   landmarks/maps/art batch — the no-model, no-character batches already generated
   earlier in the pipeline.
2. Rerun the same prompts (same palette, same anti-slop clauses, same "no characters"
   clause) with `custom_model_uri` set this time.
3. Place each contaminated-check image directly next to its original no-model
   counterpart from step 1 or step 5. Do not review the with-model version in isolation
   — the whole point of the check is the side-by-side comparison, since a subtle
   silhouette echo is much easier to spot next to a clean reference than in isolation.
4. Look specifically for shape/silhouette echoes of the character in structures,
   objects, patterns, and borders — not just for "does this still look like a landmark."
   A landmark that still reads as a landmark can still have contaminated proportions.

## Reporting the result

State plainly which outcome was observed, don't soften it:

- **No visible contamination in this sample** — safe to consider expanding custom-model
  usage further, but note this was a sample, not exhaustive coverage, and a different
  prompt (not in the sample) could still surface it.
- **Contamination observed** — name specifically which images and what the echo was (the
  tower, the crest, the pattern). Recommend keeping the with-model / without-model split
  from steps 1-5 rather than switching to model-everywhere, and don't quietly drop the
  contaminated images into the final world set.

Never report this step as "passed" or "safe" without the side-by-side visual comparison
actually having happened. This is the one step in the whole pipeline where "the API call
succeeded" is furthest from "this is fine" — the entire risk is invisible to the API and
only visible to a human looking at the pixels.
