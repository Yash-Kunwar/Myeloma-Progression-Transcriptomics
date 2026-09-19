# AGENTS.md

Operating instructions for AI coding agents working in this repository.
**Read this file in full before writing or modifying any code.**

This project has TWO simultaneous objectives:
1. Produce rigorous, reproducible computational research.
2. **Teach the human collaborator the underlying biology, statistics, and structural
   biology as the work proceeds.** The human is an engineer with interests in biology and
   a beginner in hematology/oncology. Teaching is not optional — it is a graded
   deliverable.

---

## TABLE OF CONTENTS

1. Project Mission
2. Golden Rules (hard constraints)
3. Teaching Protocol (how to explain things)
4. Disease Primer Requirements
5. Dataset Contract
6. Research Questions & Task Definitions
7. Repository Layout
8. Phase Plan & Definition of Done
9. The Structural Biology Arm (AlphaFold & friends)
10. Statistics & ML Rules
11. Ethics, Scope & Anti-Overclaiming
12. Agent Workflow & Review Gates
13. Appendix A — Glossary (living document)
14. Appendix B — Verification Checklist

---

## 1. PROJECT MISSION

**Title (working):** *Early Detection and Structure-Informed Target Discovery in
Multiple Myeloma and Related Hematologic Malignancies*

**Scientific aim.** Multiple myeloma (MM) is one of very few cancers with a clinically
defined, biopsy-accessible precursor state. The natural history is:

```
Normal plasma cells → MGUS → Smoldering MM (SMM) → Active MM → Relapsed/Refractory MM
                        ↑                     ↑
            ~1%/yr progression risk   ~10%/yr progression risk (early years)
            (VERIFY these rates against a cited source before publishing them)
```

That progression axis IS the early-detection problem. The project must:

1. Build a **QC'd, batch-corrected, multi-cohort expression compendium** spanning the
   MM progression axis, plus a second arm covering other blood cancers (leukemias).
2. Identify genes and programs that change **at the precursor-to-malignant transition**,
   not merely genes that differ between healthy and late-stage disease.
3. Train **leakage-free classifiers** for disease stage, and **survival/risk models**
   where clinical outcome data exist.
4. Explain models (SHAP) and interpret them biologically (pathway enrichment).
5. **Close the loop to therapy:** take the top prioritized protein targets into
   structure prediction (AlphaFold/ESMFold/Boltz), pocket detection, docking, and
   resistance-mutation modeling — producing *testable therapeutic hypotheses*.
6. Teach the human every concept encountered, in writing, as it arises.

**The novelty claim of this repo is the closed loop:**
`transcriptomics → prioritized target → predicted structure → druggability / epitope /
resistance hypothesis`, fully reproducible in one codebase.

---

## 2. GOLDEN RULES

Violations mean the work is rejected and must be redone.

| # | Rule |
|---|------|
| **G1** | **Never fabricate, estimate, guess, or "illustrate" a number, gene, structure, or citation.** Every value in any output, notebook, README, figure, or chat reply must come from an actual executed run or a verified source. If not yet computed, write literally: `NOT YET COMPUTED`. |
| **G2** | **No data leakage.** Variance filtering, feature selection, scaling, batch correction fitting, and imputation happen **inside** each cross-validation fold via `sklearn.Pipeline`. Never fit any of these on the full dataset before splitting. |
| **G3** | **Multiple-testing correction is mandatory.** Report raw AND Benjamini–Hochberg adjusted p-values. Default threshold `adj_p < 0.05`. State the correction method in every table caption. |
| **G4** | **Never headline plain accuracy.** Primary classification metrics: **macro-F1** and **macro ROC-AUC (one-vs-rest)**. Primary survival metric: **Harrell's C-index** (plus time-dependent AUC). Class imbalance is severe throughout. |
| **G5** | **Determinism.** `RANDOM_SEED = 42` defined once in `src/config.py`, threaded into every stochastic component (numpy, sklearn, xgboost, torch, CV splitters). Pin every version in `requirements.txt` / `environment.yml`. Log library versions to `results/environment_report.txt`. |
| **G6** | **Know your data scale before transforming.** Microarray GEO series are often already normalized/log2-transformed; RNA-seq counts are not. The loader MUST detect and report scale (min/max/median, integer-vs-float) and MUST NOT blindly apply `log2`/`log1p`. Print the decision and its justification. |
| **G7** | **One phase at a time.** Do not jump ahead. Implement → run → report real numbers → write the teaching note → **stop and wait for human review**. |
| **G8** | **Nothing exists only in a notebook cell.** Every figure saved to `figures/` at 300 DPI with descriptive filename; every table to `results/` as CSV; every model to `models/` with its config. |
| **G9** | **Verify every external fact.** Accessions, sample counts, platform IDs, gene symbols, UniProt IDs, and drug–target pairs must be verified programmatically or against a named source, and the verification recorded in `results/provenance_report.md`. Facts inherited from this file are UNVERIFIED until checked. |
| **G10** | **Batch is the enemy.** In multi-cohort work, disease stage is frequently confounded with study/platform. Every cross-cohort claim requires an explicit confounding analysis. No exceptions. See §10.4. |
| **G11** | **Teaching is a deliverable.** Every phase produces a `docs/learn/NN_<topic>.md` teaching note. A phase is NOT done without it. See §3. |
| **G12** | **No clinical claims.** Never describe any output as diagnostic, prognostic-for-a-patient, or therapeutic. See §11. |
| **G13** | **Structure prediction is a hypothesis generator, not evidence.** Always report confidence metrics (pLDDT, PAE, ipTM/pTM) alongside any structural claim, and state limitations. See §9.6. |
| **G14** | **Negative and unimpressive results are reported, not hidden.** A near-chance boundary, a poor replication rate, or a sanity-check result must be stated plainly and explained, never buried or reframed as a positive. |
| **G15** | **When something is wrong, ambiguous, or restricted, stop and ask.** Never silently substitute a dataset, guess a label mapping, or proceed past a broken assumption. |

---

## 3. TEACHING PROTOCOL

The human wants to learn the science, not just receive code. Follow these formats
exactly.

### 3.1 Concept Cards

The **first time** any scientific, statistical, or structural term appears in code,
output, or explanation, emit a **Concept Card**. Keep it tight — no padding, no filler.

**Template (use verbatim structure):**

```
┌─ CONCEPT CARD ──────────────────────────────────────────────────────────────┐
│ TERM        : <the term, plus common abbreviation>                          │
│ CATEGORY    : Biology | Clinical | Statistics | ML | Structural Biology     │
│ ONE-LINER   : <plain-English definition in ≤ 50 words>                      │
│ WHY IT      : <why this project cares about it, in ≤ 2 sentences>           │
│   MATTERS                                                                   │
│ ANALOGY     : <one engineering/CS analogy — see §3.4 for constraints>       │
│ IN OUR DATA : <exactly where it appears: file, column, variable, or figure> │
│ COMMON      : <the specific mistake beginners make with this concept>       │
│   PITFALL                                                                   │
│ VERIFY      : <how the human can confirm this independently — a command,    │
│               a database page, or a named source>                           │
│ CONFIDENCE  : VERIFIED (source cited) | UNVERIFIED (recall only)            │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Filled example (this is the reference standard for quality):**

```
┌─ CONCEPT CARD ──────────────────────────────────────────────────────────────┐
│ TERM        : Monoclonal Gammopathy of Undetermined Significance (MGUS)     │
│ CATEGORY    : Clinical                                                      │
│ ONE-LINER   : An asymptomatic precursor state where a single plasma-cell    │
│               clone overproduces one antibody, without organ damage.        │
│ WHY IT      : MGUS is the earliest detectable point on the myeloma          │
│   MATTERS     progression axis, so it defines the "early stage" our         │
│               models are trying to catch.                                   │
│ ANALOGY     : A single misbehaving worker process that has started          │
│               replicating itself but has not yet exhausted host resources   │
│               or thrown user-visible errors.                                │
│ IN OUR DATA : Expected as a value in the `stage` column of                  │
│               results/metadata.csv, derived from GEO sample characteristics.│
│               Actual label strings NOT YET VERIFIED.                        │
│ COMMON      : Treating MGUS as "early cancer." It is a precursor state and  │
│   PITFALL     most MGUS never progresses. Mislabeling it inflates apparent  │
│               model performance and is clinically wrong.                    │
│ VERIFY      : International Myeloma Working Group (IMWG) diagnostic         │
│               criteria. Confirm progression rates from a cited primary      │
│               study before writing any rate into the README.                │
│ CONFIDENCE  : UNVERIFIED (recall only — confirm against IMWG criteria)      │
└─────────────────────────────────────────────────────────────────────────────┘
```

Every emitted card is appended verbatim to `docs/learn/GLOSSARY.md` in the same turn.
A card that exists in chat but not in the glossary does not count (G11).

### 3.2 Concept Card trigger rules

Emit a card on **first appearance** of:

- a disease, cell type, gene, protein, or pathway
- a clinical stage, biomarker, or scoring system
- a statistical method, metric, or correction
- an ML technique whose *biological* implication matters
- a structural-biology term or confidence metric
- a bioinformatics-specific database, platform, or file format

Do **not** emit cards for general software concepts the human already knows
(dataframe, git, Docker, JSON, the mechanics of cross-validation).

If unsure whether the human knows a term: **emit the card.** Redundancy is cheap;
silent confusion is expensive.

### 3.3 Teaching Notes

One per phase, at `docs/learn/NN_<topic>.md`, numbered to match §8.

Required structure:

```markdown
# NN — <Topic>

**Phase:** NN
**Prerequisites:** <earlier notes to read first>
**New Concept Cards introduced:** <list>

## 1. The question this phase answers
<Plain-language framing. Why a scientist would care.>

## 2. The biology / statistics you need
<Actual mechanism. No hand-waving. Analogies allowed but never as substitutes.>

## 3. What we actually did
<Methods in prose, with the exact function/file names used.>

## 4. What we actually found
<REAL numbers only, copied from execution output. Reference saved figure paths.
 If nothing was computed yet, write NOT YET COMPUTED.>

## 5. What could be wrong
<Confounders, batch risk, sample size, purity, label ambiguity. Be specific.>

## 6. Checkpoint
<3–5 questions the human should be able to answer, with answers in a
 collapsed <details> block.>

## 7. Going deeper
<Named sources for self-study. Never invent citations — if unsure, say
 "search term: <term>" instead of fabricating a paper.>
```

### 3.4 Analogy rules

1. **One analogy per card or concept.** Never stack them.
2. **Draw from software/systems engineering** — the human's native domain.
3. **State the break point** whenever an analogy could mislead, on the next line:
   `ANALOGY BREAKS DOWN: <where it fails>`
4. **Never analogize a clinical decision.** Do not compare treating a patient to
   deploying a fix, restarting a service, or rolling back a release. The consequences
   are asymmetric and the analogy trivializes them.
5. **Never let an analogy replace mechanism.** If the human needs the real biology,
   give the real biology.

### 3.5 Checkpoints

At each phase gate, ask the human 3–5 comprehension questions. At least one must be a
**trap question** targeting the phase's most likely misconception (e.g. "our top gene
has adj_p = 1e-12 — does that mean it causes progression?"). Provide answers in a
collapsed block so the human can self-test first.

If the human answers incorrectly, **re-teach before proceeding.** Do not advance the
phase.

### 3.6 Escalating depth

Teach the same concept at increasing depth as it recurs, rather than repeating the card:

```
Tier 1 (first mention)  → Concept Card
Tier 2 (when used)      → how it is computed, with the actual code path
Tier 3 (when it bites)  → failure modes, assumptions violated, alternatives
```

---

## 4. DISEASE PRIMER REQUIREMENTS

Before **any** modeling begins, Phase 0 produces `docs/learn/01_blood_cancer_primer.md`.
This is a hard prerequisite gate (§12).

### 4.1 Required primer content

| # | Topic | Must cover |
|---|---|---|
| **P1** | Normal hematopoiesis | Stem cell → lineage branching → mature cells. Where B cells and plasma cells sit. |
| **P2** | What a plasma cell is | Terminal B-cell differentiation, antibody factory, normally rare in marrow, long-lived. |
| **P3** | Why myeloma exists | Clonal plasma cell expansion in bone marrow; monoclonal protein; consequences (bone lesions, anemia, renal impairment, hypercalcemia). |
| **P4** | The progression axis | NORMAL_PC → MGUS → SMM → MM → RRMM, with the fact that most precursor cases never progress. |
| **P5** | Myeloma vs. leukemia vs. lymphoma | Which cell, which compartment, why they are separate diseases despite all being "blood cancer." |
| **P6** | Molecular subtypes | Translocations, hyperdiploidy, 1q gain, del(17p), and that MM is genetically heterogeneous, not one disease. |
| **P7** | How samples are obtained | Bone marrow aspirate, CD138⁺ selection, and why purity confounds everything (see `docs/agent/DATA.md` §4.4). |
| **P8** | Current therapy landscape | Proteasome inhibitors, IMiDs, anti-CD38 antibodies, BCMA-directed CAR-T and bispecifics — at concept level, to motivate the structural arm. |
| **P9** | What "cure" means here | MRD negativity, remission, relapse. Why MM is generally treatable-but-relapsing rather than cured. |
| **P10** | Why early detection is contested | Overdiagnosis, overtreatment, and the open question of treating high-risk SMM. |

### 4.2 Primer rules

- Every clinical rate, criterion, or drug class must carry `VERIFIED (<source>)` or
  `UNVERIFIED` (G9).
- Every gene or protein named must carry an HGNC symbol and, where relevant, a UniProt
  accession — both verified, not recalled.
- The primer must end with a **"what we do not know" section** listing genuine open
  questions in the field, so the human understands the project is entering a live debate.
- Maximum length ~2,500 words. This is orientation, not a textbook.

---

## 5. DATASET CONTRACT

**Full specification lives in `docs/agent/DATA.md`. Read it before Phase 1.**
The binding summary:

1. **Nothing in the accession registry is trusted.** Every accession, sample count, and
   platform ID must be fetched, asserted, compared to the claim, and recorded in
   `data/registry/datasets_verified.yaml`. Mismatch → loud warning → **stop** (G15).
2. **Never substitute a similar dataset silently.** If an accession is wrong or
   superseded, report and ask.
3. **Never commit controlled-access data** (MMRF CoMMpass, dbGaP, EGA) in any form.
   Restricted paths live only in gitignored `data/raw/restricted/`.
4. **Cross-cohort duplicate detection is mandatory** before any external validation
   claim: GSM ID overlap plus near-duplicate expression correlation. Findings go to
   `results/sample_overlap_report.csv`.
5. **Labels come from metadata only** — never inferred from expression (circularity).
   Original text preserved in `label_raw`; canonical mapping via an explicit,
   human-reviewable `STAGE_MAP` in `src/config.py`.
6. **Canonical stage vocabulary:** `NORMAL_PC, MGUS, SMM, MM_NEW, MM_RELAPSE, PCL,
   CELL_LINE, OTHER_BM, UNKNOWN`. Unmapped strings → `UNKNOWN` + printed warning.
7. **`CELL_LINE` excluded from all patient-level analyses by default**, with the
   exclusion stated in every affected caption.
8. **Purity is a first-class covariate**, captured in a `purification` column, never
   folded into stage.
9. **Scale detection before transformation** (G6) — the loader reports and justifies;
   it never blindly transforms.
10. **Reference signatures** (GEP70, EMC-92, IFM-15, UAMS subtypes) must come from
    primary sources with recorded provenance. If unobtainable, the overlap test is
    **not run** and is marked `NOT YET COMPUTED`.

---

## 6. RESEARCH QUESTIONS & TASK DEFINITIONS

Tasks are ordered by scientific value, not by ease. Do not lead with the easy one.

### RQ1 — Can we characterize and detect the precursor-to-malignant transition?

**This is the headline.**

| Task | Definition | Primary metric | Notes |
|---|---|---|---|
| **T1.1** | Ordinal stage classification: `NORMAL_PC → MGUS → SMM → MM_NEW` | macro-F1, macro ROC-AUC (OvR), plus ordinal-aware metric (e.g. quadratic-weighted kappa) | Cell lines excluded. Stage order is meaningful — a NORMAL_PC→MM error is worse than MGUS→SMM. |
| **T1.2** | The critical boundary: `MGUS + SMM` (precursor) vs. `MM_NEW` (malignant) | macro-F1, macro ROC-AUC, PR-AUC | This is the clinically meaningful decision boundary. Report PR-AUC because the classes are imbalanced. |
| **T1.3** | The hardest boundary: `MGUS` vs. `SMM` | macro-F1, balanced accuracy | Expect weak performance. A near-chance result here is a **legitimate and publishable negative finding** (G14) — precursor states may not be transcriptionally separable in bulk data. |
| **T1.4** | Sanity check: `NORMAL_PC` vs. `MM_NEW` | macro ROC-AUC | Expect near-perfect. Report it, then **explicitly explain why it is not impressive** (huge effect size, tiny normal-donor n, purity confound). Required paragraph in README. |

### RQ2 — Which genes and programs change at the transition, and do they replicate?

| Task | Definition | Primary output |
|---|---|---|
| **T2.1** | Differential expression across adjacent stage pairs (NORMAL_PC↔MGUS, MGUS↔SMM, SMM↔MM_NEW) | `results/deg_<pairA>_vs_<pairB>.csv` with logFC, raw p, BH adj_p |
| **T2.2** | Monotonic trend genes: expression changing consistently along the ordered axis | `results/monotonic_trend_genes.csv` (Jonckheere–Terpstra or Spearman vs. stage rank) |
| **T2.3** | Pathway enrichment per transition (KEGG, GO-BP, MSigDB Hallmark) | `results/enrichment_<transition>_<library>.csv` |
| **T2.4** | **Signature overlap validation:** do data-driven top-N genes recover GEP70 / EMC-92 / IFM-15 above chance? | `results/signature_overlap_test.csv` — hypergeometric p, observed vs. expected overlap, universe size stated |
| **T2.5** | **Replication:** do T2.1 hits reproduce in an independent cohort? | `results/replication_report.csv` — direction concordance, effect-size correlation, % replicated at adj_p < 0.05 |

**T2.5 is non-optional.** A gene list that does not replicate is a batch artifact until
proven otherwise. Report the replication rate prominently, even if it is poor (G14).

### RQ3 — Does the pipeline generalize to other blood cancers?

| Task | Definition | Primary metric |
|---|---|---|
| **T3.1** | Multi-class leukemia/lymphoid malignancy classification (Arm B) | macro-F1, per-class F1, confusion matrix |
| **T3.2** | Cross-disease feature comparison: are MM transition genes shared or MM-specific? | overlap analysis + `results/cross_disease_gene_overlap.csv` |

### RQ4 — Where outcome data exist, can we model risk?

| Task | Definition | Primary metric |
|---|---|---|
| **T4.1** | Overall survival modeling in cohorts with curated outcomes | **Harrell's C-index** (CV mean ± std), time-dependent AUC |
| **T4.2** | Comparison against published prognostic signatures as baselines | ΔC-index vs. GEP70/EMC-92 score, with CI |
| **T4.3** | Progression-to-MM modeling in precursor cohorts, **if and only if** progression events exist in metadata | C-index; otherwise `NOT YET COMPUTED — no progression events in available metadata` |

**T4.3 warning:** most public precursor series do **not** contain progression follow-up.
Do not fabricate an outcome variable, do not proxy progression with stage label, and do
not silently reframe T4.3 as T1.2. If events are absent, say so.

### RQ5 — Can transcriptomic hits become structure-informed therapeutic hypotheses?

| Task | Definition | Primary output |
|---|---|---|
| **T5.1** | Target prioritization scoring of RQ2 genes | `results/target_priority.csv` |
| **T5.2** | Structure prediction for top targets | `data/structures/` + `results/structure_qc.csv` |
| **T5.3** | Pocket detection / epitope surface analysis | `results/pockets.csv`, `results/epitope_report.csv` |
| **T5.4** | Docking of known/candidate ligands | `results/docking_scores.csv` |
| **T5.5** | Resistance-mutation interface modeling (WT vs. mutant) | `results/resistance_models.csv` |

Full specification in §9 and `docs/agent/STRUCTURE.md`.

### 6.1 Task ordering rule

Implement in this order: **T1.4 (sanity) → T1.1 → T1.2 → T2.1 → T2.3 → T2.4 → T2.5 →
T1.3 → T3.x → T4.x → T5.x**.

Rationale: establish the sanity check early so a broken pipeline is caught immediately,
but **never present T1.4 as a result**. The README leads with T1.2 and T2.5.

---

## 7. REPOSITORY LAYOUT

See the layout block in §6 of the core mission section. Binding additions:

| Rule | Requirement |
|---|---|
| **R1** | `python -m src.run_all` regenerates **every** figure in `figures/` and **every** CSV in `results/` from scratch, with no notebook execution required. |
| **R2** | Notebooks are thin narrative wrappers over `src/`. **No analysis logic lives only in a notebook.** If a notebook cell computes something reportable, that computation belongs in `src/`. |
| **R3** | `src/config.py` is the single source of truth for paths, `RANDOM_SEED`, thresholds, `STAGE_MAP`, and library-agnostic constants. No magic numbers elsewhere. |
| **R4** | Long-running structural jobs cache to `data/structures/` and are skipped if present, printing an explicit cache-hit notice with the cached file's checksum. |
| **R5** | `results/environment_report.txt` is regenerated on every `run_all` invocation, capturing Python version, OS, and all pinned library versions (G5). |
| **R6** | Every figure filename encodes phase and content: `NN_<slug>.png` (e.g. `04_volcano_mgus_vs_mmnew.png`). |
| **R7** | `data/raw/` is never modified after download. All derived artifacts go to `data/processed/`. |
| **R8** | `.gitignore` must exclude `data/raw/restricted/`, large matrices, and any credential file. Verify this before the first commit. |

---

## 8. PHASE PLAN & DEFINITION OF DONE

Work sequentially. A phase is complete only when **every** DoD box is checked and the
human has approved the gate (§12). Full detail in `docs/agent/PHASES.md`.

### Phase 0 — Orientation & Environment
- [ ] `docs/learn/00_orientation.md` — project map, what each phase will produce
- [ ] `docs/learn/01_blood_cancer_primer.md` — all P1–P10 topics (§4)
- [ ] `environment.yml` + `requirements.txt`, pinned; environment builds clean
- [ ] `src/config.py` with `RANDOM_SEED`, paths, empty `STAGE_MAP` scaffold
- [ ] `results/environment_report.txt` generated
- **Gate:** human confirms they understand the disease framing and can answer the primer checkpoint.

### Phase 1 — Data Acquisition & Verification
- [ ] `src/data_loader.py` implementing the loader contract (`docs/agent/DATA.md` §5)
- [ ] Every candidate accession fetched and verified; `data/registry/datasets_verified.yaml` written
- [ ] `results/provenance_report.md` — claimed vs. actual for every dataset
- [ ] Scale detection report per dataset; transformation decision printed and justified (G6)
- [ ] `results/sample_overlap_report.csv` — cross-cohort duplicate check
- [ ] `docs/learn/02_reading_geo_data.md`
- **Gate:** no unexplained shape/platform mismatch; duplicates resolved or flagged.

### Phase 2 — QC, Metadata & Harmonization
- [ ] `src/qc.py`, `src/metadata.py`, `src/harmonize.py`
- [ ] Missing-value audit; per-sample and per-feature NaN profile
- [ ] **Figures:** `02_expression_histograms.png`, `02_sample_boxplots.png`, `02_pca_by_study.png`, `02_pca_by_stage.png`
- [ ] `results/metadata.csv` with `label_raw`, `stage`, `purification`, `study`, `platform`
- [ ] Class-count table printed; sums exactly to retained sample count
- [ ] Zero unmapped labels, or every unmapped string listed with counts
- [ ] `results/confounding_report.csv` — stage × study contingency table + association test (§10.4)
- [ ] Purity proxy computed; correlation with stage reported
- [ ] `docs/learn/03_qc_and_batch_effects.md`
- **Gate:** the confounding report is reviewed *before* any DEG or model is run. If stage and study are near-perfectly confounded, **stop and redesign** — do not proceed.

### Phase 3 — Differential Expression
- [ ] `src/annotate.py` — probe→HGNC mapping, duplicate collapse rule documented
- [ ] `src/deg.py` — adjacent-pair DE (T2.1), monotonic trend (T2.2)
- [ ] Covariate adjustment for study/purity where design permits (§10.3)
- [ ] `results/deg_*.csv` with logFC, raw p, BH adj_p, n per group
- [ ] Counts of significant genes at `adj_p < 0.05` and `|logFC| > 1` reported per contrast
- [ ] `docs/learn/04_differential_expression.md`
- **Gate:** significance counts are plausible and the multiple-testing method is stated in every table caption.

### Phase 4 — Visualization
- [ ] **Figures:** `04_volcano_<contrast>.png`, `04_heatmap_top50_<contrast>.png`, `04_stage_trajectory_pca.png`, optional `04_umap.png`
- [ ] Heatmaps annotated with **both** stage and study, so batch structure is visible
- [ ] `docs/learn/05_reading_these_figures.md`
- **Gate:** captions state factually whether separation is visible. **Never claim clustering that is not there** (G1).

### Phase 5 — Feature Selection
- [ ] `src/features.py` — variance filter, ANOVA F, L1-logistic, RF importance, mutual information
- [ ] Selection implemented as **pipeline steps**, verified fit-inside-fold (G2)
- [ ] Top-50/100/500 sets per method; `results/feature_rankings.csv`
- [ ] Method-agreement Jaccard matrix; `results/feature_method_overlap.csv`
- [ ] `docs/learn/06_feature_selection.md`
- **Gate:** a leakage unit test (§10.2) passes, demonstrating selection sees only training folds.

### Phase 6 — Classification
- [ ] `src/models.py` — Logistic Regression → Random Forest → XGBoost → optional MLP **last**
- [ ] `RepeatedStratifiedKFold(n_splits=5, n_repeats=5, random_state=42)`
- [ ] **Grouped CV by study** for any cross-cohort task (`StratifiedGroupKFold`), so folds never split a study
- [ ] Class weighting: `class_weight='balanced'` (LR/RF) or `sample_weight` (XGBoost)
- [ ] Metrics: accuracy, macro precision/recall/F1, macro ROC-AUC (OvR), PR-AUC, per-class F1
- [ ] Ordinal-aware metric for T1.1 (quadratic-weighted kappa)
- [ ] `results/model_metrics.csv` with **mean ± std across folds**, never a single split
- [ ] **Figures:** `06_confusion_matrix_<task>_<model>.png`, `06_roc_curves_<task>.png`, `06_pr_curves_<task>.png`
- [ ] Permutation baseline: labels shuffled, full pipeline rerun; `results/permutation_baseline.csv`
- [ ] `docs/learn/07_classification_and_metrics.md`
- **Gate:** permutation baseline sits near chance. If a shuffled-label model scores well, the pipeline leaks — **stop and fix** before reporting anything.

### Phase 7 — Explainability
- [ ] `src/explain.py` — SHAP on the best tree model, computed **per fold** then aggregated
- [ ] **Figures:** `07_shap_beeswarm_<task>.png`, `07_shap_bar_<task>.png`, `07_shap_waterfall_sample<N>.png`
- [ ] `results/shap_gene_ranking.csv` — mean |SHAP| with across-fold std
- [ ] Stability check: rank correlation of SHAP orderings across folds; `results/shap_stability.csv`
- [ ] `docs/learn/08_explainability.md` — including the causality warning (§10.7)
- **Gate:** SHAP rankings are stable across folds, or the instability is reported as a limitation.

### Phase 8 — Pathway Enrichment
- [ ] `src/enrichment.py` — `gseapy` against KEGG, GO-BP, MSigDB Hallmark, Reactome
- [ ] Both over-representation (on DE gene sets) **and** GSEA (on the full ranked list)
- [ ] Background universe = genes measurable after filtering, stated explicitly (§10.6)
- [ ] `results/enrichment_<contrast>_<library>.csv` with adj_p and gene-set size
- [ ] **Figures:** `08_enrichment_dotplot_<contrast>.png`
- [ ] `docs/learn/09_pathway_enrichment.md`
- **Gate:** universe size stated in every table caption; no enrichment claim rests on a gene set with < 5 overlapping genes.

### Phase 9 — Signature Validation & Replication
- [ ] Reference signatures obtained from primary sources; `data/signatures/PROVENANCE.md`
- [ ] Identifier mapping with loss reported; `results/signature_mapping_report.csv`
- [ ] Hypergeometric overlap test (T2.4); `results/signature_overlap_test.csv`
- [ ] Independent-cohort replication (T2.5); `results/replication_report.csv`
- [ ] **Figures:** `09_replication_effect_size_scatter.png`, `09_signature_overlap.png`
- [ ] `docs/learn/10_validation_and_replication.md`
- **Gate:** replication rate reported prominently regardless of outcome (G14). A poor rate is reported, not hidden.

### Phase 10 — Survival Modeling (conditional)
- [ ] Confirm outcome fields actually exist; if not, write `NOT YET COMPUTED — no outcome data` and skip
- [ ] `src/survival.py` — Cox PH, penalized Cox, Random Survival Forest
- [ ] Harrell's C-index, CV mean ± std; time-dependent AUC
- [ ] Published-signature baselines for ΔC-index comparison
- [ ] **Figures:** `10_km_curves_riskgroups.png`, `10_cindex_comparison.png`
- [ ] `docs/learn/11_survival_analysis.md`
- **Gate:** risk-group KM curves derived from **held-out** predictions only. Never split on training-set risk scores.

### Phase 11 — Target Prioritization
- [ ] `src/targets.py` — composite scoring (§9.2)
- [ ] External evidence joined: DepMap dependency, HPA localization, UniProt topology, existing drug/clinical status
- [ ] `results/target_priority.csv` with every component score and its source
- [ ] `docs/learn/12_target_prioritization.md`
- **Gate:** every external evidence field carries a verified source (G9). Top targets manually sanity-checked against known MM biology.

### Phase 12 — Structural Biology Arm
- [ ] Full DoD in §9.7 and `docs/agent/STRUCTURE.md`
- [ ] `docs/learn/13_structure_prediction.md`, `docs/learn/14_docking_and_druggability.md`
- **Gate:** every structural claim carries pLDDT/PAE/pTM and an explicit limitation statement (G13).

### Phase 13 — Synthesis & Write-Up
- [ ] `README.md` — aim, methods, **real** results table, key figures, limitations, scope disclaimer
- [ ] `docs/learn/15_what_we_found_and_what_it_means.md`
- [ ] `results/RESULTS_SUMMARY.md` — every headline number with its source file
- [ ] Limitations section covering: sample size, batch confounding, purity, single-platform bias, absent progression outcomes, structure-prediction uncertainty, no wet-lab validation
- [ ] Full `python -m src.run_all` reproducibility check from a clean clone
- **Gate:** the human can defend every number in the README, and no clinical claim appears anywhere (G12).

---

## 9. THE STRUCTURAL BIOLOGY ARM

**Full specification in `docs/agent/STRUCTURE.md`. Binding rules here.**

### 9.1 What this arm is and is not

This arm converts a prioritized gene into a **testable therapeutic hypothesis**. It does
not detect cancer, does not predict progression, and does not demonstrate efficacy.

Explicitly out of scope for structure prediction:
- ❌ predicting whether a patient progresses
- ❌ predicting the functional effect of an arbitrary missense variant from structure alone
  (structure often barely changes for pathogenic variants)
- ❌ replacing crystallography, cryo-EM, or any assay
- ❌ claiming a docking score is a binding affinity

### 9.2 Target prioritization score

Composite, with every component sourced and weighted transparently in `src/config.py`:

| Component | Source | Notes |
|---|---|---|
| Transition effect size | our DEG output | logFC at the precursor→malignant boundary |
| Statistical robustness | our DEG output | adj_p, plus replication status from T2.5 |
| Model importance | our SHAP output | mean \|SHAP\| with cross-fold stability |
| Cell-surface accessibility | UniProt topology + HPA | required for CAR-T/bispecific hypotheses |
| Cancer-cell dependency | DepMap CRISPR | dependency in MM lines vs. other lineages |
| Normal-tissue expression | HPA / GTEx | **penalty** for broad essential-tissue expression (toxicity risk) |
| Existing tractability | drug databases, clinical status | flags both "validated" and "already crowded" |

Rule: **no single component may dominate.** Report the full component breakdown per target,
not just the composite, so the human can see *why* something ranked highly.

### 9.3 Structure prediction tooling

| Tool | Use | Constraint |
|---|---|---|
| **AlphaFold DB lookup** | first action — check if a structure already exists | Always check before predicting. Cite the DB entry. |
| **PDB experimental lookup** | second action | An experimental structure beats any prediction. Use it. |
| **ColabFold / AlphaFold2** | monomer + complex prediction | Record MSA depth; shallow MSAs → low confidence |
| **ESMFold** | fast single-sequence prediction | Lower accuracy; use for triage only, never for final claims |
| **AlphaFold3 / Boltz / Chai** | ligand- and complex-aware prediction | Check licence terms before use; record model version |

Rule: **never predict what already exists experimentally.** The pipeline must query PDB
and AlphaFold DB first and log the decision.

### 9.4 Sequence input rules

- Canonical sequence from **UniProt**, accession recorded, isoform explicitly chosen.
- For surface targets, model the **ectodomain** separately (topology-defined boundaries
  from UniProt), not the full-length protein — transmembrane and cytoplasmic regions
  degrade pocket/epitope analysis.
- Signal peptides removed where appropriate, and the removal documented.
- Every sequence used is written to `data/structures/<target>/input.fasta` with its
  UniProt accession and version in the header.

### 9.5 Pockets, epitopes, docking

- Pocket detection on the predicted structure, filtered to residues with adequate pLDDT.
- Epitope/surface analysis for surface targets: accessibility, predicted glycosylation
  sites (these can block antibody binding), domain boundaries.
- Docking: report the scoring function, software version, search box, and number of runs.
  Always include **decoy/negative controls** — dock known non-binders and report their
  scores alongside candidates.
- Resistance modeling: predict WT and mutant, compare interface geometry, and report
  confidence for **both**. Never compare a high-confidence WT to a low-confidence mutant.

### 9.6 Confidence reporting (enforces G13)

Every structural claim must be accompanied by:

| Metric | Required for |
|---|---|
| **pLDDT** — per-residue, **and specifically for the pocket/epitope residues** | all predictions |
| **PAE / PAE matrix** | domain-boundary and multi-domain claims |
| **pTM** | overall fold confidence |
| **ipTM** | any complex/interface claim |
| **MSA depth** | AlphaFold-family predictions |

Banned: reporting a single whole-protein mean pLDDT as justification for a pocket claim.

### 9.7 Structural arm DoD

- [ ] PDB + AlphaFold DB checked first; decision logged per target
- [ ] `data/structures/<target>/` containing input FASTA, predicted structure, confidence files
- [ ] `results/structure_qc.csv` — pLDDT summary, pocket-residue pLDDT, pTM, ipTM, MSA depth
- [ ] `results/pockets.csv`, `results/epitope_report.csv`
- [ ] `results/docking_scores.csv` including decoy controls
- [ ] `results/resistance_models.csv` with WT and mutant confidence side by side
- [ ] **Figures:** `12_structure_<target>_plddt.png`, `12_pae_<target>.png`, `12_pocket_<target>.png`
- [ ] Every claim paired with a stated limitation

---

## 10. STATISTICS & ML RULES

**Full specification in `docs/agent/METHODS.md`.** Binding rules:

### 10.1 Cross-validation
- `RepeatedStratifiedKFold(n_splits=5, n_repeats=5, random_state=42)` for within-cohort tasks.
- `StratifiedGroupKFold` grouped **by study** for any cross-cohort task — folds must never split a study.
- Never report a single train/test split. Sample sizes here are too small for one holdout to be meaningful.
- Nested CV whenever hyperparameters are tuned: outer loop for evaluation, inner loop for tuning.

### 10.2 Leakage tests

Three mandatory tests, implemented as unit tests in `tests/test_leakage.py` and run
before any metric is reported:

| Test | Method | Pass condition |
|---|---|---|
| **LT1 — Selector isolation** | Instrument the feature selector to record which sample indices it observed during `fit`. | Recorded indices are a strict subset of the training fold. Zero test-fold indices seen. |
| **LT2 — Permutation baseline** | Shuffle labels, rerun the entire pipeline unchanged. | Macro-F1 ≈ chance for the class distribution; macro ROC-AUC ≈ 0.5. |
| **LT3 — Duplicate isolation** | Using `results/sample_overlap_report.csv`, confirm no near-duplicate sample pair is split across train and test. | Zero cross-fold duplicate pairs. |

If **LT2** produces above-chance performance, the pipeline leaks. **Stop. Do not report
any result.** Find the leak first. This is the single most common way gene-expression ML
papers become irreproducible.

### 10.3 Differential expression
- Default: moderated t-statistics (limma-style) or Welch's t-test. State which and why.
- **Adjust for study and purity as covariates** wherever the design permits it. If stage
  and study are perfectly confounded for a given contrast, that contrast **cannot** be
  adjusted — report it as unadjustable rather than pretending otherwise.
- Report `n` per group in every DEG table. A contrast with n=4 vs. n=6 is a pilot
  observation, not a finding, and must be labeled as such.
- For the ordinal axis, use a trend test (Jonckheere–Terpstra or Spearman vs. stage rank),
  not a series of pairwise tests reinterpreted as a trend.

### 10.4 Confounding analysis (enforces G10)

Before **any** cross-cohort DEG or model:

1. Build the `stage × study` contingency table → `results/confounding_report.csv`.
2. Test association (chi-square or Fisher, as appropriate for cell counts).
3. Compute Cramér's V as an effect size for the confounding strength.
4. Run PCA and report **variance explained by study vs. by stage** on the top PCs.
5. Visualize both: `02_pca_by_study.png` and `02_pca_by_stage.png`, same axes.

Decision rule:

```
IF study explains more top-PC variance than stage
   AND stage×study Cramér's V is high
THEN cross-cohort claims are UNSAFE.
     → Restrict primary analysis to within-study contrasts
     → Use cross-study only for replication, never for discovery
     → State this prominently in README limitations
```

Batch correction (ComBat, `removeBatchEffect`, etc.) is permitted **only** when:
- the batch variable is not collinear with the outcome, and
- the correction is fit **inside** the CV fold (G2), and
- corrected and uncorrected results are both reported.

Never batch-correct on the full dataset and then cross-validate. That leaks label
information through the correction.

### 10.5 Class imbalance
- Always report per-class metrics alongside macro averages. A macro-F1 of 0.62 can hide a
  class at 0.0.
- Use `class_weight='balanced'` (LR/RF) or `sample_weight` (XGBoost).
- **Do not use SMOTE or any synthetic oversampling on high-dimensional expression data**
  without explicit justification and a side-by-side comparison against class weighting.
  Interpolating between samples in 20,000-dimensional space creates biologically
  meaningless points.
- If any class has n < 10, say so in every caption and treat its metrics as indicative only.

### 10.6 Enrichment statistics
- Background universe = genes measurable in the processed matrix **after filtering**.
  Not all human genes. Not the raw probe count. State the universe size in every caption.
- Report gene-set size and the number of overlapping genes for every enriched term.
- No enrichment claim rests on fewer than 5 overlapping genes.
- Run both over-representation (on the DE gene set) and GSEA (on the full ranked list).
  Where they disagree, report the disagreement.
- Correct across gene sets within each library, and state that the correction is
  within-library.

### 10.7 The causality warning

SHAP importance, DEG significance, and model weights measure **association under this
model, in this dataset**. They do not establish:

- that a gene causes progression
- that perturbing the gene would change disease course
- that the gene is a valid drug target

Every teaching note and README section touching feature importance must carry this
statement. Phrase findings as "associated with," "discriminates between," or
"prioritized for follow-up" — never "drives," "causes," or "responsible for."

### 10.8 Survival modeling
- Harrell's C-index as primary, CV mean ± std. Time-dependent AUC as secondary.
- Check proportional-hazards assumptions for any Cox model; report Schoenfeld residual
  tests.
- Risk-group KM curves must be derived from **held-out** predictions only. Splitting on
  training-set risk scores manufactures separation and is a known fraud pattern.
- Compare against published signature baselines with confidence intervals on the
  ΔC-index. A model that fails to beat GEP70 should say so plainly (G14).

---

## 11. ETHICS, SCOPE & ANTI-OVERCLAIMING

### 11.1 Required scope statement

This paragraph appears verbatim in `README.md`, in `docs/learn/00_orientation.md`, and in
any external write-up:

> This work is hypothesis-generating computational research. It uses bone-marrow-derived
> expression profiles from public cohorts, so it does not constitute a non-invasive
> screening test. No wet-lab, functional, or clinical validation was performed. Predicted
> protein structures and docking results are computational hypotheses, not evidence of
> binding or efficacy. Nothing in this repository is a diagnostic, prognostic, or
> therapeutic recommendation, and it must not be used to inform any clinical decision.

### 11.2 Banned language

| Never write | Write instead |
|---|---|
| "detects cancer early" | "discriminates precursor from malignant samples in this cohort" |
| "predicts progression" | "associated with stage label in cross-sectional data" |
| "identifies the driver" | "prioritizes candidate genes for follow-up" |
| "novel drug target" | "candidate target hypothesis requiring experimental validation" |
| "binds with high affinity" | "docking score of X under scoring function Y; not an affinity" |
| "validated biomarker" | "overlaps a published signature above chance" |
| "cures" / "treatment" | out of scope — do not use |

### 11.3 Cross-sectional vs. longitudinal

Nearly all public precursor data is **cross-sectional**: different patients at different
stages, sampled once. It cannot tell you what happens to an individual over time.

Therefore: a model that separates MGUS from MM is **not** a progression predictor. It is a
stage classifier. Conflating these is the central scientific error available in this
project, and the agent must actively guard against it in every write-up.

### 11.4 Data ethics
- No attempt to re-identify any donor, ever.
- No linkage of public expression data against any other identifying resource.
- Controlled-access data never enters the repository (§5, `docs/agent/DATA.md`).
- If a dataset's licence or terms of use are unclear, **stop and ask** (G15).

### 11.5 Honest failure

If the headline result is negative — precursor states are not separable, the signature
does not replicate, no druggable pocket is found — that is the finding. Report it clearly
and prominently. A well-executed negative result is a stronger portfolio artifact than an
overclaimed positive one, and vastly stronger than a leaked one.

---

## 12. AGENT WORKFLOW & REVIEW GATES

### 12.1 The per-phase loop

```
1. RESTATE    → the phase goal, in one paragraph, in your own words
2. PLAN       → files to create/modify, functions, expected outputs, expected runtime
3. CONFIRM    → STOP. Wait for human approval of the plan.
4. IMPLEMENT  → code in src/, thin wrapper in notebooks/
5. EXECUTE    → run it; capture real stdout verbatim
6. REPORT     → actual numbers, exact figure paths, anything that failed
7. TEACH      → docs/learn/NN_*.md + all new Concept Cards appended to GLOSSARY.md
8. GATE       → present the DoD checklist; STOP. Wait for review.
```

Never merge phases. Never pass a gate unprompted. Never begin step 4 before step 3
returns approval.

### 12.2 Hard stop conditions

Stop immediately and escalate to the human when:

| Condition | Why |
|---|---|
| An accession's actual content contradicts `docs/agent/DATA.md` | may invalidate the whole design |
| Stage and study are near-perfectly confounded | cross-cohort discovery is impossible |
| **LT2 permutation baseline scores above chance** | the pipeline leaks |
| Cross-cohort duplicate samples are found in a train/test split | validation is invalid |
| A required label mapping is ambiguous | guessing corrupts everything downstream |
| An outcome field expected for survival analysis does not exist | do not proxy it |
| A reference signature cannot be obtained from a primary source | do not reconstruct from memory |
| A structural prediction has pocket-residue pLDDT below threshold | claim is unsupportable |
| Any result would require a clinical claim to be interesting | reframe or drop it |

### 12.3 Reporting format for every phase

```
PHASE NN — <name>

STATUS: COMPLETE | BLOCKED | PARTIAL

WHAT RAN:
  <command(s) executed>

REAL NUMBERS:
  <copied verbatim from stdout — never retyped from memory>

FILES WRITTEN:
  figures/  <exact paths>
  results/  <exact paths>
  docs/learn/  <exact paths>

WHAT FAILED OR IS MISSING:
  <explicit; write NONE only if genuinely none>

UNVERIFIED CLAIMS REMAINING:
  <anything still resting on recall>

DoD CHECKLIST:
  [x] / [ ] per item

NEXT PHASE REQUIRES FROM HUMAN:
  <decisions, approvals, credentials, or answers needed>
```

### 12.4 When the agent is uncertain

Say so explicitly. Acceptable and expected:

- "I do not know whether this series used CD138 selection. It must be checked before the
  purity analysis is valid."
- "I cannot verify this signature's gene list from a primary source. The overlap test is
  `NOT YET COMPUTED`."
- "This C-index is from n=48 with 19 events. It is not interpretable as a stable estimate."

Unacceptable: filling the gap with a plausible-sounding number, gene, or citation (G1).

---

## 13. APPENDIX A — GLOSSARY (LIVING DOCUMENT)

`docs/learn/GLOSSARY.md` is the accumulated set of every Concept Card emitted, and is
maintained under these rules:

1. **Append-only during a phase.** Cards are added as they are emitted, verbatim.
2. **Alphabetized by term within category sections:** Biology, Clinical, Data/Platform,
   Statistics, ML, Structural Biology.
3. **Confidence upgrades are edits, not new cards.** When an `UNVERIFIED` card is later
   verified, edit the `CONFIDENCE` line in place and append the source. Log the upgrade in
   `results/provenance_report.md`.
4. **Every card carries `IN OUR DATA`.** If the term has no concrete presence yet, write
   `NOT YET PRESENT — expected in <file/column>` rather than omitting the field.
5. **Cross-references are explicit.** If a card depends on another concept, name it:
   `SEE ALSO: <term>`. No implicit chains.
6. **No orphan terms.** A term appearing anywhere in `docs/learn/`, `README.md`, or a
   figure caption must have a card. A pre-commit check (`tests/test_glossary.py`) scans
   those files for capitalized biological/statistical terms and fails on any term absent
   from the glossary.
7. **Deprecated terms are struck, not deleted.** If a term turns out to be wrong or
   misapplied (e.g. a misremembered gene symbol), mark the card as deprecated in place
   rather than removing it.

---

## 14. APPENDIX B — VERIFICATION CHECKLIST

Run this checklist at every phase gate. Any unchecked box blocks the gate.

### 14.1 Provenance facts

- [ ] Every accession used appears in `data/registry/datasets_verified.yaml` with actual
      (not claimed) sample count, feature count, platform, organism, tissue
- [ ] `results/provenance_report.md` shows claimed-vs-actual for every dataset, with
      mismatches flagged
- [ ] Every gene symbol in any output resolves to a current HGNC-approved symbol
- [ ] Every protein has a recorded UniProt accession **with isoform specified**
- [ ] Every clinical rate, criterion, or drug class carries `VERIFIED (<source>)`
- [ ] Zero invented citations. Where a source is unknown, the text reads
      `UNVERIFIED — search term: <term>` instead
- [ ] Every reference signature has `data/signatures/PROVENANCE.md` with source and
      access date, or is marked `NOT YET COMPUTED`

### 14.2 Data integrity

- [ ] Loader assertions pass (unique sample IDs, index/column alignment, numeric dtype,
      non-empty platform)
- [ ] Scale detection report printed per dataset; transformation decision justified (G6)
- [ ] Missing-value audit complete; samples with >5% missing listed and dispositioned
- [ ] Duplicate feature IDs resolved in `annotate.py` with the collapse rule documented
- [ ] `results/sample_overlap_report.csv` exists; cross-cohort duplicates resolved
- [ ] `label_raw` preserved for every sample; `STAGE_MAP` is explicit and human-reviewed
- [ ] Zero labels silently mapped; every unmapped string listed with its count
- [ ] `CELL_LINE` samples excluded from patient-level analyses, exclusion stated in captions
- [ ] `purification` column populated; purity proxy computed and correlated against stage

### 14.3 Confounding (G10)

- [ ] `results/confounding_report.csv` exists with stage × study table, association test,
      Cramér's V
- [ ] Top-PC variance attributed to study vs. stage, reported numerically
- [ ] `02_pca_by_study.png` and `02_pca_by_stage.png` produced on identical axes
- [ ] Decision rule in §10.4 applied and its outcome written into README limitations
- [ ] If batch correction was used: not collinear with outcome, fit inside folds,
      corrected **and** uncorrected results both reported

### 14.4 Leakage (G2)

- [ ] `tests/test_leakage.py` passes all three tests
- [ ] **LT1** — selector observed zero test-fold indices
- [ ] **LT2** — permutation baseline at chance; `results/permutation_baseline.csv` written
- [ ] **LT3** — zero near-duplicate pairs split across folds
- [ ] Every preprocessing step (scaling, filtering, imputation, selection, correction) is
      a pipeline step, not a pre-split operation
- [ ] Nested CV used wherever hyperparameters were tuned

### 14.5 Statistics

- [ ] Every p-value reported alongside its BH-adjusted counterpart
- [ ] Correction method stated in every table caption (G3)
- [ ] `n` per group reported in every DEG table; small-n contrasts labeled as pilot
- [ ] Trend tests used for the ordinal axis, not pairwise tests reinterpreted as a trend
- [ ] Enrichment universe size stated in every caption; no claim on < 5 overlapping genes
- [ ] GSEA and over-representation both run; disagreements reported
- [ ] Survival: PH assumption checked, Schoenfeld residuals reported, KM risk groups from
      held-out predictions only

### 14.6 Metrics & reporting

- [ ] No plain accuracy headlined anywhere (G4)
- [ ] Macro-F1 and macro ROC-AUC reported as **mean ± std across folds**
- [ ] Per-class metrics reported alongside macro averages
- [ ] Any class with n < 10 flagged in every caption
- [ ] PR-AUC reported for imbalanced binary tasks
- [ ] Quadratic-weighted kappa reported for the ordinal task
- [ ] T1.4 (normal vs. MM) accompanied by its required "why this is not impressive"
      paragraph
- [ ] SHAP stability across folds reported; instability disclosed if present

### 14.7 Structural arm (G13)

- [ ] PDB and AlphaFold DB queried before any prediction; decision logged per target
- [ ] UniProt accession, isoform, and sequence boundaries recorded per target
- [ ] Ectodomain modeled separately for surface targets, with topology source cited
- [ ] `results/structure_qc.csv` contains pLDDT summary, **pocket-residue pLDDT**, pTM,
      ipTM, MSA depth
- [ ] No claim rests on a whole-protein mean pLDDT
- [ ] Docking reports scoring function, software version, search box, run count
- [ ] Decoy/negative controls docked and reported alongside candidates
- [ ] Resistance models report WT and mutant confidence side by side
- [ ] Every structural claim paired with an explicit stated limitation

### 14.8 Teaching (G11)

- [ ] `docs/learn/NN_<topic>.md` exists for this phase, with all seven required sections
- [ ] Every new term has a Concept Card, emitted in chat **and** appended to
      `docs/learn/GLOSSARY.md`
- [ ] Every card has `IN OUR DATA` and `CONFIDENCE` populated
- [ ] Analogies obey §3.4; break points stated where needed; no clinical-decision analogies
- [ ] Checkpoint questions present, including at least one trap question, with answers in
      a collapsed block
- [ ] `tests/test_glossary.py` finds no orphan terms

### 14.9 Reproducibility & hygiene

- [ ] `python -m src.run_all` regenerates every figure and CSV from a clean clone