# Bulk Image Generation Workflow Skill — Design

## Why

Gap-research identified `bulk-image-generation-workflow` as a Tier 2, uncontested gap: no
prior art in the Claude Code skill ecosystem wraps Ideogram's batch-generation tool into a
guided workflow, and no differentiation angle is required beyond matching this repo's
existing "structured JSON caption + anti-slop discipline + compositional-deconstruction"
methodology.

The connected Ideogram MCP already exposes the tool this skill wraps:
`mcp__ideogram__generate_images_bulk` (confirmed live via direct tool-schema inspection,
not assumed from docs) — takes `prompts: list[str]` (1-500 items), submits as an async
background job, and shares one set of non-prompt parameters
(`aspect_ratio`, `resolution`, `rendering_speed`, `style_type`, `negative_prompt`, `seed`,
`magic_prompt_option`, `custom_model_uri`, `collection_id`, `private`) across every prompt
in the batch — there is no per-prompt override for any of those fields.

The core use case: a user has (or wants) one locked visual direction — a single structured
JSON caption, in the schema `skills/ideogram-prompt/references/json-caption-schema.md`
already documents — and wants many on-brief variations of it in one shot (a sticker pack,
a set of pose/prop options, a range of product-shot angles) rather than hand-writing and
submitting N separate `generate_image` calls. Because the batch's style parameters are
fixed for the whole call, "on-brief variation at scale" is fundamentally a prompt-text
problem: every variation has to come from differences in what's written into each prompt
string, not from differences in the shared call parameters. That's the problem this skill
solves, and it's also why "how do you turn one caption into N prompts without drifting off
the brief" is the one genuinely new piece of judgment this skill needs beyond what
`collections-management` and `custom-model-training` (pure MCP orchestration, no
prompt-writing) already model.

## Scope

One new skill: `skills/bulk-image-generation-workflow/`. Triggers on explicit batch-style
requests: "generate a batch of," "give me N variations," "generate variations of this,"
"bulk generate," "sticker pack," "generate 20 options of X." Covers the full loop —
resolving/locking the base caption, drafting the N-prompt array, confirming batch size,
submitting the batch, tracking it to completion, and reviewing/culling the results — not
just prompt-drafting advice.

Does not re-implement caption-writing or the JSON schema itself — this skill assumes the
user either already has a locked structured JSON caption or wants one built quickly using
`ideogram-prompt`'s existing schema and precise-mode guidance, and points there rather than
duplicating it. Does not re-implement collection filing — keepers get handed to the
existing `collections-management` skill. Single-image generation (`generate_image`) stays
out of scope; this skill exists specifically for the `generate_images_bulk` batch path.

## Architecture

Structural pattern matches the repo: `SKILL.md` + `references/` + `examples/` + `evals/`.

This skill sits between the two existing architectural patterns in this repo rather than
matching either one exactly:

- **Pure orchestration skills** (`collections-management`, `custom-model-training`) chain
  sequential MCP calls and need no prompt-composition reference material at all.
- **Pure prompt-composition skills** (`brand-identity-sheet`, `character-model-sheet`,
  `moodboard-generator`) produce one deconstructable image and carry the full
  `composition-spec-format.md` / `panel-anatomy.md` / `anti-slop-discipline.md` trio.

`bulk-image-generation-workflow` is mostly orchestration (submit one async batch job,
poll it, then review N results) but has one prompt-composition component with no existing
home: turning one locked JSON caption into an array of N on-brief prompt strings. That
gets its own lightweight reference (`variation-strategy.md`) rather than the full
composition-spec trio, because this skill isn't deconstructing a single multi-panel image
— each output is an independent single-subject image, evaluated against the same locked
caption, not against a bbox layout.

## Components

- **`SKILL.md`** — frontmatter triggers on the batch-request phrasing above. Body
  documents the 6-step workflow (below) and links out to `ideogram-prompt` for caption
  construction and `collections-management` for post-review filing rather than duplicating
  either.
- **`references/variation-strategy.md`** — the rule for turning one JSON caption into N
  prompts without drifting off-brief: lock `style_description` (aesthetics, lighting,
  medium, `art_style`/`photo`, `color_palette`) identically across every prompt in the
  batch; vary only `compositional_deconstruction` (elements, pose, prop, angle, framing)
  per prompt. Covers batch-size judgment (start small — roughly 8-20 — before committing
  to a full 100+ run; the 1-500 cap is a hard tool limit, not a target) and the "one locked
  style_description per batch" rule: if the user wants genuinely different lighting or
  medium on some of the images, that's a second `generate_images_bulk` call with its own
  locked caption, not one call with drifting style fields.
- **`references/review-culling-guide.md`** — how to review a completed batch: pull results
  (`get_generation_status` for the job's progress table, `get_recent_generations` as a
  fallback lookup by `response_id`), score each image against the locked
  `style_description` fields and its own per-prompt compositional intent, record a
  one-line keep/reject reason per image, and produce a shortlist. Covers handoff to
  `collections-management` (file the shortlist) and `upscale_image` (finalize keepers), and
  the "revise-and-resubmit-small" loop for a batch that mostly missed the brief, instead of
  re-submitting the full original count.
- **`examples/`** — one worked example to be produced by actually running the pipeline
  once an initial batch job completes (locked caption → N-prompt array → submitted batch →
  reviewed/culled shortlist), matching how `custom-model-training`'s and
  `collections-management`'s examples were produced.
- **`evals/evals.json`** — 3 eval prompts in the existing schema (`id`, `eval_name`,
  `prompt`, `expected_output`, `files`), matching `collections-management`'s format.

## Data flow / workflow

1. **Resolve the base caption** — use an existing locked structured JSON caption if the
   user (or project context) already has one; otherwise build one quickly per
   `ideogram-prompt`'s schema and precise-mode guidance rather than re-deriving the schema
   here. This caption's `style_description` is what stays locked for the whole batch.
2. **Choose the variation axis** — per `references/variation-strategy.md`, decide what
   changes per prompt (pose, prop, angle, framing, a swapped `compositional_deconstruction`
   element) while `style_description` stays byte-for-byte identical across every prompt.
   Confirm the axis with the user if it isn't obvious from their request.
3. **Draft the N-prompt array** — write each variation as a prompt string (JSON-in-prompt
   or schema-structured prose, per `ideogram-prompt`'s two modes), producing the `prompts`
   array `generate_images_bulk` takes. Enforce the tool's 1-500 bound before submitting; if
   the user's variation count would exceed 500, ask them to narrow the axis or split into
   multiple batches rather than silently truncating the list.
4. **Confirm batch size before submitting** — no dry-run/preview exists for this tool, and
   each image in the batch consumes generation credits on submission. For anything past a
   small batch (roughly 20+), get an explicit go-ahead on the count before calling the
   tool, the same "confirm before the costly, hard-to-undo thing" spirit
   `collections-management` applies to destructive flags, adapted here to spend instead of
   deletion.
5. **Submit and track** — call `generate_images_bulk(prompts=[...], ...)` once, with the
   shared non-prompt parameters (aspect_ratio/resolution/rendering_speed/style_type/
   negative_prompt/seed/magic_prompt_option/custom_model_uri/collection_id/private) applied
   to the whole batch. It's an async job — tell the user they can keep chatting while it
   renders, and use `get_generation_status` to check progress. The exact shape of what
   `generate_images_bulk` returns for a multi-image job (a single batch identifier vs. one
   `request_id` per prompt) isn't stated in the tool's description text; read the actual
   response fields the first time it's called in a session rather than assuming a shape
   ahead of time — same discipline `custom-model-training`'s spec applied to `get_model`'s
   unconfirmed status field.
6. **Review and cull** — once results are available, follow
   `references/review-culling-guide.md`: score each image against the locked
   `style_description` and its own per-prompt intent, record keep/reject with a reason,
   produce a shortlist. Hand the shortlist to `collections-management` to file it and to
   `upscale_image` for any finals that need higher resolution — this skill doesn't
   reimplement either.

## Error handling

- Batch count outside the 1-500 bound → ask the user to narrow the variation axis or split
  into multiple `generate_images_bulk` calls; never silently truncate or pad the list.
- Large-batch submission without explicit size confirmation → don't submit; ask first,
  since there's no dry-run and every image in the batch spends credits on submission.
- A drafted variation prompt that changes `style_description` fields instead of only
  `compositional_deconstruction` → flag it during drafting (or in review, if it slipped
  through) as off-brief, and fix the prompt or split it into a separate batch rather than
  letting one drifted prompt sit in an otherwise-locked batch.
- Partial batch failure (some prompts fail to render, or return a safety-filter blocked
  placeholder) → surface the real per-item outcome; a blocked/failed image is a reportable
  failure, not a keeper, and the batch is not "fully successful" just because most of it
  rendered.
- Unconfirmed response shape from `generate_images_bulk` (see step 5 above) → read what the
  actual response contains rather than asserting a specific field name or structure in
  advance; report what was actually observed.
- Bad batch (most images miss the brief on review) → per
  `references/review-culling-guide.md`, revise the drifting part of the caption or the
  variation axis and resubmit a smaller follow-up batch; don't resubmit the full original
  count unchanged and hope for different results.

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts:
1. "Generate 12 on-brief variations of [a locked caption], varying the pose and prop" —
   verifies `style_description` stays locked, only `compositional_deconstruction` varies
   across the 12 prompts, and they're submitted as one `generate_images_bulk` call.
2. "Give me a sticker pack of 30 versions of this mascot in different poses" — verifies the
   batch-size confirmation step triggers before submitting (30 is past the small-batch
   threshold), and that review/culling guidance is applied once results are back.
3. "Generate 20 variations, but I want completely different lighting on half of them" —
   verifies the skill recognizes this as two locked captions, not one batch with drifting
   `style_description`, and either asks to split it into two `generate_images_bulk` calls
   or declines to fold both lighting directions into a single batch.

## Assumptions

Documented per the "don't stop to ask, state the call explicitly" instruction for this
autonomous run:

- **Shared params apply batch-wide, confirmed from the tool schema** — `generate_images_bulk`
  has no per-prompt override for `aspect_ratio`, `resolution`, `rendering_speed`,
  `style_type`, `negative_prompt`, `seed`, `magic_prompt_option`, `custom_model_uri`,
  `collection_id`, or `private`; all of it is one shared set of arguments alongside the
  `prompts` array. This is the basis for the whole "variation must live in prompt text"
  architecture above — it's read directly off the live tool schema, not inferred.
- **Batch-size confirmation threshold (~20) is a chosen default, not an Ideogram rule** —
  there's no documented or tool-enforced threshold for when batch spend needs confirming.
  20 is picked as a reasonable "small enough to not need a stop" cutoff consistent with the
  1-500 hard cap; a future revision can tune this if it proves wrong in practice.
- **`generate_images_bulk`'s multi-image response shape is unconfirmed** — the tool's
  description states it runs as a background job and results render in a carousel as they
  complete, but doesn't state whether the structured response carries one job-level
  identifier or a `request_id` per prompt. Treated as unverified per the standing rule
  against asserting third-party API behavior as fact without a verified source; the skill
  reads whatever the response actually contains.
- **Review/culling is Claude-assisted visual judgment, not an automated scorer** — no tool
  on the connected Ideogram MCP does automated quality scoring or brief-matching, so
  "review guidance" means a human-plus-Claude pass against the locked caption's fields, not
  a metric or classifier this skill invokes.
- **`collection_id` on `generate_images_bulk` and the separate `collections-management`
  skill are complementary, not merged** — a batch can optionally be auto-filed into an
  existing collection at submission time via the tool's own `collection_id` param, but
  post-review curation (filing only the keepers, removing rejects) still goes through
  `collections-management` rather than this skill reimplementing add/remove logic.

## Out of scope

- Building a structured JSON caption from scratch — delegates to `ideogram-prompt`'s
  existing schema and precise-mode guidance rather than duplicating it here.
- Filing images into collections — delegates to the existing `collections-management`
  skill; this skill only hands off a shortlist, it doesn't call
  `add_images_to_collection`/`create_collection` itself.
- Custom model training as a generation backend — `custom_model_uri` is passed through
  untouched if the user supplies one, but dataset creation/training/polling stays entirely
  in `custom-model-training`.
- Automated/algorithmic image-quality scoring or classifier-based culling — review
  guidance in this skill is judgment-based (Claude plus the user, against the locked
  caption), not a metric pipeline.
- Cross-batch or cross-session near-duplicate detection — not built in v1.
- Single-image generation (`generate_image`) — stays covered by `ideogram-prompt`; this
  skill is scoped to the batch path only.
