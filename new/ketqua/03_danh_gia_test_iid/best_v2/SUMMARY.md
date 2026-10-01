# Best V2 — Test-IID frozen evaluation

- Samples/families: **1,000 / 200**
- Grounding point-in-target: **92.81%** (835 target-present samples)
- Mean / median point-to-target error: **8.44 / 0.00 px**
- Answerability accuracy / macro-F1: **76.80% / 77.52%**
- Source uncertainty micro/macro-F1: **62.14% / 67.74%**
- Relation-edge accuracy: **77.69%** (650 edges)
- Risk AUROC / AUPRC: **0.9455 / 0.9620**
- Risk calibration Brier / NLL / ECE: **0.0961 / 0.2985 / 0.0465**
- Frozen-threshold coverage / selective risk: **25.50% / 2.35%**
- EXECUTE count / error rate: **255 / 2.35%**
- Conformal region empirical/nominal coverage: **86.83% / 90.00%**

No model, calibrator, threshold, or prompt was fitted on Test-IID.
