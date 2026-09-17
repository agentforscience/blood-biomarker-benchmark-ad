# Code

## User-specified repositories: none

The research topic specification contained **no `code_references` section, no specified papers or
datasets, and no "LOCAL RESOURCES - ALREADY STAGED" section**. There were therefore no mandated
repositories to clone and no staged local resources to verify. This is recorded explicitly so a
later phase does not assume something was missed.

## No external repositories cloned — and why

A GitHub search across five query formulations
(`ADNI plasma biomarker csv`, `plasma ptau217 amyloid dataset`,
`alzheimer blood biomarker panel machine learning`, `amyloid positivity prediction plasma`,
`SomaScan alzheimer plasma`) returned **essentially nothing usable**: four queries returned zero
repositories and the fifth returned a single 7-star repo
(`robertfhillary/gwas_ewas_AD_plasma_biomarkers`) which runs BayesR+ GWAS/EWAS on plasma
biomarkers — a different question (genetic/epigenetic architecture), not panel-size benchmarking.
It was not cloned because it would not be used.

This is the expected outcome: the planned analyses (feature selection, cross-validated ROC/AUC,
DeLong tests, bootstrap CIs, calibrated simulation) are **standard scikit-learn / scipy /
statsmodels work**. Cloning a repository would add dependency risk without adding capability.

## Analysis tooling written in this phase (`tools/`)

| Script | Purpose |
|---|---|
| `tools/epmc_search.py` | Europe PMC systematic search: cursor-paged, multi-query, dedupes by DOI/PMID, records which queries matched each record. |
| `tools/queries.json` | The 15 structured search queries used for the review. |
| `tools/rank.py` | Relevance scoring: weighted keyword patterns (core analytes, panel-size language, preclinical population, named cohorts), recency bonus, log-citation term, cross-query agreement, OA bonus; negative weights for animal/in-vitro/other-disease work. |
| `tools/dl_pdfs.py` | Open-access PDF fetcher (Europe PMC render + PMC fallback), validates `%PDF` magic bytes and size before saving. |
| `tools/build_gse275392.py` | Assembles GSE275392 into a 53×1305 matrix + phenotype table. Parses the GEO SOFT family file for labels and **handles the one UTF-16-encoded sample file** among 52 UTF-8 files. |
| `tools/validate_dataset.py` | Sanity checks (shape, NaN/inf, index alignment) plus a feasibility panel-size probe with feature selection **inside** the CV folds, run on both the full cohort and the deconfounded APOE33 stratum. |

Re-running the pipeline end to end:
```bash
source .venv/bin/activate
python tools/epmc_search.py tools/queries.json papers/epmc_raw.json   # literature
python tools/rank.py
python tools/dl_pdfs.py 45
bash datasets/GSE275392/download.sh                                   # data
python -W ignore tools/validate_dataset.py
```

## Environment

Isolated `uv` venv at `.venv/` (Python 3.12.8), dependencies tracked in `pyproject.toml`:
`pypdf`, `requests`, `httpx`, `pandas`, `numpy`, `scikit-learn`, `scipy`, `openpyxl`.
Note `scikit-learn` is 1.8+, where `LogisticRegression(penalty=...)` is deprecated in favour of
`l1_ratio` / `C` — use `C=` and, for elastic net, `l1_ratio=`.

## Likely additional dependencies for the experiment phase
`matplotlib` / `seaborn` (AUC(k) curves), and `statsmodels` if DeLong or LRT tests are wanted
(DeLong is ~30 lines of numpy and is often simpler to implement directly).
