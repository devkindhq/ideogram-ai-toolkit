---
name: remove-background-workflow
description: Turns Ideogram's `remove_background` call into a guided workflow — resolve the source image, strip its background to a transparent PNG, save the result with its identifiers, and point to common follow-ups. Use when the user explicitly asks to "remove the background from this," "make this transparent," "cut out the subject," "isolate this on transparent," "strip the background," or "give me this on transparent/white/none." Only triggers on that kind of explicit request — it does not auto-run after `generate_image`, `edit_image`, or any other skill to proactively offer background removal.
---

# Remove Background Workflow

This skill orchestrates `remove_background` as its core call, pulling in `upload_image`,
`get_recent_generations`, and `get_generation_status` as supporting tools depending on how
the source image gets resolved. It's a thin orchestration workflow around a single MCP
call, not a prompt-composition skill — there's no `composition-spec-format.md`/
`panel-anatomy.md`-style reference here.

This skill handles exactly one image per invocation. If the user asks for several images at
once, run the workflow below once per image and report each result separately —
`remove_background` itself only accepts one source per call.

## Before you start

Read `references/background-removal-patterns.md` before running step 2 of the workflow
below. It covers the single-source rule (why `image_response_id` and `image_upload_id` can
never both be set), the private/public default, and how to handle the running/pending
envelope some calls return instead of an immediate result.

## Workflow

### 1. Resolve the source image

Figure out what the user means by "this image" before calling anything:

- **A response from earlier this session** — if the user is pointing at something just
  generated or edited, pull the `response_id` from `structured_content.response_ids` on the
  prior `generate_image`/`edit_image`/etc. call.
- **A freshly attached local file** — run the `upload_image` flow first to mint an
  `image_upload_id`: call `upload_image(filename=...)`, run the returned curl command in the
  sandbox, and read the response's `id` field — not `image_url` — as the `image_upload_id`.
- **A vague reference** ("the logo from earlier," "that last one") with no id yet in
  context — call `get_recent_generations` and match by description and recency. If more than
  one plausible candidate exists, ask the user which one they mean rather than guessing.

Never pass both `image_response_id` and `image_upload_id` in the same call — exactly one of
the two, based on which path above resolved the source.

### 2. Run background removal

Call `remove_background(image_response_id=... OR image_upload_id=..., private=...)`. Omit
`private` by default; only pass `private=False` if the user explicitly asks to publish the
result to the public Ideogram feed. Check the response for a running/pending status before
treating it as final — if present, poll via `get_generation_status(request_id=...)` the same
way a fresh generation is polled. See `references/background-removal-patterns.md` for why
this check exists and what was actually observed the first time this ran for real.

### 3. Save the output

Once the call completes, save the transparent PNG and record its identifiers — source
`response_id`/`image_upload_id`, output identifier, and the `private` flag used — to the
project's existing output location, the same folder `brand-identity-sheet`,
`character-model-sheet`, or `moodboard-generator` already save to for that project, per the
toolkit's "No Context Lost" habit. Confirm the file actually wrote before reporting success.

### 4. Note follow-ups

After saving, mention these without executing them unless the user asks:

- **Compositing onto a new background** via
  `edit_image(image_response_ids=[output_id], prompt="<new background description>")` —
  `remove_background` only strips the background, it doesn't add one.
- **Reframing or upscaling** the transparent result via `reframe_image`/`upscale_image`,
  using the same output identifier.
- **Filing into a collection** via `collections-management`'s `add_images_to_collection`, if
  that skill is already in use for the project — mention it as an option, don't call it
  automatically.

## Error handling

- Ambiguous source (multiple plausible recent images match a vague reference) → ask the
  user which one, don't guess.
- No resolvable source at all → ask the user to specify or attach an image; don't call
  `remove_background` with a guessed id.
- Both id types would otherwise apply (a `response_id` and an `image_upload_id` both look
  plausible) → ask the user which one, never pass both.
- `remove_background` call fails → surface the actual error to the user verbatim and stop;
  no fallback to a different tool and no fabricated workaround.
- Running/pending status never resolves after polling → report the stuck or failed state
  honestly, don't claim the output is ready.
- Save step fails → report the failure, don't claim the output was saved if it wasn't.

## Save what you made

After every successful removal, save the transparent PNG plus its source and output
identifiers and the `private` flag used to the project's existing output location, rather
than leaving identifiers only in the conversation, per the toolkit's "No Context Lost"
habit.

## Reference files

- `references/background-removal-patterns.md` — the single-source rule, the private/public
  default, and how to handle the running-envelope response. Read before running step 2 of
  the workflow above.
