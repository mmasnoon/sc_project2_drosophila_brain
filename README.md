The Drosophila Brain on Cocaine — Single-Cell RNA-Seq Reanalysis
Project 2 reanalysis of Baker et al. (2021, Genome Research), "The Drosophila brain on cocaine at single-cell resolution" (GEO: GSE152495), using a Scanpy/Leiden pipeline.
Overview
Brains from adult Drosophila melanogaster were analyzed using the 10x Genomics single-cell RNA sequencing after the administration of sucrose (control condition) or sucrose with cocaine. This repository performs an analysis similar to the original study's QC, normalization, clustering, marker annotation, differential expression, and pathway enrichment on the same data set and compares findings with the original paper's atlas of 36 clusters.
Status: Draft (Milestone 1) — see Report for current findings and known limitations.
Repository Structure
.
├── data/                          # 10x Genomics count matrices (not tracked in git — see Data below)
│   ├── Female_Cocaine_1/
│   │   ├── barcodes.tsv.gz
│   │   ├── features.tsv.gz
│   │   └── matrix.mtx.gz
│   ├── Female_Cocaine_2/
│   ├── Female_Sucrose_1/
│   ├── Female_Sucrose_2/
│   ├── Male_Cocaine_1/
│   ├── Male_Cocaine_2/
│   ├── Male_Sucrose_1/
│   └── Male_Sucrose_2/
├── notebooks/
│   └── project2_analysis.ipynb    # Full pipeline, Steps 8.1–8.7
├── results/
│   ├── tables/                    # DE gene lists, cluster markers, enrichment results (CSV)
│   └── adata_final.h5ad           # Final annotated AnnData checkpoint
├── figures/                       # All figures exported from the notebook
├── report/
│   ├── Project2_Drosophila_Cocaine_Report.docx
│   └── Project2_Drosophila_Cocaine_Report.pdf
├── paper.pdf                      # Primary reference (Baker et al. 2021)
├── project2_guide.pdf             # Course project guide
├── requirements.txt               # Pinned dependencies (uv pip freeze)
└── README.md

Environment Setup
This project uses uv for dependency management.
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create and activate the environment
uv venv --python 3.11
source .venv/bin/activate          # Windows: .venv\Scripts\Activate.ps1

# Install dependencies
uv pip install -r requirements.txt

# Register the Jupyter kernel and launch
python -m ipykernel install --user --name sc-project2
jupyter lab

To reproduce the exact pinned environment from scratch:
uv pip freeze > requirements.txt

Data
Raw 10x Genomics matrices are not tracked in this repository (large binary files). Place the 8 sample folders under data/ exactly as shown in Repository Structure before running the notebook — each folder must contain barcodes.tsv.gz, features.tsv.gz, and matrix.mtx.gz. Sample metadata (sex, treatment, replicate) is parsed automatically from folder names (e.g., Female_Cocaine_1).
Source data: GEO accession GSE152495.
Running the Analysis
Open notebooks/project2_analysis.ipynb in JupyterLab and run all cells top to bottom. The notebook is organized to match the project guide's Step 8.1–8.7 walkthrough:
Step
Section
8.1
Load 10x data, build unified AnnData
8.2
QC & filtering
8.3
Normalization & scaling
8.4
PCA, UMAP, Leiden clustering
8.5
Marker-based cluster annotation
8.6
Cocaine vs. Sucrose differential expression, by sex
8.7
Pathway enrichment (GSEApy)

Checkpoints are written to results/ after each major stage; figures are written to figures/.
⚠️ Memory note: the full QC-passed dataset (~89,000 cells) may exceed available memory during scaling/PCA on a standard laptop. The current pipeline uses a stratified subsample (4,000 cells/sample, 32,000 total) as a documented workaround — see the Report's Limitations section.
Report & Deliverables
The full write-up — introduction, methods, results (Figures 1–5), discussion, limitations, and the required AI Usage Disclosure appendix — is in report/Project2_Drosophila_Cocaine_Report.pdf.
Milestone
Deadline
Tag
Draft Report & Pipeline
Aug 31, 2026
v1.0-peerreview
Peer Review
Sep 4, 2026
—
Final Submission
Sep 9, 2026
v2.0-final

Key Findings (Draft)
Clustering: 23 Leiden clusters recovered at resolution 0.8 (vs. 36 in the original study), likely reflecting the reduced cell count analyzed under the memory-constrained subsample.
Differential expression: 70 significant cocaine-responsive genes in males, 156 in females (Wilcoxon, |log2FC|>1, adj. p<0.05) — a female-skewed result that contrasts with the original study's reported male bias and is treated as an open question, not a confirmed replication.
Data-quality caveats: extreme fold-changes in roX1/roX2 and Yp1–3 are flagged as likely artifactual (batch effect and low-detection-rate noise, respectively) — see Report Section 3.5 and 5.
Citation
Baker, B.M., Mokashi, S.S., Shankar, V., Hatfield, J.S., Hannah, R.C., Mackay, T.F.C., & Anholt, R.R.H. (2021). The Drosophila brain on cocaine at single-cell resolution. Genome Research, 31(10), 1927–1937. https://doi.org/10.1101/gr.268037.120
AI Usage Disclosure
Generative AI (Claude, Anthropic) was used as a learning and debugging aid during pipeline development and report drafting, per the course's Academic Integrity policy. Full prompt log and validation notes are in the Report's Appendix.

# sc_project2_drosophila_brain
