# Báo cáo Ngày 3 — Tracking Annotation

Họ và tên: `Nguyễn Thị My`  
MSSV: `2A202602061`  
Ngày: `2026-09-15`  
Bài tập cá nhân: Lộ trình Level 2 -> Level 3 (Day 3 - Video Tracking)

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT Online (Rectangle Track mode, nhãn `vehicle`) |
| Thời gian gán `clip_02` (warm-up) | 25 phút (60 frame) |
| Thời gian gán `clip_01` | 55 phút (190 frame) |
| Số track đã vẽ trong `clip_01` | 10 track (Pre-gold) -> 8 track hợp lệ |
| Số keyframe trung bình mỗi track | ~ 4-6 keyframe / track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Phương tiện xuất hiện từ xa ở đường chân trời (Rìa trên khung hình):** Ban đầu xe chỉ là một chấm mờ có kích thước vài pixel rất khó xác định chủng loại. Xử lý: Đợi đến frame xe phóng to và nhận diện rõ ràng là xe 4 bánh (frame 79 và frame 101) mới bắt đầu đặt keyframe đầu tiên.
2. **Xe buýt lớn rẽ và bị che khuất một phần bởi biển báo/cột đèn:** Khi xe buýt dài di chuyển qua điểm che, bbox phải được co lại chỉ ôm lấy phần thân xe nhìn thấy được (visible mask), đồng thời giữ nguyên ID của xe buýt qua toàn bộ 190 frame.
3. **Phương tiện rời khỏi góc dưới khung hình:** Xe đi dần ra khỏi tầm nhìn của camera, cần xác định chính xác frame cuối cùng còn thấy xe và bấm ngay phím `Outside` (`O`) ở frame kế tiếp để tránh bbox bị treo lơ lửng ở mép ảnh.

---

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1 (Chỉ nhìn ID):** Kiểm tra tính liên tục của ID trong suốt clip. Kết quả: Toàn bộ các track giữ nguyên ID, không có ID nào bị nhấp nháy hoặc nhảy ID sang xe khác (0 ID Switch).
- **Lượt 2 (Frame đầu và frame cuối):** Phát hiện 2 track (Track 6 và Track 10) bị treo 3 frame sau khi xe đã ra khỏi ảnh do quên bấm Outside; đồng thời phát hiện Track 7 và Track 8 vẽ hơi sớm khi xe chưa vào hẳn làn.
- **Lượt 3 (Frame giữa 2 keyframe):** Kiểm tra độ khít (interpolation drift). Thêm 1 số keyframe bổ sung tại các frame 55, 93, 141 khi xe bắt đầu đổi hướng di chuyển.

Kiểm chéo với: `Self-QC & Peer Review (PAIR-DAY03-01)`. Chi tiết ở `reports/review_partner.md`.  
Số lỗi bạn tìm được trong bản của bạn ấy: `3 lỗi (entry sớm, ghost box, class FP)`.  
Số lỗi bạn ấy tìm được trong bản của bạn: `3 lỗi (tương tự)`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

- Ca xe dừng đỗ ở góc đường tại frame 1-15: Một bên cho rằng nên bấm Outside vì xe đứng im, bên còn lại giữ nguyên track. Thống nhất: Giữ nguyên track vì xe vẫn hiện diện thực tế trong khung hình và thuộc lớp xe 4 bánh hợp lệ. Luật đã được cập nhật vào `GUIDELINE_MINI.md`.

---

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `d184b4dfba12ecf98164b2a4e989905b39303fbe66f2a220bc08b31c8ce23468` |
| Thời điểm khóa | `2026-09-15T09:55:12.989083+00:00` |
| Số row / frame / track trước khi mở reference | 740 rows / 190 frames / 10 tracks |

| Bản đánh giá | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| **Bản pre-gold** | 0.687 | 0.609 | 0.777 | 0.838 | **0.847** | **0.649** | **0.820** | 184 | 17 | 0 |
| **Sau rework** | **0.882** | **0.801** | **0.970** | **1.000** | **0.890** | **0.752** | **1.000** | 147 | 0 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐÃ ĐẠT TOÀN BỘ (100% PASS CỔNG)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Sai class (Class FP) | 1 - 90 | ID 4 | Xóa bỏ toàn bộ Track 4 (đối tượng không thuộc lớp xe 4 bánh) |
| Sai class (Class FP) | 1 - 36 | ID 5 | Xóa bỏ toàn bộ Track 5 |
| BBox vẽ đón đầu quá sớm | 72 - 78 | ID 7 | Chuyển keyframe bắt đầu về frame 79 khi xe hiện rõ |
| BBox vẽ đón đầu quá sớm | 80 - 100 | ID 8 | Chuyển keyframe bắt đầu về frame 101 |
| BBox treo sau khi rời khung | 149 - 151 | ID 6 | Bấm phím Outside (`O`) tại frame 149 |
| BBox treo sau khi rời khung | 169 - 171 | ID 10 | Bấm phím Outside (`O`) tại frame 169 |

---

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25` / `0.70` / `960` / `[2, 5, 7]` |
| device | `0` (GPU CUDA) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| **Bạn (Rework) vs gold** | 0.882 | 0.801 | 0.970 | 1.000 | 0.890 | 0.752 | 1.000 | 147 | 0 | 0 |
| **ByteTrack control vs gold** | 0.623 | 0.569 | 0.687 | 0.790 | 0.843 | 0.691 | 0.756 | 97 | 83 | 3 |
| **BoT-SORT + ReID vs gold** | 0.691 | 0.633 | 0.764 | 0.819 | 0.887 | 0.769 | 0.791 | 90 | 45 | 2 |
| **ReID vs bạn** | 0.624 | 0.523 | 0.751 | 0.819 | 0.793 | 0.616 | 0.791 | 90 | 194 | 2 |

---

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

* Ở cả hai bản pre-gold và sau rework, IDF1 (0.847 -> 0.890) đều cao hơn MOTA (0.649 -> 0.752).
* Trong bài toán tracking, nếu một hệ thống có MOTA cao nhưng IDF1 thấp, điều đó cho thấy detector tìm vật thể rất tốt ở từng frame riêng lẻ (ít FP, ít FN), nhưng module association liên tục làm đứt gãy hoặc tráo đổi ID của các track.
* Công thức tính $\text{MOTA} = 1 - \frac{\sum(\text{FN} + \text{FP} + \text{IDSW})}{\sum \text{GT}}$. Trong MOTA, mỗi lần xảy ra ID switch chỉ bị tính là 1 lỗi duy nhất (trừ 1 điểm vào tử số), sau đó các frame tiếp theo nếu vẫn khớp vị trí thì vẫn được tính là True Positive. Ngược lại, **IDF1** đánh giá tính toàn vẹn danh tính trên toàn bộ chiều dài track thông qua tỷ lệ $\text{IDTP} / (\text{IDTP} + 0.5(\text{IDFP} + \text{IDFN}))$. Nếu một track dài 100 frame bị đổi ID ở giữa (frame 50), IDF1 sẽ phạt nặng một nửa chiều dài track đó (50 frame trở thành IDFP/IDFN), trong khi MOTA chỉ phạt đúng 1 điểm $\text{IDSW}=1$.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

* **So sánh chỉ số:** BoT-SORT + ReID đạt điểm cao hơn ByteTrack trên mọi tiêu chí danh tính: IDF1 đạt **0.887** (so với 0.843 của ByteTrack), AssA đạt **0.764** (so với 0.687), và FN giảm gần một nửa từ 83 xuống còn 45. Số lần IDSW giảm từ 3 xuống 2.
* **Minh chứng qua Frame Sequence:** Tại đoạn frame 85 - 115 khi xe di chuyển qua khu vực bị cây và biển báo che khuất: ByteTrack chỉ dựa vào Kalman Filter dự đoán chuyển động (motion/IoU) nên khi xe giảm tốc và bị che khuất lâu, bounding box dự đoán bị lệch dẫn đến mất dấu track (FN cao ở track 8 và track 6). Ngược lại, BoT-SORT tích hợp thêm đặc trưng ngoại hình (ReID embedding) và cơ chế Camera Motion Compensation (CMC), giúp nhận dạng lại chính xác thân xe sau khi lộ ra khỏi vật cản, nâng tỷ lệ bao phủ của track rõ rệt.
* *Lưu ý khoa học:* Thí nghiệm này là so sánh cấp hệ thống (system comparison) giữa 2 pipeline tracking hoàn chỉnh, không cô lập tuyệt đối hiệu ứng nhân quả (causal effect) của riêng module ReID do BoT-SORT còn có các cải tiến khác về Kalman Filter state vector và CMC so với ByteTrack.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

* DetA tăng từ 0.569 (ByteTrack) lên 0.633 (BoT-SORT), số lượng False Negatives (FN) giảm gần một nửa từ 83 xuống 45 frame bỏ sót. Số lượng False Positives (FP) giảm từ 97 xuống 90.
* **Bản chất lỗi còn lại:** Lỗi lớn nhất hiện tại thuộc về **Detector** (YOLO26n chạy zero-shot với trọng số COCO tổng quát). Cụ thể:
  1. Detector phát hiện nhầm các vật thể tĩnh ở góc đường (như ID 7 / ID 10 tồn tại 42-43 frame tĩnh) tạo ra ~40 FP.
  2. Bounding box của detector đôi khi chưa ôm khít mép xe ở các góc xa.
  3. Module Association đã hoạt động rất tốt (AssA = 0.764), lỗi còn lại chủ yếu do detector cung cấp đầu vào có nhiễu.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

* **Vị trí:** Frame `16 - 116` / ID `7` của model ReID.
* **Phân tích:** Model ReID duy trì một track (ID 7) suốt 43 frame tại một vật thể tĩnh bên lề đường (quầy hàng/bốt giao thông) do detector nhận nhầm là xe và ReID embedding duy trì liên tục vị trí tĩnh này.
* **Kết luận:** Nhãn của con người (bạn) hoàn toàn chính xác khi không gán vật thể này, trong khi Model bị mắc lỗi False Positive nghiêm trọng do giới hạn của detector.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

* **Vị trí:** Frame `101 - 106` / Track xe con rẽ vào từ ngã tư.
* **Phân tích:** Khi xem lại bất đồng giữa nhãn tay và ReID tại frame 101, ReID đã bắt đầu nhận diện xe rất sớm từ frame 101 và giữ ID ổn định. Nhờ quan sát bbox của ReID, tôi đã nhận ra mình từng vẽ đón đầu quá sớm ở frame 80-100 khi xe chưa lộ diện (ở bản pre-gold), và sau đó điều chỉnh lại điểm bắt đầu chuẩn xác từ frame 101 tương thích hoàn hảo với thời điểm xe thực sự xuất hiện.

---

## 6. Nếu phải gán thêm 10 clip nữa

Nếu phải thực hiện gán nhãn hàng loạt cho các dự án tracking tiếp theo:
1. **Cập nhật `GUIDELINE_MINI.md`:** Bổ sung quy chuẩn rõ ràng về diện tích pixel tối thiểu (ví dụ: chiều cao bbox tối thiểu $\ge 20\text{px}$) để thống nhất thời điểm bắt đầu vẽ track cho xe từ xa; quy định rõ danh mục loại trừ cho các vật thể tĩnh ven đường.
2. **Cải tiến quy trình:** Luôn áp dụng phím tắt kiểm tra frame đầu/cuối (`|<` và `>|`), dùng phím `Outside` (`O`) ngay lập tức khi xe chạm rìa; chạy tool `check_mot_labels.py` ngay sau mỗi clip trước khi chuyển sang clip mới.

---

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
