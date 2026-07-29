# Style extraction workflow

Use this when the user has one or more reference images and wants a new subject rendered in that same visual language — a moodboard photo, a competitor's ad, a piece of art they like, or a batch of past outputs they want to stay consistent with.

## Steps

1. **Describe each reference.** Call `mcp__ideogram__describe_image` on each reference image URL. This returns a structured description covering medium, palette, lighting, composition, texture/grain, and mark-making.
2. **Extract the shared recipe.** If there's more than one reference, don't average them into mush — name what's actually shared (e.g. "three spot colors across all three, even though each image reads differently") and what's incidental to just one image. Write the recipe as prose grouped under a few headers: Medium, Palette, Composition, Texture. Keep it as reusable technique, not a description of the specific reference photo.
3. **Save the recipe** as a JSON or markdown block if the user wants to reuse it across sessions — paste-back-in should be enough for the skill to reapply the same look to a new subject next time.
4. **Apply to a new subject.** Write a fresh prompt (prose or JSON caption, see `json-caption-schema.md`) that carries the recipe's medium/palette/composition/texture into the new subject. Don't just copy the reference's literal content — the goal is the same visual *system*, applied to something new.
5. **Generate and compare.** Use `mcp__ideogram__generate_image`. If the result drifts from the recipe, tighten the weakest field first (usually palette or medium) rather than rewriting the whole prompt.

## Direct style description (no reference image)

The same recipe structure works from a plain description with no image at all — e.g. "1970s Italian horror movie poster, warm earth-tone palette, bold condensed display type, heavy grain." Convert the description straight into the Medium/Palette/Composition/Texture fields and generate; skip the `describe_image` step since there's nothing to describe.

## Iterating on an existing generation

- `mcp__ideogram__remix_image` — variation guided by a new prompt, similarity controlled by `image_weight` (higher = closer to parent). Use when the user wants "more like this but—".
- `mcp__ideogram__edit_image` — literal edit of an existing image (add/remove/change something in it). Use only when the user means "change this specific image," not "make something similar." Don't substitute `remix_image` or `generate_image` if `edit_image` is what they asked for and it fails — surface the failure instead of silently switching tools.
