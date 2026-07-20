<a id="mục-lục"></a>
# Backend — Laravel

> Eloquent ORM · Migration · Artisan · Middleware · Service Container & Provider · Queue/Job ·
> Event/Listener · Blade · Validation · Cache · routing Laravel.
> (Câu hỏi về cú pháp/engine PHP thuần nằm ở [php-core.md](./php-core.md).)

## 📑 Mục lục

1. [Service Container trong Laravel là gì?](#q1)
   - 1.1. [`bind` vs `singleton` vs `instance` khác nhau thế nào?](#q1-1)
   - 1.2. [Service Container và Service Provider khác gì nhau?](#q1-2)
2. [Service Provider trong Laravel là gì?](#q2)
   - 2.1. [`register()` vs `boot()` khác nhau thế nào?](#q2-1)
   - 2.2. [Deferred Provider là gì? Khi nào dùng?](#q2-2)
3. [Facade trong Laravel là gì?](#q3)
   - 3.1. [Facade hoạt động bên trong như thế nào?](#q3-1)
   - 3.2. [Facade vs Dependency Injection — nên dùng cái nào?](#q3-2)
4. [Contract trong Laravel là gì?](#q4)
   - 4.1. [Contract vs Facade — khác nhau và khi nào dùng?](#q4-1)
5. [Cơ chế driver theo config (`CACHE_DRIVER`, `DB_CONNECTION`...) dùng design pattern gì?](#q5)
   - 5.1. [Manager pattern hoạt động thế nào? Tự đăng ký custom driver ra sao?](#q5-1)
6. [Vòng đời của một request trong Laravel](#q6)
   - 6.1. [Phân biệt `get()` và `input()` khi lấy dữ liệu Request](#q6-1)
7. [Có mấy loại middleware trong Laravel?](#q7)
   - 7.1. [Before vs After middleware & terminable middleware](#q7-1)
   - 7.2. [Truyền tham số cho middleware & middleware priority](#q7-2)
   - 7.3. [Middleware lọc request ở đâu? Cơ chế lọc (Pipeline) hoạt động thế nào?](#q7-3)
   - 7.4. [CSRF token & VerifyCsrfToken middleware trong Laravel](#q7-4)
   - 7.5. [Có áp dụng CSRF cho nhóm route `api` không? Có cần thiết không?](#q7-5)
8. [Eloquent và Query Builder trong Laravel là gì?](#q8)
   - 8.1. [Xem/đổi Eloquent & Query Builder sang raw SQL (để debug)](#q8-1)
   - 8.2. [Laravel Debugbar là gì?](#q8-2)
   - 8.3. [Laravel Telescope là gì?](#q8-3)
   - 8.4. [Thứ tự thực thi: lệnh nào chạy trước, lệnh nào chạy sau?](#q8-4)
   - 8.5. [N+1 query problem và eager loading](#q8-5)
   - 8.6. [So sánh `with()` và `load()`](#q8-6)
   - 8.7. [Cơ chế ánh xạ record của Eloquent (Active Record & hydration)](#q8-7)
   - 8.8. [`$fillable` và `$guarded` (mass assignment) trong model](#q8-8)
9. [Eloquent Relationship trong Laravel là gì?](#q9)
   - 9.1. [`hasMany`/`hasOne` vs `belongsTo` — khóa ngoại đặt ở đâu?](#q9-1)
   - 9.2. [Many-to-many (pivot) & Polymorphic relationship](#q9-2)
10. [Eloquent Scope (Query Scope) là gì?](#q10)
    - 10.1. [Global Scope & Soft Deletes](#q10-1)
11. [Collection trong Laravel là gì?](#q11)
    - 11.1. [Collection vs array thường & LazyCollection (xử lý dữ liệu lớn)](#q11-1)
12. [Observer trong Laravel (Eloquent Observer) là gì?](#q12)
    - 12.1. [Các sự kiện vòng đời & vì sao observer không chạy với mass update?](#q12-1)
    - 12.2. [Observer vs Event/Listener — khi nào dùng cái nào?](#q12-2)
13. [Cache trong Laravel (định nghĩa & áp dụng)](#q13)
    - 13.1. [Cache invalidation & Cache tags](#q13-1)
14. [Các thư viện thường dùng với Composer trong Laravel](#q14)
    - 14.1. [Các package chính chủ (first-party) của Laravel](#q14-1)
    - 14.2. [Các authentication package (Sanctum vs Passport, Breeze/Jetstream/Fortify, Socialite)](#q14-2)
15. [Phân biệt SPA và MPA trong Laravel](#q15)

---

<a id="q1"></a>
## 1. Service Container trong Laravel là gì?

### 📖 Giới thiệu kiến thức

**Service Container** (còn gọi là **IoC Container** — Inversion of Control) là **trái tim** của Laravel:
một "chiếc hộp" trung tâm chịu trách nhiệm **tạo (resolve), quản lý và inject** các object cùng
**dependency** của chúng. Thay vì class tự `new` ra thứ nó cần, container sẽ **tự dựng và đưa vào** —
đây chính là **Dependency Injection (DI)**.

Hai khái niệm cốt lõi:

- **Binding (đăng ký):** "dạy" container biết cách tạo ra một service. Thường bind một **interface**
  với một **implementation cụ thể**.
- **Resolving (lấy ra):** yêu cầu container trả về một instance đã dựng sẵn dependency. Thực hiện qua
  `app()->make()`, hoặc tự động qua **type-hint** ở constructor/method.

**Vì sao quan trọng?** Container giúp tuân thủ **Dependency Inversion** (SOLID): class phụ thuộc vào
**abstraction** (interface), không phụ thuộc class cụ thể → dễ **thay thế, test (mock), bảo trì**.

**Automatic resolution (tự động giải quyết):** điểm mạnh nhất. Khi type-hint một class ở constructor,
container **tự đọc qua Reflection**, đệ quy dựng mọi dependency lồng nhau mà không cần khai báo thủ công:

```php
class OrderService
{
    // Chỉ cần type-hint — container TỰ tạo PaymentGateway và inject vào
    public function __construct(private PaymentGateway $gateway) {}
}

// Ở đâu đó: không cần new OrderService(new StripeGateway(...))
$service = app(OrderService::class);   // container tự lo toàn bộ chuỗi dependency
```

**Binding với interface (cách dùng phổ biến nhất):**

```php
// Trong một Service Provider, method register():
$this->app->bind(PaymentGateway::class, StripeGateway::class);

// Từ giờ, mọi nơi type-hint PaymentGateway sẽ nhận StripeGateway.
// Muốn đổi sang PayPal → chỉ sửa 1 dòng bind, không đụng code nghiệp vụ.
```

> Liên hệ: Service Container là hiện thực của **Dependency Injection / IoC** — xem nguyên lý
> **Dependency Inversion** trong `Common/solid-design-pattern.md`. Container được tạo ở bước
> `bootstrap/app.php` và được dùng xuyên suốt vòng đời request (xem câu 6).

### 🎯 Tóm tắt trả lời phỏng vấn

- Service Container (IoC Container) là nơi **tạo, quản lý và inject dependency** cho toàn bộ ứng dụng Laravel.
- Hai thao tác chính: **binding** (đăng ký cách tạo service, thường interface → implementation) và
  **resolving** (lấy instance ra qua `make()` hoặc type-hint).
- **Automatic resolution:** type-hint ở constructor → container tự dùng **Reflection** dựng cả chuỗi dependency.
- Lợi ích: tuân thủ **Dependency Inversion**, code lỏng lẻo (loose coupling), dễ **mock khi test** và dễ thay thế implementation.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. `bind` vs `singleton` vs `instance` khác nhau thế nào?

#### 📖 Giới thiệu kiến thức

Cả ba đều là cách **binding** vào container, khác nhau ở **vòng đời (lifecycle)** của instance:

| Cách bind | Mỗi lần resolve trả về | Khi nào dùng |
|-----------|------------------------|--------------|
| `bind()` | **Instance MỚI** mỗi lần `make()` | Service có state riêng, cần bản sao mới mỗi lần |
| `singleton()` | **Cùng MỘT instance** trong suốt request | Service nặng, stateless, dùng chung (DB manager, config, logger) |
| `instance()` | Trả về **object có sẵn** bạn đưa vào | Khi bạn đã tự tạo object và muốn đăng ký nó |

```php
// bind: mỗi make() → object mới
$this->app->bind(Transformer::class, fn () => new Transformer());

// singleton: chỉ tạo 1 lần, các lần sau trả lại đúng instance đó (lazy)
$this->app->singleton(CacheManager::class, fn ($app) => new CacheManager($app));

// instance: đăng ký một object đã khởi tạo sẵn
$client = new ApiClient(config('services.api.key'));
$this->app->instance(ApiClient::class, $client);
```

- **`bind` mặc định là lazy:** closure chỉ chạy **khi service thực sự được resolve**, không phải lúc bind.
- **`singleton`** rất hay gặp cho các manager dùng chung để tránh tạo lại object tốn kém.

#### 🎯 Tóm tắt trả lời phỏng vấn

- `bind`: trả **instance mới mỗi lần** resolve — cho service có state riêng.
- `singleton`: trả **cùng một instance** suốt vòng đời request — cho service nặng/stateless dùng chung.
- `instance`: đăng ký **một object đã tạo sẵn** vào container.
- Cả `bind` và `singleton` đều **lazy** (closure chỉ chạy khi resolve).

[↑ Mục lục](#mục-lục)

<a id="q1-2"></a>
### 1.2. Service Container và Service Provider khác gì nhau?

#### 📖 Giới thiệu kiến thức

Hai khái niệm hay bị nhầm vì luôn đi cùng nhau, nhưng vai trò khác hẳn:

- **Service Container** là **cái hộp** — cơ chế lưu trữ binding và resolve/inject dependency.
- **Service Provider** là **nơi khai báo** — chỗ tập trung để bạn **đăng ký (bind) service vào container**
  và **khởi động (boot)** chúng. Đây là điểm vào trung tâm để cấu hình mọi thành phần của một package/module.

Mỗi provider có 2 method:

- **`register()`**: **chỉ bind** service vào container. KHÔNG được dùng service khác ở đây vì có thể chưa
  được register xong.
- **`boot()`**: chạy **sau khi tất cả provider đã register** → được phép dùng mọi binding (đăng ký route,
  event, view composer, policy...).

```php
class PaymentServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // chỉ đăng ký vào container
        $this->app->singleton(PaymentGateway::class, StripeGateway::class);
    }

    public function boot(): void
    {
        // dùng service, đăng ký route/event... (mọi thứ đã sẵn sàng)
        Event::listen(PaymentReceived::class, SendReceiptEmail::class);
    }
}
```

Cách nhớ: **Provider** *cho* vào, *cấu hình* container; **Container** *giữ* và *trả ra* khi cần.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Container** = cơ chế giữ binding và resolve/inject dependency. **Provider** = nơi khai báo, bind service vào container.
- Provider có `register()` (**chỉ bind**, không dùng service khác) và `boot()` (chạy **sau khi mọi thứ đã register**, được dùng tự do).
- Mọi cấu hình service của package/module được gom vào provider → một điểm vào sạch sẽ.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Service Provider trong Laravel là gì?

### 📖 Giới thiệu kiến thức

**Service Provider** là **điểm trung tâm để khởi động (bootstrap)** ứng dụng Laravel. Gần như **mọi thứ**
của framework — kết nối DB, queue, mail, validation, route, event... — đều được **đăng ký và cấu hình**
thông qua các provider. Có thể coi provider là "**nơi bạn nói cho Laravel biết phải dựng những gì và dựng
như thế nào**".

Mỗi provider kế thừa `Illuminate\Support\ServiceProvider` và có **2 method chính**:

- **`register()`**: **chỉ** dùng để **bind service vào Service Container**. Tuyệt đối không truy cập service
  khác ở đây vì provider khác có thể **chưa được register**.
- **`boot()`**: chạy **sau khi TẤT CẢ provider đã register xong** → được phép dùng mọi binding để đăng ký
  route, event listener, view composer, policy, validation rule, publish config...

```php
class RepositoryServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // bind interface → implementation (chỉ đăng ký, chưa dùng)
        $this->app->bind(UserRepository::class, EloquentUserRepository::class);
    }

    public function boot(): void
    {
        // mọi service đã sẵn sàng → an toàn để dùng/đăng ký thêm
        View::composer('dashboard', DashboardComposer::class);
    }
}
```

**Quy trình hoạt động trong vòng đời (xem câu 6):** lúc bootstrap, Kernel chạy bootstrapper
`RegisterProviders` (gọi `register()` của **mọi** provider) **trước**, rồi `BootProviders`
(gọi `boot()` của mọi provider) **sau**. Thứ tự 2 pha này chính là lý do tách `register`/`boot`.

**Đăng ký provider ở đâu:**
- **Laravel ≤ 10:** liệt kê trong mảng `providers` của `config/app.php`.
- **Laravel 11+:** gom vào `bootstrap/providers.php` (tự động cập nhật khi tạo provider qua Artisan).

```bash
php artisan make:provider RepositoryServiceProvider
```

**Package Discovery:** package bên thứ ba có thể khai báo provider trong `composer.json` (mục
`extra.laravel.providers`) để Laravel **tự đăng ký** mà không cần sửa file config thủ công.

> Liên hệ: Provider làm việc trực tiếp với **Service Container** (câu 1) — nó là *nơi khai báo*, container là
> *cơ chế giữ và resolve*. Xem thêm so sánh nhanh ở mục 1.2.

### 🎯 Tóm tắt trả lời phỏng vấn

- Service Provider là **điểm trung tâm để bootstrap** ứng dụng — gần như mọi thành phần Laravel được đăng ký qua provider.
- Có 2 method: **`register()`** (chỉ bind service vào container) và **`boot()`** (chạy sau khi mọi provider đã register, dùng để đăng ký route/event/view composer...).
- Đăng ký ở `config/app.php` (≤10) hoặc `bootstrap/providers.php` (11+); package có thể **auto-discovery** qua `composer.json`.
- Provider là *nơi khai báo*, Service Container là *cơ chế giữ & resolve* — hai mặt của cùng cơ chế DI.

[↑ Mục lục](#mục-lục)

<a id="q2-1"></a>
### 2.1. `register()` vs `boot()` khác nhau thế nào?

#### 📖 Giới thiệu kiến thức

Đây là câu hỏi "ăn điểm" để kiểm tra bạn có hiểu **thứ tự bootstrap** hay không. Mấu chốt: Laravel **không**
chạy từng provider trọn vẹn rồi mới sang provider kế tiếp. Nó chạy theo **2 pha riêng biệt, quét toàn bộ
provider 2 lượt**:

```
Lượt 1 (RegisterProviders):  A::register() → B::register() → C::register()   ← bind tất cả
Lượt 2 (BootProviders):      A::boot()     → B::boot()     → C::boot()       ← rồi mới dùng
```

Chính vì có **ranh giới** giữa 2 lượt mà ta phải tách vai trò:

**1. `register()` — chỉ "khai báo nguyên liệu", chưa "nấu".**
Nhiệm vụ duy nhất: **bind** vào Service Container (`bind`, `singleton`, `instance`). Tại thời điểm này
**chưa chắc** provider khác đã register, nên nếu bạn gọi một service do provider *khác* cung cấp, nó có thể
**chưa tồn tại** → lỗi. Đây là lý do quy tắc "register chỉ để bind".

```php
public function register(): void
{
    // ✅ ĐÚNG: chỉ đăng ký cách tạo service
    $this->app->singleton(PaymentGateway::class, StripeGateway::class);

    // ❌ SAI: dùng một service/binding của provider khác ngay tại đây
    // $gateway = $this->app->make(PaymentGateway::class); // có thể chưa register xong
    // Config::get(...) / Route::get(...) cũng KHÔNG nên đặt ở register()
}
```

**2. `boot()` — lúc này "mọi nguyên liệu đã sẵn", thoải mái nấu.**
Chạy **sau khi TẤT CẢ provider đã register**, nên mọi binding chắc chắn đã có. Đây là nơi đặt các việc
**"dùng" service và đăng ký hành vi của app**:

```php
public function boot(): void
{
    // đăng ký route, event, view composer, policy, validation rule, observer...
    Route::middleware('web')->group(base_path('routes/custom.php'));
    Event::listen(OrderPaid::class, SendInvoice::class);
    View::composer('sidebar', SidebarComposer::class);
    Gate::policy(Post::class, PostPolicy::class);
    Validator::extend('cccd', fn ($attr, $value) => preg_match('/^\d{12}$/', $value));
    Post::observe(PostObserver::class);   // gọi model → cần container/DB đã sẵn sàng
}
```

**Ví dụ minh họa rõ ranh giới — vì sao thứ tự lại quan trọng:**

```php
// CacheServiceProvider
public function register(): void
{
    $this->app->singleton('my-cache', fn () => new RedisCache(config('cache.prefix')));
}

// ReportServiceProvider — KHÔNG biết Cache provider đã register chưa
public function register(): void
{
    // ❌ Rủi ro: 'my-cache' có thể chưa được bind nếu CacheServiceProvider chạy sau
    // $cache = $this->app->make('my-cache');
}
public function boot(): void
{
    // ✅ An toàn: tới pha boot thì 'my-cache' chắc chắn đã bind ở lượt register trước đó
    $cache = $this->app->make('my-cache');
    $cache->warmUp();
}
```

**`boot()` hỗ trợ method injection** — type-hint tham số, container tự resolve và inject (vì container đã
sẵn sàng hoàn toàn ở pha này). `register()` thì không có khả năng này:

```php
public function boot(Router $router, Dispatcher $events): void
{
    $router->pushMiddlewareToGroup('web', LocaleMiddleware::class);
    $events->listen(UserRegistered::class, SendWelcomeEmail::class);
}
```

**Tóm gọn bằng bảng:**

| | `register()` | `boot()` |
|---|--------------|----------|
| Chạy ở pha | **register** (lượt 1, trước) | **boot** (lượt 2, sau khi mọi provider đã register) |
| Mục đích | **Chỉ bind** service vào container | **Dùng** service: route, event, view composer, policy, observer... |
| Được dùng service/binding khác? | **KHÔNG** (có thể chưa register) | **CÓ** (mọi binding đã sẵn sàng) |
| Hỗ trợ method injection? | Không | **Có** — type-hint tham số, container tự inject |
| Ví dụ điển hình | `$this->app->singleton(...)` | `Route::...`, `Event::listen(...)`, `View::composer(...)` |

**Lỗi kinh điển:** gọi một service (vd `Auth`, hoặc binding của provider khác), đọc `config()`, hay đăng ký
route ngay trong `register()` → có thể gặp lỗi vì thứ đó **chưa được register/boot**. Quy tắc vàng:
**`register` chỉ để BIND, mọi việc "DÙNG" để dành cho `boot`.**

#### 🎯 Tóm tắt trả lời phỏng vấn

- Laravel bootstrap theo **2 pha quét toàn bộ provider**: chạy `register()` của **mọi** provider trước, rồi mới chạy `boot()` của **mọi** provider.
- `register()`: chạy **trước**, **chỉ bind** service vào container; **không** được dùng service/binding/config của provider khác (có thể chưa sẵn).
- `boot()`: chạy **sau** khi tất cả đã register → được **dùng tự do** mọi binding để đăng ký route, event, view composer, policy, observer, validation rule...
- `boot()` còn hỗ trợ **method injection** (type-hint tham số → container tự inject); `register()` thì không.
- Quy tắc vàng: **`register` = BIND, `boot` = USE.**

[↑ Mục lục](#mục-lục)

<a id="q2-2"></a>
### 2.2. Deferred Provider là gì? Khi nào dùng?

#### 📖 Giới thiệu kiến thức

Mặc định **mọi** provider đều được register ở **mỗi** request → nếu provider nặng nhưng ít khi dùng tới,
đây là lãng phí. **Deferred Provider** (provider trì hoãn) giải quyết việc này: provider chỉ được nạp
**khi service nó cung cấp thực sự được resolve**, giúp **giảm thời gian khởi động** mỗi request.

Cách làm: implement interface `DeferrableProvider` và khai báo method `provides()` trả về danh sách
binding mà provider chịu trách nhiệm:

```php
use Illuminate\Contracts\Support\DeferrableProvider;

class ReportServiceProvider extends ServiceProvider implements DeferrableProvider
{
    public function register(): void
    {
        $this->app->singleton(ReportGenerator::class, fn () => new ReportGenerator());
    }

    // Laravel chỉ load provider này khi ai đó resolve ReportGenerator
    public function provides(): array
    {
        return [ReportGenerator::class];
    }
}
```

**Điều kiện dùng được:** provider **chỉ làm việc trong `register()`** (binding lazy). Nếu provider cần
`boot()` để đăng ký route/event chạy mỗi request thì **không** trì hoãn được (vì những việc đó phải chạy
sớm). Laravel lưu một "manifest" ánh xạ service → provider để biết khi nào cần nạp.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Deferred Provider chỉ được nạp **khi service nó cung cấp được resolve** → **giảm thời gian bootstrap** mỗi request.
- Cách làm: implement `DeferrableProvider` + khai báo `provides()` liệt kê các binding.
- Chỉ áp dụng cho provider **chỉ bind trong `register()`** (lazy); provider cần `boot()` đăng ký route/event thì không trì hoãn được.

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Facade trong Laravel là gì?

### 📖 Giới thiệu kiến thức

**Facade** cung cấp một **giao diện "tĩnh" (static-like)** để truy cập nhanh các service đang nằm trong
**Service Container** (câu 1). Thay vì phải resolve service ra rồi gọi method, bạn gọi trực tiếp kiểu
`Cache::get(...)`, `DB::table(...)`, `Route::get(...)`, `Log::info(...)` — cú pháp **ngắn gọn, dễ đọc**
mà vẫn tận dụng toàn bộ DI bên dưới.

```php
// Dùng Facade — gọn, đọc như câu lệnh tĩnh
$value = Cache::get('key');
$users = DB::table('users')->where('active', 1)->get();

// Tương đương khi KHÔNG dùng Facade (resolve thủ công từ container):
$value = app('cache')->get('key');
$users = app('db')->table('users')->where('active', 1)->get();
```

**Điểm mấu chốt: Facade KHÔNG phải static thật.** Tên gọi "static interface" chỉ là **bề ngoài về cú pháp**.
Thực chất mỗi lời gọi static được chuyển hướng (proxy) tới **một instance thật** lấy từ container. Nhờ vậy
Facade **vẫn testable** (mock được) — khác hẳn static method cứng nhắc kiểu helper class thông thường.

**Vì sao quan trọng / khi nào gặp:**
- Hầu hết API "đẹp" của Laravel (`Cache`, `DB`, `Route`, `Queue`, `Auth`, `Storage`...) đều là Facade.
- Facade được "bật" rất sớm trong vòng đời request qua bootstrapper **`RegisterFacades`** (xem câu 6):
  nó đăng ký các alias (vd `Cache` → `Illuminate\Support\Facades\Cache`) để bạn gọi mà không cần `use`
  đường dẫn dài hay namespace đầy đủ.

**Real-time Facade:** từ Laravel 5.4, bạn có thể biến **bất kỳ class nào** thành facade ngay lập tức bằng
cách thêm tiền tố `Facades\` vào namespace khi import — không cần tự viết facade class:

```php
use Facades\App\Services\Publisher;   // tiền tố Facades\

Publisher::publish($podcast);          // gọi như facade, vẫn resolve instance từ container
```

> Liên hệ: Facade là "mặt tiền" gọn gàng cho cơ chế **Service Container / DI** (câu 1). Về mặt mẫu thiết kế,
> nó gần với **Proxy/Service Locator** hơn là Facade Pattern kinh điển trong GoF — xem
> `Common/solid-design-pattern.md`.

### 🎯 Tóm tắt trả lời phỏng vấn

- Facade cung cấp **giao diện tĩnh tiện lợi** để truy cập service trong Service Container (vd `Cache::get()`, `DB::table()`).
- **Không phải static thật**: mỗi lời gọi được proxy tới **instance thật resolve từ container** → vẫn **mock/test được**.
- Được bật sớm qua bootstrapper **`RegisterFacades`** (đăng ký alias để gọi mà không cần namespace đầy đủ).
- Có **Real-time Facade** (`Facades\...`) biến class bất kỳ thành facade mà không cần viết facade class.

[↑ Mục lục](#mục-lục)

<a id="q3-1"></a>
### 3.1. Facade hoạt động bên trong như thế nào?

#### 📖 Giới thiệu kiến thức

Bí mật nằm ở **magic method `__callStatic()`** của PHP và lớp cha `Illuminate\Support\Facades\Facade`.
Mỗi facade chỉ cần khai báo **một method duy nhất**: `getFacadeAccessor()` — trả về **"khóa" binding**
trong container của service thật.

```php
class Cache extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'cache';   // khóa binding trong Service Container
    }
}
```

Khi bạn gọi `Cache::get('key')`, vì class `Cache` **không có** method static `get()`, PHP rơi vào
`__callStatic('get', ['key'])` của lớp cha `Facade`. Lớp cha sẽ:

```php
// Rút gọn logic trong Illuminate\Support\Facades\Facade
public static function __callStatic($method, $args)
{
    $instance = static::getFacadeRoot();   // (1) lấy instance thật từ container
    return $instance->$method(...$args);   // (2) gọi method THẬT trên instance đó
}

// getFacadeRoot() đại khái:
//   app()[ static::getFacadeAccessor() ]   → resolve service theo khóa 'cache'
```

Tức là: **static call → `__callStatic` → resolve instance từ container theo accessor → gọi method thật**.
Facade chỉ là lớp "chuyển tiếp" (forwarding), không chứa logic nghiệp vụ.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Facade kế thừa lớp `Facade`, chỉ khai báo `getFacadeAccessor()` trả về **khóa binding** của service.
- Gọi static (vd `Cache::get`) rơi vào **`__callStatic`** → lớp cha **resolve instance thật từ container** theo accessor → **gọi method thật** trên instance đó.
- Vậy facade chỉ là **lớp chuyển tiếp tới container**, không phải static method thật.

[↑ Mục lục](#mục-lục)

<a id="q3-2"></a>
### 3.2. Facade vs Dependency Injection — nên dùng cái nào?

#### 📖 Giới thiệu kiến thức

Cả hai đều truy cập cùng service trong container, khác nhau ở **cách lấy** và **độ tường minh**:

| | Facade | Dependency Injection (type-hint) |
|---|--------|----------------------------------|
| Cú pháp | `Cache::get()` — gọn, không cần khai báo | Inject qua constructor/method, rồi `$this->cache->get()` |
| Dependency | **Ẩn** (nhìn thân method mới thấy dùng gì) | **Tường minh** ở constructor signature |
| Dễ test | Mock được qua `Cache::shouldReceive(...)` | Mock tự nhiên qua inject mock vào constructor |
| Phù hợp | Code ngắn, controller/route, prototyping | Class có nhiều dependency, cần rõ ràng & dễ test quy mô lớn |

**Test một Facade:** vì facade proxy tới container, Laravel cho phép **tráo (swap) bằng mock** dễ dàng:

```php
// Trong test — kỳ vọng Cache::get('key') được gọi đúng 1 lần
Cache::shouldReceive('get')->once()->with('key')->andReturn('value');
```

**Nhược điểm của Facade:** dependency bị **ẩn** → một class lạm dụng nhiều facade sẽ khó nhìn ra nó thực sự
phụ thuộc những gì (vi phạm tinh thần **Dependency Inversion**). Vì vậy nhiều team ưu tiên **constructor
injection** cho service/class nghiệp vụ phức tạp, và dùng **Facade cho nơi tiện lợi** (controller, route,
config nhanh).

> So sánh nhanh: Facade và DI **không loại trừ nhau** — cùng dựa trên Service Container. Chọn theo tiêu chí
> *tường minh dependency* (DI) hay *gọn cú pháp* (Facade).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai cùng truy cập service trong container; **Facade gọn nhưng giấu dependency**, **DI tường minh nhưng dài hơn**.
- Facade vẫn test được qua `Facade::shouldReceive(...)` (tráo mock); DI test bằng cách inject mock vào constructor.
- Nhược điểm Facade: dependency **ẩn**, lạm dụng làm class khó đọc/khó kiểm soát (ngược tinh thần Dependency Inversion).
- Thực tế: **DI cho class nghiệp vụ phức tạp**, **Facade cho chỗ tiện lợi** (controller, route).

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Contract trong Laravel là gì?

### 📖 Giới thiệu kiến thức

**Contract** là **tập hợp các interface** do Laravel cung cấp (nằm trong namespace
`Illuminate\Contracts`), **định nghĩa các dịch vụ cốt lõi** của framework. Mỗi service "đẹp" mà bạn hay
dùng đều có một contract tương ứng: `Illuminate\Contracts\Cache\Repository` (Cache),
`Illuminate\Contracts\Queue\Queue` (Queue), `Illuminate\Contracts\Mail\Mailer` (Mail),
`Illuminate\Contracts\Filesystem\Filesystem`...

Nói ngắn gọn: **Contract = interface; implementation thật = class do Laravel viết.** Service Container
(câu 1) đã **bind sẵn** mỗi contract với implementation mặc định, nên bạn chỉ cần **type-hint contract**
ở constructor là container tự inject đúng class:

```php
use Illuminate\Contracts\Cache\Repository as Cache;

class ReportService
{
    // Phụ thuộc vào INTERFACE (contract), không phải class cụ thể
    public function __construct(private Cache $cache) {}

    public function handle(): void
    {
        $this->cache->get('report:last');   // container đã inject implementation thật
    }
}
```

**Vì sao tồn tại / vì sao quan trọng:**
- **Loose coupling:** code của bạn phụ thuộc vào **abstraction** (interface) thay vì class cụ thể — đúng
  tinh thần **Dependency Inversion** (SOLID). Muốn thay implementation chỉ cần re-bind trong provider,
  không đụng code nghiệp vụ.
- **Dependency tường minh:** nhìn constructor là biết class cần gì — rõ ràng hơn Facade (vốn giấu phụ thuộc).
- **Dễ test:** type-hint interface → trong test chỉ cần inject một mock implement cùng contract.
- **Tài liệu hóa API:** contract mô tả gọn các method mà một service phải có — như một "hợp đồng" rõ ràng
  giữa người dùng và framework.

**Lưu ý quan trọng:** trong **đa số trường hợp KHÔNG cần tự bind** contract — Laravel đã bind sẵn contract
lõi với implementation mặc định. Bạn chỉ cần tự bind khi viết **interface của riêng mình** (vd
`UserRepository` → `EloquentUserRepository`) trong Service Provider.

> Liên hệ: Contract chính là phần "interface" trong cặp **interface → implementation** mà Service Container
> bind (câu 1) và là biểu hiện trực tiếp của **Dependency Inversion** — xem `Common/solid-design-pattern.md`.

### 🎯 Tóm tắt trả lời phỏng vấn

- Contract là **tập interface** (`Illuminate\Contracts`) định nghĩa các service cốt lõi của Laravel (Cache, Queue, Mail, Filesystem...).
- Container đã **bind sẵn** mỗi contract với implementation mặc định → chỉ cần **type-hint contract** là được inject đúng class.
- Lợi ích: **loose coupling**, dependency **tường minh**, **dễ test/mock**, tuân thủ **Dependency Inversion**.
- Thường **không phải tự bind** contract lõi; chỉ bind khi tạo **interface của riêng mình**.

[↑ Mục lục](#mục-lục)

<a id="q4-1"></a>
### 4.1. Contract vs Facade — khác nhau và khi nào dùng?

#### 📖 Giới thiệu kiến thức

Cả hai đều là cách truy cập **cùng một service** trong container, nhưng triết lý ngược nhau về **độ tường
minh của dependency**:

| | Contract (interface + DI) | Facade |
|---|---------------------------|--------|
| Cách dùng | Type-hint interface ở constructor → dùng `$this->cache` | Gọi static `Cache::get()` |
| Dependency | **Tường minh** ở constructor signature | **Ẩn** (nhìn thân method mới thấy) |
| Cú pháp | Dài hơn (phải khai báo, inject) | Rất gọn, không cần `use` đường dẫn dài |
| Coupling | Phụ thuộc **abstraction** → đổi implementation dễ | Phụ thuộc alias facade |
| Hợp nhất | Cả hai cùng resolve từ **Service Container** | |

**Điểm chung quan trọng:** Laravel docs nói rõ — **không có khác biệt thực tế về độ "test được"** giữa
Facade và Contract. Cả hai đều mock được. Việc chọn là **vấn đề phong cách**, miễn là nhất quán.

**Khi nào dùng cái nào (gợi ý thực tế):**
- **Contract / DI:** cho **class nghiệp vụ phức tạp**, nhiều dependency, hoặc khi xây **package** muốn
  giảm coupling tối đa và liệt kê rõ phụ thuộc.
- **Facade:** cho chỗ cần **gọn nhẹ, nhanh** — controller, route closure, prototyping.

```php
// Cùng truy cập Cache, hai phong cách:

// 1) Contract (DI) — tường minh
public function __construct(private \Illuminate\Contracts\Cache\Repository $cache) {}

// 2) Facade — gọn
\Cache::get('key');
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai truy cập **cùng service trong container**: **Contract** dùng type-hint interface (dependency **tường minh**), **Facade** gọi static (dependency **ẩn**, cú pháp gọn).
- Laravel khẳng định **không khác biệt về khả năng test** — chọn là **vấn đề phong cách**, cốt yếu nhất quán.
- Thực tế: **Contract/DI cho class/package phức tạp** (giảm coupling), **Facade cho chỗ tiện lợi** (controller, route).

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Cơ chế driver theo config (`CACHE_DRIVER`, `DB_CONNECTION`...) dùng design pattern gì?

### 📖 Giới thiệu kiến thức

Khi bạn đặt `CACHE_DRIVER=redis` (hay `file`, `memcached`, `array`...) thì `Cache::get()` tự động chạy
đúng driver tương ứng mà **không phải sửa một dòng code nghiệp vụ nào**. Đây là một mẫu thiết kế lặp lại
**khắp Laravel** cho mọi thứ cấu hình bằng "driver": `DB_CONNECTION`, `QUEUE_CONNECTION`, `MAIL_MAILER`,
`FILESYSTEM_DISK`, `SESSION_DRIVER`, `BROADCAST_DRIVER`...

Cơ chế này là sự kết hợp của **2 pattern**:

**1. Strategy Pattern (trọng tâm câu hỏi).**
Tất cả driver của cùng một loại đều **implement chung một interface (contract)** — vd cache là
`Illuminate\Contracts\Cache\Store`. Code nghiệp vụ chỉ phụ thuộc vào abstraction đó, không quan tâm
implementation cụ thể. Đổi `CACHE_DRIVER` chính là **tráo "chiến lược" (thuật toán) tại runtime** — đúng
định nghĩa Strategy: *định nghĩa một họ thuật toán, đóng gói từng cái, cho chúng thay thế lẫn nhau*.

```php
// Mọi driver cache đều tuân theo cùng một contract
interface Store {
    public function get($key);
    public function put($key, $value, $seconds);
    // ...
}
// RedisStore, FileStore, MemcachedStore, ArrayStore... đều implement Store
// → client gọi Cache::get() mà không cần biết đang chạy driver nào
```

**2. Factory / Manager Pattern (phần "tạo ra" driver).**
Việc *đọc config rồi quyết định khởi tạo class driver nào* do **`CacheManager`** đảm nhiệm. Laravel gọi đây
là **Manager pattern** — bản chất là một **Factory**: nó đóng gói logic khởi tạo, người dùng không tự `new`
driver. `CacheManager` được bind **singleton** trong Service Container (xem câu 1), đọc default driver từ
`config/cache.php` (vốn đọc từ biến môi trường `CACHE_DRIVER`).

> Liên hệ: "interface chung mà các driver implement" chính là **Contract** (câu 4); khả năng tráo
> implementation mà không sửa code là biểu hiện của **Dependency Inversion / Open–Closed** —
> xem `Common/solid-design-pattern.md` (Strategy & Factory).

### 🎯 Tóm tắt trả lời phỏng vấn

- Trọng tâm là **Strategy Pattern**: các driver (`redis`, `file`, `memcached`...) cùng implement một contract (`Store`), nên đổi `CACHE_DRIVER` chỉ là **tráo implementation tại runtime** mà không đụng code nghiệp vụ.
- Phần *chọn & tạo* driver theo config do **`CacheManager`** lo — đây là **Factory / Manager pattern**.
- Mẫu này lặp lại khắp Laravel: `DB_CONNECTION`, `QUEUE_CONNECTION`, `MAIL_MAILER`, `FILESYSTEM_DISK`, `SESSION_DRIVER`...
- Câu trả lời "ăn điểm": **Strategy là chính, Factory/Manager là điểm cộng**.

[↑ Mục lục](#mục-lục)

<a id="q5-1"></a>
### 5.1. Manager pattern hoạt động thế nào? Tự đăng ký custom driver ra sao?

#### 📖 Giới thiệu kiến thức

Các "Manager" của Laravel (`CacheManager`, `SessionManager`, `QueueManager`, `MailManager`...) phần lớn
kế thừa lớp cơ sở `Illuminate\Support\Manager`. Cơ chế:

- `getDefaultDriver()` đọc tên driver mặc định từ config (vd `config('cache.default')` ← `CACHE_DRIVER`).
- Với mỗi driver có sẵn, Manager có một method `create{Driver}Driver()` — vd `createRedisDriver()`,
  `createFileDriver()`. Khi resolve, Manager map tên driver → gọi đúng method tạo instance.
- Kết quả được **cache lại** (`$this->drivers[$name]`) để lần sau trả về cùng instance.

```php
// Rút gọn tinh thần của Illuminate\Support\Manager::driver()
public function driver($name = null)
{
    $name = $name ?: $this->getDefaultDriver();          // đọc CACHE_DRIVER
    return $this->drivers[$name] ??= $this->createDriver($name);  // factory + cache
}
```

**Tự thêm custom driver:** không cần sửa core — dùng method `extend()` (đăng ký một closure factory),
thường đặt trong `boot()` của Service Provider:

```php
public function boot(): void
{
    Cache::extend('mongo', function ($app) {
        // trả về một instance Repository bọc Store tự viết
        return Cache::repository(new MongoStore(/* ... */));
    });
}
// Giờ có thể đặt CACHE_DRIVER=mongo
```

Đây là minh họa đẹp cho **Open–Closed Principle**: mở rộng thêm driver mới **mà không sửa** code framework.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Manager (vd `CacheManager`) đọc driver mặc định từ config qua `getDefaultDriver()`, rồi gọi `create{Driver}Driver()` tương ứng để **tạo và cache** instance — đúng tinh thần **Factory**.
- Tự thêm driver mới bằng **`Cache::extend('ten', closure)`** (thường trong `boot()` của provider) → không phải sửa core.
- Cơ chế `extend` là ví dụ điển hình của **Open–Closed Principle**.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Vòng đời của một request trong Laravel

### 📖 Giới thiệu kiến thức

**Vòng đời request (request lifecycle)** mô tả **toàn bộ hành trình** của một HTTP request từ lúc trình
duyệt gửi đến, đi qua các thành phần của Laravel, cho tới khi trả về response. Hiểu nó giúp bạn biết
**middleware, service provider, container** chạy ở đâu — cực kỳ quan trọng khi debug và tối ưu.

**Sơ đồ tổng quát:**

```
Browser
  │  HTTP request
  ▼
public/index.php           ← điểm vào DUY NHẤT của mọi request
  │  (1) nạp Composer autoload + bootstrap/app.php → tạo Application (Service Container)
  ▼
HTTP Kernel                ← Illuminate\Foundation\Http\Kernel
  │  (2) chạy bootstrappers: load env, config, đăng ký Service Providers, facade, exception handler
  ▼
Middleware (global)        ← (3) request đi QUA pipeline middleware: maintenance, CSRF, auth...
  ▼
Router                     ← (4) match route → áp route middleware → gọi Controller/Closure
  ▼
Controller / Action        ← (5) logic nghiệp vụ, trả về Response
  │
  ▼  response đi NGƯỢC ra qua middleware
HTTP Kernel → index.php → Browser   ← (6) gửi response
  │
  ▼  (7) terminate(): chạy terminable middleware (lưu session, ghi log...) SAU khi đã gửi response
```

**Giải thích từng bước (chi tiết):**

**1. Web server → Entry point `public/index.php`.** Nginx/Apache được cấu hình trỏ document root về
thư mục `public/`, nên **mọi** request (`/`, `/users`, `/api/...`) đều đổ về **một file duy nhất**
`index.php` (mô hình **Front Controller**). File này rất ngắn:

```php
define('LARAVEL_START', microtime(true));

require __DIR__.'/../vendor/autoload.php';      // (a) nạp Composer autoload (PSR-4)

$app = require_once __DIR__.'/../bootstrap/app.php';   // (b) tạo Application (Service Container)

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);  // (c) resolve HTTP Kernel từ container

$response = $kernel->handle(                      // (d) đẩy request vào Kernel → nhận Response
    $request = Illuminate\Http\Request::capture()  // tạo Request từ biến toàn cục PHP ($_GET, $_POST...)
);

$response->send();                                // (e) gửi response ra browser

$kernel->terminate($request, $response);          // (f) chạy việc "hậu kỳ" sau khi đã gửi
```

**2. `bootstrap/app.php` → tạo Application (IoC Container).** Đây là nơi khởi tạo
`Illuminate\Foundation\Application` — **Service Container** trung tâm. Nó bind sẵn **HTTP Kernel**,
**Console Kernel** và **Exception Handler** vào container. Container này về sau lo việc **resolve &
inject** mọi dependency (xem DI/IoC).

**3. HTTP Kernel chạy các bootstrappers.** `$kernel->handle()` trước tiên gọi `bootstrap()` để chạy
**6 bootstrapper theo thứ tự** — đây là phần "dựng sân khấu":

| # | Bootstrapper | Việc làm |
|---|--------------|----------|
| 1 | `LoadEnvironmentVariables` | Đọc file `.env` |
| 2 | `LoadConfiguration` | Gộp toàn bộ `config/*.php` |
| 3 | `HandleExceptions` | Đăng ký error/exception handler |
| 4 | `RegisterFacades` | Bật **Facades** (`Cache::`, `DB::`...) |
| 5 | `RegisterProviders` | Gọi `register()` của **mọi Service Provider** |
| 6 | `BootProviders` | Gọi `boot()` của mọi Service Provider |

- **Service Provider là trái tim của bootstrap:** `register()` **chỉ bind** service vào container
  (KHÔNG được dùng service khác vì có thể chưa register xong); `boot()` chạy **sau khi tất cả đã
  register** nên được phép dùng mọi thứ (đăng ký route, event, view composer...). Provider được liệt kê
  ở `config/app.php` (hoặc `bootstrap/providers.php` từ Laravel 11). Có **deferred provider** chỉ load
  khi service được dùng → tối ưu khởi động.

**4. Request đi qua Middleware pipeline.** Sau bootstrap, request được đẩy qua một **pipeline** middleware
theo mô hình "củ hành" (onion) — đi **vào** qua từng lớp, rồi **đi ra** ngược lại:

```php
// Bản chất: mỗi middleware bọc middleware kế tiếp (giống Decorator/Pipeline)
return $middleware->handle($request, function ($request) use ($next) {
    return $next($request);   // gọi lớp tiếp theo; code TRƯỚC dòng này chạy lượt vào, SAU chạy lượt ra
});
```

Phân tầng middleware: **global** (chạy mọi request — vd `TrustProxies`, `HandleCors`,
`ValidatePostSize`) → **group** `web` (vd `EncryptCookies`, `StartSession`, `VerifyCsrfToken`) hoặc
`api` (vd `throttle`) → **route middleware** (vd `auth`, `can`). Thứ tự có thể bị chi phối bởi
`$middlewarePriority`.

**5. Router dispatch → Controller.** Khi đã qua middleware "trước", **Router** so khớp URL + HTTP method
với route đăng ký, áp **route middleware** riêng, rồi giải quyết handler. Tại đây diễn ra:
- **Route–Model Binding:** `Route::get('/users/{user}')` + type-hint `User $user` → Laravel **tự query**
  model theo id.
- **Dependency Injection:** tham số controller được **container tự resolve** (vd inject `Request`,
  service, repository).
- **FormRequest validation:** nếu type-hint một `FormRequest`, **validate chạy trước** khi vào thân
  controller.

**6. Controller → Response.** Controller chạy logic nghiệp vụ (gọi service/Eloquent...) và trả về một
**Response** (view Blade, JSON, redirect...). Laravel chuẩn hóa nó thành `Illuminate\Http\Response`.

**7. Response đi ngược ra qua middleware.** Response **đi ngược** qua các lớp middleware (phần code
"sau `$next`" chạy ở lượt này — vd thêm header, mã hóa cookie), về Kernel, về `index.php`, và
`$response->send()` **gửi ra browser**.

**8. Terminate (hậu kỳ).** Sau khi đã gửi response, Kernel gọi `terminate()` để chạy **terminable
middleware** — các việc "dọn dẹp" như **lưu session** (`StartSession`), ghi log, mà **không bắt user
phải chờ** (response đã đi rồi).

**Lưu ý phiên bản (Laravel 11+):** skeleton mới **bỏ `app/Http/Kernel.php`** lộ liễu; phần cấu hình
(middleware, route, exception) **gom vào `bootstrap/app.php`** qua
`Application::configure()->withMiddleware(...)->withRouting(...)->create()`. **Bản chất 8 bước trên
KHÔNG đổi** — Kernel vẫn tồn tại bên trong framework, chỉ là nơi khai báo cấu hình được gom gọn lại.

### 🎯 Tóm tắt trả lời phỏng vấn

- Mọi request vào **một entry point `public/index.php`** → nạp autoload + `bootstrap/app.php` để tạo
  **Application (Service Container)**.
- Request được đưa vào **HTTP Kernel**, Kernel chạy **bootstrappers**: load env/config, **đăng ký &
  boot Service Providers** (register trước, boot sau), facade, exception handler.
- Request **đi qua pipeline middleware** (maintenance, CSRF, auth...) → tới **Router** match route, áp
  route middleware → gọi **Controller**.
- Controller trả **Response**, response **đi ngược ra qua middleware** rồi **gửi về browser**.
- Cuối cùng `terminate()` chạy **terminable middleware** (lưu session, log) **sau khi** đã trả response.
- **Laravel 11+:** cấu hình gom vào `bootstrap/app.php` (bỏ `Kernel.php` lộ ra ngoài) nhưng **bản chất
  vòng đời không đổi**.

[↑ Mục lục](#mục-lục)

<a id="q6-1"></a>
### 6.1. Phân biệt `get()` và `input()` khi lấy dữ liệu Request

#### 📖 Giới thiệu kiến thức

Cả hai đều dùng để **đọc dữ liệu người dùng gửi lên** từ object `Illuminate\Http\Request`, nhưng **nguồn
lấy** và **khả năng** khác nhau — đây là điểm hay nhầm.

**`input($key, $default)` — method của Laravel** (trait `InteractsWithInput`):

- Lấy từ **cả query string LẪN body** (form-data, `x-www-form-urlencoded`, và **JSON** — tự decode), bất
  kể HTTP method.
- Hỗ trợ **dot notation** cho mảng/JSON lồng nhau: `input('user.name')`, `input('items.0.id')`.
- Có **giá trị mặc định**: `input('page', 1)`.
- Là cách **khuyến nghị** trong Laravel.

**`get($key, $default)` — KHÔNG phải method của Laravel mà kế thừa từ Symfony** Request:

- Tra cứu theo thứ tự: **route parameters (attributes) → query → body** → có thể **vô tình trả về route
  parameter** trùng tên (bẫy kinh điển).
- **KHÔNG hỗ trợ dot notation**.
- Với request **JSON**, body không được nạp vào ParameterBag của Symfony → `get()` thường **không lấy
  được** field JSON (trong khi `input()` lấy được).

```php
// JSON body: { "user": { "name": "An" } } ; URL: /submit?page=2

$request->input('user.name');   // "An"   (dot notation + đọc JSON)
$request->input('page', 1);     // "2"    (có default)

$request->get('user.name');     // null   (không hỗ trợ dot notation)
$request->get('page');          // "2"    (lấy từ query)

// BẪY route param: Route::post('/post/{page}', ...) gọi /post/99?page=2
$request->get('page');          // "99"   ← trả ROUTE param, KHÔNG phải query!
$request->input('page');        // "2"    ← lấy đúng input
```

**Các method liên quan (chọn đúng nguồn dữ liệu):**

- `query('k')` — **chỉ** query string; `post('k')` — **chỉ** body (Symfony).
- `all()`, `only([...])`, `except([...])`, `has('k')`, `filled('k')`, `boolean('k')`, `integer('k')`.
- `$request->name` — dynamic property, tương đương `input('name')`.

> Kết luận: trong Laravel **ưu tiên `input()`** (hoặc `query()`/`post()` khi cần đúng nguồn); **tránh
> `get()`** vì precedence route-param + thiếu hỗ trợ JSON/dot-notation dễ gây bug khó hiểu.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`input()`** = method **Laravel**: lấy từ **query + body (kể cả JSON)**, hỗ trợ **dot notation** +
  **default**, hành vi nhất quán → **nên dùng**.
- **`get()`** = method **kế thừa Symfony**: thứ tự **route param → query → body**, **không** dot notation,
  **không** đọc JSON body tốt → dễ **vô tình lấy route parameter** trùng tên. Hạn chế dùng.
- Lấy đúng nguồn: **`query()`** (chỉ query string), **`post()`** (chỉ body); kèm
  `all/only/except/has/filled`.
- Câu chốt: trong Laravel ưu tiên **`input()`/`query()`**, tránh `get()`.

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. Có mấy loại middleware trong Laravel?

### 📖 Giới thiệu kiến thức

**Middleware** là các "lớp lọc" mà HTTP request **đi xuyên qua** trước khi tới controller (và response
đi ngược ra) — mô hình "củ hành" (onion) trong request lifecycle (xem câu 6). Mỗi middleware có thể
**kiểm tra, sửa, hoặc chặn** request (vd: chưa đăng nhập → redirect về `/login`).

Dựa trên **phạm vi áp dụng & cách đăng ký**, Laravel có **3 loại** middleware:

**1. Global middleware — chạy cho MỌI request.**
Áp dụng cho toàn bộ HTTP request của ứng dụng, không cần gán vào route. Ví dụ có sẵn: `TrustProxies`,
`HandleCors`, `ValidatePostSize`, `PreventRequestsDuringMaintenance`.

**2. Group middleware — chạy cho một NHÓM route.**
Gom nhiều middleware dưới một tên nhóm để áp một lượt. Hai nhóm mặc định:
- **`web`**: `EncryptCookies`, `StartSession`, `VerifyCsrfToken`, `ShareErrorsFromSession`... (cho route web có session, cookie, CSRF).
- **`api`**: thường có `throttle:api`, `SubstituteBindings` (cho API stateless).

**3. Route middleware (alias middleware) — gán cho route/controller CỤ THỂ.**
Được đặt **alias** (tên ngắn) rồi gán vào từng route khi cần: `auth`, `verified`, `can`, `throttle`,
`guest`...

```php
// Gán theo alias trên route
Route::get('/profile', [ProfileController::class, 'show'])
    ->middleware(['auth', 'verified']);

// Áp cả nhóm
Route::middleware('web')->group(function () { /* ... */ });
```

**Đăng ký ở đâu — khác nhau theo phiên bản:**

```php
// Laravel ≤ 10: trong app/Http/Kernel.php
protected $middleware = [...];               // global
protected $middlewareGroups = ['web' => [...], 'api' => [...]];  // group
protected $middlewareAliases = ['auth' => Authenticate::class];   // alias/route

// Laravel 11+: gom vào bootstrap/app.php
->withMiddleware(function (Middleware $middleware) {
    $middleware->append(EnsureTokenIsValid::class);      // thêm global
    $middleware->appendToGroup('web', VerifyAge::class); // thêm vào group
    $middleware->alias(['admin' => EnsureUserIsAdmin::class]); // đặt alias
})
```

```bash
php artisan make:middleware EnsureUserIsAdmin
```

> Liên hệ: vị trí middleware trong toàn bộ hành trình request xem câu 6 (bước 4). Thứ tự chạy giữa các
> middleware xem mục 7.2 (priority).

### 🎯 Tóm tắt trả lời phỏng vấn

- Theo phạm vi/cách đăng ký có **3 loại**: **global** (chạy mọi request), **group** (theo nhóm route — `web`, `api`), **route/alias** (gán cho route cụ thể qua tên ngắn như `auth`, `throttle`).
- Middleware lọc request theo mô hình "củ hành": chạy lượt **vào** trước controller, lượt **ra** sau khi có response.
- Đăng ký: Laravel ≤10 ở `app/Http/Kernel.php` (`$middleware` / `$middlewareGroups` / `$middlewareAliases`); Laravel 11+ gom vào `bootstrap/app.php` qua `withMiddleware()`.
- Tạo bằng `php artisan make:middleware`.

[↑ Mục lục](#mục-lục)

<a id="q7-1"></a>
### 7.1. Before vs After middleware & terminable middleware

#### 📖 Giới thiệu kiến thức

Ngoài cách phân loại theo phạm vi, middleware còn phân theo **thời điểm chạy** so với controller — quyết
định bởi việc bạn đặt logic **trước hay sau** lời gọi `$next($request)`:

```php
class ExampleMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        // === BEFORE: chạy TRƯỚC controller (lượt "vào") ===
        // vd: kiểm tra auth, rate limit → có thể chặn luôn, không gọi $next

        $response = $next($request);   // đẩy request đi tiếp vào trong

        // === AFTER: chạy SAU controller (lượt "ra") ===
        // vd: thêm header, log, sửa response
        $response->headers->set('X-Custom', 'value');

        return $response;
    }
}
```

- **Before middleware:** xử lý **trước** khi request tới controller — phù hợp việc *gác cổng* (auth,
  CSRF, rate limit). Có thể **chặn** request bằng cách trả response ngay mà không gọi `$next`.
- **After middleware:** xử lý **sau** khi controller trả response — phù hợp việc *hậu xử lý* response
  (thêm header, nén, ghi log).

**Terminable middleware** là loại đặc biệt: có thêm method `terminate()` chạy **sau khi response ĐÃ được
gửi về browser** (ở bước `$kernel->terminate()` — xem câu 6). Dùng cho việc nặng/không cần user chờ:

```php
class StartSession
{
    public function handle($request, Closure $next) { return $next($request); }

    // chạy sau khi response đã gửi đi → user không phải đợi
    public function terminate($request, $response): void
    {
        // vd: lưu session xuống storage, ghi log
    }
}
```

> Lưu ý: terminable chỉ chạy khi dùng FPM/web server truyền thống (response gửi xong mới terminate).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Before middleware**: logic **trước** `$next()` → gác cổng (auth, rate limit), có thể **chặn** request.
- **After middleware**: logic **sau** `$next()` → hậu xử lý response (thêm header, log).
- **Terminable middleware**: có method **`terminate()`** chạy **sau khi response đã gửi** → việc nặng không bắt user chờ (lưu session, log). Đây là bước cuối của vòng đời request.

[↑ Mục lục](#mục-lục)

<a id="q7-2"></a>
### 7.2. Truyền tham số cho middleware & middleware priority

#### 📖 Giới thiệu kiến thức

**Middleware có tham số (parameters):** một middleware có thể nhận thêm đối số sau `$next` để tái sử dụng
cho nhiều ngữ cảnh. Truyền qua cú pháp `:` và phân tách bằng `,` trên route:

```php
class EnsureRole
{
    public function handle(Request $request, Closure $next, string ...$roles)
    {
        if (! $request->user()?->hasAnyRole($roles)) {
            abort(403);
        }
        return $next($request);
    }
}

// Trên route: truyền tham số sau dấu ':'
Route::put('/post/{id}', ...)->middleware('role:admin,editor');
```

**Middleware priority (thứ tự ưu tiên):** khi nhiều middleware cùng áp lên một route, **thứ tự chạy có thể
quan trọng** (vd `StartSession` phải chạy trước `Authenticate` vì auth cần session). Mặc định middleware
chạy theo thứ tự khai báo, nhưng Laravel có danh sách **`$middlewarePriority`** ép một số middleware lõi
chạy đúng trình tự bất kể khai báo. Laravel 11+ chỉnh qua `$middleware->priority([...])` trong
`bootstrap/app.php`.

```php
// Laravel 11+: ép thứ tự ưu tiên
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\Auth\Middleware\Authenticate::class,
        // ...
    ]);
})
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- Middleware **nhận tham số** qua `$next` rồi truyền trên route bằng cú pháp `'role:admin,editor'` → tái sử dụng một middleware cho nhiều ngữ cảnh.
- Khi nhiều middleware chồng nhau, **thứ tự chạy** có thể quyết định đúng/sai (vd session phải trước auth).
- Laravel dùng **`$middlewarePriority`** (Laravel 11+: `$middleware->priority([...])`) để ép một số middleware lõi chạy đúng trình tự bất kể thứ tự khai báo.

[↑ Mục lục](#mục-lục)

<a id="q7-3"></a>
### 7.3. Middleware lọc request ở đâu? Cơ chế lọc (Pipeline) hoạt động thế nào?

#### 📖 Giới thiệu kiến thức

**Câu hỏi 1 — lọc ở ĐÂU?** Request không bị lọc rải rác; nó được đẩy qua middleware tại **HTTP Kernel**,
**sau bootstrap nhưng trước khi tới controller**. Cụ thể có **2 chặng lọc**:

```
index.php → Kernel::handle()
   └─ sendRequestThroughRouter():
        (A) chạy GLOBAL middleware (cho mọi request)   ← chặng 1, trong Kernel
        └─ Router::dispatch() → match route
             └─ runRouteWithinStack():
                  (B) chạy GROUP + ROUTE middleware     ← chặng 2, trong Router
                       └─ Controller / action
```

- **Chặng 1 (trong Kernel):** global middleware chạy trước, **chưa cần biết** route nào.
- **Chặng 2 (trong Router):** sau khi match route mới biết route đó thuộc group nào (`web`/`api`) và có
  route middleware gì (`auth`...) → chạy tiếp các middleware đó rồi mới vào controller.

**Câu hỏi 2 — CƠ CHẾ lọc?** Cả hai chặng đều dùng cùng một công cụ: lớp **`Illuminate\Pipeline\Pipeline`**.
Đây là hiện thực của **Pipeline / Chain pattern** theo mô hình "củ hành": request đi **xuyên qua** từng lớp,
mỗi lớp tự quyết định cho đi tiếp (`$next`) hay chặn lại.

```php
// Tinh thần trong Kernel::sendRequestThroughRouter()
return (new Pipeline($this->app))
    ->send($request)                 // gửi request vào ống
    ->through($this->middleware)     // qua mảng middleware
    ->then(fn ($request) => $this->router->dispatch($request)); // đích cuối: router/controller
```

**Bí mật bên trong — `array_reduce` cuộn các closure lồng nhau (chính là "củ hành"):**

```php
// Rút gọn Pipeline::then()
protected function then(Closure $destination)
{
    $pipeline = array_reduce(
        array_reverse($this->pipes),  // duyệt NGƯỢC để lớp đầu tiên bọc ngoài cùng
        $this->carry(),               // hàm bọc mỗi middleware quanh phần còn lại
        $destination                  // lõi trong cùng: gọi controller
    );
    return $pipeline($this->passable);  // kích hoạt: gọi lớp ngoài cùng → lan vào trong
}

// carry() trả về closure biến mỗi middleware thành một lớp bọc:
//   fn ($passable) => $pipe->handle($passable, $nextLayer)
// → $next chính là "lớp tiếp theo" đã được dựng sẵn
```

Kết quả là một chuỗi hàm lồng nhau, đại khái:

```php
$global1->handle($req, fn ($req) =>
    $global2->handle($req, fn ($req) =>
        $router->dispatch($req)   // lõi
    )
);
```

Mỗi middleware:
- Code **trước `$next($request)`** chạy ở **lượt đi vào** (lọc/chặn request).
- Gọi `$next($request)` để **đẩy vào lớp trong**; **không gọi** `$next` ⇒ **chặn**, request không bao giờ tới controller.
- Code **sau `$next(...)`** chạy ở **lượt đi ra** (sửa response).

```php
public function handle(Request $request, Closure $next)
{
    if (! $request->user()) {
        return redirect('/login');   // CHẶN: trả response luôn, không gọi $next → cắt ống
    }
    return $next($request);          // CHO ĐI TIẾP vào lớp trong
}
```

> Liên hệ: đây là cùng cơ chế "onion" mô tả ở câu 6 (bước 4) — chỉ khác là ở đây ta nhìn vào **lớp
> Pipeline + `array_reduce`** đứng sau nó. Pipeline pattern cũng được nhắc trong `Common/solid-design-pattern.md`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Request bị lọc ở **HTTP Kernel, sau bootstrap & trước controller**, qua **2 chặng**: **global middleware** chạy trong Kernel; **group + route middleware** chạy trong **Router** sau khi đã match route.
- Cơ chế là lớp **`Pipeline`** (Pipeline/Chain pattern, mô hình "củ hành"): `send()->through()->then()`.
- Bên trong dùng **`array_reduce`** cuộn các middleware thành **closure lồng nhau**; mỗi lớp nhận `$next` là lớp kế tiếp.
- Code **trước `$next()`** chạy lượt vào (lọc/chặn), **sau `$next()`** chạy lượt ra (sửa response); **không gọi `$next` = chặn request**, không tới controller.

[↑ Mục lục](#mục-lục)

<a id="q7-4"></a>
### 7.4. CSRF token & VerifyCsrfToken middleware trong Laravel

#### 📖 Giới thiệu kiến thức

**CSRF (Cross-Site Request Forgery)** là tấn công lừa trình duyệt nạn nhân (đang đăng nhập) gửi một request
ngoài ý muốn (khái niệm chung xem `Common/security.md`). Laravel **chống CSRF sẵn** bằng **synchronizer
token pattern**: sinh một **token ngẫu nhiên** cho mỗi phiên, gắn vào form, và **kiểm tra** ở mỗi request
làm thay đổi trạng thái.

**Cơ chế — `VerifyCsrfToken` middleware (nằm trong group `web`):**
Với mọi request `POST`/`PUT`/`PATCH`/`DELETE`, middleware so khớp token gửi lên với token lưu trong session.
Không khớp → ném `419 Page Expired`. Request `GET`/`HEAD`/`OPTIONS` (chỉ đọc) **không** bị kiểm tra.

**Gắn token vào form (Blade):**

```blade
<form method="POST" action="/profile">
    @csrf   {{-- sinh <input type="hidden" name="_token" value="..."> --}}
    ...
</form>
```

`@csrf` tương đương `csrf_field()`; lấy giá trị token thô bằng helper `csrf_token()` hoặc `Session::token()`.

**Với AJAX / JavaScript:** gửi token qua header **`X-CSRF-TOKEN`** (thường nhúng token vào thẻ `<meta>` rồi
đọc ra), hoặc dùng cookie **`XSRF-TOKEN`** + header **`X-XSRF-TOKEN`** (axios tự động làm việc này):

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

```js
fetch('/profile', {
    method: 'POST',
    headers: { 'X-CSRF-TOKEN': document.querySelector('meta[name=csrf-token]').content },
});
```

**Loại trừ route khỏi CSRF (vd webhook bên thứ ba):** khai báo `$except` (Laravel ≤10) hoặc
`validateCsrfTokens(except: [...])` trong `bootstrap/app.php` (Laravel 11+):

```php
// Laravel 11+ (bootstrap/app.php)
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: ['stripe/*', 'webhook/*']);
})
```

**Lưu ý quan trọng (hay hỏi):**
- CSRF token bảo vệ **request đổi trạng thái** từ form/web (dùng session-cookie). **API thuần stateless**
  (token Bearer/Sanctum API token) **không cần** CSRF — vì không dựa vào cookie tự gửi.
- **SPA cùng domain** dùng **Sanctum**: Sanctum dựa trên cookie nên **vẫn cần** CSRF (gọi
  `/sanctum/csrf-cookie` để nhận `XSRF-TOKEN`).
- Lỗi **419** thường do thiếu `@csrf`, session hết hạn, hoặc cache trang cũ.
- Nếu site **dính XSS** thì token cũng vô dụng (JS độc đọc được token) → chống XSS là tiền đề.

> Liên hệ: đây là hiện thực Laravel của khái niệm **CSRF** trong `Common/security.md` (câu 5 ở file đó);
> bản thân `VerifyCsrfToken` là một **group middleware** của `web` (xem câu 7).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Laravel chống CSRF bằng **synchronizer token**: middleware **`VerifyCsrfToken`** (group `web`) kiểm tra
  token ở request **POST/PUT/PATCH/DELETE**; sai → **419**. `GET/HEAD/OPTIONS` không kiểm tra.
- Gắn token: **`@csrf`** (`csrf_field()`) trong form; AJAX gửi qua header **`X-CSRF-TOKEN`** (đọc từ
  `<meta>`/`csrf_token()`) hoặc cookie **`XSRF-TOKEN`** → header **`X-XSRF-TOKEN`** (axios tự làm).
- Loại trừ route bằng **`$except`** / **`validateCsrfTokens(except: [...])`** (Laravel 11+) — vd webhook.
- **API stateless (Bearer/Sanctum token) không cần CSRF**; **SPA cùng domain dùng Sanctum (cookie) vẫn
  cần** (gọi `/sanctum/csrf-cookie`). Khái niệm chung: xem `Common/security.md`.

[↑ Mục lục](#mục-lục)

<a id="q7-5"></a>
### 7.5. Có áp dụng CSRF cho nhóm route `api` không? Có cần thiết không?

#### 📖 Giới thiệu kiến thức

**1. Có làm được không? → Về kỹ thuật là CÓ, nhưng mặc định Laravel KHÔNG.**
`VerifyCsrfToken` chỉ nằm trong group **`web`**; group **`api`** **không** có nó. Bạn *có thể* tự thêm
middleware CSRF vào group `api`, nhưng làm vậy đi ngược thiết kế **stateless** của API và hầu như luôn là
sai hướng.

**2. Có cần thiết không? → Phụ thuộc CƠ CHẾ XÁC THỰC, đa số trường hợp là KHÔNG.**

Mấu chốt: CSRF chỉ nguy hiểm khi xác thực dựa vào **cookie tự động gửi kèm**. Vì trình duyệt **tự đính
cookie** vào mọi request tới domain (kể cả request từ site khác), kẻ tấn công mới "mượn" được phiên.

- **API stateless dùng token trong header** (Bearer token / **Sanctum personal access token** / **Passport
  OAuth**): token nằm ở `Authorization: Bearer ...`, **KHÔNG tự động gửi kèm** như cookie → site ngoài
  **không thể** đính token của nạn nhân vào request → **không có nguy cơ CSRF → KHÔNG cần** token CSRF.
  Thêm CSRF ở đây chỉ gây phiền (client phải đi lấy token, phá vỡ tính stateless).

- **Khi nào API LẠI cần CSRF?** Khi API được gọi từ **SPA cùng domain xác thực bằng cookie/session** —
  điển hình **Laravel Sanctum (SPA authentication)**. Lúc này request dựa vào session-cookie nên **vẫn cần
  CSRF**. Nhưng bạn **không tự gắn** `VerifyCsrfToken` vào `api`; **Sanctum lo sẵn** qua middleware
  `EnsureFrontendRequestsAreStateful` + cơ chế cookie `XSRF-TOKEN` (frontend gọi `/sanctum/csrf-cookie`
  trước để nhận token).

**Quy tắc dễ nhớ:**

| Cơ chế auth của API | Có nguy cơ CSRF? | Cần CSRF token? |
|---------------------|:---:|:---:|
| Token trong header (Bearer / Sanctum token / OAuth) | Không | **Không** (và không nên thêm) |
| Cookie/session (SPA cùng domain — Sanctum SPA) | Có | **Có** (Sanctum xử lý sẵn) |

> Tư duy chốt: **CSRF gắn với cookie, không gắn với "đây là route web hay api".** Hỏi "request này xác thực
> bằng gì?" — header token thì bỏ CSRF; cookie/session thì cần CSRF.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Được không:** kỹ thuật thì CÓ thể thêm CSRF vào group `api`, nhưng mặc định Laravel **không** (api được
  thiết kế **stateless**, CSRF chỉ ở group `web`).
- **Cần không:** **đa số là KHÔNG.** API stateless xác thực bằng **token trong header** (Bearer/Sanctum
  token/OAuth) → token **không tự gửi kèm** như cookie → **không có nguy cơ CSRF → không cần**.
- **Ngoại lệ CẦN:** API phục vụ **SPA cùng domain dùng cookie/session** (**Laravel Sanctum SPA**) → vẫn
  cần CSRF, và **Sanctum xử lý sẵn** (`EnsureFrontendRequestsAreStateful` + `/sanctum/csrf-cookie`).
- **Quy tắc:** CSRF gắn với **cookie**, không gắn với việc route là web hay api → auth **header token** bỏ
  CSRF, auth **cookie/session** cần CSRF.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. Eloquent và Query Builder trong Laravel là gì?

### 📖 Giới thiệu kiến thức

Laravel cung cấp **2 tầng (layer) để làm việc với database**, xếp chồng lên nhau:

**1. Query Builder** — API "fluent" (nối chuỗi method) để **dựng câu SQL bằng PHP** mà không phải viết
SQL thô. Nó làm việc trực tiếp với **bảng/row** (trả về object `stdClass` thuần), **không gắn với model**.

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->where('active', 1)
    ->orderBy('name')
    ->get();                       // Collection các stdClass

DB::table('users')->where('id', 1)->update(['active' => 0]);
```

Query Builder dùng **PDO parameter binding** ⇒ tự chống **SQL Injection** (xem `Common/security.md`).

**2. Eloquent ORM** — **ORM theo mẫu Active Record**: mỗi **bảng** ánh xạ tới một **model class**, mỗi
**row** là một **instance** của model. Bạn thao tác với **object** thay vì câu SQL.

```php
// app/Models/User.php  ↔  bảng users
$users = User::where('active', 1)->orderBy('name')->get();  // Collection các User
$user  = User::find(1);
$user->active = 0;
$user->save();                     // Active Record: object tự biết cách lưu chính nó
```

**Quan hệ giữa hai cái (mấu chốt):** Eloquent **xây trên** Query Builder. Khi gọi `User::where(...)`,
Eloquent ủy thác (delegate) xuống một query builder bên dưới (`Eloquent\Builder` bọc `Query\Builder`), rồi
**hydrate** kết quả thành các model. Nhờ vậy hầu hết method của Query Builder (`where`, `join`, `orderBy`,
`select`...) đều dùng được trên Eloquent.

#### Eloquent thêm gì so với Query Builder?

- **Model = object có hành vi:** accessor/mutator, casts, `$fillable`/`$guarded` (mass-assignment
  protection), giá trị mặc định.
- **Relationship:** `hasOne`, `hasMany`, `belongsTo`, `belongsToMany`, `morph...` + **eager loading**
  (`with()`) chống N+1 (xem 8.5).
- **Events & Observer** (xem câu 12), **Soft Deletes**, **timestamps tự động**, **Query Scopes** (xem câu 10),
  **Serialization** (`toArray`/`toJson`), **Factory/Seeder**.

Đổi lại, Eloquent **nặng hơn** (phải hydrate object, dùng magic method/Reflection) → với truy vấn cực lớn
hoặc thao tác hàng loạt, Query Builder (hoặc raw SQL) **nhanh và tốn ít bộ nhớ hơn**.

#### Khi nào dùng cái nào?

| | Eloquent | Query Builder |
|---|----------|---------------|
| Đơn vị làm việc | **Model object** (Active Record) | **Bảng/row** thuần (`stdClass`) |
| Tốt cho | CRUD nghiệp vụ, có quan hệ, cần event/scope/cast | Báo cáo, aggregate phức tạp, bulk update, join nhiều bảng |
| Hiệu năng | Tiện nhưng nặng hơn (hydrate object) | Nhẹ, nhanh hơn cho tập dữ liệu lớn |
| Khả đọc | Cao, hướng đối tượng | Gần với SQL |

Thực tế: **mặc định dùng Eloquent** cho domain logic; **rớt xuống Query Builder / raw** khi cần hiệu năng
cho query nặng, aggregate, hoặc thao tác hàng loạt.

> Liên hệ: cả hai đều tham số hoá truy vấn (PDO) ⇒ an toàn trước SQL Injection. Eloquent là minh hoạ trực
> tiếp của **Active Record pattern**.

### 🎯 Tóm tắt trả lời phỏng vấn

- Laravel có **2 tầng truy cập DB**: **Query Builder** (API fluent dựng SQL, làm việc với **bảng/row
  thuần `stdClass`**) và **Eloquent ORM** (theo **Active Record**: bảng ↔ model, row ↔ object).
- **Eloquent xây trên Query Builder** → dùng lại được `where/join/orderBy...`, nhưng kết quả được
  **hydrate thành model** và có thêm **relationship, eager loading, events/observer, soft delete, casts,
  scopes, timestamps**.
- **Đánh đổi:** Eloquent tiện & OOP nhưng **nặng hơn**; Query Builder/raw **nhẹ & nhanh** cho query lớn,
  aggregate, bulk update.
- **Chọn:** mặc định Eloquent cho nghiệp vụ; chuyển sang Query Builder/raw khi cần hiệu năng hoặc truy vấn
  phức tạp/hàng loạt. Cả hai đều **chống SQL Injection** nhờ parameter binding.

[↑ Mục lục](#mục-lục)

<a id="q8-1"></a>
### 8.1. Xem/đổi Eloquent & Query Builder sang raw SQL (để debug)

#### 📖 Giới thiệu kiến thức

Cả Eloquent lẫn Query Builder cuối cùng đều sinh ra **một câu SQL + mảng binding**. Khi debug, ta cần
xem chính xác câu SQL đó. Laravel cung cấp nhiều cách:

**1. `->toSql()` + `->getBindings()` — xem SQL (chưa chạy).**
`toSql()` trả về chuỗi SQL với **placeholder `?`** (KHÔNG kèm giá trị); `getBindings()` trả mảng giá trị.

```php
$query = User::where('active', 1)->where('age', '>', 18);

$query->toSql();
// "select * from `users` where `active` = ? and `age` > ?"

$query->getBindings();   // [1, 18]
```

**2. `->toRawSql()` — SQL đã chèn sẵn giá trị (Laravel 10.15+).** Tiện để copy chạy thẳng trong DB client.

```php
$query->toRawSql();
// select * from `users` where `active` = 1 and `age` > 18
```

**3. `->dd()` / `->dump()` — in tại chỗ.** `dd()` in SQL + bindings rồi **dừng**; `dump()` in rồi **chạy
tiếp**. Có biến thể raw: `->ddRawSql()` / `->dumpRawSql()` (Laravel 10.15+).

```php
User::where('active', 1)->dd();          // in SQL + bindings rồi dừng
User::where('active', 1)->dumpRawSql();  // in SQL đã chèn giá trị, chạy tiếp
```

**4. Bắt MỌI query thực sự chạy — query log.** `toSql()` chỉ là chuỗi (query chưa chạy); muốn xem các
query **đã thực thi** (kèm thời gian) thì dùng query log hoặc listener:

```php
DB::enableQueryLog();
$users = User::with('posts')->get();
dump(DB::getQueryLog());
// [ ['query' => 'select * from ...', 'bindings' => [...], 'time' => 1.23], ... ]

// Hoặc lắng nghe realtime mọi query
DB::listen(fn ($q) => logger($q->sql, $q->bindings));   // $q->time = ms
```

**5. Công cụ ngoài:** **Laravel Debugbar** và **Telescope** hiển thị mọi query + bindings + thời gian theo
từng request — tiện nhất để soi N+1 (xem 8.5) và query chậm.

> Lưu ý: `toSql()` **không thực thi** query nên không tốn DB; còn `dd()`/query log phản ánh query thật sự
> chạy. Khi báo cáo bug, nên đưa cả **SQL + bindings** (hoặc `toRawSql`) để tái hiện chính xác.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`->toSql()`** trả SQL với `?` (chưa chạy) + **`->getBindings()`** xem giá trị; **`->toRawSql()`**
  (Laravel 10.15+) cho SQL đã chèn sẵn giá trị để copy chạy thử.
- In tại chỗ: **`->dd()` / `->dump()`** (và `->ddRawSql()` / `->dumpRawSql()`).
- Xem query **đã thực thi**: **`DB::enableQueryLog()` + `DB::getQueryLog()`**, hoặc realtime
  **`DB::listen()`** (có `sql`, `bindings`, `time`).
- Công cụ trực quan: **Laravel Debugbar / Telescope** — soi nhanh số query, bindings, thời gian (phát hiện
  N+1, query chậm).

[↑ Mục lục](#mục-lục)

<a id="q8-2"></a>
### 8.2. Laravel Debugbar là gì?

#### 📖 Giới thiệu kiến thức

**Laravel Debugbar** (package `barryvdh/laravel-debugbar`) tích hợp thư viện **PHP Debug Bar** vào Laravel,
hiển thị một **thanh debug ở cuối trang web** với nhiều tab thông tin về **chính request hiện tại** — công
cụ "must-have" khi phát triển để soi nhanh truy vấn, request, hiệu năng mà không cần `dd()` thủ công (xem
8.1).

**Cài đặt (chỉ môi trường dev):**

```bash
composer require barryvdh/laravel-debugbar --dev
```

Package **tự đăng ký** qua package discovery; bật/tắt theo `APP_DEBUG` hoặc biến `DEBUGBAR_ENABLED` trong
`.env`.

**Các tab (collector) quan trọng:**

| Tab | Cho biết |
|-----|----------|
| **Queries** | **Mọi câu SQL đã chạy** + bindings + **thời gian** + **số lượng** → soi **N+1** (xem 8.5) & query chậm |
| **Timeline** | Thời gian từng giai đoạn (boot, app, từng query) |
| **Models** | Số model được **hydrate** (cảnh báo nạp quá nhiều bản ghi) |
| **Request** | Input, headers, session, cookies của request |
| **Route** | Route khớp, controller, middleware áp dụng |
| **Views** | Các view được render + dữ liệu truyền vào |
| **Exceptions / Messages** | Lỗi và log tự ghi qua Debugbar |

**Ghi log / đo thời gian thủ công:**

```php
Debugbar::info($user);                 // đẩy biến ra tab Messages
Debugbar::warning('Cảnh báo');
Debugbar::measure('Xử lý báo cáo', function () {
    // ... đoạn code nặng → hiện thời gian trên Timeline
});
```

**Lưu ý quan trọng:**
- **CHỈ dùng ở dev** (`--dev`); **tuyệt đối không bật ở production** — lộ thông tin nhạy cảm (query, env,
  session) và làm **chậm + tốn bộ nhớ**.
- Debugbar **chèn HTML** vào response nên hợp với **trang web**; với **API/JSON** nó không hiển thị trực
  tiếp → thường dùng **Telescope** thay thế.
- Khi **benchmark hiệu năng thật**, hãy **tắt** Debugbar vì nó tự thêm overhead.

> So với **Telescope** (xem 8.3): Debugbar là **thanh inline theo từng request** (nhẹ, tức thì, hợp web UI);
> Laravel Telescope là **dashboard lưu lịch sử** request/query/job/mail/exception (hợp **API** và staging).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Laravel Debugbar** (`barryvdh/laravel-debugbar`) = thanh debug cuối trang, hiển thị thông tin request
  hiện tại qua các tab: **Queries (SQL + bindings + time + số lượng), Timeline, Models, Request, Route,
  Views, Exceptions...**.
- Công dụng chính: **đếm/soi query → phát hiện N+1** (xem 8.5) và query chậm, xem dữ liệu
  request/route/view ngay trên trang; ghi tay bằng `Debugbar::info()/measure()`.
- Cài `--dev`, bật theo `APP_DEBUG`/`DEBUGBAR_ENABLED`; tự đăng ký qua package discovery.
- **Chỉ dùng ở dev, không bật production** (lộ thông tin + chậm). Với **API** dùng **Telescope** (dashboard
  lưu lịch sử) thay vì Debugbar (thanh inline theo request).

[↑ Mục lục](#mục-lục)

<a id="q8-3"></a>
### 8.3. Laravel Telescope là gì?

#### 📖 Giới thiệu kiến thức

**Laravel Telescope** (package chính thức `laravel/telescope`) là một **dashboard debug & monitoring** cho
ứng dụng Laravel. Khác Debugbar (xem 8.2) chỉ hiện thanh inline cho **request đang xem**, Telescope **ghi
lại và lưu lịch sử** mọi hoạt động vào DB, rồi hiển thị ở trang riêng `/telescope` — xem lại được **các
request đã xảy ra trước đó**, kể cả với **API/queue/CLI** (những thứ không có giao diện HTML để gắn thanh).

**Cài đặt (chủ yếu cho dev/staging):**

```bash
composer require laravel/telescope --dev
php artisan telescope:install      # publish config + migration
php artisan migrate                # tạo bảng lưu entries
```

Truy cập tại `/telescope`. Việc ai được xem được kiểm soát bằng **`Gate::define('viewTelescope', ...)`**
trong `TelescopeServiceProvider` (quan trọng khi bật ở môi trường không phải local).

**Telescope theo dõi gì (các "watcher"):**

| Watcher | Ghi lại |
|---------|---------|
| **Requests** | Mỗi HTTP request: payload, response, thời gian |
| **Queries** | Mọi SQL + bindings + thời gian, **đánh dấu query chậm** (slow) |
| **Models** | Sự kiện model (created/updated/deleted) |
| **Jobs / Queue** | Job đã dispatch, thành công/thất bại, retry |
| **Exceptions** | Lỗi kèm stack trace |
| **Mail / Notifications** | Email, notification đã gửi (preview được) |
| **Events, Cache, Redis, Schedule, Gate** | Event, cache hit/miss, lệnh redis, cron, kết quả authorization |

**Lưu ý quan trọng (hay hỏi):**
- Telescope **lưu dữ liệu vào DB** → ở **production** sẽ phình rất nhanh và tốn tài nguyên. Nếu buộc bật
  production thì phải **giới hạn** (sampling/`filter`) và **dọn định kỳ**: `php artisan telescope:prune`
  (thường chạy bằng scheduler).
- Bảo vệ truy cập bằng **authorization gate** — tránh lộ dữ liệu nhạy cảm (request, query, mail...).
- Cài `--dev`; nhiều team chỉ bật ở **local/staging**.

> So sánh với **Debugbar** (xem 8.2): **Debugbar** = thanh inline, tức thì, theo **một request web** (nhẹ,
> hợp lúc code UI). **Telescope** = dashboard **lưu lịch sử** nhiều request, hợp **API/queue/job** và điều
> tra sự cố sau khi đã xảy ra. Hai công cụ **bổ sung nhau**, không loại trừ.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Laravel Telescope** (`laravel/telescope`) = **dashboard debug/monitoring** chính thức, **ghi lịch sử**
  hoạt động app vào DB và xem ở `/telescope`.
- Theo dõi qua các **watcher**: **Requests, Queries (đánh dấu query chậm), Models, Jobs/Queue, Exceptions,
  Mail/Notifications, Events, Cache, Redis, Schedule, Gate**.
- Cài `--dev` + `telescope:install` + `migrate`; bảo vệ bằng **gate `viewTelescope`**.
- **Lưu ý:** lưu vào DB nên ở **production** phải **prune** (`telescope:prune`) + sampling, và **khóa quyền
  truy cập**. Cài `--dev`, thường chỉ bật local/staging.
- **Debugbar vs Telescope:** Debugbar = thanh **inline theo 1 request web** (tức thì); Telescope =
  **dashboard lưu lịch sử**, hợp **API/queue** và điều tra sau sự cố. Bổ sung nhau.

[↑ Mục lục](#mục-lục)

<a id="q8-4"></a>
### 8.4. Thứ tự thực thi: lệnh nào chạy trước, lệnh nào chạy sau?

#### 📖 Giới thiệu kiến thức

Phải tách **2 tầng** mới trả lời đúng:

**Tầng 1 — Builder (PHP): các method chỉ "DỰNG" query, chưa chạy DB.**
Các method như `where`, `select`, `orderBy`, `join`, `limit`... chỉ **tích lũy** vào đối tượng builder.
**Thứ tự bạn chain phần lớn KHÔNG quan trọng** — Laravel gom tất cả lại thành **một câu SQL** hoàn chỉnh.

```php
// Hai cách viết dưới đây sinh ra SQL GIỐNG HỆT nhau
User::where('active', 1)->orderBy('name')->get();
User::orderBy('name')->where('active', 1)->get();
```

Query **chỉ thực sự chạy** khi gọi một **"terminal/executing method"** — method trả về kết quả thay vì
builder: `get()`, `first()`, `find()`, `value()`, `pluck()`, `count()`, `exists()`, `sum()`, `paginate()`,
`chunk()`, `cursor()`...

```php
$q = User::where('active', 1);   // CHƯA chạy DB — chỉ là builder
$q->orderBy('name');             // vẫn chưa chạy
$users = $q->get();              // <-- TỚI ĐÂY query mới chạy
```

> Ngoại lệ cần nhớ: thứ tự **có** ý nghĩa khi gom điều kiện **OR** — phải bọc trong closure để đóng ngoặc
> đúng: `->where('a', 1)->where(fn ($q) => $q->where('b', 2)->orWhere('c', 3))`. Các method tuần tự như
> `when()`, `tap()` cũng phụ thuộc thứ tự.

**Tầng 2 — Database: SQL được xử lý theo THỨ TỰ LOGIC, khác thứ tự viết.**
Khi DB chạy một câu `SELECT`, nó xử lý các mệnh đề theo trình tự:

```
FROM / JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT/OFFSET
```

```sql
-- Thứ tự VIẾT
SELECT user_id, COUNT(*) FROM orders
WHERE status = 'paid' GROUP BY user_id HAVING COUNT(*) > 3
ORDER BY user_id LIMIT 10;

-- Thứ tự DB THỰC THI (logic):
-- 1.FROM orders → 2.WHERE status='paid' → 3.GROUP BY user_id
-- 4.HAVING COUNT(*)>3 → 5.SELECT → 6.ORDER BY → 7.LIMIT 10
```

**Hệ quả thực tế (hay hỏi):**
- **Alias** định nghĩa ở `SELECT` **không dùng được trong `WHERE`** (vì `WHERE` chạy **trước** `SELECT`),
  nhưng **dùng được** trong `ORDER BY`/`HAVING` (chạy sau).
- Lọc **trước khi gom nhóm** → dùng **`WHERE`**; lọc **trên kết quả aggregate** (sau `GROUP BY`) → dùng
  **`HAVING`**.
- `LIMIT` chạy **cuối cùng** → luôn áp dụng sau khi đã `ORDER BY`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Tầng Builder:** `where/select/orderBy/join...` chỉ **dựng** query, **chưa** chạy DB; thứ tự chain phần
  lớn **không ảnh hưởng** (ra cùng SQL). Query chỉ chạy khi gọi method **terminal**: `get/first/find/
  count/exists/paginate/chunk/cursor...`.
- **Tầng DB:** `SELECT` xử lý theo **thứ tự logic**:
  **FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT**.
- **Hệ quả:** alias ở `SELECT` **không** dùng được trong `WHERE` (WHERE chạy trước) nhưng được ở
  `ORDER BY`/`HAVING`; lọc trước nhóm dùng `WHERE`, lọc trên aggregate dùng `HAVING`.
- Ngoại lệ builder: gom điều kiện **OR** phải bọc closure để đóng ngoặc đúng.

[↑ Mục lục](#mục-lục)

<a id="q8-5"></a>
### 8.5. N+1 query problem và eager loading

#### 📖 Giới thiệu kiến thức

**N+1 query problem** là lỗi hiệu năng kinh điển của ORM: để lấy dữ liệu liên quan của N bản ghi, code vô
tình chạy **1 query lấy danh sách + N query phụ** (mỗi bản ghi một query) thay vì gộp lại.

```php
// ❌ N+1: 1 query lấy posts + mỗi lần $post->user lại 1 query
$posts = Post::all();              // 1 query
foreach ($posts as $post) {
    echo $post->user->name;        // mỗi vòng lặp +1 query (lazy loading)
}
// 100 posts → 101 queries
```

Nguyên nhân là **lazy loading** (mặc định của relationship): quan hệ chỉ được query **khi lần đầu truy
cập** `$post->user`.

**Eager loading** giải quyết bằng `with()`: nạp trước quan hệ trong **một số ít query** (dùng `WHERE IN`):

```php
// ✅ Eager load: chỉ 2 query bất kể bao nhiêu post
$posts = Post::with('user')->get();
// Query 1: select * from posts
// Query 2: select * from users where id in (1, 2, 3, ...)
foreach ($posts as $post) {
    echo $post->user->name;        // không phát sinh thêm query
}
```

Mở rộng hay dùng:

- **Lồng / nhiều quan hệ:** `with(['user', 'comments.author'])`.
- **Chọn cột / có điều kiện:** `with(['user:id,name'])`, `with(['comments' => fn ($q) => $q->latest()])`.
- **Lazy eager load:** `$posts->load('user')` khi đã có sẵn collection.
- **`withCount('comments')`** để đếm quan hệ mà không cần nạp toàn bộ.
- **Phát hiện sớm:** `Model::preventLazyLoading()` (trong `boot()` của provider, ở môi trường dev) sẽ ném
  exception khi lỡ lazy load.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **N+1** = lấy quan hệ của N bản ghi bằng **1 + N** query do **lazy loading** (quan hệ chỉ query khi truy
  cập lần đầu) → chậm khi N lớn.
- **Khắc phục: eager loading `with()`** → gộp còn **~2 query** (dùng `WHERE IN`). Hỗ trợ lồng
  (`comments.author`), chọn cột (`user:id,name`), thêm điều kiện bằng closure.
- Bổ sung: **`load()`** (lazy eager khi đã có collection), **`withCount()`** để đếm, và
  **`preventLazyLoading()`** ở môi trường dev để phát hiện N+1 sớm.

[↑ Mục lục](#mục-lục)

<a id="q8-6"></a>
### 8.6. So sánh `with()` và `load()`

#### 📖 Giới thiệu kiến thức

Cả `with()` và `load()` đều là **eager loading** và đều **chống N+1** (xem 8.5); khác biệt nằm ở **thời
điểm** nạp quan hệ: nạp **trước** khi lấy model, hay nạp **sau** khi đã có model trong tay.

**`with()` — eager loading "từ đầu".** Khai báo quan hệ cần nạp **ngay khi dựng query**, trước khi
truy vấn chạy. Dùng khi bạn **kiểm soát câu query gốc**.

```php
// Quyết định nạp 'comments' NGAY khi build query
$posts = Post::with('comments')->where('active', 1)->get();
// → 2 query chạy cùng lúc lấy posts
```

**`load()` — lazy eager loading "sau khi đã có".** Gọi trên một **model/collection đã tồn tại** để nạp
thêm quan hệ. Dùng khi bạn **đã có sẵn model** (do nơi khác trả về, hoặc nạp có điều kiện).

```php
$posts = Post::all();              // 1 query — chưa có comments
// ... một lúc sau, khi xác định là cần:
if ($wantComments) {
    $posts->load('comments');      // +1 query, nạp cho collection đã có
}
```

**Cùng số query, khác ngữ cảnh:** cả hai đều sinh ra "1 query gốc + 1 query mỗi quan hệ" (dùng
`WHERE IN`). Chọn theo việc bạn **đã có collection hay chưa**:

| | `with()` | `load()` |
|---|----------|----------|
| Thời điểm | **Trước** khi query chạy (lúc build) | **Sau** khi đã có model/collection |
| Áp dụng lên | Query/Builder (`Model::query()`) | Instance/Collection đã tồn tại |
| Khi nào dùng | Biết trước cần quan hệ, kiểm soát query gốc | Đã có model sẵn, hoặc nạp **có điều kiện** |
| Tránh nạp thừa | Luôn nạp | Chỉ nạp khi thực sự cần |

**Biến thể quan trọng — `loadMissing()`:** chỉ nạp quan hệ **nếu chưa được nạp**, tránh query lại thừa
khi không chắc collection đã có quan hệ hay chưa:

```php
$posts->loadMissing('comments');   // đã có 'comments' thì bỏ qua, chưa có mới nạp
```

Họ `load*` còn có `loadCount()`, `loadMorph()`; và `load()` cũng nhận closure điều kiện / chọn cột y như
`with()`: `$posts->load(['comments' => fn ($q) => $q->latest()])`.

> Bẫy hay gặp: dùng `load()` **trong vòng lặp** trên từng model (`$post->load(...)` mỗi vòng) lại tái tạo
> đúng N+1. Khi đã có collection, hãy gọi `load()` **một lần trên cả collection**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai đều **eager loading chống N+1**; khác ở **thời điểm**: `with()` nạp **trước** (khai báo lúc dựng
  query), `load()` nạp **sau** (trên model/collection **đã có sẵn**).
- Dùng **`with()`** khi kiểm soát query gốc và biết trước cần quan hệ; dùng **`load()`** khi đã có model
  trong tay hoặc cần **nạp có điều kiện** (tránh nạp thừa).
- **`loadMissing()`** chỉ nạp nếu chưa nạp (tránh query lặp). Cả hai sinh **cùng số query** (`WHERE IN`).
- Bẫy: gọi `load()` **trong vòng lặp từng model** → lại thành N+1; hãy `load()` **một lần trên cả
  collection**.

[↑ Mục lục](#mục-lục)

<a id="q8-7"></a>
### 8.7. Cơ chế ánh xạ record của Eloquent (Active Record & hydration)

#### 📖 Giới thiệu kiến thức

Eloquent theo **Active Record pattern**: **model class ↔ bảng**, **instance ↔ row (record)**, **cột ↔
thuộc tính** của object. Câu hỏi "Eloquent ánh xạ record thế nào?" là hỏi *làm sao một dòng DB trở thành
một model object và ngược lại*.

**1. Ánh xạ ngầm theo convention (không khai báo từng cột).**
Eloquent **không** bắt khai báo property cho mỗi cột. Tên bảng suy ra từ tên model (số nhiều, snake_case:
`User` → `users`), khóa chính mặc định `id`, có `created_at`/`updated_at`. Đổi được qua `$table`,
`$primaryKey`, `$timestamps`.

**2. Mảng `$attributes` — trái tim của ánh xạ.**
Mỗi model giữ một mảng nội bộ **`$attributes` = `['cột' => giá_trị']`** chứa dữ liệu thật của row. Vì model
**không có** property `name` thật, khi bạn gọi `$user->name` → rơi vào **magic method `__get`** →
`getAttribute('name')` → trả `$attributes['name']`. Gán `$user->name = 'A'` → **`__set`** →
`setAttribute()` → ghi vào `$attributes`. **Đây là lý do model "tự biết" mọi cột mà không cần khai báo.**

**3. Hydration — biến row (mảng) thành object.**
Khi query chạy, PDO trả về **mảng/`stdClass` thô**; Eloquent **hydrate**: tạo instance rồi đổ raw values vào
`$attributes` qua **`newFromBuilder()`**, đánh dấu **`exists = true`** và lưu một bản sao **`$original`**.
Hydration **bỏ qua `__construct`** và coi dữ liệu là "raw từ DB".

**4. Casts & accessor/mutator — biến đổi khi đọc/ghi.**
`$attributes` lưu **giá trị thô**; giá trị "đẹp" được tính khi truy cập:
- **`$casts`**: đổi kiểu khi đọc (`'is_active' => 'boolean'`, `'options' => 'array'` cho cột JSON,
  `'published_at' => 'datetime'` → Carbon).
- **Accessor/Mutator** (`Attribute::make(get:..., set:...)`): biến đổi tùy biến khi đọc/ghi.

**5. Dirty tracking — nhờ so `$attributes` với `$original`.**
Vì lúc hydrate đã lưu `$original`, khi `save()` Eloquent so sánh để biết cột nào "dirty" → chỉ **`UPDATE`
những cột thay đổi**. Hỗ trợ `isDirty()`, `getChanges()`, `wasChanged()`.

**6. Mass assignment.** `create()`/`fill([...])` đổ mảng vào attributes nhưng lọc qua **`$fillable`/
`$guarded`** để chống mass-assignment.

```php
$user = User::find(1);
// Bên trong: $attributes = ['id'=>1,'name'=>'An','is_active'=>1,'options'=>'{"k":1}']
//            $original   = bản sao (để so dirty)

$user->name;          // __get → getAttribute('name') → $attributes['name'] = 'An'
$user->is_active;     // cast 'boolean' → true (dù DB lưu 1)
$user->options;       // cast 'array'   → ['k' => 1] (decode JSON)

$user->name = 'Bình'; // __set → setAttribute → $attributes['name'] = 'Bình'
$user->isDirty('name');   // true (khác $original)
$user->save();            // chỉ UPDATE cột dirty (name)
```

```php
protected $casts = [
    'is_active'    => 'boolean',
    'options'      => 'array',     // cột JSON ↔ array PHP
    'published_at' => 'datetime',  // ↔ Carbon
];
```

> Liên hệ: vì dựa vào **magic method `__get/__set` + Reflection**, Eloquent linh hoạt nhưng **"nặng" hơn**
> Query Builder (xem câu 8). Khác **Data Mapper** (vd Doctrine — object tách rời khỏi DB), **Active Record**
> gắn trực tiếp object với row, nên model vừa giữ dữ liệu vừa biết cách tự lưu.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Eloquent theo **Active Record**: model↔bảng, instance↔row, cột↔thuộc tính; ánh xạ **theo convention**
  (tên bảng số nhiều, khóa `id`) — **không khai báo từng cột**.
- **Cốt lõi: mảng `$attributes`** giữ dữ liệu row; truy cập `$model->col` đi qua **magic `__get/__set` →
  `getAttribute/setAttribute`** đọc/ghi `$attributes` (lý do model "tự biết" cột).
- **Hydration:** kết quả query (mảng thô) được đổ vào model qua **`newFromBuilder()`** (set `exists=true`,
  lưu **`$original`**), bỏ qua constructor.
- **Cast + accessor/mutator** biến đổi giá trị khi đọc/ghi (boolean, array/json, datetime); `$attributes`
  vẫn lưu raw.
- **Dirty tracking** nhờ so `$attributes` vs `$original` → `save()` chỉ UPDATE cột đổi; mass assignment lọc
  qua `$fillable`/`$guarded`.

[↑ Mục lục](#mục-lục)

<a id="q8-8"></a>
### 8.8. `$fillable` và `$guarded` (mass assignment) trong model

#### 📖 Giới thiệu kiến thức

**Mass assignment** = gán **nhiều thuộc tính cùng lúc bằng một mảng** qua `create()`, `fill()`, `update()`.
Rất tiện, nhưng **nguy hiểm** nếu đổ thẳng input người dùng vào model: kẻ tấn công có thể nhồi thêm field
ngoài ý muốn (vd `is_admin = 1`, `role = 'admin'`) → **mass assignment vulnerability**.

Để chặn, Eloquent bắt mỗi model **khai báo rõ cột nào được phép gán hàng loạt**, bằng **một trong hai**
property đối lập nhau:

**`$fillable` — allowlist (danh sách CHO PHÉP):** chỉ cột liệt kê mới được mass-assign.

```php
protected $fillable = ['name', 'email', 'password'];
```

**`$guarded` — blocklist (danh sách CẤM):** mọi cột đều được gán **trừ** các cột liệt kê.

```php
protected $guarded = ['id', 'is_admin'];
// $guarded = []  → cho phép TẤT CẢ cột (chỉ dùng khi đã validate input kỹ ở tầng khác)
```

**Lưu ý quan trọng (hay hỏi):**
- Dùng **một trong hai**, không dùng cả hai cùng lúc (`$fillable` được ưu tiên).
- Chỉ ảnh hưởng **mass assignment** (`create/fill/update` với mảng). **Gán trực tiếp KHÔNG bị chặn:**

```php
$user->is_admin = true;   // ✅ vẫn được — gán thuộc tính trực tiếp KHÔNG qua mass assignment
$user->save();
```

- Khi `fill($array)`, cột không nằm trong allowlist bị **bỏ qua âm thầm** (mặc định). Bật
  `Model::preventSilentlyDiscardingAttributes()` (trong `boot()`, môi trường dev) để **ném exception** giúp
  phát hiện sớm.

```php
// ❌ Rủi ro nếu fillable/guarded khai báo lỏng
User::create($request->all());           // attacker gửi kèm is_admin=1

// ✅ An toàn: chỉ field whitelisted được gán; nên lọc input thêm
protected $fillable = ['name', 'email', 'password'];
User::create($request->only(['name', 'email', 'password']));
```

**Chọn `$fillable` hay `$guarded`?**
- **`$fillable` (allowlist) — khuyến nghị**: an toàn hơn vì rõ ràng; thêm cột mới mặc định **bị chặn** cho
  tới khi bạn chủ động khai báo.
- **`$guarded` (blocklist)**: tiện khi bảng nhiều cột và muốn mở gần hết, nhưng rủi ro: thêm cột mới mặc
  định **được phép** → dễ lọt cột nhạy cảm.

> Liên hệ: đây là bộ lọc ở bước `fill()` của cơ chế gán thuộc tính (xem 8.7). Nên kết hợp với **validation**
> / `$request->only()` thay vì chỉ dựa vào fillable.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Mass assignment** = gán nhiều cột bằng mảng (`create/fill/update`); rủi ro đổ thẳng `$request->all()`
  cho phép attacker set field nhạy cảm (`is_admin`) → **mass assignment vulnerability**.
- **`$fillable`** = **allowlist** (chỉ cột liệt kê được gán) — **nên dùng**, an toàn hơn. **`$guarded`** =
  **blocklist** (cấm cột liệt kê, còn lại cho phép); `$guarded = []` = mở tất cả.
- Dùng **một trong hai**; chỉ áp cho **mass assignment**, **không** chặn gán trực tiếp `$model->col = ...`.
- Cột không hợp lệ bị **bỏ qua âm thầm** → bật **`preventSilentlyDiscardingAttributes()`** ở dev để báo
  lỗi; nên kết hợp **validate / `$request->only()`**.

[↑ Mục lục](#mục-lục)

<a id="q9"></a>
## 9. Eloquent Relationship trong Laravel là gì?

### 📖 Giới thiệu kiến thức

**Relationship** (quan hệ) là cách Eloquent **mô hình hoá liên kết giữa các bảng** (theo khoá ngoại) bằng
cách khai báo **method trên model**. Nhờ đó bạn truy cập dữ liệu liên quan **như một thuộc tính object**,
không phải tự viết `JOIN` — vd `$post->comments`, `$comment->post->title`.

**Các loại quan hệ chính:**

| Loại | Method | Ví dụ |
|------|--------|-------|
| One-to-One | `hasOne` / `belongsTo` | `User` có 1 `Phone` |
| One-to-Many | `hasMany` / `belongsTo` | `Post` có nhiều `Comment` |
| Many-to-Many | `belongsToMany` (qua **pivot**) | `User` ↔ `Role` |
| Has-Many-Through | `hasManyThrough` | `Country` → `User` → `Post` |
| Polymorphic | `morphOne`/`morphMany`/`morphTo`, `morphToMany` | `Comment` cho cả `Post` lẫn `Video` |

```php
class Post extends Model
{
    // 1 Post có nhiều Comment
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}

class Comment extends Model
{
    // Chiều ngược: Comment thuộc về 1 Post
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

**Hai cách truy cập — điểm rất hay hỏi:**

```php
// 1) Như THUỘC TÍNH (dynamic property) → chạy query lần đầu rồi cache, trả về kết quả (Collection/model)
$comments = $post->comments;            // lazy load, trả Collection<Comment>

// 2) Như METHOD → trả về quan hệ/query builder để THÊM điều kiện rồi mới lấy
$recent = $post->comments()            // HasMany (query builder)
    ->where('approved', true)
    ->latest()
    ->get();
```

- Truy cập **thuộc tính** (`$post->comments`): tiện, nhưng nếu lặp trên nhiều model dễ dính **N+1** → dùng
  eager loading `with()` (xem 8.5, 8.6).
- Gọi **method** (`$post->comments()`): khi cần thêm `where/orderBy/limit` trước khi truy vấn.

**Tạo & tận dụng quan hệ:**
- Insert qua quan hệ: `$post->comments()->create([...])` (tự gán khoá ngoại).
- Kiểm tra tồn tại / lọc theo quan hệ: `Post::has('comments')`, `Post::whereHas('comments', fn ($q) => ...)`.
- Đếm: `Post::withCount('comments')`.

> Liên hệ: hiệu năng khi nạp quan hệ (N+1, `with`/`load`) xem **8.5** và **8.6**; quan hệ là nền tảng của
> Eloquent ORM ở **câu 8**.

### 🎯 Tóm tắt trả lời phỏng vấn

- Relationship = khai báo **method trên model** để mô hình hoá liên kết bảng theo khoá ngoại, truy cập dữ
  liệu liên quan **như thuộc tính** (`$post->comments`) thay vì tự `JOIN`.
- **Các loại:** one-to-one (`hasOne`/`belongsTo`), one-to-many (`hasMany`/`belongsTo`), many-to-many
  (`belongsToMany` qua pivot), `hasManyThrough`, và **polymorphic** (`morphMany`/`morphTo`).
- **2 cách dùng:** truy cập **thuộc tính** (`$post->comments` — lazy load, trả Collection) vs gọi **method**
  (`$post->comments()` — trả query builder để thêm `where/orderBy`).
- Tiện ích: `create()` qua quan hệ, `has`/`whereHas` để lọc, `withCount` để đếm; nhớ **eager load `with()`**
  tránh **N+1** (xem 8.5/8.6).

[↑ Mục lục](#mục-lục)

<a id="q9-1"></a>
### 9.1. `hasMany`/`hasOne` vs `belongsTo` — khóa ngoại đặt ở đâu?

#### 📖 Giới thiệu kiến thức

Đây là điểm gây nhầm nhiều nhất. Mấu chốt: **khoá ngoại (foreign key) nằm ở bảng "con" (bảng sở hữu khoá,
phía "many"/"thuộc về")**, và model của bảng đó dùng **`belongsTo`**; model phía "cha" dùng
**`hasMany`/`hasOne`**.

```text
Bảng comments có cột post_id  →  comments là bảng "con" (giữ FK)
   Comment::post()  → belongsTo(Post)     (bên GIỮ khoá ngoại)
   Post::comments() → hasMany(Comment)    (bên KIA của khoá ngoại)
```

**Quy ước đặt tên khoá (Laravel tự suy ra):**
- `belongsTo(Post::class)` → mặc định tìm cột **`post_id`** ở bảng hiện tại, trỏ tới `posts.id`.
- `hasMany(Comment::class)` → mặc định tìm cột **`post_id`** ở bảng `comments`, dựa trên `posts.id`.

Nếu đặt tên khác quy ước, **truyền tham số để override**:

```php
// belongsTo(related, foreignKey, ownerKey)
return $this->belongsTo(Post::class, 'article_id', 'id');

// hasMany(related, foreignKey, localKey)
return $this->hasMany(Comment::class, 'article_id', 'id');
```

**`hasOne` vs `hasMany`:** cùng "cha giữ quan hệ", chỉ khác **số lượng** — `hasOne` trả **một** model,
`hasMany` trả **Collection**. Chiều ngược của cả hai đều là `belongsTo`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Khoá ngoại nằm ở bảng "con"** (phía "thuộc về"/"many"); model đó dùng **`belongsTo`**, model "cha" dùng
  **`hasMany`/`hasOne`**.
- Quy ước: `belongsTo(Post)` tìm cột **`post_id`**; đặt tên khác thì override
  `belongsTo(related, foreignKey, ownerKey)` / `hasMany(related, foreignKey, localKey)`.
- **`hasOne` vs `hasMany`** chỉ khác số lượng (một model vs Collection); chiều ngược của cả hai là
  **`belongsTo`**.

[↑ Mục lục](#mục-lục)

<a id="q9-2"></a>
### 9.2. Many-to-many (pivot) & Polymorphic relationship

#### 📖 Giới thiệu kiến thức

**Many-to-many — `belongsToMany` qua bảng pivot.** Khi hai bảng quan hệ "nhiều–nhiều" (vd `User` ↔ `Role`),
cần một **bảng trung gian (pivot)** chứa hai khoá ngoại — mặc định tên là ghép **số ít, theo thứ tự bảng
chữ cái**: `role_user` (cột `user_id`, `role_id`).

```php
class User extends Model
{
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);   // pivot mặc định: role_user
    }
}

// Thao tác trên pivot
$user->roles()->attach($roleId);          // thêm liên kết
$user->roles()->detach($roleId);          // gỡ liên kết
$user->roles()->sync([1, 2, 3]);          // đồng bộ đúng tập này (xoá cái thừa)
$names = $user->roles->pluck('name');     // đọc các role

// Lấy thêm cột trên pivot
return $this->belongsToMany(Role::class)->withPivot('assigned_at')->withTimestamps();
// đọc: $user->roles->first()->pivot->assigned_at
```

**Polymorphic — một quan hệ trỏ tới NHIỀU loại model.** Vd `Comment` có thể thuộc về `Post` **hoặc**
`Video`. Thay vì tạo `post_id` và `video_id` riêng, dùng cặp cột **`*_id` + `*_type`** (lưu cả id lẫn tên
class) trên bảng `comments`: `commentable_id`, `commentable_type`.

```php
class Comment extends Model
{
    public function commentable(): MorphTo
    {
        return $this->morphTo();              // đọc commentable_id + commentable_type
    }
}

class Post extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
// Video cũng morphMany Comment 'commentable' → dùng chung một bảng comments
$post->comments;            // các comment của post
$comment->commentable;      // trả về Post HOẶC Video tuỳ commentable_type
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Many-to-many:** dùng **`belongsToMany`** qua **bảng pivot** (mặc định ghép số ít theo a–z, vd
  `role_user`). Quản lý liên kết bằng **`attach`/`detach`/`sync`**; thêm cột pivot bằng
  **`withPivot()`/`withTimestamps()`**, đọc qua `->pivot`.
- **Polymorphic:** một quan hệ trỏ tới **nhiều loại model** nhờ cặp cột **`*_id` + `*_type`**
  (vd `commentable_id`/`commentable_type`). Dùng **`morphTo`** (bên con) và **`morphMany`/`morphOne`** (bên
  cha); biến thể nhiều–nhiều là **`morphToMany`**.
- Lợi ích: tái sử dụng **một bảng** (vd `comments`) cho nhiều entity thay vì nhiều cột khoá ngoại riêng.

[↑ Mục lục](#mục-lục)

<a id="q10"></a>
## 10. Eloquent Scope (Query Scope) là gì?

### 📖 Giới thiệu kiến thức

**Scope** giúp **đóng gói các ràng buộc truy vấn hay lặp lại** (`where`, `orderBy`...) thành một "tên" tái
sử dụng, giữ code controller/service **gọn và DRY**, đặt tên **theo ngữ nghĩa nghiệp vụ**
(`Post::published()` thay vì rải `where('status', 'published')` khắp nơi). Có **2 loại**: **Local Scope** và
**Global Scope**.

**Local Scope — định nghĩa method `scopeXxx` trên model, gọi bằng `xxx()`** (bỏ tiền tố `scope`):

```php
use Illuminate\Database\Eloquent\Builder;

class Post extends Model
{
    // scopePublished → gọi Post::published()
    public function scopePublished(Builder $query): void
    {
        $query->where('status', 'published');
    }

    // Dynamic scope: nhận tham số sau $query
    public function scopeOfType(Builder $query, string $type): void
    {
        $query->where('type', $type);
    }
}

// Dùng — gọi như method, chain bình thường với query builder
$posts = Post::published()->ofType('news')->latest()->get();
```

- Tên `scopePublished` → gọi `Post::published()`; **dynamic scope** truyền đối số khi gọi:
  `ofType('news')`.
- Scope chỉ là cách **gói `where` lại** → bên trong vẫn là query builder, **chain được** với mọi method khác.

**Global Scope — tự động áp cho MỌI truy vấn** của model mà không cần gọi thủ công (xem 10.1). Ví dụ kinh
điển: **SoftDeletes** dùng global scope để tự thêm `where deleted_at is null`.

> Liên hệ: scope là lớp tiện ích trên **Query Builder/Eloquent** (câu 8); giúp tuân thủ **DRY** — gom
> logic truy vấn vào model thay vì lặp ở nhiều nơi.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Scope** = đóng gói ràng buộc truy vấn **tái sử dụng** trên model (DRY, tên có ngữ nghĩa nghiệp vụ).
- **Local scope:** method **`scopeXxx(Builder $q)`** trên model, gọi bằng **`Model::xxx()`** (bỏ prefix
  `scope`); **dynamic scope** nhận thêm tham số (`scopeOfType($q, $type)` → `ofType('news')`). Chain được
  như query builder.
- **Global scope:** **tự áp cho mọi truy vấn** của model (xem 10.1) — ví dụ **SoftDeletes**.

[↑ Mục lục](#mục-lục)

<a id="q10-1"></a>
### 10.1. Global Scope & Soft Deletes

#### 📖 Giới thiệu kiến thức

**Global Scope** áp ràng buộc cho **TẤT CẢ** truy vấn của một model — tự động, không cần gọi tay. Đăng ký
trong method `booted()` của model:

```php
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class ActiveScope implements Scope
{
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('active', true);
    }
}

class User extends Model
{
    protected static function booted(): void
    {
        static::addGlobalScope(new ActiveScope);
        // hoặc dạng closure có tên:
        // static::addGlobalScope('active', fn (Builder $b) => $b->where('active', true));
    }
}
// Từ đây: User::all() tự kèm "where active = true"
```

**Bỏ qua global scope khi cần:**

```php
User::withoutGlobalScope(ActiveScope::class)->get();   // bỏ 1 scope
User::withoutGlobalScopes()->get();                    // bỏ tất cả
```

**SoftDeletes — global scope điển hình:** trait `SoftDeletes` thêm `SoftDeletingScope` tự chèn
`where deleted_at is null` vào mọi query → bản ghi "xóa mềm" bị ẩn mặc định (cần cột `deleted_at`).

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;            // cần cột deleted_at (migration: $table->softDeletes())
}

$post->delete();               // soft delete: chỉ set deleted_at, KHÔNG xóa thật
Post::withTrashed()->get();    // gồm cả bản ghi đã xóa (bỏ qua scope)
Post::onlyTrashed()->get();    // chỉ bản ghi đã xóa
$post->restore();              // khôi phục (xóa deleted_at)
$post->forceDelete();          // xóa cứng khỏi DB
```

> **Cảnh báo (hay hỏi):** global scope áp **ngầm** nên dễ "quên" nó tồn tại → muốn lấy đủ dữ liệu phải tắt
> tường minh bằng `withoutGlobalScope(s)`. Cũng cẩn thận với update/delete hàng loạt vì scope vẫn được áp.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Global scope** = ràng buộc **tự áp cho mọi truy vấn** của model; đăng ký trong **`booted()`** qua
  **`addGlobalScope(class implements Scope | 'tên', closure)`**.
- Bỏ qua bằng **`withoutGlobalScope(Class)`** / **`withoutGlobalScopes()`**.
- **SoftDeletes** là global scope điển hình: trait `SoftDeletes` + cột `deleted_at` → tự thêm
  `where deleted_at is null`; thao tác kèm **`withTrashed()` / `onlyTrashed()` / `restore()` /
  `forceDelete()`**.
- Lưu ý: scope áp **ngầm** dễ quên → muốn lấy đầy đủ phải **tắt scope tường minh**.

[↑ Mục lục](#mục-lục)

<a id="q11"></a>
## 11. Collection trong Laravel là gì?

### 📖 Giới thiệu kiến thức

**Collection** (`Illuminate\Support\Collection`) là một **lớp bọc (wrapper) quanh mảng PHP**, cung cấp
một API **fluent, chainable** để duyệt/biến đổi dữ liệu (map, filter, reduce, group...) thay cho các hàm
`array_*` rời rạc và khó đọc. Đây là một trong những tiện ích được dùng **nhiều nhất** trong Laravel.

```php
$result = collect([1, 2, 3, 4, 5])
    ->filter(fn ($n) => $n % 2 === 0)   // [2, 4]
    ->map(fn ($n) => $n * 10)           // [20, 40]
    ->values()
    ->all();                            // [20, 40]
```

- Tạo bằng helper **`collect([...])`** hoặc nhận sẵn từ Laravel.
- Phần lớn method **không sửa collection gốc** mà **trả về một collection mới** (immutable-style) → tiện
  nối chuỗi (method chaining).

**Liên hệ với Eloquent (mấu chốt):** mọi truy vấn Eloquent trả về nhiều bản ghi (`get()`, `all()`,
relationship...) đều cho ra một **`Eloquent\Collection`** — lớp **kế thừa** base Collection và thêm các
method chuyên cho model (`find`, `load`, `modelKeys`, `pluck` theo key, `contains` theo khoá...). Vì vậy
toàn bộ API Collection dùng được trên kết quả Eloquent.

```php
$emails = User::all()                   // Eloquent\Collection các model User
    ->where('active', true)             // ⚠ lọc TRONG PHP (đã load vào RAM)
    ->pluck('email');                   // lấy mảng email
```

> **Bẫy hay hỏi — `Collection::where()` ≠ `QueryBuilder::where()`:** `where()` trên Collection lọc **trong
> PHP sau khi đã tải toàn bộ dữ liệu vào bộ nhớ**, còn `where()` trên Query Builder/Eloquent **lọc ở tầng
> DB** (sinh `WHERE` trong SQL). Với tập lớn, **lọc ở DB** hiệu quả hơn nhiều — đừng `User::all()->where(...)`.

#### Các method hay gặp

- **Biến đổi:** `map`, `filter`, `reject`, `reduce`, `flatMap`, `mapWithKeys`.
- **Lấy/gom:** `pluck`, `keyBy`, `groupBy`, `partition`, `chunk`, `collapse`.
- **Sắp xếp:** `sortBy`, `sortByDesc`, `sortKeys`, `values`.
- **Tổng hợp:** `sum`, `avg`, `min`, `max`, `count`, `countBy`.
- **Tìm/kiểm tra:** `first`, `firstWhere`, `contains`, `every`, `some`, `unique`, `where`.
- **Duyệt:** `each`, `tap`, `dd`/`dump`.

**Higher-order messages** — cú pháp rút gọn cho method nhận closure đơn giản:

```php
$users->each->markAsActive();          // tương đương ->each(fn ($u) => $u->markAsActive())
$names = $users->map->name;            // lấy thuộc tính name của từng phần tử
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Collection** = lớp bọc quanh mảng (`Illuminate\Support\Collection`), cung cấp API **fluent, chainable**
  (`map`, `filter`, `reduce`, `pluck`, `groupBy`...) thay cho hàm `array_*` rời rạc.
- Phần lớn method **trả về collection mới** (immutable-style) → dễ nối chuỗi. Tạo bằng **`collect()`**.
- **Eloquent trả về `Eloquent\Collection`** (kế thừa base + thêm method cho model) nên dùng được toàn bộ
  API Collection trên kết quả query.
- **Bẫy:** `Collection::where()` lọc **trong PHP** (sau khi load), còn `QueryBuilder::where()` lọc **ở DB**
  — với tập lớn phải lọc ở DB, đừng `all()->where()`.
- Có **higher-order messages** (`$users->each->save()`, `$users->map->name`) cho cú pháp gọn.

[↑ Mục lục](#mục-lục)

<a id="q11-1"></a>
### 11.1. Collection vs array thường & LazyCollection (xử lý dữ liệu lớn)

#### 📖 Giới thiệu kiến thức

**Collection vs array thường:**

| | Array + hàm `array_*` | Collection |
|---|----------------------|------------|
| Cú pháp | Hàm rời rạc, lồng nhau (`array_map(array_filter(...))`) đọc ngược | **Chainable**, đọc xuôi theo luồng xử lý |
| Khả đọc | Khó khi nhiều bước | Cao, biểu đạt ý đồ rõ |
| Tính năng | Cơ bản | 100+ method tiện (`groupBy`, `pluck`, `chunk`...) |
| Chi phí | Nhẹ nhất | Có **overhead nhỏ** (bọc object) |

Với vài phần tử thì khác biệt không đáng kể; điểm cộng chính của Collection là **khả đọc & biểu đạt**.

**LazyCollection — cho dữ liệu CỰC LỚN:** Collection thường **giữ toàn bộ dữ liệu trong RAM**, nên duyệt
hàng triệu bản ghi sẽ **tràn bộ nhớ**. `Illuminate\Support\LazyCollection` dùng **PHP generator** để xử lý
**từng phần tử một**, không nạp tất cả vào bộ nhớ cùng lúc.

```php
// ❌ all() nạp TOÀN BỘ user vào RAM → dễ hết bộ nhớ
User::all()->each(fn ($u) => $u->process());

// ✅ cursor() trả về LazyCollection — chỉ giữ 1 model trong RAM tại một thời điểm
User::cursor()->each(fn ($u) => $u->process());
```

**`chunk()` / `chunkById()`** là cách khác để xử lý theo **lô** (mỗi lần lấy N bản ghi), hợp khi cần
update hàng loạt mà vẫn giới hạn bộ nhớ:

```php
User::where('active', false)->chunkById(1000, function ($users) {
    foreach ($users as $user) {
        // xử lý theo lô 1000, không nạp hết bảng
    }
});
```

> Phân biệt nhanh: **`cursor()`** (LazyCollection) chạy **một query**, stream từng dòng từ DB; **`chunk()`**
> chạy **nhiều query** mỗi query lấy một lô. Cả hai đều giảm bộ nhớ so với `all()`/`get()`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Collection vs array:** Collection cho cú pháp **chainable, dễ đọc** và 100+ method tiện, đổi lại có
  **overhead nhỏ**; array thuần nhẹ hơn nhưng code rời rạc/khó đọc khi nhiều bước.
- **LazyCollection** (`Illuminate\Support\LazyCollection`) dùng **generator**, xử lý **từng phần tử**,
  **không nạp hết vào RAM** → dành cho **dataset cực lớn**. Eloquent **`->cursor()`** trả về LazyCollection.
- **`chunk()`/`chunkById()`**: xử lý theo **lô** (nhiều query, mỗi query một lô) cho thao tác hàng loạt.
- Quy tắc: tập nhỏ → Collection thường; tập rất lớn → **`cursor()`** (1 query, stream) hoặc **`chunk()`**
  (nhiều query theo lô) để tránh tràn bộ nhớ.

[↑ Mục lục](#mục-lục)

<a id="q12"></a>
## 12. Observer trong Laravel (Eloquent Observer) là gì?

### 📖 Giới thiệu kiến thức

Mỗi model Eloquent **tự phát ra (fire) một loạt sự kiện (event)** tại các mốc trong vòng đời của nó:
khi được tạo, cập nhật, xóa, khôi phục... Ví dụ khi gọi `$user->save()`, Eloquent lần lượt bắn
`saving` → (`creating` hoặc `updating`) → ghi DB → (`created` hoặc `updated`) → `saved`.

**Observer** là một **class chuyên gom toàn bộ listener cho các event đó về một chỗ**. Thay vì rải rác
nhiều closure (`User::creating(fn ($u) => ...)`) trong `boot()` của model, bạn tạo một class có các method
trùng tên với event và viết logic vào đó — code **gọn, tập trung, dễ test, đúng Single Responsibility**.

Đây chính là hiện thực của **Observer pattern**: model là "subject" phát thông báo, observer là "người
quan sát" phản ứng — model **không cần biết** ai đang lắng nghe (loose coupling).

**Tạo observer:**

```bash
php artisan make:observer UserObserver --model=User
```

```php
class UserObserver
{
    // chạy TRƯỚC khi insert — sửa dữ liệu hoặc hủy thao tác (return false)
    public function creating(User $user): void
    {
        $user->uuid = (string) Str::uuid();
        $user->slug = Str::slug($user->name);
    }

    // chạy SAU khi đã ghi DB — gửi mail, clear cache, ghi audit log
    public function created(User $user): void
    {
        Mail::to($user)->queue(new WelcomeMail($user));
    }

    public function updated(User $user): void
    {
        Cache::forget("user:{$user->id}");
    }

    public function deleting(User $user): bool
    {
        // trả về false để CHẶN việc xóa
        return $user->orders()->doesntExist();
    }
}
```

**Đăng ký observer** — hai cách:

```php
// Cách 1: trong boot() của một Service Provider (vd AppServiceProvider)
public function boot(): void
{
    User::observe(UserObserver::class);
}

// Cách 2 (Laravel 10.32+): attribute ngay trên model — gọn, không cần provider
#[\Illuminate\Database\Eloquent\Attributes\ObservedBy([UserObserver::class])]
class User extends Model { /* ... */ }
```

**Vì sao quan trọng / khi nào dùng:** mọi logic "tự động đi kèm vòng đời model" nên đặt ở observer —
auto-fill `uuid`/`slug`, hash password, set `created_by`, **audit log**, **clear/invalidate cache**, gửi
notification, đồng bộ dữ liệu sang hệ thống khác. Nhờ đó controller/service chỉ lo nghiệp vụ chính.

> Liên hệ: Observer là biểu hiện của **Observer pattern** — xem `Common/solid-design-pattern.md`. Về
> bản chất nó là một lớp tiện lợi bọc cơ chế **Event/Listener** của Laravel (xem 12.2).

### 🎯 Tóm tắt trả lời phỏng vấn

- Eloquent model tự bắn các **event vòng đời** (`creating/created`, `updating/updated`, `saving/saved`, `deleting/deleted`, `restoring/restored`...). **Observer** là class **gom mọi listener** đó về một nơi.
- Là hiện thực của **Observer pattern**: model (subject) phát thông báo, observer phản ứng → tách logic phụ ra khỏi nghiệp vụ chính (audit log, cache, slug, notification...).
- Đăng ký qua `Model::observe(Observer::class)` trong `boot()` hoặc attribute `#[ObservedBy(...)]` (Laravel 10.32+).
- Event "**ing**" (creating, deleting...) chạy **trước** khi ghi DB và có thể **return false để hủy**; event quá khứ (created, deleted...) chạy **sau**.

[↑ Mục lục](#mục-lục)

<a id="q12-1"></a>
### 12.1. Các sự kiện vòng đời & vì sao observer không chạy với mass update?

#### 📖 Giới thiệu kiến thức

**Danh sách event đầy đủ** (theo thứ tự khi xảy ra):

| Nhóm | Event | Thời điểm |
|------|-------|-----------|
| Đọc | `retrieved` | Sau khi model được lấy từ DB |
| Lưu | `saving` → `saved` | Bao quanh **mọi** lần `save()` (cả tạo lẫn cập nhật) |
| Tạo | `creating` → `created` | Chỉ khi **insert** bản ghi mới |
| Cập nhật | `updating` → `updated` | Chỉ khi **update** bản ghi đã có |
| Xóa | `deleting` → `deleted` | Bao quanh `delete()` |
| Soft delete | `trashed`, `restoring` → `restored` | Khi soft-delete / khôi phục |
| Khác | `replicating`, `forceDeleting` → `forceDeleted` | Khi nhân bản / xóa cứng |

**Phân biệt mấu chốt — `saving` vs `creating`/`updating`:** `saving`/`saved` chạy cho **cả** insert lẫn
update (logic dùng chung cho mọi lần ghi); còn `creating`/`updating` chỉ chạy cho đúng một trong hai.
Ví dụ: chuẩn hóa dữ liệu chung → để ở `saving`; gửi mail chào mừng (chỉ khi tạo mới) → để ở `created`.

**Cái bẫy kinh điển (hay được hỏi): observer KHÔNG chạy với mass update/delete.**

```php
// ❌ KHÔNG bắn updating/updated — không có model instance nào được load
User::where('active', false)->update(['active' => true]);

// ❌ KHÔNG bắn deleting/deleted
User::where('expired', true)->delete();

// ✅ Bắn đầy đủ event vì thao tác trên từng instance đã load
$user->update(['active' => true]);
User::find(1)->delete();
```

**Lý do:** event Eloquent gắn với **một instance model cụ thể**. Mass update/delete chạy thẳng một câu
SQL `UPDATE/DELETE ... WHERE ...` ở tầng query builder, **không hydrate** model nào → không có instance để
bắn event. Muốn chạy logic cho từng dòng thì phải lặp (`->get()` rồi `->each->update()`) — đánh đổi hiệu năng.

**Bẫy thứ hai — "quiet" bỏ qua event:** các method `saveQuietly()`, `updateQuietly()`,
`deleteQuietly()` (và block `Model::withoutEvents(fn () => ...)`) **cố ý không bắn** event → observer
không chạy. Hữu ích khi seed dữ liệu hoặc tránh vòng lặp event, nhưng phải nhớ là logic trong observer
sẽ bị bỏ qua.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Event vòng đời: `retrieved`, `saving/saved`, `creating/created`, `updating/updated`, `deleting/deleted`, `restoring/restored`, `trashed`, `replicating`, `forceDeleted`.
- `saving` chạy cho **cả tạo lẫn cập nhật**; `creating`/`updating` chỉ chạy cho **một trong hai** → đặt logic đúng chỗ.
- **Bẫy lớn:** observer **không chạy với mass update/delete** (`->where()->update()/delete()`) vì không có model instance được hydrate — chỉ là một câu SQL thuần.
- Các method **`saveQuietly`/`updateQuietly`/`deleteQuietly`** và `withoutEvents()` cũng **bỏ qua** event → observer im lặng.

[↑ Mục lục](#mục-lục)

<a id="q12-2"></a>
### 12.2. Observer vs Event/Listener — khi nào dùng cái nào?

#### 📖 Giới thiệu kiến thức

Cả hai đều dựa trên cùng cơ chế phát/lắng nghe (Observer pattern), nhưng khác nhau về **phạm vi** và
**mức tách rời**:

| | Eloquent Observer | Event / Listener |
|---|-------------------|------------------|
| Gắn với | **Vòng đời model** (cố định: creating, saved...) | **Bất kỳ sự việc nghiệp vụ nào** bạn tự định nghĩa |
| Sự kiện | Tập event có sẵn của Eloquent | Class event tùy ý (`OrderShipped`, `PaymentFailed`...) |
| Mục đích | Logic đi kèm thao tác CRUD của một model | Phản ứng với hành động nghiệp vụ, lan tỏa nhiều phía |
| Bất đồng bộ | Đồng bộ trong cùng request | Listener có thể `ShouldQueue` → chạy nền |
| Mức tách rời | Vừa phải (vẫn quanh model) | Cao — nhiều listener cho một event, ai cũng tự đăng ký |

**Quy tắc chọn:**
- **Observer:** việc xảy ra **mỗi khi một model được tạo/sửa/xóa** và **luôn đi kèm** thao tác đó —
  auto-fill field, set slug, clear cache theo bản ghi, audit log thay đổi của bảng.
- **Event/Listener:** một **sự kiện nghiệp vụ** không nhất thiết là 1 thao tác CRUD, hoặc cần **nhiều phía
  cùng phản ứng** / **chạy nền (queue)** — vd `OrderPaid` kéo theo: gửi hóa đơn, cộng điểm thưởng, thông
  báo kho. Mỗi việc là một listener độc lập, dễ bật/tắt.

Trên thực tế chúng **bổ sung cho nhau**: observer có thể **dispatch một event** để kích hoạt chuỗi xử lý
nghiệp vụ phức tạp (vd `created()` của observer `dispatch(new UserRegistered($user))`).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Observer** gắn cứng với **vòng đời model Eloquent** (CRUD) — dùng cho logic luôn đi kèm thao tác trên một model (slug, cache, audit log).
- **Event/Listener** cho **sự kiện nghiệp vụ tùy ý**, tách rời cao, **nhiều listener** cùng phản ứng và có thể **chạy nền qua queue** (`ShouldQueue`).
- Hai cái **bổ sung nhau**: observer có thể `dispatch` một event để khởi động chuỗi xử lý nghiệp vụ lớn hơn.

[↑ Mục lục](#mục-lục)

<a id="q13"></a>
## 13. Cache trong Laravel (định nghĩa & áp dụng)

### 📖 Giới thiệu kiến thức

**Cache** là cơ chế **lưu tạm kết quả của các thao tác tốn kém** (query DB nặng, gọi API ngoài, tính toán
phức tạp...) vào nơi **truy xuất nhanh** (bộ nhớ/Redis), để các lần sau **lấy lại ngay** thay vì làm lại từ
đầu → **giảm tải DB và tăng tốc độ phản hồi**.

Laravel cung cấp một **API thống nhất** cho nhiều backend (driver) qua **`Cache` facade** hoặc contract
`Illuminate\Contracts\Cache\Repository`. Đổi `CACHE_DRIVER` là đổi nơi lưu mà **không sửa code nghiệp vụ**
(cơ chế Strategy/Manager — xem câu 5).

**Các driver thường gặp:** `redis`, `memcached` (production — nhanh, chia sẻ giữa nhiều server),
`file`, `database` (đơn giản, không cần service ngoài), `array` (chỉ trong 1 request — hữu ích khi test).

**Các thao tác cốt lõi:**

```php
use Illuminate\Support\Facades\Cache;

Cache::put('key', $value, 600);          // lưu 600 giây (TTL); now()->addMinutes(10) cũng được
Cache::get('key', $default);             // lấy; không có thì trả $default
Cache::has('key');                       // kiểm tra tồn tại
Cache::forget('key');                    // xoá 1 key
Cache::forever('key', $value);           // lưu vĩnh viễn (tự quản lý xoá)
```

**`remember()` — pattern hay dùng nhất (cache-aside):** lấy từ cache nếu có; nếu chưa có thì **chạy
closure, lưu kết quả rồi trả về** — gói gọn "check → miss → query → set" trong một lời gọi:

```php
$users = Cache::remember('users:active', 600, function () {
    return User::where('active', true)->get();   // chỉ chạy khi cache miss
});

Cache::rememberForever('settings', fn () => Setting::all());
```

**Khi nào nên áp dụng:**
- Dữ liệu **đọc nhiều, ghi ít**, **ít thay đổi**: danh mục, cấu hình, menu, kết quả tổng hợp/report.
- Query/aggregate **tốn kém** hoặc gọi **API bên thứ ba** chậm.
- Giảm tải DB ở các trang **traffic cao**.

**Khi nào KHÔNG nên / cẩn thận:** dữ liệu **thay đổi liên tục** hoặc đòi hỏi **chính xác tuyệt đối tức
thời** (số dư tài khoản, tồn kho real-time) → cache dễ gây **stale data**; phải có chiến lược invalidation
(xem 13.1).

> Liên hệ: việc chọn driver theo `CACHE_DRIVER` là minh hoạ **Strategy + Manager pattern** (xem câu 5 &
> 5.1). Ngoài cache thủ công, Laravel còn có **query cache helper** và **cache trong config/route/view**
> (`php artisan config:cache`, `route:cache`).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Cache** = lưu tạm kết quả thao tác tốn kém vào nơi truy xuất nhanh (Redis/memory) để lần sau lấy ngay
  → **giảm tải DB, tăng tốc**.
- Laravel có **API thống nhất** qua `Cache` facade / contract; đổi **`CACHE_DRIVER`** (redis, memcached,
  file, database, array) **không cần sửa code** (Strategy/Manager — xem câu 5).
- Thao tác chính: `put/get/has/forget/forever`; **hay dùng nhất là `remember(key, ttl, closure)`**
  (cache-aside: miss thì chạy closure rồi lưu).
- **Áp dụng** cho dữ liệu đọc nhiều–ghi ít, query nặng, API ngoài; **tránh** cho dữ liệu thay đổi liên tục
  / cần chính xác tức thời (nguy cơ **stale data** → cần invalidation, xem 13.1).

[↑ Mục lục](#mục-lục)

<a id="q13-1"></a>
### 13.1. Cache invalidation & Cache tags

#### 📖 Giới thiệu kiến thức

> "There are only two hard things in Computer Science: cache invalidation and naming things." — vấn đề khó
> nhất của cache là **biết khi nào dữ liệu cache đã cũ và phải xoá/cập nhật** (stale data).

**Các chiến lược invalidation thường dùng:**

1. **TTL (time-based):** đặt thời gian sống cho key → tự hết hạn. Đơn giản nhất, chấp nhận dữ liệu "cũ" tối
   đa bằng TTL.
2. **Xoá chủ động khi dữ liệu đổi (write-through/event-based):** khi update model thì **`Cache::forget()`**
   key liên quan — thường đặt trong **Observer** (xem câu 12) để tự động:

```php
class PostObserver
{
    public function saved(Post $post): void
    {
        Cache::forget("post:{$post->id}");
        Cache::forget('posts:list');
    }
}
```

**Cache tags — xoá theo nhóm (chỉ driver hỗ trợ: redis, memcached; KHÔNG có ở `file`/`database`):**
gắn "nhãn" cho nhiều key để xoá cả nhóm trong một lệnh, thay vì nhớ từng key:

```php
Cache::tags(['posts', 'users'])->put('post:1', $post, 600);
Cache::tags(['posts'])->put('posts:list', $list, 600);

// Xoá MỌI key mang tag 'posts' cùng lúc
Cache::tags(['posts'])->flush();
```

**Lưu ý quan trọng:**
- **Cache stampede / thundering herd:** khi một key "hot" hết hạn, nhiều request cùng miss và đồng loạt
  query DB. Giảm bằng `Cache::remember` + **lock** (`Cache::lock()`) hoặc cập nhật cache nền.
- Tags **không dùng được với `file`/`database` driver** → cần redis/memcached.
- Nguyên tắc: cache key nên **có quy ước đặt tên rõ ràng** (`post:{id}`, `posts:list`) để invalidation dễ.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Cache invalidation** = giữ cache không bị **stale**; 2 cách chính: **TTL** (tự hết hạn) và **xoá chủ
  động** `Cache::forget()` khi dữ liệu đổi (thường đặt trong **Observer**).
- **Cache tags** (`Cache::tags([...])`) gắn nhãn nhiều key để **xoá cả nhóm** bằng `flush()` — chỉ hỗ trợ
  **redis/memcached**, không có ở `file`/`database`.
- Cảnh giác **cache stampede** (nhiều request cùng miss 1 key hot) → dùng `remember` + **`Cache::lock()`**.
- Đặt **quy ước tên key** rõ ràng để invalidation dễ kiểm soát.

[↑ Mục lục](#mục-lục)

<a id="q14"></a>
## 14. Các thư viện thường dùng với Composer trong Laravel

### 📖 Giới thiệu kiến thức

**Composer** là trình quản lý dependency của PHP. Trong Laravel, cài package bằng
`composer require vendor/package` (thêm **`--dev`** cho package chỉ dùng khi phát triển). Nhờ **package
auto-discovery**, hầu hết package **tự đăng ký** service provider/alias mà không cần khai báo tay (xem câu 2).

Phân loại theo mục đích các package phổ biến nhất:

| Nhóm | Package | Công dụng |
|------|---------|-----------|
| **Auth & API** | `laravel/sanctum` | Token API + xác thực SPA (xem 7.4) |
| | `laravel/passport` | OAuth2 server đầy đủ |
| | `laravel/socialite` | Đăng nhập mạng xã hội (Google, Facebook...) |
| | `laravel/breeze`, `laravel/jetstream` | Scaffolding auth (UI login/register) |
| **Phân quyền** | `spatie/laravel-permission` | Role & permission |
| **Queue** | `laravel/horizon` | Dashboard giám sát queue Redis |
| **Search** | `laravel/scout` | Full-text search (Algolia/Meilisearch) |
| **Payment** | `laravel/cashier` | Subscription billing (Stripe/Paddle) |
| **File/Media** | `spatie/laravel-medialibrary` | Quản lý file đính kèm model |
| | `intervention/image` | Xử lý/resize ảnh |
| | `barryvdh/laravel-dompdf` | Xuất PDF |
| | `maatwebsite/excel` | Import/export Excel/CSV |
| **HTTP client** | `guzzlehttp/guzzle` | HTTP client (nền của `Http` facade) |
| **Debug/Dev** | `barryvdh/laravel-debugbar` | Thanh debug (xem 8.2) |
| | `laravel/telescope` | Dashboard debug/monitor (xem 8.3) |
| | `laravel/pint` | Code style fixer (PSR-12) |
| | `laravel/sail` | Môi trường Docker dev |
| | `barryvdh/laravel-ide-helper` | Gợi ý IDE cho facade/model |
| **Testing** | `pestphp/pest`, `phpunit/phpunit` | Test framework |
| | `mockery/mockery`, `fakerphp/faker` | Mock & sinh dữ liệu giả |
| **Tiện ích** | `spatie/laravel-activitylog` | Ghi audit log |
| | `spatie/laravel-backup` | Backup DB/file |
| | `spatie/laravel-query-builder` | Build query từ query string của API |

> Mẹo nhận diện: **`laravel/*`** là package **chính chủ** (first-party, xem 14.1); **`spatie/*`** và
> **`barryvdh/*`** là 2 vendor **cộng đồng** uy tín, chất lượng cao, rất hay gặp trong dự án thực tế.

### 🎯 Tóm tắt trả lời phỏng vấn

- Cài bằng `composer require vendor/pkg` (`--dev` cho dev); **auto-discovery** tự đăng ký provider/alias.
- **Auth/API:** **Sanctum** (token/SPA), **Passport** (OAuth2), **Socialite** (social login); scaffolding
  **Breeze/Jetstream**. **Phân quyền:** **spatie/laravel-permission**.
- **Queue:** **Horizon** · **Search:** **Scout** · **Payment:** **Cashier**.
- **File/PDF/Excel/ảnh:** **medialibrary, laravel-dompdf, maatwebsite/excel, intervention/image** ·
  **HTTP:** **guzzle**.
- **Debug/dev:** **Debugbar (8.2), Telescope (8.3), Pint, Sail, ide-helper** · **Test:** **Pest/PHPUnit,
  Mockery, Faker**.
- Phân biệt **package chính chủ `laravel/*`** vs cộng đồng **`spatie/*`, `barryvdh/*`**.

[↑ Mục lục](#mục-lục)

<a id="q14-1"></a>
### 14.1. Các package chính chủ (first-party) của Laravel

#### 📖 Giới thiệu kiến thức

Ngoài framework, đội ngũ Laravel duy trì nhiều package **chính chủ** (`laravel/*`) giải quyết các nhu cầu
phổ biến — biết tên chúng giúp ghi điểm khi phỏng vấn:

| Package | Công dụng |
|---------|-----------|
| **Sanctum** | Token API nhẹ + auth cho SPA cùng domain (cookie) |
| **Passport** | OAuth2 server đầy đủ (access/refresh token) |
| **Fortify** | Backend xác thực headless (không kèm UI) |
| **Breeze / Jetstream** | Scaffolding auth: Breeze tối giản, Jetstream nhiều tính năng (2FA, team) |
| **Socialite** | Đăng nhập OAuth mạng xã hội |
| **Horizon** | Dashboard + cấu hình queue worker trên Redis |
| **Telescope** | Dashboard debug/monitoring (xem 8.3) |
| **Scout** | Full-text search driver-based (Meilisearch, Algolia) |
| **Cashier** | Quản lý subscription/billing (Stripe, Paddle) |
| **Dusk** | Browser testing (end-to-end, điều khiển trình duyệt thật) |
| **Octane** | Tăng tốc app bằng server thường trú (Swoole/RoadRunner/FrankenPHP) |
| **Pulse** | Giám sát hiệu năng & sức khỏe ứng dụng (real-time) |
| **Reverb / Echo** | WebSocket server (Reverb) + client broadcasting (Echo) |
| **Pennant** | Feature flags (bật/tắt tính năng) |
| **Sail / Pint** | Docker dev (Sail), code style fixer (Pint) |

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Auth:** **Sanctum** (token/SPA), **Passport** (OAuth2), **Fortify** (auth headless),
  **Breeze/Jetstream** (scaffolding), **Socialite** (social login).
- **Queue/Realtime:** **Horizon** (queue Redis), **Reverb/Echo** (WebSocket/broadcasting).
- **Hiệu năng/Giám sát:** **Octane** (server thường trú), **Telescope** (debug), **Pulse** (monitoring).
- **Khác:** **Scout** (search), **Cashier** (billing), **Dusk** (browser test), **Pennant** (feature
  flags), **Sail** (Docker dev), **Pint** (code style).

[↑ Mục lục](#mục-lục)

<a id="q14-2"></a>
### 14.2. Các authentication package (Sanctum vs Passport, Breeze/Jetstream/Fortify, Socialite)

#### 📖 Giới thiệu kiến thức

Laravel có sẵn **auth cơ bản** (guard `session`, `Auth` facade) cho web truyền thống. Khi cần hơn, dùng
các package **chính chủ**, chia theo vai trò:

**A. "Bộ máy" cấp/xác thực token (chọn cơ chế auth) — quan trọng nhất:**

- **Sanctum** — nhẹ, phổ biến nhất. Hai chế độ: **API token** (personal access token cho mobile/3rd-party)
  và **SPA authentication** dựa trên **cookie + session** cho SPA cùng domain (xem 7.5). Hợp API nội bộ,
  SPA, mobile — khi **không cần OAuth2 đầy đủ**.
- **Passport** — **OAuth2 server** đầy đủ (authorization code, client credentials, refresh token, scopes).
  Nặng hơn; dùng khi cần cấp token chuẩn **OAuth2 cho bên thứ ba**, nhiều client.

> **Sanctum vs Passport (câu kinh điển):** mặc định chọn **Sanctum** (đơn giản, đủ cho SPA/mobile/API nội
> bộ). Chỉ chọn **Passport** khi **thực sự cần OAuth2 server** (third-party developers, scope/grant chuẩn).

```php
// Sanctum: cấp API token
$token = $user->createToken('mobile')->plainTextToken;
// Client gửi: Authorization: Bearer <token>
// Bảo vệ route:
Route::middleware('auth:sanctum')->get('/me', fn (Request $r) => $r->user());
```

**B. Logic xác thực không kèm UI (headless):**

- **Fortify** — cung cấp **route + controller logic** cho login, register, reset password, email
  verification, **2FA**... **không có giao diện**. Dùng khi tự làm frontend (vd SPA) nhưng muốn tái dùng
  logic auth chuẩn. **Jetstream dùng Fortify** bên dưới.

> **SPA (Single Page Application) là gì?** Là ứng dụng web mà **toàn bộ giao diện do JavaScript dựng ở trình
> duyệt** (Vue, React, Angular...): tải **một trang HTML duy nhất**, sau đó tự đổi nội dung khi điều hướng
> mà **không reload cả trang**. Frontend (SPA) thường **tách riêng** khỏi backend Laravel và lấy dữ liệu qua
> **API JSON** — ngược với app truyền thống nơi Laravel render HTML bằng Blade mỗi request.
>
> **Ví dụ cụ thể — SPA Vue + Laravel API, auth bằng Sanctum (SPA mode):**
> 1. Frontend Vue chạy ở `app.example.com`; API Laravel ở `api.example.com` (cùng domain cha).
> 2. Trước khi đăng nhập, SPA gọi `GET /sanctum/csrf-cookie` → nhận cookie `XSRF-TOKEN` (xem 7.5).
> 3. SPA `POST /login` (email + password) → Laravel xác thực, **set session cookie**.
> 4. Các request sau (vd `GET /api/user`) trình duyệt **tự kèm cookie** → middleware `auth:sanctum` nhận ra
>    user mà không cần gửi token thủ công.
>
> Ở đây Laravel **không trả HTML giao diện**, chỉ trả **JSON**; phần **UI đăng nhập/đăng ký do bạn tự dựng**
> ở Vue, còn **logic auth** (kiểm tra credential, reset password, 2FA) thì **Fortify lo** — đó chính là ý
> "tự làm frontend (SPA) nhưng tái dùng logic auth chuẩn".

**C. Scaffolding (logic + UI dựng sẵn) — tạo nhanh:**

- **Breeze** — scaffolding **tối giản**: login/register/reset/verify + UI đơn giản (Blade, hoặc Vue/React
  qua Inertia, hoặc API stack). Hợp dự án nhỏ / học.
- **Jetstream** — scaffolding **nhiều tính năng**: 2FA, quản lý session, **team management**, API token
  (qua Sanctum), profile. Stack **Livewire** hoặc **Inertia (Vue/React)**.

**D. Đăng nhập mạng xã hội:**

- **Socialite** — OAuth **login qua provider** (Google, Facebook, GitHub, Twitter/X...). **Bổ trợ**, không
  thay thế cơ chế auth chính.

| Package | Vai trò | Khi nào dùng |
|---------|---------|--------------|
| **Sanctum** | Token API + SPA cookie auth | Mặc định cho API/SPA/mobile |
| **Passport** | OAuth2 server đầy đủ | Cần OAuth2 chuẩn cho bên thứ ba |
| **Fortify** | Logic auth **headless** | Tự làm UI, muốn logic auth có sẵn |
| **Breeze** | Scaffolding auth tối giản | Dự án nhỏ, khởi động nhanh |
| **Jetstream** | Scaffolding nhiều tính năng (2FA, team) | Dự án cần sẵn nhiều tính năng |
| **Socialite** | Đăng nhập OAuth mạng xã hội | Thêm login Google/FB/GitHub |

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Built-in:** guard `session` + `Auth` facade cho web cơ bản.
- **Cơ chế token/API:** **Sanctum** (nhẹ — API token + SPA cookie; **mặc định nên dùng**) vs **Passport**
  (**OAuth2 server** đầy đủ — khi cần cấp token chuẩn cho bên thứ ba).
- **Logic auth không UI:** **Fortify** (headless). **Scaffolding có UI:** **Breeze** (tối giản) vs
  **Jetstream** (2FA, team — dùng Sanctum + Fortify bên dưới).
- **Social login:** **Socialite** (Google/FB/GitHub...).
- **Chốt Sanctum vs Passport:** mặc định **Sanctum**; chỉ **Passport** khi cần **OAuth2 server** thực sự.

[↑ Mục lục](#mục-lục)

<a id="q15"></a>
## 15. Phân biệt SPA và MPA trong Laravel

### 📖 Giới thiệu kiến thức

Đây là **hai kiến trúc render giao diện** cho ứng dụng web; "trong Laravel" là hỏi Laravel hỗ trợ mỗi kiểu
thế nào.

**MPA (Multi-Page Application) — đa trang, render phía server.**
Mô hình truyền thống: mỗi lần điều hướng, **server trả về một trang HTML mới** (full page reload). Trong
Laravel: controller → render **Blade** → trả HTML hoàn chỉnh. Mỗi route là một trang.

**SPA (Single Page Application) — đơn trang, render phía client.**
Tải **một trang HTML duy nhất**, sau đó **JavaScript (Vue/React/Angular)** dựng và đổi giao diện ở trình
duyệt, lấy dữ liệu qua **API JSON**, **không reload** (xem ví dụ luồng ở 14.2). Trong Laravel: Laravel đóng
vai **API backend**, frontend là project Vue/React riêng.

**So sánh:**

| Tiêu chí | MPA (Blade) | SPA (Vue/React + API) |
|----------|-------------|------------------------|
| Render | **Server** render HTML mỗi request | **Client** render bằng JS, 1 trang HTML |
| Điều hướng | **Reload** cả trang | Đổi nội dung **không reload** (router client) |
| Tốc độ | Lần đầu nhanh; chuyển trang chậm hơn | Lần đầu tải JS nặng; chuyển trang **mượt** |
| SEO | **Tốt sẵn** (HTML đầy đủ) | Khó hơn (cần **SSR**/prerender) |
| Độ phức tạp | Đơn giản, ít JS | Phức tạp: build tool, state, API, auth tách |
| Auth | Session-cookie + Blade `@csrf` | Sanctum (SPA cookie/token), CSRF (xem 7.5) |
| Tải server | Server lo cả render | Server chỉ trả **JSON** (nhẹ render) |
| Trải nghiệm | Như web truyền thống | **App-like** (giống ứng dụng) |

**Cách Laravel hỗ trợ — 3 hướng (điểm "ăn điểm"):**

1. **MPA thuần — Blade:** controller trả view Blade. Đơn giản nhất, hợp SEO.
2. **SPA tách rời — Laravel API + Vue/React riêng:** Laravel chỉ là API (trả JSON); frontend là project
   riêng; auth qua **Sanctum** (xem 14.2 & 7.5).
3. **Hybrid — Inertia.js / Livewire** (rất Laravel):
   - **Inertia.js:** viết frontend bằng **Vue/React** nhưng **không cần xây API riêng** — controller trả
     "page" Inertia, trải nghiệm SPA (không reload) mà **routing/controller vẫn ở Laravel**. Gọi vui là
     "SPA mà không cần API".
   - **Livewire:** dựng UI **động bằng PHP/Blade** (component server-render, cập nhật qua AJAX ngầm) — cảm
     giác reactive mà **viết rất ít JavaScript**.

**Khi nào chọn gì:**
- **MPA/Blade:** site nội dung, trang admin đơn giản, cần **SEO**, team ít JS.
- **SPA tách rời:** app tương tác cao, **nhiều client dùng chung API** (web + mobile), có team frontend riêng.
- **Inertia/Livewire:** muốn trải nghiệm SPA nhưng giữ **sự đơn giản của Laravel monolith**, không muốn xây
  & duy trì API riêng.

### 🎯 Tóm tắt trả lời phỏng vấn

- **MPA:** server render HTML mỗi request (Laravel = **Blade**), điều hướng **reload cả trang** → đơn giản,
  **SEO tốt**, nhưng chuyển trang kém mượt.
- **SPA:** tải **1 trang HTML**, JS (Vue/React) render ở client, lấy data qua **API JSON**, **không reload**
  → mượt/app-like nhưng **phức tạp** (build tool, state, API, auth tách) và **khó SEO** (cần SSR).
- **Laravel hỗ trợ 3 hướng:** (1) **Blade** (MPA); (2) **API + Vue/React riêng** (SPA, auth Sanctum — xem
  14.2/7.5); (3) **Hybrid: Inertia** (SPA không cần API) / **Livewire** (reactive bằng PHP/Blade).
- **Chọn:** MPA cho site nội dung/SEO/đơn giản; SPA cho app tương tác cao & nhiều client; **Inertia/Livewire**
  khi muốn cảm giác SPA mà vẫn giữ Laravel monolith đơn giản.

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
