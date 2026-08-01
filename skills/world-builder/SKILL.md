---
name: world-builder
description: Builds out a fictional "world" around an already-trained custom character model — a staged, multi-batch pipeline (no-mascot moodboard, paired with/without-model scene tests, society & culture batch, family/village/society deep pass, landmarks/maps/art pass, optional contamination-check redo) that goes from "I have a trained mascot" to "I have a whole visual world this character lives in," using `generate_image`, `generate_images_bulk`, `remix_image`, and `describe_image`. Use whenever the user wants to expand a locked mascot or character (already produced by `character-model-sheet` and trained by `custom-model-training`) into a broader brand world — community scenes, rituals, hierarchy, family/village life, landmarks, maps, art objects — or asks for "world-building," "brand world," "the world Bell-Boy lives in," "expand the universe around this character," or "society/culture pass." Distinct from `character-model-sheet` (locks the one character's own design) and `custom-model-training` (turns that locked design into a `custom_model_uri`) — this skill is the next step after both exist, and refuses to start without a trained model in hand. Also distinct from `moodboard-generator`'s generic pre-brand exploration board — this skill's moodboard step is one stage in a longer character-anchored pipeline, not a standalone deliverable.
---

# World Builder

A trained custom model locks one character's likeness. It does not, by itself, tell you
what that character's *world* looks like — the palette its packaging uses, whether it
has a village, what its landmarks are, whether "family" scenes read as a community or as
a crowd of clones. This skill is the staged pipeline for answering that, developed over
several real sessions building out the world around StoreAlert's mascot ("Bell-Boy") and
generalized here so it applies to any already-trained character.

The whole pipeline is sequenced around one constraint: **every image that uses the
custom model spends training-specific budget and risks compounding whatever quirks the
model has (see `references/character-batch-discipline.md`), so cheaper no-model passes
come first, cheaper batches come before expensive ones, and nothing gets called "done"
without a human visually reviewing it.**

## Prerequisite — do not start without this

A custom model must already exist for the character, trained via the
`custom-model-training` skill (dataset → `train_model` → `get_model` polling →
`custom_model_uri`). If the user hasn't mentioned a `custom_model_uri`, ask for it or
run `mcp__ideogram__list_models` to find it before doing anything else. Don't start
generating "world" images against a character description alone — the entire point of
this pipeline is testing how a *trained* model behaves across many contexts, not
describing the character freshly in each prompt (which `character-model-sheet` already
does, and which drifts exactly the way `custom-model-training`'s existence is meant to
prevent).

## Cross-cutting discipline — applies to every step below, no exceptions

Read these two reference files once, before step 1, and re-apply them on every single
prompt for the rest of the pipeline:

- **`references/palette-lock.md`** — the locked palette (paper/dominant, primary,
  secondary, ink/trim, and at most one reserved accent used in exactly one place) must
  be defined and quoted, verbatim, in every prompt from step 1 onward. A world built on
  a drifting palette isn't a world, it's six unrelated images.
- **`references/anti-slop-discipline.md`** — the reusable, brand-agnostic ban list
  (glowing orbs, neural-network nodes, circuit-board textures, gradient washes,
  stock-photo people, glossy mirror-shine, plastic-toy uncanny valley, and the rest).
  Run the pre-generation gate in this file before every `generate_image` /
  `generate_images_bulk` call, the same discipline `character-model-sheet` and
  `brand-identity-sheet` already apply to their own single-image gates, scaled up to a
  multi-batch pipeline.

And read `references/character-batch-discipline.md` before step 2 (the first step that
puts the custom model in front of more than one character) — it covers the
character-count clause, the no-characters clause, and the known family-resemblance
limitation that governs every batch from here on.

## Workflow

### 1. Moodboard pass — no mascot, establish the visual language

Before spending any custom-model budget, generate a pure brand-world moodboard with the
character **absent**. Do not pass `custom_model_uri` on this call. Prompt for palette,
material, texture, and mood only — no characters, no mascots, no people. This is the
step that locks the palette (per `references/palette-lock.md`) and the material/texture
language everything downstream has to agree with, before the character is anywhere in
frame. Use `mcp__ideogram__generate_image`, `style_type: "DESIGN"`, no custom model.

If the project already has a `moodboard-generator` board, this step can extend that
board's palette rather than re-deriving it from scratch — but the character-absence rule
still applies even when reusing an existing palette.

### 2. Paired with/without family — does the mascot integrate into real usage?

For a set of product/UI-style scenes (packaging shot, app screen, storefront signage,
whatever the brand's real usage contexts are), generate each scene **twice**, same
composition/palette/lighting held identical between the pair:

- **With family**: `custom_model_uri` set, mascot present, exact character count stated
  (see `references/character-batch-discipline.md`).
- **Without family**: no `custom_model_uri`, mascot and all characters absent, explicit
  "no characters, mascots, or people anywhere in frame" clause.

The point of the pair is diagnostic, not decorative: it tests whether the mascot reads
as belonging in the scene or as pasted on top of it. Review both halves of each pair
side by side, not independently — a mascot that looks fine alone but breaks the
composition once the "without" half is generated for comparison is a real finding, not
a false alarm.

### 3. Society & culture batch — paired with no-model "world artifacts"

Two batches, run as a pair the same way step 2 was paired, but now testing
civic/cultural depth instead of product usage:

- **Character-driven batch** (`custom_model_uri` set): community roles, rituals
  (graduation, festival), hierarchy/rank lineups. State the exact character count in
  every prompt — this is the step where crowd-runaway risk is highest, since "hierarchy
  lineup" and "festival" both invite the model to keep adding figures unless capped
  explicitly.
- **World-artifacts batch** (no `custom_model_uri`, no characters): maps, a landmark,
  heraldry/crest, currency, textile pattern, a public space. Explicit
  "no characters or mascots or people anywhere in frame" clause on every prompt in this
  batch.

Submit each batch via `mcp__ideogram__generate_images_bulk` if it's more than a couple
of prompts (see `bulk-image-generation-workflow` for the batch-submission pattern this
reuses) — but keep the character-driven and world-artifacts batches as two separate
`generate_images_bulk` calls, never mixed into one, since they use different
`custom_model_uri` settings and that parameter is shared across an entire batch (there
is no per-prompt override).

### 4. Family/village/society deep pass — the larger character-driven batch

A larger batch (roughly a dozen images, adjust to the world's actual scope) using the
custom model throughout, covering: family units, village life, town hall, classroom,
elder council, processions, home life, friendships, harbor, park, celebrations. Every
prompt states its exact character count and quotes the locked palette. This is the
deepest character-driven pass in the pipeline — expect (and explicitly review for, not
silently accept) the family-resemblance limitation described in
`references/character-batch-discipline.md`, since this batch has the highest character
density of any step.

Submit via `generate_images_bulk` given the batch size; track every `request_id` per
`references/batch-tracking.md`.

### 5. Landmarks, maps & art pass — pure world-building, no characters at all

No `custom_model_uri`, no characters anywhere. Landmarks (halls, towers, plazas,
libraries, galleries, amphitheaters), a world map, art objects (sculptures, murals,
ceramics, stained glass, monuments). Same locked palette, same anti-slop gate, explicit
"no characters or mascots or people anywhere in frame" clause on every prompt. This pass
exists independently of whether step 3's world-artifacts batch already touched some of
the same territory (a map, a landmark) — step 5 goes deeper and wider, treating those as
a first pass rather than the final one.

### 6. Contamination-check redo — optional, flag the risk explicitly

Optional step, and a real risk, not a formality. If the team wants to know whether it's
viable to use the trained custom model *everywhere* (including no-character world-only
prompts), rerun a sample of step 1's or step 5's no-model prompts through the custom
model and compare.

Read `references/contamination-check.md` before doing this. The short version: a model
trained on one character's likeness carries a real risk of that character's silhouette
bleeding into architecture, object, or map prompts that never asked for a character at
all — a tower that's subtly bell-shaped, a crest that echoes the mascot's face, a map
border pattern that repeats the character's silhouette. This has to be **visually
reviewed image by image**, not assumed safe because the prompt said "no characters."
Never report this step as "passed" without someone actually looking at the images
side-by-side against the clean no-model versions from steps 1 and 5.

## Tracking discipline — every batch, every step

Every batch in every step above gets tracked per `references/batch-tracking.md`:
`request_id`/`job_id` for traceability, whether `custom_model_uri` was used (and which
one), and an explicit `"visual_review_status": "pending"` until a human actually looks
at the results and confirms them. An API call returning 200 is not the same thing as the
batch being done — don't mark anything "done" on API success alone.

## Save what you made

Per the toolkit's "No Context Lost" habit, write every step's prompts, batch/request
IDs, `custom_model_uri` usage, and review status to the project's world-building folder
(check for an existing `logo-explorations/`, `branding/`, or `world/` folder first and
match it; create `world-building/` under the project folder if none exists). Save
incrementally as each step completes — this is a long, multi-session pipeline by design,
and nothing generated in any step should live only in the conversation if the session
ends mid-pipeline.

## Reference files

- `references/palette-lock.md` — how to define and lock the paper/primary/secondary/
  ink-trim/accent palette before step 1, and the discipline for quoting it verbatim in
  every subsequent prompt.
- `references/anti-slop-discipline.md` — the reusable, brand-agnostic ban list (not
  brand-specific) and the pre-generation gate to run before every call in every step.
- `references/character-batch-discipline.md` — the exact-character-count clause, the
  no-characters clause, and the known family-resemblance/near-clone limitation of
  single-character-trained models — what it is, why it's expected rather than a bug, and
  how to review for it ("does this read as a species/community, or does it read as
  broken").
- `references/batch-tracking.md` — the per-batch tracking record (request/job IDs,
  `custom_model_uri` used or not, visual-review status) to keep on every batch across
  every step.
- `references/contamination-check.md` — the optional step-6 risk check: what
  "contamination" looks like when a single-character model is run against no-character
  world prompts, and how to review for it honestly instead of assuming safety.
