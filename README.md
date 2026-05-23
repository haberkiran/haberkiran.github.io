# Portfolio: Technical Living Document

A personal portfolio built with **Astro** and **Three.js**.

## Landing Page

The index page features an isometric 3D wave mesh implemented with custom GLSL shaders.

- **PS1 Aesthetics**: Low-poly geometry with intentional vertex snapping and affine texture warping.
- **Performance**: Rendered as a single Astro island using `client:only="three"`.

## Stack

- **Framework**: Astro (Static Output)
- **3D Engine**: Three.js + GLSL
- **Styling**: Vanilla CSS (Custom Properties)
- **Typography**: Self-hosted Iosevka (Normal & Slab variants)
- **Hosting**: GitHub Pages

## Setup & Development

### Prerequisites

- Node.js (>= 22.12.0)

### Installation

```sh
npm install
```

### Development

```sh
npm run dev
```

### Build

```sh
npm run build
```

## Deployment

This project is configured for GitHub Pages. On push to `main`, a GitHub Action builds the site and deploys the `./dist` directory.
