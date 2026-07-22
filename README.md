# Down Syndrome Drug-Response Biomarker Analysis

This project investigates whether **Memantine** shifts protein expression in trisomic (Down syndrome model) mice toward healthy control levels, using the UCI **Mice Protein Expression** dataset (1,080 mice, 77 proteins, 8 groups defined by Genotype × Treatment × Behavior).

Source: Higuera, Gardiner & Cios (2015), *Self-Organizing Feature Maps Identify Proteins Critical to Learning in a Mouse Model of Down Syndrome*, PLoS ONE.

## Project structure

| File | Purpose |
|---|---|
| `mice_protein_eda_1.ipynb` | EDA: missing-value analysis, KNN imputation, outlier/skew checks, Welch's t-test filtering |
| `model_building_2.ipynb` | Random Forest classifier, leakage-safe split, cross-validation, permutation importance, hypergeometric overlap test |
| `streamlit_app.py` | Interactive app (Story / Protein Explorer / AI Engine tabs) |
| `Data_Cortex_Nuclear.xls` | Raw dataset |
| `model_results.json`, `permutation_importance.csv`, `gini_importance.csv`, `treatment_response_analysis.csv` | Exported outputs the app reads directly, so it doesn't need to retrain on every load |
| `requirements.txt` | Pinned dependency versions |

## Key results

- **Cross-validated accuracy:** 98.6% ± 0.7% (5-fold, on training data) — the more stable number to quote, since it's an average across 5 different splits rather than a single measurement.
- **Held-out test accuracy:** 99.5–100% depending on the exact scikit-learn version (see caveat below). Test accuracy is a single measurement on one 216-mouse split, so it's more sensitive to small implementation differences than the cross-validated figure.
- **Top candidate biomarker:** `pPKCG_N` — also identified as Memantine-responsive in the source paper.
- **EDA/Random Forest overlap:** 3 of the EDA's top-10 statistically-filtered candidates also appear in the model's top-15 permutation-important proteins. A hypergeometric test puts the chance of this overlap (or better) happening at random at **p = 0.30** — not statistically significant, so this agreement is reported as a modest, non-conclusive signal rather than strong independent validation.

## A note on reproducibility

Running the exact same code (same `random_state=42`) on different scikit-learn versions has produced slightly different test-set results — 215/216 correct on one version, 216/216 on another. This is a known characteristic of `RandomForestClassifier`: ties between candidate splits are broken using scikit-learn's internal randomization, and small implementation changes between library versions can shift how those ties resolve. It does not indicate a bug, data leakage, or an error in the analysis — the cross-validated accuracy (98.6% ± 0.7%), which averages over 5 different splits, is unaffected and is the number this project treats as authoritative. `requirements.txt` pins compatible version ranges to keep this reproducible going forward.

## Limitations

- This is an exploratory, hypothesis-generating analysis on a single dataset. Findings would need independent replication before being treated as validated biomarkers.
- The EDA's biomarker-candidate filter (shift toward control + p < 0.05) is a simple dual filter, not a formal multiple-comparisons correction like Bonferroni or Benjamini-Hochberg FDR.
- KNN imputation assumes mice with similar overall protein profiles have similar values for missing proteins — a reasonable assumption, not a guarantee.

## Running locally

```bash
pip install -r requirements.txt
jupyter notebook mice_protein_eda_1.ipynb   # run first, generates treatment_response_analysis.csv
jupyter notebook model_building_2.ipynb     # run second, generates model_results.json + importance CSVs
streamlit run streamlit_app.py
```
