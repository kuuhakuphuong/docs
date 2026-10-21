# Các mô hình cơ sở dữ liệu hiện nay: vai trò, chức năng, ưu nhược điểm, cấu hình và câu lệnh cần chú ý

> **Viết cho:** người cần chọn cơ sở dữ liệu cho một dự án, hoặc muốn hiểu sự khác nhau giữa các mô hình trước khi học sâu từng hệ thống.
> **Phạm vi:** 11 nhóm mô hình, mỗi nhóm có vai trò, ưu nhược điểm, cấu hình quan trọng, và câu lệnh cần chú ý. Cuối tài liệu có bảng so sánh và danh sách kiểm tra vận hành.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:**
> - `[Đã kiểm chứng]` = đối chiếu với tài liệu chính thức trong lần này.
> - `[Trí nhớ]` = kiến thức nền, cần kiểm tra lại với tài liệu của phiên bản bạn dùng trước khi áp dụng.
> - `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về cấu hình và câu lệnh:** tên tham số và câu lệnh có thể khác giữa các phiên bản. Mọi giá trị trong tài liệu là ví dụ, không phải khuyến nghị cho môi trường thật.

---

## Mục lục

- **Phần A. Nền tảng: vì sao có nhiều mô hình**
  - 1. Database là gì, và các trục để phân loại
  - 2. Bốn trục đánh đổi thường gặp
- **Phần B. Mô hình quan hệ (Relational)**
- **Phần C. Mô hình tài liệu (Document)**
- **Phần D. Mô hình khóa-giá trị (Key-Value)**
- **Phần E. Mô hình cột rộng (Wide-Column)**
- **Phần F. Mô hình đồ thị (Graph)**
- **Phần G. Mô hình chuỗi thời gian (Time-Series)**
- **Phần H. Mô hình tìm kiếm toàn văn (Search)**
- **Phần I. Mô hình vector (Vector)**
- **Phần J. Mô hình phân tích cột (Columnar, OLAP)**
- **Phần K. Mô hình SQL phân tán (NewSQL)**
- **Phần L. Mô hình đa mô hình (Multi-model)**
- **Phần M. So sánh tổng hợp và cách chọn**
- **Phần N. Danh sách kiểm tra vận hành**
- **Phần O. Thuật ngữ**
- **Phần P. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. NỀN TẢNG: VÌ SAO CÓ NHIỀU MÔ HÌNH

## 1. Database là gì, và các trục để phân loại

**Cơ sở dữ liệu (database)** là hệ thống lưu, tìm, cập nhật dữ liệu, và đảm bảo dữ liệu đáng tin khi có nhiều người dùng và sự cố. **Hệ quản trị cơ sở dữ liệu (DBMS)** là phần mềm thực hiện việc đó.

Không có một mô hình tốt nhất. Mỗi mô hình là một tập lựa chọn về **cách tổ chức dữ liệu**, **cách truy vấn**, và **cách đánh đổi**. Có bốn trục phân loại thường gặp:

| Trục | Câu hỏi |
|---|---|
| **Mô hình dữ liệu** | Dữ liệu là bảng, tài liệu, cặp khóa-giá trị, cột, đồ thị, hay vector? |
| **Khối lượng công việc** | Chủ yếu là giao dịch nhỏ (OLTP) hay phân tích lượng lớn (OLAP)? |
| **Phân tán** | Chạy trên một máy, hay nhiều máy? |
| **Đảm bảo nhất quán** | Cần ACID đầy đủ, hay chấp nhận nhất quán yếu hơn để đổi lấy sẵn sàng? |

## 2. Bốn trục đánh đổi thường gặp

Các đánh đổi đã được trình bày chi tiết trong tài liệu về phân tán. Tóm tắt để dùng khi đọc các mục sau:

| Đánh đổi | Ý nghĩa |
|---|---|
| **Nhất quán và sẵn sàng** | Khi có sự cố mạng, chọn trả lời chắc chắn đúng hay luôn trả lời (CAP) |
| **Nhất quán và độ trễ** | Ngay cả khi bình thường, đảm bảo mạnh làm chậm (PACELC) |
| **Linh hoạt lược đồ và kiểm soát dữ liệu** | Lược đồ linh hoạt dễ phát triển nhanh nhưng dễ sinh dữ liệu không đồng nhất |
| **Đọc nhanh và ghi nhanh** | Chỉ mục và cấu trúc tối ưu cho đọc thường làm ghi chậm |

---

# PHẦN B. MÔ HÌNH QUAN HỆ (RELATIONAL)

**Hệ thống tiêu biểu:** PostgreSQL, MySQL/MariaDB, SQL Server, Oracle Database.

## B.1. Vai trò và chức năng

- Lưu dữ liệu dưới dạng bảng, có khóa, quan hệ, và ràng buộc.
- Truy vấn bằng SQL, nối bảng linh hoạt.
- Giao dịch ACID, là lựa chọn mặc định cho nghiệp vụ cần đúng đắn: tài chính, đơn hàng, tồn kho.
- Phù hợp khi dữ liệu có cấu trúc rõ, và quan hệ giữa các thực thể quan trọng.

## B.2. Ưu điểm

- Ràng buộc toàn vẹn mạnh, bảo vệ dữ liệu ngay từ cơ sở dữ liệu.
- SQL chuẩn hóa, nhiều công cụ và kỹ sư quen thuộc.
- Trình tối ưu hóa truy vấn trưởng thành.
- Giao dịch đa câu lệnh, mức cô lập rõ ràng.

## B.3. Nhược điểm

- Mở rộng ngang (nhiều máy ghi) khó hơn so với các mô hình phân tán thiết kế sẵn.
- Thay đổi lược đồ trên bảng lớn có thể tốn thời gian và cần kế hoạch.
- Nối nhiều bảng lớn có thể chậm nếu thiết kế chỉ mục kém.

## B.4. Cấu hình quan trọng (PostgreSQL)

Ví dụ các tham số thường được điều chỉnh. Giá trị phụ thuộc vào RAM, tải, và phần cứng.

| Tham số | Vai trò | Ghi chú |
|---|---|---|
| `shared_buffers` | Bộ nhớ đệm trang dùng chung | Thường đặt một phần RAM, cần đo |
| `work_mem` | Bộ nhớ cho sắp xếp và băm trong mỗi thao tác | Nhân với số thao tác đồng thời, cẩn thận |
| `max_connections` | Số kết nối tối đa | Nên dùng bộ gộp kết nối (connection pooler) thay vì tăng quá mức |
| `wal_level` | Mức ghi nhật ký WAL | `replica` là mặc định phổ biến cho sao chép |
| `synchronous_commit` | Có chờ ghi WAL xuống đĩa khi commit không | Tắt tăng tốc nhưng có thể mất vài giao dịch gần nhất khi sự cố |
| `default_transaction_isolation` | Mức cô lập mặc định | `read committed` là mặc định |
| `autovacuum` | Dọn phiên bản cũ tự động | Không nên tắt trong môi trường thật |

[Trí nhớ; tham số có trong tài liệu PostgreSQL, cần đối chiếu phiên bản]

## B.5. Câu lệnh cần chú ý

```sql
-- Xem kế hoạch thực thi thật
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM don_hang WHERE khach_id = 42;

-- Tạo chỉ mục không khóa bảng ghi (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_don_khach ON don_hang(khach_id);

-- Cập nhật thống kê và dọn phiên bản cũ
ANALYZE don_hang;
VACUUM (VERBOSE, ANALYZE) don_hang;

-- Giao dịch với mức cô lập
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- ...
COMMIT;

-- Thay đổi lược đồ an toàn: thêm cột có DEFAULT trên bảng lớn cần kiểm tra kỹ
ALTER TABLE don_hang ADD COLUMN ghi_chu TEXT;
```

**Cần chú ý:**
- `CREATE INDEX` thông thường khóa ghi trên bảng. Dùng `CONCURRENTLY` trên hệ thống đang chạy. [Trí nhớ]
- `UPDATE` hoặc `DELETE` không có `WHERE` sẽ áp dụng cho mọi hàng. Luôn chạy `SELECT` với cùng điều kiện trước.
- Với bảng rất lớn, thêm cột có giá trị mặc định có thể tốn nhiều thời gian, tùy phiên bản. [Trí nhớ]

---

# PHẦN C. MÔ HÌNH TÀI LIỆU (DOCUMENT)

**Hệ thống tiêu biểu:** MongoDB, CouchDB, Amazon DocumentDB (tương thích MongoDB).

## C.1. Vai trò và chức năng

- Lưu bản ghi dạng tài liệu (JSON hoặc BSON), có cấu trúc lồng nhau.
- Phù hợp dữ liệu có cấu trúc thay đổi, hoặc đối tượng gắn với nhau theo đơn vị tự nhiên (hồ sơ người dùng, danh mục sản phẩm, nội dung).
- Mở rộng ngang bằng sharding, đọc và ghi trên replica set.

## C.2. Ưu điểm

- Lược đồ linh hoạt, dễ phát triển nhanh.
- Đọc một đối tượng đầy đủ trong một lần, không cần nối nhiều bảng.
- Sharding và replica set có sẵn.

## C.3. Nhược điểm

- Dữ liệu lặp lại nếu nhúng sai, dẫn đến cập nhật nhiều nơi.
- Nối giữa các collection hạn chế hơn quan hệ, và có chi phí.
- Lược đồ linh hoạt dễ sinh dữ liệu không đồng nhất nếu không có validation.
- Một tài liệu rất lớn làm chậm đọc và ghi, và có giới hạn kích thước (MongoDB giới hạn 16 MB mỗi tài liệu [Trí nhớ]).

## C.4. Cấu hình quan trọng (MongoDB)

Các tham số được nêu trong tài liệu replication và sharding đã kiểm chứng:

| Cấu hình | Vai trò | Ghi chú |
|---|---|---|
| Replica set | Nhóm bản sao có một primary và nhiều secondary | Cơ bản cho sẵn sàng |
| `electionTimeoutMillis` | Thời gian chờ trước khi bầu primary mới | Mặc định 10 giây; giảm thì nhanh phát hiện lỗi nhưng dễ bầu không cần thiết |
| Write concern `w` | Số node phải xác nhận ghi | `majority` an toàn hơn trước chuyển đổi primary |
| Read preference | Đọc từ primary hay secondary | Mặc định là primary |
| Read concern | Mức đảm bảo dữ liệu đọc | `local` có thể đọc dữ liệu chưa được xác nhận |
| Shard key | Khóa phân mảnh collection | Quyết định phân phối và hiệu năng; khó đổi, dù đã có reshard từ 5.0 |

[Đã kiểm chứng: trang replication và sharding của MongoDB 9.0]

## C.5. Câu lệnh cần chú ý (mongosh)

```javascript
// Tạo chỉ mục trước khi truy vấn thường xuyên
db.orders.createIndex({ khach_id: 1, ngay: -1 })

// Xem kế hoạch truy vấn
db.orders.find({ khach_id: 42 }).sort({ ngay: -1 }).explain("executionStats")

// Cập nhật một trường, không ghi đè cả tài liệu
db.orders.updateOne({ _id: "don-101" }, { $set: { trang_thai: "da_giao" } })

// Ghi có xác nhận từ đa số node
db.orders.insertOne(
  { _id: "don-102", khach_id: 42, tong_tien: 750000 },
  { writeConcern: { w: "majority" } }
)

// Đa tài liệu: giao dịch
const phien = db.getMongo().startSession()
phien.startTransaction()
// ... thao tác với phien ...
phien.commitTransaction()
```

**Cần chú ý:**
- `updateOne` với `$set` khác với thay thế toàn bộ tài liệu. Hiểu nhầm có thể làm mất trường.
- Truy vấn không có chỉ mục là quét toàn bộ collection, nguyên nhân chậm phổ biến.
- Tránh `$where` và các biểu thức JavaScript trong truy vấn, vừa chậm vừa có rủi ro bảo mật. [Trí nhớ]
- Giao dịch có đọc phải dùng read preference `primary`. [Đã kiểm chứng]

---

# PHẦN D. MÔ HÌNH KHÓA-GIÁ TRỊ (KEY-VALUE)

**Hệ thống tiêu biểu:** Redis, Amazon DynamoDB, Memcached (chỉ cache), etcd (khóa-giá trị có tính nhất quán mạnh cho cấu hình và điều phối).

## D.1. Vai trò và chức năng

- Tra cứu cực nhanh theo khóa.
- Dùng làm cache, lưu phiên đăng nhập, bộ đếm, hàng đợi đơn giản, khóa phân tán.
- Dữ liệu trong giá trị thường là không cấu trúc hoặc cấu trúc đơn giản (chuỗi, danh sách, tập hợp, hash trong Redis).

## D.2. Ưu điểm

- Độ trễ rất thấp, thông lượng cao.
- Mô hình đơn giản, dễ mở rộng.
- Nhiều cấu trúc dữ liệu tiện ích (Redis).

## D.3. Nhược điểm

- Truy vấn theo giá trị hoặc theo quan hệ rất hạn chế.
- Nếu chạy trong bộ nhớ, dữ liệu có thể mất khi sự cố nếu không cấu hình bền vững.
- Dễ dùng sai thành cơ sở dữ liệu chính khi cần ràng buộc và truy vấn phức tạp.

## D.4. Cấu hình quan trọng (Redis)

Phần persistence đã được kiểm chứng từ tài liệu Redis mới nhất:

| Cấu hình | Vai trò | Ghi chú |
|---|---|---|
| `save <giây> <số thay đổi>` | Chụp ảnh RDB định kỳ | Ví dụ `save 60 1000`: chụp nếu có ít nhất 1000 thay đổi trong 60 giây. RDB là tệp nhỏ gọn, phù hợp sao lưu |
| `appendonly yes` | Bật AOF (nhật ký mọi lệnh ghi) | Bền vững hơn RDB |
| `appendfsync` | Tần suất đồng bộ đĩa của AOF | `always` an toàn nhất nhưng chậm; `everysec` là mặc định khuyến nghị (mất tối đa khoảng một giây dữ liệu); `no` nhanh nhưng không bền bằng |
| `maxmemory` | Giới hạn bộ nhớ | Cần đặt khi chạy làm cache |
| `maxmemory-policy` | Chính sách loại bỏ khi đầy | Ví dụ `allkeys-lru` khi dùng làm cache |

[Đã kiểm chứng: tài liệu persistence của Redis. Khuyến nghị theo tài liệu: dùng RDB để sao lưu và khởi động nhanh, AOF để giảm mất dữ liệu; dùng cả hai nếu cần mức an toàn gần PostgreSQL]

## D.5. Câu lệnh cần chú ý

```redis
SET phien:abc123 "user-42" EX 1800        # đặt khóa với hạn 30 phút
GET phien:abc123
INCR luot_xem:san_pham_7                  # bộ đếm nguyên tử
EXPIRE khoa 60
TTL khoa                                  # xem thời gian còn lại

SCAN 0 MATCH phien:* COUNT 100            # duyệt khóa theo lô
KEYS phien:*                              # TRÁNH trên hệ thống thật: quét toàn bộ, chặn server

CONFIG GET appendonly
CONFIG SET appendonly yes
BGSAVE                                    # chụp RDB nền
BGREWRITEAOF                              # viết lại AOF nền
INFO persistence                          # kiểm tra trạng thái lưu trữ
```

**Cần chú ý:**
- `KEYS` quét toàn bộ khóa và chặn server, không dùng trên hệ thống sản xuất. Dùng `SCAN`. [Trí nhớ]
- Với bật AOF trên hệ đang chạy, cần làm theo đúng quy trình trong tài liệu Redis (bật qua `CONFIG SET`, rồi cập nhật tệp cấu hình bằng `CONFIG REWRITE`). Nếu chỉ sửa tệp và khởi động lại, cấu hình có thể mất và dẫn đến mất dữ liệu. [Đã kiểm chứng]
- Redis 8.10 trở lên có họ lệnh `BACKUP` để sao lưu không cần dừng ghi: `BACKUP START`, `BACKUP LIST`, `BACKUP SEAL`, `BACKUP CLEANUP`. [Đã kiểm chứng: tài liệu persistence mới nhất]

---

# PHẦN E. MÔ HÌNH CỘT RỘNG (WIDE-COLUMN)

**Hệ thống tiêu biểu:** Apache Cassandra, ScyllaDB, Apache HBase, Google Bigtable.

## E.1. Vai trò và chức năng

- Lưu dữ liệu theo **khóa phân vùng (partition key)** và **khóa phân nhóm (clustering key)**.
- Thiết kế cho ghi lượng lớn và sẵn sàng cao trên nhiều node, không có node chính cố định (Cassandra).
- Dùng cho dữ liệu chuỗi thời gian quy mô lớn, nhật ký sự kiện, hồ sơ người dùng lớn, cá nhân hóa.

## E.2. Ưu điểm

- Mở rộng tuyến tính khi thêm node.
- Ghi nhanh, chịu được lỗi node.
- Có thể cấu hình mức nhất quán theo từng truy vấn.

## E.3. Nhược điểm

- Thiết kế bảng phải làm theo truy vấn: một bảng cho một kiểu truy vấn, dữ liệu có thể lặp lại.
- Không hỗ trợ nối bảng và ràng buộc khóa ngoại như quan hệ.
- Transaction hạn chế, lightweight transaction (LWT) chậm.
- Xóa dữ liệu tạo "bia mộ" (tombstone), làm chậm đọc nếu nhiều. [Trí nhớ]

## E.4. Cấu hình quan trọng (Cassandra)

| Cấu hình | Vai trò | Ghi chú |
|---|---|---|
| Replication strategy | Số bản sao và phân bố giữa các datacenter | `NetworkTopologyStrategy` cho nhiều datacenter [Trí nhớ] |
| Consistency level | Số bản sao phải phản hồi khi đọc hoặc ghi | Ví dụ `ONE`, `QUORUM`, `LOCAL_QUORUM`, `ALL` [Trí nhớ; tài liệu DML hiện tại không nêu chi tiết] |
| Partition key | Quyết định node lưu dữ liệu | Chọn để phân bố đều và tránh phân vùng quá lớn [Trí nhớ] |
| Clustering key | Thứ tự trong phân vùng | Phục vụ truy vấn theo khoảng |
| TTL | Tự động xóa sau thời gian | Tạo tombstone khi hết hạn [Trí nhớ] |

## E.5. Câu lệnh cần chú ý (CQL)

```sql
-- Bảng theo truy vấn: lấy đơn của khách theo thời gian giảm dần
CREATE TABLE don_theo_khach (
    khach_id   uuid,
    ngay       timestamp,
    don_id     uuid,
    tong_tien  decimal,
    PRIMARY KEY ((khach_id), ngay, don_id)
) WITH CLUSTERING ORDER BY (ngay DESC, don_id ASC);

-- INSERT là upsert: ghi đè nếu khóa đã tồn tại, không báo lỗi
INSERT INTO don_theo_khach (khach_id, ngay, don_id, tong_tien)
VALUES (?, ?, ?, ?);

-- Batch có chủ đích: nhóm thao tác cùng phân vùng
BEGIN BATCH
  INSERT INTO don_theo_khach (...) VALUES (...);
  UPDATE khach_thong_ke SET so_don = so_don + 1 WHERE khach_id = ?;
APPLY BATCH;

-- Lightweight transaction: chậm, dùng tiết kiệm
INSERT INTO khach (khach_id, email) VALUES (?, ?) IF NOT EXISTS;
```

**Cần chú ý (theo tài liệu CQL DML hiện tại):** [Đã kiểm chứng]
- `INSERT` và `UPDATE` không kiểm tra dữ liệu đã tồn tại. Đây là hành vi upsert.
- `BATCH` nhằm giảm số lượt đi lại mạng và đảm bảo cô lập trong một phân vùng. **Không phải giao dịch kiểu SQL.** Batch đa phân vùng có chi phí ghi nhật ký lô.
- Thứ tự áp dụng trong batch không đảm bảo nếu không có timestamp riêng.
- `IF NOT EXISTS` và điều kiện `IF` dùng Paxos, chi phí cao, dùng có chọn lọc.
- Batch counter không idempotent.
- Tránh `ALLOW FILTERING` và quét toàn bảng. [Trí nhớ]

---

# PHẦN F. MÔ HÌNH ĐỒ THỊ (GRAPH)

**Hệ thống tiêu biểu:** Neo4j, Amazon Neptune, ArangoDB (đa mô hình), JanusGraph.

## F.1. Vai trò và chức năng

- Lưu **đỉnh** (thực thể) và **cạnh** (quan hệ), mỗi bên có thuộc tính.
- Truy vấn đường đi, láng giềng, mẫu quan hệ nhiều bước.
- Dùng cho gợi ý, phát hiện gian lận qua mạng quan hệ, quản lý quyền truy cập phức tạp, phân tích mạng xã hội, đồ thị tri thức.

## F.2. Ưu điểm

- Truy vấn quan hệ nhiều bước nhanh và tự nhiên.
- Mô hình gần với nghiệp vụ có quan hệ phức tạp.

## F.3. Nhược điểm

- Không phù hợp cho dữ liệu chủ yếu là bảng phẳng.
- Phân mảnh đồ thị khó hơn phân mảnh bảng, vì cạnh có thể nối các phần không cùng máy. [Trí nhớ]
- Hệ sinh thái công cụ và nhân lực ít hơn quan hệ.

## F.4. Cấu hình quan trọng (Neo4j)

- Chỉ mục trên thuộc tính thường dùng để tìm điểm bắt đầu.
- Bộ nhớ heap và bộ nhớ đệm trang, cần đặt theo kích thước đồ thị. [Trí nhớ]
- Cụm (cluster) để sẵn sàng cao. [Trí nhớ]

## F.5. Câu lệnh cần chú ý (Cypher)

```cypher
// Tạo chỉ mục để tìm điểm bắt đầu nhanh
CREATE INDEX nguoi_dung_email IF NOT EXISTS FOR (u:NguoiDung) ON (u.email);

// Tìm bạn của bạn, giới hạn số kết quả
MATCH (a:NguoiDung {email: $email})-[:BAN]->(b)-[:BAN]->(c)
WHERE c <> a
RETURN DISTINCT c.ten LIMIT 20;

// MERGE: tạo nếu chưa có, tránh trùng
MERGE (u:NguoiDung {email: $email})
ON CREATE SET u.ten = $ten

// Xem kế hoạch
EXPLAIN MATCH (u:NguoiDung {email: $email}) RETURN u;
PROFILE MATCH (u:NguoiDung {email: $email}) RETURN u;
```

**Cần chú ý:**
- Không có chỉ mục trên thuộc tính tìm kiếm thì bước đầu sẽ quét toàn đồ thị. [Trí nhớ]
- `MATCH` không giới hạn có thể trả về rất nhiều kết quả, luôn dùng `LIMIT` khi khám phá.
- Đường đi dài không giới hạn có thể bùng nổ về số lượng. Giới hạn độ sâu. [Trí nhớ]

---

# PHẦN G. MÔ HÌNH CHUỖI THỜI GIAN (TIME-SERIES)

**Hệ thống tiêu biểu:** InfluxDB, TimescaleDB (tiện ích mở rộng của PostgreSQL), Prometheus (giám sát), QuestDB.

## G.1. Vai trò và chức năng

- Lưu các điểm dữ liệu có dấu thời gian: số liệu cảm biến, metric hạ tầng, giao dịch tần suất cao.
- Truy vấn theo khoảng thời gian, tổng hợp theo cửa sổ (ví dụ trung bình mỗi phút).
- Tự động giữ dữ liệu trong thời hạn, và hạ độ phân giải dữ liệu cũ (downsampling). [Trí nhớ]

## G.2. Ưu điểm

- Ghi nhanh với khối lượng lớn.
- Nén tốt dữ liệu theo thời gian.
- Truy vấn theo cửa sổ thời gian hiệu quả.

## G.3. Nhược điểm

- Không phù hợp cho dữ liệu không có trục thời gian.
- Cập nhật và xóa điểm riêng lẻ thường kém hiệu quả. [Trí nhớ]
- Số lượng chuỗi (cardinality) cao, tức quá nhiều nhãn duy nhất, làm tốn tài nguyên. [Trí nhớ]

## G.4. Cấu hình quan trọng

- **Chính sách lưu giữ (retention):** thời gian giữ dữ liệu. [Trí nhớ]
- **Downsampling:** tạo bản tổng hợp dữ liệu cũ. [Trí nhớ]
- **Cardinality:** hạn chế nhãn có giá trị không giới hạn như ID người dùng trong nhãn metric. [Trí nhớ]

## G.5. Câu lệnh cần chú ý

**TimescaleDB (PostgreSQL):**

```sql
CREATE EXTENSION IF NOT EXISTS timescaledb;
SELECT create_hypertable('cam_bien', 'thoi_gian');

-- Tổng hợp theo khung 5 phút
SELECT time_bucket('5 minutes', thoi_gian) AS khung, avg(nhiet_do)
FROM cam_bien
WHERE thoi_gian > now() - interval '1 day'
GROUP BY khung ORDER BY khung;
```

**Prometheus (truy vấn PromQL):**

```promql
rate(http_requests_total[5m])
```

**Cần chú ý:** cú pháp trên là minh họa theo tài liệu thường gặp. Kiểm tra lại với phiên bản của từng hệ thống. [Trí nhớ]

---

# PHẦN H. MÔ HÌNH TÌM KIẾM TOÀN VĂN (SEARCH)

**Hệ thống tiêu biểu:** Elasticsearch, OpenSearch, Apache Solr, Meilisearch.

## H.1. Vai trò và chức năng

- Lập **chỉ mục ngược (inverted index)**: từ khóa dẫn đến danh sách tài liệu chứa nó.
- Tìm kiếm toàn văn, xếp hạng theo độ liên quan, gợi ý, tìm kiếm mờ (fuzzy).
- Phân tích và tổng hợp (aggregation) trên tập lớn.

## H.2. Ưu điểm

- Tìm kiếm văn bản nhanh và linh hoạt, xử lý tiếng Việt cần bộ phân tích phù hợp.
- Mở rộng ngang và có hệ sinh thái công cụ đầy đủ.

## H.3. Nhược điểm

- Không phải nguồn dữ liệu chính đáng tin: nên đồng bộ từ cơ sở dữ liệu gốc. [Suy luận]
- Ghi chỉ mục có độ trễ ngắn, không tức thời. [Trí nhớ]
- Tốn bộ nhớ và đĩa.

## H.4. Cấu hình và câu lệnh cần chú ý (Elasticsearch)

```json
PUT san_pham
{
  "mappings": {
    "properties": {
      "ten":   { "type": "text", "analyzer": "standard" },
      "gia":   { "type": "integer" },
      "danh_muc": { "type": "keyword" }
    }
  }
}
```

```json
GET san_pham/_search
{
  "query": {
    "bool": {
      "must":   [{ "match": { "ten": "áo thun" } }],
      "filter": [{ "range": { "gia": { "lte": 500000 } } }]
    }
  }
}
```

**Cần chú ý:**
- `text` dùng để tìm kiếm toàn văn, `keyword` dùng để lọc chính xác. Chọn sai kiểu làm truy vấn không đúng. [Trí nhớ]
- Đặt điều kiện lọc trong `filter` thay vì `must`, vì `filter` không tính điểm và được cache. [Trí nhớ]
- Chọn số shard lúc tạo index, khó đổi sau. [Trí nhớ]

---

# PHẦN I. MÔ HÌNH VECTOR (VECTOR)

**Hệ thống tiêu biểu:** pgvector (tiện ích PostgreSQL), Milvus, Qdrant, Weaviate, Pinecone (dịch vụ quản lý), các thư viện chỉ mục như FAISS.

## I.1. Vai trò và chức năng

- Lưu **vector nhúng (embedding)**: biểu diễn số của văn bản, hình ảnh, âm thanh do mô hình học máy tạo ra.
- Tìm **láng giềng gần nhất (nearest neighbor)** theo khoảng cách, phục vụ tìm kiếm ngữ nghĩa, hệ thống gợi ý, truy xuất cho mô hình ngôn ngữ (RAG).

## I.2. Ưu điểm

- Tìm theo ý nghĩa, không chỉ theo từ khóa.
- Có thể tích hợp với cơ sở dữ liệu quan hệ (pgvector), giảm số hệ thống phải vận hành.

## I.3. Nhược điểm

- Tìm chính xác tuyệt đối đắt; các chỉ mục xấp xỉ đánh đổi độ chính xác lấy tốc độ. [Trí nhớ]
- Chất lượng kết quả phụ thuộc mô hình tạo embedding.
- Cập nhật và xây chỉ mục có chi phí, nhất là với tập lớn. [Trí nhớ]

## I.4. Cấu hình và câu lệnh cần chú ý (pgvector)

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE tai_lieu (
    id        bigserial PRIMARY KEY,
    noi_dung  text,
    embedding vector(1536)     -- số chiều phải khớp mô hình embedding
);

-- Chỉ mục xấp xỉ HNSW (đánh đổi giữa tốc độ, độ chính xác, bộ nhớ)
CREATE INDEX ON tai_lieu USING hnsw (embedding vector_cosine_ops);

-- Tìm 5 láng giềng gần nhất theo cosine
SELECT id, noi_dung
FROM tai_lieu
ORDER BY embedding <=> $1
LIMIT 5;
```

**Cần chú ý:**
- Số chiều phải khớp mô hình tạo embedding. Sai số chiều sẽ báo lỗi hoặc cho kết quả vô nghĩa. [Trí nhớ]
- Các loại chỉ mục phổ biến là HNSW và IVFFlat, với đánh đổi khác nhau về bộ nhớ, tốc độ xây, và độ chính xác. [Trí nhớ]
- Các toán tử khoảng cách (cosine, L2, tích trong) phải khớp với loại chỉ mục đã tạo. [Trí nhớ]

**Ghi chú về nguồn:** trang tài liệu pgvector không truy cập được trong lần tra cứu này (trả về 404). Nội dung trên cần đối chiếu với README của dự án pgvector.

---

# PHẦN J. MÔ HÌNH PHÂN TÍCH CỘT (COLUMNAR, OLAP)

**Hệ thống tiêu biểu:** ClickHouse, Google BigQuery, Snowflake, Amazon Redshift, DuckDB (nhúng trong tiến trình), Apache Druid, Apache Pinot.

## J.1. Vai trò và chức năng

- Lưu dữ liệu **theo cột** thay vì theo hàng, tối ưu cho truy vấn tổng hợp trên nhiều hàng và ít cột.
- Dùng cho báo cáo, phân tích, nhật ký lớn, bảng điều khiển.
- Thường không dùng làm nguồn giao dịch chính.

## J.2. Ưu điểm

- Truy vấn phân tích cực nhanh trên tập lớn: chỉ đọc các cột cần.
- Nén tốt, vì các giá trị cùng cột giống nhau.

## J.3. Nhược điểm

- Ghi từng hàng nhỏ kém hiệu quả; thường ghi theo lô.
- Cập nhật và xóa từng hàng tốn kém, tùy hệ thống. [Trí nhớ]
- Không phù hợp cho giao dịch nhỏ đồng thời nhiều.

## J.4. Cấu hình và câu lệnh cần chú ý (ClickHouse)

```sql
CREATE TABLE su_kien
(
    thoi_gian  DateTime,
    nguoi_dung UInt64,
    loai       LowCardinality(String),
    gia_tri    Float64
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(thoi_gian)
ORDER BY (loai, thoi_gian);

-- Thống kê theo ngày, chỉ đọc cột cần
SELECT toDate(thoi_gian) AS ngay, count() AS so_su_kien
FROM su_kien
WHERE thoi_gian >= now() - INTERVAL 30 DAY
GROUP BY ngay ORDER BY ngay;
```

**Cần chú ý:**
- `ORDER BY` và `PARTITION BY` quyết định hiệu năng truy vấn và cách xóa dữ liệu cũ. Chọn theo truy vấn thường gặp. [Trí nhớ]
- Tránh cập nhật từng hàng với số lượng lớn. Ưu tiên ghi lô. [Trí nhớ]
- `LowCardinality` phù hợp cột có ít giá trị khác nhau. [Trí nhớ]

---

# PHẦN K. MÔ HÌNH SQL PHÂN TÁN (NEWSQL)

**Hệ thống tiêu biểu:** CockroachDB, Google Spanner, TiDB, YugabyteDB.

## K.1. Vai trò và chức năng

- Giữ giao diện SQL và giao dịch ACID như quan hệ, nhưng phân tán trên nhiều node và nhiều vùng.
- Tự phân mảnh dữ liệu và tự cân bằng tải.
- Nhắm đến nghiệp vụ cần cả quy mô ngang và nhất quán mạnh.

## K.2. Ưu điểm

- Giữ được SQL và giao dịch quen thuộc khi mở rộng.
- Sẵn sàng cao, tự phục hồi khi node hỏng.

## K.3. Nhược điểm

- Độ trễ giao dịch cao hơn cơ sở dữ liệu một máy, đặc biệt khi giao dịch trải qua nhiều vùng. [Suy luận từ PACELC]
- Cấu hình và vận hành phức tạp hơn.
- Một số tính năng SQL có thể khác hoặc chưa đầy đủ so với PostgreSQL. [Trí nhớ]

## K.4. Cấu hình và câu lệnh cần chú ý

- Chọn khóa chính để tránh **điểm nóng (hot spot)**, ví dụ dùng UUID thay vì số tăng dần, hoặc băm. [Trí nhớ]
- Đặt vùng và chính sách sao chép theo yêu cầu địa lý. [Trí nhớ]
- Giữ giao dịch ngắn và tránh giao dịch chạy lâu có tranh chấp.
- Kiểm tra cú pháp và hành vi trên đúng phiên bản vì các hệ này phát triển nhanh. [Suy luận]

---

# PHẦN L. MÔ HÌNH ĐA MÔ HÌNH (MULTI-MODEL)

**Hệ thống tiêu biểu:** Azure Cosmos DB, ArangoDB, PostgreSQL (với JSONB, mở rộng vector, time-series), Couchbase.

## L.1. Vai trò và chức năng

- Một hệ thống hỗ trợ nhiều mô hình dữ liệu: tài liệu, đồ thị, khóa-giá trị, cột.
- Giảm số lượng hệ thống cần vận hành.

## L.2. Ưu điểm

- Đơn giản hóa kiến trúc khi nhu cầu đa dạng nhưng quy mô vừa.
- Dữ liệu có thể truy vấn từ nhiều góc nhìn.

## L.3. Nhược điểm

- Mỗi mô hình thường không đạt hiệu năng tối ưu bằng hệ thống chuyên biệt.
- Hiểu rõ từng mô hình con để tránh dùng sai.

## L.4. Cấu hình và lưu ý

- Với PostgreSQL, `JSONB` có chỉ mục GIN:

```sql
CREATE INDEX idx_meta ON san_pham USING GIN (thuoc_tinh jsonb_path_ops);
SELECT * FROM san_pham WHERE thuoc_tinh @> '{"mau": "xanh"}';
```

[Trí nhớ]

**Nguyên tắc:** bắt đầu với một hệ thống đa mô hình khi quy mô nhỏ và nhu cầu chưa rõ. Tách dần khi một nhu cầu trở thành nút thắt.

---

# PHẦN M. SO SÁNH TỔNG HỢP VÀ CÁCH CHỌN

## M.1. Bảng so sánh

| Mô hình | Mô hình dữ liệu | Truy vấn mạnh | Phân tán | Nhất quán điển hình | Khối lượng tốt nhất |
|---|---|---|---|---|---|
| Quan hệ | Bảng, quan hệ | Nối, ràng buộc | Chủ yếu một máy, có sao chép | ACID mạnh | OLTP vừa và lớn |
| Tài liệu | JSON/BSON lồng nhau | Theo tài liệu, chỉ mục | Sharding có sẵn | Có thể cấu hình | OLTP, nội dung |
| Khóa-giá trị | Khóa và giá trị | Theo khóa | Thường có sẵn | Thường yếu hơn, cấu hình | Cache, phiên, đếm |
| Cột rộng | Phân vùng và phân nhóm | Theo khóa phân vùng | Rất tốt | Có thể cấu hình | Ghi lớn, sẵn sàng cao |
| Đồ thị | Đỉnh và cạnh | Đường đi, láng giềng | Khó hơn | Tùy hệ thống | Quan hệ phức tạp |
| Chuỗi thời gian | Điểm theo thời gian | Khoảng và tổng hợp | Tùy hệ thống | Thường ghi thêm | Metric, cảm biến |
| Tìm kiếm | Chỉ mục ngược | Toàn văn, xếp hạng | Có sẵn | Gần thời gian thực | Tìm kiếm, lọc |
| Vector | Embedding | Láng giềng gần nhất | Tùy hệ thống | Tùy hệ thống | Tìm ngữ nghĩa, RAG |
| Cột (OLAP) | Cột | Tổng hợp lớn | Có sẵn | Thường phân tích, ghi lô | Báo cáo, phân tích |
| NewSQL | Quan hệ phân tán | SQL, giao dịch | Rất tốt | ACID phân tán | Quy mô ngang với ACID |
| Đa mô hình | Nhiều | Phụ thuộc mô hình | Tùy hệ thống | Tùy hệ thống | Nhu cầu đa dạng, quy mô vừa |

## M.2. Cây quyết định đơn giản

```
Dữ liệu chủ yếu là bảng có quan hệ và cần ràng buộc chặt?
    → Quan hệ (PostgreSQL)

Cần quy mô ngang với ACID và SQL?
    → NewSQL

Dữ liệu là tài liệu có cấu trúc lồng và thay đổi thường xuyên?
    → Tài liệu (MongoDB)

Cần tra cứu theo khóa cực nhanh, cache, phiên?
    → Khóa-giá trị (Redis)

Ghi lượng rất lớn, sẵn sàng cao, truy vấn theo khóa phân vùng đã biết trước?
    → Cột rộng (Cassandra)

Quan hệ nhiều bước giữa thực thể là trọng tâm?
    → Đồ thị (Neo4j)

Dữ liệu là điểm theo thời gian, metric, cảm biến?
    → Chuỗi thời gian (TimescaleDB, InfluxDB)

Tìm kiếm văn bản, gợi ý, lọc phức tạp?
    → Tìm kiếm (Elasticsearch, OpenSearch)

Tìm theo ý nghĩa, hệ thống RAG?
    → Vector (pgvector, Qdrant)

Báo cáo và phân tích lượng lớn, ít ghi từng hàng?
    → Cột (ClickHouse, BigQuery)
```

## M.3. Nguyên tắc chọn

1. **Bắt đầu từ truy vấn và đảm bảo cần thiết**, không từ công nghệ được ưa thích.
2. **Ưu tiên quan hệ** khi chưa có lý do rõ ràng để chọn khác. Đa số ứng dụng được phục vụ tốt bởi PostgreSQL.
3. **Dùng nhiều hệ thống có chủ đích**, mỗi hệ thống một vai trò, và đồng bộ có kiểm soát.
4. **Đo trước khi quyết định** với dữ liệu gần thật.
5. **Đánh giá chi phí vận hành**, không chỉ hiệu năng. Số hệ thống càng nhiều, công vận hành càng lớn.

---

# PHẦN N. DANH SÁCH KIỂM TRA VẬN HÀNH

## N.1. Sao lưu và phục hồi

- [ ] Có sao lưu tự động, lưu ở nơi khác vật lý.
- [ ] Đã **thử khôi phục** từ bản sao lưu ít nhất định kỳ. Sao lưu chưa được thử khôi phục không đáng tin.
- [ ] Đo thời gian khôi phục thực tế (RTO) và lượng dữ liệu có thể mất (RPO).

## N.2. Bảo mật

- [ ] Bật xác thực, không để mặc định.
- [ ] Không mở cổng cơ sở dữ liệu ra Internet công khai.
- [ ] Mã hóa kết nối (TLS) và mã hóa lưu trữ khi cần.
- [ ] Phân quyền tối thiểu cho từng người dùng và ứng dụng.
- [ ] Không lưu mật khẩu trong mã nguồn.

## N.3. Giám sát

- [ ] Theo dõi độ trễ, số kết nối, dung lượng đĩa, bộ nhớ.
- [ ] Cảnh báo khi sao chép chậm hoặc replica lệch.
- [ ] Theo dõi truy vấn chậm.

## N.4. Thay đổi lược đồ

- [ ] Có công cụ migration có phiên bản.
- [ ] Thử migration trên bản sao trước khi áp dụng.
- [ ] Thay đổi trên bảng lớn lên kế hoạch, dùng công cụ không khóa khi có.

## N.5. Dung lượng

- [ ] Ước lượng tăng trưởng dữ liệu và tải.
- [ ] Đo lại định kỳ.

---

# PHẦN O. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| OLTP | Xử lý giao dịch trực tuyến: nhiều giao dịch nhỏ |
| OLAP | Xử lý phân tích: truy vấn tổng hợp lớn |
| Replica set / sao chép | Nhiều bản sao dữ liệu để sẵn sàng |
| Sharding / phân mảnh | Chia dữ liệu cho nhiều máy |
| Chỉ mục ngược | Từ khóa dẫn đến tài liệu chứa nó |
| Embedding | Vector số biểu diễn ý nghĩa |
| HNSW, IVFFlat | Hai loại chỉ mục xấp xỉ cho vector |
| Tombstone | Đánh dấu xóa, còn giữ tạm thời |
| Cardinality | Số giá trị duy nhất của một nhãn hoặc cột |
| Partition key | Khóa quyết định nơi lưu dữ liệu |
| Clustering key | Khóa quyết định thứ tự trong phân vùng |
| Lightweight transaction | Giao dịch nhẹ dùng điều kiện, chậm hơn |
| WAL, AOF, oplog | Nhật ký ghi của PostgreSQL, Redis, MongoDB |
| RPO, RTO | Lượng dữ liệu có thể mất, thời gian khôi phục tối đa |
| Hot spot | Điểm nóng: một phần dữ liệu nhận quá nhiều tải |
| Downsampling | Giảm độ phân giải dữ liệu cũ |

---

# PHẦN P. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## P.1. Nguồn đã đối chiếu trong lần này

- **Redis, Persistence** (phiên bản mới nhất, có mục BACKUP từ 8.10): https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/ [đã đọc]
- **Apache Cassandra 5.0, CQL DML**: https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html [đã đọc; trang hiển thị banner bản prerelease, có liên kết tới 6.0-alpha2]
- **MongoDB 9.0, Replication và Sharding**: https://www.mongodb.com/docs/manual/replication/ và https://www.mongodb.com/docs/manual/sharding/ [đã đọc trong tài liệu phân tán trước đó]
- **PostgreSQL, tài liệu mức cô lập và bộ lập kế hoạch**: đã đọc trong tài liệu cơ sở dữ liệu trước.

## P.2. Nguồn chưa truy cập được

- **pgvector** trên trang tài liệu PostgreSQL: trả về 404. Cần đối chiếu với README của dự án pgvector.

## P.3. Điểm cần xác minh

1. **Mức nhất quán và tombstone của Cassandra:** trang DML không nêu chi tiết. Cần đọc tài liệu về consistency và về tombstone.
2. **Giới hạn 16 MB của tài liệu MongoDB:** kiến thức nền, cần kiểm tra lại trong tài liệu.
3. **Các cấu hình PostgreSQL** (`shared_buffers`, `work_mem`, và các tham số khác): kiến thức nền, cần đối chiếu phiên bản.
4. **Cú pháp Neo4j, ClickHouse, Elasticsearch, TimescaleDB, PromQL:** minh họa theo kiến thức nền. Cần chạy thử hoặc đối chiếu tài liệu chính thức.
5. **Các hệ thống mục J, K, L, và phần lớn G:** mô tả khái quát, chưa đối chiếu nguồn trong lần này.
6. **Các khuyến nghị về thiết kế** (partition key, hot spot, chỉ mục) là nguyên tắc chung, không phải quy tắc tuyệt đối. Cần đo trên dữ liệu thật.
7. **Các câu lệnh** trong tài liệu chưa được chạy thử. Hãy kiểm tra trên môi trường thử nghiệm trước khi dùng trên dữ liệu thật, nhất là các lệnh xóa, cập nhật, và thay đổi lược đồ.

---

**Tóm tắt để tra nhanh:**
- **Quan hệ** cho nghiệp vụ cần ràng buộc chặt và giao dịch.
- **Tài liệu** cho dữ liệu lồng và thay đổi cấu trúc.
- **Khóa-giá trị** cho tra cứu nhanh, cache, phiên.
- **Cột rộng** cho ghi lớn và sẵn sàng cao, thiết kế theo truy vấn.
- **Đồ thị** cho quan hệ nhiều bước.
- **Chuỗi thời gian, tìm kiếm, vector, cột (OLAP)** cho các bài toán chuyên biệt.
- **NewSQL và đa mô hình** cho quy mô ngang hoặc nhu cầu đa dạng.
- Mọi lựa chọn cần kèm sao lưu đã thử khôi phục, giám sát, và bảo mật.
