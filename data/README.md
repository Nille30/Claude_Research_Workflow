# data/

- `raw/` — **READ-ONLY.** WRDS pulls and timestamped scrape snapshots. Never edited by hand,
  never committed if heavy/confidential (see `.claude/rules/confidential-data.md`). WRDS
  credentials never enter the repo or Claude's context.
- `clean/` — processed data built by code under `../code/`. Reproducible from `raw/` + code.

Every transform lives in code. If a file in `raw/` changes, that's a red flag.
