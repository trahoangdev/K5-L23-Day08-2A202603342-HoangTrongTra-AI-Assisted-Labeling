# Báo cáo Day 08 — 2A202603342 - Hoàng Trọng Trà

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m, `imgsz=1024` | 0.30 mặc định cho từng lớp | Hai lượt pilot với YOLO26n cho thấy ngưỡng 0.50 loại cả box xe máy đúng 0.46 ở ảnh 011 (`r01`, `r02`). Lượt toàn tập với YOLO26m vẫn chưa đủ căn cứ chọn ngưỡng riêng. |
| three_wheeler | YOLOE-26s, `imgsz=1024`, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | Lượt pilot tìm được tuk-tuk xanh ảnh 002 và hai xích lô ở ảnh 032 (`r03`). Lượt quét ngưỡng trên gold đạt recall 7/7 và precision 7/9 ở ngưỡng 0.15. |

- Đầu vào Pha 3 là hai lượt CUDA trên đủ 36 ảnh ở `conf=0.05` và `imgsz=1024`. YOLO26m tạo 1.672 box và YOLOE-26s tạo 125 box. Các nhận xét từ `r01` đến `r04` thuộc lượt thử Pha 2 ở `imgsz=640`.
- Quét ngưỡng trên 4 ảnh gold (nguồn `qc/threshold_sweep.csv`). Recall cao nhất trong dải 0.10–0.50 là `person` 22/30, `bicycle` 4/5, `motorbike` 12/18 và `car` 7/8. Cả bốn lớp đều dưới mục tiêu 90%. `bus` có 1 vật và `truck` có 0 vật, dưới mức tối thiểu 5 vật mỗi lớp. Với `three_wheeler`, ngưỡng 0.15 khớp 7/7 vật với 2 box thừa. Ở ngưỡng 0.30, lớp này chỉ khớp 5/7 vật.
- Tham số gộp cho Pha 4 là `--conf 0.30 --thr "three_wheeler=0.15"`.
- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 Tiny (`sam2.1_t.pt`). Giữ cùng phiên bản và tham số để đối chiếu với lượt CPU. Trên RTX 3050, lượt `r04` tạo polygon cho 10/10 box ở 2 ảnh, trung bình 3,72 giây/ảnh. Ở ảnh 013, box `person` chứa hai người nên mask lấy vùng của cả hai người.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26n gọi tuk-tuk bị cắt ở mép trái ảnh 011 là `truck` (0.34, `r01`). Đổi ngưỡng chỉ lọc box theo điểm. YOLOE nhận diện xe đó là `three_wheeler` ở `r03`, nên dùng YOLOE để bổ sung lớp này.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Xe máy của người áo hồng ở ảnh 011 có box đúng ở 0.46 trong `r01` và mất khi tăng ngưỡng lên 0.50 trong `r02`.
- Giới hạn: dùng cùng 4 ảnh gold để quét ngưỡng và tính precision/recall, nên kết quả có thể lạc quan khi áp dụng cho 24 ảnh batch. Mẫu ít vật làm tỷ lệ dao động mạnh. Chẳng hạn, nhận đúng 1/1 xe buýt ở ngưỡng 0.30 vẫn chỉ là kết quả trên một xe.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

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

- Rà soát CP1: Học viên đã rà soát nhãn của 4 ảnh trong CVAT. Bản COCO còn 8 box có cờ `can_review` gồm 2 ở `img_012.jpg`, 4 ở `img_020.jpg` và 2 ở `img_036.jpg`.
- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
