# Variation Strategy

This file covers what `SKILL.md` points here for before drafting the prompt array
(workflow steps 2 and 3): which fields of the locked JSON caption are allowed to vary
across a `generate_images_bulk` batch, which aren't, and how to judge batch size once
the variation axis is chosen. It also separates what's actually confirmed about the
connected `generate_images_bulk` tool from what isn't, per the standing rule against
stating third-party API behavior as fact without a verified source.

## Lock style, vary composition

Every prompt submitted in one `generate_images_bulk` call must carry the identical
`style_description` object, byte-for-byte — the same `aesthetics`, `lighting`,
`medium`, `photo`/`art_style`, and `color_palette` values in every single prompt
string. That's not a stylistic preference, it's a structural consequence of the tool:
`generate_images_bulk` takes one `prompts` list and one shared set of non-prompt
parameters, with no per-prompt override for anything. If `style_description` isn't
locked identically across every prompt, there's no mechanism in the tool that keeps the
batch on-brief — each prompt renders independently, so a caption that drifts on style
between prompt 3 and prompt 4 produces two different looks in the same batch with
nothing tying them back together.

The only part of the caption that's supposed to change from prompt to prompt is
`compositional_deconstruction` — the `elements` list, an individual element's `desc`,
a swapped prop or pose, `bbox` placement, or framing/angle language in the caption's
prose. That's the entire variation surface. If a change doesn't live in
`compositional_deconstruction`, it shouldn't be in the diff between two prompts in the
same batch.

**Well-formed pair** (style locked, composition varied):

- Prompt A: `style_description` unchanged; `compositional_deconstruction.elements[0].desc`
  = "a skateboarder holding a skateboard under one arm"
- Prompt B: same `style_description`, same everything else; only
  `compositional_deconstruction.elements[0].desc` changes to "a skateboarder holding a
  coffee cup in one hand"

Only the element's `desc` moved. Every other field, including all of
`style_description`, is identical between the two prompts.

**Drifted pair** (the failure mode this rule prevents):

- Prompt A: `style_description.aesthetics` = "gritty documentary photojournalism,"
  `color_palette` = muted earth tones
- Prompt B: same composition swap as above, but `style_description.aesthetics` also
  changed to "clean studio product photography" and `color_palette` shifted to
  saturated primary colors

Prompt B isn't a variation of Prompt A anymore — it's a different brief that happens to
share a batch. The two images will look like they came from different shoots, and
because `generate_images_bulk` has no per-prompt style override, nothing in the call
itself would have caught this before submission. Catch it during drafting: any diff
touching a `style_description` field between two prompts in the same batch is a defect,
not a variation.

## One locked style per batch

If the user's actual ask spans two different looks — "half in warm golden-hour light,
half in cool blue studio light," or any other request that implies two different
`style_description` objects — that's two locked captions and two separate
`generate_images_bulk` calls, not one call with a `style_description` that drifts
partway through the `prompts` array.

The reason is the same architectural fact as above: `generate_images_bulk` has exactly
one style-equivalent influence per call — the shared non-prompt parameters plus
whatever's common across the prompt strings — because there is no per-prompt override.
A single call can represent exactly one locked style. Trying to make one call cover two
looks doesn't produce "half golden-hour, half studio" as a coherent split; it produces
an inconsistent batch where some images happen to land on one brief and some on the
other, with no field in the call or the response that says which prompts were "supposed"
to be which style. Splitting into two calls with two distinct locked
`style_description` objects is what actually delivers what the user asked for, and it's
also what makes the two halves reviewable and re-submittable independently if one half
misses the brief.

## Batch-size judgment

`generate_images_bulk`'s `prompts` parameter accepts 1–500 prompt strings — confirmed
directly from the tool's own description text ("Generate images in bulk from a list of
prompts (1-500)"), not a guideline invented for this skill. Treat 1–500 as the tool's
ceiling, not a target: there's no benefit to filling out a batch to some round number
if the variation axis doesn't actually support that many distinct, on-brief prompts.

Before committing to a full-size run, start smaller — roughly 8–20 prompts — to confirm
the chosen variation axis and the locked style are actually producing on-brief results.
This is the same ~20-prompt figure `SKILL.md`'s workflow step 4 uses as the threshold
for getting an explicit go-ahead before submitting; it isn't a second, unrelated number.
A batch small enough to validate cheaply and small enough to need confirmation before a
larger spend are the same size for the same reason — below it, a bad axis or a drifted
style costs a handful of wasted generations to discover; above it, the same mistake
costs a full-size batch's worth of credits before anyone looks at the results.

## What's confirmed vs. what to verify

**Confirmed (from direct inspection of the live connected MCP's tool schema and
description text):**

- The 1–500 prompt-count bound comes from the tool's own description text, quoted
  above. Note it's stated in prose, not enforced as a `minItems`/`maxItems` constraint
  on the `prompts` array in the tool's JSON schema itself — the schema only types
  `prompts` as an array of strings. Treat the 1–500 figure as the tool's documented
  contract, not something the schema will reject a bad call against before submission.
- The "shared parameters, no per-prompt override" architecture is confirmed by the
  schema shape itself: `prompts` is the only array-valued (per-item) field on the tool;
  every other parameter (`aspect_ratio`, `resolution`, `rendering_speed`, `style_type`,
  `negative_prompt`, `seed`, `magic_prompt_option`, `custom_model_uri`, `collection_id`,
  `private`) is a single scalar value applied to the whole call.
- The custom-model-only scoping is confirmed directly from the tool's description text:
  "`style_type`, `negative_prompt`, `magic_prompt_option`, `seed`: these apply ONLY to a
  custom-model (v3) generation and are ignored on the default v4 path. Omit them unless
  the user is using a custom model and asks."

**Unverified / unknown — don't assert this as fact:**

- The exact response shape for a multi-prompt `generate_images_bulk` submission (one
  job-level identifier covering the whole batch vs. one row per prompt) is not stated
  anywhere in the tool's description text. Don't assume either shape in advance. Read it
  from the actual response the first time `generate_images_bulk` is called in a
  session, and report what was actually observed rather than a plausible-sounding
  assumed value — the same discipline `collections-management/references/collection-patterns.md`
  applies to its own unconfirmed field shapes.
