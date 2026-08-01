---
name: concept-grid-generator
description: Generates a single "concept grid" image — a labeled comparison sheet showing several DISTINCT motif/icon concepts as tiles side by side, one locked palette and one locked rendering technique held constant across every tile — and actually renders it via the ideogram MCP tools, not just drafts a prompt. Use whenever the user wants to compare multiple genuinely different icon/motif ideas cheaply before committing to a full board, asks for a "concept grid," "icon exploration grid," "concept comparison sheet," or "let's see a few directions before we build the full sheet," or is choosing between candidate motifs for a mark that will later go into `brand-identity-sheet` or `character-model-sheet`. Distinct from `bulk-image-generation-workflow` (N variations of ONE locked caption/idea — same concept, different pose/prop/angle) and from `moodboard-generator` (a whole-brand loose exploration board with an unlocked, still-open palette) — this skill locks the palette and technique first and varies only the *concept* across tiles, and produces one comparison image, not a batch of N separate images. Trigger this before either full-board skill when there's more than one candidate motif in play; trigger a full-board skill once a single tile wins.
---

# Concept Grid Generator

A concept grid is a single generated image that puts several genuinely distinct
motif/icon concepts side by side as labeled tiles, with the palette and the rendering
technique held identical across every tile. It exists to make choosing between
candidate ideas cheap: one generation, one review pass, instead of running a full
`brand-identity-sheet` or `character-model-sheet` for every candidate and comparing
finished boards after the fact.

## Why this exists — cheap N-distinct-concepts comparison vs. N expensive full boards

A full identity sheet or character sheet is a composition-heavy, multi-panel,
`QUALITY`-rendered generation — it's the right tool once a direction is picked, but
it's the wrong tool for picking between directions. Running three full boards to
compare three candidate motifs triples the cost and the review burden for a decision
that doesn't need six-panel context, it needs to see the marks next to each other.
A concept grid answers "which of these ideas is worth developing" in one generation
by holding everything except the concept itself constant, so the only variable a
reviewer has to judge is the idea.

This is different from varying a locked idea (`bulk-image-generation-workflow`, which
holds `style_description` byte-for-byte identical and varies pose/prop/angle within
one concept) and different from exploring a whole brand loosely (`moodboard-generator`,
which explores nine categories — palette, type, imagery, mood — with the palette
itself still open). Here, the palette and the rendering technique are the two things
locked *before* drafting a single tile, and what varies is the concept — the actual
form/idea each tile represents.

## The two-stage handoff — Set A (this skill) → Set B (lock-in)

Treat this skill as Set A and `brand-identity-sheet` / `character-model-sheet` as
Set B. Never skip straight to a full board for every candidate concept — that's the
expensive path this skill exists to avoid. And never treat the grid image itself as a
finished deliverable — a tile in a comparison sheet is a rough, small, same-technique
sketch next to five others; it hasn't been rendered at the scale, polish, or
system-context a shipped identity sheet or character sheet needs.

The handoff, in order:

1. Run this skill first. Produce the grid, review it with whoever's deciding, and get
   an explicit pick — one winning concept, named.
2. Route only the winner forward, as a separate follow-up generation, into
   `brand-identity-sheet` (if the winner is a brand mark/icon direction) or
   `character-model-sheet` (if the winner is a character/mascot direction). That skill
   develops the winning concept into its own full panel set, typically with the same
   locked palette carried forward and the rendering technique either kept or
   deliberately upgraded now that only one concept needs full development.
3. Do not resurrect the losing tiles as separate full-board generations "just in case"
   — the grid already did the comparison work; if a second concept genuinely needs a
   fair look, run a fresh, smaller grid rather than promoting every candidate.

## Workflow

### 1. Lock the palette and the rendering technique before drafting concepts

Decide these two things first, and hold them identical across every tile in the grid:

- **Palette** — the exact hex values in play (pull from an existing `brand.md` if one
  exists, or from whatever the user states directly). A concept grid is not the place
  to explore palette — that's `moodboard-generator`'s job. If no palette exists yet,
  either ask for one or run `moodboard-generator` first.
- **Rendering technique** — one named, concrete technique (e.g. "flat cut-paper
  negative-space cutout," "single-line continuous contour," "matte 3D claymation
  render," "risograph two-tone halftone"). This is what keeps the grid reading as one
  coherent comparison sheet rather than six unrelated icon styles competing for
  attention — the reviewer should only be reacting to the *idea* in each tile, not to
  inconsistent rendering pulling their eye around.

### 2. Write each concept as one tight, abstract, single-mark sentence

Same discipline as `logo-prompting`'s "honest specificity, not adjective soup" — each
concept description needs a concrete visual referent an image model can actually
render, not an adjective. Pick concepts genuinely distinct in *form*, not
palette-swaps or minor pose variations of one idea; if two candidate concepts would
produce near-identical silhouettes, that's one concept, not two, and one of them
should be dropped or reworked before drafting the grid prompt. Aim for one clear noun
or compact visual idea per concept — "a folded paper crane mid-unfold" not "something
that evokes transformation and lightness."

### 3. Write the grid prompt

Structure, in order:

1. Frame it explicitly as an **icon exploration grid / concept comparison sheet** —
   naming it this way (not "moodboard," not "icon set") is what keeps Ideogram
   rendering distinct, individually-labeled tiles instead of one blended composition.
2. State the grid dimensions (e.g. "a 2x3 grid of six square tiles," "a 3x2 grid of
   six tiles") so the tile count and layout are unambiguous.
3. List each concept as its own captioned clause, in reading order, each naming its
   own single-mark idea from step 2 plus a short label caption under the tile.
4. State the palette once, as the shared palette across every tile — never restate it
   per-tile, since restating invites drift.
5. State the rendering technique once, as the shared technique across every tile, for
   the same reason.
6. Apply the anti-slop gate (step 3.5 below) and fold its findings into the prompt.
7. Note clean, consistent tile composition — even tile spacing, a shared background
   treatment behind every tile, thin dividing lines or a grid frame, and a consistent
   caption style/position under each tile — this is what keeps the sheet reading as
   one deliberate comparison tool rather than a collage.

Keep it one paragraph, same as the sibling board skills.

### 3.5. Run the anti-slop pre-generation gate

Reuse `brand-identity-sheet`'s anti-slop gate exactly — see
`references/anti-slop-discipline.md` in this skill, which is that file reused
unmodified. Score the drafted prompt before calling `generate_image`. A concept grid
has N times the surface area for a banned cliché that a single mark does — six tiles
means six independent chances for the model to reach for a neural-network node, a
glowing orb, a headset icon, or a gradient wash instead of the brand's actual motif
vocabulary, so check **each tile individually** against the universal-ban list and the
brand's own avoid-list, not just the prompt as a whole. Anything scoring under 3 on
any axis for any tile means revise that tile's concept clause before generating.

### 4. Generate it

Call `mcp__ideogram__generate_image` with the paragraph prompt. Use `style_type:
"DESIGN"` and `aspect_ratio: "1x1"` or `"4x5"` depending on tile count/layout — a grid
of square tiles reads cleanest as a square canvas, a taller grid (more rows than
columns) reads better as `4x5`. Use `rendering_speed: "QUALITY"` — captions and tile
dividers are text/composition-heavy and need to stay legible.

### 5. Save the artifact and route the winner forward

Save the prompt, the `request_id`, and the permalink to the project's branding folder
(check for an existing `logo-explorations/` or `branding/` folder first and match it),
clearly labeled as a Set-A / comparison artifact — e.g.
`concept-grid-<label>.md` — not as a finished deliverable. Record which concept was
picked and why. Once a winner is named, hand it off as a separate follow-up
generation to `brand-identity-sheet` or `character-model-sheet` per the two-stage
handoff above — this skill's job ends at the pick, not at developing the winner
further.

## Reference files

- `references/anti-slop-discipline.md` — `brand-identity-sheet`'s anti-slop gate,
  reused unmodified, with a note on why a grid needs the per-tile check.
- `examples/product-icon-concept-grid.md` — a fully worked, fully fictional example:
  a locked palette, a locked rendering technique, six distinct concepts, the full
  grid prompt, and a closing handoff note naming the winning concept.
