# Apache Kafka: tài liệu tổng quan về mô hình, kiến trúc, đối tượng, sao chép, workflow, câu lệnh, và gỡ lỗi

> **Viết cho:** người muốn hiểu Kafka từ khái niệm đến vận hành: từ luồng sự kiện, topic và partition, đến sao chép, bảo đảm giao nhận, và xử lý sự cố. Cấu trúc tương tự các tài liệu Docker, Swarm, Compose và Kubernetes để dễ đối chiếu.
> **Cách đọc:** Phần A đến C là mô hình và kiến trúc. Phần D đến F là topic, sao chép, producer và consumer. Phần G là bảo mật và cấu hình mạng. Phần I là tham chiếu lệnh. Phần J là gỡ lỗi. Các lệnh, cấu hình và đoạn mã có chú thích từng dòng bên cạnh.
> **Phiên bản tham chiếu:** Trang tải về và quickstart chính thức (tra cứu 2026-10-21) ghi bản mới nhất trong hướng dẫn là **Kafka 4.3.1**, yêu cầu **Java 17 trở lên**. Trang tải về liệt kê các dòng từ 4.3 trở xuống.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với kafka.apache.org; `[Trí nhớ]` = kiến thức nền, cần kiểm tra; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về mã:** các lệnh và cấu hình chưa được chạy thử trong lần này. Tên tham số cấu hình có thể thay đổi giữa các phiên bản. Kiểm tra với tài liệu đúng phiên bản bạn dùng.

---

## Mục lục

- **Phần A. Kafka là gì và giải quyết vấn đề gì**
  - 1. Từ hàng đợi đến luồng sự kiện
  - 2. Kafka làm được gì
  - 3. Phiên bản và KRaft
- **Phần B. Kiến trúc cluster**
  - 4. Broker
  - 5. Controller và KRaft
  - 6. Client: producer, consumer, và các thành phần mở rộng
  - 7. Luồng một bản ghi đi qua hệ thống
- **Phần C. Các khái niệm cốt lõi**
  - 8. Event (record): key, value, timestamp, header
  - 9. Topic và partition
  - 10. Offset
  - 11. Segment, retention, và compaction
- **Phần D. Sao chép và bền vững**
  - 12. Replica: leader và follower
  - 13. ISR, high watermark, và leader election
  - 14. `acks`, `min.insync.replicas`, và bền vững dữ liệu
  - 15. Unclean leader election
- **Phần E. Producer**
  - 16. Gửi bản ghi và partitioner
  - 17. Idempotence và transaction
  - 18. Nén và batch
- **Phần F. Consumer**
  - 19. Consumer group và phân công partition
  - 20. Commit offset và điều chỉnh vị trí đọc
  - 21. Rebalance
  - 22. Lag
- **Phần G. Mạng, bảo mật, và cấu hình**
  - 23. Listener, advertised.listeners, và bootstrap servers
  - 24. Bảo mật: TLS và SASL
  - 25. Cấu hình quan trọng của broker
  - 26. Lưu trữ và đĩa
- **Phần H. Workflow**
  - 27. Thiết kế topic
  - 28. Workflow phát triển cục bộ
  - 29. Quy trình vận hành và thay đổi
  - 30. Kiểm thử và CI
- **Phần I. Câu lệnh**
  - 31. Khởi động cục bộ (KRaft)
  - 32. Quản lý topic
  - 33. Producer và consumer console
  - 34. Consumer group và offset
  - 35. Cấu hình động
  - 36. Leader, phân bổ, và sao lưu partition
  - 37. Kiểm tra và hiệu năng
- **Phần J. Gỡ lỗi (troubleshooting)**
  - 38. Quy trình gỡ lỗi
  - 39. Không kết nối được broker
  - 40. Producer lỗi hoặc chậm
  - 41. Consumer lag và rebalance
  - 42. Under-replicated và offline partition
  - 43. Đĩa, retention, và kích thước bản ghi
  - 44. KRaft và controller
- **Phần K. Vận hành và giám sát**
  - 45. Chỉ số cần theo dõi
  - 46. Nâng cấp và bảo trì broker
- **Phần L. So sánh với các hệ thống khác**
- **Phần M. Thuật ngữ**
- **Phần N. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. KAFKA LÀ GÌ VÀ GIẢI QUYẾT VẤN ĐỀ GÌ

## 1. Từ hàng đợi đến luồng sự kiện

Khi nhiều hệ thống cần trao đổi dữ liệu, cách trực tiếp (mỗi hệ thống gọi thẳng nhau) tạo ra mạng lưới kết nối khó bảo trì. Hàng đợi tin nhắn tách người gửi và người nhận, nhưng thường xóa tin sau khi được đọc.

**Apache Kafka** là **nền tảng truyền luồng sự kiện** (event streaming platform). Kafka lưu các sự kiện thành luồng bền vững, nhiều bên có thể đọc cùng luồng đó, và sự kiện không bị xóa khi đã đọc. Thời gian lưu giữ do cấu hình quyết định. [Đã kiểm chứng: trang giới thiệu nêu sự kiện không bị xóa khi đọc, mà được giữ theo cấu hình retention]

## 2. Kafka làm được gì

Theo trang giới thiệu chính thức: [Đã kiểm chứng]
- **Publish và subscribe** luồng sự kiện, bao gồm nhập và xuất dữ liệu liên tục từ hệ thống khác.
- **Lưu trữ** luồng sự kiện một cách bền vững và đáng tin cậy, trong thời gian cần thiết.
- **Xử lý** luồng sự kiện khi xảy ra hoặc sau đó.

**Ứng dụng thường gặp:** theo dõi hoạt động người dùng, nhật ký, pipeline dữ liệu, tích hợp microservice, xử lý luồng thời gian thực, đồng bộ dữ liệu giữa hệ thống. [Trí nhớ]

**Không phải:** Kafka không thay thế cơ sở dữ liệu truy vấn tổng quát. Nó tối ưu cho ghi nối tiếp và đọc tuần tự theo offset. [Suy luận]

## 3. Phiên bản và KRaft

- Bản mới nhất trong hướng dẫn chính thức là **Kafka 4.3.1**, yêu cầu **Java 17+**. [Đã kiểm chứng]
- Quickstart chạy broker ở chế độ **KRaft** (Kafka Raft), không dùng ZooKeeper. [Đã kiểm chứng]
- Phiên bản 4.x chỉ hỗ trợ KRaft; ZooKeeper đã bị gỡ khỏi Kafka từ 4.0. [Trí nhớ; cần đối chiếu ghi chú phát hành]

**Lưu ý:** tài liệu cũ hoặc hướng dẫn trên mạng có thể mô tả ZooKeeper. Nếu gặp, đó là mô hình cũ. [Suy luận]

---

# PHẦN B. KIẾN TRÚC CLUSTER

```
┌──────────────────────── CLUSTER KAFKA ────────────────────────┐
│                                                                │
│  CONTROLLER (KRaft quorum)                                     │
│    • Quản lý metadata: topic, partition, leader, cấu hình      │
│    • Lưu nhật ký metadata, bầu leader partition                │
│                                                                │
│  BROKER 1            BROKER 2            BROKER 3              │
│  ┌──────────┐        ┌──────────┐        ┌──────────┐         │
│  │ P0 leader│        │ P0 follower       │ P0 follower│        │
│  │ P1 follow│        │ P1 leader│        │ P1 follower│        │
│  │ P2 follow│        │ P2 follow│        │ P2 leader  │        │
│  └──────────┘        └──────────┘        └──────────┘         │
└────────────▲───────────────────────────────────▲──────────────┘
             │ produce                           │ consume
       ┌─────┴─────┐                       ┌─────┴─────┐
       │ Producer  │                       │ Consumer  │
       └───────────┘                       └───────────┘
```

## 4. Broker

**Broker** là máy chủ lưu partition và phục vụ producer, consumer. Một cluster gồm nhiều broker. Mỗi partition có một leader và một số follower trên các broker khác. [Đã kiểm chứng: trang giới thiệu gọi broker là các server tạo thành tầng lưu trữ]

## 5. Controller và KRaft

**KRaft** là cơ chế quản lý metadata bằng Raft ngay trong Kafka, thay thế ZooKeeper. [Trí nhớ]

- **Controller:** một nhóm node (thường 3 hoặc 5) đồng thuận về metadata: topic, partition, leader, cấu hình, thành viên.
- Có thể chạy **combined mode** (broker và controller trên cùng process) cho cluster nhỏ, hoặc **tách riêng** cho cluster lớn. [Trí nhớ]
- Metadata được ghi vào nhật ký metadata nội bộ, các broker đọc và cập nhật theo. [Trí nhớ]

**Số lượng controller:** nên dùng số lẻ, tối thiểu 3 trong môi trường sản xuất, theo cùng nguyên tắc đa số như các hệ thống Raft khác. [Suy luận]

## 6. Client: producer, consumer, và các thành phần mở rộng

| Thành phần | Vai trò |
|---|---|
| **Producer** | Ghi bản ghi vào topic |
| **Consumer** | Đọc bản ghi từ topic, theo offset |
| **Kafka Connect** | Nhập/xuất dữ liệu giữa Kafka và hệ thống khác (cơ sở dữ liệu, object storage) [Trí nhớ] |
| **Kafka Streams** | Thư viện xử lý luồng trong ứng dụng [Trí nhớ] |
| **Schema Registry** | Quản lý schema của bản ghi (thường là sản phẩm bên thứ ba) [Trí nhớ] |

## 7. Luồng một bản ghi đi qua hệ thống

```
1. Producer gửi bản ghi (key, value) đến topic
2. Partitioner chọn partition (theo key, hoặc luân phiên)
3. Leader của partition ghi bản ghi vào log cục bộ
4. Follower kéo bản ghi từ leader và ghi vào log của mình
5. Khi đủ replica trong ISR đã ghi (theo acks), leader xác nhận cho producer
6. Bản ghi trở nên khả dụng cho consumer đọc (đến high watermark)
7. Consumer đọc theo offset và commit offset đã xử lý
```

[Đã kiểm chứng về các khái niệm; thứ tự chi tiết là suy luận từ mô hình sao chép]

---

# PHẦN C. CÁC KHÁI NIỆM CỐT LÕI

## 8. Event (record): key, value, timestamp, header

Mỗi sự kiện gồm: [Đã kiểm chứng]
- **key** (tùy chọn): quyết định partition. Bản ghi cùng key vào cùng partition.
- **value:** nội dung sự kiện (bytes, thường là JSON, Avro, Protobuf).
- **timestamp:** thời điểm ghi nhận.
- **headers** (tùy chọn): cặp khóa-giá trị siêu dữ liệu.

```
key:       "don-hang-101"
value:     {"trang_thai": "da_dat", "tong": 750000}
timestamp: 2026-10-21T09:00:00Z
headers:   trace_id=abc123
```

## 9. Topic và partition

- **Topic:** tên luồng sự kiện, như một thư mục chứa các bản ghi. [Đã kiểm chứng]
- **Partition:** topic được chia thành nhiều partition, phân bố trên các broker. Partition là đơn vị song song và đơn vị thứ tự. [Đã kiểm chứng]

**Quy tắc thứ tự:** Kafka đảm bảo thứ tự **trong một partition**, không đảm bảo thứ tự giữa các partition. Các bản ghi cùng key vào cùng partition, nên giữ được thứ tự theo key. [Đã kiểm chứng]

**Số partition:** quyết định mức song song tối đa của consumer group. Tăng sau này có thể làm thay đổi phân bổ key sang partition mới. [Trí nhớ]

```sh
bin/kafka-topics.sh --create --topic don-hang --partitions 6 --replication-factor 3 --bootstrap-server localhost:9092
# --partitions 6: chia 6 partition
# --replication-factor 3: mỗi partition có 3 bản sao
```

## 10. Offset

**Offset** là số thứ tự của bản ghi trong một partition, bắt đầu từ 0 và tăng dần. Consumer nhớ offset để biết đã đọc đến đâu. [Đã kiểm chứng: trang giới thiệu nói consumer đọc theo thứ tự ghi trong partition]

Bộ ba định danh: `(topic, partition, offset)`.

## 11. Segment, retention, và compaction

- **Segment:** partition được lưu thành các tệp đoạn (segment). Khi đầy, tạo segment mới. Segment cũ bị xóa theo retention. [Trí nhớ]
- **Retention theo thời gian hoặc kích thước:** giữ bản ghi trong một khoảng hoặc đến một dung lượng, sau đó xóa. [Đã kiểm chứng: trang giới thiệu nói retention theo cấu hình từng topic]
- **Log compaction:** giữ bản ghi mới nhất cho mỗi key, xóa các bản cũ hơn. Phù hợp cho trạng thái dạng bảng (table). [Trí nhớ]

```sh
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name don-hang \
  --alter --add-config retention.ms=604800000       # giữ 7 ngày (mili-giây)
```

---

# PHẦN D. SAO CHÉP VÀ BỀN VỮNG

## 12. Replica: leader và follower

Mỗi partition có **replication factor** (ví dụ 3): số bản sao trên các broker khác nhau. Một bản là **leader** (nhận đọc và ghi), các bản còn lại là **follower** (sao chép từ leader). [Đã kiểm chứng: trang giới thiệu nói sao chép theo từng partition, hệ số phổ biến là 3]

## 13. ISR, high watermark, và leader election

- **ISR (In-Sync Replicas):** tập replica đang bắt kịp leader. [Trí nhớ]
- **High watermark:** offset cao nhất đã được sao chép đủ, consumer chỉ đọc đến đây. [Trí nhớ]
- **Leader election:** khi leader hỏng, một replica trong ISR được bầu làm leader mới. [Trí nhớ]

## 14. `acks`, `min.insync.replicas`, và bền vững dữ liệu

Hai cấu hình quyết định mức bền vững khi ghi:

| `acks` (producer) | Ý nghĩa | Đánh đổi |
|---|---|---|
| `0` | Không chờ xác nhận | Nhanh nhất, có thể mất dữ liệu |
| `1` | Chờ leader ghi xong | Có thể mất nếu leader hỏng trước khi follower sao chép |
| `all` (hoặc `-1`) | Chờ mọi replica trong ISR ghi xong | An toàn nhất, chậm hơn |

[Trí nhớ; tên và ý nghĩa là kiến thức nền của Kafka producer]

**`min.insync.replicas`** (cấu hình broker hoặc topic): số replica tối thiểu trong ISR phải có để chấp nhận ghi khi `acks=all`. Ví dụ `replication.factor=3`, `min.insync.replicas=2`: chịu được một broker hỏng mà vẫn ghi được. [Trí nhớ]

**Cảnh báo:** nếu ISR nhỏ hơn `min.insync.replicas`, producer nhận lỗi `NotEnoughReplicas`. Đây là cơ chế bảo vệ có chủ đích. [Trí nhớ]

```sh
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name don-hang \
  --alter --add-config min.insync.replicas=2        # cần ít nhất 2 replica trong ISR
```

## 15. Unclean leader election

Nếu cho phép bầu leader từ replica **ngoài ISR** (unclean election), cluster có thể tiếp tục khi mất hết ISR, nhưng có nguy cơ **mất dữ liệu** đã được xác nhận. [Trí nhớ]

**Khuyến nghị:** giữ tắt (mặc định) cho dữ liệu quan trọng; chấp nhận gián đoạn thay vì mất dữ liệu. [Suy luận]

---

# PHẦN E. PRODUCER

## 16. Gửi bản ghi và partitioner

- Bản ghi có key: partitioner băm key để chọn partition, nên cùng key vào cùng partition.
- Bản ghi không key: chọn partition luân phiên hoặc theo batch (tùy phiên bản và cấu hình). [Trí nhớ]

## 17. Idempotence và transaction

- **Idempotent producer** (`enable.idempotence=true`): chống ghi trùng khi producer retry, nhờ số thứ tự theo producer và partition. [Trí nhớ]
- **Transaction:** ghi nhiều partition hoặc topic thành một đơn vị nguyên tử, consumer đọc với `isolation.level=read_committed` để chỉ thấy bản ghi đã commit. [Trí nhớ]

**Lưu ý:** idempotence và transaction giảm trùng lặp và đảm bảo nguyên tử trong phạm vi Kafka. Không tự động đảm bảo tính nguyên tử với hệ thống bên ngoài như cơ sở dữ liệu. [Suy luận]

## 18. Nén và batch

- `compression.type` (ví dụ `lz4`, `zstd`, `snappy`, `gzip`): giảm dung lượng mạng và đĩa. [Trí nhớ]
- `linger.ms` và `batch.size`: gom nhiều bản ghi trước khi gửi, tăng thông lượng với đánh đổi độ trễ. [Trí nhớ]

---

# PHẦN F. CONSUMER

## 19. Consumer group và phân công partition

**Consumer group:** nhóm consumer chia nhau partition của topic. Mỗi partition chỉ được một consumer trong group đọc tại một thời điểm. [Trí nhớ]

- Nhiều group đọc cùng topic độc lập nhau.
- Số consumer hữu ích tối đa trong một group bằng số partition. Consumer thừa ở trạng thái nhàn rỗi. [Trí nhớ]

## 20. Commit offset và điều chỉnh vị trí đọc

- Consumer **commit offset** để lưu tiến độ. Offset commit được lưu trong topic nội bộ `__consumer_offsets`. [Trí nhớ]
- `auto.offset.reset`: chọn điểm bắt đầu khi không có offset đã commit (`earliest` hoặc `latest`). [Trí nhớ]
- `enable.auto.commit`: tự động commit định kỳ. Cân nhắc khi cần đảm bảo "xử lý xong mới commit". [Trí nhớ]

**Ngữ nghĩa giao nhận:**
- **At most once:** commit trước khi xử lý. Có thể mất khi lỗi.
- **At least once:** xử lý xong rồi commit. Có thể xử lý lặp, nên xử lý phải idempotent.
- **Exactly once:** cần transaction và cấu hình phù hợp, phạm vi nội bộ Kafka. [Trí nhớ]

## 21. Rebalance

Rebalance xảy ra khi thành viên group thay đổi (thêm, bớt consumer, hết timeout). Trong lúc rebalance, consumer trong group tạm dừng đọc. [Trí nhớ]

**Giảm rebalance không cần thiết:**
- Giữ `session.timeout.ms` và `heartbeat.interval.ms` hợp lý.
- Xử lý mỗi bản ghi đủ nhanh, tránh vượt `max.poll.interval.ms`. [Trí nhớ]
- Dùng chiến lược phân công phù hợp (cooperative rebalancing) nếu phiên bản hỗ trợ. [Trí nhớ]

## 22. Lag

**Consumer lag** là khoảng cách giữa offset mới nhất của partition và offset đã commit của group. Lag tăng nghĩa là consumer không theo kịp producer. [Trí nhớ]

```sh
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group nhom-xu-ly
# Cột LAG cho biết khoảng cách còn lại trên từng partition
```

---

# PHẦN G. MẠNG, BẢO MẬT, VÀ CẤU HÌNH

## 23. Listener, advertised.listeners, và bootstrap servers

- **`listeners`:** địa chỉ và cổng broker thực sự lắng nghe. [Trí nhớ]
- **`advertised.listeners`:** địa chỉ broker **quảng bá** cho client. Client dùng địa chỉ này để kết nối lại sau khi lấy metadata. [Trí nhớ]
- **`bootstrap.servers`:** danh sách vài broker để client bắt đầu kết nối; không cần liệt kê hết. [Trí nhớ]

**Lỗi thường gặp:** `advertised.listeners` chứa tên nội bộ (như `kafka-1`) mà client bên ngoài không phân giải được. Client kết nối được bootstrap nhưng sau đó không kết nối được partition. [Suy luận]

## 24. Bảo mật: TLS và SASL

- **TLS:** mã hóa lưu lượng giữa client và broker, và giữa các broker. [Trí nhớ]
- **SASL:** xác thực người dùng (PLAIN, SCRAM, OAUTHBEARER, GSSAPI tùy cấu hình). [Trí nhớ]
- **ACL:** phân quyền đọc, ghi, tạo topic theo người dùng hoặc nhóm. [Trí nhớ]

```properties
# client.properties (ví dụ SASL/SCRAM qua TLS)
security.protocol=SASL_SSL                  # xác thực SASL và mã hóa TLS
sasl.mechanism=SCRAM-SHA-512                # cơ chế SASL
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="app" password="thay-doi";
ssl.truststore.location=/etc/kafka/truststore.jks   # kho tin cậy để xác minh broker
ssl.truststore.password=thay-doi
```

[Tên tham số là kiến thức nền; cần kiểm tra theo phiên bản]

**Nguyên tắc:** không đặt mật khẩu trong file commit. Dùng secret manager hoặc biến môi trường. [Suy luận]

## 25. Cấu hình quan trọng của broker

| Tham số | Ý nghĩa | Ghi chú |
|---|---|---|
| `node.id` | Định danh node trong KRaft | Duy nhất mỗi node |
| `process.roles` | `broker`, `controller`, hoặc cả hai | Chế độ combined hoặc tách riêng [Trí nhớ] |
| `controller.quorum.voters` | Danh sách controller | Dạng `id@host:port` [Trí nhớ] |
| `log.dirs` | Thư mục lưu dữ liệu | Nên đặt trên đĩa riêng, nhanh |
| `num.partitions` | Số partition mặc định cho topic mới | Đặt theo mức song song cần thiết |
| `default.replication.factor` | Hệ số sao chép mặc định | Thường đặt 3 |
| `min.insync.replicas` | ISR tối thiểu khi ghi `acks=all` | Thường đặt 2 khi RF = 3 |
| `unclean.leader.election.enable` | Cho phép bầu leader ngoài ISR | Giữ `false` cho dữ liệu quan trọng |
| `log.retention.hours` | Thời gian giữ mặc định | Có thể ghi đè theo topic |
| `auto.create.topics.enable` | Tự tạo topic khi producer ghi | Nên tắt trong sản xuất |

[Tên và ý nghĩa là kiến thức nền; cần đối chiếu tài liệu cấu hình của phiên bản]

## 26. Lưu trữ và đĩa

- Dùng đĩa riêng cho `log.dirs`, ưu tiên SSD cho cluster nhiều ghi.
- Theo dõi dung lượng: đầy đĩa làm broker dừng ghi và có thể offline partition. [Suy luận]
- Ước lượng dung lượng: tốc độ ghi × thời gian retention × hệ số sao chép, cộng biên an toàn. [Suy luận]

---

# PHẦN H. WORKFLOW

## 27. Thiết kế topic

Trước khi tạo topic, trả lời:
1. Sự kiện này là gì? Đặt tên topic theo sự kiện hoặc miền (ví dụ `don-hang.da-dat`).
2. Key là gì? Key quyết định thứ tự và phân bổ. Chọn key có đủ cardinality để phân tải đều.
3. Cần bao nhiêu partition? Theo lưu lượng và số consumer song song tối đa.
4. Hệ số sao chép và `min.insync.replicas`?
5. Retention bao lâu? Có cần compaction không?
6. Định dạng dữ liệu và schema có kiểm soát tiến hóa không?

**Quy tắc tăng partition:** tăng sau này làm thay đổi ánh xạ key sang partition, có thể phá vỡ giả định thứ tự theo key. Cân nhắc kỹ trước khi tăng. [Trí nhớ]

## 28. Workflow phát triển cục bộ

```yaml
# docker-compose.yml (ví dụ một broker KRaft, minh họa)
services:
  kafka:
    image: apache/kafka:4.3.1                 # image chính thức, ghim phiên bản (kiểm tra tag thật)
    ports:
      - "9092:9092"                           # cổng client từ máy
    environment:
      KAFKA_NODE_ID: 1                        # định danh node
      KAFKA_PROCESS_ROLES: broker,controller  # chế độ combined cho thử nghiệm
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092   # địa chỉ client sẽ dùng
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1   # chỉ 1 broker nên RF = 1
```

[Tên image và biến môi trường cần kiểm tra trên tài liệu của image chính thức; đây là cấu hình thử nghiệm, không dùng sản xuất]

```sh
docker compose up -d                                   # khởi động broker
docker compose exec kafka /opt/kafka/bin/kafka-topics.sh --create --topic thu --bootstrap-server localhost:9092
```

## 29. Quy trình vận hành và thay đổi

- Thay đổi cấu hình topic qua `kafka-configs.sh`, ghi lại trong repo.
- Tăng partition hoặc thay đổi replication factor có kế hoạch, theo dõi lag và ISR.
- Di chuyển replica bằng công cụ phân bổ lại (mục 36), theo đợt nhỏ.
- Thông báo cho đội consumer trước khi thay đổi có tác động đến thứ tự hoặc lưu lượng.

## 30. Kiểm thử và CI

- **Kiểm thử đơn vị:** dùng mock hoặc bộ kiểm thử của thư viện client; tách logic xử lý bản ghi khỏi I/O Kafka.
- **Kiểm thử tích hợp:** chạy Kafka thật trong container (ví dụ qua Testcontainers hoặc Compose), tạo topic, produce, consume, kiểm tra kết quả. [Trí nhớ]
- **Kiểm thử idempotent:** gửi trùng bản ghi, xác nhận đầu ra không đổi.
- **Kiểm thử sự cố:** dừng một broker khi đang ghi, xác nhận không mất bản ghi đã xác nhận với `acks=all`.

---

# PHẦN I. CÂU LỆNH

Các lệnh nằm trong thư mục `bin/` của bản phát hành. Trên Windows dùng phiên bản `.bat` tương ứng, ví dụ `bin\windows\kafka-topics.bat`. [Trí nhớ]

## 31. Khởi động cục bộ (KRaft)

```sh
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"                     # tạo định danh cluster
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/server.properties   # định dạng thư mục log
bin/kafka-server-start.sh config/server.properties                         # khởi động broker
```

[Đã kiểm chứng: trình tự từ quickstart chính thức]

## 32. Quản lý topic

```sh
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list                         # liệt kê topic
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic don-hang \
  --partitions 6 --replication-factor 3                                              # tạo topic
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic don-hang    # chi tiết: leader, replica, ISR
bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic don-hang --partitions 12   # tăng partition
bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic thu           # xóa topic (không hoàn tác)
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --under-replicated-partitions    # partition thiếu replica
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --unavailable-partitions         # partition không có leader
```

**Ghi chú:** `--describe` là lệnh quan trọng nhất khi gỡ lỗi. Đọc cột `Leader`, `Replicas`, `Isr`. [Suy luận]

## 33. Producer và consumer console

```sh
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic don-hang
# Mỗi dòng nhập vào là một bản ghi. Thoát bằng Ctrl-C.

bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic don-hang \
  --property parse.key=true --property key.separator=:    # nhập key:value
# Ví dụ nhập: don-101:{"tong":750000}

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic don-hang \
  --from-beginning                                          # đọc từ đầu topic
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic don-hang \
  --property print.key=true --property print.timestamp=true   # in kèm key và timestamp
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic don-hang \
  --group nhom-thu-nghiem                                   # đọc như một thành viên của group
```

## 34. Consumer group và offset

```sh
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list                       # danh sách group
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group nhom-xu-ly # offset và LAG từng partition
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --members --group nhom-xu-ly   # thành viên
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group nhom-xu-ly --topic don-hang \
  --reset-offsets --to-earliest --execute                    # đặt offset về đầu (group phải không hoạt động)
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group nhom-xu-ly --topic don-hang \
  --reset-offsets --to-datetime 2026-10-21T00:00:00.000 --execute   # đặt offset theo thời điểm
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group nhom-xu-ly --delete   # xóa group (không còn thành viên)
```

**Cảnh báo:** `--reset-offsets --execute` thay đổi vị trí đọc. Dừng consumer trước và chạy `--dry-run` để xem trước. [Suy luận]

## 35. Cấu hình động

```sh
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name don-hang --describe
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name don-hang \
  --alter --add-config retention.ms=86400000                 # 1 ngày
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name don-hang \
  --alter --delete-config retention.ms                       # quay về mặc định
bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-default --describe   # cấu hình mặc định broker
```

## 36. Leader, phân bổ, và sao chép partition

```sh
bin/kafka-leader-election.sh --bootstrap-server localhost:9092 --election-type preferred \
  --all-topic-partitions                                     # đưa leader về replica ưu tiên sau khi broker khởi động lại
bin/kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --generate \
  --topics-to-move-json-file topics.json --broker-list "1,2,3"   # sinh kế hoạch phân bổ (xem trước)
bin/kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --execute \
  --reassignment-json-file plan.json                         # thực thi kế hoạch
bin/kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --verify \
  --reassignment-json-file plan.json                         # kiểm tra tiến độ
bin/kafka-log-dirs.sh --bootstrap-server localhost:9092 --describe --broker-list 1   # dung lượng thư mục log theo broker
```

[Tên cờ cần kiểm tra theo phiên bản]

## 37. Kiểm tra và hiệu năng

```sh
bin/kafka-features.sh --bootstrap-server localhost:9092 describe      # tính năng và phiên bản được bật
bin/kafka-producer-perf-test.sh --topic thu --num-records 100000 --record-size 1024 \
  --throughput -1 --producer-props bootstrap.servers=localhost:9092 acks=all   # đo thông lượng ghi
bin/kafka-consumer-perf-test.sh --bootstrap-server localhost:9092 --topic thu \
  --messages 100000                                          # đo thông lượng đọc
```

[Tên công cụ hiệu năng và cờ cần kiểm tra theo phiên bản]

---

# PHẦN J. GỠ LỖI

## 38. Quy trình gỡ lỗi

```
1. Broker có sống không?                  kiểm tra tiến trình, cổng 9092, log broker
2. Metadata có ổn không?                  kafka-topics --describe (leader, ISR)
3. Partition có leader không?             --describe --unavailable-partitions
4. Replica có đủ không?                   --describe --under-replicated-partitions
5. Consumer có theo kịp không?            kafka-consumer-groups --describe (LAG)
6. Đĩa, retention, kích thước?            kafka-log-dirs, df -h trên broker
7. Controller và KRaft có khỏe không?     log controller, kafka-features, kafka-metadata-quorum
```

## 39. Không kết nối được broker

| Triệu chứng | Nguyên nhân thường gặp | Hướng xử lý |
|---|---|---|
| Bootstrap được, sau đó timeout | `advertised.listeners` sai | Đặt địa chỉ client phân giải được |
| `Connection refused` | Broker chưa chạy, sai cổng, tường lửa | Kiểm tra tiến trình, cổng, mạng |
| `SSL handshake failed` | Sai truststore, sai giao thức | Kiểm tra cấu hình TLS hai đầu |
| `SaslAuthenticationException` | Sai tài khoản hoặc cơ chế SASL | Kiểm tra `sasl.mechanism` và thông tin đăng nhập |
| Lỗi trong Docker/Kubernetes | Quảng bá địa chỉ nội bộ | Tách listener nội bộ và bên ngoài |

```sh
bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9092   # kiểm tra kết nối và phiên bản API
```

## 40. Producer lỗi hoặc chậm

| Lỗi | Nguyên nhân | Hướng xử lý |
|---|---|---|
| `NotEnoughReplicasException` | ISR nhỏ hơn `min.insync.replicas` | Khôi phục broker hoặc replica |
| `RecordTooLargeException` | Bản ghi vượt `max.message.bytes` hoặc `max.request.size` | Tăng giới hạn ở cả broker, topic, producer; hoặc chia nhỏ bản ghi |
| `TimeoutException` | Broker không phản hồi kịp | Kiểm tra tải, mạng, `delivery.timeout.ms` |
| Thông lượng thấp | Batch nhỏ, nén không bật | Điều chỉnh `linger.ms`, `batch.size`, `compression.type` |

## 41. Consumer lag và rebalance

```sh
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group nhom-xu-ly
# Cột LAG tăng liên tục: consumer chậm hơn producer
# Cột CONSUMER-ID trống: partition không có consumer
```

| Triệu chứng | Nguyên nhân | Hướng xử lý |
|---|---|---|
| Lag tăng dần | Xử lý chậm, ít consumer, số partition ít | Tối ưu xử lý, thêm consumer (không vượt số partition) |
| Rebalance liên tục | Consumer timeout, xử lý quá lâu giữa các lần poll | Điều chỉnh timeout, tách việc nặng |
| Lỗi `OffsetOutOfRange` | Offset đã bị xóa theo retention | Chọn `auto.offset.reset`, hoặc reset offset có chủ đích |
| Xử lý trùng | At least once, không idempotent | Thiết kế xử lý idempotent |

## 42. Under-replicated và offline partition

```sh
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --under-replicated-partitions
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --unavailable-partitions
```

**Under-replicated:** ISR nhỏ hơn số replica. Thường do broker chậm, đĩa đầy, hoặc mạng.
**Offline:** partition không có leader, không đọc hoặc ghi được. Nghiêm trọng hơn.

**Hướng xử lý:**
- Kiểm tra broker chứa replica: đã dừng, quá tải, hay đĩa đầy.
- Khôi phục broker, rồi kiểm tra ISR tự phục hồi.
- Không bật unclean election để "cứu" partition offline khi chưa cân nhắc mất dữ liệu. [Suy luận]

## 43. Đĩa, retention, và kích thước bản ghi

```sh
df -h                                                   # trên broker: dung lượng đĩa log.dirs
bin/kafka-log-dirs.sh --bootstrap-server localhost:9092 --describe --broker-list 1   # dung lượng theo topic
```

- Đĩa đầy: broker dừng ghi, partition có thể offline. Mở rộng đĩa hoặc giảm retention, và tránh để đến mức đầy. [Suy luận]
- Retention không xóa: kiểm tra `retention.ms`, `retention.bytes`, và topic có compaction không. [Trí nhớ]

## 44. KRaft và controller

```sh
bin/kafka-metadata-quorum.sh --bootstrap-server localhost:9092 describe --status    # trạng thái quorum controller [Trí nhớ]
bin/kafka-metadata-quorum.sh --bootstrap-server localhost:9092 describe --replication   # độ trễ replica metadata [Trí nhớ]
```

| Triệu chứng | Nguyên nhân | Hướng xử lý |
|---|---|---|
| Không có leader controller | Mất đa số controller | Khôi phục controller; không tự ý bỏ quorum |
| Metadata không cập nhật | Controller chậm hoặc mạng | Kiểm tra mạng, log controller |
| Lỗi định dạng thư mục metadata | `storage format` chưa chạy hoặc sai cluster ID | Kiểm tra cluster ID, định dạng đúng |

**Cảnh báo:** thay đổi thành viên quorum controller cần thao tác có kế hoạch. Đọc tài liệu KRaft của phiên bản đang dùng trước khi thực hiện. [Suy luận]

---

# PHẦN K. VẬN HÀNH VÀ GIÁM SÁT

## 45. Chỉ số cần theo dõi

Các chỉ số thường dùng (tên JMX có thể thay đổi theo phiên bản): [Trí nhớ]

| Chỉ số | Ý nghĩa | Cảnh báo khi |
|---|---|---|
| Under-replicated partitions | Partition thiếu replica | Khác 0 kéo dài |
| Offline partitions count | Partition không có leader | Khác 0 |
| Active controller count | Số controller hoạt động | Khác 1 (trong KRaft) |
| Consumer group lag | Độ trễ consumer | Tăng liên tục |
| Request latency (produce, fetch) | Độ trễ yêu cầu | Tăng bất thường |
| Bytes in/out per second | Lưu lượng | Gần giới hạn |
| Disk usage | Dung lượng đĩa | Trên 70–80% |
| Rebalance rate | Số lần rebalance | Cao bất thường |

## 46. Nâng cấp và bảo trì broker

- Nâng cấp **từng broker một**, đợi partition đồng bộ lại trước khi chuyển sang broker tiếp. [Suy luận]
- Trước khi dừng broker, kiểm tra `--under-replicated-partitions` không có partition nào đang thiếu. [Suy luận]
- Sau khởi động lại, chạy `kafka-leader-election.sh --election-type preferred` để đưa leader về vị trí cân bằng. [Trí nhớ]
- Đọc ghi chú nâng cấp của phiên bản, đặc biệt các thay đổi cấu hình và định dạng metadata. [Suy luận]
- Sao lưu cấu hình và thông tin cluster ID trước khi nâng cấp. [Suy luận]

---

# PHẦN L. SO SÁNH VỚI CÁC HỆ THỐNG KHÁC

| Tiêu chí | Kafka | Hàng đợi truyền thống (RabbitMQ, SQS) | Cơ sở dữ liệu |
|---|---|---|---|
| Mô hình | Nhật ký bền vững, nhiều consumer đọc độc lập | Hàng đợi, thường xóa khi đã nhận | Bảng, truy vấn |
| Đọc lại | Có (theo offset) | Thường không | Có (truy vấn) |
| Thứ tự | Trong partition | Tùy hệ thống | Tùy truy vấn |
| Thông lượng | Rất cao | Trung bình đến cao | Tùy |
| Thời gian lưu | Theo retention | Thường ngắn | Lâu dài |
| Độ phức tạp vận hành | Cao hơn | Thấp hơn | Trung bình |

[Suy luận và kiến thức nền; so sánh khái quát, không phải benchmark]

---

# PHẦN M. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Event / Record | Một sự kiện: key, value, timestamp, header |
| Topic | Luồng sự kiện có tên |
| Partition | Phân đoạn của topic, đơn vị song song và thứ tự |
| Offset | Số thứ tự bản ghi trong partition |
| Broker | Máy chủ lưu partition |
| Controller | Node đồng thuận metadata (KRaft) |
| KRaft | Cơ chế metadata bằng Raft, thay ZooKeeper |
| Replica / Leader / Follower | Bản sao / bản chính / bản sao chép |
| ISR | Tập replica đang bắt kịp leader |
| High watermark | Offset đã sao chép đủ, consumer đọc đến đây |
| `acks` | Mức xác nhận ghi của producer |
| `min.insync.replicas` | ISR tối thiểu khi ghi an toàn |
| Unclean leader election | Bầu leader từ replica ngoài ISR |
| Segment | Tệp chứa một đoạn log của partition |
| Retention | Thời gian hoặc dung lượng giữ bản ghi |
| Compaction | Giữ bản ghi mới nhất cho mỗi key |
| Consumer group | Nhóm consumer chia nhau partition |
| Rebalance | Phân bổ lại partition khi thành viên đổi |
| Lag | Khoảng cách offset giữa producer và consumer |
| Idempotence | Chống ghi trùng khi retry |
| Transaction | Ghi nguyên tử nhiều bản ghi, đọc với `read_committed` |
| `__consumer_offsets` | Topic nội bộ lưu offset đã commit |
| Advertised listener | Địa chỉ broker quảng bá cho client |

---

# PHẦN N. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## N.1. Nguồn đã đối chiếu trong lần này

- **Kafka Quickstart**: https://kafka.apache.org/quickstart [đã đọc. Ghi nhận: Kafka 4.3.1, yêu cầu Java 17+; khởi động KRaft bằng `kafka-storage.sh` và `kafka-server-start.sh`; các lệnh tạo topic, producer, consumer]
- **Kafka Introduction (Getting Started, bản 4.3)**: https://kafka.apache.org/43/getting-started/introduction/ [đã đọc. Ghi nhận: event streaming; publish, store, process; event, topic, partition, broker, replication; retention theo topic; key quyết định partition; thứ tự trong partition]
- **Trang tải về**: https://kafka.apache.org/downloads [trả về trang chuyển hướng, không đọc được danh sách phiên bản]

## N.2. Nguồn chưa đối chiếu

- Tài liệu Kafka đầy đủ (Design, Implementation, Operations): cấu hình broker, producer, consumer; ISR; KRaft chi tiết.
- Tài liệu cấu hình từng tham số (`min.insync.replicas`, `unclean.leader.election.enable`, `acks`, `auto.offset.reset`).
- Ghi chú phát hành của Kafka 4.0 đến 4.3 (gỡ ZooKeeper, thay đổi cấu hình).
- Tài liệu bảo mật (TLS, SASL, ACL) và tài liệu metrics JMX.
- Tài liệu công cụ `kafka-metadata-quorum`, `kafka-features`, `kafka-broker-api-versions`, `kafka-producer-perf-test`.

## N.3. Điểm cần xác minh

1. **Gỡ ZooKeeper từ Kafka 4.0:** kiến thức nền, cần đối chiếu ghi chú phát hành.
2. **Tên và cờ của các công cụ** (`kafka-metadata-quorum`, `kafka-features`, `kafka-log-dirs`, `kafka-reassign-partitions`, `kafka-leader-election`): cần kiểm tra theo phiên bản.
3. **Các tham số cấu hình** trong bảng mục 25 và cấu hình bảo mật mục 24: kiến thức nền.
4. **Tên và biến môi trường của image Docker** trong mục 28: cần kiểm tra trên tài liệu image chính thức. Cấu hình đó chỉ để thử nghiệm.
5. **Tên tham số JMX** và ngưỡng cảnh báo trong mục 45: có thể khác theo phiên bản và công cụ giám sát.
6. **Thông số về ngữ nghĩa giao nhận** (at most once, at least once, exactly once) và phạm vi exactly once: kiến thức nền, nên đối chiếu tài liệu chính thức.
7. **Các ví dụ lệnh và cấu hình chưa được chạy thử.** Nên thử trên cluster cục bộ trước khi dùng cho dữ liệu thật.
8. **Khuyến nghị về số partition, retention và đĩa** là hướng dẫn chung, cần đo trên tải thực tế.

---

**Tóm tắt để tra nhanh:**
- **Kafka** = nền tảng luồng sự kiện bền vững; sự kiện không bị xóa khi đọc, được giữ theo retention.
- **Kiến trúc:** broker lưu partition; controller KRaft quản lý metadata (không dùng ZooKeeper từ 4.x).
- **Đối tượng:** topic chia thành partition; mỗi bản ghi có offset; cùng key thì cùng partition, giữ thứ tự theo key.
- **Bền vững:** replication factor 3, `acks=all`, `min.insync.replicas=2`, tắt unclean election cho dữ liệu quan trọng.
- **Consumer:** group chia partition; commit offset; lag là chỉ số sức khỏe chính; rebalance cần kiểm soát.
- **Workflow:** thiết kế topic (key, partition, retention) → thử cục bộ → kiểm thử idempotent và sự cố → vận hành có kế hoạch.
- **Gỡ lỗi:** `kafka-topics --describe` (leader, ISR) → under-replicated/unavailable → consumer group lag → đĩa và metadata quorum.
- **Cẩn trọng:** không bật unclean leader election khi chưa cân nhắc mất dữ liệu, không tăng partition mà không tính đến thứ tự theo key, và không để đĩa đầy.
