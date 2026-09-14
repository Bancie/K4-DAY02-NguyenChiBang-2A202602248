# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** NGUYỄN CHÍ BẰNG<br>
**MSSV:** 2A202602248<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_022`, xe vàng–trắng lớn phía trước (hộp YOLO dòng 1, lớp `bus`; khoảng tâm 0.384, 0.724)
- Dấu hiệu nhìn thấy: thân khách dài, khớp nối hai khoang, nhiều cửa sổ dọc thân, mặt đầu xe buýt rõ
- Quy tắc áp dụng: thân xe khách dài, nhiều cửa sổ hoặc hàng ghế → `bus`; không gán `van` cho thân xe buýt
- Quyết định: `bus`
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng 100%, so với van hộp nhỏ trên cùng ảnh; nếu vẫn không chắc thì `review_state=needs_review` và hỏi Lab Coach, không đoán theo kích thước hộp

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038`, xe cứu hộ trắng chữ 公安 (hộp YOLO dòng 1, lớp `truck`; khoảng tâm 0.597, 0.659)
- Dấu hiệu nhìn thấy: cabin tách sàn, cần cẩu / thiết bị kéo phía sau, không phải thân hộp kín một khối
- Quy tắc áp dụng: thùng, ben, sàn hàng hoặc thiết bị công vụ rõ → `truck`; van kín một khối không vào lớp này
- Quyết định: `truck`
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? So với van đỏ thân hộp gần xe tải phủ bạt trên cùng ảnh (`van`); nếu thiết bị công vụ không rõ thì `needs_review`, không gán `car` chỉ vì cabin giống ô tô con

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008`, mảnh xe sát mép phải–dưới (hộp `car`, xtl≈620, ytl≈566, xbr=640, ybr=640)
- Dấu hiệu nhìn thấy khi phóng 100%: chỉ còn một góc thân tối, không thấy cabin, bánh hay kiểu dáng đủ để chắc lớp
- Giá trị `visibility`: `unclear`
- Giá trị `boundary`: `truncated`
- Trạng thái `review_state`: `needs_review`
- Lý do: mép ảnh cắt gần hết vật thể; bằng chứng phân lớp không đủ. Giữ hộp và đánh `needs_review` chứ không đoán `car`/`truck`. Cụm xe < khoảng 15 px trên `drive_033` xử lý cùng cách.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [ ] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 105 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.

Đã xem 19 hộp `needs_review` còn lại (`drive_008` 4, `drive_033` 9, `drive_038` 6): giữ cờ và không đoán lớp. Không gán người, xe máy, xe đạp.
