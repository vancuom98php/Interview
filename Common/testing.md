<a id="mục-lục"></a>
# Common — Testing

> Kiểm thử phần mềm: unit test · integration test · e2e · TDD/BDD · test double (mock/stub/spy) ·
> code coverage · kim tự tháp test (testing pyramid).

## 📑 Mục lục

1. [Phân biệt unit test và functional test](#q1)
2. [Phân biệt integration test và functional test](#q2)
3. [E2E test (End-to-End) là gì?](#q3)
4. [Phân biệt integration test và e2e test](#q4)
5. [Testing Pyramid (kim tự tháp test) là gì?](#q5)
   - 5.1. [Testing Trophy (Kent C. Dodds) & so với Pyramid](#q5-1)
6. [Phân biệt smoke test và regression test](#q6)
7. [Test double (đối tượng giả thay dependency thật) là gì?](#q7)
   - 7.1. [Phân biệt mock, stub và spy](#q7-1)
   - 7.2. [Mục đích của mock/stub/spy & tại sao phải tách?](#q7-2)
8. [AAA pattern (Arrange–Act–Assert) là gì?](#q8)
9. [Phân biệt TDD và BDD](#q9)
10. [Testing trong Laravel (trình bày ngắn gọn)](#q10)
    - 10.1. [Triển khai mock, stub, spy trong Laravel](#q10-1)
11. [Testing trong NestJS (trình bày ngắn gọn)](#q11)
    - 11.1. [Viết unit test trong NestJS: dùng gì & tổ chức thế nào?](#q11-1)
    - 11.2. [Triển khai mock, stub, spy trong NestJS](#q11-2)
    - 11.3. [Testing trong NestJS có tạo data thật không?](#q11-3)

---

<a id="q1"></a>
## 1. Phân biệt unit test và functional test

### 📖 Giới thiệu kiến thức

Hai khái niệm này **dễ nhầm** vì được phân loại theo **hai trục khác nhau**, không phải hai lựa chọn loại
trừ nhau:

- **Unit test** phân loại theo **phạm vi (scope/level)** — kiểm thử **đơn vị nhỏ nhất** của code.
- **Functional test** phân loại theo **mục đích (what) & góc nhìn** — kiểm thử một **chức năng hoàn chỉnh**
  có chạy **đúng yêu cầu (requirement/spec)** theo góc nhìn người dùng hay không.

**1) Unit test — "code có chạy đúng không?" (white-box)**

- **Phạm vi:** 1 **hàm / method / class** được **cô lập**; mọi dependency (DB, API, service khác) đều được
  **mock/stub**.
- **Góc nhìn:** **white-box** — người viết **biết code bên trong**, test theo logic/nhánh code.
- **Ai viết:** thường là **developer**, chạy trong CI mỗi lần commit.
- **Tốc độ:** **rất nhanh** (không I/O thật) → chạy hàng nghìn test trong vài giây.
- **Mục tiêu:** khẳng định **một mảnh logic** đúng, khoanh vùng lỗi chính xác.

**2) Functional test — "chức năng có đúng yêu cầu không?" (black-box)**

- **Phạm vi:** một **chức năng/tính năng hoàn chỉnh** (vd: "đăng nhập", "tạo đơn hàng"), thường đi qua
  **nhiều thành phần thật** phối hợp (controller → service → DB...).
- **Góc nhìn:** **black-box** — chỉ quan tâm **input → output** đúng **spec**, **không cần biết** code
  bên trong cài đặt thế nào.
- **Ai viết:** **QA/tester** (hoặc dev), dựa trên **tài liệu yêu cầu**.
- **Tốc độ:** **chậm hơn** vì dùng dependency thật (DB, HTTP...).
- **Mục tiêu:** khẳng định **hành vi nghiệp vụ** đúng với mong đợi người dùng.

**Điểm mấu chốt — không loại trừ nhau:** "functional" là **ô lớn (umbrella)** phân theo *mục đích*, bao
gồm cả **integration, system, E2E, smoke, regression**... miễn là kiểm tra **chức năng so với yêu cầu**.
Đối lập của functional test là **non-functional test** (performance, security, load, usability). Còn
unit/integration/E2E là phân theo *phạm vi*. Vì vậy một **E2E test** vừa là test theo phạm vi cao nhất,
vừa **là một dạng functional test**.

| Tiêu chí | **Unit test** | **Functional test** |
|---|---|---|
| **Phân loại theo** | Phạm vi (nhỏ nhất) | Mục đích (đúng yêu cầu) |
| **Đối tượng test** | 1 hàm/class cô lập | 1 chức năng hoàn chỉnh |
| **Góc nhìn** | White-box (biết code) | Black-box (chỉ input→output) |
| **Dependency** | Mock/stub hết | Dùng thành phần **thật** |
| **Ai viết** | Developer | QA/tester (hoặc dev) |
| **Tốc độ** | Rất nhanh | Chậm hơn |
| **Xác nhận** | Code logic đúng | Nghiệp vụ đúng spec |

**Ví dụ cùng một hàm `calcDiscount`:**

```js
// UNIT TEST — cô lập hàm thuần, không DB/HTTP; kiểm tra logic từng nhánh
it('giảm 10% cho đơn > 1 triệu', () => {
  expect(calcDiscount(1_200_000)).toBe(120_000);   // chỉ test 1 mảnh logic
});

// FUNCTIONAL TEST — test cả chức năng "đặt hàng" theo yêu cầu, qua API + DB thật
it('đặt đơn 1.2tr thì tổng tiền phải trừ đúng khuyến mãi', async () => {
  const res = await request(app).post('/orders')
    .send({ items: [{ id: 1, price: 1_200_000 }] });
  expect(res.status).toBe(201);
  expect(res.body.total).toBe(1_080_000);          // kiểm tra hành vi nghiệp vụ đúng spec
});
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Khác trục phân loại:** **unit** phân theo **phạm vi** (đơn vị nhỏ nhất — 1 hàm/class cô lập);
  **functional** phân theo **mục đích** (chức năng có đúng **yêu cầu** không).
- **Unit test:** **white-box**, **mock hết** dependency, **dev viết**, **rất nhanh**, khẳng định
  **code logic** đúng và **khoanh vùng lỗi** chính xác.
- **Functional test:** **black-box** (chỉ quan tâm **input→output** theo spec), dùng **thành phần thật**,
  **QA/tester viết**, chậm hơn, khẳng định **hành vi nghiệp vụ** đúng.
- **Không loại trừ nhau:** functional là **ô lớn** gồm integration/system/E2E...; đối lập của nó là
  **non-functional** (performance/security). Một **E2E test cũng là một functional test**.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Phân biệt integration test và functional test

### 📖 Giới thiệu kiến thức

Giống cặp [unit vs functional](#q1), đây cũng là **hai trục phân loại khác nhau** — nên dễ chồng lấn:

- **Integration test** phân loại theo **phạm vi (scope)** — nằm **giữa** unit và E2E, kiểm thử **nhiều
  thành phần ghép lại** có phối hợp đúng không.
- **Functional test** phân loại theo **mục đích (what)** — chức năng có chạy **đúng yêu cầu** không, nhìn
  từ góc **người dùng**.

**1) Integration test — "các mảnh ghép có nói chuyện đúng không?"**

- **Mục tiêu:** tập trung vào **điểm ghép nối (interface/contract/data flow)** giữa các thành phần — vd:
  service ↔ repository ↔ **DB thật**, module A ↔ module B, code ↔ **API bên ngoài**. Bug hay gặp: sai
  kiểu dữ liệu qua ranh giới, query sai, mapping ORM lệch, contract API đổi.
- **Góc nhìn:** thường **white/gray-box** — người viết **biết đang ghép thành phần nào** với nhau.
- **Dependency:** dùng **một phần thật** (thường là DB thật), có thể vẫn **mock service ngoài**.
- **Ai viết:** thường là **developer**.
- **Đặc điểm:** có thể **thuần kỹ thuật**, không nhất thiết ứng với một yêu cầu nghiệp vụ đầy đủ (vd chỉ
  kiểm tra "repository lưu đúng bản ghi xuống DB").

**2) Functional test — "chức năng có đúng spec không?"**

- **Mục tiêu:** kiểm chứng **hành vi nghiệp vụ** của **một chức năng hoàn chỉnh** so với **yêu cầu**
  (vd: "đặt hàng > 1 triệu thì được giảm 10%").
- **Góc nhìn:** **black-box** — chỉ quan tâm **input → output** theo spec, không cần biết bên trong ghép
  mấy thành phần.
- **Ai viết:** **QA/tester** (hoặc dev) dựa trên tài liệu yêu cầu.

**Điểm mấu chốt — khác trọng tâm:** integration hỏi *"các thành phần ghép nối có đúng không?"*, functional
hỏi *"chức năng có đúng yêu cầu người dùng không?"*. Chúng **chồng lấn** nhưng không trùng: một integration
test có thể **kỹ thuật thuần** (không map tới requirement nào), còn functional test có thể **chỉ cần black-box
qua API** mà không quan tâm bên trong có bao nhiêu tầng. Khi một bài test vừa ghép nhiều thành phần thật vừa
kiểm chứng đúng nghiệp vụ → nó **vừa là integration vừa là functional test**.

| Tiêu chí | **Integration test** | **Functional test** |
|---|---|---|
| **Phân loại theo** | Phạm vi (nhiều thành phần ghép) | Mục đích (đúng yêu cầu) |
| **Trọng tâm** | **Điểm ghép nối** (interface/data flow) | **Hành vi nghiệp vụ** theo spec |
| **Góc nhìn** | White/gray-box (biết thành phần nào ghép) | Black-box (chỉ input→output) |
| **Dependency** | Một phần thật (DB thật), có thể mock service ngoài | Theo góc người dùng, thường thật |
| **Có gắn requirement?** | Không nhất thiết (có thể thuần kỹ thuật) | Luôn gắn với **yêu cầu chức năng** |
| **Ai viết** | Developer | QA/tester (hoặc dev) |

**Ví dụ phân biệt trọng tâm:**

```js
// INTEGRATION TEST — kiểm tra ghép nối service ↔ DB thật đúng (kỹ thuật)
it('OrderRepository lưu và đọc lại đúng bản ghi', async () => {
  const saved = await repo.create({ total: 1_080_000 });
  const found = await repo.findById(saved.id);        // đi qua DB thật
  expect(found.total).toBe(1_080_000);                // quan tâm: mapping ORM + DB đúng
});

// FUNCTIONAL TEST — kiểm tra chức năng "đặt hàng có giảm giá" đúng yêu cầu (nghiệp vụ)
it('đơn 1.2tr được giảm 10% theo chính sách', async () => {
  const res = await request(app).post('/orders')
    .send({ items: [{ id: 1, price: 1_200_000 }] });
  expect(res.body.total).toBe(1_080_000);             // quan tâm: business rule đúng spec
});
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Khác trục & trọng tâm:** **integration** phân theo **phạm vi**, hỏi *"các thành phần **ghép nối** có
  đúng không?"* (interface/data flow); **functional** phân theo **mục đích**, hỏi *"**chức năng** có đúng
  **yêu cầu** người dùng không?"*.
- **Integration test:** white/gray-box, tập trung **điểm ghép** (service↔DB, module↔module, API ngoài),
  dùng **một phần thật** (DB thật) — có thể **thuần kỹ thuật**, không map trực tiếp tới 1 requirement.
- **Functional test:** black-box, kiểm chứng **hành vi nghiệp vụ** theo **spec**, luôn gắn với một
  **yêu cầu chức năng**.
- **Chồng lấn nhưng không trùng:** một bài vừa ghép nhiều thành phần thật vừa đúng nghiệp vụ thì **vừa là
  integration vừa là functional**; nhưng integration có thể chỉ test kỹ thuật, còn functional có thể chỉ
  black-box qua API.

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. E2E test (End-to-End) là gì?

### 📖 Giới thiệu kiến thức

**E2E (End-to-End) test** là loại kiểm thử **toàn bộ luồng ứng dụng từ đầu đến cuối**, **mô phỏng hành
vi người dùng thật** đi qua **tất cả các tầng** của hệ thống — giao diện → API → business logic →
database → dịch vụ ngoài — như một **hệ thống hoàn chỉnh đang chạy thật**. Khác với unit/integration test
(kiểm thử từng phần), E2E kiểm tra cả **"hành trình" (user journey)** để chắc rằng các thành phần **ghép
lại vẫn chạy đúng** theo góc nhìn người dùng.

**Vị trí trong Testing Pyramid (kim tự tháp test):** E2E nằm ở **đỉnh** — **ít nhất, chậm nhất, đắt nhất**
nhưng cho **độ tin cậy "thật" cao nhất** (gần với trải nghiệm người dùng nhất).

```
        ▲  E2E        ← ít, chậm, đắt — test cả hệ thống thật (user journey)
       ▲▲  Integration ← nhiều thành phần phối hợp (service + DB thật)
      ▲▲▲  Unit        ← nhiều nhất, nhanh nhất — 1 hàm/class cô lập, mock dependency
```

**Phân biệt 3 tầng (hay hỏi):**

| Loại | Phạm vi | Dependency | Tốc độ | Số lượng |
|---|---|---|---|---|
| **Unit** | 1 hàm/class **cô lập** | **Mock/stub** hết | Rất nhanh | Nhiều nhất |
| **Integration** | Nhiều thành phần phối hợp | DB/service **thật một phần** | Trung bình | Vừa |
| **E2E** | **Cả hệ thống** chạy thật | Thật gần như toàn bộ | Chậm | Ít nhất |

**Ví dụ luồng E2E:** test kịch bản *"đăng ký → nhận email xác thực → đăng nhập → thêm giỏ hàng → thanh
toán"* — chạy qua **giao diện/endpoint thật**, dữ liệu đi xuống **DB thật (môi trường test)**, đúng như
người dùng thao tác.

```js
// E2E frontend — Playwright: điều khiển trình duyệt THẬT như người dùng
test('user có thể đăng nhập', async ({ page }) => {
  await page.goto('https://app.test/login');
  await page.fill('#email', 'a@x.com');
  await page.fill('#password', '123456');
  await page.click('button[type=submit]');
  await expect(page).toHaveURL(/dashboard/);   // xác nhận vào được trang chính
});

// E2E backend API — Supertest: gọi HTTP THẬT lên server + DB test
it('POST /login trả token', async () => {
  const res = await request(app.getHttpServer())
    .post('/login')
    .send({ email: 'a@x.com', password: '123456' });
  expect(res.status).toBe(201);
  expect(res.body.accessToken).toBeDefined();
});
```

**Ưu điểm:** bắt được **bug tích hợp** mà unit/integration bỏ sót (config sai, route hỏng, DB migration
thiếu, luồng auth gãy...); phản ánh **trải nghiệm thật** của người dùng.

**Nhược điểm:** **chậm**, dễ **flaky (chập chờn — lúc pass lúc fail)** do phụ thuộc mạng/thời gian/dữ liệu,
**tốn công bảo trì**, và **khó debug** khi fail (lỗi có thể ở bất kỳ tầng nào). → Vì vậy nên **ít**, chỉ
phủ các **luồng quan trọng (critical path / happy path)**, để phần còn lại cho unit/integration.

**Công cụ phổ biến:** **Cypress, Playwright, Selenium** (web UI); **Supertest + Jest** (API backend);
**Appium** (mobile).

### 🎯 Tóm tắt trả lời phỏng vấn

- **E2E test** = kiểm thử **toàn bộ luồng từ đầu đến cuối**, **mô phỏng người dùng thật** đi qua **mọi
  tầng** (UI → API → logic → DB → service ngoài) trên **hệ thống chạy thật** → xác nhận các phần **ghép
  lại vẫn đúng**.
- **Vị trí:** **đỉnh Testing Pyramid** — **ít, chậm, đắt** nhất nhưng **tin cậy thật** cao nhất.
- **So với:** **Unit** (1 hàm cô lập, mock hết, nhanh, nhiều) · **Integration** (nhiều thành phần + DB
  thật một phần) · **E2E** (cả hệ thống thật, như user).
- **Đánh đổi:** bắt được **bug tích hợp** & phản ánh trải nghiệm thật, nhưng **chậm/flaky/khó bảo trì** →
  chỉ nên phủ **critical path**, còn lại để unit/integration.
- **Công cụ:** **Cypress/Playwright/Selenium** (web), **Supertest** (API), **Appium** (mobile).

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Phân biệt integration test và e2e test

### 📖 Giới thiệu kiến thức

Khác với cặp "X vs functional" (khác **trục** phân loại), integration và e2e **cùng một trục — phạm vi
(scope)**; chúng chỉ khác nhau ở **độ rộng của phạm vi ghép nối**. Trong [Testing Pyramid](#q5):
integration nằm **giữa**, e2e nằm **đỉnh**.

**1) Integration test — ghép một *nhóm* thành phần**

- **Phạm vi:** **vài thành phần** phối hợp (một *lát cắt* của hệ thống) — vd: service ↔ repository ↔
  **DB thật**, hoặc module A ↔ module B. **Không** chạy cả hệ thống.
- **Điểm vào (entry point):** gọi **trực tiếp** vào code/hàm/API nội bộ — **không qua UI thật**.
- **Môi trường:** **một phần** — thường DB thật, nhưng **mock service ngoài** (payment, email...).
- **Trọng tâm:** **điểm ghép nối (interface/contract)** giữa các thành phần đúng chưa.
- **Tốc độ:** **nhanh vừa**, ổn định hơn.

**2) E2E test — ghép *toàn bộ* hệ thống như người dùng thật**

- **Phạm vi:** **cả hệ thống** chạy thật, đi qua **mọi tầng** (UI → API → logic → DB → service ngoài).
- **Điểm vào:** **UI/điểm vào thật** như người dùng (click nút, điền form) hoặc endpoint public.
- **Môi trường:** **giống production** — gần như **mọi thứ thật**, kể cả service ngoài.
- **Trọng tâm:** **cả hành trình người dùng (user journey)** chạy thông suốt.
- **Tốc độ:** **chậm nhất, dễ flaky, đắt** nhất.

**Điểm mấu chốt — khác ở *độ rộng* và *điểm vào*:** integration kiểm tra **một nhóm mảnh ghép** qua **gọi
code trực tiếp**; e2e kiểm tra **toàn hệ thống** qua **hành động người dùng thật** trên môi trường
production-like. Nói cách khác: **mọi e2e đều là integration ở mức rộng nhất, nhưng integration không phải
e2e** vì nó chỉ ghép một phần và không mô phỏng người dùng đầu-cuối.

| Tiêu chí | **Integration test** | **E2E test** |
|---|---|---|
| **Độ rộng phạm vi** | Vài thành phần (một lát cắt) | **Toàn bộ** hệ thống |
| **Điểm vào** | Gọi code/API nội bộ trực tiếp | **UI/điểm vào thật** như user |
| **Dependency** | DB thật, **mock service ngoài** | Gần như **mọi thứ thật** |
| **Môi trường** | Một phần/cô lập | **Giống production** |
| **Trọng tâm** | Điểm **ghép nối** giữa thành phần | **User journey** đầu-cuối |
| **Tốc độ / độ ổn định** | Nhanh vừa, ổn định hơn | **Chậm, dễ flaky, đắt** |
| **Số lượng (pyramid)** | Vừa (tầng giữa) | **Ít** (đỉnh) |

**Ví dụ cùng chức năng "đăng nhập":**

```js
// INTEGRATION — gọi thẳng API + DB thật, KHÔNG qua trình duyệt; mock service ngoài nếu có
it('POST /login đúng mật khẩu thì trả token', async () => {
  const res = await request(app).post('/login')
    .send({ email: 'a@x.com', password: '123456' });   // đi qua controller→service→DB thật
  expect(res.body.accessToken).toBeDefined();          // quan tâm: các tầng ghép nối đúng
});

// E2E — điều khiển TRÌNH DUYỆT THẬT như người dùng, cả hành trình
test('user đăng nhập rồi vào được dashboard', async ({ page }) => {
  await page.goto('https://app.test/login');
  await page.fill('#email', 'a@x.com');
  await page.fill('#password', '123456');
  await page.click('button[type=submit]');             // thao tác như người thật
  await expect(page).toHaveURL(/dashboard/);           // quan tâm: cả user journey thông suốt
});
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Cùng trục (phạm vi), khác độ rộng:** integration ghép **vài thành phần** (một lát cắt); e2e ghép
  **toàn bộ** hệ thống. Trong pyramid: integration ở **giữa**, e2e ở **đỉnh**.
- **Integration:** vào bằng **gọi code/API nội bộ** (không qua UI), DB thật + **mock service ngoài**,
  trọng tâm **điểm ghép nối** — nhanh, ổn định hơn.
- **E2E:** vào bằng **UI/điểm vào thật** như người dùng, môi trường **giống production**, gần như **mọi thứ
  thật**, trọng tâm **cả user journey** — chậm, dễ **flaky**, đắt.
- **Quan hệ:** **e2e là integration ở mức rộng nhất + mô phỏng người dùng đầu-cuối**; integration không
  phải e2e vì chỉ ghép một phần và không đi qua UI thật.

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Testing Pyramid (kim tự tháp test) là gì?

### 📖 Giới thiệu kiến thức

**Testing Pyramid** (Mike Cohn đề xuất) là **mô hình chiến lược** khuyên nên **phân bổ số lượng test theo
tầng** như một **kim tự tháp**: càng lên cao **càng ít test**. Nó trả lời câu hỏi *"nên viết bao nhiêu test
mỗi loại?"* — chứ không định nghĩa từng loại (đã có ở [câu 1–4](#q1)).

```
        ▲  E2E        ← ÍT nhất — chậm, đắt, dễ flaky; chỉ phủ critical path
       ▲▲  Integration ← VỪA — kiểm điểm ghép nối giữa các thành phần
      ▲▲▲  Unit        ← NHIỀU nhất — nhanh, rẻ, ổn định; nền của kim tự tháp
```

**Vì sao hình kim tự tháp (đáy rộng, đỉnh hẹp)?** Đi từ đáy lên đỉnh, mỗi tầng **chậm hơn, đắt hơn, khó bảo
trì & dễ flaky hơn** nhưng **giống thực tế hơn**. Nên đặt **nhiều test ở tầng nhanh/rẻ (unit)** để phản hồi
lỗi tức thì, và **ít test ở tầng chậm/đắt (e2e)** chỉ cho các **luồng quan trọng** → vừa **bao phủ tốt** vừa
**chạy nhanh, ổn định, dễ bảo trì**.

**Các phản mẫu (anti-pattern) hay hỏi:**
- **Ice-cream cone (kim tự tháp ngược):** nhiều E2E/manual test, ít unit → bộ test **chậm, giòn (brittle),
  khó bảo trì**. Đây là "mùi" tổ chức thiếu unit test.
- **Testing hourglass (đồng hồ cát):** nhiều unit + nhiều e2e nhưng **thiếu integration** ở giữa → bỏ sót
  **bug ghép nối** giữa các thành phần.

**Biến thể hiện đại — Testing Trophy** (Kent C. Dodds, phổ biến ở frontend/JS): đề cao **integration test**
thay vì unit làm tầng "đáng đồng tiền" nhất → chi tiết & so sánh ở [5.1](#q5-1).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Testing Pyramid** = mô hình **phân bổ số lượng test theo tầng**: **nhiều unit (đáy) → vừa integration →
  ít e2e (đỉnh)**.
- **Lý do:** lên cao thì **chậm/đắt/flaky hơn nhưng thực tế hơn** → dồn test vào tầng **nhanh & rẻ** để phản
  hồi nhanh, chỉ để **ít e2e** cho **critical path**.
- **Mục tiêu:** bộ test **bao phủ tốt + chạy nhanh + ổn định + dễ bảo trì**.
- **Anti-pattern:** **ice-cream cone** (nhiều e2e/manual, ít unit → chậm, giòn); **hourglass** (thiếu
  integration → sót bug ghép nối).
- **Biến thể:** **Testing Trophy** (Kent C. Dodds) đề cao **integration** — cùng tinh thần "ưu tiên tầng
  giá-trị-cao, chi-phí-thấp" ([5.1](#q5-1)).

[↑ Mục lục](#mục-lục)

<a id="q5-1"></a>
### 5.1. Testing Trophy (Kent C. Dodds) & so với Pyramid

#### 📖 Giới thiệu kiến thức

**Testing Trophy** là mô hình phân bổ test do **Kent C. Dodds** đề xuất (~2018), rất phổ biến trong
**frontend/JS** (gắn với thư viện **Testing Library**). Hình dạng một **chiếc cúp**, 4 tầng từ dưới lên:

```
        ▲   E2E         ← ít — cả user flow thật
      ▟███▙  Integration ← PHÌNH TO NHẤT — nhiều unit ghép lại, "đáng đồng tiền"
       ██   Unit         ← hàm/đơn vị lẻ
       ▀▀   Static       ← đáy: TypeScript, ESLint — bắt lỗi typo/kiểu KHÔNG cần chạy
```

**Hai khác biệt lớn so với [Pyramid](#q5):**
1. **Thêm tầng Static** ở đáy — **lint + type check** bắt lỗi mà không cần chạy test (pyramid không có).
2. **Integration là tầng lớn nhất** (không phải unit). Câu thần chú của Kent:
   ***"Write tests. Not too many. Mostly integration."*** (Viết test. Đừng quá nhiều. Chủ yếu là integration.)

**Vì sao đề cao integration?** Dựa trên nguyên tắc:
> *"The more your tests resemble the way your software is used, the more confidence they can give you."*
> (Test càng giống cách người dùng thật sử dụng, càng cho bạn nhiều **độ tin cậy**.)

- **Unit + mock quá nhiều** dễ **pass giả**: hàm lẻ đúng nhưng **ghép lại vẫn hỏng**; lại **bám chặt
  implementation** → refactor một chút là test đỏ.
- **Integration** kiểm nhiều đơn vị **ghép thật**, ít mock → **bắt bug thật hơn**, **ít giòn**, refactor
  thoải mái. Nhờ tooling hiện đại (**jsdom, Testing Library, MSW**) mà integration **đủ nhanh**, làm mất
  giả định cũ "integration chậm" của pyramid → **dám viết nhiều integration hơn**.

| Tiêu chí | **Pyramid** (Cohn) | **Testing Trophy** (Dodds) |
|---|---|---|
| **Tầng nhiều nhất** | **Unit** (đáy rộng) | **Integration** (giữa phình to) |
| **Có tầng Static?** | Không | **Có** (lint/type ở đáy) |
| **Triết lý** | Nhiều unit nhanh & rẻ | *"Mostly integration"* — ROI/độ tin cậy cao nhất |
| **Bối cảnh** | Backend truyền thống | **Frontend/JS hiện đại** (Testing Library) |
| **Giả định về integration** | Chậm → hạn chế | Đủ nhanh (jsdom/MSW) → ưu tiên |

**Không mâu thuẫn nhau:** cả hai cùng thông điệp gốc **"dồn test vào tầng cho nhiều giá trị với chi phí
thấp, hạn chế tầng chậm-đắt (e2e)"**. Trophy chỉ **đánh giá lại** đâu là tầng đó cho **web app hiện đại**
(integration) và **thêm static analysis**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Testing Trophy** (Kent C. Dodds) = biến thể hiện đại của Pyramid, phổ biến **frontend/JS**; 4 tầng:
  **Static → Unit → Integration (lớn nhất) → E2E**.
- **Khác Pyramid ở 2 điểm:** (1) **thêm tầng Static** (TypeScript/ESLint, bắt lỗi không cần chạy); (2)
  **Integration là tầng lớn nhất** thay vì unit → *"Write tests. Not too many. Mostly integration."*.
- **Lý do:** *"test càng giống cách dùng thật, càng nhiều độ tin cậy"* — integration **ít mock, ít giòn,
  bắt bug thật hơn**; tooling mới (jsdom/Testing Library/MSW) làm integration **đủ nhanh**.
- **Chốt:** cùng tinh thần với Pyramid (ưu tiên tầng giá-trị-cao/chi-phí-thấp), chỉ **đổi trọng tâm sang
  integration** cho web app hiện đại + **thêm static**.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Phân biệt smoke test và regression test

### 📖 Giới thiệu kiến thức

Đây là **phân loại test theo *mục đích & thời điểm chạy*** (khác trục với unit/integration/e2e vốn theo
*phạm vi*). Một bộ smoke hay regression **có thể gồm nhiều loại** unit/integration/e2e — điểm khác là **chạy
để làm gì**.

**1) Smoke test — "cổng kiểm tra nhanh sau build"**

- Kiểm **nông & rộng** các **chức năng cốt lõi nhất** (đăng nhập được, trang chủ load, tạo được đơn...) để
  xác nhận **build có đủ ổn định để test tiếp** hay không. Còn gọi **Build Verification Test (BVT)**.
- **Chạy đầu tiên**, ngay **sau mỗi build/deploy mới**, **trước** khi test sâu.
- **Fail → loại build ngay**, khỏi tốn công test tiếp. Nhanh, thường tự động trong CI như một **gate**.
- *Nguồn tên:* phần cứng — cắm điện lên, nếu **bốc khói** thì hỏng, khỏi thử tiếp.

**2) Regression test — "lưới an toàn chống hồi quy"**

- **Chạy lại** các test case **hiện có** sau khi **thay đổi code** (fix bug, thêm feature, refactor) để chắc
  thay đổi **không làm hỏng chức năng đang chạy tốt**. *"Regression" = thứ đang chạy đúng bỗng hỏng lại.*
- Kiểm **sâu & bao quát** toàn bộ chức năng cũ → bộ test **lớn, chậm** hơn, thường **tự động hóa**; có thể
  chạy **selective regression** (chỉ vùng bị ảnh hưởng) cho nhanh.
- **Chạy sau khi có thay đổi**, và **trước khi release**.

| Tiêu chí | **Smoke test** | **Regression test** |
|---|---|---|
| **Mục đích** | Build có **đủ ổn định để test tiếp**? | Thay đổi mới có **làm hỏng chức năng cũ**? |
| **Độ sâu/rộng** | **Nông & rộng** (critical path) | **Sâu & bao quát** (toàn bộ chức năng cũ) |
| **Khi chạy** | **Ngay sau build**, trước khi test sâu | **Sau khi có thay đổi**, trước release |
| **Quy mô** | Ít test, chạy **nhanh** | Nhiều test, **chậm/lớn** |
| **Vai trò** | **Gate** (cổng chặn build hỏng) | **Safety net** (chống hồi quy) |
| **Khi fail** | **Loại build** ngay | Có **regression bug** cần sửa |

**Dễ nhầm — smoke vs sanity:** **sanity test** kiểm **hẹp & sâu** một chức năng cụ thể vừa sửa (sau minor
change) để xem "có hợp lý không"; còn **smoke** kiểm **rộng & nông** toàn hệ thống sau build mới. Smoke là
**gate rộng**, sanity là **kiểm điểm hẹp**.

> Quan hệ với [Testing Pyramid](#q5): smoke thường là **một tập con nhỏ** các test critical (rút từ mọi
> tầng) chạy trước; regression là **toàn bộ** bộ test tích lũy chạy lại — càng nhiều test tự động ở đáy
> pyramid thì regression càng rẻ và đáng tin.

### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai phân loại theo **mục đích chạy** (không phải phạm vi) — một bộ có thể gồm cả unit/integration/e2e.
- **Smoke test:** kiểm **nông & rộng** vài **chức năng cốt lõi** **ngay sau build mới**, để quyết định
  *"build có đáng test tiếp không"* → fail thì **loại build ngay**. Là **gate**, nhanh. (≈ Build Verification)
- **Regression test:** **chạy lại test cũ** sau **thay đổi code** để chắc **không làm hỏng chức năng đang
  chạy tốt** → **sâu, bao quát, chậm**, thường tự động. Là **lưới an toàn** chống hồi quy.
- **Chốt:** smoke = *"còn sống không?"* (rộng-nông, trước), regression = *"có làm hỏng cái cũ không?"*
  (sâu-rộng, sau thay đổi). Đừng nhầm smoke với **sanity** (hẹp-sâu, kiểm 1 chức năng vừa sửa).

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. Test double (đối tượng giả thay dependency thật) là gì?

### 📖 Giới thiệu kiến thức

Khi [unit test](#q1) một hàm, ta muốn kiểm **riêng logic của nó**, không muốn gọi **dependency thật** (DB,
API, gửi mail, service khác) vì chúng **chậm, không ổn định, có side-effect thật** (gửi mail thật, ghi DB
thật) và **khó điều khiển**. Giải pháp: thay dependency thật bằng một **đối tượng giả** có cùng "hình dạng"
(interface) để hàm vẫn chạy được trong môi trường **cô lập, nhanh, kiểm soát được**.

**Test Double** là **thuật ngữ chung (umbrella)** cho **mọi đối tượng giả** đó (Gerard Meszaros đặt ra, ví
như "diễn viên đóng thế" trong phim). Nó **không phải một kỹ thuật cụ thể**, mà là *danh mục* gồm **5 loại**,
phân theo **mức độ thông minh** và **mục đích dùng**:

| Loại | Là gì | Dùng để |
|---|---|---|
| **Dummy** | Đối tượng **rỗng**, chỉ để **bù chỗ** tham số, **không bao giờ dùng thật** | Điền cho đủ chữ ký hàm |
| **Stub** | Trả **câu trả lời định sẵn** cho lời gọi | Điều khiển **đầu vào gián tiếp** → state verification |
| **Spy** | Stub **có ghi lại** cách bị gọi | Assert tương tác **sau khi chạy** |
| **Mock** | Được **đặt kỳ vọng trước**, **tự verify** | **Behavior verification** |
| **Fake** | **Bản cài đặt rút gọn** nhưng chạy được (vd DB in-memory, mảng thay repo) | Thay hạ tầng nặng bằng bản nhẹ |

**Phân biệt nhanh 2 nhánh hay lẫn:**
- **Fake vs các loại khác:** fake **có logic thật** (một SQLite `:memory:` vẫn lưu/truy vấn đúng), còn
  stub/mock chỉ **trả lời cứng/theo kịch bản**, không có logic.
- **Ba loại "thông minh" (stub/spy/mock)** khác nhau ở **verify kiểu gì** → xem chi tiết ở [7.1](#q7-1).

> Lưu ý thuật ngữ: trong đời thường (và nhiều framework như Jest), người ta gọi gộp tất cả là **"mock"**.
> Nhưng đúng lý thuyết, **"mock" chỉ là 1 trong 5 loại** test double.

**Ví dụ — thay dependency thật bằng test double:** `OrderService` phụ thuộc `PaymentGateway`; khi test ta
**không** muốn gọi cổng thanh toán thật (tốn tiền/tạo giao dịch thật, phụ thuộc mạng, chậm, kết quả không
ổn định) → thay bằng một test double.

```ts
class OrderService {
  constructor(private paymentGateway: PaymentGateway) {}
  checkout(amount: number) {
    return this.paymentGateway.charge(amount);
  }
}

// test double thay cho PaymentGateway thật
const fakePaymentGateway = {
  charge: () => ({ success: true, transactionId: 'TEST-001' }),
};
const orderService = new OrderService(fakePaymentGateway);   // inject bản giả qua constructor

const result = orderService.checkout(500_000);
expect(result.success).toBe(true);
// PaymentGateway = dependency thật · fakePaymentGateway = test double
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Test double** = **thuật ngữ chung** cho **mọi đối tượng giả** thay dependency thật khi test, để test
  **cô lập, nhanh, kiểm soát được**, tránh side-effect thật (mail/DB...).
- Gồm **5 loại**: **dummy** (bù chỗ, không dùng) · **stub** (trả lời định sẵn) · **spy** (stub + ghi lại) ·
  **mock** (đặt kỳ vọng, tự verify) · **fake** (bản cài đặt rút gọn chạy được, vd DB in-memory).
- **Fake** khác phần còn lại ở chỗ **có logic thật**; **stub/spy/mock** chỉ trả lời cứng/theo kịch bản.
- Đời thường hay gọi gộp là **"mock"**, nhưng đúng lý thuyết mock chỉ là **1 trong 5 loại**. Ba loại hay hỏi
  **stub/spy/mock** khác nhau ở **verify kiểu gì** → [7.1](#q7-1).

[↑ Mục lục](#mục-lục)

<a id="q7-1"></a>
### 7.1. Phân biệt mock, stub và spy

#### 📖 Giới thiệu kiến thức

Ba loại [test double](#q7) hay bị lẫn nhất. Điểm mấu chốt phân biệt nằm ở **verify kiểu gì**:
- **State verification** — kiểm tra **kết quả/trạng thái** đầu ra sau khi chạy.
- **Behavior verification** — kiểm tra **cách các dependency bị gọi** (gọi hàm nào, mấy lần, tham số gì).

**1) Stub — "trả lời định sẵn"**

- Cung cấp **câu trả lời cố định (canned answer)** cho lời gọi trong test → điều khiển **đầu vào gián tiếp**
  (indirect input) cho hàm đang test.
- **Không** assert việc nó bị gọi. Phục vụ **state verification**: *"cho input này thì output đúng chưa?"*.
- Câu thần chú: **"stubs enable"** (giúp hàm chạy được).

**2) Spy — "stub có ghi âm"**

- Là **stub + ghi lại** cách nó bị gọi (số lần, tham số...) để **assert *sau khi* chạy** (act → then assert).
- Có thể **bọc (wrap) object thật** và vẫn gọi thật trong khi ghi lại (vd `jest.spyOn`, Mockito `spy()`).
- Verify tương tác nhưng theo kiểu **"chạy xong rồi mới kiểm"**.

**3) Mock — "đặt kỳ vọng trước, tự verify"**

- Được **lập trình kỳ vọng (expectation) trước**: *"phải gọi `sendMail()` đúng 1 lần với email X"*. Nếu
  không đúng → **test fail**. Phục vụ **behavior verification**.
- Câu thần chú: **"mocks verify"** (kiểm tra tương tác).

| Tiêu chí | **Stub** | **Spy** | **Mock** |
|---|---|---|---|
| **Mục đích** | Trả lời định sẵn | Stub + **ghi lại** lời gọi | Đặt **kỳ vọng** + tự verify |
| **Kiểu verify** | State (đầu ra) | Behavior (assert **sau**) | Behavior (kỳ vọng **trước**) |
| **Có assert tương tác?** | Không | Có — sau khi chạy | Có — fail tự động nếu sai kỳ vọng |
| **Gọi code thật?** | Không | Có thể (wrap) | Không |
| **Thần chú** | *stubs enable* | *record then assert* | *mocks verify* |

**Thực tế (Jest/Sinon/Mockito):** ranh giới **mờ dần** vì một object thường kiêm nhiều vai. `jest.fn()` vừa
là spy (ghi lại `.mock.calls`) vừa stub được (`mockReturnValue`); Sinon tách rõ `stub()`, `spy()`, `mock()`.

```js
// STUB — ép trả về giá trị cố định để test nhánh logic (không quan tâm bị gọi mấy lần)
const repo = { getPrice: jest.fn().mockReturnValue(100) };   // canned answer
expect(calcTotal(repo, 2)).toBe(200);                        // state verification

// SPY — để hàm chạy thật/giả rồi KIỂM SAU xem có gọi đúng không
const logger = { warn: jest.fn() };
processOrder(logger, badOrder);
expect(logger.warn).toHaveBeenCalledTimes(1);                // assert sau khi chạy
expect(logger.warn).toHaveBeenCalledWith('đơn không hợp lệ');

// MOCK — đặt KỲ VỌNG trước bằng thư viện expect-style (Sinon)
const mail = sinon.mock(mailer);
mail.expects('send').once().withArgs('a@x.com');             // kỳ vọng trước
placeOrder(mailer, 'a@x.com');
mail.verify();                                               // sai kỳ vọng → fail
```

**Ví dụ cụ thể theo constructor injection (Jest):**

```ts
// 1) STUB — cấu hình method trả về kết quả định trước; chỉ quan tâm SUT xử lý kết quả thế nào
const userRepositoryStub = {
  findById: () => ({ id: 1, name: 'Cuom', active: true }),   // trả dữ liệu giả
};
const service = new UserService(userRepositoryStub);
expect(service.getUserName(1)).toBe('Cuom');                 // KHÔNG quan tâm findById gọi mấy lần

// 2) MOCK — xác minh dependency được GỌI ĐÚNG CÁCH (số lần, tham số)
const emailServiceMock = { sendWelcomeEmail: jest.fn() };
const register = new RegisterService(emailServiceMock);
register.register({ email: 'cuom@example.com' });
expect(emailServiceMock.sendWelcomeEmail).toHaveBeenCalledTimes(1);
expect(emailServiceMock.sendWelcomeEmail).toHaveBeenCalledWith('cuom@example.com');

// 3) SPY — bọc method THẬT để ghi nhận cách nó bị gọi (mặc định vẫn chạy thật)
const emailService = new EmailService();
const sendSpy = jest.spyOn(emailService, 'sendWelcomeEmail')
  .mockImplementation(() => undefined);                      // chặn gửi mail thật (tùy chọn)
new RegisterService(emailService).register({ email: 'cuom@example.com' });
expect(sendSpy).toHaveBeenCalledWith('cuom@example.com');
```

**Cách nhớ nhanh (một câu mỗi loại):**
- **Stub:** *"Khi gọi thì hãy **trả về** dữ liệu này."*
- **Mock:** *"Tôi **mong** method này phải được gọi như thế này."*
- **Spy:** *"Hãy **theo dõi** xem method này được gọi ra sao."*

**Ví dụ thực tế — một chức năng `checkout(order)` dùng cả 3:**
- **Stub** cổng thanh toán để nó trả về `"SUCCESS"` (điều khiển đầu vào cho nhánh cần test).
- **Mock** để kiểm `paymentGateway.charge(500000)` **được gọi đúng 1 lần** (kiểm hành vi/side-effect).
- **Spy** để theo dõi `logger.info()` (hoặc một method thật) mà vẫn giữ behavior thật.

> Nhắc lại: trong Jest, **`jest.fn()` vừa đóng vai stub vừa vai mock** — tên gọi tùy **cách dùng** (cấu hình
> `mockReturnValue` = stub; assert `toHaveBeenCalled*` = mock), không phụ thuộc API tạo đối tượng.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả 3 đều là **test double** (đối tượng giả thay dependency thật khi test). Khác nhau ở **verify kiểu gì**.
- **Stub:** trả **câu trả lời định sẵn** để hàm chạy được → **state verification** (kiểm đầu ra), **không**
  quan tâm bị gọi ra sao. *"stubs enable"*.
- **Spy:** **stub có ghi lại** lời gọi (số lần, tham số) → **assert sau khi chạy** (act → then assert); có
  thể **bọc object thật**. *"record then assert"*.
- **Mock:** **đặt kỳ vọng trước** về cách bị gọi, **tự fail** nếu sai → **behavior verification**.
  *"mocks verify"*.
- **Chốt:** **stub = trạng thái**, **mock/spy = hành vi**; khác nhau ở chỗ **mock kỳ vọng trước**, **spy ghi
  lại rồi kiểm sau**. Thực tế (Jest/Sinon) ranh giới mờ vì 1 object kiêm nhiều vai.

[↑ Mục lục](#mục-lục)

<a id="q7-2"></a>
### 7.2. Mục đích của mock/stub/spy & tại sao phải tách?

#### 📖 Giới thiệu kiến thức

**Mục đích chung (cả 3):** đều là [test double](#q7) để **cô lập SUT khỏi dependency thật** → test **nhanh,
ổn định, không side-effect thật** (không gửi mail/ghi DB thật), **kiểm soát được** cả những tình huống **khó
dựng thật** (mạng lỗi, timeout, dependency trả về giá trị hiếm).

**Mục đích riêng — chia theo *kiểm cái gì*:**

| Loại | Phục vụ | Trả lời câu hỏi | Kiểm chứng |
|---|---|---|---|
| **Stub** | Điều khiển **đầu vào gián tiếp** (indirect input) | *"Khi dependency trả về X thì SUT xử lý đúng chưa?"* | **State** — kiểm **kết quả** SUT |
| **Mock / Spy** | Kiểm **đầu ra gián tiếp** (indirect output) | *"SUT có **gọi đúng** dependency không?"* | **Behavior** — kiểm **tương tác** |

- **Stub** giúp SUT **chạy được đường mình muốn test** (kể cả nhánh khó: ép throw error, trả `null`, trả
  danh sách rỗng) rồi kiểm **giá trị SUT sinh ra**.
- **Mock/Spy** kiểm những **side-effect không nhìn thấy qua return value** — vd SUT có thật sự gọi
  `mailer.send()` / `logger.warn()` / `repo.save()` đúng tham số, đúng số lần không.

**Tại sao phải tách (3 lý do cốt lõi):**

1. **Khác thứ cần assert → tránh test sai trọng tâm.** Stub = state verification (**không** assert lời gọi);
   mock/spy = behavior verification (**có** assert lời gọi). Gọi đúng tên loại giúp biết test này **quan tâm
   cái gì**.

2. **Tránh over-specification → test bớt giòn (brittle).** Nếu dùng **mock** (kỳ vọng lời gọi) ở chỗ chỉ cần
   **stub**, test sẽ **bám chặt implementation**: chỉ cần đổi *cách* SUT gọi nội bộ (dù kết quả vẫn đúng) là
   test **đỏ oan** → cản refactor. Ngược lại, cần kiểm side-effect (gửi mail) mà chỉ **stub** → **bỏ sót bug**.
   Tách rõ giúp **chỉ verify đúng cái cần**.

3. **Diễn đạt ý định (intent) cho người đọc.** Tên loại là tài liệu sống: *stub* nói "tôi chỉ cần nó trả về
   X"; *mock/spy* nói "tôi cần chắc nó **được gọi**". Đọc test hiểu ngay mục tiêu.

**Nguyên tắc vàng — "Stub queries, mock commands"** (theo Command–Query Separation):
- **Query** (hàm **trả dữ liệu**, không đổi trạng thái, vd `repo.find()`) → dùng **stub**: chỉ cần giá trị
  trả về, **đừng** assert nó được gọi (gọi mấy lần là chuyện nội bộ).
- **Command** (hàm **gây side-effect**, không trả gì ý nghĩa, vd `mailer.send()`, `repo.save()`) → dùng
  **mock/spy**: **phải** assert nó được gọi đúng, vì đó là **hành vi quan trọng** không thấy qua kết quả.

> Nhờ nguyên tắc này, bạn tránh được lỗi phổ biến **"mock mọi thứ"**: mock cả các query → test giòn, khó đọc.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Mục đích chung:** cô lập SUT khỏi dependency thật → test **nhanh, ổn định, không side-effect**, kiểm
  soát được cả case khó dựng (lỗi, timeout).
- **Mục đích riêng:** **Stub** điều khiển **đầu vào** (ép dependency trả giá trị để test nhánh logic) → kiểm
  **kết quả** (state); **Mock/Spy** kiểm **đầu ra/tương tác** (SUT có **gọi đúng** dependency không) →
  behavior.
- **Tại sao tách:** (1) khác **thứ cần assert** nên tránh test sai trọng tâm; (2) tránh **over-specification**
  → test bớt **giòn**, không cản refactor (dùng mock chỗ chỉ cần stub → test đỏ oan; chỉ stub chỗ cần verify
  side-effect → sót bug); (3) **diễn đạt ý định** rõ cho người đọc.
- **Quy tắc thực dụng:** ***"Stub queries, mock commands"*** — hàm **truy vấn** thì stub (đừng assert lời
  gọi), hàm **gây side-effect** thì mock/spy (phải assert). Tránh **"mock mọi thứ"**.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. AAA pattern (Arrange–Act–Assert) là gì?

### 📖 Giới thiệu kiến thức

**AAA (Arrange–Act–Assert)** là mẫu **cấu trúc thân của một test case** thành **3 giai đoạn rõ ràng**, giúp
test **dễ đọc, dễ bảo trì** và mỗi test tập trung vào **một hành vi**:

1. **Arrange (Chuẩn bị):** dựng bối cảnh — khởi tạo đối tượng cần test (SUT), chuẩn bị dữ liệu đầu vào,
   set up **[test double](#q7)** (mock/stub) cho dependency.
2. **Act (Hành động):** thực thi **đúng một** hành vi cần kiểm — gọi hàm/method đang test, lấy kết quả.
3. **Assert (Kiểm chứng):** so kết quả/trạng thái thực tế với **kỳ vọng** (`expect(...)`).

Nhiều dev còn tách rành mạch bằng **comment `// Arrange // Act // Assert`** hoặc **dòng trống** giữa 3 khối.

**Vì sao nên theo AAA?**
- **Dễ đọc:** người khác nhìn phát biết "chuẩn bị gì → làm gì → mong đợi gì".
- **Một test một việc:** nếu thấy **2 khối Act** → dấu hiệu test đang ôm quá nhiều, nên tách.
- **Giảm bug trong chính test:** không lẫn lộn setup với assertion.

```js
it('cộng thuế 10% vào giá', () => {
  // Arrange — chuẩn bị SUT + dữ liệu
  const cart = new Cart();
  cart.add({ price: 100 });

  // Act — thực thi ĐÚNG MỘT hành vi
  const total = cart.totalWithTax(0.1);

  // Assert — kiểm chứng kỳ vọng
  expect(total).toBe(110);
});
```

**Liên hệ Given–When–Then (BDD):** AAA chính là bản "kỹ thuật" của **Given–When–Then** trong [BDD](#q9):
**Given ≈ Arrange · When ≈ Act · Then ≈ Assert**. Khác biệt: GWT viết bằng **ngôn ngữ nghiệp vụ** cho người
không kỹ thuật; AAA là **khung code** cho dev.

**Biến thể liên quan — "4 phase test"** (Gerard Meszaros): thêm pha **Teardown** (dọn dẹp sau test, vd đóng
kết nối/xóa data) → **Setup → Exercise → Verify → Teardown**. AAA là bản gọn phổ biến nhất (teardown thường
để `afterEach` lo).

### 🎯 Tóm tắt trả lời phỏng vấn

- **AAA (Arrange–Act–Assert)** = mẫu chia thân một test thành **3 pha**: **Arrange** (dựng SUT + data +
  mock/stub) → **Act** (thực thi **đúng 1** hành vi) → **Assert** (so kết quả với kỳ vọng).
- **Lợi ích:** test **dễ đọc, một test một việc, dễ bảo trì**; thấy **2 khối Act** là dấu hiệu nên tách test.
- **Liên hệ BDD:** trùng ý với **Given–When–Then** (Given≈Arrange, When≈Act, Then≈Assert) — GWT là bản
  **ngôn ngữ nghiệp vụ**, AAA là bản **khung code**.
- **Mở rộng:** thêm pha **Teardown** (dọn dẹp) → 4-phase test; thường giao cho `afterEach`.

[↑ Mục lục](#mục-lục)

<a id="q9"></a>
## 9. Phân biệt TDD và BDD

### 📖 Giới thiệu kiến thức

Cả hai đều là **phương pháp phát triển "test trước code sau"**, nhưng khác nhau ở **góc nhìn** và **ngôn
ngữ mô tả**: TDD viết test ở góc **kỹ thuật (developer)**, BDD mô tả **hành vi ở góc nghiệp vụ (business)**.
BDD thực chất là **TDD được nâng lên** để tập trung vào *hành vi* và dùng *ngôn ngữ chung* cho cả người
không kỹ thuật.

**1) TDD (Test-Driven Development) — lái thiết kế bằng test**

- Chu trình **Red → Green → Refactor**: (1) viết **test fail** cho một mẩu chức năng nhỏ; (2) viết **code
  tối thiểu** cho pass; (3) **refactor** giữ test xanh.
- **Góc nhìn:** developer, kiểm ở mức **đơn vị code** (thường là [unit test](#q1)); thiết kế **nảy sinh**
  từ test.
- **Ngôn ngữ:** viết bằng **code assert** (`assertEquals`, `expect().toBe()`), chỉ dev đọc.
- **Trả lời câu hỏi:** *"code này chạy đúng chưa?"*.

**2) BDD (Behavior-Driven Development) — lái bằng hành vi nghiệp vụ**

- Mô tả **hành vi mong đợi** dưới dạng **kịch bản (scenario)** bằng **ngôn ngữ tự nhiên** theo mẫu
  **Given–When–Then** (cú pháp **Gherkin**) mà **người không kỹ thuật cũng đọc được**.
- Nhấn mạnh **cộng tác** giữa **dev – QA – business ("three amigos")**, dùng **ubiquitous language** (ngôn
  ngữ thống nhất) → thu hẹp khoảng cách giữa yêu cầu và code.
- **Bắt đầu từ:** một **tính năng/hành vi** (feature) thay vì một hàm.
- **Trả lời câu hỏi:** *"hệ thống có hành xử đúng như nghiệp vụ mong đợi không?"*.

| Tiêu chí | **TDD** | **BDD** |
|---|---|---|
| **Trọng tâm** | Đúng **đơn vị code** (implementation) | Đúng **hành vi** theo nghiệp vụ |
| **Góc nhìn** | Developer (kỹ thuật) | Dev + QA + business (three amigos) |
| **Ngôn ngữ mô tả** | Code assert | **Ngôn ngữ tự nhiên** (Given–When–Then/Gherkin) |
| **Đơn vị xuất phát** | 1 hàm/class | 1 feature/hành vi |
| **Ai đọc được** | Chỉ dev | Cả người **không kỹ thuật** |
| **Công cụ** | PHPUnit, JUnit, Jest | Cucumber, Behat (PHP), SpecFlow |

**Quan hệ:** không loại trừ nhau — **BDD "bọc" TDD**: mô tả hành vi bằng scenario (BDD) rồi vẫn dùng vòng
Red-Green-Refactor (TDD) để cài đặt. Nhiều người coi **BDD là "TDD làm đúng"**, chuyển câu hỏi từ *"test cái
gì"* sang *"hành vi nào cần có"*.

```php
// TDD — test kỹ thuật, chỉ dev đọc (PHPUnit)
public function test_chuyen_tien_tru_dung_so_du(): void
{
    $acc = new Account(100);
    $acc->transfer(30);
    $this->assertEquals(70, $acc->balance());   // "code đúng chưa?"
}
```

```gherkin
# BDD — kịch bản hành vi, business đọc được (Gherkin/Cucumber/Behat)
Feature: Chuyển tiền
  Scenario: Chuyển tiền khi đủ số dư
    Given tài khoản có số dư 100
    When tôi chuyển 30
    Then số dư còn lại phải là 70          # "hệ thống hành xử đúng không?"
```

> Lưu ý: cú pháp **`describe()/it('should...')`** trong Jest/Mocha/RSpec là **spec-style** lấy cảm hứng BDD
> (mô tả hành vi bằng câu), nhưng **BDD đầy đủ** còn gồm cả **cộng tác + Gherkin**, không chỉ là cách đặt tên test.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Giống:** đều **test-first**, viết kỳ vọng trước rồi mới code.
- **TDD:** góc **developer**, lái **thiết kế đơn vị code** bằng vòng **Red–Green–Refactor**, viết bằng **code
  assert** → *"code chạy đúng chưa?"*.
- **BDD:** góc **nghiệp vụ**, mô tả **hành vi** bằng **ngôn ngữ tự nhiên Given–When–Then (Gherkin)** cho cả
  người **không kỹ thuật**, nhấn mạnh **cộng tác dev–QA–business** → *"hệ thống hành xử đúng mong đợi không?"*.
- **Công cụ:** TDD → PHPUnit/JUnit/Jest; BDD → **Cucumber/Behat/SpecFlow**.
- **Quan hệ:** **BDD là TDD tập trung vào hành vi + ngôn ngữ chung**; BDD có thể **bọc** TDD (scenario ngoài,
  Red-Green-Refactor bên trong).

[↑ Mục lục](#mục-lục)

<a id="q10"></a>
## 10. Testing trong Laravel (trình bày ngắn gọn)

### 📖 Giới thiệu kiến thức

Laravel **tích hợp sẵn testing** dựa trên **PHPUnit** (từ Laravel 11 có thể chọn **Pest** khi cài). Cấu hình
ở `phpunit.xml`, env test lấy từ `.env.testing` (thường đặt `APP_ENV=testing`, DB **SQLite `:memory:`** cho nhanh).

**1) Hai loại test — 2 thư mục:**

| Thư mục | Kế thừa | Có boot Laravel? | Dùng cho |
|---|---|---|---|
| `tests/Unit` | `PHPUnit\...\TestCase` | **Không** | Test **1 class/hàm cô lập** (logic thuần), rất nhanh |
| `tests/Feature` | `Tests\TestCase` | **Có** (full app) | Test qua **HTTP/route, DB, service** — phổ biến nhất |

> Đây chính là [unit vs functional test](#q1) áp vào Laravel: `Feature` test là dạng
> **functional/integration** đi qua nhiều tầng thật.

**2) Lệnh & tạo test:**

```bash
php artisan test               # chạy toàn bộ (đẹp hơn ./vendor/bin/phpunit)
php artisan test --filter=Login   # lọc theo tên
php artisan make:test LoginTest          # tạo Feature test
php artisan make:test SumTest --unit     # tạo Unit test
```

**3) HTTP test + assertion:** `$this->get/post/getJson(...)` rồi assert `assertStatus/assertOk/assertRedirect/
assertSee/assertJson/assertJsonFragment`.

**4) Database test:** trait **`RefreshDatabase`** (migrate rồi rollback trong transaction, mỗi test DB sạch);
assert `assertDatabaseHas/assertDatabaseMissing/assertModelExists`. Dữ liệu mẫu bằng **model factory**
(`User::factory()->create()`).

**5) Auth & Fake:** `actingAs($user)` để giả đăng nhập; **fake các service ngoài** để không gọi thật:
`Mail::fake()`, `Queue::fake()`, `Event::fake()`, `Notification::fake()`, `Storage::fake()`, `Http::fake()`.
Mock class bất kỳ: `$this->mock(Service::class)` (dùng Mockery).

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class OrderTest extends TestCase
{
    use RefreshDatabase;                        // reset DB mỗi test

    public function test_user_dat_hang_thanh_cong(): void
    {
        Mail::fake();                           // chặn gửi mail thật
        $user = User::factory()->create();      // seed dữ liệu

        $res = $this->actingAs($user)           // giả đăng nhập
            ->postJson('/orders', ['product_id' => 1, 'qty' => 2]);

        $res->assertStatus(201)                 // kiểm HTTP response
            ->assertJsonFragment(['status' => 'pending']);
        $this->assertDatabaseHas('orders', [    // kiểm DB đã ghi
            'user_id' => $user->id, 'qty' => 2,
        ]);
        Mail::assertSent(OrderPlaced::class);   // kiểm side-effect
    }
}
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Nền tảng:** Laravel tích hợp sẵn testing trên **PHPUnit** (Laravel 11+ có tùy chọn **Pest**), chạy bằng
  **`php artisan test`**; env test ở `.env.testing`, thường **SQLite `:memory:`**.
- **2 loại:** `tests/Unit` (**không boot** app — test class/hàm cô lập, nhanh) và `tests/Feature` (**boot full
  app** — test qua HTTP/route/DB, dạng functional/integration, phổ biến nhất).
- **DB test:** trait **`RefreshDatabase`** giữ DB sạch mỗi test; **factory** tạo dữ liệu; assert
  `assertDatabaseHas/Missing`.
- **HTTP:** `$this->getJson/postJson(...)` + `assertStatus/assertJson...`; **`actingAs()`** để giả đăng nhập.
- **Cô lập service ngoài:** **`Mail::fake()`, `Queue::fake()`, `Event::fake()`, `Http::fake()`**... để test
  side-effect mà không gọi thật.

[↑ Mục lục](#mục-lục)

<a id="q10-1"></a>
### 10.1. Triển khai mock, stub, spy trong Laravel

#### 📖 Giới thiệu kiến thức

Laravel dùng thư viện **Mockery** làm engine mocking, cộng với **container swap** (tráo binding trong service
container) và bộ **`::fake()`** dựng sẵn. Đây là cách hiện thực 3 loại [test double](#q7) đã học ở [7.1](#q7-1):

**1) STUB — trả lời định sẵn (`shouldReceive()->andReturn()`, không đặt kỳ vọng số lần):**

```php
// Stub qua container: DI resolve PaymentGateway sẽ nhận bản giả trả về cứng
$this->mock(PaymentGateway::class, function (MockInterface $mock) {
    $mock->shouldReceive('charge')->andReturn(true);   // canned answer, KHÔNG ->once()
});
$this->assertTrue(app(OrderService::class)->checkout($order));  // state verification

// Stub facade
Cache::shouldReceive('get')->andReturn('cached-value');

// Stub HTTP client: trả response cứng cho URL khớp
Http::fake(['api.xyz/*' => Http::response(['ok' => true], 200)]);
```

**2) MOCK — đặt kỳ vọng TRƯỚC, tự verify (`->once()`, `->with()`):**

```php
// $this->mock() tráo instance vào container; Mockery::close() ở teardown tự verify
$this->mock(Mailer::class, function (MockInterface $mock) {
    $mock->shouldReceive('send')
         ->once()                       // kỳ vọng: đúng 1 lần
         ->with('a@x.com')              // đúng tham số
         ->andReturn(true);             // sai kỳ vọng → test FAIL tự động
});
app(OrderService::class)->place($order, 'a@x.com');   // behavior verification
```

**3) SPY — ghi lại rồi kiểm SAU (`$this->spy()` + `shouldHaveReceived`):**

```php
// spy để code chạy, sau đó mới assert đã bị gọi thế nào (act → then assert)
$spy = $this->spy(Logger::class);
app(OrderService::class)->place($order);
$spy->shouldHaveReceived('warn')->once()->with('đơn trùng');   // kiểm sau

// Các ::fake() của Laravel chính là SPY dựng sẵn: chạy xong assert
Mail::fake();
app(OrderService::class)->place($order, 'a@x.com');
Mail::assertSent(OrderPlaced::class, fn ($m) => $m->hasTo('a@x.com'));   // verify sau
Bus::fake();      // ... Bus::assertDispatched(Job::class);
Event::fake();    // ... Event::assertDispatched(OrderPlaced::class);
```

**Phân biệt các API dễ lẫn:**
- **`$this->mock()`** — thay **hoàn toàn** object (mọi method không khai báo → lỗi nếu bị gọi).
- **`$this->partialMock()`** — chỉ giả **method khai báo**, còn lại **chạy thật**.
- **`$this->spy()`** — như mock nhưng theo hướng **ghi lại rồi kiểm sau** thay vì kỳ vọng trước.
- **`$this->instance(Abstract::class, $obj)`** — tự tay bind một instance (kể cả fake) vào container.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Laravel mock bằng **Mockery** + **tráo binding trong container** (`$this->mock/partialMock/spy/instance`)
  và bộ **`::fake()`** dựng sẵn.
- **Stub** = `shouldReceive('x')->andReturn($v)` **không** đặt số lần → trả lời cứng (state verification);
  HTTP stub = `Http::fake([...=>Http::response()])`.
- **Mock** = `$this->mock(Class, fn($m) => $m->shouldReceive('x')->once()->with(...))` → **kỳ vọng trước**,
  `Mockery::close()` tự verify, sai là **fail**.
- **Spy** = `$this->spy(Class)` rồi `shouldHaveReceived(...)`; hoặc các **`::fake()`** (`Mail/Bus/Event/
  Queue::fake()`) rồi `assertSent/assertDispatched(...)` → **ghi lại, kiểm sau**.
- Nhớ khác biệt: **`mock`** thay hết · **`partialMock`** giữ method còn lại chạy thật · **`spy`** kiểm sau.

[↑ Mục lục](#mục-lục)

<a id="q11"></a>
## 11. Testing trong NestJS (trình bày ngắn gọn)

### 📖 Giới thiệu kiến thức

NestJS mặc định test bằng **Jest** kèm package **`@nestjs/testing`**. File unit đặt `*.spec.ts` (cạnh source),
file E2E đặt `test/*.e2e-spec.ts`. Cấu hình Jest nằm trong `package.json` (hoặc `jest.config`).

```bash
npm run test          # unit + integration (*.spec.ts)
npm run test:watch    # chế độ watch
npm run test:cov      # đo coverage
npm run test:e2e      # E2E (test/*.e2e-spec.ts)
```

**Công cụ lõi — `Test.createTestingModule()`:** dựng một **module DI thu nhỏ** giống module thật, khai báo
`providers/controllers`, `.compile()` rồi lấy instance ra bằng `module.get<T>(Token)`. Nhờ **dependency
injection** (xem [nestjs.md](../Backend/nestjs.md)), ta **override provider** để tráo dependency bằng bản giả.

```ts
// UNIT TEST một service — mock repository, chỉ test logic service
describe('OrdersService', () => {
  let service: OrdersService;
  const repo = { create: jest.fn(), findOne: jest.fn() };   // bản giả

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: OrderRepository, useValue: repo },        // override bằng mock
      ],
    }).compile();
    service = moduleRef.get<OrdersService>(OrdersService);
  });

  it('tạo đơn gọi repo.create đúng 1 lần', async () => {
    repo.create.mockResolvedValue({ id: 1 });                // stub trả về
    await service.create({ total: 100 });
    expect(repo.create).toHaveBeenCalledTimes(1);            // spy: kiểm sau
  });
});
```

**Ba cách override provider:** `.overrideProvider(Token).useValue(obj)` (bản giả có sẵn) · `.useClass(FakeCls)`
· `.useFactory(...)`. Ngoài ra `.useMocker()` **tự động mock** mọi dependency chưa khai báo.

**E2E test:** dựng app thật bằng `createNestApplication()` rồi bắn HTTP thật qua **Supertest**:

```ts
const moduleRef = await Test.createTestingModule({ imports: [AppModule] }).compile();
app = moduleRef.createNestApplication();
await app.init();
return request(app.getHttpServer()).post('/orders').send({ total: 100 }).expect(201);
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Nền tảng:** NestJS test bằng **Jest** + **`@nestjs/testing`**; `*.spec.ts` (unit), `test/*.e2e-spec.ts`
  (E2E); chạy `npm run test / test:e2e / test:cov`.
- **Lõi:** **`Test.createTestingModule({...}).compile()`** dựng **module DI thu nhỏ**, lấy instance qua
  **`module.get(Token)`**.
- **Mock dependency:** **`overrideProvider(Token).useValue/useClass/useFactory`** (hoặc `useMocker` tự động)
  — tận dụng **DI** để tráo bản giả.
- **E2E:** **`createNestApplication()` + Supertest** bắn HTTP thật, giống `tests/Feature` của Laravel.

[↑ Mục lục](#mục-lục)

<a id="q11-1"></a>
### 11.1. Viết unit test trong NestJS: dùng gì & tổ chức thế nào?

#### 📖 Giới thiệu kiến thức

**Dùng gì (bộ công cụ):**

| Thành phần | Vai trò |
|---|---|
| **Jest** | Test runner + assertion (`expect`) + mocking (`jest.fn`, `jest.spyOn`) — mặc định của Nest |
| **ts-jest** | Biên dịch TypeScript cho Jest |
| **`@nestjs/testing`** | Cung cấp **`Test.createTestingModule()`** để dựng module DI trong test |
| *(tùy chọn)* **jest-mock-extended** | Tạo mock **type-safe** theo interface (`mock<OrderRepo>()`) |

> **Unit test trong Nest = test 1 provider (service/guard/pipe...) cô lập**, **mock toàn bộ dependency**.
> Vì Nest dựa trên **DI**, ta không `new` class thủ công mà để `createTestingModule` **inject bản giả** vào.

**Tổ chức — 4 quy ước chính:**

1. **Đặt file cạnh source, đuôi `*.spec.ts`** (co-location): `orders.service.ts` → `orders.service.spec.ts`.
   (E2E thì để riêng `test/*.e2e-spec.ts`.)
2. **`describe` gom theo class/method, `it` mô tả 1 hành vi** (spec-style, lấy cảm hứng BDD): mỗi `it` chỉ
   kiểm **một** kỳ vọng chính.
3. **`beforeEach` dựng module + lấy instance**: tạo lại **module DI mới mỗi test** để các test **độc lập**,
   không rò trạng thái.
4. **Theo AAA (Arrange–Act–Assert)** trong mỗi `it`, và **`afterEach(jest.clearAllMocks)`** để reset mock
   giữa các test.

```ts
// orders.service.spec.ts — đặt cạnh orders.service.ts
describe('OrdersService', () => {
  let service: OrdersService;
  // mock dependency: chỉ khai báo method service dùng tới
  const repo = { create: jest.fn(), findOne: jest.fn() };
  const mailer = { send: jest.fn() };

  beforeEach(async () => {                                  // dựng lại module mỗi test → độc lập
    const moduleRef = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: OrderRepository, useValue: repo },       // tráo dependency bằng bản giả
        { provide: Mailer, useValue: mailer },
      ],
    }).compile();
    service = moduleRef.get(OrdersService);
  });

  afterEach(() => jest.clearAllMocks());                    // reset lời gọi giữa các test

  describe('create()', () => {                              // gom test theo method
    it('lưu đơn và trả về id', async () => {
      // Arrange
      repo.create.mockResolvedValue({ id: 1, total: 100 });
      // Act
      const result = await service.create({ total: 100 });
      // Assert
      expect(result.id).toBe(1);
      expect(repo.create).toHaveBeenCalledWith({ total: 100 });
    });

    it('ném lỗi khi total <= 0', async () => {
      await expect(service.create({ total: 0 })).rejects.toThrow(BadRequestException);
      expect(repo.create).not.toHaveBeenCalled();          // không chạm DB khi invalid
    });
  });
});
```

**Test các loại provider khác (cùng khuôn):** test **controller** → mock service; test **guard** → tạo
`ExecutionContext` giả rồi assert `canActivate`; test **pipe** → gọi `transform()` với input; test **custom
provider** → override token của nó. Tất cả đều qua `createTestingModule` + `useValue`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Dùng gì:** **Jest** (runner + assert + mock) + **ts-jest** + **`@nestjs/testing`** (`Test.createTesting
  Module`); tùy chọn **jest-mock-extended** để mock type-safe.
- **Bản chất unit test Nest:** test **1 provider cô lập**, **mock hết dependency**, để **DI inject bản giả**
  qua `providers: [{ provide: Token, useValue: mock }]` thay vì `new` thủ công.
- **Tổ chức:** file **`*.spec.ts` đặt cạnh source**; **`describe`** gom theo class/method, **`it`** một hành
  vi; **`beforeEach`** dựng module mới mỗi test (độc lập); theo **AAA** + **`clearAllMocks`** để reset.
- **Mọi loại (controller/guard/pipe/service)** đều cùng khuôn: `createTestingModule` + `overrideProvider/
  useValue`.

[↑ Mục lục](#mục-lục)

<a id="q11-2"></a>
### 11.2. Triển khai mock, stub, spy trong NestJS

#### 📖 Giới thiệu kiến thức

Khác Laravel (dùng Mockery — có mock *kỳ vọng trước*), NestJS mock **hoàn toàn bằng Jest**. Mọi bản giả tạo
bằng **`jest.fn()`** rồi tráo vào DI qua **`overrideProvider().useValue()`** (hoặc provider `useValue`). Ánh xạ
3 loại [test double](#q7) ([7.1](#q7-1)):

**1) STUB — trả lời định sẵn (`mockReturnValue`/`mockResolvedValue`):**

```ts
const repo = { findOne: jest.fn().mockResolvedValue({ id: 1, total: 100 }) }; // canned answer
// { provide: OrderRepository, useValue: repo }
const order = await service.get(1);
expect(order.total).toBe(100);          // state verification, không quan tâm gọi mấy lần
```

**2) SPY — bọc method thật rồi kiểm SAU (`jest.spyOn`):**

```ts
const spy = jest.spyOn(mailer, 'send');   // bọc method thật, vẫn gọi thật (trừ khi mockImplementation)
await service.place(order, 'a@x.com');
expect(spy).toHaveBeenCalledWith('a@x.com');   // act → then assert
```

**3) MOCK — `jest.fn()` + assert tương tác SAU:**

```ts
const repo = { create: jest.fn().mockResolvedValue({ id: 1 }) };
await service.create(dto);
expect(repo.create).toHaveBeenCalledTimes(1);        // behavior verification
expect(repo.create).toHaveBeenCalledWith(dto);
```

> **Điểm khác cốt lõi với Laravel:** Jest **không có mock "kỳ vọng trước, tự fail"** như Mockery
> (`shouldReceive->once()`). Trong Jest, **mock và spy hòa làm một**: đều là **`jest.fn()` ghi lại lời gọi**
> rồi **`expect(fn).toHaveBeenCalled...` kiểm sau** (act → then assert). Vì vậy ở NestJS, sự phân biệt
> stub/spy/mock chủ yếu là **ý định**, không phải API khác nhau.

#### 🎯 Tóm tắt trả lời phỏng vấn

- NestJS mock **toàn bộ bằng Jest**: tạo bản giả `jest.fn()` rồi tráo vào DI qua
  **`overrideProvider(Token).useValue(...)`** (hoặc provider `useValue`).
- **Stub** = `jest.fn().mockReturnValue/mockResolvedValue(v)` → trả lời cứng (state verification).
- **Spy** = **`jest.spyOn(obj, 'method')`** bọc method thật, rồi `expect(spy).toHaveBeenCalledWith(...)` (kiểm sau).
- **Mock** = `jest.fn()` rồi assert **`toHaveBeenCalledTimes/With`** sau khi chạy (behavior verification).
- **Khác Laravel:** Jest **không có mock kỳ vọng-trước** như Mockery → **mock ≈ spy** (đều ghi lại, kiểm sau);
  phân biệt chủ yếu ở **ý định**, không phải API.

[↑ Mục lục](#mục-lục)

<a id="q11-3"></a>
### 11.3. Testing trong NestJS có tạo data thật không?

#### 📖 Giới thiệu kiến thức

**Trả lời ngắn: tùy tầng test.** Bám theo [Testing Pyramid](#q5):

| Tầng | Có chạm DB thật? | Cách xử lý data |
|---|---|---|
| **Unit test** ([11.1](#q11-1)) | **KHÔNG** | Mock repository/DataSource bằng `jest.fn()` — **không có data thật**, không I/O |
| **Integration / E2E** | **CÓ** — nhưng vào **DB test riêng**, không phải production | Seed data thật rồi **dọn sạch** giữa các test |

**1) Unit test — KHÔNG tạo data thật.** Bản chất unit là **cô lập**: repository bị thay bằng bản giả, ví dụ
`{ provide: getRepositoryToken(Order), useValue: { save: jest.fn() } }`. Không hề kết nối DB → nhanh, không
side-effect. "Data" chỉ là object trong RAM do mock trả về.

**2) Integration/E2E — CÓ tạo data thật, nhưng trên DB "dùng xong bỏ".** Tuyệt đối **không dùng DB
production**. Các lựa chọn phổ biến:
- **SQLite `:memory:`** — nhanh, sạch mỗi lần khởi tạo (nhưng khác dialect với Postgres/MySQL thật).
- **DB test riêng** (Postgres/MySQL cho môi trường test) — sát production nhất.
- **Testcontainers** — spin up **Postgres/MySQL thật trong Docker** cho mỗi lần chạy test rồi hủy → vừa
  thật vừa cô lập, đang là cách được ưa chuộng.

**Giữ DB sạch giữa các test (quan trọng để test độc lập):**
- **Transaction rollback** mỗi test (bọc test trong transaction rồi rollback — nhanh nhất).
- **Truncate/clear** bảng ở `afterEach` (`repository.clear()` / `dataSource.synchronize(true)`).
- **Migrate lại / drop-create** schema ở `beforeAll` (chậm hơn, chắc chắn nhất).
- Seed dữ liệu bằng **factory/fixture** trong `beforeEach`.

**3) Service ngoài luôn giả — kể cả E2E.** Gửi mail, thanh toán, gọi API bên thứ 3... **không bao giờ gọi
thật** khi test → mock/`Http` fake/`nock`. "Data thật" chỉ nói tới **DB của chính app**, không phải bên ngoài.

```ts
// UNIT — mock repo, KHÔNG có DB
providers: [
  OrdersService,
  { provide: getRepositoryToken(Order), useValue: { save: jest.fn(), find: jest.fn() } },
]

// E2E — DB test thật (SQLite :memory:), dọn sạch sau mỗi test
beforeAll(async () => {
  const moduleRef = await Test.createTestingModule({
    imports: [
      TypeOrmModule.forRoot({ type: 'sqlite', database: ':memory:',
        entities: [Order], synchronize: true }),        // DB test, tự tạo schema
      OrdersModule,
    ],
  }).compile();
  app = moduleRef.createNestApplication();
  await app.init();
  dataSource = moduleRef.get(DataSource);
});
afterEach(async () => {
  await dataSource.getRepository(Order).clear();          // xóa data giữa các test
});
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Tùy tầng:** **unit test KHÔNG** tạo data thật (mock repository, chạy trong RAM); **integration/E2E CÓ**
  tạo data thật nhưng trên **DB test riêng**, **không bao giờ dùng production**.
- **DB test hay dùng:** **SQLite `:memory:`** (nhanh), **DB test riêng** (sát thật), hoặc **Testcontainers**
  (Postgres/MySQL thật trong Docker rồi hủy).
- **Giữ độc lập:** dọn DB giữa các test bằng **transaction rollback** / **truncate ở `afterEach`** / **migrate
  lại**; seed bằng **factory/fixture**.
- **Service ngoài** (mail, payment, API 3rd-party) **luôn mock** kể cả E2E — "data thật" chỉ là **DB của app**.

[↑ Mục lục](#mục-lục)

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
