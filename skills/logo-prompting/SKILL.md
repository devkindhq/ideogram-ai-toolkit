---
name: logo-prompting
description: Living reference for writing logo and brand-mark image-generation prompts (Ideogram-first, via ideogram-pack:ideogram-core-workflow-a). Use whenever the user asks for a logo prompt, a moodboard prompt, a wordmark/icon direction, or wants feedback on a logo image-gen prompt they've drafted — even if they just paste a prompt and say "is this good" or "make this better." Also use when a brand.md exists and the user wants its Visual layer turned into an actual generation prompt. This skill accumulates real examples over time in examples/ — check there for a matching style family before starting from scratch, and append new accepted prompts back into it after each real logo job.
---

# Logo Prompting

Writing a good logo prompt is a compression problem: you're translating a brand's strategy and voice into a short, literal, image-model-legible spec — while actively fighting the model's own priors, which default to AI-startup visual clichés (gradients, neural nodes, glowing orbs, soundwave bars) unless explicitly told not to.

This skill exists because a logo prompt has two failure modes, and they pull in opposite directions:
1. **Too vague** → the model falls back to generic SaaS-logo training-data averages.
2. **Too literal / over-specified** → the model draws exactly the clichéd icon you named (a phone for a calling app, a shield for security, a lightbulb for ideas) instead of an abstract mark that earns its meaning.

Good logo prompting threads between these: specific about constraints (palette, type, mood, what to avoid), abstract about the mark itself (suggest, don't depict).

## Before writing a prompt: gather the four inputs

Don't start typing a prompt cold. Pull these four things first — they're the actual inputs, the prompt is just their compressed form:

1. **Brand truth** — read the project's `brand.md` if one exists (Strategy + Voice + Visual layers). If there's no `brand.md`, ask for the equivalent: what the brand is, what it's not, the archetype, the core promise. A logo prompt with no brand truth behind it is decoration, not identity.
2. **The Visual layer specifically** — Colors (exact hex, named tokens, and their *usage rule* — which color is primary vs. reserved-for-one-state), Typography (display face + weight + roman-only), Style keywords + reference brands. See `references/brand-visual-layer.md`.
3. **Intake context** — what is this mark actually for (wordmark, icon-only, favicon, both), what does it need to survive (16px favicon, dark mode, embroidery, print), who reads it (a shift worker at 2am glancing at a phone screen reads differently than a landing-page visitor). See `references/logo-offer-questions.md` for the fuller intake list — use it as an interview checklist when the user hasn't already supplied this.
4. **The anti-slop guardrails** — the specific clichés this brand must never look like (see `references/anti-slop-discipline.md`). Every brand has its own version of "don't look like a generic AI startup" — for a fintech it might be "no padlock icon," for a care-work app it's "no clinical cross," etc. Name the negatives explicitly; a model that isn't told what to avoid will reach for it.

## Writing the prompt

**Structure, in order:**
1. What it is (wordmark / mark-only / lockup) and for what brand, one clause of category context.
2. Type treatment: exact face or face-description, weight, case, roman-only if that matters.
3. Color: exact hex or named token, and *which* color dominates vs. which is a reserved accent — don't let the model treat every brand color as equal-weight.
5. The motif, if there is one — described as an abstract suggestion ("a small integrated detail suggesting X"), never as a literal object unless the brand truly wants a literal object.
6. Negative space: the specific clichés to exclude (from `references/anti-slop-discipline.md`), stated plainly — "no gradients, no neural-network nodes, no glowing orb" reads better to Ideogram than a vague "make it not look like AI."
7. Mood / reference anchor: one sentence naming the *feeling* and, if useful, a real-world reference object (a radio dial, a carbon-copy ticket, a uniform patch) rather than a design-adjective salad.
8. A legibility/use-case constraint if relevant ("reads clearly at 16px favicon size").

**One prompt, one direction.** If the user wants multiple directions to choose from, write multiple *structurally distinct* prompts (different form, different type treatment, different motif — see the "structural variety over palette-swaps" rule below), not one prompt with several palette options bolted on.

### Structural variety over palette-swaps

Borrowed from the Hallmark design skill's core insight: when producing more than one direction, each one needs its **own locked token set** (its own named colors, its own type pairing, its own motif) and its own *form* (wordmark vs. badge vs. mark-only vs. monospace-as-identity). Two directions that share a form and only swap the accent color are not two directions — they're one direction with a color picker. See `examples/fleetline-four-directions.md` for a worked example of four genuinely distinct directions from one brand brief.

### Honest specificity, not adjective soup

"Modern, clean, professional, innovative" tells an image model nothing — every logo in its training data claims those words. Replace adjectives with things the model can actually render: an exact hex, an exact typeface name or type-genre, a named real-world reference object, an explicit exclusion list. If you catch yourself writing three unanchored adjectives in a row, stop and ask which one has a concrete visual referent.

## The moodboard prompt (when the ask is bigger than one mark)

Sometimes the request is "help me get oriented on the whole visual identity" before locking a single logo — that's a moodboard prompt, not a logo prompt. Use the 3×3 grid template in `references/moodboard-template.md` (Color Palette / Typography / Logo Exploration / Iconography / Photography & Graphics / Material Samples / Abstract Patterns / Mood Imagery / Application Mockup). Fill each panel from the brand's Visual layer the same way you would a logo prompt — same anti-slop discipline applies to every panel, not just the logo-exploration one.

## After the fact: capture what worked

This skill is meant to get better as real logo prompts get run and judged. When a prompt produces a result the user actually likes (or explicitly corrects), that's signal worth keeping:

- Add or update a file under `examples/<brand-or-style-family>.md` with the brief, the final prompt, and one line on what worked or what had to change and why.
- If a *pattern* emerges across multiple brands (not just one brand's specific palette, but a reusable move — e.g. "monospace-as-primary-identity reads as 'record-keeping, non-digital' reliably"), promote it into this SKILL.md or into `references/anti-slop-discipline.md` as a named technique, not just left buried in one example file.
- Group examples by **style family** (e.g. `analog-industrial.md`, `dark-mode-minimal.md`, `paper-record.md`), not by client name alone — the point is to make the pattern reusable across brands, the way `examples/fleetline-four-directions.md` already demonstrates four such families from a single brief.

## Reference files

- `references/brand-visual-layer.md` — how to read a brand.md Visual layer (Colors/Typography/Style) and turn its usage rules into prompt language.
- `references/logo-offer-questions.md` — the intake checklist for a logo job when no brief exists yet.
- `references/anti-slop-discipline.md` — the anti-AI-slop guardrails (no gradients, no neural-network/circuit-board/glowing-orb clichés, locked named tokens, structural variety), adapted from the Hallmark design skill for image-generation prompts specifically.
- `references/moodboard-template.md` — the 3×3 moodboard prompt template.
- `examples/fleetline-four-directions.md` — worked example: one fictional brand ("Fleetline," a voice-dispatch AI product), four structurally distinct logo directions from a single brief.
