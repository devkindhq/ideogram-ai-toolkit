# Remove Background Workflow Skill — Design

## Why

`mcp__ideogram__remove_background` is a confirmed-live tool on the connected Ideogram MCP
(verified via direct tool-schema inspection, not assumed from docs) with no wrapping skill
anywhere in this repo. It's an uncontested, low-competition gap — unlike the Tier 1 skills
(`custom-model-training`, `collections-management`), there's no differentiation angle to
argue for; the tool itself is simple (one image in, one transparent PNG out) and the gap is
that nothing in the toolkit turns it into a guided, repeatable workflow with the repo's
usual identifier-saving and follow-up discipline.

The core use case: a user has a generated or uploaded image and wants the subject isolated
on a transparent background, usually as a step toward something else — compositing onto a
new background, dropping into a mockup, or using as a clean asset outside Ideogram
entirely. Left unwrapped, this is a single raw tool call with no guidance on which source
identifier to pass, no saved record of the output, and no pointer to the natural next step
(`edit_image` to composite onto a new background), which is exactly the kind of gap the
other Tier 2 skills in this repo already close for their respective tools.

## Scope

One new skill: `skills/remove-background-workflow/`. Triggers only on explicit requests to
remove or isolate a background — "remove the background from this," "make this
transparent," "cut out the subject," "isolate this on transparent," "strip the background."

Single-image workflow. The brief specifies "resolve the source image" (singular) and
`remove_background` itself only accepts one source per call — this skill does not add a
batching/looping layer over multiple images in one invocation. If the user wants several
images processed, the skill runs the same single-image workflow once per image and reports
each result, rather than building bulk orchestration this tool doesn't natively support.

No compositing logic is implemented inside this skill. `remove_background` only removes
backgrounds; producing a *new* background is a separate `edit_image` call with its own
prompt-composition concerns that belong to `ideogram-prompt`/`logo-prompting`-style skills,
not duplicated here. This skill documents compositing as the most common follow-up and
hands off the resulting `response_id`, but does not perform the composite itself.

## Assumptions

Two things aren't stated in the `remove_background` tool description and aren't verifiable
without an actual call, so they're documented here rather than asserted as fact per the
standing rule against stating third-party API behavior without a verified source:

1. **Sync vs. async response shape.** `edit_image`'s description explicitly says it
   "returns a `running` envelope just like a fresh generation" that the caller polls via
   `get_generation_status`. `remove_background`'s description says only that it returns a
   transparent PNG, with no mention of a running/pending state. Because it's still a
   model-backed transform (not a metadata-only operation), this skill assumes it *may*
   follow the same running-envelope pattern and instructs checking the response for a
   status field before treating it as complete — see Data flow step 2. If the response is
   in fact always synchronous, the poll step is a no-op (nothing to poll) and costs nothing.
2. **Output identifier shape.** This skill assumes the completed response carries a
   `response_id` (or equivalent) usable as `image_response_ids` input to `edit_image`,
   `reframe_image`, or `upscale_image`, consistent with every other Ideogram generation-type
   tool in this toolkit. If the actual response shape differs, `references/` should be
   corrected the first time this skill is run for real (matching how `collections-management`
   and `custom-model-training` were built from one real worked example, not written blind).

## Architecture

Structural pattern matches `collections-management` and the other existing skills:
`SKILL.md` + `references/` + `examples/` + `evals/evals.json` under
`skills/remove-background-workflow/`.

This is a thin orchestration workflow around a single MCP call, not a prompt-composition
skill — there is no `composition-spec-format.md`/`panel-anatomy.md`-style reference here,
matching how `collections-management` also skipped that pattern for the same reason (no
prompt is being composed; the tool takes an image reference and a flag, not a text prompt).

One unified skill covering the full arc — resolve source, run removal, save output, point
to follow-ups — rather than splitting "removal" and "follow-up compositing" into two
skills. The compositing step is explicitly out of this skill's execution scope (see Scope),
so the split would only add a discovery decision without adding capability.

## Components

- **`SKILL.md`** — frontmatter triggers on: "remove the background," "make this
  transparent," "cut out this image," "isolate the subject on transparent," "strip the
  background," "give me this on transparent/white/none." Body documents the 4-step
  workflow (below) and points to `references/` before step 1.
- **`references/background-removal-patterns.md`** — the exactly-one-source rule (why
  `remove_background` differs from `edit_image`: it takes exactly one of
  `image_response_id` / `image_upload_id`, never both, unlike `edit_image` which can
  combine multiple references), the private/public default rule (omit `private` unless
  the user explicitly asks to publish), the running-envelope handling from the Assumptions
  note above, and the compositing follow-up pattern via `edit_image`.
- **`examples/`** — one worked example, produced by actually running the pipeline once
  (resolve a real source image, remove its background, save the transparent PNG with
  identifiers, and show the optional composite follow-up).
- **`evals/evals.json`** — 3 eval prompts in the existing schema (`id`, `eval_name`,
  `prompt`, `expected_output`, `files`), matching `collections-management/evals/evals.json`'s
  format.

## Data flow / workflow

1. **Resolve the source image** — determine what the user means by "this image":
   - A `response_id` from something generated/edited earlier this session
     (`structured_content.response_ids` on a prior `generate_image`/`edit_image`/etc. call).
   - A freshly attached local file → run the `upload_image` flow first to mint an
     `image_upload_id`, per that tool's own instructions.
   - A vague reference ("the logo from earlier," "that last one") with no id yet in
     context → call `get_recent_generations` and match by description/recency; if more
     than one plausible candidate exists, ask the user which one rather than guessing.
   Never pass both `image_response_id` and `image_upload_id` in the same call — the tool
   takes exactly one source, unlike `edit_image`'s "at least one, may combine both."
2. **Run background removal** — call `remove_background(image_response_id=... OR
   image_upload_id=..., private=...)`. Omit `private` by default (uses the account's plan
   default); only pass `private=False` if the user explicitly asks to publish the result to
   the public Ideogram feed. Check the response for a running/pending status before treating
   it as final; if present, poll via `get_generation_status` the same way a fresh generation
   is polled (see Assumptions — this may be a no-op if the call is actually synchronous).
3. **Save the output** — once complete, save the transparent PNG and record its identifiers
   (source `response_id`/`image_upload_id`, output `response_id`, and `private` flag used)
   to the project's existing output location — the same folder `brand-identity-sheet`,
   `character-model-sheet`, or `moodboard-generator` already save to for this project, per
   the toolkit's "No Context Lost" habit. Confirm the file actually wrote before reporting
   success.
4. **Note follow-ups** — after saving, mention the common next steps without executing them
   unless asked:
   - **Composite onto a new background** — `edit_image(image_response_ids=[output_id],
     prompt="<new background description>")`, since `remove_background` only strips the
     background, it doesn't add one.
   - **Reframe or upscale** the transparent-background result via `reframe_image` /
     `upscale_image` using the same output `response_id`.
   - **File into a collection** — if `collections-management` is in use for this project,
     mention `add_images_to_collection` as an option; do not call it automatically, matching
     that skill's own explicit-trigger-only scope.

## Error handling

- Ambiguous source (multiple plausible "recent" images, no clear single match) → ask the
  user which one, don't guess.
- No resolvable source at all (nothing recent, nothing uploaded, no id given) → ask the
  user to specify or attach an image, don't call `remove_background` with a guessed id.
- Both `image_response_id` and `image_upload_id` would otherwise apply (e.g., user both
  uploaded a file and referenced an earlier generation) → ask which one they mean; never
  pass both.
- `remove_background` call fails (e.g., no clear subject, unsupported image) → surface the
  actual error to the user verbatim and stop. Do not fall back to a different tool or
  fabricate a workaround, matching the strict-failure-handling language already used on
  `edit_image` in this MCP.
- Running/pending status never resolves after polling → report the stuck/failed state
  honestly; don't claim the output is ready.
- Save step fails (disk write error, path doesn't exist) → report the failure; don't claim
  the output was saved if it wasn't.

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts:

1. "Remove the background from this product shot I just generated" — resolves a
   `response_id` from the current session, calls `remove_background`, saves output with
   identifiers.
2. "I uploaded a logo, can you make it transparent" — runs the `upload_image` flow first to
   get an `image_upload_id`, then `remove_background` with that id, verifying the skill
   doesn't also pass a stale `image_response_id` from earlier in the conversation alongside
   it.
3. "Remove the background from this, then put it on a beach scene" — verifies the skill
   completes the removal, saves the transparent output, and hands off its `response_id` to
   a follow-up `edit_image` composite call rather than trying to add a background inside
   `remove_background` itself (which the tool doesn't support).

## Out of scope (this spec)

- Batch/bulk background removal across many images in one invocation — v1 runs the
  single-image workflow once per image if several are requested.
- Compositing logic (choosing/generating the new background) — documented as a follow-up
  via `edit_image`, not implemented inside this skill.
- Automatic collection filing — mentioned as an option, never auto-invoked, matching
  `collections-management`'s own explicit-trigger-only scope.
- Any dedicated UI/preview step beyond saving the file and reporting identifiers.
