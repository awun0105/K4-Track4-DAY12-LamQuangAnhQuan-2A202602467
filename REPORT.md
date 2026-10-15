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

(trả lời ở đây)

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

(trả lời ở đây)

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

(trả lời ở đây)

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

(trả lời ở đây)

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

(trả lời ở đây)

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

(trả lời ở đây)
