# Ngôn ngữ lập trình Go: từ lý thuyết đến lập trình, tự động hóa và kiểm thử

> **Viết cho:** người mới học Go, lập trình viên từ ngôn ngữ khác chuyển sang, và người muốn hiểu vai trò của Go trong hệ thống thực tế.
> **Cách đọc:** theo phần. Phần A và B giải thích "vì sao Go được thiết kế như vậy", là nền để hiểu phần C trở đi. Các đoạn mã có chú thích, có thể chạy được với Go hiện hành.
> **Phiên bản tham chiếu:** Go 1.27.x (bản vá mới nhất trong lần tra cứu là go1.27.3, phát hành 2026-10-19).
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần tra cứu này với go.dev; `[Trí nhớ]` = kiến thức nền, nên kiểm tra lại với tài liệu chính thức; `[Suy luận]` = tôi suy ra hoặc minh họa.

---

## Mục lục

- **Phần A. Go là gì và vì sao được tạo ra**
  - 1. Lịch sử và mục tiêu
  - 2. Triết lý thiết kế
  - 3. Go được biên dịch và chạy như thế nào
- **Phần B. Các khái niệm cốt lõi của ngôn ngữ**
  - 4. Cú pháp cơ bản và kiểu dữ liệu
  - 5. Hàm, nhiều giá trị trả về, và `defer`
  - 6. Slice và map
  - 7. Struct, phương thức, và nhúng (embedding)
  - 8. Interface: hành vi thay vì khai báo
  - 9. Xử lý lỗi
  - 10. Panic và recover
  - 11. Generics (từ Go 1.18)
- **Phần C. Đồng thời (concurrency)**
  - 12. Goroutine
  - 13. Channel và select
  - 14. Các mẫu đồng thời thường gặp
  - 15. Context và hủy tác vụ
  - 16. Đồng thời và song song khác nhau thế nào
- **Phần D. Tổ chức mã: package và module**
  - 17. Package
  - 18. Module và go.mod
  - 19. Hiển thị công khai và nội bộ
- **Phần E. Công cụ chuẩn của Go**
  - 20. Bảng công cụ dòng lệnh
  - 21. Định dạng và kiểm tra tĩnh
- **Phần F. Kiểm thử**
  - 22. Kiểm thử đơn vị
  - 23. Table-driven test và subtest
  - 24. Kiểm thử HTTP và kiểm thử với phụ thuộc
  - 25. Benchmark
  - 26. Fuzzing
  - 27. Phát hiện data race
  - 28. Độ bao phủ (coverage)
- **Phần G. Tự động hóa**
  - 29. Makefile và tác vụ lặp lại
  - 30. go generate
  - 31. Tích hợp liên tục (CI)
  - 32. Đóng gói và phát hành
  - 33. Đo và tối ưu hiệu năng
- **Phần H. Vai trò của Go trong thực tế**
  - 34. Go được dùng ở đâu
  - 35. Go so với một số ngôn ngữ khác
  - 36. Các điểm cần cẩn trọng
- **Phần I. Lộ trình học**
- **Phần J. Thuật ngữ**
- **Phần K. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. GO LÀ GÌ VÀ VÌ SAO ĐƯỢC TẠO RA

## 1. Lịch sử và mục tiêu

Go được phát triển tại Google bởi Robert Griesemer, Rob Pike, và Ken Thompson, bắt đầu năm 2007, công bố công khai năm 2009. [Trí nhớ]

Động lực: các hệ thống lớn của Google được viết bằng C++ và Java, và gặp các vấn đề:
- **Thời gian biên dịch dài**, làm chậm vòng lặp phát triển.
- **Sự phức tạp** của ngôn ngữ, khó đọc mã của người khác.
- **Đồng thời khó viết đúng** trên nhiều lõi CPU.

Go được thiết kế để **đơn giản, biên dịch nhanh, dễ đọc, và xử lý đồng thời tốt**. [Trí nhớ]

## 2. Triết lý thiết kế

Các nguyên tắc nổi bật trong thiết kế Go:

| Nguyên tắc | Ý nghĩa dễ hiểu | Hệ quả |
|---|---|---|
| **Đơn giản** | Ít khái niệm, ít cách làm cùng một việc | Không có kế thừa lớp, không có ngoại lệ (exception) |
| **Rõ ràng hơn thông minh** | Viết ra điều bạn muốn, không che giấu | Lỗi được trả về như giá trị, không nhảy ngầm |
| **Kết hợp hơn kế thừa** | Ghép các phần nhỏ thay vì cây phân cấp | Embedding và interface thay cho lớp cha con |
| **Công cụ tích hợp** | Format, test, benchmark có sẵn trong bộ công cụ | Ít tranh cãi về phong cách, dự án đồng nhất |
| **Đồng thời là công dân hạng nhất** | Goroutine và channel là một phần ngôn ngữ | Viết chương trình nhiều luồng ít phức tạp hơn |

Trong tài liệu chính thức Effective Go, mục tiêu được thể hiện qua các khuyến nghị về cách viết mã "idiomatic", tức theo phong cách chuẩn của cộng đồng Go. [Đã kiểm chứng]

## 3. Go được biên dịch và chạy như thế nào

**Quy trình:**

```
Mã nguồn .go
    │  go build (biên dịch)
    ▼
Tệp nhị phân đơn (static binary)  ← chứa cả thư viện và bộ chạy (runtime)
    │  chạy trực tiếp
    ▼
Tiến trình hệ điều hành
```

Điểm khác biệt quan trọng:

- **Biên dịch thành mã máy trực tiếp**, không qua bytecode như Java hay máy ảo như .NET. Đây là lý do Go nhanh khởi động. [Trí nhớ]
- **Tệp nhị phân thường tự chứa** (static), có thể chạy trên máy khác cùng hệ điều hành và kiến trúc mà không cần cài thêm. Thường là một ưu điểm lớn khi triển khai. [Trí nhớ]
- **Có bộ chạy (runtime)** bên trong tệp nhị phân, lo quản lý goroutine, bộ nhớ, và thu gom rác. [Trí nhớ]
- **Có bộ thu gom rác (garbage collector)**, nên lập trình viên không tự giải phóng bộ nhớ. [Đã kiểm chứng: nguồn Effective Go nói về cấp phát và con trỏ, GC nằm trong phần runtime]

**Chạy nhanh, không tạo tệp:**

```sh
go run main.go
```

**Biên dịch thành tệp:**

```sh
go build -o app .
```

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 4. Cú pháp cơ bản và kiểu dữ liệu

**Chương trình tối thiểu:**

```go
package main

import "fmt"

func main() {
    fmt.Println("Xin chào, Go!")
}
```

- Mỗi tệp thuộc một **package**. Chương trình chạy bắt đầu từ `package main` và hàm `main`.
- `import` khai báo gói cần dùng. Gói không dùng sẽ gây lỗi biên dịch, giúp mã sạch.

**Khai báo biến:**

```go
var ten string = "An"    // khai báo đầy đủ
tuoi := 25                // khai báo ngắn, Go tự suy ra kiểu int
const PI = 3.14159        // hằng số
```

- `:=` chỉ dùng bên trong hàm, và ít nhất một biến bên trái phải mới.
- Biến khai báo mà không dùng sẽ gây lỗi biên dịch.

**Kiểu cơ bản:**

| Nhóm | Kiểu | Ghi chú |
|---|---|---|
| Số nguyên | `int`, `int8`, `int64`, `uint`, ... | `int` có kích thước phụ thuộc kiến trúc |
| Số thực | `float32`, `float64` | |
| Chuỗi | `string` | Bất biến, mã hóa UTF-8 |
| Luận lý | `bool` | `true`, `false` |
| Byte, rune | `byte` (= uint8), `rune` (= int32) | `rune` là một ký tự Unicode |

**Không có chuyển đổi kiểu ngầm:**

```go
var a int = 10
var b float64 = float64(a)   // phải chuyển kiểu tường minh
```

Điều này tránh nhiều lỗi âm thầm. [Trí nhớ]

**Vòng lặp duy nhất là `for`:**

```go
for i := 0; i < 5; i++ { }        // kiểu C
for n > 0 { n-- }                  // kiểu while
for { break }                      // vô hạn
for i, v := range danhSach { }     // duyệt phần tử
```

**Điều kiện `if`** không cần ngoặc đơn, nhưng bắt buộc có ngoặc nhọn. Có thể khai báo ngay trong `if`:

```go
if err := thucHien(); err != nil {
    return err
}
```

**`switch`** không tự rơi xuống (no fallthrough mặc định), nên không cần `break` trong mỗi nhánh.

## 5. Hàm, nhiều giá trị trả về, và `defer`

**Hàm trả về nhiều giá trị** là kiểu thông dụng nhất:

```go
func chia(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("không chia cho 0")
    }
    return a / b, nil
}
```

Mẫu `(kết quả, error)` là cách xử lý lỗi chuẩn trong Go (xem mục 9).

**`defer`** lên lịch một lời gọi chạy khi hàm kết thúc, kể cả khi có lỗi. Dùng để đóng tài nguyên:

```go
func docTep(ten string) error {
    f, err := os.Open(ten)
    if err != nil {
        return err
    }
    defer f.Close()          // chạy khi hàm kết thúc

    // ... đọc tệp ...
    return nil
}
```

**Lưu ý:** các `defer` chạy theo thứ tự ngược (LIFO). [Trí nhớ]

## 6. Slice và map

**Slice** là một cửa sổ linh hoạt trên mảng, dùng nhiều hơn mảng cố định:

```go
so := []int{1, 2, 3}
so = append(so, 4)        // thêm phần tử
fmt.Println(len(so))      // độ dài: 4
fmt.Println(cap(so))      // sức chứa: có thể lớn hơn
```

**Điều cần hiểu:** slice là **tham chiếu tới mảng bên dưới**. Hai slice có thể cùng chia sẻ một mảng. Thay đổi phần tử qua một slice có thể thấy được ở slice kia. [Trí nhớ]

**Map** là bảng băm:

```go
tuoi := map[string]int{"An": 25, "Bình": 30}
tuoi["Chi"] = 22
if t, ok := tuoi["An"]; ok {
    fmt.Println("tuổi An:", t)
}
delete(tuoi, "Bình")
```

**Lưu ý:** đọc khóa không tồn tại trả về giá trị mặc định (0 cho int), không lỗi. Dùng dạng `v, ok :=` để phân biệt. Map không an toàn cho đồng thời nếu không có đồng bộ hóa (mục 14). [Đã kiểm chứng: đồng bộ hóa là chủ đề được Effective Go nêu khi nói về chia sẻ bộ nhớ]

## 7. Struct, phương thức, và nhúng (embedding)

**Struct** gom các trường lại:

```go
type SinhVien struct {
    MSSV string
    Ten  string
    Diem float64
}

sv := SinhVien{MSSV: "SV01", Ten: "An", Diem: 8.5}
```

**Phương thức** là hàm gắn với một kiểu:

```go
func (s SinhVien) XepLoai() string {
    if s.Diem >= 8 {
        return "Giỏi"
    }
    return "Khá"
}

func (s *SinhVien) CongDiem(d float64) {   // nhận con trỏ để sửa đổi
    s.Diem += d
}
```

**Quy tắc chọn receiver:**
- **Con trỏ (`*T`)** khi cần sửa đổi, hoặc struct lớn (tránh sao chép).
- **Giá trị (`T`)** khi chỉ đọc, struct nhỏ, và muốn bất biến.

**Nhúng (embedding):** thay cho kế thừa, Go đưa một kiểu vào kiểu khác, và các phương thức được "thăng cấp":

```go
type Nguoi struct {
    Ten string
}

func (n Nguoi) Chao() string { return "Xin chào, " + n.Ten }

type GiangVien struct {
    Nguoi            // nhúng: GiangVien có Ten và Chao()
    Khoa string
}

gv := GiangVien{Nguoi: Nguoi{Ten: "Dũng"}, Khoa: "CNTT"}
fmt.Println(gv.Chao())   // gọi phương thức của Nguoi
```

**Đây không phải kế thừa:** `GiangVien` không là một `Nguoi`, mà **chứa** một `Nguoi` và có thể dùng các phương thức của nó. Sự khác biệt này quan trọng khi thiết kế. [Suy luận từ thiết kế]

## 8. Interface: hành vi thay vì khai báo

**Interface** mô tả tập phương thức. Một kiểu **tự động** thỏa interface nếu có đủ phương thức, không cần khai báo rõ ràng:

```go
type Ghi interface {
    Write(p []byte) (n int, err error)
}

type TepLog struct{ /* ... */ }

func (t TepLog) Write(p []byte) (int, error) {
    // ghi dữ liệu
    return len(p), nil
}

// TepLog tự động thỏa Ghi, không cần "implements"
var g Ghi = TepLog{}
```

**Tại sao quan trọng:**
- Hàm nhận interface có thể nhận **bất kỳ** kiểu nào có hành vi phù hợp. Đây là cách viết mã linh hoạt và dễ kiểm thử.
- Interface nhỏ là phong cách chuẩn: chỉ một hoặc vài phương thức. Đặt tên theo hành vi với hậu tố `-er`: `Reader`, `Writer`. [Đã kiểm chứng: Effective Go]
- Kiểm tra biên dịch để chắc một kiểu thỏa interface:

```go
var _ Ghi = (*TepLog)(nil)
```

**Kiểu assertion và type switch:**

```go
func mo(v interface{}) {
    switch x := v.(type) {
    case int:
        fmt.Println("số nguyên", x)
    case string:
        fmt.Println("chuỗi", x)
    default:
        fmt.Println("kiểu khác")
    }
}
```

**Lưu ý:** `interface{}` (hoặc `any` từ Go 1.18) chấp nhận mọi kiểu, nhưng lạm dụng sẽ mất an toàn kiểu. Ưu tiên interface cụ thể. [Suy luận]

## 9. Xử lý lỗi

Trong Go, lỗi là **giá trị** kiểu `error`, không phải ngoại lệ bị ném ngầm:

```go
type error interface {
    Error() string
}
```

**Mẫu chuẩn:**

```go
f, err := os.Open("cau-hinh.json")
if err != nil {
    return fmt.Errorf("mở cấu hình: %w", err)
}
```

**Tài liệu chính thức nhấn mạnh:** bỏ qua lỗi là "thực hành rất tệ". [Đã kiểm chứng: Effective Go]

**Gói lỗi (wrapping)** bằng `%w` để giữ chuỗi nguyên nhân:

```go
if errors.Is(err, os.ErrNotExist) {
    // xử lý trường hợp không có tệp
}

var pe *fs.PathError
if errors.As(err, &pe) {
    fmt.Println("đường dẫn lỗi:", pe.Path)
}
```

- `errors.Is` kiểm tra lỗi có phải một giá trị cụ thể trong chuỗi gói không.
- `errors.As` lấy kiểu lỗi cụ thể từ chuỗi gói.

**Thông báo lỗi:** viết chữ thường, không kết thúc bằng dấu chấm, và cho biết nguồn gốc. Ví dụ: `"open config.json: no such file or directory"`. [Đã kiểm chứng: Effective Go nêu nguyên tắc thông báo lỗi nêu nguồn gốc]

## 10. Panic và recover

- **`panic`** dừng chương trình khi gặp lỗi không thể phục hồi, tương tự ngoại lệ không bắt được.
- **`recover`** chỉ hoạt động trong hàm `defer`, để bắt panic và tiếp tục.

```go
func anToan(f func()) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("đã bắt panic:", r)
        }
    }()
    f()
}
```

**Quy tắc thực hành:** thư viện nên tránh panic, trừ khi lỗi là lập trình sai nghiêm trọng. Trong ứng dụng, `recover` ở biên, ví dụ trong máy chủ HTTP, để một yêu cầu lỗi không làm sập toàn bộ dịch vụ. [Đã kiểm chứng: Effective Go nói về panic và recover, khuyên hạn chế ở thư viện]

## 11. Generics (từ Go 1.18)

Generics cho phép viết hàm và kiểu dùng cho nhiều kiểu, với kiểm tra kiểu lúc biên dịch:

```go
func Max[T int | float64](a, b T) T {
    if a > b {
        return a
    }
    return b
}

fmt.Println(Max(3, 7))        // T = int
fmt.Println(Max(2.5, 1.0))    // T = float64
```

**Khi nào dùng:** cấu trúc dữ liệu tổng quát (danh sách, tập hợp), hàm tiện ích trên nhiều kiểu. **Khi nào không:** nếu một interface đơn giản làm được việc, interface thường rõ ràng hơn. [Suy luận]

---

# PHẦN C. ĐỒNG THỜI

Đây là điểm mạnh nổi tiếng nhất của Go. Hiểu phần này giúp hiểu vì sao Go được chọn cho máy chủ.

## 12. Goroutine

**Goroutine** là một hàm chạy đồng thời với các hàm khác trong cùng không gian địa chỉ. [Đã kiểm chứng: định nghĩa trong Effective Go]

```go
go xuLy(yeuCau)    // chạy xuLy trong goroutine mới, không chờ
```

**Đặc điểm:**
- **Nhẹ:** ngăn xếp ban đầu nhỏ, lớn lên khi cần. Có thể tạo hàng trăm nghìn goroutine. [Trí nhớ]
- **Ghép kênh (multiplexed):** runtime của Go ánh xạ nhiều goroutine lên ít luồng hệ điều hành. [Đã kiểm chứng: Effective Go mô tả goroutine được ghép lên luồng hệ điều hành]
- **Không có ID công khai**, không có cơ chế "chờ" trực tiếp. Dùng channel hoặc `sync.WaitGroup` để phối hợp. [Trí nhớ]

**Ví dụ: chạy nhiều tác vụ và chờ hết:**

```go
var wg sync.WaitGroup
for _, url := range danhSachURL {
    wg.Add(1)
    go func(u string) {
        defer wg.Done()
        taiVe(u)
    }(url)           // truyền url vào tham số để tránh biến chia sẻ
}
wg.Wait()
```

**Lưu ý:** trước Go 1.22, biến vòng lặp dùng chung giữa các lần lặp, gây lỗi nếu goroutine đọc sau. Go 1.22 đổi để mỗi lần lặp có biến riêng. Mã cũ vẫn nên truyền tham số tường minh. [Đã kiểm chứng: Effective Go và mã phát hành]

## 13. Channel và select

**Channel** là đường truyền giữa các goroutine. Triết lý của Go: **"Đừng giao tiếp bằng cách chia sẻ bộ nhớ; hãy chia sẻ bộ nhớ bằng cách giao tiếp."** [Đã kiểm chứng: câu này trích từ Effective Go]

```go
ch := make(chan int)        // không đệm: gửi chờ người nhận
go func() {
    ch <- 42                // gửi
}()
x := <-ch                   // nhận, chờ đến khi có dữ liệu
fmt.Println(x)
```

**Channel đệm:** gửi không chờ khi còn chỗ.

```go
buf := make(chan string, 10)
```

**`select`** chờ nhiều channel, chọn cái nào sẵn sàng:

```go
select {
case msg := <-kenh1:
    fmt.Println("từ kênh 1:", msg)
case msg := <-kenh2:
    fmt.Println("từ kênh 2:", msg)
case <-time.After(2 * time.Second):
    fmt.Println("hết thời gian chờ")
}
```

**Đóng channel:** `close(ch)` báo không còn dữ liệu. Người nhận dùng `v, ok := <-ch` để biết channel đã đóng. Chỉ người gửi nên đóng channel. [Trí nhớ]

## 14. Các mẫu đồng thời thường gặp

**Mẫu 1: semaphore bằng channel đệm** giới hạn số tác vụ chạy cùng lúc. Lấy từ Effective Go:

```go
var sem = make(chan struct{}, 4)   // tối đa 4 tác vụ

func xuLy(r *YeuCau) {
    sem <- struct{}{}              // chờ có chỗ
    defer func() { <-sem }()       // trả chỗ
    thucHien(r)
}
```

**Mẫu 2: worker pool**, một tập goroutine lấy việc từ một channel:

```go
func workerPool(jobs <-chan int, results chan<- int, n int) {
    for i := 0; i < n; i++ {
        go func() {
            for j := range jobs {
                results <- j * j
            }
        }()
    }
}
```

**Mẫu 3: pipeline** nối nhiều bước bằng channel, mỗi bước là một goroutine.

**Mẫu 4: bảo vệ trạng thái bằng mutex** khi chia sẻ bộ nhớ là hợp lý:

```go
type Dem struct {
    mu sync.Mutex
    n  int
}

func (d *Dem) Tang() {
    d.mu.Lock()
    defer d.mu.Unlock()
    d.n++
}
```

Effective Go lưu ý rằng mutex có thể phù hợp hơn channel trong một số trường hợp, ví dụ bộ đếm tham chiếu. [Đã kiểm chứng]

**Cạm bẫy phổ biến:**
- **Rò rỉ goroutine:** goroutine chờ channel mãi không ai gửi. Luôn có đường thoát, thường qua `context`.
- **Deadlock:** mọi goroutine đều chờ nhau. Go báo `fatal error: all goroutines are asleep - deadlock!` khi xảy ra trong chương trình chính. [Trí nhớ]
- **Data race:** truy cập đồng thời không đồng bộ. Phát hiện bằng `-race` (mục 27).

## 15. Context và hủy tác vụ

Gói `context` mang thời hạn, tín hiệu hủy, và giá trị theo yêu cầu qua các lớp hàm:

```go
func laySoLieu(ctx context.Context, url string) error {
    ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return err
    }
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    return nil
}
```

**Quy ước:** `ctx` là tham số đầu tiên của hàm, và không lưu `context` trong struct. [Trí nhớ]

## 16. Đồng thời và song song khác nhau thế nào

Tài liệu chính thức phân biệt: **đồng thời (concurrency)** là cấu trúc chương trình thành các thành phần độc lập có thể chạy xen kẽ; **song song (parallelism)** là chạy đồng thời thật sự trên nhiều CPU. [Đã kiểm chứng: Effective Go]

**Ví dụ minh họa:** một quầy thu ngân với nhiều hàng đợi là đồng thời, còn có nhiều thu ngân làm việc cùng lúc là song song. Go giúp viết đồng thời dễ dàng, và runtime tự quyết định song song dựa trên `GOMAXPROCS`. [Suy luận]

---

# PHẦN D. TỔ CHỨC MÃ: PACKAGE VÀ MODULE

## 17. Package

- Một thư mục là một package. Mọi tệp `.go` trong cùng thư mục khai báo cùng tên package.
- Tên package ngắn, chữ thường, không gạch dưới: `http`, `json`, `user`.
- Import bằng đường dẫn đầy đủ: `import "net/http"`.

## 18. Module và go.mod

**Module** là tập các package có chung một đường dẫn gốc và phiên bản. Được định nghĩa trong `go.mod`:

```sh
mkdir baitap && cd baitap
go mod init example.com/baitap
```

Tệp `go.mod` mẫu:

```
module example.com/baitap

go 1.27

require github.com/google/uuid v1.6.0
```

**Các lệnh thường dùng:**

| Lệnh | Tác dụng |
|---|---|
| `go mod init <tên>` | Tạo go.mod |
| `go get <gói>@<phiên bản>` | Thêm hoặc nâng cấp phụ thuộc |
| `go mod tidy` | Thêm phụ thuộc thiếu, bỏ phụ thuộc thừa |
| `go mod verify` | Kiểm tra tính toàn vẹn của phụ thuộc |
| `go list -m all` | Liệt kê phụ thuộc |

**`go.sum`** ghi hash để đảm bảo phụ thuộc không bị đổi lặng lẽ. Nên commit cả `go.mod` và `go.sum`. [Trí nhớ]

## 19. Hiển thị công khai và nội bộ

- Tên **viết hoa chữ cái đầu** là công khai (exported): `TinhTong`, `SinhVien`.
- Tên **viết thường** là nội bộ trong package: `tinhNoiBo`.

Đây là cách Go thay thế các từ khóa `public`, `private`. [Trí nhớ]

**Thư mục `internal/`:** package trong `internal/` chỉ import được từ module cha. Dùng để giấu mã nội bộ khỏi người dùng module. [Trí nhớ]

---

# PHẦN E. CÔNG CỤ CHUẨN CỦA GO

## 20. Bảng công cụ dòng lệnh

| Lệnh | Chức năng | Ghi chú |
|---|---|---|
| `go run` | Biên dịch và chạy | Tiện khi phát triển |
| `go build` | Biên dịch thành tệp nhị phân | `-o` để đặt tên tệp |
| `go install` | Biên dịch và cài vào `$GOBIN` | Dùng cho công cụ dòng lệnh |
| `go test` | Chạy kiểm thử | Mục 22 |
| `go vet` | Kiểm tra lỗi đáng ngờ | Mục 21 |
| `go fmt` | Định dạng mã | Chạy `gofmt` trên package |
| `go doc` | Xem tài liệu | Ví dụ: `go doc fmt.Println` |
| `go mod` | Quản lý module | Mục 18 |
| `go generate` | Chạy công cụ sinh mã | Mục 30 |
| `go env` | Xem biến môi trường của Go | |
| `go version` | Xem phiên bản | |
| `go tool pprof` | Phân tích hiệu năng | Mục 33 |

## 21. Định dạng và kiểm tra tĩnh

**`gofmt`:** định dạng mã theo một chuẩn duy nhất. Tài liệu chính thức khuyên để công cụ quyết định phong cách, không tranh luận. Go dùng tab để thụt lề, và mọi mã trong thư viện chuẩn đều được định dạng bằng `gofmt`. [Đã kiểm chứng]

**`go vet`:** phát hiện các mẫu mã đáng ngờ: định dạng `Printf` sai, khóa sao chép, hoặc gán không dùng. [Trí nhớ]

**Công cụ bên thứ ba thường dùng** (không thuộc bộ chuẩn):
- **staticcheck:** phát hiện nhiều lỗi và mã không hiệu quả hơn `go vet`. [Trí nhớ]
- **golangci-lint:** chạy nhiều bộ kiểm tra cùng lúc, cấu hình qua tệp. [Trí nhớ]

**Gợi ý:** chạy `gofmt` và `go vet` trước mọi commit. Đưa vào CI (mục 31).

---

# PHẦN F. KIỂM THỬ

Kiểm thử là thế mạnh tích hợp sẵn của Go: không cần thư viện bên ngoài để viết và chạy test.

## 22. Kiểm thử đơn vị

**Quy ước:**
- Tệp test kết thúc bằng `_test.go`, nằm cùng thư mục với mã.
- Hàm test bắt đầu bằng `Test`, nhận `*testing.T`.

```go
// cong.go
package mathx

func Cong(a, b int) int {
    return a + b
}
```

```go
// cong_test.go
package mathx

import "testing"

func TestCong(t *testing.T) {
    got := Cong(2, 3)
    want := 5
    if got != want {
        t.Errorf("Cong(2, 3) = %d; muốn %d", got, want)
    }
}
```

**Chạy:**

```sh
go test ./...            # toàn bộ dự án
go test -v ./mathx       # chi tiết từng test
go test -run TestCong    # chỉ test có tên khớp
```

**Các phương thức báo lỗi:**
- `t.Error`, `t.Errorf`: ghi lỗi, tiếp tục test.
- `t.Fatal`, `t.Fatalf`: ghi lỗi, dừng test ngay. Dùng khi tiếp tục không có ý nghĩa.
- `t.Skip`: bỏ qua test.
- `t.Helper()`: đánh dấu hàm trợ giúp để báo lỗi trỏ đúng dòng gọi. [Trí nhớ]

## 23. Table-driven test và subtest

Đây là mẫu chuẩn của Go, gọn và dễ mở rộng:

```go
func TestCong(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {"cả hai dương", 2, 3, 5},
        {"có số âm", -1, 4, 3},
        {"cả hai âm", -2, -3, -5},
        {"có số 0", 0, 7, 7},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Cong(tt.a, tt.b); got != tt.want {
                t.Errorf("Cong(%d, %d) = %d; muốn %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

**Lợi ích:**
- Thêm trường hợp chỉ cần thêm một dòng.
- `t.Run` tạo **subtest** có tên riêng, chạy được riêng bằng `go test -run 'TestCong/có_số_âm'`.
- Mỗi trường hợp độc lập, lỗi một cái không làm dừng các cái khác.

**Song song:** `t.Parallel()` cho phép test chạy đồng thời. Cẩn thận khi biến dùng chung. [Trí nhớ]

**Trợ giúp TestMain:** hàm `TestMain(m *testing.M)` chạy thiết lập và dọn dẹp cho toàn bộ package. [Trí nhớ]

**Dữ liệu test:** thư mục `testdata/` được Go bỏ qua khi biên dịch, và là nơi lý tưởng để đặt tệp dữ liệu mẫu. [Trí nhớ]

## 24. Kiểm thử HTTP và kiểm thử với phụ thuộc

**Kiểm thử handler HTTP bằng `httptest`:** không cần khởi động máy chủ thật.

```go
func TestChao(t *testing.T) {
    req := httptest.NewRequest("GET", "/chao?ten=An", nil)
    w := httptest.NewRecorder()

    Chao(w, req)

    if w.Code != http.StatusOK {
        t.Fatalf("mã trả về = %d", w.Code)
    }
    if body := w.Body.String(); body != "Xin chào, An" {
        t.Errorf("nội dung = %q", body)
    }
}
```

**Kiểm thử với phụ thuộc (dependency):** thay vì gọi cơ sở dữ liệu thật, định nghĩa interface và truyền bản giả (fake):

```go
type KhoDuLieu interface {
    LaySinhVien(mssv string) (SinhVien, error)
}

// Bản giả cho test
type khoGia struct{ data map[string]SinhVien }

func (k khoGia) LaySinhVien(mssv string) (SinhVien, error) {
    sv, ok := k.data[mssv]
    if !ok {
        return SinhVien{}, errors.New("không tìm thấy")
    }
    return sv, nil
}
```

**Vì sao cách này tốt:** interface nhỏ giúp thay thế dễ dàng. Go không cần thư viện mock nặng nề, mặc dù có các công cụ sinh mock cho trường hợp lớn. [Suy luận]

**Kiểm thử tích hợp:** dùng build tag hoặc biến môi trường để phân tách test chạy chậm với cơ sở dữ liệu thật:

```go
//go:build integration

package repo
```

Chạy bằng `go test -tags=integration ./...`. [Trí nhớ]

## 25. Benchmark

Benchmark đo thời gian thực thi, hàm bắt đầu bằng `Benchmark`:

```go
func BenchmarkCong(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Cong(2, 3)
    }
}
```

```sh
go test -bench=. -benchmem
```

- `b.N` do Go tự điều chỉnh để đo đủ lâu cho kết quả ổn định.
- `-benchmem` hiển thị số lần cấp phát bộ nhớ, rất hữu ích để tối ưu.

**Lưu ý:** benchmark phải đo đúng thứ cần đo. Dùng `b.ResetTimer()` để loại trừ phần khởi tạo. Tránh để trình biên dịch loại bỏ kết quả không dùng, bằng cách gán vào biến toàn cục hoặc dùng kết quả. [Trí nhớ]

## 26. Fuzzing

**Fuzzing** sinh đầu vào ngẫu nhiên có định hướng, tìm trường hợp làm chương trình lỗi. Go hỗ trợ fuzzing trong bộ công cụ chuẩn từ **Go 1.18**. [Đã kiểm chứng]

```go
func FuzzCong(f *testing.F) {
    f.Add(2, 3)                      // hạt giống (seed)
    f.Fuzz(func(t *testing.T, a, b int) {
        got := Cong(a, b)
        if got != a+b {
            t.Errorf("Cong(%d, %d) = %d", a, b, got)
        }
    })
}
```

```sh
go test                      # chạy hạt giống như test thường
go test -fuzz=FuzzCong      # chạy fuzzer
```

**Chi tiết quan trọng từ tài liệu chính thức:** [Đã kiểm chứng]
- Đối số fuzz chỉ là: `string`, `[]byte`, kiểu số nguyên và không dấu, `float32`, `float64`, `bool`.
- Đầu vào lỗi được lưu vào `testdata/fuzz/FuzzCong/`, và trở thành hạt giống cho các lần chạy sau. Khi sửa lỗi, nó đóng vai trò test hồi quy.
- Mỗi lần thực thi có giới hạn **1 giây**. Hàm fuzz cần nhanh và xác định (không giữ trạng thái giữa các lần gọi).
- Chạy vô hạn cho đến khi tìm thấy lỗi, hoặc dùng `-fuzztime` để giới hạn.

## 27. Phát hiện data race

Data race xảy ra khi hai goroutine truy cập cùng biến, ít nhất một là ghi, mà không đồng bộ. Đây là lỗi khó tái hiện.

```sh
go test -race ./...
go run -race main.go
```

Bộ phát hiện race (race detector) báo vị trí cụ thể, với stack trace của hai truy cập. Chạy `-race` trong CI là thực hành chuẩn. Lưu ý: chương trình chạy chậm hơn và tốn bộ nhớ hơn khi bật. [Trí nhớ]

## 28. Độ bao phủ (coverage)

```sh
go test -cover ./...
go test -coverprofile=cover.out ./...
go tool cover -html=cover.out     # mở báo cáo trực quan
```

**Cách hiểu:** độ bao phủ cho biết bao nhiêu dòng được chạy trong test. Độ bao phủ 100% **không đảm bảo** chương trình đúng, chỉ đảm bảo các dòng đã được chạy. Nên kết hợp với test biên và fuzzing. [Suy luận]

---

# PHẦN G. TỰ ĐỘNG HÓA

## 29. Makefile và tác vụ lặp lại

Go không có hệ thống build riêng như Maven, nhưng thường đi kèm công cụ tác vụ đơn giản, ví dụ `make`:

```makefile
.PHONY: test lint build

test:
	go test -race -cover ./...

lint:
	go vet ./...
	gofmt -l .

build:
	go build -o bin/app ./cmd/app
```

```sh
make test
```

**Lưu ý:** Makefile trên Windows cần Make hoặc môi trường tương đương (WSL, Git Bash). Nhiều dự án dùng Taskfile (YAML) hoặc script PowerShell thay thế. [Suy luận]

## 30. go generate

`go generate` chạy công cụ sinh mã được khai báo bằng chú thích:

```go
//go:generate stringer -type=TrangThai
type TrangThai int
```

```sh
go generate ./...
```

Dùng cho: sinh mã từ schema, sinh mock, sinh chuỗi hằng số, sinh mã protobuf. [Trí nhớ]

## 31. Tích hợp liên tục (CI)

Một pipeline CI điển hình cho dự án Go:

```
1. Kiểm tra định dạng:  gofmt -l . (không được có tệp in ra)
2. Kiểm tra tĩnh:       go vet ./...
3. Kiểm tra phụ thuộc:  go mod tidy && git diff --exit-code
4. Kiểm thử:            go test -race -cover ./...
5. Fuzzing ngắn:        go test -fuzz=. -fuzztime=30s ./...   (tùy chọn)
6. Biên dịch:           go build ./...
7. Đóng gói:            tạo tệp nhị phân cho từng nền tảng
```

**Ví dụ GitHub Actions** (minh họa, cần điều chỉnh phiên bản):

```yaml
name: ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.27'
      - run: go vet ./...
      - run: go test -race -cover ./...
```

[Suy luận; tên action có thể thay đổi theo thời điểm, nên kiểm tra tài liệu chính thức của GitHub Actions]

**Nguyên tắc:** CI nên chạy đúng các lệnh bạn chạy trên máy cá nhân. Nếu CI và máy khác nhau, lỗi sẽ khó tìm.

## 32. Đóng gói và phát hành

**Biên dịch chéo (cross-compilation):** Go có thể biên dịch cho nền tảng khác mà không cần máy đó:

```sh
GOOS=linux GOARCH=amd64 go build -o app-linux ./cmd/app
GOOS=windows GOARCH=amd64 go build -o app.exe ./cmd/app
```

**Phiên bản:** gắn thông tin phiên bản lúc biên dịch:

```sh
go build -ldflags "-X main.version=1.2.0" -o app ./cmd/app
```

**Container:** tệp nhị phân tĩnh rất hợp với ảnh container tối giản. Có thể dùng ảnh rỗng (scratch) hoặc ảnh distroless. [Trí nhớ]

## 33. Đo và tối ưu hiệu năng

**Quy trình:** đo trước, tối ưu sau. Không tối ưu theo cảm tính.

1. **Benchmark** để có số liệu gốc (mục 25).
2. **Profiling** để tìm nút thắt:

```go
import _ "net/http/pprof"    // thêm vào dịch vụ HTTP để mở endpoint pprof
```

```sh
go test -bench=. -cpuprofile cpu.out
go tool pprof cpu.out
```

3. **Sửa một chỗ**, đo lại, so sánh.
4. **Ghi lại kết quả**.

**Kiểm tra cấp phát bộ nhớ:** `-benchmem` và `go test -memprofile`. [Trí nhớ]

---

# PHẦN H. VAI TRÒ CỦA GO TRONG THỰC TẾ

## 34. Go được dùng ở đâu

| Lĩnh vực | Ví dụ | Lý do Go phù hợp |
|---|---|---|
| Máy chủ web và API | Dịch vụ backend, gateway | Đồng thời tốt, thư viện HTTP mạnh |
| Hạ tầng đám mây | Các công cụ container, orchestration | Tệp nhị phân tĩnh, triển khai đơn giản |
| Công cụ dòng lệnh | CLI nội bộ, công cụ DevOps | Khởi động nhanh, một tệp duy nhất |
| Hệ thống dữ liệu | Dịch vụ xử lý luồng, cơ sở dữ liệu | Hiệu năng và kiểm soát bộ nhớ |
| Mạng | Proxy, dịch vụ truyền tải | Goroutine và channel phù hợp với I/O |

**Một số dự án nổi tiếng được viết bằng Go:** Docker, Kubernetes, Terraform. [Trí nhớ, nên kiểm chứng với nguồn chính thức của từng dự án]

## 35. Go so với một số ngôn ngữ khác

| Tiêu chí | Go | Java | Python | Rust |
|---|---|---|---|---|
| Biên dịch | Nhanh, ra mã máy | Ra bytecode, chạy trên JVM | Thông dịch, có bytecode | Chậm hơn, ra mã máy |
| Đồng thời | Goroutine, channel | Thread, executor, virtual thread (JDK mới) | asyncio, GIL giới hạn song song | Async, ownership |
| Quản lý bộ nhớ | GC | GC | GC | Không GC, ownership |
| Độ phức tạp ngôn ngữ | Thấp | Trung bình đến cao | Thấp | Cao |
| Hiệu năng | Cao | Cao | Thấp hơn | Rất cao |
| Tệp triển khai | Một tệp nhị phân | JAR, cần JVM | Cần interpreter và thư viện | Tệp nhị phân |

[Suy luận và kiến thức nền; so sánh mang tính khái quát, không phải benchmark]

## 36. Các điểm cần cẩn trọng

- **Xử lý lỗi dài dòng:** kiểu `if err != nil` lặp lại nhiều. Đây là đánh đổi có chủ đích để lỗi rõ ràng.
- **Generics mới:** từ Go 1.18, nhiều thư viện cũ chưa dùng. Cần kiểm tra phiên bản.
- **Không có kế thừa:** cần thói quen thiết kế mới, dựa trên interface và kết hợp.
- **GC:** tuy hiệu quả, nhưng có thể gây độ trễ trong hệ thống yêu cầu rất thấp. Cần đo.
- **Goroutine không tự dọn:** rò rỉ goroutine làm tăng bộ nhớ âm thầm. Cần kiểm tra bằng profiling.
- **Chưa có phiên bản cuối cùng cho mọi công cụ bên thứ ba:** kiểm tra mức hỗ trợ trước khi chọn thư viện.

---

# PHẦN I. LỘ TRÌNH HỌC

1. **Cú pháp và kiểu:** biến, hàm, slice, map, struct, lỗi. Làm vài chương trình nhỏ: đọc tệp, đếm từ.
2. **Interface và phương thức:** viết một interface nhỏ và hai kiểu thỏa mãn nó.
3. **Kiểm thử:** viết table-driven test cho mọi hàm. Dùng `go test -cover`.
4. **Đồng thời:** goroutine, channel, `sync.WaitGroup`, `select`. Viết worker pool đơn giản.
5. **Context và timeout:** thêm `context` vào hàm gọi mạng.
6. **Module và công cụ:** `go mod`, `gofmt`, `go vet`, `-race`.
7. **Fuzzing và benchmark:** thêm `FuzzXxx` cho hàm phân tích dữ liệu, đo với `-bench`.
8. **Tự động hóa:** Makefile hoặc script, và một pipeline CI.
9. **Dự án thật:** API HTTP nhỏ có kiểm thử, cấu hình, log, và đóng gói.

**Nguồn học chính thức:** Tour of Go, Effective Go, và tài liệu của gói `testing`. [Trí nhớ, cần xác nhận URL]

---

# PHẦN J. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Goroutine | Hàm chạy đồng thời, nhẹ, do runtime quản lý |
| Channel | Đường truyền dữ liệu giữa goroutine |
| Select | Chờ nhiều channel, chọn cái sẵn sàng |
| Interface | Tập phương thức mô tả hành vi, thỏa ngầm định |
| Embedding | Nhúng kiểu để tái sử dụng phương thức |
| Slice | Cửa sổ linh hoạt trên mảng |
| Map | Bảng băm khóa-giá trị |
| Defer | Lên lịch lời gọi chạy khi hàm kết thúc |
| Panic / Recover | Dừng khẩn cấp và bắt lại |
| Module | Tập package có go.mod, đơn vị phiên bản |
| Package | Thư mục chứa mã, đơn vị tổ chức |
| Exported | Tên viết hoa, công khai ngoài package |
| Table-driven test | Mẫu kiểm thử bằng bảng trường hợp |
| Subtest | Kiểm thử con có tên, chạy riêng được |
| Benchmark | Đo thời gian thực thi |
| Fuzzing | Sinh đầu vào ngẫu nhiên để tìm lỗi |
| Race detector | Công cụ phát hiện truy cập đồng thời không đồng bộ |
| Coverage | Tỷ lệ dòng mã được test chạy qua |
| Garbage collector (GC) | Thu gom bộ nhớ không còn dùng |
| Static binary | Tệp nhị phân tự chứa |
| Cross-compilation | Biên dịch cho nền tảng khác |
| CI | Tích hợp liên tục |
| pprof | Công cụ phân tích hiệu năng |
| Context | Mang thời hạn và tín hiệu hủy qua các lời gọi |
| Race condition | Kết quả phụ thuộc thứ tự thực thi không kiểm soát |
| Deadlock | Các goroutine chờ nhau mãi |
| Idiomatic | Viết theo phong cách chuẩn của cộng đồng Go |

---

# PHẦN K. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## K.1. Nguồn tham khảo

- **Go Release History** (phiên bản và chính sách hỗ trợ): https://go.dev/doc/devel/release [đã đọc]
  - Chính sách hỗ trợ: mỗi phiên bản lớn được hỗ trợ đến khi có hai phiên bản lớn mới hơn.
  - Bản vá mới nhất trong lần tra cứu: go1.27.3, phát hành 2026-10-19.
- **Effective Go**: https://go.dev/doc/effective_go [đã đọc phần lớn; lưu ý trang ghi nhận đã viết cho bản 2009 và không cập nhật đầy đủ, chưa bao gồm generics và module]
- **Fuzzing trong Go**: https://go.dev/doc/security/fuzz/ [đã đọc]
- **Tour of Go**: https://go.dev/tour/ [Trí nhớ, chưa mở trong lần này]
- **Tài liệu gói testing**: https://pkg.go.dev/testing [Trí nhớ]
- **Tài liệu go command** (build, test, mod, vet): https://go.dev/doc/cmd [Trí nhớ]

## K.2. Điểm cần xác minh

1. **Phiên bản mới nhất:** lần tra cứu cho thấy go1.27.3. Nên kiểm tra lại trước khi cài đặt, vì phiên bản thay đổi thường xuyên.
2. **Effective Go** không cập nhật đầy đủ. Các phần về generics, module, và công cụ hiện đại cần đối chiếu thêm với các trang khác.
3. **Các ví dụ đoạn mã** trong tài liệu được viết theo kiến thức Go hiện hành, nhưng **chưa được biên dịch và chạy** trong lần này. Nên thử trên máy trước khi dùng.
4. **Ví dụ GitHub Actions** (mục 31): tên action và phiên bản có thể đã thay đổi.
5. **Các dự án nổi tiếng** (Docker, Kubernetes, Terraform) dựa trên kiến thức nền, chưa đối chiếu nguồn chính thức.
6. **So sánh ngôn ngữ** (mục 35) là khái quát, không phải benchmark đo lường.
7. **Các chi tiết** như `t.Helper`, `TestMain`, build tag `integration`, `GOMAXPROCS`, và `errors.Join` đều là kiến thức nền, nên đối chiếu với `pkg.go.dev`.
8. **Go 1.22 loop variable** được nêu trong tài liệu, nhưng tôi chưa đọc trực tiếp phần ghi chú phát hành của phiên bản đó.

---

**Tóm tắt để tra nhanh:**
- **Go** = ngôn ngữ đơn giản, biên dịch ra một tệp nhị phân, có GC, và đồng thời là công dân hạng nhất qua goroutine và channel.
- **Cấu trúc:** package theo thư mục, module theo `go.mod`, hiển thị công khai bằng chữ hoa.
- **Lỗi** là giá trị, xử lý bằng `if err != nil`, gói bằng `%w`.
- **Interface** thỏa ngầm định, mô tả hành vi, giúp kiểm thử dễ.
- **Kiểm thử** tích hợp sẵn: `go test`, table-driven test, `httptest`, benchmark, fuzzing, `-race`, coverage.
- **Tự động hóa:** `gofmt`, `go vet`, `go mod tidy`, `go generate`, CI chạy đúng các lệnh trên.
