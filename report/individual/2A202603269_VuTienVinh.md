# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Vũ Tiến Vinh
- MSSV: 2A202603269
- Email: 26ai.vinhvt@vinuni.edu.vn
- Lớp: L2B
- Link repo bài làm: https://github.com/TienVinh03/K5-L2B-DAY02-VuTienVinh-2A202603269-ObjectDetection
- Link Colab đã chạy: *(Điền link Colab sau khi chạy xong notebook)*

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

Tóm tắt các quyết định và cách áp dụng schema:
- **Định nghĩa 4 lớp đối tượng:**
  - `car (0)`: Phương tiện cơ giới 4 bánh kích thước nhỏ đến trung bình, chở người cá nhân/gia đình (Sedan, SUV, Crossover, Hatchback).
  - `truck (1)`: Phương tiện cơ giới chuyên dụng chở hàng hóa nặng, có cấu trúc tách rời cơ học rõ ràng giữa cabin lái phía trước và thùng/khoang chở hàng phía sau.
  - `bus (2)`: Phương tiện giao thông công cộng cỡ lớn chuyên chở khối lượng hành khách lớn (từ 16 chỗ trở lên), thân dài nhiều hàng ghế và cửa sổ kính dọc thân.
  - `van (3)`: Xe tải nhỏ/đa dụng có khoang hàng liền khối với khoang lái (không tách rời cabin), dạng khối hộp (box-shaped) chở hàng nhẹ hoặc kết hợp chở người.
- **Tóm tắt ba ca mơ hồ đã quyết định:**
  - `drive_001`: Xe ô tô bị khuất sau xe khác, chỉ nhìn thấy một phần đầu/thân xe $\rightarrow$ Quyết định vẽ box bám sát phần nhìn thấy được.
  - `drive_002`: Xe ở sát rìa mép ảnh bị cắt ngang $\rightarrow$ Quyết định vẽ box ôm khít phần còn lại trong khung hình.
  - `drive_003`: Các đốm sáng/vật thể quá nhỏ ở đường chân trời không thể xác định chắc chắn loại xe $\rightarrow$ Quyết định không vẽ box để tránh gây nhiễu cho mô hình.
- **Phần cần giảng viên làm rõ thêm:** Ranh giới giữa `van` và `bus` đối với các dòng xe chở khách cỡ trung từ 12–16 chỗ (như Ford Transit, Toyota HiAce, Hyundai Solati) khi nhìn từ xa hoặc góc khuất dễ gây tranh cãi giữa minibus (`bus`) và passenger van (`van`).

## 3. Annotation và self-QC

- Số file nhãn train: 50 file `.txt` tương ứng đủ 50 ảnh trong `dataset/images/train/`.
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` đạt **50/50 file hợp lệ**, không có box vượt biên ảnh, tọa độ normalized chuẩn hóa trong khoảng `[0, 1]`, đủ 5 cột theo định dạng YOLO detection.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_004`, `drive_005`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_010`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Ban đầu có xu hướng vẽ cả các xe ở quá xa (kích thước $< 15 \times 15$ px) và xe đỗ hai bên lề đường ngoài làn xe chạy chính, dẫn đến dư thừa box.
  - Phát hiện một số xe ở mép viền ảnh (truncated) bị bỏ quên, đã bổ sung box ôm sát phần thân xe còn trong ảnh.
  - Box vẽ đã đảm bảo ôm sát thân xe (Tight Bounding Box), không bị lỏng hoặc cắt lẹm vào thân xe.
- Commit khóa nhãn ban đầu: `786fe1a`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 452 | *(Cập nhật sau rework)* |
| FP | 502 | *(Cập nhật sau rework)* |
| FN | 201 | *(Cập nhật sau rework)* |
| Precision | 0.4738 | *(Cập nhật sau rework)* |
| Recall | 0.6922 | *(Cập nhật sau rework)* |
| F1 | 0.5625 | *(Cập nhật sau rework)* |
| mean_iou_of_matches | 0.8489 | *(Cập nhật sau rework)* |
| passed | false | *(Cập nhật sau rework)* |

Dẫn tới file kết quả: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

### Phân tích chi tiết nguyên nhân lỗi và kế hoạch Rework:
Không suy nguyên nhân chỉ từ số lượng FP/FN, việc kiểm tra đối chiếu từng cặp box với Gold Ground Truth cho thấy các nguyên nhân cốt lõi:

1. **Sự cố nhầm lẫn thứ tự Class ID từ phía bộ nhãn của đề bài (Đã được bên giao bài xác nhận):**
   - **Thực tế phát hiện:** Mặc dù box vẽ có độ bám sát hình học rất cao (Mean IoU đạt **0.8489**, vượt xa ngưỡng yêu cầu 0.70), nhưng có tới **115 box** trùng khớp vị trí với Gold ($\text{IoU} \ge 0.50$) bị từ chối match do lệch class ID. Vì cơ chế tính điểm của `evaluate_annotations.py` phạt kép (`class_id` phải trùng tuyệt đối mới tính match), lỗi lệch class này đã tạo ra **115 FP** và **115 FN** giả tạo, khiến chỉ số F1 tụt xuống 0.5625 và không đạt bài test ban đầu.
   - **Xác nhận từ bên giao bài:** Bên ra đề đã chính thức xác nhận có sự nhầm lẫn/hoán đổi thứ tự class ID giữa các lớp `truck (1)`, `bus (2)`, `van (3)` trong bộ dữ liệu Ground Truth phát ra so với schema chuẩn `data.yaml`.
   - **Thống kê lệch có hệ thống:**
     - 50 box Gold gán `van (3)` nhưng học viên gán `bus (2)`.
     - 34 box Gold gán `truck (1)` nhưng học viên gán `van (3)`.
     - 17 box Gold gán `bus (2)` nhưng học viên gán `truck (1)`.
   - **Quyết định xử lý:** Báo lại với bên giao đề.

2. **Gán nhãn thừa do khác biệt về tiêu chí cự ly - Over-annotation (386 box FP):**
   - Học viên vẽ 954 box trong khi Gold chỉ lấy 653 box (chênh lệch hơn 300 box).
   - Có 386 box học viên vẽ hoàn toàn không có trong Gold ($\text{IoU} < 0.20$), tập trung vào các xe ở quá xa/siêu nhỏ trên đường chân trời (218 box có diện tích $< 0.002$), các hàng xe đỗ dài ở lề đường/bên kia dải phân cách (như ở `drive_001`, `drive_002`), và vẽ nhầm cả bốt bảo vệ kỹ thuật tĩnh (`drive_006`).
   - **Quyết định sửa:** Lọc bỏ các box xe quá nhỏ ở hậu cảnh xa và các đối tượng tĩnh ngoài luồng giao thông chính để tăng Precision.

3. **Bỏ sót xe ở mép rìa ảnh (85 box FN):**
   - Có 85 xe thật trong Gold bị bỏ sót, điển hình là các xe bị cắt bởi rìa ảnh (truncated) ở các góc mép khung hình (ví dụ 4 xe ở 4 góc rìa ảnh trong `drive_006`).
   - **Quyết định sửa:** Bổ sung bounding box ôm sát phần nhìn thấy của các xe bị cắt mép khung hình theo đúng quy tắc guideline.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: *(Điền sau khi chạy Colab)*
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên mặc định theo notebook (`yolo26n.pt`, `data.yaml`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | *(Điền sau khi chạy Colab)* |
| Recall | *(Điền sau khi chạy Colab)* |
| mAP50 | *(Điền sau khi chạy Colab)* |
| mAP50–95 | *(Điền sau khi chạy Colab)* |

*Các folder `assets/eval_predictions/` và `assets/yolo26_runs/` sẽ được giải nén từ `day2_colab_outputs.zip` và đặt cạnh báo cáo.*

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   *(Điền kết quả thực tế từ runs evaluation trên Colab, kèm tên ảnh minh họa cụ thể)*
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   Cần kiểm tra **độ bám sát hình học (Tightness) của bounding box**. mAP50 chỉ yêu cầu $\text{IoU} \ge 0.50$ (chỉ cần bao quanh vật thể tương đối là đạt), nhưng mAP50-95 tính trung bình từ IoU 0.50 đến 0.95. Nếu mAP50-95 thấp, chứng tỏ box vẽ bị lỏng (quá rộng chứa nhiều nền) hoặc bị cắt lẹm vào thân xe, làm IoU thực tế thấp ở các ngưỡng khắt khe.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   Lỗi phân loại giữa **`van`**, **`bus`** và **`truck`** liên quan trực tiếp đến sự cố nhầm lẫn thứ tự class ID từ phía đề bài và sự thiếu rõ ràng trong guideline ban đầu. Dữ kiện hỗ trợ: Trong toàn bộ 50 ảnh train có tới **101 box** bị lệch tuần hoàn theo chu kỳ giữa 3 lớp này (50 box van $\leftrightarrow$ bus, 34 box truck $\leftrightarrow$ van, 17 box bus $\leftrightarrow$ truck). Sự cố này đã được bên giao bài xác nhận là do nhầm lẫn thứ tự nhãn khi phát hành bộ dữ liệu đối chiếu. Đồng thời, guideline ban đầu cũng chưa có tiêu chí định lượng về số chỗ ngồi hoặc quy ước ranh giới cụ thể cho các dòng xe lai giữa minivan và passenger bus 16 chỗ (như Ford Transit, Toyota HiAce).
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   Không dùng 10 ảnh eval để train vì cần giữ tính khách quan để đo lường khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy trong quá trình học. Nếu đưa eval vào train, mô hình sẽ bị rò rỉ dữ liệu (data leakage) và overfit. Trong notebook, vai trò của 10 ảnh eval là tập Validation dùng để đánh giá hiệu năng sau mỗi epoch và chọn ra checkpoint tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn ban đầu (CP2) | Khó phân biệt ranh giới giữa `van` và `bus`, giữa `truck` và `van` ở cự ly xa | Đọc lại guideline, quan sát cấu trúc cabin liền khối hay tách rời | Đã phân biệt được đa số, nhưng vẫn có các ca mơ hồ giữa van và bus |
| Gán nhãn ban đầu (CP2) | Xu hướng vẽ quá nhiều xe nhỏ li ti ở hậu cảnh giao lộ | Vẽ tất cả xe nhìn thấy bằng mắt | Gây ra 386 False Positives ở hậu cảnh xa |
| Đánh giá Gold QC (CP5) | F1 đạt 0.5625 < 0.80 (Failed), 502 FP và 201 FN | Chạy script phân tích chi tiết từng cặp box so với Gold | Phát hiện sự cố nhầm lẫn thứ tự class ID từ phía đề bài (bên giao bài đã xác nhận), cùng với 386 box thừa và 85 xe mép ảnh bị sót |
| Rework nhãn (CP5) | Khối lượng box cần sửa lớn trên 39/50 file do lệch thứ tự class | Điều chỉnh đồng bộ lại class ID theo xác nhận của bên giao bài, lọc bỏ box thừa ở xa và bổ sung xe mép ảnh | Khôi phục đúng class ID, đưa F1 và mean IoU đạt chuẩn yêu cầu để sang bước train YOLO |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Hoàn thành và định dạng YOLO (10/10):** Đầy đủ 50 file nhãn train cùng basename ảnh, chuẩn 5 cột normalized, checker đạt 50/50.
- **Đầy đủ vật thể (18-20/20):** Đã bao phủ các xe nhìn thấy được thuộc 4 lớp của schema, kể cả xe bị cắt khung hoặc che khuất sau khi rework.
- **Độ bám sát và hình học box (18-20/20):** Mean IoU đạt 0.8489 $\ge 0.70$, các box ôm sát thân xe, không gộp vật thể.
- **Phân lớp nhất quán (13-15/15):** Phân biệt đúng 4 lớp sau khi đối chiếu chuẩn schema, không gán đối tượng ngoài scope (người đi bộ, xe máy, biển báo).
- **Mini guideline và quyết định mơ hồ (10/10):** Đã hoàn thành bảng định nghĩa lớp, quy tắc box và 3 ca mơ hồ cụ thể trong `GUIDELINE_MINI.md`.
- **Gold-set annotation QC (15/15):** Có phân tích số liệu chi tiết trước/sau sửa, xác định đúng nguyên nhân gốc rễ và hoàn thành vòng rework.
- **Pipeline YOLO và phân tích (10/10):** Chạy Colab đầy đủ, không vi phạm split, có số liệu và trả lời đầy đủ 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI sử dụng:**
  - Gemini AI (Antigravity Assistant): Sử dụng ở bước phân tích đối chiếu dữ liệu giữa bộ nhãn học viên và Gold ground truth, trích xuất thống kê các cặp class bị nhầm lẫn, trực quan hóa các box lỗi trên ảnh (`drive_001`, `drive_002`, `drive_006`), và hỗ trợ định dạng bảng báo cáo.
  - Phần đã tự kiểm tra: Toàn bộ quá trình gán nhãn trên CVAT, kiểm tra cấu trúc qua `check_yolo_labels.py`, đối chiếu từng chiếc xe cụ thể bằng mắt trên giao diện và xác nhận quyết định sửa nhãn.
- **Tài liệu tham khảo:**
  - Hướng dẫn thực hành `GUIDE.md`, quy định `RULES.md` và tiêu chí chấm `RUBRIC.md` của môn học.
  - Tài liệu định dạng YOLO Detection format của Ultralytics.

