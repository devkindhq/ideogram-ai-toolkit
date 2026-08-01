# Batch Tracking

A world-building pipeline runs six-plus batches, often across multiple sessions, mixing
single `generate_image` calls with `generate_images_bulk` submissions, some with the
custom model and some without. Without a tracking record per batch, it becomes
impossible to answer basic questions later ("which of these used the trained model?",
"did anyone actually look at the family/village batch before it got called done?") —
this file is the minimum record to keep for every batch in every step.

## What to record per batch

For every `generate_image` call and every `generate_images_bulk` submission in the
pipeline, record:

- **Step** — which of the six pipeline steps this batch belongs to (moodboard / paired
  with-without / society-culture / family-village deep pass / landmarks-maps-art /
  contamination-check).
- **`request_id` / `job_id`** — from the tool response. For a `generate_images_bulk`
  call, capture whatever identifier(s) the actual response contains (read the real
  response shape rather than assuming a specific field name in advance, same discipline
  `bulk-image-generation-workflow` applies).
- **`custom_model_uri` used** — the exact URI if the custom model was used, or explicitly
  `none` if this was a no-model / world-only batch. Never leave this field blank or
  implied.
- **Prompts submitted** — the exact prompt text(s), not a paraphrase, so a later review
  or regeneration can be compared against exactly what was asked for.
- **`visual_review_status`** — one of `pending`, `reviewed-pass`, `reviewed-revise`.
  Every batch starts at `pending` the moment the API call returns success, and stays
  `pending` until a human has actually looked at the resulting images. An API 200 is not
  a review.

## The "not yet visually reviewed" rule

Do not describe any batch as "done," "complete," or ready to hand off downstream (to a
designer, to a client deck, to the next pipeline step) based on the generation call
succeeding. A successful API call means the images exist and are worth reviewing next —
nothing more. Only mark `visual_review_status: reviewed-pass` after someone has actually
looked at the images against:

- the locked palette (`palette-lock.md`),
- the anti-slop ban list (`anti-slop-discipline.md`),
- the character-count / no-characters clause for that batch (`character-batch-discipline.md`),
- and, for character-driven batches, the "species/community vs. broken" read described
  in `character-batch-discipline.md`.

If review finds a problem, record `visual_review_status: reviewed-revise` along with a
one-line reason, and treat the batch as needing a follow-up regeneration — not as
something to quietly accept because redoing it is inconvenient this late in a
multi-session pipeline.

## Where to save this

Same location as the rest of the pipeline's output — the project's world-building folder
(see the SKILL.md's "Save what you made" section). One row per batch is enough; a simple
markdown table (step, request/job ID, `custom_model_uri`, prompt summary, review status)
keeps this legible across a pipeline that may span many sessions and many weeks.
