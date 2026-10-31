# Panel Anatomy — The 9-Panel Moodboard Template

Seeded from a real, tested Notion prompt (⭐⭐⭐⭐ rated, used on Ideogram). Unlike
`brand-identity-sheet`'s pick-and-choose panel set, all nine panels below are used on
every moodboard — the point of this format is breadth-in-one-glance across a brand's
whole visual territory, not a curated subset.

## The template variables

Fill these from step 1 of the main workflow before drafting panel descriptions:

- `{Adjectives}` — three words describing the brand's personality
- `{Color Focus}` — chosen palette or accent hue (can be open/exploratory)
- `{Logo Concept}` — primary symbol or motif envisioned (can be blank/exploratory)
- `{Imagery & Texture}` — preferred visuals or material references
- `{Emotional Impact}` — the single feeling the whole board should evoke
- `{Notes}` — anything else (competitors to avoid, constraints, a reference image)

## The nine panels

### 1. Color Palette
Swatches reflecting `{Color Focus}` — 4-6 named color chips, not just a gradient strip.
If the palette is still open, this panel is allowed to *propose* 2-3 candidate
directions rather than commit to one; that's the exploratory function a moodboard
serves that `brand-identity-sheet` (which requires a locked palette already) doesn't.

### 2. Typography
Type samples that embody `{Adjectives}` — a display face paired with a body face,
shown at a few sizes, chosen for how the *shapes* of the letterforms carry the
adjectives (a "grounded, unhurried" brand reads differently in type than a "sharp,
urgent" one). This is a feel-test, not a locked type system yet.

Specifically, it is a feel-test on letterform *character*: weight, formality, era. It is
never a candidate for the actual typeface, because the model renders shapes resembling
type rather than setting a face. If a brand's typography is already locked in a real font
file, this panel cannot second-guess that choice. It can only confirm or complicate the
mood the locked choice was going for.

### 3. Logo Exploration
Loose sketches or stylized drafts of `{Logo Concept}` — 2-3 small rough variations
rather than one finished mark. This panel is deliberately allowed to be more literal
and unpolished than a real `logo-prompting` output; its job is to spark a direction for
a later logo pass, not ship a final mark. If there's no logo concept yet, this panel can
explore 2-3 abstract directions instead of forcing a literal symbol.

### 4. Iconography Style
Supporting geometric or illustrative icons in a consistent style — testing "what does
the *system* of icons around this brand look like," distinct from the app-icon grid in
`brand-identity-sheet` (which tests specific material samples for a locked brand). Here
the test is simpler: line weight, corner radius, geometric vs. organic — the icon
*language*, not specific icon content.

### 5. Photography & Graphics
Visuals aligned with `{Imagery & Texture}` — real-world reference imagery (a
photography style, a graphic treatment) that signals the brand's visual world. Name
actual references (a photographer's aesthetic, a specific material or lighting quality)
rather than "some nice photos."

If the brand's claim runs through people at all, trust, care, service, a made-by-humans
position, anything in the strategy or voice layer naming who the brand is *for* or *by*,
this panel needs an explicit instruction about who is in frame and at what eye level.
Silence here does not produce a neutral panel. It produces whatever the model's training
average supplies, which skews to either nobody or the wrong somebody: a generic tech
worker at a desk, when the claim was about a different kind of person entirely. Naming the
subject is as load-bearing here as naming the material vocabulary is for Material Samples.

### 6. Material Samples
Textural swatches — paper grain, brushed metal, woven fabric, whatever
`{Imagery & Texture}` points to physically. This panel tests tactile/material identity
separate from color or type — what would this brand feel like if you could touch it.

### 7. Abstract Patterns
Graphics reinforcing the brand mood without being literal — a repeating geometric
motif, a texture-as-pattern, something that could become a background treatment or
packaging pattern later. Distinct from panel 6 (a *sample* of a material) — this is a
*pattern* built from the brand's visual language.

### 8. Mood Imagery
Photography or illustration evoking `{Emotional Impact}` directly — the panel most
explicitly testing "does this feel like the one feeling we're going for," often the
panel a user reacts to most strongly first.

The same applies here as in Photography & Graphics. If the feeling depends on a
relationship between people, or between a person and their work, that relationship needs a
named subject and a named vantage point, not just an adjective. "Calm and still" alone
will as readily produce an empty room as a person at ease in one.

### 9. Application Mockup
One simple collateral example (a business card, a social post, a product label) showing
several of the above elements working together in a real context. This is the "does
this actually cohere as a system, not just nine unrelated panels" check — the payoff
panel that proves the exploration adds up to something usable.

## What makes a panel earn its place

Every panel should trace back to one of the six template variables — if you can't say
which input a panel is testing, it's decorative filler, not exploration. The whole
board should read as nine angles on the *same* three adjectives and one feeling, not
nine unrelated pretty pictures.

## Depicting versus embodying

A panel can fail while doing exactly what it was asked to do, if what it was asked to do
was draw the *symbol* of a value rather than find the visual choice that makes the value
true. This shows up most on relational values, humanism, trust, care, warmth, honesty,
because each has an obvious literal icon (a pair of hands, a handshake, a heart) that a
model reaches for the moment the word appears, and the icon reads as a cliche standing in
for the claim rather than as the claim.

**Depicting a value draws its symbol. Embodying a value makes a choice that only makes
sense if the value is real.**

A drawn hand is a symbol of care. A photograph of a real nurse at a patient's eye level,
not looking down at them, is care embodied in a framing decision; nobody frames it that
way by accident. A drawn padlock is a symbol of security. A product screenshot showing
exactly what data is and is not requested, in a UI that could ship, is security embodied
in a design decision.

The test: **if the visual choice would look identical whether or not the brand actually
holds the value, it is a symbol.** Swap the brand and the panel still works, which is the
same test this file already applies to adjectives and material vocabulary, one level up.
If the choice only makes sense *because* the value is real, a specific casting, a specific
framing, a specific thing shown or deliberately withheld, it is closer to an embodiment,
and worth pursuing even when it is harder to write as a single prompt clause than the
literal symbol would be.

This is not a ban on literal symbols. A security product's mark genuinely can be a
padlock, per `logo-prompting`'s own exception clause. It means knowing which one you are
doing, and not mistaking the presence of the symbol for having made the claim visually
true.
