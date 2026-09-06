# Morphological Fingerprints for Drug Sensitivity Prediction

Cell Painting profiles as a complementary drug representation for predicting cancer drug sensitivity.

This repository holds the analysis pipeline for a study asking whether compound-level Cell Painting morphological profiles (JUMP-CP) carry drug-sensitivity information that molecular structure does not, tested against the GDSC2 pharmacogenomic panel.

Manuscript in preparation.

## Key findings

**Chemical and morphological representations capture largely separate information.** Across 175 drugs, pairwise chemical similarity (Tanimoto) and morphological similarity (cosine) correlate at Spearman r = 0.031. That is statistically detectable under permutation testing (p = 0.006) but negligible in practice: structure explains under 0.1% of the variance in morphology.

**Morphology beats molecular structure by a wide margin.** Pearson R = 0.701 ± 0.176 for morphology alone against 0.351 ± 0.221 for structure alone, across 25 random seeds with drug-level evaluation (paired t = 6.25, p < 0.0001).

**The advantage is not an artifact of the neural architecture.** It reproduces across four independent model families: MLP, Random Forest, Gradient Boosting, and Ridge regression. Every one of them shows the same ordering.

**Model complexity bought nothing.** Random Forest on concatenated features scored R = 0.776 ± 0.101, the best result of any model or representation tested here, and it significantly beat the best neural model (cross-attention, R = 0.728 ± 0.128; paired t = 4.40, p < 0.001). Tuned linear regression tied the best neural model rather than losing to it (RidgeCV R = 0.730 ± 0.145; p = 0.88). Among the fusion strategies themselves, cross-attention showed no reliable gain over plain concatenation (p = 0.493), and gated fusion did measurably worse (p = 0.003). With 123 training drugs, what you feed the model matters more than how clever the model is.

**SHAP attribution is dominated by morphology, uniformly.** Exact TreeExplainer values on the best model, pooled over all 25 seeds, put morphological features at 95.0 ± 1.4% of total attribution. That reliance barely moves across drug mechanism classes (92.3% to 98.2%). Feature-level analysis explains why: a small shared set of features, mostly nuclear RNA content and DNA granularity, accounts for roughly 9 of the top 10 features in every pathway examined. A size-matched permutation test still found genuine pathway-specific signal for two of four pathways tested (mitosis, p = 0.015; protein stability and degradation, p = 0.019) sitting on top of that shared signature.

## Repository structure

```
morphological-fingerprints-drug-sensitivity/
├── data/
│   ├── raw/                    # Not included, see Data availability
│   └── processed/              # Derived datasets (large parquet excluded)
├── notebooks/
│   ├── 00_overlap_check.ipynb        # GDSC2–JUMP-CP compound matching via InChIKey
│   ├── 01_data_exploration.ipynb     # Dataset QC, IC50 matrix, profile subsetting
│   ├── 02_preprocessing.ipynb        # Consensus profiles, Morgan fingerprints, feature QC
│   ├── 03_modality_analysis.ipynb    # PCA, pairwise similarity, permutation testing
│   ├── 04_baseline_models.ipynb      # Early single-seed MLP baselines
│   ├── 05_cross_attention_model.ipynb # Fusion architecture development
│   ├── 06_explainability.ipynb       # Early SHAP exploration
│   ├── 07_figures_paper.ipynb        # Manuscript figures
│   └── 08_final_evaluation.ipynb     # 25-seed evaluation, classical baselines,
│                                     #   exact SHAP, permutation-tested similarity
├── results/
│   ├── figures/                # Exploratory and publication figures
│   ├── predictions/            # Per-drug predictions, every seed and condition
│   └── *.json                  # Model performance and statistical test outputs
├── requirements.txt
└── README.md
```

Notebook 08 is the canonical source for everything reported in the manuscript. Notebooks 04 through 06 hold earlier development work; where their outputs disagree with notebook 08, notebook 08 wins.

## Dataset summary

| Parameter | Value |
|---|---|
| Matched drugs (GDSC2 ∩ JUMP-CP) | 175 / 229 with usable IDs (76.4%) |
| Cancer cell lines | 969 |
| LN_IC50 measurements (unique pairs) | 149,679 |
| Sensitivity matrix sparsity | 11.7% |
| Morphological features (post-QC) | 3,178 |
| Morgan fingerprint dimensions | 2,048 |
| Train / validation / test split | 123 / 26 / 26 drugs |

Compound matching used the first 14 characters of the InChIKey, the molecular connectivity layer. That recovered 77 compounds beyond exact matching, which would otherwise have been dropped over salt-form and stereochemistry annotation differences between the two databases. No drug in the matched set had more than one candidate JUMP-CP compound at that prefix, so no distinct stereoisomers were merged.

## Model results

Five MLP conditions, 25 seeds each, scored at the drug level:

| Model | Pearson R | RMSE |
|---|---|---|
| Structure-only MLP | 0.351 ± 0.221 | 1.962 ± 0.323 |
| Morphology-only MLP | 0.701 ± 0.176 | 1.402 ± 0.360 |
| Concatenation MLP | 0.720 ± 0.135 | 1.376 ± 0.350 |
| Cross-attention fusion | 0.728 ± 0.128 | 1.351 ± 0.315 |
| Gated fusion | 0.645 ± 0.148 | 1.507 ± 0.323 |

Classical baselines on the same splits and the same 25 seeds, across all three feature representations:

| Model | Pearson R | RMSE |
|---|---|---|
| Random Forest, structure | 0.409 ± 0.210 | 1.785 ± 0.309 |
| Random Forest, morphology | 0.764 ± 0.112 | 1.273 ± 0.302 |
| **Random Forest, concatenation** | **0.776 ± 0.101** | **1.247 ± 0.288** |
| Gradient Boosting, structure | 0.372 ± 0.223 | 1.831 ± 0.286 |
| Gradient Boosting, morphology | 0.753 ± 0.088 | 1.327 ± 0.260 |
| Gradient Boosting, concatenation | 0.767 ± 0.095 | 1.276 ± 0.253 |
| RidgeCV, structure | 0.333 ± 0.251 | 1.831 ± 0.336 |
| RidgeCV, morphology | 0.731 ± 0.157 | 1.335 ± 0.370 |
| RidgeCV, concatenation | 0.730 ± 0.145 | 1.345 ± 0.374 |

Splits are made by drug, not by drug–cell-line pair, so no test compound is ever seen during training. Since the models take only compound features and never see cell-line identity, a model's prediction is constant across cell lines for a given drug. Metrics are therefore computed per drug: one prediction against that drug's mean observed LN_IC50, 26 test drugs per seed.

## Setup

```bash
conda create -n jumpcp python=3.10 -y
conda activate jumpcp
pip install -r requirements.txt
```

Run the notebooks in order. Notebooks 00 through 02 build the dataset, 03 runs the modality analysis, 08 produces every model result in the manuscript, and 07 draws the figures from what 08 saved.

Notebook 08 trains 125 models and takes roughly 15 hours on CPU. The loop is resumable: before training anything it checks whether that seed and condition already has a saved prediction file, so you can rerun the cell after an interruption and it picks up where it stopped.

One caveat on reproducibility. Multi-threaded CPU training is not bit-for-bit reproducible even with fixed seeds, because thread scheduling changes the order of floating-point accumulation. Individual per-seed numbers may shift slightly on a rerun; the distribution across 25 seeds is stable. The classical models are fully deterministic given a fixed random_state.

## Data availability

Raw data is not in this repository, but both sources are public:

- **GDSC2 drug response**, release 8.4, from the Genomics of Drug Sensitivity in Cancer database
- **JUMP-CP morphological profiles** (cpg0016), from the JUMP Cell Painting Consortium via the Cell Painting Gallery on AWS S3

This study uses `profiles_var_mad_int.parquet`, the interpretable profiles without Harmony batch correction. That was deliberate: Harmony makes individual feature names meaningless, which would have gutted the SHAP analysis.

The matched profile matrix (`jump_profiles_matched.parquet`) is excluded for size but regenerates from notebook 01.

## Citation

Citation to be added on publication.

## License

MIT.

## Contact

Amir J. Shokrzadeh, amirrshokrzadeh@gmail.com
ORCID: 0009-0009-2766-7992
