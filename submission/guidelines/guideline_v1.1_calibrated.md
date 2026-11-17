# Label contract — Phố châu Á — Bổ sung v1.1 — 2A202603281 - Nguyễn Viết Minh Hoàng

Đọc cùng [label contract v1.0](../../guideline/guideline_v1.0_label_contract.md). Các mã ca trỏ đến [bảng quyết định calibration](../qc/calibration_decisions.csv).

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 7 | Có mức `conf` để chấp nhận box máy ngay không? | **Bổ sung vào mục 3.** Dùng ngưỡng `conf` để lọc đầu vào và dùng điểm để ưu tiên thứ tự duyệt. Mọi box qua ngưỡng đều phải đối chiếu lớp và các cạnh box với ảnh gốc trước khi chấp nhận. Trong lần chạy này, ngưỡng đầu vào là 0.30. | [img_001](../../data/images/img_001.jpg). Box xe tiền cảnh có `conf=0.8399` và IoU ≈ 0.975 so với gold. YOLO26s gán `truck`, còn nhãn gold là `car`. | CAL-01 |

Câu 1–6 của v1.0 còn mở. CAL-03 ghi lại một trường hợp liên quan đến câu 4. Lần calibration này tiếp tục dùng ngưỡng nhìn thấy từ 20% theo v1.0 và giữ cờ `can_review` ở ca cần đối chiếu.

## 2. Ca khó bổ sung vào bảng ở mục 5 của v1.0

| `case_id` | Ảnh | Tình huống và bằng chứng | Cách xử lý | Quy tắc áp dụng |
|---|---|---|---|---|
| CAL-02 | [img_011](../../data/images/img_011.jpg), [img_032](../../data/images/img_032.jpg) | YOLO26s `r01` gán tuk-tuk xanh bị cắt mép trái là `truck` (0.7281). `r02` gán xích lô lưng ghế đen sát lề phải là `bicycle` (0.5183). | Gán mỗi xe một box `three_wheeler`. Với tuk-tuk, ôm phần nhìn thấy và kéo cạnh box đến mép ảnh. | Mục 2 và 4.1.3 của v1.0 đã quy định lớp và cách vẽ box khi vật bị cắt mép. |
| CAL-03 | [img_012](../../data/images/img_012.jpg) | Gold có nhãn `person` số 35, `can_review=true`, tại khoang hở của tuk-tuk ở giữa phía sau. Mui và khung xe che một phần người. | Giữ box `person` theo phần nhìn thấy và giữ cờ `can_review` để đối chiếu mức che khuất. | Áp dụng mục 3, 4.1.2 và 5 của v1.0. Cách ước lượng tỉ lệ nhìn thấy vẫn là nội dung cần calibration thêm ở câu 4. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ cách dùng `conf` ở mục 3 và bổ sung hai ca đối chiếu vào bảng mục 5. | CAL-01 chốt câu 7, CAL-02 ghi lỗi sai lớp của YOLO26s, CAL-03 ghi ca `can_review` trong gold. |
