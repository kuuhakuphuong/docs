# Ngôn ngữ Rust: từ lý thuyết đến lập trình, tự động hóa và kiểm thử

> **Viết cho:** người học Rust, lập trình viên đến từ Go, Python, JavaScript muốn hiểu vì sao Rust có ownership và borrow checker, và người muốn dùng Rust cho công cụ hiệu năng cao, hệ thống, hoặc WebAssembly.
> **Cách đọc:** Phần A và B giải thích triết lý và ownership, là nền của mọi thứ phía sau. Hiểu ownership trước khi học các phần khác.
> **Phiên bản tham chiếu:** trên releases.rs (tra cứu 2026-10-21), bản stable mới nhất là **Rust 1.99.1**. Bản kế tiếp đang ở beta là Rust 1.100.0, dự kiến 12/11/2026. Bản nightly là 1.101.0, dự kiến 24/12/2026. Trang không ghi ngày phát hành của 1.99.0 và 1.99.1.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với The Rust Programming Language (The Book) hoặc releases.rs; `[Trí nhớ]` = kiến thức nền, cần kiểm tra lại; `[Suy luận]` = tôi suy ra hoặc minh họa.

---

## Mục lục

- **Phần A. Rust là gì và vì sao được thiết kế như vậy**
  - 1. Lịch sử và mục tiêu
  - 2. Triết lý: an toàn bộ nhớ không cần GC
  - 3. Rust biên dịch và chạy thế nào
- **Phần B. Các khái niệm cốt lõi**
  - 4. Biến, kiểu, và bất biến mặc định
  - 5. Ownership: quyền sở hữu
  - 6. Move, clone, và Copy
  - 7. Borrowing: mượn tham chiếu
  - 8. Lifetime: thời gian sống của tham chiếu
  - 9. Struct, enum, và `match`
  - 10. `Option` và `Result`: xử lý vắng mặt và lỗi
  - 11. Trait: hành vi chung
  - 12. Generics
  - 13. Closure và iterator
  - 14. Panic và xử lý lỗi
  - 15. Smart pointer: `Box`, `Rc`, `Arc`
- **Phần C. Tổ chức mã: crate, module, Cargo**
  - 16. Crate và module
  - 17. Cargo và `Cargo.toml`
- **Phần D. Đồng thời**
  - 18. Thread và `Send`, `Sync`
  - 19. Kênh và async
- **Phần E. Công cụ chất lượng mã**
  - 20. `rustfmt`, `clippy`, và `cargo check`
- **Phần F. Kiểm thử**
  - 21. Unit test và integration test
  - 22. Assert, `should_panic`, và test trả về `Result`
  - 23. Doc test, benchmark, và phụ thuộc
  - 24. Độ bao phủ
- **Phần G. Tự động hóa**
  - 25. Script, build script, và tác vụ
  - 26. Tích hợp liên tục
  - 27. Đóng gói và phát hành
- **Phần H. Vai trò của Rust trong thực tế**
  - 28. Rust được dùng ở đâu
  - 29. So sánh với Go, Python, JavaScript, C++
  - 30. Các điểm cần cẩn trọng
- **Phần I. Lộ trình học**
- **Phần J. Thuật ngữ**
- **Phần K. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. RUST LÀ GÌ VÀ VÌ SAO ĐƯỢC THIẾT KẾ NHƯ VẬY

## 1. Lịch sử và mục tiêu

Rust bắt đầu như dự án cá nhân của Graydon Hoare năm 2006, được Mozilla tài trợ từ 2009, và phiên bản 1.0 ra mắt năm 2015. [Trí nhớ]

Mục tiêu: **hiệu năng ngang C/C++, nhưng an toàn bộ nhớ được kiểm tra lúc biên dịch**, không dùng bộ thu gom rác. [Trí nhớ]

## 2. Triết lý: an toàn bộ nhớ không cần GC

Trong C và C++, nhiều lỗi nghiêm trọng đến từ bộ nhớ: dùng sau khi giải phóng, giải phóng hai lần, truy cập ngoài vùng, đua dữ liệu. Các ngôn ngữ có GC (Go, Java, Python, JavaScript) tránh phần lớn lỗi này, nhưng trả giá bằng độ trễ và chi phí bộ nhớ.

Rust đạt mục tiêu thứ ba: **quyết định việc giải phóng bộ nhớ lúc biên dịch**, dựa trên ba quy tắc ownership (mục 5) và kiểm tra mượn (borrow checker). Kết quả là không cần GC, và nhiều lỗi bộ nhớ bị chặn trước khi chạy. [Đã kiểm chứng: The Book, chương ownership]

**Đánh đổi:** trình biên dịch khắt khe, học mất thời gian hơn. Một số mẫu mã an toàn nhưng khó viết lúc đầu. [Suy luận]

## 3. Rust biên dịch và chạy thế nào

```
Mã nguồn .rs
    │  rustc (qua cargo build) — kiểm tra ownership, kiểu, borrow
    ▼
Tệp nhị phân gốc (mã máy)
    │  chạy trực tiếp
    ▼
Tiến trình
```

- Rust biên dịch thành **mã máy gốc**, không có bytecode và không có máy ảo.
- Không có GC, nên không có thời gian dừng do thu gom rác. [Trí nhớ]
- Trình biên dịch LLVM là nền tảng tối ưu hóa. [Trí nhớ]

**Chạy nhanh:**

```sh
cargo new hello
cd hello
cargo run
```

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 4. Biến, kiểu, và bất biến mặc định

```rust
let x = 5;          // bất biến: không gán lại
let mut y = 10;     // có thể gán lại
y += 1;

let ten: &str = "An";
let tuoi: u32 = 25;
```

- Biến **bất biến mặc định**. Cần `mut` để thay đổi. [Đã kiểm chứng: The Book]
- Kiểu số: `i32`, `u64`, `f64`, `bool`, `char` (ký tự Unicode 4 byte).
- Kiểu chuỗi: `String` (sở hữu, có thể mở rộng) và `&str` (lát cắt, mượn). [Trí nhớ]

**Che giấu biến (shadowing):** khai báo lại cùng tên, có thể đổi kiểu:

```rust
let so = "42";
let so: i32 = so.parse().unwrap();
```

## 5. Ownership: quyền sở hữu

Ba quy tắc theo The Book: [Đã kiểm chứng]
1. Mỗi giá trị trong Rust có một **chủ sở hữu (owner)**.
2. Tại một thời điểm, chỉ có **một** chủ sở hữu.
3. Khi chủ sở hữu ra khỏi phạm vi, giá trị bị **hủy (drop)** và bộ nhớ được giải phóng.

```rust
{
    let s = String::from("xin chào");   // s sở hữu chuỗi
    println!("{s}");
}   // s ra khỏi phạm vi: bộ nhớ được giải phóng tại đây
```

Đây là cách Rust thay thế `free` của C và GC của Go, mà không cần chạy thu gom lúc thực thi. [Suy luận từ quy tắc]

## 6. Move, clone, và Copy

**Move:** gán một giá trị sở hữu bộ nhớ heap (như `String`) cho biến khác làm chuyển quyền sở hữu, và biến cũ không còn dùng được:

```rust
let s1 = String::from("hello");
let s2 = s1;
// println!("{s1}");   // lỗi E0382: s1 đã bị move
```

Lý do: nếu cả hai biến cùng giải phóng bộ nhớ, sẽ có lỗi giải phóng hai lần (double free). Rust vô hiệu hóa `s1` để chỉ `s2` giải phóng. [Đã kiểm chứng: The Book]

**Clone:** sao chép sâu, tường minh:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();   // cả hai còn dùng được
```

Gọi `clone` là tín hiệu có thể tốn chi phí. [Đã kiểm chứng: The Book]

**Copy:** kiểu có kích thước cố định, nằm hoàn toàn trên stack (số nguyên, `bool`, số thực, `char`, tuple chỉ gồm kiểu `Copy`) được sao chép khi gán, và biến cũ vẫn dùng được:

```rust
let x = 5;
let y = x;     // sao chép, x vẫn hợp lệ
```

[Đã kiểm chứng: The Book]

## 7. Borrowing: mượn tham chiếu

Thay vì chuyển quyền sở hữu, có thể **mượn** qua tham chiếu:

```rust
fn do_dai(s: &String) -> usize {   // mượn, không sở hữu
    s.len()
}

let s = String::from("hello");
let n = do_dai(&s);               // s vẫn thuộc về bên ngoài
println!("{s} có {n} ký tự");
```

**Quy tắc mượn:** [Đã kiểm chứng]
- Có thể có **nhiều tham chiếu chỉ đọc** (`&T`) cùng lúc, hoặc **đúng một tham chiếu có thể ghi** (`&mut T`), không đồng thời.
- Tham chiếu phải luôn hợp lệ, không được trỏ đến dữ liệu đã bị hủy.

```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;               // được: nhiều tham chiếu chỉ đọc
println!("{r1} {r2}");

let r3 = &mut s;           // được: r1, r2 không còn dùng nữa
r3.push_str(", world");
```

Trình kiểm tra mượn (borrow checker) bảo đảm các quy tắc này lúc biên dịch. Đây là lý do Rust tránh được đua dữ liệu trong mã an toàn. [Suy luận]

## 8. Lifetime: thời gian sống của tham chiếu

Lifetime là cách trình biên dịch theo dõi tham chiếu còn hợp lệ bao lâu. Hầu hết trường hợp được suy luận tự động. Khi cần viết tường minh:

```rust
fn dai_hon<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

`'a` nói rằng kết quả sống ít nhất bằng thời gian sống chung của hai tham chiếu đầu vào. [Trí nhớ]

**Lưu ý:** lifetime là khái niệm khó nhất với người mới. Bắt đầu với hàm nhận và trả tham chiếu, rồi mở rộng sang struct chứa tham chiếu. [Suy luận]

## 9. Struct, enum, và `match`

```rust
struct SinhVien {
    mssv: String,
    ten: String,
    diem: f64,
}

impl SinhVien {
    fn xep_loai(&self) -> &'static str {
        if self.diem >= 8.0 { "Giỏi" } else { "Khá" }
    }
}
```

**Enum** biểu diễn một trong nhiều dạng, có thể mang dữ liệu:

```rust
enum HinhHoc {
    Tron { ban_kinh: f64 },
    Vuong { canh: f64 },
}
```

**`match`** bắt buộc xử lý đủ mọi nhánh (kiểm tra đầy đủ):

```rust
fn dien_tich(h: &HinhHoc) -> f64 {
    match h {
        HinhHoc::Tron { ban_kinh } => std::f64::consts::PI * ban_kinh * ban_kinh,
        HinhHoc::Vuong { canh } => canh * canh,
    }
}
```

Thêm một dạng mới vào enum sẽ khiến mọi `match` không đầy đủ báo lỗi biên dịch. [Đã kiểm chứng: cơ chế match và enum trong The Book; kiểm tra đầy đủ là đặc điểm chung]

## 10. `Option` và `Result`: xử lý vắng mặt và lỗi

Rust không có `null` và không có ngoại lệ ẩn. Thay vào đó:

```rust
enum Option<T> { Some(T), None }
enum Result<T, E> { Ok(T), Err(E) }
```

```rust
fn tim_sv(ds: &[SinhVien], mssv: &str) -> Option<&SinhVien> {
    ds.iter().find(|s| s.mssv == mssv)
}

fn doc_so(chuoi: &str) -> Result<i32, std::num::ParseIntError> {
    chuoi.trim().parse::<i32>()
}
```

**Toán tử `?`** trả lỗi sớm từ hàm:

```rust
fn doc_tep(ten: &str) -> std::io::Result<String> {
    let noi_dung = std::fs::read_to_string(ten)?;   // lỗi thì trả về ngay
    Ok(noi_dung)
}
```

**Xử lý:** `match`, `if let`, `unwrap_or`, `map`, `and_then`, `ok_or`. Tránh `unwrap()` trong mã sản phẩm, vì nó gây panic khi lỗi. [Trí nhớ]

## 11. Trait: hành vi chung

Trait giống interface trong Go và Java, mô tả hành vi:

```rust
trait Ve {
    fn ve(&self) -> String;

    fn ve_mac_dinh(&self) -> String {       // có thể có mặc định
        format!("[{}]", self.ve())
    }
}

struct Hinh;
impl Ve for Hinh {
    fn ve(&self) -> String { "hình".into() }
}
```

**Khác với Go:** triển khai trait phải được khai báo tường minh bằng `impl Trait for Type`, không ngầm định. [Trí nhớ]

**Trait phổ biến:** `Debug` (in gỡ lỗi), `Clone`, `Copy`, `PartialEq`, `Display`, `Iterator`. Nhiều trait có thể tự sinh bằng `#[derive(...)]`:

```rust
#[derive(Debug, Clone, PartialEq)]
struct Diem { x: i32, y: i32 }
```

## 12. Generics

```rust
fn lon_nhat<T: PartialOrd + Copy>(ds: &[T]) -> T {
    let mut m = ds[0];
    for &x in ds {
        if x > m { m = x; }
    }
    m
}
```

Generics trong Rust được **đơn hình hóa (monomorphized)** lúc biên dịch: mỗi kiểu cụ thể tạo mã riêng, không có chi phí lúc chạy. [Trí nhớ]

Ràng buộc qua trait: `T: PartialOrd` hoặc mệnh đề `where` cho gọn. [Trí nhớ]

## 13. Closure và iterator

```rust
let so = vec![1, 2, 3, 4];
let chan: Vec<i32> = so.iter()
    .filter(|&&x| x % 2 == 0)
    .map(|x| x * 10)
    .collect();
// [20, 40]
```

- **Iterator** là lazy: không làm gì cho đến khi `collect`, `sum`, `for`... tiêu thụ.
- **Closure** có thể mượn hoặc chiếm quyền biến ngoài, tùy cách dùng. [Trí nhớ]

## 14. Panic và xử lý lỗi

- **`panic!`** dừng luồng khi lỗi không thể phục hồi, hoặc khi có lỗi lập trình (ví dụ chỉ số ngoài phạm vi). [Trí nhớ]
- Với lỗi có thể xử lý, dùng `Result`.
- Mặc định, panic làm thoát luồng (thread) và unwinding giải phóng tài nguyên. Có thể cấu hình `panic = "abort"` để thoát ngay. [Trí nhớ]

## 15. Smart pointer: `Box`, `Rc`, `Arc`

| Kiểu | Mục đích | Ghi chú |
|---|---|---|
| `Box<T>` | Đưa dữ liệu lên heap, sở hữu duy nhất | Dùng cho kiểu đệ quy, kiểu kích thước lớn [Trí nhớ] |
| `Rc<T>` | Nhiều chủ sở hữu trong một luồng | Đếm tham chiếu [Trí nhớ] |
| `Arc<T>` | Nhiều chủ sở hữu giữa nhiều luồng | Đếm tham chiếu nguyên tử [Trí nhớ] |
| `RefCell<T>` | Mượn có kiểm tra lúc chạy | Dùng khi cần thay đổi qua tham chiếu chung [Trí nhớ] |

**Lưu ý:** `Rc` và `RefCell` là lựa chọn có chủ đích. Dùng quá nhiều làm mất lợi ích kiểm tra lúc biên dịch. [Suy luận]

---

# PHẦN C. TỔ CHỨC MÃ: CRATE, MODULE, CARGO

## 16. Crate và module

- **Crate:** đơn vị biên dịch. Một crate là một thư viện (`lib.rs`) hoặc một chương trình (`main.rs`). [Đã kiểm chứng: The Book]
- **Module:** tổ chức mã trong crate bằng `mod`, mặc định riêng tư, công khai bằng `pub`.

```rust
mod toan {
    pub fn cong(a: i32, b: i32) -> i32 { a + b }
    fn noi_bo() {}              // riêng tư
}

use toan::cong;
```

**Cấu trúc tệp:** `mod xu_ly;` tìm tệp `xu_ly.rs` hoặc `xu_ly/mod.rs` cùng thư mục. [Trí nhớ]

## 17. Cargo và `Cargo.toml`

Cargo là hệ thống build và quản lý gói chính thức của Rust. [Đã kiểm chứng: The Book]

```toml
[package]
name = "congcu"
version = "0.1.0"
edition = "2024"

[dependencies]
serde = { version = "1", features = ["derive"] }

[dev-dependencies]
pretty_assertions = "1"
```

[Phiên bản edition và thư viện trong ví dụ là minh họa; kiểm tra trên crates.io]

| Lệnh | Tác dụng |
|---|---|
| `cargo new <tên>` | Tạo dự án mới |
| `cargo build` | Biên dịch (debug) |
| `cargo build --release` | Biên dịch tối ưu |
| `cargo run` | Biên dịch và chạy |
| `cargo test` | Chạy kiểm thử |
| `cargo add <gói>` | Thêm phụ thuộc |
| `cargo update` | Cập nhật phụ thuộc theo ràng buộc |
| `cargo doc --open` | Sinh và mở tài liệu |

**Tệp `Cargo.lock`:** ghim phiên bản phụ thuộc. Với ứng dụng nên commit; với thư viện, thường không. [Trí nhớ]

---

# PHẦN D. ĐỒNG THỜI

## 18. Thread và `Send`, `Sync`

```rust
use std::thread;

let handle = thread::spawn(|| {
    println!("chạy trong luồng khác");
});
handle.join().unwrap();
```

**`Send` và `Sync`** là trait đánh dấu do trình biên dịch kiểm tra: [Đã kiểm chứng: đây là điểm nhấn trong The Book về đồng thời]
- `Send`: giá trị có thể chuyển sang luồng khác.
- `Sync`: tham chiếu đến giá trị có thể dùng từ nhiều luồng.

Nhờ đó, đua dữ liệu được chặn lúc biên dịch trong mã an toàn.

**Chia sẻ trạng thái:** `Arc<Mutex<T>>` là mẫu phổ biến:

```rust
use std::sync::{Arc, Mutex};

let dem = Arc::new(Mutex::new(0));
let d2 = Arc::clone(&dem);
// ... trong luồng: *d2.lock().unwrap() += 1;
```

## 19. Kênh và async

**Kênh (channel)** truyền dữ liệu giữa luồng:

```rust
use std::sync::mpsc;

let (tx, rx) = mpsc::channel();
std::thread::spawn(move || tx.send(42).unwrap());
println!("{}", rx.recv().unwrap());
```

**Async/await:** Rust có `async`/`await` nhưng **không có runtime tích hợp**. Cần chọn thư viện như Tokio hoặc async-std. [Trí nhớ]

```rust
// Cargo.toml: tokio = { version = "1", features = ["full"] }
#[tokio::main]
async fn main() {
    let ket = tokio::spawn(async { 1 + 1 }).await.unwrap();
    println!("{ket}");
}
```

[Trí nhớ; kiểm tra API Tokio hiện hành]

---

# PHẦN E. CÔNG CỤ CHẤT LƯỢNG MÃ

## 20. `rustfmt`, `clippy`, và `cargo check`

| Công cụ | Chức năng | Ghi chú |
|---|---|---|
| `cargo check` | Kiểm tra lỗi nhanh, không sinh mã máy | Nhanh hơn `cargo build` [Trí nhớ] |
| `rustfmt` | Định dạng chuẩn | `cargo fmt` [Trí nhớ] |
| `clippy` | Lint nhiều mẫu mã không tốt | `cargo clippy` [Trí nhớ] |

```sh
cargo check
cargo fmt --check
cargo clippy -- -D warnings
```

**Gợi ý:** trong CI, chạy `cargo fmt --check` và `cargo clippy` với `-D warnings` để coi cảnh báo là lỗi. [Suy luận]

---

# PHẦN F. KIỂM THỬ

## 21. Unit test và integration test

**Unit test** nằm trong cùng tệp với mã, trong module `tests` có `#[cfg(test)]`:

```rust
pub fn cong(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn cong_hai_so_duong() {
        assert_eq!(cong(2, 3), 5);
    }
}
```

**Đã kiểm chứng:** `#[cfg(test)]` chỉ biên dịch module khi chạy test; `use super::*;` đưa mục của module cha vào; hàm không có `#[test]` là hàm trợ giúp, nên mọi test phải được đánh dấu rõ. [Đã kiểm chứng: The Book, chương viết kiểm thử]

**Integration test** nằm trong thư mục `tests/` ở gốc crate, được biên dịch như crate riêng và chỉ dùng được API công khai. [Trí nhớ; chi tiết trong chương 11 của The Book chưa đối chiếu hết trong lần này]

```
congcu/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    └── tich_hop.rs
```

## 22. Assert, `should_panic`, và test trả về `Result`

```rust
#[test]
fn assert_co_ban() {
    assert!(cong(1, 1) == 2);
    assert_eq!(cong(2, 2), 4);
    assert_ne!(cong(2, 2), 5);
    assert_eq!(cong(2, 3), 5, "sai với đầu vào {}", 2);   // thông báo tùy chỉnh
}

#[test]
#[should_panic(expected = "không hợp lệ")]
fn mong_doi_panic() {
    kiem_tra_tuoi(-1);
}

#[test]
fn test_tra_ve_result() -> Result<(), String> {
    let ket = cong(2, 3);
    if ket == 5 { Ok(()) } else { Err(format!("sai: {ket}")) }
}
```

**Đã kiểm chứng:** `assert!`, `assert_eq!`, `assert_ne!`, `#[should_panic]` với tham số `expected`, và test trả về `Result<(), String>` cho phép dùng `?`. Không kết hợp được `#[should_panic]` với test trả về `Result`. [Đã kiểm chứng: The Book, chương viết kiểm thử]

**Lưu ý:** `assert_eq!` yêu cầu kiểu triển khai `PartialEq` và `Debug`, thường bằng `#[derive(PartialEq, Debug)]`. [Đã kiểm chứng]

**Chạy:**

```sh
cargo test                    # chạy tất cả
cargo test cong              # lọc theo tên
cargo test -- --nocapture    # hiện đầu ra in ra
cargo test -- --test-threads=1
```

Mỗi test chạy trong luồng riêng, và test thất bại khi hàm panic. [Đã kiểm chứng: The Book]

## 23. Doc test, benchmark, và phụ thuộc

**Doc test:** ví dụ trong tài liệu được chạy như test:

```rust
/// Cộng hai số.
///
/// ```
/// assert_eq!(congcu::cong(2, 3), 5);
/// ```
pub fn cong(a: i32, b: i32) -> i32 { a + b }
```

`cargo test` có phần `Doc-tests` riêng. [Đã kiểm chứng: The Book đề cập phần doc tests trong đầu ra]

**Benchmark:** benchmark chuẩn từng là tính năng nightly. Có thể dùng crate như `criterion` cho benchmark ổn định. [Đã kiểm chứng một phần: The Book ghi "benchmarks, nightly-only at the time of writing"; trạng thái hiện tại cần kiểm tra]

**Phụ thuộc kiểm thử:** `[dev-dependencies]` trong Cargo.toml chỉ dùng khi test. [Đã kiểm chứng: đặc tả Cargo]

**Mock:** Rust không có mock tích hợp như Python hay Go. Mẫu phổ biến: định nghĩa trait cho phụ thuộc, rồi viết bản giả thực thi trait đó. Thư viện như `mockall` tự sinh mock. [Trí nhớ]

## 24. Độ bao phủ

```sh
cargo install cargo-llvm-cov
cargo llvm-cov
```

Công cụ bao phủ dựa trên LLVM, cần cài riêng. [Trí nhớ; kiểm tra tên và cách cài theo tài liệu công cụ]

**Cách hiểu:** độ bao phủ cho biết dòng đã chạy, không đảm bảo đúng. Với Rust, trình biên dịch đã chặn nhiều lỗi bộ nhớ, nên test tập trung vào logic. [Suy luận]

---

# PHẦN G. TỰ ĐỘNG HÓA

## 25. Script, build script, và tác vụ

**Build script (`build.rs`)** chạy trước khi biên dịch crate, dùng để sinh mã hoặc liên kết thư viện C: [Trí nhớ]

```rust
// build.rs
fn main() {
    println!("cargo:rerun-if-changed=src/");
}
```

**Tác vụ lặp lại:** dùng Makefile, `just` (công cụ lệnh đơn giản), hoặc `xtask` (mẫu cộng đồng: crate phụ viết tác vụ bằng Rust). [Trí nhớ]

**Alias trong Cargo** cho lệnh thường dùng: `.cargo/config.toml`:

```toml
[alias]
ci = "clippy -- -D warnings"
```

```sh
cargo ci
```

## 26. Tích hợp liên tục

Pipeline điển hình:

```
1. Cài toolchain ổn định:       rustup
2. Kiểm tra định dạng:          cargo fmt --check
3. Lint:                        cargo clippy -- -D warnings
4. Kiểm tra biên dịch:          cargo check
5. Kiểm thử:                    cargo test
6. Build phát hành:             cargo build --release
```

**Ví dụ GitHub Actions** (minh họa):

```yaml
name: ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt, clippy
      - run: cargo fmt --check
      - run: cargo clippy -- -D warnings
      - run: cargo test
```

[Suy luận; tên action cần kiểm tra]

## 27. Đóng gói và phát hành

- Tệp nhị phân: `cargo build --release` tạo tệp trong `target/release/`. [Đã kiểm chứng: The Book]
- Thư viện: `cargo publish` lên crates.io sau khi kiểm tra `cargo package`. [Trí nhớ]
- Biên dịch chéo: dùng `rustup target add` cho nền tảng đích, hoặc công cụ như `cross`. [Trí nhớ]

---

# PHẦN H. VAI TRÒ CỦA RUST TRONG THỰC TẾ

## 28. Rust được dùng ở đâu

| Lĩnh vực | Ví dụ | Lý do |
|---|---|---|
| Công cụ hệ thống | Trình biên dịch, công cụ dòng lệnh | Hiệu năng và an toàn |
| Hạ tầng | Dịch vụ mạng, proxy | Không GC, độ trễ thấp |
| WebAssembly | Module hiệu năng cao trong trình duyệt | Biên dịch sang Wasm |
| Nhúng và OS | Firmware, thành phần kernel | Kiểm soát bộ nhớ |
| Công cụ JavaScript và Python | Bộ công cụ build, phân tích, lint nhanh | Nhanh và an toàn |

[Suy luận và kiến thức nền; không đầy đủ]

## 29. So sánh với Go, Python, JavaScript, C++

| Tiêu chí | Rust | Go | Python | JavaScript | C++ |
|---|---|---|---|---|---|
| Quản lý bộ nhớ | Ownership, không GC | GC | GC (đếm tham chiếu + GC) | GC | Thủ công (RAII) |
| Hiệu năng | Rất cao | Cao | Thấp hơn | Trung bình đến cao | Rất cao |
| Độ khó học | Cao | Thấp đến trung bình | Thấp | Thấp đến trung bình | Cao |
| Đồng thời | Thread, Send/Sync, async | Goroutine | GIL, asyncio | Event loop | Thread thủ công |
| Lỗi bộ nhớ | Chặn lúc biên dịch | Giảm nhờ GC | Giảm nhờ GC | Giảm nhờ GC | Dễ xảy ra |

[Suy luận; so sánh khái quát]

## 30. Các điểm cần cẩn trọng

- **Đường cong học tập:** ownership và lifetime là khái niệm mới với đa số người. Mất thời gian ban đầu, nhưng ít lỗi sau này. [Suy luận]
- **Thời gian biên dịch:** dự án lớn có thể biên dịch lâu. Dùng `cargo check` khi phát triển. [Trí nhớ]
- **`unwrap()` và `expect()`:** dễ gây panic, dùng có chủ đích. [Trí nhớ]
- **`unsafe`:** cho phép bỏ qua một số kiểm tra. Khối `unsafe` cần được kiểm chứng cẩn thận và giữ nhỏ. [Trí nhớ]
- **Async không có runtime sẵn:** cần chọn runtime (Tokio, async-std). [Trí nhớ]
- **Phiên bản:** Rust phát hành định kỳ, bản mới nhất là 1.99.1 trong lần tra cứu. Đọc ghi chú phát hành khi nâng cấp. [Đã kiểm chứng: releases.rs]

---

# PHẦN I. LỘ TRÌNH HỌC

1. **Cú pháp cơ bản:** biến, kiểu, hàm, `if`, `loop`, `match`.
2. **Ownership, borrowing, lifetime:** đọc kỹ chương ownership của The Book. Đây là phần quan trọng nhất.
3. **Struct, enum, `Option`, `Result`:** mô hình hóa dữ liệu và xử lý lỗi.
4. **Trait và generics:** viết hàm dùng chung cho nhiều kiểu.
5. **Iterator và closure:** xử lý dữ liệu theo chuỗi biến đổi.
6. **Kiểm thử:** unit test, integration test, `cargo test`.
7. **Cargo và crate:** thêm phụ thuộc, tổ chức module.
8. **Đồng thời:** thread, channel, `Arc<Mutex<T>>`, rồi async với Tokio.
9. **Dự án:** một CLI có kiểm thử, `clippy`, `fmt`, và CI.

**Nguồn học chính thức:** The Rust Programming Language (doc.rust-lang.org/book), Rust by Example, và Cargo Book. [Đã kiểm chứng: The Book đã đối chiếu trong lần này]

---

# PHẦN J. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Ownership | Mỗi giá trị có một chủ sở hữu, hủy khi chủ ra khỏi phạm vi |
| Move | Chuyển quyền sở hữu, biến cũ không dùng được |
| Clone | Sao chép sâu tường minh |
| Copy | Sao chép ngầm cho kiểu nhỏ trên stack |
| Borrowing | Mượn tham chiếu, không sở hữu |
| Borrow checker | Kiểm tra quy tắc mượn lúc biên dịch |
| Lifetime | Thời gian sống của tham chiếu |
| Trait | Tập hành vi có thể triển khai cho kiểu |
| Generic | Kiểu tham số hóa, đơn hình hóa lúc biên dịch |
| Option / Result | Kiểu cho vắng mặt và lỗi |
| Panic | Dừng luồng vì lỗi không phục hồi |
| Crate | Đơn vị biên dịch và đóng gói |
| Module | Đơn vị tổ chức mã trong crate |
| Cargo | Công cụ build và quản lý gói |
| Smart pointer | Con trỏ có ngữ nghĩa sở hữu, như `Box`, `Rc`, `Arc` |
| `Send` / `Sync` | Trait đánh dấu an toàn khi dùng đa luồng |
| Unsafe | Khối cho phép bỏ qua một số kiểm tra |
| Clippy | Công cụ lint của Rust |
| Doc test | Ví dụ trong tài liệu được chạy như test |
| Edition | Phiên bản ngôn ngữ theo chu kỳ (ví dụ 2024) |

---

# PHẦN K. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## K.1. Nguồn đã đối chiếu trong lần này

- **releases.rs** (lịch phát hành): https://releases.rs/ [đã đọc. Ghi nhận: stable mới nhất 1.99.1; beta 1.100.0 dự kiến 12/11/2026; nightly 1.101.0 dự kiến 24/12/2026. Không có ngày phát hành cho 1.99.0 và 1.99.1]
- **The Rust Programming Language, chương 4 (Ownership)**: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html [đã đọc: quy tắc ownership, move, clone, Copy]
- **The Rust Programming Language, chương 11 (Writing Tests)**: https://doc.rust-lang.org/book/ch11-01-writing-tests.html [đã đọc: `#[test]`, assert, `should_panic`, test trả về `Result`, cấu trúc unit test]

## K.2. Nguồn chưa đối chiếu

- Chương 4 phần borrowing và slices, chương 10 (generics, trait, lifetime), chương 15 (smart pointer), chương 16 (đồng thời), chương 11 phần integration test và điều khiển chạy test.
- Cargo Book, Rust by Example, tài liệu Tokio, `clippy`, `rustfmt`, `cargo-llvm-cov`, `mockall`, `criterion`.
- Đặc tả ngôn ngữ (Rust Reference).

## K.3. Điểm cần xác minh

1. **Phiên bản edition:** ví dụ dùng `edition = "2024"` (minh họa). Cần kiểm tra edition mới nhất và yêu cầu của từng dự án.
2. **Ngày phát hành 1.99.0 và 1.99.1:** trang releases.rs không ghi. Cần đối chiếu blog Rust.
3. **Trạng thái benchmark:** The Book (bản đã đọc) ghi benchmark là nightly. Trạng thái hiện tại cần kiểm tra.
4. **Các ví dụ async với Tokio** dùng API hiện hành theo kiến thức nền. Cần kiểm tra tài liệu Tokio.
5. **Cú pháp `cargo llvm-cov`** và cách cài: kiến thức nền, cần kiểm tra.
6. **Ví dụ mã chưa được biên dịch và chạy.** Nên thử trên máy, đặc biệt các đoạn lifetime, `Arc<Mutex<T>>`, và kênh.
7. **So sánh ngôn ngữ** là khái quát, không phải đo lường.

---

**Tóm tắt để tra nhanh:**
- **Rust** = hiệu năng ngang C/C++, an toàn bộ nhớ kiểm tra lúc biên dịch, không GC.
- **Ownership:** mỗi giá trị một chủ sở hữu; move khi gán; clone để sao chép sâu; Copy cho kiểu nhỏ.
- **Borrowing:** nhiều `&T` hoặc một `&mut T`, không đồng thời; lifetime cho tham chiếu.
- **Kiểu:** `Option` và `Result` thay null và ngoại lệ; `match` kiểm tra đầy đủ.
- **Đồng thời:** `Send`/`Sync` kiểm tra lúc biên dịch; async cần runtime như Tokio.
- **Kiểm thử:** `#[test]`, `assert_eq!`, `should_panic`, unit test trong `#[cfg(test)]`, integration test trong `tests/`.
- **Công cụ:** `cargo` cho build, test, và dependency; `rustfmt` và `clippy` cho chất lượng mã.
- **Cẩn trọng:** học ownership mất thời gian, `unwrap` gây panic, `unsafe` cần kiểm chứng, và cần chọn runtime cho async.
