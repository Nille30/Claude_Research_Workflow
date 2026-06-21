---
paths:
  - "Slides/**/*.tex"
---

# Task Completion Verification Protocol

**At the end of EVERY task, Claude MUST verify the output works correctly.** This is non-negotiable.

## For LaTeX/Beamer Slides:
1. Compile with xelatex and check for errors
2. Open the PDF to verify figures render (`open` on macOS, `xdg-open` on Linux)
3. Check for overfull hbox warnings

## For TikZ Diagrams:
1. Compile each diagram to a standalone PDF and check for errors
2. Convert to SVG (vector format) for crisp rendering: `pdf2svg input.pdf output.svg`
3. Verify SVG files contain valid XML/SVG markup
4. **Freshness check:** Before reusing any TikZ SVG, verify extract_tikz.tex matches current Beamer source

## For R Scripts:
1. Run `Rscript scripts/R/filename.R`
2. Verify output files (PDF, RDS) were created with non-zero size
3. Spot-check estimates for reasonable magnitude

## Common Pitfalls:
- **Relative paths**: confirm paths resolve from the location where the artifact is compiled/run
- **Assuming success**: Always verify output files exist AND contain correct content
- **Stale TikZ SVGs**: extract_tikz.tex diverges from Beamer source → always diff-check

## Verification Checklist:
```
[ ] Output file created successfully
[ ] No compilation/render errors
[ ] Images/figures display correctly
[ ] Paths resolve where the artifact is built/run
[ ] Opened in viewer to confirm visual appearance
[ ] Reported results to user
```
