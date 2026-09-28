# HASEM — Pan-Cancer Detection via Healthy-Baseline Anomaly Modelling

**H**ybrid **A**nomaly–**S**upervised **E**xplainable **M**odel for pan-cancer transcriptomic screening.

HASEM detects cancer from RNA-seq expression profiles without ever training on tumour data. A principal component model is fitted exclusively to healthy tissue; every sample is then scored by how far it deviates from that healthy reference. No labelled tumour examples are required at training time, which means the approach is not limited to cancer types seen during development.

This repository contains the full analysis pipeline, results, and trained models behind the manuscript *"Learning the molecular boundary of health: HASEM, a hybrid anomaly–supervised explainable framework for pan-cancer transcriptomic outlier detection"* (submitted to *Briefings in Bioinformatics*).

## Key results

| Metric | Result |
|---|---|
| Primary detector | Q-residual (reconstruction error), not the originally proposed T²+Q combination |
| Discrimination (Q-residual alone) | AUROC 0.998, 97.86% detection, 1.01% false-alarm rate |
| Dataset | 19,109 RNA-seq profiles: 8,149 healthy (GTEx + TCGA adjacent-normal), 10,960 tumour, 32 tissue types |
| Generalisation test (LOCO-C) | Detection ≥94.97% for 24/25 tissues, even with matching healthy reference tissue fully excluded |
| Batch-effect check | Genuine technical signal found (GTEx vs TCGA-normal, AUROC 1.0000) and reported directly, alongside evidence it does not drive detection |

The manuscript describes the full methodology, ablation, and validation.

## Repository structure

```
├── notebooks/
│   └── HASEM_pipeline.ipynb        # Full pipeline: preprocessing → PCA → anomaly
│                                    # scoring → supervised confirmation → enrichment
├── models/
│   ├── pca_healthy.joblib          # Fitted healthy-reference PCA model
│   ├── scaler.joblib               # Healthy-fitted feature scaler
│   ├── logistic_regression.joblib
│   ├── random_forest.joblib
│   └── xgboost.joblib
├── results/
│   ├── unsupervised/
│   │   ├── unsup_metrics.csv               # Table 3: full statistic ablation
│   │   ├── celline_sensitivity_comparison.csv
│   │   └── ablation_T2.csv / ablation_Q.csv / ablation_AND.csv
│   ├── per_cancer/
│   │   └── per_cancer_empirical.csv        # Table 4: per-tissue detection, all 32 types
│   ├── loco/
│   │   ├── LOCO_A_results.csv              # Global-model per-cancer (Table S5)
│   │   ├── LOCO_B_results.csv              # Retrained-PCA per-cancer (Table S5)
│   │   └── LOCO_C_tissue_excluded.csv      # Table 5: matching-tissue-excluded LOCO
│   ├── supervised/
│   │   └── supervised_performance.csv      # Table 6: AUROC, AUPRC, Brier, etc.
│   ├── batch_effects/
│   │   ├── batch_effect_summary.csv
│   │   ├── true_batch_negative_control.csv # GTEx vs TCGA-normal, within healthy class
│   │   └── false_alarm_by_source.csv
│   ├── pathway_enrichment/
│   │   └── gprofiler_global_top200_FULL.csv # Table 8: 266 significant terms
│   └── contributions/
│       └── gene_contributions_global.csv    # Full ranked gene list (5,000 genes)
├── figures/
│   ├── T2_Q_scatter.png
│   ├── per_cancer_detection_rates.png
│   ├── roc_pr_curves.png
│   ├── calibration_curves.png
│   ├── LOCO_comparison.png
│   └── pca_biplot_cohort_vs_tumour.png
├── requirements.txt
└── README.md
```

## Data sources

Expression data (expected read counts per gene) were obtained from the [UCSC Xena platform](https://xenabrowser.net/), which distributes uniformly reprocessed Genomic Data Commons holdings via the [Toil recompute workflow](https://doi.org/10.1038/nbt.3772), covering:

- **GTEx** (Genotype-Tissue Expression) — healthy reference tissue
- **TCGA** (The Cancer Genome Atlas) — adult tumour samples, plus adjacent-normal samples included in the healthy reference
- **TARGET** — paediatric tumour samples

Raw data are not redistributed in this repository; see the notebook for exact dataset identifiers and download instructions.

## Reproducing the pipeline

1. Clone this repository and install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Open `notebooks/HASEM_pipeline.ipynb` and run top to bottom. A fixed random seed (42) is used throughout for reproducibility.
3. Outputs are written to `results/` in the same structure as committed here.

The `models/` directory contains the fitted objects from the run used in the manuscript. They are provided for convenience — everything in `models/` is fully regenerable by rerunning the notebook, and nothing downstream depends on trusting the committed pickle files over a fresh run.

## Method summary

1. **Healthy-baseline PCA.** Features are centred, scaled, and a PCA model is fitted using healthy samples only (95% cumulative explained variance retained).
2. **Anomaly scoring.** Every sample — healthy or tumour — is projected into this space and scored by Q-residual (reconstruction error against the healthy model). Hotelling T² is computed but, per the ablation in the manuscript, does not improve on Q-residual alone.
3. **Supervised confirmation.** Logistic regression, random forest, and XGBoost classifiers convert anomaly scores into calibrated probabilities.
4. **Validation.** Per-cancer detection, three leave-one-cancer-out variants of increasing stringency, a cell-line sensitivity check, and a provenance/batch-effect analysis are all reported — including a genuine batch signal the study does not correct away, only interrogates directly.
5. **Interpretability.** Gene-level contributions to the Q-residual are ranked and submitted to pathway over-representation analysis (g:Profiler).

## Citation

If you use this code or these results, please cite:

> [Author names]. Learning the molecular boundary of health: HASEM, a hybrid anomaly–supervised explainable framework for pan-cancer transcriptomic outlier detection. *Briefings in Bioinformatics* (submitted).

A DOI for this repository, via Zenodo, will be added here once minted.

## License

This project is licensed under the Apache License, Version 2.0. See the LICENSE file, or view the license text at apache.org/licenses/LICENSE-2.0.

## Acknowledgements

This work uses data from the GTEx, TCGA, and TARGET consortia and their participants, distributed via the UCSC Xena platform. We used an AI language assistant (Claude, Anthropic) during code development to identify and correct errors in the analysis pipeline; we did not use it to draft or generate analysis, results, or interpretation. See the manuscript's Methods section for the full disclosure.
