# Kịch bản thuyết trình Đề án tốt nghiệp (15 phút)

**Đề tài:** Nghiên cứu mô hình học tăng cường đa tác tử trong bài toán điều khiển tín hiệu giao thông tại Việt Nam.

---

## 1. Slide tiêu đề (0:00 - 0:35)
**Lời nói:**  
Kính thưa các thầy cô trong Hội đồng, em tên là Nguyễn Thanh Hùng. Hôm nay em xin trình bày đề án tốt nghiệp thạc sĩ với đề tài **“Nghiên cứu mô hình học tăng cường đa tác tử trong bài toán điều khiển tín hiệu giao thông tại Việt Nam”** dưới sự hướng dẫn của TS. Nguyễn Đình Hoá.

---

## 2. Nội dung trình bày (0:35 - 1:00)
**Lời nói:**  
Bài trình bày của em gồm bốn phần chính: bối cảnh nghiên cứu và mục tiêu đề án, mô hình bài toán và các thuật toán được triển khai, thiết lập thực nghiệm và quy trình đánh giá, cuối cùng là kết quả, thảo luận và kết luận.

---

## 3. Bối cảnh và khoảng trống nghiên cứu (1:00 - 2:00)
**Lời nói:**  
Điểm xuất phát của đề án là hạn chế của điều khiển đèn tín hiệu chu kỳ cố định khi lưu lượng giao thông biến động theo thời gian. Trong bối cảnh Việt Nam, giao thông còn có đặc trưng hỗn hợp với tỷ lệ xe máy cao và hành vi di chuyển linh hoạt. Các nút giao trong cùng mạng lưới cũng ảnh hưởng lẫn nhau, nên tối ưu riêng từng nút chưa đủ. Vì vậy, đề án tập trung đánh giá MARL trên kịch bản mô phỏng phản ánh giao thông hỗn hợp tại khu vực Thanh Xuân, Hà Nội.

---

## 4. Mục tiêu và đóng góp (2:00 - 2:50)
**Lời nói:**  
Đề án hướng đến bốn việc. Trước hết là xây dựng kịch bản Thanh Xuân UTM từ bản đồ và dữ liệu khảo sát. Tiếp theo, đề án đặc tả môi trường đa tác tử trong SUMO, gồm quan sát cục bộ, hành động có ràng buộc và hàm phần thưởng. Trên môi trường đó, em triển khai MAPPO, HAPPO, GAT-MAPPO và GAT-HAPPO, rồi đánh giá đa hạt giống so với điều khiển cố định và Max-Pressure. Đóng góp nằm ở quy trình thực nghiệm này, không phải ở việc đề xuất thuật toán mới.

---

## 5. Kịch bản và dữ liệu thực nghiệm (2:50 - 3:50)
**Lời nói:**  
Kịch bản thực nghiệm được xây dựng cho khu vực Thanh Xuân, Hà Nội. Mạng lưới đường được trích xuất từ OpenStreetMap và chuyển sang SUMO. Nhu cầu giao thông được xây dựng từ dữ liệu khảo sát UTM-Hanoi và điều chỉnh về mức 5.000 phương tiện mỗi giờ. Dữ liệu này là cơ sở xây dựng nhu cầu cho mô phỏng. Mô hình chuyển làn phụ được bật cho xe máy để phản ánh tốt hơn đặc trưng giao thông hỗn hợp. Môi trường đưa 41 bộ điều khiển đèn tín hiệu vào quá trình huấn luyện.

---

## 6. Mô hình bài toán (3:50 - 4:50)
**Lời nói:**  
Bài toán được mô hình hóa dưới dạng trò chơi Markov hợp tác. Sau mỗi 5 giây mô phỏng, môi trường đọc trạng thái giao thông từ SUMO và tạo vector quan sát cơ sở 8 chiều cho từng tác tử, gồm thông tin hàng đợi, xe đang di chuyển, áp lực, thời gian chờ và trạng thái giai đoạn xanh. Với các biến thể GAT, môi trường bổ sung tỷ lệ số giai đoạn xanh, nên đầu vào có 9 chiều. Tác tử hoặc duy trì giai đoạn xanh hiện tại, hoặc yêu cầu chuyển sang giai đoạn xanh kế tiếp. Slide sau trình bày phần thưởng và ràng buộc khi thực thi hành động.

---

## 7. Hàm phần thưởng và ràng buộc hành động (4:50 - 5:40)
**Lời nói:**  
Hàm phần thưởng không tối ưu một chỉ số đơn lẻ mà phạt các trạng thái giao thông bất lợi, gồm áp lực cục bộ, hàng đợi dừng, thời gian chờ tăng thêm và dịch chuyển cưỡng bức. Về hành động, tác tử luôn có thể duy trì giai đoạn xanh hiện tại. Khi yêu cầu chuyển, môi trường kiểm tra thứ tự giai đoạn xanh và thời gian xanh tối thiểu trước khi thực thi. Nhờ đó chính sách không tạo lệnh điều khiển vi phạm chu kỳ đèn.

---

## 8. Các phương pháp được so sánh (5:40 - 6:40)
**Lời nói:**  
Đề án so sánh sáu phương pháp. Hai phương pháp đối chứng là điều khiển cố định và Max-Pressure, trong đó Max-Pressure là một đối chứng thích ứng mạnh. Bốn phương pháp MARL là MAPPO, HAPPO, GAT-MAPPO và GAT-HAPPO. Các phương pháp này khác nhau ở hai điểm chính: actor dùng chung hay actor riêng, và có hay không sử dụng bộ mã hóa đồ thị GAT.

---

## 9. Quy trình huấn luyện và đánh giá (6:40 - 7:40)
**Lời nói:**  
Việc huấn luyện dùng kịch bản Thanh Xuân UTM ở mức nhu cầu 5.000 phương tiện mỗi giờ. Mỗi lần mô phỏng kéo dài 3.600 giây; với bước quyết định 5 giây, mỗi lần có 720 bước. Mỗi mô hình được huấn luyện trong 500 lần, tương ứng 360.000 bước quyết định. Sau đó, bản lưu mô hình cuối cùng được đánh giá trên 10 hạt giống ngẫu nhiên.

---

## 10. Quy trình huấn luyện (7:40 - 8:30)
**Lời nói:**  
Trong mỗi bước, SUMO trả về trạng thái cho 41 tác tử. Môi trường tạo quan sát và bộ lọc hành động theo từng tác tử; chính sách chọn hành động, sau đó SUMO thực thi 5 giây, tính phần thưởng và lưu dữ liệu vào bộ đệm. Khi bộ đệm đủ 128 bước hoặc lần mô phỏng kết thúc, hệ thống tính GAE, cập nhật PPO rồi xóa bộ đệm. Nếu lần mô phỏng chưa kết thúc, luồng quay lại bước quyết định; nếu đã kết thúc và chưa đủ số lần huấn luyện đặt trước, hệ thống khởi tạo lần mô phỏng tiếp theo.

---

## 11. Chỉ số đánh giá (8:30 - 9:15)
**Lời nói:**  
Đề án dùng bốn chỉ số. Số phương tiện hoàn thành hành trình phản ánh năng lực giải tỏa của mạng. Thời gian chờ trung bình phản ánh chất lượng di chuyển, nên thấp hơn là tốt hơn; tốc độ trung bình phản ánh mức độ thông thoáng, nên cao hơn thường là tốt hơn. Dịch chuyển cưỡng bức được xem là chỉ số hỗ trợ theo dõi bế tắc nghiêm trọng trong mô phỏng, vì nó còn chịu ảnh hưởng của hình học mạng đường, cấu hình mô phỏng và dao động giữa các hạt giống.

---

## 12. Kết quả đánh giá (9:15 - 10:30)
**Lời nói:**  
Bảng này tóm tắt kết quả trung bình và độ lệch chuẩn trên 10 hạt giống đánh giá. MAPPO đạt thời gian chờ trung bình thấp nhất và tốc độ trung bình cao nhất. GAT-MAPPO đạt số phương tiện hoàn thành hành trình cao nhất, nhưng mức chênh lệch so với MAPPO nhỏ. Số sự kiện dịch chuyển cưỡng bức không cho thấy xu hướng cải thiện rõ ràng dưới các chính sách MARL.

---

## 13. Thảo luận kết quả (10:30 - 11:30)
**Lời nói:**  
MAPPO đạt thời gian chờ trung bình thấp nhất và tốc độ trung bình cao nhất, còn GAT-MAPPO có số phương tiện hoàn thành hành trình cao nhất nhưng chỉ nhỉnh hơn MAPPO 2,8 xe. Max-Pressure là đối chứng thích ứng mạnh và tốt hơn điều khiển cố định ở ba chỉ số vận hành chính. HAPPO gần như ngang với Max-Pressure, còn GAT-HAPPO không vượt MAPPO hoặc GAT-MAPPO ở các chỉ số chính. Trong đề án, dịch chuyển cưỡng bức được xem là chỉ số hỗ trợ.

---

## 14. Hạn chế và hướng phát triển (11:30 - 12:30)
**Lời nói:**  
Đề án hiện mới được kiểm chứng trong môi trường mô phỏng SUMO, chưa có kiểm chứng ngoài thực địa. Kết quả còn phụ thuộc vào chất lượng mạng đường, dữ liệu khảo sát và các giả định mô phỏng. Thực nghiệm mới tập trung vào một mức nhu cầu chính là 5.000 phương tiện mỗi giờ, trong khi dịch chuyển cưỡng bức còn dao động và chưa được cải thiện rõ ràng. Hướng tiếp theo là mở rộng mức nhu cầu, tăng số hạt giống và cải thiện dữ liệu đầu vào.

---

## 15. Kết luận (12:30 - 13:30)
**Lời nói:**  
Tóm lại, đề án đã xây dựng quy trình mô phỏng, huấn luyện và đánh giá điều khiển tín hiệu giao thông đa tác tử. Kịch bản Thanh Xuân UTM cho phép đánh giá các phương pháp trong bối cảnh giao thông hỗn hợp tại Hà Nội. Theo ba chỉ số vận hành chính, MAPPO và GAT-MAPPO nổi bật nhất trong nhóm phương pháp được đánh giá. Tuy vậy, không có phương pháp nào vượt trội trên toàn bộ chỉ số, và GAT không luôn cải thiện so với biến thể không dùng GAT.
