# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 3 ảnh (0 ca khó + 3 ngẫu nhiên), 68 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 6/68 = 8.8% | 68 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 3/3 = 100.0% | 3 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 493/655 = 75.3% / 66/655 = 10.1% / 96/655 = 14.7% | 655 box máy |
| Máy: added | 97/656 = 14.8% | 656 đối tượng sau duyệt |
| Bắt lỗi bẫy | 9/12 = 75.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 4 | img_003.jpg, img_006.jpg, img_031.jpg |
| sai_vi_tri_kich_thuoc | 2 | img_003.jpg, img_006.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 3 ảnh lỗi trong 3 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **100%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 100.0% > 2%
- lỗi hệ thống: 'thua' chiếm 4/6 lỗi
- bắt bẫy 75% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
