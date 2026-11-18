# Báo cáo Day 08 — 2A202603201 - Bùi Quang Huy

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m (r02, r06) | 0.15 | <<vì sao>> |
| motorbike | YOLO26m (r02, r06) | 0.2 | <<vì sao>> |
| car | YOLO26m (r02, r06) | 0.4 | <<vì sao>> |
| bicycle / bus / truck | YOLO26m (r02, r06) | 0.3 (mặc định) | gold chỉ có 2 / 1 / 0 vật (< 5), không đủ để chọn ngưỡng riêng (`threshold_sweep.md`) |
| three_wheeler | YOLOE-26s, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` (r04, r07) | 0.15 (script gợi ý) | <<vì sao>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,motorbike=0.2,car=0.4,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 759 box trên 28 ảnh (24 batch + 4 gold); sau đó `inject_traps.py --student "2A202603201"` cài 12 bẫy.
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 493 | 75.3% | 655 box máy (đã loại 25 đối tượng quanh bẫy) |
| edited | 66 | 10.1% | 655 box máy |
| deleted | 96 | 14.7% | 655 box máy |
| added | 97 (12 là đổi lớp → máy sót thật ≈ 85) | 14.8% | 656 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** 55/80 = 68.8% (KTC 95%: 58–78%) / 55/71 = 77.5% (KTC 95%: 66–86%) (mẫu số: 71 đối tượng gold, không tính ignore_region; 80 box máy trên 4 ảnh gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 9/12 = 75.0% (KTC 95%: 47–91%, dưới ngưỡng 80%); loại nào lọt: extra 1/3 (img_006: box `truck` giả), loose 1/3 (img_025: box `motorbike` bị kéo lệch), wrong_class 1/3 (img_008: `person` bị đổi thành `bicycle`); missing bắt 3/3; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 27/4 = 6.75 phút/ảnh (29/4 = 7.25 nếu tính 2 phút sửa gold) vs duyệt 60/24 = 2.5 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk) | SAM 2.1 b (`sam2.1_b.pt`) | 2 dương + 1 âm (click âm đặt trên người lái) | <<>> | <<>> | `tuktuk_img013.jpg` |
| img_003 (xe ba bánh bán hoa quả) | SAM 2.1 b | 1 dương + 1 box | <<>> | <<>> | `three_wheeler_img003.jpg` |
| img_017 (tuk-tuk, lần 1) | SAM 2.1 b | 1 dương, không box | <<>> | <<>> | `three_wheeler_img017.jpg` |
| img_017 (tuk-tuk, lần 2) | SAM 2.1 b | 3 dương + 1 box | <<>> | <<>> | `three_wheeler_img017_box.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code (trợ lý trong VS Code): khôi phục môi trường và chạy các script theo GUIDE; làm trang HTML xem ảnh `viz/` để tôi so sánh các run; chuẩn hoá chính tả / thuật ngữ cột `what_went_right` / `what_went_wrong` trong `model_log.csv` (nội dung quan sát là của tôi), viết nháp cột `decision` và nhận xét r06 / r07 từ output script; tôi đã đọc lại, đối chiếu với ảnh `viz/` và sửa. <<bổ sung nếu dùng thêm>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không: chưa chạy model nào trước khi export gold. Sau lần export đầu (78 box), `validate_labels` cảnh báo 8 box cao < 20 px; tôi xem lại ảnh, xoá 5 box (img_001: 3 bicycle; img_020: 2 person), giữ 3 person nhỏ (img_001: 2, img_036: 1) vì vẫn nhìn rõ là người (ca khó, xem CAL-01). Sửa mất 2 phút (`time_log.csv` dòng `gold_fix`). Gold cuối: 73 box.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11 Home, không GPU (torch 2.15 CPU), Python 3.12.10 trong venv riêng (Python mặc định của máy là 3.14, ngoài khoảng 3.10–3.13 nên không dùng); ultralytics 8.4.215; model: yolo26m (tải thêm bằng `download_models.py --set core`), yolo26s, yoloe-26s-seg, sam2.1_b, mobile_sam; chạy hết trên CPU, không cần Colab (yolo26m ~0.9–1.2 s/ảnh, YOLOE ~0.4 s/ảnh, YOLO26+SAM 2.1 ~8.4 s/ảnh); CVAT v2.82.1 chạy bằng Docker (dùng lại stack CVAT có sẵn từ các lab trước thay vì bản 2.81.0 để không hạ cấp database). <<vướng mắc khác nếu có>>
- Dùng cache `data/cache/` thay vì tự chạy? Không: tự chạy `y26_all` và `yoloe_all` trên máy (CPU).
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
