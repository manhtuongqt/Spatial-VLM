# Development result interpretation

Status: **EXPLORATORY_DEV_ONLY**. These values were observed on the 80-family dev
split used to select the checkpoint; they are not locked-test or calibrated thesis
results.

## What passed

- Full runtime gate: PASS (`320/320` train families, `80/80` dev families).
- Selected checkpoint: epoch 10 / step 2,000, chosen only by minimum dev total
  loss (`1.351613`).
- Checkpoint root: 15/15 immutable checkpoints valid; best and last pointers valid.
- Dev answerability: accuracy `0.7475`, macro-F1 `0.7420`.
- False `FOUND` on ground-truth `AMBIGUOUS ∪ ABSENT`: `9/95 = 9.47%`.
- Dev `FOUND` grounding: point-in-target `144/168 = 85.71%`, point-in-interior
  `139/168 = 82.74%`, mean heatmap mass-in-target `0.5451`.
- Dataset/baseline hashes remained unchanged. Calibration/Test remained sealed.

## Required limitations

- Source micro-F1 is only `0.5876`. Dataset V2.1.1 has no positive `spatial`
  source label, and `semantic`/`relation` sources always co-occur. Therefore this
  run cannot support a scientific claim of five-way independent source
  attribution. The active four-source result is exploratory only.
- Relation subgroup values have unequal and sometimes very small denominators.
  For example `between_in_depth` has 3 FOUND dev records and
  `nearer_than_both` has 1. They must not be interpreted as stable estimates.
- `right_of` is currently `3/7` on FOUND dev records and should be included in the
  development failure audit before architecture freeze.
- The reported ~0.24 GiB peak VRAM is sidecar training after pooled features have
  been cached. It is not end-to-end RoboRefer memory. The preflight tower extraction
  peak was about 6.14 GB (`5.72 GiB`).
- Five variants per family are correlated. Family is the independent statistical
  unit even though all variants were used during optimization/evaluation.

## Next scientifically valid action

Freeze or amend the architecture using only train/dev evidence. Before collecting
Calibration, explicitly audit the selected checkpoint's errors by answerability,
relation, variant and family. Do not fit a calibrator or open Test-IID/Test-OOD
until the architecture/checkpoint policy is frozen.
