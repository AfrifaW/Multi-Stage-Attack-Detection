# Multi-stage Attack Detection and Prediction in IIoT Supply Chains Using Graph Neural Networks

A reproducible implementation of a stage-aware intrusion detection framework for
Industrial IoT (IIoT) supply chains, run on **ToN-IoT** (main dataset) and
**Edge-IIoTset** (second dataset), on identical pipeline code driven entirely by
per-dataset YAML config.

## 1. Setup

```bash
cd iiot-gnn-ids
python3 -m venv .venv && source .venv/bin/activate   # optional but recommended
pip install -r requirements.txt
```

Requires Python 3.10+. Tested with `torch==2.13 (cpu)` and `torch_geometric==2.8`.

## 2. Get the data

This repo does **not** bundle the real datasets (license + size). Download them and
place the CSV(s) here:

| Dataset | Expected file | Place in |
|---|---|---|
| ToN-IoT (Network) | `Train_Test_Network.csv` | `data/raw/toniot/` |
| Edge-IIoTset | `ML-EdgeIIoT-dataset.csv` | `data/raw/edgeiiotset/` |

- ToN-IoT: https://research.unsw.edu.au/projects/toniot-datasets
- Edge-IIoTset: Ferrag et al., IEEE Access 2022 (dataset hosted via IEEE
  DataPort / Kaggle mirrors — search "Edge-IIoTset").

If the exact filename above isn't found, the adapter falls back to globbing
`data/raw/<dataset>/*.csv`, so multiple CSV shards are fine too.

**No real data yet / just want to see the pipeline run?** Generate small synthetic
CSVs that mimic each dataset's real schema:

```bash
python scripts/generate_synthetic_data.py --n-assets 40 --hours 6 --rate-hz 0.05
```

⚠️ **Synthetic data is for smoke-testing the code only.** Results produced on it
have no scientific meaning — every script that runs on synthetic data will still
happily produce metrics/figures, but they describe the synthetic generator, not
either real dataset. Delete `data/raw/*/*.csv` and drop in the real files before
drawing any conclusion.

## 3. Run

Each stage can be run independently, or all at once:

```bash
# Everything, both datasets (stage detection + forecasting + sensitivity sweeps)
python scripts/run_all.py

# Faster run (fewer CV folds) -- good for a first smoke test
python scripts/run_all.py --quick

# One dataset only, skip the slow reviewer-response sweeps
python scripts/run_all.py --datasets toniot --skip-sensitivity
```

Or invoke each stage directly:

```bash
# Stage detection (RF + GCN + stacking meta-learner)
python scripts/run_stage_detection.py --dataset toniot
python scripts/run_stage_detection.py --dataset edgeiiotset

# Impact forecasting (dual-stream GRU). For Edge-IIoTset this first checks a
# data-quality gate (asset cardinality, timestamp parseability) and WRITES A
# REPORT explaining why it was skipped if the gate fails, instead of
# producing untrustworthy numbers -- see results/edgeiiotset/impact_forecasting/.
python scripts/run_impact_forecasting.py --dataset toniot
python scripts/run_impact_forecasting.py --dataset edgeiiotset

# Reviewer-response experiments (window/horizon sensitivity, graph ablation,
# meta-learner ablation, RF tuning table, inference latency table)
python scripts/run_sensitivity.py --dataset toniot
python scripts/run_sensitivity.py --dataset edgeiiotset
```

Useful flags (all scripts): `--graph-mode {interaction_only,temporal_only,both}`,
`--meta-learner {logistic_regression,mlp,gradient_boosting}`, `--n-folds`,
`--delta-t` (window size override), `--seed`. Run `--help` on any script for the
full list.

## 4. Where results go

```
results/
  toniot/
    stage_detection/
      stage_metrics.json          # per-class P/R/F1, Macro-F1, confusion matrix, bootstrap CI
      stage_per_class_metrics.csv
      stage_summary.csv
      confusion_matrix_stacked_test.png
      macro_f1_comparison.png
    impact_forecasting/
      forecast_metrics.json       # ROC-AUC, PR-AUC, F1, Capture@{1,2,5,10}%, bootstrap CI
      forecast_summary.csv
      forecast_predictions.csv    # per-instance (asset, cut window, y_true, y_proba)
      pr_curve_test.png
    sensitivity/
      window_sensitivity.{csv,png}
      horizon_sensitivity.{csv,png}
      graph_ablation.{csv,png}
      meta_learner_ablation.{csv,png}
      baseline_rf_tuning.csv
      inference_latency.csv
  edgeiiotset/
    ... same layout ...
    impact_forecasting/SKIPPED_report.json   # only if the data-quality gate fails
```

## 5. Configuration

Everything data- or model-specific lives in YAML, not in code:

- `configs/toniot.yaml`, `configs/edgeiiotset.yaml` — dataset schema candidates
  (columns the adapter looks for), window size, forecasting horizon, split
  fractions, graph construction, and every model hyperparameter.
- `configs/stage_mapping_toniot.yaml`, `configs/stage_mapping_edgeiiotset.yaml` —
  raw attack-type → {Benign, IAD, LMEP, IMP} mapping, with the rationale for each
  mapping documented inline.

To change a hyperparameter, edit the relevant YAML — no code changes needed.
`src/utils/config.override()` is used internally (e.g. by `run_sensitivity.py`'s
window-size sweep) to produce a modified in-memory copy without touching the file.

```
