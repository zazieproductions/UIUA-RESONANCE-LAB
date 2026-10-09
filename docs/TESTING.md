# Testing Strategy

Uiua Resonance Lab is a single-file creative-coding application with no build step, no external test harness, and no logic that runs outside the browser. As such, the testing approach balances lightweight automated checks with a rigorous manual verification matrix, rather than introducing a heavy testing infrastructure that would violate the project's zero-dependency constraint.

---

## Testing Philosophy

The three properties worth protecting are:

1. **Functional correctness** — every button, slider, preset, and toggle does what the user expects.
2. **Rendering determinism** — given the same preset and phase, the tensor field is visually stable (no flicker, no NaN, no seams).
3. **Performance stability** — the render loop sustains 60 FPS at default settings on reference hardware.

Because the pure `render()` and `synth()` functions are by far the most logic-dense code paths, they are the natural unit-testing targets *if* testing is ever extracted into an automated harness. In v0.14, testing is manual but systematic.

---

## Manual Verification Checklist

Use this checklist when validating a new build or PR. All checks should pass before requesting review.

### Load & First Paint
- [ ] Page loads without console errors or warnings (besides known CDN/Tailwind dev messages).
- [ ] All fonts apply correctly (Space Grotesk for UI, Fira Code for mono).
- [ ] The CRT scanline overlay is visible on close inspection (subtle dark lines on the tensor canvas).
- [ ] Header badge reads `TACIT ARRAY v0.14`.
- [ ] Default preset is **Cymatic Lattice** (first preset button active/teal).
- [ ] Code buffer pre-populates with `` ÷2 ○ ×τ ⊞(○+) ×0.05 ⇡64 ×0.05 ⇡64 ``.
- [ ] Default parameter readouts: `220.00 Hz`, `64 × 64 cells`, `1.00×`.
- [ ] FPS counter starts incrementing within 1 second of load and stabilizes near the display refresh rate.

### Preset Switching
For each of the four presets (Cymatic, Fibonacci, Torus, Cellular):
- [ ] Clicking the preset button updates the code buffer text.
- [ ] Clicking updates the explanation panel text.
- [ ] Clicking updates the `FIELD:` HUD badge inside the canvas.
- [ ] Clicking applies the active button style (teal background/border) and removes it from the previous preset.
- [ ] The tensor field visually changes to reflect the new preset's topology.
- [ ] Stack inspector updates to reflect the new code content.
- [ ] If audio is playing, wavetable refreshes to the new preset without a crash.

### Glyph Palette
- [ ] All 24 glyphs render as buttons in the palette grid.
- [ ] Hovering any glyph shows its tooltip (name, glyph, arity) positioned above the button.
- [ ] Clicking a glyph inserts it at the caret in the code buffer.
- [ ] Inserting a glyph updates the stack inspector.
- [ ] Clicking multiple glyphs in sequence inserts them all without cursor jumps.

### Code Buffer & Evaluation
- [ ] Typing in the textarea works normally.
- [ ] Clicking **EVALUATE ⊞** briefly flashes a teal ring on the button (visual feedback).
- [ ] After evaluation, stack inspector updates to reflect the new code.
- [ ] Eval does not crash even with empty buffer or nonsensical input (graceful lexical matching).

### Parameter Sliders
For each slider (freq/res/speed):
- [ ] Dragging slider smoothly updates readout text.
- [ ] Dragging frequency slider updates synth wavetable (when audio is active) without crashes.
- [ ] Dragging resolution slider updates `[N N]` badge and re-renders field at new cell density.
- [ ] Dragging speed slider accelerates/decelerates the visual animation smoothly.
- [ ] Sliders respond to keyboard arrow keys after being tabbed to.
- [ ] No slider goes outside its labeled range (verify min/max readouts).

### Audio
- [ ] On first click of **SYNTHESIZE**, audio plays (requires user gesture).
- [ ] Audio does not autoplay on page load.
- [ ] After starting, mute label changes to `AUDIO DSP: ACTIVE`, button label changes to **HALT SYNTH** (amber).
- [ ] Oscilloscope switches from synthetic preview to live `AnalyserNode` waveform when audio plays.
- [ ] Waveform is visible in the scope strip and updates in real time.
- [ ] Clicking **HALT SYNTH** stops audio; mute label returns to `AUDIO DSP: MUTED`.
- [ ] Header mute button toggles audio identically to the synthesize button.
- [ ] No clipping/audible distortion at default volume.
- [ ] Audio loops continuously without dropping out for at least 30 seconds.
- [ ] Changing frequency while playing produces new timbre with no crash (sub-ms click is acceptable).
- [ ] Changing preset while playing produces new timbre with no crash.

### Oscilloscope
- [ ] Scope renders a continuous waveform trace when audio is muted (synthetic preview).
- [ ] Scope renders live analyser data when audio is active.
- [ ] Resizing the window horizontally rescopes the scope to match its container width.
- [ ] Center line (horizontal axis) is visible.

### Stack Inspector
- [ ] Default (Cymatic) stack shows at least TENSOR and SIGNAL entries.
- [ ] Stack depth counter matches number of items.
- [ ] Stack items show shape, kind, preview, and a tag chip.
- [ ] Clearing code buffer collapses to a single SCALAR entry.
- [ ] Typing ⇡ or "range" adds an INDEX entry.

### Responsive Layout
- [ ] At desktop width (>1024 px): left panel (7 cols) and right panel (5 cols) side-by-side.
- [ ] At narrow viewport (<1024 px): single-column stack; left panel first, right panel below.
- [ ] At very narrow viewport (<380 px): preset buttons reflow to 2 columns (or 1 at extreme widths); no horizontal scroll.
- [ ] No content overflows or gets clipped at common viewports: 1920×1080, 1440×900, 1280×800, 768×1024 (tablet), 390×844 (iPhone), 360×800 (Android).

### Animation & Visual Integrity
- [ ] Tensor animation is smooth (stutter indicates main-thread blocking).
- [ ] No visual tearing or artifacts at the tensor canvas edge.
- [ ] Colormap transitions smoothly across the purple-to-teal range — no abrupt bands.
- [ ] Gridlines overlay is subtle but visible; not overpowering the field.
- [ ] FPS counter reads 60 (or device refresh rate) at N=64; does not drop below ~45 even at N=128 on reference hardware.

---

## Browser Matrix

Test on at minimum the following before release:

| Browser | Version | Platform | Priority |
|---|---|---|---|
| Chrome | latest stable | macOS / Windows | P0 (reference) |
| Firefox | latest stable | macOS / Windows | P0 |
| Safari | latest stable | macOS | P0 |
| Edge | latest stable | Windows | P1 |
| Safari (iOS) | latest stable | iPhone | P1 |
| Chrome (Android) | latest stable | Android phone | P1 |
| Firefox ESR | ESR | Linux | P2 |

For PRs, testing on one P0 desktop browser is sufficient; maintainers will verify cross-browser before release.

---

## Performance Spot-Check

For any change that touches the render loop, audio fill loop, or adds new UI:

1. Open page, let it stabilize 5 seconds.
2. Open DevTools → Performance tab.
3. Record 5 seconds of idle animation at N=64.
4. Record 5 seconds at N=128.
5. Record 5 seconds while dragging each slider back and forth.
6. Verify:
   - FPS stays at target (60 @ 64, ≥48 @ 128).
   - No long tasks (>50 ms) on the main thread.
   - No GC pauses visible in the recording during idle animation.
   - Memory heap does not grow monotonically (should stabilize within a few MB).

Report any regressions in the PR description.

---

## Visual Regression (Manual)

Because the Lab uses deterministic math, visual output for a given `(preset, t, f, resolution)` is reproducible. When adding or modifying a preset, take a screenshot at t ≈ 0 (immediately after preset load with the default carrier 220 Hz) and compare against a reference if one exists. No automated screenshot diffing is set up in v0.14 (zero-dependency constraint), but contributors should be able to eyeball structural consistency.

For v0.15, a simple screenshot comparison tool (e.g., Playwright in a one-off script, not as a hard test dependency) may be introduced.

---

## Audio Regression (Listening Test)

When modifying synthesis:

1. For each preset, listen for at least 30 seconds.
2. Verify no obvious clipping, DC offset thumps, or discontinuities.
3. Verify loop seam (every 2 seconds) is inaudible for tonal presets (Cymatic, Torus) — a click/tic at the loop boundary is a regression unless the preset is explicitly textural (Cellular).
4. Frequency slider sweep from 55 Hz to 880 Hz should produce smooth pitch change without audio thread errors.
5. No sustained sub-bass energy that would distort laptop speakers (lowest fundamental is 55 Hz — acceptable).

---

## Unit / Invariant Testing (Future)

The pure function interfaces (`render`, `synth`, `colorMap`, `GLYPHS`, `PRESETS`) make natural unit-test targets. A future v0.16+ addition may include a small `test.html` that exercises:

- **Bounded-output invariants:**
  - For each preset, `render(u, v, t, f) ∈ [-1.2, 1.2]` across a sweep of inputs.
  - For each preset, `synth(time, f) ∈ [-1.0, 1.0]` across a 2-second sweep.
- **Glyph integrity:**
  - Every entry in `GLYPHS` has a single-character glyph, valid arity in {0,1,2}, and valid category.
- **Preset completeness:**
  - Every preset key has all six fields (title, code, doc, mode, render, synth).
  - `typeof render === 'function'`, `typeof synth === 'function'`.
- **Colormap sanity:**
  - `colorMap(-1)`, `colorMap(0)`, `colorMap(1)` return valid RGB triples with channels in [0,255].
  - `colorMap(x)` is monotonic in luminance across the range.

This would be implemented as a single `tests.html` (still zero-build) rather than introducing a Node test runner.

---

## CI / Automation

In v0.14 there is no CI pipeline (the project has no build step to gate). When GitHub Actions is introduced, the minimum viable CI checks would be:

1. **HTML validation** — run `index.html` through an HTML5 validator to catch malformed markup.
2. **Link check** — verify all `/docs/*.md` relative links resolve.
3. **Syntax check** — run the JavaScript portion through a lightweight parser (e.g., via `node --check` after extracting the script block) to catch syntax errors before merge.

This is tracked in [ROADMAP.md](ROADMAP.md) under v0.17 tooling.

---

## Filing Bugs Found During Testing

Bugs discovered during verification should be filed as issues using the [Bug Report template](../.github/ISSUE_TEMPLATE/bug_report.md), with:
- Browser/version/OS
- Steps to reproduce
- Preset + parameters
- Console errors (if any)
- Screenshot/recording for visual/audio bugs
