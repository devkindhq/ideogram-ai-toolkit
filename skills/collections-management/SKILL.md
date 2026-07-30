---
name: collections-management
description: Organizes Ideogram images — generated this session or previously uploaded — into named collections: create a collection, find-or-add to an existing one, browse what's in it, rename it, or remove images/delete it. Use whenever the user asks to "create a collection," "organize into a collection," "add to my collection," "what's in my [name] collection," or "rename/delete a collection." Only triggers on that kind of explicit request — it does not auto-run after `brand-identity-sheet`, `character-model-sheet`, `moodboard-generator`, or `custom-model-training` to proactively offer filing their output; those four skills are unmodified and unaware of this one.
---

# Collections Management

Ideogram lets you group images into named collections via `create_collection`,
`list_collections`, `add_images_to_collection`, `remove_images_from_collection`,
`rename_collection`, `delete_collection`, and `get_images_by_collection_id`. This skill
orchestrates those calls in sequence to keep a project's images organized instead of
scattered across loose generations that have to be re-found later by scrolling
`get_recent_generations`.

This is an orchestration workflow across sequential MCP calls, not a prompt-composition
skill — there's no `composition-spec-format.md`/`panel-anatomy.md`-style reference here.
Flat collections are the only supported structure in v1: `create_collection` and
`list_collections` both support nesting via `parent_collection_id`, but this skill never
sets or reads that field.

## Before you start

Read `references/collection-patterns.md` before creating or deleting anything. It covers
the find-or-create pattern (why you must check `list_collections` for a name match before
calling `create_collection`) and the destructive-confirmation rule (when `delete=True` and
`delete_assets=True` are safe to pass). Both apply before you touch step 1 or step 5 below.

## Workflow

### 1. Resolve the collection

Call `list_collections` (root-level only, since v1 is flat) and check for a name match
before creating anything new. Exactly one match → use that collection. Multiple ambiguous
matches (same name or close variants) → ask the user which one they mean, don't guess.
No match → call `create_collection(name)`. See `references/collection-patterns.md` for why
this check has to happen before create — `create_collection` doesn't enforce unique names,
so skipping this step produces duplicate collections across sessions.

### 2. Add images

Call `add_images_to_collection(collection_id, image_response_ids=[...])` for images
generated earlier this session (from `generate_image`'s `structured_content.response_ids`),
or `image_upload_ids=[...]` for previously uploaded images. Batch every image from the
current task into one call — don't loop one call per image. Read the response's
`added_count`, `failed_count`, and any per-asset failure reasons, and report real failures
instead of claiming the add fully succeeded.

### 3. Browse

Call `get_images_by_collection_id(collection_id)` when the user asks what's in a
collection, or to pull a `response_id`/`image_id` for a follow-up remix/edit/reframe/
upscale call. If the response includes a `next_cursor`, paginate by passing it back in as
`cursor` until there isn't one.

### 4. Rename

Call `rename_collection(collection_id, name)`. Straightforward passthrough, no special
handling needed.

### 5. Remove or delete

`remove_images_from_collection` unlinks specific images from a collection.
`delete_collection` removes the whole collection. Both default to preserving the
underlying image assets. Never pass `delete=True` (on remove) or `delete_assets=True` (on
delete) without an explicit, action-specific confirmation obtained in the current
exchange — a blanket "yes" given earlier in the conversation for something else does not
count as confirming this specific destructive action.

## Error handling

- Ambiguous name match during find-or-create (multiple collections with the same or
  similar name) → ask the user which one, don't guess.
- Partial `add_images_to_collection` failure → surface the real per-asset failure reasons
  from the response; don't report the add as fully successful.
- Destructive flags (`delete=True`, `delete_assets=True`) → never passed without an
  explicit, action-specific confirmation from the user in the current exchange.
- Collection not found by name when the user expected it to exist → say so and ask
  whether to create it, rather than silently creating a new one that might fork from what
  the user meant.

## Save what you made

After any create, add, rename, or delete, save the `collection_id`, collection name, and
any relevant `response_id`s to the project's existing output location — the same folder
`brand-identity-sheet`, `character-model-sheet`, or `moodboard-generator` already save to
for this project. Following the toolkit's "No Context Lost" habit: a `collection_id` that
only exists in the conversation is one the user has to rediscover by calling
`list_collections` later; save it now.

## Reference files

- `references/collection-patterns.md` — the find-or-create pattern and the destructive-
  confirmation rule, with the reasoning behind both. Read before running step 1 or step 5.
