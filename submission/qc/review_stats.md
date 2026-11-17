# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 416; đối tượng sau duyệt: 474; ngưỡng accepted: IoU >= 0.9
- Đã **loại 29 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 316 | 316/416 = 76.0% | 416 box máy |
| edited (sửa) | 45 | 45/416 = 10.8% | 416 box máy |
| deleted (xoá) | 55 | 55/416 = 13.2% | 416 box máy |
| added (vẽ thêm) | 113 | 113/474 = 23.8% | 474 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 102.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 182 | 12 | 9 | 50 |
| bicycle | 2 | 3 | 1 | 4 |
| motorbike | 95 | 15 | 11 | 29 |
| car | 30 | 4 | 4 | 22 |
| bus | 2 | 0 | 1 | 1 |
| truck | 0 | 0 | 3 | 4 |
| three_wheeler | 5 | 11 | 26 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 47 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 42 | 20 | 5 | 67.7% (KTC 95%: 55–78%) | 89.4% (KTC 95%: 77–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - wrong_class: 3/3
  - missing: 1/3
  - extra: 3/3
  - lọt: img_018.jpg (missing), img_005.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
