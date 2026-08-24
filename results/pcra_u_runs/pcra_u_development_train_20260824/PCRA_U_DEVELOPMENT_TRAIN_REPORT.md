# P-CRA-U development training — exploratory report

- Decision: `DEVELOPMENT_TRAIN_COMPLETE_FREEZE_ARCHITECTURE_NEXT`
- Scientific status: `EXPLORATORY_DEV_ONLY`.
- Train: `320 families / 1,600 samples`; dev selection: `80 families / 400 samples`.
- Epochs/steps: `15` / `3000`.
- Selected checkpoint: `results/pcra_u_runs/pcra_u_development_train_20260824/checkpoints/development/step_000002000`.
- Dev total loss: `1.351613`.
- Dev answerability macro-F1: `0.742024`.
- Dev false-FOUND on AMBIGUOUS/ABSENT: `0.094737`.
- Dev FOUND point-in-target: `0.857143`.
- Dev source micro-F1: `0.587629` (four active sources; exploratory).
- Peak VRAM: `0.243 GiB`.
- Calibration/Test remained sealed; Tables 2–6 remain NOT_RUN.

These dev values are model-development diagnostics, not final thesis test results.
