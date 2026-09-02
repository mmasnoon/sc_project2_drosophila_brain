# Cocaine-Induced Transcriptional Responses in the Drosophila Brain

## Project Overview

### Scientific Question

Cocaine inhibits monoamine reuptake, driving heightened synaptic signaling and altered locomotion in *Drosophila melanogaster*. Bulk RNA-seq cannot resolve which specific brain cell populations mediate this response.

This project investigates which transcriptionally distinct brain cell types respond most strongly to acute cocaine exposure, and whether the response differs by sex. The analysis follows a computational workflow covering quality control, normalization, dimensionality reduction, Leiden clustering, canonical-marker cell-type annotation, sex-stratified differential expression, and pathway enrichment.

### Dataset

The dataset consists of 10x Genomics single-cell gene-expression profiles from adult *Drosophila* brains following acute consumption of sucrose (control) or sucrose supplemented with cocaine, in both sexes and in duplicate (8 samples total: 2 sexes × 2 treatments × 2 replicates).

The raw dataset comprises approximately **88,991 cells × 17,481 genes** before quality control. After QC filtering (≥200 genes/cell, <10% mitochondrial reads), **88,923 cells × 13,140 genes** remain. Due to local memory constraints, a stratified subsample of **4,000 cells per sample (32,000 cells total)** is carried through normalization, HVG selection (2,000 genes), and clustering.

Source: Baker et al. (2021), GEO accession [GSE152495](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE152495).

## Analysis Workflow

The computational workflow is organized into five major stages:

```
Raw 10x Genomics data (8 samples)
        │
        ▼
01. Data loading & quality control
        │
        ▼
02. Normalization, HVG selection & scaling
        │
        ▼
03. PCA, neighborhood graph, UMAP & Leiden clustering
        │
        ▼
04. Canonical-marker cluster annotation
        │
        ▼
05. Cocaine vs. Sucrose differential expression & pathway enrichment
```

### Main analysis steps

**Data loading and QC**
- Load 10x Genomics matrices for all 8 samples; parse sex/treatment/replicate from folder names.
- Merge into a single AnnData object.
- Annotate mitochondrial (`mt:` prefix) and ribosomal (`RpS`/`RpL` prefix) genes.
- Filter cells with <200 genes detected, genes detected in <3 cells, and cells with >10% mitochondrial reads.

**Normalization and feature selection**
- Store raw counts in a dedicated layer.
- Normalize total counts to 10,000 per cell; log-transform.
- Identify 2,000 highly variable genes (batch-aware, `batch_key="sample"`).
- Regress out total counts and mitochondrial percentage; scale.

**Dimensionality reduction and clustering**
- PCA (30 components, `arpack` solver).
- 15-nearest-neighbor graph on the first 30 PCs.
- UMAP embedding.
- Leiden clustering at resolution 0.8 (matching Baker et al.'s reported parameter).
- Characterize clusters using canonical neurotransmitter/cell-type markers: `repo`, `elav`, `ey`, `Fas2`, `VAChT`, `Gad1`, `VGlut`, `ple`, `SerT`, `Tdc2`.

**Differential expression**
- Split by sex; compare Cocaine vs. Sucrose within each sex (Wilcoxon rank-sum test).
- Significance threshold: |log2FC| > 1.0, adjusted p < 0.05.

**Pathway enrichment**
- Select significant cocaine-responsive genes (top 150 by adjusted p-value, per sex).
- Query valid Enrichr library names for the fly organism at runtime (rather than hardcoding library versions).
- Analyze GO Biological Process and KEGG gene sets via GSEApy/FlyEnrichr.

## Quickstart Guide with uv

### 1. Prerequisites

Install:
- Git
- Python 3.11+
- uv

Check that they are available:

```bash
git --version
python --version
uv --version
```

If uv is not installed:

```bash
# Windows PowerShell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# Linux/macOS
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Restart the terminal after installation if necessary.

### 2. Clone the repository

```bash
git clone https://github.com/[YourUsername]/sc-project2-drosophila-brain.git
cd sc-project2-drosophila-brain
```

### 3. Create the uv environment

```bash
uv venv --python 3.11
```

Activate it:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# Linux/macOS
source .venv/bin/activate
```

### 4. Install project dependencies

The repository contains a pinned `requirements.txt`.

```bash
uv pip install -r requirements.txt
```

The analysis uses packages including:
- Scanpy
- AnnData
- NumPy / Pandas
- Matplotlib
- GSEApy
- Jupyter / ipykernel

### 5. Download the project data

Raw 10x Genomics matrices are too large to be stored directly in this GitHub repository.

**Project data (10x matrices):**
https://drive.google.com/drive/folders/1mmVogbMchQA73XL2I-hw2v9OieI1soM-?usp=drive_link

Download the required files and place them under `data/`. Each sample folder must contain `barcodes.tsv.gz`, `features.tsv.gz`, and `matrix.mtx.gz` so the notebook can automatically detect the sample:

```
data/
├── Female_Cocaine_1/
│   ├── matrix.mtx.gz
│   ├── barcodes.tsv.gz
│   └── features.tsv.gz
├── ...
└── Male_Sucrose_2/
    ├── matrix.mtx.gz
    ├── barcodes.tsv.gz
    └── features.tsv.gz
```

For later stages, the following processed checkpoint file(s) can be used to avoid repeating the most computationally expensive steps:

- `results/adata_final.h5ad` *(add this if you save a checkpoint — not currently in the repo tree above)*

### 6. Run the analysis

```bash
uv run jupyter lab
```

Then open `notebooks/notebook.ipynb` and run all cells top to bottom.

**Note:** the full QC-passed dataset (~89,000 cells) may exceed available local RAM during scaling/PCA. The current pipeline uses a stratified subsample (4,000 cells/sample) as a documented workaround — if running on a machine with more memory (or in Google Colab), this subsampling step can be removed to analyze the full dataset.

## Key Findings

**1. Cluster resolution falls short of the published atlas.** Leiden clustering at resolution 0.8 (the same resolution reported by Baker et al.) recovers 23 clusters here, versus 36 in the original ~86,000-cell study — most plausibly a consequence of the 32,000-cell subsample used to work within local memory limits.

**2. Canonical markers resolve major cell classes.** Glial (`repo`), GABAergic (`Gad1`), and multiple cholinergic (`VAChT`) clusters are clearly recovered; a putative Kenyon-cell cluster (`ey`+/`Fas2`+) is also identified. Dopaminergic, serotonergic, and octopaminergic markers (`ple`, `SerT`, `Tdc2`) co-occur in a single cluster rather than resolving separately, consistent with under-clustering.

**3. Sex-stratified DE shows a female-skewed, not male-skewed, response.** 70 significant cocaine-responsive genes are identified in males vs. 156 in females (24 shared) — the opposite direction from Baker et al.'s reported male bias. This is treated as an open discrepancy requiring further investigation, not a confirmed finding.

**4. Two gene groups drive the most extreme fold-changes and are flagged as likely artifacts.** `lncRNA:roX1`/`roX2` show large, population-wide shifts in both sexes with no known cocaine-response mechanism (possible batch effect); `Yp1`–`Yp3` show fold-changes consistent with low-detection-rate noise (possible ambient RNA) rather than genuine signal.

**5. Pathway enrichment diverges by sex.** Male cocaine-responsive genes enrich for oxidative phosphorylation and wounding-response pathways; female cocaine-responsive genes enrich for proteolysis/autophagy and cytoskeletal-remodeling pathways.

## Important Interpretation Note

The current pipeline analyzes a **stratified subsample (32,000 of 88,923 QC-passed cells)**, not the full dataset, due to local memory constraints. Cluster count, and potentially the direction of the sex-bias finding, should be interpreted with this limitation in mind until the pipeline is re-run on the full dataset (e.g., on a higher-memory machine or in Google Colab).

Additionally, the cluster-annotation marker panel in the current notebook excludes `elav` and `VGlut` due to a variable-scope bug (genes were checked against the HVG-restricted gene list rather than the full `adata.raw` gene list); cluster identities in Table 1 of the report should be treated as provisional until this is corrected.

## Repository Structure

```
sc-project2-drosophila-brain/
│
├── notebooks/
│   └── notebook.ipynb
│
├── results/
│   ├── figures/
│   │   ├── dotplot_canonical_markers.png
│   │   ├── female_pathway_enrichment.png
│   │   ├── female_volcano.png
│   │   ├── filter_genes_dispersion_HVG.png
│   │   ├── male_pathway_enrichment.png
│   │   ├── male_volcano.png
│   │   ├── pca_variance_ratio_PCA_variance.png
│   │   ├── umap_clusters_metadata.png
│   │   └── violin_QC_violin.png
│   │
│   └── tables/
│       ├── cluster_markers_wilcoxon.csv
│       ├── female_Cocaine_vs_Sucrose_all_DE.csv
│       ├── female_Cocaine_vs_Sucrose_significant_DE.csv
│       ├── female_enrichment_results.csv
│       ├── female_only_DE_genes.csv
│       ├── male_Cocaine_vs_Sucrose_all_DE.csv
│       ├── male_Cocaine_vs_Sucrose_significant_DE.csv
│       ├── male_enrichment_results.csv
│       ├── male_only_DE_genes.csv
│       ├── shared_DE_genes.csv
│       └── final_analysis_summary.csv
│
├── report/                     # add your report PDF/docx here
├── data/                       # not tracked — see Quickstart, Step 5
├── .gitignore
├── LICENSE
├── requirements.txt
└── README.md
```

## Reproducibility

The project records main software dependencies in `requirements.txt` with pinned versions, e.g.:

```
scanpy==1.11.x
anndata==0.11.x
pandas==2.2.x
gseapy==1.3.x
```

Fixed random seeds (`random_state=0`) are used for PCA, neighbor-graph construction, UMAP, and Leiden clustering to ensure reproducible cluster counts and embeddings across runs.

## Results and Figures

Generated figures are available under `results/figures/`:

| File | Contents |
|---|---|
| `violin_QC_violin.png` | QC metrics (genes/counts/%mt/%ribo) by condition |
| `filter_genes_dispersion_HVG.png` | Highly-variable-gene selection |
| `pca_variance_ratio_PCA_variance.png` | PCA variance ratio |
| `umap_clusters_metadata.png` | UMAP colored by cluster, sex, treatment, condition |
| `dotplot_canonical_markers.png` | Canonical marker dotplot across clusters |
| `male_volcano.png` / `female_volcano.png` | Cocaine vs. Sucrose DE, by sex |
| `male_pathway_enrichment.png` / `female_pathway_enrichment.png` | Pathway enrichment, by sex |

Corresponding result tables are available under `results/tables/`, including per-cluster markers, full and significant DE gene lists (all/male-only/female-only/shared), and enrichment results.

The complete project report is available at `report/Project2_Drosophila_Cocaine_Report.pdf`.

## Reference

The biological framework and experimental design for this project are based on:

Baker, B.M., Mokashi, S.S., Shankar, V., Hatfield, J.S., Hannah, R.C., Mackay, T.F.C., & Anholt, R.R.H. (2021). The Drosophila brain on cocaine at single-cell resolution. *Genome Research*, 31(10), 1927–1937. DOI: [10.1101/gr.268037.120](https://doi.org/10.1101/gr.268037.120)

## License

This project is distributed under the license included in the repository.
