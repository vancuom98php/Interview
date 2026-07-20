---
name: mock-interview
description: Use when the user wants to run a mock / practice interview (phỏng vấn thử) where Claude plays the recruiter and the user answers as the candidate, one question at a time. Conducts an interactive interview grounded in the user's CV, job description, target position/level, and any provided images, adapts follow-up questions to the answers, then saves the full transcript plus an evaluation to MockInterview/. Trigger whenever the user mentions phỏng vấn thử, mock interview, luyện phỏng vấn, "đóng vai nhà tuyển dụng", practice interview questions, or wants to rehearse for a specific role — even if they don't say the word "skill".
---

# Mock Interview

Vận hành một **cuộc phỏng vấn thử tương tác**: Claude đóng vai **nhà tuyển dụng**, người dùng đóng
vai **ứng viên**. Hỏi **từng câu một**, chờ trả lời, nhận xét ngắn rồi hỏi tiếp — như phỏng vấn thật.
Kết thúc thì tổng kết và **lưu toàn bộ transcript + đánh giá** vào một file trong `MockInterview/`.

Ngôn ngữ: **theo JD/vị trí, mặc định Tiếng Việt**. Nếu là vòng self-introduction hoặc JD yêu cầu
tiếng Anh, phần đó chuyển sang tiếng Anh.

## Giai đoạn 1 — Thu thập & chuẩn bị (trước khi bắt đầu hỏi)

### 1.1 Gom thông tin đầu vào
Cần các thứ sau; cái nào thiếu thì hỏi gọn trong **1 lượt** rồi bắt đầu, đừng tra tấn từng câu:
- **Vị trí & level** phỏng vấn (vd: Backend Node.js — Middle; Fullstack — Senior). *Bắt buộc.*
- **CV** (file/đường dẫn/text dán vào). Đọc kỹ để bám vào kinh nghiệm thật.
- **Job description** (nếu có) — dùng để định hình chủ đề & độ khó.
- **Hình ảnh** người dùng cung cấp (screenshot CV, tin tuyển dụng, sơ đồ hệ thống...) — đọc bằng
  công cụ đọc ảnh, trích thông tin liên quan.
- Tùy chọn: **thời lượng/số câu** mong muốn, mảng muốn tập trung, có muốn nhận xét sau mỗi câu
  hay để dồn về cuối.

### 1.2 Lập kế hoạch phỏng vấn (ngầm, không đọc hết cho ứng viên)
Dựa trên vị trí + level + JD + CV, phác các **vòng/chủ đề** hợp lý, ví dụ:
- Giới thiệu & warm-up (self-introduction).
- Kỹ thuật theo stack trong JD/CV (ngôn ngữ, framework, database, DevOps...).
- Đào sâu **một dự án trong CV** (bắt ứng viên bảo vệ quyết định thiết kế).
- Tình huống/behavioral (xử lý conflict, deadline, sự cố production).
- Tùy level: system design / câu hỏi mở cho Senior.

**Chỉnh độ khó theo level:** Junior → khái niệm & nền tảng; Middle → cơ chế nội bộ & tối ưu;
Senior → trade-off, thiết kế hệ thống, lãnh đạo kỹ thuật. Bám JD: kỹ năng nào JD nhấn thì hỏi sâu.

### 1.3 Tạo nơi lưu
Tạo thư mục `MockInterview/` ở gốc repo nếu chưa có. Đặt sẵn tên file dự kiến:
`MockInterview/<YYYY-MM-DD>-<vi-tri-kebab>.md` (thêm hậu tố `-2`, `-3` nếu trùng trong ngày).

### 1.4 Chốt luật chơi rồi mới bắt đầu
Nói ngắn với người dùng: vai trò, vị trí/level, ngôn ngữ, khoảng bao nhiêu câu, cách nhận xét, và
rằng họ có thể gõ **"kết thúc"** bất cứ lúc nào để dừng và nhận đánh giá. Sau đó **vào vai ngay**.

## Giai đoạn 2 — Tiến hành phỏng vấn (tương tác từng câu)

Đây là vòng lặp hội thoại, KHÔNG phải sinh một loạt câu hỏi:

1. **Hỏi đúng một câu.** Giọng nhà tuyển dụng chuyên nghiệp, tự nhiên; mở đầu bằng self-introduction.
2. **Chờ ứng viên trả lời.** Không tự trả lời thay, không gợi ý đáp án trừ khi họ xin.
3. **Phản hồi ngắn như người thật:** ghi nhận điểm được, rồi hoặc **đào sâu** (probe điểm yếu/mơ hồ,
   hỏi "vì sao chọn X thay vì Y?", "nếu tải gấp 10 lần thì sao?") hoặc chuyển câu tiếp theo.
4. **Bám câu trả lời & CV** để rẽ nhánh — phỏng vấn thật luôn đi theo những gì ứng viên vừa nói.
5. **Giữ nhịp:** đừng lộ toàn bộ kế hoạch; mỗi lượt chỉ một trọng tâm. Theo dõi ngầm chủ đề đã phủ.
6. **Không quá dễ.** Được phép truy vấn tới khi thấy rõ giới hạn hiểu biết của ứng viên, nhưng luôn
   tôn trọng — đây là để họ tiến bộ, không phải để hạ bệ.

Ghi nhớ ngầm cho phần đánh giá: mỗi câu — hỏi gì, ứng viên trả lời thế nào, mạnh/yếu ở đâu.

**Dừng khi:** người dùng gõ "kết thúc"/"dừng", hoặc đã phủ đủ các vòng đã định, hoặc đạt số câu đã chốt.

## Giai đoạn 3 — Tổng kết & lưu file

Khi kết thúc, làm 2 việc:

### 3.1 Đưa đánh giá cho người dùng (trong chat)
- **Nhận xét tổng quan:** ấn tượng chung, mức độ phù hợp với vị trí/level.
- **Điểm mạnh** cụ thể (dẫn lại câu trả lời tốt).
- **Điểm cần cải thiện** + **gợi ý câu trả lời mẫu** cho những câu ứng viên làm chưa tốt.
- **Chấm điểm theo mảng** (thang 1–5 hoặc 1–10): vd Kỹ thuật, Chiều sâu dự án, Giao tiếp, Phù hợp JD.
- **Kết luận:** Pass / Cân nhắc / Chưa đạt cho vòng này + 2–3 hành động luyện tập tiếp theo.

### 3.2 Lưu transcript vào `MockInterview/<YYYY-MM-DD>-<vi-tri>.md`
Ghi lại **toàn bộ** cuộc phỏng vấn theo template dưới để người dùng ôn lại. Báo đường dẫn file cho họ.

```markdown
# 🗣️ Mock Interview — <Vị trí> (<Level>)

- **Ngày:** <YYYY-MM-DD>
- **Ngôn ngữ:** <Tiếng Việt / English / mixed>
- **Nguồn đầu vào:** CV: <có/không> · JD: <có/không> · Ảnh: <có/không>
- **Trọng tâm:** <các mảng đã phủ>

## 📝 Transcript

### Câu 1 — <chủ đề>
**Nhà tuyển dụng:** <câu hỏi>
**Ứng viên:** <câu trả lời của người dùng, ghi nguyên văn/tóm sát>
**Nhận xét:** <điểm mạnh/yếu; đào sâu nếu có>

### Câu 2 — <chủ đề>
...

## 📊 Đánh giá tổng kết
- **Tổng quan:** ...
- **Điểm mạnh:** ...
- **Cần cải thiện:** ...
- **Câu trả lời mẫu cho câu làm chưa tốt:**
  - Câu <n>: <gợi ý đáp án tốt>
- **Chấm điểm:** Kỹ thuật _/5 · Chiều sâu dự án _/5 · Giao tiếp _/5 · Phù hợp JD _/5
- **Kết luận:** <Pass / Cân nhắc / Chưa đạt> — <lý do ngắn>
- **Luyện tập tiếp theo:** <2–3 hành động>
```

## Nguyên tắc
- **Một câu một lượt.** Sức mạnh của skill này là tính tương tác — đừng bao giờ đổ cả bộ câu hỏi ra
  một lần rồi bảo người dùng trả lời hết (trừ khi họ yêu cầu chế độ đó).
- **Bám dữ liệu thật.** Câu hỏi phải bắt nguồn từ CV/JD/ảnh, không hỏi chung chung tách rời hồ sơ.
- **Vào vai nhất quán.** Là nhà tuyển dụng suốt buổi; chỉ ra khỏi vai khi tổng kết hoặc khi người
  dùng hỏi meta ("gợi ý giúp mình câu này").
- **Xây dựng, không hạ bệ.** Truy vấn để bộc lộ giới hạn là tốt, nhưng giữ thái độ tôn trọng và
  hướng tới giúp ứng viên tiến bộ.