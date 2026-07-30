# Anchorpoint Logo Collection — Result

Worked example produced by actually running the `collections-management` workflow
(steps 1–3, "resolve, add, browse," plus the rename step) end to end against the
connected Ideogram MCP (2026-07-30).

## Identifiers

- `collection_id`: `et9fwsyeQ-WmxiMypPAztg` (created as `anchorpoint-logo-collection`,
  later renamed to `anchorpoint-brand-logos`)
- Uploaded image identifiers (`image_upload_id`, one per source file):
  - `anchorpoint-logo-1.png` → `XYN-04ylRZy7l5b4WEtOHw`
  - `anchorpoint-logo-2.png` → `gq-Bs51SSuO0ffTDH5p-2g`
  - `anchorpoint-logo-3.png` → `W0tqzJTCRySU1RiGwBTq7Q`
  - `anchorpoint-logo-4.png` → `kD7nkbcuSIyiZIg4WMnESg`
  - `anchorpoint-logo-5.png` → `ts-KkzvfQ5WAJBD-plqdZw`
- `add_images_to_collection` result: `added_count: 5`, `failed_count: 0`,
  `failed_assets: null` — no partial failures to report.
- Final `asset_count` on the collection (from the `rename_collection` response):
  `5`.

## Steps run

1. **`list_collections`** (no args) — returned `{"collections": [], "next_cursor": null}`.
   No pre-existing collection with any name, confirming a clean create for
   `anchorpoint-logo-collection` was safe.
2. **`create_collection(name="anchorpoint-logo-collection")`** — returned
   `collection_id: et9fwsyeQ-WmxiMypPAztg`, `parent_collection_id: null` (flat, v1-only
   as required), `metrics.asset_count: 0`.
3. **`upload_image`** — called once per file (the tool only accepts a single
   `filename`, unlike the dataset-training tool's multi-file `upload_dataset_assets`).
   Each call returned a one-time `upload_url`; the returned `curl` command was run in
   the sandbox for each of the 5 files and returned HTTP 200 with a JSON body
   containing the real `id` used above. All 5 uploads succeeded (`"success": true`).
4. **`add_images_to_collection(collection_id="et9fwsyeQ-WmxiMypPAztg",
   image_upload_ids=[<all 5 ids above>])`** — one batched call, not looped. Response:
   `{"added_count": 5, "failed_count": 0, "failed_assets": null}`.
5. **`get_images_by_collection_id(collection_id="et9fwsyeQ-WmxiMypPAztg")`** — returned
   all 5 assets (`asset_type: UPLOAD`, `image_id` matching the upload ids above,
   `filename` matching the 5 source filenames), and `next_cursor: null` — no
   pagination needed since all 5 items fit in a single page.
6. **`rename_collection(collection_id="et9fwsyeQ-WmxiMypPAztg",
   name="anchorpoint-brand-logos")`** — response confirmed `name:
   "anchorpoint-brand-logos"` and `metrics.asset_count: 5`.

## What was verified

- The find-or-create check (`list_collections` before `create_collection`) actually
  prevents duplicate names: this run confirmed zero existing collections before
  creating, per `references/collection-patterns.md`'s rule.
- `add_images_to_collection` does report `added_count`/`failed_count`/`failed_assets`
  as documented — this run had no failures, so the per-asset failure-reason shape
  documented as "unverified" in `collection-patterns.md` is still unconfirmed; only the
  all-success path was exercised here.
- `get_images_by_collection_id` returned all 5 uploaded images with `next_cursor: null`
  in a single page — pagination behavior beyond 5 items was not exercised.
- `rename_collection` is a straightforward passthrough as documented: the returned
  object is the full updated collection record, not just a name field.

## Source images

The 5 source files were copied from
`skills/custom-model-training/examples/anchorpoint-logos/` into this directory
(`anchorpoint-logo-1.png` through `anchorpoint-logo-5.png`) so this example is
self-contained, per Task 3's Step 7.
