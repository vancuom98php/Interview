<a id="mục-lục"></a>
# Backend — JavaScript Core

> Kiểu dữ liệu · scope/hoisting · closure · `this` · prototype · `==` vs `===` · event loop ·
> async/await & Promise · ES6+ (let/const, arrow, destructuring, spread).
> (Câu hỏi về Node.js/Express/NestJS nằm ở [nodejs.md](./nodejs.md).)

## 📑 Mục lục

1. [Phân biệt TypeScript và JavaScript](#q1)
   - 1.1. [Các kiểu dữ liệu trong TypeScript](#q1-1)
   - 1.2. [Phân biệt object, type và interface trong TypeScript](#q1-2)
   - 1.3. [Generics (generic types) trong TypeScript](#q1-3)
   - 1.4. [Type guards (thu hẹp kiểu) trong TypeScript](#q1-4)
   - 1.5. [Conditional types trong TypeScript](#q1-5)
   - 1.6. [Advanced types: keyof, mapped types & utility types](#q1-6)
2. [ES5, ES6 và ES7 khác nhau những gì?](#q2)
   - 2.1. [Babel là gì?](#q2-1)
   - 2.2. [Webpack là gì?](#q2-2)
   - 2.3. [Phân biệt Webpack và Vite](#q2-3)
3. [Phân biệt tham trị (pass by value) và tham chiếu (pass by reference) trong JavaScript](#q3)
   - 3.1. [Mutable và Immutable trong JavaScript](#q3-1)
4. [Truthy và Falsy trong JavaScript/TypeScript](#q4)
5. [Phân biệt `for...in` và `for...of` trong JavaScript](#q5)
6. [Map và Set trong JavaScript](#q6)
7. [Phân biệt anonymous function và named function trong JavaScript](#q7)
   - 7.1. [Phân biệt arrow function và regular function](#q7-1)
8. [Vòng đời thực thi của JavaScript (JS lifecycle)](#q8)
   - 8.1. [Các thành phần của cơ chế bất đồng bộ: Execution Context, Call Stack, Task Queue, Web APIs, Event Loop](#q8-1)
9. [Bất đồng bộ (asynchronous) trong JavaScript là gì?](#q9)
   - 9.1. [Callback là gì? Callback hell?](#q9-1)
   - 9.2. [Promise là gì? States, then/catch/finally, chaining](#q9-2)
   - 9.3. [Phân biệt Promise.all, Promise.race, Promise.allSettled và Promise.any](#q9-3)
   - 9.4. [async/await là gì?](#q9-4)
   - 9.5. [Phân biệt callback, Promise và async/await](#q9-5)
   - 9.6. [Phân biệt microtask và macrotask](#q9-6)
   - 9.7. [RxJS là gì?](#q9-7)
10. [Proxy trong JavaScript là gì?](#q10)
    - 10.1. [Reflect dùng để làm gì? Vì sao đi cùng Proxy?](#q10-1)
    - 10.2. [Vue 3 dùng Proxy cho reactivity như thế nào?](#q10-2)

---

<a id="q1"></a>
## 1. Phân biệt TypeScript và JavaScript

### 📖 Giới thiệu kiến thức

**JavaScript (JS)** là ngôn ngữ kịch bản (scripting) **động, thông dịch**, chạy trực tiếp trên trình
duyệt và Node.js. Kiểu dữ liệu được xác định **lúc runtime** (dynamic typing) — biến có thể đổi kiểu
tự do, lỗi kiểu chỉ lộ ra khi chạy.

**TypeScript (TS)** là **superset (tập cha) của JavaScript** do Microsoft phát triển: **mọi code JS
hợp lệ đều là TS hợp lệ**, nhưng TS bổ sung **static typing (kiểu tĩnh)** và nhiều tính năng cấp ngôn
ngữ. Điểm cốt lõi: **trình duyệt và Node.js KHÔNG chạy được TS trực tiếp** — TS phải được **biên dịch
(compile/transpile) về JavaScript** bằng `tsc` (TypeScript Compiler) hoặc các bundler (Vite, esbuild,
Babel) trước khi chạy.

```ts
// TypeScript — có kiểu tĩnh
function sum(a: number, b: number): number {
  return a + b;
}
sum(1, "2"); // ❌ Lỗi NGAY lúc compile, trước khi chạy

// JavaScript tương đương (sau khi biên dịch — kiểu bị "xóa")
function sum(a, b) {
  return a + b;
}
sum(1, "2"); // "12" — chạy bình thường, bug âm thầm
```

**Đặc điểm quan trọng — Type Erasure (xóa kiểu):** mọi annotation kiểu (`: number`, `interface`,
`type`) **chỉ tồn tại lúc compile** để kiểm tra; khi biên dịch ra JS chúng **bị xóa hoàn toàn**. Nghĩa
là TypeScript **không** giúp ích gì lúc runtime — không tự validate dữ liệu từ API/người dùng (vẫn phải
tự validate). TS là **công cụ cho lập trình viên (developer experience)**, không phải tính năng runtime.

**Những gì TypeScript thêm so với JavaScript:**
- **Static typing:** `string`, `number`, `boolean`, `union` (`string | number`), `enum`, generics...
- **interface / type:** mô tả "hình dạng" object, hợp đồng (contract) giữa các phần.
- **Hỗ trợ IDE mạnh:** autocomplete, gợi ý, refactor an toàn, "go to definition" chính xác.
- **Bắt lỗi sớm** (compile-time) thay vì lúc runtime → giảm bug khi codebase/đội lớn.
- Tính năng nâng cao: `readonly`, access modifier (`private/public`), decorators (NestJS dùng nhiều).

**So sánh nhanh:**

| Tiêu chí | **JavaScript** | **TypeScript** |
|---|---|---|
| **Kiểu (typing)** | Động (dynamic), kiểm tra lúc **runtime** | Tĩnh (static), kiểm tra lúc **compile** |
| **Thực thi** | Chạy trực tiếp (browser/Node) | Phải **compile về JS** mới chạy |
| **Bắt lỗi kiểu** | Khi chạy (muộn) | Khi biên dịch (sớm) |
| **Quan hệ** | Ngôn ngữ gốc | **Superset** của JS (thêm tính năng) |
| **Runtime** | Bản thân là runtime | **Type bị xóa** — không tồn tại lúc chạy |
| **IDE/Tooling** | Hạn chế (không biết kiểu) | Rất mạnh (autocomplete, refactor) |
| **Phù hợp** | Script nhỏ, prototype nhanh | Dự án lớn, nhiều người, dài hạn |

**Khi nào dùng cái nào:** dự án nhỏ/script nhanh/prototype → JS đủ và gọn. Dự án **lớn, nhiều người,
vòng đời dài** → TS đáng giá: kiểu là "tài liệu sống", refactor an toàn, bắt lỗi sớm. Đánh đổi: TS cần
**bước build** và chi phí học/viết type.

### 🎯 Tóm tắt trả lời phỏng vấn

- **JavaScript** là ngôn ngữ **động, thông dịch**, kiểm tra kiểu lúc **runtime**, chạy trực tiếp trên
  browser/Node. **TypeScript** là **superset của JS** (mọi JS hợp lệ là TS hợp lệ) thêm **static
  typing** và phải **compile về JS** mới chạy được.
- **Khác cốt lõi:** TS **bắt lỗi kiểu lúc compile** (sớm) còn JS chỉ lộ lúc runtime (muộn).
- **Type Erasure:** kiểu trong TS **bị xóa khi biên dịch** → không tồn tại lúc runtime, không tự
  validate dữ liệu ngoài → vẫn phải validate input thủ công.
- **Lợi ích TS:** IDE mạnh (autocomplete/refactor), kiểu như tài liệu sống, ít bug ở codebase lớn —
  đánh đổi là cần bước build và chi phí viết type.
- **Chọn:** script nhỏ/prototype → JS; dự án lớn, nhiều người, dài hạn → TS.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Các kiểu dữ liệu trong TypeScript

#### 📖 Giới thiệu kiến thức

TypeScript bổ sung một **hệ thống kiểu (type system)** giàu hơn JavaScript. Có thể chia kiểu dữ liệu TS
thành **4 nhóm** để dễ nhớ: ① kiểu nguyên thủy (giống JS), ② kiểu đặc biệt riêng của TS, ③ kiểu
object/cấu trúc, ④ kiểu nâng cao (kết hợp kiểu).

**① Kiểu nguyên thủy (Primitive types)** — y hệt JavaScript, gồm **7** kiểu:

| Kiểu | Mô tả | Ví dụ |
|---|---|---|
| `string` | Chuỗi | `let s: string = "hi"` |
| `number` | Số (cả nguyên & thực, **không** tách int/float) | `let n: number = 3.14` |
| `boolean` | Đúng/sai | `let ok: boolean = true` |
| `bigint` | Số nguyên **rất lớn** (vượt `Number.MAX_SAFE_INTEGER`) | `let big: bigint = 123n` |
| `symbol` | Định danh **duy nhất**, không trùng | `let id: symbol = Symbol("id")` |
| `null` | Cố ý "rỗng" | `let a: null = null` |
| `undefined` | Chưa gán giá trị | `let b: undefined = undefined` |

```ts
let name: string = "An";
let age: number = 25;
let active: boolean = true;
// TS tự suy luận kiểu (type inference) nếu không chú thích:
let city = "HN"; // TS hiểu city: string → city = 10 ❌ lỗi compile
```

**② Kiểu đặc biệt của TypeScript** (không có khái niệm tương đương trong JS thuần):

- **`any`** — **tắt** kiểm tra kiểu: biến nhận **mọi** giá trị, "thoát" khỏi type system. Tiện khi
  migrate code cũ nhưng **lạm dụng = mất hết lợi ích của TS**.
- **`unknown`** — "**`any` an toàn**": cũng nhận mọi giá trị, **nhưng phải kiểm tra/thu hẹp kiểu
  (narrowing) trước khi dùng**. Ưu tiên dùng thay cho `any`.
- **`void`** — hàm **không trả về** giá trị (ngầm trả `undefined`).
- **`never`** — kiểu **không bao giờ có giá trị**: hàm luôn `throw`, vòng lặp vô tận, hoặc nhánh
  **không thể xảy ra** (dùng để kiểm tra exhaustiveness của `switch`).

```ts
let x: any = 5;        x = "str";  x.foo(); // ✅ compile, ❌ dễ bug runtime
let y: unknown = 5;    y = "str";
// y.toUpperCase();    // ❌ lỗi: phải narrow trước
if (typeof y === "string") y.toUpperCase(); // ✅ OK

function log(msg: string): void { console.log(msg); }       // void
function fail(m: string): never { throw new Error(m); }     // never
```

**③ Kiểu object & cấu trúc:**

- **`object`** — bất kỳ giá trị **không phải primitive**.
- **Array** — `number[]` hoặc `Array<number>` (generic).
- **Tuple** — mảng **cố định độ dài & kiểu từng vị trí**: `[string, number]`.
- **`enum`** — tập **hằng số có tên** (numeric hoặc string enum). Đặc trưng TS: **compile ra object JS
  thật** → tồn tại lúc runtime (khác `interface`/`type` bị **type erasure**, xem [câu 1](#q1)).

```ts
let nums: number[] = [1, 2, 3];
let pair: [string, number] = ["age", 25]; // tuple: đúng kiểu theo vị trí

enum Role { Admin, User, Guest }   // Admin=0, User=1, Guest=2
let r: Role = Role.Admin;

enum Status { Active = "ACTIVE", Banned = "BANNED" } // string enum
```

**④ Kiểu nâng cao (kết hợp kiểu):**

- **Union `A | B`** — giá trị thuộc **một trong** nhiều kiểu (rất hay dùng).
- **Intersection `A & B`** — **gộp** nhiều kiểu (object có đủ thuộc tính cả hai).
- **Literal types** — dùng **giá trị cụ thể** làm kiểu: `"GET" | "POST"`, `1 | 2 | 3`.
- **`type` (type alias) vs `interface`** — đặt tên cho "hình dạng" object; `interface` thiên về mô tả
  object/hợp đồng và có thể kế thừa/merge, `type` linh hoạt hơn (union, primitive, tuple).

```ts
let id: string | number = "abc"; id = 123;        // union
type Method = "GET" | "POST" | "PUT";             // literal union
let m: Method = "GET";                             // chỉ nhận đúng 3 giá trị

interface User { id: number; name: string; }       // interface
type Point = { x: number; y: number };             // type alias
type Staff = User & { salary: number };            // intersection
```

> **Điểm hay bị hỏi:** ① `any` vs `unknown` (cả hai nhận mọi giá trị, nhưng `unknown` **bắt buộc
> narrow** trước khi dùng → an toàn hơn); ② `void` vs `never` (`void` = không trả gì/`undefined`;
> `never` = **không bao giờ** trả về); ③ TS **không** thêm kiểu số nguyên/thực riêng — chỉ có `number`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Chia làm **4 nhóm**: ① **nguyên thủy** (như JS): `string`, `number`, `boolean`, `bigint`, `symbol`,
  `null`, `undefined`; ② **đặc biệt của TS**: `any`, `unknown`, `void`, `never`; ③ **object/cấu trúc**:
  `object`, array (`number[]`), **tuple** `[string, number]`, **`enum`**; ④ **nâng cao**: **union**
  `A | B`, **intersection** `A & B`, **literal** `"GET" | "POST"`, `type`/`interface`.
- **`number`** dùng cho cả số nguyên & thực (không tách int/float); **`bigint`** cho số nguyên rất lớn;
  **`symbol`** là định danh duy nhất.
- **`any`** tắt kiểm tra kiểu (lạm dụng = mất lợi ích TS); **`unknown`** là "`any` an toàn" — phải
  **narrow** trước khi dùng. **`void`** = hàm không trả gì; **`never`** = không bao giờ trả về (throw/
  loop vô tận).
- **Tuple** = mảng cố định độ dài & kiểu từng vị trí; **`enum`** = tập hằng số có tên và **tồn tại lúc
  runtime** (khác `interface`/`type` bị type erasure).
- TS còn có **type inference** — tự suy kiểu khi không chú thích.

[↑ Mục lục](#mục-lục)

<a id="q1-2"></a>
### 1.2. Phân biệt object, type và interface trong TypeScript

#### 📖 Giới thiệu kiến thức

Ba khái niệm này **dễ bị nhầm là cùng loại**, nhưng thực ra thuộc **hai phạm trù khác nhau**:

- **`object`** là **một KIỂU dữ liệu** (giống `string`, `number`).
- **`type` và `interface`** là **hai CÁCH khai báo / đặt tên kiểu** (không phải kiểu cụ thể) — dùng để
  mô tả "hình dạng" (shape) của một object.

**① `object` — kiểu "bất kỳ giá trị non-primitive":**

`object` (chữ thường) đại diện cho **mọi giá trị không phải primitive** (không phải `string`, `number`,
`boolean`, `null`, `undefined`, `symbol`, `bigint`). Nó **rất chung chung** — chỉ nói "đây là object"
chứ **không mô tả** object có thuộc tính gì → hầu như **không dùng để định kiểu dữ liệu thực tế**.

```ts
let o: object = { name: "An" };
o = [1, 2, 3];      // ✅ mảng cũng là object
o = "hello";        // ❌ string là primitive
// o.name;          // ❌ object không biết có thuộc tính 'name'
```

> Phân biệt 3 thứ hay lẫn: **`object`** (non-primitive) ≠ **`{}`** (gần như mọi giá trị trừ
> `null`/`undefined`) ≠ **`Object`** (interface global của JS). Thực tế nên mô tả **shape cụ thể**
> (`{ name: string }`) thay vì dùng `object` trống.

**② `interface` — khai báo "hợp đồng" (contract) cho object:**

`interface` mô tả **cấu trúc** một object phải có. Thiên về **OOP**, dùng nhiều cho public API, class
(`implements`). Hai đặc tính **riêng** của `interface`:
- **`extends`** — kế thừa/mở rộng interface khác.
- **Declaration merging** — khai báo **trùng tên** nhiều lần thì TS **tự gộp** (hữu ích khi mở rộng type
  của thư viện bên thứ ba).

```ts
interface User {
  id: number;
  name: string;
}
interface User { email: string; }     // ✅ merge → User có id, name, email
interface Admin extends User {          // ✅ kế thừa
  role: string;
}
// class implements interface → buộc phải có đủ id, name, email
class Account implements User {
  id = 1; name = "An"; email = "an@x.com";
}
```

**③ `type` (type alias) — đặt tên cho BẤT KỲ kiểu nào:**

`type` linh hoạt hơn: đặt tên cho **mọi kiểu** — không chỉ object mà cả primitive, **union**,
**intersection**, **tuple**, **mapped/conditional type**. Đổi lại, `type` **không** merge được (khai
báo trùng tên = lỗi) và gộp kiểu bằng **`&`** thay cho `extends`.

```ts
type ID = string | number;              // union — interface KHÔNG làm được
type Point = { x: number; y: number };  // shape object
type Staff = User & { salary: number }; // intersection (gộp)
type Pair = [string, number];           // tuple
type Role = "admin" | "user";           // literal union
```

**So sánh `interface` vs `type` (phần hay hỏi nhất):**

| Tiêu chí | **`interface`** | **`type`** |
|---|---|---|
| Mô tả shape object | ✅ | ✅ |
| Union / tuple / primitive | ❌ | ✅ |
| Mapped / conditional type | ❌ | ✅ |
| Mở rộng | `extends` | `&` (intersection) |
| **Declaration merging** (trùng tên → gộp) | ✅ | ❌ (lỗi) |
| `implements` cho class | ✅ (tự nhiên) | ✅ (nếu là shape object) |

**Khi nào dùng cái nào:**
- **`interface`** → mô tả **shape của object** và **public API / class** (dễ extend & merge, hợp OOP).
- **`type`** → mọi trường hợp còn lại: **union, alias primitive, tuple, mapped/conditional**, hoặc khi
  cần kết hợp kiểu phức tạp.
- Quy ước thực tế: ưu tiên `interface` cho object; `type` khi cần khả năng `interface` không có.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Khác phạm trù:** `object` là **một kiểu** (mọi giá trị non-primitive); còn `type`/`interface` là
  **hai cách đặt tên/khai báo kiểu** để mô tả shape object.
- **`object`** rất chung chung, không mô tả thuộc tính → thực tế hiếm dùng, nên mô tả shape cụ thể.
  Phân biệt `object` (non-primitive) ≠ `{}` (gần như mọi giá trị) ≠ `Object` (global).
- **`interface`**: mô tả hợp đồng object, thiên OOP, có **`extends`** và **declaration merging** (trùng
  tên tự gộp) — hợp cho public API/class.
- **`type`**: linh hoạt hơn — đặt tên cho **mọi kiểu** (union, tuple, primitive, mapped/conditional),
  gộp bằng **`&`**, nhưng **không merge** được.
- **Chọn:** object & public API → `interface`; union/tuple/alias/mapped → `type`.

[↑ Mục lục](#mục-lục)

<a id="q1-3"></a>
### 1.3. Generics (generic types) trong TypeScript

#### 📖 Giới thiệu kiến thức

**Generics** = **kiểu được tham số hóa (type parameter)**: viết hàm / class / type hoạt động với
**nhiều kiểu khác nhau** mà **vẫn giữ type safety**. Hiểu nôm na: giống tham số của hàm, nhưng tham số
ở đây là **kiểu**, được điền vào lúc gọi/dùng. Generic giải bài toán: muốn tái dùng cho nhiều kiểu
nhưng **không muốn mất kiểu** (dùng `any` sẽ mất, không autocomplete, không bắt lỗi).

```ts
// ❌ Dùng any: mất kiểu trả về
function identityAny(v: any): any { return v; }
const a = identityAny(5);  // a: any → mất kiểu

// ✅ Generic: T được suy ra theo lúc gọi
function identity<T>(value: T): T { return value; }
const b = identity(5);        // T = number → b: number (TS tự suy luận)
const c = identity<string>("hi"); // điền T = string tường minh
```

**Áp dụng được cho:** hàm, `interface`/`type`, và class:

```ts
interface Box<T> { value: T; }          // generic interface
type Pair<K, V> = { key: K; value: V }; // nhiều type param

class Stack<T> {                         // generic class
  private items: T[] = [];
  push(item: T) { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}
const s = new Stack<number>(); s.push(1); // chỉ nhận number
```

**Constraint (`extends`) — giới hạn kiểu:** ép `T` phải thỏa một shape nào đó để dùng được thuộc tính:

```ts
// T phải có .length → mới gọi được x.length
function len<T extends { length: number }>(x: T): number {
  return x.length;
}
len("abc");      // ✅ 3
len([1, 2, 3]);  // ✅ 3
// len(123);     // ❌ number không có length

// keyof + generic: lấy thuộc tính an toàn theo key
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
getProp({ name: "An", age: 25 }, "name"); // trả về string, gõ sai key → lỗi compile
```

**Default type param:** `<T = string>` cho kiểu mặc định khi không truyền.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Generics** = **kiểu tham số hóa** (`<T>`): viết hàm/class/type dùng cho **nhiều kiểu** mà **vẫn giữ
  type safety** — thay cho `any` (vốn làm mất kiểu, mất autocomplete, mất bắt lỗi).
- `T` thường **được TS tự suy luận** theo đối số lúc gọi, hoặc điền tường minh `identity<string>(...)`.
- Dùng cho **hàm, `interface`/`type`, class**; có thể nhiều tham số `<K, V>` và **default** `<T = X>`.
- **Constraint** `<T extends Shape>` giới hạn kiểu để dùng được thuộc tính; kết hợp **`keyof`**
  (`<K extends keyof T>`) để truy cập thuộc tính an toàn theo key.
- Lợi ích: **tái sử dụng + an toàn kiểu**; là nền tảng của conditional types ([1.5](#q1-5)) và utility
  types ([1.6](#q1-6)).

[↑ Mục lục](#mục-lục)

<a id="q1-4"></a>
### 1.4. Type guards (thu hẹp kiểu) trong TypeScript

#### 📖 Giới thiệu kiến thức

**Type guard (type narrowing)** là kỹ thuật **thu hẹp kiểu** của một biến trong một nhánh code, để TS
hiểu **chính xác** kiểu tại đó. Cần thiết vì biến kiểu **union** (`string | number`) hay `unknown` chưa
dùng trực tiếp được — phải kiểm tra trước. Vì kiểu **bị xóa lúc runtime** (type erasure — xem
[câu 1](#q1)), type guard dùng các phép kiểm tra **chạy thật lúc runtime** để TS suy luận kiểu.

**Các type guard có sẵn:**

```ts
// 1) typeof — cho primitive
function f(x: string | number) {
  if (typeof x === "string") x.toUpperCase(); // ở đây x: string
  else x.toFixed(2);                          // ở đây x: number
}

// 2) instanceof — cho class
function g(e: unknown) {
  if (e instanceof Error) console.log(e.message); // e: Error
}

// 3) in — kiểm tra có thuộc tính
type Dog = { bark(): void }; type Cat = { meow(): void };
function speak(a: Dog | Cat) {
  if ("bark" in a) a.bark(); else a.meow();
}

// 4) truthiness & so sánh (null check)
function h(s?: string) { if (s) s.length; } // loại undefined
```

**User-defined type guard (type predicate `x is T`):** tự viết hàm trả về `param is Type` để TS narrow:

```ts
interface Fish { swim(): void; } interface Bird { fly(): void; }
function isFish(a: Fish | Bird): a is Fish {
  return (a as Fish).swim !== undefined;
}
function move(a: Fish | Bird) {
  if (isFish(a)) a.swim(); // TS biết a: Fish
  else a.fly();
}
```

**Discriminated union (tagged union) — pattern rất mạnh:** mỗi nhánh có một **thuộc tính phân biệt
(discriminant)** là literal type; TS narrow qua `switch`/`if`:

```ts
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "square"; side: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.r ** 2; // s: circle
    case "square": return s.side ** 2;         // s: square
  }
}
```

> **`as` (type assertion) khác type guard:** `as` chỉ **ép TS tin** kiểu (không kiểm tra runtime, dễ
> sai/che giấu bug); type guard **kiểm tra thật** rồi mới narrow → an toàn hơn.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Type guard / narrowing** = thu hẹp kiểu của biến trong một nhánh để TS biết **chính xác** kiểu —
  cần cho **union** và **`unknown`**. Hoạt động bằng kiểm tra **chạy thật lúc runtime** (vì kiểu bị xóa).
- **Có sẵn:** `typeof` (primitive), `instanceof` (class), `in` (có thuộc tính), truthiness/`null` check.
- **User-defined:** hàm trả về **type predicate `x is T`** để tự định nghĩa cách narrow.
- **Discriminated union:** union có thuộc tính **discriminant** (literal, vd `kind`) → narrow gọn qua
  `switch`/`if`; pattern phổ biến & an toàn (kết hợp `never` để check exhaustiveness).
- **`as` ≠ type guard:** `as` chỉ ép TS tin (không kiểm tra runtime, dễ giấu bug); type guard kiểm tra
  thật → an toàn hơn.

[↑ Mục lục](#mục-lục)

<a id="q1-5"></a>
### 1.5. Conditional types trong TypeScript

#### 📖 Giới thiệu kiến thức

**Conditional type** = **kiểu có điều kiện**: chọn kiểu này hay kiểu kia dựa trên một điều kiện kiểu,
cú pháp giống **toán tử ba ngôi cho type**:

```ts
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;
type A = IsString<string>; // true
type B = IsString<number>; // false
```

**`infer` — trích xuất (extract) kiểu trong điều kiện:** khai báo một "biến kiểu" để lấy ra phần kiểu
bên trong. Đây là cơ chế đứng sau nhiều utility type:

```ts
// Lấy kiểu trả về của một hàm
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type R1 = MyReturnType<() => number>;        // number
type R2 = MyReturnType<(x: string) => User>; // User

// Lấy kiểu phần tử của mảng
type ElementType<T> = T extends (infer U)[] ? U : T;
type E = ElementType<string[]>; // string
```

**Distributive conditional types — phân phối trên union:** khi `T` là **union** và đặt "trần"
(`T extends ...`), conditional type **áp riêng cho từng thành viên** rồi gộp lại:

```ts
type ToArray<T> = T extends any ? T[] : never;
type X = ToArray<string | number>; // string[] | number[]  (KHÔNG phải (string|number)[])

// Ứng dụng: loại bỏ một kiểu khỏi union (chính là Exclude built-in)
type MyExclude<T, U> = T extends U ? never : T;
type Y = MyExclude<"a" | "b" | "c", "b">; // "a" | "c"
```

**Một số utility type built-in dùng conditional + `infer`:** `Exclude<T,U>`, `Extract<T,U>`,
`NonNullable<T>`, `ReturnType<T>`, `Parameters<T>`, `InstanceType<T>` (xem thêm [1.6](#q1-6)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Conditional type** = "ternary cho kiểu": **`T extends U ? X : Y`** — chọn kiểu theo điều kiện; dựng
  trên **generics** ([1.3](#q1-3)).
- **`infer`** = trích xuất một phần kiểu trong điều kiện (vd lấy **return type** của hàm, **element
  type** của mảng) — cơ chế đứng sau nhiều utility type.
- **Distributive:** khi `T` là **union**, conditional **áp cho từng thành viên** rồi gộp
  (`ToArray<string|number>` → `string[] | number[]`).
- Là nền của các built-in: **`Exclude`, `Extract`, `NonNullable`, `ReturnType`, `Parameters`**.

[↑ Mục lục](#mục-lục)

<a id="q1-6"></a>
### 1.6. Advanced types: keyof, mapped types & utility types

#### 📖 Giới thiệu kiến thức

Nhóm tính năng cho phép **tạo kiểu mới từ kiểu cũ** (type transformation) — đỉnh cao của type system TS.

**① `keyof` — lấy union các key:**

```ts
type User = { id: number; name: string; age: number };
type Keys = keyof User;        // "id" | "name" | "age"
```

**② Indexed access type `T[K]` — lấy kiểu của một thuộc tính:**

```ts
type NameType = User["name"];          // string
type IdOrName = User["id" | "name"];   // number | string
```

**③ Mapped types — "lặp qua từng key" để tạo type mới:** cú pháp `{ [K in keyof T]: ... }`. Có thể
thêm/bớt **modifier** `readonly` và `?` (dùng `-readonly`, `-?` để gỡ):

```ts
// biến mọi field thành readonly
type ReadonlyUser = { readonly [K in keyof User]: User[K] };
// biến mọi field thành optional
type PartialUser  = { [K in keyof User]?: User[K] };
// gỡ optional (-?) và gỡ readonly (-readonly)
type RequiredUser = { [K in keyof PartialUser]-?: PartialUser[K] };
```

**④ Utility types built-in** (chính là mapped + conditional type đóng gói sẵn — dùng hằng ngày):

| Utility | Tác dụng |
|---|---|
| `Partial<T>` | Mọi thuộc tính thành **optional** |
| `Required<T>` | Mọi thuộc tính thành **bắt buộc** |
| `Readonly<T>` | Mọi thuộc tính thành **readonly** |
| `Pick<T, K>` | **Chọn** một số key từ `T` |
| `Omit<T, K>` | **Loại** một số key khỏi `T` |
| `Record<K, V>` | Tạo object với key `K`, value `V` |
| `Exclude<T, U>` / `Extract<T, U>` | Loại / giữ kiểu trong union |
| `NonNullable<T>` | Loại `null` & `undefined` |
| `ReturnType<T>` / `Parameters<T>` | Lấy kiểu return / tham số của hàm |

```ts
type P    = Partial<User>;             // { id?, name?, age? }
type Card = Pick<User, "id" | "name">; // { id: number; name: string }
type NoId = Omit<User, "id">;          // { name: string; age: number }
type Dict = Record<string, number>;    // { [k: string]: number }
```

**⑤ Template literal types** — ghép literal type như chuỗi (mạnh khi tạo key động):

```ts
type Event = "click" | "hover";
type Handler = `on${Capitalize<Event>}`; // "onClick" | "onHover"
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`keyof T`** → union các key; **`T[K]`** (indexed access) → kiểu của thuộc tính.
- **Mapped types** `{ [K in keyof T]: ... }` → **tạo kiểu mới bằng cách lặp qua key**; thêm/bớt
  modifier `readonly`/`?` (gỡ bằng `-readonly`/`-?`).
- **Utility types** = mapped + conditional ([1.5](#q1-5)) đóng gói sẵn: **`Partial`, `Required`,
  `Readonly`, `Pick`, `Omit`, `Record`, `Exclude`/`Extract`, `NonNullable`, `ReturnType`/`Parameters`**.
- **Template literal types** ghép literal thành chuỗi (vd `` `on${Capitalize<E>}` `` → `"onClick"`).
- Tất cả giúp **biến đổi kiểu** (type transformation) → DRY, không phải khai báo lại type thủ công.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. ES5, ES6 và ES7 khác nhau những gì?

### 📖 Giới thiệu kiến thức

**ECMAScript (ES)** là **đặc tả chuẩn** (specification) mà JavaScript hiện thực — do **TC39** quản lý.
Tên phiên bản có 2 cách gọi: theo **số thứ tự** (ES5, ES6, ES7...) hoặc theo **năm** (ES2015,
ES2016...). Từ ES6 trở đi, mỗi năm ra một bản:
- **ES5 = 2009**, **ES6 = ES2015**, **ES7 = ES2016**, ES8 = ES2017 (`async/await`), ...

**ES5 (2009) — nền tảng "JavaScript cũ":** bản được hỗ trợ rộng rãi nhất suốt thời gian dài, code chạy
mọi trình duyệt. Đặc trưng:
- Khai báo biến chỉ có **`var`** (function-scoped, bị hoisting → dễ lỗi scope).
- Hàm thường (`function`), chưa có arrow function.
- Thêm method mảng hữu ích: `forEach`, `map`, `filter`, `reduce`; `Object.create`, `Object.defineProperty`;
  hỗ trợ **JSON** (`JSON.parse/stringify`); **strict mode** (`"use strict"`).

**ES6 / ES2015 — bản nâng cấp LỚN nhất** (interviewer hỏi nhiều nhất). Những tính năng cốt lõi:
- **`let` / `const`** — block-scoped, có **TDZ**, thay thế `var` (xem [câu 8](#q8) về hoisting).
- **Arrow function** `() => {}` — ngắn gọn, **không có `this` riêng** (lấy `this` từ scope ngoài —
  lexical this).
- **Template literals** `` `Hello ${name}` `` (chuỗi nhiều dòng, nội suy biến).
- **Destructuring** `const {a, b} = obj; const [x, y] = arr;`
- **Spread / rest** `...` (gộp/sao chép mảng-object, gom tham số).
- **Default parameters** `function f(a = 1) {}`.
- **Class** (cú pháp `class`, `extends`, `super` — sugar trên prototype).
- **Promise** (chuẩn hóa xử lý bất đồng bộ — xem [câu 9.2](#q9-2)).
- **Module** `import` / `export` (ES Modules).
- **Mới khác:** `Map`, `Set`, `Symbol`, generator (`function*`), `for...of`, computed property name.

**ES7 / ES2016 — bản RẤT NHỎ**, chỉ thêm đúng **2 thứ**:
- **Toán tử lũy thừa** `**`: `2 ** 3 === 8` (thay cho `Math.pow(2, 3)`).
- **`Array.prototype.includes`**: `[1,2,3].includes(2) === true` (kiểm tra phần tử, xử lý `NaN` tốt
  hơn `indexOf`).

```js
// ES5
var nums = [1, 2, 3];
var doubled = nums.map(function (n) { return n * 2; });

// ES6
const nums2 = [1, 2, 3];
const doubled2 = nums2.map((n) => n * 2);          // arrow + const
const [first, ...rest] = nums2;                     // destructuring + rest
const msg = `Phần tử đầu: ${first}`;                // template literal

// ES7
console.log(2 ** 10);                               // 1024 (lũy thừa)
console.log(nums2.includes(2));                     // true
```

> **Lưu ý phỏng vấn:** trọng tâm gần như luôn là **ES5 vs ES6** (vì ES6 là bước nhảy lớn); **ES7 chỉ
> là bản nhỏ** (`**` và `includes`). Nhiều người gọi chung **"ES6+"** để chỉ JavaScript hiện đại (ES6
> trở lên). Trình duyệt cũ không hỗ trợ ES6+ → cần **Babel** transpile về ES5.

### 🎯 Tóm tắt trả lời phỏng vấn

- **ECMAScript** là chuẩn của JavaScript (TC39). **ES5 = 2009**, **ES6 = ES2015**, **ES7 = ES2016**
  (từ ES6 đặt tên theo năm).
- **ES5:** nền tảng "JS cũ" — chỉ có **`var`**, method mảng (`map/filter/reduce`), JSON, strict mode.
- **ES6 (ES2015) — bản nâng cấp lớn nhất:** **`let/const`**, **arrow function** (lexical `this`),
  **template literal**, **destructuring**, **spread/rest**, **default param**, **class**, **Promise**,
  **module `import/export`**, `Map/Set/Symbol`, generator, `for...of`.
- **ES7 (ES2016) — rất nhỏ:** chỉ thêm **toán tử lũy thừa `**`** và **`Array.includes`**.
- Trọng tâm là **ES5 vs ES6**; gọi chung **"ES6+"** = JS hiện đại; trình duyệt cũ cần **Babel**.

[↑ Mục lục](#mục-lục)

<a id="q2-1"></a>
### 2.1. Babel là gì?

#### 📖 Giới thiệu kiến thức

**Babel** là một **JavaScript compiler / transpiler** — công cụ **chuyển code JS hiện đại (ES6+, ESNext)
thành JS tương thích ngược (thường là ES5)** để chạy được trên trình duyệt/môi trường **cũ** không hỗ
trợ cú pháp mới. Đây chính là lý do nhắc ở [câu 2](#q2): viết JS hiện đại nhưng vẫn cần chạy trên IE11,
trình duyệt đời cũ → Babel "dịch ngược" cú pháp.

**Transpile (transpiler) là gì?** Là **source-to-source compile** — dịch từ ngôn ngữ này sang ngôn ngữ
**cùng mức trừu tượng** (JS → JS), khác với compiler truyền thống dịch xuống mã máy/bytecode. Babel viết
lại `const`, arrow function, `class`, optional chaining... thành cú pháp ES5 **tương đương**.

```js
// Input (ES6+)
const greet = (name) => `Hi ${name}`;
class A { #secret = 1; }

// Output Babel (ES5, rút gọn)
var greet = function (name) { return "Hi " + name; };
// class & private field → viết lại bằng function + closure
```

**Babel dùng để làm gì (không chỉ ES6→ES5):**
- Transpile **ES6+ → ES5** (mục đích phổ biến nhất).
- Biên dịch **JSX** (React) → `React.createElement`.
- **Strip type của TypeScript** qua `@babel/preset-typescript` — nhưng **chỉ xóa type, KHÔNG
  type-check** (khác `tsc`; liên hệ type erasure ở [câu 1](#q1)).
- Dùng **tính năng đề xuất (proposal/stage)** chưa chuẩn hóa.

**Cách hoạt động — 3 giai đoạn (giống pipeline compiler & AST ở [câu 8](#q8)):**

```
1. Parse     : code → AST (Abstract Syntax Tree)
2. Transform : plugin biến đổi AST (đây là nơi "phép thuật" xảy ra)
3. Generate  : AST → code JS đầu ra
```

**Plugin & Preset:**
- **Plugin** = **một** phép biến đổi (vd `transform-arrow-functions`).
- **Preset** = **bộ plugin** đóng gói sẵn. Quan trọng nhất là **`@babel/preset-env`**: tự chọn cần
  transpile gì dựa trên **target trình duyệt** (cấu hình qua **browserslist**) → không transpile thừa.

**Syntax vs API — điểm CỐT LÕI hay bị hỏi:**
- Babel chỉ chuyển được **cú pháp (syntax)**: arrow, `let/const`, `class`, destructuring, optional chaining...
- **API/tính năng runtime mới** (`Promise`, `Map`, `Set`, `Array.prototype.includes`, `fetch`...)
  **KHÔNG thể "viết lại bằng cú pháp"** → cần **polyfill** (thường là **core-js**) để **bổ sung phần
  hiện thực** cho môi trường cũ.

**Quan hệ với bundler:** Babel thường chạy trong **Webpack** (qua `babel-loader`), **Rollup**, hoặc
song song với **Vite/esbuild/SWC** (các tool mới viết bằng Go/Rust nhanh hơn, đang dần thay Babel ở
nhiều dự án, nhưng nguyên lý transpile vẫn vậy).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Babel** là **transpiler JS**: chuyển **JS hiện đại (ES6+) → JS cũ (ES5)** để chạy trên trình duyệt/
  môi trường **không hỗ trợ cú pháp mới**. **Transpile** = dịch **source → source** cùng mức trừu tượng
  (JS→JS), khác compile xuống mã máy.
- Không chỉ ES6→ES5: còn biên dịch **JSX**, **strip type TypeScript** (`@babel/preset-typescript` — chỉ
  xóa type, **không type-check**), dùng tính năng **proposal** mới.
- **Hoạt động 3 bước:** **Parse → AST → Transform (plugin) → Generate** code.
- **Plugin** = 1 phép biến đổi; **Preset** = bộ plugin. **`@babel/preset-env`** + **browserslist** tự
  transpile theo target → không thừa.
- **Mấu chốt:** Babel chỉ chuyển **cú pháp**; **API mới** (`Promise`, `Map`, `includes`...) cần
  **polyfill (core-js)** vì không thể viết lại bằng cú pháp.

[↑ Mục lục](#mục-lục)

<a id="q2-2"></a>
### 2.2. Webpack là gì?

#### 📖 Giới thiệu kiến thức

**Webpack** là một **module bundler** (công cụ đóng gói module) cho JavaScript. Nó nhận **rất nhiều
file rời rạc** trong dự án (JS, CSS, ảnh, font, JSON...), phân tích quan hệ phụ thuộc giữa chúng rồi
**gộp lại thành một (hoặc vài) file tối ưu** để trình duyệt tải.

**Vì sao cần bundler?** Một app frontend hiện đại được chia thành hàng trăm module nhỏ (`import` /
`require` lẫn nhau) cho dễ bảo trì. Nhưng:
- Trình duyệt cũ không hiểu cú pháp ES Module; tải hàng trăm file riêng lẻ → **rất nhiều HTTP request**, chậm.
- Cần xử lý cả tài nguyên không phải JS (SCSS, ảnh, SVG...) và biến đổi code (Babel, TypeScript).

Webpack giải quyết bằng cách dựng một **dependency graph**: bắt đầu từ **entry**, lần theo mọi `import`
để gom toàn bộ module, áp dụng biến đổi, rồi xuất ra **bundle** ở **output**.

**4 khái niệm cốt lõi (interview hay hỏi):**

| Khái niệm | Vai trò |
|-----------|---------|
| **Entry** | Điểm bắt đầu để build dependency graph (vd `src/index.js`). |
| **Output** | Nơi và tên file bundle xuất ra (vd `dist/bundle.[contenthash].js`). |
| **Loaders** | Webpack **chỉ hiểu JS/JSON**. Loader giúp "dịch" file khác thành module: `babel-loader` (ES6+/JSX → ES5), `css-loader`, `sass-loader`, `vue-loader`, `ts-loader`... |
| **Plugins** | Mở rộng việc làm ở phạm vi build: `HtmlWebpackPlugin` (sinh HTML), `MiniCssExtractPlugin` (tách CSS ra file), `DefinePlugin` (inject biến env), minify... |

**Tính năng tối ưu quan trọng:**
- **Code splitting**: tách bundle thành nhiều chunk (vd vendor riêng, tải theo route) để **lazy-load**,
  giảm dung lượng tải ban đầu.
- **Tree shaking**: loại bỏ code **không được dùng** (dead code) dựa trên ES Module tĩnh.
- **Hot Module Replacement (HMR)**: thay module đang sửa **ngay khi dev** mà không reload cả trang.
- **Asset hashing**: thêm hash vào tên file (`[contenthash]`) để **cache-busting** khi deploy.

```js
// webpack.config.js (rút gọn)
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',                 // điểm vào
  output: {                                // file xuất
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.[contenthash].js',
  },
  module: {
    rules: [
      { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ }, // dịch ES6+ → ES5
      { test: /\.css$/, use: ['style-loader', 'css-loader'] },          // import CSS vào JS
    ],
  },
  plugins: [ new HtmlWebpackPlugin({ template: './src/index.html' }) ],  // tự sinh index.html
  mode: 'production', // bật minify + tree shaking; 'development' để dev nhanh + HMR
};
```

**Webpack vs Babel:** Babel là **transpiler** (chỉ chuyển **cú pháp** JS→JS), còn Webpack là **bundler**
(gom & tối ưu **toàn bộ** tài nguyên). Chúng **bổ trợ**: Webpack gọi Babel qua `babel-loader` để dịch
từng file JS trong lúc đóng gói — xem [Babel](#q2-1).

**So với tool mới:** Webpack mạnh, cấu hình linh hoạt nhưng **cấu hình phức tạp** và build chậm với
dự án lớn. Các tool mới — **Vite** (dev dùng native ESM + esbuild), **esbuild/Rollup/Turbopack** — nhanh
hơn nhiều và đang dần thay thế; nhưng nguyên lý "dựng dependency graph rồi bundle" thì vẫn giống nhau.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Webpack** là **module bundler**: nhận nhiều file (JS, CSS, ảnh...), dựng **dependency graph** từ
  **entry**, rồi gộp & tối ưu thành **bundle** ở **output** cho trình duyệt tải.
- **4 khái niệm cốt lõi:** **Entry** (điểm vào) · **Output** (file xuất) · **Loaders** (dịch file
  non-JS như `babel-loader`, `css-loader` thành module) · **Plugins** (mở rộng build: sinh HTML, tách
  CSS, minify...).
- **Tối ưu:** **code splitting** (lazy-load chunk), **tree shaking** (xóa dead code), **HMR** (hot
  reload khi dev), **content hash** (cache-busting).
- **Khác Babel:** Babel chỉ **transpile cú pháp**; Webpack **bundle toàn bộ tài nguyên** và **gọi Babel
  qua `babel-loader`** trong lúc đóng gói — hai thứ bổ trợ nhau.
- **Hiện tại:** Webpack cấu hình phức tạp/chậm với dự án lớn → đang dần được thay bởi **Vite/esbuild**
  (nhanh hơn), nhưng nguyên lý bundle vẫn vậy.

[↑ Mục lục](#mục-lục)

<a id="q2-3"></a>
### 2.3. Phân biệt Webpack và Vite

#### 📖 Giới thiệu kiến thức

Cả hai đều là **build tool** cho frontend, nhưng khác nhau về **triết lý xử lý ở môi trường dev** —
đây là điểm mấu chốt khiến Vite nhanh hơn rất nhiều.

**Webpack — "bundle trước, rồi mới chạy dev server":**
Khi `npm run dev`, Webpack phải **dựng dependency graph và bundle TOÀN BỘ** ứng dụng trước khi server
sẵn sàng. Dự án càng lớn → khởi động (cold start) càng lâu, và mỗi lần sửa file, nó phải build lại
phần liên quan rồi mới HMR. Webpack viết bằng **JavaScript**, chạy trên Node nên xử lý chậm.

**Vite — "tận dụng native ES Module của trình duyệt, không bundle khi dev":**
Trình duyệt hiện đại đã hiểu `import`/`export` (native ESM). Vì vậy ở chế độ dev, Vite **không bundle**:
nó khởi động server gần như **tức thì**, và chỉ khi trình duyệt **yêu cầu một module cụ thể**, Vite mới
**transform & trả file đó on-demand** (dùng **esbuild** viết bằng Go — nhanh gấp 10–100 lần Babel).
→ Cold start cực nhanh, HMR gần như tức thời bất kể dự án lớn cỡ nào.

**Lưu ý quan trọng — production thì Vite VẪN bundle:** Native ESM không bundle sẽ tạo quá nhiều request,
không tối ưu cho production. Nên khi `vite build`, Vite dùng **Rollup** để bundle + tree-shaking + tối
ưu giống Webpack. Vậy khác biệt tốc độ **chủ yếu nằm ở dev**, không phải production.

**Bảng so sánh:**

| Tiêu chí | Webpack | Vite |
|----------|---------|------|
| **Dev server** | Bundle toàn bộ trước → cold start chậm | Native ESM, không bundle → start gần như tức thì |
| **Xử lý khi sửa code** | Build lại phần liên quan rồi HMR | Transform on-demand, HMR gần như tức thời |
| **Tốc độ build (engine)** | JS (chạy trên Node) | Dev: **esbuild** (Go); Build: **Rollup** |
| **Production build** | Webpack tự bundle | Dùng **Rollup** bundle |
| **Cấu hình** | Phức tạp, dài (`webpack.config.js`) | Tối giản, nhiều mặc định hợp lý (`vite.config.js`) |
| **Hệ sinh thái** | Cực lớn, lâu đời, nhiều loader/plugin | Mới hơn, đang phát triển nhanh, plugin theo chuẩn Rollup |
| **Phù hợp** | Dự án lớn/legacy, cần cấu hình tuỳ biến sâu | Dự án mới, ưu tiên DX & tốc độ |

**Tóm lại cơ chế:** Webpack **bundle-based**, Vite **native-ESM-based (ở dev)**. Đây là lý do gốc rễ
của khác biệt tốc độ. Xem thêm [Webpack](#q2-2) và [Babel](#q2-1).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai là **build tool**, khác ở **cách xử lý môi trường dev**:
  - **Webpack** = **bundle-based**: phải **bundle toàn bộ app trước** khi chạy dev server → **cold
    start chậm** với dự án lớn. Viết bằng JS, chạy trên Node.
  - **Vite** = tận dụng **native ES Module** của trình duyệt → **dev không bundle**, chỉ **transform
    on-demand** từng module bằng **esbuild (Go)** → **start gần như tức thì**, HMR siêu nhanh.
- **Production:** Vite **vẫn bundle** bằng **Rollup** (vì native ESM tạo quá nhiều request) → khác biệt
  tốc độ **chủ yếu ở dev**.
- **Cấu hình:** Webpack phức tạp/linh hoạt; Vite tối giản, nhiều mặc định sẵn.
- **Một câu:** *Webpack bundle trước rồi chạy; Vite chạy trước nhờ native ESM rồi transform khi cần —
  nên Vite nhanh hơn nhiều ở dev, còn production cả hai đều bundle.*

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Phân biệt tham trị (pass by value) và tham chiếu (pass by reference) trong JavaScript

### 📖 Giới thiệu kiến thức

Đây là một trong những khái niệm **nền tảng nhất** về cách JavaScript lưu trữ và truyền dữ liệu — gốc
rễ của nhiều bug "vô tình sửa nhầm dữ liệu gốc".

**Trước hết, JS có 2 nhóm kiểu dữ liệu:**

- **Kiểu nguyên thủy (primitive):** `number`, `string`, `boolean`, `null`, `undefined`, `symbol`,
  `bigint`. Là giá trị **bất biến (immutable)**, lưu **trực tiếp giá trị** (khái niệm hóa: nằm trên
  **stack**).
- **Kiểu tham chiếu (reference / object):** `object`, `array`, `function`. Biến **không** chứa object
  trực tiếp mà chứa **một tham chiếu (địa chỉ)** trỏ tới object nằm trên **heap**.

**① Tham trị (pass/copy by value) — với primitive:**

Khi gán hoặc truyền một primitive, JS **sao chép GIÁ TRỊ**. Hai biến độc lập hoàn toàn — sửa cái này
không ảnh hưởng cái kia.

```js
let a = 10;
let b = a;       // copy giá trị → b là bản sao độc lập
b = 20;
console.log(a);  // 10  (a KHÔNG đổi)
```

**② Tham chiếu (copy by reference) — với object:**

Khi gán hoặc truyền một object, JS **sao chép THAM CHIẾU** (địa chỉ), không sao chép object. Hai biến
**cùng trỏ tới một object** trên heap → **mutate** (sửa thuộc tính) qua biến này thì biến kia cũng "thấy".

```js
let o1 = { x: 1 };
let o2 = o1;     // copy tham chiếu → o1, o2 cùng trỏ 1 object
o2.x = 99;
console.log(o1.x); // 99  (o1 BỊ đổi theo)
```

**③ Mấu chốt hay bị hỏi — "JS thực ra LUÔN là pass by value":**

Khi truyền object vào hàm, cái được copy là **giá trị của tham chiếu** (bản sao địa chỉ). Vì vậy:
- **Mutate** object qua tham số → **ảnh hưởng** object gốc (vì cùng địa chỉ).
- **Gán lại (reassign)** tham số một object mới → **KHÔNG** ảnh hưởng biến ngoài (chỉ đổi bản sao địa chỉ
  cục bộ). Đây là lý do người ta gọi cơ chế của JS là **"call by sharing"** (pass-by-reference-value).

```js
function changePrimitive(n) { n = 100; }      // primitive: copy giá trị
let num = 5;
changePrimitive(num);
console.log(num);   // 5  (không đổi)

function mutateObj(obj) { obj.name = "B"; }    // MUTATE → ảnh hưởng gốc
let u = { name: "A" };
mutateObj(u);
console.log(u.name); // "B"

function reassignObj(obj) { obj = { name: "C" }; } // REASSIGN → KHÔNG ảnh hưởng gốc
let v = { name: "A" };
reassignObj(v);
console.log(v.name); // "A"  (chỉ đổi biến cục bộ trong hàm)
```

**④ So sánh `===` với object — so theo tham chiếu, không theo nội dung:**

```js
console.log({ a: 1 } === { a: 1 }); // false — 2 object khác địa chỉ
let p = { a: 1 }; let q = p;
console.log(p === q);               // true  — cùng tham chiếu
```

**⑤ `const` với object — cấm gán lại, vẫn mutate được:**

`const` chỉ khóa **liên kết biến (binding/địa chỉ)**, không khóa **nội dung** object.

```js
const arr = [1, 2];
arr.push(3);   // ✅ OK — mutate nội dung
// arr = [];   // ❌ TypeError — gán lại tham chiếu
```

**⑥ Hệ quả thực tế — Shallow copy vs Deep copy:**

Vì object copy theo tham chiếu, muốn có **bản sao độc lập** phải copy chủ động:
- **Shallow copy (sao chép nông):** `{ ...obj }`, `Object.assign({}, obj)`, `arr.slice()`, `[...arr]`.
  Chỉ copy **tầng đầu tiên** — object **lồng bên trong vẫn dùng chung tham chiếu**.
- **Deep copy (sao chép sâu):** `structuredClone(obj)` (hiện đại, khuyến nghị),
  `JSON.parse(JSON.stringify(obj))` (hạn chế: mất `function`, `undefined`, `Date` → string...),
  hoặc `lodash.cloneDeep`.

```js
const original = { name: "An", address: { city: "HN" } };

const shallow = { ...original };
shallow.name = "Binh";          // ✅ không ảnh hưởng original (tầng đầu)
shallow.address.city = "HCM";   // ⚠️ original.address.city CŨNG đổi (nested dùng chung)

const deep = structuredClone(original); // copy sâu — hoàn toàn độc lập
deep.address.city = "DN";               // original không đổi
```

> Liên hệ: phân nhóm primitive nằm ở [câu 1.1](#q1-1); chuyện stack/heap & thu hồi object không còn
> tham chiếu (Garbage Collection) ở [câu 8](#q8).

### 🎯 Tóm tắt trả lời phỏng vấn

- **2 nhóm kiểu:** **primitive** (`number, string, boolean, null, undefined, symbol, bigint`) truyền
  **theo trị** (copy giá trị); **object/array/function** truyền **theo tham chiếu** (copy địa chỉ — cùng
  trỏ 1 object trên heap).
- **Primitive:** gán/truyền tạo **bản sao độc lập** → sửa bản sao không ảnh hưởng gốc. **Object:** copy
  **tham chiếu** → **mutate** thuộc tính sẽ ảnh hưởng gốc.
- **Mấu chốt:** JS **luôn là pass-by-value** — nhưng với object, "giá trị" được truyền chính là **tham
  chiếu (địa chỉ)**. Vì vậy **mutate** object qua tham số thì ảnh hưởng gốc, còn **gán lại (reassign)**
  tham số thì **không** (chỉ đổi biến cục bộ). Cơ chế này gọi là **"call by sharing"**.
- **`===`** với object so theo **tham chiếu**, không theo nội dung (`{a:1} === {a:1}` → `false`).
- **`const`** object: cấm **gán lại** biến nhưng vẫn **mutate** nội dung được.
- **Tránh sửa nhầm gốc:** **shallow copy** (`{...o}`, `Object.assign`, `slice`) chỉ copy tầng đầu
  (nested vẫn chung); **deep copy** (`structuredClone`, `JSON.parse(JSON.stringify())`, `lodash.cloneDeep`).

[↑ Mục lục](#mục-lục)

<a id="q3-1"></a>
### 3.1. Mutable và Immutable trong JavaScript

#### 📖 Giới thiệu kiến thức

Đây là phần **đào sâu** của [câu 3](#q3): pass-by-value/reference nói về *cách truyền* dữ liệu, còn
mutable/immutable nói về *dữ liệu có sửa được tại chỗ hay không*. Hai khái niệm đi liền nhau và cùng
gốc: **primitive là immutable, object là mutable**.

**① Định nghĩa:**

- **Mutable (khả biến):** giá trị **sửa được tại chỗ** (in-place) sau khi tạo — vẫn cùng một địa chỉ
  trên heap, chỉ nội dung thay đổi. Trong JS: **`object`, `array`, `function`, `Map`, `Set`, `Date`**.
- **Immutable (bất biến):** giá trị **không thể thay đổi** sau khi tạo. Muốn "đổi" thì phải tạo **giá
  trị mới**. Trong JS: **7 kiểu primitive** (`number`, `string`, `boolean`, `null`, `undefined`,
  `symbol`, `bigint`).

**② Primitive là immutable — "sửa" thực ra là gán lại (reassign):**

Điểm hay gây nhầm: khi viết `let x = 5; x = 6;` trông như "sửa" `5` thành `6`, nhưng thực ra bản thân
giá trị `5` **không đổi** — biến `x` chỉ được **trỏ sang** một giá trị `6` mới. Ta thay đổi **biến**,
không phải **giá trị**.

```js
let x = 5;
x = 6;         // KHÔNG sửa số 5 → tạo/gán giá trị mới 6 cho biến x

let s = "hello";
s[0] = "H";           // ❌ vô hiệu (im lặng) — string bất biến, không sửa được từng ký tự
console.log(s);       // "hello"
s = s.toUpperCase();  // ✅ cách duy nhất để "đổi": TẠO chuỗi mới rồi gán lại
console.log(s);       // "HELLO"
```

> Mọi method chuỗi (`toUpperCase`, `slice`, `replace`, `trim`...) **không sửa chuỗi gốc** mà luôn
> **trả về chuỗi MỚI** — đúng bản chất immutable.

**③ Object/array là mutable — sửa tại chỗ, ảnh hưởng mọi tham chiếu:**

```js
const arr = [1, 2, 3];
arr.push(4);        // mutate tại chỗ → [1,2,3,4], vẫn cùng địa chỉ
const user = { name: "An" };
user.name = "Binh"; // mutate thuộc tính → cùng object

const ref = user;
ref.name = "Chi";
console.log(user.name); // "Chi" — mọi biến trỏ tới object đều "thấy" thay đổi
```

Đây là gốc rễ của bug "vô tình sửa nhầm dữ liệu gốc" (đã nói ở [câu 3](#q3)).

**④ `const` KHÔNG phải immutable — hai chuyện khác nhau:**

`const` chỉ khóa **binding** (không cho **gán lại** biến), **không** khóa **nội dung**. Object khai báo
bằng `const` vẫn **mutate** thoải mái.

```js
const obj = { a: 1 };
obj.a = 99;   // ✅ OK — mutate nội dung
obj.b = 2;    // ✅ OK — thêm field
// obj = {};  // ❌ TypeError — không được gán lại binding
```

**⑤ Làm object thật sự bất biến — `Object.freeze` (shallow):**

```js
const config = Object.freeze({ env: "prod", db: { host: "localhost" } });
config.env = "dev";          // ❌ vô hiệu (strict mode sẽ throw)
console.log(config.env);     // "prod"
console.log(Object.isFrozen(config)); // true

config.db.host = "1.2.3.4";  // ⚠️ VẪN sửa được! freeze chỉ NÔNG (tầng đầu)
```

`Object.freeze` chỉ đóng băng **tầng đầu tiên** → muốn bất biến sâu phải **deep freeze** (đệ quy freeze
mọi nested object).

**⑥ Lập trình theo hướng immutable (immutable update) — rất quan trọng trong React/Redux:**

Thay vì mutate, ta **tạo bản sao mới** kèm thay đổi. Lý do: React/Redux **so sánh tham chiếu** (`===`,
shallow compare) để phát hiện thay đổi và re-render — nếu mutate tại chỗ thì tham chiếu không đổi →
**không** re-render. Ngoài ra immutable giúp code **dễ đoán**, dễ debug, hỗ trợ undo/redo, time-travel.

```js
// ❌ Mutate — React không nhận ra thay đổi (cùng tham chiếu)
state.list.push(item);

// ✅ Immutable — tạo mảng/object MỚI
const newList  = [...state.list, item];              // thêm phần tử
const newState = { ...state, count: state.count + 1 }; // cập nhật field
const filtered = state.list.filter(x => x.id !== id);  // xóa phần tử
```

- **Method không mutate (nên dùng):** `map`, `filter`, `reduce`, `slice`, `concat`, spread `[...]`.
- **Method có mutate (tránh khi cần immutable):** `push`, `pop`, `splice`, `sort`, `reverse`.
- Với nested sâu: `structuredClone` để deep copy, hoặc thư viện **Immer** (viết code "trông như mutate"
  nhưng sinh ra bản bất biến), **Immutable.js**.

**⑦ Bảng tổng kết:**

| | **Immutable** | **Mutable** |
|---|---|---|
| Kiểu | primitive (`number`, `string`, `boolean`, `null`, `undefined`, `symbol`, `bigint`) | `object`, `array`, `function`, `Map`, `Set`, `Date` |
| Sửa tại chỗ | ❌ không — phải tạo giá trị mới | ✅ có — cùng địa chỉ, đổi nội dung |
| "Đổi giá trị" | reassign biến / trả về giá trị mới | mutate thuộc tính/phần tử |
| Ép bất biến | vốn đã bất biến | `Object.freeze` (nông) / deep freeze |

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Immutable (bất biến):** không sửa được sau khi tạo, "đổi" = tạo **giá trị mới** → **7 primitive**
  (`number, string, boolean, null, undefined, symbol, bigint`). **Mutable (khả biến):** sửa **tại chỗ**
  → **object, array, function, Map, Set, Date**.
- **Primitive immutable:** `x = 6` không sửa số `5`, chỉ **trỏ biến** sang giá trị mới; string bất biến
  → mọi method (`toUpperCase`, `slice`...) **trả chuỗi mới**, không sửa gốc.
- **Object mutable:** mutate tại chỗ → mọi biến cùng tham chiếu đều thấy (gốc của bug sửa nhầm dữ liệu).
- **`const` ≠ immutable:** `const` khóa **gán lại binding**, **không** khóa nội dung → object `const` vẫn
  mutate được. Muốn bất biến thật: **`Object.freeze`** (nhưng **nông** — nested vẫn sửa được → cần deep
  freeze).
- **Immutable update** (React/Redux): tạo bản sao mới (`[...arr]`, `{...obj}`, `map/filter`) thay vì
  mutate, để **so sánh tham chiếu** phát hiện thay đổi → re-render đúng; nested sâu dùng `structuredClone`
  hoặc **Immer**.

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Truthy và Falsy trong JavaScript/TypeScript

### 📖 Giới thiệu kiến thức

JavaScript là ngôn ngữ **dynamic typing** với cơ chế **type coercion (ép kiểu ngầm)**. Khi một giá trị
**không phải boolean** được đặt vào **ngữ cảnh cần boolean** — điều kiện của `if`, `while`, toán tử
`!`, `&&`, `||`, toán tử ba ngôi `? :` — JS **tự ép nó về `true` hoặc `false`** (thuật toán nội bộ
`ToBoolean`). Từ đó sinh ra hai khái niệm:

- **Falsy**: những giá trị bị ép về **`false`**.
- **Truthy**: những giá trị bị ép về **`true`** (tức là **mọi thứ còn lại**).

**① Danh sách falsy — CHỈ có đúng 8 giá trị (phải thuộc lòng):**

| Giá trị | Ghi chú |
|---|---|
| `false` | boolean false |
| `0` | số 0 |
| `-0` | số 0 âm |
| `0n` | BigInt zero |
| `""` (`''`, ` `` `) | chuỗi **rỗng** |
| `null` | |
| `undefined` | |
| `NaN` | Not a Number |

> Mẹo nhớ: **false, 3 loại "số 0" (`0`, `-0`, `0n`), chuỗi rỗng, `null`, `undefined`, `NaN`**.
> (Về mặt kỹ thuật còn `document.all` là falsy — một ngoại lệ lịch sử của trình duyệt, gần như không
> bao giờ gặp.)

**② Truthy = tất cả những gì KHÔNG nằm trong danh sách falsy.** Nhiều giá trị dễ tưởng là "rỗng/false"
nhưng thực chất **truthy** — đây là nguồn bug kinh điển:

```js
Boolean("0");       // true  — chuỗi "0" KHÔNG rỗng
Boolean("false");   // true  — chuỗi "false" vẫn là chuỗi có ký tự
Boolean(" ");       // true  — chuỗi 1 dấu cách (không rỗng)
Boolean([]);        // true  — MẢNG RỖNG là truthy (!)
Boolean({});        // true  — OBJECT RỖNG là truthy (!)
Boolean(-1);        // true  — mọi số khác 0
Boolean(Infinity);  // true
Boolean(function(){}); // true — hàm là object
```

**③ Cách ép kiểu tường minh về boolean:** dùng `Boolean(x)` hoặc toán tử **`!!x`** (hai lần phủ định).

```js
!!"hello";  // true
!!0;        // false
!!null;     // false
```

**④ `&&` và `||` trả về GIÁ TRỊ GỐC, không phải boolean (short-circuit):** đây là điểm hay bị hỏi.

```js
// || trả về toán hạng truthy ĐẦU TIÊN (hoặc cái cuối nếu tất cả falsy)
"" || "default";        // "default"
"An" || "default";      // "An"
0 || null || "x";       // "x"

// && trả về toán hạng falsy ĐẦU TIÊN (hoặc cái cuối nếu tất cả truthy)
"a" && "b";             // "b"
0 && "b";               // 0
user && user.name;      // an toàn: nếu user falsy → trả user, không crash
```

**⑤ Cạm bẫy `||` cho giá trị mặc định — và `??` (nullish coalescing) sửa nó:** dùng `||` để gán default
sẽ **loại nhầm** các falsy hợp lệ như `0` và `""`. Nếu **chỉ muốn loại `null`/`undefined`**, dùng `??`.

```js
function setVolume(v) {
  const vol = v || 50;   // ❌ setVolume(0) → vol = 50 (mất giá trị 0 hợp lệ!)
  const vol2 = v ?? 50;  // ✅ setVolume(0) → vol2 = 0; chỉ null/undefined mới → 50
}

const count = 0;
count || "N/A";   // "N/A"  ❌
count ?? "N/A";   // 0       ✅
```

**⑥ Cạm bẫy kiểm tra "mảng/object rỗng":** vì `[]` và `{}` là **truthy**, `if (arr)` **không** kiểm tra
được rỗng — phải kiểm tra `.length` / số key.

```js
const arr = [];
if (arr) { /* LUÔN chạy — arr là truthy dù rỗng */ }
if (arr.length) { /* đúng: chỉ chạy khi có phần tử */ }

const obj = {};
if (Object.keys(obj).length === 0) { /* object rỗng */ }
```

**⑦ Trong TypeScript:** truthy/falsy hoạt động **y hệt JavaScript lúc runtime** (vì kiểu bị **type
erasure** — xem [câu 1](#q1)). Điểm cộng của TS là **truthiness narrowing**: dùng truthy check để **thu
hẹp kiểu** (loại `null`/`undefined`) trong một nhánh:

```ts
function greet(name?: string) {
  // name: string | undefined
  if (name) {
    name.toUpperCase(); // ✅ trong nhánh này TS biết name: string
  }
}
```

> ⚠️ Cạm bẫy TS/JS: truthiness check loại **cả** `""` và `0`, không chỉ `null/undefined`. Với
> `if (user.age)` mà `age = 0` là hợp lệ → dùng `if (user.age !== undefined)` hoặc `age != null` thay vì
> truthy check. (`x != null` với `==` lỏng bắt **cả** `null` và `undefined`.)

**⑧ Lưu ý phân biệt với `==` (loose equality):** truthy/falsy áp dụng khi giá trị **đứng một mình** trong
ngữ cảnh boolean; còn `==` là **so sánh 2 giá trị** với bộ quy tắc ép kiểu **khác**. Vì vậy có nghịch lý:
`if ([])` là truthy (chạy), nhưng `[] == false` lại là `true`. Đừng lẫn hai cơ chế — thực tế ưu tiên
`===` và kiểm tra tường minh.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Falsy** = giá trị bị ép về `false` trong ngữ cảnh boolean; **truthy** = mọi thứ còn lại. JS ép kiểu
  ngầm (`ToBoolean`) khi gặp `if`, `while`, `!`, `&&`, `||`, `? :`.
- **Đúng 8 falsy** (thuộc lòng): **`false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`**. Mọi
  thứ khác truthy — kể cả **`"0"`, `"false"`, `" "`, `[]`, `{}`** (mảng/object rỗng là **truthy**).
- Ép tường minh bằng **`Boolean(x)`** hoặc **`!!x`**.
- **`&&`/`||` trả về giá trị gốc** (short-circuit), không phải boolean: `||` lấy truthy đầu tiên, `&&`
  lấy falsy đầu tiên → dùng cho default value và optional chaining thủ công.
- **Cạm bẫy:** `||` để gán mặc định loại nhầm `0` và `""` → dùng **`??`** khi chỉ muốn loại
  `null`/`undefined`; `if(arr)` không phát hiện mảng rỗng → kiểm `arr.length`.
- **TypeScript:** giống hệt JS runtime (type erasure) + hỗ trợ **truthiness narrowing** để thu hẹp kiểu
  (loại `null`/`undefined`) trong nhánh `if (x)`.

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Phân biệt `for...in` và `for...of` trong JavaScript

### 📖 Giới thiệu kiến thức

Cả hai đều là vòng lặp ES6-style để "duyệt" dữ liệu, nhưng **duyệt hai thứ hoàn toàn khác nhau** và
dùng cho **hai loại đối tượng khác nhau**. Nhầm lẫn giữa chúng là lỗi rất phổ biến, đặc biệt khi làm
với mảng và object.

**① `for...in` — duyệt KEY (tên thuộc tính) của object:**

`for...in` lặp qua **tất cả các key (property name) enumerable** của một object, **bao gồm cả thuộc
tính kế thừa** từ prototype chain. Biến lặp nhận **key ở dạng chuỗi**. Nó ra đời từ rất sớm (ES1) và
được thiết kế **cho object**.

```js
const obj = { name: "An", age: 25 };
for (const key in obj) {
  console.log(key, obj[key]); // "name" An → "age" 25  (key là CHUỖI)
}
```

Đặc điểm quan trọng: `for...in` **đi ngược lên prototype chain** → duyệt cả key kế thừa:

```js
const parent = { inherited: 1 };
const child = Object.create(parent);
child.own = 2;

for (const k in child) console.log(k);            // "own", "inherited"  (cả key kế thừa)
for (const k in child)                             // lọc chỉ key của chính object:
  if (Object.hasOwn(child, k)) console.log(k);     // "own"  (Object.hasOwn / hasOwnProperty)
```

> Thứ tự duyệt của `for...in` **không đảm bảo tuyệt đối** với key kiểu số (integer-like); vì vậy
> **không nên dùng `for...in` để lặp mảng** — vừa duyệt cả thuộc tính "lạ" gắn thêm vào mảng, vừa trả
> **index dạng chuỗi** (`"0"`, `"1"`...) chứ không phải giá trị.

**② `for...of` — duyệt VALUE của iterable:**

`for...of` (ES6) lặp qua **giá trị (value)** của một **iterable** — tức object có cài đặt
`Symbol.iterator`: **Array, String, Map, Set, arguments, NodeList, TypedArray, generator**. Biến lặp
nhận thẳng **giá trị**, **không** duyệt thuộc tính kế thừa.

```js
const arr = ["a", "b", "c"];
for (const v of arr) console.log(v);   // "a", "b", "c"  (VALUE trực tiếp)

for (const ch of "Hi") console.log(ch); // "H", "i"  (String cũng iterable)

const set = new Set([1, 1, 2]);
for (const v of set) console.log(v);    // 1, 2

const map = new Map([["a", 1], ["b", 2]]);
for (const [k, v] of map) console.log(k, v); // "a" 1 → "b" 2  (Map — for...in KHÔNG dùng được)
```

**Mấu chốt — `for...of` KHÔNG lặp được plain object** (object thường không phải iterable):

```js
const obj = { name: "An" };
// for (const v of obj) {}   // ❌ TypeError: obj is not iterable
// Cách đúng: chuyển object thành iterable bằng Object.keys/values/entries
for (const [k, v] of Object.entries(obj)) console.log(k, v); // "name" An
```

**③ So sánh trực tiếp trên mảng (rất hay được hỏi):**

```js
const arr = ["a", "b", "c"];
arr.extra = "x";                        // gắn thêm 1 property "lạ" vào mảng

for (const i in arr) console.log(i);    // "0","1","2","extra"  → KEY (gồm cả prop thêm)
for (const v of arr) console.log(v);    // "a","b","c"          → VALUE (đúng phần tử mảng)
```

**Bảng so sánh:**

| Tiêu chí | **`for...in`** | **`for...of`** |
|---|---|---|
| **Duyệt cái gì** | **Key** (tên thuộc tính, dạng chuỗi) | **Value** (giá trị) |
| **Dùng cho** | **Object** (mọi object) | **Iterable**: Array, String, Map, Set, generator... |
| **Plain object** | ✅ dùng được | ❌ `TypeError` (không iterable) — dùng `Object.entries` |
| **Thuộc tính kế thừa** | ✅ có (đi lên prototype chain) | ❌ không |
| **Trên mảng** | index dạng chuỗi + prop lạ (**không nên**) | phần tử mảng (**nên dùng**) |
| **Ra đời** | ES1 (cũ) | ES6 |
| **`break`/`continue`** | ✅ | ✅ (và có `for await...of` cho async) |

> Ghi nhớ mẹo: **`for...in` = `in`dex/key** (duyệt key), **`for...of` = value `of` iterable** (duyệt
> giá trị). Muốn vừa key vừa value của object → `Object.entries(obj)` kết hợp `for...of`. Khái niệm
> object vs iterable liên hệ 2 nhóm kiểu dữ liệu ở [câu 3](#q3).

### 🎯 Tóm tắt trả lời phỏng vấn

- **`for...in` duyệt KEY** (tên thuộc tính, dạng chuỗi) của **object**, **bao gồm cả thuộc tính kế
  thừa** từ prototype chain — dùng để lặp **object**.
- **`for...of` duyệt VALUE** của **iterable** (Array, String, Map, Set, generator...), trả thẳng giá
  trị, **không** đụng tới thuộc tính kế thừa.
- **`for...of` không lặp được plain object** (không phải iterable → `TypeError`) → phải dùng
  `Object.keys/values/entries`. Ngược lại `for...in` **không nên dùng cho mảng** (trả index dạng chuỗi,
  duyệt cả property "lạ", thứ tự không đảm bảo).
- **Khuyến nghị:** lặp **mảng/iterable → `for...of`**; lặp **key của object → `for...in`** (lọc bằng
  `Object.hasOwn`) hoặc `Object.entries` + `for...of`.
- **Mẹo nhớ:** `for...in` = key/index; `for...of` = value.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Map và Set trong JavaScript

### 📖 Giới thiệu kiến thức

**Map** và **Set** là hai cấu trúc dữ liệu (collection) mới của **ES6**, bổ sung cho `Object` và
`Array`. Cả hai đều **iterable** (duyệt được bằng `for...of` — xem [câu 5](#q5)), giữ **đúng thứ tự
thêm vào (insertion order)**, và có thuộc tính **`.size`**.

**① `Set` — tập hợp các giá trị KHÔNG trùng lặp (unique):**

`Set` lưu một tập giá trị mà **mỗi giá trị chỉ xuất hiện một lần** (tự loại trùng). Giá trị thuộc
**bất kỳ kiểu nào**. So sánh trùng lặp theo cơ chế **SameValueZero** (gần như `===`, nhưng coi `NaN`
bằng chính nó).

```js
const s = new Set([1, 2, 2, 3]); // trùng bị loại
s.add(4);            // thêm
s.has(2);            // true — kiểm tra tồn tại (O(1))
s.delete(3);         // xóa
s.size;              // 3
for (const v of s) console.log(v);   // duyệt giá trị
s.forEach((v) => console.log(v));

// Ứng dụng kinh điển: loại trùng trong mảng
const unique = [...new Set([1, 1, 2, 3, 3])]; // [1, 2, 3]
```

**② `Map` — tập cặp key–value, KEY LÀ BẤT KỲ KIỂU NÀO:**

`Map` lưu các cặp **key → value**, điểm mạnh nhất là **key có thể là bất kỳ kiểu gì** (object, function,
`NaN`, primitive...), khác hẳn `Object` chỉ nhận key là **string/symbol** (mọi key khác bị ép thành
chuỗi).

```js
const m = new Map();
const objKey = { id: 1 };
m.set("name", "An");       // key chuỗi
m.set(objKey, "metadata"); // key là OBJECT — Object thường không làm được
m.set(1, "number key");    // key số (Object sẽ ép thành "1")

m.get(objKey);   // "metadata"
m.has("name");   // true
m.delete(1);
m.size;          // 2
for (const [k, v] of m) console.log(k, v); // duyệt cặp key-value
```

**③ `Map` vs `Object` (câu hỏi kinh điển):**

| Tiêu chí | **`Map`** | **`Object`** |
|---|---|---|
| **Kiểu của key** | **Bất kỳ** (object, function, `NaN`, primitive) | Chỉ **string / symbol** (khác bị ép thành chuỗi) |
| **Thứ tự key** | **Đảm bảo** thứ tự thêm vào | Phần lớn theo thứ tự thêm, nhưng key kiểu số bị sắp riêng |
| **Kích thước** | **`.size`** trực tiếp | Phải `Object.keys(o).length` |
| **Duyệt** | Iterable — `for...of`, `forEach` thẳng | Cần `Object.keys/values/entries` |
| **Key mặc định** | **Không có** (sạch hoàn toàn) | Kế thừa từ prototype (`toString`, `constructor`...) → dễ xung đột |
| **Hiệu năng** | Tối ưu cho **thêm/xóa thường xuyên** | Hợp với record cố định |
| **JSON** | **Không** `JSON.stringify` trực tiếp | `JSON.stringify` được |

> Chọn: cần **key không phải chuỗi**, thêm/xóa nhiều, cần `.size` và duyệt gọn → **`Map`**. Cần dữ
> liệu dạng record cấu trúc cố định, serialize JSON → **`Object`**. Cần **tập hợp giá trị unique** →
> **`Set`** (thay cho mảng khi cần chống trùng & kiểm tra tồn tại nhanh).

**④ `WeakMap` / `WeakSet` — bản "yếu", thân thiện garbage collection:**

- **Key (WeakMap) / phần tử (WeakSet) BẮT BUỘC là object**, và được giữ **yếu (weak reference)**: nếu
  không còn tham chiếu nào khác tới object đó, nó **bị GC thu hồi tự động** (liên hệ Garbage Collection
  ở [câu 8](#q8)) → tránh **memory leak**.
- **Không iterable**, **không `.size`**, không `clear()` — vì nội dung có thể biến mất bất cứ lúc nào.
- Dùng để: gắn **metadata/cache riêng cho object** mà không cản trở việc giải phóng bộ nhớ.

```js
const wm = new WeakMap();
let user = { name: "An" };
wm.set(user, "dữ liệu phụ");
user = null; // không còn tham chiếu → entry trong WeakMap tự bị GC thu hồi
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Set** = tập giá trị **không trùng lặp** (bất kỳ kiểu), có `add/has/delete/size`, iterable. Ứng dụng
  kinh điển: **loại trùng mảng** `[...new Set(arr)]` và kiểm tra tồn tại **O(1)**.
- **Map** = tập cặp **key–value** mà **key là BẤT KỲ kiểu nào** (object, function, primitive...), giữ
  **thứ tự thêm vào**, có `.size`, duyệt bằng `for...of`.
- **Map vs Object:** Map cho key mọi kiểu / có `.size` / iterable trực tiếp / không key mặc định / tối
  ưu thêm-xóa; Object chỉ key string-symbol, cần `Object.keys`, có prototype, nhưng **JSON.stringify
  được**. → key phi-chuỗi & thêm/xóa nhiều dùng **Map**; record cố định & serialize dùng **Object**.
- **WeakMap/WeakSet:** key/phần tử **bắt buộc là object**, giữ **weak** → **không cản GC** (tránh memory
  leak), **không iterable, không `.size`**; dùng cho metadata/cache gắn theo object.
- Cả Map và Set đều **iterable** (`for...of`) và giữ **insertion order**.

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. Phân biệt anonymous function và named function trong JavaScript

### 📖 Giới thiệu kiến thức

Trong JavaScript có **3 cách** định nghĩa hàm, và chúng dẫn tới phân biệt named/anonymous:

- **Function declaration (khai báo hàm):** `function foo() {}` — **luôn có tên**.
- **Function expression (biểu thức hàm):** gán hàm vào biến — `const foo = function () {}` (**anonymous**)
  hoặc `const foo = function bar() {}` (**named function expression — NFE**).
- **Arrow function:** `const foo = () => {}` — **luôn là anonymous** (không có tên riêng).

**Named function** = hàm có **định danh** ngay sau từ khóa `function`. **Anonymous function** = hàm
**không có tên**, thường dùng làm **biểu thức/callback/đối số/IIFE**.

```js
// Named function declaration — có tên 'greet'
function greet(name) { return `Hi ${name}`; }

// Anonymous function expression — không tên, gán vào biến
const greet2 = function (name) { return `Hi ${name}`; };

// Named function expression (NFE) — tên 'factorial' chỉ thấy BÊN TRONG hàm
const fact = function factorial(n) {
  return n <= 1 ? 1 : n * factorial(n - 1);
};

// Arrow — luôn anonymous
const greet3 = (name) => `Hi ${name}`;
```

**Các khác biệt cốt lõi:**

**① Hoisting (nâng khai báo) — khác biệt quan trọng nhất:**
- **Function declaration** (named, là *statement*) được **hoist đầy đủ** → gọi được **trước** khi khai báo.
- **Function expression** (anonymous gán vào biến) **không** được hoist như hàm: chỉ **biến** được hoist
  (với `var` = `undefined` → gọi sẽ `TypeError`; với `let/const` rơi vào **TDZ** → `ReferenceError`).

```js
sayHi();                       // ✅ "Hi" — declaration hoist đầy đủ
function sayHi() { console.log("Hi"); }

sayBye();                      // ❌ TypeError: sayBye is not a function
var sayBye = function () { console.log("Bye"); };
```

**② Thuộc tính `.name`:**
- Named function: `.name` = tên thật.
- Anonymous gán vào biến: ES6 **suy ra (infer)** tên từ tên biến.
- Anonymous truyền trực tiếp (vd vào `forEach`): `.name === ""`.

```js
function foo() {}                 foo.name;          // "foo"
const bar = function () {};       bar.name;          // "bar" (suy từ biến)
[].forEach(function () {});       // hàm này .name === ""
```

**③ Đệ quy & tự tham chiếu (self-reference):**
- **Named function** (hoặc **NFE**) **tự gọi chính nó bằng tên** → an toàn cho đệ quy. Với NFE, tên chỉ
  **lộ trong scope của chính hàm** (bên ngoài không thấy) → tránh ô nhiễm scope ngoài và không sợ biến
  bị gán lại.
- **Anonymous** không có tên để tự gọi (chỉ dựa vào biến ngoài — mà biến có thể bị reassign).

```js
const f = function factorial(n) { return n <= 1 ? 1 : n * factorial(n - 1); };
// factorial(5);   // ❌ ReferenceError — tên không lộ ra ngoài
f(5);              // ✅ 120
```

**④ Debug / stack trace:**
- Named function hiện **tên có nghĩa** trong stack trace → dễ tìm lỗi.
- Anonymous thường hiện **`<anonymous>`** → khó debug.

**⑤ Trường hợp dùng:**
- **Anonymous:** callback, **IIFE**, event handler, các method mảng (`map`/`filter`/`forEach`), hàm
  dùng-một-lần ngắn gọn.
- **Named:** hàm **tái sử dụng**, **đệ quy**, cần rõ ràng và **stack trace dễ đọc**.

```js
setTimeout(function () { console.log("done"); }, 1000); // anonymous callback
(function () { console.log("IIFE"); })();               // IIFE
[1, 2, 3].map((x) => x * 2);                            // arrow callback
```

**Bảng so sánh:**

| Tiêu chí | **Named function** | **Anonymous function** |
|---|---|---|
| **Tên** | Có định danh sau `function` | Không có tên |
| **Dạng phổ biến** | Declaration `function foo(){}` | Expression/callback/IIFE/arrow |
| **Hoisting** | Declaration hoist **đầy đủ** (gọi trước được) | Không (theo biến: `var`=undefined / `let`,`const`=TDZ) |
| **`.name`** | Tên thật | Suy từ biến, hoặc `""` nếu truyền trực tiếp |
| **Đệ quy/tự gọi** | Dễ (gọi bằng tên) | Khó (phải dựa biến ngoài) |
| **Debug/stack trace** | Hiện tên rõ ràng | Hiện `<anonymous>` |
| **Hợp với** | Tái sử dụng, đệ quy, rõ ràng | Callback, IIFE, dùng một lần |

> Liên hệ: chi tiết hoisting & function declaration vs expression nằm trong vòng đời thực thi
> ([câu 8](#q8)); arrow function (luôn anonymous, không có `this` riêng) xem thêm ở [câu 2](#q2) (ES6).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Named function** có định danh sau `function` (vd `function foo(){}`); **anonymous** không có tên,
  thường làm **expression/callback/IIFE/đối số**. **Arrow function luôn là anonymous**.
- **Hoisting (khác biệt lớn nhất):** **function declaration** (named) được **hoist đầy đủ** → gọi được
  trước khi khai báo; **function expression** (anonymous gán vào biến) thì **không** (`var`=`undefined`
  → `TypeError`; `let/const` → **TDZ** → `ReferenceError`).
- **`.name`:** named có tên thật; anonymous gán vào biến được ES6 **suy ra** tên; truyền trực tiếp thì
  `.name === ""`.
- **Đệ quy:** named (hoặc **named function expression**) tự gọi bằng tên an toàn; NFE tên **chỉ lộ trong
  scope của chính nó**. Anonymous khó tự tham chiếu.
- **Debug:** named hiện tên trong **stack trace** → dễ debug; anonymous hiện `<anonymous>`.
- **Dùng:** anonymous cho **callback/IIFE/hàm một lần**; named cho **tái sử dụng, đệ quy, rõ ràng**.

[↑ Mục lục](#mục-lục)

<a id="q7-1"></a>
### 7.1. Phân biệt arrow function và regular function

#### 📖 Giới thiệu kiến thức

**Arrow function** (ES6) không chỉ là cú pháp ngắn gọn hơn **regular function** — nó khác về **ngữ
nghĩa** ở nhiều điểm quan trọng, đặc biệt là `this`.

**① `this` — khác biệt QUAN TRỌNG NHẤT:**
- **Regular function:** có **`this` riêng**, được xác định **động** theo **cách gọi** (call-site):
  gọi như method → `this` = object; gọi thường → `this` = `undefined` (strict) / `window` (non-strict);
  có thể đổi bằng `call`/`apply`/`bind`.
- **Arrow function:** **không có `this` riêng** → dùng **lexical `this`** (kế thừa `this` của scope bao
  quanh **lúc định nghĩa**). `call`/`apply`/`bind` **không** đổi được `this` của arrow.

```js
const obj = {
  name: "An",
  regular() { return this.name; },   // this = obj
  arrow: () => this.name,            // this = scope ngoài (KHÔNG phải obj)
};
obj.regular(); // "An"
obj.arrow();   // undefined
```

**Ứng dụng kinh điển — giữ `this` trong callback:**

```js
function Timer() {
  this.seconds = 0;
  setInterval(() => { this.seconds++; }, 1000); // ✅ arrow giữ this = instance Timer
  // setInterval(function () { this.seconds++; }, 1000); // ❌ this = window/undefined
}
```

**② `arguments` object:**
- Regular function có object **`arguments`** (danh sách đối số).
- Arrow function **không có** `arguments` → dùng **rest parameter** `(...args)` thay thế (hoặc lấy
  `arguments` của hàm bao ngoài).

```js
function reg() { return arguments.length; }   // ✅
const arrow = (...args) => args.length;        // arrow dùng rest
```

**③ Dùng làm constructor (`new`):**
- Regular function **dùng được với `new`** (là constructor, có `[[Construct]]`).
- Arrow function **không thể** dùng `new` → `TypeError` (không có `[[Construct]]`, không có `prototype`).

```js
function Person(n) { this.name = n; }
new Person("An");                  // ✅
const P = (n) => { this.name = n; };
// new P("An");                    // ❌ TypeError: P is not a constructor
```

**④ Thuộc tính `prototype`:** regular function **có** `prototype` (phục vụ `new`); arrow **không có**.

**⑤ Hoisting:** regular **declaration** được hoist đầy đủ (gọi trước được); arrow là **expression**
gán vào biến → không hoist (theo `var`/`let`/`const` — xem [câu 7](#q7)).

**⑥ Generator:** regular function dùng được `function*` + `yield`; **arrow không thể** là generator.

**⑦ Cú pháp ngắn + implicit return:** arrow cho phép **return ngầm** khi body 1 biểu thức.

```js
const sum = (a, b) => a + b;            // implicit return
const make = () => ({ ok: true });      // trả object phải bọc ()
```

**Khi nào KHÔNG nên dùng arrow:** làm **method của object** hoặc **method trên prototype** (vì `this`
sẽ không trỏ object/instance), **event handler** cần `this` = element DOM, hoặc khi cần `arguments`.

**Bảng so sánh:**

| Tiêu chí | **Regular function** | **Arrow function** |
|---|---|---|
| **`this`** | Riêng, **động** theo call-site | **Lexical** (kế thừa scope ngoài), không đổi được |
| **`call`/`apply`/`bind`** | Đổi được `this` | **Không** đổi được `this` |
| **`arguments`** | Có | **Không** (dùng rest `...args`) |
| **`new` (constructor)** | Được | **Không** (TypeError) |
| **`prototype`** | Có | Không |
| **Hoisting** | Declaration hoist đầy đủ | Không (là expression) |
| **Generator (`yield`)** | Được | Không |
| **Hợp với** | Method, constructor, cần `this` động | Callback, giữ `this` ngoài, hàm ngắn |

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Khác biệt lớn nhất là `this`:** regular function có **`this` riêng, xác định động theo cách gọi**
  (đổi được bằng `call/apply/bind`); arrow **không có `this` riêng** → dùng **lexical this** (kế thừa
  scope lúc định nghĩa), `bind` cũng không đổi được.
- Arrow **không có `arguments`** (dùng rest `...args`), **không dùng `new`** (không phải constructor,
  không có `prototype`), **không làm generator**.
- Arrow là **expression** → không hoisting như regular **declaration**; cú pháp ngắn + **implicit return**.
- **Dùng arrow** cho **callback / giữ `this` của scope ngoài** (vd `setInterval`, `map`); **dùng regular**
  cho **method object, constructor, prototype, event handler** cần `this` động.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. Vòng đời thực thi của JavaScript (JS lifecycle)

### 📖 Giới thiệu kiến thức

Có thể hiểu vòng đời thực thi của JavaScript qua **6 giai đoạn chính**:

```
1. Tải JavaScript
        ↓
2. Parse mã nguồn
        ↓
3. Tạo Execution Context
        ↓
4. Chạy code qua Call Stack
        ↓
5. Xử lý async qua Event Loop
        ↓
6. Thu hồi bộ nhớ bằng Garbage Collector
```

#### 1. Tải mã JavaScript

Trình duyệt gặp thẻ `<script src="app.js"></script>` thì tải file JS về. **Mặc định**, trình duyệt có
thể **tạm dừng phân tích HTML** để tải và thực thi JavaScript.

- **`defer`:** tải song song với HTML, **chạy sau khi HTML parse xong**, **giữ đúng thứ tự** các file.
- **`async`:** tải song song, **tải xong lúc nào chạy lúc đó**, **không đảm bảo thứ tự**.

```html
<script src="app.js" defer></script>
<script src="app.js" async></script>
```

#### 2. Phân tích cú pháp — Parsing

JavaScript Engine đọc mã nguồn và kiểm tra cú pháp. Lỗi cú pháp (vd `const name = ;`) khiến đoạn mã
**không thể thực thi** và sinh **`SyntaxError`**. Sau đó engine chuyển mã nguồn thành cấu trúc cây
**AST — Abstract Syntax Tree**.

```js
const name = "An";
console.log(name);
```

#### 3. Tạo Execution Context

Trước khi chạy code, JS tạo **môi trường thực thi (Execution Context)**. Có 3 loại phổ biến:
**Global**, **Function**, và **Eval** Execution Context.

```js
const x = 10;
function calculate() {
  const y = 20;
  return x + y;
}
calculate();
// Global Execution Context → calculate() Function Execution Context
```

Mỗi execution context thường chứa: **biến và hàm**, **scope chain**, giá trị **`this`**, và **tham
chiếu đến môi trường bên ngoài**.

**Hai giai đoạn của Execution Context:**

**Giai đoạn 1 — Creation Phase:** JS **đăng ký biến và hàm trước** khi chạy từng dòng (hiện tượng
**hoisting**).

```js
console.log(name);  // undefined
var name = "Nam";
// Được hiểu gần giống:  var name;  console.log(name);  name = "Nam";
```

Với **function declaration**, hàm có thể được gọi **trước** vị trí khai báo:

```js
sayHello();                         // "Hello"
function sayHello() { console.log("Hello"); }
```

Nhưng **`let` và `const`** nằm trong **Temporal Dead Zone (TDZ)**:

```js
console.log(name);  // ❌ ReferenceError
let name = "Nam";
```

**Giai đoạn 2 — Execution Phase:** JS chạy từng câu lệnh **theo thứ tự từ trên xuống**.

```js
const a = 10;
const b = 20;
const result = a + b;
console.log(result);
// Gán a = 10 → gán b = 20 → tính result = 30 → in result
```

#### 4. Call Stack

JS dùng **Call Stack** để quản lý các hàm đang thực thi, theo cơ chế **LIFO — Last In, First Out**
(hàm vào sau kết thúc trước).

```js
function first()  { second(); }
function second() { third(); }
function third()  { console.log("Done"); }
first();

/* Call Stack thay đổi:
Global
Global → first
Global → first → second
Global → first → second → third
→ third pop → second pop → first pop
*/
```

#### 5. Xử lý bất đồng bộ qua Event Loop

JS về cơ bản chỉ có **một luồng thực thi chính**. Các tác vụ như `setTimeout`, gọi API, sự kiện click,
đọc file, `Promise`... được **môi trường bên ngoài** JS Engine (Web APIs / Node APIs) hỗ trợ xử lý.

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
console.log("C");
// Kết quả: A → C → B
```

```
Call Stack → Web APIs / Node APIs → Task Queue → Event Loop → Call Stack
```

`setTimeout(..., 0)` **không** chạy ngay, mà callback chỉ được đưa vào hàng đợi **sau ít nhất 0ms**.

**Microtask và Macrotask:** `Promise` callback thuộc **Microtask Queue**; `setTimeout` thuộc
**Macrotask Queue** (còn gọi Task Queue).

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Kết quả: 1 → 4 → 3 → 2
```

Thứ tự ưu tiên: **1. Code đồng bộ → 2. Microtask Queue → 3. Macrotask Queue.**

- **Microtask:** `Promise.then/catch/finally`, `queueMicrotask()`.
- **Macrotask:** `setTimeout()`, `setInterval()`, DOM events, I/O callbacks.

#### 6. Quản lý bộ nhớ và Garbage Collection

JS **tự động cấp phát bộ nhớ** khi tạo biến/object. Khi object **không còn được tham chiếu**, Garbage
Collector có thể thu hồi vùng nhớ.

```js
let user = { name: "Nam", age: 25 };
user = null; // object cũ không còn tham chiếu → có thể bị GC thu hồi
```

JS xác định object còn dùng hay không dựa trên **khả năng truy cập từ các "root"**: biến global, call
stack, closure đang hoạt động, DOM đang được tham chiếu.

Ví dụ dễ gây **rò rỉ bộ nhớ (memory leak)** — nếu phần tử/component bị xóa nhưng listener, timer hoặc
tham chiếu vẫn tồn tại:

```js
const button = document.querySelector("#button");
button.addEventListener("click", handleClick);
// xóa button mà không removeEventListener → listener + tham chiếu còn → chưa giải phóng
```

**Ví dụ tổng hợp:**

```js
console.log("Start");
setTimeout(() => console.log("Timeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
function test() { console.log("Function"); }
test();
console.log("End");

/* Kết quả:
Start
Function
End
Promise
Timeout
*/
```

Vì thứ tự thực thi là: **Code đồng bộ → Microtask → Macrotask**.

> **Trong Node.js:** nguyên lý chung vẫn vậy, nhưng Event Loop được chia thành nhiều **phase** (timers,
> poll, check, close callbacks) và có thêm **`process.nextTick()`** với độ ưu tiên riêng — chi tiết ở
> [nodejs.md câu 2](./nodejs.md#q2).

### 🎯 Tóm tắt trả lời phỏng vấn

- Vòng đời thực thi JS gồm **6 giai đoạn:** ① **Tải** JS (`defer` chạy sau parse & giữ thứ tự;
  `async` xong lúc nào chạy lúc đó) → ② **Parse** (dựng **AST**, lỗi → `SyntaxError`) → ③ tạo
  **Execution Context** → ④ chạy qua **Call Stack** → ⑤ **async qua Event Loop** → ⑥ **Garbage Collection**.
- **Execution Context có 2 pha:** **Creation** (đăng ký trước biến/hàm → **hoisting**: `var`=`undefined`,
  function declaration gọi được trước, `let/const` vào **TDZ**) rồi **Execution** (chạy từng dòng).
- **Call Stack** đơn luồng, **LIFO** — hàm vào sau kết thúc trước.
- **Async:** tác vụ async giao cho Web/Node APIs; thứ tự ưu tiên **đồng bộ → Microtask (`Promise`) →
  Macrotask (`setTimeout`)**. `setTimeout(0)` chỉ là "vào hàng đợi sau ≥ 0ms", không chạy ngay.
- **Garbage Collection:** object **không còn truy cập được từ root** (global, call stack, closure, DOM)
  sẽ bị thu hồi; coi chừng **memory leak** từ listener/timer/tham chiếu còn sót.
- **Node.js** cùng nguyên lý nhưng Event Loop chia **nhiều phase** + có **`process.nextTick`**
  ([nodejs.md câu 2](./nodejs.md#q2)).

[↑ Mục lục](#mục-lục)

<a id="q8-1"></a>
### 8.1. Các thành phần của cơ chế bất đồng bộ: Execution Context, Call Stack, Task Queue, Web APIs, Event Loop

#### 📖 Giới thiệu kiến thức

Đây là phần **đào sâu** cơ chế đã nêu ở [câu 8](#q8): để hiểu vì sao JS đơn luồng vẫn xử lý được bất
đồng bộ, cần nắm **5 mảnh ghép** sau và cách chúng phối hợp.

**a. Execution Context (ngữ cảnh thực thi)**

Hiểu đơn giản, execution context là **môi trường mà một đoạn code chạy bên trong** — nó chứa code đó
cùng mọi thứ liên quan mà code cần (biến cục bộ, tham chiếu hàm con, `this`, scope chain...). Tưởng
tượng nó như **một "cái hộp" bọc quanh một function**.

Có **3 loại**:
- **Global context:** context "gốc", giống hàm `main` trong C/C++. Được tạo **đầu tiên** và đẩy vào
  call stack đầu tiên. JS **không có hàm `main`** nhưng ngầm định luôn có global context.
- **Function context:** context bọc một function. **Mỗi lần gọi hàm tạo ra một context mới** — gọi
  hàm 3 lần thì có 3 context (đây là nền tảng của đệ quy và closure).
- **Eval context:** context cho code trong `eval()`. **Bị tránh dùng** vì không an toàn.

**b. Call Stack (ngăn xếp lời gọi)**

Stack là cấu trúc **LIFO (Last In First Out)** — vào sau ra trước, như **chồng đĩa** (đĩa đặt cuối lấy
ra đầu). **Call stack** là một stack chứa các execution context; **chỉ context ở đỉnh (top) mới đang
được thực thi**.

Quy trình:
1. Chương trình chạy → **global context** được đẩy vào đầu tiên và chạy.
2. Gặp lời gọi hàm → tạo context của hàm đó và **đẩy ngay vào đỉnh** → context đang chạy tạm dừng,
   hàm mới (ở đỉnh) bắt đầu chạy.
3. Hàm này gọi hàm khác → lặp lại, stack cao dần.
4. Hàm chạy xong → context của nó **bị pop khỏi stack** → context kế tiếp (giờ ở đỉnh) chạy tiếp.
5. Cứ vậy tới khi global context bị pop → **call stack rỗng**.

Vì **JS đơn luồng → chỉ có MỘT call stack** → tại một thời điểm **chỉ một context ở đỉnh được chạy**.
Hệ quả: một hàm chạy lâu (CPU nặng) sẽ **chặn (block)** toàn bộ stack — không gì khác chạy được.

**c. Task Queue & Job Queue (hàng đợi)**

Queue là cấu trúc **FIFO (First In First Out)** — vào trước ra trước, như **xếp hàng**.

- **Task queue (callback queue / macrotask queue):** chứa callback của các hàm bất đồng bộ "thường"
  (vd `setTimeout`, sự kiện DOM, I/O). Khi Web API xử lý xong, callback được đẩy vào đây.
- **Job queue (microtask queue):** hàng đợi **thứ hai**, dành cho callback của **Promise**
  (`.then/.catch/.finally`, `queueMicrotask`). **Độ ưu tiên cao hơn** task queue → job được chạy
  trước callback thường (xem [câu 9.2](#q9-2) về Promise).

Event loop chỉ đẩy callback từ queue vào call stack **khi stack đã rỗng**, mỗi lần một callback.

**d. Web APIs & Node C++ APIs**

Đây là các API **do môi trường host cung cấp**, **không thuộc bản thân ngôn ngữ JS**:
- **Browser — Web APIs:** `setTimeout`, DOM, `fetch`/AJAX, `addEventListener`...
- **Node.js — C++ APIs (qua libuv):** đọc/ghi file (`fs`), network, query database...

Khi JS gọi một API bất đồng bộ kèm callback (hoặc Promise), **việc xử lý được giao cho host** (browser/
Node), **không chạy trên call stack của JS**. Xử lý xong, callback tương ứng được đẩy vào **task queue**
(hoặc **job queue** nếu là Promise).

Hai điểm rất hay được hỏi:
- **JS đơn luồng nhưng Web APIs thì không:** browser/Node thực thi các API này trên **nhiều thread**
  của riêng nó. Nên "JS đơn luồng" chỉ nói về **code JS của bạn**, còn tác vụ nền (tải file, timer,
  network) chạy song song bên dưới.
- **Thứ tự đẩy vào queue phụ thuộc task nào XONG trước, không phải gọi trước:** `setTimeout(fn, 1000)`
  dù gọi **sau** `setTimeout(fn, 3000)` vẫn có callback vào queue **trước** (vì hết hạn sớm hơn).

**e. Event Loop (vòng lặp sự kiện)**

Event loop là một **vòng lặp vô tận do host cung cấp** (browser, hoặc **libuv** ở Node) — **không phải
nằm trong V8**; V8 chỉ là engine biên dịch & chạy JS. Nhiệm vụ của nó rất đơn giản, lặp mãi:

1. **Kiểm tra call stack có rỗng không.**
2. Nếu **rỗng** → lấy callback từ queue đẩy vào stack để chạy.
3. Callback chạy xong, stack lại rỗng → lấy callback tiếp theo... cứ thế.

Quy tắc ưu tiên: **vét sạch job queue (microtask) trước**, rồi mới lấy **một** task (macrotask) từ task
queue. Giữa các macrotask (sau khi đã vét microtask), **trình duyệt có thể chen vào để render UI**.
Khi stack rỗng và mọi queue rỗng, **event loop vẫn không dừng** — nó tiếp tục chờ task mới (vd click,
timer) → đây là lý do trang web "sống" và phản hồi sự kiện liên tục.

**Bức tranh tổng thể:**

```
   ┌─────────────┐  gọi API async   ┌────────────────────┐
   │  Call Stack │ ───────────────▶ │  Web APIs / libuv  │  (đa luồng, xử lý nền)
   │ (1 luồng JS)│                  └─────────┬──────────┘
   └──────▲──────┘                            │ xong → đẩy callback
          │ stack rỗng → đẩy 1 callback        ▼
   ┌──────┴───────────────────────────────────────────────┐
   │  Event Loop:  microtask (Job Queue) ƯU TIÊN > macrotask (Task Queue)  │
   └───────────────────────────────────────────────────────┘
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Execution context** = "cái hộp" môi trường mà code chạy bên trong (biến, `this`, scope). 3 loại:
  **Global** (tạo đầu tiên, như `main`), **Function** (mỗi lần gọi hàm tạo 1 context mới), **Eval** (tránh dùng).
- **Call Stack** = stack **LIFO** chứa các context; **chỉ context ở đỉnh chạy**. JS đơn luồng → **một
  call stack** → hàm nặng sẽ **block** mọi thứ.
- **Queue**: **Task queue (macrotask)** chứa callback `setTimeout`/DOM/I/O; **Job queue (microtask)**
  chứa callback **Promise**, **ưu tiên cao hơn**. Event loop chỉ đẩy callback vào stack **khi stack rỗng**.
- **Web APIs / Node C++ APIs** do **host cung cấp** (không thuộc JS), chạy tác vụ nền **đa luồng** rồi
  đẩy callback về queue. ⇒ **JS đơn luồng nhưng Web APIs đa luồng**. Thứ tự vào queue theo **task nào
  xong trước**, không theo thứ tự gọi.
- **Event Loop** = vòng lặp vô tận **của host (browser/libuv), KHÔNG nằm trong V8**: stack rỗng thì lấy
  callback từ queue đẩy vào — **vét hết microtask trước, rồi 1 macrotask**, render UI xen giữa. Hết
  queue vẫn **không dừng**, tiếp tục chờ sự kiện mới.

[↑ Mục lục](#mục-lục)

<a id="q9"></a>
## 9. Bất đồng bộ (asynchronous) trong JavaScript là gì?

### 📖 Giới thiệu kiến thức

**Đồng bộ (synchronous):** các lệnh chạy **tuần tự**, lệnh trước xong thì lệnh sau mới chạy. Khi gặp
tác vụ tốn thời gian (đọc file, query DB, gọi network...), chương trình **phải đứng chờ** → gây
**blocking** (UI treo, "Not responding").

**Bất đồng bộ (asynchronous):** khi gặp tác vụ tốn thời gian, chương trình **không đứng chờ** mà chạy
tiếp các lệnh sau; khi tác vụ kia xong sẽ có **cơ chế thông báo lại** (callback). Đây là **non-blocking**.

> **Lưu ý quan trọng (rất hay bị hiểu sai):** bất đồng bộ **không làm code chạy nhanh hơn** — nó chỉ
> **tránh lãng phí thời gian chờ** tác vụ I/O bằng cách làm việc khác trong lúc chờ.

**JavaScript có thật sự "bất đồng bộ"?** Bản thân **JS là đơn luồng (single-threaded) và đồng bộ**. Bạn
**không thể** tự viết một hàm JS thuần và bắt nó chạy non-blocking — một vòng lặp vô tận luôn làm
blocking. Tính bất đồng bộ **đến từ môi trường host**, không phải ngôn ngữ:
- **Trình duyệt:** cung cấp **Web APIs** (`setTimeout`, `fetch`/AJAX, DOM events...).
- **Node.js:** cung cấp **C++/libuv APIs** (file system, network, timer...).

Callback/Promise/async-await **không tạo ra** sự bất đồng bộ — chúng chỉ là **kỹ thuật để xử lý kết
quả** của các tác vụ async (do host thực thi) một cách gọn gàng, không bị blocking.

**Cơ chế chạy (nối với [câu 8](#q8) — event loop):**

```
                   ┌─────────────┐
   gọi hàm async   │  Web APIs / │   tác vụ xong
 ───────────────►  │  libuv      │ ─────────────┐
                   │ (đa luồng,  │              │ đẩy callback
 Call Stack        │  do host)   │              ▼
 (đơn luồng,       └─────────────┘     ┌──────────────────────┐
  LIFO)                                │ Job/Microtask Queue   │ ◄─ Promise (ưu tiên cao)
     ▲                                 ├──────────────────────┤
     │   Event Loop: stack rỗng        │ Task/Macrotask Queue  │ ◄─ setTimeout, I/O...
     └──── thì lấy từ queue ◄──────────┴──────────────────────┘
```

1. Gọi hàm async (Web API) kèm callback → giao cho host xử lý (ngoài call stack).
2. Host xử lý xong → đẩy callback vào **queue**: Promise → **microtask (job) queue**; `setTimeout`/I/O
   → **macrotask (task) queue**.
3. **Event Loop:** khi **call stack rỗng**, lấy callback từ queue đẩy vào stack chạy. **Microtask ưu
   tiên hơn macrotask** (xem chi tiết thứ tự ở [câu 8 mục 5](#q8) & [nodejs.md câu 2](./nodejs.md#q2)).

**Khi code JS quá nặng (CPU-bound) thì sao?** Async không giải quyết được vì JS vẫn đơn luồng → dùng
**Web Worker** (trình duyệt) / **Worker Threads** (Node — xem [nodejs.md câu 3](./nodejs.md#q3)) để
chạy trên thread riêng. (Nhược điểm: worker **không truy cập được DOM**, `window`, `document`.)

**3 kỹ thuật xử lý bất đồng bộ** (tiến hóa theo thời gian, giải quyết nhược điểm của nhau):
**Callback** ([8.1](#q9-1)) → **Promise** ([8.2](#q9-2)) → **async/await** ([8.4](#q9-4)).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Đồng bộ** = chạy tuần tự, tác vụ nặng gây **blocking**. **Bất đồng bộ** = không đứng chờ tác vụ
  I/O, làm việc khác rồi nhận thông báo khi xong → **non-blocking**. Async **không làm code nhanh hơn**,
  chỉ tránh lãng phí thời gian chờ.
- **Bản thân JS là đơn luồng & đồng bộ.** Tính bất đồng bộ đến từ **host** (Web APIs của trình duyệt /
  libuv của Node), **không** từ ngôn ngữ. Callback/Promise/async-await chỉ là **cách xử lý kết quả**
  async cho gọn, không tự tạo ra async.
- **Cơ chế:** hàm async → host xử lý (ngoài stack) → callback vào **queue** → **Event Loop** đưa lại
  call stack khi rỗng; **microtask (Promise) ưu tiên hơn macrotask (setTimeout/I/O)**.
- **CPU-bound** không cứu được bằng async → dùng **Web Worker / Worker Threads** (thread riêng, không
  truy cập DOM).
- 3 kỹ thuật: **Callback → Promise → async/await** (mỗi cái khắc phục nhược điểm cái trước).

[↑ Mục lục](#mục-lục)

<a id="q9-1"></a>
### 9.1. Callback là gì? Callback hell?

#### 📖 Giới thiệu kiến thức

**Callback** là kỹ thuật **truyền một hàm vào một hàm khác làm tham số**, để hàm kia gọi lại nó khi
chạy xong. Với async, **async callback** đảm bảo đoạn code chỉ chạy **sau khi** tác vụ bất đồng bộ
hoàn thành:

```js
setTimeout(function () {
  console.log("Done"); // chạy SAU 1 giây, khi Web API setTimeout xong
}, 1000);
```

**Error handling:** các API async theo kiểu callback thường báo lỗi qua **tham số của callback** (vd
`error`-first callback trong Node, hoặc `status` trong jQuery):

```js
$.get("index.html", function (data, status) {
  if (status === "success") {
    // xử lý data
  } else {
    console.log("Error");
  }
});
```

**Callback hell:** khi cần làm **nhiều tác vụ async liên tiếp**, callback bị **lồng vào nhau** ngày
càng sâu → code khó đọc, khó bắt lỗi (còn gọi *pyramid of doom*):

```js
$.get("index.html", function () {
  $.get("style.css", function () {
    $.get("script.js", function () {
      // ... càng nhiều file càng lồng sâu
    });
  });
});
```

→ Để giải quyết callback hell, JS đưa ra **Promise** ([câu 9.2](#q9-2)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Callback** = truyền hàm vào hàm khác để được gọi lại khi xong; **async callback** đảm bảo code chạy
  **sau** khi tác vụ async hoàn thành.
- Lỗi thường báo qua **tham số callback** (error-first / status).
- **Callback hell** = nhiều callback async lồng nhau → khó đọc, khó bắt lỗi → động lực ra đời
  **Promise**.

[↑ Mục lục](#mục-lục)

<a id="q9-2"></a>
### 9.2. Promise là gì? States, then/catch/finally, chaining

#### 📖 Giới thiệu kiến thức

**Promise** là **object đại diện cho kết quả của một tác vụ sẽ hoàn thành trong tương lai** — thành
công (**resolve**) hoặc thất bại (**reject**). Nó giải quyết callback hell.

**3 trạng thái (state):**
- **Pending:** đang thực hiện, chưa xong.
- **Fulfilled:** xong, **thành công** (đã gọi `resolve`).
- **Rejected:** xong, **thất bại** (đã gọi `reject`).
- *(**Settled** = đã xong, gộp chung fulfilled + rejected.)*

**Cấu trúc — executor:** constructor `Promise` nhận một hàm **executor** với 2 tham số `resolve`,
`reject`. Promise **chạy executor ngay khi tạo**; thường gọi `resolve()`/`reject()` **bên trong** tác
vụ async (đặt ngoài thì promise kết thúc ngay trước khi async xong):

```js
let p = new Promise((resolve, reject) => {
  $.get("camly.mp3", (data, status) => {
    if (status === "success") resolve(data); // thành công → gửi data
    else reject(status);                      // thất bại → gửi lỗi
  });
});

p.then((data) => console.log("Đã nhận data", data)) // callback khi resolve
 .catch((err) => console.log("Lỗi", err));          // callback khi reject
```

> **Vì sao Promise hoạt động đúng?** Callback resolve/reject được đẩy vào **job (microtask) queue** —
> luôn chạy **sau cùng** (khi call stack rỗng). Nhờ đó vẫn kịp gắn `.then()/.catch()` trước khi callback
> chạy, và đảm bảo chúng chỉ chạy **sau khi** tác vụ async xong.

**`then` / `catch` / `finally`:**
- `then(cb)`: gắn callback cho trường hợp **thành công**.
- `catch(cb)`: gắn callback cho trường hợp **thất bại**.
- `finally(cb)`: **luôn** chạy khi promise **settled** (dù thành công hay thất bại).

**Trả về promise từ hàm** (để tái dùng):

```js
function downloadFile(url) {
  return new Promise((resolve, reject) => {
    $.get(url, (data, status) =>
      status === "success" ? resolve(data) : reject(status));
  });
}
```

**Promise chaining (giải quyết callback hell):** `then()` **trả về một promise mới**, nên có thể **nối
chuỗi** các `then()` ở **cùng cấp** (không lồng nhau). Mấu chốt: **`return` một promise** trong `then`
để mắt xích kế tiếp chờ nó:

```js
downloadFile("camly.mp3")
  .then(() => {
    console.log("Tải xong camly.mp3");
    return downloadFile("camly.zip"); // return → then kế tiếp chờ promise này
  })
  .then(() => console.log("Tải xong camly.zip"))
  .catch((err) => console.log("Lỗi:", err)); // 1 catch ở cuối bắt mọi lỗi
```

**Các static method hữu ích:**
- `Promise.resolve(v)` / `Promise.reject(e)`: tạo nhanh promise đã fulfilled/rejected (tiện truyền giá
  trị qua mắt xích `then`).
- `Promise.all([...])`: chờ **tất cả** thành công (1 cái fail → reject ngay).
- `Promise.allSettled([...])`: chờ **tất cả settled**, trả kết quả từng cái (không reject sớm).
- `Promise.race([...])`: lấy kết quả của promise **xong đầu tiên** (kể cả lỗi).
- `Promise.any([...])`: lấy promise **thành công đầu tiên**.

→ Khi nối **quá nhiều** `then()` vẫn dài dòng (*promise hell*) → dùng **async/await** ([câu 9.4](#q9-4)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Promise** = object đại diện kết quả tương lai của tác vụ async: **pending → fulfilled (resolve) /
  rejected (reject)** (xong = **settled**).
- **Executor** `(resolve, reject)` chạy **ngay khi tạo**; gọi `resolve`/`reject` **trong** tác vụ async.
  Callback được đưa vào **microtask queue** nên chạy sau cùng → kịp gắn `then`/`catch`.
- `then` (thành công) / `catch` (thất bại) / `finally` (luôn chạy khi settled).
- **Chaining** khắc phục callback hell: `then` trả về promise mới → **`return` promise** để nối chuỗi
  cùng cấp, một `catch` cuối bắt mọi lỗi.
- Static: `Promise.all` (chờ tất cả), `allSettled`, `race` (xong đầu tiên), `any` (thành công đầu tiên).

[↑ Mục lục](#mục-lục)

<a id="q9-3"></a>
### 9.3. Phân biệt Promise.all, Promise.race, Promise.allSettled và Promise.any

#### 📖 Giới thiệu kiến thức

Đây là **4 static method gộp nhiều promise** (combinators) — nhận vào **một mảng (iterable) promise**
và trả về **một promise mới**. Khác nhau ở **điều kiện kết thúc** và **giá trị trả về**. Dùng khi cần
chạy nhiều tác vụ async **song song** rồi tổng hợp kết quả.

**1. `Promise.all` — "tất cả phải thành công":**
- **Resolve** khi **mọi** promise fulfilled → trả về **mảng kết quả** (đúng thứ tự input).
- **Reject ngay** (fail-fast) khi **bất kỳ** promise nào reject → trả về lỗi đầu tiên đó (các promise
  khác vẫn chạy nhưng kết quả bị bỏ).
- Dùng khi: cần **tất cả** dữ liệu mới xử lý được (vd gọi 3 API rồi gộp).

**2. `Promise.allSettled` — "chờ tất cả xong, không quan tâm thành/bại":** (ES2020)
- **Luôn resolve** khi **mọi** promise **settled** (không bao giờ reject).
- Trả về mảng object: `{ status: "fulfilled", value }` hoặc `{ status: "rejected", reason }`.
- Dùng khi: muốn chạy hết và biết **từng cái** ra sao (vd gửi nhiều request, cái nào lỗi thì báo riêng).

**3. `Promise.race` — "ai xong trước (thắng/thua) thì lấy":**
- **Settle ngay** theo promise **đầu tiên settled** — dù **fulfilled hay rejected**.
- Dùng khi: **timeout** (đua tác vụ với một promise hẹn giờ), lấy nguồn phản hồi nhanh nhất.

**4. `Promise.any` — "thành công đầu tiên":** (ES2021)
- **Resolve** theo promise **đầu tiên fulfilled** (bỏ qua các cái reject).
- **Reject** chỉ khi **tất cả** đều reject → trả về **`AggregateError`** (gộp mọi lỗi).
- Dùng khi: chỉ cần **một** nguồn thành công (vd thử nhiều mirror, cái nào OK trước thì dùng).

```js
const ok   = (v, ms) => new Promise((res) => setTimeout(() => res(v), ms));
const fail = (e, ms) => new Promise((_, rej) => setTimeout(() => rej(e), ms));

// all: cần tất cả thành công
await Promise.all([ok("A", 10), ok("B", 20)]);        // ["A", "B"]
await Promise.all([ok("A", 10), fail("X", 5)]);       // ❌ reject "X" (fail-fast)

// allSettled: chờ hết, báo từng cái
await Promise.allSettled([ok("A", 10), fail("X", 5)]);
// [{status:"fulfilled",value:"A"}, {status:"rejected",reason:"X"}]

// race: cái đầu tiên settle (kể cả lỗi)
await Promise.race([ok("A", 50), fail("X", 10)]);     // ❌ reject "X" (xong trước)

// any: cái fulfilled đầu tiên
await Promise.any([fail("X", 10), ok("B", 50)]);      // "B" (bỏ qua reject)
await Promise.any([fail("X", 10), fail("Y", 20)]);    // ❌ AggregateError
```

**Bảng so sánh:**

| Method | Resolve khi | Reject khi | Giá trị trả về |
|---|---|---|---|
| **`all`** | **Tất cả** fulfilled | **Bất kỳ** reject (fail-fast) | Mảng kết quả |
| **`allSettled`** | **Tất cả** settled (luôn resolve) | **Không bao giờ** | Mảng `{status, value/reason}` |
| **`race`** | Cái **đầu tiên** fulfilled | Cái **đầu tiên** reject | Giá trị/lỗi của cái settle đầu tiên |
| **`any`** | Cái **đầu tiên** fulfilled | **Tất cả** reject | Giá trị; lỗi = `AggregateError` |

> **Mẹo nhớ:** `all` = **AND** (mọi cái thành công); `any` = **OR** (một cái thành công); `race` =
> **cái đầu tiên** xong (bất kể thành/bại); `allSettled` = **chờ hết**, không reject. *(Lưu ý: cả 4 đều
> chạy các promise **song song** ngay khi tạo — chúng chỉ khác cách **chờ & gộp** kết quả.)*

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả 4 nhận **mảng promise**, chạy **song song**, trả về **một promise** — khác ở điều kiện kết thúc:
- **`all`** = chờ **tất cả thành công**, trả mảng kết quả; **reject ngay** nếu một cái fail (fail-fast).
- **`allSettled`** = chờ **tất cả settled**, **luôn resolve**, trả `{status, value/reason}` từng cái.
- **`race`** = lấy cái **settle đầu tiên** (thành công **hoặc** thất bại) → hợp làm **timeout**.
- **`any`** = lấy cái **fulfilled đầu tiên**, bỏ qua reject; reject chỉ khi **tất cả** fail →
  **`AggregateError`**.
- Mẹo: **`all`=AND, `any`=OR, `race`=cái đầu tiên, `allSettled`=chờ hết không reject**.

[↑ Mục lục](#mục-lục)

<a id="q9-4"></a>
### 9.4. async/await là gì?

#### 📖 Giới thiệu kiến thức

**`async/await`** (ES2017) là **đường cú pháp (syntactic sugar) trên Promise**, giúp viết code bất đồng
bộ **trông như đồng bộ** → dễ đọc, hợp tư duy tuần tự, tránh *promise hell*.

- **`async`**: đặt trước khai báo hàm → hàm **luôn trả về một Promise**; `return value` trở thành data
  mà promise resolve.
- **`await`**: đặt trước một promise (hoặc hàm async) → **tạm dừng** luồng trong hàm cho tới khi promise
  có kết quả, rồi **trả về giá trị đã resolve**. Chỉ dùng được **bên trong hàm `async`** (hoặc top-level
  trong ES module).

```js
async function getValue() {
  return 10;                 // tương đương Promise.resolve(10)
}
getValue().then((v) => console.log(v)); // 10
```

**So sánh với Promise chaining** — cùng tải 3 file tuần tự:

```js
// Promise chaining (dễ thành "promise hell")
function downloadAll() {
  downloadFile("index.html")
    .then(() => { console.log("Xong index.html"); return downloadFile("style.css"); })
    .then(() => { console.log("Xong style.css");  return downloadFile("script.js"); })
    .then(() => console.log("Xong script.js"));
}

// async/await — gọn & giống code đồng bộ
async function downloadAll() {
  const f1 = await downloadFile("index.html"); console.log("Xong index.html");
  const f2 = await downloadFile("style.css");  console.log("Xong style.css");
  const f3 = await downloadFile("script.js");  console.log("Xong script.js");
}
```

**Error handling:** dùng `try/catch/finally` như code đồng bộ (thay cho `.catch()`):

```js
async function downloadAll() {
  try {
    const f1 = await downloadFile("index.html");
    const f2 = await downloadFile("style.css");
  } catch (err) {
    console.log("Lỗi:", err); // bắt được mọi promise reject ở trên
  }
}
```

**Lưu ý khi dùng:**
- `await` **chặn các lệnh phía sau trong hàm** để chờ promise → nếu các tác vụ **độc lập**, dùng
  `await` tuần tự gây chậm; nên chạy song song bằng `Promise.all([...])` rồi `await`.
- Chỉ là sugar trên Promise — **không thay đổi** bản chất event loop. Trình duyệt rất cũ không hỗ trợ →
  cần **Babel** transpile về Promise.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`async/await`** = syntactic sugar trên **Promise**, viết async **trông như đồng bộ** → dễ đọc, tránh
  promise hell.
- **`async`** làm hàm **luôn trả Promise** (`return` = data resolve); **`await`** tạm dừng trong hàm chờ
  promise rồi lấy giá trị resolve — chỉ dùng trong hàm `async`.
- **Bắt lỗi** bằng `try/catch/finally` (thay cho `.catch()`).
- **Lưu ý:** `await` tuần tự **chặn** lệnh sau → tác vụ độc lập nên dùng **`Promise.all`** cho song song;
  bản chất vẫn là Promise (cần Babel cho trình duyệt cũ).

[↑ Mục lục](#mục-lục)

<a id="q9-5"></a>
### 9.5. Phân biệt callback, Promise và async/await

#### 📖 Giới thiệu kiến thức

Cả ba đều là **kỹ thuật xử lý kết quả của tác vụ bất đồng bộ** (không tự tạo ra async — xem
[câu 9](#q9)). Chúng **tiến hóa nối tiếp nhau**, mỗi cái sinh ra để khắc phục nhược điểm của cái trước:
**Callback → Promise → async/await**.

**1. Callback** ([câu 9.1](#q9-1)) — truyền hàm vào hàm async để được gọi lại khi xong.
- ✅ Đơn giản, có từ đầu, không cần cú pháp đặc biệt.
- ❌ Nhiều tác vụ liên tiếp → **callback hell** (lồng sâu, khó đọc); **bắt lỗi rời rạc** (mỗi callback
  tự kiểm tra); khó kiểm soát luồng (chạy song song/tuần tự).

**2. Promise** ([câu 9.2](#q9-2)) — object đại diện kết quả tương lai (pending → fulfilled/rejected).
- ✅ **Chaining `.then()`** phẳng, không lồng; **một `.catch()`** bắt lỗi cả chuỗi; có **`Promise.all/
  race/any/allSettled`** điều phối nhiều tác vụ; callback vào **microtask queue**.
- ❌ Chuỗi `.then()` dài vẫn rườm rà (**promise hell**); luồng vẫn không "thẳng" như code đồng bộ.

**3. async/await** ([câu 9.4](#q9-4)) — syntactic sugar **trên Promise**.
- ✅ Viết async **trông như đồng bộ** → dễ đọc nhất; bắt lỗi bằng **`try/catch/finally`** quen thuộc;
  debug/step qua dễ.
- ❌ Vẫn là Promise bên dưới; `await` tuần tự dễ vô tình **chặn** tác vụ độc lập (phải nhớ dùng
  `Promise.all`); trình duyệt rất cũ cần Babel.

**Cùng một việc — tải 2 file tuần tự, viết bằng 3 cách:**

```js
// 1) Callback (lồng nhau → callback hell)
download("a.txt", function (a) {
  download("b.txt", function (b) {
    console.log("xong", a, b);
  });
});

// 2) Promise (chaining phẳng)
download("a.txt")
  .then((a) => download("b.txt"))
  .then((b) => console.log("xong"))
  .catch((err) => console.log("Lỗi:", err));

// 3) async/await (giống đồng bộ)
async function run() {
  try {
    const a = await download("a.txt");
    const b = await download("b.txt");
    console.log("xong", a, b);
  } catch (err) {
    console.log("Lỗi:", err);
  }
}
```

**Bảng so sánh:**

| Tiêu chí | **Callback** | **Promise** | **async/await** |
|---|---|---|---|
| **Bản chất** | Hàm truyền vào | Object kết quả tương lai | Sugar trên Promise |
| **Ra đời** | Sớm nhất | ES2015 (ES6) | ES2017 |
| **Đọc/bảo trì** | Kém khi lồng (callback hell) | Tốt (chaining phẳng) | **Tốt nhất** (như đồng bộ) |
| **Bắt lỗi** | Thủ công từng callback | `.catch()` cho cả chuỗi | `try/catch/finally` |
| **Nhiều tác vụ song song** | Tự xoay xở (khó) | `Promise.all/race/any` | `await Promise.all([...])` |
| **Nhược điểm** | Callback hell | Promise hell (chuỗi dài) | `await` dễ chặn tác vụ độc lập |

> **Mấu chốt:** cả ba **không khác nhau về bản chất async** (đều dựa trên event loop & queue — xem
> [câu 9.6](#q9-6)), chỉ khác về **cách viết & độ dễ đọc**. async/await không "mạnh hơn" Promise — nó
> *là* Promise, chỉ gọn hơn. Thực tế thường **kết hợp:** `async/await` cho luồng chính + `Promise.all`
> cho song song.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả ba là **cách xử lý kết quả async**, tiến hóa nối tiếp: **Callback → Promise → async/await**, mỗi
  cái khắc phục nhược điểm cái trước.
- **Callback:** đơn giản nhưng **callback hell** + bắt lỗi rời rạc. **Promise:** chaining phẳng, một
  `.catch()`, có `Promise.all/race/any`; nhược: **promise hell**. **async/await:** sugar trên Promise,
  **dễ đọc nhất** (như đồng bộ), bắt lỗi bằng **`try/catch`**.
- **Quan trọng:** async/await **chính là Promise** (không mạnh hơn, chỉ gọn hơn); cả ba **cùng cơ chế
  event loop/queue** ([câu 9.6](#q9-6)) — chỉ khác cú pháp & độ dễ đọc.
- **Thực tế:** dùng **async/await** cho luồng chính + **`Promise.all`** cho tác vụ độc lập chạy song song.

[↑ Mục lục](#mục-lục)

<a id="q9-6"></a>
### 9.6. Phân biệt microtask và macrotask

#### 📖 Giới thiệu kiến thức

Khi tác vụ async xong, callback của nó được xếp vào **một trong hai loại hàng đợi**, và Event Loop xử
lý chúng với **độ ưu tiên khác nhau** — đây là gốc giải thích thứ tự output của mọi câu đố async.

**Macrotask (task / callback queue):**
- Gồm: **`setTimeout`**, **`setInterval`**, **`setImmediate`** (Node), **I/O callback**, **DOM events**,
  **`MessageChannel`**.
- Mỗi vòng lặp Event Loop chỉ lấy **MỘT** macrotask để chạy.

**Microtask (job queue):**
- Gồm: **`Promise.then/catch/finally`**, **`queueMicrotask()`**, **`await`** (phần sau await),
  **`MutationObserver`**, **`process.nextTick`** (Node — ưu tiên cao nhất, là hàng riêng).
- **Ưu tiên cao hơn** macrotask.

**Quy tắc vàng (cốt lõi):** sau khi chạy xong **một** macrotask (và sau khi code đồng bộ chạy hết),
Event Loop sẽ **vét SẠCH toàn bộ microtask queue** (kể cả microtask mới sinh ra trong lúc đó) **TRƯỚC
KHI** lấy macrotask tiếp theo.

```
Đồng bộ (hết call stack)
   → vét SẠCH microtask queue
   → 1 macrotask
   → vét SẠCH microtask queue
   → 1 macrotask
   → ... (lặp lại)
```

**Thứ tự ưu tiên tổng quát:** **Code đồng bộ → Microtask → Macrotask.**

```js
console.log("1");                                  // đồng bộ
setTimeout(() => console.log("2"), 0);             // macrotask
Promise.resolve().then(() => console.log("3"));    // microtask
console.log("4");                                  // đồng bộ
// Kết quả: 1 → 4 → 3 → 2
```

**Ví dụ "bẫy" — microtask sinh thêm microtask vẫn chạy hết trước macrotask:**

```js
console.log("start");
setTimeout(() => console.log("timeout"), 0);       // macrotask
Promise.resolve().then(() => {
  console.log("promise 1");
  Promise.resolve().then(() => console.log("promise 2")); // microtask mới
});
console.log("end");
/* Kết quả:
start
end
promise 1
promise 2   ← microtask mới vẫn được vét hết TRƯỚC "timeout"
timeout
*/
```

> **So sánh nhanh:**
>
> | | **Microtask** | **Macrotask** |
> |---|---|---|
> | **Gồm** | `Promise.then`, `queueMicrotask`, `await`, `MutationObserver`, `process.nextTick` | `setTimeout`, `setInterval`, `setImmediate`, I/O, DOM events |
> | **Ưu tiên** | Cao hơn | Thấp hơn |
> | **Số lượng chạy mỗi vòng** | **Vét sạch cả queue** | Chỉ **1 task** rồi vét microtask |

**Lưu ý:** lạm dụng microtask đệ quy (vd `Promise.then` gọi lại chính nó liên tục, hoặc
`process.nextTick` đệ quy) có thể **"bỏ đói" (starve)** macrotask → timer/I/O không bao giờ được chạy.
*(Chi tiết phase & `process.nextTick` của Node ở [nodejs.md câu 2](./nodejs.md#q2).)*

#### 🎯 Tóm tắt trả lời phỏng vấn

- Callback async xếp vào **2 hàng đợi**: **Macrotask** (`setTimeout`, `setInterval`, I/O, DOM events,
  `setImmediate`) và **Microtask** (`Promise.then`, `queueMicrotask`, `await`, `process.nextTick`).
- **Microtask ưu tiên cao hơn macrotask.**
- **Quy tắc vàng:** sau code đồng bộ và sau **mỗi** macrotask, Event Loop **vét sạch toàn bộ microtask
  queue** (kể cả microtask mới sinh) **trước khi** chạy macrotask kế tiếp.
- Thứ tự tổng quát: **đồng bộ → microtask → macrotask** (xem ví dụ `1 → 4 → 3 → 2`).
- **Bẫy:** microtask đệ quy có thể **bỏ đói macrotask**. Node chia thêm **phase** + có `process.nextTick`
  ([nodejs.md câu 2](./nodejs.md#q2)).

[↑ Mục lục](#mục-lục)

<a id="q9-7"></a>
### 9.7. RxJS là gì?

#### 📖 Giới thiệu kiến thức

**RxJS (Reactive Extensions for JavaScript)** là **thư viện lập trình phản ứng (reactive programming)**
dựa trên **luồng dữ liệu bất đồng bộ (async data streams)** — bản hiện thực cho JS của mô hình
**ReactiveX**. Ý tưởng cốt lõi: **coi mọi thứ là một "luồng" (stream)** — click, request HTTP, timer,
thay đổi state... đều là **dãy giá trị phát ra theo thời gian**; ta **lắng nghe** và **biến đổi** luồng
đó bằng các **operators** theo phong cách **functional**. Đây là **cách xử lý bất đồng bộ thứ tư**, bổ
sung cho callback → Promise → async/await (xem [câu 9](#q9)).

**① 3 mảnh ghép cơ bản — Observable, Observer, Subscription:**

- **Observable** — "nguồn phát" (producer): định nghĩa **một luồng** có thể phát 3 loại thông báo:
  **`next(value)`** (một giá trị mới, phát 0..∞ lần), **`error(err)`** (lỗi → kết thúc luồng),
  **`complete()`** (hoàn tất → kết thúc). `error` và `complete` **loại trừ nhau** và chỉ xảy ra một lần.
- **Observer** — object **lắng nghe**, gồm 3 callback `next` / `error` / `complete`.
- **Subscription** — kết quả `.subscribe()`; gọi **`.unsubscribe()`** để **hủy** (dừng nhận + dọn tài
  nguyên → tránh **memory leak**).

```js
import { Observable } from "rxjs";

// Tự tạo một Observable (hiếm khi cần, thường dùng creation operator có sẵn)
const obs$ = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  setTimeout(() => { subscriber.next(3); subscriber.complete(); }, 1000);
  // Hàm teardown: chạy khi unsubscribe/complete → nơi dọn timer, listener...
  return () => console.log("cleanup");
});

const sub = obs$.subscribe({
  next:  (v)   => console.log("value:", v), // value:1, value:2, (1s sau) value:3
  error: (e)   => console.error(e),
  complete: () => console.log("done"),      // done
});
// sub.unsubscribe(); // hủy sớm nếu cần
```

**② Tạo Observable — creation operators** (thường dùng thay vì `new Observable`):

```js
import { of, from, fromEvent, interval, timer, throwError } from "rxjs";

of(1, 2, 3);                       // phát lần lượt 1,2,3 rồi complete (đồng bộ)
from([1, 2, 3]);                   // từ array / Promise / iterable
from(fetch("/api"));               // bọc 1 Promise thành Observable
fromEvent(button, "click");        // từ DOM event (luồng vô hạn)
interval(1000);                    // phát 0,1,2,... mỗi 1s
timer(2000, 1000);                 // chờ 2s, rồi phát mỗi 1s
```

**③ Cold vs Hot Observable (điểm hay bị hỏi):**

- **Cold** — producer **được tạo riêng cho từng subscriber**; mỗi lần `subscribe` là một lần chạy
  **độc lập từ đầu** (vd `of`, `from(fetch)`, HTTP của Angular). → 2 subscriber nhận 2 luồng riêng.
- **Hot** — producer **dùng chung**, phát bất kể có ai nghe hay không; subscriber vào sau **bỏ lỡ** giá
  trị đã phát (vd `fromEvent`, `Subject`). Biến cold → hot/multicast bằng `share()`, `Subject`.

**④ Operators — biến đổi luồng trong `.pipe()`:** operator là **hàm thuần**, nhận Observable trả
Observable mới (không đổi luồng gốc). Phân nhóm hay dùng:

| Nhóm | Operators tiêu biểu | Công dụng |
|---|---|---|
| **Transformation** | `map`, `scan`, `pluck`, `bufferTime` | Biến đổi từng giá trị |
| **Filtering** | `filter`, `take`, `takeUntil`, `debounceTime`, `distinctUntilChanged`, `first`, `skip` | Lọc/giảm giá trị |
| **Combination** | `merge`, `concat`, `combineLatest`, `forkJoin`, `zip`, `withLatestFrom` | Gộp nhiều luồng |
| **Flattening** | `switchMap`, `mergeMap`, `concatMap`, `exhaustMap` | Luồng lồng luồng (gọi API...) |
| **Error/Utility** | `catchError`, `retry`, `tap`, `finalize`, `startWith` | Lỗi & phụ trợ |

**⑤ Flattening operators — khác biệt CỰC KỲ hay hỏi:** khi mỗi giá trị lại sinh **một Observable con**
(vd mỗi lần gõ phím → 1 request), cách xử lý request cũ khi request mới tới **quyết định chọn operator**:

| Operator | Hành vi khi có giá trị mới trong lúc inner cũ chưa xong | Dùng khi |
|---|---|---|
| **`switchMap`** | **Hủy** inner cũ, chuyển sang cái mới | **Search/autocomplete** (chỉ cần kết quả mới nhất) |
| **`mergeMap`** (`flatMap`) | Chạy **song song** tất cả, không hủy | Request độc lập cần chạy đồng thời |
| **`concatMap`** | **Xếp hàng** — xong cái trước mới chạy cái sau | Cần **đúng thứ tự** (vd ghi tuần tự) |
| **`exhaustMap`** | **Bỏ qua** cái mới nếu inner cũ chưa xong | Chống **double-submit** (nút login) |

**⑥ Subject & các biến thể — vừa phát vừa nghe (multicast):** `Subject` vừa là Observable vừa là
Observer → dùng làm **event bus / state store**:

- **`Subject`** — multicast thuần; subscriber chỉ nhận giá trị **phát sau khi** đã subscribe.
- **`BehaviorSubject`** — có **giá trị hiện tại**; subscriber mới nhận **ngay giá trị gần nhất**
  (cần seed giá trị khởi tạo) → hợp làm **state** (giống store).
- **`ReplaySubject`** — **phát lại N giá trị gần nhất** cho subscriber mới.
- **`AsyncSubject`** — chỉ phát **giá trị cuối cùng** khi `complete`.

```js
import { BehaviorSubject } from "rxjs";
const currentUser$ = new BehaviorSubject(null); // state khởi tạo null
currentUser$.subscribe((u) => console.log("A:", u)); // A: null (nhận ngay giá trị hiện tại)
currentUser$.next({ name: "An" });                    // A: {name:'An'}
currentUser$.subscribe((u) => console.log("B:", u)); // B: {name:'An'} (subscriber mới vẫn có giá trị)
```

**⑦ Xử lý lỗi:** dùng **`catchError`** (trả về Observable thay thế để luồng không "chết"),
**`retry(n)`** (thử lại n lần), **`finalize`** (chạy khi hoàn tất/lỗi/unsubscribe — dọn dẹp).

```js
import { of } from "rxjs";
import { catchError, retry } from "rxjs/operators";
source$.pipe(
  retry(2),                                  // thử lại tối đa 2 lần nếu lỗi
  catchError((err) => of([]))                // lỗi → trả mảng rỗng, không vỡ luồng
);
```

**⑧ Ví dụ thực tế — search có debounce + hủy request cũ (kinh điển của RxJS):**

```js
import { fromEvent, of } from "rxjs";
import { map, debounceTime, distinctUntilChanged, switchMap, catchError } from "rxjs/operators";

const input = document.querySelector("#search");
const sub = fromEvent(input, "input").pipe(
  map((e) => e.target.value),
  debounceTime(300),          // chờ 300ms ngưng gõ mới phát → giảm số request
  distinctUntilChanged(),      // bỏ qua nếu từ khóa không đổi
  switchMap((q) =>             // request mới tới → HỦY request cũ (chống race condition)
    from(fetch(`/api?q=${q}`).then((r) => r.json())).pipe(catchError(() => of([])))
  )
).subscribe((data) => render(data));

// Khi component unmount:
sub.unsubscribe(); // hoặc dùng takeUntil(destroy$) / async pipe (Angular)
```

**⑨ Tránh memory leak — phải unsubscribe:** Observable **vô hạn** (`fromEvent`, `interval`, `Subject`)
không tự dừng → phải hủy: gọi `unsubscribe()` thủ công, dùng **`takeUntil(destroy$)`**, hoặc **`async`
pipe** của Angular (tự subscribe/unsubscribe). Observable **hữu hạn tự complete** (`of`, `from(fetch)`,
HTTP) thì ít lo hơn.

> **Khi nào dùng RxJS:** luồng sự kiện phức tạp theo thời gian — search/autocomplete (debounce), typeahead,
> drag-drop, websocket/real-time, polling, gộp nhiều nguồn async, cần **hủy** request (`switchMap`). Gắn
> liền **Angular** (RxJS là mặc định, `HttpClient` trả Observable). **Đánh đổi:** learning curve dốc,
> dễ over-engineer — một request đơn giản thì **Promise/async-await gọn hơn**. So sánh sâu Observable vs
> Promise: Observable **nhiều giá trị / lazy / hủy được / giàu operators**; Promise **1 giá trị / eager /
> không hủy**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **RxJS** = thư viện **lập trình phản ứng** xử lý **luồng dữ liệu bất đồng bộ theo thời gian** (coi
  click/HTTP/timer là **stream**). Là **cách xử lý async thứ tư** sau callback/Promise/async-await.
- **3 mảnh ghép:** **Observable** (nguồn phát, phát `next`/`error`/`complete`) — **Observer** (lắng nghe)
  — **Subscription** (`.subscribe()` → `.unsubscribe()`). Tạo bằng creation operator: `of`, `from`,
  `fromEvent`, `interval`, `timer`.
- **Khác Promise:** Observable phát **nhiều giá trị** (Promise 1), **lazy** — chỉ chạy khi `subscribe`
  (Promise **eager**), **hủy được** (`unsubscribe`), kèm kho **operators** trong `.pipe()`.
- **Operators** phân nhóm: transformation (`map`), filtering (`filter`, `debounceTime`,
  `distinctUntilChanged`, `takeUntil`), combination (`combineLatest`, `forkJoin`, `merge`), error
  (`catchError`, `retry`). **Flattening** (hay hỏi): **`switchMap`** (hủy cái cũ — search), **`mergeMap`**
  (song song), **`concatMap`** (tuần tự), **`exhaustMap`** (bỏ qua cái mới — chống double-submit).
- **Subject** = vừa phát vừa nghe (multicast/event bus): **`BehaviorSubject`** (giữ giá trị hiện tại →
  làm state), **`ReplaySubject`** (phát lại N gần nhất), **`AsyncSubject`** (giá trị cuối).
- **Cold vs Hot:** cold = mỗi subscriber một luồng riêng (HTTP); hot = dùng chung, vào sau bỏ lỡ
  (`Subject`, `fromEvent`). **Nhớ unsubscribe** luồng vô hạn (`takeUntil`/`async` pipe) để tránh leak.
- **Dùng khi** luồng sự kiện phức tạp (search debounce, real-time, hủy request) — gắn liền **Angular**;
  async đơn giản thì Promise/async-await đủ.

[↑ Mục lục](#mục-lục)

<a id="q10"></a>
## 10. Proxy trong JavaScript là gì?

### 📖 Giới thiệu kiến thức

**`Proxy`** là một object built-in của ES6 cho phép bạn **bọc một object khác** và **chặn (intercept)
+ tùy biến** các thao tác cơ bản trên nó: đọc thuộc tính, ghi thuộc tính, gọi hàm, kiểm tra tồn tại,
xóa key... Nói cách khác, Proxy đặt một "lớp giữa" giữa code và object thật để bạn xen vào hành vi mặc định.

Cú pháp:

```js
const proxy = new Proxy(target, handler);
```

- **`target`**: object gốc cần bọc.
- **`handler`**: object chứa các **"trap"** — hàm chặn cho từng thao tác. Trap nào không khai báo thì
  Proxy thực hiện hành vi mặc định trên `target`.

Các trap hay dùng: `get` (đọc prop), `set` (ghi prop), `has` (toán tử `in`), `deleteProperty`
(`delete`), `apply` (gọi function), `construct` (`new`), `ownKeys` (liệt kê key).

**Ví dụ — validation + giá trị mặc định khi đọc:**

```js
const user = { name: 'An', age: 25 };

const safeUser = new Proxy(user, {
  get(target, prop) {
    // trả về 'N/A' thay vì undefined cho prop không tồn tại
    return prop in target ? target[prop] : 'N/A';
  },
  set(target, prop, value) {
    if (prop === 'age' && (typeof value !== 'number' || value < 0)) {
      throw new TypeError('age phải là số không âm');
    }
    target[prop] = value;
    return true; // bắt buộc: báo set thành công
  },
});

console.log(safeUser.name);   // 'An'
console.log(safeUser.email);  // 'N/A'  (không tồn tại)
safeUser.age = 30;            // OK
safeUser.age = -5;            // ❌ throw TypeError
```

**Ứng dụng thực tế:** validation, object chỉ-đọc (readonly), logging/auditing truy cập,
lazy loading, default value, **reactivity** (Vue 3, MobX), giả lập API, bảo vệ thuộc tính private.

**Lưu ý:**
- Proxy chặn được **cả thuộc tính thêm sau khi tạo** (khác hẳn `Object.defineProperty` chỉ gắn từng key).
- Có chi phí hiệu năng nhỏ so với truy cập object thường → dùng đúng chỗ.

> Liên hệ: ý tưởng "object đại diện kiểm soát truy cập tới object thật" chính là **Proxy Pattern**
> (xem `Common/solid-design-pattern.md` §2.6). `Proxy` của JS là hiện thực hóa pattern này ở mức ngôn ngữ.

### 🎯 Tóm tắt trả lời phỏng vấn

- `Proxy` (ES6) bọc một object và cho phép **chặn + tùy biến** các thao tác cơ bản qua các **trap**
  (`get`, `set`, `has`, `deleteProperty`, `apply`...).
- Cú pháp `new Proxy(target, handler)`; trap không khai báo thì chạy hành vi mặc định.
- Dùng cho: validation, readonly, logging, default value, lazy load và đặc biệt là **reactivity** (Vue 3).
- Ưu điểm so với `Object.defineProperty`: chặn được **mọi thao tác kể cả key thêm/xóa sau này**.

[↑ Mục lục](#mục-lục)

<a id="q10-1"></a>
### 10.1. Reflect dùng để làm gì? Vì sao đi cùng Proxy?

#### 📖 Giới thiệu kiến thức

**`Reflect`** là một object built-in cung cấp các **method tương ứng 1-1 với các trap của Proxy**
(`Reflect.get`, `Reflect.set`, `Reflect.has`, `Reflect.deleteProperty`...). Nó thực hiện đúng **hành vi
mặc định** của thao tác đó trên object.

Trong handler của Proxy, thay vì tự viết `target[prop] = value`, ta nên gọi `Reflect.set(...)` vì:

1. **Giữ đúng hành vi gốc** một cách chuẩn xác (kể cả getter/setter, prototype chain).
2. **Xử lý `receiver` đúng** — quan trọng khi object có getter dùng `this`, tránh bug khi proxy
   nằm trong chuỗi kế thừa.
3. Trả về **giá trị/boolean chuẩn** mà trap cần (vd `set` cần trả `true/false`).

```js
const handler = {
  get(target, prop, receiver) {
    console.log(`đọc ${prop}`);
    return Reflect.get(target, prop, receiver); // hành vi mặc định, đúng `this`
  },
  set(target, prop, value, receiver) {
    console.log(`ghi ${prop} = ${value}`);
    return Reflect.set(target, prop, value, receiver); // trả về true/false chuẩn
  },
};

const obj = new Proxy({ _x: 1, get x() { return this._x; } }, handler);
obj.x; // log "đọc x" rồi "đọc _x", trả về 1 — receiver đảm bảo `this` đúng
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- `Reflect` cung cấp method **tương ứng 1-1 với các trap của Proxy**, chạy đúng **hành vi mặc định**.
- Trong trap nên dùng `Reflect.*` thay cho thao tác thủ công để **giữ đúng `receiver`/`this`**,
  hỗ trợ getter/setter và prototype chain, và trả về giá trị chuẩn.
- Proxy + Reflect là cặp đôi tự nhiên: Proxy *chặn*, Reflect *thực thi mặc định một cách an toàn*.

[↑ Mục lục](#mục-lục)

<a id="q10-2"></a>
### 10.2. Vue 3 dùng Proxy cho reactivity như thế nào?

#### 📖 Giới thiệu kiến thức

Đây là ví dụ kinh điển về Proxy trong thực tế. **Vue 2** dùng `Object.defineProperty` để theo dõi
thay đổi, nhưng nó có hạn chế: **không phát hiện được việc thêm/xóa thuộc tính mới** và **thay đổi
phần tử mảng theo index** (phải dùng `Vue.set`).

**Vue 3** chuyển sang **`Proxy`** trong hàm `reactive()`. Khi bạn truy cập một thuộc tính (`get`),
Vue **track** dependency (component nào đang dùng nó); khi bạn gán (`set`), Vue **trigger** cập nhật
lại đúng những chỗ phụ thuộc. Vì Proxy chặn ở cấp object nên Vue 3 phát hiện được **cả prop mới thêm,
prop bị xóa, và thay đổi mảng** mà không cần API đặc biệt.

```js
// Mô phỏng đơn giản cơ chế reactive() của Vue 3
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key, receiver) {
      track(target, key);                       // ghi nhận dependency
      return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
      const result = Reflect.set(target, key, value, receiver);
      trigger(target, key);                      // chạy lại effect phụ thuộc
      return result;
    },
  });
}
```

Hệ quả thực tế trong Vue 3:
- `state.newProp = 1` **tự reactive** ngay (Vue 2 thì không).
- Không cần `Vue.set` / `Vue.delete` nữa.
- `reactive()` chỉ áp cho **object/array**; với giá trị nguyên thủy (number, string) thì dùng `ref()`
  (bọc trong `{ value }` rồi mới reactive).

> Chi tiết về reactivity, `ref` vs `reactive`, Composition API nằm ở `Frontend/vuejs.md`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Vue 3 dùng `Proxy` (trong `reactive()`) thay cho `Object.defineProperty` của Vue 2.
- Trap `get` để **track** dependency, trap `set` để **trigger** re-render đúng chỗ phụ thuộc.
- Nhờ Proxy chặn ở cấp object, Vue 3 phát hiện được **thêm/xóa prop và thay đổi mảng** — bỏ được `Vue.set`.
- Giá trị nguyên thủy không proxy được trực tiếp → dùng `ref()` (bọc `{ value }`).

[↑ Mục lục](#mục-lục)
