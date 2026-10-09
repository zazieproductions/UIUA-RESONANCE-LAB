# Audio Pipeline

This document explains the digital signal processing (DSP) architecture of Uiua Resonance Lab: wavetable synthesis, harmonic design per preset, gain staging, the analyser/oscilloscope path, and browser-specific considerations.

---

## DSP Graph Topology

```
┌────────────────────────┐
│ AudioBufferSourceNode  │     loop = true
│ (2 s mono wavetable)   │     playbackRate = 1.0
└───────────┬────────────┘
            │
            ▼
┌────────────────────────┐
│       GainNode         │     gain.value = 0.2   (master)
└───────────┬────────────┘
            │
     ┌──────┴──────┐
     ▼             ▼
┌──────────┐  ┌───────────────┐
│Analyser  │  │  destination  │ → speakers / headphones
│Node      │  │  (speakers)   │
│fftSize=256│ └───────────────┘
└─────┬────┘
      │
      ▼
 Oscilloscope Canvas
(getByteTimeDomainData)
```

This is a deliberately minimal graph. There are no filters, envelopes, LFO nodes, panners, or effects in the Web Audio graph itself — all timbral shaping is performed at wavetable-fill time by the pure `synth()` function of each preset. This keeps the graph predictable and the CPU footprint near zero.

---

## Wavetable Strategy

### Why wavetable (pre-rendered buffer) instead of `AudioWorklet` or `ScriptProcessorNode`?

Three reasons:

1. **Simplicity.** A pre-filled `AudioBuffer` with `loop = true` is approximately four lines of code and has zero per-sample JavaScript cost during playback.
2. **Determinism.** Because each preset's `synth()` function is pure, we can fill the buffer once and trust that looping repeats exactly — no GC pauses, no callback drift.
3. **Portfolio-scale sufficiency.** Four presets, static frequencies, no real-time modulation of timbre parameters (only frequency changes trigger a re-fill). A full AudioWorklet would be over-engineering for v0.14.

### Buffer Parameters

| Parameter | Value | Rationale |
|---|---|---|
| Sample rate | `audioCtx.sampleRate` (typically 44100 or 48000 Hz) | Use system-native rate for fidelity |
| Length | 2 seconds (`sampleRate * 2` samples) | Long enough for bell-decay presets to have a full ADSR-ish envelope before looping |
| Channels | 1 (mono) | No stereo field in v0.14 |
| Playback mode | `loop = true` | Seamless looping (preset-dependent) |

### Buffer Re-fill Strategy

The buffer is regenerated whenever the carrier frequency or preset changes (see `refreshAudioBuffer()`). The current implementation stops the node, rebuilds the buffer, and starts a new node — a blunt approach, but acceptable given:
- Frequency changes are user-initiated (slider drag), not continuous controller input.
- The gap between stop and start is sub-millisecond and only occurs on explicit user action.
- Future versions will use `AudioScheduledSourceNode` cross-fading for seamless parameter changes.

---

## Master Gain Staging

The master gain is set to **0.2** (≈ −14 dBFS). This is chosen because:

- The richest preset (Cymatic Lattice) sums three harmonics at relative amplitudes 0.5 + 0.25 + 0.15 = 0.9 peak — within ±1 but with headroom.
- Integer math and wavetable quantization can introduce up to ~1 dB of peak overshoot.
- A 0.2 master ceiling ensures even in worst-case presets, output stays roughly in the −10 to −14 dBFS range, leaving headroom for browser EQ, system volume, and user amplifiers without clipping.
- No normalization is performed — each preset's inherent loudness is preserved as part of its artistic character.

---

## Harmonic Design by Preset

See [PRESETS.md](PRESETS.md) for equations. Summarized here from a DSP perspective:

### Cymatic Lattice — Additive Organ
| Partial | Ratio | Amp | Role |
|---|---|---|---|
| Fundamental | 1.000f | 0.50 | Core pitch |
| Perfect fifth | 1.500f | 0.25 | Openness |
| Stacked ninth/octave | 2.250f | 0.15 | Air/shimmer |

Consonant harmonic series ⇒ rich, stable, organ-like. No beating. RMS ≈ 0.4.

### Fibonacci Bells — Inharmonic Decay
| Partial | Ratio | Amp | Envelope |
|---|---|---|---|
| Fundamental | 1.000f | 0.50 | Slow decay (1.5/s) |
| Golden clang | φ·f ≈ 1.618f | 0.30 | Fast decay (2.2/s) |
| Golden shimmer | φ²·f ≈ 2.618f | 0.20 | Fastest decay (3.5/s) |

Inharmonic ratios (based on φ) produce a metallic, bell-like beating pattern. The golden partial never aligns with the fundamental, creating the characteristic "WAR-bling" of a real struck bell.

### Phase Toroid — FM Sine
Single oscillator with linear frequency modulation from a 3 Hz LFO, ±45 Hz deviation:
- Deviation/carrier ratio at 220 Hz: ~0.2 ⇒ shallow vibrato.
- Output amplitude 0.6 ⇒ close to but not exceeding ±0.6 at master-gain input.
- Clean, theremin-like tone.

### Cellular Dilation — Arpeggiated Wave-Shaped
| Stage | Detail |
|---|---|
| Oscillator | Sine at f_k, where f_k cycles through just-interval ratios [1, 1.25, 1.333, 1.5, 1.875, 2] |
| Waveshaping | Hard `sin > 0 ? +0.35 : −0.35` (square-wave-ish) |
| Step rate | 16 steps/sec ⇒ 16th-note arpeggio at 240 BPM feel |
| Envelope | Per-step: exp(−20·t_mod), fast pluck shape |
| Loop artifact | Visible click at loop boundary is intentional/acceptable (textural) |

---

## Oscilloscope (Analyser Path)

### Configuration
- **AnalyserNode.fftSize = 256** → 128 frequency bins, 256 time-domain samples per frame.
- The oscilloscope reads time-domain data via `getByteTimeDomainData()`, returning a `Uint8Array(256)` of values in `[0, 255]` with 128 = zero crossing.

### Rendering
- When audio is playing: reads live analyser data and plots across the scope strip width.
- When muted: synthesizes a "preview" trace directly from `preset.synth()` at a scaled-down frequency (f·0.08) so the visual waveform animates in sync with the tensor field even without audio.

The oscilloscope canvas is resized to match its parent `<div>` dimensions on resize; DPR is not applied to keep strokes crisp and performant.

---

## Browser Compatibility Notes

| Browser | Web Audio Support | Notes |
|---|---|---|
| Chrome / Edge (≥ 90) | ✅ Full | Reference platform |
| Firefox (≥ 88) | ✅ Full | `AudioContext` still requires user gesture for `resume()` |
| Safari (≥ 14.1) | ✅ Full | WebKit prefix `webkitAudioContext` is handled via `window.AudioContext \|\| window.webkitAudioContext` |
| iOS Safari (≥ 14.5) | ✅ Full | Strictest autoplay policy — single user gesture *must* occur before `resume()`, which our code enforces |
| Old Edge (Legacy) / IE | ❌ | Not supported |

### Known Quirks
- **Chrome:** After tab suspension (tab discarded in background), `AudioContext` may transition to `interrupted` state. Toggling mute off/on recreates nodes if needed. Tracked as a future hardening item.
- **Firefox:** `AnalyserNode` time-domain data sometimes has minor DC offset at low buffer sizes; does not affect visual quality.
- **iOS Safari:** Master gain must be set before `start()` is called, which our `initAudio()` does. Changing gain after `start()` still works but may have a 10 ms ramp.

---

## Performance

| Cost | Quantity | Budget |
|---|---|---|
| Buffer fill at 44.1 kHz | ~88,200 samples per fill | ~5 ms on M1 (one-time) |
| Analyser FFT (256) | Per frame | Negligible (<0.1 ms) |
| Scope rendering | Per frame (rAF) | ~0.2 ms |
| Nodes in graph | 4 (source, gain, analyser, destination) | Trivial |
| Runtime JS per audio callback | 0 (buffered) | — |

After initial buffer fill, audio playback consumes effectively zero JavaScript CPU — the browser mixes the looping buffer on a real-time thread independent of the main thread. This is the core reason the render loop can maintain 60 FPS even while audio is active.

---

## Future Directions (Audio)

See [ROADMAP.md](ROADMAP.md) for milestones, but notable audio-specific directions:

1. **AudioWorklet-based real-time synthesis** for seamless parameter modulation (v0.17+).
2. **ADSR envelope** for percussive presets instead of static-loop buffer.
3. **Stereo field** with panning tied to X-axis of tensor.
4. **Parameter smoothing** (exponential ramps) on `frequency` slider changes to avoid clicks.
5. **Master effects chain** — convolution reverb (short plate), soft clipper/limiter, multimode filter.
6. **MIDI input** for external control of carrier frequency and preset switching.
7. **Record/export** — capture rendered audio as WAV for use in DAWs.

---

## Headphone & Speaker Recommendations

The Cymatic and Torus presets are best experienced on headphones or studio monitors with reasonable low-end extension (≥ 60 Hz), but nothing in the synthesis uses sub-bass energy (lowest fundamental is 55 Hz) so laptop speakers remain serviceable. The Cellular Dilation preset's high-frequency content can sound bright on aggressive tweeters — the master gain of 0.2 is tuned to avoid listener fatigue over extended sessions.
