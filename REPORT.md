# REPORT — Lab Day 12

| | |
|---|---|
| Họ và tên | Nguyễn Hoàng Duy |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | 2A202602751 |
| GitHub | HoangDiine |
| Seed chia dữ liệu / seed mô hình | 3897 / 5421, 5422, 5423 |
| Số seed huấn luyện bạn dùng (`--seeds`) | 3 |

## Kết quả cuối

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---:|---:|---|
| constant-velocity | 1.255 | 19.8 | BLOCK |
| IDM | 1.533 | 0.0 | baseline |
| EgoMLP | 0.757 | 9.7–34.2 | BLOCK |
| PercMLP | 0.614 | 0.5–0.5 | BLOCK |
| PercMLP+ (sau khi sửa) | 0.815 | 0.1–0.1 | RELEASE |

## Câu 1 · Dữ liệu
Bản kiểm toán có 992.679 dòng, 9.993 quỹ đạo thô; loại 383 quỹ đạo trùng, còn 9.610.
`Acc_LV` chỉ tương quan 0,045 với đạo hàm `Speed_LV`, trong khi `Acc_FV` đạt 0,929; gia tốc xe dẫn được suy ra từ tốc độ đã làm mượt.
Dữ liệu lấy mẫu 10 Hz, mỗi quỹ đạo dài 70–109 bước (trung vị 106); chia theo quỹ đạo tránh để khung hình trùng/na ná lọt cả train và test.
Pipeline tìm được 614 ca phanh dẫn (6,4%), nên các kết quả ở lát cắt phanh cần được đọc cùng độ phủ dữ liệu.

## Câu 2 · Phát lại
Phát lại cùng mã khác 0/600 đoạn; bản float32 khác bit ở 600/600 nhưng không đoạn nào vượt dung sai, sai số cực đại 2,3×10⁻⁵.
AEB làm đổi 11/600 đoạn; ở đoạn 136, frame 34–38 có TTC 1,48–1,42 s, khe hở giảm 4,46 xuống 3,45 m và lệnh phanh đổi khoảng −5,33 thành −6,00 m/s².
Các frame đó chạm ngưỡng AEB 1,5 s khi xe dẫn đang tiến gần, nên thay đổi xuất hiện đúng lúc bộ điều khiển phải phanh mạnh hơn.
Phát nhanh nhất không có deadline (0 trễ/0 rớt); phát thời gian thực ghi nhận 582 frame trễ, 572 frame rớt trong 59.612 frame và 374/600 đoạn thay đổi.

## Câu 3 · Kịch bản
Lưới ở tốc độ cruise 12 m/s gồm 60 khoảng cách × 45 tốc độ tiến gần = 2.700 ô; gia tốc phanh cơ cấu chấp hành tối đa là 9 m/s².
Giới hạn phanh người lái ở phân vị 99,9 là 3,6 m/s²; 9% ô cần phanh mạnh hơn mức này, trong khi log chỉ có 614 ca phanh trên 9.610 quỹ đạo.
Với độ trễ 0 s, vùng va chạm bất khả thi chiếm 2,2%; với 0,6 s, con số là 7,4%, tăng 5,2 điểm phần trăm.
Mô phỏng IDM tăng cùng mức 2,2%→7,4%; kiểm chứng biên vật lý có 0 ô sai lệch ngoài frontier, nhưng các tình huống cắt vào vẫn là dữ liệu tổng hợp.

## Câu 4 · Vòng hở và vòng kín
L2 lần lượt là constant-velocity 1,255 m, IDM 1,533 m, EgoMLP 0,757 m, PercMLP 0,614 m; va chạm tương ứng là 19,8%, 0%, 9,7–34,2%, 0,5%.
Thứ hạng đổi ở bốn cặp: constant-velocity/IDM, constant-velocity/EgoMLP, IDM/EgoMLP và IDM/PercMLP; L2 tốt nhất không đồng nghĩa lái an toàn nhất.
IDM có L2 kém PercMLP nhưng dùng khoảng cách và tốc độ xe dẫn để phản ứng, nên va chạm log là 0% so với 0,5% của PercMLP.
EgoMLP đạt L2 0,757 m tốt hơn constant-velocity 1,255 m nhưng va chạm trung bình 24,2% (9,7–34,2% theo seed), vì nó không quan sát xe dẫn để dự đoán phanh.

## Câu 5 · Một lần chạy chưa đủ
Ba seed EgoMLP cho tỷ lệ va chạm 28,5% (5421), 34,2% (5422) và 9,7% (5423), tức biên độ 24,5 điểm phần trăm.
Khoảng Wilson 95% của seed tốt nhất 5423 là 8,7–10,8%; seed xấu nhất 5422 là 32,5–36,0%, hai khoảng không chồng lấp.
Nếu chỉ chạy seed 5423, EgoMLP có thể trông như mức 9,7%; chỉ chạy seed 5422 sẽ cho 34,2% và bị chặn ở R2.
Trung bình ba seed là 24,2%, vì vậy cần báo cả dải seed và độ bất định, không chọn một lần chạy thuận lợi.

## Câu 6 · Tác nhân phản ứng
Headway trung vị của người lái là 1,86 s; IDM phía sau khớp nhất ở T=1,4 s (W1=0,35), tốt hơn xe không phản ứng (W1=0,87).
Khi xe sau phát lại, IDM có 73,0% va chạm phía sau trên lưới ở độ trễ 0 s; khi xe sau phản ứng, tỷ lệ phía sau là 0%.
Ở độ trễ 0,6 s, va chạm phía sau IDM giảm từ 68,3% trong thế giới replay xuống 0% trong thế giới phản ứng; tổng ô có va chạm giảm 75,7% xuống 7,4%.
Replay giả định xe sau tiếp tục quỹ đạo cũ dù xe thử đã phanh; mô hình phản ứng sửa giả định đó, nên kết luận an toàn phải dựa trên cả hai thế giới.

## Câu 7 · Cổng phát hành và sửa
PercMLP ban đầu qua R2/R3 nhưng trượt R4 với 19,22% vùng cắt vào quy được cho bộ điều khiển (giới hạn 2%) và R6 với 26,7% so với baseline 7,4%.
Bản sửa bổ sung 20.000 kịch bản cắt vào và 2 vòng DAgger trên trạng thái chính planner tự đi vào; việc này nhắm vào thiếu phủ kịch bản và lỗi tích lũy khi tự lái.
PercMLP+ giảm vùng R4 còn 0,15%, R6 còn 7,6%, va chạm log 0,1% và qua cả 7 quy tắc; L2 lại tăng từ 0,614 lên 0,815 m.
Nó chưa chứng minh tốt hơn IDM: IDM vẫn có 0% va chạm log và là thầy tạo nhãn; chỉ nên xem đây là ứng viên qua gate mô phỏng, cần kiểm tra độc lập trước khi thay IDM.

## Câu 8 · Bài lab này không chứng minh được gì
Dữ liệu chỉ có 9.610 quỹ đạo làm sạch ở 10 Hz và 614 ca phanh dẫn; các tình huống hiếm hoặc khác miền có thể chưa được đại diện.
Lưới cắt vào gồm 20.000 tình huống tổng hợp; huấn luyện thấy tốc độ đến 16 m/s còn kiểm tra mở rộng tới 20 m/s, nên vẫn không bao quát mọi hành vi ngoài đường.
Mô phỏng dùng giới hạn phanh 9 m/s², IDM và mô hình xe sau giản lược; nhiễu cảm biến ±1 m và ±0,5 m/s không thay thế được lỗi/độ trễ cảm biến thật.
Ba seed mô hình và một simulator chưa đủ để kết luận phát hành; bước tiếp theo là thêm seed/kịch bản, kiểm tra hardware-in-the-loop, chạy thử đường kín và đánh giá với cảm biến, cơ cấu chấp hành thật.
