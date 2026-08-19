# Multi-stage Attack Detection and Prediction in IIoT Supply Chains Using Graph Neural Networks

A reproducible implementation of a stage-aware intrusion detection framework for
Industrial IoT (IIoT) supply chains, run on **ToN-IoT** (main dataset) and
**Edge-IIoTset** (second dataset), on identical pipeline code driven entirely by
per-dataset YAML config.

Structure

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

