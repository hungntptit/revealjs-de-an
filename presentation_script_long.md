# Kịch bản thuyết trình mở rộng (khoảng 14-15 phút)

**Đề tài:** Nghiên cứu mô hình học tăng cường đa tác tử trong bài toán điều khiển tín hiệu giao thông tại Việt Nam.

> Bản này dùng để tập nói đầy đủ. Khi thời gian bị rút ngắn, ưu tiên bản `presentation_script.md`.

---

## 1. Slide tiêu đề (0:00 - 0:35)

Kính thưa các thầy cô trong Hội đồng, em tên là Nguyễn Thanh Hùng. Hôm nay em xin trình bày đề án tốt nghiệp thạc sĩ với đề tài **“Nghiên cứu mô hình học tăng cường đa tác tử trong bài toán điều khiển tín hiệu giao thông tại Việt Nam”**, dưới sự hướng dẫn của TS. Nguyễn Đình Hoá.

Đề án tập trung vào việc xây dựng một quy trình mô phỏng, huấn luyện và đánh giá các phương pháp điều khiển tín hiệu giao thông đa tác tử trong bối cảnh giao thông hỗn hợp.

---

## 2. Nội dung trình bày (0:35 - 1:00)

Bài trình bày của em gồm bốn phần. Đầu tiên là bối cảnh nghiên cứu và mục tiêu đề án. Tiếp theo là mô hình bài toán và các phương pháp được so sánh. Phần thứ ba trình bày kịch bản, quy trình huấn luyện và đánh giá. Cuối cùng là kết quả, nhận xét, hạn chế và kết luận.

---

## 3. Bối cảnh và khoảng trống nghiên cứu (1:00 - 2:00)

Điểm xuất phát của đề án là hạn chế của điều khiển đèn tín hiệu theo chu kỳ cố định. Cách điều khiển này được thiết lập trước, nên khó phản ứng khi lưu lượng thay đổi theo thời gian hoặc khi ùn tắc chỉ xuất hiện ở một số hướng đi nhất định.

Trong bối cảnh Việt Nam, giao thông còn có đặc trưng hỗn hợp, đặc biệt là tỷ lệ xe máy cao và hành vi di chuyển linh hoạt. Ngoài ra, các nút giao trong cùng mạng lưới có ảnh hưởng lẫn nhau. Nếu một nút giao bị ùn tắc, hàng xe có thể lan sang các nút lân cận. Vì vậy, nếu chỉ tối ưu riêng từng nút thì chưa đủ.

Từ đó, đề án tập trung đánh giá các thuật toán học tăng cường đa tác tử trên một kịch bản mô phỏng phản ánh đặc trưng giao thông hỗn hợp tại khu vực Thanh Xuân, Hà Nội.

---

## 4. Mục tiêu và đóng góp (2:00 - 2:55)

Trong đề án này, em tập trung vào bốn nội dung chính.

Thứ nhất, xây dựng kịch bản Thanh Xuân UTM từ dữ liệu bản đồ và dữ liệu khảo sát nhu cầu di chuyển. Thứ hai, xây dựng môi trường điều khiển đa tác tử trong SUMO, bao gồm quan sát cục bộ, hành động có ràng buộc và hàm phần thưởng. Thứ ba, triển khai bốn thuật toán MARL là MAPPO, HAPPO, GAT-MAPPO và GAT-HAPPO. Cuối cùng, các phương pháp được đánh giá với nhiều hạt giống cố định và so sánh với hai đối chứng là điều khiển cố định và Max-Pressure.

Đóng góp chính của đề án nằm ở quy trình thực nghiệm này: cùng một kịch bản, cùng giao thức đánh giá và các phương pháp được so sánh trên các chỉ số thống nhất. Đề án không đề xuất một thuật toán mới.

---

## 5. Kịch bản và dữ liệu thực nghiệm (2:55 - 4:05)

Kịch bản thực nghiệm được xây dựng cho khu vực Thanh Xuân, Hà Nội. Mạng lưới đường được trích xuất từ OpenStreetMap, sau đó chuyển đổi và hiệu chỉnh để sử dụng trong SUMO.

Nhu cầu giao thông được xây dựng từ dữ liệu khảo sát UTM-Hanoi và điều chỉnh về mức 5.000 phương tiện mỗi giờ. Thành phần phương tiện gồm xe máy, ô tô con và xe buýt. Để quy đổi mức chiếm dụng giao thông, xe máy được tính là 0,5 PCU, ô tô con là 1,0 PCU và xe buýt là 3,0 PCU.

Đề án cũng bật mô hình chuyển làn dạng sublane cho xe máy để mô phỏng tốt hơn đặc trưng dòng xe hỗn hợp. Trong toàn mạng lưới, 41 bộ điều khiển đèn tín hiệu được đưa vào quá trình huấn luyện như các tác tử điều khiển.

---

## 6. Mô hình bài toán (4:05 - 5:10)

Bài toán được mô hình hóa dưới dạng trò chơi Markov hợp tác nhiều tác tử. Mỗi tác tử tương ứng với một bộ điều khiển đèn tín hiệu và cùng hướng tới cải thiện trạng thái vận hành của mạng lưới.

Sau mỗi 5 giây mô phỏng, môi trường đọc trạng thái giao thông từ SUMO và tạo quan sát cho từng tác tử. Quan sát cơ sở có 8 thành phần: hàng đợi vào, số xe đang di chuyển vào, hàng đợi ra, áp lực cục bộ, mức tăng thời gian chờ, giai đoạn xanh hiện tại, thời gian đã qua trong giai đoạn và khả năng chuyển pha. Trong đó, hàng đợi vào là PCU của xe dừng chờ trên làn tiếp cận; số xe đang di chuyển vào là PCU của xe vẫn chạy trên các làn đó. Áp lực cục bộ so sánh tổng lưu lượng đi vào với tổng lưu lượng trên các làn thoát.

Với các biến thể dùng GAT, đầu vào được bổ sung một đặc trưng liên quan đến cấu hình pha đèn, nên mỗi nút có vector đặc trưng 9 chiều. GAT dùng toàn bộ các vector này cùng quan hệ lân cận giữa các nút để tổng hợp thông tin không gian.

Về hành động, tác tử có thể duy trì giai đoạn xanh hiện tại hoặc yêu cầu chuyển sang giai đoạn xanh kế tiếp. Phần tiếp theo giải thích cách môi trường giới hạn các yêu cầu chuyển pha này.

---

## 7. Hàm phần thưởng và ràng buộc hành động (5:10 - 6:00)

Hàm phần thưởng không tối ưu trực tiếp một chỉ số đơn lẻ. Thay vào đó, nó phạt các trạng thái giao thông bất lợi, gồm áp lực cục bộ cao, hàng đợi xe dừng, mức tăng thời gian chờ và các sự kiện dịch chuyển cưỡng bức.

Về hành động, tác tử luôn có thể duy trì giai đoạn xanh hiện tại. Khi tác tử yêu cầu chuyển, môi trường kiểm tra thứ tự giai đoạn xanh trong chu kỳ và điều kiện thời gian xanh tối thiểu trước khi thực thi. Sau đó, SUMO vẫn đi qua các pha chuyển tiếp cần thiết.

Cơ chế này bảo đảm chính sách không sinh ra các lệnh điều khiển vi phạm chu kỳ đèn, đồng thời không cho tác tử đổi pha quá sớm.

---

## 8. Các phương pháp được so sánh (6:00 - 7:00)

Đề án so sánh sáu phương pháp. Hai đối chứng là điều khiển cố định và Max-Pressure. Điều khiển cố định dùng chương trình đèn thiết lập trước. Max-Pressure là phương pháp thích ứng, lựa chọn pha dựa trên áp lực giao thông, và là một đối chứng mạnh.

Bốn phương pháp học tăng cường là MAPPO, HAPPO, GAT-MAPPO và GAT-HAPPO. Các phương pháp này khác nhau theo hai trục thể hiện trong ma trận trên slide.

Trục thứ nhất là actor dùng chung hay actor riêng. MAPPO dùng actor chung, còn HAPPO dùng actor riêng cho từng tác tử và cập nhật tuần tự. Trục thứ hai là có hay không có bộ mã hóa đồ thị GAT. GAT-MAPPO và GAT-HAPPO bổ sung GAT để khai thác thông tin từ các nút giao lân cận.

---

## 9. Quy trình huấn luyện và đánh giá (7:00 - 7:55)

Việc huấn luyện sử dụng kịch bản Thanh Xuân UTM với mức nhu cầu 5.000 phương tiện mỗi giờ. Một lần mô phỏng kéo dài 3.600 giây. Vì tác tử ra quyết định mỗi 5 giây, một lần mô phỏng có tối đa 720 bước quyết định.

Mỗi mô hình được huấn luyện trong 500 lần mô phỏng, tương ứng 360.000 bước quyết định. Sau huấn luyện, bản lưu mô hình cuối cùng được dùng để đánh giá trên 10 hạt giống cố định. Các phương pháp đối chứng cũng được chạy với cùng các hạt giống để bảo đảm điều kiện so sánh nhất quán.

---

## 10. Quy trình huấn luyện (7:55 - 9:00)

Slide này mô tả vòng lặp huấn luyện chi tiết hơn.

Mỗi lần mô phỏng bắt đầu bằng việc khởi tạo SUMO. Môi trường tạo quan sát cho 41 tác tử và lọc các hành động không hợp lệ. Từ quan sát đó, actor cho điểm các hành động hợp lệ và chọn hành động; critic đồng thời ước lượng giá trị kỳ vọng của trạng thái hiện tại.

SUMO thực thi các hành động trong 5 giây mô phỏng. Sau bước này, môi trường nhận trạng thái mới, tính phần thưởng cho từng tác tử và lưu dữ liệu tương tác vào rollout. Dữ liệu lưu gồm quan sát, hành động, log-xác suất, giá trị critic, phần thưởng và cờ kết thúc.

Khi rollout đã đủ dữ liệu cập nhật hoặc khi lần mô phỏng kết thúc, hệ thống tính lợi thế bằng GAE. PPO dùng lợi thế này để cập nhật chính sách có giới hạn thay đổi, đồng thời cập nhật mạng giá trị. Nếu lần mô phỏng chưa kết thúc, quy trình quay lại bước tạo quan sát. Nếu đã kết thúc, hệ thống bắt đầu lần mô phỏng tiếp theo cho đến khi hoàn thành số lần huấn luyện đã đặt.

---

## 11. Chỉ số đánh giá (9:00 - 9:45)

Đề án dùng bốn chỉ số đánh giá chính.

Số phương tiện hoàn thành hành trình phản ánh năng lực giải tỏa của mạng lưới. Thời gian chờ trung bình phản ánh độ trễ của phương tiện, nên thấp hơn là tốt hơn. Tốc độ trung bình phản ánh mức độ thông thoáng, nên cao hơn thường là tốt hơn.

Chỉ số cuối là số sự kiện dịch chuyển cưỡng bức. Trong đề án, đây là chỉ số hỗ trợ để theo dõi các tình huống bế tắc nghiêm trọng trong mô phỏng. Chỉ số này còn chịu ảnh hưởng của hình học mạng đường, cấu hình mô phỏng và dao động giữa các hạt giống, nên không được dùng một mình để kết luận phương pháp nào tốt hơn.

---

## 12. Kết quả đánh giá (9:45 - 11:05)

Bảng này tóm tắt giá trị trung bình và độ lệch chuẩn trên 10 hạt giống đánh giá.

MAPPO đạt thời gian chờ trung bình thấp nhất, 42,79 giây, đồng thời đạt tốc độ trung bình cao nhất, 13,19 mét trên giây. GAT-MAPPO có số phương tiện hoàn thành hành trình cao nhất, 4.583,3 xe.

Riêng số sự kiện dịch chuyển cưỡng bức không cho thấy xu hướng cải thiện rõ ràng khi dùng các chính sách MARL. Vì vậy, các kết quả cần được đọc theo từng chỉ số thay vì tìm một phương pháp thắng tuyệt đối trên tất cả tiêu chí.

---

## 13. Nhận xét và thảo luận (11:05 - 12:10)

Từ bảng kết quả, MAPPO và GAT-MAPPO là hai phương pháp nổi bật nhất, nhưng mỗi phương pháp mạnh ở một khía cạnh khác nhau. MAPPO tốt nhất về thời gian chờ và tốc độ trung bình. GAT-MAPPO cao nhất về số phương tiện hoàn thành hành trình.

Max-Pressure vẫn là một đối chứng thích ứng mạnh và tốt hơn điều khiển cố định ở ba chỉ số vận hành chính. HAPPO cho kết quả gần với Max-Pressure, còn GAT-HAPPO chưa vượt được MAPPO và GAT-MAPPO ở các chỉ số chính.

Vì vậy, kết quả cho thấy không phải cứ dùng actor riêng hoặc bổ sung GAT là kết quả sẽ cải thiện đồng đều. Hiệu quả còn phụ thuộc vào kịch bản, cấu trúc mạng, dữ liệu đầu vào và lựa chọn siêu tham số.

---

## 14. Hạn chế và hướng phát triển (12:10 - 13:10)

Kết quả của đề án hiện mới được kiểm chứng trong môi trường mô phỏng SUMO, chưa có kiểm chứng ngoài thực địa. Kết quả cũng phụ thuộc vào chất lượng mạng đường, dữ liệu khảo sát nhu cầu và các giả định khi xây dựng kịch bản.

Thực nghiệm mới tập trung vào một mức nhu cầu chính là 5.000 phương tiện mỗi giờ. Ngoài ra, số sự kiện dịch chuyển cưỡng bức còn dao động và chưa được cải thiện rõ ràng.

Các hướng tiếp theo gồm mở rộng đánh giá sang nhiều mức nhu cầu khác nhau, tăng số hạt giống, cải thiện dữ liệu đầu vào và tiếp tục kiểm tra độ ổn định của các chính sách trong các điều kiện giao thông khác nhau.

---

## 15. Kết luận (13:10 - 14:10)

Tóm lại, đề án đã xây dựng được quy trình mô phỏng, huấn luyện và đánh giá cho bài toán điều khiển tín hiệu giao thông đa tác tử. Kịch bản Thanh Xuân UTM cho phép đánh giá các phương pháp trong bối cảnh giao thông hỗn hợp tại Hà Nội.

Xét theo ba chỉ số vận hành chính, MAPPO và GAT-MAPPO là hai phương pháp nổi bật nhất trong nhóm được đánh giá. Tuy nhiên, không có phương pháp nào tốt nhất trên tất cả các chỉ số, và việc bổ sung GAT không phải lúc nào cũng cải thiện kết quả so với biến thể không dùng GAT.

Do đó, đóng góp thực nghiệm của đề án là cung cấp một quy trình có thể tiếp tục mở rộng để đánh giá các phương pháp MARL trên các kịch bản giao thông hỗn hợp.

---

## 16. Cảm ơn (14:10 - 14:20)

Trên đây là phần trình bày của em về đề án tốt nghiệp thạc sĩ. Em xin trân trọng cảm ơn các thầy cô đã lắng nghe và em mong nhận được các câu hỏi, góp ý từ Hội đồng.
