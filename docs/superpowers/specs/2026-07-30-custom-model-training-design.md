# Custom Model Training Skill — Design

## Why

Gap-research (see `04-projects/devkind/ideogram-ai-toolkit.md` in the Devkind vault, dated 2026-07-30) found zero prior art anywhere in the Claude Code skill ecosystem for an Ideogram custom-model-training workflow. It's also the highest-value gap identified: it chains directly onto the existing `brand-identity-sheet` skill — a locked identity sheet becomes training data for a custom model that then generates on-brand assets indefinitely. No competitor has built this pipeline.

The connected Ideogram MCP already exposes the full tool surface needed: `create_dataset`, `upload_dataset_assets`, `list_datasets`, `train_model`, `get_model`, `list_models` (confirmed live via direct tool-schema inspection, not assumed from docs).

## Scope

One new skill: `skills/custom-model-training/`. Full pipeline, not a planning-only guide — mirrors the "actually calls the tools" discipline the other 5 skills already follow (see `brand-identity-sheet/SKILL.md`'s framing: "actually renders it via the ideogram MCP tools, not just drafts a prompt").

Deliberately excludes collections management — a separate, independent gap with its own tool surface (`create_collection`, `list_collections`, `add/remove_images_from_collection`, `rename_collection`, `delete_collection`, `get_images_by_collection_id`). That gets its own spec/plan/implementation cycle after this one ships.

## Architecture

Structural pattern matches the existing 5 skills: `SKILL.md` + `references/` + `examples/` + `evals/`. Key difference: this skill is an **orchestration workflow** across sequential MCP calls, not a prompt-composition skill — so it does not need the `composition-spec-format.md` / `panel-anatomy.md` reference pattern the visual-generation skills use.

## Components

- **`SKILL.md`** — frontmatter description written to trigger pushily on: "custom model," "fine-tune," "train a model on these images," "on-brand model," "consistent character/product across generations." Body documents the 6-step workflow (below).
- **`references/dataset-requirements.md`** — honest split of confirmed vs. unverified facts:
  - Confirmed (from direct tool-schema inspection): `create_dataset` takes only `name`; `upload_dataset_assets` returns one-time, short-lived, single-use upload URLs that MUST be run sequentially, not concurrently; `train_model` takes `dataset_id` + `model_name` and runs asynchronously; `get_model` takes `model_id`.
  - Unverified / explicitly flagged as unknown: minimum/recommended image count, supported image formats, expected training duration, the exact status field/enum `get_model` returns when a model is ready. The skill tells the user to expect some trial-and-error rather than asserting invented numbers — consistent with the global rule against stating third-party API behavior as fact without a verified source.
- **`examples/`** — one worked example, produced by actually running the pipeline once (see Testing below). Given training is async and may be slow, the example may need to document the polling loop rather than only the final result.
- **`evals/evals.json`** — 2-3 eval prompts in the existing schema (matching `brand-identity-sheet/evals/evals.json`'s format: `id`, `eval_name`, `prompt`, `expected_output`, `files`, `expectations`).

## Data flow / workflow

1. **Resolve input** — user points to a local folder of reference images (this version supports local-folder input only; in-session-generation-as-source was considered and deferred, not built).
2. `create_dataset(name)` → `dataset_id`.
3. `upload_dataset_assets(dataset_id, filenames)` → array of one-time `curl_command`s; run each **sequentially** via Bash (the tool's own description warns against concurrent use — URLs are single-use and short-lived).
4. `train_model(dataset_id, model_name)` → async job starts, returns some job/model identifier.
5. Poll `get_model(model_id)` until the response indicates the model is ready. Since the exact status field isn't documented ahead of time, the skill reads whatever fields the response actually contains rather than assuming a specific enum value up front, and treats an unrecognized/stuck state as a reportable condition rather than an error to paper over.
6. On ready, extract `custom_model_uri` from the response and call `generate_image(prompt, custom_model_uri=...)` to close the loop and prove the trained model actually works — completing the "locked sheet → training data → on-brand generation" pipeline described in the vault strategy note.

## Error handling

- Empty or missing input folder → ask the user for a valid path; never fabricate a dataset from nothing.
- An upload `curl_command` failing → surface the real error to the user; do not blindly retry, since a stale single-use URL needs to be re-requested via `upload_dataset_assets`, not resubmitted.
- Training failure, or `get_model` staying in an unrecognized/non-ready state past a reasonable number of polls → report status honestly to the user rather than claiming success or silently giving up.

## Testing

Standard `evals/evals.json` pattern used by the other skills, 2-3 realistic prompts (e.g. "train a custom model on these 5 logo renders, then generate a poster using it"). Assertions to be drafted once the skill exists, following the skill-creator eval workflow.

## Out of scope (this spec)

- Collections management (separate future skill).
- In-session-generation-as-training-source (folder-based input only, for now).
- Any UI/dashboard for browsing trained models (`list_models` is used only as a lookup step, not a feature to build around).
