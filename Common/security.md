<a id="mục-lục"></a>
# Common — Bảo mật Web (Web Security)

> Khái niệm bảo mật web chung: CIA, OWASP Top 10, XSS, CSRF, SQL Injection, authentication/authorization.
> (Bảo mật gắn chặt 1 công nghệ — vd "Laravel chống CSRF thế nào" — nằm ở file công nghệ đó.)

## 📑 Mục lục

- [1. Bảo mật trong web là gì?](#q1)
- [2. Các lỗ hổng bảo mật web phổ biến](#q2)
- [3. XSS (Cross-Site Scripting) là gì?](#q3)
- [4. SQL Injection (SQLi) là gì?](#q4)
- [5. CSRF (Cross-Site Request Forgery) là gì?](#q5)
- [6. CORS (Cross-Origin Resource Sharing) là gì?](#q6)
- [7. SSRF (Server-Side Request Forgery) là gì?](#q7)
- [8. Phân biệt authentication và authorization](#q8)
- [9. Có mấy cách authentication (xác thực)?](#q9)
  - [9.1. Basic Authentication là gì?](#q9-1)
  - [9.2. Session-based Authentication là gì?](#q9-2)
  - [9.3. Token-based Authentication là gì?](#q9-3)
  - [9.4. Bearer token là gì?](#q9-4)
  - [9.5. JWT gồm những phần nào? Verify thế nào?](#q9-5)
  - [9.6. Tại sao JWT chia 3 phần mà không chỉ dùng signature?](#q9-6)
  - [9.7. Payload của JWT thường nên chứa những gì?](#q9-7)
  - [9.8. Phân biệt HS256, RS256 và ES256](#q9-8)
  - [9.9. Nếu phải "pin" alg, thì header alg còn tác dụng gì?](#q9-9)
  - [9.10. JWE là gì? Khác JWS/JWT thế nào?](#q9-10)
  - [9.11. Access token vs Refresh token; vì sao refresh dài hơn; cách revoke token](#q9-11)
  - [9.12. Refresh token được lưu ở server bằng cách nào?](#q9-12)
  - [9.13. Refresh token rotation + reuse detection — mỗi lần /refresh làm gì?](#q9-13)
  - [9.14. Cách revoke hàng loạt JWT đã phát (đổi mật khẩu, force logout)](#q9-14)
  - [9.15. OAuth2 là gì? Flow và các thành phần chính của OAuth2](#q9-15)
- [10. Broken Authentication / Session là gì?](#q10)
  - [10.1. Lưu access/refresh token ở đâu trên client & truyền access token thế nào?](#q10-1)
- [11. SSL là gì? Cách sử dụng?](#q11)
  - [11.1. Phân biệt SSL và TLS](#q11-1)
  - [11.2. HTTPS và HTTP khác nhau thế nào?](#q11-2)
- [12. Sensitive Data Exposure là gì?](#q12)
  - [12.1. Hashing và Encryption khác nhau thế nào? Cách lưu password vào DB?](#q12-1)
  - [12.2. Cách compare (verify) password đã hash khi đăng nhập?](#q12-2)
  - [12.3. Sử dụng salt trong hash password như thế nào?](#q12-3)
  - [12.4. Phân biệt salt và pepper chi tiết](#q12-4)
  - [12.5. Chặn user đặt mật khẩu mới trùng 3 mật khẩu gần nhất (password history)](#q12-5)
- [13. Các hàm hash và mã hoá phổ biến](#q13)
- [14. So sánh constant-time là gì? (timing attack)](#q14)

---

<a id="q1"></a>
## 1. Bảo mật trong web là gì?

### 📖 Giới thiệu kiến thức

**Bảo mật web (Web Security)** là tập hợp các biện pháp bảo vệ ứng dụng web, dữ liệu và người dùng
khỏi các mối đe dọa trên môi trường Internet. Mục tiêu xoay quanh bộ ba **CIA**:

- **Confidentiality (Bảo mật)** — dữ liệu chỉ người được phép mới xem được (mã hóa, phân quyền).
- **Integrity (Toàn vẹn)** — dữ liệu không bị sửa đổi trái phép (chữ ký số, hash, HTTPS).
- **Availability (Sẵn sàng)** — hệ thống luôn hoạt động (chống DDoS, rate limit, backup).

Các nhóm rủi ro phổ biến được tổng hợp trong **OWASP Top 10** (Injection, Broken Access Control,
XSS, CSRF, Security Misconfiguration...). Bảo mật là *cross-cutting concern* — phải áp dụng xuyên suốt
từ input validation, authentication/authorization, mã hóa truyền tải (TLS), tới quản lý session và secret.

### 🎯 Tóm tắt trả lời phỏng vấn

- Bảo mật web = bảo vệ ứng dụng/dữ liệu/người dùng theo 3 trụ cột **CIA**: Confidentiality, Integrity, Availability.
- Tham chiếu chuẩn là **OWASP Top 10**: các lỗ hổng hay gặp như **XSS, CSRF, SQL Injection, Broken Access Control**.
- Biện pháp cốt lõi: **validate input, escape output, dùng HTTPS/TLS, authentication + authorization, prepared statement, rate limiting, quản lý secret an toàn**.
- Tư duy: bảo mật là việc xuyên suốt (defense in depth), không phải bước làm cuối.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Các lỗ hổng bảo mật web phổ biến

### 📖 Giới thiệu kiến thức

Phần lớn lỗ hổng web đến từ **một gốc rễ chung**: *tin tưởng dữ liệu/đầu vào mà lẽ ra không nên tin*
(input từ người dùng, từ request, từ hệ thống ngoài). Tổ chức **OWASP** định kỳ tổng hợp các nhóm rủi
ro nguy hiểm nhất trong **OWASP Top 10**. Dưới đây là các lỗ hổng hay được hỏi nhất.

#### 1. Injection — tiêu biểu là SQL Injection

Kẻ tấn công **chèn mã** vào dữ liệu đầu vào để "lừa" trình thông dịch (SQL, OS command, LDAP...) thực
thi ý đồ của chúng. Phổ biến nhất là **SQL Injection**: nối thẳng input vào câu SQL.

```php
// ❌ DỄ BỊ TẤN CÔNG — nối chuỗi trực tiếp
$sql = "SELECT * FROM users WHERE email = '$email'";
// input: ' OR '1'='1  → trả về toàn bộ user, hoặc xoá bảng...

// ✅ AN TOÀN — prepared statement (tham số hoá)
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?');
$stmt->execute([$email]);
```

**Chống:** **prepared statement / parameterized query** (quan trọng nhất), ORM, validate input,
nguyên tắc **least privilege** cho tài khoản DB. Chi tiết và các dạng SQLi xem [câu 4](#q4).

#### 2. XSS (Cross-Site Scripting)

Chèn **JavaScript độc hại** chạy trên trình duyệt nạn nhân → đánh cắp cookie/session, mạo danh thao
tác. Chi tiết và ví dụ xem [câu 3](#q3). **Chống:** escape output theo ngữ cảnh, template
auto-escape, **CSP**, cookie `HttpOnly`.

#### 3. CSRF (Cross-Site Request Forgery)

Lợi dụng việc trình duyệt **tự gửi kèm cookie** đăng nhập, kẻ tấn công lừa nạn nhân (đang đăng nhập)
gửi một **request ngoài ý muốn** tới site nạn nhân (vd form ẩn tự submit chuyển tiền).

**Chống:** **CSRF token** (token ngẫu nhiên gắn vào form, server kiểm tra), cookie `SameSite`, kiểm tra
header `Origin`/`Referer`, yêu cầu xác nhận lại với hành động nhạy cảm. Chi tiết xem [câu 5](#q5).

> Phân biệt nhanh: **XSS** lợi dụng việc site *tin code chạy trên trang*; **CSRF** lợi dụng việc site
> *tin request kèm cookie hợp lệ*.

#### 4. Broken Access Control / IDOR

Người dùng truy cập được tài nguyên/chức năng **không thuộc quyền** của mình. Dạng hay gặp là **IDOR**
(Insecure Direct Object Reference): đổi `id` trên URL để xem dữ liệu người khác.

```text
GET /invoices/1001   → của tôi
GET /invoices/1002   → của người khác, nhưng vẫn xem được vì server không kiểm tra chủ sở hữu
```

**Chống:** **kiểm tra quyền ở server cho MỌI request** (đừng tin client), deny-by-default, kiểm tra
ownership, không lộ ID tuần tự nhạy cảm.

#### 5. Broken Authentication / Session

Cơ chế đăng nhập/phiên yếu: mật khẩu yếu, brute-force không giới hạn, session token đoán được, không
hết hạn, lộ token. **Chống:** hash mật khẩu mạnh (**bcrypt/Argon2**), **rate limit** + khoá tài khoản,
**MFA**, session token ngẫu nhiên, xoay token sau đăng nhập, set timeout, cookie `HttpOnly`+`Secure`.
Chi tiết xem [câu 10](#q10).

#### 6. Sensitive Data Exposure

Lộ dữ liệu nhạy cảm do **không mã hoá**: truyền không TLS, lưu mật khẩu plaintext, log chứa thông tin
nhạy cảm. **Chống:** **HTTPS/TLS** bắt buộc, mã hoá dữ liệu khi lưu (at rest), không log secret, hash
mật khẩu (không "mã hoá hai chiều"). Chi tiết xem [câu 12](#q12).

#### 7. Security Misconfiguration

Cấu hình sai/ mặc định không an toàn: bật `debug` ở production, lộ stack trace, để mật khẩu mặc định,
mở cổng/dịch vụ thừa, header bảo mật thiếu. **Chống:** tắt debug ở prod, gỡ tính năng/tài khoản mặc
định, cập nhật bản vá, set **security headers** (CSP, HSTS, X-Frame-Options...).

#### 8. SSRF (Server-Side Request Forgery)

Lừa **server** gửi request tới đích kẻ tấn công chọn (vd `http://169.254.169.254` lấy credential trên
cloud). **Chống:** whitelist domain/IP được phép, chặn dải IP nội bộ, không cho user kiểm soát URL mà
server sẽ gọi. Chi tiết xem [câu 7](#q7).

#### 9. Các lỗ hổng khác hay gặp

- **Insecure Deserialization** — deserialize dữ liệu không tin cậy → thực thi code.
- **Path Traversal** — `../../etc/passwd` để đọc file ngoài thư mục cho phép.
- **Unrestricted File Upload** — upload file `.php` rồi truy cập để chạy.
- **Open Redirect** — chuyển hướng tới URL độc hại qua tham số không kiểm soát.
- **Using Components with Known Vulnerabilities** — dùng thư viện/dependency có CVE chưa vá.

#### Bảng tóm tắt

| Lỗ hổng | Bản chất | Phòng chống cốt lõi |
|---------|----------|---------------------|
| SQL Injection | Chèn mã vào câu truy vấn | **Prepared statement**, ORM, validate |
| XSS | Chèn JS chạy ở trình duyệt nạn nhân | **Escape output**, CSP, HttpOnly |
| CSRF | Lợi dụng cookie tự gửi kèm | **CSRF token**, SameSite |
| Broken Access Control / IDOR | Truy cập tài nguyên không có quyền | **Kiểm tra quyền server-side**, ownership |
| Broken Authentication | Đăng nhập/session yếu | Hash mạnh, rate limit, MFA |
| Sensitive Data Exposure | Lộ dữ liệu không mã hoá | HTTPS, mã hoá at rest, hash password |
| Security Misconfiguration | Cấu hình sai/mặc định | Tắt debug, security headers, vá bản |
| SSRF | Lừa server gọi request | Whitelist, chặn IP nội bộ |

### 🎯 Tóm tắt trả lời phỏng vấn

- Hầu hết lỗ hổng có **gốc chung: tin dữ liệu đầu vào không nên tin**; chuẩn tham chiếu là **OWASP Top 10**.
- Nhóm hay hỏi nhất: **SQL Injection** (chống bằng **prepared statement**), **XSS** (escape output +
  CSP + HttpOnly), **CSRF** (CSRF token + SameSite), **Broken Access Control/IDOR** (kiểm tra quyền
  **server-side** cho mọi request), **Broken Authentication** (hash bcrypt/Argon2, rate limit, MFA).
- Phân biệt **XSS vs CSRF:** XSS = site tin *code chạy trên trang*; CSRF = site tin *request kèm cookie
  hợp lệ*.
- Bổ sung: **Sensitive Data Exposure** (HTTPS, hash password), **Security Misconfiguration** (tắt debug,
  security headers), **SSRF**, path traversal, file upload, dependency có CVE.
- Tư duy chốt: **không tin input, kiểm tra quyền ở server, defense in depth, dùng HTTPS, vá dependency thường xuyên.**

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. XSS (Cross-Site Scripting) là gì?

### 📖 Giới thiệu kiến thức

**XSS (Cross-Site Scripting)** là một lỗ hổng bảo mật web cho phép kẻ tấn công **chèn mã độc** (thường
là **JavaScript**) vào nội dung của website, để mã đó **chạy trên trình duyệt của người dùng khác**.
Đây là một trong những lỗ hổng trong danh mục ở [câu 2](#q2).

Ví dụ, website hiển thị trực tiếp dữ liệu người dùng nhập:

```html
<div>Xin chào, <script>alert('XSS')</script></div>
```

Nếu dữ liệu không được xử lý an toàn, đoạn `<script>` có thể được trình duyệt **thực thi**.

#### XSS có thể gây ra gì?

Kẻ tấn công có thể:

- **Đánh cắp token đăng nhập** hoặc dữ liệu nhạy cảm mà JavaScript truy cập được.
- **Thực hiện hành động dưới danh nghĩa người dùng**.
- Thay đổi nội dung giao diện.
- Chuyển hướng người dùng sang website giả mạo.
- Ghi lại nội dung người dùng nhập (keylogging).
- Phát tán mã độc sang những người dùng khác.

> Lưu ý: cookie có thuộc tính **`HttpOnly`** không thể bị đọc trực tiếp bằng JavaScript, **nhưng** XSS
> vẫn có thể **thực hiện yêu cầu bằng phiên đăng nhập hiện tại** của nạn nhân (gửi request kèm cookie).

#### Ba loại XSS phổ biến

**1. Stored XSS**

Mã độc được **lưu trong cơ sở dữ liệu** (vd: bình luận, bài viết, tên người dùng). Mỗi khi người khác
mở trang, mã độc sẽ chạy.

```text
Kẻ tấn công nhập bình luận chứa script
→ Server lưu vào database
→ Người dùng khác xem bình luận
→ Script được thực thi
```

Đây thường là loại **nguy hiểm nhất** vì có thể ảnh hưởng đến nhiều người.

**2. Reflected XSS**

Payload nằm trong **URL hoặc request** và được server **phản chiếu trực tiếp** vào HTML.

```text
https://example.com/search?q=<script>alert(1)</script>
```

Nếu server đưa `q` vào trang mà không escape, script có thể chạy khi nạn nhân mở liên kết.

**3. DOM-based XSS**

Lỗ hổng xảy ra **hoàn toàn ở phía trình duyệt**, do JavaScript lấy dữ liệu không tin cậy rồi đưa vào
DOM bằng **API nguy hiểm**:

```js
// ❌ nguy hiểm — innerHTML thực thi HTML/script
const value = location.hash.substring(1);
document.getElementById("result").innerHTML = value;
```

```js
// ✅ an toàn hơn — textContent chỉ in văn bản, không thực thi
const value = location.hash.substring(1);
document.getElementById("result").textContent = value;
```

#### Cách phòng chống

Nguyên tắc quan trọng nhất là **encode dữ liệu theo đúng ngữ cảnh đầu ra**, thay vì chỉ lọc một vài từ
khoá như `<script>` (blacklist dễ bị vượt qua).

- **Escape dữ liệu** trước khi hiển thị trong HTML.
- Dùng **`textContent`** thay cho `innerHTML` khi chỉ cần hiển thị văn bản.
- **Không** dùng `eval()`, `document.write()` hoặc tạo code từ dữ liệu người dùng.
- Dùng **thư viện sanitize uy tín** (vd DOMPurify) khi bắt buộc cho phép HTML.
- Áp dụng **Content Security Policy (CSP)**.
- Đặt cookie với **`HttpOnly`, `Secure`, `SameSite`**.
- Không cho phép URL nguy hiểm như `javascript:`.
- Dùng cơ chế **auto-escaping của framework** và tránh vô hiệu hóa nó.

Ví dụ trong **React**, cách thông thường được escape tự động:

```jsx
<div>{userInput}</div>
```

Nhưng đoạn sau có **rủi ro** nếu `userInput` chưa được sanitize:

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

Tóm lại, **XSS xảy ra khi website biến dữ liệu không đáng tin cậy thành mã có thể thực thi trong trình
duyệt.**

### 🎯 Tóm tắt trả lời phỏng vấn

- **XSS** = website biến **dữ liệu không đáng tin** thành mã (thường là JS) **chạy được trên trình
  duyệt nạn nhân** → đánh cắp token/session, mạo danh thao tác, đổi giao diện, redirect, keylog.
- **3 loại:** **Stored** (lưu DB, ảnh hưởng nhiều người — nguy hiểm nhất), **Reflected** (payload trong
  URL/request, phản chiếu lại), **DOM-based** (lỗi ở JS client đưa data vào DOM).
- **Chống (cốt lõi):** **encode/escape output theo đúng ngữ cảnh** (không dùng blacklist); ưu tiên
  **`textContent` thay `innerHTML`**; tránh `eval`/`document.write`; **CSP**; cookie **`HttpOnly` +
  `Secure` + `SameSite`**; sanitize (DOMPurify) khi buộc cho phép HTML; dùng auto-escape của framework.
- **Lưu ý:** `HttpOnly` chặn JS đọc cookie, nhưng XSS **vẫn gửi request bằng session nạn nhân** được.
- Câu chốt: "Không bao giờ tin dữ liệu người dùng; luôn escape/encode khi render ra trình duyệt."

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. SQL Injection (SQLi) là gì?

### 📖 Giới thiệu kiến thức

**SQL Injection** là lỗ hổng cho phép kẻ tấn công **chèn (inject) mã SQL** vào câu truy vấn của ứng
dụng thông qua dữ liệu đầu vào không được xử lý an toàn. Khi đó **dữ liệu của người dùng bị diễn dịch
thành một phần câu lệnh SQL**, khiến database thực thi ý đồ của kẻ tấn công thay vì truy vấn ban đầu.
Đây là một trong những lỗ hổng nguy hiểm nhất trong danh mục ở [câu 2](#q2).

#### Nguyên nhân gốc: nối chuỗi input vào câu SQL

```php
// ❌ DỄ BỊ TẤN CÔNG — nối thẳng input vào câu lệnh
$sql = "SELECT * FROM users WHERE email = '$email' AND password = '$password'";
```

Nếu kẻ tấn công nhập `email` = `' OR '1'='1' -- ` thì câu lệnh trở thành:

```sql
SELECT * FROM users WHERE email = '' OR '1'='1' -- ' AND password = '...'
```

- `'1'='1'` luôn đúng → trả về **toàn bộ user**.
- `--` biến phần còn lại thành **comment**, vô hiệu hoá việc kiểm tra password → **đăng nhập trái phép**.

#### SQLi có thể gây ra gì?

- **Bypass đăng nhập** (auth bypass) như ví dụ trên.
- **Đọc trộm dữ liệu** nhạy cảm (toàn bộ bảng user, thẻ, mật khẩu hash...).
- **Sửa/xoá dữ liệu** (`UPDATE`, `DELETE`, `DROP TABLE`).
- Trong một số cấu hình, **đọc/ghi file** hoặc **thực thi lệnh hệ điều hành** trên server DB.

#### Các dạng SQLi phổ biến

- **In-band** — lấy kết quả ngay trên cùng kênh request:
  - **Error-based**: dựa vào thông báo lỗi DB để moi thông tin.
  - **Union-based**: dùng `UNION SELECT` để ghép dữ liệu bảng khác vào kết quả.
- **Blind SQLi** — ứng dụng không trả dữ liệu/lỗi trực tiếp, kẻ tấn công suy luận qua phản hồi:
  - **Boolean-based**: dựa vào trang đổi đúng/sai theo điều kiện chèn.
  - **Time-based**: dùng `SLEEP()` và đo thời gian phản hồi để suy ra từng bit dữ liệu.
- **Out-of-band** — moi dữ liệu qua kênh khác (DNS/HTTP) khi không có phản hồi trực tiếp.

#### Cách phòng chống

**Quan trọng nhất: Prepared Statement (parameterized query)** — tách **câu lệnh** khỏi **dữ liệu**, nên
input của người dùng **luôn được coi là giá trị**, không bao giờ là mã SQL:

```php
// ✅ AN TOÀN — PDO prepared statement
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ? AND password = ?');
$stmt->execute([$email, $password]);
```

```php
// ✅ Named placeholder cũng được
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
```

Các lớp phòng thủ bổ sung (defense in depth):

- **Dùng ORM / query builder** (Eloquent, Doctrine...) — mặc định đã tham số hoá.
- **Validate & ép kiểu input** (vd ID phải là số nguyên).
- **Least privilege**: tài khoản DB của app chỉ có quyền tối thiểu (không `DROP`, không `FILE`).
- **Không lộ thông báo lỗi DB** ra người dùng (tắt debug ở production) → chặn error-based.
- **Whitelist** với phần không thể tham số hoá (vd tên cột/bảng khi sắp xếp động).
- **WAF** như lớp chặn phụ (không thay thế prepared statement).

> Lưu ý: **escape thủ công** (vd `addslashes`, `mysqli_real_escape_string`) là cách cũ, **dễ sai và
> không an toàn bằng** prepared statement — đừng dựa vào nó làm biện pháp chính.

#### Dấu hiệu ứng dụng có nguy cơ SQL Injection

Khi review code, các "mùi" (code smell) sau cho thấy app dễ dính SQLi:

- Câu SQL được tạo bằng **nối chuỗi**.
- Dùng **interpolation** biến trực tiếp trong truy vấn (vd `"... WHERE id = $id"`).
- **Truyền toàn bộ request** (vd cả `$_GET`/`$_POST`) vào hàm query.
- **Hiển thị lỗi database** chi tiết ra giao diện.
- Tài khoản database có **quyền quá lớn** (root/admin).
- Có nhiều **raw query không bind parameter**.
- **Tên cột sắp xếp hoặc bộ lọc lấy trực tiếp từ URL** (không qua allowlist).
- **Mật khẩu được so sánh trực tiếp trong SQL** (thay vì băm rồi so trong code).
- **Không có kiểm thử bảo mật** cho API.

#### Checklist bảo mật SQLi

- [ ] Mọi giá trị đầu vào đều dùng **parameter binding**.
- [ ] **Không nối chuỗi** dữ liệu người dùng vào SQL.
- [ ] Tên bảng, tên cột và hướng sắp xếp dùng **allowlist**.
- [ ] Tài khoản database tuân theo **quyền tối thiểu** (least privilege).
- [ ] **Không dùng tài khoản root** cho ứng dụng.
- [ ] **Không hiển thị lỗi SQL** chi tiết ra ngoài.
- [ ] Mật khẩu được **băm bằng Argon2 hoặc bcrypt** (không lưu/so sánh plaintext).
- [ ] **Raw query** phải được review kỹ.
- [ ] Thêm **kiểm thử** các đầu vào bất thường (fuzzing/security test).
- [ ] **Theo dõi log, cảnh báo** và **giới hạn tần suất** request (rate limit).

### 🎯 Tóm tắt trả lời phỏng vấn

- **SQL Injection** = chèn mã SQL qua input không an toàn, khiến **dữ liệu người dùng bị diễn dịch thành
  câu lệnh SQL** → DB chạy theo ý kẻ tấn công.
- **Nguyên nhân gốc:** **nối chuỗi input vào câu SQL**. Ví dụ kinh điển `' OR '1'='1' -- ` để bypass
  đăng nhập.
- **Hậu quả:** auth bypass, đọc trộm/sửa/xoá dữ liệu, đôi khi đọc file hay RCE trên server DB.
- **Các dạng:** in-band (error-based, union-based), **blind** (boolean-based, time-based), out-of-band.
- **Chống (cốt lõi):** **Prepared statement / parameterized query** (tách lệnh khỏi dữ liệu) — biện pháp
  số 1; thêm **ORM**, **validate/ép kiểu**, **least privilege** cho DB user, ẩn lỗi DB, whitelist tên
  cột/bảng, WAF. Escape thủ công là cách cũ, **không nên** làm biện pháp chính.

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. CSRF (Cross-Site Request Forgery) là gì?

### 📖 Giới thiệu kiến thức

**CSRF (Cross-Site Request Forgery)** — *giả mạo yêu cầu xuyên trang* — là lỗ hổng khiến **trình duyệt
của nạn nhân (đang đăng nhập) gửi một request ngoài ý muốn** tới một website mà họ đã xác thực. Kẻ tấn
công không cần đọc dữ liệu hay đánh cắp cookie, chúng chỉ cần **khiến nạn nhân "thực hiện hành động"**
(chuyển tiền, đổi email, xoá dữ liệu...). Đây là một trong những lỗ hổng trong danh mục ở [câu 2](#q2).

#### Cơ chế: vì sao tấn công thành công?

Mấu chốt là **trình duyệt tự động đính kèm cookie** (gồm cookie phiên đăng nhập) vào *mọi* request gửi
tới một domain — kể cả request được kích hoạt từ **một trang web khác**. Server thấy request có cookie
hợp lệ nên **tưởng là nạn nhân chủ động thực hiện**.

```html
<!-- Trang evil.com mà nạn nhân vô tình mở (đang đăng nhập bank.com ở tab khác) -->
<form action="https://bank.com/transfer" method="POST" id="f">
  <input type="hidden" name="to" value="attacker">
  <input type="hidden" name="amount" value="1000">
</form>
<script>document.getElementById('f').submit();</script>  <!-- tự submit -->
```

Vì trình duyệt tự gửi kèm cookie `bank.com`, lệnh chuyển tiền được thực thi **dưới danh nghĩa nạn nhân**.

#### Điều kiện để CSRF xảy ra

- Nạn nhân **đang đăng nhập** (có session active).
- Hành động được kích hoạt bằng **request có thể đoán trước** (không có giá trị bí mật riêng mỗi lần).
- Site **chỉ dựa vào cookie** để xác thực request (cookie tự gửi kèm).

#### Phân biệt với XSS (rất hay hỏi)

- **XSS** lợi dụng việc site **tin code chạy trên trang** → chèn JS độc.
- **CSRF** lợi dụng việc site **tin request kèm cookie hợp lệ** → mượn phiên của nạn nhân.

> Quan trọng: nếu site **đã dính XSS** thì **CSRF token cũng vô dụng** (JS độc đọc được token). Vì vậy
> chống XSS là tiền đề cho chống CSRF.

#### Cách phòng chống

- **CSRF token (synchronizer token)** — biện pháp chính: server sinh một **token ngẫu nhiên, gắn vào
  form/phiên**, và **kiểm tra lại** khi nhận request. Trang ngoài **không biết token** nên không giả
  mạo được.

  ```html
  <form action="/transfer" method="POST">
    <input type="hidden" name="_token" value="a1b2c3...ngẫu_nhiên">
    ...
  </form>
  ```

- **Cookie `SameSite`** (`Lax`/`Strict`) — báo trình duyệt **không gửi cookie** cho request đến từ site
  khác → chặn phần lớn CSRF ở tầng trình duyệt.
- **Kiểm tra header `Origin`/`Referer`** — đảm bảo request xuất phát từ chính domain của mình.
- **Double-submit cookie** — gửi token vừa trong cookie vừa trong request, so khớp hai bên (khi không
  muốn lưu token ở server).
- **Yêu cầu xác thực lại** (nhập mật khẩu/OTP) cho hành động nhạy cảm.
- Dùng đúng **HTTP method theo ngữ nghĩa**: hành động thay đổi trạng thái phải là `POST`/`PUT`/`DELETE`,
  không dùng `GET` (vì `GET` dễ bị kích hoạt qua `<img>`, link...).

> Thực tế các framework (Laravel, Django, Rails...) đã có sẵn **CSRF middleware**: tự sinh token, tự
> kiểm tra — chỉ cần nhúng token vào form và **không tắt** cơ chế này.

### 🎯 Tóm tắt trả lời phỏng vấn

- **CSRF** = lừa **trình duyệt nạn nhân (đang đăng nhập)** gửi một **request ngoài ý muốn** tới site đã
  xác thực → thực hiện hành động (chuyển tiền, đổi email...) **dưới danh nghĩa nạn nhân**.
- **Vì sao được:** trình duyệt **tự đính kèm cookie phiên** vào mọi request tới domain, kể cả request
  khởi phát từ trang khác → server tưởng là nạn nhân chủ động.
- **Phân biệt XSS:** XSS = site tin *code chạy trên trang*; CSRF = site tin *request kèm cookie hợp lệ*.
  Nếu đã dính XSS thì CSRF token cũng vô dụng.
- **Chống (cốt lõi):** **CSRF token** (token ngẫu nhiên gắn form, server kiểm tra) + cookie
  **`SameSite`**; thêm kiểm tra **Origin/Referer**, double-submit cookie, re-auth cho hành động nhạy
  cảm, và **không dùng `GET`** cho thao tác đổi trạng thái. Framework thường có sẵn CSRF middleware.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. CORS (Cross-Origin Resource Sharing) là gì?

### 📖 Giới thiệu kiến thức

**CORS (Cross-Origin Resource Sharing)** là cơ chế dùng **HTTP header** để server **khai báo cho trình
duyệt biết những origin nào khác được phép** truy cập tài nguyên của mình — tức là **nới lỏng có kiểm
soát** quy tắc bảo mật mặc định của trình duyệt là **Same-Origin Policy (SOP)**.

#### Same-Origin Policy (SOP) — nền tảng cần hiểu trước

**Origin** = bộ ba **scheme + host + port** (vd `https://app.com:443`). SOP là quy tắc bảo mật của trình
duyệt: mặc định **JavaScript của trang này KHÔNG được đọc response từ một origin khác**. Nhờ vậy, một
trang độc không thể dùng JS đọc trộm dữ liệu từ trang ngân hàng mà bạn đang đăng nhập.

| URL so với `https://app.com` | Cùng origin? | Vì sao |
|------------------------------|:---:|--------|
| `https://app.com/api` | ✅ | giống scheme/host/port |
| `http://app.com` | ❌ | khác scheme |
| `https://api.app.com` | ❌ | khác host (subdomain) |
| `https://app.com:8080` | ❌ | khác port |

CORS sinh ra để giải quyết nhu cầu chính đáng: frontend `https://app.com` cần gọi API ở
`https://api.app.com`. Server API dùng CORS header để **cho phép** origin đó.

#### Cơ chế: server "cấp phép" bằng response header

Trình duyệt gửi request kèm header `Origin`; server trả về các header `Access-Control-*` để báo có cho
phép hay không. Nếu thiếu header phù hợp, **trình duyệt chặn không cho JS đọc response** (lỗi quen thuộc
"blocked by CORS policy"). Các header quan trọng:

- `Access-Control-Allow-Origin` — origin được phép (vd `https://app.com`, hoặc `*` cho tài nguyên public).
- `Access-Control-Allow-Methods` — method được phép (GET, POST, PUT...).
- `Access-Control-Allow-Headers` — header tuỳ biến được phép gửi.
- `Access-Control-Allow-Credentials: true` — cho phép gửi kèm **cookie/credential**.
- `Access-Control-Max-Age` — cache kết quả preflight trong bao lâu.

#### Simple request vs Preflight request

- **Simple request** — `GET`/`HEAD`/`POST` với `Content-Type` đơn giản (`text/plain`,
  `application/x-www-form-urlencoded`, `multipart/form-data`): trình duyệt gửi thẳng rồi mới kiểm tra
  header response.
- **Preflight** — khi request "không đơn giản" (method `PUT`/`DELETE`/`PATCH`, có header tuỳ biến, hoặc
  `Content-Type: application/json`): trình duyệt **tự gửi trước một request `OPTIONS`** để "hỏi phép".
  Chỉ khi server trả về cho phép thì request thật mới được gửi.

```text
Trình duyệt:  OPTIONS /api/orders     (preflight — kèm Origin + Access-Control-Request-Method)
Server:       200, Access-Control-Allow-Origin: https://app.com
                   Access-Control-Allow-Methods: POST, PUT
Trình duyệt:  POST /api/orders        (request thật — vì đã được cấp phép)
```

#### Hiểu đúng (rất hay bị nhầm)

- **CORS không phải để bảo vệ server của bạn.** Nó do **trình duyệt thực thi**, nhằm bảo vệ **người
  dùng**. Client không phải trình duyệt (cURL, Postman, server-to-server) **không bị CORS chặn**.
- **CORS không chống CSRF.** SOP/CORS quản việc *đọc response*; còn CSRF lợi dụng việc *gửi* request và
  cookie tự đính kèm (xem [câu 5](#q5)). Cấu hình CORS sai còn có thể **mở thêm** rủi ro.
- **Lỗ hổng CORS misconfiguration:** nếu server **phản chiếu (reflect) bừa header `Origin`** vào
  `Access-Control-Allow-Origin` **kèm** `Allow-Credentials: true`, thì *bất kỳ site nào* cũng đọc được
  dữ liệu đã đăng nhập của nạn nhân. Và **không được** dùng `*` chung với `Allow-Credentials: true`.

### 🎯 Tóm tắt trả lời phỏng vấn

- **CORS** = cơ chế dùng **HTTP header** để server cho phép **origin khác** truy cập tài nguyên, tức
  **nới lỏng có kiểm soát Same-Origin Policy (SOP)** của trình duyệt.
- **SOP:** mặc định JS **không đọc được response từ origin khác** (origin = **scheme + host + port**);
  CORS giúp frontend gọi API khác domain một cách hợp lệ.
- **Cách hoạt động:** trình duyệt gửi `Origin`; server trả `Access-Control-Allow-Origin/-Methods/
  -Headers/-Credentials`. Request "không đơn giản" (PUT/DELETE, JSON, custom header) sẽ có **preflight
  `OPTIONS`** hỏi phép trước.
- **Nhớ kỹ:** CORS **do trình duyệt thực thi, bảo vệ người dùng — không phải tường lửa cho server**
  (cURL/Postman không bị chặn) và **không chống CSRF** (xem [câu 5](#q5)).
- **Bẫy bảo mật:** đừng **reflect `Origin`** kèm `Allow-Credentials: true`, và **không** dùng `*` với
  credentials.

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. SSRF (Server-Side Request Forgery) là gì?

### 📖 Giới thiệu kiến thức

**SSRF (Server-Side Request Forgery)** là lỗ hổng cho phép kẻ tấn công **lừa server gửi request tới một
đích do chúng chọn**. Khác CSRF (lợi dụng *trình duyệt* nạn nhân), SSRF lợi dụng **chính server** làm
"bàn đạp" — nguy hiểm vì server thường **đứng ở vị trí tin cậy**, truy cập được **mạng nội bộ và dịch vụ
mà bên ngoài không với tới được**. Đây là một trong những lỗ hổng trong danh mục ở [câu 2](#q2).

#### SSRF xảy ra khi nào?

Khi ứng dụng **lấy một URL từ người dùng** rồi tự đi gọi nó ở phía server, ví dụ: tải ảnh từ URL,
webhook, tạo PDF/thumbnail từ link, "URL preview", proxy/fetch hộ...

```php
// ❌ Server gọi thẳng URL do người dùng cung cấp
$url = $_GET['image_url'];
$data = file_get_contents($url);   // kẻ tấn công truyền URL nội bộ
```

Kẻ tấn công truyền vào URL trỏ tới **tài nguyên nội bộ** thay vì ảnh:

```text
http://169.254.169.254/latest/meta-data/iam/security-credentials/   ← metadata cloud (AWS)
http://localhost:6379/                                              ← Redis nội bộ
http://192.168.1.10/admin                                          ← service trong mạng riêng
file:///etc/passwd                                                 ← đọc file cục bộ
```

#### Vì sao đặc biệt nguy hiểm

- **Đọc cloud metadata** (`169.254.169.254`) → lấy **credential tạm thời** của máy chủ trên AWS/GCP/Azure
  → leo thang chiếm tài khoản cloud. Đây là kịch bản SSRF kinh điển.
- **Quét cổng & dò dịch vụ nội bộ** (port scan từ bên trong), truy cập DB/Redis/admin panel chỉ mở nội bộ.
- **Vượt qua firewall**: request đi *từ trong ra*, được tường lửa tin tưởng.
- **Đọc file cục bộ** qua scheme `file://`, hoặc dùng `gopher://` để gửi payload tới service khác.

> **Blind SSRF:** ngay cả khi response không trả về cho kẻ tấn công, chúng vẫn xác nhận được mục tiêu
> qua kênh phụ (vd server tự gọi tới domain do chúng kiểm soát — quan sát DNS/HTTP log).

#### Cách phòng chống

- **Allowlist** domain/IP được phép gọi (whitelist) — biện pháp mạnh nhất; tránh dùng blacklist.
- **Chặn dải IP nội bộ & đặc biệt**: `127.0.0.0/8`, `10/8`, `172.16/12`, `192.168/16`,
  `169.254.169.254` (metadata), `::1`...
- **Chỉ cho phép scheme an toàn** (`http`/`https`), chặn `file://`, `gopher://`, `dict://`...
- **Validate sau khi resolve DNS** và **cẩn thận với redirect** — kẻ tấn công có thể dùng **DNS
  rebinding** hoặc redirect để vượt qua kiểm tra ban đầu (resolve domain hợp lệ → đổi sang IP nội bộ).
- **Không trả raw response** của request nội bộ về cho client.
- **Chặn metadata endpoint** ở tầng hạ tầng (vd dùng **IMDSv2** yêu cầu token trên AWS).
- Cho service đi qua **proxy/egress gateway** kiểm soát đích ra ngoài; áp **network segmentation**.

### 🎯 Tóm tắt trả lời phỏng vấn

- **SSRF** = lừa **server** gửi request tới đích kẻ tấn công chọn (khác CSRF dùng *trình duyệt* nạn
  nhân). Xảy ra khi app **lấy URL từ người dùng rồi tự gọi** (tải ảnh, webhook, PDF, URL preview).
- **Nguy hiểm vì** server ở vị trí tin cậy, với tới **mạng nội bộ**: đọc **cloud metadata
  `169.254.169.254`** để lấy credential, truy cập DB/Redis nội bộ, port scan, vượt firewall, đọc file
  (`file://`).
- **Blind SSRF**: không cần response trực tiếp, xác nhận qua DNS/HTTP out-of-band.
- **Chống (cốt lõi):** **allowlist domain/IP**, **chặn dải IP nội bộ + metadata IP**, chỉ cho
  `http`/`https`, cẩn thận **redirect & DNS rebinding**, không trả raw response, dùng **IMDSv2** + egress
  proxy/network segmentation.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. Phân biệt authentication và authorization

### 📖 Giới thiệu kiến thức

Hai khái niệm nghe giống nhau (đều bắt đầu bằng "auth") nhưng **trả lời hai câu hỏi khác nhau** và diễn
ra ở **hai bước khác nhau** trong luồng bảo mật:

- **Authentication (AuthN) — "Bạn là ai?"**: **xác minh danh tính**. Người dùng chứng minh mình đúng là
  người họ nói (nhập mật khẩu, OTP, token, vân tay...). Kết quả: hệ thống *tin* bạn là ai. Các cơ chế
  authentication xem [câu 9](#q9).
- **Authorization (AuthZ) — "Bạn được phép làm gì?"**: sau khi biết bạn là ai, **kiểm tra quyền** truy
  cập tài nguyên/hành động (đọc file này? xoá user kia? vào trang admin?).

#### Thứ tự luôn cố định: AuthN trước, AuthZ sau

Phải biết **bạn là ai** rồi mới quyết định được **bạn được phép gì**. Không thể phân quyền cho một danh
tính chưa xác thực.

```text
Request → [Authentication] xác minh danh tính → [Authorization] kiểm tra quyền → cho/không cho truy cập
```

#### Ví dụ đời thực

Vào một toà nhà văn phòng:
- **Authentication** = bảo vệ kiểm tra **thẻ nhân viên** để biết bạn là ai (đúng là nhân viên công ty).
- **Authorization** = thẻ đó có **mở được cửa phòng server** hay không (bạn có quyền vào khu vực đó không).

Là nhân viên hợp lệ (đã authenticated) **không** đồng nghĩa được vào mọi phòng (authorized).

#### Trong ứng dụng web

- **AuthN**: login → server cấp **session/JWT** đại diện danh tính (xem [câu 9](#q9)).
- **AuthZ**: mỗi request tới tài nguyên nhạy cảm, server kiểm **role/permission** (RBAC), thuộc tính
  (ABAC), hoặc **ownership** (bản ghi này có thuộc về bạn không).

```php
// Bước AuthN: xác minh danh tính khi login
if (! Auth::attempt(['email' => $email, 'password' => $password])) {
    return response('Sai thông tin đăng nhập', 401);   // 401: chưa xác thực được
}

// Bước AuthZ: đã biết là ai, kiểm tra có quyền không
if (! $user->can('delete', $post)) {
    return response('Không có quyền', 403);             // 403: đã authn nhưng bị cấm
}
```

#### HTTP status hay bị hỏi: 401 vs 403

- **401 Unauthorized** — thực chất là **"unauthenticated"**: chưa đăng nhập / token sai/hết hạn → *chưa
  biết bạn là ai*.
- **403 Forbidden** — **đã authenticated nhưng không đủ quyền** (authz thất bại) → *biết bạn là ai nhưng
  không cho phép*.

#### Bảng so sánh

| | Authentication (AuthN) | Authorization (AuthZ) |
|--|--|--|
| Câu hỏi | **Bạn là ai?** | **Bạn được phép làm gì?** |
| Mục đích | Xác minh danh tính | Kiểm tra/cấp quyền truy cập |
| Thứ tự | **Trước** | **Sau** (cần authn xong) |
| Dựa vào | Credentials: mật khẩu, OTP, token | Roles / permissions / policies |
| Người dùng | Chủ động cung cấp & biết | Do hệ thống quyết, thường ẩn với user |
| HTTP lỗi | **401** Unauthorized | **403** Forbidden |
| Cơ chế | Password, JWT, OAuth, MFA... | RBAC, ABAC, ACL, ownership check |
| Lỗ hổng | **Broken Authentication** ([câu 10](#q10)) | **Broken Access Control / IDOR** ([câu 2](#q2)) |

> Trong mô hình **AAA** (Authentication, Authorization, Accounting), đây là hai chữ "A" đầu; chữ thứ ba
> **Accounting** = ghi log/theo dõi những gì người dùng đã làm.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Authentication = "bạn là ai?"** (xác minh danh tính — login bằng mật khẩu/OTP/token). **Authorization
  = "bạn được phép làm gì?"** (kiểm tra quyền truy cập tài nguyên/hành động).
- **Thứ tự cố định:** authN **trước**, authZ **sau** — phải biết là ai rồi mới xét quyền.
- **Ví dụ:** thẻ nhân viên chứng minh bạn là ai (**authN**); thẻ có mở được phòng server không (**authZ**).
  Là nhân viên hợp lệ không có nghĩa vào được mọi phòng.
- **401 vs 403 (hay hỏi):** **401** = chưa xác thực (chưa biết bạn là ai); **403** = đã xác thực nhưng
  **không đủ quyền**.
- **Lỗ hổng tương ứng:** authN yếu → **Broken Authentication** ([câu 10](#q10)); authZ yếu → **Broken
  Access Control / IDOR** ([câu 2](#q2)). Cơ chế authZ điển hình: **RBAC/ABAC**, kiểm tra ownership server-side.

[↑ Mục lục](#mục-lục)

<a id="q9"></a>
## 9. Có mấy cách authentication (xác thực)?

### 📖 Giới thiệu kiến thức

**Authentication (xác thực)** trả lời câu hỏi **"bạn là ai?"** — xác minh danh tính người/hệ thống
trước khi cho truy cập. (Khác **Authorization** = "bạn được phép làm gì?" — phân quyền, xem [câu 8](#q8).)

#### Phân loại theo yếu tố xác thực (authentication factors)

- **Something you know** — cái bạn *biết*: mật khẩu, PIN, câu hỏi bí mật.
- **Something you have** — cái bạn *có*: điện thoại (OTP), security key, thẻ.
- **Something you are** — cái bạn *là*: sinh trắc học (vân tay, khuôn mặt, mống mắt).

> Kết hợp **≥ 2 nhóm khác nhau** = **MFA / 2FA** (Multi-Factor Authentication). Dùng 2 mật khẩu **không**
> phải MFA vì cùng một yếu tố ("know").

#### Các cơ chế authentication phổ biến trong web/API

1. **HTTP Basic / Digest Auth** — gửi `username:password` (Basic mã hoá base64) trong header
   `Authorization: Basic ...`. Rất đơn giản nhưng **bắt buộc HTTPS** (base64 không phải mã hoá); ít dùng
   cho ứng dụng người dùng, hay gặp ở API nội bộ/tool.

2. **Session-based (Cookie-based) — *stateful***. Sau khi login, **server tạo session** và lưu ở session
   store (memory/Redis/DB), trả **session id trong cookie**. Mỗi request trình duyệt **tự gửi cookie** →
   server tra session để biết là ai. Ưu: **dễ thu hồi** (xoá session), kiểm soát tập trung. Nhược: **khó
   scale ngang** (cần shared store), cookie tự gửi → **phải chống CSRF** (xem [câu 5](#q5)).

3. **Token-based (JWT) — *stateless***. Sau login, server cấp **JWT** ký số (`header.payload.signature`).
   Client lưu và gửi qua header **`Authorization: Bearer <token>`**. Server chỉ **verify chữ ký**, không
   cần lưu state → **scale tốt**, hợp mobile/microservice/cross-domain. Nhược: **khó thu hồi trước hạn**
   (dùng short TTL + refresh token, hoặc blacklist). Lưu token ở đâu: xem [câu 10.1](#q10-1).

   ```http
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   ```

4. **OAuth 2.0 / OpenID Connect (OIDC)** — **uỷ quyền (delegated)**. **OAuth2** cho phép app truy cập tài
   nguyên thay user mà **không cần biết mật khẩu** (cấp access token). **OIDC** xây trên OAuth2 để **xác
   thực danh tính** (id_token). Đây là nền của **"Login with Google/Facebook/GitHub"**.

5. **API Key** — chuỗi bí mật cấp cho client/service, gửi kèm request (header/query). Hợp **service-to-
   service** hoặc nhận diện app; **không nên** thay auth người dùng (không gắn danh tính user, khó xoay).

6. **SSO (Single Sign-On)** — đăng nhập **một lần** dùng được cho nhiều ứng dụng, qua **SAML** (thiên
   doanh nghiệp) hoặc **OIDC**. Một **IdP (Identity Provider)** xác thực tập trung.

7. **MFA / 2FA** — thêm lớp thứ hai: **OTP/TOTP** (Google Authenticator), SMS/email code, **push
   notification**, hoặc **security key**. Giảm mạnh rủi ro lộ mật khẩu.

8. **Passwordless / WebAuthn / Passkeys (FIDO2)** — bỏ mật khẩu: **magic link**, **OTP**, hoặc **passkey**
   dùng khoá công khai + sinh trắc học trên thiết bị. **Chống phishing tốt**, đang là xu hướng.

9. **Certificate-based / mTLS** — client trình **chứng chỉ số**, hai bên xác thực lẫn nhau. Dùng cho hệ
   thống bảo mật cao, service mesh (zero-trust).

#### Stateful (session) vs Stateless (token) — rất hay hỏi

| | Session-based | Token-based (JWT) |
|--|--|--|
| Lưu state | **Server lưu** session | **Không** (token tự chứa) |
| Truyền | Cookie (session id) | Header `Authorization: Bearer` |
| Scale ngang | Cần shared store | Dễ (stateless) |
| Thu hồi | **Dễ** (xoá session) | **Khó** (cần blacklist / short TTL) |
| Rủi ro chính | **CSRF** (cookie tự gửi) | **XSS** nếu lưu token ở storage |

### 🎯 Tóm tắt trả lời phỏng vấn

- **Authentication = "bạn là ai?"**, phân theo **3 yếu tố**: cái bạn **biết** (mật khẩu), **có**
  (OTP/security key), **là** (sinh trắc học); kết hợp **≥2 yếu tố khác nhau = MFA**.
- Cơ chế web/API phổ biến: **HTTP Basic**, **Session/Cookie (stateful)**, **Token/JWT (stateless,
  `Bearer`)**, **OAuth2/OIDC** (login-with-Google, uỷ quyền), **API key** (service-to-service), **SSO**
  (SAML/OIDC), **MFA/2FA**, **passwordless/WebAuthn-Passkey**, **mTLS**.
- **Session vs Token (hay hỏi):** session lưu state server + cookie → **dễ thu hồi nhưng khó scale &
  dính CSRF**; JWT stateless + header `Bearer` → **scale tốt nhưng khó thu hồi & rủi ro XSS-storage**.
- Chốt: chọn cơ chế theo bối cảnh (web truyền thống → **session**; API/mobile/microservice → **token/
  OAuth**), luôn **HTTPS**, thêm **MFA** cho tài khoản quan trọng. Cơ chế yếu → sinh lỗ hổng **Broken
  Authentication** (xem [câu 10](#q10)).

[↑ Mục lục](#mục-lục)

<a id="q9-1"></a>
### 9.1. Basic Authentication là gì?

#### 📖 Giới thiệu kiến thức

**HTTP Basic Authentication** (RFC 7617) là cách xác thực **đơn giản nhất** của HTTP: client gửi thẳng
**`username:password`** trong header ở **mỗi request**.

**Cách hoạt động:**

```text
1. Client gọi API không kèm credential
2. Server trả 401 + header:  WWW-Authenticate: Basic realm="..."   (trình duyệt hiện popup nhập user/pass)
3. Client gửi lại kèm:       Authorization: Basic base64(username:password)
4. Server decode base64 → kiểm tra user/pass → cho qua hoặc 401
```

```http
Authorization: Basic Y3VvbTpzZWNyZXQ=      # base64("cuom:secret")
```

- **`base64` KHÔNG phải mã hoá** — ai bắt được request cũng giải ra mật khẩu → **bắt buộc HTTPS**.
- **Stateless**: server không lưu phiên; credential được gửi lại **mỗi request**.

**Ưu điểm:**
- **Cực đơn giản**, là chuẩn HTTP sẵn có, không cần DB/session store.
- Hợp **API nội bộ, tool, server-to-server**, hoặc bảo vệ nhanh một endpoint.

**Nhược điểm:**
- **Gửi mật khẩu ở mọi request** → bề mặt lộ lớn; **vô dụng nếu không có HTTPS** (base64 dễ giải).
- **Không có logout thật** (trình duyệt cache credential), **không hết hạn / không thu hồi** được.
- Không hợp **user-facing** (UX popup, khó thêm MFA, khó reset).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Basic Auth** = gửi `Authorization: Basic base64(user:pass)` ở **mỗi request**; server decode & verify.
  **Stateless**, `base64` **không phải mã hoá** → **bắt buộc HTTPS**.
- **Ưu:** cực đơn giản, chuẩn HTTP, không cần session store — hợp **API nội bộ/tool/server-to-server**.
- **Nhược:** gửi mật khẩu mỗi request, **không logout/hết hạn/thu hồi**, dễ lộ nếu thiếu HTTPS, không hợp
  ứng dụng người dùng (khó MFA).

[↑ Mục lục](#mục-lục)

<a id="q9-2"></a>
### 9.2. Session-based Authentication là gì?

#### 📖 Giới thiệu kiến thức

**Session-based Authentication** là mô hình **stateful**: sau khi đăng nhập, **server giữ trạng thái
phiên**, client chỉ cầm một **session id** để tham chiếu. (Chi tiết Session/Cookie xem
[common.md — Session, Cookie, Cache](./common.md#q4).)

**Cách hoạt động:**

```text
1. Client POST /login (user + pass)
2. Server xác thực → tạo Session (lưu ở server: RAM/Redis/DB) → cấp session id
3. Server trả:  Set-Cookie: SID=abc123; HttpOnly; Secure; SameSite=Lax
4. Request sau: trình duyệt TỰ gửi  Cookie: SID=abc123
5. Server tra session store bằng SID → biết là ai (không cần login lại)
6. Logout → server xoá session; cookie thành vô nghĩa
```

**Ưu điểm:**
- **Dễ thu hồi**: xoá session ở server là **đăng xuất tức thì** (chủ động huỷ được mọi phiên).
- Server **kiểm soát hoàn toàn**; **session id opaque** (không chứa data → không lộ thông tin).
- Cookie **`HttpOnly`** chống JS đọc trộm; đơn giản, hợp **web truyền thống** (server-rendered).

**Nhược điểm:**
- **Stateful** → phải lưu session store; **khó scale ngang**: nhiều server cần **shared store** (Redis)
  hoặc **sticky session**.
- Cookie **tự gửi kèm** → **dính CSRF** ([câu 5](#q5)) → cần `SameSite`/CSRF token.
- Kém tự nhiên cho **mobile / cross-domain / microservice**; mỗi request phải **tra store** (overhead).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Session-based (stateful):** login → **server lưu session** + cấp **session id qua cookie**; mỗi
  request server **tra store** để biết user; logout = **xoá session** ở server.
- **Ưu:** **dễ thu hồi** (huỷ phiên tức thì), server kiểm soát, session id **opaque**, cookie `HttpOnly`.
- **Nhược:** **stateful → khó scale** (cần shared store/Redis, sticky session), **dính CSRF** (cookie tự
  gửi), kém hợp mobile/cross-domain, tra store mỗi request.

[↑ Mục lục](#mục-lục)

<a id="q9-3"></a>
### 9.3. Token-based Authentication là gì?

#### 📖 Giới thiệu kiến thức

**Token-based Authentication** là mô hình **stateless**: sau khi đăng nhập, server cấp một **token** (phổ
biến là **JWT**) **tự chứa** thông tin; client tự lưu và **đính token vào mỗi request**. Server chỉ **verify
token** là biết user — **không cần lưu phiên** ở server.

**Cách hoạt động:**

```text
1. Client POST /login (user + pass)
2. Server xác thực → ký & cấp TOKEN (JWT), thường kèm refresh token
3. Client lưu token (in-memory / cookie HttpOnly)
4. Request sau: gửi  Authorization: Bearer <token>
5. Server VERIFY chữ ký + hạn (exp) → biết user, KHÔNG tra DB phiên
6. Token hết hạn → dùng refresh token lấy token mới
```

- Token gửi dạng **bearer token** qua header `Authorization: Bearer` ([câu 9.4](#q9-4)); thường là **JWT**
  ([câu 9.5](#q9-5)); quản lý vòng đời bằng **access + refresh token** ([câu 9.11](#q9-11)).

**Ưu điểm:**
- **Stateless → scale ngang dễ** (không cần shared session store); hợp **SPA, mobile, microservice,
  cross-domain**.
- Dùng header `Bearer` (không tự gửi kèm) → **không dính CSRF**; một token dùng cho **nhiều service**.

**Nhược điểm:**
- **Khó thu hồi trước hạn** (token còn hạn là còn dùng được) → cần **short TTL + refresh**, hoặc
  **blacklist** ([câu 9.11](#q9-11)).
- Payload JWT **đọc được** (đừng để secret); token **lớn hơn** session id (gửi mỗi request).
- Lưu ở `localStorage` thì **rủi ro XSS** ([câu 3](#q3)) → cân nhắc nơi lưu ([câu 10.1](#q10-1)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Token-based (stateless):** login → server cấp **token (JWT) tự chứa**; client gửi
  **`Authorization: Bearer <token>`** mỗi request; server **verify chữ ký** (không tra DB phiên).
- **Ưu:** **stateless → scale dễ**, hợp **mobile/SPA/microservice/cross-domain**, không dính CSRF (dùng
  header), 1 token cho nhiều service.
- **Nhược:** **khó thu hồi** (cần short TTL + refresh / blacklist), token to hơn session id, payload đọc
  được, rủi ro **XSS** nếu lưu sai chỗ.
- **So với session:** session **stateful, dễ thu hồi nhưng khó scale + CSRF**; token **stateless, scale
  tốt nhưng khó thu hồi + rủi ro XSS-storage**.

[↑ Mục lục](#mục-lục)

<a id="q9-4"></a>
### 9.4. Bearer token là gì?

#### 📖 Giới thiệu kiến thức

**Bearer** nghĩa là **"người mang/người cầm"**. **Bearer token** là loại token mà **bất kỳ ai đang sở
hữu (bearer) token đó đều dùng được** — server **không đòi hỏi chứng minh thêm** rằng bạn đúng là chủ
hợp lệ. Giống **tiền mặt hoặc vé xem phim**: *ai cầm cũng tiêu/vào được* → **lộ/mất là bị dùng ngay**.

> Đối lập với bearer token là **sender-constrained / Proof-of-Possession (PoP) token** (vd **mTLS-bound**,
> **DPoP**): token bị **gắn với một khoá của client**, nên kẻ trộm được token mà không có khoá thì **không
> dùng được**. Bearer token đơn giản & phổ biến hơn, nhưng rủi ro hơn nếu bị lộ.

#### Cách truyền: header `Authorization: Bearer`

Chuẩn hoá trong **RFC 6750** (OAuth2). Client gắn token vào header:

```http
GET /api/me HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

- Token có thể là **JWT** (tự chứa, verify bằng chữ ký — xem [câu 9.5](#q9-5)) **hoặc opaque** (chuỗi
  ngẫu nhiên, server tra store để biết là ai).
- **Access token** trong OAuth2 mặc định là **bearer token** (xem [câu 9.15](#q9-15)).

#### Vì "ai cầm cũng dùng được" → phải bảo vệ kỹ

- **Bắt buộc HTTPS/TLS** — nếu không, token bị **sniff** trên đường truyền là dùng được ngay.
- **Thời hạn ngắn** (access token ngắn hạn + refresh) → giảm thiệt hại khi lộ (xem [câu 9.11](#q9-11)).
- **Lưu an toàn ở client** — tránh `localStorage` nếu lo **XSS**; ưu tiên in-memory / cookie `HttpOnly`
  (xem [câu 10.1](#q10-1)).
- **KHÔNG đặt token trên URL/query** — bị ghi vào **log, history, header `Referer`** → rò rỉ.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Bearer token** = token mà **ai sở hữu cũng dùng được**, server không đòi chứng minh thêm — như **tiền
  mặt / vé**: lộ là bị dùng ngay.
- **Truyền qua header `Authorization: Bearer <token>`** (RFC 6750). Token có thể là **JWT** (tự chứa) hoặc
  **opaque** (tra store). Access token của OAuth2 mặc định là bearer.
- Vì rủi ro khi lộ nên: **bắt buộc HTTPS**, **thời hạn ngắn**, **lưu an toàn** (không localStorage/URL).
- Đối lập: **sender-constrained token** (mTLS-bound, **DPoP**) — gắn khoá client nên kẻ trộm token không
  dùng được; an toàn hơn nhưng phức tạp hơn.

[↑ Mục lục](#mục-lục)

<a id="q9-5"></a>
### 9.5. JWT gồm những phần nào? Verify thế nào?

#### 📖 Giới thiệu kiến thức

**JWT (JSON Web Token)** là một chuẩn (RFC 7519) để đóng gói thông tin (**claims**) thành một token
**gọn, tự chứa (self-contained)** và **ký số** để chống giả mạo. Nhờ *tự chứa*, server chỉ cần **verify
chữ ký** là biết token hợp lệ — **không cần tra database/session** → nền tảng của **authentication
stateless** (xem cơ chế token ở [câu 9](#q9)).

#### Cấu trúc: 3 phần, ngăn bởi dấu chấm

Một JWT có dạng `xxxxx.yyyyy.zzzzz` — **`Header.Payload.Signature`**, mỗi phần **Base64URL-encode**.

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   .   eyJzdWIiOiIxMjMiLCJyb2xlIjoiYWRtaW4iLCJleHAiOjE3MDB9   .   3Rk...chữ_ký
        Header (base64url)                            Payload (base64url)                             Signature
```

**1. Header** — metadata: thuật toán ký & loại token.

```json
{ "alg": "HS256", "typ": "JWT" }
```

**2. Payload** — chứa **claims** (thông tin). Ba nhóm:
- **Registered claims** (chuẩn, viết tắt): `iss` (issuer), `sub` (subject/user id), `aud` (audience),
  `exp` (hết hạn), `iat` (issued at), `nbf` (not before), `jti` (id token).
- **Public / Private claims** — tự định nghĩa: `userId`, `role`, `email`...

```json
{ "sub": "123", "role": "admin", "exp": 1700000000 }
```

> ⚠️ **Payload chỉ Base64 — KHÔNG mã hoá.** Bất kỳ ai cũng **đọc được** nội dung. **Không** để dữ liệu
> nhạy cảm (mật khẩu, thẻ) trong payload. Ký số chỉ đảm bảo **toàn vẹn (integrity)**, không phải bí mật.

**3. Signature** — "con dấu" chống sửa: ký lên `base64url(header) + "." + base64url(payload)` bằng khoá:

```text
HMACSHA256( base64url(header) + "." + base64url(payload), secret )
```

Chỉ đổi 1 ký tự ở header/payload → chữ ký không khớp → token bị từ chối.

#### Verify (xác minh) một JWT — các bước

1. **Tách** token thành 3 phần theo dấu `.`.
2. **Tự tính lại chữ ký** của `header.payload` bằng khoá server giữ, rồi **so khớp** với signature trong
   token (so sánh **constant-time** để tránh timing attack — xem [câu 14](#q14)). Khớp → token **chưa bị sửa** & do đúng bên
   phát hành ký.
3. **Kiểm tra claims:** `exp` (chưa hết hạn), `nbf`/`iat` hợp lệ, `iss`/`aud` đúng kỳ vọng.

**Hai kiểu ký & cách verify:**
- **HS256 (HMAC — đối xứng):** **một secret** dùng cho **cả ký và verify**. Đơn giản, nhưng mọi bên verify
  đều phải giữ secret → chỉ hợp khi issuer = verifier (cùng service).
- **RS256 (RSA — bất đối xứng):** **private key ký**, **public key verify**. Bên verify (Resource Server)
  chỉ cần **public key** → hợp **nhiều bên / microservice / OAuth2** (không phải chia sẻ secret).

```php
// Ví dụ khái niệm với firebase/php-jwt
use Firebase\JWT\JWT;
use Firebase\JWT\Key;

// Phát hành (sign)
$token = JWT::encode(['sub' => 123, 'role' => 'admin', 'exp' => time() + 900], $secret, 'HS256');

// Verify — sai chữ ký / hết hạn sẽ ném exception
$decoded = JWT::decode($token, new Key($secret, 'HS256'));  // luôn CHỈ ĐỊNH alg mong đợi
```

#### Bẫy bảo mật hay bị hỏi

- **`alg: none`**: kẻ tấn công đổi header thành `{"alg":"none"}` và bỏ chữ ký. Thư viện/cấu hình phải
  **từ chối** token không ký.
- **Algorithm confusion (RS256 → HS256):** attacker đổi `alg` sang HS256 rồi dùng **public key làm secret
  HMAC**. Chống bằng cách **luôn chỉ định (pin) alg mong đợi** ở phía verify, không tin `alg` trong token.
- **Không thu hồi được trước hạn:** JWT hợp lệ tới khi `exp`. Giảm thiểu bằng **access token ngắn hạn +
  refresh token**, hoặc **blacklist/denylist** theo `jti`.
- **Lưu token an toàn** ở client: xem [câu 10.1](#q10-1) (in-memory / cookie `HttpOnly`), rủi ro **XSS**
  ([câu 3](#q3)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **JWT = 3 phần `Header.Payload.Signature`**, mỗi phần **Base64URL**. **Header**: `alg`+`typ`; **Payload**:
  các **claims** (`sub`, `role`, `exp`, `iss`, `aud`...); **Signature**: ký lên `header.payload` bằng khoá.
- **Payload chỉ base64, KHÔNG mã hoá** → ai cũng đọc được; đừng để dữ liệu nhạy cảm. Ký chỉ đảm bảo **toàn
  vẹn**, không bí mật.
- **Verify:** tách 3 phần → **tự ký lại `header.payload` bằng khoá và so khớp signature** (constant-time)
  → **kiểm claims** `exp`/`nbf`/`iss`/`aud`.
- **HS256** (HMAC, 1 secret ký & verify — cùng service) vs **RS256** (private ký / public verify — hợp
  nhiều bên, microservice, OAuth2).
- **Bẫy:** chặn **`alg: none`**, chống **algorithm confusion** (luôn **pin alg** mong đợi), JWT **khó thu
  hồi** → dùng token ngắn hạn + refresh/blacklist.

[↑ Mục lục](#mục-lục)

<a id="q9-6"></a>
### 9.6. Tại sao JWT chia 3 phần mà không chỉ dùng signature?

#### 📖 Giới thiệu kiến thức

Câu hỏi ngầm giả định "signature là đủ", nhưng **signature một mình vô nghĩa**. Nhớ lại cấu trúc
`Header.Payload.Signature` ([câu 9.5](#q9-5)), **mỗi phần một vai trò không thể thiếu:**

- **Payload** — chở **claims** (bạn là ai, quyền gì, `exp`...). **Đây mới là thông tin** JWT muốn truyền.
  Bỏ payload → token **rỗng**, server không biết là ai → phải **tra DB lại** → **mất tính self-contained
  (stateless)** — vốn là lý do dùng JWT.
- **Header** — khai báo **cách verify**: thuật toán `alg` (HS256/RS256 — [câu 9.8](#q9-8)) và `kid` (chọn
  đúng khoá khi có nhiều khoá/JWKS). Không có header → verifier **không biết kiểm chữ ký bằng cách nào**.
- **Signature** — **con dấu chống giả mạo** cho **header + payload**. Nó *bảo vệ* nội dung chứ **không
  chứa** nội dung.

#### Vì sao signature một mình không thay thế được cả token?

**Signature = `Sign( base64(header) + "." + base64(payload) )`** — nó là **hàm băm + ký của hai phần
kia**, không phải dữ liệu độc lập. Hệ quả:

1. **Không có thông tin để dùng.** Signature chỉ là chuỗi ngẫu nhiên ~256 bit; **không đọc ra được** user
   id, quyền hay hạn dùng. Server cần **payload** mới biết cấp quyền cho ai.
2. **Không thể verify nếu thiếu input gốc.** Cách verify là **server TỰ TÍNH LẠI** signature từ
   `header.payload` rồi **so khớp**. Nếu chỉ gửi mỗi signature → **không có input để tính lại** → không
   đối chiếu được. Bắt buộc phải gửi kèm header + payload ở dạng **tái tạo được y hệt** (nên dùng
   **Base64URL**, không mã hoá).
3. **Không biết thuật toán verify.** Thiếu header thì không rõ dùng HMAC hay RSA, khoá nào → phải hardcode,
   mất **linh hoạt** (đổi thuật toán/xoay khoá).

> **Ẩn dụ:** JWT như một **hợp đồng có đóng dấu**. **Payload** = nội dung hợp đồng, **Header** = ghi chú
> "dấu này kiểu gì, của phòng ban nào", **Signature** = con dấu. Chỉ đưa **mỗi con dấu** mà không có tờ
> hợp đồng thì người nhận **không biết nó niêm phong cho cái gì** và **không đối chiếu được**.

- Cũng vì vậy header/payload để **Base64 (đọc được), không mã hoá**: mục tiêu là **toàn vẹn (integrity)**
  — chứng minh *chưa bị sửa* — chứ không phải giữ bí mật (đừng để secret trong payload).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Signature không *chứa* thông tin, nó chỉ *bảo vệ* thông tin.** Ba phần ba vai trò: **Payload** =
  dữ liệu (claims), **Header** = cách verify (`alg`/`kid`), **Signature** = chống giả mạo.
- **Signature = ký của `header.payload`** → muốn verify, server phải **tự tính lại từ header+payload rồi
  so khớp**. Chỉ có signature thì **không có input để tái tạo** → không verify được, và **không đọc ra**
  user/quyền.
- Bỏ payload → **token rỗng, mất self-contained** (phải tra DB). Bỏ header → **không biết thuật toán/khoá**
  để verify.
- Header/payload để **Base64 không mã hoá** vì mục tiêu là **toàn vẹn**, không phải bí mật.

[↑ Mục lục](#mục-lục)

<a id="q9-7"></a>
### 9.7. Payload của JWT thường nên chứa những gì?

#### 📖 Giới thiệu kiến thức

Vì **payload chỉ Base64, ai cũng đọc được** và **token gửi kèm mỗi request** (xem [câu 9.5](#q9-5),
[câu 9.6](#q9-6)), nguyên tắc chọn nội dung payload là: **đủ để xác thực & phân quyền stateless**,
**nhỏ gọn**, và **không chứa bí mật**.

#### NÊN chứa

**Registered claims (chuẩn — nên có):**
- **`sub`** — subject: **định danh chủ thể** (thường là user id).
- **`iss`** — issuer: **ai phát hành** token (auth server nào).
- **`aud`** — audience: **token dành cho service/API nào** (verifier kiểm để không nhận token của hệ khác).
- **`exp`** — expiration: **thời điểm hết hạn** (gần như **bắt buộc** — token phải có vòng đời).
- **`iat`** — issued at; **`nbf`** — not before (chưa hiệu lực trước mốc này).
- **`jti`** — JWT id: mã duy nhất → phục vụ **revoke / chống replay** (xem [câu 9.11](#q9-11)).

**Custom claims — thông tin định danh & phân quyền *tối thiểu*:**
- `role` / `scope` / `permissions` — để **authorize nhanh** ngay tại verifier, không cần query DB.
- `tenant_id` (multi-tenant), `email`/`name` **nếu thật sự cần** & không nhạy cảm.

```json
{
  "sub": "1234",
  "iss": "https://auth.myapp.com",
  "aud": "myapp-api",
  "role": "admin",
  "scope": ["read:orders", "write:orders"],
  "iat": 1700000000,
  "exp": 1700000900,
  "jti": "a1b2c3d4"
}
```

#### KHÔNG nên chứa

- **Dữ liệu nhạy cảm / bí mật:** mật khẩu, password hash, số thẻ, PII nhạy cảm, secret, API key —
  vì payload **đọc được** (chỉ Base64). Muốn giấu thì phải **JWE** ([câu 9.10](#q9-10)); liên hệ
  [câu 12](#q12).
- **Dữ liệu lớn:** payload to → token to → tốn băng thông vì gửi **mỗi request**.
- **Dữ liệu hay đổi / cần realtime:** token **bất biến tới khi hết hạn** → nếu nhét `role`, trạng thái
  ban/khoá... rồi đổi ở DB, **token cũ vẫn mang giá trị cũ (stale)**. Thông tin cần cập nhật tức thì phải
  **kiểm ở server**, đừng tin claim trong token.

> Nguyên tắc chốt: payload = **"tấm thẻ" tối thiểu để nhận diện + phân quyền** (id + role/scope + exp),
> **không phải nơi lưu hồ sơ user** hay dữ liệu bí mật.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Nên có:** registered claims **`sub`** (user id), **`iss`**, **`aud`**, **`exp`** (gần như bắt buộc),
  `iat`/`nbf`, **`jti`** (revoke/replay); + custom **tối thiểu** như **`role`/`scope`** để authorize nhanh.
- **KHÔNG chứa:** **dữ liệu nhạy cảm/bí mật** (payload chỉ Base64, đọc được → muốn giấu dùng **JWE**),
  **dữ liệu lớn** (token gửi mỗi request), **dữ liệu hay đổi/realtime** (token stale tới khi hết hạn).
- **Nguyên tắc:** payload **nhỏ gọn, không nhạy cảm, đủ để phân quyền stateless** — là "tấm thẻ" nhận
  diện, không phải nơi lưu hồ sơ user.

[↑ Mục lục](#mục-lục)

<a id="q9-8"></a>
### 9.8. Phân biệt HS256, RS256 và ES256

#### 📖 Giới thiệu kiến thức

Cả ba đều là **thuật toán ký JWT** (khai báo ở claim `alg` trong header) và **đều băm bằng SHA-256** —
khác nhau ở **cách ký/verify** (xem cấu trúc & verify JWT ở [câu 9.5](#q9-5)):

- **HS256 = HMAC + SHA-256 (đối xứng — symmetric).** Dùng **một secret key duy nhất** cho **cả ký lẫn
  verify**. Ai có secret thì vừa tạo được token vừa kiểm tra được.
- **RS256 = RSA signature + SHA-256 (bất đối xứng — asymmetric).** Dùng **cặp khoá**: **private key ký**,
  **public key verify**. Bên verify chỉ cần **public key** (không nhạy cảm).
- **ES256 = ECDSA (đường cong P-256) + SHA-256 (bất đối xứng).** Cũng **private ký / public verify** như
  RS256, nhưng dựa trên **Elliptic Curve Cryptography (ECC)** → **khoá & chữ ký nhỏ hơn nhiều**, ký nhanh.

#### Bảng so sánh

| Tiêu chí | **HS256** | **RS256** | **ES256** |
|----------|-----------|-----------|-----------|
| Loại | Đối xứng (HMAC) | Bất đối xứng (RSA) | Bất đối xứng (ECDSA) |
| Khoá | **1 secret** dùng chung | **private** ký + **public** verify (RSA) | **private** ký + **public** verify (EC) |
| Ai verify được | Chỉ bên **có secret** | Ai có **public key** | Ai có **public key** |
| Chia sẻ khoá | Chia secret → **rủi ro** | Chỉ chia **public** (an toàn) | Chỉ chia **public** (an toàn) |
| Kích thước khoá | Ngắn | **Lớn** (≥ 2048-bit) | **Nhỏ** (256-bit ≈ RSA 3072-bit) |
| Kích thước chữ ký | Nhỏ | **Lớn** | **Nhỏ** |
| Tốc độ | Nhanh nhất | Ký **chậm**, verify nhanh | **Ký nhanh**, gọn nhẹ |
| Hợp với | **Issuer = verifier** (1 service) | Nhiều bên / OAuth2 (**phổ biến nhất**) | Nhiều bên, cần **gọn/nhanh** (mobile, IoT) |

#### Khi nào dùng cái nào?

- **HS256** — khi **bên ký cũng là bên verify** (monolith, cùng một backend giữ secret). Đơn giản, nhanh.
  Nhược: muốn service khác verify thì **phải chia sẻ secret** → lộ secret là **giả mạo được token**.
- **RS256** — khi **bên verify khác bên ký**: **microservice, third-party, OAuth2/OIDC**. Auth server ký
  bằng **private key**; các resource server verify bằng **public key** (phân phối qua **JWKS endpoint**).
  Không phải chia sẻ bí mật → an toàn khi mở rộng. Là mặc định **phổ biến nhất** cho hệ phân tán.
- **ES256** — **cùng mục đích như RS256** (private ký / public verify, hợp nhiều bên) nhưng dùng **ECC**:
  khoá **256-bit** cho độ an toàn tương đương **RSA 3072-bit**, **chữ ký nhỏ hơn**, **ký nhanh** → hợp
  **mobile, IoT, token cần gọn, hệ high-throughput**. Ngày càng được ưu tiên cho hệ mới.

> **ECC là gì?** Elliptic Curve Cryptography — mã hoá khoá công khai dựa trên đường cong elliptic, đạt
> **cùng mức an toàn với khoá ngắn hơn nhiều** so với RSA. ES256 dùng đường cong **P-256 (secp256r1)**.
>
> ⚠️ **Lưu ý ECDSA:** mỗi lần ký cần một **số ngẫu nhiên `k` (nonce) bí mật**; nếu **k bị lặp hoặc RNG
> yếu** thì **lộ private key** (sự cố PS3 kinh điển). Thư viện uy tín xử lý bằng **nonce tất định RFC 6979**
> — nên **dùng lib chuẩn, đừng tự cài ECDSA**.

#### Ví dụ cụ thể (code)

**HS256 — một secret cho cả ký & verify** (bên ký = bên verify):

```js
const jwt = require('jsonwebtoken');
const secret = process.env.JWT_SECRET;        // MỘT secret duy nhất, giữ kín ở server

// Ký
const token = jwt.sign({ sub: 123, role: 'admin' }, secret,
                       { algorithm: 'HS256', expiresIn: '15m' });

// Verify — dùng CHÍNH secret đó (bên verify buộc phải biết secret)
const payload = jwt.verify(token, secret, { algorithms: ['HS256'] }); // luôn pin alg
```

**RS256 — private ký, public verify** (cặp khoá tách biệt):

```bash
# Sinh cặp khoá RSA
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

```js
const jwt = require('jsonwebtoken');
const privateKey = fs.readFileSync('private.pem'); // CHỈ auth server giữ (bí mật)
const publicKey  = fs.readFileSync('public.pem');  // phát cho MỌI bên verify (công khai)

// Auth server KÝ bằng private key
const token = jwt.sign({ sub: 123, role: 'admin' }, privateKey,
                       { algorithm: 'RS256', expiresIn: '15m' });

// Resource server VERIFY chỉ cần public key — không có private nên KHÔNG giả mạo được token
const payload = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

**ES256 — giống RS256 nhưng cặp khoá EC (nhỏ hơn):**

```bash
# Sinh cặp khoá EC đường cong P-256 (thay cho RSA)
openssl ecparam -name prime256v1 -genkey -noout -out ec-private.pem
openssl ec -in ec-private.pem -pubout -out ec-public.pem
```

```js
// Chỉ đổi khoá sang EC và algorithm sang 'ES256' — API y hệt RS256
const token   = jwt.sign({ sub: 123, role: 'admin' }, ecPrivateKey,
                         { algorithm: 'ES256', expiresIn: '15m' });
const payload = jwt.verify(token, ecPublicKey, { algorithms: ['ES256'] });
```

**Tình huống minh hoạ (vì sao microservice chọn RS256/ES256):**

```text
                 ┌──────────────┐   private.pem (ký)
   Login ──────► │ Auth Service │───────────────────┐
                 └──────────────┘                    ▼  cấp JWT (RS256)
                                                  ┌────────┐
       public.pem ───────────────────────────────┤ Client │
       (JWKS công khai)                           └────────┘
                 ┌───────────────┐                    │ gửi kèm Bearer token
   verify ◄──────┤ Order Service │◄───────────────────┤
   (public key)  └───────────────┘                    │
                 ┌─────────────────┐                   │
   verify ◄──────┤ Payment Service │◄──────────────────┘
   (public key)  └─────────────────┘
```

- **RS256:** chỉ **Auth Service giữ private key**; Order/Payment chỉ cần **public key** để verify (lấy qua
  **JWKS endpoint**). Một service bị xâm nhập cũng **không ký giả** được token.
- **Nếu dùng HS256:** phải **copy cùng một secret** sang cả 3 service → **bất kỳ service nào lộ secret là
  giả mạo được token cho toàn hệ thống**. Đó là lý do hệ phân tán ưu tiên RS256.

#### Bẫy bảo mật: Algorithm confusion (RS256 → HS256)

Nếu server **không cố định (pin) `alg` mong đợi**, kẻ tấn công đổi `alg` từ RS256 sang **HS256** rồi dùng
**public key (vốn công khai) làm secret HMAC** để ký → **giả mạo token**. → **Luôn chỉ định thuật toán
mong đợi** ở phía verify, không tin `alg` trong token (xem [câu 9.5](#q9-5)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả ba ký JWT & **cùng băm SHA-256**; khác ở scheme: **HS256 = HMAC đối xứng** (1 secret ký & verify);
  **RS256 = RSA bất đối xứng** & **ES256 = ECDSA bất đối xứng** (đều private ký / public verify).
- **HS256**: nhanh, đơn giản, hợp khi **issuer = verifier** (cùng service); nhược là **phải chia sẻ
  secret** nếu nhiều bên → rủi ro.
- **RS256**: hợp **nhiều bên / microservice / OAuth2** — chỉ chia **public key** (qua **JWKS**), không lộ
  bí mật; đổi lại **khoá lớn, ký chậm, chữ ký lớn**. Là mặc định phổ biến nhất.
- **ES256**: cùng mô hình public/private như RS256 nhưng dùng **ECC** → **khoá & chữ ký nhỏ hơn nhiều,
  ký nhanh** (256-bit ≈ RSA 3072-bit) → hợp **mobile/IoT/gọn nhẹ**; đang là xu hướng cho hệ mới.
- **Nhớ:** luôn **pin `alg`** để chống **algorithm confusion (RS256→HS256)**.

[↑ Mục lục](#mục-lục)

<a id="q9-9"></a>
### 9.9. Nếu phải "pin" alg, thì header `alg` của JWT còn tác dụng gì?

#### 📖 Giới thiệu kiến thức

Câu hỏi ẩn một **hiểu lầm cần đính chính**: **"pin alg" KHÔNG có nghĩa là bỏ qua/xoá header `alg`.**
Pin nghĩa là **chỉ chấp nhận `alg` nằm trong danh sách cho phép (allowlist)** mà server tự định — chứ
verifier **vẫn đọc** header để biết *dùng thuật toán & khoá nào* mà kiểm chữ ký ([câu 9.5](#q9-5)).

> "Pin" = "đừng để **token tự quyết** thuật toán" — không phải "đừng đọc header".

#### Vai trò thật của `alg` (và `kid`) trong header

1. **Cho verifier biết thuật toán để kiểm chữ ký.** Không có `alg`, thư viện không biết verify bằng HMAC
   hay RSA/ECDSA → không thể parse/verify token theo chuẩn.
2. **Crypto agility (linh hoạt thuật toán).** Hệ thống có thể **hỗ trợ nhiều thuật toán song song** và
   **migrate dần** (vd chuyển RS256 → ES256) mà **không phá token cũ**: token cũ mang `alg` cũ, token mới
   mang `alg` mới — verifier phân biệt qua header và verify đúng loại.
3. **Chọn khoá qua `kid` (key id).** Khi **xoay khoá (rotation)** hoặc dùng **JWKS** có nhiều public key,
   header (`alg` + `kid`) cho biết **dùng khoá nào** để verify.
4. **Interoperability (chuẩn JOSE/JWS).** Token dùng chung giữa nhiều hệ/bên thứ ba cần `alg` để mọi
   thư viện hiểu và verify được — đây là một phần bắt buộc của chuẩn.

#### Lỗ hổng nằm ở đâu?

Lỗ hổng **KHÔNG do sự tồn tại của header**, mà do **tin `alg` một cách mù quáng** — verifier chấp nhận
*bất kỳ* `alg` nào token khai, kể cả:
- **`alg: none`** → bỏ hoàn toàn chữ ký.
- **Đổi `RS256` → `HS256`** → algorithm confusion ([câu 9.8](#q9-8)).

Header vốn **trung tính** (chỉ là metadata); cái sai là verifier **không giới hạn** các giá trị chấp nhận.

**Đúng cách:** đọc `alg` **nhưng ép nằm trong allowlist cố định** (vd `algorithms: ['RS256']`) + chọn
khoá theo `kid`. Đó chính là ý "pin".

> **Ẩn dụ:** `alg` như nhãn *"mở bằng chìa số 3"* dán trên ổ khoá. Người gác **đọc nhãn để lấy đúng
> chìa**, nhưng **chỉ chấp nhận các loại chìa hợp lệ** — tuyệt đối không nghe theo nhãn *"không cần chìa"*
> (`alg: none`). Đọc nhãn ≠ tin nhãn vô điều kiện.

#### Nếu hệ thống chỉ dùng đúng một thuật toán?

Ví dụ **toàn bộ access token đều RS256** — lúc này `alg` trông **có vẻ hơi dư thừa** (verifier đằng nào
cũng chỉ chấp nhận RS256). Nhưng nó **vẫn không thừa**, vì:

1. **Là một phần bắt buộc của JWS Protected Header.** Chuẩn (RFC 7515) **quy định `alg` là trường bắt
   buộc** — bỏ đi thì token **không còn hợp lệ** theo JWS, thư viện chuẩn sẽ từ chối.
2. **Cho biết token *tuyên bố* được ký bằng RS256** — làm token **tự mô tả (self-describing)**, không phụ
   thuộc ngữ cảnh bên ngoài để hiểu cách nó được ký.
3. **Cho phép verifier *từ chối sớm*** (fail-fast): nếu header ghi thuật toán **khác** allowlist (vd
   `none`, `HS256`), verifier **loại ngay** trước cả bước tính chữ ký → vừa an toàn vừa tiết kiệm.
4. **Giữ khả năng tương tác (interoperability)** với **thư viện & hệ thống chuẩn JWT/JWS** — mọi bên đều
   trông đợi `alg`; thiếu nó là **lệch chuẩn**, khó dùng chung.

> Nói cách khác: **"chỉ dùng 1 thuật toán *hôm nay*"** không có nghĩa header thừa — nó vẫn giữ tính
> **hợp chuẩn, tự mô tả, fail-fast**, và để ngỏ **crypto agility** cho tương lai (mai này thêm/đổi thuật
> toán không phải phá cấu trúc token).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **"Pin alg" = giới hạn `alg` trong allowlist, KHÔNG phải xoá header.** Verifier vẫn đọc header để **chọn
  đúng thuật toán/khoá** kiểm chữ ký.
- `alg`/`kid` phục vụ: **báo thuật toán để verify**, **crypto agility** (hỗ trợ & migrate nhiều thuật
  toán mà không phá token cũ), **chọn khoá khi rotation/JWKS**, **interoperability (chuẩn JOSE)**.
- **Lỗ hổng do tin `alg` mù quáng** (chấp nhận `none` hoặc đổi RS256→HS256), **không phải do header tồn
  tại**. Header trung tính; verifier phải là bên giới hạn.
- **Đúng:** đọc `alg` **nhưng ép allowlist** cố định + chọn khoá theo `kid` — đó là "pin".
- **Dù chỉ dùng 1 thuật toán** (vd toàn RS256), `alg` **vẫn không thừa**: **bắt buộc theo JWS**, giúp token
  **tự mô tả**, cho verifier **từ chối sớm** nếu header ghi thuật toán khác, và **tương thích chuẩn JWT/JWS**.

[↑ Mục lục](#mục-lục)

<a id="q9-10"></a>
### 9.10. JWE là gì? Khác JWS/JWT thế nào?

#### 📖 Giới thiệu kiến thức

**JWE (JSON Web Encryption)** (RFC 7516) là chuẩn để **MÃ HOÁ** nội dung một token → đảm bảo **bí mật
(confidentiality)**: bên thứ ba chặn được token cũng **không đọc được** payload. Đây là câu trả lời cho
"muốn giấu dữ liệu trong token thì làm sao" — vì **JWT thường (JWS) chỉ *ký*, không *mã hoá*** (payload
chỉ Base64, ai cũng đọc — xem [câu 9.5](#q9-5), [câu 9.6](#q9-6)).

> Bộ chuẩn **JOSE** có 2 nhánh: **JWS** (JSON Web Signature — *ký*, đảm bảo **toàn vẹn**) và **JWE**
> (*mã hoá*, đảm bảo **bí mật**). Khi nói "JWT" hầu như luôn ngầm là **JWS**.

#### Cấu trúc: JWE có **5 phần** (JWS chỉ có 3)

Ngăn bởi dấu chấm: `Header.EncryptedKey.IV.Ciphertext.Tag`

1. **JOSE Header** — `alg` (thuật toán mã hoá **khoá**) + `enc` (thuật toán mã hoá **nội dung**). Vẫn
   plaintext (Base64).
2. **Encrypted Key** — **Content Encryption Key (CEK)** đã được mã hoá.
3. **Initialization Vector (IV)** — vector khởi tạo cho mã hoá đối xứng.
4. **Ciphertext** — **payload đã mã hoá** (không đọc được nếu thiếu khoá).
5. **Authentication Tag** — tag AEAD, đảm bảo **toàn vẹn** cùng lúc với mã hoá.

**Cơ chế (hybrid, giống TLS):** dùng `alg` (vd **RSA-OAEP**, **ECDH-ES**, hoặc `dir`) để **bọc/trao CEK**;
rồi **CEK** (đối xứng, vd `enc = A256GCM`) **mã hoá payload**. → vừa **bí mật** vừa **toàn vẹn** (AEAD).

#### JWS vs JWE

| | **JWS** (JWT thường) | **JWE** |
|--|--|--|
| Mục đích | **Ký** → toàn vẹn + xác thực nguồn | **Mã hoá** → bí mật |
| Payload | **Base64, đọc được** | **Ciphertext, KHÔNG đọc được** |
| Số phần | 3 | **5** |
| Chống | Sửa đổi (tamper) | Đọc trộm + sửa đổi (AEAD) |
| Phổ biến | **Rất phổ biến** | Ít hơn (phức tạp) |

#### Khi nào dùng? Nested JWT

- Dùng **JWE** khi token **chứa dữ liệu nhạy cảm** không muốn client / bên trung gian đọc (PII, claim nội bộ).
- **Nested JWT**: **ký trước (JWS) rồi mã hoá (JWE)** → vừa **xác thực nguồn** vừa **giấu nội dung**.
- Thực tế **đa số hệ thống dùng JWS** + **HTTPS** + **không để secret trong payload** (đơn giản, đủ dùng);
  chỉ dùng JWE khi thật sự cần **giấu payload đầu-cuối** (liên hệ [câu 12](#q12)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **JWE = JSON Web Encryption**: **mã hoá** payload token → **bí mật** (chặn được cũng không đọc). Khác
  **JWS/JWT thường** vốn **chỉ ký** (payload Base64 đọc được) → chỉ đảm bảo **toàn vẹn**.
- **JOSE** có 2 nhánh: **JWS** (ký) & **JWE** (mã hoá); "JWT" thường ngầm là JWS.
- **JWE có 5 phần** (`Header.EncryptedKey.IV.Ciphertext.Tag`), cơ chế **hybrid**: `alg` bọc **CEK**, `enc`
  (vd A256GCM) mã hoá payload → **bí mật + toàn vẹn (AEAD)**.
- **Dùng khi** cần giấu dữ liệu nhạy cảm trong token; có thể **nested** (JWS rồi JWE). Đa số vẫn chỉ dùng
  **JWS + HTTPS + không để secret trong payload**.

[↑ Mục lục](#mục-lục)

<a id="q9-11"></a>
### 9.11. Access token vs Refresh token khác nhau thế nào? Vì sao refresh dài hơn? Cách revoke token?

#### 📖 Giới thiệu kiến thức

Trong mô hình token-based (JWT — xem [câu 9.5](#q9-5)), server thường cấp **hai** token có vai trò khác
nhau để cân bằng **bảo mật** và **trải nghiệm (UX)**.

#### Access token vs Refresh token

| | **Access token** | **Refresh token** |
|--|--|--|
| Mục đích | "Vé vào cửa" — gọi API/tài nguyên | "Vé gia hạn" — xin access token mới |
| Thời hạn | **Ngắn** (5–15 phút) | **Dài** (ngày/tuần/tháng) |
| Gửi khi nào | **Mọi request** tới Resource Server (`Bearer`) | **Chỉ** khi gọi endpoint `/refresh` |
| Bề mặt lộ | **Cao** (đi khắp nơi) | **Thấp** (1 endpoint, tần suất thấp) |
| Lưu ở đâu | In-memory (client) | Cookie `HttpOnly`+`Secure` ([câu 10.1](#q10-1)) |
| Thu hồi | Khó (stateless) → để ngắn hạn | Dễ hơn (thường lưu server → revoke được) |

#### Vì sao refresh token lại dài hạn hơn?

Đây là một **đánh đổi bảo mật ↔ UX** có chủ đích:

- **Access token phải ngắn** vì nó **đi kèm mọi request** → bề mặt tấn công lớn, dễ lộ (log, proxy, XSS
  nếu lưu sai). Ngắn hạn → nếu lộ, kẻ tấn công **chỉ dùng được vài phút** rồi token tự chết.
- Nhưng nếu access token ngắn mà **không có refresh** → người dùng phải **đăng nhập lại liên tục** (UX
  tệ). **Refresh token dài hạn** cho phép **giữ phiên lâu** mà không bắt nhập lại mật khẩu: khi access
  token hết hạn, client âm thầm gọi `/refresh` để lấy access token mới.
- Refresh token **an toàn để giữ lâu hơn** vì **ít lộ**: chỉ truyền tới **một endpoint** `/refresh`, tần
  suất thấp, lưu ở nơi **JS không đọc được** (cookie `HttpOnly`). Access token thì ngược lại.

→ Tóm lại: **tách vai trò** — access token *dùng thường xuyên nên chấp nhận ngắn*; refresh token *giữ
phiên nên bảo vệ kỹ & cho sống lâu, nhưng phải revoke được*.

```text
Login            → access (15m) + refresh (7d)
Access hết hạn   → POST /refresh (kèm refresh token) → access mới (+ refresh mới nếu rotation)
Refresh hết hạn  → bắt buộc đăng nhập lại
```

#### Cách revoke (thu hồi) token

Vấn đề gốc: **JWT stateless** — server không lưu nên **không thể "xoá" một access token đã phát**; nó
hợp lệ tới khi `exp`. Các chiến lược thu hồi:

1. **Access token ngắn hạn** — cách đơn giản nhất: token tự hết hạn nhanh → **giảm nhu cầu** revoke tức thì.
2. **Revoke refresh token (phổ biến nhất)** — lưu refresh token (hoặc `jti` của nó) ở **DB/Redis**. Khi
   **logout / đổi mật khẩu / nghi bị lộ** → **xoá hoặc đánh dấu thu hồi** → không `/refresh` được nữa.
   Access token còn sống tới `exp` nhưng vì ngắn nên rủi ro thấp.
3. **Blacklist / denylist access token theo `jti`** — lưu `jti` các token bị thu hồi vào **Redis với TTL =
   thời gian còn lại tới `exp`**; mỗi request kiểm blacklist. Thu hồi *ngay* được nhưng **mất tính
   stateless** (phải tra store mỗi request).
4. **Token version** — lưu cột `token_version` ở user; nhét vào claim. Khi **logout-all / đổi mật khẩu** →
   **tăng version** → mọi token mang version cũ bị từ chối. Rẻ, thu hồi hàng loạt tốt.
5. **Refresh token rotation + reuse detection** — mỗi lần `/refresh` cấp **refresh token mới** và **vô
   hiệu cái cũ**. Nếu một refresh token cũ **bị dùng lại** → dấu hiệu bị đánh cắp → **thu hồi cả
   "family"** (buộc login lại). Chuẩn bảo mật hiện đại.
6. **Đổi secret ký** — cực đoan: **vô hiệu toàn bộ** token đang lưu hành cùng lúc.

> Điểm mấu chốt hay hỏi: revoke JWT **buộc phải thêm một chút state** (Redis/DB) — hoàn toàn stateless thì
> không revoke tức thì được. Thực tế: **access ngắn hạn + revoke ở tầng refresh token** là kết hợp cân bằng nhất.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Access token**: ngắn hạn (5–15p), gửi kèm **mọi request** (`Bearer`) để truy cập tài nguyên → bề mặt
  lộ cao nên phải ngắn. **Refresh token**: dài hạn (ngày/tuần), **chỉ** gửi tới `/refresh` để **lấy access
  token mới**, lưu **cookie `HttpOnly`**.
- **Vì sao refresh dài hơn:** đánh đổi **bảo mật ↔ UX**. Access ngắn để nếu lộ chỉ hại vài phút; refresh
  dài để **giữ phiên** không phải login lại. Refresh **an toàn giữ lâu** vì ít lộ (1 endpoint, tần suất
  thấp, `HttpOnly`).
- **Revoke:** JWT stateless không xoá được token đã phát → cần thêm chút state. Cách phổ biến: **access
  ngắn hạn**, **revoke refresh token** (lưu ở DB/Redis, xoá khi logout), **blacklist theo `jti`** (Redis
  TTL tới `exp`), **token version** (đổi mật khẩu → tăng version), **refresh token rotation + reuse
  detection**. Cực đoan: **đổi secret ký**.
- Câu chốt: "Access ngắn + revoke ở tầng refresh token" là kết hợp cân bằng bảo mật/UX nhất.

[↑ Mục lục](#mục-lục)

<a id="q9-12"></a>
### 9.12. Refresh token được lưu ở server bằng cách nào?

#### 📖 Giới thiệu kiến thức

Access token thì **stateless** (server không lưu, chỉ verify chữ ký). Nhưng **refresh token dài hạn &
cần thu hồi được** ([câu 9.11](#q9-11)) → hầu hết hệ thống **lưu refresh token ở server** để **kiểm tra
còn hiệu lực & revoke**. Đây chính là "chút state" mà token-based phải chấp nhận ở tầng refresh.

#### Lưu ở đâu?

- **RDBMS (bảng `refresh_tokens`)** — khi cần **audit, truy vấn theo user/thiết bị**, quan hệ rõ ràng.
- **Redis (key + TTL)** — khi cần **tra nhanh & tự hết hạn** (`EX = thời hạn refresh`); phổ biến cho hệ
  quy mô lớn.

#### Quy tắc vàng: **KHÔNG lưu plaintext — lưu HASH**

Giống lưu mật khẩu: chỉ lưu **hash của refresh token** (thường **SHA-256** là đủ vì refresh token là
chuỗi **ngẫu nhiên entropy cao** — không cần bcrypt/argon2 chậm). Khi client gửi refresh token lên,
server **hash rồi tra**. → Nếu **DB bị lộ**, kẻ tấn công **không dùng lại được** token (không đảo ngược
được hash).

```sql
CREATE TABLE refresh_tokens (
  id           BIGINT PRIMARY KEY,
  user_id      BIGINT NOT NULL,
  token_hash   CHAR(64) NOT NULL,     -- SHA-256 hex — KHÔNG lưu token thô
  expires_at   TIMESTAMP NOT NULL,
  revoked_at   TIMESTAMP NULL,        -- đánh dấu thu hồi (logout / nghi lộ)
  replaced_by  BIGINT NULL,           -- rotation: trỏ tới token kế tiếp (reuse detection)
  user_agent   VARCHAR(255),          -- gắn thiết bị/phiên
  ip           VARCHAR(45),
  created_at   TIMESTAMP
);
```

```text
# Hoặc Redis: key theo hash, tự hết hạn theo TTL
SET refresh:<sha256(token)>  {"user_id":123,"jti":"...","device":"..."}  EX 604800
```

#### Nhờ lưu server → revoke được

- **Logout** → xoá / set `revoked_at` cho record → refresh sau bị từ chối.
- **Đổi mật khẩu / logout-all** → xoá **mọi** refresh token của user.
- **Rotation + reuse detection:** mỗi lần `/refresh` cấp token mới & vô hiệu token cũ (`replaced_by`);
  nếu **token cũ bị dùng lại** → nghi bị đánh cắp → **revoke cả family** (xem [câu 9.11](#q9-11)).

> **Refresh dạng JWT thì sao?** Dù JWT tự chứa, muốn revoke vẫn phải lưu **`jti` vào allowlist/denylist**
> ở server. → **Vẫn cần một store.** (Token client giữ ở cookie `HttpOnly` — xem [câu 10.1](#q10-1);
> server chỉ giữ **bản hash / jti** để đối chiếu.)

#### 🎯 Tóm tắt trả lời phỏng vấn

- Refresh token **dài hạn & cần revoke** → **lưu ở server** (DB hoặc **Redis + TTL**); access token thì
  stateless không lưu.
- **KHÔNG lưu plaintext — lưu HASH** (SHA-256 đủ vì token entropy cao): DB lộ cũng **không dùng lại**
  được. Client gửi token → server **hash rồi tra**.
- Lưu kèm: **user_id, expires_at, revoked_at, thiết bị/ip**, và **`replaced_by`** cho rotation.
- Nhờ có store → **revoke được** (logout, đổi mật khẩu, rotation + reuse detection). **Refresh dạng JWT
  vẫn phải lưu `jti`** để thu hồi.

[↑ Mục lục](#mục-lục)

<a id="q9-13"></a>
### 9.13. Refresh token rotation + reuse detection — mỗi lần `/refresh` làm gì?

#### 📖 Giới thiệu kiến thức

Hai khái niệm đi kèm nhau (nối tiếp [câu 9.11](#q9-11), [câu 9.12](#q9-12)):

- **Rotation (xoay vòng):** mỗi lần dùng refresh token gọi `/refresh`, server **cấp refresh token MỚI**
  và **vô hiệu token cũ** → refresh token là **dùng một lần (one-time use)**.
- **Reuse detection (phát hiện tái sử dụng):** nếu một refresh token **đã bị xoay** (đã dùng/vô hiệu) mà
  **lại được dùng lần nữa** → dấu hiệu **bị đánh cắp** → **thu hồi cả "family"** → buộc login lại.

**Vì sao hiệu quả:** refresh token hợp lệ **chỉ nên xuất hiện đúng một lần**. Nếu kẻ trộm dùng token cũ,
**cả nạn nhân lẫn kẻ trộm** rồi sẽ cùng gọi `/refresh` với token đã xoay → **phát sinh reuse** → hệ thống
biết có rò rỉ và cắt toàn bộ phiên.

#### Token family

Tất cả refresh token sinh ra từ **một lần đăng nhập** chia sẻ một **`family_id`** (hoặc nối nhau qua
`replaced_by`). Reuse ở **bất kỳ** token nào trong family → **revoke cả family** (không chỉ 1 token).

#### Flow mỗi lần `POST /refresh`

```text
1. Client gửi refresh token RT_old (từ cookie HttpOnly)
2. Server hash(RT_old) → tra store (xem schema ở câu 9.12)
3a. Không thấy / hết hạn                → 401 (từ chối)
3b. Thấy nhưng ĐÃ revoked/used (đã xoay) → ⚠ REUSE DETECTED
        → revoke CẢ family (mọi RT cùng family_id) → 401, buộc login lại
3c. Hợp lệ & chưa dùng:
      - Sinh RT_new (cùng family_id) → lưu hash(RT_new)
      - Đánh dấu RT_old = revoked, replaced_by = RT_new.id   (vô hiệu token cũ)
      - Cấp access token mới + RT_new
      - Set-Cookie RT_new (HttpOnly) → trả về client
```

```js
async function refresh(rtRaw) {
  const rec = await db.findRefreshByHash(sha256(rtRaw));
  if (!rec || rec.expiresAt < now()) throw Unauthorized;

  if (rec.revokedAt) {                    // token đã xoay mà vẫn bị dùng → BỊ ĐÁNH CẮP
    await db.revokeFamily(rec.familyId);  // cắt toàn bộ phiên
    throw Unauthorized;
  }
  const rtNew = randomToken();            // xoay: cấp token mới
  const recNew = await db.insertRefresh({ hash: sha256(rtNew), familyId: rec.familyId,
                                          userId: rec.userId, expiresAt: in7days() });
  await db.markRevoked(rec.id, { replacedBy: recNew.id });   // vô hiệu token cũ
  return { accessToken: signAccess(rec.userId), refreshToken: rtNew };
}
```

#### Lưu ý thực tế

- **Race/retry hợp lệ:** mạng chập chờn khiến client **retry `/refresh` với RT_old** sau khi đã xoay →
  có thể **false-positive reuse**. Xử lý: một **grace window ngắn** cho token vừa xoay, hoặc chấp nhận
  đánh đổi **an toàn hơn** (cứ revoke family + bắt login lại).
- Refresh token nên **`HttpOnly`+`Secure`+`SameSite`** (xem [câu 10.1](#q10-1)); server chỉ giữ **hash**
  (xem [câu 9.12](#q9-12)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Rotation:** mỗi `/refresh` **cấp refresh token mới + vô hiệu token cũ** (one-time use). **Reuse
  detection:** token đã xoay mà bị dùng lại → **nghi đánh cắp** → **revoke cả family** → buộc login lại.
- **Flow:** hash RT_old → tra store → nếu **đã revoked = reuse → cắt family**; nếu hợp lệ → **sinh RT_new
  (cùng family), đánh dấu RT_old revoked + `replaced_by`**, cấp access + RT_new.
- **Token family:** mọi RT từ một lần login chung `family_id`; reuse ở bất kỳ token nào → revoke cả family.
- **Lưu ý:** retry mạng có thể gây false-positive → dùng **grace window**; RT giữ ở **cookie HttpOnly**,
  server chỉ lưu **hash**.

[↑ Mục lục](#mục-lục)

<a id="q9-14"></a>
### 9.14. Cách revoke hàng loạt JWT đã phát cho user (đổi mật khẩu, force logout)

#### 📖 Giới thiệu kiến thức

Đây là câu đào sâu tiếp nối phần revoke ở [câu 9.11](#q9-11): không phải thu hồi **một** token, mà **vô
hiệu hoá TẤT CẢ token đang lưu hành của một user (hoặc toàn hệ thống)** trong các tình huống:

- **Đổi mật khẩu / reset mật khẩu** → mọi phiên cũ phải văng ra (đề phòng mật khẩu cũ đã bị lộ).
- **Force logout / "đăng xuất khỏi mọi thiết bị"** (logout-all).
- **Nghi tài khoản bị chiếm** (đổi email, phát hiện đăng nhập lạ), **admin khoá/ban user**, **đổi role/
  quyền** cần áp dụng ngay.

**Gốc rễ của vấn đề — JWT stateless:** access token JWT **tự chứa & tự xác thực bằng chữ ký**, server
**không lưu** nên **không có gì để "xoá"**. Một token đã phát **mặc định hợp lệ tới khi `exp`**. Muốn
thu hồi *trước hạn* thì **bắt buộc thêm một chút state** ở server để đối chiếu mỗi request. Với **revoke
hàng loạt**, ta cần một "công tắc" chung đánh sập nhiều token cùng lúc thay vì liệt kê từng cái. Có 3
nhóm cách chính:

**Cách 1 (khuyến nghị) — `token_version` / `tokenVersion` per-user:**
Lưu một số nguyên `token_version` ở bản ghi user; **nhúng giá trị đó vào payload** khi phát token. Mỗi
request, sau khi verify chữ ký, so `payload.ver` với `token_version` hiện tại trong DB — **khác nhau →
từ chối**.

```js
// Khi phát token
const token = jwt.sign({ sub: user.id, ver: user.tokenVersion }, SECRET, { expiresIn: '15m' });

// Middleware xác thực
function verify(token) {
  const p = jwt.verify(token, SECRET);              // 1) verify chữ ký + exp
  const user = db.getUser(p.sub);                    // 2) đọc version hiện tại
  if (p.ver !== user.tokenVersion) throw Unauthorized; // 3) version cũ → đã bị revoke hàng loạt
  return p;
}

// Đổi mật khẩu / force logout-all → TĂNG version một phát, mọi token cũ chết ngay
async function onPasswordChange(userId) {
  await db.increment('users', userId, 'tokenVersion'); // ver: 4 → 5
}
```

→ **Ưu:** revoke **toàn bộ token của user** chỉ bằng **1 lần ghi** (tăng 1 số); không cần liệt kê token,
không phình store. **Nhược:** phải **tra user mỗi request** (mất phần nào tính stateless) — nhưng thường
đã cần tra user rồi, và có thể **cache `token_version` vào Redis** cho nhanh.

**Cách 2 — `passwordChangedAt` / timestamp mốc thời gian:**
Không cần cột version riêng: lưu **thời điểm đổi mật khẩu** (`passwordChangedAt`) và so với **`iat`**
(issued-at) của token — **token phát TRƯỚC mốc đó → từ chối**. Rất hợp riêng ca đổi mật khẩu.

```js
if (user.passwordChangedAt && payload.iat * 1000 < user.passwordChangedAt.getTime()) {
  throw Unauthorized; // token cũ hơn lần đổi mật khẩu gần nhất → vô hiệu
}
```

**Cách 3 — Denylist/blocklist theo `jti` hoặc `user_id` (Redis):**
- **Theo `jti`** (từng token): thu hồi lẻ, ít hợp "hàng loạt".
- **Theo user** cho revoke hàng loạt: lưu key `revoke:user:<id> = <timestamp>` trên Redis (TTL = tuổi
  thọ tối đa của token). Mỗi request kiểm: nếu `token.iat < revoke:user:<id>` → chặn. Bản chất giống
  cách 2 nhưng đặt ở **Redis**, tự hết hạn theo TTL nên **không phình vô hạn**.

**Cách 4 (hạt nhân nguyên tử) — đổi secret ký:** đổi `SECRET` (hoặc khoá ký) → **mọi token toàn hệ
thống chết ngay**. Chỉ dùng khi **sự cố diện rộng** (nghi lộ khoá), vì nó **đá văng tất cả user**, không
chọn lọc được 1 user.

**Kết hợp với refresh token:** các cách trên vô hiệu **access token**. Song song phải **xoá/đánh dấu
revoked mọi refresh token của user** trong store ([câu 9.12](#q9-12)) để họ **không `/refresh` ra token
mới** được. Vì **access token để ngắn hạn (5–15p)**, cửa sổ rủi ro giữa lúc revoke và lúc access cũ tự
hết hạn là rất nhỏ — nên nhiều hệ thống chỉ cần **revoke refresh + access ngắn hạn** là đủ, không cần
kiểm version mỗi request.

**So sánh nhanh:**

| Cách | Revoke hàng loạt | Chi phí mỗi request | Ghi chú |
|------|------------------|----------------------|---------|
| `token_version` per-user | ✅ 1 lần ghi/user | Tra user (cache được) | Khuyến nghị, linh hoạt |
| `passwordChangedAt` (so `iat`) | ✅ theo user | Tra user | Gọn cho ca đổi mật khẩu |
| Denylist theo user (Redis) | ✅ theo user | 1 lần GET Redis | Tự hết hạn theo TTL |
| Đổi secret ký | ✅ toàn hệ thống | 0 | Đá văng **mọi** user |
| Access token ngắn hạn | ⚠ gián tiếp | 0 | Giảm nhu cầu revoke tức thì |

> **Chốt:** không có JWT nào "xoá" được khi đang stateless; muốn **force logout hàng loạt** phải thêm
> **một mốc/biến chung per-user** (version hoặc timestamp) để đối chiếu, cộng với **xoá refresh token**.
> Cách phổ biến & cân bằng nhất: **`token_version` (hoặc `passwordChangedAt`) + revoke refresh token +
> access token ngắn hạn**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Vấn đề: JWT **stateless** → không "xoá" được token đã phát; muốn **revoke hàng loạt** (đổi mật khẩu,
  force logout-all, ban user) phải **thêm một biến/mốc chung per-user** để đối chiếu mỗi request.
- **Cách khuyến nghị — `token_version`:** lưu số version ở user, **nhúng vào payload**; đổi mật khẩu /
  logout-all → **tăng version 1 phát** → mọi token mang version cũ bị từ chối. Chỉ 1 lần ghi, revoke sạch.
- **Biến thể — `passwordChangedAt`:** so `iat` của token với thời điểm đổi mật khẩu; token cũ hơn → chặn.
  Hoặc **denylist theo user trên Redis** (TTL tự dọn).
- **Cực đoan — đổi secret ký:** vô hiệu **toàn bộ** token cả hệ thống (đá văng mọi user), chỉ cho sự cố lớn.
- **Kết hợp bắt buộc:** đồng thời **xoá/revoke mọi refresh token của user** ([câu 9.12](#q9-12)) để không
  `/refresh` ra token mới; nhờ **access token ngắn hạn** nên cửa sổ rủi ro rất nhỏ.
- Câu chốt: **"`token_version`/`passwordChangedAt` + revoke refresh token + access ngắn hạn"** là combo cân bằng nhất.

[↑ Mục lục](#mục-lục)

<a id="q9-15"></a>
### 9.15. OAuth2 là gì? Flow và các thành phần chính của OAuth2

#### 📖 Giới thiệu kiến thức

**OAuth 2.0** là một **framework uỷ quyền (authorization framework)** cho phép một ứng dụng **truy cập
tài nguyên của người dùng** trên một dịch vụ khác **mà không cần biết mật khẩu** của họ. Ý tưởng cốt lõi:
thay vì đưa mật khẩu, người dùng cấp cho app một **access token** có **phạm vi (scope)** và **thời hạn**
giới hạn.

> Quan trọng: **OAuth2 là *authorization* (uỷ quyền truy cập tài nguyên), KHÔNG phải authentication.**
> Muốn dùng để **xác thực danh tính** ("đăng nhập bằng Google") thì phải dùng **OpenID Connect (OIDC)** —
> một lớp phủ **trên** OAuth2, bổ sung `id_token` (JWT chứa thông tin danh tính). Nhầm OAuth2 = login là
> lỗi hay bị bắt (xem authn vs authz ở [câu 8](#q8)).

#### 4 thành phần (roles) chính của OAuth2

| Role | Vai trò | Ví dụ ("app X xin xem Google Contacts") |
|------|---------|------------------------------------------|
| **Resource Owner** | Người dùng sở hữu dữ liệu, người *cấp quyền* | Bạn (chủ tài khoản Google) |
| **Client** | Ứng dụng muốn truy cập tài nguyên thay bạn | App X |
| **Authorization Server** | Xác thực Resource Owner & **cấp token** | Máy chủ OAuth của Google (accounts.google.com) |
| **Resource Server** | Nơi chứa tài nguyên, chấp nhận access token | Google Contacts API |

> Authorization Server và Resource Server có thể là **cùng** hoặc **khác** hệ thống.

#### Các "vật phẩm" (tokens & credentials) chính

- **Authorization Code** — mã tạm ngắn hạn, dùng **một lần**, đổi lấy token (ở Authorization Code Flow).
- **Access Token** — "chìa khoá" để gọi Resource Server (thường là **JWT** hoặc opaque), **ngắn hạn**,
  gửi qua header `Authorization: Bearer <token>`.
- **Refresh Token** — **dài hạn**, dùng để **lấy access token mới** khi hết hạn (không phải gửi lại mật khẩu).
- **Client ID / Client Secret** — định danh & bí mật của Client, đăng ký trước với Authorization Server.
- **Scope** — phạm vi quyền được xin (vd `read:contacts`, `email`). **Redirect URI** — nơi Authorization
  Server trả code về (phải đăng ký trước, chống hijack).

#### Flow chính: Authorization Code Flow (phổ biến & an toàn nhất)

Dùng cho web app có backend. Điểm mấu chốt: **access token không lộ ra trình duyệt/URL**, chỉ backend mới
đổi được code lấy token (nhờ `client_secret`).

```text
1. User bấm "Login with Google" trên Client
2. Client redirect user → Authorization Server
   (kèm client_id, redirect_uri, scope, response_type=code, state)
3. User đăng nhập & đồng ý cấp quyền (consent) tại Authorization Server
4. Authorization Server redirect về redirect_uri, kèm ?code=AUTH_CODE&state=...
5. Client (backend) gọi Authorization Server: đổi code + client_secret  →  ACCESS TOKEN (+ refresh token)
6. Client dùng access token gọi Resource Server (Authorization: Bearer ...)
7. Resource Server verify token → trả tài nguyên
```

**Sequence diagram — Authorization Code Flow (kèm PKCE + refresh):**

```mermaid
sequenceDiagram
    autonumber
    actor RO as Resource Owner<br/>(User + Browser)
    participant C as Client<br/>(App backend)
    participant AS as Authorization Server<br/>(vd: accounts.google.com)
    participant RS as Resource Server<br/>(vd: Google Contacts API)

    Note over C: Đăng ký trước:<br/>client_id, client_secret, redirect_uri, scope

    RO->>C: Bấm "Login with Google"
    Note over C: (PKCE) tạo code_verifier<br/>→ code_challenge = SHA256(verifier)<br/>tạo state ngẫu nhiên
    C-->>RO: 302 Redirect → Authorization Server<br/>(client_id, redirect_uri, scope,<br/>response_type=code, state, code_challenge)
    RO->>AS: GET /authorize (theo redirect)
    AS-->>RO: Trang login + màn hình consent
    RO->>AS: Đăng nhập & đồng ý cấp quyền (consent)
    AS-->>RO: 302 Redirect → redirect_uri<br/>(?code=AUTH_CODE & state)
    RO->>C: GET redirect_uri?code=AUTH_CODE&state

    Note over C: Kiểm state khớp giá trị đã gửi<br/>(chống CSRF)

    C->>AS: POST /token<br/>(code, client_id, client_secret,<br/>redirect_uri, code_verifier)
    Note over AS: Xác thực client + verify<br/>SHA256(code_verifier)==code_challenge
    AS-->>C: access_token (+ refresh_token, expires_in)

    C->>RS: GET /contacts<br/>Authorization: Bearer access_token
    RS->>RS: Verify token (chữ ký / introspection) + scope
    RS-->>C: 200 Dữ liệu tài nguyên
    C-->>RO: Hiển thị dữ liệu

    Note over C,AS: === Khi access_token hết hạn ===
    C->>AS: POST /token<br/>(grant_type=refresh_token, refresh_token)
    AS-->>C: access_token mới (+ refresh_token mới nếu rotation)
```

**Ý nghĩa các tham số trong request `/authorize` (bước redirect user → Authorization Server):**

| Tham số | Ý nghĩa | Vai trò / vì sao cần |
|---------|---------|----------------------|
| `client_id` | **Định danh công khai** của Client, cấp khi đăng ký app với Authorization Server | Để AS biết **app nào** đang xin quyền, tra ra cấu hình (redirect_uri hợp lệ, scope được phép). Không bí mật, có thể lộ ra URL. |
| `redirect_uri` | **URL** mà Authorization Server sẽ trả `code` về sau khi user consent | Phải **khớp chính xác** giá trị đã đăng ký trước → **chống hijack** (kẻ tấn công không thể đổi hướng code sang server của chúng). |
| `scope` | **Phạm vi quyền** app xin (vd `read:contacts`, `email`, `profile`) | Nguyên tắc **least privilege** — chỉ xin đúng thứ cần. AS hiển thị scope này trên màn hình consent để user biết mình đang cấp gì. |
| `response_type=code` | Khai báo dùng **Authorization Code Flow** (muốn nhận về *code*, không nhận thẳng token) | Phân biệt với các flow khác (vd `token` = Implicit — đã lỗi thời). `code` an toàn hơn vì token chỉ được cấp ở bước backend đổi code. |
| `state` | Chuỗi **ngẫu nhiên, không đoán được** Client tự tạo mỗi lần login | Client lưu lại rồi **so khớp** khi nhận `code` về → **chống CSRF** (kẻ tấn công không ép được nạn nhân dùng code của chúng). |
| `code_challenge` | **Bản băm** `SHA256(code_verifier)` (PKCE), gửi kèm khi khởi tạo | AS lưu lại; ở bước đổi token, Client phải xuất trình `code_verifier` gốc để AS băm lại và **so khớp** → **chống đánh cắp authorization code** (bắt buộc cho SPA/mobile không giữ được secret). |

> `client_secret` **KHÔNG** xuất hiện ở request này (nó đi qua trình duyệt, dễ lộ). Secret chỉ được gửi ở
> bước backend gọi `POST /token` (kênh server-to-server) — đó là lý do access token không bao giờ lộ ra browser.

- **`state`**: giá trị ngẫu nhiên Client gửi đi và kiểm lại khi nhận code → **chống CSRF** trên bước redirect.
- **PKCE** (`code_challenge`/`code_verifier`): mở rộng bắt buộc cho **SPA/mobile** (client không giữ được
  secret) → chống đánh cắp authorization code.

#### Các grant type (flow) khác — chọn theo loại client

- **Authorization Code + PKCE** — **khuyến nghị mặc định** cho web, **SPA và mobile** (thay cho Implicit).
- **Client Credentials** — **máy-với-máy** (không có user): Client tự dùng `client_id/secret` lấy token
  cho chính nó (vd service gọi service, cron job).
- **Refresh Token** — dùng refresh token đổi lấy access token mới.
- **Implicit Flow** — *(cũ, đã khuyến cáo bỏ)* trả token thẳng trên URL fragment → lộ token, thay bằng
  Auth Code + PKCE.
- **Resource Owner Password Credentials (ROPC)** — *(cũ, hạn chế dùng)* Client cầm trực tiếp username/
  password → chỉ dùng cho app "first-party" cực tin cậy; ngược tinh thần OAuth2.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **OAuth2** = **framework uỷ quyền (authorization)** cho phép app truy cập tài nguyên của user trên dịch
  vụ khác **mà không cần mật khẩu**, bằng **access token** có **scope + thời hạn**. Muốn *đăng nhập*
  (authentication) phải dùng **OIDC** phủ trên OAuth2 (thêm `id_token`).
- **4 roles:** **Resource Owner** (user), **Client** (app xin quyền), **Authorization Server** (cấp
  token), **Resource Server** (giữ tài nguyên, nhận token).
- **Token/credential chính:** **authorization code** (đổi lấy token), **access token** (`Bearer`, ngắn
  hạn), **refresh token** (dài hạn, lấy access token mới), **client_id/secret**, **scope**, **redirect_uri**.
- **Authorization Code Flow (an toàn nhất):** redirect user → login & consent ở Authorization Server →
  nhận **code** → backend đổi code + secret lấy **access token** → gọi Resource Server bằng `Bearer`.
  Dùng **`state`** chống CSRF, **PKCE** cho SPA/mobile.
- **Grant types:** Auth Code + **PKCE** (mặc định web/SPA/mobile), **Client Credentials** (máy-máy),
  **Refresh Token**; tránh **Implicit** & **ROPC** (đã lỗi thời).

[↑ Mục lục](#mục-lục)

<a id="q10"></a>
## 10. Broken Authentication / Session là gì?

### 📖 Giới thiệu kiến thức

**Broken Authentication** là nhóm lỗ hổng ở **cơ chế xác thực (authentication)** và **quản lý phiên
(session management)** yếu, khiến kẻ tấn công **chiếm được tài khoản** — qua việc đoán/đánh cắp mật
khẩu, hoặc **chiếm session token** để mạo danh người dùng mà không cần mật khẩu. Đây là một trong những
lỗ hổng trong danh mục ở [câu 2](#q2).

> Phân biệt nhanh: **Authentication** = "bạn là ai?" (đăng nhập). **Authorization** = "bạn được phép
> làm gì?" (phân quyền — lỗi ở đây là *Broken Access Control*). Câu này nói về **authentication + session**.

#### Các điểm yếu thường gặp

- **Mật khẩu yếu / không có chính sách** mật khẩu; cho phép mật khẩu đã lộ.
- **Không chống brute-force / credential stuffing** (thử hàng loạt mật khẩu, hoặc dùng cặp user-pass
  rò rỉ từ site khác).
- **Lưu mật khẩu sai cách**: plaintext, hoặc hash yếu (MD5/SHA1 không salt).
- **Session token đoán được** hoặc bị lộ (đặt trong URL, log...).
- **Session không hết hạn / không timeout**; **không huỷ session khi logout**.
- **Session Fixation**: kẻ tấn công ép nạn nhân dùng một session id biết trước, sau khi nạn nhân đăng
  nhập thì chiếm phiên đó.
- **Thiếu MFA**; truyền credential **không qua TLS**.

#### Các kiểu tấn công tiêu biểu

- **Brute force / Credential stuffing** — dò mật khẩu hoặc tái sử dụng cặp đăng nhập đã rò rỉ.
- **Session hijacking** — đánh cắp session token (qua XSS, sniffing, log...) rồi mạo danh.
- **Session fixation** — như trên: cố định session id trước khi nạn nhân đăng nhập.

#### Cách phòng chống

**Phía xác thực (authentication):**

- **Băm mật khẩu mạnh có salt**: **Argon2** hoặc **bcrypt** (đừng MD5/SHA1, đừng "mã hoá 2 chiều").

  ```php
  $hash = password_hash($password, PASSWORD_ARGON2ID);  // lưu $hash
  if (password_verify($input, $hash)) { /* đúng mật khẩu */ }
  ```

- **Rate limit + khoá tài khoản tạm thời** sau nhiều lần sai; thêm **CAPTCHA**.
- **MFA / 2FA** cho tài khoản quan trọng.
- **Chính sách mật khẩu** hợp lý + đối chiếu danh sách mật khẩu đã lộ (have i been pwned...).

**Phía quản lý phiên (session):**

- **Session token ngẫu nhiên, đủ dài** (dùng cơ chế của framework, không tự chế).
- **Đổi (regenerate) session id ngay sau khi đăng nhập** → chống **session fixation**.

  ```php
  // sau khi xác thực thành công
  session_regenerate_id(true);   // cấp session id mới, huỷ id cũ
  ```

- Cookie phiên đặt **`HttpOnly` + `Secure` + `SameSite`**; **không để session id trong URL**.
- **Timeout** theo thời gian idle + thời gian tuyệt đối; **huỷ session phía server khi logout**.
- **HTTPS/TLS** bắt buộc cho toàn bộ luồng đăng nhập.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Broken Authentication** = cơ chế **đăng nhập + quản lý phiên** yếu → kẻ tấn công **chiếm tài khoản**:
  đoán/đánh cắp mật khẩu, hoặc **chiếm session token** để mạo danh.
- **Điểm yếu hay gặp:** mật khẩu yếu/hash sai, không chống brute-force/credential stuffing, session token
  đoán được hoặc đặt trong URL, session không hết hạn/không huỷ khi logout, **session fixation**, thiếu MFA.
- **Chống — xác thực:** băm **Argon2/bcrypt** (có salt), **rate limit + lockout + CAPTCHA**, **MFA**,
  chặn mật khẩu đã lộ.
- **Chống — session:** token **ngẫu nhiên**, **`session_regenerate_id` sau đăng nhập** (chống fixation),
  cookie **`HttpOnly`+`Secure`+`SameSite`**, **timeout**, **huỷ session khi logout**, **HTTPS** bắt buộc.
- Phân biệt: đây là **authentication/session**; "được phép làm gì" là **authorization** (Broken Access
  Control — lỗ hổng khác).

[↑ Mục lục](#mục-lục)

<a id="q10-1"></a>
### 10.1. Lưu access/refresh token ở đâu trên client & truyền access token thế nào?

#### 📖 Giới thiệu kiến thức

Bối cảnh: backend trả về **access token** (ngắn hạn, vd 15 phút) + **refresh token** (dài hạn, vd 7–30
ngày). Client cần **lưu** để gửi kèm request và **refresh** khi access token hết hạn. Câu hỏi gồm 2 phần:
lưu ở đâu cho an toàn, và **truyền access token lên backend ở đâu**.

**Các nơi lưu được & đánh đổi (tradeoff XSS vs CSRF):**

| Nơi lưu | Ưu | Nhược |
|---------|-----|-------|
| **localStorage / sessionStorage** | Đơn giản, JS đọc & gắn header dễ | **JS đọc được → XSS đánh cắp token** (rủi ro cao); không tự gửi kèm |
| **Cookie thường** (JS đọc được) | Tự gửi kèm request | **Tệ nhất**: vừa bị **XSS** đọc, vừa bị **CSRF** (tự gửi kèm) |
| **Cookie `HttpOnly`+`Secure`+`SameSite`** | **JS KHÔNG đọc được → chống XSS đánh cắp**; tự gửi kèm | Bị **CSRF** (vì tự gửi) → cần chống CSRF; cần cấu hình CORS credentials |
| **In-memory** (biến/state JS) | An toàn nhất với access token (không persist, mất khi reload) | Mất khi reload tab → cần refresh token để lấy lại |

**Khuyến nghị thực tế (pattern phổ biến & an toàn):**
- **Refresh token → cookie `HttpOnly` + `Secure` + `SameSite`.** Vì refresh token **dài hạn & nhạy cảm
  nhất**, phải để **JS không đọc được** (chống XSS đánh cắp); cookie chỉ gửi tới endpoint `/refresh`.
- **Access token → lưu in-memory** (biến trong app/state). Ngắn hạn nên mất khi reload không sao — dùng
  refresh token (trong cookie HttpOnly) để **lấy access token mới** khi mở lại app.
- **Tránh để token (nhất là refresh) trong `localStorage`** vì **XSS đọc được** (xem [câu 3](#q3)). Nếu
  buộc dùng cookie cho cả access token → `HttpOnly`+`SameSite` và **chống CSRF** (xem [câu 5](#q5)).

**Truyền access token lên backend khi xác thực — ở đâu?**
- **Chuẩn: HTTP header `Authorization: Bearer <access_token>`** (RFC 6750) cho token-based / stateless API.
  Backend đọc header rồi verify token.

  ```http
  GET /api/me HTTP/1.1
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  ```

- **KHÔNG đặt token trong URL/query string** — bị ghi vào **log server, lịch sử trình duyệt, header
  `Referer`** → rò rỉ.
- Nếu dùng **cookie `HttpOnly`** (mô hình cookie-based): token **tự gửi kèm** mỗi request, không cần header
  — nhưng **phải chống CSRF**.

**Hai mô hình tổng kết:**
- **Bearer token (header):** access token gửi qua `Authorization: Bearer`. Hợp **API stateless, mobile,
  cross-domain**; **không bị CSRF** (không tự gửi kèm) nhưng rủi ro **XSS** nếu lưu ở storage.
- **Cookie HttpOnly:** token tự gửi kèm; **chống XSS đọc tốt** nhưng **phải chống CSRF** (SameSite + CSRF
  token).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Access token** (ngắn hạn) → tốt nhất **lưu in-memory** (mất khi reload thì dùng refresh token lấy lại).
  **Refresh token** (dài hạn, nhạy cảm) → **cookie `HttpOnly`+`Secure`+`SameSite`** để JS không đọc được
  (chống XSS đánh cắp).
- **Tránh `localStorage`** cho token (nhất là refresh) vì **XSS đọc được**; **cookie thường** (JS đọc được)
  là tệ nhất (XSS + CSRF).
- **Truyền access token:** chuẩn là header **`Authorization: Bearer <token>`** (RFC 6750); **không** đặt
  trong **URL/query** (bị log/referer leak).
- **Tradeoff:** Bearer/header **không bị CSRF** nhưng rủi ro **XSS-storage**; cookie **HttpOnly** chống XSS
  đọc tốt nhưng **phải chống CSRF**. Liên hệ **XSS** ([câu 3](#q3)) & **CSRF** ([câu 5](#q5)).

[↑ Mục lục](#mục-lục)

<a id="q11"></a>
## 11. SSL là gì? Cách sử dụng?

### 📖 Giới thiệu kiến thức

**SSL (Secure Sockets Layer)** là giao thức **mã hoá kết nối** giữa client và server để dữ liệu truyền
đi **không bị đọc trộm hay sửa đổi** trên đường truyền. SSL đã **lỗi thời** và được thay bằng **TLS
(Transport Layer Security)** — bản kế nhiệm an toàn hơn. Ngày nay khi nói "SSL" thực chất người ta đang
dùng **TLS**, nhưng tên "SSL certificate" vẫn quen dùng.

> **HTTPS = HTTP + TLS.** Chính TLS là lớp làm nên chữ "S" (Secure) trong HTTPS.

#### SSL/TLS bảo vệ điều gì? (3 đảm bảo)

- **Confidentiality (bí mật)** — dữ liệu được **mã hoá**, bên thứ ba sniff cũng không đọc được.
- **Integrity (toàn vẹn)** — phát hiện nếu dữ liệu bị **sửa** trên đường truyền (MAC).
- **Authentication (xác thực server)** — nhờ **chứng chỉ số (certificate)**, client biết chắc mình đang
  nói chuyện với đúng server (chống **man-in-the-middle**), không phải kẻ mạo danh.

#### Cơ chế: TLS handshake (mã hoá lai — hybrid)

Điểm cốt lõi hay hỏi: TLS dùng **bất đối xứng để trao đổi khoá**, rồi **đối xứng để mã hoá dữ liệu** (vì
đối xứng nhanh hơn nhiều).

```text
1. Client Hello  → gửi TLS version, cipher suites hỗ trợ, random
2. Server Hello  → chọn cipher, gửi CHỨNG CHỈ (chứa public key, do CA ký)
3. Client verify chứng chỉ với CA tin cậy (còn hạn? đúng domain? chuỗi tin cậy?)
4. Hai bên trao đổi & thống nhất KHOÁ PHIÊN (session key) — dùng bất đối xứng/ECDHE
5. Từ đây: mã hoá dữ liệu bằng khoá phiên (ĐỐI XỨNG, nhanh)
```

- **Chứng chỉ (certificate)** do **CA (Certificate Authority)** ký, gắn **public key** với **domain**.
  Trình duyệt tin CA → tin server. Có các mức: **DV** (domain), **OV** (tổ chức), **EV** (mở rộng).
- **Asymmetric** (RSA/ECDSA) chỉ dùng lúc **handshake** để trao khoá; **symmetric** (AES) mã hoá **toàn
  bộ dữ liệu** sau đó.

#### Cách sử dụng (triển khai HTTPS thực tế)

1. **Có domain** và trỏ DNS về server.
2. **Lấy chứng chỉ TLS**: xin từ CA. Miễn phí phổ biến là **Let's Encrypt** (dùng **Certbot** tự cấp &
   **tự gia hạn**); hoặc mua chứng chỉ OV/EV.
3. **Cài chứng chỉ lên web server / load balancer** (Nginx, Apache, hoặc **TLS termination** ở LB/CDN
   như Cloudflare, AWS ALB/ACM):

   ```nginx
   server {
       listen 443 ssl;
       server_name example.com;
       ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
       ssl_protocols       TLSv1.2 TLSv1.3;   # tắt SSLv3/TLS1.0/1.1 (cũ, yếu)
   }
   ```

4. **Ép HTTPS**: redirect `80 → 443` và bật **HSTS** (`Strict-Transport-Security`) để trình duyệt **luôn**
   dùng HTTPS.
5. **Tự động gia hạn** chứng chỉ (Let's Encrypt hết hạn 90 ngày) — Certbot cron/systemd timer.

> **TLS termination:** thường TLS được "kết thúc" ở **load balancer/CDN/reverse proxy**; từ đó vào mạng
> nội bộ có thể là HTTP (tin cậy) hoặc TLS lại (mTLS) tuỳ mô hình bảo mật.

### 🎯 Tóm tắt trả lời phỏng vấn

- **SSL/TLS** = giao thức **mã hoá kết nối** client–server; **SSL đã lỗi thời, nay dùng TLS**. **HTTPS =
  HTTP + TLS**.
- Đảm bảo **3 điều:** **bí mật** (mã hoá), **toàn vẹn** (chống sửa), **xác thực server** (qua **chứng chỉ**
  do **CA** ký → chống man-in-the-middle).
- **Handshake (hybrid):** dùng **bất đối xứng** (RSA/ECDHE) để **trao khoá phiên**, rồi **đối xứng** (AES)
  để mã hoá dữ liệu (nhanh hơn). Server gửi **certificate** chứa public key, client verify với CA.
- **Cách dùng:** có domain → **xin chứng chỉ** (Let's Encrypt/Certbot miễn phí, tự gia hạn; hoặc mua) →
  **cài lên Nginx/LB/CDN** → **redirect 80→443 + HSTS** → **tự động gia hạn**. Tắt giao thức cũ (SSLv3/
  TLS1.0/1.1), dùng **TLS 1.2/1.3**.

[↑ Mục lục](#mục-lục)

<a id="q11-1"></a>
### 11.1. Phân biệt SSL và TLS

#### 📖 Giới thiệu kiến thức

Về bản chất, **TLS là phiên bản kế nhiệm của SSL** — cùng một mục đích (mã hoá kết nối), nhưng TLS **mới
hơn và an toàn hơn**. **SSL đã lỗi thời và bị "khai tử"** (mọi phiên bản SSL đều không còn an toàn); thứ
thực sự chạy ngày nay là **TLS**. Sở dĩ vẫn nghe "SSL" khắp nơi (vd "SSL certificate") là do **thói quen
đặt tên** — về kỹ thuật đó là **TLS certificate**.

**Dòng thời gian (lịch sử):**

| Phiên bản | Năm | Trạng thái |
|-----------|-----|------------|
| SSL 1.0 | — | Không phát hành (lỗi bảo mật) |
| SSL 2.0 | 1995 | **Deprecated** (không an toàn) |
| SSL 3.0 | 1996 | **Deprecated** (dính tấn công **POODLE**) |
| **TLS 1.0** | 1999 | Deprecated (2021) |
| **TLS 1.1** | 2006 | Deprecated (2021) |
| **TLS 1.2** | 2008 | ✅ Còn dùng rộng rãi |
| **TLS 1.3** | 2018 | ✅ **Mới nhất**, nhanh & an toàn nhất |

**Khác biệt kỹ thuật chính (SSL → TLS):**
- **Bảo mật mạnh hơn:** TLS dùng thuật toán **MAC** hiện đại (HMAC) thay cho MAC yếu của SSL; loại bỏ các
  cipher yếu (RC4, MD5...).
- **Handshake tối ưu hơn:** **TLS 1.3** rút gọn handshake còn **1 vòng (1-RTT)**, hỗ trợ **0-RTT** →
  kết nối nhanh hơn; bỏ các bước/thuật toán không an toàn, bắt buộc **forward secrecy** (ECDHE).
- **Alert & message** rõ ràng hơn; nhiều cải tiến chống các tấn công đã biết (POODLE, BEAST...).

> Nói ngắn khi phỏng vấn: **"SSL và TLS về vai trò là một (mã hoá on-transit), nhưng SSL là thế hệ cũ đã
> bị bỏ; TLS là bản thay thế an toàn — nên gọi đúng phải là TLS."**

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Cùng mục đích** (mã hoá kết nối client–server), nhưng **TLS là bản kế nhiệm, mới & an toàn hơn SSL**.
  **Mọi phiên bản SSL đều đã lỗi thời/không an toàn** (SSL 3.0 dính **POODLE**).
- Tên **"SSL certificate"** chỉ là **thói quen** — kỹ thuật đó là **TLS**; thứ chạy thật hiện nay là TLS.
- **TLS mạnh hơn:** thuật toán MAC/cipher hiện đại (bỏ RC4/MD5), **handshake tối ưu** (TLS 1.3 chỉ
  **1-RTT**, có 0-RTT, bắt buộc **forward secrecy**).
- Thực tế nên bật **TLS 1.2 & 1.3**, **tắt SSLv3/TLS 1.0/1.1** (xem cấu hình ở [câu 11](#q11)).

[↑ Mục lục](#mục-lục)

<a id="q11-2"></a>
### 11.2. HTTPS và HTTP khác nhau thế nào?

#### 📖 Giới thiệu kiến thức

Khác biệt **cốt lõi** rất gọn: **HTTPS = HTTP + TLS**. Cùng một giao thức truyền tải web, nhưng HTTPS
chạy **bên trong một kết nối TLS đã mã hoá** (xem [câu 11](#q11)), còn HTTP thì **truyền văn bản thô
(plaintext)**.

- **HTTP (HyperText Transfer Protocol)** — cổng **80**, **không mã hoá**. Mọi thứ (URL path, header,
  cookie, mật khẩu, body) đi ở dạng **rõ** → bất kỳ ai trên đường truyền (Wi-Fi công cộng, ISP, proxy)
  đều có thể **đọc trộm (sniff), sửa đổi, hoặc mạo danh (MITM)**.
- **HTTPS (HTTP Secure)** — cổng **443**, HTTP bọc trong **TLS** → có **3 đảm bảo**: **bí mật** (mã hoá),
  **toàn vẹn** (chống sửa), **xác thực server** (qua **chứng chỉ** do CA ký).

#### Bảng so sánh

| Tiêu chí | HTTP | HTTPS |
|----------|------|-------|
| Mã hoá | ❌ Plaintext | ✅ Mã hoá bằng **TLS** |
| Cổng mặc định | **80** | **443** |
| Chứng chỉ (certificate) | Không cần | **Cần TLS cert** từ CA |
| Bảo vệ | Không | Bí mật + toàn vẹn + xác thực server |
| Nguy cơ | Sniff, MITM, sửa dữ liệu | Chống được sniff/MITM |
| URL & trình duyệt | `http://` — hiện **"Not Secure"** | `https://` — hiện **ổ khoá** |
| Hiệu năng/tính năng | Không có HTTP/2*, nhiều API bị chặn | Hỗ trợ **HTTP/2, HTTP/3**; bắt buộc cho **service worker, geolocation, PWA...** |
| SEO | Bất lợi | Được Google **ưu tiên** |

<sub>* HTTP/2 về lý thuyết có thể chạy không TLS (h2c) nhưng trình duyệt **chỉ hỗ trợ HTTP/2 qua HTTPS**.</sub>

#### Hiểu đúng (hay bị nhầm)

- **HTTPS không mã hoá *mọi thứ*:** **domain** (qua SNI) và **IP** đích vẫn quan sát được; TLS chỉ giấu
  **path, header, cookie, body**.
- **HTTPS ≠ website an toàn/uy tín.** Ổ khoá chỉ đảm bảo **kết nối được mã hoá & đúng server của domain
  đó** — **không** đảm bảo nội dung không lừa đảo (site phishing vẫn có thể xin được cert Let's Encrypt).
- HTTP thiếu cả 3 đảm bảo → **tuyệt đối không** truyền dữ liệu nhạy cảm qua HTTP (liên hệ
  [câu 12](#q12)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **HTTPS = HTTP + TLS.** HTTP truyền **plaintext** (cổng **80**) → bị **sniff/MITM/sửa**; HTTPS bọc
  **TLS** (cổng **443**) → **mã hoá + toàn vẹn + xác thực server** qua **chứng chỉ CA**.
- Khác biệt kèm theo: HTTPS cần **certificate**, trình duyệt hiện **ổ khoá** (HTTP hiện *Not Secure*), hỗ
  trợ **HTTP/2/3**, bắt buộc cho nhiều Web API, và **tốt cho SEO**.
- **Nhớ kỹ:** HTTPS **không giấu domain/IP** (chỉ giấu path/header/body) và **không đảm bảo website uy
  tín** — chỉ đảm bảo *kết nối an toàn tới đúng server*.

[↑ Mục lục](#mục-lục)

<a id="q12"></a>
## 12. Sensitive Data Exposure là gì?

### 📖 Giới thiệu kiến thức

**Sensitive Data Exposure** (lộ dữ liệu nhạy cảm) là việc dữ liệu quan trọng — mật khẩu, thẻ ngân hàng,
thông tin cá nhân (PII), hồ sơ y tế, token/API key... — **bị lộ do không được bảo vệ đúng cách**: không
mã hoá, mã hoá yếu, hoặc rò rỉ qua log/lỗi/URL. Đây là một trong những lỗ hổng trong danh mục ở
[câu 2](#q2).

> Trong **OWASP Top 10 2021**, nhóm này được mở rộng và đổi tên thành **Cryptographic Failures** — nhấn
> mạnh nguyên nhân gốc thường là **dùng sai (hoặc thiếu) mã hoá**.

#### Nguyên nhân thường gặp

- **Truyền không mã hoá**: dùng HTTP thuần thay vì **HTTPS/TLS** → bị sniff trên đường truyền.
- **Lưu mật khẩu sai**: plaintext hoặc hash yếu (MD5/SHA1) — xem thêm [câu 10](#q10).
- **Không mã hoá dữ liệu khi lưu** (at rest): DB/backup/file để trần.
- **Thuật toán/khoá yếu**: DES, RC4, key ngắn, **hardcode key trong source**, không xoay khoá.
- **Rò rỉ qua kênh phụ**: log chứa secret, **thông báo lỗi lộ thông tin**, dữ liệu nhạy cảm trên **URL**,
  bị **cache**.
- **API trả về thừa trường** (vd trả cả `password_hash`, `ssn` khi client không cần).

#### Phân biệt Hashing vs Encryption (hay hỏi)

- **Hashing** — **một chiều**, không khôi phục lại được. Dùng cho **mật khẩu** (Argon2/bcrypt).
- **Encryption** — **hai chiều** (có key giải mã). Dùng cho dữ liệu **cần đọc lại** (số thẻ, thông tin
  cá nhân) — vd **AES-256**.

→ Mật khẩu thì **băm**, không "mã hoá hai chiều"; dữ liệu cần dùng lại thì **mã hoá** có quản lý khoá.
(Đào sâu ở [câu 12.1](#q12-1).)

#### Cách phòng chống

- **TLS ở mọi nơi** (kèm **HSTS**) cho dữ liệu khi truyền (in transit).
- **Mã hoá khi lưu** (at rest) bằng thuật toán mạnh (**AES-256**); **băm mật khẩu** bằng Argon2/bcrypt.
- **Quản lý khoá đúng**: không hardcode, dùng **secret manager/KMS/Vault**, **xoay khoá** định kỳ.
- **Giảm thiểu lưu trữ (data minimization)**: không lưu thứ không cần (vd không lưu CVV thẻ).
- **Không log/không hiển thị** secret; **mask** dữ liệu (vd `**** **** **** 1234`).
- **Tắt cache** cho response nhạy cảm; **không đặt dữ liệu nhạy cảm trên URL**.
- **Giới hạn trường trả về** của API (chỉ trả thứ client cần).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Sensitive Data Exposure** = dữ liệu nhạy cảm (mật khẩu, thẻ, PII, token) **bị lộ vì không bảo vệ
  đúng cách**: không mã hoá, mã hoá yếu, hoặc rò rỉ qua log/lỗi/URL. OWASP 2021 gọi là **Cryptographic
  Failures**.
- **Nguyên nhân:** thiếu **TLS**, lưu mật khẩu plaintext/hash yếu, không mã hoá at rest, thuật toán/khoá
  yếu hoặc **hardcode key**, log/URL/error lộ secret, API trả thừa trường.
- **Hashing vs Encryption:** mật khẩu → **băm một chiều** (Argon2/bcrypt); dữ liệu cần đọc lại → **mã
  hoá hai chiều** (AES-256) có quản lý khoá.
- **Chống (cốt lõi):** **TLS + HSTS**, **mã hoá at rest**, **quản lý khoá** (KMS/Vault, không hardcode),
  **data minimization**, không log/mask secret, tắt cache nhạy cảm, giới hạn trường API trả về.

[↑ Mục lục](#mục-lục)

<a id="q12-1"></a>
### 12.1. Hashing và Encryption khác nhau thế nào? Cách lưu password vào DB?

#### 📖 Giới thiệu kiến thức

Đây là câu **cực hay bị hỏi** vì nhiều người nhầm ba khái niệm **hashing / encryption / encoding**.

**1) Ba khái niệm — đừng nhầm:**

| | **Hashing** | **Encryption** | **Encoding** |
|---|-------------|----------------|--------------|
| Chiều | **Một chiều** (không giải ngược) | **Hai chiều** (mã hoá ↔ giải mã) | Hai chiều (đổi biểu diễn) |
| Khoá (key) | **Không** | **Có** — cần key để giải | **Không** |
| Mục đích | Định danh/toàn vẹn, **lưu mật khẩu** | **Giữ bí mật** dữ liệu cần đọc lại | Đổi định dạng cho dễ truyền/lưu |
| Output cố định? | Cùng input → cùng hash (độ dài cố định) | Có thể khác nhau (IV/nonce) | Xác định |
| Ví dụ | SHA-256, **bcrypt, Argon2** | **AES-256**, RSA | **Base64**, URL-encode, Hex |

> ⚠️ **Base64 KHÔNG phải bảo mật** — chỉ là encoding, ai cũng decode được. Nhầm "đã Base64 nên an toàn"
> là lỗi kinh điển.

**2) Vì sao mật khẩu phải HASH chứ không ENCRYPT?**

- Hệ thống **không bao giờ cần đọc lại mật khẩu gốc** — khi user đăng nhập, ta chỉ cần **băm lại rồi so
  khớp**. Vậy nên không cần khả năng giải ngược.
- Nếu **encrypt** (hai chiều): phải giữ **key giải mã** ở đâu đó. DB rò rỉ **kèm key** → lộ **toàn bộ**
  mật khẩu gốc. Hashing một chiều thì dù lộ DB, attacker vẫn phải brute-force từng cái.

→ **Nguyên tắc:** *mật khẩu → **hash**; dữ liệu cần đọc lại (số thẻ, PII) → **encrypt** có quản lý khoá.*

**3) Nhưng KHÔNG phải hash nào cũng được — 3 tầng bảo vệ khi lưu password:**

- **(a) Không dùng hash NHANH** (MD5, SHA-1, SHA-256). Chúng thiết kế để **nhanh** → GPU băm **hàng tỉ
  lần/giây** → brute-force & tra **rainbow table** (bảng hash dựng sẵn) rất nhanh. (Xem SHA-256 ở
  [algorithm.md câu 2](./algorithm.md#q2).)
- **(b) Salt — bắt buộc.** **Salt** là chuỗi **ngẫu nhiên, riêng mỗi user**, trộn vào trước khi băm:
  - Vô hiệu hoá **rainbow table** (bảng dựng sẵn không còn đúng vì mỗi user có salt khác nhau).
  - Hai user **cùng mật khẩu** → hash **khác nhau** → attacker không đoán được "nhiều người dùng chung pass".
  - Salt **không cần bí mật**, lưu **cùng** hash (bcrypt/Argon2 tự nhúng salt vào chuỗi output).
- **(c) Hash CHẬM & thích nghi (adaptive):** dùng **bcrypt** hoặc **Argon2** (khuyến nghị **Argon2id**) —
  có **work factor / cost** (bcrypt) hoặc **memory + time + parallelism** (Argon2) điều chỉnh được để cố
  tình **làm chậm**, khiến brute-force **quá đắt**. Máy mạnh lên thì chỉ cần **tăng cost**. (Chi tiết cơ
  chế work factor & memory-hard ở [algorithm.md câu 3](./algorithm.md#q3).)
- *(tuỳ chọn nâng cao)* **Pepper** — một secret **chung toàn hệ thống**, lưu **tách khỏi DB** (env/KMS),
  trộn thêm ngoài salt. DB lộ mà không lộ pepper → hash vẫn khó phá.

**4) Trong DB thực sự lưu gì?**

Chỉ lưu **một cột chuỗi hash** (vd `password` / `password_hash`). Với bcrypt/Argon2, chuỗi này **đã tự
gói** thuật toán + tham số cost + salt + hash — **không cần cột salt riêng**:

```text
$argon2id$v=19$m=65536,t=4,p=1$<salt-base64>$<hash-base64>
  │         │      │              │             └ hash cuối
  │         │      └ tham số (memory=64MB, time=4, parallel=1)
  │         └ version           └ salt (ngẫu nhiên, nhúng sẵn)
  └ thuật toán
```

**5) Luồng đăng ký & đăng nhập:**

```php
// ĐĂNG KÝ — băm rồi lưu chuỗi hash (KHÔNG bao giờ lưu plaintext)
$hash = password_hash($plainPassword, PASSWORD_ARGON2ID);   // tự sinh salt + cost
// INSERT INTO users(email, password) VALUES(?, $hash)

// ĐĂNG NHẬP — băm lại input & so khớp (so sánh constant-time bên trong)
if (password_verify($inputPassword, $hashFromDB)) {
    // đúng mật khẩu → tạo session/token
    // (tuỳ chọn) nâng cost khi cần: nếu password_needs_rehash() → băm lại & cập nhật
}
```

```js
// Node.js — Argon2 (hoặc bcrypt)
import argon2 from 'argon2';
const hash = await argon2.hash(plainPassword);        // lưu hash
const ok   = await argon2.verify(hashFromDB, input);  // xác thực
```

> **Không tự cài thuật toán, không tự nối salt bằng tay** — dùng API sẵn (`password_hash`/`password_verify`,
> `argon2`/`bcrypt`). Chúng đã lo salt, cost và **so sánh constant-time** (xem [câu 14](#q14)) đúng cách.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Hashing = một chiều, không có khoá, không giải ngược** (SHA-256, bcrypt, Argon2). **Encryption = hai
  chiều, có khoá giải mã** (AES-256), dùng cho dữ liệu **cần đọc lại**. (**Base64 chỉ là encoding, không
  bảo mật**.)
- **Password → HASH, không encrypt:** hệ thống chỉ cần **băm lại & so khớp**, không cần đọc lại pass gốc;
  nếu encrypt thì lộ DB **kèm key** là lộ hết.
- **Lưu password đúng cách (3 tầng):** (1) **không** dùng hash nhanh MD5/SHA (GPU + rainbow table);
  (2) **salt** ngẫu nhiên riêng mỗi user (chống rainbow table, che user trùng pass); (3) **hash chậm thích
  nghi** **bcrypt/Argon2id** có **cost** tăng dần. Tuỳ chọn thêm **pepper** lưu tách DB.
- **DB chỉ lưu 1 chuỗi hash** — bcrypt/Argon2 tự nhúng **thuật toán + cost + salt** vào chuỗi, không cần
  cột salt riêng.
- **Thực hành:** `password_hash`/`password_verify` (PHP) hoặc `argon2`/`bcrypt` (Node) — **đừng tự chế**;
  chúng đã lo salt, cost và **so sánh constant-time**.

[↑ Mục lục](#mục-lục)

<a id="q12-2"></a>
### 12.2. Cách compare (verify) password đã hash khi đăng nhập?

#### 📖 Giới thiệu kiến thức

**Câu hỏi cốt lõi:** hash là **một chiều, không giải ngược** — vậy khi user đăng nhập, làm sao so mật
khẩu họ nhập với hash trong DB? **Không giải mã hash** (không thể). Thay vào đó: **băm lại mật khẩu vừa
nhập bằng ĐÚNG salt + tham số của hash đã lưu, rồi so hai hash với nhau.**

**Vì sao "băm lại rồi so" hoạt động?** Vì hash **tất định (deterministic)**: cùng input + cùng salt +
cùng cost → **luôn ra cùng hash**. Nên nếu mật khẩu nhập đúng, hash mới sẽ khớp hash cũ.

**Vấn đề: salt lấy đâu ra để băm lại?** Salt ngẫu nhiên riêng mỗi user. Với **bcrypt/Argon2**, salt và
tham số cost **được nhúng ngay trong chuỗi hash** đã lưu — nên thư viện **tự trích ra** để băm lại. Đó là
lý do chỉ cần lưu **một cột hash** (xem [câu 12.1](#q12-1)).

**Các bước bên trong `password_verify($input, $storedHash)`:**

```text
storedHash = $argon2id$v=19$m=65536,t=4,p=1$<salt>$<hash-cũ>

1. Parse storedHash  → tách ra: thuật toán, cost (m,t,p), salt, hash-cũ
2. Băm lại $input    → dùng CHÍNH salt + cost vừa tách  → hash-mới
3. So sánh hash-mới với hash-cũ  → bằng SO SÁNH CONSTANT-TIME (chống timing attack)
4. Trả về true/false
```

Bước 3 dùng **constant-time comparison** (không dừng sớm ở byte khác đầu tiên) để không rò rỉ thông tin
qua thời gian phản hồi — chi tiết ở [câu 14](#q14).

**❌ Hai lỗi sai kinh điển khi tự so:**

```php
// SAI 1 — tự băm lại bằng password_hash rồi so chuỗi.
// bcrypt/Argon2 sinh SALT NGẪU NHIÊN MỖI LẦN → hash khác nhau → luôn FALSE dù pass đúng!
if (password_hash($input, PASSWORD_ARGON2ID) === $storedHash) { ... }   // ❌ không bao giờ khớp

// SAI 2 — với hash + salt tự quản, so bằng '===' → RÒ RỈ THỜI GIAN (timing attack)
if (hash('sha256', $salt.$input) === $storedHash) { ... }               // ❌ dừng sớm, và SHA quá nhanh
```

**✅ Đúng — dùng hàm verify chuyên dụng (đã tự trích salt + so constant-time):**

```php
// PHP
if (password_verify($input, $storedHash)) {
    // đúng mật khẩu → tạo session/token

    // (nên có) nếu chuẩn cost đã nâng, băm lại để "nâng cấp" hash cũ
    if (password_needs_rehash($storedHash, PASSWORD_ARGON2ID)) {
        $new = password_hash($input, PASSWORD_ARGON2ID);   // lúc này còn plaintext trong tay
        // UPDATE users SET password = $new WHERE id = ...
    }
}
```

```js
// Node.js
const ok = await argon2.verify(storedHash, input);   // Argon2 tự trích salt + so an toàn
// hoặc bcrypt: const ok = await bcrypt.compare(input, storedHash);
```

> **Chốt:** không "so hash", mà **verify** — đưa **plaintext + storedHash** vào hàm verify; nó tự băm lại
> đúng salt và so **constant-time**. Đừng tự nối salt, đừng dùng `==`/`===` trên hash mật khẩu.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Hash **một chiều** nên **không giải ngược** hash để lấy pass gốc. Verify = **băm lại mật khẩu vừa nhập
  bằng đúng salt + cost của hash đã lưu, rồi so hai hash** (hash tất định nên input đúng sẽ khớp).
- **Salt** lấy từ đâu? bcrypt/Argon2 **nhúng salt + cost ngay trong chuỗi hash** → hàm verify tự trích ra
  để băm lại; đó là lý do chỉ cần lưu 1 cột hash.
- **Dùng hàm chuyên dụng:** `password_verify` (PHP), `argon2.verify` / `bcrypt.compare` (Node) — chúng tự
  trích salt **và** so **constant-time** (chống timing attack, [câu 14](#q14)).
- **Tránh 2 lỗi:** (1) băm lại bằng `password_hash` rồi so chuỗi → **luôn sai** vì salt ngẫu nhiên mỗi
  lần; (2) so bằng `==`/`===` → rò rỉ thời gian. Có thể kèm `password_needs_rehash` để nâng cấp cost.

[↑ Mục lục](#mục-lục)

<a id="q12-3"></a>
### 12.3. Sử dụng salt trong hash password như thế nào?

#### 📖 Giới thiệu kiến thức

**Salt** là một chuỗi **ngẫu nhiên**, **duy nhất cho mỗi mật khẩu**, được **trộn vào mật khẩu trước khi
băm**. Mục đích: làm cho hai người **cùng mật khẩu** vẫn ra **hash khác nhau**, và vô hiệu hoá các bảng
hash dựng sẵn. Đây là phần đào sâu của tier "salt" đã nêu ở [câu 12.1](#q12-1).

**1) Vì sao PHẢI có salt? — 3 mối đe doạ salt hoá giải:**

- **Rainbow table** — bảng khổng lồ **map sẵn** `hash → password` cho hàng tỉ mật khẩu phổ biến. Nếu băm
  "trần" (không salt), attacker chỉ cần **tra bảng** là ra pass ngay. Salt ngẫu nhiên khiến bảng dựng sẵn
  **vô dụng** (phải dựng lại bảng riêng cho **từng** salt → bất khả thi).
- **Hash trùng lộ mật khẩu trùng** — không salt: hai user cùng pass `123456` → **cùng hash**. Nhìn DB là
  biết "những ai xài chung một mật khẩu", và crack 1 lần là ra hết. Salt riêng mỗi user → hash **khác nhau**.
- **Precomputation nói chung** — attacker không thể **tính trước** hash trước khi biết salt (salt chỉ có
  sau khi DB bị lộ) → buộc phải brute-force **online**, **từng account một**.

```text
Không salt:  hash("123456")           = 8d969eef...   ← ai xài 123456 cũng ra hash này → tra bảng 1 phát
Có salt:     hash("x7Qa" + "123456")  = 4f2b9c...      user A (salt x7Qa)
             hash("pL0k" + "123456")  = e91d77...      user B (salt pL0k) → KHÁC HẲN dù cùng pass
```

**2) Salt phải như thế nào?**

- **Ngẫu nhiên mật mã (CSPRNG)** — dùng `random_bytes()` (PHP), `crypto.randomBytes()` (Node), **không**
  dùng `rand()`/`mt_rand()` hay timestamp (đoán được).
- **Duy nhất mỗi user** (và **mỗi lần đổi mật khẩu** sinh salt mới).
- **Đủ dài** — thường **≥ 16 byte (128-bit)**. bcrypt dùng salt 128-bit, Argon2 khuyến nghị ≥ 16 byte.
- **KHÔNG cần bí mật** — salt là để chống precompute, **không phải secret**. Cứ lưu **cùng** hash trong DB.

**3) Lưu salt ở đâu? — bcrypt/Argon2 tự lo:**

Không cần cột `salt` riêng. Các hàm `password_hash` (bcrypt/Argon2) **tự sinh salt** và **nhúng thẳng** vào
chuỗi kết quả, nên chỉ lưu **một cột hash** (xem thêm [câu 12.1](#q12-1)):

```text
$2y$12$Wq3v9kZ8s1uJrN7pQxO2ee<...>     ← bcrypt
 │  │  └ SALT (22 ký tự, 128-bit)  └ hash
 │  └ cost = 12 (2^12 vòng)
 └ thuật toán ($2y$ = bcrypt)
```

Khi verify, hàm **đọc lại salt từ chính chuỗi này** để băm lại input rồi so ([câu 12.2](#q12-2)).

**4) Code — để thư viện tự sinh salt (khuyến nghị):**

```php
// PHP — KHÔNG tự nối salt; password_hash tự sinh salt CSPRNG + nhúng vào output
$hash = password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);
// hoặc Argon2id: password_hash($password, PASSWORD_ARGON2ID);
// → lưu $hash (đã gồm salt). Verify: password_verify($input, $hash)
```

```js
// Node.js — bcrypt tự sinh salt theo saltRounds
const hash = await bcrypt.hash(password, 12);   // salt nhúng trong hash
// hoặc argon2.hash(password) — Argon2id, tự salt
```

**5) ❌ Sai lầm thường gặp với salt:**

- **Salt tĩnh/dùng chung** (hardcode 1 salt cho mọi user) → mất tác dụng chống "hash trùng" & rainbow riêng.
- **Salt quá ngắn / không ngẫu nhiên** (dùng username, email, timestamp) → đoán được, dựng bảng được.
- **Tự nối salt rồi băm bằng MD5/SHA** — salt đúng nhưng hash **quá nhanh** vẫn bị brute-force. Salt **không
  thay** cho hàm băm chậm; phải đi kèm **bcrypt/Argon2** ([algorithm.md câu 3](./algorithm.md#q3)).
- **Nhầm salt với pepper** — **salt**: ngẫu nhiên/công khai/mỗi user, lưu cùng DB. **Pepper**: một secret
  **chung**, lưu **tách DB** (env/KMS), trộn thêm để phòng khi DB lộ. Dùng cả hai là tốt nhất.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Salt = chuỗi ngẫu nhiên, riêng mỗi user, trộn vào mật khẩu trước khi băm.** Mục đích: chống **rainbow
  table** và khiến hai người cùng pass ra **hash khác nhau**.
- **Yêu cầu:** sinh bằng **CSPRNG** (`random_bytes`/`crypto.randomBytes`), **duy nhất mỗi user**, **≥ 16
  byte**, **không cần bí mật** (lưu cùng hash).
- **Lưu ở đâu:** với **bcrypt/Argon2**, salt được **tự sinh và nhúng trong chuỗi hash** → chỉ lưu 1 cột;
  verify đọc lại salt từ đó ([câu 12.2](#q12-2)).
- **Thực hành:** để `password_hash`/`bcrypt`/`argon2` **tự sinh salt** — đừng tự nối tay.
- **Tránh:** salt tĩnh/dùng chung, salt ngắn/đoán được, salt + hash nhanh (MD5/SHA) — salt **không thay**
  cho hàm băm chậm. Phân biệt **salt** (công khai, mỗi user) vs **pepper** (secret chung, lưu tách DB).

[↑ Mục lục](#mục-lục)

<a id="q12-4"></a>
### 12.4. Phân biệt salt và pepper chi tiết

#### 📖 Giới thiệu kiến thức

Cả **salt** và **pepper** đều là **giá trị thêm vào mật khẩu trước khi băm** để tăng an toàn, nhưng khác
nhau ở **4 điểm cốt lõi**: **có bí mật không**, **phạm vi** (mỗi user hay toàn hệ thống), **lưu ở đâu**, và
**chống mối đe doạ nào**.

**Bảng so sánh:**

| Tiêu chí | **Salt** | **Pepper** |
|----------|----------|------------|
| Bản chất | Chuỗi **ngẫu nhiên** | **Secret** ngẫu nhiên (như một khoá) |
| Bí mật? | **Không** — công khai được | **CÓ** — phải giữ kín |
| Phạm vi | **Riêng mỗi user** (mỗi hash 1 salt) | **Chung** toàn hệ thống (1 giá trị cho nhiều user) |
| Lưu ở đâu | **Cùng DB**, nhúng trong chuỗi hash | **TÁCH khỏi DB**: biến môi trường, **KMS/Vault/HSM** |
| Chống gì | **Rainbow table** + che **pass trùng** | Thêm lớp: **DB lộ mà pepper KHÔNG lộ** → vẫn không phá được |
| Có sẵn trong bcrypt/Argon2? | **Có** — tự sinh | **Không** — phải **tự thêm** |
| Xoay vòng (rotation) | Không cần (đổi pass → salt mới) | Có thể, nhưng cần **versioning** |

**Khác biệt then chốt — mô hình đe doạ (threat model):**

- **Salt** bảo vệ ngay cả khi salt bị lộ (nó vốn công khai): nó chỉ chặn **precompute/rainbow table** và
  che pass trùng. Nhưng nếu **DB lộ**, attacker vẫn **brute-force offline** từng account được (vì có cả
  salt lẫn hash).
- **Pepper** vá đúng lỗ hổng đó: pepper **không nằm trong DB**. Nếu attacker chỉ lấy được **DB** (qua SQL
  injection, lộ backup...) **mà không lấy được application secret** → thiếu pepper → **không thể brute-force**
  (không biết đã trộn thêm gì). Điểm mấu chốt: **pepper chỉ có giá trị khi kênh làm lộ DB KHÁC kênh giữ
  secret**.
- Hệ quả: nếu attacker **chiếm luôn app server / KMS** (lấy được pepper) thì pepper **vô dụng** — lúc đó chỉ
  còn **salt + hàm băm chậm** che chắn. → **Pepper là defense-in-depth, KHÔNG thay** salt/slow hash.

**Cách áp dụng pepper ĐÚNG:**

- **Đừng** chỉ nối chuỗi `pepper . password` rồi bcrypt — **bcrypt cắt input ở 72 byte**, pepper dài có thể
  bị bỏ qua.
- Khuyến nghị: **HMAC mật khẩu với pepper key trước**, rồi mới đưa vào bcrypt/Argon2:

```php
// pepper lấy từ secrets manager / biến môi trường — KHÔNG hardcode, KHÔNG để trong DB
$pepper = getenv('PASSWORD_PEPPER');           // vd 32 byte ngẫu nhiên

// ĐĂNG KÝ: HMAC(password, pepper) → rồi mới hash chậm (salt do bcrypt tự lo)
$pre  = hash_hmac('sha256', $password, $pepper, true);
$hash = password_hash(base64_encode($pre), PASSWORD_ARGON2ID);   // lưu $hash

// ĐĂNG NHẬP: lặp lại đúng bước tiền xử lý rồi verify
$pre = hash_hmac('sha256', $input, $pepper, true);
if (password_verify(base64_encode($pre), $hashFromDB)) { /* đúng */ }
```

> Lý do HMAC trước: pepper đóng vai **khoá** (không phải salt), HMAC trộn nó an toàn và cho output cố định
> ngắn gọn để bcrypt/Argon2 không bị giới hạn độ dài.

**Xoay pepper (rotation):** vì pepper dùng chung, muốn đổi phải lưu **version** (vd cột `pepper_id`) để biết
hash cũ dùng pepper nào, rồi **re-hash dần** khi user đăng nhập thành công. Salt không có vấn đề này — mỗi
lần đổi mật khẩu là một salt mới tự nhiên.

**Kết luận thực dụng:** **salt là bắt buộc** (và bcrypt/Argon2 lo sẵn); **pepper là tuỳ chọn nâng cao** cho
hệ nhạy cảm, thêm một lớp phòng thủ khi **chỉ DB bị lộ**. Dùng **cả hai** là tốt nhất.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Salt:** ngẫu nhiên, **công khai**, **mỗi user**, lưu **cùng DB** (bcrypt/Argon2 nhúng sẵn) → chống
  **rainbow table** + che pass trùng.
- **Pepper:** một **secret chung** toàn hệ thống, lưu **TÁCH DB** (env/KMS) → thêm lớp: **DB lộ mà pepper
  không lộ** thì hash **vẫn không brute-force được**.
- **Khác cốt lõi:** **bí mật hay không** + **nơi lưu** + **phạm vi**. Salt chống **precompute**; pepper chống
  **brute-force offline** — **chỉ khi** kênh lộ DB **khác** kênh giữ secret.
- **Pepper là defense-in-depth:** attacker chiếm cả app (lấy được pepper) → pepper vô dụng, **không thay**
  salt/slow hash. **bcrypt/Argon2 có salt sẵn nhưng KHÔNG có pepper** — phải tự thêm.
- **Áp dụng đúng:** `bcrypt(HMAC(password, pepper))` (tránh giới hạn 72 byte của bcrypt); pepper ở secrets
  manager; có thể **versioning** để xoay. Salt không cần xoay.

[↑ Mục lục](#mục-lục)

<a id="q12-5"></a>
### 12.5. Chặn user đặt mật khẩu mới trùng 3 mật khẩu gần nhất (password history)

#### 📖 Giới thiệu kiến thức

**Bài toán:** khi user đổi mật khẩu, mật khẩu mới **không được trùng 3 mật khẩu gần nhất** họ từng dùng
(password history / prevent password reuse — yêu cầu phổ biến trong chính sách bảo mật, NIST/ISO).

**⚠️ Cái bẫy kỹ thuật (điểm mấu chốt):** ta **lưu hash chứ không lưu plaintext**, mà bcrypt/Argon2 sinh
**salt ngẫu nhiên mỗi bản ghi** → **cùng một mật khẩu băm hai lần ra hai hash KHÁC nhau** ([câu 12.3](#q12-3)).
Vì vậy **KHÔNG thể** phát hiện trùng bằng cách so **chuỗi hash**. Bắt buộc phải dùng **`password_verify(mật
khẩu mới, hash cũ)`** cho **từng** hash trong lịch sử — để nó tự trích salt của hash cũ, băm lại mật khẩu
mới bằng đúng salt đó rồi so ([câu 12.2](#q12-2)).

**Thiết kế:**

**1) Schema — bảng lịch sử (không bao giờ lưu plaintext):**

```sql
CREATE TABLE password_histories (
    id          BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id     BIGINT NOT NULL,
    password_hash VARCHAR(255) NOT NULL,   -- hash bcrypt/Argon2 (đã gồm salt)
    created_at  DATETIME NOT NULL,
    INDEX idx_user_created (user_id, created_at)   -- lấy nhanh N cái gần nhất
);
```

**2) Luồng đổi mật khẩu:**

```php
function changePassword(int $userId, string $current, string $new): void
{
    $user = getUser($userId);

    // (a) Re-auth: xác nhận đúng mật khẩu HIỆN TẠI trước khi cho đổi
    if (!password_verify($current, $user->password)) {
        throw new Error('Mật khẩu hiện tại không đúng');
    }

    // (b) Lấy N=3 hash gần nhất của user (bao gồm cả mật khẩu đang dùng)
    //     Gộp cả $user->password để chắc chắn tính luôn mật khẩu hiện tại.
    $recentHashes = queryColumn(
        "SELECT password_hash FROM password_histories
         WHERE user_id = ? ORDER BY created_at DESC LIMIT 3", [$userId]
    );

    // (c) So mật khẩu MỚI với từng hash cũ bằng password_verify (KHÔNG so chuỗi hash!)
    foreach ($recentHashes as $oldHash) {
        if (password_verify($new, $oldHash)) {
            throw new Error('Mật khẩu mới không được trùng 3 mật khẩu gần nhất');
        }
    }

    // (d) Hợp lệ → băm & cập nhật, ghi lịch sử, rồi prune giữ bảng gọn
    $newHash = password_hash($new, PASSWORD_ARGON2ID);
    transaction(function () use ($userId, $newHash) {
        updateUserPassword($userId, $newHash);
        insertHistory($userId, $newHash);              // created_at = now()
        pruneHistory($userId, keep: 3);                // xoá bản ghi cũ hơn 3 cái gần nhất
    });
}
```

**3) Các điểm cần lưu ý:**

- **Định nghĩa "3 gần nhất":** thường **tính cả mật khẩu hiện tại** (current + 2 cái trước). Cứ **ghi mọi
  mật khẩu vào history** ngay khi đổi → truy vấn `LIMIT 3` là đúng. Cần chốt rõ N với đề bài.
- **Prune để bảng không phình:** chỉ giữ vài bản ghi/user (vd N hoặc N+1). Nếu cần **audit** thì giữ nhiều
  hơn nhưng vẫn chỉ **query 3 cái** khi kiểm tra.
- **Có pepper?** phải áp dụng **cùng bước tiền xử lý** (`HMAC(password, pepper)`) trước khi verify, đồng bộ
  với lúc tạo hash ([câu 12.4](#q12-4)) — nếu không sẽ luôn "không trùng" sai.
- **Hiệu năng:** mỗi `password_verify` (bcrypt/Argon2) cố tình **chậm** (~50–200ms). 3 lần ≈ vài trăm ms —
  **chấp nhận được** vì đổi mật khẩu **không phải hot path**. Đừng hạ cost chỉ để nhanh chỗ này.
- **Race condition:** hai request đổi mật khẩu song song → bọc bước cập nhật + ghi history trong
  **transaction** (khoá theo `user_id`) để lịch sử nhất quán.
- **Đổi thuật toán/cost theo thời gian không sao:** `password_verify` đọc tham số **từ chính từng hash cũ**,
  nên hash cũ (bcrypt cost 10) và mới (Argon2id) vẫn kiểm được bình thường.
- **Không dùng hash nhanh (MD5/SHA) để "so cho tiện":** vẫn phải là bcrypt/Argon2; salt + slow hash là bắt
  buộc ([câu 12.1](#q12-1)).

> **Laravel:** hiện thực gọn bằng một **custom validation rule** (`Rule` gọi `Hash::check($new, $oldHash)`
> qua từng bản ghi history) + model `PasswordHistory`; `Hash::check` chính là `password_verify`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Cái bẫy:** hash có **salt ngẫu nhiên mỗi bản ghi** → cùng mật khẩu ra hash khác nhau → **KHÔNG so chuỗi
  hash được**. Phải **`password_verify(mật khẩu mới, hash cũ)`** cho **từng** hash lịch sử.
- **Thiết kế:** bảng **`password_histories`** (user_id, **password_hash**, created_at) — chỉ lưu **hash**,
  không plaintext. Lấy **3 hash gần nhất**, verify mật khẩu mới với từng cái; **trùng bất kỳ → từ chối**.
- **Sau khi hợp lệ:** băm mật khẩu mới, cập nhật user, **ghi history**, **prune** giữ N bản ghi. Trước đó
  **re-auth** bằng mật khẩu hiện tại.
- **Lưu ý:** có **pepper** thì HMAC trước khi verify; **3 lần bcrypt** không phải hot path nên OK; bọc
  **transaction** chống race; đổi cost/thuật toán vẫn verify được; định nghĩa rõ "3 gần nhất" (thường gồm
  current).

[↑ Mục lục](#mục-lục)

<a id="q13"></a>
## 13. Các hàm hash và mã hoá phổ biến

### 📖 Giới thiệu kiến thức

Đây là câu hỏi **tổng quan về primitive mật mã** — nắm được "có những họ hàm nào, dùng vào việc gì" là
nền để trả lời mọi câu về lưu mật khẩu, JWT, TLS, chữ ký số. Nhắc lại phân biệt ba nhóm (chi tiết ở
[câu 12.1](#q12-1)):

- **Hash** — **một chiều**, không khoá, không giải ngược (integrity, lưu mật khẩu).
- **Encryption** — **hai chiều**, có **khoá** để giải mã (giữ bí mật dữ liệu cần đọc lại).
- **MAC / Chữ ký số** — chứng minh **toàn vẹn + nguồn gốc** của dữ liệu.

Và nhớ: **Base64 / Hex chỉ là encoding**, không phải mã hoá — không có tính bảo mật.

---

#### A. CÁC HÀM HASH

Một hàm hash mật mã tốt cần: **deterministic** (cùng input → cùng output), **fixed-size** (đầu ra độ dài
cố định), **pre-image resistance** (từ hash không suy ngược ra input), **second pre-image & collision
resistance** (khó tìm hai input cùng hash), và **avalanche effect** (đổi 1 bit input → đổi ~nửa số bit
output). Chia làm ba nhóm theo mục đích:

**1) Hash đa dụng — thiết kế để NHANH** (integrity, checksum, dedup, dựng chữ ký số):

| Hàm | Độ dài output | Trạng thái |
|-----|:---:|-----------|
| **MD5** | 128-bit | ❌ **Đã vỡ** (collision) — chỉ dùng checksum không bảo mật |
| **SHA-1** | 160-bit | ❌ **Đã vỡ** (collision 2017) — ngừng dùng |
| **SHA-2** (SHA-256/384/512) | 256–512-bit | ✅ Tiêu chuẩn hiện tại |
| **SHA-3** (Keccak) | tuỳ chọn | ✅ Chuẩn mới, cấu trúc khác SHA-2 (dự phòng) |

> ⚠️ Vì **nhanh**, nhóm này **TUYỆT ĐỐI KHÔNG** dùng để lưu mật khẩu (GPU băm hàng tỉ lần/giây +
> rainbow table). Chúng dành cho **kiểm tra toàn vẹn file, checksum, làm khối xây chữ ký số/HMAC**.

**2) Hash mật khẩu — thiết kế để CHẬM & tốn tài nguyên** (adaptive):

- **bcrypt** — có **work factor / cost** tăng dần theo sức máy.
- **scrypt** — **memory-hard** (tốn RAM), chống ASIC/GPU.
- **Argon2** (khuyến nghị **Argon2id**) — chỉnh được **memory + time + parallelism**; hiện đại nhất.

→ Dùng cho **lưu mật khẩu** (chi tiết cơ chế ở [câu 12.1](#q12-1) và [algorithm.md câu 3](./algorithm.md#q3)).

**3) HMAC (Hash-based MAC) — hash CÓ khoá:** kết hợp một hàm hash với một **secret key** để tạo mã xác
thực: `HMAC-SHA256(key, message)`. Cùng một message nhưng **không có key thì không giả được** MAC. Dùng
cho: **ký JWT HS256** ([câu 9.5](#q9-5)), xác thực **webhook**, chống giả mạo request. (So sánh MAC phải
**constant-time** — [câu 14](#q14).)

---

#### B. CÁC HÀM MÃ HOÁ (ENCRYPTION)

Chia hai loại lớn theo **khoá**:

**1) Mã hoá đối xứng (Symmetric) — MỘT khoá cho cả mã hoá và giải mã:**

- **AES** (Advanced Encryption Standard) — chuẩn de-facto, key **128/192/256-bit**. Chạy theo **mode**:
  - **AES-GCM** — *authenticated encryption* (vừa mã hoá vừa đảm bảo toàn vẹn) → **khuyên dùng**.
  - **AES-CBC / CTR** — chỉ mã hoá, cần ghép thêm MAC để chống sửa; CBC cần **IV** ngẫu nhiên.
- **ChaCha20-Poly1305** — nhanh trên thiết bị không có tăng tốc AES (mobile), cũng authenticated.
- **DES / 3DES / RC4** — ❌ **cũ, yếu, đã loại bỏ**.

→ **Ưu:** rất nhanh, hợp **mã hoá khối lượng lớn** (dữ liệu at rest, file, DB). **Nhược:** **bài toán phân
phối khoá** — hai bên phải cùng biết một khoá bí mật mà không lộ khi trao đổi.

**2) Mã hoá bất đối xứng (Asymmetric) — CẶP khoá public/private:**

- **RSA** — cổ điển, key ≥ **2048-bit**; dùng mã hoá khoá phiên & chữ ký.
- **ECC** (Elliptic Curve: **ECDSA**, **ECDH**, **Ed25519**) — khoá **ngắn hơn** mà an toàn tương đương
  RSA (ECC 256-bit ≈ RSA 3072-bit) → nhanh, nhẹ, đang thay thế RSA.

Cơ chế: mã bằng **public key** → chỉ **private key** giải được (giữ bí mật); hoặc ký bằng **private key**
→ ai cũng verify bằng **public key** (chữ ký số). **Ưu:** giải quyết **phân phối khoá** (public key phát
công khai). **Nhược:** **chậm** → không mã hoá dữ liệu lớn.

**3) Mã hoá lai (Hybrid) — cách TLS/HTTPS thực sự làm:**

**Vấn đề (vì sao phải lai):** hai loại trên mỗi loại giỏi một nửa —
- **Đối xứng (AES):** rất **nhanh**, hợp dữ liệu lớn, nhưng vướng **phân phối khoá** — làm sao đưa **khoá
  chung** qua mạng hở mà kẻ nghe lén (sniff) không tóm được?
- **Bất đối xứng (RSA/ECC):** **giải quyết phân phối khoá** (public key phát công khai) nhưng **chậm** →
  không dùng để mã dữ liệu lớn.

**Ý tưởng lai:** dùng **bất đối xứng chỉ để trao một khoá phiên** (session key) **một lần**, rồi dùng
**đối xứng (AES) mã toàn bộ dữ liệu** về sau → **lấy cái nhanh của đối xứng + giải quyết phân phối khoá
của bất đối xứng**.

**Các bước cụ thể** (mô hình RSA key-transport cho dễ hình dung):

```text
1. Client sinh NGẪU NHIÊN 1 session key (khoá đối xứng, vd AES-256 key)
2. Client mã hoá session key đó bằng PUBLIC key của server
3. Gửi gói đã mã qua mạng → kẻ nghe lén chỉ thấy khối rác;
   chỉ PRIVATE key của server mới giải ra được
4. Server dùng PRIVATE key giải → hai bên giờ CÙNG có 1 session key
5. Từ đây: mọi dữ liệu mã bằng AES + session key đó (NHANH)
```

**Hình dung nhanh (analogy):** public key như một **ổ khoá đang mở** ai cũng bấm khoá được, nhưng **chỉ
chủ nhân** (private key) mới mở ra. Bỏ session key vào hộp → bấm ổ khoá của server → gửi đi; dọc đường
không ai mở được, chỉ server mở.

> **Lưu ý chính xác:** TLS **hiện đại thường không** dùng RSA key-transport như trên, mà dùng **ECDHE
> key-agreement** — hai bên *cùng tạo ra* session key thay vì một bên gửi, để có **forward secrecy** (lộ
> private key sau này cũng không giải được phiên cũ). Nhưng **ý tưởng lai vẫn y hệt**: bất đối xứng lo
> khâu khoá, đối xứng mã dữ liệu. Chi tiết handshake xem [câu 11](#q11).

**Dùng ở đâu:** **TLS/HTTPS**, **JWE** (`alg` bọc **CEK** — [câu 9.10](#q9-10)), **PGP/email**, **Signal**.

---

#### C. CHỮ KÝ SỐ (Digital Signature)

Ghép **hash + bất đối xứng**: băm message → **ký hash bằng private key**; bên nhận **verify bằng public
key**. Đảm bảo cùng lúc **3 tính chất**: **integrity** (không bị sửa), **authenticity** (đúng người ký),
**non-repudiation** (không chối được đã ký). Đây là nền của JWT **RS256/ES256** ([câu 9.8](#q9-8)),
chứng chỉ TLS, ký code/commit.

---

#### Bảng chọn công cụ theo nhu cầu (rất hay hỏi)

| Nhu cầu | Dùng gì |
|---------|---------|
| **Lưu mật khẩu** | **Argon2id** (hoặc bcrypt/scrypt) — hash chậm + salt |
| **Kiểm tra toàn vẹn file / checksum** | **SHA-256** |
| **Xác thực message/webhook có khoá** | **HMAC-SHA256** |
| **Mã hoá dữ liệu at rest / khối lượng lớn** | **AES-256-GCM** (đối xứng) |
| **Trao đổi khoá / chữ ký số** | **RSA-2048+** hoặc **ECC (Ed25519)** (bất đối xứng) |
| **Bảo mật đường truyền** | **TLS** = hybrid (asymmetric trao khoá + AES mã dữ liệu) |
| **Chỉ đổi định dạng (KHÔNG bảo mật)** | Base64 / Hex (encoding) |

### 🎯 Tóm tắt trả lời phỏng vấn

- **3 nhóm primitive:** **Hash** (một chiều), **Encryption** (hai chiều, có khoá), **MAC/chữ ký số**
  (toàn vẹn + nguồn gốc). **Base64 là encoding, không phải bảo mật.**
- **Hash chia 3 loại theo mục đích:** (1) **đa dụng nhanh** — MD5/SHA-1 (đã vỡ), **SHA-256/SHA-3** cho
  checksum & toàn vẹn; (2) **hash mật khẩu chậm** — **bcrypt/scrypt/Argon2id**; (3) **HMAC** = hash **có
  khoá** để xác thực message (JWT HS256, webhook). **Không dùng SHA nhanh để lưu mật khẩu.**
- **Encryption 2 loại:** **Đối xứng** (1 khoá, **AES-256-GCM**) — nhanh, cho dữ liệu lớn nhưng vướng
  **phân phối khoá**; **Bất đối xứng** (public/private, **RSA/ECC**) — giải quyết phân phối khoá & làm
  **chữ ký số** nhưng **chậm**. **TLS lai cả hai:** bất đối xứng trao **session key**, rồi AES mã dữ liệu.
- **Chữ ký số** = hash + ký bằng private key, verify bằng public key → **integrity + authenticity +
  non-repudiation** (nền RS256/ES256, chứng chỉ TLS).
- **Chọn nhanh:** mật khẩu → **Argon2id**; toàn vẹn → **SHA-256**; xác thực có khoá → **HMAC-SHA256**;
  mã dữ liệu → **AES-256-GCM**; trao khoá/ký → **RSA-2048+/Ed25519**.

[↑ Mục lục](#mục-lục)

<a id="q14"></a>
## 14. So sánh constant-time là gì? (timing attack)

### 📖 Giới thiệu kiến thức

**So sánh constant-time (constant-time comparison)** là cách so sánh hai chuỗi/byte **mất thời gian như
nhau bất kể chúng khác nhau ở đâu** — nhằm chống **timing attack** (tấn công kênh phụ dựa vào **thời
gian**). Kỹ thuật này được nhắc khi verify chữ ký JWT ([câu 9.5](#q9-5)) và HMAC ([câu 9.8](#q9-8)).

#### Vì sao so sánh chuỗi thông thường lại nguy hiểm?

Phép so sánh chuỗi bình thường (`==`, `strcmp`, `===`) **dừng ngay khi gặp byte đầu tiên khác nhau**
(*short-circuit*). Nghĩa là **thời gian so sánh tỉ lệ với số ký tự khớp từ đầu**:

```text
So "aXXXXX" với secret "abcdef" → sai ngay ký tự 2 → trả về RẤT NHANH
So "abcdeX" với secret "abcdef" → khớp 5 ký tự → trả về CHẬM HƠN một chút
```

Kẻ tấn công **đo thời gian phản hồi** → biết mình đoán đúng được bao nhiêu ký tự đầu → **dò từng byte một**
(byte-by-byte) cho tới khi ra toàn bộ secret. Áp dụng để đoán: **HMAC/chữ ký, token, API key, session id,
CSRF token, thậm chí so sánh hash mật khẩu**.

#### Constant-time hoạt động thế nào?

**Luôn so sánh HẾT toàn bộ độ dài**, không dừng sớm — thường bằng cách **XOR từng byte rồi OR dồn** kết
quả, chỉ đến cuối mới kết luận:

```text
diff = 0
for i in range(len):
    diff |= a[i] XOR b[i]     # khác nhau ở đâu cũng vẫn chạy hết
return diff == 0             # thời gian không phụ thuộc VỊ TRÍ khác biệt
```

→ Thời gian **không rò rỉ** thông tin về việc khớp được bao nhiêu ký tự.

#### Dùng hàm có sẵn (đừng tự chế)

| Ngôn ngữ | Hàm constant-time |
|----------|-------------------|
| PHP | **`hash_equals($known, $user)`** |
| Node.js | **`crypto.timingSafeEqual(a, b)`** |
| Python | **`hmac.compare_digest(a, b)`** |
| Go | **`subtle.ConstantTimeCompare(a, b)`** |
| Java | `MessageDigest.isEqual(a, b)` |

```php
// ❌ Rò rỉ thời gian — dừng sớm ở byte khác đầu tiên
if ($userToken === $realToken) { ... }

// ✅ Constant-time
if (hash_equals($realToken, $userToken)) { ... }
```

> Lưu ý: với **mật khẩu**, các hàm như `password_verify` (bcrypt/Argon2) **đã tự so sánh constant-time**
> bên trong — không cần tự làm. Constant-time cần chú ý khi **tự so sánh HMAC/token/chữ ký thủ công**.

### 🎯 Tóm tắt trả lời phỏng vấn

- **So sánh constant-time** = so sánh hai chuỗi **mất thời gian như nhau dù khác ở đâu**, để chống **timing
  attack**.
- **Vấn đề:** so sánh thường (`==`) **dừng sớm** ở byte khác đầu tiên → thời gian rò rỉ "đoán đúng mấy ký
  tự" → attacker **dò secret từng byte** (HMAC, token, API key, session id).
- **Cách làm:** **luôn duyệt hết** toàn bộ (XOR-OR dồn), không short-circuit → thời gian độc lập với vị
  trí khác biệt.
- **Thực tế:** dùng hàm sẵn — **`hash_equals`** (PHP), **`crypto.timingSafeEqual`** (Node),
  **`hmac.compare_digest`** (Python); **đừng tự chế**. `password_verify` đã constant-time sẵn.

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
