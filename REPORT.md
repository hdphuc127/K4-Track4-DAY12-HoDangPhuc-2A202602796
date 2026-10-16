# REPORT — Lab Day 12

| | |
|---|---|
| Họ và tên | Hồ Đăng Phúc |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | 202602796 |
| GitHub | hdphuc127 |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | 3820 / 4421 |
| Số seed huấn luyện bạn dùng (`--seeds`) | 3 (4421, 4422, 4423) |

## Kết quả cuối

Dải qua 3 seed (min–max); constant-velocity và IDM là luật tất định nên chỉ có 1 giá trị.

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1,259 | 20,40 | CHẶN (R2, R3, R4, R6) |
| IDM | 1,545 | 0,17 | (baseline) |
| EgoMLP | 0,774 (0,769–0,781) | 19,44 (18,18–20,74) | CHẶN (R2–R6) |
| PercMLP | 0,627 (0,622–0,637) | 1,70 (1,01–2,81) | CHẶN (R2–R6) |
| PercMLP+ (sau khi sửa) | 0,820 (std 0,013) | 0,44 (0,38–0,49) | QUA (cả 7 luật) |

## Câu 1 · Dữ liệu
*Tóm tắt: 383 quỹ đạo trùng bị loại, `Acc_LV` không dùng được, chia theo quỹ đạo.*

Bước 0 cho thấy: 992 679 dòng, 9 993 quỹ đạo, bước thời gian 0,1 s, mỗi quỹ đạo 70–109 bước. Có 383 quỹ đạo trùng hệt nhau nhưng nằm ở các id cách xa nhau, còn lại 9 610 quỹ đạo sạch. Cột `Acc_LV` không dùng được: tương quan với d(Speed_LV)/dt chỉ 0,045 (trong khi cột gia tốc của xe theo sau là 0,929), nên nó không phải gia tốc thật của xe dẫn. Tôi thay bằng gia tốc xe dẫn tính từ `Speed_LV` đã làm mượt. Khoảng cách khớp `pos_diff − length` (lệch tối đa 3,6e-11 m, offset trung bình 4,5 m). Phải chia theo quỹ đạo (không chia theo cửa sổ/khung) vì các cửa sổ liền nhau của cùng một quỹ đạo gần như giống nhau; chia theo khung sẽ để train và test chứa cùng một chuyến. Các bản trùng ở id xa nhau cũng gây rò rỉ nếu không loại trước khi chia.

## Câu 2 · Phát lại
*Tóm tắt: refactor 0/600 vượt dung sai, AEB 9/600, real-time lộ 372/600 đoạn lệch.*

Trên 600 đoạn: bản refactor float32 không khớp bit ở cả 600 đoạn nhưng vượt dung sai (atol = rtol = 1e-3) ở 0 đoạn; sai khác lớn nhất 2,1e-5 m/s². Chạy lại cùng mã hai lần thì giống hệt (0 đoạn lệch). Thay đổi AEB (phanh khẩn khi TTC < 1,5 s) làm lệch 9/600 đoạn vượt dung sai (10 đoạn không khớp bit), sai khác lớn nhất 1,84 m/s². Lệch ít vì AEB chỉ kích hoạt ở những khung xe đang tiến sát lead (closing > 0 và gap/closing < 1,5 s); mọi khung còn lại lệnh vẫn là IDM thuần nên giống hệt. Phát theo thời gian thật lộ ra điều mà chế độ nhanh nhất che đi: 577 khung bị rơi và 582 khung trễ trên 59 511 khung, khiến 372/600 đoạn lệch vượt dung sai (tối đa 13,19 m/s²), trong khi chế độ "fastest" có 0 rơi, 0 trễ, 0 đoạn lệch.

## Câu 3 · Kịch bản
*Tóm tắt: 2,2 % → 7,4 % ô không tránh được khi trễ 0 → 0,6 s; 8,6 % lưới vượt giới hạn phanh người.*

Lưới cắt ngang ở v_ego = 12 m/s, khoảng cách 2,76–59,3 m, dv 0–12 m/s, tổng 2 700 ô. Với độ trễ phản ứng 0 s có 2,2 % ô không ai tránh được; với 0,6 s là 7,4 %. Vùng đâm của IDM đúng bằng vùng không tránh được ở cả hai độ trễ (phần quy được cho bộ điều khiển = 0), nên trễ 0 → 0,6 s làm vùng đâm của IDM tăng từ 2,2 % lên 7,4 % (~3,3 lần); simulator khớp biên vật lý (0 ô lệch). Log phủ vùng khó chưa đủ: khai thác được 614 sự kiện phanh (6,4 % quỹ đạo), giới hạn phanh mạnh nhất của người lái (p99,9) chỉ khoảng 3,62 m/s², trong khi 8,6 % lưới đòi hỏi phanh mạnh hơn mức đó — phần này không có tương ứng trong log.

## Câu 4 · Vòng hở và vòng kín

| Bộ điều khiển | L2 (m) | Hạng vòng hở | Va chạm trên log | Hạng vòng kín |
|---|---|---|---|---|
| PercMLP | 0,627 | 1 | 1,70 % | 2 |
| EgoMLP | 0,774 | 2 | 19,44 % | 3 |
| constant-velocity | 1,259 | 3 | 20,40 % | 4 |
| IDM | 1,545 | 4 | 0,17 % | 1 |

Các cặp đổi thứ tự: (const-vel, IDM), (IDM, EgoMLP), (IDM, PercMLP) — IDM tệ nhất ở vòng hở nhưng tốt nhất ở vòng kín. L2 của IDM cao vì nó là luật giữ khoảng cách với tham số riêng, không bắt chước gia tốc của người lái trong log; nhưng nó luôn phanh khi lead chậm lại (đỉnh phanh trung vị 0,96 m/s²) nên giữ được khoảng cách và chỉ 5/2 883 quỹ đạo va chạm. EgoMLP chỉ nhìn trạng thái xe mình nên học giá trị trung bình của người lái: L2 tốt hơn constant-velocity (0,77 so với 1,26) nhưng không thấy lead, đỉnh phanh trung vị chỉ 0,27 m/s², nên khi lead phanh nó vẫn đâm (19,4 %; trên nhóm lead phanh là 42 %). L2 là sai lệch trung bình trên mọi khung, phần lớn là cruise dễ, còn va chạm do một số ít khung then chốt.

## Câu 5 · Một lần chạy chưa đủ
*Tóm tắt: EgoMLP 18,2–20,7 % giữa 3 seed.*

Va chạm của EgoMLP qua 3 seed là 19,39 %, 18,18 %, 20,74 % (trung bình 19,44 %), dao động 2,6 điểm. Khoảng Wilson 95 % mỗi seed rộng khoảng ±1,5 điểm (seed 4421: 17,99–20,87 %; seed 4423: 19,30–22,26 %), nên sai khác giữa seed ngang cỡ độ bất định của mẫu test. Nếu chỉ chạy một seed tôi có thể báo 18,2 % hoặc 20,7 % tuỳ may; nguy hiểm vì một con số đơn lẻ trông chắc chắn. Với PercMLP rõ hơn: 1,0 % đến 2,8 % giữa các seed (gấp ~3 lần), chọn seed tốt sẽ trông như qua cổng R2, chọn seed xấu thì không.

## Câu 6 · Tác nhân phản ứng
Hiệu chỉnh T cho tài xế IDM mô phỏng bằng khoảng cách Wasserstein với phân bố log: T = 1,4 s (W1 = 0,351; tác nhân không phản ứng có W1 = 0,863; headway trung vị của người 1,86 s). Xe sau cách 18,8 m, trễ phản ứng 0,6 s. Tỷ lệ ô lưới có va chạm (t_react = 0,6 s, %):

| Bộ điều khiển | Xe sau phát lại (front / rear / any) | Xe sau phản ứng (any) |
|---|---|---|
| const-vel | 77,6 / 0 / 77,6 | 77,6 |
| IDM | 7,4 / 68,3 / 75,7 | 7,4 |
| EgoMLP | 76,0 / 0 / 76,0 | 76,0 |
| PercMLP | 20,8 / 56,0 / 76,7 | 22,0 |
| PercMLP+ | 9,1 / 65,9 / 75,0 | 9,1 |

Xe sau chỉ phát lại là khuôn cố định, không biết xe mình vừa phanh gấp, nên bộ điều khiển phanh mạnh (IDM, PercMLP+) bị xe sau "đâm" từ phía sau (68,3 % ô với IDM) — va chạm giả vì xe sau thật sẽ giảm tốc. Với xe sau phản ứng, lỗi phía sau về 0 và IDM chỉ còn 7,4 % (đúng bằng phần không tránh được). Với phát lại, mọi bộ điều khiển đều ~75–78 % và gần như không phân biệt được; kết luận "IDM tốt hơn rõ rệt" chỉ hiện ra trong thế giới phản ứng.

## Câu 7 · Cổng phát hành và sửa
*Tóm tắt: PercMLP bị chặn R2–R6; PercMLP+ qua cổng nhưng chưa tốt hơn IDM.*

PercMLP bị chặn bởi R2 (2,81 %, cận trên 3,48 % so với ngưỡng 0,17 % + 1 điểm), R3 (nhóm lead phanh 11,0 % so với 2,0 % + 5 điểm), R4 (14,6 % lưới so với 0 % + 2 điểm), R5 (1,0–2,8 % giữa các seed) và R6 (22,0 % so với 7,4 %). Nguyên nhân gốc: phanh gấp chỉ chiếm 4,6 % cửa sổ huấn luyện nên mô hình thấy rất ít tình huống nguy hiểm, và sai lệch nhỏ tích luỹ trong vòng kín đưa nó vào trạng thái chưa từng thấy; ở tốc độ cruise 20 m/s chưa thấy lúc huấn luyện, diện tích đâm quy được cho mô hình là 0,20–0,24. Cách sửa: thêm 20 000 kịch bản tổng hợp phủ vùng khó và 2 vòng DAgger — trực tiếp bù phần dữ liệu hiếm và lệch phân bố. Sau sửa: va chạm 0,38–0,49 %, lưới 1,7 %, ở 20 m/s còn 0,9–5,4 %; qua cả 7 luật. Nhưng PercMLP+ không tốt hơn IDM: 0,44 % so với 0,17 % trên log, diện tích quy được 0,2–1,7 % so với 0 % của IDM, L2 vòng hở xấu đi (0,82 so với 0,63), và ở 20 m/s vẫn còn đâm trong khi IDM thì không. Cổng chỉ nói "không tệ hơn quá một ngưỡng", nên tôi khuyến nghị chưa phát hành làm bộ điều khiển chính: chạy shadow mode / giữ IDM làm dự phòng và kiểm thêm ở nhiều tốc độ trước khi quyết định.

## Câu 8 · Bài lab này không chứng minh được gì
1. Độ phủ dữ liệu: chỉ 614 sự kiện phanh, người lái phanh tối đa ~3,6 m/s² nên 8,6 % lưới nằm ngoài những gì log từng thấy; kết luận về vùng khó dựa vào kịch bản tổng hợp, không phải hành vi thật.
2. Mô phỏng đơn giản: một tốc độ cruise chính (12 m/s, kiểm thêm 20 m/s), động lực học điểm, xe sau là IDM với T = 1,4 s hiệu chỉnh từ log, trễ cố định 0,6 s. Kết luận về va chạm phía sau phụ thuộc mạnh vào mô hình xe sau (75 % so với 7 % giữa phát lại và phản ứng).
3. Thống kê: 3 seed, 1 lần chia dữ liệu, 2 883 quỹ đạo kiểm thử; va chạm PercMLP đã dao động ~3 lần giữa các seed nên "qua cổng" của PercMLP+ (0,4–0,5 %) có thể không giữ khi thêm seed hay chia lại. Ngưỡng của cổng (+1, +2 điểm) cũng là lựa chọn tuỳ ý.
4. Chỉ đo va chạm, không đo độ thoải mái, nhiễu cảm biến thật (chỉ mô phỏng 1 m cho gap, 0,5 m/s cho tốc độ) hay độ trễ phần cứng.

Bước tiếp theo: mở rộng tốc độ và kịch bản (cut-in thật, đường cong), thêm seed và các lần chia dữ liệu, chạy shadow mode trên xe thật để so với IDM, rồi thử trên track kín có xe sau do người điều khiển trước khi ra đường.
