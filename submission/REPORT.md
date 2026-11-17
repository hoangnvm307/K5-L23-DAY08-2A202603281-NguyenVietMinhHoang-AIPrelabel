# Báo cáo Day 08 - 2A202603281 - Nguyễn Viết Minh Hoàng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26s | 0.30 (mặc định) | Recall cao nhất 27/40 (67,5%), dưới mục tiêu 90%. |
| bicycle | YOLO26s | 0.30 (mặc định) | Recall cao nhất 1/6 (16,7%), dưới mục tiêu 90%. |
| motorbike | YOLO26s | 0.30 (mặc định) | Recall cao nhất 12/28 (42,9%), dưới mục tiêu 90%. |
| car | YOLO26s | 0.30 (mặc định) | Recall cao nhất 7/8 (87,5%), dưới mục tiêu 90%. |
| bus | YOLO26s | 0.30 (mặc định) | Chỉ có 1 vật gold, dưới mức tối thiểu 5 vật để chọn ngưỡng riêng. |
| truck | YOLO26s | 0.30 (mặc định) | Có 0 vật gold, nên chưa đo được recall cho lớp này. |
| three_wheeler | YOLOE-26s (`tuk-tuk`, `cyclo`, `cargo tricycle`) | 0.30 (mặc định) | Recall cao nhất 6/7 (85,7%), dưới mục tiêu 90%. |

Part 2 thử hai model trên pilot (`r01`, `r02`, `r03` trong `model_log.csv`). Để quét ngưỡng, cả hai model chạy trực tiếp trên CPU với `imgsz=640` và `conf=0.05`. YOLO26s sinh 1.288 box trên 36 ảnh và YOLOE-26s sinh 116 box trên 36 ảnh (`r05`, `r06` trong `work/runs.csv`). Các tỷ lệ trong bảng lấy từ `qc/threshold_sweep.csv`, ghép box cùng lớp với IoU ≥ 0,50 trên 4 ảnh gold. Khi gộp ở Part 4, đặt `--conf 0.3` cho cả 7 lớp.

Lệnh gộp đã chạy trong PowerShell với `$p` trỏ tới Python của môi trường Day 08.

```powershell
& $p -B scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --split 'batch,gold' --out work/prelabel/merged_review/pre.coco.json
```

- Part 5 dùng MobileSAM (`mobile_sam.pt`, biến thể SAM 1). Trọng số có sẵn và chạy trực tiếp trên CPU.
- Ở `img_001.jpg`, box máy có điểm `0.8399` và trùng rất sát với nhãn gold (`IoU ≈ 0.975`), nhưng máy gọi `truck` còn gold là `car`. Sửa bằng cách đổi lớp trong CVAT. Ngưỡng confidence chỉ quyết định giữ hay bỏ box.
- Giới hạn của phép quét: chỉ có 4 ảnh gold, số vật mỗi lớp ít và chênh lệch. Dùng cùng bộ ảnh để thử ngưỡng và đánh giá có thể làm số đo lạc quan. Với kết quả lần này, cả 7 lớp đều dùng ngưỡng mặc định.

## 2. AI error profile (từ `qc/review_stats.md`)

File `prelabel/pre_traps_import.coco.json` đã được nạp bằng COCO 1.0 vào task `batch-review` (#16). Bản export `batch-review.zip` chứa 337 box trên 24 ảnh và được lưu nguyên nội dung JSON vào `labels/batch_reviewed.json`. Cả 337 box khớp với file import theo tên ảnh, lớp và tọa độ `bbox` làm tròn đến 0,01 px. `review_stats.py` lọc các bẫy và box lân cận trước khi tính các tỷ lệ trong bảng.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 327 | 327/327 = 100,0% | 327 box máy |
| edited | 0 | 0/327 = 0,0% | 327 box máy |
| deleted | 0 | 0/327 = 0,0% | 327 box máy |
| added | 0 | 0/327 = 0,0% | 327 đối tượng trong bản export |

- Ở ngưỡng `0.30`, máy khớp `0/6` xe đạp gold, gồm ba xe trong `img_001.jpg` và ba xe trong `img_036.jpg`. Lớp `bus` cũng có recall `0/1`, với một vật gold duy nhất.
- Trên 4 ảnh gold, pre-label đã gộp ở `work/prelabel/merged_review/pre.coco.json` có precision **36/44 = 81,8%** và recall **36/90 = 40,0%**. Ghép một-một theo lớp với `IoU ≥ 0.50`, áp dụng `ignore_region` như trong `scripts/review_stats.py`. Khoảng tin cậy Wilson 95% lần lượt là 68–90% và 30–50%. Các giá trị này chỉ đo nhãn máy trên tập gold dùng để thử ngưỡng.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được **0/12 = 0,0%**. Kết quả theo từng loại đều là **0/3** gồm bỏ sót, box thừa, sai lớp và box lỏng. `qc/review_stats.md` có danh sách theo ảnh. Bản export giữ nguyên các box nhập vào. Thao tác duyệt thực tế trong CVAT chưa được xác nhận.
- Tỷ lệ `accepted` **327/327 = 100,0%** cùng với tỷ lệ bắt bẫy **0/12** cho thấy chỉ đếm box giữ lại sẽ đánh giá quá cao chất lượng. Các bẫy tương ứng với bốn bước cần kiểm lại trên ảnh gốc: tìm vật sót, xoá box thừa, xác minh lớp và chỉnh cạnh box.

## 4. Máy có đáng dùng không?

- Gán tay đã đo trong `time_log.csv` là **23 phút / 4 ảnh = 5,75 phút/ảnh**. Chưa có thời gian thực đo của lượt duyệt batch để tính mức tiết kiệm. Khi có số liệu, cần lưu ý lượt gán gold diễn ra trước và hai tập có mật độ đối tượng khác nhau.
- Pre-label có thể dùng để đưa ra box gợi ý. Trên gold, **36/44** box gợi ý khớp nhưng chỉ tìm được **36/90** vật. Khi dùng cho batch, cần dành một lượt quét độc lập để bổ sung vật sót.
- **Ba quy tắc đề xuất cho lượt duyệt lại**
  1. Ẩn box máy và quét toàn ảnh trước khi bật lại nhãn ở cả 24/24 ảnh.
  2. Đối chiếu lần lượt vật sót, box thừa, lớp và bốn cạnh box trên từng ảnh. Ghi một dấu kiểm cho mỗi bước, kể cả khi giữ nguyên nhãn.
  3. Chia lô thành nhóm 10, 10 và 4 ảnh. Bấm giờ thao tác thực tế và ghi thời gian nghỉ riêng trong `time_log.csv`.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` – tuk-tuk nhìn ngang | MobileSAM (SAM 1) | 7 (4 dương, 3 âm) | Đã tách được phần lớn người lái. Không thấy hàng hóa rõ. | Phủ được mui và thân xe. Còn thiếu một phần bánh trước, sàn để chân và thanh khung mảnh. | [`mask_013.jpg`](masks/mask_013.jpg) |
| `img_004.jpg` – tuk-tuk tiền cảnh bên phải | MobileSAM (SAM 1) | 7 (3 dương, 4 âm) | Hai người đứng cạnh xe ở ngoài mask. Mask phủ một phần vật sáng màu ở ghế sau, chưa rõ đó có phải hàng hóa. | Điểm âm dưới gầm giảm vùng nền bị tô. Mask còn thiếu đèn bên phải và biển TAXI trên mui. | [`mask_004.jpg`](masks/mask_004.jpg) |
| `img_017.jpg` – tuk-tuk xanh gần máy ảnh nhất | MobileSAM (SAM 1) | 8 (5 dương, 3 âm) | Không thấy người lái hoặc hàng hóa rõ trên xe được chọn. | Đổi điểm trên mui thành điểm dương và thêm điểm ở đầu xe, bánh trước giúp lấy đủ các phần lớn. Còn thiếu biển TAXI và một phần khung mảnh ở khoang hở. | [`mask_017.jpg`](masks/mask_017.jpg) |

Mục 4.2 dùng đường biên phần nhìn thấy, giữ kính xe, loại bóng đổ và tách vật chạm nhau. Khoang hở lớn của tuk-tuk là chỗ cần kiểm theo quy tắc lỗ thủng. Mục 4.2 chưa nêu cách xử lý hàng rời trong mask. Quy tắc về hàng hóa nhô ra ở mục 4.1 áp dụng cho box.

## 6. Calibration (Part 6)

- Có 3 quyết định trong [calibration_decisions.csv](qc/calibration_decisions.csv). CAL-01 chốt câu hỏi mở số 7 bằng ca YOLO26s gán nhầm xe tiền cảnh trong `img_001` dù `conf=0.8399` và IoU với gold khoảng 0.975.
- [Guideline v1.1](guidelines/guideline_v1.1_calibrated.md) làm rõ cách dùng ngưỡng: lọc pre-label ở đầu vào, sau đó kiểm tra lớp và các cạnh của mọi box còn lại trên ảnh gốc.
- CAL-02 ghi lỗi sai lớp của YOLO26s trên tuk-tuk và xích lô theo quy tắc v1.0. CAL-03 ghi một box `person` đã được đánh dấu `can_review` trong gold. Cách ước lượng tỉ lệ nhìn thấy ở những ca bị mui che vẫn là câu hỏi mở.

## 7. QC và quyết định (Part 6)

- Mẫu: **5/24 ảnh**, seed 42. Cả 5 ảnh được chọn ngẫu nhiên vì bản export có 0 ảnh được đánh dấu `can_review`. ChatGPT đối chiếu ảnh gốc với ảnh có số ID box. Học viên cần xác nhận lại các phát hiện trong `qc/qc_review.csv`.
- Lỗi theo **đối tượng**: **14/65 = 21,5%** theo cách đếm của script, gồm 9 box có lỗi và 5 vật thiếu box. Lỗi theo **ảnh**: **5/5 = 100,0%**, vượt ngưỡng 2%.
- Ba loại lỗi ghi nhận: **6** lỗi vị trí hoặc kích thước, ví dụ box #337 cắt mui đỏ của xích lô ở `img_035.jpg`. **5** lỗi bỏ sót, ví dụ thiếu người ngồi sát mép phải ở `img_006.jpg`. **3** box thừa, ví dụ box #291 gán áo khoác treo thành `person` ở `img_031.jpg`.
- **Giới hạn mẫu nhỏ**: `qc/qc_report_auto.md` tính cận trên Wilson 95% là **100%** cho kết quả 5/5 ảnh lỗi. Đây là kết quả trên mẫu, chưa đủ để ước lượng chính xác tỷ lệ lỗi của cả 24 ảnh.
- Script kiểm tên ảnh, lớp hợp lệ, tọa độ box và tính tỷ lệ từ phiếu QC. Người kiểm cần so ảnh gốc với box để phát hiện box chưa ôm sát, nhãn sai lớp hoặc vật bị bỏ sót trong cả hai lượt gán.

**Quyết định: Rework**, ngày 2026-11-17. Ảnh trong mẫu đọc được và có thể xác định lỗi theo các quy tắc hiện có. Tỷ lệ ảnh lỗi **5/5 = 100,0%** vượt 2% và tỷ lệ bắt bẫy **0/12 = 0,0%** dưới 80%.

Việc tiếp theo: duyệt lại đủ 24 ảnh trong CVAT theo checklist và bấm giờ. Export lại, tính lại tỷ lệ bắt bẫy, tạo bản ứng viên mới và ghi kết quả QC vòng sau.

## 8. Ghi chú cá nhân

- Hỗ trợ ở bước calibration: ChatGPT đối chiếu ảnh gốc với nhãn gold, đầu ra YOLO26s và model log. Phần giải thích về `conf` trong v1.1 cụ thể hóa cách dùng ngưỡng đã ghi ở mục 1.
- Hỗ trợ ở Part 4–5: ChatGPT chạy script gộp và cài bẫy, tạo task `batch-review` và nạp COCO vào CVAT local, chạy MobileSAM offline, chỉnh điểm click và xem ảnh overlay. File nạp được kiểm bằng `validate_labels.py` và đối chiếu tên ảnh với `work/tasks/review/`. Sau khi nạp, số box toàn task và số box của ảnh đầu được đối chiếu trực tiếp trong CVAT.
- **Dùng AI để làm bài**: ChatGPT kiểm tra ZIP COCO, chạy script review và QC, đối chiếu cả 5 ảnh gốc với ảnh có ID box rồi ghi các lỗi nhìn thấy vào phiếu QC. Đã kiểm đủ tên ảnh bằng `validate_labels.py` và đối chiếu số liệu trên báo cáo với `review_stats.json`, `qc_review.csv` và bản export.
- Gold có bị nhiễm trước khi gán xong hay không cần học viên xác nhận. `time_log.csv` ghi nhận 4 ảnh được gán tay trong CVAT.
- Môi trường: Windows, YOLO26s và YOLOE chạy CPU theo `work/runs.csv`, MobileSAM chạy offline. Cấu hình Compose mặc định `v2.74.1`, cần xác nhận phiên bản CVAT đang chạy.
- Các lượt `r05` và `r06` có đầu ra từ suy luận trực tiếp trong `work/runs.csv`.
- `validate_labels.py` cảnh báo box `car` #239 ở `img_027.jpg` chỉ cao 17,4 px, dưới ngưỡng 20 px. Đây là cảnh báo về nhãn cần sửa trong lượt duyệt lại.
- Thay đổi đề xuất cho lần sau: kiểm ảnh gốc trước khi bật box máy và ghi thời gian ngay trong lượt duyệt.
