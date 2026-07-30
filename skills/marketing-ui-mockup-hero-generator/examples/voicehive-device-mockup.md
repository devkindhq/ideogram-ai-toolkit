# Worked Example: VoiceHive Device-in-Context Product Mockup

A real, end-to-end run of workflow steps 1-5 and 8 for the device-in-context mockup
format, using the fictional "VoiceHive" brand (already established in
`brand-identity-sheet/examples/`) for cross-skill consistency, per the spec's Testing
item 2: "a device mockup showing VoiceHive's app running on a laptop, sitting on a
desk, for the pitch deck." No reframe was run for this example — the spec's Testing
item 2 requests only the single desk-scene placement, no additional variant.

## Compositional layer assignment (workflow step 2)

- **Background/Environment** — a wooden desk surface with a coffee cup and a
  notebook nearby, soft natural window light.
- **Device Frame/Chrome** — a MacBook Pro-style laptop, silver aluminum body,
  three-quarter angle.
- **Supporting UI Chrome** — a minimal browser tab bar visible at the top of the
  screen.
- **Product Visual** — VoiceHive's actual call-transcription UI: a live-call panel
  and a transcript pane.

Per `mockup-anatomy.md`, this format has **no separate Nav Bar layer** — confirmed:
the drafted composition has no top-level nav bar; any in-app navigation exists only as
part of the Product Visual's on-screen content.

## 1. The paragraph prompt

> A photorealistic device-in-context product mockup for the voice-AI brand VoiceHive:
> a silver aluminum MacBook Pro-style laptop sits at a three-quarter angle on a warm
> wooden desk surface, softly lit by natural window light from the left, with a white
> ceramic coffee cup and an open notebook resting nearby. The laptop screen shows a
> minimal browser tab bar at the top, and below it VoiceHive's call-transcription app
> in use: a live-call panel on the left with an animated waveform, and a transcript
> pane on the right showing readable lines of dialogue text scrolling in real time.
> The screen has a subtle, realistic glass reflection of the window light, without
> obscuring the on-screen content. Warm, editorial product-photography lighting,
> shallow depth of field with the desk foreground slightly soft, shot for a
> pitch-deck slide. No neural-network nodes, no glowing orb, no purple-to-pink
> gradient wash, no illegible scribble-text on the transcript pane.

**What this prompt does right for the `REALISTIC` photographic register:** the
desk/laptop material details (silver aluminum, warm wood, natural window light,
shallow depth of field) are named specifically enough to anchor a photographic result
rather than a flat product render; the on-screen content (live-call panel, transcript
pane with legible dialogue) is described concretely enough that the screen reads as
real software rather than an indistinct glow — directly the risk this format's
`REALISTIC` style path carries, per the anti-slop gate's "Screen content legibility"
axis below.

## Anti-slop gate scoring (workflow step 4, `references/anti-slop-discipline.md`)

Particular attention to "Screen content legibility," since a `REALISTIC`
device-in-context shot risks the screen content reading as an indistinct glow rather
than legible UI at a distance.

| Axis | Score | Why |
|---|---|---|
| Copy completeness | 3 | The transcript pane's dialogue is described as "readable lines of dialogue text" rather than exact quoted copy — acceptable for a photographic desk-scene shot where legible-but-generic call dialogue is standard set dressing, but scores lower than Task 5/7's exact-copy prompts since no literal transcript lines were pre-written. |
| Chart/data specificity | n/a | No chart/stat element in this format's layer assignment — not applicable. |
| Universal-ban sweep | 5 | Prompt explicitly negates neural-network nodes, glowing orb, purple-to-pink gradient wash, and illegible scribble-text. |
| Screen content legibility | 4 | Live-call panel and transcript pane are both named with specific functional content (waveform, scrolling dialogue) rather than a vague "app UI," giving the model concrete elements to render legibly rather than defaulting to decorative glow. |
| Layer completeness | 5 | All four layers from the assignment above are named in reading order: Background/Environment, Device Frame/Chrome, Supporting UI Chrome, Product Visual. |

All applicable axes score 3 or above — proceeded to generation without revision.

## Generation (workflow step 5)

Called `mcp__ideogram__generate_image` with the prompt above, `style_type:
"REALISTIC"`, `rendering_speed: "QUALITY"`, `aspect_ratio: "4x3"` (a
pitch-deck-slide-friendly ratio for a desk-scene device mockup).

- **Request ID:** `IzPjiWiTQNK2CRBiAKTHCw`
- **Response ID:** `0AhyEGDgXUWmawtiw6a2cQ`
- **Image URL:** https://ideogram.ai/assets/image/balanced/response/0AhyEGDgXUWmawtiw6a2cQ@2k
- **Permalink:** https://ideogram.ai/g/IzPjiWiTQNK2CRBiAKTHCw/0
- **Downloaded to:** `examples/images/voicehive-device-mockup.webp` (real file, 1024x768 WebP, confirmed via `file`)

![VoiceHive device-in-context mockup](images/voicehive-device-mockup.webp)

**Note on the actual result vs. the prompt:** the generation did not render a literal
audio waveform as asked — instead it rendered a call header ("Call: Marcus Chen — Q2
Strategy" with a running timer and call-control icons) above the "Live Transcription"
pane, which fulfills the same functional role (a legible, specific call-in-progress
UI) even though the exact visual element differs from the prompt's literal ask. This
is documented honestly here rather than described as if the waveform were present —
the compositional deconstruction below describes what the image actually shows.

No reframe was run for this example, per the spec's Testing item 2 (no additional
placement requested).

## 2. The compositional deconstruction

Bbox values are estimated on a normalized 0-1000 grid (image is downscaled/estimated
by eye, not measured from source pixel coordinates — coarse-but-honest per
`references/composition-spec-format.md`'s guidance).

```json
{
  "high_level_description": "A photorealistic device-in-context product mockup showing VoiceHive's call-transcription app open on a silver laptop on a wooden desk, with a coffee cup, notebook, and plant nearby, shot for a pitch deck.",
  "compositional_deconstruction": {
    "background": "A warm wooden desk surface under soft natural window light, with a monstera plant at upper-left, a white ceramic coffee cup and saucer at right, an open notebook with a pen and an AirPods-style case at lower-left.",
    "elements": [
      {
        "type": "obj",
        "bbox": [220, 60, 1000, 780],
        "desc": "Device Frame/Chrome — a silver aluminum MacBook-style laptop, open at a three-quarter angle, screen tilted toward the viewer, keyboard and trackpad visible below."
      },
      {
        "type": "obj",
        "bbox": [250, 110, 700, 150],
        "desc": "Supporting UI Chrome — a minimal browser tab bar at the top of the screen with a single active tab labeled 'VoiceHive' and standard back/forward/address-bar icons."
      },
      {
        "type": "text",
        "bbox": [260, 175, 620, 210],
        "text": "Call: Marcus Chen — Q2 Strategy",
        "desc": "Product Visual — the active-call header at the top of the app screen, with a running call timer beneath it and two green call-control icons plus a small participant avatar at top-right."
      },
      {
        "type": "text",
        "bbox": [470, 260, 700, 460],
        "text": "Live Transcription",
        "desc": "Product Visual — the transcript pane on the right side of the screen, headed 'Live Transcription', containing several legible lines of speaker-labeled dialogue ('Marcus:' / 'You:')."
      },
      {
        "type": "text",
        "bbox": [260, 455, 340, 470],
        "text": "VoiceHive",
        "desc": "Product Visual — a small wordmark in the lower-left corner of the app screen, confirming brand attribution on-screen."
      }
    ]
  }
}
```
