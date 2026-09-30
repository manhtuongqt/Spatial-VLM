# GitHub milestone: Best V2 + Test-IID

- Ngày chốt: 2026-09-30
- Nhánh: `results/best-v2-test-iid-20260930`
- Tag: `results-best-v2-test-iid-20260930`
- Phạm vi commit: `new/ketqua/README.md`, file mốc, manifest và các thư mục
  `00_tong_quan_va_nguon_goc` đến `07_cau_hinh_va_tai_lieu_ky_thuat`.

## Nội dung được chốt

- Best V2 checkpoint: epoch 13, global step 1120.
- Test-IID: 200 family, 1.000 variant.
- RoboRefer gốc và best V2 trên cùng Test-IID.
- Paired bootstrap 95% theo family.
- Log ba seed V3, được ghi nhận là thí nghiệm âm vì không đạt gate grounding
  97%.

## Không đưa vào commit này

- `08_ket_qua_giai_doan_truoc/`: kết quả cũ khác protocol, có dữ liệu trùng và
  một số file vượt giới hạn kích thước GitHub.
- Feature cache, RGB-D thô và checkpoint trung gian.
- Các thay đổi khác đang tồn tại trong working tree.

Mục đích của nhánh/tag riêng là giữ mốc kết quả này độc lập với code và kết quả
cũ trên `main`. Không được hiểu tag này là kết quả OOD hoặc robot end-to-end.
