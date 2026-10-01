# RoboRefer gốc vs best V2 — Test-IID

So sánh ghép cặp trên cùng 1.000 sample thuộc **200 family độc lập**. CI 95% là percentile cluster bootstrap; mỗi lần lặp lấy lại 200 family và luôn giữ 5 variant của family đi cùng nhau.

| Phạm vi grounding | n sample | RoboRefer gốc | Best V2 | V2 − gốc |
|---|---:|---:|---:|---:|
| Target hiện hữu (chính) | 835 | 724/835 (86.71% [83.23%, 89.97%]) | 775/835 (92.81% [90.25%, 95.21%]) | +6.11 pp [+1.78 pp, +10.49 pp] |
| Ground-truth FOUND | 420 | 397/420 (94.52% [91.93%, 96.84%]) | 405/420 (96.43% [94.67%, 98.06%]) | +1.90 pp [-0.98 pp, +5.01 pp] |

## Kết quả ghép cặp chính

- Cùng đúng: 668; chỉ RoboRefer đúng: 56; chỉ V2 đúng: 107; cùng sai: 4.
- Kiểm định hoán vị sign-flip theo family: p = 0.00744993 (100,000 lần).
- RoboRefer parse hợp lệ trên toàn bộ Test-IID: 1000/1000; phân bố {'EXACT_ONE_NORMALIZED_POINT': 1000}.

## Interior grounding

- Target hiện hữu — RoboRefer: 723/835 (86.59% [83.11%, 89.84%]); V2: 764/835 (91.50% [88.78%, 94.05%]); chênh lệch +4.91 pp [+0.48 pp, +9.37 pp].
- Truth FOUND — RoboRefer: 397/420 (94.52% [91.93%, 96.84%]); V2: 401/420 (95.48% [93.40%, 97.39%]); chênh lệch +0.95 pp [-2.10 pp, +4.11 pp].

## Khoảng cách tới target

- Target hiện hữu, RoboRefer (chỉ point parse được): mean 27.915px (95% CI 20.186–36.288), median 0.000px, p95 232.801px, max 451.249px.
- Target hiện hữu, V2: mean 8.444px (95% CI 5.567–11.607), median 0.000px, p95 92.073px, max 228.586px.
- Chênh lệch mean distance ghép cặp V2 − RoboRefer (chỉ các cặp parse được): -19.471px (95% CI -28.561 đến -10.937px).

## Đầy đủ point-in-target theo phân tầng target-present

| Phân tầng | Giá trị | n | family | RoboRefer gốc (95% CI) | Best V2 (95% CI) | V2 − gốc (95% CI) |
|---|---|---:|---:|---:|---:|---:|
| variant | clean | 154 | 154 | 128/154 (83.12% [77.07%, 88.82%]) | 140/154 (90.91% [86.16%, 95.27%]) | +7.79 pp [+0.00 pp, +15.54 pp] |
| variant | depth_corruption | 154 | 154 | 125/154 (81.17% [74.83%, 87.20%]) | 136/154 (88.31% [83.11%, 93.08%]) | +7.14 pp [-1.33 pp, +15.75 pp] |
| variant | occlusion_view_counterfactual | 154 | 154 | 124/154 (80.52% [74.05%, 86.62%]) | 136/154 (88.31% [83.04%, 93.10%]) | +7.79 pp [-0.64 pp, +16.20 pp] |
| variant | relation_counterfactual | 173 | 173 | 155/173 (89.60% [84.80%, 93.96%]) | 164/173 (94.80% [91.23%, 97.73%]) | +5.20 pp [-0.56 pp, +10.92 pp] |
| variant | semantic_counterfactual | 200 | 200 | 192/200 (96.00% [93.00%, 98.50%]) | 199/200 (99.50% [98.50%, 100.00%]) | +3.50 pp [+1.00 pp, +6.50 pp] |
| answerability_state | AMBIGUOUS | 105 | 35 | 39/105 (37.14% [24.00%, 51.04%]) | 99/105 (94.29% [86.46%, 100.00%]) | +57.14 pp [+41.38 pp, +72.04 pp] |
| answerability_state | FOUND | 420 | 200 | 397/420 (94.52% [91.94%, 96.80%]) | 405/420 (96.43% [94.66%, 98.06%]) | +1.90 pp [-0.98 pp, +4.99 pp] |
| answerability_state | INSUFFICIENT_EVIDENCE | 310 | 127 | 288/310 (92.90% [89.37%, 96.04%]) | 271/310 (87.42% [81.79%, 92.38%]) | -5.48 pp [-12.46 pp, +0.99 pp] |
| relation | behind | 78 | 56 | 70/78 (89.74% [79.73%, 97.44%]) | 77/78 (98.72% [95.52%, 100.00%]) | +8.97 pp [+0.90 pp, +19.23 pp] |
| relation | between_in_depth | 30 | 10 | 27/30 (90.00% [74.07%, 100.00%]) | 23/30 (76.67% [54.55%, 93.94%]) | -13.33 pp [-39.39 pp, +9.09 pp] |
| relation | direct | 338 | 200 | 311/338 (92.01% [88.10%, 95.69%]) | 307/338 (90.83% [86.17%, 95.29%]) | -1.18 pp [-7.43 pp, +5.23 pp] |
| relation | farther_than | 46 | 26 | 35/46 (76.09% [57.78%, 92.11%]) | 44/46 (95.65% [87.88%, 100.00%]) | +19.57 pp [+0.00 pp, +39.13 pp] |
| relation | front_of | 137 | 53 | 115/137 (83.94% [73.81%, 92.75%]) | 130/137 (94.89% [87.97%, 100.00%]) | +10.95 pp [+0.00 pp, +22.50 pp] |
| relation | left_of | 53 | 29 | 51/53 (96.23% [89.58%, 100.00%]) | 47/53 (88.68% [78.12%, 96.43%]) | -7.55 pp [-17.74 pp, +0.00 pp] |
| relation | nearer_than | 57 | 27 | 45/57 (78.95% [60.98%, 94.83%]) | 55/57 (96.49% [88.46%, 100.00%]) | +17.54 pp [+3.77 pp, +33.33 pp] |
| relation | nearer_than_both | 39 | 13 | 27/39 (69.23% [43.59%, 91.67%]) | 38/39 (97.44% [91.67%, 100.00%]) | +28.21 pp [+3.33 pp, +55.56 pp] |
| relation | right_of | 57 | 27 | 43/57 (75.44% [56.52%, 92.86%]) | 54/57 (94.74% [86.27%, 100.00%]) | +19.30 pp [-1.72 pp, +40.91 pp] |
| family_category | direct_grounding | 138 | 30 | 119/138 (86.23% [78.03%, 93.79%]) | 108/138 (78.26% [68.07%, 88.11%]) | -7.97 pp [-22.76 pp, +6.85 pp] |
| family_category | front_behind_camera | 131 | 30 | 121/131 (92.37% [84.76%, 98.11%]) | 127/131 (96.95% [91.41%, 100.00%]) | +4.58 pp [-2.60 pp, +13.01 pp] |
| family_category | multi_anchor_depth_order | 129 | 30 | 110/129 (85.27% [76.03%, 93.55%]) | 121/129 (93.80% [87.93%, 98.26%]) | +8.53 pp [-2.61 pp, +19.74 pp] |
| family_category | nearer_farther | 138 | 35 | 115/138 (83.33% [73.68%, 92.00%]) | 134/138 (97.10% [93.33%, 100.00%]) | +13.77 pp [+5.11 pp, +23.53 pp] |
| family_category | occlusion_depth_evidence | 149 | 35 | 126/149 (84.56% [75.30%, 92.98%]) | 145/149 (97.32% [92.50%, 100.00%]) | +12.75 pp [+2.78 pp, +23.23 pp] |
| family_category | relation_2d | 150 | 40 | 133/150 (88.67% [80.15%, 96.00%]) | 140/150 (93.33% [89.09%, 97.08%]) | +4.67 pp [-3.50 pp, +13.33 pp] |
| target_object_group | box | 91 | 20 | 88/91 (96.70% [91.36%, 100.00%]) | 61/91 (67.03% [55.17%, 79.69%]) | -29.67 pp [-41.58 pp, -17.57 pp] |
| target_object_group | container | 122 | 30 | 119/122 (97.54% [94.62%, 100.00%]) | 121/122 (99.18% [97.27%, 100.00%]) | +1.64 pp [-1.50 pp, +4.94 pp] |
| target_object_group | cube | 32 | 10 | 30/32 (93.75% [82.35%, 100.00%]) | 29/32 (90.62% [70.97%, 100.00%]) | -3.12 pp [-25.00 pp, +16.67 pp] |
| target_object_group | fruit | 478 | 110 | 378/478 (79.08% [73.79%, 84.20%]) | 456/478 (95.40% [92.94%, 97.52%]) | +16.32 pp [+10.57 pp, +22.18 pp] |
| target_object_group | mug | 112 | 30 | 109/112 (97.32% [92.93%, 100.00%]) | 108/112 (96.43% [91.87%, 100.00%]) | -0.89 pp [-6.87 pp, +5.15 pp] |

Từng cặp sample và số liệu máy đọc nằm trong `paired_samples.csv`, `stratified_grounding.csv` và `full_comparison.json`.
