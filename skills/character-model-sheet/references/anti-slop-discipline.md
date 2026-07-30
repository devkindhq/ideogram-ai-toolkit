# Anti-Slop / On-Model Discipline for Character Sheets

A character sheet has a failure mode brand-identity sheets don't have to worry about as
acutely: the *same character* has to read as the same character across four turnaround
views plus a head close-up plus (sometimes) a technical diagram. Ideogram drifting
proportions, colors, or style between panels is the single most common way a character
sheet comes out unusable — a design that isn't consistent isn't locked, and a sheet
that isn't locked can't go to an animator or modeler. This file is the gate you run
**before** calling `generate_image`, not a style suggestion to eyeball after.

## Why consistency drifts even with a good prompt

The most common way a character-sheet prompt still comes out off-model even though
every panel is individually well-described: the writer describes the character once at
the top of the prompt, then re-describes it slightly differently for each panel (a
different color word, a different proportion note) without noticing the drift. Ideogram
has no memory of "the character I drew in the first panel" — every panel is generated
from the words actually present in the prompt, so any inconsistency in the words
becomes an inconsistency in the image.

**The fix is mechanical: describe the character fully once, then put a dedicated
consistency directive near the top of the prompt** restating that identical body
proportions, colors, and design details must hold across every view and panel. Treat
this the same way `brand-identity-sheet` treats quoting a brand's avoid-list verbatim —
it's not decoration, it's the instruction that actually binds.

## Universal bans / risks (apply to every sheet)

- **Off-model drift between turnaround views** — a front view and a back view that
  don't obviously belong to the same character (different proportions, different
  accessory placement, different color saturation). The consistency directive above is
  the primary defense; naming the character's silhouette and exact colors once, in
  full, is the secondary one.
- **Style drift between the turnaround and the detail/close-up panels** — the
  turnaround rendered as clean 3D and the face close-up rendered as flat illustration
  (or vice versa). State the render/material style explicitly and repeat "same style
  across all panels" in the prompt.
- **Dramatic/inconsistent lighting between panels** — a character sheet is a reference
  document, not a mood piece. Use flat, even, shadow-minimal studio lighting stated
  once and implied to apply to every panel ("soft, even studio lighting across all
  panels, no dramatic shadows") rather than letting each panel invent its own lighting
  mood.
- **Unlabeled or mislabeled panels** — every view and detail panel needs its caption
  (FRONT, 3/4 VIEW, SIDE, BACK, FRONT FACE, etc.) actually rendered in the image, not
  left to the viewer to infer from pose alone.
- **Invented text** — don't let the model add a random tagline, a fake logo, or
  placeholder Latin text anywhere on the sheet. Every text-bearing panel in the prompt
  should be explicitly named with its intended content (character name, trait labels,
  height numbers).
- **Generic stock-mascot clichés** — a default "cute AI robot" look (single round eye,
  antenna, primary-color plastic) is fine only if the character brief actually calls
  for it; if the brief has a distinct silhouette or material ask, don't let the model
  fall back to the generic mascot archetype instead.
- **A background that competes with the panels** — keep it a clean white/light-gray
  ground with thin grid lines and thin black panel borders; a busy or thematic
  background (space scene, gradient wash) undermines the "reference document" framing
  that makes the sheet actually usable downstream.

## Pre-generation gate — run this before calling `generate_image`

Score the drafted paragraph prompt 1-5 on each axis below. Anything under 3 on any
axis means revise the prompt before generating — a second draft is cheap, a
regenerated image is not.

| Axis | What you're checking |
|---|---|
| **Character-specificity** | Is the character described once, fully (silhouette, proportions, colors with role, wardrobe/accessories), rather than vaguely re-described per panel? |
| **Consistency-directive presence** | Does the prompt include an explicit clause instructing identical proportions/colors/style across every panel — not just implied by describing the character once? |
| **Style-lock** | Is the render/material style (3D toy-figure, flat vector, painterly concept art, etc.) named once and stated to hold across all panels? |
| **Panel-labeling completeness** | Does every view and detail panel have its caption named in the prompt (FRONT, SIDE, BACK, etc.), not left implicit? |
| **Restraint** | Is every panel earning its place for this character (no diagram/accessory panel for a character with nothing to diagram), and is the background a clean neutral ground rather than a themed scene? |

Do not skip this because the character-truth read (step 1) already named colors and
personality correctly — brief accuracy and on-model consistency are two different
failure modes, and a prompt can nail the former while still drifting the character's
proportions or style between panels in the latter.
