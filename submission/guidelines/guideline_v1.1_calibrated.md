# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603269 - Vũ Tiến Vinh

Bản này bổ sung v1.0, không thay đổi danh mục lớp, ngưỡng QC hoặc mục `rules` trong taxonomy. Các ca được ghi trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt

| Câu hỏi v1.0 | Quyết định | Ảnh ví dụ | case_id |
|---|---|---|---|
| 1 — Xe đạp điện / xe máy điện thuộc lớp nào? | Xe hai bánh có động cơ, kể cả xe đạp điện, thuộc motorbike; chưa xác định được thì giữ can_review theo mục 4.3 của v1.0. | [img_025.jpg](../../data/images/img_025.jpg), hai box #307 và #308 trong bản ứng viên. | CAL-04 |
| 7 — Có ngưỡng conf để chấp nhận box mà không kiểm tra không? | Mọi box đều phải đối chiếu ảnh; conf chỉ để lọc và ưu tiên kiểm tra. Không dùng riêng điểm cao để bỏ qua việc kiểm tra lớp và hình học theo v1.0. | [img_002.jpg](../../data/images/img_002.jpg): box ở mép phải mang lớp truck, conf=0.7624; học viên xác định vật là car trong r01/r03. | CAL-02 |

Quyết định CAL-02 và CAL-04 được học viên xác nhận, chốt câu hỏi mở số 7 và số 1. Các câu hỏi 2–6 của v1.0 chưa được chốt thêm trong bản này; nếu gặp ca thực tế chưa thống nhất cách gán, cần ghi lại để làm rõ.

## 2. Ca khó và cách áp dụng quy tắc

| Ảnh | Vấn đề và nguồn | Quyết định áp dụng | Bổ sung/làm rõ | case_id |
|---|---|---|---|---|
| [img_007.jpg](../../data/images/img_007.jpg) | `loi_may`: xích lô bên trái bị YOLO26-s gọi là motorbike ở r01 và r03. | Xích lô thuộc three_wheeler, theo mục 2 và bảng ca khó của v1.0. | Minh họa lỗi máy/áp dụng; không đổi định nghĩa lớp đã rõ. | CAL-01 |
| [img_005.jpg](../../data/images/img_005.jpg), [img_018.jpg](../../data/images/img_018.jpg) | `bay_lot`: còn hai bẫy missing theo review_stats; học viên đã ẩn nhãn nhưng tự đánh giá việc vẽ lại box chưa bao quát hết trường hợp. | Áp dụng lại nguyên tắc gán mọi vật thuộc phạm vi, nhìn rõ và đủ lớn của v1.0; rà cả vùng chưa có box theo checklist. | Ghi kiểm tra theo mã ảnh, không xem thao tác ẩn nhãn là bằng chứng đã tìm đủ vật. Đây là làm rõ quy trình áp dụng; chưa khẳng định hai ca đã được sửa. | CAL-03 |

## 3. Ca cần làm rõ thêm

Ảnh `img_025.jpg` có hai box motorbike #307 và #308 được đánh can_review vì học viên nghi là xe đạp điện. Quy tắc phân lớp đã được chốt tại CAL-04. Học viên đã kiểm bổ sung mẫu QC, gồm hai box này và các box ở xa, rồi xác nhận không phát hiện lỗi. Giữ nhãn hiện có; không thay đổi thuộc tính can_review. Nếu chưa xác định được động cơ, tiếp tục giữ can_review theo quy tắc đã chốt.

## 4. Lịch sử thay đổi

| Phiên bản | Thay đổi | Căn cứ |
|---|---|---|
| v1.0 | Bản giảng viên, giữ nguyên. | Guideline gốc. |
| v1.1 | Chốt câu hỏi mở số 1 và 7; thêm ví dụ áp dụng lớp xích lô, rà vật thiếu box và xử lý xe nghi là xe đạp điện. | CAL-01, CAL-02, CAL-03, CAL-04; quan sát và quyết định do học viên xác nhận. |
