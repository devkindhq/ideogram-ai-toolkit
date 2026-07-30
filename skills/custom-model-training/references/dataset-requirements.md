# Dataset & Training Requirements

This file separates what's actually confirmed about the Ideogram custom-model-training
tools from what's genuinely unknown. Per the standing rule against stating third-party
API behavior as fact without a verified source, the unknowns below are flagged as
unknowns rather than filled in with plausible-sounding numbers — treat them as things
to discover by running the pipeline, not settled facts.

## Confirmed (from direct inspection of the live tool schemas)

- **Minimum dataset size: 15 images.** Confirmed empirically (2026-07-30) — calling
  `train_model` on a dataset with only 5 uploaded assets returned a 400 error:
  `"Dataset requires at least 15 images for training, but only 5 were provided."`
  Don't attempt `train_model` until the dataset has at least 15 successfully uploaded
  assets — check the dataset's asset count first if there's any doubt.
- `create_dataset(name: string)` — takes only a `name`. No other required fields.
- `upload_dataset_assets(dataset_id: string, filenames: string[])` — returns one
  `curl_command` per filename. Each upload URL is **one-time, short-lived, and
  single-use**. The tool's own description explicitly warns to run the returned
  commands **sequentially**, not concurrently.
- `train_model(dataset_id: string, model_name: string)` — starts an **asynchronous**
  training job. The call returns before training completes.
- `get_model(model_id: string)` — returns the current state of a specific trained
  model, keyed by its identifier.
- `list_models(scope?: "owned" | "shared", status?: string[])` — can filter by scope
  and status, but the exact values `status` accepts aren't documented anywhere findable
  — see below.
- `list_datasets()` — no parameters.

## Unverified / unknown

Don't assert any of these as fact — they're things to observe empirically the first
time the pipeline actually runs, and worth telling the user about once observed:

- **Supported image formats.** Not documented. WEBP uploads succeeded in testing.
  If an upload fails specifically on format grounds, that's a real error to surface,
  not something to guess around in advance.
- **Content that trips the safety check.** `upload_dataset_assets` can silently reject
  an individual file with `FAILED_SAFETY_CHECK` even for an apparently benign
  brand-identity image (observed 2026-07-30, cause unclear) — check the response's
  `failed_assets` array after every upload rather than assuming success from a 200
  HTTP status, and don't count a failed asset toward the 15-image minimum.
- **Expected training duration: 1h19m–20+ hours, highly variable.** Confirmed empirically
  (2026-07-30) by comparing `creation_time` to `last_update_time` across a batch of
  `COMPLETED` models returned by `list_models` (other organizations' shared models,
  not ours — but same training pipeline): observed durations ranged from ~1h35m
  ("Precision details") to ~20h40m ("Apple Beats"), with most falling in the 1.5-4 hour
  range. Our own trained model (`fizzwright-hopcarry-brand-v1`, 15 images — the
  documented minimum) completed in **1h19m** (`2026-07-30T02:34:48Z` →
  `2026-07-30T03:54:09Z`), the fastest observed so far, and the only data point where
  the dataset size is actually known. This is one sample, not a trend — don't assume a
  15-image dataset always trains this fast. Do NOT expect training to finish within
  minutes. Poll every 10-15 minutes for the first hour, then widen to 30-60 minute
  intervals — polling every minute wastes calls against a job that realistically takes
  over an hour.
- **`last_update_time` does not change during training.** Confirmed empirically
  (2026-07-30): while `status` is `"TRAINING"`, `last_update_time` stays identical to
  `creation_time` — it does not tick forward as training progresses. This is normal,
  not a sign of a stalled job. It only updates once the job transitions to a terminal
  state (e.g. `"COMPLETED"`). Don't treat a frozen `last_update_time` alone as evidence
  of failure — only genuinely excessive elapsed time (many hours past the observed
  range above) or an explicit failure-looking status value warrants flagging it to the
  user.
- **Confirmed status field: `get_model` returns a top-level `status` string.** Observed
  values so far: `"TRAINING"` (in progress) and `"COMPLETED"` (done, `is_available_for_generation: true`
  and `custom_model_uri` populated with a value like `"model/<id>/version/0"`). No
  failure-state value has been observed yet — if one appears, record it here.

## Why this matters

Every one of these unknowns becomes obvious in about one minute of actually running the
pipeline. Guessing at them ahead of time doesn't save time — it just risks the skill
confidently stating something wrong (e.g. "you need at least 10 images") that has no
basis, which is worse than no answer at all. Tell the user to expect a bit of
trial-and-error on first run, and once real values are observed, they can be added here
as confirmed facts.
