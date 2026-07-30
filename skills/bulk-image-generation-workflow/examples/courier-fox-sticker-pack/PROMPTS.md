# Courier Fox Sticker Pack — Prompts

Locked `style_description` (byte-for-byte identical across all 10 prompts, per
`references/variation-strategy.md`):

```json
{
  "aesthetics": "friendly, energetic, modern mascot design",
  "lighting": "even flat lighting, no cast shadows",
  "medium": "graphic_design",
  "art_style": "flat vector illustration, bold clean outlines, simplified geometric shapes",
  "color_palette": ["#FF6B35", "#FFFFFF", "#2B2D42", "#F7C59F"]
}
```

Locked background (also byte-for-byte identical across all 10 prompts):
`"A solid #FFFFFF background with no other elements."`

Variation axis: fox pose + one held prop, per prompt (`compositional_deconstruction.elements[0].desc`).

Below is the literal 10-item `prompts` array submitted to `generate_images_bulk`
(`aspect_ratio: "1x1"`), each entry the exact JSON-in-string sent:

1. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot running forward holding a brown parcel under one arm."}]}}`
2. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot sitting cross-legged, sipping from a to-go coffee cup."}]}}`
3. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot standing and waving one paw in a friendly greeting."}]}}`
4. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot riding a bicycle with a wicker basket full of packages on the front."}]}}`
5. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot holding a clipboard, checking off a delivery with a pencil."}]}}`
6. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot carrying a tall stack of cardboard boxes in both arms."}]}}`
7. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot riding a small delivery scooter with one package strapped to the back."}]}}`
8. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot giving a thumbs up with a package tucked under the other arm."}]}}`
9. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot knocking on a door while holding a package in the other paw."}]}}`
10. `{"style_description":{"aesthetics":"friendly, energetic, modern mascot design","lighting":"even flat lighting, no cast shadows","medium":"graphic_design","art_style":"flat vector illustration, bold clean outlines, simplified geometric shapes","color_palette":["#FF6B35","#FFFFFF","#2B2D42","#F7C59F"]},"compositional_deconstruction":{"background":"A solid #FFFFFF background with no other elements.","elements":[{"type":"obj","desc":"A cartoon fox courier mascot jumping mid-air excitedly while clutching a small parcel."}]}}`

`aspect_ratio`: `"1x1"` (single value applied to the whole batch call, not per-prompt).
