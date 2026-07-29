# ideogram-ai-toolkit

A Claude Code skill for [Ideogram AI](https://ideogram.ai) image generation — two prompting modes (loose vs. structured), color-palette control, reference-image style extraction, and the difference between remix and edit. Built by [Devkind](https://devkind.com.au).

Ideogram 4 was trained on structured JSON captions, not plain text — this toolkit documents that schema (adapted from the official [ideogram-oss/ideogram4](https://github.com/ideogram-oss/ideogram4) docs, Apache-2.0) and packages practical prompting technique around it, scoped to what the connected Ideogram MCP (`generate_image`, `describe_image`, `remix_image`, `edit_image`) actually supports.

## What's in here

```
skills/ideogram-prompt/
├── SKILL.md                              # the skill itself
└── references/
    ├── json-caption-schema.md            # Ideogram 4's structured caption schema
    └── style-extraction-workflow.md      # describe → extract recipe → apply → generate loop
```

## Install

This is a Claude Code [skill](https://docs.claude.com/en/docs/claude-code/skills). Requires the [Ideogram MCP](https://ideogram.ai/features/mcp/) to be connected (`claude mcp add ideogram --transport http https://mcp.ideogram.ai/mcp`).

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
