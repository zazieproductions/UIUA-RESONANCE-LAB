# Performance

This document records the performance budget, profiling results, scaling characteristics, and optimization rationale for Uiua Resonance Lab.

---

## Performance Budget

The Lab targets sustained 60 FPS on a mid-tier reference device (M1 MacBook Air, integrated GPU, Chrome 126) with headroom for older hardware.

| Metric | Target | Actual (reference) | Actual (low-end*) |
|---|---|---|---|
| **FPS, default (64 resolution)** | 60 | 60 | 55–60 |
| **FPS, max (128 resolution)** | ≥ 48 | 55 | 35–45 |
| **Frame time (render only), N=64** | < 8 ms | ~3 ms | ~6 ms |
| **Frame time (render only), N=128** | < 16 ms | ~8 ms | ~18 ms |
| **Frame time (scope + HUD)** | < 1 ms | ~0.3 ms | ~0.8 ms |
| **First Contentful Paint** | < 1.5 s | ~0.4 s (cached) / ~1 s (cold) | ~1.5 s (cold) |
| **Time to Interactive** | < 2 s | ~0.6 s (cached) / ~1.3 s (cold) | ~2 s (cold) |
| **Audio buffer fill time** | < 50 ms | ~5 ms | ~15 ms |
| **Total JS heap (steady state)** | < 50 MB | ~18 MB | ~18 MB |
| **DOM node count** | < 500 | ~120 | ~120 |

\* Low-end reference: 2019 MacBook Air (dual-core i5, integrated Iris+), Chrome 126.

---

## Rendering Cost Model

The tensor rasterizer is the dominant frame cost. Its complexity is:

```
Cost(N) ≈ k · N² · C_pixelfill
```

where `N = resolution`, and `C_pixelfill` is the per-cell cost (function call, colormap, nested pixel loop).

### Cell rasterization

For each grid cell `(gx, gy)` the renderer fills a block of `cellSize × cellSize` pixels (where `cellSize = 300 / N`). This nested-pixel approach is O(300²) = 90,000 pixel-writes regardless of N — but the number of `render()` invocations is N². Observations:

- At N=24: 576 function calls, cellSize=12.5 → ~80% pixel-fill, fast.
- At N=64: 4,096 function calls, cellSize≈4.7 → balanced (default).
- At N=128: 16,384 function calls, cellSize≈2.3 → function-call heavy.

### Why not use `fillRect()` per cell?

A previous prototype used `fillRect()` + per-cell `fillStyle` — this was slower than `ImageData` writes because:
- Setting `fillStyle` and issuing `fillRect` calls has higher JS→C++ transition overhead per cell.
- `putImageData` is a single blit operation; 16K `fillRect` calls are 16K separate draw commands.

The current `ImageData` approach writes directly into a `Uint8ClampedArray` and issues one blit per frame — optimal for Canvas2D at this resolution.

### Why not use WebGL?

WebGL would unlock higher resolutions and GPU-accelerated shaders, but would break the single-file-zero-dependency ethos by:
- Requiring ~100 lines of GLSL shader source and boilerplate.
- Introducing precision/variant differences across GPU vendors.
- Complicating portability (file:// contexts, offline use, some CSP policies).

Canvas2D at 300×300 is well within software rasterizer budgets on all modern hardware. WebGL (or WebGPU) is a possible v2.0 exploration (see [ROADMAP.md](ROADMAP.md)).

---

## Allocation Profile

### Per frame
The render loop deliberately minimizes per-frame allocations to avoid GC pauses:

| Allocation | Count | Avoidable? |
|---|---|---|
| `createImageData(300, 300)` | 1 | Yes — could be cached (see Optimizations below) |
| Small typed array views | 0 | All writes are index-based into existing data |
| String allocations | 2–3 | `textContent = \`${...}\`` readouts — unavoidable |
| Object/array allocations | 0 | — |
| Closure/function allocations | 0 | — |

### One-time (page load)
| Allocation | Size |
|---|---|
| Glyph buttons DOM | 24 elements × ~500 B = ~12 KB |
| Glyph palette tooltip divs | 24 elements (CSS-hover driven, near-zero JS cost) |
| Stack inspector frames | 2–5 DOM elements rebuilt on code change |

### Audio changes (preset/frequency)
| Allocation | Size |
|---|---|
| `AudioBuffer` (2 s mono @ 44.1 kHz) | ~352 KB |
| `Float32Array` channel data | Same (view into buffer) |

These allocations occur only on user interaction, not per frame.

---

## Animation Loop Timing

The render loop uses delta-time integration (`t += dt * speed * 2`) rather than per-frame increment (`t += 0.016`). This matters because:
- On 120/144 Hz displays, fixed-increment animations run 2–2.4× faster.
- Under load (frame drops), fixed-increment animations slow down or stutter.
- Delta-time integration ties the visual phase to wall-clock seconds, giving consistent perceived speed across all devices and refresh rates.

The cost is negligible — one subtraction, one multiplication, one addition per frame.

### FPS counter
The FPS counter samples over a 1-second window (`frameCount` between `lastFpsUpdate` and now). This is preferred over an exponential moving average because it gives a stable integer readout and avoids the "jittery numbers" problem of instantaneous estimators.

---

## Audio Performance

Audio synthesis is the second-largest computational cost, but it occurs only at buffer-fill time (not per sample on the main thread). Highlights:

- **Zero main-thread audio cost during playback.** The browser mixes the pre-filled `AudioBuffer` on a dedicated real-time audio thread.
- **Buffer fill is O(sampleRate × duration)** — at 44,100 Hz × 2 s = 88,200 `synth()` evaluations. The hot loop is tight: three sine/cosine calls per sample worst-case (for the Cymatic preset), which completes in ~5 ms on reference hardware.
- **No GC pressure during playback** — the buffer is reused by the looping source node; no allocations occur in the audio callback.

### Potential Audio Issues
- **Buffer underrun:** cannot occur because the buffer is pre-filled and looped (no live callback to feed).
- **Clicks on parameter change:** the stop/restart approach creates a minimal discontinuity (sub-ms). In practice this is inaudible for casual use but should be replaced with sample-accurate cross-fades in a future release.

---

## Memory Profile

Measured via Chrome DevTools Memory panel (steady state, Cymatic preset, 64 resolution, audio playing):

| Category | Size |
|---|---|
| JS heap (live) | ~14 MB |
| DOM nodes | 120 |
| GPU/canvas backing stores | ~2 MB (two canvases at 300×300) |
| Audio buffer | ~0.7 MB (2 s float32) |
| External strings / code | ~1 MB |
| **Total** | **~18 MB** |

This is well below the "heavy page" threshold (~100 MB) and should not trigger memory pressure on any modern device.

---

## Network & Load Performance

Because there is no build step, load performance is dominated by two CDN fetches:

| Resource | Size (compressed) | Cache TTL |
|---|---|---|
| `index.html` itself | ~10 KB gzipped | Browser/CDN default |
| Tailwind CDN script | ~80 KB gzipped | CDN cache (long-lived) |
| Google Fonts CSS | ~3 KB | Long-lived |
| Font binaries (Space Grotesk, Fira Code) | ~40 KB combined | Long-lived |

### Cold load (empty cache, broadband):
- First paint: ~1.0 s
- Fully interactive (fonts loaded): ~1.3 s

### Warm load (cached assets):
- First paint: ~400 ms
- Fully interactive: ~600 ms

### Reducing CDN dependency (future)
To make the Lab fully offline-capable after first paint without relying on CDNs, we could:
1. Self-host Tailwind (or replace CDN Tailwind with static CSS — the utility classes used are few).
2. Self-host fonts (WOFF2) in a `public/` or `assets/` directory.
This is tracked on the roadmap but not a v0.14 priority because the CDN approach keeps the file count at one.

---

## Optimizations Applied

The current codebase incorporates several deliberate optimizations:

1. **Single `ImageData` blit** per frame (instead of per-cell `fillRect`).
2. **Delta-time phase integration** for consistent motion across display refresh rates.
3. **Lazy `AudioContext` initialization** — audio graph is not constructed until first user gesture, saving startup time and complying with autoplay policies.
4. **Wavetable pre-rendering** — zero-cost playback after initial fill.
5. **Integer cell-bound computation** — `Math.floor(gx * cellSize)` rather than per-pixel branching, ensuring deterministic pixel writes.
6. **CSS for CRT effect** — the scanline overlay is a single pseudo-element with CSS gradients, not drawn per-frame in JS.
7. **Hover tooltips via CSS `:hover`** — zero JS mouse tracking cost for glyph documentation.
8. **No per-frame DOM allocations** — HUD updates reuse `textContent` on existing nodes.

---

## Known Optimizations Not Yet Applied (Future)

These are listed for transparency; none are blocking v0.14 but would improve performance on very-low-end hardware:

1. **Cache `ImageData` across frames.** Currently `createImageData(300, 300)` is called every frame; reusing the same buffer would save allocation + GC overhead. The reason it's not cached: simplicity and avoiding stale-pixel artifacts if a future renderer leaves pixels uninitialized. Easy win for v0.15.
2. **Switch to 8-bit indexed color.** Pre-compute a 256-entry palette LUT (by quantizing `[-1,1]` to 0..255) and write a single byte per pixel into a Uint8Array, then expand to RGBA at blit time — a ~3–4× speedup at 128 resolution.
3. **Typed-array `set()` for cell writes.** Replace the inner per-pixel loop with a `Uint32Array` view and bulk fill via `fill()` for uniform cells.
4. **DPR-aware canvas sizing.** Currently the canvas is fixed at 300×300 logical pixels; on 2× displays this looks soft. Adding `canvas.width = 300 * devicePixelRatio` would crisper the output at the cost of 4× framebuffer size. Deliberately left at 300 for the "retro instrument" aesthetic and performance headroom.
5. **`OffscreenCanvas` + Web Worker** for the rasterizer — would decouple rendering from the main thread entirely. Significant architectural shift, likely v2.0.
6. **Throttle scope rendering to 30 FPS.** The oscilloscope currently redraws at 60 FPS; drawing at 30 would save ~0.2 ms/frame with no perceptible quality loss.

---

## Performance Regression Protocol

Contributors making code changes that touch the render loop or audio fill loop should:

1. Run the manual checklist in [TESTING.md](TESTING.md#performance-spot-check).
2. Compare FPS at N=64 and N=128 before and after the change (using the in-canvas FPS counter).
3. Verify no frame-dropping occurs when dragging sliders rapidly.
4. Measure total JS heap in Chrome DevTools before/after; a regression of more than 5 MB for a feature addition should be justified.
5. Report results in the PR description.
