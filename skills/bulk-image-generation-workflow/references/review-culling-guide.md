# Review & Culling Guide

This file covers what `SKILL.md` points here for before reviewing results (workflow step
6): how to pull a batch's results back, how to score each image, where the shortlist goes
afterward, and what to do when a batch comes back bad. It builds on
`references/variation-strategy.md`'s locked-`style_description` /
per-prompt-`compositional_deconstruction` split — reviewers score against that same
distinction, not a separate rubric.

## Pull the results

There are two ways to get a batch's images back, and which one applies depends on whether
the `request_id`(s) from the `generate_images_bulk` submission were kept.

**`get_generation_status(request_id=...)`** is the primary path. Pass the `request_id` for
the job to make sure it's included, or omit `request_id` entirely to list everything from
the current session. In terminal hosts the response is a markdown table —
`Request ID | Status | Prompt | URL` — meant to be rendered verbatim rather than
reformatted. The same rows are available in `structured_content.rows` for the model to
introspect programmatically (matching a prompt back to its rendered URL, filtering by
status, and so on) rather than parsing the rendered markdown back out.

**`get_recent_generations(n=..., filter_mode="GENERATIONS")`** is the fallback when a
`request_id` from the batch wasn't kept, or when reviewing images that fell out of the
current session's tracked jobs. It returns the `n` most recent generations, newest first,
capped at 50 per call (the tool defaults to 10 if `n` is omitted — for a batch of any real
size, pass an explicit `n` rather than relying on the default). `filter_mode="GENERATIONS"`
(the tool's own default) restricts results to AI-generated images only, excluding upscales,
uploads, and edits that would otherwise mix into the list under `filter_mode="EVERYTHING"`.
Each item carries a `response_id` — this is the identifier to carry forward into
`edit_image`, `remix_image`, `reframe_image`, or `upscale_image` for anything shortlisted
below.

## Score each image

Every image in the batch gets checked against two separate things, because the batch was
built out of two separate parts (per `variation-strategy.md`):

1. **The locked `style_description`.** Does the image actually match the aesthetics,
   lighting, medium, and palette that were supposed to be identical across the whole
   batch? This is the check for whether the batch actually rendered as one coherent look,
   not several.
2. **Its own per-prompt `compositional_deconstruction` intent.** Did the specific pose,
   prop, angle, or element that *this* prompt asked for actually render? This is the check
   for whether the variation axis did its job on this particular prompt.

An image can fail either check independently of the other — a render can nail the
requested pose while drifting off the locked palette, or match the style perfectly while
missing the requested prop entirely. Both are grounds for rejection.

Record a one-line keep/reject note per image, and make the reason concrete: name which
locked `style_description` field or which per-prompt element failed to render as asked,
not a vague "looks off" or "doesn't work." "Reject — lighting reads as flat studio light,
not the locked golden-hour aesthetics" is a usable note; "reject, off" is not, because it
gives nobody (including a future re-review) anything to act on. The output of this pass is
a shortlist of keeps with reasons, plus reject reasons for anything that didn't make it.

## Handoff

The shortlist has two possible destinations, and this skill doesn't implement either
directly:

- **Filing into a collection** goes to `collections-management` — see `SKILL.md`'s
  Workflow step 6. This skill doesn't call `create_collection` or
  `add_images_to_collection` itself; hand the shortlisted `response_id`s off to that skill
  once the shortlist is final.
- **Upscaling a keeper** that needs higher resolution before final delivery goes to
  `upscale_image(image_response_id=...)`. `upscale_factor` defaults to `X2` (the tool also
  supports `X1`, `X4`, `X8`). The tool accepts exactly one of `image_response_id` or
  `image_upload_id` — for anything coming out of this review pass, that's always
  `image_response_id`, since the images already exist as generations rather than fresh
  uploads.

## Partial failures are real failures

A prompt that returns a safety-filter-blocked placeholder, or otherwise fails to render, is
a reportable failure in the review pass — not something to silently drop from the count.
Report how many of the N submitted prompts actually succeeded, not just how many keepers
were found among the successes. A batch where 15 of 20 prompts rendered and 8 of those 15
were keepers is a batch with 5 failures and 8 keeps, not "8 good ones out of 15" with the
missing 5 unmentioned. The distinction matters because a high failure rate is itself a
signal that something about the prompt array or the locked style is triggering rejections
upstream of the aesthetic review, and that signal disappears if failures are quietly
excluded from the report.

## Bad batch: revise and resubmit small

If most images in a batch miss the brief on review, the fix is not to resubmit the same
prompt array and hope for a different outcome. Instead:

1. **Identify which part actually drifted**, using the same locked-style /
   varied-composition split from `variation-strategy.md`: did `style_description` fail to
   stay locked across the prompts (a drafting defect), or did the variation axis itself —
   the specific poses, props, or angles chosen — produce compositions that miss the brief
   even though the style held (an axis-choice problem)? These have different fixes: a
   locked-style leak gets corrected in the caption; a bad axis gets rethought in what
   varies, per `variation-strategy.md`'s guidance on choosing the axis.
2. **Revise that specific part** of the caption or the prompt array — not the whole thing,
   and not by guessing at a different unrelated change.
3. **Resubmit a smaller follow-up batch** to validate the fix, using the same roughly
   8–20-prompt starting range `variation-strategy.md`'s batch-size judgment section
   recommends for a first run. Validate the fix cheaply before committing to a full-size
   batch again — don't resubmit the full original count unchanged, since that repeats
   whatever produced the bad batch at full cost instead of confirming the fix first.

## Self-review checklist

Before treating this file as done, confirm it actually covers, with real specific text
rather than a placeholder:

- Both retrieval paths (`get_generation_status` and `get_recent_generations`), including
  when each applies and what each returns.
- Scoring against both the locked `style_description` and the per-prompt
  `compositional_deconstruction` intent, with a concrete one-line keep/reject reason
  format.
- The handoff to `collections-management` (filing) and `upscale_image` (resolution), with
  the correct parameter names and defaults for `upscale_image`.
- Partial failures counted and reported as failures, not silently excluded from the
  keeper count.
- The bad-batch loop: identify the drifted part, revise it specifically, resubmit at the
  smaller 8–20-prompt range rather than the full original count.
