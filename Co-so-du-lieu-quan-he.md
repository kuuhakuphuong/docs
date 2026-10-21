# Cơ sở dữ liệu quan hệ: Tài liệu nghiên cứu về kiến trúc, xử lý luồng và kiểm thử

> **Viết cho:** sinh viên và kỹ sư muốn hiểu cơ sở dữ liệu quan hệ ở mức nguyên lý và triển khai, theo cách tiếp cận nghiên cứu khoa học.
> **Ngày tra cứu:** 2026-10-21.
> **Ví dụ hệ thống chính:** PostgreSQL 18 (tài liệu chính thức, trang về cách lập kế hoạch và mức cô lập). Các nguyên lý được áp dụng chung cho đa số hệ quản trị.
> **Cách đọc nhãn:**
> - `[Đã kiểm chứng]`: đã đối chiếu với nguồn trong lần tra cứu này.
> - `[Trí nhớ]`: kiến thức nền, chưa đối chiếu với bản gốc.
> - `[Suy luận]`: tôi suy ra hoặc minh họa.

---

## Tóm tắt

Cơ sở dữ liệu quan hệ biểu diễn dữ liệu dưới dạng quan hệ (bảng) toán học, truy vấn bằng đại số quan hệ hoặc ngôn ngữ khai báo như SQL, và đảm bảo tính đúng đắn qua giao dịch có tính chất ACID. Tài liệu này trình bày (i) nền tảng lý thuyết từ Codd (1970), (ii) kiến trúc của một hệ quản trị hiện đại, (iii) luồng xử lý truy vấn và giao dịch, bao gồm điều khiển đồng thời, phục hồi và tối ưu hóa, (iv) phương pháp kiểm thử và đánh giá tính đúng đắn. Tài liệu nhấn mạnh rằng mỗi khẳng định về hành vi hệ thống cần được kiểm chứng bằng thử nghiệm có kiểm soát.

---

## Mục lục

- Phần A. Nền tảng lý thuyết
  - 1. Mô hình quan hệ
  - 2. Đại số quan hệ
  - 3. Ràng buộc toàn vẹn và khóa
  - 4. Lý thuyết chuẩn hóa
  - 5. SQL: ngôn ngữ khai báo và các khác biệt với lý thuyết
- Phần B. Kiến trúc hệ quản trị
  - 6. Các thành phần và vai trò
  - 7. Lưu trữ: trang, heap, buffer pool
  - 8. Chỉ mục
  - 9. Danh mục hệ thống (catalog)
- Phần C. Luồng xử lý truy vấn
  - 10. Từ chuỗi SQL đến kết quả
  - 11. Tối ưu hóa dựa trên chi phí
  - 12. Thuật toán nối (join)
  - 13. Thực thi: mô hình Volcano và song song
- Phần D. Giao dịch và điều khiển đồng thời
  - 14. ACID
  - 15. Mức cô lập và các dị thường
  - 16. MVCC
  - 17. Khóa, deadlock và các phương pháp khác
  - 18. Serializability và SSI
- Phần E. Phục hồi
  - 19. Write-ahead logging
  - 20. Checkpoint và phục hồi sau sự cố
  - 21. Sao lưu và nhân bản
- Phần F. Kiểm thử và đánh giá
  - 22. Các mức kiểm thử
  - 23. Kiểm thử tính đúng của truy vấn
  - 24. Kiểm thử tính đúng của giao dịch
  - 25. Kiểm thử phục hồi và lỗi
  - 26. Đo hiệu năng
  - 27. Quy trình thử nghiệm có kiểm soát
- Phần G. Thiết kế thực tế
  - 28. Chuẩn hóa so với phi chuẩn hóa
  - 29. Chỉ mục và chi phí
- Phần H. Thuật ngữ
- Phần I. Nguồn tham khảo
- Phần J. Điểm cần xác minh

---

# PHẦN A. NỀN TẢNG LÝ THUYẾT

## 1. Mô hình quan hệ

E. F. Codd đề xuất mô hình quan hệ năm 1970 trong bài "A Relational Model of Data for Large Shared Data Banks", đăng trên *Communications of the ACM*, tập 13, số 6, trang 377–387. [Đã kiểm chứng] Luận điểm chính: người dùng cần được che giấu khỏi cách tổ chức dữ liệu trong máy. Mô hình dựa trên:

- **Quan hệ n-ngôi (n-ary relation):** một tập các bộ (tuple) có cùng tập thuộc tính. Về hình thức, một quan hệ là tập con của tích Descartes của các miền.
- **Tính độc lập dữ liệu:** ứng dụng không phụ thuộc vào đường dẫn lưu trữ.
- **Ngôn ngữ con dữ liệu phổ quát:** truy vấn trên quan hệ, không phụ thuộc vào cấu trúc vật lý.

Hệ quả nền tảng: một quan hệ không có thứ tự bộ và không có bộ trùng lặp (theo lý thuyết thuần túy). SQL vi phạm điều này: bảng có thể có bộ trùng lặp và thứ tự kết quả chỉ được đảm bảo khi có `ORDER BY`. Đây là một khác biệt quan trọng sẽ quay lại ở mục 5.

## 2. Đại số quan hệ

Đại số quan hệ gồm các phép toán trên quan hệ, là nền để hiểu kế hoạch truy vấn. [Trí nhớ]

| Phép toán | Ký hiệu | Ý nghĩa |
|---|---|---|
| Chọn (selection) | σ | Lọc bộ theo điều kiện |
| Chiếu (projection) | π | Giữ một số thuộc tính |
| Tích Descartes | × | Ghép mọi cặp bộ |
| Nối (join) | ⋈ | Tích có điều kiện, tương đương σ(×) |
| Hợp, giao, hiệu | ∪, ∩, − | Trên quan hệ cùng lược đồ |
| Đổi tên | ρ | Đổi tên quan hệ hoặc thuộc tính |

**Lý do quan trọng:** các phép toán này có tính chất đại số, ví dụ đẩy phép chọn xuống sớm hơn phép nối không làm đổi kết quả. Bộ tối ưu hóa khai thác các đẳng thức này để viết lại truy vấn. [Suy luận]

## 3. Ràng buộc toàn vẹn và khóa

- **Khóa ứng viên (candidate key):** tập thuộc tính tối thiểu xác định duy nhất một bộ.
- **Khóa chính (primary key):** một khóa ứng viên được chọn.
- **Khóa ngoại (foreign key):** thuộc tính tham chiếu đến khóa của quan hệ khác; đảm bảo toàn vẹn tham chiếu.
- **Ràng buộc miền, `NOT NULL`, `CHECK`, `UNIQUE`:** ràng buộc cục bộ trên thuộc tính hoặc bộ.

**Null:** SQL dùng logic ba giá trị (đúng, sai, không xác định). Điều này làm đại số quan hệ phức tạp hơn lý thuyết thuần túy. [Trí nhớ]

## 4. Lý thuyết chuẩn hóa

Mục tiêu: loại bỏ dư thừa và dị thường khi cập nhật (chèn, xóa, sửa). Các khái niệm: phụ thuộc hàm (functional dependency, FD), dạng chuẩn. [Trí nhớ]

| Dạng chuẩn | Điều kiện chính |
|---|---|
| 1NF | Mọi thuộc tính giá trị nguyên tử |
| 2NF | 1NF và không có phụ thuộc bộ phận vào khóa ứng viên |
| 3NF | 2NF và không có phụ thuộc bắc cầu từ khóa qua thuộc tính không khóa |
| BCNF | Mọi xác định (determinant) đều là siêu khóa |
| 4NF | Không có phụ thuộc đa trị không tầm thường |

**Tính chất phân rã:** phân rã một quan hệ thành các quan hệ con cần đảm bảo **phân rã không mất mát (lossless join)** và, nếu có thể, **bảo toàn phụ thuộc (dependency preservation)**. Đánh đổi thường gặp: BCNF luôn đạt được phân rã không mất mát, nhưng có thể không bảo toàn phụ thuộc; 3NF luôn đạt được cả hai. [Trí nhớ]

## 5. SQL: ngôn ngữ khai báo và các khác biệt với lý thuyết

SQL là ngôn ngữ chuẩn cho hệ quản trị quan hệ, dựa trên mô hình của Codd nhưng có các khác biệt: [Trí nhớ]

| Lý thuyết | SQL |
|---|---|
| Quan hệ là tập, không trùng bộ | Bảng có thể có bộ trùng |
| Không có thứ tự | Thứ tự chỉ có khi `ORDER BY` |
| Logic hai giá trị | Logic ba giá trị với `NULL` |
| Phép toán thuần | Có phép tổng hợp, cửa sổ (window), đệ quy (`WITH RECURSIVE`) |

**Ý nghĩa cho người học:** khi thiết kế hoặc kiểm thử, cần nhớ rằng một truy vấn SQL có thể cho kết quả khác với đại số thuần, đặc biệt với `NULL` và trùng lặp. Đây là nguồn lỗi phổ biến.

---

# PHẦN B. KIẾN TRÚC HỆ QUẢN TRỊ

## 6. Các thành phần và vai trò

Đây là kiến trúc khái quát. Hầu hết hệ quản trị hiện đại có các thành phần tương tự, dù tên gọi khác nhau.

```
                   ┌────────────────────────────────────────────┐
  Client (SQL) ───►│ Giao thức kết nối (wire protocol)           │
                   ├────────────────────────────────────────────┤
                   │ Parser  →  Analyzer  →  Rewriter           │
                   ├────────────────────────────────────────────┤
                   │ Planner / Optimizer (chi phí, thống kê)    │
                   ├────────────────────────────────────────────┤
                   │ Executor (toán tử, song song)              │
                   ├──────────────────┬─────────────────────────┤
                   │ Transaction mgr  │ Lock / Snapshot mgr     │
                   ├──────────────────┴─────────────────────────┤
                   │ Access methods: heap, B-tree, hash, GIN... │
                   ├────────────────────────────────────────────┤
                   │ Buffer manager (buffer pool)               │
                   ├──────────────────┬─────────────────────────┤
                   │ WAL (nhật ký)    │ Storage (file, trang)   │
                   └──────────────────┴─────────────────────────┘
                   Catalog (siêu dữ liệu) nằm xuyên suốt các tầng
```

| Thành phần | Vai trò |
|---|---|
| Parser | Phân tích cú pháp SQL thành cây |
| Analyzer | Kiểm tra tên bảng, cột, kiểu dữ liệu; phân giải tham chiếu |
| Rewriter | Áp dụng quy tắc (ví dụ mở rộng view) |
| Planner / Optimizer | Chọn kế hoạch thực thi có chi phí ước lượng thấp nhất |
| Executor | Chạy kế hoạch, xử lý từng dòng |
| Transaction manager | Cấp định danh giao dịch, quản lý commit/abort |
| Concurrency control | Đảm bảo cô lập (khóa, MVCC) |
| Access methods | Cấu trúc truy cập: heap, B-tree, hash |
| Buffer manager | Cache trang trong bộ nhớ |
| WAL | Ghi nhật ký trước khi thay đổi dữ liệu, đảm bảo bền vững |
| Catalog | Siêu dữ liệu: bảng, cột, chỉ mục, thống kê |

[Trình tự parse → analyze → rewrite → plan → execute được PostgreSQL mô tả trong tài liệu nội bộ. Đã kiểm chứng: PostgreSQL 18, mục 51 "Overview of PostgreSQL Internals"]

## 7. Lưu trữ: trang, heap, buffer pool

- **Trang (page/block):** đơn vị I/O. PostgreSQL dùng trang 8 KB mặc định. [Trí nhớ]
- **Heap:** bảng được lưu dạng tập trang không có thứ tự, bộ được tìm qua định danh vật lý (ví dụ `ctid` trong PostgreSQL). [Trí nhớ]
- **Buffer pool:** vùng nhớ đệm trang. Trang được đọc vào buffer, sửa trong buffer, và ghi xuống đĩa sau. Chính sách thay thế (ví dụ clock sweep) quyết định trang nào bị đẩy ra. [Trí nhớ]
- **Tính đúng đắn:** trang đã sửa trong buffer nhưng chưa ghi xuống đĩa được bảo vệ bằng WAL (mục 19).

**Lý do thiết kế:** I/O đĩa chậm hơn bộ nhớ hàng nghìn lần, nên mọi thiết kế đều tìm cách giảm I/O ngẫu nhiên và tận dụng cache. [Suy luận]

## 8. Chỉ mục

| Loại | Đặc điểm | Dùng cho |
|---|---|---|
| **B+-tree** | Cân bằng, đọc theo khoảng, có thứ tự | Mặc định; so sánh, khoảng, sắp xếp |
| Hash | Tra cứu theo khóa bằng | Chỉ hỗ trợ so sánh bằng |
| GiST, SP-GiST, GIN, BRIN | Tổng quát: hình học, toàn văn, mảng, khối lớn | Dữ liệu đặc thù |

[B-tree là loại mặc định trong PostgreSQL; các loại khác được mô tả trong tài liệu chính thức. Đã kiểm chứng ở mức tên gọi]

**Chỉ mục che phủ (covering index):** chứa đủ cột để truy vấn không cần đọc heap. **Chỉ mục từng phần (partial index):** chỉ đánh chỉ mục cho bộ thỏa điều kiện. [Trí nhớ]

## 9. Danh mục hệ thống (catalog)

Catalog là các bảng siêu dữ liệu: bảng, cột, kiểu, chỉ mục, ràng buộc, và thống kê cho bộ tối ưu hóa. Trong PostgreSQL, thống kê được thu thập bởi `ANALYZE` và hiển thị qua `pg_stats`. [Đã kiểm chứng: nguồn tài liệu planner nêu thống kê từ `ANALYZE`, lưu trong `pg_statistic`]

---

# PHẦN C. LUỒNG XỬ LÝ TRUY VẤN

## 10. Từ chuỗi SQL đến kết quả

```
SQL ──► Parse (cây cú pháp)
    ──► Analyze (cây truy vấn có kiểu, đã phân giải tên)
    ──► Rewrite (mở rộng view, quy tắc)
    ──► Plan (chọn kế hoạch có chi phí thấp nhất)
    ──► Execute (duyệt cây toán tử, sinh dòng)
    ──► Kết quả
```

**Điểm then chốt:** bước Plan là nơi hiệu năng được quyết định. Hai truy vấn cho kết quả giống nhau có thể có hiệu năng khác nhau hàng nghìn lần tùy kế hoạch. [Suy luận]

## 11. Tối ưu hóa dựa trên chi phí

Bộ tối ưu hóa hiện đại có thể mô tả như sau. [Đã kiểm chứng: PostgreSQL 18, mục 51.5]

1. **Tìm đường (path):** với mỗi quan hệ, xét các cách truy cập. Luôn có quét tuần tự (sequential scan). Có thể dùng chỉ mục khi điều kiện khớp khóa chỉ mục và toán tử thuộc lớp toán tử của chỉ mục.
2. **Ước lượng chi phí:** dựa trên thống kê: số dòng, phân bố giá trị, giá trị phổ biến nhất, tương quan.
3. **Chọn đường rẻ nhất** và xây cây kế hoạch đầy đủ cho bộ thực thi.

**Vai trò của thống kê:** nếu thống kê sai, ước lượng số dòng sai, và kế hoạch có thể tệ. Đây là nguyên nhân phổ biến của truy vấn chậm đột ngột. [Suy luận]

**Tìm kiếm không gian kế hoạch:** với n quan hệ, số thứ tự nối tăng rất nhanh. Hệ thống dùng tìm kiếm gần đầy đủ cho số quan hệ nhỏ và heuristic cho số lớn. PostgreSQL dùng `geqo_threshold` để chuyển sang Genetic Query Optimizer khi số quan hệ đạt ngưỡng. [Đã kiểm chứng: nguồn có nêu cơ chế GEQO; giá trị mặc định 12 được nêu từ tài liệu tham số cấu hình, chưa mở trực tiếp trong lần này]

## 12. Thuật toán nối (join)

| Thuật toán | Cách hoạt động | Phù hợp khi |
|---|---|---|
| **Nested loop** | Với mỗi dòng bên trái, quét bên phải | Bên phải nhỏ hoặc có chỉ mục theo khóa nối |
| **Hash join** | Xây bảng băm từ bên phải, thăm dò bằng bên trái | Hai bên lớn, không có thứ tự, đẳng thức |
| **Merge join** | Sắp xếp hai bên theo khóa nối, quét song song | Dữ liệu đã có thứ tự, hoặc chỉ mục cung cấp thứ tự |

[Đã kiểm chứng: ba chiến lược được nêu trong tài liệu planner PostgreSQL]

**Thứ tự nối:** với nhiều bảng, bộ tối ưu ưu tiên nối các bảng có mệnh đề nối trong `WHERE`; nối tích Descartes (không điều kiện) chỉ xét khi không còn lựa chọn khác. [Đã kiểm chứng]

**Độ phức tạp ước lượng:** nested loop có chi phí khoảng O(|R|·|S|) nếu không có chỉ mục; hash join khoảng O(|R| + |S|) trung bình khi bảng băm vừa bộ nhớ; merge join có chi phí sắp xếp O(n log n). [Trí nhớ]

## 13. Thực thi: mô hình Volcano và song song

**Mô hình iterator (Volcano):** mỗi toán tử có ba hàm `open`, `next`, `close`. Dòng được kéo từ gốc xuống lá. Đây là mô hình phổ biến. [Trí nhớ: Graefe 1994]

**Hạn chế:** mỗi lần gọi `next` có chi phí hàm, và cache CPU kém hiệu quả với lô dòng nhỏ. Các hệ thống hiện đại dùng xử lý theo lô (vectorized) hoặc biên dịch truy vấn để giảm chi phí này. [Trí nhớ]

**Song song:** chia quét bảng thành khối, nhiều tiến trình xử lý song song, rồi gộp kết quả. Có thể áp dụng cho quét, nối băm, tổng hợp. [Trí nhớ]

---

# PHẦN D. GIAO DỊCH VÀ ĐIỀU KHIỂN ĐỒNG THỜI

## 14. ACID

| Tính chất | Ý nghĩa |
|---|---|
| **Atomicity** (nguyên tử) | Giao dịch hoặc hoàn tất toàn bộ, hoặc không để lại dấu vết |
| **Consistency** (nhất quán) | Giao dịch đưa cơ sở dữ liệu từ trạng thái hợp lệ sang trạng thái hợp lệ, theo ràng buộc |
| **Isolation** (cô lập) | Giao dịch đồng thời không nhìn thấy trạng thái trung gian của nhau (theo một mức cô lập) |
| **Durability** (bền vững) | Giao dịch đã commit được giữ lại kể cả khi sự cố |

[Trí nhớ; các định nghĩa chuẩn trong lý thuyết giao dịch]

**Lưu ý:** "Consistency" trong ACID là tính chất của ứng dụng kết hợp với ràng buộc của hệ thống; không phải cùng nghĩa với consistency trong hệ phân tán (xem CAP). [Suy luận]

## 15. Mức cô lập và các dị thường

Bốn mức chuẩn theo SQL và các dị thường mà mỗi mức cho phép: [Đã kiểm chứng: PostgreSQL 18, mục 13.2]

| Mức cô lập | Dirty read | Non-repeatable read | Phantom read | Serialization anomaly |
|---|---|---|---|---|
| Read uncommitted | Cho phép (PostgreSQL không cho phép thực sự) | Có thể | Có thể | Có thể |
| Read committed (mặc định PostgreSQL) | Không | Có thể | Có thể | Có thể |
| Repeatable read | Không | Không | Cho phép (PostgreSQL không cho phép) | Có thể |
| Serializable | Không | Không | Không | Không |

**Định nghĩa dị thường:**
- **Dirty read:** đọc dữ liệu do giao dịch chưa commit ghi.
- **Non-repeatable read:** đọc lại một dòng và thấy giá trị đã đổi do giao dịch khác commit.
- **Phantom read:** chạy lại truy vấn và thấy tập dòng thỏa điều kiện đã đổi.
- **Serialization anomaly:** kết quả commit đồng thời không tương ứng với bất kỳ thứ tự tuần tự nào.

**Ví dụ serialization anomaly** (từ tài liệu PostgreSQL): bảng `mytab` có lớp 1 với giá trị 10, 20 và lớp 2 với giá trị 100, 200. Giao dịch A tính tổng lớp 1 (30) rồi chèn 30 vào lớp 2. Giao dịch B tính tổng lớp 2 (300) rồi chèn 300 vào lớp 1. Dưới Repeatable Read cả hai có thể commit, nhưng không có thứ tự tuần tự nào cho kết quả này. Dưới Serializable, một giao dịch sẽ thất bại. [Đã kiểm chứng]

## 16. MVCC

**Multiversion Concurrency Control (MVCC):** mỗi thao tác ghi tạo phiên bản mới của dòng thay vì ghi đè. Mỗi giao dịch đọc một **snapshot** nhìn thấy các phiên bản đã commit tại thời điểm đó. [Đã kiểm chứng: nguồn PostgreSQL mô tả MVCC là nền của mọi mức cô lập]

**Lợi ích:** đọc không chặn ghi, và ghi không chặn đọc trong nhiều trường hợp.

**Chi phí:**
- Phiên bản cũ tích tụ, cần thu gom (trong PostgreSQL là `VACUUM`).
- Bảng và chỉ mục có thể phình to nếu thu gom không kịp.
- Bloat ảnh hưởng hiệu năng quét. [Trí nhớ]

**Khác biệt theo mức cô lập trong PostgreSQL:** Read Committed lấy snapshot mới cho mỗi câu lệnh; Repeatable Read lấy snapshot một lần cho cả giao dịch. Repeatable Read là **snapshot isolation**, mạnh hơn yêu cầu tối thiểu của chuẩn SQL nhưng vẫn cho phép một số dị thường tuần tự. [Đã kiểm chứng]

## 17. Khóa, deadlock và các phương pháp khác

**Khóa hai pha (Two-Phase Locking, 2PL):** giao dịch lấy khóa trong pha tăng, và giải phóng trong pha giảm; đảm bảo serializability. Biến thể nghiêm ngặt (strict 2PL) giữ khóa ghi đến commit để tránh đọc dữ liệu chưa commit. [Trí nhớ]

**Deadlock:** hai giao dịch chờ nhau. Hệ thống phát hiện bằng đồ thị chờ (wait-for graph) và hủy một giao dịch. [Trí nhớ]

**Phương pháp khác:**
- **Timestamp ordering:** gán thứ tự theo dấu thời gian; hủy giao dịch vi phạm thứ tự. [Trí nhớ]
- **Optimistic concurrency control (OCC):** thực hiện không khóa, kiểm tra xung đột khi commit. [Trí nhớ]
- **Khóa ở mức hàng, bảng, và khóa ý định (intention lock):** cho phép khóa phân cấp. [Trí nhớ]

## 18. Serializability và SSI

**Serializability:** kết quả đồng thời tương đương với một thứ tự tuần tự nào đó. Đây là tiêu chuẩn đúng đắn mạnh nhất cho giao dịch. [Trí nhớ]

**Serializable Snapshot Isolation (SSI):** PostgreSQL triển khai mức Serializable bằng SSI. Hệ thống dùng snapshot isolation, và thêm giám sát phụ thuộc đọc-ghi bằng **predicate lock** (hiển thị trong `pg_locks` với chế độ `SIReadLock`). Các khóa này không chặn và không gây deadlock; chúng chỉ dùng để phát hiện phụ thuộc nguy hiểm, và giao dịch gây dị thường bị hủy. Ứng dụng phải thử lại khi gặp lỗi (SQLSTATE `40001`). [Đã kiểm chứng]

**Thông báo lỗi tiêu biểu** (từ tài liệu PostgreSQL):
```
ERROR:  could not serialize access due to read/write dependencies among transactions
```

**Hệ quả thiết kế:** mức Serializable đòi hỏi ứng dụng có vòng lặp thử lại. Không có vòng lặp thì lỗi trở thành lỗi người dùng. [Suy luận]

---

# PHẦN E. PHỤC HỒI

## 19. Write-ahead logging

**Nguyên tắc WAL:** mọi thay đổi trang dữ liệu phải được ghi nhật ký xuống đĩa **trước** khi trang đó được ghi xuống dữ liệu chính. Giao dịch chỉ commit khi bản ghi commit đã bền trên đĩa. [Trí nhớ]

**Lợi ích:** ghi nhật ký là tuần tự (nhanh), và phục hồi có thể làm lại (redo) hoặc hoàn tác (undo) từ nhật ký.

**Các khái niệm:** LSN (Log Sequence Number), bản ghi redo, bản ghi undo, `fsync`. [Trí nhớ]

## 20. Checkpoint và phục hồi sau sự cố

- **Checkpoint:** ghi các trang bẩn xuống đĩa và đánh dấu điểm bắt đầu phục hồi, để giới hạn thời gian redo. [Trí nhớ]
- **Phục hồi (ARIES):** ba pha: phân tích (analysis), làm lại (redo), hoàn tác (undo). ARIES là thuật toán kinh điển của Mohan và cộng sự (1992). [Trí nhớ]

**Kiểm tra:** sau khi phục hồi, dữ liệu phải thỏa mọi giao dịch đã commit và không chứa dữ liệu của giao dịch chưa commit. Đây là tiêu chí kiểm thử cụ thể (mục 25).

## 21. Sao lưu và nhân bản

- **Sao lưu toàn phần và gia tăng:** sao lưu toàn bộ dữ liệu, và sao lưu nhật ký liên tục để phục hồi tới thời điểm bất kỳ (point-in-time recovery). [Trí nhớ]
- **Nhân bản vật lý:** bản sao đọc theo nhật ký WAL. [Trí nhớ]
- **Nhân bản logic:** truyền thay đổi theo bảng hoặc dòng, linh hoạt hơn nhưng phức tạp hơn. [Trí nhớ]
- **Đồng bộ và bất đồng bộ:** đồng bộ giữ dữ liệu bền khi mất máy chủ, nhưng tăng độ trễ commit. [Trí nhớ]

---

# PHẦN F. KIỂM THỬ VÀ ĐÁNH GIÁ

Mục này là phần cốt lõi cho việc "kiểm tra". Mỗi lớp kiểm thử có câu hỏi riêng.

## 22. Các mức kiểm thử

| Mức | Câu hỏi | Ví dụ |
|---|---|---|
| Đơn vị (unit) | Hàm, toán tử đúng không? | Hàm tính tổng với NULL |
| Hồi quy (regression) | Thay đổi có làm hỏng hành vi cũ không? | Bộ truy vấn cố định với kết quả đã biết |
| Tính đúng của truy vấn | Truy vấn có cho kết quả đúng ngữ nghĩa không? | So với đại số hoặc với một engine tham chiếu |
| Tính đúng của giao dịch | Dị thường có xảy ra không? | Kiểm tra theo mức cô lập |
| Phục hồi | Sau sự cố, dữ liệu còn đúng không? | Giết tiến trình giữa chừng, kiểm tra |
| Hiệu năng | Thời gian, thông lượng có đạt không? | Benchmark chuẩn |
| Bền vững | Hệ thống có ổn định dài hạn không? | Chạy liên tục, theo dõi bloat, rò rỉ |

## 23. Kiểm thử tính đúng của truy vấn

**Phương pháp 1: kiểm thử dựa trên bảng kết quả đã biết.** Tạo dữ liệu nhỏ, viết kết quả mong đợi bằng tay, so sánh. Phù hợp để kiểm tra `NULL`, `DISTINCT`, nối ngoài (outer join), nhóm (group by). [Suy luận]

**Phương pháp 2: so sánh chéo (differential testing).** Chạy cùng truy vấn trên hai hệ thống hoặc hai kế hoạch khác nhau, so kết quả. Đặc biệt hữu ích để kiểm tra bộ tối ưu hóa: cùng truy vấn nhưng ép kế hoạch khác nhau thì kết quả phải giống. [Suy luận]

**Phương pháp 3: kiểm thử ngẫu nhiên (property-based và fuzzing).** Sinh truy vấn ngẫu nhiên, so sánh với engine tham chiếu hoặc với tính chất đã biết (ví dụ `SELECT` với điều kiện `TRUE` không đổi kết quả). Các công cụ như SQLancer được dùng trong nghiên cứu để tìm lỗi logic trong các hệ quản trị. [Trí nhớ, cần xác minh tên công cụ và kết quả]

**Phương pháp 4: kiểm thử biến đổi đại số.** Áp dụng các đẳng thức đại số (đẩy chọn xuống, giao hoán nối) và kiểm tra kết quả không đổi. [Suy luận]

**Kiểm tra kế hoạch:** `EXPLAIN` và `EXPLAIN ANALYZE` cho thấy kế hoạch thực tế và số dòng thực tế so với ước lượng. Chênh lệch lớn giữa ước lượng và thực tế là dấu hiệu cần chạy `ANALYZE` hoặc xem xét lại thống kê. [Đã kiểm chứng: thống kê và kế hoạch được mô tả trong tài liệu PostgreSQL; cú pháp `EXPLAIN` là chuẩn của PostgreSQL, mức chi tiết cần đối chiếu]

## 24. Kiểm thử tính đúng của giao dịch

**Mục tiêu:** xác định xem hệ thống có tạo ra dị thường mà mức cô lập đã hứa hẹn không chặn không.

**Phương pháp:**
1. **Viết kịch bản đồng thời** có chủ đích, mô tả các giao dịch và thứ tự xen kẽ thao tác. Ví dụ: ví dụ serialization anomaly ở mục 15.
2. **Chạy với nhiều thứ tự xen kẽ** và ghi lại kết quả.
3. **Kiểm tra serializability** bằng cách so với mọi thứ tự tuần tự, hoặc bằng đồ thị phụ thuộc (dependency graph): đồ thị có chu trình nghĩa là không serializable.
4. **Kiểm tra ngẫu nhiên đa luồng:** nhiều luồng chạy giao dịch ngẫu nhiên trên tập khóa nhỏ (để tăng xung đột), ghi lịch sử, rồi phân tích lịch sử đó.

**Công cụ tham khảo:** Jepsen (kiểm thử hệ phân tán, có phân tích lịch sử) và Elle (kiểm tra serializability từ lịch sử giao dịch). [Trí nhớ, cần xác minh]

**Lưu ý:** kiểm thử chỉ tìm được lỗi nếu kịch bản kích hoạt được lỗi. Không có lỗi tìm được không chứng minh hệ thống đúng. [Suy luận]

## 25. Kiểm thử phục hồi và lỗi

**Phương pháp:**
- **Giết tiến trình** (kill -9) hoặc mất điện giả lập giữa các giao dịch; khởi động lại và kiểm tra:
  - mọi giao dịch đã báo commit còn nguyên;
  - không có dữ liệu của giao dịch chưa commit;
  - ràng buộc toàn vẹn còn đúng.
- **Lỗi I/O giả lập:** làm `fsync` thất bại, làm ghi đĩa bị ngắt. [Suy luận]
- **Kiểm tra toàn vẹn sau phục hồi:** so sánh tổng hoặc checksum với trạng thái mong đợi từ nhật ký ứng dụng.

**Tiêu chí thành công:** tất cả các điểm kiểm tra đều đạt, lặp lại nhiều lần với các điểm sự cố khác nhau.

## 26. Đo hiệu năng

**Nguyên tắc đo lường:**
- Tách khởi động nguội (cold) và ấm (warm).
- Đo nhiều lần, báo cáo trung bình, độ lệch chuẩn và phân vị (p50, p95, p99).
- Cố định cấu hình phần cứng, phiên bản, tham số, và tập dữ liệu.
- Tránh đo trên máy đang chạy việc khác.

**Các benchmark chuẩn:** TPC-C (giao dịch, OLTP) và TPC-H (phân tích, OLAP) là hai bộ chuẩn phổ biến của Transaction Processing Performance Council. [Trí nhớ, cần xác minh phiên bản]

**Chỉ số:** thông lượng (giao dịch/giây), độ trễ, tỷ lệ trúng cache buffer, số lần I/O mỗi truy vấn, kích thước bloat.

## 27. Quy trình thử nghiệm có kiểm soát

Để kết quả đáng tin, nên theo quy trình sau.

1. **Đặt câu hỏi và giả thuyết:** ví dụ "chỉ mục B-tree trên cột `customer_id` giảm thời gian truy vấn lọc theo khách hàng ít nhất 10 lần".
2. **Chuẩn bị môi trường:** cùng phần cứng, cùng phiên bản, cấu hình ghi trong tệp.
3. **Chuẩn bị dữ liệu:** sinh dữ liệu với phân bố thực tế (không chỉ đều), có tham số seed cố định.
4. **Xác định biến độc lập và phụ thuộc:** một biến thay đổi mỗi lần (ví dụ có chỉ mục hay không).
5. **Chạy lặp:** nhiều lần cho mỗi điều kiện, xen kẽ thứ tự để tránh thiên lệch.
6. **Thu thập số liệu:** từ `EXPLAIN ANALYZE`, và từ thống kê hệ thống.
7. **Phân tích:** báo cáo khoảng tin cậy, không chỉ giá trị trung bình.
8. **Kiểm soát thiên lệch:** cache, thứ tự chạy, nền.
9. **Ghi lại đầy đủ:** mã, cấu hình, dữ liệu, và kết quả để người khác tái lập được.

**Nguyên tắc kiểm chứng:** mỗi kết luận cần một thí nghiệm có thể sai. Nếu một thay đổi "có vẻ nhanh hơn" nhưng không có số đo lặp lại, đó chưa phải kết luận.

---

# PHẦN G. THIẾT KẾ THỰC TẾ

## 28. Chuẩn hóa so với phi chuẩn hóa

| Tiêu chí | Chuẩn hóa | Phi chuẩn hóa |
|---|---|---|
| Dư thừa | Thấp | Cao |
| Toàn vẹn cập nhật | Dễ đảm bảo | Khó hơn, cần trigger hoặc ứng dụng |
| Truy vấn đọc | Nhiều phép nối | Ít phép nối |
| Phù hợp | OLTP, dữ liệu thay đổi thường | OLAP, đọc nhiều, ghi ít |

**Nguyên tắc thực hành:** chuẩn hóa đến 3NF hoặc BCNF làm mặc định; chỉ phi chuẩn hóa khi có đo lường cho thấy nút thắt, và ghi rõ đánh đổi về toàn vẹn. [Suy luận]

## 29. Chỉ mục và chi phí

- Chỉ mục tăng tốc đọc nhưng làm chậm ghi và tốn bộ nhớ/đĩa.
- Chỉ mục không được sử dụng là chi phí thuần. Kiểm tra bằng thống kê sử dụng chỉ mục.
- Chỉ mục trên cột có độ chọn lọc thấp (ví dụ cờ đúng/sai với phân bố lệch) thường ít hiệu quả. [Suy luận]

**Quy trình:** đo truy vấn chậm, xem `EXPLAIN ANALYZE`, đề xuất chỉ mục, đo lại trên dữ liệu thật. Không thêm chỉ mục theo cảm tính.

---

# PHẦN H. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Quan hệ (relation) | Tập các bộ có cùng thuộc tính |
| Khóa ứng viên / chính / ngoại | Các loại khóa đảm bảo tính duy nhất và tham chiếu |
| Phụ thuộc hàm (FD) | X xác định Y nếu hai bộ bằng nhau trên X thì bằng nhau trên Y |
| Dạng chuẩn (NF) | Mức ràng buộc tránh dư thừa và dị thường |
| Đại số quan hệ | Tập phép toán trên quan hệ |
| Parser / Planner / Executor | Các giai đoạn xử lý truy vấn |
| Buffer pool | Vùng nhớ đệm trang |
| B+-tree | Cấu trúc chỉ mục cân bằng |
| ACID | Nguyên tử, nhất quán, cô lập, bền vững |
| Dị thường | Kết quả sai do đồng thời: dirty, non-repeatable, phantom, serialization |
| MVCC | Điều khiển đồng thời đa phiên bản |
| Snapshot | Ảnh chụp các phiên bản đã commit tại một thời điểm |
| 2PL | Khóa hai pha |
| SSI | Serializable Snapshot Isolation |
| Predicate lock | Khóa theo điều kiện, dùng để phát hiện phụ thuộc |
| WAL | Ghi nhật ký trước khi ghi dữ liệu |
| Checkpoint | Điểm đồng bộ để giới hạn phục hồi |
| ARIES | Thuật toán phục hồi kinh điển |
| Nested loop / Hash / Merge | Ba thuật toán nối |
| Volcano | Mô hình thực thi iterator |
| Bloat | Dữ liệu thừa do phiên bản cũ chưa thu gom |
| Serializability | Kết quả tương đương một thứ tự tuần tự |

---

# PHẦN I. NGUỒN THAM KHẢO

**Nền tảng lý thuyết:**
- Codd, E. F. (1970). "A Relational Model of Data for Large Shared Data Banks." *Communications of the ACM*, 13(6), 377–387. Bản PDF tại trường đại học: https://web.eecs.umich.edu/~michjc/eecs584/Papers/codd_1970.pdf [đã tra cứu, thông tin xuất bản đã đối chiếu từ nhiều nguồn]
- Các nguồn bản PDF khác: https://www.comp.nus.edu.sg/~lingtw/papers/codd.pdf và https://cs-people.bu.edu/mathan/reading-groups/papers-classics/codd.pdf [đã tra cứu]
- Bản mô tả trên lịch sử thông tin: https://historyofinformation.com/detail.php?id=94 [đã tra cứu]

**Tài liệu hệ thống (PostgreSQL 18):**
- Mức cô lập giao dịch: https://www.postgresql.org/docs/current/transaction-iso.html [đã đọc, phiên bản 18]
- Bộ lập kế hoạch: https://www.postgresql.org/docs/current/planner-optimizer.html [đã đọc, phiên bản 18, mục 51.5]
- Tổng quan nội bộ: mục 51 của tài liệu PostgreSQL 18 [đã tham chiếu]

**Tài liệu kinh điển (cần đối chiếu, chưa tra cứu trong lần này):**
- Selinger, P. G., et al. (1979). "Access Path Selection in a Relational Database Management System." SIGMOD. Bài gốc về tối ưu hóa dựa trên chi phí. [Trí nhớ]
- Gray, J. và Reuter, A. (1993). *Transaction Processing: Concepts and Techniques*. Morgan Kaufmann. [Trí nhớ]
- Mohan, C., et al. (1992). "ARIES: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging." *ACM TODS*. [Trí nhớ]
- Bernstein, P. A., Hadzilacos, V., Goodman, N. (1987). *Concurrency Control and Recovery in Database Systems*. Addison-Wesley. [Trí nhớ]
- Berenson, H., et al. (1995). "A Critique of ANSI SQL Isolation Levels." SIGMOD. Phê bình các mức cô lập chuẩn. [Trí nhớ]
- Cahill, M., Röhm, U., Fekete, A. (2008). "Serializable Isolation for Snapshot Databases." SIGMOD. Nền tảng SSI. [Trí nhớ]
- Graefe, G. (1994). "Volcano: An Extensible and Parallel Query Evaluation System." *IEEE TKDE*. [Trí nhớ]
- TPC-C, TPC-H: https://www.tpc.org/ [Trí nhớ]
- Jepsen: https://jepsen.io/ [Trí nhớ]
- Elle (kiểm tra serializability): tìm bài "Elle: Inferring Isolation Anomalies from Experimental Observations" của Kingsbury và Alvaro (2020). [Trí nhớ]

---

# PHẦN J. ĐIỂM CẦN XÁC MINH

1. Các tài liệu kinh điển ở Phần I đánh dấu `[Trí nhớ]` cần tra cứu bản gốc trước khi trích dẫn chính thức.
2. Giá trị mặc định `geqo_threshold` (12) cần đối chiếu với tài liệu cấu hình của phiên bản đang dùng.
3. Mô tả độ phức tạp của các thuật toán nối (mục 12) là ước lượng phổ biến; độ phức tạp thực tế phụ thuộc vào bộ nhớ, phân bố, và chi tiết cài đặt.
4. Tên và phạm vi của SQLancer, Elle, Jepsen cần xác minh bằng nguồn chính thức.
5. Phần lớn kiến trúc trong Phần B mô tả khái quát; chi tiết triển khai khác nhau giữa các hệ quản trị.
6. Chưa có số liệu thực nghiệm trong tài liệu này. Để có kết quả, cần chạy thí nghiệm theo quy trình ở mục 27 trên phần cứng của bạn.

---

**Tóm lược:**
- Mô hình quan hệ của Codd cung cấp ngữ nghĩa; SQL là lớp khai báo có khác biệt với lý thuyết (trùng bộ, NULL).
- Luồng truy vấn: parse → analyze → rewrite → plan (chi phí, thống kê) → execute.
- Giao dịch dựa trên ACID, mức cô lập quyết định dị thường nào được phép; PostgreSQL dùng MVCC và SSI.
- Phục hồi dựa trên WAL và checkpoint.
- Kiểm thử gồm: đối chiếu kết quả truy vấn, kịch bản đồng thời với kiểm tra serializability, kiểm thử phục hồi bằng sự cố giả lập, và đo hiệu năng có kiểm soát theo quy trình khoa học.
