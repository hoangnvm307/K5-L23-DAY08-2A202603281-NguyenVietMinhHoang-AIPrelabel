# Thông tin học viên

- Họ và tên: Nguyễn Viết Minh Hoàng
- MSHV: 2A202603281
- Lớp (L2 / L3 / khác): L2.
- Tài khoản GitHub: `hoangnvm307`
- Repo bài nộp (URL): https://github.com/hoangnvm307/K5-L23-DAY08-2A202603281-NguyenVietMinhHoang-AIPrelabel
- Commit chốt bài (hash): `8a93e5f41ca63e7016d1838734cfc102eb756cff` (commit chứa nội dung đã kiểm tra).
- Máy: Windows. Các lượt chạy toàn bộ tập ảnh dùng CPU theo `work/runs.csv`.
- CVAT (Docker): cấu hình Compose mặc định `v2.74.1`. Cần xác nhận phiên bản thực tế trong CVAT.

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted **327/327 = 100,0%**, edited **0/327 = 0,0%**, deleted **0/327 = 0,0%** trên các box máy ngoài vùng bẫy. Added **0/327 = 0,0%** trên các đối tượng export ngoài vùng bẫy. Nguồn `submission/qc/review_stats.md`.
- So với 4 ảnh gold: precision **36/44 = 81,8%** và recall **36/90 = 40,0%**, tính từ `submission/prelabel/pre_traps.coco.json` và `submission/labels/gold_manual.json` theo cách ghép của `scripts/review_stats.py`.
- Bắt bẫy: **0/12 = 0,0%** theo `submission/qc/review_stats.md`.
- Thời gian theo `submission/time_log.csv`: gán tay **23 phút / 4 ảnh = 5,75 phút/ảnh**. Duyệt batch có pre-label **72 phút / 24 ảnh = 3 phút/ảnh**.
- Lỗi QC: **5/5 ảnh = 100,0%** và **14/65 theo đối tượng = 21,5%** theo `submission/qc/qc_report_auto.md`. Tử số theo đối tượng gồm 5 vật bỏ sót.
- **Quyết định: Rework.** Tỷ lệ ảnh lỗi 5/5 vượt ngưỡng 2% và tỷ lệ bắt bẫy 0/12 dưới ngưỡng 80%.
