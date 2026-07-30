# Examples

This directory is intentionally empty at ship time. There is no worked example here yet
by design, not by omission: per the design spec's Components section, this skill ships
with "one worked example, produced by actually running the pipeline once (empty until
then)." No pattern/texture job has been run through the live Ideogram MCP yet, so there
is nothing real to show, and fabricating one here would risk being mistaken for an actual
result later.

## Adding the first real worked example

The next time this skill is actually run against the live Ideogram MCP for a real
pattern/texture request, save the result here as `<slug>-pattern.md` (or
`<slug>-texture.md`), following the format used by
`skills/moodboard-generator/examples/anchorpoint-moodboard.md`:

1. **The paragraph prompt** — the exact prompt sent to generate the base tile, plus the
   generation parameters used (model call, `aspect_ratio`, `rendering_speed`,
   `style_type` and whether it was set or omitted and why, `custom_model_uri` if any).
2. **The verification result** — per `references/tiling-verification.md`, the 2x2
   verification-render prompt used, and the honest seam-artifact checklist result
   ("no visible seams in this best-effort check," or the specific seam issue found and
   how the prompt was revised). This section goes between the generation section and the
   compositional-deconstruction section, matching this skill's own workflow order.
3. **The compositional deconstruction** — the JSON structure describing what the
   generation actually produced (motif, palette with hex values, repeat type, scale),
   in the same normalized-coordinate style as the moodboard example.
4. **A "what worked" closing note** — a short, honest note on what held up in the result
   (or didn't), in the same spirit as the moodboard example's closing section.

Do not invent a fictional worked example to fill this file in the meantime. Leave this
README as the only file in this directory until a real job has actually been run.
