# Morphological Fingerprints for Drug Sensitivity Prediction
Cell Painting profiles as a complementary drug representation for predicting cancer drug sensitivity

This repository contains the full analysis pipeline for a study investigating whether compound-level Cell Painting morphological profiles (JUMP-CP) improve quantitative drug sensitivity prediction when integrated with molecular structure representations, evaluated on the GDSC2 pharmacogenomic panel.

📄 Manuscript: in preparation

## Key Findings

- **Chemical and morphological representations are largely orthogonal.** Pairwise chemical similarity (Tanimoto) and morphological similarity (cosine) across 175 drugs show near-zero correlation (Spearman r = 0.031), indicating the two modalities capture independent information.
- **Morphology substantially outperforms molecular structure for drug sensitivity prediction:** Pearson R = 0.548 ± 0.155 (morphology-only) vs. 0.281 ± 0.167 (structure-only), evaluated across 25 random seeds (t = 6.50, p < 0.0001).
- **This advantage holds even against classical machine learning baselines.** Random Forest and Gradient Boosting trained on the same Morgan fingerprints (R = 0.414 ± 0.202 and R = 0.372 ± 0.224, respectively) outperform the structure-only MLP, but morphology-based models still significantly outperform the strongest of these classical baselines (morphology-only vs. Random Forest: p = 0.012).
- **Fusion architecture matters far less than input representation, but is not negligible.** Concatenation (R = 0.554 ± 0.130), cross-attention (R = 0.575 ± 0.116), and gated fusion (R = 0.492 ± 0.118) were evaluated across 25 random seeds. Cross-attention showed no statistically significant difference from concatenation (p = 0.111), but gated fusion performed significantly worse than concatenation (p = 0.005) — the choice of input representation (morphology vs. structure) still has far more impact on performance than the choice of fusion architecture.
- **SHAP attribution shows the model relies predominantly on morphological features** (86.9% of total attribution), with systematic variation across drug mechanism classes (81.3–92.2%), consistent with pathway-specific phenotypic signatures.

## Repository Structure

```
morphological-fingerprints-drug-sensitivity/
├── data/
│   ├── raw/                    # Not included — see Data Availability below
│   └── processed/              # Derived datasets (included, except large parquet)
├── notebooks/
│   ├── 00_overlap_check.ipynb           # Initial GDSC2–JUMP-CP overlap verification
│   ├── 01_data_exploration.ipynb        # Dataset loading, QC, fuzzy InChIKey matching
│   ├── 02_preprocessing.ipynb           # Feature engineering, Morgan fingerprints, QC
│   ├── 03_modality_analysis.ipynb       # Chemical vs. morphological complementarity analysis
│   ├── 04_baseline_models.ipynb         # Unimodal/concatenation baselines — original 5-seed pilot (superseded, see 08)
│   ├── 05_cross_attention_model.ipynb   # Cross-attention and gated fusion — original 5-seed pilot (superseded, see 08)
│   ├── 06_explainability.ipynb          # SHAP attribution analysis
│   ├── 07_figures_paper.ipynb           # Publication-ready figure generation
│   └── 08_additional_baselines.ipynb    # Classical ML baselines (Random Forest, Gradient Boosting) + 25-seed evaluation of all five model conditions — canonical source for all results reported in the manuscript
├── results/
│   ├── figures/                # All generated figures (exploratory + publication-ready)
│   ├── *.json                  # Model performance results (all conditions, all seeds — see multiseed_25_results.json and rf_gbm_25seed_results.json for the results used in the manuscript)
│   ├── *.csv                   # SHAP attribution tables
│   └── *.pt                    # Trained model checkpoints
├── requirements.txt
├── .gitignore
└── README.md
```

## Dataset Summary

| Parameter | Value |
|---|---|
| Matched drugs (GDSC2 ∩ JUMP-CP) | 175 / 229 with usable IDs (76.4%) |
| Cancer cell lines | 969 |
| LN_IC50 measurements (unique pairs) | 149,679 |
| Sensitivity matrix sparsity | 11.7% |
| Morphological features (post-QC) | 3,178 |
| Morgan fingerprint dimensions | 2,048 |

Compound matching used a fuzzy InChIKey strategy (first 14 characters — the molecular connectivity layer), recovering 77 additional compounds beyond exact matching that would otherwise have been excluded due to salt-form and stereochemistry annotation differences between GDSC2 and JUMP-CP.

## Model Results

| Model | Pearson R | RMSE | Seeds |
|---|---|---|---|
| Structure-only MLP | 0.281 ± 0.167 | 2.428 ± 0.266 | 25 |
| Morphology-only MLP | 0.548 ± 0.155 | 2.027 ± 0.238 | 25 |
| Concatenation MLP | 0.554 ± 0.130 | 2.017 ± 0.221 | 25 |
| Cross-attention fusion | 0.575 ± 0.116 | 1.975 ± 0.226 | 25 |
| Gated fusion | 0.492 ± 0.118 | 2.113 ± 0.252 | 25 |
| Random Forest (Morgan fp) | 0.414 ± 0.202 | 1.781 ± 0.315 | 25 |
| Gradient Boosting (Morgan fp) | 0.372 ± 0.224 | 1.832 ± 0.289 | 25 |

All models evaluated on held-out test drugs using a drug-level train/val/test split (70/15/15) to prevent information leakage and assess generalization to unseen compounds. Random Forest and Gradient Boosting are classical machine learning baselines trained on Morgan fingerprints alone, evaluated as a robustness check rather than as part of the core five-condition MLP ablation.

## Setup

```bash
# Create and activate environment (Python 3.10)
conda create -n proj python=3.10 -y
conda activate proj

# Install dependencies
pip install -r requirements.txt
```

Notebooks are designed to be run sequentially (00 → 08), with each notebook reading processed outputs saved by the previous one.

## Data Availability

Raw data files are not included in this repository due to size constraints:

- **GDSC2 drug response data:** available from the Genomics of Drug Sensitivity in Cancer database (release 8.4)
- **JUMP-CP morphological profiles:** available from the JUMP Cell Painting Consortium (cpg0016), hosted on the Cell Painting Gallery (AWS S3, public access)

The processed compound-level morphological profile matrix (`jump_profiles_matched.parquet`) is also excluded for size reasons but can be regenerated by running Notebook 01 against the raw JUMP-CP source.

## Citation

If you use this code or pipeline, please cite the associated manuscript (citation to be added upon publication).

## License

This project is licensed under the MIT License — see LICENSE for details.

## Contact

Amir J. Shokrzadeh — amirrshokrzadeh@gmail.com
ORCID: 0009-0009-2766-7992
