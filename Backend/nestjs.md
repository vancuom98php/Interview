<a id="mục-lục"></a>
# Backend — NestJS

> NestJS · Module/Controller/Provider · Dependency Injection & IoC · Middleware/Guard/Interceptor/Pipe/Filter ·
> Validation (DTO) · Dynamic Module · Lifecycle hooks · Testing · Microservices.
> (Câu hỏi về Node.js runtime/event loop nằm ở [nodejs.md](./nodejs.md); JavaScript thuần ở [js-core.md](./js-core.md).)

## 📑 Mục lục

1. [NestJS là gì? Vì sao dùng NestJS?](#q1)
2. [Building blocks: Module, Controller, Provider (Service)](#q2)
   - 2.1. [Use Case pattern là gì? (tổ chức business logic)](#q2-1)
3. [Dependency Injection & IoC trong NestJS](#q3)
   - 3.1. [Custom providers: useClass, useValue, useFactory, useExisting](#q3-1)
   - 3.2. [Injection scope: Singleton, Request, Transient](#q3-2)
   - 3.3. [Circular dependency & forwardRef](#q3-3)
4. [DTO (Data Transfer Object) là gì? Vì sao nên dùng class?](#q4)
   - 4.1. [DTO vs Entity khác nhau thế nào?](#q4-1)
   - 4.2. [Khi nào cần nhiều DTO cho một resource?](#q4-2)
5. [Phân biệt Middleware, Guard, Interceptor, Pipe, Exception Filter](#q5)
   - 5.1. [Pipe & Validation (DTO + class-validator)](#q5-1)
   - 5.2. [Guard & xác thực/phân quyền (JWT)](#q5-2)
   - 5.3. [Phân biệt Middleware và Guard](#q5-3)
   - 5.4. [Interceptor (transform, logging, cache)](#q5-4)
   - 5.5. [Phân biệt Guard và Interceptor](#q5-5)
   - 5.6. [Exception Filter (xử lý lỗi tập trung)](#q5-6)
   - 5.7. [Vòng đời đầy đủ của request (thứ tự chi tiết)](#q5-7)
6. [Dynamic Module & forRoot/forFeature (Global module)](#q6)
7. [Lifecycle hooks trong NestJS](#q7)
8. [Testing trong NestJS](#q8)
9. [Microservices trong NestJS](#q9)

---

<a id="q1"></a>
## 1. NestJS là gì? Vì sao dùng NestJS?

### 📖 Giới thiệu kiến thức

**Bối cảnh:** với Node.js, framework phổ biến nhất là **Express** — nhưng Express **rất tối giản
(unopinionated)**: không quy định cấu trúc thư mục, không có DI, không chuẩn tổ chức code. App nhỏ thì
nhanh, nhưng app lớn / nhiều người làm dễ **mỗi nơi một kiểu**, khó bảo trì và test.

**NestJS là gì:** là một **framework Node.js cấp cao (progressive framework)** viết bằng **TypeScript**,
xây **trên nền Express** (mặc định) hoặc **Fastify** (tùy chọn). Nest **không thay thế** Express mà **bọc
lên trên** để cung cấp một **kiến trúc có quy ước (opinionated architecture)** rõ ràng: **module, controller,
provider, dependency injection** — lấy cảm hứng mạnh từ **Angular** (ở frontend).

**Đặc điểm cốt lõi:**
- **TypeScript first + Decorator:** dùng `@Module()`, `@Controller()`, `@Injectable()`, `@Get()`... →
  code khai báo (declarative), rõ ràng, type-safe.
- **Kiến trúc module hóa:** app chia thành các **module** đóng gói tính năng → dễ mở rộng, tái sử dụng.
- **Dependency Injection tích hợp sẵn** (xem [câu 3](#q3)): Nest có **IoC container** tự quản lý và
  "tiêm" dependency → code **loosely coupled**, dễ test.
- **Platform-agnostic:** trừu tượng hóa HTTP layer nên **đổi Express ↔ Fastify** gần như không sửa code
  nghiệp vụ.
- **Hệ sinh thái tích hợp:** sẵn module cho TypeORM/Prisma, config, validation, auth (Passport), queue,
  WebSocket, GraphQL, **microservices** (xem [câu 9](#q9)).

**NestJS vs Express (điểm hay hỏi):**

| Tiêu chí | **Express** | **NestJS** |
|---|---|---|
| **Triết lý** | Tối giản, **unopinionated** | **Opinionated** (kiến trúc sẵn) |
| **Ngôn ngữ** | JS (TS phải tự cấu hình) | **TypeScript first** |
| **Cấu trúc** | Tự do, tự tổ chức | **Module/Controller/Provider** chuẩn |
| **DI** | Không có | **Có sẵn** (IoC container) |
| **Test** | Tự dựng | Hỗ trợ sẵn (`@nestjs/testing`) |
| **Hợp với** | App nhỏ, cần tự do tối đa | App **lớn, enterprise, nhiều người** |

> **Chốt:** Nest = "Express/Fastify + kiến trúc + DI + TypeScript + hệ sinh thái", đánh đổi một chút
> "tự do" và **learning curve** để lấy **tính nhất quán, khả năng mở rộng & bảo trì** cho app lớn.

### 🎯 Tóm tắt trả lời phỏng vấn

- **NestJS** = framework Node.js viết bằng **TypeScript**, xây **trên Express (hoặc Fastify)**, cung cấp
  **kiến trúc có quy ước** (module/controller/provider) lấy cảm hứng từ **Angular**.
- **Giải quyết:** Express quá tối giản → app lớn dễ lộn xộn. Nest áp **cấu trúc chuẩn + DI sẵn** → code
  **nhất quán, loosely coupled, dễ test & mở rộng**.
- **Điểm mạnh:** TypeScript + decorator, **module hóa**, **Dependency Injection**, platform-agnostic
  (Express↔Fastify), hệ sinh thái tích hợp (TypeORM, validation, auth, microservices...).
- **Chọn:** app **lớn/enterprise/team đông** → Nest; script nhỏ / cần tự do tối đa → Express thuần.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Building blocks: Module, Controller, Provider (Service)

### 📖 Giới thiệu kiến thức

Đây là **3 khối xây dựng nền tảng** của mọi app Nest. Hiểu vai trò và ranh giới của chúng là hiểu được
cách Nest tổ chức code (mô hình gần với **MVC + layered architecture**).

**① Controller — tầng nhận request (routing layer):**
Lo việc **nhận HTTP request và trả response**. Khai báo bằng `@Controller('route')`; mỗi method gắn
decorator HTTP (`@Get`, `@Post`, `@Param`, `@Body`...). Nguyên tắc quan trọng: **controller nên "mỏng"**
— chỉ điều phối, **không chứa business logic** (đẩy logic xuống service).

**② Provider / Service — tầng nghiệp vụ (business logic):**
"Provider" là khái niệm rộng chỉ **bất cứ thứ gì có thể được inject** (service, repository, factory,
helper...). Phổ biến nhất là **service** — class gắn `@Injectable()` chứa **logic nghiệp vụ** và thao tác
dữ liệu. Controller **inject** service để dùng (xem [câu 3](#q3)).

**③ Module — đơn vị đóng gói (organizational unit):**
Class gắn `@Module()` gom **controller + provider liên quan** thành một khối tính năng. App luôn có ít
nhất một **root module** (`AppModule`); từ đó chia thành các **feature module** (UsersModule, AuthModule...).
`@Module()` có 4 thuộc tính chính:
- **`controllers`**: các controller thuộc module.
- **`providers`**: các provider Nest sẽ khởi tạo & quản lý trong module này.
- **`imports`**: import module khác để **dùng provider được export** của nó.
- **`exports`**: "công khai" provider ra ngoài để module khác import dùng được.

```ts
// users.service.ts — nghiệp vụ
@Injectable()
export class UsersService {
  private users = [];
  findAll() { return this.users; }          // business logic ở đây
}

// users.controller.ts — chỉ điều phối, "mỏng"
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {} // DI
  @Get()
  findAll() { return this.usersService.findAll(); }
}

// users.module.ts — đóng gói
@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],                  // cho module khác dùng lại
})
export class UsersModule {}
```

> **Ranh giới tư duy:** Controller = "cửa vào HTTP" (mỏng); Service/Provider = "bộ não nghiệp vụ";
> Module = "hộp đóng gói + kết nối" các thành phần và điều khiển phạm vi hiển thị của provider
> (`exports`/`imports`).

### 🎯 Tóm tắt trả lời phỏng vấn

- **3 khối nền tảng:** **Controller** (nhận request/trả response — nên **mỏng**, không chứa logic),
  **Provider/Service** (`@Injectable()`, chứa **business logic**), **Module** (`@Module()`, **đóng gói**
  controller + provider theo tính năng).
- **`@Module()`** có 4 phần: **`controllers`**, **`providers`**, **`imports`** (dùng provider của module
  khác), **`exports`** (công khai provider ra ngoài).
- App có **root module** (`AppModule`) → chia thành **feature module** (UsersModule, AuthModule) để mở
  rộng, tái sử dụng.
- **Luồng:** request → Controller → gọi Service → trả kết quả. Controller **inject** Service qua DI.

[↑ Mục lục](#mục-lục)

<a id="q2-1"></a>
### 2.1. Use Case pattern là gì? (tổ chức business logic)

#### 📖 Giới thiệu kiến thức

Ở [câu 2](#q2), business logic nằm trong **Service**. Cách mặc định trong tutorial là **một Service "béo"**
(`UsersService` có 20 method, chục dependency). Khi app lớn dần, service này thành **god class**: khó đọc,
khó test, vi phạm **SRP** (Single Responsibility). **Use Case pattern** (đến từ **Clean Architecture /
Hexagonal**) là cách tổ chức khác: **tách mỗi thao tác nghiệp vụ thành một class riêng**.

**Use Case là gì:** một **Use Case** (còn gọi **Interactor** / **Application Service**) đại diện cho **đúng
một hành động nghiệp vụ của ứng dụng** — một việc mà hệ thống làm được (`CreateUser`, `PlaceOrder`,
`TransferMoney`). Đặc trưng:
- **Một use case = một class, thường một method public `execute()`** → SRP triệt để.
- **Điều phối (orchestrate):** nhận input (thường là **DTO/command**), phối hợp **domain entity** +
  **repository/port**, trả output. Nó **không chứa** chi tiết HTTP (đó là việc controller) hay chi tiết DB
  (đó là việc repository) — chỉ chứa **luồng nghiệp vụ**.

**Kiến trúc phân tầng đi kèm (Clean Architecture):**
```
Presentation (Controller)  →  Application (Use Cases)  →  Domain (Entity, rule)
                                      ↓ phụ thuộc abstraction (port/interface)
                               Infrastructure (Repository impl, DB, API ngoài)
```
Use case **chỉ phụ thuộc abstraction** (interface/port), không phụ thuộc implementation cụ thể → **dependency
inversion**. Nest wire implementation vào qua **custom provider token** (xem [câu 3.1](#q3-1)).

**Cài trong NestJS:** mỗi use case là một class **`@Injectable()`** với method `execute()`; controller
**mỏng**, chỉ gọi use case:
```ts
// application/use-cases/create-user.use-case.ts
@Injectable()
export class CreateUserUseCase {
  constructor(
    @Inject('UserRepository') private readonly users: UserRepositoryPort,  // port, không phải impl
    private readonly hasher: PasswordHasher,
  ) {}

  async execute(input: CreateUserDto): Promise<User> {
    if (await this.users.findByEmail(input.email))
      throw new ConflictException('Email đã tồn tại');
    const user = User.create(input.name, input.email, await this.hasher.hash(input.password));
    return this.users.save(user);                       // chỉ điều phối, không dính SQL
  }
}

// users.controller.ts — controller mỏng, mỗi route gọi 1 use case
@Post()
create(@Body() dto: CreateUserDto) {
  return this.createUserUseCase.execute(dto);
}
```

**Vì sao dùng (so với fat service):**
- **SRP & dễ đọc:** mỗi file một nghiệp vụ, tên class nói rõ ý định.
- **Dễ test:** test từng use case **cô lập**, chỉ mock vài port (xem [câu 8](#q8)) — không phải dựng cả
  service khổng lồ.
- **Decoupling:** logic **không dính framework/DB**; đổi ORM hay đổi Nest sang thứ khác ít ảnh hưởng.
- **OCP (mở rộng):** thêm nghiệp vụ = thêm use case mới, **không sửa** cái cũ.

**Đánh đổi (trade-off):** **nhiều class/boilerplate hơn**, và cần map giữa các tầng. → **overkill cho CRUD
đơn giản**; chỉ nên áp khi **domain phức tạp, nhiều rule, cần test kỹ & sống lâu**. App nhỏ dùng service
thường là đủ.

> **Chốt tư duy:** Use Case = "một class cho **một hành động nghiệp vụ**" (`execute()`), thay cho service
> béo → **SRP, testable, decoupled khỏi framework/DB**. Đổi lại nhiều boilerplate → dùng khi domain đủ phức
> tạp, không phải mặc định cho mọi app.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Use Case** (Interactor/Application Service — từ **Clean Architecture**) = **một class cho một thao tác
  nghiệp vụ**, thường một method **`execute()`**, thay cho **fat service** (god class vi phạm SRP).
- **Vai trò:** **điều phối** input (DTO) + domain entity + **repository/port** để chạy một luồng nghiệp vụ;
  **không** chứa chi tiết HTTP (controller lo) hay DB (repository lo). Phụ thuộc **abstraction/port** →
  dependency inversion, Nest wire impl qua **custom provider token**.
- **Lợi ích:** **SRP**, **dễ test cô lập** (mock port), **decoupling khỏi framework/DB**, **OCP** (thêm use
  case không sửa cũ).
- **Đánh đổi:** nhiều **boilerplate** → **overkill cho CRUD đơn giản**; dùng khi **domain phức tạp**.

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Dependency Injection & IoC trong NestJS

### 📖 Giới thiệu kiến thức

Đây là **trái tim của NestJS** và là câu hỏi phỏng vấn gần như chắc chắn được hỏi.

**Vấn đề nếu KHÔNG có DI:** nếu `UsersController` tự tạo dependency của mình (`new UsersService()`), thì
controller **phụ thuộc cứng (tight coupling)** vào cách khởi tạo service — khó thay thế, khó mock khi
test, và nếu service cần dependency khác thì phải sửa khắp nơi.

**IoC (Inversion of Control) — đảo ngược quyền điều khiển:** thay vì class **tự tạo** dependency, việc
tạo và cung cấp dependency được **giao cho một "container" bên ngoài**. Class chỉ **khai báo "tôi cần gì"**,
container lo phần "lấy ở đâu, tạo thế nào". → Class **không còn kiểm soát** vòng đời dependency (bị "đảo
ngược" cho framework).

**DI (Dependency Injection) — kỹ thuật hiện thực IoC:** container **"tiêm"** dependency vào class (phổ
biến nhất qua **constructor**). Trong Nest:
1. Provider gắn `@Injectable()` → đăng ký vào **IoC container**.
2. Khai báo provider trong `providers` của module.
3. Nơi cần chỉ việc **khai báo ở constructor** — Nest **tự resolve và inject**.

```ts
@Injectable()
export class UsersService { /* ... */ }

@Controller('users')
export class UsersController {
  // KHÔNG new UsersService — Nest tự tiêm vào nhờ kiểu ở constructor
  constructor(private readonly usersService: UsersService) {}
}
```

**Cơ chế hoạt động:** Nest đọc **metadata kiểu (type metadata)** ở constructor (nhờ TypeScript +
`reflect-metadata`) để biết cần inject gì, rồi tra trong container. Mỗi provider được nhận diện bằng
**token** — mặc định token chính là **class**, nhưng có thể là **string/symbol** cho custom provider
(xem [câu 3.1](#q3-1)).

**Lợi ích:**
- **Loose coupling:** đổi implementation không phải sửa nơi dùng.
- **Dễ test:** trong unit test, **override provider** bằng mock dễ dàng (xem [câu 8](#q8)).
- **Tái sử dụng & quản lý vòng đời tập trung** (singleton mặc định — xem [câu 3.2](#q3-2)).

### 🎯 Tóm tắt trả lời phỏng vấn

- **IoC** = đảo ngược quyền tạo dependency: class **không tự `new`**, mà **container** lo tạo & cung cấp.
  **DI** = kỹ thuật container **tiêm** dependency vào (thường qua **constructor**).
- **Trong Nest:** provider gắn **`@Injectable()`** → khai báo trong `providers` → nơi cần chỉ **khai báo
  ở constructor**, Nest tự resolve nhờ **type metadata** (`reflect-metadata`) và nhận diện qua **token**
  (mặc định là class).
- **Lợi ích:** **loose coupling**, **dễ test** (override bằng mock), quản lý vòng đời tập trung.
- Là **cơ chế cốt lõi** khiến code Nest module hóa, dễ bảo trì & mở rộng.

[↑ Mục lục](#mục-lục)

<a id="q3-1"></a>
### 3.1. Custom providers: useClass, useValue, useFactory, useExisting

#### 📖 Giới thiệu kiến thức

Cách khai báo `providers: [UsersService]` chỉ là **rút gọn** của `{ provide: UsersService, useClass:
UsersService }`. Khi cần linh hoạt hơn (inject giá trị, đổi implementation theo môi trường, tạo bất đồng
bộ...), dùng **custom provider** với cú pháp đầy đủ `{ provide: TOKEN, use... }`.

**4 dạng chính:**

- **`useClass`** — cung cấp **một class** cho token; có thể **đổi implementation** theo điều kiện.
  ```ts
  { provide: ConfigService, useClass: process.env.NODE_ENV === 'prod' ? ProdConfig : DevConfig }
  ```
- **`useValue`** — cung cấp **một giá trị/object có sẵn** (constant, mock, thư viện ngoài). Hay dùng với
  **token là string/symbol**:
  ```ts
  { provide: 'API_KEY', useValue: 'abc-123' }
  // Inject: constructor(@Inject('API_KEY') private key: string) {}
  ```
- **`useFactory`** — cung cấp giá trị **được tạo động bằng một factory function**; có thể **inject
  dependency khác** qua `inject`, và **hỗ trợ async** (trả Promise):
  ```ts
  {
    provide: 'DB_CONNECTION',
    useFactory: async (config: ConfigService) => createConnection(config.dbUrl),
    inject: [ConfigService],
  }
  ```
- **`useExisting`** — tạo **alias (bí danh)** trỏ tới provider đã có (nhiều token → cùng một instance):
  ```ts
  { provide: 'LOGGER', useExisting: ConsoleLogger }
  ```

> **Khi nào dùng `@Inject()`?** Khi token **không phải class** (string/symbol), TypeScript không suy ra
> được kiểu → phải chỉ token tường minh bằng `@Inject('TOKEN')`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Dạng `providers: [X]` là rút gọn của `{ provide: X, useClass: X }`. Custom provider dùng cú pháp
  `{ provide: TOKEN, use... }` để linh hoạt hơn.
- **4 dạng:** **`useClass`** (đổi implementation theo điều kiện) · **`useValue`** (giá trị/mock/constant
  có sẵn) · **`useFactory`** (tạo động, **inject được dependency**, **hỗ trợ async**) · **`useExisting`**
  (alias cho provider đã có).
- Token **không phải class** (string/symbol) → inject bằng **`@Inject('TOKEN')`**.

[↑ Mục lục](#mục-lục)

<a id="q3-2"></a>
### 3.2. Injection scope: Singleton, Request, Transient

#### 📖 Giới thiệu kiến thức

**Scope** quyết định **provider được tạo bao nhiêu instance và tồn tại bao lâu**. Mặc định Nest ưu tiên
hiệu năng bằng **singleton**.

**3 scope:**
- **`DEFAULT` (Singleton) — mặc định:** **một instance duy nhất** cho toàn app, chia sẻ mọi nơi. Tạo một
  lần lúc khởi động → **hiệu năng tốt nhất**. Phù hợp đa số trường hợp (service stateless).
- **`REQUEST`:** **tạo instance mới cho MỖI request**, hủy sau khi request xong. Dùng khi cần **state
  riêng theo request** (vd giữ thông tin user/tenant hiện tại). **Đánh đổi:** tốn tài nguyên hơn và
  "lây" scope — provider dùng nó cũng thành request-scoped.
- **`TRANSIENT`:** **mỗi nơi inject nhận một instance riêng** (không chia sẻ). Dùng khi mỗi consumer cần
  một bản độc lập.

```ts
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })   // instance mới mỗi request
export class RequestContextService {}
```

> **Lưu ý hiệu năng (hay hỏi):** ưu tiên **singleton**. `REQUEST` scope khiến Nest phải tạo lại cây
> dependency mỗi request → **chậm hơn**, và **lan truyền scope** lên các provider phụ thuộc. Chỉ dùng khi
> thật sự cần state theo request.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Scope** = số instance & vòng đời của provider. **3 loại:**
  - **DEFAULT (Singleton):** 1 instance toàn app — **mặc định**, hiệu năng tốt nhất (service stateless).
  - **REQUEST:** 1 instance **mỗi request** — cho state theo request (user/tenant); tốn tài nguyên hơn,
    **lây scope**.
  - **TRANSIENT:** mỗi nơi inject **một instance riêng**.
- **Chốt:** mặc định & nên ưu tiên **singleton**; chỉ dùng REQUEST khi cần state riêng theo request (chấp
  nhận chậm hơn).

[↑ Mục lục](#mục-lục)

<a id="q3-3"></a>
### 3.3. Circular dependency & forwardRef

#### 📖 Giới thiệu kiến thức

**Vấn đề — circular dependency (phụ thuộc vòng):** xảy ra khi **hai bên cần nhau tạo thành vòng tròn**.
Có 2 mức:
- **Provider ↔ Provider:** `CatsService` inject `DogsService`, mà `DogsService` lại inject `CatsService`.
- **Module ↔ Module:** `CatsModule` import `DogsModule`, mà `DogsModule` cũng import `CatsModule`.

Khi khởi tạo, **IoC container** (xem [câu 3](#q3)) phải tạo A trước để tiêm vào B, nhưng tạo A lại cần B
đã sẵn sàng → **"con gà & quả trứng"**. Container không biết tạo cái nào trước → báo lỗi kiểu *"Nest can't
resolve dependencies"* hoặc dependency bị **`undefined`**.

**`forwardRef` — giải pháp: trì hoãn (lazy) việc resolve tham chiếu:** `forwardRef(() => X)` bọc class
trong một **arrow function**. Nhờ đó Nest **không đọc `X` ngay lúc định nghĩa** (khi `X` có thể chưa được
khai báo) mà **gọi hàm để lấy `X` sau**, lúc mọi thứ đã nạp xong → phá được vòng "ai định nghĩa trước".

**Mấu chốt: phải dùng forwardRef ở CẢ HAI phía của vòng.**

**① Provider ↔ Provider** — dùng kèm `@Inject()`:

```ts
@Injectable()
export class CatsService {
  constructor(
    @Inject(forwardRef(() => DogsService))   // trì hoãn resolve DogsService
    private dogsService: DogsService,
  ) {}
}

@Injectable()
export class DogsService {
  constructor(
    @Inject(forwardRef(() => CatsService))   // BẮT BUỘC forwardRef ngược lại
    private catsService: CatsService,
  ) {}
}
```

**② Module ↔ Module** — bọc trong `imports`:

```ts
@Module({
  imports: [forwardRef(() => DogsModule)],   // cả 2 module cùng forwardRef
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}

@Module({
  imports: [forwardRef(() => CatsModule)],
  providers: [DogsService],
  exports: [DogsService],
})
export class DogsModule {}
```

**Lưu ý quan trọng:**
- Với forwardRef, lúc constructor chạy, instance bên kia **có thể chưa init xong** → **đừng gọi** service
  được inject **ngay trong constructor**; hãy gọi ở **method** (lúc runtime) hoặc trong `onModuleInit`.
- **forwardRef là "giải pháp chữa cháy", không phải thiết kế tốt.** Circular dependency thường là **dấu
  hiệu thiết kế chưa ổn**. Cách xử lý gốc rễ nên ưu tiên:
  1. **Tách phần dùng chung** ra một **service/module thứ ba** mà cả hai cùng phụ thuộc (phá vòng).
  2. Dùng **`ModuleRef`** để **lấy dependency động lúc runtime** (`this.moduleRef.get(X, { strict: false })`)
     thay vì inject qua constructor.
  3. Với logic liên hệ lỏng lẻo: dùng **event** (EventEmitter) để decouple.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Circular dependency** = hai bên cần nhau thành vòng (**provider ↔ provider** hoặc **module ↔ module**)
  → IoC container không biết tạo cái nào trước → lỗi *"can't resolve dependencies"* / `undefined`.
- **`forwardRef(() => X)`** bọc tham chiếu trong **arrow function** để Nest **resolve trễ (lazy)** → phá
  vòng. Dùng cho provider (kèm **`@Inject(forwardRef(...))`**) hoặc module (**`imports: [forwardRef(...)]`**).
- **Bắt buộc đặt forwardRef ở CẢ HAI phía** của vòng.
- **Lưu ý:** đừng gọi service inject **trong constructor** (chưa init xong); và forwardRef chỉ là **chữa
  cháy** — tốt hơn nên **refactor**: tách service/module thứ ba, hoặc dùng **`ModuleRef`** lấy động, hoặc
  **event** để decouple.

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. DTO (Data Transfer Object) là gì? Vì sao nên dùng class?

### 📖 Giới thiệu kiến thức

**DTO (Data Transfer Object)** là một **object dùng để định nghĩa *hình dạng dữ liệu* được truyền đi** —
điển hình là **payload của request** (`body`, `query`, `params`) hoặc dữ liệu trả về client. Nói ngắn gọn:
DTO là **"hợp đồng dữ liệu" (data contract)** giữa client và server, mô tả *"request này gồm những field nào,
kiểu gì"*. Bản thân DTO **không chứa business logic** — nó chỉ là cấu trúc dữ liệu.

**Vì sao cần DTO (giá trị nó mang lại):**
- **Type-safety & tự tài liệu hóa:** nhìn DTO là biết ngay endpoint nhận gì → code rõ ràng, IDE gợi ý tốt.
- **Một nguồn sự thật (single source of truth)** cho shape dữ liệu đầu vào → tránh mỗi controller tự đoán.
- **Nền tảng cho validation:** gắn decorator `class-validator` (`@IsEmail`, `@MinLength`...) lên DTO để
  **`ValidationPipe`** tự kiểm tra (xem [câu 5.1](#q5-1)).
- **Nền tảng cho transform:** `class-transformer` biến payload JSON thô thành **instance DTO** đúng kiểu.
- **Tách contract khỏi entity:** DTO ≠ **Entity** (model DB). Entity ánh xạ bảng DB; DTO là dữ liệu vào/ra
  API. Tách ra để **không lộ field nội bộ** (vd `password`, `isAdmin`) và **chống over-posting** (client
  gửi thừa field không mong muốn).

**Điểm mấu chốt hay hỏi — vì sao dùng `class` chứ không phải `interface`/`type`?**
`interface`/`type` của TypeScript **bị xóa hoàn toàn khi biên dịch (type erasure)** → lúc runtime **không
còn tồn tại**. Trong khi đó Nest cần DTO **tồn tại ở runtime** để:
1. **`ValidationPipe`** đọc **metadata** (qua `reflect-metadata`) và các **decorator** ràng buộc — decorator
   chỉ gắn được lên **class**.
2. **`class-transformer`** tạo được **instance** thật (`plainToClass`) để ép kiểu.

→ Vì vậy DTO trong Nest **luôn khai báo bằng `class`**.

```ts
// create-user.dto.ts — DTO cho dữ liệu tạo user
import { IsString, IsEmail, MinLength, IsOptional } from 'class-validator';

export class CreateUserDto {
  @IsString() @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsOptional() @IsString()
  bio?: string;
}

// controller: Nest bind body → instance CreateUserDto, ValidationPipe tự kiểm tra
@Post()
create(@Body() dto: CreateUserDto) {
  return this.usersService.create(dto);   // dto đã hợp lệ & đúng kiểu
}
```

**Tái sử dụng DTO với `@nestjs/mapped-types`** (tránh lặp lại): từ một DTO gốc sinh ra biến thể:
- **`PartialType(CreateUserDto)`** — mọi field thành **optional** (hợp cho `UpdateUserDto`).
- **`PickType` / `OmitType`** — lấy/bỏ một số field. **`IntersectionType`** — gộp 2 DTO.

```ts
export class UpdateUserDto extends PartialType(CreateUserDto) {}  // reuse, khỏi khai báo lại
```

> **Chốt tư duy:** DTO = "hợp đồng shape của dữ liệu vào/ra", **dùng `class`** để sống ở runtime (cho
> validation + transform), **tách khỏi Entity** để bảo vệ dữ liệu nội bộ. Nó là **input** của tầng
> Pipe/Validation (xem [câu 5.1](#q5-1)).

### 🎯 Tóm tắt trả lời phỏng vấn

- **DTO (Data Transfer Object)** = object định nghĩa **hình dạng dữ liệu truyền đi** (payload request/response) —
  một **"hợp đồng dữ liệu"**, **không chứa business logic**.
- **Lợi ích:** type-safe & tự tài liệu hóa, **một nguồn sự thật** cho input, **nền tảng cho validation**
  (class-validator) và **transform** (class-transformer), **tách contract khỏi Entity** (chống lộ field
  nội bộ & over-posting).
- **Vì sao dùng `class` chứ không `interface`?** interface/type **bị xóa khi compile** → không còn ở runtime;
  Nest cần class tồn tại runtime để đọc **metadata/decorator** (ValidationPipe) và tạo **instance**
  (class-transformer).
- **Tái sử dụng:** `@nestjs/mapped-types` — **`PartialType`** (update), `PickType`/`OmitType`/`IntersectionType`.

[↑ Mục lục](#mục-lục)

<a id="q4-1"></a>
### 4.1. DTO vs Entity khác nhau thế nào?

#### 📖 Giới thiệu kiến thức

Cả **DTO** và **Entity** đều là **class mô tả dữ liệu**, nên rất dễ nhầm — thậm chí nhiều dự án nhỏ dùng
luôn Entity làm DTO. Nhưng chúng phục vụ **hai mục đích khác nhau, sống ở hai tầng khác nhau**.

**① Entity — mô hình của tầng dữ liệu (persistence layer):**
Entity **ánh xạ tới một bảng trong database** (qua ORM: TypeORM `@Entity`, Prisma model, Mongoose schema).
Nó gắn **decorator của ORM** (`@Column`, `@PrimaryGeneratedColumn`, `@OneToMany`...), chứa **toàn bộ cột +
quan hệ** của bảng, kể cả **field nội bộ** (password hash, `role`, `isDeleted`, timestamp). Entity **dính
chặt với DB** — vòng đời của nó là vòng đời bản ghi.

```ts
// user.entity.ts — ánh xạ bảng, tầng persistence
@Entity('users')
export class User {
  @PrimaryGeneratedColumn() id: number;
  @Column() name: string;
  @Column() email: string;
  @Column() passwordHash: string;      // field nội bộ — KHÔNG bao giờ trả ra API
  @Column({ default: 'user' }) role: string;
  @OneToMany(() => Order, (o) => o.user) orders: Order[];
}
```

**② DTO — hợp đồng của tầng biên (boundary/transport layer):**
DTO mô tả **dữ liệu vào/ra qua API**, sống ở **tầng controller**. Nó gắn **decorator validation**
(`class-validator`), chỉ chứa **đúng field cần cho một thao tác cụ thể**, và **không biết gì về DB**. Vì
mỗi thao tác cần shape khác nhau nên thường có **nhiều DTO** cho cùng một resource (`CreateUserDto`,
`UpdateUserDto`, `UserResponseDto`).

```ts
// create-user.dto.ts — hợp đồng đầu vào, tầng boundary
export class CreateUserDto {
  @IsString() @MinLength(2) name: string;
  @IsEmail() email: string;
  @MinLength(8) password: string;      // nhận password thô, KHÔNG có passwordHash/role/id
}
```

**Bảng so sánh (điểm hay hỏi):**

| Tiêu chí | **DTO** | **Entity** |
|---|---|---|
| **Mục đích** | Hợp đồng dữ liệu **vào/ra API** | Ánh xạ **bảng DB** |
| **Tầng** | Boundary (**controller**) | Persistence/domain (**DB**) |
| **Decorator** | `class-validator` (`@IsEmail`...) | ORM (`@Entity`, `@Column`, relation) |
| **Phạm vi field** | **Chỉ** field của thao tác đó | **Toàn bộ** cột + quan hệ (cả field nội bộ) |
| **Coupling** | **Không** dính DB | **Dính** ORM/DB |
| **Số biến thể** | Nhiều (Create/Update/Response) | Thường **1 per bảng** |

**Vì sao KHÔNG nên dùng Entity trực tiếp làm DTO (mấu chốt):**
1. **Bảo mật đầu ra:** trả thẳng Entity dễ **lộ field nhạy cảm** (passwordHash, role, internal flag). DTO
   response chỉ phơi ra field an toàn.
2. **Bảo mật đầu vào (over-posting / mass-assignment):** nếu bind body thẳng vào Entity, client có thể gửi
   thừa `role: "admin"`, `isVerified: true`... để **leo quyền**. DTO + `whitelist: true` chặn field lạ.
3. **Decoupling:** đổi schema DB (đổi tên cột, tách bảng) **không phá vỡ API contract**, và ngược lại.
4. **Shape khác nhau:** request ≠ response ≠ bản ghi DB (vd nhận `password` thô, lưu `passwordHash`, trả về
   không có cả hai) → không thể ép chung một class.

**Mapping giữa hai bên:** service là nơi **chuyển DTO ↔ Entity** (thủ công, hoặc `class-transformer` /
AutoMapper). Với response, nên map Entity → **Response DTO** hoặc dùng **serialization** của Nest
(`ClassSerializerInterceptor` + `@Exclude()` trên field như `passwordHash`) thay vì trả entity thô.

```ts
// service: DTO vào → tạo Entity → lưu; entity ra → Response DTO
async create(dto: CreateUserDto): Promise<UserResponseDto> {
  const user = this.repo.create({ ...dto, passwordHash: hash(dto.password) }); // DTO → Entity
  const saved = await this.repo.save(user);
  return plainToInstance(UserResponseDto, saved, { excludeExtraneousValues: true }); // Entity → DTO
}
```

> **Chốt tư duy:** **Entity = "dữ liệu trông thế nào trong DB"** (persistence), **DTO = "dữ liệu đi qua API
> trông thế nào"** (contract). Tách hai lớp này ra để **bảo mật** (không lộ/không cho set field nội bộ) và
> **decoupling** (DB đổi không kéo theo API đổi).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Entity** = mô hình **tầng DB** (ORM `@Entity`/`@Column`), ánh xạ bảng, chứa **mọi cột + field nội bộ**,
  **dính DB**. **DTO** = hợp đồng **tầng API** (`class-validator`), chỉ chứa **field của một thao tác**,
  **không dính DB**, thường có **nhiều biến thể** (Create/Update/Response).
- **Vì sao tách:** (1) **bảo mật ra** — không lộ field nhạy cảm (passwordHash, role); (2) **bảo mật vào** —
  chống **over-posting/mass-assignment** (client gửi `role:admin`); (3) **decoupling** — đổi schema DB không
  phá API; (4) request/response/DB có **shape khác nhau**.
- **Mapping** ở **service** (thủ công / `class-transformer`); response nên qua **Response DTO** hoặc
  **`ClassSerializerInterceptor` + `@Exclude()`** thay vì trả Entity thô.

[↑ Mục lục](#mục-lục)

<a id="q4-2"></a>
### 4.2. Khi nào cần nhiều DTO cho một resource?

#### 📖 Giới thiệu kiến thức

Một resource (vd `User`) hiếm khi chỉ cần **một** DTO. Lý do gốc: **shape dữ liệu thay đổi theo *thao tác*,
*chiều truyền*, và *ngữ cảnh người xem*** — mà một DTO chỉ nên mô tả **đúng một hợp đồng**. Nhồi hết vào một
"god DTO" (field nào cũng `optional` + validation điều kiện chằng chịt) sẽ **mất type-safety và validation
kém chính xác**. Dưới đây là các trường hợp điển hình cần tách DTO:

**① Create vs Update (khác về field bắt buộc):**
Tạo mới yêu cầu **đủ field** (`CreateUserDto` — name, email, password đều required); cập nhật (PATCH) chỉ
gửi **một phần**, nên field thành **optional**. Dùng `PartialType` để tái sử dụng thay vì viết lại.
```ts
export class CreateUserDto { @IsString() name: string; @IsEmail() email: string; @MinLength(8) password: string; }
export class UpdateUserDto extends PartialType(CreateUserDto) {}   // mọi field optional
```

**② Request vs Response (khác theo chiều truyền):**
Input nhận `password` thô nhưng **không có** `id/createdAt`; output **không** trả `password` nhưng **có**
`id`, `createdAt`, và các field dẫn xuất. → cần `CreateUserDto` (vào) và `UserResponseDto` (ra) riêng.

**③ Theo vai trò / ngữ cảnh người xem (khác mức hiển thị):**
Cùng một user, **admin** thấy `email, role, isBanned`; **người dùng khác** chỉ thấy public profile
(`id, name, avatar`). → `AdminUserDto` vs `PublicUserDto` để **không lộ dữ liệu nhạy cảm theo ngữ cảnh**.

**④ Theo endpoint: list vs detail (khác độ "nặng"):**
Trang danh sách chỉ cần vài field nhẹ (`UserListItemDto`: id, name, avatar); trang chi tiết cần đầy đủ +
quan hệ (`UserDetailDto`: kèm orders, address...). Tách để **tránh over-fetching** và payload phình to.

**⑤ Theo thao tác đặc thù (sub-action):**
Những hành động không map 1–1 với CRUD cần DTO riêng, cực gọn: `ChangePasswordDto` (oldPassword,
newPassword), `UpdateEmailDto`, `ResetPasswordDto`... — validation sát đúng thao tác đó.

**⑥ Theo version API (giữ backward compatibility):**
`v1/UserDto` và `v2/UserDto` có shape khác nhau → tách để **đổi v2 không phá client dùng v1**.

**Cân bằng — đừng lạm dụng, dùng mapped-types để tái sử dụng:**
Nhiều DTO cho **rõ ràng, validation chính xác, an toàn** nhưng đổi lại **nhiều class + cần mapping**. Để
không lặp code, `@nestjs/mapped-types` (hoặc `@nestjs/swagger`) cung cấp: **`PartialType`** (optional hóa),
**`PickType`** (lấy vài field), **`OmitType`** (bỏ vài field), **`IntersectionType`** (gộp) — sinh biến thể
từ một DTO gốc.
```ts
export class UpdateUserDto extends PartialType(OmitType(CreateUserDto, ['password'] as const)) {}
```

> **Nguyên tắc:** **một DTO = một use-case/contract**. Tách khi shape khác nhau theo *thao tác / chiều /
> ngữ cảnh / version*; nhưng **tái sử dụng bằng mapped-types** để tránh bùng nổ class trùng lặp. Dự án
> CRUD siêu đơn giản có thể dùng ít DTO, nhưng production nên tách theo các trục trên.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cần **nhiều DTO** khi **shape dữ liệu đổi theo trục** khác nhau, vì mỗi DTO chỉ nên là **một hợp đồng**:
  - **Create vs Update** (required ↔ optional — `PartialType`).
  - **Request vs Response** (vào có `password`, ra có `id/createdAt`, giấu field nhạy cảm).
  - **Theo vai trò/ngữ cảnh** (`AdminUserDto` vs `PublicUserDto`).
  - **List vs Detail** (nhẹ ↔ đầy đủ + relations → tránh over-fetching).
  - **Sub-action** (`ChangePasswordDto`...), **version API** (v1 vs v2, giữ backward compat).
- **Đừng nhồi "god DTO"** (optional + validation điều kiện) → mất type-safety. **Tái sử dụng** bằng
  `PartialType`/`PickType`/`OmitType`/`IntersectionType` để tránh lặp.
- **Chốt:** *một DTO = một use-case*; tách theo thao tác/chiều/ngữ cảnh/version, tái dùng bằng mapped-types.

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Phân biệt Middleware, Guard, Interceptor, Pipe, Exception Filter

### 📖 Giới thiệu kiến thức

Đây là **5 loại "enhancer"** chèn vào **vòng đời xử lý request (request lifecycle)** — mỗi loại một nhiệm
vụ riêng. Câu hỏi kinh điển nhất là **thứ tự chạy** và **dùng cái nào cho việc gì**.

**Thứ tự trong request lifecycle (RẤT hay hỏi):**

```
Request
  → ① Middleware            (global → module-bound)
  → ② Guard                 (global → controller → route)  — quyết cho qua hay chặn
  → ③ Interceptor (before)  — bao quanh handler (phần "trước")
  → ④ Pipe                  — transform & validate input
  → ⑤ Route Handler (Controller) → Service
  → ⑥ Interceptor (after)   — biến đổi response (first-in, last-out)
  → ⑦ Exception Filter      — nếu có lỗi ném ra ở bất kỳ đâu
Response
```

**Vai trò từng loại:**
- **① Middleware:** hàm chạy **sớm nhất**, giống middleware Express (`req`, `res`, `next`). Truy cập
  request/response thô. Dùng cho: logging, CORS, gắn thuộc tính vào request. **Không** biết handler nào
  sẽ chạy.
- **② Guard:** trả **`true`/`false`** để **cho phép hay chặn** request tới handler. Dùng cho **xác thực /
  phân quyền (authentication/authorization)** — xem [câu 5.2](#q5-2).
- **③⑥ Interceptor:** **bao quanh** handler (AOP), can thiệp **cả trước lẫn sau**. Dùng: **biến đổi
  response**, logging thời gian, cache, bọc lỗi — xem [câu 5.4](#q5-4).
- **④ Pipe:** **transform** và **validate** dữ liệu đầu vào ngay trước handler. Dùng: validate DTO, ép
  kiểu param — xem [câu 5.1](#q5-1).
- **⑦ Exception Filter:** **bắt exception** ném ra và định dạng response lỗi — xem [câu 5.6](#q5-6).

**Bảng so sánh:**

| Loại | Nhiệm vụ chính | Đầu ra đặc trưng | Ví dụ dùng |
|---|---|---|---|
| **Middleware** | Xử lý request thô sớm nhất | gọi `next()` | Logging, CORS, gắn requestId |
| **Guard** | Cho qua / chặn | `true` / `false` | Auth, phân quyền (role) |
| **Interceptor** | Bao quanh handler (before+after) | biến đổi luồng (Observable) | Transform response, cache, đo thời gian |
| **Pipe** | Transform + validate input | giá trị đã xử lý (hoặc ném lỗi) | Validate DTO, ép kiểu `ParseIntPipe` |
| **Exception Filter** | Bắt & định dạng lỗi | response lỗi | Chuẩn hóa format lỗi API |

> **Mẹo nhớ:** **Guard** = "được vào không?"; **Pipe** = "dữ liệu vào có hợp lệ/đúng kiểu không?";
> **Interceptor** = "bọc trước–sau để biến đổi"; **Filter** = "có lỗi thì xử ra sao?"; **Middleware** =
> "chạm request sớm nhất kiểu Express".

### 🎯 Tóm tắt trả lời phỏng vấn

- **5 enhancer**, thứ tự: **Middleware → Guard → Interceptor(before) → Pipe → Handler → Interceptor(after)
  → Exception Filter**.
- **Middleware:** request thô sớm nhất (logging/CORS). **Guard:** trả `true/false` để **cho qua/chặn**
  (auth/phân quyền). **Interceptor:** **bao quanh** handler (transform response, cache, đo thời gian).
  **Pipe:** **transform + validate** input (DTO, ép kiểu). **Exception Filter:** **bắt & định dạng lỗi**.
- **Mẹo:** Guard = "được vào?", Pipe = "input hợp lệ?", Interceptor = "bọc trước–sau", Filter = "lỗi thì
  sao?", Middleware = "kiểu Express".

[↑ Mục lục](#mục-lục)

<a id="q5-1"></a>
### 5.1. Pipe & Validation (DTO + class-validator)

#### 📖 Giới thiệu kiến thức

**Pipe** hoạt động ngay **trước route handler**, làm 2 việc: **transformation** (ép kiểu/biến đổi dữ liệu)
và **validation** (kiểm tra hợp lệ, không hợp thì ném lỗi). Nest có sẵn `ParseIntPipe`, `ParseUUIDPipe`,
`ValidationPipe`...

**Validate bằng DTO + class-validator (cách chuẩn):**
- **DTO (Data Transfer Object):** một class mô tả **hình dạng dữ liệu** nhận vào, gắn **decorator ràng
  buộc** của `class-validator` (`@IsString`, `@IsEmail`, `@MinLength`...).
- **`ValidationPipe`:** đọc các decorator đó, tự validate `body`/`param`/`query` theo DTO; sai → tự trả
  **400 Bad Request** với chi tiết lỗi.

```ts
// create-user.dto.ts
export class CreateUserDto {
  @IsString() @MinLength(2)
  name: string;
  @IsEmail()
  email: string;
}

// bật global (main.ts): app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
@Post()
create(@Body() dto: CreateUserDto) {   // Nest tự validate theo DTO trước khi vào đây
  return this.usersService.create(dto);
}
```

> **Option hay dùng:** `whitelist: true` (loại field không khai báo trong DTO), `transform: true` (tự
> chuyển payload thành **instance của DTO** và ép kiểu primitive). Kết hợp `@nestjs/mapped-types`
> (`PartialType`) để tái dùng DTO cho update.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Pipe** chạy **trước handler**, làm **transform** (ép kiểu) + **validate** (sai thì ném lỗi). Có sẵn
  `ParseIntPipe`, `ValidationPipe`...
- **Validate chuẩn:** định nghĩa **DTO** + decorator **class-validator** (`@IsEmail`, `@MinLength`...),
  bật **`ValidationPipe`** → tự kiểm tra input, sai trả **400** kèm chi tiết.
- Option quan trọng: **`whitelist`** (bỏ field lạ), **`transform`** (payload → instance DTO, ép kiểu).

[↑ Mục lục](#mục-lục)

<a id="q5-2"></a>
### 5.2. Guard & xác thực/phân quyền (JWT)

#### 📖 Giới thiệu kiến thức

**Guard** là nơi chuẩn để làm **authentication (bạn là ai?)** và **authorization (bạn được làm gì?)**.
Guard implement `CanActivate`, trả **`true`** (cho qua) hoặc **`false`/ném lỗi** (chặn → thường 403/401).
Guard chạy **sau middleware, trước interceptor/pipe** → biết được sẽ vào route/controller nào (qua
`ExecutionContext`), nên hợp để đọc metadata (vd role yêu cầu).

**Auth với JWT (thường qua Passport — `@nestjs/passport`):**
- **AuthGuard('jwt'):** guard kiểm tra token JWT trong header, xác thực chữ ký, gắn `user` vào request.
- **Phân quyền theo role:** dùng **custom decorator** (`@Roles('admin')`) đặt metadata + một **RolesGuard**
  đọc metadata bằng **`Reflector`** rồi so với role của user.

```ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  canActivate(ctx: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', ctx.getHandler());
    if (!roles) return true;
    const { user } = ctx.switchToHttp().getRequest();
    return roles.includes(user.role);       // true = cho qua
  }
}

@Get('admin')
@Roles('admin')
@UseGuards(AuthGuard('jwt'), RolesGuard)     // xác thực rồi phân quyền
getAdminData() { /* ... */ }
```

> **Guard vs Middleware (hay hỏi):** middleware chạy sớm hơn nhưng **không biết** handler nào sẽ chạy;
> guard có **`ExecutionContext`** → biết controller/handler + đọc metadata → **hợp cho phân quyền**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Guard** implement `CanActivate`, trả **`true`/`false`** để **cho qua hay chặn** → nơi chuẩn cho
  **authentication & authorization**.
- **JWT:** dùng **`AuthGuard('jwt')`** (Passport) xác thực token & gắn `user`; **phân quyền** bằng
  `@Roles()` + **RolesGuard** đọc metadata qua **`Reflector`**.
- **Hơn middleware:** guard có **`ExecutionContext`** → biết handler đích & đọc metadata (role), phù hợp
  phân quyền.

[↑ Mục lục](#mục-lục)

<a id="q5-3"></a>
### 5.3. Phân biệt Middleware và Guard

#### 📖 Giới thiệu kiến thức

Cả **Middleware** và **Guard** đều chạy **sớm, trước route handler** và đều có thể **chặn request** — nên
rất hay bị hỏi so sánh. Nhưng chúng khác nhau về **nguồn gốc, khả năng "biết ngữ cảnh", và mục đích**.

**① Middleware — hàm chạy sớm nhất ở tầng HTTP (kế thừa từ Express):**
Middleware là khái niệm **của Express**, Nest dùng lại. Nó nhận **`(req, res, next)`**, làm việc trên
**request/response thô** và phải gọi **`next()`** để đi tiếp (hoặc kết thúc sớm bằng cách trả response).
Điểm mấu chốt: middleware chạy **trước cả guard**, nên **chưa biết** route/handler nào sẽ xử lý request
(**không có `ExecutionContext`**, không đọc được metadata như `@Roles`). Gắn middleware **không dùng
decorator** mà qua module implement `NestModule` + `configure()` và `forRoutes()` (theo **path**).

```ts
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.originalUrl}`);
    next();                              // phải gọi next() để đi tiếp
  }
}
@Module({ /* ... */ })
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*'); // gắn theo path, không theo handler
  }
}
```

**② Guard — "người gác cổng" của Nest, quyết cho qua hay chặn:**
Guard là khái niệm **riêng của Nest**, implement **`CanActivate`**, trả **`true`/`false`** (hoặc ném lỗi)
để **cho request vào handler hay không**. Guard chạy **sau middleware**, và quan trọng nhất: có
**`ExecutionContext`** → **biết chính xác controller/handler đích** và **đọc được metadata** qua
**`Reflector`** (vd role yêu cầu từ `@Roles('admin')`). Vì vậy guard là nơi chuẩn cho **authentication &
authorization**. Gắn guard bằng **decorator `@UseGuards()`** (global/controller/route) — **khai báo
(declarative)**, dễ áp cho **từng route** cụ thể.

**③ Vì sao KHÔNG dùng middleware để phân quyền?** Vì middleware **không biết** request sẽ vào handler nào
→ không đọc được metadata role gắn trên route. Guard có `ExecutionContext` nên làm được → phân quyền là
việc của guard, không phải middleware.

**Bảng so sánh:**

| Tiêu chí | **Middleware** | **Guard** |
|---|---|---|
| **Nguồn gốc** | Của Express (Nest dùng lại) | **Riêng của Nest** (`CanActivate`) |
| **Thứ tự chạy** | **Sớm nhất** | Sau middleware, trước interceptor/pipe |
| **Biết handler đích?** | ❌ Không (không có `ExecutionContext`) | ✅ Có → đọc metadata qua **`Reflector`** |
| **Chữ ký / đầu ra** | `(req, res, next)` → gọi `next()` | trả **`true`/`false`** (hoặc ném lỗi) |
| **Mục đích chính** | Request thô: logging, CORS, body-parse, gắn field | **Auth & phân quyền** (cho qua/chặn) |
| **Cách gắn** | `NestModule.configure()` + `forRoutes()` (theo **path**) | **`@UseGuards()`** decorator (theo **handler**) |
| **Truy cập DI** | Được (nếu class-based) | Được (first-class) |

> **Chốt tư duy:** Middleware = "chạm request **sớm nhất, thô nhất**, kiểu Express, chưa biết đích".
> Guard = "**người gác cổng của Nest**, biết đích + đọc metadata → quyết **cho vào hay chặn**". Việc
> cross-cutting cấp platform (log/CORS) → middleware; việc **auth/authorization** → guard.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai chạy **trước handler** và đều **chặn được request**, nhưng: **Middleware** là khái niệm **của
  Express** — chữ ký `(req, res, next)`, làm việc trên request **thô**, chạy **sớm nhất**, **không biết**
  handler đích (**không có `ExecutionContext`**). **Guard** là **của Nest** (`CanActivate`), trả
  **`true`/`false`**, chạy **sau middleware**, có **`ExecutionContext`** → biết handler & **đọc metadata**
  qua **`Reflector`**.
- **Mục đích:** middleware cho việc **thô/cross-cutting** (logging, CORS, body-parse, gắn requestId);
  guard cho **authentication & authorization** (cho qua/chặn theo role).
- **Cách gắn:** middleware qua **`NestModule.configure()` + `forRoutes()`** (theo **path**); guard qua
  **`@UseGuards()`** decorator (theo **handler/route** — declarative).
- **Mấu chốt:** không dùng middleware để phân quyền vì nó **không biết** route đích/metadata → đó là việc
  của **guard**.

[↑ Mục lục](#mục-lục)

<a id="q5-4"></a>
### 5.4. Interceptor (transform, logging, cache)

#### 📖 Giới thiệu kiến thức

**Interceptor** hiện thực **AOP (Aspect-Oriented Programming)**: **bao quanh** route handler, can thiệp
**trước và sau** khi handler chạy. Implement `NestInterceptor` với `intercept(context, next)`, trong đó
**`next.handle()`** trả về một **RxJS `Observable`** của response — ta dùng operator (`map`, `tap`,
`catchError`...) để biến đổi.

**Công dụng điển hình:**
- **Biến đổi response:** bọc mọi response về format chuẩn `{ data, statusCode, timestamp }`.
- **Logging / đo thời gian:** log trước khi vào handler, đo bằng `tap` sau khi xong.
- **Cache:** trả cache nếu có, bỏ qua handler.
- **Bọc/biến đổi lỗi:** đọc lỗi từ pipe/handler/service bằng `catchError`.

```ts
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(ctx: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    return next.handle().pipe(              // phần "after": biến đổi response
      map((data) => ({ data, timestamp: new Date().toISOString() })),
      tap(() => console.log(`⏱ ${Date.now() - now}ms`)),
    );
  }
}
```

> **Thứ tự resolve (hay hỏi):** interceptor chạy **first-in, last-out** — phần "before" theo thứ tự
> global→controller→route, còn phần "after" (response) chạy **ngược lại** route→controller→global (giống
> cơ chế bọc hàm).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Interceptor** = **AOP**: **bao quanh** handler, can thiệp **trước & sau**. `intercept(ctx, next)` với
  `next.handle()` trả **RxJS Observable** → biến đổi bằng `map`/`tap`/`catchError`.
- **Dùng cho:** **transform response** (format chuẩn), **logging/đo thời gian**, **cache**, **bọc lỗi**.
- Phần "after" resolve **first-in, last-out** (response đi ngược route→controller→global).

[↑ Mục lục](#mục-lục)

<a id="q5-5"></a>
### 5.5. Phân biệt Guard và Interceptor

#### 📖 Giới thiệu kiến thức

Guard và Interceptor đều là enhancer của Nest, đều chạy **sau middleware** và đều có **`ExecutionContext`**
(biết handler đích, đọc được metadata) — nên hay bị hỏi so sánh. Nhưng chúng khác nhau căn bản ở **"vai
trò"** và **"thời điểm can thiệp"**: **Guard là người gác cổng — quyết cho request VÀO hay không (nhị phân,
một chiều)**; **Interceptor bao quanh handler — biến đổi luồng CẢ trước lẫn sau (hai chiều)**.

**① Guard — "được vào hay không?" (một chiều, chặn/cho qua):**
Implement **`CanActivate`**, trả **`true`/`false`** (hoặc ném lỗi). Chạy **trước** khi handler được gọi và
chỉ có một nhiệm vụ: **cho request đi tiếp hay chặn** (thường cho **authentication/authorization** — xem
[câu 5.2](#q5-2)). Guard **không đụng vào response**, cũng **không bao quanh** handler — nó là một **cánh
cổng** một chiều: qua được thì đi tiếp, không thì dừng (đoản mạch sang exception filter).

**② Interceptor — "bọc trước & sau để biến đổi" (hai chiều, AOP):**
Implement **`NestInterceptor`** với `intercept(ctx, next)`, trong đó **`next.handle()`** trả **RxJS
Observable** của response. Interceptor **bao quanh** handler (xem [câu 5.4](#q5-4)): chạy code **trước**
handler, rồi **sau** handler **biến đổi/đọc response** (bằng `map`, `tap`, `catchError`...). Nó **không
quyết cho qua hay chặn** mà tập trung **transform response, logging/đo thời gian, cache, bọc lỗi**.

**③ Thứ tự & quan hệ trong lifecycle:** Guard chạy **trước** Interceptor(before). Nghĩa là **guard gác trước
tiên**; chỉ khi guard cho qua thì interceptor mới bắt đầu "bọc" quanh handler. Nếu guard `false` → **đoản
mạch**, interceptor phần "before" **có thể chưa kịp chạy trọn** và response đi thẳng tới exception filter
(xem lifecycle đầy đủ [câu 5.7](#q5-7)).

**Bảng so sánh:**

| Tiêu chí | **Guard** | **Interceptor** |
|---|---|---|
| **Interface** | `CanActivate` | `NestInterceptor` (`intercept`) |
| **Đầu ra** | `true`/`false` (hoặc ném lỗi) | **Observable** (biến đổi luồng) |
| **Chiều can thiệp** | **Một chiều** (chỉ trước handler) | **Hai chiều** (trước **và** sau handler) |
| **Đụng vào response?** | ❌ Không | ✅ Có (transform/bọc response) |
| **Mục đích chính** | **Auth & phân quyền** (cho qua/chặn) | **Transform response**, logging, cache, bọc lỗi |
| **Thứ tự** | Trước interceptor | Bao quanh handler (sau guard) |
| **Bản chất** | "Cổng" nhị phân | **AOP** — "củ hành" bọc quanh |

> **Chốt tư duy:** **Guard = "được vào không?"** (nhị phân, chặn/cho qua, không chạm response).
> **Interceptor = "bọc trước–sau để biến đổi"** (hai chiều, transform response/logging/cache). Cần **kiểm
> soát truy cập** → guard; cần **can thiệp/định dạng dữ liệu quanh handler** → interceptor.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Cả hai** chạy sau middleware, có **`ExecutionContext`**, nhưng khác **vai trò**: **Guard** (`CanActivate`)
  trả **`true`/`false`** để **cho qua/chặn** — **một chiều**, cho **auth/phân quyền**, **không đụng response**.
  **Interceptor** (`NestInterceptor`) **bao quanh** handler — **hai chiều** (trước & sau), trả **Observable**
  để **transform response, logging, cache, bọc lỗi**.
- **Thứ tự:** Guard chạy **trước** Interceptor; guard chặn thì **đoản mạch** sang exception filter.
- **Chốt:** Guard = "được vào không?"; Interceptor = "bọc trước–sau để biến đổi". Kiểm soát truy cập → guard;
  biến đổi dữ liệu quanh handler → interceptor.

[↑ Mục lục](#mục-lục)

<a id="q5-6"></a>
### 5.6. Exception Filter (xử lý lỗi tập trung)

#### 📖 Giới thiệu kiến thức

Nest có sẵn một **exception layer**: khi code ném lỗi mà không bắt, một **global exception filter** mặc
định sẽ bắt và trả response lỗi chuẩn. Nest khuyến khích ném **`HttpException`** (và các lớp con:
`NotFoundException`, `BadRequestException`, `UnauthorizedException`...).

```ts
throw new NotFoundException('User not found');  // → tự trả 404 với body chuẩn
```

**Custom Exception Filter:** khi muốn **tự định dạng response lỗi** (chuẩn hóa format cho toàn API, log
tập trung, đổi status...), tạo class implement `ExceptionFilter` + `@Catch()`:

```ts
@Catch(HttpException)
export class HttpErrorFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const res = host.switchToHttp().getResponse();
    const status = exception.getStatus();
    res.status(status).json({               // format lỗi thống nhất
      statusCode: status,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}
// dùng: @UseFilters(HttpErrorFilter) hoặc app.useGlobalFilters(new HttpErrorFilter());
```

> **`@Catch()` không tham số** = bắt **mọi** exception; `@Catch(HttpException)` = chỉ bắt loại chỉ định.
> Filter chạy **cuối cùng** trong lifecycle (xem [câu 5](#q5)).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Exception Filter** = tầng **bắt & định dạng lỗi**. Nest có filter mặc định; nên ném **`HttpException`**
  và các lớp con (`NotFoundException`...) → tự trả status + body chuẩn.
- **Custom filter:** implement `ExceptionFilter` + **`@Catch()`** để **chuẩn hóa format lỗi** toàn API,
  log tập trung. `@Catch()` trống = bắt mọi lỗi; `@Catch(HttpException)` = chỉ loại chỉ định.
- Filter chạy **cuối cùng** trong request lifecycle.

[↑ Mục lục](#mục-lục)

<a id="q5-7"></a>
### 5.7. Vòng đời đầy đủ của request (thứ tự chi tiết)

#### 📖 Giới thiệu kiến thức

[Câu 5](#q5) đã tóm gọn thứ tự 5 enhancer. Ở mức chi tiết, mỗi loại enhancer còn được **bind ở 3 cấp**
và chạy **theo cấp**: **global → controller → route**. Ngoài ra request có **2 nhánh**: **đường vào
(inbound)** đi tới handler, và **đường ra (outbound)** trả response — trong đó **interceptor** là loại
**duy nhất** tham gia cả hai chiều (bao quanh handler), nên phần "after" của nó chạy **ngược lại**.

**Trình tự đầy đủ theo tài liệu chính thức (từ khi request tới → response):**

```
      Incoming Request
        │  ── ĐƯỜNG VÀO (inbound) ──────────────────────────────▼
 1. Middleware            : global  → module-bound          (theo thứ tự đăng ký)
 2. Guards               : global  → controller → route     → false/throw = CHẶN ngay
 3. Interceptors (before): global  → controller → route     (phần "trước" của pipe RxJS)
 4. Pipes                : global  → controller → route → route-param  (transform + validate)
 5. Route Handler (Controller)  →  Service                  ← nơi xử lý nghiệp vụ
        │  ── ĐƯỜNG RA (outbound) ─────────────────────────────▲
 6. Interceptors (after) : route  → controller → global      (NGƯỢC lại — first-in, last-out)
 7. Exception Filters    : route  → controller → global      (CHỈ khi có exception ném ra)
        │
      Server Response
```

**Những điểm mấu chốt hay bị hỏi sâu:**

1. **Granularity 3 cấp (global → controller → route):** với mỗi stage, enhancer **global chạy trước**,
   rồi tới cấp **controller** (`@UseGuards` trên class), rồi cấp **route** (trên method). Đây là chi tiết
   [câu 5](#q5) chưa nêu.
2. **Interceptor bao quanh handler → phần "after" chạy NGƯỢC:** inbound global→controller→route, nhưng
   outbound (response) là route→controller→global (cơ chế bọc hàm, xem [câu 5.4](#q5-4)).
3. **Guard chặn = "đoản mạch" (short-circuit):** nếu guard trả `false`/ném lỗi, request **không đi tiếp**
   tới interceptor/pipe/handler mà **nhảy thẳng** sang **exception filter** (thường 403/401).
4. **Exception ở BẤT KỲ đâu** (guard, pipe, handler, service, interceptor) đều bị **exception filter**
   bắt; filter cũng resolve **route → controller → global** (cụ thể nhất trước).
5. **Middleware khác biệt:** chạy **sớm nhất**, ở tầng Express (`req/res/next`), **chưa biết** handler nào
   sẽ chạy (không có `ExecutionContext`) → không hợp làm phân quyền (đó là việc của guard — [câu 5.2](#q5-2)).
6. **Pipe chạy theo từng tham số** (`@Body`, `@Param`, `@Query`) ngay trước khi handler nhận giá trị.

**Làm rõ "3 cấp" & chiều resolve (điểm hay hỏi vặn):**

**① "3 cấp" là gì?** Mỗi enhancer gắn được ở **3 phạm vi**, và khi request khớp route thì **cả 3 cùng
tham gia**:

```ts
app.useGlobalGuards(new AuthGuard());   // ① GLOBAL  — toàn app (hoặc APP_GUARD qua DI)

@UseGuards(RolesGuard)                    // ② CONTROLLER — mọi route trong controller
@Controller('users')
export class UsersController {
  @UseGuards(OwnerGuard)                  // ③ ROUTE — chỉ đúng route này
  @Get(':id') findOne() {}
}
// Khi gọi GET /users/:id → chạy AuthGuard → RolesGuard → OwnerGuard (global→controller→route)
```

Trực giác **chiều vào**: đi từ **cổng ngoài cùng** (bảo vệ chung) → **cửa tầng** → **cửa phòng** — kiểm
tra chung trước, cụ thể sau. Áp dụng cho **guard, pipe, và phần "before" của interceptor**.

**② Vì sao interceptor phần "after" chạy NGƯỢC?** Vì interceptor **không chạy 1 lần mà "bọc" quanh
handler** (before + after), giống **hàm lồng hàm / củ hành (matryoshka)**:

```
global {
  controller {
    route {  HANDLER  }   ← lõi trong cùng
  }
}

Luồng:  global(before) → controller(before) → route(before)
              → HANDLER →
        route(after) → controller(after) → global(after)     ← NGƯỢC: hẹp → rộng
```

Đây là quy tắc **first-in, last-out** (giống stack): thằng **vào đầu tiên** (global) thì **ra cuối cùng**.
Gốc rễ: code *trước* `next.handle()` là "before", code trong `.pipe(map(...))` *sau* nó là "after"; vì
global gọi vào controller gọi vào route gọi handler, nên response "trồi ngược" qua route→controller→global.

**③ "Resolve ngược" của exception filter KHÁC với interceptor** — chỗ dễ nhầm nhất: với filter,
`route → controller → global` **không phải** "cả 3 cùng chạy", mà là **thứ tự Nest đi tìm filter nào bắt
lỗi — và CHỈ MỘT filter bắt**:

```
Có exception → tìm filter khớp @Catch(...) ở ROUTE?  có → dùng, DỪNG.
                 ↓ không
               tìm ở CONTROLLER?                       có → dùng, DỪNG.
                 ↓ không
               dùng GLOBAL (thường là default filter của Nest — lưới an toàn cuối).
```

→ "Ngược" ở đây = **filter càng gần route (cụ thể) càng được ưu tiên**; global chỉ là fallback. Khác hẳn
interceptor (**cả chuỗi** cùng chạy) — filter chỉ **1 cái trúng**.

> **Phân biệt với [Lifecycle hooks câu 7](#q7):** đây là vòng đời của **một request** (mỗi request một
> lần). Còn `onModuleInit`/`onApplicationShutdown`... là vòng đời của **ứng dụng & provider** (chạy lúc
> app bật/tắt, không theo request). Đừng lẫn hai loại "lifecycle".

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Thứ tự đầy đủ:** **Middleware → Guard → Interceptor(before) → Pipe → Handler/Service → Interceptor(after)
  → Exception Filter → Response.**
- **Mỗi enhancer bind 3 cấp** (global/controller/route) và **cả 3 cùng chạy** theo chiều vào **global →
  controller → route** (rộng→hẹp) — áp dụng cho guard, pipe, interceptor(before).
- **Interceptor phần "after" chạy NGƯỢC** (route → controller → global) vì nó **bọc** quanh handler như
  củ hành → **first-in, last-out**. Interceptor là loại **duy nhất** can thiệp **cả trước lẫn sau**.
- **Exception filter khác:** `route → controller → global` **không phải** "cả 3 cùng chạy" mà là **thứ tự
  ưu tiên chọn 1 filter** bắt lỗi (cụ thể nhất thắng, global là fallback). **Guard chặn** thì **đoản mạch**
  thẳng sang filter; **exception ở bất kỳ đâu** đều rơi vào filter.
- **Middleware** sớm nhất (tầng Express, chưa biết handler); **pipe** chạy **theo từng param**.
- **Đừng lẫn** với **lifecycle hooks** ([câu 7](#q7)) — đó là vòng đời **app/provider**, không phải request.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Dynamic Module & forRoot/forFeature (Global module)

### 📖 Giới thiệu kiến thức

**Static module** (`@Module({...})`) có cấu hình **cố định**. Nhưng nhiều module hạ tầng (Config,
TypeORM, JWT...) cần **nhận cấu hình khi import** (connection string, secret...) → cần **Dynamic Module**:
một module **được cấu hình lúc chạy** bằng cách gọi **method tĩnh trả về một `DynamicModule`**.

**Quy ước đặt tên (convention):**
- **`forRoot()` / `forRootAsync()`:** cấu hình **toàn cục, một lần** (vd `TypeOrmModule.forRoot({...})`,
  `ConfigModule.forRoot()`). `...Async` cho phép nhận config **bất đồng bộ** (đọc từ ConfigService).
- **`forFeature()`:** đăng ký **phần cụ thể** cho từng feature module (vd `TypeOrmModule.forFeature([User])`
  đăng ký repository cho entity `User`).

```ts
// tạo dynamic module
@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [{ provide: 'CONFIG_OPTIONS', useValue: options }, ConfigService],
      exports: [ConfigService],
      global: true,            // thành Global Module (khỏi import lại nhiều nơi)
    };
  }
}
// dùng: imports: [ConfigModule.forRoot({ folder: './config' })]
```

**Global Module:** module gắn **`@Global()`** (hoặc `global: true` trong DynamicModule) → provider export
của nó **dùng được ở mọi nơi mà không cần import lại**. Dùng cho thứ **dùng khắp app** (config, logger).
**Lưu ý:** lạm dụng `@Global` làm mất tính tường minh của dependency → chỉ dùng khi thật sự cần.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Dynamic Module** = module **nhận cấu hình khi import**, tạo bằng **method tĩnh trả `DynamicModule`**
  (khác static module cấu hình cố định).
- **Quy ước:** **`forRoot()/forRootAsync()`** cấu hình **toàn cục một lần** (TypeOrm, Config); **`forFeature()`**
  đăng ký **phần cụ thể** cho feature module (vd repository theo entity). `...Async` để nhận config bất đồng bộ.
- **Global Module** (`@Global()` / `global: true`): provider export **dùng khắp nơi khỏi import lại** —
  hợp config/logger, nhưng **đừng lạm dụng** (mất tính tường minh).

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. Lifecycle hooks trong NestJS

### 📖 Giới thiệu kiến thức

Ngoài vòng đời **request** (xem [câu 5](#q5)), bản thân **ứng dụng và provider** cũng có **vòng đời** —
Nest cung cấp **lifecycle hooks** để chèn logic vào lúc **khởi tạo** và **tắt (shutdown)**. Cài bằng cách
implement interface tương ứng.

**Giai đoạn khởi tạo (bootstrap):**
- **`onModuleInit()`** (`OnModuleInit`): chạy sau khi **module & các dependency của nó** đã khởi tạo xong.
  Hợp để **khởi tạo kết nối, seed dữ liệu, warm-up cache**.
- **`onApplicationBootstrap()`** (`OnApplicationBootstrap`): chạy sau khi **toàn bộ module** đã init xong
  và app sẵn sàng nhận request.

**Giai đoạn tắt (shutdown — cần bật `app.enableShutdownHooks()`):**
- **`onModuleDestroy()`**: khi nhận tín hiệu tắt (SIGTERM...), trước khi hủy.
- **`beforeApplicationShutdown()`**: có thể chờ tác vụ async trước khi đóng.
- **`onApplicationShutdown()`**: dọn dẹp cuối cùng (**đóng DB connection, flush log, ngắt queue**) →
  tránh mất dữ liệu / graceful shutdown.

```ts
@Injectable()
export class TasksService implements OnModuleInit, OnModuleDestroy {
  onModuleInit() { console.log('Khởi tạo: kết nối, seed...'); }
  onModuleDestroy() { console.log('Dọn dẹp: đóng kết nối...'); }
}
```

> **Ứng dụng thực tế:** **graceful shutdown** — khi deploy/scale (K8s gửi SIGTERM), dùng shutdown hooks để
> đóng kết nối DB, hoàn tất request đang chạy, ngắt consumer queue **an toàn** trước khi process chết.

### 🎯 Tóm tắt trả lời phỏng vấn

- Ngoài vòng đời request, **app & provider** có **lifecycle hooks** cho lúc **init** và **shutdown**.
- **Init:** **`onModuleInit`** (module & dependency đã sẵn sàng → kết nối/seed) → **`onApplicationBootstrap`**
  (toàn app đã init, sẵn sàng nhận request).
- **Shutdown** (bật `enableShutdownHooks()`): **`onModuleDestroy`** → **`beforeApplicationShutdown`** →
  **`onApplicationShutdown`** → dọn dẹp (đóng DB, flush log) cho **graceful shutdown**.
- **Dùng thực tế:** graceful shutdown khi deploy/scale (SIGTERM) để không mất dữ liệu / request dở dang.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. Testing trong NestJS

### 📖 Giới thiệu kiến thức

Nest được thiết kế **dễ test** nhờ **Dependency Injection** (xem [câu 3](#q3)): vì dependency được inject
(không `new` cứng), ta có thể **thay bằng mock** dễ dàng. Nest cung cấp package **`@nestjs/testing`**.

**Unit test (test 1 provider/controller, mock dependency):**
- Dùng **`Test.createTestingModule({...})`** tạo một **module test** trong bộ nhớ.
- **`.overrideProvider(X).useValue(mockX)`** để thay dependency thật bằng mock.
- `.compile()` rồi lấy instance qua `moduleRef.get(Service)`.

```ts
describe('UsersController', () => {
  let controller: UsersController;
  const mockService = { findAll: jest.fn(() => ['A']) };

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [UsersService],
    })
      .overrideProvider(UsersService)   // thay service thật bằng mock
      .useValue(mockService)
      .compile();
    controller = moduleRef.get(UsersController);
  });

  it('findAll trả danh sách', () => {
    expect(controller.findAll()).toEqual(['A']);
    expect(mockService.findAll).toHaveBeenCalled();
  });
});
```

**E2E test:** dùng `Test.createTestingModule({ imports: [AppModule] })` + **`supertest`** để gọi HTTP thật
qua `app.getHttpServer()`, kiểm tra cả pipeline (guard/pipe/interceptor).

> **Điểm mạnh nhờ DI:** vì mọi dependency đi qua container, việc **override bằng mock** rất gọn → test
> **cô lập (isolated)**, nhanh, không đụng DB/service ngoài. Đây là lý do kiến trúc DI của Nest được
> đánh giá cao về testability.

### 🎯 Tóm tắt trả lời phỏng vấn

- Nest **dễ test nhờ DI** — dependency inject nên **thay bằng mock** dễ. Dùng package **`@nestjs/testing`**.
- **Unit test:** **`Test.createTestingModule()`** + **`.overrideProvider(X).useValue(mock)`** + `.compile()`,
  rồi `moduleRef.get()`; mock dependency bằng `jest.fn()`.
- **E2E test:** import `AppModule` + **`supertest`** gọi `app.getHttpServer()` để kiểm tra cả pipeline.
- **Lợi ích:** test **cô lập**, nhanh, không phụ thuộc DB/service ngoài.

[↑ Mục lục](#mục-lục)

<a id="q9"></a>
## 9. Microservices trong NestJS

### 📖 Giới thiệu kiến thức

Ngoài HTTP app, Nest hỗ trợ xây **microservices** qua package **`@nestjs/microservices`**. Ở đây
"microservice" nghĩa là **ứng dụng dùng transport layer khác HTTP** để giao tiếp (message-based), không
nhất thiết là kiến trúc microservice đầy đủ.

**Transport layer (transporter):** Nest trừu tượng hóa nhiều cơ chế truyền tin — đổi transporter **gần
như không sửa code nghiệp vụ**:
- **TCP** (mặc định, đơn giản), **Redis** (pub/sub), **NATS**, **MQTT**, **RabbitMQ (RMQ)**, **Kafka**,
  **gRPC**.

**Hai kiểu giao tiếp (rất hay hỏi):**
- **Request–response (`@MessagePattern`):** client gửi message và **chờ kết quả trả về** (giống RPC).
  Client dùng `client.send(pattern, data)` → trả Observable.
- **Event-based (`@EventPattern`):** client **phát sự kiện và không chờ** phản hồi (fire-and-forget) —
  hợp mô hình event-driven. Client dùng `client.emit(pattern, data)`.

```ts
// Microservice phía nhận
@Controller()
export class MathController {
  @MessagePattern({ cmd: 'sum' })          // request-response
  sum(data: number[]): number { return data.reduce((a, b) => a + b, 0); }

  @EventPattern('user_created')            // event, không trả kết quả
  handleUserCreated(data: any) { /* xử lý side-effect */ }
}
```

> **Khi nào dùng:** tách hệ thống thành nhiều service giao tiếp qua message broker (Kafka/RabbitMQ) để
> **decoupling, scale độc lập, chịu tải cao**. Chọn `@MessagePattern` khi **cần kết quả**, `@EventPattern`
> khi chỉ **thông báo sự kiện** (không cần phản hồi).

### 🎯 Tóm tắt trả lời phỏng vấn

- Nest hỗ trợ **microservices** qua **`@nestjs/microservices`** — app dùng **transport layer khác HTTP**
  (message-based).
- **Transporter** đa dạng, đổi gần như không sửa code: **TCP** (mặc định), **Redis, NATS, MQTT, RabbitMQ,
  Kafka, gRPC**.
- **2 kiểu giao tiếp:** **`@MessagePattern`** = **request–response** (chờ kết quả, kiểu RPC, `client.send`);
  **`@EventPattern`** = **event** fire-and-forget (không chờ, `client.emit`).
- **Chọn:** cần kết quả → `@MessagePattern`; chỉ thông báo sự kiện → `@EventPattern`.

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
