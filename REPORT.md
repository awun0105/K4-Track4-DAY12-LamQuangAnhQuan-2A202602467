# REPORT — Lab Day 12

> Báo cáo này dùng số liệu từ lần chạy trên Google Colab với MSSV `202602467` và `--seeds 3` (mỗi mô hình học huấn luyện 3 seed). Mọi con số lấy từ `evidence/*.json`; `evidence/verify.txt` là kết quả kiểm tra tự động.

| | |
|---|---|
| Họ và tên | `Lâm Quang Anh Quân` |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `202602467` |
| GitHub | `awun0105` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `7379` / `7992` (các seed mô hình 7992–7994) |
| Số seed huấn luyện bạn dùng (`--seeds`) | `3` (chạy với `--seeds 3`; mặc định của lệnh là 5) |

## Kết luận: có nên phát hành bản mới không?

**Trả lời ngắn: chưa nên thay IDM bằng PercMLP+.** PercMLP+ đã qua cổng phát hành (7/7 quy tắc), nhưng cổng chỉ chứng minh rằng nó *không kém* IDM trong giới hạn cho phép. Không có bằng chứng nào cho thấy nó *tốt hơn* IDM.

Ba lý do chính:
- **Trên log thật**, PercMLP+ đâm 7 trên 2.883 quỹ đạo (ở seed xấu nhất), còn IDM chỉ đâm 2 trên 2.883.
- **Khi tốc độ tăng lên 20 m/s**, tức ngoài khoảng dữ liệu huấn luyện (tối đa 16 m/s), PercMLP+ đâm 1,2–6,2 % lưới trong khi IDM vẫn là 0,0 %.
- **L2 vòng hở của PercMLP+ là 0,800 m**, tệ hơn PercMLP (0,621 m). Sửa mô hình giúp nó an toàn hơn trong các kịch bản đã thử, nhưng không giúp nó bám người lái tốt hơn.

Đề xuất trước khi quyết định:
1. Kiểm thử với nhiều cách chia dữ liệu và nhiều seed hơn (ít nhất 5 seed như mặc định), để chắc các con số không chỉ đúng với một split (7379) và ba seed này.
2. Thử ở tốc độ 16–20 m/s và các ca phanh mạnh hơn 3,6 m/s², là mức mạnh nhất người lái trong log từng làm.
3. Chạy PercMLP+ song song với IDM ở chế độ quan sát (chỉ ghi lại đề xuất, không điều khiển xe) trên dữ liệu thật. Chỉ sau đó mới xét cho phép điều khiển.

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json`. Với mô hình học, con số là **dải qua 3 seed**, từ seed tốt nhất đến seed xấu nhất.

Cách đọc bảng:
- **L2 vòng hở** là sai lệch vị trí dự đoán so với người lái thật, đo bằng mét. Thấp hơn nghĩa là bám người lái tốt hơn. Số sau dấu ± là độ lệch chuẩn giữa các seed.
- **Va chạm vòng kín** là tỷ lệ quỹ đạo trên log mà xe tự lái đâm xe dẫn. Thấp hơn nghĩa là an toàn hơn.
- **Quyết định cổng** là kết quả kiểm tra 7 quy tắc. "CHẶN (R2, R3, …)" liệt kê các quy tắc bị rớt.

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1,254 | 19,6 | CHẶN (R2, R3, R4, R6) |
| IDM | 1,500 | 0,07 (2 trên 2.883 quỹ đạo) | (baseline) |
| EgoMLP | 0,758 ± 0,002 (3 seed) | 16,1 … 32,5 (trung bình 23,8; seed xấu nhất 32,5) | CHẶN (R2, R3, R4, R5, R6) |
| PercMLP | 0,621 ± 0,003 (3 seed) | 0,7 … 2,1 (trung bình 1,6; seed xấu nhất 2,1) | CHẶN (R2, R3, R4, R5, R6) |
| PercMLP+ (sau khi sửa) | 0,800 ± 0,014 (3 seed) | 0,21 … 0,24 (trung bình 0,23; seed xấu nhất 0,24, CI 95 % [0,12; 0,50]) | ĐẠT CỔNG (7/7): không kém IDM, chưa tốt hơn |

## Thuật ngữ nhanh

- **Vòng hở:** chấm mô hình bằng cách dự đoán lại đúng quỹ đạo người lái đã đi, không cho nó tự lái.
- **Vòng kín:** cho bộ điều khiển tự lái trong mô phỏng; xe dẫn được phát lại từ log, rồi đếm xem xe ta có đâm không.
- **L2:** sai lệch vị trí dự đoán so với người lái thật, trung bình trên các khoảng dự đoán 1, 2 và 3 giây (mét).
- **TTC (thời gian va chạm):** khoảng cách chia cho tốc độ tiến gần. TTC càng nhỏ thì càng nguy hiểm.
- **Khoảng cách thời gian (headway):** khoảng cách tới xe phía trước chia cho tốc độ, đo bằng giây. Người lái trong log trung bình giữ khoảng 1,87 s.
- **Seed:** một lần huấn luyện mô hình với số khởi tạo ngẫu nhiên khác nhau, trên cùng một bộ dữ liệu.
- **Dung sai:** sai lệch nhỏ được chấp nhận khi so sánh hai lần chạy (ở đây 0,001).
- **Quy tắc cổng R1–R7:** R1 phát lại tái lập được; R2 va chạm trên log; R3 ca xe dẫn phanh; R4 lưới cắt vào; R5 ổn định giữa các seed; R6 xe sau phản ứng; R7 đủ số tình huống.

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

- **Ba vấn đề ở dữ liệu:**
  1. Cột gia tốc của xe dẫn (`Acc_LV`) không khớp với sự thay đổi tốc độ của chính xe đó. Tương quan chỉ **0,045**, gần 0 nghĩa là gần như không liên quan. Trong khi đó, cột gia tốc của xe theo sau (`Acc_FAV`) khớp, tương quan **0,929**.
  2. Có **383** quỹ đạo trùng hệt nhau trên tổng **9.993** (khoảng 3,8 %). Các bản trùng nằm ở những mã ID cách xa nhau, nên không dễ nhận ra.
  3. Mỗi quỹ đạo dài 70–109 khung (10 khung mỗi giây), và các khung liền kề gần như giống nhau.
- **Cách xử lý:** không dùng `Acc_LV`. Thay bằng gia tốc tính từ tốc độ của xe dẫn: làm mượt bằng trung bình trượt 5 điểm, rồi lấy sai phân trung tâm. Ngưỡng "xe dẫn đang phanh" (−2 m/s²) được áp lên đại lượng này.
- **Kết quả làm sạch:** loại 383 quỹ đạo trùng, còn **9.610** quỹ đạo. Chia 30 % làm tập test: **2.883** quỹ đạo test và **6.727** quỹ đạo train (seed chia 7379).
- **Vì sao chia theo quỹ đạo:** hai khung liền kề của cùng một quỹ đạo gần như trùng nhau. Nếu chia theo khung, một phần khung giống hệt nhau sẽ nằm vừa ở train vừa ở test, và kết quả test sẽ đẹp giả tạo. Chia theo quỹ đạo thì tập test là dữ liệu mô hình chưa từng thấy.
- **Kiểm tra thêm:** khoảng cách trong log khớp công thức `Spatial_Gap = Pos_LV − Pos_FAV − 4,5 m`, sai lệch chỉ khoảng 3,6·10⁻¹¹ m, nên có thể tin dữ liệu khoảng cách.

**Kết luận:** dữ liệu thô không dùng được ngay. Nếu tin cột `Acc_LV`, ta sẽ tìm sai các ca phanh. Nếu không loại 383 bản trùng và không chia theo quỹ đạo, kết quả test sẽ bị thổi phồng. Mọi con số ở các câu sau đều dựa trên bước làm sạch này.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

- **Công cụ phát lại có tái lập được không?** Chạy lại cùng một mã thì **0 / 600** đoạn khác nhau, tức kết quả lặp lại được.
- **Bản refactor float32 có "đúng" không?** Cả **600 / 600** đoạn đều khác nhau từng bit, sai lệch lớn nhất 4,0·10⁻⁴. Nhưng **0 / 600** đoạn vượt dung sai 0,001. Khác từng bit không có nghĩa là sai, vì cộng số thực không có tính kết hợp: (0,1+0,2)+0,3 = 0,6000000000000001, còn 0,1+(0,2+0,3) = 0,6. Dung sai 0,001 đủ rộng để bỏ qua nhiễu số học, nhưng đủ hẹp để vẫn phát hiện thay đổi thật.
- **Thay đổi AEB làm lệch bao nhiêu đoạn?** AEB là phanh khẩn cấp, kích hoạt khi TTC dưới 1,5 s. Nó làm lệch **8 / 600** đoạn, không phải tất cả.
- **Vì sao lệch đúng những khung đó?** Mở đoạn 64: lệch ở 3 khung (75–77). Ở các khung này TTC tụt xuống 1,44–1,48 s, khoảng cách còn 4,4–5,0 m, và AEB ép lệnh từ khoảng −4,4 m/s² thành −6,0 m/s². Đoạn 78 chỉ lệch ở 1 khung (TTC 1,47 s).
- **Phát theo thời gian thật lộ ra điều gì?** Phát nhanh nhất cho **0** khung trễ và **0** khung rớt, vì nó không có hạn chót: khung sau được gửi ngay khi khung trước xử lý xong. Phát theo thời gian thật cho **581** khung trễ và **571** khung rớt trên tổng 59.551 khung. Nguyên nhân: khoảng 1 % khung gặp một lần xử lý chậm 140 ms, dài hơn chu kỳ 100 ms, nên lệnh đến muộn và các khung đến lúc đang bận bị bỏ. Kết quả là **378 / 600** đoạn đổi kết quả.

**Kết luận:** phát nhanh nhất che mất lỗi về thời gian, tức trễ và rớt khung, là loại lỗi xuất hiện khi xe chạy thật. Vì vậy kiểm thử phát lại phải mô phỏng thời gian thật, và phải có dung sai để không báo động giả vì nhiễu số học. Việc chỉ 8 / 600 đoạn lệch cũng cho thấy thay đổi AEB chỉ ảnh hưởng đúng vùng TTC thấp, không làm đổi hành vi lái thường ngày.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

- **Lưới tình huống:** 60 khoảng cách (từ 2,8 đến 59,3 m) × 45 tốc độ tiến gần (từ 0 đến 12 m/s; xe ta chạy cruise 12 m/s), tổng **2.700** ô. Mỗi ô là một tình huống có xe cắt vào.
- **Ô không ai tránh được** là các ô mà ngay cả khi phanh hết cỡ vẫn đâm. Con số là **2,2 %** khi không có độ trễ phản ứng, và **7,4 %** khi phản ứng trễ 0,6 s. Bộ mô phỏng khớp đúng ranh giới này, với 0 ô lệch ở cả hai độ trễ.
- **Độ trễ làm vùng đâm của IDM tăng bao nhiêu?** Từ **2,2 %** lên **7,4 %** lưới, tức tăng 5,2 điểm phần trăm. Vùng "tránh được nhưng vẫn đâm" của IDM là **0,0 %** ở cả hai độ trễ, nghĩa là mọi lần IDM đâm đều nằm trong vùng không ai tránh được.
- **Giới hạn phanh của người lái trong log:** người lái theo sau phanh mạnh nhất khoảng **3,6 m/s²** (phân vị 99,9 %), trong khi xe có thể phanh tới 9 m/s². **8,6 %** lưới đòi hỏi phanh mạnh hơn mức người lái từng làm.
- **Log có phủ vùng khó không?** Có 614 sự kiện phanh trong log, nhưng chỉ 33 sự kiện có tốc độ tiến gần trên 4 m/s, và **không có sự kiện nào** ở khoảng cách dưới 10 m với tốc độ tiến gần trên 4 m/s. Đó chính là vùng nguy hiểm nhất của lưới.

**Kết luận:** vùng nguy hiểm nhất (khoảng cách nhỏ, tiến gần nhanh) không có trong log. Mọi nhận định về mô hình học ở vùng đó vì vậy là suy diễn, không phải đo đạc. Còn IDM không đâm oan: mọi lần đâm của nó đều thuộc vùng không ai tránh được, nên nó là chuẩn so sánh hợp lý.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Bộ điều khiển | L2 vòng hở (hạng) | Đâm trên log, vòng kín (hạng; mô hình học lấy trung bình 3 seed) |
|---|---|---|
| PercMLP | 0,621 m (1) | 1,6 % (2) |
| EgoMLP | 0,758 m (2) | 23,8 % (4) |
| constant-velocity | 1,254 m (3) | 19,6 % (3) |
| IDM | 1,500 m (4) | 0,07 % (1) |

- **Cặp nào đổi thứ tự?** Có bốn cặp: (constant-velocity, IDM), (constant-velocity, EgoMLP), (IDM, EgoMLP), (IDM, PercMLP). Đáng chú ý nhất: IDM có L2 xấu nhất nhưng an toàn nhất, còn EgoMLP có L2 tốt hơn constant-velocity nhưng đâm nhiều hơn.
- **Vì sao IDM có L2 lớn mà vẫn lái tốt?** L2 đo xem mô hình có bám đúng tốc độ người lái trong 3 giây tới hay không. IDM khác người lái ở hai điểm: nó giữ khoảng cách thời gian 1,2 s, trong khi người lái trong log trung bình giữ 1,87 s; và dự báo của nó giả định xe dẫn giữ nguyên tốc độ. Vì vậy quỹ đạo của nó lệch khỏi người lái. Nhưng trong vòng kín, IDM thấy khoảng cách thật ở mỗi bước và phanh ngay, nên chỉ đâm 2 trên 2.883 quỹ đạo, cả hai đều là ca xe dẫn phanh.
- **Vì sao EgoMLP bám tốt hơn constant-velocity mà vẫn đâm?** EgoMLP chỉ nhìn tốc độ quá khứ của chính xe ta. Nhờ đó nó học được xu hướng tăng hoặc giảm tốc chung của người lái, nên L2 tốt hơn, kể cả trên các ca xe dẫn phanh (1,102 m so với 1,903 m). Nhưng nó không biết khoảng cách tới xe phía trước, nên không phanh kịp: trên 166 quỹ đạo có xe dẫn phanh, nó đâm 45–58 %, gần bằng constant-velocity (53,6 %).

**Kết luận:** L2 đo việc "giống người lái", không đo "an toàn". Một mô hình có thể bắt chước người lái khá tốt mà vẫn đâm, còn một bộ điều khiển có L2 lớn vẫn có thể an toàn hơn. Vì vậy không nên chọn bộ điều khiển chỉ bằng L2; phải kiểm tra bằng vòng kín.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

- **EgoMLP dao động bao nhiêu?** Từ **16,1 %** đến **32,5 %** qua 3 seed, dải hơn 16 điểm phần trăm: seed 7992: 22,6 %; 7993: 16,1 %; 7994: 32,5 %.
- **Khoảng Wilson nói gì?** Khoảng tin cậy 95 % của seed tốt nhất (7993: từ 14,8 đến 17,5 %) và của seed xấu nhất (7994: từ 30,8 đến 34,2 %) **không chồng nhau**. Vì vậy chênh lệch giữa các seed là thật, không chỉ là nhiễu khi lấy mẫu 2.883 quỹ đạo.
- **Nhưng Wilson có giới hạn:** nó chỉ đo sai số lấy mẫu trên một tập test, không cho biết kết quả thay đổi thế nào khi huấn luyện lại. Muốn biết điều đó phải xem dải qua các seed. PercMLP cũng dao động từ 0,7 đến 2,1 %, và seed tệ nhất làm nó rớt quy tắc R5 (dải không được vượt 1 điểm).
- **Nếu chỉ chạy một seed thì sao?** Chạy seed 7993, bạn sẽ báo "EgoMLP khoảng 16 %, tốt hơn constant-velocity (19,6 %)". Chạy seed 7994, bạn sẽ báo "32,5 %, hơn 1,6 lần constant-velocity". Hai kết luận trái ngược nhau chỉ vì chọn seed. Và 2 trên 3 seed của EgoMLP còn tệ hơn constant-velocity.

**Kết luận:** một lần chạy chỉ là một mẫu ngẫu nhiên, không phải sự thật. Cùng một mô hình, chỉ đổi seed đã có thể được báo là "tốt" hoặc "tệ". Vì vậy kết quả phải báo kèm dải qua các seed. Cổng phát hành dùng seed tệ nhất (R2), không dùng seed đẹp nhất, và đó là cách làm đúng khi quyết định phát hành.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

| Bộ điều khiển | Xe sau chỉ phát lại: tỷ lệ đâm cả lưới (t = 0 / 0,6 s) | Xe sau biết phản ứng: tỷ lệ đâm cả lưới (t = 0 / 0,6 s) | Đâm từ phía sau khi xe sau phát lại |
|---|---|---|---|
| IDM | 75,3 % / 75,7 % | 2,2 % / 7,4 % | 73,0 % / 68,3 % |
| PercMLP | 74,3 % / 73,3 % | 16,9 % / 31,0 % | 61,9 % / 43,2 % |
| constant-velocity | 77,6 % / 77,6 % | 77,6 % / 77,6 % | 0 % |
| EgoMLP | 77,3 % / 77,4 % | 77,3 % / 77,4 % | 0 % |

- **T hiệu chỉnh là bao nhiêu?** **1,4 s**. Đây là khoảng cách thời gian làm cho xe sau giống người lái trong log nhất. Độ giống được đo bằng W1, tức khoảng cách giữa hai phân bố khoảng cách thời gian (càng nhỏ càng giống). T = 1,4 s cho W1 = 0,36, còn một xe sau không bao giờ phản ứng có W1 = 0,86. Như vậy xe sau đã hiệu chỉnh giống người lái thật hơn khoảng 2,4 lần. (T = 1,2 s cho W1 gần như bằng.)
- **Vì sao IDM bị tính "va chạm giả"?** Xe sau chỉ phát lại giữ tốc độ đã ghi và không biết xe ta vừa phanh. Khi IDM phanh gấp để tránh xe dẫn, khoảng cách với xe sau co lại, và xe sau đâm vào đuôi xe ta. Con số **73,0 %** của IDM là va chạm giả, vì người lái thật phía sau sẽ phanh. Lỗi không nằm ở IDM.
- **Kết luận bị đổi thế nào?** Trong thế giới phát lại, IDM đâm **75,3 %** lưới, gần bằng constant-velocity (77,6 %), nên trông như hỏng ngang constant-velocity. Trong thế giới có xe sau phản ứng, IDM chỉ đâm **2,2 %** (7,4 % khi trễ 0,6 s) và là bộ tốt nhất. PercMLP đâm 16,9 % và 31,0 %, đều là va chạm phía trước.

**Kết luận:** va chạm phía sau trong thế giới phát lại che mất hành vi phía trước, và còn đổ lỗi nhầm cho IDM vì một giả định của mô phỏng. Khi so sánh bộ điều khiển, phải dùng thế giới có xe sau phản ứng. Kết luận lấy từ thế giới phát lại có thể ngược hẳn.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

- **Quy tắc nào chặn PercMLP?** Năm quy tắc:
  - **R2** (va chạm trên log): đâm 2,1 % ở seed xấu nhất, cận trên 95 % là 2,7 %. IDM là 0,07 %.
  - **R3** (ca xe dẫn phanh): 10,2 %, trong khi IDM là 1,2 %.
  - **R4** (lưới cắt vào): 23,5 % số ô tránh được nhưng vẫn đâm, IDM là 0,0 %.
  - **R5** (ổn định giữa các seed): dải 0,7–2,1 %, vượt giới hạn 1 điểm.
  - **R6** (xe sau phản ứng): 31,0 % lưới, IDM là 7,4 %.
- **Nguyên nhân gốc có hai loại:**
  - (a) **Log không chứa kịch bản đó** (R4, R6): cắt vào ở tốc độ cao không có trong dữ liệu, nên PercMLP đâm ở những chỗ IDM tránh được.
  - (b) **Sai số tích lũy khi tự lái** (R2, R3, R5): L2 vòng hở của PercMLP tốt nhất (0,621 m), nhưng khi tự lái các sai số nhỏ cộng dồn thành va chạm, và kết quả đổi theo seed.
- **Sửa bằng gì, và vì sao đúng nguyên nhân?**
  - Với (a): thêm 20.000 kịch bản cắt vào mô phỏng, có nhãn do luật IDM gán, để mô hình học các tình huống log không có.
  - Với (b): DAgger 2 vòng. Cho mô hình tự lái trên log huấn luyện, ghi lại các trạng thái nó tự đi vào, gắn nhãn IDM cho đúng các trạng thái đó, rồi huấn luyện lại. Nhờ vậy mô hình học cách quay về đúng hướng khi đã lệch, thay vì chỉ bắt chước người lái.
  - Mỗi cách sửa nhắm trực tiếp vào một nguyên nhân ở trên.
- **Kết quả sau sửa:** R4 từ 23,5 % xuống 0,2 %; R2 từ 2,1 % xuống 0,2 % (cận trên 0,5 %); R3 từ 10,2 % xuống 2,4 %; R5 từ dải 0,7–2,1 % xuống 0,2 %; R6 từ 31,0 % xuống 7,6 % (ngưỡng 9,4 %). PercMLP+ qua **7/7** quy tắc.
- **PercMLP+ có tốt hơn IDM không?** **Không.** Cổng chỉ kiểm tra "không kém hơn", và PercMLP+ chỉ nằm trong biên cho phép. Cụ thể: trên log, nó đâm 7 trên 2.883 quỹ đạo (seed xấu nhất), IDM đâm 2; trên lưới cắt vào, 0,2 % so với 0,0 %; ở 20 m/s, 1,2–6,2 % so với 0,0 %; L2 vòng hở 0,800 m, tệ hơn PercMLP (0,621 m).
- **Có nên phát hành không, và nên làm gì trước?** Không nên thay IDM bằng PercMLP+ ngay. Trước khi quyết định, cần: (i) kiểm thử với nhiều split và nhiều seed hơn; (ii) thử ở 16–20 m/s và các ca phanh mạnh hơn 3,6 m/s², vì 8,6 % lưới chưa được kiểm chứng; (iii) chạy PercMLP+ song song với IDM ở chế độ quan sát trên dữ liệu thật.

**Kết luận:** sửa đúng nguyên nhân đã đưa PercMLP+ qua cổng, nhưng qua cổng không đồng nghĩa với tốt hơn. Cổng "không kém hơn" chỉ ngăn mô hình tệ đi, không chứng minh mô hình hay hơn. PercMLP trước khi sửa có L2 tốt nhất nhưng khi tự lái lại đâm nhiều hơn IDM. Đây là lý do không nên chọn mô hình chỉ bằng L2.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

- **Chỉ có một chiều dọc, không chuyển làn.** Mô phỏng chỉ có khoảng cách và tốc độ tiến gần, nên không đánh giá được xe chen từ bên hông, né chướng ngại hay đường cong. Vì vậy "không đâm xe phía trước trong một chiều" chưa đủ để kết luận an toàn trong giao thông thật. Đây là giới hạn lớn nhất đối với quyết định phát hành.
- **Xe dẫn trong log không phản ứng với xe thử.** Quỹ đạo xe dẫn được phát lại nguyên văn. Mục về xe sau cho thấy kết luận có thể đổi khi xe sau phản ứng. Tương tự, nếu xe ta lái khác đi thì xe dẫn không đổi theo, nên số va chạm vòng kín có thể lệch so với thực tế.
- **Log thiếu phanh mạnh.** Người lái phanh mạnh nhất khoảng 3,6 m/s², trong khi 8,6 % lưới cần mạnh hơn. Chỉ có 166 quỹ đạo test có xe dẫn phanh, nên số liệu trên nhóm này dao động mạnh; đó là lý do R3 phải dùng biên 5 điểm. Quyết định ở vùng thiếu dữ liệu là ngoại suy, không phải đo đạc.
- **Mô phỏng không có độ trễ và nhiễu thật.** Vòng kín chỉ có nhiễu Gauss (khoảng cách ±1 m, tốc độ ±0,5 m/s), không có khung trễ hay rớt như phát lại thời gian thật (581 khung trễ, 571 khung rớt). Kết quả vòng kín vì vậy có thể lạc quan hơn triển khai thực tế.
- **Ít dữ liệu về môi trường và số lần chạy.** Bài này chỉ có một lần chia dữ liệu (split 7379), một tốc độ cruise (12 m/s) và 3 seed. Mô hình học dùng phép tính số thực, nên có thể cho kết quả hơi khác khi chạy trên máy hoặc thư viện khác; vì vậy phải ghi rõ môi trường chạy và chạy lại khi cần kiểm tra. Split, tốc độ hoặc số seed khác có thể cho con số khác.

**Để thu hẹp khoảng cách đến thế giới thật:** chạy vòng kín với nhiều split, nhiều seed và nhiều tốc độ; thu thập hoặc tổng hợp thêm ca phanh mạnh và cắt vào từ dữ liệu thật; thêm mô hình cảm biến và độ trễ; thử trên xe thật ở chế độ quan sát (shadow mode) trước khi cho điều khiển.

**Kết luận:** bài lab chứng minh được *phương pháp* kiểm thử (đo đúng, tìm đúng vùng lỗi, không tin một con số đơn lẻ), nhưng không chứng minh được một hệ thống là an toàn. Với PercMLP+, bằng chứng hiện tại đủ để nói "không tệ hơn IDM trong các điều kiện đã thử", nhưng chưa đủ để thay IDM.
