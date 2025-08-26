````markdown
# PulmoGrad: Reproducible ML Workflow for COPD Severity *(Everything is empty until a decision)*

## What it is
Leakage-safe, imbalance-aware COPD severity pipeline (**ADASYN** in-train, **nested CV** with Hyperopt, **RF/XGB/LGBM/CatBoost**, **soft-voting ensemble**, probability **calibration**, **ROC/PR/Calibration** curves, **SHAP**, per-fold metrics, and complete **OOF probabilities** for supplements).

---

## Quick start
```bash
# CPU or GPU; prefer a clean venv/conda env
# Everything is empty until a decision — repo ships without data

# 1) Create environment
conda create -n pulmograd python=3.10 -y
conda activate pulmograd
pip install -r requirements.txt    # or: conda env create -f environment.yml

# 2) Put your dataset in data/raw/
#    Expected columns:
#      id, label (MILD|MODERATE|SEVERE|VERY_SEVERE), [feature_1 ... feature_n]
#    Optional: group_id for patient-wise splitting

# 3) Train, evaluate, and export artifacts
python -m pulmograd.cli fit --config configs/default.yaml

# 4) Regenerate figures/tables (ROC/PR/Calibration/SHAP, confusion matrices)
python -m pulmograd.cli report --config configs/default.yaml
````

---

## Repository layout

```
PulmoGrad/
├─ README.md
├─ requirements.txt        # or environment.yml
├─ configs/
│  └─ default.yaml
├─ data/
│  ├─ raw/                # place original CSVs here (not tracked)
│  ├─ interim/            # cached folds, temp artifacts
│  └─ processed/          # model-ready data (auto)
├─ src/
│  └─ pulmograd/
│     ├─ main.py          # entrypoint (CLI or run script)
└─ outputs/
   ├─ figures/            # S1–S5, PR, calibration, SHAP, CM
   ├─ tables/             # per-fold metrics with CIs, thresholds
   ├─ probs/              # out-of-fold probabilities (CSV)
   ├─ logs/
   └─ models/
```

---

## Minimal configuration

**`configs/default.yaml`:**

```yaml
data:
  path: data/raw/your_dataset.csv
  id_col: id
  label_col: label
  group_col: null
  label_map: {MILD: 0, MODERATE: 1, SEVERE: 2, VERY_SEVERE: 3}

validation:
  outer_folds: 5
  inner_folds: 3
  stratify: true
  group_stratify: false
  random_seed: 42
  n_jobs: -1
  ci_bootstrap: 2000

imbalance:
  enabled: true
  method: ADASYN
  params: {sampling_strategy: "auto", n_neighbors: 5, random_state: 42}

preprocess:
  numeric_imputer: median
  categorical_imputer: most_frequent
  one_hot: true
  scale_numeric: false

models:
  rf:   {enabled: true,  hyperopt: {n_trials: 60}}
  xgb:  {enabled: true,  hyperopt: {n_trials: 80}}
  lgbm: {enabled: true,  hyperopt: {n_trials: 80}}
  cat:  {enabled: true,  hyperopt: {n_trials: 80}}

ensemble:
  enabled: true
  kind: soft_voting
  weights: {rf: 1.0, xgb: 1.0, lgbm: 1.0, cat: 1.0}

calibration:
  enabled: true
  method: isotonic
  bins_for_ece: 10

metrics:
  primary: macro_roc_auc
  others: [macro_pr_auc, accuracy, macro_f1, brier, ece10]

outputs:
  probs_dir: outputs/probs
  tables_dir: outputs/tables
  models_dir: outputs/models
  logs_dir: outputs/logs
  figures_dir: outputs/figures
```

---

## What you get after `fit`

**`outputs/tables/`**

* `outer_fold_metrics.csv` — per-fold metrics + mean ± SD; 95% CI via bootstrap
* `thresholds.csv` — VAL-optimized & clinical thresholds

**`outputs/probs/`** *(for supplements)*

* `rf_fold{k}_probs.csv`, `xgb_fold{k}_probs.csv`, `lgbm_fold{k}_probs.csv`, `cat_fold{k}_probs.csv`, `ensemble_fold{k}_probs.csv`
* Columns: `id`, `y_true`, `y_pred_[class0..class3]`, `fold`
* **Please refer to *Supplementary Data S1 (probs/)* for out-of-fold probabilities.**

**`outputs/figures/`**

* *Supplementary Figure S1–S5*: per-fold ROC for RF/XGB/LGBM/Cat/Ensemble
* Precision–Recall curves, calibration plots, SHAP (summary & dependence), confusion matrices

---

## Data schema & privacy

* `id`: unique row identifier (hash or surrogate)
* `label`: `MILD | MODERATE | SEVERE | VERY_SEVERE`
* `features`: routine clinical & spirometric variables (e.g., FEV1, FVC, FEV1/FVC, age, BMI, comorbidities)
* `group_id` *(optional)*: ensures no patient overlap across folds
* **Privacy**: de-identify all data; **do not** upload raw PHI

---

## Validation notes

* All preprocessing, ADASYN, tuning, and calibration are performed **inside** training folds.
* Reported performance is from **outer** held-out folds only (unseen test).
* OOF probabilities are exported to enable independent re-scoring and figure regeneration.

---

## Citation

If this repository helps your work, please cite:

```bibtex
@software{PulmoGrad_2025,
  title  = {PulmoGrad: Reproducible ML Workflow for COPD Severity},
  author = {Your Name and Contributors},
  year   = {2025},
  url    = {https://github.com/emrancub/PulmoGrad}
}
```

---

## License & disclaimer

**MIT.**
Not a medical device. Research use only; do not use for autonomous clinical decision-making.

```
```
