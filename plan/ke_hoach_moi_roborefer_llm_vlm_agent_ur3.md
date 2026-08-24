# KẾ HOẠCH NGHIÊN CỨU V2 — ROBOREFER + P-CRA + LLM/VLM AGENT + UR3

> **Tên đề tài:** Nghiên cứu phương pháp LLM/VLM Agent lập kế hoạch vòng kín với hiệu chuẩn độ bất định không gian và can thiệp thích ứng cho thao tác gắp đặt bằng tay máy UR3.
>
> **Ngày cập nhật:** 18/08/2026
>
> **Model nền:** RoboRefer-2B-SFT
>
> **Họ phương pháp học đề xuất:** **P-CRA — Probabilistic Cross-modal Relation-Aware**; tiếng Việt: **phương pháp xác suất liên phương thức có nhận biết quan hệ**
>
> **Nền tảng:** ROS 2 Humble, MoveIt 2, Gazebo, UR3, gripper, camera RGB-D D435i eye-in-hand
>
> **Phần cứng phát triển hiện tại:** RTX 2000 Ada 16 GiB
>
> **Trạng thái:** Baseline B0/B1/B2 và pilot 10 cảnh đã hoàn thành ở chế độ shadow. Chưa fine-tune P-CRA, chưa hiệu chuẩn uncertainty, chưa thực hiện pick-and-place vòng kín trong protocol pilot.

`P-CRA` hiện là tên làm việc nội bộ. Không tuyên bố tên hoặc kiến trúc này là mới so với toàn bộ literature trước khi hoàn thành novelty review riêng.

---

## 0. Quyết định định hướng

Đồ án không chỉ trả lời câu hỏi “RoboRefer trỏ vào đâu?”. Câu hỏi trung tâm là:

> Khi VLM trả một point 2D nhưng chưa cung cấp rủi ro đã hiệu chuẩn, robot phải thu thập và hợp nhất bằng chứng RGB-D nào, xác định nguyên nhân bất định ra sao, chọn can thiệp nào và xác minh thế nào để hoàn thành pick-and-place an toàn trong vòng kín?

Định hướng được khóa ở ba tầng:

1. **Perception và spatial uncertainty:** RoboRefer và P-CRA xác định target, phân phối vị trí, answerability và nguồn bất định.
2. **Calibration và adaptive intervention:** biến raw score thành grounding/action risk đã hiệu chuẩn; chọn `EXECUTE`, `REOBSERVE`, `REPROMPT`, `ASK_USER` hoặc `ABSTAIN`.
3. **Closed-loop manipulation:** MoveIt 2 và UR3 lập kế hoạch, thực thi, xác minh và recovery.

P-CRA được chia thành hai mức để tránh làm thay đổi model trước khi có bằng chứng:

- **P-CRA-U — Probabilistic Cross-modal Relation-Aware Uncertainty:** nhánh phụ ước lượng bất định xác suất liên phương thức có nhận biết quan hệ. Nhánh này đọc feature RGB, depth và câu lệnh nhưng không thay đổi media token đi vào Qwen2. Đây là phương pháp học ưu tiên.
- **P-CRA-F — Probabilistic Cross-modal Relation-Aware Fusion:** bộ hợp nhất đặc trưng xác suất liên phương thức có nhận biết quan hệ. Nhánh này thay đổi feature mà Qwen2 sử dụng bằng residual fusion. Chỉ triển khai nếu depth-sensitivity test và failure audit chứng minh fusion là nút thắt.

Adapter không thay thế calibration, policy, geometric verification hoặc robot safety. P-CRA cũng không phải toàn bộ Agent; nó là mô-đun perception–uncertainty bên trong Agent.

### 0.1. Giải nghĩa P-CRA-U và P-CRA-F bằng tiếng Anh và tiếng Việt

Tên `P-CRA` được dùng cho **một họ phương pháp**, không phải một mô-đun duy nhất. Ý nghĩa từng thành phần như sau:

| Ký hiệu | Tiếng Anh | Tiếng Việt | Ý nghĩa trong đồ án |
|---|---|---|---|
| `P` | Probabilistic | Xác suất | Mô hình tạo phân phối hoặc raw score thay vì chỉ một quyết định cứng. Raw score vẫn phải được hiệu chuẩn độc lập trước khi gọi là xác suất đúng. |
| `C` | Cross-modal | Liên phương thức | Kết hợp bằng chứng từ RGB, depth và ngôn ngữ. |
| `R` | Relation | Quan hệ | Xét các quan hệ như trái/phải, trước/sau, gần/xa, ở giữa và quan hệ với vật mốc. |
| `A` | Aware | Có nhận biết | Mức sử dụng RGB/depth được điều kiện hóa theo quan hệ trong câu lệnh, không hợp nhất mọi nguồn theo một cách cố định. |
| `U` | Uncertainty | Độ bất định | Ước lượng model có chắc về target, vị trí, quan hệ và depth hay không. |
| `F` | Fusion | Hợp nhất | Hợp nhất feature RGB–depth–language để có thể cải thiện trực tiếp suy luận point của RoboRefer. |

Tên đầy đủ và bản dịch dùng thống nhất trong báo cáo:

| Viết tắt | Tên tiếng Anh đầy đủ | Tên tiếng Việt dùng trong đồ án |
|---|---|---|
| **P-CRA-U** | **Probabilistic Cross-modal Relation-Aware Uncertainty** | **Mô-đun xác suất ước lượng độ bất định liên phương thức có nhận biết quan hệ** |
| **P-CRA-F** | **Probabilistic Cross-modal Relation-Aware Fusion** | **Mô-đun xác suất hợp nhất đặc trưng liên phương thức có nhận biết quan hệ** |

Hiểu ngắn gọn:

- **P-CRA-U hỏi:** “Có nên tin point RoboRefer vừa trả không; hệ thống đang không chắc ở đâu và do nguồn nào?” Nó chạy như một **nhánh phụ song song**: chỉ đọc đặc trưng của RoboRefer, không sửa đường suy luận gốc và không tự thay point gốc.
- **P-CRA-F hỏi:** “Có thể tạo point tốt hơn bằng cách đưa bằng chứng depth vào đúng vị trí và đúng quan hệ trong câu lệnh không?” Nó là **nhánh can thiệp vào đặc trưng**: dùng một phần hiệu chỉnh bổ sung có điều kiện theo quan hệ trước khi các media token được Qwen2 sử dụng, vì vậy có thể làm thay đổi point đầu ra.

Hai nhánh khác nhau về vai trò và mức rủi ro kỹ thuật:

| Tiêu chí | P-CRA-U | P-CRA-F |
|---|---|---|
| Vai trò chính | Định lượng spatial uncertainty, answerability và nguồn bất định | Cải thiện trực tiếp grounding bằng fusion RGB–depth–language |
| Vị trí can thiệp | Nhánh phụ sau hoặc song song với đặc trưng thị giác | Trên đường đặc trưng chính trước Qwen2 |
| Có sửa point RoboRefer không? | Không; chỉ so sánh point gốc với heatmap/MAP của nhánh phụ | Có thể có, vì feature đầu vào Qwen2 đã được hiệu chỉnh |
| Output chính | Heatmap, top-k mode, entropy, answerability, wrong-object score, source scores | Đặc trưng RGB/depth đã hợp nhất và point/answer mới của RoboRefer |
| Mục tiêu sử dụng | Calibration và chọn `EXECUTE`, `REOBSERVE`, `ASK_USER`, `ABSTAIN` | Tăng accuracy ở các câu lệnh thật sự phụ thuộc depth/quan hệ |
| Thứ tự triển khai | Là phương pháp ưu tiên, triển khai trước | Chỉ mở khi audit chứng minh fusion là nút thắt |
| Rủi ro | Thấp hơn vì giữ nguyên đường suy luận nền | Cao hơn vì có thể làm giảm khả năng tương thích hoặc direct grounding |

Trong luận văn, lần xuất hiện đầu tiên phải ghi đủ tên Anh–Việt; các phần sau mới dùng `P-CRA-U` và `P-CRA-F`. Từ **xác suất** trong tên chỉ mô tả dạng đầu ra phân phối/raw probabilistic score, không đồng nghĩa các score đã được calibrated.

---

## 1. Phạm vi và cách mô tả hệ thống

### 1.1. RoboRefer thực sự nhận và trả gì

Trong nhánh RGB-D:

```text
RGB + biểu diễn relative depth + câu lệnh
                    ↓
             RoboRefer-2B-SFT
                    ↓
        point estimate 2D chuẩn hóa (u, v)
```

Depth có thể tham gia suy luận bên trong RoboRefer. Không được mô tả rằng model luôn chọn point chỉ từ RGB rồi mới đọc depth.

Sau khi RoboRefer trả point, pipeline robot mới dùng registered metric depth để suy ra hình học:

```text
z = Dmetric(u, v)
X = (u - cx) z / fx
Y = (v - cy) z / fy
Pcamera = (X, Y, z)
Pbase = Tbase_camera · Pcamera
```

Trong thực tế phải dùng vùng depth, point cloud hoặc robust local statistic thay vì tin một pixel đơn lẻ.

Với prompt và decoding của pilot, model bị yêu cầu trả đúng một point. Cách phát biểu học thuật chính xác là:

> RoboRefer cung cấp point estimate cho tác vụ đang xét, nhưng API hiện tại không cung cấp phân phối không gian, answerability hoặc xác suất grounding failure đã hiệu chuẩn kèm theo point đó.

Không khẳng định rằng RoboRefer về bản chất chỉ có thể sinh một point cho mọi tác vụ. Đây là ràng buộc của protocol/output hiện tại.

### 1.2. Hai representation depth phải được giữ riêng

Mỗi sample lưu cả:

- `depth_metric.npy`: registered depth float theo mét, dùng cho geometry, 3D uncertainty và robot.
- `depth_roborefer.png`: relative inverse-depth view phù hợp input distribution của RoboRefer.

Không dùng PNG đã chuẩn hóa theo từng frame để tuyên bố sai số XYZ metric. Không dùng Depth Anything relative depth làm metric ground truth khi D435i/Gazebo depth đã có.

### 1.3. Năm tầng output phải tách biệt

| Tầng | Output | Thành phần sinh output |
|---|---|---|
| Model nền | Raw answer, point chuẩn hóa, point pixel | RoboRefer |
| P-CRA | Heatmap, top-k modes, answerability, raw source scores | P-CRA-U/P-CRA-F |
| Geometry | Mask/component, depth statistics, point cloud, center/dimensions/covariance 3D | RGB-D estimator |
| Decision | Calibrated risk, intervention, reason code | Calibrator + Agent policy |
| Robot/evaluator | Plan, grasp/place/verify và ground-truth metrics | MoveIt 2, UR3, evaluator |

Attention, gate value, geometric support ratio và raw neural score chỉ là **evidence** trước calibration. Không gọi chúng là xác suất đúng thực tế.

### 1.4. Các loại mask không được trộn

- `M_target`: pixel thuộc đúng target instance; dùng cho semantic/spatial grounding.
- `M_interior`: vùng target sau erosion và valid-depth; dùng cho point placement tránh biên.
- `M_graspable`: vùng đáp ứng surface, gripper clearance và grasp policy.
- `M_reachable`: vùng/pose đạt IK và collision constraints ở robot state cụ thể.
- `M_place`: vùng đặt hợp lệ.

Không huấn luyện spatial grounding chỉ bằng `M_target ∩ M_reachable`. Một vật đúng nhưng tạm thời không reachable vẫn phải được nhận diện là đúng target. Grounding risk và action risk phải tách riêng.

---

## 2. Bằng chứng thực nghiệm hiện có

### 2.1. Pilot B0/B1/B2 ngày 13/08/2026

Protocol đã hoàn thành:

- 10/10 cảnh Gazebo được capture và input QC đạt;
- 20/20 truy vấn RoboRefer thật thành công;
- 30/30 prediction hợp lệ;
- B0 và B1 dùng cùng RGB và prompt;
- B2 dùng nguyên point B1, không query lại;
- model, scene, gate, annotation và artifact được khóa hash;
- semantic oracle chỉ được mở sau prediction lock;
- không target handoff và không thao tác robot.

Kết quả:

| Mode | Point hit trên 8 positive | Eroded-mask hit 4 px | False accept | False reject | Shadow task success |
|---|---:|---:|---:|---:|---:|
| B0 RGB-only | 8/8 | 7/8 | 2 | 0 | 8/10 |
| B1 RGB-D | 8/8 | 8/8 | 2 | 0 | 8/10 |
| B2 B1 + depth gate | 8/8 | 8/8 | 2 | 1 | 7/10 |

Latency trung bình:

- B0: khoảng `1.060 s`;
- B1: khoảng `1.831 s`;
- B2 gate: khoảng `3.64 ms`.

B1 tốn thêm khoảng `0.771 s`, tương đương 73%, nhưng chưa thay đổi target selection ở pilot này.

Artifact tham chiếu:

- [Báo cáo pilot B0/B1/B2](../results/roborefer_pilot_v0_20260813_173305/RESULT.md)
- [Evaluator report](../results/roborefer_pilot_v0_20260813_173305/evaluation/RESULT.md)
- [GUI đối chiếu 10 cảnh](../results/roborefer_pilot_v0_20260813_173305/DEMO_GUI.html)

### 2.2. Diễn giải đúng từ pilot

1. Grounding trên tám cảnh có một target đang tốt ở quy mô pilot.
2. RGB-D chỉ tạo khác biệt rõ ở scene 7: point dịch từ vùng cách biên khoảng 3 px sang khoảng 8 px.
3. B0 và B1 có độ dịch point trung bình khoảng 3.94 px; phần lớn scene gần như giống nhau.
4. Chưa có bằng chứng đủ rằng RoboRefer đang khai thác depth hiệu quả cho quan hệ 3D.
5. Scene 9 cho thấy policy bị ép chọn một trong nhiều ứng viên vì output schema không cho phép hỏi lại.
6. Scene 10 cho thấy target-absent hallucination: model trỏ vào chuối khi khoan điện không tồn tại.
7. Depth component gate chỉ xác nhận bề mặt hình học quanh seed; nó không xác nhận identity hoặc answerability.

### 2.3. Root cause của scene 7

Depth component nối với seed B1 có diện tích `39,117 px`, trong khi gate khóa tối đa `36,864 px` tương đương 12% ảnh. Component bị loại vì quá lớn, rồi code trả reason chung “no valid depth component”. Visible semantic mask của chuối khoảng `18,032 px`, cho thấy candidate depth có khả năng dính thêm bề mặt cùng độ sâu.

Kết luận:

- đây là false rejection của downstream segmentation/gate;
- không được tăng threshold trên chính locked pilot rồi coi kết quả mới là test độc lập;
- gate v2 phải log reason cụ thể như `AREA_TOO_LARGE`, pre-filter area, component count, border contact và seed distance;
- P-CRA fusion không tự động sửa lỗi component dính nền.

### 2.4. Giới hạn bằng chứng

- Pilot chỉ có 10 cảnh, mỗi task family gần như một mẫu.
- Scene 6 mới dùng proxy visible-mask cho chiều cao vật lý.
- Scene 8 chưa có amodal/raycast evidence đầy đủ cho occlusion.
- `eroded-mask hit` 4 px không đồng nghĩa robot-safe grasp.
- `shadow task success` không phải physical pick-and-place success.
- Đã có lỗi executable permission, trajectory timeout và MoveIt teardown; hạ tầng chưa đủ ổn định cho locked robot test.

Pilot được giữ bất biến như bằng chứng thăm dò và dùng để tạo giả thuyết, không dùng làm train/calibration set hoặc để chọn threshold cuối.

---

## 3. Khoảng trống nghiên cứu

### 3.1. Point estimate không đi kèm rủi ro đã hiệu chuẩn

RoboRefer chưa trực tiếp cung cấp:

- phân phối spatial trên target;
- top-k target modes;
- xác suất target tồn tại và câu lệnh có answerable hay không;
- xác suất chọn sai instance;
- uncertainty của XYZ;
- xác suất action failure.

Nếu point được truyền thẳng sang controller thì hệ thống đang blind-trust giao diện perception–control.

### 3.2. Nhiều nguồn bất định đang bị trộn

Tối thiểu phải tách:

1. `semantic`: sai class/instance hoặc nhiều instance tương đương;
2. `relation`: sai target–relation–anchor hoặc multi-reference;
3. `spatial`: point gần biên, sai vùng hoặc heatmap đa mode;
4. `depth`: missing/noisy/bias/edge dropout;
5. `occlusion`: target hoặc anchor chỉ quan sát một phần;
6. `calibration`: intrinsics, hand–eye, timestamp và TF;
7. `geometry`: component/mask/plane/shape sai;
8. `action`: IK, collision, grasp, place và verification.

Source label có thể multi-causal. `source_scores` dùng independent sigmoid và không bắt buộc cộng thành 1.

### 3.3. Fusion RGB–depth hiện là implicit fusion

RGB và depth đi qua hai SigLIP tower/projector riêng, sau đó được chèn thành các media token riêng để Qwen2 xử lý. Self-attention có thể học tương tác nhưng chưa có mô-đun tường minh buộc depth contribution thay đổi theo relation và anchor.

Đây là giả thuyết cần audit. Không mặc định rằng thêm adapter chắc chắn tốt hơn.

### 3.4. Hard gate không tạo Agent vòng kín

Hard gate hiện chỉ `PASS/FAIL`. Agent cần chọn can thiệp phù hợp nguyên nhân:

- ambiguity → `ASK_USER` hoặc `REPROMPT`;
- depth/mask kém → `REOBSERVE`;
- plan không khả thi → `REPLAN` hoặc đổi grasp;
- grasp verification fail → `REGRASP`;
- risk vẫn cao hoặc hết budget → `ABSTAIN`.

---

## 4. Mục tiêu, câu hỏi nghiên cứu và giả thuyết

### 4.1. Mục tiêu tổng quát

Xây dựng LLM/VLM Agent dùng RoboRefer và RGB-D để:

1. grounding target từ ngôn ngữ;
2. biểu diễn uncertainty spatial và depth;
3. hiệu chuẩn grounding risk và action risk;
4. chọn can thiệp thích ứng;
5. thực hiện pick-and-place bằng UR3;
6. xác minh và recovery trong vòng kín.

### 4.2. Câu hỏi nghiên cứu

- **RQ1:** RGB-D RoboRefer cải thiện task type nào so với RGB-only trong eye-in-hand UR3?
- **RQ2:** Correct depth có làm output khác đáng kể so với shuffled, flat hoặc inverted depth ở task phụ thuộc depth không?
- **RQ3:** Heatmap, answerability, source evidence và geometric evidence dự báo grounding failure tốt đến mức nào?
- **RQ4:** Có thể hiệu chuẩn `r_ground` và `r_action` đủ tốt để selective execution không?
- **RQ5:** Adaptive intervention có giảm wrong execution và phục hồi false rejection so với always-execute/hard gate không?
- **RQ6:** Closed-loop verification và recovery cải thiện end-to-end task success bao nhiêu?
- **RQ7 có điều kiện:** P-CRA-F có cải thiện depth-dependent/multi-reference grounding so với P-CRA-U và RoboRefer gốc không?

### 4.3. Giả thuyết

- **H1:** RGB-D hữu ích nhất ở near/far, front/behind, overlapping projection và multi-reference có depth contrast.
- **H2:** Spatial heatmap, answerability và disagreement với point gốc dự báo semantic/spatial failure tốt hơn một hard threshold hình học.
- **H3:** Kết hợp learned evidence với depth/geometry/planning evidence cải thiện risk–coverage so với từng nguồn riêng.
- **H4:** Calibration độc lập giảm accepted-incorrect risk tại coverage tương đương.
- **H5:** Source-aware intervention tốt hơn policy luôn retry cùng một cách.
- **H6:** P-CRA-F chỉ có lợi khi fusion error chiếm tỷ lệ đáng kể; nó không sửa trực tiếp segmentation, TF hoặc grasp failure.

---

## 5. Kiến trúc hệ thống mục tiêu

```text
OBSERVE
  RGB + registered metric depth + CameraInfo + TF + robot state
        │
        ├─────────────── RoboRefer gốc ───────────────→ point p_RR
        │                      │
        │                      └→ RGB/depth feature
        │                                 │
        │               contextual target/relation/anchor queries
        │                                 │
        │                           P-CRA-U sidecar
        │                ┌────────────────┼─────────────────┐
        │                ↓                ↓                 ↓
        │         target heatmap    answerability      source scores
        │                └────────────────┼─────────────────┘
        │                                 ↓
        ├→ metric geometry → mask/point cloud/3D/covariance/relation margins
        │                                 ↓
        └──────────── disagreement + geometry + planning evidence
                                          ↓
                                  independent calibration
                                          ↓
                               r_ground và r_action
                                          ↓
DECIDE
  EXECUTE / REOBSERVE / REPROMPT / ASK_USER / REPLAN / ABSTAIN
                                          ↓
ACT
  MoveIt 2 + UR3 pick/place
                                          ↓
VERIFY
  correct target / attach / lift / place / detach
                                          ↓
DONE hoặc RECOVER → OBSERVE
```

### 5.1. Hai mức rủi ro

```text
r_ground = P(target instance hoặc point sai | perception evidence)
r_action = P(thao tác thất bại/không an toàn | grounding, geometry, plan)
```

Tách hai rủi ro để không nhầm:

- đúng target nhưng grasp pose kém;
- sai target nhưng trajectory hoàn toàn hợp lệ;
- perception tốt nhưng attach/place thất bại.

### 5.2. Agent state tối thiểu

| State | Trách nhiệm |
|---|---|
| `OBSERVE` | Thu RGB-D đồng bộ, CameraInfo, TF và robot state |
| `GROUND` | Gọi RoboRefer, khóa raw point và feature provenance |
| `ESTIMATE` | Tạo region, point cloud, relations và 3D uncertainty |
| `ASSESS` | Chạy P-CRA, calibrator và tính hai mức risk |
| `DECIDE` | Chọn intervention có reason code và budget |
| `EXECUTE` | Plan/approach/grasp/lift/place bằng MoveIt 2 |
| `VERIFY` | Kiểm tra correct target, attach, lift, place và detach |
| `RECOVER` | Reobserve, reprompt, replan, regrasp hoặc corrective place |
| `DONE` | Hoàn thành có verifier xác nhận |
| `ABORT` | Dừng an toàn với lý do có cấu trúc |

VLM không được quyền bỏ qua collision checking, workspace, joint limits hoặc emergency stop.

---

## 6. P-CRA-U — Uncertainty sidecar / Nhánh phụ ước lượng bất định ưu tiên

### 6.1. Tensor contract theo implementation hiện tại

Với một tile `448×448`, hai SigLIP tower có patch size 14 và hidden size 1152:

```text
R0 = Ergb(I) ∈ R^(32×32×1152)
D0 = Edepth(Zrel) ∈ R^(32×32×1152)
```

Projector hiện là `mlp_downsample_3x3_fix`, nên sau projector:

```text
R = Prgb(R0) ∈ R^(11×11×1536)
D = Pdepth(D0) ∈ R^(11×11×1536)
```

Kích thước `11×11` xuất hiện vì projector pad grid 32 lên 33 rồi downsample 3×3. Checkpoint dùng dynamic tiling, vì vậy số tile thay đổi theo aspect ratio. Implementation P-CRA bắt buộc phải:

- giữ tile index, tile bounding box và mapping về ảnh gốc;
- assert RGB/depth có cùng tile count và spatial alignment;
- merge overlap giữa tile heatmap;
- không giả định mọi input chỉ có một grid 32×32.

Spatial head ưu tiên đọc `R0,D0` trước projector để giữ độ phân giải 32×32. Nhánh Qwen2 vẫn dùng `R,D` nguyên bản.

### 6.2. Contextual relation query extractor

Câu lệnh được token hóa và qua một contextualizer nhỏ trước khi learned queries cross-attend. Không chỉ cross-attend trực tiếp với lexical embedding chưa có ngữ cảnh.

Output:

```text
q_target
q_relation_1 ... q_relation_L
q_anchor_1 ... q_anchor_K
anchor_valid_mask
```

Thiết kế phải hỗ trợ:

- direct grounding: không có anchor;
- một anchor: left/right/front/behind;
- hai hoặc nhiều anchor: between, near A but far B;
- nhiều clause trong cùng instruction.

Trong prototype, chọn `Kmax=3`, `Lmax=3`. Slot thừa được mask. Target/anchor attention được giám sát bằng instance masks và relation labels của simulator.

### 6.3. Relation-conditioned sidecar fusion

Tại patch `i`:

```text
g_i = sigmoid(Wg [R0_i ; D0_i ; q_relation])
F_i = LayerNorm(R0_i + g_i ⊙ Wd D0_i + Arel(R0_i, D0_i, queries))
```

Ý nghĩa:

- color/name/direct task có thể ưu tiên RGB;
- near/far/front/behind có thể tăng depth contribution;
- depth hole/noise làm giảm contribution;
- anchor queries giúp depth evidence gắn với đúng reference object.

Ở P-CRA-U, `F` chỉ đi vào auxiliary heads. Không thay thế RGB/depth token của RoboRefer.

### 6.4. Spatial probability head

Spatial head tạo logit trên từng patch/tile, remap về một heatmap thống nhất trên ảnh gốc rồi softmax:

```text
P(u,v) = softmax(h_loc(F, queries))
Σ P(u,v) = 1
```

Output không chỉ có kỳ vọng. Bắt buộc lưu:

- MAP point;
- top-k connected modes và probability mass;
- entropy;
- peak margin `p1-p2`;
- số mode;
- local covariance quanh mode được chọn;
- disagreement giữa MAP và `p_RR`.

Không dùng global mean làm robot point khi heatmap đa mode vì mean có thể nằm giữa hai vật.

### 6.5. Target, anchor và grasp heads

Tách các head:

- target attention/mask;
- từng anchor attention/mask;
- target interior probability;
- graspability probability tùy chọn.

Grounding head học `M_target/M_interior`; action head học `M_graspable/M_reachable`. Không dùng reachability để thay đổi semantic target label.

### 6.6. Answerability và wrong-object head

P-CRA-U phải xuất:

```text
p_answerable_raw
p_wrong_object_raw_if_answered
decision logits: FOUND / AMBIGUOUS / ABSENT / INSUFFICIENT_EVIDENCE
```

`p_wrong_object` dùng:

- pooled fused feature;
- heatmap entropy, peak margin và mode count;
- target/anchor attention quality;
- distance/disagreement giữa point RoboRefer và P-CRA MAP;
- source uncertainty scores;
- geometry evidence chỉ khi phase tương ứng cho phép.

Raw score chưa được gọi là calibrated probability. Failure head được huấn luyện sau khi location head ổn định để nhãn instance của peak không thay đổi liên tục.

### 6.7. Source uncertainty head

P-CRA-U dự báo các nguồn quan sát được từ RGB, relative depth và câu lệnh:

```text
semantic
relation
spatial
depth
occlusion
```

Tầng `ASSESS` bên ngoài P-CRA ghép thêm các nguồn từ hệ thống:

```text
calibration
geometry
action
```

Trong simulator, perturbation source là supervision mạnh. Trong dữ liệu thật, source có thể không xác định hoàn toàn; dùng multi-label/weak label và không tuyên bố causal identification chỉ từ observation.

### 6.8. Metric depth và 3D uncertainty

P-CRA nhận relative depth feature để reasoning, nhưng geometry sidecar đọc registered metric depth. Nếu học depth residual:

```text
μ_z(u,v) = z_metric(u,v) + Δz(F,u,v)
s_z(u,v) = log σ_z²(u,v)
```

Khi sample spatial point:

```text
(u_k,v_k) ~ selected spatial mode
z_k ~ p(z | u_k,v_k)
Pcamera_k = backproject(u_k,v_k,z_k,K)
Pbase_k = Tbase_camera · Pcamera_k
```

Tính `μ_XYZ, Σ_XYZ` bằng Monte Carlo hoặc Jacobian propagation. Bản đầy đủ phải đưa vào uncertainty của intrinsics, hand–eye, timestamp/TF và không giả định `(u,v)` độc lập với `z`.

---

## 7. P-CRA-F — Relation-aware residual fusion / Hợp nhất bằng nhánh hiệu chỉnh dư có nhận biết quan hệ

### 7.1. Điều kiện mở nhánh

Chỉ mở P-CRA-F nếu development audit thỏa đồng thời:

1. Có tối thiểu 30 scene families depth-dependent được kiểm soát.
2. RGB-D gốc không cải thiện rõ so với RGB-only trên nhóm đó.
3. Correct depth không tạo khác biệt đủ so với shuffled/flat/inverted depth.
4. Tối thiểu khoảng 20% actionable perception failures thuộc relation/depth fusion, không phải component, calibration hoặc action.
5. Đã có train/dev/calibration riêng; locked test chưa mở.

Ngưỡng cuối phải pre-register trước audit chính thức. Con số 20% ở đây là working gate cho development, không phải kết quả thống kê.

### 7.2. Thiết kế residual bảo toàn compatibility

Không thay hai media streams bằng một stream mới. Dùng residual zero-initialized:

```text
R' = R + α · ΔR(R,D,Q)
D' = D + β · ΔD(R,D,Q)
```

với `α,β` khởi tạo 0 hoặc rất nhỏ. Giữ nguyên:

- thứ tự image/depth token;
- số token;
- hidden size 1536;
- Qwen2 output format.

P-CRA-F phải được so sánh với P-CRA-U để tách lợi ích của fusion khỏi lợi ích của uncertainty/calibration.

### 7.3. Điều kiện giữ P-CRA-F trong phương pháp cuối

- cải thiện paired metric trên depth-dependent validation/development theo protocol khóa;
- không giảm direct grounding quá tolerance pre-register;
- không làm calibration xấu hơn;
- latency/VRAM nằm trong budget;
- không chỉ sửa một task family hoặc một scene đã xem.

Quyết định giữ/bỏ P-CRA-F phải được freeze trước khi mở locked test. Nếu không đạt trên development, báo negative result và giữ P-CRA-U làm phương pháp chính.

---

## 8. Dataset và counterfactual generation

### 8.1. Đơn vị split

Đơn vị dữ liệu là `scene-query family`, không phải frame. Mọi clean/noisy version, paraphrase, view và retry của cùng family phải nằm cùng split.

Một family tối thiểu có:

1. clean;
2. semantic counterfactual;
3. relation counterfactual;
4. depth corruption;
5. occlusion/view counterfactual.

Không ép mọi version phải answerable. Mỗi version lưu valid target set và expected intervention.

### 8.2. Lộ trình quy mô

**Prototype schema:**

- 50 families × 5 ≈ 250 samples.

**Development dataset:**

- 300–500 families;
- dùng để debug loader, losses, uncertainty signals và intervention.

**Full simulator target nếu tài nguyên cho phép:**

| Split | Families | Samples x5 | Mục đích |
|---|---:|---:|---|
| Train | 4,000 | 20,000 | Train P-CRA/heads |
| Validation | 400 | 2,000 | Model/feature selection |
| Calibration | 800 | 4,000 | Calibrate probability/conformal |
| Test | 800 | 4,000 | Locked Test-IID/Test-OOD |
| Tổng | 6,000 | 30,000 |  |

Test 800 families được chia trước thành IID và OOD theo asset/layout/noise generator. Không chuyển family giữa split sau khi xem output.

### 8.3. Dữ liệu cần lưu từ Gazebo

- RGB gốc và JPEG/PNG thực sự gửi model;
- `depth_metric.npy` và `depth_roborefer.png`;
- semantic/instance segmentation;
- target mask và từng anchor mask;
- interior, valid-depth, graspable, reachable và placement mask tách riêng;
- object poses, dimensions và semantic IDs evaluator-only;
- camera intrinsics/extrinsics, distortion và TF timestamp;
- robot state và candidate approach policy;
- relation graph, reference frame và clause labels;
- target ID, anchor IDs, valid target IDs;
- answerable state và desired intervention;
- perturbation source, severity, generator version và seed;
- RGB-depth-label time spread và sensor QC;
- code/config/model/checkpoint hashes.

### 8.4. Counterfactual theo nguồn

| Nguồn | Cách tạo | Nhãn/expected action |
|---|---|---|
| Semantic | Thêm vật cùng class/màu/hình dạng | `AMBIGUOUS`, `ASK_USER` |
| Absent | Chuyển target khỏi FOV | `ABSENT`, `ABSTAIN` |
| Relation | Đảo layout hoặc đổi anchor/query | target mới hoặc ambiguous |
| Depth | Bias, Gaussian noise, holes, edge dropout, shuffled/inverted | `REOBSERVE` nếu evidence kém |
| Occlusion | Visible fraction 75/50/25%, edge truncation | `REOBSERVE` hoặc execute theo risk |
| Calibration | Perturb intrinsics/extrinsics/time alignment | calibration risk |
| Action | Block reachability/collision/grasp approach | `REPLAN/ABSTAIN` |
| OOD | Mesh, material, camera pose, relation composition mới | đánh giá generalization |

Generator phải randomize để model không học artifact cố định của perturbation.

### 8.5. Record schema rút gọn

```json
{
  "id": "scene_0042_q02_depth_medium",
  "family_id": "scene_0042_q02",
  "image": "scene_0042.png",
  "depth_relative": "scene_0042_depth_roborefer.png",
  "depth_metric": "scene_0042_depth_m.npy",
  "target_mask": "apple_02_target.png",
  "target_interior_mask": "apple_02_interior.png",
  "graspable_mask": "apple_02_graspable.png",
  "anchor_masks": ["cup_blue_01.png"],
  "instruction": "Point to the apple behind the blue cup.",
  "spatial_label": {
    "target_id": "apple_02",
    "anchor_ids": ["cup_blue_01"],
    "relations": ["behind"],
    "reference_frame": "camera"
  },
  "uncertainty_label": {
    "answerable": true,
    "sources": ["depth"],
    "severity": 2,
    "expected_intervention": "REOBSERVE"
  }
}
```

Dataset loader phải trả metadata/masks cho auxiliary heads nhưng tuyệt đối không chèn evaluator-only labels vào conversation hoặc inference inputs.

### 8.6. Camera thật

Thu riêng data thật sau khi simulator pipeline ổn định:

- 150–250 mẫu chỉ đủ proof-of-concept calibration tổng thể;
- cần nhiều hơn nếu tuyên bố coverage theo từng subgroup;
- family grouping vẫn áp dụng;
- threshold Gazebo không được tuyên bố có coverage tương tự trên camera thật nếu chưa recalibrate.

---

## 9. Loss và quy trình huấn luyện

### 9.1. Location mass loss

Vì nhiều pixel trong target/interior mask đều đúng:

```text
L_loc = -log(Σ_(u,v ∈ M_target) P(u,v) + ε)
L_interior = -log(Σ_(u,v ∈ M_interior) P(u,v) + ε)
```

Không ép heatmap khớp một arbitrary center point.

### 9.2. Target–anchor mask loss

```text
L_mask = Dice/BCE(A_target, M_target)
       + Σ_k valid_k · Dice/BCE(A_anchor_k, M_anchor_k)
```

### 9.3. Answerability và source loss

```text
L_answerable = BCE(p_answerable_raw, y_answerable)
L_source = BCE(source_logits, multi_hot_sources)
```

`AMBIGUOUS`, `ABSENT` và `INSUFFICIENT_EVIDENCE` cần nhãn riêng ngoài binary answerability khi dataset đủ lớn.

### 9.4. Depth heteroscedastic loss

Trên valid metric depth:

```text
L_depth = exp(-s_z) · |z - μ_z| + s_z
```

hoặc Gaussian NLL. Phải mask invalid depth. Khi metric depth đã có, ưu tiên học residual/quality thay vì tái tạo metric depth từ relative PNG.

### 9.5. Ranking và failure loss

```text
L_rank = max(0, margin - (u_perturbed - u_clean))
L_wrong = BCE(p_wrong_raw, 1[peak instance != target instance])
```

Không buộc P-CRA phải đồng ý với point RoboRefer. Disagreement là evidence; agreement loss quá mạnh có thể khiến sidecar sao chép lỗi của model nền.

### 9.6. Grasp/action loss

Nếu triển khai learned action head:

- loss trên `M_graspable` tách khỏi `M_target`;
- reachability/plan success là action labels;
- không backprop action failure thành semantic class error nếu target vẫn đúng.

### 9.7. Lịch huấn luyện

**Stage T0 — Baseline extraction**

- đóng băng toàn bộ RoboRefer;
- chạy B0/B1 trên train/dev;
- lưu `p_RR`, feature provenance, đúng/sai instance, 2D/3D error;
- chạy depth counterfactual.

**Stage T1 — P-CRA-U trên feature cache**

- đóng băng RGB/depth tower, projectors và Qwen2;
- cache pre-projector features cùng tile mapping;
- train relation extractor, P-CRA fusion và location/mask/answerability heads;
- BF16, effective batch 16–32, batch vật lý 1–2 với gradient accumulation;
- head LR khởi điểm `3e-4`, adapter LR `1e-4`, 5–10 epochs;
- mục tiêu khoảng 3–5 triệu trainable parameters.

**Stage T2 — Failure/source heads**

- khóa location head đã ổn định;
- chạy lại train set để xác định predicted peak instance;
- train wrong-object/source heads;
- tune trên validation, không trên calibration/test.

**Stage T3 — Calibration**

- đóng băng model và feature set;
- fit temperature/Platt/isotonic hoặc conformal threshold trên calibration split;
- freeze calibrator và decision thresholds trước test.

**Stage T4 — P-CRA-F/LoRA tùy điều kiện**

- chỉ chạy sau gate ở mục 7;
- LoRA rank 8 trên `q_proj,k_proj,v_proj,out_proj` của hai block cuối RGB/depth tower;
- Qwen2 vẫn đóng băng ở thử nghiệm đầu;
- LR `1e-5` đến `2e-5`, 2–3 epochs, gradient checkpointing;
- feature cache T1 không còn hợp lệ và phải recompute.

Existing LoRA selector phải được thay bằng explicit module filter cho RGB tower, depth tower và layer indices; không dùng nguyên broad `find_all_linear_names()`.

---

## 10. Calibration và selective prediction

### 10.1. Từ raw evidence đến calibrated risk

Lộ trình:

1. heuristic evidence baseline;
2. logistic regression/MLP nhẹ cho error prediction;
3. Platt, temperature hoặc isotonic calibration;
4. conformal/selective prediction nếu định nghĩa coverage phù hợp.

Calibration report gồm:

- reliability diagram;
- Brier score;
- ECE;
- AUROC/AUPRC error detection;
- risk–coverage và AURC;
- accepted-incorrect risk;
- false accept/false reject theo subgroup.

### 10.2. Spatial conformal region

Vì ground truth là valid region chứ không phải một point duy nhất, nonconformity phải được định nghĩa theo set. Hai lựa chọn phải được thử trên validation rồi pre-register một cách:

```text
s_i = min_(c ∈ M_valid_i) [-log P_i(c)]
```

hoặc score dựa trên cumulative probability mass của `M_valid`. Coverage event phải ghi rõ là:

- prediction set intersect valid target region;
- selected point thuộc target interior;
- hoặc accepted action thành công.

Ba event này không tương đương. Spatial coverage không thay thế action-safety calibration.

### 10.3. Minimum evidence cho negative safety claim

Nếu mục tiêu là false-accept rate dưới khoảng 5%, locked test cần ít nhất khoảng 60 negative scenes và không có false accept để upper 95% bound xấp xỉ 5% theo rule-of-three. Đây chỉ là minimum; subgroup claims cần nhiều mẫu hơn.

---

## 11. Geometric estimator và gate v2

### 11.1. Vai trò đúng

Gate v2 kiểm tra geometric support và grasp feasibility. Nó không xác nhận semantic identity hoặc answerability.

### 11.2. Cải tiến bắt buộc

- reason code cụ thể: `NO_VALID_DEPTH`, `AREA_TOO_SMALL`, `AREA_TOO_LARGE`, `NO_SEED_SUPPORT`, `BORDER_TRUNCATED`, `PLANE_MERGE`, `UNSTABLE_3D`;
- log tất cả component trước/after filter;
- table-plane removal hoặc above-plane clustering;
- point-seeded RGB-depth segmentation thay vì threshold depth thuần khi cần;
- component splitting khi dính bề mặt cùng độ sâu;
- mask centroid/distance-transform point cho grasp candidate;
- surface normal, object extent và clearance;
- covariance qua nhiều frame/view;
- không tune threshold trên locked pilot/test.

### 11.3. Diagnostic replay

Được phép replay nhiều gate configs trên pilot v0 để tạo giả thuyết, nhưng mọi kết quả phải ghi `post-hoc diagnostic`, không thay thế score v0. Threshold cuối được chọn trên development/calibration scenes mới.

---

## 12. Adaptive intervention policy

| Evidence/reason chính | Hành động | Điều kiện tiếp theo |
|---|---|---|
| `AMBIGUOUS`, nhiều heatmap modes | `ASK_USER` | Câu trả lời phải phân biệt được instance |
| Semantic disagreement, prompt nhạy | `REPROMPT` | Paraphrase phải khóa nghĩa và budget |
| Depth holes/noise/plane merge/border | `REOBSERVE` | Đổi eye-in-hand pose có chủ đích |
| Relation margin thấp | `REOBSERVE` | View mới phải tăng expected information |
| Grounding tốt, plan fail | `REPLAN` | Đổi grasp pose/approach, không gọi VLM vô cớ |
| Grasp verification fail | `REOBSERVE + REGRASP` | Không báo success giả |
| `ABSENT` hoặc risk vượt budget | `ABSTAIN` | Robot đứng yên, log reason |
| Cả hai risk dưới threshold | `EXECUTE` | Safety layer vẫn kiểm collision/workspace |

Budget khởi điểm:

- tối đa 2 `REOBSERVE`;
- tối đa 1 `REPROMPT`;
- tối đa 1 `ASK_USER`;
- tối đa 1 `REGRASP`;
- collision/safety violation → abort ngay;
- hết budget và risk còn cao → `ABSTAIN`.

Budget, thresholds và mapping reason→action phải khóa trước locked test.

---

## 13. Baseline và ablation

| Mã | Cấu hình | Câu hỏi |
|---|---|---|
| B0 | RoboRefer RGB-only, forced point, shadow always-execute | Baseline tối thiểu |
| B1 | RoboRefer RGB-D gốc, forced point | Depth gốc giúp gì? |
| B1-CF | B1 với flat/shuffled/inverted depth | Model có thực sự dùng depth? |
| B2 | B1 + hard depth-component gate | Hard gate giảm risk hay chỉ giảm coverage? |
| P0 | P-CRA-U heatmap, không calibration | Probabilistic head có tín hiệu không? |
| P1 | P-CRA-U + calibrated risk + adaptive intervention | Phương pháp chính |
| P1-G | P1 bỏ geometric evidence | Learned uncertainty đóng góp gì? |
| P1-U | Geometry/policy không P-CRA-U | P-CRA-U đóng góp gì? |
| P2 | P1 + P-CRA-F residual fusion | Fusion mới có cải thiện không? |
| P2-R | P2 không relation conditioning | Relation query đóng góp gì? |
| P2-L | P2 + LoRA encoder | Encoder adaptation có cần thiết không? |

Ablation tối thiểu phải tách:

- lợi ích depth input;
- lợi ích probabilistic output;
- lợi ích calibration;
- lợi ích intervention;
- lợi ích relation-conditioned fusion;
- lợi ích LoRA.

---

## 14. Evaluation protocol và metrics

### 14.1. Perception

- exact output/parse rate;
- point-in-target-mask;
- point-in-interior-mask, không gọi là physical safety;
- target instance accuracy;
- PCK và 2D distance;
- relation accuracy theo task type;
- heatmap NLL/mass-in-target;
- top-k target recall;
- 3D localization error;
- latency và VRAM.

### 14.2. Uncertainty/calibration

- Brier, ECE, reliability diagram;
- AUROC/AUPRC cho wrong/answerable detection;
- risk–coverage và AURC;
- empirical conformal coverage/gap;
- accepted-incorrect risk;
- false accept/false reject;
- metrics theo semantic/relation/depth/occlusion/OOD subgroup.

### 14.3. Intervention

- intervention rate;
- success sau từng intervention type;
- unnecessary intervention rate;
- average retry và added latency;
- recovery success;
- budget exhaustion rate.

### 14.4. Robot

- correct-target handoff;
- IK/planning success;
- collision clearance;
- grasp/contact/attach/lift;
- place/detach;
- post-action verification;
- wrong-object execution;
- safe abstention;
- end-to-end completion time/success.

Không gộp successful completion, safe abstention, false rejection và wrong execution thành một success rate duy nhất.

### 14.5. Statistical protocol

- paired comparison trên cùng scene/prompt/view;
- confidence intervals cho rates;
- McNemar hoặc paired bootstrap khi phù hợp;
- report per-family và per-task, không chỉ micro-average;
- không tuyên bố significance từ pilot 10 cảnh.

---

## 15. Protocol chống leakage và provenance

### 15.1. Trước mỗi locked run

Khóa SHA-256 của:

- scene/data generator configs;
- split manifest;
- prompts;
- model/checkpoint;
- P-CRA weights;
- calibrator;
- gate/policy thresholds;
- evaluator annotations;
- code commit/source inventory.

### 15.2. Trong inference

Không được đọc:

- Gazebo target pose/ID;
- semantic label/mask;
- target bbox;
- evaluator relation answer;
- expected intervention.

### 15.3. Sau prediction lock

Evaluator mới được mở oracle để:

- xác định target/instance hit;
- gắn failure label;
- tính calibration/action metrics;
- đánh giá post-action result.

Mọi failed infrastructure run được giữ riêng và không chuyển thành model failure hoặc loại bỏ âm thầm.

---

## 16. Failure taxonomy

| Mã | Lỗi |
|---|---|
| S1 | Sai semantic class |
| S2 | Đúng class nhưng sai instance |
| S3 | Ambiguous nhưng vẫn execute |
| S4 | Target absent nhưng vẫn execute |
| R1 | Sai relation |
| R2 | Sai/mất anchor hoặc multi-reference |
| P1 | Point ngoài target |
| P2 | Point trúng target nhưng sát biên |
| D1 | Depth invalid/noisy/bias |
| D2 | Component dính nền/vật khác |
| D3 | Border truncation/partial view |
| G1 | 3D center/dimensions/surface sai |
| T1 | Intrinsics/TF/time sync/calibration sai |
| U1 | Raw risk không tách được đúng/sai |
| U2 | Calibration sai hoặc overconfident |
| A1 | IK/planning/reachability/collision fail |
| A2 | Grasp/attach/lift fail |
| A3 | Place/detach/verification fail |
| I1 | Chọn sai intervention |
| I2 | Intervention đúng nhưng không phục hồi |
| X1 | Infrastructure/runtime failure |

Failure taxonomy quyết định sửa layer nào; không dùng một adapter cho mọi lỗi.

---

## 17. Kế hoạch triển khai theo work package

### WP0 — Khóa baseline và terminology

**Công việc**

- giữ bất biến pilot v0;
- đổi cách gọi `safe hit` thành `eroded-mask hit` trong tài liệu mới;
- chuẩn hóa model/P-CRA/geometry/decision/robot outputs;
- bổ sung reason codes chi tiết.

**Đầu ra**

- protocol v1 draft;
- schema và failure taxonomy v2;
- baseline report có provenance.

### WP1 — Depth sensitivity diagnostic

Trên cùng RGB và prompt, chạy:

- RGB-only;
- correct registered relative depth;
- flat depth;
- shuffled depth khác scene;
- inverted near/far;
- localized holes/edge corruption.

Đo point displacement, instance switch, heatmap/answer changes và latency theo task type.

**Đầu ra:** quyết định go/no-go P-CRA-F.

### WP2 — Dataset v1

- triển khai family generator;
- lưu metric/relative depth và masks tách biệt;
- counterfactual/answerability labels;
- tile mapping;
- split validator chống family leakage;
- capture QC và immutable manifests.

**Gate:** 50-family prototype replay/validate 100% trước khi scale.

### WP3 — P-CRA-U prototype

- feature hooks trước projector;
- contextual multi-anchor query extractor;
- aligned tile fusion;
- target/anchor heatmap;
- answerability/source heads;
- unit tests tensor shape, alignment và deterministic cache.

**Gate:** location mass/answerability có tín hiệu trên dev và không cần oracle khi inference.

### WP4 — Calibration

- train wrong/error predictor;
- fit calibrator trên split riêng;
- reliability/risk–coverage report;
- freeze thresholds;
- compare B2 với P1 offline.

### WP5 — Geometry v2 và intervention

- table-plane/component improvements;
- reason-aware policy;
- reobserve poses;
- structured reprompt/ask schema;
- replay state transitions và retry budget.

### WP6 — Closed-loop UR3

- verified target handoff;
- plan/execute;
- attach/lift/place/detach verification;
- recovery/regrasp;
- negative controls không chuyển động.

### WP7 — P-CRA-F tùy điều kiện

- residual fusion;
- optional last-two-block LoRA;
- ablations và resource report;
- chỉ giữ nếu qua gate.

### WP8 — Locked evaluation và luận văn

- freeze toàn hệ thống;
- Test-IID/OOD;
- Gazebo và camera/UR3 thật khi an toàn;
- thống kê, limitations, artifacts và video.

---

## 18. Lịch 16 tuần cập nhật

| Tuần | Công việc | Mốc |
|---:|---|---|
| 1 | Khóa plan v2, schema, terminology, pilot evidence | WP0 hoàn thành |
| 2 | Depth counterfactual runner và task set | WP1 chạy được |
| 3 | Phân tích depth sensitivity, khóa P-CRA-F gate | Go/no-go sơ bộ |
| 4 | Family generator và 50-family prototype | Dataset schema pass |
| 5 | Target/anchor/interior/grasp labels + split validator | Data QC pass |
| 6 | Feature hooks/cache/tile mapping | Tensor contract pass |
| 7 | Relation extractor + heatmap head | P-CRA-U smoke test |
| 8 | Answerability/source heads | Development report |
| 9 | Wrong-object head và geometric evidence model | Error predictor pass |
| 10 | Calibration/conformal và threshold freeze | Calibration gate |
| 11 | Gate v2 + reason-aware policy | Offline P1 vs B2 |
| 12 | REOBSERVE/REPROMPT/ASK/ABSTAIN | Agent replay pass |
| 13 | Closed-loop Gazebo smoke tests | WP6 smoke pass |
| 14 | P-CRA-F nếu gate mở; nếu không scale/evaluate P1 | Method freeze |
| 15 | Locked Test-IID/OOD và paired robot trials | Final metrics |
| 16 | Luận văn, figures, artifacts, demo dự phòng | Hoàn thiện |

Nếu P-CRA-F mở, scope robot/intervention không được bỏ. Nếu thiếu thời gian, bỏ P-CRA-F trước vì calibrated intervention và closed-loop verification bám sát tên đề tài hơn.

---

## 19. Việc cần làm ngay trong hai tuần tới

### P0 — Không fine-tune ngay

1. Khóa plan v2 và protocol depth sensitivity.
2. Tạo test families depth-dependent đủ kiểm soát.
3. Chạy correct/flat/shuffled/inverted depth trên cùng RGB/prompt.
4. Bổ sung gate reason codes và component diagnostics.
5. Chốt data schema tách target/interior/graspable/reachable.
6. Xây 50-family dataset prototype có positive, ambiguous và absent.
7. Viết split validator theo `family_id`.

### P1 — Chuẩn bị P-CRA-U

8. Hook `R0,D0` trước projector và xác minh shape/alignment.
9. Ghi tile mapping về ảnh 640×480.
10. Thiết kế multi-anchor query slots và supervision.
11. Chạy feature cache memory/storage benchmark.
12. Implement location-mass loss và answerability head tối thiểu.

### Chưa làm

- chưa bật full tower fine-tuning;
- chưa bật LoRA broad selector;
- chưa gọi raw score là probability;
- chưa dùng pilot v0 để chọn threshold cuối;
- chưa mở locked test;
- chưa publish target sang robot từ phương pháp chưa hiệu chuẩn.

---

## 20. Go/no-go và tiêu chí hoàn thành

### 20.1. Gate P-CRA-U prototype

- feature cache deterministic và aligned;
- heatmap mass-in-target tốt hơn uniform/trivial baseline;
- answerability/error detection tốt hơn geometry-only baseline trên dev;
- không đọc evaluator oracle khi inference;
- inference latency/VRAM được ghi đầy đủ.

### 20.2. Gate selective execution trước robot

- point/instance metrics trên positive held-out đạt ngưỡng pre-register;
- accepted positive correctness mục tiêu ≥95%;
- false reject mục tiêu ≤10% ở coverage đã chọn;
- tối thiểu 60 negative locked scenes và mục tiêu 0 false accept trước manipulation;
- calibration report hợp lệ;
- five consecutive full infrastructure runs không timeout/crash;
- surface, clearance, IK và collision checks pass.

Nếu không đạt, hệ thống chỉ chạy shadow evaluation.

### 20.3. Mức tối thiểu để bảo vệ

- baseline B0/B1/B2 tái lập;
- P-CRA-U hoặc uncertainty estimator có calibration hợp lệ;
- Agent có `EXECUTE`, `REOBSERVE`, `ABSTAIN`;
- closed-loop verification;
- locked positive/negative evaluation;
- false accept, false reject, risk–coverage và end-to-end result được báo riêng.

### 20.4. Mức đầy đủ

- thêm `REPROMPT`, `ASK_USER`, `REPLAN`, `REGRASP`;
- Test-IID/OOD và camera thật;
- P-CRA-F/LoRA nếu gate mở;
- ablation fusion, uncertainty, geometry và intervention;
- paired Gazebo/UR3 evaluation.

### 20.5. Không bắt buộc

- RoboRefer-RFT nếu checkpoint chưa có;
- P-CRA-F nếu diagnostic không ủng hộ;
- RoboPoint, GroundingDINO hoặc SAM2 trong core method;
- mọi trial đều thành công.

Negative result có protocol sạch vẫn là kết quả nghiên cứu hợp lệ.

---

## 21. Rủi ro và phương án

| Rủi ro | Hệ quả | Phương án |
|---|---|---|
| P-CRA scope quá lớn | Không hoàn thành Agent/robot | Làm P-CRA-U trước, P-CRA-F có gate |
| Model ít dùng depth | B1 tốn latency nhưng ít lợi ích | Counterfactual audit, P-CRA-F hoặc bỏ depth ở task không cần |
| Point đúng, component sai | False reject/geometry error | Gate v2, plane removal, reobserve |
| Ambiguous/absent overconfidence | Wrong-object execution | Answerability, calibration, ask/abstain |
| Source labels bị hiểu là causal | Claim quá mức | Multi-label evidence, counterfactual validation |
| Heatmap đa mode bị Gaussian hóa | Point trung bình sai | Top-k modes và local covariance |
| Relative depth dùng như metric | XYZ/covariance sai | Metric depth sidecar + CameraInfo/TF |
| Reachability trộn với semantics | Model học sai target | Tách grounding/action masks và losses |
| Dynamic tile mapping sai | Heatmap coordinate sai | Lưu mapping, alignment tests |
| LoRA selector quá rộng | VRAM/regression | Explicit last-block filter |
| Calibration overfit | Risk không đáng tin | Family split, calibration độc lập, locked test |
| GPU 16 GiB OOM | Training dừng | Feature cache, frozen backbone, batch nhỏ, accumulation |
| Infrastructure timeout/crash | Nhầm lỗi model | Tách X1, five-run stability gate |
| Sim-to-real gap | Coverage không chuyển miền | Noise/OOD, camera-real recalibration |

---

## 22. Kịch bản demo cuối

### Demo A — Execute có kiểm soát

1. Câu lệnh có relation rõ.
2. RoboRefer và P-CRA thống nhất target.
3. GUI hiển thị heatmap, top modes, geometry và calibrated risks.
4. Agent chọn `EXECUTE`.
5. UR3 pick/place.
6. Verifier xác nhận `DONE`.

### Demo B — Reobserve phục hồi geometry

1. Mask chạm biên hoặc component dính nền.
2. Source evidence nghiêng về depth/occlusion.
3. Agent chọn `REOBSERVE` và đổi eye-in-hand pose.
4. Geometry/risk được cập nhật.
5. Risk thấp → execute; vẫn cao → abstain.

### Demo C — Ambiguous target

1. Hai instance cùng thỏa instruction.
2. Heatmap có nhiều mode và `p_answerable` thấp.
3. Agent chọn `ASK_USER`.
4. Instruction mới phân biệt instance.
5. Ground/assess lại rồi mới hành động.

### Demo D — Target absent

1. Target không có trong FOV.
2. P-CRA/calibrator trả absent/high risk.
3. Không tạo verified handoff.
4. Robot đứng yên và log `ABSTAIN`.

### Demo E — Action recovery

1. Grounding đúng nhưng attach/lift verification fail.
2. Agent không gọi lỗi này là semantic failure.
3. Reobserve/replan/regrasp trong budget.
4. Kết thúc `DONE` hoặc `ABORT` trung thực.

Demo phải có success, recovery và safe refusal; không chỉ chọn một video đẹp.

---

## 23. Phát biểu đóng góp trong luận văn

### Phát biểu dự kiến

> Đồ án đề xuất một LLM/VLM Agent vòng kín cho thao tác UR3, trong đó RoboRefer cung cấp point estimate, P-CRA-U bổ sung phân phối spatial, answerability và uncertainty evidence có điều kiện theo quan hệ, còn tầng calibration chuyển evidence thành grounding/action risk để chọn can thiệp thích ứng và xác minh thao tác.

Nếu P-CRA-F qua gate:

> Phương pháp còn bổ sung residual relation-conditioned RGB-D fusion bảo toàn hai media streams của RoboRefer và được đánh giá riêng bằng ablation trên các tác vụ depth-dependent và multi-reference.

### Không được phát biểu

- RoboRefer xuất trực tiếp XYZ metric hoàn chỉnh.
- Attention/gate bằng 1 nghĩa là model chắc chắn 100%.
- Một covariance Gaussian mô tả đầy đủ ambiguity đa mode.
- P-CRA tự sửa mọi lỗi segmentation, TF, planning và grasp.
- Safe hit 4 px nghĩa là robot-safe.
- Một pilot 10 cảnh chứng minh phương pháp tổng quát.
- Threshold chỉnh sau khi xem test vẫn là đánh giá độc lập.

---

## 24. Artifact cuối cùng

- plan/protocol và pretrial registrations;
- dataset schema, family split và validation report;
- baseline B0/B1/B1-CF/B2 outputs;
- P-CRA-U weights/config và P-CRA-F nếu có;
- calibrator và thresholds;
- per-episode provenance/feature/risk/decision logs;
- heatmap, modes, mask, 3D covariance và intervention panels;
- robot event/verification logs;
- Test-IID/OOD metrics và confidence intervals;
- failure taxonomy report;
- code/model/config hashes;
- RESULT.md ghi đầy đủ PASS, FAIL, abstention và infrastructure errors;
- demo video có execute, recovery và refusal.

---

## 25. Câu chốt

Đóng góp trung tâm của đồ án không phải là tạo thêm một confidence number hoặc ép RoboRefer luôn trả point chính xác hơn. Đóng góp cần chứng minh là:

> Một cơ chế biến point estimate và bằng chứng RGB-D thành phân phối spatial, rủi ro đã hiệu chuẩn và quyết định can thiệp có cấu trúc; sau đó sử dụng xác minh vòng kín để UR3 chỉ hành động khi đủ bằng chứng, biết thu thập thêm quan sát khi có thể phục hồi và dừng an toàn khi không thể giải quyết bất định.

P-CRA-U là con đường ưu tiên để hiện thực hóa phần perception–uncertainty. P-CRA-F là nhánh nâng cao có điều kiện. Calibration, adaptive intervention và closed-loop verification vẫn là trục xuyên suốt gắn trực tiếp nhất với tên đề tài.
