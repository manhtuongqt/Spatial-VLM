# Kho kết quả tập trung của đồ án

Thư mục này là bản **sao chép có tổ chức** của các số liệu và kết quả đang có
trong workspace. File nguồn vẫn được giữ nguyên để không làm hỏng đường dẫn của
các script huấn luyện và đánh giá.

## Kết quả chính thức nên dùng trong báo cáo

1. `02_huan_luyen_v2/`: cấu hình, lịch sử huấn luyện, đánh giá dev, calibrator
   và checkpoint tốt nhất V2 tại epoch 13 / global step 1120.
2. `03_danh_gia_test_iid/`: kết quả chính thức trên Test-IID 200 family / 1.000
   mẫu, gồm best V2, RoboRefer gốc và so sánh paired bootstrap theo family.
3. `01_du_lieu_test_iid/`: protocol, freeze lock, manifest và báo cáo QC dùng
   để chứng minh tính truy nguyên của Test-IID.

Các thư mục còn lại phục vụ đối chiếu:

- `04_thi_nghiem_v3_3_seed/`: log của ba seed V3; đây là thí nghiệm âm, không
  seed nào đạt điều kiện chọn checkpoint grounding 97%.
- `05_baseline_dev_legacy/`: lần chạy baseline trên dev cũ; không trộn với bảng
  Test-IID chính thức.
- `06_hinh_qc/`: ảnh preview và ảnh QC thật, không phải ảnh sinh bằng AI.
- `07_cau_hinh_va_tai_lieu_ky_thuat/`: config và tài liệu mô tả kiến trúc.
- `08_ket_qua_giai_doan_truoc/`: số liệu/văn bản của các giai đoạn trước, giữ
  nguyên cấu trúc nguồn. Không dùng chung protocol với Test-IID nếu chưa kiểm tra.

## Cấu trúc thư mục

```text
ketqua/
├── 00_tong_quan_va_nguon_goc/
├── 01_du_lieu_test_iid/
├── 02_huan_luyen_v2/
├── 03_danh_gia_test_iid/
├── 04_thi_nghiem_v3_3_seed/
├── 05_baseline_dev_legacy/
├── 06_hinh_qc/
├── 07_cau_hinh_va_tai_lieu_ky_thuat/
└── 08_ket_qua_giai_doan_truoc/
```

## Quy tắc khoa học cần giữ

- Kết quả Test-IID chính thức nằm ở `03_danh_gia_test_iid/`.
- Bootstrap phải lấy mẫu theo 200 family, không coi 1.000 variant là 1.000 mẫu
  độc lập.
- Best V2 vẫn dùng threshold chung 0.5 cho source uncertainty; chưa được mô tả
  là đã tối ưu threshold riêng từng source.
- Prediction Test-IID V2 không có RoboRefer disagreement thực; không được tuyên
  bố tín hiệu đó đã đóng góp vào kết quả chính thức.
- Chưa có đánh giá robot end-to-end, metric 3D hay OOD trong gói kết quả chính.

## Thành phần không nhân đôi

Feature cache, ảnh RGB-D thô, checkpoint trung gian và toàn bộ thư mục `old/`
không được sao chép để tránh tăng thêm nhiều GB và tránh trùng bản. Đường dẫn của
chúng được ghi trong `00_tong_quan_va_nguon_goc/NOT_COPIED.md` và
`LEGACY_OLD_FILE_INDEX.tsv`.

`MANIFEST_SHA256.txt` chứa checksum của mốc GitHub chính thức gồm các thư mục
`00`–`07`. `MANIFEST_LOCAL_ALL_SHA256.txt` kiểm kê toàn bộ kho cục bộ, bao gồm
cả kết quả giai đoạn trước trong thư mục `08`.
