# RabbitMQ: tài liệu tổng quan về mô hình, kiến trúc, đối tượng, cluster, workflow, câu lệnh, và gỡ lỗi

> **Viết cho:** người muốn hiểu RabbitMQ từ mô hình định tuyến và hàng đợi, đến cluster, độ bền, và xử lý sự cố. Cấu trúc tương tự tài liệu Kafka và Kubernetes để dễ đối chiếu.
> **Cách đọc:** Phần A đến C là mô hình. Phần D đến F là queue, exchange, và consumer. Phần G là cluster, bảo mật và cấu hình. Phần I là lệnh. Phần J là gỡ lỗi. Các cấu hình, lệnh và đoạn mã có chú thích từng dòng bên cạnh.
> **Phiên bản tham chiếu:** trang tài liệu chính thức (tra cứu 2026-10-21) ghi bản server mới nhất là **4.4.0**. Dòng phát hành gồm 4.4, 4.3, 4.2, 4.1, 4.0 và 3.13. Trang không ghi ngày phát hành và phiên bản Erlang/OTP tương ứng.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với rabbitmq.com; `[Trí nhớ]` = kiến thức nền, cần kiểm tra; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về mã:** các lệnh và cấu hình chưa được chạy thử trong lần này. Tên lệnh con của `rabbitmqctl` và `rabbitmq-diagnostics` đã thay đổi qua nhiều phiên bản. Kiểm tra `--help` trên đúng phiên bản bạn dùng.

---

## Mục lục

- **Phần A. RabbitMQ là gì và giải quyết vấn đề gì**
  - 1. Message broker và hàng đợi
  - 2. RabbitMQ làm được gì
  - 3. Phiên bản và dòng hỗ trợ
- **Phần B. Mô hình AMQP**
  - 4. Producer, consumer, connection, channel
  - 5. Exchange, binding, routing key
  - 6. Queue
  - 7. Message: body, property, header
- **Phần C. Kiến trúc cluster**
  - 8. Node, cluster, vhost
  - 9. Các loại queue: classic, quorum, stream
  - 10. Luồng một message đi qua hệ thống
- **Phần D. Exchange và định tuyến**
  - 11. Các loại exchange
  - 12. Dead-letter exchange và TTL
- **Phần E. Quorum queue (chi tiết)**
  - 13. Raft, nhóm thành viên, và đa số
  - 14. Giới hạn và điều cần biết
  - 15. Delivery limit và message độc hại (poison message)
- **Phần F. Độ tin cậy phía client**
  - 16. Publisher confirm
  - 17. Consumer acknowledgement và prefetch
  - 18. Ngữ nghĩa giao nhận và xử lý trùng
- **Phần G. Cluster, bảo mật, và cấu hình**
  - 19. Hình thành cluster và cookie
  - 20. Partition (phân mảnh mạng)
  - 21. Cảnh báo bộ nhớ và đĩa
  - 22. Bảo mật: người dùng, quyền, TLS
  - 23. Cấu hình quan trọng
- **Phần H. Workflow**
  - 24. Thiết kế topology
  - 25. Workflow phát triển cục bộ
  - 26. Quy trình vận hành và thay đổi
  - 27. Kiểm thử và CI
- **Phần I. Câu lệnh**
  - 28. Trạng thái và chẩn đoán
  - 29. Queue, exchange, và binding
  - 30. Connection và channel
  - 31. Người dùng, vhost, và quyền
  - 32. Policy và giới hạn
  - 33. Quorum queue
  - 34. Cluster
  - 35. Plugin và quản lý
- **Phần J. Gỡ lỗi (troubleshooting)**
  - 36. Quy trình gỡ lỗi
  - 37. Cảnh báo bộ nhớ hoặc đĩa (alarm, flow control)
  - 38. Hàng đợi tăng, consumer chậm
  - 39. Consumer không nhận message hoặc message bị kẹt
  - 40. Message bị lặp, message độc hại
  - 41. Kết nối và kênh rò rỉ
  - 42. Cluster: node không tham gia, partition, mất quorum
  - 43. Quorum queue không có đa số
  - 44. Xác thực và quyền
  - 45. TLS và mạng
  - 46. Crash dump của runtime
- **Phần K. Vận hành và giám sát**
  - 47. Chỉ số cần theo dõi
  - 48. Sao lưu và khôi phục
  - 49. Nâng cấp
- **Phần L. So sánh với Kafka**
- **Phần M. Thuật ngữ**
- **Phần N. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. RABBITMQ LÀ GÌ VÀ GIẢI QUYẾT VẤN ĐỀ GÌ

## 1. Message broker và hàng đợi

Khi hai dịch vụ trao đổi công việc trực tiếp, bên gửi phải chờ bên nhận sẵn sàng, và lỗi bên nhận làm mất yêu cầu. **Message broker** đứng giữa: bên gửi đặt message vào hàng đợi, bên nhận lấy ra khi có khả năng xử lý. Hai bên không cần biết nhau và không cần cùng chạy đúng lúc. [Suy luận]

**RabbitMQ** là message broker mã nguồn mở, triển khai giao thức AMQP 0-9-1 (và hỗ trợ thêm các giao thức khác qua plugin). [Trí nhớ]

## 2. RabbitMQ làm được gì

- Gửi công việc nền (background job) từ web đến worker.
- Phân phối message theo quy tắc định tuyến linh hoạt (topic, fan-out, direct).
- Đảm bảo message không mất khi consumer lỗi, nhờ acknowledgement và queue bền vững.
- Giới hạn tốc độ tiêu thụ (prefetch, flow control) để bảo vệ consumer.
- Quản lý qua giao diện web (management plugin) và API HTTP. [Trí nhớ]

**Khác với Kafka:** RabbitMQ thiên về **định tuyến và phân phối công việc**, message thường bị xóa sau khi được xác nhận. Kafka thiên về **nhật ký sự kiện** đọc lại được theo offset. Xem Phần L. [Suy luận]

## 3. Phiên bản và dòng hỗ trợ

Theo trang tải về chính thức: [Đã kiểm chứng]
- Bản server mới nhất trong tài liệu là **4.4.0**.
- Các dòng phát hành được liệt kê: 4.4, 4.3, 4.2, 4.1, 4.0, 3.13.
- Trang không ghi ngày phát hành và phiên bản Erlang/OTP yêu cầu. Cần xem trang "Erlang/OTP for RabbitMQ" và lịch hỗ trợ.

**Thay đổi lớn đáng chú ý** (theo tài liệu quorum queue đã đọc): [Đã kiểm chứng]
- Từ **4.0**, classic queue mirroring đã bị gỡ. Thay bằng quorum queue hoặc stream.
- Từ **4.0**, giới hạn redelivery mặc định (`delivery-limit`) là 20.
- Từ **4.3**, ưu tiên nghiêm ngặt 0–31 được bật sẵn cho quorum queue.

---

# PHẦN B. MÔ HÌNH AMQP

## 4. Producer, consumer, connection, channel

| Khái niệm | Vai trò |
|---|---|
| **Producer** | Ứng dụng gửi message |
| **Consumer** | Ứng dụng nhận và xử lý message |
| **Connection** | Kết nối TCP dài hạn từ ứng dụng đến broker |
| **Channel** | Luồng ảo nhẹ chạy trên một connection, dùng cho thao tác. Nên dùng nhiều channel thay vì nhiều connection |

[Trí nhớ; mô hình AMQP]

**Lưu ý:** mở một connection cho mỗi thao tác rất tốn tài nguyên. Giữ connection lâu dài, dùng channel để song song. [Suy luận]

## 5. Exchange, binding, routing key

Message không gửi trực tiếp vào queue mà gửi đến **exchange**. Exchange dùng **binding** (quy tắc liên kết exchange với queue) và **routing key** (nhãn của message) để quyết định queue nào nhận message. [Trí nhớ]

```
Producer ──► Exchange ──(binding + routing key)──► Queue A ──► Consumer A
                                              └──► Queue B ──► Consumer B
```

## 6. Queue

Queue là nơi message chờ. Các thuộc tính chính: tên, tính bền vững (`durable`), tính độc quyền (`exclusive`), tự xóa (`auto-delete`), và kiểu queue (classic, quorum, stream). [Trí nhớ]

## 7. Message: body, property, header

- **Body:** nội dung (bytes). Thường là JSON, Protobuf, hoặc văn bản.
- **Properties:** `delivery_mode` (2 = bền vững), `content_type`, `message_id`, `correlation_id`, `expiration`, `priority`. [Trí nhớ]
- **Headers:** cặp khóa-giá trị tùy chỉnh. Dùng cho trace và metadata. [Trí nhớ]

---

# PHẦN C. KIẾN TRÚC CLUSTER

```
┌──────────────────────── CLUSTER RABBITMQ ────────────────────────┐
│  Node 1 (rabbit@n1)     Node 2 (rabbit@n2)     Node 3 (rabbit@n3) │
│  ┌──────────────┐       ┌──────────────┐       ┌──────────────┐   │
│  │ vhost "/"    │◄─────►│ vhost "/"    │◄─────►│ vhost "/"    │   │
│  │ exchanges    │  Erlang distribution (cookie)  │              │   │
│  │ queues: Q1 ◉ │       │ Q1 follower  │       │ Q1 follower  │   │
│  │ (quorum)     │       │              │       │              │   │
│  └──────────────┘       └──────────────┘       └──────────────┘   │
│  Metadata (exchange, queue, binding, user) được đồng bộ giữa node │
└──────────────────────────────────────────────────────────────────┘
```

## 8. Node, cluster, vhost

- **Node:** một tiến trình RabbitMQ (chạy trên Erlang/OTP). [Trí nhớ]
- **Cluster:** nhóm node chia sẻ metadata (exchange, queue, binding, user, vhost). [Trí nhớ]
- **Vhost (virtual host):** không gian cô lập trong broker, có quyền và tài nguyên riêng. Vhost mặc định là `/`. [Trí nhớ]

**Lưu ý:** nội dung queue **không** tự động sao chép giữa các node với queue classic. Với queue cần sẵn sàng cao, dùng quorum queue hoặc stream (mục 9). [Trí nhớ]

## 9. Các loại queue: classic, quorum, stream

| Loại | Đặc điểm | Dùng khi |
|---|---|---|
| **Classic** | Một node chủ, không sao chép (classic mirroring đã gỡ từ 4.0) | Queue tạm, không cần sẵn sàng cao, độ trễ thấp nhất |
| **Quorum** | Sao chép bằng Raft, bền vững, mặc định khuyến nghị cho dữ liệu quan trọng | Hàng đợi lâu dài, cần sẵn sàng cao |
| **Stream** | Nhật ký bền vững, đọc lại theo offset, nhiều consumer đọc độc lập | Fan-out lớn, đọc lại, luồng sự kiện |

[Đã kiểm chứng: tài liệu quorum queue khuyến nghị quorum queue cho hàng đợi lâu dài, và nêu streams là cấu trúc sao chép khác; classic mirroring đã bị gỡ từ 4.0]

**Quy tắc chọn:** queue tạm, exclusive, hoặc cần độ trễ tối thiểu thì classic phù hợp hơn. Đa số trường hợp sản xuất cần độ bền thì chọn quorum. [Đã kiểm chứng một phần]

## 10. Luồng một message đi qua hệ thống

```
1. Producer mở channel, khai báo exchange/queue (nếu chưa có)
2. Producer publish message kèm routing key
3. Exchange tra binding, đưa message vào các queue khớp
4. Với quorum queue: leader sao chép đến đa số thành viên, rồi broker gửi publisher confirm
5. Consumer đăng ký (basic.consume) với prefetch giới hạn
6. Broker gửi message cho consumer
7. Consumer xử lý, gửi ack (hoặc nack/reject)
8. Broker xóa message khỏi queue khi đã ack; nếu nack/reject, xử lý theo cấu hình (requeue, DLX)
```

[Đã kiểm chứng về publisher confirm trong quorum queue; các bước còn lại là mô hình AMQP chuẩn]

---

# PHẦN D. EXCHANGE VÀ ĐỊNH TUYẾN

## 11. Các loại exchange

| Loại | Cách định tuyến | Ví dụ |
|---|---|---|
| **direct** | Khớp chính xác routing key | `don-hang.thanh-toan` → queue thanh toán |
| **topic** | Khớp mẫu có ký tự đại diện (`*` một từ, `#` nhiều từ) | `don-hang.*` nhận `don-hang.da-dat` |
| **fanout** | Gửi đến mọi queue được bind, bỏ qua routing key | Thông báo đến nhiều dịch vụ |
| **headers** | Khớp theo header | Ít dùng hơn [Trí nhớ] |

```sh
rabbitmqadmin declare exchange name=don-hang type=topic      # khai báo exchange kiểu topic (cần plugin management)
rabbitmqadmin declare binding source=don-hang destination=thanh-toan routing_key="don-hang.#"
```

[Trí nhớ; rabbitmqadmin cần plugin rabbitmq_management]

## 12. Dead-letter exchange và TTL

- **Dead-letter exchange (DLX):** nơi chuyển message khi bị từ chối, hết hạn (TTL), vượt giới hạn độ dài, hoặc vượt delivery limit. [Trí nhớ; đã kiểm chứng với quorum queue]
- **TTL:** thời gian sống của message (`expiration` hoặc `x-message-ttl`) hoặc của queue (`x-expires`). [Trí nhớ]

```
Queue chính "don-hang" ──(reject/hết hạn)──► DLX "don-hang.dlx" ──► Queue "don-hang.that-bai"
```

**Lưu ý:** queue TTL có hạn chế với quorum queue (mục 14). [Đã kiểm chứng]

---

# PHẦN E. QUORUM QUEUE (CHI TIẾT)

## 13. Raft, nhóm thành viên, và đa số

Quorum queue dùng **Raft** để sao chép log. Mỗi quorum queue có một nhóm thành viên (member), mỗi thành viên nằm trên một node khác nhau. [Đã kiểm chứng]

- **Kích thước nhóm mặc định:** 3, đặt bằng `x-quorum-initial-group-size`. Khuyến nghị số lẻ. [Đã kiểm chứng]
- **Đa số:** `(N/2)+1`. Nhóm 3 chịu được 1 lỗi, nhóm 5 chịu được 2. [Đã kiểm chứng]
- **Số node:** để khai báo nhóm 3, cần ít nhất 2 node đang chạy. [Đã kiểm chứng]
- **Giới hạn kích thước:** hiệu năng giảm với nhóm lớn hơn 5; tài liệu khuyến nghị không quá 7 node. [Đã kiểm chứng]
- **Publisher confirm:** chỉ được gửi sau khi đa số nhóm đã sao chép message. [Đã kiểm chứng]

**Khai báo quorum queue** (AMQP, qua tham số đối số):

```python
import pika                                                   # thư viện Python cho AMQP 0-9-1

conn = pika.BlockingConnection(pika.ConnectionParameters("localhost"))   # kết nối đến broker
ch = conn.channel()                                           # mở channel
ch.queue_declare(
    queue="don-hang",                                         # tên queue
    durable=True,                                             # bền vững (quorum luôn bền vững)
    arguments={"x-queue-type": "quorum",                      # chọn kiểu quorum
               "x-quorum-initial-group-size": 3},             # nhóm 3 thành viên
)
```

[Đã kiểm chứng: tên đối số `x-queue-type` và `x-quorum-initial-group-size`; API pika cần kiểm tra phiên bản]

## 14. Giới hạn và điều cần biết

Theo tài liệu quorum queue: [Đã kiểm chứng]
- **Luôn bền vững:** không hỗ trợ queue không bền vững.
- **Không exclusive:** dùng single active consumer thay cho exclusive consumer.
- **Không hỗ trợ global QoS:** dùng prefetch theo consumer. Đặt global QoS sẽ gây lỗi kênh.
- **Ưu tiên:** từ 4.3, ưu tiên nghiêm ngặt 0–31 luôn bật; `x-max-priority` bị bỏ qua.
- **TTL của queue:** hỗ trợ một phần. Mỗi message có TTL tốn thêm 16 byte bộ nhớ.
- **Giới hạn độ dài:** hỗ trợ `drop-head` và `reject-publish`, không hỗ trợ `reject-publish-dlx`.
- **Bộ nhớ:** mỗi message cần ít nhất 32 byte chỉ mục trong bộ nhớ. Tài liệu ước lượng ít nhất 1 MB cho mỗi 30.000 message. Nên giữ queue ngắn và đặt giới hạn độ dài.
- **Kiểu queue** chỉ đặt khi khai báo, không đổi qua policy.
- **Không dùng cho** queue tạm, khai báo/xóa liên tục, backlog 5 triệu message trở lên, hoặc fan-out lớn (dùng stream). [Đã kiểm chứng]

## 15. Delivery limit và message độc hại (poison message)

**Poison message:** message làm consumer thất bại liên tục, bị requeue và lặp mãi. [Trí nhớ]

Quorum queue theo dõi số lần giao sai qua header `x-delivery-count`. [Đã kiểm chứng]
- **Mặc định** `delivery-limit` là 20 (từ 4.0). Vượt giới hạn, message bị loại bỏ, hoặc chuyển đến DLX nếu có.
- Từ 4.3, giới hạn đếm theo `delivery-count`. `nack` và `released` thông thường không tính, `reject` và crash kênh thì tính. [Đã kiểm chứng]
- Đặt bằng policy key `delivery-limit`. `-1` tắt giới hạn, tài liệu không khuyến nghị.
- `dead-letter-strategy`: `at-least-once` cần `overflow` là `reject-publish`. Mặc định là `at-most-once`. [Đã kiểm chứng]
- Từ 4.3, `x-delayed-retry-type`, `delayed-retry-min`/`-max` thêm backoff tuyến tính cho message requeue. [Đã kiểm chứng]

```sh
rabbitmqctl set_policy dlx-don-hang "^don-hang$" \
  '{"delivery-limit":5,"dead-letter-exchange":"don-hang.dlx"}' --apply-to queues
# Đặt giới hạn 5 lần giao sai, chuyển sang DLX khi vượt
```

[Cú pháp `set_policy` và tham số JSON cần kiểm tra theo phiên bản; tên key đã được đối chiếu]

---

# PHẦN F. ĐỘ TIN CẬY PHÍA CLIENT

## 16. Publisher confirm

Publisher confirm là cơ chế broker báo cho producer biết message đã được ghi nhận (với quorum queue: đã sao chép đến đa số). Không có confirm, producer không biết message có được nhận hay không. [Đã kiểm chứng]

```python
ch.confirm_delivery()                                          # bật publisher confirm trên channel
ch.basic_publish(
    exchange="",                                               # exchange mặc định, định tuyến theo tên queue
    routing_key="don-hang",                                    # gửi đến queue don-hang
    body=b'{"ma":101}',                                        # nội dung message
    properties=pika.BasicProperties(delivery_mode=2),          # 2 = persistent (bền vững)
    mandatory=True,                                            # báo lỗi nếu không có queue nhận
)
# Nếu broker từ chối hoặc không xác nhận, pika ném ngoại lệ (cần xử lý và thử lại)
```

[Tên tham số pika cần kiểm tra theo phiên bản]

## 17. Consumer acknowledgement và prefetch

- **`ack`:** xác nhận đã xử lý xong, broker xóa message.
- **`nack` / `reject`:** từ chối, có thể yêu cầu requeue hoặc chuyển DLX.
- **Prefetch (`basic_qos`):** số message tối đa chưa ack mà consumer nhận cùng lúc. Đặt giá trị hợp lý để không dồn message vào consumer chậm. [Trí nhớ]

```python
ch.basic_qos(prefetch_count=10)                               # tối đa 10 message chưa ack mỗi consumer
def xu_ly(channel, method, props, body):                      # hàm callback
    try:
        ...                                                   # xử lý nghiệp vụ
        channel.basic_ack(delivery_tag=method.delivery_tag)   # xác nhận thành công
    except Exception:
        channel.basic_nack(delivery_tag=method.delivery_tag, requeue=False)   # từ chối, không requeue (đi DLX)
ch.basic_consume(queue="don-hang", on_message_callback=xu_ly, auto_ack=False)   # tắt auto ack
ch.start_consuming()
```

**Cảnh báo:** `auto_ack=True` xóa message ngay khi gửi đi. Nếu consumer crash, message mất. [Trí nhớ]

## 18. Ngữ nghĩa giao nhận và xử lý trùng

- **At most once:** auto ack hoặc gửi không confirm; có thể mất.
- **At least once:** ack sau khi xử lý xong; có thể xử lý trùng nếu consumer crash sau khi xử lý nhưng trước khi ack.
- **Exactly once:** không có sẵn từ broker; đạt bằng xử lý idempotent (kiểm tra message ID đã xử lý trong cơ sở dữ liệu). [Suy luận]

**Khuyến nghị:** thiết kế consumer idempotent, dùng `message_id` làm khóa khử trùng. [Suy luận]

---

# PHẦN G. CLUSTER, BẢO MẬT, VÀ CẤU HÌNH

## 19. Hình thành cluster và cookie

- Các node giao tiếp qua Erlang distribution và xác thực bằng **Erlang cookie**. Mọi node trong cluster phải có cùng cookie. [Trí nhớ]
- Cookie là bí mật: không đưa vào repo, quyền file hạn chế (`~/.erlang.cookie` hoặc `/var/lib/rabbitmq/.erlang.cookie`). [Trí nhớ]
- Node mới tham gia bằng `join_cluster`. Hostname phải phân giải được giữa các node. [Trí nhớ]

```sh
# Trên node mới (rabbit@n2) tham gia cluster có rabbit@n1
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@n1
rabbitmqctl start_app
rabbitmqctl cluster_status
```

[Trí nhớ; tên lệnh con cần kiểm tra theo phiên bản]

## 20. Partition (phân mảnh mạng)

Khi mạng giữa các node bị đứt, cluster có thể bị **partition**: mỗi nhóm nghĩ nhóm kia đã chết. Cấu hình `cluster_partition_handling` quyết định hành vi: [Trí nhớ]
- `autoheal`: chọn một bên thắng, bên kia tự khởi động lại.
- `pause_minority`: nhóm thiểu số tự tạm dừng.
- `ignore`: giữ nguyên, nguy cơ phân kỳ dữ liệu.

**Khuyến nghị:** dùng số node lẻ và `pause_minority` cho cluster cần nhất quán. [Suy luận]

## 21. Cảnh báo bộ nhớ và đĩa

RabbitMQ đặt **alarm** và chặn publisher (flow control) khi: [Trí nhớ]
- **Bộ nhớ** vượt `vm_memory_high_watermark` (mặc định là một phần RAM).
- **Đĩa trống** thấp hơn `disk_free_limit`.

Khi alarm bật, các connection publish bị chặn, và trạng thái `blocked` xuất hiện trong management. Đây là cơ chế bảo vệ, không phải lỗi thông thường. [Trí nhớ]

## 22. Bảo mật: người dùng, quyền, TLS

- **Người dùng và tag:** `administrator`, `monitoring`, `management`, `policymaker`. Mặc định có người dùng `guest` chỉ cho phép kết nối từ localhost. [Trí nhớ]
- **Quyền:** cấu hình, ghi, đọc theo regex trên từng vhost. [Trí nhớ]
- **TLS:** mã hóa AMQP và management API, cấu hình trong `rabbitmq.conf`. [Trí nhớ]

```sh
rabbitmqctl add_user app 'mat-khau-manh'
rabbitmqctl set_user_tags app management
rabbitmqctl set_permissions -p / app "^app-.*" "^app-.*" "^app-.*"   # cấu hình, ghi, đọc
rabbitmqctl delete_user guest                                         # xóa tài khoản mặc định khi triển khai thật
```

[Đã kiểm chứng ở mức khái niệm; cú pháp cần kiểm tra theo phiên bản]

## 23. Cấu hình quan trọng

File cấu hình chính: `rabbitmq.conf` (định dạng khóa = giá trị). [Trí nhớ]

```ini
# rabbitmq.conf (ví dụ, cần kiểm tra theo phiên bản)
listeners.tcp.default = 5672              # cổng AMQP
management.tcp.port = 15672              # cổng giao diện quản trị (cần plugin management)
vm_memory_high_watermark.relative = 0.6  # ngưỡng alarm bộ nhớ: 60% RAM
disk_free_limit.absolute = 2GB           # cảnh báo khi đĩa trống dưới 2 GB
cluster_partition_handling = pause_minority   # xử lý partition
default_user = app                       # người dùng mặc định (đừng để guest khi triển khai)
```

| Tham số | Ý nghĩa |
|---|---|
| `vm_memory_high_watermark` | Ngưỡng bộ nhớ kích hoạt alarm |
| `disk_free_limit` | Dung lượng đĩa trống tối thiểu |
| `cluster_partition_handling` | Hành vi khi partition |
| `loopback_users` | Người dùng được kết nối từ localhost |
| `consumer_timeout` | Thời gian chờ ack trước khi trả message về queue (quorum/classic) [Trí nhớ] |

[Tên và cú pháp cần kiểm tra trong tài liệu cấu hình của phiên bản đang dùng]

---

# PHẦN H. WORKFLOW

## 24. Thiết kế topology

Trước khi tạo exchange và queue, trả lời:
1. Message là sự kiện, lệnh, hay công việc? Sự kiện thì cân nhắc stream; công việc thì queue.
2. Queue cần bền vững không? Đa số dữ liệu quan trọng: quorum.
3. Định tuyến theo gì? Chọn loại exchange tương ứng.
4. Xử lý thất bại thế nào? Thiết kế DLX và delivery limit từ đầu.
5. Mỗi consumer xử lý với tốc độ nào? Đặt prefetch và số consumer tương ứng.
6. Có cần tách vhost theo môi trường hoặc đội không?

## 25. Workflow phát triển cục bộ

```yaml
# docker-compose.yml (minh họa, một node có giao diện quản trị)
services:
  rabbitmq:
    image: rabbitmq:4.4-management         # image chính thức có sẵn plugin management (kiểm tra tag thật)
    ports:
      - "5672:5672"                        # AMQP cho ứng dụng
      - "15672:15672"                      # giao diện quản trị
    environment:
      RABBITMQ_DEFAULT_USER: dev           # tài khoản thử nghiệm
      RABBITMQ_DEFAULT_PASS: dev-only      # chỉ dùng cục bộ
    volumes:
      - rabbit-data:/var/lib/rabbitmq      # giữ dữ liệu giữa các lần chạy
volumes:
  rabbit-data:
```

```sh
docker compose up -d                       # khởi động
# Giao diện: http://localhost:15672 (dev / dev-only)
```

[Tên image và biến môi trường cần kiểm tra trên trang image chính thức. Đây chỉ là cấu hình thử nghiệm]

## 26. Quy trình vận hành và thay đổi

- Thay đổi policy và queue qua công cụ có phiên bản (script hoặc IaC), không chỉ qua giao diện.
- Thay đổi có kế hoạch: thông báo đội consumer khi đổi định tuyến, kiểu queue, hoặc DLX.
- Kiểm tra trạng thái trước và sau thay đổi (mục 28).
- Lưu cấu hình định nghĩa (definitions) định kỳ để khôi phục (mục 48).

## 27. Kiểm thử và CI

- **Kiểm thử đơn vị:** tách logic xử lý message khỏi I/O AMQP, kiểm thử bằng dữ liệu giả.
- **Kiểm thử tích hợp:** chạy RabbitMQ thật trong container (Testcontainers hoặc Compose), kiểm tra publish, route, consume, DLX.
- **Kiểm thử hành vi thất bại:** consumer nack, message vượt delivery limit, DLX nhận đúng.
- **Kiểm thử idempotent:** gửi trùng cùng `message_id`, xác nhận kết quả không đổi.

---

# PHẦN I. CÂU LỆNH

Công cụ chính: `rabbitmqctl` (quản trị node và cluster), `rabbitmq-diagnostics` (chẩn đoán), `rabbitmq-queues` (quản lý quorum queue), `rabbitmq-plugins` (plugin), `rabbitmqadmin` (qua HTTP API, cần plugin management).

## 28. Trạng thái và chẩn đoán

```sh
rabbitmqctl status                              # tổng quan node: phiên bản, bộ nhớ, đĩa, listener
rabbitmqctl cluster_status                      # thành viên cluster và trạng thái
rabbitmqctl environment                         # cấu hình hiệu lực của node [Đã kiểm chứng]
rabbitmq-diagnostics check_running              # node có đang chạy đầy đủ không
rabbitmq-diagnostics check_local_alarms         # alarm cục bộ (bộ nhớ, đĩa)
rabbitmq-diagnostics check_port_connectivity    # các cổng có mở không
rabbitmq-diagnostics memory_breakdown           # phân rã bộ nhớ theo thành phần
rabbitmq-diagnostics environment                # cấu hình (giống environment)
rabbitmq-diagnostics observer                   # giao diện xem tiến trình (cần GUI) [Trí nhớ]
```

[Tên lệnh con đã đối chiếu một phần; cú pháp chi tiết cần kiểm tra theo phiên bản]

## 29. Queue, exchange, và binding

```sh
rabbitmqctl list_queues name type messages consumers memory          # queue, kiểu, số message, consumer, bộ nhớ
rabbitmqctl list_exchanges name type                                 # exchange
rabbitmqctl list_bindings                                            # binding
rabbitmqctl purge_queue <tên-queue>                                  # xóa toàn bộ message (không hoàn tác)
rabbitmqctl delete_queue <tên-queue>                                 # xóa queue
```

**Cảnh báo:** `purge_queue` và `delete_queue` xóa message và không hoàn tác. Kiểm tra số message và xác nhận trước khi chạy. [Suy luận]

## 30. Connection và channel

```sh
rabbitmqctl list_connections name user state channels                # kết nối đang mở
rabbitmqctl list_channels connection name consumer_count messages_unacknowledged   # kênh
rabbitmqctl list_consumers                                           # consumer và prefetch
rabbitmqctl close_connection "<pid-hoặc-tên>" "lý do"                # đóng kết nối cụ thể
```

## 31. Người dùng, vhost, và quyền

```sh
rabbitmqctl list_users                                # người dùng và tag
rabbitmqctl add_user <tên> <mật-khẩu>                 # thêm người dùng
rabbitmqctl change_password <tên> <mật-khẩu>
rabbitmqctl set_user_tags <tên> <tag>                 # gán tag (administrator, monitoring, ...)
rabbitmqctl list_vhosts                               # vhost
rabbitmqctl add_vhost <tên>                           # tạo vhost
rabbitmqctl list_permissions -p /                     # quyền trên vhost
rabbitmqctl set_permissions -p <vhost> <user> <cấu-hình> <ghi> <đọc>
rabbitmqctl list_user_permissions <user>              # quyền của người dùng
```

## 32. Policy và giới hạn

```sh
rabbitmqctl list_policies                                          # policy hiện có
rabbitmqctl set_policy <tên> "<regex>" '<json>' --apply-to queues  # đặt policy cho queue
rabbitmqctl clear_policy <tên>                                     # xóa policy
rabbitmqctl set_vhost_limits -p <vhost> '{"max-connections": 100, "max-queues": 200}'   # giới hạn vhost [Trí nhớ]
rabbitmqctl set_user_limits <user> '{"max-connections": 10}'        # giới hạn người dùng [Trí nhớ]
```

## 33. Quorum queue

```sh
rabbitmq-queues check_if_node_is_quorum_critical     # node này có phải thành viên quan trọng không
rabbitmq-queues check_if_node_is_mirror_sync_critical  # tên cũ, có thể đổi (kiểm tra phiên bản) [Trí nhớ]
rabbitmq-queues peek <tên-queue> 5                   # xem 5 message đầu [Trí nhớ]
rabbitmq-queues add_member <tên-queue> rabbit@n3     # thêm thành viên vào nhóm
rabbitmq-queues delete_member <tên-queue> rabbit@n3  # bỏ thành viên
rabbitmq-queues rebalance all                        # cân bằng leader giữa các node [Trí nhớ]
rabbitmq-queues grow rabbit@n3 all --vhost-pattern ".*" --queue-pattern ".*"   # mở rộng nhóm sang node mới [Trí nhớ]
```

[Tên lệnh con và cờ thay đổi qua phiên bản. Đọc `--help` trước khi chạy]

## 34. Cluster

```sh
rabbitmqctl join_cluster rabbit@n1          # tham gia cluster (sau stop_app)
rabbitmqctl stop_app                        # dừng ứng dụng RabbitMQ, giữ node
rabbitmqctl start_app                       # khởi động lại ứng dụng
rabbitmqctl forget_cluster_node rabbit@n3   # gỡ node đã hỏng khỏi metadata cluster
rabbitmqctl await_online_nodes 2            # chờ đủ 2 node online (dùng trong script) [Trí nhớ]
rabbitmqctl reset                           # reset node về trạng thái trống (xóa dữ liệu; thận trọng)
```

**Cảnh báo:** `reset` và `forget_cluster_node` có thể mất dữ liệu hoặc làm cluster mất đồng bộ. Chỉ dùng khi đã hiểu tác động. [Trí nhớ]

## 35. Plugin và quản lý

```sh
rabbitmq-plugins list                                  # plugin có sẵn và trạng thái
rabbitmq-plugins enable rabbitmq_management            # bật giao diện quản trị và HTTP API
rabbitmq-plugins disable rabbitmq_management
rabbitmq-plugins enable rabbitmq_prometheus            # bật endpoint metrics cho Prometheus [Trí nhớ]
```

---

# PHẦN J. GỠ LỖI

## 36. Quy trình gỡ lỗi

```
1. Node có sống và đầy đủ không?      rabbitmq-diagnostics check_running
2. Có alarm không?                    rabbitmq-diagnostics check_local_alarms (và trạng thái blocked trên management)
3. Cluster có đủ thành viên không?    rabbitmqctl cluster_status
4. Queue nào bất thường?              rabbitmqctl list_queues name type messages consumers
5. Consumer có nhận không?            rabbitmqctl list_consumers, list_channels (unacked)
6. Có kết nối/kênh rò rỉ không?       rabbitmqctl list_connections, list_channels
7. Log nói gì?                        log của node (xem đường dẫn trong rabbitmqctl environment)
```

## 37. Cảnh báo bộ nhớ hoặc đĩa (alarm, flow control)

**Triệu chứng:** publisher bị chặn (`blocked`), tốc độ ghi về 0, log có cảnh báo về memory hoặc disk alarm. [Trí nhớ]

```sh
rabbitmq-diagnostics check_local_alarms                # alarm đang bật
rabbitmq-diagnostics memory_breakdown                  # thành phần nào dùng nhiều bộ nhớ
rabbitmqctl list_queues name messages memory | sort -k3 -n -r | head   # queue tốn bộ nhớ nhất [Suy luận]
df -h                                                  # đĩa trên node
```

| Nguyên nhân | Hướng xử lý |
|---|---|
| Backlog lớn, message nằm trong bộ nhớ | Tiêu thụ nhanh hơn, thêm consumer, hoặc purge có kiểm soát (sau khi xác nhận nghiệp vụ) |
| Message lớn | Đưa payload lớn ra object storage, message chỉ chứa tham chiếu |
| Đĩa đầy | Giải phóng đĩa, mở rộng đĩa, kiểm tra log và dữ liệu cũ |
| Ngưỡng quá thấp so với RAM | Điều chỉnh `vm_memory_high_watermark` có cân nhắc |

**Lưu ý:** không tắt alarm để "chạy tiếp". Alarm là cơ chế bảo vệ dữ liệu. [Suy luận]

## 38. Hàng đợi tăng, consumer chậm

```sh
rabbitmqctl list_queues name messages_ready messages_unacknowledged consumers   # ready: chờ; unacked: đang xử lý
rabbitmqctl list_consumers                                                    # prefetch và trạng thái
```

| Triệu chứng | Nguyên nhân | Hướng xử lý |
|---|---|---|
| `messages_ready` tăng | Consumer không đủ hoặc chậm | Thêm consumer, tối ưu xử lý |
| `messages_unacknowledged` cao, không giảm | Consumer nhận nhiều nhưng chậm ack | Giảm `prefetch_count`, xem nghẽn trong xử lý |
| `consumers = 0` | Consumer chết hoặc không kết nối | Kiểm tra ứng dụng, mạng, và restart |
| Prefetch quá lớn | Một consumer giữ hết message | Giảm prefetch để phân phối đều |

## 39. Consumer không nhận message hoặc message bị kẹt

- **Queue không có consumer:** kiểm tra `consumers` trong `list_queues`.
- **Message kẹt ở `unacknowledged`:** consumer nhận nhưng chưa ack, hoặc crash. Khi kết nối đóng, message được trả về queue. [Trí nhớ]
- **Consumer timeout:** nếu không ack trong thời gian quy định, broker trả message về queue (cấu hình `consumer_timeout` hoặc `x-consumer-timeout`). [Trí nhớ; tài liệu quorum queue có nhắc `x-consumer-timeout`]
- **Routing sai:** message không vào queue mong muốn. Kiểm tra binding và routing key:

```sh
rabbitmqctl list_bindings source_name destination_name routing_key      # xem binding
```

- Với publish `mandatory=true`, broker trả về message nếu không có queue nhận. Ứng dụng cần xử lý sự kiện return. [Trí nhớ]

## 40. Message bị lặp, message độc hại

**Dấu hiệu:** cùng một message được xử lý lặp đi lặp lại, `x-delivery-count` tăng dần. [Đã kiểm chứng: header `x-delivery-count` được tài liệu quorum queue nêu]

```sh
rabbitmqctl list_queues name messages_unacknowledged    # xem queue có message bị kẹt không
rabbitmq-queues peek <tên-queue> 1                      # xem một message đầu (kiểm tra header) [Trí nhớ]
```

**Hướng xử lý:**
- Đặt `delivery-limit` và DLX (mục 15) để message độc hại không lặp mãi.
- Consumer phân loại lỗi: lỗi tạm thời thì nack có requeue, lỗi dữ liệu thì reject không requeue (chuyển DLX).
- Sửa lỗi nghiệp vụ, rồi đưa message từ DLX về queue có kiểm soát (shovel hoặc script). [Suy luận]

## 41. Kết nối và kênh rò rỉ

```sh
rabbitmqctl list_connections name user state channels client_properties | head -n 50
rabbitmqctl list_channels connection number consumer_count
```

**Dấu hiệu rò rỉ:** số connection hoặc channel tăng mãi, không giảm khi tải ổn định. Thường do ứng dụng mở connection mới cho mỗi thao tác hoặc không đóng channel. [Trí nhớ]

**Hướng xử lý:**
- Tái sử dụng connection, dùng channel để song song.
- Đảm bảo đóng channel và connection khi xong (khối `finally` hoặc context manager).
- Đặt giới hạn `max-connections` và `max-channels` cho user hoặc vhost để phát hiện sớm. [Trí nhớ]

## 42. Cluster: node không tham gia, partition, mất quorum

| Triệu chứng | Nguyên nhân thường gặp | Hướng xử lý |
|---|---|---|
| `join_cluster` lỗi `badcookie` | Erlang cookie khác nhau | Đồng bộ cookie, khởi động lại |
| Không kết nối được node khác | Hostname không phân giải, cổng 4369, 25672 bị chặn | Kiểm tra DNS, tường lửa |
| Node không khởi động lại sau partition | Chế độ xử lý partition | Đọc cấu hình, khôi phục theo đúng chế độ |
| Node cũ hỏng, không còn | Metadata cluster còn trỏ đến node đó | `forget_cluster_node` sau khi chắc chắn node đã mất |

**Ghi chú:** cổng 4369 (epmd) và 25672 (distribution) là cổng nội bộ giữa các node. [Trí nhớ]

```sh
rabbitmqctl cluster_status            # kiểm tra node nào đang thuộc cluster và đang chạy
rabbitmq-diagnostics check_port_connectivity    # kiểm tra cổng
```

## 43. Quorum queue không có đa số

**Triệu chứng:** publish hoặc consume bị lỗi, hoặc queue báo không sẵn sàng, khi số node đang chạy dưới đa số của nhóm. [Đã kiểm chứng: đa số là `(N/2)+1`]

```sh
rabbitmq-queues check_if_node_is_quorum_critical       # node có đang là thành viên quan trọng không [Trí nhớ]
rabbitmqctl cluster_status                             # node nào đang online
```

**Hướng xử lý:**
- Khôi phục node đã hỏng nếu có thể. Đây là cách an toàn nhất.
- Nếu không khôi phục được: thêm node mới vào nhóm (`add_member`) và gỡ thành viên hỏng (`delete_member`), theo tài liệu quorum queue của phiên bản đang dùng. [Trí nhớ]
- Với nhóm 3 thành viên và hỏng 2 node, không có đa số, thao tác phục hồi có rủi ro mất dữ liệu. Cần đọc kỹ tài liệu trước khi làm. [Suy luận]

## 44. Xác thực và quyền

| Lỗi | Nguyên nhân | Hướng xử lý |
|---|---|---|
| `ACCESS_REFUSED - Login was refused` | Sai mật khẩu, người dùng không tồn tại | Kiểm tra `list_users`, đổi mật khẩu |
| `ACCESS_REFUSED` từ máy khác với `guest` | `guest` chỉ cho localhost | Tạo người dùng khác |
| `access to queue refused` | Thiếu quyền đọc/ghi trên queue | `list_user_permissions`, `set_permissions` |
| `NOT_ALLOWED - vhost not found` | Sai tên vhost hoặc người dùng không có quyền vhost | `list_vhosts`, cấp quyền |
| CLI không kết nối node | Thiếu cookie hoặc sai quyền file cookie | Kiểm tra `.erlang.cookie` (xem mục 19) |

```sh
rabbitmqctl authenticate_user <tên> <mật-khẩu>        # kiểm tra thông tin đăng nhập [Trí nhớ]
rabbitmqctl list_user_permissions <tên>
```

## 45. TLS và mạng

- **Lỗi bắt tay TLS:** sai chứng chỉ, sai chuỗi tin cậy, hoặc phiên bản TLS không khớp. Kiểm tra cấu hình `ssl_options` và chứng chỉ CA. [Trí nhớ]
- **Kết nối bị đóng ngay:** cổng sai, tường lửa, hoặc proxy ngoài không chuyển đúng giao thức. [Trí nhớ]
- **Kiểm tra cổng:** `rabbitmq-diagnostics check_port_connectivity`, và kiểm tra từ máy khách bằng công cụ mạng. [Trí nhớ]

Tài liệu troubleshooting chính thức liệt kê các mục riêng cho TLS, LDAP, OAuth 2.0, và phân tích gói tin bằng tcpdump và Wireshark. [Đã kiểm chứng]

## 46. Crash dump của runtime

Khi node crash, Erlang có thể tạo tệp `erl_crash.dump`. [Đã kiểm chứng: tài liệu troubleshooting nêu tệp này]

```sh
head -n 3 ./erl_crash.dump           # xem lý do kết thúc ở đầu tệp [Đã kiểm chứng]
```

Công cụ xem dump là `cdv` (Erlang Crash Dump Viewer), cần GUI với observer và Wx. [Đã kiểm chứng]

**Lưu ý:** gửi dump cho đội hỗ trợ cần kiểm tra nội dung nhạy cảm trước (dump có thể chứa dữ liệu message và thông tin đăng nhập). [Suy luận]

---

# PHẦN K. VẬN HÀNH VÀ GIÁM SÁT

## 47. Chỉ số cần theo dõi

| Chỉ số | Ý nghĩa | Cảnh báo khi |
|---|---|---|
| Alarm bộ nhớ, đĩa | Publisher bị chặn | Bật bất kỳ lúc nào |
| Số message ready | Backlog | Tăng liên tục |
| Số message unacknowledged | Đang xử lý dở | Không giảm |
| Số consumer | Có consumer hay không | Bằng 0 với queue quan trọng |
| Số connection và channel | Rò rỉ | Tăng mãi |
| Thành viên quorum queue online | Đa số | Dưới đa số |
| Tỷ lệ publish/deliver | Lưu lượng | Bất thường so với nền |

Plugin `rabbitmq_prometheus` xuất metrics cho Prometheus. Dashboard chuẩn có sẵn trong tài liệu cộng đồng. [Trí nhớ]

## 48. Sao lưu và khôi phục

- **Định nghĩa (definitions):** exchange, queue, binding, user, vhost, policy có thể xuất ra JSON để khôi phục cấu hình. [Trí nhớ]
- **Dữ liệu message:** cần sao lưu thư mục dữ liệu của node (`/var/lib/rabbitmq`) với node đã dừng hoặc theo hướng dẫn chính thức. [Trí nhớ]
- **Kiểm tra khôi phục:** thử trên môi trường thử nghiệm định kỳ.

```sh
rabbitmqadmin export definitions.json                # xuất cấu hình (cần plugin management) [Trí nhớ]
rabbitmqadmin import definitions.json                # nhập cấu hình
```

## 49. Nâng cấp

- Đọc ghi chú phát hành và tài liệu nâng cấp của dòng phiên bản, đặc biệt các thay đổi về quorum queue và tính năng bị gỡ (classic mirroring gỡ từ 4.0). [Đã kiểm chứng]
- Kiểm tra phiên bản Erlang/OTP tương thích trước khi nâng. [Đã kiểm chứng: trang yêu cầu xem trang Erlang/OTP riêng]
- Nâng từng node, theo thứ tự tài liệu, giữ đa số quorum trong suốt quá trình. [Suy luận]
- Sao lưu definitions và dữ liệu trước khi nâng. [Suy luận]

---

# PHẦN L. SO SÁNH VỚI KAFKA

| Tiêu chí | RabbitMQ | Kafka |
|---|---|---|
| Mô hình | Hàng đợi và định tuyến (AMQP) | Nhật ký sự kiện bền vững |
| Message sau khi xử lý | Xóa khi được ack | Giữ đến hết retention |
| Đọc lại | Hạn chế (stream để đọc lại) | Đọc lại theo offset |
| Định tuyến | Exchange linh hoạt (direct, topic, fanout) | Theo key và partition |
| Thứ tự | Trong queue (có hạn chế với nhiều consumer và requeue) | Trong partition |
| Thông lượng | Tốt, thường thấp hơn Kafka ở khối lượng rất lớn | Rất cao |
| Phù hợp | Công việc nền, RPC bất đồng bộ, định tuyến phức tạp | Luồng sự kiện, pipeline dữ liệu, fan-out lớn |

[Suy luận và kiến thức nền; so sánh khái quát, không phải benchmark]

---

# PHẦN M. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Message broker | Hệ thống trung gian nhận và phân phối message |
| AMQP | Giao thức nhắn tin mở, RabbitMQ triển khai |
| Connection / Channel | Kết nối TCP / luồng ảo trên kết nối |
| Exchange | Nơi nhận message từ producer và định tuyến |
| Binding | Liên kết exchange với queue |
| Routing key | Nhãn để định tuyến message |
| Queue | Nơi message chờ consumer |
| Classic / Quorum / Stream | Ba loại queue |
| Raft | Thuật toán đồng thuận sao chép log trong quorum queue |
| Quorum (đa số) | `(N/2)+1` thành viên online |
| Publisher confirm | Broker xác nhận đã nhận và sao chép message |
| Acknowledgement (ack/nack/reject) | Xác nhận xử lý, từ chối |
| Prefetch | Số message chưa ack consumer nhận cùng lúc |
| Dead-letter exchange (DLX) | Nơi nhận message bị từ chối hoặc hết hạn |
| Delivery limit | Số lần giao sai tối đa trước khi loại bỏ |
| Poison message | Message làm consumer thất bại liên tục |
| Vhost | Không gian cô lập trong broker |
| Erlang cookie | Bí mật xác thực giữa các node |
| Partition handling | Hành vi khi mạng giữa node bị đứt |
| Alarm (memory, disk) | Cơ chế chặn publisher khi tài nguyên thấp |
| Definitions | Cấu hình exchange, queue, user, policy dạng JSON |

---

# PHẦN N. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## N.1. Nguồn đã đối chiếu trong lần này

- **RabbitMQ, Download (dòng phát hành)**: https://www.rabbitmq.com/docs/download [đã đọc. Ghi nhận: bản mới nhất 4.4.0; dòng 4.4, 4.3, 4.2, 4.1, 4.0, 3.13; trang không ghi ngày phát hành và yêu cầu Erlang/OTP]
- **RabbitMQ, Quorum Queues**: https://www.rabbitmq.com/docs/quorum-queues [đã đọc. Ghi nhận: Raft; nhóm mặc định 3; đa số `(N/2)+1`; giới hạn và hành vi (không exclusive, không global QoS, ưu tiên 0–31 từ 4.3); delivery-limit mặc định 20 từ 4.0; x-delivery-count; dead-letter-strategy; publisher confirm]
- **RabbitMQ, Troubleshooting (trang điều hướng)**: https://www.rabbitmq.com/docs/troubleshooting [đã đọc. Ghi nhận: các chủ đề chẩn đoán; `rabbitmqctl environment`; `erl_crash.dump`; `cdv`. Trang không chứa hướng dẫn chi tiết từng lỗi]

## N.2. Nguồn chưa đối chiếu

- Hướng dẫn chi tiết từng chủ đề troubleshooting (alarm bộ nhớ và đĩa, partition, connection leak, cluster formation, TLS, LDAP, OAuth).
- Tài liệu cú pháp đầy đủ của `rabbitmqctl`, `rabbitmq-diagnostics`, `rabbitmq-queues`, `rabbitmqadmin`.
- Tài liệu cấu hình `rabbitmq.conf` và các tham số `consumer_timeout`, `cluster_partition_handling`, `disk_free_limit`.
- Tài liệu client: pika, và các thư viện AMQP khác.
- Tài liệu Erlang/OTP tương thích với 4.4.0.

## N.3. Điểm cần xác minh

1. **Ngày phát hành và Erlang/OTP của 4.4.0:** chưa xác nhận.
2. **Lệnh con và cờ** của `rabbitmq-queues` (`grow`, `rebalance`, `peek`, `check_if_node_is_mirror_sync_critical`) và `rabbitmq-diagnostics`: kiến thức nền, cần đọc `--help` trên phiên bản đang dùng.
3. **Cú pháp `set_policy`, `set_vhost_limits`, `set_user_limits`**: kiến thức nền.
4. **Tham số `rabbitmq.conf`** trong mục 23 và mục 22: kiến thức nền.
5. **Tên image Docker và biến môi trường** trong mục 25: cần kiểm tra trên trang image chính thức.
6. **Mã lỗi AMQP** (`ACCESS_REFUSED`, `NOT_ALLOWED`) và các thông báo mô tả trong mục 44: kiến thức nền, cần đối chiếu thông báo thực tế.
7. **Hành vi partition handling** và cổng 4369, 25672: kiến thức nền.
8. **Ví dụ pika** chưa được chạy thử. Tên tham số có thể khác theo phiên bản.
9. **Chỉ số và ngưỡng cảnh báo** trong mục 47 là hướng dẫn chung, cần đo trên tải thực tế.
10. **Mục 43 (quorum không có đa số)** có rủi ro mất dữ liệu. Đọc tài liệu chính thức trước khi thao tác.

---

**Tóm tắt để tra nhanh:**
- **RabbitMQ** = message broker AMQP: producer → exchange (định tuyến) → queue → consumer (ack).
- **Ba loại queue:** classic (không sao chép từ 4.0 bỏ mirroring), quorum (Raft, khuyến nghị cho dữ liệu quan trọng), stream (đọc lại theo offset).
- **Quorum queue:** nhóm 3, đa số `(N/2)+1`, publisher confirm sau khi đa số sao chép, delivery-limit mặc định 20, dùng DLX cho message độc hại.
- **Độ tin cậy:** publisher confirm, consumer ack có kiểm soát, prefetch hợp lý, consumer idempotent.
- **Gỡ lỗi:** `check_running` → `check_local_alarms` (alarm) → `cluster_status` → `list_queues` (ready vs unacked) → `list_connections`/`list_channels` (rò rỉ) → log và `erl_crash.dump`.
- **Cẩn trọng:** không tắt alarm, không `purge`/`reset`/`forget_cluster_node` khi chưa hiểu tác động, không để `guest` khi triển khai, và không đưa Erlang cookie vào repo.
