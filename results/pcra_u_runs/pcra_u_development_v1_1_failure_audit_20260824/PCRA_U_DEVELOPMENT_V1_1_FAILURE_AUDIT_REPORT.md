# P-CRA-U V1.1 failure audit

- Audit: `PASS`.
- Decision: `RETAIN_V1_STEP_2000_REJECT_V1_1`.
- Completed evidence: 3 fresh seeds × 20 epochs = 60 DEV checkpoints; 12,000 optimizer steps total.
- Eligible checkpoints: `0/60`. No V1.1 prediction replay or bootstrap was authorized.
- Retained model: locked V1 epoch 10 / step 2000.
- Calibration and Test remained sealed.

## Best observed ineligible checkpoints (diagnostic only)

| Seed | Epoch | C | Gc | Ga | A | F | Failed eligibility checks |
|---:|---:|---:|---:|---:|---:|---:|:---|
| 24082026 | 19 | 0.660068 | 0.500000 | 0.773810 | 0.762057 | 0.147368 | all_found_grounding_at_least_minimum, false_found_rate_at_most_maximum |
| 24082027 | 6 | 0.655862 | 0.531250 | 0.732143 | 0.609747 | 0.010526 | all_found_grounding_at_least_minimum, answerability_macro_f1_at_least_minimum |
| 24082028 | 16 | 0.643905 | 0.468750 | 0.761905 | 0.731878 | 0.105263 | all_found_grounding_at_least_minimum |

These checkpoints are ranked with the locked rule `max C → min F → max Gc → earliest epoch`; the ranking is descriptive and creates no best pointer or promotion authority.

## Post-report TypeError

The persisted campaign report and locked source show that all three `selected_checkpoint_dev_metrics` values were null. After writing the campaign report and campaign gate, the table-only loop attempted `metrics["clean_found"]`, yielding `TypeError: 'NoneType' object is not subscriptable`. This occurred after training, seed reports and all 60 immutable checkpoints were written. It therefore changed no weight, metric, checkpoint, eligibility result, or scientific decision; only `three_seed_selected_checkpoints.csv` was not created.

The seed-report `exact_locked_dev_denominators=false` flag was mechanically caused by absent selected metrics. Direct inspection of every epoch JSON confirms exact denominators `400/80/168/32/95` for samples/families/all-FOUND/clean-FOUND/false-FOUND-risk.

## Integrity gates

- `exact_three_seeds_twenty_epochs_sixty_epoch_json`: `PASS`
- `all_sixty_epoch_artifacts_valid_exact_denominators`: `PASS`
- `zero_of_sixty_checkpoints_eligible`: `PASS`
- `all_twelve_thousand_optimizer_steps_finite_and_sequential`: `PASS`
- `all_sixty_checkpoint_manifests_and_hashes_manager_audit_pass`: `PASS`
- `all_checkpoint_epoch_metric_crosslinks_pass`: `PASS`
- `all_seed_reports_fail_only_without_eligible_selection`: `PASS`
- `no_best_pointer_and_no_winner_predictions`: `PASS`
- `campaign_failure_report_and_protected_hashes_consistent`: `PASS`
- `locked_live_inputs_match_campaign_snapshots`: `PASS`
- `table_generation_typeerror_isolated_after_training_and_reports`: `PASS`
- `locked_v1_epoch10_step2000_unchanged_and_verified`: `PASS`
- `critical_inputs_unchanged_during_audit`: `PASS`
- `calibration_and_tests_sealed`: `PASS`
- `prediction_and_bootstrap_prerequisites_failed_so_not_run`: `PASS`

## Scientific boundary

V1.1 is rejected under the precommitted development policy because no epoch jointly met Ga, macro-F1 and false-FOUND eligibility. The audit does not reinterpret an ineligible checkpoint, run post-hoc inference, or open Calibration/Test. A future V1.2 experiment requires a new precommitted contract.
