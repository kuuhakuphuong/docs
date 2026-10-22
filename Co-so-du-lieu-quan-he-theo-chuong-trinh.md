# Cơ sở dữ liệu quan hệ: Tài liệu học tập chi tiết theo chương trình môn học

> **Phạm vi:** 8 chuyên đề của môn Cơ sở dữ liệu, kèm lý thuyết, ví dụ và bài tập tự kiểm tra.
> **Ví dụ xuyên suốt:** hệ thống quản lý ngân hàng đơn giản (khách hàng, tài khoản, giao dịch).
> **Ghi chú về cú pháp:** các ví dụ SQL viết theo chuẩn gần PostgreSQL. MySQL, SQL Server và Oracle có khác biệt nhỏ ở kiểu tự tăng (`AUTO_INCREMENT`, `IDENTITY`), hàm thời gian và cú pháp trigger.
> **Giáo trình tham khảo:** Garcia-Molina, Ullman, Widom, *Database Systems: The Complete Book* (Pearson). Bài giảng trên lớp là tài liệu chính.

---

## Mục lục

- Chương 0. Tổng quan về cơ sở dữ liệu và DBMS
- Chương 1. Mô hình dữ liệu quan hệ
- Chương 2. Thiết kế CSDL: ERD, ánh xạ sang bảng và chuẩn hóa
- Chương 3. Đại số quan hệ
- Chương 4. SQL: DDL và DML
- Chương 5. Tối ưu câu truy vấn
- Chương 6. An toàn dữ liệu, ràng buộc, giao dịch và trigger
- Chương 7. Cơ sở dữ liệu phân tán
- Chương 8. Các vấn đề nâng cao và bài tập lớn
- Phụ lục A. Quy trình thiết kế 7 bước (tóm tắt)
- Phụ lục B. Bài tập tự kiểm tra và đáp án
- Phụ lục C. Thuật ngữ

---

## Chương 0. Tổng quan về cơ sở dữ liệu và DBMS

### 0.1. Khái niệm

**Cơ sở dữ liệu (CSDL)** là tập hợp dữ liệu có liên kết logic với nhau, phản ánh thông tin của một tổ chức hoặc doanh nghiệp.

> **Lưu ý thuật ngữ:** slide ghi "tập hợp các siêu dữ liệu". Cách định nghĩa phổ biến hơn là "tập hợp dữ liệu có liên kết logic". **Siêu dữ liệu** (metadata) là thông tin mô tả dữ liệu, ví dụ tên bảng, kiểu cột, ràng buộc. Siêu dữ liệu được lưu trong **danh mục hệ thống** (catalog) của DBMS.

**Hệ quản trị CSDL (DBMS - Database Management System)** là phần mềm giúp tạo lập CSDL và kiểm soát mọi truy cập vào CSDL đó. Ví dụ: PostgreSQL, MySQL, Oracle, SQL Server.

### 0.2. Bốn vai trò cốt lõi của DBMS

| Vai trò | Ý nghĩa | Ví dụ |
|---|---|---|
| Quản lý khối lượng dữ liệu lớn | Lưu trữ và tổ chức dữ liệu quy mô lớn | Bảng giao dịch hàng trăm triệu dòng |
| Hỗ trợ truy cập khối lượng lớn | Tìm kiếm, trích xuất nhanh | Tìm giao dịch của một khách trong 1 mili-giây nhờ chỉ mục |
| Hỗ trợ truy cập đồng thời | Nhiều người dùng thao tác cùng lúc mà không sai lệch | Hai nhân viên cùng cập nhật số dư một tài khoản |
| An toàn và bảo mật | Phục hồi khi sự cố, chặn truy cập trái phép | Mất điện giữa chừng vẫn không mất giao dịch đã xác nhận |

### 0.3. Kiến trúc tổng quát

```
Ứng dụng / người dùng
        │  SQL
        ▼
┌───────────────────────────────┐
│ DBMS                          │
│  - Bộ phân tích & tối ưu truy vấn │
│  - Bộ thực thi                │
│  - Quản lý giao dịch, khóa    │
│  - Quản lý bộ nhớ đệm, lưu trữ │
│  - Ghi nhật ký (log), phục hồi │
└───────────────────────────────┘
        │
        ▼
   Tệp dữ liệu + Catalog + Nhật ký
```

### 0.4. Mô hình dữ liệu quan hệ là gì

Mô hình quan hệ, do E. F. Codd đề xuất năm 1970, biểu diễn dữ liệu dưới dạng **bảng** gồm các **dòng** và **cột**. Đây là mô hình được dùng phổ biến nhất hiện nay.

Ví dụ: bảng TAIKHOAN gồm các cột *Số tài khoản*, *Họ và tên*, *Số dư*.

| SoTK | HoTen | SoDu |
|---|---|---|
| 0011000001 | Nguyễn Văn An | 15 000 000 |
| 0011000002 | Trần Thị Bình | 2 500 000 |

---

## Chương 1. Mô hình dữ liệu quan hệ

### 1.1. Các khái niệm nền tảng

- **Miền (domain):** tập giá trị hợp lệ cho một thuộc tính. Ví dụ: miền của *SoDu* là số thập phân không âm.
- **Quan hệ (relation):** một bảng, có tên, gồm lược đồ và thể hiện.
- **Lược đồ quan hệ:** $R(A_1, A_2, \dots, A_n)$, mô tả tên bảng và tên các thuộc tính. Ví dụ: `TAIKHOAN(SoTK, MaKH, LoaiTK, SoDu, NgayMo)`.
- **Thuộc tính (attribute):** tương ứng với một cột.
- **Bộ (tuple):** tương ứng với một dòng.
- **Bậc (degree):** số thuộc tính. **Lực lượng (cardinality):** số bộ.
- **Thể hiện (instance):** tập bộ hiện có tại một thời điểm. Lược đồ ít thay đổi, thể hiện thay đổi liên tục.

### 1.2. Tính chất của quan hệ toán học

1. Không có hai bộ giống hệt nhau. Bảng SQL thực tế có thể có dòng trùng nếu không khai báo khóa, nhưng về lý thuyết thì không được phép.
2. Các bộ không có thứ tự. Thứ tự dòng khi truy vấn chỉ có nghĩa khi có `ORDER BY`.
3. Các thuộc tính không có thứ tự.
4. Mỗi giá trị thuộc tính là **nguyên tử** (không lặp, không gộp nhiều giá trị trong một ô). Đây là điều kiện của dạng chuẩn 1NF.

**Ví dụ vi phạm nguyên tử:**

| MaKH | SoDienThoai |
|---|---|
| KH01 | 0901111111, 0902222222 |

Cách sửa: tách thành bảng riêng `SDT_KHACHHANG(MaKH, SoDienThoai)`.

### 1.3. Khóa

- **Siêu khóa (superkey):** tập thuộc tính xác định duy nhất một bộ.
- **Khóa ứng viên (candidate key):** siêu khóa tối thiểu, không thể bỏ bớt thuộc tính nào mà vẫn xác định duy nhất.
- **Khóa chính (primary key):** một khóa ứng viên được chọn để định danh bộ. Không được rỗng và không được trùng.
- **Khóa ngoại (foreign key):** thuộc tính (hoặc tập thuộc tính) trong bảng này tham chiếu tới khóa chính của bảng khác, dùng để duy trì liên kết.

**Ví dụ:** trong bảng `KHACHHANG(MaKH, CCCD, HoTen, NgaySinh)`:
- Khóa ứng viên: `MaKH` và `CCCD` (cả hai đều duy nhất).
- Khóa chính có thể chọn là `MaKH`. `CCCD` vẫn là **khóa dự tuyển** và nên khai báo `UNIQUE`.
- Siêu khóa: `{MaKH}`, `{MaKH, HoTen}`, `{CCCD, NgaySinh}`... Chỉ `{MaKH}` và `{CCCD}` là khóa ứng viên vì chúng tối thiểu.

### 1.4. Giá trị rỗng (NULL)

`NULL` nghĩa là "không biết" hoặc "không áp dụng", không phải số 0 hay chuỗi rỗng. So sánh với `NULL` cho kết quả *không xác định*:

```sql
SELECT NULL = NULL;        -- kết quả: NULL (không phải TRUE)
SELECT * FROM KHACHHANG WHERE DiaChi = NULL;   -- sai: không trả về dòng nào
SELECT * FROM KHACHHANG WHERE DiaChi IS NULL;  -- đúng
```

Khóa chính không được chứa `NULL`.

---

## Chương 2. Thiết kế CSDL: ERD, ánh xạ sang bảng và chuẩn hóa

Chương này đi qua các bước 1 đến 4 của quy trình thiết kế (Phụ lục A).

### 2.1. Mô hình thực thể - liên kết (ERD)

Các thành phần:

- **Thực thể (entity):** đối tượng cần quản lý, ví dụ KHÁCH HÀNG, TÀI KHOẢN.
- **Thuộc tính (attribute):** đặc điểm của thực thể. Có các loại:
  - đơn/đa trị (một hay nhiều giá trị),
  - đơn/kết hợp (một giá trị hay gồm nhiều phần, ví dụ *Địa chỉ* gồm Số nhà, Phố, Thành phố),
  - suy diễn (có thể tính từ thuộc tính khác, ví dụ *Tuổi* suy từ *Ngày sinh*).
- **Liên kết (relationship):** mối quan hệ giữa các thực thể.
- **Bản số (cardinality):** số lượng tối thiểu và tối đa mà một thực thể tham gia vào liên kết.

**Các loại liên kết:**

| Loại | Ví dụ | Ý nghĩa |
|---|---|---|
| 1 - 1 | KHÁCH HÀNG — CCCD_GỐC | Mỗi khách có một bản ghi căn cước chính |
| 1 - N | KHÁCH HÀNG — TÀI KHOẢN | Một khách có nhiều tài khoản, mỗi tài khoản thuộc một khách |
| N - N | TÀI KHOẢN — DỊCH VỤ | Một tài khoản đăng ký nhiều dịch vụ, một dịch vụ có nhiều tài khoản |

**Thực thể yếu (weak entity):** không tự tồn tại độc lập, cần khóa của thực thể chủ. Ví dụ: `NGUOITHAN(MaKH, STT, HoTen)` phụ thuộc vào `KHACHHANG`. Khóa đầy đủ là `(MaKH, STT)`.

**Sơ đồ ERD ví dụ (ngân hàng):**

```
KHACHHANG ──1────────N── TAIKHOAN ──1────────N── GIAODICH
 (MaKH, HoTen,            (SoTK, LoaiTK,          (MaGD, SoTien,
  CCCD, NgaySinh)          SoDu)                   NgayGD)
```

Ghi chú: giao dịch là liên kết giữa hai tài khoản. Trong thiết kế đầy đủ, nó có thể là một thực thể riêng với hai liên kết 1 - N tới TAIKHOAN (tài khoản nguồn và tài khoản đích). Sơ đồ trên đã đơn giản hóa.

### 2.2. Ánh xạ từ ERD sang lược đồ quan hệ

| Thành phần ERD | Cách ánh xạ |
|---|---|
| Thực thể mạnh | Một bảng. Thuộc tính khóa trở thành khóa chính |
| Thuộc tính đơn | Một cột |
| Thuộc tính kết hợp | Tách thành nhiều cột (ví dụ `DiaChi_SoNha`, `DiaChi_Pho`) |
| Thuộc tính đa trị | Một bảng riêng gồm khóa của thực thể + giá trị đa trị |
| Liên kết 1 - N | Thêm khóa ngoại vào phía N, trỏ tới phía 1 |
| Liên kết 1 - 1 | Thêm khóa ngoại vào một trong hai bên (thường là bên có tham gia bắt buộc) |
| Liên kết N - N | Tạo bảng trung gian, khóa chính là hợp của hai khóa ngoại |
| Thực thể yếu | Bảng có khóa chính = khóa chủ + khóa bộ phận |

**Ví dụ ánh xạ N - N:** sinh viên đăng ký môn học.

```
SINHVIEN(MaSV, HoTen)
MONHOC(MaMH, TenMH)
DANGKY(MaSV, MaMH, NgayDK, DiemSo)   -- bảng trung gian
   MaSV: FK → SINHVIEN
   MaMH: FK → MONHOC
   PK = (MaSV, MaMH)
```

### 2.3. Phụ thuộc hàm (functional dependency)

Phụ thuộc hàm $X \rightarrow Y$ nghĩa là: hai bộ có cùng giá trị ở $X$ thì cũng có cùng giá trị ở $Y$.

Ví dụ: `MaKH → HoTen, DiaChi` nghĩa là với cùng mã khách hàng thì họ tên và địa chỉ phải giống nhau.

Hai quy tắc cần nhớ:
- **Phụ thuộc đầy đủ:** $Y$ phụ thuộc vào toàn bộ khóa, không phụ thuộc chỉ một phần.
- **Phụ thuộc bắc cầu:** $X \rightarrow Y$ và $Y \rightarrow Z$ suy ra $X \rightarrow Z$ (trong đó $Y$ không phải khóa).

### 2.4. Chuẩn hóa: mục đích và các dị thường

**Mục đích:** loại bỏ dư thừa và tránh ba dị thường:

1. **Dị thường thêm (insert):** không thêm được dữ liệu này nếu thiếu dữ liệu khác.
2. **Dị thường xóa (delete):** xóa một dòng làm mất thông tin không liên quan.
3. **Dị thường sửa (update):** sửa một giá trị phải sửa nhiều dòng; sửa sót thì dữ liệu mâu thuẫn.

**Ví dụ gốc gây dị thường:** một bảng ghi chi tiết đơn hàng.

```
DONHANG_CHITIET(MaDH, NgayDH, MaKH, HoTenKH, DiaChiKH, MaSP, TenSP, DonGia, SoLuong)
Khóa: (MaDH, MaSP)
```

Phụ thuộc hàm:
- `MaDH → NgayDH, MaKH`
- `MaKH → HoTenKH, DiaChiKH`
- `MaSP → TenSP, DonGia`
- `(MaDH, MaSP) → SoLuong`

Dị thường: nếu khách KH01 đổi địa chỉ, phải sửa ở mọi đơn hàng của KH01 (dị thường sửa). Nếu sản phẩm chưa được đặt lần nào, không thể lưu thông tin sản phẩm đó (dị thường thêm).

### 2.5. Các dạng chuẩn

**1NF:** mọi thuộc tính là nguyên tử. Bảng trên đã đạt 1NF.

**2NF:** 1NF và mọi thuộc tính không khóa phụ thuộc đầy đủ vào khóa chính.
- Vi phạm: `TenSP, DonGia` chỉ phụ thuộc `MaSP` (một phần khóa `(MaDH, MaSP)`). `NgayDH, MaKH` chỉ phụ thuộc `MaDH`.

Tách thành:
```
DONHANG(MaDH, NgayDH, MaKH, HoTenKH, DiaChiKH)      -- PK: MaDH
SANPHAM(MaSP, TenSP, DonGia)                        -- PK: MaSP
CHITIET(MaDH, MaSP, SoLuong)                        -- PK: (MaDH, MaSP)
```

**3NF:** 2NF và không có phụ thuộc bắc cầu từ khóa qua thuộc tính không khóa.
- Vi phạm trong `DONHANG`: `MaDH → MaKH → HoTenKH, DiaChiKH`.

Tách thành:
```
DONHANG(MaDH, NgayDH, MaKH)                         -- FK: MaKH → KHACHHANG
KHACHHANG(MaKH, HoTenKH, DiaChiKH)                  -- PK: MaKH
SANPHAM(MaSP, TenSP, DonGia)
CHITIET(MaDH, MaSP, SoLuong)
```

Bây giờ đổi địa chỉ khách chỉ cần sửa một dòng trong `KHACHHANG`.

**BCNF (Boyce - Codd):** với mọi phụ thuộc hàm không tầm thường $X \rightarrow Y$, $X$ phải là siêu khóa. BCNF chặt hơn 3NF một chút. Trong ví dụ trên, mọi vế trái đều là khóa, nên kết quả đã đạt BCNF.

> **Đánh đổi:** chuẩn hóa càng sâu thì càng nhiều bảng, và truy vấn càng phải JOIN nhiều hơn. Trong thực tế, người ta đôi khi cố ý phi chuẩn hóa để tăng tốc đọc. Xem Chương 5 về chi phí JOIN.

### 2.6. Phép tách không mất thông tin (lossless join)

Khi tách bảng $R$ thành $R_1$ và $R_2$, phải đảm bảo nối lại (JOIN) được đúng dữ liệu ban đầu. Điều kiện đủ: giao của $R_1$ và $R_2$ là khóa của một trong hai bảng.

Ví dụ: $R_1 = DONHANG(MaDH, MaKH)$, $R_2 = KHACHHANG(MaKH, HoTenKH)$. Giao là `MaKH`, là khóa của `KHACHHANG`, nên phép tách là không mất thông tin.

---

## Chương 3. Đại số quan hệ

Đại số quan hệ là ngôn ngữ truy vấn hình thức, mỗi phép toán nhận một hoặc hai quan hệ và cho ra một quan hệ. SQL được xây dựng dựa trên nền tảng này. Dữ liệu mẫu:

```
KHACHHANG(MaKH, HoTen, ThanhPho)
  KH01  Nguyễn Văn An   Hà Nội
  KH02  Trần Thị Bình   TP.HCM
  KH03  Lê Văn Cường    Hà Nội

TAIKHOAN(SoTK, MaKH, LoaiTK, SoDu)
  0011  KH01  THANHTOAN   15 000 000
  0012  KH01  TIETKIEM    50 000 000
  0013  KH02  THANHTOAN    2 500 000
```

### 3.1. Các phép toán cơ bản

| Ký hiệu | Tên | Ý nghĩa | Tương đương SQL |
|---|---|---|---|
| $\sigma_{\text{điều kiện}}(R)$ | Chọn (selection) | Lọc dòng | `WHERE` |
| $\pi_{A,B}(R)$ | Chiếu (projection) | Giữ một số cột, loại trùng | `SELECT DISTINCT A, B` |
| $\rho$ | Đổi tên | Đặt tên lại quan hệ/thuộc tính | `AS` |
| $R \times S$ | Tích Descartes | Ghép mọi cặp dòng | `FROM R, S` |
| $R \cup S$ | Hợp | Các dòng ở R hoặc S (cùng lược đồ) | `UNION` |
| $R \cap S$ | Giao | Dòng có ở cả hai | `INTERSECT` |
| $R - S$ | Hiệu | Dòng ở R nhưng không ở S | `EXCEPT` |
| $R \bowtie S$ | Nối tự nhiên | Nối theo thuộc tính trùng tên | `NATURAL JOIN` / `JOIN ... ON` |
| $R \div S$ | Chia | Các giá trị liên kết với *mọi* giá trị của S | `GROUP BY ... HAVING COUNT(...)` |

### 3.2. Ví dụ

**(a) Chọn:** tài khoản có số dư trên 10 triệu.
$$\sigma_{SoDu > 10000000}(TAIKHOAN)$$
```sql
SELECT * FROM TAIKHOAN WHERE SoDu > 10000000;
```
Kết quả: tài khoản 0011 và 0012.

**(b) Chiếu:** danh sách thành phố của khách hàng.
$$\pi_{ThanhPho}(KHACHHANG)$$
```sql
SELECT DISTINCT ThanhPho FROM KHACHHANG;
```
Kết quả: Hà Nội, TP.HCM. Phép chiếu loại trùng; `SELECT` trong SQL không tự loại trùng nếu không có `DISTINCT`.

**(c) Nối tự nhiên:** tên khách kèm số tài khoản.
$$\pi_{HoTen, SoTK}(KHACHHANG \bowtie TAIKHOAN)$$
```sql
SELECT k.HoTen, t.SoTK
FROM KHACHHANG k
JOIN TAIKHOAN t ON t.MaKH = k.MaKH;
```

**(d) Chọn kết hợp nối:** khách ở Hà Nội có tài khoản thanh toán.
$$\pi_{HoTen}\Big(\sigma_{ThanhPho='Hà Nội'}(KHACHHANG) \bowtie \sigma_{LoaiTK='THANHTOAN'}(TAIKHOAN)\Big)$$
```sql
SELECT DISTINCT k.HoTen
FROM KHACHHANG k
JOIN TAIKHOAN t ON t.MaKH = k.MaKH
WHERE k.ThanhPho = 'Hà Nội' AND t.LoaiTK = 'THANHTOAN';
```

**(e) Hiệu:** khách hàng chưa có tài khoản tiết kiệm.
$$\pi_{MaKH}(KHACHHANG) - \pi_{MaKH}(\sigma_{LoaiTK='TIETKIEM'}(TAIKHOAN))$$
```sql
SELECT MaKH FROM KHACHHANG
EXCEPT
SELECT MaKH FROM TAIKHOAN WHERE LoaiTK = 'TIETKIEM';
```
Kết quả: KH02 và KH03.

**(f) Chia:** khách có *cả hai* loại tài khoản (THANHTOAN và TIETKIEM).

Gọi $S = \{THANHTOAN, TIETKIEM\}$, và $R = \pi_{MaKH, LoaiTK}(TAIKHOAN)$. Khi đó $R \div S$ trả về các MaKH mà với mọi loại trong S đều có trong R.

```sql
SELECT MaKH
FROM TAIKHOAN
WHERE LoaiTK IN ('THANHTOAN', 'TIETKIEM')
GROUP BY MaKH
HAVING COUNT(DISTINCT LoaiTK) = 2;
```
Kết quả: KH01.

### 3.3. Các phép suy dẫn

Phép giao, phép nối theo điều kiện, phép nối ngoài (outer join) có thể suy ra từ các phép cơ bản:

$$R \cap S = R - (R - S)$$
$$R \bowtie_{\theta} S = \sigma_{\theta}(R \times S)$$

Phép nối ngoài trái không thuộc đại số gốc của Codd, nhưng SQL hỗ trợ qua `LEFT JOIN`.

**Ví dụ `LEFT JOIN`:** liệt kê mọi khách kể cả khách chưa có tài khoản.
```sql
SELECT k.MaKH, k.HoTen, t.SoTK
FROM KHACHHANG k
LEFT JOIN TAIKHOAN t ON t.MaKH = k.MaKH;
```

### 3.4. Lỗi thường gặp

- Nhầm $\sigma$ với $\pi$: $\sigma$ lọc **dòng**, $\pi$ chọn **cột**.
- Quên `DISTINCT` khi cần tập hợp theo đúng định nghĩa đại số.
- Dùng `JOIN` khi thực ra cần `EXISTS` hoặc `NOT EXISTS`. Ví dụ trường hợp (e) có thể viết bằng `NOT EXISTS` và cho kết quả giống hệt.

---

## Chương 4. SQL: DDL và DML

SQL (Structured Query Language) gồm:
- **DDL (Data Definition Language):** định nghĩa cấu trúc. `CREATE`, `ALTER`, `DROP`.
- **DML (Data Manipulation Language):** thao tác dữ liệu. `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- **DCL (Data Control Language):** phân quyền. `GRANT`, `REVOKE` (xem Chương 6).
- **TCL (Transaction Control Language):** `BEGIN`, `COMMIT`, `ROLLBACK` (xem Chương 6).

### 4.1. DDL: tạo bảng và ràng buộc

```sql
CREATE TABLE KHACHHANG (
    MaKH      CHAR(8)      PRIMARY KEY,
    HoTen     VARCHAR(100) NOT NULL,
    CCCD      CHAR(12)     NOT NULL UNIQUE,
    NgaySinh  DATE,
    ThanhPho  VARCHAR(50)
);

CREATE TABLE TAIKHOAN (
    SoTK      CHAR(10)      PRIMARY KEY,
    MaKH      CHAR(8)       NOT NULL REFERENCES KHACHHANG(MaKH),
    LoaiTK    VARCHAR(20)   NOT NULL CHECK (LoaiTK IN ('THANHTOAN', 'TIETKIEM')),
    SoDu      DECIMAL(15,0) NOT NULL DEFAULT 0 CHECK (SoDu >= 0),
    NgayMo    DATE          NOT NULL DEFAULT CURRENT_DATE
);

CREATE TABLE GIAODICH (
    MaGD        BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    SoTK_Nguon  CHAR(10)      NOT NULL REFERENCES TAIKHOAN(SoTK),
    SoTK_Dich   CHAR(10)               REFERENCES TAIKHOAN(SoTK),
    SoTien      DECIMAL(15,0) NOT NULL CHECK (SoTien > 0),
    NgayGD      TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Các loại ràng buộc:**

| Ràng buộc | Mục đích | Ví dụ trên |
|---|---|---|
| `PRIMARY KEY` | Định danh duy nhất, không NULL | `MaKH` |
| `UNIQUE` | Duy nhất, cho phép NULL (tùy hệ) | `CCCD` |
| `NOT NULL` | Bắt buộc có giá trị | `HoTen` |
| `FOREIGN KEY ... REFERENCES` | Toàn vẹn tham chiếu | `TAIKHOAN.MaKH` |
| `CHECK` | Điều kiện miền | `SoDu >= 0` |
| `DEFAULT` | Giá trị mặc định | `NgayMo` |

**Hành vi khi xóa cha:** mặc định `FOREIGN KEY` từ chối xóa khách còn tài khoản. Có thể chỉ định:
```sql
MaKH CHAR(8) REFERENCES KHACHHANG(MaKH) ON DELETE CASCADE   -- xóa khách thì xóa luôn tài khoản
```
Nên hạn chế `CASCADE` với dữ liệu tài chính, vì xóa nhầm sẽ mất lịch sử.

**Sửa và xóa cấu trúc:**
```sql
ALTER TABLE KHACHHANG ADD COLUMN Email VARCHAR(100);
ALTER TABLE TAIKHOAN ALTER COLUMN SoDu SET DEFAULT 0;
DROP TABLE GIAODICH;                    -- xóa hẳn bảng và dữ liệu
TRUNCATE TABLE GIAODICH;                -- xóa toàn bộ dòng, giữ cấu trúc
```

### 4.2. DML: thêm, sửa, xóa

```sql
INSERT INTO KHACHHANG (MaKH, HoTen, CCCD, ThanhPho)
VALUES ('KH01', 'Nguyễn Văn An', '001099000111', 'Hà Nội');

INSERT INTO TAIKHOAN (SoTK, MaKH, LoaiTK, SoDu)
VALUES ('0011000001', 'KH01', 'THANHTOAN', 15000000);

UPDATE TAIKHOAN
SET SoDu = SoDu + 1000000
WHERE SoTK = '0011000001';

DELETE FROM TAIKHOAN
WHERE SoTK = '0011000001' AND SoDu = 0;
```

> **Cẩn thận:** `UPDATE` và `DELETE` không có `WHERE` sẽ áp dụng cho **toàn bộ bảng**. Trong môi trường thật, nên chạy `SELECT` với cùng điều kiện trước khi sửa.

### 4.3. SELECT: cấu trúc và thứ tự xử lý

```sql
SELECT   k.ThanhPho, COUNT(*) AS SoTaiKhoan, SUM(t.SoDu) AS TongSoDu
FROM     KHACHHANG k
JOIN     TAIKHOAN t ON t.MaKH = k.MaKH
WHERE    t.LoaiTK = 'THANHTOAN'
GROUP BY k.ThanhPho
HAVING   SUM(t.SoDu) > 5000000
ORDER BY TongSoDu DESC
LIMIT    10;
```

**Thứ tự logic thực thi** (khác với thứ tự viết):
1. `FROM` và `JOIN`: ghép bảng.
2. `WHERE`: lọc dòng.
3. `GROUP BY`: gom nhóm.
4. `HAVING`: lọc nhóm.
5. `SELECT`: chọn và tính biểu thức.
6. `DISTINCT`.
7. `ORDER BY`.
8. `LIMIT` / `OFFSET`.

Hệ quả thực tế: không thể dùng bí danh cột trong `WHERE` (vì `WHERE` chạy trước `SELECT`), nhưng dùng được trong `ORDER BY`.

### 4.4. Hàm gộp (aggregate)

`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`. Lưu ý:
- `COUNT(*)` đếm mọi dòng. `COUNT(cot)` bỏ qua NULL.
- Cột không nằm trong `GROUP BY` thì không được xuất hiện trong `SELECT` (trừ khi là hàm gộp).

```sql
SELECT LoaiTK, COUNT(*), AVG(SoDu)
FROM TAIKHOAN
GROUP BY LoaiTK;
```

### 4.5. Truy vấn lồng (subquery)

**Truy vấn con trong WHERE:** khách có số dư tài khoản cao hơn trung bình.
```sql
SELECT SoTK, SoDu
FROM TAIKHOAN
WHERE SoDu > (SELECT AVG(SoDu) FROM TAIKHOAN);
```

**Dùng EXISTS:** khách có ít nhất một tài khoản tiết kiệm.
```sql
SELECT k.HoTen
FROM KHACHHANG k
WHERE EXISTS (
    SELECT 1 FROM TAIKHOAN t
    WHERE t.MaKH = k.MaKH AND t.LoaiTK = 'TIETKIEM'
);
```

**Cạm bẫy `NOT IN` với NULL:** nếu danh sách con có NULL, `NOT IN` không trả về dòng nào. Nên dùng `NOT EXISTS`.

### 4.6. View và CTE

**View** là truy vấn được đặt tên, không lưu dữ liệu (view thường):
```sql
CREATE VIEW v_TongSoDuTheoKhach AS
SELECT k.MaKH, k.HoTen, SUM(t.SoDu) AS TongSoDu
FROM KHACHHANG k
JOIN TAIKHOAN t ON t.MaKH = k.MaKH
GROUP BY k.MaKH, k.HoTen;

SELECT * FROM v_TongSoDuTheoKhach WHERE TongSoDu > 10000000;
```

**CTE (WITH)** tách truy vấn phức tạp thành các bước rõ ràng:
```sql
WITH tong AS (
    SELECT MaKH, SUM(SoDu) AS TongSoDu
    FROM TAIKHOAN
    GROUP BY MaKH
)
SELECT k.HoTen, tong.TongSoDu
FROM tong
JOIN KHACHHANG k ON k.MaKH = tong.MaKH
WHERE tong.TongSoDu > 10000000;
```

---

## Chương 5. Tối ưu câu truy vấn

### 5.1. Vì sao cần tối ưu

Cùng một kết quả có thể được tính bằng nhiều cách với chi phí rất khác nhau. Ví dụ: quét 10 triệu dòng rồi mới lọc, hoặc lọc trước rồi mới nối. Bộ tối ưu (optimizer) chọn cách rẻ nhất trong các cách tương đương.

### 5.2. Các giai đoạn xử lý truy vấn

```
SQL text
  │ 1. Phân tích cú pháp (parser)     → cây phân tích
  │ 2. Kiểm tra ngữ nghĩa (binder)    → tên bảng, cột có tồn tại không
  │ 3. Biến đổi đại số (rewriter)     → cây đại số quan hệ tương đương
  │ 4. Tối ưu dựa trên chi phí        → kế hoạch vật lý rẻ nhất
  │ 5. Thực thi (executor)            → kết quả
```

### 5.3. Biến đổi đại số tương đương

Các luật thường dùng:
1. **Đẩy phép chọn xuống sớm:** $\sigma_{c}(R \bowtie S) = \sigma_{c}(R) \bowtie S$ khi $c$ chỉ dùng thuộc tính của $R$.
2. **Đẩy phép chiếu xuống sớm:** bỏ các cột không cần để giảm dữ liệu trung gian.
3. **Phép nối và phép chọn có tính giao hoán và kết hợp** theo các điều kiện phù hợp.

**Ví dụ:**
```sql
SELECT k.HoTen, t.SoTK
FROM KHACHHANG k
JOIN TAIKHOAN t ON t.MaKH = k.MaKH
WHERE k.ThanhPho = 'Hà Nội' AND t.SoDu > 10000000;
```

Cách 1 (chưa tối ưu): nối 1 triệu khách với 5 triệu tài khoản (tạo 5 triệu tỷ cặp, bộ lọc chạy sau) rồi mới lọc.

Cách 2 (đẩy phép chọn xuống):
$$\pi_{HoTen, SoTK}\Big(\sigma_{ThanhPho='Hà Nội'}(KHACHHANG) \bowtie \sigma_{SoDu>10000000}(TAIKHOAN)\Big)$$

Chỉ còn khoảng 100 nghìn khách Hà Nội nối với khoảng 300 nghìn tài khoản thỏa điều kiện. Số dòng trung gian giảm mạnh.

Lưu ý: optimizer hiện đại tự làm bước này. Sinh viên cần hiểu nguyên lý để đọc được kế hoạch thực thi và viết truy vấn có cấu trúc tốt.

### 5.4. Chi phí và thống kê

Optimizer ước lượng **số dòng** và **chi phí I/O, CPU** cho từng phương án dựa trên **thống kê** (số dòng, số giá trị phân biệt, phân bố dữ liệu, kích thước bảng). Khi thống kê cũ hoặc sai, kế hoạch có thể rất tệ.

Trong PostgreSQL, lệnh `ANALYZE` cập nhật thống kê. Trong MySQL là `ANALYZE TABLE`.

### 5.5. Thuật toán nối (join)

| Thuật toán | Ý tưởng | Khi nào tốt |
|---|---|---|
| Nested loop | Với mỗi dòng của R, quét S | R rất nhỏ, S có chỉ mục |
| Hash join | Băm bảng nhỏ vào bộ nhớ, quét bảng lớn để tra | Nối bằng `=`, bộ nhớ đủ |
| Merge join | Sắp xếp hai bên theo khóa rồi gộp | Dữ liệu đã có thứ tự hoặc có chỉ mục |

Mô hình chi phí đơn giản: nested loop tốn khoảng $|R| \cdot |S|$ phép so sánh; hash join khoảng $|R| + |S|$ (khi đủ bộ nhớ).

### 5.6. Chỉ mục (index)

Chỉ mục phổ biến là **B+ tree**: cây cân bằng, mỗi nút có nhiều khóa, tìm kiếm mất khoảng $O(\log_b N)$ với $b$ là bậc của cây.

```sql
CREATE INDEX idx_taikhoan_makh ON TAIKHOAN(MaKH);
CREATE INDEX idx_giaodich_ngay ON GIAODICH(NgayGD);
```

**Lưu ý:**
- PostgreSQL **không tự tạo chỉ mục** cho khóa ngoại. Nên tự tạo để JOIN và `ON DELETE` nhanh hơn.
- Chỉ mục làm `INSERT`, `UPDATE`, `DELETE` chậm hơn và tốn bộ nhớ. Không nên đánh chỉ mục cho mọi cột.
- Chỉ mục không được dùng khi điều kiện bọc hàm, ví dụ `WHERE UPPER(HoTen) = ...`, trừ khi có chỉ mục biểu thức.

### 5.7. Đọc kế hoạch thực thi

```sql
EXPLAIN ANALYZE
SELECT k.HoTen, t.SoTK
FROM KHACHHANG k
JOIN TAIKHOAN t ON t.MaKH = k.MaKH
WHERE k.ThanhPho = 'Hà Nội' AND t.SoDu > 10000000;
```

Kết quả minh họa (định dạng rút gọn, số liệu giả định):
```
Hash Join  (cost=... rows=12000) (actual rows=11873)
  Hash Cond: (t.makh = k.makh)
  -> Seq Scan on taikhoan t  (filter: sodu > 10000000)
  -> Hash
       -> Seq Scan on khachhang k  (filter: thanhpho = 'Hà Nội')
```

Cách đọc:
- `Seq Scan`: quét toàn bảng. `Index Scan`: dùng chỉ mục.
- So sánh `rows` ước lượng và `actual rows`. Lệch lớn nghĩa là thống kê sai.
- `cost` là đơn vị nội bộ của optimizer, không phải thời gian thực.
- `ANALYZE` trong `EXPLAIN ANALYZE` thực sự chạy truy vấn, nên cẩn thận với lệnh ghi.

### 5.8. Ví dụ tối ưu hóa thực tế

Truy vấn chậm:
```sql
SELECT * FROM GIAODICH WHERE DATE(NgayGD) = '2026-10-01';
```
Hàm `DATE()` bọc cột khiến chỉ mục trên `NgayGD` không dùng được.

Viết lại:
```sql
SELECT * FROM GIAODICH
WHERE NgayGD >= '2026-10-01' AND NgayGD < '2026-10-02';
```
Điều kiện là khoảng trên cột gốc, chỉ mục được dùng.

Đây là mẫu chung: **đặt cột về một vế, tránh bọc hàm lên cột trong `WHERE`**.

---

## Chương 6. An toàn dữ liệu, ràng buộc, giao dịch và trigger

### 6.1. Bảo mật và phân quyền

**Phân quyền tùy ý (DAC):** chủ đối tượng cấp quyền cho người khác.
```sql
CREATE ROLE nhanvien_giaodich;
CREATE ROLE kiemtoan;

GRANT SELECT, INSERT ON GIAODICH TO nhanvien_giaodich;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO kiemtoan;
REVOKE INSERT ON GIAODICH FROM nhanvien_giaodich;
```

**Dùng view để ẩn cột nhạy cảm:** nhân viên chăm sóc khách không được thấy CCCD.
```sql
CREATE VIEW v_khachhang_cskh AS
SELECT MaKH, HoTen, ThanhPho
FROM KHACHHANG;

GRANT SELECT ON v_khachhang_cskh TO nhanvien_cskh;
-- không cấp quyền trên bảng KHACHHANG gốc
```

**Phân quyền theo dòng (row-level security)** trong PostgreSQL cho phép giới hạn dòng theo người dùng. Ví dụ mỗi chi nhánh chỉ thấy giao dịch của mình.

**Chống SQL injection:** không ghép chuỗi nhập từ người dùng vào câu SQL. Dùng truy vấn tham số hóa.

```python
# SAI: nguy cơ tiêm SQL
cur.execute("SELECT * FROM KHACHHANG WHERE HoTen = '" + ten + "'")

# ĐÚNG: truy vấn tham số hóa (cú pháp tùy thư viện)
cur.execute("SELECT * FROM KHACHHANG WHERE HoTen = %s", (ten,))
```

Ví dụ tấn công: người dùng nhập `' OR '1'='1` khiến điều kiện luôn đúng và lộ toàn bộ bảng.

### 6.2. Ràng buộc toàn vẹn

Có ba nhóm:
- **Toàn vẹn thực thể:** khóa chính duy nhất và không NULL.
- **Toàn vẹn tham chiếu:** khóa ngoại phải trỏ tới bản ghi tồn tại (hoặc NULL).
- **Toàn vẹn miền / nghiệp vụ:** `CHECK`, `NOT NULL`, `UNIQUE`, và các quy tắc phức tạp hơn (xem trigger).

Nên khai báo ràng buộc trong CSDL, không chỉ trong ứng dụng, vì ứng dụng có thể có nhiều phiên bản hoặc nhiều chương trình cùng ghi dữ liệu.

### 6.3. Giao dịch và ACID

Giao dịch là một chuỗi thao tác được coi là một đơn vị không thể chia nhỏ.

| Tính chất | Ý nghĩa | Ví dụ chuyển tiền |
|---|---|---|
| **A**tomicity - Nguyên tử | Hoặc làm hết, hoặc không làm gì | Trừ tiền A và cộng tiền B cùng thành công hoặc cùng bị hủy |
| **C**onsistency - Nhất quán | Đi từ trạng thái hợp lệ sang trạng thái hợp lệ | Tổng tiền trước và sau chuyển không đổi |
| **I**solation - Cô lập | Giao dịch đồng thời không thấy trạng thái dở dang của nhau | Người khác không thấy số dư trung gian |
| **D**urability - Bền vững | Sau khi COMMIT, dữ liệu không mất khi sự cố | Mất điện ngay sau COMMIT vẫn còn giao dịch |

**Ví dụ chuyển tiền:**
```sql
BEGIN;

UPDATE TAIKHOAN SET SoDu = SoDu - 1000000 WHERE SoTK = '0011000001';
UPDATE TAIKHOAN SET SoDu = SoDu + 1000000 WHERE SoTK = '0013000001';

INSERT INTO GIAODICH (SoTK_Nguon, SoTK_Dich, SoTien)
VALUES ('0011000001', '0013000001', 1000000);

COMMIT;   -- hoặc ROLLBACK nếu có lỗi
```

Nếu `SoDu` của tài khoản nguồn không đủ, ràng buộc `CHECK (SoDu >= 0)` sẽ báo lỗi, và giao dịch bị hủy toàn bộ.

**Lỗi cập nhật mất (lost update):** hai giao dịch cùng đọc số dư 100, mỗi bên cộng 10 rồi ghi lại. Kết quả là 110 thay vì 120. Các cơ chế khóa hoặc mức cô lập phù hợp ngăn lỗi này. Với cập nhật dạng `SoDu = SoDu + 10`, hệ quản trị thường tự xử lý an toàn trong một lệnh đơn. Vấn đề xuất hiện khi ứng dụng đọc rồi tính rồi ghi.

Các mức cô lập chi tiết và kỹ thuật khóa thuộc phần nâng cao của môn học, nếu bài giảng có đề cập.

### 6.4. Trigger

**Trigger** là thủ tục tự động chạy khi có sự kiện `INSERT`, `UPDATE` hoặc `DELETE` trên một bảng. Dùng để duy trì ràng buộc phức tạp hoặc ghi nhật ký.

Thành phần:
- **Sự kiện:** `BEFORE` / `AFTER` + `INSERT` / `UPDATE` / `DELETE`.
- **Phạm vi:** `FOR EACH ROW` (mỗi dòng) hoặc `FOR EACH STATEMENT` (mỗi câu lệnh).
- **Hành động:** một hàm trả về `TRIGGER`.
- Trong hàm, `NEW` là bản ghi mới, `OLD` là bản ghi cũ.

**Ví dụ 1: cập nhật số dư tự động khi có giao dịch (PostgreSQL).**
```sql
CREATE OR REPLACE FUNCTION fn_capnhat_sodu() RETURNS TRIGGER AS $$
BEGIN
    -- Trừ tiền tài khoản nguồn
    UPDATE TAIKHOAN
    SET SoDu = SoDu - NEW.SoTien
    WHERE SoTK = NEW.SoTK_Nguon;

    -- Cộng tiền tài khoản đích (nếu có)
    UPDATE TAIKHOAN
    SET SoDu = SoDu + NEW.SoTien
    WHERE SoTK = NEW.SoTK_Dich;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_giaodich_sodu
AFTER INSERT ON GIAODICH
FOR EACH ROW
EXECUTE FUNCTION fn_capnhat_sodu();
```

Lưu ý: mô hình này đơn giản hóa. Trong hệ thống thật, người ta thường để ứng dụng hoặc một thủ tục lưu sẵn (stored procedure) trong một giao dịch điều khiển luồng này, vì trigger ẩn logic và khó gỡ lỗi.

**Ví dụ 2: ghi nhật ký thay đổi số dư (audit log).**
```sql
CREATE TABLE LICHSU_SODU (
    ID        BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    SoTK      CHAR(10),
    SoDuCu    DECIMAL(15,0),
    SoDuMoi   DECIMAL(15,0),
    ThoiDiem  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION fn_log_sodu() RETURNS TRIGGER AS $$
BEGIN
    IF NEW.SoDu IS DISTINCT FROM OLD.SoDu THEN
        INSERT INTO LICHSU_SODU (SoTK, SoDuCu, SoDuMoi)
        VALUES (OLD.SoTK, OLD.SoDu, NEW.SoDu);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_log_sodu
AFTER UPDATE ON TAIKHOAN
FOR EACH ROW
EXECUTE FUNCTION fn_log_sodu();
```

**Ví dụ 3: chặn xóa tài khoản còn số dư.**
```sql
CREATE OR REPLACE FUNCTION fn_chan_xoa() RETURNS TRIGGER AS $$
BEGIN
    IF OLD.SoDu <> 0 THEN
        RAISE EXCEPTION 'Không thể xóa tài khoản % còn số dư', OLD.SoTK;
    END IF;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_chan_xoa
BEFORE DELETE ON TAIKHOAN
FOR EACH ROW
EXECUTE FUNCTION fn_chan_xoa();
```

**Khi nào nên và không nên dùng trigger:**

| Nên | Không nên |
|---|---|
| Ghi nhật ký, kiểm toán | Logic nghiệp vụ phức tạp, khó đọc |
| Duy trì ràng buộc không khai báo được bằng `CHECK` | Gọi dịch vụ bên ngoài (mạng, email) |
| Đồng bộ dữ liệu dẫn xuất (bảng tổng hợp) | Trigger gọi trigger dài chuỗi, khó lần vết |

### 6.5. Kiểm soát ràng buộc: DEFERRABLE

Ràng buộc có thể được kiểm tra cuối giao dịch thay vì ngay sau mỗi lệnh (`DEFERRABLE INITIALLY DEFERRED`). Hữu ích khi hai bảng tham chiếu vòng nhau.

---

## Chương 7. Cơ sở dữ liệu phân tán

### 7.1. Khái niệm

CSDL phân tán là một CSDL logic duy nhất, nhưng dữ liệu nằm vật lý trên nhiều trạm (node) nối qua mạng. Ví dụ: ngân hàng có chi nhánh Hà Nội, Đà Nẵng, TP.HCM, mỗi nơi lưu dữ liệu khách hàng của mình.

Lợi ích: gần người dùng, giảm tải tập trung, tăng khả năng chịu lỗi.
Thách thức: đồng bộ, truy vấn xuyên trạm, mạng có thể đứt, giao dịch phân tán phức tạp hơn.

### 7.2. Phân mảnh (fragmentation)

**Phân mảnh ngang (horizontal):** chia **dòng** theo điều kiện. Mỗi mảnh có cùng lược đồ.

```
KHACHHANG (gốc)
  KH_HaNoi  = σ_{ThanhPho='Hà Nội'}(KHACHHANG)   → trạm Hà Nội
  KH_KhacHN = σ_{ThanhPho<>'Hà Nội'}(KHACHHANG)  → trạm còn lại
Tái tạo: KHACHHANG = KH_HaNoi ∪ KH_KhacHN
```

**Phân mảnh dọc (vertical):** chia **cột**, mỗi mảnh giữ khóa chính.

```
KHACHHANG_CB(MaKH, HoTen, ThanhPho)        → trạm xử lý thường ngày
KHACHHANG_NS(MaKH, CCCD, NgaySinh)         → trạm bảo mật, quyền truy cập hạn chế
Tái tạo: KHACHHANG = KHACHHANG_CB ⋈ KHACHHANG_NS  (theo MaKH)
```

**Phân mảnh hỗn hợp:** kết hợp cả hai.

**Yêu cầu đúng đắn của phân mảnh:**
1. **Đầy đủ:** mọi dòng/cột gốc đều nằm trong ít nhất một mảnh.
2. **Không chồng chéo** (với phân mảnh ngang): mỗi dòng nằm đúng một mảnh (có thể cho phép chồng khi chủ đích nhân bản).
3. **Tái tạo được:** có thể ghép lại đúng dữ liệu gốc.

### 7.3. Nhân bản và phân bổ

- **Nhân bản (replication):** một mảnh có nhiều bản sao. Tăng khả năng sẵn sàng, đọc nhanh hơn, nhưng phải đồng bộ bản sao.
  - *Đồng bộ:* ghi xong trên mọi bản mới báo thành công. An toàn nhưng chậm.
  - *Bất đồng bộ:* ghi trên bản chính rồi lan sang bản phụ sau. Nhanh nhưng có thể đọc phải dữ liệu cũ.
- **Phân bổ (allocation):** quyết định mảnh nào đặt ở trạm nào.

### 7.4. Tính trong suốt (transparency)

Mục tiêu: người dùng viết truy vấn như trên một CSDL duy nhất.

| Loại trong suốt | Người dùng không phải biết |
|---|---|
| Phân mảnh | Dữ liệu bị chia thành mảnh nào |
| Vị trí | Mảnh nằm ở trạm nào |
| Nhân bản | Có bao nhiêu bản sao |
| Hỏng hóc | Trạm nào vừa lỗi |

Ví dụ: `SELECT * FROM KHACHHANG WHERE HoTen LIKE 'Nguyễn%'` được hệ thống biến thành truy vấn trên tất cả các mảnh rồi gộp kết quả.

### 7.5. Xử lý truy vấn phân tán

Ví dụ: tìm tổng số dư theo thành phố, trong đó `KHACHHANG` phân mảnh theo thành phố ở hai trạm, `TAIKHOAN` ở trạm thứ ba.

Nguyên tắc tối ưu:
1. **Đẩy phép chọn và chiếu về trạm chứa dữ liệu** để giảm dữ liệu truyền qua mạng.
2. **Chọn trạm nối phù hợp** (nối tại trạm có dữ liệu ít hơn, hoặc semi-join).
3. **Gộp kết quả bộ phận** (ví dụ `SUM` cục bộ rồi cộng lại ở trạm điều phối).

Ví dụ `SUM` theo nhóm trên hai trạm:
```
Trạm A: SELECT ThanhPho, SUM(SoDu) FROM ... GROUP BY ThanhPho
Trạm B: SELECT ThanhPho, SUM(SoDu) FROM ... GROUP BY ThanhPho
Trạm điều phối: gộp và cộng lại theo ThanhPho
```
Gộp theo từng phần giảm mạnh lượng dữ liệu truyền đi.

### 7.6. Giao dịch phân tán và cam kết hai pha (2PC)

Một giao dịch ghi trên nhiều trạm cần đảm bảo tất cả cùng commit hoặc cùng rollback.

**Giao thức Two-Phase Commit:**

```
Điều phối viên (coordinator)            Các trạm tham gia (participant)
Pha 1 (PREPARE):
  gửi "chuẩn bị" ─────────────────────▶ ghi log, khóa tài nguyên
                                        trả lời YES (sẵn sàng) hoặc NO
  ◀──────────────────────────────────── 
Pha 2 (COMMIT / ABORT):
  nếu tất cả YES: gửi COMMIT ─────────▶ commit, giải phóng khóa
  nếu có NO: gửi ABORT ───────────────▶ rollback
```

Nhược điểm: nếu điều phối viên sập sau pha 1, các trạm có thể bị treo, giữ khóa chờ. Đây là lý do các hệ thống hiện đại thường dùng cách khác khi có thể (ví dụ saga, bù trừ theo nghiệp vụ).

**Ví dụ chuyển tiền liên chi nhánh:** tài khoản nguồn ở Hà Nội, đích ở TP.HCM. Dùng 2PC để trừ và cộng đồng bộ. Nếu trạm TP.HCM không phản hồi, trạm Hà Nội phải hoàn tác.

### 7.7. Định lý CAP (tổng quan)

Trong hệ thống phân tán, khi xảy ra phân vùng mạng (partition), phải chọn giữa **nhất quán** (Consistency) và **sẵn sàng** (Availability). Đây là nguyên lý được thảo luận rộng rãi trong các hệ thống phân tán. Chi tiết tùy giáo trình.

---

## Chương 8. Các vấn đề nâng cao và bài tập lớn

### 8.1. Các chủ đề nâng cao thường gặp

- Tối ưu sâu: thống kê, kế hoạch thực thi song song.
- Chỉ mục nâng cao: chỉ mục đa cột, chỉ mục từng phần, chỉ mục biểu thức.
- Dữ liệu bán cấu trúc (JSON trong PostgreSQL), so sánh với NoSQL.
- Hiệu năng: phân trang (keyset pagination thay cho `OFFSET` lớn), tối ưu `JOIN`.
- Kiến trúc: sao lưu, nhân bản, sẵn sàng cao.

### 8.2. Bài tập lớn: yêu cầu và quy trình

Bài tập lớn là đồ án ứng dụng toàn bộ kiến thức môn học vào một bài toán thực tế. Các bước làm nên theo thứ tự quy trình ở Phụ lục A.

**Cấu trúc báo cáo đề xuất:**

1. Giới thiệu bài toán và phạm vi.
2. Phân tích yêu cầu: danh sách nghiệp vụ, người dùng, các truy vấn chính.
3. Thiết kế khái niệm: ERD, mô tả thực thể và liên kết, bản số.
4. Thiết kế logic: lược đồ quan hệ, khóa, phụ thuộc hàm, chứng minh chuẩn hóa đến 3NF hoặc BCNF.
5. Thiết kế vật lý: script `CREATE TABLE`, chỉ mục, kiểu dữ liệu, lý do chọn.
6. Bảo mật: vai trò, quyền, view, trigger, nếu có.
7. Dữ liệu mẫu và truy vấn: ít nhất một truy vấn đơn giản, một truy vấn JOIN, một truy vấn GROUP BY, một truy vấn lồng. Kèm kết quả và `EXPLAIN` cho truy vấn chậm nhất.
8. Ứng dụng demo (nếu có): giao diện kết nối CSDL, thao tác CRUD, chống SQL injection.
9. Kết luận, hạn chế và hướng phát triển.

**Chuẩn bị cho demo và vấn đáp:**
- Chạy được script từ đầu trên CSDL trống, không lỗi.
- Giải thích được vì sao chọn từng khóa, từng bảng và mức chuẩn hóa.
- Trả lời được: "nếu dữ liệu tăng 100 lần thì sao?", "nếu hai người cùng sửa một dòng thì sao?", "truy vấn này chậm ở đâu?".

### 8.3. Phân bổ thời lượng và đánh giá

**Thời lượng:**

| Hình thức | Số tiết |
|---|---|
| Giảng lý thuyết | 24 |
| Tiểu luận / bài tập | 6 |
| Tự học, tự nghiên cứu | 30 |

**Đánh giá:**

| Thành phần | Tỷ trọng | Nội dung |
|---|---|---|
| Điểm chuyên cần | 10% | Điểm danh, tham gia lớp |
| Bài tập lớn | 90% | Gồm hai phần dưới |
| - Báo cáo tổng thể | 30% | Gửi qua email, nộp tuần cuối |
| - Demo và vấn đáp | 60% | Demo trực tiếp với giảng viên, theo lịch thi chính thức |

Phần demo và vấn đáp chiếm tỷ trọng lớn nhất, nên cần chuẩn bị kỹ hơn cho phần này.

---

## Phụ lục A. Quy trình thiết kế CSDL quan hệ (7 bước + ứng dụng)

| Bước | Tên | Đầu ra | Ví dụ đầu ra |
|---|---|---|---|
| 1 | Phân tích yêu cầu | Danh sách nghiệp vụ, dữ liệu, truy vấn | "Khách có nhiều tài khoản; mỗi giao dịch thuộc một tài khoản nguồn" |
| 2 | Thiết kế khái niệm | Sơ đồ ERD | Sơ đồ KHACHHANG – TAIKHOAN – GIAODICH |
| 3 | Thiết kế logic | Lược đồ quan hệ | `TAIKHOAN(SoTK, MaKH, ...)` với FK |
| 4 | Chuẩn hóa | Lược đồ đã đạt 3NF/BCNF | Tách `CHITIET`, `SANPHAM`, `KHACHHANG` như mục 2.5 |
| 5 | Thiết kế vật lý | Script DDL, chỉ mục, kiểu dữ liệu | `CREATE TABLE ...`, `CREATE INDEX ...` |
| 6 | Thiết kế bảo mật | Vai trò, quyền, ràng buộc, trigger | `GRANT`, `CHECK`, trigger nhật ký |
| 7 | Hiệu chỉnh (tuning) | Chỉ mục bổ sung, viết lại truy vấn | Thêm chỉ mục, sửa `DATE(col)` như mục 5.8 |
| + | Xây dựng ứng dụng | Giao diện kết nối CSDL | Form nhập tài khoản, báo cáo |

**Lưu ý:** trong thực tế, các bước không hoàn toàn tuần tự. Hay gặp là quay lại bước 2 hoặc 3 khi phát hiện thiếu thực thể ở bước 5.

---

## Phụ lục B. Bài tập tự kiểm tra và đáp án

Dùng dữ liệu của Chương 3.

**Bài 1.** Viết đại số quan hệ và SQL cho: tên các khách hàng ở TP.HCM.

**Bài 2.** Viết SQL tìm mã khách hàng có tổng số dư trên 20 triệu.

**Bài 3.** Lược đồ `DONHANG(MaDH, MaKH, HoTenKH, MaSP, TenSP, SoLuong)` với khóa `(MaDH, MaSP)`. Xác định các dị thường và đưa về 3NF.

**Bài 4.** Cho $R = \{(\text{A}, \text{x}), (\text{A}, \text{y}), (\text{B}, \text{x})\}$ và $S = \{\text{x}, \text{y}\}$ (trên thuộc tính thứ hai). Tính $R \div S$.

**Bài 5.** Truy vấn sau chậm. Đề xuất cách sửa:
```sql
SELECT * FROM TAIKHOAN WHERE CAST(SoDu AS TEXT) LIKE '1%';
```

### Đáp án

**Bài 1.**
$$\pi_{HoTen}(\sigma_{ThanhPho='TP.HCM'}(KHACHHANG))$$
```sql
SELECT HoTen FROM KHACHHANG WHERE ThanhPho = 'TP.HCM';
```
Kết quả: Trần Thị Bình.

**Bài 2.**
```sql
SELECT MaKH
FROM TAIKHOAN
GROUP BY MaKH
HAVING SUM(SoDu) > 20000000;
```
Kết quả: KH01 (15 + 50 = 65 triệu). KH02 không đủ.

**Bài 3.** Phụ thuộc hàm: `MaDH → MaKH, NgayDH`; `MaKH → HoTenKH`; `MaSP → TenSP`; `(MaDH, MaSP) → SoLuong`.
- Vi phạm 2NF: `TenSP` phụ thuộc một phần `MaSP`; `MaKH` phụ thuộc một phần `MaDH`.
- Vi phạm 3NF: `MaDH → MaKH → HoTenKH`.
- Kết quả:
```
DONHANG(MaDH, MaKH)
KHACHHANG(MaKH, HoTenKH)
SANPHAM(MaSP, TenSP)
CHITIET(MaDH, MaSP, SoLuong)
```

**Bài 4.** $R \div S$ là tập các giá trị A mà liên kết với *mọi* y trong S. A liên kết với x và y, nên kết quả là $\{A\}$. B chỉ có x, không đủ.

**Bài 5.** `CAST` bọc cột khiến chỉ mục không dùng được và làm khớp chuỗi thay vì số. Nên dùng khoảng số:
```sql
SELECT * FROM TAIKHOAN WHERE SoDu >= 1000000 AND SoDu < 2000000;
```
Viết khoảng phụ thuộc vào ý nghĩa "bắt đầu bằng 1" mà đề bài muốn. Cách trên là một cách diễn giải, cần làm rõ với yêu cầu thực tế.

---

## Phụ lục C. Thuật ngữ

| Thuật ngữ | Tiếng Anh | Giải thích ngắn |
|---|---|---|
| Quan hệ | Relation | Bảng |
| Thuộc tính | Attribute | Cột |
| Bộ | Tuple | Dòng |
| Khóa | Key | Tập thuộc tính định danh duy nhất |
| Khóa ngoại | Foreign key | Tham chiếu khóa chính của bảng khác |
| Phụ thuộc hàm | Functional dependency | $X \rightarrow Y$ |
| Dị thường | Anomaly | Lỗi do dư thừa khi thêm, xóa, sửa |
| Chuẩn hóa | Normalization | Đưa lược đồ về dạng chuẩn |
| Bản số | Cardinality | Số lượng tham gia của liên kết |
| Đại số quan hệ | Relational algebra | Ngôn ngữ truy vấn hình thức |
| Tối ưu hóa | Query optimization | Chọn kế hoạch thực thi rẻ nhất |
| Chỉ mục | Index | Cấu trúc tăng tốc tìm kiếm |
| Giao dịch | Transaction | Đơn vị thao tác nguyên tử |
| Trigger | Trigger | Thủ tục tự chạy theo sự kiện |
| Phân mảnh | Fragmentation | Chia dữ liệu thành mảnh |
| Nhân bản | Replication | Sao chép dữ liệu sang nhiều trạm |
| Cam kết hai pha | Two-phase commit | Giao thức commit giao dịch phân tán |
