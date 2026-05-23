# Portfolio Site — Design Document
> For use as a coding prompt. Implement exactly as specified unless noted as flexible.

---

## Overview

A personal portfolio site for a high school student / incoming UCSB Computer Engineering student based in San Diego. The site should feel like a living document — purposeful, technically honest, and quietly confident. It is not a resume site. It does not posture. The hero is a simulation; the nav is plain text; the pages are documents.

Primary audiences: internship/hiring managers, university professors, the open source / maker community.

---

## Hosting & Framework

- **Framework**: Astro (static output)
- **Hosting**: GitHub Pages
- **No React**. No Tailwind. Vanilla CSS with custom properties.
- The hero is one Astro island (interactive component, Three.js). All other pages are fully static.
- Iosevka font is self-hosted as woff2, built from source. Do not use Google Fonts or any external font CDN.

---

## Typography

**Font family: Iosevka** — self-hosted, two variants:

| Usage | Variant | Weight |
|---|---|---|
| Site name / your name | Iosevka Slab | 400 |
| Nav links, body, code, all UI text | Iosevka (Normal) | 400 |

- No other typefaces. No fallback to system monospace if avoidable.
- Font size base: `13px` for nav / UI. `12px` for meta / footer text.
- Line height: `1.6` for body copy. `1.4` for nav.
- Letter spacing: `0.01em` on meta text only.

---

## Palette

```css
:root {
  --bg:         #b2b6ba;   /* concrete grey — site background */
  --bg-sidebar: #b2b6ba;   /* sidebar matches, no contrast shift */
  --border:     #8a8e92;   /* sidebar divider, subtle rule */
  --text:       #1a1e22;   /* near-black — name, body copy */
  --text-meta:  #4a5058;   /* muted — sandiego::compe@ucsb, timestamps */
  --link:       #0000ee;   /* hyperlink blue — all nav links, all links */
  --link-visited:#551a8b;  /* visited / hover state — standard browser purple */
  --blue-mesh:  #0000ee;   /* hero mesh color base */
}
```

No additional accent colors. The blue does all the work. Do not introduce gradients, shadows, or decorative color outside the hero canvas.

---

## Layout

### Global

```
[ sidebar 152px ] [ hero / page content flex:1 ]
```

- Fixed sidebar on all pages.
- Sidebar has a `0.5px` border-right in `--border`.
- No top nav, no hamburger menu, no mobile nav drawer (mobile is out of scope for now).
- No footer.

### Sidebar contents (top to bottom)

```
[name]          ← Iosevka Slab, 13px, --text, no underline
                   two lines if needed

[systems]       ← nav links, Iosevka Normal, 13px, --link, underlined
[photography]
[about]
[now]

[sandiego::     ← --text-meta, 10px, bottom of sidebar, margin-top: auto
compe@ucsb]
```

- Nav links use raw anchor tags. No custom styling beyond color and underline.
- Active page link renders in `--link-visited` with no other change.
- `sandiego::compe@ucsb` split across two lines exactly as shown. The `::` is C++ scope resolution — do not change this formatting.

---

## Hero (index page)

### Concept

A full-viewport isometric 3D wave mesh rendered in Three.js. The wave is the entire content of the index page — there is no headline, no tagline, no CTA. The mesh is the only thing on the right side of the sidebar.

### Implementation: Three.js + GLSL

Use Three.js (r128, available in Astro). Implement the wave as a **vertex shader** operating on a `PlaneGeometry` tilted into isometric perspective.

**Geometry**:
- `PlaneGeometry` with ~14 cols × 11 rows segments (adjustable, keep it coarse — low poly is intentional)
- Rotated to approximate isometric view: `rotateX(-Math.PI / 3.5)`, fine-tune for correct iso feel
- Scaled to fill the viewport with slight overflow (mesh bleeds past canvas edges)

**Vertex shader — wave**:
```glsl
uniform float uTime;
attribute float aSnap;

void main() {
  vec3 pos = position;
  float wave = sin(pos.x * 0.22 - uTime * 0.85 + pos.y * 0.16) * 38.0
             + cos(pos.y * 0.18 - uTime * 0.50 + pos.x * 0.20) * 20.0
             + sin((pos.x + pos.y) * 0.13 - uTime * 0.60) * 14.0;
  pos.z += wave;
  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}
```

**PS1 vertex snapping** — apply in vertex shader, snap to world-space grid before projection. Snap resolution: `7.0` units (aggressive — visible stuttering between positions is intentional):
```glsl
float snapRes = 7.0;
pos = floor(pos / snapRes + 0.5) * snapRes;
```
Apply snap *before* wave displacement so the wave itself stutters, not just the base geometry.

**PS1 affine texture warping** — implement in fragment shader. Pass UV coordinates as a `varying` from the vertex shader *without* perspective correction (do not use `gl_Position.w` to correct). This causes the blue fill on each triangle face to swim and warp as the geometry moves, especially on steeply angled faces. The effect should be subtle on near-flat faces and pronounced on steep trough/peak transitions.

```glsl
// vertex shader — pass raw affine UVs
varying vec2 vAffineUV;
void main() {
  // ... wave + snap code ...
  vAffineUV = uv; // NOT divided by w — affine, not perspective correct
}

// fragment shader — use warped UVs for subtle noise/depth lookup
varying vec2 vAffineUV;
void main() {
  // depth-based blue shading using vAffineUV
  float depth = vAffineUV.y;
  vec3 color = mix(vec3(0.0, 0.0, 0.39), vec3(0.18, 0.18, 0.93), depth);
  gl_FragColor = vec4(color, 0.5); // semi-transparent fill
}
```

**Wireframe**: Render the mesh twice — once as `MeshBasicMaterial` with semi-transparent fill (fragment shader above), once as `WireframeGeometry` with `LineBasicMaterial` in `#0000ee` at low opacity (~0.4–0.6, depth-faded).

**Nodes**: Render peak and trough vertices as `Points` with a `PointsMaterial`. Peak nodes (z > 34): full `#0000ee`, size 3. Trough nodes (z < -34): dim blue, size 1.5. Mid nodes: size 1.8, opacity 0.7.

**Background**: `#b2b6ba` — set as Three.js scene background color, not CSS.

**Wave speed**: `uTime` increments at `0.006` per frame (~0.36/sec at 60fps). Do not increase.

**Color banding**: Posterize the depth gradient in the fragment shader to 5 discrete steps rather than a smooth gradient:
```glsl
float bands = 5.0;
depth = floor(depth * bands) / bands;
```

---

## Page structure

### `/systems`
- Lists embedded projects as plain linked text entries
- Each project: name (link), one-line description, tags in `--text-meta`
- No cards, no borders, no icons
- Projects (initial):
  - **Orchid watering system** — automated pump + moisture sensor, SPI, DHT22, Flask dashboard, Pi
  - **Kindle dashboard** — e-ink always-on display, Pi backend, weather + calendar + plant status
  - **ESP32 satellite sensor** *(in progress)* — wireless moisture node, expands orchid system

### `/photography`
- Grid of images, no captions unless hovered
- Hover state: filename in `--text-meta`, monospace, bottom-left of image
- No lightbox for now — images link to full res
- Grid: CSS grid, `auto-fill`, `minmax(220px, 1fr)`, `gap: 4px`

### `/about`
- Plain prose, two to three paragraphs
- No headings, no bullet lists
- Content: Scripps embedded internship, incoming UCSB CompE, hardware interests, current projects
- Ends with a link to `/now`

### `/now`
- Short, datestamped, informal
- What you're currently building, reading, or thinking about
- Updated manually — no CMS, just edit the `.astro` file
- Convention: inspired by nownownow.com

---

## Interaction & Animation

- **Hero only**: Three.js animation loop, continuous, no pause
- **No scroll animations**, no entrance transitions, no page transitions
- **Cursor**: default system cursor everywhere. No custom cursor.
- **Link hover**: color shifts from `--link` to `--link-visited`. No other effect.
- **No hover effects** on photography grid images beyond the filename reveal

---

## What this site is not

- Not a resume. Do not add a resume download link unless explicitly requested later.
- Not a contact form. Email can be added as a plain `mailto:` link in `/about` if wanted.
- Not responsive (mobile out of scope for v1).
- Not dark mode. The grey is the palette. Do not add a theme toggle.

---

## File structure (Astro)

```
src/
  components/
    WaveHero.astro     ← Three.js island, client:only="three"
    Sidebar.astro      ← shared sidebar with nav
  layouts/
    Base.astro         ← html shell, imports Iosevka, sidebar
  pages/
    index.astro        ← WaveHero only
    systems.astro
    photography.astro
    about.astro
    now.astro
  styles/
    global.css         ← CSS custom properties, resets, base type
public/
  fonts/
    iosevka.woff2
    iosevka-slab.woff2
  photos/              ← photography assets
```

---

## Open questions (do not implement yet)

- Your actual name and GitHub URL
- Iosevka build plan (stylistic set preferences for `@`, `::`, `a`, `i`)
- Whether `/photography` gets its own sub-navigation or stays flat
- Mouse parallax on hero (subtle iso perspective shift on mousemove) — shelved for v2
- ESP32 satellite sensor integration into `/systems` live data feed — shelved for later
- Moving nodes as nav on `/systems` page specifically — shelved for v2
