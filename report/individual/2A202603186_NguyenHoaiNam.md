# Báo cáo Day 13 — Nguyễn Hoài Nam, 2A202603186

## 1. Kết luận trước

- Mã số: 2A202603186 · task `D13-2A202603186`
- 10 frame: s01f17, s01f55, s02f19, s02f53, s03f16, s03f46, s04f38, s04f56, s05f08, s05f40
- Hệ điều hành: Windows · trình duyệt: Google Chrome · CVAT 2.81.0 (cvat-sdk 2.83.0, chạy được, chỉ có cảnh báo lệch phiên bản)
- MSSV bạn ngồi cạnh: 2A202603327

Tôi vẽ tổng cộng **115 cuboid** trên 10 frame (85 Car, 15 Cyclist, 8 Pedestrian, 4 Truck, 3 Bus). Frame ít nhất là **s05f40 (5 cuboid)**: một xe buýt dừng sát phía trước bên trái (cách 10,1 m) che gần hết góc nhìn phía trước bên trái, đường phía sau lại vắng nên ít vật để vẽ; frame nhiều nhất là s01f55 (20 cuboid, hàng xe dừng ở giao lộ và nhóm xe đạp bên phải). `selfcheck` cờ **0/115 cuboid**; một số cuboid xa (> 35 m) không có `ground_gap_m` vì quanh đó quá ít điểm mặt đường. Phần tôi tin nhất là **hướng** (xe cùng chiều 261°–280°, xe ngược chiều 89°–94°) và **đáy** (|`ground_gap_m`| ≤ 0,12 m với mọi cuboid có số). Phần yếu nhất là **vật xa 30–50 m**: kích thước chủ yếu lấy theo bảng 4.3 (4,5 × 1,8 m) và class của nhóm xe hai bánh ở xa dựa vào ảnh camera nhiều hơn point cloud.

Cách dùng CVAT:

- Đổi góc nhìn: kéo chuột trong khung 3D lớn để xoay, cuộn chuột để phóng to/thu nhỏ, nút mũi tên và `U I O J K L` để dời/nghiêng camera; xoay đến khi nhìn thẳng xuống để có ảnh nhìn từ trên xuống.
- Vẽ và chỉnh cuboid: chọn công cụ cuboid ở thanh trái → chọn label → **Shape**, đặt hộp lên cụm điểm; nháy đúp để chọn hộp rồi kéo cạnh/góc trong ba khung **Top / Side / Front** cho sát điểm và đáy chạm đường.
- Hướng và kiểm trên ảnh: bật **Appearance → Cuboid orientation** để thấy vạch đỏ (+X của hộp) chỉ đầu xe; chạy `python scripts/lab.py preview` để chiếu cuboid lên 7 ảnh camera và ảnh BEV trong `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-s01f17-3d.png`, `t1-s01f17-top.png`, `t1-s01f55-3d.png`, `t1-s01f55-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
| --- | --- | --- | --- | --- | --- | --- |
| s01f17 | Phía trước (−y) ở **cạnh trái** ảnh; +x (trái xe) chỉ **xuống dưới**; +y (sau xe) chỉ **sang phải** | Dốc nhẹ: đáy xe gần ~1,7–1,9 m dưới LiDAR; phía sau càng xa đáy càng cao (−1,56 m ở 22 m, −1,28 m ở 47 m), phía trước thấp hơn (−2,11 m ở 29 m) | ~47 m (Car idx7, phía sau, 62 điểm) | 7 Car, 1 Bus | Phía sau: 5 xe con cùng chiều ở 8–47 m (idx3–7, thấy trên cam09); phía sau bên phải: xe idx4 (14 m, cam08); bên phải là dải phân cách cây xanh và chiều đường ngược lại | Các cụm điểm dày dọc hai bên là bồn cây, cây xanh, hàng rào; vài cụm thấp xa phía trước bên phải sau dải phân cách quá ít điểm, camera không xác định được là xe → không vẽ |
| s01f55 | Phía trước (−y) ở **cạnh trái** ảnh; +x chỉ **xuống dưới**; +y chỉ **sang phải** | Phẳng, đáy xe quanh −1,80 đến −1,90 m (thấp hơn LiDAR ~1,85 m) | ~44 m (Cyclist idx18, phía trước bên phải, 15 điểm) | 13 Car, 1 Bus/xe khách nhỏ, 6 Cyclist | Phía sau: 2 xe con sát đuôi (7 m, idx12, idx13) và 1 xe ở 39 m (idx14); bên phải: nhóm 5 xe hai bánh ở 39–44 m và 1 người đạp xe trên vỉa hè 22 m (cam07) | Phía trước bên trái sau hàng xe dừng (> 40 m) có vài cụm thưa, bị hàng xe che, không đủ cơ sở → không vẽ |

Chỉ **camera** cho biết (từ hai frame trên):

1. Màu và kiểu xe: xe con gần nhất ở s01f17 là sedan màu trắng; vật vàng ở 29 m là xe buýt chở khách (thấy cửa, kính, bảng hiệu) chứ không phải xe tải.
2. Biển chỉ dẫn trên giá long môn, vạch kẻ làn và mũi tên trên mặt đường (s01f17, s01f55) — point cloud chỉ thấy mặt phẳng.
3. Nhóm vật nhỏ ở s01f55 bên phải là **người đi xe đạp/xe máy** (thấy người ngồi trên xe), point cloud chỉ có 12–19 điểm mỗi vật, không tách được người với xe.

Chỉ **LiDAR** cho biết:

1. Khoảng cách tính bằng mét: xe buýt ở s01f17 cách tâm 29,3 m, xe con gần nhất 8,2 m.
2. Vật phía sau và hai bên không có trong cam01: 5 xe phía sau ở s01f17, xe sát đuôi 7 m ở s01f55 — chỉ thấy toàn cảnh 360° trên ảnh nhìn từ trên xuống.
3. Độ dốc mặt đường: chênh lệch z đáy giữa xe phía trước (−2,11 m) và xe phía sau (−1,28 m) ở s01f17 cho thấy đường dốc nhẹ, camera không đo được.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
| --- | --- | --- |
| s02f19 | Cạnh trái | Đường có lan can giữa: xe ngược chiều (yaw ~90°) ở bên trái; người đi bộ và xe hai bánh trên vỉa hè bên phải, một xe hai bánh đi ngang (yaw 180°) |
| s02f53 | Cạnh trái | Đường hai bên cây cao, ít điểm hai bên do tán cây; xe ngược chiều sau hàng rào bên trái |
| s03f16 | Cạnh trái | Xe tải nhỏ sát bên trái (2,7 m, 8481 điểm) che hẳn phía bên trái; nhóm người đi bộ ở vạch qua đường xa 30–42 m bên phải |
| s03f46 | Cạnh trái | Xe đen phía trước 9,5 m có ít điểm hơn xe cùng tầm (dùng cho hiện tượng 3); hai xe máy bên phải 7–12 m |
| s04f38 | Cạnh trái | Đường trên cao có lan can vàng bên trái; xe tải phía sau bên trái 15,6 m |
| s04f56 | Cạnh trái | Xe SUV trắng sát phía sau bên phải 6 m (4224 điểm, đông nhất frame); xe hai bánh xa 39 m phía trước |
| s05f08 | Cạnh trái | Giao lộ lớn, nhiều xe các hướng; vạch qua đường cho rất nhiều điểm mặt đường |
| s05f40 | Cạnh trái | Xe buýt dừng sát phía trước bên trái (10,1 m, 7124 điểm) che phần lớn góc phía trước bên trái |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
| --- | --- | --- | --- | --- | --- | --- |
| (a) xe con gần | s01f17, idx1 | Car | Có, yaw 266° (cùng chiều, ~270°) | Có, z đáy −1,87 m, `ground_gap_m` 0,02 m | Có, cam01/cam03 thấy đuôi sedan trắng | Có: 4,25 × 1,80 × 1,53 m (sedan 3,8–5,0 × 1,70–1,90 × 1,40–1,60) |
| (b) xe buýt hoặc xe tải | s01f17, idx2 | Bus | Có, yaw 273° | Có, z đáy −2,11 m, `ground_gap_m` 0,03 m (đường dốc xuống phía trước) | Có, cam03 thấy đuôi xe buýt; bật occluded vì đầu xe bị che | Có: 11,0 × 2,55 × 3,38 m (buýt thường 10,0–12,5 × 2,50–2,60 × 3,00–3,50); chiều dài suy theo bảng 4.3 vì đầu xe bị che |
| (c) người đi bộ hoặc xe hai bánh | s01f55, idx20 | Cyclist | Có, yaw 270° (đi cùng chiều) | Có, đáy −1,29 m đặt trên mặt vỉa hè (cao hơn mặt đường), `ground_gap_m` 0,00 m | Có, cam07 thấy người đạp xe đạp | Gần đủ: 1,75 × 0,60 × 1,55 m; cao 1,55 m thấp hơn khoảng 1,60–1,90 m của xe đạp có người (chỉ 53 điểm, phần đầu người thưa) |

Vòng hiệu chuẩn: không tham gia (không tạo và không vẽ trên task chung `D13-HC485087`), nên không có số liệu so sánh với các bạn.

Vật bỏ qua vì không đủ cơ sở:

- s01f17, phía trước bên phải sau dải phân cách, xa > 40 m: vài cụm thấp, camera không thấy rõ là xe → UNCERTAIN_SCOPE / LOW_VISIBILITY.
- s01f55, phía trước bên trái sau hàng xe dừng, > 40 m: cụm thưa bị che → LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Tất cả lấy từ frame của tôi. Nguyên nhân đều là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
| --- | --- | --- | --- | --- |
| 1. Bóng che | s01f55 idx9 (Car, 20,8 m) · `t4-1-bong-che.png` · frame của tôi | Giả thuyết: xe trắng idx1 ngay phía trước (10,3 m) chắn tia laser, chỉ phần nóc xe idx9 cao hơn mới nhận tia | Chỉ 62 điểm, gần như chỉ có điểm phần nóc trong Top/Side/Front; xe cùng tầm không bị che (s01f55 idx5, 16,3 m) có 416 điểm | Bật **occluded**; cuboid vẫn bao cả phần suy luận được với kích thước xe con đầy đủ 4,5 × 1,8 m, đáy đặt xuống mặt đường chứ không theo đáy cụm điểm |
| 2. Thưa ở xa | s01f55 idx18 (Cyclist, 43,9 m) · `t4-2-thua-o-xa.png` · frame của tôi | Giả thuyết: khoảng cách giữa các tia tăng theo khoảng cách, vật nhỏ ở xa chỉ bị vài tia quét qua | Chỉ 15 điểm trong hộp, ba khung Top/Side/Front gần như trống | Theo quy tắc 15–20%: vẽ vì camera cam03 thấy rõ người đi xe và cụm điểm đủ suy vị trí; kích thước theo bảng 4.3, ghi **LOW_VISIBILITY**. Nếu không có camera xác nhận thì bỏ qua |
| 3. Phản xạ thấp / ít điểm bất thường | s03f46 idx2 (Car màu đen, 9,5 m) · `t4-3-phan-xa-thap.png` · frame của tôi | Giả thuyết: sơn đen và kính xe hấp thụ/phản xạ lệch tia laser nên ít điểm quay về | 1195 điểm ở 9,5 m, ít hơn hẳn xe đỏ cùng frame ở 4,4 m (2614 điểm) và xe ở cùng tầm frame khác (s03f16 idx3, 9,9 m: 1480 điểm); slide: ~1968 điểm ở 0–10 m. Mặt sau trong khung Front lấm tấm | Không khẳng định nguyên nhân; vẫn bao theo hình dạng xe, đối chiếu camera để hộp không bị hụt so với thân xe thật |
| 4. Điểm lạ | s01f55, dưới đuôi Car idx2 · `t4-4-diem-la.png` · frame của tôi | Giả thuyết: tia laser phản xạ nhiều lần qua thân xe bóng/kính tạo điểm ảo | Khoảng 5 điểm ở z ≈ −3,1 m, tức thấp hơn mặt đường ~1,3 m, ở x ≈ 2,1 m, y ≈ −6,4 m | **Không gán thành vật**, không kéo đáy hộp xe idx2 xuống theo các điểm này |

## 5. Số từ cuboid của tôi (Bước 5)

Số lấy từ `submission/D13-2A202603186-bangA.csv`; số điểm từ cột `points_in` của `D13-2A202603186-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| s01f17, 1, Car | −1,108 m | 1,525 m | −1,871 m | 8,20 m | 8,27 m | 266° | Chạm đường (`ground_gap_m` 0,02 m) |
| s01f55, 20, Cyclist | −0,515 m | 1,550 m | −1,290 m | 21,98 m | 21,98 m | 270° | Chạm mặt vỉa hè (gap 0,00 m); cao hơn −1,7 m ~0,4 m nhưng không lơ lửng vì vật đứng trên vỉa hè |
| s01f17, 7, Car | −0,535 m | 1,490 m | −1,280 m | 47,09 m | 47,09 m | 270° | Chạm đường (gap 0,00 m); lệch −1,7 m ~0,42 m do đường dốc lên phía sau, không phải lơ lửng |

(a) Cuboid **gần nhất**: s03f16 idx1 Truck, `dist_m` = 2,71 m, r = 2,81 m. Cuboid **xa nhất**: s01f17 idx7 Car, `dist_m` = 47,09 m, r = 47,09 m (z chỉ −0,54 m nên r gần bằng `dist_m`; với vật gần thì z đóng góp nhiều hơn: 2,81 m so với 2,71 m).

(b) Người đi bộ s02f19 idx12: 0,74 × 0,56 × 1,68 m, cả ba chiều nằm trong khoảng người lớn (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m). Xe con s01f17 idx1: 4,25 × 1,80 × 1,53 m, nằm trong khoảng sedan (3,8–5,0 × 1,70–1,90 × 1,40–1,60 m). Ca ngoài khoảng: Cyclist s02f19 idx14 rộng 1,19 m, vượt khoảng xe máy có người (0,70–0,95 m); giả thuyết là vật đang đi ngang (yaw 180°) nên điểm của tay lái và người trải rộng, cộng với điểm của vật sát bên (idx15 cách ~1,1 m) bị gộp vào hộp. Cyclist s01f55 idx20 cao 1,55 m, thấp hơn 1,60 m vì đầu người chỉ có vài điểm.

(c) Xe cùng chiều phía trước: s01f17 idx1, yaw = 266°, gần 270° ✓. Xe ngược chiều: s02f19 idx2 (bên trái phía trước, 12,2 m), yaw = 92°, gần 90° ✓.

Mật độ theo khoảng cách (cùng frame s01f17 để cùng điều kiện quét):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
| --- | --- | --- |
| s01f17, 1, Car | 8,20 m | 1731 |
| s01f17, 4, Car | 14,43 m | 800 |
| s01f17, 5, Car | 22,34 m | 284 |
| s01f17, 6, Car | 34,56 m | 30 |

Quy luật: xa gấp khoảng 2,7 lần (8,2 → 22,3 m) thì số điểm còn khoảng 1/6 (1731 → 284), giảm nhanh xấp xỉ theo 1/d² hoặc hơn. So với đường cong ở slide (1968 / 718 / 214 / 72 điểm ở 0–10 / 10–20 / 20–30 / 30–40 m), ba cuboid đầu khớp khá sát (1731 / 800 / 284); riêng idx6 ở 34,6 m chỉ 30 điểm, thấp hơn mức 72 — giả thuyết là idx6 nằm cùng làn ngay sau idx5 (x 0,8 m so với 0,2 m) nên bị idx5 che một phần.

Tự kiểm: sau `preview` / `selfcheck` tôi sửa hai cuboid (so bảng A trước và sau khi sửa): (1) s02f19 idx14 (Cyclist): dời tâm x từ −6,761 m sang −7,170 m, thu dài từ 2,00 m xuống 1,62 m và rộng từ 1,24 m xuống 1,19 m, chỉnh yaw từ 183° về 180°; (2) s02f53 idx5 (Car): hạ tâm z từ −0,883 m xuống −1,090 m và giảm cao từ 1,93 m xuống 1,88 m, nên z đáy hạ từ khoảng −1,85 m xuống −2,03 m cho đáy chạm mặt đường quanh xe. Ngoài ra vẽ thêm 6 hộp (s01f17 idx7, idx8; s01f55 idx20; s02f19 idx15; s03f16 idx19; s04f38 idx10). `selfcheck` không còn cờ nào nên không có cảnh báo phải giữ nguyên; một số cuboid xa không có `ground_gap_m` (ví dụ s01f55 idx8, idx11, idx15–19) vì quanh đó quá ít điểm mặt đường — tôi giữ nguyên, đáy đặt theo mặt đường của các xe gần cùng làn.

Đổi kiểm: đã đổi kiểm với 2A202603327 (chỉ xem trên màn hình, không gửi file). Bạn ấy đối chiếu hai con số trong mục 5 với bảng A mở trên máy tôi: z đáy s01f17 idx1 = −1,871 m và `dist_m` xa nhất = 47,09 m. Kết quả: khớp cả hai, không phải sửa gì.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Model 3D học trực tiếp trên tọa độ point cloud và phải xuất vị trí, kích thước bằng mét để hệ thống lái tính khoảng cách phanh, làn đường; nhãn tính bằng pixel thay đổi theo camera và khoảng cách nên model không học được kích thước thật của vật. Vật thưa điểm cần quy tắc riêng vì nếu người gán lúc vẽ lúc bỏ, hoặc đoán kích thước tùy ý, model sẽ học tín hiệu mâu thuẫn ở vùng xa — vừa bỏ sót vật thật vừa báo nhầm vật từ vài điểm nhiễu.

(b) Cách dùng AI:

- Công cụ: Claude (Anthropic).
- Câu hỏi chính: cài đặt và cấu hình lab (`configs/lab.ini`, `.venv`, pip), gỡ lỗi CVAT local trên Docker (traefik không mount được file cấu hình sau khi chuyển thư mục CVAT, Redis `cvat_redis_inmem` hỏng file AOF), lệnh Git và cách đọc kết quả `check_submission.py`.
- Cách dùng kết quả: làm theo lệnh gợi ý để sửa môi trường. AI không vẽ cuboid.
(c) Điều còn chưa chắc:
- s01f55 idx11 gán Bus nhưng dài 6,0 m, ngoài khoảng xe buýt thường; có thể là xe khách nhỏ, cần hỏi coach nên là Bus hay Car (UNCERTAIN_CLASS).
- s01f17 idx6 có thể bị idx5 che một phần nhưng tôi chưa bật occluded (ATTRIBUTE_CHECK).
- Nhóm xe hai bánh ở s01f55 (39–44 m) và nhóm người đi bộ ở s03f16 (40–43 m): vị trí từng vật trong nhóm dựa nhiều vào camera, hộp có thể lệch quá 0,2 m so với nhãn chuẩn.
