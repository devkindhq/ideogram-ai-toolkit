# Anti-Slop Discipline for Seamless Patterns & Textures

Adapted from the sibling skills' anti-slop files (`moodboard-generator`,
`brand-identity-sheet`, `logo-prompting`), which are themselves adapted from
Hallmark's slop-test discipline. But pattern/texture generation has a failure mode
none of those sibling skills share: this skill's output has to actually tile, not
just look good as a single image. A border/vignette or an obvious copy-paste-rotation
tell doesn't just read as generic AI slop the way a cliche gradient or stock-photo
handshake does in a moodboard panel — it mechanically breaks the tiled read. A
bordered "pattern swatch" cannot repeat edge to edge, and a visibly copy-pasted motif
grid gives away the seam the moment it's tiled. That's why two of the axes below are
flagged as must-revise, not just noted: on this skill, some slop is a hard failure,
not a style concern.

## Bans

Unless the user has explicitly asked for one of these as a deliberate, named
exception, no drafted prompt should produce:

- **A hard border or vignette framing the tile** — the single most common failure
  mode for this skill. A framed "pattern swatch" image cannot tile, because the frame
  itself is not part of the repeating field; it's a fixed edge that breaks continuity
  the instant the tile repeats.
- **An obvious repeated-rotation tell** — the exact same motif instance copy-pasted
  at a visible grid interval with zero variation between instances. This reads as an
  AI-pattern giveaway rather than an organic repeating design, and undermines the
  "infinite continuous field" read `tile-prompt-recipe.md` requires.
- **Muddy default "boho" florals or "modern geometric" filler with no stated palette
  behind it** — generic filler that ignores whatever colorway or motif detail the
  user actually gave, producing a pattern that could belong to any brief instead of
  this one.
- **Gradient-mesh backgrounds standing in for a real texture** — a soft color blend
  used as a shortcut for material detail that was supposed to be photographic or
  tactile, when the brief called for a REALISTIC-register material texture.
- **Halftone-dot spam as generic "texture" filler** — a default AI-image texture tell
  with no relationship to the requested material, dropped in as filler when the
  prompt doesn't specify an actual surface.
- **Directional single-source lighting or cast shadows** — breaks the flat tiled read
  per `tile-prompt-recipe.md`'s framing rule, since a single light source implies one
  fixed viewing frame, not an infinite repeating field; the shadow repeats with the
  tile instead of reading as one continuous light source across the whole field.

## Pre-generation gate

Score the drafted prompt 1-5 on each axis below before calling `generate_image`.
**Border/vignette absence** and **Rotation-variety** are must-revise axes: scoring
under 3 on either one means the prompt must be revised before generating, not just
noted, because both directly break tileability rather than just weakening the
aesthetic (per `SKILL.md`'s Workflow step 3). The other three axes call for revision
if they score low too, but they don't block generation the way the first two do.

| Axis | What you're checking |
|---|---|
| **Border/vignette absence** (must-revise if under 3) | Does the prompt explicitly state full-bleed, no frame, no vignette? |
| **Rotation-variety** (must-revise if under 3) | Does the prompt imply organic variation between motif instances rather than one instance mechanically copy-pasted? |
| **Edge-continuity language present** | Does the prompt include the specific edge-crossing/reconnecting language from `tile-prompt-recipe.md` (element 3), not just the word "seamless" alone? |
| **Flat framing** | Does the prompt explicitly rule out directional lighting, cast shadows, and vignette? |
| **Palette/motif specificity** | Does the prompt name the user's actual stated palette/motif rather than defaulting to generic "boho floral" or "modern geometric" filler? |

These five axes map directly onto `tile-prompt-recipe.md`'s five recipe elements:
edge-continuity language and flat framing check recipe elements 3 and 4 by name;
border/vignette absence and palette/motif specificity check the same concerns recipe
elements 4 and 5 (flat framing, DESIGN-vs-REALISTIC register) raise from the
opposite direction — is the border/lighting exclusion actually in the prompt, is the
register-specific palette actually named. Rotation-variety has no single recipe
element counterpart; it's the anti-slop-specific check on motif scale (recipe element
2) actually being followed organically rather than mechanically. Repeat type (recipe
element 1) intentionally has no gate axis of its own here — whether the prompt says
straight, half-drop, or brick isn't a slop concern, it's a factual default-disclosure
question already handled by Workflow step 1, so duplicating it on this gate would be
redundant, not an oversight.

## Why this can't be skipped

There's no mechanical tiling flag anywhere in the connected surface to catch a
bordered or copy-pasted-looking result after the fact — this gate is the only
pre-generation check standing between a drafted prompt and an expensive, un-tileable
render. Skipping it because the brief seemed simple is exactly when the most common
failure (a framed "pattern swatch" look) tends to slip through: a simple brief
produces a simple prompt, and a simple prompt is the one most likely to omit the
full-bleed/no-border language a well-composed one would have included by habit.
