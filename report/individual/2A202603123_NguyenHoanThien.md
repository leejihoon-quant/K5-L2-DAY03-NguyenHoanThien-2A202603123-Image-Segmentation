# Báo cáo thực hành Day 3 


- Mã học viên theo lớp: 2A202603123
- Ngày / CVAT local: 11/11/2026 / CVAT local (Docker, http://localhost:8080)
- Công cụ đã dùng:
  - **Model gốc dùng để gợi ý (pre-annotation):**
    - Vùng (semantic và vùng nền của panoptic): **SegFormer-B2**, trọng số `nvidia/segformer-b2-finetuned-cityscapes-1024-1024` (thư viện Hugging Face Transformers, huấn luyện sẵn trên Cityscapes, 19 lớp, dự đoán từng pixel).
    - Từng vật (instance, vật thể của panoptic): **YOLO11m-seg** (`yolo11m-seg.pt`, thư viện Ultralytics, huấn luyện sẵn trên COCO, ra box + lớp + mask cho từng vật; ngưỡng tin cậy 0,25, ảnh vào 1024).
    - Hậu xử lý: chuyển mask thành polygon bằng OpenCV (`findContours`, `approxPolyDP`), bỏ vùng nền quá nhỏ, với panoptic thì trừ vùng vật thể khỏi vùng nền; nạp vào CVAT qua API với nguồn `auto`.
  - **Nguồn dữ liệu annotation cuối cùng trong CVAT (ghi rõ):**
    - `cp1_holes`, `cp2_slice`, `cp5_occlusion`, `cp3_thin`, `cp4_curb`, `cp6_coverage`, `easy_semantic`, `medium_instance`, `hard_panoptic`: Phần lớn được giữ nguyên theo model gốc, sau đó người kiểm tra lại, dựa trên gợi ý, đóng góp của claude opus 5.5 và review, sửa lỗi bằng tay theo đúng guidline. 
  - Claude (trợ lý AI) được dùng làm công cụ hỗ trợ chạy model, hướng dẫn thao tác, soạn script đánh giá.
  - Ground truth chỉ dùng cục bộ để đánh giá; không nằm trong fork.

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh [LMS](https://aithucchien.vinuni.edu.vn) đã nhận diện bạn.

## 1. Bài đã nộp

Số ảnh hoàn thành lấy trực tiếp từ CVAT (ảnh có ít nhất một polygon). Tên ZIP theo quy ước `<task>.zip`; **cần xác nhận file thật trong `submissions/` trước khi nộp**. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

(Số polygon trong CVAT: easy 52, medium 79, hard 71, cp1 9, cp2 15, cp5 35, cp3 27, cp4 5, cp6 24.)

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: medium_instance, 000000458325.jpg, người đi ván trượt ở giữa ảnh, quay lưng, đứng riêng giữa đường.
- Class và quy tắc tôi dùng để chọn biên: person; polygon 30 điểm đi theo mép ngoài của đầu, áo, hai tay, quần, giày; không lấy ván trượt; không chừa viền nền.
- Nếu dùng gợi ý sau đó: IoU 0,84; model bám chân tốt hơn, mask tay bám khuỷu và bàn tay trái tốt hơn, hai mask ngang nhau.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `easy_semantic`, ảnh `81ae7cbb-6bc63a4a.jpg`, vùng bụi cây tối bên trái và sau cột biển "TOWN SPEED LIMIT 30" ở bên phải ảnh (khoảng x 1018–1170, y 387–465 trên ảnh 1280×720), polygon `vegetation` id 3977.
- Lỗi thuộc loại: biên / phủ vùng (polygon tự cắt chéo và để hở một mảng chưa gán nhãn).
- Bằng chứng tôi nhìn thấy: khi vẽ chồng polygon lên ảnh phóng to, polygon vegetation có một đoạn tự cắt nhau (điểm 1066,394 → 1089,390 → 1064,390). Giữa mép phải của vegetation (x≈1094–1102) và mép trái của polygon building (x≈1116–1130) còn một dải bụi cây tối không thuộc nhãn nào.
- Quy tắc và hành động sửa: vùng cây thì gán `vegetation`; polygon không được tự cắt; hai vùng kề nhau dùng chung biên, không để hở. Tôi thay polygon 3977 bằng polygon 28 điểm không còn đoạn tự cắt, có biên phải trùng với mép polygon building, rồi vẽ chồng lại để kiểm tra. Biển báo và cột để trống vì label set của task chỉ có 5 lớp (road, sidewalk, building, vegetation, sky).
- Sau sửa đã Save và export lại chưa? Đã lưu lên CVAT. Chưa export lại ZIP: [CHƯA ĐIỀN: tự export Segmentation mask 1.1 và ghi tên file ở mục 1].

Kết quả tự đánh giá (script cục bộ so với ground truth ba tier; **không** áp dụng cho sáu checkpoint vì không có ground truth, các trạm này ghi: chưa có điểm). Đây là số gần đúng do tôi tự tính, không phải điểm chính thức của lớp.
| Tier | Metric  | Bản trong CVAT (đã chỉnh sửa) |
| --- | --- |  ---: |
| Easy (semantic) | mIoU |  0,905 |
| Medium (instance) | IoU khớp × recall |  0,865 (0,877 × 0,986) |
| Hard (panoptic) | PQ  | 0,606 |


## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic`, `81ae7cbb-6bc63a4a.jpg`, biển "TOWN SPEED LIMIT 30" và cột ở bên phải ảnh | Gán vào `building` hoặc `vegetation` ở phía sau, hoặc để trống | Label set của task chỉ có road, sidewalk, building, vegetation, sky, không có lớp biển/cột | Để trống, không gán nhãn. Hỏi coach: biển và cột nhỏ để trống có bị tính là thiếu phủ không |
| `medium_instance`, `000000373353.jpg`, taxi bị người đi bộ che một phần ở giữa dưới; `000000458325.jpg`, dãy xe đỗ che nhau hai bên đường | Vẽ riêng phần nhìn thấy của từng xe, hoặc vẽ cả hình xe kể cả phần bị che | Hướng dẫn polygon yêu cầu vẽ phần nhìn thấy và đánh dấu `occluded`; polygon model cũng chỉ lấy phần nhìn thấy | Giữ phần nhìn thấy. Hỏi coach: bài COCO này có cần bật `occluded` cho từng xe không |
| `medium_instance`, `000000458325.jpg`, đám người nhỏ ở xa giữa ảnh | Gán nhãn từng người rất nhỏ, hoặc bỏ qua / gộp thành một mảng | Model đã tạo sẵn polygon cho từng người; mỗi người chỉ vài chục pixel và polygon nhiều chỗ dính nhau; tài liệu không nêu ngưỡng kích thước tối thiểu | Giữ nguyên polygon của model, không vẽ lại. Hỏi coach: có ngưỡng kích thước để bỏ qua người ở xa không |
| `medium_instance`, `000000458325.jpg`, ván trượt dưới chân người ở giữa ảnh | Tính ván trượt là một phần của `person` (người đang đứng trên nó), hoặc loại ra vì là vật khác | Label set của task có person, bicycle, car, motorcycle, bus, truck, không có lớp ván trượt; một object chỉ nên chứa một lớp | Loại ván trượt khỏi polygon person khi vẽ lại, và không gán nhãn cho ván. Hỏi coach: ván trượt có tính là vật cần gán nhãn trong bài COCO này không |
| `cp2_slice`, `000000017627.jpg` (khoảng x≈30, y≈232) và `cp5_occlusion`, `000000336232.jpg` (khoảng x≈477, y≈47) | Một vật hay hai vật: các polygon `car`/`truck` (cp2) và `bus`/`truck` (cp5) chồng lên cùng một vị trí, có thể là một vật bị model nhận hai lớp, hoặc hai vật thật | Hai polygon khác lớp nằm cùng chỗ thì ít khả năng là hai vật khác nhau; cần nhìn ảnh để xác nhận | [BẠN TỰ ĐIỀN SAU KHI MỞ ẢNH: giữ lớp đúng, xóa polygon thừa, hay giữ cả hai và vì sao] |
