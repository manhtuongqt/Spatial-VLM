# RefSpatial unbiased challenge v1

`primary_challenge.jsonl` has 180 RGB-D source-structure examples that were selected without reading any RoboRefer/B0 prediction. Every primary family is excluded from the B1 train/dev/diagnostic dataset and the former B0 agreement queue.

The primary set is source-labelled only. Report its result as an internal machine-source stress evaluation, never as human-certified accuracy, B2 evidence, or Gazebo transfer.

`manual_tie_ambiguity_queue.jsonl` contains 100 source-quarantined tie or missing-rank-sequence cases. It is unscored until independent human decisions are complete in `manual_tie_ambiguity_decisions.csv`.

The evaluation order is fixed: first lock this manifest, then run B0 and B1 with the same RGB-D prompt and decoding protocol. B0-low-agreement cases may be reported only as a post-hoc stress subgroup; they must not replace the full primary paired comparison.
