# Platform Dimension Map

## Overview

This is the one reference surface with no equivalent in any sibling skill — none of the five existing skills reconcile Ideogram's fixed `aspect_ratio` and `resolution` enums against external, real-world pixel specs, because they all render at a fixed square or portrait ratio for a design-sheet or moodboard use case. This document provides the authoritative mapping between platform-native dimensions and Ideogram's supported aspect ratios.

The verified `aspect_ratio` enum values supported by `mcp__ideogram__generate_image` are:

```
1x3, 3x1, 1x2, 2x1, 9x16, 16x9, 10x16, 16x10, 2x3, 3x2, 3x4, 4x3, 4x5, 5x4, 1x1
```

Throughout this file, **ratios are computed as `width/height`** for comparison purposes. For example, Instagram Feed (portrait) is 1080×1350, so the ratio is 1080/1350 = 0.8.

---

## Platform Dimensions Reference Table

| Placement | Native pixel spec | Native ratio | Nearest `aspect_ratio` | Fit | Exact `resolution` match | Safe zone |
|---|---|---|---|---|---|---|
| Instagram Feed (square) | 1080×1080 | 1:1 (1.0) | `1x1` | Exact | `1024x1024` or `2048x2048` | None significant; avoid extreme edges for crop safety on older grid views. |
| Instagram Feed (portrait) | 1080×1350 | 4:5 (0.8) | `4x5` | Exact | `896x1120` | None significant. |
| Instagram Story / Reels | 1080×1920 | 9:16 (0.5625) | `9x16` | Exact | `1440x2560` | Top ~14% and bottom ~14% of the frame are reserved for the profile handle, reply bar, and sticker tray — keep headline/CTA text out of those bands. |
| Facebook Feed | 1200×630 | ~1.905 | `2x1` (2.0) | Near-fit (narrower than native) | none exact | Bottom-left corner may be covered by the page name/sponsored label on some placements. |
| Facebook Cover Photo | 820×312 | ~2.628 | `3x1` (3.0) | Near-fit | none exact | Bottom ~20% is covered by the profile-photo overlay and page name on desktop and mobile. |
| LinkedIn Feed Post | 1200×627 | ~1.914 | `2x1` (2.0) | Near-fit | none exact | Bottom strip may be covered by the "Promoted" label on sponsored posts. |
| LinkedIn Personal/Company Banner | 1584×396 (personal) / 1128×191 (company) | 4:1 / ~5.9 | `3x1` (3.0) | Mismatch — no supported ratio reaches 4:1+ | none exact | Center-left ~280px is covered by the profile photo on personal banners; keep headline text right-of-center. |
| X (Twitter) Post | 1200×675 | 16:9 (1.778) | `16x9` | Exact | `1280x720` | None significant. |
| X (Twitter) Header | 1500×500 | 3:1 (3.0) | `3x1` | Exact | none exact at this exact pixel count, but ratio is exact | Bottom-left is covered by the profile photo and name overlay — keep headline text upper-right or center. |
| YouTube Thumbnail | 1280×720 | 16:9 (1.778) | `16x9` | Exact | `1280x720` (pixel-exact) | Bottom-right ~60×20px is covered by the video-duration badge — keep CTA/logo elements clear of that corner. |
| Pinterest Pin (standard) | 1000×1500 | 2:3 (0.667) | `2x3` | Exact | `1664x2496` | Bottom ~15% may be covered by the save-button overlay and pin title on some surfaces. |
| Generic Display — Medium Rectangle | 300×250 | 1.2 | `5x4` (1.25) | Near-fit | none exact | None significant; text must remain legible at this small native size — favor larger type than a full-size social placement would need. |
| Generic Display — Leaderboard | 728×90 | ~8.09 | `3x1` (3.0) | Severe mismatch | none exact | Recommend the user crop/pad this placement with their own tooling rather than expect a direct render to look right — the gap between 3:1 and 8:1 is too wide to disclose-and-ship as a reasonable approximation. |
| Generic Display — Skyscraper | 160×600 | ~0.267 | `1x3` (0.333) | Near-fit | none exact | None significant; extremely narrow canvas — favor a single short headline, no subhead, CTA only if space allows. |

---

## How to Read the Fit Column

The **Fit** column classifies how well Ideogram's nearest supported `aspect_ratio` matches the platform's native dimensions:

- **Exact**: The nearest `aspect_ratio` enum value equals the platform's native ratio (rounding to 2 decimal places). The generated image will fill the platform's native space without meaningful cropping or padding. Examples: Instagram Feed (square), Instagram Story/Reels, X Post, YouTube Thumbnail.

- **Near-fit**: The nearest enum value is within roughly 0.4 of the native ratio — close enough to disclose as a minor approximation but not refuse. The image will be slightly cropped or require light padding depending on platform display logic. Examples: Facebook Feed, Facebook Cover Photo, LinkedIn Feed Post.

- **Mismatch**: The nearest enum value shares the same general orientation as the platform's native ratio (e.g., both landscape, both elongated) but the native ratio is substantially wider or narrower than what the nearest enum can reach — a gap beyond a minor approximation but not extreme enough to require external handling. The shape is in the right neighborhood but does not match the native ratio's magnitude. Example: LinkedIn Personal/Company Banner (native 4:1+ vs. nearest `3x1` at 3.0).

- **Severe mismatch**: The gap between the nearest enum value and the native ratio is wide enough that the skill should recommend the user handle that placement outside this skill entirely rather than disclose-and-ship a poor approximation. Example: Generic Display — Leaderboard (native 8.09 vs. nearest `3x1` at 3.0 is a 2.7x gap).

This **Fit** classification is what the "Resolve platform dimensions" workflow step in `SKILL.md` uses to decide whether to disclose-and-proceed or recommend an external step.

---

## Resolution Enum Note

The `aspect_ratio` parameter is the **primary lever** for platform-dimension resolution. The `resolution` enum is only worth setting explicitly when an exact pixel-dimension match exists in the enum (as documented per-row in the table above); otherwise, passing `aspect_ratio` alone and letting Ideogram pick a default resolution for that ratio is sufficient and simpler. This reduces request complexity and avoids over-specifying constraints.

---

## Verification

All native ratio values recomputed from native pixel specs and confirmed:
- Instagram Feed (square): 1080/1080 = 1.0 ✓
- Instagram Feed (portrait): 1080/1350 = 0.8 ✓
- Instagram Story/Reels: 1080/1920 = 0.5625 ✓
- Facebook Feed: 1200/630 = 1.905 ✓
- Facebook Cover Photo: 820/312 = 2.628 ✓
- LinkedIn Feed Post: 1200/627 = 1.914 ✓
- LinkedIn Personal Banner: 1584/396 = 4.0 (personal) / 1128/191 = 5.907 (company) ✓
- X Post: 1200/675 = 1.778 ✓
- X Header: 1500/500 = 3.0 ✓
- YouTube Thumbnail: 1280/720 = 1.778 ✓
- Pinterest Pin (standard): 1000/1500 = 0.667 ✓
- Generic Display Medium Rectangle: 300/250 = 1.2 ✓
- Generic Display Leaderboard: 728/90 = 8.09 ✓
- Generic Display Skyscraper: 160/600 = 0.267 ✓

All nearest `aspect_ratio` selections confirmed by distance comparison. LinkedIn Personal/Company Banner row explicitly states no supported ratio reaches 4:1+, as the highest supported ratio in the enum is `3x1` (3.0).
