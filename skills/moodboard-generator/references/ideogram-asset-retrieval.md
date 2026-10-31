# Retrieving a generated Ideogram asset

Everything here is directly observed behaviour from one real run (a brand moodboard,
2026-09-21), not a documented Ideogram API guarantee. If a later run sees different
behaviour, correct this file rather than assuming that run was anomalous.

**`permalink_url` is not an image.** `generate_image` and `remix_image` return a response
containing `permalink_url`, and that URL resolves to an HTML page. Do not save it as an
image file and do not hand it to anything expecting image bytes.

**The real asset URL comes from `list_images`.** Call `mcp__ideogram__list_images` and read
the `url` field on the matching entry. It is shaped:

```
https://ideogram.ai/assets/image/balanced/response/<response_id>@2k
```

`list_images` took no `limit` argument in the observed run (it was rejected as an
unexpected keyword) and returned roughly 69,000 characters, which overflows the tool
output cap and spills to a tool-results file. Filter by `response_id` rather than reading
it whole:

```bash
jq -r '.assets[] | select(.response_id=="A" or .response_id=="B") | "\(.response_id) \(.url)"' <tool-results-file>
```

**The file is WebP whatever you name it.** Save it with a `.webp` extension honestly, then
convert before viewing or handing off:

```bash
sips -s format png board.webp --out board.png
```

Skipping the conversion means the image cannot be viewed, which means the
post-generation checklist cannot be run, which means the board gets written up on the
strength of its prompt alone. That is the failure this file exists to prevent.
