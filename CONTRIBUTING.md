# Contributing

First off — thank you for considering a contribution to **Uiua Resonance Lab**. This project is a creative-coding artifact built with rigor, and every glyph, preset, or refactor that lands here should hold to the same bar: mathematically coherent, architecturally minimal, and aesthetically considered.

The following guidelines exist to make contributions predictable and reviewable, not to gatekeep. Use judgment; exceptions exist for good reasons — explain them in your PR description.

---

## Code of Conduct

This project adheres to the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to the maintainers via GitHub or email.

---

## Ways to Contribute

| Kind | Examples |
|---|---|
| **Bug reports** | Rendering glitches, audio crackling, stack-state mismatches, mobile layout breakage |
| **New presets** | Novel mathematical artifacts (toroidal knots, reaction-diffusion approximations, fractal interference) |
| **New glyphs** | Additional Uiua primitives with correct stack semantics and hover docs |
| **Performance** | Frame-budget improvements, rasterization optimizations, memory reductions |
| **Accessibility** | Keyboard navigation gaps, contrast issues, screen-reader semantics |
| **Documentation** | Typo fixes, clarifications, new diagrams, architecture expansions |
| **Tooling** | Local-dev scripts, GitHub Actions workflows, linting configuration |

---

## Development Setup

This project is intentionally tool-free. You do not need Node, Python, or any build tool to hack on it.

```bash
# 1. Fork and clone
git clone https://github.com/<your-username>/UIUA-RESONANCE-LAB.git
cd UIUA-RESONANCE-LAB

# 2. Create a feature branch
git checkout -b feat/your-feature-name

# 3. Serve locally (any static server works)
python3 -m http.server 8080
# or: npx serve .
# then open http://localhost:8080

# 4. Make your changes in index.html (or docs/)
# 5. Verify against the checklist in docs/TESTING.md
# 6. Commit and push
git add .
git commit -m "feat: concise imperative summary"
git push origin feat/your-feature-name

# 7. Open a Pull Request against main
```

---

## Coding Conventions

### General
- **Zero dependencies.** Do not introduce npm packages, CDN libraries, or framework imports unless the change is discussed in an issue and accepted. Tailwind via CDN is already approved.
- **Single-file constraint.** Runtime logic belongs in `index.html`. Do not split JavaScript into separate `.js` files; this project's portability guarantee requires a single deployable artifact. Documentation and meta-files (`.github/`, `docs/`) are exempt.
- **Modern JS.** Use ES2020+ features (`const`/`let`, arrow functions, template literals, destructuring, optional chaining where useful). Avoid transpilation-era patterns (`var`, `function` declarations for closures).

### JavaScript
- **Indentation:** 2 spaces.
- **Quotes:** Single quotes for strings; backticks only for template literals.
- **Semicolons:** Use them.
- **Naming:** `camelCase` for variables and functions, `UPPER_SNAKE_CASE` for constants (e.g., `GLYPHS`, `PRESETS`), descriptive over terse (`currentPreset` not `cp`).
- **Pure functions where possible.** Preset `render` and `synth` functions must be referentially transparent — no mutation of outer state, no DOM access, no randomness without deterministic seeding.
- **State locality.** Keep mutable state declared at the top of the `<script>` block in a single `State` region (see `index.html` comments). Do not scatter `let` bindings across the file.

### CSS
- **Design tokens live in `:root`** as CSS custom properties. New palette entries must be added to the token layer, not hard-coded.
- **Prefer Tailwind utility classes** for layout and spacing; use scoped `<style>` blocks only for effects that Tailwind cannot express (CRT scanlines, keyframe animations, custom scrollbars, font imports).
- **Respect the color system** — teal (`--accent`) for primary/active states, amber for temporal indicators, purple/magenta for secondary and modulation channels, slate greys for chrome.

### HTML
- **Semantic elements first:** `<header>`, `<main>`, `<section>`, `<button>`, `<label>` — not clickable `<div>` soup.
- **All interactive controls** must be real `<button>` or `<input>` elements with discernible accessible names.
- **Language attribute** (`<html lang="en">`) and `<meta charset>` must remain on the root.

### Documentation
- Markdown follows [CommonMark](https://commonmark.org/) spec.
- Line length of 100 characters in prose where feasible.
- Code blocks must specify a language for syntax highlighting.
- ASCII diagrams are encouraged; use them to clarify data flow.

---

## Commit Message Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/) where practical:

```
<type>(<scope>): <imperative summary>

<body — optional, explains why>
```

**Types:** `feat`, `fix`, `docs`, `perf`, `refactor`, `a11y`, `test`, `chore`, `style`

**Examples:**
```
feat(preset): add toroidal-knot interference preset
fix(audio): prevent buffer-source double-start on rapid toggle
docs(glyphs): document arity-0 constant τ and η
perf(render): reduce overdraw via integer cell-bound computation
```

---

## Pull Request Process

1. **Open an issue first** for non-trivial changes (new presets, architectural shifts, new dependencies). Trivial fixes (typos, obvious bugs) can skip the issue.
2. **PRs should be small and focused.** One concern per PR. A PR that refactors the render loop *and* adds three new presets will be asked to split.
3. **Describe the change.** Include:
   - What changed and why
   - Screenshots/GIFs for visual changes
   - Any new glyph or preset's mathematical basis
   - Browser/device tested on
4. **Run the manual test checklist** in [docs/TESTING.md](docs/TESTING.md) before requesting review. Note any items you could not verify.
5. **Update documentation.** New glyphs → update `docs/GLYPHS.md`. New presets → update `docs/PRESETS.md`. Architectural changes → update `docs/ARCHITECTURE.md`.
6. **Maintainer review.** Expect feedback within a week. Address review comments with additional commits; do not force-push during review unless asked.
7. **Merge strategy.** PRs are squashed into `main` with a conventional-commit subject line.

---

## Adding a New Preset

Presets are the most common contribution. Follow this contract exactly:

```javascript
{
  key: {
    title: 'Display Name',
    code: '`uiua-glyph-expression`',
    doc: 'One-sentence mathematical / physical description.',
    mode: 'Short render-mode label for the HUD',
    render: (u, v, t, f) => { /* return scalar in [-1, 1] */ },
    synth: (time, f) => { /* return sample in [-1, 1] */ }
  }
}
```

**Rules:**
- `render(u, v, t, f)` must be pure and return a value approximately bounded in `[-1, 1]` for sensible colormapping.
- `synth(time, f)` must be pure and return a sample bounded in `[-1, 1]` to avoid clipping at the gain stage.
- Register the preset in the `PRESETS` object **and** add a corresponding `.preset-btn` to the selector HTML (preserving the 01–04 enumeration pattern).
- Document the preset in [docs/PRESETS.md](docs/PRESETS.md), including the mathematical formulation and harmonic structure.

---

## Adding a New Glyph

Glyphs encode real Uiua primitives (see [uiua.org/docs](https://www.uiua.org/docs)). Do not invent fictional primitives — the project's credibility depends on glyph fidelity to the actual language.

Each glyph entry must conform to:

```javascript
{
  glyph: '⊞',       // Single Unicode glyph character
  name: 'table',    // Lowercase Uiua primitive name
  doc: '…',         // One-line documentation, accurate to Uiua semantics
  arity: 2,         // 0 = constant, 1 = monadic, 2 = dyadic, 2+ for modifiers
  cat: 'array'      // One of: array, math, transform, modifier, inspect, search, filter
}
```

After adding, update [docs/GLYPHS.md](docs/GLYPHS.md) with the new entry.

---

## Reporting Bugs

Use the [Bug Report template](.github/ISSUE_TEMPLATE/bug_report.md). A good bug report includes:

- Steps to reproduce
- Expected vs. actual behavior
- Browser, version, OS, device
- Screenshot or screen recording if visual/audio
- Whether the issue reproduces in an incognito window (rules out extensions)

---

## Questions?

Open a [Discussion](https://github.com/zazieproductions/UIUA-RESONANCE-LAB/discussions) or ask in an issue. We're happy to help new contributors get oriented.

⊞ ○ ∿ — happy hacking.
