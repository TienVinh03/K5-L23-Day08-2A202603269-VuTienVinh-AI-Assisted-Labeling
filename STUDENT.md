# Thông tin học viên

- Họ và tên: Vũ Tiến Vinh
- MSHV: 2A202603269
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: TienVinh03
- Repo bài nộp (URL): (https://github.com/TienVinh03/K5-L23-Day08-2A202603269-VuTienVinhAI-Assisted-Labeling)
- Commit chốt bài (hash): eaaed50de83f3bb5d0bec9d4b5fe99836bb13f1b
- Máy: HP Victus 16-e0xxx; Windows 11 Home Single Language 64-bit; AMD Ryzen 7 5800H; RAM 20 GB; GPU NVIDIA GeForce RTX 3050 Ti Laptop và AMD Radeon Graphics. Bài hiện chạy CPU với PyTorch 2.15.0+cpu (CUDA không khả dụng trong môi trường .venv hiện tại).
- CVAT (Docker): v2.81.0 (Docker Desktop, Linux/WSL 2), http://localhost:8081

## Kết quả tóm tắt (số từ submission/qc/*)

- Máy: accepted / edited / deleted: 316/416 = 76,0% / 45/416 = 10,8% / 55/416 = 13,2% (mẫu số: 416 box máy); added: 113/474 = 23,8% (mẫu số: 474 đối tượng sau duyệt).
- So với gold: precision / recall của **máy**: 42/62 = 67,7% / 42/47 = 89,4% (4 ảnh gold; 47 đối tượng gold).
- Bắt bẫy: 10/12 = 83,3%.
- Thời gian: gán tay 16/4 = 4 phút/ảnh; duyệt có pre-label 48/24 = 2 phút/ảnh.
- Lỗi QC quan sát được: 0/6 ảnh = 0,0%; 0/103 đối tượng = 0,0%. Tự kiểm sau khi xem lại các box cần phóng to; cận trên Wilson 95% theo ảnh là 39%, chưa chứng minh toàn lô có lỗi ≤ 2%.
- **Quyết định:** RELEASE theo tiêu chí nghiệm thu của bài: 0/6 ảnh lỗi quan sát được ≤ 2%, chưa ghi nhận lỗi hệ thống trong mẫu, bắt bẫy 10/12 = 83,3% ≥ 80%. Giữ giới hạn mẫu nhỏ và hai bẫy missing còn lọt trong REPORT.md mục 7.
