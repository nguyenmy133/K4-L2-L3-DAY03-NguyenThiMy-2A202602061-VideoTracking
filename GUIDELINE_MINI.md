# Mini annotation guideline — Ngày 3 (tracking)

> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Họ và tên: `Nguyễn Thị My`  
MSSV: `2A202602061`  
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của người gán:
- Tuyệt đối không gán xe 2 bánh (mô tô, xe máy, xe điện 2 bánh) dù xuất hiện trên làn đường.
- Không gán các vật thể tĩnh ven đường như quầy hàng, bốt điện dù có hình khối chữ nhật tương tự thân xe.

---

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật áp dụng | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che dưới **25 frame** (~ 2 giây @ 12.5 fps) | Giữ tính liên tục của track (AssA/IDF1 cao), đúng bản chất vật thể trong thế giới thực. |
| Xe bị che lâu hơn ngưỡng trên | Vẫn giữ ID nếu chắc chắn đó là cùng một phương tiện dựa trên quỹ đạo và màu sắc; nếu mất dấu hoàn toàn và xuất hiện lại không chắc chắn thì ngắt và tạo track mới. | Tránh nối nhầm track khi có nhiều xe cùng loại đi qua điểm che khuất. |
| Xe rời khung hình rồi quay lại | Mặc định: **tạo track mới** | Khi xe đã rời khỏi cảm biến/camera, không giả định phương tiện quay lại là cùng một đối tượng nếu không có ReID global camera. |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID riêng biệt cho từng xe theo đúng hướng di chuyển (vector vận tốc) trước lúc giao nhau. | Tránh lỗi ID Switch (IDSW) cực kỳ nghiêm trọng trong MOT. |

---

## 3. Luật bbox

| Tình huống | Luật áp dụng |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bounding box ôm sát phần nhìn thấy và chạm đúng rìa ảnh, tuyệt đối không suy đoán phần thân xe nằm ngoài khung hình. |
| Xe bị xe khác che một phần | Bounding box ôm phần **nhìn thấy được (visible object)**, không vẽ tràn sang phần bị che khuất. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định rõ ràng là xe bốn bánh (thấy rõ hình dáng đầu xe hoặc thân xe, không gán khi chỉ là vệt mờ ở đường chân trời). |
| Xe đang đỗ, không di chuyển | Gán và giữ nguyên bounding box cố định nếu là xe 4 bánh hợp lệ; không bấm Outside trừ khi xe bị che hoặc rời khung hình. |
| Keyframe đặt dày ở đâu | Đặt dày ở: (1) Xe bắt đầu rẽ/đổi hướng, (2) Xe tăng tốc hoặc giảm tốc đột ngột, (3) Xe đi qua vùng bị che khuất một phần. |

---

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1: Xe xuất hiện từ xa ở rìa trên khung hình
- **Clip / frame / ID:** `clip_01` / Frame `72 - 78` / Track `7` (sau rework tương ứng Gold Track `5`).
- **Tình huống:** Xe bắt đầu nhú ra từ góc xa ở phía trên ngã tư, ban đầu chỉ là một chấm sáng mờ khó phân biệt xe máy hay ô tô con.
- **Quyết định:** Không gán vội ở frame 72-78. Bắt đầu vẽ track từ frame 79 khi xe đã hiện rõ hình dạng đầu xe 4 bánh.
- **Lý do:** Gán quá sớm khi chưa xác định được loại phương tiện sẽ tạo ra nhiều bbox thừa (False Positives) và làm giảm MOTA.

### Ca 2: Xe buýt lớn rẽ và bị biển báo che khuất
- **Clip / frame / ID:** `clip_01` / Frame `85 - 100` / Track `6` (Gold Track `4`).
- **Tình huống:** Chiếc xe buýt dài di chuyển qua khúc cua và bị cột đèn/biển báo giao thông che một phần thân xe.
- **Quyết định:** Giữ nguyên ID cho toàn bộ hành trình của xe buýt. Bbox ở các frame bị che chỉ ôm phần thân xe nhìn thấy được.
- **Lý do:** Thời gian che khuất ngắn (< 15 frame) và kích thước xe buýt rất lớn, quỹ đạo rõ ràng nên việc giữ ID giúp đạt IDF1 tuyệt đối.

### Ca 3: Xe đi khuất khỏi mép dưới khung hình
- **Clip / frame / ID:** `clip_01` / Frame `148 - 151` / Track `6`.
- **Tình huống:** Xe buýt di chuyển xuống mép dưới và rời hẳn khung hình tại frame 148.
- **Quyết định:** Bấm phím `Outside` (`O`) ngay tại frame 149 để ngắt track hoàn toàn.
- **Lý do:** Nếu không bấm Outside, CVAT sẽ giữ bbox đứng im ở mép ảnh cho tới hết clip, tạo ra 3-4 frame bbox treo (ghost boxes) làm giảm độ chính xác.

---

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật được rút ra và chuẩn hóa lại:
1. **Quy tắc Outside khi kết thúc track:** Luôn kiểm tra frame cuối cùng của mỗi track bằng cách tua từng frame (phím `F`/`D`) và bấm `O` ngay frame sau khi xe hoàn toàn ra khỏi khung.
2. **Quy tắc phân loại phương tiện (Classes):** Chỉ tập trung vào 4-wheel vehicles (bỏ qua xe máy/người ở góc trái frame 1-90).
3. **Quy tắc bắt đầu track:** Không đón đầu xe khi còn là chấm mờ, chỉ kích hoạt track khi nhận diện chắc chắn là xe 4 bánh.
