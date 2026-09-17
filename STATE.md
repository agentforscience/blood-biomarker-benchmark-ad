# Research State

- Current phase: `None`
- Pipeline completed: `False`

## Previous phases

resource_finder (succeeded), experiment_runner (failed)

## Current phase context

- Phase: `experiment_runner`
- Status: `failed`
- Started: `2026-09-04T18:35:36.849470Z`
- Next steps:
  - Validate the report and experimental artifacts before finalizing.

## Workspace check

- Expected: `/workspaces/benchmarking_blood_based_prote_20260904_071340_3cf214d6`
- Actual: `/app`
- Directory usable: `True`
- Current process matches workspace: `False`

## Output validation

- Valid: `False`
- Expected: `REPORT.md`
- Missing: `REPORT.md`
- Outside workspace: None

## Agent notes

<!-- NEURICO_AGENT_NOTES_START -->
### resource_finder
<!-- NEURICO_AGENT_NOTES_START:resource_finder -->
**Phase 1 (resource_finder): COMPLETE** — 2026-09-04

## Completed
- Isolated `uv` venv at `.venv/` (Python 3.12.8), deps in `pyproject.toml`.
  NOTE: removed the `[build-system]`/hatchling block — it fails without a package dir. `uv add` works.
- Paper-finder service was **DOWN** (localhost:8000). Built a Europe PMC pipeline instead
  (`tools/epmc_search.py`, 15 queries) → **548 unique records screened**, **45 OA PDFs** downloaded,
  4 deep-read via chunker.
- Downloaded + assembled the one usable public dataset (**GSE275392**) and 10 reference tables.
- Artifacts on disk: `planning.md`, `literature_review.md`, `resources.md`,
  `papers/README.md`, `datasets/README.md`, `code/README.md`, `tools/` (6 scripts),
  `papers/epmc_ranked.json` (all 548 records w/ abstracts).

## Key findings (evidence paths in literature_review.md §2)
1. **BINDING CONSTRAINT:** no public individual-level dataset has core analytes + broad panel +
   amyloid status in a preclinical cohort. ADNI/A4/BioFINDER/Knight/Bio-Hermes/SAMS/UKB all need a
   signed DUA. Verified exhaustively across GEO/Dryad/Zenodo/figshare/OSF/HF/GitHub/Kaggle
   (negative log in `resources.md` §Challenges). Do not re-run this search.
2. **`datasets/GSE275392/`** — SomaScan 1,305 proteins x 53 non-demented elderly + `amyloid_status`.
   The only public option. TWO TRAPS, both handled:
   - **APOE is perfectly confounded with amyloid** (all 18 amyloid- are APOE33; all 17 APOE44 are
     amyloid+). **PRIMARY ANALYSIS = APOE33-only stratum, n=36, 18/18 balanced.** Full cohort is
     secondary + must be labelled confounded.
   - SomaScan **has no p-tau217/p-tau181/NEFL** (no phospho-epitopes). GFAP/MAPT/APOE/APP present.
   - Also: 39 duplicate protein column names; 1 of 53 files is UTF-16; log2-transform the RFUs.
3. **Literature strongly supports the hypothesis, by a stronger route than proposed:**
   - Trelle 2026 (n=315 CU, `papers/2026_plasma_proteomic_signatures_*.pdf`): **pTau217/Aβ42
     (2 analytes) AUC 0.940**; BD-pTau217 alone 0.920. **GFAP (0.646) and NfL (0.582) do NOT beat
     the age+sex+APOE covariate model (0.773); NfL is significantly WORSE.** Only ~5 of 123-130
     panel proteins associate with amyloid status.
   - Bio-Hermes: **295 proteins → AUC 0.79-0.81**, BELOW single p-tau217 (~0.90).
   - Knight ADRC (n=3,232, 120-plex): only **8/120** proteins associate with amyloid PET.
   - A4/LEARN (n=1,209 CU): adding Aβ42/40+GFAP+NfL to a p-tau217 model gains only **1-2% AUC**.
4. **Two sub-claims of the hypothesis will likely FAIL and must be reported honestly:**
   **NfL** is useless/harmful for amyloid in CU, and **MTBR-tau243** is a tau-tangle *staging*
   marker, not an early amyloid marker. The "3-5 analyte" core is likely **over-specified**.
5. **MANDATORY BASELINE the literature usually omits: age + sex + APOE-ε4 (AUC ~0.75-0.77).**
   Any panel not clearly beating this has demonstrated nothing.
6. **Endpoint discipline:** amyloid positivity != cognitive decline. GFAP/NfL look useful for the
   latter, near-useless for the former. Conflating them yields a wrong conclusion.
7. Feasibility probe (`tools/validate_dataset.py`, selection inside CV folds) already reproduces
   the predicted shape on the APOE33 stratum: AUC rises to **0.750 (k=10) / 0.762 (k=20)** then
   DECLINES to 0.658 at k=1305 — and the whole curve sits far below the p-tau217 benchmark (~0.90).

## Direction budget (top 3 kept; full rationale + 8 pruned directions in `planning.md`)
- **D1** Empirical AUC(k) saturation curve on GSE275392 (real data; weak power, n=36 primary).
- **D2** Literature-calibrated simulation of core-4 vs large panels (covers the named analytes,
  which no public data provides; anchors = ADNI p-tau217 AUC 0.904 / Aβ42/40 0.831 in
  `datasets/literature_reference/.../S7_ADNI_validation_cohort.xlsx`, correlations in S2).
- **D3** Quantitative meta-analytic synthesis of published (panel size → AUC) pairs from the corpus.
- Pruned: ADNI/UKB applications (infeasible), CSF data (wrong fluid), blood transcriptomics
  (wrong analyte), diagnosis-as-endpoint (wrong endpoint), deep learning (overfits at n=36),
  Dryad "Data from:" records (inspected — supplementary PDFs only, no individual data).

## Next phase: experiment_runner — concrete next steps
1. Read `planning.md` §5 first: pre-registered commitments (APOE33 primary; selection inside folds;
   report the AUC ratio AUC(k)/AUC(k_max) with bootstrap CIs; report the WHOLE curve incl. decline).
2. Extend `tools/validate_dataset.py` into the real D1 experiment: repeated nested CV
   (e.g. 10x5), bootstrap CIs, add the age+sex+APOE baseline, L2/elastic-net + random forest,
   k = 1,2,3,5,10,20,50,100,300,1305.
3. Implement D2 simulation calibrated to the S7/S1/S2 tables; sweep n informative extras, their
   effect sizes, correlation with the core markers, sample size, and prevalence.
4. Implement D3 extraction from `papers/epmc_ranked.json` + the 45 PDFs; stratify by cohort stage,
   reference standard, and platform; report heterogeneity, do not naively pool.
5. Likely needed: `uv add matplotlib seaborn` (+ DeLong test, ~30 lines of numpy).

## Uncertainties / risks to carry forward
- **n=36 primary stratum gives wide CIs.** D1 alone cannot confirm or refute the 90% threshold;
  it constrains the *shape* of the curve. Lean on D2/D3 for the quantitative claim, and do not
  overstate D1.
- GSE275392's AUC ceiling (~0.76) is platform-limited, NOT a refutation of blood biomarkers
  generally — it reflects the absence of p-tau217. Must be stated explicitly in any writeup.
- D2 is simulation: present as a sensitivity/boundary analysis, never as empirical evidence.
- Cross-study AUCs in D3 are not directly comparable (differing prevalence, reference standards,
  platforms). Report heterogeneity.
<!-- NEURICO_AGENT_NOTES_END:resource_finder -->

### experiment_runner
<!-- NEURICO_AGENT_NOTES_START:experiment_runner -->
Update this section at the end of the `experiment_runner` phase.
<!-- NEURICO_AGENT_NOTES_END:experiment_runner -->

<!-- NEURICO_AGENT_NOTES_END -->
