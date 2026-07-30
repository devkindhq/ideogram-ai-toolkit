# Sheet Anatomy — Named Panel Types

Pick from these when assembling a character model sheet prompt. Not every sheet needs
every panel — pick the ones that earn their place for the character at hand, and add
new panel types here when a real job establishes one worth reusing.

## Header / overview block
Small text panels at the top of the sheet: a bold character-name headline, a
"[Name] — Character Model Sheet" sub-headline, and 3-4 compact info boxes covering
character overview (name/alias/role/archetype), personality & traits, wardrobe/
accessories, and color palette. This is the "spec sheet" framing that keeps the whole
board reading as a production document, not fan art.

## Full body turnaround
Front, 3/4, side, and back views of the character in the same neutral pose, each
labeled (FRONT / 3-4 VIEW / SIDE / BACK). Name the camera angle precisely rather than
leaving it to "turnaround" alone — orthographic phrasing reads more reliably to the
image model than the genre name:
- Front: "front view, facing directly toward camera"
- 3/4: "three-quarter view, character rotated roughly 45 degrees"
- Side: "profile view, 90 degree turn, facing left or right"
- Back: "rear view, facing directly away from camera"

Run a height-scale ruler alongside the views (inches and/or cm, with tick marks) to
lock proportions — this is what separates a production turnaround from a loose
four-pose illustration.

## Head & detail sheet
Close-up panels isolating the parts of the character that carry the most design
information at small scale:
- **Front face** close-up — full facial expression and detail.
- **Signature head-worn/hair detail** — headphones, a hat, ears, horns, whatever the
  character's defining head accessory is, from whatever angle best shows its
  construction.
- **Hand/gesture or held-prop detail** — a signature pose (a wave, a salute) or a prop
  the character carries, rendered at a scale that shows finger/prop construction
  clearly.

## Expression sheet (optional add-on to head & detail)
A row of 3-6 small head-only panels showing the same face in different expressions
(neutral, happy, surprised, angry, etc.), each labeled. Only include when the character
is expression-driven (a mascot, an animated character with a wide emotional range) — it
adds real production value for animators but isn't needed for a static-personality
character like a product mascot with one default expression. When included, hold face
proportions, eye size, and color identical to the front-face panel in the head & detail
sheet — this panel exists to test expression range, not to re-audition the face design.

## Accessories & interior/technical diagram
Only include if the character has a mechanical, robotic, or gadget-driven identity: an
exploded or cross-section diagram of internal components or worn accessories, with thin
leader lines connecting each part to a small labeled caption (e.g. "Emission core &
symbol projection mechanism," "Wireless receiver unit"). Skip entirely for organic/
human characters unless they carry a single notable prop worth diagramming on its own
(a weapon, a gadget) — in that case, diagram just the prop, not an "interior" the
character doesn't have.

## Color palette bar
Primary/secondary/accent swatches as small labeled rectangles or a stacked bar, usually
docked near the header block. State each swatch's role, not just its hex — "primary:
bell body orange," not an unweighted color chip.

## Adding a new panel type
If a real job produces a panel worth reusing across characters (not just this one
character's specific execution of it), add it here with the same "what it's for, when
to include it" framing — the point is to grow this into a shared vocabulary, the same
way `brand-identity-sheet`'s and `logo-prompting`'s panel/guardrail vocabularies grew
from real jobs.
