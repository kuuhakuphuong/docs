# Ngôn ngữ JavaScript: từ lý thuyết đến lập trình, tự động hóa và kiểm thử

> **Viết cho:** người mới học JavaScript, lập trình viên từ ngôn ngữ khác muốn hiểu cơ chế bất đồng bộ và các khác biệt của JavaScript, và người cần viết công cụ, API, hoặc ứng dụng web và Node.js.
> **Cách đọc:** theo phần. Phần A và B giải thích JavaScript là gì và cách nó chạy, là nền cho các phần sau. Phần B.12 về vòng lặp sự kiện là phần quan trọng nhất để hiểu hành vi bất đồng bộ.
> **Phiên bản tham chiếu:**
> - Chuẩn ngôn ngữ: ECMAScript. Trang đặc tả tại tc39.es hiện mang tiêu đề "ECMAScript 2027 Language Specification", có khả năng là bản nháp đang phát triển. Các bản đã phát hành chính thức cần kiểm tra tại ecma-international.org.
> - Môi trường chạy Node.js: bản **Current** là v26 (phát hành 2026-05-05); bản **Active LTS** là v24 (bản vá mới nhất v24.22.0). Node.js 25 và 23 đã hết hỗ trợ.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với tài liệu chính thức hoặc MDN; `[Trí nhớ]` = kiến thức nền, cần kiểm tra lại; `[Suy luận]` = tôi suy ra hoặc minh họa.

---

## Mục lục

- **Phần A. JavaScript là gì và vì sao quan trọng**
  - 1. Lịch sử và vị trí
  - 2. JavaScript chạy ở đâu
  - 3. Chuẩn ECMAScript và các phiên bản
- **Phần B. Các khái niệm cốt lõi**
  - 4. Biến, kiểu, và `var`, `let`, `const`
  - 5. Kiểu nguyên thủy, `null`, `undefined`, và ép kiểu
  - 6. Toán tử bằng: `==` và `===`
  - 7. Hàm, phạm vi, và closure
  - 8. `this`
  - 9. Object, mảng, và phương thức hiện đại
  - 10. Lớp (class) và kế thừa nguyên mẫu
  - 11. Module ES
  - 12. Vòng lặp sự kiện, Promise, async/await
  - 13. Xử lý lỗi
  - 14. Iterator, generator, và Symbol
- **Phần C. Môi trường và tổ chức dự án**
  - 15. Node.js, trình duyệt, và npm
  - 16. `package.json` và quản lý phụ thuộc
- **Phần D. Công cụ chất lượng mã**
  - 17. Lint, định dạng, và kiểm tra
- **Phần E. Kiểm thử**
  - 18. Node test runner, Vitest, Jest
  - 19. Mock và phụ thuộc
  - 20. Kiểm thử giao diện và độ bao phủ
- **Phần F. Tự động hóa**
  - 21. Script và thao tác tệp, tiến trình
  - 22. Dòng lệnh
  - 23. Tích hợp liên tục
  - 24. Đóng gói và phát hành
- **Phần G. Vai trò của JavaScript trong thực tế**
  - 25. Dùng ở đâu
  - 26. So sánh với một số ngôn ngữ khác
  - 27. Các điểm cần cẩn trọng
- **Phần H. Lộ trình học**
- **Phần I. Thuật ngữ**
- **Phần J. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. JAVASCRIPT LÀ GÌ VÀ VÌ SAO QUAN TRỌNG

## 1. Lịch sử và vị trí

JavaScript được Brendan Eich tạo ra năm 1995 tại Netscape, ban đầu để làm cho trang web tương tác. [Trí nhớ]

Hôm nay JavaScript là ngôn ngữ duy nhất chạy gốc trong trình duyệt, và cũng chạy trên máy chủ qua Node.js. Vì vậy nó gắn với hầu hết các sản phẩm web. [Suy luận]

**Đừng nhầm với Java:** hai ngôn ngữ không liên quan trực tiếp, chỉ có tên giống nhau do chiến lược tiếp thị thời đó. [Trí nhớ]

## 2. JavaScript chạy ở đâu

- **Trình duyệt:** Chrome, Firefox, Safari, Edge đều có engine JavaScript. Engine phổ biến gồm V8 (Chrome, Node.js), SpiderMonkey (Firefox), JavaScriptCore (Safari). [Trí nhớ]
- **Node.js:** môi trường chạy JavaScript ngoài trình duyệt, thêm API cho tệp, mạng, tiến trình. [Trí nhớ]
- **Các môi trường khác:** Deno, Bun, và runtime nhúng trong ứng dụng. [Trí nhớ]

**Cơ chế chung:** mã được engine phân tích, biên dịch sang dạng trung gian hoặc mã máy (thường bằng JIT, biên dịch đúng lúc chạy), rồi thực thi. Chi tiết tối ưu thay đổi theo engine. [Trí nhớ]

**Chạy nhanh:**

```sh
node --version
node main.js
node -e "console.log(1 + 1)"
```

**Phiên bản Node.js:** bản Current v26 là bản mới nhất có tính năng mới. Bản Active LTS v24 phù hợp môi trường production, theo trang chính thức nên dùng bản Active hoặc Maintenance LTS cho ứng dụng thực tế. [Đã kiểm chứng]

## 3. Chuẩn ECMAScript và các phiên bản

JavaScript là hiện thực hóa của đặc tả **ECMAScript** (ECMA-262). Mỗi năm có phiên bản mới, thêm tính năng qua quy trình TC39 gồm các giai đoạn đề xuất từ 0 đến 4. [Trí nhớ]

Trang đặc tả chính thức hiện mang tiêu đề ECMAScript 2027, với các mục như `using` và `await using` (quản lý tài nguyên tường minh), iterator helpers, và `GroupBy`. Trang chưa cho biết đây là bản đã phát hành hay bản nháp. [Đã kiểm chứng: có mục tương ứng trong trang; trạng thái phát hành cần kiểm tra]

**Cách kiểm tra tính năng trên môi trường:** MDN có bảng tương thích và trình duyệt, Node.js có thông tin qua `node --v8-options` hoặc tài liệu. [Trí nhớ]

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 4. Biến, kiểu, và `var`, `let`, `const`

```js
const ten = "An";     // không gán lại được (liên kết không đổi, object bên trong vẫn có thể đổi)
let diem = 8;          // gán lại được
var cu = 1;            // cách cũ, tránh dùng trong mã mới
```

**Khác biệt chính:**

| Từ khóa | Phạm vi | Gán lại | Hoisting |
|---|---|---|---|
| `var` | Hàm | Có | Khởi tạo `undefined` |
| `let` | Khối `{}` | Có | Có nhưng vùng chết tạm thời (TDZ) |
| `const` | Khối `{}` | Không | Có nhưng vùng chết tạm thời (TDZ) |

**Khuyến nghị:** mặc định dùng `const`, chỉ dùng `let` khi cần gán lại, không dùng `var`. [Suy luận]

**Vùng chết tạm thời (TDZ):** truy cập `let` hoặc `const` trước khi khai báo gây `ReferenceError`. [Trí nhớ]

## 5. Kiểu nguyên thủy, `null`, `undefined`, và ép kiểu

**Kiểu nguyên thủy (primitive):** `number`, `bigint`, `string`, `boolean`, `undefined`, `null`, `symbol`. [Trí nhớ]

```js
typeof 42;          // "number"
typeof 10n;         // "bigint"
typeof "a";         // "string"
typeof undefined;   // "undefined"
typeof null;        // "object"  ← lỗi lịch sử, không phải kiểu object thật
```

**`null` và `undefined`:**
- `undefined`: biến đã khai báo nhưng chưa gán, hoặc thuộc tính không tồn tại.
- `null`: giá trị "rỗng" do lập trình viên gán có chủ đích.

**Ép kiểu ngầm (coercion)** là nguồn lỗi phổ biến:

```js
"5" + 3;       // "53"  (chuỗi)
"5" - 3;       // 2     (số)
[] + {};       // "[object Object]"
```

**Số thực và sai số:** `0.1 + 0.2 === 0.3` là `false`. Với tiền, dùng số nguyên (đơn vị nhỏ nhất) hoặc thư viện số thập phân. [Trí nhớ]

## 6. Toán tử bằng: `==` và `===`

- `===` so sánh **không ép kiểu**: an toàn, nên dùng mặc định.
- `==` ép kiểu trước khi so sánh, dễ gây kết quả khó đoán.

```js
0 == "";        // true
0 === "";       // false
null == undefined;   // true
null === undefined;  // false
```

**Quy tắc thực hành:** luôn dùng `===` và `!==`. Ngoại lệ hiếm: `x == null` để kiểm tra cả `null` và `undefined`. [Suy luận]

**Giá trị sai (falsy):** `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Mọi giá trị khác là đúng (truthy), kể cả `[]` và `{}`. [Trí nhớ]

## 7. Hàm, phạm vi, và closure

**Ba cách khai báo hàm:**

```js
function cong(a, b) { return a + b; }        // khai báo hàm
const nhan = (a, b) => a * b;                // hàm mũi tên
const chia = function (a, b) { return a / b; };  // biểu thức hàm
```

**Hàm mũi tên** không có `this` và `arguments` riêng, khác với hàm thường. Điều này quan trọng khi viết phương thức (mục 8). [Trí nhớ]

**Tham số mặc định và rest:**

```js
function chao(ten = "bạn", ...khac) {
  return `Xin chào ${ten}`;
}
```

**Closure:** hàm ghi nhớ biến của phạm vi nơi nó được tạo, kể cả sau khi phạm vi đó kết thúc:

```js
function taoBoDem() {
  let dem = 0;
  return () => ++dem;
}

const dem1 = taoBoDem();
dem1();   // 1
dem1();   // 2
```

**Ứng dụng:** che giấu dữ liệu, tạo hàm có trạng thái, cấu hình sẵn cho hàm. [Suy luận]

**Cạm bẫy kinh điển với vòng lặp và `var`:**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);   // in 3, 3, 3
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);   // in 0, 1, 2
}
```

`let` tạo biến mới cho mỗi vòng lặp. [Trí nhớ]

## 8. `this`

`this` phụ thuộc vào **cách hàm được gọi**, không phụ thuộc vào nơi viết hàm:

```js
const nguoi = {
  ten: "An",
  chao() {
    return this.ten;       // this là object gọi phương thức
  },
};

nguoi.chao();              // "An"
const f = nguoi.chao;
f();                       // this không xác định: lỗi hoặc undefined trong strict mode
```

**Quy tắc thực hành:**
- Với phương thức của object, dùng cú pháp `chao() {}` hoặc hàm thường.
- Với callback cần `this` của ngoài, dùng hàm mũi tên.
- `bind`, `call`, `apply` đặt `this` tường minh. [Trí nhớ]

## 9. Object, mảng, và phương thức hiện đại

**Object và thuộc tính:**

```js
const sv = { mssv: "SV01", ten: "An", diem: 8.5 };
sv.ten;                   // "An"
sv["ten"];                // cách khác
const { ten, diem } = sv; // destructuring
const copy = { ...sv, diem: 9 };   // spread, tạo object mới
```

**Mảng và các phương thức chức năng (không đổi mảng gốc):**

```js
const so = [1, 2, 3, 4];
so.map(x => x * 2);            // [2, 4, 6, 8]
so.filter(x => x % 2 === 0);   // [2, 4]
so.reduce((a, b) => a + b, 0); // 10
so.find(x => x > 2);           // 3
so.some(x => x > 3);           // true
so.every(x => x > 0);          // true
```

**Phương thức đổi mảng gốc:** `push`, `pop`, `splice`, `sort`, `reverse`. Cần chú ý khi dùng. [Trí nhớ]

**Sao chép mảng không đổi gốc (hiện đại):** `toSorted`, `toReversed`, `with`. [Trí nhớ; kiểm tra hỗ trợ môi trường]

```js
const tang = so.toSorted((a, b) => a - b);
```

**Map và Set:**

```js
const tuoi = new Map([["An", 25]]);
tuoi.set("Bình", 30);
tuoi.get("An");

const duy = new Set([1, 2, 2, 3]);   // {1, 2, 3}
```

**Optional chaining và nullish coalescing:**

```js
const thanh_pho = user?.dia_chi?.thanh_pho ?? "không rõ";
```

`?.` dừng khi gặp `null` hoặc `undefined`. `??` chỉ thay thế khi giá trị là `null` hoặc `undefined`, khác với `||` vốn thay cả `0` và `""`. [Trí nhớ]

**Sao chép sâu:** `structuredClone(obj)` là cách chuẩn hiện nay. [Trí nhớ]

## 10. Lớp (class) và kế thừa nguyên mẫu

```js
class SinhVien {
  #diem = [];                     // trường riêng thật sự (cú pháp #)

  constructor(mssv, ten) {
    this.mssv = mssv;
    this.ten = ten;
  }

  themDiem(d) {
    this.#diem.push(d);
  }

  get diemTb() {
    return this.#diem.length ? this.#diem.reduce((a, b) => a + b, 0) / this.#diem.length : 0;
  }
}
```

**Kế thừa:** `class A extends B`. JavaScript dựa trên **chuỗi nguyên mẫu (prototype chain)**; `class` là cú pháp đường (syntactic sugar) trên cơ chế đó. [Trí nhớ]

**Trường riêng `#`:** thật sự riêng tư trong runtime, khác với quy ước `_` của Python. [Trí nhớ]

## 11. Module ES

```js
// toan.js
export function cong(a, b) { return a + b; }
export default function nhan(a, b) { return a * b; }

// main.js
import nhan, { cong } from "./toan.js";
```

- **ESM** (`import`/`export`) là chuẩn hiện đại, dùng trong trình duyệt và Node.js.
- **CommonJS** (`require`/`module.exports`) là hệ cũ, vẫn còn nhiều trong hệ sinh thái Node.js. [Trí nhớ]

**Trong Node.js:** một tệp được xem là ESM khi `package.json` có `"type": "module"`, hoặc dùng đuôi `.mjs`. [Trí nhớ]

**Khi import tương đối trong ESM,** cần ghi đuôi `.js` đầy đủ. [Trí nhớ]

## 12. Vòng lặp sự kiện, Promise, async/await

Đây là phần **quan trọng nhất** để hiểu JavaScript bất đồng bộ.

### 12.1. Mô hình chạy

JavaScript chạy **một luồng** và thực thi từng công việc (job) **đến khi xong** trước khi làm công việc khác. Các thao tác I/O không chặn: chúng được xử lý bởi môi trường, rồi callback được đưa vào hàng đợi. [Đã kiểm chứng: MDN, Event loop]

```
┌──────────────┐
│  Call stack  │  ← hàm đang chạy
└──────────────┘
┌──────────────┐
│ Microtask    │  ← Promise.then, queueMicrotask, await
│ queue        │
└──────────────┘
┌──────────────┐
│ Task queue   │  ← setTimeout, sự kiện, I/O callback
└──────────────┘
```

### 12.2. Thứ tự thực thi

1. Chạy công việc hiện tại đến khi call stack rỗng.
2. **Rút sạch hàng đợi microtask**, kể cả các microtask được thêm trong lúc rút.
3. Lấy một task từ hàng đợi task, rồi lặp lại bước 2.

[Đã kiểm chứng: trang MDN mô tả microtask được rút hết trước khi lấy task tiếp theo]

**Ví dụ:**

```js
console.log("A");
setTimeout(() => console.log("B"), 0);          // task
Promise.resolve().then(() => console.log("C")); // microtask
console.log("D");
// Kết quả: A, D, C, B
```

- `A` và `D` chạy đồng bộ.
- `C` chạy trước `B` vì microtask được rút trước task.
- `B` chạy cuối dù delay là 0 ms.

[Ví dụ minh họa theo quy tắc của MDN]

### 12.3. Promise

Promise đại diện cho kết quả của thao tác bất đồng bộ, ở một trong ba trạng thái: pending, fulfilled, rejected.

```js
function doiMs(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

doiMs(1000)
  .then(() => console.log("xong"))
  .catch((err) => console.error(err))
  .finally(() => console.log("luôn chạy"));
```

**Các tổ hợp:**
- `Promise.all([...])`: chờ tất cả, lỗi một cái làm cả thất bại.
- `Promise.allSettled([...])`: chờ tất cả, trả trạng thái từng cái.
- `Promise.race([...])`: lấy cái đầu tiên xong.
- `Promise.any([...])`: lấy cái đầu tiên thành công. [Trí nhớ]

### 12.4. async/await

```js
async function layDuLieu(url) {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

- `async` hàm luôn trả về Promise.
- `await` tạm dừng hàm đó đến khi Promise xong, không chặn luồng.
- Lỗi trong `await` được ném như ngoại lệ đồng bộ, bắt bằng `try/catch`. [Trí nhớ]

**Cạm bẫy:** nhiều `await` tuần tự chậm hơn khi các thao tác độc lập. Dùng `Promise.all` cho các tác vụ song song. [Suy luận]

```js
// Chậm: chờ lần lượt
const a = await layA();
const b = await layB();

// Nhanh hơn: chạy song song
const [a2, b2] = await Promise.all([layA(), layB()]);
```

**Cạm bẫy khác:** quên `await` làm nhận Promise thay vì giá trị. Một số lỗi chỉ lộ ra ở xử lý tiếp theo. [Suy luận]

## 13. Xử lý lỗi

```js
try {
  JSON.parse("{không hợp lệ");
} catch (err) {
  console.error("parse lỗi:", err.message);
} finally {
  console.log("dọn dẹp");
}
```

**Lớp lỗi:** `Error` và các lớp con như `TypeError`, `RangeError`, `SyntaxError`. [Trí nhớ]

**Tạo lỗi có ngữ cảnh:**

```js
throw new Error("không đọc được cấu hình", { cause: err });
```

Thuộc tính `cause` giữ nguyên nhân gốc. [Trí nhớ]

**Lỗi không được bắt trong Promise:** nếu `Promise` bị reject mà không có `catch`, môi trường báo `unhandledRejection`. Trong Node.js, mặc định điều này làm tiến trình dừng ở các phiên bản hiện đại. [Trí nhớ]

## 14. Iterator, generator, và Symbol

**Iterator** là đối tượng có phương thức `next()` trả `{ value, done }`. Các kiểu lặp như `for...of`, spread, destructuring đều dùng giao thức này. [Trí nhớ]

```js
function* demTu(n) {        // generator
  for (let i = 1; i <= n; i++) {
    yield i;                // tạm dừng và trả giá trị
  }
}

for (const v of demTu(3)) console.log(v);   // 1, 2, 3
```

**Symbol:** giá trị duy nhất, thường dùng làm khóa thuộc tính ẩn. [Trí nhớ]

**Iterator helpers** (đang được chuẩn hóa trong các phiên bản mới, kiểm tra hỗ trợ): `map`, `filter`, `take` trên iterator. Đặc tả ECMAScript mới có mục liên quan. [Đã kiểm chứng: trang đặc tả nêu iterator helpers]

---

# PHẦN C. MÔI TRƯỜNG VÀ TỔ CHỨC DỰ ÁN

## 15. Node.js, trình duyệt, và npm

| Môi trường | Có gì | Không có gì |
|---|---|---|
| Trình duyệt | DOM, `fetch`, sự kiện giao diện | Hệ thống tệp, tiến trình |
| Node.js | `fs`, `path`, `child_process`, máy chủ HTTP | DOM |

**npm** là trình quản lý gói mặc định, đi kèm Node.js. Các lựa chọn khác: pnpm, yarn, bun. [Trí nhớ]

```sh
npm init -y
npm install express
npm install -D typescript
npm run <tên-script>
```

## 16. `package.json` và quản lý phụ thuộc

```json
{
  "name": "congcu",
  "version": "0.1.0",
  "type": "module",
  "engines": { "node": ">=24" },
  "scripts": {
    "start": "node src/index.js",
    "test": "node --test",
    "lint": "eslint ."
  },
  "dependencies": {},
  "devDependencies": {}
}
```

- `"type": "module"` bật ESM mặc định cho các tệp `.js`. [Trí nhớ]
- `"engines"` khai báo phiên bản Node.js hỗ trợ. [Trí nhớ]
- Lockfile (`package-lock.json`) ghim phiên bản phụ thuộc. Nên commit và dùng `npm ci` trong CI. [Trí nhớ]

---

# PHẦN D. CÔNG CỤ CHẤT LƯỢNG MÃ

## 17. Lint, định dạng, và kiểm tra

| Công cụ | Chức năng | Ghi chú |
|---|---|---|
| **ESLint** | Lint, phát hiện mẫu đáng ngờ | Cấu hình bằng `eslint.config.js` [Trí nhớ] |
| **Prettier** | Định dạng tự động | Phổ biến [Trí nhớ] |
| **Biome** | Lint và định dạng trong một công cụ | Nhanh [Trí nhớ] |

**Chế độ nghiêm ngặt (strict mode):** thêm `"use strict";` ở đầu tệp, hoặc ESM luôn ở chế độ này. Chế độ này gây lỗi với các hành vi nguy hiểm như gán vào biến không khai báo. [Trí nhớ]

**Kiểm tra kiểu:** JavaScript thuần không có kiểm tra kiểu tĩnh. Có thể dùng JSDoc với `// @ts-check` hoặc chuyển sang TypeScript (xem tài liệu TypeScript). [Trí nhớ]

```js
// @ts-check
/** @param {number} a @param {number} b @returns {number} */
function cong(a, b) { return a + b; }
```

---

# PHẦN E. KIỂM THỬ

## 18. Node test runner, Vitest, Jest

**Node test runner** là công cụ kiểm thử tích hợp của Node.js, không cần cài thêm:

```js
// cong.test.js
import { test } from "node:test";
import assert from "node:assert/strict";
import { cong } from "./toan.js";

test("cộng hai số dương", () => {
  assert.equal(cong(2, 3), 5);
});
```

```sh
node --test
```

**Vitest:** công cụ hiện đại, nhanh, cú pháp tương tự Jest, có sẵn hỗ trợ ESM. [Trí nhớ]

**Jest:** lâu đời, phổ biến, cấu hình nhiều hơn với ESM. [Trí nhớ]

**Table-driven (tương tự Go và Python):** dùng `test.each` trong Vitest hoặc Jest, hoặc lặp trong Node test runner. [Trí nhớ]

## 19. Mock và phụ thuộc

Node test runner có `mock` tích hợp:

```js
import { mock, test } from "node:test";
import assert from "node:assert/strict";

test("gọi hàm một lần", () => {
  const f = mock.fn(() => 100);
  assert.equal(f(), 100);
  assert.equal(f.mock.callCount(), 1);
});
```

[Trí nhớ; kiểm tra API theo phiên bản Node]

**Nguyên tắc:** tách phụ thuộc (mạng, đồng hồ, tệp) thành tham số hoặc module có thể thay, để dễ kiểm thử. [Suy luận]

## 20. Kiểm thử giao diện và độ bao phủ

- **Playwright, Cypress:** kiểm thử đầu cuối trên trình duyệt. [Trí nhớ]
- **Testing Library:** kiểm thử giao diện theo cách người dùng tương tác. [Trí nhớ]
- **Độ bao phủ:** Node.js có tùy chọn `--experimental-test-coverage` (kiểm tra trạng thái theo phiên bản). Vitest có `--coverage`. [Trí nhớ]

```sh
node --test --experimental-test-coverage
npx vitest run --coverage
```

**Cách hiểu:** độ bao phủ cho biết dòng đã chạy, không đảm bảo logic đúng. [Suy luận]

---

# PHẦN F. TỰ ĐỘNG HÓA

## 21. Script và thao tác tệp, tiến trình

**Thao tác tệp với `node:fs/promises`:**

```js
import { readFile, writeFile, readdir } from "node:fs/promises";
import path from "node:path";

const noiDung = await readFile("du_lieu.txt", "utf8");
await writeFile("ket_qua.txt", noiDung.toUpperCase(), "utf8");

for (const tep of await readdir("logs")) {
  console.log(path.extname(tep), tep);
}
```

- Trong ESM, dùng `await` ở cấp cao nhất được hỗ trợ trong module. [Trí nhớ]

**Chạy tiến trình con với `node:child_process`:**

```js
import { execFile } from "node:child_process";
import { promisify } from "node:util";

const chay = promisify(execFile);
const { stdout } = await chay("git", ["status", "--short"]);
console.log(stdout);
```

**Cần chú ý:**
- Truyền đối số dưới dạng mảng, không ghép chuỗi với shell, để tránh chèn lệnh. [Trí nhớ]
- Xử lý lỗi: `execFile` ném lỗi khi mã thoát khác 0. [Trí nhớ]

## 22. Dòng lệnh

**Đọc đối số:** `process.argv` cho dữ liệu thô. Thư viện như `commander`, `yargs`, hoặc `parseArgs` của `node:util` (tích hợp) cho cú pháp đầy đủ. [Trí nhớ]

```js
import { parseArgs } from "node:util";

const { values, positionals } = parseArgs({
  options: { chi_tiet: { type: "boolean", short: "v" } },
  allowPositionals: true,
});
console.log(positionals, values.chi_tiet);
```

## 23. Tích hợp liên tục

Pipeline điển hình:

```
1. Cài đúng phiên bản Node:   setup-node với node-version
2. Cài phụ thuộc theo lockfile: npm ci
3. Lint:                       npm run lint
4. Kiểm tra kiểu (nếu có):     npm run typecheck
5. Kiểm thử:                   npm test
6. Build:                      npm run build
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
      - uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

[Suy luận; kiểm tra phiên bản action và phiên bản Node]

## 24. Đóng gói và phát hành

- Thư viện: khai báo `main`, `exports`, và `types` (nếu có) trong `package.json`.
- Kiểm tra gói trước khi phát hành: `npm pack --dry-run`. [Trí nhớ]
- Phát hành lên npm: `npm publish`, nên có phiên bản rõ ràng theo semver. [Trí nhớ]

---

# PHẦN G. VAI TRÒ CỦA JAVASCRIPT TRONG THỰC TẾ

## 25. Dùng ở đâu

| Lĩnh vực | Ví dụ | Ghi chú |
|---|---|---|
| Frontend | React, Vue, Svelte, Angular | Chạy trong trình duyệt |
| Backend | Express, Fastify, NestJS | Node.js |
| Công cụ dòng lệnh | Công cụ build, CLI | Node.js |
| Ứng dụng di động, desktop | React Native, Electron | Tái dùng kiến thức web |
| Tự động hóa và kiểm thử | Playwright, scripts | Node.js |

[Suy luận và kiến thức nền; không đầy đủ]

## 26. So sánh với một số ngôn ngữ khác

| Tiêu chí | JavaScript | TypeScript | Python | Go |
|---|---|---|---|---|
| Kiểu | Động | Tĩnh (tùy chọn chặt) | Động | Tĩnh |
| Chạy | Trực tiếp (engine) | Biên dịch sang JS | Bytecode | Mã máy |
| Đồng thời | Event loop, một luồng | Như JavaScript | asyncio, thread, process | Goroutine |
| Hệ sinh thái web | Gốc | Nhiều, chuẩn hóa | Ít cho frontend | Ít cho frontend |

[Suy luận; so sánh khái quát]

## 27. Các điểm cần cẩn trọng

- **Ép kiểu ngầm:** dùng `===`, và hiểu quy tắc chuyển đổi kiểu. [Trí nhớ]
- **`this` thay đổi theo cách gọi:** cẩn thận khi tách phương thức khỏi object. [Trí nhớ]
- **Promise không `await`:** lỗi bị bỏ qua hoặc kết quả sai thời điểm. [Suy luận]
- **Chặn event loop:** vòng lặp CPU nặng làm đứng toàn bộ ứng dụng, cả giao diện và máy chủ. [Đã kiểm chứng: đặc điểm chạy một luồng và run-to-completion theo MDN]
- **Phụ thuộc npm:** số lượng gói lớn tạo rủi ro chuỗi cung ứng. Dùng lockfile, kiểm tra gói, và audit định kỳ. [Trí nhớ]
- **Phiên bản Node:** chọn bản LTS được hỗ trợ cho production. [Đã kiểm chứng]
- **Dữ liệu ngoài:** `JSON.parse` và đầu vào người dùng không tin cậy, cần kiểm tra. [Suy luận]

---

# PHẦN H. LỘ TRÌNH HỌC

1. **Cú pháp và kiểu:** `let`, `const`, kiểu nguyên thủy, `===`, ép kiểu.
2. **Hàm và phạm vi:** hàm mũi tên, closure, `this`.
3. **Mảng và object:** phương thức chức năng, destructuring, spread, optional chaining.
4. **Bất đồng bộ:** event loop, Promise, async/await. Đây là phần quan trọng nhất.
5. **Module và Node.js:** ESM, `fs/promises`, npm, `package.json`.
6. **Kiểm thử:** Node test runner hoặc Vitest, mock.
7. **Kiểm tra kiểu:** JSDoc với `@ts-check`, hoặc chuyển sang TypeScript.
8. **Dự án:** một API hoặc CLI nhỏ có lint, test, và CI.

**Nguồn học chính thức:** MDN Web Docs (developer.mozilla.org) và tài liệu Node.js (nodejs.org/docs). [Đã kiểm chứng: trang MDN Event loop đã đọc]

---

# PHẦN I. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| ECMAScript | Đặc tả chuẩn của JavaScript |
| Engine | Phần mềm thực thi JavaScript (V8, SpiderMonkey, JavaScriptCore) |
| Hoisting | Khai báo được đưa lên đầu phạm vi |
| TDZ | Vùng chết tạm thời của `let` và `const` |
| Closure | Hàm ghi nhớ biến của phạm vi tạo ra nó |
| Prototype chain | Chuỗi kế thừa thuộc tính giữa các object |
| Event loop | Vòng lặp điều phối công việc bất đồng bộ |
| Call stack | Ngăn xếp các hàm đang chạy |
| Task (macrotask) | Công việc trong hàng đợi task: timer, I/O, sự kiện |
| Microtask | Công việc ưu tiên cao: Promise, `await` |
| Promise | Đại diện kết quả của thao tác bất đồng bộ |
| async/await | Cú pháp viết Promise như mã đồng bộ |
| Generator | Hàm tạm dừng và tiếp tục bằng `yield` |
| Iterator | Đối tượng có `next()` trả `{ value, done }` |
| Symbol | Giá trị duy nhất, thường làm khóa ẩn |
| Optional chaining | `?.` dừng khi gặp `null` hoặc `undefined` |
| Nullish coalescing | `??` thay khi giá trị là `null` hoặc `undefined` |
| Destructuring | Tách giá trị từ object hoặc mảng |
| ESM / CommonJS | Hai hệ thống module |
| Strict mode | Chế độ nghiêm ngặt, bắt nhiều lỗi |
| Lockfile | Tệp ghim phiên bản phụ thuộc |
| JSDoc | Chú thích kiểu trong comment |

---

# PHẦN J. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## J.1. Nguồn đã đối chiếu trong lần này

- **Đặc tả ECMAScript** (tc39): https://tc39.es/ecma262/ [đã đọc phần mục lục. Ghi nhận tiêu đề "ECMAScript 2027 Language Specification", có mục `using`, iterator helpers, `GroupBy`. Không xác nhận được trạng thái phát hành]
- **Node.js, Previous Releases**: https://nodejs.org/en/about/previous-releases [đã đọc. Ghi nhận: v26 Current (phát hành 2026-05-05); v24 LTS, bản mới nhất v24.22.0 là Active LTS; v22 LTS; v25, v23, v21, v20 đã hết hỗ trợ]
- **MDN, Event loop**: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop [đã đọc. Ghi nhận: run-to-completion, hàng đợi task và microtask, thứ tự rút microtask trước task]

## J.2. Nguồn chưa đối chiếu

- MDN về Promise, async/await, destructuring, optional chaining, Map và Set.
- Tài liệu Node.js về `node:test`, `node:fs/promises`, `node:child_process`, `parseArgs`, `--experimental-test-coverage`.
- Tài liệu Vitest, Jest, ESLint, Prettier, Biome, Playwright.
- Chi tiết TC39 (các giai đoạn đề xuất, và các tính năng cụ thể của ECMAScript mới).

## J.3. Điểm cần xác minh

1. **Trạng thái phát hành ECMAScript 2027:** trang đặc tả hiển thị tiêu đề nhưng chưa rõ đã phát hành. Cần kiểm tra ecma-international.org.
2. **Phiên bản Node.js:** trang previous-releases có cột "Last updated" nhưng giá trị giống ngày hết hỗ trợ. Nên kiểm tra trang download để xác nhận phase LTS của v22 và v24.
3. **Các API của `node:test` và `mock`** thay đổi theo phiên bản Node. Cần kiểm tra tài liệu đúng phiên bản.
4. **Cờ `--experimental-test-coverage`** có thể đã ổn định hoặc đổi tên tùy phiên bản.
5. **Hỗ trợ `Array.prototype.toSorted` và iterator helpers** thay đổi theo môi trường, cần kiểm tra MDN bảng tương thích.
6. **Ví dụ mã chưa được chạy thử.** Nên chạy trên máy trước khi dùng, đặc biệt các đoạn `node:test`, `mock`, `parseArgs`, và cấu hình CI.
7. **So sánh ngôn ngữ** là khái quát, không phải đo lường.

---

**Tóm tắt để tra nhanh:**
- **JavaScript** là ngôn ngữ chạy trong trình duyệt và Node.js, kiểu động, một luồng, theo đặc tả ECMAScript.
- **Dùng `const`/`let`, `===`, và hiểu `this`** là nền tảng.
- **Vòng lặp sự kiện:** chạy xong công việc hiện tại, rút sạch microtask (Promise, await), rồi mới lấy task (timer, I/O).
- **Bất đồng bộ:** Promise và async/await; dùng `Promise.all` cho tác vụ song song.
- **Môi trường:** Node.js v24 là Active LTS cho production; v26 là Current.
- **Kiểm thử:** Node test runner hoặc Vitest; kiểm tra kiểu bằng TypeScript hoặc JSDoc.
- **Cẩn trọng:** ép kiểu ngầm, `this`, Promise không `await`, chặn event loop, và rủi ro chuỗi cung ứng npm.
