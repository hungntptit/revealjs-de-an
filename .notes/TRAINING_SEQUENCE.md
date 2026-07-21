# Sơ đồ tuần tự huấn luyện

Sơ đồ dưới đây đối chiếu với mã nguồn thí nghiệm tại `src/train.py`, `src/env.py` và `src/mappo.py` của kịch bản Thanh Xuân UTM.

Các điểm cần nhớ khi trình bày:

- Một lần mô phỏng (episode) có giới hạn thời gian 3.600 giây, tương đương tối đa 720 bước quyết định 5 giây. Lần mô phỏng cũng có thể kết thúc sớm khi SUMO không còn phương tiện dự kiến.
- Mỗi bước tạo dữ liệu cho toàn bộ 41 tác tử điều khiển. Mỗi tác tử nhận quan sát và mặt nạ hành động riêng, sau đó có một hành động trong vector hành động chung.
- Bộ đệm dữ liệu tương tác (`RolloutBuffer`) có sức chứa mặc định 128 bước. GAE và PPO được thực hiện khi bộ đệm đầy hoặc lần mô phỏng kết thúc, do đó một lần mô phỏng có thể có nhiều lần cập nhật.
- Bản lưu mới nhất được ghi sau mỗi lần mô phỏng; bản lưu định kỳ theo số lần mô phỏng được ghi mỗi 5 lần theo cấu hình mặc định.

```mermaid
sequenceDiagram
    autonumber
    participant T as Vòng lặp huấn luyện
    participant E as Môi trường SUMO
    participant S as SUMO/TraCI
    participant A as Actor-Critic MARL
    participant B as Bộ đệm dữ liệu
    participant P as GAE và PPO
    participant C as Bản lưu mô hình

    loop Tối đa 500 lần mô phỏng
        T->>E: reset(seed + chỉ số lần mô phỏng)
        E->>S: Khởi động SUMO, đồng bộ đèn tín hiệu
        S-->>E: Trạng thái giao thông ban đầu
        E-->>T: Quan sát 8 chiều, action mask, node features cho 41 tác tử

        loop Mỗi bước quyết định, tối đa 720 bước
            T->>A: node features, global state, action masks
            Note over A: Mỗi tác tử có mặt nạ và hành động riêng<br/>MAPPO/GAT-MAPPO sinh vector hành động chung<br/>HAPPO/GAT-HAPPO dùng actor riêng theo tác tử
            A-->>T: actions, log probabilities, critic values
            T->>E: step(action_dict cho 41 tác tử)
            E->>S: Yêu cầu chuyển pha hợp lệ
            loop 5 giây mô phỏng
                S-->>E: simulationStep, trạng thái xe và đèn
            end
            E-->>T: next observations, rewards, terminated/truncated, metrics
            T->>B: Lưu node/global state, masks, actions,<br/>log probabilities, values, rewards, done

            alt Bộ đệm đủ 128 bước hoặc lần mô phỏng kết thúc
                alt Lần mô phỏng chưa kết thúc
                    T->>A: Ước lượng bootstrap value của trạng thái kế tiếp
                    A-->>T: last values
                else Lần mô phỏng kết thúc
                    T->>T: last values = 0
                end
                T->>B: Tính returns và advantages bằng GAE
                T->>P: Cập nhật actor và critic bằng PPO
                P-->>T: actor loss, critic loss, entropy
                T->>B: Xóa bộ đệm dữ liệu
            end
        end

        T->>C: Ghi latest checkpoint và run checkpoint
        opt Lần mô phỏng chia hết cho 5
            T->>C: Ghi checkpoint định kỳ theo số lần mô phỏng
        end
        T->>T: Ghi chỉ số lần mô phỏng vào CSV
end
```

## Sơ đồ vòng lặp lồng nhau

```mermaid
flowchart TB
    Start([Bắt đầu huấn luyện]) --> EpisodeStart

    subgraph Outer["Vòng lặp lần mô phỏng: tối đa 500 lần"]
        direction TB
        EpisodeStart["reset SUMO với seed của lần mô phỏng<br/>Khởi tạo bộ đệm dữ liệu"] --> StepStart

        subgraph Inner["Vòng lặp bước quyết định: tối đa 720 bước"]
            direction TB
            StepStart["SUMO tạo trạng thái"] --> Observation["Tạo quan sát và action mask<br/>cho 41 tác tử"]
            Observation --> Action["Mỗi tác tử chọn hành động"]
            Action --> EnvStep["Thực thi trong SUMO 5 giây<br/>Nhận reward và trạng thái kế tiếp"]
            EnvStep --> Store["Lưu dữ liệu vào bộ đệm"]
            Store --> UpdateReady{"Bộ đệm đủ 128 bước<br/>hoặc lần mô phỏng kết thúc?"}
            UpdateReady -- "Chưa" --> StepStart
            UpdateReady -- "Có" --> GAE["Tính returns và advantages bằng GAE"]
            GAE --> PPO["Cập nhật actor và critic bằng PPO"]
            PPO --> Clear["Xóa bộ đệm dữ liệu"]
            Clear --> EpisodeDone{"Lần mô phỏng kết thúc?"}
            EpisodeDone -- "Chưa" --> StepStart
        end

        EpisodeDone -- "Rồi" --> EpisodeLog["Ghi chỉ số của lần mô phỏng"]
    end

    EpisodeLog --> MoreEpisodes{"Đã đủ 500 lần mô phỏng?"}
    MoreEpisodes -- "Chưa" --> EpisodeStart
    MoreEpisodes -- "Rồi" --> FinalModel([Mô hình cuối cùng])
```
