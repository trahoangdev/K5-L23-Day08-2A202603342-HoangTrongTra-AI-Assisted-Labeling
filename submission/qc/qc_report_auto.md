# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 175 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 14/175 = 8.0% | 175 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 7/8 = 87.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 446/518 = 86.1% / 14/518 = 2.7% / 57/518 = 11.0% | 518 box máy |
| Máy: added | 45/505 = 8.9% | 505 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 7 | img_003.jpg, img_006.jpg, img_013.jpg |
| sai_vi_tri_kich_thuoc | 5 | img_003.jpg, img_014.jpg, img_017.jpg |
| thua | 1 | img_003.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 7 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **98%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 87.5% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 7/14 lỗi

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
