---
name: social-marketing-asset-generator
description: Produces a finished social/marketing image asset — a social graphic, banner, or display ad — with its headline, subhead, CTA, and brand mark baked directly into the pixels by Ideogram, not composited afterward. Use when the user asks something like "make me an Instagram ad for X," "banner ad for our launch," "social graphic announcing Y," or "give me this in Story + Feed + LinkedIn sizes." Triggers only on explicit request for finished ad/banner/social-graphic creative — it is not auto-wired from `brand-identity-sheet`, `logo-prompting`, `moodboard-generator`, or `character-model-sheet`, and does not fire just because one of those skills ran. It is a sibling to, not a replacement for, those four skills: they establish or explore brand truth (a moodboard, a locked identity system, a single mark, a character sheet); this skill spends an already-decided brand truth and literal copy on one rendered creative asset.
---

# Social Marketing Asset Generator

This skill's differentiation angle: baked-in typography via structured JSON captioning,
not the generate-text-free-then-composite-in-HTML/CSS pattern every confirmed competitor
in this space uses. Ideogram renders the scene and the headline/subhead/CTA typography
together in one call, leaning into its specific strength — reliable in-image text
rendering with full control over placement, size, weight, and color. See
`references/typography-baking-discipline.md` for the full writeup of why this beats the
composite-afterward approach, and what it costs (garbled-text risk, caught and regenerated
against rather than accepted).

This is a prompt-composition skill — same family as `logo-prompting`, `brand-identity-sheet`,
and `moodboard-generator` — not an MCP-orchestration skill. Its center of gravity is
composing a legible, on-brand, on-copy JSON caption; the MCP calls it chains
(`generate_image` → optional `generate_images_bulk` fan-out → optional `edit_image`
touch-up) exist to serve that composition, not the other way around.

It is not a copywriting or ad-strategy skill. It accepts literal copy — headline, subhead,
CTA, offer/price if any — as intake and renders it. It never runs AIDA/PAS strategy
frameworks, writes ad copy unprompted, or invents a claim, price, or offer that wasn't
supplied. A rendered image with a fabricated discount figure or an unapproved claim baked
into the pixels is a real failure this skill guards against, not a nice-to-have.

## Before you start

- `references/composition-spec-format.md` — read before composing any JSON caption. It
  names the six ad-creative element roles and gives a full worked example.
- `references/anti-slop-discipline.md` — read before running the pre-generation gate. It
  lists the universal and ad-specific bans the gate scores the caption against.
- `references/platform-dimension-map.md` — read before resolving any placement's
  `aspect_ratio`. It maps platform-native pixel specs to Ideogram's supported enum and
  classifies the fit.
- `references/typography-baking-discipline.md` — read before the legibility review or any
  touch-up request. It has the legibility rules, the one-message-hierarchy rule, and the
  `edit_image`-vs-regenerate decision table.

## Workflow

### 1. Intake

Gather brand truth: the project's existing `brand.md` if one exists; otherwise the brand
name, colors, and a product/offer image if relevant. Gather the literal copy strings —
headline, subhead, CTA, and offer/price if any — and the target placement(s).

If literal copy isn't supplied and the ask is vague ("make an ad for our sale"), ask for
the exact strings before drafting anything. Never invent a claim, price, or offer detail —
this is the one input worth stopping for, since a fabricated number baked into pixels is a
real failure, not a stylistic shortcut.

### 2. Resolve platform dimensions

For each requested placement, look up its row in `references/platform-dimension-map.md`.

- **Exact** fit → proceed silently.
- **Near-fit** → proceed, but mention the approximation in the final save-what-you-made
  summary (Step 8).
- **Severe mismatch** (currently only the Leaderboard row) or any placement not in the
  table with no close ratio match → tell the user up front which placement(s) will be an
  approximation — or, for a severe mismatch, recommend handling that placement outside
  this skill — before generating anything.

### 3. Compose the JSON caption

Build the caption per `references/composition-spec-format.md`'s six element roles:
background/scene, product/subject, headline, subhead/tagline, CTA, brand-mark. Place each
text element's `bbox` inside the resolved platform's safe zone from
`platform-dimension-map.md`. Apply the legibility rules from
`references/typography-baking-discipline.md` while drafting, not after: max words per
line, text size relative to canvas, contrast-checked color pairing, and the
one-message-hierarchy rule (headline dominant, subhead secondary, CTA tertiary).

### 4. Anti-slop gate

Score the drafted caption against `references/anti-slop-discipline.md`'s four-axis
pre-generation gate — cliché avoidance, message hierarchy, copy fidelity, safe-zone
placement — before generating. Any axis scoring under 3 means revise the caption now, not
after generating. Only send to `generate_image` once all four axes score 3 or higher.

### 5. Generate

**Single placement**: call `mcp__ideogram__generate_image` with the composed prompt (the
JSON caption, passed as the `prompt` string), the resolved `aspect_ratio` (and
`resolution` only where `platform-dimension-map.md` lists an exact pixel match for that
row), `style_type: "DESIGN"`, `rendering_speed: "QUALITY"`.

**Multiple placements from one concept**: call `mcp__ideogram__generate_images_bulk` with
one prompt per resolved platform variant — same underlying concept, text-element `bbox`s
adjusted per platform's safe zones. If bulk generation errors or is unavailable (it's a
Pro-only feature), fall back to sequential `generate_image` calls per platform and tell
the user that's what happened and why.

### 6. Legibility review

Inspect the generated image for garbled or illegible rendered text — Ideogram's known
failure mode on dense in-image copy, named in `references/anti-slop-discipline.md`'s
ad-specific bans. If the text is garbled, regenerate from the JSON caption with adjusted
text-element sizing/spacing (larger `bbox`, fewer words per line, per
`typography-baking-discipline.md`'s legibility rules) rather than accepting a broken
render.

### 7. Touch-up loop (optional)

For a localized fix, follow the `edit_image`-vs-regenerate decision table in
`references/typography-baking-discipline.md`:

- `mcp__ideogram__edit_image` for a genuinely localized change — recolor a CTA button,
  swap a logo mark, fix one word.
- A fresh `generate_image` call from a corrected JSON caption for anything bigger — a full
  headline, layout, or placement change.

State explicitly which route is being taken and why, so the user isn't surprised by a
freshly regenerated asset when they asked for a quick touch-up.

### 8. Save

Write the final JSON caption(s), prompt(s), the platform/dimension mapping actually used
(including any disclosed near-fit approximations or severe-mismatch recommendations from
Step 2), and the resulting image URL(s)/`response_id`(s) to the project's existing output
location. Check for an existing `logo-explorations/` or `branding/` folder first and match
whatever convention `brand-identity-sheet`, `logo-prompting`, and `moodboard-generator`
already use for that project — same pattern `moodboard-generator/SKILL.md` follows. Per
the toolkit's "No Context Lost" habit, nothing generated in this skill should live only in
the conversation.

## Error handling

- **No literal copy supplied and the ask is vague** — ask for exact headline/CTA/offer
  strings before drafting; never fabricate a claim, price, or offer detail.
- **Requested placement has no close `aspect_ratio` match** — tell the user explicitly
  which placement(s) will be a near-fit approximation before generating (or, for a severe
  mismatch, recommend handling that placement outside this skill). Don't silently ship a
  wrong-ratio asset as if it matched spec.
- **Garbled or illegible rendered text** — treat as a failed render, not an acceptable
  result. Regenerate with adjusted text-element `bbox`/sizing rather than delivering it.
- **`generate_images_bulk` unavailable** (non-Pro account or a tool error) — fall back to
  sequential `generate_image` calls per platform and tell the user that's what happened,
  rather than silently failing the whole request.
- **`edit_image` requested for a change bigger than a localized touch-up** — don't force
  it through `edit_image`. Explain that a full copy/layout change goes through a fresh
  `generate_image` call from a corrected caption instead, per its own strict
  single-image-edit contract.
- **Existing brand logo must appear pixel-exact and unaltered** — flag that Ideogram
  generation can't guarantee pixel-fidelity reproduction of an existing logo asset;
  recommend the user composite that exact logo file in afterward with their own tooling,
  rather than claiming to bake in a logo it can only approximate.

## Reference files

- `references/composition-spec-format.md` — the ad-creative element vocabulary (six
  element roles) and a full worked example.
- `references/anti-slop-discipline.md` — universal and ad-specific bans, plus the
  four-axis pre-generation gate.
- `references/platform-dimension-map.md` — the placement table (Instagram, Facebook,
  LinkedIn, X, YouTube, Pinterest, generic display) and the Fit classification
  (Exact/Near-fit/Mismatch/Severe mismatch).
- `references/typography-baking-discipline.md` — legibility rules, the
  one-message-hierarchy rule, and the `edit_image`-vs-regenerate decision table.
- `examples/` — worked ad-creative jobs go here as they happen; empty for now.
