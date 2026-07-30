# Seamless Pattern & Texture Generator Skill — Design

## Why

Gap-research (see `04-projects/devkind/ideogram-ai-toolkit.md` in the Devkind vault, dated 2026-07-30) identified seamless pattern/texture generation as a Tier 1 gap: zero confirmed prior art anywhere in the Claude Code skill ecosystem for this deliverable type — fabric, wallpaper, packaging, and surface-design tiles built on Ideogram. It's an uncontested space; no differentiation angle beyond matching the repo's existing house methodology is required.

The connected Ideogram MCP exposes `generate_image`, `generate_images_bulk`, and `remix_image` for this workflow (confirmed live via direct tool-schema inspection, not assumed from docs). Direct inspection also surfaced the fact that shapes this entire design: **none of `generate_image`, `generate_images_bulk`, or `remix_image` (nor `reframe_image`) expose a seamless/tileable boolean, an edge-wrap mode, or an offset/repeat-preview capability.** There is no mechanical "make this tile" switch anywhere in the connected surface. Seamlessness here is achieved entirely through prompt language plus a best-effort visual verification step — never asserted as a guarantee the API enforces. This is the load-bearing fact for the whole spec, in the same spirit as `custom-model-training`'s honest split of confirmed-vs-unverified dataset requirements.

A second confirmed fact from the same schema read: `style_type`, `negative_prompt`, and `seed` on `generate_image`/`generate_images_bulk` apply **only** when `custom_model_uri` is set (custom/v3 pipeline) and are silently ignored on the default v4 path this skill uses. That means the "flat graphic pattern" vs. "photographic woven texture" register, and any "no border / no seam" exclusion, has to be written into the prompt prose itself — not delegated to `style_type` or `negative_prompt`, which will not error but will also do nothing on a standard call.

## Scope

One new skill: `skills/seamless-pattern-texture-generator/`. Covers two related deliverable families:

- **Repeating surface-design patterns** — florals, geometrics, stripes, damask, plaids, novelty/conversational prints — for wallpaper, packaging, textile print, and digital surface design.
- **Tileable textures/materials** — fabric weaves, paper grain, wood, stone, leather, packaging substrate textures — where photographic material fidelity matters more than a graphic motif.

Both are the same underlying problem (a single tile that must read as continuous when repeated), so one skill covers both rather than splitting by deliverable type. Triggers on explicit request only ("seamless pattern," "tileable texture," "repeating pattern," "wallpaper design," "fabric print," "surface pattern," "this needs to tile") — no proactive auto-wiring into `brand-identity-sheet`, `moodboard-generator`, or any other sibling skill, matching the `collections-management` precedent.

Raster tile art only, in the register Ideogram can actually produce. Excluded: engineering-repeat production output (half-drop/brick repeats measured to exact print-mill tolerances, vector separations, Pantone-matched production files) — see Out of scope.

## Architecture

Structural pattern matches the **prompt-composition** skills (`ideogram-prompt`, `logo-prompting`, `moodboard-generator`, `brand-identity-sheet`, `character-model-sheet`), not the orchestration-workflow pattern of `collections-management`/`custom-model-training`. Generating one seamless tile is fundamentally a single-prompt engineering problem — write the prompt correctly, generate, verify visually, iterate — not a multi-step sequential-API pipeline. `SKILL.md` + `references/` + `examples/` + `evals/evals.json`, same as every prompt-composition skill.

One adaptation from the panel-grid skills: the shared compositional-deconstruction schema (`high_level_description` + `elements[]` of `{type, bbox, desc}`) is reused unmodified, but `elements` here describe **motif instances placed within one continuous tile canvas** (e.g. "large floral cluster, upper-left, stems connecting diagonally to the cluster at lower-right to preserve edge continuity") rather than named UI panels the way `moodboard-generator`'s 3×3 grid does. This is a structural adaptation, not a schema change.

## Components

- **`SKILL.md`** — frontmatter triggers on: "seamless pattern," "tileable," "repeating pattern," "wallpaper design," "fabric print/texture," "packaging surface pattern," "surface design," "this needs to tile edge to edge." Body documents the workflow below and states the no-native-tiling-flag limitation up front, the same way `ideogram-prompt`'s SKILL.md states the JSON-caption/magic-prompt mechanics up front.
- **`references/tile-prompt-recipe.md`** — the structured-caption recipe for a seamless tile: repeat type (straight/half-drop/brick — default straight, see Data flow step 1), motif scale (fine/medium/large relative to tile edge), edge-continuity language ("motif elements cross and reconnect at all four edges," "no motif is cropped or orphaned at a tile boundary"), flat/top-down framing (no directional single-source lighting, no cast shadow, no vignette — any of these breaks the tiled read), and the DESIGN-vs-REALISTIC register written in prose (since `style_type` is a no-op on the default v4 path — see Why).
- **`references/tiling-verification.md`** — the visual-QA step: since there is no pixel-level tile/offset tool in the connected surface, verification means generating a second, explicitly-labeled proof image describing "this exact pattern repeated in a 2×2 grid, edges aligned, no visible seam" and inspecting it for seam artifacts (motif discontinuity, color-band edges, an obvious grid line down the middle). Documents that this is a best-effort visual approximation, not a pixel-exact guarantee, and that production tiling (e.g. for a textile mill) should get a human/downstream-tool check (a Photoshop offset filter or equivalent) before print.
- **`references/anti-slop-discipline.md`** — pattern-specific slop bans (adapted from the sibling anti-slop files): a hard border or vignette framing the tile (the single most common failure — a framed "pattern swatch" image cannot tile), an obvious repeated-rotation tell (the exact same motif instance copy-pasted at a visible grid interval with zero variation), muddy default "boho" florals or "modern geometric" filler with no stated palette behind it, gradient-mesh backgrounds standing in for a real texture, halftone-dot spam as generic "texture" filler, and directional single-source lighting/cast shadows (breaks the flat tiled read).
- **`references/composition-spec-format.md`** — the shared JSON schema, adapted per Architecture above: `elements[]` entries describe motif instances and how they cross tile edges, not named panels.
- **`examples/`** — one worked example, produced by actually running the pipeline once (empty until then).
- **`evals/evals.json`** — 3 eval prompts in the existing schema (`id`, `eval_name`, `prompt`, `expected_output`, `files`).

## Data flow / workflow

1. **Gather the brief.** Repeat category (floral / geometric / stripe / damask / plaid / novelty / material-texture), end use (fabric / wallpaper / packaging / general surface), colorway (hex if known, named palette if not), motif scale, and any reference image or material name. If the user says "tile" without specifying straight/half-drop/brick, default to a **straight repeat** and say so explicitly — half-drop/brick require offset math this skill cannot verify without a crop/offset tool it doesn't have, so silently defaulting to one without saying so would misrepresent what was actually produced.
2. **Write the prompt** per `references/tile-prompt-recipe.md`: seamless/tileable framing stated explicitly and early in the prompt, edge-continuity instructions, flat top-down framing with no directional lighting, no border/vignette, the DESIGN-vs-REALISTIC register spelled out in prose (not via `style_type`, which is inert on the default v4 path), and the exact hex palette if the user gave one.
3. **Run the anti-slop gate** (`references/anti-slop-discipline.md`) before generating — border/vignette presence and the copy-paste-rotation tell are the two checks worth failing the prompt over, since both directly break tileability, not just aesthetics.
4. **Generate.** `mcp__ideogram__generate_image` for a single tile; `mcp__ideogram__generate_images_bulk` (1-500 prompts) when the user wants several distinct colorways or scales rendered in parallel — batch these as one bulk call, not a loop of single calls. `aspect_ratio: "1x1"` as the default repeat-unit shape (square tile), `rendering_speed: "QUALITY"` for the deliverable render, since edge-continuity is a composition/legibility-sensitive property in the same way panel text is for `moodboard-generator`.
5. **Verify.** Per `references/tiling-verification.md`, generate a second, explicitly-labeled verification-only image asking for the same pattern shown repeated 2×2 with aligned edges, and report honestly whether seams are visible. This is a visual approximation the skill performs itself — it is not a guarantee, and the skill says so rather than implying the base tile is production-proven.
6. **Style-lock variants.** Once the user reacts to a base tile, use `mcp__ideogram__remix_image` for colorway/scale variations that keep the same motif — tune `image_weight` toward the higher end (60-80) since a pattern variant needs to preserve the *exact* repeat structure, not just "same energy," which is a tighter constraint than `moodboard-generator`'s loose 30-50 remix range.
7. **Compositional deconstruction.** Write the JSON breakdown per the adapted `composition-spec-format.md` — motif instances and how they connect across edges — so the tile's structure is reusable later without re-describing it from scratch.
8. **Save.** Prompt, JSON, verification result, and image URL(s) to `04-projects/<project>/` (check for an existing patterns/surface-design folder first, matching whatever convention that project already uses) — nothing generated here lives only in the conversation, per the "No Context Lost" rule.

## Error handling

- No native tiling flag exists anywhere in the connected surface → never state or imply a generated tile is mechanically guaranteed seamless; always frame it as prompt-engineered and visually verified.
- Verification proof (step 5) shows visible seams → say so plainly and revise the prompt (stronger edge-continuity language, simpler/larger motif, remove any lighting/shadow cues) rather than shipping the seamed tile as if it passed.
- Bordered or vignetted output (a common default reading of "pattern" prompts) → treat as a failed generation requiring a prompt revision ("no border, no frame, no vignette, full-bleed edge to edge"), not something to upscale or ship as-is.
- Ambiguous repeat type (user says "tile" with no straight/half-drop/brick specified) → default to straight and state the default explicitly, per Data flow step 1.
- `style_type`/`negative_prompt` set by mistake on a non-custom-model call → they will be silently ignored by the API, not error; the skill should not rely on them for register or exclusions on the default v4 path, and should say so if the user expects them to have an effect.
- `generate_images_bulk` partial failures → report per-prompt outcomes honestly rather than treating the whole batch as done if some prompts failed.

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts:
1. "Design a seamless floral wallpaper pattern in blush and sage" — base single-tile generation, checks seamless/edge-continuity framing lands in the prompt, no border/vignette, straight-repeat default stated, verification step run.
2. "Give me 5 colorway variations of this seamless geometric pattern for packaging" — checks `generate_images_bulk` is used (batched, not looped) and each colorway keeps the same motif/repeat structure.
3. "Make this fabric texture into a tileable swatch, but the last one had visible seams" — checks the skill treats a failed verification honestly (revises the prompt with stronger edge-continuity/no-shadow language) rather than re-shipping the same seamed result or silently upscaling it.

## Out of scope (this spec)

- Engineering-repeat production output — half-drop/brick repeats measured to exact print-mill tolerances, vector separations, or Pantone-matched production files. This skill produces raster tile art with repeat-type stated in the prompt, not print-production-ready deliverables.
- Any pixel-level tile/offset/wrap post-processing tool — not available in the connected MCP surface; verification is visual/best-effort only (`references/tiling-verification.md`).
- Proactive/automatic triggering from `brand-identity-sheet`, `moodboard-generator`, or any other sibling skill — explicit-request trigger only, matching the `collections-management` precedent.

## Assumptions

- This skill is structurally a prompt-composition skill (like `logo-prompting`/`moodboard-generator`), not an orchestration workflow (like `collections-management`/`custom-model-training`) — seamless-tile generation is one prompting problem with a visual-verification loop, not a multi-step sequential-API pipeline.
- No native seamless/tileable capability exists in the connected Ideogram MCP surface (confirmed by direct schema inspection of `generate_image`, `generate_images_bulk`, `remix_image`, `reframe_image`) — this is treated as a hard constraint to surface to the user, not a gap to paper over with confident-sounding prompt language alone.
- `style_type`/`negative_prompt`/`seed` are no-ops on the default v4 generation path (confirmed by direct schema inspection) — the skill relies on prose framing for register and exclusions instead, and does not carry forward the sibling skills' pattern of setting `style_type: "DESIGN"` as if it does something on a non-custom-model call.
- Straight repeat is the default assumed repeat type when the user doesn't specify one, since half-drop/brick repeat correctness can't be mechanically verified without an offset tool this skill doesn't have.
