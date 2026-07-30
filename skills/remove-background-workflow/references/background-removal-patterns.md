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

## Running-envelope handling — unverified, confirm during first real run

**What's confirmed (from the tools' own description text):** `edit_image`'s
tool description explicitly states that it "returns a `running` envelope just
like a fresh generation," to be polled via `get_generation_status`.
`remove_background`'s tool description says only that it "returns a transparent
PNG" for a single image/upload source plus a `private` flag — it does not say
one way or the other whether a running/pending state is ever part of that
response.

**What's assumed, not confirmed:** `remove_background` is a model-backed
transform, not a metadata-only operation like the collection tools in
`collections-management`. Because model-backed calls elsewhere in this toolkit
(`edit_image`, `generate_image`) do return a running envelope, this skill
assumes `remove_background` *may* follow the same pattern until proven
otherwise, and checks the response for a status field before treating it as
complete rather than assuming the PNG is always returned synchronously.

**This is [unverified as of the design spec — confirm during the first real
run, see below].** It stays an open question until a live call is actually made
and the real response shape is inspected. Once that run happens (planned as a
later task in this implementation, not this file), replace this section with
what was actually observed — either a synchronous PNG return with no running
state, or a genuine running envelope requiring a poll via
`get_generation_status` — and update `SKILL.md`'s workflow step 2 language to
match reality instead of this assumption.

## Output-identifier shape — unverified, confirm during first real run

**What's assumed:** every other generation-type tool in this toolkit
(`generate_image`, `edit_image`, `upscale_image`, `reframe_image`) returns a
`response_id` (surfaced via `structured_content.response_ids`) that can be fed
back in as `image_response_ids` to a later call. This skill assumes
`remove_background`'s completed response carries an equivalent identifier —
named `response_id` or something functionally the same — usable the same way to
chain into `edit_image`, `reframe_image`, or `upscale_image` on the transparent
result.

**This is likewise unverified.** Nothing has directly inspected
`remove_background`'s actual completed response shape yet. Don't assert the
field name as fact in the meantime — treat it as a plausible assumption based on
pattern consistency with the rest of the toolkit, not as something read off a
real response. Once a real call is made and the response is inspected, correct
this section to name the actual field the completed response uses, the same way
this whole section should be replaced rather than patched once real data exists.

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
