# 🎯 Interview — Tài liệu ôn phỏng vấn Fullstack

> Bộ câu hỏi & câu trả lời phỏng vấn vị trí **Fullstack Developer** (PHP/Laravel · JS/Node ·
> Vue.js · Database · DevOps), tổ chức theo chủ đề. Mỗi câu trả lời gồm **2 phần**:
> *kiến thức để hiểu sâu* + *bản tóm tắt để nói khi phỏng vấn*.

---

## 🚀 Bắt đầu nhanh

1. **Đọc câu hỏi:** mở file theo chủ đề trong các thư mục bên dưới. Mỗi file có **Mục lục (📑)**
   ở đầu — click để nhảy thẳng tới câu hỏi.
2. **Thêm câu hỏi mới:** dán câu hỏi cho Claude (hoặc gọi skill `interview-answer`). Claude sẽ tự
   *phân loại đúng file → research → viết theo format 2 phần → cập nhật Mục lục*.
3. **Bộ rule đầy đủ:** xem [`CLAUDE.md`](./CLAUDE.md) — nguồn quy tắc chính cho định tuyến,
   định dạng, đánh số và sắp xếp câu hỏi.

---

## 🗂️ Cấu trúc thư mục

| Thư mục / File | Nội dung | Ngôn ngữ |
|----------------|----------|----------|
| [`Introduce/introduce.md`](./Introduce/introduce.md) | Giới thiệu bản thân, behavioral, câu hỏi HR | 🇬🇧 English |
| [`Common/common.md`](./Common/common.md) | HTTP/REST, REST/GraphQL, Session/Cookie/Cache, microservice, kiến trúc | 🇻🇳 Tiếng Việt |
| [`Common/security.md`](./Common/security.md) | Bảo mật web chung: CIA, OWASP, XSS, CSRF, SQLi | 🇻🇳 Tiếng Việt |
| [`Common/oop.md`](./Common/oop.md) | OOP: 4 tính chất, interface, abstract, composition | 🇻🇳 Tiếng Việt |
| [`Common/solid-design-pattern.md`](./Common/solid-design-pattern.md) | SOLID & Design Pattern, DRY/KISS/YAGNI | 🇻🇳 Tiếng Việt |
| [`Common/testing.md`](./Common/testing.md) | Testing: unit/integration/e2e, TDD, mock/stub | 🇻🇳 Tiếng Việt |
| [`Common/ai.md`](./Common/ai.md) | AI/LLM chung: LLM, token/context, prompt, RAG/embedding, agent, function calling | 🇻🇳 Tiếng Việt |
| [`Common/resolve-problem.md`](./Common/resolve-problem.md) | Bài toán kỹ thuật cụ thể đã giải: tối ưu, debug, race condition (STAR) | 🇻🇳 Tiếng Việt |
| [`Backend/php-core.md`](./Backend/php-core.md) | PHP core: cú pháp, engine, OOP, phiên bản, Composer | 🇻🇳 Tiếng Việt |
| [`Backend/laravel.md`](./Backend/laravel.md) | Laravel: Eloquent, Artisan, middleware, queue, Blade | 🇻🇳 Tiếng Việt |
| [`Backend/js-core.md`](./Backend/js-core.md) | JavaScript core: closure, event loop, Promise, ES6+ | 🇻🇳 Tiếng Việt |
| [`Backend/nodejs.md`](./Backend/nodejs.md) | Node.js: runtime, module, stream, Express, NestJS, npm | 🇻🇳 Tiếng Việt |
| [`Backend/database-system.md`](./Backend/database-system.md) | Hệ CSDL: SQL, index, transaction, NoSQL, Redis | 🇻🇳 Tiếng Việt |
| [`Backend/devops.md`](./Backend/devops.md) | AWS, Cloud, Docker, CI/CD, Nginx, Linux | 🇻🇳 Tiếng Việt |
| [`Frontend/vuejs.md`](./Frontend/vuejs.md) | Vue.js, Composition API, Pinia, Nuxt.js | 🇻🇳 Tiếng Việt |
| [`ProjectHighlights/`](./ProjectHighlights/) | Kể về dự án thật: tổng quan (bối cảnh/vai trò/kiến trúc) + điểm nhấn kỹ thuật, theo STAR (skill `project-highlight`) | 🇻🇳 Tiếng Việt |
| [`MockInterview/`](./MockInterview/) | Transcript + đánh giá các buổi phỏng vấn thử (skill `mock-interview`) | 🇻🇳 Tiếng Việt |

---

## 🛠️ Skills hỗ trợ

Ngoài `interview-answer` (thêm câu hỏi vào tài liệu), repo có thêm 2 skill luyện phỏng vấn:

- **`project-highlight`** — Đưa **đường dẫn một project** → skill khảo sát codebase và ghi ra
  `ProjectHighlights/<ten-project>.md` gồm 5–10 **điểm nhấn kỹ thuật** đáng kể (kiến trúc, tối ưu
  hiệu năng, đồng thời, bảo mật, đoạn code khéo...), kèm cách trình bày theo STAR, dẫn chứng
  `file:line` và các câu hỏi đào sâu có thể bị hỏi.
  > _"Highlight kỹ thuật của project `/path/to/my-project`."_

- **`mock-interview`** — **Phỏng vấn thử tương tác**: Claude đóng vai nhà tuyển dụng, hỏi **từng câu
  một** dựa trên **CV / JD / ảnh + vị trí, level**; bạn trả lời như ứng viên. Gõ **"kết thúc"** để
  dừng → nhận đánh giá (điểm mạnh/yếu, câu trả lời mẫu, chấm điểm theo mảng, kết luận) và lưu toàn
  bộ transcript vào `MockInterview/<ngày>-<vị-trí>.md`. Ngôn ngữ mặc định Tiếng Việt (theo JD).
  > _"Phỏng vấn thử vị trí Backend Node.js (Middle), CV ở `cv.pdf`, JD dán bên dưới..."_

---

## 📐 Quy ước trình bày

- Mỗi file có **Mục lục** ở đầu, dùng **anchor HTML** (`<a id="qN">`) nên link không vỡ dù
  tiêu đề có dấu tiếng Việt.
- Câu hỏi lớn đánh số `1, 2, 3...`; câu đào sâu là mục con `1.1, 1.2...`.
- Mỗi câu trả lời theo đúng thứ tự 2 phần:
  1. **📖 Giới thiệu kiến thức** — nền tảng để hiểu sâu.
  2. **🎯 Tóm tắt trả lời phỏng vấn** — bản súc tích để trình bày.
- **Code example** chỉ thêm khi câu mang tính kỹ thuật và ví dụ giúp làm rõ.
- File tiếng Việt giữ nguyên **thuật ngữ kỹ thuật tiếng Anh** (index, middleware, dependency
  injection...).
