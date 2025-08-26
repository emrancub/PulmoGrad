# PulmoGrad: Reproducible ML Workflow for COPD Severity (Everything is empty until a decision)

**What it is.** Leakage-safe, imbalance-aware COPD severity pipeline (ADASYN in-train, nested CV with Hyperopt, RF/XGB/LGBM/CatBoost, soft-voting hybrid, ROC/SHAP, full artifacts).

## Quick start
```bash
# CPU or GPU; prefer a clean venv/conda env

# Everything is empty until a decision

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
│     ├─ cli.py           # entrypoints: fit/report
│     ├─ data.py          # I/O, schema checks, group splits
│     ├─ features.py      # preprocessing pipelines
│     ├─ imbalance.py     # ADASYN (train-only)
│     ├─ models.py        # RF/XGB/LGBM/CatBoost + ensemble
│     ├─ tune.py          # Hyperopt inner-CV search
│     ├─ calibration.py   # isotonic / Platt
│     ├─ metrics.py       # AUC/PR, F1, Brier, ECE, bootstrap CIs
│     ├─ plots.py         # ROC/PR, calibration, SHAP, confusion
│     └─ utils.py         # seeds, logging, helpers
└─ outputs/
   ├─ figures/            # S1–S5, PR, calibration, SHAP, CM
   ├─ tables/             # per-fold metrics with CIs, thresholds
   ├─ probs/              # out-of-fold probabilities (CSV)
   ├─ logs/
   └─ models/
