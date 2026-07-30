# Anchorpoint Logo Upscale — Result

Worked example produced by actually running the `upscale-image-workflow` workflow
(resolve, confirm, run, save) end to end against the connected Ideogram MCP
(2026-07-30).

## Identifiers

- `image_upload_id`: `YYssktShTaCnV32I_wME3A` (returned by the sandbox `curl` upload of
  `anchorpoint-logo-1.png`; this is a fresh id obtained in this session, not reused from
  any prior session)
- `request_id` (from the `upscale_image` call itself): `zQWpcp7ETAmPzBg1vrQ7jw`
- `response_ids` (array, from the `upscale_image` call): `["7ozOJaKtV2GaNRpIlVN5Wg"]`
- `response_id` (singular, confirmed via `get_generation_status` once the job finished):
  `7ozOJaKtV2GaNRpIlVN5Wg` — matches `response_ids[0]` above
- `image_urls[0]`: `https://ideogram.ai/assets/image/balanced/response/7ozOJaKtV2GaNRpIlVN5Wg@2k`
- `thumbnail_urls[0]`: `https://ideogram.ai/assets/image/thumbnail/response/7ozOJaKtV2GaNRpIlVN5Wg@2k`
- `permalink_urls[0]`: `https://ideogram.ai/g/zQWpcp7ETAmPzBg1vrQ7jw/0`

## Steps run

1. **`upload_image(filename="anchorpoint-logo-1.png")`** — returned a one-time
   `upload_url` (`https://mcp.ideogram.ai/uploads/Gh8w3Q80sCNGvDY9PGP6yIAtmepAWBPAUBBWavfnVac`)
   and an `instructions` curl command.
2. **Ran the returned curl command in the sandbox** against the real PNG bytes at
   `skills/collections-management/examples/anchorpoint-logo-collection/anchorpoint-logo-1.png`.
   Response: HTTP 200,
   `{"success":true,"error_message":null,"id":"YYssktShTaCnV32I_wME3A","asset_id":null,"canvas_transaction_id":null,"file_name":"anchorpoint-logo-1","image_url":"https://ideogram.ai/api/images/ephemeral/YYssktShTaCnV32I_wME3A.png?exp=1785480685&sig=8b40cabcfa9651f667effee49694ee01b061a98184a51b7878cb952ad970d5c5"}`.
   The `id` field (`YYssktShTaCnV32I_wME3A`) is the `image_upload_id` used in step 4.
3. **Stated settings before calling** (Workflow step 2): no user-stated `upscale_factor`
   for this worked run, so the tool's `X2` default was used, stated plainly rather than
   blocked on a confirmation question. `upscale_details_weight`, `prompt`,
   `collection_id`, and `seed` were all omitted since none apply to this worked example.
   `private` was also omitted, deferring to the account's plan default per the tool's own
   documented behavior.
4. **`upscale_image(image_upload_id="YYssktShTaCnV32I_wME3A")`** — no other parameters
   set, exercising the `X2` default path. Immediate raw response:
   `{"status":"running","request_id":"zQWpcp7ETAmPzBg1vrQ7jw","num_images_requested":1,"operation_label":"Upscaled","prompt":"","poll_url":"https://mcp.ideogram.ai/widget/generations/zQWpcp7ETAmPzBg1vrQ7jw","poll_token":"OT8C3iwmBTaFjdEQW_balLq4Z2EJ7CKRpvxXEqEGkbY","notices":[],"response_ids":["7ozOJaKtV2GaNRpIlVN5Wg"],"image_urls":["https://ideogram.ai/assets/image/balanced/response/7ozOJaKtV2GaNRpIlVN5Wg@2k"],"thumbnail_urls":["https://ideogram.ai/assets/image/thumbnail/response/7ozOJaKtV2GaNRpIlVN5Wg@2k"],"permalink_urls":["https://ideogram.ai/g/zQWpcp7ETAmPzBg1vrQ7jw/0"]}`.
   Polled via `get_generation_status(request_id="zQWpcp7ETAmPzBg1vrQ7jw")` twice: first
   poll returned `status: "running"`; second poll returned `status: "done"`,
   `response_id: "7ozOJaKtV2GaNRpIlVN5Wg"` (matching `response_ids[0]` from the initial
   call), `permalink_url: "https://ideogram.ai/g/zQWpcp7ETAmPzBg1vrQ7jw/0"`.

## What was verified

- The initial `upscale_image` response is asynchronous: it returns `status: "running"`
  immediately, not a completed result — this differs from the "at minimum expect a
  `response_id`" phrasing in `SKILL.md`'s step 4 and `upscale-settings.md`'s
  confirmed-vs-unverified split. The field actually present on the immediate response is
  a plural `response_ids` array (containing one id here), not a singular `response_id`.
  A singular `response_id` field only appears once polled to completion via
  `get_generation_status`.
- Confirmed the immediate response does include `image_urls` and `thumbnail_urls` array
  fields (one entry each here) alongside `permalink_urls` — three separate URL-bearing
  fields, not one generic "download URL" field.
- Confirmed no `upscale_factor` echo field is present anywhere in the response (neither
  the immediate `upscale_image` response nor the `get_generation_status` row) — the
  resolved `X2` default is not echoed back by either tool, only asserted client-side per
  step 2 of the workflow.
- Confirmed no image dimension fields (width/height) are present in either response.
- Confirmed `get_generation_status` is required to observe completion (`status: "done"`)
  and the resulting singular `response_id`; the raw `upscale_image` call alone does not
  report completion status beyond `"running"`.

## Source image

Copied from
`skills/collections-management/examples/anchorpoint-logo-collection/anchorpoint-logo-1.png`
into this directory (`anchorpoint-logo-1.png`), so this example is self-contained, per
Task 3's Step 6.
