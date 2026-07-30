# Fizzwright/Hopcarry Custom Model — Result

Worked example produced by actually running the `custom-model-training` pipeline end to end (2026-07-30).

## Identifiers

- `dataset_id`: `LegGhp1HRoePI_bPzWEW8A` (`fizzwright-hopcarry-training-set`, 15 uploaded assets)
- `model_id`: `ufUDWjYqSN-UWZklfVMQ-A`
- `model_name`: `fizzwright-hopcarry-brand-v1`
- `custom_model_uri`: `model/RvRKzGXWQMq3OQPdKuA--g/version/0`
- Training duration: `2026-07-30T02:34:48Z` → `2026-07-30T03:54:09Z` (1h19m) — on the fast end of the observed 1.5–20+ hour range documented in `references/dataset-requirements.md`.

## Proof generation

- Prompt: "Kip, the friendly courier fox mascot, delivering a Fizzwright craft-soda order to a customer's front door. Kip stands mid-doorstep holding a satchel with two Fizzwright bottles peeking out, one paw offering the delivery box forward with a cheerful grin, tail mid-wag. Warm porch lighting, welcome mat, a house door slightly ajar. Consistent Fizzwright/Hopcarry brand palette and character design."
- Settings: `rendering_speed: QUALITY`, `aspect_ratio: 4x5`, `custom_model_uri` set to the trained model above.
- `response_id`: `FzJwY0uTUlmNVtDba_82Vw`
- Output file: `kip-delivering-fizzwright-order.webp` (896x1120)
- Permalink: https://ideogram.ai/g/4790mtH0StqlcoPo6nWNag/0

## Additional generations (2026-07-30, same trained model)

Two more calls against the same `custom_model_uri`, to see how the model behaves both
on-brand in a new scene and completely off the brand it was trained on.

### On-brand: new scene, same character

- Prompt: "Kip, the friendly courier fox mascot, surfing on a giant Fizzwright bottle cap across a wave of soda bubbles at a sunny beach. Kip has an ecstatic grin, one paw raised, tail streaming behind in the wind, wearing the same Hopcarry courier satchel. Bright summer lighting, splashing citrus-soda foam, consistent Fizzwright/Hopcarry brand palette and character design."
- Settings: `rendering_speed: QUALITY`, `aspect_ratio: 4x5`, `custom_model_uri` set to the trained model above.
- `response_id`: `pm6QrXy9VTejS0bxaE8sUQ`
- Output file: `kip-surfing-bottlecap-wave.webp` (896x1120)
- Permalink: https://ideogram.ai/g/EPQbSUSXTqiYRuB0L2ka8g/0
- Result: Kip and the Fizzwright brand palette/character design carried over correctly into an entirely new scene (beach/surf) that wasn't in the training set — this is the actual test of whether training generalizes past the training images, not just reproduces them.

### Off-brand: unrelated prompt, same custom model

- Prompt: "A steampunk astronaut riding a mechanical clockwork horse across a red desert canyon at sunset, brass gears and steam vents on the horse's legs, dust trailing behind, dramatic warm lighting." (no mention of Fizzwright, Hopcarry, or Kip)
- Settings: `rendering_speed: QUALITY`, `aspect_ratio: 4x5`, `custom_model_uri` set to the trained model above.
- `response_id`: `5bJUHosMX7OGx9u_P-mTzg`
- Output file: `off-brand-steampunk-astronaut-test.webp` (896x1120)
- Permalink: https://ideogram.ai/g/g04-YjfpTECGl_mbMFe7nw/0
- Result: worth a human eyeball check, but the useful question this answers is whether a custom model trained on a specific brand/character bleeds its style (color palette, rendering style) into unrelated subjects, or cleanly ignores the training when the prompt doesn't invoke it. Compare this file against a default (non-custom-model) generation of the same prompt if you want to isolate the training's actual influence.
