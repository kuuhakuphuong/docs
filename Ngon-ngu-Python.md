# Ngôn ngữ lập trình Python: từ lý thuyết đến lập trình, tự động hóa và kiểm thử

> **Viết cho:** người mới học Python, lập trình viên từ ngôn ngữ khác chuyển sang, và người muốn dùng Python để viết công cụ, tự động hóa, kiểm thử, hoặc phân tích dữ liệu.
> **Cách đọc:** theo phần. Phần A và B giải thích vì sao Python hoạt động như vậy, nền cho các phần sau. Các đoạn mã có chú thích và được viết theo Python hiện hành.
> **Phiên bản tham chiếu:** Python 3.15 (bản bugfix mới nhất đã phát hành, ngày 2026-10-09 theo Python Developer's Guide). Python 3.14 cũng đang được hỗ trợ bugfix. Python 3.10 đã hết hỗ trợ từ 2026-10-01.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với tài liệu chính thức; `[Trí nhớ]` = kiến thức nền, cần kiểm tra lại; `[Suy luận]` = tôi suy ra hoặc minh họa.

---

## Mục lục

- **Phần A. Python là gì và vì sao được thiết kế như vậy**
  - 1. Lịch sử và triết lý
  - 2. Python chạy như thế nào
  - 3. Kiểu dữ liệu: động, mạnh, và gợi ý kiểu
- **Phần B. Các khái niệm cốt lõi**
  - 4. Cú pháp, thụt lề, và kiểu cơ bản
  - 5. Cấu trúc dữ liệu: list, tuple, dict, set
  - 6. Hàm, tham số, và các cạm bẫy
  - 7. Comprehension, generator, và lazy evaluation
  - 8. Lớp, đối tượng, và dataclass
  - 9. Ngoại lệ và quản lý tài nguyên (context manager)
  - 10. Decorator
  - 11. Gợi ý kiểu (type hints) và Protocol
- **Phần C. Tổ chức mã, môi trường, và phụ thuộc**
  - 12. Module và package
  - 13. Môi trường ảo, pip, và pyproject.toml
- **Phần D. Đồng thời: thread, process, asyncio**
  - 14. GIL và ý nghĩa thực tế
  - 15. Thread, process, và asyncio: chọn cái nào
- **Phần E. Công cụ chất lượng mã**
  - 16. Định dạng, lint, kiểm tra kiểu
- **Phần F. Kiểm thử**
  - 17. unittest và pytest
  - 18. Fixture và parametrize
  - 19. Mock và phụ thuộc
  - 20. Kiểm thử thuộc tính và doctest
  - 21. Độ bao phủ
- **Phần G. Tự động hóa**
  - 22. Script và thao tác tệp, tiến trình
  - 23. Dòng lệnh với argparse
  - 24. Tác vụ lặp lại, pre-commit, nox/tox
  - 25. Tích hợp liên tục (CI)
  - 26. Đóng gói và phát hành
- **Phần H. Vai trò của Python trong thực tế**
  - 27. Python được dùng ở đâu
  - 28. Python so với một số ngôn ngữ khác
  - 29. Các điểm cần cẩn trọng
- **Phần I. Lộ trình học**
- **Phần J. Thuật ngữ**
- **Phần K. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. PYTHON LÀ GÌ VÀ VÌ SAO ĐƯỢC THIẾT KẾ NHƯ VẬY

## 1. Lịch sử và triết lý

Python do Guido van Rossum tạo ra vào cuối thập niên 1980, phát hành lần đầu năm 1991. [Trí nhớ]

Triết lý thể hiện trong **"The Zen of Python"** (PEP 20), có thể xem bằng lệnh `import this`. Một số nguyên tắc: [Trí nhớ]

- Đẹp hơn xấu, rõ ràng hơn ẩn ý.
- Đơn giản hơn phức tạp.
- Nên có một, và chỉ một, cách hiển nhiên để làm việc.
- Lỗi không nên bị bỏ qua một cách im lặng.

**Hệ quả:** Python ưu tiên **dễ đọc** và **năng suất**. Đổi lại, mã thường chậm hơn ngôn ngữ biên dịch như Go hoặc C. Nhiều thư viện hiệu năng cao được viết bằng C và được Python gọi qua giao diện. [Suy luận]

## 2. Python chạy như thế nào

**CPython** (triển khai tham chiếu, phổ biến nhất) làm việc như sau: [Trí nhớ]

```
Mã nguồn .py
    │  biên dịch thành bytecode (.pyc)
    ▼
Bytecode
    │  máy ảo Python thực thi từng lệnh
    ▼
Kết quả
```

- Mã được **biên dịch thành bytecode**, không phải mã máy. Máy ảo đọc và chạy bytecode.
- Không có bước biên dịch riêng cho người dùng: chạy `python main.py` là đủ.
- Bộ nhớ được quản lý bằng **đếm tham chiếu** (reference counting) và **bộ thu gom rác** cho tham chiếu vòng. [Trí nhớ]

**Chạy nhanh:**

```sh
python main.py
python -c "print('xin chào')"
```

**Kiểm tra phiên bản:**

```sh
python --version
```

**Lưu ý:** trên Windows có thể dùng `py` launcher để chọn phiên bản: `py -3.14 main.py`. [Trí nhớ]

## 3. Kiểu dữ liệu: động, mạnh, và gợi ý kiểu

- **Kiểu động (dynamic typing):** kiểu được kiểm tra lúc chạy, không phải lúc biên dịch. Biến không khai báo kiểu cố định.
- **Kiểu mạnh (strong typing):** Python không tự ép kiểu một cách ngầm. `"1" + 1` gây lỗi, không cho kết quả bất ngờ. [Trí nhớ]
- **Gợi ý kiểu (type hints):** từ Python 3.5, có thể chú thích kiểu. Python **không thực thi** chúng khi chạy; công cụ như mypy hoặc IDE kiểm tra. [Đã kiểm chứng: tài liệu typing nêu rõ runtime không thực thi gợi ý kiểu, do công cụ bên thứ ba sử dụng]

```python
def cong(a: int, b: int) -> int:
    return a + b
```

**Ý nghĩa:** gợi ý kiểu là tài liệu có thể kiểm tra được, không phải ràng buộc lúc chạy.

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 4. Cú pháp, thụt lề, và kiểu cơ bản

**Thụt lề quyết định khối lệnh.** Không có ngoặc nhọn. Dùng 4 dấu cách là chuẩn (PEP 8). [Trí nhớ]

```python
ten = "An"
tuoi = 25
gia = 1_000_000          # dấu gạch dưới để dễ đọc
la_sinh_vien = True
```

**Kiểu cơ bản:**

| Kiểu | Ví dụ | Ghi chú |
|---|---|---|
| `int` | `42`, `10**20` | Số nguyên không giới hạn độ lớn |
| `float` | `3.14` | Số thực nhị phân, có sai số làm tròn |
| `Decimal` | `Decimal("0.1")` | Dùng cho tiền tệ, từ module `decimal` |
| `str` | `"xin chào"` | Chuỗi Unicode bất biến |
| `bool` | `True`, `False` | Là con của `int` |
| `None` | `None` | Giá trị rỗng |
| `bytes` | `b"abc"` | Dữ liệu nhị phân bất biến |

**Chuỗi định dạng (f-string):**

```python
ten = "An"
diem = 8.5
print(f"Sinh viên {ten} được {diem:.1f} điểm")
```

**Vòng lặp và điều kiện:**

```python
for i in range(5):
    if i % 2 == 0:
        print(i)
else:
    print("vòng lặp kết thúc bình thường")   # else của for chạy khi không có break

while True:
    break
```

**Toán tử so sánh chuỗi:** `a < b < c` là hợp lệ và có nghĩa như trong toán học. [Trí nhớ]

**Kiểm tra thành viên:** `in` cho list, tuple, dict, set, chuỗi. [Trí nhớ]

## 5. Cấu trúc dữ liệu: list, tuple, dict, set

| Kiểu | Có thứ tự | Có thể thay đổi | Trùng lặp | Ví dụ |
|---|---|---|---|---|
| `list` | Có | Có | Có | `[1, 2, 3]` |
| `tuple` | Có | Không | Có | `(1, 2, 3)` |
| `dict` | Có (theo thứ tự chèn) | Có | Khóa không trùng | `{"a": 1}` |
| `set` | Không | Có | Không | `{1, 2, 3}` |

```python
so = [3, 1, 2]
so.append(4)
so.sort()

toa_do = (10, 20)            # tuple không đổi được
x, y = toa_do                # giải nén

tuoi = {"An": 25, "Bình": 30}
tuoi["Chi"] = 22
for ten, t in tuoi.items():
    print(ten, t)

tap = {1, 2, 2, 3}           # {1, 2, 3}
```

**`dict.get`** tránh lỗi khi khóa không tồn tại:

```python
tuoi.get("Dũng", 0)          # trả về 0 nếu không có
```

## 6. Hàm, tham số, và các cạm bẫy

```python
def chao(ten: str, *, ngu: str = "xin chào") -> str:
    return f"{ngu}, {ten}!"

chao("An")
chao("An", ngu="Hi")         # tham số từ khóa, bắt buộc sau dấu *
```

- `*args` nhận tham số vị trí dạng tuple, `**kwargs` nhận tham số từ khóa dạng dict.
- Tham số sau `*` chỉ truyền được theo tên.

**Cạm bẫy kinh điển: giá trị mặc định là đối tượng có thể thay đổi.**

```python
def them(x, ds=[]):      # SAI: danh sách được tạo một lần, dùng chung
    ds.append(x)
    return ds

them(1)   # [1]
them(2)   # [1, 2]  — không mong đợi
```

**Sửa:**

```python
def them(x, ds=None):
    if ds is None:
        ds = []
    ds.append(x)
    return ds
```

Đây là lỗi rất hay gặp, nên nhớ quy tắc: **không dùng list, dict, set làm mặc định**. [Trí nhớ]

**Hàm là đối tượng hạng nhất:** có thể truyền, trả về, và gán vào biến. [Trí nhớ]

## 7. Comprehension, generator, và lazy evaluation

**List comprehension:**

```python
binh_phuong = [x * x for x in range(10) if x % 2 == 0]
```

**Generator** tạo giá trị lần lượt, tiết kiệm bộ nhớ:

```python
def so_chan(n):
    for i in range(n):
        if i % 2 == 0:
            yield i

for v in so_chan(10**6):   # không tạo sẵn cả triệu phần tử
    if v > 10:
        break
```

**Biểu thức sinh (generator expression):**

```python
tong = sum(x * x for x in range(10**6))   # không dựng list trung gian
```

**Lưu ý:** generator chỉ duyệt được một lần. [Trí nhớ]

## 8. Lớp, đối tượng, và dataclass

```python
class SinhVien:
    def __init__(self, mssv: str, ten: str):
        self.mssv = mssv
        self.ten = ten
        self._diem: list[float] = []       # quy ước: _ là nội bộ

    def them_diem(self, d: float) -> None:
        self._diem.append(d)

    def diem_tb(self) -> float:
        return sum(self._diem) / len(self._diem) if self._diem else 0.0

    def __repr__(self) -> str:
        return f"SinhVien({self.mssv!r}, {self.ten!r})"
```

- `self` là tham chiếu đến đối tượng hiện tại, tường minh (không ngầm như `this`).
- Tên bắt đầu bằng `__` và kết thúc bằng `__` là **phương thức đặc biệt** (dunder), như `__init__`, `__repr__`, `__eq__`. [Trí nhớ]
- Python không có quyền riêng tư tuyệt đối. Quy ước `_` là nội bộ, `__` là được đổi tên (name mangling). [Trí nhớ]

**Dataclass** cho lớp chủ yếu chứa dữ liệu, tự sinh `__init__`, `__repr__`, `__eq__`:

```python
from dataclasses import dataclass, field

@dataclass
class SanPham:
    ma: str
    ten: str
    gia: int
    the: list[str] = field(default_factory=list)   # mặc định an toàn
```

**Kế thừa:** Python hỗ trợ đa kế thừa, nhưng ưu tiên **kết hợp (composition)** và **protocol** khi có thể. [Suy luận]

## 9. Ngoại lệ và quản lý tài nguyên (context manager)

**Ngoại lệ** là cơ chế xử lý lỗi chính:

```python
try:
    with open("cau_hinh.json", encoding="utf-8") as f:
        du_lieu = f.read()
except FileNotFoundError as e:
    print("không tìm thấy tệp:", e)
except OSError as e:
    print("lỗi hệ thống tệp:", e)
else:
    print("đọc thành công")          # chạy khi không có ngoại lệ
finally:
    print("luôn chạy")
```

**Ném ngoại lệ:**

```python
def chia(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("mẫu số không được bằng 0")
    return a / b
```

**Chuỗi ngoại lệ:** `raise X from Y` giữ nguyên nguyên nhân gốc. [Trí nhớ]

**Context manager** đảm bảo tài nguyên được dọn dẹp, kể cả khi lỗi:

```python
with open("du_lieu.txt", encoding="utf-8") as f:   # tự đóng tệp
    noi_dung = f.read()
```

**Tự viết context manager:**

```python
from contextlib import contextmanager
import time

@contextmanager
def do_thoi_gian(nhan: str):
    bat_dau = time.perf_counter()
    try:
        yield
    finally:
        print(f"{nhan}: {time.perf_counter() - bat_dau:.3f}s")

with do_thoi_gian("tính toán"):
    sum(range(10**6))
```

**Lưu ý:** dùng `encoding="utf-8"` khi mở tệp văn bản. Bỏ qua tham số này làm kết quả phụ thuộc hệ điều hành, đặc biệt trên Windows. [Trí nhớ]

## 10. Decorator

Decorator là hàm nhận hàm và trả về hàm khác, thường dùng để thêm hành vi:

```python
import functools, time

def do_thoi_gian_ham(f):
    @functools.wraps(f)              # giữ tên và tài liệu của hàm gốc
    def boc(*args, **kwargs):
        bat_dau = time.perf_counter()
        ket_qua = f(*args, **kwargs)
        print(f"{f.__name__}: {time.perf_counter() - bat_dau:.3f}s")
        return ket_qua
    return boc

@do_thoi_gian_ham
def cham():
    time.sleep(0.1)
```

**Cần chú ý:** thiếu `functools.wraps` làm mất tên và tài liệu hàm, gây khó debug và ảnh hưởng công cụ kiểm thử. [Trí nhớ]

## 11. Gợi ý kiểu (type hints) và Protocol

**Kiểu gợi ý phổ biến:**

```python
from collections.abc import Sequence, Mapping
from typing import Optional

def tinh_tb(diem: Sequence[float]) -> float:
    return sum(diem) / len(diem)

def tim_sv(ds: Mapping[str, str], mssv: str) -> Optional[str]:
    return ds.get(mssv)
```

Từ Python 3.10, có thể viết `str | None` thay vì `Optional[str]`. [Trí nhớ]

**Generic (Python 3.12 trở lên, cú pháp PEP 695):** theo tài liệu typing chính thức: [Đã kiểm chứng]

```python
def dau[T](l: Sequence[T]) -> T:
    return l[0]
```

**Protocol (kiểu cấu trúc):** một lớp thỏa Protocol nếu có đủ phương thức, không cần kế thừa:

```python
from typing import Protocol

class DongDuoc(Protocol):
    def close(self) -> None: ...

def dong_tat_ca(ds: list[DongDuoc]) -> None:
    for x in ds:
        x.close()
```

Tài liệu typing chính thức mô tả Protocol là cách kiểu cấu trúc (structural subtyping), tương tự interface ngầm định của Go. [Đã kiểm chứng]

**Các khái niệm khác trong typing:** `TypedDict` (dict có khóa và kiểu xác định), `Literal`, `Final`, `TypeGuard`/`TypeIs`, `Self`, `ParamSpec` (cho decorator). [Đã kiểm chứng: danh sách trong tài liệu typing]

**Kiểm tra kiểu bằng mypy:**

```sh
pip install mypy
mypy src/
```

Tài liệu typing chính thức tham chiếu mypy, và ghi chú một số phần vẫn đặc thù mypy. [Đã kiểm chứng]

---

# PHẦN C. TỔ CHỨC MÃ, MÔI TRƯỜNG, VÀ PHỤ THUỘC

## 12. Module và package

- **Module:** mỗi tệp `.py` là một module. `import` đưa module vào.
- **Package:** thư mục chứa `__init__.py` (hoặc package không có `__init__.py` theo namespace package). [Trí nhớ]

```
du_an/
├── pyproject.toml
├── src/
│   └── congcu/
│       ├── __init__.py
│       ├── xu_ly.py
│       └── cli.py
└── tests/
    └── test_xu_ly.py
```

```python
from congcu.xu_ly import lam_sach
from congcu import cli
```

**Mẫu `src/`:** đặt mã trong `src/` tránh việc test import nhầm mã chưa cài đặt. Đây là khuyến nghị phổ biến trong cộng đồng. [Suy luận]

**Điểm vào thực thi:** `if __name__ == "__main__":` chỉ chạy khi tệp được gọi trực tiếp, không chạy khi import. [Trí nhớ]

```python
def main() -> None:
    print("chạy")

if __name__ == "__main__":
    main()
```

## 13. Môi trường ảo, pip, và pyproject.toml

**Môi trường ảo (virtual environment)** cô lập thư viện của từng dự án:

```sh
python -m venv .venv
.venv\Scripts\activate          # Windows PowerShell: .venv\Scripts\Activate.ps1
source .venv/bin/activate       # Linux, macOS
pip install -U pip
```

**Cài và quản lý phụ thuộc:**

```sh
pip install requests
pip install -r requirements.txt
pip freeze > requirements.txt   # ghi phiên bản đang cài
pip list --outdated
```

**`pyproject.toml`** là tệp cấu hình chuẩn của dự án hiện đại, mô tả metadata, phụ thuộc, và công cụ build:

```toml
[project]
name = "congcu"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.32",
]

[project.scripts]
congcu = "congcu.cli:main"

[build-system]
requires = ["setuptools>=68"]
build-backend = "setuptools.build_meta"
```

**Khuyến nghị:** ghim phiên bản Python tối thiểu bằng `requires-python`, và giữ tệp khóa phụ thuộc (lockfile) nếu có công cụ hỗ trợ. [Suy luận]

**Công cụ quản lý môi trường và phụ thuộc hiện đại** (ví dụ `uv`, Poetry, PDM): có thể thay pip và venv thủ công, nhanh hơn và có lockfile. Cần kiểm tra tài liệu của từng công cụ trước khi chọn. [Trí nhớ]

---

# PHẦN D. ĐỒNG THỜI

## 14. GIL và ý nghĩa thực tế

**GIL (Global Interpreter Lock)** là khóa trong CPython giúp một thời điểm chỉ một luồng thực thi bytecode Python. [Trí nhớ]

**Hệ quả:**
- Luồng (thread) không tăng tốc tính toán CPU thuần túy trên CPython truyền thống.
- Luồng vẫn hữu ích khi chờ I/O (mạng, tệp, cơ sở dữ liệu), vì GIL được nhả khi chờ.
- Từ Python 3.13, có bản build **free-threaded** (không có GIL) như tùy chọn thử nghiệm. [Trí nhớ; cần kiểm tra trạng thái ở phiên bản bạn dùng]

## 15. Thread, process, và asyncio: chọn cái nào

| Nhu cầu | Công cụ | Ghi chú |
|---|---|---|
| Nhiều I/O chờ (gọi API nhiều lần) | `asyncio` hoặc `ThreadPoolExecutor` | asyncio nhẹ hơn với hàng nghìn kết nối |
| Tính toán CPU nặng | `multiprocessing` hoặc `ProcessPoolExecutor` | Mỗi process có GIL riêng |
| Đơn giản, I/O vừa | `concurrent.futures` | API thống nhất |

**asyncio:**

```python
import asyncio

async def tai(url: str) -> str:
    await asyncio.sleep(1)     # giả lập I/O
    return f"đã tải {url}"

async def main() -> None:
    ket_qua = await asyncio.gather(
        tai("a.com"), tai("b.com"), tai("c.com")
    )
    print(ket_qua)

asyncio.run(main())
```

Ba tác vụ chạy đồng thời, tổng khoảng 1 giây, không phải 3 giây. [Suy luận]

**ThreadPoolExecutor:**

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=8) as ex:
    ket_qua = list(ex.map(lambda u: len(u), ["a", "bb", "ccc"]))
```

**Cạm bẫy:** gọi hàm chặn (blocking) bên trong coroutine `async` làm đứng toàn bộ vòng lặp. Dùng thư viện async hoặc chuyển sang luồng. [Trí nhớ]

---

# PHẦN E. CÔNG CỤ CHẤT LƯỢNG MÃ

## 16. Định dạng, lint, kiểm tra kiểu

| Công cụ | Chức năng | Ghi chú |
|---|---|---|
| **ruff** | Lint và định dạng, rất nhanh | Thay thế nhiều công cụ cũ, phổ biến [Trí nhớ] |
| **black** | Định dạng tự động | Phong cách không tranh luận [Trí nhớ] |
| **mypy** | Kiểm tra gợi ý kiểu | Tài liệu typing tham chiếu [Đã kiểm chứng] |
| **pyright** | Kiểm tra kiểu nhanh, dùng trong VS Code | [Trí nhớ] |
| **pylint** | Lint sâu hơn, nhiều quy tắc | [Trí nhớ] |

```sh
ruff check .
ruff format .
mypy src/
```

**Gợi ý:** chạy lint và kiểm tra kiểu trong CI và trước khi commit, như đã làm với `gofmt` và `go vet` trong tài liệu Go. [Suy luận]

---

# PHẦN F. KIỂM THỬ

## 17. unittest và pytest

**unittest** là thư viện chuẩn, không cần cài thêm:

```python
import unittest

def cong(a: int, b: int) -> int:
    return a + b

class TestCong(unittest.TestCase):
    def test_hai_so_duong(self):
        self.assertEqual(cong(2, 3), 5)

if __name__ == "__main__":
    unittest.main()
```

**pytest** là công cụ kiểm thử phổ biến nhất trong cộng đồng. Tài liệu chính thức mô tả cài đặt bằng `pip install -U pytest` và kiểm tra bằng `pytest --version`. Trang tài liệu hiển thị phiên bản 9.1.1. [Đã kiểm chứng]

**Quy tắc tìm test của pytest:** [Đã kiểm chứng]
- Tệp có tên `test_*.py` hoặc `*_test.py`.
- Hàm có tiền tố `test_`.
- Lớp có tiền tố `Test`, không cần kế thừa lớp nào.

```python
# test_cong.py
from congcu.toan import cong

def test_hai_so_duong():
    assert cong(2, 3) == 5

def test_so_am():
    assert cong(-1, 4) == 3
```

```sh
pytest                 # chạy toàn bộ
pytest -v              # chi tiết
pytest -k cong         # lọc theo tên
pytest -x              # dừng ở lỗi đầu tiên
```

**Assert và thông báo lỗi:** pytest hiển thị giá trị trung gian khi `assert` thất bại, nên thường không cần viết thông báo riêng. [Đã kiểm chứng: ví dụ trong tài liệu pytest cho thấy `assert 4 == 5`]

## 18. Fixture và parametrize

**Fixture** cung cấp dữ liệu hoặc tài nguyên cho test:

```python
import pytest

@pytest.fixture
def ds_sinh_vien():
    return [{"mssv": "SV01", "ten": "An"}, {"mssv": "SV02", "ten": "Bình"}]

def test_so_luong(ds_sinh_vien):
    assert len(ds_sinh_vien) == 2
```

**Fixture có dọn dẹp:**

```python
@pytest.fixture
def tep_tam(tmp_path):
    p = tmp_path / "du_lieu.txt"
    p.write_text("abc", encoding="utf-8")
    yield p
    # dọn dẹp sau test (tmp_path tự xóa theo chu kỳ, nhưng có thể làm thêm ở đây)
```

**Parametrize** chạy một test với nhiều đầu vào, tương tự table-driven test của Go:

```python
@pytest.mark.parametrize(
    "a, b, ky_vong",
    [
        (2, 3, 5),
        (-1, 4, 3),
        (-2, -3, -5),
        (0, 7, 7),
    ],
)
def test_cong(a, b, ky_vong):
    assert cong(a, b) == ky_vong
```

**Kiểm tra ngoại lệ:**

```python
def test_chia_cho_0():
    with pytest.raises(ValueError):
        chia(1, 0)
```

## 19. Mock và phụ thuộc

Thay thế phụ thuộc bên ngoài (mạng, cơ sở dữ liệu, đồng hồ) bằng đối tượng giả:

```python
from unittest.mock import Mock, patch

def test_lay_gia_dung_api():
    gia_gia = Mock(return_value=100)
    assert gia_gia() == 100
    gia_gia.assert_called_once()

def test_goi_mang_bi_gia_lap():
    with patch("congcu.dich_vu.requests.get") as get:
        get.return_value.json.return_value = {"gia": 50}
        # gọi hàm cần test ở đây
        get.assert_called_once()
```

**Nguyên tắc:** patch đúng nơi hàm được **sử dụng**, không phải nơi nó được định nghĩa. Đây là lỗi phổ biến nhất khi dùng `patch`. [Trí nhớ]

**Dùng `monkeypatch` của pytest** là cách gọn cho nhiều tình huống:

```python
def test_bien_moi_truong(monkeypatch):
    monkeypatch.setenv("API_KEY", "gia")
    import os
    assert os.environ["API_KEY"] == "gia"
```

## 20. Kiểm thử thuộc tính và doctest

**Kiểm thử thuộc tính (property-based)** bằng thư viện Hypothesis: sinh đầu vào ngẫu nhiên và kiểm tra tính chất bất biến. [Trí nhớ]

```python
from hypothesis import given, strategies as st

@given(st.integers(), st.integers())
def test_cong_giao_hoan(a, b):
    assert cong(a, b) == cong(b, a)
```

**Doctest** là ví dụ trong docstring, vừa là tài liệu vừa là test:

```python
def cong(a: int, b: int) -> int:
    """Cộng hai số.

    >>> cong(2, 3)
    5
    """
    return a + b
```

```sh
python -m doctest -v congcu/toan.py
```

## 21. Độ bao phủ

```sh
pip install pytest-cov
pytest --cov=congcu --cov-report=term-missing
```

**Cách hiểu:** độ bao phủ cho biết dòng nào đã chạy trong test. 100% không đảm bảo đúng, chỉ đảm bảo đã chạy qua. Kết hợp với test biên và property-based test. [Suy luận]

---

# PHẦN G. TỰ ĐỘNG HÓA

Python rất mạnh cho tự động hóa tác vụ hệ thống, nhờ thư viện chuẩn phong phú.

## 22. Script và thao tác tệp, tiến trình

**Đường dẫn với `pathlib`:**

```python
from pathlib import Path

thu_muc = Path("logs")
thu_muc.mkdir(exist_ok=True)

for tep in thu_muc.glob("*.log"):
    print(tep.name, tep.stat().st_size)
```

**Đọc và ghi tệp:**

```python
Path("ket_qua.txt").write_text("xong\n", encoding="utf-8")
noi_dung = Path("ket_qua.txt").read_text(encoding="utf-8")
```

**Chạy lệnh hệ thống với `subprocess`:**

```python
import subprocess

ket_qua = subprocess.run(
    ["git", "status", "--short"],
    capture_output=True,
    text=True,
    check=True,            # ném CalledProcessError nếu mã thoát khác 0
)
print(ket_qua.stdout)
```

**Cần chú ý:**
- Truyền lệnh dưới dạng danh sách, không ghép chuỗi với `shell=True`, để tránh chèn lệnh (command injection). [Trí nhớ]
- Dùng `check=True` để lỗi không bị bỏ qua.

**Ghi nhật ký (logging):** dùng `logging` thay vì `print` cho công cụ thật:

```python
import logging

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger(__name__)
log.info("bắt đầu xử lý")
```

## 23. Dòng lệnh với argparse

```python
import argparse

def main() -> None:
    parser = argparse.ArgumentParser(description="Đếm dòng trong tệp")
    parser.add_argument("tep", help="đường dẫn tệp")
    parser.add_argument("--chi-tiet", action="store_true", help="in thêm thông tin")
    args = parser.parse_args()

    with open(args.tep, encoding="utf-8") as f:
        so_dong = sum(1 for _ in f)
    print(so_dong)

if __name__ == "__main__":
    main()
```

```sh
python dem_dong.py du_lieu.txt --chi-tiet
```

Thư viện chuẩn còn có `click` và `typer` (bên thứ ba) cho giao diện dòng lệnh phong phú hơn. [Trí nhớ]

## 24. Tác vụ lặp lại, pre-commit, nox/tox

**Tác vụ lặp lại:** dùng script Python, Makefile, hoặc công cụ như `nox` (viết tác vụ bằng Python) và `tox` (kiểm thử trên nhiều phiên bản Python).

**`pre-commit`** chạy kiểm tra trước mỗi commit:

```yaml
# .pre-commit-config.yaml (minh họa)
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.9     # kiểm tra phiên bản mới nhất trước khi dùng
    hooks:
      - id: ruff
      - id: ruff-format
```

```sh
pip install pre-commit
pre-commit install
```

[Trí nhớ; số phiên bản phải kiểm tra lại]

**`nox` (ví dụ):**

```python
import nox

@nox.session(python=["3.12", "3.14"])
def tests(session):
    session.install("pytest", ".")
    session.run("pytest")
```

## 25. Tích hợp liên tục (CI)

Pipeline điển hình:

```
1. Cài Python đúng phiên bản
2. Cài phụ thuộc (kể cả công cụ kiểm thử)
3. Kiểm tra định dạng và lint:   ruff check . && ruff format --check .
4. Kiểm tra kiểu:                mypy src/
5. Kiểm thử:                     pytest --cov=congcu
6. Đóng gói:                     python -m build
```

**Ví dụ GitHub Actions** (minh họa, cần điều chỉnh):

```yaml
name: ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.14"
      - run: pip install -e ".[dev]"
      - run: ruff check .
      - run: mypy src/
      - run: pytest --cov=congcu
```

[Suy luận; tên action và phiên bản cần kiểm tra]

## 26. Đóng gói và phát hành

```sh
pip install build
python -m build            # tạo tệp sdist và wheel trong dist/
pip install dist/congcu-0.1.0-py3-none-any.whl
```

**Wheel** là định dạng cài đặt nhanh, chuẩn hóa. **sdist** là mã nguồn đóng gói. [Trí nhớ]

**Phát hành lên PyPI:** dùng `twine upload dist/*` sau khi kiểm tra kỹ. Nên thử trên TestPyPI trước. [Trí nhớ]

---

# PHẦN H. VAI TRÒ CỦA PYTHON TRONG THỰC TẾ

## 27. Python được dùng ở đâu

| Lĩnh vực | Ví dụ | Lý do Python phù hợp |
|---|---|---|
| Khoa học dữ liệu và học máy | NumPy, pandas, scikit-learn, PyTorch | Hệ sinh thái mạnh, cú pháp dễ đọc |
| Web backend | Django, FastAPI, Flask | Năng suất, thư viện đa dạng |
| Tự động hóa và DevOps | Script hệ thống, công cụ hạ tầng | Thư viện chuẩn và subprocess |
| Kiểm thử và QA | pytest, Selenium, Playwright | Dễ viết và đọc test |
| Scripting và công cụ dòng lệnh | argparse, click, typer | Phát triển nhanh |
| Giáo dục | Nhập môn lập trình | Cú pháp gần ngôn ngữ tự nhiên |

[Suy luận và kiến thức nền; danh sách không đầy đủ]

## 28. Python so với một số ngôn ngữ khác

| Tiêu chí | Python | Go | Java | JavaScript |
|---|---|---|---|---|
| Kiểu | Động, có gợi ý kiểu | Tĩnh | Tĩnh | Động |
| Thực thi | Thông dịch (bytecode) | Biên dịch ra mã máy | Bytecode trên JVM | Engine JS (JIT) |
| Tốc độ thuần | Chậm hơn | Nhanh | Nhanh | Nhanh (JIT) |
| Đồng thời | asyncio, thread, process (GIL) | Goroutine | Thread, virtual thread | Event loop |
| Năng suất | Rất cao | Cao | Trung bình | Cao |
| Triển khai | Cần interpreter và môi trường | Một tệp nhị phân | JAR, cần JVM | Cần runtime |

[Suy luận; so sánh khái quát, không phải benchmark]

## 29. Các điểm cần cẩn trọng

- **Hiệu năng:** Python chậm hơn cho vòng lặp CPU. Dùng NumPy hoặc viết phần nóng bằng ngôn ngữ khác, và đo trước khi tối ưu. [Suy luận]
- **Phụ thuộc:** quản lý phiên bản thư viện bằng môi trường ảo và lockfile, tránh xung đột.
- **Kiểu động:** lỗi kiểu có thể đến lúc chạy. Dùng gợi ý kiểu và mypy để bắt sớm.
- **GIL:** không dùng thread để tăng tốc tính toán CPU trên CPython thông thường.
- **Phiên bản:** Python 3.10 đã hết hỗ trợ từ 2026-10-01. Nên chọn phiên bản được hỗ trợ.
- **Bảo mật:** tránh `eval`, `exec`, `pickle` với dữ liệu không tin cậy, và tránh `shell=True` khi ghép chuỗi. [Trí nhớ]

---

# PHẦN I. LỘ TRÌNH HỌC

1. **Cú pháp và kiểu:** biến, chuỗi, list, dict, vòng lặp, hàm. Làm chương trình đếm từ trong tệp.
2. **Cấu trúc dữ liệu và comprehension:** viết các xử lý danh sách bằng comprehension.
3. **Lớp và dataclass:** mô hình hóa một miền nhỏ (sinh viên, sản phẩm).
4. **Ngoại lệ và context manager:** đọc và ghi tệp an toàn.
5. **Module, package, môi trường ảo:** tổ chức một dự án có `pyproject.toml`.
6. **Kiểm thử:** viết test bằng pytest, dùng `parametrize` và `fixture`.
7. **Gợi ý kiểu:** thêm type hints, chạy mypy.
8. **Tự động hóa:** script với `pathlib`, `subprocess`, `argparse`, và `logging`.
9. **Đồng thời:** `asyncio` cho tác vụ I/O.
10. **Dự án thật:** một công cụ dòng lệnh có kiểm thử, lint, CI, và đóng gói.

**Nguồn học chính thức:** tài liệu Python (docs.python.org), tutorial chính thức, và tài liệu pytest. [Đã kiểm chứng: docs.python.org và docs.pytest.org được đối chiếu trong lần này]

---

# PHẦN J. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| CPython | Triển khai tham chiếu của Python, viết bằng C |
| Bytecode | Mã trung gian máy ảo Python thực thi |
| GIL | Khóa toàn cục giới hạn một luồng bytecode tại một thời điểm (CPython) |
| Dynamic typing | Kiểu được kiểm tra lúc chạy |
| Type hints | Chú thích kiểu, không được thực thi lúc chạy |
| Protocol | Kiểu cấu trúc: thỏa nếu có đủ phương thức |
| Dunder | Phương thức đặc biệt `__xxx__` |
| Decorator | Hàm nhận và trả về hàm, thêm hành vi |
| Generator | Hàm trả giá trị lần lượt bằng `yield` |
| Comprehension | Biểu thức sinh list, dict, set ngắn gọn |
| Context manager | Đối tượng dùng với `with`, đảm bảo dọn dẹp |
| Dataclass | Lớp tự sinh các phương thức cơ bản cho dữ liệu |
| Virtual environment | Môi trường cô lập thư viện cho từng dự án |
| pyproject.toml | Tệp cấu hình dự án chuẩn hiện đại |
| Wheel / sdist | Định dạng gói cài đặt / mã nguồn đóng gói |
| Fixture | Dữ liệu hoặc tài nguyên dùng chung cho test trong pytest |
| Parametrize | Chạy một test với nhiều tham số |
| Mock | Đối tượng giả thay thế phụ thuộc |
| Property-based testing | Sinh đầu vào ngẫu nhiên để kiểm tra tính chất |
| Doctest | Test nằm trong docstring |
| Coroutine | Hàm `async`, tạm dừng và tiếp tục |
| Event loop | Vòng lặp điều phối tác vụ của asyncio |
| Lint | Kiểm tra mã để tìm lỗi và vi phạm phong cách |

---

# PHẦN K. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## K.1. Nguồn đã đối chiếu trong lần này

- **Python Developer's Guide, phiên bản**: https://devguide.python.org/versions/ [đã đọc. Ghi nhận: 3.15 là bugfix (phát hành 2026-10-09), 3.14 là bugfix, 3.13 đến 3.11 là security, 3.10 đã hết hỗ trợ từ 2026-10-01, nhánh main là Python 3.16 đang ở giai đoạn feature]
- **pytest, Getting Started**: https://docs.pytest.org/en/stable/getting-started.html [đã đọc. Ghi nhận: phiên bản hiển thị 9.1.1; quy tắc tìm test]
- **Python, thư viện typing**: https://docs.python.org/3/library/typing.html [đã đọc một phần. Ghi nhận: gợi ý kiểu không được thực thi lúc chạy; generic cú pháp PEP 695 từ 3.12; Protocol; tham chiếu mypy. Trang dài, phần TypedDict chưa đọc]

## K.2. Nguồn chưa đối chiếu

- Tài liệu chính thức về `dataclasses`, `asyncio`, `subprocess`, `pathlib`, `argparse`, `logging`, `unittest.mock`.
- Tài liệu của Hypothesis, pytest-cov, pre-commit, nox, ruff, mypy (phiên bản cụ thể).
- PEP 8, PEP 20 (The Zen of Python) và PEP 703 (free-threaded).

## K.3. Điểm cần xác minh

1. **Trạng thái build free-threaded (không GIL):** từ Python 3.13 là tùy chọn. Cần kiểm tra trạng thái ở phiên bản 3.14 và 3.15.
2. **Số phiên bản các công cụ** (ruff, pre-commit hook, actions): thay đổi thường xuyên, kiểm tra trước khi dùng.
3. **Các ví dụ mã** trong tài liệu chưa được chạy thử. Hãy chạy trên máy trước khi dùng, đặc biệt các đoạn `asyncio`, `unittest.mock`, và `subprocess`.
4. **Ví dụ GitHub Actions và nox**: cần đối chiếu cú pháp hiện hành.
5. **Các so sánh ngôn ngữ** (mục 28) là khái quát, không phải đo lường.
6. **Các khuyến nghị về `src/` layout và công cụ quản lý phụ thuộc** là thực hành cộng đồng, không phải quy định.
7. **Lộ trình và nguồn học** cần kiểm tra lại URL tài liệu chính thức.

---

**Tóm tắt để tra nhanh:**
- **Python** = ngôn ngữ dễ đọc, kiểu động, chạy bằng bytecode trên CPython; có gợi ý kiểu được công cụ kiểm tra.
- **Cấu trúc:** module và package, môi trường ảo, `pyproject.toml`.
- **Đồng thời:** GIL giới hạn tính toán CPU trên một luồng; dùng asyncio cho I/O, process cho CPU.
- **Kiểm thử:** pytest với fixture, parametrize, mock; doctest và property-based testing bổ sung.
- **Tự động hóa:** `pathlib`, `subprocess`, `argparse`, `logging`, pre-commit, CI chạy ruff, mypy, pytest.
- **Cẩn trọng:** kiểu động, hiệu năng, mặc định là đối tượng có thể đổi, `shell=True`, phiên bản đã hết hỗ trợ.
