# Thông tin học viên

- Họ và tên: Hoàng Trọng Trà
- MSHV: 2A202603342
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: trahoangdev
- Repo bài nộp (URL): https://github.com/trahoangdev/K5-L23-Day08-2A202603342-HoangTrongTra-AI-Assisted-Labeling
- Commit chốt bài (hash): Chờ hash sau khi commit Pha 6. Commit đầu vào là `f58f936`.
- Máy: Windows 11 Home Single Language, NVIDIA GeForce RTX 3050 Laptop GPU (4 GB VRAM). Pha 2 chạy trên GPU với PyTorch 2.15.0+cu132 (CUDA 13.2).
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 446/518 (86,1%) / 14/518 (2,7%) / 57/518 (11,0%), mẫu số 518 box máy; added: 45/505 (8,9%), mẫu số 505 đối tượng sau duyệt.
- So với gold: precision / recall của **máy** 51/69 (73,9%) trên 69 box máy / 51/69 (73,9%) trên 69 đối tượng gold.
- Bắt bẫy: 9/12 (75,0%) ở lượt mù, 11/12 (91,7%) sau lượt rà lại.
- Thời gian: gán tay 8 phút/ảnh (32 phút / 4 ảnh gold); duyệt có pre-label bằng AI 0,35 phút/ảnh (8,31 phút / 24 ảnh). Lượt học viên rà gold mất riêng 6 phút/ảnh (24 phút / 4 ảnh).
- Lỗi QC: 87,5% theo ảnh (7/8), 8,0% theo đối tượng (14/175).
- **Quyết định:** Rework — lý do một dòng: 7/8 ảnh mẫu có lỗi, vượt ngưỡng 2%, và bỏ sót chiếm 7/14 lỗi.
