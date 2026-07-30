# Anti-Slop Discipline for UI Mockups

A marketing-UI mockup has its own failure mode, distinct from an identity sheet's
icon-grid genericness or a moodboard's mood-drift: the generic "fake AI dashboard"
look — charts with no legible data, sidebars that imply navigation going nowhere, and
screens that read as "UI-shaped" rather than as an actual product. This file is the
gate you run **before** calling `generate_image`, not a style suggestion to eyeball
after.

This targets this skill's own failure mode, not the brand-panel slop
`brand-identity-sheet/references/anti-slop-discipline.md` and
`moodboard-generator/references/anti-slop-discipline.md` guard against — a mockup can
pass every brand-color and typography check and still be slop if the dashboard it
shows has no real content behind it.

## Universal bans for UI mockups

- **Meaningless charts** — a bar/line chart with no legible axis labels, no
  plausible data trend, or values that don't relate to any real metric named in the
  brief. Every chart in the prompt must be described with a real metric name and a
  plausible trend direction (e.g. "a line chart labeled 'Weekly Active Users' trending
  upward from roughly 200 to 850"), never "a chart" or "some graphs."
- **Decorative sidebar icons implying navigation that goes nowhere** — a sidebar
  with 6+ icon-only nav items with no labels and no coherent relationship to the
  product's actual feature set. Name real nav-item labels tied to what the product
  does.
- **Illegible or garbled micro-text used as texture** — small blocks of text
  rendered purely as a visual filler/texture element rather than real copy, which is
  exactly the failure mode Ideogram's text-rendering strength is supposed to avoid.
  Any text block in the prompt must either be real, quoted copy, or be omitted from
  the composition entirely — never "text-shaped scribbles" implied by a vague
  description.
- **Stock-SaaS purple-gradient hero backgrounds** — the same purple-to-pink/cyan-to-
  magenta gradient wash `brand-identity-sheet`'s own anti-slop file bans, applied here
  specifically to hero and dashboard backgrounds, unless the brand's own locked
  palette explicitly calls for it.
- **AI-glow product screen with no actual content** — a device or dashboard screen
  rendered with a vague "glowing UI" impression instead of describable, specific
  on-screen elements (real stat card labels, a real chart, real nav items). If the
  prompt can't name what's actually on the screen, the screen isn't specified yet —
  go back and specify it before generating.

## Real-copy requirement gate

A rule specific to UI mockups, not present in the sibling skills' anti-slop files:
before drafting the prompt, confirm real headline/subhead/CTA/nav copy was gathered in
workflow step 1 (per `SKILL.md`). If any text element in the planned composition still
has no real copy assigned, that is a slop risk on its own — Ideogram will either
render garbled placeholder-shaped text or fall back to the model's own generic filler
— not just a content-completeness issue. Treat a missing-copy element as blocking,
the same severity as a banned visual cliché above.

## Pre-generation gate — run this before calling `generate_image`

Score the drafted paragraph prompt 1-5 on each axis below. Anything under 3 on any
axis means revise the prompt before generating — a second draft is cheap, a
regenerated image is not.

| Axis | What you're checking |
|---|---|
| **Copy completeness** | Does every text-bearing element (headline, subhead, CTA, nav items, chart labels) have real, specific copy assigned — not a placeholder, not left vague? |
| **Chart/data specificity** | Does every chart or stat element name a real metric and a plausible trend or value, rather than "a chart" or "some data"? |
| **Universal-ban sweep** | Read the drafted prompt sentence by sentence against the "Universal bans for UI mockups" list above. Is every banned pattern absent, or present only as a brand-justified, explicitly-named exception? |
| **Screen content legibility** | Could every described on-screen element actually be read/understood in the finished image, or does the description leave "what's actually on this screen" vague enough that Ideogram will default to decorative filler? |
| **Layer completeness** | Does the drafted prompt name every compositional layer `mockup-anatomy.md` says this format requires (per the per-format layer table), with nothing silently dropped? |

Do not skip this because the brief-gathering step (workflow step 1) already collected
real copy — copy completeness and the other four axes are different failure modes,
and a prompt can nail the former while still defaulting to a meaningless chart or a
decorative dead-end sidebar in the latter.
