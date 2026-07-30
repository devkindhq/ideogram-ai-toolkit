---
name: photographic-icon-set-generator
description: Generates consistent, locked-style sets of photographic, textured, or 3D-material-rendered icons via Ideogram — 3D-rendered, claymation, hand-painted, or isometric-with-material-texture icon packs that hold a shared style recipe across every icon in the batch. Use when the user asks for "3D icon set," "claymation icons," "textured/material icon pack," "isometric icon set with material texture," "photographic icon set," "match this icon style," or "add an icon to my [style] set." Out of scope — flat, line, glyph, minimal, or generic vector-style icon sets (these need deterministic SVG code, not diffusion output); trigger `ideogram-prompt` for single one-off "generate me a 3D camera icon" requests instead. This skill exists specifically for the multi-icon consistency problem, where the same locked style recipe carries across a batch of subject variations.
---

# Photographic Icon Set Generator

Ideogram's `generate_images_bulk`, `remix_image`, and `edit_image` tools let you batch-generate and correct consistent icon sets. This skill orchestrates those calls to hold a shared "style block" — a paragraph of prompt text describing the render technique, lighting, material, and visual language — repeated byte-identical across every icon prompt in the batch, ensuring the set reads as a unified design, not a scatter of unrelated renders.

The single load-bearing technical fact this skill rests on: on Ideogram's default v4 pipeline (no `custom_model_uri` set), the parameters `style_type`, `negative_prompt`, `magic_prompt_option`, and `seed` are all ignored. Consistency has to be carried entirely by holding the style block as a repeated, byte-identical string in the prompt text of every `generate_images_bulk` entry — not by any generation parameter. This means accuracy and discipline in the locked style description is everything.

This is a hybrid skill — prompt-composition discipline (like `logo-prompting` and `character-model-sheet`) *plus* a multi-call orchestration workflow (like `collections-management`). The output is both a "set style spec" JSON (so later sessions can extend the set without re-deriving the recipe) and a project markdown file tracking generations, corrections, and extensions.

## Before you start

Read these three reference files in order before your first generation and before any correction or extension:

1. `references/style-lock-recipe.md` — how to draft, lock, and hold the style block identical across every prompt. Read this before step 2 (Lock the style block).
2. `references/icon-anti-slop-discipline.md` — the pre-generation gate scoring table. Run your drafted prompts through this before calling `generate_images_bulk` (step 4).
3. `references/set-consistency-workflow.md` — the full batch → review → correct → extend loop, including when and how to use `edit_image` with multi-reference anchoring and when the `custom-model-training` escalation path applies.

## Workflow

### 1. Gather the icon list and style direction

Ask for (rather than inventing) the list of icon concepts or subjects (e.g. "clock, notebook, camera, headphones, palette, pencil, document, package, key"), the target count, and one of the four named render styles or a described custom raster style: 3D-rendered (clean product-design lighting, often with soft shadows and reflection surfaces), claymation (chunky, squished, hand-modeled toy aesthetic), hand-painted (textured, artistic, brush marks, organic-feeling), or isometric-with-material-texture (geometric 45-degree angle, textured surfaces showing wood/metal/cloth/glass). If the request is actually for flat, line, glyph, vector, or minimal icon sets, say so now and stop — those are out of scope for this skill; they need deterministic SVG code, not diffusion output. Do not proceed with a raster substitute. There's no artificial cap below `generate_images_bulk`'s own 1-500 `prompts[]` limit, but sanity-check an unusually large requested count with the user first (Pro-subscription and cost implications) rather than queuing a large batch silently.

### 2. Lock the style block

Draft the style block once, following `references/style-lock-recipe.md`'s checklist: material/render technique (the four named styles, or a custom described raster style), lighting (key light direction, fill-light ratio, specular highlights or matte finish), camera angle (overhead, 45-degree isometric, straight-on), background/ground (usually clear, translucent, or flat color to keep focus on the icon), and color story (do icons share a unified palette, or does each one vary?). Write this as one continuous, detailed description — not re-described per icon, not abbreviated differently per subject. This exact text will be repeated byte-identical at the start of every icon's prompt.

### 3. Draft one prompt per icon

Each entry in the eventual `prompts[]` array is: `[style block, held identical] + [subject block, varies per icon: the noun and its core visual identity] + [composition block: isolated single subject, centered, consistent margin, consistent ground/shadow treatment]`. Keep them in the icon list's order. Example for a "3D-rendered product design" clock icon: "[style block about 3D rendering, lighting, and background] A rounded wall clock with a white face, black numerals and hands, minimal design, centered, shadow beneath on neutral gray, single icon isolated." Each prompt is a single paragraph.

### 4. Run the anti-slop gate

Before calling `generate_images_bulk`, score your drafted prompts 1-5 against the five axes in `references/icon-anti-slop-discipline.md`'s pre-generation table: style-block fidelity (is the style block copy-pasted byte-identical into every prompt, not re-typed or paraphrased?), subject-block restraint (does every subject block name one clear icon-object with no second subject, implied scene, or narrative moment?), ground/shadow explicitness (does every prompt name the exact grounded shadow and background treatment from the style block?), material-finish explicitness (does every prompt restate the actual requested surface finish rather than leaving it to the model's glossy-plastic default?), and the set-level restraint check (scanning all N prompts together, does any single subject block risk pulling that one icon toward a different camera framing or material read than the rest?). Anything scoring under 3 on any axis means revise that prompt now — regenerating a full batch is more expensive than fixing N prompt strings first. Do not skip this gate.

### 5. Generate the batch

Call `generate_images_bulk(prompts=[...])` with the prompt array. Set `aspect_ratio` to the user's preference or default to `"1x1"` (square, the standard icon format), `rendering_speed: "QUALITY"` (icon sets benefit from detail consistency and legibility, even if QUALITY is slower), and `resolution` only if the user needs specific pixel dimensions (e.g., `1024x1024` for a standard web icon). Leave `style_type`, `negative_prompt`, `magic_prompt_option`, and `seed` unset — they're ignored on the default v4 pipeline — unless `custom_model_uri` is set via the escalation path in step 8, in which case they become active levers. This is a background job; tell the user images render as they complete, and offer to poll `get_generation_status` if the client is text-only or prefers not to wait for auto-render.

### 6. Review the finished set as a set, not icon-by-icon

Pull all N generated images and look at them together — the entire reason this skill exists is to catch drift that a per-icon review would miss. Check: material consistency (does the 3D rendering, claymation squish, hand-painted texture, or isometric geometry hold across all icons?), lighting consistency (are shadows and highlights in the same direction?), camera-angle consistency (are all icons viewed from the same angle?), and scale consistency (do the icons feel like they belong to the same family, or does one look disproportionately larger or smaller?). If all icons pass, you're done with generation. If you see drift, move to step 7.

### 7. Correct any outlier

Prefer `edit_image` with 2-3 "anchor" icons from the set that show the target style well passed as `image_response_ids`, plus a corrective prompt describing the fix (e.g., "match the material and lighting of these two anchor icons, fix the 3D shading on this one"). Multi-reference conditioning pulls the result toward the group in a way a single reference can't as reliably. Use `remix_image` against one anchor image (with `image_weight` toward the higher end, 70-85, for "match this closely") as the simpler fallback when only one strong anchor exists. Do *not* regenerate the whole batch for a partial-drift problem — a fresh `generate_images_bulk` call has no better odds of consistency without also fixing the underlying style-block text, and it discards the icons that already matched.

### 8. Extend the set later

When adding new icons to an existing set, use the same multi-reference `edit_image` pattern: pass 2-3 existing icons as `image_response_ids` (the anchor style references) plus a prompt for the new subject and composition. Do *not* call `generate_images_bulk` again — a new batch call has no memory of the earlier batch's exact style-block wording. If the project's icon count and consistency needs have grown past what prompt-text-only style-locking is holding up (10+ icons across multiple sessions, or drift becoming harder to correct per-icon), this is the point to raise the `custom-model-training` escalation path documented in `references/set-consistency-workflow.md` — phrased as an option to consider if and when that skill exists in this repo, not something this skill triggers on its own. Custom model training lets you fine-tune a dedicated model on your locked style, making future batches and extensions far more stable.

### 9. Save what was made

Write the locked style block text, the per-icon prompt (in order), the resulting `response_id` and image URL per icon, and any correction or extension notes (which icons were corrected, what `edit_image` call fixed them, what new icons were added via which multi-reference call) to a "set style spec" JSON plus a project markdown file. Check for an existing `logo-explorations/`, `branding/`, or icon-specific folder first and match it — same convention as the sibling skills. Follow "No Context Lost" — nothing generated here should live only in the conversation.

## The "set style spec" artifact

This skill introduces a new JSON shape for storing icon sets so they're reusable across sessions. It's deliberately distinct from `composition-spec-format.md`'s bbox schema (which annotates image layout) and instead focuses on the generation recipe itself:

```json
{
  "style_block": "The exact, byte-identical text repeated across every prompt...",
  "icons": [
    {
      "subject": "clock",
      "prompt": "[style block] + [subject specific text] + [composition text]",
      "response_id": "resp_...",
      "image_url": "https://..."
    },
    ...
  ],
  "corrections": [
    {
      "target_icon": "camera",
      "target_response_id": "resp_...",
      "anchor_icons": ["clock", "notebook"],
      "anchor_response_ids": ["resp_...", "resp_..."],
      "corrective_prompt": "match the material and lighting...",
      "resulting_response_id": "resp_...",
      "resulting_image_url": "https://..."
    }
  ],
  "extensions": [
    {
      "new_icon": "palette",
      "anchor_icons": ["clock", "notebook", "camera"],
      "anchor_response_ids": ["resp_...", "resp_...", "resp_..."],
      "extension_prompt": "add a paint palette...",
      "resulting_response_id": "resp_...",
      "resulting_image_url": "https://..."
    }
  ]
}
```

Why this shape: the `style_block` can be copied verbatim into any new `edit_image` call, and the `icons[]`, `corrections[]`, and `extensions[]` logs mean a later session can add another icon without re-deriving the recipe from scratch or wondering which icons are the strongest anchors for the set's look.

## Error handling

- **Flat/vector/line/glyph request** → state plainly this is out of scope for this skill; it needs deterministic SVG code, not diffusion output. Recommend `ideogram-prompt` for a one-off icon if needed, or a dedicated vector tool if the user needs a full set.
- **Undecided style direction** → ask the user to pick one of the four named styles (3D-rendered, claymation, hand-painted, isometric-with-material-texture) or describe a custom raster style. Do not invent a default silently; the whole value proposition is a *deliberately locked* shared recipe.
- **`generate_images_bulk` requires Pro subscription** → if the call errors on subscription tier, surface the real error and ask before trying anything else. Do not silently fall back to looping individual `generate_image` calls — that breaks the batch consistency workflow.
- **Partial batch failure (some prompts succeed, others fail)** → report exactly which icons failed and why (bad prompt, server error, timeout, etc.). Do not claim the whole set completed if some entries didn't.
- **Style drift detected across the set** → run the targeted correction loop (workflow step 7) first, using 2-3 anchor icons and `edit_image`. Do not regenerate the whole batch blindly without fixing the underlying style-block text.
- **Correcting an outlier the user actually preferred as-is** → confirm which icon is the intended "anchor" for the set's look before conforming others to it. Do not assume the majority of the batch is automatically the correct target if the user liked a minority outlier better.

## Save what you made

After generating, correcting, or extending a set, save the "set style spec" JSON and a project markdown file to the project's existing output location — the same folder `brand-identity-sheet`, `character-model-sheet`, or `moodboard-generator` already save to for this project. Use a file path like `icon-set-<subject>-<style>.json` for the spec and `icon-set-<subject>-<style>-notes.md` for the markdown (e.g., `icon-set-ui-elements-3d-rendered.json` and `icon-set-ui-elements-3d-rendered-notes.md`). Following the toolkit's "No Context Lost" habit: a `response_id` or a locked `style_block` that only exists in the conversation is one the user has to re-derive by hand later; save it now so the next session can extend the set without guessing.

## Reference files

- `references/style-lock-recipe.md` — the discipline for drafting, locking, and holding the style block identical across every prompt: material/render technique, lighting, camera angle, background/ground, color story. The single most important document for this skill. Read before step 2.
- `references/icon-anti-slop-discipline.md` — the pre-generation gate scoring table (style-block fidelity, subject-block restraint, ground/shadow explicitness, material-finish explicitness, set-level restraint check). Run your prompts through this before `generate_images_bulk`. Read before step 4.
- `references/set-consistency-workflow.md` — the full batch → review → correct → extend loop: when to use `edit_image` with multi-reference anchoring, fallback to `remix_image`, the partial-correction pattern, and when the `custom-model-training` escalation path applies (if/when that skill exists in this repo).
