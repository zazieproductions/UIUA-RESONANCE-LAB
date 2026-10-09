# API Reference

This document specifies the internal module contracts used within **Uiua Resonance Lab**. Because the application is a single-file vanilla JavaScript project, these APIs are not exposed as ES modules or npm packages; this document instead serves as a formal contract for contributors modifying or extending the engine.

---

## Table of Contents

1. [Glyph Database](#glyph-database)
2. [Preset Interface](#preset-interface)
3. [Render Function Signature](#render-function-signature)
4. [Synth Function Signature](#synth-function-signature)
5. [Color Map](#color-map)
6. [DOM Helpers](#dom-helpers)
7. [Audio Engine](#audio-engine)
8. [State Shape](#state-shape)
9. [Events](#events)

---

## Glyph Database

### `GLYPHS: ReadonlyArray<GlyphDef>`

Declared as a top-level `const` array. Never mutated at runtime.

```typescript
interface GlyphDef {
  glyph: string;   // Single Unicode character (Uiua primitive)
  name: string;    // Lowercase primitive identifier (kebab-case)
  doc: string;     // One-sentence human-readable description
  arity: 0 | 1 | 2; // Stack arguments consumed (0 = constant, 1 = monad, 2 = dyad)
  cat: GlyphCategory;
}

type GlyphCategory =
  | 'array'      // Tensor construction / manipulation (⊞, ⇡, ♭, ∺, ⊂, ◰)
  | 'math'       // Scalar / element-wise math (○, ∿, τ, ×, +, ÷, ⁿ, √, ∡, η)
  | 'transform'  // Structural array transforms (⇌, ⍉)
  | 'modifier'   // Higher-order combinators (⍥, ∵, ⌿)
  | 'inspect'    // Metadata / observation (△)
  | 'search'     // Pattern matching (⌕)
  | 'filter';    // Masking / selection (▽)
```

### Invariants
- `glyph.length === 1` for every entry.
- `name` matches the Uiua primitive identifier at [uiua.org/docs](https://uiua.org/docs).
- `doc` is a complete sentence ending in a period.
- `arity` correctly reflects the number of *array arguments* consumed. Modifiers that take function arguments still declare the arity of their data operands.

---

## Preset Interface

### `PRESETS: Record<string, Preset>`

Declared as a top-level constant mapping preset keys to preset objects.

```typescript
interface Preset {
  title: string;       // Human-readable display name (Title Case)
  code: string;        // Uiua glyph expression (backtick-quoted when rendered)
  doc: string;         // One-paragraph description of the artifact's math/physics
  mode: string;        // Short HUD label for the display-mode corner badge
  render: RenderFn;    // Pure: tensor field scalar function
  synth: SynthFn;      // Pure: audio sample function
}
```

### Preset Keys (v0.14)

| Key | Title |
|---|---|
| `'cymatics'` | Cymatic Lattice |
| `'subharmonic'` | Fibonacci Subharmonic Bells |
| `'torus'` | Phase Toroid Drift |
| `'stochastic'` | Cellular Dilation Matrix |

### Adding a Preset
See [CONTRIBUTING.md](../CONTRIBUTING.md#adding-a-new-preset) for the required steps. Every new preset MUST:

1. Append a new entry to `PRESETS` with the four metadata fields and both function fields.
2. Add a corresponding `.preset-btn` element to the selector grid in the `<body>` markup.
3. Be accompanied by an entry in [PRESETS.md](PRESETS.md).

---

## Render Function Signature

```typescript
type RenderFn = (u: number, v: number, t: number, f: number) => number;
```

### Parameters

| Name | Type | Range | Description |
|---|---|---|---|
| `u` | `number` | `[-10, 10]` | X-axis coordinate in continuous tensor space |
| `v` | `number` | `[-10, 10]` | Y-axis coordinate in continuous tensor space |
| `t` | `number` | `[0, ∞)` rad | Accumulated phase (runs at `speed * 2` rad/sec) |
| `f` | `number` | `[55, 880]` Hz | Current carrier frequency |

### Return
- `number` — scalar field value, **approximately bounded in `[-1, 1]`** for correct colormapping. Values outside this range will be clamped by `colorMap()`.

### Purity Contract
A valid `RenderFn` must:
- Be referentially transparent: identical inputs ⇒ identical output.
- Read no external mutable state (no closure over `resolution`, `speed`, DOM, etc.).
- Perform no DOM access, no `Math.random()`, no `Date.now()`, no I/O.
- Perform no mutation of arguments or shared objects.
- Run in well under 1 µs per cell (budget: 128 × 128 = 16,384 cells per frame at 60 FPS ⇒ ~1 µs/cell budget).

---

## Synth Function Signature

```typescript
type SynthFn = (time: number, f: number) => number;
```

### Parameters

| Name | Type | Range | Description |
|---|---|---|---|
| `time` | `number` | `[0, 2.0]` sec | Sample time within the wavetable (buffer is 2-second looping) |
| `f` | `number` | `[55, 880]` Hz | Current carrier frequency |

### Return
- `number` — audio sample, **bounded in `[-1, 1]`** to prevent clipping after the gain stage.

### Purity Contract
Same as `RenderFn`: referentially transparent, no external state, no side effects. This allows the wavetable to be pre-computed deterministically and looped seamlessly.

### Looping Requirement
Because the buffer is 2 seconds long and loops via `audioNode.loop = true`, `synth(0, f)` and `synth(2.0, f)` should ideally produce the same value (waveform continuity) to avoid a click at the loop point. Presets that fail this invariant will produce an audible 2-second periodic artifact — acceptable for textural sounds (e.g., Cellular Dilation), but should be avoided for tonal presets.

---

## Color Map

### `colorMap(val: number): [r: number, g: number, b: number]`

Maps a scalar in `[-1, 1]` to an RGB triple in `[0, 255]`.

```typescript
function colorMap(val: number): [number, number, number];
```

### Behavior
- Clamps `val` to `[-1, 1]` via `Math.max(0, Math.min(1, (val + 1) * 0.5))`.
- Splits into two piecewise-linear segments:
  - `n < 0.5`: interpolates deep ink-blue → electric purple.
  - `n ≥ 0.5`: interpolates electric purple → vibrant teal highlight.
- Returns integer `[r, g, b]` values each in `[0, 255]`.

### Stops (normalized 0..1)
| Stop | Approx. color | RGB |
|---|---|---|
| 0.0 | Deep ink | `(10, 12, 28)` |
| 0.5 | Electric purple | `(150, 42, 238)` |
| 1.0 | Uiua teal | `(94, 241, 255)` |

### Extending
To introduce alternative colormaps (roadmap item), model them as `(val: number) => [number, number, number]` and add a UI selector. The current `colorMap` function is the default for v0.14.

---

## DOM Helpers

### `insertGlyph(char: string): void`
Inserts a glyph character into the code buffer at the current caret/selection position and refocuses the textarea. Triggers `updateStackView()` after insertion.

### `loadPreset(key: string): void`
Loads a preset into the active state:
1. Sets `currentPreset = key`.
2. Populates the code buffer with `PRESETS[key].code`.
3. Updates the display-mode HUD label.
4. Updates the explanation panel with the preset's `doc` string.
5. Re-applies styling to preset selector buttons (active/inactive).
6. Calls `updateStackView()`.

Note: does **not** refresh the audio buffer — callers must invoke `refreshAudioBuffer()` separately if audio is playing.

### `updateStackView(): void`
Reads the current code buffer text, pattern-matches for known glyphs, and renders a stack frame list into `#stack-items`. This is a purely *presentational* simulator — it does not parse or evaluate Uiua. Recognized patterns:

| Pattern | Pushed frame |
|---|---|
| Contains `⊞` or `table` | Tensor 2D `[N N]` |
| Contains `○`, `∿`, or `sin` | Signal vector `[N*N]` |
| Contains `⇡` or `range` | Index sequence `[N]` |
| (none matched) | Scalar phase float |

---

## Audio Engine

### `initAudio(): void`
Lazily initializes the Web Audio graph on first user gesture. Idempotent (safe to call repeatedly):
- Creates `AudioContext` if absent.
- Creates `GainNode` (gain = 0.2) connected to `AnalyserNode` (fftSize = 256), which connects to `audioCtx.destination`.
- Calls `audioCtx.resume()` if context is suspended.

### `toggleAudioPlayback(): void`
Toggles audio playback state. On play:
1. Calls `initAudio()`.
2. Fills a 2-second mono `AudioBuffer` at the context sample rate by iterating `synth(sampleTime, frequency)`.
3. Creates an `AudioBufferSourceNode` with `loop = true`.
4. Connects to `gainNode` and calls `start()`.
5. Flips UI state (button label, mute indicator).

On stop:
1. Stops and disconnects the source node.
2. Nullifies `audioNode`.
3. Flips UI state.

### `refreshAudioBuffer(): void`
If audio is currently playing, calls `toggleAudioPlayback()` twice (stop then start) to regenerate the wavetable with current frequency/preset. Crude but simple and reliable for v0.14.

---

## State Shape

All mutable state is declared at the top of the `<script>` block with `let`:

```typescript
// Engine state
let currentPreset: string;                // key into PRESETS
let isRunning: boolean;                   // master loop gate (reserved)
let t: number;                            // accumulated phase (rad)
let frequency: number;                    // carrier (Hz)
let resolution: number;                   // grid resolution per axis
let speed: number;                        // phase velocity multiplier
let isAudioPlaying: boolean;              // DSP gate

// Web Audio nodes (lazily initialized)
let audioCtx: AudioContext | null;
let audioNode: AudioBufferSourceNode | null;
let gainNode: GainNode | null;
let analyser: AnalyserNode | null;

// Render-loop bookkeeping
let lastTime: number;                     // previous rAF timestamp
let frameCount: number;                   // frames since last FPS update
let lastFpsUpdate: number;                // last FPS sample timestamp
```

DOM references are cached in `const` bindings after the DOM is ready (script block is at end of `<body>`, so DOM is fully parsed at execution time):

```typescript
const tensorCanvas = document.getElementById('tensorCanvas') as HTMLCanvasElement;
const tensorCtx = tensorCanvas.getContext('2d')!;
const scopeCanvas = document.getElementById('scopeCanvas') as HTMLCanvasElement;
const scopeCtx = scopeCanvas.getContext('2d')!;
const codeArea = document.getElementById('uiua-code') as HTMLTextAreaElement;
// ... etc.
```

---

## Render Loop

### `renderLoop(now: number): void`
A single function invoked via `requestAnimationFrame(renderLoop)`. The `now` argument is the high-resolution timestamp provided by `requestAnimationFrame` (ms since page load).

**Per-frame steps (in order):**
1. Compute `dt = (now - lastTime) / 1000` (seconds).
2. Advance phase: `t += dt * speed * 2`.
3. FPS sampling: increment `frameCount`; every 1000 ms, write to `#fps-counter`.
4. Update `#time-val` with `t % (2π)` to three decimal places.
5. Resolve canvas backing store to 300×300.
6. Create `ImageData`, rasterize grid cells via preset `render()`, apply `colorMap()`, fill pixel blocks.
7. Overlay gridlines.
8. Resize and paint oscilloscope strip — live analyser data if playing, else synthetic preview.
9. Schedule next frame: `requestAnimationFrame(renderLoop)`.

### Timing Model
- **dt integration** rather than frame-count integration ensures that phase `t` advances at a wall-clock-independent rate, so the animation doesn't speed up on high-refresh displays or slow down under load.
- The `speed * 2` multiplier yields a base rate of 2 rad/sec at speed = 1.0, giving a visual period of π ≈ 3.14 seconds — slow enough to see structure, fast enough to feel alive.

---

## Events

### UI Inputs

| Element | Event | Effect |
|---|---|---|
| `.preset-btn` | `click` | `loadPreset(data-preset)` + `refreshAudioBuffer()` |
| `#btn-eval` | `click` | Briefly flashes button ring; calls `updateStackView()` |
| `#btn-play-synth` | `click` | `toggleAudioPlayback()` |
| `#btn-master-mute` | `click` | `toggleAudioPlayback()` |
| Glyph palette buttons | `click` | `insertGlyph(g.glyph)` |
| `#param-freq` | `input` | Set `frequency`; updates readout; `refreshAudioBuffer()` |
| `#param-res` | `input` | Set `resolution`; updates readout; `updateStackView()` |
| `#param-speed` | `input` | Set `speed`; updates readout (no audio refresh needed — only affects phase) |
| `window` | `resize` | Resizes scope canvas to match parent |

### Audio Context Lifecycle
- Per browser autoplay policies, `AudioContext` is created in `suspended` state and only resumed after a user gesture on `toggleAudioPlayback()`.
- The master mute button and the synthesize button are wired to the same toggle for consistency.

---

## Constants Summary

| Constant | Value | Rationale |
|---|---|---|
| Canvas logical size | 300×300 px | Keeps rasterization cheap while retaining detail at N=128 |
| Audio buffer length | 2 seconds | Long enough for bell-decay presets; short enough to re-fill quickly on param change |
| Master gain | 0.2 | Prevents clipping when three harmonics sum; leaves headroom |
| Analyser FFT size | 256 | Smooth scope trace with minimal CPU overhead |
| Phase base rate | 2 rad/sec | Comfortable visual speed at default 1× velocity |
| Tensor plane span | 20 units (±10) | Produces ~7 cymatic cycles at ω=220 Hz, w≈2.75 |
| Max raster resolution | 128 | Keeps worst-case frame time under 16 ms on reference hardware |
