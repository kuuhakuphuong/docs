# Thuật toán và độ phức tạp: từ cơ bản đến nâng cao

> **Viết cho:** người học thuật toán (sinh viên, lập trình viên muốn hiểu sâu, người chuẩn bị phỏng vấn kỹ thuật hoặc nghiên cứu), đi từ khái niệm đến các thuật toán nâng cao và lý thuyết độ phức tạp.
> **Cách đọc:** theo lớp. Mục đánh dấu **[Nâng cao]** có thể bỏ qua ở lần đọc đầu.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần tra cứu này; `[Trí nhớ]` = kiến thức nền, nên tra sách giáo khoa hoặc bài báo gốc trước khi trích dẫn; `[Suy luận]` = tôi suy ra hoặc minh họa.

---

## Mục lục

- **Lớp 0. Nền tảng: thuật toán là gì, đo như thế nào**
  - 1. Thuật toán và đặc tính của nó
  - 2. Ký hiệu tiệm cận: O, Ω, Θ
  - 3. Phân tích độ phức tạp thời gian và không gian
  - 4. Quy tắc phân tích thực hành
- **Lớp 1. Kỹ thuật thiết kế cơ bản**
  - 5. Chia để trị (divide and conquer)
  - 6. Quy hoạch động (dynamic programming)
  - 7. Tham lam (greedy)
  - 8. Quay lui và nhánh cận (backtracking, branch and bound)
- **Lớp 2. Các thuật toán nền tảng**
  - 9. Sắp xếp
  - 10. Tìm kiếm và cấu trúc dữ liệu cân bằng
  - 11. Đồ thị: duyệt, đường đi ngắn nhất, cây khung
- **Lớp 3. Thuật toán nâng cao**
  - 12. Hàng đợi ưu tiên nâng cao: Fibonacci heap
  - 13. Luồng cực đại (max flow)
  - 14. Cây khung nhỏ nhất nâng cao
  - 15. Xử lý chuỗi: KMP, Aho–Corasick
  - 16. Nhân nhanh: Karatsuba, FFT
  - 17. Cấu trúc tập hợp rời rạc: union-find
  - 18. Cây phân đoạn và cây Fenwick
  - 19. Băm và thuật toán ngẫu nhiên
- **Lớp 4. Độ phức tạp và giới hạn**
  - 20. Lớp P, NP, NP-đầy đủ
  - 21. Phép quy về (reduction) và chứng minh NP-đầy đủ
  - 22. Thuật toán xấp xỉ
  - 23. Độ phức tạp không gian, các lớp khác
- **Lớp 5. Đo, kiểm thử, và chọn thuật toán trong thực tế**
  - 24. Đo thực nghiệm
  - 25. Kiểm thử thuật toán
- **Phụ lục**
  - A. Bảng độ phức tạp các thuật toán chuẩn
  - B. Công thức truy hồi và định lý thợ (Master theorem)
  - C. Thuật ngữ
  - D. Nguồn tham khảo
  - E. Điểm cần xác minh

---

# LỚP 0. NỀN TẢNG

## 1. Thuật toán và đặc tính của nó

**Thuật toán (algorithm)** là một dãy bước hữu hạn, không mơ hồ, nhận đầu vào và cho đầu ra, thực hiện được. [Trí nhớ]

Một thuật toán tốt có ba đặc tính:
- **Đúng (correct):** với mọi đầu vào hợp lệ, cho kết quả đúng. Cần chứng minh, không chỉ kiểm thử.
- **Dừng (terminates):** luôn kết thúc.
- **Hiệu quả (efficient):** dùng ít tài nguyên (thời gian, bộ nhớ) khi đầu vào lớn.

**Ví dụ:** tìm số lớn nhất trong một danh sách có n phần tử. Thuật toán: duyệt một lần, giữ số lớn nhất. Đúng (bất biến: sau mỗi bước, `max` là lớn nhất của phần đã duyệt), dừng (có n bước), tốn O(n) thời gian.

## 2. Ký hiệu tiệm cận: O, Ω, Θ

Ta quan tâm đến **tốc độ tăng** của chi phí khi đầu vào lớn dần, không quan tâm hằng số hay chi tiết nhỏ. [Trí nhớ]

| Ký hiệu | Ý nghĩa | Tương tự |
|---|---|---|
| **f(n) = O(g(n))** | f tăng không nhanh hơn g (cận trên) | f ≤ g, sai khác hằng số |
| **f(n) = Ω(g(n))** | f tăng ít nhất nhanh như g (cận dưới) | f ≥ g |
| **f(n) = Θ(g(n))** | f tăng cùng tốc độ với g (chặt) | f và g cùng bậc |
| **o, ω** | Cận trên, cận dưới không chặt | f nhỏ hơn hẳn g |

**Ví dụ:** f(n) = 3n² + 10n + 7 là Θ(n²). Bỏ hằng số và số hạng bậc thấp.

**Bảng bậc thường gặp (từ nhanh đến chậm):**

| Ký hiệu | Tên | Ví dụ |
|---|---|---|
| O(1) | Hằng số | Truy cập mảng theo chỉ số |
| O(log n) | Logarit | Tìm kiếm nhị phân |
| O(n) | Tuyến tính | Duyệt mảng một lần |
| O(n log n) | Tuyến tính-logarit | Sắp xếp tối ưu so sánh |
| O(n²) | Bình phương | Hai vòng lặp lồng nhau |
| O(2ⁿ) | Hàm mũ | Duyệt mọi tập con |
| O(n!) | Giai thừa | Duyệt mọi hoán vị |

**Lưu ý:** n là kích thước đầu vào. Với đồ thị thường dùng V (đỉnh) và E (cạnh). [Trí nhớ]

## 3. Phân tích độ phức tạp thời gian và không gian

- **Độ phức tạp thời gian:** số phép toán cơ bản theo kích thước đầu vào.
- **Độ phức tạp không gian:** bộ nhớ phụ thêm, không tính đầu vào (trừ khi nói khác).
- **Trường hợp tốt nhất, trung bình, xấu nhất:** khác nhau với cùng thuật toán. Ví dụ quicksort có trường hợp xấu nhất O(n²), trung bình O(n log n). [Trí nhớ]

**Phân tích phân bổ (amortized):** chi phí trung bình trên một chuỗi thao tác, dù một thao tác riêng có thể đắt. Ví dụ: thêm phần tử vào mảng động, thỉnh thoảng phải nhân đôi kích thước (O(n)), nhưng trung bình mỗi lần thêm là O(1). [Trí nhớ]

## 4. Quy tắc phân tích thực hành

1. **Đếm vòng lặp:** vòng lặp đơn theo n là O(n); vòng lồng nhau là O(n·m).
2. **Vòng lặp nhân đôi hoặc chia đôi:** O(log n).
3. **Đệ quy:** viết công thức truy hồi, rồi giải (mục Phụ lục B).
4. **Cộng khi nối tiếp, nhân khi lồng nhau.**
5. **Bỏ hằng số và bậc thấp** khi viết O.
6. **Chú ý cấu trúc dữ liệu:** `list.contains` trong vòng lặp là O(n), thành O(n²) tổng.

---

# LỚP 1. KỸ THUẬT THIẾT KẾ CƠ BẢN

Thuật toán thường được thiết kế theo một số khuôn mẫu. Biết khuôn mẫu giúp nhận ra bài toán.

## 5. Chia để trị (divide and conquer)

**Ý tưởng:** chia bài toán thành các bài con nhỏ hơn cùng dạng, giải đệ quy, rồi gộp kết quả.

```
solve(n):
    if n nhỏ: giải trực tiếp
    chia n thành các phần nhỏ hơn
    kết quả con = solve(phần con) cho mỗi phần
    gộp kết quả con
```

**Ví dụ:** merge sort: chia mảng đôi, sắp xếp hai nửa, trộn. Độ phức tạp O(n log n). [Trí nhớ]

## 6. Quy hoạch động (dynamic programming)

**Ý tưởng:** bài toán có **cấu trúc con tối ưu** (lời giải tối ưu chứa lời giải tối ưu của bài con) và **bài con gối nhau** (cùng bài con xuất hiện nhiều lần). Lưu kết quả bài con để không tính lại.

**Hai cách cài đặt:**
- **Từ trên xuống (memoization):** đệ quy kèm bảng ghi nhớ.
- **Từ dưới lên (tabulation):** điền bảng theo thứ tự tăng dần.

**Ví dụ: dãy Fibonacci**

```python
def fib(n):
    dp = [0] * (n + 1)
    if n >= 1:
        dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```

Đệ quy thuần là O(2ⁿ). Bảng quy hoạch động là O(n) thời gian, O(n) không gian (có thể giảm xuống O(1)).

**Ví dụ khác:** bài toán cái túi (knapsack) 0/1 với n vật và sức chứa W có độ phức tạp O(nW). Đây là thời gian **giả đa thức**, vì nó phụ thuộc vào giá trị W chứ không chỉ kích thước đầu vào theo bit. [Trí nhớ]

## 7. Tham lam (greedy)

**Ý tưởng:** ở mỗi bước, chọn phương án tốt nhất trước mắt. Đúng khi bài toán có **tính chất lựa chọn tham lam** và **cấu trúc con tối ưu**. [Trí nhớ]

**Ví dụ:** lập lịch chọn nhiều hoạt động không chồng lấn: sắp xếp theo thời điểm kết thúc, chọn hoạt động kết thúc sớm nhất còn tương thích. O(n log n). [Trí nhớ]

**Cảnh báo:** tham lam không luôn đúng. Trước khi dùng, cần chứng minh. Ví dụ: với đồng xu tùy ý, tham lam có thể không cho số xu tối thiểu. [Trí nhớ]

## 8. Quay lui và nhánh cận (backtracking, branch and bound)

- **Quay lui:** thử lần lượt các lựa chọn, quay lại khi gặp ngõ cụt. Ví dụ: bài toán N hậu, sudoku. Thường là O(kⁿ) nhưng có cắt tỉa. [Trí nhớ]
- **Nhánh cận:** như quay lui, nhưng ước lượng cận trên hoặc cận dưới để bỏ nhánh không thể tốt hơn lời giải hiện có. Dùng cho bài tối ưu hóa khó. [Trí nhớ]

---

# LỚP 2. THUẬT TOÁN NỀN TẢNG

## 9. Sắp xếp

| Thuật toán | Trung bình | Xấu nhất | Bộ nhớ phụ | Ổn định | Ghi chú |
|---|---|---|---|---|---|
| Chèn (insertion) | O(n²) | O(n²) | O(1) | Có | Tốt cho mảng nhỏ hoặc gần đã sắp |
| Trộn (merge) | O(n log n) | O(n log n) | O(n) | Có | Ổn định, dùng cho danh sách liên kết |
| Nhanh (quick) | O(n log n) | O(n²) | O(log n) | Không | Nhanh trong thực tế, cần chọn pivot tốt |
| Vun đống (heap) | O(n log n) | O(n log n) | O(1) | Không | Không ổn định, không cần bộ nhớ phụ |
| Đếm (counting) | O(n + k) | O(n + k) | O(k) | Có | Khóa là số nguyên trong khoảng k |

**Giới hạn dưới:** sắp xếp dựa trên so sánh cần Ω(n log n) so sánh trong trường hợp xấu nhất. [Trí nhớ]

**Ổn định (stable):** giữ thứ tự tương đối của các phần tử bằng nhau.

## 10. Tìm kiếm và cấu trúc dữ liệu cân bằng

- **Tìm kiếm nhị phân:** trên mảng đã sắp, O(log n). [Trí nhớ]
- **Cây tìm kiếm nhị phân cân bằng (AVL, đỏ-đen):** thêm, xóa, tìm đều O(log n). [Trí nhớ]
- **Bảng băm (hash table):** trung bình O(1) cho tra cứu, thêm, xóa; xấu nhất O(n) khi đụng độ nhiều. [Trí nhớ]

## 11. Đồ thị: duyệt, đường đi ngắn nhất, cây khung

### 11.1. Biểu diễn

- **Danh sách kề:** O(V + E) bộ nhớ. Phổ biến nhất.
- **Ma trận kề:** O(V²) bộ nhớ, tra cạnh O(1).

### 11.2. Duyệt

- **BFS (duyệt theo chiều rộng):** dùng hàng đợi, tìm đường đi ngắn nhất theo số cạnh trên đồ thị không trọng số. O(V + E). [Trí nhớ]
- **DFS (duyệt theo chiều sâu):** dùng ngăn xếp hoặc đệ quy, phát hiện chu trình, thành phần liên thông, sắp xếp topo. O(V + E). [Trí nhớ]

### 11.3. Đường đi ngắn nhất

| Thuật toán | Điều kiện | Độ phức tạp | Ghi chú |
|---|---|---|---|
| **Dijkstra** | Trọng số không âm | O((V + E) log V) với heap nhị phân | Tham lam, chọn đỉnh gần nhất chưa xử lý |
| **Bellman–Ford** | Cho phép trọng số âm | O(V·E) | Phát hiện chu trình âm |
| **Floyd–Warshall** | Mọi cặp đỉnh | O(V³) | Quy hoạch động |

[Trí nhớ; các độ phức tạp này là cách trình bày chuẩn trong giáo trình]

**Lưu ý:** Dijkstra sai khi có cạnh âm. Đây là lỗi phổ biến.

### 11.4. Cây khung nhỏ nhất (MST)

- **Kruskal:** sắp cạnh theo trọng số, thêm cạnh nếu không tạo chu trình (dùng union-find). O(E log E). [Trí nhớ]
- **Prim:** mở rộng cây từ một đỉnh, luôn chọn cạnh nhỏ nhất nối ra ngoài. O(E log V) với heap nhị phân. [Trí nhớ]

---

# LỚP 3. THUẬT TOÁN NÂNG CAO

## 12. Hàng đợi ưu tiên nâng cao: Fibonacci heap

**Vấn đề:** Dijkstra và Prim cần thao tác "giảm khóa" (decrease-key) nhiều lần. Với heap nhị phân, mỗi thao tác là O(log n).

**Fibonacci heap** của Fredman và Tarjan (1984; công bố trên tạp chí 1987) cho phân phối chi phí tốt hơn: thao tác giảm khóa có chi phí phân bổ O(1), xóa phần tử nhỏ nhất là O(log n) phân bổ. [Đã kiểm chứng]

**Hệ quả:** Dijkstra với Fibonacci heap đạt O(E + V log V), tốt hơn O((V+E) log V) khi đồ thị thưa-vừa. Fredman và Tarjan cũng dùng cấu trúc này để cải thiện thuật toán tìm MST và đường đi ngắn nhất cặp đôi. [Đã kiểm chứng]

**Lưu ý thực tế:** Fibonacci heap có hằng số lớn và khó cài đặt. Trong thực tế, heap nhị phân hoặc heap cặp (pairing heap) thường nhanh hơn. [Trí nhớ]

## 13. Luồng cực đại (max flow)

**Bài toán:** trên mạng có nguồn s, đích t, mỗi cạnh có sức chứa. Tìm lượng luồng lớn nhất từ s đến t.

**Định lý max-flow min-cut:** luồng cực đại bằng lát cắt nhỏ nhất. [Trí nhớ]

| Thuật toán | Độ phức tạp | Ý tưởng |
|---|---|---|
| Ford–Fulkerson | Phụ thuộc giá trị luồng, có thể không đa thức | Tìm đường tăng, đẩy luồng |
| Edmonds–Karp | O(V·E²) | Ford–Fulkerson với BFS tìm đường ngắn nhất |
| Dinic | O(V²·E) | Xây đồ thị phân tầng, đẩy theo khối |
| Push–relabel | O(V²√E) (dạng tối ưu) | Đẩy luồng cục bộ, gán nhãn chiều cao |

[Trí nhớ; độ phức tạp là các cận chuẩn trong giáo trình, cần đối chiếu sách cụ thể]

**Ứng dụng:** ghép cặp hai phía, phân phối tài nguyên, phát hiện lát cắt trong mạng. [Trí nhớ]

## 14. Cây khung nhỏ nhất nâng cao

Ngoài Kruskal và Prim (mục 11.4), có các thuật toán tốt hơn trong trường hợp đặc biệt:
- **Borůvka:** song song hóa được, O(E log V). [Trí nhớ]
- **Chazelle (2000):** đạt O(E·α(E,V)) với α là hàm nghịch đảo Ackermann. [Trí nhớ, cần kiểm tra]
- **Fredman–Tarjan với Fibonacci heap:** O(E + V log V) cho Prim. [Đã kiểm chứng: mục 12]

## 15. Xử lý chuỗi: KMP, Aho–Corasick

**Bài toán tìm mẫu:** tìm mọi lần xuất hiện của mẫu P (độ dài m) trong văn bản T (độ dài n).

**Tìm đơn giản:** O(n·m) trong trường hợp xấu nhất. [Trí nhớ]

**KMP (Knuth–Morris–Pratt):** tiền xử lý mẫu thành mảng "hàm lỗi" (failure function) để không bao giờ lùi con trỏ trên văn bản. Tổng O(n + m). [Trí nhớ]

**Aho–Corasick:** tìm đồng thời nhiều mẫu, xây một automaton từ tập mẫu. O(tổng độ dài mẫu + n + số lần xuất hiện). Ứng dụng: lọc từ khóa, phát hiện mẫu trong mạng. [Trí nhớ]

**Hậu tố (suffix array, suffix tree):** cấu trúc chỉ mục cho phép tìm mẫu trong O(m) sau khi xây. Xây suffix array trong O(n log n) hoặc nhanh hơn. [Trí nhớ]

## 16. Nhân nhanh: Karatsuba, FFT

**Vấn đề:** nhân hai số n chữ số, hoặc hai đa thức bậc n.

### 16.1. Karatsuba

Nhân hai số bằng cách chia đôi và giảm số phép nhân con từ 4 xuống 3:

```
x = a·B + b,   y = c·B + d
xy = ac·B² + (ad + bc)·B + bd
ad + bc = (a + b)(c + d) − ac − bd      ← chỉ cần 3 phép nhân
```

Độ phức tạp O(n^log₂3) ≈ O(n^1.585). Karatsuba được công bố vào đầu thập niên 1960 (cùng Ofman, bài báo năm 1962). [Trí nhớ, cần kiểm tra năm và chi tiết]

### 16.2. FFT (biến đổi Fourier nhanh)

Nhân đa thức bậc n bằng cách chuyển sang miền điểm, nhân từng điểm, rồi chuyển ngược. Thời gian O(n log n). [Trí nhớ]

**Ứng dụng:** xử lý tín hiệu, nhân số nguyên lớn trong thư viện, tính tích chập. [Trí nhớ]

## 17. Cấu trúc tập hợp rời rạc: union-find

**Bài toán:** quản lý các tập rời nhau, hỗ trợ hai thao tác: `find` (thuộc tập nào) và `union` (hợp hai tập).

**Cài đặt:** cây có gốc, dùng **nén đường đi (path compression)** và **hợp theo hạng (union by rank)**.

**Độ phức tạp:** với m thao tác trên n phần tử, tổng là O(m·α(n)), trong đó α là **hàm nghịch đảo Ackermann**, tăng cực kỳ chậm (với mọi n thực tế, α(n) ≤ 4). [Trí nhớ; cận của Tarjan và Van Leeuwen]

**Ứng dụng:** Kruskal, phát hiện chu trình, thành phần liên thông động. [Trí nhớ]

## 18. Cây phân đoạn và cây Fenwick

Cho mảng, cần trả lời truy vấn tổng hoặc min/max trên đoạn và cập nhật điểm:

- **Cây phân đoạn (segment tree):** truy vấn và cập nhật O(log n). Linh hoạt với nhiều phép toán kết hợp. [Trí nhớ]
- **Cây Fenwick (Binary Indexed Tree):** O(log n) cho cập nhật và truy vấn tổng tiền tố, đơn giản và nhỏ gọn hơn. [Trí nhớ]

## 19. Băm và thuật toán ngẫu nhiên

### 19.1. Băm

- **Hàm băm tốt:** phân bố đều, nhanh.
- **Băm mở (open addressing) và băm nối (chaining):** xử lý đụng độ.
- **Băm đa thức (rolling hash, Rabin–Karp):** tính băm chuỗi cửa sổ trượt trong O(1) mỗi bước, dùng cho tìm chuỗi. [Trí nhớ]

### 19.2. Thuật toán ngẫu nhiên

- **Las Vegas:** luôn đúng, thời gian ngẫu nhiên. Ví dụ quicksort với pivot ngẫu nhiên. [Trí nhớ]
- **Monte Carlo:** thời gian cố định, có xác suất sai nhỏ. Ví dụ kiểm tra số nguyên tố Miller–Rabin. [Trí nhớ]

**Lợi ích:** thuật toán ngẫu nhiên thường đơn giản và nhanh, với xác suất sai hoặc chậm có thể kiểm soát. [Suy luận]

---

# LỚP 4. ĐỘ PHỨC TẠP VÀ GIỚI HẠN

## 20. Lớp P, NP, NP-đầy đủ

- **P:** các bài toán quyết định giải được bằng thuật toán **đa thức** (O(nᶜ) với hằng c). [Trí nhớ]
- **NP:** các bài toán quyết định mà **lời giải có thể kiểm tra** trong thời gian đa thức. [Trí nhớ]
- **NP-đầy đủ (NP-complete):** thuộc NP, và mọi bài NP khác đều quy về được nó trong thời gian đa thức. [Trí nhớ]

**Định lý Cook–Levin (1971):** bài toán thỏa mãn công thức logic (SAT) là NP-đầy đủ. [Đã kiểm chứng: nguồn về Karp nêu Cook 1971 cho SAT]

**Karp (1972):** dùng phép quy về từ SAT, chỉ ra 21 bài toán tổ hợp và đồ thị khác là NP-đầy đủ, trong đó có clique, vertex cover, Hamilton cycle, và bài toán tập đỉnh độc lập. [Đã kiểm chứng]

**Câu hỏi P = NP?** chưa được giải. Nếu P = NP, nhiều bài toán mật mã sẽ bị phá. Phần lớn nhà nghiên cứu tin P ≠ NP, nhưng đây là giả thuyết chưa chứng minh. [Trí nhớ]

## 21. Phép quy về (reduction) và chứng minh NP-đầy đủ

**Quy về A → B:** nếu giải được B, ta giải được A, qua một phép biến đổi đầu vào trong thời gian đa thức.

**Quy trình chứng minh bài X là NP-đầy đủ:**
1. Chứng minh X ∈ NP: lời giải kiểm tra được trong thời gian đa thức.
2. Chọn một bài NP-đầy đủ đã biết (ví dụ 3-SAT, Clique).
3. Xây phép biến đổi từ đầu vào của bài đã biết sang đầu vào của X.
4. Chứng minh: đầu vào có lời giải ⇔ đầu ra có lời giải, và phép biến đổi chạy trong thời gian đa thức.

**Ví dụ ý tưởng:** Vertex Cover quy về từ Clique bằng cách lấy phần bù của đồ thị. [Trí nhớ]

## 22. Thuật toán xấp xỉ

Với bài NP-khó, nếu không cần đáp án tối ưu tuyệt đối, có thể dùng **thuật toán xấp xỉ** cho nghiệm trong phạm vi tỷ lệ α so với tối ưu, trong thời gian đa thức. [Trí nhớ]

**Ví dụ:**
- Vertex cover: thuật toán chọn cạnh tham lam cho nghiệm không quá 2 lần tối ưu. [Trí nhớ]
- Bài người du lịch (TSP) metric: thuật toán Christofides cho tỷ lệ 3/2. [Trí nhớ]

**Lược đồ xấp xỉ đa thức (PTAS):** với mọi ε > 0, có thuật toán (1+ε)-xấp xỉ trong thời gian đa thức theo n (có thể phụ thuộc ε). [Trí nhớ]

## 23. Độ phức tạp không gian và các lớp khác

- **L (logarit không gian):** bài giải được với O(log n) bộ nhớ làm việc. [Trí nhớ]
- **PSPACE:** giải được với bộ nhớ đa thức, có thể rất lâu. Chứa NP. [Trí nhớ]
- **EXPTIME:** giải được trong thời gian hàm mũ. [Trí nhớ]
- **co-NP:** phần bù của NP. Câu hỏi "NP = co-NP?" chưa được giải. [Trí nhớ]

**Quan hệ đã biết:** L ⊆ NL ⊆ P ⊆ NP ⊆ PSPACE ⊆ EXPTIME, trong đó nhiều dấu bao hàm chưa biết là thật sự hay bằng nhau. [Trí nhớ]

---

# LỚP 5. ĐO, KIỂM THỬ VÀ CHỌN THUẬT TOÁN

## 24. Đo thực nghiệm

Độ phức tạp tiệm cận không phải toàn bộ câu chuyện. Hằng số, cache, và phân bố dữ liệu có thể đảo ngược kết luận. Ví dụ: quicksort thường nhanh hơn merge sort trên mảng ngẫu nhiên, dù cả hai đều O(n log n). [Trí nhớ]

**Quy trình đo đáng tin:**
1. Chọn kích thước đầu vào tăng dần (ví dụ n = 10³, 10⁴, 10⁵, 10⁶).
2. Nhiều lần cho mỗi kích thước, báo cáo trung vị và phân vị.
3. Vẽ đồ thị log-log để thấy độ dốc (ước lượng bậc đa thức).
4. Tách thời gian khởi tạo khỏi thời gian thuật toán.
5. Ghi lại phần cứng, ngôn ngữ, phiên bản, cờ biên dịch.

**Cảnh báo:** độ dốc đo được chỉ là gợi ý, không chứng minh. Chứng minh vẫn cần lập luận toán học.

## 25. Kiểm thử thuật toán

**Ba loại kiểm thử chính:**

| Loại | Cách làm | Ví dụ |
|---|---|---|
| **Ví dụ thủ công** | Đầu vào nhỏ, kết quả biết trước | Danh sách rỗng, một phần tử, trùng lặp |
| **So sánh với phiên bản chậm** | Chạy thuật toán nhanh và phiên bản vét cạn trên đầu vào ngẫu nhiên nhỏ, so kết quả | Dijkstra so với Bellman–Ford |
| **Tính chất (property-based)** | Kiểm tra bất biến cho mọi đầu vào sinh ngẫu nhiên | Đầu ra sắp xếp là hoán vị của đầu vào và không giảm |

**Kiểm thử biên:** rỗng, một phần tử, số âm, số cực lớn, trùng lặp, đồ thị không liên thông, chu trình.

**Kiểm thử hiệu năng:** đo trên dữ liệu đại diện, cả trường hợp xấu nhất thường gặp.

**Nguyên tắc:** kiểm thử không chứng minh đúng. Với thuật toán quan trọng, cần chứng minh bằng bất biến và quy nạp. [Suy luận]

---

# PHỤ LỤC

## A. Bảng độ phức tạp các thuật toán chuẩn

| Bài toán | Thuật toán | Thời gian | Không gian |
|---|---|---|---|
| Sắp xếp | Merge sort | O(n log n) | O(n) |
| Sắp xếp | Quicksort | O(n log n) TB, O(n²) XN | O(log n) |
| Tìm kiếm trên mảng đã sắp | Nhị phân | O(log n) | O(1) |
| Đường ngắn nhất, không âm | Dijkstra (heap nhị phân) | O((V+E) log V) | O(V) |
| Đường ngắn nhất, Fibonacci | Dijkstra (Fibonacci heap) | O(E + V log V) | O(V) |
| Đường ngắn nhất, có âm | Bellman–Ford | O(VE) | O(V) |
| Mọi cặp đỉnh | Floyd–Warshall | O(V³) | O(V²) |
| Cây khung | Kruskal | O(E log E) | O(E) |
| Cây khung | Prim (heap nhị phân) | O(E log V) | O(V) |
| Luồng cực đại | Edmonds–Karp | O(VE²) | O(E) |
| Tìm mẫu | KMP | O(n + m) | O(m) |
| Nhiều mẫu | Aho–Corasick | O(tổng mẫu + n) | O(tổng mẫu) |
| Nhân số | Karatsuba | O(n^1.585) | O(n) |
| Nhân đa thức | FFT | O(n log n) | O(n) |
| Tập rời | Union-find | O(m·α(n)) | O(n) |
| Đoạn | Segment tree | O(log n) mỗi thao tác | O(n) |

[Trí nhớ; cần đối chiếu sách giáo khoa cụ thể]

## B. Công thức truy hồi và định lý thợ (Master theorem)

Đệ quy dạng T(n) = a·T(n/b) + f(n), với a ≥ 1, b > 1:

Gọi d = log_b(a). Khi f(n) = Θ(n^d):
- Nếu f(n) = O(n^(d−ε)) với ε > 0: T(n) = Θ(n^d).
- Nếu f(n) = Θ(n^d): T(n) = Θ(n^d · log n).
- Nếu f(n) = Ω(n^(d+ε)) và thỏa điều kiện đều (regularity): T(n) = Θ(f(n)).

**Ví dụ:** merge sort: T(n) = 2T(n/2) + O(n). a = 2, b = 2, d = 1, f(n) = Θ(n¹). Trường hợp 2: T(n) = Θ(n log n). [Trí nhớ]

**Ví dụ Karatsuba:** T(n) = 3T(n/2) + O(n). d = log₂3 ≈ 1.585, f(n) = O(n^(d−ε)). Trường hợp 1: T(n) = Θ(n^1.585). [Trí nhớ]

## C. Thuật ngữ

| Thuật ngữ | Giải thích |
|---|---|
| Thuật toán | Dãy bước hữu hạn, xác định, giải bài toán |
| Độ phức tạp | Tốc độ tăng tài nguyên theo kích thước đầu vào |
| Tiệm cận | Hành vi khi kích thước tiến tới vô cùng |
| Amortized | Chi phí trung bình trên chuỗi thao tác |
| Cấu trúc con tối ưu | Lời giải tối ưu chứa lời giải tối ưu của bài con |
| Bài con gối nhau | Cùng bài con xuất hiện nhiều lần |
| Tham lam | Chọn tối ưu cục bộ ở mỗi bước |
| Quay lui | Thử và quay lại |
| Phép quy về | Biến đổi bài này thành bài khác |
| P, NP, NP-đầy đủ | Các lớp độ phức tạp |
| Thuật toán xấp xỉ | Nghiệm gần tối ưu trong thời gian đa thức |
| Las Vegas / Monte Carlo | Hai loại thuật toán ngẫu nhiên |
| Max-flow min-cut | Luồng cực đại bằng lát cắt nhỏ nhất |
| α(n) | Hàm nghịch đảo Ackermann, tăng rất chậm |
| Master theorem | Định lý giải công thức truy hồi chia để trị |

## D. Nguồn tham khảo

**Sách giáo khoa:**
- Cormen, Leiserson, Rivest, Stein. *Introduction to Algorithms* (CLRS), xuất bản lần thứ tư. [Trí nhớ, cần kiểm tra phiên bản]
- Kleinberg, Tardos. *Algorithm Design*. [Trí nhớ]
- Sipser, M. *Introduction to the Theory of Computation*. Dùng cho phần P, NP, và các lớp độ phức tạp. [Trí nhớ]
- Arora, S., Barak, B. *Computational Complexity: A Modern Approach*. [Trí nhớ]

**Bài báo:**
- Karp, R. M. (1972). "Reducibility among combinatorial problems." Trong *Complexity of Computer Computations*. Danh sách 21 bài NP-đầy đủ. [Đã kiểm chứng qua nguồn tổng hợp Wikipedia về "Karp's 21 NP-complete problems": https://en.wikipedia.org/wiki/Karp%27s_21_NP-complete_problems]
- Fredman, M. L., Tarjan, R. E. (1987). "Fibonacci heaps and their uses in improved network optimization algorithms." *Journal of the ACM*, 34(3). Bản PDF: https://web.eecs.umich.edu/~pettie/matching/Fredman-Tarjan-Fibonacci-Heaps.pdf [Đã kiểm chứng: nội dung và năm công bố 1984/1987]
- Tarjan, R. E. (1975), bài phân tích union-find. [Trí nhớ]
- Chazelle, B. (2000), thuật toán MST. [Trí nhớ, cần kiểm tra]
- Cook, S. A. (1971), "The complexity of theorem-proving procedures." [Trí nhớ, nhưng khớp với thông tin trong nguồn Karp]
- Karatsuba, A., Ofman, Y. (1962). Nhân nhanh. [Trí nhớ, cần kiểm tra]

**Tài liệu tổng quan:**
- Wikipedia "Karp's 21 NP-complete problems": https://en.wikipedia.org/wiki/Karp%27s_21_NP-complete_problems [đã tra cứu]

## E. Điểm cần xác minh

1. **Năm và chi tiết của Karatsuba (1960/1962):** tìm kiếm không cho kết quả, cần đối chiếu bài gốc.
2. **Độ phức tạp của thuật toán Chazelle cho MST:** dựa vào trí nhớ.
3. **Bảng độ phức tạp:** các công thức là trình bày chuẩn trong giáo trình, nhưng cần kiểm tra lại với sách cụ thể trước khi trích dẫn.
4. **Độ phức tạp của luồng cực đại (push–relabel, Dinic):** dựa vào trí nhớ, cần đối chiếu.
5. **Định lý thợ:** dạng trường hợp 3 có điều kiện regularity, cần kiểm tra chi tiết trong CLRS.
6. **Số liệu ví dụ quicksort so với merge sort:** là nhận định phổ biến, không phải kết quả đo của tôi.
7. **Ngoài phần Fibonacci heap và Karp 21 bài, các tham chiếu bài báo khác chưa đọc trực tiếp.**

---

**Tóm lược:**
- Đo thuật toán bằng ký hiệu tiệm cận, nhưng hằng số và cache vẫn quan trọng khi triển khai.
- Năm khuôn mẫu thiết kế cơ bản: chia để trị, quy hoạch động, tham lam, quay lui, nhánh cận.
- Thuật toán nâng cao (Fibonacci heap, max flow, KMP, FFT, union-find) mở rộng khả năng giải quyết nhiều bài toán thực tế.
- Lý thuyết độ phức tạp (P, NP, NP-đầy đủ) cho biết khi nào không nên tìm lời giải tối ưu chính xác, mà nên dùng xấp xỉ hoặc heuristic.
