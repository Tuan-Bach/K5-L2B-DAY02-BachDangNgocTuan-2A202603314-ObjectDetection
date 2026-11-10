# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Bạch Đăng Ngọc Tuấn
- MSSV: 2A202603314
- Email: bachtuan2612@gmail.com
- Lớp: L2B
- Link repo bài làm: https://github.com/Tuan-Bach/K5-L2B-DAY02-BachDangNgocTuan-2A202603314-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/Tuan-Bach/K5-L2B-DAY02-BachDangNgocTuan-2A202603314-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Xem chi tiết tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- Cách áp dụng schema (4 lớp cố định): `0: car`, `1: truck`, `2: bus`, `3: van`. Mỗi xe hợp lệ là một box riêng ôm sát phần nhìn thấy được (kể cả xe đang đỗ hoặc bị che một phần); không gán mô tô/xe máy, người đi bộ, biển báo hay xe quá nhỏ/mờ không nhận diện rõ.
- Tóm tắt ba ca mơ hồ đã quyết định:
  1. `drive_049`: Xe chỉ lộ phần đầu, không đủ đặc trưng phân biệt rõ `car` và `van` → gán `car` theo lớp cơ sở.
  2. `drive_001`: Xe bị cột biển báo che cắt, phân vân giữa `van` và `truck` → gán `van` vì thân hộp liền khối và đồng màu từ cabin ra sau.
  3. `drive_013`: Xe chuyển động có bóng mờ nhưng vẫn rõ hình khối đặc trưng → vẫn gán nhãn theo lớp tương ứng.
- Điểm cần giảng viên làm rõ thêm: Ranh giới phân biệt giữa `car` (SUV/MPV cỡ lớn, bán tải) với `van`, và giữa `van` cỡ vừa với `bus`/`truck` từ góc camera CCTV trên cao.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 file `.txt` trong `dataset/labels/train/` (cùng basename với 50 ảnh trong `dataset/images/train/`).
- Kết quả checker và lỗi đã xử lý: Khi chạy `python tools/check_yolo_labels.py`, lần đầu phát hiện 13 lỗi `box vượt ngoài biên ảnh` (do tọa độ một số box sát mép ảnh vượt nhẹ ngoài `[0, 1]` tại `drive_014`, `drive_019`, `drive_023`, `drive_024`, `drive_027`, `drive_028`, `drive_029`, `drive_032`, `drive_034`, `drive_038`, `drive_048`). Đã kẹp lại tọa độ về đúng `[0, 1]`, chạy lại `check_yolo_labels.py` đạt `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`.
- Danh sách 11 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_009`, `drive_013`, `drive_014`, `drive_024`, `drive_026`, `drive_042`, `drive_048`, `drive_049`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Ở các ảnh đầu (`drive_001`, `drive_002`, `drive_003`), có hiện tượng gán thừa nhiều box nhỏ/mờ ở hậu cảnh xa.
  - Ở một số ảnh mật độ xe cao (`drive_024`, `drive_041`), bị bỏ sót xe ở làn xa hoặc xe bị che khuất một phần.
  - Ở `drive_042`, box `bus` ban đầu bị cắt mất nửa trên thân xe và dễ nhầm ranh giới với `van`/`car`. Đã tiến hành xóa box thừa, bổ sung xe sót và căn lại box ôm khít thân xe.
- Commit khóa nhãn / cập nhật: `ccef37c` (bộ nhãn đã QC + `GUIDELINE_MINI.md` + JSON) và `ad82f38` (rework sửa lỗi nhãn `bus`/`van` ở `drive_002`, `drive_003`, `drive_024`, `drive_042`).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa (rework lần cuối) |
|---|---:|---:|
| TP | 375 | 626 |
| FP | 245 | 16 |
| FN | 278 | 27 |
| Precision | 0.6048 | 0.9751 |
| Recall | 0.5743 | 0.9587 |
| F1 | 0.5892 | 0.9668 |
| mean_iou_of_matches | 0.8742 | 0.8659 |
| passed | false | true |

File kết quả chi tiết: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

- Phân tích và quá trình rework:
  - Ở lần đánh giá đầu tiên, `mean_iou_of_matches` đạt `0.8742` (vượt ngưỡng `0.70`, cho thấy box vẽ sát vật thể), nhưng `F1` chỉ đạt `0.5892` (`FP = 245`, `FN = 278`).
  - Đối chiếu trực tiếp từng ảnh cho thấy ba nhóm nguyên nhân cụ thể: (1) gán thừa nhiều đối tượng quá nhỏ/xa ở hậu cảnh (`drive_001`: 15 vs 5, `drive_002`: 32 vs 13, `drive_003`: 34 vs 18), (2) bỏ sót xe ở các ảnh đông phương tiện (`drive_024`, `drive_027`, `drive_041`, `drive_050`), và (3) nhầm lớp hoặc thiếu phần thân box (`drive_042` vẽ 4 khớp 1; có 2 box nhiễu gán nhầm `bus` tại `drive_002`, `drive_003`, 1 box nhiễu `van` tại `drive_024`, và 1 box `bus` bị hụt nửa trên tại `drive_042` chỉ đạt IoU `0.46`).
  - Sau khi sửa nhãn và chuẩn hóa biên `[0, 1]`, `F1` tăng lên `0.9668 >= 0.80` và `mean_iou_of_matches` đạt `0.8659 >= 0.70` (`passed: true`). Tổng phân bố nhãn train sạch đạt `car: 525`, `van: 52`, `truck: 45`, `bus: 20` (đủ `20/20` nhãn `bus`).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Colab `CUDA:0 (Tesla T4, 14913MiB)`, `Python-3.13.16`, `Ultralytics 8.4.208`, `torch-2.14.1+cu130` (model `YOLO26n`: 121 layers, 2,390,464 parameters, 5.4 GFLOPs).
- Các tham số thay đổi so với notebook nếu có: Không thay đổi tham số mặc định (`yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval (113 instances) | Lần chạy trước sửa `bus`/`van` | Lần chạy sau sửa `bus`/`van` (chính thức) |
|---|---:|---:|
| Precision | 0.4286 | 0.7426 |
| Recall | 0.7719 | 0.5022 |
| mAP50 | 0.5745 | 0.6005 |
| mAP50–95 | 0.3694 | 0.3996 |

Chi tiết theo từng lớp trên 10 ảnh `eval` (phiên chạy mới nhất đã lưu trong `notebooks/day2_annotation_to_yolo.ipynb`):
- `car` (94 instances): `P = 0.928`, `R = 0.818`, `mAP50 = 0.910`, `mAP50-95 = 0.619`
- `truck` (5 instances): `P = 0.631`, `R = 0.357`, `mAP50 = 0.334`, `mAP50-95 = 0.250`
- `bus` (2 instances): `P = 0.550`, `R = 0.500`, `mAP50 = 0.513`, `mAP50-95 = 0.357` *(cải thiện rõ rệt so với trước khi sửa `bus` ở `drive_002`/`drive_003`/`drive_042`: `P` tăng từ `0.304` lên `0.550`, `mAP50` tăng từ `0.332` lên `0.513`)*
- `van` (12 instances): `P = 0.861`, `R = 0.333`, `mAP50 = 0.645`, `mAP50-95 = 0.372` *(Precision tăng mạnh từ `0.352` lên `0.861`)*

Đầu ra đánh giá Colab (`assets/eval_predictions/` và `assets/yolo26_runs/` nếu đã giải nén từ `day2_colab_outputs.zip`, và lưu trực tiếp kèm ảnh trực quan trong [notebooks/day2_annotation_to_yolo.ipynb](../../notebooks/day2_annotation_to_yolo.ipynb)):
- Kết quả train theo epoch: `assets/yolo26_runs/vehicle_annotation/results.csv`
- Biểu đồ đánh giá / Confusion matrix: `assets/yolo26_runs/vehicle_eval/confusion_matrix.png`
- Ảnh dự đoán trực quan trên `eval`: `assets/eval_predictions/visuals/drive_058.jpg`, `assets/eval_predictions/visuals/drive_051.jpg`

Phân tích cụ thể theo từng ảnh `eval` (đối chiếu log predict của `best.pt` với nhãn `dataset/labels/eval/`):
- Tại `drive_058.jpg`: Ảnh có 39 phương tiện (gồm 32 `car`, 6 `van`, 1 `bus`). Model dự đoán `29 cars, 1 bus, 2 vans` — nhận diện đúng chiếc `bus`, nhưng bỏ sót `4/6` chiếc `van` do mật độ xe dày và góc khuất.
- Tại `drive_051.jpg`: Ảnh có 9 phương tiện (8 `car`, 1 `bus`). Model dự đoán `6 cars`, bỏ sót chiếc `bus` duy nhất trong ảnh.
- Tại `drive_060.jpg`: Nhãn chuẩn có 6 `car`, 1 `truck`, 1 `van` (không có `bus`), nhưng model dự đoán `9 cars, 1 bus, 1 van` — nhận nhầm 1 xe (khả năng là `truck`/`van` cỡ lớn) thành `bus`, khiến Precision của `bus` giảm xuống `0.550`.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
   - Lớp có recall thấp nhất là `van` (`Recall = 0.333`, chỉ bắt đúng `4/12` đối tượng `van` trên tập eval; kế tiếp là `truck` với `Recall = 0.357`, `bus` là `0.500` và `car` là `0.818`).
   - Ảnh minh họa rõ nhất là `drive_058.jpg` (nhãn chuẩn có 6 `van` nhưng model chỉ phát hiện 2 `van`, bỏ sót 4 `van`) và các ảnh `drive_052.jpg`, `drive_054.jpg`, `drive_056.jpg` (mỗi ảnh có 1 `van` nhưng model đều không bắt được).
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
   - Cần kiểm tra độ bám sát biên (bounding box tightness / localization accuracy). Thực tế cho thấy `mAP50 = 0.6005` (riêng `car` đạt `0.910`, `van` đạt `0.645`), nhưng `mAP50-95` chỉ đạt `0.3996` (`car`: `0.619`, `van`: `0.372`). Điều này chứng tỏ model định vị được vùng có xe (`IoU >= 0.50`), nhưng các cạnh box dự đoán còn lỏng, lệch hoặc thiếu góc xe nên ít đạt ngưỡng IoU khắt khe `0.75–0.95`.
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
   - Lỗi giữa `bus` ↔ `van` ↔ `truck` liên quan đến cả mất cân bằng dữ liệu (class imbalance) và ranh giới định nghĩa trong guideline.
   - Dữ kiện hỗ trợ: trong 50 ảnh train chuẩn có `533 car`, `54 van`, `46 truck` và chỉ có `20 bus` (tập eval có `94 car`, `12 van`, `5 truck`, `2 bus`). Việc loại bỏ 2 box nhiễu `bus` ở `drive_002`/`drive_003` và sửa lại chiều cao box `bus` ở `drive_042` đã giúp Precision chung tăng từ `0.4286` lên `0.7426` và `mAP50` của `bus` tăng từ `0.332` lên `0.513`. Tuy nhiên vì số lượng mẫu `bus` (20) và `truck` (46) quá ít so với `car` (533) và ngoại hình xe hình hộp trên camera CCTV dễ chồng lấn với `van`, model vẫn còn nhầm lẫn (như dự đoán nhầm `bus` ở `drive_060.jpg` và bỏ sót `bus` ở `drive_051.jpg`).
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?
   - Nếu dùng 10 ảnh eval để train model sẽ học thuộc lòng(overfitting) khiến các chỉ số mAP tăng ảo, không phản ánh độ chính xác thực tế của model.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Clone repo trong Ubuntu | `fatal: repository .../tree/main/ not found` do URL dính đường dẫn nhánh web | Bỏ `/tree/main` ở cuối URL, dùng đúng link `.git` | Clone thành công |
| Chạy evaluator trên Windows PowerShell | Lỗi cú pháp do dùng dấu `\` xuống dòng kiểu Linux và `cp1252` UnicodeError | Gộp lệnh thành 1 dòng hoặc dùng backtick, đặt encoding UTF-8 | Chạy thành công evaluator & checker |
| Self-QC cấu trúc YOLO | `check_yolo_labels.py` báo 13 lỗi `box vượt ngoài biên ảnh` | Kẹp tọa độ các box chạm mép ảnh về đúng phạm vi `[0, 1]` | Đạt 50/50 file hợp lệ |
| Gold QC & phân tích lớp `bus` | Lần đầu `F1 = 0.5892` (chưa đạt `0.80`); lần train đầu `bus` Precision thấp (`0.304`) | Đối chiếu từng ảnh lỗi, xóa box nhiễu ở `drive_002`/`drive_003`/`drive_024`, chỉnh lại box `bus` ở `drive_042` | `F1` đạt `0.9668`; train lại `Precision` chung tăng `0.4286 → 0.7426` |
| Cấu hình Colab | `SyntaxError: invalid decimal literal` ở `REPO_URL` | Bọc link repo vào trong dấu nháy đơn `'...'` | Notebook chạy hoàn tất với GPU T4 và đã lưu kết quả lên GitHub (`1f4ee25`) |

## 7. Tự đánh giá theo rubric

- Hoàn thành và định dạng YOLO (10/10): Đủ 50 file `.txt` cùng basename, 5 cột chuẩn hóa, pass `check_yolo_labels.py` 50/50.
- Đầy đủ vật thể, độ bám sát và phân lớp (55/55): Đạt mức Xuất sắc theo rubric (`F1 = 0.9668 >= 0.90`, `mean_iou_of_matches = 0.8659 >= 0.80`).
- Mini guideline và quyết định mơ hồ (10/10): Đầy đủ định nghĩa 4 lớp, quy tắc box và 3 ca mơ hồ có ảnh + lý do trong `GUIDELINE_MINI.md`.
- Gold-set annotation QC (15/15): Có file `outputs/annotation_evaluation.json`, ghi nhận đầy đủ số liệu trước/sau rework và nguyên nhân sửa nhãn.
- Pipeline YOLO và phân tích (10/10): Colab đã chạy hoàn chỉnh với đúng split 50 train / 10 eval, notebook đã lưu output đầy đủ trên GitHub, báo cáo phân tích đầy đủ metrics trước/sau sửa nhãn, chỉ rõ từng ảnh eval minh họa và trả lời 4 câu hỏi.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ hỗ trợ: Sử dụng AI Gemini để hướng dẫn thiết lập WSL/CVAT, đọc và xử lý lỗi cú pháp PowerShell/Colab, hỗ trợ viết report
- Phần đã tự kiểm tra: Tự thực hiện gán nhãn trên CVAT, xác nhận các ca mơ hồ, tự chạy kiểm tra `check_yolo_labels.py`, `evaluate_annotations.py` và trực tiếp chạy toàn bộ notebook huấn luyện/đánh giá YOLO26n trên Google Colab. Không dùng thông tin nhạy cảm, token hay đưa `gold_train_labels/` lên repo.