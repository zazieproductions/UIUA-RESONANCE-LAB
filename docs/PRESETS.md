# Preset Catalog

Each preset in Uiua Resonance Lab is a self-contained audiovisual artifact defined by two pure functions — a `render(u, v, t, f)` mapping tensor coordinates to a scalar field, and a `synth(time, f)` mapping sample time to a waveform. This document describes the mathematical and acoustical design of each shipping preset.

---

## Preset Anatomy Recap

```javascript
{
  title: 'Human Name',
  code: '⊞○ ... ⇡64',     // Glyph expression (displayed in code buffer)
  doc: 'Plain-text description',
  mode: 'HUD label',
  render: (u, v, t, f) => scalar,  // [-1, 1]
  synth: (time, f)   => sample     // [-1, 1]
}
```

See [API.md](API.md#preset-interface) for the formal contract.

---

## 01 — Cymatic Lattice

**Key:** `cymatics`
**Mode:** 2D Wave Interference
**Code:** `` ÷2 ○ ×τ ⊞(○+) ×0.05 ⇡64 ×0.05 ⇡64 ``
**Glyph signature:** `⊞ ○ + ×τ ×0.05 ⇡`

### Visual Topology

A Chladni-plate-style interference pattern formed by the superposition of two traveling waves at a right angle, with a diagonal cross-term adding a third interference axis.

```
render(u, v, t, f) =
    sin(u·ω + t) · cos(v·ω − 0.7t)
  + 0.5 · sin((u+v)·ω·0.6 + 1.4t)
```

where `ω = f / 80`.

- The first term produces a tilted standing wave with counter-propagating phase between X and Y axes.
- The second term adds a diagonal shear component at 60% frequency and 140% phase velocity, breaking rectangular symmetry and creating organic, quasi-cymatic nodal clusters.
- Result: a constantly morphing lattice of resonance peaks and nodal lines, reminiscent of sand on a vibrating plate.

### Harmonic Structure

Additive synthesis with three harmonics:

```
synth(time, f) =
    0.50 · sin(2π·f·time)        // fundamental
  + 0.25 · sin(2π·1.5f·time)     // perfect fifth
  + 0.15 · sin(2π·2.25f·time)    // harmonic stack
```

A consonant, organ-like timbre. The 1.5f partial (perfect fifth) and 2.25f (ninth + octave) produce a bright, open tonal quality that pairs with the visual lattice.

### Parameter Response
- **Freq:** Raises ω, denser nodal pattern and higher pitch.
- **Resolution:** Finer/denser grid rendering (no effect on synth).
- **Speed:** Scales the 2 rad/sec base phase rate; higher values produce frenetic shimmer.

---

## 02 — Fibonacci Subharmonic Bells

**Key:** `subharmonic`
**Mode:** Golden Tensor Decay
**Code:** `` ⍚×ⁿ ÷44100 ⇡44100 ○ ×τ ×[220 330 550 880] ``
**Glyph signature:** `×ⁿ ÷ ⇡ ○ ×τ ×`

### Visual Topology

A radially symmetric field based on the golden ratio φ = (1+√5)/2 ≈ 1.618, with exponential radial decay:

```
render(u, v, t, f) =
  sin(r·0.2·φ − 2t) · exp(−((r·0.03) mod 1.5))
```

where `r = √(u² + v²)`, `φ = (1+√5)/2`.

- Radial waves propagate outward from the center at a golden-ratio wavelength, creating bell-like concentric rings.
- The modulo-exponential envelope creates a repeating "ripple decay" pattern that mimics the amplitude envelope of a struck bell.
- The φ-based wavelength ensures that rings never perfectly align on rational multiples, producing the characteristic beating/inharmonic shimmer of real bells.

### Harmonic Structure

Inharmonic additive synthesis with golden-ratio partials and time-varying decay:

```
synth(time, f) = decay · (
    0.50 · sin(2π·f·time)                          // fundamental
  + 0.30 · sin(2π·f·φ·time) · exp(−2.2·time)       // golden partial
  + 0.20 · sin(2π·f·φ²·time) · exp(−3.5·time)      // double-golden partial
)
where decay = exp(−1.5·time)
```

- The fundamental decays slowly, while the higher (golden-ratio) partials bloom and die faster, mimicking a bell's initial "clang" followed by the sustained hum of the fundamental.
- Note: because the wavetable is statically filled for 2 seconds and looped, the decay "restarts" every loop period. For a v0.14 portfolio piece this is acceptable; a future version will use a dynamics-aware scheduler.

### Parameter Response
- **Freq:** Shifts the golden-ratio fundamental.
- **Speed:** Rotates the radial pattern faster; bells appear to "spin."

---

## 03 — Phase Toroid Drift

**Key:** `torus`
**Mode:** Clifford Torus Angle Field
**Code:** `` ⊞∡ ⇌ ⇡64 ⇡64 ∺ ○ ×0.1 ⇡64 ``
**Glyph signature:** `⊞∡ ⇌ ⇡ ∺ ○ ×`

### Visual Topology

A winding phase field derived from the polar angle of the coordinate vector:

```
render(u, v, t, f) =
  sin(4·θ + 0.15·r − 2.5t)
```

where `θ = atan2(v, u)`, `r = √(u² + v²)`.

- The fourfold angular term (`4θ`) creates four spiral arms radiating from the center.
- The radial term (`0.15r`) adds a gentle twist that shears the arms, evoking the seam on a Clifford torus projected onto the plane.
- Time integration gives a continuous apparent rotation about the center.

### Harmonic Structure

Simple frequency-modulated sine with a slow LFO:

```
synth(time, f) =
  0.6 · sin(2π·(f + 45·sin(3·time))·time)
```

- A 3 Hz LFO modulates carrier frequency by ±45 Hz, producing a gentle siren/vibrato effect that evokes the toroidal "drift" of the visual.
- Single-oscillator design keeps the timbre clean and theremin-like, matching the geometric purity of the spiral.

### Parameter Response
- **Freq:** Raises center carrier.
- **Speed:** Increases drift rate — both visual rotation and perceived LFO rate scale together (audio doesn't change, but the visual syncs to speed).

---

## 04 — Cellular Dilation Matrix

**Key:** `stochastic`
**Mode:** Minkowski Crystal Automata
**Code:** `` ⍥(♭ ⊞+ ⌕ ▽) 4 ⇡64 ``
**Glyph signature:** `⍥ ♭ ⊞+ ⌕ ▽ ⇡`

### Visual Topology

A deterministic block-cellular pattern driven by bitwise XOR on grid coordinates and time:

```
render(u, v, t, f) =
  sign(cx·37 XOR cy·59 XOR ⌊8t⌋ > 3)
  · sin(0.1u) · cos(0.1v)
```

where `cx = ⌊(u+32)/8⌋`, `cy = ⌊(v+32)/8⌋`.

- Coordinates are quantized to 8-unit blocks, creating an 8×8 macro-cell grid over the tensor field.
- A cheap integer hash (37 and 59 are common hash-function multipliers) XORed with a quantized time counter produces a deterministic "flipping" pattern — every ~125 ms (at speed 1), a new set of blocks inverts polarity.
- The global `sin(0.1u)·cos(0.1v)` envelope ensures the field remains smoothly varying *within* each block while block boundaries flip discretely — the "Minkowski dilation" effect where flat facets continuously mutate their sign.

### Harmonic Structure

A square-wave-like arpeggiator cycling through a fixed frequency ratio series:

```
synth(time, f) =
  (sin(2π·f_k·time) > 0 ? 0.35 : -0.35)
  · exp(−(time mod 0.0625)·20)
```

where `f_k = f · ratios[⌊16·time⌋ mod 6]` and `ratios = [1, 1.25, 1.333, 1.5, 1.875, 2]`.

- The ratios form just-interval steps (unison, major third, perfect fourth, perfect fifth, major seventh, octave) producing a crystalline, constantly shifting melodic pattern.
- Each 62.5 ms "step" triggers a fast exponential decay envelope (20/sec), mimicking a plucked/struck crystalline texture.
- The hard-clipped sine gives a square-wave-ish timbre without actual hard clipping at the DAC (the waveshaping is kept within ±0.35).

### Parameter Response
- **Freq:** Transposes the entire arpeggio.
- **Speed:** Increases the visual block-flip rate (the audio arpeggio rate is currently tied to sample time, not speed — a known v0.14 limitation).

---

## Preset Comparison Matrix

| Preset | Field Type | Visual Symmetry | Synthesis | Timbre |
|---|---|---|---|---|
| Cymatic Lattice | Wave interference | Rectangular lattice + diagonal shear | 3-part additive | Organ-like, consonant |
| Fibonacci Bells | Radial golden-ratio | Radial, aperiodic rings | Inharmonic + decay | Bell-like, clanging |
| Phase Toroid | Polar angular | 4-fold rotational spiral | FM sine (LFO) | Theremin-like, drifting |
| Cellular Dilation | Block-cellular hash | 8×8 macro-cell facets | Arpeggiated square-waveshaped | Crystalline, percussive |

---

## Composing a New Preset (Checklist)

When designing a new preset for contribution (see [CONTRIBUTING.md](../CONTRIBUTING.md#adding-a-new-preset)):

1. **Math first.** Write the field equation before touching code. Verify it produces values approximately in `[-1, 1]`.
2. **Pure functions.** `render` and `synth` must be referentially transparent — no closure over state, no randomness.
3. **Coordinate sanity.** Use `u, v ∈ [-10, 10]` as your canvas; remember the default ω for cymatics is `f/80` ≈ 2.75 as a reference wavelength.
4. **Audio bounds.** Synth must return values in `[-1, 1]`. If multiple harmonics sum, attenuate each so the sum stays bounded (e.g., 0.5 + 0.25 + 0.15 = 0.9).
5. **Loop continuity.** For tonal presets, ensure `synth(0, f) ≈ synth(2, f)` to avoid a 2-second loop click. Textural/percussive presets can relax this.
6. **Glyph expression.** Write a plausible Uiua-style tacit expression in `code` that hints at the array operations involved. You do not need to write a *runnable* Uiua program, but it should be syntactically suggestive.
7. **Documentation.** Add a section to this document with the equations and description, in the style above.
