# Báo cáo Day 13 — Nguyễn Nhân Thành Long, 2A202603310


## 1. Kết luận trước
- Mã số: `2A202603310`; task: `D13-2A202603310`.
- 10 frame thuộc 5 sequence: `s01f24`, `s01f55`, `s02f02`, `s02f40`, `s03f18`, `s03f41`, `s04f13`, `s04f49`, `s05f05`, `s05f22`.
- Môi trường: Windows 11, Chrome, CVAT 2.8.1.
- Tổng cộng 53 cuboid. Frame có ít cuboid nhất là `s03f41.pcd` với 4 hộp; đây là số hộp đã gán, không khẳng định frame chỉ có bốn vật.
- Phiếu tự kiểm có cảnh báo ở 2/53 cuboid. Cả hai cảnh báo là điểm sát ngoài mép ở hai Pedestrian trong `s01f24` (idx 10 và 11); tôi đã kiểm tra và xác nhận là đúng.
- Tôi tin nhất hướng và vị trí đáy của các xe con: đa số yaw xe cùng chiều gần 270° và đáy gần mặt đường. Phần còn yếu là kích thước xe buýt mẫu và ranh giới hai hộp Pedestrian gần nhau.

## 2. Đọc frame (Bước 2)
Ảnh: `t1-s01f24-3d.png`, `t1-s01f24-top.png`, `t1-s01f55-3d.png`, `t1-s01f55-top.png`.

| Frame | Hướng trong ảnh nhìn từ trên xuống | Mặt đường và vật xa | Vật quan sát được / cụm chưa đọc được |
|---|---|---|---|
| `s01f24.pcd` | Phía trước ở cạnh trên; +x sang trái, +y xuống dưới | Đường tương đối phẳng, thấp hơn LiDAR khoảng 1,6 m; còn nhận ra xe khoảng 40 m | Có xe ở hai bên; cụm xa, rời rạc chưa đủ căn cứ để gọi tên |
| `s01f55.pcd` | Phía trước ở cạnh trên; +x sang trái, +y xuống dưới | Đường tương đối phẳng, thấp hơn LiDAR khoảng 1,6 m; còn nhận ra xe khoảng 40 m | Có xe ở hai bên; cụm xa, rời rạc chưa đủ căn cứ để gọi tên |

Camera cho biết màu sắc, trạng thái đèn và chữ/biển hiệu trong ảnh. LiDAR cho biết khoảng cách, hình dạng/phần bề mặt có điểm phản hồi và độ cao tương đối của vật so với mặt đường.

| Frame lướt | Phía trước trong ảnh nhìn từ trên xuống | Điều lạ |
|---|---|---|
| `s02f02.pcd` | Cạnh trên | Không ghi nhận |
| `s02f40.pcd` | Cạnh trên phải | Không ghi nhận |
| `s03f18.pcd` | Cạnh dưới trái | Không ghi nhận |
| `s03f41.pcd` | Cạnh trên | Không ghi nhận |
| `s04f13.pcd` | Cạnh trên phải | Không ghi nhận |
| `s04f49.pcd` | Cạnh phải | Không ghi nhận |
| `s05f05.pcd` | Cạnh phải | Không ghi nhận |
| `s05f22.pcd` | Cạnh trái | Không ghi nhận |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f24`, 1 | Car | Có; 270,0° | Có; z_bottom = −1,717 m | Có; ảnh `t2-a-car.png` | Có; 4,394 × 1,772 × 1,458 m nằm trong khoảng xe con |
| (b) xe buýt | `s01f24`, 5 | Bus | Có; 269,9° | Có; z_bottom = −1,592 m | Có; ảnh `t2-b-bus-truck.png` | Không; 9,223 × 2,943 × 2,556 m đều ngoài khoảng xe buýt thường ở bảng 4.3; cần xem lại kích thước/loại xe |
| (c) người đi bộ | `s01f24`, 10 | Pedestrian | Chưa thể kết luận chắc từ bảng A; yaw = 240,9° | Có; z_bottom = −1,589 m | Có; ảnh `t2-c-ped-cyclist.png` | 0,368 × 0,608 × 1,659 m; chiều dài thấp hơn mốc người lớn 0,4 m khoảng 8%, trong sai số ±10% |

Vật bỏ qua: không có.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân dưới đây là giả thuyết dựa trên point cloud; ảnh camera bổ sung cho việc nhận diện vật khi có thể.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f24.pcd` / `t4-1-bong-che.png` / task cá nhân | Xe hoặc vật lớn chắn tia LiDAR | Có vùng thưa/khuyết điểm phía sau vật, trong khi các vùng lân cận vẫn có điểm | Vùng khuất không đồng nghĩa với đường trống. Nếu nhận diện được vật, đánh dấu `occluded` và đặt cuboid bao phần có thể suy luận hợp lý. |
| 2. Thưa ở xa | `s01f24.pcd` / `t4-2-thua-o-xa.png` / task cá nhân | Mật độ điểm giảm theo khoảng cách; vật xa chỉ phản hồi một phần | Cụm điểm ở xa rời rạc, ít điểm hơn các vật gần | Chỉ vẽ khi thấy khoảng 15–20% thể tích hoặc có đủ căn cứ suy luận; nếu không, bỏ qua và ghi `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f55.pcd` / `t4-3-phan-xa-thap.png` / task cá nhân | Bề mặt tối, kính hoặc góc tới tia có thể làm phản hồi yếu; chưa thể xác định nguyên nhân chỉ từ point cloud | Một số cụm có điểm phản hồi thưa; ảnh lưu không đủ để kết luận màu/chất liệu hay intensity | Xem camera để đối chiếu; giữ đây là giả thuyết, không nới cuboid để bao vùng không có điểm và không kết luận vật thể chỉ từ độ thưa. |
| 4. Điểm lạ | `s04f49.pcd` / `t4-4-diem-la.png` / task cá nhân | Có thể là nhiễu đo, phản xạ ngoài ý muốn hoặc điểm thuộc vật chưa nhận diện; nguồn gốc chưa rõ | Có các điểm rời rạc không tạo thành hình dạng vật rõ ràng trong ảnh chụp | Không gán điểm/cụm điểm này thành một vật nếu không đủ căn cứ; ảnh hiện có không đủ để khẳng định chúng nằm dưới mặt đường. |

## 5. Số từ cuboid của tôi (Bước 5)
Mặt đường khoảng −1,7 m. Số liệu lấy từ Bảng A và phiếu tự kiểm.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy (m) | dist_m (m) | r (m) | yaw (°) | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| Gần nhất: `s05f05`, 1, Car | −0,843 | 1,756 | −1,720 | 3,46 | 3,56 | 270,1 | Đáy gần mặt đường; `ground_gap_m` = 0,06 m |
| Xa nhất: `s01f55`, 7, Car | −1,104 | 1,525 | −1,866 | 38,80 | 38,81 | 270,5 | Đáy gần mặt đường; `ground_gap_m` = 0,05 m |
| Người đi bộ: `s01f24`, 10, Pedestrian | −0,760 | 1,659 | −1,589 | 15,26 | 15,28 | 240,9 | `ground_gap_m` = 0,06 m; hướng mặt/đi cần xác nhận |

Xe gần nhất và xa nhất đều có yaw gần 270°, phù hợp hướng cùng chiều. Xe ngược chiều là `s05f22`, idx 5, Car, yaw 90,1°, gần 90°.

Theo bảng 4.3, người ở idx 10 có kích thước 0,368 × 0,608 × 1,659 m: chiều dài thấp hơn mốc người lớn 0,4 m khoảng 8%, còn rộng và cao nằm trong khoảng. Xe con dùng để so sánh là `s01f24`, idx 1, kích thước 4,394 × 1,772 × 1,458 m; cả ba chiều nằm trong khoảng sedan/hatchback.

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---:|---:|
| `s05f05`, 1, Car | 3,46 | 5.387 |
| `s05f22`, 3, Car | 14,72 | 591 |
| `s03f18`, 3, Car | 20,98 | 353 |
| `s01f55`, 7, Car | 38,80 | 44 |

Trong các ví dụ này, khoảng cách tăng khoảng 11 lần (3,46 lên 38,80 m) thì số điểm trong hộp giảm khoảng 123 lần (5.387 xuống 44). Xu hướng giảm theo khoảng cách phù hợp với đường cong trong slide, nhưng số điểm từng hộp khác trung vị do vật, góc quét và kích thước hộp khác nhau.

Tự kiểm: cảnh báo `quá chật` ở `s01f24`, idx 10 và 11 đã được kiểm tra và xác nhận đúng. Vì hai hộp Pedestrian gần nhau, cần phân biệt điểm ngoài mép thuộc người nào trước khi nới để không gộp hai vật.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
Nhãn 3D dùng mét vì mô hình cần học vị trí, kích thước và hướng vật trong không gian thật; pixel chỉ mô tả vị trí trên một ảnh camera. Vật thưa điểm cần quy tắc riêng vì hình dạng bị thiếu khiến hộp và class dễ suy đoán sai, nên chỉ gán khi có đủ phần quan sát hoặc bằng chứng bổ sung.

- Công cụ AI: Codex.
- Câu hỏi chính: hỗ trợ hoàn thiện báo cáo bằng cách đối chiếu hướng dẫn, Bảng A và phiếu tự kiểm.
- Cách dùng: dùng để kiểm tra số liệu và soạn diễn giải.

Điều còn chưa chắc: kích thước xe buýt mẫu chưa phù hợp bảng 4.3; điểm ngoài mép của hai hộp Pedestrian cần phân loại đúng vật trước khi chỉnh.