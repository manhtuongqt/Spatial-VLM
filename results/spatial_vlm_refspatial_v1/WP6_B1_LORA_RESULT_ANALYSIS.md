# Báo cáo phân tích LoRA B1 trên RefSpatial

**Ngày chạy:** 09/09/2026
**Trạng thái:** B1 pilot hoàn tất; chưa đủ bằng chứng để khẳng định B1 cải thiện RoboRefer, mở B2, hoặc chuyển sang Gazebo.

## 1. Mục tiêu và phạm vi

B0 là RoboRefer-2B-SFT gốc. B1 là cùng backbone, gắn LoRA để kiểm tra liệu fine-tuning trên các câu hỏi spatial ranking của RefSpatial có cải thiện pointing và độ bất định hay không.

Tập train là `D_tabletop_machine_v1`, một pilot được lọc tự động. Nó **không phải** `D_tabletop_clean_v1`, không có chứng nhận nhãn độc lập của người review, không dùng cho B2 reasoning supervision và không dùng làm claim cuối cùng.

| Thành phần | Số lượng | Ghi chú |
|---|---:|---|
| Train | 750 QA RGB-D | 650 scene-family, family-disjoint với eval |
| Train thực dùng | 744 QA | 6 mẫu cuối không được dùng vì ranh giới batch/gradient accumulation |
| Dev | 90 QA | chỉ đánh giá |
| Diagnostic | 90 QA | chỉ đánh giá |
| WP6 primary challenge | 180 QA | 180 family; không chọn theo output B0 và không chồng với B1 |
| Manual tie/ambiguity queue | 100 QA | chưa score, chờ human review |

Mỗi QA gồm RGB, depth, instruction và tọa độ target chuẩn hóa. SAM2 không được dùng.

## 2. Cấu hình B1 LoRA và khả thi tài nguyên

| Thuộc tính | Giá trị |
|---|---|
| Base model | RoboRefer-2B-SFT |
| Trainable parameter | 18,464,768 / 2,475,464,416 (0.7459%) |
| LoRA | rank 16, alpha 32, dropout 0.05 |
| Phần được adapter hóa | 392 tensor LLM; không có vision/depth tensor |
| Precision | base 4-bit NF4, compute bfloat16 |
| Batch vật lý / accumulation | 1 / 8 |
| Epoch / optimizer step | 1 / 93 |
| Learning rate | 2e-4 |
| Thời gian train | 915.5 giây |
| GPU | RTX 2000 Ada 16 GB |
| Peak VRAM allocated / reserved | 4.70 / 9.82 GiB |

Loss train giảm từ mean 3.225 ở năm step đầu xuống 0.227 ở năm step cuối. Đây chỉ chứng minh B1 tối ưu được objective train; nó không chứng minh generalization hay reasoning tốt hơn.

Nguồn chi tiết: [B1_PILOT_REPORT.md](wp4_b1_lora_pilot/B1_PILOT_REPORT.md), [B1_PILOT_REPORT.json](wp4_b1_lora_pilot/B1_PILOT_REPORT.json).

## 3. Vì sao không dùng WP5 để kết luận

WP5 đánh giá trên 90 dev và 90 diagnostic từ chính pilot machine-filtered. Các mẫu đó từng qua điều kiện B0 agreement, nên B0 và B1 đều đạt Hit@.08 = 100%. Kết quả này bị selection bias: nó phù hợp để xác nhận pipeline chạy đúng, nhưng không phân biệt được năng lực B0 với B1 hoặc calibration khi có lỗi.

Vì vậy WP6 dùng 180 primary mẫu được chốt bằng source structure trước khi chạy B0/B1. Các family WP6 bị loại khỏi B1 train/dev/diagnostic và B0 agreement queue cũ. Primary vẫn chỉ có source label, nên kết quả là **machine-source evaluation**, không phải human-certified accuracy.

Nguồn: [WP5 report](wp5_b0_b1_machine_eval/B0_B1_REPORT.md), [WP6 protocol](wp6_unbiased_challenge/CHALLENGE_PROTOCOL.md).

## 4. Protocol đánh giá WP6

- Cùng RGB, depth và instruction cho B0/B1; target chỉ được đọc sau generation để score.
- Một greedy output cho pointing; năm output stochastic với seed cố định để đo self-consistency.
- Correctness: normalized L2 point error <= 0.08.
- Confidence: tỷ lệ output stochastic nằm trong bán kính 0.08 của greedy output.
- Uncertainty proxy: `1 - confidence`.
- Calibration: ECE 10 bins và Brier trên correctness của greedy output.
- Paired comparison: 180 cặp family-disjoint, McNemar exact và 10,000 family-cluster bootstrap draws.

Self-consistency là một uncertainty proxy, không phải PCRAU calibrator đã fit. Vì vậy ECE/Brier ở đây nói về proxy này trên source labels, không chứng minh calibration cho robot thật.

## 5. Kết quả primary WP6

| Metric | B0 | B1 | Nhận xét |
|---|---:|---:|---|
| N / parse rate | 180 / 100% | 180 / 100% | không có lỗi định dạng output |
| Hit@.08 | 96.67% (174/180) | 96.67% (174/180) | không đổi |
| Wilson 95% CI của Hit@.08 | 92.92–98.46% | 92.92–98.46% | cùng accuracy |
| Mean point error | 0.01310 | 0.01262 | B1 thấp hơn rất nhỏ |
| Mean confidence | 0.9911 | 0.9944 | B1 tự tin hơn |
| ECE | 0.0356 | 0.0344 | giảm rất nhỏ |
| Brier | 0.0324 | 0.0304 | giảm rất nhỏ |
| Error-detection AUROC | 0.656 | 0.576 | B1 kém hơn B0 trong việc xếp lỗi cao uncertainty |

Kết quả paired:

- Delta Hit@.08 B1−B0 = **0.000**; bootstrap 95% CI `[-0.0167, 0.0167]`.
- B0 sai/B1 đúng = 1; B0 đúng/B1 sai = 1; McNemar p = 1.000.
- Delta mean point error B1−B0 = **-0.000479**; bootstrap 95% CI `[-0.002798, 0.001452]`, có chứa 0.
- Delta mean confidence B1−B0 = +0.00333.

Do đó, B1 **không có cải thiện pointing accuracy có ý nghĩa** trên primary challenge. Mean error giảm rất nhỏ nhưng chưa phân biệt được khỏi nhiễu mẫu. B1 hơi tăng confidence, trong khi error-AUROC giảm, nên không có bằng chứng rằng LoRA làm uncertainty hữu ích hơn.

### Theo relation

| Relation | N | B0 Hit@.08 | B1 Hit@.08 | Kết luận |
|---|---:|---:|---:|---|
| horizontal ordinal ranking | 162 | 97.53% | 97.53% | accuracy không đổi |
| leftmost ranking | 11 | 90.91% | 90.91% | N quá nhỏ; confidence đều 1.0 dù có lỗi |
| rightmost ranking | 7 | 85.71% | 85.71% | N quá nhỏ; confidence đều 1.0 dù có lỗi |

Những lỗi extremum tuy ít nhưng quan trọng: cả B0 lẫn B1 có thể lặp lại một điểm sai với confidence 1.0. Đây là bằng chứng trực tiếp rằng stochastic self-consistency hiện chưa đủ làm safety signal cho robot.

## 6. Low-agreement stress subgroup hậu nghiệm

Nhóm này được xác định **sau** khi có B0, bằng điều kiện B0 confidence <= 0.60. Có chỉ 2/180 mẫu; cả hai B0 đều đúng.

| Metric | B0 | B1 |
|---|---:|---:|
| N | 2 | 2 |
| Hit@.08 | 100% | 100% |
| Mean confidence | 0.50 | 0.90 |
| ECE | 0.50 | 0.10 |
| Brier | 0.26 | 0.02 |

N=2 quá nhỏ và nhóm được chọn theo B0 output, nên các số này chỉ mô tả hành vi. Chúng không được dùng để kết luận B1 tốt hơn hoặc thay thế kết quả primary.

## 7. Kết luận và quyết định

1. B1 LoRA chạy ổn định trên GPU 16 GB và học được loss train, nên hạ tầng LoRA khả thi.
2. WP5 không phù hợp để kết luận vì dữ liệu được chọn bởi B0 agreement.
3. WP6 loại bỏ selection đó ở bước chọn mẫu, nhưng source labels vẫn chưa qua human certification.
4. Trên WP6 primary, B1 không cải thiện Hit@.08, không có paired delta có ý nghĩa, và không cải thiện error-detection uncertainty.
5. Không mở B2, không đưa B1 làm model được chọn cho Gazebo, và không trình bày đây là kết quả reasoning/PCRAU đã được xác nhận.

## 8. Công việc tiếp theo

1. Review bằng người các lỗi chung và hai lỗi không trùng của B0/B1 trong WP6; xác minh target, object set, ordinal direction và frame.
2. Review hàng đợi 100 tie/ambiguity để tạo benchmark human-certified, tách riêng train và evaluation.
3. Thiết kế objective dữ liệu mới nếu muốn cải thiện reasoning/uncertainty: label ambiguity, abstention/uncertainty target và hard negatives phải được xác nhận, thay vì chỉ tăng số QA machine-filtered.
4. Chỉ đánh giá Gazebo sau khi benchmark human-certified cho thấy improvement trên pointing và uncertainty; khi đó dùng top-table camera cùng coordinate convention đã khóa.

Nguồn kết quả máy: [WP6 challenge report](wp6_unbiased_challenge/evaluation/B0_B1_CHALLENGE_REPORT.md), [WP6 metrics JSON](wp6_unbiased_challenge/evaluation/B0_B1_CHALLENGE_METRICS.json), [primary manifest](wp6_unbiased_challenge/primary_challenge.jsonl).
