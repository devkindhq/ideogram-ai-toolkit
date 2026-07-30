# Tile Prompt Recipe

Structured-caption recipe for writing a seamless/tileable pattern or texture prompt.
There is no `generate_image`, `generate_images_bulk`, `remix_image`, or `reframe_image`
parameter in the connected Ideogram MCP for seamless tiling, edge-wrap, or repeat
preview — every one of the five elements below exists because the prompt prose is
carrying work an API flag would otherwise do. Write all five into the prompt, in the
order given, before running the anti-slop gate (`references/anti-slop-discipline.md`)
or calling `generate_image`.

## 1. Repeat type

Name the repeat type explicitly, by name, in the prompt. Three repeat types this skill
supports:

- **Straight repeat** — motifs align on a simple grid, same position in every tile.
  This is the default when the user says "tile" or "pattern" without specifying a
  repeat type. Default to straight and say so explicitly in your response (per Workflow
  step 1) — half-drop and brick repeats require offset math (shifting alternating rows
  or columns by a fraction of the tile dimension) that this skill has no crop/offset
  tool to verify, so silently defaulting to one without saying so would misrepresent
  what was actually produced.
- **Half-drop repeat** — alternating columns are offset vertically by half the tile
  height, common in floral and damask prints.
- **Brick repeat** — alternating rows are offset horizontally by half the tile width,
  common in masonry-style and some geometric prints.

Example prompt fragments:

- Straight: "designed as a straight repeat, motifs aligned on a simple grid."
- Half-drop: "designed as a half-drop repeat, alternating columns offset by half the
  tile height."
- Brick: "designed as a brick repeat, alternating rows offset by half the tile width,
  like a running-bond brick course."

## 2. Motif scale

State motif scale relative to the tile edge, not in absolute units — the model has no
reliable sense of physical size, but "relative to the tile" gives it a proportion it can
act on. Three scale bands:

- **Fine scale** — motifs are small and dense relative to the tile, many repeats
  visible even at a small zoom. Example: "fine-scale ditsy floral, motifs no larger
  than 1/20th of the tile width, densely scattered."
- **Medium scale** — motifs occupy a clear but moderate fraction of the tile, a
  handful of repeats visible per tile edge. Example: "medium-scale geometric motif,
  each unit roughly 1/6th of the tile width, evenly spaced."
- **Large scale** — motifs are large relative to the tile, sometimes only one or two
  full motifs visible per tile edge, a "statement" read. Example: "large-scale
  statement floral, a single bloom cluster spanning nearly half the tile."

If the user gives no scale, ask or default to medium scale and say so explicitly, the
same disclosure discipline as the repeat-type default.

## 3. Edge-continuity language

This is the entire mechanism this skill has for seamlessness, not a nice-to-have
descriptor. There is no mechanical tiling flag anywhere in the connected surface —
edge-continuity language in the prompt, plus the visual-verification step in
`references/tiling-verification.md`, is the whole strategy. Treat this section as
non-optional in every prompt this skill writes.

Include language in this category, adapted to the specific motif:

- "motif elements cross and reconnect at all four edges"
- "no motif is cropped or orphaned at a tile boundary"
- "the pattern reads as an infinite continuous field, not a single bounded
  illustration"

Example: for a floral straight repeat — "leaf and stem elements cross the tile edges
and reconnect seamlessly on the opposite side, no motif is cropped or orphaned at a
boundary, the composition reads as an infinite continuous field rather than a single
bounded illustration."

Because there is no offset/crop tool backing this up, this phrasing is doing 100% of
the seamlessness work at generation time — the 2×2 verification step in
`references/tiling-verification.md` is the only check on whether it worked, not a
formality after the fact.

## 4. Flat/top-down framing

State the framing plainly: flat, top-down, no directional single-source lighting, no
cast shadow, no vignette. Any of these three breaks the tiled read, because they all
imply a fixed light source or fixed viewpoint relative to a single bounded frame —
which directly contradicts "infinite repeating field." A cast shadow or vignette on one
tile looks wrong the instant it repeats, because the shadow/vignette repeats with it
instead of reading as one continuous light source across the whole field.

Example: "flat, top-down orthographic view, even diffused lighting with no directional
shadow, no vignette, no drop shadow, full-bleed edge to edge with no border or frame."

## 5. DESIGN-vs-REALISTIC register

`style_type` is a no-op on the default v4 generation path this skill uses — it only
takes effect when `custom_model_uri` is set (the custom/v3 pipeline). This skill does
not rely on `custom_model_uri`, so the DESIGN-vs-REALISTIC register must be written
into the prompt prose directly. Never tell the user `style_type` will set this register
on a default-path call.

Two full example prompt-opening sentences, one per register:

- **DESIGN register** (flat graphic surface pattern): "A flat vector-style graphic
  surface pattern, hard-edged shapes, no gradients or lighting, solid flat color
  fills."
- **REALISTIC register** (photographic material texture): "A photographic close-up
  texture of woven cotton fabric, natural fiber detail, even diffused studio lighting
  with no directional shadow."

Both sentences carry the register entirely through word choice and description — never
through a parameter. State the register explicitly and early in the prompt, in the
opening sentence, so it isn't lost among the repeat-type, motif-scale, and
edge-continuity language that follows it.
