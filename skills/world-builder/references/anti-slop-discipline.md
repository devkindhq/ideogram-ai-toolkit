# Anti-Slop Discipline for World-Building

A brand-agnostic ban list — unlike `brand-identity-sheet`'s or `character-model-sheet`'s
anti-slop files, this one isn't seeded from any specific brand's avoid-list, because a
world-building pipeline spans many unrelated projects. Run this gate before every
`generate_image` or `generate_images_bulk` call in every step of the pipeline, with-model
and without-model alike.

If the project *does* have its own brand doc with an explicit avoid-list, copy those
specific banned nouns into the prompt's negative-space clause verbatim, same as
`brand-identity-sheet` requires — this file's list is the floor, not a replacement for a
brand-specific one.

## Universal ban list (generic AI-image clichés, not brand-specific)

Unless a specific step deliberately calls for one of these as a named exception, every
prompt in every step must exclude:

- **Glowing orbs / floating light spheres** standing in for "energy," "technology," or
  "magic" with no narrative reason.
- **Neural-network node diagrams / circuit-brain imagery** — the reflexive "this is AI"
  visual, regardless of whether the brand is AI-adjacent.
- **Circuit-board textures** applied to surfaces that have no in-world reason to be
  circuitry (walls, fabric, skin, landscape).
- **Gradient washes** used as a background filler in place of an actual described
  material, sky, or setting.
- **Stock-photo people** — generic, anonymous, professionally-lit "diverse team in an
  office" figures with no connection to the world being built. If the step calls for
  characters, they must be the trained character (with-model steps) or explicitly
  described in-world figures, never stock-photo filler.
- **Glossy mirror-shine** on surfaces that should read as matte, worn, or handmade —
  applies especially to the landmarks/art/artifacts steps, where over-polished CGI shine
  undercuts the "lived-in world" quality those steps are trying to establish.
- **Plastic-toy uncanny valley** — overly smooth, synthetic-looking material rendering
  that reads as a 3D-render placeholder rather than a real material in the world.

## Pre-generation gate — run this before every call

Score the drafted prompt 1-5 on each axis. Anything under 3 on any axis means revise
before generating — regenerating a batch that already spent custom-model or bulk-call
budget is expensive; revising a prompt is not.

| Axis | What you're checking |
|---|---|
| **Ban-list clearance** | Does the prompt avoid every item in the universal ban list above (and any brand-specific avoid-list, quoted verbatim)? |
| **Palette-lock adherence** | Does the prompt quote the locked palette (`palette-lock.md`) with exact hex values and roles, not a paraphrase? |
| **Character-presence correctness** | For a with-model prompt: is the exact character count stated? For a without-model / world-only prompt: is the explicit "no characters, mascots, or people anywhere in frame" clause present? (See `character-batch-discipline.md`.) |
| **In-world specificity** | Are materials, settings, and objects named concretely (a real material, a real architectural type, a real object) rather than left to generic AI-image defaults? |
| **Restraint** | Is the prompt asking for only what this step needs, not stacking unrelated elements from other steps into one image? |

Do not skip this gate because an earlier step in the pipeline already passed it — each
step's prompts are new prompts and need their own pass through this table, even when
reusing the same locked palette and ban list.
