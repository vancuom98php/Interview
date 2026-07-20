---
name: project-highlight
description: Use when the user gives a project path and wants to highlight / extract the important technical techniques of that real codebase for interview preparation. Analyzes the codebase at the provided path and writes an interview-ready highlight document — architecture, notable patterns, performance/security handling, hard problems solved, tech-stack choices — to ProjectHighlights/<project>.md. Trigger whenever the user provides a project/repo path and asks to "highlight kỹ thuật", "điểm nhấn kỹ thuật", extract talking points, or prep to talk about a project in an interview, even if they don't say the word "skill".
---

# Project Highlight

Phân tích một project thật (theo đường dẫn người dùng cung cấp) rồi rút ra **các điểm nhấn kỹ
thuật đáng nói khi phỏng vấn**, ghi thành một file trong `ProjectHighlights/`.

Mục tiêu không phải mô tả toàn bộ codebase, mà là **chọn lọc** những kỹ thuật/quyết định thiết kế
mà một ứng viên nên chủ động kể để gây ấn tượng — kèm cách trình bày và các câu hỏi đào sâu nhà
tuyển dụng có thể hỏi tiếp. Đây là tài liệu để *nói*, không phải tài liệu kỹ thuật để onboard.

Ngôn ngữ: **Tiếng Việt**, giữ nguyên thuật ngữ kỹ thuật tiếng Anh (middleware, index, queue...).

## Đầu vào
- **Bắt buộc:** đường dẫn tới thư mục project. Nếu người dùng chưa đưa, hỏi đúng 1 câu ngắn.
- **Tùy chọn:** người dùng nêu vai trò của họ trong dự án, hoặc muốn nhấn mảng nào (backend,
  performance, DevOps...). Nếu có, ưu tiên khai thác đúng mảng đó.

## Quy trình (làm theo thứ tự)

### 1. Khảo sát codebase để hiểu bức tranh lớn
Đọc trước những thứ cho biết *hình dạng* của dự án, đừng đọc tràn lan:
- README / docs, file cấu hình dependency (`package.json`, `composer.json`, `go.mod`,
  `requirements.txt`...), file compose/deploy (`Dockerfile`, `docker-compose`, CI config).
- Cây thư mục để nhận ra kiến trúc (layered, modular, microservice, monorepo...).
- Vài module cốt lõi: entrypoint, tầng service/domain, nơi xử lý data, auth, background jobs.

Mục đích: xác định **stack, kiến trúc, và những chỗ "có gì đó thú vị"** — không phải đọc hết mọi file.

### 2. Chọn lọc điểm nhấn (đây là phần quan trọng nhất)
Chỉ giữ những thứ **đáng kể khi phỏng vấn** — ưu tiên cái khó, cái thể hiện tư duy thiết kế, hoặc
giải quyết một vấn đề thực. Săn tìm theo các nhóm sau (không cần đủ hết, chọn cái thực sự có):

- **Kiến trúc & tổ chức code:** pattern tổng thể, cách phân tách trách nhiệm, ranh giới module,
  quyết định monolith/microservice và lý do.
- **Xử lý hiệu năng:** caching, index, tối ưu query/N+1, batching, pagination, lazy loading,
  queue/async, xử lý dữ liệu lớn.
- **Độ tin cậy & đồng thời:** transaction, lock, idempotency, retry, xử lý race condition, rate limit.
- **Bảo mật:** auth/authz, validation, chống XSS/CSRF/SQLi, quản lý secret.
- **Tích hợp & hạ tầng:** third-party API, message broker, Docker/K8s, CI/CD, cấu hình môi trường.
- **Chất lượng:** testing, error handling, logging/observability, migration.
- **Đoạn code khéo:** thuật toán/logic nghiệp vụ đáng chú ý, abstraction gọn, tự viết thư viện nhỏ.

Với mỗi điểm nhấn, **định vị bằng chứng trong code** (`file:line`) để người dùng mở lại xem được.

Số lượng: nhắm **5–10 điểm nhấn chất lượng**. Thà ít mà sâu còn hơn liệt kê dàn trải.

### 3. Viết file `ProjectHighlights/<ten-project>.md`
Tạo thư mục `ProjectHighlights/` ở gốc repo nếu chưa có. Tên file theo tên project (kebab-case).
Nếu file đã tồn tại: cập nhật/bổ sung, đừng ghi đè mù. Dùng đúng cấu trúc ở mục Template bên dưới.

### 4. Tự kiểm trước khi kết thúc
- Mỗi điểm nhấn có: nó là gì → vì sao đáng nói → cách trình bày → dẫn chứng code → câu hỏi đào sâu.
- Điểm nhấn phản ánh **đúng những gì có trong code**, không bịa công nghệ không tồn tại.
- Mục lục (📑) khớp anchor. Ngôn ngữ đúng, thuật ngữ tiếng Anh giữ nguyên.

## Template file highlight

ALWAYS dùng đúng khung này:

```markdown
<a id="muc-luc"></a>
# 🎯 Điểm nhấn kỹ thuật — <Tên Project>

> Nguồn: `<đường dẫn project>` · Ngày phân tích: <YYYY-MM-DD>
> Tài liệu để **kể khi phỏng vấn** — mỗi điểm nhấn kèm cách trình bày & câu hỏi đào sâu.

## 📌 Tổng quan
- **Loại dự án / mục đích:** ...
- **Stack chính:** <ngôn ngữ, framework, DB, hạ tầng>
- **Kiến trúc tổng thể:** <mô tả 1–2 câu>
- **Vai trò của bạn (nếu biết):** ...

## 📑 Mục lục
1. [<Tên điểm nhấn 1>](#h1)
2. [<Tên điểm nhấn 2>](#h2)
...

---

<a id="h1"></a>
## 1. <Tên điểm nhấn — ngắn, gợi kỹ thuật>

**📖 Nó là gì & vì sao đáng nói**
<Giải thích kỹ thuật/quyết định thiết kế; vấn đề nó giải quyết; vì sao thú vị trong dự án này.>

**🎯 Cách trình bày khi phỏng vấn** (hướng STAR khi hợp lý)
- **Bối cảnh/vấn đề:** ...
- **Giải pháp & lý do chọn:** ...
- **Kết quả:** <số liệu nếu có>

**🔎 Dẫn chứng trong code**
- `path/to/file.ext:42` — <chỗ này làm gì>

**💬 Câu hỏi đào sâu có thể bị hỏi**
- <câu 1> → <gợi ý hướng trả lời>
- <câu 2>

[↑ Mục lục](#muc-luc)
```

## Nguyên tắc
- **Trung thực với code.** Chỉ highlight cái thật sự tồn tại; nếu một mảng (vd testing) yếu/thiếu,
  đừng bịa — có thể ghi ngắn ở phần lưu ý để người dùng chủ động ứng phó nếu bị hỏi.
- **Chọn lọc hơn liệt kê.** Một nhà tuyển dụng nhớ 5 điểm sắc chứ không nhớ 30 gạch đầu dòng.
- Nếu project quá lớn, ưu tiên mảng người dùng chỉ định; nêu rõ đã tập trung vào phần nào.