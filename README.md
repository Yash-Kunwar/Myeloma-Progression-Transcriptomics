# Myeloma Progression Transcriptomics

Gene expression analysis of Multiple Myeloma disease progression using the **GSE6477** public dataset (Affymetrix HG-U133A microarray, 152 patient samples across 5 disease stages).

> **Status: Active development** — core analysis complete, additional validation in progress.

---

## What is this project?

Multiple Myeloma (MM) is a blood cancer that starts as a pre-cancerous condition and progresses through distinct stages. This project asks: **can we see that progression in gene expression data?**

Using 152 bone marrow biopsy samples from patients at different stages of the disease, this analysis:
- Identifies which genes turn on or off at each stage transition
- Finds which biological pathways are activated or suppressed
- Builds machine learning models that distinguish healthy plasma cells from malignant ones
- Investigates whether the signals are genuine cancer biology or artefacts from sample contamination

## Dataset

| Property | Value |
|---|---|
| GEO Accession | [GSE6477](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE6477) |
| Platform | Affymetrix HG-U133A |
| Samples after QC | 152 |
| Genes after QC | 13,237 |
| Disease stages | Normal PC, MGUS, SMM, MM (newly diagnosed), MM (relapsed) |

> Data is downloaded automatically by GEOparse on first run — not stored in this repository.

## Disease stages

```
Normal Plasma Cell → MGUS → SMM → MM (Newly Diagnosed) → MM (Relapsed)
     (healthy)     (pre-cancer)  (smoldering)   (active cancer)    (returned)
```

- **MGUS** — Monoclonal Gammopathy of Undetermined Significance: abnormal protein in blood, no symptoms
- **SMM** — Smoldering Multiple Myeloma: more abnormal cells, still no organ damage
- **MM** — Active disease causing bone damage, kidney problems, or anemia

## Notebooks

| Notebook | What it does |
|---|---|
| [eda.ipynb](eda.ipynb) | Exploratory data analysis: sample QC, PCA, stage distributions |
| [analysis.ipynb](analysis.ipynb) | Differential expression, GSEA pathway analysis, UMAP, purity correction |
| [ml.ipynb](ml.ipynb) | Machine learning: binary Normal vs Disease classifier, 5-class staging, ordinal progression score |

## Key findings

**Pathway analysis (GSEA)**
- 27 Hallmark pathways are significantly different between Normal Plasma Cells and MGUS
- After removing 97 granulocyte-contaminated genes: 25 of 27 pathways survive — the signal is real biology, not a contamination artefact
- Cell cycle pathways (E2F Targets, G2M Checkpoint, Myc Targets) ramp up across progression
- Immune/inflammatory pathways (TNF-α/NF-κB, Complement) are suppressed in normal plasma cells

**Machine learning**
- Binary classifier (Normal vs Disease): 99.3% accuracy, AUC = 1.000 across 5-fold CV
- Normal_PC separates perfectly from all disease stages at the RNA level
- SMM is the hardest stage to classify — it sits on a continuum with no discrete gene expression boundary
- Ordinal progression score (Frank-Hall model): Spearman ρ = 0.692, monotonically increasing across all 5 stages

**Purity confound investigation**
- 94 of 238 genes down-regulated in the Normal→MGUS transition are correlated with granulocyte markers (39%)
- These likely reflect differences in bone marrow cell composition, not plasma cell biology
- After excluding these genes, core findings hold — particularly TNF-α/NF-κB signalling and Protein Secretion

## Reproducing this analysis

```bash
# Clone the repo
git clone https://github.com/Yash-Kunwar/Myeloma-Progression-Transcriptomics.git
cd Myeloma-Progression-Transcriptomics

# Install dependencies (Python 3.10+)
pip install -r requirements.txt

# Run notebooks in order: eda → analysis → ml
# GEO data (~27 MB) downloads automatically on first run
```

## Repository structure

```
├── eda.ipynb                     # Exploratory analysis
├── analysis.ipynb                # Main biological analysis
├── ml.ipynb                      # Machine learning models
├── requirements.txt              # Python dependencies
├── results/
│   ├── figures/                  # All generated plots
│   ├── pathways/                 # GSEA results (CSV)
│   ├── ml/                       # ML metrics and predictions
│   └── *.csv                     # Differential expression results
└── AGENTS.md                     # Analysis methodology guidelines
```

## Methods summary

- **Differential expression**: Welch's t-test with Benjamini-Hochberg FDR correction (threshold: FDR < 0.05, |log2FC| > 1)
- **GSEA**: Pre-ranked GSEA using MSigDB Hallmark gene sets (gseapy, 1000 permutations)
- **Dimensionality reduction**: UMAP on top 5,000 most variable genes
- **Classification**: Logistic Regression (L2) and Random Forest, 5-fold stratified CV, balanced class weights
- **Ordinal modelling**: Frank-Hall binary decomposition across 4 stage thresholds
- **Purity correction**: Granulocyte marker correlation analysis + hypergeometric enrichment test

## License

This project uses the publicly available GSE6477 dataset. Code is released under the MIT License.
