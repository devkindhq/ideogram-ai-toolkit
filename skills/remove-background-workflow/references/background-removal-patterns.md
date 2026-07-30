# Background Removal Patterns

This file covers what `SKILL.md` points here for before running step 2 of the
workflow: the single-source rule, the private/public default, and how to handle
the running/pending envelope some calls return instead of an immediate result. It
also covers the shape of the completed output and where compositing-style
follow-ups belong, separating what's actually confirmed about `remove_background`
from what's assumed pending a real run, per the standing rule against stating
third-party API behavior as fact without a verified source.

## Single-source rule: exactly one of `image_response_id` or `image_upload_id`

`remove_background` takes exactly one of `image_response_id` or `image_upload_id`
— never both. This is a narrower contract than `edit_image`, which accepts "at
least one, may combine both" and produces a single image conditioned on every
reference passed. `remove_background` isn't a composition call; it's a transform
applied to one specific source image, so there's no equivalent of "combine both
references" for it to fall back on. Passing both isn't a richer request the way
it might be for `edit_image` — it's an ambiguous one, because the tool has no
defined behavior for reconciling two different source images into one background
removal.

That ambiguity shows up in practice whenever a user's request could plausibly
resolve to either path at once — for example, they attach a file in the same
message where they also reference "the one I generated earlier." Both an
`image_upload_id` (from the attachment) and an `image_response_id` (from the
earlier generation) look valid at that point. The mitigation: don't guess which
one takes precedence and don't pass both. Ask the user which image they mean,
the same way `SKILL.md`'s error-handling section already directs for a vague or
multiply-matching reference.

## Private/public default: don't publish unless asked

The rule comes straight from the tool's own description: omitting `private`
uses the account's plan default — paid plans default to private, free/Basic
plans default to public, and Enterprise-tier generations are always private
regardless of what's passed. Because that default already varies by account
tier, this skill doesn't try to second-guess or override it — it just omits
`private` unless the user gives an explicit reason to set it.

The one reason to pass `private=False` is an explicit ask from the user to
publish the result to the public Ideogram feed. The failure mode this guards
against is a skill silently publishing a user's output because it defaulted to
"public" out of convenience, or because it read "the user didn't say private"
as license to publish. Absence of an instruction is not itself an instruction —
only publish on a direct request to do so.

## Running-envelope handling — confirmed by a real run (2026-07-30)

**Confirmed:** `remove_background` does return a running envelope, the same
shape as `edit_image`/`generate_image`. A real call against a real uploaded
logo (`skills/remove-background-workflow/examples/logo-background-removal/`)
returned `{"status": "running", "request_id": "t-W7Rk2tRLS5-34A3C-SsA",
"response_ids": ["X87m5cVHUfW2S7c2m23FNA"], "image_urls": [...],
"thumbnail_urls": [...], "permalink_urls": [...], ...}` immediately — no
transparent PNG data inline, and `status` was `"running"`, not a completed
result. Polling `get_generation_status(request_id="t-W7Rk2tRLS5-34A3C-SsA")`
was required: the first poll still showed `"status": "running"` for that row,
and a second poll returned `"status": "done"` with the same `response_id`
(`X87m5cVHUfW2S7c2m23FNA`) now populated on that row. Two polls were needed in
this run before the job resolved.

Practical note: the `response_ids` array is present in the envelope even while
`status` is still `"running"` — it names the eventual output id in advance,
but the row for that `request_id` in `get_generation_status` isn't marked
`"done"` until the job actually finishes. Don't treat the presence of
`response_ids` in the initial envelope as proof the image is ready; check
`status` (or poll `get_generation_status` until its row for that `request_id`
shows `"done"`) before downloading or handing the id to another tool.

The image itself is served as `image/webp` at the `image_urls`/`image_urls[n]`
URL (`https://ideogram.ai/assets/image/balanced/response/<response_id>@2k`)
regardless of the `.png` naming convention used elsewhere — passing an
`Accept: image/png` header did not change this. To save a real PNG on disk,
download the URL and convert locally (e.g. `sips -s format png`), rather than
assuming the raw bytes are already PNG-encoded.

## Output-identifier shape — confirmed by a real run (2026-07-30)

**Confirmed:** `remove_background`'s completed response uses `response_id`,
exactly matching the field name every other generation-type tool in this
toolkit (`generate_image`, `edit_image`, `upscale_image`, `reframe_image`)
uses. It's surfaced the same way: `response_ids` (plural array, one entry) on
the running/initial envelope, and `response_id` (singular) on the resolved row
returned by `get_generation_status`. In this run the real value was
`X87m5cVHUfW2S7c2m23FNA`.

That id chains into `edit_image` exactly as assumed: calling
`edit_image(image_response_ids=["X87m5cVHUfW2S7c2m23FNA"], prompt="place this
logo mark on a warm sunset gradient background")` was accepted and produced a
new running envelope (`request_id: xBM-QEmeSa29GcFk9XfoMQ`) that resolved to
`response_id: _PBjo4RLV8Ost167OxQnDQ` after polling — confirming the
compositing follow-up documented below actually works with the output
identifier from a real `remove_background` call.

## Compositing and other follow-ups: mention, don't auto-invoke

`remove_background` only removes a background — it never adds one. There is no
argument or mode that puts a new background behind the cut-out subject as part
of this call. Producing a new background is always a separate step:
`edit_image(image_response_ids=[output_id], prompt="<new background
description>")`, using the transparent output's identifier as the reference.
Composing that follow-up prompt well — describing the new background, matching
lighting or style to the subject — is a prompt-composition concern that belongs
to skills built for that (`ideogram-prompt`, `logo-prompting`-style skills), not
to this workflow. This skill's job stops at producing a clean transparent
result and identifying that a compositing step exists; it doesn't try to write
that follow-up prompt itself.

The same "mention, don't auto-invoke" treatment applies to two other optional
follow-ups on the same output identifier:

- **Reframing or upscaling** the transparent result via `reframe_image` or
  `upscale_image`, passed the same output identifier.
- **Filing the result into a collection** via `collections-management`'s
  `add_images_to_collection`, if that skill is already in use for the project.

All three — compositing, reframe/upscale, and filing into a collection — get
named to the user as available next steps after a successful removal. None of
them get called automatically. The reasoning is the same one `SKILL.md`'s
"Note follow-ups" step already states: this is a thin orchestration workflow
around one call, not a skill that chains itself into other tools' territory
without being asked.
