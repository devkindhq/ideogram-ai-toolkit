# Ideogram 4 JSON caption schema

Source: `ideogram-oss/ideogram4` repo, `docs/prompting.md` (official, open-source model docs — verified by fetching the file directly from GitHub, not from the unverifiable `ideogram.ai/skill/*.md` install link some blog posts reference).

This schema describes what Ideogram 4 was actually trained on: structured JSON captions, not plain text. The model's own inference pipeline runs plain-text prompts through a "magic prompt" LLM step that expands them into this JSON shape before generation. Sending the JSON directly skips that reinterpretation — what you write is what renders.

**Caveat for this skill:** this schema is documented against Ideogram's raw Python `pipe()` / hosted API. The `mcp__ideogram__generate_image` tool available in this session takes a plain `prompt` string with no `magic_prompt` toggle (unlike `edit_image`, which does expose `magic_prompt: AUTO/ON/OFF`). It is unconfirmed whether pasting this JSON verbatim into `generate_image`'s `prompt` field reliably bypasses magic prompt the same way the raw API does. Treat JSON-in-prompt as an experiment worth trying when precision matters, not a guaranteed mechanism — compare the output against a well-written prose prompt and keep whichever wins.

## Top-level fields

1. `high_level_description` — optional string, strongly recommended. One or two sentences summarizing the whole image.
2. `style_description` — optional object. Controls style, lighting, medium, palette.
3. `compositional_deconstruction` — **required** object. Background + per-element spatial layout.

## `style_description`

Must contain **exactly one** of `photo` (photographic) or `art_style` (illustration/painting/3D/etc). `aesthetics`, `lighting`, `medium` are required alongside it. `color_palette` is optional but if present must come last.

**Key order is strict:**
| Type | Order |
|---|---|
| Photo | `aesthetics`, `lighting`, `photo`, `medium`, `color_palette` |
| Non-photo | `aesthetics`, `lighting`, `medium`, `art_style`, `color_palette` |

| Field | Type | Notes |
|---|---|---|
| `aesthetics` | string | e.g. "moody, cinematic, desaturated" |
| `lighting` | string | e.g. "golden hour, rim light, dramatic shadows" |
| `photo` | string | camera/lens, e.g. "35mm, f/1.4, bokeh" — use OR `art_style`, not both |
| `medium` | string | `"photograph"`, `"illustration"`, `"3d_render"`, `"painting"`, `"graphic_design"`, etc |
| `art_style` | string | e.g. "flat vector illustration, bold outlines" |
| `color_palette` | list[str] | up to 16 uppercase `#RRGGBB` hex codes, steers dominant colors |

## `compositional_deconstruction`

`background` (string, required, comes first) + `elements` (list, required).

Each element has a fixed key order by `type`:
| Type | Order |
|---|---|
| `"obj"` | `type`, `bbox`, `desc`, `color_palette` |
| `"text"` | `type`, `bbox`, `text`, `desc`, `color_palette` |

- `bbox`: `[y_min, x_min, y_max, x_max]`, normalized 0–1000, origin top-left. Optional.
- `desc`: detailed description of the element.
- `text`: literal text to render (only for `type: "text"`).
- `color_palette`: optional per-element palette, up to 5 hex entries.

Hex colors must be uppercase `#RRGGBB` (no shorthand, no lowercase). Split multi-line copy into one `text` element per line rather than one element with embedded line breaks — it renders more reliably. Give each text element's `bbox` its own clear space; overlapping bboxes between elements tend to render as garbled or overlapping text.

For lighting control, a `color_palette` reads better with both a highlight and a shadow hex included, not just the dominant color — and if a specific background tone matters, name that hex explicitly rather than leaving it to the `background` prose alone.

## Full worked example

```json
{
  "high_level_description": "A lone sailboat on calm water at sunset.",
  "style_description": {
    "aesthetics": "serene, warm, golden hour",
    "lighting": "golden hour backlighting, warm atmospheric haze",
    "photo": "wide angle, f/8, long exposure",
    "medium": "photograph",
    "color_palette": ["#FF6B35", "#F7C59F", "#004E89", "#1A659E", "#2B2D42"]
  },
  "compositional_deconstruction": {
    "background": "A calm ocean stretching to a low horizon, sky washed in orange and pink with thin wisps of cloud.",
    "elements": [
      {"type": "obj", "desc": "A single sailboat with a white triangular sail, silhouetted against the setting sun."}
    ]
  }
}
```

```json
{
  "high_level_description": "A clean, modern business card layout for a tech company.",
  "style_description": {
    "aesthetics": "minimal, professional, geometric",
    "lighting": "even, diffuse studio lighting",
    "medium": "graphic_design",
    "art_style": "flat vector design, generous whitespace, sans-serif typography",
    "color_palette": ["#FFFFFF", "#F0F0F0", "#333333", "#0066FF", "#00CC88"]
  },
  "compositional_deconstruction": {
    "background": "A solid off-white card surface with subtle paper texture.",
    "elements": [
      {"type": "text", "text": "ACME TECH", "desc": "Bold dark grey sans-serif company name across the upper third of the card."},
      {"type": "text", "text": "hello@acme.tech", "desc": "Small blue sans-serif contact email near the bottom of the card."}
    ]
  }
}
```

## Safety filter

NSFW prompts are blocked and return a blocked-image placeholder. False-positive rate is higher for plain-text prompts than for structured JSON — another reason precision-JSON is worth trying when a prompt keeps getting incorrectly blocked.
