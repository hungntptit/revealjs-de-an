# Ghi chú ôn phản biện

Tài liệu này dùng để nhớ các khái niệm dễ bị hỏi khi bảo vệ. Câu trả lời nên ngắn, sau đó chỉ mở rộng khi Hội đồng hỏi tiếp.

## Luồng bài toán

**Một tác tử là gì?**  
Trong đề án, một tác tử tương ứng với một bộ điều khiển đèn tín hiệu được đưa vào quá trình huấn luyện. Có 41 tác tử điều khiển trong kịch bản.

**Một bước quyết định diễn ra như thế nào?**  
Sau mỗi 5 giây mô phỏng, SUMO cung cấp trạng thái giao thông. Môi trường tạo quan sát cho từng tác tử, lọc các hành động không hợp lệ, thực thi hành động và tính phần thưởng. Dữ liệu tương tác sau đó được dùng để cập nhật chính sách khi huấn luyện.

**Lần mô phỏng (episode) là gì?**  
Một lần mô phỏng kéo dài tối đa 3.600 giây. Với bước quyết định 5 giây, một lần mô phỏng có tối đa 720 bước quyết định.

**Khác nhau giữa trạng thái và quan sát?**  
Trạng thái là thông tin giao thông mà SUMO cung cấp cho môi trường. Quan sát là vector thông tin cục bộ mà môi trường trích xuất và đưa cho một tác tử để chọn hành động.

## Quan sát, hành động và phần thưởng

**Quan sát 8 chiều gồm gì?**  
Gồm hàng đợi đầu vào `q_in`, dòng xe đầu vào `m_in`, hàng đợi đầu ra `q_out`, áp lực `pressure`, mức tăng thời gian chờ `delta_wait`, pha hiện tại đã chuẩn hóa `phase_norm`, thời gian pha đã chuẩn hóa `elapsed_norm` và khả năng chuyển pha `can_switch`.

**Áp lực giao thông là gì?**  
Đây là đại lượng phản ánh mất cân bằng giữa nhu cầu đi vào và khả năng thoát ra ở khu vực nút giao. Áp lực cao cho thấy nguy cơ tích tụ phương tiện, nên được đưa vào quan sát và phần thưởng.

**Hành động của tác tử là gì?**  
Tác tử có hai lựa chọn ở mức quyết định: giữ pha hiện tại hoặc yêu cầu chuyển sang giai đoạn xanh kế tiếp.

**Bộ lọc hành động là gì và vì sao cần nó?**  
Đó là cơ chế loại các hành động không thỏa cấu hình chu kỳ đèn và ràng buộc an toàn. Tác tử chỉ được chuyển sang pha xanh kế tiếp khi đã qua thời gian xanh tối thiểu. Vì vậy, chính sách không thể phát lệnh chuyển pha tùy ý.

**Phần thưởng âm có nghĩa mô hình được huấn luyện kém không?**  
Không. Phần thưởng được thiết kế chủ yếu dưới dạng các khoản phạt cho trạng thái bất lợi. Giá trị âm phản ánh tổng mức phạt trong một bước; điều quan trọng là chính sách có giảm được mức phạt tương đối trong quá trình huấn luyện hay không.

**Phần thưởng gồm những gì?**  
Phần thưởng phạt áp lực cục bộ, hàng đợi dừng, phần thời gian chờ tăng thêm và dịch chuyển cưỡng bức. Mục đích là hướng chính sách tránh ùn tắc, thay vì tối ưu một chỉ số đơn lẻ.

**Dịch chuyển cưỡng bức là gì?**  
Đây là sự kiện SUMO đưa một phương tiện ra khỏi mạng trong tình huống không thể tiếp tục mô phỏng bình thường, thường liên quan đến bế tắc nghiêm trọng. Trong đề án, đây là chỉ số hỗ trợ vì còn chịu ảnh hưởng của cấu hình mô phỏng và dao động theo hạt giống.

## Học tăng cường đa tác tử

**MARL là gì?**  
MARL là học tăng cường đa tác tử: nhiều tác tử cùng ra quyết định trong một môi trường có ảnh hưởng lẫn nhau. Trong đề án, các tác tử cùng hướng tới cải thiện vận hành của toàn mạng lưới.

**Actor và critic là gì?**  
Actor là mạng chọn hành động từ quan sát. Critic ước lượng giá trị kỳ vọng của trạng thái để đánh giá hành động và hỗ trợ cập nhật actor.

**Lô dữ liệu tương tác (rollout) là gì?**  
Lô dữ liệu tương tác là một đoạn dữ liệu được thu thập khi chính sách hiện tại chạy trong môi trường. Ở mỗi bước quyết định, môi trường lưu tối thiểu quan sát, hành động được chọn, phần thưởng, giá trị do critic dự đoán, trạng thái kế tiếp và thông tin kết thúc lần mô phỏng.

Trong đề án, một lô dữ liệu tương tác được tạo bằng việc lặp luồng `SUMO → quan sát → bộ lọc hành động → hành động → phần thưởng` trong quá trình mô phỏng. Lô dữ liệu tương tác không phải là một lần cập nhật mạng. Nó được lưu trong bộ đệm dữ liệu tương tác và là đầu vào cho bước huấn luyện: critic dùng dữ liệu này để ước lượng giá trị, GAE tính lợi thế `A_t`, sau đó PPO dùng lợi thế đó để cập nhật actor và critic.

**Phân biệt lô dữ liệu tương tác và lần mô phỏng:**  
Lần mô phỏng (episode) là toàn bộ một lần chạy tối đa 3.600 giây. Lô dữ liệu tương tác (rollout) là một đoạn dữ liệu được gom để thực hiện cập nhật; tùy cấu hình, nó có thể trùng với một lần mô phỏng hoặc chỉ là một phần của lần mô phỏng.

**PPO là gì?**  
PPO, hay Proximal Policy Optimization, là phương pháp cập nhật actor sao cho chính sách mới không đi quá xa chính sách đã dùng để thu thập dữ liệu. Sau một lô dữ liệu tương tác, actor cũ đã sinh ra các hành động và phần thưởng. PPO dùng các dữ liệu đó để làm actor mới ưu tiên các hành động có lợi thế dương và giảm xác suất của hành động có lợi thế âm.

Điểm quan trọng là PPO không cho phép xác suất chọn một hành động thay đổi quá mạnh trong một lần cập nhật. Nó xét tỷ lệ giữa xác suất hành động dưới chính sách mới và dưới chính sách cũ:

`r_t(theta) = pi_theta(a_t | o_t) / pi_theta_cu(a_t | o_t)`.

Nếu tỷ lệ này vượt quá một khoảng cho phép, mục tiêu tối ưu bị cắt (clipping). Nhờ đó, một batch dữ liệu không làm chính sách đổi quá đột ngột và gây mất ổn định. Trong đề án, PPO là bước cập nhật chính sách sau khi đã thu thập dữ liệu tương tác và tính lợi thế bằng GAE.

**GAE là gì?**  
GAE, hay Generalized Advantage Estimation, là cách ước lượng lợi thế của hành động. Lợi thế trả lời câu hỏi: tại thời điểm đó, hành động vừa chọn tốt hơn hay kém hơn mức mà critic dự đoán cho trạng thái bao nhiêu.

Trước hết, critic dự đoán giá trị trạng thái `V(s_t)`. Sai số TD một bước được tính theo:

`delta_t = r_t + gamma V(s_(t+1)) - V(s_t)`.

GAE không chỉ dùng riêng sai số ở bước hiện tại mà cộng có trọng số các sai số TD ở những bước sau. Trọng số được điều khiển bởi `gamma` và `lambda`. `gamma` biểu thị mức coi trọng phần thưởng tương lai; `lambda` điều chỉnh mức kết hợp thông tin một bước với thông tin nhiều bước.

- `lambda` nhỏ: ước lượng dựa nhiều vào ít bước gần, phương sai thấp hơn nhưng có thể chệch nhiều hơn.
- `lambda` lớn: sử dụng nhiều thông tin tương lai hơn, ít chệch hơn nhưng có thể dao động nhiều hơn.

Vì vậy, GAE tạo một ước lượng lợi thế cân bằng giữa độ chệch và phương sai. Trong vòng huấn luyện, dữ liệu tương tác từ SUMO đi qua critic để tính giá trị, GAE tạo `A_t`, rồi PPO dùng `A_t` để cập nhật actor. Critic cũng được cập nhật để dự đoán giá trị trạng thái chính xác hơn.

**Nhớ nhanh mối liên hệ GAE--PPO:**  
GAE trả lời “hành động này tốt hay kém hơn dự đoán bao nhiêu”; PPO dùng câu trả lời đó để cập nhật actor một cách có giới hạn.

**MAPPO là gì?**  
MAPPO là PPO đa tác tử. Trong cách triển khai của đề án, các tác tử sử dụng actor chung và critic tập trung trong quá trình huấn luyện ở bối cảnh nhiều nút giao có ảnh hưởng lẫn nhau.

**HAPPO khác MAPPO thế nào?**  
HAPPO dùng actor riêng cho từng tác tử và cập nhật theo thứ tự tuần tự. MAPPO dùng actor chung. Đây là khác biệt chính cần nhớ trong đề án.

## GAT và quan hệ đồ thị

**GNN là gì?**  
GNN là mạng nơ-ron xử lý dữ liệu dạng đồ thị. Đồ thị trong đề án biểu diễn các nút giao và quan hệ lân cận giữa chúng.

**GAT là gì?**  
GAT, hay Graph Attention Network, là một dạng GNN dùng cơ chế attention để gán trọng số khác nhau cho thông tin từ các nút lân cận khi tạo biểu diễn cho một nút.

**Message passing là gì?**  
Đó là quá trình một nút nhận và tổng hợp thông tin biểu diễn từ các nút lân cận theo cạnh của đồ thị. Với GAT, mức đóng góp của từng nút lân cận được attention điều chỉnh thay vì coi hoàn toàn như nhau.

**GAT được thêm vào đâu?**  
GAT thay phần mã hóa MLP bằng bộ mã hóa đồ thị. Đầu vào gồm đặc trưng các nút và ma trận kề; đầu ra là embedding cho từng nút để actor hoặc critic sử dụng.

**GAT có luôn làm kết quả tốt hơn không?**  
Không. Trong kết quả của đề án, GAT-MAPPO cao nhất về số xe hoàn thành nhưng MAPPO tốt hơn về thời gian chờ và tốc độ. Vì vậy, GAT không bảo đảm cải thiện đồng đều trên mọi chỉ số.

## Huấn luyện và đánh giá

**Checkpoint cuối là gì?**  
Đây là bản lưu mô hình sau huấn luyện, kèm cấu hình thí nghiệm cần thiết. Đề án dùng bản lưu cuối cùng của mỗi mô hình để đánh giá đa hạt giống.

**Hạt giống ngẫu nhiên là gì? Vì sao cần 10 hạt giống?**  
Hạt giống khởi tạo các thành phần ngẫu nhiên của lần chạy mô phỏng. Chạy 10 hạt giống giúp đánh giá kết quả không chỉ theo một lần chạy đơn lẻ và cho phép báo cáo trung bình cùng độ lệch chuẩn.

**Trung bình ± độ lệch chuẩn nghĩa là gì?**  
Trung bình là mức kết quả điển hình trên các hạt giống. Độ lệch chuẩn cho biết mức dao động quanh trung bình; giá trị nhỏ hơn thường cho thấy kết quả ổn định hơn.

**Max-Pressure là gì?**  
Đây là phương pháp điều khiển thích ứng chọn pha dựa trên áp lực giao thông. Nó là đối chứng mạnh hơn điều khiển chu kỳ cố định trong kịch bản này.

## Kết quả cần nhớ

- MAPPO: thời gian chờ trung bình thấp nhất, 42,79 giây; tốc độ trung bình cao nhất, 13,19 m/s.
- GAT-MAPPO: số xe hoàn thành hành trình cao nhất, 4.583,3 xe; chỉ cao hơn MAPPO 2,8 xe.
- Max-Pressure: tốt hơn điều khiển cố định ở ba chỉ số vận hành chính, nên là đối chứng thích ứng mạnh.
- Dịch chuyển cưỡng bức: không có xu hướng cải thiện rõ; chỉ dùng làm chỉ số hỗ trợ.

## Giới hạn trả lời an toàn

- Không nói đề án đề xuất thuật toán mới. Đóng góp là xây dựng kịch bản, môi trường, triển khai bốn biến thể và đánh giá trong cùng giao thức.
- Không khẳng định mô hình có thể triển khai ngay ngoài thực địa. Kết quả hiện ở mức mô phỏng SUMO.
- Không khẳng định một thuật toán tốt nhất trên mọi chỉ số. MAPPO và GAT-MAPPO nổi bật ở các chỉ số khác nhau.
- Khi bị hỏi ngoài phạm vi thực nghiệm: “Nội dung này chưa được em nghiên cứu trong phạm vi đề án; em sẽ tìm hiểu thêm.”
