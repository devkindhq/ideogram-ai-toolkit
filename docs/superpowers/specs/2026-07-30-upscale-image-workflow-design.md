# Upscale Image Workflow Skill — Design

## Why

Gap-research (conducted 2026-07-30) flagged `upscale_image` as a Tier 2 gap: uncontested and low-competition, no
prior art to differentiate against and no differentiation angle required. The connected
Ideogram MCP already exposes the tool needed: `mcp__ideogram__upscale_image` (confirmed
live via direct tool-schema inspection, not assumed from docs).

The core use case: a user has an image from earlier in the session (a `generate_image`,
`remix_image`, `edit_image`, `reframe_image`, or `remove_background` result), from
`get_recent_generations`, from a collection, or a local file they want uploaded, and wants
a higher-resolution version of it saved with its identifiers so it can be found again —
the same "don't lose track of what got made" problem `collections-management` solves for
organizing images, applied here to a single upscale call.

## Scope

One new skill: `skills/upscale-image-workflow/`. Triggers on explicit requests to upscale,
enlarge, or increase the resolution of a specific image ("upscale this," "make this
higher-res," "upscale that logo render 4x," "increase the resolution on the image I just
uploaded"). It wraps exactly one MCP tool (`upscale_image`) in a guided workflow: resolve
the target image, confirm the settings, run it, save the output's identifiers.

It does not wrap `remix_image`, `reframe_image`, `edit_image`, or `remove_background` —
those are separate tools with separate concerns, already out of scope per the tool STRICT
instruction on `upscale_image` itself: "upscale" means raise the resolution of *this*
image, nothing else, and the skill must not substitute another generation tool if the
upscale call fails.

## Assumptions

Two non-obvious calls made without a human to check with mid-task:

1. **"Confirm upscale factor/settings" means stating the resolved settings before
   running, not blocking on a yes/no every time.** If the user's request already specifies
   a factor ("upscale this 4x"), that counts as confirmed — no need to re-ask. If it
   doesn't ("upscale this"), the skill states it's using the X2 default (the tool's own
   default) before calling, rather than blocking on a confirmation question for a
   reversible, cheap default. The skill only asks a real question when resolving *which
   image* is ambiguous (see Data flow, step 1) — that's the one place guessing wrong is
   costly (upscaling the wrong image) versus low-cost (running with the default factor).
2. **"Save output with identifiers" means persisting the returned identifiers (response
   ID, resolved `upscale_factor`, source image identifier) to the project's existing local
   output location, matching how the other five skills already save their output** —
   not necessarily downloading and storing upscaled pixel data locally. The tool's
   description documents its *input* schema in full but not its output/response shape, so
   the exact field name(s) the upscaled image comes back under (e.g. whether there's a
   downloadable URL alongside a `response_id`) aren't confirmed ahead of time. The skill
   reads whatever fields the live response actually contains the first time it's called
   and saves those, rather than asserting an unverified response shape now.

## Architecture

Structural pattern matches the existing six skills: `SKILL.md` + `references/` +
`examples/` + `evals/`. Like `collections-management` and `custom-model-training`, this is
an **orchestration workflow** around MCP calls, not a prompt-composition skill — no
`composition-spec-format.md`/`panel-anatomy.md`-style reference here.

It's the thinnest skill in the repo: one tool, one call per invocation (no multi-step
pipeline like `custom-model-training`'s create-dataset → upload → train → poll chain, and
no multi-tool surface like `collections-management`'s seven tools). The guided-workflow
value is in resolving the target image correctly, surfacing settings before running, and
not losing the output's identifiers afterward — not in orchestrating many calls.

## Components

- **`SKILL.md`** — frontmatter written to trigger on: "upscale this," "upscale that
  render," "make this higher resolution," "increase the resolution," "upscale my upload
  4x." Body documents the 4-step workflow (below).
- **`references/upscale-settings.md`** — the identifier-resolution rule (`upscale_image`
  requires *exactly one* of `image_response_id` or `image_upload_id` — never both, never
  neither) and a settings guide: what each optional parameter (`upscale_factor`,
  `upscale_details_weight`, `prompt`, `collection_id`, `private`, `seed`) does and when to
  set it versus omit it and let the backend default apply. Also carries the "what's
  confirmed vs. unverified" split for this tool, per the standing rule against stating
  third-party API behavior as fact without a verified source (see Assumptions above).
- **`examples/`** — one worked example, produced by actually running the pipeline once
  (upscale a real generated image, confirm the result via the response's own identifiers).
- **`evals/evals.json`** — 3 eval prompts in the schema `collections-management/evals/evals.json`
  uses: `id`, `eval_name`, `prompt`, `expected_output`, `files`.

## Data flow / workflow

1. **Resolve the target image** — determine exactly one identifier to pass:
   `image_response_id` (from this session's `generate_image`/`remix_image`/`edit_image`/
   `reframe_image`/`remove_background` `structured_content.response_ids`, from
   `get_recent_generations`, or from `get_images_by_collection_id`) or `image_upload_id`
   (from `upload_image`, for a local file the user points to that hasn't been uploaded
   yet — call `upload_image` first rather than inventing an ID). If more than one
   candidate image is plausibly "that image" (e.g. several renders made earlier this
   session with nothing distinguishing which one the user means), ask which one — don't
   guess. Never pass both identifiers or neither.
2. **Confirm settings** — state the resolved settings before calling: `upscale_factor`
   (the user's stated value, or the X2 default per the tool's own default — see
   Assumptions above for why this doesn't block on a question), `upscale_details_weight`
   (only set if the user asked for more/less added detail; otherwise omit for the backend
   default), `prompt` (only set if the user wants to steer what detail gets added;
   otherwise omit and let the backend auto-describe the image), `collection_id` (only if
   the user asked to save directly into an existing collection), `private` (omit unless
   the user explicitly asks to publish to the public Ideogram feed).
3. **Run the upscale** — call `upscale_image` with the resolved identifier and settings.
   Per the tool's own STRICT instruction: if the call fails, surface the failure to the
   user verbatim and stop. Never fall back to `generate_image`, `remix_image`, or any
   other tool as a substitute for a failed upscale.
4. **Save output with identifiers** — persist the response's identifiers (whatever the
   live response actually contains — at minimum expect a `response_id`, per the pattern
   every other Ideogram generation tool in this toolkit follows), the resolved
   `upscale_factor`, and the source image's identifier to the project's existing output
   location — the same place `brand-identity-sheet`, `character-model-sheet`,
   `moodboard-generator`, or `collections-management` already save to for this project.
   If a `collection_id` was passed in step 2, note that the upscaled image was also filed
   there.

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
  wasn't observed, per the Assumptions note on the unverified response shape.

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts:
1. "Upscale this logo render 4x" — resolves a single unambiguous `image_response_id` from
   earlier in the session, confirms `upscale_factor=X4` (user-specified, no need to ask),
   runs the call, and saves the returned identifiers.
2. "Upscale the image I just uploaded, and add a lot of extra detail" — resolves
   `image_upload_id` from a prior `upload_image` call (not `image_response_id`), sets
   `upscale_details_weight` high per the request, and confirms it only set that parameter
   because the user specifically asked for more detail.
3. "Upscale that" with two unrelated renders made earlier in the same session and no
   distinguishing detail in the request — verifies the skill asks which image instead of
   guessing, per the ambiguous-target rule in Data flow step 1.

## Out of scope (this spec)

- Wrapping `remix_image`, `reframe_image`, `edit_image`, or `remove_background` — separate
  tools, separate skills if/when those gaps are addressed.
- Batch/multi-image upscaling in a single skill invocation — `upscale_image` is a
  single-image tool; v1 of this skill matches that, one call per invocation.
- Automatically upscaling output from other skills (e.g. auto-upscaling every
  `brand-identity-sheet` render) — explicit-request trigger only, matching the
  `collections-management` precedent of not auto-wiring into the other skills.
- Asserting undocumented facts about `upscale_image`'s behavior (processing time, exact
  output pixel dimensions per factor, any cost/rate-limit implications of `X8` versus
  `X2`) — none of these are confirmed from the tool's own schema/description, so the skill
  doesn't state them as fact.
