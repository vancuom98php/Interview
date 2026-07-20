<a id="mục-lục"></a>
# Backend — PHP Core

> Cú pháp & engine PHP · kiểu dữ liệu · OOP trong PHP · phiên bản & tính năng mới · Composer · PHP-FPM/OPcache.
> (Câu hỏi về Laravel/Eloquent/Artisan nằm ở [laravel.md](./laravel.md).)

## 📑 Mục lục

1. [Falsy và Truthy trong PHP là gì?](#q1)
   - 1.1. [Phân biệt `isset()` và `empty()`](#q1-1)
2. [Tham trị (pass by value) và tham chiếu (pass by reference) trong PHP](#q2)
3. [Hàm ẩn danh, Lambda và Closure trong PHP](#q3)
4. [Lớp trừu tượng (abstract class) trong PHP](#q4)
5. [Interface trong PHP là gì?](#q5)
6. [Trait trong PHP là gì?](#q6)
7. [`static` trong PHP là gì?](#q7)
8. [Phân biệt `$this` và `self` trong PHP](#q8)
9. [Phân biệt `self` và `static` (Late Static Binding)](#q9)
10. [`final` trong PHP là gì?](#q10)
11. [`readonly` trong PHP là gì?](#q11)
12. [Phân biệt `include`, `include_once`, `require`, `require_once`](#q12)
13. [Hàm `__autoload()` và cơ chế autoload trong PHP](#q13)
14. [Các cách cài thư viện vào dự án PHP thuần](#q14)
15. [Các cách chạy một file PHP (hoặc một dự án PHP)](#q15)
16. [PHP phiên bản mới nhất là gì? Có gì hơn các phiên bản thấp hơn nó?](#q16)

---

<a id="q1"></a>
## 1. Falsy và Truthy trong PHP là gì?

### 📖 Giới thiệu kiến thức

PHP là ngôn ngữ **dynamic typing** và có cơ chế **type juggling**: khi một giá trị được dùng trong
**ngữ cảnh boolean** (điều kiện `if`, toán tử `&&`/`||`/`!`, vòng `while`...), PHP **tự ép** giá trị
đó về `true` hoặc `false`. Dựa vào kết quả ép đó, mọi giá trị được chia thành 2 nhóm:

- **Falsy** — bị ép thành `false`.
- **Truthy** — bị ép thành `true` (tất cả những gì *không* falsy).

#### Danh sách các giá trị Falsy (nên thuộc lòng)

Trong PHP chỉ có **một số ít** giá trị là falsy — học thuộc danh sách này là đủ, còn lại đều truthy:

| Giá trị falsy | Ghi chú |
|---------------|---------|
| `false` | boolean false |
| `0` , `-0` | integer 0 |
| `0.0` , `-0.0` | float 0 |
| `""` (chuỗi rỗng) | string rỗng |
| `"0"` | **chuỗi chứa đúng ký tự `0`** (bẫy hay gặp) |
| `[]` (mảng rỗng) | array không phần tử |
| `null` | bao gồm cả biến chưa được gán |

#### Những trường hợp TRUTHY dễ nhầm

Ngược lại, nhiều giá trị **trông như “rỗng/sai”** nhưng thực ra là **truthy**:

```php
var_dump((bool) "0");      // false  ← chuỗi "0" là FALSY
var_dump((bool) "0.0");    // true   ← nhưng "0.0" lại TRUTHY!
var_dump((bool) "false");  // true   ← chuỗi "false" là TRUTHY
var_dump((bool) " ");      // true   ← chuỗi 1 dấu cách là TRUTHY
var_dump((bool) "00");     // true   ← chỉ đúng "0" mới falsy
var_dump((bool) [0]);      // true   ← mảng CÓ phần tử (dù phần tử là 0) là TRUTHY
var_dump((bool) -1);       // true   ← số âm là TRUTHY (chỉ 0 mới falsy)
var_dump((bool) "0.00");   // true
```

→ Điểm bẫy kinh điển: chỉ chuỗi **`"0"`** và **`""`** là falsy; mọi chuỗi khác (kể cả `"0.0"`,
`"false"`, `" "`) đều truthy. Và **mảng rỗng `[]` là falsy**, nhưng mảng có phần tử thì truthy bất kể
phần tử bên trong là gì.

#### Vì sao quan trọng?

Viết `if ($value)` là đang **ép kiểu ngầm**. Với dữ liệu chuỗi từ form/DB, sự khác biệt
`"0"` (falsy) vs `"0.0"` (truthy) có thể gây bug logic. Khi cần kiểm tra **chính xác**, nên dùng so
sánh tuyệt đối `===` thay vì dựa vào type juggling:

```php
if ($qty === 0) { ... }        // rõ ràng: đúng số 0
if ($value !== '') { ... }     // rõ ràng: khác chuỗi rỗng
```

### 🎯 Tóm tắt trả lời phỏng vấn
- PHP có **type juggling**: trong ngữ cảnh boolean, mọi giá trị bị ép về `true`/`false`. **Falsy** =
  bị ép thành `false`, **Truthy** = phần còn lại.
- **Các giá trị falsy** cần nhớ: `false`, `0`, `0.0`, `""`, **`"0"`**, `[]` (mảng rỗng), `null`
  (và biến chưa gán). Tất cả còn lại là **truthy**.
- **Bẫy hay hỏi:** chuỗi `"0"` **falsy** nhưng `"0.0"`, `"false"`, `" "` đều **truthy**; mảng rỗng
  `[]` falsy nhưng `[0]` truthy; số âm truthy.
- Khi cần chắc chắn, dùng so sánh tuyệt đối **`===`** thay vì dựa vào ép kiểu ngầm trong `if`.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Phân biệt `isset()` và `empty()`

#### 📖 Giới thiệu kiến thức

Cả hai là **language construct** (không phải hàm thật) dùng để kiểm tra biến **mà không phát warning**
khi biến chưa tồn tại — đây là lý do chính khiến ta dùng chúng thay vì truy cập trực tiếp.

**`isset($x)` — “biến có tồn tại và KHÁC null không?”**

- Trả về `true` khi biến **đã được khai báo** và giá trị **không phải `null`**.
- Trả về `false` nếu biến **chưa tồn tại** hoặc giá trị **là `null`**.
- Nhận **nhiều tham số**: `isset($a, $b)` chỉ `true` khi *tất cả* đều set.
- Kiểm tra key mảng an toàn: `isset($arr['key'])` → `true` nếu key tồn tại và không null.

**`empty($x)` — “biến có RỖNG (falsy) hoặc không tồn tại không?”**

- Trả về `true` khi biến **không tồn tại** *hoặc* giá trị là **falsy** (xem lại câu 1: `0`, `""`,
  `"0"`, `[]`, `null`, `false`...).
- Trả về `false` khi biến tồn tại **và** có giá trị truthy.
- Cũng **không phát warning** với biến chưa khai báo.

Mấu chốt: `empty()` dùng đúng **bộ quy tắc Falsy** ở câu 1. Vì vậy `empty("0")` trả về `true` (vì
chuỗi `"0"` là falsy) — một bẫy rất hay gặp khi xử lý dữ liệu form.

#### Bảng so sánh theo từng giá trị

| Giá trị của `$x` | `isset($x)` | `empty($x)` |
|------------------|:-----------:|:-----------:|
| Biến chưa khai báo | `false` | `true` |
| `null` | `false` | `true` |
| `""` (chuỗi rỗng) | `true` | `true` |
| `"0"` | `true` | `true` |
| `0` / `0.0` | `true` | `true` |
| `false` | `true` | `true` |
| `[]` (mảng rỗng) | `true` | `true` |
| `"hello"` | `true` | `false` |
| `1` / `"0.0"` / `[0]` | `true` | `false` |

→ Quy tắc nhớ nhanh: **`isset` chỉ quan tâm “có tồn tại & khác null”**; **`empty` quan tâm “rỗng theo
nghĩa falsy”**. Một biến có thể `isset` = `true` nhưng `empty` cũng `= true` (vd `$x = 0` hay `""`).

```php
$data = ['name' => '', 'age' => 0, 'role' => null];

isset($data['name']);   // true  — key tồn tại, giá trị "" khác null
empty($data['name']);   // true  — "" là falsy

isset($data['role']);   // false — giá trị null
empty($data['role']);   // true

isset($data['city']);   // false — key không tồn tại (không warning)
empty($data['city']);   // true
```

#### Liên hệ: toán tử `??` (null coalescing)

`??` hoạt động đúng theo logic của `isset`: trả về vế phải khi vế trái **chưa set hoặc null** — gọn
hơn nhiều so với viết `isset()` thủ công:

```php
$name = $data['name'] ?? 'Khách'; // dùng 'Khách' nếu key không tồn tại HOẶC null
// tương đương: isset($data['name']) ? $data['name'] : 'Khách'
```

Lưu ý: `??` chỉ chặn **null/chưa set**, *không* chặn chuỗi rỗng `""` hay `0`. Muốn “rỗng thì lấy mặc
định” thì dùng `?:` (vì nó theo logic falsy): `$name = $data['name'] ?: 'Khách';`.

#### 🎯 Tóm tắt trả lời phỏng vấn
- Cả hai **không gây warning** khi biến chưa tồn tại — đó là công dụng chính.
- **`isset($x)`** = `true` khi biến **tồn tại VÀ khác `null`** (null → `false`). Nhận nhiều tham số,
  kiểm tra key mảng an toàn.
- **`empty($x)`** = `true` khi biến **không tồn tại HOẶC giá trị falsy** (`0`, `""`, `"0"`, `[]`,
  `false`, `null`). Tức `empty` dùng đúng bộ quy tắc **falsy** ở câu trên.
- **Khác biệt cốt lõi:** `isset` hỏi *“có & khác null”*, `empty` hỏi *“có rỗng (falsy) không”*. Vd
  `$x = 0`: `isset` → `true` nhưng `empty` → `true`. Bẫy: **`empty("0")` là `true`**.
- Liên quan: **`??`** đi theo logic `isset` (chỉ chặn null/chưa set), còn **`?:`** đi theo logic falsy.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Tham trị (pass by value) và tham chiếu (pass by reference) trong PHP

### 📖 Giới thiệu kiến thức

Đây là cách PHP **truyền giá trị của biến** (vào hàm, hoặc khi gán biến này cho biến khác). Có 2 cơ chế:

#### 1. Tham trị — pass by value (mặc định)

Khi truyền một biến vào hàm, PHP truyền một **bản sao của giá trị**. Hàm thao tác trên bản sao →
**sửa bên trong hàm KHÔNG ảnh hưởng** biến gốc bên ngoài.

```php
function addOne(int $n) {
    $n = $n + 1;   // chỉ sửa bản sao
}

$a = 10;
addOne($a);
echo $a; // 10 — biến gốc không đổi
```

#### 2. Tham chiếu — pass by reference (thêm dấu `&`)

Thêm `&` trước tham số → hàm nhận **chính biến gốc** (cùng vùng nhớ), không phải bản sao. Sửa trong
hàm **ảnh hưởng trực tiếp** biến ngoài.

```php
function addOne(int &$n) {  // & = nhận tham chiếu
    $n = $n + 1;
}

$a = 10;
addOne($a);
echo $a; // 11 — biến gốc bị thay đổi
```

`&` cũng dùng được khi **gán biến** (`$b = &$a;` → `$b` và `$a` là cùng một ô nhớ) và trong
**`foreach`** (`foreach ($arr as &$item)` để sửa trực tiếp phần tử mảng).

#### 3. Điểm hay nhầm nhất: Object

Từ PHP 5, một biến object **không chứa object**, mà chứa một **handle (định danh)** trỏ tới object
trong bộ nhớ. Vì vậy khi truyền object **theo tham trị**, ta copy *handle* — nhưng bản sao handle
**vẫn trỏ tới CÙNG một object**:

```php
class Box { public int $value = 0; }

function fillBox(Box $b) {        // tham trị: $b là bản sao handle, vẫn trỏ cùng object
    $b->value = 99;              // ✅ sửa thuộc tính → ảnh hưởng object gốc
    $b = new Box();              // ❌ gán object MỚI → chỉ đổi bản sao handle, không ảnh hưởng ngoài
}

$box = new Box();
fillBox($box);
echo $box->value; // 99
```

→ Nhiều người lầm tưởng "object luôn pass by reference". **Sai**: object truyền theo *tham trị của
handle*. Khác biệt: **sửa thuộc tính** thì ảnh hưởng (vì cùng object), nhưng **gán object mới
(reassign)** thì *không* ảnh hưởng biến ngoài — còn pass-by-reference thật (`&`) thì reassign cũng ảnh hưởng.

#### 4. Cơ chế Copy-on-Write (CoW)

Để khỏi tốn bộ nhớ, PHP **không copy ngay** khi truyền theo tham trị: ban đầu các biến **dùng chung**
một vùng giá trị (zval), chỉ đến khi có **ghi (write)** thì PHP mới thực sự tách bản sao. Về *ngữ
nghĩa* vẫn là tham trị, nhưng về *hiệu năng* việc copy được trì hoãn — nên truyền mảng lớn theo tham
trị không tự động đắt như ta tưởng (chỉ đắt khi sửa).

| Tiêu chí | Tham trị (mặc định) | Tham chiếu (`&`) |
|----------|---------------------|-------------------|
| Truyền cái gì | **Bản sao** giá trị | **Chính biến gốc** (cùng ô nhớ) |
| Sửa trong hàm | Không ảnh hưởng biến ngoài | **Ảnh hưởng** biến ngoài |
| Cú pháp | `function f($x)` | `function f(&$x)` |
| Object: sửa property | Ảnh hưởng (cùng object qua handle) | Ảnh hưởng |
| Object: gán object mới | **Không** ảnh hưởng ngoài | **Có** ảnh hưởng ngoài |

### 🎯 Tóm tắt trả lời phỏng vấn
- **Mặc định PHP là pass by value** — truyền **bản sao giá trị**, nên sửa biến trong hàm **không** đổi
  biến gốc.
- **Thêm `&`** trước tham số (hoặc khi gán `$b = &$a`) → **pass by reference**: cùng vùng nhớ, sửa
  trong hàm **đổi luôn** biến ngoài.
- **Object là bẫy hay gặp:** object truyền theo *tham trị của **handle*** (không phải reference thật).
  Vì handle trỏ cùng object nên **sửa thuộc tính thì ảnh hưởng**; nhưng **gán object mới (reassign)
  thì không** ảnh hưởng biến ngoài. Muốn reassign cũng ảnh hưởng thì phải dùng `&`.
- **Copy-on-Write:** PHP trì hoãn copy đến khi có ghi → truyền mảng lớn theo tham trị **không tốn RAM
  cho tới khi sửa**.
- **Lưu ý thực tế:** dùng `&` dè chừng — gây **side-effect** ngầm, khó debug; thường ưu tiên trả về giá
  trị mới thay vì sửa qua reference.

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Hàm ẩn danh, Lambda và Closure trong PHP

### 📖 Giới thiệu kiến thức

PHP cho phép tạo **hàm không có tên (anonymous function)** và gán nó vào biến, truyền vào hàm khác, hay
trả về từ hàm. Đây là nền tảng của lập trình hàm (functional) trong PHP — dùng nhiều cho **callback**
(`array_map`, `usort`, route handler, event listener...).

#### Phân biệt thuật ngữ: Lambda vs Closure

Hai từ hay bị dùng lẫn, nhưng có sắc thái khác nhau:

- **Lambda** (hàm ẩn danh "thuần") — hàm không tên, **không bắt (capture)** biến nào từ scope bên ngoài.
- **Closure** — hàm ẩn danh **có "bao bọc" (capture) biến từ phạm vi xung quanh** để dùng lại bên trong,
  ngay cả khi scope ngoài đã kết thúc.

> Về kỹ thuật, trong PHP **mọi hàm ẩn danh đều là một object của class `Closure`** (dù có capture hay
> không). Vì vậy người ta thường gọi chung là "closure"; "lambda" chỉ là cách nói cho hàm không capture.

```php
$sayHi = function (string $name): string {
    return "Hi, $name";
};

echo $sayHi('Cuom');          // Hi, Cuom
var_dump($sayHi instanceof Closure);   // true — hàm ẩn danh là object Closure
```

#### `use` — bắt biến từ scope ngoài (value vs reference)

Khác với hàm thường, hàm ẩn danh **không tự thấy** biến ở scope cha; muốn dùng phải khai báo qua `use`.
Và giống như [câu 2](#q2) (tham trị/tham chiếu), việc capture có **2 kiểu** — theo giá trị hoặc theo
tham chiếu:

```php
$rate = 10;

// (1) capture theo GIÁ TRỊ: chốt giá trị $rate ngay lúc ĐỊNH NGHĨA closure
$byValue = function (int $x) use ($rate): int {
    return $x + $rate;
};

// (2) capture theo THAM CHIẾU: dùng & — closure "nhìn" biến gốc, đổi sau vẫn thấy
$byRef = function (int $x) use (&$rate): int {
    return $x + $rate;
};

$rate = 100;
echo $byValue(1);   // 11   ← vẫn dùng $rate=10 đã chốt lúc định nghĩa
echo $byRef(1);     // 101  ← thấy giá trị mới của $rate
```

→ Đây là điểm **bẫy kinh điển**: `use ($x)` chụp **bản sao tại thời điểm định nghĩa**; `use (&$x)` mới
theo dõi biến gốc.

#### Arrow function `fn` (PHP 7.4) — closure rút gọn

`fn` là cú pháp ngắn cho closure, **tự động capture biến ngoài theo giá trị** (không cần `use`), thân chỉ
là **một biểu thức**:

```php
$rate = 10;
$add = fn(int $x): int => $x + $rate;   // tự thấy $rate (by value), khỏi use
echo $add(1);   // 11
```

Rất tiện làm callback gọn:

```php
$nums = [1, 2, 3, 4];
$doubled = array_map(fn($n) => $n * 2, $nums);   // [2, 4, 6, 8]
```

#### Closure trong context object: `$this`, `bindTo`

Closure tạo bên trong một method **tự động bind `$this`** của object đó, nên truy cập được thành viên
của object. Ngoài ra có thể **đổi `$this`** mà closure trỏ tới bằng `Closure::bind()` / `bindTo()`:

```php
class Counter
{
    private int $count = 0;

    public function makeIncrementer(): Closure
    {
        return function () { return ++$this->count; };   // $this = object Counter
    }
}

$inc = (new Counter())->makeIncrementer();
echo $inc(); // 1
echo $inc(); // 2
```

#### First-class callable syntax `...` (PHP 8.1)

Từ PHP 8.1, có cú pháp tạo closure từ một hàm/method có sẵn bằng `(...)` — gọn và an toàn kiểu hơn
chuỗi `'strlen'`:

```php
$fn = strlen(...);          // tạo Closure trỏ tới hàm strlen
echo $fn('hello');          // 5

$lengths = array_map(strlen(...), ['a', 'bb', 'ccc']);   // [1, 2, 3]
```

### 🎯 Tóm tắt trả lời phỏng vấn
- **Hàm ẩn danh** = hàm không tên, gán được vào biến/truyền làm callback. Trong PHP **mọi hàm ẩn danh
  đều là object của class `Closure`**.
- **Lambda vs Closure:** *lambda* = hàm ẩn danh **không capture** biến ngoài; *closure* = hàm ẩn danh
  **có capture** biến từ scope xung quanh (giữ lại dùng sau).
- **Bắt biến bằng `use`:** `use ($x)` capture **theo giá trị** (chốt lúc định nghĩa); `use (&$x)`
  capture **theo tham chiếu** (theo dõi biến gốc) — y như tham trị/tham chiếu ở [câu 2](#q2).
- **Arrow function `fn` (7.4):** closure rút gọn, **tự capture theo giá trị** (khỏi `use`), thân là một
  biểu thức — tiện làm callback.
- Nâng cao: closure trong method **tự bind `$this`** (đổi được bằng `bindTo`); **first-class callable**
  `strlen(...)` (8.1) tạo closure từ hàm có sẵn.

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Lớp trừu tượng (abstract class) trong PHP

### 📖 Giới thiệu kiến thức

**Lớp trừu tượng (abstract class)** là một class được khai báo với từ khóa `abstract`, dùng làm
**khuôn mẫu/lớp cha** cho các class khác kế thừa, và **không thể tạo object trực tiếp** từ nó (không
`new` được). Nó là một cách hiện thực tính **Abstraction** của OOP: định nghĩa *cái khung* (class con
phải có gì), còn *chi tiết thực thi* để class con tự lo.

#### Hai thành phần đặc trưng

- **Abstract method** — method được khai báo (tên + tham số + kiểu trả về) nhưng **không có thân
  (body)**. Nó là một *hợp đồng*: **class con bắt buộc phải override (implement)** method này.
- **Abstract class** có thể chứa **cả abstract method lẫn method thường (có thân)**, kèm
  **property, constant, constructor** — không bị giới hạn chỉ khai báo như interface.

> Quy tắc: hễ một class có **ít nhất một abstract method** thì **bản thân class đó phải là `abstract`**.

```php
abstract class Shape
{
    public function __construct(protected string $color) {}

    abstract public function area(): float;     // abstract method — KHÔNG có thân

    public function describe(): string          // method thường — CÓ thân, dùng chung cho mọi con
    {
        return "Hình {$this->color} có diện tích " . $this->area();
    }
}

class Circle extends Shape
{
    public function __construct(string $color, private float $radius)
    {
        parent::__construct($color);
    }

    public function area(): float               // BẮT BUỘC implement abstract method
    {
        return 3.14 * $this->radius ** 2;
    }
}

$c = new Circle('đỏ', 5);
echo $c->describe();        // Hình đỏ có diện tích 78.5

// $s = new Shape('đỏ');   // ❌ Error: Cannot instantiate abstract class Shape
```

#### Khi nào dùng

- Khi một nhóm class có **quan hệ "is-a" chặt** và **chia sẻ code/state chung**: phần giống nhau viết
  sẵn một lần ở abstract class (như `describe()`), phần mỗi class khác nhau thì để abstract bắt con tự
  định nghĩa (như `area()`).
- Đây chính là tinh thần của **Template Method pattern**: lớp cha định nghĩa khung xử lý, để lại vài
  bước cho lớp con điền vào.

#### Phân biệt với Interface (rất hay hỏi)

| Tiêu chí | Abstract class | Interface |
|----------|----------------|-----------|
| Tạo object trực tiếp | ❌ Không | ❌ Không |
| Method có thân (concrete) | ✅ Có (lẫn abstract method) | ❌ Không (chỉ khai báo signature) |
| Property / state | ✅ Có | ❌ Không (chỉ `constant`) |
| Access modifier của method | `public` / `protected` / `private` | Chỉ `public` |
| Kế thừa/triển khai | `extends` **1** abstract class | `implements` **nhiều** interface |
| Mục đích | Chia sẻ **code + contract** chung | Định nghĩa **contract thuần** |

→ Chọn **abstract class** khi các lớp con cần **chia sẻ code/thuộc tính chung**; chọn **interface**
khi chỉ cần một **hợp đồng** mà nhiều class (kể cả không liên quan) đều có thể tuân theo, hoặc khi cần
một class "tuân theo nhiều hợp đồng" (đa kế thừa hành vi). Một class có thể **vừa `extends` một abstract
class vừa `implements` nhiều interface**.

### 🎯 Tóm tắt trả lời phỏng vấn
- **Abstract class** = lớp khai báo bằng `abstract`, **không `new` trực tiếp được**, dùng làm **lớp
  cha/khuôn mẫu** — hiện thực tính **Abstraction**.
- Có thể chứa **abstract method** (chỉ khai báo, không thân → **con bắt buộc override**) **lẫn method
  thường có thân, property, constructor**. Hễ có 1 abstract method thì class phải là abstract.
- **Mục đích:** gom **code/state dùng chung** ở lớp cha, để **phần khác nhau** cho lớp con tự định
  nghĩa (Template Method pattern).
- **Khác interface:** abstract class **có method có thân + property** và chỉ `extends` được **một**;
  interface là **contract thuần** (chỉ signature + constant, `public`) nhưng `implements` được
  **nhiều**. Dùng abstract khi cần chia sẻ code chung; dùng interface khi chỉ cần hợp đồng.

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Interface trong PHP là gì?

### 📖 Giới thiệu kiến thức

**Interface** là một **"hợp đồng" (contract) thuần**: nó khai báo một class **phải có những method
nào** (tên, tham số, kiểu trả về) nhưng **không cài đặt** chúng. Class muốn dùng interface phải
`implements` và **định nghĩa đầy đủ** mọi method mà interface yêu cầu. Đây là cách hiện thực mạnh nhất
cho tính **Abstraction** và **Polymorphism** của OOP.

#### Đặc điểm cốt lõi

- Khai báo bằng từ khóa `interface`; class "ký hợp đồng" bằng `implements`.
- **Mọi method đều ngầm định `public` và không có thân** (chỉ signature). Không được để `private`/`protected`.
- **Không có property/state**, chỉ được khai báo **constant**.
- **Một class `implements` được NHIỀU interface** (ngăn cách bằng dấu phẩy) → đây là cách PHP cho phép
  một class tuân theo nhiều hợp đồng, bù cho việc **PHP không hỗ trợ đa kế thừa class**.
- Một interface có thể `extends` (kế thừa) **nhiều** interface khác.
- **Không thể `new`** một interface.

```php
interface PaymentGateway
{
    public function pay(float $amount): bool;
    public function refund(string $transactionId): bool;
}

class StripeGateway implements PaymentGateway
{
    public function pay(float $amount): bool { /* gọi API Stripe */ return true; }
    public function refund(string $transactionId): bool { return true; }
}

class PaypalGateway implements PaymentGateway
{
    public function pay(float $amount): bool { /* gọi API PayPal */ return true; }
    public function refund(string $transactionId): bool { return true; }
}
```

#### Implements nhiều interface

```php
interface Swimmer { public function swim(): void; }
interface Flyer   { public function fly(): void; }

class Duck implements Swimmer, Flyer
{
    public function swim(): void { /* ... */ }
    public function fly(): void  { /* ... */ }
}
```

#### Vì sao quan trọng — "lập trình hướng giao diện"

Nguyên tắc **"program to an interface, not an implementation"**: code nên phụ thuộc vào **interface
(abstraction)** thay vì class cụ thể. Nhờ vậy:

- **Dễ thay thế implementation** mà không sửa nơi gọi — đúng tinh thần **Dependency Inversion (DIP)**
  trong SOLID.
- **Polymorphism**: một hàm nhận `PaymentGateway` chạy được với *mọi* implementation
  (`StripeGateway`, `PaypalGateway`...).
- **Dễ test**: dễ tạo **mock/fake** tuân theo interface để thay cho dịch vụ thật.

**Ví dụ cụ thể** — hàm `checkout()` chỉ phụ thuộc vào interface `PaymentGateway`, nên cùng một đoạn
code chạy được với mọi cổng thanh toán, và đặc biệt là **thay được bằng fake khi test**:

```php
// Hàm nghiệp vụ chỉ "biết" interface, không biết class cụ thể nào
class CheckoutService
{
    public function checkout(
        PaymentGateway $paymentGateway,
        float $amount
    ): bool {
        return $paymentGateway->pay($amount);
    }
}

// (1) Polymorphism + (2) Dễ thay thế: chỉ đổi object truyền vào, KHÔNG sửa checkout()
// CheckoutService không cần biết đối tượng cụ thể là Stripe hay PayPal. Nó chỉ cần đối tượng tuân theo PaymentGateway.
// Đây là một ứng dụng quan trọng của đa hình và dependency injection.
$service = new CheckoutService();

$service->checkout(new StripeGateway(), 100);   // chạy qua Stripe
$service->checkout(new PaypalGateway(), 100);   // đổi sang PayPal — nơi gọi y nguyên

// (3) Dễ test: viết một fake tuân theo interface, không gọi API thật, không mất tiền thật
class FakePaymentGateway implements PaymentGateway
{
    public bool $paid = false;

    public function pay(float $amount): bool
    {
        $this->paid = true;   // chỉ ghi nhận, không gọi mạng
        return true;
    }

    public function refund(string $transactionId): bool { return true; }
}

// Trong unit test:
$fake = new FakePaymentGateway();
checkout($fake, 100);
assert($fake->paid === true);   // kiểm tra checkout() có gọi pay() hay không
```

→ Vì `checkout()` nhận **kiểu interface**, lúc chạy thật ta truyền `StripeGateway`, còn lúc test ta
truyền `FakePaymentGateway` — **không phải sửa một dòng nào** trong `checkout()`. Đây chính là lợi ích
"dễ thay thế + dễ test" mà interface mang lại.

#### So với abstract class (xem thêm bảng ở câu 4)

Điểm khác cốt lõi: **interface là hợp đồng thuần** (không code, không state, `implements` được
**nhiều**); **abstract class** thì **chia sẻ được code + property** nhưng chỉ `extends` được **một**.
Một class có thể đồng thời **`extends` một abstract class và `implements` nhiều interface**.

> Quy tắc chọn nhanh: cần **chia sẻ code/state chung** giữa các lớp con họ hàng → **abstract class**;
> chỉ cần **một hợp đồng** mà nhiều class (kể cả không liên quan) cùng tuân theo, hoặc cần "đa kế
> thừa" hành vi → **interface**.

### 🎯 Tóm tắt trả lời phỏng vấn
- **Interface** = **hợp đồng thuần**: khai báo class phải có method nào (chỉ signature, không thân),
  class dùng `implements` và phải **định nghĩa đầy đủ** các method đó.
- **Đặc điểm:** mọi method **`public`, không thân**; **không có property** (chỉ `constant`); **không
  `new` được**; **`implements` được nhiều interface** (bù cho việc PHP không đa kế thừa class).
- **Mục đích:** "lập trình hướng giao diện" — code phụ thuộc abstraction nên **dễ thay thế
  implementation** (DIP), tận dụng **polymorphism**, và **dễ mock khi test**.
- **Khác abstract class:** interface không có code/state và `implements` được nhiều; abstract class
  chia sẻ code + property nhưng chỉ `extends` một. Một class có thể vừa `extends` abstract vừa
  `implements` nhiều interface.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Trait trong PHP là gì?

### 📖 Giới thiệu kiến thức

**Trait** là một cơ chế **tái sử dụng code theo chiều ngang (horizontal reuse)** trong PHP: nó là một
"gói" gồm **method (và property)** có sẵn thân, được **"dán" (copy) vào nhiều class không liên quan
huyết thống** bằng từ khóa `use`. Trait sinh ra để giải quyết hạn chế **PHP không hỗ trợ đa kế thừa
class** — một class chỉ `extends` được **một** cha, nhưng có thể `use` **nhiều** trait.

#### Vấn đề mà trait giải quyết

Giả sử cả `Post` và `Video` (không cùng class cha) đều cần khả năng "ghi log" và "đánh dấu thời gian".
Nếu nhét vào class cha chung thì gượng ép (chúng không thật sự "is-a" gì chung); copy-paste code thì vi
phạm DRY. Trait cho phép **viết một lần, dùng lại ở nhiều class rời rạc**:

```php
trait Loggable
{
    public function log(string $message): void
    {
        echo '[' . static::class . "] $message\n";
    }
}

trait Timestampable
{
    public ?string $createdAt = null;

    public function touch(): void
    {
        $this->createdAt = date('Y-m-d H:i:s');
    }
}

class Post
{
    use Loggable, Timestampable;   // dùng NHIỀU trait cùng lúc
}

class Video
{
    use Loggable;
}

$p = new Post();
$p->touch();
$p->log('created');     // [Post] created
```

#### Đặc điểm cốt lõi

- **Không thể khởi tạo trực tiếp** (`new Trait()` ❌) — trait chỉ để "trộn" vào class.
- Chứa được **method** (kể cả `static`, `abstract`) **và property** — khác interface (chỉ signature).
- Method của trait được **copy vào class** như thể viết trực tiếp trong class đó; vì vậy bên trong trait
  vẫn dùng được `$this`, `static::`, truy cập property/method của class dùng nó.
- Một class `use` được **nhiều trait**; một trait cũng có thể `use` trait khác.

#### Xử lý xung đột tên (conflict)

Khi hai trait có **method trùng tên**, PHP báo lỗi fatal; ta giải quyết bằng `insteadof` (chọn cái
nào) và `as` (đổi tên / đổi visibility):

```php
trait A { public function hello() { echo 'A'; } }
trait B { public function hello() { echo 'B'; } }

class C
{
    use A, B {
        A::hello insteadof B;   // dùng hello() của A
        B::hello as helloB;     // giữ bản của B dưới tên khác
    }
}
```

#### Thứ tự ưu tiên (precedence)

Khi cùng một method tồn tại ở nhiều nơi, PHP ưu tiên theo: **method của class hiện tại** > **method từ
trait** > **method kế thừa từ class cha**. Tức class tự định nghĩa sẽ "đè" method của trait, còn trait
lại "đè" method từ cha.

#### Trait vs Interface vs Abstract class (rất hay hỏi)

| Tiêu chí | Trait | Interface | Abstract class |
|----------|-------|-----------|----------------|
| Mục đích | **Tái sử dụng code** (chia sẻ implementation) | Định nghĩa **hợp đồng** | Khung cha + chia sẻ code |
| Có thân method | ✅ Có | ❌ Không | ✅ Có (lẫn abstract) |
| Có property/state | ✅ Có | ❌ (chỉ constant) | ✅ Có |
| Quan hệ với class | "**has/uses**" (trộn vào) | "**can-do**" (ký hợp đồng) | "**is-a**" (kế thừa) |
| Số lượng dùng được | `use` **nhiều** | `implements` **nhiều** | `extends` **một** |
| Khởi tạo trực tiếp | ❌ | ❌ | ❌ |

> Cách phân biệt nhanh: **interface** = "class này *làm được gì*" (contract, không code); **trait** =
> "*dùng lại đoạn code này*" (implementation, không contract); **abstract class** = "class con *là một
> loại* của cha + dùng chung code". Thực tế hay đi cặp: **interface khai báo hợp đồng, trait cung cấp
> sẵn phần cài đặt** cho hợp đồng đó.

#### Lưu ý / nhược điểm

- Trait làm **mờ nguồn gốc method** (đọc class không thấy ngay method đến từ đâu) → lạm dụng gây khó
  đọc, khó debug.
- Trait **không phải kiểu (type)**: không thể type-hint `function f(Loggable $x)` nếu `Loggable` là
  trait. Cần kiểu thì dùng **interface** (thường kết hợp: interface để type-hint + trait để cài đặt).
- Dễ bị xung đột property/method khi trộn nhiều trait → cân nhắc **composition** nếu quan hệ phức tạp.

### 🎯 Tóm tắt trả lời phỏng vấn
- **Trait** = cơ chế **tái sử dụng code theo chiều ngang**: một gói method/property có sẵn thân, được
  `use` "trộn" vào **nhiều class không cùng cha** — giải quyết việc **PHP không đa kế thừa class**.
- **Khác interface:** trait **có code + state** (interface chỉ signature); **khác abstract class:** một
  class `use` được **nhiều trait** nhưng chỉ `extends` **một** cha. Trait **không `new` được** và
  **không phải một kiểu** (không type-hint được).
- Method trait được **copy vào class**, vẫn dùng `$this`/`static::`. Ưu tiên: **class > trait > cha**.
  Xung đột tên xử lý bằng **`insteadof`** và **`as`**.
- **Mẹo chọn:** cần *hợp đồng* → interface; cần *dùng lại code* → trait; thường **kết hợp interface +
  trait**. Tránh lạm dụng vì trait làm **mờ nguồn gốc method**, khó debug.

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. `static` trong PHP là gì?

### 📖 Giới thiệu kiến thức

Từ khóa `static` trong OOP đánh dấu một thành viên (property/method) **thuộc về chính class**, **không
thuộc về từng object (instance)**. Nghĩa là **mọi instance dùng chung một bản duy nhất**, và có thể
truy cập **mà không cần `new`**, gọi trực tiếp qua `TênClass::`.

#### Static property & static method

```php
class Counter
{
    public static int $count = 0;        // thuộc về class — dùng chung cho mọi instance

    public static function increment(): void
    {
        self::$count++;                  // KHÔNG có $this trong static method
    }
}

Counter::increment();
Counter::increment();
echo Counter::$count;   // 2  — truy cập qua class, không cần tạo object
```

Đặc điểm quan trọng:
- **Không gắn với instance** nên trong static method **không dùng được `$this`**.
- Truy cập bằng `::` (toán tử phạm vi): từ ngoài là `Counter::$count`, từ trong class là `self::` hoặc
  `static::`.
- **Dùng cho:** hàm tiện ích (helper/utility), biến đếm/đăng ký dùng chung, factory method,
  singleton, hằng cấu hình.

#### `self::` vs `static::` — Late Static Binding (điểm nâng cao hay hỏi)

- `self::` luôn trỏ tới **class nơi viết câu lệnh đó** (cố định lúc biên dịch).
- `static::` trỏ tới **class thực sự được gọi lúc runtime** (gọi là **Late Static Binding**), nên nó
  "nhìn thấy" class con.

```php
class Model
{
    public static function create(): static
    {
        return new static();   // static = class được gọi thật, không phải Model
    }
}
class User extends Model {}

$u = User::create();   // trả về object User (nhờ late static binding), không phải Model
```

> Lưu ý phân biệt với **biến static trong hàm** (`static $x = 0;` bên trong một function) — đó là biến
> **giữ giá trị qua các lần gọi hàm**, khác với static member của class.

### 🎯 Tóm tắt trả lời phỏng vấn
- `static` đánh dấu property/method **thuộc về class, không thuộc instance** → **mọi object dùng chung
  một bản**, gọi trực tiếp qua `Class::` mà **không cần `new`**.
- Trong static method **không có `$this`** (vì không gắn với object cụ thể).
- **Dùng cho:** helper/utility, biến đếm dùng chung, factory method, singleton.
- Nâng cao: **`self::`** trỏ class nơi viết code (cố định); **`static::`** trỏ class gọi lúc runtime
  (**Late Static Binding**) → `new static()` tạo đúng object của class con.
- Đừng nhầm với **biến `static` trong hàm** (giữ giá trị qua các lần gọi) — khác với static member.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. Phân biệt `$this` và `self` trong PHP

### 📖 Giới thiệu kiến thức

Cả `$this` và `self` đều dùng để tham chiếu *bên trong* một class, nhưng trỏ tới **hai thứ khác nhau**:

- **`$this`** trỏ tới **object hiện tại (instance)** đang gọi method — dùng để truy cập thành viên
  **non-static** qua toán tử `->`.
- **`self`** trỏ tới **chính class nơi câu lệnh được viết** (không phải object) — dùng để truy cập
  thành viên **static**, **constant**, static method qua toán tử `::`.

```php
class User
{
    public const ROLE = 'user';
    public static int $count = 0;

    public function __construct(public string $name)
    {
        self::$count++;                 // self:: → static property của CLASS (đếm chung)
    }

    public function describe(): string
    {
        // $this-> → dữ liệu của object hiện tại; self:: → hằng của class
        return $this->name . ' - ' . self::ROLE;
    }
}

$u = new User('Cuom');
echo $u->describe();   // Cuom - user
echo User::$count;     // 1
```

#### Bảng so sánh

| Tiêu chí | `$this` | `self` |
|----------|---------|--------|
| Trỏ tới | **Object hiện tại** (instance) | **Class** nơi viết code |
| Toán tử | `->` | `::` |
| Truy cập | Thành viên **non-static** (property/method của object) | Thành viên **static**, **constant** |
| Có trong static method? | ❌ Không (không có object) | ✅ Có |
| Khi nào "chốt" mục tiêu | **Runtime**, theo object thật → có **đa hình** | **Compile-time**, cố định class định nghĩa |

> **Lưu ý:** `self` còn được đem so sánh với `static` (Late Static Binding) — `self` cố định ở class
> định nghĩa, còn `static` theo class gọi lúc runtime. Đây là một chủ đề riêng, xem chi tiết ở
> [câu 9](#q9).

### 🎯 Tóm tắt trả lời phỏng vấn
- **`$this`** = **object hiện tại** (instance), truy cập thành viên **non-static** qua `->`. Không tồn
  tại trong static method.
- **`self`** = **class nơi viết code**, truy cập thành viên **static / constant** qua `::`. Dùng được
  trong static method.
- Khác cốt lõi: **`$this` runtime theo object thật** (đa hình); **`self` cố định tại compile-time** ở
  class định nghĩa.
- Phân biệt thêm **`self` vs `static`** (Late Static Binding): xem [câu 9](#q9).

[↑ Mục lục](#mục-lục)

<a id="q9"></a>
## 9. Phân biệt `self` và `static` (Late Static Binding)

### 📖 Giới thiệu kiến thức

Cả `self::` và `static::` đều dùng `::` để truy cập thành viên static/method ở **cấp class**, nhưng
khác nhau ở chỗ **chúng "trỏ" tới class nào khi có kế thừa**:

- **`self::`** được **resolve lúc biên dịch (compile-time)**, **luôn** trỏ tới **class nơi câu lệnh
  được viết** — không bao giờ đổi, kể cả khi method được kế thừa và gọi từ class con.
- **`static::`** được **resolve lúc chạy (runtime)** theo **class thực sự được gọi** — đây gọi là
  **Late Static Binding (LSB)**, có từ **PHP 5.3**. Nó "forward" được tới class con.

#### Ví dụ trực quan: `self::class` vs `static::class`

```php
class Base
{
    public static function whoSelf(): string   { return self::class; }
    public static function whoStatic(): string { return static::class; }
}
class Child extends Base {}

echo Child::whoSelf();    // Base   ← self cố định ở class ĐỊNH NGHĨA (Base)
echo Child::whoStatic();  // Child  ← static theo class ĐƯỢC GỌI (Child)
```

#### Vì sao Late Static Binding ra đời

Trước PHP 5.3 chỉ có `self::`, nên một method ở class cha **không thể** tham chiếu tới class con đã
override. LSB (`static::`) sinh ra để giải quyết đúng việc đó. Ứng dụng kinh điển là **`new static()`**
— factory method ở class cha nhưng tạo ra đúng object của class con (nền tảng của Active Record như
Eloquent):

```php
class Model
{
    public static function make(): static
    {
        return new static();   // tạo instance của class GỌI, không phải Model
    }

    public function type(): string { return 'model'; }
}

class User extends Model
{
    public function type(): string { return 'user'; }
}

echo Model::make()->type();   // model
echo User::make()->type();    // user   ← nhờ new static() (LSB)
// nếu viết `new self()` trong make() thì User::make() vẫn trả về Model → sai
```

#### Bảng so sánh

| Tiêu chí | `self::` | `static::` |
|----------|----------|------------|
| Thời điểm resolve | **Compile-time** | **Runtime** (Late Static Binding) |
| Trỏ tới class | **Nơi định nghĩa** method | **Nơi gọi** (có thể là class con) |
| "Thấy" class con khi kế thừa? | ❌ Không | ✅ Có |
| Có từ phiên bản | Luôn có | PHP **5.3+** |
| Dùng khi | Muốn **cố định** ở class hiện tại | Muốn **forward** tới class con (factory, fluent) |

### 🎯 Tóm tắt trả lời phỏng vấn
- Cả hai truy cập thành viên cấp class qua `::`, khác ở chỗ **trỏ tới class nào khi kế thừa**.
- **`self::`** resolve **compile-time**, luôn là **class nơi viết code** — không thấy class con.
- **`static::`** resolve **runtime** theo **class được gọi** (**Late Static Binding**, PHP 5.3+) — thấy
  được class con.
- Ứng dụng quan trọng nhất: **`new static()`** tạo đúng object của class con (factory method, Active
  Record). Nếu dùng `new self()` thì luôn ra class cha → sai ý đồ.
- Chốt: cần **cố định** class hiện tại → `self::`; cần **forward** theo class con → `static::`.

[↑ Mục lục](#mục-lục)

<a id="q10"></a>
## 10. `final` trong PHP là gì?

### 📖 Giới thiệu kiến thức

Từ khóa `final` dùng để **ngăn việc mở rộng hoặc ghi đè** — nó "khóa" một thành phần lại, không cho
class con thay đổi. Có 3 chỗ dùng:

- **`final class`** — **không class nào được `extends`** class này.
- **`final method`** — class con **không được override (ghi đè)** method này.
- **`final` constant** (từ PHP 8.1) — hằng không bị class con ghi đè.

```php
final class Money       // ❌ không ai extends Money được
{
    final public function format(): string   // (trong class thường) con không override được
    {
        return number_format(100);
    }
}

class Base
{
    final public function id(): int { return 1; }
}

class Child extends Base
{
    // public function id(): int { return 2; }  // ❌ Error: Cannot override final method
}
```

#### Mục đích — vì sao cần `final`

- **Bảo vệ hành vi cốt lõi**: đảm bảo một logic quan trọng (vd tính tiền, kiểm tra bảo mật) **không bị
  lớp con sửa đổi** làm sai lệch invariant.
- **Thể hiện ý đồ thiết kế**: báo rõ "class/method này **không thiết kế để kế thừa**", tránh người
  khác lạm dụng inheritance theo cách ngoài dự kiến.
- Khuyến khích **composition over inheritance**: thay vì kế thừa sâu nhiều tầng (dễ vỡ), `final` đẩy ta
  sang hướng kết hợp object — dễ bảo trì hơn.

#### Quan hệ với `abstract`

`final` và `abstract` **đối lập nhau**: `abstract` **bắt buộc** phải kế thừa/triển khai, còn `final`
**cấm** kế thừa/override → **không thể** khai báo một class/method vừa `abstract` vừa `final`.

### 🎯 Tóm tắt trả lời phỏng vấn
- `final` dùng để **chặn mở rộng/ghi đè**: **`final class`** không cho `extends`; **`final method`**
  không cho class con override; (PHP 8.1) **`final` constant** không bị ghi đè.
- **Mục đích:** **bảo vệ logic cốt lõi** khỏi bị lớp con sửa, **thể hiện thiết kế** "không dành để kế
  thừa", và khuyến khích **composition over inheritance**.
- **Đối lập với `abstract`**: abstract *bắt buộc* kế thừa/override, final *cấm* — nên không thể vừa
  `abstract` vừa `final`.

[↑ Mục lục](#mục-lục)

<a id="q11"></a>
## 11. `readonly` trong PHP là gì?

### 📖 Giới thiệu kiến thức

`readonly` (giới thiệu từ **PHP 8.1**) là một **modifier** cho **thuộc tính (property)** của class:
thuộc tính `readonly` chỉ được **gán giá trị đúng một lần** (thường trong constructor), sau đó
**không thể thay đổi** — mọi nỗ lực ghi tiếp sẽ ném `Error`. Đây là công cụ chính để tạo **immutable
object** (object bất biến) trong PHP một cách gọn gàng, thay vì tự viết getter + để property `private`
mà không có setter.

```php
class User
{
    public readonly string $name;

    public function __construct(string $name)
    {
        $this->name = $name;   // ✅ gán lần đầu trong constructor — hợp lệ
    }
}

$u = new User('Cuom');
echo $u->name;     // Cuom
$u->name = 'X';    // ❌ Error: Cannot modify readonly property User::$name
```

#### Các quy tắc cần nhớ

- **Bắt buộc là typed property** — phải khai báo kiểu (`string`, `int`, `User`...). Không dùng được
  với property không khai báo kiểu.
- **Không có giá trị mặc định** — không thể `public readonly int $x = 0;`. Giá trị phải được gán lúc
  runtime (trong constructor/method).
- **Chỉ gán được từ bên trong scope class** và **đúng một lần**. Sau khi đã khởi tạo, không sửa được
  kể cả từ bên trong class.
- **Không áp dụng cho `static` property.**

#### Kết hợp constructor property promotion (rất phổ biến)

```php
class Point
{
    public function __construct(
        public readonly int $x,
        public readonly int $y,
    ) {}
}
```

#### `readonly` là "nông" (shallow) — bẫy hay gặp

`readonly` chỉ ngăn **gán lại chính property đó**. Nếu property giữ một **object**, ta vẫn **sửa được
thuộc tính bên trong** object đó (vì không gán lại property, chỉ thay đổi state của object mà nó trỏ tới):

```php
class Box { public int $value = 0; }

class Container
{
    public function __construct(public readonly Box $box) {}
}

$c = new Container(new Box());
$c->box->value = 99;   // ✅ ĐƯỢC — sửa state bên trong object, không gán lại $box
$c->box = new Box();   // ❌ Error — gán lại property readonly
```

> Ngược lại, với property kiểu **array**, sửa một phần tử bị tính là **ghi vào property** → vẫn ném
> `Error` (array là value type, không phải handle như object).

#### `readonly class` (PHP 8.2)

PHP 8.2 cho phép gắn `readonly` cho **cả class** — khi đó **mọi property** của class tự động là
`readonly`, khỏi lặp lại từ khóa:

```php
readonly class Money
{
    public function __construct(
        public int $amount,
        public string $currency,
    ) {}
}
```

#### Muốn "thay đổi" object readonly thì làm sao?

Vì không sửa trực tiếp được, pattern chuẩn là tạo **bản sao đã đổi giá trị**. PHP 8.5 hỗ trợ
`clone()` đổi thuộc tính ngay khi clone (xem câu PHP version):

```php
$editor = clone($admin, ['role' => 'editor']); // tạo object mới, giữ nguyên $admin gốc
```

### 🎯 Tóm tắt trả lời phỏng vấn
- `readonly` (PHP **8.1**) là modifier cho **property**: chỉ gán **một lần** (thường trong
  constructor), sau đó **bất biến** — ghi tiếp sẽ ném `Error`. Dùng để tạo **immutable object** gọn gàng.
- **Quy tắc:** phải là **typed property**, **không có default value**, chỉ gán **từ trong scope class
  đúng một lần**, không dùng cho `static`.
- **Là "nông" (shallow):** ngăn gán lại property, nhưng nếu property là **object** thì vẫn **sửa được
  state bên trong** object đó; với **array** thì sửa phần tử cũng bị chặn.
- **PHP 8.2:** `readonly class` → **mọi property** tự động readonly.
- Muốn "đổi giá trị" thì tạo **bản sao mới** (PHP 8.5: `clone($obj, [...])`). Hay dùng cho **DTO,
  Value Object**, giảm bug do mutation ngoài ý muốn.

[↑ Mục lục](#mục-lục)

<a id="q12"></a>
## 12. Phân biệt `include`, `include_once`, `require`, `require_once`

### 📖 Giới thiệu kiến thức

Đây là 4 **language construct** (không phải hàm) dùng để **nạp nội dung một file PHP vào file
hiện tại tại thời điểm chạy**. Về bản chất, PHP đọc file được chỉ định, chèn code của nó vào đúng
vị trí câu lệnh, và file được nạp **kế thừa scope biến** tại dòng gọi.

Chúng khác nhau ở **2 trục**:

#### Trục 1 — Mức độ xử lý lỗi khi không tìm thấy file: `include` vs `require`

- **`include`**: nếu không tìm thấy file → phát **Warning** (`E_WARNING`), nhưng **script vẫn chạy
  tiếp** các dòng sau.
- **`require`**: nếu không tìm thấy file → phát **Fatal error** (`E_ERROR` / `E_COMPILE_ERROR`) và
  **dừng toàn bộ script** ngay lập tức.

→ Dùng `require` cho file **bắt buộc phải có** (config, bootstrap, file khai báo class cốt lõi);
dùng `include` cho phần **tùy chọn** (ví dụ một banner, một template phụ — thiếu thì trang vẫn nên hiển thị).

#### Trục 2 — Có nạp lại nếu file đã được nạp trước đó không: bản thường vs `_once`

- **Bản thường** (`include` / `require`): mỗi lần gọi là **nạp lại**, kể cả file đã nạp rồi.
- **Bản `_once`** (`include_once` / `require_once`): PHP ghi nhớ (theo **đường dẫn thực** của file)
  những file đã nạp; nếu file **đã được nạp rồi thì bỏ qua**, không nạp lại.

→ `_once` tránh lỗi **"Cannot redeclare function/class"** khi một file khai báo hàm/class bị nạp
nhiều lần (ví dụ qua nhiều nhánh `require`).

#### Bảng so sánh

| Construct | Không tìm thấy file | Nếu file đã nạp trước đó |
|-----------|---------------------|--------------------------|
| `include` | **Warning** → chạy tiếp | Nạp lại |
| `include_once` | **Warning** → chạy tiếp | **Bỏ qua** (chỉ 1 lần) |
| `require` | **Fatal** → dừng script | Nạp lại |
| `require_once` | **Fatal** → dừng script | **Bỏ qua** (chỉ 1 lần) |

#### Ví dụ

```php
// config.php
return ['db' => 'mysql', 'debug' => true];
```

```php
// index.php
$config = require __DIR__ . '/config.php'; // file trả về giá trị → require nhận được array

require_once __DIR__ . '/helpers.php'; // file khai báo function → dùng _once tránh redeclare

include __DIR__ . '/banner.php'; // phần phụ, thiếu cũng không sao → include
```

> **Lưu ý:** vì là language construct, có thể viết không ngoặc: `require 'file.php';`. Cả 4 đều có
> thể trả về giá trị nếu file được nạp dùng `return` (như ví dụ `config.php`).

#### Thực tế hiện đại

Trong dự án hiện đại dùng **Composer**, ta hầu như **không viết `require_once` thủ công** cho từng
class nữa. Composer sinh sẵn `vendor/autoload.php` (ta chỉ `require` đúng 1 file này), sau đó class
được nạp **tự động theo chuẩn PSR-4** thông qua autoloader — vừa gọn, vừa tránh nạp file không cần
thiết. Cơ chế autoload này được nói kỹ ở [câu 13](#q13).

### 🎯 Tóm tắt trả lời phỏng vấn
- Cả 4 đều **nạp nội dung một file PHP** vào file hiện tại lúc runtime; file được nạp **kế thừa
  scope biến** của dòng gọi.
- Khác nhau ở **2 trục**:
  - **`include` vs `require`** — xử lý lỗi: `include` thiếu file chỉ **Warning** rồi chạy tiếp;
    `require` thiếu file là **Fatal error** và **dừng script**.
  - **bản thường vs `_once`** — `_once` chỉ nạp **đúng một lần**, nếu đã nạp thì bỏ qua → tránh lỗi
    **redeclare** class/function.
- Quy tắc chọn: **`require`/`require_once`** cho file **bắt buộc** (config, class cốt lõi),
  **`include`** cho phần **tùy chọn**; thêm hậu tố **`_once`** cho file khai báo hàm/class.
- Thực tế: với **Composer autoload (PSR-4)**, ta chỉ `require 'vendor/autoload.php'` một lần và để
  autoloader nạp class, gần như không cần `require_once` thủ công.

[↑ Mục lục](#mục-lục)

<a id="q13"></a>
## 13. Hàm `__autoload()` và cơ chế autoload trong PHP

### 📖 Giới thiệu kiến thức

Ở [câu 12](#q12) ta thấy: muốn dùng một class ở file khác thì phải `require`/`require_once` đúng file
chứa nó. Khi dự án có hàng trăm class, việc viết `require_once` thủ công cho từng file vừa dài vừa dễ
sót. **Autoload** sinh ra để giải quyết: **PHP tự động nạp file chứa class ngay khi class được dùng
lần đầu** — lập trình viên không cần `require` thủ công nữa.

#### 1. Hàm `__autoload()` — cơ chế cũ (đã bị loại bỏ)

`__autoload()` là cơ chế autoload **kiểu cũ** của PHP. Hàm này được PHP **tự động gọi** khi chương
trình sử dụng một class **chưa được khai báo** hoặc **chưa được `include`/`require`**.

```php
function __autoload($className)
{
    require_once 'classes/' . $className . '.php';
}

$user = new User();
```

Khi PHP gặp `new User();` nhưng chưa tìm thấy class `User`, PHP sẽ gọi `__autoload('User')`, sau đó
file `classes/User.php` được nạp. File đó có thể chứa:

```php
<?php

class User
{
    public function getName(): string
    {
        return 'Cuom';
    }
}
```

**Vấn đề của `__autoload()`:**

- Chỉ khai báo được **một** hàm autoload.
- Khó kết hợp nhiều thư viện.
- Dễ xảy ra **xung đột** tên hàm.
- Không phù hợp với ứng dụng hiện đại dùng **namespace**.
- Đã bị **deprecated từ PHP 7.2**.
- Đã bị **loại bỏ trong PHP 8.0**.

> ⚠️ Vì vậy, **không nên** sử dụng `__autoload()` trong code mới — nhắc tới nó chủ yếu để hiểu lịch sử.

#### 2. Cơ chế autoload trong PHP

Autoload là cơ chế **tự động nạp file** chứa class, interface, trait hoặc enum khi chúng được sử dụng
lần đầu. Thay vì viết thủ công:

```php
require_once 'User.php';
require_once 'Product.php';
require_once 'Order.php';
```

ta đăng ký một **autoloader**:

```php
spl_autoload_register(function (string $className): void {
    require_once 'classes/' . $className . '.php';
});
```

Khi chạy `$user = new User();`, PHP sẽ:

1. Kiểm tra class `User` đã tồn tại chưa.
2. Nếu chưa, gọi lần lượt các hàm autoload đã đăng ký.
3. Autoloader tìm và nạp file chứa class.
4. PHP kiểm tra lại class `User`.
5. Nếu vẫn không tồn tại, PHP phát sinh lỗi.

#### 3. Sử dụng `spl_autoload_register()`

Cú pháp cơ bản:

```php
spl_autoload_register(function (string $className): void {
    $file = __DIR__ . '/classes/' . $className . '.php';

    if (file_exists($file)) {
        require_once $file;
    }
});
```

Cấu trúc thư mục:

```text
project/
├── index.php
└── classes/
    ├── User.php
    └── Product.php
```

File `classes/User.php`:

```php
<?php

class User
{
    public function sayHello(): string
    {
        return 'Hello';
    }
}
```

File `index.php`:

```php
<?php

spl_autoload_register(function (string $className): void {
    $file = __DIR__ . '/classes/' . $className . '.php';

    if (is_file($file)) {
        require_once $file;
    }
});

$user = new User();

echo $user->sayHello();   // Hello
```

#### 4. Có thể đăng ký nhiều autoloader

Khác với `__autoload()`, `spl_autoload_register()` cho phép **đăng ký nhiều autoloader**. PHP sẽ gọi
các autoloader **theo thứ tự đăng ký** cho đến khi class được tìm thấy.

```php
spl_autoload_register(function (string $className): void {
    $file = __DIR__ . '/app/' . $className . '.php';

    if (is_file($file)) {
        require_once $file;
    }
});

spl_autoload_register(function (string $className): void {
    $file = __DIR__ . '/libraries/' . $className . '.php';

    if (is_file($file)) {
        require_once $file;
    }
});
```

#### 5. Autoload với namespace

Giả sử có class `App\Services\MailService` đặt tại `src/Services/MailService.php`:

```php
<?php

namespace App\Services;

class MailService
{
}
```

Autoloader ánh xạ namespace → đường dẫn:

```php
spl_autoload_register(function (string $className): void {
    $prefix = 'App\\';
    $baseDirectory = __DIR__ . '/src/';

    if (!str_starts_with($className, $prefix)) {
        return;
    }

    $relativeClass = substr($className, strlen($prefix));

    $file = $baseDirectory
        . str_replace('\\', DIRECTORY_SEPARATOR, $relativeClass)
        . '.php';

    if (is_file($file)) {
        require_once $file;
    }
});
```

Khi dùng `use App\Services\MailService; $mail = new MailService();`, class `App\Services\MailService`
được chuyển thành đường dẫn `src/Services/MailService.php`. Đây là nguyên tắc **gần giống chuẩn PSR-4**.

#### 6. Autoload theo chuẩn PSR-4

**PSR-4** quy định cách **ánh xạ namespace thành cấu trúc thư mục**. Ví dụ:

```text
Namespace: App\Controllers\UserController
File:      src/Controllers/UserController.php
```

Với ánh xạ `App\ => src/`, phần còn lại `Controllers\UserController` được chuyển thành
`Controllers/UserController.php`. Do đó class đầy đủ `App\Controllers\UserController` nằm trong
`src/Controllers/UserController.php`.

#### 7. Composer autoload

Trong dự án PHP hiện đại, **nên dùng Composer** thay vì tự viết autoloader (xem thêm [câu 14](#q14)).

File `composer.json`:

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

Cấu trúc:

```text
project/
├── composer.json
├── vendor/
├── index.php
└── src/
    └── Services/
        └── MailService.php
```

File `src/Services/MailService.php`:

```php
<?php

namespace App\Services;

class MailService
{
    public function send(): string
    {
        return 'Mail sent';
    }
}
```

Sau khi cấu hình, chạy `composer dump-autoload` để Composer sinh lại bảng autoload. Trong `index.php`:

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use App\Services\MailService;

$mailService = new MailService();

echo $mailService->send();   // Mail sent
```

Composer sẽ tự động tìm và nạp file `src/Services/MailService.php`.

#### 8. Các kiểu autoload của Composer

- **PSR-4** — phổ biến nhất, ánh xạ namespace → thư mục:

  ```json
  { "autoload": { "psr-4": { "App\\": "src/" } } }
  ```

- **PSR-0** — chuẩn cũ, hiện ít sử dụng:

  ```json
  { "autoload": { "psr-0": { "Legacy\\": "src/" } } }
  ```

- **Classmap** — Composer quét thư mục và tạo danh sách class; phù hợp với code cũ không tuân thủ PSR-4:

  ```json
  { "autoload": { "classmap": ["app/Models", "app/Legacy"] } }
  ```

- **Files** — tự động `include` các file chứa **function/constant** (vd file helper):

  ```json
  { "autoload": { "files": ["app/helpers.php"] } }
  ```

  ```php
  <?php
  // app/helpers.php
  function formatPrice(float $price): string
  {
      return number_format($price) . ' VNĐ';
  }
  ```

  Sau khi chạy `composer dump-autoload`, hàm dùng trực tiếp được: `echo formatPrice(100000);`

#### 9. So sánh `__autoload()` và `spl_autoload_register()`

| Tiêu chí | `__autoload()` | `spl_autoload_register()` |
|----------|----------------|---------------------------|
| Số autoloader | Chỉ một | Có thể nhiều |
| Hỗ trợ PHP 8 | ❌ Không | ✅ Có |
| Khả năng mở rộng | Thấp | Tốt |
| Kết hợp thư viện | Khó | Dễ |
| Hỗ trợ namespace | Có thể nhưng bất tiện | Tốt |
| Composer sử dụng | ❌ Không | ✅ Có |
| Nên sử dụng | ❌ Không | ✅ Có |

#### 10. Những lưu ý khi viết autoloader

- **Không** dùng trực tiếp `$className` để tạo đường dẫn mà không kiểm soát (rủi ro path traversal):

  ```php
  require $className . '.php';   // ❌ nguy hiểm
  ```

  Nên **giới hạn theo namespace/thư mục** và kiểm tra file tồn tại trước khi nạp:

  ```php
  spl_autoload_register(function (string $className): void {
      $prefix = 'App\\';

      if (!str_starts_with($className, $prefix)) {
          return;
      }

      $relativeClass = substr($className, strlen($prefix));

      $file = __DIR__ . '/src/' . str_replace('\\', '/', $relativeClass) . '.php';

      if (is_file($file)) {
          require_once $file;
      }
  });
  ```

- **Không** ném lỗi ngay trong autoloader khi không tìm thấy file — vì có thể còn autoloader khác xử lý
  class đó; cứ `return` để PHP gọi autoloader tiếp theo:

  ```php
  if (!is_file($file)) {
      return;
  }
  ```

> **Kết luận:** `__autoload()` là cơ chế cũ và **không còn dùng được trên PHP 8 trở lên**. Trong PHP
> hiện đại: dùng **`spl_autoload_register()`** khi muốn tự xây autoloader; dùng **Composer + PSR-4**
> cho dự án thực tế và chỉ cần `require 'vendor/autoload.php'` **một lần** ở entry point. Đây cũng là
> chuỗi tiến hóa của cùng một ý tưởng: `__autoload()` → `spl_autoload_register()` → **Composer PSR-4**.

### 🎯 Tóm tắt trả lời phỏng vấn
- **Autoload** = cơ chế **PHP tự nạp file chứa class/interface/trait/enum lần đầu được dùng**, để khỏi
  `require` thủ công.
- **`__autoload($className)`** là hàm magic toàn cục đời cũ làm việc đó, nhưng **chỉ được có một** →
  thư viện đụng nhau, không hỗ trợ namespace tốt. Vì vậy nó **deprecated từ PHP 7.2 và bị XÓA từ PHP
  8.0** — không dùng nữa.
- Thay thế là **`spl_autoload_register()`**: **đăng ký được NHIỀU autoloader** (gọi theo thứ tự đăng
  ký), nhận callable bất kỳ, ánh xạ **namespace → đường dẫn file** → nền tảng cho chuẩn **PSR-4**.
- **Composer** là cách dùng thực tế: khai báo `psr-4` trong `composer.json`, chạy `composer
  dump-autoload`, rồi chỉ `require 'vendor/autoload.php'` **một lần** ở entry point. Composer còn hỗ trợ
  **PSR-0** (cũ), **classmap** (code không theo PSR-4), **files** (nạp helper/function).
- **Lưu ý khi tự viết autoloader:** giới hạn theo namespace + kiểm tra `is_file()` (tránh path
  traversal), và **không ném lỗi** khi không thấy file — cứ `return` để autoloader khác xử lý tiếp.

[↑ Mục lục](#mục-lục)

<a id="q14"></a>
## 14. Các cách cài thư viện vào dự án PHP thuần

### 📖 Giới thiệu kiến thức

"Thư viện" (library/package) là code có sẵn của bên thứ ba để tái sử dụng. Trong PHP có **2 loại
thư viện** khác bản chất, kéo theo cách cài khác nhau:

- **Thư viện viết bằng PHP** (PHP-land): code `.php` thuần, vd Guzzle, Carbon, PHPMailer.
- **PHP extension** (viết bằng C): nạp vào engine, vd `redis`, `imagick`, `gd`, `pdo_mysql`.

#### 1. Composer — cách chuẩn hiện đại (nên dùng) ⭐

**Composer** là **dependency manager** chính thức của PHP, lấy package từ kho **Packagist**. Đây là
cách được khuyên dùng cho mọi dự án PHP mới, kể cả "PHP thuần" không framework.

```bash
# cài 1 package — Composer tự thêm vào composer.json và tải về vendor/
composer require guzzlehttp/guzzle

# cài tất cả package đã khai báo (khi clone dự án về)
composer install
```

Sau khi cài, chỉ cần nạp **một** file autoload, mọi class dùng được ngay theo chuẩn **PSR-4** (không
phải `require` từng file):

```php
require __DIR__ . '/vendor/autoload.php';

use GuzzleHttp\Client;
$client = new Client();
```

Composer sinh ra:
- **`composer.json`** — khai báo dependency (commit vào git).
- **`composer.lock`** — khóa **chính xác version** đã cài → mọi máy cài giống nhau (commit vào git).
- **`vendor/`** — chứa code thư viện + autoloader (thường **gitignore**, cài lại bằng `composer install`).

Ưu điểm: tự quản lý **version & dependency lồng nhau**, autoload PSR-4, dễ cập nhật (`composer update`).

#### 2. Tải thủ công + `require`/`include` (cách cũ, trước Composer)

Tải file thư viện (từ GitHub, trang chủ...) về, đặt vào thư mục dự án rồi nạp thủ công:

```php
require_once __DIR__ . '/libs/PHPMailer/src/PHPMailer.php';
```

→ Cách này **không quản lý version**, không tự xử lý **dependency lồng nhau**, dễ xung đột và khó cập
nhật. Chỉ nên dùng cho thư viện rất nhỏ, lẻ, hoặc môi trường không cho cài Composer.

#### 3. Cài PHP extension (PECL / package manager hệ điều hành)

Với thư viện viết bằng **C** (extension), không dùng Composer mà cài ở tầng engine, rồi **bật trong
`php.ini`**:

```bash
# qua PECL
pecl install redis

# hoặc qua package manager của OS (thường gặp hơn)
sudo apt install php-redis      # Debian/Ubuntu
```

```ini
; bật trong php.ini
extension=redis
```

Kiểm tra đã bật chưa: `php -m` (liệt kê module) hoặc `phpinfo()`.

#### 4. PEAR (legacy — gần như không còn dùng)

**PEAR** là trình quản lý package **đời cũ** trước Composer (`pear install ...`). Ngày nay hầu như
chỉ gặp ở dự án rất cũ; dự án mới **không nên** dùng.

#### Bảng so sánh

| Cách | Dùng cho | Quản lý version/dependency | Khuyến nghị |
|------|----------|----------------------------|-------------|
| **Composer** | Thư viện PHP-land | ✅ Có (composer.lock) + autoload PSR-4 | ⭐ Chuẩn hiện đại |
| Tải thủ công + `require` | Thư viện PHP nhỏ, lẻ | ❌ Không | Chỉ khi bất đắc dĩ |
| PECL / `apt` + `php.ini` | PHP **extension** (C) | Theo OS/PECL | Bắt buộc cho extension |
| PEAR | (legacy) | Cũ | ❌ Tránh |

### 🎯 Tóm tắt trả lời phỏng vấn
- **Cách chuẩn hiện đại là Composer** — dependency manager lấy package từ **Packagist**:
  `composer require vendor/package`, rồi chỉ cần `require 'vendor/autoload.php'` là dùng được nhờ
  **autoload PSR-4**. Nó quản lý **version & dependency lồng nhau** qua `composer.json` + khóa version
  bằng `composer.lock`.
- **Tải thủ công + `require_once`** là cách cũ trước Composer: không quản lý version/dependency, dễ
  xung đột — chỉ dùng cho thư viện nhỏ lẻ hoặc khi không cài được Composer.
- **PHP extension** (viết bằng C như `redis`, `gd`, `imagick`) **không** cài qua Composer mà qua
  **PECL** hoặc package manager hệ điều hành (`apt install php-redis`), rồi **bật trong `php.ini`**
  (`extension=...`).
- **PEAR** là trình quản lý đời cũ, ngày nay **tránh dùng**.
- Một câu chốt: *"Dự án PHP thuần hiện nay mình vẫn dùng Composer để cài thư viện, chỉ riêng extension C thì cài qua PECL/OS và bật trong php.ini."*

[↑ Mục lục](#mục-lục)

<a id="q15"></a>
## 15. Các cách chạy một file PHP (hoặc một dự án PHP)

### 📖 Giới thiệu kiến thức

PHP là ngôn ngữ **thông dịch** (interpreted): không biên dịch ra file chạy độc lập, mà cần **PHP
engine** đọc và thực thi mã nguồn. Engine giao tiếp với môi trường qua một lớp gọi là **SAPI**
(Server API). Mỗi cách chạy PHP tương ứng một SAPI khác nhau — đây chính là gốc rễ phân biệt các cách
dưới đây. Chia làm 2 nhóm lớn: chạy bằng **dòng lệnh (CLI)** và chạy qua **web server**.

#### A. Chạy qua dòng lệnh — CLI (SAPI = `cli`)

**1. Chạy trực tiếp một file**

```bash
php script.php
```

Engine thực thi file rồi in kết quả ra terminal. Dùng cho **script, cron job, command-line tool,
hàng đợi (queue worker), migration...**

**2. Chạy code inline với `-r`**

```bash
php -r "echo 1 + 1;"   # in ra 2 — tiện test nhanh, không cần tạo file
```

**3. Shell tương tác với `-a`**

```bash
php -a   # gõ từng dòng PHP và xem kết quả ngay (REPL)
```

#### B. Chạy qua web server (phục vụ HTTP request)

**4. PHP built-in web server (chỉ để DEV)**

PHP có sẵn một web server nhẹ, bật bằng một lệnh — rất tiện khi phát triển:

```bash
php -S localhost:8000              # thư mục hiện tại làm docroot
php -S localhost:8000 -t public    # chỉ định docroot là thư mục public/
```

> ⚠️ **Không dùng cho production** — đơn luồng, thiếu tính năng bảo mật/hiệu năng. Chỉ để dev/test.

**5. Web server thật + PHP-FPM (PRODUCTION)**

Trên production, một **web server** (Nginx/Apache) nhận HTTP request và đẩy file `.php` cho PHP xử lý.
Phổ biến nhất hiện nay:

- **Nginx + PHP-FPM**: Nginx nhận request, forward request file `.php` qua giao thức **FastCGI** tới
  **PHP-FPM** (FastCGI Process Manager) — một SAPI quản lý sẵn một *pool* tiến trình PHP để xử lý song
  song, hiệu năng cao.
- **Apache + mod_php** (nhúng PHP vào tiến trình Apache) hoặc **Apache + PHP-FPM**.

```nginx
# ví dụ cấu hình Nginx forward .php sang PHP-FPM
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;          # PHP-FPM lắng nghe ở đây
    fastcgi_index index.php;
    include fastcgi_params;
}
```

Với dự án PHP/Laravel, **mọi request đi vào một file entry duy nhất** (thường là `public/index.php`),
file này bootstrap toàn bộ ứng dụng (front controller pattern).

#### C. Các cách tiện ích khác

- **CLI của framework**: Laravel có `php artisan serve` (thực chất bọc lại built-in server cho dev);
  Symfony có `symfony serve`.
- **Docker**: đóng gói bằng image có sẵn như `php:8.5-fpm` hoặc `php:8.5-apache` — chạy `docker run`,
  môi trường đồng nhất giữa các máy.

#### Bảng tổng hợp

| Cách | Lệnh tiêu biểu | SAPI | Dùng khi |
|------|----------------|------|----------|
| Chạy file CLI | `php script.php` | cli | script, cron, queue, tool |
| Code inline | `php -r "..."` | cli | test nhanh |
| Shell tương tác | `php -a` | cli | thử nghiệm REPL |
| Built-in server | `php -S localhost:8000` | cli-server | **dev** web app |
| Nginx/Apache + PHP-FPM | cấu hình web server | fpm | **production** web app |
| Apache + mod_php | cấu hình Apache | apache2handler | web app (cũ hơn) |

### 🎯 Tóm tắt trả lời phỏng vấn
- PHP cần **engine** thông dịch; cách chạy khác nhau ứng với **SAPI** khác nhau. Chia 2 nhóm: **CLI**
  và **web server**.
- **CLI:** `php file.php` (chạy script/cron/queue), `php -r "code"` (chạy nhanh đoạn code), `php -a`
  (shell tương tác).
- **Web — dev:** `php -S localhost:8000 -t public` dùng **built-in server** (tiện nhưng **không cho
  production**).
- **Web — production:** **web server (Nginx/Apache) + PHP-FPM**. Nginx nhận request, forward `.php`
  qua **FastCGI** tới pool tiến trình **PHP-FPM**; request đi vào **một entry file** `public/index.php`
  (front controller). Apache thì có thêm `mod_php`.
- Tiện ích: framework có CLI riêng (`php artisan serve`), và thường chạy trong **Docker** (`php:8.5-fpm`)
  để đồng nhất môi trường.

[↑ Mục lục](#mục-lục)

<a id="q16"></a>
## 16. PHP phiên bản mới nhất là gì? Có gì hơn các phiên bản thấp hơn nó?

### 📖 Giới thiệu kiến thức

#### Phiên bản PHP mới nhất

Tính đến ngày **13/06/2026**, phiên bản ổn định mới nhất là **PHP 8.5.7**. Đây là bản vá lỗi
thuộc nhánh PHP 8.5; nhánh PHP 8.5 được phát hành lần đầu ngày **20/11/2025**.

Bạn có thể kiểm tra phiên bản đang dùng bằng:

```bash
php -v
```

#### PHP 8.5 có gì mới?

**1. Pipe operator `|>`**

Cho phép truyền kết quả của hàm trước vào hàm tiếp theo, giúp code xử lý dữ liệu dễ đọc hơn.

PHP cũ:

```php
$title = strtolower(
    str_replace(' ', '-', trim($title))
);
```

PHP 8.5:

```php
$title = $title
    |> trim(...)
    |> (fn(string $value) => str_replace(' ', '-', $value))
    |> strtolower(...);
```

Điểm mạnh là luồng xử lý được đọc **từ trái sang phải**, thay vì phải đọc các hàm lồng nhau từ
trong ra ngoài.

**2. `clone()` có thể thay đổi thuộc tính**

PHP 8.5 cho phép clone object và thay một số thuộc tính ngay trong quá trình clone. Tính năng này
đặc biệt hữu ích với immutable object và readonly class.

```php
readonly class User
{
    public function __construct(
        public string $name,
        public string $role,
    ) {}
}

$admin = new User('Cuom', 'admin');

$editor = clone($admin, [
    'role' => 'editor',
]);
```

Ở các phiên bản thấp hơn, bạn thường phải tự viết phương thức kiểu `withRole()` hoặc tạo object
mới thủ công.

**3. Thuộc tính `#[\NoDiscard]`**

Dùng để cảnh báo khi lập trình viên gọi một hàm nhưng bỏ qua giá trị trả về quan trọng.

```php
#[\NoDiscard]
function saveUser(): bool
{
    return true;
}

saveUser(); // Cảnh báo vì kết quả bị bỏ qua
```

Khi chủ động muốn bỏ qua kết quả:

```php
(void) saveUser();
```

Tính năng này hữu ích cho các hàm như lưu dữ liệu, kiểm tra quyền, xử lý transaction hoặc trả về
đối tượng kết quả.

**4. Closure trong constant expression**

PHP 8.5 cho phép dùng closure và first-class callable trong nhiều biểu thức hằng, chẳng hạn:

- Giá trị mặc định của parameter.
- Giá trị mặc định của property.
- Attribute parameter.
- Constant và class constant.

Điều này giúp các thư viện và framework tạo API cấu hình linh hoạt hơn.

**5. URI extension mới**

PHP 8.5 bổ sung extension xử lý URI/URL theo chuẩn **RFC 3986** và **WHATWG URL**.

So với `parse_url()`, API mới hướng đối tượng và chặt chẽ hơn:

```php
use Uri\Rfc3986\Uri;

$uri = new Uri('https://example.com/products?page=2');

echo $uri->getHost();
```

Nó phù hợp hơn cho ứng dụng cần parse, validate và biến đổi URL an toàn.

**6. Fatal error có stack trace tốt hơn**

Một số fatal error, chẳng hạn vượt quá thời gian thực thi, giờ có thể kèm backtrace. Điều này giúp
tìm vị trí lỗi nhanh hơn khi debug production.

**7. Hỗ trợ ảnh HEIF, HEIC và SVG tốt hơn**

PHP 8.5 bổ sung `getimagesize($file);` cho:

- HEIF/HEIC.
- SVG khi có libxml.
- Thông tin đơn vị chiều rộng và chiều cao.

Điều này hữu ích với hệ thống upload ảnh từ iPhone và các ứng dụng xử lý media.

**8. Cookie partitioned**

Các hàm như `setcookie()`, `session_start()`, `session_set_cookie_params()` hỗ trợ thuộc tính
cookie **partitioned**. Tính năng này phục vụ các trường hợp cookie bên thứ ba trong bối cảnh
trình duyệt ngày càng siết chặt quyền riêng tư.

**9. cURL tái sử dụng kết nối tốt hơn**

PHP 8.5 hỗ trợ **persistent cURL share handles**, giúp tái sử dụng DNS cache và connection giữa
nhiều request PHP hiệu quả hơn. Điều này có thể mang lại lợi ích cho ứng dụng gọi nhiều API bên
ngoài, microservice hoặc payment gateway.

#### So sánh với các phiên bản thấp hơn

| Phiên bản | Điểm nổi bật | Tình trạng hỗ trợ |
|-----------|--------------|-------------------|
| PHP 7.x | Nhanh hơn PHP 5, scalar type, return type | Đã hết hỗ trợ |
| PHP 8.0 | JIT, attributes, union types, constructor promotion, match | Đã hết hỗ trợ |
| PHP 8.1 | Enum, readonly property, Fiber, intersection type | Đã hết hỗ trợ |
| PHP 8.2 | Readonly class, DNF types, nhiều dynamic property bị deprecated | Chỉ còn cập nhật bảo mật đến 31/12/2026 |
| PHP 8.3 | Typed class constants, cải thiện readonly cloning, Randomizer | Cập nhật bảo mật đến 31/12/2027 |
| PHP 8.4 | Property hooks, asymmetric visibility, lazy objects | Hỗ trợ tích cực đến 31/12/2026 |
| PHP 8.5 | Pipe operator, URI API, Clone With, NoDiscard, constant closures | Hỗ trợ tích cực đến 31/12/2027 |

Theo lịch chính thức, **PHP 8.5 được hỗ trợ tích cực đến 31/12/2027** và nhận bản vá bảo mật đến
**31/12/2029**.

### 🎯 Tóm tắt trả lời phỏng vấn
- **Mới nhất là PHP 8.5** (nhánh ra 20/11/2025, bản vá hiện tại 8.5.x). Bản cũ như **PHP 7.x đã
  end-of-life** — không còn vá bảo mật, nên thực tế phải dùng bản còn được support.
- **5 điểm mới đáng nhớ của 8.5:** **pipe operator `|>`** (đọc luồng trái→phải, bỏ hàm lồng nhau),
  **`clone()` đổi thuộc tính** (hợp readonly/immutable), **`#[\NoDiscard]`** (cảnh báo bỏ qua giá
  trị trả về), **URI extension** chuẩn RFC 3986/WHATWG, và **closure trong constant expression**.
- **Cải thiện vận hành:** fatal error có backtrace, `getimagesize` hỗ trợ HEIF/HEIC/SVG, cookie
  **partitioned**, persistent **cURL share handle** (tái dùng connection/DNS cache).
- **Bức tranh dòng 8.x:** 8.0 JIT + union types + match + attributes; 8.1 enum + readonly + Fiber;
  8.2 readonly class; 8.3 typed constants; 8.4 property hooks + asymmetric visibility; 8.5 pipe
  operator. Tóm lại bản mới hơn ở **hiệu năng, type system mạnh hơn, cú pháp gọn hơn, và còn được
  vá bảo mật**.

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
