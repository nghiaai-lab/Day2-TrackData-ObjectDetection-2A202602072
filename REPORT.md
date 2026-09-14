# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** CẦN BỔ SUNG  
**MSSV:** CẦN BỔ SUNG  
**Hình thức:** Cá nhân — cần người nộp xác nhận  
**Mã cặp:** `SOLO`

## 1. Bài độc lập và nguồn dữ liệu

- **Mã SHA-256 của ZIP ảnh được cấp:** `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`.
- **Bốn mã ảnh:** `drive_008`, `drive_022`, `drive_033`, `drive_038`.
- **Số vật thể thực tế:** 67.
- **Mã SHA-256 của gói YOLO của tôi:** `395fa9aef2bd0478d86513af312500395c2222778e9f53febaa3bdfebbf15524`.
- **Mã SHA-256 của gói CVAT gốc của tôi:** `bb41152e201560cfc6a8a2637ce0a1086c5adb71137df3dd60e1040a367148c5`.
- **Nguồn đối chiếu:** bộ tham chiếu giảng dạy do người hướng dẫn thực hành cấp (`teaching_reference`).
- **Mã SHA-256 của gói đối chiếu:** `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`.
- **Mã lần phát và thời điểm nhận bộ tham chiếu:** CẦN BỔ SUNG; dữ liệu hiện có không lưu hai trường này.
- **Nguồn ảnh:** kho nguồn của giảng viên tại Git commit `710d2c157c5750f71271354d0d31b24457c64e7c`; kho mô tả ảnh bắt nguồn từ UA-DETRAC qua bản tái xuất SQiFeng/traffic-vehicle-detection và công bố CC BY 4.0. Kho đã đổi tên ảnh, không giữ mã bản ghi gốc và liên kết bản tái xuất không còn truy cập được; đây là giới hạn truy nguyên, không phải xác minh pháp lý độc lập.

`input_pool_audit.json` xác nhận bốn ảnh đầu vào không thay đổi. Bài YOLO và CVAT có mã hash riêng, cùng chứa 4 ảnh và 67 annotation. Audit chéo ghép đủ 67/67 hộp, với IoU nhỏ nhất `0.9999455843`, cho thấy hai định dạng được xuất từ cùng trạng thái annotation. Tuy nhiên, các tệp kỹ thuật không chứng minh được thời điểm bộ tham chiếu được phát; vì vậy cần bổ sung mã lần phát và thời gian nhận trước khi khẳng định đầy đủ tính độc lập theo quy trình.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_008`, hộp 7, `xyxy=(364.00,110.84,607.40,334.50)` | `bus` | Thân xe dài, nhiều cửa sổ hành khách, cấu trúc xe buýt rõ | Xe khách thân dài và nhiều cửa sổ thuộc `bus`; còn phân vân thì giữ `needs_review` |
| `drive_038`, hộp 1, `xyxy=(288.00,341.60,465.13,498.79)` | `truck` | Có sàn/thiết bị công vụ phía sau tách khỏi cabin | Xe có thùng, sàn hàng hoặc thiết bị công vụ rõ thuộc `truck` |
| `drive_008`, hộp 1, `xyxy=(393.41,427.56,505.07,561.30)` | `car` | Dáng sedan/hatchback, khoang hành khách kiểu ô tô con | Ô tô con không có thùng hàng, thân bus hoặc thân van được gán `car` |
| `drive_008`, hộp 2, `xyxy=(284.57,251.21,347.10,354.40)` | `van` | Thân hộp nhỏ và kín, cao hơn ô tô con nhưng không dài như bus | Xe thân hộp nhỏ chở người/hàng được gán `van` |

Ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau: hộp số 7 của `drive_008` có **lớp** `bus`, mô tả loại phương tiện; đồng thời có **thuộc tính** `visibility=clear`, `boundary=inside`, `review_state=needs_review`, mô tả điều kiện quan sát và mức chắc chắn. Thay đổi `review_state` không tự động đổi lớp của hộp.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Schema YOLO khai báo thứ tự `truck, bus, car, van`, khác thứ tự chuẩn | lớp/schema | `audit_yolo_export` báo schema không khớp `car, truck, bus, van` | Đổi schema thành `0 car, 1 truck, 2 bus, 3 van` và remap đồng thời mọi class ID; không chỉ sửa `data.yaml` |
| `train.txt` dùng đường dẫn `data/images/train/...` trong khi ảnh nằm ở `images/train/...` | phạm vi/đóng gói | So đường dẫn trong danh sách huấn luyện với tên entry thực tế trong ZIP | Bỏ tiền tố `data/`; kiểm lại để mọi đường dẫn trong `train.txt` đều tồn tại |
| Hai gói YOLO và CVAT có nguy cơ lệch do khác cách biểu diễn tọa độ | hình học/định dạng | Audit chéo class và IoU giữa 67 cặp hộp | Xác nhận cùng trạng thái annotation; IoU nhỏ nhất `0.9999455843`, cao hơn sàn kiểm tra `0.995` |

- **Số hộp `needs_review` trước và sau khi kiểm:** không có số liệu “trước” được lưu; sau audit còn **2** hộp: `drive_008` hộp bus và `drive_038` hộp truck.
- **Một quyết định chưa đủ bằng chứng và cách xin hỗ trợ:** phương tiện công vụ ở `drive_038`, `xyxy=(288.00,341.60,465.13,498.79)`, đang gán `truck` nhưng giữ `needs_review`. Tôi sẽ gửi ảnh phóng 100%, tọa độ hộp và quy tắc “có sàn/thiết bị công vụ rõ” cho Lab Coach để xác nhận, thay vì đổi nhãn theo bộ tham chiếu một cách máy móc.

## 4. Một dòng nhãn YOLO

- **Dòng `class x_center y_center width height`:** `0 0.701937 0.772547 0.174469 0.208969` trong nhãn của `drive_008`.
- **Tên lớp và tọa độ điểm ảnh `xyxy`:** class `0` là `car`; với ảnh 640×640, tọa độ xấp xỉ `(393.41, 427.56, 505.07, 561.30)` px.
- **Kiểm tra quy đổi:** `x_center≈449.24`, `y_center≈494.43`, `width≈111.66`, `height≈133.74` px; từ đó `x1=x_center-width/2`, `y1=y_center-height/2`, `x2=x_center+width/2`, `y2=y_center+height/2`.

Một dòng đúng năm trường và có tọa độ trong `[0,1]` vẫn có thể sai: class ID có thể trỏ tới sai lớp do thứ tự schema, hộp có thể ôm cả nền hoặc nhiều xe, có thể bỏ sót/ghi trùng vật thể, hoặc annotation có thể nằm ngoài phạm vi bốn lớp. Kiểm tra cú pháp không thay thế được kiểm tra ngữ nghĩa và hình học trên ảnh.

## 5. Huấn luyện và dự đoán thử

- **Ba mã ảnh huấn luyện:** `drive_022`, `drive_033`, `drive_038`.
- **Mã ảnh thẩm định:** `drive_008`.
- **Thiết lập:** Ultralytics `8.4.145`, mô hình `yolo11n.pt`, 8 epochs, seed 42, CPU, thời gian 26,45 giây. Lần chạy chỉ dùng để phản hồi và tìm lỗi dữ liệu, không phải benchmark sản xuất.
- **Mô tả dự đoán trong `detect_result.jpg`:** ảnh kết quả không hiển thị hộp dự đoán nào trên `drive_008`, dù ảnh có nhiều phương tiện rõ ràng.
- **Dự đoán gợi ý cần kiểm lại:** kiểm tra lại schema class ID, đường dẫn train/validation, việc nạp nhãn, ngưỡng confidence khi lưu ảnh và sự mất cân bằng lớp (`car` chiếm 52/67 hộp). Đồng thời cần xem log huấn luyện để phân biệt mô hình chưa học được với trường hợp dự đoán bị ngưỡng lọc bỏ.
- **Minh chứng có thể bác bỏ nhận định “mô hình chưa học được”:** kết quả suy luận lưu cả confidence cho thấy có nhiều box hợp lý ngay dưới ngưỡng hiển thị, hoặc đánh giá trên một tập ảnh độc lập lớn hơn cho recall tốt và ổn định qua nhiều seed.

![Kết quả dự đoán trên drive_008](detect_result.jpg)

Kết quả trên bốn ảnh không phải đánh giá mô hình dùng thực tế vì tập dữ liệu quá nhỏ, chỉ có ba ảnh huấn luyện và một ảnh thẩm định, các ảnh cùng nguồn/cảnh giao thông, không có test set độc lập, phân bố lớp rất lệch và huấn luyện chỉ 8 epochs. Không thể từ lần chạy này suy ra khả năng tổng quát hóa, độ bền trước điều kiện mới hoặc các chỉ số an toàn vận hành.

## 6. Đối chiếu nhãn

- **Cách ghép:** ghép tối ưu theo IoU hình học, không dùng lớp khi ghép.
- **Số hộp ghép được:** 44.
- **IoU trung bình và trung vị:** `0.831294` và `0.851503`.
- **Mức đồng thuận lớp:** `0.704545`, tức **31/44 = 70,45%** cặp ghép.
- **Số hộp phía tôi không ghép được:** 23.
- **Số hộp phía đối chiếu không ghép được:** 6.
- **Một điểm khác biệt cụ thể:** tại `drive_022`, hộp 1 của tôi và hộp 2 của bộ tham chiếu gần như cùng hình học (`IoU=0.975136`) nhưng tôi gán `bus`, còn bộ tham chiếu gán `van`. Đây chủ yếu là bất đồng phân lớp, không phải vị trí hộp.
- **Quy tắc hoặc hành động sửa phát sinh:** làm rõ ranh giới `bus`–`van` dựa trên chiều dài thân, số/cấu trúc cửa sổ và bố cục xe khách; xem lại các cặp `truck`–`bus` và `car`–`truck` có IoU cao nhưng khác lớp. Không tự động coi bộ tham chiếu là đúng; mọi thay đổi phải quay lại ảnh và quy tắc nhìn thấy được.

![Ảnh phủ đối chiếu hình học](comparison_overlay.png)

Mức đồng thuận cao không chứng minh mọi nhãn đều đúng vì hai bên có thể cùng áp dụng sai một quy tắc, cùng bỏ sót vật thể hoặc đặt hộp giống nhau quanh sai đối tượng. Chỉ số tổng hợp cũng che khuất sai khác theo lớp và vật thể nhỏ. Trong kết quả này vẫn có 23 hộp phía tôi và 6 hộp phía đối chiếu không ghép được, cùng 13/44 cặp ghép bất đồng lớp; do đó phải kiểm tra từng khác biệt quan trọng trên ảnh.

Ngưỡng IoU `0.01` chỉ là sàn dùng trong thuật toán ghép, **không phải ngưỡng đạt chính thức**. Bộ tham chiếu phục vụ phản hồi sau bài làm độc lập và không phải kết luận về chất lượng sản xuất.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [ ] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình — cần kiểm lại kho trước khi commit; các tệp này hiện tồn tại ngoài báo cáo.
- [ ] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập — cần chạy kiểm tra cuối trên nội dung staging trước khi đẩy lên GitHub.

**Minh chứng mạnh nhất:** audit chéo giữa gói YOLO và CVAT ghép đủ **67/67 hộp**, xác nhận cùng trạng thái annotation với IoU nhỏ nhất `0.9999455843`, đồng thời đủ ba thuộc tính cho mọi hộp trong bản CVAT.

**Câu hỏi còn lại cho Lab Coach:** hai hộp đang `needs_review` ở `drive_008` và `drive_038` nên giữ lớp `bus`/`truck` hay cần đổi theo quy tắc phân lớp nào; và với các bất đồng có IoU cao như `drive_022` (`bus` so với `van`), dấu hiệu nhìn thấy nào được ưu tiên để chốt lớp?

## 8. Tệp bằng chứng

- [Thông tin lần huấn luyện](training_run.json)
- [Audit gói YOLO](my_export_audit.json)
- [Audit gói CVAT](my_native_export_audit.json)
- [Audit ảnh đầu vào](input_pool_audit.json)
- [Tóm tắt đối chiếu](comparison_summary.json)
- [Bảng IoU theo cặp](comparison_iou.csv)
- [Nguồn gốc ảnh](IMAGE_ATTRIBUTION.md)
- [Ảnh dự đoán](detect_result.jpg)
- [Ảnh phủ đối chiếu](comparison_overlay.png)

