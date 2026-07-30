---
name: upscale-image-workflow
description: Raises the resolution of one specific Ideogram image — from this session's generations, `get_recent_generations`, a collection, or a local upload — via the connected `upscale_image` tool. Use whenever the user asks to "upscale this," "make this higher-res," "upscale that logo render 4x," or "increase the resolution on the image I just uploaded." Wraps exactly one tool (`upscale_image`) and does NOT wrap `remix_image`, `reframe_image`, `edit_image`, or `remove_background` — those are separate tools, out of scope here. It does not auto-trigger from `brand-identity-sheet`, `character-model-sheet`, `moodboard-generator`, or `collections-management`; those four are unmodified and unaware of this one, and this only runs on explicit request.
---

# Upscale Image Workflow

Ideogram exposes one tool for raising an image's resolution: `upscale_image`. This skill
orchestrates a single call to that tool — resolve which image, confirm the settings, run
it, save the output's identifiers. It takes exactly one call per invocation: no multi-step
pipeline, no multi-tool surface. It's the thinnest skill in the repo.

This is an orchestration workflow around one MCP call, not a prompt-composition skill —
there's no `composition-spec-format.md`/`panel-anatomy.md`-style reference here.

## Before you start

Read `references/upscale-settings.md` before running step 1 or step 2 of the workflow
below. It covers the identifier-resolution rule (`upscale_image` requires exactly one of
`image_response_id` or `image_upload_id` — never both, never neither) and the settings
guide for every optional parameter.

## Workflow

### 1. Resolve the target image

Determine exactly one identifier to pass. Either:

- `image_response_id` — from this session's `generate_image`/`remix_image`/`edit_image`/
  `reframe_image`/`remove_background` calls' `structured_content.response_ids`, from
  `get_recent_generations`'s per-item `response_id`, or from
  `get_images_by_collection_id`'s per-asset `response_id`/`image_id`.
- `image_upload_id` — from `upload_image`, for a local file the user points to that hasn't
  been uploaded yet. Call `upload_image` first rather than inventing an ID: per
  `upload_image`'s own description, run its returned `instructions` curl command in the
  sandbox and read the real `id` back from the curl response.

If more than one candidate image is plausibly "that image" (e.g. several renders made
earlier this session with nothing distinguishing which one the user means), ask which one
— don't guess. Never pass both identifiers or neither. See
`references/upscale-settings.md` for the full identifier-resolution rule.

### 2. Confirm settings

State the resolved settings before calling:

- `upscale_factor` — the user's stated value if given (that counts as confirmed, no need
  to re-ask), or the tool's own `X2` default, stated plainly rather than blocked on a
  confirmation question.
- `upscale_details_weight` — only set if the user asked for more/less added detail;
  otherwise omit for the backend default.
- `prompt` — only set if the user wants to steer what detail gets added; otherwise omit —
  the backend auto-describes the image when none is supplied, per the tool's own
  description.
- `collection_id` — only if the user asked to save directly into an existing collection.
  If so, follow `collections-management`'s find-or-create pattern from
  `collection-patterns.md` to resolve which `collection_id` to pass, rather than inventing
  one.
- `private` — omit unless the user explicitly asks to publish to the public Ideogram feed.
  Per the tool's own description, paid accounts default to private and free/Basic accounts
  default to public, and enterprise generations are always private regardless — state that
  as the only asserted fact about the default, since it's the tool's own documented
  behavior.

### 3. Run the upscale

Call `upscale_image` with the resolved identifier and settings. Per the tool's own STRICT
instruction: if the call fails, surface the failure to the user verbatim and stop. Never
fall back to `generate_image`, `remix_image`, or any other tool as a substitute for a
failed upscale.

### 4. Save the output

Persist the response's identifiers (whatever the live response actually contains — at
minimum expect a `response_id`, matching every other Ideogram generation tool in this
toolkit), the resolved `upscale_factor`, and the source image's identifier to the
project's existing output location — the same place `brand-identity-sheet`,
`character-model-sheet`, `moodboard-generator`, or `collections-management` already save
to for this project. If a `collection_id` was passed in step 2, note that the upscaled
image was also filed there. The exact response field name(s) beyond `response_id` (e.g.
whether there's a downloadable URL) are read from the live response the first time it's
called, not asserted in advance — per `references/upscale-settings.md`'s
confirmed-vs-unverified split.

## Error handling

- Ambiguous target image (multiple plausible candidates, none clearly "that image") → ask
  the user which one, don't guess.
- Local file not yet uploaded → call `upload_image` first; never fabricate an
  `image_upload_id`.
- Both `image_response_id` and `image_upload_id` resolved, or neither → resolve to exactly
  one before calling; the tool takes exactly one and the skill must not pass an invalid
  combination.
- `upscale_image` call fails → surface the failure verbatim per the tool's own STRICT
  instruction; never retry silently with different settings and never fall back to a
  different generation tool pretending it's an upscale.
- Response shape differs from what was assumed (e.g. missing an expected identifier field)
  → report what the response actually contains rather than asserting a field exists that
  wasn't observed.

## Save what you made

After a successful upscale, save the real identifiers (per step 4 of the Workflow above)
to the project's existing output location rather than leaving them only in the
conversation, per the toolkit's "No Context Lost" habit.

## Reference files

- `references/upscale-settings.md` — the identifier-resolution rule and the settings
  guide for every optional `upscale_image` parameter, plus the confirmed-vs-unverified
  split for this tool's response shape. Read before running step 1 or step 2.
