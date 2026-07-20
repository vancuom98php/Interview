<a id="mục-lục"></a>
# Frontend — CSS & SCSS

> CSS · SCSS/Sass · Preprocessor · Variables · Nesting · Mixin · Function · @extend · Control directives.

## 📑 Mục lục

1. [SCSS có gì hơn CSS?](#q1)

---

<a id="q1"></a>
## 1. SCSS có gì hơn CSS?

### 📖 Giới thiệu kiến thức

**SCSS (Sassy CSS)** là cú pháp phổ biến nhất của **Sass** — một **CSS preprocessor**. Điểm cốt lõi cần hiểu: trình duyệt **không hiểu SCSS trực tiếp**. Ta viết code bằng cú pháp mở rộng (`.scss`), rồi một bước **compile** (build step) sẽ dịch nó ra **CSS thuần** (`.css`) để browser đọc. Nói cách khác, SCSS không thêm khả năng mới cho browser — nó là công cụ giúp **viết** CSS gọn hơn, có tổ chức hơn, dễ bảo trì hơn (DRY, maintainability), đặc biệt với dự án lớn.

SCSS là **superset của CSS**: mọi file CSS hợp lệ cũng là SCSS hợp lệ. Vì vậy có thể đổi tên `.css` → `.scss` là chạy được ngay, rồi bổ sung dần các tính năng nâng cao.

Những tính năng SCSS có mà CSS thuần (truyền thống) thiếu:

**1. Variables (biến)** — lưu giá trị dùng lại (màu, spacing, font). CSS hiện đại có custom properties (`--x`, chạy ở *runtime*), còn biến SCSS (`$x`) hoạt động lúc *compile-time*, cho phép tính toán/xử lý mạnh hơn.
```scss
$primary: #3498db;
$spacing: 16px;

.button {
  background: $primary;
  padding: $spacing;
}
```

**2. Nesting (lồng selector)** — viết theo cấu trúc HTML, thể hiện rõ quan hệ cha–con. `&` đại diện cho selector cha.
```scss
.card {
  padding: 20px;
  .title { font-size: 18px; }        // .card .title
  &:hover { box-shadow: 0 2px 8px rgba(0,0,0,.2); }  // .card:hover
}
```

**3. Mixin** — khối CSS tái sử dụng, **có tham số** (giống hàm). Tránh lặp code.
```scss
@mixin flex-center($direction: row) {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: $direction;
}
.box { @include flex-center(column); }
```

**4. Functions & toán tử** — tính toán ngay lúc compile: `math.div()`, `darken()`, `lighten()`, cộng/trừ/nhân/chia...
```scss
@use "sass:math";
.col { width: math.div(100%, 3); }                  // chia cột
.btn:hover { background: darken($primary, 10%); }   // làm tối màu 10%
```

**5. `@extend` / placeholder (`%`)** — kế thừa style, share chung để không lặp lại khai báo.
```scss
%btn-base { padding: 10px; border-radius: 4px; }
.btn-primary { @extend %btn-base; background: blue; }
```

**6. Partials & module (`@use` / `@import`)** — tách code thành nhiều file nhỏ (`_variables.scss`, `_buttons.scss`) rồi gộp lại → quản lý dự án lớn dễ hơn.

**7. Control directives (logic)** — có `@if`, `@for`, `@each`, `@while` để **sinh CSS tự động**.
```scss
@for $i from 1 through 3 {
  .mt-#{$i} { margin-top: #{$i * 8}px; }   // sinh .mt-1, .mt-2, .mt-3
}
```

**Đánh đổi:** SCSS cần **build step** (compile), thêm một chút phức tạp vào toolchain. Ngoài ra CSS hiện đại đã thu hẹp khoảng cách: custom properties (`--x`), native nesting, `calc()`... nên với dự án nhỏ, CSS thuần nhiều khi đã đủ.

**So sánh nhanh:**

| Tính năng | CSS thuần | SCSS |
|-----------|-----------|------|
| Variables | ✅ `--x` (runtime) | ✅ `$x` (compile-time, mạnh hơn) |
| Nesting | ⚠️ mới hỗ trợ, hạn chế | ✅ đầy đủ |
| Mixin / function | ❌ | ✅ |
| Loop / if (logic) | ❌ | ✅ |
| `@extend` (kế thừa) | ❌ | ✅ |
| Cần compile | Không | **Có** (build step) |

### 🎯 Tóm tắt trả lời phỏng vấn
- SCSS là một **CSS preprocessor** (cú pháp của Sass), **compile ra CSS thuần** để browser chạy — browser không hiểu SCSS trực tiếp.
- Nó là **superset của CSS**: mọi CSS hợp lệ đều là SCSS hợp lệ.
- Điểm hơn chính: **variables, nesting, mixin (có tham số), function/toán tử, `@extend`, partials, và control directives (`@if`/`@for`/`@each`)**.
- Lợi ích lớn nhất: **DRY + maintainability** cho dự án lớn; đổi lại phải thêm **build step**.
- CSS hiện đại (custom properties, native nesting) đã thu hẹp khoảng cách, nên dự án nhỏ dùng CSS thuần vẫn ổn.

[↑ Mục lục](#mục-lục)
