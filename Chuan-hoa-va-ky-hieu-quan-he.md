# Chuẩn hóa, phụ thuộc hàm, bao đóng, khóa, và ký hiệu trong cơ sở dữ liệu quan hệ

> **Viết cho:** người học thiết kế cơ sở dữ liệu, sinh viên cần làm bài chuẩn hóa, hoặc người muốn đọc đúng các ký hiệu trong sách và đề thi.
> **Cách đọc:** phần A đến E là lý thuyết theo thứ tự logic. Phần F có bài tập có lời giải. Phần G là bảng ký hiệu tổng hợp.
> **Ngày soạn:** 2026-10-21.
> **Nhãn nguồn:** tất cả định nghĩa và thuật toán trong tài liệu này là kiến thức chuẩn của lý thuyết cơ sở dữ liệu quan hệ, có trong các giáo trình như Silberschatz–Korth–Sudarshan, Elmasri–Navathe, và Ullman. `[Trí nhớ]` đánh dấu các mục tôi chưa đối chiếu trực tiếp với trang sách trong lần này. `[Suy luận]` đánh dấu các ví dụ và diễn giải do tôi tự xây dựng.

---

## Mục lục

- **Phần A. Ký hiệu nền tảng**
  - A.1. Lược đồ quan hệ và thể hiện
  - A.2. Ký hiệu tập thuộc tính
  - A.3. Ký hiệu phép toán quan hệ
- **Phần B. Phụ thuộc hàm (functional dependency)**
  - B.1. Định nghĩa
  - B.2. Phụ thuộc hàm là tính chất của ngữ nghĩa
  - B.3. Phụ thuộc hàm tầm thường và không tầm thường
  - B.4. Hệ tiên đề Armstrong
  - B.5. Bao đóng của tập thuộc tính
  - B.6. Bao đóng của tập phụ thuộc hàm
  - B.7. Phủ tối tiểu (minimal cover)
- **Phần C. Khóa**
  - C.1. Siêu khóa, khóa ứng viên, khóa chính
  - C.2. Thuộc tính khóa và không khóa
  - C.3. Tìm khóa ứng viên
  - C.4. Khóa ngoại và toàn vẹn tham chiếu
- **Phần D. Chuẩn hóa**
  - D.1. Vì sao cần chuẩn hóa: ba loại dị thường
  - D.2. Dạng chuẩn 1NF
  - D.3. Dạng chuẩn 2NF
  - D.4. Dạng chuẩn 3NF
  - D.5. Dạng chuẩn BCNF
  - D.6. So sánh các dạng chuẩn
  - D.7. Các dạng chuẩn cao hơn (4NF, 5NF) theo khái quát
- **Phần E. Phân rã**
  - E.1. Phân rã không mất mát (lossless join)
  - E.2. Bảo toàn phụ thuộc
  - E.3. Thuật toán tổng hợp 3NF
  - E.4. Thuật toán phân rã BCNF
  - E.5. Đánh đổi giữa BCNF và bảo toàn phụ thuộc
- **Phần F. Bài tập có lời giải**
- **Phần G. Bảng ký hiệu tổng hợp**
- **Phần H. Thuật ngữ**
- **Phần I. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. KÝ HIỆU NỀN TẢNG

## A.1. Lược đồ quan hệ và thể hiện

| Ký hiệu | Đọc là | Ý nghĩa |
|---|---|---|
| R(A, B, C) | Lược đồ quan hệ R có các thuộc tính A, B, C | Cấu trúc: tên và tập thuộc tính. Không chứa dữ liệu |
| R | Tên lược đồ, cũng là tập thuộc tính khi ngữ cảnh rõ | |
| r | Thể hiện (instance) của R | Tập các bộ tại một thời điểm |
| t, t₁, t₂ | Các bộ (tuple, hàng) trong r | |
| t[X] | Giá trị của bộ t trên tập thuộc tính X | Nếu X = {A, B}, t[X] là cặp (t[A], t[B]) |
| dom(A) | Miền giá trị của thuộc tính A | Tập giá trị hợp lệ |

**Phân biệt quan trọng:** lược đồ là **khuôn**, thể hiện là **dữ liệu trong khuôn**. Phụ thuộc hàm là khẳng định về **lược đồ**, nó phải đúng với mọi thể hiện hợp lệ. [Suy luận từ định nghĩa chuẩn]

## A.2. Ký hiệu tập thuộc tính

| Ký hiệu | Ý nghĩa |
|---|---|
| X, Y, Z | Tập thuộc tính (thường viết chữ hoa đầu bảng chữ cái) |
| A, B, C | Một thuộc tính đơn |
| XY | Viết tắt của X ∪ Y, dùng khi đã rõ đây là tập |
| X ∪ Y | Hợp hai tập thuộc tính |
| X ∩ Y | Giao hai tập thuộc tính |
| X − Y | Hiệu, thuộc tính trong X không trong Y |
| ∅ | Tập rỗng |
| X ⊆ Y | X là tập con của Y |
| X ⊂ Y | X là tập con thực sự của Y |

**Quy ước viết tắt:** trong sách, khi viết `AB → C`, nghĩa là `{A, B} → {C}`. Dấu phẩy và khoảng trắng thường bị bỏ.

## A.3. Ký hiệu phép toán quan hệ

Các phép này dùng khi phân rã và kiểm tra tính đúng của phân rã.

| Ký hiệu | Tên | Ý nghĩa |
|---|---|---|
| π_X(r) | Chiếu (projection) | Giữ các cột X của r, loại bỏ trùng lặp |
| σ_θ(r) | Chọn (selection) | Giữ các bộ thỏa điều kiện θ |
| r ⋈ s | Nối tự nhiên (natural join) | Ghép các bộ có giá trị trùng trên thuộc tính chung |
| ρ | Đổi tên | Đổi tên quan hệ hoặc thuộc tính |

**Ví dụ:** nếu r(A, B, C) có các bộ (1,2,3) và (1,2,4), thì π_{A,B}(r) chỉ còn một bộ (1,2) vì phép chiếu loại trùng lặp. [Suy luận]

---

# PHẦN B. PHỤ THUỘC HÀM

## B.1. Định nghĩa

**Phụ thuộc hàm** X → Y trên lược đồ R (X, Y ⊆ R) nghĩa là: với mọi thể hiện hợp lệ r của R, và với mọi cặp bộ t₁, t₂ thuộc r:

```
nếu  t₁[X] = t₂[X]   thì   t₁[Y] = t₂[Y]
```

Đọc: "X xác định Y", hoặc "Y phụ thuộc hàm vào X". Nếu hai bộ giống nhau trên X thì phải giống nhau trên Y. [Trí nhớ, định nghĩa chuẩn]

**Ví dụ:** `MSSV → HoTen`. Nếu hai sinh viên có cùng MSSV, họ phải có cùng họ tên. Ngược lại, `HoTen → MSSV` không nhất thiết đúng, vì hai người có thể trùng tên.

## B.2. Phụ thuộc hàm là tính chất của ngữ nghĩa

Điểm này nhiều người mới bỏ qua. **Ta không thể kiểm tra một phụ thuộc hàm bằng cách nhìn vào dữ liệu hiện có.** Dữ liệu hiện tại có thể vô tình thỏa một FD không phải là FD thật.

- Nếu dữ liệu vi phạm FD, thì FD chắc chắn không đúng.
- Nếu dữ liệu thỏa FD, FD vẫn có thể sai, chỉ là chưa có ví dụ phản chứng.

**Cách xác định đúng:** hỏi ngữ nghĩa nghiệp vụ. Ví dụ: "một mã môn học có thể có hai tên khác nhau không?" Nếu không, `MaMH → TenMH` là một FD cần ghi nhận. [Suy luận]

## B.3. Phụ thuộc hàm tầm thường và không tầm thường

- **Tầm thường (trivial):** Y ⊆ X. Ví dụ `AB → A` luôn đúng, không cần ghi.
- **Không tầm thường (nontrivial):** Y ⊄ X. Ví dụ `A → B` khi B không thuộc A.
- **Hoàn toàn không tầm thường:** X ∩ Y = ∅. Đây là dạng thường được ghi trong thiết kế.

[Trí nhớ]

## B.4. Hệ tiên đề Armstrong

Armstrong (1974) đưa ra ba quy tắc suy diễn. Chúng **đúng** (sound: suy ra đúng cho mọi FD được suy ra từ tập đã cho) và **đủ** (complete: suy ra được mọi FD đúng theo F). [Trí nhớ]

| Quy tắc | Phát biểu | Ý nghĩa |
|---|---|---|
| **Phản xạ (Reflexivity)** | Nếu Y ⊆ X thì X → Y | FD tầm thường luôn đúng |
| **Tăng trưởng (Augmentation)** | Nếu X → Y thì XZ → YZ | Thêm cùng thuộc tính vào hai vế |
| **Bắc cầu (Transitivity)** | Nếu X → Y và Y → Z thì X → Z | Ghép hai FD |

**Các quy tắc suy dẫn thường dùng:**

| Quy tắc | Suy ra từ |
|---|---|
| Hợp (Union): X → Y và X → Z ⊢ X → YZ | Tăng trưởng + bắc cầu |
| Tách (Decomposition): X → YZ ⊢ X → Y | Phản xạ + bắc cầu |
| Bắc cầu giả (Pseudotransitivity): X → Y và WY → Z ⊢ WX → Z | Tăng trưởng + bắc cầu |

**Ví dụ suy dẫn:** từ `A → B` và `BC → D`, suy ra `AC → D`. Chứng minh: `AC → BC` (tăng trưởng của `A → B` với C), `BC → D` (cho sẵn), nên `AC → D` (bắc cầu). [Suy luận]

## B.5. Bao đóng của tập thuộc tính (attribute closure)

**Định nghĩa:** với tập thuộc tính X và tập phụ thuộc hàm F, bao đóng X⁺ (đọc "X cộng" hoặc "bao đóng của X") là tập **tất cả các thuộc tính A** sao cho X → A được suy ra từ F. [Trí nhớ]

Ý nghĩa: X⁺ là tập thuộc tính mà **biết X là đủ để xác định**.

**Thuật toán tính X⁺:**

```
KẾT QUẢ := X
lặp:
    với mỗi phụ thuộc hàm U → V trong F:
        nếu U ⊆ KẾT QUẢ và V ⊄ KẾT QUẢ:
            KẾT QUẢ := KẾT QUẢ ∪ V
cho đến khi KẾT QUẢ không đổi
trả về KẾT QUẢ
```

**Ví dụ:** R(A, B, C, D, E), F = {A → B, B → C, CD → E}. Tính {A, D}⁺:

| Bước | KẾT QUẢ | FD áp dụng |
|---|---|---|
| Khởi đầu | {A, D} | |
| 1 | {A, D, B} | A → B (vì A có trong KẾT QUẢ) |
| 2 | {A, D, B, C} | B → C (vì B có trong KẾT QUẢ) |
| 3 | {A, B, C, D, E} | CD → E (vì C và D đều có) |

Kết luận: {A, D}⁺ = {A, B, C, D, E}, tức {A, D} là siêu khóa của R. [Suy luận]

**Các ứng dụng của bao đóng:**
1. **Kiểm tra X → Y có suy ra từ F không:** tính X⁺, rồi xem Y ⊆ X⁺ không.
2. **Tìm khóa:** X là siêu khóa khi X⁺ = R.
3. **Kiểm tra lossless join và bảo toàn phụ thuộc** (phần E).

## B.6. Bao đóng của tập phụ thuộc hàm

**F⁺** là tập **tất cả** các phụ thuộc hàm được suy ra từ F bằng hệ tiên đề Armstrong. [Trí nhớ]

F⁺ thường rất lớn (lũy thừa theo số thuộc tính), nên trong thực hành ta không liệt kê hết, mà dùng bao đóng của tập thuộc tính để kiểm tra từng FD.

**Kiểm tra X → Y ∈ F⁺:** tính X⁺, xem Y ⊆ X⁺. Đây là điểm mấu chốt của toàn bộ lý thuyết chuẩn hóa. [Trí nhớ]

**Ký hiệu:**
- F⁺: bao đóng của F.
- X⁺: bao đóng của tập thuộc tính X dưới F.
- Hai tập F và G **tương đương** khi F⁺ = G⁺. [Trí nhớ]

## B.7. Phủ tối tiểu (minimal cover)

Phủ tối tiểu Fc của F là một tập phụ thuộc hàm tương đương F, và thỏa ba điều kiện:
1. Vế phải của mỗi FD chỉ có **một thuộc tính**.
2. Không có FD **dư thừa** (bỏ đi một FD vẫn giữ nguyên bao đóng).
3. Không có thuộc tính **dư thừa** ở vế trái (bỏ một thuộc tính khỏi vế trái vẫn giữ nguyên bao đóng).

**Thuật toán (ba bước):**

1. **Tách vế phải:** đổi `X → YZ` thành `X → Y` và `X → Z`.
2. **Loại thuộc tính dư ở vế trái:** với mỗi FD `XA → B`, nếu `(X)⁺` dưới F đã chứa B thì bỏ A khỏi vế trái.
3. **Loại FD dư thừa:** với mỗi FD `X → A`, nếu `X⁺` dưới `F − {X → A}` chứa A thì bỏ FD này.

**Ví dụ:** F = {A → BC, B → C, A → B, AB → C}.

- Bước 1: {A → B, A → C, B → C, A → B, AB → C}.
- Bước 2: với AB → C, thử bỏ A: B⁺ = {B, C} chứa C, nên A dư. Thay bằng B → C (đã có).
- Bước 3: loại trùng A → B. Với A → C, tính A⁺ dưới {A → B, B → C}: A⁺ = {A, B, C}, chứa C, nên A → C là dư.

Kết quả: **Fc = {A → B, B → C}**. [Suy luận]

---

# PHẦN C. KHÓA

## C.1. Siêu khóa, khóa ứng viên, khóa chính

| Khái niệm | Định nghĩa | Ký hiệu |
|---|---|---|
| **Siêu khóa (superkey)** | Tập K ⊆ R sao cho K⁺ = R. Mọi tập chứa một siêu khóa cũng là siêu khóa | K là siêu khóa |
| **Khóa ứng viên (candidate key)** | Siêu khóa **tối thiểu**: bỏ bất kỳ thuộc tính nào thì không còn là siêu khóa | K là khóa ứng viên |
| **Khóa chính (primary key)** | Một khóa ứng viên được chọn làm định danh chính của bảng | PK |
| **Khóa thay thế (alternate key)** | Khóa ứng viên không được chọn làm khóa chính | |

[Trí nhớ]

**Lưu ý:** mọi lược đồ có ít nhất một khóa ứng viên (vì R luôn là siêu khóa, và ta thu nhỏ dần). Một lược đồ có thể có nhiều khóa ứng viên. [Trí nhớ]

## C.2. Thuộc tính khóa và không khóa

- **Thuộc tính khóa (prime attribute):** thuộc ít nhất một khóa ứng viên.
- **Thuộc tính không khóa (non-prime attribute):** không thuộc khóa ứng viên nào.

Phân loại này là nền để định nghĩa 2NF và 3NF. [Trí nhớ]

## C.3. Tìm khóa ứng viên

**Thuật toán đơn giản:**
1. Xác định các thuộc tính **không bao giờ** nằm ở vế phải của FD (phải thuộc mọi khóa). Đặt vào tập K₀.
2. Tính K₀⁺. Nếu bằng R, K₀ là khóa duy nhất.
3. Nếu không, thêm dần các thuộc tính khác vào K₀ và kiểm tra bao đóng. Mỗi tập tìm được và tối thiểu là một khóa ứng viên.

**Mẹo thực hành:** thuộc tính chỉ xuất hiện ở vế trái của FD là ứng viên mạnh để nằm trong khóa. Thuộc tính chỉ xuất hiện ở vế phải thì không thuộc khóa nào (trừ khi đó là thuộc tính duy nhất có thể). [Suy luận]

**Ví dụ:** R(A, B, C, D), F = {A → B, B → C, C → D}. Thuộc tính A không xuất hiện ở vế phải, nên A thuộc mọi khóa. A⁺ = {A, B, C, D} = R. Khóa duy nhất là {A}. [Suy luận]

## C.4. Khóa ngoại và toàn vẹn tham chiếu

**Khóa ngoại (foreign key):** tập thuộc tính FK trong bảng con, tham chiếu đến khóa (thường là khóa chính) của bảng cha.

**Ràng buộc toàn vẹn tham chiếu (referential integrity):** mọi giá trị khác NULL của FK phải tồn tại trong khóa được tham chiếu. [Trí nhớ]

**Ký hiệu:** R.FK → S.PK, hoặc R(FK) ⊆ S(PK), đọc là "FK của R tham chiếu PK của S". [Trí nhớ]

**Các hành động khi vi phạm (SQL):**

| Hành động | Khi xóa hoặc sửa bảng cha | Ghi chú |
|---|---|---|
| `NO ACTION` / `RESTRICT` | Từ chối thao tác | Mặc định phổ biến |
| `CASCADE` | Xóa hoặc sửa theo bảng con | Dễ gây xóa dây chuyền |
| `SET NULL` | Đặt FK thành NULL | Cột FK phải cho phép NULL |
| `SET DEFAULT` | Đặt FK về giá trị mặc định | |

**Ví dụ SQL:**

```sql
CREATE TABLE lop (
    ma_lop   VARCHAR(10) PRIMARY KEY,
    khoa     VARCHAR(50) NOT NULL
);

CREATE TABLE sinh_vien (
    mssv     VARCHAR(10) PRIMARY KEY,
    ho_ten   VARCHAR(100) NOT NULL,
    ma_lop   VARCHAR(10) REFERENCES lop(ma_lop) ON DELETE RESTRICT
);
```

**Lưu ý:** khóa ngoại được kiểm tra bởi hệ quản trị, không phải chỉ là chú thích. Đây là cách biến khái niệm lý thuyết thành ràng buộc thực thi được. [Suy luận]

---

# PHẦN D. CHUẨN HÓA

## D.1. Vì sao cần chuẩn hóa: ba loại dị thường

Chuẩn hóa là quá trình tách lược đồ thành các lược đồ nhỏ hơn để **loại bỏ dư thừa và dị thường**. [Trí nhớ]

Xét bảng gộp R(MSSV, HoTen, MaLop, TenLop, MaMH, TenMH, Diem) với các FD:
- MSSV → HoTen, MSSV → MaLop
- MaLop → TenLop
- MaMH → TenMH
- {MSSV, MaMH} → Diem

| Dị thường | Mô tả | Ví dụ |
|---|---|---|
| **Chèn (insertion)** | Không thể thêm dữ liệu này nếu thiếu dữ liệu khác | Không thêm được một môn mới khi chưa có sinh viên đăng ký |
| **Xóa (deletion)** | Xóa một thông tin làm mất thông tin khác | Xóa sinh viên cuối cùng của một lớp làm mất tên lớp |
| **Cập nhật (update)** | Sửa một giá trị phải sửa nhiều chỗ, dễ sót | Đổi tên lớp phải sửa ở mọi dòng sinh viên |

Mục tiêu chuẩn hóa: mỗi FD được biểu diễn trong một lược đồ mà vế trái là khóa. [Suy luận]

## D.2. Dạng chuẩn 1NF

**Định nghĩa:** mọi thuộc tính có **giá trị nguyên tử** (atomic): không chứa tập, danh sách, hoặc nhóm lặp. [Trí nhớ]

**Vi phạm:** ô `MonHoc = "Toán, Lý, Hóa"`. Sửa: tách thành nhiều hàng, hoặc thành bảng riêng.

**Lưu ý:** 1NF là yêu cầu tối thiểu. Hầu hết hệ quản trị hiện đại không cho phép vi phạm, nhưng kiểu JSON hoặc mảng trong PostgreSQL có thể làm dạng này phức tạp hơn. [Trí nhớ]

## D.3. Dạng chuẩn 2NF

**Định nghĩa:** lược đồ ở 1NF, và **mọi thuộc tính không khóa phụ thuộc đầy đủ** vào khóa chính. Không có **phụ thuộc bộ phận (partial dependency)**: thuộc tính không khóa chỉ phụ thuộc vào một phần của khóa phức hợp. [Trí nhớ]

**Chỉ áp dụng khi khóa có nhiều thuộc tính.** Nếu khóa chỉ có một thuộc tính, lược đồ tự động ở 2NF nếu ở 1NF.

**Ví dụ vi phạm:** R(MSSV, MaMH, HoTen, Diem), khóa {MSSV, MaMH}. FD `MSSV → HoTen` là phụ thuộc bộ phận, vì HoTen chỉ cần MSSV, không cần MaMH.

**Sửa:** tách thành SV(MSSV, HoTen) và DiemThi(MSSV, MaMH, Diem).

## D.4. Dạng chuẩn 3NF

**Định nghĩa (dạng chặt cho thiết kế):** với mọi FD không tầm thường X → A trong F⁺, một trong hai điều sau đúng:
1. X là siêu khóa; hoặc
2. A là thuộc tính khóa (prime). [Trí nhớ]

Tương đương ngắn gọn: **không có thuộc tính không khóa nào phụ thuộc bắc cầu vào khóa**. [Trí nhớ]

**Ví dụ vi phạm:** R(MSSV, MaLop, TenLop), khóa {MSSV}. FD `MSSV → MaLop` và `MaLop → TenLop`, nên `MSSV → TenLop` là bắc cầu. TenLop không khóa, và MaLop không phải siêu khóa, nên vi phạm.

**Sửa:** tách thành SV(MSSV, MaLop) và Lop(MaLop, TenLop).

**Lưu ý:** định nghĩa 3NF cho phép một vi phạm nếu A là thuộc tính khóa. Điều này làm 3NF yếu hơn BCNF, như ở mục D.5. [Trí nhớ]

## D.5. Dạng chuẩn BCNF (Boyce–Codd)

**Định nghĩa:** với mọi FD không tầm thường X → A trong F⁺, **X là siêu khóa**. [Trí nhớ]

Đây là dạng chặt hơn 3NF: không cho phép vế phải là thuộc tính khóa khi vế trái không phải siêu khóa.

**Ví dụ phân biệt 3NF và BCNF:** R(SinhVien, MonHoc, GiangVien), với các FD:
- {SinhVien, MonHoc} → GiangVien
- GiangVien → MonHoc

Khóa ứng viên: {SinhVien, MonHoc} và {SinhVien, GiangVien}. Thuộc tính MonHoc là khóa (thuộc {SinhVien, MonHoc}).

- Với FD GiangVien → MonHoc: vế trái GiangVien không phải siêu khóa, nên **vi phạm BCNF**. 
- Nhưng MonHoc là thuộc tính khóa, nên **thỏa 3NF**.

Kết luận: lược đồ này ở 3NF nhưng không ở BCNF. [Suy luận từ định nghĩa]

## D.6. So sánh các dạng chuẩn

| Dạng | Điều kiện chính | Bảo toàn phụ thuộc luôn đạt? | Phân rã không mất mát luôn đạt? |
|---|---|---|---|
| 1NF | Giá trị nguyên tử | Không áp dụng | Không áp dụng |
| 2NF | Không phụ thuộc bộ phận | Không luôn đạt | Không áp dụng |
| 3NF | Không bắc cầu vào thuộc tính không khóa | **Có** | **Có** |
| BCNF | Vế trái mọi FD không tầm thường là siêu khóa | **Không** | **Có** |

Đây là kết quả quan trọng của lý thuyết: **3NF luôn đạt được cả hai tính chất**, còn BCNF chỉ luôn đạt được phân rã không mất mát, có thể làm mất phụ thuộc hàm. [Trí nhớ, kết quả chuẩn trong Silberschatz và Elmasri–Navathe]

## D.7. Các dạng chuẩn cao hơn (khái quát)

- **4NF:** loại bỏ **phụ thuộc đa trị (multivalued dependency, MVD)**, ký hiệu X ↠ Y. Ví dụ: một giáo viên có nhiều môn và nhiều lớp độc lập nhau. [Trí nhớ]
- **5NF (PJNF):** loại bỏ **phụ thuộc chiếu nối (join dependency)**. Dạng này ít được dùng trong thực tế. [Trí nhớ]

**Ký hiệu MVD:** X ↠ Y nghĩa là với mỗi giá trị X, tập các giá trị Y **độc lập** với các thuộc tính còn lại. [Trí nhớ]

---

# PHẦN E. PHÂN RÃ

## E.1. Phân rã không mất mát (lossless join)

**Phân rã** R thành R₁, R₂ (với R₁ ∪ R₂ = R) là **không mất mát** nếu với mọi thể hiện hợp lệ r:

```
r = π_{R₁}(r) ⋈ π_{R₂}(r)
```

Nghĩa là nối lại hai phần không tạo ra bộ dư thừa hoặc mất bộ. [Trí nhớ]

**Kiểm tra cho phân rã hai lược đồ (phương pháp nhanh):** phân rã R thành R₁ và R₂ là không mất mát khi:

```
(R₁ ∩ R₂) → R₁    hoặc    (R₁ ∩ R₂) → R₂    thuộc F⁺
```

Nghĩa là thuộc tính chung phải xác định một trong hai phần. [Trí nhớ]

**Ví dụ:** R(A, B, C) với F = {A → B}. Phân rã thành R₁(A, B) và R₂(A, C). R₁ ∩ R₂ = {A}. A⁺ = {A, B}, chứa R₁ = {A, B}. Nên phân rã **không mất mát**. [Suy luận]

**Ví dụ ngược:** R(A, B, C) với F = {B → C}. Phân rã thành R₁(A, B) và R₂(B, C)? Đây là không mất mát vì B → C. Thử phân rã R₁(A, B) và R₂(A, C): R₁ ∩ R₂ = {A}, A⁺ = {A} (không có FD nào cho A), không chứa cả R₁ lẫn R₂. **Mất mát.** [Suy luận]

**Kiểm tra tổng quát (thuật toán Tableau, Aho–Sagiv–Ullman):** với phân rã nhiều lược đồ, dùng bảng với ký hiệu a_j và b_ij, lặp áp dụng FD, nếu một hàng thành toàn a thì không mất mát. [Trí nhớ, không trình bày đầy đủ ở đây]

## E.2. Bảo toàn phụ thuộc

Phân rã **bảo toàn phụ thuộc** nếu:

```
(F₁ ∪ F₂ ∪ ... ∪ Fₖ)⁺ = F⁺
```

với Fᵢ là tập FD được chiếu lên Rᵢ. Mục đích: kiểm tra mọi FD ban đầu chỉ cần kiểm tra trên **một** bảng, không cần nối lại. [Trí nhớ]

**Ví dụ mất bảo toàn:** R(A, B, C), F = {AB → C, C → B}. Phân rã thành R₁(A, C) và R₂(B, C). FD `AB → C` không còn nằm trọn trong một bảng, và việc kiểm tra cần nối lại. Đây là một trường hợp mất bảo toàn. [Suy luận]

## E.3. Thuật toán tổng hợp 3NF

Đầu vào: R và F. Đầu ra: phân rã ở 3NF, không mất mát và bảo toàn phụ thuộc. [Trí nhớ, thuật toán chuẩn của Bernstein 1976]

```
1. Tìm phủ tối tiểu Fc của F (mục B.7).
2. Với mỗi FD X → A trong Fc, tạo lược đồ Rᵢ(X ∪ {A}), nếu chưa có lược đồ nào chứa XA.
3. Nếu không lược đồ nào chứa khóa ứng viên của R, chọn một khóa K và tạo lược đồ R_k(K).
4. Loại các lược đồ mà tập thuộc tính là con của lược đồ khác.
```

**Ví dụ:** R(A, B, C, D), F = {A → B, B → C}. Khóa: {A, D} (vì A⁺ = {A, B, C}, cần thêm D). Phủ tối tiểu đã là {A → B, B → C}.

- Tạo R₁(A, B) từ A → B.
- Tạo R₂(B, C) từ B → C.
- Khóa {A, D} không nằm trong lược đồ nào, nên tạo R₃(A, D).

Kết quả: R₁(A, B), R₂(B, C), R₃(A, D). [Suy luận]

## E.4. Thuật toán phân rã BCNF

```
1. Đặt Q = {R}.
2. Trong khi có lược đồ S trong Q vi phạm BCNF với FD X → A (X không là siêu khóa của S):
       tách S thành:
           S₁ = X⁺ ∩ S       (chứa X và A và các thuộc tính xác định bởi X)
           S₂ = (S − X⁺) ∪ X  
       thay S bằng S₁ và S₂ trong Q.
3. Trả về Q.
```

Thuật toán này luôn cho phân rã không mất mát, nhưng **không** đảm bảo bảo toàn phụ thuộc. [Trí nhớ]

**Ví dụ:** R(SinhVien, MonHoc, GiangVien) với F = {SinhVien MonHoc → GiangVien, GiangVien → MonHoc}.

- FD vi phạm: GiangVien → MonHoc (GiangVien không là siêu khóa).
- GiangVien⁺ = {GiangVien, MonHoc}. Tách:
  - S₁ = {GiangVien, MonHoc}
  - S₂ = {SinhVien, GiangVien}
- Kiểm tra: S₁ có khóa GiangVien. S₂ có khóa {SinhVien, GiangVien}. Cả hai đều BCNF.
- Mất bảo toàn: FD `SinhVien MonHoc → GiangVien` không còn nằm trong một bảng nào. [Suy luận]

## E.5. Đánh đổi giữa BCNF và bảo toàn phụ thuộc

| Lựa chọn | Ưu điểm | Nhược điểm |
|---|---|---|
| **3NF tổng hợp** | Không mất mát, bảo toàn phụ thuộc | Có thể còn dư thừa nhỏ (vẫn cho phép một số vi phạm BCNF) |
| **BCNF** | Loại bỏ mọi dị thường theo FD | Có thể mất phụ thuộc, khó kiểm tra ràng buộc |

**Thực hành:** nhiều thiết kế chọn 3NF, và khi cần, dùng ràng buộc hoặc trigger để kiểm tra FD bị mất. Đây là lựa chọn kỹ thuật, không có đáp án tuyệt đối. [Suy luận]

---

# PHẦN F. BÀI TẬP CÓ LỜI GIẢI

## Bài 1. Bao đóng

R(A, B, C, D, E), F = {A → B, BC → D, D → E}. Tính {A, C}⁺.

**Lời giải:**
- Khởi đầu: {A, C}.
- A → B: thêm B. {A, B, C}.
- BC → D: B và C đều có. Thêm D. {A, B, C, D}.
- D → E: thêm E. {A, B, C, D, E}.

**Kết quả:** {A, C}⁺ = R. {A, C} là siêu khóa.

## Bài 2. Khóa ứng viên

R(A, B, C, D), F = {AB → C, C → D, D → A}. Tìm mọi khóa ứng viên.

**Lời giải:**
- Thuộc tính B không xuất hiện ở vế phải của FD nào. Vậy B thuộc mọi khóa.
- B⁺ = {B}. Cần thêm.
- Thử {B, A}: {A, B}⁺ = {A, B, C, D}, vì AB → C, C → D. Là siêu khóa. Tối thiểu? Bỏ A: B⁺ = {B}, không đủ. Bỏ B: A⁺ = {A}, không đủ. Vậy {A, B} là khóa ứng viên.
- Thử {B, C}: {B, C}⁺ = {B, C, D, A} (C → D, D → A). Là siêu khóa. Tối thiểu? Bỏ B: C⁺ = {C, D, A} không có B, nhưng có đủ R? C⁺ thiếu B. Không phải siêu khóa. Bỏ C: B⁺ = {B}. Vậy {B, C} là khóa ứng viên.
- Thử {B, D}: {B, D}⁺ = {B, D, A, C} (D → A, AB → C). Là siêu khóa, tối thiểu tương tự. Khóa ứng viên.

**Kết quả:** khóa ứng viên là {A, B}, {B, C}, {B, D}. Thuộc tính khóa: A, B, C, D (tất cả). [Suy luận; bài này minh họa nhiều khóa ứng viên]

## Bài 3. Phủ tối tiểu

F = {A → BC, B → C, A → B, AB → C}. Tìm phủ tối tiểu.

**Lời giải:** xem mục B.7. **Fc = {A → B, B → C}.**

## Bài 4. Xác định dạng chuẩn

R(MSSV, MaMH, TenMH, Diem), F = {MSSV MaMH → Diem, MaMH → TenMH}. Khóa: {MSSV, MaMH}. Lược đồ đang ở dạng chuẩn nào?

**Lời giải:**
- 1NF: giả sử giá trị nguyên tử. Đạt.
- 2NF: `MaMH → TenMH` là phụ thuộc bộ phận, vì TenMH chỉ cần MaMH, là một phần của khóa. **Vi phạm 2NF.**

**Kết quả:** lược đồ chỉ đạt 1NF. [Suy luận]

**Sửa:** tách MH(MaMH, TenMH) và DiemThi(MSSV, MaMH, Diem). Cả hai đạt BCNF.

## Bài 5. Phân rã không mất mát

R(A, B, C, D), F = {A → B, C → D}. Phân rã thành R₁(A, B, C) và R₂(C, D). Kiểm tra có không mất mát không.

**Lời giải:** R₁ ∩ R₂ = {C}. C⁺ = {C, D}, chứa R₂ = {C, D}. Thỏa điều kiện `(R₁ ∩ R₂) → R₂`. **Không mất mát.** [Suy luận]

## Bài 6. Phân rã BCNF

R(A, B, C), F = {AB → C, C → B}. Phân rã sang BCNF.

**Lời giải:**
- Khóa: {A, B} và {A, C}. Đều là siêu khóa cho R.
- Kiểm tra vi phạm BCNF: FD `C → B`, vế trái C không là siêu khóa (C⁺ = {B, C}, không có A). **Vi phạm.**
- Tách bằng C⁺ = {B, C}: S₁ = {B, C}, S₂ = {A, C}.
- Kiểm tra S₁(B, C): FD C → B, C là khóa của S₁. BCNF.
- Kiểm tra S₂(A, C): không có FD nào trên S₂ ngoài tầm thường. BCNF.
- Mất bảo toàn: `AB → C` không còn nằm trong một bảng nào. [Suy luận]

**Kết quả:** BCNF gồm S₁(B, C) và S₂(A, C), với mất bảo toàn phụ thuộc.

---

# PHẦN G. BẢNG KÝ HIỆU TỔNG HỢP

| Ký hiệu | Đọc | Ý nghĩa |
|---|---|---|
| R(A, B, C) | Lược đồ R | Tên và các thuộc tính |
| r | Thể hiện | Dữ liệu của R tại một thời điểm |
| t, t₁, t₂ | Bộ | Một hàng |
| t[X] | t trên X | Giá trị các thuộc tính X của t |
| dom(A) | Miền của A | Tập giá trị hợp lệ |
| X → Y | X xác định Y | Phụ thuộc hàm |
| X ↠ Y | X đa xác định Y | Phụ thuộc đa trị |
| F | Tập phụ thuộc hàm | Các FD đã cho |
| F⁺ | F bao đóng | Mọi FD suy ra được từ F |
| X⁺ | X bao đóng | Tập thuộc tính xác định bởi X |
| Fc | Phủ tối tiểu | Tập FD tương đương tối giản |
| F ≡ G | F tương đương G | F⁺ = G⁺ |
| K | Khóa | Siêu khóa hoặc khóa ứng viên, tùy ngữ cảnh |
| PK | Khóa chính | Khóa ứng viên được chọn |
| FK | Khóa ngoại | Tham chiếu đến khóa bảng khác |
| R.FK → S.PK | | Khóa ngoại tham chiếu khóa chính |
| π_X(r) | Chiếu | Giữ các cột X |
| σ_θ(r) | Chọn | Giữ các bộ thỏa θ |
| r ⋈ s | Nối tự nhiên | Ghép các bộ có giá trị chung |
| ⊆ / ⊂ | Con / con thực sự | So sánh tập |
| ∩ / ∪ / − | Giao / hợp / hiệu | Phép toán tập |
| ∅ | Tập rỗng | |
| ⊢ | Suy ra | Dùng trong tiên đề Armstrong |
| ⇔ | Khi và chỉ khi | |

---

# PHẦN H. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Phụ thuộc hàm (FD) | Ràng buộc: hai bộ giống nhau trên X thì giống nhau trên Y |
| Phụ thuộc bộ phận | FD từ một phần của khóa phức hợp đến thuộc tính không khóa |
| Phụ thuộc bắc cầu | X → Y → Z, với Z không khóa |
| Phụ thuộc đa trị (MVD) | Các giá trị Y độc lập với các thuộc tính khác, theo từng X |
| Tiên đề Armstrong | Ba quy tắc suy diễn FD: phản xạ, tăng trưởng, bắc cầu |
| Bao đóng | Tập thuộc tính hoặc FD có thể suy ra |
| Siêu khóa | Tập xác định toàn bộ lược đồ |
| Khóa ứng viên | Siêu khóa tối thiểu |
| Khóa chính | Khóa ứng viên được chọn làm định danh |
| Khóa ngoại | Cột tham chiếu khóa của bảng khác |
| Thuộc tính khóa (prime) | Thuộc tính thuộc ít nhất một khóa ứng viên |
| Phủ tối tiểu | Tập FD tương đương, không còn thành phần dư |
| 1NF, 2NF, 3NF, BCNF | Các dạng chuẩn |
| Dị thường | Lỗi chèn, xóa, cập nhật do dư thừa |
| Phân rã không mất mát | Nối lại các phần được đúng dữ liệu gốc |
| Bảo toàn phụ thuộc | Mọi FD vẫn kiểm tra được trên một bảng |
| Tính toàn vẹn tham chiếu | Khóa ngoại phải tham chiếu được đến bản ghi tồn tại |
| Denormalization | Cố ý phi chuẩn hóa để tăng tốc đọc |

---

# PHẦN I. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## I.1. Nguồn tham khảo

Các định nghĩa và thuật toán trong tài liệu này là kiến thức chuẩn. Nên đối chiếu với các sách sau khi trích dẫn chính thức:

- Silberschatz, A., Korth, H. F., Sudarshan, S. *Database System Concepts*. Các chương về thiết kế cơ sở dữ liệu và chuẩn hóa. [Trí nhớ]
- Elmasri, R., Navathe, S. B. *Fundamentals of Database Systems*. Chương về lý thuyết thiết kế quan hệ. [Trí nhớ]
- Ullman, J. D. *Principles of Database and Knowledge-Base Systems*, tập 1. [Trí nhớ]
- Armstrong, W. W. (1974). "Dependency Structures of Data Base Relationships." *Proceedings of IFIP Congress 74*. Nguồn của hệ tiên đề Armstrong. [Trí nhớ]
- Bernstein, P. A. (1976). "Synthesizing Third Normal Form Relations from Functional Dependencies." *ACM TODS*. Nguồn thuật toán tổng hợp 3NF. [Trí nhớ]
- Codd, E. F. (1970), đã nêu trong các tài liệu trước. Đề xuất mô hình quan hệ và chuẩn hóa ban đầu. [Đã kiểm chứng trong các lần tra cứu trước, qua CACM 13(6)]

## I.2. Điểm cần xác minh

1. **Trích dẫn đầy đủ** của các sách và bài báo ở I.1 (năm, nhà xuất bản, số trang) cần đối chiếu với nguồn gốc.
2. **Thuật toán tableau** kiểm tra không mất mát cho phân rã nhiều lược đồ chỉ được nhắc tên, chưa trình bày đầy đủ.
3. **Định nghĩa 3NF** có nhiều cách phát biểu tương đương. Tài liệu này dùng dạng "siêu khóa hoặc thuộc tính khóa".
4. **Thuật toán tách BCNF** có nhiều biến thể về cách chọn FD vi phạm. Kết quả có thể khác nhau tùy thứ tự chọn. Đây là hành vi đã biết, không phải lỗi.
5. **Các ví dụ** trong tài liệu được tự xây dựng để minh họa. Chưa chạy trên công cụ kiểm chứng tự động.
6. **Bài 2 (khóa ứng viên)** có nhiều cách lập luận. Lời giải nên được kiểm tra lại bằng cách tính bao đóng của mọi tập con.
7. **Mục D.7 (4NF, 5NF)** chỉ là khái quát. Cần giáo trình để có định nghĩa và thuật toán đầy đủ.

---

**Tóm tắt để tra nhanh:**
- **FD** X → Y: hai bộ bằng nhau trên X thì bằng nhau trên Y. Là khẳng định về ngữ nghĩa.
- **Bao đóng** X⁺: tập thuộc tính xác định được từ X. Dùng để kiểm tra FD và tìm khóa.
- **Khóa**: siêu khóa tối thiểu. Khóa chính là khóa được chọn. Khóa ngoại là tham chiếu đến khóa bảng khác.
- **Chuẩn hóa**: 1NF (nguyên tử), 2NF (không phụ thuộc bộ phận), 3NF (không bắc cầu), BCNF (vế trái FD là siêu khóa).
- **Phân rã**: không mất mát (kiểm tra bằng R₁ ∩ R₂ → R₁ hoặc R₂) và bảo toàn phụ thuộc. 3NF đạt được cả hai, BCNF chỉ đạt không mất mát.
