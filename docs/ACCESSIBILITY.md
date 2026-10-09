# Accessibility

Uiua Resonance Lab is a visual-audio interactive artifact. Making it accessible requires attention to contrast, keyboard navigation, screen-reader semantics, motion sensitivity, and the needs of users who cannot rely on the visual or audio channel in isolation. This document records WCAG alignment, current status, and known gaps.

---

## WCAG 2.2 Alignment Summary

The project targets **WCAG 2.2 Level AA** conformance. Status is assessed below per-guideline.

### Perceivable

| Guideline | Status | Notes |
|---|---|---|
| 1.1.1 Non-text Content | 🟡 Partial | Canvas visualizations do not yet have text alternatives; see [Alt text for tensor field](#alt-text-for-tensor-field). All functional controls (buttons, inputs) have accessible names via visible text labels. |
| 1.2.1 Audio-only / Video-only | 🟢 N/A | No pre-recorded audio or video; synthesis is real-time and optional. |
| 1.3.1 Info and Relationships | 🟢 Good | Semantic HTML (`header`, `main`, `button`, `input`), proper heading hierarchy (only one `<h1>` per page), labels via visible text. |
| 1.3.2 Meaningful Sequence | 🟢 Good | DOM order matches visual reading order. |
| 1.3.3 Sensory Characteristics | 🟢 Good | Controls are not identified by shape/color/location alone — they all have visible text labels. |
| 1.4.1 Use of Color | 🟢 Good | Status indicators (active preset, audio state) use both color and text labels ("AUDIO DSP: ACTIVE", "Cymatic Lattice"). |
| 1.4.2 Audio Control | 🟢 Good | Audio does not auto-play; a prominent toggle stops it within one click. |
| 1.4.3 Contrast (Minimum) | 🟢 Pass | All UI text meets 4.5:1 contrast against its background. Body text is `#e2e8f0` on `#090a0f` (≈ 14:1). Teal accent text `#5ef1d2` on `#090a0f` measures ≈ 10:1. |
| 1.4.4 Resize Text | 🟢 Good | Layout uses rem/relative units where practical; text scales to 200% without horizontal scrolling. |
| 1.4.10 Reflow | 🟡 Partial | Layout is responsive via Tailwind grid but has not been audited at 320px viewport width with 400% zoom exhaustively. |
| 1.4.11 Non-text Contrast | 🟢 Good | Buttons and focus rings meet 3:1 contrast against adjacent backgrounds. |
| 1.4.12 Text Spacing | 🟢 Good | No fixed text containers that break with increased letter/word spacing. |
| 1.4.13 Content on Hover/Focus | 🟢 Good | Glyph hover tooltips are dismissible, hoverable, and persistent (CSS `:hover` naturally). |

### Operable

| Guideline | Status | Notes |
|---|---|---|
| 2.1.1 Keyboard | 🟢 Good | All buttons and sliders are native HTML elements, reachable via Tab, operable via Enter/Space/Arrow keys. |
| 2.1.2 No Keyboard Trap | 🟢 Good | No focus traps; Tab order flows naturally. |
| 2.3.1 Three Flashes / Threshold | 🟢 Pass | No flashing content; animations are smooth continuous fields with no stroboscopic components. |
| 2.3.3 Animation from Interactions | 🟡 Partial | The CRT scanline effect and glow pulses are subtle decorative animations that can be disabled via `prefers-reduced-motion` — see [Reduced Motion](#reduced-motion). |
| 2.4.3 Focus Order | 🟢 Good | Tab order matches visual order (header → preset buttons → code buffer → evaluate/synthesize buttons → sliders → stack). |
| 2.4.7 Focus Visible | 🟢 Good | Tailwind's default focus ring plus explicit `focus:ring-teal-400/50` and `focus:border-teal-400` on the textarea; native buttons show default focus indicators. |
| 2.5.1 Pointer Gestures | 🟢 Good | No multipoint or path-based gestures required. |
| 2.5.4 Motion Actuation | 🟢 N/A | No motion/tilt controls. |

### Understandable

| Guideline | Status | Notes |
|---|---|---|
| 3.1.1 Language | 🟢 Good | `<html lang="en">` set. |
| 3.2.1 On Focus | 🟢 Good | No unexpected context changes on focus. |
| 3.2.2 On Input | 🟢 Good | Changing sliders updates previews in-place; no navigation or focus jumps. |
| 3.3.1 Error Identification | 🟢 N/A | No form submission / no errors to display. |

### Robust

| Guideline | Status | Notes |
|---|---|---|
| 4.1.1 Parsing | 🟢 Good | Valid HTML5; single root, properly nested elements. |
| 4.1.2 Name, Role, Value | 🟡 Partial | Native controls have implicit roles/names; canvas and oscilloscope regions have adjacent labels but no `aria-label` directly on the canvases (see [Canvas A11y](#canvas-accessibility)). |

---

## Color & Contrast

### Palette Analysis

All UI colors come from CSS custom properties defined in `:root`:

| Token | Value | Contrast on `--bg` (#090a0f) | Use |
|---|---|---|---|
| `--accent` | `#5ef1d2` | ≈ 10.1:1 AAA | Primary actions, active glyph, teal text |
| `--accent-dim` | `#256658` | ≈ 3.2:1 | Glow borders only (not text) |
| `--amber` | `#f59e0b` | ≈ 8.1:1 AAA | Time/temporal indicators |
| `--magenta` | `#ec4899` | ≈ 5.4:1 AA | Velocity/speed |
| `--purple` | `#a855f7` | ≈ 5.3:1 AA | Resolution / secondary |
| Body text | `#e2e8f0` | ≈ 14.0:1 AAA | Main copy |
| Muted text | `#94a3b8` | ≈ 6.2:1 AA | Secondary labels |
| Border | `#1f2333` | ≈ 1.2:1 | Decorative only (with secondary border/background cues) |

### Contrast Audit Method
All text/background pairs were verified with the WebAIM Contrast Checker (WCAG AA requires 4.5:1 for body text, 3:1 for large text). The `--accent-dim` token is explicitly not used for text; it appears only as a glow/box-shadow.

### Color Vision Deficiency
The primary teal/magenta/purple distinction is also supplemented by:
- **Text labels** (not color alone).
- **Distinct icon shapes / positions** (each HUD badge has a different position).
- **Mono-coded tags** in the stack inspector (TENSOR, SIGNAL, INDEX, SCALAR).

---

## Keyboard Navigation

All interactive elements are native HTML controls. Tab order (DOM order):

1. Mute/Audio toggle button (header)
2. Preset selector buttons × 4
3. Evaluate button
4. Synthesize/Halt button
5. Code textarea
6. Glyph palette buttons × 24
7. Frequency slider
8. Resolution slider
9. Speed slider
10. Uiua.org external link

- **Buttons** activate on Enter or Space (native behavior).
- **Sliders** (`<input type="range">`) adjust via arrow keys (native behavior, ±0.5 Hz / ±8 cells / ±0.05×).
- **Textarea** is a standard text input; users can type or paste glyphs directly; Tab moves focus out of the textarea (browser default).
- **Focus is always visible** via the browser's default focus ring or explicit Tailwind focus styles.

### Known Keyboard Issues
- There is no **keyboard shortcut** to jump between presets, toggle audio, or evaluate. This is on the roadmap (v0.16).
- The glyph palette contains 24 buttons; tabbing through them is tedious. A future improvement may introduce a left/right arrow-key grid navigation pattern.

---

## Reduced Motion

The Lab uses `requestAnimationFrame` for its core tensor animation — disabling this would fundamentally break the application (the tensor field *is* the animation). However, the decorative non-essential animations should honor `prefers-reduced-motion`:

```css
@media (prefers-reduced-motion: reduce) {
  .pulsing-dot { animation: none; opacity: 1; }
  .glyph-btn { transition: none; }
  /* Canvas animation is essential motion — continues but at a slower rate could be offered */
}
```

Currently, this media query is **not yet applied** — it's a documented gap slated for v0.15. The core tensor animation is essential to the experience (it's an instrument, not a decorative page), but the pulsing glow, hover transitions, and animate-pulse effects are decorative and should be disabled.

---

## Canvas Accessibility

The tensor field and oscilloscope are rendered into `<canvas>` elements, which are not inherently accessible to screen readers.

### Current State
- **Adjacent labels:** Each canvas has a visible text label immediately above it ("Tensor Spatial Manifest", "Harmonic Waveform Slice") plus in-canvas HUD overlays (`FIELD: …`, `Amp: …`).
- **FPS counter and matrix-shape** readouts are text nodes that screen readers can pick up.
- **No `aria-label`** on the canvas elements themselves.
- **No `role="img"`** or descriptive text alternative for the tensor field.

### Plan
For a future release:
1. Add `role="img"` to the tensor canvas and an `aria-label` describing the active preset (e.g., "Cymatic lattice: a 64-by-64 two-dimensional wave interference pattern animating at 220 Hz carrier frequency").
2. Provide an `aria-live` region that periodically announces significant state changes (preset changed, frequency changed, audio toggled) without being verbose.
3. Consider a "Sonify Only" mode where the audio represents the field without requiring sight (the audio already does this, but the UI could make it explicit).

---

## Screen Reader Testing (Informational)

Informal testing was conducted with:

| Screen Reader | Browser | Observations |
|---|---|---|
| VoiceOver (macOS 14) | Safari 17 | Reads all buttons, sliders, labels in correct order. Canvas elements are announced as "image" without alt text (gap). |
| NVDA 2024.1 | Firefox 126 | Similar to VoiceOver; sliders announce current values correctly. |
| TalkBack (Android 14) | Chrome | Focus order works; touch exploration finds all controls. |

Formal screen-reader testing with real users is welcomed.

---

## Audio Accessibility

- Audio never auto-plays (complies with WCAG 1.4.2 and browser autoplay policies).
- A prominent mute toggle is available in the header at all times.
- The master gain is set conservatively (−14 dBFS) to avoid startling users when first engaged.
- No audio content carries essential information that isn't also represented visually (the visual field is the "information" — audio is a complementary aesthetic channel).

### For Deaf/HoH Users
The application is fully usable without audio — the visual tensor field, all controls, the oscilloscope (which still renders a synthetic preview trace when muted), and all status readouts function identically.

---

## Responsive Behavior

- Layout collapses to single-column below the `lg` Tailwind breakpoint (≈ 1024 px) via the `lg:col-span-*` grid system.
- The engine badge in the header (`hidden sm:flex`) is hidden on very small viewports but all functional controls remain accessible.
- Sliders, buttons, and the textarea all use touch-friendly sizes (per WCAG 2.5.5 Target Size — at least 24×24 CSS pixels; all interactive controls exceed this).
- Font sizes use relative units and respect user default font-size settings.

---

## Known Gaps & Roadmap

| Issue | WCAG | Target |
|---|---|---|
| `prefers-reduced-motion` not wired to decorative animations | 2.3.3 | v0.15 |
| Canvas `aria-label` and `role="img"` | 1.1.1 / 4.1.2 | v0.15 |
| No keyboard shortcuts for preset/eval/mute | 2.1 (enhancement) | v0.16 |
| 24-glyph palette is slow to tab through | 2.4.3 (enhancement) | v0.16 |
| 400% zoom reflow not exhaustively tested | 1.4.10 | v0.15 |
| No audio-description track (N/A for realtime; but written descriptions in presets doc compensate) | 1.2.3 | Documentation addresses it |
| Screen-reader verbosity tuning | 4.1.3 | v0.16 |

---

## Reporting Accessibility Issues

If you encounter an accessibility barrier — difficulty navigating with a keyboard, contrast issue, screen-reader confusion, or motion sensitivity problem — please file an issue using the Bug Report template and tag it `a11y`. Accessibility bugs are triaged at the highest priority alongside correctness and security issues.
