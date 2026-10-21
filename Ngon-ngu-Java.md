# Ngôn ngữ Java: từ lý thuyết đến lập trình, tự động hóa và kiểm thử

> **Viết cho:** người học Java, lập trình viên từ Python, JavaScript, Go muốn hiểu hệ sinh thái JVM, và người làm backend doanh nghiệp cần nắm công cụ build, kiểm thử, và đóng gói.
> **Cách đọc:** Phần A và B giải thích JVM và các khái niệm cốt lõi, là nền cho mọi phần sau.
> **Phiên bản tham chiếu:**
> - Bản JDK GA mới nhất trên openjdk.org là **JDK 27**, phát hành 2026-09-15. Trang này không đề cập LTS.
> - JDK 25 là bản LTS gần nhất theo kiến thức nền (phát hành 2025-09). Cần kiểm tra lộ trình hỗ trợ của Oracle và các nhà phân phối khác.
> - JUnit: trang tài liệu hiển thị **JUnit 6.1.3** là phiên bản hiện tại.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này; `[Trí nhớ]` = kiến thức nền, cần kiểm tra lại; `[Suy luận]` = tôi suy ra hoặc minh họa.
>
> **Lưu ý về nguồn:** trang hỗ trợ của Oracle trả về 403 khi tra cứu, nên thông tin LTS và ngày hết hỗ trợ chưa được xác nhận trực tiếp.

---

## Mục lục

- **Phần A. Java là gì và vì sao được thiết kế như vậy**
  - 1. Lịch sử và triết lý
  - 2. Nền tảng: JDK, JRE, JVM
  - 3. JVM chạy mã như thế nào
  - 4. Bộ thu gom rác (GC)
- **Phần B. Các khái niệm cốt lõi**
  - 5. Biến, kiểu nguyên thủy, và kiểu tham chiếu
  - 6. Lớp, đối tượng, và đóng gói
  - 7. Kế thừa, interface, và abstract class
  - 8. Generics và kiểu an toàn
  - 9. Collection framework
  - 10. Lambda và Stream
  - 11. Ngoại lệ và `try-with-resources`
  - 12. Các tính năng hiện đại: record, sealed, pattern matching, `var`, text block
  - 13. `Optional` và tránh `null`
- **Phần C. Đồng thời**
  - 14. Thread và `ExecutorService`
  - 15. `CompletableFuture`
  - 16. Virtual thread (Java 21 trở lên)
  - 17. Đồng bộ hóa: `synchronized`, `Lock`, `java.util.concurrent`
- **Phần D. Tổ chức mã, build, và phụ thuộc**
  - 18. Package và module
  - 19. Maven và Gradle
- **Phần E. Công cụ chất lượng mã**
  - 20. Định dạng, lint, và phân tích tĩnh
- **Phần F. Kiểm thử**
  - 21. JUnit 6: test, assertion, và vòng đời
  - 22. Parameterized test
  - 23. Mock với Mockito
  - 24. Kiểm thử tích hợp và độ bao phủ
- **Phần G. Tự động hóa**
  - 25. Tác vụ build và script
  - 26. Tích hợp liên tục
  - 27. Đóng gói: JAR, jlink, jpackage, container
- **Phần H. Vai trò của Java trong thực tế**
  - 28. Java được dùng ở đâu
  - 29. So sánh với Go, Python, C#, Kotlin
  - 30. Các điểm cần cẩn trọng
- **Phần I. Lộ trình học**
- **Phần J. Thuật ngữ**
- **Phần K. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. JAVA LÀ GÌ VÀ VÌ SAO ĐƯỢC THIẾT KẾ NHƯ VẬY

## 1. Lịch sử và triết lý

Java được James Gosling và nhóm tại Sun Microsystems phát triển, công bố năm 1995 với khẩu hiệu "Write once, run anywhere": viết một lần, chạy trên nhiều nền tảng nhờ máy ảo. [Trí nhớ]

Oracle mua lại Sun năm 2010 và tiếp tục phát triển ngôn ngữ. OpenJDK là dự án mã nguồn mở chính thức. [Trí nhớ]

**Triết lý nổi bật:**
- **Kiểu tĩnh và kiểm tra lúc biên dịch**, giảm lỗi lúc chạy.
- **Chạy trên máy ảo (JVM)**, tách mã khỏi phần cứng.
- **Tương thích ngược** là ưu tiên lớn, nên hệ thống cũ vẫn chạy được qua nhiều phiên bản.
- **Hệ sinh thái rộng** cho doanh nghiệp: thư viện, framework, công cụ trưởng thành.

## 2. Nền tảng: JDK, JRE, JVM

| Thành phần | Vai trò |
|---|---|
| **JVM** (Java Virtual Machine) | Máy ảo chạy bytecode, quản lý bộ nhớ, JIT, GC |
| **JRE** (Runtime Environment) | JVM cộng thư viện chuẩn, dùng để chạy |
| **JDK** (Development Kit) | JRE cộng công cụ phát triển: `javac`, `jar`, `jshell`, `javadoc` |

Từ Java 9, JRE không còn được phân phối riêng rộng rãi; thay vào đó dùng `jlink` để tạo runtime rút gọn cho ứng dụng. [Trí nhớ]

**Chạy nhanh:**

```sh
java --version
javac Main.java
java Main
```

## 3. JVM chạy mã như thế nào

```
Mã nguồn .java
    │  javac (biên dịch)
    ▼
Bytecode .class (hoặc .jar)
    │  JVM nạp và xác minh bytecode
    ▼
Interpreter + JIT compiler → mã máy
    │
    ▼
Chạy
```

- `javac` biên dịch mã nguồn thành **bytecode** độc lập với hệ điều hành. [Trí nhớ]
- JVM **xác minh** bytecode trước khi chạy, giúp bảo vệ khỏi mã không hợp lệ. [Trí nhớ]
- **JIT (Just-In-Time)** biên dịch mã hay dùng thành mã máy lúc chạy, tối ưu dựa trên thống kê thực tế. Đây là lý do Java có hiệu năng cao sau khi khởi động. [Trí nhớ]

**Đánh đổi:** thời gian khởi động và bộ nhớ nền cao hơn Go hoặc Rust, nhưng throughput dài hạn tốt. [Suy luận]

## 4. Bộ thu gom rác (GC)

JVM quản lý bộ nhớ bằng GC. Các bộ GC chính: [Trí nhớ]

| GC | Đặc điểm | Ghi chú |
|---|---|---|
| **G1** | Mặc định từ Java 9 | Cân bằng thời gian dừng và thông lượng |
| **ZGC** | Thời gian dừng rất thấp | Phù hợp độ trễ nhạy cảm |
| **Shenandoah** | Thời gian dừng thấp, đồng thời | Có trong OpenJDK |
| **Parallel** | Thông lượng tối đa | Thời gian dừng dài hơn |

**Tùy chọn JVM thường dùng:**

```sh
java -Xms512m -Xmx2g -XX:+UseG1GC -jar app.jar
java -XX:+UseZGC -jar app.jar
```

[Trí nhớ; tên tùy chọn có thể thay đổi, kiểm tra theo phiên bản]

**Lưu ý:** `-Xmx` giới hạn heap tối đa. Quá nhỏ gây `OutOfMemoryError`; quá lớn làm GC chậm và tốn RAM. Đo trước khi đặt. [Suy luận]

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 5. Biến, kiểu nguyên thủy, và kiểu tham chiếu

```java
int so = 42;
long lon = 10_000_000_000L;
double thuc = 3.14;
boolean dung = true;
char ky = 'A';

String ten = "An";        // kiểu tham chiếu
final int hang = 1;       // không gán lại
```

**Kiểu nguyên thủy:** `byte`, `short`, `int`, `long`, `float`, `double`, `boolean`, `char`. Không phải object, lưu trực tiếp giá trị. [Trí nhớ]

**Kiểu tham chiếu:** object, mảng, chuỗi. Biến giữ địa chỉ, không giữ nội dung.

**Tự động đóng/mở hộp (autoboxing):** `int` tự chuyển sang `Integer` khi cần. Cẩn thận so sánh `Integer` bằng `==`, nên dùng `.equals`. [Trí nhớ]

```java
Integer a = 127, b = 127;
a == b;        // true (cache), nhưng với 128 có thể false
a.equals(b);   // luôn đúng cách
```

## 6. Lớp, đối tượng, và đóng gói

```java
public class SinhVien {
    private final String mssv;
    private String ten;
    private double diem;

    public SinhVien(String mssv, String ten) {
        this.mssv = mssv;
        this.ten = ten;
    }

    public String getTen() { return ten; }

    public void congDiem(double d) {
        if (d < 0) throw new IllegalArgumentException("điểm âm");
        diem += d;
    }
}
```

**Phạm vi truy cập:** `private`, package-private (mặc định), `protected`, `public`. [Trí nhớ]

**Quy ước getter/setter** là phong cách của JavaBean. Hiện nay với dữ liệu bất biến, dùng `record` (mục 12). [Trí nhớ]

## 7. Kế thừa, interface, và abstract class

```java
interface Ve {
    void ve();
    default void veMacDinh() { ve(); }     // phương thức mặc định (Java 8)
}

abstract class Hinh implements Ve {
    abstract double dienTich();
}

class HinhTron extends Hinh {
    private final double r;
    HinhTron(double r) { this.r = r; }
    double dienTich() { return Math.PI * r * r; }
    public void ve() { System.out.println("tròn"); }
}
```

- Java chỉ có **kế thừa đơn lớp**, nhưng một lớp có thể triển khai **nhiều interface**. [Trí nhớ]
- `interface` có thể có phương thức mặc định và phương thức tĩnh. [Trí nhớ]
- Ưu tiên **kết hợp** và interface hơn kế thừa sâu. [Suy luận]

## 8. Generics và kiểu an toàn

```java
public static <T extends Comparable<T>> T lonNhat(List<T> ds) {
    T m = ds.get(0);
    for (T x : ds) {
        if (x.compareTo(m) > 0) m = x;
    }
    return m;
}
```

**Type erasure:** generics bị xóa khi biên dịch, không có thông tin kiểu lúc chạy. Hệ quả: không thể `new T()` hay kiểm tra `instanceof List<String>`. [Trí nhớ]

**Wildcard:** `? extends T` (chỉ đọc) và `? super T` (chỉ ghi), theo nguyên tắc PECS. [Trí nhớ]

## 9. Collection framework

| Interface | Triển khai phổ biến | Đặc điểm |
|---|---|---|
| `List` | `ArrayList`, `LinkedList` | Có thứ tự, trùng lặp |
| `Set` | `HashSet`, `TreeSet`, `LinkedHashSet` | Không trùng lặp |
| `Map` | `HashMap`, `TreeMap`, `LinkedHashMap` | Khóa-giá trị |
| `Queue`, `Deque` | `ArrayDeque`, `PriorityQueue` | Hàng đợi |

```java
List<String> ds = new ArrayList<>(List.of("An", "Bình"));
Map<String, Integer> tuoi = new HashMap<>();
tuoi.put("An", 25);
tuoi.getOrDefault("Chi", 0);
```

**Bất biến:** `List.of`, `Map.of`, `Set.of` tạo collection không đổi được. [Trí nhớ]

## 10. Lambda và Stream

```java
List<Integer> so = List.of(1, 2, 3, 4);

List<Integer> chanNhan10 = so.stream()
    .filter(x -> x % 2 == 0)
    .map(x -> x * 10)
    .toList();          // [20, 40]
```

- **Lambda** là hàm ẩn danh, triển khai interface chức năng (một phương thức trừu tượng). [Trí nhớ]
- **Stream** xử lý dữ liệu theo chuỗi biến đổi, lazy đến khi có phương thức kết thúc như `toList`, `collect`, `sum`. [Trí nhớ]
- `Stream.toList()` có từ Java 16. [Trí nhớ]

## 11. Ngoại lệ và `try-with-resources`

- **Ngoại lệ kiểm tra (checked):** phải khai báo hoặc bắt, ví dụ `IOException`.
- **Ngoại lệ không kiểm tra (unchecked):** kế thừa `RuntimeException`, ví dụ `NullPointerException`, `IllegalArgumentException`. [Trí nhớ]

```java
try (BufferedReader r = Files.newBufferedReader(Path.of("du_lieu.txt"))) {
    String dong = r.readLine();
} catch (IOException e) {
    throw new UncheckedIOException(e);
}
```

`try-with-resources` tự đóng tài nguyên implement `AutoCloseable`, kể cả khi lỗi. [Trí nhớ]

**Chuỗi nguyên nhân:** `new RuntimeException("thông báo", cause)` giữ nguyên nhân gốc. [Trí nhớ]

## 12. Các tính năng hiện đại: record, sealed, pattern matching, `var`, text block

Java đã phát triển nhanh từ Java 8. Các tính năng hiện đại thường gặp: [Trí nhớ; xác nhận phiên bản từng tính năng]

```java
// var: suy luận kiểu cục bộ (Java 10)
var ten = "An";

// text block (Java 15)
String json = """
    {
      "ten": "An"
    }
    """;

// record: dữ liệu bất biến (Java 16)
record DiemSo(String mon, double so) {}

// sealed: giới hạn lớp con (Java 17)
sealed interface Hinh permits HinhTron, HinhVuong {}
record HinhTron(double r) implements Hinh {}
record HinhVuong(double c) implements Hinh {}

// switch expression và pattern matching (phiên bản ổn định gần đây)
double dt = switch (hinh) {
    case HinhTron t -> Math.PI * t.r() * t.r();
    case HinhVuong v -> v.c() * v.c();
};
```

**Lưu ý:** cú pháp pattern matching trong `switch` đã ổn định ở các phiên bản mới; kiểm tra phiên bản bạn dùng. [Suy luận]

## 13. `Optional` và tránh `null`

```java
Optional<SinhVien> sv = timSinhVien("SV01");
String ten = sv.map(SinhVien::getTen).orElse("không rõ");
```

**Nguyên tắc:** `Optional` là kiểu trả về cho khả năng không có giá trị. Không dùng `Optional` làm tham số hay trường. [Trí nhớ]

---

# PHẦN C. ĐỒNG THỜI

## 14. Thread và `ExecutorService`

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> f = pool.submit(() -> 6 * 7);
System.out.println(f.get());
pool.shutdown();
```

- Dùng **pool** thay vì tạo thread thủ công.
- Luôn gọi `shutdown` để giải phóng tài nguyên. [Trí nhớ]

## 15. `CompletableFuture`

```java
CompletableFuture<String> a = CompletableFuture.supplyAsync(() -> "A");
CompletableFuture<String> b = CompletableFuture.supplyAsync(() -> "B");

String ket = a.thenCombine(b, (x, y) -> x + y).join();   // "AB"
```

Tương tự Promise trong JavaScript, nhưng chạy trên thread pool. [Trí nhớ]

## 16. Virtual thread (Java 21 trở lên)

Virtual thread là thread nhẹ do JVM quản lý, có thể tạo hàng triệu, phù hợp mô hình "một thread một yêu cầu" cho I/O: [Trí nhớ]

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> xuLyYeuCau());
}
```

**Lưu ý:** virtual thread giúp tăng số tác vụ I/O đồng thời, không tăng tốc tính toán CPU. [Suy luận]

## 17. Đồng bộ hóa: `synchronized`, `Lock`, `java.util.concurrent`

```java
class Dem {
    private int n = 0;
    public synchronized void tang() { n++; }
}
```

**Lựa chọn:**
- `synchronized`: đơn giản, đủ cho nhiều trường hợp.
- `ReentrantLock`: linh hoạt hơn (thử khóa, hẹn giờ). [Trí nhớ]
- `AtomicInteger` và các lớp `java.util.concurrent.atomic`: không khóa cho biến đơn. [Trí nhớ]
- `ConcurrentHashMap`: map đồng thời. [Trí nhớ]

---

# PHẦN D. TỔ CHỨC MÃ, BUILD, PHỤ THUỘC

## 18. Package và module

- **Package:** nhóm lớp theo tên miền ngược, ví dụ `com.example.congcu`. [Trí nhớ]
- **Java Platform Module System (JPMS)** từ Java 9: khai báo module bằng `module-info.java`, kiểm soát phụ thuộc và gói công khai. [Trí nhớ]

```java
module com.example.congcu {
    requires java.net.http;
    exports com.example.congcu.api;
}
```

Nhiều dự án vẫn dùng classpath thông thường, JPMS là tùy chọn. [Suy luận]

## 19. Maven và Gradle

**Maven** dùng `pom.xml`, cấu hình khai báo, quy ước thư mục chuẩn: [Trí nhớ]

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>congcu</artifactId>
  <version>0.1.0</version>

  <properties>
    <maven.compiler.release>21</maven.compiler.release>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>6.1.3</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

[Phiên bản JUnit lấy từ trang tài liệu đã kiểm chứng; phiên bản Maven plugin và `release` cần kiểm tra]

```sh
mvn clean verify
mvn dependency:tree
```

**Gradle** dùng Groovy hoặc Kotlin DSL (`build.gradle` hoặc `build.gradle.kts`), linh hoạt hơn và nhanh hơn với build lớn. [Trí nhớ]

```sh
./gradlew build
./gradlew test
```

**Chọn:** Maven đơn giản và chuẩn hóa; Gradle linh hoạt và nhanh cho dự án lớn, nhiều module. [Suy luận]

---

# PHẦN E. CÔNG CỤ CHẤT LƯỢNG MÃ

## 20. Định dạng, lint, và phân tích tĩnh

| Công cụ | Chức năng | Ghi chú |
|---|---|---|
| **Checkstyle** | Kiểm tra phong cách | Cấu hình được [Trí nhớ] |
| **PMD** | Phát hiện mẫu mã xấu | [Trí nhớ] |
| **SpotBugs** | Phát hiện lỗi tiềm tàng từ bytecode | Kế thừa FindBugs [Trí nhớ] |
| **Spotless** | Plugin định dạng cho Maven, Gradle | Dùng Google Java Format, Eclipse formatter [Trí nhớ] |
| **Error Prone** | Kiểm tra lúc biên dịch của Google | [Trí nhớ] |

**Gợi ý:** chạy `spotless:check` và `spotbugs` trong CI. [Suy luận]

---

# PHẦN F. KIỂM THỬ

## 21. JUnit 6: test, assertion, và vòng đời

Trang tài liệu JUnit (docs.junit.org) hiển thị **JUnit 6.1.3** là phiên bản hiện tại, gồm JUnit Platform, JUnit Jupiter, và JUnit Vintage (đã được đánh dấu cho việc di chuyển). [Đã kiểm chứng]

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CongTest {

    @BeforeEach
    void chuanBi() { /* thiết lập trước mỗi test */ }

    @Test
    void congHaiSoDuong() {
        assertEquals(5, CongCu.cong(2, 3));
    }

    @Test
    void chiaChoKhong() {
        assertThrows(ArithmeticException.class, () -> CongCu.chia(1, 0));
    }

    @AfterEach
    void donDep() { /* dọn dẹp sau mỗi test */ }
}
```

**Ghi chú:** tên annotation và assertion được lấy từ kiến thức nền của JUnit 5/6, cần đối chiếu trang "Annotations" và "Assertions" trong tài liệu. [Trí nhớ; trang tổng quan đã đọc không liệt kê chi tiết]

**Chạy:**

```sh
mvn test
./gradlew test
```

## 22. Parameterized test

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "-1, 4, 3",
    "0, 7, 7"
})
void cong(int a, int b, int kyVong) {
    assertEquals(kyVong, CongCu.cong(a, b));
}
```

Đây là tương đương table-driven test trong Go và `pytest.mark.parametrize` trong Python. [Suy luận; trang "Parameterized Classes and Tests" của JUnit cần đối chiếu]

## 23. Mock với Mockito

```java
import static org.mockito.Mockito.*;

var repo = mock(KhoDuLieu.class);
when(repo.laySinhVien("SV01")).thenReturn(new SinhVien("SV01", "An"));

var dv = new DichVu(repo);
assertEquals("An", dv.tenSinhVien("SV01"));
verify(repo, times(1)).laySinhVien("SV01");
```

**Nguyên tắc:** tách phụ thuộc vào interface để mock dễ. Mockito là thư viện phổ biến nhất, [Trí nhớ; kiểm tra phiên bản]. Một số dự án dùng AssertJ cho assertion đọc tự nhiên hơn. [Trí nhớ]

## 24. Kiểm thử tích hợp và độ bao phủ

- **Testcontainers:** chạy cơ sở dữ liệu hoặc dịch vụ thật trong container khi test. [Trí nhớ]
- **Spring Boot Test:** kiểm thử tích hợp cho ứng dụng Spring. [Trí nhớ]
- **JaCoCo:** đo độ bao phủ, tích hợp với Maven và Gradle. [Trí nhớ]

```sh
mvn verify          # với plugin jacoco đã cấu hình
./gradlew jacocoTestReport
```

**Cách hiểu:** độ bao phủ không đảm bảo đúng logic. Kết hợp với test biên và test tích hợp. [Suy luận]

---

# PHẦN G. TỰ ĐỘNG HÓA

## 25. Tác vụ build và script

- **Maven:** `mvn` với các plugin, và profile để tách môi trường.
- **Gradle:** `build.gradle.kts` với task tùy biến.
- **Script shell:** gọi `java -jar` hoặc `mvn` trong pipeline.

```kotlin
// build.gradle.kts (minh họa)
tasks.register("xinChao") {
    doLast { println("xin chào") }
}
```

[Trí nhớ]

## 26. Tích hợp liên tục

Pipeline điển hình:

```
1. Cài JDK đúng phiên bản:   setup-java
2. Cache phụ thuộc:          Maven hoặc Gradle cache
3. Kiểm tra định dạng:       spotless:check
4. Lint và phân tích tĩnh:   checkstyle, spotbugs
5. Kiểm thử:                 mvn verify hoặc ./gradlew check
6. Đóng gói:                 mvn package
```

**Ví dụ GitHub Actions** (minh họa):

```yaml
name: ci
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven
      - run: mvn -B verify
```

[Suy luận; phiên bản JDK và action cần kiểm tra]

## 27. Đóng gói: JAR, jlink, jpackage, container

- **JAR:** tệp nén chứa `.class` và tài nguyên. Tạo bằng `mvn package` hoặc `jar`. [Trí nhớ]
- **Fat JAR (uber JAR):** gom luôn phụ thuộc, chạy bằng `java -jar app.jar`. [Trí nhớ]
- **`jlink`:** tạo runtime rút gọn chỉ gồm module cần thiết. [Trí nhớ]
- **`jpackage`:** tạo bộ cài đặt native cho Windows, macOS, Linux. [Trí nhớ]
- **Container:** ảnh Docker với JRE hoặc JDK nhẹ, phổ biến trong triển khai đám mây. [Trí nhớ]

```sh
java -jar target/congcu-0.1.0.jar
```

---

# PHẦN H. VAI TRÒ CỦA JAVA TRONG THỰC TẾ

## 28. Java được dùng ở đâu

| Lĩnh vực | Ví dụ | Lý do |
|---|---|---|
| Backend doanh nghiệp | Spring Boot, Jakarta EE | Hệ sinh thái lớn, ổn định, tương thích ngược |
| Hệ thống dữ liệu | Kafka, Spark, Flink, Hadoop | Chạy trên JVM, hiệu năng và hệ sinh thái |
| Tài chính, ngân hàng | Hệ thống giao dịch | Kiểm soát kiểu, GC đã trưởng thành |
| Android | Ứng dụng Android (dần chuyển sang Kotlin) | Lịch sử Java trên Android [Trí nhớ] |
| Công cụ | Maven, Gradle, IDE | Viết bằng Java |

[Suy luận và kiến thức nền; không đầy đủ]

## 29. So sánh với Go, Python, C#, Kotlin

| Tiêu chí | Java | Go | Python | C# | Kotlin |
|---|---|---|---|---|---|
| Kiểu | Tĩnh | Tĩnh | Động (có gợi ý) | Tĩnh | Tĩnh |
| Chạy | Bytecode, JIT | Mã máy | Bytecode | Bytecode (.NET), JIT | Bytecode trên JVM |
| GC | Có (nhiều lựa chọn) | Có | Có | Có | Có |
| Đồng thời | Thread, virtual thread, CompletableFuture | Goroutine | asyncio, GIL | async/await, Task | Coroutine |
| Khởi động | Chậm hơn | Nhanh | Nhanh | Trung bình | Chậm hơn |
| Hệ sinh thái doanh nghiệp | Rất mạnh | Tốt | Tốt | Rất mạnh | Mạnh (cùng JVM) |

[Suy luận; so sánh khái quát]

## 30. Các điểm cần cẩn trọng

- **Khởi động và bộ nhớ:** JVM tốn tài nguyên nền. Đo trước khi chọn cho dịch vụ nhỏ hoặc serverless. [Suy luận]
- **`null`:** vẫn có thể xảy ra `NullPointerException`. Dùng `Optional` cho giá trị trả về có thể vắng, và kiểm tra đầu vào. [Trí nhớ]
- **Tương thích phiên bản:** kiểm tra phiên bản JDK trong dự án và trong CI, tránh tính năng không khớp. [Suy luận]
- **Lỗ hổng bảo mật trong thư viện:** kiểm tra phụ thuộc định kỳ (OWASP Dependency-Check, hoặc công cụ tương tự). [Trí nhớ]
- **Deserialization không tin cậy:** `ObjectInputStream` với dữ liệu ngoài rất nguy hiểm, nên tránh. [Trí nhớ]
- **Virtual thread và `synchronized`:** một số trường hợp làm giảm lợi ích của virtual thread, cần kiểm tra theo tài liệu phiên bản đang dùng. [Trí nhớ]

---

# PHẦN I. LỘ TRÌNH HỌC

1. **Cú pháp và kiểu:** biến, kiểu nguyên thủy, chuỗi, mảng, điều kiện, vòng lặp.
2. **Lớp và đối tượng:** constructor, đóng gói, interface, kế thừa.
3. **Collection và generics:** `List`, `Map`, `Set`, generics cơ bản.
4. **Lambda và Stream:** biến đổi dữ liệu theo chuỗi.
5. **Ngoại lệ và tài nguyên:** `try-with-resources`.
6. **Kiểm thử:** JUnit, Mockito, parameterized test.
7. **Build:** Maven hoặc Gradle, cấu trúc dự án chuẩn.
8. **Đồng thời:** `ExecutorService`, `CompletableFuture`, virtual thread.
9. **Dự án:** một API nhỏ (ví dụ Spring Boot hoặc chỉ dùng JDK HTTP), có test, CI, và đóng gói.

**Nguồn học chính thức:** Java Tutorials (dev.java), tài liệu OpenJDK, JUnit User Guide (docs.junit.org). [Đã kiểm chứng: trang JUnit đã đọc; các nguồn khác là tham chiếu]

---

# PHẦN J. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| JVM | Máy ảo chạy bytecode Java |
| JDK / JRE | Bộ phát triển / môi trường chạy |
| Bytecode | Mã trung gian độc lập nền tảng, file `.class` |
| JIT | Biên dịch đúng lúc chạy, tối ưu theo thống kê |
| GC | Bộ thu gom rác |
| Heap | Vùng nhớ cấp phát đối tượng |
| Autoboxing | Tự chuyển kiểu nguyên thủy sang kiểu hộp |
| Type erasure | Xóa thông tin generic lúc biên dịch |
| Record | Lớp dữ liệu bất biến rút gọn |
| Sealed | Giới hạn các lớp con được phép |
| Lambda | Hàm ẩn danh |
| Stream | Chuỗi xử lý dữ liệu lazy |
| Optional | Kiểu cho giá trị có thể vắng |
| Virtual thread | Thread nhẹ do JVM quản lý |
| CompletableFuture | Kết quả bất đồng bộ có thể kết hợp |
| JPMS | Hệ thống module của Java |
| Fat JAR | JAR gom luôn phụ thuộc |
| jlink / jpackage | Tạo runtime rút gọn / bộ cài đặt |
| Mock | Đối tượng giả thay phụ thuộc |
| Parameterized test | Test chạy với nhiều tham số |

---

# PHẦN K. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## K.1. Nguồn đã đối chiếu trong lần này

- **OpenJDK, Projects (JDK)**: https://openjdk.org/projects/jdk/ [đã đọc. Ghi nhận: JDK 27 GA ngày 2026-09-15; JDK 28 đang phát triển; trang không đề cập LTS]
- **JUnit 5/6 User Guide (docs.junit.org)**: https://docs.junit.org/current/user-guide/ [đã đọc. Ghi nhận: phiên bản hiện tại 6.1.3; gồm Platform, Jupiter, Vintage; trang tổng quan không liệt kê chi tiết annotation và assertion]

## K.2. Nguồn không truy cập được

- **Oracle Java SE Support Roadmap**: trả về 403 Forbidden. Thông tin LTS, ngày hết hỗ trợ chưa được xác nhận.

## K.3. Nguồn chưa đối chiếu

- Java Tutorials (dev.java), JEP (Java Enhancement Proposals) để xác nhận phiên bản từng tính năng.
- Tài liệu Maven, Gradle, Mockito, AssertJ, JaCoCo, Testcontainers, Spring Boot.
- Chi tiết các bộ GC và tùy chọn JVM.

## K.4. Điểm cần xác minh

1. **Phiên bản LTS hiện tại:** Tài liệu này dựa trên kiến thức nền rằng JDK 25 là LTS (2025-09), nhưng chưa xác nhận từ nguồn. Cần kiểm tra trang hỗ trợ của Oracle hoặc các nhà phân phối (Temurin, Corretto) để biết LTS nào đang được hỗ trợ và đến khi nào.
2. **Ngày phát hành và phạm vi tính năng** của từng phiên bản (record, sealed, pattern matching, virtual thread): kiến thức nền, cần kiểm tra JEP.
3. **Phiên bản Mockito, JaCoCo, Maven plugin**: cần kiểm tra trên Maven Central.
4. **Cú pháp pattern matching trong `switch`** đã ổn định ở phiên bản nào: cần kiểm tra.
5. **Ví dụ mã chưa được biên dịch và chạy.** Nên thử trên máy, đặc biệt các đoạn JUnit, Mockito, và CompletableFuture.
6. **Các so sánh ngôn ngữ** là khái quát, không phải đo lường.

---

**Tóm tắt để tra nhanh:**
- **Java** = ngôn ngữ kiểu tĩnh chạy trên JVM, bytecode + JIT + GC; tương thích ngược mạnh.
- **Nền tảng:** JDK có `javac`, JRE/JVM để chạy; chọn bộ GC theo nhu cầu độ trễ hay thông lượng.
- **Cú pháp hiện đại:** `var`, record, sealed, pattern matching, text block, Stream, `Optional`.
- **Đồng thời:** `ExecutorService`, `CompletableFuture`, virtual thread cho I/O.
- **Build:** Maven (khai báo, chuẩn hóa) hoặc Gradle (linh hoạt, nhanh cho dự án lớn).
- **Kiểm thử:** JUnit 6 (phiên bản hiện tại theo tài liệu), Mockito, parameterized test, JaCoCo.
- **Cẩn trọng:** khởi động và bộ nhớ, `null`, deserialization không tin cậy, và xác nhận phiên bản LTS trước khi triển khai.
