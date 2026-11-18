# Báo cáo Day 08 — 2A202603201 - Bùi Quang Huy

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m (r02, r06) | 0.15 | Hạ từ 0.3 xuống 0.15: recall trên gold tăng 24/37 → 29/37 (65% → 78%), box thừa chỉ tăng 10 → 15; xoá box thừa nhanh hơn tự vẽ vật sót. Không ngưỡng nào đạt recall 90%. |
| motorbike | YOLO26m (r02, r06) | 0.2 | Cân bằng: ở 0.2 sót 6/16, thừa 3 (0.15: sót 5 nhưng thừa 5; 0.3: sót 7). Không ngưỡng nào đạt recall 90%. |
| car | YOLO26m (r02, r06) | 0.4 | Recall không đổi 7/8 từ 0.1 đến 0.5, box thừa giảm 7 → 4 khi lên 0.4: tăng ngưỡng không mất box đúng. |
| bicycle / bus / truck | YOLO26m (r02, r06) | 0.3 (mặc định) | gold chỉ có 2 / 1 / 0 vật (< 5), không đủ để chọn ngưỡng riêng (`threshold_sweep.md`) |
| three_wheeler | YOLOE-26s, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` (r04, r07) | 0.15 (script gợi ý) | YOLO26 không có lớp này (r02: tuk-tuk / xích lô bị sót hoặc gọi car / truck / motorbike / bicycle); YOLOE ở 0.15 bắt 7/7 xe ba bánh trên gold, precision 7/9 = 78%. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,motorbike=0.2,car=0.4,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 759 box trên 28 ảnh (24 batch + 4 gold); sau đó `inject_traps.py --student "2A202603201"` cài 12 bẫy.
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 b (`sam2.1_b.pt`): bản khuyến nghị cân bằng chất lượng / tốc độ; chạy được trên CPU (~8.4 s/ảnh ở r05, chấp nhận được cho vài vật); SAM 3 cần GPU và xin quyền nên không dùng.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Xe ba bánh bị YOLO26 gọi sai lớp: img_002 tuk-tuk bên phải là `truck` 0.73 (điểm cao vẫn sai), tuk-tuk lớn giữa ảnh chỉ ra `truck` 0.16 (hạ ngưỡng chỉ biến "sót" thành "sai lớp"); img_007 xích lô → `motorbike`, img_032 xích lô → `bicycle` (r02, r03). Nguyên nhân: YOLO26 học trên COCO, không có lớp xe ba bánh. Sửa: lấy lớp `three_wheeler` từ YOLOE và khi review đổi lớp mọi box trên xe ba bánh (CAL-02).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_002 tuk-tuk bị gọi `truck` 0.73; điểm thấp mà sai: img_031 box `person` 0.15 trên áo khoác treo (QC #574), img_003 box `truck` 0.25 trùng (QC #18); điểm thấp mà đúng: tuk-tuk lớn ở img_002 được YOLOE gọi đúng nhưng YOLO26 chỉ cho 0.16.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: precision / recall 68.8% / 77.5% trên gold là in-sample, thực tế trên batch có thể thấp hơn; mỗi lớp chỉ vài chục vật hoặc ít hơn (car 8, three_wheeler 7, bicycle 2, bus 1, truck 0) nên lệch 1 vật đổi số đo 3–14%; bicycle / bus / truck không đủ mẫu để chọn ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 493 | 75.3% | 655 box máy (đã loại 25 đối tượng quanh bẫy) |
| edited | 66 | 10.1% | 655 box máy |
| deleted | 96 | 14.7% | 655 box máy |
| added | 97 (12 là đổi lớp → máy sót thật ≈ 85) | 14.8% | 656 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler`: chỉ 6 accepted so với 25 deleted và 12 edited (cộng 5 added). YOLO26 không có lớp này nên các box `car` / `truck` / `bicycle` trên xe ba bánh phải xoá hoặc đổi lớp (img_004, img_013), còn YOLOE thì box lỏng và có box thừa (img_002, img_011 trong r04). Sau đó là `car`: 24 added trên 39 accepted, máy sót nhiều ô tô.
- So với gold: precision / recall của **máy** 55/80 = 68.8% (KTC 95%: 58–78%) / 55/71 = 77.5% (KTC 95%: 66–86%) (mẫu số: 71 đối tượng gold, không tính ignore_region; 80 box máy trên 4 ảnh gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 9/12 = 75.0% (KTC 95%: 47–91%, dưới ngưỡng 80%); loại nào lọt: extra 1/3 (img_006: box `truck` giả), loose 1/3 (img_025: box `motorbike` bị kéo lệch), wrong_class 1/3 (img_008: `person` bị đổi thành `bicycle`); missing bắt 3/3; *(Số đo trên là của lần duyệt đầu, trước khi mở `traps_key.json`. Sau khi export và xem đáp án, tôi đã sửa 3 bẫy lọt (img_006 xoá box `truck` giả; img_008 đổi lại `person`; img_025 siết box `motorbike`). Bản sửa nằm trong `labels/qc_fixed.json` và chỉ được dùng cho `release/`; `batch_reviewed.json` và `review_stats.*` giữ nguyên lần duyệt đầu để số bắt bẫy không bị thổi phồng.)* vì sao (bạn đã bỏ bước nào của checklist)? (1) Tôi không ẩn nhãn máy để quét ảnh trước mà nhìn thẳng vào box máy (bỏ bước 0); (2) box nhỏ / ở xa chỉ nhìn lướt, không kiểm tên lớp từng box — nên `person` bị đổi thành `bicycle` ở img_008 (box chỉ ~30×46 px) lọt qua (bước 3, sai lớp); (3) box trông đúng chỗ, đúng cỡ thì tôi bấm qua, không kiểm cạnh có ôm sát không — nên box lỏng img_025 lọt (bước 4) và box `truck` giả ở img_006 nằm cạnh xe thật trông hợp lý nên không bị xoá (bước 2, thừa); (4) duyệt 24 ảnh trong khoảng 60 phút (2.5 phút/ảnh, nhanh hơn mức ~3 phút GUIDE gợi ý), không nghỉ giữa chừng nên cuối lô vội. Cả 3 bẫy missing đều bắt được: tôi tìm vật sót tốt hơn là nghi ngờ box máy đã có.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 75.3% (493/655) đi cùng bắt bẫy 75% (9/12) cho thấy một phần "accepted" là do tôi tin box máy (thiên kiến tự động hoá), không phải do đã kiểm: cả 3 bẫy lọt đều là box máy **có sẵn** (thừa, lỏng, sai lớp), loại lỗi phải nghi ngờ nhãn máy mới thấy. QC cũng xác nhận điều này: 4/6 lỗi còn lại là box thừa của máy tôi chưa xoá. Vì vậy accepted 75% không có nghĩa máy đúng 75%.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 27/4 = 6.75 phút/ảnh (29/4 = 7.25 nếu tính 2 phút sửa gold) vs duyệt 60/24 = 2.5 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): so sánh này nghiêng về phía máy — khi duyệt tôi đã quen giao diện CVAT và quen loại cảnh sau 4 ảnh gold; 4 ảnh gold và 24 ảnh batch khác nhau về độ dày vật; và duyệt nhanh một phần là do tôi đã bỏ bớt bước kiểm (xem mục 3), nên 2.5 phút/ảnh là con số lạc quan.
- Kết luận: **có, trừ `three_wheeler`**, dựa trên: duyệt 2.5 phút/ảnh so với gán tay 6.75 phút/ảnh (nhanh ~2.7 lần) và accepted 75.3% với person / motorbike / car; nhưng `three_wheeler` chỉ 6 accepted so với 25 deleted + 12 edited nên phải kiểm từng box; và vì recall của máy trên gold chỉ 55/71 = 77.5% (máy sót ≈ 85 vật trong batch) nên vẫn phải quét cả ảnh tìm vật sót, không chỉ sửa box máy.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Ẩn nhãn máy và đếm trước:** mỗi ảnh ẩn nhãn máy, đếm số vật từng lớp và ghi ra, rồi mới bật nhãn máy; số box cuối lệch số đếm thì xem lại. Kiểm tra được: có bảng đếm cho 24/24 ảnh.
  2. **Kiểm từng box theo danh sách:** dùng danh sách object bên phải CVAT, bấm qua từng box và xác nhận lớp + cạnh, kể cả box nhỏ / ở xa. Kiểm tra được: bắt bẫy ≥ 10/12 (≥ 80%) ở lần sau, đặc biệt bẫy sai lớp và lỏng.
  3. **Tối thiểu ~3 phút/ảnh, nghỉ sau mỗi 10 ảnh:** không duyệt nhanh hơn mức GUIDE gợi ý. Kiểm tra được: `time_log.csv` ≥ 72 phút cho 24 ảnh, ghi rõ các lần nghỉ.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk) | SAM 2.1 b (`sam2.1_b.pt`) | 2 dương + 1 âm (click âm đặt trên người lái) | Có: mask vẫn dính một phần người lái dù đã click âm lên người | Mask phủ trọn xe + mui, nhưng một click âm không đủ tách người lái ngồi sát khung xe | `tuktuk_img013.jpg` |
| img_003 (xe ba bánh bán hoa quả) | SAM 2.1 b | 1 dương + 1 box | Có: gồm cả hoa quả (đúng CAL-04) và cả hai người ngồi trên xe (sai) | Box gợi ý bao cả người nên SAM gộp người vào mask xe; cần thêm click âm lên từng người | `three_wheeler_img003.jpg` |
| img_017 (tuk-tuk, lần 1) | SAM 2.1 b | 1 dương, không box | Không (mask quá nhỏ) | Một click chỉ ra một mẩu nhỏ của xe (0.4% ảnh): "một click, nhiều mask" — SAM chọn phần nhỏ nhất quanh điểm click | `three_wheeler_img017.jpg` |
| img_017 (tuk-tuk, lần 2) | SAM 2.1 b | 3 dương + 1 box | Không có người lái trong xe; không có hàng | Thêm 2 click + box thì mask phủ gần đủ xe (12.6% ảnh), gồm mui và bánh | `three_wheeler_img017_box.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc "hai vật chạm nhau mỗi vật một polygon, không chồng" (4.2.4) và "người là vật riêng" (mục 5) cho biết mask img_013 và img_003 đang sai vì gộp người vào xe. Guideline chưa nói rõ: (a) hàng chất trên xe có thuộc mask không — tôi áp CAL-04 (hàng trong thùng thuộc xe); (b) cần bao nhiêu click / có nên dùng box gợi ý — img_017 cho thấy 1 click là không đủ, nên thêm quy tắc "mỗi vật ≥ 3 click dương + click âm lên người lái / người ngồi trên xe".
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thử (không bắt buộc); chỉ dùng `sam2_points.py` offline.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`: CAL-01…04; 1 `loi_may`, 3 `cau_hoi_mo`); quan trọng nhất: **CAL-02** — xe ba bánh luôn là `three_wheeler` bất kể máy gọi là gì, vì đây là lớp máy tệ nhất (6 accepted, 25 deleted, 12 edited trong `review_stats.md`) và lỗi sai lớp này không sửa được bằng ngưỡng.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu hỏi mở 3 (hàng trong thùng tính vào box xe, nhô ra > ~20% thì không — CAL-04, img_003) và 6 (mỗi vật một box, giữ box ôm sát nhất — CAL-03, img_003 / img_006); quy tắc xe ba bánh (CAL-02, img_004 / img_007 / img_032); nới 4.1.6 cho người đứng riêng nhận ra rõ cao 17–20 px (CAL-01, img_001 / img_036).
- Nguyên nhân gốc: **máy** cho CAL-02 (YOLO26 học trên COCO, không có lớp xe ba bánh; guideline v1.0 đã đúng, chỉ cần nhấn mạnh khi review); **guideline mơ hồ** cho CAL-01, CAL-03, CAL-04 (v1.0 chưa nói rõ / còn để câu hỏi mở). Lỗi "thừa" trong QC (box trùng sót lại) thì phần lớn là **tôi duyệt sót**, không phải guideline sai.

## 7. QC và quyết định (Part 6)

- Mẫu: 3 ảnh (0 ảnh `can_review` + 3 ngẫu nhiên, seed 42: img_003, img_006, img_031; 68 đối tượng; giảm từ 5 xuống 3 ảnh do thiếu thời gian, theo GUIDE "nếu bị trễ giờ"); người kiểm: tôi (**hạn chế**: tôi vừa là người duyệt).
- Lỗi theo **đối tượng**: 6/68 = 8.8% · theo **ảnh**: 3/3 = 100% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: thừa 4/6 (img_003 #18 box truck trùng; img_003 #20 vật chỉ thấy một phần, không nên gán; img_006 #91 box motorbike trùng; img_031 #574 box person trên áo khoác treo), sai vị trí/kích thước 2/6 (img_003 #6 box person chỉ bao đầu; img_006 #111 box motorbike quá rộng). Không có bỏ sót trong mẫu.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 3/3 ảnh lỗi, cận trên 95% (Wilson) = 100%; với 3 ảnh thì kể cả 0 lỗi cũng chỉ cho cận trên 56%, nên mẫu này không thể chứng minh ngưỡng ≤ 2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script đếm được box thêm / xoá / sửa so với pre-label, bắt bẫy, khớp IoU với gold; script **không** kiểm được: vật mà cả máy và tôi cùng bỏ sót (không có box nào để so), box có ôm sát thật không ở ảnh không có gold, và tôi có thật sự làm đủ checklist 4 bước hay không; QC bằng mắt chỉ trên 3/24 ảnh, do chính tôi kiểm.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đủ để gán (các câu hỏi mở gặp phải đã chốt ở v1.1) và dữ liệu không hỏng; Rework vì (1) ảnh có lỗi 3/3 = 100% > ngưỡng 2%, (2) lỗi lặp lại: "thừa" chiếm 4/6 lỗi (box trùng, box máy conf thấp chưa xoá), (3) bắt bẫy 9/12 = 75% < 80% nên lô duyệt chưa đáng tin. Không thể Release.
Nếu Rework / Hold, việc cần làm tiếp: duyệt lại cả 24 ảnh theo guideline v1.1, tập trung (a) xoá box trùng / box thừa conf thấp (CAL-03), (b) soát box trên xe ba bánh (CAL-02), (c) siết box lỏng; dùng `ignore_region` cho cụm xe nhỏ ở 6 ảnh dày; sau đó QC lại với mẫu lớn hơn (≥ 5 ảnh, gồm ca khó) và người kiểm khác nếu có.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code (trợ lý trong VS Code): khôi phục môi trường và chạy các script theo GUIDE; làm trang HTML xem ảnh `viz/` để tôi so sánh các run; chuẩn hoá chính tả / thuật ngữ cột `what_went_right` / `what_went_wrong` trong `model_log.csv` (nội dung quan sát là của tôi), viết nháp cột `decision` và nhận xét r06 / r07 từ output script; tôi đã đọc lại, đối chiếu với ảnh `viz/` và sửa. Ở Part 4–6, Claude chạy `review_stats` / `qc_report`, điền số liệu từ output script vào REPORT và STUDENT.md, đổi mã `error_type` trong `qc_review.csv` theo ghi chú của tôi; Claude giải thích cây Hold → Rework → Release và đưa các phương án calibration kèm khuyến nghị, tôi chọn (Rework; CAL-02/03/04), Claude viết thành câu trong `calibration_decisions.csv`, `guideline_v1.1_calibrated.md` và REPORT mục 6–7. Tương tự, nguyên nhân bẫy lọt, kết luận "máy có đáng dùng", 3 quy tắc giảm thiên kiến và nhận xét mask (mục 3–5) là tôi chọn trong các phương án Claude đưa ra (sau khi tự mở ảnh mask kiểm tra), Claude viết thành câu; lý do ngưỡng ở mục 1 do Claude giải thích từ `threshold_sweep.csv`, tôi đồng ý chọn phương án ngưỡng theo lớp.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không: chưa chạy model nào trước khi export gold. Sau lần export đầu (78 box), `validate_labels` cảnh báo 8 box cao < 20 px; tôi xem lại ảnh, xoá 5 box (img_001: 3 bicycle; img_020: 2 person), giữ 3 person nhỏ (img_001: 2, img_036: 1) vì vẫn nhìn rõ là người (ca khó, xem CAL-01). Sửa mất 2 phút (`time_log.csv` dòng `gold_fix`). Gold cuối: 73 box.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11 Home, không GPU (torch 2.15 CPU), Python 3.12.10 trong venv riêng (Python mặc định của máy là 3.14, ngoài khoảng 3.10–3.13 nên không dùng); ultralytics 8.4.215; model: yolo26m (tải thêm bằng `download_models.py --set core`), yolo26s, yoloe-26s-seg, sam2.1_b, mobile_sam; chạy hết trên CPU, không cần Colab (yolo26m ~0.9–1.2 s/ảnh, YOLOE ~0.4 s/ảnh, YOLO26+SAM 2.1 ~8.4 s/ảnh); CVAT v2.82.1 chạy bằng Docker (dùng lại stack CVAT có sẵn từ các lab trước thay vì bản 2.81.0 để không hạ cấp database). Thư mục cài đặt `day08` (venv + model) từng bị xoá nhầm vào Thùng rác, đã khôi phục lại.
- Dùng cache `data/cache/` thay vì tự chạy? Không: tự chạy `y26_all` và `yoloe_all` trên máy (CPU).
- Sửa nhãn sau khi xem đáp án bẫy: 3 bẫy lọt được sửa vào `labels/qc_fixed.json` (dùng cho release), không chạy lại `review_stats`; xem mục 3.
- Lỗi / bug script đã gặp (nếu có): không gặp bug script. Lần đầu điền `qc_review.csv` bằng chữ tự do nên `qc_report.py` bỏ qua cả 5 dòng; đã đổi sang mã `error_type` hợp lệ.
- Một điều bạn làm khác lần sau: luôn ẩn nhãn máy và quét / đếm vật trước khi xem box máy, vì cả 3 bẫy lọt đều là box máy có sẵn mà tôi đã tin theo.
