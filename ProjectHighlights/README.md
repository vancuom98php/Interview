# 🎯 Project Highlights

Nơi **duy nhất** để chuẩn bị phần *kể về dự án* khi phỏng vấn — gộp cả **tổng quan dự án**
(bối cảnh, vai trò, kiến trúc, công nghệ, thách thức, kết quả) lẫn **điểm nhấn kỹ thuật** đáng nói,
rút trực tiếp từ codebase thật (không viết tay chung chung).

> Trước đây vai trò này bị tách đôi giữa `Project/project.md` và thư mục này — nay đã gộp về đây.
> Riêng **bài toán/tình huống kỹ thuật cụ thể đã giải** (race condition, tối ưu, debug bug khó...)
> vẫn ở [`Common/resolve-problem.md`](../Common/resolve-problem.md).

- Mỗi project → một file `.md` (vd `ProjectHighlights/ten-project.md`).
- Nội dung được sinh bởi skill [`project-highlight`](../.claude/skills/project-highlight/SKILL.md):
  đưa đường dẫn project → skill khảo sát codebase → chọn lọc 5–10 điểm nhấn (kiến trúc, tối ưu
  hiệu năng, đồng thời, bảo mật, đoạn code khéo...) → mỗi điểm có *là gì → vì sao đáng nói → cách
  trình bày (STAR) → dẫn chứng `file:line` → câu hỏi đào sâu*.

## Cách tạo

Đưa đường dẫn project cho Claude, ví dụ:

> "Highlight kỹ thuật của project `/path/to/my-project`."

## Danh sách project

| File | Mảng nhấn | Ghi chú |
|------|-----------|---------|
| [virtual-office-backend.md](virtual-office-backend.md) | OAuth2 flow (`sync-meeting`) | NestJS + BullMQ + Redis; Factory 3 provider Google/Teams/Zoom, distributed lock chống race khi refresh token |
