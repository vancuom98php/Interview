# CLAUDE.md — Rules cho tài liệu ôn phỏng vấn Fullstack

Đây là bộ quy tắc **bắt buộc** mà Claude phải tuân theo mỗi khi người dùng đưa một câu hỏi
phỏng vấn để research và ghi vào tài liệu. Mục tiêu: mỗi câu hỏi rơi đúng file, đúng định dạng,
có mục lục click được.

> Khi người dùng dán một câu hỏi (hoặc danh sách câu hỏi), hãy: **(1) phân loại → chọn file đúng →
> (2) research → (3) viết theo format 2 phần → (4) cập nhật Mục lục.** Không hỏi lại nếu đã đủ rõ.

---

## 1. Định tuyến câu hỏi → file (Routing)

Phân loại theo *kiến thức cốt lõi* của câu hỏi, không theo từ khóa bề mặt.

| File | Thuộc về | Ngôn ngữ trả lời |
|------|----------|------------------|
| `Introduce/introduce.md` | Giới thiệu bản thân, behavioral, "tell me about yourself", điểm mạnh/yếu, tình huống, câu hỏi HR | **Tiếng Anh** |
| `Common/common.md` | HTTP/REST, HTTP method, REST/GraphQL, Session/Cookie/Cache, microservice, kiến trúc chung, clean code, cấu trúc dữ liệu & giải thuật | Tiếng Việt |
| `Common/security.md` | **Bảo mật web ở mức khái niệm chung**: CIA, OWASP Top 10, XSS, CSRF, SQL Injection, auth | Tiếng Việt |
| `Common/oop.md` | OOP: 4 tính chất, class/object, interface, abstract, composition vs inheritance | Tiếng Việt |
| `Common/solid-design-pattern.md` | 5 nguyên lý SOLID, Design Pattern (Factory, Singleton, Strategy, Observer...), DRY/KISS/YAGNI | Tiếng Việt |
| `Common/testing.md` | Testing: unit/integration/e2e, TDD/BDD, mock/stub/spy, coverage, testing pyramid | Tiếng Việt |
| `Common/algorithm.md` | Thuật toán & cấu trúc dữ liệu: Big-O, mảng/chuỗi, stack/queue, linked list, hash map, cây/đồ thị, sorting/searching, đệ quy, quy hoạch động, two-pointer/sliding-window | Tiếng Việt |
| `Common/git.md` | Git & version control: workflow, branch/merge/rebase, staging area, conflict, remote, các lệnh cốt lõi (commit, stash, cherry-pick, reset/revert), Git flow | Tiếng Việt |
| `Common/ai.md` | AI/LLM cho fullstack dev (khái niệm chung): LLM là gì, token/context window, prompt engineering, RAG & embedding, vector DB, function/tool calling, agent, fine-tuning vs RAG, hallucination & đạo đức AI | Tiếng Việt |
| `Backend/php-core.md` | PHP core: cú pháp, engine, OOP trong PHP, kiểu dữ liệu, phiên bản & tính năng, Composer, OPcache/PHP-FPM | Tiếng Việt |
| `Backend/laravel.md` | Laravel: Eloquent, migration, Artisan, middleware, service container, queue/job, event, Blade, validation | Tiếng Việt |
| `Backend/js-core.md` | JavaScript core: closure, `this`, prototype, event loop, async/await, Promise, ES6+ | Tiếng Việt |
| `Backend/nodejs.md` | Node.js: runtime/V8, event loop (libuv), module, stream, cluster, Express, npm | Tiếng Việt |
| `Backend/nestjs.md` | NestJS: module/controller/provider, dependency injection & IoC, middleware/guard/interceptor/pipe/filter, validation (DTO), dynamic module, lifecycle hooks, testing, microservices | Tiếng Việt |
| `Backend/database-system.md` | SQL, MySQL/PostgreSQL, index, transaction, ACID, normalization, NoSQL, Redis, tối ưu query | Tiếng Việt |
| `Backend/devops.md` | AWS, cloud server, Docker, Kubernetes, CI/CD, Nginx, Linux, deploy, monitoring | Tiếng Việt |
| `Frontend/vuejs.md` | Vue.js, Composition API, reactivity, Vuex/Pinia, Vue Router, Nuxt.js | Tiếng Việt |
| `ProjectHighlights/<project>.md` | Kinh nghiệm & điểm nhấn kỹ thuật rút ra từ project thật: bối cảnh, vai trò, kiến trúc, công nghệ đã dùng, thách thức & cách xử lý, kết quả — kèm dẫn chứng `file:line` và câu hỏi đào sâu. Sinh bởi skill `project-highlight`. Trả lời theo hướng STAR | Tiếng Việt |
| `Common/resolve-problem.md` | Bài toán/tình huống kỹ thuật thực tế đã giải: tối ưu hiệu năng, debug bug khó, scale, đồng bộ dữ liệu, race condition, xử lý dữ liệu lớn. Trả lời theo hướng STAR | Tiếng Việt |

**Quy tắc ranh giới:**
- Bảo mật ở mức **khái niệm chung** → `Common/security.md`. Bảo mật **gắn chặt 1 công nghệ** (vd: "Laravel chống CSRF thế nào") → file công nghệ đó (`laravel.md`).
- **Thuật toán/cấu trúc dữ liệu lý thuyết & bài coding** (Big-O, sorting, leetcode-style) → `Common/algorithm.md`.
- **NestJS** (module/DI/guard/interceptor/pipe, decorator của Nest...) → `Backend/nestjs.md`. **Node.js runtime thuần** (event loop/libuv, stream, cluster, Express) → `Backend/nodejs.md`.
- **Project**: câu hỏi tổng quan về dự án (bối cảnh, vai trò, kiến trúc, công nghệ, điểm nhấn kỹ thuật) → thư mục `ProjectHighlights/` (tạo/bổ sung file qua skill `project-highlight` từ codebase thật, không viết tay chung chung). Câu hỏi về **một bài toán/tình huống kỹ thuật cụ thể đã giải quyết** (tối ưu, debug, scale, race condition...) → `Common/resolve-problem.md`.
- **AI/LLM ở mức khái niệm chung** (LLM, prompt, RAG, embedding, agent, function calling...) → `Common/ai.md`. AI **gắn chặt 1 công nghệ/dự án** thì về file tương ứng.
- Nếu một câu dính nhiều chủ đề: chọn file theo *trọng tâm câu hỏi*; nếu thật sự ngang nhau thì hỏi người dùng 1 câu ngắn.

---

## 2. Định dạng câu trả lời (BẮT BUỘC 2 phần)

Mỗi câu hỏi gồm **đúng 2 phần**, theo thứ tự:

1. **📖 Giới thiệu kiến thức** — giải thích nền tảng để *hiểu* chủ đề: định nghĩa, bối cảnh,
   tại sao quan trọng, các thành phần liên quan. Có thể dài, đủ để học.
2. **🎯 Tóm tắt trả lời phỏng vấn** — bản súc tích, *đủ để nói khi phỏng vấn*: gạch đầu dòng
   hoặc 3–6 câu, đúng trọng tâm, dễ thuộc.

- **Code example:** chỉ thêm khi câu hỏi mang tính kỹ thuật và ví dụ giúp làm rõ (PHP, JS, SQL...).
  Câu lý thuyết thuần thì không bắt buộc. Snippet ngắn gọn, có chú thích.
- **Ngôn ngữ:** theo bảng routing. Với file tiếng Việt: giữ nguyên thuật ngữ kỹ thuật tiếng Anh
  (dependency injection, index, middleware...), không dịch cứng.
- File `Introduce`: cả câu hỏi và trả lời đều bằng **tiếng Anh**.

---

## 3. Đánh số & Mục lục

- Mỗi file có **Mục lục (📑) ở trên cùng**, mỗi mục là link click được nhảy tới câu tương ứng.
- Câu hỏi lớn đánh số `1, 2, 3...`. Câu hỏi phụ/đào sâu là **mục con**: `1.1, 1.2, 2.1...`.
  - Ví dụ: `1. Bảo mật trong web là gì?` → `1.1. Nêu ví dụ cụ thể (XSS)`.
- **Anchor tường minh** để link không vỡ:
  - Câu lớn: đặt `<a id="q1"></a>` ngay trên heading; TOC trỏ `#q1`.
  - Câu con: `<a id="q1-1"></a>`; TOC trỏ `#q1-1`.
- Mỗi câu có dòng `[↑ Mục lục](#mục-lục)` để quay lại đầu file.

### 3.1. Nhóm & sắp xếp khi thêm câu mới (BẮT BUỘC)

Câu mới **không chỉ append** vào cuối. Mỗi lần thêm câu, phải tổ chức lại cả file:

1. **Gom nhóm theo chủ đề liên quan** — đặt các câu cùng một mảng kiến thức cạnh nhau
   (vd trong `laravel.md`: nhóm "Eloquent cơ bản" → nhóm "relationship/query" → nhóm
   "queue/performance"). Một câu là *biến thể/đào sâu* của câu khác thì để làm **mục con** (`N.m`).
2. **Sắp xếp độ khó tăng dần** — trong từng nhóm và trên toàn file, đi từ **cơ bản → nâng cao**
   (định nghĩa/khái niệm trước, cơ chế nội bộ & tối ưu/edge-case sau).
3. **Đánh số lại toàn bộ** — sau khi sắp xếp, **renumber** liên tục `1, 2, 3...` (và mục con `N.m`),
   cập nhật **anchor** (`<a id="qN">` / `<a id="qN-m">`) cho khớp, rồi **dựng lại Mục lục** từ đầu.
4. **Bảo toàn nội dung** — chỉ đổi vị trí, số thứ tự và anchor; **không sửa nội dung** câu trả lời
   đã có (trừ khi người dùng yêu cầu).

> Nguyên tắc: đọc file từ trên xuống phải có cảm giác một lộ trình học mạch lạc, từ dễ tới khó.

---

## 4. Template chuẩn cho 1 câu hỏi

```markdown
<a id="q1"></a>
## 1. <Câu hỏi lớn>

### 📖 Giới thiệu kiến thức
<phần nền tảng để hiểu>

### 🎯 Tóm tắt trả lời phỏng vấn
- <ý chính 1>
- <ý chính 2>

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. <Câu hỏi con>

#### 📖 Giới thiệu kiến thức
...

#### 🎯 Tóm tắt trả lời phỏng vấn
...

[↑ Mục lục](#mục-lục)
```

Xem ví dụ hoàn chỉnh đã điền trong `Common/common.md` (câu 1 + 1.1).
