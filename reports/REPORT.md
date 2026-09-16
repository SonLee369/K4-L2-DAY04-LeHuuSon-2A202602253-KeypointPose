# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Lê Hữu Sơn Ngày: 16/09/2026

## 1. Nhãn của tôi

| Chỉ số                       |       Giá trị |
| ---------------------------- | ------------: |
| Số ảnh đã gán                |            20 |
| Số skeleton                  |            27 |
| v=2 / v=1 / v=0              | 333 / 97 / 29 |
| Thời gian trung bình mỗi ảnh |      ~ 4 phút |

Ba khớp có `%v=1` cao nhất (chép từ `reports/visibility_report.md`):

1. left_hip
2. right_hip
3. left_knee

Chúng có đúng là những khớp bạn thấy khó gán nhất không? Nếu không, giải thích.

Đúng. Các khớp vùng hông (left_hip, right_hip) thường không thể nhìn thấy trực tiếp trên bất kỳ người mặc quần áo nào mà phải dựa vào ước lượng giải phẫu. Sự che khuất bởi trang phục khiến vị trí chính xác của chúng có sự phân vân cao, dẫn đến tỷ lệ cờ `v=1` (bị che) rất lớn.

## 2. Chấm với gold

| Chỉ số                | Trước rework | Sau rework |
| --------------------- | -----------: | ---------: |
| OKS trung bình        |        0.885 |      0.962 |
| OKS@0.50              |        0.920 |      1.000 |
| OKS@0.75              |        0.850 |      0.985 |
| Lỗi `dao_trai_phai`   |            2 |          0 |
| Lỗi `nham_nguoi`      |            0 |          0 |
| Lỗi `xoa_khop_bi_che` |            4 |          0 |

**Tôi đã sửa gì giữa hai lần chạy** (ghi cụ thể: ảnh nào, người thứ mấy, khớp nào):

- `train_16`, người thứ 1, khớp `left_shoulder` và `right_shoulder`: Đổi lại hai điểm do lỗi đảo trái/phải (ngược chiều so với hai mắt).
- `train_16`, người thứ 1, khớp `left_hip` và `right_hip`: Đổi lại hai điểm do lỗi đảo trái/phải.
- `train_04`, người thứ 1, 7 khớp (`left_knee`, `right_knee`,...): Sửa từ cờ `v=0` thành `v=1` và vẽ chấm ước lượng do người nằm gọn trong khung hình nhưng khớp bị che bởi đồ vật.
- `train_10`, người thứ 1, 4 khớp: Sửa từ `v=0` thành `v=1` vì lỗi xoá khớp bị che khi người vẫn nằm gọn trong ảnh.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?** Ảnh đó dễ hay khó? Nếu là ảnh dễ,
bạn nghĩ vì sao mình vẫn sai?

Lỗi đảo trái/phải xảy ra ở `train_16`. Đây là một ảnh dễ và đối tượng hiển thị tương đối rõ. Nguyên nhân sai là do chủ quan làm nhanh và xác định trái/phải theo góc nhìn của bức ảnh thay vì theo cơ thể giải phẫu của người trong ảnh (quên luật tưởng tượng mình đứng vào chỗ người đó).

## 3. Kiểm chéo

Bạn cùng nhóm: File gold

Khớp lệch `%v=1` nhiều nhất giữa hai bảng đếm:

| Khớp       | Bạn |  Họ | Lệch | Nguyên nhân (guideline hay gán sai?)                                            |
| ---------- | --: | --: | ---: | ------------------------------------------------------------------------------- |
| left_hip   | 75% | 40% |  35% | Guideline chưa thống nhất điểm đặt hông trên người mặc áo khoác dài.            |
| right_knee | 60% | 30% |  30% | Guideline chưa rõ về việc khi nào dùng `v=0` hay `v=1` khi khớp mấp mé mép ảnh. |

Luật mới đã bổ sung vào `GUIDELINE_MINI.md` sau khi thống nhất:

- Hông (hip): Đặt chấm ước lượng tại vị trí thắt lưng giải phẫu, luôn gán `v=1` nếu mặc áo form rộng che khuất đường viền cơ thể.
- Mép ảnh: Khớp chỉ được đánh `v=0` (không đặt chấm) nếu ước lượng vị trí tuyệt đối của nó đã nằm TRÀN ra ngoài mép khung hình.

## 4. Model

| Chỉ số         | yolo26n-pose gốc | Sau fine-tune |   Chênh |
| -------------- | ---------------: | ------------: | ------: |
| pose_mAP50     |           0.8450 |        0.8450 | +0.0000 |
| pose_mAP50-95  |           0.6853 |        0.6908 | +0.0055 |
| pose_precision |           0.9734 |        0.9792 | +0.0058 |
| pose_recall    |           0.8462 |        0.8462 | +0.0000 |
| box_mAP50-95   |           0.8119 |        0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

1. `pose_mAP50-95` thay đổi bao nhiêu? Nếu nó giảm, 20 ảnh của bạn dạy được model
   điều gì mà COCO chưa dạy, và nó làm hỏng điều gì?
   Chỉ số `pose_mAP50-95` tăng nhẹ `+0.0055`. Bộ 20 ảnh đã dạy thêm cho model một chút về độ chính xác cục bộ (precision tăng `+0.0058`) của vị trí khớp. Tuy nhiên, do dữ liệu quá nhỏ (20 ảnh), nó bắt đầu gây nhiễu cho khả năng nhận diện người tổng quát, khiến `box_mAP50-95` giảm `-0.0078`.

2. `box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm _người_ dễ hơn hay tìm
   _khớp_ dễ hơn? Vì sao?
   Model tìm người dễ hơn tìm khớp rất nhiều, minh chứng là `box_mAP50-95` (`0.8041`) cao hơn hẳn `pose_mAP50-95` (`0.6908`) sau fine-tune. Việc tìm người (bounding box) chỉ yêu cầu mô hình khoanh đúng vùng không gian chứa cơ thể, trong khi tìm khớp (pose) yêu cầu mô hình phải hiểu đúng cấu trúc giải phẫu và nội suy toạ độ chính xác của 17 điểm nhỏ dù bị che khuất.

3. Một ảnh test model đoán sai - gọi tên lỗi theo bốn loại của slide 43
   (lệch nhẹ / đảo trái/phải / nhầm người / trượt hẳn):
   Ở ảnh `test_09.jpg` (có 2 người với inference nhanh 11.2ms), model dự đoán gặp lỗi "nhầm người" khi các khớp chân của người phía trước bị kéo nhầm sang cơ thể của người đứng ngay sát phía sau do occlusion che khuất lẫn nhau.

4. Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ai đúng, và bạn dựa vào đâu?
   Ảnh `train_06` có OKS thấp nhất (`0.668`). Dựa trên nguyên tắc ưu tiên trí tuệ con người với các trường hợp bị che khuất, nhãn của con người (tôi) đúng hơn. Model thường trượt hẳn khi điểm giải phẫu bị ẩn hoàn toàn sau vật cản lớn, trong khi người gán có thể nội suy chính xác tỷ lệ cơ thể theo `GUIDE.md`.

5. Ảnh bạn gán tệ nhất có _cũng_ là ảnh model đoán tệ nhất không? Nếu có, điều đó
   nói gì về bức ảnh đó?
   Bức ảnh có sự bất đồng lớn nhất là `train_03` (model đoán có 4 người, nhưng tôi chỉ gán 2 người). Điều này cho thấy bức ảnh này chứa nhiều người ở hậu cảnh (background) có kích thước quá nhỏ. Model cố gắng bám theo và khoanh vùng tất cả, trong khi tôi đã tuân thủ chặt chẽ rule bỏ qua những người quá nhỏ không đủ tiêu chuẩn gán.

## 5. Một rule evidence bạn đã dùng

Ở ảnh `train_04`, đối với người thứ 1, tôi phải phân định trạng thái của các khớp như `left_wrist` và `right_wrist`. Dù người này nằm gọn giữa bức ảnh, phần bàn tay đã bị khuất đằng sau vật thể họ đang ôm phía trước mặt. Căn cứ theo vị trí khuỷu tay và góc vát của cánh tay, tôi vẫn chấm điểm tay ở phía sau vật thể và set `v=1` thay vì `v=0`. Rule áp dụng là: Khớp không ra khỏi mép ảnh hình chữ nhật thì bắt buộc phải là `v=1` (bị che), không được xoá (v=0) để đảm bảo bộ xương luôn bảo toàn tính nguyên vẹn không gian.
