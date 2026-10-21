# Java: giải thích cú pháp từng dòng

> **Mục đích:** bổ sung cho tài liệu "Ngôn ngữ Java". Mỗi đoạn mã được chú thích từng dòng, và sau đoạn là bảng giải thích từ khóa, ký hiệu.
> **Cách đọc:** đọc đoạn mã trước, rồi đối chiếu chú thích. Các đoạn mã đều đã được viết theo cú pháp Java hiện đại, nhưng chưa được biên dịch và chạy trong lần này. Nên thử trên máy trước khi dùng.
> **Ngày soạn:** 2026-10-21.

---

## Mục lục

1. Khai báo biến và kiểu
2. Hằng số và kiểu tham chiếu
3. Lớp, constructor, và đóng gói
4. Interface, kế thừa, và abstract class
5. Generics
6. Collection
7. Lambda và Stream
8. Ngoại lệ và `try-with-resources`
9. `var`, text block, record, sealed, switch expression
10. `Optional`
11. Đồng thời: `ExecutorService`, `CompletableFuture`, virtual thread, `synchronized`
12. Maven: `pom.xml`
13. Gradle: `build.gradle.kts`
14. JUnit: test, assertion, vòng đời
15. Parameterized test
16. Mockito
17. GitHub Actions cho Java
18. Bảng ký hiệu chung

---

## 1. Khai báo biến và kiểu

```java
int so = 42;                    // khai báo biến kiểu int, gán 42
long lon = 10_000_000_000L;     // L ở cuối: kiểu long; _ chỉ để dễ đọc
double thuc = 3.14;             // số thực dấu phẩy động 64 bit
boolean dung = true;            // kiểu luận lý
char ky = 'A';                  // một ký tự, dùng nháy đơn
String ten = "An";              // String là kiểu tham chiếu, dùng nháy kép
```

| Cú pháp | Ý nghĩa |
|---|---|
| `int so = 42;` | `kiểu tên = giá_trị;`. Java bắt buộc khai báo kiểu, dấu `;` kết thúc câu lệnh |
| `10_000_000_000L` | Hậu tố `L` báo đây là số `long`. Số quá lớn cho `int` phải có `L` |
| `'A'` và `"A"` | `'A'` là `char` (một ký tự), `"A"` là `String` (chuỗi) |
| `_` trong số | Dấu gạch dưới bị bỏ qua, chỉ giúp đọc số lớn |

```java
String a = null;    // biến tham chiếu có thể bằng null (không có object)
a.length();         // lỗi NullPointerException vì a không trỏ tới object nào
```

## 2. Hằng số và kiểu tham chiếu

```java
final int HANG = 1;       // final: không gán lại được sau khi khởi tạo
final String TEN_APP = "congcu";
```

| Từ khóa | Ý nghĩa |
|---|---|
| `final` | Biến chỉ gán một lần. Với object, không đổi tham chiếu nhưng nội dung object vẫn có thể đổi |
| `static final` | Hằng cấp lớp, thường viết hoa, ví dụ `static final int MAX = 100;` |

## 3. Lớp, constructor, và đóng gói

```java
public class SinhVien {                          // khai báo lớp công khai, tên tệp phải là SinhVien.java
    private final String mssv;                   // trường riêng tư, không gán lại được
    private String ten;                          // trường riêng, gán được
    private double diem;                         // trường riêng, mặc định 0.0

    public SinhVien(String mssv, String ten) {   // constructor: không có kiểu trả về, tên trùng tên lớp
        this.mssv = mssv;                        // this.mssv là trường; mssv (không this) là tham số
        this.ten = ten;
    }

    public String getTen() {                     // phương thức công khai, trả về String
        return ten;                              // trả về giá trị của trường ten
    }

    public void congDiem(double d) {             // void: không trả về gì
        if (d < 0) {                             // điều kiện, đặt trong ngoặc tròn
            throw new IllegalArgumentException("điểm âm");   // ném ngoại lệ
        }
        diem += d;                               // tương đương diem = diem + d
    }
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `public` | Ai cũng truy cập được |
| `private` | Chỉ trong lớp này |
| (không ghi gì) | Package-private: truy cập được trong cùng package |
| `protected` | Lớp này, lớp con và cùng package |
| `this` | Tham chiếu đến object hiện tại. Dùng để phân biệt trường với tham số trùng tên |
| `new SinhVien("SV01", "An")` | Tạo object mới, gọi constructor |
| `void` | Phương thức không trả về giá trị |
| `throw new ...` | Ném ngoại lệ, dừng luồng bình thường |

```java
SinhVien sv = new SinhVien("SV01", "An");   // kiểu biến, tên biến = new Lớp(...)
sv.congDiem(1.5);                            // gọi phương thức: object.tênPhươngThức(...)
```

## 4. Interface, kế thừa, và abstract class

```java
interface Ve {                                   // khai báo interface
    void ve();                                   // phương thức trừu tượng: không có thân hàm
    default void veMacDinh() {                   // default: có thân hàm, lớp con dùng được mà không cần viết lại
        ve();
    }
}

abstract class Hinh implements Ve {              // abstract: không tạo được object trực tiếp; implements: triển khai interface
    abstract double dienTich();                  // phương thức trừu tượng, lớp con phải cài đặt
}

class HinhTron extends Hinh {                    // extends: kế thừa lớp Hinh (chỉ một lớp cha)
    private final double r;
    HinhTron(double r) { this.r = r; }           // constructor một dòng
    double dienTich() { return Math.PI * r * r; }  // cài đặt phương thức trừu tượng
    public void ve() { System.out.println("tròn"); }  // cài đặt phương thức từ interface, phải là public
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `interface` | Khai báo hợp đồng hành vi |
| `implements` | Lớp cài đặt một hoặc nhiều interface |
| `extends` | Lớp kế thừa đúng một lớp cha |
| `abstract` | Lớp hoặc phương thức chưa đầy đủ, không tạo trực tiếp được |
| `default` | Phương thức interface có thân hàm mặc định |
| `@Override` | Chú thích tùy chọn, báo phương thức ghi đè. Nên thêm để trình biên dịch kiểm tra |

## 5. Generics

```java
public static <T extends Comparable<T>> T lonNhat(List<T> ds) {
    // <T extends Comparable<T>>: T là kiểu tham số, phải so sánh được với chính nó
    // List<T>: danh sách các phần tử kiểu T
    // T (sau tên hàm): kiểu trả về
    T m = ds.get(0);                    // lấy phần tử đầu tiên
    for (T x : ds) {                    // vòng for-each: duyệt từng phần tử
        if (x.compareTo(m) > 0) {       // compareTo > 0: x lớn hơn m
            m = x;
        }
    }
    return m;
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `<T>` | Khai báo kiểu tham số |
| `<T extends X>` | Ràng buộc: T phải là X hoặc lớp con |
| `List<String>` | Danh sách chỉ chứa String |
| `? extends T` | Wildcard chỉ đọc: danh sách của kiểu con của T |
| `? super T` | Wildcard chỉ ghi: danh sách của kiểu cha của T |
| `for (T x : ds)` | Vòng for-each, duyệt phần tử của tập hợp |

**Lưu ý:** Java xóa thông tin generic lúc biên dịch (type erasure), nên không viết được `new T()`.

## 6. Collection

```java
List<String> ds = new ArrayList<>(List.of("An", "Bình"));
// List<String>: kiểu biến là List chứa String
// new ArrayList<>(...): tạo ArrayList; <> để trống, Java suy luận kiểu (diamond)
// List.of(...): tạo danh sách bất biến chứa các giá trị

Map<String, Integer> tuoi = new HashMap<>();   // Map khóa String, giá trị Integer
tuoi.put("An", 25);                            // thêm hoặc ghi đè cặp khóa-giá trị
int t = tuoi.getOrDefault("Chi", 0);           // lấy giá trị; nếu không có thì trả 0
```

| Cú pháp | Ý nghĩa |
|---|---|
| `new ArrayList<>()` | Tạo đối tượng, `<>` là diamond operator |
| `List.of(a, b)` | Tạo list bất biến, không thêm hay xóa được |
| `Map.put(k, v)` | Thêm hoặc ghi đè |
| `Map.get(k)` | Lấy giá trị, trả `null` nếu không có |
| `getOrDefault(k, d)` | Lấy giá trị, trả `d` nếu không có |

**Lưu ý:** `Integer` là kiểu hộp của `int`, nên `Map<String, Integer>` không dùng được `int` trực tiếp, nhưng Java tự chuyển (autoboxing).

## 7. Lambda và Stream

```java
List<Integer> so = List.of(1, 2, 3, 4);

List<Integer> ketQua = so.stream()          // chuyển list thành luồng xử lý
    .filter(x -> x % 2 == 0)                // giữ phần tử chẵn; x -> ... là lambda
    .map(x -> x * 10)                       // biến đổi: nhân 10
    .toList();                              // kết thúc luồng, trả về List
// ketQua = [20, 40]
```

| Cú pháp | Ý nghĩa |
|---|---|
| `x -> x % 2 == 0` | Lambda: tham số `x`, mũi tên `->`, thân là biểu thức |
| `(a, b) -> a + b` | Lambda nhiều tham số, cần ngoặc tròn |
| `() -> 42` | Lambda không tham số |
| `.stream()` | Tạo luồng từ collection |
| `.filter(...)` | Giữ phần tử thỏa điều kiện |
| `.map(...)` | Biến đổi từng phần tử |
| `.toList()` | Kết thúc luồng, trả về list (Java 16 trở lên) |

**Lưu ý:** `filter` và `map` là lazy, chỉ chạy khi có thao tác kết thúc như `toList()`, `collect()`, `sum()`.

## 8. Ngoại lệ và `try-with-resources`

```java
try (BufferedReader r = Files.newBufferedReader(Path.of("du_lieu.txt"))) {
    // try (...): khai báo tài nguyên, Java tự đóng khi khối kết thúc
    String dong = r.readLine();          // đọc một dòng
    System.out.println(dong);
} catch (IOException e) {                // bắt ngoại lệ kiểu IOException
    throw new UncheckedIOException(e);   // chuyển thành ngoại lệ không kiểm tra
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `try { } catch (X e) { }` | Thử khối lệnh, nếu lỗi kiểu X thì xử lý ở catch |
| `finally { }` | Luôn chạy, dùng để dọn dẹp (ít cần khi có try-with-resources) |
| `throw new X("...")` | Ném ngoại lệ |
| `throws X` (trong chữ ký hàm) | Khai báo hàm có thể ném X, người gọi phải xử lý |
| `try (Tài_nguyên r = ...)` | Tài nguyên tự đóng khi khối kết thúc (cần implement `AutoCloseable`) |

**Ngoại lệ kiểm tra (checked):** `IOException` bắt buộc phải xử lý. **Không kiểm tra (unchecked):** `RuntimeException` và các lớp con, như `NullPointerException`, không bắt buộc.

## 9. `var`, text block, record, sealed, switch expression

```java
var ten = "An";                       // var: Java suy luận kiểu từ giá trị, ten là String

String json = """                     // text block: chuỗi nhiều dòng, bắt đầu bằng ba nháy kép
    {
      "ten": "An"
    }
    """;                              // kết thúc bằng ba nháy kép

record DiemSo(String mon, double so) {}   // record: lớp dữ liệu bất biến, Java tự sinh constructor, getter, equals, toString

sealed interface Hinh permits HinhTron, HinhVuong {}   // sealed: chỉ HinhTron và HinhVuong được implement Hinh
record HinhTron(double r) implements Hinh {}
record HinhVuong(double c) implements Hinh {}

double dt = switch (hinh) {                    // switch biểu thức: trả về giá trị
    case HinhTron t -> Math.PI * t.r() * t.r();  // pattern: nếu là HinhTron thì đặt tên là t
    case HinhVuong v -> v.c() * v.c();         // nếu là HinhVuong thì đặt tên là v
};                                             // dấu ; vì đây là câu lệnh gán
```

| Cú pháp | Ý nghĩa |
|---|---|
| `var x = ...;` | Suy luận kiểu cục bộ, không dùng được cho tham số hàm hay trường |
| `"""..."""` | Text block, giữ nguyên xuống dòng và thụt lề |
| `record Ten(...) {}` | Khai báo record: trường đặt trong ngoặc, lấy bằng `t.r()` không phải `t.getR()` |
| `sealed ... permits ...` | Giới hạn lớp con được phép |
| `switch (x) { case K -> ...; }` | Switch biểu thức kiểu mới, không cần `break` |
| `case Kiểu t ->` | Pattern matching: kiểm tra kiểu và gán vào biến `t` cùng lúc |

**Lưu ý:** cú pháp pattern matching và sealed có ở các phiên bản Java mới. Kiểm tra phiên bản JDK đang dùng.

## 10. `Optional`

```java
Optional<SinhVien> sv = timSinhVien("SV01");     // Optional: có thể có hoặc không có SinhVien
String ten = sv
    .map(SinhVien::getTen)                       // nếu có thì lấy tên; SinhVien::getTen là tham chiếu phương thức
    .orElse("không rõ");                         // nếu không có thì dùng giá trị thay thế
```

| Cú pháp | Ý nghĩa |
|---|---|
| `Optional<T>` | Hộp chứa T hoặc rỗng |
| `Optional.of(x)` | Tạo hộp có giá trị, `x` không được null |
| `Optional.empty()` | Tạo hộp rỗng |
| `SinhVien::getTen` | Tham chiếu phương thức, tương đương `sv -> sv.getTen()` |
| `.orElse(x)` | Trả giá trị hoặc `x` nếu rỗng |
| `.isPresent()` | Kiểm tra có giá trị không |

## 11. Đồng thời: `ExecutorService`, `CompletableFuture`, virtual thread, `synchronized`

**ExecutorService:**

```java
ExecutorService pool = Executors.newFixedThreadPool(4);   // tạo pool 4 luồng
Future<Integer> f = pool.submit(() -> 6 * 7);             // gửi tác vụ, nhận Future (kết quả sẽ có sau)
System.out.println(f.get());                              // get() chờ đến khi có kết quả, in 42
pool.shutdown();                                          // đóng pool, không nhận việc mới
```

| Cú pháp | Ý nghĩa |
|---|---|
| `Executors.newFixedThreadPool(n)` | Pool cố định n luồng |
| `pool.submit(task)` | Gửi tác vụ có kết quả |
| `Future<T>` | Đại diện kết quả sẽ có sau |
| `f.get()` | Chờ và lấy kết quả (có thể ném ngoại lệ) |
| `shutdown()` | Đóng pool sau khi xong việc đang chạy |

**CompletableFuture:**

```java
CompletableFuture<String> a = CompletableFuture.supplyAsync(() -> "A");   // chạy nền, trả về "A"
CompletableFuture<String> b = CompletableFuture.supplyAsync(() -> "B");
String ket = a.thenCombine(b, (x, y) -> x + y).join();                    // ghép hai kết quả, join() chờ
// ket = "AB"
```

| Cú pháp | Ý nghĩa |
|---|---|
| `supplyAsync(() -> ...)` | Chạy tác vụ có trả về, không chặn |
| `thenCombine(b, (x, y) -> ...)` | Khi cả hai xong, ghép kết quả |
| `join()` | Chờ kết quả, không ném ngoại lệ kiểm tra |

**Virtual thread (Java 21 trở lên):**

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {   // tạo executor, mỗi tác vụ một virtual thread
    executor.submit(() -> xuLyYeuCau());                              // gửi tác vụ, không cần quản lý số luồng
}                                                                     // kết thúc khối: tự đóng và chờ
```

**synchronized:**

```java
class Dem {
    private int n = 0;
    public synchronized void tang() {   // synchronized: một luồng tại một thời điểm được chạy phương thức này
        n++;                            // n++ tăng 1
    }
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `synchronized` (phương thức) | Khóa theo object, chỉ một luồng vào tại một thời điểm |
| `n++` | Tăng giá trị, nhưng không an toàn đa luồng nếu không khóa |

## 12. Maven: `pom.xml`

```xml
<project>                                       <!-- gốc của tệp cấu hình Maven -->
  <modelVersion>4.0.0</modelVersion>            <!-- phiên bản mô hình POM, luôn là 4.0.0 -->
  <groupId>com.example</groupId>                <!-- định danh tổ chức, thường là tên miền ngược -->
  <artifactId>congcu</artifactId>               <!-- tên dự án -->
  <version>0.1.0</version>                      <!-- phiên bản dự án -->

  <properties>                                  <!-- biến dùng lại trong tệp -->
    <maven.compiler.release>21</maven.compiler.release>   <!-- biên dịch cho Java 21 -->
  </properties>

  <dependencies>                                <!-- danh sách phụ thuộc -->
    <dependency>
      <groupId>org.junit.jupiter</groupId>      <!-- nơi phát hành thư viện -->
      <artifactId>junit-jupiter</artifactId>    <!-- tên thư viện -->
      <version>6.1.3</version>                  <!-- phiên bản -->
      <scope>test</scope>                       <!-- chỉ dùng khi test, không đóng gói vào sản phẩm -->
    </dependency>
  </dependencies>
</project>
```

| Thẻ | Ý nghĩa |
|---|---|
| `<groupId>`, `<artifactId>`, `<version>` | Tọa độ duy nhất của thư viện (Maven coordinates) |
| `<scope>test</scope>` | Phạm vi: `compile` (mặc định), `test`, `provided`, `runtime` |
| `<properties>` | Biến cấu hình |

**Lệnh thường dùng:**

```sh
mvn clean verify          # xóa kết quả cũ, chạy build và test
mvn dependency:tree       # xem cây phụ thuộc
```

## 13. Gradle: `build.gradle.kts`

```kotlin
plugins {
    java                                       // bật plugin Java
}

java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(21))   // dùng JDK 21
    }
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:6.1.3")   // phụ thuộc chỉ dùng khi test
}

tasks.test {
    useJUnitPlatform()                         // dùng JUnit Platform để chạy test
}

tasks.register("xinChao") {                    // tạo tác vụ tùy biến tên xinChao
    doLast {                                   // chạy sau khi các bước khác xong
        println("xin chào")
    }
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `plugins { }` | Bật tính năng, như biên dịch Java |
| `testImplementation(...)` | Thêm phụ thuộc cho mã test |
| `tasks.test { }` | Cấu hình tác vụ `test` |
| `tasks.register("tên") { }` | Đăng ký tác vụ mới |
| `doLast { }` | Thân tác vụ, chạy cuối cùng |

**Lệnh:** `./gradlew build`, `./gradlew test`, `./gradlew xinChao`.

## 14. JUnit: test, assertion, vòng đời

```java
import org.junit.jupiter.api.*;                              // import các annotation: @Test, @BeforeEach...
import static org.junit.jupiter.api.Assertions.*;            // import tĩnh: gọi assertEquals không cần tiền tố

class CongTest {                                             // lớp test, thường đặt tên kết thúc bằng Test

    @BeforeEach                                              // chạy trước mỗi test
    void chuanBi() { /* thiết lập */ }

    @Test                                                    // đánh dấu đây là một test
    void congHaiSoDuong() {                                  // tên test mô tả hành vi
        assertEquals(5, CongCu.cong(2, 3));                  // kiểm tra: mong đợi 5, thực tế là cong(2, 3)
    }

    @Test
    void chiaChoKhong() {
        assertThrows(ArithmeticException.class,              // mong đợi ngoại lệ kiểu ArithmeticException
                     () -> CongCu.chia(1, 0));               // lambda chứa đoạn code sẽ ném lỗi
    }

    @AfterEach                                               // chạy sau mỗi test
    void donDep() { /* dọn dẹp */ }
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `import static ...` | Đưa phương thức tĩnh vào phạm vi, dùng trực tiếp `assertEquals` |
| `@Test` | Đánh dấu phương thức là test |
| `@BeforeEach` / `@AfterEach` | Chạy trước và sau mỗi test |
| `assertEquals(mong_doi, thuc_te)` | Kiểm tra bằng nhau. **Thứ tự: mong đợi trước** |
| `assertThrows(Lớp, () -> ...)` | Kiểm tra đoạn code có ném ngoại lệ đúng loại |
| `void` | Test không trả về giá trị |

**Lưu ý:** tên annotation và assertion theo kiến thức nền JUnit 5 và 6, trang tổng quan đã đọc không liệt kê chi tiết. Cần đối chiếu trang "Annotations" và "Assertions" trong tài liệu chính thức.

## 15. Parameterized test

```java
import org.junit.jupiter.params.ParameterizedTest;           // annotation cho test nhiều tham số
import org.junit.jupiter.params.provider.CsvSource;          // nguồn dữ liệu dạng CSV

@ParameterizedTest                                           // test chạy nhiều lần với tham số khác nhau
@CsvSource({                                                 // mỗi chuỗi là một bộ tham số, ngăn cách bằng dấu phẩy
    "2, 3, 5",                                               // a=2, b=3, kỳ vọng=5
    "-1, 4, 3",
    "0, 7, 7"
})
void cong(int a, int b, int kyVong) {                        // tham số khớp thứ tự cột trong CSV
    assertEquals(kyVong, CongCu.cong(a, b));
}
```

| Cú pháp | Ý nghĩa |
|---|---|
| `@ParameterizedTest` | Test có tham số |
| `@CsvSource({...})` | Mỗi chuỗi là một lần chạy, các giá trị ngăn cách bằng dấu phẩy |
| Tham số phương thức | Nhận giá trị theo thứ tự trong CSV |

## 16. Mockito

```java
import static org.mockito.Mockito.*;                        // import tĩnh các hàm mock, when, verify...

var repo = mock(KhoDuLieu.class);                           // tạo đối tượng giả của interface KhoDuLieu
when(repo.laySinhVien("SV01"))                              // khi gọi laySinhVien("SV01")...
    .thenReturn(new SinhVien("SV01", "An"));                // ...trả về object này

var dv = new DichVu(repo);                                  // tạo dịch vụ dùng repo giả
assertEquals("An", dv.tenSinhVien("SV01"));                 // kiểm tra kết quả
verify(repo, times(1)).laySinhVien("SV01");                 // kiểm tra repo được gọi đúng 1 lần
```

| Cú pháp | Ý nghĩa |
|---|---|
| `mock(Lớp.class)` | Tạo đối tượng giả. `Lớp.class` là đối tượng mô tả lớp |
| `when(...).thenReturn(...)` | Thiết lập giá trị trả về khi gọi |
| `verify(obj, times(n)).method()` | Kiểm tra số lần gọi |

**Lưu ý:** đây là mẫu cơ bản. Phiên bản Mockito và cú pháp cần kiểm tra theo tài liệu của dự án.

## 17. GitHub Actions cho Java

```yaml
name: ci                                      # tên workflow
on: [push, pull_request]                      # chạy khi push hoặc mở pull request
jobs:
  build:                                      # tên job
    runs-on: ubuntu-latest                    # máy chạy: Ubuntu mới nhất
    steps:                                    # các bước tuần tự
      - uses: actions/checkout@v4             # tải mã nguồn về máy chạy
      - uses: actions/setup-java@v4           # cài JDK
        with:                                 # tham số cho bước trên
          distribution: temurin               # nhà phân phối JDK
          java-version: 21                    # phiên bản JDK
          cache: maven                        # lưu cache Maven để nhanh hơn
      - run: mvn -B verify                    # chạy lệnh shell; -B là chế độ không tương tác
```

| Cú pháp | Ý nghĩa |
|---|---|
| `on:` | Điều kiện kích hoạt |
| `jobs:` | Danh sách công việc |
| `runs-on:` | Hệ điều hành của máy chạy |
| `uses:` | Dùng một action có sẵn |
| `with:` | Tham số truyền cho action |
| `run:` | Chạy một lệnh shell |

**Lưu ý:** phiên bản action (`@v4`) và JDK cần kiểm tra theo tài liệu hiện hành.

## 18. Bảng ký hiệu chung

| Ký hiệu | Ý nghĩa |
|---|---|
| `;` | Kết thúc câu lệnh |
| `{ }` | Khối lệnh, phạm vi |
| `( )` | Tham số, điều kiện, ép kiểu |
| `[ ]` | Mảng |
| `< >` | Generics |
| `//` và `/* */` | Chú thích |
| `/** */` | Chú thích tài liệu (javadoc) |
| `@` | Annotation |
| `::` | Tham chiếu phương thức |
| `->` | Lambda hoặc nhánh switch mới |
| `.` | Truy cập thành viên: `object.thuocTinh`, `Lop.tinhTinh` |
| `?` | Wildcard trong generics |
| `_` | Phân tách chữ số trong số, bỏ qua |
