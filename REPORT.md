# Báo cáo Day 5 — điền trực tiếp trong fork của bạn


- Mã học viên theo lớp: 2A202602131
- Ngày / CVAT local: 2026-09-17 / CVAT local http://localhost:8080 (v2.74.1)
- Công cụ đã dùng: CVAT REST API do AI agent điều khiển; gợi ý tự động YOLO11-seg + SegFormer-B5 Cityscapes (nuclio); hậu xử lý bằng script (map class, lấp lỗ, trừ things khỏi stuff, polygon sửa theo tọa độ quan sát trên overlay); QC bằng overlay + `scripts/inspect_submissions.py`. Người học tự sửa `medium_instance` và `hard_panoptic` trên CVAT sau khi tự chấm. Không dùng SAM.

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh VLearn đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `submissions/medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `submissions/hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `submissions/cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `submissions/cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `submissions/cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `submissions/cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `submissions/cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `submissions/cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Nếu export lỗi, ghi task, dữ liệu đã Save đến đâu và lỗi đã báo coach.

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: không có object tự vẽ; toàn bộ mask Medium do model YOLO11-seg tạo và agent hậu xử lý (người học chọn không vẽ tay).
- Class và quy tắc tôi dùng để chọn biên: biên theo phần nhìn thấy của mask model; lỗ kín bên trong mỗi instance được lấp (kính/khe nằm trong mask); hai mask cùng class IoU > 0.7 coi là trùng, giữ confidence cao hơn; ngưỡng confidence 0.3.
- Nếu dùng gợi ý sau đó: `000000181542.jpg` — người lái xe máy phía trước bên trái (x≈109–212, y≈111–385) có 2 mask person (#10 0.70 và #12 0.59) trùng nhau → giữ #10, bỏ #12 để một người chỉ có một mask; các xe máy/người khác giữ đề xuất vì biên khớp phần nhìn thấy trên overlay.
- Nếu không dùng gợi ý: có dùng gợi ý. Quyết định gán nhãn khác: ở Hard, pixel của things được trừ khỏi stuff để panoptic không chồng lấn.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `hard_panoptic` / `000000460147.jpg` / xe van tối màu sát mép trái (x≈0–91, y≈235–290) và xe van đen trên tầng xe chở ô tô (x≈344–417, y≈216–290).
- Lỗi thuộc loại: sai lớp / thiếu-thừa vật / gộp-tách / biên / phủ vùng / khác: sai lớp, kèm thiếu vật nhỏ.
- Bằng chứng tôi nhìn thấy: hai xe van thân hộp bị gán `car`; cuối đường (x≈350–420, y≈125–175) sót 2 xe buýt, 3 xe máy, 3 người nhỏ. Ở `000000350023.jpg` có mảng `sidewalk` rải rác không phải vỉa hè.
- Quy tắc và hành động sửa: van chở hàng thân hộp → `truck`, mỗi xe một mask; thêm vật nhỏ nhìn thấy rõ; bỏ mảng `sidewalk` sai. Ở `medium_instance`, thêm người/xe bị sót ở `000000373353.jpg` và `000000458325.jpg`, bỏ mask trùng.
- Sau sửa đã Save và export lại chưa? Đã sửa trên CVAT, Save và export lại `COCO 1.0` cho `hard_panoptic` và `medium_instance`.


## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `easy_semantic/7ee6d192-89e2408b.jpg` — sườn đồi cỏ khô hai bên cao tốc (y≈300–510) | (a) `vegetation`; (b) không thuộc 5 class (terrain) | SegFormer ra `terrain`; classes.json dùng trainId Cityscapes, nơi terrain tách khỏi vegetation | Để trống. Hỏi coach: cỏ khô/đất trống có tính vào `vegetation` không? |
| 2. `cp5_occlusion/000000336232.jpg` — sedan xám-xanh sau đầu người lái xe máy (x≈90–198, y≈123–181) | (a) hai mảng rời là 2 xe; (b) 1 xe bị che | Cùng màu, thẳng hàng, đèn pha ở mảng phải, đuôi ở mảng trái; model có #17 phủ cả 2 mảng và #15 chỉ mảng phải | Một instance: giữ #17, bỏ #15; không vẽ xuyên phần bị che |
| 3. `hard_panoptic/000000460147.jpg` — xe van tối màu sát mép trái (x≈0–91, y≈235–290) | (a) `car` vì cỡ xe con chở khách; (b) `truck` vì thân hộp chở hàng | YOLO lưỡng lự truck 0.45 / car 0.40; xe có thân hộp chở hàng | Đổi sang `truck`. Hỏi coach: van chở hàng cỡ nhỏ tính `truck` hay `car`? |
