# Empirical Finance & Accounting Research — Claude Code Setup

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

My personal, verification-gated Claude Code setup for empirical economics / finance / accounting research: **Python/Marimo analysis, LaTeX papers, and Beamer talk decks**. Branch or copy it per paper, fill the placeholders in [CLAUDE.md](CLAUDE.md), and start.

You describe what you want — a data analysis, a manuscript review, a replication package, a talk deck — and Claude plans the approach, runs specialized review agents, fixes issues, verifies quality, and presents results, with quality gates deciding when it's good enough.

---

## Quick Start

```bash
# Branch (or copy) this template for a new project
git switch -c paper/<short-name>

# Check the toolchain (reports what's missing, with install links)
./scripts/validate-setup.sh

# (once per clone) install the pre-commit quality gate
./scripts/install-hooks.sh

# Start Claude Code
claude
```

Then fill in the `[PLACEHOLDERS]` at the top of [CLAUDE.md](CLAUDE.md) (project name, research question, track), drop your data in `data/raw/`, and describe your first task. Most of the time you just say what you want and Claude picks the right skill; for non-trivial work it enters plan mode first.

> **Permission fatigue:** at default settings Claude asks before each tool call. After a few approvals, toggle **Auto-accept edits** or run `claude --permission-mode acceptEdits`. `.claude/settings.json` already pre-approves common Bash/Edit patterns.

---

## How it works

- **Goal-first, gate-enforced.** You state a goal; the work loops toward it under gates. Specialist agents do the labor; the quality gate (≥80) and the `verifier` agent decide when it's done.
- **Real gates, not reminders.** `./scripts/install-hooks.sh` installs a pre-commit hook that scores staged `.py` / `.R` / `.tex` files (≥80) on *every* commit. A `git-guardrails` hook blocks destructive git (`reset --hard`, `clean -f`, `push --force`).
- **An orchestration runtime.** Reviews fan out to forked specialist agents, reduce over a shared finding schema, judge with a hallucination gate, and loop until dry — see [`orchestrator-protocol.md`](.claude/rules/orchestrator-protocol.md).
- **Reproducible by construction.** Marimo `.py` notebooks are the source of truth; numbers in the paper trace back to the cell that produced them; raw data is read-only.
- **Not a daemon.** The loop is always you- or skill-initiated. You stay the auditor.

---

## Directory layout

```
literature/   reference PDFs (shared bib: Bibliography_base.bib at root)
data/raw      READ-ONLY source data (gitignored)         data/clean  built by code
code/         Marimo .py notebooks (+ R/ fallback, _scratch/)
results/      generated tables/ figures/ diagnostics/  (\input into paper & slides)
paper/        LaTeX manuscript        Slides/  Beamer talk decks (Preambles/ = header)
submissions/  per-journal bundles
.claude/      skills, agents, rules, references          quality_reports/  plans + logs
```

---

## What's included

Specialized **skills** (`/...` commands), **review agents**, and **rules** under [`.claude/`](.claude/). The most-used skills are grouped by workflow in [CLAUDE.md](CLAUDE.md); the full set is in [`.claude/skills/`](.claude/skills/). Highlights:

- **Data / reproducibility** — `/data-analysis` (Python/Marimo, R fallback), `/did-event-study`, `/simulation-study`, `/diagnose`, `/power-analysis`, `/audit-reproducibility`, `/replication-package`, `/capture-environment`.
- **Papers / review** — `/review-paper` (`--peer [journal]` simulated editor + referees, calibrated to finance / top-5 econ / accounting), `/seven-pass-review`, `/respond-to-referees`, `/verify-claims`, `/humanize`, `/submission-disclosures`.
- **Slides (talks)** — `/compile-latex`, `/slide-excellence`, `/visual-audit`, `/extract-tikz`, `/new-diagram`.
- **Research / writing** — `/interview-me`, `/lit-review`, `/research-ideation`, `/preregister`, `/grant-proposal`.

Journal calibration lives in [`.claude/references/journal-profiles.md`](.claude/references/journal-profiles.md); field norms in [`.claude/references/discipline-cards.md`](.claude/references/discipline-cards.md).

---

## Prerequisites

| Tool | Required for | Install |
|------|-------------|---------|
| [Claude Code](https://code.claude.com/docs/en/overview) | Everything | [claude.ai/install](https://claude.ai/install) |
| git | Version control | [git-scm.com](https://git-scm.com/downloads) |
| [uv](https://docs.astral.sh/uv/) + Python 3 | Analysis (Marimo notebooks) + internal scripts | [astral.sh/uv](https://docs.astral.sh/uv/) |
| XeLaTeX | LaTeX papers + Beamer slides | [TeX Live](https://tug.org/texlive/) / [MacTeX](https://tug.org/mactex/) |
| R *(fallback only)* | Methods with no Python implementation | [r-project.org](https://www.r-project.org/) |
| pdf2svg *(optional)* | TikZ → SVG (`/extract-tikz`) | `brew install pdf2svg` |
| [gh CLI](https://cli.github.com/) *(optional)* | PR workflow | `brew install gh` |

`./scripts/validate-setup.sh` reports which are present and what each unlocks.

---

## Quality gates

| Score | Checkpoint | Meaning |
|-------|------------|---------|
| 80 | Commit | Good enough to save |
| 90 | PR | Ready to share |
| 95 | Excellence | Aspirational |

Scored by `scripts/quality_score.py` (`.py` / `.R` / `.tex`) and enforced by `/commit` + the pre-commit hook. Bypass sparingly with `SKIP_QUALITY_GATE=1` or `--no-verify`.

---

## Credits

This setup is adapted from Pedro Sant'Anna's [Claude Code academic workflow](https://psantanna.com/claude-code-my-workflow/) and incorporates, with permission:

- The `/review-paper --peer` pipeline shape, disposition taxonomy, and journal-calibration schema from **[clo-author](https://github.com/hugosantanna/clo-author)** (Hugo Sant'Anna, UAB).
- The TikZ measurement / collision-avoidance rules from **[MixtapeTools](https://github.com/scunning1975/MixtapeTools)** (Scott Cunningham, Baylor).

---

## License

MIT License. See [LICENSE](LICENSE).
