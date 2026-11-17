# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 366; đối tượng sau duyệt: 514; ngưỡng accepted: IoU >= 0.9
- Đã **loại 20 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 224 | 224/366 = 61.2% | 366 box máy |
| edited (sửa) | 88 | 88/366 = 24.0% | 366 box máy |
| deleted (xoá) | 53 | 53/366 = 14.5% | 366 box máy |
| added (vẽ thêm) | 202 | 202/514 = 39.3% | 514 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 193.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 125 | 49 | 25 | 78 |
| bicycle | 3 | 0 | 3 | 5 |
| motorbike | 59 | 34 | 10 | 70 |
| car | 33 | 5 | 8 | 25 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 6 | 2 |
| three_wheeler | 0 | 0 | 0 | 20 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 55 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 28 | 18 | 27 | 60.9% (KTC 95%: 46–74%) | 50.9% (KTC 95%: 38–64%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 12/12 = 100.0% (KTC 95%: 76–100%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - wrong_class: 3/3
  - missing: 3/3
  - extra: 3/3

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
