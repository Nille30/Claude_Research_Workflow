# Memory Model: two tiers (committed vs local)

Persisted learnings live in two places. Knowing which tier a fact belongs to keeps the committed memory useful and portable across machines.

## MEMORY.md (repo root, committed)

**Purpose:** cross-session learnings worth keeping with the project.

- Format: `[LEARN:category] wrong → right` for corrections; structured `**Why:**` + `**How to apply:**` for feedback/project entries.
- Loaded into context each session — **keep it concise** (aim < 200 lines).
- Reviewed after significant sessions (plan approval, feature work, a correction).

What belongs here: workflow patterns, design decisions, reproducibility lessons, recurring mistakes and their fixes — anything that helps the *next* session on this project.

## .claude/state/personal-memory.md (gitignored, local only)

**Purpose:** machine-specific and personal facts that should NOT travel with the repo.

- Machine setup (e.g. `XeLaTeX needs TEXINPUTS=../Preambles on this Mac`).
- Tool-version quirks; local data paths; personal preferences.
- No size limit; never loaded automatically.

## Cross-machine

Generic learnings sync via git (MEMORY.md); personal facts stay local (each machine builds its own `personal-memory.md`). Restricted-data *paths* live in personal-memory, never in committed files (see [`confidential-data.md`](confidential-data.md)).

## Promotion

[`/promote-memory`](../skills/promote-memory/SKILL.md) graduates a generic `[LEARN]` entry from the gitignored `personal-memory.md` to the committed `MEMORY.md` via a five-critic council vote — the gate that keeps machine-specific noise out of the shared memory.

## Quick test

> "Would a future me on a *different machine* benefit from this?" → MEMORY.md. > "Is this only true on *this* setup?" → personal-memory.md.
