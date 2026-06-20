# Morphological Fingerprints for Drug Sensitivity Prediction

**Cell Painting profiles as a complementary drug representation for predicting cancer drug sensitivity**

This repository contains the full analysis pipeline for a study investigating whether compound-level Cell Painting morphological profiles (JUMP-CP) improve quantitative drug sensitivity prediction when integrated with molecular structure representations, evaluated on the GDSC2 pharmacogenomic panel.

📄 Manuscript: *in preparation for submission to the Journal of Chemical Information and Modeling*

---

## Key Findings

- **Chemical and morphological representations are largely orthogonal.** Pairwise chemical similarity (Tanimoto) and morphological similarity (cosine) across 175 drugs show near-zero correlation (Spearman r = 0.031), indicating the two modalities capture independent information.
- **Morphology substantially outperforms molecular structure** for drug sensitivity prediction: Pearson R = 0.511 ± 0.059 (morphology-only) vs. 0.232 ± 0.065 (structure-only), evaluated across 5 random seeds.
- **Simple feature concatenation outperforms learned fusion mechanisms** under low-data conditions: concatenation (R = 0.549 ± 0.066) outperformed both cross-attention (R = 0.597, single seed) and gated fusion (R = 0.585, single seed) — neither complex architecture improved over the multi-seed concatenation baseline.
- **SHAP attribution shows the model relies predominantly on morphological features** (86.9% of total attribution), with systematic variation across drug mechanism classes (81.3–92.2%), consistent with pathway-specific phenotypic signatures.

---

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
│   ├── 04_baseline_models.ipynb         # Unimodal and concatenation baselines, multi-seed eval
│   ├── 05_cross_attention_model.ipynb   # Cross-attention and gated fusion architectures
│   ├── 06_explainability.ipynb          # SHAP attribution analysis
│   └── 07_figures_paper.ipynb           # Publication-ready figure generation
├── results/
│   ├── figures/                # All generated figures (exploratory + publication-ready)
│   ├── *.json                  # Model performance results (all conditions, all seeds)
│   ├── *.csv                   # SHAP attribution tables
│   └── *.pt                    # Trained model checkpoints
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Dataset Summary

| Parameter | Value |
|---|---|
| Matched drugs (GDSC2 ∩ JUMP-CP) | 175 / 286 (76.4%) |
| Cancer cell lines | 969 |
| LN_IC50 measurements | 153,882 |
| Sensitivity matrix sparsity | 11.7% |
| Morphological features (post-QC) | 3,178 |
| Morgan fingerprint dimensions | 2,048 |

Compound matching used a **fuzzy InChIKey strategy** (first 14 characters — the molecular connectivity layer), recovering 77 additional compounds beyond exact matching that would otherwise have been excluded due to salt-form and stereochemistry annotation differences between GDSC2 and JUMP-CP.

---

## Model Results

| Model | Pearson R | RMSE | Seeds |
|---|---|---|---|
| Structure-only MLP | 0.232 ± 0.065 | 2.479 ± 0.251 | 5 |
| Morphology-only MLP | 0.511 ± 0.059 | 2.106 ± 0.230 | 5 |
| **Concatenation MLP** | **0.549 ± 0.066** | **2.023 ± 0.218** | 5 |
| Cross-attention fusion | 0.597 | 2.066 | 1* |
| Gated fusion | 0.585 | 2.056 | 1* |

*Single-seed evaluation due to computational constraints — interpret with caution.

All models evaluated on held-out test drugs using a **drug-level train/val/test split** (70/15/15) to prevent information leakage and assess generalization to unseen compounds.

---

## Setup

```bash
# Create and activate environment (Python 3.10)
conda create -n proj python=3.10 -y
conda activate proj

# Install dependencies
pip install -r requirements.txt
```

Notebooks are designed to be run sequentially (00 → 07), with each notebook reading processed outputs saved by the previous one.

---

## Data Availability

Raw data files are not included in this repository due to size constraints:

- **GDSC2 drug response data**: available from the [Genomics of Drug Sensitivity in Cancer](https://www.cancerrxgene.org/) database (release 8.4)
- **JUMP-CP morphological profiles**: available from the [JUMP Cell Painting Consortium](https://github.com/jump-cellpainting/datasets) (cpg0016), hosted on the [Cell Painting Gallery](https://registry.opendata.aws/cellpainting-gallery/) (AWS S3, public access)

The processed compound-level morphological profile matrix (`jump_profiles_matched.parquet`, 357 MB) is also excluded for size reasons but can be regenerated by running Notebook 01 against the raw JUMP-CP source.

---

## Citation

If you use this code or pipeline, please cite the associated manuscript (citation to be added upon publication).

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## Contact

Amir J. Shokrzadeh — amirrshokrzadeh@gmail.com
ORCID: [0009-0009-2766-7992](https://orcid.org/0009-0009-2766-7992)