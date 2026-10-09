# Uiua Glyph Dictionary

This document catalogs every glyph exposed in the Lab's clickable palette. Each entry reflects a real primitive from the [Uiua array programming language](https://uiua.org/docs); the descriptions here summarize semantics as they are presented in the Lab's educational UI.

> **Note:** The v0.14 Lab does not ship a real Uiua parser or evaluator. Glyphs are presented for educational, aesthetic, and interface-building purposes. The stack inspector responds to lexical matches to convey tacit-programming intuition. A live evaluator is planned for v1.0 (see [Roadmap](ROADMAP.md)).

---

## Reading a Glyph Entry

Every entry lists:

- **Glyph** — the Unicode character rendered in the palette.
- **Name** — the Uiua primitive identifier.
- **Arity** — number of stack arguments consumed (0 = constant, 1 = monadic function, 2 = dyadic function, "modifier" = higher-order combinator).
- **Category** — functional family (array, math, transform, modifier, inspect, search, filter).
- **Signature** — stack-effect notation (bottom-to-top, like Uiua itself).
- **Description** — semantics in one sentence.
- **Example (Uiua)** — a tiny illustrative expression where appropriate.

---

## Array Constructors & Combinators

### ⊞ · table
- **Arity:** 2 (dyadic)
- **Category:** array
- **Signature:** `f A B → C` — applies a dyadic function `f` between every element pair of `A` and `B` (the outer/tensor/table product)
- **Description:** Lifts a dyadic operation into the cross product of two arrays, producing a matrix whose `[i,j]` entry is `f(A[i], B[j])`. The workhorse for 2-D field generation in the Lab.
- **Example:** `⊞+ ⇡4 ⇡3` produces a 4×3 addition table.

### ⇡ · range
- **Arity:** 1 (monadic)
- **Category:** array
- **Signature:** `n → [0 1 2 … n-1]`
- **Description:** Generates an integer array from 0 to n−1. The foundational index vector in Uiua.
- **Example:** `⇡5` yields `[0 1 2 3 4]`.

### ∺ · each (per element)
- **Arity:** modifier (1 function + 1 array)
- **Category:** modifier
- **Signature:** `f A → B` — applies `f` to each element of `A`
- **Description:** Maps a function element-wise across an array. The "map" combinator of the array world.

### ⊂ · join
- **Arity:** 2 (dyadic)
- **Category:** array
- **Signature:** `A B → C` — concatenates two arrays along their leading axis
- **Description:** Appends `B` after `A` without nesting.

### ∺ · stitch (`∺` in the palette)
- **Arity:** 2 (dyadic)
- **Category:** array
- **Signature:** `A B → C` — pairs matching slices (rows) of `A` and `B` together
- **Description:** Interleaves corresponding rows; useful for building coordinate pairs from separate axis vectors.

### ◰ · classify
- **Arity:** 1 (monadic)
- **Category:** array
- **Signature:** `A → B` — groups identical elements into equivalence classes (rank groups)
- **Description:** Produces a group-identifier for each unique value, analogous to "label connected components."

### ♭ · flatten
- **Arity:** 1 (monadic)
- **Category:** array
- **Signature:** `A → v` (vector)
- **Description:** Collapses all array axes into a single 1-D vector in row-major order.

---

## Structural Transforms

### ⇌ · reverse
- **Arity:** 1 (monadic)
- **Category:** transform
- **Signature:** `A → B`
- **Description:** Reverses elements along the leading axis.

### ⍉ · transpose
- **Arity:** 1 (monadic)
- **Category:** transform
- **Signature:** `M → Mᵀ`
- **Description:** Permutes array axes (matrix transpose in 2-D; generalized axis permutation for higher ranks).

---

## Math Primitives

### ○ · sine
- **Arity:** 1 (monadic)
- **Category:** math
- **Signature:** `x → sin(x)` (radians)
- **Description:** Trigonometric sine, the foundation of periodic wave generation.

### ∿ · wave
- **Arity:** 1 (monadic)
- **Category:** math
- **Signature:** `x → oscillate(x)`
- **Description:** Periodic wave/mapping function used for harmonic phase generation in the Lab. In actual Uiua, `∿` is "sin" but the Lab treats it as a generic wave oscillator for synthesis vocabulary.

### × · mul
- **Arity:** 2 (dyadic)
- **Category:** math
- **Signature:** `a b → a·b`
- **Description:** Element-wise multiplication (scalar or array).

### + · add
- **Arity:** 2 (dyadic)
- **Category:** math
- **Signature:** `a b → a+b`
- **Description:** Element-wise addition.

### ÷ · div
- **Arity:** 2 (dyadic)
- **Category:** math
- **Signature:** `a b → a÷b`
- **Description:** Element-wise division.

### ⁿ · power
- **Arity:** 2 (dyadic)
- **Category:** math
- **Signature:** `a b → aᵇ`
- **Description:** Raises `a` to the exponent `b`, element-wise.

### √ · sqrt
- **Arity:** 1 (monadic)
- **Category:** math
- **Signature:** `x → √x`
- **Description:** Square root of each element.

### ∡ · angle
- **Arity:** 2 (dyadic)
- **Category:** math
- **Signature:** `y x → atan2(y,x)`
- **Description:** Complex phase angle (atan2) of a 2-D vector.

### τ · tau
- **Arity:** 0 (constant)
- **Category:** math
- **Signature:** `→ τ` where τ = 2π ≈ 6.28318530718
- **Description:** The circle constant τ (one full turn in radians).

### η · eta
- **Arity:** 0 (constant)
- **Category:** math
- **Signature:** `→ η`
- **Description:** Small step factor or decay constant; in the Lab it serves as an infinitesimal/epsilon placeholder.

---

## Modifiers (Higher-Order Combinators)

### ⍥ · repeat
- **Arity:** modifier (1 function + 1 count + 1 array)
- **Category:** modifier
- **Signature:** `f n A → fⁿ(A)` — applies `f`, `n` times
- **Description:** Iterates a function a fixed number of times. Central to the Cellular Dilation preset (`⍥(♭ ⊞+ ⌕ ▽) 4`).

### ∵ · each
*See entry under Array Combinators — listed twice in palette categories? No: `∵` in GLYPHS is "each"; `∺` is "stitch". Both documented in their respective categories.*

### ⌿ · reduce
- **Arity:** modifier (1 function + 1 array)
- **Category:** modifier
- **Signature:** `f A → s` (scalar or reduced-rank)
- **Description:** Left-fold (reduce) of a dyadic function across an array's leading axis. Example: `⌿+ ⇡5` sums 0..4 = 10.

---

## Search & Filter

### ⌕ · find
- **Arity:** 2 (dyadic)
- **Category:** search
- **Signature:** `pattern haystack → indices`
- **Description:** Locates positions of a pattern array within a larger array, returning index/occurrence information.

### ▽ · keep / filter
- **Arity:** 2 (dyadic)
- **Category:** filter
- **Signature:** `mask array → filtered`
- **Description:** Masks/filters elements of an array by a boolean or index set.

---

## Inspection

### △ · shape
- **Arity:** 1 (monadic)
- **Category:** inspect
- **Signature:** `A → dimensions`
- **Description:** Returns the shape (dimension vector) of an array, e.g., `△ ⊞+ ⇡3 ⇡4` yields `[3 4]`.

---

## Quick-Reference Table

| Glyph | Name | Arity | Category |
|:---:|:---|:---:|:---|
| ⊞ | table | 2 | array |
| ○ | sine | 1 | math |
| ∿ | wave | 1 | math |
| ⇡ | range | 1 | array |
| ⇌ | reverse | 1 | transform |
| ⍉ | transpose | 1 | transform |
| ♭ | flatten | 1 | array |
| △ | shape | 1 | inspect |
| ⍥ | repeat | modifier | modifier |
| ∵ | each | modifier | modifier |
| ⌕ | find | 2 | search |
| ▽ | keep | 2 | filter |
| τ | tau | 0 | math |
| × | mul | 2 | math |
| + | add | 2 | math |
| ÷ | div | 2 | math |
| ⁿ | power | 2 | math |
| ∺ | stitch | 2 | array |
| ⊂ | join | 2 | array |
| ◰ | classify | 1 | array |
| ⌿ | reduce | modifier | modifier |
| √ | sqrt | 1 | math |
| ∡ | angle | 2 | math |
| η | eta | 0 | math |

---

## Sources & Fidelity

Glyph primitives and names aim to match the canonical Uiua documentation at [uiua.org/docs](https://uiua.org/docs). Where the Lab makes pedagogical simplifications (e.g., `∿` as a generic wave rather than strict "sin"), this is noted in the entry. As the Lab's interpreter matures (v1.0), these descriptions will be tightened to reflect exact Uiua semantics.

Contributors adding new glyphs must verify their entries against the Uiua reference. Fictional or invented primitives will not be accepted — see [CONTRIBUTING.md](../CONTRIBUTING.md#adding-a-new-glyph).
