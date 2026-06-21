# Discipline Cards

Short reference cards naming each discipline's dominant paper-type frequencies, top journals, preregistration norms, and method conventions. Read by `/research-ideation`, `/interview-me`, `/preregister`, and the `editor` agent (in `/review-paper --peer`) when the user gives a `paper_type` or domain hint without specifying a target journal.

**Scope.** Ships three cards — **economics**, **finance**, and **accounting** — the fields this template targets. To add another discipline, copy a card section, fill the four fields (paper-type frequencies, journals, preregistration norms, method conventions), and reference the new short-name from `journal-profiles.md` and `methods-referee.md`.

**Maintenance.** When you add a journal profile to `journal-profiles.md`, cross-reference it here. When you add a paper type to `methods-referee.md`, cross-reference it here.

---

## Economics (`econ`)

**Paper-type frequencies (rough share of empirical work in top-5 journals).**

| Type | Share | Notes |
|---|---|---|
| Reduced-form | ~55% | DiD, IV, RD, event study, synthetic control. The dominant mode. |
| Structural | ~20% | DSGE, GE, IO empirical. Concentrated in macro / IO / labour. |
| Theory + empirics | ~15% | Theory-paper-with-empirical-test or empirical-paper-with-theory-section. |
| Descriptive | ~5% | Measurement / data-construction. Often the AEA P&P route. |
| Formal-theory | ~5% | Pure theory (micro, IO, contracts). More common in ECMA / TE / JET. |

**Dominant journals (shipped in `journal-profiles.md`).** AER, QJE, JPE, ECMA, ReStud. AEA P&P (proceedings) for descriptive / measurement work.

**Preregistration norms.**
- **Field experiments / RCTs:** mandatory in the **AEA RCT Registry** since 2018 for AEA-journal submission. Use `/preregister --style aea-rct`.
- **Lab experiments:** OSF / AsPredicted increasingly common; not yet uniformly required.
- **Observational / archival:** preregistration uncommon; pre-analysis plans appearing in some applied-micro corners.
- **Replication packages:** AEA Data and Code Availability Policy enforced; replication archive at JEL data archive.

**Method conventions.**
- Significance stars: AEA journals do **NOT** use stars in tables (since 2018 AEA Code style guide). Other journals (e.g., ReStud, JPubE) still allow them.
- Standard-error reporting: clustered SEs at treatment-assignment level expected; Conley / spatial SEs required for spatial data.
- Code: R, Stata, Python, Julia all accepted; replication packages must be self-contained and deterministic (`set.seed`).

**Cross-references.** `methods-referee.md` paper types: reduced-form, structural, theory+empirics, descriptive, formal-theory. `journal-profiles.md`: AER, QJE, JPE, ECMA, ReStud.

---

## Finance (`finance`)

**Paper-type frequencies (rough share of empirical work in top-3 journals).**

| Type | Share | Notes |
|---|---|---|
| Reduced-form | ~50% | Corporate-finance causal (DiD, IV, RD) + empirical asset-pricing tests. The dominant mode. |
| Structural | ~20% | Dynamic asset pricing, structural corporate, IO-finance. |
| Theory + empirics | ~15% | Asset-pricing/corporate theory with an empirical test. |
| Descriptive | ~10% | New data / measurement (microstructure, fintech, text). |
| Formal-theory | ~5% | Pure theory (continuous-time AP, contract theory). |

**Dominant journals (shipped in `journal-profiles.md`).** JF, JFE, RFS. Field outlets: JFQA, RCFS, Management Science (finance), JFI.

**Preregistration norms.**
- **Observational / archival (the bulk):** preregistration uncommon.
- **Field / household / behavioral experiments:** AEA RCT Registry or OSF where applicable.
- **Replication packages:** JF (AFA) and RFS (SFS) require code/data at acceptance; JFE enforces a code/data policy. WRDS is the data backbone — document pulls, don't deposit licensed raw data.

**Method conventions.**
- Significance: stars common, but factor-zoo scrutiny means **report t-stats** (Newey-West / GMM for asset pricing); a *new factor* needs t > 3 (Harvey–Liu–Zhu).
- Standard errors: cluster by firm and/or time (Petersen 2009); for asset-pricing tests, Newey-West / Fama-MacBeth.
- Asset-pricing hygiene: NYSE vs. all-stock breakpoints, value- vs. equal-weighting, microcap exclusion, look-ahead/survivorship checks.
- Code: Python and R dominant (Python is this template's default); WRDS/CRSP/Compustat the data backbone.

**Cross-references.** `methods-referee.md` paper types: reduced-form, structural, theory+empirics, descriptive, formal-theory. `journal-profiles.md`: JF, JFE, RFS.

---

## Accounting (`accounting`)

**Paper-type frequencies (rough share of empirical work in top-5 journals).**

| Type | Share | Notes |
|---|---|---|
| Reduced-form | ~55% | Archival causal/association: disclosure, earnings, governance, audit, tax. |
| Descriptive | ~15% | Measurement (earnings quality, readability, new constructs). |
| Theory + empirics | ~15% | Analytical model + archival test (esp. RAST, JAE). |
| Experimental | ~10% | Behavioral / JDM accounting (TAR, CAR) — use the survey-experiment lens. |
| Formal-theory | ~5% | Analytical accounting (disclosure / contracting theory). |

**Dominant journals (shipped in `journal-profiles.md`).** JAR, JAE, TAR, CAR, RAST.

**Preregistration norms.**
- **Archival (the bulk):** preregistration uncommon.
- **Experimental accounting:** OSF / AsPredicted increasingly expected; Registered Reports appearing (JAR Registered Reports).
- **Replication packages:** data/code increasingly requested, less uniform than econ/finance; WRDS/Compustat backbone.

**Method conventions.**
- Significance: stars conventional. Construct-validity scrutiny is high (proxies, not observables).
- Standard errors: two-way clustered by firm and year (Petersen 2009; Gow–Ormazabal–Taylor 2010).
- Identification: "association dressed as causation" is a desk-level concern (Roberts–Whited); expect an explicit identification strategy.
- Code: SAS and Stata historically common; **Python/R increasingly standard (and the convention in this template)**; WRDS/Compustat backbone.

**Cross-references.** `methods-referee.md` paper types: reduced-form, descriptive, theory+empirics, survey-experiment (for experiments), formal-theory. `journal-profiles.md`: JAR, JAE, TAR, CAR, RAST.

---

## How skills consume these cards

- **`/research-ideation`** — when the user names a topic without a discipline, the skill may infer one from context (citation style, vocabulary). The card supplies the default `paper_type` distribution to bias hypothesis generation.
- **`/interview-me`** — Phase 1 paper-type question uses the card's frequency table to order the option list (most-likely-first per discipline).
- **`/preregister`** — `--style` defaults to the card's preregistration-norms suggestion (e.g., `aea-rct` for econ/finance field experiments, `osf` for experimental accounting).
- **`editor`** (`/review-paper --peer`) — when the user gives `--peer` without naming a specific journal but with a discipline hint, the editor uses the card's "Dominant journals" list as the candidate set and asks for clarification.

---

## Adding a new discipline card

Copy this template:

```markdown
## [Discipline name] (`short-slug`)

**Paper-type frequencies.**
| Type | Share | Notes |
|---|---|---|
| ... |

**Dominant journals (shipped in `journal-profiles.md`).** [list]. [Optional: subfield outlets.]

**Preregistration norms.**
- [registry conventions per study type]

**Method conventions.**
- [significance stars / SE conventions / replication norms / dominant code language]

**Cross-references.** `methods-referee.md` paper types: [list]. `journal-profiles.md`: [list].
```

Then:

1. Add the card section above (alphabetically by short-slug).
2. Add concrete journal profiles to `journal-profiles.md` for at least the top-3 journals.
3. Add paper types to `methods-referee.md` if your field uses categories not already there (e.g., qualitative-case-study for sociology, mixed-methods for public health).
4. Cross-reference the new short-slug from `/research-ideation` and `/interview-me` if those skills should respect the new defaults.

---

## Where this file lives

- **File:** `.claude/references/discipline-cards.md`
- **Schema parallel:** `.claude/references/journal-profiles.md` (per-journal) and `.claude/references/audit-pet-peeves.md` (living-catalogue format).
- **Consumed by:** `/research-ideation`, `/interview-me`, `/preregister`, `editor` agent.
