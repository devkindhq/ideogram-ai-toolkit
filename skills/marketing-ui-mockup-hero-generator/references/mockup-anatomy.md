# Mockup Anatomy — Named Format Types and Compositional Layers

This is the fixed vocabulary of this skill: three format types and the compositional
layers each is built from. Read this **before** drafting a prompt (workflow step 2 in
`SKILL.md`), not reverse-engineered only after generation — assigning layers is a
planning step, not a description written after the fact. The three formats, in the
order this file covers them: landing-page hero shot, device-in-context product
mockup, dashboard screenshot.

## Landing-page hero shot

A full-bleed marketing hero image with headline, subheadline, and CTA button copy
baked in via Ideogram's text rendering, typically showing the product in a
browser/device frame or as a standalone visual. This is the format for "the top of
the landing page" — the shot a visitor sees before scrolling.

Its compositional layers, in reading order:

- **Background/Environment** — the base surface the hero sits on (a solid brand
  color, a soft gradient, an environmental scene). Must be a deliberate, named choice
  — never left as "whatever Ideogram defaults to."
- **Nav Bar** (optional, only if the product visual includes browser/app chrome) —
  real nav-item labels, not "Home / About / Contact" filler, pulled from the
  project's actual site structure if known.
- **Hero Copy Zone** — the real headline and subheadline text, quoted verbatim in
  the prompt.
- **CTA Placement** — the real button label, plus its position relative to the copy
  zone (directly below, inline beside it).
- **Device Frame/Chrome** (optional) — a browser window frame or bare device bezel
  containing the product visual, if the hero shows the product "inside" a frame
  rather than as a standalone graphic.
- **Product Visual** — the actual UI/product being shown, described with real
  feature specifics, not "a modern dashboard."

## Device-in-context product mockup

A product's UI shown inside a physical device shell (laptop, phone, tablet, browser
chrome) placed in an environmental or lifestyle context (desk surface, hand, studio
backdrop). This is the format for pitch-deck slides and "see it in the real world"
shots — photorealism of the environment is doing most of the selling.

Its compositional layers, in reading order:

- **Background/Environment** — the physical setting (a desk surface with specific
  real objects, a hand holding the device, a studio backdrop with named lighting).
  This layer carries more weight here than in the hero format, since photorealism of
  the environment is most of what sells the shot.
- **Device Frame/Chrome** — the specific device named explicitly (a MacBook
  Pro-style laptop, an iPhone-style handset, a tablet), described by real physical
  details (bezel color, screen reflection behavior), not "a laptop."
- **Supporting UI Chrome** — the browser/app chrome visible around the product
  screen inside the device (tab bar, status bar), kept minimal and legible.
- **Product Visual** — the actual product screen shown on the device, described
  with real feature specifics.

This format has **no Nav Bar layer as a top-level element** — any nav bar exists
only as part of the Product Visual layer's on-screen content, not as a separate
compositional element the way it is in the hero format.

## Dashboard screenshot

A single stylized app/dashboard screen (nav, cards, charts, sidebar) rendered to
look polished and "real" for App Store listings, pitch decks, or marketing pages —
not a literal render of any actual live codebase. This is the format for "show the
product working" without a device frame or environmental context around it.

Its compositional layers, in reading order:

- **Nav Bar / Sidebar** — real, specific nav-item labels (not "Item 1, Item 2"),
  consistent with the product's actual feature set if known.
- **Supporting UI Chrome** — stat cards, charts, tables — each one named with real,
  specific data semantics (e.g. "a line chart labeled 'Weekly Active Users' trending
  upward, with a visible Y-axis scale," not "a chart").
- **Background/Environment** — the screen's own background treatment (app chrome
  color, canvas color).

This format has **no separate Device Frame/Chrome or environmental-context layer**
— the screen fills the frame, unlike the device-in-context mockup. It also has **no
CTA Placement layer** as defined for the hero format — a dashboard may contain
buttons, but they're part of Supporting UI Chrome, not a singular hero CTA.

## Per-format layer table

| Layer | Landing-page hero | Device-in-context mockup | Dashboard screenshot |
|---|---|---|---|
| Background/Environment | Yes | Yes | Yes |
| Nav Bar | Yes (optional — only when the product visual includes browser/app chrome) | No — see device-in-context section: any nav exists only inside the Product Visual layer, never as a separate compositional element | Yes — called "Nav Bar / Sidebar" in this format, per the dashboard section above |
| Hero Copy Zone | Yes | No — see device-in-context section: no headline/subhead layer, the product screen itself is the subject | No — see dashboard section: the screen fills the frame, there is no hero-copy layer |
| CTA Placement | Yes | No — see device-in-context section: no singular hero CTA, only whatever CTA-like buttons exist inside the Product Visual's on-screen content | No — see dashboard section: any buttons are part of Supporting UI Chrome, not a singular hero CTA |
| Device Frame/Chrome | Yes (optional — only if the product visual sits inside a frame rather than standing alone) | Yes | No — see dashboard section: the screen fills the frame, no separate device shell or environmental context |
| Supporting UI Chrome | No — the hero format's on-screen browser detail is covered by Device Frame/Chrome, not a separate Supporting UI Chrome layer | Yes (minimal chrome inside the device, e.g. a browser tab bar) | Yes (stat cards, charts, tables) |
