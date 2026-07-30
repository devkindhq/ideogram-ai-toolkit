# Upscale Settings

This file covers the two things `SKILL.md` points here for before running step 1 or step
2 of the workflow: the identifier-resolution rule (why `upscale_image` needs exactly one
of `image_response_id` / `image_upload_id`, never both, never neither) and a settings
guide for every optional `upscale_image` parameter. It also separates what's actually
confirmed about the connected Ideogram MCP's `upscale_image` tool from what isn't, per the
standing rule against stating third-party API behavior as fact without a verified source.

## Identifier resolution: exactly one, never both, never neither

Every `upscale_image` call must pass exactly one of `image_response_id` or
`image_upload_id`. On the live connected MCP's tool schema, both parameters are
independently nullable strings — there's no `oneOf`/`required` enforcement visible on the
schema that would reject an invalid combination on the tool's side. That means the "exactly
one" rule isn't something the tool enforces for you; the skill is the only thing enforcing
it, and it has to resolve to exactly one identifier before calling, never leave it to the
tool to guess.

Where each identifier comes from:

- **`image_response_id`** — from a `response_id` already produced this session. Sources:
  `generate_image`/`remix_image`/`edit_image`/`reframe_image`/`remove_background`'s
  `structured_content.response_ids`, `get_recent_generations`'s per-item `response_id`, or
  `get_images_by_collection_id`'s per-asset `response_id`.
- **`image_upload_id`** — from `upload_image`'s curl-response `id`, for a local file that
  isn't an uploaded asset yet. This is only reachable by actually calling `upload_image`
  and reading its returned `id` back — never fabricated.

**Failure mode this prevents:** passing both produces an ambiguous request the tool wasn't
designed to disambiguate — there's nothing in the schema saying which one wins. Passing
neither leaves the tool with no image to upscale at all. Either way, the failure surfaces
at the tool boundary, not before, which is exactly why the skill has to do the resolution
itself rather than pass through whatever it happens to have on hand.

## Settings guide

One subsection per optional `upscale_image` parameter — what it does, and the rule for
when to set it versus omit it for the backend default.

### `upscale_factor`

Enum `X1`/`X2`/`X4`/`X8`. Defaults to `X2` per the tool's own default. Set it when the user
states a factor ("upscale this 4x" → `X4`). When the user doesn't specify one, omit it and
let the `X2` default apply — but state that default plainly before calling. This does not
require a blocking confirmation question, per the Assumptions rule in the design spec: a
stated default is not the same as a guessed value, so it doesn't need to be gated behind a
question the way an ambiguous target image does.

### `upscale_details_weight`

Integer 1-100, controls how much detail the upscaler adds, per the tool's own description.
Only set it when the user explicitly asks for more or less added detail. Otherwise omit it
for the backend default — there's no basis for picking a number in this range on the
user's behalf when they haven't expressed a preference.

### `prompt`

Optional. Per the tool's own description, the backend describes the image automatically
when no prompt is supplied. Only set it when the user wants to steer what detail gets
added — e.g., they want the upscaler to sharpen a specific element rather than whatever it
would infer on its own. Otherwise omit it; the auto-description is the tool's documented
fallback, not a gap the skill needs to fill.

### `collection_id`

Optional; saves the upscaled image into an existing collection. Only set it when the user
asks to save directly into an existing collection. Resolving which `collection_id` to pass
follows `collections-management`'s find-or-create pattern from `collection-patterns.md`
(call `list_collections`, match by name) rather than inventing one — the same reasoning
applies here as there: guessing an ID risks filing the upscale into the wrong collection
silently.

### `private`

Per the tool's own description, omitting this uses the account's plan default: paid
accounts default to private, free/Basic accounts default to public, and enterprise
generations are always private regardless. Pass `False` only when the user explicitly asks
to publish to the public Ideogram feed. Don't pass `True` speculatively to "be safe" — that
overrides an account-tier default the user didn't ask to override.

### `seed`

Optional integer. The tool's own description doesn't specify its exact effect on upscaling
beyond accepting an integer — it isn't documented as controlling determinism or
reproducibility for this tool the way seeds sometimes do elsewhere. State only what's
confirmed (it's an accepted parameter) and don't assert unconfirmed behavior about
determinism or reproducibility. Only set it if the user supplies a specific seed value
themselves; don't invent one on their behalf.

## What's confirmed vs. what to verify

**Confirmed (from direct inspection of the live connected MCP's tool schema on
2026-07-30):** the full parameter list (`image_response_id`, `image_upload_id`,
`upscale_factor` with its 4-value enum and `X2` default, `upscale_details_weight`,
`prompt`, `collection_id`, `private`, `seed`) and the tool's STRICT no-fallback-on-failure
instruction — these are read directly off the live schema/description, not assumed from
docs.

**Unverified / unknown — don't assert these as fact:** the exact shape of the tool's
*response*. The description documents the input schema in full but not the output —
whether there's a downloadable URL alongside a `response_id`, the field name for it if so,
processing time, exact output pixel dimensions per factor, and any cost/rate-limit
implications of `X8` vs `X2` are all unconfirmed. Read the response's actual fields the
first time `upscale_image` is called in a session, and report what was actually observed
rather than a plausible-sounding assumed value.

## Why this matters

Both failure modes above are silent at first and only become visible once it's too late to
fix cheaply. Passing an invalid identifier combination either upscales the wrong image or
gets rejected by the tool — neither is caught by the schema itself, so the only backstop is
the skill resolving to exactly one identifier before calling. Asserting an unconfirmed
response field is the same shape of problem on the output side: reporting a save location
or field that doesn't actually exist in the response leaves the user's real output
identifiers unrecorded even though the skill claimed success. In both cases the failure
looks fine in the moment and only surfaces later, when the user goes looking for an image
that was never actually saved where they were told it was.
