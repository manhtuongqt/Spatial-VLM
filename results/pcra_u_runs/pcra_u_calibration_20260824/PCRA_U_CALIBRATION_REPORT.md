# P-CRA-U calibration fit report

- Status khoa học: `CALIBRATION_FIT_ONLY`; đây không phải kết quả tổng quát hóa trên Test.
- Gate: `PASS`; quyết định vận hành: `ABSTAIN_ALL`.
- Dữ liệu: `200` family độc lập / `1000` variants phụ thuộc.
- P-CRA-U và RoboRefer hoàn toàn đóng băng; chỉ hai mô hình Platt một biến được fit.
- Test-IID/Test-OOD không được mở hay đọc.

## Chẩn đoán cross-fit chính

Năm fold được chia xác định theo family. Full-fit được lưu để dùng về sau; cross-fit dưới đây dùng để giảm lạc quan khi mô tả Calibration.

| Calibrator | Brier ↓ | NLL ↓ | ECE ↓ | AURC ↓ | Risk@80% ↓ | Coverage@5% ↑ |
|---|---:|---:|---:|---:|---:|---:|
| Spatial Platt cross-fit | 0.1333 | 0.4008 | 0.0384 | 0.3425 | 0.5250 | 0.0000 |
| Multimodal Platt cross-fit | 0.1331 | 0.4005 | 0.0375 | 0.3420 | 0.5250 | 0.0000 |

Mọi khoảng tin cậy trong CSV được bootstrap 5.000 lần theo family, giữ nguyên năm variants của family được lấy mẫu.

## Operating point đã khóa

- Không threshold nào đồng thời đạt empirical risk ≤ 5% và ít nhất 60 accepted family.
- Kết quả trung thực được khóa là `ABSTAIN_ALL`, coverage bằng 0; không nới rule sau khi xem số liệu.

## Diễn giải trung thực

Trên cross-fit Calibration, multimodal có point estimate Brier và AURC cùng thấp hơn spatial. Đây vẫn chưa phải bằng chứng Test.

AURC dùng tích phân hình thang trên risk–coverage curve theo các threshold risk duy nhất. Risk/coverage là theo variants; CI cluster theo family và threshold yêu cầu ít nhất 60 family độc lập.

## Ranh giới kết quả

- Chỉ Bảng 4 dạng `CALIBRATION_FIT_ONLY` được tạo.
- Bảng 2, 3, 5 và 6 vẫn `NOT_RUN`.
- Sau freeze PASS, hành động duy nhất được phép là capture và mở locked Test-IID/Test-OOD một lần.
