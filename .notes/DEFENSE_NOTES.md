# Ghi chú ôn phản biện

Tài liệu này dùng để nhớ các khái niệm dễ bị hỏi khi bảo vệ. Câu trả lời nên ngắn, sau đó chỉ mở rộng khi Hội đồng hỏi tiếp.

## Luồng bài toán

**Một tác tử là gì?**  
Trong đề án, một tác tử tương ứng với một bộ điều khiển đèn tín hiệu được đưa vào quá trình huấn luyện. Có 41 tác tử điều khiển trong kịch bản.

**Một bước quyết định diễn ra như thế nào?**  
Môi trường thực nghiệm được xây dựng trên SUMO. Sau mỗi 5 giây mô phỏng, lớp môi trường điều khiển lấy trạng thái từ SUMO, tạo quan sát cho từng tác tử, lọc các hành động không hợp lệ, thực thi hành động và tính phần thưởng. Dữ liệu tương tác sau đó được dùng để cập nhật chính sách khi huấn luyện.

**Lần mô phỏng (episode) là gì?**  
Một lần mô phỏng kéo dài tối đa 3.600 giây. Với bước quyết định 5 giây, một lần mô phỏng có tối đa 720 bước quyết định.

**Khác nhau giữa trạng thái và quan sát?**  
Trạng thái là thông tin giao thông mà SUMO cung cấp cho môi trường. Quan sát là vector thông tin cục bộ mà môi trường trích xuất và đưa cho một tác tử để chọn hành động.

## Dòng dữ liệu và cấu trúc dữ liệu

Mục này mô tả dữ liệu đang di chuyển trong lần chạy hiện hành. Ký hiệu `N = 41` là số tác tử điều khiển, `F_obs = 8` là số chiều quan sát cơ sở, `F_node = 9` là số chiều đặc trưng nút trong code, `A_max` là số hành động lớn nhất của một tác tử và `T <= 128` là số bước trong một rollout.

### 1. Cấu hình, tệp kịch bản và đối tượng môi trường

**`ScenarioPaths` là gì?**
Đây là dataclass chứa các đường dẫn của một kịch bản: `network.net.xml`, `demand.rou.xml`, `vtypes.add.xml`, `scenario.sumocfg` và các thư mục `runs/checkpoints`, `runs/logs`, `runs/eval`, `runs/results`. Nó bảo đảm huấn luyện và đánh giá cùng dùng một vị trí tệp rõ ràng.

**`EnvConfig` chứa gì?**
Các tham số môi trường gồm bước quyết định `delta_time = 5`, thời lượng lần mô phỏng `episode_seconds = 3600`, thời gian xanh tối thiểu `min_green_seconds = 10`, ngưỡng xe dừng `queue_speed_threshold = 0.1 m/s` và tham số xây đồ thị chính sách.

**`TrainConfig` chứa gì?**
Đây là cấu hình huấn luyện: tên thuật toán, số lần mô phỏng, `rollout_steps = 128`, số epoch cập nhật, kích thước minibatch, `gamma`, `gae_lambda`, `clip_ratio`, learning rate, entropy coefficient và kích thước các tầng ẩn.

**`SignalSpec` là gì?**
`signal_specs: dict[str, SignalSpec]` lưu cấu hình đèn của từng bộ điều khiển, với khóa là `agent_id`. Mỗi `SignalSpec` có danh sách các pha gốc, các chỉ số giai đoạn xanh, ánh xạ từ pha gốc sang giai đoạn xanh, giai đoạn xanh kế tiếp và đường đi qua pha vàng/đỏ để chuyển sang giai đoạn đó.

**`RuntimeSignalState` là gì?**
`signal_states: dict[str, RuntimeSignalState]` là trạng thái thay đổi trong lúc SUMO chạy. Với mỗi đèn, nó lưu `active_stage`, `pending_target_stage`, cờ `in_transition`, `stage_entry_time` và `raw_phase` hiện tại.

**Các tập tác tử được lưu ra sao?**
- `all_agent_ids`: mọi bộ điều khiển đèn trong mạng.
- `policy_agent_ids`: 41 bộ điều khiển được học.
- `infra_agent_ids`: bộ điều khiển còn lại, vẫn chạy trong SUMO nhưng không nhận hành động từ policy.
- `policy_agent_index: dict[str, int]`: ánh xạ ID tác tử sang vị trí `0..N-1`, bảo đảm mọi mảng giữ cùng thứ tự.

### 2. Dữ liệu một bước tương tác

**Dữ liệu thô từ SUMO là gì?**
Qua TraCI, môi trường nhận danh sách xe trên từng làn, tốc độ xe, thời gian chờ, pha đèn hiện tại, thời gian pha đã chạy, xe đến đích và xe bị dịch chuyển cưỡng bức. Các dữ liệu này là dữ liệu mô phỏng thô, chưa phải đầu vào trực tiếp của actor.

**`obs` có kiểu và kích thước gì?**
`obs: dict[str, np.ndarray]`, trong đó mỗi khóa là một `agent_id` và mỗi giá trị là vector `float32` có dạng `[8]`:

`[incoming_queue, incoming_moving, outgoing_queue, local_pressure, waiting_delta, current_stage_norm, elapsed_norm, can_switch]`.

Ba thành phần lưu lượng dùng PCU, không phải số xe thô. `incoming_queue` và `outgoing_queue` là PCU xe dừng; `incoming_moving` là PCU xe đang di chuyển.

**`snapshot` có vai trò gì?**
`snapshot: dict[str, dict[str, float]]` là bản ghi trung gian theo tác tử, gồm `incoming_queue_pcu`, `incoming_moving_pcu`, `outgoing_queue_pcu`, `pressure_pcu` và `waiting_delta_s`. Nó dùng để tính phần thưởng và tổng hợp chỉ số, không truyền thẳng vào mạng.

**`node_features` có kiểu và kích thước gì?**
`node_features: np.ndarray` có dạng `[N, 9]` và kiểu `float32`. Môi trường xếp chồng các vector quan sát 8 chiều theo thứ tự `policy_agent_ids`, rồi ghép thêm một cột cố định: tỷ lệ số giai đoạn xanh của tác tử so với số giai đoạn xanh lớn nhất trong mạng.

Phân biệt cần nhớ: `obs` cục bộ là 8 chiều; `node_features` trong code là 9 chiều. `node_features` là đầu vào chung mà vòng huấn luyện truyền cho agent. Với GAT, đây cũng là ma trận đặc trưng nút đi cùng ma trận kề.

**`global_state` có kiểu và kích thước gì?**
`global_state: np.ndarray` có dạng `[N * 9]`, tức `[369]` khi `N = 41`. Nó là `node_features.reshape(-1)`, được critic tập trung dùng để ước lượng giá trị.

**`policy_adj` có kiểu và kích thước gì?**
`policy_adj: np.ndarray` có dạng `[N, N]`, tức `[41, 41]`. Phần tử bằng 1 biểu thị hai tác tử là lân cận trong đồ thị chính sách, có cả self-loop. GAT dùng ma trận này để truyền thông tin giữa các nút; các biến thể không GAT vẫn giữ nó trong `info` và checkpoint để truy vết cấu hình.

**`info` chứa gì?**
`info: dict[str, object]` do `reset()` và `step()` trả về, gồm:

- `action_masks: dict[str, np.ndarray]`;
- `metrics: dict[str, float]`;
- `global_state: np.ndarray[N * 9]`;
- `node_features: np.ndarray[N, 9]`;
- `policy_adj: np.ndarray[N, N]` và thống kê đồ thị;
- danh sách tác tử policy/hạ tầng, kích thước hành động và dữ liệu debug pha đèn.

**`action_masks` có kiểu và kích thước gì?**  
`action_masks` là cách code biểu diễn **cơ chế lọc hành động** trong luận văn. Ở dạng dict, mỗi tác tử có một mảng Boolean độ dài `A_max`. Sau khi gọi `mask_dict_to_array`, dữ liệu thành `np.ndarray[N, A_max]`. Với một tác tử có `n_i` giai đoạn xanh, không gian hành động riêng có `1 + n_i` vị trí: hành động 0 là duy trì; các vị trí còn lại là yêu cầu chuyển. Tại một thời điểm, mask chỉ bật hành động duy trì và, nếu đã đủ thời gian xanh tối thiểu, một hành động chuyển sang giai đoạn xanh kế tiếp.

**Actor trả về gì?**
`agent.act(...)` trả về ba mảng theo thứ tự tác tử:

- `actions: np.ndarray[N]`, kiểu `int64`;
- `log_probs: np.ndarray[N]`, kiểu `float32`;
- `values: np.ndarray[N]`, kiểu `float32` do critic ước lượng.

`actions` được đổi thành `action_dict: dict[str, int]` để môi trường biết hành động nào thuộc về đèn nào.

**`env.step(action_dict)` trả về gì?**
`next_obs, reward_dict, terminated, truncated, info`.

- `next_obs` có cùng cấu trúc với `obs`.
- `reward_dict: dict[str, float]` có một reward cho từng policy agent.
- `terminated` là True khi SUMO đã hết xe cần mô phỏng.
- `truncated` là True khi đạt giới hạn 3.600 giây nhưng vẫn còn xe.
- `done = terminated or truncated`.

Trong `step`, môi trường kiểm tra action mask, gửi lệnh chuyển pha hợp lệ, chạy SUMO 5 bước một giây, rồi thu thập trạng thái tiếp theo và reward.

**Reward được đổi thành mảng như thế nào?**
`reward_dict_to_array` đổi `reward_dict` thành `rewards: np.ndarray[N]` theo đúng thứ tự `policy_agent_ids`. Điều này giúp reward thẳng hàng với `actions`, `values` và các hàng của `node_features`.

### 3. Bộ đệm rollout và cập nhật PPO

**`RolloutBuffer` giữ những cấu trúc nào?**
Trong tối đa `T = 128` bước, bộ đệm dùng các list để tích lũy:

| Trường | Dạng sau khi gom đủ T bước | Ý nghĩa |
|---|---|---|
| `node_features` | `[T, N, 9]` float32 | Đặc trưng đầu vào actor |
| `global_states` | `[T, N*9]` float32 | Trạng thái toàn cục cho critic |
| `action_masks` | `[T, N, A_max]` bool | Hành động hợp lệ tại lúc chọn |
| `actions` | `[T, N]` int64 | Hành động thực hiện |
| `log_probs` | `[T, N]` float32 | Log-xác suất theo policy cũ |
| `values` | `[T, N]` float32 | Giá trị critic dự đoán |
| `rewards` | `[T, N]` float32 | Reward từ môi trường |
| `dones` | `[T]` float32 | Cờ kết thúc lần mô phỏng |

Mỗi bước, `RolloutBuffer.add()` sao chép dữ liệu để các mảng đã lưu không bị thay đổi khi vòng lặp sang bước mới.

**Khi nào buffer được cập nhật?**
Vòng huấn luyện cập nhật khi `rollout.full` (đủ 128 bước) hoặc `done`. Nếu buffer đầy giữa một lần mô phỏng, nó là rollout ngắn hạn; lần mô phỏng không bị reset. Sau cập nhật, `rollout.clear()` xóa toàn bộ dữ liệu cũ, nên rollout không gối đầu.

**`last_values` là gì?**
Nếu rollout chưa kết thúc lần mô phỏng, critic nhận trạng thái kế tiếp để tạo `last_values: np.ndarray[N]`; đây là bootstrap cho GAE. Nếu `done`, `last_values` là vector 0 vì không có trạng thái tiếp theo trong lần mô phỏng đó.

**GAE tạo ra dữ liệu gì?**
`compute_returns_and_advantages()` biến list reward/value thành mảng và tạo:

- `advantages: np.ndarray[T, N]`, lợi thế của từng hành động;
- `returns: np.ndarray[T, N]`, mục tiêu hồi quy của critic, bằng `advantages + values`.

Trước PPO, `as_tensors()` đổi toàn bộ dữ liệu cần thiết sang `torch.Tensor` trên CPU hoặc GPU. Trong thực nghiệm này, SUMO và mô hình được chạy trên CPU.

**PPO sử dụng batch như thế nào?**
PPO chuẩn hóa `advantages`, xáo trộn trục thời gian `T` thành các minibatch và lặp `update_epochs`. Actor dùng `node_features`, `action_masks`, `actions`, `old_log_probs` và `advantages` để tính clipped policy loss. Critic dùng `global_states` và `returns` để tính MSE value loss.

**Bốn thuật toán khác nhau ở cấu trúc nào?**
- MAPPO: actor chung cho mọi tác tử; critic tập trung.
- HAPPO: một actor cho mỗi tác tử; cập nhật actor tuần tự; critic tập trung.
- GAT-MAPPO: actor/critic có bộ mã hóa GAT, nhận thêm `policy_adj`.
- GAT-HAPPO: actor riêng theo tác tử, cập nhật tuần tự và có bộ mã hóa GAT.

Cả bốn dùng cùng dạng dữ liệu môi trường, rollout, `advantages`, `returns` và quy trình PPO/GAE.

### 4. Bản lưu mô hình, đánh giá và dữ liệu kết quả

**Checkpoint là một dict gồm gì?**
Checkpoint lưu `algo`, `episode`, `metadata`, `model`, `optimizer` và `last_result_row`. Metadata gồm chữ ký tệp kịch bản, tên kịch bản, số chiều đầu vào, danh sách tác tử, kích thước hành động, ma trận kề, cấu hình GAT và kích thước tầng ẩn.

Khi đánh giá, code so sánh `policy_agent_ids`, `action_dims`, chữ ký kịch bản và, với mô hình đồ thị, tham số/matrix đồ thị. Nếu không khớp, checkpoint bị từ chối thay vì đánh giá nhầm kịch bản.

**Đánh giá khác huấn luyện ở dữ liệu nào?**
Đánh giá lặp cùng luồng `obs → node_features/global_state → action_masks → agent.act → action_dict → env.step`, nhưng gọi `agent.act(..., deterministic=True)`. `log_probs` và `values` trả về bị bỏ qua; không tạo `RolloutBuffer`, không tính GAE và không gọi `update`.

**`metrics` có kiểu và trường nào?**
`metrics: dict[str, float]` gồm `throughput`, `completed_trips`, `avg_trip_wait_s`, `teleports`, `mean_speed_mps`, `max_active_vehicles`, `total_queue_pcu` và `total_pressure_pcu`.

**Dữ liệu kết quả đa hạt giống được lưu như thế nào?**
`raw_rows: list[dict[str, object]]` có một dòng cho mỗi cặp phương pháp--hạt giống. Các trường gồm protocol ID, seed, loại controller, thuật toán, tên/cấu hình kịch bản, checkpoint và các metrics. Danh sách này được ghi thành `*_raw.csv`.

Từ các raw rows, code tạo `summary_rows: list[dict[str, object]]`: mỗi phương pháp có một dòng tổng hợp số seed, mean, standard deviation, khoảng tin cậy 95% cho bốn chỉ số chính và chênh lệch trung bình so với điều khiển cố định. Dữ liệu này được ghi thành `*_summary.csv` và là nguồn cho bảng/hình Chương 3.

### Chuỗi cần nhớ nhanh

`SUMO/TraCI → obs dict[N × 8] → node_features[N × 9] + global_state[N×9] + action_masks[N×A_max] → actor: actions/log_probs + critic: values → action_dict → env.step → reward_dict → rewards[N] → RolloutBuffer[T,...] → advantages/returns[T,N] → PPO update → checkpoint → đánh giá đa hạt giống → raw CSV → summary CSV → bảng và hình.`

## Quan sát, hành động và phần thưởng

**Quan sát 8 chiều gồm gì?**  
Gồm hàng đợi đầu vào `q_in`, dòng xe đầu vào `m_in`, hàng đợi đầu ra `q_out`, áp lực `pressure`, mức tăng thời gian chờ `delta_wait`, pha hiện tại đã chuẩn hóa `phase_norm`, thời gian pha đã chuẩn hóa `elapsed_norm` và khả năng chuyển pha `can_switch`.

**Phân biệt `q_in` và `m_in` thế nào?**

Cả hai đều được tính trên các làn tiếp cận và đều dùng đơn vị PCU. `q_in` là tổng PCU của xe đang dừng chờ, có tốc độ không quá `0,1 m/s`. `m_in` là tổng PCU của xe vẫn đang di chuyển, có tốc độ lớn hơn ngưỡng này. Hai đại lượng tách dòng xe đi vào thành phần đã ùn và phần còn đang tiếp cận nút giao.

**Các thành phần quan sát còn lại tính thế nào?**

- `q_out`: tổng PCU xe dừng chờ trên các làn thoát.
- `pressure = (q_in + m_in) - x_out`, trong đó `x_out` là tổng PCU của tất cả xe trên các làn thoát, gồm cả xe dừng lẫn xe đang chạy.
- `delta_wait`: tổng thời gian chờ tăng thêm của xe trên làn tiếp cận trong một bước quyết định 5 giây.
- `phase_norm`: chỉ số giai đoạn xanh hiện tại, chuẩn hóa về `[0, 1]`.
- `elapsed_norm`: thời gian đã qua của giai đoạn hiện tại, chuẩn hóa về `[0, 1]` với mốc 60 giây.
- `can_switch`: bằng `1,0` khi đã thỏa thời gian xanh tối thiểu, ngược lại bằng `0,0`.

**PCU là gì?**  
PCU, hay đơn vị xe con quy đổi, dùng để quy đổi các loại phương tiện khác nhau về cùng một đơn vị đo. Nhờ đó, hàng đợi và áp lực giao thông phản ánh tải trọng dòng xe hỗn hợp thay vì chỉ đếm số lượng xe.

Trong đề án, xe máy được quy đổi là `0,5` PCU, ô tô con là `1,0` PCU và xe buýt là `3,0` PCU. Vì vậy, một xe buýt được tính có ảnh hưởng lớn hơn xe máy khi tính hàng đợi, dòng xe và áp lực cục bộ.

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

### Cách actor, critic, GAE và PPO tính trong mã nguồn

**Actor chọn hành động như thế nào?**

Với mỗi tác tử, actor nhận `node_features` của tác tử đó và xuất một vector điểm số chưa chuẩn hóa, gọi là `logits`, cho các hành động có thể có. Cơ chế lọc hành động đặt điểm số của các hành động không hợp lệ về một giá trị rất nhỏ, nên phân phối xác suất sau đó chỉ còn các hành động hợp lệ.

Từ các `logits` đã lọc, code tạo phân phối rời rạc `Categorical(logits=masked_logits)`. Trong huấn luyện, actor lấy mẫu một hành động từ phân phối này để còn khám phá. Đồng thời, code lưu `log_prob`, tức log-xác suất của chính hành động đã lấy mẫu. Khi đánh giá, code không lấy mẫu mà chọn hành động có `logit` lớn nhất trong các hành động hợp lệ.

Nói ngắn gọn khi bảo vệ: **actor không trực tiếp chọn thời lượng đèn; actor cho điểm các lệnh giữ/chuyển hợp lệ, rồi trong huấn luyện lấy mẫu theo các điểm đó.**

**Critic tính gì và từ đâu?**

Critic không chọn hành động. Nó ước lượng `V_t`, là tổng reward kỳ vọng còn lại từ trạng thái hiện tại. Code trả về một giá trị cho mỗi tác tử: `values: [N]`. Với MAPPO và HAPPO, critic tập trung nhận `global_state`, là vector ghép đặc trưng của 41 tác tử. Với GAT-MAPPO và GAT-HAPPO, critic nhận ma trận đặc trưng nút và ma trận kề để mã hóa quan hệ lân cận trước khi xuất các giá trị.

Giá trị critic không phải reward thật. Nó là dự đoán trước khi SUMO chạy thêm 5 giây. Reward thật sau bước đó sẽ được dùng để kiểm tra và cải thiện dự đoán này.

**GAE được tính như thế nào?**

Với mỗi bước `t`, code trước hết tính sai số TD:

`delta_t = r_t + gamma * V_(t+1) * (1 - done_t) - V_t`.

Trong đó, `r_t` là reward môi trường trả về, `V_t` là critic dự đoán ở bước hiện tại, `V_(t+1)` là giá trị bootstrap của trạng thái kế tiếp, còn `done_t` bằng 1 ở bước kết thúc lần mô phỏng để không cộng giá trị của trạng thái không tồn tại. Nếu rollout đầy nhưng episode chưa kết thúc, `V_(t+1)` lấy từ critic ở quan sát kế tiếp. Nếu episode kết thúc, giá trị này là 0.

Sau đó code duyệt ngược từ cuối rollout về đầu và tích lũy:

`A_t = delta_t + gamma * lambda * (1 - done_t) * A_(t+1)`.

Đây là GAE. `A_t` dương nghĩa là hành động tốt hơn dự đoán của critic; âm nghĩa là kém hơn dự đoán. Mục tiêu cho critic là `return_t = A_t + V_t`.

**PPO cập nhật như thế nào?**

Trước cập nhật, code chuẩn hóa toàn bộ `advantages` trong rollout. Với từng minibatch và từng epoch, actor tính lại log-xác suất của chính các hành động đã thực hiện bằng trọng số mới. Từ đó, code tính tỷ số:

`ratio_t = exp(new_log_prob_t - old_log_prob_t)`.

`old_log_prob_t` được lưu lúc thu thập rollout; `new_log_prob_t` được tính lại lúc cập nhật. Hàm mất mát actor là:

`L_actor = -mean(min(ratio_t * A_t, clip(ratio_t, 1-epsilon, 1+epsilon) * A_t))`.

Nếu `A_t` dương, actor tăng xác suất hành động đó; nếu `A_t` âm, actor giảm xác suất. Phép `clip` giới hạn mức thay đổi trong một lần cập nhật. Code còn trừ thêm `entropy_coef * entropy` vào loss actor để tránh phân phối hành động quá sớm trở nên quá chắc chắn.

Critic được cập nhật riêng bằng sai số bình phương trung bình:

`L_critic = MSE(V_t, return_t)`.

Hai optimizer Adam cập nhật actor và critic riêng; code cũng cắt gradient theo `max_grad_norm` để tránh bước cập nhật quá lớn.

**Điểm khác của HAPPO ở bước PPO:**

MAPPO cập nhật một actor chung cho mọi tác tử. HAPPO trước hết cập nhật critic, sau đó cập nhật từng actor riêng theo thứ tự. Khi cập nhật actor sau, code nhân thêm tỷ số thay đổi tích lũy của các actor đã cập nhật trước đó. Đây là lý do gọi là cập nhật tuần tự.

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

**Hạt giống là gì? Vì sao cần 10 hạt giống?**

Hạt giống khởi tạo các thành phần giả ngẫu nhiên của một lần chạy SUMO. Kết quả cuối dùng một tập 10 hạt giống cố định, được xác định trước: `7, 11, 19, 23, 31, 37, 41, 43, 47, 53`. Mỗi phương pháp được chạy trên cùng tập này để kết quả có thể tái lập và so sánh công bằng.

Chạy 10 hạt giống giúp đánh giá kết quả không chỉ theo một lần chạy đơn lẻ và cho phép báo cáo trung bình cùng độ lệch chuẩn.

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
