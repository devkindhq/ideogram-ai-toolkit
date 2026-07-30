# Set-Consistency Workflow: Batch → Review → Correct → Extend

This file expands `SKILL.md` workflow steps 5-8: generating the batch, reviewing it as a
set, correcting drift, and extending the set later. It also covers the
`remix_image` vs. `edit_image` decision, the custom-model-training escalation path, and an
honest callout on the one part of this workflow that rests on an unverified assumption.

## 1. Batch-generate

Call `generate_images_bulk(prompts=[...])`. Per the tool's own description, this is
submitted as a background job that doesn't block the conversation — the user can keep
chatting while images render into a carousel as they complete, in carousel-capable
clients. In a text-only client, or whenever the user asks for status, call
`get_generation_status` (optionally passing the specific `request_id` from this batch) to
get back a markdown status table (`Request ID | Status | Prompt | URL`) that should be
rendered verbatim rather than re-summarized.

`generate_images_bulk` requires a Pro subscription, per the tool's own description. If the
call errors on subscription tier, surface that error to the user verbatim and stop —
ask before doing anything else. Do not silently fall back to looping individual
`generate_image` calls to work around the error. That "workaround" changes both cost (N
separate calls instead of one batch) and behavior (no shared batch semantics) without
telling the user, and it's exactly the kind of quiet behavior-swap this toolkit's
"surface every failure" discipline exists to prevent.

## 2. Review the finished set as a set

Once all N images have rendered, look at the full set together — not one at a time in
isolation. This is the core value proposition of the whole skill: a per-icon review would
approve every individual image (each one, viewed alone, looks fine), and only a
side-by-side set review catches the icon that's subtly off-angle or a shade glossier than
its siblings. Checking icons one-by-one as they render, and moving on once each one looks
acceptable in isolation, defeats the entire reason this skill exists.

Of the five axes in `style-lock-recipe.md`'s checklist, three are the hardest to hold
across a batch and deserve the closest look during this pass:

- **Material/finish consistency** — does the requested surface (matte plastic, clay
  sheen, gouache texture, brushed metal) read the same way on every icon, or has one
  icon drifted toward a glossier or flatter finish than the rest?
- **Lighting consistency** — are the key-light direction, shadow angle, and highlight
  placement the same across every icon, or has one icon's lighting rotated or softened
  relative to the group?
- **Camera-angle consistency** — is every icon viewed from the same angle (the same
  isometric tilt, the same three-quarter angle, the same straight-on framing), or has one
  icon been rendered from a subtly different perspective?

If all icons pass on all five axes, generation is done. If any icon drifts on any axis,
move to the correction step below.

## 3. Correct any outlier: `remix_image` vs. `edit_image`

**Prefer `edit_image` with multi-reference anchoring.** Pass 2-3 "anchor" icons — icons
from the finished batch that clearly show the target style — as `image_response_ids`,
plus a corrective prompt describing the fix (e.g., "match the material and lighting of
these two anchor icons; fix the 3D shading on this one so the highlight direction and
matte finish match the rest of the set"). Multi-reference conditioning pulls the result
toward the group in a way a single reference can't as reliably, per `edit_image`'s own
support for combining multiple `image_response_ids` into one edit conditioned on all of
them at once.

**Fall back to `remix_image` against exactly one anchor image** when only one strong
anchor icon exists in the set (e.g., a 3-icon batch where only one icon clearly nails the
target look). Pass that icon's `image_response_id` and tune `image_weight` toward the
higher end — 70-85 out of the tool's 1-100 range, where higher means closer to the
original — for "match this closely."

**Both tools fail loudly, by design.** Per both tools' own descriptions, `remix_image` and
`edit_image` are strict: "remix" means a variation of *this* image, "edit" means edit
*this* image, not "generate something similar." If either call fails or can't proceed for
any reason, that failure is surfaced to the user verbatim and the tool stops — it does not
fall back to a fresh `generate_image` or `generate_images_bulk` call to produce a
"similar" result instead. Match that discipline here: if a correction call errors, report
the real error and ask before trying anything else. Do not quietly swap in a regeneration
that only approximates what was asked for.

## 4. Don't regenerate the whole batch

A partial-drift problem is a targeted correction, not a reason to redo the batch. Two
reasons this matters:

- **No better odds the second time.** A fresh `generate_images_bulk` call has no better
  odds of consistency than the first attempt unless the style-block text itself is also
  fixed — Ideogram has no memory of the first batch, so resubmitting the same prompts (or
  prompts with the same underlying style-block wording) is likely to reproduce the same
  drift, not fix it.
- **Wasted work and wasted spend.** A full regeneration discards every icon that already
  matched the target look — work that was already correct — and resubmits a Pro-gated
  batch call to redo it. The targeted `edit_image`/`remix_image` correction from step 3
  fixes only the icon that actually drifted, at a fraction of the cost.

## 5. Extend the set later

When adding new icons to an existing set, use the same multi-reference `edit_image`
pattern from step 3: pass 2-3 existing icons' `image_response_ids` as anchors, plus a
prompt for the new subject that follows the exact subject-block phrasing style already
used for the rest of the set (see `style-lock-recipe.md`'s subject-block guidance — name
the subject concretely, keep it short, let the anchors carry the style). Do not call
`generate_images_bulk` again for an extension. A new batch call has no memory of the
earlier batch's exact style-block wording, so it's a weaker consistency guarantee than
conditioning directly on the images that already exist and already match.

## 6. Custom-model-training escalation path

Raise this option when a project's icon count and consistency needs have grown past what
prompt-text-only style-locking is holding up: 10+ icons in the set, or icons generated
across multiple sessions where drift has crept in and per-icon corrections are becoming
harder to keep track of.

The mechanism: train a small custom model on 5-10 already-locked icons via
`custom-model-training`. As of this writing, that skill exists only as a design spec at
`docs/superpowers/specs/2026-07-30-custom-model-training-design.md` in this repo, not a
shipped skill — say so plainly if a user asks to invoke it before it's built. Once it
exists and a custom model has been trained, pass the resulting `custom_model_uri` into
`generate_images_bulk`. That unlocks `style_type` and `seed` as real levers on the
custom (v3) pipeline — per `generate_images_bulk`'s own tool description, `style_type`,
`negative_prompt`, `magic_prompt_option`, and `seed` apply only to a custom-model (v3)
generation and are ignored on the default v4 path used everywhere else in this workflow.

This is documented here as an option, never auto-invoked. Matching
`collections-management`'s "explicit trigger only" discipline for its own non-auto-wiring
into sibling skills, this skill raises the escalation path when the threshold above is
met but never chains into `custom-model-training` on its own.

## 7. Unverified: does a shared `seed` actually help across varying subjects?

Once `custom_model_uri` is set, `seed` becomes a live, non-ignored parameter — but what it
buys you for *this* workflow is not something this skill can state as fact.

Per the repo-wide rule against stating unverified third-party API behavior as fact: it is
unconfirmed whether passing a shared `seed` value across `prompts[]` entries with
genuinely different subjects (a clock, a notebook, a camera) meaningfully improves
material/lighting consistency across those icons. Seed reproducibility is documented for
identical-prompt reruns — the same prompt, same seed, same result — not for a batch of
varying subject prompts that merely share one seed value while the subject text differs
per entry.

Treat trying a shared `seed` as an experiment worth comparing against a no-seed baseline
on a real project (once a custom model makes `seed` a live parameter at all), not as a
guaranteed consistency lever to assume up front. If a user asks whether shared `seed`
will lock consistency across a varying-subject batch, say plainly that this is unverified
and worth testing, not a confirmed mechanism.

## Self-review checklist

- [ ] Batch-generate section covers the background-job behavior, `get_generation_status`
      polling, the Pro-subscription requirement, and the "surface the error, don't
      silently loop `generate_image`" rule.
- [ ] Review-as-a-set section states the per-icon-review-would-pass framing as the core
      value proposition, and names material, lighting, and camera-angle consistency as
      the three hardest axes to hold.
- [ ] Correction section states the `edit_image`-with-2-3-anchors default, the
      single-anchor `remix_image` fallback with an `image_weight` range, and the
      fail-loudly/no-silent-fallback rule for both tools.
- [ ] The "don't regenerate the whole batch" rule is present with both reasons: no better
      odds without a style-block fix, and wasted work/spend on already-correct icons.
- [ ] Extend-the-set section is present and reuses the same multi-reference `edit_image`
      pattern rather than a fresh batch call.
- [ ] Custom-model-training escalation path states the trigger threshold, the mechanism,
      the current spec-only status of that skill, and the "option, never auto-invoked"
      framing.
- [ ] The unverified-`seed` callout is present and explicit about what's confirmed
      (identical-prompt reruns) versus what isn't (varying-subject batches).
