<a id="mục-lục"></a>
# Common — SOLID & Design Pattern

> 5 nguyên lý SOLID (SRP · OCP · LSP · ISP · DIP) · Design Pattern (Creational/Structural/Behavioral:
> Factory, Singleton, Strategy, Observer, Repository, Adapter...) · nguyên tắc thiết kế (DRY/KISS/YAGNI).

## 📑 Mục lục

- [1. SOLID là gì?](#q1)
  - [1.1. Dependency Injection (DI) và IoC là gì?](#q1-1)
- [2. Design Pattern là gì?](#q2)
  - [2.1. Singleton Pattern là gì?](#q2-1)
  - [2.2. Factory Pattern là gì?](#q2-2)
    - [2.2.1. So sánh Factory Method và Abstract Factory](#q2-2-1)
  - [2.3. Builder Pattern là gì?](#q2-3)
  - [2.4. Decorator Pattern là gì?](#q2-4)
  - [2.5. Facade Pattern là gì?](#q2-5)
  - [2.6. Proxy Pattern là gì?](#q2-6)
  - [2.7. Adapter Pattern là gì?](#q2-7)
  - [2.8. Strategy Pattern là gì?](#q2-8)
  - [2.9. Observer Pattern là gì?](#q2-9)
  - [2.10. Command Pattern là gì?](#q2-10)
  - [2.11. Repository Pattern là gì?](#q2-11)
  - [2.12. Các pattern trên được Laravel / NestJS / Vue triển khai thế nào?](#q2-12)
- [3. Phân biệt SOLID và Design Pattern khác nhau thế nào?](#q3)
- [4. Architecture Pattern (Mẫu kiến trúc) là gì?](#q4)
  - [4.1. Microservice là gì?](#q4-1)
  - [4.2. Phân biệt Monolith với Microservices](#q4-2)
  - [4.3. Các mô hình khác ngoài MVC (MVP, MVVM, Flux/Redux)](#q4-3)
- [5. Layered Architecture (Kiến trúc phân tầng) là gì?](#q5)
- [6. Clean Architecture là gì?](#q6)
  - [6.1. Clean Architecture thường được triển khai ở đâu?](#q6-1)
  - [6.2. Phân biệt Layered Architecture với Clean Architecture](#q6-2)
  - [6.3. Với `OrderService` "nồi lẩu hạ tầng" (DB + cache + payment + email), Clean Architecture xử lý thế nào?](#q6-3)

---

<a id="q1"></a>
## 1. SOLID là gì?

### 📖 Giới thiệu kiến thức

**SOLID** là tập hợp **5 nguyên lý thiết kế** hướng đối tượng, do **Robert C. Martin (Uncle Bob)**
tổng hợp, giúp viết code **dễ bảo trì, dễ mở rộng và dễ hiểu**. Mỗi chữ cái là một nguyên lý:

| Chữ | Tên nguyên lý | Ý nghĩa cốt lõi |
|-----|---------------|-----------------|
| **S** | **Single Responsibility** (SRP) | Một class chỉ nên có **một lý do để thay đổi** (một trách nhiệm duy nhất). |
| **O** | **Open/Closed** (OCP) | **Mở** cho mở rộng, **đóng** cho sửa đổi — thêm tính năng bằng cách *thêm code mới*, không sửa code cũ. |
| **L** | **Liskov Substitution** (LSP) | Object lớp con phải **thay thế được** lớp cha mà không làm hỏng chương trình. |
| **I** | **Interface Segregation** (ISP) | Nhiều interface **nhỏ, chuyên biệt** tốt hơn một interface lớn "ôm đồm". |
| **D** | **Dependency Inversion** (DIP) | Phụ thuộc vào **abstraction (interface)**, không phụ thuộc vào **implementation cụ thể**. |

**1. SRP — Single Responsibility Principle.** Một class ôm quá nhiều việc (vừa xử lý nghiệp vụ, vừa
ghi log, vừa gửi email) → mỗi thay đổi đều có nguy cơ làm hỏng phần khác. Tách mỗi trách nhiệm ra
một class riêng.

**2. OCP — Open/Closed Principle.** Khi cần thêm hành vi mới, ta *thêm class/strategy mới* thay vì
sửa `if/else` trong class cũ — tránh phá vỡ code đã chạy ổn định. (Strategy Pattern ở [câu 2](#q2)
là cách hiện thực OCP điển hình.)

```php
// ❌ Vi phạm OCP: thêm hình mới phải sửa hàm này
function area($shape) {
    if ($shape->type === 'circle') return 3.14 * $shape->r ** 2;
    if ($shape->type === 'square') return $shape->side ** 2;
    // thêm 'triangle' → phải sửa ở đây...
}

// ✅ Tuân thủ OCP: thêm hình mới chỉ cần thêm class mới
interface Shape { public function area(): float; }
class Circle implements Shape {
    public function __construct(private float $r) {}
    public function area(): float { return 3.14 * $this->r ** 2; }
}
class Square implements Shape {
    public function __construct(private float $side) {}
    public function area(): float { return $this->side ** 2; }
}
// Thêm Triangle implements Shape — KHÔNG đụng vào code cũ.
```

**3. LSP — Liskov Substitution Principle.** Nếu `B extends A`, thì mọi chỗ dùng `A` đều phải thay
được bằng `B` mà không gây lỗi/bất ngờ. Ví dụ kinh điển vi phạm: `Square extends Rectangle` nhưng
việc set width/height của Square phá vỡ hành vi mong đợi của Rectangle.

**4. ISP — Interface Segregation Principle.** Đừng ép class implement những method nó không dùng.
Tách interface lớn `Worker { work(); eat(); }` thành `Workable` và `Eatable` để `Robot` chỉ cần
implement `Workable` (robot không `eat`).

**5. DIP — Dependency Inversion Principle.** Module cấp cao không phụ thuộc trực tiếp vào module
cấp thấp; cả hai phụ thuộc vào **abstraction**. Đây là nền tảng của **Dependency Injection**.

```php
// ❌ Phụ thuộc trực tiếp implementation
class OrderService {
    private MySQLRepository $repo;            // bị khóa cứng vào MySQL
    public function __construct() { $this->repo = new MySQLRepository(); }
}

// ✅ Phụ thuộc abstraction → dễ thay/test (inject MySQL, Mongo, hay Mock)
interface OrderRepository { public function save(array $o): void; }
class OrderService {
    public function __construct(private OrderRepository $repo) {}  // inject từ ngoài
}
```

**Tại sao quan trọng:** SOLID giúp code **loose coupling** (ít phụ thuộc chéo), **dễ test** (mock
dependency qua interface), **dễ mở rộng** (thêm tính năng không sửa code cũ). SOLID là **nguyên lý
nền tảng** mà nhiều **Design Pattern** ([câu 2](#q2)) hiện thực hóa.

**Lưu ý:** SOLID là *kim chỉ nam*, không phải luật cứng. Áp dụng máy móc (tách quá nhiều class,
interface cho mọi thứ) gây over-engineering, vi phạm KISS/YAGNI. Cân bằng theo ngữ cảnh.

### 🎯 Tóm tắt trả lời phỏng vấn

- **SOLID** = 5 nguyên lý thiết kế OOP (Uncle Bob) giúp code **dễ bảo trì, mở rộng, test**:
  - **S** – Single Responsibility: một class, **một trách nhiệm** (một lý do để đổi).
  - **O** – Open/Closed: **mở** cho mở rộng, **đóng** cho sửa đổi (thêm code mới, không sửa cũ).
  - **L** – Liskov Substitution: lớp con **thay thế được** lớp cha mà không gây lỗi.
  - **I** – Interface Segregation: nhiều interface **nhỏ, chuyên biệt** hơn một interface lớn.
  - **D** – Dependency Inversion: phụ thuộc **abstraction (interface)**, không phụ thuộc
    implementation → nền tảng của Dependency Injection.
- Lợi ích: **loose coupling**, dễ test (mock qua interface), dễ mở rộng. Nhiều **Design Pattern**
  chính là cách hiện thực SOLID.
- Là kim chỉ nam, **không áp dụng máy móc** kẻo over-engineering (KISS/YAGNI).

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Dependency Injection (DI) và IoC là gì?

#### 📖 Giới thiệu kiến thức

Đây là cặp khái niệm hay bị nhầm lẫn. Cần tách rõ **3 tầng**: **DIP** (nguyên lý) → **IoC** (nguyên
tắc rộng) → **DI** (kỹ thuật cụ thể) → **IoC Container** (công cụ tự động hóa).

**IoC — Inversion of Control (Đảo ngược quyền điều khiển):** là một **nguyên tắc thiết kế rộng**, theo
đó **quyền điều khiển luồng chương trình / việc tạo & nối object** được **chuyển từ code của bạn sang
một bên ngoài** (framework hoặc container). Thay vì code của bạn chủ động gọi và tạo mọi thứ, framework
**gọi ngược lại code của bạn** — nguyên tắc *"Hollywood: Don't call us, we'll call you"*. Ví dụ IoC:
framework gọi event handler/lifecycle hook của bạn, container tự tạo object cho bạn, router tự gọi
controller.

**DI — Dependency Injection (Tiêm phụ thuộc):** là **một kỹ thuật cụ thể để hiện thực IoC** — thay vì
một object **tự tạo** các dependency của nó (`new`), thì dependency được **truyền (inject) từ bên
ngoài** vào. DI chỉ là *một trong nhiều cách* làm IoC, nhưng là cách phổ biến nhất.

```php
// ❌ KHÔNG dùng DI: OrderService tự tạo dependency → khóa cứng, khó test
class OrderService {
    private MysqlOrderRepository $repo;
    public function __construct() {
        $this->repo = new MysqlOrderRepository();   // tự "new" → coupling chặt
    }
}

// ✅ Dùng DI (constructor injection): dependency được tiêm từ ngoài qua interface
class OrderService {
    public function __construct(private OrderRepository $repo) {}  // không quan tâm class cụ thể
}
// Nơi khác quyết định tiêm cái gì → dễ thay MySQL/Mongo/Mock:
$service = new OrderService(new MysqlOrderRepository());
```

**Quan hệ với DIP (xem [câu 1](#q1)):** **DIP** là *nguyên lý* (phụ thuộc abstraction); **DI** là
*cơ chế* giúp tuân DIP (tiêm abstraction vào qua constructor); **IoC** là *nguyên tắc bao trùm* (đảo
quyền điều khiển, DI là một dạng của nó). Nói gọn: **DIP nói "nên thế nào", DI nói "làm thế nào", IoC
là tinh thần chung.**

**3 kiểu Dependency Injection:**
1. **Constructor Injection (khuyến nghị):** tiêm qua hàm khởi tạo → dependency là **bắt buộc & bất
   biến**, object luôn ở trạng thái hợp lệ. Phổ biến nhất.
2. **Setter/Property Injection:** tiêm qua setter/thuộc tính → dùng cho dependency **tùy chọn**.
3. **Method/Interface Injection:** tiêm trực tiếp qua tham số method khi gọi.

**IoC Container (DI Container):** là thành phần của framework **tự động tạo object và nối (wire) các
dependency** cho bạn — bạn chỉ khai báo "interface này dùng implementation kia", container lo phần còn
lại (kể cả dependency lồng nhau nhiều tầng).
- **Laravel — Service Container:** `$this->app->bind(OrderRepository::class, MysqlOrderRepository::class)`;
  khi controller **type-hint** `OrderRepository`, Laravel **tự resolve & inject** (autowiring).
- **NestJS:** `@Injectable()` + constructor injection, module khai báo provider — container tự lo.
- **Spring (Java):** `@Autowired`; **Vue:** `provide()` / `inject()` cũng là một dạng DI ở cấp component.

**Lợi ích:**
- **Loose coupling:** object không phụ thuộc class cụ thể → dễ thay thế.
- **Dễ test:** inject **mock/stub** thay cho dependency thật (không cần DB/HTTP thật).
- **Cấu hình tập trung:** đổi implementation ở **một chỗ** (binding trong container), không sửa nơi dùng.
- Là nền tảng để hiện thực **Repository ([câu 2.11](#q2-11))** và **Clean Architecture ([câu 6](#q6))**
  (tiêm adapter vào port).

**Lưu ý:** đừng nhầm **DI ≠ DI Container** — bạn có thể làm DI thủ công (tự `new` rồi truyền vào) mà
**không cần** container; container chỉ **tự động hóa** việc đó. Lạm dụng container cho app nhỏ cũng là
phức tạp thừa.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **IoC (Inversion of Control)** là **nguyên tắc rộng**: đảo quyền điều khiển việc tạo/nối object &
  luồng chạy từ code của bạn sang **framework/container** ("Don't call us, we'll call you").
- **DI (Dependency Injection)** là **một kỹ thuật cụ thể hiện thực IoC**: thay vì object **tự `new`**
  dependency, **tiêm từ ngoài** vào (thường qua **constructor**) → object phụ thuộc **interface**, không
  phụ thuộc class cụ thể.
- **Quan hệ:** **DIP** (nguyên lý SOLID) ← **DI** (cơ chế thực hiện) ← **IoC** (tinh thần bao trùm);
  **IoC Container** (Laravel Service Container, NestJS, Spring) **tự động hóa** việc tạo & inject.
- **3 kiểu DI:** Constructor (khuyến nghị), Setter, Method injection.
- **Lợi ích:** loose coupling, **dễ test** (inject mock), cấu hình tập trung. **Lưu ý:** DI ≠ DI
  Container — DI làm thủ công được, container chỉ tự động hóa.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Design Pattern là gì?

### 📖 Giới thiệu kiến thức

**Design Pattern (mẫu thiết kế)** là **giải pháp tổng quát, đã được kiểm chứng** cho những **vấn đề
thiết kế phần mềm lặp đi lặp lại** trong lập trình hướng đối tượng. Nó **không phải đoạn code copy-paste**
mà là một **khuôn mẫu / công thức** mô tả *cách tổ chức các class và object* để giải quyết một vấn đề —
bạn áp dụng ý tưởng đó vào ngữ cảnh cụ thể của mình.

Khái niệm được phổ biến bởi cuốn sách kinh điển **"Design Patterns: Elements of Reusable Object-Oriented
Software" (1994)** của nhóm tác giả thường gọi là **GoF (Gang of Four)**, mô tả **23 pattern** nền tảng.

**Tại sao cần Design Pattern?**
- **Tái sử dụng kinh nghiệm:** dùng lại giải pháp mà các kỹ sư đi trước đã tối ưu, không phải "phát
  minh lại bánh xe".
- **Ngôn ngữ chung (vocabulary):** khi nói *"chỗ này dùng Factory"* hay *"đây là Observer"*, cả team
  hiểu ngay kiến trúc mà không cần giải thích dài dòng → giao tiếp hiệu quả.
- **Code dễ bảo trì & mở rộng:** pattern thường hướng tới *lỏng lẻo phụ thuộc* (loose coupling),
  dễ thay đổi mà không phá vỡ phần khác.

**Phân loại (theo GoF) — 3 nhóm:**

| Nhóm | Mục đích | Ví dụ tiêu biểu |
|------|----------|-----------------|
| **Creational** (khởi tạo) | Cơ chế **tạo object** linh hoạt, che giấu logic khởi tạo | Factory Method, Abstract Factory, **Singleton**, Builder, Prototype |
| **Structural** (cấu trúc) | **Lắp ghép** class/object thành cấu trúc lớn hơn | Adapter, Decorator, Facade, Proxy, Composite, Bridge |
| **Behavioral** (hành vi) | Cách các object **giao tiếp & phân chia trách nhiệm** | **Strategy**, **Observer**, Repository*, Command, Iterator, Template Method, State |

> \*Repository không thuộc 23 GoF gốc nhưng rất phổ biến trong backend (Laravel...).

**Ví dụ ngắn — Strategy Pattern** (đóng gói thuật toán có thể thay đổi):

```php
interface PaymentStrategy {
    public function pay(int $amount): string;
}

class MomoPayment implements PaymentStrategy {
    public function pay(int $amount): string { return "Trả $amount qua Momo"; }
}
class VnpayPayment implements PaymentStrategy {
    public function pay(int $amount): string { return "Trả $amount qua VNPay"; }
}

class Checkout {
    public function __construct(private PaymentStrategy $strategy) {}
    public function process(int $amount): string {
        return $this->strategy->pay($amount);   // không cần biết là Momo hay VNPay
    }
}

// Dễ dàng đổi thuật toán thanh toán mà không sửa class Checkout:
echo (new Checkout(new MomoPayment()))->process(100);
echo (new Checkout(new VnpayPayment()))->process(100);
```

**Phân biệt với khái niệm gần kề:**
- **Design Pattern vs Nguyên lý (SOLID/DRY/KISS):** nguyên lý là **kim chỉ nam trừu tượng** ("nên thế
  nào"); pattern là **giải pháp cụ thể** ("làm thế nào"). Pattern tốt thường tuân theo SOLID.
- **Design Pattern vs Architecture Pattern:** Design Pattern ở **cấp class/object** (Factory, Observer);
  Architecture Pattern ở **cấp hệ thống** (MVC, Microservices, Layered).
- **Anti-pattern:** giải pháp *trông có vẻ đúng* nhưng gây hại về lâu dài (vd: God Object).

**Lưu ý quan trọng:** đừng **lạm dụng** pattern. Áp pattern vào chỗ không cần thiết làm code phức tạp
hơn (over-engineering, vi phạm KISS/YAGNI). Pattern chỉ nên dùng khi *thực sự có vấn đề mà nó giải quyết*.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Design Pattern** là **giải pháp tổng quát, đã được kiểm chứng** cho các **vấn đề thiết kế
  lặp lại** trong OOP — là *khuôn mẫu cách tổ chức class/object*, không phải code có sẵn để copy.
- Phổ biến bởi sách **GoF (1994)** với **23 pattern**, chia **3 nhóm**: **Creational** (tạo object —
  Factory, Singleton), **Structural** (lắp ghép — Adapter, Decorator), **Behavioral** (hành vi —
  Strategy, Observer).
- **Lợi ích:** tái sử dụng kinh nghiệm, tạo *ngôn ngữ chung* cho team, code dễ bảo trì & mở rộng
  (loose coupling).
- Khác **nguyên lý** (SOLID/DRY — kim chỉ nam trừu tượng) và **architecture pattern** (MVC,
  Microservices — cấp hệ thống).
- **Không lạm dụng:** dùng sai chỗ gây over-engineering, vi phạm KISS/YAGNI.

[↑ Mục lục](#mục-lục)

<a id="q2-1"></a>
### 2.1. Singleton Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Singleton** là một **Creational Pattern** đảm bảo một class **chỉ có duy nhất MỘT instance** trong
toàn bộ vòng đời ứng dụng, đồng thời cung cấp **một điểm truy cập toàn cục (global access point)** tới
instance đó.

**Khi nào dùng?** Khi việc có nhiều instance là vô nghĩa hoặc gây hại — những tài nguyên *dùng chung*
mà cả ứng dụng chỉ nên có một: **kết nối database (connection pool)**, **logger**, **config manager**,
**cache**.

**Cách hiện thực (3 yếu tố bắt buộc):**
1. **Constructor `private`** → chặn `new` từ bên ngoài.
2. **Một thuộc tính `static`** giữ instance duy nhất.
3. **Một method `static` `getInstance()`** → tạo instance ở lần gọi đầu (lazy), các lần sau trả về
   instance đã có.

```php
class Database
{
    private static ?Database $instance = null;   // giữ instance duy nhất
    private \PDO $connection;

    // private → không cho new Database() từ ngoài
    private function __construct()
    {
        $this->connection = new \PDO('mysql:host=localhost;dbname=app', 'user', 'pass');
    }

    public static function getInstance(): Database
    {
        if (self::$instance === null) {          // lazy: chỉ tạo khi cần lần đầu
            self::$instance = new self();
        }
        return self::$instance;
    }

    // Chặn clone & unserialize để không tạo được bản sao
    private function __clone() {}
    public function __wakeup() { throw new \Exception("Không thể unserialize Singleton"); }

    public function query(string $sql) { return $this->connection->query($sql); }
}

// Dùng — luôn là cùng một object:
$db1 = Database::getInstance();
$db2 = Database::getInstance();
var_dump($db1 === $db2);   // true → cùng instance
```

**Vấn đề cần lưu ý (interviewer rất hay "đào"):**
- **Thread-safety:** trong môi trường đa luồng, hai luồng có thể cùng vượt qua kiểm tra `=== null`
  và tạo 2 instance → cần khóa (double-checked locking). *(PHP-FPM mỗi request một process nên ít gặp,
  nhưng Java/C#/Node-worker thì phải xử lý.)*
- **Bị coi là *anti-pattern* khi lạm dụng:** Singleton thực chất là **global state** trá hình → tạo
  **phụ thuộc ẩn** (hidden dependency), **vi phạm DIP** (xem [câu 1](#q1)), khiến code **khó test**
  (không mock/thay thế được) và khó tái sử dụng.
- **Giải pháp hiện đại:** thay vì tự viết Singleton, dùng **Dependency Injection Container** quản lý
  vòng đời object ở scope "singleton" (Laravel `$this->app->singleton()`, NestJS provider mặc định
  là singleton). Cách này giữ lợi ích "một instance" nhưng **inject qua interface** nên *vẫn test được*.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Singleton** là Creational Pattern đảm bảo class **chỉ có 1 instance** duy nhất và cung cấp
  **điểm truy cập toàn cục** tới nó.
- Hiện thực: **constructor `private`** + thuộc tính **`static`** giữ instance + method **`getInstance()`**
  tạo lazy ở lần đầu, trả lại instance cũ các lần sau (nhớ chặn `clone`/`unserialize`).
- **Dùng cho:** DB connection, logger, config, cache — tài nguyên dùng chung chỉ nên có một.
- **Nhược điểm:** là **global state** trá hình → phụ thuộc ẩn, **vi phạm DIP**, **khó test**; cần lo
  **thread-safety** ở môi trường đa luồng.
- **Hiện đại:** ưu tiên **DI Container** quản lý scope singleton (Laravel/NestJS) thay vì tự viết —
  vừa "một instance" vừa test được.

[↑ Mục lục](#mục-lục)

<a id="q2-2"></a>
### 2.2. Factory Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Factory Pattern** là một **Creational Pattern** đóng gói **logic khởi tạo object** vào một nơi
riêng (một "nhà máy"), để **nơi sử dụng không cần biết & không phụ thuộc vào class cụ thể** được tạo
ra. Thay vì rải `new ConcreteClass()` khắp code, ta gọi factory và nhận về object qua **interface chung**.

**Vấn đề nó giải quyết:** khi việc tạo object phức tạp, hoặc loại object cần tạo **phụ thuộc điều kiện
runtime** (vd: tạo cổng thanh toán theo lựa chọn của user). Dùng `new` trực tiếp sẽ khóa cứng code
vào class cụ thể → khó mở rộng, **vi phạm OCP & DIP** (xem [câu 1](#q1)).

**Phân biệt 3 biến thể (interviewer hay hỏi):** cùng dùng một "sản phẩm" là `PaymentGateway`
(Momo/VNPay) để thấy rõ sự khác nhau giữa 3 biến thể qua ví dụ bên dưới.

| Biến thể | "Nhà máy" là gì | Ai quyết định tạo class nào | Tạo bao nhiêu loại sản phẩm |
|----------|-----------------|------------------------------|------------------------------|
| **Simple Factory** | **1 method** nhận tham số | `switch/match` bên trong method | 1 sản phẩm |
| **Factory Method** | **1 method abstract**, mỗi **subclass** override | **Lớp con** (mỗi loại = 1 subclass factory) | 1 sản phẩm |
| **Abstract Factory** | **1 interface factory** có nhiều method | Chọn **factory cụ thể** lúc khởi tạo | **Cả họ** sản phẩm liên quan |

Sản phẩm dùng chung cho cả 3 ví dụ:

```php
interface PaymentGateway {
    public function pay(int $amount): string;
}
class MomoGateway implements PaymentGateway {
    public function pay(int $amount): string { return "Momo: $amount"; }
}
class VnpayGateway implements PaymentGateway {
    public function pay(int $amount): string { return "VNPay: $amount"; }
}
```

---

**① Simple Factory** — *một method gom toàn bộ logic khởi tạo, dùng `switch/match` để chọn class.*
Không thuộc 23 GoF nhưng phổ biến nhất thực tế vì đơn giản.

```php
class PaymentFactory {
    public static function create(string $type): PaymentGateway
    {
        return match ($type) {                 // logic khởi tạo gom 1 chỗ
            'momo'  => new MomoGateway(),
            'vnpay' => new VnpayGateway(),
            default => throw new \InvalidArgumentException("Không hỗ trợ: $type"),
        };
    }
}

// Nơi dùng KHÔNG cần biết class cụ thể, chỉ làm việc qua interface:
$gateway = PaymentFactory::create($request->method);  // 'momo' hoặc 'vnpay'
echo $gateway->pay(100);
// Thêm cổng mới (ZaloPay) → thêm class + 1 dòng trong factory, nơi dùng không đổi.
```

> **Giải thích:** chỉ có **một** factory duy nhất; thêm loại mới phải **sửa** thân hàm `create`
> (thêm nhánh `match`). Đơn giản nhưng khi danh sách phình to thì nhánh `match` chính là chỗ vi
> phạm OCP.

---

**② Factory Method** — *không dùng `switch`; thay vào đó mỗi loại sản phẩm có **một subclass factory
riêng** override method tạo.* Việc "chọn tạo class nào" chuyển từ `if/switch` thành **chọn subclass**.

```php
abstract class PaymentService {
    // Factory Method: lớp con quyết định tạo gateway nào
    abstract protected function createGateway(): PaymentGateway;

    // Logic nghiệp vụ chung, dùng sản phẩm do lớp con tạo ra
    public function checkout(int $amount): string {
        $gateway = $this->createGateway();      // không biết class cụ thể
        return "Thanh toán -> " . $gateway->pay($amount);
    }
}

class MomoService extends PaymentService {
    protected function createGateway(): PaymentGateway { return new MomoGateway(); }
}
class VnpayService extends PaymentService {
    protected function createGateway(): PaymentGateway { return new VnpayGateway(); }
}

// Chọn "nhà máy" bằng cách chọn subclass, không còn switch:
$service = new MomoService();          // hoặc new VnpayService()
echo $service->checkout(100);
// Thêm ZaloPay → tạo ZaloService extends PaymentService, KHÔNG sửa class cũ → tuân OCP tốt hơn.
```

> **Giải thích:** lớp cha `PaymentService` định nghĩa *khung* (`checkout`) nhưng **hoãn việc khởi
> tạo** cho lớp con qua `createGateway()`. Thêm loại mới = **thêm subclass mới**, không đụng code cũ
> → tuân OCP chặt hơn Simple Factory. Đánh đổi: sinh ra nhiều class hơn.

---

**③ Abstract Factory** — *tạo **cả một họ sản phẩm liên quan** đi cùng nhau.* Ví dụ mỗi nhà cung cấp
thanh toán có **2 sản phẩm gắn bó**: cổng thanh toán (`PaymentGateway`) **và** bộ tạo hóa đơn
(`Refund`) tương ứng — không được trộn Momo gateway với VNPay refund.

```php
interface Refund {                              // sản phẩm thứ 2 trong "họ"
    public function refund(int $amount): string;
}
class MomoRefund  implements Refund { public function refund(int $a): string { return "Hoàn Momo: $a"; } }
class VnpayRefund implements Refund { public function refund(int $a): string { return "Hoàn VNPay: $a"; } }

// Abstract Factory: 1 interface, NHIỀU method tạo các sản phẩm cùng họ
interface PaymentProviderFactory {
    public function createGateway(): PaymentGateway;
    public function createRefund(): Refund;
}

class MomoFactory implements PaymentProviderFactory {
    public function createGateway(): PaymentGateway { return new MomoGateway(); }
    public function createRefund(): Refund        { return new MomoRefund(); }
}
class VnpayFactory implements PaymentProviderFactory {
    public function createGateway(): PaymentGateway { return new VnpayGateway(); }
    public function createRefund(): Refund        { return new VnpayRefund(); }
}

// Chọn 1 factory → đảm bảo cả họ sản phẩm ĐỒNG BỘ (không lẫn Momo gateway + VNPay refund):
function process(PaymentProviderFactory $factory) {
    echo $factory->createGateway()->pay(100);
    echo $factory->createRefund()->refund(50);
}
process(new MomoFactory());   // toàn bộ là Momo
process(new VnpayFactory());  // toàn bộ là VNPay
```

> **Giải thích:** điểm khác cốt lõi là Abstract Factory tạo **nhiều sản phẩm liên quan** (gateway +
> refund) và **đảm bảo chúng đồng bộ** theo cùng một "họ". Dùng khi bạn có nhiều nhóm sản phẩm phải
> đi cùng nhau (kinh điển: bộ UI Windows vs macOS — Button + Checkbox + Scrollbar cùng phong cách).

**Tóm lại sự khác biệt:** Simple Factory & Factory Method tạo **1 loại** sản phẩm (khác nhau ở *chỗ
chọn class*: `switch` vs `subclass`); Abstract Factory tạo **cả họ** sản phẩm liên quan đồng bộ.

**Lợi ích:**
- **Tách logic khởi tạo** khỏi logic nghiệp vụ (SRP) → một chỗ duy nhất để sửa khi cách tạo thay đổi.
- **Tuân OCP & DIP:** thêm loại sản phẩm mới ít/không sửa code cũ; nơi dùng phụ thuộc **interface**,
  không phụ thuộc class cụ thể.
- **Dễ test:** có thể cho factory trả về mock/stub.

**Lưu ý:** Simple Factory với `match/switch` lớn dần cũng có thể vi phạm OCP (mỗi loại mới phải sửa
factory). Khi đó cân nhắc **Factory Method** hoặc đăng ký (registry/map). Đừng dùng factory cho object
khởi tạo tầm thường (`new` một dòng) — gây phức tạp thừa (KISS).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Factory Pattern** là Creational Pattern **đóng gói logic khởi tạo object** vào một "nhà máy", để
  nơi dùng nhận object qua **interface chung** mà **không phụ thuộc class cụ thể**.
- **Giải quyết:** tránh rải `new` khắp nơi, đặc biệt khi loại object cần tạo **phụ thuộc điều kiện
  runtime** → tuân **OCP & DIP**, dễ mở rộng.
- **3 biến thể:** **Simple Factory** (switch/match trả object — phổ biến nhất), **Factory Method**
  (lớp con quyết định tạo gì), **Abstract Factory** (tạo cả họ object liên quan).
- **Lợi ích:** tách logic tạo khỏi nghiệp vụ (SRP), thêm loại mới ít sửa code cũ, dễ test (trả mock).
- **Lưu ý:** `switch` lớn dần vẫn có thể phá OCP → cân nhắc registry/Factory Method; đừng lạm dụng
  cho khởi tạo tầm thường (KISS).

[↑ Mục lục](#mục-lục)

<a id="q2-2-1"></a>
#### 2.2.1. So sánh Factory Method và Abstract Factory

##### 📖 Giới thiệu kiến thức

Đây là câu hay bị nhầm vì cả hai cùng nhóm Creational, tên gần giống nhau. Khác biệt cốt lõi nằm ở
**số "họ" sản phẩm** chúng tạo ra và **cơ chế** dùng để chọn class (xem chi tiết từng biến thể ở
[câu 2.2](#q2-2)).

> Lưu ý thuật ngữ: khi nói "**Factory**" trong câu hỏi này, ý chỉ **Factory Method** (mẫu GoF tạo
> *một* sản phẩm). "Simple Factory" (1 method + `switch`) chỉ là dạng rút gọn, không phải mẫu GoF.

**Điểm khác nhau cốt lõi:**

| Tiêu chí | **Factory Method** | **Abstract Factory** |
|----------|--------------------|----------------------|
| **Tạo gì** | **1 sản phẩm** (một product hierarchy) | **Cả họ nhiều sản phẩm liên quan** (nhiều hierarchy) |
| **Cơ chế** | **Kế thừa** — lớp con override 1 method tạo | **Composition** — truyền/inject 1 object factory vào dùng |
| **Cấu trúc** | **1 method** tạo object trong abstract class | **1 interface** chứa **nhiều method** create |
| **Mục tiêu** | Hoãn việc *"tạo class nào"* cho lớp con | Đảm bảo **cả nhóm sản phẩm đồng bộ** một "họ" |
| **Số class sinh ra** | Ít hơn | Nhiều hơn (mỗi họ = 1 factory, nhiều product) |
| **Ví dụ** | `MomoService::createGateway()` | `MomoFactory{ createGateway(); createRefund(); }` |

**Quan hệ giữa hai mẫu:** không đối lập — **Abstract Factory thường được hiện thực *bằng* nhiều
Factory Method**. Mỗi method `createGateway()`, `createRefund()` bên trong một Abstract Factory chính
là một Factory Method. Hiểu đơn giản: **Abstract Factory = một nhóm Factory Method gom lại để tạo ra
một họ sản phẩm đi cùng nhau.**

**Cách nhớ nhanh:**
- Chỉ cần tạo **một loại** object, muốn lớp con quyết định loại cụ thể → **Factory Method**.
- Cần tạo **nhiều loại** object **phải khớp nhau thành một bộ** (không được trộn lẫn họ) →
  **Abstract Factory**. Ví dụ kinh điển: bộ UI cho **Windows** vs **macOS** — `Button` + `Checkbox` +
  `Scrollbar` phải cùng một phong cách; chọn `WindowsFactory` thì *mọi* widget đều theo Windows.

**So sánh trực quan qua code** (rút gọn từ ví dụ ở [câu 2.2](#q2-2)):

```php
// FACTORY METHOD: 1 method, lớp con quyết định tạo gateway nào
abstract class PaymentService {
    abstract protected function createGateway(): PaymentGateway;   // chỉ 1 sản phẩm
}
class MomoService extends PaymentService {
    protected function createGateway(): PaymentGateway { return new MomoGateway(); }
}

// ABSTRACT FACTORY: 1 interface, NHIỀU method tạo cả họ (gateway + refund) đồng bộ
interface PaymentProviderFactory {
    public function createGateway(): PaymentGateway;   // sản phẩm 1
    public function createRefund(): Refund;            // sản phẩm 2 — cùng họ
}
class MomoFactory implements PaymentProviderFactory {
    public function createGateway(): PaymentGateway { return new MomoGateway(); }
    public function createRefund(): Refund          { return new MomoRefund(); }
}
```

##### 🎯 Tóm tắt trả lời phỏng vấn

- **Khác biệt cốt lõi ở số "họ" sản phẩm:** **Factory Method** tạo **1 sản phẩm** (dùng **kế thừa** —
  lớp con override method tạo); **Abstract Factory** tạo **cả họ nhiều sản phẩm liên quan, đồng bộ**
  (dùng **composition** — chọn 1 factory object có nhiều method create).
- **Cấu trúc:** Factory Method = **1 method** trong abstract class; Abstract Factory = **1 interface
  nhiều method** create.
- **Quan hệ:** không đối lập — **Abstract Factory thường được hiện thực bằng nhiều Factory Method**
  (mỗi method create là một Factory Method).
- **Khi nào dùng:** cần tạo *một loại* và muốn lớp con quyết định → **Factory Method**; cần tạo *một
  bộ* sản phẩm phải khớp nhau (UI Windows vs macOS, gateway + refund cùng nhà cung cấp) →
  **Abstract Factory**.

[↑ Mục lục](#mục-lục)

<a id="q2-3"></a>
### 2.3. Builder Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Builder Pattern** là một **Creational Pattern** dùng để **xây dựng một object phức tạp theo từng
bước**, tách **quá trình khởi tạo** ra khỏi **biểu diễn cuối cùng** của object. Cùng một quy trình
build có thể tạo ra nhiều "phiên bản" object khác nhau.

**Vấn đề nó giải quyết — "telescoping constructor":** khi một object có **nhiều tham số**, nhất là
nhiều tham số **tùy chọn (optional)**, constructor phình to và khó dùng:

```php
// ❌ Constructor "telescoping": dài, khó đọc, không nhớ thứ tự, nhiều null
$pizza = new Pizza('large', true, false, true, false, true, 'thin');
//                  size,  cheese, olives, bacon, ...  -> tham số nào là gì??
```

Builder thay thế bằng cách **gọi từng method đặt từng thuộc tính** rồi `build()` ra object hoàn chỉnh
— đọc như văn xuôi, chỉ set thứ mình cần.

**Cách hiện thực phổ biến (fluent builder):** mỗi method `setX()` trả về `$this` để **xâu chuỗi
(method chaining)**; cuối cùng `build()` trả về product.

```php
class Pizza {
    public function __construct(
        public string $size,
        public array $toppings = [],
        public string $crust = 'normal',
    ) {}
}

class PizzaBuilder {
    private string $size = 'medium';
    private array $toppings = [];
    private string $crust = 'normal';

    public function size(string $s): static   { $this->size = $s;  return $this; }  // trả $this → chaining
    public function addTopping(string $t): static { $this->toppings[] = $t; return $this; }
    public function crust(string $c): static  { $this->crust = $c; return $this; }

    public function build(): Pizza {
        // có thể validate trước khi tạo (vd: size hợp lệ?) rồi mới dựng object
        return new Pizza($this->size, $this->toppings, $this->crust);
    }
}

// Đọc như văn xuôi, chỉ set thứ cần — KHÔNG quan tâm thứ tự tham số:
$pizza = (new PizzaBuilder())
    ->size('large')
    ->crust('thin')
    ->addTopping('cheese')
    ->addTopping('bacon')
    ->build();
```

**Thành phần (bản GoF đầy đủ):**
- **Product:** object phức tạp cần tạo (`Pizza`).
- **Builder:** interface khai báo các bước build.
- **Concrete Builder:** cài đặt các bước, giữ trạng thái trung gian, `build()` trả product.
- **Director (tùy chọn):** đóng gói **trình tự build** cố định (vd `makeMargherita()` gọi sẵn các
  bước), để client không cần biết thứ tự. Thực tế (PHP/JS) thường bỏ Director, dùng fluent builder.

**Ứng dụng thực tế:**
- **Laravel Query Builder:** `DB::table('users')->where('active', 1)->orderBy('name')->limit(10)->get();`
  — xây query phức tạp từng bước.
- **HTTP client, PHPMailer, StringBuilder (Java), `URLSearchParams`/request builder (JS).**
- Bất cứ API nào dùng **method chaining** để cấu hình dần một object.

**Phân biệt với Factory ([câu 2.2](#q2-2)):**
- **Factory** tập trung *"tạo loại object nào"* — trả về object trong **một bước**, che giấu class cụ thể.
- **Builder** tập trung *"tạo object phức tạp **như thế nào**"* — dựng **qua nhiều bước**, hợp khi
  object có nhiều thành phần/tùy chọn hoặc cần cấu hình dần.

**Lợi ích & lưu ý:**
- Tránh constructor khổng lồ; code khởi tạo **dễ đọc**, linh hoạt (chỉ set thứ cần); tách logic build
  (có thể validate trong `build()`) → tuân **SRP**.
- **Lưu ý:** với object **đơn giản** (ít field) thì Builder là *thừa* (over-engineering, vi phạm KISS)
  — chỉ dùng khi việc khởi tạo thực sự phức tạp/nhiều biến thể.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Builder Pattern** (Creational) **xây object phức tạp theo từng bước**, tách *quá trình khởi tạo*
  khỏi *biểu diễn cuối*; thường dùng **method chaining** (`setX()` trả `$this`) rồi `build()`.
- **Giải quyết:** vấn đề **"telescoping constructor"** — object nhiều tham số (nhất là optional) khiến
  constructor dài, khó đọc, dễ nhầm thứ tự.
- **Thành phần:** Product, Builder, Concrete Builder, **Director** (tùy chọn — đóng gói trình tự build).
- **Thực tế:** Laravel **Query Builder**, HTTP client, PHPMailer, StringBuilder — mọi fluent API.
- **Khác Factory:** Factory chọn *"tạo loại nào"* trong 1 bước; Builder lo *"tạo phức tạp **thế nào**"*
  qua nhiều bước. Đừng dùng cho object đơn giản (KISS).

[↑ Mục lục](#mục-lục)

<a id="q2-4"></a>
### 2.4. Decorator Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Decorator Pattern** là một **Structural Pattern** cho phép **gắn thêm hành vi/trách nhiệm mới vào
một object một cách linh hoạt lúc runtime**, bằng cách **bọc (wrap)** object gốc trong một object
"decorator" có **cùng interface**. Nó là **giải pháp thay thế cho kế thừa** để mở rộng chức năng —
ưu tiên **composition over inheritance**.

**Vấn đề nó giải quyết — "bùng nổ class do kế thừa":** giả sử cần Coffee có thể thêm sữa, đường,
kem... Nếu dùng kế thừa, phải tạo `CoffeeWithMilk`, `CoffeeWithMilkAndSugar`,
`CoffeeWithMilkSugarCream`... → số class **bùng nổ tổ hợp**. Decorator giải quyết bằng cách **bọc
chồng nhiều lớp** tùy ý lúc runtime, mỗi lớp thêm một phần hành vi.

**Cách hoạt động:** decorator **giữ một tham chiếu** tới object nó bọc (cùng interface), khi được gọi
nó **ủy thác (delegate)** sang object bên trong rồi **thêm phần của mình** vào trước/sau.

```php
interface Coffee {
    public function cost(): int;
    public function description(): string;
}

// Object gốc (ConcreteComponent)
class SimpleCoffee implements Coffee {
    public function cost(): int { return 20; }
    public function description(): string { return "Cà phê"; }
}

// Decorator cơ sở: cùng interface + giữ tham chiếu object bị bọc
abstract class CoffeeDecorator implements Coffee {
    public function __construct(protected Coffee $coffee) {}   // bọc object cùng interface
}

class MilkDecorator extends CoffeeDecorator {
    public function cost(): int { return $this->coffee->cost() + 5; }          // ủy thác + thêm
    public function description(): string { return $this->coffee->description() . " + sữa"; }
}
class SugarDecorator extends CoffeeDecorator {
    public function cost(): int { return $this->coffee->cost() + 2; }
    public function description(): string { return $this->coffee->description() . " + đường"; }
}

// Bọc chồng tùy ý lúc runtime — không cần class CoffeeWithMilkAndSugar:
$coffee = new MilkDecorator(new SugarDecorator(new SimpleCoffee()));
echo $coffee->description();   // "Cà phê + đường + sữa"
echo $coffee->cost();          // 27
```

**Thành phần:**
1. **Component:** interface chung (`Coffee`) cho cả object gốc lẫn decorator.
2. **Concrete Component:** object gốc cần được "trang trí" (`SimpleCoffee`).
3. **Decorator (base):** cùng interface, **giữ tham chiếu** tới một Component.
4. **Concrete Decorator:** thêm hành vi cụ thể, gọi object bên trong rồi bổ sung phần của nó.

**Ứng dụng thực tế:**
- **Middleware/pipeline** (Laravel middleware, PSR-15): mỗi middleware bọc request/response thêm một
  lớp xử lý (auth → log → CORS...).
- **Java I/O Streams:** `new BufferedReader(new FileReader(...))` — chồng tính năng buffer lên reader.
- **PHP stream filters**, các lớp wrapper cache/log quanh một service.

**Phân biệt nhanh:**
- **Decorator vs kế thừa:** kế thừa thêm hành vi **lúc compile, tĩnh**; Decorator thêm **lúc runtime,
  linh hoạt**, bọc chồng được nhiều lớp.
- **Decorator vs Proxy:** cùng "bọc" và cùng interface, nhưng **mục đích khác** — Decorator **thêm
  chức năng**; Proxy **kiểm soát truy cập** (lazy load, cache, kiểm tra quyền) tới object thật.

**Lợi ích & lưu ý:**
- Thêm/bớt trách nhiệm linh hoạt **không sửa class gốc** → tuân **OCP**; mỗi decorator một trách
  nhiệm → **SRP**; tránh bùng nổ class so với kế thừa.
- **Lưu ý:** bọc quá nhiều lớp khiến khó debug (stack sâu, khó lần ra ai thêm gì); thứ tự bọc có thể
  ảnh hưởng kết quả.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Decorator Pattern** (Structural) **gắn thêm hành vi cho object lúc runtime** bằng cách **bọc
  (wrap)** nó trong một object cùng interface — là cách mở rộng chức năng thay cho kế thừa
  (**composition over inheritance**).
- **Giải quyết:** tránh **bùng nổ class do tổ hợp kế thừa** (CoffeeWithMilkAndSugar...) — thay vào đó
  **bọc chồng** nhiều decorator tùy ý.
- **Cách chạy:** decorator giữ tham chiếu object bị bọc, **ủy thác** rồi **thêm phần của mình**.
- **Thực tế:** middleware (Laravel/PSR-15), Java I/O `BufferedReader(new FileReader())`, wrapper
  cache/log.
- **Khác Proxy:** cùng bọc nhưng Decorator **thêm chức năng**, Proxy **kiểm soát truy cập**. Tuân
  **OCP & SRP**; lưu ý bọc nhiều lớp khó debug.

[↑ Mục lục](#mục-lục)

<a id="q2-5"></a>
### 2.5. Facade Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Facade Pattern** là một **Structural Pattern** cung cấp một **interface đơn giản, thống nhất** cho
một **hệ thống con (subsystem) phức tạp** gồm nhiều class. "Facade" nghĩa là **mặt tiền** — che giấu
toàn bộ sự phức tạp bên trong sau một cửa vào duy nhất, dễ dùng.

**Vấn đề nó giải quyết:** khi để hoàn thành một tác vụ, client phải gọi **nhiều class theo đúng trình
tự** và biết cách chúng phối hợp (vd: đặt hàng = kiểm kho + trừ tiền + tạo vận đơn + gửi email).
Bắt client ôm hết logic phối hợp này gây **coupling chặt** với chi tiết bên trong. Facade gom toàn bộ
vào **một method dễ gọi**, client chỉ cần biết "tôi muốn đặt hàng".

```php
// Hệ thống con phức tạp — nhiều class riêng lẻ
class Inventory { public function check(int $id): bool { return true; } }
class Payment   { public function charge(int $amount): bool { return true; } }
class Shipping  { public function ship(int $id): string { return "Đang giao #$id"; } }

// FACADE: một cửa vào duy nhất, giấu trình tự phối hợp bên trong
class OrderFacade {
    public function __construct(
        private Inventory $inventory = new Inventory(),
        private Payment $payment = new Payment(),
        private Shipping $shipping = new Shipping(),
    ) {}

    public function placeOrder(int $productId, int $amount): string {
        if (!$this->inventory->check($productId)) return "Hết hàng";
        if (!$this->payment->charge($amount))     return "Thanh toán lỗi";
        return $this->shipping->ship($productId);          // gom cả quy trình vào 1 method
    }
}

// Client chỉ gọi 1 method, KHÔNG cần biết Inventory/Payment/Shipping phối hợp ra sao:
echo (new OrderFacade())->placeOrder(productId: 42, amount: 100);
```

**Đặc điểm chính:**
- Facade **không thêm chức năng mới**, chỉ **đơn giản hóa cách dùng** subsystem có sẵn.
- Client **vẫn có thể** truy cập trực tiếp các class bên trong nếu cần — Facade chỉ là *lối tắt tiện
  lợi*, không chặn đường.
- Giảm phụ thuộc của client vào chi tiết subsystem → **loose coupling**.

**Ứng dụng thực tế:**
- **Laravel Facades** (`Cache::get()`, `DB::table()`, `Route::get()`): cung cấp giao diện static
  gọn cho các service phức tạp trong service container. *(Lưu ý: implementation của Laravel hơi khác
  GoF — nó là static proxy resolve service từ container — nhưng tinh thần "mặt tiền đơn giản" thì giống.)*
- **SDK/thư viện client** bọc nhiều lời gọi API thành vài method dễ dùng; lớp service gom nhiều
  repository/external call.

**Phân biệt nhanh:**
- **Facade vs Adapter:** Adapter **chuyển đổi** một interface có sẵn sang interface khác mà client cần
  (làm cho *tương thích*); Facade **đơn giản hóa** một subsystem (làm cho *dễ dùng*), không nhằm khớp
  interface cụ thể.
- **Facade vs Decorator ([câu 2.4](#q2-4)):** Decorator **thêm hành vi** và **giữ nguyên interface**
  của object bị bọc; Facade tạo **interface mới, đơn giản hơn** che một nhóm class.

**Lợi ích & lưu ý:**
- Giảm độ phức tạp phía client, **loose coupling** với subsystem, dễ đọc, dễ thay đổi bên trong mà
  client không bị ảnh hưởng.
- **Lưu ý:** Facade dễ phình thành **"God Object"** ôm quá nhiều việc nếu nhồi mọi thứ vào — nên giữ
  mỗi facade tập trung một nhóm nghiệp vụ (tuân SRP).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Facade Pattern** (Structural) cung cấp **một interface đơn giản, thống nhất** che giấu sự phức tạp
  của **một subsystem nhiều class** — "mặt tiền" với một cửa vào dễ dùng.
- **Giải quyết:** client không phải gọi nhiều class theo đúng trình tự (kiểm kho + trừ tiền + giao
  hàng...) — gom hết vào **một method** (`placeOrder`).
- **Đặc điểm:** **không thêm chức năng**, chỉ đơn giản hóa; client vẫn truy cập trực tiếp subsystem
  nếu cần → giảm coupling.
- **Thực tế:** **Laravel Facades** (`Cache::`, `DB::`), SDK/service layer bọc nhiều call.
- **Khác Adapter** (chuyển đổi interface cho *tương thích*) và **Decorator** (thêm hành vi, giữ
  interface). Lưu ý đừng để Facade phình thành God Object.

[↑ Mục lục](#mục-lục)

<a id="q2-6"></a>
### 2.6. Proxy Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Proxy Pattern** là một **Structural Pattern** cung cấp một **object đại diện (surrogate/placeholder)**
đứng **thay cho một object thật**, nhằm **kiểm soát việc truy cập** tới object đó. Proxy có **cùng
interface** với object thật, nên client dùng proxy y như dùng object thật — nhưng proxy chèn thêm
logic *trước/sau* khi chuyển lời gọi (ủy thác) sang object thật.

**Vấn đề nó giải quyết:** đôi khi ta cần **làm gì đó quanh việc truy cập** một object — trì hoãn khởi
tạo tốn kém, kiểm tra quyền, cache kết quả, ghi log, hay gọi qua mạng — mà **không muốn sửa object
thật** và không muốn client phải tự lo. Proxy bọc object thật và làm việc đó hộ.

**Các loại Proxy phổ biến:**

| Loại | Mục đích |
|------|----------|
| **Virtual Proxy** | **Lazy loading** — trì hoãn khởi tạo object nặng đến khi thực sự cần |
| **Protection Proxy** | **Kiểm soát quyền** truy cập (chỉ admin mới gọi được) |
| **Remote Proxy** | Đại diện cho object ở **tiến trình/máy khác** (gọi RPC/API trông như gọi local) |
| **Caching Proxy** | **Cache** kết quả để lần sau khỏi gọi lại object thật |
| **Logging/Smart Proxy** | Ghi log, đếm tham chiếu... quanh mỗi lời gọi |

**Ví dụ (PHP) — Virtual Proxy lazy-load ảnh nặng:**

```php
interface Image {
    public function display(): string;
}

// Object thật: tốn kém khi khởi tạo (đọc file từ disk)
class RealImage implements Image {
    public function __construct(private string $file) {
        echo "Tải ảnh $file từ disk...\n";       // nặng, chỉ nên làm khi cần
    }
    public function display(): string { return "Hiển thị {$this->file}"; }
}

// Proxy: cùng interface, hoãn tạo RealImage đến khi display() lần đầu
class ImageProxy implements Image {
    private ?RealImage $real = null;
    public function __construct(private string $file) {}

    public function display(): string {
        if ($this->real === null) {                // lazy: chỉ tạo khi thực sự dùng
            $this->real = new RealImage($this->file);
        }
        return $this->real->display();             // ủy thác sang object thật
    }
}

$image = new ImageProxy('photo.jpg');   // CHƯA tải file — nhẹ
echo $image->display();                 // Lúc này mới tải + hiển thị
```

**Ứng dụng thực tế:**
- **Eloquent Lazy Loading (Laravel):** `$user->posts` chỉ truy vấn DB khi bạn *thực sự* truy cập
  quan hệ — đây là tinh thần Virtual Proxy.
- **Reverse proxy (Nginx), API Gateway:** đứng trước server thật, kiểm soát/định tuyến request (ở
  quy mô hạ tầng).
- **Caching layer, ORM proxy object, lazy-loaded service trong DI container.**

**Phân biệt nhanh (interviewer cực hay hỏi cả cụm Structural):**
- **Proxy vs Decorator ([câu 2.4](#q2-4)):** cùng "bọc" và cùng interface, nhưng **mục đích khác** —
  Decorator **thêm chức năng** cho object; Proxy **kiểm soát truy cập** (lazy/quyền/cache), *không*
  đổi chức năng. Decorator thường bọc chồng nhiều lớp; Proxy thường một lớp.
- **Proxy vs Facade ([câu 2.5](#q2-5)):** Proxy có **cùng interface** với **một** object thật; Facade
  tạo **interface mới đơn giản hơn** che **nhiều** class của một subsystem.

**Lợi ích & lưu ý:**
- Thêm kiểm soát (lazy/quyền/cache/log) **không sửa object thật & không phiền client** → tuân **OCP &
  SRP**.
- **Lưu ý:** thêm một lớp gián tiếp → tăng độ phức tạp; proxy lazy/cache có thể gây hành vi khó đoán
  (lần gọi đầu chậm, cache cũ) nếu không cẩn thận.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Proxy Pattern** (Structural) là **object đại diện** đứng thay object thật để **kiểm soát truy cập**
  tới nó; proxy **cùng interface** nên client dùng y như object thật, nhưng chèn logic trước/sau khi
  **ủy thác**.
- **Các loại:** **Virtual** (lazy load object nặng), **Protection** (kiểm soát quyền), **Remote**
  (object ở máy khác), **Caching** (cache kết quả).
- **Thực tế:** **Eloquent lazy loading** (`$user->posts`), reverse proxy/API gateway, caching layer.
- **Khác Decorator:** cùng bọc & cùng interface, nhưng Proxy **kiểm soát truy cập** còn Decorator
  **thêm chức năng**. **Khác Facade:** Proxy cùng interface một object; Facade là interface mới cho
  cả subsystem.
- Tuân **OCP & SRP**; lưu ý thêm lớp gián tiếp làm tăng độ phức tạp.

[↑ Mục lục](#mục-lục)

<a id="q2-7"></a>
### 2.7. Adapter Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Adapter Pattern** là một **Structural Pattern** đóng vai trò **"bộ chuyển đổi"** giúp **hai interface
không tương thích làm việc được với nhau**. Nó **bọc một object có interface khác** và **dịch** các
lời gọi sang interface mà code của bạn mong đợi — giống như **phích cắm chuyển đổi** (adapter điện)
giúp thiết bị 2 chấu cắm vào ổ 3 chấu.

**Vấn đề nó giải quyết:** code của bạn được viết để làm việc với một interface chuẩn (vd
`PaymentGateway::pay()`), nhưng một **thư viện/SDK bên thứ ba** lại có method tên khác
(`StripeApi::makeCharge()`). Bạn **không sửa được code thư viện**, và không muốn rải lời gọi đặc thù
của nó khắp nơi. Adapter **bọc** thư viện đó và phơi ra đúng interface bạn cần.

```php
// Interface mà hệ thống của bạn mong đợi
interface PaymentGateway {
    public function pay(int $amount): string;
}

// Thư viện bên thứ 3 — interface KHÁC, không sửa được
class StripeApi {
    public function makeCharge(float $dollars): string {
        return "Stripe đã thu \${$dollars}";
    }
}

// ADAPTER: bọc StripeApi, dịch pay() -> makeCharge()
class StripeAdapter implements PaymentGateway {
    public function __construct(private StripeApi $stripe) {}
    public function pay(int $amount): string {
        $dollars = $amount / 23000;                  // dịch dữ liệu: VND -> USD
        return $this->stripe->makeCharge($dollars);  // dịch lời gọi
    }
}

// Code của bạn chỉ làm việc với PaymentGateway, KHÔNG cần biết Stripe:
function checkout(PaymentGateway $gateway, int $amount): string {
    return $gateway->pay($amount);
}
echo checkout(new StripeAdapter(new StripeApi()), 230000);
```

**Hai dạng:**
- **Object Adapter (phổ biến, dùng composition):** adapter **giữ tham chiếu** tới object cần adapt
  (như ví dụ trên). Linh hoạt, hợp với PHP (đơn kế thừa).
- **Class Adapter (dùng kế thừa đa tầng):** adapter **kế thừa** cả interface đích lẫn lớp cần adapt —
  cần đa kế thừa, ít dùng trong PHP/Java.

**Ứng dụng thực tế:**
- **Tích hợp SDK/thư viện bên thứ ba** có interface khác chuẩn nội bộ (payment, SMS, storage...).
- **Laravel Flysystem:** một interface lưu trữ chung, bên dưới là các **adapter** cho Local/S3/FTP —
  đổi `disk` mà code không đổi. **PSR adapter** (cache, log) cũng vậy.
- **Bọc legacy code / API cũ** để dùng được trong hệ thống mới mà không viết lại.

**Phân biệt nhanh (hoàn chỉnh cụm Structural):**
- **Adapter vs Decorator ([câu 2.4](#q2-4)):** Decorator **giữ nguyên interface** và **thêm hành vi**;
  Adapter **đổi interface** (làm cho *tương thích*), không thêm chức năng.
- **Adapter vs Facade ([câu 2.5](#q2-5)):** Adapter làm một interface có sẵn **khớp** interface client
  cần (thường bọc **1** class); Facade tạo **interface mới đơn giản hơn** cho **cả subsystem** nhiều class.
- **Adapter vs Proxy ([câu 2.6](#q2-6)):** Proxy **giữ cùng interface** và kiểm soát truy cập; Adapter
  **đổi sang interface khác** cho tương thích.

**Lợi ích & lưu ý:**
- Tích hợp được code/thư viện **không tương thích** mà **không sửa** chúng (tuân **OCP**); tách phần
  phụ thuộc bên thứ ba vào một chỗ (dễ thay nhà cung cấp — tuân **DIP**, xem [câu 1](#q1)).
- **Lưu ý:** chỉ là lớp "dịch", **không thêm chức năng**; quá nhiều adapter chồng nhau là dấu hiệu
  thiết kế interface chưa thống nhất.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Adapter Pattern** (Structural) là **bộ chuyển đổi** giúp **hai interface không tương thích làm việc
  với nhau** — bọc một object và **dịch** lời gọi sang interface mà client mong đợi (ví von: phích
  cắm chuyển đổi).
- **Giải quyết:** tích hợp **thư viện/SDK bên thứ ba** hoặc **legacy code** có interface khác, mà
  **không sửa** chúng → code client chỉ phụ thuộc interface chuẩn của mình.
- **Hai dạng:** **Object Adapter** (composition — phổ biến) và **Class Adapter** (kế thừa).
- **Thực tế:** Laravel **Flysystem** (adapter Local/S3/FTP), PSR cache/log adapter, bọc SDK payment/SMS.
- **Khác Decorator** (giữ interface + thêm hành vi), **Facade** (interface mới cho cả subsystem),
  **Proxy** (giữ interface + kiểm soát truy cập). Tuân **OCP & DIP**; chỉ "dịch", không thêm chức năng.

[↑ Mục lục](#mục-lục)

<a id="q2-8"></a>
### 2.8. Strategy Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Strategy Pattern** là một **Behavioral Pattern** cho phép **định nghĩa một họ thuật toán (algorithm),
đóng gói mỗi thuật toán vào một class riêng và làm cho chúng có thể thay thế lẫn nhau** lúc runtime.
Nhờ vậy thuật toán **biến đổi độc lập** với code sử dụng nó — đổi cách làm mà không sửa nơi gọi.

**Vấn đề nó giải quyết — khối `if/else` (hoặc `switch`) phình to:** khi một tác vụ có **nhiều cách
thực hiện** (nhiều thuật toán) và phải chọn theo điều kiện runtime, nhét hết vào một hàm với
`if/else` khiến hàm cồng kềnh, khó test, **vi phạm OCP** (thêm cách mới phải sửa hàm cũ — xem
[câu 1](#q1)).

```php
// ❌ Vi phạm OCP: thêm hãng vận chuyển mới phải sửa hàm này
function shippingFee(string $carrier, int $weight): int {
    if ($carrier === 'ghn')   return $weight * 10000;
    if ($carrier === 'ghtk')  return $weight * 8000 + 5000;
    // thêm 'viettel' → phải sửa ở đây...
}
```

**Cách Strategy giải quyết:** tách mỗi thuật toán thành một **Concrete Strategy** cùng implement một
**interface chung**; một class **Context** giữ tham chiếu tới strategy và **ủy thác** lời gọi cho nó.
Đổi hành vi = đổi strategy được inject, không đụng Context.

```php
interface ShippingStrategy {
    public function fee(int $weight): int;          // hợp đồng chung cho mọi thuật toán
}

class GhnShipping implements ShippingStrategy {
    public function fee(int $weight): int { return $weight * 10000; }
}
class GhtkShipping implements ShippingStrategy {
    public function fee(int $weight): int { return $weight * 8000 + 5000; }
}

class Order {                                       // Context
    public function __construct(private ShippingStrategy $shipping) {}
    public function total(int $weight): int {
        return $this->shipping->fee($weight);       // không cần biết là hãng nào
    }
}

// Chọn/đổi thuật toán lúc runtime — thêm hãng mới chỉ cần thêm class, KHÔNG sửa Order:
$order = new Order(new GhnShipping());
echo $order->total(2);                              // 20000
$order = new Order(new GhtkShipping());             // đổi chiến lược
echo $order->total(2);                              // 21000
```

**3 thành phần:**
1. **Strategy (interface):** khai báo hợp đồng chung mọi thuật toán phải theo (`fee()`).
2. **Concrete Strategy:** từng cài đặt thuật toán cụ thể (`GhnShipping`, `GhtkShipping`).
3. **Context:** giữ một strategy, **ủy thác** việc thực thi cho nó; client có thể đổi strategy.

**Ứng dụng thực tế:**
- **Thanh toán nhiều cổng** (`PaymentStrategy` — chính là ví dụ ở [câu 2](#q2)), **tính phí ship/giảm
  giá theo nhiều quy tắc**, **nén dữ liệu** (zip/gzip/rar), **sắp xếp** với comparator khác nhau.
- **Laravel:** hệ thống **driver** (cache, queue, filesystem, auth guard) bản chất là Strategy — đổi
  `.env` để chọn driver mà không sửa code dùng.

**Phân biệt nhanh:**
- **Strategy vs Factory ([câu 2.2](#q2-2)):** Factory lo *"tạo ra object nào"*; Strategy lo *"dùng
  thuật toán/hành vi nào"*. Thường đi cùng nhau: Factory tạo ra Concrete Strategy phù hợp.
- **Strategy vs State:** cấu trúc giống nhau (Context + các object hành vi), nhưng **ý định khác** —
  State **tự chuyển** giữa các trạng thái theo logic nội bộ; Strategy do **client chọn** và thường
  không tự đổi.
- **Strategy vs `if/else`:** Strategy thay nhánh điều kiện bằng đa hình (polymorphism) → tuân OCP,
  dễ test từng thuật toán độc lập.

**Lợi ích & lưu ý:**
- Thêm thuật toán mới **không sửa code cũ** (OCP); nơi dùng phụ thuộc **interface** (DIP); mỗi thuật
  toán một class **dễ test riêng** (SRP).
- **Lưu ý:** sinh nhiều class hơn; với 2–3 nhánh đơn giản, ít đổi thì `match/if` có thể đủ — đừng lạm
  dụng (KISS). Client phải biết các strategy để chọn đúng cái.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Strategy Pattern** (Behavioral) **đóng gói một họ thuật toán vào các class riêng cùng interface,
  cho phép thay thế lẫn nhau** lúc runtime → thuật toán biến đổi độc lập với nơi dùng.
- **Giải quyết:** thay khối **`if/else`/`switch`** chọn cách làm bằng **đa hình** → tuân **OCP**
  (thêm thuật toán = thêm class, không sửa code cũ).
- **3 thành phần:** Strategy (interface), Concrete Strategy (từng thuật toán), **Context** (giữ &
  ủy thác cho strategy).
- **Thực tế:** payment gateway, tính phí ship/giảm giá, nén dữ liệu, **driver của Laravel** (cache/
  queue/auth).
- **Khác State** (State tự chuyển trạng thái, Strategy do client chọn) và **Factory** (tạo object vs
  chọn thuật toán). Đừng lạm dụng khi chỉ vài nhánh đơn giản (KISS).

[↑ Mục lục](#mục-lục)

<a id="q2-9"></a>
### 2.9. Observer Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Observer Pattern** là một **Behavioral Pattern** định nghĩa quan hệ **một–nhiều** giữa các object:
khi **một object (Subject / Publisher)** thay đổi trạng thái, **tất cả các object phụ thuộc
(Observer / Subscriber)** sẽ **tự động được thông báo và cập nhật**. Đây chính là mô hình
**publish–subscribe (pub/sub)** ở cấp object.

**Vấn đề nó giải quyết:** khi nhiều thành phần cần **phản ứng theo một sự kiện** (vd: user vừa đăng
ký → gửi email chào mừng + ghi log + cộng điểm thưởng). Nếu nhét hết vào một chỗ, Subject phải **biết
và gọi trực tiếp** từng thành phần → **coupling chặt**, thêm hành vi mới phải sửa Subject (vi phạm
OCP & SRP — xem [câu 1](#q1)). Observer **đảo ngược** điều đó: Subject chỉ "phát thông báo", ai quan
tâm thì **tự đăng ký** lắng nghe.

**3 thành phần:**
1. **Subject (Publisher):** giữ danh sách observer, có method `subscribe()`/`unsubscribe()` và
   `notify()` để phát thông báo.
2. **Observer (Subscriber):** định nghĩa interface với method `update()` được Subject gọi khi có sự kiện.
3. **Concrete Observer:** cài đặt cụ thể phản ứng khi nhận thông báo.

**Ví dụ (PHP) — user đăng ký, nhiều observer cùng phản ứng:**

```php
interface Observer {
    public function update(string $email): void;     // được gọi khi có sự kiện
}

class UserRegistration {                             // Subject / Publisher
    private array $observers = [];

    public function subscribe(Observer $o): void { $this->observers[] = $o; }

    public function register(string $email): void {
        // ... lưu user vào DB ...
        foreach ($this->observers as $o) {           // phát thông báo cho TẤT CẢ observer
            $o->update($email);
        }
    }
}

class SendWelcomeEmail implements Observer {
    public function update(string $email): void { echo "Gửi email chào mừng tới $email\n"; }
}
class WriteLog implements Observer {
    public function update(string $email): void { echo "Ghi log: $email đã đăng ký\n"; }
}

$registration = new UserRegistration();
$registration->subscribe(new SendWelcomeEmail());
$registration->subscribe(new WriteLog());
// Thêm hành vi mới (cộng điểm) → tạo class observer mới + subscribe, KHÔNG sửa UserRegistration:
$registration->register('a@example.com');
```

**Ứng dụng thực tế:**
- **Event/Listener trong Laravel** (`UserRegistered` event → nhiều listener), Node.js `EventEmitter`.
- **Reactivity của Vue/React** (state đổi → component lắng nghe re-render).
- **Cơ chế notification, webhook, message broker** (pub/sub) là Observer ở quy mô lớn.

**Lợi ích & lưu ý:**
- **Loose coupling:** Subject không cần biết observer là ai, chỉ biết chúng implement `Observer`
  → tuân **OCP** (thêm observer không sửa Subject) và **DIP** (phụ thuộc interface).
- **Lưu ý:** thứ tự thông báo khó kiểm soát; observer lỗi có thể ảnh hưởng luồng (nên tách bất đồng
  bộ qua queue nếu nặng); dễ rò rỉ bộ nhớ nếu quên `unsubscribe` (memory leak).

**Phân biệt nhanh với Pub/Sub qua message broker:** Observer cổ điển — Subject **gọi trực tiếp**
observer (đồng bộ, cùng tiến trình). Pub/Sub mở rộng (Kafka/RabbitMQ) — có **message broker trung
gian**, publisher và subscriber **không biết nhau**, thường bất đồng bộ & khác tiến trình.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Observer Pattern** (Behavioral) định nghĩa quan hệ **một–nhiều**: khi **Subject** đổi trạng thái,
  mọi **Observer** đã đăng ký được **tự động thông báo & cập nhật** — chính là **pub/sub** ở cấp object.
- **3 thành phần:** Subject (giữ danh sách + `notify`), Observer (interface `update`), Concrete Observer
  (phản ứng cụ thể).
- **Giải quyết:** nhiều thành phần cần phản ứng theo một sự kiện mà **không coupling chặt** — Subject
  chỉ phát thông báo, ai quan tâm tự **subscribe** → tuân **OCP & DIP**.
- **Thực tế:** Laravel Event/Listener, Node `EventEmitter`, reactivity Vue/React, webhook/message broker.
- **Lưu ý:** khó kiểm soát thứ tự, observer lỗi ảnh hưởng luồng (nên đẩy queue nếu nặng), nhớ
  `unsubscribe` tránh memory leak.

[↑ Mục lục](#mục-lục)

<a id="q2-10"></a>
### 2.10. Command Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Command Pattern** là một **Behavioral Pattern** **đóng gói một yêu cầu (request/hành động) thành một
object** độc lập. Nhờ "biến hành động thành object", ta có thể **truyền đi, lưu vào hàng đợi, ghi log,
hoàn tác (undo)** hay thực thi sau — những việc không làm được nếu chỉ gọi method trực tiếp.

**Vấn đề nó giải quyết:** khi nơi **phát lệnh** (vd: nút bấm, scheduler, API) không nên biết **chi
tiết ai xử lý và xử lý thế nào**. Gọi trực tiếp `receiver->doX()` khiến hai bên **coupling chặt** và
không thể queue/undo/log một cách thống nhất. Command **chèn một object ở giữa** đại diện cho hành
động, tách **"yêu cầu làm gì"** khỏi **"thực hiện ra sao"**.

**4 thành phần:**
1. **Command (interface):** khai báo `execute()` (và `undo()` nếu cần).
2. **Concrete Command:** gắn **Receiver** với một hành động cụ thể; `execute()` gọi method của Receiver.
3. **Receiver:** nơi **thực sự làm việc** (business logic).
4. **Invoker:** giữ và **kích hoạt** command (`execute()`), không cần biết command làm gì bên trong.

```php
interface Command {
    public function execute(): void;
    public function undo(): void;
}

// Receiver — nơi thực sự xử lý
class Light {
    public function on(): void  { echo "Đèn bật\n"; }
    public function off(): void { echo "Đèn tắt\n"; }
}

// Concrete Command — gắn receiver với hành động
class TurnOnLight implements Command {
    public function __construct(private Light $light) {}
    public function execute(): void { $this->light->on(); }
    public function undo(): void    { $this->light->off(); }
}

// Invoker — chỉ "bấm nút", không biết chi tiết
class RemoteControl {
    private array $history = [];
    public function press(Command $command): void {
        $command->execute();
        $this->history[] = $command;          // lưu lại để undo / log / replay
    }
    public function undoLast(): void {
        if ($cmd = array_pop($this->history)) $cmd->undo();
    }
}

$remote = new RemoteControl();
$remote->press(new TurnOnLight(new Light()));   // Đèn bật
$remote->undoLast();                            // Đèn tắt (hoàn tác)
```

**Ứng dụng thực tế:**
- **Hàng đợi / Job (Laravel Queue):** mỗi **Job** là một command được serialize, đẩy vào queue, worker
  lấy ra gọi `handle()` — đúng tinh thần "đóng gói hành động để thực thi sau".
- **Undo/Redo** trong editor, **transaction/replay**, **Artisan/CLI command**, nút bấm trong GUI,
  **macro** (gộp nhiều command).

**Phân biệt nhanh:**
- **Command vs Strategy ([câu 2.8](#q2-8)):** cùng đóng gói hành vi vào object, nhưng **ý định khác** —
  Strategy đóng gói **thuật toán có thể thay thế** cho *cùng một việc*; Command đóng gói **một yêu cầu/
  hành động** (kèm receiver) để **queue/undo/log/trì hoãn**.
- **Command vs gọi method trực tiếp:** gọi trực tiếp thực thi *ngay*; Command **vật thể hóa** lời gọi
  → lưu trữ, truyền đi, đảo ngược được.

**Lợi ích & lưu ý:**
- **Tách Invoker khỏi Receiver** (loose coupling); thêm lệnh mới **không sửa Invoker** (OCP — xem
  [câu 1](#q1)); hỗ trợ **undo/redo, queue, log, macro**.
- **Lưu ý:** mỗi hành động sinh một class → **nhiều class hơn**; với hành động đơn giản, một lần,
  không cần undo/queue thì Command là *thừa* (KISS).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Command Pattern** (Behavioral) **đóng gói một yêu cầu/hành động thành object**, nhờ đó có thể
  **truyền đi, đưa vào hàng đợi, ghi log, hoàn tác (undo)** hoặc thực thi sau.
- **4 thành phần:** Command (interface `execute`), Concrete Command (gắn receiver + hành động),
  **Receiver** (thực sự xử lý), **Invoker** (kích hoạt command, không biết chi tiết).
- **Giải quyết:** tách **nơi phát lệnh** khỏi **nơi xử lý** → loose coupling, thêm lệnh không sửa
  Invoker (OCP).
- **Thực tế:** **Laravel Job/Queue** (mỗi job là một command), undo/redo editor, Artisan command, macro.
- **Khác Strategy:** Strategy = thuật toán thay thế cho cùng việc; Command = đóng gói *hành động* để
  queue/undo/log. Đừng lạm dụng cho hành động đơn giản (KISS).

[↑ Mục lục](#mục-lục)

<a id="q2-11"></a>
### 2.11. Repository Pattern là gì?

#### 📖 Giới thiệu kiến thức

**Repository Pattern** là một mẫu thiết kế **trung gian giữa tầng nghiệp vụ (business logic) và tầng
truy cập dữ liệu (data layer)**. Nó đóng vai như một **"bộ sưu tập object trong bộ nhớ"** (in-memory
collection): code nghiệp vụ chỉ gọi `findById()`, `save()`, `all()`... mà **không cần biết dữ liệu
đến từ đâu** — MySQL, PostgreSQL, API bên ngoài hay cache.

> **Lưu ý nguồn gốc:** Repository **không thuộc 23 GoF** mà đến từ **DDD / Patterns of Enterprise
> Application Architecture (Martin Fowler)** — là *enterprise data-access pattern*, không cùng nhóm
> Creational/Structural/Behavioral. Nhưng rất phổ biến trong backend nên hay được hỏi cùng Design Pattern.

**Vấn đề nó giải quyết:** khi logic truy vấn (Eloquent/SQL) **rải khắp controller/service**, code
nghiệp vụ **bị khóa cứng vào cách lưu trữ** → khó test (phải có DB thật), khó đổi nguồn dữ liệu, truy
vấn lặp lại nhiều nơi. Repository **gom logic truy cập dữ liệu vào một chỗ** và cho nghiệp vụ phụ thuộc
vào **interface** thay vì implementation cụ thể (tuân **DIP** — xem [câu 1](#q1)).

```php
// 1) Interface — hợp đồng truy cập dữ liệu, nghiệp vụ chỉ biết tới cái này
interface UserRepository {
    public function findById(int $id): ?User;
    public function save(User $user): void;
    public function activeUsers(): array;
}

// 2) Cài đặt cụ thể bằng Eloquent (có thể thay bằng API/MongoDB mà không đụng nghiệp vụ)
class EloquentUserRepository implements UserRepository {
    public function findById(int $id): ?User { return User::find($id); }
    public function save(User $user): void   { $user->save(); }
    public function activeUsers(): array     { return User::where('active', 1)->get()->all(); }
}

// 3) Service nghiệp vụ phụ thuộc INTERFACE, không phụ thuộc Eloquent
class UserService {
    public function __construct(private UserRepository $repo) {}   // inject qua interface
    public function activate(int $id): void {
        $user = $this->repo->findById($id);
        // ... logic nghiệp vụ ...
        $this->repo->save($user);
    }
}
```

**Gắn với Laravel:** bind interface → implementation trong **Service Provider**, rồi inject vào
controller/service:

```php
// AppServiceProvider::register()
$this->app->bind(UserRepository::class, EloquentUserRepository::class);
// Đổi nguồn dữ liệu = đổi 1 dòng bind, nghiệp vụ không cần sửa.
```

**Lợi ích:**
- **Tách nghiệp vụ khỏi cách lưu trữ** (DIP) → đổi MySQL ↔ API ↔ Mongo không sửa code nghiệp vụ.
- **Dễ test:** mock `UserRepository` để unit-test service mà không cần DB thật.
- **Gom truy vấn một chỗ** (SRP) → tránh lặp, dễ tối ưu.

**Phân biệt nhanh:**
- **Repository vs DAO (Data Access Object):** DAO bám sát **bảng/persistence** (CRUD theo table);
  Repository hướng **collection của domain object/aggregate**, trừu tượng hơn, gần nghiệp vụ.
- **Repository vs Active Record (Eloquent):** Active Record — **model tự lo lưu trữ**
  (`$user->save()`); Repository tách phần lưu trữ ra một lớp riêng để nghiệp vụ không phụ thuộc ORM.

**Lưu ý quan trọng (interviewer thích đào):** trong Laravel, Eloquent **đã là một abstraction** trên
DB, nên bọc thêm Repository đôi khi là **over-engineering / leaky abstraction** (lặp lại API Eloquent,
mất các tiện ích như scope, eager loading). **Chỉ nên dùng** khi: cần đổi/đa nguồn dữ liệu, domain
phức tạp (DDD), hoặc yêu cầu test nghiêm ngặt. App CRUD đơn giản dùng thẳng Eloquent thường gọn hơn (KISS).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Repository Pattern** là lớp **trung gian giữa nghiệp vụ và data layer**, hành xử như **collection
  object trong bộ nhớ** — nghiệp vụ gọi `findById/save/...` mà **không cần biết dữ liệu đến từ đâu**.
- **Không thuộc GoF** — là *enterprise data pattern* (Fowler/DDD), rất phổ biến trong backend/Laravel.
- **Giải quyết:** gỡ logic truy vấn khỏi controller/service, cho nghiệp vụ phụ thuộc **interface**
  (tuân **DIP**) → đổi nguồn dữ liệu dễ, **dễ test** (mock repo), gom truy vấn một chỗ (SRP).
- **Laravel:** bind `interface → implementation` trong Service Provider rồi inject.
- **Khác Active Record** (Eloquent — model tự lưu) và **DAO** (bám sát bảng). **Lưu ý:** bọc Repository
  trên Eloquent có thể **over-engineering** — chỉ dùng khi cần đa nguồn dữ liệu/DDD/test nghiêm ngặt.

[↑ Mục lục](#mục-lục)

<a id="q2-12"></a>
### 2.12. Các pattern trên được Laravel / NestJS / Vue triển khai thế nào?

#### 📖 Giới thiệu kiến thức

Một câu rất hay được hỏi để kiểm tra **bạn có nhận ra pattern trong code thật hay không** — chứ không
chỉ thuộc định nghĩa. Dưới đây ánh xạ từng pattern (2.1–2.11) sang **ví dụ cụ thể** trong Laravel
(PHP), NestJS (Node) và Vue (frontend).

---

**2.1. Singleton**
- **Laravel:** **Service Container** binding singleton — `$this->app->singleton(X::class, ...)`; các
  manager (config, db, cache) chỉ resolve **một instance** dùng chung toàn app.
- **NestJS:** Provider mặc định là **singleton scope** — một `@Injectable()` service được tạo **một
  lần** và inject khắp nơi.
- **Vue:** Một **Pinia store** là singleton trong phạm vi app; instance `createApp()` là duy nhất.

**2.2. Factory** *(gồm Factory Method/Abstract Factory — 2.2.1)*
- **Laravel:** **Manager/Driver** (`CacheManager::createRedisDriver()`, `createFileDriver()`) là
  Factory Method; **Model Factory** (`UserFactory`) sinh dữ liệu test; `Validator::make()`,
  `View::make()`.
- **NestJS:** **Factory provider** `{ provide: X, useFactory: () => ... }`; dynamic module
  `forRootAsync({ useFactory })` tạo provider theo cấu hình runtime.
- **Vue:** Các factory function `createApp()`, `createRouter()`, `createPinia()`, `h()` (createVNode).

**2.3. Builder**
- **Laravel:** **Query Builder** `DB::table('users')->where('active',1)->orderBy('name')->get()`;
  `Schema::create()` với Blueprint; HTTP client `Http::withHeaders(...)->post(...)`.
- **NestJS:** Swagger `new DocumentBuilder().setTitle('API').setVersion('1.0').build()`; TypeORM
  `repo.createQueryBuilder('u').where(...).getMany()`; test `Test.createTestingModule({...})`.
- **Vue:** Chuỗi fluent `createApp(App).use(router).use(pinia).component(...).mount('#app')`.

**2.4. Decorator**
- **Laravel:** **Middleware pipeline** (`Illuminate\Pipeline`) — mỗi middleware **bọc** request/response
  thêm một lớp (auth → log → CORS).
- **NestJS:** **Interceptors** (`NestInterceptor` + `@UseInterceptors`) bọc handler, thêm logic
  trước/sau (logging, transform response, cache). *(Lưu ý: cú pháp `@Controller`/`@Get` là **TS
  decorator** — tính năng ngôn ngữ để gắn metadata, KHÁC Decorator Pattern; Interceptor mới đúng pattern.)*
- **Vue:** **Wrapper component / HOC**, `<Transition>`/`<KeepAlive>` bọc thêm hành vi cho component
  con; composable bọc/bổ sung logic.

**2.5. Facade**
- **Laravel:** **Facades** đúng tên — `Cache::get()`, `DB::table()`, `Route::get()` *(thực chất là
  static proxy resolve service từ container — tinh thần "mặt tiền đơn giản").*
- **NestJS:** **Service layer** gom nhiều repository/service con thành vài method gọn cho controller;
  `HttpService` (`@nestjs/axios`) bọc axios.
- **Vue:** **Composable** `useAuth()`, `useCart()` che một subsystem logic phức tạp sau API đơn giản.

**2.6. Proxy**
- **Laravel:** **Eloquent lazy loading** — `$user->posts` chỉ truy vấn DB khi *thực sự* truy cập
  (virtual proxy); `LazyCollection`; Facade là static proxy.
- **NestJS:** **Guards** (`CanActivate`) kiểm soát truy cập route trước khi tới handler = **protection
  proxy**.
- **Vue:** **Reactivity dùng `Proxy` của JavaScript theo nghĩa đen** — `reactive(obj)` bọc object bằng
  `new Proxy(...)` để **chặn get/set** mà theo dõi & cập nhật. Đây là Proxy "thật" nhất trong cả 3 FW:

```js
// Bản chất reactive() của Vue 3 (rút gọn)
function reactive(target) {
  return new Proxy(target, {
    get(obj, key) { track(obj, key); return obj[key]; },      // theo dõi phụ thuộc
    set(obj, key, val) { obj[key] = val; trigger(obj, key); return true; } // kích hoạt re-render
  });
}
```

**2.7. Adapter**
- **Laravel:** **Flysystem** — `Storage::disk('s3')` / `'local'` / `'ftp'`, mỗi backend là một
  **adapter** sau interface chung; cache/queue/session **driver** adapt nhiều backend.
- **NestJS:** **Platform adapter** — `ExpressAdapter` vs `FastifyAdapter` adapt hai HTTP framework về
  một interface Nest; **WebSocket adapter** (`IoAdapter`); transport adapter của microservice.
- **Vue:** **Custom renderer** (`createRenderer`) adapt Vue sang môi trường khác (canvas, native...);
  adapter trong các thư viện (Vue I18n, storage).

**2.8. Strategy**
- **Laravel:** **Driver/guard** đổi qua config = strategy — cache `redis/file/memcached`, auth guard
  `session/token`, hash `bcrypt/argon`; đổi `.env` không sửa code.
- **NestJS:** **Passport Strategy** đúng tên — `class JwtStrategy extends PassportStrategy(Strategy)`,
  `LocalStrategy`, `GoogleStrategy`; mỗi cách xác thực là một strategy thay thế được.
- **Vue:** Quy tắc validation thay thế (vee-validate), `scrollBehavior` của Vue Router, truyền các
  hàm xử lý khác nhau qua props.

**2.9. Observer**
- **Laravel:** **Events & Listeners**; **Model Observer** đúng tên — `UserObserver` với
  `created()/updated()/deleted()`; broadcasting.
- **NestJS:** `@nestjs/event-emitter` (`eventEmitter.emit()` + `@OnEvent('user.created')`); **RxJS
  Observable** dùng xuyên suốt.
- **Vue:** **Toàn bộ hệ reactivity** — `watch`/`watchEffect` *quan sát* state đổi để chạy lại; `emit`
  + `v-on`/`@event` lắng nghe sự kiện component.

**2.10. Command**
- **Laravel:** **Artisan Command** (class kế thừa `Command`, có `handle()`); **Job/Queue** — mỗi job
  là một command được serialize đẩy vào queue; `Bus::dispatch()`.
- **NestJS:** **CQRS** (`@nestjs/cqrs`) đúng pattern — `class CreateUserCommand`, `@CommandHandler`,
  `commandBus.execute(new CreateUserCommand(...))`.
- **Vue:** **Pinia actions / Vuex mutations** — `store.commit('addItem', payload)` đóng gói một thay
  đổi state thành "lệnh" (Vuex còn log/replay được qua devtools — đúng tinh thần Command).

**2.11. Repository**
- **Laravel:** **Không có built-in** (Eloquent là **Active Record** — model tự lưu); thường **tự viết**
  `EloquentUserRepository implements UserRepository` rồi bind trong Service Provider.
- **NestJS:** **TypeORM/Mongoose Repository built-in** — `@InjectRepository(User) private repo:
  Repository<User>`; `repo.find()`, `repo.save()`. Đây là Repository "có sẵn" rõ nhất:

```ts
@Injectable()
export class UserService {
  constructor(@InjectRepository(User) private repo: Repository<User>) {}
  findActive() { return this.repo.find({ where: { active: true } }); }
}
```

- **Vue:** Frontend không có repository theo nghĩa DB; tương đương là **service/composable bọc API**
  (`useUserApi()`, `userRepository.ts` gọi axios), Pinia store đóng vai data layer.

#### 🎯 Tóm tắt trả lời phỏng vấn

Bảng "một ví dụ tiêu biểu" mỗi framework:

| Pattern | Laravel | NestJS | Vue |
|---------|---------|--------|-----|
| **Singleton** | Container `singleton()` | Provider (singleton scope) | Pinia store |
| **Factory** | Manager/Driver, Model Factory | `useFactory` provider | `createApp/createRouter` |
| **Builder** | **Query Builder** | Swagger `DocumentBuilder`, TypeORM QB | `app.use().use().mount()` |
| **Decorator** | Middleware pipeline | **Interceptors** | `<Transition>`/HOC |
| **Facade** | **Facades** (`Cache::`,`DB::`) | Service layer, `HttpService` | Composable `useXxx()` |
| **Proxy** | Eloquent lazy loading | Guards (access control) | **`reactive()` = ES Proxy** |
| **Adapter** | **Flysystem** drivers | Express/Fastify adapter | Custom renderer |
| **Strategy** | Driver/guard qua config | **Passport Strategy** | validation rule, scrollBehavior |
| **Observer** | Events, **Model Observer** | EventEmitter, RxJS | **reactivity** (`watch`/`emit`) |
| **Command** | Artisan, **Job/Queue** | **CQRS** `CommandBus` | Vuex/Pinia actions |
| **Repository** | tự viết (Eloquent=Active Record) | **TypeORM `Repository<T>`** | service/composable bọc API |

- Các ví dụ **đúng tên pattern** dễ ghi điểm nhất: **NestJS Passport `Strategy`**, **NestJS CQRS
  `Command`**, **Laravel Model `Observer`**, **NestJS TypeORM `Repository`**, **Laravel `Facades`**.
- Ví dụ **"aha" nhất:** **Vue `reactive()` dùng `Proxy` của JS theo nghĩa đen** để chặn get/set →
  chính là Proxy Pattern ở tầng ngôn ngữ.
- Lưu ý sắc thái: **Laravel Facade ≠ GoF Facade** (là static proxy), **NestJS `@decorator` syntax ≠
  Decorator Pattern** (Interceptor mới đúng), **Laravel dùng Active Record** nên Repository phải tự viết.

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Phân biệt SOLID và Design Pattern khác nhau thế nào?

### 📖 Giới thiệu kiến thức

Đây là câu hỏi **phân biệt khái niệm** rất hay gặp, vì hai thứ này dễ bị nhầm là "cùng một loại".
Thực ra chúng ở **hai tầng khác nhau** và **bổ trợ cho nhau**:

- **SOLID là *nguyên lý* (principle)** — trả lời câu hỏi **"code nên được thiết kế thế nào để tốt?"**.
  Nó là **kim chỉ nam trừu tượng**, mang tính định hướng, *không* quy định cách code cụ thể.
- **Design Pattern là *giải pháp* (solution)** — trả lời câu hỏi **"giải quyết vấn đề thiết kế lặp
  lại này bằng cách nào?"**. Nó là **khuôn mẫu cụ thể** về cách bố trí class/object.

Nói ngắn gọn: **SOLID nói "nên thế nào", Design Pattern nói "làm thế nào"**.

**Mối quan hệ:** chúng **không đối lập** mà **đi cùng nhau** — phần lớn Design Pattern chính là
**cách hiện thực hóa các nguyên lý SOLID**. Ví dụ:
- **Strategy Pattern** hiện thực **OCP** (thêm thuật toán mới không sửa code cũ) và **DIP** (phụ
  thuộc interface `Strategy`).
- **Factory Pattern** hỗ trợ **DIP & OCP** (tách logic khởi tạo, thêm loại sản phẩm mới dễ dàng).
- **Observer Pattern** giúp **SRP & OCP** (tách phần "thông báo" khỏi phần "xử lý nghiệp vụ").

**Bảng so sánh:**

| Tiêu chí | **SOLID** | **Design Pattern** |
|----------|-----------|--------------------|
| **Bản chất** | Nguyên lý / nguyên tắc (principle) | Giải pháp mẫu (solution template) |
| **Trả lời câu hỏi** | Code *nên* thiết kế thế nào | *Làm thế nào* để giải vấn đề cụ thể |
| **Mức trừu tượng** | Trừu tượng, định hướng | Cụ thể, có cấu trúc class/object rõ ràng |
| **Số lượng** | 5 nguyên lý | 23 pattern GoF (+ nhiều pattern khác) |
| **Nguồn gốc** | Robert C. Martin (Uncle Bob) | GoF — Gang of Four (1994) |
| **Ví dụ** | "Phụ thuộc abstraction" (DIP) | "Dùng Factory để tạo object" |
| **Quan hệ** | *Mục tiêu* cần đạt | *Công cụ* để đạt mục tiêu (thường thỏa SOLID) |

**Một cách ví von dễ nhớ:** SOLID giống như **luật giao thông / nguyên tắc lái xe an toàn** (giữ
khoảng cách, không vượt ẩu), còn Design Pattern giống như **các kỹ thuật lái cụ thể đã được đúc kết**
(cách vào cua, cách đỗ xe song song). Bạn áp dụng kỹ thuật (pattern) để tuân thủ nguyên tắc (SOLID).

> Lưu ý: dùng Design Pattern **không tự động** đảm bảo code tốt — áp pattern sai chỗ vẫn vi phạm
> SOLID (vd: Singleton lạm dụng trở thành global state, phá vỡ DIP & khó test).

### 🎯 Tóm tắt trả lời phỏng vấn

- **SOLID là *nguyên lý* (nói "nên thế nào"), Design Pattern là *giải pháp cụ thể* (nói "làm thế nào")**
  — hai tầng khác nhau, **bổ trợ** chứ không đối lập.
- **SOLID**: 5 nguyên lý trừu tượng (Uncle Bob) định hướng *mục tiêu* thiết kế tốt.
  **Design Pattern**: 23 mẫu GoF cụ thể, là *công cụ* để đạt mục tiêu đó.
- Phần lớn Design Pattern **hiện thực hóa SOLID**: Strategy → OCP/DIP, Factory → DIP/OCP,
  Observer → SRP/OCP.
- Ví von: SOLID = *nguyên tắc lái xe an toàn*; Design Pattern = *kỹ thuật lái cụ thể* để tuân thủ.
- Lưu ý: dùng pattern **không tự động** = code tốt; áp sai chỗ (vd Singleton lạm dụng) vẫn phá SOLID.

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Architecture Pattern (Mẫu kiến trúc) là gì?

### 📖 Giới thiệu kiến thức

**Architecture Pattern (mẫu kiến trúc)** là **giải pháp tổng quát, đã được kiểm chứng** cho việc tổ
chức **cấu trúc tổng thể của cả hệ thống/ứng dụng** — quyết định ứng dụng được chia thành những phần
lớn nào, mỗi phần chịu trách nhiệm gì, và chúng giao tiếp với nhau ra sao.

**Khác Design Pattern ở "tầm" (scale):** đây là điểm interviewer hay so sánh — cùng là "pattern"
nhưng ở hai cấp độ khác nhau:

| | **Design Pattern** | **Architecture Pattern** |
|---|---|---|
| **Phạm vi** | Cấp **class/object** (vi mô) | Cấp **toàn hệ thống** (vĩ mô) |
| **Giải quyết** | Vấn đề thiết kế cục bộ trong code | Cách tổ chức & chia tách toàn ứng dụng |
| **Ví dụ** | Factory, Singleton, Observer | MVC, Layered, Microservices, Event-Driven |
| **Ảnh hưởng** | Một vài class | Toàn bộ codebase, deployment, team |
| **Chi phí thay đổi** | Thấp (refactor cục bộ) | Rất cao (ảnh hưởng cả hệ thống) |

**Các Architecture Pattern phổ biến:**

- **Layered (N-tier) — phân tầng:** chia ứng dụng thành các tầng chồng lên nhau, mỗi tầng chỉ gọi
  tầng ngay dưới: **Presentation → Business/Service → Data Access → Database**. Phổ biến, dễ hiểu,
  dễ test theo tầng; nhược điểm là dễ thành "khối nguyên" cứng nhắc nếu lạm dụng.

- **MVC (Model–View–Controller):** tách **Model** (dữ liệu/nghiệp vụ) — **View** (giao diện) —
  **Controller** (điều phối request, nối Model & View). Là nền tảng của Laravel, Spring, Rails...
  Giúp tách *logic* khỏi *hiển thị*. *(Biến thể: MVP, MVVM — MVVM là nền của Vue/Angular; chi tiết ở [câu 4.3](#q4-3).)*

- **Microservices:** chia ứng dụng thành **nhiều service nhỏ, độc lập**, mỗi service lo một nghiệp
  vụ (orders, payments, users), deploy riêng, giao tiếp qua API/message queue. Ưu: scale & deploy
  độc lập, đội nhỏ tự chủ. Nhược: phức tạp vận hành (network, distributed transaction, monitoring).
  *Đối nghịch với **Monolith** (gói tất cả trong một ứng dụng).*

- **Event-Driven:** các thành phần giao tiếp qua **sự kiện** (event) bất đồng bộ qua message
  broker (Kafka, RabbitMQ). Lỏng lẻo phụ thuộc cao (loose coupling), chịu tải tốt; khó debug luồng.

- **Client–Server, Hexagonal (Ports & Adapters), CQRS...** là các mẫu khác tùy bài toán.

**Quan hệ với Design Pattern & SOLID:** Architecture Pattern dựng *khung lớn*; bên trong mỗi thành
phần ta vẫn dùng **Design Pattern** ([câu 2](#q2)) và tuân **SOLID** ([câu 1](#q1)). Chúng bổ sung
cho nhau ở các cấp độ khác nhau (xem thêm phân biệt ở [câu 3](#q3)).

**Lưu ý:** không có kiến trúc "tốt nhất" tuyệt đối — chọn theo **bối cảnh** (quy mô team, yêu cầu
scale, độ phức tạp nghiệp vụ). Nhảy thẳng lên Microservices cho một app nhỏ là **over-engineering**;
ngược lại Monolith quá tải khi hệ thống lớn cũng là vấn đề. Nhiều team bắt đầu **Monolith** rồi tách
dần khi cần.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Architecture Pattern** là mẫu tổ chức **cấu trúc tổng thể của cả hệ thống**: chia ứng dụng thành
  các phần lớn, trách nhiệm mỗi phần và cách chúng giao tiếp.
- **Khác Design Pattern ở tầm:** Design Pattern ở cấp **class/object** (Factory, Singleton);
  Architecture Pattern ở cấp **toàn hệ thống** (MVC, Layered, Microservices, Event-Driven).
- Phổ biến: **Layered** (phân tầng), **MVC** (Model–View–Controller, nền của Laravel), **Microservices**
  (nhiều service độc lập, đối nghịch Monolith), **Event-Driven** (giao tiếp qua event bất đồng bộ).
- **Bổ trợ nhau:** Architecture dựng khung lớn; bên trong vẫn dùng Design Pattern & tuân SOLID.
- **Chọn theo bối cảnh**, không có kiến trúc tốt nhất tuyệt đối — tránh over-engineering (đừng vội
  Microservices cho app nhỏ); nhiều team bắt đầu Monolith rồi tách dần.

[↑ Mục lục](#mục-lục)

<a id="q4-1"></a>
### 4.1. Microservice là gì?

#### 📖 Giới thiệu kiến thức

**Microservices** là một **kiểu Architecture Pattern** ([câu 4](#q4)) trong đó ứng dụng được chia thành
**một tập hợp các service nhỏ, tự chủ (autonomous)** — thay vì gói tất cả trong một khối (**Monolith**).
Mỗi microservice:
- Lo đúng **một nghiệp vụ (business capability / bounded context)** — vd `Orders`, `Payments`, `Users`.
- Chạy trong **process riêng**, được **deploy độc lập**.
- **Thường có database riêng** (decentralized data — mỗi service tự quản dữ liệu của mình).
- Giao tiếp với service khác **qua network** bằng **API nhẹ (REST/gRPC)** hoặc **message/event**
  (Kafka, RabbitMQ).
- **Tự chủ công nghệ (polyglot):** mỗi service có thể chọn ngôn ngữ/DB riêng phù hợp.

**6 đặc trưng cốt lõi (theo Martin Fowler / Sam Newman — hay được hỏi):**

1. **Tổ chức quanh nghiệp vụ (business capability), không theo tầng kỹ thuật** — mỗi service là một
   "lát dọc" đầy đủ (từ API xuống DB) cho một miền nghiệp vụ.
2. **Independently deployable** — sửa & deploy một service **không cần build lại cả hệ thống**.
3. **Loose coupling + high cohesion** — service ít phụ thuộc nhau, mỗi cái gắn kết chặt bên trong.
4. **Decentralized (phi tập trung)** — dữ liệu (database per service) và cả cách quản trị đều phân tán.
5. **Fault isolation (cô lập lỗi)** — một service chết **không nhất thiết** kéo sập cả hệ thống.
6. **Autonomous team** — mỗi đội "sở hữu" trọn một service (đúng tinh thần **Conway's Law**).

**Hạ tầng đi kèm (vì hệ phân tán):** một hệ microservices thực tế cần thêm nhiều thành phần mà Monolith
không cần:

```
                 ┌───────────────┐
   Client  ───▶  │  API Gateway  │  (điểm vào duy nhất, định tuyến, auth, rate-limit)
                 └──────┬────────┘
          ┌────────────┼──────────────┐
      ┌───▼───┐    ┌───▼────┐    ┌────▼────┐
      │ Users │    │ Orders │    │ Payment │   ← mỗi service + DB riêng
      │  svc  │    │  svc   │    │   svc   │
      └───┬───┘    └───┬────┘    └────┬────┘
          └─────▶ Message Broker (Kafka/RabbitMQ) ◀─────┘   (giao tiếp bất đồng bộ)
   + Service Discovery · Config Server · Load Balancer · Monitoring/Tracing · Docker/K8s
```

**Hai kiểu giao tiếp:**
- **Đồng bộ (synchronous):** service gọi trực tiếp service khác qua **REST/gRPC** và **chờ kết quả**
  (request–response).
- **Bất đồng bộ (asynchronous):** service **phát event** lên message broker, không chờ — hợp mô hình
  **event-driven**, giảm coupling và chịu tải tốt.

**Ưu điểm:** scale & deploy **độc lập từng service**, đội nhỏ tự chủ, **cô lập lỗi**, tự do công nghệ.

**Thách thức (đánh đổi lớn):**
- **Phức tạp phân tán:** network latency, **partial failure** (một service lỗi), cần retry/circuit breaker.
- **Nhất quán dữ liệu khó:** không còn ACID xuyên service → phải dùng **Saga / eventual consistency**.
- **Vận hành nặng:** cần **DevOps/monitoring/distributed tracing**, orchestration (K8s) → chi phí hạ tầng lớn.
- **Testing & debug khó** hơn (luồng đi qua nhiều service).

**Khi nào dùng:** hệ thống **lớn, nhiều team, cần scale/deploy độc lập**. Với app nhỏ → **over-engineering**;
nên bắt đầu **Monolith / modular monolith** rồi tách dần (xem so sánh chi tiết ở [câu 4.2](#q4-2)). Về
**triển khai cụ thể trong NestJS** (transporter, `@MessagePattern`/`@EventPattern`), xem
[NestJS — Microservices](../Backend/nestjs.md#q8).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Microservices** = kiểu kiến trúc chia ứng dụng thành **nhiều service nhỏ, tự chủ**, mỗi service lo
  **một nghiệp vụ (bounded context)**, **deploy độc lập**, **DB riêng**, giao tiếp qua **API (REST/gRPC)
  hoặc message/event** — đối nghịch với **Monolith** (một khối).
- **Đặc trưng cốt lõi:** tổ chức quanh **nghiệp vụ**, **independently deployable**, **loose coupling**,
  **decentralized (DB per service)**, **cô lập lỗi**, **team tự chủ**.
- **Giao tiếp:** **đồng bộ** (REST/gRPC, chờ kết quả) hoặc **bất đồng bộ** (event qua Kafka/RabbitMQ).
- **Ưu:** scale/deploy độc lập, cô lập lỗi, tự do công nghệ. **Nhược:** **phức tạp phân tán** (partial
  failure), **nhất quán dữ liệu khó** (Saga, eventual consistency), **vận hành/DevOps nặng**.
- **Chọn khi:** hệ thống lớn, nhiều team; app nhỏ thì là over-engineering → **Monolith First** rồi tách dần.

[↑ Mục lục](#mục-lục)

<a id="q4-2"></a>
### 4.2. Phân biệt Monolith với Microservices

#### 📖 Giới thiệu kiến thức

Đây là hai cách **đóng gói & triển khai (deployment) toàn hệ thống** — khác với Layered/Clean (cách
tổ chức code *bên trong* một ứng dụng). Câu hỏi kinh điển khi phỏng vấn về scale & vận hành.

**Monolith (kiến trúc nguyên khối):** **toàn bộ ứng dụng** (UI, business logic, data access của mọi
nghiệp vụ — users, orders, payments...) được build và **deploy như một đơn vị duy nhất** (một
codebase, một process, thường một database). Đây là cách mặc định của Laravel, Rails, Spring MVC.

```
MONOLITH                         MICROSERVICES
┌───────────────────────┐        ┌────────┐ ┌────────┐ ┌─────────┐
│  1 ứng dụng / 1 deploy │        │ Users  │ │ Orders │ │ Payment │  ← deploy riêng
│  Users · Orders · Pay │        │ svc    │ │ svc    │ │ svc     │
│  ───────────────────  │        │  DB    │ │  DB    │ │  DB     │  ← DB riêng
│      1 Database       │        └────────┘ └────────┘ └─────────┘
└───────────────────────┘            └── giao tiếp qua API / message queue ──┘
```

**Microservices:** chia hệ thống thành **nhiều service nhỏ, độc lập**, mỗi service lo **một nghiệp vụ
(bounded context)**, **deploy riêng**, **thường có database riêng**, giao tiếp qua **API (REST/gRPC)
hoặc message queue** (Kafka/RabbitMQ).

**So sánh theo các tiêu chí quan trọng:**

| Tiêu chí | **Monolith** | **Microservices** |
|---|---|---|
| **Đơn vị deploy** | Một khối duy nhất | Mỗi service deploy độc lập |
| **Codebase** | Một repo/codebase | Nhiều codebase (mỗi service) |
| **Database** | Thường dùng chung 1 DB | Mỗi service một DB (database per service) |
| **Scale** | Scale **cả khối** (nhân bản nguyên app) | Scale **từng service** theo tải riêng |
| **Giao tiếp nội bộ** | Gọi hàm trong process (nhanh, đơn giản) | Gọi qua network (API/queue) → có latency, có thể lỗi |
| **Triển khai/CI-CD** | Đơn giản (1 pipeline) | Phức tạp (nhiều pipeline, orchestration) |
| **Công nghệ** | Đồng nhất một stack | Mỗi service chọn stack riêng (polyglot) |
| **Lỗi lan truyền** | Một lỗi nặng có thể sập cả app | Cô lập tốt hơn (1 service lỗi không nhất thiết sập cả hệ thống) |
| **Transaction** | ACID dễ (cùng 1 DB) | Khó — phải dùng **distributed transaction / Saga** |
| **Phù hợp** | App vừa & nhỏ, team nhỏ, MVP/startup | Hệ thống lớn, nhiều team, cần scale & deploy độc lập |

**Ưu/nhược cốt lõi:**
- **Monolith — ưu:** đơn giản phát triển/test/deploy, gọi nội bộ nhanh, transaction dễ, ít chi phí hạ
  tầng. **Nhược:** khi lớn → codebase cồng kềnh, build/deploy chậm, một thay đổi nhỏ phải deploy cả
  khối, khó scale riêng phần nóng, công nghệ bị "khóa".
- **Microservices — ưu:** scale & deploy độc lập, đội nhỏ tự chủ, cô lập lỗi, tự do công nghệ.
  **Nhược:** **phức tạp vận hành** (network, service discovery, monitoring, distributed tracing),
  **nhất quán dữ liệu khó** (eventual consistency, Saga), tốn chi phí hạ tầng & DevOps.

**Lưu ý thực tế (rất hay được hỏi):** **đừng bắt đầu bằng Microservices**. Lời khuyên phổ biến là
**"Monolith First"** (Martin Fowler) — bắt đầu bằng monolith (hoặc **modular monolith**: monolith chia
module rõ ràng), khi nghiệp vụ & team đủ lớn và đã hiểu rõ ranh giới mới tách dần thành microservices.
Nhảy thẳng vào microservices cho app nhỏ là **over-engineering**: gánh toàn bộ chi phí phân tán mà
chưa cần. Microservices là vấn đề **tổ chức & vận hành** nhiều như kỹ thuật (Conway's Law).

> **Quan hệ với Layered/Clean ([câu 5](#q5), [câu 6](#q6)):** Monolith vs Microservices là *cách chia
> & deploy hệ thống ở tầm vĩ mô*; bên trong **mỗi microservice** (hoặc mỗi module của modular monolith)
> vẫn tổ chức code theo **Layered hoặc Clean Architecture**. Hai trục này độc lập nhau.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Monolith:** toàn bộ ứng dụng là **một khối, deploy một lần, thường chung 1 DB**. **Microservices:**
  **nhiều service nhỏ độc lập**, mỗi service một nghiệp vụ, **deploy riêng, DB riêng**, giao tiếp qua
  **API/message queue**.
- **Khác biệt cốt lõi:** đơn vị **deploy & scale** (cả khối vs từng service) và **giao tiếp** (gọi
  hàm in-process vs qua network).
- **Monolith** đơn giản, nhanh, transaction dễ → hợp app vừa/nhỏ & team nhỏ; nhược: cồng kềnh, khó
  scale riêng khi lớn. **Microservices** scale/deploy độc lập, cô lập lỗi, tự do công nghệ; nhược:
  **phức tạp vận hành & nhất quán dữ liệu** (Saga, eventual consistency).
- **Lời khuyên:** **"Monolith First"** — bắt đầu monolith/modular monolith rồi tách dần khi cần; đi
  thẳng microservices cho app nhỏ là over-engineering.
- Hai trục độc lập: **bên trong mỗi microservice vẫn dùng Layered/Clean** ([câu 5](#q5), [câu 6](#q6)).

[↑ Mục lục](#mục-lục)

<a id="q4-3"></a>
### 4.3. Các mô hình khác ngoài MVC (MVP, MVVM, Flux/Redux)

#### 📖 Giới thiệu kiến thức

**MVC không phải mô hình duy nhất.** Nó thuộc **họ mẫu kiến trúc tầng trình bày (presentation
architecture)** — tất cả cùng mục tiêu **tách logic khỏi giao diện (UI)**, chỉ khác nhau ở **ai đứng
giữa Model và View** và **cách View được cập nhật**. Hỏi "mô hình khác ngoài MVC" thường ý là các anh
em cùng họ này (MVP, MVVM), đôi khi là mô hình quản lý state (Flux/Redux).

**Nhắc lại MVC (Model–View–Controller):** **Controller** nhận request, thao tác **Model**, rồi chọn
**View** để render; **View có thể đọc thẳng Model**. Điển hình **server-side**: Laravel, Rails, Spring
MVC, ASP.NET MVC. Nhược: khi UI phức tạp, Controller dễ "phình" và View–Model còn dính nhau.

**1) MVP (Model–View–Presenter):** thay Controller bằng **Presenter**. Điểm khác cốt lõi: **View thụ
động (passive view)** — không chứa logic, chỉ hiển thị và **ủy quyền mọi sự kiện cho Presenter** qua
một **interface**. Presenter giữ **toàn bộ UI logic**, cập nhật Model và **đẩy dữ liệu ngược lại View**.
View và Model **không biết nhau**. Quan hệ **View–Presenter thường 1:1**. → **Rất dễ test** (Presenter
độc lập UI). Điển hình: **Android đời cũ, WinForms, GWT**.

**2) MVVM (Model–View–ViewModel):** thay bằng **ViewModel** — nơi phơi ra **dữ liệu + command** cho
View. Điểm khác cốt lõi: **data binding (thường 2 chiều)** — View **tự động** cập nhật khi ViewModel
đổi và ngược lại, **không cần code thủ công** đẩy dữ liệu như MVP. **ViewModel không tham chiếu View**
(một ViewModel phục vụ được nhiều View). Điển hình: **Vue, Angular, WPF/Knockout**. *(React thường
được xếp vào nhóm **một chiều/Flux** hơn là MVVM thuần.)*

**3) Ngoài họ MVC — luồng dữ liệu một chiều (Flux/Redux):** không có "controller/presenter", mà theo
**unidirectional data flow**: **Action → Store (reducer) → View → Action...**. Một **store tập trung**,
state **immutable**, cập nhật bằng **hàm thuần (reducer)** → **dễ trace/debug/time-travel**. Là mô hình
**quản lý state** cho UI phản ứng, phổ biến với **React (Redux/Zustand)**; **Vuex/Pinia** cũng lấy cảm
hứng từ đây. Biến thể liên quan: **MVI** (Model–View–Intent, reactive — Android/Kotlin).

**So sánh nhanh MVC vs MVP vs MVVM:**

| Tiêu chí | **MVC** | **MVP** | **MVVM** |
|---|---|---|---|
| **Trung gian** | Controller | Presenter | ViewModel |
| **View ↔ Model** | View **đọc được** Model | **Tách hẳn** (qua Presenter) | **Tách hẳn** (qua binding) |
| **Cập nhật View** | Controller chọn View | Presenter **đẩy** vào View (qua interface) | **Data binding** tự động (2 chiều) |
| **UI logic nằm ở** | Controller (+ chút ở View) | Presenter (**View thụ động**) | ViewModel |
| **Quan hệ với View** | 1 Controller : nhiều View | **1 : 1** | 1 ViewModel : nhiều View |
| **Khả năng test** | Trung bình | **Cao** | **Cao** |
| **Điển hình** | Laravel, Rails, Spring MVC | Android cũ, WinForms | Vue, Angular, WPF |

> **Lưu ý phân tầng câu hỏi:** MVC/MVP/MVVM/Flux ở **tầng trình bày (một ứng dụng)**. Nếu interviewer
> hỏi "mô hình khác" ở **tầng hệ thống** thì đó là **Layered** ([câu 5](#q5)), **Clean** ([câu 6](#q6)),
> **Hexagonal, Microservices** ([câu 4.1](#q4-1)), **Event-Driven, CQRS** — xem [câu 4](#q4). Nên hỏi lại
> "anh/chị muốn ở tầng UI hay tầng kiến trúc hệ thống?" để trả lời đúng trọng tâm.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Họ MVC cùng mục tiêu **tách UI khỏi logic**, khác ở **trung gian** & **cách View cập nhật**:
  - **MVC:** **Controller** điều phối, **View đọc Model** — server-side (**Laravel/Rails/Spring**).
  - **MVP:** **Presenter** giữ toàn bộ UI logic, **View thụ động**, View–Presenter **1:1**, **dễ test** —
    Android cũ/WinForms.
  - **MVVM:** **ViewModel + data binding 2 chiều**, View tự cập nhật, ViewModel **không biết View** —
    **Vue/Angular/WPF**.
- **Ngoài họ MVC:** **Flux/Redux** — **luồng dữ liệu một chiều**, **store tập trung**, reducer thuần →
  dễ debug (React; Vuex/Pinia lấy cảm hứng). Thêm **MVI**.
- **Ở tầng hệ thống** (nếu hỏi rộng hơn): **Layered, Clean, Hexagonal, Microservices, Event-Driven,
  CQRS** — xem [câu 4](#q4)–[câu 6](#q6).
- **Chốt:** không có mô hình tốt nhất tuyệt đối — chọn theo nền tảng/framework (server-side → MVC;
  UI có binding → MVVM; UI phản ứng phức tạp → Flux/Redux).

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Layered Architecture (Kiến trúc phân tầng) là gì?

### 📖 Giới thiệu kiến thức

**Layered Architecture (kiến trúc phân tầng / N-tier)** là **mẫu kiến trúc phổ biến và lâu đời nhất**:
chia ứng dụng thành các **tầng (layer) chồng lên nhau theo chiều dọc**, mỗi tầng có một trách nhiệm
rõ ràng và **chỉ giao tiếp với tầng ngay dưới nó** (quy tắc *layers of isolation*). Đây là kiểu kiến
trúc mặc định mà hầu hết framework MVC (Laravel, Spring, .NET MVC) hướng tới.

**Các tầng kinh điển (từ trên xuống):**

```
┌─────────────────────────────────────────────┐
│  Presentation Layer                          │  ← UI / Controller / API endpoint
│  (nhận request, trả response, validate input)│     — không chứa business logic
├─────────────────────────────────────────────┤
│  Business / Service Layer                    │  ← nghiệp vụ, rule, điều phối use case
├─────────────────────────────────────────────┤
│  Data Access / Persistence Layer             │  ← Repository / DAO / ORM, truy vấn DB
├─────────────────────────────────────────────┤
│  Database Layer                              │  ← MySQL / PostgreSQL / ...
└─────────────────────────────────────────────┘
   Phụ thuộc: mỗi tầng gọi XUỐNG tầng ngay dưới ──▼
```

1. **Presentation:** giao diện / controller — nhận input, validate, format output. Không chứa nghiệp vụ.
2. **Business (Service):** chứa **business logic** — quy tắc nghiệp vụ, tính toán, điều phối.
3. **Data Access (Persistence):** đóng gói cách truy cập dữ liệu (Repository/DAO/ORM), giấu chi tiết SQL.
4. **Database:** nơi lưu dữ liệu thật.

**Đặc trưng quan trọng — chiều phụ thuộc:** trong Layered, **phụ thuộc đi từ trên xuống dưới**
(Presentation → Business → Data Access → Database). Tầng trên biết và gọi tầng dưới; tầng dưới
**không biết** tầng trên. Hệ quả: **tầng Business phụ thuộc trực tiếp vào tầng Data Access** (và gián
tiếp vào DB) — đây chính là điểm khác cốt lõi với Clean Architecture (xem [câu 6.2](#q6-2)).

**Closed layer vs Open layer:** mặc định các tầng là **"closed"** — request **phải đi qua tuần tự**
từng tầng, không được nhảy cóc (Presentation không gọi thẳng Data Access). Điều này giúp **cô lập thay
đổi** (sửa một tầng ít ảnh hưởng tầng khác) nhưng có thể gây *"sinkhole anti-pattern"*: nhiều request
chỉ đi xuyên qua tầng giữa mà không xử lý gì. Khi cần, có thể mở một tầng thành **"open"** để cho phép
bỏ qua.

**Ưu điểm:**
- **Đơn giản, dễ hiểu, quen thuộc** — gần như mọi dev đều biết; là lựa chọn mặc định an toàn.
- **Tách trách nhiệm rõ theo tầng kỹ thuật** (UI / nghiệp vụ / dữ liệu), dễ phân công.
- **Dễ bắt đầu**, ít boilerplate, hợp với phần lớn ứng dụng CRUD và monolith.

**Nhược điểm:**
- **Coupling theo chiều dọc:** nghiệp vụ (Business) **phụ thuộc trực tiếp** vào hạ tầng (Data
  Access/ORM) → muốn đổi DB hay test nghiệp vụ mà không đụng DB **khó hơn**.
- Dễ thành **"khối nguyên" cứng nhắc** nếu hệ thống lớn; thay đổi xuyên tầng phải sửa nhiều nơi.
- Tầng giữa dễ thành **sinkhole** (chỉ pass-through, không thêm giá trị).

**Ví dụ minh họa (order API) — code & cấu trúc thư mục:**

Thư mục chia theo **tầng kỹ thuật**; luồng đi thẳng một chiều xuống DB:

```
src/                                     Flow:
├── controllers/OrderController.ts        HTTP Request
├── services/OrderService.ts        →       ↓ OrderController → OrderService
├── repositories/OrderRepository.ts →       ↓ OrderRepository → MySQL
└── models/Order.ts
```

```ts
class OrderService {
  constructor(private orderRepository: OrderRepository) {}
  async createOrder(data: CreateOrderRequest) {
    return this.orderRepository.create(data);      // service điều phối xuống repository
  }
}
class OrderRepository {                             // concrete impl — BIẾT Prisma
  async create(data: CreateOrderRequest) {
    return prisma.order.create({ data });
  }
}
```

**Failure mode hay gặp (phải cảnh giác):** nếu chỉ "chia folder theo tầng" mà **không kiểm soát chiều
phụ thuộc**, `OrderService` dần **phình ra**, tự gọi thẳng đủ loại hạ tầng:

```ts
class OrderService {
  async createOrder(data) {
    await prisma.order.findFirst(...);             // DB
    await redis.set(...);                          // cache
    await stripe.paymentIntents.create(...);       // payment
    await sendEmail(...);                          // email / AWS
  }
}
```

→ business logic **trộn lẫn** infrastructure (Prisma/Redis/Stripe/Email) → khó test, khó đổi. Đây **không
phải lỗi bắt buộc** của Layered mà là hệ quả khi **thiếu kỷ luật về hướng phụ thuộc** — Clean chữa đúng
điểm này bằng cách **đảo hướng** (xem [câu 6](#q6) & [câu 6.2](#q6-2); cách refactor chính đoạn code này ở [câu 6.3](#q6-3)).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Layered Architecture** là mẫu kiến trúc phổ biến nhất: chia app thành các **tầng ngang chồng lên
  nhau** — **Presentation → Business/Service → Data Access → Database** — mỗi tầng chỉ gọi tầng dưới.
- **Chiều phụ thuộc đi từ trên xuống:** tầng trên gọi tầng dưới; đặc biệt **Business phụ thuộc trực
  tiếp vào Data Access/DB** (đây là điểm khác cốt lõi với Clean Architecture).
- Tầng thường **"closed"** (request đi tuần tự, không nhảy cóc) → cô lập thay đổi nhưng dễ dính
  *sinkhole* (tầng giữa chỉ pass-through).
- **Ưu:** đơn giản, quen thuộc, ít boilerplate, hợp CRUD/monolith. **Nhược:** nghiệp vụ coupling với hạ
  tầng → khó đổi DB / khó test lõi độc lập, dễ cứng nhắc khi lớn.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Clean Architecture là gì?

### 📖 Giới thiệu kiến thức

**Clean Architecture** là một mẫu kiến trúc do **Robert C. Martin (Uncle Bob)** đề xuất, tổ chức code
thành các **tầng đồng tâm (concentric layers)** với **một quy tắc cốt lõi duy nhất — Dependency Rule:
mọi phụ thuộc chỉ được hướng vào trong** (tầng ngoài biết tầng trong, tầng trong **không biết gì** về
tầng ngoài). Mục tiêu: **tách nghiệp vụ cốt lõi khỏi framework, database, UI, dịch vụ ngoài** — để
business logic **độc lập** và **test được mà không cần** DB/web thật.

Nó là phiên bản tổng quát của **Hexagonal (Ports & Adapters)** và **Onion Architecture** — cùng ý
tưởng "lõi nghiệp vụ ở giữa, chi tiết kỹ thuật ở ngoài", khác tên gọi. Clean Architecture chính là
**SOLID áp ở cấp kiến trúc**, đặc biệt là **DIP** (xem [câu 1](#q1)).

**4 tầng (từ trong ra ngoài):**

```
┌─────────────────────────────────────────────┐
│  Frameworks & Drivers  (DB, Web, UI, ngoài)  │  ← dễ thay đổi nhất
│  ┌───────────────────────────────────────┐  │
│  │  Interface Adapters                   │  │  ← Controller, Presenter,
│  │  (Controller / Gateway / Repository)  │  │     Repository implementation
│  │  ┌─────────────────────────────────┐  │  │
│  │  │  Use Cases (Application logic)  │  │  │  ← điều phối nghiệp vụ ứng dụng
│  │  │  ┌───────────────────────────┐  │  │  │
│  │  │  │  Entities (Domain rules)  │  │  │  │  ← lõi, ổn định nhất
│  │  │  └───────────────────────────┘  │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
   Dependency Rule: mũi tên phụ thuộc chỉ hướng VÀO TRONG ──►
```

1. **Entities (Enterprise Business Rules):** object & quy tắc nghiệp vụ cốt lõi nhất, **ổn định nhất**,
   không phụ thuộc bất cứ thứ gì bên ngoài.
2. **Use Cases (Application Business Rules):** logic đặc thù của ứng dụng, **điều phối** entities để
   thực hiện một tác vụ (vd "đăng ký user", "đặt hàng").
3. **Interface Adapters:** chuyển đổi dữ liệu giữa use case và thế giới ngoài — Controller, Presenter,
   **implementation của Repository/Gateway** (xem [Repository câu 2.11](#q2-11), [Adapter câu 2.7](#q2-7)).
4. **Frameworks & Drivers:** Laravel/NestJS, DB (MySQL/Mongo), web, UI, API ngoài — **dễ thay đổi
   nhất**, đẩy ra rìa.

**Dependency Rule & cách "đảo ngược":** tầng trong **không được** import tầng ngoài. Nhưng use case
vẫn cần lưu DB? → use case **định nghĩa interface (port)** mà nó cần (`UserRepository`), tầng ngoài
**cài đặt (adapter)** interface đó. Nhờ **DIP**, mũi tên phụ thuộc bị đảo: chi tiết (DB) phụ thuộc vào
abstraction (interface) của nghiệp vụ, không ngược lại.

```
Use Case  ──gọi──►  interface UserRepository   (nằm ở tầng trong)
                          ▲ implements
EloquentUserRepository ───┘                     (nằm ở tầng ngoài, phụ thuộc VÀO TRONG)
```

**Lợi ích:**
- **Độc lập framework/DB/UI:** đổi MySQL → Mongo, web → CLI, Laravel → framework khác **không đụng**
  lõi nghiệp vụ.
- **Dễ test:** unit-test use case bằng cách mock các port, không cần DB/HTTP thật.
- **Ranh giới rõ ràng**, dễ bảo trì khi hệ thống lớn & phức tạp nghiệp vụ.

**Tư duy cốt lõi (khác Layered):** Layered hay hỏi *"code này thuộc tầng nào — controller/service/repo?"*.
Clean hỏi trước hết: *"**Business rule** là gì, và làm sao để **database/framework không chi phối** nó?"*.
Vì thế use case được viết để **đọc lên là thấy nghiệp vụ**, không phải thấy thao tác kỹ thuật:

```ts
class CreateOrderUseCase {
  async execute(input: CreateOrderInput) {
    const customer = await this.customerRepository.findById(input.customerId);
    if (!customer) throw new CustomerNotFoundError();     // rule: customer phải tồn tại

    const order = Order.create(customer);
    for (const item of input.items) {
      const product = await this.productRepository.findById(item.productId);
      order.addItem(product, item.quantity);              // rule: product active, đơn có item
    }
    order.applyDiscount();                                // rule nghiệp vụ (tính total, discount)

    await this.orderRepository.save(order);               // lưu qua PORT — không biết DB nào
  }
}
```

Đọc code thấy ngay ý định **"CREATE ORDER"**, chứ không phải "query DB → map model → gọi ORM → insert".
Mọi truy cập hạ tầng đều qua **port** (`customerRepository`, `orderRepository`) nên use case **không biết**
Prisma/MySQL/Mongo là gì — đó là biểu hiện của Dependency Rule ở cấp nghiệp vụ.

**Lưu ý (rất quan trọng):** Clean Architecture **đánh đổi bằng nhiều boilerplate** — nhiều tầng, nhiều
interface, nhiều mapping (entity ↔ model ↔ DTO). Với **app CRUD đơn giản**, áp Clean Architecture là
**over-engineering** (vi phạm KISS/YAGNI) — dùng thẳng MVC + Eloquent gọn hơn. Nó **đáng giá** khi
**nghiệp vụ phức tạp, vòng đời dài, nhiều người làm**, cần thay đổi/ test độc lập.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Clean Architecture** (Uncle Bob) tổ chức code thành **tầng đồng tâm** với **Dependency Rule: phụ
  thuộc chỉ hướng vào trong** — lõi nghiệp vụ **không biết** gì về framework/DB/UI.
- **4 tầng:** **Entities** (domain rules, lõi) → **Use Cases** (application logic) → **Interface
  Adapters** (controller, repository impl) → **Frameworks & Drivers** (DB, web, UI — rìa ngoài).
- **Cách hoạt động:** tầng trong định nghĩa **interface (port)**, tầng ngoài **cài đặt (adapter)** →
  nhờ **DIP** mà chi tiết phụ thuộc nghiệp vụ, không ngược lại. Cùng họ với **Hexagonal/Onion**.
- **Lợi ích:** độc lập framework/DB/UI, **dễ test** (mock port), ranh giới rõ — chính là **SOLID ở
  cấp kiến trúc**.
- **Lưu ý:** nhiều boilerplate/mapping → **over-engineering** cho app CRUD đơn giản; chỉ đáng khi
  nghiệp vụ phức tạp, dài hạn, nhiều người.

[↑ Mục lục](#mục-lục)

<a id="q6-1"></a>
### 6.1. Clean Architecture thường được triển khai ở đâu?

#### 📖 Giới thiệu kiến thức

Clean Architecture **không dành cho mọi dự án** — nó đáng giá ở những nơi mà **lợi ích "độc lập &
dễ test" vượt qua chi phí boilerplate**. Có thể nhìn "ở đâu" theo 3 góc: **loại dự án nào**, **ngôn
ngữ/framework nào hay dùng**, và **vị trí nào trong kiến trúc tổng thể**.

**1. Loại dự án / hệ thống phù hợp:**
- **Nghiệp vụ phức tạp, vòng đời dài (enterprise):** ngân hàng/fintech, bảo hiểm, ERP, healthcare,
  logistics, e-commerce quy mô lớn — nơi quy tắc nghiệp vụ nhiều, thay đổi thường xuyên và phải sống
  nhiều năm.
- **Dự án theo DDD (Domain-Driven Design):** Clean/Hexagonal là **khung triển khai tự nhiên** cho DDD
  (domain ở lõi, hạ tầng ở rìa).
- **Team lớn, nhiều người, yêu cầu test cao:** ranh giới rõ giúp chia việc và viết unit test cho lõi
  mà không cần DB/HTTP.
- **Hệ thống cần "sống lâu hơn công nghệ":** dự kiến đổi DB, đổi framework, thêm kênh giao tiếp (web +
  mobile + CLI + queue) → giữ lõi không phụ thuộc chi tiết.

**2. Ngôn ngữ / framework hay áp dụng:**
- **.NET (C#) / ASP.NET Core:** rất phổ biến — có template Clean Architecture nổi tiếng (Jason Taylor).
- **Java / Spring Boot:** Hexagonal/Clean phổ biến trong enterprise.
- **NestJS (Node):** cấu trúc module + DI + provider khiến Clean/Hexagonal **dễ áp** (tách
  domain/application/infrastructure). *(So sánh framework xem [câu 2.12](#q2-12).)*
- **Mobile — Flutter, Android (Kotlin), iOS (Swift):** Clean Architecture **cực kỳ phổ biến** ở mobile,
  chia rõ `presentation / domain / data`.
- **Go, Python (FastAPI):** nhiều backend service dùng.
- **Laravel (PHP):** **làm được nhưng phải tự dựng** — tách domain/use case khỏi Eloquent, dùng
  **Repository interface** ([câu 2.11](#q2-11)) + bind ở Service Provider (DIP — [câu 1](#q1)). Vì
  Laravel mặc định là MVC + Active Record nên thường **chỉ dự án lớn** mới áp Clean Architecture đầy đủ.

**3. Vị trí trong kiến trúc tổng thể:**
- **Bên trong mỗi Microservice:** vĩ mô là Microservices ([câu 4](#q4)), nhưng **vi mô mỗi service**
  thường tổ chức theo Clean/Hexagonal → mỗi service có lõi domain riêng, độc lập hạ tầng.
- Cũng áp được trong **Monolith** (modular monolith): mỗi module một lõi nghiệp vụ sạch.

**Nơi KHÔNG nên dùng:** prototype/MVP, app CRUD nhỏ, startup cần ship nhanh, admin tool đơn giản →
MVC + Active Record (Eloquent/TypeORM) gọn hơn nhiều; áp Clean Architecture lúc này là over-engineering.

**Quy tắc chọn nhanh (rất hay hỏi):**
- **CRUD/admin đơn giản** (User, Product, Category, Banner, Setting... với Create/Update/Delete/List):
  chỉ cần **`Controller → Service → Repository`** (Layered). Đừng dựng cả
  `CreateBannerUseCase + BannerDomainService + BannerRepositoryPort + BannerRepositoryAdapter +
  BannerFactory + BannerMapper` chỉ cho một `POST /banners` → đó là **over-engineering**.
- **Hệ thống nhiều rule nghiệp vụ** (workflow & state transition, permission theo organization,
  retry/checkpoint, append-dedup, orchestration pipeline...): thứ cần **bảo vệ** là **business rule**,
  không phải framework/DB → nghiêng mạnh về **Clean / Hexagonal**. Câu chốt: *bảo vệ business rule,
  đừng để framework (FastAPI/LangGraph...) hay DB (PostgreSQL/Redis...) chi phối nó.*

**Biểu hiện cấu trúc thư mục thường gặp** (vd backend):

```
src/
 ├── domain/         # Entities + business rules thuần (không import framework)
 ├── application/    # Use cases + interface (port): UserRepository, ...
 ├── infrastructure/ # Adapter: EloquentUserRepository, API client, DB config
 └── presentation/   # Controller / HTTP / CLI (gọi use case)
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Phù hợp khi:** nghiệp vụ **phức tạp & vòng đời dài** (fintech, ERP, bảo hiểm, e-commerce lớn), dự
  án theo **DDD**, team lớn, yêu cầu test cao, cần độc lập framework/DB/UI.
- **Ngôn ngữ/FW hay dùng:** **.NET**, **Spring (Java)**, **NestJS**, **mobile (Flutter/Android/iOS)**,
  Go; **Laravel làm được nhưng phải tự dựng** (Repository interface + DIP) vì mặc định là MVC + Active Record.
- **Vị trí:** thường là **cấu trúc bên trong mỗi microservice** (hoặc mỗi module của modular monolith) —
  vĩ mô chia service, vi mô mỗi service là Clean/Hexagonal.
- **KHÔNG dùng cho:** prototype/MVP/CRUD nhỏ → MVC + Active Record gọn hơn (tránh over-engineering).
- Dấu hiệu thực tế: thư mục tách **domain / application / infrastructure / presentation**.

[↑ Mục lục](#mục-lục)

<a id="q6-2"></a>
### 6.2. Phân biệt Layered Architecture với Clean Architecture

#### 📖 Giới thiệu kiến thức

Hai kiến trúc cùng "chia tầng" nhưng **khác nhau ở chiều phụ thuộc và thứ gì nằm ở trung tâm** — đây
là câu interviewer hay hỏi để xem ứng viên có hiểu **Dependency Rule / DIP** hay không.

**1. Hướng phụ thuộc (điểm khác cốt lõi):**
- **Layered:** phụ thuộc đi **một chiều từ trên xuống** — Presentation → Business → **Data Access →
  Database**. Nghĩa là **business logic phụ thuộc vào hạ tầng** (ORM/DB nằm "dưới" và nghiệp vụ gọi
  xuống nó).
- **Clean:** phụ thuộc **luôn hướng vào trong** (Dependency Rule) — lõi nghiệp vụ (Entities/Use Cases)
  **không phụ thuộc gì cả**; hạ tầng (DB, framework) nằm ở **rìa ngoài** và **phụ thuộc vào lõi** thông
  qua **interface (port)** + **DIP** ([câu 1.1](#q1-1)). Mũi tên phụ thuộc bị **đảo ngược** so với Layered.

```
LAYERED                          CLEAN
Presentation                     (ngoài) Frameworks/DB
   │ phụ thuộc ▼                          │ phụ thuộc ▼ (hướng VÀO TRONG)
Business  ──► Data Access ──► DB    Interface Adapters
(nghiệp vụ phụ thuộc hạ tầng)            │ ▼
                                    Use Cases ──► Entities  (lõi, KHÔNG phụ thuộc ai)
```

**2. Cái gì ở "trung tâm":**
- **Layered:** không có khái niệm trung tâm; tổ chức theo **vai trò kỹ thuật** (UI / nghiệp vụ / dữ
  liệu). Trên thực tế **database thường là "trái tim"** — thiết kế hay bắt đầu từ bảng/ORM.
- **Clean:** **business domain (Entities + Use Cases) là trung tâm**; DB/framework/UI chỉ là **chi
  tiết có thể thay thế** ("DB là chi tiết, không phải kiến trúc" — Uncle Bob).

**3. Khả năng test & độc lập:**
- **Layered:** test nghiệp vụ thường **cần (hoặc mock nặng) tầng Data Access/DB** vì coupling dọc.
- **Clean:** test use case bằng cách **mock port** → **không cần DB/HTTP thật**; đổi MySQL→Mongo,
  web→CLI không đụng lõi.

**4. Chi phí & độ phức tạp:**
- **Layered:** **ít boilerplate**, dễ bắt đầu — hợp CRUD/monolith vừa và nhỏ.
- **Clean:** **nhiều tầng, nhiều interface, nhiều mapping** (entity ↔ model ↔ DTO) → over-engineering
  cho app nhỏ; đáng giá khi nghiệp vụ phức tạp, vòng đời dài (xem [câu 6.1](#q6-1)).

**5. Ví dụ code minh họa (PHP) — cùng use case "đăng ký user":**

```php
// ===== LAYERED: Service gọi THẲNG DB → nghiệp vụ phụ thuộc hạ tầng =====
class UserService {
    public function register(string $email): void {
        // ...business rule (validate, chống trùng)...
        DB::table('users')->insert(['email' => $email]);   // dính chặt DB → khó test/đổi
    }
}
```

```php
// ===== CLEAN: domain ở trung tâm, KHÔNG biết DB → hạ tầng phụ thuộc nghiệp vụ =====

// 1) Entity (lõi trong cùng) — không phụ thuộc gì
class User {
    public function __construct(public string $email) {}
}

// 2) Use case định nghĩa PORT (interface) mà nó cần
interface UserRepository {
    public function save(User $user): void;
}

class RegisterUser {
    public function __construct(private UserRepository $repo) {}   // phụ thuộc ABSTRACTION
    public function handle(string $email): User {
        $user = new User($email);        // business rule thuần domain
        $this->repo->save($user);        // gọi qua interface, KHÔNG biết đó là DB gì
        return $user;
    }
}

// 3) Infrastructure (rìa ngoài) IMPLEMENT port → dependency hướng VÀO TRONG
class EloquentUserRepository implements UserRepository {
    public function save(User $user): void {
        DB::table('users')->insert(['email' => $user->email]);
    }
}
// Đổi sang Postgres/Mongo/InMemory? Chỉ viết class mới implement UserRepository,
// RegisterUser (domain) KHÔNG đổi 1 dòng → đó là sức mạnh của Clean.
```

Mấu chốt: ở Layered, `UserService` gọi thẳng `DB` (nghiệp vụ → hạ tầng). Ở Clean, `RegisterUser` (trong)
**sở hữu** interface `UserRepository`, còn `EloquentUserRepository` (ngoài) **phụ thuộc ngược vào** nó
bằng cách implement → dependency **chảy vào trong** đúng Dependency Rule. Test `RegisterUser` chỉ cần một
`FakeUserRepository` trong RAM, khỏi cần DB thật.

**6. Cấu trúc thư mục "hét lên" (Screaming Architecture) điều gì:** nhìn folder là đoán được kiến trúc:

```
LAYERED — chia theo TẦNG KỸ THUẬT        CLEAN — chia theo NGHIỆP VỤ
src/                                     src/
├── controllers/                         ├── domain/order/         Order, OrderItem, OrderStatus
├── services/                            ├── application/
├── repositories/                        │   ├── order/            CreateOrderUseCase, CancelOrderUseCase
└── models/                              │   └── ports/            OrderRepository (interface)
                                         ├── infrastructure/       PrismaOrderRepository, StripeGateway, SESMailService
Mở ra thấy "LOẠI KỸ THUẬT"               └── presentation/http/    OrderController
(không biết app LÀM GÌ)                  Mở ra thấy NGHIỆP VỤ: CreateOrder, CancelOrder...
```

- **Layered:** mở folder thấy **loại kỹ thuật** (controller/service/repository) — không biết app *làm gì*.
- **Clean:** mở folder thấy **use case nghiệp vụ** (CreateOrder, CancelOrder) — kiến trúc **"screaming"
  domain**; mọi hạ tầng (Prisma/Stripe/SES) bị đẩy hết ra rìa `infrastructure/`.

**7. Hai bẫy interviewer RẤT hay hỏi:**

**(a) Layered "chỉ chia folder" → Service phình thành nồi lẩu hạ tầng.** Vì không ai chặn chiều phụ
thuộc, theo thời gian `OrderService` gọi thẳng cả **Prisma + Redis + Stripe + Email + AWS** → business
logic **trộn lẫn** infrastructure, hết test/đổi được. Đây **không phải lỗi bắt buộc** của Layered mà là
**failure mode** khi chia tầng nhưng **không kiểm soát dependency**.

**(b) "Clean ≠ Layered nhiều tầng hơn" — tên folder KHÔNG quyết định kiến trúc.** Có đủ
`domain/ application/ infrastructure/` mà use case vẫn **import thẳng Prisma** thì vẫn là **Layered trá
hình**, vi phạm Dependency Rule:

```ts
// ❌ Có folder application/ nhưng VẪN vi phạm Clean: Application → Prisma (hướng RA NGOÀI)
class CreateOrderUseCase {
  constructor(private prisma: PrismaClient) {}
}

// ✅ Clean: use case phụ thuộc PORT (abstraction); Prisma implement nó → hướng VÀO TRONG
class CreateOrderUseCase {
  constructor(private orders: OrderRepository) {}   // OrderRepository là interface ở application/ports
}
```

→ **Hướng dependency** mới quyết định Clean hay không, **không phải tên thư mục**.

| Tiêu chí | **Layered (N-tier)** | **Clean Architecture** |
|---|---|---|
| **Hướng phụ thuộc** | Trên → dưới (nghiệp vụ → hạ tầng) | Luôn vào trong (hạ tầng → nghiệp vụ) |
| **Trung tâm** | Tầng kỹ thuật; DB hay là "trái tim" | **Business domain** (Entities/Use Cases) |
| **Quan hệ với DB/Framework** | Nghiệp vụ **phụ thuộc** ORM/DB | DB/Framework là **chi tiết ở rìa**, thay được |
| **Cơ chế then chốt** | Tầng "closed", gọi tuần tự | **Dependency Rule + DIP** (port/adapter) |
| **Test lõi độc lập** | Khó (cần/mock DB) | Dễ (mock port, không cần DB) |
| **Boilerplate** | Ít | Nhiều (interface, mapping) |
| **Hợp với** | CRUD, monolith vừa/nhỏ | Nghiệp vụ phức tạp, dài hạn, DDD |

**Quan hệ:** Clean Architecture **cũng là một dạng phân tầng**, nhưng là **Layered đã "bẻ cong" chiều
phụ thuộc bằng DIP** để lõi nghiệp vụ độc lập. Nói cách khác: Layered chia tầng theo *kỹ thuật* và để
nghiệp vụ phụ thuộc hạ tầng; Clean chia tầng theo *mức độ ổn định của nghiệp vụ* và đảo phụ thuộc để
hạ tầng phục vụ nghiệp vụ.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Giống:** cả hai đều **chia tầng** với ranh giới trách nhiệm rõ. Clean thực chất là Layered **đã đảo
  chiều phụ thuộc**.
- **Khác cốt lõi — hướng phụ thuộc:** Layered phụ thuộc **từ trên xuống** → **nghiệp vụ phụ thuộc hạ
  tầng (DB/ORM)**; Clean phụ thuộc **luôn hướng vào trong** (Dependency Rule + DIP) → **hạ tầng phụ
  thuộc nghiệp vụ**, DB/framework chỉ là chi tiết ở rìa.
- **Trung tâm:** Layered xoay quanh **tầng kỹ thuật / DB**; Clean xoay quanh **business domain**.
- **Hệ quả:** Clean **dễ test lõi & đổi DB/framework** hơn nhưng **nhiều boilerplate**; Layered **đơn
  giản, ít boilerplate**, hợp CRUD/monolith — Clean đáng dùng khi nghiệp vụ phức tạp & dài hạn.
- **Nhìn folder:** Layered "hét" **tầng kỹ thuật** (controllers/services/repositories); Clean "hét"
  **nghiệp vụ** (CreateOrder/CancelOrder) — *Screaming Architecture*.
- **Bẫy hay hỏi:** (a) Layered chỉ chia folder mà không chặn dependency → Service **trộn Prisma/Redis/
  Stripe/Email**; (b) có folder `domain/application/infrastructure/` mà use case **import thẳng ORM** thì
  **vẫn không phải Clean** — **hướng dependency** quyết định, **không phải tên thư mục**.
- **Câu nhớ nhanh:** *Layered hỏi "code thuộc tầng nào?"; Clean hỏi "ai được phép phụ thuộc vào ai?"*.

[↑ Mục lục](#mục-lục)

<a id="q6-3"></a>
### 6.3. Với `OrderService` "nồi lẩu hạ tầng" (DB + cache + payment + email), Clean Architecture xử lý thế nào?

#### 📖 Giới thiệu kiến thức

Đây là câu hỏi thực chiến nối tiếp failure mode ở [câu 5](#q5): một `OrderService` gọi thẳng
Prisma + Redis + Stripe + Email. Vấn đề **không phải "code xấu"** mà là **hướng phụ thuộc sai** —
nghiệp vụ (`createOrder`) đang **phụ thuộc vào 4 chi tiết kỹ thuật**, nên muốn test phải dựng cả DB,
Redis, Stripe sandbox; muốn đổi Stripe → VNPay phải sửa vào giữa business logic.

```ts
// ❌ Trước: 1 hàm ôm cả nghiệp vụ lẫn 4 loại hạ tầng
class OrderService {
  async createOrder(data) {
    await prisma.order.findFirst(...);             // DB
    await redis.set(...);                          // cache
    await stripe.paymentIntents.create(...);       // payment
    await sendEmail(...);                          // email / AWS
  }
}
```

**Nguyên tắc chữa (1 câu):** *mỗi chi tiết hạ tầng phải bị đẩy ra sau một **port (interface)** do tầng
trong sở hữu; use case chỉ còn kể lại **nghiệp vụ**.*

---

**Bước 1 — Chẩn đoán: tách "cái gì" (nghiệp vụ) khỏi "làm thế nào" (kỹ thuật).**

Đọc lại từng dòng và hỏi *"đây có phải quy tắc nghiệp vụ không?"*:

| Dòng code cũ | Bản chất | Là nghiệp vụ? | Xử lý |
|---|---|---|---|
| `prisma.order.findFirst()` | truy xuất/lưu đơn hàng | Ý định "lấy đơn hàng" **là** nghiệp vụ, *Prisma* thì không | → port `OrderRepository` |
| `redis.set()` | tối ưu tốc độ | **Không** — thuần kỹ thuật | → **không đưa vào use case**, bọc bằng Decorator |
| `stripe.paymentIntents.create()` | "thu tiền đơn hàng" **là** nghiệp vụ, *Stripe* thì không | Một nửa | → port `PaymentGateway` |
| `sendEmail()` | "khách được thông báo" là **hệ quả** của nghiệp vụ | Một nửa | → Domain Event + port `Mailer` |

> Mẹo nhận biết: thứ nào **đổi vendor mà nghiệp vụ không đổi** (MySQL→Mongo, Stripe→VNPay, SES→SendGrid)
> thì thứ đó là **chi tiết**, phải nằm sau port.

---

**Bước 2 — Domain: đưa rule vào Entity (không import gì cả).**

```ts
// domain/order/Order.ts — KHÔNG import prisma/stripe/redis
export class Order {
  private constructor(
    public readonly id: OrderId,
    public readonly customerId: CustomerId,
    private items: OrderItem[],
    public status: OrderStatus,
  ) {}

  static create(customerId: CustomerId, items: OrderItem[]): Order {
    if (items.length === 0) throw new EmptyOrderError();          // rule nghiệp vụ
    return new Order(OrderId.next(), customerId, items, OrderStatus.PENDING);
  }

  total(): Money {                                                // rule tính tiền
    return this.items.reduce((sum, i) => sum.add(i.subtotal()), Money.zero());
  }

  markPaid(ref: PaymentRef): void {
    if (this.status !== OrderStatus.PENDING) throw new InvalidStateError(); // state transition
    this.status = OrderStatus.PAID;
  }
}
```

**Bước 3 — Application: khai báo port mà use case cần** (interface nằm ở **tầng trong**, đây là mấu chốt DIP).

```ts
// application/ports/*.ts — chỉ nói "TÔI CẦN GÌ", không nói dùng công nghệ nào
export interface OrderRepository  { findById(id: OrderId): Promise<Order | null>;
                                    save(order: Order): Promise<void>; }
export interface PaymentGateway   { charge(amount: Money, customerId: CustomerId): Promise<PaymentRef>; }
export interface EventPublisher   { publish(events: DomainEvent[]): Promise<void>; }
export interface UnitOfWork       { run<T>(work: () => Promise<T>): Promise<T>; }   // transaction boundary
```

**Bước 4 — Use case: chỉ còn nghiệp vụ, đọc lên hiểu ngay ý định.**

```ts
// application/order/CreateOrderUseCase.ts
export class CreateOrderUseCase {
  constructor(
    private readonly orders: OrderRepository,     // ← toàn ABSTRACTION
    private readonly payments: PaymentGateway,
    private readonly events: EventPublisher,
    private readonly uow: UnitOfWork,
  ) {}

  async execute(input: CreateOrderInput): Promise<OrderId> {
    const order = Order.create(input.customerId, input.items);   // rule domain

    // Gọi hạ tầng ngoài (HTTP) NGOÀI transaction DB — tránh giữ lock trong lúc chờ mạng
    const ref = await this.payments.charge(order.total(), order.customerId);
    order.markPaid(ref);                                          // rule state transition

    await this.uow.run(async () => {
      await this.orders.save(order);                              // lưu qua PORT
      await this.events.publish([new OrderCreated(order.id)]);    // outbox: cùng transaction
    });

    return order.id;                                              // KHÔNG biết Prisma/Redis/Stripe/SES
  }
}
```

**Bước 5 — Infrastructure: adapter implement port (phụ thuộc **hướng vào trong**).**

```ts
// infrastructure/persistence/PrismaOrderRepository.ts
export class PrismaOrderRepository implements OrderRepository {
  async findById(id: OrderId) { const row = await prisma.order.findFirst({ where: { id: id.value } });
                                return row ? OrderMapper.toDomain(row) : null; }
  async save(order: Order)    { await prisma.order.upsert(OrderMapper.toPersistence(order)); }
}

// infrastructure/payment/StripePaymentGateway.ts
export class StripePaymentGateway implements PaymentGateway {
  async charge(amount: Money, customerId: CustomerId) {
    try {
      const pi = await stripe.paymentIntents.create({ amount: amount.cents, customer: customerId.value });
      return new PaymentRef(pi.id);
    } catch (e) {
      throw new PaymentDeclinedError(e.message);   // dịch lỗi vendor → lỗi DOMAIN
    }
  }
}
```

> Chi tiết quan trọng: adapter phải **dịch lỗi & dịch kiểu dữ liệu**. Nếu use case phải `catch (e: Stripe.errors.CardError)`
> thì Stripe đã "rò rỉ" vào tầng trong — vẫn vi phạm Dependency Rule (leaky abstraction).

---

**Bước 6 — Xử lý riêng 2 ca khó: cache và email.**

**(a) Cache (Redis) — KHÔNG tạo `CacheService` rồi tiêm vào use case.** Cache là chi tiết tối ưu, nghiệp vụ
không cần biết. Cách sạch: **Decorator/Proxy** bọc quanh chính port `OrderRepository`
(xem [Decorator câu 2.4](#q2-4), [Proxy câu 2.6](#q2-6)):

```ts
// infrastructure/persistence/CachedOrderRepository.ts
export class CachedOrderRepository implements OrderRepository {
  constructor(private inner: OrderRepository, private redis: Redis) {}
  async findById(id: OrderId) {
    const hit = await this.redis.get(`order:${id.value}`);
    if (hit) return OrderMapper.toDomain(JSON.parse(hit));
    const order = await this.inner.findById(id);                 // fallback DB thật
    if (order) await this.redis.setex(`order:${id.value}`, 300, JSON.stringify(order));
    return order;
  }
  async save(order: Order) {
    await this.inner.save(order);
    await this.redis.del(`order:${order.id.value}`);             // invalidate
  }
}
```

→ Use case **không đổi 1 dòng**; bật/tắt cache chỉ là chuyện wiring. Đây là **OCP + Decorator** ở tầng hạ tầng.

**(b) Email — không gọi `sendEmail()` giữa use case.** Gửi mail là **hệ quả (side effect)**, không phải
bước nghiệp vụ; nhét vào use case sẽ khiến "tạo đơn" fail chỉ vì SES lỗi. Cách sạch: use case **phát
Domain Event** `OrderCreated`, một **handler ở tầng ngoài** lắng nghe và gửi mail (thường qua queue):

```ts
// infrastructure/messaging/SendOrderConfirmation.ts  (tầng ngoài, subscribe event)
export class SendOrderConfirmation {
  constructor(private mailer: Mailer, private orders: OrderRepository) {}
  async handle(e: OrderCreated) { /* ...load order, gọi this.mailer.send(...) */ }
}
```

Để event không "mất" khi commit DB thành công nhưng publish lỗi → dùng **Outbox pattern**: ghi event vào
bảng `outbox` **cùng transaction** với `orders.save()`, một worker đọc outbox rồi mới bắn ra queue.

---

**Bước 7 — Composition Root: nơi DUY NHẤT biết mọi công nghệ.**

```ts
// NestJS: infrastructure/order.module.ts
providers: [
  { provide: 'OrderRepository',
    useFactory: (redis) => new CachedOrderRepository(new PrismaOrderRepository(), redis),
    inject: [Redis] },
  { provide: 'PaymentGateway', useClass: StripePaymentGateway },
  { provide: CreateOrderUseCase,
    useFactory: (r, p, e, u) => new CreateOrderUseCase(r, p, e, u),
    inject: ['OrderRepository', 'PaymentGateway', 'EventPublisher', 'UnitOfWork'] },
]
```

```php
// Laravel: app/Providers/DomainServiceProvider.php
$this->app->bind(OrderRepository::class, fn () => new CachedOrderRepository(
    new EloquentOrderRepository(), app('redis')
));
$this->app->bind(PaymentGateway::class, StripePaymentGateway::class);
```

Toàn bộ tên vendor (Prisma, Redis, Stripe, SES) **chỉ xuất hiện ở đây và trong `infrastructure/`**.

---

**Kết quả — trước / sau:**

| Tiêu chí | ❌ Trước (Service nồi lẩu) | ✅ Sau (Clean) |
|---|---|---|
| Test "tạo đơn" | cần DB + Redis + Stripe sandbox | `new CreateOrderUseCase(fakeRepo, fakeGateway, ...)` — chạy trong RAM, mili giây |
| Đổi Stripe → VNPay | sửa **giữa** business logic | viết `VnpayPaymentGateway implements PaymentGateway`, đổi 1 dòng binding |
| Bật/tắt cache | rải `redis.get/set` khắp service | bọc/bỏ `CachedOrderRepository` ở composition root |
| SES chết | **tạo đơn fail** | đơn vẫn tạo; mail retry qua queue/outbox |
| Đọc code | thấy thao tác kỹ thuật | thấy **nghiệp vụ**: create → charge → markPaid → save |

```ts
// Test use case không cần bất kỳ hạ tầng nào
const repo = new InMemoryOrderRepository();
const uc = new CreateOrderUseCase(repo, new AlwaysApprovePayment(), new NoopPublisher(), new NoopUow());
await uc.execute({ customerId, items: [item] });
expect((await repo.findById(id)).status).toBe(OrderStatus.PAID);
```

**Mức độ "vừa đủ" (đừng over-engineering):** với app nhỏ, không cần đủ 7 bước. Thứ tự ưu tiên khi refactor
dần: **(1)** tách hạ tầng *ngoài mạng* (payment/email) ra port trước — đây là thứ làm test đau nhất;
**(2)** tách DB ra `Repository`; **(3)** cache bằng Decorator; **(4)** đến khi rule nghiệp vụ đủ nhiều mới
tách Entity/Use Case đầy đủ. Một `PaymentGateway` interface + `OrderRepository` interface đã giải quyết
~80% cơn đau mà chỉ tốn ~2 file.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Chẩn bệnh:** vấn đề không phải code xấu mà là **hướng phụ thuộc sai** — nghiệp vụ đang phụ thuộc
  Prisma/Redis/Stripe/SES. Clean chữa bằng cách **đẩy mỗi chi tiết ra sau một port do tầng trong sở hữu** (DIP).
- **Cách làm:** rule vào **Entity `Order`** (`create`, `total`, `markPaid`) → **use case** `CreateOrderUseCase`
  chỉ nhận `OrderRepository`, `PaymentGateway`, `EventPublisher` → **adapter** `PrismaOrderRepository`,
  `StripePaymentGateway` implement chúng ở `infrastructure/` → **composition root** (NestJS module /
  Laravel Service Provider) là nơi duy nhất biết tên công nghệ.
- **Cache không tiêm vào use case:** bọc **Decorator** `CachedOrderRepository implements OrderRepository`
  → bật/tắt cache chỉ là chuyện wiring, nghiệp vụ không đổi 1 dòng.
- **Email không gọi trực tiếp:** use case phát **domain event `OrderCreated`**, handler ngoài gửi mail
  (queue + **outbox pattern**) → SES chết thì đơn hàng vẫn tạo được.
- **Adapter phải dịch lỗi vendor thành lỗi domain** (`PaymentDeclinedError`), nếu use case còn `catch`
  lỗi Stripe thì abstraction bị **rò rỉ**, vẫn vi phạm Dependency Rule.
- **Chi tiết ghi điểm:** gọi API ngoài (charge) **ngoài** transaction DB để không giữ lock khi chờ mạng;
  test use case chỉ cần fake in-memory, không cần DB/Redis/Stripe.
- **Thực dụng:** không cần làm hết ngay — tách **payment/email ra port trước**, rồi repository, rồi cache
  bằng decorator; app CRUD nhỏ thì dừng ở đó là đủ.

[↑ Mục lục](#mục-lục)

---

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
