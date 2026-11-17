# Báo cáo Day 08 — 2A202603269 - Vũ Tiến Vinh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

Các lượt chạy Part 2 được ghi trong `submission/model_log.csv` và đối chiếu với `work/runs.csv`. YOLO26 dùng cỡ **s**, YOLOE dùng trọng số **YOLOE-26s-seg**. Tất cả các lượt `r01–r06` chạy bằng **CPU** (`--device cpu`).

### Model và ngưỡng theo lớp

| Lớp | Model + cỡ | Ngưỡng áp dụng | Cơ sở từ log và kết quả quét ngưỡng |
|---|---|---:|---|
| `person` | YOLO26-s | 0.35 | Nguồn `r05`; trên 15 đối tượng gold, recall = 14/15 = 93,3%, precision = 14/25 = 56,0%. Script gợi ý ngưỡng 0.35. |
| `bicycle` | YOLO26-s | 0.30, mặc định | Nguồn `r05`; gold chỉ có 4 đối tượng, dưới mức tối thiểu 5 để chọn ngưỡng riêng. |
| `motorbike` | YOLO26-s | 0.30, mặc định | Nguồn `r05`; gold có 12 đối tượng. Script báo không có ngưỡng ≤ 0.5 đạt recall 90%; recall cao nhất khoảng 83%. |
| `car` | YOLO26-s | 0.45 | Nguồn `r05`; trên 8 đối tượng gold, recall = 8/8 = 100%, precision = 8/11 = 72,7%. Script gợi ý ngưỡng 0.45. |
| `bus` | YOLO26-s | 0.30, mặc định | Nguồn `r05`; gold chỉ có 1 đối tượng, không đủ mẫu để chọn ngưỡng riêng. |
| `truck` | YOLO26-s | 0.30, mặc định | Nguồn `r05`; không có đối tượng truck trong gold để chọn ngưỡng riêng. |
| `three_wheeler` | YOLOE-26s-seg | 0.15 | Nguồn `r06`; trên 7 đối tượng gold, recall = 7/7 = 100%, precision = 7/9 = 77,8%. Script gợi ý ngưỡng 0.15. |

YOLOE sử dụng các prompt mặc định trong `submission/taxonomy.json`: `tuk-tuk`, `cyclo`, `cargo tricycle`, cùng ánh xạ về `three_wheeler`. Lượt `r02` thử trên pilot ở ngưỡng 0.25; lượt `r06` sinh ứng viên trên toàn bộ 36 ảnh ở ngưỡng 0.05.

Nguồn số đo ngưỡng là `submission/qc/threshold_sweep.csv` và `threshold_sweep.md`. Script so với **4 ảnh gold, 47 đối tượng**, ghép đúng lớp với IoU ≥ 0.5; gợi ý ngưỡng cao nhất không vượt quá 0.5 còn đạt recall ≥ 90%, khi lớp có ít nhất 5 đối tượng gold. Các lớp không được gợi ý ngưỡng riêng dùng `--conf 0.3`.

### Lệnh gộp pre-label

Lệnh sau tái tạo đúng file gộp đang dùng: **28 ảnh, 493 box**, gồm 24 ảnh batch và 4 ảnh gold. Thứ tự nguồn là YOLOE trước, YOLO26 sau.

```powershell
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,car=0.45,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

Nguồn đối chiếu: `work/prelabel/merged_review/pre.coco.json` và hai file dự đoán của `r05`, `r06`.

### Phiên bản SAM và quan sát thực nghiệm

Phiên bản đã dùng là **SAM 2.1**, trọng số `sam2.1_b.pt`, trong lượt `r04`: YOLO26-s tạo box rồi SAM tạo mask cho `person,motorbike,car` trên `img_004.jpg` và `img_013.jpg`. Log ghi **10 box và 10 mask trên 2 ảnh**, trung bình **5,43 giây/ảnh trên CPU** cho toàn bộ lượt detector + SAM.

Cơ sở quan sát đã ghi trong log là mask bám khá sát biên vật. Hạn chế đã thấy: tại `img_013.jpg`, mask `person` ở giữa ảnh còn dính nhẹ vào nền. Chưa quan sát thấy sai lớp trong các ảnh đã xem của lượt này. Log Part 2 hiện không có lượt so sánh trực tiếp với MobileSAM.

### Lỗi còn tồn tại khi thay đổi ngưỡng

Các nhận xét của `r01` (YOLO26-s, conf 0.25) và `r03` (YOLO26-s, conf 0.5) đều ghi nhận:
- `img_002.jpg`: ô tô ở mép phải ảnh bị gán thành `truck`.
- `img_007.jpg`: xích lô bên trái ảnh, thuộc `three_wheeler`, bị gán thành `motorbike`.

Như vậy, trong những ca đã quan sát, tăng ngưỡng từ 0.25 lên 0.5 vẫn giữ lại các box sai lớp này. Thay ngưỡng chỉ lọc box theo điểm tin cậy; YOLO26 dùng danh mục COCO đóng, không có lớp `three_wheeler`. Recipe bổ sung YOLOE với prompt cho lớp này, đồng thời việc duyệt trên CVAT vẫn cần sửa những nhãn còn sai.

YOLOE cũng có lỗi theo nhận xét `r02`: `img_002.jpg` có ô tô ở mép phải bị gán thành `three_wheeler`; `img_007.jpg` bị bỏ sót box xích lô bên trái. Đây là các ca cần đối chiếu khi đánh giá recipe.

### Ví dụ box có điểm cao nhưng sai lớp

Tại `img_002.jpg`, ô tô ở mép phải theo quan sát đã ghi trong `model_log.csv` bị YOLO26-s gán thành `truck`. Trong đầu ra `r03`, box này có **conf = 0.7624** và vẫn được giữ khi ngưỡng là 0.5.

Nguồn điểm conf: `work/prelabel/t_y26_c05/pre.coco.json`. Nhận xét về lớp đúng là quan sát đã ghi cho `r01`, `r03`.

### Giới hạn của kết quả

Ngưỡng được chọn trên chính 4 ảnh gold rồi precision/recall cũng được đo trên các ảnh đó. Vì vậy, số đo tại ngưỡng đã chọn có thiên lệch lạc quan do chọn và chấm trên cùng dữ liệu; chúng chưa phải kết quả trên một tập đánh giá độc lập.

Số mẫu mỗi lớp còn ít, đặc biệt `bicycle` có 4 đối tượng, `bus` có 1 và `truck` có 0. Do đó, giữ ngưỡng mặc định cho các lớp này theo quy tắc của script; kết quả trên 4 ảnh gold không đủ để khẳng định hiệu quả tương đương trên toàn bộ batch.

## 2. AI error profile (từ `qc/review_stats.md`)

Nguồn: `submission/qc/review_stats.md`, đối chiếu với `submission/qc/review_stats.json` (chế độ detection).

Bảng AI error profile tính trên **24 ảnh batch** đã duyệt. Ngưỡng giữ nguyên là **IoU ≥ 0.9**, sau khi ghép box cùng lớp theo quy ước của script.

Script đã loại **29 bản ghi đối tượng nằm quanh vùng bẫy** khỏi phép tính profile (tính trên cả nhãn máy và nhãn sau duyệt). Các lỗi được gài nhân tạo dùng để đo người duyệt, không phản ánh lỗi tự nhiên của model, nên không đưa vào bảng chất lượng máy.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---|---|
| accepted (giữ nguyên) | 316 | 316/416 = 76,0% | 416 box máy |
| edited (sửa) | 45 | 45/416 = 10,8% | 416 box máy |
| deleted (xóa) | 55 | 55/416 = 13,2% | 416 box máy |
| added (vẽ thêm) | 113 | 113/474 = 23,8% | 474 đối tượng sau duyệt |

Trong số deleted và added, **11 trường hợp là đổi lớp**: một box máy sai lớp được tính là xóa và một box đúng lớp được tính là thêm. Vì vậy, 113 box added không đồng nghĩa với 113 đối tượng bị bỏ sót. Theo cách ước tính của script, số đối tượng máy sót thật **≈ 113 − 11 = 102**.

### Theo lớp

| Lớp | accepted | edited | deleted | added |
|---|---:|---:|---:|---:|
| `person` | 182 | 12 | 9 | 50 |
| `bicycle` | 2 | 3 | 1 | 4 |
| `motorbike` | 95 | 15 | 11 | 29 |
| `car` | 30 | 4 | 4 | 22 |
| `bus` | 2 | 0 | 1 | 1 |
| `truck` | 0 | 0 | 3 | 4 |
| `three_wheeler` | 5 | 11 | 26 | 3 |

Theo bảng tổng hợp, `three_wheeler` có số box bị xóa nhiều nhất (**26**), cùng **11** box được sửa và **5** box giữ nguyên. `person` có số box vẽ thêm nhiều nhất (**50**), tiếp theo là `motorbike` (**29**) và `car` (**22**). Số liệu này mô tả các thay đổi sau duyệt; bảng tổng hợp chưa xác định nguyên nhân cụ thể hoặc ảnh minh họa cho từng lỗi. Các ví dụ ảnh và nguyên nhân cần được bổ sung từ quan sát khi review.

### Máy so với gold

Đối chiếu pre-label của máy với **4 ảnh gold, 47 đối tượng**: **TP = 42, FP = 20, FN = 5**.

- **Precision:** 42/(42 + 20) = **42/62 = 67,7%**; mẫu số là 62 box máy trên gold. Khoảng tin cậy 95% theo script: **55–78%**.
- **Recall:** 42/(42 + 5) = **42/47 = 89,4%**; mẫu số là 47 đối tượng gold. Khoảng tin cậy 95% theo script: **77–95%**.

Các số đo trên là precision/recall của **máy**, chưa phải độ chính xác của nhãn sau duyệt. Ngưỡng được chọn trên chính 4 ảnh gold này nên kết quả có thiên lệch lạc quan; mẫu nhỏ cũng khiến khoảng tin cậy rộng. Vì 4 ảnh gold không có trong file batch sau duyệt, script không tính residual error hoặc new error cho gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Nguồn: `submission/qc/review_stats.md` và `submission/qc/review_stats.json`, tính từ `batch_reviewed.json` sau khi hoàn tất duyệt 24 ảnh.

### Kết quả bắt bẫy

Bắt được **10/12 bẫy = 83,3%**, đạt ngưỡng tối thiểu **80%** của bài. Khoảng tin cậy 95% theo script là **55–95%**.

| Loại bẫy | Bắt được / Tổng | Tỉ lệ bắt được | Còn lọt |
|---|---:|---:|---:|
| `loose` — box lỏng/lệch | 3/3 | 100% | 0 |
| `wrong_class` — sai lớp | 3/3 | 100% | 0 |
| `missing` — thiếu box | 1/3 | 33,3% | 2 |
| `extra` — box thừa | 3/3 | 100% | 0 |
| **Tổng** | **10/12** | **83,3%** | **2** |

Hai bẫy còn lọt đều thuộc loại `missing`:
- `img_018.jpg`.
- `img_005.jpg`.

Theo tiêu chí chấm của script, hai trường hợp này chưa có box sau duyệt cùng lớp và đủ độ trùng với box tham chiếu của bẫy. Vì vậy, đây là các ca cần xem lại trong bước rà đối tượng bị bỏ sót.

### Nguyên nhân và đối chiếu checklist

Tôi có thực hiện bước **ẩn nhãn máy để rà vật**. Theo tự đánh giá, tôi cho rằng hai bẫy `missing` tại `img_018.jpg` và `img_005.jpg` còn lọt vì việc vẽ bổ sung/vẽ lại box chưa bao quát hết các trường hợp cần xử lý.

Đây là nhận định của tôi về quá trình duyệt. Kết quả script xác nhận hai ca chưa đạt tiêu chí bắt bẫy, còn nguyên nhân cụ thể của từng ca cần được đối chiếu thêm với nhãn đã sửa. Vì tôi có ẩn nhãn để rà vật, không quy hai bẫy lọt thành việc bỏ qua bước này; hạn chế tôi tự ghi nhận là mức độ bao quát khi rà và vẽ lại box.

### Đối chiếu với accepted

Tỉ lệ `accepted` là **316/416 box máy = 76,0%**; tỉ lệ bắt bẫy là **10/12 = 83,3%**. Hai chỉ số có mẫu số và mục đích khác nhau: accepted mô tả các box máy được giữ theo tiêu chí IoU của script, còn bắt bẫy kiểm tra khả năng sửa các lỗi đã biết được gài vào nhãn.

Vì vậy, accepted không thay thế việc đo bắt bẫy. Trong kết quả lần này, người duyệt đã bắt được toàn bộ bẫy sai lớp, box thừa và box lỏng, nhưng vẫn để lọt 2/3 bẫy thiếu box. Chưa thể dùng riêng hai tỉ lệ trên để khẳng định đã phát hiện hết lỗi của máy hoặc không còn thiên kiến tự động hóa.

### Giới hạn của phép đo

Bộ thử chỉ có **12 bẫy**; một bẫy tương ứng khoảng **8,3 điểm phần trăm**. Khoảng tin cậy rộng nên kết quả 83,3% cần được đọc cùng loại bẫy và hai ảnh còn lọt. Đạt ngưỡng bắt bẫy chưa thay thế kiểm tra QC và quyết định chất lượng ở Part 6.

## 4. Máy có đáng dùng không?

Nguồn: `submission/time_log.csv` và `submission/qc/review_stats.md` (đối chiếu với `review_stats.json`).

### Thời gian thực tế

| Công việc | Tổng thời gian | Số ảnh | Trung bình |
|---|---:|---:|---:|
| Gán tay gold (`gold_manual`) | 16 phút | 4 | 16/4 = **4 phút/ảnh** |
| Duyệt có pre-label (`batch_review`) | 48 phút | 24 | 48/24 = **2 phút/ảnh** |

Mức giảm thời gian trung bình theo ảnh trong phép so sánh này là **(4 − 2)/4 × 100% = 50%**.

Hai tập ảnh có số lượng, số đối tượng và độ phức tạp khác nhau. Tôi cũng đã làm quen với guideline và cảnh phố khi gán gold trước khi duyệt batch. Vì vậy, chưa thể quy toàn bộ mức giảm thời gian cho AI hoặc khẳng định mọi lô ảnh khác đều tiết kiệm 50%. Phép tính trên so sánh hai bước gán tay/duyệt đã ghi trong log, không phải tổng thời gian cài đặt và chạy toàn bộ quy trình.

### Công sức sửa và chất lượng sau review

- **Giữ nguyên:** 316/416 box máy = **76,0%**.
- **Sửa:** 45/416 box máy = **10,8%**.
- **Xóa:** 55/416 box máy = **13,2%**.
- **Vẽ thêm:** 113/474 đối tượng sau duyệt = **23,8%**. Added có bao gồm trường hợp đổi lớp như đã giải thích ở mục 2.
- **Bắt bẫy:** 10/12 = **83,3%**; còn lọt hai bẫy `missing` tại `img_018.jpg` và `img_005.jpg`.

Tỉ lệ giữ nguyên cho biết phần nhãn máy được giữ theo tiêu chí ghép của script; không tự chứng minh tất cả các box đó đúng thực tế. Công sửa, xóa, vẽ thêm và các bẫy còn lọt cần được cân nhắc cùng với thời gian.

### Đánh giá của tôi

**Có đáng dùng như công cụ tạo nhãn nháp có người duyệt.** Trong lần thực hành này, thời gian duyệt trung bình là 2 phút/ảnh so với 4 phút/ảnh khi gán tay, và 316/416 box máy được giữ nguyên. Tuy nhiên, tôi vẫn phải sửa 45 box, xóa 55 box và vẽ thêm 113 box; hai bẫy thiếu box vẫn còn lọt. Vì vậy, lợi ích tôi ghi nhận là hỗ trợ tạo bản nháp và giảm thao tác vẽ từ đầu, đi kèm yêu cầu kiểm tra đầy đủ theo guideline. Đánh giá này không thay thế QC hoặc quyết định Release / Hold / Rework ở Part 6.

### Ba quy tắc giảm thiên kiến cho lần review tiếp theo

Các quy tắc dưới đây là kế hoạch áp dụng và kiểm chứng cho lần duyệt tiếp theo; không phải khẳng định đã có đầy đủ nhật ký chi tiết của lần review vừa rồi.

1. **Ẩn nhãn và quét ảnh trước:** ở mỗi ảnh, ẩn toàn bộ nhãn máy khoảng 15 giây để rà đối tượng trước khi đối chiếu với các box có sẵn. Ghi một ô hoàn thành theo mã ảnh trong checklist; kiểm tra bằng số ảnh đã thực hiện trên tổng số ảnh của lô. Tiếp tục rà các vùng chưa có box khi kiểm tra bỏ sót.
2. **Kiểm tra biên box khi phóng to:** với box vừa và nhỏ cần kiểm tra, phóng to tối thiểu 200% để xem phần nhìn thấy của vật và các cạnh box. Ghi mã ảnh cùng ID hoặc vị trí các box đã kiểm tra, kèm lỗi phát hiện nếu có. Dùng bản ghi thao tác này để kiểm chứng; không dùng điều kiện `edited > 5%` làm bằng chứng đã kiểm tra đúng.
3. **Nghỉ và ghi giờ theo đợt:** sau mỗi 8 ảnh duyệt liên tục, nghỉ 2 phút. Ghi thời điểm hoặc số phút làm/nghỉ theo từng đợt; thời gian nghỉ được ghi riêng với thời gian review trong `time_log.csv`. Dùng nhật ký này để đối chiếu số đợt nghỉ và thời gian thực tế.

## 5. Mask SAM cho 3 vật (Part 5)

Sử dụng **SAM 2.1**, trọng số `sam2.1_b.pt`, chạy bằng **CPU** qua `scripts/sam2_points.py`. Đã tạo ba ảnh kết quả cho **ba đối tượng khác nhau** trên hai ảnh nguồn.

Các điểm gợi ý được công cụ hỗ trợ chuẩn bị từ ví dụ trong `GUIDE.md` và các box trong `submission/labels/batch_reviewed.json`. Trường hợp 1 dùng tọa độ ví dụ của đề; trường hợp 2 và 3 dùng thêm box đã duyệt để gợi ý đối tượng. Box gợi ý được ghi riêng, không cộng vào số click dương/âm.

| Ảnh / đối tượng | SAM | Số click | Box gợi ý | Số liệu mask từ script | Nhận xét trực quan | File kết quả |
|---|---|---|---|---|---|---|
| img_013.jpg — xe ba bánh, annotation #115 | SAM 2.1 | 3 (2 dương, 1 âm) | Không | 317.255 px; 1 polygon | Theo nhận xét của học viên: mask bám sát vật, thiếu vài pixel nhỏ; không thấy dính nền, người hoặc hàng hóa. | [case01_tuktuk_img013.jpg](masks/case01_tuktuk_img013.jpg) |
| img_004.jpg — xe ba bánh, annotation #24 | SAM 2.1 | 2 (1 dương, 1 âm) | Có | 93.041 px; 1 polygon | Theo nhận xét của học viên: mask bám sát vật, thiếu vài pixel nhỏ; không thấy dính nền, người hoặc hàng hóa. | [case02_three_wheeler_img004.jpg](masks/case02_three_wheeler_img004.jpg) |
| img_013.jpg — người, annotation #113 | SAM 2.1 | 2 (1 dương, 1 âm) | Có | 32.737 px; 2 polygon | Theo nhận xét của học viên: mask bám sát người, thiếu vài pixel nhỏ; không thấy dính nền, hàng hóa hoặc đối tượng khác. | [case03_person_img013.jpg](masks/case03_person_img013.jpg) |

### Lệnh và tọa độ đã chạy

**1. img_013.jpg — xe ba bánh, annotation #115**

```powershell
python scripts/sam2_points.py --image img_013.jpg --pos "400,480" --pos "600,110" --neg "720,300" --sam sam2.1_b.pt --device cpu --out submission/masks --name case01_tuktuk_img013
```

**2. img_004.jpg — xe ba bánh, annotation #24**

```powershell
python scripts/sam2_points.py --image img_004.jpg --pos "654,413" --neg "480,362" --box "474.04,230,834.61,596.88" --sam sam2.1_b.pt --device cpu --out submission/masks --name case02_three_wheeler_img004
```

**3. img_013.jpg — người, annotation #113**

```powershell
python scripts/sam2_points.py --image img_013.jpg --pos "720,300" --neg "600,110" --box "619.27,193.48,874.6,521.8" --sam sam2.1_b.pt --device cpu --out submission/masks --name case03_person_img013
```

### Đối chiếu guideline 4.2

Khi xem ba ảnh kết quả, đối chiếu các quy tắc đã có trong guideline v1.0:
- Mask bám biên phần nhìn thấy của đối tượng.
- Xử lý khoảng trống/lỗ theo ngưỡng diện tích nêu trong guideline.
- Bóng đổ không thuộc vật; kính xe thuộc vật.
- Hai vật chạm nhau được tách riêng; kiểm tra vùng dính người, hàng hóa hoặc nền theo đối tượng đang chọn.

Học viên đã xem ba ảnh và nhận xét mask bám sát đối tượng, còn thiếu vài pixel nhỏ, không thấy dính nền, người/hàng hóa hoặc đối tượng khác. Đây là nhận xét trực quan; chưa có phép đo độ lệch biên để khẳng định đạt giới hạn khoảng 3 px của guideline. Diện tích và số polygon vẫn là số đo riêng do script sinh ra.

### Bằng chứng thực nghiệm

Thông số, số click, box gợi ý, diện tích, số polygon và argv từng lượt được lưu trong `submission/masks/part5_runs.json`. Log console gốc nằm ở `work/part5/`. Các ảnh là kết quả chạy thực tế của script; nhận xét trực quan do học viên xác nhận sau khi xem ảnh và được ghi cùng thông số trong part5_runs.json. Phần thực hiện này dùng script offline, chưa ghi nhận thử nghiệm Interactor SAM trong CVAT.

## 6. Calibration (Part 6)

- Đã ghi **4 quyết định thực tế** trong `submission/qc/calibration_decisions.csv`, gồm `loi_may`, `cau_hoi_mo`, `bay_lot` và `can_review`.
- **CAL-01 — img_007.jpg:** áp dụng định nghĩa three_wheeler cho xích lô bị YOLO26-s gán sai; đây là lỗi máy/áp dụng quy tắc đã rõ.
- **CAL-02 — img_002.jpg:** học viên chốt mọi box đều phải đối chiếu ảnh; conf chỉ để lọc và ưu tiên kiểm tra.
- **CAL-03 — img_005.jpg, img_018.jpg:** làm rõ việc rà đủ đối tượng, kể cả vùng không có box, sau hai bẫy missing còn lọt.
- **CAL-04 — img_025.jpg:** hai box #307 và #308 được tick can_review vì học viên nghi là xe đạp điện. Học viên chốt quy tắc: xe hai bánh có động cơ, kể cả xe đạp điện, thuộc motorbike; chưa xác định được thì giữ can_review.
- Guideline v1.1 đã ghi các quyết định tương ứng, ảnh ví dụ và lịch sử thay đổi; đã chốt câu hỏi mở số 1 và 7. Taxonomy ở v1.1, giữ nguyên rules, các lớp và thuộc tính.
- **Kết quả kiểm tra bổ sung:** học viên đã xem lại mẫu QC với ảnh phóng to hỗ trợ các box nhỏ/xa và #307/#308, xác nhận không phát hiện lỗi. Giữ nhãn hiện có; không thay đổi thuộc tính can_review. Quy tắc CAL-04 vẫn yêu cầu giữ can_review khi chưa xác định được động cơ.
- CAL-01 là áp dụng quy tắc cũ; CAL-02/CAL-04 chốt câu hỏi mở; CAL-03 ghi nhận hạn chế bao quát khi review. Không dùng những quyết định xe điện/hàng cồng kềnh của mẫu để giả định đã thực hiện.

## 7. QC và quyết định (Part 6)

Nguồn: `submission/qc/qc_sample.csv`, `qc_review.csv` và `qc_report_auto.md`.

- Bản ứng viên gồm **24 ảnh batch, 493 bản ghi đối tượng/vùng**, trong đó có **5 ignore_region**.
- Lấy mẫu bằng `python scripts/qc_sample.py --n 5 --seed 42`: **6 ảnh, 103 đối tượng**, gồm **1 ảnh can_review + 5 ảnh ngẫu nhiên**.
- Các ảnh: img_003.jpg, img_006.jpg, img_013.jpg, img_014.jpg, img_025.jpg, img_033.jpg. Người kiểm: **Vũ Tiến Vinh**; đây là tự kiểm, không phải đánh giá độc lập.
- Sau vòng xem đầu còn box ở xa chưa rõ, học viên đã kiểm bổ sung với ảnh phóng to và xác nhận **không phát hiện lỗi** trong cả 6 ảnh. Phiếu qc_review.csv ghi `khong` cho từng ảnh. Các ảnh phóng to tạm đã được xóa theo yêu cầu; ảnh QC gốc được giữ trong `work/qc/review/`.
- Hai box motorbike #307/#308 tại img_025.jpg được giữ theo kết quả QC của học viên và quy tắc CAL-04; thuộc tính can_review không bị sửa tự động.

| Chỉ số QC quan sát được | Kết quả | Mẫu số |
|---|---|---|
| Ảnh có lỗi | **0/6 = 0,0%** | 6 ảnh trong mẫu |
| Đối tượng có lỗi | **0/103 = 0,0%** | 103 đối tượng trong mẫu |
| Loại lỗi / lỗi hệ thống | Không ghi nhận | Theo phiếu kiểm của học viên |
| Bắt bẫy Part 4 | **10/12 = 83,3%** | 12 bẫy |

Script `qc_report.py` gợi ý **RELEASE** theo phiếu QC hiện tại và ngưỡng trong taxonomy. Quyết định dưới đây áp dụng các tiêu chí nghiệm thu của bài vào số liệu đã ghi nhận. Hai bẫy missing tại img_005.jpg và img_018.jpg vẫn là hạn chế đã ghi nhận ở Part 4; kết quả mẫu QC không khẳng định hai ca này đã được sửa.

**Giới hạn mẫu nhỏ:** với 0 ảnh lỗi trong 6 ảnh mẫu, cận trên 95% theo Wilson do script tính là **39%**. Vì vậy, kết quả quan sát 0% chưa chứng minh tỉ lệ ảnh lỗi thật của toàn bộ lô ≤ 2%. Việc tự kiểm và độ phân giải ảnh gốc cũng giới hạn khả năng phát hiện lỗi.

**Quyết định: RELEASE theo tiêu chí nghiệm thu của bài.** Tỉ lệ ảnh lỗi quan sát được là **0/6 = 0,0%**, không vượt ngưỡng 2%; chưa ghi nhận lỗi hệ thống trong mẫu QC; tỉ lệ bắt bẫy **10/12 = 83,3%** đạt ngưỡng tối thiểu 80%. Các ca guideline thực tế đã nêu có quy tắc xử lý trong v1.1, và chưa ghi nhận ảnh hỏng không dùng được. Quyết định này dựa trên mẫu QC và tiêu chí của bài, không khẳng định toàn bộ lô không còn lỗi. Hai bẫy missing còn lọt và giới hạn Wilson 39% được giữ nguyên trong báo cáo; chưa có bằng chứng hai bẫy này đã được sửa.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Antigravity AI Assistant làm kỹ sư Tech Lead đồng hành hỗ trợ giải thích kiến thức, gợi ý quy trình thực hiện, phân tích nguyên nhân lỗi và chạy các lệnh kiểm tra tự động; toàn bộ thao tác gán nhãn thủ công trên CVAT và quan sát hình ảnh đều do học viên trực tiếp thực hiện.
- Gold có bị nhiễm không? Theo quá trình làm bài, 4 ảnh gold được tự gán tay trước khi chạy model trên chính các ảnh gold. Lượt kiểm tra setup dùng ảnh mẫu bus.jpg của Ultralytics; các lượt --split all được chạy sau khi hoàn tất gold.
- Máy / GPU / CVAT dùng: HP Victus 16, Windows 11 64-bit, Ryzen 7 5800H, RAM 20 GB; có RTX 3050 Ti nhưng các lượt r01–r06 và Part 5 chạy CPU. Python 3.12.10, PyTorch 2.15.0+cpu; CVAT 2.81.0 tại http://localhost:8081, Docker Desktop dùng Linux/WSL 2.
- Dùng cache `data/cache/` thay vì tự chạy? Các lượt ghi trong work/runs.csv được chạy bằng script trên CPU; kết quả Part 2–3 sử dụng các file đầu ra của r01–r06, không dùng file dự đoán từ data/cache/.
- Điều chỉnh script hỗ trợ setup: day08_setup.py kiểm tra đúng CVAT 2.81.0 ở cổng 8081 và dùng UTF-8 cho đầu ra tiếng Việt. Khi ghi CSV qua PowerShell, xử lý mã hóa để giữ đúng dấu tiếng Việt.
- Một điều bạn làm khác lần sau: Sẽ luôn ẩn nhãn máy trước khi xem ảnh để tránh bị bẫy bỏ sót do tâm lý phụ thuộc vào AI.
