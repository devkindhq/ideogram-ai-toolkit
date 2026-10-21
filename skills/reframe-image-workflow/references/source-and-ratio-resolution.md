# Source and Ratio Resolution

This file covers the two things `SKILL.md` points here before touching step 1 or step 2 of
the workflow: how each of the three realistic source paths resolves to the one id
`reframe_image` needs, and how a plain-language ratio request maps to a concrete
`aspect_ratio` string. It also separates what's actually confirmed about the connected
Ideogram MCP's `reframe_image` and collection tools from what isn't, per the standing rule
against stating third-party API behavior as fact without a verified source.

## Exactly one source

`reframe_image` accepts exactly one of `image_response_id` or `image_upload_id` — never
both, never neither. Three realistic paths lead to a resolved id:

**Generated this session** — the `response_id` lives in the `structured_content.response_ids`
field of the prior `generate_image`, `edit_image`, `remix_image`, or `upscale_image` call's
result. If the user references something not immediately visible in the current context
("reframe the logo from earlier"), call `get_recent_generations` to locate it. Pass the
resolved value as `image_response_id`.

**Local file on disk** — call `upload_image` with the file. The tool returns a single-use
`curl` command and an `id`. Run the `curl` command immediately — the upload URL is
short-lived, the same constraint `custom-model-training`-style dataset uploads document for
`upload_dataset_assets`. Pass the returned `id` as `image_upload_id`. If the `curl` command
fails because the URL expired, call `upload_image` again for a fresh one; never retry the
same dead command.

**From an existing collection** — call `get_images_by_collection_id` to locate the item.
Per the tool's own description (confirmed by direct inspection of the connected MCP's
tool schema, not carried from the design spec's prose): "Each item in the returned `assets`
list has either `response_id` (for AI-generated images) or `image_id` (for uploaded
images)." That means the primary, confirmed way to tell the two cases apart is which key
is present on the item — a `response_id` field means it's a generation candidate for
`image_response_id`; an `image_id` field means it's an upload candidate for
`image_upload_id`.

`SKILL.md` (this skill's own entry point) describes this same distinction using an
`asset_type` field with `GENERATION`/`UPLOAD` string values, inferred from
`collections-management`'s worked example rather than independently confirmed. That
`asset_type`/`GENERATION`/`UPLOAD` language does not appear in the live tool schema's own
description text, which documents key-presence (`response_id` vs. `image_id`) as the
distinguishing signal instead. This file flags that discrepancy explicitly rather than
quietly using different terminology from `SKILL.md`: treat the key-presence rule above as
the confirmed mechanism, and if the actual response also carries a literal `asset_type`
field (as it did in `collections-management`'s worked example), that's consistent
corroborating evidence, not a contradiction — but don't rely on `asset_type` being present
as a hard requirement, since it isn't documented on `get_images_by_collection_id`'s own
schema. Either signal, correctly read, resolves to the same id. If `reframe_image` rejects
an id resolved this way, treat that as a real tool-response error to surface to the user,
not a bug in this resolution logic to silently retry or paper over.

**Disambiguation rule** — if more than one candidate plausibly matches a vague source
reference (e.g. two moodboards generated earlier in the session), list the candidates with
enough detail to distinguish them (prompt snippet, creation order, filename) and ask which
one. Never guess.

## Aspect-ratio mapping table

| User request | `aspect_ratio` |
|---|---|
| Square / Instagram post | `1:1` |
| Instagram Story / Reel | `9:16` |
| Portrait | `4:5` |
| Landscape | `16:9` |
| YouTube thumbnail | `16:9` |
| Banner (wide web banner) | `21:9` |
| Widescreen desktop wallpaper | `16:9` |

The tool's own description states it "accepts either colon or x notation" (e.g. `9:16` or
`9x16`) — either is valid, this skill defaults to colon notation for consistency in what it
passes and what it records in `RESULT.md` files. If the user already gives an explicit
ratio, use it directly rather than round-tripping it through this table.

## Confirmed vs. unverified on ratio support

`reframe_image`'s `aspect_ratio` parameter is a free-form string with no enum in its tool
schema, unlike `generate_image`'s constrained 15-value list. It's reasonable to expect
similar ratios work, but this is not confirmed fact — this skill does not pre-validate the
ratio table above against a hardcoded "valid ratios" list client-side. Instead, pass through
whatever ratio the user confirms and report the tool's actual response (success, or a
rejection with its real error message) rather than asserting in advance that a given ratio
will or won't be accepted. This matches the standing rule against stating unverified
third-party API behavior as fact.

## What's confirmed vs. what to verify

**Confirmed (from direct inspection of the live connected MCP's tool schemas):**
- The exactly-one-source constraint: `reframe_image`'s own description states "Pass exactly
  one source" between `image_response_id` and `image_upload_id`.
- The free-form `aspect_ratio` string format, accepting colon or `x` notation.
- The `private` default behavior: paid accounts default to private, free/Basic accounts
  default to public, and Enterprise accounts are always private regardless of the flag.
- The optional `seed` parameter.
- `get_images_by_collection_id`'s items carry either a `response_id` (generated images) or
  an `image_id` (uploaded images) key — this is the tool's own documented distinguishing
  signal, confirmed directly from its schema description text.

**Not independently confirmed by this skill — verify from the tool's actual response the
first time it's exercised, don't assume:**
- Whether every ratio in the mapping table above is actually accepted by `reframe_image`.
- The `asset_type: GENERATION`/`UPLOAD` field and value pair that `SKILL.md` describes for
  collection items. The live tool schema documents key-presence (`response_id` vs.
  `image_id`) as the confirmed distinguishing mechanism instead; whether a literal
  `asset_type` field with those exact string values is also present on every response is
  inferred from `collections-management`'s worked example, not independently verified by
  this skill's own spec. Either reading resolves to the same source id, but don't assert
  the `asset_type` field's presence or exact values as confirmed fact.
