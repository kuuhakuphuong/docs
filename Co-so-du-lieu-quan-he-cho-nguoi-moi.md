# Cơ sở dữ liệu quan hệ cho người mới

> **Viết cho:** người chưa học hoặc mới học về cơ sở dữ liệu, muốn hiểu từ những khái niệm đầu tiên đến cách hệ thống xử lý giao dịch và kiểm thử.
> **Ví dụ xuyên suốt:** một cửa hàng online nhỏ (khách hàng, sản phẩm, đơn hàng).
> **Hệ thống minh họa:** PostgreSQL. Các ví dụ SQL dùng cú pháp PostgreSQL.
> **Cách đọc:** đọc theo thứ tự lớp. Mục đánh dấu **[Nâng cao]** có thể bỏ qua ở lần đọc đầu.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu với nguồn chính thức; `[Trí nhớ]` = kiến thức nền, nên tra bản gốc trước khi trích dẫn; `[Suy luận]` = tôi tự suy ra hoặc minh họa.

---

## Mục lục

- **Lớp 0. Bắt đầu từ con số không**
  - 1. Cơ sở dữ liệu là gì, và vì sao không dùng Excel
  - 2. Bảng, hàng, cột
  - 3. Khóa chính và khóa ngoại
  - 4. Bốn thao tác cơ bản: SELECT, INSERT, UPDATE, DELETE
  - 5. Nối bảng (JOIN)
- **Lớp 1. Hệ thống gồm những phần nào**
  - 6. Sơ đồ ba khối
  - 7. Dữ liệu nằm ở đâu: đĩa, trang, bộ nhớ đệm
  - 8. Chỉ mục: mục lục của bảng
  - 9. Catalog: sổ mục lục của hệ thống
- **Lớp 2. Chuyện gì xảy ra khi chạy một truy vấn**
  - 10. Từ câu SQL đến kết quả
  - 11. Đọc kế hoạch thực thi bằng EXPLAIN
- **Lớp 3. Nhiều người cùng dùng một lúc**
  - 12. Giao dịch (transaction) và ACID
  - 13. Bài toán chiếc áo cuối cùng
  - 14. Mức cô lập (isolation level)
  - 15. [Nâng cao] MVCC
  - 16. [Nâng cao] Khóa, deadlock, SSI
- **Lớp 4. Khi máy bị sập giữa chừng**
  - 17. Nhật ký trước khi ghi (WAL), bằng ví dụ sổ thu chi
  - 18. [Nâng cao] Checkpoint và ARIES
- **Lớp 5. Kiểm thử một cơ sở dữ liệu ứng dụng**
  - 19. Ba câu hỏi cần trả lời khi kiểm thử
  - 20. Bài kiểm thử hoàn chỉnh cho cửa hàng
  - 21. Đo hiệu năng một cách đáng tin
- **Phụ lục**
  - A. Chuẩn hóa (normalization) theo ví dụ
  - B. Thuật ngữ
  - C. Nguồn tham khảo
  - D. Điểm cần xác minh

---

# LỚP 0. BẮT ĐẦU TỪ CON SỐ KHÔNG

## 1. Cơ sở dữ liệu là gì, và vì sao không dùng Excel

Giả sử bạn bán hàng online. Bạn cần lưu: ai là khách hàng, bán những sản phẩm nào, mỗi đơn hàng gồm những gì. Có thể dùng một file Excel. Nhưng khi:

- hai người cùng đặt món cuối cùng trong cùng một giây, ai được mua?
- máy tính bị tắt điện giữa lúc đang ghi đơn hàng, đơn đó còn hay mất?
- bạn muốn hỏi "khách nào đã mua sản phẩm A trong tháng 3?" trên 10 triệu dòng,

thì file Excel không trả lời được đáng tin cậy. **Cơ sở dữ liệu (database)** là phần mềm lo việc lưu, tìm, sửa dữ liệu, và đảm bảo ba điều: dữ liệu đúng, không mất, và nhiều người dùng cùng lúc không làm hỏng nhau.

**Hệ quản trị cơ sở dữ liệu (DBMS)** là phần mềm như PostgreSQL, MySQL, SQL Server. Bạn nói chuyện với nó bằng **SQL** (Structured Query Language).

## 2. Bảng, hàng, cột

Dữ liệu được tổ chức thành **bảng** (table). Ví dụ bảng khách hàng:

```
khach_hang
+----+-----------+------------------------+
| id | ten       | email                  |
+----+-----------+------------------------+
|  1 | An        | an@example.com         |
|  2 | Bình      | binh@example.com       |
|  3 | Chi       | chi@example.com        |
+----+-----------+------------------------+
```

- **Cột (column)** là một thuộc tính, ví dụ `ten`, `email`. Mỗi cột có **kiểu dữ liệu** (số, chữ, ngày).
- **Hàng (row)** là một bản ghi, ví dụ khách số 1 là An.

Trong thuật ngữ lý thuyết, hàng còn gọi là **bộ (tuple)**, cột là **thuộc tính (attribute)**, bảng là **quan hệ (relation)**. Mô hình này do E. F. Codd đề xuất năm 1970. [Đã kiểm chứng]

Tạo bảng:

```sql
CREATE TABLE khach_hang (
    id     INTEGER PRIMARY KEY,
    ten    TEXT NOT NULL,
    email  TEXT UNIQUE NOT NULL
);
```

Ý nghĩa: `NOT NULL` nghĩa là bắt buộc có giá trị, `UNIQUE` nghĩa là không được trùng.

## 3. Khóa chính và khóa ngoại

- **Khóa chính (primary key):** cột định danh duy nhất cho mỗi hàng. Trong bảng `khach_hang`, cột `id` là khóa chính.
- **Khóa ngoại (foreign key):** cột trỏ đến khóa chính của bảng khác. Đây là cách các bảng liên kết với nhau.

Bảng sản phẩm và đơn hàng:

```sql
CREATE TABLE san_pham (
    id        INTEGER PRIMARY KEY,
    ten       TEXT NOT NULL,
    gia       NUMERIC(12, 0) NOT NULL CHECK (gia >= 0),
    ton_kho   INTEGER NOT NULL CHECK (ton_kho >= 0)
);

CREATE TABLE don_hang (
    id          INTEGER PRIMARY KEY,
    khach_id    INTEGER NOT NULL REFERENCES khach_hang(id),
    ngay        DATE NOT NULL,
    tong_tien   NUMERIC(12, 0) NOT NULL DEFAULT 0
);

CREATE TABLE chi_tiet_don (
    don_id       INTEGER NOT NULL REFERENCES don_hang(id),
    san_pham_id  INTEGER NOT NULL REFERENCES san_pham(id),
    so_luong     INTEGER NOT NULL CHECK (so_luong > 0),
    gia_luc_mua  NUMERIC(12, 0) NOT NULL,
    PRIMARY KEY (don_id, san_pham_id)
);
```

Điểm quan trọng:
- `REFERENCES` là khóa ngoại: hệ thống **từ chối** đơn hàng có `khach_id` không tồn tại. Đây là cách bảo vệ tính đúng đắn ngay trong cơ sở dữ liệu, không để việc đó cho ứng dụng.
- `CHECK (ton_kho >= 0)` là ràng buộc: không cho tồn kho âm.
- `gia_luc_mua` lưu giá tại thời điểm mua. Nếu chỉ lưu tham chiếu đến `san_pham.gia`, khi giá đổi, lịch sử đơn cũ sẽ sai.

**Ràng buộc toàn vẹn (integrity constraint):** các quy tắc như khóa, `NOT NULL`, `CHECK`, khóa ngoại. Mục đích là để cơ sở dữ liệu tự từ chối dữ liệu sai. [Đã kiểm chứng: khái niệm chuẩn trong SQL]

## 4. Bốn thao tác cơ bản

**Đọc (SELECT):**

```sql
SELECT ten, gia FROM san_pham WHERE gia < 500000;
```

**Thêm (INSERT):**

```sql
INSERT INTO khach_hang (id, ten, email) VALUES (4, 'Dũng', 'dung@example.com');
```

**Sửa (UPDATE):**

```sql
UPDATE san_pham SET gia = 450000 WHERE id = 7;
```

**Xóa (DELETE):**

```sql
DELETE FROM khach_hang WHERE id = 4;
```

**Cảnh báo cho người mới:** `UPDATE` và `DELETE` không có `WHERE` sẽ áp dụng cho **mọi hàng**. Luôn chạy `SELECT` với cùng điều kiện trước, để xem trước kết quả.

## 5. Nối bảng (JOIN)

Muốn biết khách An mua những gì, cần ghép ba bảng:

```sql
SELECT k.ten AS khach, sp.ten AS san_pham, c.so_luong
FROM khach_hang k
JOIN don_hang d        ON d.khach_id = k.id
JOIN chi_tiet_don c    ON c.don_id = d.id
JOIN san_pham sp      ON sp.id = c.san_pham_id
WHERE k.ten = 'An';
```

Cách đọc: với mỗi khách, tìm các đơn của khách đó; với mỗi đơn, tìm các dòng chi tiết; với mỗi dòng chi tiết, lấy tên sản phẩm. Nối là thao tác phổ biến nhất trong cơ sở dữ liệu quan hệ, và cũng là thao tác tốn kém nhất. Phần lớn việc tối ưu hóa ở lớp 2 là về cách nối.

---

# LỚP 1. HỆ THỐNG GỒM NHỮNG PHẦN NÀO

## 6. Sơ đồ ba khối

Khi bạn chạy một câu SQL, dữ liệu đi qua ba khối:

```
 ┌──────────────┐        ┌──────────────────────────┐        ┌──────────────┐
 │  Ứng dụng    │ ─SQL─► │  Engine (DBMS)           │ ─đọc/ghi─► │  Đĩa        │
 │  (web, app)  │ ◄─kết─ │  - phân tích, lập kế hoạch│ ◄────── │  (file, trang)│
 │              │   quả  │  - giao dịch, khóa       │        │              │
 └──────────────┘        │  - nhật ký (WAL)         │        └──────────────┘
                         └──────────────────────────┘
```

- **Ứng dụng:** trang web, app di động, hoặc công cụ như `psql`.
- **Engine:** phần "não" của hệ quản trị. Nó hiểu SQL, quyết định cách lấy dữ liệu, và đảm bảo các quy tắc giao dịch.
- **Đĩa:** nơi dữ liệu nằm lâu dài.

Sơ đồ này đơn giản hóa. Lớp 1 và các lớp sau sẽ mở rộng từng phần.

## 7. Dữ liệu nằm ở đâu: đĩa, trang, bộ nhớ đệm

**Đĩa chậm hơn bộ nhớ rất nhiều**, nên engine không đọc từng hàng từ đĩa. Thay vào đó:

- Dữ liệu được chia thành **trang (page)**, mỗi trang khoảng 8 KB trong PostgreSQL. Đọc và ghi theo trang. [Đã kiểm chứng: kích thước mặc định 8 KB, nguồn kỹ thuật PostgreSQL]
- Các trang được đưa vào **bộ nhớ đệm (buffer pool)**. Lần đọc sau nếu trang đã có trong bộ nhớ, không cần đọc đĩa.
- Khi bộ nhớ đầy, engine đẩy trang ít dùng nhất ra đĩa.

**Hệ quả thực tế:** truy vấn lần đầu thường chậm hơn lần sau, vì lần đầu phải đọc đĩa. Đây là lý do khi đo hiệu năng phải tách "nguội" và "ấm" (mục 21).

## 8. Chỉ mục: mục lục của bảng

Nếu không có chỉ mục, tìm khách có email `an@example.com` phải duyệt toàn bộ bảng, gọi là **quét tuần tự (sequential scan)**. Với triệu dòng, cách này chậm.

**Chỉ mục (index)** là một cấu trúc phụ, giống mục lục sách, giúp tìm nhanh:

```sql
CREATE INDEX idx_khach_email ON khach_hang(email);
```

Loại chỉ mục phổ biến nhất là **B-tree**: cây cân bằng, tìm theo bằng và theo khoảng đều nhanh. Mặc định của PostgreSQL là B-tree. [Đã kiểm chứng]

**Đánh đổi:**
- Đọc nhanh hơn.
- Ghi chậm hơn, vì mỗi lần thêm hoặc sửa hàng, chỉ mục cũng phải cập nhật.
- Tốn thêm bộ nhớ và đĩa.

**Nguyên tắc:** chỉ tạo chỉ mục cho cột thường xuyên dùng trong `WHERE`, `JOIN`, hoặc `ORDER BY`. Đừng đánh chỉ mục cho mọi cột.

## 9. Catalog: sổ mục lục của hệ thống

Catalog là các bảng nội bộ lưu thông tin về chính cơ sở dữ liệu: có những bảng nào, cột nào, kiểu gì, chỉ mục nào, và thống kê về dữ liệu.

Thống kê rất quan trọng. Lệnh `ANALYZE` thu thập thống kê như số hàng, giá trị phổ biến. Bộ lập kế hoạch dùng thống kê này để ước lượng chi phí. [Đã kiểm chứng: tài liệu PostgreSQL về bộ lập kế hoạch]

```sql
ANALYZE san_pham;
```

Khi thống kê cũ, bộ lập kế hoạch có thể chọn kế hoạch tệ. Đây là nguyên nhân phổ biến của truy vấn chậm đột ngột.

---

# LỚP 2. CHUYỆN GÌ XẢY RA KHI CHẠY MỘT TRUY VẤN

## 10. Từ câu SQL đến kết quả

```
SQL
 │
 ▼  1. Phân tích cú pháp (parse)     → kiểm tra đúng ngữ pháp
 │
 ▼  2. Phân giải (analyze)           → tên bảng, cột có tồn tại không? kiểu có khớp không?
 │
 ▼  3. Viết lại (rewrite)            → mở rộng view, áp quy tắc
 │
 ▼  4. Lập kế hoạch (plan)           → chọn cách lấy dữ liệu rẻ nhất
 │
 ▼  5. Thực thi (execute)            → chạy kế hoạch, sinh ra từng hàng
 │
 ▼  Kết quả
```

**Điểm then chốt là bước 4.** Cùng một câu SQL có thể được thực hiện bằng nhiều cách: quét toàn bảng, dùng chỉ mục, nối theo thứ tự này hoặc thứ tự khác. Engine ước lượng chi phí của từng cách và chọn cách rẻ nhất. Chi phí dựa trên thống kê, không phải thời gian chạy thực tế.

**Ví dụ trực quan:** tìm sản phẩm có giá dưới 500.000:
- Cách 1: đọc toàn bộ bảng `san_pham`, lọc từng hàng.
- Cách 2: nếu có chỉ mục trên `gia`, đi thẳng đến các giá trị dưới 500.000.

Nếu bảng có 20 hàng, hai cách gần như bằng nhau. Nếu bảng có 20 triệu hàng và chỉ 0,1% thỏa điều kiện, cách 2 nhanh hơn nhiều.

## 11. Đọc kế hoạch thực thi bằng EXPLAIN

`EXPLAIN` cho thấy engine dự định làm gì. `EXPLAIN ANALYZE` chạy thật và cho thấy kết quả thực tế.

```sql
EXPLAIN ANALYZE
SELECT ten, gia FROM san_pham WHERE gia < 500000;
```

Kết quả minh họa (định dạng có thể khác theo phiên bản):

```
Seq Scan on san_pham  (cost=0.00..1.05 rows=3 width=36)
                      (actual time=0.010..0.012 rows=3 loops=1)
  Filter: (gia < 500000)
  Rows Removed by Filter: 2
Planning Time: 0.080 ms
Execution Time: 0.030 ms
```

**Cách đọc từng phần:**
- `Seq Scan`: quét toàn bảng. Nếu có chỉ mục phù hợp, có thể thấy `Index Scan` hoặc `Bitmap Heap Scan`.
- `cost=0.00..1.05`: chi phí ước lượng. Đơn vị tương đối, không phải mili-giây.
- `rows=3`: số hàng **ước lượng**. Đem so với `actual rows` (số hàng thực tế). Nếu chênh lệch lớn, thống kê đang sai, và nên chạy `ANALYZE`.
- `Filter`: điều kiện áp dụng trên từng hàng.

**Quy trình chuẩn khi thấy truy vấn chậm:** chạy `EXPLAIN ANALYZE`, tìm nút tốn thời gian nhất, kiểm tra ước lượng hàng có đúng không, rồi mới quyết định tạo chỉ mục hoặc viết lại truy vấn. Đừng tạo chỉ mục theo cảm tính.

---

# LỚP 3. NHIỀU NGƯỜI DÙNG CÙNG LÚC

## 12. Giao dịch (transaction) và ACID

**Giao dịch** là một nhóm thao tác được xem như một đơn vị: hoặc làm hết, hoặc không làm gì.

```sql
BEGIN;
UPDATE san_pham SET ton_kho = ton_kho - 1 WHERE id = 7;
INSERT INTO don_hang (id, khach_id, ngay, tong_tien) VALUES (101, 1, '2026-10-21', 450000);
INSERT INTO chi_tiet_don (don_id, san_pham_id, so_luong, gia_luc_mua) VALUES (101, 7, 1, 450000);
COMMIT;
```

Nếu giữa chừng có lỗi, `ROLLBACK` hủy hết. Không thể có tình trạng trừ kho mà không có đơn hàng.

**ACID** là bốn tính chất mà giao dịch phải có: [Trí nhớ, định nghĩa chuẩn]

| Chữ | Tên | Ý nghĩa đơn giản |
|---|---|---|
| **A** | Atomicity (nguyên tử) | Làm hết hoặc không làm gì |
| **C** | Consistency (nhất quán) | Sau giao dịch, mọi ràng buộc vẫn đúng |
| **I** | Isolation (cô lập) | Giao dịch đồng thời không thấy trạng thái dở dang của nhau |
| **D** | Durability (bền vững) | Đã `COMMIT` thì không mất, kể cả khi sập điện |

## 13. Bài toán chiếc áo cuối cùng

Trong kho còn 1 chiếc áo (`ton_kho = 1`). Hai khách An và Bình cùng bấm "mua" trong một giây. Nếu cả hai cùng thực hiện như sau, có thể xảy ra lỗi:

```
Thời điểm   An (giao dịch A)                     Bình (giao dịch B)
   1        đọc ton_kho: 1 (đủ)
   2                                              đọc ton_kho: 1 (đủ)
   3        ghi ton_kho = 0, tạo đơn
   4                                              ghi ton_kho = 0, tạo đơn
```

Kết quả: hai đơn, nhưng kho chỉ có một chiếc. Đây là **lỗi mất cập nhật (lost update)**: ghi của A bị ghi đè bởi B.

**Cách sửa đơn giản nhất:** để chính câu lệnh cập nhật kiểm tra điều kiện:

```sql
UPDATE san_pham
SET ton_kho = ton_kho - 1
WHERE id = 7 AND ton_kho >= 1;
```

Kiểm tra số hàng bị ảnh hưởng: nếu bằng 0, nghĩa là hết hàng, và giao dịch phải hủy. Cách này an toàn vì thao tác kiểm tra và ghi là một bước.

**Hoặc khóa hàng khi đọc:**

```sql
SELECT ton_kho FROM san_pham WHERE id = 7 FOR UPDATE;
```

Câu này khóa hàng đến khi giao dịch kết thúc, người khác phải chờ.

## 14. Mức cô lập (isolation level)

Mức cô lập quyết định một giao dịch được thấy những gì do giao dịch khác để lại. Mỗi mức cho phép hoặc ngăn chặn các **dị thường** khác nhau. [Đã kiểm chứng: PostgreSQL 18, mục về mức cô lập]

| Dị thường | Ý nghĩa |
|---|---|
| **Dirty read** | Đọc dữ liệu của giao dịch chưa commit |
| **Non-repeatable read** | Đọc cùng một hàng hai lần, thấy hai giá trị khác nhau |
| **Phantom read** | Chạy cùng truy vấn hai lần, thấy tập hàng khác nhau |
| **Serialization anomaly** | Kết quả không khớp với bất kỳ thứ tự tuần tự nào |

| Mức | Dirty | Non-repeatable | Phantom | Serialization |
|---|---|---|---|---|
| Read committed (mặc định PostgreSQL) | Không | Có thể | Có thể | Có thể |
| Repeatable read | Không | Không | Không (trong PostgreSQL) | Có thể |
| Serializable | Không | Không | Không | Không |

**Lời khuyên thực hành:**
- Mặc định `READ COMMITTED` đủ cho phần lớn ứng dụng web.
- Khi có logic phức tạp (ví dụ đọc số dư rồi quyết định ghi), dùng `REPEATABLE READ` hoặc `SERIALIZABLE`, và **phải** sẵn sàng thử lại giao dịch khi bị lỗi (SQLSTATE `40001`). [Đã kiểm chứng]

Ví dụ đặt mức cô lập:

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- các câu lệnh
COMMIT;
```

## 15. [Nâng cao] MVCC

**MVCC (Multiversion Concurrency Control)** là cách PostgreSQL cho phép đọc không bị chặn bởi ghi. Mỗi lần `UPDATE`, engine không ghi đè hàng cũ, mà tạo một **phiên bản mới**. Mỗi giao dịch nhìn thấy một ảnh chụp (snapshot) các phiên bản đã commit tại thời điểm nó bắt đầu.

Hình dung như một cuốn sổ có nhiều trang: người đọc được xem trang tại thời điểm họ mở sổ, người ghi thêm trang mới.

**Hệ quả thực tế:**
- Đọc và ghi ít chặn nhau.
- Phiên bản cũ tích tụ, cần được dọn. PostgreSQL dùng `VACUUM` để dọn. [Đã kiểm chứng]
- Bảng có nhiều cập nhật có thể phình to (bloat) nếu `VACUUM` không kịp.

## 16. [Nâng cao] Khóa, deadlock, SSI

- **Khóa hàng (row lock):** `FOR UPDATE` như ở mục 13.
- **Deadlock:** giao dịch A chờ B, B chờ A. Hệ thống phát hiện và hủy một bên. Ứng dụng cần thử lại.
- **SSI (Serializable Snapshot Isolation):** cách PostgreSQL đạt mức Serializable. Hệ thống theo dõi các phụ thuộc đọc-ghi và hủy giao dịch nào gây dị thường. [Đã kiểm chứng]

**Quy tắc thiết kế:** giữ giao dịch ngắn. Giao dịch dài giữ khóa lâu, làm người khác phải chờ, và tăng khả năng deadlock.

---

# LỚP 4. KHI MÁY BỊ SẬP GIỮA CHỪNG

## 17. Nhật ký trước khi ghi (WAL), bằng ví dụ sổ thu chi

Hãy tưởng tượng một người kế toán có hai quyển sổ:
- **Sổ nhật ký:** ghi mọi thu chi theo thứ tự thời gian, ghi ngay lập tức.
- **Sổ cái:** tổng hợp số dư từng tài khoản, cập nhật chậm hơn.

Nếu mất điện giữa lúc cập nhật sổ cái, người kế toán mở sổ nhật ký ra và làm lại các mục chưa được tổng hợp. Đây là ý tưởng của **Write-Ahead Logging (WAL)**: **ghi nhật ký trước, ghi dữ liệu sau**. [Đã kiểm chứng: nguyên tắc WAL trong tài liệu nội bộ PostgreSQL, và khái niệm chuẩn]

Quy tắc:
1. Mọi thay đổi được ghi vào nhật ký (ghi tuần tự, nhanh) trước.
2. Chỉ khi nhật ký đã ghi xuống đĩa, trang dữ liệu mới được ghi.
3. Khi `COMMIT`, nhật ký của giao dịch phải bền trên đĩa, rồi mới báo thành công cho ứng dụng.

**Hệ quả:** nếu mất điện, khi khởi động lại, hệ thống đọc nhật ký để làm lại (redo) các giao dịch đã commit và hủy (undo) các giao dịch chưa commit. Đây là lý do `COMMIT` là một lời hứa: đã báo thành công thì dữ liệu không mất.

## 18. [Nâng cao] Checkpoint và ARIES

- **Checkpoint:** định kỳ ghi các trang bẩn xuống đĩa và đánh dấu điểm bắt đầu. Nhờ đó, khi phục hồi, không phải đọc lại toàn bộ nhật ký từ đầu. [Trí nhớ]
- **ARIES:** thuật toán phục hồi kinh điển của Mohan và cộng sự (1992), gồm ba pha: phân tích, làm lại, hoàn tác. [Trí nhớ, cần tra bản gốc]

Với người mới, chỉ cần nắm: **nhật ký giúp phục hồi, và checkpoint giúp phục hồi nhanh**.

---

# LỚP 5. KIỂM THỬ MỘT CƠ SỞ DỮ LIỆU ỨNG DỤNG

## 19. Ba câu hỏi cần trả lời khi kiểm thử

1. **Dữ liệu có đúng không?** Ràng buộc có chặn được dữ liệu sai không, truy vấn có trả kết quả đúng không?
2. **Giao dịch đồng thời có an toàn không?** Hai người cùng mua món cuối có bị bán trùng không?
3. **Khi sự cố xảy ra, dữ liệu có còn đúng không?** Giết tiến trình giữa chừng có làm mất đơn đã báo thành công không?

Mỗi câu hỏi cần một loại kiểm thử riêng.

## 20. Bài kiểm thử hoàn chỉnh cho cửa hàng

**Kiểm thử 1: ràng buộc chặn dữ liệu sai**

```sql
-- Phải thất bại: tồn kho âm
UPDATE san_pham SET ton_kho = -1 WHERE id = 7;

-- Phải thất bại: khách không tồn tại
INSERT INTO don_hang (id, khach_id, ngay, tong_tien) VALUES (999, 9999, '2026-10-21', 0);
```

Tiêu chí đạt: cả hai câu đều báo lỗi. Nếu một trong hai thành công, ràng buộc đang thiếu.

**Kiểm thử 2: hai người mua món cuối cùng**

Chuẩn bị: `san_pham` có id 7 với `ton_kho = 1`.

Chạy hai giao dịch song song (ví dụ mở hai cửa sổ `psql`, hoặc hai luồng ứng dụng), mỗi giao dịch chạy:

```sql
BEGIN;
UPDATE san_pham SET ton_kho = ton_kho - 1 WHERE id = 7 AND ton_kho >= 1;
-- kiểm tra số hàng cập nhật bằng 1 trong ứng dụng; nếu bằng 0, ROLLBACK
COMMIT;
```

Tiêu chí đạt: sau cả hai, `ton_kho = 0` và chỉ một giao dịch thành công. Lặp lại 100 lần với dữ liệu reset giữa các lần, và đếm số lần bán vượt tồn kho. Kết quả đúng phải là 0.

**Kiểm thử 3: sự cố giữa chừng**

1. Bắt đầu giao dịch mua hàng, thực hiện một số bước, chưa `COMMIT`.
2. Giết tiến trình (ví dụ đóng cửa sổ, hoặc dừng dịch vụ đột ngột).
3. Khởi động lại, kiểm tra: đơn chưa commit không tồn tại, tồn kho không bị trừ.

Lặp lại với nhiều điểm dừng khác nhau. Tiêu chí: không có trạng thái nửa vời.

**Kiểm thử 4: đối chiếu tổng**

Sau mỗi bài kiểm thử, kiểm tra một bất biến:

```sql
-- Tổng tiền đơn hàng phải bằng tổng chi tiết
SELECT d.id
FROM don_hang d
JOIN (SELECT don_id, SUM(so_luong * gia_luc_mua) AS tong
      FROM chi_tiet_don GROUP BY don_id) c ON c.don_id = d.id
WHERE d.tong_tien <> c.tong;
```

Kết quả phải rỗng. Nếu có dòng trả về, dữ liệu đã bị lệch.

**Nguyên tắc:** kiểm thử không chứng minh hệ thống đúng. Nó chỉ tìm được lỗi nếu kịch bản kích hoạt được lỗi. Không tìm thấy lỗi không có nghĩa là không có lỗi. [Suy luận]

## 21. Đo hiệu năng một cách đáng tin

**Quy tắc cơ bản:**
- Đo nhiều lần, báo cáo trung vị và phân vị (p95, p99), không chỉ trung bình.
- Tách lần chạy đầu (nguội, cache trống) và lần sau (ấm).
- Cố định dữ liệu, cấu hình, phần cứng. Ghi lại tất cả.
- Đừng đo khi máy đang chạy việc khác.

**Ví dụ quy trình:** muốn biết chỉ mục trên `don_hang(khach_id)` có giúp không:
1. Đo truy vấn "lịch sử mua của khách X" 50 lần, không có chỉ mục.
2. Tạo chỉ mục, chạy `ANALYZE`.
3. Đo lại 50 lần.
4. So sánh trung vị và p95. Nếu giảm rõ, giữ chỉ mục. Đồng thời đo thời gian ghi để xem có chậm đi không.

---

# PHỤ LỤC

## A. Chuẩn hóa (normalization) theo ví dụ

**Vấn đề:** nếu gộp mọi thứ vào một bảng:

```
don_hang_gop
+---------+-----------+-----------------+-----------------+-------+
| don_id  | khach_ten | khach_email     | san_pham        | so_luong |
+---------+-----------+-----------------+-----------------+-------+
| 101     | An        | an@example.com  | Áo thun         | 2     |
| 102     | An        | an@example.com  | Quần jean       | 1     |
```

Tên và email của An bị lặp lại ở mỗi đơn. Khi An đổi email, phải sửa nhiều chỗ, và nếu sót một chỗ thì dữ liệu mâu thuẫn. Đây là **dị thường cập nhật**.

**Giải pháp:** tách thành các bảng như ở mục 3. Thông tin khách chỉ nằm ở một chỗ, đơn hàng chỉ tham chiếu đến `khach_id`.

**Các dạng chuẩn (tóm tắt):**
- **1NF:** mỗi ô chỉ chứa một giá trị. Không lưu "Áo thun, Quần jean" trong một ô.
- **2NF và 3NF:** mọi thuộc tính không khóa phải phụ thuộc vào khóa chính, không phụ thuộc vào thuộc tính không khóa khác.
- **BCNF:** dạng chuẩn chặt hơn một chút, tránh một số mâu thuẫn còn sót của 3NF. [Trí nhớ]

**Thực hành:** đa số hệ thống chuẩn hóa đến 3NF hoặc BCNF. Chỉ phi chuẩn hóa khi đo được nút thắt cụ thể.

## B. Thuật ngữ

| Thuật ngữ | Giải thích ngắn |
|---|---|
| Bảng (table) / quan hệ | Tập các hàng có cùng cột |
| Hàng (row) / bộ (tuple) | Một bản ghi |
| Cột (column) / thuộc tính | Một trường dữ liệu |
| Khóa chính (primary key) | Định danh duy nhất của hàng |
| Khóa ngoại (foreign key) | Cột trỏ đến khóa của bảng khác |
| Ràng buộc (constraint) | Quy tắc bảo vệ dữ liệu |
| Chỉ mục (index) | Cấu trúc phụ để tìm nhanh |
| Trang (page) | Đơn vị đọc ghi đĩa |
| Bộ nhớ đệm (buffer pool) | Trang đã đọc, giữ trong RAM |
| Catalog | Siêu dữ liệu của hệ thống |
| Kế hoạch thực thi (execution plan) | Cách engine dự định lấy dữ liệu |
| Giao dịch (transaction) | Nhóm thao tác là một đơn vị |
| ACID | Nguyên tử, nhất quán, cô lập, bền vững |
| Mức cô lập (isolation level) | Mức độ một giao dịch thấy thay đổi của giao dịch khác |
| Dị thường (anomaly) | Kết quả sai do chạy đồng thời |
| MVCC | Điều khiển đồng thời bằng nhiều phiên bản |
| WAL | Ghi nhật ký trước khi ghi dữ liệu |
| Checkpoint | Điểm đồng bộ để rút ngắn phục hồi |
| Chuẩn hóa (normalization) | Tổ chức bảng để tránh dư thừa |

## C. Nguồn tham khảo

- E. F. Codd (1970). "A Relational Model of Data for Large Shared Data Banks." *Communications of the ACM*, 13(6), 377–387. PDF: https://web.eecs.umich.edu/~michjc/eecs584/Papers/codd_1970.pdf [đã tra cứu]
- Tài liệu PostgreSQL 18, mức cô lập giao dịch: https://www.postgresql.org/docs/current/transaction-iso.html [đã đọc]
- Tài liệu PostgreSQL 18, bộ lập kế hoạch: https://www.postgresql.org/docs/current/planner-optimizer.html [đã đọc]
- Tài liệu PostgreSQL 18, tổng quan nội bộ (mục 51). [đã tham chiếu]
- Mohan và cộng sự (1992), ARIES. [Trí nhớ]
- Gray, J. và Reuter, A. (1993). *Transaction Processing: Concepts and Techniques*. [Trí nhớ]

Tài liệu nâng cao đầy đủ hơn (kiến trúc chi tiết, phục hồi, kiểm thử nâng cao): xem [Co-so-du-lieu-quan-he.md](Co-so-du-lieu-quan-he.md).

## D. Điểm cần xác minh

1. Các câu SQL trong tài liệu được viết theo PostgreSQL và chưa được chạy thử trong lần này. Nên chạy trên một bản PostgreSQL để kiểm chứng.
2. Kết quả `EXPLAIN` ở mục 11 là minh họa, định dạng thực tế có thể khác theo phiên bản.
3. Định nghĩa ACID và ARIES dựa trên kiến thức nền, chưa đối chiếu với bản gốc.
4. Ví dụ bảng `don_hang_gop` ở phụ lục A là minh họa.
