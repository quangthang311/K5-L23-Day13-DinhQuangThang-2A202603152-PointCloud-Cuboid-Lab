# Báo cáo Day 13 — Đinh Quang Thắng - 2A202603152

## 1. Kết luận trước

Nhiệm vụ `D13-2A202603152` gồm 10 frame: `s01f36`, `s01f54`, `s02f29`, `s02f55`, `s03f24`, `s03f58`, `s04f00`, `s04f58`, `s05f15` và `s05f31`.

Bảng A có **62 cuboid**. Năm frame có ít hộp nhất là `s01f36`, `s02f55`, `s03f24`, `s03f58` và `s05f31`, mỗi frame có 5. Riêng `s01f36`, các vật còn lại không có đủ điểm để gán nhãn. Script kiểm tra cũng báo frame này thấp hơn mốc 8 cuboid. Phiếu tự kiểm không gắn cờ cuboid nào; Bảng A có một cuboid được đánh dấu occluded (`s05f15`, idx 6). Tôi thấy hướng yaw của những xe có nhiều điểm khá rõ; khó đánh giá hơn là class và kích thước của vật thưa ở xa, nhất là Pedestrian `s03f24`, idx 4, chỉ có 20 điểm.

Trong CVAT, tôi dùng góc nhìn 3D và Top để định vị vật. Ảnh T2 cũng có các view Top/Side/Front và ảnh camera để đối chiếu hình hộp, mũi tên `Cuboid orientation`, loại vật và đầu xe.

## 2. Đọc frame (Bước 2)

Ảnh T1: `t1-s01f36-3d.png`, `t1-s01f36-top.png`, `t1-s01f54-3d.png`, `t1-s01f54-top.png`. Hướng nhìn trên ảnh top khác nhau giữa hai frame: ở `s01f36`, phía trước (−y) ở cạnh trên, +x sang trái, +y xuống; ở `s01f54`, phía trước (−y) ở cạnh dưới, +x sang trái, +y lên. Theo hệ VF, +x luôn sang trái, +y ra sau; mặt đường danh định khoảng 1,7 m dưới gốc LiDAR. Xa nhất dưới đây là vật đã có cuboid trong Bảng A, không phải tầm xa tối đa của toàn bộ point cloud.

| Frame | Hướng trên ảnh top | Mặt đường | Vật xa nhất đã gán | Loại vật trong Bảng A | Vật phía sau / hai bên | Cụm chưa đọc chắc |
|---|---|---|---|---|---|---|
| `s01f36` | −y ở trên; +x trái, +y dưới | Tương đối phẳng; khoảng 1,7 m dưới LiDAR | Cyclist, idx 3: 11,83 m | 2 Car, 3 Cyclist | Có vật ở phía sau (+y) và bên phải (−x); cam01 chỉ nhìn trước | Các cụm điểm nhỏ ở xa/rìa cảnh không đủ căn cứ để nhận thêm vật |
| `s01f54` | −y ở dưới; +x trái, +y trên | Tương đối phẳng; khoảng 1,7 m dưới LiDAR | Pedestrian, idx 9: 14,32 m | 6 Car, 1 Cyclist, 2 Pedestrian | Có xe phía sau (+y) và các vật phía bên phải (−x) | Các mục tiêu nhỏ ở xa chưa thể phân loại chắc chỉ từ point cloud |

Ảnh camera giúp nhận biết màu sắc, dáng vật, người lái và đầu/đuôi xe. LiDAR cho biết vị trí, kích thước theo mét, phân bố điểm và hình dạng ở những phía camera không nhìn thấy; từ đó có thể kiểm tra đáy hộp với mặt đường.

| Frame | Hướng | Điều đáng chú ý từ Bảng A và phiếu tự kiểm |
|---|---|---|
| `s02f29` | −y trước; cạnh ảnh không ghi nhận | 6 Car; có xe idx 3 yaw 90,9°, ngược hướng so với xe cùng chiều. |
| `s02f55` | −y trước; cạnh ảnh không ghi nhận | 5 Car; các xe xa nhất được gán ở khoảng 34 m. |
| `s03f24` | −y trước; cạnh ảnh không ghi nhận | Pedestrian idx 4 ở 48,65 m chỉ có 20 điểm; hiện tượng thưa ở xa. |
| `s03f58` | −y trước; cạnh ảnh không ghi nhận | Có Bus và Cyclist; kích thước Bus lệch một số khoảng tham chiếu (xem mục 3). |
| `s04f00` | −y trước; cạnh ảnh không ghi nhận | 7 Car. |
| `s04f58` | −y trước; cạnh ảnh không ghi nhận | Có 4 Cyclist; vật xa nhất đã gán ở 35,25 m. |
| `s05f15` | −y trước; cạnh ảnh không ghi nhận | Car idx 6 được đánh dấu occluded; ảnh hiện tượng bóng che và phản xạ thấp lấy ở frame này. |
| `s05f31` | −y trước; cạnh ảnh không ghi nhận | Car idx 3 gần nhất toàn bộ dữ liệu, dist_m 4,22 m. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

| Vật | Frame, idx | Class | Mũi tên yaw | Đáy chạm đường | Đã kiểm Top/Side/Front + ảnh | So với bảng 4.3 |
|---|---|---|---|---|---|---|
| (a) Xe con gần trong ảnh T2 | `s01f36`, idx 2 | Car | Hướng phù hợp; yaw 279,4° | Có; ground gap 0,05 m | Ảnh T2 có ba view và các camera | 4,321 × 2,052 × 1,492 m. Dài và cao nằm trong khoảng sedan; rộng hơn mức sedan 0,152 m nhưng vẫn trong khoảng SUV/minivan. |
| (b) Xe buýt | `s03f58`, idx 2 | Bus | Hướng phù hợp; yaw 270,1° | Có; ground gap 0,06 m | Ảnh T2 có ba view và các camera | 12,397 × 3,190 × 2,746 m. Chiều dài nằm trong khoảng xe buýt thường; rộng hơn giới hạn 2,60 m và thấp hơn giới hạn 3,00 m. |
| (c) Xe hai bánh có người | `s03f58`, idx 5 | Cyclist | Hướng phù hợp; yaw 270,9° | Chưa chắc; ground gap +0,26 m, cần xem lại Side/Front | Ảnh T2 có ba view và các camera | 1,501 × 0,740 × 1,289 m. Kích thước chưa khớp trọn khoảng tham chiếu của xe đạp hoặc xe máy có người; cần kiểm tra lại hộp và loại xe. |

## 4. Bốn hiện tượng chất lượng (Bước 4)

Các frame dưới đây thuộc task cá nhân `D13-2A202603152`. Nguyên nhân nêu ra chỉ là giả thuyết.

| Hiện tượng | Frame / ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán |
|---|---|---|---|---|
| 1. Bóng che | `s05f15`; `t4-1-bong-che.png`; task cá nhân | Xe gần có thể chắn tia LiDAR tới vùng phía sau | Có vùng thiếu điểm phía sau vật che; Bảng A đánh dấu Car idx 6 là occluded | Bật occluded khi vật bị che; chỉ suy luận phần khuất khi có căn cứ, không xem vùng trống là đường chắc chắn |
| 2. Thưa ở xa | `s03f24`; `t4-2-thua-o-xa.png`; task cá nhân | Điểm phản hồi giảm theo khoảng cách | Pedestrian idx 4 ở 48,65 m có 20 điểm | Chỉ gán khi thấy khoảng 15–20% thể tích hoặc ảnh camera đủ rõ; nếu không, bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm | `s05f15`; `t4-3-phan-xa-thap.png`; task cá nhân | Màu tối, kính hoặc góc tới có thể làm giảm phản hồi | Camera thấy xe tối màu; điểm trên bề mặt xe thưa | Không nới hộp theo suy đoán; dùng point cloud và ảnh làm căn cứ, không khẳng định nguyên nhân |
| 4. Điểm lạ | `s01f36`; `t4-4-diem-la.png`; task cá nhân | Có thể là nhiễu hoặc phản xạ nhiều đường; chưa xác định | Có điểm rời/cụm điểm không gắn rõ với vật trong khung 3D | Không tạo cuboid cho điểm lẻ; chỉ gán vật khi có đủ bằng chứng |

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy (m) | dist_m (m) | r (m) | yaw_deg | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| `s05f31`, 3, Car | −0,970 | 1,518 | −1,729 | 4,22 | 4,33 | 270,3° | Chạm đường theo ground gap 0,06 m |
| `s03f58`, 2, Bus | −0,298 | 2,746 | −1,671 | 22,53 | 22,53 | 270,1° | Chạm đường theo ground gap 0,06 m |
| `s03f24`, 4, Pedestrian | −0,258 | 1,953 | −1,234 | 48,65 | 48,65 | 281,0° | Ground gap cục bộ 0,01 m; đáy cao hơn mức đường danh định −1,7 m khoảng 0,47 m, nên cần xem xét địa hình/ước lượng mặt đường tại vị trí xa |

Cuboid gần nhất là `s05f31`, idx 3: dist_m 4,22 m, r 4,33 m. Xa nhất là `s03f24`, idx 4: dist_m 48,65 m, r 48,65 m. Xe con `s05f31`, idx 3 có kích thước 4,494 × 2,158 × 1,518 m. Chiều dài và cao nằm trong khoảng sedan; chiều rộng vượt mức sedan 0,258 m và vượt giới hạn SUV/minivan 0,058 m, nên cần kiểm tra lại hộp hoặc loại xe. Cyclist `s03f58`, idx 5 có kích thước 1,501 × 0,740 × 1,289 m. So với xe đạp có người, hộp ngắn hơn 0,099 m, rộng hơn 0,040 m và thấp hơn 0,311 m; so với xe máy có người, hộp ngắn hơn 0,299 m và thấp hơn 0,111 m. Cần xem lại trên camera và ba view. Xe cùng chiều phía trước `s01f36`, idx 1 có yaw 279,4° (gần 270°); xe ngược chiều `s02f29`, idx 3 có yaw 90,9° (gần 90°).

| Cuboid (frame, idx, class) | dist_m (m) | points_in |
|---|---:|---:|
| `s05f15`, 1, Car | 4,85 | 5.554 |
| `s03f58`, 4, Car | 14,99 | 718 |
| `s03f24`, 1, Car | 24,76 | 204 |
| `s03f24`, 2, Car | 37,50 | 108 |

Trong các mẫu này, khoảng cách tăng từ 4,85 m lên 37,50 m (khoảng 7,7 lần), còn số điểm giảm từ 5.554 xuống 108 (khoảng 1,9%). Vì đây là các xe khác nhau nên hình dạng, góc và mức che khuất cũng ảnh hưởng. Mẫu 10–20 m và 20–30 m gần các mốc trên slide (718 và 214 điểm); mẫu 0–10 m cao hơn mốc 1.968. Mẫu ở 37,50 m thuộc bin 30–50 m nên không so trực tiếp với mốc 30–40 m.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D dùng mét để mô hình học vị trí, kích thước và hướng của vật trong không gian, thay vì chỉ học vị trí trên ảnh camera. Vật ở xa thường thưa điểm; cần quy tắc riêng để tránh bỏ sót hoặc gán hộp khi chưa đủ căn cứ.

- **Công cụ:** AI assistant.
- **Câu hỏi chính:** So sánh số liệu, đối chiếu Bảng A, phiếu tự kiểm và ảnh.
- **Cách dùng kết quả:** Dùng phần tổng hợp làm bản nháp, rồi tự kiểm tra lại số liệu với CSV và ảnh trong repo.

Điểm cần xem lại là class và kích thước của Cyclist `s03f58`, idx 5, cùng Pedestrian `s03f24`, idx 4 vì chỉ có 20 điểm. Nguyên nhân của các điểm lạ và phản hồi thấp cũng chưa xác định được.