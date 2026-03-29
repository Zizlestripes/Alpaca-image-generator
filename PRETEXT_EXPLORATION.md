# Exploration: chenglou/pretext

**Repository**: https://github.com/chenglou/pretext
**Package**: `@chenglou/pretext`
**Current version**: 0.0.2 (released 2026-03-28)
**License**: MIT

---

## What Is It?

`pretext` is a pure JavaScript/TypeScript library for measuring and laying out multiline text **without touching the DOM**. It side-steps expensive browser operations like `getBoundingClientRect` and `offsetHeight` (which trigger layout reflow) by implementing its own text measurement using the browser's canvas font engine.

It supports all languages, emojis, and mixed bidirectional (RTL/LTR) text.

---

## Why It Matters

DOM measurement is one of the most common performance bottlenecks in web UIs:

- Every call to `offsetHeight`, `getBoundingClientRect`, etc. forces a synchronous layout reflow
- In lists, feeds, or virtualized UIs with dynamic content heights, this is catastrophic for performance
- Server-side rendering has no DOM at all, making height prediction impossible without a library like this

`pretext` solves this by computing paragraph heights and line layouts purely in JavaScript using canvas measurement under the hood.

---

## Architecture

### Source Files (`src/`)

| File | Purpose |
|------|---------|
| `layout.ts` | Core library — public API entry point |
| `analysis.ts` | Text processing and segment analysis |
| `measurement.ts` | Canvas-based font measurement |
| `bidi.ts` | Bidirectional text (RTL/LTR) handling |
| `line-break.ts` | Line-breaking algorithm |
| `layout.test.ts` | Tests |
| `test-data.ts` | Test fixtures |

### Two-Phase Design

The library separates concerns cleanly:

1. **`prepare()` phase** — Normalizes text, measures segments using canvas. Approximately 19ms for a batch of 500 texts. This is the expensive phase.
2. **`layout()` phase** — Pure arithmetic on the prepared data. Approximately 0.09ms per call. This is the hot path (e.g., on resize).

This split means you can batch-prepare on mount and then cheaply re-layout on resize without re-measuring.

---

## Public API

### Use Case 1: Measure Paragraph Height

```ts
import { prepare, layout } from '@chenglou/pretext';

// One-time preparation (batched for performance)
const prepared = await prepare(texts, { font: '16px Arial' });

// Fast layout recalculation (e.g., on resize)
const { height } = layout(prepared[0], { width: 300 });
```

### Use Case 2: Manual Line Layout

```ts
import { prepareWithSegments, layoutWithLines, walkLineRanges, layoutNextLine } from '@chenglou/pretext';

// For fixed-width layouts with per-line control
const prepared = await prepareWithSegments(text, { font: '16px Arial' });
const lines = layoutWithLines(prepared, { width: 400 });

// Inspect line widths without building strings
walkLineRanges(lines, (start, end, width) => { ... });

// Dynamic width per line (text flowing around floated images)
const line = layoutNextLine(state, { width: computeWidthForLine(lineIndex) });
```

### Utilities

- `clearCache()` — Clear internal measurement cache
- `setLocale(locale)` — Set locale for language-specific line breaking

---

## Performance Benchmarks (March 2026)

| Operation | Chrome | Safari |
|-----------|--------|--------|
| `prepare()` (500 texts) | ~18ms | ~18ms |
| `layout()` | ~0.09ms | ~0.09ms |
| `layoutWithLines()` | 0.05ms | 0.05ms |
| Arabic prose (63k+ segments) | 63.50ms prepare | similar |
| Interleaved DOM tasks | 43.50ms | 149.00ms |

**Key insight**: `layout()` is the resize hot path. `prepare()` is where the per-script cost lives.

---

## Accuracy

As of March 2026, all three major browsers pass the full test corpus:

- Chrome: **7680/7680** ✓
- Safari: **7680/7680** ✓
- Firefox: **7680/7680** ✓

Test corpus: 4 fonts × 8 sizes × 8 widths × 30 texts

---

## Supported Rendering Targets

- **DOM** — Standard HTML rendering
- **Canvas** — `<canvas>` 2D context
- **SVG** — Vector graphics
- **Server-side** — No DOM required

---

## Caveats

- Targets common text settings (`normal` whitespace, standard word-breaking)
- Supports `pre-wrap` mode (added in v0.0.2) for textarea-style text
- Does not handle all CSS text properties
- Avoid `system-ui` font on macOS for accuracy

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 0.0.2 | 2026-03-28 | Added `pre-wrap` whitespace mode support |
| 0.0.1 | 2026-03-27 | Improved Safari line-breaking for soft-hyphens; browser tooling stability |
| 0.0.0 | 2026-03-26 | Initial release with `prepare()`, `layout()`, and advanced APIs |

---

## Installation

```sh
npm install @chenglou/pretext
```

---

## Potential Use Cases

- **Virtualized lists** — Accurately predict item heights before render to avoid layout shift
- **Masonry layouts** — Compute column assignments without DOM measurement
- **Server-side rendering** — Pre-compute text heights for above-the-fold content
- **Canvas/WebGL text** — Layout text for non-DOM rendering contexts
- **Rich text editors** — Custom cursor/selection logic without relying on DOM ranges
