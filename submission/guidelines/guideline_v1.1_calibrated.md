# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603342 - Hoàng Trọng Trà

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện hoặc xe máy điện nhỏ nên gán lớp nào? | Gán `bicycle` nếu thấy bộ bàn đạp hoạt động và kết cấu xe đạp. Gán `motorbike` nếu thấy sàn để chân và kết cấu xe ga. Nếu bị che các dấu hiệu, chọn lớp theo phần thấy rõ nhất và bật `can_review`. Ảnh 009 minh họa hai kiểu hình dáng, chưa xác định xe nào dùng điện. | `img_009.jpg` | `CAL-01` |
| 2 | Có nên gom xe máy đỗ kín ở gần camera? | Vẽ từng xe nếu tách được phần thân dựa trên ít nhất hai dấu hiệu cùng xe, ví dụ yên và bánh. Chỉ dùng ignore_region cho cụm từ 5 vật quá nhỏ hoặc dính đến mức không thể xác định từng xe sau khi phóng to. | `img_026.jpg` | `CAL-02` |
| 3 | Hàng nhô ra khỏi giỏ xe có tính vào box và mask? | Giữ giỏ, giá và thùng gắn cố định. Với box, bỏ phần hàng rời nhô quá khoảng 20% chiều dài xe. Với mask, đi theo bề mặt xe và phần gắn cố định nhìn thấy, tách hàng rời nếu phân biệt được biên. | `img_031.jpg` | `CAL-03` |
| 4 | Nhất quán ngưỡng nhìn thấy người bị che như thế nào? | Giữ ngưỡng khoảng 20% phần cơ thể nhìn thấy. Dùng đầu kèm một phần thân rõ hoặc thân kèm chi làm dấu hiệu hỗ trợ ước lượng. Chỉ thấy một mẩu đầu và rõ ràng dưới ngưỡng thì bỏ. Ca sát ngưỡng thì gán tạm và bật can_review. | `img_013.jpg`, `img_017.jpg` | `CAL-04` |
| 5 | Phần xuyên qua kính hoặc qua cửa hở có thuộc mask xe? | Giữ bề mặt kính lắp trên xe và vùng phản chiếu trên mặt kính. Loại nền nhìn qua cửa hở lớn và phần người lộ trực tiếp trước xe. | `img_004.jpg`, `img_013.jpg` | `CAL-05` |
| 6 | Xử lý thế nào khi nhiều box xe máy chồng nhau trong ảnh đêm? | Đối chiếu yên, đuôi và bánh, nối các phần của cùng xe trước khi xóa box. Giữ hai box nếu thấy hai xe riêng. Nếu một xe có hai box, giữ một box ôm phần thấy được và rà các mép bị che. | `img_026.jpg` | `CAL-06` |
| 7 | Có đặt một điểm conf để bỏ qua việc duyệt box không? | Dùng conf để sắp thứ tự kiểm. Mọi box được giữ vẫn phải qua kiểm lớp, tính riêng biệt và bốn mép. | `img_017.jpg` | `CAL-07` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_003.jpg` | `CAL-08` · YOLOE-26s tách mui và thùng xe bán hàng thành nhiều box three_wheeler. | YOLOE-26s vẽ box cho mui, thùng và cả xe. | Đối chiếu các bộ phận cùng khung xe. Giữ một box cho cả xe và xóa box bộ phận trùng. | Mục 2 và 4.1.1. Mui, khung và thùng gắn liền của cùng một xe ba bánh tính là một đối tượng. |
| `img_013.jpg` | `CAL-09` · Chỉ thấy một dải thân xe sát mép phải, box bus #133 đang có cờ can_review. | YOLOE-26s gợi ý lớp xe ba bánh ở mép. Sau đối chiếu giữ nhãn bus có cờ can_review. | Khi có dấu hiệu nhận dạng như nhiều ô kính liền nhau, gán phần xe nhìn thấy sát mép và giữ can_review nếu vẫn chưa chắc lớp. | Mục 4.1.3 và 4.3. Tách việc cắt khung ảnh khỏi che khuất. Dừng box tại mép ảnh, gắn cờ khi lớp còn mơ hồ. |
| `img_017.jpg` | `CAL-10` · Box nguồn của bẫy loose phủ phần của nhiều tuk-tuk. Phiếu QC cũng thấy #158 lấn sang xe liền kề. | YOLOE-26s có box gộp nhiều xe. Kiểm tra bẫy loose cho thấy box nguồn cũng phủ phần của nhiều xe. | Đếm xe theo mui, thân và bánh, rồi siết từng box đến phần nhìn thấy của đúng xe. | Mục 4.1.1. Với hai xe chạm nhau, kiểm mép tiếp giáp sau khi xác định chi tiết nào thuộc từng xe. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Bổ sung dấu hiệu phân lớp xe, cách kiểm vật che nhau, điều kiện dùng `ignore_region`, và phạm vi box/mask ở vùng kính, hàng và mép tiếp giáp. | Chốt 7 câu hỏi mở (`CAL-01`–`CAL-07`) và thêm quy tắc từ lỗi máy, ca `can_review` và bẫy lọt (`CAL-08`–`CAL-10`). |
