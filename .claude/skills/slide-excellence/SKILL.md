---
name: slide-excellence
description: Multi-agent comprehensive slide review (visual + proofreading, plus TikZ conditionally). Use when user says "full review", "excellence pass", "comprehensive check", "review everything", "pre-release review", "slide excellence", or before shipping a deck. Fanout wrapper — for a single lens, use `/visual-audit` or `/proofread` directly.
argument-hint: "[TEX filename] [--fast]"
allowed-tools: ["Read", "Grep", "Glob", "Write", "Bash", "Task"]
context: fork
---

# Slide Excellence Review

Run a comprehensive multi-dimensional review of lecture slides. Multiple agents analyze the file independently, then results are synthesized.

> **Which slide-review skill do I want?**
>
> - **`/slide-excellence`** (this skill) — multi-agent fanout (visual + proofread, plus TikZ conditionally). Best for **pre-release** checks.
> - **`/visual-audit`** — single lens, layout/overflow/font/spacing only. Fast.
> - **`/proofread`** — single lens, grammar/typos/overflow/terminology.

**Important:** this orchestrator does **conditional** dispatch — it only spawns the subagents that can actually produce useful output for the given file. No more running `tikz-reviewer` on a file with zero TikZ.

## Step 1: Identify the File

Parse `$ARGUMENTS` for the filename. Resolve path in `Slides/`.

The file type is Beamer (`.tex`).

## Step 2: Pre-flight — Detect Conditions

Before spawning any agent, probe the file to determine which reviews make sense:

```bash
FILE="$resolved_path"

# Has TikZ diagrams?
has_tikz=$(grep -c '\\begin{tikzpicture}' "$FILE" 2>/dev/null); has_tikz=${has_tikz:-0}
```

Report the detection:

```
File:         path/to/file.tex
Type:         Beamer (.tex)
TikZ blocks:  3
```

## Step 3: Run Review Agents in Parallel

Spawn only the agents whose conditions hold:

**Always-on for slides (`.tex`):**

- **Agent A: Visual Audit** (`slide-auditor`)
  Overflow, font consistency, box fatigue, spacing, images.
  Save: `quality_reports/[FILE]_visual_audit.md`.

- **Agent B: Proofreading** (`proofreader`)
  Grammar, typos, consistency, academic quality, citations.
  Save: `quality_reports/[FILE]_proofread_report.md`.

**Conditional:**

- **Agent C: TikZ Review** (`tikz-reviewer`) — only if `has_tikz > 0`.
  Measurement-based collision audit (Bézier, gaps, boundaries, margins).
  Save: `quality_reports/[FILE]_tikz_review.md`.

**De-duplication:** if the user has already run one of these skills on this file in the current session (e.g., ran `/proofread` first, now running `/slide-excellence`), ask whether to reuse the existing report or re-run. Default: reuse (saves tokens).

## Step 4: Synthesize Combined Summary (reduce typed findings)

This is **fan-out → reduce** ([`orchestrator-protocol.md`](../../rules/orchestrator-protocol.md)): each agent returns `FINDING`s + a `SCORECARD` in the shared schema ([`orchestration-schemas.md`](../../references/orchestration-schemas.md)), and this step **stacks the typed scorecards** rather than re-reading each report by eye. The Overall Quality Score is the gate predicate over summed CRITICAL/MAJOR/MINOR counts. (Conditional dispatch means a skipped lens contributes no findings, not zeros to average.)

Only include sections for agents that actually ran.

```markdown
# Slide Excellence Review: [Filename]

**File:** [path]
**Type:** Beamer (.tex)
**Detected:** TikZ=N
**Agents spawned:** [A, B, C] (skipped: C [no TikZ])

## Overall Quality Score: [EXCELLENT / GOOD / NEEDS WORK / POOR]

| Dimension | Critical | Medium | Low |
|-----------|----------|--------|-----|
| Visual/Layout | | | |
| Proofreading | | | |
| TikZ (if ran) | | | |

### Critical Issues (Immediate Action Required)
### Medium Issues (Next Revision)
### Recommended Next Steps
```

## Step 5: Report Token/Time Budget

After completion, print an estimate:

```
Spawned N agents; approx token usage ~XXk. Sequential fallback
(one agent at a time) would cost ~XXk but take ~5× longer. For
cost-conscious reviews, run individual subagent skills directly
(/proofread, /visual-audit).
```

## Flag Reference

| Flag | Effect |
|---|---|
| `--fast` | Spawn a single synthesis agent reading the file directly, rather than parallel subagents. Cheaper (~8k vs ~50k tokens) but less thorough. |

## Quality Score Rubric

| Score | Critical | Medium | Meaning |
|-------|----------|--------|---------|
| Excellent | 0-2 | 0-5 | Ready to present |
| Good | 3-5 | 6-15 | Minor refinements |
| Needs Work | 6-10 | 16-30 | Significant revision |
| Poor | 11+ | 31+ | Major restructuring |

## Why conditional dispatch matters

An earlier version of this orchestrator spawned every subagent regardless of file content. Running `tikz-reviewer` on a TikZ-free deck produced an empty report (wasted tokens).

Conditional dispatch cuts token cost by never running a reviewer that can't produce useful output.
