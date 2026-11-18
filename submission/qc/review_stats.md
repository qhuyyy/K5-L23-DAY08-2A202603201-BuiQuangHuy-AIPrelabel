# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 655; đối tượng sau duyệt: 656; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 493 | 493/655 = 75.3% | 655 box máy |
| edited (sửa) | 66 | 66/655 = 10.1% | 655 box máy |
| deleted (xoá) | 96 | 96/655 = 14.7% | 655 box máy |
| added (vẽ thêm) | 97 | 97/656 = 14.8% | 656 đối tượng sau duyệt |

Trong số deleted + added, **12 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 85.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 315 | 24 | 38 | 27 |
| bicycle | 5 | 1 | 2 | 6 |
| motorbike | 123 | 26 | 22 | 29 |
| car | 39 | 3 | 5 | 24 |
| bus | 3 | 0 | 0 | 3 |
| truck | 2 | 0 | 4 | 3 |
| three_wheeler | 6 | 12 | 25 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 71 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 55 | 25 | 16 | 68.8% (KTC 95%: 58–78%) | 77.5% (KTC 95%: 66–86%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - extra: 2/3
  - loose: 2/3
  - wrong_class: 2/3
  - lọt: img_006.jpg (extra), img_025.jpg (loose), img_008.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
