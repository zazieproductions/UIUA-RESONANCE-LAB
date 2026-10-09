# Changelog

All notable changes to **Uiua Resonance Lab** are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.14] — Tacit Array (Current)

*Release date: 2025-10-09*

### Added
- **Four curated preset artifacts** — Cymatic Lattice, Fibonacci Subharmonic Bells, Phase Toroid Drift, and Cellular Dilation Matrix — each with distinct visual topology and synthesis profile.
- **Glyph palette** of 24 click-to-insert Uiua-inspired primitives covering array construction, math transforms, modifiers, filtering, and inspection categories.
- **Tacit program buffer** with hover-documented stack signatures (arity, category) and live evaluation affordance.
- **24-glyph stack inspector** that reflects tensor, signal, and index items based on code-buffer content, with depth counter and type tags.
- **Dynamic hyper-parameter T-vector controls** — carrier frequency (55–880 Hz, 0.5 Hz step), tensor resolution (24–128, step 8), phase-modulation velocity (0.1–4.0×).
- **Real-time Web Audio synthesis engine** using procedural wavetable generation, `BufferSourceNode`, gain staging, and `AnalyserNode`-driven oscilloscope.
- **300×300 Canvas2D tensor rasterizer** with deterministic five-stop cyan/violet colormap and CRT scanline overlay.
- **Oscilloscope strip** displaying either live `AnalyserNode` time-domain data or synthetic fallback waveform when audio is muted.
- **FPS counter** with 1-second sample window and matrix-shape indicator.
- **Tailwind CDN styling** with explicit CSS custom-property design tokens (`--bg`, `--card-bg`, `--border`, `--accent`, `--glow`, `--amber`, `--magenta`, `--purple`).
- **Typography pairing** — Space Grotesk for display and UI copy, Fira Code for monospaced glyphs and stack diagnostics.
- **Mute toggle** in the header with pulsing DSP status indicator.
- **Responsive layout** via Tailwind 12-column grid (collapses to single-column below `lg` breakpoint).

### Architecture
- **Zero-dependency, zero-build** single-file application — `index.html` contains all markup, styling, and runtime logic.
- **Deterministic render loop** driven by `requestAnimationFrame` with delta-time integration for phase-velocity stability across framerates.
- **Hot-swappable preset system** — each preset exports a pure `render(u, v, t, f)` function and a pure `synth(time, f)` function, enabling live replacement without engine restart.
- **Graceful audio lifecycle** — `AudioContext` is lazily initialized on first user gesture (browser autoplay-policy compliant), suspended/resumed on toggle, and buffer refreshed automatically when parameters change.

### Documentation
- Full repository documentation suite (see [docs/](docs/)): Architecture, API, Glyph Dictionary, Preset Catalog, Audio Pipeline, Performance, Accessibility, Testing, Deployment, and Roadmap.

---

## [Unreleased]

See [docs/ROADMAP.md](docs/ROADMAP.md) for planned work.

---

### Versioning Notes

- **Major (1.x.x)** will ship when a formal Uiua expression parser and live evaluator replaces the simulated stack.
- **Minor (0.x.0)** will accompany each new preset family, audio feature, or rendering mode.
- **Patch (0.0.x)** will ship for bug fixes, a11y improvements, performance tuning, and documentation corrections.
