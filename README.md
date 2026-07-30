# ideogram-ai-toolkit

A set of Claude Code skills for [Ideogram AI](https://ideogram.ai) image generation — general prompting technique (loose vs. structured, color-palette control, reference-image style extraction, remix vs. edit), plus purpose-built skills covering brand systems, characters, marketing creative, icon sets, patterns/textures, bulk generation, custom model training, collections, and post-processing (background removal, upscaling). Built by [Devkind](https://devkind.com.au).

Ideogram 4 was trained on structured JSON captions, not plain text — this toolkit documents that schema (adapted from the official [ideogram-oss/ideogram4](https://github.com/ideogram-oss/ideogram4) docs, Apache-2.0) and packages practical prompting technique around it, scoped to what the connected Ideogram MCP actually supports.

## What's in here

```
skills/ideogram-prompt/                       # general Ideogram prompting technique
├── SKILL.md
├── references/                               # JSON caption schema, style-extraction workflow
└── examples/                                 # pointer to the root examples/ set (see below)
skills/logo-prompting/                        # logo/brand-mark prompt writing, Ideogram-first
├── SKILL.md
├── references/                               # anti-slop discipline, brand visual-layer reading, intake checklist
└── examples/                                 # accumulated real logo-prompt directions by style family
skills/moodboard-generator/                   # pre-logo 3x3 brand-exploration board (loose adjectives, unlocked palette)
├── SKILL.md
├── references/                               # panel anatomy, composition-spec JSON schema, anti-slop discipline
└── examples/                                 # a worked prompt + full JSON breakdown
skills/brand-identity-sheet/                  # whole brand system in one generated image (wordmark, icons, buttons)
├── SKILL.md
├── references/                               # panel anatomy, composition-spec JSON schema, anti-slop discipline
├── examples/                                 # a worked prompt + full JSON breakdown
└── evals/                                    # eval scenarios for this skill
skills/character-model-sheet/                 # multi-panel character turnaround sheet (mascot, game/animation character)
├── SKILL.md
├── references/                               # panel anatomy, composition-spec JSON schema, anti-slop discipline
├── examples/                                 # a worked prompt + full JSON breakdown
└── evals/                                    # eval scenarios for this skill
skills/marketing-ui-mockup-hero-generator/    # landing-page hero shots, device mockups, dashboard screenshots
├── SKILL.md
├── references/
├── examples/                                 # worked hero/device-mockup/dashboard examples
└── evals/
skills/social-marketing-asset-generator/      # finished social/ad creative with copy baked into the pixels
├── SKILL.md
├── references/                               # composition-spec format, anti-slop discipline, platform dimension map, typography-baking discipline
├── examples/                                 # populated as worked jobs are run — see examples/STATUS.md
└── evals/
skills/photographic-icon-set-generator/       # consistent 3D/claymation/textured icon packs sharing a locked style recipe
├── SKILL.md
├── references/                               # style-lock recipe, anti-slop discipline, set-consistency workflow
├── examples/                                 # a worked claymation icon set
└── evals/
skills/seamless-pattern-texture-generator/    # seamless/tileable patterns and material textures
├── SKILL.md
├── references/                               # tile-prompt recipe, tiling verification, anti-slop discipline, composition-spec format
├── examples/                                 # populated once a real job is run — see examples/README.md
└── evals/
skills/bulk-image-generation-workflow/        # one locked caption → N on-brief variations, batch-submitted and culled
├── SKILL.md
├── references/                               # variation strategy, review/culling guide
├── examples/                                 # a worked 10-prompt sticker-pack batch, shortlisted to 6
└── evals/
skills/custom-model-training/                 # train a custom Ideogram model on a folder of reference images
├── SKILL.md
├── references/
├── examples/                                 # a worked training set + proof generation
└── evals/
skills/collections-management/                # create/browse/rename/delete Ideogram collections, file images into them
├── SKILL.md
├── references/
├── examples/                                 # a worked logo collection
└── evals/
skills/remove-background-workflow/            # strip an image's background to a transparent PNG
├── SKILL.md
├── references/                               # background-removal patterns
├── examples/                                 # a worked before/after
└── evals/
skills/upscale-image-workflow/                # raise the resolution of one specific Ideogram image
├── SKILL.md
├── references/                               # upscale settings
├── examples/                                 # a worked upscale
└── evals/
examples/                                     # generated showcase assets + the exact prompts used
```

See [`examples/`](examples/README.md) for the full write-up of the root showcase set — each image below links to its exact prompt.

<table>
  <tr>
    <td align="center" width="33%">
      <a href="examples/README.md#1-editorial-product-photo--precise-color_palette-control">
        <img src="examples/images/01-editorial-product-photo.png" width="200"><br>
        Editorial product photo
      </a>
    </td>
    <td align="center" width="33%">
      <a href="examples/README.md#2-vintage-sci-fi-poster--art_style--in-image-text-via-bounding-boxes">
        <img src="examples/images/02-vintage-scifi-poster.png" width="200"><br>
        Vintage sci-fi poster
      </a>
    </td>
    <td align="center" width="33%">
      <a href="examples/README.md#3-packaging-label--graphic_design-medium-with-multiple-text-elements">
        <img src="examples/images/03-packaging-label.png" width="200"><br>
        Packaging label
      </a>
    </td>
  </tr>
  <tr>
    <td align="center" width="33%">
      <a href="examples/README.md#4-abstract-3d-icon--3d_render-medium">
        <img src="examples/images/04-abstract-3d-icon.png" width="200"><br>
        Abstract 3D icon
      </a>
    </td>
    <td align="center" width="33%">
      <a href="examples/README.md#5-style-extraction--reference-image-to-new-subject">
        <img src="examples/images/05a-style-extraction-reference.png" width="200"><br>
        Style extraction (reference)
      </a>
    </td>
    <td align="center" width="33%">
      <a href="examples/README.md#5-style-extraction--reference-image-to-new-subject">
        <img src="examples/images/05b-style-extraction-applied.png" width="200"><br>
        Style extraction (applied)
      </a>
    </td>
  </tr>
  <tr>
    <td align="center" width="33%">
      <a href="examples/README.md#6-brand-identity-moodboard--33-panel-grid">
        <img src="examples/images/06-moodboard-anchorpoint.png" width="200"><br>
        Brand identity moodboard
      </a>
    </td>
    <td align="center" width="33%">
      <a href="skills/brand-identity-sheet/examples/fizzwright-identity-sheet.md">
        <img src="skills/brand-identity-sheet/examples/images/fizzwright-identity-sheet.webp" width="200"><br>
        Brand identity sheet (Fizzwright)
      </a>
    </td>
    <td align="center" width="33%">
      <a href="skills/character-model-sheet/examples/kip-hopcarry-character-sheet.md">
        <img src="skills/character-model-sheet/examples/images/kip-hopcarry-character-sheet.webp" width="200"><br>
        Character model sheet (Kip)
      </a>
    </td>
  </tr>
  <tr>
    <td align="center" width="33%">
      <a href="skills/marketing-ui-mockup-hero-generator/examples/">
        <img src="skills/marketing-ui-mockup-hero-generator/examples/images/technauts-landing-hero.webp" width="200"><br>
        Landing-page hero (Technauts)
      </a>
    </td>
    <td align="center" width="33%">
      <a href="skills/marketing-ui-mockup-hero-generator/examples/">
        <img src="skills/marketing-ui-mockup-hero-generator/examples/images/voicehive-device-mockup.webp" width="200"><br>
        Device mockup (VoiceHive)
      </a>
    </td>
    <td align="center" width="33%">
      <a href="skills/photographic-icon-set-generator/examples/claymation-recipe-app-icons/">
        <img src="skills/photographic-icon-set-generator/examples/claymation-recipe-app-icons/icon-home.png" width="200"><br>
        Claymation icon set
      </a>
    </td>
  </tr>
  <tr>
    <td align="center" width="33%">
      <a href="skills/bulk-image-generation-workflow/examples/courier-fox-sticker-pack/">
        <img src="skills/bulk-image-generation-workflow/examples/courier-fox-sticker-pack/images/courier-fox-01-running-holding-parcel.webp" width="200"><br>
        Bulk sticker pack (courier fox)
      </a>
    </td>
    <td align="center" width="33%">
      <a href="skills/custom-model-training/examples/fizzwright-hopcarry-training-set/">
        <img src="skills/custom-model-training/examples/fizzwright-hopcarry-training-set/kip-surfing-bottlecap-wave.webp" width="200"><br>
        Custom model output (Kip)
      </a>
    </td>
    <td align="center" width="33%">
      <a href="skills/remove-background-workflow/examples/logo-background-removal/">
        <img src="skills/remove-background-workflow/examples/logo-background-removal/anchorpoint-logo-1-transparent.png" width="200"><br>
        Background removal (Anchorpoint)
      </a>
    </td>
  </tr>
  <tr>
    <td align="center" width="33%">
      <a href="skills/logo-prompting/examples/fleetline-four-directions.md">
        <img src="skills/logo-prompting/examples/images/fleetline-01-wordmark.webp" width="200"><br>
        Logo, four directions (Fleetline)
      </a>
    </td>
  </tr>
</table>

## Install

This is a set of Claude Code [skills](https://docs.claude.com/en/docs/claude-code/skills). Requires the [Ideogram MCP](https://ideogram.ai/features/mcp/) to be connected (`claude mcp add ideogram --transport http https://mcp.ideogram.ai/mcp`).

Each skill installs independently — swap `ideogram-prompt` below for any of the 14 skills under `skills/` as needed.

**Via the [skills CLI](https://www.skills.sh)** (reads straight from this GitHub repo, no unreviewed single-URL installer):
```bash
npx skills add https://github.com/devkindhq/ideogram-ai-toolkit/tree/main/skills/ideogram-prompt
# or, from a project root, target it explicitly:
npx skills add devkindhq/ideogram-ai-toolkit -s ideogram-prompt
```
Add `-g` to install globally (available in every session) instead of just the current project.

**Project-scoped, manual** (checked into a repo, shared with your team):
```bash
git clone https://github.com/devkindhq/ideogram-ai-toolkit.git /tmp/ideogram-ai-toolkit
cp -r /tmp/ideogram-ai-toolkit/skills/ideogram-prompt <your-project>/.claude/skills/ideogram-prompt
```

**Personal, manual** (available in every session):
```bash
git clone https://github.com/devkindhq/ideogram-ai-toolkit.git ~/ideogram-ai-toolkit
ln -s ~/ideogram-ai-toolkit/skills/ideogram-prompt ~/.claude/skills/ideogram-prompt
```

We deliberately don't ship a one-line `curl ... | skill.md` installer — pulling an unreviewed instruction file straight into `~/.claude/skills/` from a single URL is a real prompt-injection / supply-chain risk for an AI agent, and we'd rather you read the skill before trusting it. Clone it, read `SKILL.md`, then symlink or copy it in.

## Why this exists

Some public prompting guides for Ideogram reference a one-line curl installer for a Claude Code skill hosted at a single URL outside GitHub. That URL wasn't independently verifiable at the time this toolkit was built (returned HTTP 403, no cached or archived copy found), so this repo exists as an open, reviewable alternative: same underlying technique (documented against the real, open-source `ideogram-oss/ideogram4` schema and the actual Ideogram MCP tool surface), but as source you can read before you run it.

## License

Apache License 2.0 — see [LICENSE](LICENSE) and [NOTICE](NOTICE). The JSON caption schema reference is adapted from `ideogram-oss/ideogram4` (Apache-2.0).
