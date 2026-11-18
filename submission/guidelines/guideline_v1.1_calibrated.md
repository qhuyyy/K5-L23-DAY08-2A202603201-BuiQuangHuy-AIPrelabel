# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603201 - Bùi Quang Huy

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng nhô ra: xe ba bánh / xe đạp chở hàng, box xe có gồm hàng không? | Box xe **ôm cả hàng chất trong thùng / khay** gắn liền xe. Phần hàng **nhô ra ngoài thân xe hơn ~20% chiều dài xe** thì không tính (giữ nguyên ngưỡng 4.1.5). | img_003 (xe ba bánh bán hoa quả) | CAL-04 |
| 6 | Box máy chồng nhau quá nhiều: gộp hay giữ? | **Mỗi vật đúng một box.** Hai box cùng lớp chồng lên một vật thì **giữ box ôm sát nhất, xoá box kia**. **Không gộp** thành một box lớn hơn (trái 4.1.1 ôm sát). | img_003 (#18 truck trùng), img_006 (#91 / #111 motorbike) | CAL-03 |

**Chưa chốt** (không gặp đủ ca trong lô này để quyết): câu 1 (xe đạp điện), câu 2 (khối xe máy dày gần camera), câu 4 (người trong xe có mui), câu 5 (kính xe trong mask), câu 7 (mức `conf` tối thiểu).

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_007, img_032, img_004 | Tuk-tuk, xích lô, ba gác | YOLO26 (COCO, không có lớp xe ba bánh) gọi là `car` / `truck` / `motorbike` / `bicycle` hoặc bỏ sót; khi review 25 box `three_wheeler` bị xoá, 12 bị sửa | Luôn gán `three_wheeler` | **Mục 2 + 4.3:** mọi xe có 3 bánh (đếm được 3 bánh, hoặc thấy mui / thùng hàng gắn liền khung xe ba bánh) là `three_wheeler`, **bất kể máy gọi là gì**. Khi review phải soát mọi box `car` / `truck` / `motorbike` / `bicycle` nằm trên xe ba bánh. Lỗi này **không sửa được bằng ngưỡng**. (CAL-02) |
| img_036, img_001 | Người đứng riêng, cao 17–19.8 px (dưới / sát ngưỡng 20 px) nhưng vẫn nhận ra rõ là người | — (gold gán tay) | Giữ box `person` riêng | **Mục 4.1.6:** ngưỡng 20 px áp cho vật **không nhận ra rõ** hoặc nằm trong cụm ≥ 5 vật; vật **đứng riêng, nhận ra rõ lớp**, cao 17–20 px vẫn vẽ box riêng. (CAL-01) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 3 (hàng trong thùng tính vào box xe, nhô ra > ~20% thì không) và 6 (mỗi vật một box, giữ box ôm sát nhất); thêm quy tắc xe ba bánh luôn là `three_wheeler` bất kể nhãn máy; nới 4.1.6 cho vật đứng riêng nhận ra rõ cao 17–20 px | CAL-04, CAL-03, CAL-02, CAL-01 |
