# Báo cáo thực hành Day 3 — Đinh Hữu Minh (`2A202603129`)

**Cách dùng:** Đã điền theo bài làm thực tế trên CVAT và các file đã kiểm tra trong `submissions/`. Kiểm tra lại trước khi push fork và nộp link trên [LMS](https://aithucchien.vinuni.edu.vn).

- Mã học viên theo lớp: 2A202603129
- Ngày / CVAT local: 11–12/11/2026 — kết hợp CVAT Online (`app.cvat.ai`) và CVAT local (`http://localhost:8090`)
- Công cụ đã dùng: SAM 3 (trên CVAT Online ở các task đầu trước khi gặp lỗi quota/plugin), SAM 2.1 Small (Nuclio trên CVAT local), Polygon, Brush và Eraser

## 1. Bài đã nộp

Toàn bộ 9/9 task đã hoàn thành gán nhãn, Save, export từ CVAT vào `submissions/` và kiểm tra cấu trúc bằng `scripts/inspect_submissions.py --dir submissions` đạt **9/9 `[OK]`** (không sửa tay JSON/PNG bên trong ZIP). Cột điểm bên dưới giữ nguyên điểm tối đa của đề.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh `000000181542.jpg` (`medium_instance`), đối tượng `car 26` (ô tô ở rìa trái phía sau hai người đi xe máy và phía trước xe buýt).
- Class và quy tắc tôi dùng để chọn biên: Class `car`. Chọn vẽ thủ công đối tượng này trước vì đường biên thân xe khá thẳng và ít đoạn ngoặt, dễ dàng tô sát phần nhìn thấy và dừng đúng ranh giới bị người đi xe máy phía trước che khuất (không tự đoán phần bị che).
- Nếu dùng gợi ý sau đó: vùng gợi ý sai/đúng, hành động sửa/giữ và lý do: Đối với `motorcycle 30` trong cùng ảnh `000000181542.jpg`, mask gợi ý của AI tô lấp luôn phần khoảng trống giữa bánh xe; tôi đã xóa bớt mask ở giữa các nan hoa bánh xe máy vì nan xe `motorcycle 30` khá thưa, chỉ giữ phần lốp, nan hoa và thân xe nhìn thấy.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `cp5_occlusion` — ảnh `000000336232.jpg` (các đối tượng `car` trên phố dốc).
- Lỗi thuộc loại: sai lớp / thiếu-thừa vật / gộp-tách / biên / phủ vùng / khác: **Thừa vật do trùng lặp instance + export sai định dạng** (bản `cp5_occlusion.zip` ban đầu xuất nhầm `Segmentation mask 1.1`, sau khi xuất sang `COCO 1.0` lại bị lặp nhiều bản `car` y hệt nhau; đồng thời 4 task COCO khác cũng xuất nhầm `Segmentation mask 1.1` và `cp4_curb.zip` bị đặt nhầm tên thành `cp4.curb.zip`).
- Bằng chứng tôi nhìn thấy: Khi kiểm tra `cp5_occlusion.zip` lần đầu sau khi đổi sang COCO 1.0, file có `113` annotation `RLE` nhưng chỉ có `68` bản duy nhất — có `45` mask `car` bị trùng y hệt `bbox` và `area` (ví dụ `car` `[376.0, 231.0, 264.0, 196.0]` diện tích `37291.0` và `[407.0, 143.0, 129.0, 94.0]` diện tích `9280.0` đều xuất hiện 2 lần). Đồng thời `scripts/inspect_submissions.py` báo lỗi thiếu COCO JSON cho `medium_instance`, `hard_panoptic`, `cp1_holes`, `cp2_slice`, `cp5_occlusion`.
- Quy tắc và hành động sửa: Mở lại `cp5_occlusion` trong CVAT kiểm tra cột **Objects** và xóa các bản `car` trùng lặp để giữ đúng mỗi vật một instance (còn `53` annotation duy nhất: `43 car`, `2 bus`, `3 motorcycle`, `4 person`, `1 truck`, `0` trùng lặp); xuất lại chuẩn **`COCO 1.0`** từ CVAT cho cả 5 task instance/panoptic và đổi tên đúng thành `cp4_curb.zip`.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại toàn bộ; chạy `scripts/inspect_submissions.py --dir submissions` cho kết quả **9/9 `[OK]`** và không còn lỗi.

Kết quả kiểm tra cục bộ bằng `scripts/inspect_submissions.py --dir submissions`: **9/9 `[OK]`** (0 lỗi; chưa có điểm tự đánh giá `/82` từ ground truth trên hệ thống tại thời điểm ghi báo cáo).

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp6_coverage` / `7daa6479-67988f3f.jpg` — xe buýt `MTA 2555` giữa đường, xe tải `CAPITAL` bên trái và phần nắp capo xe gắn camera ở cạnh dưới | (1) Gán xe buýt, xe tải hoặc nắp capo vào lớp `car` để tăng độ phủ; (2) Để trống `background` vì không có trong `classes.json` của trạm. | `data/checkpoints/cp6_coverage/classes.json` chỉ có 7 lớp (`road`, `sidewalk`, `building`, `vegetation`, `sky`, `car`, `person`). Theo `CHECKPOINTS.md`, không tô ép vùng ngoài class của trạm hoặc chưa chắc. Trong `cp6_coverage.zip`, `background` chiếm `32.48%` (`car` `7.49%`, `building` `28.12%`, `sidewalk` `19.47%`, `road` `11.75%`, `sky` `0.68%`). | Không tô ép xe buýt `2555`, xe tải `CAPITAL` và nắp capo vào `car`; nhờ coach xác nhận các đối tượng thuộc loại phương tiện ngoài schema (`bus`/`truck`) và nắp capo xe chụp thì giữ `background` như vậy có hợp lệ không. |
| `cp1_holes` / `000000144300.jpg` — kính chắn gió trong suốt ở đầu `motorcycle` đỏ-trắng và cụm giá đỡ đỏ / khoảng trống bánh sau | (1) Khoét thủng kính chắn gió nhìn xuyên ra xe caravan phía sau và khoét khoảng hở quanh bánh sau; (2) Giữ kính chắn gió nằm kín bên trong một mask liền của `motorcycle`. | `CHECKPOINTS.md` mục `cp1_holes` yêu cầu giữ kính/khe trong mask theo quy tắc task, không khoét lỗ tùy tiện. Trong `cp1_holes.zip`, đối tượng `motorcycle` (`bbox [45.0, 68.0, 540.0, 322.0]`, `area 107094.0`) được giữ là `1` polygon liền. | Giữ phần kính chắn gió liền bên trong mask `motorcycle`; nhờ coach xác nhận thêm giá đỡ bánh sau màu đỏ (`paddock stand`) có gộp vào mask xe hay tách bỏ. |
| `cp4_curb` / `7d83710e-4697c3b2.jpg` — bồn đất trồng cây xen giữa bó vỉa với vỉa hè và phần sân bê tông đỗ 2 ô tô sau hàng rào thấp bên phải | (1) Gộp phần sân bê tông đỗ ô tô màu sáng bên phải (hoặc kéo qua bồn đất) vào `sidewalk`; (2) Chỉ gán `sidewalk` cho lối đi bộ nâng cao tính từ mép ngoài bó vỉa. | `data/checkpoints/cp4_curb/classes.json` chỉ có `road` và `sidewalk`; `CHECKPOINTS.md` yêu cầu tách ranh theo bó vỉa và chức năng, không chỉ theo màu sáng/tối của bề mặt. Trong `cp4_curb.zip`, `road` chiếm `14.32%` (`x = 0..543`) và `sidewalk` chiếm `14.52%` (`x = 101..1279`). | Tách `road`–`sidewalk` theo bó vỉa và lối đi bộ thực tế; hỏi coach xác nhận phần sân bê tông đỗ xe sau hàng rào thấp bên phải và dải đất trồng cây giáp bó vỉa có tính vào `sidewalk` hay giữ `background`. |
