# Exploration: meodai/heerich

**Repository**: https://github.com/meodai/heerich
**Package**: `heerich`
**Version**: 0.1.0
**License**: MIT
**Author**: meodai (David Aerne)

---

## What Is It?

`heerich.js` is a compact single-file JavaScript library for building **3D voxel scenes** and rendering them to **SVG** — with zero external dependencies, no WebGL, no Canvas.

It's named after **Erwin Heerich**, a German sculptor known for his precise geometric cardboard works. The library brings that same aesthetic to code: crisp vector output, CSG-style boolean geometry, and a clean API.

---

## Why It Matters

Most 3D engines require WebGL (heavy, GPU-dependent) or Canvas (raster output, not scalable). Heerich occupies a different niche:

- **SVG output** — fully scalable, editable, embeddable in web pages, printable
- **Zero runtime dependencies** — single file, works anywhere
- **Voxel-based** — simple grid model, no mesh complexity
- **CSG operations** — build complex shapes by combining primitives with boolean logic
- **Interactive** — per-face `data-*` attributes enable click/hover on individual voxel faces
- **Generative art-friendly** — procedural placement, coordinate-driven dynamic styles

---

## Architecture

### Single-file library

The entire engine lives in `src/heerich.js`, exporting one class: `Heerich`.

### Voxel Storage

Voxels are stored in a `Map` keyed by **packed 30-bit integers** (the `_k` function), encoding `[x, y, z]` coordinates into a single number for fast lookups. The coordinate space spans **-512 to 511** on each axis.

### Rendering Pipeline

```
Voxel Map
    ↓
Shape generators (box, sphere, line, test function)
    ↓
Face generation (getFaces)
    ↓
Backface culling + neighbor occlusion culling
    ↓
Projection (oblique or perspective)
    ↓
Depth sorting
    ↓
SVG polygon output (toSVG)
```

Face computation is **lazy** — results are cached and only recomputed when voxels change (dirty flag).

---

## Public API

### Constructor

```js
import { Heerich } from 'heerich';

const scene = new Heerich({
  tileSize: 20,           // voxel tile size in pixels
  style: { fill: '#fff', stroke: '#000' },  // default face style
  camera: { type: 'oblique', angle: 45 }    // camera config
});
```

### Camera Modes

```js
scene.setCamera({ type: 'oblique', angle: 45 });      // pixel-art aesthetic
scene.setCamera({ type: 'perspective', distance: 10 }); // vanishing-point 3D
```

### Adding Geometry

```js
// Rectangular block
scene.addBox({ from: [0,0,0], to: [3,3,3], style: { fill: 'red' } });

// Sphere
scene.addSphere({ center: [5,0,5], radius: 3 });

// Line with brush
scene.addLine({ from: [0,0,0], to: [10,0,10], radius: 1 });

// Procedural — add voxels wherever test returns true
scene.addWhere({ test: ([x,y,z]) => Math.random() > 0.5, bounds: [...] });
```

### Boolean Operations

```js
// CSG-style subtraction
scene.removeBox({ from: [1,1,1], to: [2,2,2] });          // subtract
scene.addBox({ mode: 'intersect', from: [0,0,0], to: [5,5,5] });
// modes: 'union' | 'subtract' | 'intersect' | 'exclude'
```

### Per-Face Styling

```js
scene.addBox({
  from: [0,0,0], to: [4,4,4],
  style: {
    top:    { fill: '#fff' },
    bottom: { fill: '#000' },
    left:   { fill: '#aaa' },
    right:  { fill: '#888' },
    front:  { fill: '#ccc' },
    back:   { fill: '#555' },
  }
});

// Dynamic styles from coordinates
scene.addBox({
  style: ([x, y, z]) => ({
    fill: `hsl(${x * 10}, 70%, 60%)`
  })
});
```

### Rotation

```js
scene.rotate({ axis: 'y', turns: 1 });  // 90° increments
```

### Queries

```js
scene.hasVoxel([x, y, z]);         // boolean
scene.getVoxel([x, y, z]);         // voxel data
scene.getNeighbors([x, y, z]);     // 6 axis-aligned neighbors
scene.forEach((pos, voxel) => {});  // iterate all
```

### Rendering

```js
// Render to SVG string
const svg = scene.toSVG({ padding: 20 });
document.body.innerHTML = svg;

// Stateless rendering (no voxel storage)
const faces = scene.getFacesFrom({ test: fn, bounds: [...] });

// Compute bounding box for custom viewBox
const viewBox = scene.getOptimalViewBox(20);
```

### Serialization

```js
const json = scene.toJSON();
const restored = Heerich.fromJSON(json);
// Note: function-based styles are lost in serialization
```

---

## Installation

```sh
npm install heerich
```

Or via script tag (UMD build):
```html
<script src="heerich.umd.js"></script>
```

Exports both **ESM** and **CommonJS** formats.

---

## Key Internal Algorithms

| Algorithm | Description |
|-----------|-------------|
| Coordinate packing (`_k`) | Encodes `[x,y,z]` → 30-bit integer for O(1) Map lookups |
| `_rot90` | 90° rotation transforms in voxel space |
| Backface culling | Hides faces pointing away from camera (perspective mode) |
| Neighbor occlusion | Hides faces shared between two filled voxels |
| Depth sorting | Painter's algorithm — sorts faces back-to-front for correct overlap |
| Face generation | Two paths: oblique (fixed projection) and perspective (vanishing point) |

---

## Use Cases

- **Generative art / creative coding** — procedural voxel scenes with dynamic color
- **Data visualization** — 3D bar charts, voxel maps rendered as scalable SVG
- **Game UI / icons** — isometric pixel-art style UI elements
- **Print / publication graphics** — SVG output is print-ready at any resolution
- **Interactive diagrams** — per-face `data-*` attributes enable CSS/JS interactivity
- **Architectural / sculptural sketches** — inspired directly by Heerich's cardboard geometry

---

## Demos Included

| File | Description |
|------|-------------|
| `index.html` | Main showcase / documentation site |
| `minesweeper.html` | Interactive Minesweeper game rendered in voxel SVG |
| `hero.js` | Hero scene generator for the project page |

---

## Build

```sh
npm run dev          # Vite dev server
npm run build        # Compile library (UMD + ESM)
npm run build:site   # Build documentation site
```
