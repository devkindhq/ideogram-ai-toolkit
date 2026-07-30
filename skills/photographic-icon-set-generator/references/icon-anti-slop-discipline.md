# Anti-Slop / On-Model Discipline for Photographic Icon Sets

A photographic icon set has a failure mode that single-icon sets don't have to worry about as
acutely: the *same icon subject* has to read as the same photographic material, camera angle,
and ground treatment across the entire batch. Ideogram drifting shadows, backgrounds, or
material finish between icons is the single most common way a cohesive icon set comes out
unusable — a set that isn't consistent isn't locked, and a set that isn't locked can't ship as
a visual system. This file is the gate you run **before** calling `generate_images_bulk`, not
a style suggestion to eyeball after.

Because `negative_prompt` is ignored on the default v4 pipeline (per the Global Constraints),
every ban below is phrased as a positive instruction to embed directly in the prompt text,
never as a negative-prompt string. A restatement of the locked material finish or shadow
treatment in your prompt is instruction; a deleted-out negative entry is decoration.

## Glossy-app-icon-store clichés

The most common way an icon batch drifts even though every subject block is individually
well-described: Ideogram reaches for its default app-icon-store aesthetic regardless of the
requested material or ground treatment. Three specific defaults slip in repeatedly:

- **Floating drop shadows detached from the subject** — the model tends to place a soft,
  rounded drop shadow hovering below the icon (a 1990s web shadow). Instead of letting this
  happen, state explicitly in every prompt: "soft grounded contact shadow, not a floating
  drop shadow" — reference the exact shadow treatment from the locked style block. Name the
  shape and angle of contact it makes with the ground.

- **Gradient bubble or circle backgrounds behind the subject** — the model defaults to a
  semi-transparent circular gradient behind the icon, or a subtle bubble highlight. This
  breaks visual rhythm against the locked plain/staged ground. Instead, state explicitly in
  every prompt: "against the locked plain [ground treatment]" or "no background gradient or
  bubble highlight" — restate the exact background from the style block by name.

- **Bevel-and-emboss glossy-plastic finish applied regardless of material request** — even
  when you've specified matte clay, waxy painted gouache, or brushed metal, the model glossy-plastic
  sheen shows up as a default. Instead of hoping the material request sticks, restate it
  explicitly in every prompt: "matte finish, no glossy plastic sheen" or "waxy paint, not
  plastic" or "brushed metal with no gloss" — repeat the exact surface finish from the style
  block because unstated finish defaults to glossy plastic.

## Mini scene / illustration drift

An icon that adds a second subject, an implied environment, or a narrative moment stops
reading as an icon in a set and breaks visual rhythm against the other N-1 icons that stayed
single-subject. Examples: a "home" icon that becomes a small illustrated house-in-a-yard scene
instead of a single centered icon-object; a "phone" icon that includes a hand holding it; a
"calendar" icon that includes a pen and desk implied behind it.

**The fix:** State explicitly in the composition block of every prompt that the subject is a
single, isolated icon-object — one clear form, no secondary props, no implied environment
beyond the locked ground treatment. Apply this constraint identically to every icon in the
set regardless of how "busy" a particular subject concept might otherwise invite. Use phrasing
like: "single icon-object, centered, isolated against the ground; no secondary props, no
implied scene or environment."

## Camera-angle / material drift between icons

This is the failure mode specific to *sets* (not present in a single-icon anti-slop gate).
Even with a well-written style block and consistent prompt language, a batch can still drift
after generation — e.g., icon #4 rendered from a slightly different angle than icon #1,
or icon #7 with a noticeably glossier material finish than the rest. This usually happens
because the subject block for that one icon implicitly pulled the render away from the locked
recipe (e.g., a naturally tall subject like a standing figure invites a different framing
than a naturally wide subject like a rectangular landscape).

Preventing this drift entirely at prompt time is not possible — the model still generates each
`prompts[]` entry independently, without knowledge of the prior icons. **This is specifically
what workflow step 6 (review as a set) exists to catch, and workflow step 7 (correction via
`edit_image`/`remix_image` against anchor icons) exists to fix.** A well-written batch-level
anti-slop gate can reduce the odds of this happening, but it is not a full guarantee.

## Pre-generation gate — run this before calling `generate_images_bulk`

Score the drafted `prompts[]` batch 1-5 on each axis below. Anything under 3 on any
axis means revise the prompts before generating — a second draft is cheap, a
regenerated batch is not.

| Axis | What you're checking |
|---|---|
| **Style-block fidelity** | Is the style block copy-pasted byte-identical into every `prompts[]` entry, not re-typed or paraphrased per icon? |
| **Subject-block restraint** | Does every subject block name one clear icon-object concretely, with no second subject, implied scene, or narrative moment? |
| **Ground/shadow explicitness** | Does every prompt name the exact grounded shadow and background treatment from the style block, rather than leaving "background" or "shadow" unstated? |
| **Material-finish explicitness** | Does every prompt restate the actual requested surface finish (matte, waxy, painted, brushed) rather than leaving finish to the model's glossy-plastic default? |
| **Set-level restraint check** | Scanning all N drafted prompts together (not one at a time): does any single icon's subject block risk pulling that one icon toward a different camera framing or material read than the rest? |

Do not skip this because the style-lock recipe (SKILL.md workflow step 3) already named
materials and shadows correctly — prompt fidelity and on-model consistency are two different
failure modes, and a batch can nail the former while still drifting one icon's finish or
angle between generation and the final set.
