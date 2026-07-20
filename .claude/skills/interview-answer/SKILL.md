---
name: interview-answer
description: Use when the user provides a fullstack interview question (PHP/Laravel, JS/Node, database, devops, Vue, OOP/SOLID/design pattern/microservice, or English self-introduction) to research and write into this Interview document. Routes the question to the correct topic file and writes a two-part answer (knowledge intro + interview summary) with an updated clickable table of contents.
---

# Interview Answer

Quy trình chuẩn để thêm một câu hỏi phỏng vấn vào tài liệu ôn tập này.
Đọc kỹ `CLAUDE.md` ở gốc project — đó là nguồn rule chính. Skill này là checklist thực thi.

## Khi nào dùng
Người dùng dán 1 câu hỏi (hoặc danh sách câu hỏi) phỏng vấn fullstack và muốn ghi vào tài liệu.

## Checklist (làm theo thứ tự)

1. **Phân loại → chọn file** theo bảng routing trong `CLAUDE.md §1`.
   - Introduce (EN) · Common (gồm `ai.md`, `resolve-problem.md` — bài toán kỹ thuật đã giải theo STAR) · Backend/php · Backend/js · Backend/database-system · Backend/devops · Frontend/vuejs.
   - Câu tổng quan về dự án thật (bối cảnh/kiến trúc/điểm nhấn) không viết tay ở đây — dùng skill `project-highlight` ghi vào `ProjectHighlights/`.
   - Nếu trọng tâm không rõ giữa 2 file, hỏi đúng 1 câu ngắn rồi tiếp tục.

2. **Chèn đúng nhóm, không chỉ append** (`CLAUDE.md §3.1`): đặt câu mới cạnh các câu cùng chủ đề;
   câu đào sâu của câu khác → mục con (`N.m`); sắp xếp **cơ bản → nâng cao** trong nhóm và toàn file.

3. **Research** nội dung chính xác. Ưu tiên dùng skill `find-docs` cho kiến thức công nghệ cụ thể
   (Laravel, Vue, Node, AWS...) để tránh sai lệch phiên bản.

4. **Viết theo format 2 phần** (`CLAUDE.md §2`):
   - `📖 Giới thiệu kiến thức` (để hiểu) → `🎯 Tóm tắt trả lời phỏng vấn` (để nói).
   - Thêm code example nếu kỹ thuật và hữu ích.
   - Tiếng Việt cho mọi file trừ `Introduce` (tiếng Anh).

5. **Renumber + Anchor + Mục lục** (`CLAUDE.md §3 & §3.1`):
   - Sau khi sắp xếp, **đánh số lại toàn bộ** câu trong file cho liên tục; cập nhật `<a id="qN">`/`qN-m` khớp.
   - **Dựng lại Mục lục** với link `[N. ...](#qN)`, mục con thụt vào.
   - Thêm dòng `[↑ Mục lục](#mục-lục)` cuối mỗi câu.
   - Chỉ đổi vị trí/số/anchor; **không sửa nội dung** câu đã có.

6. **Tự kiểm**: nhóm hợp lý, thứ tự cơ bản→nâng cao, số liên tục, anchor khớp link TOC, đủ 2 phần, đúng ngôn ngữ.

## Template
Xem `CLAUDE.md §4` và ví dụ đã điền trong `Common/common.md`.
