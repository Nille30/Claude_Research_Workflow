# CLAUDE.md — Empirical Finance & Accounting Research (Python/Marimo)

**Type:** Personal reusable research template — branch/copy it per new paper, then fill the placeholders below.
**Role:** Empirical financial economist / data scientist.
**Two research tracks:** (1) causal inference, (2) empirical asset pricing (often deliberately non-causal). The track determines which review lens applies.

**Current project:** `[PROJECT NAME]` — `[one-line description]`
**Research question:** `[fill in]`
**Track:** `[causal | asset-pricing]`

---

## Core principles

1. **Plan before build.** Non-trivial tasks start in plan mode; save the plan to `quality_reports/plans/`. Agree the approach before any file is touched.
2. **Verify before done.** Every reported number traces back to the exact cell/script that produced it. Nothing is "done" until it re-runs clean (the `verifier` agent enforces this).
3. **Reproducible by construction.** Marimo `.py` notebooks are the source of truth (reactive, no hidden state). Seed once at the top (YYYYMMDD) for any stochastic step. Raw data is read-only; every transform lives in code.
4. **Replication-first.** When building on an existing paper or package, reproduce its reported numbers within tolerance *before* extending.
5. **Right method, right rigor.** Causal claims get an identification audit; asset-pricing results get a sorts/factor/Fama-MacBeth audit. `/review-paper --peer` picks the lens via the methods/domain referees.
6. **Pre-specify, then run.** Name the main specification before estimating; document any later deviation.
7. **Division of labour.** Claude owns LaTeX formatting and table/figure typesetting. The author owns content, claims, and interpretation.

---

## Stack

- **Analysis:** Python in **Marimo** notebooks (`.py`, committed as source), managed with **uv**. Drop to **R** only when a method has no usable Python implementation; note why in the notebook/script header.
- **No Stata. No Quarto.**
- **Writing:** LaTeX (papers) + Beamer (research-talk slides).
- **Outputs:** tables and figures are written *from code* into `results/`, then `\input{}` / `\includegraphics` into the paper and slides — never hand-copied.

---

## Environment

- **Python:** managed with **uv**. Dependencies live in `pyproject.toml`, pinned by `uv.lock`. Run everything through `uv run`. Add a package with `uv add <pkg>`.
- **R (fallback only):** pinned with **renv** when used. Note in the header why R was needed.
- Never hardcode absolute paths; everything is relative to the repo root.

---

## Commands

```bash
# Marimo (Python analysis)
uv run marimo edit code/<notebook>.py     # interactive editing
uv run python code/<notebook>.py          # headless, reproducible run (writes outputs)
uv run marimo run code/<notebook>.py      # serve as a read-only app

# R fallback
Rscript code/R/<script>.R

# LaTeX — latexmk handles passes + bib automatically
latexmk -pdf -xelatex paper/main.tex                 # compile the paper
latexmk -pdf -xelatex Slides/<deck>.tex              # compile a talk deck
latexmk -c paper/main.tex                            # clean aux files

# Quality gate (>=80 to commit) — scores .py / .R / .tex
python scripts/quality_score.py code/<notebook>.py
./scripts/validate-setup.sh                          # check the toolchain
```

---

## Methods reference (drives the review lens)

- **Causal track (by frequency):** DiD / event study → RDD → panel fixed effects → DML / causal ML → synthetic control → IV / 2SLS.
- **Asset-pricing track (by frequency):** univariate/multivariate portfolio sorts → multi-factor regressions (Fama-French & extensions) → Fama-MacBeth → machine learning.

---

## Directory map (flat layout)

```
literature/       reference PDFs (shared bib is Bibliography_base.bib at repo root)
data/
  raw/            READ-ONLY: WRDS pulls + timestamped scrape snapshots (gitignored)
  clean/          processed data built by code (gitignored if heavy)
code/             Marimo .py notebooks — data prep/merging + analysis
  R/              R fallback scripts (method has no Python implementation)
  _scratch/       throwaway exploration (not real analysis)
results/          generated tables + figures (tables/, figures/, diagnostics/)
paper/            LaTeX manuscript, built on the shipped template (main.tex)
Slides/           Beamer talk decks (Preambles/ holds the shared LaTeX header)
submissions/      per-journal bundles assembled from the folders above
.claude/          agents, skills, rules, references (this config)
quality_reports/  plans, specs, session logs
```

*(`Slides/`, `Figures/`, `Preambles/` keep their capitalization — they are wired into the slide/TikZ skills.)*

---

## Data rules — see `.claude/rules/confidential-data.md`

- **WRDS credentials never** committed, uploaded, printed, or placed in Claude's context. Local environment only.
- **Scraped data:** save a timestamped raw snapshot (HTML/JSON) + source URL *before* parsing. Parsing reproduces from the snapshot; never silently re-scrape.
- **Raw is read-only.** All cleaning/merging happens in code under `code/`, writing to `data/clean/`.

---

## Bibliography & manuscript

- The shared bibliography is `Bibliography_base.bib` (repo root); reference PDFs live in `literature/`. The manuscript and slides cite against it.
- The paper is always built on the LaTeX template in `paper/` (`main.tex` + preamble). Don't restructure the template; fill it.
- When Claude cites a source, it adds the entry to `Bibliography_base.bib` if missing and reuses the existing cite-key convention.
- Tables and figures are pulled in via `\input{}` / `\includegraphics` from `results/` — Claude wires these up rather than pasting numbers.

---

## Skills — the commands you might type

Most-used, by workflow (full index in [README.md](README.md)):

- **Data / reproducibility:** `/data-analysis` `/did-event-study` `/simulation-study` `/diagnose` `/power-analysis` `/audit-reproducibility` `/replication-package` `/capture-environment` `/review-r` `/disclosure-check`
- **Papers / review:** `/review-paper` (`--peer [journal]`) `/seven-pass-review` `/respond-to-referees` `/verify-claims` `/proofread` `/humanize` `/submission-disclosures` `/validate-bib`
- **Slides (talks):** `/compile-latex` `/slide-excellence` `/visual-audit` `/extract-tikz` `/new-diagram`
- **Research / writing:** `/interview-me` `/lit-review` `/research-ideation` `/preregister` `/grant-proposal` `/data-management-plan`
- **Meta / workflow:** `/commit` `/learn` `/new-skill` `/checkpoint` `/context-status` `/deep-audit` `/coauthor-brief`

Most of the time you just describe what you want; Claude picks the skill.

---

## Agents — invoked by skills, not by you

- **methods-referee / domain-referee / editor** — the `/review-paper --peer` peer-review panel, calibrated to a target journal (finance / top-5 econ / accounting profiles in `.claude/references/journal-profiles.md`).
- **claim-verifier** — fresh-context Chain-of-Verification guard against hallucinated citations / numbers.
- **verifier** — re-runs scripts, confirms outputs render and reported numbers match, before anything is called done.
- **sim-reviewer / r-reviewer / r-package-reviewer** — Monte Carlo + R code review (the R fallback path).
- **slide-auditor / proofreader / tikz-reviewer / humanize-auditor** — slide and prose review.

---

## Quality gates (advisory)

| Score | Checkpoint | Meaning |
|-------|------------|---------|
| 80 | Commit | Good enough to save |
| 90 | PR | Ready to share |
| 95 | Excellence | Aspirational |

Enforced by `/commit` and — once you run `./scripts/install-hooks.sh` — by the `.githooks/pre-commit` hook, which scores staged `.py` / `.R` / `.tex` files (≥80). Bypass sparingly with `SKIP_QUALITY_GATE=1` or `--no-verify`.

---

## Current state — update as you go

- [ ] `[first task]`

## Corrections (MEMORY)

Persisted lessons live in `MEMORY.md` as `[LEARN:category] wrong → right`. Check it before repeating past work.
