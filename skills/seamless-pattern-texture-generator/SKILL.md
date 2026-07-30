---
name: seamless-pattern-texture-generator
description: Writes, generates, visually verifies, and iterates on seamless/tileable surface-design patterns — florals, geometrics, stripes, damask, plaids, novelty prints — and tileable material textures — fabric weaves, paper grain, wood, stone, leather, packaging substrates — for wallpaper, packaging, textile print, and digital surface design. Trigger on explicit requests using phrases like "seamless pattern," "tileable," "repeating pattern," "wallpaper design," "fabric print/texture," "packaging surface pattern," "surface design," or "this needs to tile edge to edge." Triggers on explicit request only — it does not auto-wire from `brand-identity-sheet`, `moodboard-generator`, or any other sibling skill; those skills are unmodified and unaware of this one, the same non-auto-wiring precedent `collections-management` sets for its own triggers.
---

# Seamless Pattern & Texture Generator

No `generate_image`, `generate_images_bulk`, `remix_image`, or `reframe_image` call in
the connected Ideogram MCP exposes a seamless/tileable boolean, an edge-wrap mode, or an
offset/repeat-preview capability — confirmed by direct inspection of those tools'
schemas, not assumed from docs. Seamlessness here is achieved entirely through prompt
language plus a best-effort visual-verification step (Workflow step 5). Never state or
imply that a generated tile is mechanically guaranteed to be seamless — it is
prompt-engineered and visually checked, not API-enforced.

`style_type`, `negative_prompt`, and `seed` are no-ops on the default v4 generation path
this skill uses — they only take effect when `custom_model_uri` is set (the custom/v3
pipeline). Do not rely on them to set the DESIGN-vs-REALISTIC register or to exclude
motifs/clichés in this skill's prompts; write the register and the exclusions directly
into the prompt prose instead.

This skill covers two related deliverable families in one place — repeating
surface-design patterns and tileable textures/materials — because both are the same
underlying problem: one tile that has to read as continuous when it repeats. Structurally
this is a prompt-composition skill like `ideogram-prompt`, `logo-prompting`, and
`moodboard-generator` (one prompting problem plus a visual-verification loop), not a
multi-call orchestration workflow like `collections-management` or
`custom-model-training`.

## Before you start

Read all four reference files before running any workflow step:

- `references/tile-prompt-recipe.md` — the structured-caption recipe for writing the
  tile prompt itself.
- `references/tiling-verification.md` — the visual-QA step used in Workflow step 5.
- `references/anti-slop-discipline.md` — the pattern-specific slop bans and the
  pre-generation gate used in Workflow step 3.
- `references/composition-spec-format.md` — the shared JSON schema, adapted here to
  motif instances, used in Workflow step 7.

## Workflow

### 1. Gather the brief

Collect: repeat category (floral / geometric / stripe / damask / plaid / novelty /
material-texture), end use (fabric / wallpaper / packaging / general surface digital
use), colorway (exact hex if the user has one, a named palette if not), motif scale, and
any reference image or material name the user is pointing at.

If the user says "tile" without specifying straight, half-drop, or brick repeat, default
to a straight repeat — and say so explicitly in your response. State the reason:
half-drop and brick repeats require offset math this skill has no crop/offset tool to
verify, so silently defaulting to one without saying so would misrepresent what was
actually produced.

### 2. Write the prompt

Follow `references/tile-prompt-recipe.md`. The prompt must state the seamless/tileable
framing explicitly and early, include edge-continuity instructions, specify flat
top-down framing with no directional lighting, exclude any border or vignette, and spell
out the DESIGN-vs-REALISTIC register in prose — never via `style_type`, which is inert
on the default v4 path this skill uses. If the user gave an exact hex palette, write the
exact hex values into the prompt rather than a named-color approximation.

### 3. Run the anti-slop gate before generating

Before calling `generate_image`, score the drafted prompt against the pre-generation
gate in `references/anti-slop-discipline.md`. Of everything on that gate, border/vignette
presence and the copy-paste-rotation tell are the two checks worth failing the prompt
over — both break tileability directly, not just aesthetics, so a prompt that fails
either one gets revised before it's ever sent to the model.

### 4. Generate

Use `mcp__ideogram__generate_image` for a single tile. Use
`mcp__ideogram__generate_images_bulk` (1-500 prompts) when the user wants several
distinct colorways or scales rendered at once — batch them as one bulk call, not a loop
of single `generate_image` calls. Default to `aspect_ratio: "1x1"` as the repeat-unit
shape (a square tile) and `rendering_speed: "QUALITY"` for the deliverable render —
edge-continuity is a composition/legibility-sensitive property the same way panel text
is for `moodboard-generator`, and it's worth the slower render.

### 5. Verify

Per `references/tiling-verification.md`, generate a second, explicitly-labeled
verification-only image asking for the same pattern shown repeated 2×2 with aligned
edges, and report honestly whether seams are visible. State plainly that this is a
visual approximation performed by the skill itself, not a mechanical guarantee, and do
not imply the base tile is production-proven.

### 6. Style-lock variants

Once the user reacts to a base tile, use `mcp__ideogram__remix_image` for
colorway/scale variations that keep the same motif. Use `image_weight` in the 60-80
range — tighter than `moodboard-generator`'s 30-50 range — because a pattern variant
needs to preserve the exact repeat structure, not just "same energy."

### 7. Compositional deconstruction

Write the JSON breakdown per the adapted `references/composition-spec-format.md`:
motif instances and how they connect across the tile's edges, so the tile's structure is
reusable later without re-describing it from scratch.

### 8. Save

Write the prompt, the JSON, the verification result, and the image URL(s) to
`04-projects/<project>/` — check for an existing patterns/surface-design folder first
and match whatever convention that project already uses. Nothing generated in this
skill should live only in the conversation, per the "No Context Lost" rule.

## Error handling

- **No native tiling flag exists anywhere in the connected surface.** Never state or
  imply a generated tile is mechanically guaranteed seamless — always frame it as
  prompt-engineered and visually verified.
- **Verification image shows visible seams.** Say so plainly and revise the prompt
  (stronger edge-continuity language, a simpler or larger motif, remove lighting/shadow
  cues) rather than shipping the seamed tile as if it passed.
- **Bordered or vignetted output** (a common default reading of "pattern" prompts).
  Treat it as a failed generation requiring a prompt revision ("no border, no frame, no
  vignette, full-bleed edge to edge"), not something to upscale or ship as-is.
- **Ambiguous repeat type** — the user says "tile" with no straight/half-drop/brick
  specified. Default to straight and state the default explicitly, per Workflow step 1.
- **`style_type`/`negative_prompt` set by mistake on a non-custom-model call.** They are
  silently ignored by the API, not an error. Don't rely on them for register or
  exclusions on the default v4 path, and say so plainly if the user expects them to have
  an effect.
- **`generate_images_bulk` partial failures.** Report per-prompt outcomes honestly
  rather than treating the whole batch as done if some prompts failed.

## Reference files

- `references/tile-prompt-recipe.md` — the structured-caption recipe for writing a
  seamless/tileable pattern or texture prompt.
- `references/tiling-verification.md` — the 2×2 repeat visual-QA step and how to report
  the result honestly.
- `references/anti-slop-discipline.md` — the pattern-specific slop bans and the
  pre-generation gate.
- `references/composition-spec-format.md` — the JSON schema for the compositional
  deconstruction, adapted to motif instances.
- `examples/` — worked pattern/texture jobs go here as real jobs are run; empty for now.
