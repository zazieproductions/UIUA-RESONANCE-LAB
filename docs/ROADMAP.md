# Roadmap

This document records the forward-looking plan for Uiua Resonance Lab. It is not a promise or a contract — priorities shift with contributor availability, fresh ideas, and user feedback — but it represents the maintainer's current sense of where the project should go.

Versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). The current release is v0.14.0.

---

## Versioning Philosophy

- **0.x.y** — creative-coding instrument phase; the real evaluator is not yet present.
  - Minor bumps (0.14 → 0.15) add features, presets, a11y/perf work.
  - Patch bumps fix bugs and refine documentation.
- **1.0.0** — ships the real Uiua expression parser and evaluator so code in the tacit buffer actually executes and drives the tensor/audio engine.
- **1.x** — focus expands to breadth: deeper Uiua coverage, more presets, sharing features, export.
- **2.0** — potential architecture shift (WebGPU, AudioWorklets, or plugin API).

---

## v0.15 — Polish & Accessibility

Target: Q4 2025. A quality-release focused on closing known gaps.

### Accessibility
- [ ] Honor `prefers-reduced-motion` for decorative animations (pulsing dots, hover transitions).
- [ ] Add `role="img"` and descriptive `aria-label` to the tensor canvas.
- [ ] Add `aria-live` region for preset/frequency/audio state changes.
- [ ] Full reflow audit at 400% zoom.
- [ ] Test with three screen readers + real keyboard-only users.

### Performance
- [ ] Cache `ImageData` across frames (avoid per-frame allocation).
- [ ] Pre-compute an 8-bit indexed colormap LUT to accelerate rasterization at high resolution.
- [ ] Throttle oscilloscope rendering to 30 FPS.

### Audio
- [ ] Sample-accurate cross-fade on buffer refresh (eliminate clicks on preset/freq changes).
- [ ] Exponential ramps on master gain to prevent zipper noise.

### Engineering
- [ ] Add `tests.html` — zero-build test harness exercising invariants (output-boundedness, glyph integrity, preset completeness).
- [ ] HTML validation CI check via GitHub Actions.

---

## v0.16 — Interaction Layer

Target: Q1 2026. Keyboard and UI affordance expansion.

### Keyboard & Shortcuts
- [ ] Global shortcut system: Space = toggle audio, E = evaluate, 1–4 = preset selection, R = reset phase.
- [ ] Arrow-key grid navigation for the 24-glyph palette (right/down advance, up/left retreat).
- [ ] Focusable preset selector with arrow-key navigation.

### UX Improvements
- [ ] Preset "randomize" button that sweeps sliders to interesting positions.
- [ ] Undo/redo for code buffer (simple history stack).
- [ ] Phase reset button (sets `t = 0`).
- [ ] Toolbar for grid resolution presets (32/64/128 quick-switch).
- [ ] Dark/light theme toggle (current palette is dark-only).

### Visual Refinements
- [ ] DPR-aware canvas sizing for crisper rendering on retina displays (behind a toggle or auto-detected).
- [ ] Optional bloom/post-process glow pass via CSS or Canvas composite.

---

## v0.17 — Tooling & Hardening

Target: Q2 2026.

### CI/CD
- [ ] GitHub Actions workflow for deployment to GitHub Pages on `main` push.
- [ ] HTML validator and link-checker in CI.
- [ ] JavaScript syntax validation in CI (extract script block, run `node --check`).
- [ ] Lighthouse CI for performance budget regression checks.

### Security Hardening
- [ ] Add Subresource Integrity (SRI) attributes to CDN script/style tags.
- [ ] CSP meta tag with minimal permissions (remove `'unsafe-inline'` once Tailwind is self-hosted or built).
- [ ] `X-Frame-Options: DENY` / `frame-ancestors 'none'` headers.

### Self-Hosting Option
- [ ] Self-host Tailwind CSS (compiled to a minimal CSS file) and Google Fonts (local WOFF2).
- [ ] Service worker for offline-first PWA behavior.

---

## v0.18 / v0.19 — Expanded Presets & Glyphs

Target: throughout 2026 (rolling).

### New Presets (candidates)
- [ ] **Mandelbrot/Juliá resonance** — iterated complex maps with audio mapped to orbit length.
- [ ] **Reaction-diffusion (Gray-Scott)** — continuous cellular chemistry with slow-evolving patterns.
- [ ] **Lissajous Torus Knots** — 3-D parametric curves projected to 2-D with harmonic audio.
- [ ] **Perlin/Simplex flow field** — vector-field advection with evolving particle streams.
- [ ] **Harmonic sieve (Eratosthenes)** — prime-number visualizations rendered as interference patterns.
- [ ] **Markov-pad texture** — stochastic-state audio drone with correlated visual grain.

### New Glyphs
- [ ] Expand to ~40–50 core Uiua primitives covering: scan, group, sort, rotate, window, reshape, take, drop, fixity, and combinators (fork, atop, both).
- Tooltips link out to `uiua.org/docs/<primitive>` for deep learning.

### Community Preset Sharing
- [ ] Preset export as shareable URLs (base64-encoded preset JSON in the fragment `#...`).
- [ ] Optional preset library file (`presets/` folder or community-contributed JSON bundles).

---

## v1.0 — The Real Evaluator

The flagship release. The tacit code buffer becomes a **live Uiua expression evaluator**, not just a simulated interface.

### Parser & Evaluator
- [ ] **Lexer** for Uiua glyph grammar (glyphs, numbers, arrays, parentheses, modifiers).
- [ ] **Stack machine** runtime that maps 1:1 to Uiua semantics.
- [ ] **Array model** supporting ranks 0 (scalar), 1 (vector), 2 (matrix), and N-D arrays.
- [ ] **Execution model** where the evaluated array result directly drives the tensor rasterization (no preset `render()` function — the field is the output of the user's tacit program).
- [ ] **Audio binding** where a selected axis/operation of the evaluated array drives waveform generation.

### UX
- [ ] Glyph-at-a-time stack trace — see stack state after each primitive in the user's code.
- [ ] Error highlighting — invalid expressions show red squiggles with diagnostic messages.
- [ ] Interactive tutorial — a walkthrough that teaches Uiua by progressively building a cymatic pattern.

### Fidelity
- [ ] Test against real Uiua language outputs for primitives parity where feasible.
- [ ] Document any intentional pedagogical deviations from the Uiua spec.

---

## v1.x — Breadth & Community

- [ ] **MIDI input** — external controller support for carrier frequency, preset switching, and velocity.
- [ ] **Audio export** — render the current preset/synth to downloadable WAV.
- [ ] **Image/Video export** — capture tensor field animations as PNG/WebM/GIF.
- [ ] **Collaborative/shared state** via URL fragments (no backend required).
- [ ] **Plugin API** — allow third-party presets without forking.
- [ ] **Internationalization** (starting with es, fr, pt, ja, zh).

---

## v2.0 — Architectural Horizon (Exploratory)

A major version bump will be justified by one of:
- Migration of the rasterizer to **WebGPU** (compute-shader-based field generation, enabling much higher resolution and 3-D volume rendering).
- Migration of audio to **AudioWorklet** (real-time parameter modulation, per-sample DSP, polyphonic voices).
- Introduction of a **plugin WASM API** for user-supplied primitives or presets compiled from native code.

None of these are committed. They are research directions.

---

## Non-Goals (Explicit)

The following are **not** on the roadmap:

- **Framework migration.** Moving to React, Vue, Svelte, or similar would violate the single-artifact, zero-dependency design. A future WebGPU/AudioWorklet version will still be a single HTML file.
- **Backend/server.** This is a client-side instrument. No accounts, no cloud, no telemetry, no sync.
- **NFT/blockchain integration.** Not happening.
- **General-purpose Uiua IDE.** There is already an excellent official Uiua editor at [uiua.org/editor](https://uiua.org/editor). This project is an audiovisual instrument *inspired by* Uiua, not a replacement editor.
- **Paid or paywalled features.** The project is and will remain MIT-licensed open source.

---

## How to Influence the Roadmap

- **Open an issue** with the `enhancement` tag to propose features not listed here.
- **Vote on issues** with a 👍 reaction to signal priority.
- **Contribute PRs** against roadmap items — PRs against near-term milestones (v0.15, v0.16) will get fastest review.
- **Discuss in GitHub Discussions** for open-ended design questions.

The roadmap is a living document. It changes as the instrument evolves and as the community contributes. ⊞ ○ ∿
