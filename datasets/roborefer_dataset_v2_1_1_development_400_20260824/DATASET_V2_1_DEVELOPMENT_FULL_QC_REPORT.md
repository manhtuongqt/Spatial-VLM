# Dataset V2.1 development full-QC report

- Decision: `GO_DEVELOPMENT_TRAIN`
- Independent families: 400
- Materialized samples: 2000
- Real RGB-D/semantic captures: 800
- Observable relation checks: 356
- Relation failures: 0
- Deterministic replay files compared: 18285
- Training performed: `false`
- Calibration/Test created: `false`

## Batch gates

- `canary_000`: 40 capture; 18 relation checks
- `batch_001`: 190 capture; 77 relation checks
- `batch_002`: 190 capture; 96 relation checks
- `batch_003`: 190 capture; 80 relation checks
- `batch_004`: 190 capture; 85 relation checks

Only real sensor QC images are stored under `report_assets/real_capture_qc/`.
Table 1 contains observed dataset statistics. Tables 2–6 remain `NOT_RUN`.
This PASS authorizes the next gate, `GO_DEVELOPMENT_TRAIN`, using 320 train
families and 80 dev families; it does not authorize Calibration or Test access.
