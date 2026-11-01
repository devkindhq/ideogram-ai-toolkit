# Anchorpoint Logo 9:16 Reframe — Result

Worked example produced by actually running the `reframe-image-workflow` workflow
(steps 1–4: resolve source, confirm ratio, run reframe, save output) end to end
against the connected Ideogram MCP (2026-07-30).

## Identifiers

- `image_upload_id` (from `upload_image`): `Uf0WwogoSa-MrkRwwpHBgg`
- `aspect_ratio` passed to `reframe_image`: `9:16`
- `seed`: not passed (not requested)
- `private`: not passed (not requested)
- `reframe_image` `request_id`: `78Wm5sAKQR2G1wZuAIUE2Q`
- Output `response_id` (from `get_generation_status` once the job reached
  `status: "done"`): `q92kNPL4VNO8-IMTunX5uQ`
- Output image URL: `https://ideogram.ai/assets/image/balanced/response/q92kNPL4VNO8-IMTunX5uQ@2k`
- Permalink: `https://ideogram.ai/g/78Wm5sAKQR2G1wZuAIUE2Q/0`

## Steps run

1. **Resolve the source (local file)** — the source image was
   `skills/collections-management/examples/anchorpoint-logo-collection/anchorpoint-logo-1.webp`,
   an existing file on disk, not a same-session generation or a collection item. Per
   `references/source-and-ratio-resolution.md`'s "Local file on disk" path, called
   `upload_image(filename=".../anchorpoint-logo-1.webp")`. It returned a one-time
   `upload_url` and instructions with a `curl` command. That `curl` command was run
   immediately in the sandbox and returned HTTP 200 with `"success": true` and
   `id: "Uf0WwogoSa-MrkRwwpHBgg"` — the real `image_upload_id` used below. No retry
   was needed; the first upload URL did not expire.
2. **Confirm the target aspect ratio (named format)** — the request was for
   "Instagram Story," a named format, not an already-explicit ratio. Per the mapping
   table in `references/source-and-ratio-resolution.md`, "Instagram Story / Reel" maps
   to `9:16`. This exercised the named-format-to-ratio mapping path rather than an
   already-explicit ratio.
3. **Run `reframe_image`** — called with `image_upload_id: "Uf0WwogoSa-MrkRwwpHBgg"` and
   `aspect_ratio: "9:16"`, with neither `seed` nor `private` passed (neither was
   requested). The call returned `status: "running"`, `request_id:
   "78Wm5sAKQR2G1wZuAIUE2Q"`, and a provisional `response_ids: ["q92kNPL4VNO8-IMTunX5uQ"]`.
   Polled with `get_generation_status(request_id="78Wm5sAKQR2G1wZuAIUE2Q")`, which
   returned `status: "done"` with the same `response_id: "q92kNPL4VNO8-IMTunX5uQ"` and
   `permalink_url: "https://ideogram.ai/g/78Wm5sAKQR2G1wZuAIUE2Q/0"`.
4. **Save the output** — downloaded the output image from
   `https://ideogram.ai/assets/image/balanced/response/q92kNPL4VNO8-IMTunX5uQ@2k` via
   `curl`, which returned HTTP 200 and a 56,386-byte file. Despite the URL's `.png`-style
   path, the downloaded bytes are WebP (`RIFF ... Web/P image, VP8 encoding, 768x1376`,
   confirmed via `file`), consistent with the requested `9:16` ratio (768:1376 reduces to
   ~0.558, matching 9:16 ≈ 0.5625). Saved as `anchorpoint-logo-1-9x16.webp` in this
   directory, alongside a copy of the source (`anchorpoint-logo-1-source.webp`) so the
   example is self-contained, per `collections-management`'s worked-example pattern.

## What was verified

- The local-file source-resolution path (`upload_image` → run `curl` immediately →
  `id` becomes `image_upload_id`) works end to end; the first upload URL did not expire,
  so the "stale URL, call `upload_image` again" fallback was not exercised in this run.
- The named-format-to-ratio mapping ("Instagram Story" → `9:16`) from
  `references/source-and-ratio-resolution.md`'s table was applied and accepted by
  `reframe_image` without error.
- `reframe_image` returned `status: "running"` first (asynchronous), not an immediate
  final result; `get_generation_status` was needed to confirm `status: "done"` and
  retrieve the final `response_id`. This is new information not previously documented in
  `SKILL.md` or `references/source-and-ratio-resolution.md`, both of which describe the
  call as if it returns a result directly.
- The actual output file format returned by the image URL is WebP, not PNG, despite the
  file extension implied by the URL path — worth noting for any future skill work that
  assumes PNG output.
- `seed` and `private` were both omitted successfully with no error, confirming they are
  genuinely optional as documented.

## Source image

The source file used was
`skills/collections-management/examples/anchorpoint-logo-collection/anchorpoint-logo-1.webp`
(reused as-is per Task 3's brief), copied into this directory as
`anchorpoint-logo-1-source.webp` so this example is self-contained.
