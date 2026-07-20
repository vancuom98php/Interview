<a id="mục-lục"></a>
# Backend — Node.js

> Runtime & V8 · event loop của Node (libuv, phases) · non-blocking I/O · module (CommonJS/ESM) ·
> stream · cluster/worker_threads · Express · NestJS · npm/package.json · middleware.
> (Câu hỏi về JavaScript thuần nằm ở [js-core.md](./js-core.md).)

## 📑 Mục lục

1. [Phân biệt JavaScript, Node.js và các framework của Node.js](#q1)
2. [Tại sao JS (Node) và PHP đơn luồng nhưng vẫn dùng làm ngôn ngữ backend?](#q2)
3. [Event loop và thứ tự thực thi task trong Node.js](#q3)
4. [Phân biệt process, thread và worker_threads trong Node.js](#q4)
5. [Phân biệt npm, npx, pnpm và yarn](#q5)
   - 5.1. [Phân biệt devDependencies và dependencies trong package.json](#q5-1)
6. [Thư viện node:crypto trong JS/TS là gì?](#q6)
   - 6.1. [Thư viện bcryptjs là gì? (so với bcrypt native)](#q6-1)

---

<a id="q1"></a>
## 1. Phân biệt JavaScript, Node.js và các framework của Node.js

### 📖 Giới thiệu kiến thức

Đây là 3 **tầng khái niệm khác nhau**, không cùng loại — interviewer hỏi để xem ứng viên có phân biệt
được **ngôn ngữ ↔ môi trường chạy ↔ thư viện/khung làm việc** hay không.

**1. JavaScript — Ngôn ngữ (language):**
JavaScript là một **ngôn ngữ lập trình** (đặc tả ECMAScript). Bản thân nó chỉ định nghĩa **cú pháp và
tính năng cốt lõi** (biến, hàm, object, closure, Promise...). JS **không tự chạy được** — nó cần một
**môi trường (host)** cung cấp engine để thực thi và các API để tương tác với thế giới ngoài. Hai môi
trường phổ biến nhất: **trình duyệt** (browser) và **Node.js**.

**2. Node.js — Môi trường chạy (runtime), KHÔNG phải ngôn ngữ, KHÔNG phải framework:**
Node.js là một **JavaScript runtime** — môi trường cho phép **chạy JavaScript ở phía server / ngoài
trình duyệt**. Thành phần chính:
- **V8** (engine của Google Chrome) — biên dịch & thực thi JS.
- **libuv** — thư viện C cung cấp **event loop** và **non-blocking I/O** (đọc file, network, timer).
- **Các API riêng của Node** mà trình duyệt **không có**: `fs` (file system), `http`, `process`,
  `Buffer`, `os`... → cho phép JS truy cập hệ thống, làm backend.

> Khác biệt then chốt với browser: browser cho JS các API như `document`, `window`, DOM; Node cho JS
> các API hệ thống (`fs`, `net`...). **Cùng ngôn ngữ JS, khác bộ API do môi trường cấp.**

**3. Framework của Node.js — Thư viện/khung làm việc xây TRÊN Node:**
Node.js cho công cụ "thô" (vd module `http` để dựng server thủ công). **Framework** là các **gói (package)
viết bằng JS/TS, cài qua npm**, đóng gói sẵn cấu trúc & tính năng để phát triển nhanh hơn:
- **Express.js:** framework web **tối giản (minimal)** — routing + middleware. Linh hoạt, tự do cấu trúc.
- **NestJS:** framework **có cấu trúc rõ (opinionated)**, dùng **TypeScript**, kiến trúc module + DI
  (giống Angular/Spring) — hợp dự án lớn. *(Thường chạy trên nền Express.)*
- **Fastify, Koa, Hapi...** các lựa chọn khác.

**Quan hệ phân tầng (rất nên vẽ khi trả lời):**

```
┌──────────────────────────────────────────────┐
│  Framework (Express / NestJS / Fastify...)    │  ← gói npm, viết bằng JS/TS, build NHANH hơn
├──────────────────────────────────────────────┤
│  Node.js  (runtime: V8 + libuv + API hệ thống)│  ← MÔI TRƯỜNG chạy JS ngoài browser
├──────────────────────────────────────────────┤
│  JavaScript  (ngôn ngữ — đặc tả ECMAScript)   │  ← NGÔN NGỮ
└──────────────────────────────────────────────┘
   Viết bằng JS → chạy nhờ Node → tăng tốc bằng Framework
```

**Ví dụ trực quan — cùng một việc (HTTP server) ở 3 mức:**

```js
// JS thuần: không làm được server — không có API mạng/hệ thống.

// Node.js thuần (dùng module http có sẵn của runtime):
const http = require('http');
http.createServer((req, res) => res.end('Hello')).listen(3000);

// Express (framework trên Node — gọn & nhiều tiện ích hơn):
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello'));
app.listen(3000);
```

### 🎯 Tóm tắt trả lời phỏng vấn

- Ba thứ ở **3 tầng khác nhau**: **JavaScript = ngôn ngữ**, **Node.js = môi trường chạy (runtime)**,
  **framework = thư viện/khung xây trên Node**.
- **JavaScript** chỉ là ngôn ngữ (cú pháp + tính năng core), **không tự chạy** — cần host (browser hoặc Node).
- **Node.js** là **runtime** giúp chạy JS **ngoài trình duyệt** (server): gồm **V8** (engine) +
  **libuv** (event loop, non-blocking I/O) + **API hệ thống** (`fs`, `http`, `process`) mà browser không có.
  Node **không phải ngôn ngữ, cũng không phải framework**.
- **Framework (Express, NestJS, Fastify)** là **gói npm** viết bằng JS/TS, **xây trên Node**, đóng gói
  sẵn routing/middleware/cấu trúc để phát triển nhanh hơn. Express tối giản & linh hoạt; NestJS có cấu
  trúc rõ, dùng TS + DI, hợp dự án lớn.
- Một câu gói gọn: **"Viết bằng JavaScript, chạy nhờ Node.js, phát triển nhanh nhờ framework."**

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Tại sao JS (Node) và PHP đơn luồng nhưng vẫn dùng làm ngôn ngữ backend?

### 📖 Giới thiệu kiến thức

Câu hỏi này thường nhằm kiểm tra hiểu biết về **mô hình xử lý đồng thời (concurrency model)** — chứ
không phải để "chê" hai ngôn ngữ. Trước hết phải làm rõ: **"đơn luồng" ở Node và ở PHP không cùng nghĩa.**

**1. "Đơn luồng" nghĩa là gì ở mỗi bên?**
- **Node.js:** *thực sự* chạy **toàn bộ code JS của bạn trên MỘT main thread duy nhất** cho cả tiến
  trình; một main thread đó phục vụ **tất cả** request thông qua **event loop**.
- **PHP (mô hình kinh điển PHP-FPM/Apache):** mỗi request được xử lý **tuần tự, đồng bộ (blocking)**
  bên trong **một process/worker** — có thể gọi là **"đơn luồng theo từng request"** (share-nothing).
  Nhưng web server chạy **rất nhiều worker process song song** để phục vụ nhiều request cùng lúc.

**2. Lý do gốc rễ: backend chủ yếu là I/O-bound, không phải CPU-bound.**
Phần lớn thời gian xử lý một request web là **CHỜ I/O**: query database, gọi API ngoài, đọc/ghi file,
đợi cache... Trong lúc chờ đó **CPU gần như rảnh**. Vậy nút thắt thường **không phải thiếu luồng CPU**,
mà là **tận dụng thời gian chờ** sao cho hiệu quả. Đây là lý do "đơn luồng" vẫn dư sức làm backend —
miễn là giải quyết tốt việc chờ I/O. Node và PHP chọn **hai cách khác nhau** cho cùng bài toán này.

**3. Cách của Node.js — 1 thread + non-blocking I/O + event loop:**
- Khi gặp I/O, Node **không đứng chờ (non-blocking)**: nó đăng ký **callback**, giao tác vụ I/O cho
  **libuv/OS**, rồi **main thread quay sang xử lý request khác ngay**.
- Khi I/O xong, callback được đưa vào queue và **event loop** lấy ra chạy.
- → Chỉ **một thread** vẫn phục vụ được **hàng nghìn kết nối đồng thời** (high concurrency) với rất ít
  RAM và không tốn chi phí tạo/chuyển ngữ cảnh thread → hợp app **nhiều I/O** (real-time, API gateway).
- **Đánh đổi:** một tác vụ **CPU-bound nặng** sẽ **block** event loop, làm nghẽn mọi request khác →
  khi đó cần **`worker_threads`** hoặc **`cluster`** (xem [câu 3](#q3) và [câu 4](#q4)).

**4. Cách của PHP — blocking đồng bộ + nhiều process (share-nothing):**
- PHP truyền thống dùng **blocking I/O**: trong một request, gặp I/O thì **chờ xong mới chạy tiếp**.
- Concurrency đạt được bằng cách **chạy song song nhiều worker process**: **PHP-FPM** quản lý một
  **pool** (vd `pm.max_children = 50`) hoặc Apache `mpm_prefork`. **Nginx** nhận request rồi **phân
  phối** tới các worker đang rảnh.
- Mỗi request = **một process độc lập, "shared-nothing"**: khởi tạo → xử lý → trả response → **xóa
  sạch state**. Đơn giản, an toàn (không lo race condition giữa request), **dễ scale ngang**.
- **OS scheduler** trải nhiều process này lên **nhiều CPU core** → vẫn dùng hết phần cứng đa nhân.
- **Đánh đổi:** mỗi process **tốn RAM**, khởi tạo lại mỗi request; mức concurrency **bị giới hạn bởi
  số worker** trong pool.

**5. Sơ đồ so sánh:**

```
NODE.JS (1 process, 1 thread)        PHP-FPM (nhiều process)
┌─────────────────────────┐         ┌──────────────────────────────┐
│  Event Loop (1 thread)  │         │ Nginx → phân phối request     │
│  ├ req A ─┐ chờ I/O      │         │   ├─ Worker 1  (1 request)    │
│  ├ req B ─┤ (không block)│         │   ├─ Worker 2  (1 request)    │
│  └ req C ─┘ xử lý xen kẽ │         │   ├─ Worker 3  (1 request)    │
│  libuv/OS lo I/O nền     │         │   └─ ...  (blocking, song song)│
└─────────────────────────┘         └──────────────────────────────┘
 Concurrency TRONG 1 luồng           Concurrency bằng NHÂN BẢN process
```

**6. Điểm chốt thường bị bỏ qua:**
- "Đơn luồng" nói về **code ứng dụng bạn viết**. Bên dưới cả hai vẫn dựa vào OS; và cả hai đều **scale
  để dùng nhiều CPU core** bằng cách chạy **nhiều process**: Node dùng **`cluster`/PM2**, PHP dùng
  **FPM pool**.
- Vì backend là I/O-bound, **đơn luồng + async (Node)** hay **đồng bộ + nhiều process (PHP)** đều
  phục vụ tải cao tốt — chỉ khác cách đánh đổi RAM/độ phức tạp.
- *(Mở rộng)* PHP nay cũng có runtime async/long-running như **Swoole, RoadRunner, ReactPHP** chạy
  theo mô hình event-loop giống Node, nhưng mô hình **kinh điển** vẫn là FPM multi-process.

### 🎯 Tóm tắt trả lời phỏng vấn

- Trước hết phân biệt: **Node thực sự một main thread cho cả tiến trình** (event loop), còn **PHP là
  "đơn luồng theo từng request"** nhưng **chạy nhiều worker process song song** (PHP-FPM).
- **Lý do cốt lõi:** backend chủ yếu **I/O-bound** (chờ DB/API/file), CPU thường rảnh → nút thắt là
  **tận dụng thời gian chờ**, không phải số luồng CPU. Nên đơn luồng vẫn thừa sức làm backend.
- **Node:** 1 thread + **non-blocking I/O** + event loop → gặp I/O thì giao cho libuv, không chờ, quay
  sang request khác → **1 thread phục vụ hàng nghìn kết nối**, ít RAM. Yếu điểm: **CPU-bound block**
  event loop → cần `worker_threads`/`cluster`.
- **PHP:** **blocking đồng bộ** nhưng **nhiều process share-nothing**; Nginx phân phối request cho pool
  worker, OS trải lên nhiều core. Đơn giản, an toàn, dễ scale ngang; yếu điểm: mỗi process **tốn RAM**,
  concurrency giới hạn bởi số worker.
- Một câu gói gọn: **"Cả hai giải bài toán I/O-bound bằng concurrency, chỉ khác cách — Node concurrency
  TRONG một luồng (async), PHP concurrency bằng NHÂN BẢN process; cả hai đều scale đa core bằng nhiều process."**

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Event loop và thứ tự thực thi task trong Node.js

### 📖 Giới thiệu kiến thức

Node.js chạy JavaScript chủ yếu trên **một main thread**. **Event Loop** là cơ chế điều phối việc thực
thi của tất cả các loại task: **code đồng bộ**, callback từ **timer**, callback **I/O**, **Promise**,
**async/await**, **`setImmediate`**, **`process.nextTick`**.

Điểm quan trọng nhất cần hiểu: **không phải cứ đăng ký trước thì chắc chắn chạy trước**. Thứ tự thực
thi còn phụ thuộc vào **loại task** và **phase của Event Loop**.

#### 1. Thứ tự ưu tiên cơ bản

Có thể hiểu đơn giản theo thứ tự ưu tiên giảm dần:

```
Code đồng bộ
→ process.nextTick
→ Promise / queueMicrotask
→ Timer, I/O, setImmediate...
```

Ví dụ:

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

process.nextTick(() => {
  console.log("4");
});

console.log("5");

/* Kết quả:
1
5
4
3
2
*/
```

Giải thích:
- `console.log("1")` chạy đồng bộ.
- `setTimeout` đăng ký timer.
- `.then()` đưa callback vào **Promise microtask queue**.
- `process.nextTick` đưa callback vào **nextTick queue**.
- `console.log("5")` chạy đồng bộ.
- Sau khi call stack rỗng: Node xử lý **`process.nextTick`** → rồi **Promise microtask** → sau đó Event
  Loop mới đi tới **timer**.

#### 2. Các nhóm task chính

**Synchronous task** — chạy ngay trên call stack:

```js
console.log("A");
function test() { console.log("B"); }
test();
console.log("C");
// Kết quả: A → B → C
```

**`process.nextTick`** — độ ưu tiên **rất cao** trong Node.js:

```js
Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));
// Kết quả:
// nextTick
// promise
```

`process.nextTick` **không thực sự là một phase** của Event Loop. Queue này được xử lý **ngay sau khi
đoạn JavaScript hiện tại hoàn thành**. Không nên lạm dụng — đệ quy `nextTick` có thể khiến Event Loop
không có cơ hội xử lý timer hoặc I/O:

```js
function loop() { process.nextTick(loop); }
loop(); // "bỏ đói" timer/I/O
```

**Promise và `queueMicrotask`** — cùng thuộc **microtask queue**, chạy theo thứ tự được đưa vào queue:

```js
queueMicrotask(() => console.log("microtask 1"));
Promise.resolve().then(() => console.log("microtask 2"));
// Kết quả: microtask 1 → microtask 2
```

Thứ tự tổng quát: **`process.nextTick` queue → Promise/`queueMicrotask` queue**.

**`setTimeout`** — `setTimeout(cb, 0)` **không** có nghĩa chạy ngay lập tức:

```js
setTimeout(() => console.log("timeout"), 0);
console.log("sync");
// Kết quả: sync → timeout
```

`0` nghĩa là callback **đủ điều kiện chạy sớm nhất** khi Event Loop đến **timers phase**, không phải
chạy sau đúng 0 ms.

**`setImmediate`** — chạy trong **check phase**. Ở **top-level**, thứ tự giữa `setImmediate` và
`setTimeout(..., 0)` **không đảm bảo** (có thể `timeout` trước hoặc `immediate` trước tùy điều kiện).
Nhưng **bên trong một callback I/O**, `setImmediate` **thường chạy trước** `setTimeout(..., 0)`:

```js
const fs = require("fs");
fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
// Kết quả ổn định:
// immediate
// timeout
```

Vì callback `fs.readFile` chạy trong **poll/I/O phase**, sau đó Event Loop đi tới **check phase** (setImmediate)
trước khi quay lại **timers phase** ở vòng kế.

#### 3. Các phase của Event Loop

```
┌───────────────────────────┐
│          timers           │  ← setTimeout / setInterval
├───────────────────────────┤
│     pending callbacks      │  ← một số callback hệ thống hoãn từ vòng trước (TCP/OS)
├───────────────────────────┤
│          idle              │  ← nội bộ Node
├───────────────────────────┤
│          poll              │  ← nhận & xử lý callback I/O (đọc file, network)
├───────────────────────────┤
│          check             │  ← setImmediate()
├───────────────────────────┤
│      close callbacks       │  ← socket.on("close", cb)
└───────────────────────────┘
```

- **Timers phase:** callback của `setTimeout()` / `setInterval()`.
- **Pending callbacks phase:** một số callback hệ thống bị trì hoãn từ vòng trước, thường liên quan TCP/OS.
- **Poll phase (rất quan trọng):** nhận event I/O mới, xử lý callback I/O; có thể **chờ I/O** nếu chưa
  có task khác. Callback `fs.readFile(...)` được xử lý liên quan tới phase này.
- **Check phase:** callback của `setImmediate()`.
- **Close callbacks phase:** callback đóng connection (`socket.on("close", cb)`).

#### 4. Microtask được xử lý khi nào?

Node.js xử lý microtask **sau khi callback hiện tại hoàn thành, trước khi chạy callback tiếp theo**:

```js
setTimeout(() => {
  console.log("timeout 1");
  Promise.resolve().then(() => console.log("promise inside timeout"));
}, 0);

setTimeout(() => console.log("timeout 2"), 0);

/* Kết quả:
timeout 1
promise inside timeout
timeout 2
*/
```

Không phải `timeout 1 → timeout 2 → promise inside timeout`, vì sau callback `timeout 1`, Node **xử lý
microtask trước** khi chạy callback timer tiếp theo.

#### 5. async/await thực thi như thế nào?

`async/await` được xây dựng dựa trên **Promise**. Phần trước `await` chạy đồng bộ; khi gặp `await`,
function **tạm dừng**, phần sau `await` được đưa vào **microtask queue**, code bên ngoài tiếp tục chạy.

```js
async function test() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
}
console.log("1");
test();
console.log("2");
/* Kết quả: 1 → A → 2 → B */
```

Ví dụ phức tạp hơn:

```js
async function foo() {
  console.log("foo start");
  await bar();
  console.log("foo end");
}
async function bar() { console.log("bar"); }

console.log("script start");
setTimeout(() => console.log("timeout"), 0);
foo();
Promise.resolve().then(() => console.log("promise"));
console.log("script end");

/* Kết quả:
script start
foo start
bar
script end
foo end
promise
timeout
*/
```

`foo end` được đưa vào microtask queue khi `await bar()` hoàn tất → nó được **đăng ký trước** callback
`.then()` phía dưới nên chạy trước `promise`.

#### 6. Ví dụ tổng hợp

```js
console.log("1");

setTimeout(() => {
  console.log("2");
  process.nextTick(() => console.log("3"));
  Promise.resolve().then(() => console.log("4"));
}, 0);

setImmediate(() => console.log("5"));
process.nextTick(() => console.log("6"));
Promise.resolve().then(() => console.log("7"));

console.log("8");

/* Kết quả phổ biến:
1
8
6
7
2
3
4
5
*/
```

Giải thích: `1, 8` đồng bộ → `6` nextTick → `7` Promise microtask → `2` timer; sau callback timer:
`3` nextTick → `4` Promise microtask → `5` setImmediate. *(Ở top-level, thứ tự timer 0ms vs
setImmediate không nên dùng làm logic nghiệp vụ phụ thuộc tuyệt đối.)*

#### 7. I/O không chạy trực tiếp trên main thread

Node.js dùng **OS async APIs**, **libuv**, và **thread pool** cho một số tác vụ (file system, DNS
trong một số trường hợp, crypto, compression).

```js
const fs = require("fs");
console.log("start");
fs.readFile("./data.txt", "utf8", (err, data) => {
  if (err) return console.error(err);
  console.log("file loaded");
});
console.log("end");
/* Kết quả: start → end → file loaded */
```

Main thread **không ngồi chờ** file đọc xong. Khi I/O hoàn tất, callback được đưa về để Event Loop xử lý.

#### 8. Event Loop không làm JavaScript chạy song song

Hai callback JavaScript **không chạy đồng thời** trên cùng một main thread:

```js
setTimeout(() => {
  console.log("A start");
  const start = Date.now();
  while (Date.now() - start < 3000) { /* block main thread */ }
  console.log("A end");
}, 0);

setTimeout(() => console.log("B"), 100);

/* Kết quả: A start → A end → B */
```

Dù timer `B` đặt 100 ms, nó vẫn phải chờ callback `A` chạy xong — đây gọi là **blocking Event Loop**.
Với **CPU-heavy task**, nên cân nhắc: **Worker Threads**, **Child Process**, **queue/background
worker**, hoặc **chia nhỏ computation** (xem [câu 4](#q4)).

#### 9. Bảng thứ tự cần nhớ

| Loại task | Thứ tự ưu tiên tương đối |
|---|---|
| Code đồng bộ | 1 |
| `process.nextTick` | 2 |
| Promise `.then`, `await`, `queueMicrotask` | 3 |
| Timer, I/O, `setImmediate` | Phụ thuộc phase |
| `setTimeout` | Timers phase |
| I/O callback | Poll phase |
| `setImmediate` | Check phase |

Cách nhớ ngắn: **Sync → nextTick → Promise → Event Loop phases**. Quy tắc thực tế quan trọng nhất:
**sau mỗi callback, Node.js xử lý `process.nextTick` và microtask trước khi chuyển sang callback tiếp theo.**

### 🎯 Tóm tắt trả lời phỏng vấn

- Node.js chạy JS trên **một main thread**; **Event Loop** điều phối mọi task. **Không phải đăng ký
  trước là chạy trước** — phụ thuộc **loại task** và **phase**.
- **Thứ tự ưu tiên:** **code đồng bộ → `process.nextTick` → Promise/`queueMicrotask` → các phase Event
  Loop** (timers/poll/check...). `nextTick` ưu tiên **cao nhất**, không phải một phase mà chạy ngay sau
  đoạn JS hiện tại.
- **Quy tắc vàng:** **sau mỗi callback, Node vét sạch microtask (nextTick → Promise) trước khi chạy
  callback kế tiếp** (xem ví dụ mục 4).
- **Các phase:** timers (`setTimeout`) → pending → poll (I/O) → check (`setImmediate`) → close.
  `setTimeout(0)` chỉ là "chạy sớm nhất khi tới timers phase". **Top-level:** `setTimeout(0)` vs
  `setImmediate` **không đảm bảo thứ tự**; **trong callback I/O:** `setImmediate` chạy trước.
- **`async/await` = Promise:** phần trước `await` đồng bộ, phần sau `await` vào microtask queue.
- **I/O không chạy trên main thread** (libuv + thread pool). **Đừng block Event Loop** bằng CPU-heavy
  task → dùng Worker Threads / Child Process / chia nhỏ ([câu 4](#q4)).

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Phân biệt process, thread và worker_threads trong Node.js

### 📖 Giới thiệu kiến thức

Câu này nối tiếp event loop ([câu 3](#q3)): event loop xử lý tốt **I/O-bound**, nhưng với **CPU-bound**
(tính toán nặng) thì cần **song song hóa thật sự**. Phải hiểu 3 khái niệm OS/Node sau.

**1. Process (tiến trình):**
Một **chương trình đang chạy**, do hệ điều hành cấp **vùng nhớ (memory) riêng biệt, độc lập**. Hai
process **không chia sẻ bộ nhớ trực tiếp** — muốn trao đổi phải qua **IPC** (Inter-Process
Communication: message, socket, pipe). Mỗi tiến trình Node mặc định có **một V8 + một event loop +
một luồng chính** riêng. Nặng (tốn RAM, khởi tạo chậm) nhưng **cô lập tốt**: một process crash không
kéo sập process khác.

**2. Thread (luồng):**
Đơn vị thực thi **bên trong một process**. Nhiều thread **chia sẻ chung vùng nhớ** của process → giao
tiếp nhanh nhưng dễ **race condition**, cần khóa (lock). Nhẹ hơn process. **Quan trọng:** code
**JavaScript của bạn chạy single-threaded** (một main thread). Tuy nhiên *bên dưới*, libuv vẫn có một
**thread pool** (mặc định 4 thread) để xử lý một số I/O (fs, DNS, crypto) — nhưng bạn **không viết JS
chạy song song** trên đó.

**3. `worker_threads` (module của Node):**
Cách Node cho phép chạy **JavaScript thật sự song song trên nhiều luồng** *trong cùng một process*.
Mỗi **Worker** có **V8 instance + event loop riêng**, chạy độc lập với main thread → **không block**
event loop chính. Dùng cho **CPU-bound** (xử lý ảnh, mã hóa, tính toán lớn). Giao tiếp với main thread
qua **message** (`postMessage`/`on('message')`) hoặc **`SharedArrayBuffer`** (chia sẻ bộ nhớ có kiểm soát).

**Phân biệt nhanh:**

| | **Process** (vd `child_process`/`cluster`) | **Thread (OS)** | **`worker_threads`** |
|---|---|---|---|
| **Bộ nhớ** | Riêng biệt, độc lập | Chung trong process | Riêng V8, nhưng chia sẻ được qua `SharedArrayBuffer` |
| **Giao tiếp** | IPC (message/socket) — chậm hơn | Trực tiếp qua shared memory | `postMessage` / `SharedArrayBuffer` |
| **Chi phí** | Nặng (RAM, khởi tạo chậm) | Nhẹ | Trung bình (nhẹ hơn process) |
| **Cô lập lỗi** | Tốt (crash độc lập) | Kém (chung không gian) | Khá (1 worker lỗi không sập main) |
| **Trong Node để làm gì** | Tận dụng **nhiều CPU core** (cluster nhân bản app) | Khái niệm nền (libuv thread pool) | **CPU-bound** trong 1 process, không block event loop |

**`cluster` vs `worker_threads` — điểm hay nhầm:**
- **`cluster` / `child_process`:** tạo **nhiều process Node** (mỗi cái event loop + memory riêng) →
  thường để **scale theo số CPU core**, mỗi worker process xử lý request (web server). Cô lập mạnh,
  không chia sẻ memory.
- **`worker_threads`:** **nhiều thread trong MỘT process**, chia sẻ memory được → hợp **tính toán nặng**
  cần chia sẻ dữ liệu lớn mà không tốn chi phí copy/IPC.

**Khi nào dùng gì:**
- **I/O-bound** (đa số web app: DB, API, file): **không cần** thread/worker — event loop + async đã đủ.
- **CPU-bound** (tính toán nặng làm nghẽn event loop): dùng **`worker_threads`**.
- **Tận dụng nhiều core cho web server / chịu tải:** dùng **`cluster`** (nhiều process) — hoặc để
  process manager (PM2) / container orchestration lo.

```js
// worker_threads — đẩy tính toán nặng sang luồng khác, KHÔNG block event loop chính
// main.js
const { Worker } = require('worker_threads');
const worker = new Worker('./heavy.js');
worker.on('message', (result) => console.log('Kết quả:', result));
worker.postMessage(1e9); // gửi dữ liệu sang worker

// heavy.js
const { parentPort } = require('worker_threads');
parentPort.on('message', (n) => {
  let sum = 0;
  for (let i = 0; i < n; i++) sum += i; // CPU-bound nặng
  parentPort.postMessage(sum);          // trả về main thread
});
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Process:** chương trình đang chạy, **bộ nhớ riêng độc lập**, giao tiếp qua **IPC**, cô lập tốt
  nhưng nặng. Mỗi process Node có **V8 + event loop riêng**.
- **Thread:** đơn vị chạy **bên trong process, chia sẻ memory** → nhanh nhưng dễ race condition. Code
  **JS của bạn là single-threaded**; libuv chỉ có thread pool nội bộ cho một số I/O.
- **`worker_threads`:** module Node cho chạy **JS song song trên nhiều luồng trong 1 process**, mỗi
  worker có **V8 + event loop riêng**, giao tiếp qua `postMessage`/`SharedArrayBuffer` → dùng cho
  **CPU-bound**, không block event loop chính.
- **`cluster`/`child_process` (process) vs `worker_threads` (thread):** cluster nhân bản **nhiều
  process** để tận dụng **nhiều CPU core** (web server); worker_threads là **nhiều thread chia sẻ
  memory** trong 1 process cho tính toán nặng.
- **Chọn:** I/O-bound → event loop async là đủ; **CPU-bound → worker_threads**; **scale theo core →
  cluster** (hoặc PM2/container).

[↑ Mục lục](#mục-lục)

---

<a id="q5"></a>
## 5. Phân biệt npm, npx, pnpm và yarn

### 📖 Giới thiệu kiến thức

**Phân biệt CỐT LÕI trước tiên:** trong 4 cái tên này, **npm, yarn, pnpm là PACKAGE MANAGER** (trình
quản lý gói), còn **npx là PACKAGE RUNNER** (trình **chạy** gói) — **khác loại**, không cùng vai trò.

**Package manager** là công cụ quản lý **dependency**: cài/gỡ/cập nhật thư viện, đọc **`package.json`**,
**resolve version**, tải gói từ **registry** (npmjs.com) và dựng **`node_modules`**. Ba package manager
phổ biến của hệ Node — **npm, yarn, pnpm** — dùng **chung registry npm**, chỉ khác **cách làm việc, hiệu
năng & cú pháp**.

**① npm (Node Package Manager):**
- **Đi kèm sẵn** khi cài Node.js (mặc định, không cần cài thêm), lâu đời nhất.
- Thời 2015–2016 từng bị chê: cài **chậm** (tuần tự), **không nhất quán** giữa các máy, thiếu cache.
- **Cải thiện mạnh:** npm **v5 (2017)** có **`package-lock.json`** (deterministic) + nhanh hơn nhiều;
  **v7+** có **workspaces** (monorepo) → nay **gần ngang yarn**.

**② yarn (Yet Another Resource Negotiator):**
- Do **Facebook** ra mắt **2016** để fix npm thời đó: **`yarn.lock`** mặc định (deterministic), **cài
  song song** (nhanh hơn), **cache offline** + kiểm tra **checksum/integrity**.
- **Yarn Berry (v2+)** có **Plug'n'Play (PnP)** — **bỏ `node_modules`**, đọc gói qua `.pnp.cjs`.

**③ pnpm (performant npm):**
- Package manager thế hệ mới, tập trung vào **hiệu năng & tiết kiệm disk**. Cơ chế đặc trưng:
  **content-addressable store toàn cục** (lưu mỗi version gói **một lần duy nhất** trên máy) +
  **hard link/symlink** vào `node_modules` → cài **rất nhanh**, **tiết kiệm disk khủng**.
- `node_modules` cấu trúc **chặt (non-flat)** → tránh **phantom dependency** (chỉ import được gói **đã
  khai báo** trong `package.json`). Lockfile: **`pnpm-lock.yaml`**. Rất hợp **monorepo**.

**④ npx (Node Package eXecute) — KHÁC LOẠI:**
- Là **package runner**, đi kèm npm từ **v5.2 (2017)**. Nhiệm vụ: **chạy CLI của một package mà KHÔNG
  cần cài global**.
- Cơ chế: nếu gói chưa có, npx **tải tạm → chạy → không để lại** (giữ máy sạch, luôn dùng bản mới nhất).
  Cũng dùng để chạy binary trong `node_modules/.bin`.
- Vd kinh điển: `npx create-react-app my-app` (scaffolding một lần). Tương đương ở pnpm/yarn là
  **`pnpm dlx`** / **`yarn dlx`**.

> **Nhớ:** npm/yarn/pnpm **CÀI & QUẢN LÝ** gói; npx **CHẠY** gói (one-off, không cài global).

**So sánh 3 package manager:**

| Tiêu chí | **npm** | **yarn** | **pnpm** |
|---|---|---|---|
| Cài kèm Node | ✅ mặc định | ❌ cài thêm | ❌ cài thêm |
| Lockfile | `package-lock.json` | `yarn.lock` | `pnpm-lock.yaml` |
| Cơ chế `node_modules` | Flat (sao chép) | Flat (sao chép) | **Symlink + store chung** |
| Tiết kiệm disk | Thấp | Thấp | **Cao nhất** |
| Tốc độ | Tốt (bản mới) | Nhanh | **Nhanh nhất (thường)** |
| Phantom dependency | Có thể | Có thể | **Chặn được** |
| Điểm đặc biệt | Chuẩn phổ biến | PnP (Berry) | Store + non-flat |

**Lệnh tương đương (hay bị hỏi):**

| Tác vụ | npm | yarn | pnpm |
|---|---|---|---|
| Cài tất cả dependency | `npm install` | `yarn` | `pnpm install` |
| Thêm package | `npm install <pkg>` | `yarn add <pkg>` | `pnpm add <pkg>` |
| Thêm dev dependency | `npm install -D <pkg>` | `yarn add -D <pkg>` | `pnpm add -D <pkg>` |
| Gỡ package | `npm uninstall <pkg>` | `yarn remove <pkg>` | `pnpm remove <pkg>` |
| Chạy script | `npm run <script>` | `yarn <script>` | `pnpm <script>` |
| Chạy CLI one-off | `npx <cli>` | `yarn dlx <cli>` | `pnpm dlx <cli>` |

```bash
# Cài & quản lý gói (package manager)
npm install axios        # npm
yarn add axios           # yarn
pnpm add axios           # pnpm

# Chạy CLI một lần KHÔNG cài global (package runner)
npx create-react-app app # tải tạm → chạy → không để lại global
```

> **Lưu ý thực tế:** ① **luôn commit lockfile** để cả team & CI cài cùng version. ② **Không trộn**
> nhiều package manager trong một repo (sinh nhiều lockfile xung đột). ③ Chọn manager nào phần lớn là
> **thói quen team**; **pnpm** đang được ưa chuộng cho **monorepo / dự án lớn** nhờ tiết kiệm disk.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Khác loại trước:** **npm / yarn / pnpm** là **package manager** (cài & quản lý gói); **npx** là
  **package runner** (**chạy** CLI một lần, **không cài global**).
- **npm:** kèm Node, lâu đời; v5+ có `package-lock.json`, v7+ workspaces → nay đủ tốt. **yarn:**
  Facebook 2016, `yarn.lock` + cài song song + cache; bản **Berry (v2+)** có **PnP bỏ `node_modules`**.
- **pnpm:** dùng **store chung + symlink** → **tiết kiệm disk & nhanh nhất**, `node_modules` **non-flat**
  chặn **phantom dependency**, lockfile `pnpm-lock.yaml` — rất hợp **monorepo**.
- **npx:** chạy CLI của package mà **không cần cài global** (vd `npx create-react-app`), tải tạm rồi bỏ;
  tương đương **`yarn dlx`** / **`pnpm dlx`**.
- **Khác chính:** lockfile riêng + **cú pháp lệnh** (`install`/`add`/`remove`) + cơ chế `node_modules`.
  Thực tế: **commit lockfile, không trộn manager**; pnpm ưu thế cho dự án lớn.

[↑ Mục lục](#mục-lục)

<a id="q5-1"></a>
### 5.1. Phân biệt devDependencies và dependencies trong package.json

#### 📖 Giới thiệu kiến thức

Trong `package.json`, dependency được khai báo ở nhiều trường khác nhau. Hai trường hay gặp & hay bị
hỏi nhất là **`dependencies`** và **`devDependencies`** — khác nhau ở câu hỏi cốt lõi: **"gói này có
cần khi app CHẠY ở production không?"**

**`dependencies` — gói cần để app CHẠY (runtime):**
- Những thư viện app **phụ thuộc lúc chạy thật** ở production. Vd: **express**, **axios**, **lodash**,
  **react**, **mysql2**.
- Được cài khi người khác cài package của bạn, hoặc khi deploy production.

**`devDependencies` — gói chỉ cần lúc PHÁT TRIỂN / build / test:**
- Công cụ phục vụ **dev, build, test, lint** — **không cần** khi app đã chạy production. Vd:
  **typescript**, **webpack/vite**, **babel**, **eslint/prettier**, **jest**, **nodemon**, **`@types/*`**.

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "nodemon": "^3.0.0",
    "jest": "^29.0.0",
    "@types/express": "^4.17.0"
  }
}
```

**Cách thêm vào đúng nhóm:**

```bash
npm install express        # → dependencies (mặc định)
npm install -D typescript  # → devDependencies  (-D = --save-dev)
# yarn tương đương:
yarn add express
yarn add -D typescript
```

**Điểm CỐT LÕI — lúc cài khác nhau thế nào (hay hỏi):**
- **`npm install`** (mặc định, môi trường dev) → cài **CẢ HAI** nhóm.
- **`npm install --omit=dev`** / **`npm ci --omit=dev`** / **`NODE_ENV=production`** → **chỉ cài
  `dependencies`**, **bỏ `devDependencies`**.

→ Đây chính là **lý do phải tách**: ở production chỉ cài gói runtime → **image/bundle nhỏ hơn, cài
nhanh hơn, giảm bề mặt tấn công (ít gói = ít lỗ hổng)**. Vd Docker build production thường `npm ci
--omit=dev`.

> **Lưu ý:** ① Với **app frontend build ra bundle** (Vite/webpack), ranh giới linh hoạt hơn vì code được
> bundle sẵn — nhưng với **Node backend chạy trực tiếp source** thì phân biệt rất quan trọng. ② Còn 2
> trường khác để phân biệt: **`peerDependencies`** (gói mà **host phải cung cấp**, vd plugin yêu cầu
> `react` ở dự án dùng nó — tránh trùng bản) và **`optionalDependencies`** (cài lỗi cũng **không làm
> fail** quá trình install).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Câu hỏi phân loại: **"gói có cần khi app chạy production không?"** Cần → **`dependencies`** (express,
  axios, react...); chỉ dev/build/test → **`devDependencies`** (typescript, eslint, jest, nodemon,
  `@types/*`, bundler).
- **Thêm:** `npm install <pkg>` → dependencies; `npm install -D <pkg>` → devDependencies (yarn: `add`
  vs `add -D`).
- **Khác biệt quan trọng lúc cài:** `npm install` (dev) cài **cả hai**; còn **`npm ci --omit=dev`** /
  `--production` / `NODE_ENV=production` **chỉ cài `dependencies`**.
- **Lý do tách:** production nhẹ hơn, cài nhanh hơn, **ít bề mặt tấn công** → image gọn (Docker hay dùng
  `npm ci --omit=dev`).
- Biết thêm **`peerDependencies`** (host phải cung cấp) và **`optionalDependencies`** (cài lỗi không fail).

[↑ Mục lục](#mục-lục)

---

<a id="q6"></a>
## 6. Thư viện node:crypto trong JS/TS là gì?

### 📖 Giới thiệu kiến thức

**`crypto`** là một **core module có sẵn** của Node.js (không cần cài qua npm) cung cấp toàn bộ các
tác vụ **mật mã học (cryptography)**: **băm (hash)**, **HMAC**, **sinh số ngẫu nhiên an toàn**, **mã
hóa/giải mã đối xứng & bất đối xứng**, **ký/xác thực chữ ký số**, và **băm mật khẩu** (scrypt/pbkdf2).
Bên dưới nó gọi tới thư viện **OpenSSL** (viết bằng C) nên **nhanh và chuẩn**.

Cách import (nên dùng prefix `node:` để nói rõ đây là core module, tránh nhầm với package npm):

```ts
import crypto from 'node:crypto';        // ESM / TypeScript
// hoặc
const crypto = require('node:crypto');   // CommonJS
```

> **Liên hệ event loop:** một số hàm bất đồng bộ của crypto (`pbkdf2`, `scrypt`, `randomBytes` bản
> async, `generateKeyPair`) chạy trên **libuv thread pool** (mặc định 4 thread) → **không block main
> thread**. Xem [câu 3](#q3) (event loop) và [câu 4](#q4) (thread pool).

Các nhóm chức năng chính:

**1. Hash — băm một chiều (`createHash`):**
Băm dữ liệu bất kỳ ra một chuỗi cố định (SHA-256, SHA-512, MD5...). Dùng cho **checksum/toàn vẹn dữ
liệu**, tạo ETag, so sánh file — **KHÔNG dùng để lưu mật khẩu** (quá nhanh, xem bên dưới).

```ts
const hash = crypto.createHash('sha256')
  .update('hello world')
  .digest('hex');   // => chuỗi hex 64 ký tự
```

**2. HMAC — băm có khóa bí mật (`createHmac`):**
Băm kèm một **secret key** → dùng để **xác thực tính toàn vẹn + nguồn gốc** (message authentication).
Đây là nền tảng của **JWT ký HS256**, xác thực webhook (GitHub, Stripe...).

```ts
const sig = crypto.createHmac('sha256', 'my-secret-key')
  .update('payload')
  .digest('hex');
```

**3. Random — sinh giá trị ngẫu nhiên an toàn mật mã (CSPRNG):**
Khác `Math.random()` (KHÔNG an toàn, đoán được). Dùng cho **token, session id, salt, mã OTP, reset
password**.

```ts
crypto.randomBytes(32).toString('hex');  // token ngẫu nhiên 64 ký tự hex
crypto.randomUUID();                      // UUID v4 chuẩn, vd '3b12...-...'
crypto.randomInt(1, 100);                 // số nguyên ngẫu nhiên an toàn
```

**4. Băm mật khẩu — `scrypt` / `pbkdf2` (key derivation):**
Khi **không dùng bcrypt/Argon2** (xem [algorithm.md](../Common/algorithm.md)), crypto có sẵn **scrypt**
(memory-hard, khuyến nghị) và **pbkdf2**. Đều là **hash chậm có salt + cost** để lưu mật khẩu an toàn.

```ts
// Đăng ký: sinh salt + băm
const salt = crypto.randomBytes(16).toString('hex');
const hash = crypto.scryptSync(password, salt, 64).toString('hex');
// lưu `${salt}:${hash}` vào DB

// Đăng nhập: băm lại + so sánh AN TOÀN (constant-time, chống timing attack)
const [salt2, key2] = stored.split(':');
const derived = crypto.scryptSync(input, salt2, 64);
const ok = crypto.timingSafeEqual(Buffer.from(key2, 'hex'), derived);
```

**5. Mã hóa đối xứng (symmetric) — `createCipheriv` / `createDecipheriv`:**
Một khóa **dùng chung** để mã hóa & giải mã (AES-256-GCM khuyến nghị vì có kiểm tra toàn vẹn). Dùng
để **mã hóa dữ liệu nhạy cảm** lưu trong DB (số thẻ, token...). Cần **IV (initialization vector)** ngẫu
nhiên mỗi lần.

```ts
const key = crypto.randomBytes(32);       // khóa AES-256
const iv  = crypto.randomBytes(12);       // IV cho GCM
const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
const enc = Buffer.concat([cipher.update('secret', 'utf8'), cipher.final()]);
const tag = cipher.getAuthTag();          // tag xác thực toàn vẹn
```

**6. Mã hóa bất đối xứng & chữ ký số (asymmetric):**
Cặp khóa **public/private** (RSA, EC). Public key mã hóa → private key giải; hoặc private key **ký** →
public key **xác thực**. Nền tảng của **TLS/HTTPS**, **JWT RS256/ES256**, chữ ký điện tử.

```ts
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', { modulusLength: 2048 });
const signature = crypto.sign('sha256', Buffer.from('data'), privateKey);
const valid = crypto.verify('sha256', Buffer.from('data'), publicKey, signature);
```

**Lưu ý interviewer hay đào sâu:**
- **Hash ≠ mã hóa:** hash **một chiều** (không giải ngược); mã hóa **hai chiều** (giải được bằng khóa).
- **Không dùng `createHash('sha256')` để lưu mật khẩu** — quá nhanh, dễ brute-force. Dùng **scrypt/
  pbkdf2** (trong crypto) hoặc **bcrypt/Argon2** (thư viện ngoài).
- **Luôn dùng `crypto.randomBytes`/`randomUUID`**, KHÔNG dùng `Math.random()` cho token/salt/secret.
- **So sánh hash/secret bằng `crypto.timingSafeEqual`**, không dùng `===` → chống **timing attack**.
- **Ưu tiên bản async** (`crypto.scrypt(...)` callback/promise) trong web server để **không block event
  loop** với dữ liệu lớn; bản `...Sync` chỉ nên dùng ở script/CLI.

### 🎯 Tóm tắt trả lời phỏng vấn

- **`node:crypto`** là **core module** sẵn có của Node (bọc **OpenSSL**), cung cấp mọi tác vụ mật mã:
  **hash, HMAC, random an toàn, mã hóa đối xứng/bất đối xứng, chữ ký số, băm mật khẩu**.
- **Nhóm chính:** `createHash` (băm toàn vẹn, checksum) · `createHmac` (băm có khóa — nền JWT HS256/
  webhook) · `randomBytes`/`randomUUID` (token/salt an toàn) · `scrypt`/`pbkdf2` (băm mật khẩu chậm) ·
  `createCipheriv` (AES đối xứng) · `generateKeyPair` + `sign`/`verify` (RSA/EC, bất đối xứng).
- **Phân biệt cốt lõi:** **hash một chiều** (không giải ngược) ≠ **mã hóa hai chiều** (giải được bằng khóa).
- **Best practice:** không dùng `sha256` để lưu mật khẩu (dùng scrypt/pbkdf2 hoặc bcrypt/Argon2); dùng
  `randomBytes` thay `Math.random()`; so sánh secret bằng **`timingSafeEqual`** (chống timing attack).
- **Liên hệ runtime:** các hàm async (`pbkdf2`/`scrypt`/`generateKeyPair`) chạy trên **libuv thread
  pool** → không block event loop; ưu tiên bản async trên server, `...Sync` chỉ cho script.

[↑ Mục lục](#mục-lục)

---

<a id="q6-1"></a>
### 6.1. Thư viện bcryptjs là gì? (so với bcrypt native)

#### 📖 Giới thiệu kiến thức

**`bcryptjs`** là thư viện npm hiện thực thuật toán băm mật khẩu **bcrypt** **hoàn toàn bằng JavaScript
thuần** (pure JS), **zero-dependency**, kèm sẵn **type TypeScript**. Vì thuần JS nên nó chạy được ở
**mọi nơi**: Node.js, **trình duyệt**, và môi trường **serverless/edge** (Cloudflare Workers, Lambda...).
Cài đặt: `npm i bcryptjs`.

**Vì sao cần thư viện ngoài?** Module lõi `node:crypto` ([câu 6](#q6)) **chỉ có** `scrypt`/`pbkdf2` để
băm mật khẩu, **KHÔNG có bcrypt hay Argon2** sẵn. Muốn dùng **bcrypt** thì phải cài thư viện.

**Hai lựa chọn triển khai bcrypt trên Node — hay bị hỏi so sánh:**

| | **`bcryptjs`** (pure JS) | **`bcrypt`** (native, node.bcrypt.js) |
|---|--------------------------|---------------------------------------|
| Cài đặt | `npm i bcryptjs` — **không cần compile** | Cần **node-gyp / C++ toolchain** khi build |
| Tốc độ | **Chậm hơn ~30%** | Nhanh hơn (binding C++) |
| Chạy ở | Node **+ browser + edge/serverless** | Chỉ Node (cần native addon) |
| Phụ thuộc | **Zero dependency** | Có native binary theo OS/arch |
| Tương thích hash | **Tương thích lẫn nhau** (cùng `$2b$`) | Tương thích lẫn nhau |

→ Chọn **`bcryptjs`** khi muốn **build đơn giản, chạy đa nền tảng, serverless/edge** (không compile được
native); chọn **`bcrypt` native** khi cần **tối đa hiệu năng** và môi trường build ổn định.

**API chính** (mỗi hàm đều có bản **async** — hỗ trợ **Promise & callback** — và bản **`...Sync`**):

| Hàm | Công dụng |
|-----|-----------|
| `genSalt(rounds = 10)` | Sinh salt với **cost rounds** (mặc định 10) |
| `hash(data, saltOrRounds)` | Băm mật khẩu (truyền salt sẵn, **hoặc chỉ số rounds** → tự sinh salt) |
| `compare(data, hash)` | **So khớp** mật khẩu với hash — **constant-time** sẵn |
| `getRounds(hash)` | Đọc số rounds đã dùng → biết có cần **rehash** không |
| `truncates(pwd)` | Kiểm tra mật khẩu **có bị cắt do vượt 72 byte** không |

```ts
import bcrypt from 'bcryptjs';

// ĐĂNG KÝ — truyền thẳng số rounds, bcryptjs tự sinh salt & nhúng vào hash
const hash = await bcrypt.hash(plainPassword, 10);   // hash 60 ký tự, prefix $2b$10$...
// INSERT INTO users(email, password_hash) VALUES(?, hash)

// ĐĂNG NHẬP — so khớp (constant-time bên trong, KHÔNG tự dùng ===)
const ok = await bcrypt.compare(inputPassword, hashFromDB);

// Nâng cost theo thời gian: nếu hash cũ yếu hơn cấu hình hiện tại → băm lại
if (ok && bcrypt.getRounds(hashFromDB) < 12) {
  const newHash = await bcrypt.hash(inputPassword, 12);   // lưu lại vào DB
}
```

**Các điểm quan trọng / bẫy hay hỏi:**

- **Giới hạn 72 byte** (kế thừa từ bcrypt): mật khẩu dài hơn 72 byte UTF-8 sẽ **bị cắt âm thầm** (emoji/
  Unicode tốn nhiều byte). Nên validate bằng `bcrypt.truncates(pwd)` hoặc giới hạn độ dài đầu vào.
- **Dùng `compare`, đừng tự so hash bằng `===`** — `compare` đã **constant-time** (chống timing attack,
  xem [security.md câu 14](../Common/security.md#q14)); và vì mỗi hash có **salt riêng** nên cùng mật khẩu
  ra hash khác nhau, so chuỗi trực tiếp là **sai**.
- **Async ≠ thread pool:** khác `node:crypto` (chạy trên **libuv thread pool**, [câu 6](#q6)), bản async
  của `bcryptjs` là **JS thuần chia nhỏ công việc qua `setImmediate`** để **nhường event loop** — vẫn trên
  main thread. Vẫn nên dùng bản **async** trên web server; tránh `hashSync`/`compareSync` trong request
  handler vì **block event loop** ([câu 3](#q3)).
- **Cost mặc định 10** — nên chỉnh sao cho mỗi lần băm ~**200–500 ms** trên production; máy mạnh lên thì
  **tăng rounds**.
- **So với Argon2:** với hệ mới, **Argon2id** (thư viện `argon2`) **memory-hard**, kháng GPU/ASIC tốt hơn
  bcrypt ([algorithm.md câu 3](../Common/algorithm.md#q3)); chọn `bcryptjs` khi ưu tiên **zero native
  dependency / chạy đa nền tảng**.

> Nguyên tắc chung không đổi: **không tự cài thuật toán, không tự nối salt** — để thư viện lo salt, cost
> và so sánh constant-time.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`bcryptjs`** = hiện thực **bcrypt thuần JavaScript**, **zero-dependency**, có TypeScript, chạy được
  **Node + browser + serverless/edge** vì **không cần compile native**. Dùng để **lưu mật khẩu an toàn**.
- **Vì sao cần:** `node:crypto` chỉ có `scrypt`/`pbkdf2`, **không có bcrypt** → phải cài thư viện.
- **`bcryptjs` (pure JS) vs `bcrypt` (native C++):** bcryptjs **chậm hơn ~30%** nhưng **không cần
  node-gyp**, chạy đa nền tảng; bcrypt native nhanh hơn nhưng cần toolchain build. **Hash tương thích lẫn
  nhau** (`$2b$`).
- **API:** `genSalt`, `hash(data, rounds)`, `compare`, `getRounds`, `truncates` — mỗi hàm có bản **async
  (Promise/callback)** và **Sync**. Mặc định **cost 10**, hash **60 ký tự**, salt tự nhúng.
- **Bẫy:** giới hạn **72 byte**; dùng **`compare`** (constant-time), không so `===`; bản **async của
  bcryptjs chia nhỏ qua `setImmediate`** chứ không phải thread pool → vẫn ưu tiên async, tránh `...Sync`
  trong request; hệ mới cân nhắc **Argon2id** memory-hard.

[↑ Mục lục](#mục-lục)

---

<!--
NHẮC FORMAT (xem /CLAUDE.md §4):

<a id="qN"></a>
## N. <Câu hỏi>

### 📖 Giới thiệu kiến thức
...

### 🎯 Tóm tắt trả lời phỏng vấn
...

[↑ Mục lục](#mục-lục)
-->
