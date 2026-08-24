# Kết quả thí nghiệm

Đây là thư mục trình bày tập trung. Source run trong `results/`, `datasets/` và `protocol/` vẫn giữ nguyên để bảo toàn provenance/hash.

## Cách tìm

Tên folder có dạng `STT_YYYYMMDD_ten_thi_nghiem`. Xem `DANH_MUC_THI_NGHIEM.csv` để lọc theo ngày, status và role.

- `LATEST.md`: mốc mới nhất theo thời gian, có thể chỉ là planned;
- `LATEST_OFFICIAL.md`: kết quả official mới nhất;
- trong mỗi thí nghiệm: bảng, hình, báo cáo và manifest được chia riêng;
- không chứa `.safetensors`, `.pt`, `.npy`, checkpoint hoặc raw cache lớn.

## Thứ tự

1. [`01_20260812_roborefer_blind_tomato_can`](01_20260812_roborefer_blind_tomato_can/README.md) — `DIAGNOSTIC`
2. [`02_20260812_two_reference_oblique_tomato`](02_20260812_two_reference_oblique_tomato/README.md) — `DIAGNOSTIC`
3. [`03_20260813_two_reference_oblique_deoccluded`](03_20260813_two_reference_oblique_deoccluded/README.md) — `DIAGNOSTIC`
4. [`04_20260813_roborefer_pilot_v0_aborted`](04_20260813_roborefer_pilot_v0_aborted/README.md) — `ABORTED_BEFORE_CAPTURE`
5. [`05_20260813_roborefer_pilot_v0`](05_20260813_roborefer_pilot_v0/README.md) — `COMPLETE_DIAGNOSTIC`
6. [`06_20260818_wp0_baseline_interface`](06_20260818_wp0_baseline_interface/README.md) — `COMPLETE`
7. [`07_20260818_wp1_depth_sensitivity`](07_20260818_wp1_depth_sensitivity/README.md) — `COMPLETE_DIAGNOSTIC`
8. [`08_20260818_wp2_dataset_prototype`](08_20260818_wp2_dataset_prototype/README.md) — `COMPLETE_PROTOTYPE`
9. [`09_20260821_wp3_data_gap_audit`](09_20260821_wp3_data_gap_audit/README.md) — `COMPLETE_AUDIT`
10. [`10_20260821_wp3_feature_hook_failed_preflight`](10_20260821_wp3_feature_hook_failed_preflight/README.md) — `FAILED_PREFLIGHT`
11. [`11_20260821_wp3_feature_hook_failed_serialization`](11_20260821_wp3_feature_hook_failed_serialization/README.md) — `FAILED_SERIALIZATION`
12. [`12_20260821_wp3_feature_hook_official`](12_20260821_wp3_feature_hook_official/README.md) — `PASSED`
13. [`13_20260821_pcra_u_overfit_smoke_planned`](13_20260821_pcra_u_overfit_smoke_planned/README.md) — `PLANNED_NOT_RUN`
14. [`14_20260821_pcra_u_overfit_smoke_pass`](14_20260821_pcra_u_overfit_smoke_pass/README.md) — `PASSED`
15. [`15_20260821_ycb_inventory_v2_main_world`](15_20260821_ycb_inventory_v2_main_world/README.md) — `PASSED_ASSET_QUALIFICATION`
16. [`16_20260821_ycb_inventory_v2_color_fix`](16_20260821_ycb_inventory_v2_color_fix/README.md) — `PASSED_ASSET_COLOR_AND_SCENE_CLEANUP`
17. [`17_20260821_dataset_expansion_v2_design_lock`](17_20260821_dataset_expansion_v2_design_lock/README.md) — `PASSED_DESIGN_LOCK_NO_CAPTURE_NO_TRAINING`
18. [`18_20260821_dataset_v2_pilot_pass`](18_20260821_dataset_v2_pilot_pass/README.md) — `PASSED_PILOT_ENGINEERING_QC_NOT_OFFICIAL_DATA`
19. [`19_20260821_dataset_v2_development_capture_lock`](19_20260821_dataset_v2_development_capture_lock/README.md) — `PASSED_STATIC_PREFLIGHT_CAPTURE_AUTHORIZED_NO_CAPTURE_NO_TRAINING`
20. [`20_20260821_dataset_v2_development_batch_001_relation_fail`](20_20260821_dataset_v2_development_batch_001_relation_fail/README.md) — `FAILED_BATCH_001_RELATION_GEOMETRY_GATE_NO_TRAINING`
21. [`21_20260821_dataset_v2_relation_repair_pilot_pass`](21_20260821_dataset_v2_relation_repair_pilot_pass/README.md) — `PASSED_RELATION_REPAIR_PILOT_ENGINEERING_ONLY_NO_TRAINING`
22. [`22_20260824_dataset_v2_1_shutdown_gate_pass`](22_20260824_dataset_v2_1_shutdown_gate_pass/README.md) — `PASSED_V2_1_SHUTDOWN_GATE_NO_BASELINE_CHANGE_NO_TRAINING`
23. [`23_20260824_dataset_v2_1_development_capture_lock`](23_20260824_dataset_v2_1_development_capture_lock/README.md) — `PASSED_V2_1_STATIC_PREFLIGHT_CANARY_ONLY_NO_CAPTURE_NO_TRAINING`
24. [`24_20260824_dataset_v2_1_1_development_full_qc_pass`](24_20260824_dataset_v2_1_1_development_full_qc_pass/README.md) — `PASSED_FULL_DEVELOPMENT_QC_GO_DEVELOPMENT_TRAIN_TESTS_SEALED`
25. [`25_20260824_pcra_u_development_train_preflight_pass`](25_20260824_pcra_u_development_train_preflight_pass/README.md) — `PASSED_DEVELOPMENT_TRAIN_PREFLIGHT_GO_CANARY_TESTS_SEALED`
26. [`26_20260824_pcra_u_development_preflight_amendment_pass`](26_20260824_pcra_u_development_preflight_amendment_pass/README.md) — `PASSED_PREFLIGHT_AMENDMENT_GO_CANARY_NO_OPTIMIZER_STEP`
27. [`27_20260824_pcra_u_development_train_canary_pass`](27_20260824_pcra_u_development_train_canary_pass/README.md) — `PASSED_DEVELOPMENT_TRAIN_CANARY_GO_FULL_TRAIN_TESTS_SEALED`
28. [`28_20260824_pcra_u_development_train_exploratory_complete`](28_20260824_pcra_u_development_train_exploratory_complete/README.md) — `COMPLETED_EXPLORATORY_DEV_ONLY_FREEZE_ARCHITECTURE_NEXT_TESTS_SEALED`
29. [`29_20260824_pcra_u_development_failure_audit_architecture_decision`](29_20260824_pcra_u_development_failure_audit_architecture_decision/README.md) — `PASSED_DEVELOPMENT_FAILURE_AUDIT_ARCHITECTURE_CHECKPOINT_FROZEN_CALIBRATION_NEXT_TESTS_SEALED`
30. [`30_20260824_pcra_u_calibration_fit_freeze`](30_20260824_pcra_u_calibration_fit_freeze/README.md) — `PASSED_CALIBRATION_FIT_CALIBRATOR_THRESHOLD_FROZEN_ABSTAIN_ALL_GO_LOCKED_TEST_EVALUATION`
31. [`31_20260824_pcra_u_development_v1_1_rejected_failure_audit`](31_20260824_pcra_u_development_v1_1_rejected_failure_audit/README.md) — `COMPLETED_EXPLORATORY_V1_1_REJECTED_RETAIN_V1_TESTS_SEALED`

## Cập nhật

```bash
python protocol/publish_ketqua.py --update
```

Thêm thí nghiệm mới vào `protocol/ketqua_catalog_v1.json` với STT/ngày/giờ/slug duy nhất trước khi cập nhật.
