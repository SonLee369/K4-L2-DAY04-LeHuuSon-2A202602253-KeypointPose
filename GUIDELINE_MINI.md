# Mini guideline - nhóm: Lê Hữu Sơn | người gán: Lê Hữu Sơn | ngày: 16/09/2026

> Điền file này **trong lúc** gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> hơn 10 giây để phân vân, đó là một dòng phải ghi vào đây.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung -> `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh -> `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật của nhóm bạn (phải điền)

| Tình huống                                        | Luật nhóm bạn chọn                                                                                                                                          | Vì sao                                                                                                                                      |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Hông của người mặc quần áo dài                    | Đặt chấm ước lượng tại vị trí thắt lưng giải phẫu (dựa theo tỷ lệ vai-đùi), gán `v=1`.<br><img src="images/case1.jpg" width="150">                          | Bề mặt hông không nhìn thấy trực tiếp nhưng vẫn nằm trong khung hình, phải giữ đúng topology.                                               |
| Tai bị tóc hoặc mũ bảo hiểm che một phần          | Gán `v=1` và chấm điểm ngay dưới vành mũ/lớp tóc theo cấu trúc đầu người.<br><img src="images/case2-3.jpg" width="150">                                     | Điểm tai không lộ diện nhưng không vượt ra ngoài mép ảnh.                                                                                   |
| Người bị cắt ở mép ảnh (chỉ thấy từ hông trở lên) | Các khớp từ hông trở xuống gán `v=0` và không đặt chấm.<br><img src="images/case2-3.jpg" width="150">                                                       | Các điểm này thực sự nằm ngoài mép ảnh, nếu cố gắng nội suy sẽ tạo ra toạ độ vượt ra ngoài khung, vi phạm luật `v=0`.                       |
| Cổ tay nằm sau tay lái / sau thân mình            | Gán `v=1`, nội suy vị trí cổ tay kéo dài từ khuỷu tay.<br><img src="images/case4.jpg" width="150">                                                          | Khớp vẫn ở trong ảnh nhưng bị vật cản (tay lái) che khuất bề mặt.                                                                           |
| Hai người chồng lên nhau                          | Khớp người bị che gán `v=1`, khớp người che gán `v=2`. Chú ý click từng điểm thay vì di chuyển cả bounding box.<br><img src="images/case5.jpg" width="150"> | Giúp model phân biệt được depth (độ sâu) giữa người đứng trước và người đứng sau, tránh tình trạng gán nhầm điểm xương sang người bên cạnh. |
| Người nhỏ đến mức nào thì không gán nữa           | Bỏ qua người nếu diện tích bounding box < 1/10 chiều cao ảnh HOẶC zoom 200% không thể nhận diện đủ các chi.<br><img src="images/case6.jpg" width="150">     | Nhãn quá mờ/nhỏ sẽ tạo ra "nhiễu", model không học được cấu trúc giải phẫu chuẩn và gây mAP thấp (như ca sai lệch ở train_03).              |

Với mỗi luật, chèn **một ảnh mẫu** (screenshot từ CVAT) thay vì chỉ viết một câu.
Slide 12 nói rõ: khớp không có bề mặt nhìn thấy được thì phải có ảnh mẫu, không phải
một câu văn chung chung[cite: 4].

## 3. Ba ca mơ hồ đã gặp (bắt buộc, ghi ít nhất 3)

### Ca 1 - ảnh `train_04`, người thứ `1`, khớp `left_wrist` và `right_wrist`

- Mơ hồ ở chỗ nào: Cổ tay bị khuất hoàn toàn sau vật thể người này đang ôm phía trước, nhưng cơ thể vẫn nằm trọn ở giữa khung ảnh.
- Bạn quyết thế nào: Vẫn đặt chấm ước lượng và đánh cờ `v=1` (bị che).
- Vì sao: Khớp không thể vượt ra khỏi khung hình được nên không thể là `v=0`.
- Nếu người khác quyết ngược lại thì model học sai cái gì: Nếu đánh `v=0` (xoá khớp bị che), model sẽ học cách vứt bỏ cổ tay mỗi khi tay cầm nắm vật thể lớn, phá vỡ cấu trúc không gian của bộ xương.

### Ca 2 - ảnh `train_16`, người thứ `1`, khớp `left_shoulder` và `right_shoulder` (và vùng hông)

- Mơ hồ ở chỗ nào: Việc xác định phương hướng trái/phải dễ bị nhầm lẫn khi người trong ảnh đứng ở các góc độ hoặc tư thế vặn mình đánh lừa thị giác.
- Bạn quyết thế nào: Xác định trái/phải dựa hoàn toàn theo cơ thể giải phẫu của nhân vật, tưởng tượng mình đứng vào vị trí của họ.
- Vì sao: Để tránh hiện tượng hai xương cắt chéo nhau ở thân (đảo trái/phải).
- Nếu người khác quyết ngược lại thì model học sai cái gì: Nếu gán theo góc nhìn bức ảnh, model sẽ bị dạy cái sai này tới hai lần do cơ chế augmentation lật ảnh ngang (`fliplr=0.5`).

### Ca 3 - ảnh `train_03`, người thứ `3, 4` (các nhân vật phụ ở xa), khớp `toàn bộ`

- Mơ hồ ở chỗ nào: Bức ảnh có những người ở hậu cảnh có kích thước rất nhỏ. Model có xu hướng nhận diện họ (đoán ra 4 người), nhưng có nên gán nhãn cho họ không?
- Bạn quyết thế nào: Bỏ qua, chỉ gán nhãn cho 2 người chính yếu.
- Vì sao: Kích thước quá nhỏ không thể ước lượng chính xác cấu trúc giải phẫu. Mọi người được gán phải đủ lớn để xác định khớp.
- Nếu người khác quyết ngược lại thì model học sai cái gì: Nếu cố gán, model sẽ bị ép học các đặc trưng nhiễu từ các khối pixel mờ nhạt, làm giảm precision (`pose_precision`) tổng thể trên các đối tượng rõ ràng.

## 4. Sau khi so visibility report với bạn cùng nhóm[cite: 4]

- Khớp lệch `%v=1` nhiều nhất: `left_hip` (bạn `75%` / họ `40%`)
- Nguyên nhân là **guideline chưa rõ** hay **một trong hai bên gán sai**: Nguyên nhân do guideline chưa rõ về cách xử lý các điểm giải phẫu ước lượng đối với trang phục rộng/dài.
- Luật mới bổ sung vào mục 2 sau khi thống nhất: Khớp vùng hông (hip) bắt buộc phải set cờ `v=1` (bị che) nếu người đó mặc quần áo form rộng hoặc áo khoác dài che khuất đường viền thân thể thật; không được tự tiện đánh `v=2` (nhìn thấy rõ) dựa trên cảm quan.
