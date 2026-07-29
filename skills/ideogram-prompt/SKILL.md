---
name: ideogram-prompt
description: Prompting technique guide for Ideogram image generation via the connected Ideogram MCP (generate_image, describe_image, remix_image, edit_image). Use whenever the user wants to generate, remix, or edit an image with Ideogram — including moodboards, style-matching a reference image, extracting a visual "recipe" from existing images and reapplying it to a new subject, or getting precise control over color palette, composition, and in-image text. Trigger this even if the user doesn't say "Ideogram" by name but is clearly asking for image generation and the Ideogram MCP tools are the ones available.
---

# Ideogram Prompting

Ideogram 4 was trained on structured JSON captions, not plain text — a plain-text prompt gets expanded into that structure by a "magic prompt" step before it ever reaches the model. That gives you two ways to prompt, and picking the right one depends on how much control the user actually wants.

## Two modes

**Loose / exploratory** — write a natural-language prompt and let the model's own interpretation (magic prompt) fill in color, lighting, composition. Good for quick ideas, loose briefs, or when the user wants to be surprised. Just call `mcp__ideogram__generate_image` with a `prompt` string; don't over-specify.

**Precise** — when the user names an exact palette, a specific composition, or text that must render legibly, write the prompt as a structured caption instead of a vague adjective list. See `references/json-caption-schema.md` for the full schema (aesthetics/lighting/medium/color_palette, bounding-box elements, etc) — either follow its field structure in prose form, or paste the JSON itself into the `prompt` string as an experiment. Note: the connected `generate_image` tool has no explicit `magic_prompt` toggle (unlike `edit_image`, which does), so JSON-in-prompt isn't a guaranteed bypass the way it is in Ideogram's raw API — compare against a well-written prose version and keep whichever renders closer to what was asked for.

Either way, the highest-leverage lever is **`color_palette`**: up to 16 uppercase `#RRGGBB` hex codes steer the image's dominant colors directly, and up to 5 per element for per-subject control. If the user cares about exact colors, always name them as hex, not as color-adjectives ("teal" vs `#0F766E`).

## Style extraction (reference image → new subject)

When the user has reference images and wants a new subject in the same visual language — see `references/style-extraction-workflow.md` for the full loop: `describe_image` each reference → extract the shared recipe (medium, palette, composition, texture) → apply the recipe to a new subject → generate → compare. This also works directly from a text description with no reference image at all.

## Iterating on a result

- Want "more like this but—": `mcp__ideogram__remix_image`, similarity tuned by `image_weight`.
- Want to literally change something in an existing image: `mcp__ideogram__edit_image`. This is a different operation from remix/generate — if it fails, say so and stop rather than quietly falling back to a "similar" image instead.

## Writing prompts generally

- Replace adjective soup ("modern, clean, professional") with things the model can actually render: an exact hex, a named medium, a real-world reference object, explicit exclusions ("no gradients, no lens flare"). A model that isn't told what to avoid tends to reach for its most common training-data default.
- One clear direction per prompt. If the user wants several options, write several structurally distinct prompts (different composition or medium, not just a palette swap on the same structure) rather than one prompt with multiple options bolted in.
- NSFW prompts get blocked outright, and plain-text prompts trigger the safety filter as false positives more often than structured JSON does — if a reasonable prompt keeps getting blocked, try the structured-caption form before assuming the request itself is the problem.
