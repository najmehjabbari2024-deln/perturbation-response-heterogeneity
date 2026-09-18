# How Consistently Do Cells Respond to the Same Genetic Perturbation?

A small computational biology project using Perturb-seq data to explore how differently cells respond to the same genetic perturbation.

## Research question

If the same genetic perturbation is applied to many cells, do they respond in a similar way or differently?

I also wanted to see whether differences in the baseline state of cells could help explain this variation.

This is an exploratory project. The goal is to understand the data and response patterns, not to build the best possible prediction model.

## Dataset

The dataset is a K562 Perturb-seq dataset from the scPerturb collection.

* **File:** `AdamsonWeissman2016_GSM2406675_10X001.h5ad`
* **Source:** [scPerturb](https://projects.sanderlab.org/scperturb/)
* **Zenodo:** https://zenodo.org/records/10044268
* **Cells:** 5768
* **Genes:** 35635

The experiment uses CRISPRi perturbations targeting several genes in K562 cells.

One group, `62(mod)_pBA581`, is used as the control group in this project. Its identity could not be confirmed directly from a primary source, so this is treated as a working assumption.

The original data file is not included in this repository. See `data/README.md` for the download information.

## What I did

The analysis is divided into eight notebooks:

1. **`01_data_quality.ipynb`**
   Checked the structure of the dataset, metadata, perturbation labels, and expression matrix.

2. **`02_preprocessing.ipynb`**
   Removed cells without a usable perturbation label, filtered cells and genes, normalized the data, and applied `log1p`.

3. **`03_cellular_states.ipynb`**
   Used PCA to explore variation between cells and check the main sources of variation.

4. **`04_perturbation_response.ipynb`**
   Checked whether the perturbations affected their target genes and defined a per-cell response measure.

5. **`05_response_heterogeneity.ipynb`**
   Compared the size of the perturbation response with the variation between individual cells.

6. **`06_baseline_state_analysis.ipynb`**
   Tested whether technical features and baseline cellular state could explain response heterogeneity.

7. **`07_simple_prediction.ipynb`**
   Used a simple Ridge regression model to test whether the relationship could also predict response heterogeneity on unseen cells.

8. **`08_final_analysis.ipynb`**
   Repeated the main analyses and collected the final results, figures, and tables.

## Main findings

* The seven tested perturbations showed evidence of target-gene knockdown.
* For all perturbations, variation between individual cells was large compared with the overall response magnitude.
* Baseline features were useful for explaining variation in the response. A Ridge model reached a test-set R² of about 0.79 using sequencing depth, ribosomal fraction, and the first 10 PCA components.
* The results suggest that cells receiving the same perturbation can still show substantially different transcriptional responses.

These findings are specific to this dataset and analysis method.

## Results

The main figures are in:

`results/figures/`

The final tables are in:

`results/tables/`

The final analysis is also collected in `08_final_analysis.ipynb`.

## Limitations

* The control group is based on a working assumption rather than a directly confirmed primary-source annotation.
* This is a single pilot dataset with seven perturbations.
* The response was measured using a specific distance-based representation of gene expression.
* The analysis cannot separate biological differences in baseline state from all possible technical effects.
* The results may not generalize to other datasets, cell lines, or perturbations.

## Reproducibility

Python and the required packages are listed in `requirements.txt`.

To run the project:

1. Download the `.h5ad` dataset.
2. Put it in the `data/` folder.
3. Install the required packages.
4. Run the notebooks in numerical order.

Randomized steps use a fixed random seed.

