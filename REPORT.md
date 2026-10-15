# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | `Lâm Quang Anh Quân` |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `202602467` |
| GitHub | `awun0105` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `7379` / `7992` (các seed mô hình 7992–7996) |
| Số seed huấn luyện bạn dùng (`--seeds`) | `5` (mặc định) |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | | | |
| IDM | | | (baseline) |
| EgoMLP | | | |
| PercMLP | | | |
| PercMLP+ (sau khi sửa) | | | |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

- Ba vấn đề chính: (1) cột `Acc_LV` không khớp đạo hàm của `Speed_LV`, tương quan chỉ **0,045**, trong khi cột `Acc_FAV` của xe theo sau đạt **0,929**; (2) **383** quỹ đạo trùng hệt nhau (trên 9.993, tức 3,8 %), nằm ở các ID cách xa nhau, nên chia theo ID sẽ làm lọt dữ liệu từ train sang test; (3) quỹ đạo dài 70–109 bước (10 Hz) và các khung liền kề gần như giống nhau.
- Không dùng `Acc_LV` vì nó không đo gia tốc xe dẫn. Thay bằng gia tốc tính từ `Speed_LV`: làm mượt trung bình trượt 5 điểm rồi lấy sai phân trung tâm (`derive_lead`). Ngưỡng "xe dẫn đang phanh" −2 m/s² áp lên đại lượng này.
- Loại trùng: **383** quỹ đạo, còn **9.610** quỹ đạo sạch.
- Chia theo quỹ đạo (seed 7379, 30 % test): **6.727** quỹ đạo train và **2.883** quỹ đạo test. Chia theo khung sẽ đưa các khung gần như trùng vào cả hai tập và làm kết quả test quá lạc quan.
- Kiểm tra thêm: `Spatial_Gap` = Pos_LV − Pos_FAV − 4,5 m, khớp đến 4·10⁻¹¹ m, nên khoảng cách trong log đáng tin.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

- Chạy lại cùng mã: **0/600** đoạn khác nhau, nên công cụ phát lại tái lập được.
- Bản refactor float32 không giống từng bit ở **600/600** đoạn (sai lệch lớn nhất 4,0·10⁻⁴), nhưng **0/600** đoạn vượt dung sai (atol = rtol = 10⁻³). Dung sai hợp lý vì cộng số thực không kết hợp: (0,1+0,2)+0,3 = 0,6000000000000001 còn 0,1+(0,2+0,3) = 0,6. Sai lệch 10⁻⁴ là nhiễu số học; 10⁻³ đủ hẹp để vẫn bắt được thay đổi thật.
- AEB (phanh khẩn cấp khi TTC < 1,5 s) làm lệch **8/600** đoạn (chỉ số 64, 78, 195, 243, 247 và 3 đoạn khác), không phải tất cả.
- Mở đoạn 64: lệch ở 3 khung (75–77), TTC 1,48 → 1,44 s; lệnh IDM khoảng −4,4 m/s² bị thay bằng −6,0 m/s². Đoạn 78 lệch ở 1 khung (TTC 1,47 s). Lệch đúng ở các khung này vì đó là lúc TTC tụt xuống dưới 1,5 s ở cuối đoạn, khi khoảng cách còn 4,4–5,0 m.
- Phát nhanh nhất: **0** khung trễ và **0** khung rớt, vì không có hạn chót (deadline). Phát theo thời gian thật: **581** khung trễ và **571** khung rớt trên 59.551 khung, làm **378/600** đoạn đổi kết quả. Đó là lỗi trễ và rớt khung mà phát nhanh nhất che đi.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

- Lưới: 60 khoảng cách (2,8–59,3 m) × 45 tốc độ tiến gần (0–12 m/s, cruise 12 m/s) = **2.700** ô.
- Ô **không ai tránh được** (biên bất khả thi: phanh hết cỡ sau thời gian trễ): **2,2 %** khi t_react = 0 và **7,4 %** khi t_react = 0,6 s. Bộ mô phỏng khớp biên này với 0 ô lệch ở cả hai độ trễ.
- Độ trễ 0 → 0,6 s làm vùng đâm của IDM tăng từ **2,2 %** lên **7,4 %** lưới (+5,2 điểm). Vùng "tránh được nhưng vẫn đâm" của IDM là **0,0 %** ở cả hai độ trễ, nên mọi lần IDM đâm đều thuộc vùng bất khả thi.
- Người lái trong log phanh mạnh nhất khoảng **3,6 m/s²** (phân vị 99,9), trong khi bộ phanh làm được 9 m/s². **8,6 %** lưới đòi hỏi phanh mạnh hơn mức đó.
- Log phủ vùng khó một phần: trong 614 sự kiện phanh, chỉ 33 có tốc độ tiến gần trên 4 m/s và 7 trên 6 m/s; **không có sự kiện nào** ở gap dưới 10 m với tốc độ tiến gần trên 4 m/s, tức đúng vùng khó nhất của lưới.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

- Hạng ở vòng hở (L2 thấp hơn là tốt hơn): PercMLP 1 (0,619 m), EgoMLP 2 (0,760 m), const-vel 3 (1,254 m), IDM 4 (1,500 m).
- Hạng ở vòng kín (va chạm thấp hơn là tốt hơn): IDM 1 (0,07 %), PercMLP 2 (1,4 %, trung bình 5 seed), const-vel 3 (19,6 %), EgoMLP 4 (22,1 %, trung bình 5 seed).
- Bốn cặp đổi thứ tự: (const-vel, IDM), (const-vel, EgoMLP), (IDM, EgoMLP), (IDM, PercMLP). Đáng chú ý nhất: IDM có L2 xấu nhất nhưng an toàn nhất, còn EgoMLP có L2 tốt hơn const-vel nhưng đâm nhiều hơn.
- IDM có L2 lớn vì nó không bắt chước người lái: T = 1,2 s và s0 = 2 m, trong khi headway trung vị của người lái trong log là 1,87 s, nên quỹ đạo luôn lệch. Trong vòng kín nó thấy khoảng cách thật và phanh theo ngay, nên chỉ đâm 2 trên 2.883 quỹ đạo, cả hai đều có xe dẫn phanh.
- EgoMLP chỉ nhìn tốc độ quá khứ của chính xe ta. Nó học được xu hướng tăng/giảm tốc chung nên L2 tốt hơn const-vel, kể cả trên lát cắt xe dẫn phanh (1,096 m so với 1,903 m). Nhưng nó không thấy khoảng cách tới xe dẫn nên không phanh kịp: trên 166 quỹ đạo có xe dẫn phanh, nó đâm 44–57 %, gần bằng const-vel (53,6 %).

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

- EgoMLP qua 5 seed đâm **15,8 %** đến **35,7 %** (seed 7992: 20,2 %; 7993: 15,8 %; 7994: 35,7 %; 7995: 23,1 %; 7996: 15,9 %), dải gần 20 điểm phần trăm.
- Khoảng Wilson 95 % của seed tốt nhất (7993: [14,5; 17,2] %) và seed xấu nhất (7994: [33,9; 37,4] %) **không chồng nhau**. Chênh lệch giữa các seed là thật, không phải nhiễu của việc lấy mẫu 2.883 quỹ đạo.
- Wilson chỉ nói về sai số lấy mẫu trong một tập test; biến thiên giữa các lần huấn luyện chỉ thấy khi nhìn dải qua seed. PercMLP cũng dao động 0,5–2,8 %, và seed tệ nhất (2,84 %, CI [2,3; 3,5] %) làm nó rớt ngưỡng ổn định R5 (dải phải ≤ 1 điểm).
- Nếu chỉ chạy seed 7993, bạn sẽ báo "EgoMLP ≈ 16 %, tốt hơn const-vel (19,6 %)". Nếu chạy seed 7994, bạn sẽ báo "35,7 %, gần gấp đôi const-vel". Hai kết luận trái nhau chỉ vì chọn seed; và 3 trên 5 seed của EgoMLP còn tệ hơn const-vel.
- Vì vậy mỗi kết quả phải kèm dải qua seed, và cổng dùng seed tệ nhất (R2), không dùng seed đẹp nhất.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

| Bộ điều khiển | Xe sau phát lại: đâm cả lưới (t = 0 / 0,6 s) | Xe sau phản ứng: đâm cả lưới (t = 0 / 0,6 s) | Đâm từ phía sau khi phát lại |
|---|---|---|---|
| IDM | 75,3 % / 75,7 % | 2,2 % / 7,4 % | 73,0 % / 68,3 % |
| PercMLP | 73,7 % / 73,0 % | 19,5 % / 31,4 % | 60,5 % / 41,9 % |
| const-vel | 77,6 % / 77,6 % | 77,6 % / 77,6 % | 0 % |
| EgoMLP | 76,5 % / 76,9 % | 76,5 % / 76,9 % | 0 % |

- T hiệu chỉnh là **1,4 s** (W1 = 0,36; T = 1,2 s cho W1 gần như bằng). Xe sau không bao giờ phản ứng có W1 = 0,86, nên xe sau đã hiệu chỉnh gần phân bố headway của người lái trong log hơn khoảng 2,4 lần (headway trung vị của người lái là 1,87 s).
- Xe sau phát lại giữ tốc độ đã ghi và không biết xe ta vừa phanh. Khi IDM phanh gấp để tránh xe dẫn, khoảng cách với xe sau co lại và xe sau đâm vào đuôi xe ta. Con số **73,0 %** của IDM là va chạm giả: người lái thật phía sau sẽ phanh, nên lỗi không nằm ở IDM.
- Kết luận đổi: trong thế giới phát lại, IDM đâm **75,3 %** lưới, gần bằng const-vel (77,6 %), tức trông hỏng ngang const-vel. Trong thế giới phản ứng, IDM chỉ đâm **2,2 %** (7,4 % ở 0,6 s) và là bộ tốt nhất; PercMLP đâm 19,5 % và 31,4 % do chính va chạm phía trước.
- Vì vậy phải so sánh các bộ điều khiển trong thế giới có xe sau phản ứng; trong thế giới phát lại, va chạm phía sau che mất hành vi phía trước.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

(trả lời ở đây)

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

- **Một chiều dọc, không chuyển làn.** Mô phỏng chỉ có khoảng cách × tốc độ tiến gần, nên không đánh giá được xe chen từ bên hông, né chướng ngại hay đường cong. Với quyết định phát hành, "không đâm xe trước trong một chiều" chưa đủ để kết luận an toàn toàn cảnh.
- **Xe dẫn trong log không phản ứng với xe thử.** Đường xe dẫn được phát lại nguyên văn. Mục 6 cho thấy kết luận đổi khi xe sau phản ứng; tương tự, nếu xe ta lái khác đi thì xe dẫn không đổi theo, nên số va chạm vòng kín trên log có thể lệch so với thế giới thật.
- **Log hiếm phanh mạnh.** Người lái phanh tối đa khoảng 3,6 m/s² và 8,6 % lưới đòi hỏi mạnh hơn. Chỉ có 166 quỹ đạo trong tập test có xe dẫn phanh, nên số trên lát cắt phanh rất nhiễu (R3 phải dùng biên 5 điểm). Quyết định ở vùng này là ngoại suy, không phải đo.
- **Không có độ trễ và nhiễu thật.** Vòng kín chỉ có nhiễu Gauss (khoảng cách ±1 m, tốc độ ±0,5 m/s), không có khung trễ hay rớt như phát lại thời gian thật (581 khung trễ, 571 khung rớt). Bảng vòng kín vì vậy có thể lạc quan hơn triển khai thật.
- **Một lần chia dữ liệu, một tốc độ cruise.** Mọi số liệu dùng split 7379 và cruise 12 m/s của MSSV này. Split hoặc tốc độ khác có thể cho số khác.
- Để thu hẹp khoảng cách: chạy vòng kín với nhiều split và nhiều seed; thu thập hoặc tổng hợp thêm ca phanh mạnh và cắt vào từ dữ liệu thật; mô hình hóa cảm biến và độ trễ; thử trên xe thật ở chế độ quan sát (shadow mode) trước khi phát hành.
