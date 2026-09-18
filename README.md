# How Consistently Do Cells Respond to the Same Genetic Perturbation?

Exploring cellular heterogeneity and the predictability of transcriptional responses using Perturb-seq.

## Research question

How heterogeneous are single-cell transcriptional responses to the same genetic perturbation, and can differences in baseline cellular state explain part of this response heterogeneity?

In simpler terms: if the same genetic perturbation is applied to many cells, do those cells respond similarly to each other, or substantially differently? And if they differ, does anything measurable about a cell's state before the perturbation's effect is considered help explain why?

This is an exploratory computational biology project. The goal is not to build the most accurate perturbation-response predictor, but to quantify heterogeneity itself and check whether baseline cellular state carries information about it.

## Motivation

Most analyses of Perturb-seq data summarize a perturbation's effect at the group level (for example, differential expression between perturbed and control cells). That summary can hide a real, separate question: do the individual cells behind that summary agree with each other, or does the group-level effect emerge from a wide spread of very different individual responses? Distinguishing those two cases matters for how much a group-level effect size should be trusted to describe any single cell, and is the motivation for this project.

## Dataset

- **Source**: [scPerturb](https://projects.sanderlab.org/scperturb/) harmonized single-cell perturbation dataset collection (Peidli et al. 2022, *Nature Methods*), hosted on Zenodo.
- **File**: `AdamsonWeissman2016_GSM2406675_10X001.h5ad` (~34.6 MB)
- **Download**: `https://zenodo.org/record/10044268/files/AdamsonWeissman2016_GSM2406675_10X001.h5ad?download=1`
- **Underlying experiment**: a CRISPRi Perturb-seq pilot in K562 cells targeting 7 non-essential transcription factors and cell-cycle regulators, plus a non-targeting control. This specific pilot (Figure 6) is described in Dixit et al. 2016 (*Cell*), a companion paper sharing authors and the same guide-barcode vector system as Adamson et al. 2016 (*Cell*) — the paper this dataset is filed under in scPerturb. Both citations are relevant; Dixit et al. 2016 may be the more precise one for this specific pilot experiment.
- **Scale**: 5768 cells x 35635 genes as downloaded; 5752 cells x 14690 genes after the filtering done in `02_preprocessing.ipynb`.

Place the downloaded file at `data/AdamsonWeissman2016_GSM2406675_10X001.h5ad` before running the notebooks. The `data/` directory is not tracked in this repository; every notebook that writes an intermediate file writes it there.

### An assumption this project depends on

The dataset's `perturbation` column has 10 categories: 7 that name a target gene, one coded `62(mod)_pBA581`, and a small number of missing or unclear labels (excluded in preprocessing). `62(mod)_pBA581` is treated throughout this project as the negative control group. This was never confirmed by a single primary-source sentence naming it directly, but is supported by four independent pieces of evidence gathered across the notebooks:

1. It matches no human gene symbol.
2. It is the largest group (1769 of 5752 cells), consistent with typical control-group sizing.
3. Its plasmid number (`pBA581`) matches the numbering convention of the actual Adamson/Weissman Perturb-seq guide-barcode vector library (Addgene lists it as `pBA571`, from the same paper).
4. All 7 named-gene perturbations show reduced expression of their own target gene relative to this group specifically (see Findings below), which is what CRISPRi knockdown relative to a true control should look like.

Treat this as a well-supported working assumption, not an established fact, in any downstream use of this repository.

## Methods

Eight notebooks, run in order, each building on the last:

1. **`01_data_quality.ipynb`** — inspects the real file directly (dimensions, metadata columns, perturbation labels, matrix sparsity, raw-vs-processed state) rather than assuming its structure.
2. **`02_preprocessing.ipynb`** — removes cells with no usable perturbation label; filters low-detection cells and genes; normalizes each cell to a total of 10,000 counts; applies `log1p`; flags highly variable genes by dispersion, restricted to genes detected in at least 1% of cells (a floor added after an earlier version's dispersion ranking turned out to be dominated by genes sitting right at the minimum-detection threshold).
3. **`03_cellular_states.ipynb`** — PCA on the highly variable genes, with an explicit check for whether the leading components are just tracking sequencing depth (they are not, in this data).
4. **`04_perturbation_response.ipynb`** — validates on-target CRISPRi knockdown for each perturbation, then defines a cell's response as its Euclidean distance from the control group's centroid in highly-variable-gene expression space (not PCA space, which captured too little variance to be trustworthy for this).
5. **`05_response_heterogeneity.ipynb`** — splits response into *magnitude* (systematic shift of a perturbation group's centroid from control) and *heterogeneity* (spread of individual cells around their own group's centroid), using an exact variance-decomposition identity that ties the two back to Notebook 4's per-cell distances.
6. **`06_baseline_state_analysis.ipynb`** — regresses per-cell heterogeneity on baseline technical features (sequencing depth, ribosomal fraction) and baseline PCA coordinates, with bootstrap confidence intervals, a permutation test, and a dedicated check that the result isn't being driven by a handful of extreme low-count outlier cells.
7. **`07_simple_prediction.ipynb`** — the same relationship, evaluated properly: Ridge regression with a stratified train/test split, cross-validated regularization strength (training data only), and comparison against a naive baseline.
8. **`08_final_analysis.ipynb`** — recomputes the above into final figures and tables; no new methodology.

All random seeds are fixed (`random_state=0` throughout). No GPU, no deep learning, no foundation models — the entire pipeline runs comfortably on an ordinary laptop.

## Main findings

- **On-target knockdown was real and often strong** for all 7 perturbations (expression of each guide's own target gene, relative to control, ranged from a complete knockdown for `SNAI1` to a partial one for `EP300`).
- **Response magnitude was small and heterogeneity was large, for every perturbation, including relative to control's own baseline spread.** Cells receiving the same perturbation disagreed with each other about as much as unperturbed control cells naturally disagree among themselves. Strong, confirmed knockdown of a single target gene did not translate into a detectably consistent shift across the broader transcriptome, at least by the distance-from-centroid measure used here.
- **Baseline cellular state explains a substantial share of that heterogeneity, and this held up on held-out data.** A cell's sequencing depth, ribosomal fraction, and position along the first 10 principal components together predicted its distance from its own perturbation group's centroid with a test-set R^2 of about 0.79 — evaluated on cells the model never saw during fitting, and checked against the possibility that a small number of extreme low-count cells were driving the result (they were not).

## Limitations

- The control-group identity (`62(mod)_pBA581`) is a well-supported assumption, not a confirmed fact (see above).
- The more precise citation for this specific pilot experiment may be Dixit et al. 2016 rather than Adamson et al. 2016.
- The strongest individual predictor of heterogeneity, PC1, could reflect genuine baseline biological state or the well-documented dissociation-induced stress response common across scRNA-seq sample preparation (PC1's top gene loadings were dominated by stress/UPR-pathway genes). This project cannot distinguish the two.
- Two metadata columns, `nperts` and `percent_mito`, could not be explained (a constant value of 2 for every successfully labeled cell, and a constant 0 respectively) and were not used anywhere in the analysis.
- This is one pilot experiment with 7 perturbations and about 5750 cells. Nothing here establishes that these findings generalize to a larger Perturb-seq screen, a different cell line, or different perturbations.
- The heterogeneity metric used (spread of Euclidean distances around a group centroid) cannot distinguish a population of cells all responding by a similarly modest, continuous amount from a bimodal population where some cells respond strongly and others not at all. Both would show similarly high heterogeneity by this measure.
- "Response" is one specific, deliberately chosen representation (distance from a centroid in highly-variable-gene expression space), used consistently rather than alongside competing alternatives. A differential-expression-based definition, or a different gene set, could in principle describe the data differently.

## Reproducibility

- Python 3.11, packages listed in `requirements.txt`.
- Download the dataset (see above) into `data/`.
- Run the notebooks in numeric order; each writes its output into `data/` for the next notebook to read.
- All randomized steps (train/test splits, PCA, bootstrap, permutation tests) use a fixed seed.

## Future directions

Not part of this version, but natural extensions: additional Perturb-seq datasets or cell lines; perturbation combinations; resolving the PC1 biology-vs-technical-artifact question directly (for example with cell-cycle scoring or a dissociation-stress gene signature); a response representation based on differential expression rather than centroid distance, as a comparison; pathway-level interpretation of the genes driving heterogeneity; cross-dataset validation of the baseline-state relationship found in Notebook 6 and 7; and uncertainty-aware or active-learning approaches to deciding which cells or perturbations would be most informative to sequence next.
