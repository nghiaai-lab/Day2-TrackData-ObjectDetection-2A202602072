# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** CẦN BỔ SUNG  
**MSSV:** CẦN BỔ SUNG  
**Hình thức:** Cá nhân — cần người nộp xác nhận  
**Mã cặp:** `SOLO`

## 1. Phạm vi

- Chỉ gán các phương tiện thuộc bốn lớp `car`, `truck`, `bus`, `van`.
- Mỗi phương tiện là một hộp riêng; không gộp nhiều xe trong cùng một hộp.
- Không gán người, xe máy, xe đạp, biển báo, bóng hoặc phần phản chiếu.
- Không dùng mục tiêu 40–60 hộp làm điểm cắt. Bộ dữ liệu này có **67 vật thể** vì tất cả vật thể đủ điều kiện đều phải được gán.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; đặt `review_state=needs_review` nếu vẫn cần giữ hộp để xin ý kiến.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi; hình dáng khoang hành khách của xe con | xe có thùng/ben hoặc thiết bị công vụ rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ/chuyên dụng rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc có cấu trúc xe buýt rõ | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt dài; khoang hàng tách biệt như xe tải |

Thứ tự lớp bắt buộc trong YOLO là: `0 car, 1 truck, 2 bus, 3 van`. Khi đổi thứ tự trong `data.yaml` phải đồng thời remap class ID trong mọi tệp nhãn.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy; không bao gồm bóng đổ hoặc nhiều nền không cần thiết.
- Không ước lượng phần bị phương tiện khác che khuất.
- Vật thể chạm mép ảnh vẫn được gán nếu phần nhìn thấy đủ để phân lớp; đặt `boundary=truncated`.
- Phân biệt rõ:
  - `occluded`: vật thể bị một vật khác trong cảnh che.
  - `truncated`: vật thể bị chính mép ảnh cắt.
  - Một hộp có thể vừa `occluded` vừa `truncated` nếu thỏa cả hai điều kiện.
- Không để một hộp chứa nhiều phương tiện và không tạo hai hộp trùng cho cùng một vật thể.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Quy tắc sử dụng |
| --- | --- | --- |
| `visibility` | `clear`, `occluded`, `unclear` | Chọn theo mức bằng chứng nhìn thấy; `unclear` chỉ dùng khi hình ảnh không đủ rõ, không dùng thay cho `truncated`. |
| `boundary` | `inside`, `truncated` | `truncated` khi phương tiện bị mép ảnh cắt; nếu toàn bộ phần ngoài của hộp nằm trong ảnh thì chọn `inside`. |
| `review_state` | `confident`, `needs_review` | `needs_review` khi còn phân vân về lớp, phạm vi hoặc hình học và cần người khác xác nhận. |

YOLO không lưu ba thuộc tính này. Vì vậy phải giữ thêm gói `CVAT for images 1.1` từ cùng trạng thái annotation. Kết quả audit hiện tại xác nhận đủ cả ba thuộc tính cho **67/67 hộp**: `clear=60`, `occluded=6`, `unclear=1`; `inside=45`, `truncated=22`; `confident=65`, `needs_review=2`.

## 5. Ba tình huống mơ hồ

Các tình huống dưới đây được điền từ annotation hiện có. Tệp kết quả không chứng minh được chúng đã được ghi trước khi xem bộ tham chiếu; người nộp cần xác nhận lại trình tự làm bài.

### Tình huống A — xe buýt hay xe van?

- **Ảnh và mã vật thể:** `drive_008`, hộp số 7 theo thứ tự trong XML, `xyxy=(364.00, 110.84, 607.40, 334.50)`.
- **Dấu hiệu nhìn thấy:** thân xe rất dài, nhiều cửa sổ hành khách và cấu trúc xe buýt rõ; kích thước lớn hơn xe van thông thường.
- **Quy tắc áp dụng:** thân xe khách dài và nhiều cửa sổ được gán `bus`; kích thước lớn tự nó chưa đủ, phải kết hợp cấu trúc thân xe.
- **Quyết định:** `bus`, `visibility=clear`, `boundary=inside`, `review_state=needs_review`.
- **Nếu vẫn thiếu bằng chứng:** giữ `needs_review`, chụp vùng ảnh ở mức phóng 100% và hỏi Lab Coach xác nhận ranh giới `bus`–`van`; không đổi lớp chỉ để khớp bộ tham chiếu.

### Tình huống B — xe tải hay xe van/ô tô con?

- **Ảnh và mã vật thể:** `drive_038`, hộp số 1 theo thứ tự trong XML, `xyxy=(288.00, 341.60, 465.13, 498.79)`.
- **Dấu hiệu nhìn thấy:** phương tiện công vụ có phần sàn/thiết bị phía sau tách khỏi cabin, không có thân kín một khối như van và không có dáng xe con.
- **Quy tắc áp dụng:** phương tiện có sàn hàng hoặc thiết bị công vụ rõ được gán `truck`.
- **Quyết định:** `truck`, `visibility=clear`, `boundary=inside`, `review_state=needs_review`.
- **Nếu vẫn thiếu bằng chứng:** nhờ Lab Coach xác nhận đây là xe tải chuyên dụng; cung cấp ảnh phóng to và tọa độ hộp thay vì đoán theo màu hoặc kích thước.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- **Ảnh và mã vật thể:** `drive_008`, hộp số 9 theo thứ tự trong XML, `xyxy=(616.70, 207.15, 640.00, 278.61)`.
- **Dấu hiệu nhìn thấy khi phóng 100%:** xe nằm sát cạnh phải; phần thân tiếp tục ra ngoài ảnh, nhưng phần nhìn thấy vẫn đủ để nhận diện là ô tô con.
- **Giá trị `visibility`:** `clear`.
- **Giá trị `boundary`:** `truncated` vì `xbr=640`, đúng mép phải của ảnh 640 px.
- **Trạng thái `review_state`:** `confident`.
- **Lý do:** mất phần vật thể do mép ảnh chứ không phải do một vật khác che; bằng chứng phân lớp còn đủ rõ.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh: `drive_008`, `drive_022`, `drive_033`, `drive_038`.
- [x] Đã kiểm vật thể thiếu và trùng bằng audit và ảnh phủ đối chiếu.
- [x] Đã kiểm lớp và hình học từng hộp; hai gói xuất khớp 67 hộp với IoU nhỏ nhất `0.9999455843`.
- [x] Mỗi hộp có đủ ba thuộc tính trong bản CVAT gốc.
- [ ] Đã xử lý mọi hộp `needs_review` — hiện còn 2 hộp cần xác nhận ở `drive_008` và `drive_038`.
- [ ] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu — không có bằng chứng thời điểm trong tệp kết quả; người nộp cần xác nhận.
- [x] Bài YOLO và CVAT của cá nhân đã được kiểm về tính nhất quán định dạng.
- [ ] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu — cần bổ sung mã lần phát và thời điểm nhận để chứng minh.
- [x] Số vật thể thực tế: **67**. Mục tiêu 40–60 là mục tiêu khối lượng, không phải điểm cắt.

## 7. Bằng chứng dùng để hoàn thành phiếu

- `input_pool_audit.json`: bốn ảnh đầu vào không thay đổi.
- `my_export_audit.json`: 4 ảnh, 67 hộp, lớp và số lượng theo từng ảnh.
- `my_native_export_audit.json`: đủ thuộc tính và nhất quán giữa CVAT–YOLO.
- `comparison_summary.json`, `comparison_iou.csv`, `comparison_overlay.png`: bằng chứng đối chiếu sau bài làm độc lập.

