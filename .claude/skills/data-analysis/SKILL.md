---
name: data-analysis
description: End-to-end Python/Marimo data analysis pipeline — exploration → cleaning → regression → publication-ready tables and figures. Use when user says "analyze this dataset", "run a regression on X", "explore this CSV", "full analysis workflow", "get me summary stats and a regression", or points at a `.csv`/`.parquet`/`.dta` and asks for empirical results. Produces Marimo `.py` notebooks in `code/` and outputs to `results/`. Drops to R (`code/R/`) only when a method has no usable Python implementation.
argument-hint: "[dataset path or description of analysis goal]"
allowed-tools: ["Read", "Grep", "Glob", "Write", "Edit", "Bash", "Task", "Monitor"]
---

# Data Analysis Workflow

Run an end-to-end data analysis in **Python (Marimo notebooks, managed with uv)**: load, explore, analyze, and produce publication-ready output. Drop to **R** only when a method has no usable Python implementation (see [When to drop to R](#when-to-drop-to-r)).

**Input:** `$ARGUMENTS` — a dataset path (e.g., `data/clean/county_panel.csv`) or a description of the analysis goal (e.g., "regress wages on education with state fixed effects using CPS data").

---

## Constraints

- **Python-first.** pandas/polars for data, `pyfixest` / `statsmodels` / `linearmodels` for estimation, matplotlib (or plotnine) for figures. Run everything through `uv run` so it uses the locked environment.
- **Marimo notebooks are the source of truth** — reactive, no hidden state, committed as `.py`. Save to `code/` with descriptive names.
- **Seed once at the top** in YYYYMMDD format for any stochastic step: `RNG = np.random.default_rng(20260620)`.
- **Save all outputs** (figures, tables, serialized objects) to `results/`. Raw data in `data/raw/` is read-only; every transform lives in code, writing to `data/clean/`.
- **Tables and figures are written from code** into `results/`, then `\input{}` / `\includegraphics` into the paper and slides — never hand-copied.
- **No hardcoded absolute paths.** Everything relative to the repo root (`pathlib.Path`).
- **Review before done.** Run `scripts/quality_score.py` on the generated notebook (syntax, hardcoded paths, missing seed). For any R fallback script, run the `r-reviewer` agent.

---

## Workflow Phases

### Phase 0: Pre-Flight Report

**Before writing any analysis code, produce a Pre-Flight Report** showing you read the inputs. This prevents the common failure mode where the agent hallucinates variable names or skips project conventions.

Inspect the dataset non-destructively first, e.g.:

```bash
uv run python -c "import pandas as pd; df=pd.read_csv('data/clean/panel.csv'); print(df.shape); print(df.dtypes); print(df.head()); print(df.isna().mean())"
```

Output block (in your response to the user, before Phase 1):

```markdown
## Pre-Flight Report

**Dataset:** [path]
- Variables found: [list from df.columns]
- Rows: [count]
- Key types: [e.g., "outcome=float, treatment=binary, state=category"]
- Missing-data summary: [% missing per key var]

**Project conventions read:**
- `CLAUDE.md` — Python/Marimo, uv, seed-at-top, outputs to results/
- `.claude/rules/inference-robustness.md` — [multiple-testing / robustness items applicable]

**Task interpretation:** [one sentence restating what the user asked for]

**Plan:** [3-5 bullet outline of the notebook structure]
```

If any input cannot be read (missing file, unreadable format), stop and ask the user before proceeding.

### Phase 1: Setup and Data Loading

1. Create the Marimo notebook with a header cell (title, author, purpose, inputs, outputs).
2. Import packages in the first cell; seed once (`np.random.default_rng(YYYYMMDD)`).
3. Define `results/` output paths with `pathlib.Path`; create subdirs as needed.
4. Load and inspect the dataset (`data/clean/`), reading from `data/raw/` only to build the clean file.

### Phase 2: Exploratory Data Analysis

Generate diagnostic outputs:
- **Summary statistics:** `df.describe()`, missingness rates, dtypes
- **Distributions:** histograms for key continuous variables
- **Relationships:** scatter plots, correlation matrices
- **Time patterns:** if panel data, plot trends over time
- **Group comparisons:** if treatment/control, compare pre-treatment means

Save all diagnostic figures to `results/diagnostics/`.

### Phase 3: Main Analysis

Based on the research question:
- **Panel / fixed effects:** `pyfixest.feols` (formula interface with `| fe1 + fe2`), or `linearmodels.PanelOLS`.
- **Cross-section:** `statsmodels` OLS/GLM (`smf.ols`).
- **Standard errors:** cluster at the appropriate level and document why (`vcov={"CRV1": "firm_id"}` in pyfixest).
- **Multiple specifications:** start simple, progressively add controls.
- **Effect sizes:** report standardized effects alongside raw coefficients.
- For asset-pricing work: portfolio sorts, factor regressions, Fama-MacBeth — state NYSE-vs-all breakpoints and value-vs-equal weighting (see the asset-pricing methodology lens).
- Apply the multiple-testing / robustness standards in [`inference-robustness.md`](../../rules/inference-robustness.md).

### Phase 4: Publication-Ready Output

**Tables:**
- `pyfixest.etable(m1, m2, type="tex")` is the preferred regression-table exporter (panel FE models). For non-FE models, `statsmodels` `summary2().as_latex()` or the `stargazer` Python port.
- Include all standard elements: coefficients, SEs, significance indicators, N, R².
- Export as `.tex` into `results/tables/` for `\input{}`; optionally `.html`/`.md` for quick viewing.

**Figures:**
- matplotlib (or plotnine for a ggplot grammar).
- Save with `transparent=True` for Beamer compatibility and explicit `figsize`.
- Proper axis labels (sentence case, units).
- Save as both `.pdf` (vector, for LaTeX) and `.png` to `results/figures/`.

### Phase 5: Save and Review

1. Serialize key objects (cleaned data, fitted models, summary tables) to `results/` — `.parquet` for frames, `joblib`/`pickle` for fitted models.
2. Create `results/` subdirectories as needed (`Path(...).mkdir(parents=True, exist_ok=True)`).
3. Run the quality gate on the notebook:

   ```bash
   uv run python scripts/quality_score.py code/[notebook].py
   ```

   Address any auto-fail (syntax), hardcoded paths, or missing-seed findings.
4. For any **R fallback** script, delegate to the `r-reviewer` agent: "Review the script at code/R/[script].R" and address Critical/High issues.

---

## Notebook Structure

A Marimo notebook is a runnable `.py` file. Author cells with `uv run marimo edit code/[notebook].py`; run headless (reproducible, writes outputs) with `uv run python code/[notebook].py`. Follow this structure:

```python
import marimo

app = marimo.App(width="medium")


@app.cell
def _():
    # 0. Setup ----
    # [Descriptive Title] | Author | Purpose | Inputs | Outputs
    import marimo as mo
    import numpy as np
    import pandas as pd
    import pyfixest as pf
    import matplotlib.pyplot as plt
    from pathlib import Path

    RNG = np.random.default_rng(20260620)   # seed once, YYYYMMDD
    RESULTS = Path("results")
    (RESULTS / "tables").mkdir(parents=True, exist_ok=True)
    (RESULTS / "figures").mkdir(parents=True, exist_ok=True)
    return RNG, RESULTS, mo, np, pd, pf, plt


@app.cell
def _(pd):
    # 1. Data loading ----
    df = pd.read_csv("data/clean/panel.csv")
    return (df,)


@app.cell
def _(df):
    # 2. Exploratory analysis ---- (summary stats, diagnostic plots)
    df.describe()
    return


@app.cell
def _(df, pf):
    # 3. Main analysis ----
    m = pf.feols("y ~ treat + x | unit + year", data=df, vcov={"CRV1": "unit"})
    return (m,)


@app.cell
def _(RESULTS, m, pf):
    # 4. Tables and figures ----
    pf.etable(m, type="tex").save(RESULTS / "tables" / "main.tex")
    return


@app.cell
def _():
    # 5. Export ---- (serialize objects, savefig pdf+png)
    return


if __name__ == "__main__":
    app.run()
```

---

## When to drop to R

Stay in Python by default. Drop to R only when a method has **no usable Python implementation** — e.g. parts of the Callaway–Sant'Anna DiD stack (`did`, `DRDID`, `didFF`), `HonestDiD` sensitivity, or a niche estimator that exists only as an R package. When you do:

- Put the script in `code/R/` and follow [`r-code-conventions.md`](../../rules/r-code-conventions.md).
- Note in the notebook/script header *why* R was needed (which method, which package).
- Seed with `set.seed(YYYYMMDD)`; write outputs to the same `results/` tree so the paper `\input{}`s them identically.
- Run the `r-reviewer` agent on the script.
- For staggered DiD specifically, use the [`/did-event-study`](../did-event-study/SKILL.md) skill, which drives the canonical packages to the Sant'Anna practitioner standard.

---

## Important

- **Reproduce, don't guess.** If the user specifies a regression, run exactly that.
- **Show your work.** Print summary statistics before jumping to regression.
- **Check for issues.** Look for multicollinearity, outliers, perfect prediction, silent NA-dropping on merges.
- **Use relative paths.** All paths relative to the repository root.
- **No hardcoded values.** Use variables for sample restrictions, date ranges, etc.

## Long-running fits: use the Monitor tool

For regressions, simulations, or bootstrap loops that take more than a couple of minutes, launch via Bash with `run_in_background: true` and then use the **Monitor tool** to stream stdout into the conversation in real time. Pattern:

1. Background-launch: `uv run python code/03_analyze.py` with `run_in_background: true`. Capture the `bash_id`.
2. Use Monitor on the `bash_id` until a milestone fires (e.g., `Coefficients table written`, or process exit).
3. Continue or course-correct based on what the stream reveals.

This avoids the polling-loop anti-pattern (`sleep 30; check; sleep 30; check`) and avoids burning cache on idle waits.
