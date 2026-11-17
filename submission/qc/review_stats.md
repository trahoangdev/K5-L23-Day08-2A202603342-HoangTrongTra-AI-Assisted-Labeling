# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 518; đối tượng sau duyệt: 505; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 446 | 446/518 = 86.1% | 518 box máy |
| edited (sửa) | 14 | 14/518 = 2.7% | 518 box máy |
| deleted (xoá) | 57 | 57/518 = 11.0% | 518 box máy |
| added (vẽ thêm) | 45 | 45/505 = 8.9% | 505 đối tượng sau duyệt |

Trong số deleted + added, **8 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 37.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 258 | 3 | 9 | 12 |
| bicycle | 5 | 0 | 3 | 4 |
| motorbike | 127 | 1 | 8 | 8 |
| car | 44 | 0 | 10 | 10 |
| bus | 3 | 0 | 0 | 4 |
| truck | 2 | 0 | 4 | 3 |
| three_wheeler | 7 | 10 | 23 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 69 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 51 | 18 | 18 | 73.9% (KTC 95%: 62–83%) | 73.9% (KTC 95%: 62–83%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - loose: 2/3
  - extra: 3/3
  - missing: 3/3
  - lọt: img_017.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
