# Courier Fox Sticker Pack — Result

Worked example produced by actually running the `bulk-image-generation-workflow`
skill's full 6-step workflow end to end against the connected Ideogram MCP
(2026-07-30). Batch size was 10 prompts, under the ~20-prompt confirmation
threshold documented in `SKILL.md`'s workflow step 4, so this run proceeded
without a separate confirmation gate.

## Identifiers

- `job_id`: `XR3lAioqQSiHoVUSoSgyaQ` (`job_size: 10`)
- Submission params: `aspect_ratio: "1x1"`, no other non-default parameters.
- Per-prompt `request_id` → `response_id` (in the order submitted; see
  `PROMPTS.md` for the matching prompt text):

| # | Pose/prop desc | `request_id` | `response_id` |
|---|---|---|---|
| 1 | running forward holding parcel | `iiozZ_s9SGm7ab2YGSd7_A` | `e9Se_BwAXciyPuaMLYnnlQ` |
| 2 | sitting, sipping coffee | `f9WYFAN_SouEcq6EhMwU2g` | `YuDq_SIrVXKNpEk6PDjq8A` |
| 3 | standing, waving | `4bJcCswNRTWBiPnX8waedQ` | `ghy9IHzaWmG0voDkoq7wuw` |
| 4 | riding bicycle, basket of packages | `ohX-jAoQTOOvwjBb8Td7Dw` | `GiqRn3p9U6aJgeOlQCBqSg` |
| 5 | holding clipboard, checking off delivery | `2iuNJt6hQdi7uPPCsRdmkw` | `C6oefO7JVqK5WTFXWIrfJg` |
| 6 | carrying stack of boxes | `VAasPsfRQz2ziuca3QBZIQ` | `p_IwIkwtXFSwMTnGPjLC-A` |
| 7 | riding delivery scooter | `UjddAMZOTtOE0xNMyM_wmQ` | `kiYxeCGXXBmPMaPEEZ4y6g` |
| 8 | thumbs up, package under other arm | `T33siPVYSq2qVOtARZSz2w` | `mkM_GtahXiqOIEAaPxTcoA` |
| 9 | knocking on door, holding package | `eehiFz-DRmuWK8XGs-IVRQ` | `N6hj7TEZWMuanCanEh6Vhg` |
| 10 | jumping mid-air, clutching parcel | `Aj7NsmcOQyifvVHMv-PQoQ` | `Ywt1CBdhX9W9fFdDeJxMEQ` |

## Steps run

1. **`generate_images_bulk`** — called once with the 10-item `prompts` array
   from `PROMPTS.md` and `aspect_ratio: "1x1"`. Response returned immediately
   (`"status": "running"`, `"async": true`, `"batch": true`) with the
   `job_id`, all 10 `request_ids` (in submission order), all 10
   `response_ids`, plus parallel `image_urls`, `thumbnail_urls`, and
   `permalink_urls` arrays — all entries `"status":"running"` at submission,
   confirming the tool is fire-and-forget/async rather than blocking until
   render completion.
2. **`get_generation_status`** — polled 3 times total, passing a single
   `request_id` from the batch each time (`iiozZ_s9SGm7ab2YGSd7_A`):
   - Poll 1: all 10 rows `"status": "running"`.
   - Poll 2: 1 of 10 rows `"status": "done"` (`UjddAMZOTtOE0xNMyM_wmQ`, the
     scooter pose), the other 9 still `"running"`.
   - Poll 3: all 10 rows `"status": "done"`.
   Each poll's response also returned status rows for unrelated older
   generations from the session's history (e.g. an in-progress Bell-Boy
   app-icon job on poll 3) — passing any one `request_id` returns the whole
   session's recent job table, not just the batch it belongs to.
3. **`get_recent_generations(n=10, filter_mode="GENERATIONS")`** — called as
   the fallback lookup per `review-culling-guide.md` to get directly
   downloadable image URLs. Each asset's `thumbnail_url` followed the pattern
   `https://ideogram.ai/assets/image/thumbnail/response/<response_id>@2k` — a
   real, `curl`-able WebP file, unlike `permalink_url`
   (`https://ideogram.ai/g/<request_id>/0`), which is a client-rendered SPA
   page with no image URL embedded in its static HTML. All 10 WebP thumbnails
   were downloaded and converted to PNG (via `sips`) for visual review.
   The `prompt` field on each returned asset was NOT the literal JSON string
   submitted — Ideogram's backend expands the submitted
   `style_description`/`compositional_deconstruction` JSON into a richer
   internal JSON, synthesizing an added `high_level_description` and
   splitting the single submitted `obj` element into several finer-grained
   elements (e.g. separate elements for the fox's body, cap, bag, and held
   prop). The "JSON caption bypasses magic prompt" claim in
   `skills/ideogram-prompt/references/json-caption-schema.md` is therefore
   only partially confirmed: some server-side reinterpretation/expansion
   still happens even when structured JSON is submitted directly.

## Review

Each image scored against the locked `style_description` (flat vector, bold
clean outlines, the 4-color palette `#FF6B35`/`#FFFFFF`/`#2B2D42`/`#F7C59F`,
solid white background) and its own per-prompt pose/prop intent:

1. **Running, holding parcel** (`e9Se_BwAXciyPuaMLYnnlQ`) — **KEEP.** Pose
   matches the prompt (mid-stride run, parcel under one arm), bold clean
   outlines, correct palette, solid white background. Minor deviation: a
   faint drop shadow under the feet, which the locked `lighting` field says
   to avoid — not severe enough to reject.
2. **Sitting, sipping coffee** (`YuDq_SIrVXKNpEk6PDjq8A`) — **KEEP.** Pose
   matches exactly (fox seated, coffee cup in hand, steam lines), correct
   palette, bold clean outlines, solid white background.
3. **Standing, waving** (`ghy9IHzaWmG0voDkoq7wuw`) — **KEEP.** Pose matches
   (fox waving with a satchel bag), correct palette, bold clean outlines,
   solid white background.
4. **Riding bicycle, basket of packages** (`GiqRn3p9U6aJgeOlQCBqSg`) —
   **REJECT.** Pose matches (fox on a bicycle with a loaded basket), but the
   rendering style drifted from the locked `art_style`: softer gradient
   shading and thinner, less-bold outlines than the other 9 images, and a
   visibly muted palette rather than the locked hex values.
5. **Clipboard, checking off delivery** (`C6oefO7JVqK5WTFXWIrfJg`) — **KEEP.**
   Pose matches (fox holding clipboard and pencil), correct palette, bold
   clean outlines, solid white background.
6. **Carrying stack of boxes** (`p_IwIkwtXFSwMTnGPjLC-A`) — **REJECT.** Pose
   matches (fox carrying a tall stack of boxes), palette and outline style
   are on-brief, but the model hallucinated unrequested label text on the
   boxes ("EXPRESS", "FRAGILE", "FOX RUN") — same failure mode as image 7,
   not something the prompt asked for.
7. **Riding scooter** (`kiYxeCGXXBmPMaPEEZ4y6g`) — **REJECT.** Pose matches
   (fox riding a scooter with a package on back), style and palette are
   close, but the model hallucinated unrequested brand text ("SWIFT
   EXPRESS") on the scooter body — text was never in the prompt or the
   locked caption's elements list.
8. **Thumbs up, package under arm** (`mkM_GtahXiqOIEAaPxTcoA`) — **KEEP.**
   Pose matches exactly, correct palette, bold outlines, solid white
   background. One of the cleanest results in the batch.
9. **Knocking on door** (`N6hj7TEZWMuanCanEh6Vhg`) — **REJECT.** Pose matches
   (fox mid-knock, holding a box), but the model rendered a full orange door
   filling most of the frame instead of the locked
   `"A solid #FFFFFF background with no other elements."` background — a
   direct violation of the one axis that was supposed to stay fixed across
   the whole batch.
10. **Jumping mid-air, clutching parcel** (`Ywt1CBdhX9W9fFdDeJxMEQ`) —
    **KEEP.** Pose matches (fox mid-run/leap with a box under one arm,
    distinct silhouette from image 1's running pose), correct palette, bold
    clean outlines, solid white background.

**Shortlist (6 of 10 kept):** images 1, 2, 3, 5, 8, 10 — response_ids
`e9Se_BwAXciyPuaMLYnnlQ`, `YuDq_SIrVXKNpEk6PDjq8A`, `ghy9IHzaWmG0voDkoq7wuw`,
`C6oefO7JVqK5WTFXWIrfJg`, `mkM_GtahXiqOIEAaPxTcoA`, `Ywt1CBdhX9W9fFdDeJxMEQ`.

The 6 kept images are committed in [`images/`](images/) (fetched via each
asset's `thumbnail_url`, `.../thumbnail/response/<response_id>@2k`):

| Pose | Image |
|---|---|
| Running, holding parcel | [`images/courier-fox-01-running-holding-parcel.webp`](images/courier-fox-01-running-holding-parcel.webp) |
| Sitting, sipping coffee | [`images/courier-fox-02-sitting-sipping-coffee.webp`](images/courier-fox-02-sitting-sipping-coffee.webp) |
| Standing, waving | [`images/courier-fox-03-standing-waving.webp`](images/courier-fox-03-standing-waving.webp) |
| Holding clipboard | [`images/courier-fox-05-holding-clipboard.webp`](images/courier-fox-05-holding-clipboard.webp) |
| Thumbs up | [`images/courier-fox-08-thumbs-up.webp`](images/courier-fox-08-thumbs-up.webp) |
| Jumping mid-air | [`images/courier-fox-10-jumping-midair.webp`](images/courier-fox-10-jumping-midair.webp) |

The 4 rejected images (style drift, hallucinated text, background-lock
violation) were deliberately not downloaded/committed — this directory shows
the shortlisted output of the review-and-cull step, not the raw batch.

**Rejected (4 of 10):** images 4, 6, 7, 9 — one style-drift case where the
rendering departed from the locked bold-outline flat-vector look (image 4),
two cases of hallucinated unrequested text (images 6 and 7), and one
background-lock violation (image 9).

## What was verified

- `generate_images_bulk` really is async/fire-and-forget: the response at
  submission time contained real, populated `request_id`/`response_id`
  arrays with every row already `"status": "running"`, not a placeholder or
  a single batch-level ID.
- `get_generation_status` really does return the entire session's recent job
  table (not just the queried batch) when passed a single `request_id` —
  confirmed on all 3 polls, including an unrelated Bell-Boy job appearing
  in the poll-3 response alongside the 10 courier-fox rows.
- Polling took 3 calls from submission to full completion: all-running →
  1-of-10 done → all-10 done. This run's real timing, not a "typical"
  assumed value.
- `get_recent_generations`'s `thumbnail_url` pattern
  (`.../thumbnail/response/<response_id>@2k`) is a real, directly-downloadable
  WebP image URL — all 10 were fetched and visually reviewed, unlike
  `permalink_url`, which was confirmed (via `curl -sI` and a grep for
  `og:image`) to be a client-rendered SPA page with no embedded image URL in
  its static HTML.
- The locked JSON caption's `compositional_deconstruction.elements` (a
  single `obj` per prompt) is NOT rendered verbatim — Ideogram's backend
  expands it server-side into a richer internal JSON with an added
  `high_level_description` and multiple finer-grained elements. This means
  the batch is not fully immune to the same "unrequested text/detail"
  failure mode that plain-text prompts have — 2 of the 10 images in this run
  (images 6 and 7) got hallucinated text the prompt never asked for, despite
  using the structured-JSON format.
- Not verified by this run: behavior above the 10-prompt job size ceiling
  described here (the ~20-prompt confirmation threshold this run stayed
  under), and whether `get_generation_status` called with no `request_id`
  argument (rather than the passed-in single ID) behaves the same way.
