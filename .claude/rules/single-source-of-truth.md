---
paths:
  - "Figures/**/*"
  - "Slides/**/*.tex"
---

# Single Source of Truth: Enforcement Protocol

**The Beamer `.tex` file is the authoritative source for slide content; analysis code is the authoritative source for every reported number.** Generated artifacts are derived from these sources, never hand-edited independently.

## The SSOT Chain

```
Beamer .tex (SLIDE SOURCE OF TRUTH)
  ├── extract_tikz.tex → PDF → SVGs (derived)
  ├── Bibliography_base.bib (shared)
  └── Figures/LectureN/*.rds → charts (data source)

analysis code (RESULTS SOURCE OF TRUTH)
  └── tables / figures → \input{} into paper & slides (derived)

NEVER edit derived artifacts independently.
ALWAYS propagate changes from source → derived.
```

---

## TikZ Freshness Protocol (MANDATORY)

**Before reusing ANY extracted TikZ SVG, verify it matches the current Beamer source.**

### Diff-Check Procedure

1. Read the TikZ block from the Beamer `.tex` file
2. Read the corresponding block from `Figures/LectureN/extract_tikz.tex`
3. Compare EVERY coordinate, label, color, opacity, and anchor point
4. If ANY difference exists: update `extract_tikz.tex` from Beamer, recompile, regenerate SVGs

### When to Re-Extract

Re-extract ALL TikZ diagrams when:
- The Beamer `.tex` file has been modified since last extraction
- Any TikZ-related quality issue is reported
- Before any commit that includes TikZ changes

---

## Results Fidelity (MANDATORY)

**Every reported number traces to the exact cell/script that produced it.**

1. Tables and figures are written *from code* into the results directory
2. The paper and slides pull them in via `\input{}` / `\includegraphics`
3. Numbers are never hand-copied into prose — if a number appears in the text, it must match the generated artifact

---

## Content Fidelity Checklist

```
[ ] Math check: every equation appears with the notation the code/spec defines
[ ] Citation check: every \cite has a @key in the bibliography
[ ] Figure check: every \includegraphics resolves to a generated artifact
[ ] No hand-copied numbers: every reported value traces to a script/cell
```
