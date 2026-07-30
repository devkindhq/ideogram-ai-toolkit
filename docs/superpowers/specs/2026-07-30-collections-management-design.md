# Collections Management Skill — Design

## Why

Gap-research (see `[redacted-internal-path]` in the internal notes, dated 2026-07-30) identified collections management as a Tier 1 gap: zero prior art anywhere in the Claude Code skill ecosystem for an Ideogram collections workflow, alongside custom model training. Custom model training shipped first (`skills/custom-model-training/`); this is the second of the two Tier 1 skills.

The connected Ideogram MCP already exposes the full tool surface needed: `create_collection`, `list_collections`, `add_images_to_collection`, `remove_images_from_collection`, `rename_collection`, `delete_collection`, `get_images_by_collection_id` (confirmed live via direct tool-schema inspection, not assumed from docs).

The core use case: organizing a project's generated assets as they're made, so a brand or character's images don't stay scattered across loose generations that have to be re-found later by scrolling `get_recent_generations`. This is the same problem the Fizzwright/Kip custom-model-training example solved manually (assembling 16 loose generations into a local folder) — collections give Ideogram-side structure for that instead.

## Scope

One new skill: `skills/collections-management/`. Triggers only on explicit user request ("organize these into a collection," "create a collection for X," "what's in my Y collection," "delete my old Z collection") — it does not auto-wire into `brand-identity-sheet`, `character-model-sheet`, `moodboard-generator`, or `custom-model-training` to proactively offer filing their output. Those five skills are unmodified by this spec.

Flat collections only. Ideogram's `create_collection`/`list_collections` support nesting via `parent_collection_id`, but v1 doesn't use it — one collection per project/brand mirrors how the other skills already organize local output (one folder per project), and nesting can be added later if a real need shows up.

## Architecture

Structural pattern matches `custom-model-training`: `SKILL.md` + `references/` + `examples/` + `evals/`. This is an **orchestration workflow** across sequential MCP calls, not a prompt-composition skill — it does not need the `composition-spec-format.md` / `panel-anatomy.md` reference pattern the visual-generation skills use.

One unified skill, not split into separate "organizer" and "curator" skills — matches how every other skill in this repo is one coherent workflow, and avoids adding a skill-discovery decision ("which one triggers here?") the repo doesn't otherwise need. Destructive operations (delete/remove-with-delete) are guarded by confirmation logic inside the single skill rather than by structural separation.

## Components

- **`SKILL.md`** — frontmatter written to trigger on: "create a collection," "organize into a collection," "add to my collection," "what's in my [name] collection," "rename/delete a collection." Body documents the 5-step workflow (below).
- **`references/collection-patterns.md`** — the find-or-create pattern (why: `create_collection` doesn't enforce unique names, so naive create-always usage produces duplicate "Fizzwright" collections over multiple sessions) and the destructive-flag confirmation rule (`delete=True` on `remove_images_from_collection`, `delete_assets=True` on `delete_collection` — both are irreversible per the tool's own description).
- **`examples/`** — one worked example, produced by actually running the pipeline once (create a collection, file real generated images into it, confirm contents via `get_images_by_collection_id`).
- **`evals/evals.json`** — 3 eval prompts in the existing schema (matching `custom-model-training/evals/evals.json`'s format: `id`, `eval_name`, `prompt`, `expected_output`, `files`).

## Data flow / workflow

1. **Resolve the collection** — call `list_collections` (root-level, since v1 is flat) and check for a name match before creating. If one match, use it. If multiple ambiguous matches, ask the user which one. If none, `create_collection(name)`.
2. **Add images** — `add_images_to_collection(collection_id, image_response_ids=[...])` for images generated this session (from `generate_image`'s `structured_content.response_ids`) or `image_upload_ids=[...]` for uploaded images. Batch all images from the current task into one call rather than looping one-at-a-time. Read the response's `added_count`/`failed_count`/per-asset failure reasons and report any real failures — don't claim success if some assets failed.
3. **Browse** — `get_images_by_collection_id(collection_id)` when the user asks what's in a collection, or to retrieve a `response_id`/`image_id` to feed into a follow-up remix/edit/reframe/upscale call. Paginate via `cursor` if `next_cursor` is present.
4. **Rename** — `rename_collection(collection_id, name)`, straightforward passthrough.
5. **Remove or delete** — `remove_images_from_collection` (unlink specific images) or `delete_collection` (remove the whole collection). Both default to preserving the underlying assets. Only pass `delete=True` or `delete_assets=True` after the user has explicitly confirmed they want the actual images permanently destroyed, not just unlinked from the collection — a prior blanket "yes" earlier in the conversation does not count as confirming this specific destructive action.

## Error handling

- Ambiguous name match during find-or-create (multiple collections with the same or similar name) → ask the user which one, don't guess.
- `add_images_to_collection` partial failure → surface the real per-asset failure reasons from the response; don't report the add as fully successful.
- Destructive flags (`delete=True`, `delete_assets=True`) → never passed without an explicit, action-specific confirmation from the user in the current exchange.
- Collection not found by name and user expected it to exist → say so and ask whether to create it, rather than silently creating a new one that might fork from what the user meant.

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts:
1. "File these Anchorpoint logo renders into a collection" — create+add on a fresh (no prior matching) collection.
2. "Add this new render to my existing Fizzwright collection" — find-existing+add, verifying no duplicate collection gets created.
3. "Delete my old moodboard-drafts collection, I don't need the images either" — destructive path, verifying the skill treats "don't need the images either" as the specific confirmation needed for `delete_assets=True` rather than skipping the check.

## Out of scope (this spec)

- Nested sub-collections (`parent_collection_id`) — flat only for v1.
- Proactive/automatic filing from the other 5 generation skills — explicit-request trigger only.
- Any collection browsing UI/dashboard — `get_images_by_collection_id` is used as a lookup step, not a feature to build around.
