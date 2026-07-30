<a id="mục-lục"></a>
# Common — Thuật toán & Cấu trúc dữ liệu (Algorithm & Data Structure)

> Độ phức tạp (Big-O) · mảng/chuỗi · stack/queue · linked list · hash map · cây/đồ thị ·
> sorting/searching · đệ quy · quy hoạch động · two-pointer/sliding-window.
> (Các mảng khác đã tách riêng: [HTTP/REST](./common.md) · [Bảo mật](./security.md) ·
> [OOP](./oop.md) · [SOLID & Design Pattern](./solid-design-pattern.md) · [Testing](./testing.md).)

## 📑 Mục lục

- [1. Tìm kiếm nhị phân (Binary Search) là gì?](#q1)
- [2. SHA-256 là gì?](#q2)
- [3. bcrypt / Argon2 là gì?](#q3)
- [4. Phân biệt băm nhanh (fast hash) và băm chậm (slow hash)](#q4)

---

<a id="q1"></a>
## 1. Tìm kiếm nhị phân (Binary Search) là gì?

### 📖 Giới thiệu kiến thức

**Tìm kiếm nhị phân** là thuật toán tìm một phần tử trong **mảng đã được sắp xếp** (sorted) bằng
cách **liên tục chia đôi** phạm vi tìm kiếm. Thay vì duyệt tuần tự từng phần tử (linear search,
O(n)), nó tận dụng tính chất "đã sắp xếp" để mỗi bước **loại bỏ một nửa** số ứng viên còn lại.

**Ý tưởng cốt lõi:** so sánh phần tử cần tìm (`target`) với phần tử ở **giữa** mảng:
- Nếu **bằng** → tìm thấy, trả về vị trí.
- Nếu `target` **nhỏ hơn** phần tử giữa → đáp án (nếu có) nằm ở **nửa trái** → bỏ nửa phải.
- Nếu `target` **lớn hơn** → đáp án nằm ở **nửa phải** → bỏ nửa trái.

Lặp lại trên nửa còn lại cho đến khi tìm thấy hoặc phạm vi rỗng (không tồn tại).

**Ví dụ trực quan** — tìm `target = 23` trong mảng `[2, 5, 8, 12, 16, 23, 38, 56, 72, 91]`:

| Bước | left | right | mid | arr[mid] | So sánh | Hành động |
|------|------|-------|-----|----------|---------|-----------|
| 1 | 0 | 9 | 4 | 16 | 23 > 16 | tìm nửa phải → `left = 5` |
| 2 | 5 | 9 | 7 | 56 | 23 < 56 | tìm nửa trái → `right = 6` |
| 3 | 5 | 6 | 5 | 23 | 23 = 23 | ✅ tìm thấy tại index 5 |

Chỉ **3 bước** thay vì duyệt tới 6 phần tử — và với mảng 1 triệu phần tử cũng chỉ tốn **~20 bước**.

**Điều kiện bắt buộc:** mảng **phải đã sắp xếp**. Nếu mảng chưa sắp xếp, phải sort trước
(O(n log n)) — khi đó chỉ tìm 1 lần thì không lợi; binary search đáng giá khi **tìm nhiều lần**
trên cùng mảng đã sort.

**Độ phức tạp:**
- **Time: O(log n)** — mỗi bước giảm phạm vi đi một nửa.
- **Space: O(1)** với bản lặp (iterative); **O(log n)** với bản đệ quy (do call stack).

**Code (PHP) — bản lặp (iterative), khuyến nghị:**

```php
function binarySearch(array $arr, int $target): int
{
    $left = 0;
    $right = count($arr) - 1;

    while ($left <= $right) {
        // Tránh tràn số: dùng $left + (...) thay vì ($left + $right) / 2
        $mid = intdiv($left + ($right - $left), 2);

        if ($arr[$mid] === $target) {
            return $mid;            // tìm thấy → trả về index
        } elseif ($arr[$mid] < $target) {
            $left = $mid + 1;       // bỏ nửa trái
        } else {
            $right = $mid - 1;      // bỏ nửa phải
        }
    }

    return -1;                      // không tồn tại
}
```

**Code (JavaScript) — bản đệ quy (recursive):**

```js
function binarySearch(arr, target, left = 0, right = arr.length - 1) {
  if (left > right) return -1;                 // phạm vi rỗng → không có
  const mid = Math.floor(left + (right - left) / 2);

  if (arr[mid] === target) return mid;
  return arr[mid] < target
    ? binarySearch(arr, target, mid + 1, right) // tìm nửa phải
    : binarySearch(arr, target, left, mid - 1); // tìm nửa trái
}
```

**Bẫy thường gặp (interviewer hay hỏi):**
- **Tính `mid` bị tràn số:** `(left + right) / 2` có thể tràn khi `left + right` rất lớn. Dùng
  `left + (right - left) / 2` để an toàn.
- **Điều kiện vòng lặp:** dùng `while (left <= right)` (có dấu `=`), nếu thiếu `=` sẽ bỏ sót
  trường hợp mảng còn 1 phần tử.
- **Cập nhật cận:** phải là `mid + 1` / `mid - 1`, không phải `mid` — nếu để `mid` dễ gây
  **vòng lặp vô hạn**.
- **Biến thể:** tìm phần tử **đầu tiên/cuối cùng** thỏa điều kiện (lower/upper bound) — không dừng
  ngay khi gặp mà tiếp tục thu hẹp; đây là nền tảng của `binary search on answer`.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Binary search** là thuật toán tìm phần tử trong **mảng đã sắp xếp** bằng cách **chia đôi**
  phạm vi tìm kiếm mỗi bước → loại nửa không chứa đáp án.
- So sánh `target` với phần tử **giữa**: bằng thì xong; nhỏ hơn thì tìm **nửa trái**; lớn hơn thì
  tìm **nửa phải**. Lặp đến khi tìm thấy hoặc phạm vi rỗng.
- **Độ phức tạp: O(log n)** time, **O(1)** space (bản lặp). Mảng 1 triệu phần tử chỉ ~20 bước.
- **Điều kiện bắt buộc:** mảng **phải đã sắp xếp**.
- Lưu ý code: tính `mid` an toàn (`left + (right - left) / 2`), vòng lặp `while (left <= right)`,
  cập nhật cận `mid ± 1` để tránh lặp vô hạn.

[↑ Mục lục](#mục-lục)

---

<a id="q2"></a>
## 2. SHA-256 là gì?

### 📖 Giới thiệu kiến thức

**SHA-256** (Secure Hash Algorithm 256-bit) là một **hàm băm mật mã (cryptographic hash function)**
thuộc họ **SHA-2** (do NSA thiết kế, NIST chuẩn hoá). Nó nhận **đầu vào bất kỳ độ dài** và luôn cho ra
một **chuỗi băm cố định 256 bit** (= 32 byte, thường viết dưới dạng **64 ký tự hex**).

```text
SHA-256("hello")  = 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
SHA-256("hellp")  = 7f1a... (đổi 1 ký tự → hash khác HOÀN TOÀN — avalanche effect)
```

**4 tính chất của một hàm băm mật mã:**

1. **Deterministic (tất định):** cùng input → luôn cùng output.
2. **One-way (một chiều):** từ hash **không suy ngược** ra input (chỉ có thể thử-và-đối chiếu).
3. **Avalanche effect:** đổi **1 bit** input → hash thay đổi **~50% bit** → không đoán được quan hệ.
4. **Collision resistance:** **cực khó** tìm hai input khác nhau cho **cùng** một hash.

> **Hash ≠ mã hoá:** hash **một chiều, không có khoá, không giải ngược**; mã hoá **hai chiều, có khoá**
> để giải lại. SHA-256 là hash → dùng để **kiểm tra toàn vẹn / định danh dữ liệu**, không phải để giấu.

**SHA-256 dùng để làm gì?**
- **Kiểm tra toàn vẹn (integrity):** checksum file, verify download (so hash).
- **Chữ ký số & JWT:** phần "SHA-256" trong **HS256/RS256/ES256** chính là hàm băm này
  ([bảo mật — HS256/RS256/ES256](./security.md#q10-8)).
- **Hash token/API key** để lưu (token entropy cao nên SHA-256 là đủ —
  [bảo mật — lưu refresh token](./security.md#q10-12)).
- **Blockchain / Proof-of-Work** (Bitcoin dùng SHA-256), **Merkle tree**, **deduplication**.

**⚠️ KHÔNG dùng SHA-256 để lưu MẬT KHẨU.** Vì nó **quá nhanh** → GPU băm hàng tỷ chuỗi/giây → dễ
brute-force. Mật khẩu phải dùng **hàm băm chậm có salt** như **bcrypt/Argon2** (xem [câu 3](#q3)).

**Phân biệt nhanh trong họ SHA:**
- **SHA-1 (160-bit)** — **đã bị phá** (tìm được collision) → **không dùng** cho mục đích bảo mật.
- **SHA-2** (SHA-256, SHA-512...) — **an toàn**, phổ biến nhất hiện nay.
- **SHA-3** — chuẩn mới hơn (cấu trúc khác, Keccak), dự phòng/bổ sung cho SHA-2.

```php
// PHP
$hash = hash('sha256', 'hello');   // 64 ký tự hex

// So khớp an toàn (chống timing attack) — xem security.md
hash_equals($known, hash('sha256', $input));
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **SHA-256** = hàm băm mật mã họ **SHA-2**, biến input **bất kỳ độ dài** thành hash **cố định 256-bit
  (64 hex)**.
- **4 tính chất:** tất định, **một chiều** (không giải ngược), **avalanche** (đổi 1 bit → hash khác hẳn),
  **chống collision**. **Hash ≠ mã hoá** (không khoá, không giải lại).
- **Dùng cho:** kiểm tra **toàn vẹn/checksum**, **chữ ký số & JWT** (SHA-256 trong HS256/RS256/ES256),
  **hash token/API key**, blockchain.
- **KHÔNG dùng cho mật khẩu** (quá nhanh → dễ brute-force) → mật khẩu dùng **bcrypt/Argon2** ([câu 3](#q3)).
  **SHA-1 đã bị phá**, nên dùng **SHA-2/SHA-3**.

[↑ Mục lục](#mục-lục)

---

<a id="q3"></a>
## 3. bcrypt / Argon2 là gì?

### 📖 Giới thiệu kiến thức

**bcrypt** và **Argon2** là hai **thuật toán băm mật khẩu** (password hashing algorithm) chuyên dùng
để **lưu trữ mật khẩu an toàn**. Chúng thuộc nhóm **hàm băm chậm có chủ đích** (slow / adaptive hash),
khác hẳn với các hàm băm nhanh như MD5, SHA-1, SHA-256.

**Vì sao KHÔNG dùng MD5/[SHA-256](#q2) để lưu mật khẩu?**
- Các hàm này được thiết kế để **chạy cực nhanh** → GPU hiện đại băm được **hàng tỷ chuỗi/giây**.
  Kẻ tấn công lấy được database có thể **brute-force / dò từ điển** rất nhanh.
- Chúng **không có salt sẵn** và **không có cost** (chi phí tính toán) điều chỉnh được.

Ý tưởng của bcrypt/Argon2 là **cố tình làm chậm** việc băm (vài chục–vài trăm ms/lần) để việc dò
mật khẩu trở nên **cực kỳ tốn kém**, trong khi người dùng đăng nhập hợp lệ chỉ tốn 1 lần nên không
ảnh hưởng trải nghiệm.

**Các đặc tính chung quan trọng:**

1. **Salt tự động** — mỗi mật khẩu được thêm một chuỗi ngẫu nhiên (salt) trước khi băm, và salt được
   **nhúng luôn vào chuỗi hash kết quả**. Nhờ vậy hai người dùng cùng mật khẩu `123456` vẫn ra hai
   hash khác nhau → **vô hiệu hóa rainbow table** (bảng tra cứu hash dựng sẵn).
2. **Cost factor (work factor)** — tham số điều chỉnh **độ chậm**. Phần cứng càng mạnh, ta chỉ cần
   **tăng cost** là thuật toán chậm lại tương ứng → gọi là **adaptive** (thích nghi theo thời gian).
3. **One-way** — băm một chiều, không giải ngược được. Khi đăng nhập ta **băm lại và so sánh**, không
   bao giờ giải mã.

---

**bcrypt (1999)** — dựa trên thuật toán mã hóa Blowfish.
- Tham số chính: **cost** (còn gọi `rounds`/`work factor`), mặc định thường **10–12**. Mỗi lần tăng
  cost lên 1 → thời gian băm **gấp đôi** (2^cost vòng lặp).
- **Giới hạn:** chỉ băm tối đa **72 byte** đầu của mật khẩu (các ký tự sau bị bỏ qua).
- **Chỉ tốn CPU** — không kháng tốt tấn công bằng phần cứng chuyên dụng (GPU/ASIC) như Argon2.

Cấu trúc chuỗi hash bcrypt: `$2y$12$R9h/cIPz0gi.URNNX3kh2O...`
`$2y$` = phiên bản · `12` = cost · phần còn lại = **salt (22 ký tự) + hash** gộp chung.

**Argon2 (2015)** — **người thắng cuộc thi Password Hashing Competition**, được khuyến nghị hiện đại.
- Có **3 biến thể**: **Argon2id** (khuyến nghị mặc định, lai giữa i và d), Argon2i (kháng side-channel),
  Argon2d (kháng GPU tốt nhất nhưng dễ side-channel).
- **3 tham số** điều chỉnh, mạnh hơn bcrypt vì kiểm soát được cả **bộ nhớ**:
  - **memory cost** — lượng RAM mỗi lần băm (ví dụ 64 MB). Đây là điểm mạnh: buộc kẻ tấn công phải
    tốn **nhiều RAM**, khiến GPU/ASIC (vốn ít RAM/lõi) trở nên kém hiệu quả → **memory-hard**.
  - **time cost** — số vòng lặp (iterations).
  - **parallelism** — số luồng song song.

**So sánh nhanh:**

| Tiêu chí | MD5 / SHA-256 | bcrypt | Argon2 (id) |
|----------|---------------|--------|-------------|
| Mục đích | Băm tổng quát | Băm mật khẩu | Băm mật khẩu |
| Tốc độ | Rất nhanh (xấu cho mật khẩu) | Chậm có chủ đích | Chậm có chủ đích |
| Salt | Không (tự thêm) | Có sẵn | Có sẵn |
| Cost điều chỉnh | Không | CPU (cost) | CPU + **RAM** + luồng |
| Kháng GPU/ASIC | Kém | Trung bình | **Tốt nhất** (memory-hard) |
| Năm / trạng thái | Cũ, không dùng cho pass | 1999, vẫn ổn | 2015, khuyến nghị |

**Code (PHP) — dùng API `password_hash`, không tự cài đặt thuật toán:**

```php
// bcrypt (mặc định của Laravel), cost = 12
$hash = password_hash($plainPassword, PASSWORD_BCRYPT, ['cost' => 12]);

// Argon2id (khuyến nghị nếu môi trường hỗ trợ)
$hash = password_hash($plainPassword, PASSWORD_ARGON2ID, [
    'memory_cost' => 65536,  // 64 MB
    'time_cost'   => 4,      // 4 vòng lặp
    'threads'     => 2,      // 2 luồng
]);

// Khi đăng nhập: BĂM LẠI + so sánh (salt lấy tự động từ chính $hash lưu trong DB)
if (password_verify($plainPassword, $hashFromDB)) {
    // đúng mật khẩu
    // nâng cost theo thời gian: nếu hash cũ yếu hơn cấu hình hiện tại thì băm lại
    if (password_needs_rehash($hashFromDB, PASSWORD_ARGON2ID, $options)) {
        $newHash = password_hash($plainPassword, PASSWORD_ARGON2ID, $options);
        // lưu $newHash vào DB
    }
}
```

**Code (Node.js):**

```js
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(plain, 12);      // saltRounds = 12, salt tự sinh
const ok   = await bcrypt.compare(plain, hash); // so sánh

// Argon2 (thư viện argon2)
const argon2 = require('argon2');
const h  = await argon2.hash(plain, { type: argon2.argon2id, memoryCost: 65536, timeCost: 4 });
const ok2 = await argon2.verify(h, plain);
```

**Lưu ý interviewer hay đào sâu:**
- **Không tự thêm salt riêng** — bcrypt/Argon2 đã tự sinh và nhúng salt; tự làm dễ sai.
- **Không dùng "pepper" thay cho salt** — pepper (khóa bí mật chung, lưu ngoài DB) là lớp **bổ sung**,
  không thay thế cơ chế salt sẵn có.
- **Nên chọn Argon2id** cho hệ thống mới; **bcrypt** vẫn chấp nhận được và phổ biến (Laravel mặc định).
- **Không dùng SHA/MD5** cho mật khẩu, kể cả có salt — vì vẫn quá nhanh.
- **Cost cần đo thực tế:** chỉnh sao cho mỗi lần băm tốn ~**200–500 ms** trên server production.

### 🎯 Tóm tắt trả lời phỏng vấn

- **bcrypt và Argon2** là **thuật toán băm mật khẩu chậm có chủ đích** (adaptive hash), dùng để **lưu
  mật khẩu an toàn** — khác với MD5/SHA vốn quá nhanh nên dễ bị brute-force.
- Cả hai đều **tự sinh salt và nhúng vào hash** → chống rainbow table; đều có **cost factor** để điều
  chỉnh độ chậm, tăng dần theo sức mạnh phần cứng (**adaptive**).
- **bcrypt** (1999, từ Blowfish): tham số **cost** (~12), chỉ tốn CPU, giới hạn 72 byte mật khẩu.
- **Argon2** (2015, winner PHC): thêm **memory cost** → **memory-hard**, kháng GPU/ASIC tốt nhất;
  biến thể khuyến nghị là **Argon2id**.
- Thực hành: dùng API sẵn có (`password_hash`/`password_verify` PHP, `bcrypt`/`argon2` Node), **không
  tự cài thuật toán**; chọn Argon2id cho hệ thống mới, chỉnh cost để mỗi lần băm ~200–500 ms.

[↑ Mục lục](#mục-lục)

---

<a id="q4"></a>
## 4. Phân biệt băm nhanh (fast hash) và băm chậm (slow hash)

### 📖 Giới thiệu kiến thức

Cả hai đều là **hàm băm một chiều** (one-way), nhận input bất kỳ → ra hash cố định. Điểm khác **cốt lõi
không phải tốc độ tình cờ, mà là MỤC TIÊU THIẾT KẾ ngược nhau**:

- **Băm nhanh (fast / general-purpose hash)** — thiết kế để **chạy càng nhanh càng tốt**, throughput cao.
  Ví dụ: **MD5, SHA-1, SHA-256, SHA-3** ([câu 2](#q2)).
- **Băm chậm (slow / adaptive / password hash)** — **cố tình làm chậm và tốn tài nguyên**, có **salt +
  cost** điều chỉnh được. Ví dụ: **bcrypt, scrypt, Argon2** ([câu 3](#q3)).

#### Vì sao lại cần hai loại ngược nhau? — nằm ở "mô hình tấn công"

Tốc độ là **tốt hay xấu** tuỳ mục đích dùng:

- **Kiểm tra toàn vẹn / checksum / chữ ký số:** ta cần băm **file lớn, hàng triệu bản ghi** thật nhanh →
  **tốc độ cao là ưu điểm**. Không có "kẻ tấn công đoán input" ở đây, nên nhanh không sao.
- **Lưu mật khẩu:** kẻ tấn công lấy được DB sẽ **đoán mật khẩu bằng brute-force / từ điển**. Nếu hàm băm
  **nhanh**, GPU băm **hàng tỷ chuỗi/giây** → dò ra mật khẩu yếu trong tích tắc. Ở đây **tốc độ là kẻ
  thù** → phải dùng hàm **chậm** để mỗi lần đoán tốn ~vài trăm ms, khiến brute-force **quá đắt**. Người
  dùng hợp lệ chỉ băm **1 lần/đăng nhập** nên không bị ảnh hưởng.

#### Bảng phân biệt

| Tiêu chí | Băm nhanh (SHA-256, MD5) | Băm chậm (bcrypt, Argon2) |
|----------|--------------------------|---------------------------|
| **Mục tiêu thiết kế** | Tốc độ cao, throughput lớn | **Cố tình chậm**, tốn tài nguyên |
| **Tốc độ** | Hàng tỷ hash/giây (GPU) | ~vài chục–vài trăm ms/hash |
| **Salt** | Không có sẵn (phải tự thêm) | **Có sẵn**, tự nhúng vào hash |
| **Cost điều chỉnh (adaptive)** | Không | **Có** — tăng theo sức máy |
| **Memory-hard (tốn RAM)** | Không | **Có** (Argon2/scrypt) → kháng GPU/ASIC |
| **Chống brute-force offline** | ❌ Kém | ✅ Tốt |
| **Dùng cho** | Toàn vẹn, checksum, JWT/chữ ký, HMAC, token entropy cao, blockchain | **Lưu mật khẩu** người dùng |

#### Cái bẫy hay gặp: "SHA-256 + salt cho mật khẩu là đủ" → SAI

- **Salt** chỉ chống **rainbow table** (bảng hash dựng sẵn) và che việc hai user trùng mật khẩu — nó
  **KHÔNG** làm hàm băm chậm lại.
- Có salt nhưng vẫn là SHA-256 thì GPU vẫn băm **hàng tỷ lần/giây** để brute-force từng mật khẩu. → Muốn
  an toàn phải **vừa có salt vừa CHẬM** = đúng bản chất bcrypt/Argon2. (Chi tiết salt/pepper ở
  [security.md câu 13.3–13.4](./security.md#q13-3).)

> **Quy tắc chốt:** *dữ liệu cần verify nhanh & toàn vẹn → **băm nhanh** (SHA-256); bí mật của người
> dùng như mật khẩu → **băm chậm** (Argon2id/bcrypt).* Token/API key entropy cao (ngẫu nhiên dài) thì
> **SHA-256 là đủ** vì không thể brute-force ([security.md câu 10.12](./security.md#q10-12)).

### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai đều là **hàm băm một chiều**; khác nhau ở **mục tiêu thiết kế**: **băm nhanh** tối ưu tốc độ
  (MD5/SHA-256), **băm chậm** cố tình chậm + tốn RAM, có **salt & cost** (bcrypt/Argon2).
- **Vì sao khác nhau — mô hình tấn công:** toàn vẹn/checksum cần **nhanh** (băm file lớn); lưu mật khẩu
  thì **nhanh là kẻ thù** (GPU brute-force hàng tỷ/giây) nên phải **chậm** để mỗi lần đoán quá đắt.
- **Băm nhanh dùng cho:** kiểm tra toàn vẹn, checksum, **chữ ký số/JWT, HMAC**, hash token entropy cao,
  blockchain. **Băm chậm dùng cho:** **lưu mật khẩu** người dùng.
- **Bẫy hay hỏi:** *"SHA-256 + salt cho mật khẩu là đủ"* → **SAI**. Salt chỉ chống rainbow table, không
  làm chậm; vẫn bị brute-force tốc độ cao. Mật khẩu phải **vừa salt vừa chậm** = bcrypt/Argon2id.

[↑ Mục lục](#mục-lục)

---

<!--
NHẮC FORMAT (xem /CLAUDE.md §2 và §4):

Mỗi câu hỏi gồm ĐÚNG 2 phần, theo thứ tự:
  1. ### 📖 Giới thiệu kiến thức  — nền tảng để hiểu (định nghĩa, độ phức tạp, khi nào dùng).
  2. ### 🎯 Tóm tắt trả lời phỏng vấn — bản súc tích để nói khi phỏng vấn.

Với thuật toán: nên có Big-O (time/space), ý tưởng cốt lõi, và code example ngắn (PHP/JS/pseudocode).

<a id="q1"></a>
## 1. <Câu hỏi>

### 📖 Giới thiệu kiến thức
<nền tảng — ý tưởng, độ phức tạp, ví dụ>

### 🎯 Tóm tắt trả lời phỏng vấn
- <ý chính 1>
- <ý chính 2>

[↑ Mục lục](#mục-lục)
-->