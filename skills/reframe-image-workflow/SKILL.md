---
name: reframe-image-workflow
description: Retargets an existing Ideogram image — generated this session, uploaded from disk, or filed in a collection — to a new aspect ratio or canvas via `reframe_image`. Use whenever the user asks to "reframe this image," "change the aspect ratio," "resize the canvas," "make this a [square/story/banner/portrait]," "crop this to fit [platform]," or "extend this image to [ratio]." Only triggers on explicit request; does not auto-run after `brand-identity-sheet`, `character-model-sheet`, `moodboard-generator`, or `collections-management`.
---

# Reframe Image Workflow

Ideogram's `reframe_image` tool retargets an existing image — one generated this session, uploaded
from disk, or retrieved from a collection — to a different aspect ratio or canvas size. This skill
orchestrates that single tool call in a guided 4-step workflow: resolve the source, confirm the
target ratio, run the reframe, and save the output.

This is an orchestration workflow around one MCP tool, not a prompt-composition skill — there is no
`composition-spec-format.md` or `panel-anatomy.md`-style reference here, since the "prompt" is
just an aspect-ratio string. The skill accepts exactly one source id (`image_response_id` XOR
`image_upload_id`, never both, never neither) and one `aspect_ratio` string per invocation. Version
1 is single-image, single-ratio: multiple target ratios of the same source are run as sequential
repeats of this workflow, not batched in one call.

## Before you start

Read `references/source-and-ratio-resolution.md` before running step 1 or step 2 below. It covers
the source-id resolution rules (how to distinguish between `image_response_id` and `image_upload_id`
candidates, and when to ask for clarification), the aspect-ratio mapping table (translating user
requests like "Instagram Story" or "9:16" to the tool's ratio format), and the platform-to-ratio
cheat sheet for common reframing requests.

## Workflow

### 1. Resolve the source image

Exactly one source id must be resolved — either `image_response_id` or `image_upload_id`, never
both, and never neither. Three realistic paths lead to a resolution:

**Generated this session** — If the user references an image from an earlier call in the same
session (e.g., "reframe the logo from step 1," "the one we just generated"), pull the `response_id`
from the relevant prior `generate_image`, `edit_image`, `remix_image`, or `upscale_image` call's
`structured_content.response_ids`. If the generation is not immediately in context (user says "the
logo from earlier but I can't see it in the chat"), call `get_recent_generations` with a reasonable
limit to find it by prompt snippet or timestamp. Pass the `response_id` as `image_response_id`.

**Local file** — If the user wants to reframe an image from their own disk (not a generation from
this session, not something in a collection), call `upload_image` first. Follow its response with
the single-use `curl` command it returns immediately (do not delay or batch this step), and pass the
returned `id` as `image_upload_id` to the next step.

**From an existing collection** — If the user references an image they filed earlier in a
collection (e.g., "the logo in my Brand Assets collection"), call `get_images_by_collection_id` to
browse the collection and locate the image. Read the returned `asset_type`: treat items tagged
`GENERATION` as `image_response_id` candidates, and items tagged `UPLOAD` as `image_upload_id`
candidates (this mapping is inferred from the collection API's behavior, not independently
confirmed). If multiple items plausibly match a vague reference (e.g., three logos in the
collection), list them with distinguishing detail — prompt snippet, creation timestamp, filename —
and ask the user which one they mean instead of guessing.

If neither `image_response_id` nor `image_upload_id` is resolved at the end of this step, stop and
ask the user to clarify which image they want to reframe.

### 2. Confirm the target aspect ratio

The user's request for a new ratio must be translated into a concrete `aspect_ratio` string (e.g.,
`"9:16"` or `"16:9"`; colon or `x` notation both accepted by the tool). Three cases:

**Explicit ratio given** — If the user provides a number directly ("9:16," "1:1," "3x2"), use it as-
is. Both colon and `x` notation are valid per the tool's own description.

**Named format given** — If the user references a format by name ("Instagram Story," "YouTube
thumbnail," "square," "banner"), translate it to a ratio using the mapping table in
`references/source-and-ratio-resolution.md` before proceeding. If the reference is ambiguous
("make it wider" or "fit for social"), ask for a specific ratio or a named platform rather than
choosing one speculatively.

**Ambiguous request** — If the user's request is vague and you cannot map it to a concrete ratio
(e.g., "make it different," "try some options"), ask them to name a target ratio or format instead
of inventing a number.

Never call `reframe_image` until the exact target `aspect_ratio` string is confirmed.

### 3. Run reframe_image

Call the `reframe_image` tool with the one resolved source id and the confirmed `aspect_ratio`.

- Always pass exactly one of: `image_response_id` (for generated/collection images) or
  `image_upload_id` (for uploaded files). Never both.
- Always pass the confirmed `aspect_ratio` string.
- Pass `seed` only if the user explicitly requests a reproducible variant or specific seed number.
  Otherwise, omit it.
- Pass `private` only if the user asks to override the account's plan default. Do not set it
  speculatively. Per the tool's own description: paid accounts default to private, free/Basic
  accounts default to public, and Enterprise accounts are always private regardless of the flag.

### 4. Save the output

Following the toolkit's "No Context Lost" principle, save the new image, its `response_id`, the
source id it was reframed from, the `aspect_ratio` used, and any `seed` passed to the project's
existing asset folder. Match the save location to wherever the source image's project already
stores output — for example, alongside `brand-identity-sheet` or `moodboard-generator` output if
the reframe is part of that same project. If no existing project folder applies, create a
`reframes/` subfolder next to the source image and save there. Include both the `response_id`
returned from `reframe_image` and the metadata (source id, ratio, seed if provided) in the same
file or a metadata entry.

Do not leave the new `response_id` only in the conversation — save it to a file so you can retrieve
it later without re-running the reframe.

## Error handling

- **Both or neither source id resolved** — If you reach the end of step 1 without exactly one source
  id, stop and ask the user to clarify which image they want to reframe. Do not call `reframe_image`
  with ambiguous or missing input.
- **Ambiguous source reference** — If the user's description matches multiple candidates (e.g.,
  "the logo" when there are three logos in the collection), list each candidate with
  distinguishing detail — a prompt snippet, creation timestamp, or filename — and ask which one
  they mean rather than guessing.
- **Malformed or ambiguous target ratio** — If the user's ratio request cannot be mapped to a
  concrete string (e.g., "make it wider" with no number or platform name), confirm the exact ratio
  before calling the tool. Do not invent a number for a vague request.
- **Stale or expired upload URL** — If you executed `upload_image` earlier but the returned
  single-use `curl` command expired or failed (e.g., network timeout), call `upload_image` again
  for a fresh URL and execute the new command immediately. Do not retry the dead `curl` command.
- **Tool error or failure status** — If `reframe_image` returns an error or a failure status (e.g.,
  insufficient credits, malformed parameters), surface the real error message to the user. Do not
  report the reframe as successful without an actual `response_id` returned from the tool pointing
  to the new image.

## Out of scope

This skill does not:

- Batch multiple images or multiple ratios in one guided pass. Run the 4-step workflow sequentially
  for each image–ratio combination.
- Auto-select or guess a target ratio without confirmation from the user.
- Perform content-level editing, inpainting, or outpainting — that is the domain of `edit_image`
  and `remix_image`, different tools with different workflows.
- Hardcode or enforce a closed set of "valid" ratios client-side. The tool's aspect-ratio parameter
  is flexible; the reference file documents common formats, but the user can pass any ratio string.

## Reference files

- `references/source-and-ratio-resolution.md` — Source-id resolution rules, aspect-ratio mapping
  table (translating format names to ratio strings), and platform-to-ratio cheat sheet. Read before
  running step 1 or step 2 of the workflow.
