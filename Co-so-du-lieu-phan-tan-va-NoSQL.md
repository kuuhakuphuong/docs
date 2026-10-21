# Cơ sở dữ liệu phân tán và NoSQL, với MongoDB làm ví dụ chính

> **Viết cho:** người đã học cơ sở dữ liệu quan hệ (hoặc đã đọc tài liệu cùng bộ), muốn hiểu vì sao cần phân tán, các đánh đổi cốt lõi, và cách MongoDB áp dụng chúng.
> **Ví dụ xuyên suốt:** cửa hàng online đã dùng ở tài liệu cơ sở dữ liệu quan hệ, khi quy mô lớn lên nhiều khu vực.
> **Phiên bản MongoDB được tham chiếu:** Database Manual 9.0 (bản "Current" tại thời điểm tra cứu).
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu với tài liệu hoặc bài báo trong lần tra cứu; `[Trí nhớ]` = kiến thức nền, cần tra bản gốc; `[Suy luận]` = tôi suy ra hoặc minh họa.

---

## Mục lục

- **Lớp 0. Vì sao cần phân tán**
  - 1. Một máy không đủ, khi nào
  - 2. Ba mục tiêu: quy mô, sẵn sàng, gần người dùng
  - 3. Cái giá của phân tán
- **Lớp 1. Nền tảng: máy, mạng và lỗi**
  - 4. Node, mạng, và phân vùng mạng
  - 5. Sao chép (replication)
  - 6. Phân mảnh (partitioning, sharding)
  - 7. Mô hình nhất quán
- **Lớp 2. Các định lý và đánh đổi**
  - 8. CAP
  - 9. PACELC
  - 10. Quorum: đọc và ghi bao nhiêu máy là đủ
- **Lớp 3. Đồng thuận và bầu chọn**
  - 11. Vì sao cần đồng thuận
  - 12. Paxos, Raft: ý tưởng chung
- **Lớp 4. Hệ thống phân tán tiêu biểu**
  - 13. Dynamo: khóa-giá trị, ưu tiên sẵn sàng
  - 14. Các hệ thống khác để so sánh
- **Lớp 5. NoSQL**
  - 15. Vì sao có NoSQL
  - 16. Các loại NoSQL
  - 17. Đánh đổi: BASE và schema linh hoạt
- **Lớp 6. MongoDB**
  - 18. Mô hình dữ liệu: document và collection
  - 19. Replica set: sao chép và bầu primary
  - 20. Sharded cluster: phân mảnh
  - 21. Shard key: quyết định quan trọng nhất
  - 22. Đường đi của một lần ghi và một lần đọc
  - 23. Ba núm điều chỉnh: write concern, read concern, read preference
  - 24. Giao dịch trong MongoDB
  - 25. Các tình huống sự cố
- **Lớp 7. Kiểm thử hệ phân tán**
  - 26. Vì sao khó kiểm thử
  - 27. Kiểm thử tính nhất quán bằng lịch sử
  - 28. Kiểm thử sự cố bằng cách gây sự cố
  - 29. Thiết kế thí nghiệm có kiểm soát
- **Phụ lục**
  - A. Bảng so sánh NoSQL và quan hệ
  - B. Thuật ngữ
  - C. Nguồn tham khảo
  - D. Điểm cần xác minh

---

# LỚP 0. VÌ SAO CẦN PHÂN TÁN

## 1. Một máy không đủ, khi nào

Với cửa hàng nhỏ, một máy chủ PostgreSQL là đủ. Khi quy mô lớn lên, một máy gặp các giới hạn:

- **Dung lượng:** dữ liệu vượt quá đĩa hoặc RAM của một máy.
- **Thông lượng:** hàng nghìn giao dịch mỗi giây vượt khả năng xử lý của một máy.
- **Sẵn sàng:** máy hỏng thì dịch vụ ngừng. Với hệ thống 24/7, đây là vấn đề lớn.
- **Địa lý:** người dùng ở Hà Nội, Tokyo, New York, mỗi người muốn truy cập nhanh.

**Cơ sở dữ liệu phân tán** là hệ thống dữ liệu được lưu và xử lý trên nhiều máy, nối với nhau qua mạng, và trình bày ra ngoài như một hệ thống thống nhất.

## 2. Ba mục tiêu: quy mô, sẵn sàng, gần người dùng

| Mục tiêu | Cách đạt | Ví dụ |
|---|---|---|
| **Quy mô (scale)** | Chia dữ liệu và tải cho nhiều máy | Đơn hàng được chia theo khu vực |
| **Sẵn sàng (availability)** | Sao chép dữ liệu sang máy khác; máy hỏng thì máy khác thay | Một máy chết, máy khác nhận việc |
| **Độ trễ thấp (low latency)** | Đặt dữ liệu gần người dùng | Người Nhật đọc máy chủ Tokyo |

Ba mục tiêu này thường mâu thuẫn với nhau, và phần lớn tài liệu này nói về cách cân bằng chúng.

## 3. Cái giá của phân tán

Khi chạy trên nhiều máy, bạn phải đối mặt với:

- **Mạng không đáng tin:** gói tin có thể bị trễ, mất, hoặc đến không theo thứ tự.
- **Máy có thể chết bất cứ lúc nào,** và không phải lúc nào cũng biết chắc máy đã chết hay chỉ bị chậm.
- **Không có đồng hồ chung hoàn hảo** giữa các máy.
- **Dữ liệu có thể khác nhau giữa các bản sao** trong một khoảng thời gian.

Đây là những điều không xảy ra khi chạy trên một máy. Mọi thiết kế phân tán phải tính đến chúng. [Suy luận]

---

# LỚP 1. NỀN TẢNG: MÁY, MẠNG VÀ LỖI

## 4. Node, mạng, và phân vùng mạng

- **Node:** một máy (hoặc tiến trình) tham gia hệ thống.
- **Mạng:** các node gửi tin nhắn cho nhau.
- **Phân vùng mạng (network partition):** mạng bị cắt, chia các node thành nhóm không liên lạc được với nhau. Đây là loại lỗi quan trọng nhất trong lý thuyết phân tán.

```
  Trước phân vùng:        Sau phân vùng:
  A ── B ── C ── D        A ── B   ╳   C ── D
                                      (không liên lạc)
```

Điều khó là: từ phía B, không phân biệt được "C đã chết" với "mạng đến C bị đứt". Hệ thống phải quyết định cách phản ứng mà không chắc chắn. [Suy luận]

## 5. Sao chép (replication)

**Sao chép:** giữ nhiều bản sao của cùng một dữ liệu trên nhiều node.

Lợi ích: một bản sao hỏng thì còn bản khác, và có thể đọc từ nhiều bản để giảm tải.

Hai kiểu phổ biến:

| Kiểu | Cách hoạt động | Đánh đổi |
|---|---|---|
| **Đồng bộ (synchronous)** | Ghi xong ở đủ các bản sao rồi mới báo thành công | Chậm hơn, nhưng không mất dữ liệu khi máy chính hỏng |
| **Bất đồng bộ (asynchronous)** | Ghi ở bản chính, báo thành công, bản sao cập nhật sau | Nhanh hơn, nhưng có thể mất dữ liệu nếu bản chính hỏng trước khi sao chép xong |

**Mô hình leader-follower (primary-secondary):** một node là leader (primary) nhận mọi lần ghi; các node khác là follower (secondary) sao chép từ leader. Đây là mô hình MongoDB dùng. [Đã kiểm chứng]

**Mô hình leaderless:** mọi node đều nhận ghi, và dùng quorum để đảm bảo đúng đắn (mục 10). Dynamo và Cassandra theo hướng này. [Trí nhớ]

## 6. Phân mảnh (partitioning, sharding)

**Phân mảnh:** chia dữ liệu thành các phần (shard hoặc partition), mỗi phần nằm ở một tập node khác nhau.

Các cách chia phổ biến: [Trí nhớ]

| Cách | Mô tả | Ví dụ |
|---|---|---|
| **Theo khoảng (range)** | Mỗi shard giữ một khoảng khóa liên tiếp | Đơn hàng tháng 1–3 ở shard A, tháng 4–6 ở shard B |
| **Theo băm (hash)** | Băm khóa, chia theo giá trị băm | `hash(khach_id) mod 3` |
| **Theo thư mục (directory)** | Bảng tra cứu ghi rõ khóa nào ở đâu | Dùng khi cần linh hoạt tối đa |

**Đánh đổi:**
- **Range:** truy vấn theo khoảng nhanh (đọc đơn tháng 2 chỉ đụng shard A), nhưng dễ lệch tải nếu khóa tăng dần (mọi ghi mới đều vào shard cuối).
- **Hash:** phân bố đều, nhưng truy vấn theo khoảng phải hỏi tất cả shard.

**Consistent hashing:** cách băm đặc biệt để khi thêm hoặc bớt node, chỉ một phần nhỏ dữ liệu phải di chuyển, thay vì toàn bộ. Ý tưởng đã được Karger và cộng sự giới thiệu năm 1997. [Trí nhớ]

## 7. Mô hình nhất quán

**Nhất quán (consistency)** trong hệ phân tán có nhiều nghĩa. Các mô hình chính, từ mạnh đến yếu: [Trí nhớ, định nghĩa chuẩn]

| Mô hình | Ý nghĩa đơn giản |
|---|---|
| **Linearizability** (tuyến tính hóa) | Mọi thao tác như xảy ra tức thời tại một điểm giữa lúc bắt đầu và lúc kết thúc. Sau khi ghi thành công, mọi đọc sau đó đều thấy giá trị đó |
| **Sequential** | Mọi node thấy các thao tác theo cùng một thứ tự, nhưng thứ tự đó không nhất thiết khớp đồng hồ thực |
| **Causal** (nhân quả) | Các thao tác có quan hệ nhân quả được thấy đúng thứ tự. Thao tác độc lập có thể thấy khác thứ tự |
| **Eventual** (cuối cùng nhất quán) | Nếu không có ghi mới, cuối cùng mọi bản sao sẽ giống nhau. Không đảm bảo khi nào |

**Ví dụ cửa hàng:** An đổi địa chỉ giao hàng, rồi đặt đơn ngay. Với causal consistency, đơn mới chắc chắn dùng địa chỉ mới. Với eventual consistency, đơn có thể dùng địa chỉ cũ nếu đọc từ bản sao chưa cập nhật.

**Quy tắc:** mô hình càng mạnh thì càng đắt (chậm hơn, ít sẵn sàng hơn khi có sự cố). Chọn mô hình theo nhu cầu nghiệp vụ, không chọn mạnh nhất theo mặc định. [Suy luận]

---

# LỚP 2. CÁC ĐỊNH LÝ VÀ ĐÁNH ĐỔI

## 8. CAP

**Định lý CAP** được Eric Brewer nêu ra tại PODC năm 2000 dưới dạng phỏng đoán. Seth Gilbert và Nancy Lynch đưa ra chứng minh hình thức năm 2002 trong mô hình mạng bất đồng bộ. [Đã kiểm chứng: thông tin bài báo và người trình bày; ngày công bố có hai nguồn, 2000 và 2002]

Ba thuộc tính:
- **C (Consistency):** trong bài báo, đây là **linearizability**, không phải "ACID consistency".
- **A (Availability):** mọi yêu cầu đến node không bị lỗi đều nhận được phản hồi.
- **P (Partition tolerance):** hệ thống tiếp tục hoạt động khi mạng bị phân vùng.

**Kết luận:** khi có phân vùng mạng, không thể đồng thời đảm bảo cả C và A. [Đã kiểm chứng]

**Hiểu lầm phổ biến:**
- "Chọn 2 trong 3" là cách nói đơn giản hóa. Thực tế P không phải lựa chọn: mạng luôn có thể bị phân vùng, nên bạn phải chọn giữa C và A **khi có phân vùng**. [Đã kiểm chứng: chỉ dẫn từ bài báo và các tóm tắt]
- Khi không có phân vùng, hệ thống có thể đạt cả C và A. CAP chỉ nói về lúc có sự cố.
- Các mô hình nhất quán yếu hơn linearizability có thể đạt cả A và P. [Đã kiểm chứng: nhận định từ tóm tắt; cần đọc bài gốc để chi tiết]

## 9. PACELC

**PACELC** được Daniel Abadi đề xuất năm 2012 trên tạp chí IEEE Computer (tr. 37–42), với tên đầy đủ "Consistency tradeoffs in modern distributed database system design: CAP is only part of the story". [Đã kiểm chứng]

Ý tưởng: CAP chỉ nói về lúc có phân vùng. Nhưng **ngay cả khi không có phân vùng**, vẫn có đánh đổi giữa **nhất quán và độ trễ** (Latency). PACELC viết gọn:

```
Nếu có Phân vùng (P): chọn Availability (A) hoặc Consistency (C)
Ngược lại (E, Else):   chọn Latency (L) hoặc Consistency (C)
```

Ví dụ phân loại theo một bài tổng hợp trên web: MongoDB được xếp vào nhóm PA/EC (ưu tiên nhất quán trong cả hai trường hợp, theo cấu hình mặc định), PNUTS (của Yahoo) là PC/EL. Đây là phân loại của một tài liệu trung gian, cần kiểm chứng lại với tài liệu của MongoDB và bài gốc. [Đã kiểm chứng: nguồn tổng hợp; phân loại cần xác minh]

**Vì sao PACELC hữu ích:** nó nhắc rằng nhất quán mạnh đắt ngay cả khi mọi thứ bình thường, vì mỗi ghi phải chờ xác nhận từ các bản sao. [Suy luận]

## 10. Quorum: đọc và ghi bao nhiêu máy là đủ

Với N bản sao, một **quorum** là số bản sao tối thiểu phải phản hồi:
- **W:** số bản sao phải xác nhận ghi.
- **R:** số bản sao phải phản hồi khi đọc.

Điều kiện **R + W > N** đảm bảo mọi lần đọc giao với ít nhất một bản sao vừa được ghi. [Trí nhớ]

**Ví dụ N = 3:**

| Cấu hình | W | R | R + W > N? | Đặc điểm |
|---|---|---|---|---|
| Đọc nhanh | 3 | 1 | Có (4 > 3) | Ghi chậm, đọc nhanh |
| Cân bằng | 2 | 2 | Có (4 > 3) | Cân bằng |
| Ghi nhanh | 1 | 3 | Có (4 > 3) | Ghi nhanh, đọc chậm |
| Không đảm bảo | 1 | 1 | Không (2 ≤ 3) | Nhanh nhất, có thể đọc dữ liệu cũ |

**Lưu ý:** quorum đảm bảo giao nhau, nhưng không tự động đảm bảo linearizability trong mọi trường hợp (phụ thuộc vào cách cài đặt và xử lý lỗi). [Trí nhớ]

---

# LỚP 3. ĐỒNG THUẬN VÀ BẦU CHỌN

## 11. Vì sao cần đồng thuận

Nhiều việc phải có một **quyết định chung** giữa các node: ai là leader, thứ tự các giao dịch, thành viên nhóm. Nếu hai node cùng nghĩ mình là leader, sẽ xảy ra **split-brain**: hai bên cùng nhận ghi, dữ liệu rẽ nhánh.

**Đồng thuận (consensus)** là bài toán các node thống nhất một giá trị, dù có node chậm hoặc chết. [Trí nhớ]

## 12. Paxos, Raft: ý tưởng chung

**Paxos:** thuật toán đồng thuận do Leslie Lamport đề xuất (bài "The Part-Time Parliament", công bố năm 1998, được viết dưới dạng câu chuyện ẩn dụ). Nổi tiếng là khó hiểu và khó cài đặt đúng. [Trí nhớ]

**Raft:** Ongaro và Ousterhout giới thiệu năm 2014 (USENIX ATC), thiết kế với mục tiêu dễ hiểu hơn. [Trí nhớ; bài báo chưa được đọc trực tiếp trong lần tra cứu này]

Ý tưởng chung của Raft, đơn giản hóa:

```
1. Bầu chọn leader:   các node có "nhiệm kỳ" (term). Node chờ quá lâu không nghe leader
                      thì tự ứng cử. Ai được đa số phiếu thì làm leader.
2. Sao chép nhật ký:  leader nhận lệnh, ghi vào nhật ký, gửi cho follower.
                      Khi đa số follower xác nhận, lệnh được commit.
3. An toàn:           chỉ node có nhật ký mới nhất mới được bầu.
```

**Điều quan trọng để hiểu MongoDB:** cơ chế bầu primary trong replica set cũng dựa trên đa số. Chi tiết cụ thể về giao thức MongoDB dùng chưa được xác nhận từ trang tài liệu trong lần tra cứu này. [Đã kiểm chứng: trang tài liệu nói về bầu chọn và đa số khi có thể; không xác nhận là Raft]

**Đa số (majority):** số node cần để quyết định, thường là ⌊N/2⌋ + 1. Với 5 node, cần 3. Đa số đảm bảo hai nhóm không thể cùng có đa số, nên không có hai leader. [Suy luận]

---

# LỚP 4. HỆ THỐNG PHÂN TÁN TIÊU BIỂU

## 13. Dynamo: khóa-giá trị, ưu tiên sẵn sàng

**Dynamo** là hệ thống lưu trữ khóa-giá trị của Amazon, được mô tả bởi DeCandia và cộng sự tại SOSP 2007. [Đã kiểm chứng]

Các ý tưởng chính: [Đã kiểm chứng: tóm tắt từ các nguồn về bài báo]

- **Phân mảnh bằng vòng băm nhất quán (consistent hashing):** các node đặt trên một vòng, mỗi khóa thuộc node đầu tiên theo chiều kim đồng hồ.
- **Sao chép:** mỗi khóa được sao sang N−1 node kế tiếp trên vòng.
- **Sloppy quorum:** khi một node trong danh sách sao chép không liên lạc được, ghi có thể tạm đưa sang node khác ngoài danh sách.
- **Hinted handoff:** node tạm nhận ghi giữ "gợi ý" để chuyển lại cho node chính khi nó sống lại.
- **Vector clock:** theo dõi phiên bản để phát hiện xung đột khi cùng khóa được ghi ở hai nơi.

**Đánh đổi có chủ đích:** Dynamo ưu tiên sẵn sàng. Ghi luôn được chấp nhận, nhưng có thể sinh ra các phiên bản xung đột, và ứng dụng phải giải quyết. Đây là một ví dụ rõ về lựa chọn A trong PACELC và CAP. [Suy luận]

**Vì sao đáng học:** Dynamo là hình mẫu cho nhiều hệ NoSQL, và minh họa rõ rằng "sẵn sàng" và "nhất quán" là lựa chọn thiết kế, không phải thuộc tính miễn phí.

## 14. Các hệ thống khác để so sánh

| Hệ thống | Loại | Ghi chú | Trạng thái nguồn |
|---|---|---|---|
| **Bigtable** (Google, 2006) | Cột rộng (wide-column) | Lưu trên GFS, chia theo khoảng khóa | [Trí nhớ] |
| **Cassandra** | Cột rộng, leaderless | Kế thừa ý tưởng từ Dynamo và Bigtable | [Trí nhớ] |
| **Spanner** (Google, 2012) | NewSQL, quan hệ, phân tán | Nhất quán mạnh toàn cầu nhờ đồng hồ thời gian thực (TrueTime) | [Trí nhớ] |
| **Redis Cluster** | Khóa-giá trị trong bộ nhớ | Phân mảnh theo slot | [Trí nhớ] |

---

# LỚP 5. NOSQL

## 15. Vì sao có NoSQL

Từ khoảng 2000 đến 2010, nhiều công ty web gặp giới hạn với cơ sở dữ liệu quan hệ khi quy mô lớn: khó mở rộng ngang, lược đồ cố định khó thay đổi nhanh, và chi phí đồng bộ lớn. NoSQL (thường được hiểu là "Not Only SQL") ra đời như một tập hợp các lựa chọn thay thế. [Trí nhớ]

**Đừng hiểu NoSQL là "tốt hơn SQL".** Đó là tập các đánh đổi khác nhau. Nhiều hệ thống hiện đại dùng cả hai.

## 16. Các loại NoSQL

| Loại | Dữ liệu | Ví dụ | Tốt cho |
|---|---|---|---|
| **Khóa-giá trị** (key-value) | Cặp khóa và giá trị không cấu trúc | Redis, DynamoDB, Riak | Cache, phiên đăng nhập, giỏ hàng |
| **Tài liệu** (document) | Tài liệu JSON/BSON lồng nhau | **MongoDB**, CouchDB | Hồ sơ người dùng, danh mục sản phẩm, nội dung |
| **Cột rộng** (wide-column) | Hàng có tập cột linh hoạt | Cassandra, Bigtable, HBase | Dữ liệu chuỗi thời gian, ghi lượng lớn |
| **Đồ thị** (graph) | Đỉnh và cạnh | Neo4j | Quan hệ phức tạp: mạng xã hội, gợi ý |

## 17. Đánh đổi: BASE và schema linh hoạt

**BASE** là một khung đối lập với ACID, dùng trong nhiều hệ NoSQL: [Trí nhớ]
- **B**asically **A**vailable: hệ thống luôn trả lời, dù có thể không mới nhất.
- **S**oft state: trạng thái có thể thay đổi theo thời gian mà không có ghi mới, do đồng bộ.
- **E**ventual consistency: cuối cùng sẽ nhất quán.

**Schema linh hoạt (schema-flexible):** không cần khai báo cấu trúc trước. Mỗi tài liệu có thể có trường khác nhau. Tiện cho phát triển nhanh, nhưng dễ dẫn đến dữ liệu không đồng nhất nếu không có kỷ luật. [Suy luận]

**Lưu ý:** nhiều hệ NoSQL hiện nay hỗ trợ giao dịch và schema validation. Ranh giới giữa quan hệ và NoSQL đã mờ đi. [Suy luận]

---

# LỚP 6. MONGODB

## 18. Mô hình dữ liệu: document và collection

**Document:** một bản ghi dạng BSON (binary JSON). Có thể lồng cấu trúc.

**Collection:** tập các document, tương tự bảng trong quan hệ, nhưng không bắt buộc cùng cấu trúc.

Ví dụ đơn hàng trong cửa hàng, dạng một document:

```json
{
  "_id": "don-101",
  "khach": { "id": "kh-1", "ten": "An" },
  "ngay": "2026-10-21",
  "san_pham": [
    { "ma": "sp-7", "ten": "Áo thun", "so_luong": 2, "gia_luc_mua": 150000 },
    { "ma": "sp-9", "ten": "Quần jean", "so_luong": 1, "gia_luc_mua": 450000 }
  ],
  "tong_tien": 750000
}
```

So với quan hệ: đơn hàng và chi tiết đơn nằm **trong cùng một document**, không cần nối bảng. [Suy luận từ cấu trúc]

**Hai cách mô hình quan hệ:**
- **Nhúng (embed):** như ví dụ trên. Đọc một lần lấy đủ. Tốt khi dữ liệu con thuộc về cha và ít khi tách ra.
- **Tham chiếu (reference):** lưu `khach_id` và truy vấn riêng. Giống khóa ngoại. Tốt khi dữ liệu con dùng chung nhiều nơi hoặc rất lớn.

**Nguyên tắc:** chọn theo truy vấn thường gặp, không theo "đẹp" của lược đồ. [Suy luận]

## 19. Replica set: sao chép và bầu primary

**Replica set** là nhóm các tiến trình `mongod` giữ cùng một tập dữ liệu, để dự phòng và sẵn sàng cao. [Đã kiểm chứng]

Thành phần:
- **Primary:** nhận mọi thao tác ghi, ghi vào **oplog** (nhật ký thao tác).
- **Secondary:** sao chép oplog từ primary và áp dụng **bất đồng bộ**. Có thể phục vụ đọc nếu được cấu hình.
- **Arbiter:** tham gia bầu chọn nhưng **không giữ dữ liệu**. Dùng khi không muốn thêm bản sao thứ hai chứa dữ liệu. [Đã kiểm chứng]

```
          ghi
 Client ─────► Primary ──oplog──► Secondary 1
                  │       └──────► Secondary 2
                  │
             (arbiter: chỉ bỏ phiếu, không chứa dữ liệu)
```

**Bầu chọn và chuyển đổi:**
- Nếu primary không liên lạc với các thành viên khác quá `electionTimeoutMillis` (mặc định 10 giây), một secondary đủ điều kiện sẽ khởi xướng bầu.
- Thời gian trung vị để bầu primary mới "không nên vượt quá 12 giây" với cấu hình mặc định. [Đã kiểm chứng]
- Giảm `electionTimeoutMillis` phát hiện lỗi nhanh hơn, nhưng dễ kích hoạt bầu chọn không cần thiết, và tăng số lần rollback với ghi `w: 1`. [Đã kiểm chứng]
- Driver có thể thử lại một lần đối với một số thao tác ghi (retryable writes), giúp vượt qua chuyển đổi. [Đã kiểm chứng]

**Đánh đổi quan trọng:** sao chép là bất đồng bộ theo mặc định, nên ghi ở primary có thể chưa đến secondary khi primary hỏng. Khi đó, thao tác đã báo thành công có thể bị rollback. Mục 23 trình bày cách tránh. [Đã kiểm chứng]

## 20. Sharded cluster: phân mảnh

Sharded cluster gồm ba loại thành phần: [Đã kiểm chứng]

| Thành phần | Vai trò |
|---|---|
| **mongos** | Bộ định tuyến truy vấn. Ứng dụng phải kết nối qua mongos, **không** kết nối trực tiếp vào shard để đọc ghi |
| **Config servers** | Lưu siêu dữ liệu và cấu hình cluster. Phải là một replica set |
| **Shards** | Mỗi shard giữ một phần dữ liệu. Mỗi shard phải là một replica set |

```
            Client
              │
              ▼
   ┌───────────────────────┐
   │ mongos (router)       │◄──── Config servers (replica set)
   └───┬───────────┬───────┘       lưu: chunk nào ở shard nào
       ▼           ▼
   Shard A      Shard B    ...     (mỗi shard là một replica set)
   (RS)         (RS)
```

**Phân mảnh ở mức collection:** sharding áp dụng cho từng collection. Có thể có collection được phân mảnh và collection không, trong cùng một database. [Đã kiểm chứng]

**Chunk và balancer:**
- Dữ liệu được chia thành **chunk**, mỗi chunk là một khoảng giá trị shard key, gồm cận dưới (bao gồm) và cận trên (không bao gồm). [Đã kiểm chứng]
- **Balancer** chạy nền, di chuyển chunk giữa các shard để cân bằng. [Đã kiểm chứng]

**Từ phiên bản 8.0,** config server có thể chạy như một **config shard**, đồng thời lưu dữ liệu ứng dụng. [Đã kiểm chứng]

## 21. Shard key: quyết định quan trọng nhất

**Shard key** là một hoặc nhiều trường quyết định document nào nằm ở shard nào. Chọn sai là nguồn gốc của nhiều vấn đề về sau. [Đã kiểm chứng]

**Hai cách:**

| Cách | Cơ chế | Ưu | Nhược |
|---|---|---|---|
| **Hashed** | Băm shard key, chia theo khoảng giá trị băm | Phân bố đều, kể cả khi khóa tăng dần | Truy vấn theo khoảng khóa thường phải hỏi mọi shard |
| **Ranged** | Chia theo khoảng liên tiếp của shard key | Truy vấn theo khoảng nhắm đúng shard | Dễ lệch tải, nếu khóa không phân bố đều |

**Các điều cần biết:** [Đã kiểm chứng]
- Collection có dữ liệu cần một index bắt đầu bằng shard key. Collection rỗng thì MongoDB có thể tự tạo.
- Truy vấn **không chứa** shard key (hoặc tiền tố của shard key) bị gửi đến **mọi shard** (scatter-gather), có thể chậm.
- Giá trị shard key có thể cập nhật, trừ trường `_id` vốn bất biến.
- Từ phiên bản 5.0 có thể **reshard** để đổi shard key, và có thể **refine** bằng cách thêm trường hậu tố.
- Trường thiếu trong shard key được coi là null khi phân phối document, nhưng không dùng khi định tuyến truy vấn.

**Ví dụ cho cửa hàng:**
- Shard key `khach_id` (hashed): đơn của một khách nằm cùng shard, nhưng khách có nhiều đơn sẽ tập trung ở một shard nếu có khách "siêu lớn". Truy vấn "tổng doanh thu theo ngày" phải hỏi mọi shard. [Suy luận]
- Shard key `ngay` (ranged): truy vấn theo ngày nhanh, nhưng mọi đơn mới đều vào shard cuối, gây nóng (hot spot). [Suy luận]
- Shard key kết hợp `{khu_vuc, ngay}`: cân bằng giữa hai nhu cầu, nhưng vẫn cần kiểm chứng bằng dữ liệu thật. [Suy luận]

**Quy tắc:** chọn shard key theo **truy vấn thường gặp** và **phân bố giá trị**. Đo trước khi quyết định. [Suy luận]

## 22. Đường đi của một lần ghi và một lần đọc

**Ghi (mặc định, đơn giản hóa):**

```
1. Ứng dụng gửi ghi đến mongos
2. mongos xác định shard chứa document (dựa trên shard key và metadata)
3. Ghi đi đến primary của shard đó
4. Primary ghi vào oplog và cập nhật dữ liệu
5. Secondary sao chép oplog (bất đồng bộ)
6. Primary trả kết quả theo write concern đã chọn (mục 23)
```

**Đọc (mặc định):**

```
1. Ứng dụng gửi truy vấn đến mongos
2. Nếu truy vấn có shard key: mongos chỉ hỏi shard liên quan
   Nếu không: mongos hỏi mọi shard (scatter-gather)
3. Đọc từ primary (mặc định), trả kết quả
```

Tài liệu nói rằng theo mặc định, client đọc từ primary. Có thể chọn đọc từ secondary, nhưng khi đó dữ liệu có thể chưa phản ánh trạng thái mới nhất của primary. [Đã kiểm chứng]

## 23. Ba núm điều chỉnh: write concern, read concern, read preference

Đây là phần **quan trọng nhất để hiểu hành vi thực tế** của MongoDB.

**Write concern:** ghi phải được xác nhận bởi bao nhiêu node trước khi báo thành công. [Đã kiểm chứng]
- `w: 1`: primary xác nhận là đủ. Nhanh, nhưng có thể mất dữ liệu nếu primary hỏng trước khi sao chép.
- `w: "majority"`: đa số node xác nhận. An toàn hơn trước các chuyển đổi primary, chậm hơn.
- Giá trị mặc định hiện tại của write concern được cấu hình theo phiên bản. Trong các phiên bản gần đây, mặc định đã chuyển sang `majority`. [Trí nhớ; trang tài liệu được tra cứu không nêu giá trị mặc định, cần đối chiếu bản cụ thể]

**Read concern:** mức đảm bảo của dữ liệu được đọc. [Đã kiểm chứng]
- `local` và `available`: có thể thấy ghi chưa được xác nhận, và có thể đọc dữ liệu sau này bị rollback khi chuyển đổi primary.
- Các mức mạnh hơn như `majority` và `linearizable`: cần đối chiếu tài liệu chi tiết trước khi dùng. [Trí nhớ; chưa đọc trang riêng]

**Read preference:** đọc từ node nào. [Đã kiểm chứng]
- Mặc định: `primary`.
- Có thể chọn secondary để giảm tải, đổi lại có thể đọc dữ liệu cũ.

**Bảng lựa chọn thực hành (minh họa, không phải khuyến nghị tuyệt đối):** [Suy luận]

| Nhu cầu | Gợi ý cấu hình |
|---|---|
| Đơn hàng, thanh toán | Ghi `w: "majority"`, đọc từ primary |
| Trang sản phẩm xem nhiều | Đọc từ secondary chấp nhận dữ liệu hơi cũ |
| Log, thống kê lớn | Ghi `w: 1` có thể chấp nhận, nếu mất vài bản ghi không nghiêm trọng |

## 24. Giao dịch trong MongoDB

- MongoDB hỗ trợ **giao dịch đa tài liệu (multi-document transactions)**, cho phép nhiều thao tác trên nhiều document và collection được xử lý như một đơn vị. [Đã kiểm chứng]
- Giao dịch có đọc phải dùng read preference `primary`. [Đã kiểm chứng]
- Giao dịch có chi phí. Nếu thiết kế dữ liệu tốt, phần lớn thao tác chỉ cần một document, và đã là nguyên tử ở mức document. [Suy luận; tài liệu MongoDB nói rõ thao tác trên một document là nguyên tử, cần đối chiếu]

**Quy tắc thực hành:** đặt dữ liệu cần cập nhật cùng nhau vào cùng một document khi có thể. Dùng giao dịch khi thật sự cần, và giữ ngắn.

## 25. Các tình huống sự cố

| Tình huống | Điều gì xảy ra | Hệ quả với dữ liệu |
|---|---|---|
| Primary hỏng khi ghi `w: 1` chưa sao chép | Secondary được bầu làm primary mới. Ghi chưa sao chép bị rollback khi máy cũ quay lại | Thao tác đã báo thành công có thể mất |
| Primary hỏng khi ghi `w: "majority"` | Thao tác đã được đa số xác nhận, được giữ lại | An toàn hơn |
| Phân vùng: primary bị cô lập với đa số | Đa số bầu primary mới. Primary cũ không còn đa số, nên không nên tiếp tục nhận ghi an toàn | Cần đúng cấu hình để không mất dữ liệu |
| Shard hỏng | Các truy vấn cần shard đó bị lỗi. Phần dữ liệu còn lại vẫn đọc được (tùy truy vấn) | Phụ thuộc vào replica set của shard |
| Config server không liên lạc được | Không thể thay đổi metadata cluster (chunk, shard) | Cần khôi phục config server |

**Điểm mấu chốt:** an toàn dữ liệu không đến từ việc dùng MongoDB, mà từ việc **cấu hình đúng write concern, số node, và kiểm thử sự cố**. [Suy luận]

---

# LỚP 7. KIỂM THỬ HỆ PHÂN TÁN

## 26. Vì sao khó kiểm thử

- Lỗi xuất hiện do **thứ tự thông điệp** và **thời điểm sự cố**. Số tổ hợp rất lớn.
- Lỗi **không tái hiện** được dễ dàng.
- Chạy đúng 1.000 lần không có nghĩa là đúng.

Vì vậy, kiểm thử hệ phân tán dựa vào **lịch sử thao tác** và **gây sự cố có chủ đích**, không chỉ dựa vào kết quả mong đợi. [Suy luận]

## 27. Kiểm thử tính nhất quán bằng lịch sử

**Quy trình:**
1. Chạy nhiều client song song, thực hiện ghi và đọc trên một tập khóa nhỏ (để tăng xung đột).
2. Ghi lại **lịch sử**: mỗi thao tác có thời điểm bắt đầu, kết thúc, đầu vào, đầu ra.
3. Dùng bộ kiểm tra để xem lịch sử có **tương thích với mô hình nhất quán** đã hứa không (ví dụ linearizable).

**Công cụ tham khảo:**
- **Jepsen:** bộ công cụ kiểm thử hệ phân tán, đã phân tích nhiều hệ thống, trong đó có MongoDB. [Trí nhớ, cần xác minh các báo cáo cụ thể]
- **Các bộ kiểm tra linearizability** như Knossos, Porcupine. [Trí nhớ]

**Cảnh báo:** kiểm thử chỉ tìm được lỗi nếu kịch bản kích hoạt được lỗi. Không tìm thấy lỗi không chứng minh hệ thống đúng. [Suy luận]

## 28. Kiểm thử sự cố bằng cách gây sự cố

**Các loại sự cố có thể gây:** [Suy luận]
- Dừng tiến trình (kill) primary hoặc secondary.
- Phân vùng mạng: cô lập một node hoặc chia cluster thành hai nhóm.
- Trễ mạng và mất gói tin giả lập.
- Đồng hồ lệch giữa các node.
- Đầy đĩa, lỗi I/O.

**Kịch bản mẫu cho replica set:**
1. Ghi liên tục với `w: "majority"`, ghi lại thao tác nào báo thành công.
2. Cô lập primary khỏi đa số.
3. Chờ bầu primary mới, tiếp tục ghi.
4. Khôi phục mạng.
5. Kiểm tra: mọi thao tác báo thành công có còn trong dữ liệu không? Có thao tác nào bị rollback mà không báo?

**Tiêu chí:** với `w: "majority"`, không có thao tác đã báo thành công nào bị mất. Với `w: 1`, kỳ vọng khác. Nên kiểm chứng cả hai để thấy sự khác biệt. [Suy luận]

## 29. Thiết kế thí nghiệm có kiểm soát

1. **Câu hỏi:** ví dụ "với `w: "majority"`, sau một chuyển đổi primary, có mất thao tác đã báo thành công không?"
2. **Giả thuyết:** nêu rõ kết quả mong đợi.
3. **Môi trường:** ghi rõ phiên bản MongoDB, số node, cấu hình (`electionTimeoutMillis`, write concern), phần cứng, mạng.
4. **Biến:** một biến thay đổi mỗi lần (ví dụ write concern), các biến khác cố định.
5. **Lặp lại:** nhiều lần cho mỗi điều kiện, xen kẽ thứ tự chạy.
6. **Thu thập:** lịch sử thao tác, nhật ký MongoDB, thời gian bầu chọn.
7. **Phân tích:** đếm số thao tác mất, thời gian không khả dụng, báo cáo khoảng tin cậy.
8. **Tái lập:** lưu mã, cấu hình, và dữ liệu.

**Nguyên tắc:** mỗi kết luận cần một thí nghiệm có thể bác bỏ nó. "Có vẻ ổn" không phải kết quả.

---

# PHỤ LỤC

## A. Bảng so sánh NoSQL và quan hệ

| Tiêu chí | Quan hệ (PostgreSQL) | Tài liệu (MongoDB) | Khóa-giá trị (Dynamo-style) |
|---|---|---|---|
| Lược đồ | Cố định, khai báo trước | Linh hoạt, có thể validate | Gần như không có |
| Nối bảng | Mạnh, là cốt lõi | Hạn chế; ưu tiên nhúng | Không có |
| Giao dịch | ACID đầy đủ | Đa tài liệu, có chi phí | Thường giới hạn |
| Phân tán | Có, nhưng thường cần công cụ thêm | Gốc có sẵn (sharding, replica) | Gốc có sẵn |
| Nhất quán mặc định | Mạnh trên một node | Phụ thuộc write/read concern | Thường yếu hơn, có thể cấu hình |
| Truy vấn linh hoạt | Rất mạnh | Mạnh vừa phải | Hạn chế theo khóa |

## B. Thuật ngữ

| Thuật ngữ | Giải thích ngắn |
|---|---|
| Node | Một máy hoặc tiến trình trong hệ thống |
| Phân vùng mạng | Mạng bị cắt, chia node thành nhóm |
| Sao chép (replication) | Giữ nhiều bản sao dữ liệu |
| Phân mảnh (sharding) | Chia dữ liệu cho nhiều máy |
| Consistent hashing | Băm để thêm/bớt node ít di chuyển dữ liệu |
| Linearizability | Nhất quán mạnh nhất, như thao tác tức thời |
| Causal consistency | Giữ thứ tự theo quan hệ nhân quả |
| Eventual consistency | Cuối cùng sẽ nhất quán |
| CAP | Không thể đồng thời C và A khi có phân vùng |
| PACELC | Mở rộng CAP: ngay cả không phân vùng, có đánh đổi nhất quán và độ trễ |
| Quorum | Số bản sao tối thiểu phải phản hồi |
| Đồng thuận (consensus) | Các node thống nhất một quyết định |
| Split-brain | Hai nhóm cùng tin mình là leader |
| Leader / primary | Node nhận ghi |
| Follower / secondary | Node sao chép từ leader |
| Oplog | Nhật ký thao tác của MongoDB |
| Replica set | Nhóm bản sao của MongoDB |
| Arbiter | Thành viên chỉ bỏ phiếu, không giữ dữ liệu |
| mongos | Bộ định tuyến của sharded cluster |
| Shard key | Trường quyết định phân phối dữ liệu |
| Chunk | Khoảng giá trị shard key được quản lý như một đơn vị |
| Balancer | Tiến trình cân bằng chunk giữa các shard |
| Write concern | Số node phải xác nhận ghi |
| Read concern | Mức đảm bảo của dữ liệu đọc |
| Read preference | Đọc từ primary hay secondary |
| Scatter-gather | Truy vấn gửi đến mọi shard rồi gộp |
| Rollback | Hoàn tác thao tác chưa được sao chép khi chuyển primary |
| BASE | Khung đối lập ACID: sẵn sàng cơ bản, nhất quán cuối cùng |

## C. Nguồn tham khảo

**Bài báo kinh điển:**
- Brewer, E. (2000). Phỏng đoán nêu tại PODC. [Đã kiểm chứng: thông tin qua nguồn tổng hợp]
- Gilbert, S. và Lynch, N. (2002). "Brewer's conjecture and the feasibility of consistent, available, partition-tolerant web services." *ACM SIGACT News*, 33(2), 51–59. Bản PDF: https://www.comp.nus.edu.sg/~gilbert/pubs/BrewersConjecture-SigAct.pdf [đã tra cứu; một nguồn ghi năm 2000 cho bản đầu, cần kiểm tra thêm]
- Abadi, D. J. (2012). "Consistency Tradeoffs in Modern Distributed Database System Design: CAP is Only Part of the Story." *IEEE Computer*, 45(2), 37–42. PDF: https://cs.umd.edu/~abadi/papers/abadi-pacelc.pdf [đã tra cứu; số tập và số trang cần đối chiếu với bản in]
- DeCandia, G., et al. (2007). "Dynamo: Amazon's Highly Available Key-value Store." *SOSP 2007*. PDF: https://www.cs.purdue.edu/homes/bb/cs590/papers/amazon-dynamo-sosp2007.pdf [đã tra cứu]
- Ongaro, D. và Ousterhout, J. (2014). "In Search of an Understandable Consensus Algorithm." *USENIX ATC 2014*. [Trí nhớ; bài báo chưa được đọc trực tiếp]
- Lamport, L. (1998). "The Part-Time Parliament." *ACM TOCS*. [Trí nhớ]
- Karger, D., et al. (1997). "Consistent Hashing and Random Trees." *STOC*. [Trí nhớ]
- Chang, F., et al. (2006). "Bigtable: A Distributed Storage System for Structured Data." *OSDI*. [Trí nhớ]
- Corbett, J., et al. (2012). "Spanner: Google's Globally-Distributed Database." *OSDI*. [Trí nhớ]
- Cooper, B. F., et al. (2010). "Benchmarking Cloud Serving Systems with YCSB." *SoCC*. [Trí nhớ]

**Tài liệu MongoDB (phiên bản 9.0):**
- Replication (replica set): https://www.mongodb.com/docs/manual/replication/ [đã đọc]
- Sharding: https://www.mongodb.com/docs/manual/sharding/ [đã đọc]
- Các trang riêng về read concern, write concern, transactions: cần truy cập để đối chiếu chi tiết. [chưa đọc trong lần này]

**Kiểm thử:**
- Jepsen: https://jepsen.io/ [Trí nhớ]

## D. Điểm cần xác minh

1. **Giao thức bầu chọn của MongoDB:** trang replication không nói rõ là Raft, cũng không nêu rõ yêu cầu đa số khi bầu. Cần đối chiếu tài liệu riêng.
2. **Giá trị mặc định của write concern** trong phiên bản 9.0: tài liệu được đọc không nêu rõ. Mục 23 dựa vào kiến thức nền.
3. **Các mức read concern mạnh** (`majority`, `linearizable`): chưa đọc trang riêng.
4. **Phân loại PACELC của MongoDB** (PA/EC): từ bài tổng hợp, cần kiểm chứng.
5. **Năm công bố CAP:** một nguồn ghi 2000, nguồn khác ghi 2002. Tài liệu này dùng 2000 cho phỏng đoán và 2002 cho chứng minh.
6. **Các kết luận về Dynamo** dựa trên tóm tắt từ các nguồn. Nên đọc bài gốc để chắc chắn.
7. **Các ví dụ shard key** trong mục 21 là minh họa, cần đo trên dữ liệu thật.
8. **Chưa có thí nghiệm thực tế** trong tài liệu này. Các mục 27–29 là phương pháp, không phải kết quả.
9. Các câu lệnh và cấu hình MongoDB cụ thể (ví dụ tên tham số) cần kiểm tra trên đúng phiên bản bạn dùng trước khi chạy.
