# Architecture

This document details the system design of **Uiua Resonance Lab v0.14**: its module boundaries, data flow, rendering pipeline, audio graph, state model, and the design decisions that bind them into a single HTML artifact.

---

## Design Tenets

Before diving into diagrams, three non-negotiable constraints shaped every architectural choice:

1. **Single-artifact portability.** The entire application is one `index.html` file. No build. No external JS. No asset pipeline. The file must work when double-clicked from a desktop, served from any static host, or opened offline after first paint.
2. **Zero runtime dependencies after first paint.** Tailwind (CDN) and Google Fonts are loaded at document load but are *not* required for functional correctness — the application is usable with cached styles only, and the JS engine depends on nothing but browser-standard APIs.
3. **Deterministic real-time rendering.** Given the same preset, parameters, and phase `t`, the rendered tensor field must be byte-identical. No nondeterminism, no GC-dependent timing, no Math.random() in the hot path.

These constraints are deliberate — they are what make this a portfolio-grade systems artifact rather than a "quick creative-coding sketch."

---

## System Diagram

```
                       ┌─────────────────────────────────────────────────┐
                       │                  index.html                     │
                       │                                                  │
   ┌──────────┐        │  ┌──────────┐   ┌────────────────────────────┐   │
   │  CDN     │        │  │  <head>  │   │         <style>            │   │
   │  Fonts + │───────▶│  │  Meta    │   │  Design tokens (CSS vars)  │   │
   │  Tailwind│        │  │  Fonts   │   │  Component classes         │   │
   └──────────┘        │  └──────────┘   │  CRT / animation keyframes │   │
                       │                 └────────────────────────────┘   │
                       │                                                  │
                       │  ┌──────────────────────────────────────────┐    │
                       │  │                <body>                    │    │
                       │  │                                          │    │
                       │  │  ┌────────────── Header ──────────────┐  │    │
                       │  │  │  Brand · Engine badge · Mute toggle │  │    │
                       │  │  └────────────────────────────────────┘  │    │
                       │  │                                          │    │
                       │  │  ┌───── Main Grid (12-col lg) ────────┐  │    │
                       │  │  │                                     │  │    │
                       │  │  │  ┌── Left (7/12) ──────────────┐    │  │    │
                       │  │  │  │ Preset Selector              │    │  │    │
                       │  │  │  │ Tacit Code Buffer            │    │  │    │
                       │  │  │  │ Glyph Palette (24 buttons)   │    │  │    │
                       │  │  │  │ Hyper-Parameter Sliders      │    │  │    │
                       │  │  │  └─────────────────────────────┘    │  │    │
                       │  │  │                                     │  │    │
                       │  │  │  ┌── Right (5/12) ─────────────┐    │  │    │
                       │  │  │  │ Tensor Canvas (CRT-framed)   │    │  │    │
                       │  │  │  │ Oscilloscope Strip           │    │  │    │
                       │  │  │  │ Stack Inspector              │    │  │    │
                       │  │  │  └─────────────────────────────┘    │  │    │
                       │  │  └─────────────────────────────────────┘  │    │
                       │  └──────────────────────────────────────────┘    │
                       │                                                  │
                       │  ┌──────────────────────────────────────────┐    │
                       │  │             <script> (engine)            │    │
                       │  │  ┌─────────┐  ┌──────────┐  ┌─────────┐ │    │
                       │  │  │  GLYPHS │  │ PRESETS  │  │ State   │ │    │
                       │  │  │ (const) │  │ (const)  │  │ (let)   │ │    │
                       │  │  └─────────┘  └──────────┘  └─────────┘ │    │
                       │  │  ┌─────────┐  ┌──────────┐  ┌─────────┐ │    │
                       │  │  │ DOM     │  │ Audio    │  │ Render  │ │    │
                       │  │  │ wiring  │  │ Engine   │  │ Loop    │ │    │
                       │  │  └─────────┘  └──────────┘  └─────────┘ │    │
                       │  └──────────────────────────────────────────┘    │
                       └─────────────────────────────────────────────────┘
```

---

## Module Decomposition

The runtime script block divides cleanly into six conceptual regions, in declaration order:

### 1. Glyph Database (`GLYPHS`)
- **Type:** `ReadonlyArray<GlyphDef>` (constant, never mutated)
- **Responsibility:** Canonical definitions of 24 Uiua-inspired primitives used to populate the clickable palette and their hover tooltips.
- **Contract:** Each entry provides `glyph` (Unicode), `name`, `doc`, `arity` (0/1/2), and `cat` (functional category). See [docs/GLYPHS.md](GLYPHS.md) for the full table.

### 2. Preset Catalog (`PRESETS`)
- **Type:** `Record<string, Preset>` (constant, never mutated after declaration)
- **Responsibility:** Encapsulates all four artifact definitions. Each preset exports *two* pure functions:
  - `render(u: number, v: number, t: number, f: number): number` — tensor field scalar in `[-1, 1]`
  - `synth(time: number, f: number): number` — audio sample in `[-1, 1]`
  Plus metadata: `title`, `code` (Uiua glyph expression string), `doc`, `mode` (HUD label).
- **Purity guarantee:** Neither function reads nor writes mutable state; neither touches the DOM. This enables hot-swapping and (in principle) offline testing.
- **See:** [docs/PRESETS.md](PRESETS.md), [docs/API.md](API.md#preset-interface)

### 3. Mutable State
All mutable live state is declared in a single lexical region:

| Variable | Type | Default | Purpose |
|---|---|---|---|
| `currentPreset` | `string` | `'cymatics'` | Active preset key into `PRESETS` |
| `isRunning` | `boolean` | `true` | Global animation loop gate (reserved) |
| `t` | `number` | `0` | Accumulated phase in radians |
| `frequency` | `number` | `220` | Carrier ω in Hz |
| `resolution` | `integer` | `64` | Grid cells per tensor axis |
| `speed` | `number` | `1.0` | Phase-modulation velocity multiplier |
| `isAudioPlaying` | `boolean` | `false` | Whether DSP graph is running |
| `audioCtx` | `AudioContext \| null` | `null` | Lazy-initialized Web Audio context |
| `audioNode` | `AudioBufferSourceNode \| null` | `null` | Active wavetable node |
| `gainNode` | `GainNode \| null` | `null` | Master gain (0.2) |
| `analyser` | `AnalyserNode \| null` | `null` | FFT/scope tap |

All mutations to these variables occur inside event handlers or the render loop — no other code writes state.

### 4. DOM Wiring
This region:
- Caches references to all interactive canvas elements and DOM nodes (`getElementById`).
- Programmatically builds the glyph button grid from `GLYPHS` (avoids 24 copies of handwritten HTML).
- Attaches event listeners for preset selection, evaluate/play buttons, sliders, and mute.
- Defines UI helpers: `insertGlyph()`, `loadPreset()`, `updateStackView()`.

### 5. Audio Engine
A thin wrapper over Web Audio:
- `initAudio()` — lazily constructs `AudioContext`, gain, and analyser on first user gesture (complies with browser autoplay policies).
- `toggleAudioPlayback()` — starts or stops a looping `AudioBufferSourceNode` whose buffer is procedurally filled from the active preset's `synth()` function at the current frequency.
- `refreshAudioBuffer()` — stops and restarts the source node, used when frequency/preset changes require a new wavetable.

See [docs/AUDIO.md](AUDIO.md) for a full walkthrough of the DSP graph.

### 6. Render Loop (`renderLoop`)
A `requestAnimationFrame`-driven loop that:
1. Computes `dt` from wall-clock delta (smoothes over frame jitter).
2. Integrates phase `t += dt * speed * 2`.
3. Updates FPS counter on a 1-second cadence.
4. Rasterizes the tensor field by nested `for` loops into an `ImageData` buffer.
5. Draws teal gridline overlay.
6. Draws oscilloscope trace (live analyser data when playing, synthetic waveform otherwise).
7. Schedules next frame.

---

## Data Flow

```
   ┌───────────────────────────┐
   │  UI Event (click / input) │
   └─────────────┬─────────────┘
                 │  updates
                 ▼
   ┌───────────────────────────┐
   │     Mutable State         │ ← t advances in rAF loop
   │  (preset, freq, res, …)   │
   └─────────┬─────────┬───────┘
             │         │
             │         │
     ┌───────┘         └────────┐
     ▼                          ▼
┌────────────┐           ┌──────────────┐
│ p.render() │           │ p.synth()    │
│  (per-cell)│           │  (per-sample)│
└─────┬──────┘           └──────┬───────┘
      │ colorMap()              │ AudioBuffer
      ▼                         ▼
┌────────────┐           ┌──────────────┐
│ ImageData  │           │ AudioContext │
│ → putImage │           │ → gain →     │
│            │           │   analyser → │
└─────┬──────┘           │   destination│
      │                  └──────┬───────┘
      │                         │
      ▼                         ▼
   ┌─────────────────────────────────────┐
   │       Canvas 2D compositing         │
   │  (field + gridlines + oscilloscope) │
   └─────────────────────────────────────┘
```

### Key properties
- **Unidirectional data flow.** UI → state → pure functions → canvas. There are no two-way bindings, no observables, no reactive framework. The render loop simply reads state every frame.
- **No allocations in the hot path.** The `ImageData` object is created once per frame and filled in place; the glyph and preset objects are constant; the audio buffer is created only on preset/frequency change.
- **Separation of math from presentation.** `render()` and `synth()` know nothing about Canvas or Web Audio. The colormap and DSP plumbing live outside of them.

---

## Rendering Pipeline (Tensor Field)

The Canvas2D rasterizer operates as follows:

1. **Resize.** `tensorCanvas.width/height = 300` (logical pixels; DPR scaling is intentionally *not* applied to keep the cell-grid aesthetic crisp and performance-predictable).
2. **Allocate framebuffer.** `createImageData(300, 300)` → `Uint8ClampedArray` of length `300 × 300 × 4`.
3. **Iterate grid.** For each cell `(gx, gy)` in the active resolution (default 64×64):
   - Normalize device coordinates `(u, v)` to `[-10, 10]` in tensor space.
   - Compute scalar `val = p.render(u, v, t, f)` (approximately `[-1, 1]`).
   - Map scalar → `[r, g, b]` via `colorMap()`.
   - Rasterize into pixel block of size `cellSize = 300 / resolution`.
4. **Composite gridlines.** 8×8 subtle teal strokes over the rasterized field, giving the impression of a gridded instrument panel.
5. **Blit.** `putImageData(imgData, 0, 0)`.

### Colormap

The `colorMap()` function is a piecewise linear interpolation across five stops, designed for perceptual uniformity on dark backgrounds:

| Scalar | Region | Approx. color |
|---|---|---|
| `-1.0` | deep | `rgb(10, 12, 28)` — near-black ink blue |
| `0.0` | mid-deep | `rgb(80, 27, 133)` — electric purple |
| `+1.0` | peak | `rgb(94, 241, 228)` — Uiua teal highlight |

(Exact channel values in source. The two-segment split at `n < 0.5` and `n ≥ 0.5` avoids a branch-per-channel, keeping the inner loop tight.)

---

## Audio Graph

```
   ┌──────────────────────┐
   │ AudioBufferSourceNode│  ← 2-second looping wavetable
   │ (buffer filled from  │     filled from p.synth()
   │  preset.synth())     │
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │     GainNode         │  ← gain = 0.2 (prevents clipping)
   └──────────┬───────────┘
              ├──────────────────────┐
              ▼                      ▼
   ┌──────────────────────┐   ┌────────────────┐
   │   AnalyserNode       │   │  destination   │  ← speakers
   │   (fftSize=256)      │   │                │
   └──────────┬───────────┘   └────────────────┘
              │
              ▼
       Oscilloscope canvas
   (getByteTimeDomainData)
```

See [docs/AUDIO.md](AUDIO.md) for harmonic structure per preset, gain-staging rationale, and notes on wavetable size.

---

## Stack Inspector — Why It's Simulated

The stack inspector in the right panel does **not** parse or evaluate Uiua code. It performs lexical pattern matching on the code buffer (`text.includes('⊞')`, etc.) and displays representative stack frames. This is a deliberate v0.14 decision:

- A real Uiua evaluator in JavaScript is a substantial project (a parser, lexer, and array runtime) — equivalent in scope to the entire current app.
- The simulator conveys the *semantics* of tacit stack programming for educational and aesthetic purposes without promising false execution accuracy.
- Shipping a real evaluator is the flagship goal of v1.0 (see [docs/ROADMAP.md](ROADMAP.md)).

This distinction is made visible to the user via the "Tacit Program Buffer" framing (the buffer is described as a glyphic interface, not a live evaluator).

---

## State Transitions

```
                  ┌──────────────┐
        page load │   INITIAL    │
        ─────────▶│   (muted)    │
                  └──────┬───────┘
                         │ user clicks SYNTHESIZE / MUTE
                         ▼
                  ┌──────────────┐
                  │   AUDIO      │◀─── slider changes ──▶ refreshAudioBuffer()
                  │   PLAYING    │     (preset, freq)
                  └──────┬───────┘
                         │ user clicks HALT / MUTE
                         ▼
                  ┌──────────────┐
                  │   AUDIO      │
                  │   MUTED      │──── rAF continues visual rendering ────┐
                  └──────────────┘                                        │
                       ▲                                                  │
                       └──────────────────────────────────────────────────┘
```

Visual rendering runs continuously from `requestAnimationFrame` regardless of audio state. Muting stops only the DSP graph, not the animation — this preserves the visual instrument behavior.

---

## Coordinate Spaces

Three coordinate systems are in play; translating between them correctly is the core math of the render loop:

| Space | Range | Unit | Meaning |
|---|---|---|---|
| **Canvas pixel** | `(0..300, 0..300)` | px | Device pixels in the `<canvas>` framebuffer |
| **Grid cell** | `(0..N-1, 0..N-1)` where `N = resolution` | cells | Discrete grid coordinates, one per tensor element |
| **Tensor field** | `(u, v) ∈ [-10, 10]²` | dimensionless | Continuous mathematical plane where render functions operate |
| **Phase** | `t ∈ ℝ` (mod 2π displayed) | radians | Accumulated time, scaled by `speed` |
| **Audio time** | `time ∈ ℝ` | seconds | Wall-clock sample time into the wavetable |

The mapping `(gx, gy) → (u, v)` is linear: `u = (gx/N - 0.5) * 20`, `v = (gy/N - 0.5) * 20`. The 20-unit span is chosen because the default cymatic wavelength of `f/80` at 220 Hz ≈ 2.75 produces approximately 7 cycles across the viewport at default settings — dense enough to look rich, sparse enough to discern individual nodes.

---

## Performance Budget

| Metric | Target | Actual (M1 MacBook Air, Chrome 126) |
|---|---|---|
| Framerate | 60 FPS sustained | 60 FPS @ 64 resolution, ~55 FPS @ 128 |
| Frame time (render) | <8 ms | ~3 ms @ 64, ~8 ms @ 128 |
| Audio latency | <50 ms buffer fill | ~5 ms fill (pre-buffered) |
| First paint | <1 s on broadband | ~400 ms cached, ~1 s cold |
| Memory | <50 MB | ~18 MB (ImageData + audio buffer) |
| Bundle size | n/a (single HTML) | 37 KB uncompressed, ~10 KB gzipped |

See [docs/PERFORMANCE.md](PERFORMANCE.md) for deep analysis.

---

## File Architecture

| Concern | Location | Why |
|---|---|---|
| Entry point | `index.html` | Single-artifact guarantee |
| Community docs | `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, `LICENSE` | Repo root, GitHub conventions |
| Technical docs | `docs/*.md` | Keep README concise; deep content linked |
| Issue templates | `.github/ISSUE_TEMPLATE/` | GitHub automatic discovery |
| Build output | (none) | Zero-build guarantee |

No source-maps, no transpilation, no minification. The file is readable in "View Source" — that is intentional and part of the project's educational stance.
