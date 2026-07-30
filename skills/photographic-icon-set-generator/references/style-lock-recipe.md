# Style-Lock Recipe for Photographic Icon Sets

## Why a Style Block, Not Per-Icon Description

The mechanics of icon-set consistency depend on understanding how Ideogram's default v4 pipeline treats batched prompts. The `style_type`, `negative_prompt`, `magic_prompt_option`, and `seed` parameters are ignored on the default pipeline, which means cross-icon consistency cannot be controlled through generation parameters. It can only be achieved by repeating the exact same descriptive text across every `prompts[]` entry.

The mechanical failure mode this guards against is describing the material, lighting, or finish slightly differently for icon #3 than for icon #1 — a different color word, a looser lighting phrase, or a subtly different material name. Each icon in `prompts[]` is generated independently from the exact words in that one string. Ideogram has no memory of "the icon I rendered earlier in this batch." If icon #1's prompt says "warm key light from the upper-left" and icon #3's prompt says "soft light from the side," the two rendered icons will visually drift even though each prompt reads fine in isolation. The style block is the lever that prevents this drift: by locking the identical descriptive text verbatim across all entries, every icon inherits the same material, lighting, camera angle, and ground treatment, creating a cohesive set.

## Five Fixed Axes Checklist

### 1. Render/Material Technique
The literal rendering method — 3D-rendered, claymation/clay-sculpt, hand-painted/gouache, or isometric-with-material-texture — named explicitly, not implied by adjectives like "cute" or "modern." State the specific surface quality (matte plastic, glossy, clay sheen, brushstroke texture) so every icon in the set appears to be made from the same material and technique, not an assortment of different rendering styles.

### 2. Lighting Setup
A specific light description (e.g., "soft three-point studio lighting, warm key light from upper-left, soft fill, subtle rim light") stated once and repeated verbatim. Unspecified or loose lighting language like "nice lighting" or "bright" is the primary invitation for Ideogram to invent a different mood, key-light direction, or fill ratio per icon. Name the source direction, intensity relative adjectives (soft, harsh, dramatic), and the fill strategy.

### 3. Camera Angle/Perspective
An exact viewing angle (e.g., "three-quarter view from slightly above, 30-degree camera tilt" or "straight-on isometric projection") named once so every icon in the set reads from the same virtual camera. Vague phrasing like "normal view" or inconsistent angle descriptions across prompts causes icons to appear tilted differently or shot from different heights.

### 4. Background/Ground Treatment
The literal ground or backdrop the subject sits on or against (e.g., "seated on a matte pastel-blue rounded pedestal, soft contact shadow, plain light-gray backdrop"). An unspecified background is one of the most common sources of visible inconsistency across a batch — icons end up against different surface types, colors, or depth cues. Lock the backdrop, ground material, and shadow treatment in the style block and repeat it verbatim.

### 5. Color Story
The palette's role-based description (primary material color, accent color, backdrop color), stated as named roles rather than an unweighted adjective list. Match the guidance from `ideogram-prompt` to prefer exact hex or named-role color direction over vague color adjectives. Instead of "bright, colorful, vibrant," use "primary material color warm coral-orange, accent details in cream-white, pedestal in dusty pastel blue." Consistency in named color roles ensures the entire batch reads as a single palette, not a mix of different color moods.

## Worked Examples

### Style Block: 3D-Rendered

Rendered as a glossy 3D toy-figure icon with soft matte-plastic material and subtle ambient-occlusion shading; lit with soft three-point studio lighting, a warm key light from the upper-left, gentle fill, and a subtle cool rim light along the right edge; shown from a three-quarter view tilted slightly downward, as if photographed on a small turntable; seated on a rounded pastel-blue pedestal with a soft contact shadow beneath it, set against a plain, seamless light-gray studio backdrop; primary material color warm coral-orange, accent details in cream-white, pedestal in dusty pastel blue.

### Style Block: Claymation

Rendered as a hand-sculpted claymation/stop-motion figure with visible fingerprint and tool-mark texture in the clay surface and a soft matte, slightly waxy clay sheen; lit with warm, diffused stop-motion set lighting, a soft key light from the front-left and gentle fill, minimal harsh shadow; shown from a gentle three-quarter angle at eye level, as if photographed on a miniature tabletop set; resting on a small rough-textured clay base with a soft grounded shadow, set against a plain warm off-white studio backdrop; primary clay color muted terracotta, accent details in cream and moss-green, base in warm gray clay.

### Style Block: Hand-Painted

Rendered as a hand-painted gouache illustration with visible brushstroke texture, soft paper grain showing through thin paint layers, and gently imperfect, organic edges; lit with soft, even natural-light-style illumination with minimal cast shadow, consistent with a flat painted study rather than a photographed scene; shown from a straight-on front three-quarter view; painted directly onto a warm cream paper ground with a soft painted shadow beneath the subject, no hard background edge; primary pigment color muted sage-green, accent details in warm ochre and soft rust-red, paper ground warm cream.

### Style Block: Isometric-with-Material-Texture

Rendered as a detailed isometric 3D object with realistic material textures (brushed metal, matte plastic, or fabric weave as appropriate to the subject) rather than flat isometric color blocks; lit with clean, neutral studio lighting from directly above and slightly to the left, soft shadows, no dramatic contrast; shown in true isometric projection (equal 120-degree axes, no perspective convergence); floating slightly above a plain rounded platform tile in muted slate-gray with a soft drop shadow, set against a plain white background; primary material color deep navy-blue, accent details in warm brass/gold, platform tile in muted slate-gray.

## Subject-Block Composition

Each per-icon prompt follows the structure: `[style block, identical] + [subject block, varies] + [composition block: isolated single subject, centered, consistent margin, consistent ground/shadow treatment]`.

The subject block names the subject concretely — "a simple rounded house/home icon" rather than vague "home" — and keeps it short. The style block is already carrying the bulk of description weight, so an overloaded subject block risks pulling the icon away from the locked recipe and the achieved consistency. Let the style block do the heavy work; the subject block simply names what object is being rendered in that locked style.

## Self-Review Checklist

- [ ] Does Step 1 explain the mechanical failure mode and why style blocks solve it?
- [ ] Do all five axes (render/material, lighting, camera, ground, color) appear in the checklist with concise guidance?
- [ ] Are all four worked examples present and complete as full, copy-pasteable paragraphs with no bracketed placeholders?
- [ ] Do the four examples cover 3D-rendered, claymation, hand-painted, and isometric-with-material-texture?
- [ ] Does the closing note explain the `[style block] + [subject block] + [composition block]` structure and subject-naming guidance?
- [ ] Are all example paragraphs written as load-bearing prose, not templates to rewrite?
