# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Nguyễn Thị My (2A202602061)` |
| Reviewer | `Peer Reviewer / Self-QC Validator` |
| Pair ID | `PAIR-DAY03-01` |
| CVAT version | `CVAT Online v2.x` |
| Thời điểm review | `2026-09-15` |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 71-77 | 72-78 | 7 | Entry sớm (False Positive) | Bbox vẽ quá sớm từ frame 72 khi xe chỉ là vệt mờ xa. Rule: chỉ bắt đầu track khi rõ xe 4 bánh. | Chuyển điểm bắt đầu về frame 79 (CVAT frame 78). | fixed |
| 2 | 148-150 | 149-151 | 6 | Box treo (Ghost box) | Xe buýt đã đi ra khỏi khung hình ở frame 148 nhưng chưa bấm Outside. Rule: bấm Outside ngay frame xe ra hết. | Bấm Outside (`O`) tại frame 149. | fixed |
| 3 | 0-89 | 1-90 | 4 | Sai phân loại (Class FP) | Gán đối tượng 2 bánh/người ở góc trái. Rule: chỉ gán xe 4 bánh. | Xóa toàn bộ Track 4. | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đủ 8 track xe 4 bánh trong clip_01 |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0, IDF1 > 0.84 |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Xe buýt qua biển báo giữ nguyên ID |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Đã bấm Outside ở các điểm exit |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Bbox khít với LocA > 0.82 |
| Frame giữa hai keyframe không bị interpolation drift | PASS | Đã chèn keyframe tại frame 55, 93, 141 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `check_mot_labels.py` chạy 0 lỗi |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Cả 3 finding đã đóng `fixed` |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Toàn bộ 190 frame không nhảy ID |
| 2 — endpoint/scope | ĐÃ SỬA | Đã loại bỏ track thừa và chỉnh frame bắt đầu/kết thúc |
| 3 — geometry/interpolation | PASS | MOTP đạt 0.820 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Cần bấm `Outside` (`O`) ngay tại frame đầu tiên xe rời hoàn toàn khỏi khung hình để tránh sinh ra ghost bbox làm hỏng chỉ số MOTA.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): Track 2 đứng yên ở frame 1-15 là xe đang dừng chờ rẽ tại giao lộ, đây là chuyển động thực tế chứ không phải lỗi gán nhãn.
3. Một rule cần Lab Coach làm rõ (nếu có): Ngưỡng kích thước tối thiểu đối với xe xuất hiện từ xa ở đường chân trời.
