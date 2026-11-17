# Báo cáo Day 08 — 2A202603342 - Hoàng Trọng Trà

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m, `imgsz=1024` | 0.30 mặc định cho từng lớp | Pilot `r01` và `r02` ở `imgsz=640` cho thấy ngưỡng 0.50 loại cả box xe máy đúng 0.46 ở ảnh 011. Lượt CUDA trên 36 ảnh ở `conf=0.05` tạo 1.672 box. Trong dải 0.10–0.50, recall cao nhất vẫn dưới mục tiêu 90% với `person` 22/30, `bicycle` 4/5, `motorbike` 12/18 và `car` 7/8. Gold có 1 `bus` và 0 `truck`, dưới mức tối thiểu 5 vật mỗi lớp (`qc/threshold_sweep.csv`). |
| three_wheeler | YOLOE-26s, `imgsz=1024`, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | Pilot `r03` ở `imgsz=640` tìm được tuk-tuk xanh ảnh 002 và hai xích lô ở ảnh 032. Lượt CUDA trên 36 ảnh ở `conf=0.05` tạo 125 box. Theo `qc/threshold_sweep.csv`, ngưỡng 0.15 đạt recall 7/7 và precision 7/9. Ở ngưỡng 0.30, recall là 5/7. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `.\.venv\Scripts\python.exe scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr 'three_wheeler=0.15' --conf 0.3 --iou 0.6 --split 'batch,gold' --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 Tiny (`sam2.1_t.pt`). Giữ cùng phiên bản và tham số để đối chiếu với lượt CPU. Trên RTX 3050, lượt pilot `r04` ở `imgsz=640` tạo polygon cho 10/10 box ở 2 ảnh, trung bình 3,72 giây/ảnh. Ở ảnh 013, box `person` chứa hai người nên mask lấy vùng của cả hai người.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26n gọi tuk-tuk bị cắt ở mép trái ảnh 011 là `truck` (0.34, `r01`). Đổi ngưỡng chỉ lọc box theo điểm. YOLOE nhận diện xe đó là `three_wheeler` ở `r03`, nên dùng YOLOE để bổ sung lớp này.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Xe máy của người áo hồng ở ảnh 011 có box đúng ở 0.46 trong `r01` và mất khi tăng ngưỡng lên 0.50 trong `r02`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: kết quả có thể giảm khi áp dụng cho 24 ảnh batch. Mẫu ít vật làm tỷ lệ dao động mạnh. Chẳng hạn, nhận đúng 1/1 xe buýt ở ngưỡng 0.30 vẫn chỉ là kết quả trên một xe.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 446 | 86,1% | 518 box máy |
| edited | 14 | 2,7% | 518 box máy |
| deleted | 57 | 11,0% | 518 box máy |
| added | 45 | 8,9% | 505 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler` chỉ giữ nguyên 7/40 box sau khi loại các vùng quanh bẫy. Có 10/40 box sửa hình học và 23/40 box bị xoá. Ở `img_003.jpg`, máy vẽ các box chồng nhau cho mui và thùng của cùng một xe ba bánh. Guideline yêu cầu một box bao phần thấy được của cả xe.
- So với gold: precision / recall của **máy** lần lượt là 51/69 = 73,9% và 51/69 = 73,9% (mẫu số: 69 box máy cho precision và 69 đối tượng gold thuộc lớp mục tiêu trên 4 ảnh cho recall)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 9/12 (75%) ở lượt đầu chưa xem đáp án và 11/12 (91,7%) sau lượt rà theo kết quả chấm; loại nào lọt: lượt đầu lọt 3/3 bẫy `loose`, lượt rà lại sửa được 2/3 ở `img_028.jpg` và `img_029.jpg`; vì sao (bạn đã bỏ bước nào của checklist)? Lượt đầu kiểm chưa kỹ bốn mép box ở vùng có vật chồng nhau. Ở `img_017.jpg`, box gốc của bẫy phủ nhiều xe. Bản cuối giữ một box cho mỗi xe theo guideline, nên script vẫn tính bẫy này là lọt.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 446/518 (86,1%) đi cùng 9/12 bẫy ở lượt đầu. Lượt đầu bắt đủ 3/3 bẫy ở từng loại bỏ sót, thừa và sai lớp. Cả 3 bẫy còn lọt đều ở bước kiểm hình học, nơi cần rà cả bốn mép kể cả khi đã xác định được lớp.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): duyệt 0,35 phút/ảnh (lượt duyệt mất 8,31 phút cho 24 ảnh batch và thêm 0,95 phút để rà lại 3 ảnh).
- Kết luận: có thể dùng pre-label làm điểm bắt đầu để duyệt, dựa trên 446/518 box được giữ nguyên sau khi loại vùng quanh bẫy. Riêng `three_wheeler` cần kiểm từng xe vì chỉ giữ nguyên 7/40 box.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Mở ảnh gốc trước ở 24/24 ảnh, quét bốn góc và đếm xe ba bánh rõ trước khi bật box đề xuất.
  2. Phóng to vùng xe hoặc người chồng nhau, kiểm tra cả bốn mép trên từng box trong vùng trước khi giữ nguyên.
  3. Với xe ba bánh có nhiều box trùng, đối chiếu mui, thân và bánh để giữ đúng một box cho mỗi xe. Tick `can_review` khi ảnh che khuất đến mức chưa chốt được lớp hoặc quy tắc.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_004.jpg` | SAM 2.1 Base (`sam2.1_b.pt`, `r09`) | 11 (6 dương, 5 âm) + box | Hai người đứng cạnh xe nằm ngoài mask. Đã loại phần lớn túi và bình đặt ở bậc xe. | Phủ mái, kính và hai bánh nhìn thấy. Còn vài lỗ nhỏ ở mép ghế. Script ghi 1 polygon. | `tuktuk_img004_sam2.jpg` |
| `img_013.jpg` | SAM 2.1 Base (`sam2.1_b.pt`, `r07`) | 14 (5 dương, 9 âm) + box | Tách được phần chính của người lái sau khi bổ sung điểm âm. | Bắt được mái và bánh xe chính. Mask đứt ở vài đoạn lan can mảnh và còn lấn sang một phần bánh xe máy phía sau. Script ghi 2 polygon. | `tuktuk_img013_sam2.jpg` |
| `img_013.jpg` | MobileSAM (SAM 1, `mobile_sam.pt`, `r08`) | 14 (5 dương, 9 âm) + cùng box | Điểm âm ở đầu, thân và chân loại được phần người lái nhìn thấy. | Mái, bậc lên xuống và bánh chính liền hơn SAM 2.1 trên cùng bộ điểm. Thanh kim loại mảnh vẫn có đoạn mất. Script ghi 2 polygon. | `tuktuk_img013_sam1.jpg` |
| `img_017.jpg` | SAM 2.1 Base (`sam2.1_b.pt`, `r10`) | 10 (5 dương, 5 âm) + box | Hai người đi bộ và xe bên trái nằm ngoài mask. | Phủ mái, ghế và hai bánh nhìn thấy. Còn vài đốm màu ở mép khoảng hở sau ghế. Script ghi 1 polygon. | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc bám phần nhìn thấy, bỏ khoảng hở lớn và bỏ bóng đổ giúp xác định mép xe, nhất là cửa hở ở `img_017.jpg`. Với `img_013.jpg`, hai lượt cùng 2 điểm dương và 1 điểm âm ở thân người lái (`r05`, `r06`) vẫn tô lên đầu hoặc chân. Cần thêm điểm âm trên các phần còn dính. Mục 4.2 chưa chỉ rõ cách rà thanh kim loại mảnh khi thanh nằm sát mép người hoặc xe phía sau.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Thực hiện offline bằng `sam2_points.py` trên RTX 3050. Script lưu ảnh phủ mask cùng dấu click và ghi số polygon. Lượt này chưa thử Interactor trong CVAT.

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

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Học viên sử dụng Codex để đọc hướng dẫn.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Có hỗ trợ AI trước khi chốt nhãn gold. Học viên đã rà lại 4 ảnh trong CVAT. Bản COCO gold giữ 8 cờ `can_review` gồm 2 ở `img_012.jpg`, 4 ở `img_020.jpg` và 2 ở `img_036.jpg`. Số đo 51/69 thể hiện độ khớp với bộ tham chiếu đã rà này.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
