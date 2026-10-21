# Ngôn ngữ TypeScript: từ lý thuyết đến lập trình, tự động hóa và kiểm thử

> **Viết cho:** người học TypeScript, lập trình viên JavaScript muốn thêm kiểu tĩnh, và người muốn hiểu vai trò của TypeScript trong dự án web và Node.js.
> **Cách đọc:** theo phần. Phần A và B giải thích TypeScript là gì và kiểu hoạt động ra sao, là nền cho các phần sau.
> **Phiên bản tham chiếu:** trên trang GitHub releases (tra cứu 2026-10-21), bản stable mới nhất được đánh dấu là **TypeScript 7.0.2** (ngày 20/8, theo tài liệu asset). Có bản **7.1 Beta** (20/10) ở trạng thái thử nghiệm. Dòng 5.9 và 6.0 vẫn có thể dùng trong các dự án cũ.
> **Ngày tra cứu:** 2026-10-21.
> **Nhãn nguồn:** `[Đã kiểm chứng]` = đối chiếu trong lần này với tài liệu chính thức hoặc GitHub; `[Trí nhớ]` = kiến thức nền, cần kiểm tra lại; `[Suy luận]` = tôi suy ra hoặc minh họa.

---

## Mục lục

- **Phần A. TypeScript là gì và vì sao tồn tại**
  - 1. Lịch sử và vị trí so với JavaScript
  - 2. TypeScript chạy như thế nào
  - 3. Kiểu tĩnh có tuần tự: kiểm tra lúc biên dịch, xóa kiểu lúc chạy
- **Phần B. Các khái niệm cốt lõi**
  - 4. Cấu hình `tsconfig.json` và chế độ `strict`
  - 5. Kiểu cơ bản và suy luận kiểu
  - 6. Mảng, tuple, object, và kiểu hợp (union, intersection)
  - 7. Interface và type alias
  - 8. Hàm và kiểu tham số
  - 9. Generics
  - 10. Thu hẹp kiểu (narrowing)
  - 11. Discriminated union và kiểm tra đầy đủ
  - 12. Tiện ích kiểu (utility types) và kiểu ánh xạ
  - 13. Lớp và truy cập
  - 14. Bất đồng bộ: Promise và async/await
- **Phần C. Tổ chức mã, module, và môi trường**
  - 15. Module ES và CommonJS
  - 16. Quản lý gói với npm/pnpm và `package.json`
  - 17. Khai báo kiểu cho thư viện (`.d.ts`)
- **Phần D. Công cụ chất lượng mã**
  - 18. Biên dịch, chạy trực tiếp, và công cụ build
  - 19. Lint và định dạng
- **Phần E. Kiểm thử**
  - 20. Kiểm thử đơn vị với Vitest và Jest
  - 21. Kiểm thử kiểu (type-level testing)
  - 22. Kiểm thử giao diện và tích hợp
  - 23. Mock và phụ thuộc
  - 24. Độ bao phủ
- **Phần F. Tự động hóa**
  - 25. Script trong `package.json`
  - 26. Tích hợp liên tục (CI)
  - 27. Đóng gói và phát hành thư viện
- **Phần G. Vai trò của TypeScript trong thực tế**
  - 28. Dùng ở đâu
  - 29. TypeScript so với JavaScript và một số ngôn ngữ khác
  - 30. Các điểm cần cẩn trọng
- **Phần H. Lộ trình học**
- **Phần I. Thuật ngữ**
- **Phần J. Nguồn tham khảo và điểm cần xác minh**

---

# PHẦN A. TYPESCRIPT LÀ GÌ VÀ VÌ SAO TỒN TẠI

## 1. Lịch sử và vị trí so với JavaScript

TypeScript là ngôn ngữ do Microsoft phát triển, là **tập cha** của JavaScript: mọi chương trình JavaScript hợp lệ đều là TypeScript hợp lệ (dù không có kiểu). TypeScript thêm **hệ thống kiểu tĩnh** để phát hiện lỗi trước khi chạy. [Trí nhớ]

Vấn đề mà TypeScript giải quyết: JavaScript linh hoạt, nhưng ở các dự án lớn, lỗi kiểu (gọi phương thức không tồn tại, truyền sai tham số, quên xử lý `null`) chỉ lộ ra lúc chạy, thường ở môi trường người dùng. TypeScript chuyển một phần lỗi này lên giai đoạn biên dịch. [Suy luận]

## 2. TypeScript chạy như thế nào

```
Mã TypeScript (.ts, .tsx)
    │  tsc kiểm tra kiểu, rồi xóa chú thích kiểu
    ▼
Mã JavaScript (.js)
    │  chạy trên Node.js, trình duyệt, hoặc runtime khác
    ▼
Kết quả
```

- TypeScript **không chạy trực tiếp** trên Node.js hay trình duyệt. Nó được biên dịch thành JavaScript.
- Kiểu chỉ tồn tại lúc phát triển và biên dịch. Khi chạy, mã JavaScript không còn thông tin kiểu.
- Có thể kiểm tra kiểu mà không phát ra tệp: `tsc --noEmit`.

**Các cách chạy nhanh:**

```sh
npx tsc --init            # tạo tsconfig.json
npx tsc                   # biên dịch dự án
npx tsc --noEmit          # chỉ kiểm tra kiểu
node dist/index.js        # chạy kết quả JavaScript
```

Một số công cụ như `tsx` hoặc Node.js hiện đại có thể chạy trực tiếp tệp TypeScript, tùy phiên bản và cấu hình. [Trí nhớ; cần kiểm tra theo môi trường bạn dùng]

## 3. Kiểu tĩnh: kiểm tra lúc biên dịch, xóa kiểu lúc chạy

**Điểm then chốt:** kiểu là công cụ **tĩnh**, nên không tạo ra chi phí lúc chạy. Đổi lại, kiểu không bảo vệ được dữ liệu đến từ bên ngoài lúc chạy, ví dụ JSON từ API. Cần kiểm tra lúc chạy cho dữ liệu không tin cậy. [Suy luận]

```ts
function cong(a: number, b: number): number {
  return a + b;
}

cong(1, 2);        // ok
cong("1", 2);      // lỗi biên dịch: string không gán được cho number
```

---

# PHẦN B. CÁC KHÁI NIỆM CỐT LÕI

## 4. Cấu hình `tsconfig.json` và chế độ `strict`

Tệp `tsconfig.json` điều khiển cách TypeScript kiểm tra và biên dịch dự án:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "strict": true,
    "outDir": "dist",
    "rootDir": "src",
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

**`strict: true`** bật một nhóm kiểm tra nghiêm ngặt. Trang tài liệu chính thức nêu rằng `strict` bật nhiều hành vi kiểm tra để tăng độ đảm bảo của chương trình, và có thể tắt từng thành phần. [Đã kiểm chứng]

Nhóm `strict` gồm các cờ tiêu biểu (theo kiến thức nền, nên kiểm tra danh sách theo phiên bản): `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitThis`, `useUnknownInCatchVariables`, `alwaysStrict`. [Trí nhớ]

**Khuyến nghị:** với dự án mới, bật `strict: true` ngay từ đầu. Với dự án cũ, bật dần bằng cách tắt từng cờ gây lỗi lớn, rồi sửa dần. [Suy luận]

**Lưu ý:** trang `strict` nói rằng các phiên bản tương lai có thể thêm kiểm tra nghiêm ngặt hơn dưới cùng cờ này, nên nâng cấp TypeScript có thể làm lộ thêm lỗi kiểu. [Đã kiểm chứng]

## 5. Kiểu cơ bản và suy luận kiểu

```ts
let ten: string = "An";
let tuoi = 25;              // suy luận: number
let dangHoc = true;         // suy luận: boolean
let khong: null = null;
let chuaXacDinh: undefined = undefined;

let ghiChu: unknown = "bất kỳ";   // an toàn hơn any: phải thu hẹp trước khi dùng
let bat_ky: any = 1;              // tránh: tắt kiểm tra kiểu
```

**Suy luận kiểu:** TypeScript tự suy luận kiểu từ giá trị khởi tạo. Không cần chú thích mọi biến. [Đã kiểm chứng: handbook về narrowing và kiểu cơ bản]

**`unknown` và `any`:**
- `any` tắt kiểm tra, lan truyền lỗi tiềm tàng.
- `unknown` bắt buộc thu hẹp trước khi dùng. Ưu tiên khi nhận dữ liệu không rõ kiểu. [Trí nhớ]

## 6. Mảng, tuple, object, và kiểu hợp

```ts
const so: number[] = [1, 2, 3];
const ten: Array<string> = ["An", "Bình"];

const toaDo: [number, number] = [10, 20];   // tuple: độ dài và kiểu từng vị trí cố định

// Kiểu hợp (union): một trong các kiểu
let id: string | number = "abc";
id = 42;

// Kiểu giao (intersection): kết hợp các kiểu
type CoTen = { ten: string };
type CoTuoi = { tuoi: number };
type NguoiDung = CoTen & CoTuoi;   // { ten: string; tuoi: number }
```

**Thuộc tính tùy chọn và chỉ đọc:**

```ts
type SanPham = {
  readonly ma: string;      // không gán lại được sau khi khởi tạo
  ten: string;
  moTa?: string;            // có thể không có
};
```

## 7. Interface và type alias

Cả hai đều mô tả hình dạng của object:

```ts
interface SinhVien {
  mssv: string;
  ten: string;
  diem?: number;
}

type DiemSo = {
  mon: string;
  so: number;
};
```

**Khác biệt chính** (theo kiến thức nền): [Trí nhớ]
- `interface` mở rộng được bằng khai báo gộp (declaration merging) và `extends`.
- `type` linh hoạt hơn: biểu diễn được union, tuple, kiểu ánh xạ.
- Trong thực tế, cả hai đều dùng được. Chọn nhất quán trong dự án.

## 8. Hàm và kiểu tham số

```ts
function chao(ten: string, ngu: string = "xin chào"): string {
  return `${ngu}, ${ten}!`;
}

const nhan = (x: number): number => x * 2;

// Tham số tùy chọn và giá trị trả về không có
function ghi(log: string): void {
  console.log(log);
}

// Tham số rest
function tong(...so: number[]): number {
  return so.reduce((a, b) => a + b, 0);
}
```

**Kiểu hàm:**

```ts
type Phep = (a: number, b: number) => number;
const cong: Phep = (a, b) => a + b;
```

**Quy tắc:** nên khai báo kiểu tham số của hàm công khai. Kiểu trả về có thể để TypeScript suy luận, nhưng khai báo rõ giúp đọc và kiểm soát API tốt hơn. [Suy luận]

## 9. Generics

Generics cho phép viết hàm và kiểu dùng cho nhiều kiểu, vẫn giữ kiểm tra:

```ts
function dau<T>(mang: T[]): T | undefined {
  return mang[0];
}

const so = dau([1, 2, 3]);     // T = number
const ten = dau(["An"]);       // T = string

// Ràng buộc kiểu
function lay<T extends { id: string }>(mang: T[], id: string): T | undefined {
  return mang.find((x) => x.id === id);
}
```

**Lợi ích:** tránh viết lặp cho từng kiểu, và giữ được kiểu của kết quả. [Suy luận]

## 10. Thu hẹp kiểu (narrowing)

Narrowing là cách TypeScript **thu hẹp** kiểu của một giá trị dựa trên kiểm tra lúc chạy trong luồng điều khiển. Tài liệu chính thức nêu nhiều kỹ thuật: [Đã kiểm chứng]

**1. `typeof`:**

```ts
function dem(x: number | string): number {
  if (typeof x === "number") {
    return x;               // x: number
  }
  return x.length;          // x: string
}
```

Lưu ý từ tài liệu: `typeof null` là `"object"`, nên với kiểu object vẫn cần kiểm tra `null` riêng. [Đã kiểm chứng]

**2. Truthiness:** `0`, `NaN`, `""`, `0n`, `null`, `undefined` là giá trị giả (falsy). Tài liệu cảnh báo: bọc cả thân hàm trong `if (chuoi)` có thể bỏ sót trường hợp chuỗi rỗng. [Đã kiểm chứng]

**3. So sánh bằng:** `!= null` loại bỏ cả `null` và `undefined`. [Đã kiểm chứng]

```ts
interface Hop {
  gia: number | null | undefined;
}
function nhanHe(h: Hop, he: number) {
  if (h.gia != null) {
    h.gia *= he;            // h.gia: number
  }
}
```

**4. Toán tử `in`:** `"bay" in dongVat` thu hẹp theo sự có mặt của thuộc tính. [Đã kiểm chứng]

**5. `instanceof`:** kiểm tra chuỗi prototype, thu hẹp về lớp tương ứng. [Đã kiểm chứng]

**6. Hàm bảo vệ kiểu (user-defined type guard):** [Đã kiểm chứng]

```ts
interface Ca { boiLoi(): void }
interface Chim { bay(): void }

function laCa(x: Ca | Chim): x is Ca {
  return (x as Ca).boiLoi !== undefined;
}

function dieuKhien(x: Ca | Chim) {
  if (laCa(x)) {
    x.boiLoi();             // x: Ca
  } else {
    x.bay();                // x: Chim
  }
}
```

**Lưu ý:** hàm bảo vệ kiểu là lời hứa của lập trình viên, TypeScript tin vào nó. Nếu logic sai, kiểu sẽ sai mà không báo lỗi. [Suy luận]

## 11. Discriminated union và kiểm tra đầy đủ

Một union mà mỗi thành viên có thuộc tính chung với kiểu literal, gọi là discriminated union. TypeScript có thể thu hẹp toàn bộ union qua thuộc tính đó: [Đã kiểm chứng]

```ts
interface HinhTron { loai: "tron"; banKinh: number }
interface HinhVuong { loai: "vuong"; canh: number }
type Hinh = HinhTron | HinhVuong;

function dienTich(h: Hinh): number {
  switch (h.loai) {
    case "tron":
      return Math.PI * h.banKinh ** 2;
    case "vuong":
      return h.canh ** 2;
  }
}
```

**Kiểm tra đầy đủ bằng `never`:**

```ts
function xuLy(h: Hinh): number {
  switch (h.loai) {
    case "tron":
      return Math.PI * h.banKinh ** 2;
    case "vuong":
      return h.canh ** 2;
    default: {
      const kiemTra: never = h;   // lỗi biên dịch nếu thêm loại mới mà chưa xử lý
      return kiemTra;
    }
  }
}
```

Đây là kỹ thuật quan trọng: khi thêm một hình mới vào union, TypeScript báo lỗi ở mọi nơi chưa xử lý. [Đã kiểm chứng: tài liệu nêu việc dùng `never` để kiểm tra đầy đủ]

## 12. Tiện ích kiểu (utility types) và kiểu ánh xạ

Các tiện ích có sẵn: [Trí nhớ, tên và công dụng chuẩn]

| Tiện ích | Tác dụng | Ví dụ |
|---|---|---|
| `Partial<T>` | Mọi thuộc tính thành tùy chọn | `Partial<SinhVien>` |
| `Required<T>` | Mọi thuộc tính thành bắt buộc | |
| `Readonly<T>` | Mọi thuộc tính chỉ đọc | |
| `Pick<T, K>` | Chọn một số thuộc tính | `Pick<SinhVien, "mssv" \| "ten">` |
| `Omit<T, K>` | Bỏ một số thuộc tính | `Omit<SinhVien, "diem">` |
| `Record<K, V>` | Object với khóa K và giá trị V | `Record<string, number>` |
| `ReturnType<F>` | Kiểu trả về của hàm | |

**Kiểu ánh xạ tự viết:**

```ts
type Co<T> = { [K in keyof T]: T[K] | null };
```

## 13. Lớp và truy cập

```ts
class TaiKhoan {
  private soDu = 0;                     // chỉ trong lớp
  protected loai = "thường";            // lớp và lớp con
  public readonly ma: string;           // mặc định là public

  constructor(ma: string) {
    this.ma = ma;
  }

  nap(so: number): void {
    if (so <= 0) throw new Error("số tiền phải dương");
    this.soDu += so;
  }

  get so(): number {
    return this.soDu;
  }
}
```

**Lưu ý:** `private` của TypeScript chỉ là kiểm tra lúc biên dịch. Nó không bảo vệ lúc chạy. Dùng `#soDu` (trường riêng của JavaScript) nếu cần bảo vệ thật sự. [Trí nhớ]

## 14. Bất đồng bộ: Promise và async/await

```ts
interface KetQua { ten: string }

async function layDuLieu(url: string): Promise<KetQua> {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error(`HTTP ${res.status}`);
  }
  const data: unknown = await res.json();   // dữ liệu ngoài: bắt đầu là unknown
  return kiemTraKetQua(data);                // thu hẹp trước khi trả về
}
```

**Quy tắc:** `res.json()` trả về `any` trong thư viện chuẩn của nhiều môi trường. Gán vào `unknown` rồi thu hẹp bằng kiểm tra runtime (hoặc thư viện xác thực như Zod) là cách an toàn. [Trí nhớ]

---

# PHẦN C. TỔ CHỨC MÃ, MODULE, VÀ MÔI TRƯỜNG

## 15. Module ES và CommonJS

```ts
// tiện ích
export function cong(a: number, b: number): number {
  return a + b;
}

// nơi dùng
import { cong } from "./toan.js";
```

**Lưu ý về đuôi tệp:** với chế độ module hiện đại của Node.js (ESM), đường dẫn import thường cần đuôi `.js` ngay cả khi tệp nguồn là `.ts`. TypeScript hiểu điều này. [Trí nhớ; kiểm tra với cấu hình `module` và `moduleResolution`]

**Hai hệ module thường gặp:**
- **ESM** (`import`/`export`): chuẩn hiện đại.
- **CommonJS** (`require`/`module.exports`): cũ hơn, còn nhiều trong hệ sinh thái Node.js.

## 16. Quản lý gói với npm/pnpm và package.json

```json
{
  "name": "congcu",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit"
  },
  "devDependencies": {
    "typescript": "^7.0.2",
    "vitest": "^3.0.0"
  }
}
```

[Phiên bản `typescript` ở đây chỉ để minh họa, nên dùng phiên bản đã kiểm tra]

```sh
npm install -D typescript
npm install -D vitest
npm run build
```

**Gợi ý:** ghim phiên bản TypeScript trong dự án, và khóa phụ thuộc bằng tệp lockfile (`package-lock.json` hoặc `pnpm-lock.yaml`). [Suy luận]

## 17. Khai báo kiểu cho thư viện (.d.ts)

Tệp `.d.ts` chỉ chứa khai báo kiểu, không chứa mã chạy. Dùng khi thư viện JavaScript không có kiểu:

```ts
// types/thu-vien-cu.d.ts
declare module "thu-vien-cu" {
  export function chao(ten: string): string;
}
```

**Thư viện có sẵn kiểu:** nhiều gói đi kèm kiểu; các gói cũ có thể dùng `@types/ten-goi` từ DefinitelyTyped. [Trí nhớ]

---

# PHẦN D. CÔNG CỤ CHẤT LƯỢNG MÃ

## 18. Biên dịch, chạy trực tiếp, và công cụ build

| Công cụ | Chức năng | Ghi chú |
|---|---|---|
| `tsc` | Trình biên dịch chính thức | Kiểm tra kiểu và phát ra JavaScript |
| `tsx` | Chạy trực tiếp tệp TypeScript | Tiện cho script và phát triển [Trí nhớ] |
| `esbuild`, `swc` | Biên dịch và đóng gói nhanh | Thường không kiểm tra kiểu; cần `tsc --noEmit` riêng [Trí nhớ] |
| `vite` | Máy chủ phát triển và đóng gói frontend | Phổ biến cho dự án web [Trí nhớ] |

**Nguyên tắc:** công cụ biên dịch nhanh thường **bỏ qua kiểm tra kiểu**. Trong CI nên có bước `tsc --noEmit` riêng. [Suy luận]

## 19. Lint và định dạng

- **ESLint** với plugin TypeScript: phát hiện mẫu mã đáng ngờ và quy tắc phong cách. [Trí nhớ]
- **Prettier** hoặc **Biome**: định dạng tự động. [Trí nhớ]

```sh
npx eslint .
npx prettier --check .
```

**Gợi ý:** cấm `any` bằng quy tắc lint để kiểm soát chất lượng kiểu. [Suy luận]

---

# PHẦN E. KIỂM THỬ

## 20. Kiểm thử đơn vị với Vitest và Jest

**Vitest** là công cụ kiểm thử hiện đại, tương thích cú pháp Jest, chạy nhanh với TypeScript:

```ts
// cong.test.ts
import { describe, it, expect } from "vitest";
import { cong } from "./toan.js";

describe("cong", () => {
  it("cộng hai số dương", () => {
    expect(cong(2, 3)).toBe(5);
  });

  it("xử lý số âm", () => {
    expect(cong(-1, 4)).toBe(3);
  });
});
```

```sh
npx vitest run
npx vitest              # chế độ theo dõi
```

**Jest** là công cụ lâu đời hơn, nhiều dự án cũ dùng. Cần cấu hình thêm để xử lý TypeScript (ví dụ `ts-jest` hoặc transform của SWC). [Trí nhớ]

**Table-driven (tương tự Go):**

```ts
it.each([
  [2, 3, 5],
  [-1, 4, 3],
  [0, 7, 7],
])("cong(%i, %i) = %i", (a, b, ky) => {
  expect(cong(a, b)).toBe(ky);
});
```

## 21. Kiểm thử kiểu (type-level testing)

Kiểu cũng có thể được kiểm thử, để đảm bảo kiểu đúng với ý định:

```ts
import { expectTypeOf } from "vitest";

expectTypeOf(cong).parameters.toEqualTypeOf<[number, number]>();
expectTypeOf(cong).returns.toEqualTypeOf<number>();
```

Ngoài ra, `tsc --noEmit` là kiểm tra kiểu toàn dự án. Đây là bước không thể thiếu trong CI. [Đã kiểm chứng: `--noEmit` được nêu trong tài liệu TypeScript; `expectTypeOf` là API của Vitest, trí nhớ]

## 22. Kiểm thử giao diện và tích hợp

- **Playwright**, **Cypress**: kiểm thử đầu cuối trên trình duyệt. [Trí nhớ]
- **Testing Library**: kiểm thử component theo cách người dùng tương tác. [Trí nhớ]
- **Supertest** hoặc tương tự: kiểm thử API HTTP Node.js. [Trí nhớ]

## 23. Mock và phụ thuộc

Vitest có `vi.fn` và `vi.mock`:

```ts
import { vi, it, expect } from "vitest";

it("gọi API một lần", async () => {
  const laySo = vi.fn().mockResolvedValue(100);
  const ket = await laySo();
  expect(ket).toBe(100);
  expect(laySo).toHaveBeenCalledOnce();
});
```

**Nguyên tắc:** thay phụ thuộc bên ngoài (mạng, đồng hồ, cơ sở dữ liệu) bằng giả lập để test nhanh và ổn định. Mục tiêu tương tự `unittest.mock` ở Python và interface giả ở Go. [Suy luận]

## 24. Độ bao phủ

```sh
npx vitest run --coverage
```

Cần cài gói cung cấp coverage (ví dụ `@vitest/coverage-v8`). [Trí nhớ]

**Cách hiểu:** độ bao phủ cho biết dòng nào đã chạy. Không đảm bảo đúng logic. Kết hợp với kiểm tra kiểu và test biên. [Suy luận]

---

# PHẦN F. TỰ ĐỘNG HÓA

## 25. Script trong package.json

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "lint": "eslint .",
    "test": "vitest run",
    "build": "tsc",
    "ci": "npm run typecheck && npm run lint && npm run test && npm run build"
  }
}
```

```sh
npm run ci
```

## 26. Tích hợp liên tục (CI)

Pipeline điển hình:

```
1. Cài phụ thuộc đúng lockfile:   npm ci
2. Kiểm tra kiểu:                  tsc --noEmit
3. Lint:                           eslint .
4. Kiểm thử:                       vitest run --coverage
5. Biên dịch:                      tsc
6. Đóng gói hoặc kiểm tra gói:     npm pack --dry-run
```

**Ví dụ GitHub Actions** (minh họa, cần điều chỉnh phiên bản Node):

```yaml
name: ci
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run ci
```

[Suy luận; kiểm tra phiên bản Node và action]

## 27. Đóng gói và phát hành thư viện

- Khai báo `main`, `types`, và `exports` trong `package.json` để công cụ tìm đúng kiểu.
- Biên dịch ra `dist/` kèm tệp `.d.ts`.
- Kiểm tra gói trước khi phát hành: `npm pack --dry-run`. [Trí nhớ]
- Phát hành: `npm publish` sau khi kiểm tra kỹ và có phiên bản rõ ràng. [Trí nhớ]

---

# PHẦN G. VAI TRÒ CỦA TYPESCRIPT TRONG THỰC TẾ

## 28. Dùng ở đâu

| Lĩnh vực | Ví dụ | Lý do phù hợp |
|---|---|---|
| Frontend web | React, Vue, Angular, Svelte | Kiểm tra props, state, API |
| Backend Node.js | API, dịch vụ, GraphQL | Chia sẻ kiểu với frontend |
| Công cụ dòng lệnh | CLI, script build | Dễ bảo trì khi lớn |
| Thư viện mã nguồn mở | Gói npm | Cung cấp kiểu cho người dùng |
| Ứng dụng di động, desktop | React Native, Electron | Tái sử dụng kiến thức web |

[Suy luận và kiến thức nền; không đầy đủ]

## 29. TypeScript so với JavaScript và một số ngôn ngữ khác

| Tiêu chí | TypeScript | JavaScript | Python | Go |
|---|---|---|---|---|
| Kiểu | Tĩnh (tùy chọn chặt) | Động | Động, có gợi ý | Tĩnh |
| Chạy | Biên dịch sang JS | Trực tiếp | Bytecode | Mã máy |
| Kiểm tra lúc chạy | Không (xóa kiểu) | Không | Không (gợi ý) | Có (kiểu tĩnh) |
| Hệ sinh thái web | Rất mạnh | Gốc | Ít hơn cho frontend | Ít hơn |
| Đồng thời | Event loop, Promise | Event loop | asyncio | Goroutine |

[Suy luận; so sánh khái quát]

## 30. Các điểm cần cẩn trọng

- **Kiểu không bảo vệ dữ liệu lúc chạy.** Với JSON và input người dùng, cần kiểm tra runtime. [Suy luận]
- **Lạm dụng `any`** làm mất lợi ích của TypeScript. Dùng `unknown` và thu hẹp. [Trí nhớ]
- **`as` là khẳng định, không phải kiểm tra.** `x as Foo` không đổi dữ liệu, chỉ bảo TypeScript tin. [Trí nhớ]
- **Công cụ nhanh bỏ qua kiểm tra kiểu.** Luôn có `tsc --noEmit` trong CI. [Suy luận]
- **Phiên bản thay đổi nhanh.** TypeScript 7 là bản lớn, cần kiểm tra ghi chú phát hành khi nâng cấp. [Đã kiểm chứng: GitHub releases ghi nhận TS 7.0.2 là bản ổn định mới nhất trong lần tra cứu]
- **Cấu hình `strict`** có thể làm lộ nhiều lỗi ở dự án cũ. Nâng cấp dần. [Suy luận]

---

# PHẦN H. LỘ TRÌNH HỌC

1. **JavaScript nền tảng:** biến, hàm, object, mảng, Promise. TypeScript xây trên JavaScript.
2. **Kiểu cơ bản:** `string`, `number`, `boolean`, mảng, object, union, `unknown`.
3. **Interface và type alias:** mô hình hóa dữ liệu của một miền nhỏ.
4. **Narrowing và discriminated union:** viết hàm xử lý union có kiểm tra đầy đủ.
5. **Generics:** viết một hàm tiện ích dùng cho nhiều kiểu.
6. **Async:** Promise, async/await, xử lý lỗi.
7. **Kiểm thử:** Vitest, `it.each`, mock.
8. **Dự án:** một CLI hoặc API nhỏ, bật `strict`, có CI chạy `tsc --noEmit`, lint, và test.

**Nguồn học chính thức:** TypeScript Handbook (typescriptlang.org/docs/handbook). [Đã kiểm chứng: trang handbook narrowing đã đọc]

---

# PHẦN I. THUẬT NGỮ

| Thuật ngữ | Giải thích |
|---|---|
| Type annotation | Chú thích kiểu tường minh |
| Type inference | Suy luận kiểu tự động |
| Union / Intersection | Kiểu hợp (một trong nhiều) / giao (kết hợp) |
| Literal type | Kiểu là một giá trị cụ thể, ví dụ `"tron"` |
| Narrowing | Thu hẹp kiểu theo kiểm tra |
| Type guard | Hàm hoặc biểu thức giúp thu hẹp kiểu |
| Discriminated union | Union có thuộc tính phân biệt kiểu literal |
| Exhaustiveness | Kiểm tra đã xử lý đủ mọi nhánh của union |
| Generic | Kiểu tham số hóa |
| Utility type | Kiểu có sẵn như `Partial`, `Pick`, `Omit` |
| `unknown` | Kiểu an toàn cho giá trị chưa biết, phải thu hẹp |
| `any` | Tắt kiểm tra kiểu |
| `never` | Kiểu không có giá trị, dùng để kiểm tra đầy đủ |
| Declaration file (`.d.ts`) | Tệp chỉ chứa khai báo kiểu |
| `strict` | Nhóm cờ kiểm tra nghiêm ngặt trong tsconfig |
| Type erasure | Xóa kiểu khi biên dịch sang JavaScript |
| Structural typing | Kiểu so khớp theo cấu trúc, không theo tên |
| ESM / CommonJS | Hai hệ thống module |
| Lint | Kiểm tra mã để tìm lỗi và vi phạm phong cách |
| Type-level testing | Kiểm thử đúng đắn của kiểu |

---

# PHẦN J. NGUỒN THAM KHẢO VÀ ĐIỂM CẦN XÁC MINH

## J.1. Nguồn đã đối chiếu trong lần này

- **TypeScript Handbook: Narrowing**: https://www.typescriptlang.org/docs/handbook/2/narrowing.html [đã đọc: typeof, truthiness, equality, `in`, `instanceof`, discriminated union, user-defined type guard]
- **TSConfig: strict**: https://www.typescriptlang.org/tsconfig/strict.html [đã đọc một phần: mô tả chung; danh sách cờ cụ thể không có trong nội dung đã lấy]
- **TypeScript Release Notes**: https://www.typescriptlang.org/docs/handbook/release-notes/overview.html [trang trả về rỗng, không dùng được]
- **GitHub releases của TypeScript**: https://github.com/microsoft/TypeScript/releases [đã đọc: bản ổn định mới nhất được đánh dấu là 7.0.2 (ngày 20/8, theo asset); 7.1 Beta (20/10) là pre-release; 6.0.3, 5.9.3 là các bản ổn định gần đó]

## J.2. Nguồn chưa đối chiếu

- Tài liệu chính thức của Vitest, ESLint, Prettier, Biome, Vite, tsx.
- Tài liệu về ES modules trong Node.js và `moduleResolution`.
- Chi tiết các tiện ích kiểu như `Partial`, `Pick`, `Omit`, và `ReturnType`.
- Ghi chú phát hành của TypeScript 7 (phạm vi thay đổi, có phải bản viết lại bằng Go hay không).

## J.3. Điểm cần xác minh

1. **Phiên bản TypeScript 7:** GitHub cho thấy 7.0.2 là bản mới nhất được đánh dấu. Cần đọc ghi chú phát hành để biết thay đổi lớn, đặc biệt nếu liên quan đến trình biên dịch mới. Tài liệu này không mô tả chi tiết các thay đổi đó.
2. **Danh sách đầy đủ các cờ trong `strict`:** tài liệu được đọc không liệt kê. Danh sách trong mục 4 là kiến thức nền.
3. **Mặc định của `strict` khi tạo dự án mới:** cần kiểm tra theo `tsc --init` của phiên bản đang dùng.
4. **Cấu hình `module` và `moduleResolution`** thay đổi theo phiên bản, đặc biệt với ESM. Mẫu `tsconfig` trong tài liệu chỉ để minh họa.
5. **Ví dụ mã chưa được chạy thử.** Nên kiểm tra trên máy trước khi dùng, nhất là phần Vitest, `expectTypeOf`, và cấu hình CI.
6. **Phiên bản Node.js và các action trong CI** cần điều chỉnh theo thực tế.
7. **Các so sánh ngôn ngữ** là khái quát, không phải đo lường.

---

**Tóm tắt để tra nhanh:**
- **TypeScript** = JavaScript có hệ thống kiểu tĩnh, biên dịch sang JavaScript, xóa kiểu khi chạy.
- **Bật `strict: true`** cho dự án mới; tránh `any`, dùng `unknown` và thu hẹp.
- **Narrowing** (`typeof`, `in`, `instanceof`, discriminated union, type guard) là kỹ năng cốt lõi.
- **Kiểm tra kiểu** bằng `tsc --noEmit` trong CI; công cụ build nhanh thường không kiểm tra kiểu.
- **Kiểm thử** bằng Vitest, mock phụ thuộc, và kiểm thử kiểu khi cần.
- **Cẩn trọng:** dữ liệu ngoài cần kiểm tra lúc chạy, `as` không phải kiểm tra, và phiên bản mới (TypeScript 7) cần đọc ghi chú phát hành.
