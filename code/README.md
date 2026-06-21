# code/

Marimo `.py` notebooks (the analysis source of truth) + R fallback.

- `*.py` — Marimo notebooks. Edit with `uv run marimo edit code/<nb>.py`; run headless
  (reproducible, writes outputs) with `uv run python code/<nb>.py`. Seed once at the top (YYYYMMDD).
- `R/` — R scripts, used **only** when a method has no usable Python implementation.
  Note in the header which method/package forced R.
- `_scratch/` — throwaway exploration. Not real analysis; never `\input{}` from here.

Outputs go to `../results/`. Raw data in `../data/raw/` is read-only.
