# Spatial-VLM for RGB-D visual grounding and uncertainty

Đồ án nghiên cứu visual grounding có quan hệ không gian trên RGB-D cho robot
UR3. Hướng chính kết hợp frozen RoboRefer features với một sidecar P-CRA-U để
dự đoán target heatmap, answerability và nguồn bất định; mọi đánh giá được khóa
theo scene-query family để tránh coi các counterfactual variant là quan sát độc
lập.

## Trạng thái khoa học (2026-08-24)

- Dataset development V2.1.1: **400 family / 800 RGB-D capture / 2.000 sample**,
  chia family-disjoint thành 320 train và 80 dev; full-QC PASS.
- P-CRA-U V1: checkpoint development được chọn tại epoch 10 / step 2.000.
- Revision V1.1: train mới từ đầu với 3 seed, dropout, weight decay, learning
  rate thấp và clean-FOUND localization weighting. Cả ba seed chạy đủ 20 epoch
  nhưng không checkpoint nào qua tiêu chí eligibility đã khóa; V1 tiếp tục được
  giữ. Đây là kết quả âm trên DEV, không phải kết quả Test.
- Calibration/Test-IID/Test-OOD không được dùng để chọn V1.1; Test vẫn chưa mở.

Các con số chi tiết phải đọc từ report/CSV đã sinh, không suy diễn từ tên file
hoặc infographic planned.

## Cấu trúc repository

- `plan/`: kế hoạch đồ án và work-package roadmap.
- `protocol/`: contract, schema, manifest, preflight, capture/QC, training,
  checkpoint manager, evaluation và visualization code.
- `evidence/` hoặc các đường dẫn `results/` được force-track có chọn lọc: báo
  cáo, bảng và figure nhỏ từ dữ liệu thật; không chứa tensor/checkpoint.
- `ketqua/`: danh mục thí nghiệm và manifest provenance gọn nhẹ.
- `DEPENDENCIES.md`: phiên bản các source tree ngoài repository này.

## Quy tắc dữ liệu và reproducibility

- `datasets/`, feature cache, model weights và checkpoint không được commit vì
  dung lượng lớn và có manifest/hash riêng.
- RoboRefer luôn frozen trong các thí nghiệm sidecar; optimizer chỉ nhận tham
  số P-CRA-U.
- Train/dev được tách theo family; Calibration và Test không được dùng để chọn
  kiến trúc hoặc checkpoint.
- V1.1 chọn checkpoint bằng score đã khóa gồm clean grounding, all-FOUND
  grounding, answerability macro-F1 và false-FOUND; không chọn theo total loss.

## Điểm bắt đầu

1. Đọc `plan/ke_hoach_v3_spatial_vlm_visual_grounding_uncertainty.md`.
2. Đọc các contract hiện hành trong `protocol/`, đặc biệt dataset V2.1.1,
   development training, failure audit và calibration.
3. Dùng `protocol/run_pcra_u_development.sh` để kích hoạt đúng môi trường khi
   chạy training script đã khóa.

Repository này lưu code và bằng chứng đọc được. Dataset, cache và checkpoint
được giữ cục bộ, được định danh bằng SHA-256 trong manifest/report.

