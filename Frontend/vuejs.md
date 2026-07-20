<a id="mục-lục"></a>
# Frontend — Vue.js

> Vue.js · Composition API · Reactivity · Vuex/Pinia · Vue Router · Nuxt.js · Lifecycle.

## 📑 Mục lục

1. [Virtual DOM là gì?](#q1)
   - 1.1. [Phân biệt Virtual DOM và Real DOM](#q1-1)
   - 1.2. [Virtual DOM nằm ở vùng nhớ nào?](#q1-2)
   - 1.3. [Cách Vue 2 tối ưu diffing Virtual DOM](#q1-3)
   - 1.4. [Cách Vue 3 tối ưu diffing Virtual DOM](#q1-4)
2. [Phân biệt v-if và v-show](#q2)
3. [Vòng đời của Vue.js (lifecycle hooks)](#q3)
4. [Phân biệt watch, computed và methods](#q4)
5. [Phân biệt slot và scoped slot](#q5)
6. [Composition API là gì?](#q6)
   - 6.1. [Composables trong Vue 3](#q6-1)
7. [Vuex là gì?](#q7)
   - 7.1. [Pinia là gì? (Pinia vs Vuex)](#q7-1)
8. [Phân biệt Nuxt.js và Vue.js](#q8)
   - 8.1. [Phân biệt SSR, CSR và SSG](#q8-1)
   - 8.2. [So sánh tốc độ giữa Nuxt và Vue](#q8-2)
   - 8.3. [Hydration là gì?](#q8-3)

---

<a id="q1"></a>
## 1. Virtual DOM là gì?

### 📖 Giới thiệu kiến thức

**Trước hết — DOM thật (Real DOM):** DOM (Document Object Model) là **cây object** mà trình duyệt dựng
từ HTML để biểu diễn trang. Mỗi thẻ HTML là một node. Vấn đề: **thao tác DOM thật rất tốn kém** — mỗi
lần sửa (thêm/xóa/đổi thuộc tính) có thể kích hoạt **reflow (tính lại layout)** và **repaint (vẽ lại)**,
rất chậm nếu cập nhật nhiều node liên tục. Trong app hiện đại, UI thay đổi liên tục theo state → nếu mỗi
thay đổi đều đụng thẳng DOM thật thì hiệu năng kém.

**Virtual DOM (VDOM) là gì:** là một **biểu diễn "ảo" của DOM bằng plain JavaScript object** — một cây
node nhẹ (VNode) nằm **trong bộ nhớ**, mô tả UI **nên trông như thế nào**. Nó **không phải** DOM thật,
chỉ là bản mô tả để so sánh và tính toán. Thao tác trên object JS thì **rẻ hơn nhiều** so với đụng DOM
thật.

```js
// Một VNode về bản chất chỉ là object mô tả: tag, props, children
const vnode = {
  tag: "div",
  props: { id: "app", class: "box" },
  children: [
    { tag: "h1", props: {}, children: ["Hello"] },
    { tag: "p",  props: {}, children: ["Virtual DOM"] },
  ],
};
// ↔ tương ứng: <div id="app" class="box"><h1>Hello</h1><p>Virtual DOM</p></div>
```

**Cơ chế hoạt động — 3 bước (render → diff → patch):**

1. **Render:** khi state thay đổi, framework tạo ra một **cây VDOM mới** (từ template/render function).
2. **Diffing (so sánh):** so **VDOM mới** với **VDOM cũ** bằng thuật toán **diff** để tìm ra **khác biệt
   tối thiểu** (node nào thêm/xóa/đổi). Quá trình so + quyết định cập nhật gọi là **reconciliation**.
3. **Patch:** chỉ **áp những thay đổi thật sự cần** lên DOM thật (**batch**, minimal update) — không vẽ
   lại toàn bộ cây.

```
State đổi → tạo VDOM mới ──diff──▶ tìm khác biệt ──patch──▶ chỉ cập nhật node đổi lên Real DOM
                  ▲
             VDOM cũ (giữ lại để so sánh)
```

**Vì sao dùng Virtual DOM:**

- **Giảm thao tác DOM thật:** gom nhiều thay đổi, chỉ cập nhật phần khác biệt → tránh reflow/repaint thừa.
- **Declarative UI:** lập trình viên chỉ **mô tả UI theo state** ("UI = f(state)"), không tự tay thao
  tác DOM từng bước (imperative) → code dễ đọc, ít bug.
- **Trừu tượng hóa & đa nền tảng:** vì VDOM là mô tả độc lập, có thể render ra nơi khác ngoài trình
  duyệt — **SSR** (server-side rendering), **native** (mobile), test không cần DOM thật.

**Trong Vue:** `template` được **compiler biên dịch thành render function** trả về **VNode**. Vue 3 tối
ưu diffing nhờ **compiler thông minh**: **static hoisting** (node tĩnh chỉ tạo 1 lần), **patch flags**
(đánh dấu chỗ nào động để bỏ qua phần tĩnh khi diff), **block tree** → nhanh hơn Vue 2 rõ rệt. (React
cũng dùng VDOM với cơ chế reconciliation + Fiber.)

```js
import { h } from "vue";
// render function: h(tag, props, children) → tạo VNode
export default {
  render() {
    return h("div", { class: "box" }, [
      h("h1", "Hello"),
      h("p", this.message), // phần động: khi message đổi → diff & patch chỉ chỗ này
    ]);
  },
};
```

> **Lưu ý quan trọng (hay bị hỏi bẫy):** Virtual DOM **không phải luôn nhanh hơn** thao tác DOM trực
> tiếp — bản thân việc diff cũng tốn CPU. Giá trị thật của nó là **"đủ nhanh + trải nghiệm lập trình tốt"**
> (declarative, dễ bảo trì) cho app phức tạp. Một số framework **bỏ VDOM** để nhanh hơn: **Svelte**
> (compile ra thao tác DOM trực tiếp lúc build), **SolidJS** (fine-grained reactivity), và **Vue Vapor
> mode** đang phát triển theo hướng đó.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Virtual DOM** = **bản sao "ảo" của DOM bằng object JavaScript** nằm trong bộ nhớ, mô tả UI nên trông
  thế nào. Thao tác trên object JS **rẻ hơn nhiều** so với đụng DOM thật (vốn gây **reflow/repaint** tốn kém).
- **Cơ chế 3 bước:** state đổi → tạo **VDOM mới** → **diff** với VDOM cũ tìm khác biệt tối thiểu
  (**reconciliation**) → **patch** chỉ cập nhật node thật sự đổi lên Real DOM (batch, minimal update).
- **Lợi ích:** giảm thao tác DOM thật, cho phép **declarative UI** ("UI = f(state)"), và trừu tượng hóa
  để **SSR / render đa nền tảng / test** dễ hơn.
- **Trong Vue:** template được compile thành **render function → VNode**; Vue 3 tối ưu diff bằng
  **static hoisting + patch flags + block tree** (nhanh hơn Vue 2). React cũng dùng VDOM.
- **Bẫy:** VDOM **không phải luôn nhanh hơn** DOM trực tiếp (diff cũng tốn CPU) — nó đánh đổi lấy **DX &
  khả năng bảo trì**; Svelte/SolidJS/Vue Vapor chọn hướng bỏ VDOM để tối ưu hơn.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Phân biệt Virtual DOM và Real DOM

#### 📖 Giới thiệu kiến thức

Đây là phần đối chiếu trực tiếp hai khái niệm ở [câu 1](#q1). Điểm cần nắm trước tiên: **Virtual DOM
KHÔNG thay thế Real DOM** — nó là một **lớp trung gian** trong bộ nhớ giúp tính toán **cần đổi gì**, rồi
**cuối cùng vẫn phải cập nhật xuống Real DOM** thì người dùng mới thấy. VDOM tối ưu **quá trình đi tới**
Real DOM, chứ không loại bỏ Real DOM.

**Real DOM:** cây object **thật** do trình duyệt dựng từ HTML và **render trực tiếp lên màn hình**. Mỗi
node "nặng" (rất nhiều thuộc tính/method). Sửa Real DOM → dễ kích hoạt **reflow (tính lại layout)** +
**repaint (vẽ lại)** → **tốn kém**, nhất là khi cập nhật nhiều lần liên tục.

**Virtual DOM:** bản mô tả **nhẹ** của UI bằng **object JavaScript** nằm **trong bộ nhớ**, **không** hiển
thị trực tiếp. Thao tác trên nó rẻ; framework dùng **diff + patch** để dồn thay đổi và cập nhật Real DOM
**tối thiểu**.

**Bảng so sánh:**

| Tiêu chí | **Real DOM** | **Virtual DOM** |
|---|---|---|
| **Bản chất** | Cây object **thật** của trình duyệt | Object JS **nhẹ** trong bộ nhớ, mô tả UI |
| **Hiển thị** | Là cái người dùng **thật sự thấy** | Không hiển thị — chỉ là lớp trung gian |
| **Cập nhật** | Ghi thẳng, phản ánh ngay lên màn hình | Cập nhật lên bản ảo trước (chưa lên UI) |
| **Chi phí thao tác** | **Đắt** (reflow/repaint mỗi lần sửa) | **Rẻ** (chỉ là thao tác object JS) |
| **Cách áp thay đổi** | Có xu hướng cập nhật trực tiếp/toàn phần | **Diff** tìm khác biệt → **patch** tối thiểu |
| **Hiệu năng khi đổi nhiều** | Chậm (dễ reflow thừa) | Nhanh hơn (batch + minimal update) |
| **Độ nặng mỗi node** | Nặng (nhiều property của DOM API) | Nhẹ (chỉ `tag`, `props`, `children`) |
| **Thao tác trực tiếp** | Được (`querySelector`, `appendChild`...) | Không đụng trực tiếp — qua framework |

**Quan hệ giữa hai bên (nối [câu 1](#q1)):**

```
State đổi → tạo Virtual DOM mới ─diff→ so với VDOM cũ ─patch→ chỉ cập nhật phần đổi lên REAL DOM → UI
```

> **Chốt ý:** Real DOM là "đích đến" (thứ hiển thị); Virtual DOM là "bản nháp trong bộ nhớ" để tính ra
> thay đổi nhỏ nhất trước khi đụng vào đích đến đắt đỏ đó. Không có VDOM vẫn cập nhật được Real DOM —
> chỉ là dễ chậm và code imperative hơn.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Real DOM** = cây object **thật** của trình duyệt, **render lên màn hình**, node **nặng**, sửa trực
  tiếp gây **reflow/repaint** → **đắt**. **Virtual DOM** = bản mô tả **nhẹ** bằng **object JS trong bộ
  nhớ**, **không hiển thị**, thao tác **rẻ**.
- **Khác cốt lõi:** Real DOM cập nhật thẳng lên UI (tốn kém); Virtual DOM cập nhật bản ảo trước rồi
  **diff + patch** để chỉ đụng Real DOM ở **phần thật sự đổi** (batch, tối thiểu) → nhanh hơn khi UI
  thay đổi nhiều.
- **Mấu chốt:** VDOM **không thay thế** Real DOM — nó là **lớp trung gian** tối ưu; cuối cùng vẫn phải
  patch xuống Real DOM thì người dùng mới thấy.

[↑ Mục lục](#mục-lục)

<a id="q1-2"></a>
### 1.2. Virtual DOM nằm ở vùng nhớ nào?

#### 📖 Giới thiệu kiến thức

Khi nói Virtual DOM "nằm trong bộ nhớ" (xem [câu 1](#q1)), "bộ nhớ" ở đây là **RAM (bộ nhớ chính)**, cụ
thể hơn là **vùng heap của JavaScript engine** (ở Chrome là **V8 heap**), nằm trong **tiến trình renderer**
của trình duyệt. Diễn giải theo từng lớp:

1. **RAM, không phải ổ đĩa:** Virtual DOM là dữ liệu **runtime** — sống trong RAM khi trang đang chạy,
   tắt tab là mất.
2. **JS Heap (managed heap):** Virtual DOM chỉ là **plain JavaScript object** (`{ tag, props, children }`).
   Mọi object JS đều được cấp phát trên **heap của engine (V8)** và do **Garbage Collector** quản lý —
   VDOM cũ sau khi diff xong, không còn tham chiếu → **bị GC thu hồi**.
3. **Trong renderer process:** trình duyệt đa tiến trình, mỗi tab là một **renderer process**; VDOM nằm
   trong vùng nhớ của process đó, cùng chỗ với toàn bộ biến/object JS của app.

**Điểm mấu chốt — đây chính là lý do VDOM "nhẹ":** khác biệt nằm ở **VDOM và Real DOM ở hai vùng nhớ
khác nhau**.

| | Nằm ở đâu | Bản chất |
|---|---|---|
| **Virtual DOM** | **JS heap** (V8) | Object JS thuần, **nhẹ** |
| **Real DOM** | Vùng nhớ **C++ của trình duyệt** (Blink/WebKit), **không** thuộc JS heap | Object C++ **nặng**, nhiều thuộc tính/method |

Khi JS code "đụng" vào Real DOM (`document.querySelector`, `el.appendChild`...), nó phải **vượt qua ranh
giới JS ↔ C++ binding** (qua Web IDL bindings) — thao tác này **tốn kém**, lại có thể kích hoạt
reflow/repaint. Ngược lại, tạo/sửa/**diff** Virtual DOM chỉ là **đọc-ghi object trong cùng JS heap** →
**không vượt ranh giới, không reflow** → rẻ hơn nhiều. Vì vậy framework "làm nháp" trên VDOM ở JS heap,
tính ra thay đổi tối thiểu, rồi **mới trả giá đắt** để chạm Real DOM một cách tối thiểu.

#### 🎯 Tóm tắt trả lời phỏng vấn

- "Bộ nhớ" chứa Virtual DOM = **RAM → vùng JS heap của engine (V8)**, trong **renderer process** của
  trình duyệt; VDOM chỉ là object JS nên do **Garbage Collector** quản lý.
- **Mấu chốt:** VDOM ở **JS heap**, còn Real DOM là **object C++ nằm ở vùng nhớ riêng của trình duyệt**
  (không thuộc JS heap) → thao tác Real DOM phải **vượt ranh giới JS↔C++ binding** (đắt, dễ reflow/repaint).
- Vì vậy diff/patch trên VDOM (**cùng JS heap, không vượt ranh giới**) rẻ hơn → framework tính toán trên
  VDOM trước rồi mới cập nhật Real DOM tối thiểu.

[↑ Mục lục](#mục-lục)

<a id="q1-3"></a>
### 1.3. Cách Vue 2 tối ưu diffing Virtual DOM

#### 📖 Giới thiệu kiến thức

Vue 2 dùng Virtual DOM dựa trên **snabbdom** (có chỉnh sửa) với thuật toán diff **runtime** — nghĩa là
so sánh **lúc chạy**, chưa có "gợi ý từ compiler" như Vue 3 ([câu 1.4](#q1-4)). Để việc so sánh không
quá đắt, Vue 2 dựa trên vài **heuristic (giả định thực dụng)** và cơ chế reactivity theo component.

**① Reactivity ở mức COMPONENT — thu hẹp phạm vi diff:** mỗi component Vue 2 có **một render watcher**.
Nhờ hệ thống reactivity (`Object.defineProperty` — getter/setter) theo dõi component nào **phụ thuộc**
dữ liệu nào, khi một reactive data đổi, **chỉ component dùng nó bị đánh dấu "dirty" và re-render** →
diffing **chỉ chạy trong phạm vi component đó**, không phải toàn app. Đây là tối ưu nền tảng nhất.

**② Thuật toán diff theo heuristic — hạ độ phức tạp về O(n):** so sánh cây tổng quát tốn **O(n³)** —
quá đắt. Vue 2 (như React) dùng 2 giả định để về **O(n)**:
- **Chỉ so node ĐỒNG CẤP (same level):** không cố match chéo tầng. Nếu node ở cùng vị trí **khác kiểu**
  (khác tag) → **thay thế nguyên cả subtree**, không đào sâu match.
- **`sameVnode`:** hai node coi là "cùng một node" (để patch tại chỗ, tái dùng DOM) khi **trùng `key` +
  trùng tag**; nếu không → tạo mới.

**③ `key` + thuật toán "so 2 đầu" (double-ended / 4 con trỏ) cho danh sách con:** khi patch children
(vd `v-for`), Vue 2 dùng 4 con trỏ **oldStart, oldEnd, newStart, newEnd** so từ **cả hai đầu vào giữa**
để **tái sử dụng tối đa** DOM node cũ (xử lý nhanh các trường hợp thêm/xóa/đảo đầu-cuối). **`key`** giúp
Vue nhận diện đúng node để di chuyển/tái dùng thay vì tạo lại.

```html
<!-- ✅ có key: Vue tái dùng đúng DOM node khi list đổi thứ tự -->
<li v-for="u in users" :key="u.id">{{ u.name }}</li>
```

> Không có `key` → Vue 2 dùng **"in-place patch"** (patch tại chỗ theo index): tái dùng DOM theo vị trí,
> **nhanh nhưng dễ sai** với list có state cục bộ (input, checkbox) hoặc animation. → Luôn đặt `key` ổn
> định cho `v-for` (đừng dùng index làm key nếu list thay đổi thứ tự).

**④ Static tree hoisting (`staticRenderFns`) — bỏ qua subtree tĩnh:** compiler Vue 2 **có** phân tích
tĩnh: đánh dấu (`markStatic`) các **subtree hoàn toàn tĩnh**, tạo **một lần** rồi **bỏ qua khi
re-render/diff**. Tuy nhiên đây là mức **"cả subtree tĩnh hay không"** (all-or-nothing), **thô hơn** Vue 3.

**Hạn chế (dẫn tới cải tiến ở Vue 3 — [câu 1.4](#q1-4)):** ngoài phần static bị bỏ qua, Vue 2 vẫn phải
**duyệt diff toàn bộ phần động từ trên xuống** và **so lại mọi thuộc tính** của mỗi node động (không
biết node đó "động ở chỗ nào"). Vue 3 khắc phục bằng **patch flags + block tree** để chỉ chạm đúng node
/thuộc tính động.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Vue 2 diff **runtime** (dựa snabbdom), tối ưu bằng **heuristic + reactivity theo component**, chưa có
  gợi ý compiler như Vue 3.
- **4 tối ưu:**
  - **Reactivity mức component:** data đổi → **chỉ component liên quan re-render** (thu hẹp phạm vi diff).
  - **Diff O(n):** chỉ so **node đồng cấp**; khác tag → thay cả subtree; `sameVnode` = trùng **key + tag**.
  - **`key` + so 2 đầu (4 con trỏ):** tái dùng tối đa DOM khi patch `v-for` (không key → in-place patch,
    dễ sai với list có state).
  - **Static hoisting (`staticRenderFns`):** subtree **hoàn toàn tĩnh** tạo 1 lần, bỏ qua diff (mức thô).
- **Hạn chế:** vẫn **duyệt cả cây động từ trên xuống** và so lại mọi prop → Vue 3 cải tiến bằng
  **patch flags + block tree** ([câu 1.4](#q1-4)).

[↑ Mục lục](#mục-lục)

<a id="q1-4"></a>
### 1.4. Cách Vue 3 tối ưu diffing Virtual DOM

#### 📖 Giới thiệu kiến thức

Với Virtual DOM "cổ điển" (và Vue 2), khi state đổi, framework phải **duyệt lại toàn bộ cây VDOM từ trên
xuống** để so sánh — kể cả những node **tĩnh** (không bao giờ đổi) → lãng phí. Vue 3 giải quyết bằng ý
tưởng cốt lõi: **Compiler-Informed Virtual DOM** — vì `template` của Vue có thể **phân tích tĩnh lúc
compile**, compiler **biết trước chỗ nào tĩnh, chỗ nào động** và nhúng "gợi ý" vào render function để
runtime **bỏ qua phần tĩnh, chỉ diff phần động**. (Đây là lợi thế so với JSX của React — vốn khó phân
tích tĩnh hơn.)

**4 tối ưu chính:**

**① Patch Flags — đánh dấu node động và "động ở đâu":** khi compile, mỗi VNode **động** được gắn một
**cờ số (patch flag)** cho biết **thứ gì có thể đổi** (chỉ text, chỉ class, chỉ props...). Lúc diff,
runtime **chỉ so đúng phần được đánh dấu**, bỏ qua mọi thứ tĩnh trong node đó.

```html
<span>{{ msg }}</span>
```
```js
// Compile ra (rút gọn) — cờ 1 = TEXT: chỉ text có thể đổi
createElementVNode("span", null, _ctx.msg, 1 /* TEXT */)
// → khi diff, Vue chỉ kiểm tra text, KHÔNG so props/class/attrs
```

**② Static Hoisting — "nâng" node tĩnh ra ngoài:** node (và cả props) **hoàn toàn tĩnh** được tạo **một
lần** bên ngoài render function rồi **tái sử dụng** qua mọi lần render → không tạo lại VNode, không tham
gia diff.

```js
const _hoisted_1 = createElementVNode("p", null, "Nội dung tĩnh"); // tạo 1 lần, dùng lại mãi
function render() {
  return createElementBlock("div", null, [
    _hoisted_1,                                   // node tĩnh — bỏ qua khi diff
    createElementVNode("span", null, _ctx.msg, 1 /* TEXT */), // node động
  ]);
}
```

**③ Block Tree (dynamicChildren) — tối ưu LỚN nhất:** compiler chia template thành các **"block"**; mỗi
block giữ **một mảng phẳng chỉ chứa các node động con cháu** (`dynamicChildren`). Khi update, Vue **không
duyệt cả cây** mà **đi thẳng qua mảng phẳng các node động** → biến việc diff cây (phụ thuộc **kích thước
template**) thành diff danh sách (phụ thuộc **số node động**) — thường ít hơn rất nhiều.

```
Cây VDOM:  div ─ header ─ nav ─ ...(tĩnh)      Vue 3 chỉ duyệt mảng phẳng:
            └─ span{{msg}} (động)      ──▶      dynamicChildren = [ span{{msg}}, button@click ]
            └─ button @click (động)             (bỏ qua toàn bộ node tĩnh, không đi cả cây)
```

**④ cacheHandlers — cache event handler:** handler inline (`@click="..."`) được **cache** để không tạo
hàm mới mỗi lần render → prop `onClick` xem như **tĩnh**, khỏi patch thừa.

```js
// onClick được cache: _cache[0] || (_cache[0] = ($event) => ...)
```

> **Kết quả:** Vue 3 khiến chi phí update **tỉ lệ với số node ĐỘNG**, gần như **độc lập với tổng kích
> thước template** — khác Vue 2 (chi phí tỉ lệ toàn cây). Đây là lý do Vue 3 nhanh hơn và tốn ít bộ nhớ
> hơn đáng kể. (Hướng đi xa hơn: **Vapor mode** bỏ hẳn VDOM — xem ghi chú ở [câu 1](#q1).)

#### 🎯 Tóm tắt trả lời phỏng vấn

- Ý tưởng gốc: **Compiler-Informed Virtual DOM** — Vue compile `template` (phân tích tĩnh được) nên
  **biết trước chỗ nào động/tĩnh** và nhúng gợi ý để runtime **chỉ diff phần động**.
- **4 tối ưu:**
  - **Patch Flags:** gắn cờ cho VNode động (TEXT/CLASS/PROPS...) → diff **chỉ so đúng phần đổi**.
  - **Static Hoisting:** node/props tĩnh tạo **1 lần**, tái dùng, **không tham gia diff**.
  - **Block Tree (`dynamicChildren`):** giữ **mảng phẳng các node động** → update **đi thẳng qua node
    động**, không duyệt cả cây (tối ưu lớn nhất).
  - **cacheHandlers:** cache handler inline → khỏi patch `onClick` thừa.
- **Hệ quả:** chi phí update **tỉ lệ số node động** (gần như độc lập kích thước template) → nhanh & nhẹ
  hơn Vue 2 (vốn diff toàn cây từ trên xuống).

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Phân biệt v-if và v-show

### 📖 Giới thiệu kiến thức

Cả hai là **directive điều khiển việc hiển thị có điều kiện** một phần tử trong template — nhìn bề ngoài
giống nhau (ẩn/hiện theo một biểu thức boolean), nhưng **cơ chế bên dưới hoàn toàn khác**, gắn trực tiếp
với cách Vue render ra DOM thật (xem [câu 1](#q1)).

**① `v-if` — render có điều kiện thật sự (thêm/xóa khỏi DOM):**
Khi biểu thức **`false`**, phần tử **không được render** — **không tồn tại trong DOM thật** (và cả Virtual
DOM). Khi chuyển sang **`true`**, Vue **tạo mới** phần tử/component và chèn vào; khi về `false`, Vue **hủy
(destroy)** nó khỏi DOM. Vì vậy:
- **Kích hoạt lifecycle** (xem [câu 3](#q3)): bật lên → `mounted`; tắt đi → `unmounted`. Component con
  bị **hủy và dựng lại** mỗi lần toggle (state bên trong **mất**, chạy lại logic khởi tạo).
- **"Lazy":** nếu ban đầu `false` thì **không render gì cả** cho tới khi thành `true` → tiết kiệm nếu điều
  kiện hiếm khi đúng.
- Hỗ trợ **`v-else-if` / `v-else`** và đặt trên **`<template>`** để nhóm nhiều phần tử.

**② `v-show` — luôn render, chỉ bật/tắt CSS `display`:**
Phần tử **LUÔN được render vào DOM** bất kể điều kiện; Vue chỉ **thay đổi CSS `display: none`** để ẩn/hiện.
Phần tử **luôn nằm trong DOM**, không bị tạo/hủy → **không kích hoạt lifecycle**, state con **được giữ
nguyên**. **Không** dùng được với `<template>` và **không** kết hợp `v-else`.

**Điểm mấu chốt — chi phí (initial vs toggle):**
- **`v-if`:** chi phí **toggle cao** (thêm/xóa DOM + tạo/hủy component + lifecycle), nhưng chi phí **khởi
  tạo thấp** nếu điều kiện ban đầu `false` (không render gì).
- **`v-show`:** chi phí **khởi tạo cao hơn** (luôn render dù đang ẩn), nhưng **toggle rất rẻ** (chỉ đổi CSS).

**Bảng so sánh:**

| Tiêu chí | **`v-if`** | **`v-show`** |
|---|---|---|
| **Cơ chế** | **Thêm/xóa** phần tử khỏi DOM (render thật) | **Luôn render**, chỉ bật/tắt CSS `display` |
| **Khi `false`** | **Không có** trong DOM | Có trong DOM nhưng `display:none` |
| **Lifecycle** | Có (mount/unmount mỗi lần toggle) | **Không** (chỉ tạo 1 lần) |
| **State component con khi toggle** | **Mất** (bị hủy & dựng lại) | **Giữ nguyên** |
| **Chi phí khởi tạo** | **Thấp** (nếu ban đầu `false`) | Cao hơn (luôn render) |
| **Chi phí toggle** | **Cao** | **Rất thấp** |
| **`v-else` / `v-else-if`** | ✅ Có | ❌ Không |
| **Dùng trên `<template>`** | ✅ Được | ❌ Không |
| **Hợp khi** | Điều kiện **ít đổi**, hoặc nội dung nặng cần tránh render | Bật/tắt **thường xuyên** (tab, dropdown, tooltip) |

```html
<!-- v-if: khi isAdmin = false, khối này KHÔNG có trong DOM -->
<div v-if="isAdmin">Bảng quản trị</div>
<div v-else>Không có quyền</div>

<!-- v-show: luôn nằm trong DOM, chỉ đổi display khi toggle -->
<div v-show="isOpen">Nội dung dropdown</div>

<!-- v-if nhóm nhiều phần tử bằng <template> (v-show không làm được) -->
<template v-if="loaded">
  <h1>Tiêu đề</h1>
  <p>Nội dung</p>
</template>
```

> **Chọn cái nào?** Nguyên tắc: **toggle nhiều → `v-show`** (rẻ vì chỉ đổi CSS), **ít đổi / nội dung nặng
> → `v-if`** (tránh render khi không cần). Ngoài ra khi cần **thật sự không mount** component con (vd
> không muốn nó gọi API / chạy logic khi ẩn) thì phải dùng **`v-if`** — `v-show` vẫn render và chạy hết.

> **Lưu ý phụ (hay hỏi):** **không đặt `v-if` và `v-for` trên cùng một phần tử**. Ở **Vue 3**, `v-if` có
> **độ ưu tiên cao hơn** `v-for` (nên `v-if` không truy cập được biến của vòng lặp); ở **Vue 2** thì
> ngược lại. → Nên tách ra (dùng `<template v-for>` bọc ngoài, `v-if` bên trong) hoặc lọc bằng `computed`.

### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai đều **hiển thị có điều kiện**, nhưng khác cơ chế: **`v-if` thêm/xóa phần tử khỏi DOM** (render
  thật); **`v-show` luôn render, chỉ bật/tắt CSS `display`**.
- **`v-if`:** khi `false` thì **không tồn tại trong DOM**; toggle sẽ **tạo/hủy** phần tử → **kích hoạt
  lifecycle** (mount/unmount — [câu 3](#q3)), state con **mất**. Là **"lazy"** (ban đầu `false` → không
  render). Hỗ trợ **`v-else`/`v-else-if`** và `<template>`.
- **`v-show`:** phần tử **luôn trong DOM**, chỉ ẩn bằng `display:none` → **không** lifecycle, state **giữ
  nguyên**; không dùng được `v-else`/`<template>`.
- **Chi phí:** `v-if` **toggle đắt** nhưng khởi tạo rẻ (nếu ẩn); `v-show` khởi tạo đắt hơn nhưng **toggle
  rất rẻ**.
- **Chọn:** bật/tắt **thường xuyên** (tab, dropdown) → **`v-show`**; điều kiện **ít đổi** / nội dung nặng
  / cần **không mount** component con → **`v-if`**.

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Vòng đời của Vue.js (lifecycle hooks)

### 📖 Giới thiệu kiến thức

Mỗi component Vue trải qua một **vòng đời (lifecycle)**: được **khởi tạo → gắn vào DOM → cập nhật khi
dữ liệu đổi → gỡ khỏi DOM**. Ở mỗi mốc, Vue gọi các **lifecycle hook** — hàm cho phép ta "chen" code
vào đúng thời điểm (gọi API, thao tác DOM, dọn dẹp tài nguyên...). Vòng đời gắn chặt với Virtual DOM
(xem [câu 1](#q1)): *mount* = lần đầu patch VDOM lên Real DOM, *update* = re-render + patch khi state
đổi, *unmount* = teardown.

**4 giai đoạn & các hook (Options API — Vue 3):**

```
① Creation   : beforeCreate → created
② Mounting   : beforeMount   → mounted
③ Updating   : beforeUpdate  → updated      (lặp lại mỗi khi reactive state đổi)
④ Unmounting : beforeUnmount → unmounted
```

**① Creation — khởi tạo:**
- `beforeCreate`: chạy sớm nhất, **chưa có** reactive data (`data`, `computed`, `methods` chưa sẵn sàng).
- `created`: **reactive data & methods đã sẵn sàng**, nhưng **DOM chưa render** (`this.$el` chưa có). Hợp
  để **gọi API lấy dữ liệu**, khởi tạo logic không cần DOM. **Chạy được cả trên server (SSR).**

**② Mounting — gắn vào DOM:**
- `beforeMount`: ngay trước khi chèn DOM thật.
- `mounted`: **DOM đã render xong**, truy cập được `this.$el` / **template ref**. Hợp để **thao tác DOM**,
  tích hợp thư viện bên thứ ba (chart, map...), hoặc gọi API cần DOM. **KHÔNG chạy khi SSR** (chỉ client).

**③ Updating — cập nhật (lặp lại):** kích hoạt **mỗi khi reactive state thay đổi** → Vue re-render.
- `beforeUpdate`: sau khi state đổi, **trước khi** patch lên DOM (DOM còn giá trị cũ).
- `updated`: **sau khi** DOM đã được patch. ⚠️ **Không nên mutate state trong `updated`** → dễ gây
  **vòng lặp cập nhật vô hạn**.

**④ Unmounting — gỡ bỏ:**
- `beforeUnmount`: component sắp bị hủy, mọi thứ còn hoạt động.
- `unmounted`: đã gỡ khỏi DOM. Nơi **dọn dẹp**: gỡ event listener, clear timer/interval, `unsubscribe`
  (vd RxJS), hủy kết nối → **tránh memory leak**.

**Các hook đặc biệt khác:** `activated` / `deactivated` (component trong `<KeepAlive>`), `errorCaptured`
(bắt lỗi từ component con), `renderTracked` / `renderTriggered` (debug reactivity — chỉ dev).

**Composition API — tương đương trong `setup()`:** import các hàm `onXxx`. Lưu ý **không có**
`onBeforeCreate`/`onCreated` — vì bản thân `setup()` **chạy trước `beforeCreate`**, code khởi tạo đặt
thẳng trong `setup()`.

| Giai đoạn | **Options API** | **Composition API** (`setup`) |
|---|---|---|
| Khởi tạo | `beforeCreate`, `created` | *(viết thẳng trong `setup()`)* |
| Mount | `beforeMount` / `mounted` | `onBeforeMount` / `onMounted` |
| Update | `beforeUpdate` / `updated` | `onBeforeUpdate` / `onUpdated` |
| Unmount | `beforeUnmount` / `unmounted` | `onBeforeUnmount` / `onUnmounted` |
| KeepAlive | `activated` / `deactivated` | `onActivated` / `onDeactivated` |
| Lỗi | `errorCaptured` | `onErrorCaptured` |

```js
// Options API
export default {
  data: () => ({ users: [] }),
  created() {                       // data sẵn sàng, chưa có DOM → gọi API
    fetch("/api/users").then(r => r.json()).then(d => (this.users = d));
  },
  mounted() {                       // DOM đã có → thao tác DOM / thư viện ngoài
    this.chart = new Chart(this.$refs.canvas, {/* ... */});
  },
  beforeUnmount() {                 // dọn dẹp tránh memory leak
    this.chart.destroy();
  },
};
```

```js
// Composition API (<script setup>)
import { ref, onMounted, onUnmounted } from "vue";

const users = ref([]);
let timer;
// code ở đây tương đương "created" (chạy trong setup, trước beforeCreate)
fetch("/api/users").then(r => r.json()).then(d => (users.value = d));

onMounted(() => { timer = setInterval(poll, 1000); }); // DOM sẵn sàng
onUnmounted(() => clearInterval(timer));               // dọn dẹp
```

> **Vue 2 → Vue 3 (hay bị hỏi):** hai hook cuối **đổi tên**: `beforeDestroy` → **`beforeUnmount`**,
> `destroyed` → **`unmounted`**. Các hook khác giữ nguyên tên (Options API).

> **`created` vs `mounted` — khi nào gọi API?** Cả hai đều được; nếu **không cần DOM** thì `created`
> (sớm hơn, chạy cả SSR); nếu cần **truy cập DOM/ref** thì `mounted`. Thực tế nhiều người mặc định
> `mounted` cho gọn, nhưng nhớ `mounted` **không chạy khi SSR**.

### 🎯 Tóm tắt trả lời phỏng vấn

- Component Vue có **4 giai đoạn**, mỗi giai đoạn 1 cặp hook **before/after**:
  **① Creation** (`beforeCreate` → `created`) → **② Mounting** (`beforeMount` → `mounted`) →
  **③ Updating** (`beforeUpdate` → `updated`, lặp mỗi khi state đổi) → **④ Unmounting**
  (`beforeUnmount` → `unmounted`).
- **`created`:** reactive data sẵn sàng, **chưa có DOM** → gọi API / khởi tạo (chạy cả **SSR**).
  **`mounted`:** **DOM đã render** → thao tác DOM, template ref, thư viện ngoài (**không** chạy SSR).
- **`updated`:** DOM đã patch sau khi state đổi — **đừng mutate state ở đây** (vòng lặp vô hạn).
  **`beforeUnmount`/`unmounted`:** **dọn dẹp** (gỡ listener, clear timer, unsubscribe) → tránh memory leak.
- **Composition API:** dùng `onMounted`, `onUpdated`, `onUnmounted`... trong `setup()`; **không có**
  `onCreated`/`onBeforeCreate` vì `setup()` chạy trước — viết logic khởi tạo thẳng trong `setup()`.
- **Vue 2 → Vue 3:** `beforeDestroy` → **`beforeUnmount`**, `destroyed` → **`unmounted`**.
- Hook đặc biệt: `activated`/`deactivated` (`<KeepAlive>`), `errorCaptured`.

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Phân biệt watch, computed và methods

### 📖 Giới thiệu kiến thức

Đây là bộ ba công cụ Vue cung cấp để **xử lý logic/dữ liệu trong component**, rất hay bị hỏi vì dễ nhầm.
Chúng đều dựa trên **hệ reactivity** của Vue (xem [câu 1](#q1)) nhưng phục vụ **ba mục đích khác nhau**.
Cách phân biệt nhanh: **`computed` để TÍNH ra giá trị dẫn xuất (có cache)**, **`methods` để CHẠY hành
động khi được gọi**, **`watch` để PHẢN ỨNG (side-effect) khi dữ liệu đổi**.

**① `methods` — hàm/hành động:**
Chỉ là **hàm bình thường** gắn vào component. Chạy **mỗi khi được gọi** (từ event `@click`, hoặc gọi
trong template). **KHÔNG cache** — gọi 10 lần thì chạy 10 lần; nếu dùng trong template, **mỗi lần
component re-render** (bất kỳ reactive nào đổi) là **chạy lại**, kể cả khi dữ liệu liên quan không đổi.
→ Hợp cho **event handler** và **hành động cần chủ động gọi** (submit form, mở modal...).

**② `computed` — giá trị dẫn xuất, có cache:**
Là **thuộc tính tính toán** từ các reactive khác. Đặc điểm mấu chốt: **cache theo dependency** — Vue theo
dõi computed phụ thuộc reactive nào; **chỉ tính lại khi dependency đó thay đổi**, còn không thì **trả kết
quả đã cache** (dù được đọc nhiều lần). Phải **thuần (pure)**: **đồng bộ, không side-effect, luôn `return`
một giá trị**. → Hợp cho **dữ liệu dẫn xuất để hiển thị** (lọc/định dạng/tính tổng danh sách...).

**③ `watch` — theo dõi & phản ứng bằng side-effect:**
"Quan sát" một hoặc nhiều **reactive source**; **mỗi khi nó đổi**, chạy một **callback side-effect**.
Không trả giá trị cho template — mục đích là **làm gì đó để phản ứng** với thay đổi: **gọi API, thao tác
async, ghi localStorage, đồng bộ state khác...**. Callback nhận **giá trị mới & cũ**, và **được phép
async**. → Hợp khi cần **side-effect / async theo sau một thay đổi**.

**So sánh cốt lõi:**

| Tiêu chí | **methods** | **computed** | **watch** |
|---|---|---|---|
| **Mục đích** | Chạy **hành động** khi gọi | **Tính** giá trị dẫn xuất | **Phản ứng** khi dữ liệu đổi |
| **Cache** | ❌ Không | ✅ **Có** (theo dependency) | — (không trả giá trị) |
| **Trả về giá trị** | Có (khi gọi) | ✅ Có (dùng như property) | ❌ Không (chạy side-effect) |
| **Khi nào chạy** | Mỗi lần **được gọi**/re-render | Khi **dependency đổi** (lazy) | Khi **nguồn theo dõi đổi** |
| **Side-effect** | Được | ❌ **Không nên** (phải pure) | ✅ **Đúng mục đích** |
| **Async** | Được | ❌ Không (phải sync) | ✅ **Được** |
| **Dùng khi** | Event handler, hành động chủ động | Dữ liệu dẫn xuất để hiển thị | Gọi API / async / đồng bộ theo thay đổi |

```js
// Options API — cùng một bài toán, 3 cách khác nhau
export default {
  data: () => ({ firstName: "Anh", lastName: "Nguyen", keyword: "" }),

  computed: {
    // ✅ dẫn xuất + CACHE: chỉ tính lại khi firstName/lastName đổi
    fullName() { return `${this.firstName} ${this.lastName}`; },
  },

  methods: {
    // hành động: chạy MỖI khi gọi (không cache)
    greet() { alert(`Hello ${this.fullName}`); }, // gọi từ @click
  },

  watch: {
    // side-effect + async: keyword đổi → gọi API (debounce/async tùy ý)
    async keyword(newVal, oldVal) {
      const res = await fetch(`/api/search?q=${newVal}`);
      this.results = await res.json();
    },
  },
};
```

**`computed` vs `methods` (câu hỏi kinh điển):** nếu chỉ để **hiển thị giá trị dẫn xuất**, dùng
`computed` **hiệu quả hơn** nhờ cache — gọi `fullName` 3 lần trong template chỉ tính **1 lần**; nếu là
`method` sẽ tính **3 lần**, và **mỗi lần re-render lại tính tiếp**. Chỉ dùng `method` (thay computed) khi
**không muốn cache** (vd cần giá trị "tươi" như `Date.now()`) hoặc khi cần **truyền tham số**.

**`computed` vs `watch` (câu hỏi kinh điển):** ưu tiên **`computed`** khi **kết quả là một giá trị dẫn
xuất** từ dữ liệu khác (khai báo, ngắn gọn). Chỉ dùng **`watch`** khi cần **side-effect** (gọi API,
async) hoặc **phản ứng** mà không phải "tính ra giá trị". Nhiều trường hợp người mới lạm dụng `watch` để
gán biến — thực ra nên là `computed`.

**Trong Composition API** (xem [câu 6](#q6)) — tương đương dạng hàm:

```js
import { ref, computed, watch, watchEffect } from "vue";
const first = ref("Anh"), last = ref("Nguyen");

const fullName = computed(() => `${first.value} ${last.value}`); // computed
function greet() { alert(fullName.value); }                       // "method" = hàm thường
watch(first, (n, o) => console.log(n, o));                        // watch nguồn cụ thể
watchEffect(() => console.log(fullName.value));                   // tự dò dependency, chạy ngay
```

> **`watch` vs `watchEffect`:** `watch` theo dõi **nguồn chỉ định** (lazy — chỉ chạy khi nguồn đổi, có
> new/old value); `watchEffect` **tự động dò** mọi reactive dùng trong callback và **chạy ngay lần đầu**
> (không có old value). Dùng `watch` khi cần biết giá trị cũ hoặc kiểm soát nguồn; `watchEffect` khi chỉ
> cần "đụng gì theo nấy".

### 🎯 Tóm tắt trả lời phỏng vấn

- **3 mục đích khác nhau:** `computed` = **tính giá trị dẫn xuất (có cache)**; `methods` = **chạy hành
  động khi được gọi**; `watch` = **phản ứng bằng side-effect khi dữ liệu đổi**.
- **`computed`:** cache theo **dependency** → chỉ tính lại khi dep đổi; phải **thuần (sync, không
  side-effect, luôn return)**. Dùng cho **dữ liệu hiển thị dẫn xuất**.
- **`methods`:** **không cache** → chạy mỗi lần gọi / mỗi lần re-render nếu dùng trong template. Dùng cho
  **event handler / hành động chủ động**, hoặc khi cần **tham số** / không muốn cache.
- **`watch`:** theo dõi reactive source, chạy callback (có **new/old value**, **được async**) → dùng cho
  **gọi API / async / đồng bộ state** theo sau thay đổi. **Không trả giá trị** cho template.
- **Chọn:** hiển thị giá trị dẫn xuất → **computed** (hơn method nhờ cache, hơn watch vì khai báo gọn);
  cần side-effect/async → **watch**; hành động do user kích hoạt → **methods**.

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Phân biệt slot và scoped slot

### 📖 Giới thiệu kiến thức

**Slot là gì (nền tảng):** bình thường component nhận dữ liệu từ cha qua **props**, nhưng props chỉ hợp
truyền **giá trị**. Khi muốn cha truyền **cả một đoạn template/HTML** vào bên trong component con, ta dùng
**slot** — cơ chế **phân phối nội dung (content distribution / content projection)**. Nhờ slot, component
con trở thành **"khung" tái sử dụng** (card, modal, layout...) còn cha **quyết định nội dung** đổ vào.

**① Slot thường (default / named slot) — cha rót nội dung, con chỉ "chừa chỗ":**
Con khai báo `<slot>` làm **chỗ giữ chỗ (placeholder)**; cha đặt nội dung vào, nội dung đó được render tại
vị trí `<slot>`. Điểm mấu chốt: **nội dung slot được biên dịch trong phạm vi (scope) của CHA** → chỉ
**truy cập được dữ liệu của cha**, **không thấy** dữ liệu bên trong con.

```html
<!-- Con: BaseCard.vue -->
<div class="card">
  <slot></slot>                      <!-- chỗ giữ chỗ cho nội dung cha -->
  <slot name="footer"></slot>        <!-- named slot -->
</div>

<!-- Cha dùng: rót nội dung (chỉ dùng được data của cha) -->
<BaseCard>
  <p>{{ userOfParent }}</p>          <!-- ✅ data của cha -->
  <template #footer><button>OK</button></template>
</BaseCard>
```

**② Scoped slot — con "truyền ngược" dữ liệu ra cho cha dùng:**
Vấn đề: nếu **dữ liệu cần hiển thị nằm TRONG con** (vd component `<UserList>` tự fetch danh sách user),
mà cha lại muốn **tự quyết định cách render mỗi item**, thì slot thường **không đủ** — vì nội dung slot
chỉ thấy scope cha. **Scoped slot** giải quyết: **con "phơi" (expose) dữ liệu qua `<slot :data="...">`**,
cha nhận lại qua **`v-slot="slotProps"`** rồi tự render. Nói ngắn gọn: **scoped slot = slot có truyền
tham số từ con ra cha** → cha vừa kiểm soát giao diện, vừa dùng được dữ liệu của con.

```html
<!-- Con: UserList.vue — có data (users) và "phơi" từng user ra slot -->
<ul>
  <li v-for="u in users" :key="u.id">
    <slot :user="u" :index="u.id"></slot>   <!-- bind data ra ngoài -->
  </li>
</ul>

<!-- Cha: nhận data của con qua v-slot rồi tự quyết định cách render -->
<UserList v-slot="{ user }">
  <strong>{{ user.name }}</strong> — {{ user.email }}   <!-- ✅ dùng data của CON -->
</UserList>
```

**Khác biệt cốt lõi — SCOPE (phạm vi truy cập dữ liệu):**

| Tiêu chí | **Slot thường** | **Scoped slot** |
|---|---|---|
| **Hướng dữ liệu** | Cha → con (rót nội dung vào) | **Con → cha** (con phơi data ra cho cha) |
| **Nội dung slot thấy data của** | **Chỉ CHA** | Cha **+ data con truyền qua slot props** |
| **Cú pháp con** | `<slot></slot>` | `<slot :user="u"></slot>` (bind prop) |
| **Cú pháp cha** | Đặt nội dung trực tiếp | `v-slot="{ user }"` để nhận slot props |
| **Mục đích** | Tùy biến **nội dung tĩnh** của khung | Con quản data, cha tùy biến **cách render** data đó |
| **Ví dụ điển hình** | Card, Modal, Layout | `<Table>`, `<UserList>`, renderless component |

**Vì sao scoped slot mạnh:** nó tách **"lấy dữ liệu/logic" (do con lo)** khỏi **"cách hiển thị" (do cha
quyết)** → tạo được **renderless component** (component chỉ lo logic, giao toàn bộ UI cho cha qua scoped
slot). Đây là một pattern tái sử dụng logic mạnh, họ hàng gần với **composable** ở [câu 6.1](#q6-1) (cả
hai đều tách logic khỏi UI, nhưng scoped slot tách qua template còn composable tách qua hàm).

> **Cú pháp Vue 3 (thống nhất `v-slot`):** named slot dùng `v-slot:footer` (rút gọn **`#footer`**);
> scoped slot dùng `v-slot="slotProps"` hoặc `v-slot:default="{ user }"`. (Vue 2 cũ dùng `slot="..."` và
> `slot-scope="..."` — đã bị thay bằng `v-slot` từ Vue 2.6+.)

### 🎯 Tóm tắt trả lời phỏng vấn

- **Slot** = cơ chế **phân phối nội dung**: cha truyền cả **đoạn template** vào component con (con là
  "khung" tái sử dụng, cha quyết nội dung) — khác props chỉ truyền **giá trị**.
- **Slot thường:** con `<slot>` chừa chỗ, cha rót nội dung vào. Nội dung slot **chỉ truy cập được data
  của CHA**, không thấy data con.
- **Scoped slot:** con **phơi data ra** qua `<slot :user="u">`, cha nhận qua **`v-slot="{ user }"`** rồi
  **tự render** → dùng được **data của CON**. Bản chất: **slot có truyền tham số từ con ra cha**.
- **Khác biệt cốt lõi = scope:** slot thường dữ liệu đi **cha→con**; scoped slot cho **con→cha** (con
  quản dữ liệu/logic, cha quyết cách hiển thị) → nền tảng của **renderless component**.
- **Cú pháp Vue 3:** `#footer` (named), `v-slot="slotProps"` (scoped). Dùng cho `<Table>`, `<List>`, layout tái sử dụng.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Composition API là gì?

### 📖 Giới thiệu kiến thức

**Composition API** là **cách viết logic component mới của Vue 3** (cũng backport cho Vue 2.7), dùng
một bộ **hàm** (`ref`, `reactive`, `computed`, `watch`, `onMounted`...) để tổ chức code, thay cho cách
khai báo theo **Options API** cũ.

**Vấn đề của Options API (bối cảnh ra đời):** Options API tổ chức code theo **"loại option"** — `data`,
`methods`, `computed`, `watch`, lifecycle... Với component lớn, logic của **một tính năng** bị **xé nhỏ
rải rác** qua nhiều option (state ở `data`, xử lý ở `methods`, phụ thuộc ở `computed`...) → khó đọc, khó
theo dõi. Việc **tái sử dụng logic** giữa các component phải dùng **mixin** — vốn có nhược điểm: **xung
đột tên**, **không rõ property đến từ đâu**, quan hệ ngầm.

**Composition API giải quyết:** cho phép **gom toàn bộ state + logic của cùng một tính năng vào một
chỗ** (theo **logic concern**, không theo loại option), viết trong **`setup()`** hoặc cú pháp gọn
**`<script setup>`**. Nhờ vậy dễ đọc, và **tách logic ra hàm tái dùng** (**composable** — xem
[câu 6.1](#q6-1)).

**Các khối cơ bản (reactivity):**
- **`ref(value)`** — bọc **một giá trị** (primitive hoặc object) thành reactive; truy cập/gán qua
  **`.value`** trong script (template tự "mở" `.value`).
- **`reactive(obj)`** — làm **object** reactive (dựa **Proxy** — xem [câu 1](#q1) về Vue 3 reactivity);
  **không** cần `.value`, nhưng **không nhận primitive** và **mất reactivity khi destructure**.
- **`computed`** — giá trị **dẫn xuất có cache**. **`watch` / `watchEffect`** — chạy side-effect khi
  dữ liệu đổi. **Lifecycle** dạng hàm: `onMounted`, `onUnmounted`... (xem [câu 3](#q3)).

**`setup()` chạy trước `beforeCreate`** và **không có `this`** — làm việc trực tiếp với biến/hàm.

```vue
<script setup>
import { ref, computed, onMounted } from "vue";

const count = ref(0);                          // state
const double = computed(() => count.value * 2); // dẫn xuất, có cache
function inc() { count.value++; }               // hành vi
onMounted(() => console.log("mounted"));        // lifecycle

// <script setup>: khỏi cần return — biến/hàm tự lộ ra template
</script>

<template>
  <button @click="inc">{{ count }} → {{ double }}</button>
</template>
```

**Options API vs Composition API:**

| Tiêu chí | **Options API** | **Composition API** |
|---|---|---|
| **Tổ chức code** | Theo **loại** (`data`/`methods`/`computed`) | Theo **tính năng** (gom cùng chỗ) |
| **Tái sử dụng logic** | **Mixin** (dễ xung đột, mơ hồ) | **Composable** (rõ ràng, không xung đột) |
| **`this`** | Có (trỏ instance) | **Không** (dùng biến trực tiếp) |
| **TypeScript** | Suy luận kiểu hạn chế | **Hỗ trợ type tốt hơn** |
| **Component lớn** | Dễ phân mảnh, khó đọc | Gọn, dễ theo dõi |
| **Hợp với** | Component **nhỏ/đơn giản**, người mới | Component **phức tạp**, cần tái dùng logic |

> **Không loại trừ nhau:** Options API vẫn dùng tốt cho component đơn giản; Composition API mạnh khi
> component phức tạp hoặc cần **trích xuất logic tái dùng**. Composition API cũng **tree-shaking** tốt
> hơn (chỉ import hàm cần dùng).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Composition API** = cách tổ chức logic component của Vue 3 bằng **các hàm** (`ref`, `reactive`,
  `computed`, `watch`, `onMounted`...), viết trong **`setup()`** / **`<script setup>`** — thay cho
  **Options API** (khai báo theo `data`/`methods`/`computed`).
- **Vì sao có:** Options API khiến logic một tính năng **rải rác** qua nhiều option ở component lớn, và
  tái dùng logic bằng **mixin** thì dễ **xung đột/mơ hồ**. Composition API **gom logic theo tính năng**
  và cho **trích xuất thành composable**.
- **Khối cơ bản:** `ref` (giá trị, dùng `.value`), `reactive` (object, Proxy, không `.value`),
  `computed`, `watch/watchEffect`, lifecycle dạng hàm. `setup()` chạy **trước `beforeCreate`**, **không
  có `this`**.
- **Chọn:** component nhỏ → Options API vẫn ổn; component phức tạp / cần tái dùng logic / TypeScript →
  Composition API. **Hai cách không loại trừ nhau.**

[↑ Mục lục](#mục-lục)

<a id="q6-1"></a>
### 6.1. Composables trong Vue 3

#### 📖 Giới thiệu kiến thức

**Composable** là một **hàm tận dụng Composition API** (xem [câu 6](#q6)) để **đóng gói và tái sử dụng
logic CÓ STATE (stateful logic)**. Quy ước: tên bắt đầu bằng **`use`** (`useMouse`, `useFetch`,
`useCounter`). Đây là **cách "chính thống" thay cho mixin** để chia sẻ logic giữa các component.

**Khác gì một hàm helper thường?** Helper thuần chỉ xử lý dữ liệu (vô trạng thái). **Composable quản lý
reactive state** (`ref`/`reactive`) và có thể **hook vào lifecycle** (`onMounted`/`onUnmounted`), tận
dụng hệ reactivity của Vue — nên state trả về vẫn "sống" và tự cập nhật.

```js
// composables/useMouse.js — đóng gói logic theo dõi vị trí chuột
import { ref, onMounted, onUnmounted } from "vue";

export function useMouse() {
  const x = ref(0);
  const y = ref(0);
  function update(e) { x.value = e.pageX; y.value = e.pageY; }

  onMounted(() => window.addEventListener("mousemove", update));
  onUnmounted(() => window.removeEventListener("mousemove", update)); // tự dọn dẹp → tránh leak

  return { x, y }; // trả về ref để nơi dùng giữ được reactivity
}
```

```vue
<script setup>
import { useMouse } from "@/composables/useMouse";
const { x, y } = useMouse();   // dùng như 1 dòng; có thể gọi nhiều composable
</script>

<template>Chuột ở: {{ x }}, {{ y }}</template>
```

**Đặc điểm quan trọng:**
- **Mỗi lần gọi tạo state ĐỘC LẬP:** gọi `useMouse()` ở 2 component (hoặc 2 lần) → **2 state riêng biệt**
  (khác với **store** như Pinia — state **dùng chung**). Composable = tái dùng *logic*, không phải chia sẻ *state*.
- **Compose được:** một composable có thể **gọi composable khác** để ghép logic.
- **Quy ước:** trả về **`ref`** (để `destructure` không mất reactivity); nhận tham số có thể là `ref`
  (dùng `toValue`/`unref` để đọc). Chỉ gọi ở **`setup`/`<script setup>`** (vì đăng ký lifecycle).

**Composable vs Mixin (điểm hay hỏi):**

| Tiêu chí | **Mixin** (Options API) | **Composable** (Composition API) |
|---|---|---|
| **Nguồn gốc property** | Ẩn — không rõ từ mixin nào | **Rõ ràng** (biết từ hàm nào, biến do mình đặt) |
| **Xung đột tên** | Dễ xảy ra (trùng `data`/`methods`) | **Không** (tự đặt tên khi destructure) |
| **Ghép nhiều nguồn** | Khó lần vết | Dễ (gọi nhiều composable) |
| **TypeScript** | Kém | Tốt |

> Thực tế có thư viện **VueUse** cung cấp sẵn hàng trăm composable (`useLocalStorage`, `useDebounce`,
> `useFetch`...). Composable là **lý do lớn khiến Composition API ra đời**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Composable** = **hàm dùng Composition API để đóng gói & tái sử dụng logic có state**; quy ước tên
  **`use...`** (`useMouse`, `useFetch`). Là **cách thay mixin** để chia sẻ logic.
- Khác helper thường: composable **quản lý reactive state** (`ref`/`reactive`) và **hook lifecycle**
  (`onMounted`/`onUnmounted` tự dọn dẹp) → state trả về vẫn reactive.
- **Mỗi lần gọi = state độc lập** (khác store dùng chung); **compose được** (gọi composable trong
  composable); nên **trả về `ref`** để destructure không mất reactivity.
- **Hơn mixin:** rõ nguồn gốc, **không xung đột tên**, ghép nhiều nguồn dễ, TypeScript tốt. Thư viện phổ
  biến: **VueUse**.

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. Vuex là gì?

### 📖 Giới thiệu kiến thức

**Bối cảnh — vì sao cần Vuex:** trong Vue, dữ liệu (state) mặc định thuộc **từng component** và truyền
đi theo quan hệ cha–con: cha → con qua **props**, con → cha qua **emit event**. Với app nhỏ thì ổn, nhưng
khi app lớn lên, **nhiều component ở xa nhau cần dùng chung một state** (vd: thông tin user đăng nhập,
giỏ hàng, theme) → xuất hiện hai vấn đề:
- **Prop drilling:** phải "khoan" props qua hàng loạt component trung gian không hề dùng tới nó.
- **Event bus / chia sẻ lộn xộn:** state bị sửa ở nhiều nơi, **khó lần vết ai đổi cái gì**, dễ sinh bug.

**Vuex là gì:** là **thư viện quản lý state tập trung (centralized state management) chính thức** của
Vue, đồng thời là một **pattern** (lấy cảm hứng từ **Flux/Redux**). Ý tưởng cốt lõi: gom toàn bộ state
dùng chung vào **MỘT store tập trung** — **"single source of truth"** (một nguồn sự thật duy nhất) — và
bắt mọi thay đổi phải đi theo **luồng một chiều (unidirectional data flow)** có quy tắc, để **dễ theo dõi
và debug**. Store của Vuex cũng **reactive** (dựa hệ reactivity của Vue — xem [câu 1](#q1)): state đổi →
mọi component đang dùng **tự re-render**.

**5 khái niệm cốt lõi:**

| Khái niệm | Vai trò | Ghi chú |
|---|---|---|
| **State** | Nơi chứa dữ liệu dùng chung (single source of truth) | Truy cập qua `this.$store.state` |
| **Getters** | State **dẫn xuất, có cache** — như `computed` cho store (xem [câu 6](#q6)) | Vd lọc/đếm danh sách |
| **Mutations** | **Cách DUY NHẤT** để thay đổi state; **BẮT BUỘC đồng bộ (sync)** | Gọi bằng `commit('name', payload)` |
| **Actions** | Chứa logic **bất đồng bộ (async)** (gọi API...) rồi **commit** mutation | Gọi bằng `dispatch('name', payload)` |
| **Modules** | Chia store lớn thành nhiều **module con** (mỗi module có state/getters/... riêng) | Tránh 1 store phình to |

**Luồng dữ liệu một chiều (unidirectional) — điểm mấu chốt:**

```
Component ─dispatch─▶ Actions ─commit─▶ Mutations ─(sync)─▶ State ─reactive─▶ Component (re-render)
   │                  (async: gọi API)   (đổi state)                              ▲
   └──────────────────────────── chỉ đọc state / getters ───────────────────────┘
```

> **Vì sao tách Mutations (sync) và Actions (async)?** Mutation **phải đồng bộ** để **Vue Devtools ghi
> lại được từng thay đổi state theo thứ tự** (time-travel debugging) — nếu mutation async thì devtools
> không biết state đổi lúc nào. Mọi việc async (gọi API, `setTimeout`) dồn vào **action**, xong mới
> `commit` mutation để đổi state. Đây là câu hỏi phỏng vấn **rất hay gặp**.

```js
// store/index.js — Vuex 4 (cho Vue 3)
import { createStore } from "vuex";

export default createStore({
  state: () => ({ count: 0, users: [] }),

  getters: {                                   // như computed cho store (có cache)
    doubleCount: (state) => state.count * 2,
  },

  mutations: {                                 // ĐỒNG BỘ — cách duy nhất đổi state
    increment(state, n = 1) { state.count += n; },
    setUsers(state, users) { state.users = users; },
  },

  actions: {                                   // BẤT ĐỒNG BỘ — gọi API rồi commit
    async fetchUsers({ commit }) {
      const res = await fetch("/api/users");
      commit("setUsers", await res.json());    // xong async → commit mutation
    },
  },
});
```

```js
// Trong component (Options API)
this.$store.state.count;              // đọc state
this.$store.getters.doubleCount;      // đọc getter
this.$store.commit("increment", 5);   // gọi mutation (sync)
this.$store.dispatch("fetchUsers");   // gọi action (async)
```

> **Vuex vs Pinia (RẤT quan trọng — hay bị hỏi):** từ Vue 3, **Pinia là thư viện state management CHÍNH
> THỨC được khuyến nghị**, còn **Vuex đã ở chế độ bảo trì (maintenance mode)** — không phát triển tính
> năng mới. Pinia đơn giản hơn: **bỏ hẳn Mutations** (đổi state trực tiếp trong action), hỗ trợ
> **TypeScript** tốt, **modular sẵn** (mỗi store là một hàm), hợp **Composition API** (xem [câu 6](#q6)).
> Vì vậy: **dự án mới → dùng Pinia**; Vuex vẫn cần biết vì **nhiều dự án cũ đang chạy Vuex**.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Vuex** = thư viện **quản lý state tập trung** chính thức của Vue (theo pattern **Flux**): gom state
  dùng chung vào **một store** (**single source of truth**), reactive nên component tự re-render khi state đổi.
- **Giải quyết:** **prop drilling** và tình trạng state bị sửa lộn xộn khi **nhiều component ở xa cần dùng
  chung dữ liệu** (user, giỏ hàng, theme...).
- **5 khái niệm:** **State** (dữ liệu) · **Getters** (state dẫn xuất, có cache — như `computed`) ·
  **Mutations** (cách **duy nhất** đổi state, **bắt buộc sync**, gọi qua `commit`) · **Actions** (xử lý
  **async** rồi commit, gọi qua `dispatch`) · **Modules** (chia nhỏ store).
- **Luồng một chiều:** `dispatch → Action (async) → commit → Mutation (sync) → State → component`. Tách
  sync/async để **Devtools time-travel** ghi lại được từng thay đổi.
- **Chốt:** Vue 3 nay khuyến nghị **Pinia** (bỏ mutations, TS tốt, hợp Composition API); **Vuex đã ở
  maintenance mode** — vẫn cần biết vì nhiều dự án cũ dùng.

[↑ Mục lục](#mục-lục)

<a id="q7-1"></a>
### 7.1. Pinia là gì? (Pinia vs Vuex)

#### 📖 Giới thiệu kiến thức

**Pinia** là **thư viện quản lý state chính thức, được khuyến nghị** cho Vue (cả Vue 3 lẫn Vue 2.7) — là
**"người kế nhiệm" của Vuex** ([câu 7](#q7)). Do chính thành viên Vue core team phát triển; từ khi Vue 3
ổn định, tài liệu Vue chính thức khuyên **dùng Pinia cho dự án mới**, còn **Vuex ở chế độ bảo trì
(maintenance mode)**. Có thể hình dung Pinia là **"Vuex 5"** nhưng được thiết kế lại cho gọn và hợp
**Composition API** ([câu 6](#q6)).

**Ý tưởng cốt lõi giống Vuex:** vẫn là **store tập trung, reactive, single source of truth** để chia sẻ
state giữa nhiều component. Điểm khác nằm ở **thiết kế đơn giản hơn**.

**Ba khác biệt lớn so với Vuex (điểm hay hỏi nhất):**

1. **BỎ HẲN Mutations:** Pinia chỉ còn **State · Getters · Actions** (không còn mutation). Đổi state
   thì **sửa trực tiếp** trong action (`this.count++`) hoặc gán thẳng — không cần `commit`. Bớt rất nhiều
   boilerplate; action **vừa sync vừa async** đều được.
2. **KHÔNG có Modules lồng nhau:** thay vì một store lớn chia module + `namespaced`, Pinia cho **định
   nghĩa nhiều store phẳng, độc lập** — mỗi store một **id** riêng (tự namespace). Store nào cần thì
   **import store đó**; store có thể dùng store khác. → **modular by design**.
3. **Hỗ trợ TypeScript "chuẩn" + tự suy luận kiểu:** không cần khai báo type phức tạp như Vuex; DX và
   auto-complete tốt hơn hẳn. Pinia cũng **nhẹ (~1.5kb)** và có Devtools/SSR support.

**Hai cách định nghĩa store** (linh hoạt):

```js
// stores/counter.js — CÁCH 1: Option Store (giống Options API / gần Vuex)
import { defineStore } from "pinia";

export const useCounterStore = defineStore("counter", {
  state: () => ({ count: 0 }),
  getters: { double: (s) => s.count * 2 },  // như computed
  actions: {                                 // sync hoặc async — KHÔNG có mutation
    increment() { this.count++; },           // đổi state TRỰC TIẾP
    async fetchInit() { this.count = await api(); },
  },
});
```

```js
// stores/counter.js — CÁCH 2: Setup Store (giống composable / Composition API — xem câu 6.1)
import { defineStore } from "pinia";
import { ref, computed } from "vue";

export const useCounterStore = defineStore("counter", () => {
  const count = ref(0);                       // state ↔ ref
  const double = computed(() => count.value * 2); // getter ↔ computed
  function increment() { count.value++; }     // action ↔ function
  return { count, double, increment };
});
```

```vue
<script setup>
import { storeToRefs } from "pinia";
import { useCounterStore } from "@/stores/counter";

const store = useCounterStore();
// ⚠️ destructure trực tiếp sẽ MẤT reactivity → dùng storeToRefs cho state/getter
const { count, double } = storeToRefs(store);
const { increment } = store;                  // action thì destructure thẳng được
</script>

<template>
  <button @click="increment">{{ count }} → {{ double }}</button>
</template>
```

**Bảng so sánh Vuex vs Pinia:**

| Tiêu chí | **Vuex** ([câu 7](#q7)) | **Pinia** |
|---|---|---|
| **Trạng thái** | Maintenance mode | **Khuyến nghị chính thức** cho dự án mới |
| **Khái niệm** | State, Getters, **Mutations**, Actions | State, Getters, **Actions** (bỏ mutation) |
| **Đổi state** | Phải qua `commit` mutation (sync) | **Sửa trực tiếp** trong action (sync/async) |
| **Chia nhỏ** | **Modules** lồng nhau + `namespaced` | **Nhiều store phẳng, độc lập** (mỗi store 1 id) |
| **TypeScript** | Kém, phải khai báo nhiều | **Rất tốt, tự suy luận kiểu** |
| **Cú pháp** | Options-based | **Option Store hoặc Setup Store** (hợp Composition API) |
| **Kích thước** | Lớn hơn | **~1.5kb, nhẹ hơn** |
| **Devtools/SSR** | Có | Có |

> **Vì sao bỏ được Mutations?** Ở Vuex, mutation sync tồn tại chủ yếu để **Devtools ghi lại từng thay
> đổi** (time-travel). Pinia đạt được điều đó bằng cách **theo dõi thay đổi state qua action** mà không
> cần tách riêng mutation → vừa giữ được debug tốt, vừa **bớt boilerplate**. Đây là lý do Pinia gọn hơn
> nhiều mà không mất khả năng debug.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Pinia** = thư viện state management **chính thức được khuyến nghị** của Vue hiện nay, **kế nhiệm Vuex**
  (Vuex nay ở maintenance mode). Cùng ý tưởng: **store tập trung, reactive, single source of truth**, nhưng
  **thiết kế gọn hơn** và hợp **Composition API** ([câu 6](#q6)).
- **3 khác biệt lớn với Vuex:**
  - **Bỏ Mutations** — chỉ còn **State/Getters/Actions**; đổi state **trực tiếp** trong action (sync/async),
    không cần `commit` → ít boilerplate.
  - **Không Modules lồng nhau** — nhiều **store phẳng độc lập**, mỗi store một **id** (tự namespace),
    modular by design.
  - **TypeScript tự suy luận kiểu**, **nhẹ (~1.5kb)**, giữ **Devtools/SSR**.
- **Hai cách viết store:** **Option Store** (giống Options API/Vuex) và **Setup Store** (giống composable).
  Khi destructure state/getter phải dùng **`storeToRefs`** để **không mất reactivity**; action destructure thẳng được.
- **Chốt:** dự án mới → **Pinia**; vẫn cần biết Vuex vì nhiều dự án cũ dùng.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. Phân biệt Nuxt.js và Vue.js

### 📖 Giới thiệu kiến thức

Điểm dễ nhầm nhất: **Nuxt KHÔNG phải đối thủ của Vue** — Nuxt **xây dựng TRÊN Vue**. Quan hệ giống
**React ↔ Next.js**: Vue là **framework UI lõi**, còn Nuxt là **meta-framework (framework cấp cao hơn)**
bọc quanh Vue để lo sẵn những thứ Vue thuần bắt bạn tự làm (SSR, routing, cấu hình build, SEO...).

**Vue.js — framework UI lõi:** cung cấp component, reactivity, Virtual DOM (xem [câu 1](#q1)), lifecycle
(xem [câu 3](#q3)). Mặc định Vue tạo **SPA render phía client (CSR)**: trình duyệt tải một `index.html`
gần như rỗng + bundle JS, rồi JS **dựng toàn bộ giao diện ở client**. Muốn có router, state
management, cấu trúc thư mục, SSR... bạn phải **tự cài và cấu hình** (Vue Router, Pinia, Vite...).

**Nuxt.js — meta-framework trên Vue:** đóng gói sẵn một bộ giải pháp "có quy ước" (convention over
configuration): **nhiều chế độ render** (SSR, SSG, CSR, hybrid), **routing theo file**, **auto-import**,
**data fetching hiểu SSR**, **server API routes** (thành full-stack), tối ưu **SEO** — tất cả out of the box.

**Khác biệt cốt lõi — kiểu render & SEO:**
- Vue thuần (**CSR/SPA**): HTML ban đầu rỗng → **SEO kém**, **first paint** chậm hơn (chờ JS chạy xong).
- Nuxt (**SSR** — render HTML ở server, hoặc **SSG** — sinh HTML tĩnh lúc build): trả về **HTML đã có
  nội dung** → **SEO tốt**, hiển thị nội dung nhanh hơn; sau đó **hydration** để trang tương tác được.

**Bảng so sánh:**

| Tiêu chí | **Vue.js** | **Nuxt.js** |
|---|---|---|
| **Vai trò** | Framework UI **lõi** | **Meta-framework** xây trên Vue |
| **Render mặc định** | **CSR / SPA** (render ở client) | **SSR / SSG** (+ CSR, hybrid) |
| **SEO** | Kém (HTML ban đầu rỗng) | **Tốt** (HTML render sẵn) |
| **Routing** | Tự cài **Vue Router**, khai báo thủ công | **File-based** (thư mục `pages/`) tự sinh route |
| **Cấu trúc dự án** | Tự do, tự tổ chức | **Quy ước** (`pages`, `layouts`, `composables`, `server`) |
| **Data fetching** | Tự làm (fetch/axios trong lifecycle) | `useFetch` / `useAsyncData` (hiểu SSR, tránh double-fetch) |
| **Backend / API** | Không có | **Server routes** (Nitro) → **full-stack** |
| **Auto-import** | Không | Có (component, composable tự import) |
| **Cấu hình build** | Tự dựng (Vite/Webpack) | Sẵn & tối ưu (chỉ `nuxt.config`) |

**Khi nào dùng cái nào:**
- **Vue thuần:** SPA sau đăng nhập (dashboard, admin — **SEO không quan trọng**), widget/component nhúng
  vào trang có sẵn, hoặc muốn **toàn quyền tự cấu hình**, dự án nhỏ.
- **Nuxt:** trang **cần SEO** (landing, blog, e-commerce, marketing), cần **SSR/SSG**, muốn **full-stack**
  (API ngay trong Nuxt), hoặc muốn **cấu trúc & quy ước sẵn** để đội nhiều người làm nhanh, nhất quán.

> **Chốt:** Nuxt = Vue + SSR/SSG + file-based routing + server API + auto-import + quy ước, đóng gói sẵn.
> Không thay thế Vue mà **dùng Vue bên trong**; học Nuxt vẫn cần biết Vue trước.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Nuxt không phải đối thủ của Vue — Nuxt xây TRÊN Vue** (giống **Next.js ↔ React**). Vue là
  **framework UI lõi**; Nuxt là **meta-framework** lo sẵn những thứ Vue thuần phải tự làm.
- **Khác cốt lõi — render & SEO:** Vue mặc định **CSR/SPA** (HTML rỗng ban đầu → **SEO kém**); Nuxt
  cho **SSR/SSG** (HTML render sẵn → **SEO tốt**, hiển thị nhanh, rồi hydration).
- **Nuxt cung cấp sẵn:** **file-based routing** (`pages/`), **auto-import**, **data fetching hiểu SSR**
  (`useFetch`/`useAsyncData`), **server API routes** (full-stack), cấu trúc **quy ước**, build tối ưu sẵn.
- **Chọn:** SPA sau login / SEO không quan trọng / muốn tự do cấu hình → **Vue**; cần **SEO, SSR/SSG,
  full-stack, cấu trúc sẵn** → **Nuxt**.

[↑ Mục lục](#mục-lục)

<a id="q8-1"></a>
### 8.1. Phân biệt SSR, CSR và SSG

#### 📖 Giới thiệu kiến thức

Đây là ba **chiến lược render** một web app — chính là các chế độ mà Nuxt hỗ trợ ([câu 8](#q8)). Khác
biệt cốt lõi gói gọn trong một câu hỏi: **HTML được tạo Ở ĐÂU và VÀO LÚC NÀO?**

**① CSR — Client-Side Rendering (render ở client):**
Server trả về một `index.html` **gần như rỗng** + bundle JavaScript. Trình duyệt tải JS về, **JS mới dựng
toàn bộ giao diện** (DOM) ở client. Đây là chế độ **mặc định của Vue/React SPA**.
- **Ưu:** sau lần tải đầu, **chuyển trang cực nhanh** (không reload cả trang), tương tác mượt; **server
  nhẹ** (chỉ phục vụ file tĩnh + API).
- **Nhược:** **first load chậm** (chờ tải + chạy JS mới thấy nội dung); **SEO kém** (HTML ban đầu rỗng,
  bot khó đọc); phụ thuộc JS bật.

**② SSR — Server-Side Rendering (render ở server, mỗi request):**
Với **mỗi request**, **server chạy code tạo ra HTML đầy đủ nội dung** rồi trả về → trình duyệt hiển thị
ngay, sau đó tải JS để **hydration** (gắn sự kiện cho trang tương tác được). Dùng ở **Nuxt, Next.js**.
- **Ưu:** **SEO tốt** (HTML có sẵn nội dung); **hiển thị nội dung nhanh** (first paint sớm); hợp **dữ
  liệu động / cá nhân hóa** (mỗi request render mới nhất).
- **Nhược:** **tải server cao** (render mỗi request), **TTFB** có thể chậm hơn; cần **server Node chạy
  liên tục** → phức tạp & tốn kém hơn.

**③ SSG — Static Site Generation (sinh HTML tĩnh lúc build):**
HTML được **render sẵn MỘT LẦN lúc build**, cho ra các file HTML tĩnh → deploy lên **CDN**. Dùng ở
**Nuxt generate, Next.js static, Gatsby, VitePress**.
- **Ưu:** **nhanh nhất** (HTML nằm sẵn trên CDN, không tính toán khi request); **SEO tốt**; **rẻ & an
  toàn** (chỉ static hosting), chịu tải rất cao.
- **Nhược:** nội dung **tĩnh** — mỗi lần data đổi phải **build lại**; không hợp nội dung đổi liên tục /
  cá nhân hóa theo user. (**ISR** — Incremental Static Regeneration — giảm nhược điểm này bằng cách
  regenerate từng trang theo chu kỳ.)

**Bảng so sánh:**

| Tiêu chí | **CSR** | **SSR** | **SSG** |
|---|---|---|---|
| **HTML tạo ở đâu** | Client (trình duyệt) | Server (mỗi request) | Lúc **build** (1 lần) |
| **Khi nào** | Runtime, ở client | Runtime, mỗi request | Build time |
| **SEO** | Kém | Tốt | Tốt |
| **First paint** | Chậm | Nhanh | **Nhanh nhất** |
| **Tải server** | Thấp | **Cao** | Rất thấp (CDN) |
| **Dữ liệu động / cá nhân hóa** | Tốt (fetch ở client) | **Tốt** (mỗi request mới) | Kém (phải rebuild) |
| **Cần server chạy** | Không | **Có** (Node) | Không (chỉ static host) |
| **Use case** | Dashboard sau login, app nội bộ | E-commerce, tin tức cá nhân hóa, feed | Blog, docs, landing, marketing |

> **Điểm nối:** cả **SSR và SSG** đều gửi HTML render sẵn rồi **hydration** để tương tác — khác CSR (dựng
> DOM từ đầu ở client). Framework hiện đại (Nuxt 3) cho **chọn chế độ theo từng route** (hybrid), thậm chí
> **ISR** = SSG + tự regenerate → không còn là "chọn một trong ba" cứng nhắc.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Khác nhau ở **HTML tạo ở đâu & khi nào:**
  - **CSR:** ở **client** lúc runtime — server trả HTML rỗng + JS, browser tự dựng UI (Vue/React SPA).
  - **SSR:** ở **server**, **mỗi request** — trả HTML đầy đủ rồi hydration (Nuxt, Next).
  - **SSG:** lúc **build**, một lần → file tĩnh trên CDN (Nuxt generate, Gatsby).
- **SEO & first paint:** CSR **kém** (HTML rỗng, load chậm); SSR & SSG **tốt** (HTML sẵn), SSG **nhanh
  nhất** (CDN).
- **Đánh đổi:** CSR **server nhẹ** nhưng SEO kém; SSR hợp **dữ liệu động/cá nhân hóa** nhưng **tải server
  cao**; SSG **nhanh & rẻ nhất** nhưng nội dung **tĩnh** (đổi data phải rebuild — trừ **ISR**).
- **Chọn:** dashboard sau login → **CSR**; nội dung động cần SEO (e-commerce, tin tức) → **SSR**; nội
  dung ít đổi (blog/docs/landing) → **SSG**. Nuxt 3 cho **hybrid** theo route.

[↑ Mục lục](#mục-lục)

<a id="q8-2"></a>
### 8.2. So sánh tốc độ giữa Nuxt và Vue

#### 📖 Giới thiệu kiến thức

**Bẫy tư duy cần gỡ trước:** "Nuxt nhanh hơn Vue" hay ngược lại đều **không chính xác một cách tuyệt
đối** — vì Nuxt **chạy chính Vue bên trong** ([câu 8](#q8)). Lõi render (reactivity, diff/patch Virtual
DOM — [câu 1](#q1)) **giống hệt nhau**. Khác biệt tốc độ **không đến từ framework**, mà đến từ **chế độ
render** (CSR của Vue thuần vs SSR/SSG của Nuxt — [câu 8.1](#q8-1)). Vì vậy phải hỏi **"nhanh ở chỉ số
nào?"** chứ không có câu trả lời "cái nào nhanh hơn" chung chung.

**Tách theo từng chỉ số (metric):**

- **First Contentful Paint (FCP) — lúc thấy nội dung đầu tiên:** **Nuxt (SSR/SSG) nhanh hơn.** Nuxt trả
  **HTML đã có nội dung** → thấy ngay; Vue thuần (CSR) trả HTML rỗng, phải chờ **tải + chạy JS** mới dựng
  UI → có "màn hình trắng" ban đầu. → *Perceived performance* của Nuxt tốt hơn khi mới vào.
- **TTFB (Time To First Byte) — server trả byte đầu:** **Vue (CSR/SSG tĩnh) nhanh hơn.** File tĩnh phục
  vụ tức thì (CDN); còn **SSR của Nuxt phải render HTML mỗi request** → TTFB chậm hơn.
- **Time To Interactive (TTI) — lúc bấm được:** không hẳn Nuxt thắng. SSR thấy nội dung sớm **nhưng phải
  chờ hydration** (tải JS + gắn sự kiện) mới tương tác → có khoảng "thấy mà bấm chưa ăn" (uncanny valley);
  **hydration còn tốn CPU client**. Vue CSR thấy muộn nhưng khi hiện ra thường đã tương tác luôn.
- **Điều hướng sau khi đã tải (client-side navigation):** **tương đương.** Sau hydration, Nuxt chạy y
  như một SPA Vue → chuyển trang nhanh như nhau (cùng Vue Router, cùng runtime).
- **Tải server / khả năng chịu tải:** **Vue (CSR/SSG) nhẹ hơn** (chỉ serve tĩnh + API); **Nuxt SSR nặng**
  (render mỗi request) → cần scale server, tốn tài nguyên hơn.

**Bảng tổng hợp:**

| Chỉ số | **Vue thuần (CSR)** | **Nuxt (SSR)** | **Nuxt (SSG)** |
|---|---|---|---|
| **FCP (thấy nội dung)** | Chậm (chờ JS) | **Nhanh** | **Nhanh nhất** |
| **TTFB** | **Nhanh** | Chậm (render/request) | **Nhanh** (CDN) |
| **TTI (bấm được)** | Ổn | Chậm hơn (chờ hydration) | Chậm hơn (chờ hydration) |
| **Điều hướng nội trang** | Nhanh | Nhanh (sau hydration) | Nhanh (sau hydration) |
| **Tải server** | **Thấp** | Cao | **Rất thấp** |
| **Lõi render (diff/patch)** | Giống nhau | Giống nhau | Giống nhau |

> **Chi phí thêm của Nuxt:** SSR tốn công **render ở server** + **hydration ở client**; bù lại được
> **FCP nhanh + SEO tốt** ([câu 8.1](#q8-1)). Nuxt cũng có thể **chậm hơn ở dev** (build layer nặng hơn
> Vue CLI/Vite thuần) và bundle lớn hơn một chút do runtime SSR. Không có "bữa trưa miễn phí".

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Không có câu trả lời tuyệt đối** — Nuxt dùng **chính Vue bên trong**, lõi render **giống hệt**; khác
  biệt tốc độ đến từ **chế độ render (CSR vs SSR/SSG)**, không phải framework. Phải hỏi **"nhanh ở chỉ số
  nào"**.
- **Nuxt nhanh hơn ở:** **FCP / thấy nội dung lần đầu** và **SEO** (HTML render sẵn) → *perceived
  performance* tốt hơn khi mới vào.
- **Vue thuần nhanh hơn ở:** **TTFB** (file tĩnh/CDN, không render mỗi request) và **tải server nhẹ**.
- **Tương đương:** **điều hướng nội trang** sau khi tải (Nuxt sau hydration chạy như SPA Vue).
- **Đánh đổi:** SSR của Nuxt tốn **server render + hydration** (TTI có thể trễ), đổi lấy FCP + SEO. →
  *SEO/first paint chọn Nuxt; app nội bộ tải nhẹ/TTFB chọn Vue CSR.*

[↑ Mục lục](#mục-lục)

<a id="q8-3"></a>
### 8.3. Hydration là gì?

#### 📖 Giới thiệu kiến thức

**Hydration** là quá trình **"làm sống" HTML tĩnh** do server gửi xuống (SSR/SSG — [câu 8.1](#q8-1)) bằng
cách **gắn JavaScript + reactivity + event listener** vào **DOM đã có sẵn**, để trang từ "chỉ xem được"
trở nên **tương tác được**. Tên gọi "hydrate" (bù nước) là ẩn dụ: HTML server gửi xuống là bộ khung
**"khô"** (không có hành vi), Vue **"đổ" JS/reactivity vào cho nó "sống"**.

**Luồng hoạt động (SSR/SSG):**

1. Server trả về **HTML đã render đầy đủ nội dung** → trình duyệt **hiển thị ngay** (thấy được, nhưng
   **bấm chưa ăn** — mới chỉ là HTML tĩnh, chưa có event).
2. Trình duyệt tải **bundle JavaScript** của app.
3. Vue chạy, dựng lại **Virtual DOM** ([câu 1](#q1)) trong bộ nhớ, rồi **"khớp" (match)** nó với **DOM
   thật server đã gửi** — **không vẽ lại từ đầu**, chỉ **gắn reactivity + event listener** vào các node
   có sẵn. Đây chính là **hydration**.
4. Sau bước này trang mới **thực sự tương tác** (click, `v-model`, `@event`...) → từ đây chạy như một SPA
   bình thường.

**Chi phí — điểm hay bị hỏi:** hydration **tốn CPU ở client** (phải dựng lại VNode + duyệt cả cây để
khớp) → tạo khoảng trễ **TTI (Time To Interactive)**: người dùng **thấy nội dung sớm nhưng bấm chưa được
ngay** — hiện tượng "uncanny valley" đã nêu ở [câu 8.2](#q8-2).

**Hydration mismatch (lỗi thường gặp):** nếu **HTML server render KHÁC** với gì client dựng ra khi
hydrate → Vue cảnh báo và phải sửa lại DOM (tốn kém, có thể nháy giao diện). Nguyên nhân điển hình:
render nội dung phụ thuộc **thời gian/ngẫu nhiên/môi trường client** ở SSR (`Date.now()`, `Math.random()`,
`window`, `localStorage`...). → Tránh bằng cách không render những thứ này khác nhau giữa server và client
(hoặc chỉ render sau khi mounted ở client).

**Giảm chi phí hydration:** **partial / lazy hydration** (chỉ hydrate khi cần / khi cuộn tới),
**islands architecture** — chỉ hydrate những "đảo" cần tương tác, phần tĩnh để nguyên (Astro theo hướng
này; Nuxt 3 có `<NuxtIsland>` và lazy hydration components).

> **Lưu ý:** **CSR không có bước hydration** — vì không có HTML render sẵn để "khớp"; client **dựng DOM
> từ đầu** rồi gắn sự kiện luôn trong một lần. Hydration là khái niệm **chỉ có ở SSR/SSG**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Hydration** = quá trình Vue **gắn JS + reactivity + event listener** vào **HTML tĩnh** server gửi
  xuống (SSR/SSG) để trang **tương tác được** ("làm sống" HTML khô).
- **Luồng:** HTML đầy đủ hiển thị ngay (bấm chưa ăn) → tải JS → Vue dựng lại **Virtual DOM** và **khớp**
  với DOM có sẵn (không vẽ lại, chỉ gắn sự kiện) → trang tương tác.
- **Chi phí:** tốn **CPU client** → trễ **TTI** (thấy sớm mà bấm chưa được ngay — uncanny valley).
- **Hydration mismatch:** HTML server ≠ client (do `Date.now`/`Math.random`/`window`...) → cảnh báo +
  sửa DOM; tránh bằng cách không render nội dung khác nhau giữa server/client.
- **CSR không cần hydration** (dựng DOM từ đầu ở client). Giảm chi phí: **lazy/partial hydration,
  islands** (Nuxt 3 `<NuxtIsland>`).

[↑ Mục lục](#mục-lục)

<!--
NHẮC FORMAT (xem /CLAUDE.md §4):

<a id="q1"></a>
## 1. <Câu hỏi>

### 📖 Giới thiệu kiến thức
...

### 🎯 Tóm tắt trả lời phỏng vấn
...

[↑ Mục lục](#mục-lục)
-->
