---
name: bulk-image-generation-workflow
description: Turns one locked structured JSON caption into an on-brief batch of N prompt variations, submits them as a single `generate_images_bulk` call, tracks the async job, and reviews/culls the results into a shortlist. Use whenever the user says "generate a batch of," "give me N variations," "generate variations of this," "bulk generate," "sticker pack," or "generate 20 options of X." Does not re-implement caption-writing (defers to `ideogram-prompt` for building or refining the base caption) or collection-filing (defers to `collections-management` for organizing the shortlist afterward), and is not for single-image generation — a one-off image stays with `ideogram-prompt`'s `generate_image`.
---

# Bulk Image Generation Workflow

This skill orchestrates around one tool: `mcp__ideogram__generate_images_bulk(prompts:
list[str], ...)`. It accepts 1–500 prompt strings and submits them as one async background
job; results render into a carousel as each image completes rather than blocking on the
whole batch.

The constraint the whole skill is built on: every non-prompt parameter
(`aspect_ratio`, `resolution`, `rendering_speed`, `style_type`, `negative_prompt`, `seed`,
`magic_prompt_option`, `custom_model_uri`, `collection_id`, `private`) is shared across the
entire batch — there is no per-prompt override. On-brief variation at scale has to come
from differences in the prompt text itself, not from differences in call parameters. Note
the caveat: `style_type`, `negative_prompt`, `seed`, and `magic_prompt_option` only take
effect when `custom_model_uri` is set (the custom-model v3 path); on the default v4 path
they're ignored, so passing them there does nothing.

This skill sits between `collections-management`'s pure-orchestration pattern and the pure
prompt-composition skills' pattern: mostly orchestration, with one prompt-composition
component — turning one caption into N prompts — covered by
`references/variation-strategy.md`.

## Before you start

Read `references/variation-strategy.md` before drafting the prompt array (steps 2–3 below)
and `references/review-culling-guide.md` before reviewing results (step 6 below). Both
apply before you touch the workflow.

## Workflow

### 1. Resolve the base caption

Use an existing locked structured JSON caption if the user or project context already has
one. Otherwise build one quickly using `skills/ideogram-prompt/references/json-caption-schema.md`'s
schema and `ideogram-prompt/SKILL.md`'s precise-mode guidance, rather than re-deriving the
schema here. This caption's `style_description` object is what stays locked for the whole
batch.

### 2. Choose the variation axis

Per `references/variation-strategy.md`, decide what changes per prompt — pose, prop,
angle, framing, a swapped `compositional_deconstruction` element — while `style_description`
stays byte-for-byte identical across every prompt in the batch. If the axis isn't obvious
from the user's request, confirm it with them before drafting.

### 3. Draft the N-prompt array

Write each variation as a prompt string, using either JSON-in-prompt or schema-structured
prose per `ideogram-prompt`'s two modes, producing the `prompts` list argument. Enforce the
1–500 bound before submitting: if the requested variation count would exceed 500, ask the
user to narrow the axis or split into multiple `generate_images_bulk` calls rather than
truncating the list.

### 4. Confirm batch size before submitting

There's no dry-run or preview for this tool, and every image in the batch spends
generation credits on submission. For anything past a small batch (roughly 20+ prompts),
get an explicit go-ahead on the count before calling `generate_images_bulk` — the same
spend-before-you-commit spirit `collections-management` applies to destructive flags,
adapted here to spend instead of deletion.

### 5. Submit and track

Call `generate_images_bulk(prompts=[...], ...)` once, passing the shared non-prompt
parameters the user specified (noting the custom-model-only params from step 2 above).
Tell the user it's an async job and they can keep chatting while it renders. Use
`get_generation_status(request_id=...)` — or omit `request_id` to list everything from the
session — to check progress. Its response is a markdown table of
`Request ID | Status | Prompt | URL` rows in terminal hosts, with the same data available
in `structured_content.rows`. The exact shape `generate_images_bulk` itself returns for a
multi-prompt submission (one batch-level identifier vs. one row per prompt) should be read
from the actual response the first time it's called in a session, not assumed in advance —
the same discipline `collections-management/references/collection-patterns.md` applies to
unconfirmed field shapes.

### 6. Review and cull

Once results are available, follow `references/review-culling-guide.md`: pull results via
`get_generation_status` (or `get_recent_generations(n=..., filter_mode="GENERATIONS")` as a
fallback lookup, capped at 50/call), score each image against the locked
`style_description` and its own per-prompt compositional intent, record a one-line
keep/reject reason per image, and produce a shortlist. Hand the shortlist to
`collections-management` to file it and to `upscale_image` for any finals that need higher
resolution — this skill doesn't reimplement either.

## Error handling

- Batch count outside 1–500 → ask the user to narrow the axis or split into multiple
  calls; never silently truncate or pad the prompt list to fit.
- Large-batch submission without explicit size confirmation → don't submit; ask first,
  since there's no dry-run and every image spends credits the moment it's submitted.
- A drafted prompt that changes `style_description` fields instead of only
  `compositional_deconstruction` → flag it during drafting (or in review, if it slipped
  through) as off-brief; fix the prompt or split it into a separate batch rather than
  letting one drifted prompt sit in an otherwise-locked batch.
- Partial batch failure (some prompts fail to render or return a safety-filter-blocked
  placeholder) → surface the real per-item outcome; a blocked or failed image is a
  reportable failure, not a keeper, and the batch is not "fully successful" just because
  most of it rendered.
- Unconfirmed response shape from `generate_images_bulk` → read what the actual response
  contains rather than asserting a specific field name or structure in advance; report
  what was actually observed.
- Bad batch (most images miss the brief on review) → per
  `references/review-culling-guide.md`, revise the drifting part of the caption or the
  variation axis and resubmit a smaller follow-up batch; don't resubmit the full original
  count unchanged and hope for a different result.

## Save what you made

After drafting the prompt array, submitting, and reviewing, save the locked caption, the
full `prompts` array actually submitted, the `request_id`(s) from `generate_images_bulk`,
and the keep/reject shortlist with reasons to the project's existing output location,
rather than leaving them only in the conversation, per the toolkit's "No Context Lost"
habit.

## Reference files

- `references/variation-strategy.md` — how to pick a variation axis and draft N prompts
  that vary only that axis while keeping `style_description` locked. Read before step 2 or
  3.
- `references/review-culling-guide.md` — how to score generated images against the locked
  caption and per-prompt intent, and how to shortlist or resubmit a bad batch. Read before
  step 6.
