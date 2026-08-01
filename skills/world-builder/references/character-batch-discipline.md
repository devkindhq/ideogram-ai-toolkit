# Character-Batch Discipline

Every step in the pipeline that touches characters (with or without the custom model)
has to state, explicitly, how many characters belong in the frame and whether any
characters belong in the frame at all. This is the mechanism that keeps with/without
pairs (step 2) and world-only batches (steps 1, 3's artifacts half, 5) clean, and the
mechanism that keeps community/hierarchy scenes (steps 3-4) from turning into an
uncontrolled crowd.

## The two clauses, and when each applies

**Character-bearing prompts** (any prompt with `custom_model_uri` set, or any prompt
depicting in-world figures at all): state an explicit character count in the prompt
itself — "exactly two characters," "a single character only," "a lineup of exactly five
figures in rank order." Never leave the count implicit or open-ended ("a group of
villagers," "a family") — an unbounded count is an invitation for the model to keep
adding figures, especially in scenes that narratively suggest a crowd (a festival, a
procession, a hierarchy lineup).

**No-model / world-only prompts** (step 1's moodboard, step 3's world-artifacts half,
step 5's landmarks/maps/art pass, and any world-only prompt in a contamination check):
state the explicit clause "no characters, mascots, or people anywhere in frame" on
every single prompt, every time, even when it feels redundant. This is the clause doing
the actual work of keeping the with/without pairs (step 2) and the "pure world-building"
steps genuinely clean — omitting it because "obviously a landmark prompt doesn't need
people" is exactly how a stray figure ends up in an otherwise character-free batch.

## Known model limitation: family resemblance, not distinct individuals

A model trained on ONE character's likeness will render multi-character scenes as
strong family-resemblance / near-clone figures, differentiated only by an accessory,
outfit color, or pose — not as genuinely distinct individuals the way a model trained on
many different characters would. This is **expected behavior for a single-character
custom model, not a defect to fix or a sign the training failed.**

This shows up most in the highest character-density steps: step 3's
community/hierarchy/ritual batch and step 4's family/village/society deep pass. Expect
it there specifically, and don't be surprised when it appears.

### How to review it correctly

When reviewing a batch with multiple characters in frame, ask: **"does this read as a
species/community, or does it read as broken?"** — not "are these individually distinct
characters?" A village scene where every villager shares Ding-Bot's bell-shaped
silhouette and face, differentiated by headwear, color accent, or role prop, reads as a
believable in-world *species* or *community* the way many real animated worlds work
(think: a species of characters that all share a base design, individualized by
costume). That's a pass. A scene where the near-clone effect produces visibly broken
anatomy, illegible faces, or a genuinely uncanny pile of identical figures is a fail —
revise the prompt (lower the character count, add explicit differentiating detail per
figure) and regenerate.

Do not silently treat every instance of family resemblance as a defect and try to
"fix" it by re-prompting for "diverse, distinct characters" — a single-character-trained
model cannot deliver that, and repeated attempts just burn budget chasing a result the
model architecture doesn't support. If the world genuinely needs visually distinct
individuals (not a resembling community), that requires training additional character
models — flag that as a scope question to the user rather than trying to prompt around
it.
