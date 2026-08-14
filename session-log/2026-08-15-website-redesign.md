# Session Log — Website Redesign Ledger

**Date:** 2026-08-15
**Project:** Turab Ali Study Circle — website redesign
**Files touched:** `index.html`, `about.html`, `styles.css`, `assets/coin.png`, `assets/turab-ali.png`
**Stack:** plain HTML + CSS, Three.js (CDN), GSAP + ScrollTrigger (CDN), Python PIL for image prep. No build step, no frameworks.

---

## 1. Design principles used

- **Steve Jobs rule:** if an element does not help a parent understand "this is a serious coaching institute," delete it. No cards, no shadows on boxes, no gradient blobs, no decoration without a job.
- **Editorial light theme:** warm paper background, generous whitespace, one accent family (maroon + gold), serif display type.
- **Single column:** text column max-width ~720–780px, centered. Full-bleed only for the frame.
- **Mobile first:** every section verified at 360px width; hero content must fit the first screen without scrolling.
- **One brand mark, used once:** the emblem appears only as the hero coin. Not repeated in the nav, not shrunk into corners.

## 2. Color palette (CSS variables in `:root`)

| Token | Value | Job |
|---|---|---|
| `--paper` | `#f8f2e3` | warm ivory page background |
| `--ink` | `#241e12` | warm near-black body text |
| `--muted` | `#6e6148` | secondary text |
| `--gold` | `#9c7a28` | rules, frame lines, hover accents |
| `--gold-deep` | `#7c611c` | kickers, small caps labels |
| `--gold-bright` | `#c9a227` | highlights |
| `--red` | `#7e0b34` | deep maroon — headline, frame band, accents |
| `--edge-a` / `--edge-b` | `#b08a3c` / `#8f6d2a` | gold extrusion layers for 3D headline |

Rule: maroon and gold come from the emblem itself, so the page always matches the logo.

## 3. Typography

- **Headlines:** Playfair Display (serif). Fallbacks: New York, Georgia.
- **Body / UI:** Inter. Fallbacks: Gill Sans, system sans.
- **Urdu tagline:** Noto Nastaliq Urdu (Google Fonts), `direction: rtl`, generous `line-height` (~2) because Nastaliq is a tall script.
- **Kicker labels:** small sans, uppercase, `letter-spacing: 0.16–0.26em`, gold, with a hairline rule on each side (flexbox + `::before`/`::after` 1px lines).
- **3D headline trick:** layered `text-shadow` — first layers in dark maroon (`#5f0826`), outer layers in gold (`--edge-a`, `--edge-b`). Maroon fill + gold extrusion = minted/coin-like type. No images, pure CSS.
- **Never:** all-caps body text, cursive fonts for headlines, more than two type families per page (plus one script font for Urdu).

## 4. Logo → coin image pipeline (Python PIL)

Source: a square PNG of the emblem on a white background.

1. **Flood-fill background removal:** flood fill from the four corners (threshold ~30) to select only the outer white background — this protects white details *inside* the emblem. Invert → emblem mask.
2. **Circular crop:** intersect the mask with a centered circle (inset ~2%), feathered 0.8px for a crisp-but-clean edge.
3. **Gold base:** composite the emblem over a solid gold disc (`#C9A45C`) so the coin face has no transparency and no cut corners — any pixel not emblem is gold.
4. **Export** as `assets/coin.png` (square, ~1000px). Bump the `?v=` query param in the HTML whenever the PNG changes, to defeat browser caching.

Why this matters: naive "delete white" or square crops produce cut corners and halos. Flood-fill-from-corners + gold underlay is what makes the texture look minted.

## 5. The 3D coin (Three.js)

CSS 3D was tried first (stacked slices for thickness) and abandoned — edges rendered inconsistently and the back face broke in some browsers. Three.js is the reliable way.

**Geometry:** `THREE.CylinderGeometry(R, R, thickness, 160, 1, false)` — a cylinder is a coin. Rotated `x = π/2` so the flat caps face the viewer.

**Materials (array of 3, in cylinder order):**
1. **Edge (side):** `MeshPhongMaterial`, gold `#c9a45c`, warm specular `#ffeec0` — Phong gives the rim its metallic glint.
2. **Front face (top cap):** `MeshLambertMaterial` with the coin texture. Lambert = diffuse only, no specular blow-out on the artwork.
3. **Back face (bottom cap):** same, but with a *cloned* texture.

**Texture orientation (the fiddly part):** cylinder cap UVs are rotated. Fix with `tex.center.set(0.5, 0.5)` then `tex.rotation = Math.PI / 2` on the front, and the same on the cloned back texture so the logo reads upright when the coin flips. Verify visually — cap winding differs between Three.js versions.

**Lighting recipe (calibrated to "not washed out, not dark"):**
- `AmbientLight 0xfff6e2 @ 0.45` — base warmth
- `HemisphereLight sky 0xfff8e6 / ground 0x8a6d2a @ 0.25` — soft gold bounce
- `DirectionalLight (key) 0xffffff @ 0.5` from upper-right — the highlight that makes it metal
- `DirectionalLight (rim) 0xffd980 @ 0.3` from behind — edge separation
- Face materials get `color: 0xe8e8e8` — a ~91% brightness multiplier that takes the glare off the artwork without dulling the gold rim (rim has no multiplier).

**Camera:** pulled back (`z ≈ 4.3`) so the coin never clips the canvas edge when it tilts.

**Interaction:**
- Drag-to-spin only (`pointerdown/move/up` on the stage, `touch-action: pan-y` so vertical page scroll still works on phones). No rotation on passive mouse move — that was disturbing scroll.
- Idle: gentle sinusoidal sway.
- Tap/click: GSAP spin (`rotationY += 2π`, power ease).
- Fallback: a plain `<img>` behind the canvas if WebGL is unavailable.

## 6. Layout & motion techniques

- **Maroon ink frame:** fixed 1px gold hairline frame inset 12px, plus `box-shadow: 0 0 0 12px var(--red)` — the spread fills the gap between viewport edge and frame with flat maroon, like a book's colored edge. Cheaper and cleaner than an extra element. (8px variant in the mobile media query.)
- **Reveal on scroll:** GSAP ScrollTrigger, `opacity 0 → 1, y 30 → 0`, `power3.out`, once per element. Subtle only.
- **Hero intro:** one GSAP timeline — coin scales in, then kicker → Urdu line → headline → place → button, each overlapping by ~0.6s.
- **Native scrolling:** smooth-scroll libraries (Lenis) were removed — they made the page feel heavy. `scroll-behavior: smooth` in CSS is enough.
- **Reduced motion:** `@media (prefers-reduced-motion: reduce)` disables the ambient animation; JS checks the same flag before running GSAP.
- **Hover that doesn't move things:** links underline in gold; virtue-line rules lengthen (`width` transition); buttons deepen in color. Nothing translates or scales.

## 7. Things tried and rejected (don't redo these)

- Dark navy theme, gradient blobs, card grids — against the brief.
- CSS-only 3D coin with stacked slices — unreliable edges, broken back face.
- Lenis smooth scrolling — disturbed native scroll feel.
- Paper-grain overlay texture — read as "dirty," removed for clean flat color.
- Grayscale/black logo face — user wanted full color on both faces.
- Drop-cap giant initial letter on the dedication — too loud; small kicker with side rules won.
- Two-color mixed sentences (red lead words + black rest) — looked broken; single ink color per line.

## 8. Reusable checklist for the next site

1. Pick palette *from the logo*, define as CSS variables first.
2. Two fonts max (+1 script font if needed), real serif for display.
3. One hero object, made physical (Three.js cylinder = coin; lighting: warm ambient + one key + one rim; Lambert for artwork, Phong for metal).
4. Frame the page, don't decorate it (hairline + flat ink band).
5. Animate entrances once, subtly; never hijack scroll.
6. Verify at 360px width and on a short viewport — first screen must fit.
7. Cache-bust image assets with `?v=N` after every edit.
8. Delete anything that doesn't help a stranger understand the site in 5 seconds.

---

*Logged for future reuse. This file is documentation only — it is not linked from the website and does not affect it.*
