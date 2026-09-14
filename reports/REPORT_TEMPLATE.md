# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** NGUYỄN CHÍ BẰNG<br>
**MSSV:** 2A202602248<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_008` (`fde088a7a955343bb3366008d31e3c2a6ecf6ddbfd08e58ecd91f5e6d67a05d1`), `drive_022` (`cb8297af4cc5bb660f9d56f0e31ed77f8fa1f95b31f47bb8b8a1cd08b2f94886`), `drive_033` (`8dc05a7a8f06ed137d08b643ef676629a2c4cf3b25051852de22e7400311e465`), `drive_038` (`35294a107b157646e0619ad985226ef424968d7798f90f7ded5b32f32cebbb86`)
- Số vật thể thực tế: **105** hộp (đúng nhau trên cả hai gói xuất) — `car` 78, `truck` 9, `bus` 12, `van` 6. Phân theo ảnh: `drive_008` 29, `drive_022` 5, `drive_033` 30, `drive_038` 41. Vượt mục tiêu khối lượng 40–60 vì còn gán nhiều xe rất xa / bị cắt.
- Mã SHA-256 của gói YOLO của bạn: `d2a6a9c5554076682ae7310a1418a54111ec907a8d35d47fe0b61a866a027ae2` (`day2-my-expor.zip`, khóa lúc 2026-09-14 11:32 GMT+7)
- Mã SHA-256 của gói CVAT gốc của bạn: `0621d3fd52893e8b72ecd92fdcd06fb00f3dd274313e3bc881d0fd2fc0286612` (`job_2_dataset_2026_09_14_04_31_03_cvat for images 1.1.zip`, dump 2026-09-14 04:39 UTC)
- Nguồn đối chiếu: bộ nhãn tham chiếu do Lab Coach cấp (`day2-teaching-reference.zip`)
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: `day2-reference-4img-v1` (50 hộp); nhận khoảng **2026-09-14 11:45 GMT+7**, sau khi đã khóa hai gói xuất của mình (11:32–11:39)

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Tôi tự tạo tác vụ CVAT, tự gán cả bốn ảnh, tự xuất YOLO và CVAT for images 1.1 từ cùng job #2, rồi ghi SHA hai gói. Bộ tham chiếu chỉ mở sau khi bản xuất đã khóa; không xem nhãn người khác và không sửa TXT/XML bằng tay.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_008` sedan trắng gần camera (≈393,426–501,570) | `car` | thân sedan, không thùng/ben, không thân hộp van | sedan/hatchback/SUV/taxi → `car` |
| `drive_008` xe ben đỏ (≈238,70–345,213) | `truck` | thùng ben lộ rõ, tách khỏi cabin | thùng/ben/sàn hàng → `truck` |
| `drive_008` xe buýt vàng khớp nối | `bus` | thân khách dài, nhiều cửa sổ, khớp nối | thân xe khách dài → `bus` |
| `drive_008` van trắng giữa ảnh (≈285,251–350,359) | `van` | thân hộp kín, nhỏ, không khoang hàng tách như tải | thân hộp kín một khối → `van` |
| `drive_022` xe buýt vàng lớn | `bus` | đầu xe buýt, thân khớp nối, nhiều cửa sổ | không gán van cho thân xe khách |
| `drive_022` xe thùng trắng bên phải | `truck` | cabin tách khoang hàng hộp | khoang hàng tách biệt → `truck` |
| `drive_033` taxi vàng | `car` | thân taxi/ô tô con | taxi thuộc `car` |
| `drive_033` xe thùng trắng giữa đường | `truck` | thùng kín sau cabin | `truck` |
| `drive_038` xe cứu hộ 公安 | `truck` | cần cẩu / thiết bị công vụ trên sàn | thiết bị công vụ rõ → `truck` |
| `drive_038` xe đỏ thân hộp (gần xe tải phủ bạt) | `van` | thân hộp kín một khối, không thùng tách | `van`, không phải `truck` |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Cùng lớp `bus` trên `drive_008`: xe buýt vàng lớn có `visibility=clear`, `boundary=inside`; xe buýt nhỏ sát mép trên có `visibility=unclear`, `boundary=truncated`. Lớp nói loại phương tiện; thuộc tính nói mức bằng chứng và quan hệ với khung hình. YOLO không lưu ba thuộc tính này.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| `drive_033` sedan trắng góc phải (xtl≈562, xbr=640) để `boundary=inside` | thuộc tính | rà `xbr` chạm 640 | đổi thành `truncated`; chạm mép ảnh = `truncated` |
| `drive_022` hai ô tô nền để `boundary=truncated` dù nằm trọn trong khung | thuộc tính | so hộp với mép ảnh | đổi thành `inside` nếu không bị cắt |
| Nhiều xe rất xa trên `drive_033`/`drive_038` vẫn giữ hộp | phạm vi | đếm 105 hộp (vượt 40–60); 19 hộp `needs_review` | không đoán lớp khi phóng 100% vẫn không đủ bằng chứng; bỏ hoặc giữ `needs_review` rồi xin hỗ trợ |
| Xe máy / người / xe đạp hiện rõ trên cả bốn ảnh | phạm vi (đã đúng) | rà vật thể ngoài bốn lớp | không gán; chỉ bốn lớp `car/truck/bus/van` |

- Số hộp `needs_review` trước và sau khi kiểm: không có bản đếm trước khi rà; **sau khi tự kiểm còn 19/105** (`drive_008` 4, `drive_033` 9, `drive_038` 6) — chủ yếu xe quá nhỏ, mờ hoặc bị mép cắt. `drive_022` không còn hộp cần xem lại.
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: cụm xe sát chân trời trên `drive_033` (nhiều hộp < 10×15 px, `visibility=unclear`). Tôi đánh `needs_review` chứ không đoán `car`/`truck`. Sẽ hỏi Lab Coach: vật thể nhỏ hơn ngưỡng nào thì bỏ hẳn khỏi phạm vi.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `0 0.699094 0.778133 0.168625 0.223828` (`drive_008.txt`, dòng 1)
- Tên lớp và tọa độ điểm ảnh `xyxy`: `car`; khoảng `(393.5, 426.4, 501.4, 569.6)` trên ảnh 640×640 — khớp hộp sedan trắng gần camera trong XML
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Đúng năm trường chỉ nghĩa là cú pháp hợp lệ. Lớp có thể nhầm `van`/`bus`/`truck`; hộp có thể gộp hai xe hoặc ôm quá nhiều nền; vật thể ngoài phạm vi (xe máy, phản chiếu) vẫn ghi được thành một dòng. Thuộc tính kiểm tra không nằm trong TXT.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`: chưa chạy ô 4 của sổ thực hành nên chưa có `detect_result.jpg`. Ảnh thẩm định đông xe (`drive_008`, 29 hộp) nên mô hình 8 epoch trên 3 ảnh rất dễ bỏ sót xe nhỏ hoặc nhầm `truck`/`bus`/`van`.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Cần rà lại phạm vi (có nên giữ 105 hộp hay chỉ xe đủ bằng chứng) và ranh giới `truck`/`van`/`bus` — đây là chỗ lớp dễ lệch giữa hai nguồn nhãn.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Ảnh dự đoán thật sau khi chạy ô 4; hoặc đối chiếu hộp mô hình với hộp CVAT trên cùng `drive_008`. Một dự đoán đúng lớp/hộp sát phần nhìn thấy sẽ bác bỏ nhận định “mô hình chắc chắn rối trên ảnh này”.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

Bốn ảnh nhỏ, có thể liên quan thời gian, chia 3/1 chỉ để kiểm đường ống. mAP hay ảnh dự đoán không chấm người gán nhãn và không chứng minh mô hình dùng được ngoài lab (thiếu tập test độc lập, đa điều kiện, đủ khối lượng).

## 6. Đối chiếu nhãn

Ghép tối ưu theo IoU hình học (ngưỡng kỹ thuật 0.01, không phải ngưỡng đạt) với bộ `day2-reference-4img-v1` (50 hộp).

- Số hộp ghép được: **47**
- IoU trung bình và trung vị: **0.867** và **0.886**
- Mức đồng thuận lớp: **0.681** (32/47 hộp ghép cùng mã lớp)
- Số hộp phía bạn không ghép được: **58**
- Số hộp phía đối chiếu không ghép được: **3**
- Một điểm khác biệt cụ thể: **phạm vi**. Tôi gán 105 hộp, đối chiếu 50; 58 hộp của tôi không ghép (chủ yếu xe xa trên `drive_033`/`drive_038` và phần bị cắt trên `drive_008`). Hình học các xe lớn gần như trùng (IoU thường > 0.85). Về lớp, nhiều hộp chồng khít nhưng mã khác nhau theo cặp `bus`↔`van` và `truck`↔`bus` (ví dụ xe buýt vàng `drive_022`: tôi `bus` theo thân khách dài; đối chiếu ghi `van`). Hai khác biệt lớp không giải thích được bằng cặp đó: trên `drive_033`, xe nhỏ sau xe tải thùng và xe cam xa — tôi `truck`, đối chiếu `car`.
- Quy tắc hoặc hành động sửa phát sinh: (1) bỏ hoặc không đưa vào tập huấn luyện các hộp `unclear` quá nhỏ; (2) giữ `bus`/`truck`/`van` theo dấu hiệu nhìn thấy trên phiếu quy tắc; (3) hỏi Lab Coach schema lớp của gói tham chiếu; (4) sửa `boundary` cho hộp chạm mép. Sửa trong CVAT rồi xuất lại, không sửa TXT.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?

IoU chỉ đo hai hộp chồng khít. Hai người có thể cùng vẽ sát cùng một xe nhưng cùng sai lớp, hoặc cùng bỏ sót / cùng gán vật thể không đủ bằng chứng. Đồng thuận đo khả năng tái lập quy tắc, không phải chân lý.

## 7. Kiểm tra kho GitHub cá nhân

- [ ] Có phiếu quy tắc với ba tình huống mơ hồ.
- [ ] Có kết quả kiểm hai gói xuất.
- [ ] Có thông tin lần huấn luyện và ảnh dự đoán.
- [ ] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Hai ô cuối là cam kết khi nộp kho `KX-DAY02-...`: không tải `reports/my-export`, XML CVAT, `day2-teaching-reference.zip` hay `*.pt` lên GitHub. Các ô trên còn trống vì chưa chạy hết sổ thực hành (`day2_lab_outputs/`) và chưa điền `guideline-mini-sheet.md`.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh: hai gói xuất cùng 105 hộp; SHA đã khóa trước khi nhận `day2-reference-4img-v1`; đối chiếu 47 hộp, IoU trung vị 0.886; quyết định lớp có dấu hiệu nhìn thấy (xe ben, xe buýt khớp nối, xe cứu hộ, van đỏ). Câu hỏi: vật thể `unclear` < khoảng 15 px có được bỏ khỏi phạm vi không, và bộ tham chiếu có đúng thứ tự lớp `0 car, 1 truck, 2 bus, 3 van` không?
