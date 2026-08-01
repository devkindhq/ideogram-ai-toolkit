# Palette Lock

A world built across six-plus separate generation batches (often across several
sessions) has no other mechanism holding it together than the words repeated in every
prompt. If the palette isn't locked and quoted verbatim from the first prompt onward, it
will drift batch to batch — not because any single image is wrong, but because nothing
is forcing continuity between images generated hours or days apart, by different calls,
sometimes with and sometimes without the custom model.

## Define this before step 1, not during it

Before the moodboard pass (step 1), lock:

- **Paper / dominant color** — the base/background tone the world sits on. Exact hex.
- **Primary color** — the color that carries the most visual weight across the world
  (the mascot's own dominant color, if the character has one worth extending into the
  world). Exact hex.
- **Secondary color** — the supporting color. Exact hex.
- **Ink / trim color** — the color used for line work, text, and fine detail. Exact hex.
- **At most one reserved accent color, used in exactly one place** — not a fifth color
  free to appear anywhere. Name the one place it's allowed (e.g. "the reserved accent
  gold appears only on ceremonial/rank markers, nowhere else in the world"). This
  restraint is what keeps a six-batch world from turning into a rainbow by batch four.

If the character already has a locked palette from `character-model-sheet` or
`custom-model-training` (e.g. Ding-Bot's orange body / light-blue headphones / gold-brass
trim), that palette is the anchor — extend it into paper/primary/secondary/ink-trim/accent
roles rather than inventing a new one that competes with the character's own colors.

## Quote it verbatim, every prompt, every step

Once locked, state the palette with hex values and role labels in every single
generation prompt for the rest of the pipeline — steps 1 through 6, with-model and
without-model alike. Don't paraphrase it as "warm and earthy" after the first batch;
paraphrasing is exactly how palette drift gets introduced across a pipeline this long.
Copy the same hex/role string into the next prompt.

## What "drift" looks like if this isn't followed

- The moodboard (step 1) locks a warm terracotta/cream palette, but by the family/village
  deep pass (step 4) the buildings have drifted to a cooler blue-gray because no one
  re-quoted the original hex values.
- The reserved accent color (meant for exactly one place) starts appearing on random
  objects across batches because later prompts described it loosely as "a pop of gold"
  instead of naming its one sanctioned location.
- Two batches generated in different sessions read as two different, unrelated worlds
  when placed side by side, even though each batch individually looks fine.

Catch this at review time (per `batch-tracking.md`'s visual-review step), not just at
prompt-drafting time — a drifted batch is still a drifted batch even if the prompt that
produced it technically mentioned the right words.
