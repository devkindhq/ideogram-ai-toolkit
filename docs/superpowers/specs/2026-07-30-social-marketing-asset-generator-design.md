# Social Marketing Asset Generator Skill — Design

## Why

Gap-research (`05-knowledge/technical/ideogram-tier3-differentiation-research-2026-07-30.md`
in the Devkind vault, Gap 1) covers social/marketing asset generation — this is a Tier 3
skill per the toolkit's gap tiering, not a Tier 1 zero-prior-art gap like
`collections-management`. The gap analysis itself calls this "the most crowded gap":
multiple existing skills already target banner/ad creative. The job here isn't filling an
empty space, it's establishing a specific, defensible differentiation angle inside a
crowded one.

Verified prior art (from the research doc, not re-derived here):
- **`wachawo/claude-skills` `banner-design`** — Gemini-backed (not Ideogram), and
  explicitly instructs the model to generate **text-free** art, then composites headline/
  CTA typography afterward via HTML/CSS and a Chrome-DevTools screenshot export.
- **`coreyhaines31/marketingskills`** — model-agnostic router (Ideogram is one option among
  Gemini/GPT-Image/Flux/Midjourney), generic "Subject + Setting + Style + Lighting +
  Composition + Technical" narrative prompting. No JSON structuring, no anti-slop
  discipline, no compositional-deconstruction methodology.

No confirmed competitor combines an Ideogram-native pipeline with baked-in AI-rendered
typography *and* a structured-JSON-caption / anti-slop / compositional-deconstruction
discipline for this category. The differentiation angle (confirmed, not re-derived): lean
into Ideogram's specific strength — reliable in-image text rendering — and generate
finished assets with typography baked directly into the pixels via structured JSON
captioning, rather than the generate-text-free-then-composite-in-HTML/CSS workaround every
confirmed competitor uses. Combine that with this repo's existing anti-slop and
compositional-deconstruction discipline (`ideogram-prompt`, `logo-prompting`,
`brand-identity-sheet`, `moodboard-generator`), which no competitor applies to this
category.

The connected Ideogram MCP already exposes the tool surface needed for this:
`generate_image`, `edit_image` (iteration/touch-ups), `generate_images_bulk`
(platform-dimension variant sets) — confirmed live per the task brief.

## Scope

One new skill: `skills/social-marketing-asset-generator/`.

Deliverable: a single finished marketing/social image asset (social graphic, banner, or
display ad) with its primary copy — headline, subhead/tagline, CTA, and a brand mark if no
existing logo file needs pixel-exact reproduction — rendered directly by Ideogram, not
composited afterward. Covers organic and paid social graphics, banner/display ads, and
platform-dimension variant sets of one creative concept (e.g., the same launch graphic
resized and re-composed for Instagram Story, Facebook feed, and a LinkedIn banner in one
request).

Triggers only on explicit request: "make me an Instagram ad for X," "banner ad for our
launch," "social graphic announcing Y," "give me this in Story + Feed + LinkedIn sizes."

Sibling relationship to the five existing skills, not a replacement for any of them:
- Reuses the shared structured-JSON-caption schema from `ideogram-prompt` and the
  anti-slop / compositional-deconstruction discipline pattern from `logo-prompting` /
  `brand-identity-sheet` / `moodboard-generator`.
- Assumes brand truth already exists or is gathered fresh per ad (name, colors, literal
  copy) — it does not explore brand direction from scratch (`moodboard-generator`'s job)
  and does not lock a full multi-artifact identity system (`brand-identity-sheet`'s job).
- Structural pattern matches every sibling: `SKILL.md` + `references/` + `examples/` +
  `evals/evals.json`.

Not a copywriting or ad-strategy skill. It gathers literal copy (headline, subhead, CTA,
offer/price if any) as intake and renders it — it does not run AIDA/PAS strategy
frameworks, write ad copy unprompted, or invent claims, prices, or offers that weren't
supplied. A rendered image with a fabricated discount figure or an unapproved claim baked
into the pixels is a real failure mode this skill must guard against, not a nice-to-have.

## Assumptions

Non-obvious calls made without a human in the loop, documented here per this run's
instructions rather than left implicit:

1. **Literal-copy-only, no invented claims.** "Finished asset" means the user-supplied (or
   explicitly confirmed) copy strings are treated as must-render-exactly text elements. If
   the ask is vague ("make an ad for our sale") with no literal copy given, the skill asks
   for exact headline/CTA/offer strings before drafting rather than inventing
   plausible-sounding marketing copy that could misrepresent a real offer.
2. **Platform pixel specs aren't always exactly achievable.** `generate_image` and
   `generate_images_bulk` only expose a fixed `aspect_ratio` enum (narrowest supported is
   `1x3`/`3x1`) and a fixed `resolution` enum — neither covers every platform's exact pixel
   spec (e.g. a LinkedIn personal banner is 1584×396, a 4:1 ratio, wider than the widest
   supported `3x1`). Where no supported ratio is a close match, the skill generates at the
   nearest supported ratio and explicitly discloses the mismatch rather than claiming
   pixel-exact platform compliance it can't deliver. (`reframe_image` could close this gap
   but is out of scope here — see Out of scope.)
3. **`generate_images_bulk` needs a Pro subscription** per its own tool description. Account
   tier isn't something this skill can check in advance, so a multi-placement request
   degrades to sequential `generate_image` calls per platform variant if bulk generation
   fails or is unavailable, rather than failing the whole request.
4. **`edit_image` is for localized touch-ups, not full copy changes.** Diffusion-based image
   edits aren't guaranteed surgical text replacement — asking `edit_image` to swap an entire
   headline is not the same reliability class as asking it to recolor a CTA button or swap a
   logo mark. For a full copy or layout change, the skill defaults to a fresh
   `generate_image` call from a corrected JSON caption instead of forcing `edit_image` past
   what it reliably does, and says so.
5. **"Baked-in typography" does not mean pixel-exact existing-logo reproduction.** A
   diffusion model cannot guarantee unaltered, pixel-fidelity reproduction of an existing
   logo file the way a compositor can. If a brand's logo must appear exactly as-is
   (a legal/trademark lockup requirement, for example), that's flagged as a limitation this
   skill's *generation* step can't satisfy, rather than silently attempted and delivered as
   if it were exact.

## Architecture

Prompt-composition skill, same family as `logo-prompting` / `brand-identity-sheet` /
`moodboard-generator` — not an MCP-orchestration skill like `collections-management`. It
does chain MCP calls (`generate_image` → optional `edit_image` touch-up → optional
`generate_images_bulk` platform fan-out), but the core work is composing a legible,
on-brand, on-copy JSON caption, the same center of gravity every prompt-composition sibling
has.

Reuses, rather than diverges from, the shared JSON caption schema documented in
`ideogram-prompt/references/json-caption-schema.md`: the `compositional_deconstruction`
mechanics (per-element `bbox`, one `text` element per line, per-element `color_palette`)
are exactly what "baked-in typography" needs. This skill's own
`composition-spec-format.md` is the ad-creative-specific instantiation of that same shared
schema — background/scene, product or subject element, headline/subhead/CTA text elements,
brand-mark element — not a divergent one.

One genuinely new reference surface, absent from every sibling: `platform-dimension-map.md`.
None of the five existing skills deal with external platform pixel constraints — they all
render at a fixed square or portrait ratio for a design-sheet or moodboard use case. This
skill is the first in the repo that has to reconcile Ideogram's fixed `aspect_ratio`/
`resolution` enums against real-world platform specs, several of which (LinkedIn banner,
generic display leaderboard sizes) simply aren't representable exactly.

Anti-slop discipline gets ad-specific bans layered on top of the universal list shared with
the siblings — marketing creative has its own cliché register (stock-photo hands-on-laptop,
sunburst "SALE" badges, generic arrow-pointing-at-CTA, fake star-rating clutter) distinct
from the brand-identity-sheet/logo-prompting registers, plus one failure mode unique to
in-image text rendering: garbled or illegible copy, which this skill must catch and
regenerate against rather than accept.

## Components

- **`SKILL.md`** — frontmatter triggers on ad/banner/social-graphic requests, even without
  the word "Ideogram." Body documents the workflow below: intake → resolve platform
  dimensions → compose JSON caption → anti-slop gate → generate → optional platform
  fan-out → legibility review → optional touch-up loop → save.
- **`references/composition-spec-format.md`** — the ad-creative instantiation of the shared
  JSON caption schema: background/scene, product/subject element, headline text element,
  subhead/tagline text element, CTA text element, brand-mark element, each with a `bbox`
  placed with the target platform's safe zones in mind (see `platform-dimension-map.md`).
- **`references/anti-slop-discipline.md`** — universal bans shared in spirit with the
  sibling skills, plus ad-specific additions: no generic "SALE!" starburst badge, no
  stock-photo hands-on-laptop, no fake star-rating clutter, no arrow-pointing-at-button
  unless the brand's own reference imagery calls for it, no illegible/gibberish rendered
  text, no low-contrast text-on-busy-background that would fail at thumbnail size.
- **`references/platform-dimension-map.md`** — table of common placements (Instagram
  feed/story/reel, Facebook feed/cover, LinkedIn post/banner, X/Twitter post, YouTube
  thumbnail, Pinterest pin, generic display banner sizes) mapped to the nearest supported
  `aspect_ratio` + `resolution` enum values, each row flagged exact-fit or
  nearest-approximation, plus each platform's UI-chrome safe zones (profile photo, caption
  overlay, sponsored-label area) that baked-in text elements must avoid.
- **`references/typography-baking-discipline.md`** — the core differentiator writeup: why
  baked-in text beats generate-then-composite for this repo's methodology, legibility rules
  (max words per line, text size relative to canvas, contrast-checked background/text hex
  pairing), the one-message-hierarchy-per-asset rule (headline dominant, CTA secondary, no
  third competing text block), and the `edit_image`-vs-regenerate decision rule for
  touch-ups (Assumption 4).
- **`examples/`** — worked ad-creative jobs, added as real jobs are run; empty at spec time,
  same as every sibling skill starts.
- **`evals/evals.json`** — 3 eval prompts in the existing schema (`id`, `eval_name`,
  `prompt`, `expected_output`, `files`), matching `collections-management`'s format.

## Data flow

1. **Intake** — brand truth (existing `brand.md` if one exists; otherwise name, colors, a
   product/offer image if relevant) plus the literal copy strings (headline, subhead, CTA,
   offer/price if any) and the target placement(s). If literal copy isn't supplied, ask for
   it before drafting anything — never invent a claim, price, or offer (Assumption 1).
2. **Resolve platform dimensions** — for each requested placement, look up
   `platform-dimension-map.md` for the nearest supported `aspect_ratio`/`resolution`. If no
   close match exists, tell the user up front which placement(s) will be an approximation,
   rather than silently delivering a mismatched asset as if it hit spec (Assumption 2).
3. **Compose the JSON caption** — background/scene + product/subject + text elements
   (headline/subhead/CTA/brand-mark) per `composition-spec-format.md`, placing each
   element's `bbox` inside the resolved platform's safe zones.
4. **Anti-slop gate** — score the drafted caption against `anti-slop-discipline.md`'s bans
   before generating, same pre-generation-gate pattern `moodboard-generator` already uses.
   A caption that fails the gate gets revised, not sent as-is.
5. **Generate** — single placement: `generate_image` with the composed prompt, the
   resolved `aspect_ratio`/`resolution`, `style_type: "DESIGN"`, `rendering_speed:
   "QUALITY"` (a text-bearing, legibility-sensitive image, same rationale the sibling
   skills already use for panel/text-heavy renders). Multiple placements from one concept:
   `generate_images_bulk` with one prompt per resolved platform variant (same underlying
   concept, text-element bboxes adjusted per platform), falling back to sequential
   `generate_image` calls if bulk generation is unavailable (Assumption 3).
6. **Legibility review** — check the generated image for garbled or illegible rendered
   text, Ideogram's known failure mode on dense in-image copy. If it's garbled, regenerate
   from the JSON caption with adjusted text-element sizing/spacing rather than accepting a
   broken render.
7. **Touch-up loop (optional)** — for a genuinely localized fix (recolor a CTA button, swap
   a logo mark, adjust one line of copy), call `edit_image` per its strict "edit this exact
   image" contract. For a fuller copy or layout change, regenerate from a corrected JSON
   caption instead of forcing `edit_image` past what it reliably does (Assumption 4) — and
   say that's what's happening.
8. **Save** — per the toolkit's "No Context Lost" habit, write the final JSON caption(s),
   prompt(s), the platform/dimension mapping actually used (including any disclosed
   mismatches), and the resulting image URL(s)/`response_id`s to the project's existing
   output location, matching whatever folder convention the other skills already use for
   that project.

## Error handling

- No literal copy supplied and the ask is vague ("make an ad for the sale") → ask for exact
  headline/CTA/offer strings before drafting; never fabricate a claim, price, or offer
  detail.
- Requested platform placement has no close `aspect_ratio` match → tell the user explicitly
  which placement(s) will be a nearest-fit approximation before generating, don't silently
  ship a wrong-ratio asset as if it matched spec.
- Garbled or illegible rendered text on generation → treat as a failed render, not an
  acceptable result; regenerate with adjusted text-element bbox/sizing rather than
  delivering it.
- `generate_images_bulk` unavailable (non-Pro account, or a tool error) → fall back to
  sequential `generate_image` calls per platform and tell the user that's what happened
  (the latency/cost tradeoff), don't silently fail the whole request.
- `edit_image` requested for a change bigger than a localized touch-up (e.g., "change the
  whole headline and layout") → don't force it through `edit_image`; explain that a full
  copy/layout change goes through a fresh `generate_image` call from a corrected caption
  instead, per its own strict single-image-edit contract.
- Existing brand logo must appear pixel-exact and unaltered → flag that Ideogram generation
  can't guarantee pixel-fidelity reproduction of an existing logo asset; recommend the user
  composite that exact logo file in afterward with their own tooling for that specific
  requirement, rather than this skill claiming to bake in a logo it can only approximate
  (Assumption 5).

## Testing

Standard `evals/evals.json` pattern, 3 realistic prompts:
1. "Make me an Instagram feed ad for [product] with headline 'X' and a 'Shop Now' button" —
   literal-copy path, single placement, verifies the rendered text elements match the
   supplied copy exactly and the anti-slop gate runs before generation.
2. "I need this same launch graphic in Instagram Story, Facebook feed, and LinkedIn banner
   sizes" — platform-dimension fan-out via `generate_images_bulk`, verifies
   `platform-dimension-map.md` gets consulted per placement and the LinkedIn-banner
   near-miss (no supported ratio reaches 4:1) is disclosed to the user rather than silently
   delivered as if it matched spec.
3. "Make an ad for our big sale" (no literal copy given) — verifies the skill asks for exact
   headline/CTA/offer copy before drafting instead of inventing a claim or discount figure.

## Out of scope (this spec)

- **HTML/CSS compositing pipeline or any post-generation text overlay step.** The whole
  point of this skill is baked-in generation; a composite fallback would reproduce exactly
  the workaround this skill exists to differentiate against.
- **Ad copywriting or strategy** (AIDA/PAS frameworks, audience targeting, budget/campaign
  planning). This skill accepts literal copy as intake; it doesn't originate marketing
  strategy or write ad copy unprompted.
- **Video or animated formats** (Reels/Stories video, GIF, motion banners). Static image
  only, matching the confirmed tool surface — `generate_image`, `edit_image`, and
  `generate_images_bulk` are all static-image tools.
- **Pixel-exact reproduction of an existing, unaltered logo file** inside the generated
  composition. Flagged as a real limitation (Assumption 5), not solved by this skill.
- **`reframe_image` / `upscale_image` MCP tools.** Both exist and could help close the
  platform-dimension-mismatch gap noted in Assumption 2 (`reframe_image` in particular
  reframes an existing image to an arbitrary aspect ratio), but neither was in this skill's
  confirmed relevant-tool list for this spec. Adding `reframe_image` to close that gap is a
  reasonable follow-up, but it's a scope decision for a future revision, not this one.
- **Auto-wiring into `brand-identity-sheet` / `logo-prompting` / `moodboard-generator` /
  `character-model-sheet`** to proactively suggest generating ad creative from their
  output. Explicit-request trigger only, matching the convention
  `collections-management` already established for this repo.
