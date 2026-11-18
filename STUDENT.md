# Thông tin học viên

- Họ và tên: Bùi Quang Huy
- MSHV: 2A202603201
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: qhuyyy
- Repo bài nộp (URL): https://github.com/qhuyyy/K5-L23-Day08-2A202603201-BuiQuangHuy-AI-Assisted-Labeling
- Commit chốt bài (hash): <<>>
- Máy: Windows 11 Home, không GPU (CPU; torch 2.15.0+cpu), Python 3.12.10 (venv)
- CVAT (Docker): v2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 493 / 66 / 96 = 75.3% / 10.1% / 14.7% (mẫu số: 655 box máy); added: 97 = 14.8% (mẫu số: 656 đối tượng)
- So với gold: precision / recall của **máy** 55/80 = 68.8% / 55/71 = 77.5%
- Bắt bẫy: 9/12 = 75%
- Thời gian: gán tay 27/4 = 6.75 phút/ảnh; duyệt có pre-label 60/24 = 2.5 phút/ảnh
- Lỗi QC: 100% theo ảnh (3/3), 8.8% theo đối tượng (6/68)
- **Quyết định:** Rework — lý do một dòng: 3/3 ảnh QC có lỗi (> 2%), lỗi "thừa" lặp lại 4/6, bắt bẫy 9/12 = 75% < 80%.
