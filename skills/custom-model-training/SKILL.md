---
name: custom-model-training
description: Trains a custom Ideogram model on a folder of reference images, then generates a new image with that trained model to prove it works — a real end-to-end pipeline (create_dataset, upload_dataset_assets, train_model, poll get_model, generate_image with custom_model_uri), not just an explanation of how training works. Use whenever the user wants to "train a custom model," "fine-tune" on their own images, get "consistent" output of a specific character, product, or brand across many future generations, lock in a look so it doesn't drift between generations, or turns a set of reference images (including a previously generated brand-identity-sheet or character-model-sheet) into a reusable model. Also trigger this when the user asks what a `custom_model_uri` is or how to get one — that URI only exists after running this pipeline. Distinct from `brand-identity-sheet` and `character-model-sheet`, which produce the single locked reference image in the first place; this skill is the next step, turning that locked image (or any folder of references) into a model you can call repeatedly.
---

# Custom Model Training

Ideogram lets you train a custom model on your own reference images, then generate new
images that stay consistent with those references via a `custom_model_uri` passed to
`generate_image`. This is the skill that closes the loop other skills in this toolkit
start: `brand-identity-sheet` locks a brand system into one image; `character-model-sheet`
locks a character into one multi-panel turnaround. Once that reference exists, this
skill turns it (plus any other reference images) into a model that generates on-brand
or on-character assets indefinitely, instead of re-describing the same look in every
future prompt and hoping it stays consistent.

Always run the pipeline — create the dataset, upload the images, kick off training,
poll until it's ready, and generate a proof image with the trained model — rather than
stopping after `train_model` and telling the user to check back later. The prompt-only
version of this skill would just be a description of the Ideogram API; the value is in
actually running it, watching training through to completion, and coming back with a
generated image that demonstrates the model works.

## Before you start: read the honest facts

Read `references/dataset-requirements.md` before running the pipeline. It splits what's
actually confirmed about these tools (from direct inspection of their schemas) from
what's genuinely unknown (minimum image count, training duration, the exact "ready"
status value). Don't invent numbers for the unknowns — tell the user what's confirmed
and what you're finding out by trying it, per the standing rule against stating
third-party API behavior as fact without a verified source.

## Workflow

### 1. Resolve the input

Ask for (or confirm) a local folder of reference images if the user hasn't pointed to
one already. This version of the skill only supports a folder of existing images —
if the user wants to train on images generated earlier in this session, save those to
a folder first, then proceed the same way.

If the folder is empty, missing, or the path doesn't resolve, stop and ask for a valid
path. Don't create a dataset from nothing — an empty or near-empty dataset produces a
model that's not meaningfully trained on anything, and there's no way to walk that back
after `train_model` has started.

### 2. Create the dataset

Call `mcp__ideogram__create_dataset` with a descriptive `name` (the brand or character
name plus something like "-training-set" reads well in `list_datasets` later). Keep
the returned `dataset_id` — every subsequent call needs it.

### 3. Upload the reference images

Call `mcp__ideogram__upload_dataset_assets` with the `dataset_id` and the list of
filenames. It returns one `curl_command` per file. Run each one **sequentially** via
Bash — not in parallel, and not batched. The tool's own description warns that each
upload URL is single-use and short-lived, so firing them concurrently (or waiting too
long between generating and running one) risks a URL going stale before it's used.

If a `curl_command` fails, don't retry it as-is — a failed or stale single-use URL is
dead. Surface the real error to the user, and if the upload genuinely needs retrying,
call `upload_dataset_assets` again for that file to get a fresh URL.

### 4. Start training

Call `mcp__ideogram__train_model` with `dataset_id` and a `model_name`. This kicks off
an asynchronous job — the call returns before training finishes, so treat its response
as "training started," not "training done."

### 5. Poll until the model is ready

Call `mcp__ideogram__get_model` with the returned model identifier, on a reasonable
interval (a minute or so between checks is a sane starting point — training duration
isn't documented anywhere, so there's no known target to time against). Since the exact
status field/enum isn't confirmed ahead of time, read whatever fields the response
actually contains each time rather than assuming a specific value like `"status": "ready"`
in advance — note what the field is actually called and what values it takes once you
see them, since that's genuinely useful information to report back.

If polling goes on for an unreasonably long time, or the response settles into a state
that looks like failure rather than "still training," don't keep silently polling —
tell the user what you're seeing and ask whether to keep waiting.

### 6. Generate with the trained model

Once `get_model` indicates the model is ready, pull the `custom_model_uri` from its
response and call `mcp__ideogram__generate_image` with `custom_model_uri` set, using a
prompt that exercises what the model should now do consistently (e.g. the brand's
wordmark on a new application, or the character in a new pose). This is the actual
proof that training worked — a "training complete" status alone doesn't confirm the
model generates anything useful.

### 7. Save what you made

Following the toolkit's "No Context Lost" habit: save the dataset name, `dataset_id`,
`model_name`, `custom_model_uri`, the final generation prompt, and the resulting image
to the project's branding folder (wherever `brand-identity-sheet` or
`character-model-sheet` already save their output for this project — match that
location). A `custom_model_uri` that only exists in the conversation is one the user
has to rediscover by calling `list_models` later; save it now.

## Reference files

- `references/dataset-requirements.md` — confirmed vs. unverified facts about
  `create_dataset`, `upload_dataset_assets`, `train_model`, and `get_model`. Read this
  before running the pipeline, not after something goes wrong.
