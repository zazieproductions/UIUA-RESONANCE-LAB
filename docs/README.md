# Documentation

Technical documentation for **Uiua Resonance Lab v0.14**. Start with the [Architecture overview](ARCHITECTURE.md) for the system-level view, or dive into the area you need.

---

## Core

- [Architecture](ARCHITECTURE.md) — System diagram, module boundaries, data flow, rendering pipeline, state model, coordinate spaces.
- [API Reference](API.md) — Internal contracts: glyph schema, preset interface, `render`/`synth` signatures, color map, DOM helpers, audio engine, events.

## Language & Content

- [Glyph Dictionary](GLYPHS.md) — All 24 Uiua-inspired primitives with arity, signature, category, and examples.
- [Preset Catalog](PRESETS.md) — Mathematical breakdown of each of the four shipping artifacts, including equations and harmonic structure.
- [Audio Pipeline](AUDIO.md) — DSP graph, wavetable strategy, gain staging, oscilloscope path, browser quirks.

## Quality & Operations

- [Performance](PERFORMANCE.md) — Frame budgets, allocation profile, scaling characteristics, applied optimizations, future improvements.
- [Accessibility](ACCESSIBILITY.md) — WCAG 2.2 AA alignment, contrast audit, keyboard navigation, reduced-motion plan, screen-reader status.
- [Testing](TESTING.md) — Manual verification checklist, browser matrix, performance spot-check protocol, future automated testing plan.
- [Deployment](DEPLOYMENT.md) — Static hosting options, GitHub Pages workflow, cache strategy, security headers, release checklist.

## Planning

- [Roadmap](ROADMAP.md) — Versioned forward plan from v0.15 through v2.0, including the real evaluator (v1.0) and WebGPU horizon (v2.0).

---

## Reading Order

**If you're a recruiter or engineer reviewing the project:**
1. [README](../README.md) → 2. [Architecture](ARCHITECTURE.md) → 3. [API](API.md) → 4. [Performance](PERFORMANCE.md)

**If you want to add a preset:**
1. [CONTRIBUTING.md](../CONTRIBUTING.md) → 2. [Preset Catalog](PRESETS.md) → 3. [API: Preset Interface](API.md#preset-interface) → 4. [Testing](TESTING.md)

**If you're learning from the code:**
1. [Architecture](ARCHITECTURE.md) → 2. [Glyph Dictionary](GLYPHS.md) → 3. [Audio Pipeline](AUDIO.md) → 4. Read `index.html` from top to bottom.

---

## Conventions Used in These Docs

- Code references are `monospace`.
- Glyphs are rendered as their Unicode characters: ⊞ ○ ∿ ⇡ ⇌.
- Stack signatures use bottom-to-top notation, matching Uiua convention.
- "v0.14" refers to the current release; "v1.0" refers to the planned real-evaluator milestone.
- Status markers: 🟢 passing/good · 🟡 partial/known gap · ❌ missing/broken.
