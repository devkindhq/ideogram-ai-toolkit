# Tiling Verification

The 2×2 repeat visual-QA step used in Workflow step 5, and how to report the result
honestly.

## Why this exists

No `generate_image`, `generate_images_bulk`, `remix_image`, or `reframe_image` call in
the connected Ideogram MCP exposes a pixel-level tile/offset tool — there is no
mechanical way, anywhere in this skill's connected surface, to check whether a generated
tile actually repeats cleanly (no Photoshop-style offset filter, no edge-wrap preview, no
seamless/tileable flag to inspect). Mechanical verification simply isn't available here.

Verification instead means asking Ideogram itself to generate a second image: the same
pattern shown repeated in a labeled 2×2 grid with edges aligned, then visually inspecting
that image for seam artifacts. This is a prompt-driven visual check, not a pixel-exact
measurement — it approximates what a tiling tool would confirm mechanically, using the
same generation model that produced the base tile.

## Verification-prompt template

Do not send a bare "make it tile" or "show this repeated" follow-up with no context —
the verification call needs the base tile's motif, palette, and composition restated
explicitly, plus an explicit label marking it as a verification-only render so it is
never confused with the base tile deliverable when saving results in Workflow step 8.

Structure:

1. **Restate the base tile** — motif, palette (exact hex if used), repeat type, and scale,
   in the same terms used in the original prompt (per `tile-prompt-recipe.md`).
2. **State the verification request explicitly** — "this exact pattern repeated in a 2×2
   grid, edges aligned, no visible seam."
3. **Label it as verification-only** — state plainly this render is a QA check, not a new
   deliverable. Its image URL and the honest seam-check outcome still get written to
   `04-projects/<project>/` per Workflow step 8 (`SKILL.md` lists "the verification
   result" and "the image URL(s)" as separate save items for exactly this reason) — what
   must never happen is presenting it, labeled or unlabeled, as an alternate version of
   the base tile itself.

Example template:

> "The same [motif/palette/repeat-type/scale description restated from the base tile
> prompt], shown repeated in a 2×2 grid, edges aligned, no visible seam between tiles.
> This is a verification-only render to check tiling continuity, not a new pattern
> design."

## Seam-artifact checklist

Inspect the 2×2 proof image for each of the following:

- **Motif discontinuity at the tile boundary** — a shape that reads as whole in one tile
  but is cut off, truncated, or orphaned in the neighboring tile.
- **Visible color-band edge** — a hard line or color shift where the four copies meet,
  even if no shape is broken.
- **Grid line or seam artifact** — an obvious line running down the vertical or
  horizontal center of the composite, distinct from the pattern's own design elements.
- **Lighting or shadow discontinuity across the boundary** — any directional
  light/shadow cue that visibly breaks or reverses at a tile edge. This is a giveaway
  that the base tile wasn't flat/top-down per `tile-prompt-recipe.md`'s framing rule
  (section 4), since a fixed light source on a single bounded tile cannot repeat
  seamlessly across an infinite field.

If any of these are present, treat the tile as failing verification per the Error
handling section in `SKILL.md`: report it plainly and revise the prompt (stronger
edge-continuity language, a simpler or larger motif, remove lighting/shadow cues) rather
than shipping the seamed tile as if it passed, and never silently upscale a seamed tile
to try to hide the artifact.

## Honest framing and production handoff

This verification step is a best-effort visual approximation the skill performs itself
using Ideogram's own image generation — it is not a pixel-exact check and not a
production guarantee.

- If the 2×2 proof image shows no visible seams under the checklist above, report that
  honestly as **"no visible seams in this best-effort check."** Never report it as
  "guaranteed seamless," "confirmed seamless," or "production-ready" — those are
  mechanical-check claims this skill cannot back up, since it has no pixel-level
  tile/offset tool.
- Production tiling — for example, handing a pattern to a textile mill for print — needs
  a human or downstream-tool check (a Photoshop offset filter or equivalent) before
  print. This skill has no such tool in its connected surface, so that check must happen
  outside this skill, and the deliverable should be described to the user as
  visually-verified, not production-verified.
