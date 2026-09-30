# Thành phần không sao chép vào kho kết quả

Các thành phần dưới đây vẫn còn nguyên tại đường dẫn nguồn:

| Thành phần | Đường dẫn nguồn | Lý do |
|---|---|---|
| Feature cache Test-IID | `new/test_iid/feature_cache/` | Cache tái tạo được, dung lượng lớn |
| Ảnh và depth Test-IID đầy đủ | `new/test_iid/dataset/media/` | Dữ liệu đầu vào, không phải bảng số liệu kết quả |
| Capture RGB-D thô | `new/test_iid/dataset/raw/captures/` | Dữ liệu thô dung lượng lớn |
| Checkpoint V2 trung gian | `new/outputs/pcrau_target_v2_full_seed_24082026/checkpoints/epoch_*` | Đã giữ checkpoint best epoch 13 |
| Checkpoint V3 | `new/outputs/pcrau_target_v3_seed_*/checkpoints/` | V3 không đạt gate chọn checkpoint |
| Archive cũ | `old/results/`, `old/ketqua/` | Nhiều bản trùng và artifact thất bại; chỉ lập chỉ mục |

Không có file nguồn nào bị xóa hoặc di chuyển khi tạo kho này.
