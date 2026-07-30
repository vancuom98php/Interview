<a id="mục-lục"></a>
# Backend — Database System (Hệ CSDL)

> SQL · MySQL/PostgreSQL · Index · Transaction & ACID · Normalization · NoSQL · Redis · Tối ưu query.

## 📑 Mục lục

- [1. MySQL và PostgreSQL là gì? So sánh hai hệ CSDL](#q1)
  - [1.1. Ví dụ SQL so sánh MySQL vs PostgreSQL theo từng tiêu chí](#q1-1)
  - [1.2. So sánh CTE (`WITH`) giữa MySQL và PostgreSQL](#q1-2)
  - [1.3. Các kiểu dữ liệu nâng cao của PostgreSQL (JSONB, array, hstore, custom type, GIS)](#q1-3)
- [2. Các cách kết nối đến MySQL/PostgreSQL](#q2)
  - [2.1. So sánh mysqli, PDO (PDO_mysql), pg_connect, PDO (PDO_pgsql)](#q2-1)
  - [2.2. Hướng thủ tục (procedural) và hướng đối tượng (OOP) trong mysqli](#q2-2)
  - [2.3. Connection pooling là gì và vì sao cần?](#q2-3)
  - [2.4. 1000 user truy cập website thì cần mở bao nhiêu connection pool?](#q2-4)
  - [2.5. max_connections mặc định của MySQL và PostgreSQL là bao nhiêu?](#q2-5)
  - [2.6. Cách tăng max_connections an toàn khi bị lỗi 1040 (Too many connections)](#q2-6)
- [3. NoSQL là gì? So sánh NoSQL với SQL](#q3)
  - [3.1. Các loại NoSQL và cách chọn loại phù hợp](#q3-1)
  - [3.2. Vì sao NoSQL "scale ngang" còn SQL "scale dọc"?](#q3-2)
  - [3.3. MongoDB là gì?](#q3-3)
  - [3.4. Elasticsearch là gì?](#q3-4)
- [4. Thứ tự thực thi (logical query processing order) của các mệnh đề trong câu SQL](#q4)
  - [4.1. Phân biệt WHERE và HAVING trong SQL](#q4-1)
  - [4.2. Logic lọc theo điều kiện (vd kiểm tra range): đặt ở WHERE hay SELECT?](#q4-2)
- [5. Phân biệt LIKE và ILIKE trong SQL](#q5)
- [6. Phân biệt UNION và UNION ALL trong SQL](#q6)
- [7. JOIN và Subquery: cái nào nhanh hơn?](#q7)
- [8. So sánh Subquery và CTE trong SQL](#q8)
- [9. CTE đệ quy (Recursive CTE) là gì?](#q9)
- [10. Transaction trong SQL là gì?](#q10)
  - [10.1. Tính chất ACID của transaction](#q10-1)
  - [10.2. Deadlock trong SQL là gì?](#q10-2)
  - [10.3. Race condition là gì?](#q10-3)
  - [10.4. SELECT FOR UPDATE là gì?](#q10-4)
  - [10.5. Optimistic Locking là gì?](#q10-5)
- [11. Xoá dữ liệu có relationship (foreign key) trong MySQL](#q11)
- [12. Index trong DB: các loại, cơ chế lưu, cách query/EXPLAIN/debug](#q12)
  - [12.1. Phân tích B-Tree/B+Tree index và Hash index (kèm ví dụ)](#q12-1)
  - [12.2. Cơ chế SELECT có index nhanh hơn không index thế nào?](#q12-2)
  - [12.3. Phân biệt Clustered index vs Non-clustered index](#q12-3)
  - [12.4. Cách chọn cột nào để đánh index](#q12-4)
  - [12.5. Tối ưu index cho câu query có nhiều điều kiện WHERE](#q12-5)
  - [12.6. Page split trong index là gì?](#q12-6)
  - [12.7. Cách giảm phân mảnh (fragmentation) index trong MySQL](#q12-7)
  - [12.8. B+Tree index được lưu ở đâu?](#q12-8)
  - [12.9. So sánh PK BIGINT AUTO_INCREMENT vs PK UUID](#q12-9)
  - [12.10. Xử lý khi index quá lớn không đủ RAM](#q12-10)
  - [12.11. Xử lý khi query dùng nhiều index cùng lúc (Index Merge)](#q12-11)
- [13. Một số kỹ thuật tối ưu hoá SQL trong MySQL](#q13)
  - [13.1. EXPLAIN trong SQL — đọc execution plan](#q13-1)
  - [13.2. Full flow một câu query thực thi trong MySQL](#q13-2)
- [14. Partitioning trong SQL là gì?](#q14)
- [15. Làm thế nào để insert hàng triệu record vào DB hiệu quả?](#q15)
  - [15.1. Xử lý bản ghi trùng lặp khi insert dữ liệu vào bảng](#q15-1)
- [16. Tăng tốc truy vấn SQL ở tầng kiến trúc/hệ thống (cache, read replica, denormalization...)](#q16)
  - [16.1. Kỹ thuật Replication (nhân bản dữ liệu)](#q16-1)
  - [16.2. Hệ thống lớn check email đã dùng cực nhanh thế nào? (Bloom filter)](#q16-2)
- [17. Tại sao phải lưu timestamp (created_at/updated_at)?](#q17)
- [18. Redis — in-memory data store](#q18)
  - [18.1. Redis là gì? Cách sử dụng (data type, use case)](#q18-1)
  - [18.2. Vì sao lưu/đọc Redis nhanh hơn MySQL?](#q18-2)
  - [18.3. Cơ chế lưu key kèm TTL (thời gian sống) hoạt động thế nào?](#q18-3)
  - [18.4. Các ứng dụng thực tế của Redis](#q18-4)
  - [18.5. Thực tế hay dùng data type nào của Redis? Implement qua thư viện gì?](#q18-5)
  - [18.6. Phân biệt 2 thư viện `redis` (node-redis) và `ioredis`](#q18-6)
  - [18.7. Redis làm delayed job queue (hàng đợi job có trì hoãn) thế nào?](#q18-7)
  - [18.8. Delayed job queue khác job queue thường khi xử lý ở Redis thế nào?](#q18-8)
  - [18.9. BullMQ hoạt động thế nào bên trong Redis?](#q18-9)
  - [18.10. Redis Pub/Sub hoạt động thế nào?](#q18-10)
  - [18.11. Redis Stream hoạt động thế nào?](#q18-11)
  - [18.12. Redis distributed lock (khóa phân tán) hoạt động thế nào?](#q18-12)
  - [18.13. Redis — làm sao để tránh leak/rò rỉ dữ liệu (bảo mật)?](#q18-13)
  - [18.14. Redis — cơ chế lưu bền & khôi phục (restore) dữ liệu](#q18-14)
  - [18.15. Giải pháp data recovery hoàn chỉnh cho Redis (production playbook)](#q18-15)
  - [18.16. Redis Cluster hoạt động thế nào?](#q18-16)

---

<a id="q1"></a>
## 1. MySQL và PostgreSQL là gì? So sánh hai hệ CSDL

### 📖 Giới thiệu kiến thức

Cả hai đều là **RDBMS** (Relational Database Management System — hệ quản trị CSDL quan hệ) **mã nguồn mở,
miễn phí**, lưu dữ liệu dạng **bảng** (table) gồm hàng & cột, dùng ngôn ngữ **SQL** để truy vấn. Đây là hai
hệ CSDL quan hệ phổ biến nhất thế giới.

**MySQL là gì?**
- Ra đời 1995, hiện thuộc **Oracle**. Nổi tiếng **nhanh, nhẹ, dễ dùng, dễ triển khai**.
- Là lựa chọn kinh điển cho **web app** (bộ **LAMP**: Linux–Apache–MySQL–PHP), CMS như WordPress.
- Kiến trúc **pluggable storage engine**: **InnoDB** (mặc định, hỗ trợ transaction ACID + khóa ngoại),
  **MyISAM** (cũ, nhanh cho đọc nhưng không transaction).
- Ưu tiên **tốc độ đọc** và sự đơn giản; phù hợp ứng dụng CRUD, đọc nhiều.

**PostgreSQL là gì?**
- Ra đời 1996, phát triển bởi cộng đồng (PostgreSQL Global Development Group). Được mô tả là **"object-
  relational database"** — mạnh về **chuẩn SQL, tính đúng đắn và tính năng nâng cao**.
- Hỗ trợ nhiều kiểu dữ liệu & tính năng phong phú: **JSONB**, **array**, **hstore**, kiểu tự định nghĩa,
  **CTE đệ quy**, **window function**, **full-text search**, extension (vd **PostGIS** cho dữ liệu địa lý).
- Tuân thủ **ACID chặt chẽ**, xử lý tốt **truy vấn phức tạp**, ghi đồng thời nhiều nhờ cơ chế **MVCC** mạnh.
- Phù hợp hệ thống **phân tích, dữ liệu phức tạp, yêu cầu toàn vẹn cao** (tài chính, GIS, analytics).

**So sánh MySQL vs PostgreSQL:**

| Tiêu chí | MySQL | PostgreSQL |
|----------|-------|------------|
| Loại | Relational (RDBMS) | **Object-relational** (giàu tính năng hơn) |
| Triết lý | Nhanh, nhẹ, đơn giản | Đúng chuẩn SQL, mạnh, đầy đủ tính năng |
| Storage engine | Nhiều engine (InnoDB, MyISAM...) | Một engine thống nhất |
| Concurrency (ghi đồng thời) | Tốt (InnoDB) | **Rất tốt** — MVCC mạnh, ít khóa |
| Kiểu dữ liệu | Cơ bản + JSON | **Phong phú**: JSONB, array, hstore, custom type, GIS |
| Truy vấn phức tạp | Khá | **Xuất sắc** (CTE đệ quy, window, subquery lồng) |
| Read nặng / đơn giản | **Rất nhanh** | Nhanh |
| Write/analytics phức tạp | Khá | **Mạnh hơn** |
| Tuân thủ chuẩn SQL | Ít chặt hơn | **Chặt chẽ hơn** |
| Use case điển hình | Web app, CMS, CRUD, LAMP | Analytics, tài chính, GIS, dữ liệu phức tạp |

**Điểm chung:** đều open-source, miễn phí, đa nền tảng, dùng SQL, hỗ trợ transaction ACID (MySQL qua
InnoDB), index, replication, khóa ngoại.

### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai là **RDBMS mã nguồn mở** dùng SQL, lưu dữ liệu dạng bảng quan hệ; đều hỗ trợ ACID, index, replication.
- **MySQL:** nhanh, nhẹ, đơn giản, phổ biến cho **web app/CMS** (LAMP, WordPress); có nhiều **storage engine**
  (mặc định InnoDB). Mạnh ở **đọc nhiều, CRUD đơn giản**.
- **PostgreSQL:** "object-relational", **giàu tính năng & đúng chuẩn SQL** hơn — JSONB, array, custom type,
  CTE đệ quy, window function, PostGIS; **MVCC** mạnh cho ghi đồng thời. Mạnh ở **truy vấn phức tạp, analytics,
  toàn vẹn dữ liệu cao**.
- **Chọn thế nào:** cần **đơn giản, nhanh, đọc nhiều** → MySQL; cần **tính năng nâng cao, dữ liệu phức tạp,
  đúng đắn cao** → PostgreSQL.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Ví dụ SQL so sánh MySQL vs PostgreSQL theo từng tiêu chí

#### 📖 Giới thiệu kiến thức

Cùng làm một việc nhưng cú pháp/tính năng hai hệ khác nhau. Dưới đây là ví dụ theo từng tiêu chí trong bảng
so sánh.

**a) Chuỗi & phân biệt hoa–thường (case sensitivity).** MySQL so sánh chuỗi **không phân biệt hoa thường**
theo mặc định; PostgreSQL **có phân biệt**.

```sql
-- MySQL: 'ABC' = 'abc' cho ra TRUE (do collation ci - case-insensitive mặc định)
SELECT * FROM users WHERE name = 'an';   -- khớp cả 'An', 'AN'

-- PostgreSQL: phân biệt hoa thường → phải dùng ILIKE để không phân biệt
SELECT * FROM users WHERE name ILIKE 'an';   -- ILIKE = LIKE không phân biệt hoa thường
```

**b) Auto-increment (khóa tự tăng).** Cú pháp khác nhau rõ rệt.

```sql
-- MySQL
CREATE TABLE t (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100));

-- PostgreSQL (kiểu SERIAL hoặc IDENTITY chuẩn SQL)
CREATE TABLE t (id SERIAL PRIMARY KEY, name VARCHAR(100));
-- hoặc: id INT GENERATED ALWAYS AS IDENTITY
```

**c) Giới hạn số dòng (phân trang).** MySQL dùng `LIMIT ... OFFSET`; PostgreSQL hỗ trợ cả chuẩn SQL `FETCH`.

```sql
-- MySQL & PostgreSQL đều chạy được:
SELECT * FROM orders ORDER BY id LIMIT 10 OFFSET 20;

-- PostgreSQL còn hỗ trợ cú pháp chuẩn ANSI:
SELECT * FROM orders ORDER BY id OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;
```

**d) Upsert (insert hoặc update nếu trùng).**

```sql
-- MySQL
INSERT INTO users (id, name) VALUES (1, 'An')
ON DUPLICATE KEY UPDATE name = 'An';

-- PostgreSQL (chuẩn hơn, linh hoạt hơn)
INSERT INTO users (id, name) VALUES (1, 'An')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;
```

**e) Truy vấn phức tạp — CTE đệ quy (PostgreSQL rất mạnh).** Cả hai đều hỗ trợ (MySQL từ 8.0), nhưng đây là
điểm PostgreSQL được đánh giá cao cho dữ liệu phân cấp:

```sql
-- Duyệt cây danh mục cha–con (chạy trên cả hai, cú pháp giống nhau)
WITH RECURSIVE tree AS (
    SELECT id, parent_id, name FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.parent_id, c.name
    FROM categories c JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree;
```

**f) Kiểu dữ liệu đặc thù (PostgreSQL vượt trội — xem chi tiết mục 1.3).**

```sql
-- PostgreSQL: cột mảng native
CREATE TABLE posts (id SERIAL, tags TEXT[]);
SELECT * FROM posts WHERE 'sql' = ANY(tags);   -- tìm bài có tag 'sql'

-- MySQL không có kiểu ARRAY native → thường lưu JSON hoặc bảng phụ
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Case sensitivity:** MySQL mặc định **không** phân biệt hoa thường; PostgreSQL **có** (dùng `ILIKE` để bỏ qua).
- **Auto-increment:** MySQL `AUTO_INCREMENT`; PostgreSQL `SERIAL` / `IDENTITY`.
- **Upsert:** MySQL `ON DUPLICATE KEY UPDATE`; PostgreSQL `ON CONFLICT ... DO UPDATE` (linh hoạt hơn).
- **Phân trang:** cả hai có `LIMIT/OFFSET`; PostgreSQL thêm chuẩn ANSI `FETCH ... ROWS ONLY`.
- **Truy vấn phức tạp & kiểu dữ liệu:** PostgreSQL mạnh hơn (CTE đệ quy, **ARRAY/JSONB native**), MySQL
  thường phải lách bằng JSON hoặc bảng phụ.

[↑ Mục lục](#mục-lục)

<a id="q1-2"></a>
### 1.2. So sánh CTE (`WITH`) giữa MySQL và PostgreSQL

#### 📖 Giới thiệu kiến thức

**CTE (Common Table Expression)** là bảng tạm có tên, khai báo bằng `WITH`, dùng ngay trong một câu truy vấn
để **chia nhỏ query phức tạp cho dễ đọc** và viết **truy vấn đệ quy** (duyệt cây/đồ thị). Cả hai hệ đều hỗ
trợ, nhưng mức độ trưởng thành khác nhau.

**Điểm giống:** cú pháp cơ bản `WITH ... AS (...)` và `WITH RECURSIVE` giống nhau; đều theo chuẩn SQL.

```sql
-- Chạy giống nhau trên cả MySQL 8.0+ và PostgreSQL
WITH doanh_thu AS (
    SELECT user_id, SUM(total) AS tong FROM orders GROUP BY user_id
)
SELECT u.name, d.tong
FROM users u JOIN doanh_thu d ON d.user_id = u.id
WHERE d.tong > 1000;
```

**Khác biệt chính:**

| Tiêu chí | MySQL | PostgreSQL |
|----------|-------|------------|
| Có CTE từ | **MySQL 8.0** (2018) — bản cũ (5.7-) **không có** | Có **từ rất sớm** (8.4, 2009), rất trưởng thành |
| CTE đệ quy (`WITH RECURSIVE`) | Có (8.0+) | Có, ổn định lâu năm |
| **Materialization** (vật chất hóa) | Optimizer tự quyết, **không** có gợi ý thủ công | Điều khiển được: **`MATERIALIZED` / `NOT MATERIALIZED`** |
| CTE **ghi dữ liệu** (data-modifying) | **Không** cho `INSERT/UPDATE/DELETE` bên trong `WITH` | **Có** — `WITH ... AS (INSERT/UPDATE/DELETE ... RETURNING)` |
| Tối ưu truy vấn phức tạp | Khá | Mạnh hơn (kết hợp CTE + window/array/JSONB) |

**1. PostgreSQL kiểm soát materialization.** Từ PG 12, có thể ép CTE tính sẵn (materialize) hay nội tuyến
(inline) để tối ưu.

- **`MATERIALIZED` (vật chất hóa):** PostgreSQL **chạy hết CTE trước**, lưu kết quả vào một **bảng tạm trong
  bộ nhớ**, rồi câu ngoài mới đọc từ bảng tạm đó. CTE giống một "hàng rào tối ưu" (optimization fence) —
  optimizer **không** đẩy điều kiện của câu ngoài vào trong được.
- **`NOT MATERIALIZED` (nội tuyến/inline):** PostgreSQL **gộp (inline) định nghĩa CTE thẳng vào câu ngoài**
  như thể ta viết liền một câu. Nhờ vậy optimizer có thể **đẩy điều kiện `WHERE` xuống trong CTE** (predicate
  pushdown) và **dùng index**.

```sql
-- NOT MATERIALIZED: optimizer đẩy WHERE id = 5 vào trong CTE
WITH t AS NOT MATERIALIZED (
    SELECT * FROM big_table
)
SELECT * FROM t WHERE id = 5;
```

**Tại sao nội tuyến lại nhanh hơn ở ví dụ trên?**
- Nếu **materialize**: PostgreSQL chạy `SELECT * FROM big_table` **trước** → **quét toàn bộ** bảng (hàng
  triệu dòng) ra bảng tạm, **rồi mới** lọc `id = 5`. Rất tốn kém.
- Nếu **not materialized (inline)**: câu được gộp lại thành `SELECT * FROM big_table WHERE id = 5` → nếu cột
  `id` **có index**, PostgreSQL chỉ *index seek* đúng 1 dòng. Nhanh hơn hàng nghìn lần.

> **Mặc định (PG 12+):** CTE **được dùng đúng 1 lần và không có tác dụng phụ** sẽ được **inline tự động**.
> Nhưng nếu CTE **tốn kém và được tham chiếu nhiều lần**, nên ép **`MATERIALIZED`** để tính **một lần rồi
> tái dùng**, tránh tính lại mỗi lần. Đó là lý do PG cho ta **cả hai lựa chọn**.
>
> **MySQL:** không có cú pháp này — optimizer tự quyết materialize hay merge, **không can thiệp thủ công được**.

**2. PostgreSQL cho CTE ghi dữ liệu (data-modifying CTE).** Có thể `INSERT/UPDATE/DELETE` ngay trong `WITH` —
MySQL **không** hỗ trợ:

```sql
-- PostgreSQL: chuyển dòng đã xóa sang bảng archive trong 1 câu
WITH da_xoa AS (
    DELETE FROM orders WHERE created_at < '2020-01-01' RETURNING *
)
INSERT INTO orders_archive SELECT * FROM da_xoa;
```

**MySQL làm sao để lưu record đã xóa vào `orders_archive`?** Vì MySQL **không** cho `DELETE ... RETURNING`
trong CTE, ta tách thành **2 bước và bọc trong transaction** để đảm bảo nguyên tử (atomic): **COPY trước,
DELETE sau**.

```sql
-- MySQL: dùng transaction, insert (sao chép) TRƯỚC rồi mới delete
START TRANSACTION;
  -- (1) Sao chép các dòng cần xóa sang bảng archive
  INSERT INTO orders_archive
  SELECT * FROM orders WHERE created_at < '2020-01-01';

  -- (2) Xóa chính các dòng đó khỏi bảng gốc (cùng điều kiện)
  DELETE FROM orders WHERE created_at < '2020-01-01';
COMMIT;   -- cả 2 cùng thành công; nếu lỗi giữa chừng → ROLLBACK, không mất/lệch dữ liệu
```

> **Vì sao INSERT trước, DELETE sau?** Nếu xóa trước thì dữ liệu đã mất, không còn gì để sao chép. Bọc
> **transaction** để nếu bước (2) lỗi thì bước (1) cũng bị rollback → không có tình trạng "đã copy nhưng chưa
> xóa" hoặc ngược lại. (PostgreSQL gói cả hai vào **một câu** nên tự động nguyên tử; MySQL đạt điều đó bằng
> transaction.)
>
> **Cách khác (MySQL):** có thể dùng **trigger `BEFORE DELETE`** trên bảng `orders` để tự động ghi dòng sắp
> xóa vào `orders_archive`, khi đó chỉ cần chạy `DELETE` là archive tự xảy ra.

**3. CTE đệ quy — cả hai đều chạy được** (đã có ví dụ duyệt cây danh mục ở mục 1.1); đây là điểm PostgreSQL
được dùng lâu và ổn định, còn MySQL chỉ có từ 8.0.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **CTE (`WITH`)** = bảng tạm có tên trong 1 query, giúp chia nhỏ truy vấn phức tạp & viết truy vấn **đệ quy**.
  Cú pháp cơ bản giống nhau trên cả hai.
- **MySQL:** chỉ có CTE **từ 8.0** (bản 5.7 trở về trước không có); optimizer tự quyết materialization, **không**
  cho CTE ghi dữ liệu.
- **PostgreSQL:** hỗ trợ CTE **lâu đời & trưởng thành hơn**; điều khiển được **`MATERIALIZED`/`NOT MATERIALIZED`**;
  cho **data-modifying CTE** (`INSERT/UPDATE/DELETE ... RETURNING` trong `WITH`) — MySQL không có.
- Tóm lại: cả hai làm được CTE cơ bản & đệ quy; PostgreSQL **linh hoạt và mạnh hơn** cho truy vấn phức tạp.

[↑ Mục lục](#mục-lục)

<a id="q1-3"></a>
### 1.3. Các kiểu dữ liệu nâng cao của PostgreSQL (JSONB, array, hstore, custom type, GIS)

#### 📖 Giới thiệu kiến thức

Đây là nhóm tính năng khiến PostgreSQL được gọi là **"object-relational"** — vừa quan hệ, vừa hỗ trợ dữ liệu
phong phú mà MySQL không có (hoặc yếu hơn).

**1. JSONB — lưu JSON dạng nhị phân, có index.**
Khác `JSON` (lưu text nguyên bản), **`JSONB`** phân giải JSON thành dạng nhị phân → truy vấn nhanh, **index
được** (GIN index), hỗ trợ toán tử phong phú.

```sql
CREATE TABLE products (id SERIAL, attrs JSONB);
INSERT INTO products (attrs) VALUES ('{"color":"red","size":42}');

SELECT attrs->>'color' FROM products;          -- lấy giá trị field 'color' dạng text
SELECT * FROM products WHERE attrs @> '{"color":"red"}';  -- @> : chứa cặp key/value này
CREATE INDEX idx_attrs ON products USING GIN (attrs);     -- index cho truy vấn JSONB nhanh
```

**2. Array — mảng ngay trong 1 cột.**
Một cột chứa **nhiều giá trị cùng kiểu**, không cần bảng phụ.

```sql
CREATE TABLE posts (id SERIAL, tags TEXT[]);
INSERT INTO posts (tags) VALUES (ARRAY['sql','postgres','db']);

SELECT * FROM posts WHERE 'sql' = ANY(tags);   -- có tag 'sql'
SELECT * FROM posts WHERE tags @> ARRAY['db']; -- chứa 'db'
SELECT array_length(tags, 1) FROM posts;       -- số phần tử
```

**3. hstore — cặp key–value dạng chuỗi.**
Kiểu lưu **tập cặp `key => value`** (giá trị là text). Là tiền thân của JSONB; ngày nay JSONB thường được ưu
tiên hơn, nhưng hstore vẫn gọn khi dữ liệu chỉ là key–value phẳng.

```sql
CREATE EXTENSION IF NOT EXISTS hstore;
CREATE TABLE items (id SERIAL, props HSTORE);
INSERT INTO items (props) VALUES ('brand => sony, warranty => 12');

SELECT props->'brand' FROM items;              -- lấy value theo key
SELECT * FROM items WHERE props ? 'warranty';  -- ? : có tồn tại key 'warranty'?
```

**4. Custom type — kiểu dữ liệu tự định nghĩa.**
Tự tạo kiểu riêng: **ENUM**, **composite type** (kiểu gộp nhiều field), hoặc **domain** (kiểu có ràng buộc).

```sql
-- ENUM: tập giá trị cố định
CREATE TYPE mood AS ENUM ('happy', 'ok', 'sad');
CREATE TABLE people (name TEXT, current_mood mood);

-- Composite type: gộp nhiều field thành 1 kiểu
CREATE TYPE address AS (street TEXT, city TEXT, zip TEXT);
CREATE TABLE users (id SERIAL, addr address);

-- Domain: kiểu có ràng buộc sẵn
CREATE DOMAIN positive_int AS INT CHECK (VALUE > 0);
```

**5. GIS — dữ liệu địa lý (extension PostGIS).**
**PostGIS** biến PostgreSQL thành CSDL **không gian địa lý (spatial)** mạnh nhất: lưu điểm/đường/vùng
(`GEOMETRY`, `GEOGRAPHY`), tính khoảng cách, tìm điểm trong bán kính, giao/cắt vùng...

```sql
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE TABLE shops (id SERIAL, geom GEOGRAPHY(POINT));
INSERT INTO shops (geom) VALUES (ST_MakePoint(106.7, 10.77));  -- kinh độ, vĩ độ

-- Tìm cửa hàng trong bán kính 1km quanh một điểm
SELECT id FROM shops
WHERE ST_DWithin(geom, ST_MakePoint(106.7, 10.77)::GEOGRAPHY, 1000);
```

> **So với MySQL:** MySQL có `JSON` (không có `JSONB` với GIN index mạnh như PG), có kiểu **spatial** cơ bản
> nhưng **PostGIS của PostgreSQL mạnh & chuẩn hơn nhiều**; MySQL **không** có ARRAY, hstore hay custom
> composite/domain type native.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **JSONB:** JSON nhị phân, **index được (GIN)**, truy vấn nhanh với toán tử `->`, `->>`, `@>`. Hơn hẳn kiểu
  `JSON` text và hơn MySQL.
- **Array:** một cột chứa nhiều giá trị cùng kiểu (`TEXT[]`), truy vấn bằng `ANY`, `@>` — MySQL không có native.
- **hstore:** tập cặp `key => value` (text), tiền thân của JSONB; JSONB nay thường được ưu tiên.
- **Custom type:** tự định nghĩa **ENUM**, **composite type** (gộp field), **domain** (kiểu có ràng buộc).
- **GIS (PostGIS):** dữ liệu không gian địa lý (điểm/đường/vùng), tính khoảng cách & truy vấn theo bán kính —
  thế mạnh đặc trưng của PostgreSQL cho bản đồ/định vị.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Các cách kết nối đến MySQL/PostgreSQL

### 📖 Giới thiệu kiến thức

Dù kết nối bằng cách nào, một kết nối tới MySQL/PostgreSQL luôn cần đủ **thông tin kết nối (connection
parameters)**: **host** (địa chỉ máy chủ DB), **port** (MySQL mặc định **3306**, PostgreSQL **5432**),
**database** (tên CSDL), **user** & **password**, và tuỳ chọn **SSL/charset**. Điểm khác nhau giữa các "cách
kết nối" chỉ là **ai/công cụ nào** đứng ra mở kết nối đó.

**1. Kênh truyền (transport) — kết nối đi qua đâu?**

| Kênh | Mô tả | Khi dùng |
|------|-------|----------|
| **TCP/IP socket** (`host:port`) | Kết nối qua mạng bằng IP + port | Mặc định; DB ở máy khác hoặc container |
| **Unix domain socket** (file `.sock`) | Kết nối nội bộ qua file socket, **không qua mạng** | Cùng máy — **nhanh & an toàn hơn** (MySQL: `/var/run/mysqld/mysqld.sock`, PG: `/tmp/.s.PGSQL.5432`) |
| **Named pipe / shared memory** | Cơ chế IPC nội bộ trên **Windows** | Cùng máy trên Windows |
| **SSL/TLS** | Bọc mã hoá lên TCP | Kết nối qua Internet, yêu cầu bảo mật |
| **SSH tunnel** | Đi xuyên qua một máy trung gian bằng SSH | Truy cập DB trong mạng nội bộ/private |

**2. Các cách (công cụ) để mở kết nối:**

**a) CLI client chính thức** — công cụ dòng lệnh của hãng:

```bash
# MySQL
mysql -h 127.0.0.1 -P 3306 -u root -p mydb

# PostgreSQL
psql -h 127.0.0.1 -p 5432 -U postgres -d mydb
# hoặc dùng connection URI:
psql "postgresql://postgres:secret@127.0.0.1:5432/mydb"
```

**b) Connection string / DSN** — một chuỗi chuẩn hoá gói mọi tham số, hầu hết driver/ORM đều nhận:

```
# URI chuẩn
mysql://user:pass@host:3306/dbname?charset=utf8mb4
postgresql://user:pass@host:5432/dbname?sslmode=require

# PDO DSN (PHP) — không chứa user/pass (truyền riêng)
mysql:host=127.0.0.1;port=3306;dbname=mydb;charset=utf8mb4
pgsql:host=127.0.0.1;port=5432;dbname=mydb
```

**c) Native driver / thư viện trong ngôn ngữ lập trình** — code kết nối trực tiếp bằng driver nói **giao thức
(protocol)** của DB:

| Ngôn ngữ | MySQL | PostgreSQL |
|----------|-------|------------|
| **PHP** | `mysqli`, **PDO** (`PDO_mysql`) | `pg_connect`, **PDO** (`PDO_pgsql`) |
| **Node.js** | `mysql2` | `pg` (node-postgres) |
| **Python** | `mysql-connector`, `PyMySQL` | `psycopg2` / `psycopg3` |
| **Java** | **JDBC** (Connector/J) | **JDBC** (PgJDBC) |
| **Go** | `go-sql-driver/mysql` | `pgx`, `lib/pq` |

```php
// PHP — PDO (portable, cùng API cho MySQL & PostgreSQL, chống SQL injection bằng prepared statement)
$pdo = new PDO(
    'mysql:host=127.0.0.1;port=3306;dbname=mydb;charset=utf8mb4', // DSN
    'root', 'secret',
    [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
);
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = ?');
$stmt->execute([1]);
```

```javascript
// Node.js — PostgreSQL bằng pg (dùng Pool: xem mục 2.3)
const { Pool } = require('pg');
const pool = new Pool({ host: '127.0.0.1', port: 5432, user: 'postgres',
                        password: 'secret', database: 'mydb' });
const { rows } = await pool.query('SELECT * FROM users WHERE id = $1', [1]);
```

**d) ORM / Query Builder** — nằm **trên** native driver, cung cấp API hướng đối tượng, tự sinh SQL:

- **PHP/Laravel:** **Eloquent**/Query Builder (cấu hình trong `config/database.php`, bên dưới dùng PDO).
- **Node.js:** Prisma, TypeORM, Sequelize, Knex.
- **PHP thuần:** Doctrine. **Java:** Hibernate/JPA. **Python:** SQLAlchemy, Django ORM.

**e) GUI client (công cụ đồ hoạ)** — để quản trị/khám phá dữ liệu bằng tay:

- **MySQL:** MySQL Workbench, phpMyAdmin, HeidiSQL. **PostgreSQL:** pgAdmin.
- **Đa hệ:** DBeaver, TablePlus, DataGrip.

**3. Bảo mật khi kết nối:** không hard-code mật khẩu (dùng **biến môi trường**/secret manager); ưu tiên
**SSL/TLS** khi qua mạng; giới hạn quyền user (least privilege); dùng **prepared statement** để chống SQL
injection; với DB nội bộ nên đi qua **SSH tunnel / VPN** thay vì mở port ra Internet.

### 🎯 Tóm tắt trả lời phỏng vấn

- Mọi kết nối cần: **host, port (MySQL 3306 / PostgreSQL 5432), database, user, password** (+ SSL/charset).
- **Kênh truyền:** **TCP/IP** (`host:port`), **Unix socket** (cùng máy — nhanh/an toàn hơn), named pipe
  (Windows); bọc **SSL/TLS** hoặc **SSH tunnel** khi cần bảo mật.
- **Các cách mở kết nối:** (1) **CLI** chính thức (`mysql`, `psql`); (2) **connection string/DSN**
  (`mysql://…`, PDO DSN); (3) **native driver** trong code (PHP: mysqli/PDO; Node: mysql2/pg; Java: JDBC…);
  (4) **ORM/Query Builder** (Eloquent, Prisma, Hibernate — nằm trên driver); (5) **GUI client** (Workbench,
  DBeaver, pgAdmin).
- **Thực tế app:** hầu hết dùng **ORM → native driver**; nên bật **connection pooling** (mục 2.3) và giữ
  credential trong **biến môi trường**, ưu tiên SSL.

[↑ Mục lục](#mục-lục)

<a id="q2-1"></a>
### 2.1. So sánh mysqli, PDO (PDO_mysql), pg_connect, PDO (PDO_pgsql)

#### 📖 Giới thiệu kiến thức

Đây là các **native driver/extension của PHP** để nói chuyện trực tiếp với DB (cách (c) ở câu 2). Cần phân biệt
**hai trục**:

- **Trục extension:** `mysqli` (chỉ MySQL) · `pgsql` với hàm `pg_connect` (chỉ PostgreSQL) · **PDO** (lớp trừu
  tượng chung, chạy qua các **driver** `PDO_mysql`, `PDO_pgsql`, `PDO_sqlite`…).
- **PDO_mysql vs PDO_pgsql** chỉ là **cùng một API PDO** nhưng nạp driver khác nhau qua **DSN** — code gần như
  giống hệt, chỉ đổi tiền tố DSN (`mysql:` ↔ `pgsql:`).

> Extension cũ **`mysql_*`** (hàm `mysql_connect`) đã bị **gỡ bỏ từ PHP 7** — không dùng nữa (không có prepared
> statement, không an toàn).

**1. `mysqli` (MySQL improved).**
- **Chỉ MySQL/MariaDB.** Có **cả 2 kiểu API**: hướng thủ tục (procedural) và hướng đối tượng (OOP).
- Hỗ trợ prepared statement, transaction, stored procedure, **multi-query**, **truy vấn bất đồng bộ (async)**.
- Placeholder **chỉ dạng vị trí `?`** — **không có named parameter**; phải `bind_param` và **khai báo kiểu**
  (`i`, `s`, `d`, `b`).

**2. PDO (`PDO_mysql` / `PDO_pgsql`).**
- **Lớp trừu tượng đa CSDL** — 1 API dùng cho **12+ DB** (MySQL, PostgreSQL, SQLite, SQL Server…).
- **Chỉ OOP.** Hỗ trợ **named parameter (`:name`)** lẫn `?`; prepared statement, transaction.
- **Portable:** đổi DB thường chỉ cần đổi **DSN** (`mysql:` → `pgsql:`), code truy vấn gần như không đổi.
- Lưu ý: với MySQL, PDO **giả lập prepared statement** mặc định (`PDO::ATTR_EMULATE_PREPARES = true`); muốn
  prepared statement **thật** ở phía server thì set `false`.

**3. `pg_connect` (extension `pgsql`).**
- **Chỉ PostgreSQL.** API **thủ tục** (`pg_connect`, `pg_query`, `pg_query_params`, `pg_prepare`/`pg_execute`).
- Placeholder dạng **`$1, $2`** (kiểu PostgreSQL).
- Ưu điểm: phơi bày **tính năng đặc thù PostgreSQL** mà PDO không có — `COPY` (nạp dữ liệu khối lớn),
  **`LISTEN/NOTIFY`** (pub/sub), truy vấn async, thao tác large object.

**Bảng so sánh:**

| Tiêu chí | `mysqli` | **PDO** (`PDO_mysql`/`PDO_pgsql`) | `pg_connect` (`pgsql`) |
|----------|----------|-----------------------------------|------------------------|
| CSDL hỗ trợ | Chỉ **MySQL/MariaDB** | **12+ DB** (MySQL, PG, SQLite…) | Chỉ **PostgreSQL** |
| Kiểu API | Thủ tục **+** OOP | **Chỉ OOP** | **Chỉ thủ tục** |
| Named parameter (`:name`) | **Không** (chỉ `?`) | **Có** (`:name` và `?`) | Không (dùng `$1,$2`) |
| Portable (đổi DB) | Không | **Có** (đổi DSN) | Không |
| Prepared statement | Có | Có (MySQL mặc định **giả lập**) | Có (`pg_prepare`) |
| Tính năng đặc thù DB | MySQL (multi-query, async) | Mẫu số chung, ít feature riêng | **PG đầy đủ** (COPY, LISTEN/NOTIFY) |
| Khuyến nghị | Khi **chỉ MySQL** & cần feature mysqli | ✅ **Mặc định nên dùng** | Khi cần **feature PG đặc thù** |

**Ví dụ — cùng một truy vấn qua 3 driver:**

```php
// (1) mysqli (OOP) — chỉ MySQL, placeholder '?', phải khai báo kiểu
$mysqli = new mysqli('127.0.0.1', 'root', 'secret', 'mydb', 3306);
$stmt = $mysqli->prepare('SELECT * FROM users WHERE id = ?');
$stmt->bind_param('i', $id);          // 'i' = integer
$stmt->execute();
$rows = $stmt->get_result()->fetch_all(MYSQLI_ASSOC);

// (2) PDO — MySQL hay PostgreSQL chỉ khác DSN; named parameter :id
$dsn = 'pgsql:host=127.0.0.1;port=5432;dbname=mydb';   // đổi 'mysql:...' là sang MySQL
$pdo = new PDO($dsn, 'postgres', 'secret',
    [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]);
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = :id');
$stmt->execute(['id' => $id]);
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

// (3) pg_connect — chỉ PostgreSQL, thủ tục, placeholder $1
$conn = pg_connect('host=127.0.0.1 port=5432 dbname=mydb user=postgres password=secret');
$res  = pg_query_params($conn, 'SELECT * FROM users WHERE id = $1', [$id]);
$rows = pg_fetch_all($res);
```

> **Vì sao PDO thường được ưu tiên (và là nền của Laravel):** một API sạch, OOP, **named parameter**, **portable**
> giữa MySQL/PostgreSQL, prepared statement chống SQL injection. Chỉ chọn `mysqli` khi chắc chắn chỉ dùng MySQL
> và cần tính năng riêng của nó; chọn `pg_connect` khi cần khai thác **tính năng đặc thù PostgreSQL** (COPY,
> LISTEN/NOTIFY) mà PDO không phơi bày.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Ba lựa chọn native driver PHP: **`mysqli`** (chỉ MySQL), **`pg_connect`/`pgsql`** (chỉ PostgreSQL), và **PDO**
  (lớp trừu tượng đa DB, chạy qua driver `PDO_mysql`/`PDO_pgsql`). Extension cũ `mysql_*` đã **bị gỡ từ PHP 7**.
- **`mysqli`:** MySQL-only, có cả procedural & OOP, placeholder **chỉ `?`** (không named param), hỗ trợ
  multi-query/async đặc thù MySQL.
- **PDO:** **chỉ OOP**, **đa CSDL** (đổi DSN là đổi DB), có **named parameter `:name`**, prepared statement
  (MySQL mặc định giả lập — set `ATTR_EMULATE_PREPARES=false` để prepared thật). **`PDO_mysql` vs `PDO_pgsql`
  chỉ khác DSN**, code gần như giống hệt.
- **`pg_connect`:** PostgreSQL-only, **thủ tục**, placeholder `$1,$2`, phơi bày **feature PG đặc thù** (COPY,
  LISTEN/NOTIFY).
- **Chọn:** ✅ **PDO là mặc định** (portable, sạch, chống SQL injection); `mysqli` nếu chỉ MySQL + cần feature
  riêng; `pg_connect` nếu cần tính năng đặc thù PostgreSQL. **Laravel** dùng **PDO** bên dưới.

[↑ Mục lục](#mục-lục)

<a id="q2-2"></a>
### 2.2. Hướng thủ tục (procedural) và hướng đối tượng (OOP) trong mysqli

#### 📖 Giới thiệu kiến thức

Extension **`mysqli`** đặc biệt ở chỗ cung cấp **hai kiểu giao diện (API style)** cho **cùng một chức năng** —
ta chọn kiểu nào tùy sở thích, **kết quả và hiệu năng như nhau** (chỉ khác cú pháp). Đây là điểm riêng của
`mysqli`: **PDO chỉ OOP**, còn **`pg_connect` chỉ thủ tục** (xem mục 2.1).

**1. Hướng thủ tục (procedural).**
- Dùng các **hàm** `mysqli_*`, luôn **truyền connection (hoặc result) làm tham số đầu tiên**.
- Cú pháp gần với extension cũ `mysql_*` → **dễ chuyển code legacy** sang.

**2. Hướng đối tượng (OOP).**
- Tạo **object** `new mysqli(...)` rồi gọi **method** trên nó (`$conn->query()`), đọc **property**
  (`$conn->error`). Connection là **ngầm định** (nằm trong object), không phải truyền lại mỗi lần.
- Gọn hơn, hiện đại, dễ đọc, hợp phong cách OOP.

**Bảng ánh xạ thủ tục ↔ OOP (cùng chức năng):**

| Thủ tục (procedural) | Hướng đối tượng (OOP) |
|----------------------|-----------------------|
| `mysqli_connect($h,$u,$p,$db)` | `new mysqli($h,$u,$p,$db)` |
| `mysqli_connect_error()` | `$conn->connect_error` (property) |
| `mysqli_query($conn, $sql)` | `$conn->query($sql)` |
| `mysqli_prepare($conn, $sql)` | `$conn->prepare($sql)` |
| `mysqli_stmt_bind_param($stmt, ...)` | `$stmt->bind_param(...)` |
| `mysqli_stmt_execute($stmt)` | `$stmt->execute()` |
| `mysqli_fetch_assoc($res)` | `$res->fetch_assoc()` |
| `mysqli_num_rows($res)` | `$res->num_rows` (property) |
| `mysqli_error($conn)` | `$conn->error` (property) |
| `mysqli_close($conn)` | `$conn->close()` |

**Ví dụ — cùng một truy vấn, hai kiểu:**

```php
// ===== Thủ tục (procedural): truyền $conn vào từng hàm =====
$conn = mysqli_connect('127.0.0.1', 'root', 'secret', 'mydb', 3306);
if (mysqli_connect_errno()) {
    die('Kết nối lỗi: ' . mysqli_connect_error());
}
$stmt = mysqli_prepare($conn, 'SELECT name FROM users WHERE id = ?');
mysqli_stmt_bind_param($stmt, 'i', $id);       // 'i' = integer
mysqli_stmt_execute($stmt);
$res = mysqli_stmt_get_result($stmt);
while ($row = mysqli_fetch_assoc($res)) {
    echo $row['name'];
}
mysqli_close($conn);
```

```php
// ===== Hướng đối tượng (OOP): gọi method trên object $conn/$stmt/$res =====
$conn = new mysqli('127.0.0.1', 'root', 'secret', 'mydb', 3306);
if ($conn->connect_errno) {
    die('Kết nối lỗi: ' . $conn->connect_error);
}
$stmt = $conn->prepare('SELECT name FROM users WHERE id = ?');
$stmt->bind_param('i', $id);
$stmt->execute();
$res = $stmt->get_result();
while ($row = $res->fetch_assoc()) {
    echo $row['name'];
}
$conn->close();
```

> **Bản chất giống nhau:** cả hai gọi **cùng một extension** bên dưới — **không** khác biệt hiệu năng hay tính
> năng. Hàm thủ tục thực chất chỉ là "vỏ bọc" quanh cùng logic mà object dùng. Khác biệt duy nhất là **cú pháp**
> và việc procedural phải **truyền `$conn`** ở mỗi hàm, còn OOP giữ connection **trong object**.

**Nên dùng kiểu nào?**
- **OOP** được **khuyến nghị** cho code mới: sạch, dễ đọc, nhất quán với phần còn lại của ứng dụng OOP, và giống
  cách dùng PDO (dễ chuyển qua lại).
- **Procedural** hợp khi **bảo trì code cũ** viết theo `mysql_*`/thủ tục, hoặc script nhỏ đơn giản.
- Dù chọn kiểu nào, **luôn dùng prepared statement** (`?` + `bind_param`) để chống SQL injection.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`mysqli` cung cấp cả 2 kiểu API cho cùng chức năng** — **thủ tục** (hàm `mysqli_*`, truyền `$conn` làm tham
  số đầu) và **OOP** (`new mysqli()` rồi gọi method/property). Đây là điểm riêng của mysqli (PDO chỉ OOP,
  pg_connect chỉ thủ tục).
- **Khác biệt chỉ là cú pháp** — **cùng extension bên dưới, không khác hiệu năng/tính năng**. Procedural phải
  truyền connection mỗi lần; OOP giữ connection trong object (ngầm định), gọn & hiện đại hơn.
- **Ánh xạ:** `mysqli_query($conn,$sql)` ↔ `$conn->query($sql)`; `mysqli_error($conn)` ↔ `$conn->error`;
  `mysqli_fetch_assoc($res)` ↔ `$res->fetch_assoc()`…
- **Khuyến nghị:** ưu tiên **OOP** cho code mới (giống PDO, dễ đọc); procedural khi bảo trì code legacy. Kiểu
  nào cũng **phải dùng prepared statement** chống SQL injection.

[↑ Mục lục](#mục-lục)

<a id="q2-3"></a>
### 2.3. Connection pooling là gì và vì sao cần?

#### 📖 Giới thiệu kiến thức

Mở một kết nối DB mới **tốn kém**: bắt tay TCP, xác thực user/password, (có thể) bắt tay TLS, cấp phát tài
nguyên phía server. Nếu mỗi request web đều **mở rồi đóng** một kết nối, chi phí này lặp lại liên tục và số
kết nối đồng thời có thể **vượt giới hạn** của DB (`max_connections`) → chậm hoặc từ chối kết nối.

**Connection pool (bể kết nối)** giải quyết bằng cách **tạo sẵn và tái sử dụng** một tập kết nối:

1. Khi khởi động, pool mở sẵn N kết nối tới DB và giữ mở.
2. Khi cần truy vấn, request **mượn** một kết nối rảnh từ pool (không phải mở mới).
3. Dùng xong **trả** kết nối lại pool (không đóng thật) để request sau tái dùng.
4. Nếu hết kết nối rảnh, request **chờ** cho tới khi có cái được trả (hoặc timeout).

**Tham số quan trọng của pool:** `min`/`max` connections, `idleTimeout` (đóng kết nối rảnh quá lâu),
`connectionTimeout` (chờ tối đa để mượn), `maxLifetime` (tuổi thọ tối đa 1 kết nối).

**Pool ở hai tầng:**

- **Tầng ứng dụng (in-app pool):** thư viện tự quản pool — Node `pg.Pool`/`mysql2.createPool`, Java
  **HikariCP**, Laravel giữ kết nối theo vòng đời request. Mỗi tiến trình app có pool riêng.
- **Tầng trung gian (external pooler):** một service đứng giữa app và DB gom kết nối cho **nhiều app** —
  **PgBouncer** (PostgreSQL), **ProxySQL** (MySQL). Rất quan trọng khi có nhiều instance app (serverless,
  nhiều container) dễ làm bùng nổ số kết nối tới DB.

```javascript
// Node.js — pool cho MySQL (mysql2)
const mysql = require('mysql2/promise');
const pool = mysql.createPool({
  host: '127.0.0.1', user: 'root', password: 'secret', database: 'mydb',
  connectionLimit: 10,        // tối đa 10 kết nối tái dùng
  queueLimit: 0,              // 0 = không giới hạn hàng chờ
});
const [rows] = await pool.query('SELECT * FROM users WHERE id = ?', [1]); // tự mượn/trả
```

> **Lưu ý serverless (Lambda/Vercel):** mỗi instance mở pool riêng → tổng kết nối dễ vượt `max_connections`.
> Giải pháp: dùng **external pooler** (PgBouncer/RDS Proxy) hoặc giới hạn pool size nhỏ mỗi instance.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Mở kết nối DB **tốn kém** (TCP + auth + TLS); mở/đóng mỗi request gây chậm và dễ vượt `max_connections`.
- **Connection pool** = tạo sẵn & **tái sử dụng** một tập kết nối: request **mượn** kết nối rảnh → dùng →
  **trả** lại pool (không đóng thật); hết thì **chờ**.
- **Tham số:** `min`/`max`, `idleTimeout`, `connectionTimeout`, `maxLifetime`.
- **Hai tầng:** **in-app pool** (`pg.Pool`, HikariCP, mysql2 pool) và **external pooler** (**PgBouncer** cho
  PostgreSQL, **ProxySQL** cho MySQL) — pooler ngoài rất cần khi nhiều app instance / **serverless**.
- **Lợi ích:** giảm latency, kiểm soát số kết nối, tăng throughput & ổn định dưới tải.

[↑ Mục lục](#mục-lục)

<a id="q2-4"></a>
### 2.4. 1000 user truy cập website thì cần mở bao nhiêu connection pool?

#### 📖 Giới thiệu kiến thức

Đây là **câu hỏi bẫy kinh điển**. Trực giác sai lầm phổ biến: *"1000 user → mở 1000 connection"*. **Sai
hoàn toàn.** Số connection trong pool **không** tỉ lệ với số user, mà tỉ lệ với **số truy vấn ĐANG CHẠY đồng
thời (concurrent active queries)** tại một thời điểm.

**Vì sao không phải 1000?**

1. **User không truy vấn DB liên tục.** Một request web chỉ **giữ connection vài mili-giây** — đúng lúc chạy
   query — rồi **trả lại pool** ngay. Phần lớn thời gian user đang **đọc trang, gõ, suy nghĩ** → *không chạm
   DB*. Tại một khoảnh khắc, chỉ một **phần rất nhỏ** trong 1000 user thực sự có query đang chạy.
2. **DB chỉ chạy song song thật ~N query** với N = số CPU core. Mở 1000 connection không làm DB nhanh hơn —
   chúng chỉ **xếp hàng**, tranh CPU (**context switching**), tranh lock → **throughput GIẢM**, latency tăng.
3. **Mỗi connection tốn tài nguyên.** PostgreSQL mỗi connection là **một process** + `work_mem` (vài MB);
   1000 connection có thể ngốn **nhiều GB RAM** phía DB. Ngoài ra có trần `max_connections` (MySQL mặc định
   **151**, PostgreSQL **100**).

**Tính đúng bằng Little's Law:**

> **Số connection cần ≈ throughput (query/giây) × thời gian mỗi query giữ connection (giây).**

Ví dụ với 1000 user:
- Giả sử mỗi user thao tác **1 request mỗi 5 giây** → throughput ≈ **200 request/giây**.
- Mỗi request chạy query mất ≈ **10 ms** (0,01 s) rồi trả connection.
- → Connection thực sự **bận cùng lúc** ≈ `200 × 0,01` = **2**.
- Cộng biên an toàn cho lúc cao điểm → một pool **~10 connection là dư sức** phục vụ 1000 user.

**Công thức tham khảo của HikariCP (cho DB server):**

```
connections = (số CPU core × 2) + số đĩa (effective spindles)
```

Server 4 core → khoảng **~10 connection**. HikariCP nổi tiếng với thí nghiệm: pool **nhỏ (10)** cho throughput
**cao hơn** pool lớn (100+) — vì pool quá lớn gây quá tải context switch/lock ở DB. **"Nhỏ mà nhanh."**

**Khi nhiều app instance (rất hay gặp):**

- Tổng connection tới DB = **Σ pool của mọi instance**. Vd 10 instance × pool 20 = **200 connection** → phải
  **≤ `max_connections`** của DB.
- Với **serverless / hàng trăm instance** (Lambda, Vercel) → dễ bùng nổ connection → dùng **external pooler**
  (**PgBouncer** cho PostgreSQL, **ProxySQL** cho MySQL) để **multiplex** hàng nghìn client xuống vài chục
  connection thật tới DB (xem mục 2.3).

**Cách làm đúng trong thực tế:**

1. **Bắt đầu nhỏ** (10–20 connection/instance), **không** đoán bằng số user.
2. **Đo** dưới tải: thời gian chờ mượn connection (pool wait), số active connection, CPU của DB.
3. Nếu thường xuyên **chờ pool** → tăng dần; nếu DB CPU đã cao → **đừng tăng pool**, mà **tối ưu query**
   (index, giảm thời gian giữ connection).
4. Nhớ: query **chậm** thì mở thêm connection **không cứu được** — chỉ dồn tải; phải **làm query nhanh hơn**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **KHÔNG phải 1000.** Số connection pool tỉ lệ với **số query đồng thời đang chạy**, **không** phải số user.
- Một request chỉ **giữ connection vài ms** rồi trả lại → **một pool nhỏ phục vụ được hàng nghìn user**.
- **Little's Law:** `connections ≈ throughput(req/s) × holding_time(s)`. Vd 200 req/s × 10 ms = **~2**; cộng
  biên an toàn → pool **~10** là đủ cho 1000 user.
- **Pool quá lớn phản tác dụng:** tốn RAM (PG mỗi conn = 1 process), **context switch**, tranh lock, vượt
  `max_connections` → **chậm hơn**. DB N core chỉ chạy song song ~N query. HikariCP: `(core×2)+đĩa` ≈ vài chục.
- **Nhiều instance:** tổng pool mọi instance phải **≤ `max_connections`**; serverless/nhiều instance →
  **PgBouncer/ProxySQL** để multiplex. Query chậm thì **tối ưu query**, đừng tăng pool.

[↑ Mục lục](#mục-lục)

<a id="q2-5"></a>
### 2.5. max_connections mặc định của MySQL và PostgreSQL là bao nhiêu?

#### 📖 Giới thiệu kiến thức

**`max_connections`** là số **kết nối đồng thời tối đa** mà DB server chấp nhận. Vượt quá → client mới bị **từ
chối** (không phải xếp hàng). Đây là trần cứng liên quan trực tiếp tới việc tính pool (mục 2.4).

**Giá trị mặc định:**

| Hệ | `max_connections` mặc định | Ghi chú |
|----|----------------------------|---------|
| **MySQL** | **151** | Từ MySQL 5.5+ (trước đó là 100). Có **+1** kết nối dự phòng cho user quyền `SUPER`/`CONNECTION_ADMIN` để admin vẫn vào được khi đầy. |
| **PostgreSQL** | **100** | Trong đó `superuser_reserved_connections` (mặc định **3**) dành riêng cho superuser → user thường dùng được **~97**. |

**Kiểm tra giá trị hiện tại:**

```sql
-- MySQL
SHOW VARIABLES LIKE 'max_connections';
SHOW STATUS LIKE 'Threads_connected';   -- số kết nối đang mở
SHOW STATUS LIKE 'Max_used_connections'; -- đỉnh đã từng dùng

-- PostgreSQL
SHOW max_connections;
SELECT count(*) FROM pg_stat_activity;  -- số kết nối đang mở
```

**Thay đổi giá trị:**

```sql
-- MySQL: đổi runtime (áp dụng ngay, MẤT khi restart)
SET GLOBAL max_connections = 500;
-- Vĩnh viễn: sửa my.cnf →  [mysqld]  max_connections = 500  → restart
```

```conf
# PostgreSQL: sửa postgresql.conf
max_connections = 200
# → BẮT BUỘC RESTART (không reload được, vì phải cấp phát lại shared memory)
```

> **Khác biệt quan trọng:** MySQL đổi `max_connections` **nóng** (runtime) được; PostgreSQL **phải restart**
> vì nó cấp phát vùng nhớ chia sẻ (shared memory) theo số connection ngay lúc khởi động.

**Lỗi khi vượt trần (rất hay hỏi):**
- **MySQL:** `ERROR 1040 (HY000): Too many connections`.
- **PostgreSQL:** `FATAL: sorry, too many clients already`.

**Có nên cứ tăng đại `max_connections`?** — **Không.** Mỗi connection tốn RAM (PostgreSQL mỗi connection là
**một process** + `work_mem` vài MB → 500 connection có thể ngốn hàng GB). Tăng trần chỉ **hoãn** vấn đề; cách
đúng là **dùng connection pool / external pooler** (PgBouncer, ProxySQL) để **ít connection thật** mà phục vụ
nhiều client (xem mục 2.3–2.4). Trần cao chỉ để **an toàn**, không phải để lấp đầy.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **MySQL:** mặc định **151** (+1 dự phòng cho superuser). **PostgreSQL:** mặc định **100** (trừ 3 reserved cho
  superuser → user thường ~97).
- **Kiểm tra:** MySQL `SHOW VARIABLES LIKE 'max_connections'`; PostgreSQL `SHOW max_connections`.
- **Đổi:** MySQL `SET GLOBAL max_connections=…` (nóng, mất khi restart) hoặc my.cnf; PostgreSQL sửa
  `postgresql.conf` rồi **phải restart** (cấp phát shared memory).
- **Vượt trần:** MySQL lỗi **1040 "Too many connections"**; PostgreSQL **"too many clients already"**.
- **Đừng tăng đại** — mỗi connection tốn RAM (PG = 1 process/conn); nên dùng **connection pool / PgBouncer**
  thay vì nống trần (mục 2.3–2.4).

[↑ Mục lục](#mục-lục)

<a id="q2-6"></a>
### 2.6. Cách tăng max_connections an toàn khi bị lỗi 1040 (Too many connections)

#### 📖 Giới thiệu kiến thức

Lỗi **`ERROR 1040 (HY000): Too many connections`** nghĩa là số kết nối đang mở đã chạm trần `max_connections`
(mặc định 151 — xem mục 2.5). Phản xạ sai là **nống trần thật cao ngay**; cách an toàn phải **chẩn đoán trước,
tăng có tính toán, rồi sửa gốc**.

**Bước 1 — Vào được DB dù đang đầy (đừng hoảng).**
MySQL **giữ sẵn 1 slot dự phòng** cho user có quyền `SUPER`/`CONNECTION_ADMIN`. Nên admin **vẫn đăng nhập
được** khi user thường bị chặn → dùng slot này để xử lý:

```sql
mysql -u root -p        -- vào bằng account có SUPER (dùng slot dự phòng)
```

**Bước 2 — Chẩn đoán: vì sao hết connection?** (Rất quan trọng — thường KHÔNG phải do trần thấp.)

```sql
SHOW STATUS LIKE 'Threads_connected';    -- đang mở bao nhiêu
SHOW STATUS LIKE 'Max_used_connections'; -- đỉnh đã dùng (so với max_connections)
SHOW PROCESSLIST;                        -- xem từng kết nối đang làm gì
SHOW VARIABLES LIKE 'wait_timeout';      -- idle bao lâu thì tự đóng
```

- **Nhiều dòng `Sleep`** trong `PROCESSLIST` → connection **rò rỉ** (app không đóng) hoặc **thiếu pooling** /
  `wait_timeout` quá dài → đây mới là **nguyên nhân thật**, không phải trần thấp.
- Nhiều query **chạy lâu** (state `Sending data`, `Locked`) → query chậm **giữ connection lâu** → cạn slot.

**Bước 3 — Giải toả tức thời.**

```sql
-- (a) Giết các connection treo/sleeping để lấy lại slot ngay
KILL <process_id>;                       -- id lấy từ SHOW PROCESSLIST

-- (b) Tăng trần NÓNG (hiệu lực ngay, KHÔNG cần restart — nhưng mất khi restart)
SET GLOBAL max_connections = 300;
```

**Bước 4 — Tăng có tính toán, đừng đặt bừa.** Mỗi connection tốn RAM. Ước lượng thô:

> **RAM ≈ bộ nhớ global + (max_connections × bộ nhớ mỗi connection).**

- Bộ nhớ **mỗi connection** = tổng các buffer per-thread: `sort_buffer_size`, `join_buffer_size`,
  `read_buffer_size`, `thread_stack`, `binlog_cache_size`… (thường vài MB). Đảm bảo RAM server chịu được,
  **không** thì server **swap/OOM** còn tệ hơn lỗi 1040.
- Kiểm tra thêm giới hạn OS: **`open_files_limit`/`ulimit -n`** (mỗi connection tốn file descriptor) — tăng
  `max_connections` mà không tăng FD limit sẽ lỗi mở kết nối.

**Bước 5 — Ghi vĩnh viễn** (để không mất khi restart):

```conf
# my.cnf  →  [mysqld]
max_connections = 300
open_files_limit = 4096        # kèm theo nếu cần
```
Sau đó restart, hoặc giữ đồng bộ giá trị runtime (Bước 3b) với file config.

**Bước 6 — Sửa GỐC (quan trọng nhất, nếu không sẽ tái phát):**

1. **Bật/điều chỉnh connection pool** ở tầng app — giới hạn & tái dùng connection (mục 2.3–2.4). Đây thường là
   gốc rễ: app mở quá nhiều kết nối ngắn hoặc rò rỉ.
2. **Giảm `wait_timeout`** để idle connection tự đóng nhanh, trả slot:
   ```sql
   SET GLOBAL wait_timeout = 120;   -- vd 120s thay vì mặc định 28800s (8 giờ)
   ```
3. **Tối ưu query chậm** (index) để không giữ connection lâu.
4. Nhiều app instance / tải lớn → đặt **ProxySQL** phía trước để **multiplex** (giảm connection thật tới DB).

> **Nguyên tắc:** tăng `max_connections` chỉ là **giải pháp tạm** để hệ thống sống lại; **pooling + đóng
> connection đúng + query nhanh** mới là cách bền vững. Nống trần vô tội vạ chỉ đổi lỗi 1040 lấy nguy cơ **hết
> RAM**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Lỗi **1040 "Too many connections"** = chạm trần `max_connections`. **Đừng nống trần thật cao ngay** — chẩn
  đoán trước.
- **Vào bằng account `SUPER`** (MySQL chừa 1 slot dự phòng) → `SHOW PROCESSLIST`/`Threads_connected` xem nguyên
  nhân: nhiều **`Sleep`** = rò rỉ/thiếu pooling; query chậm = giữ connection lâu.
- **Giải toả ngay:** `KILL` connection treo; tăng nóng `SET GLOBAL max_connections = N;` (không cần restart,
  mất khi restart).
- **Tăng an toàn:** tính RAM `≈ global + (max_connections × buffer mỗi conn)`; kiểm tra **`open_files_limit`**;
  ghi vào **my.cnf** cho vĩnh viễn.
- **Sửa gốc:** bật **connection pool**, **giảm `wait_timeout`**, **tối ưu query chậm**, dùng **ProxySQL**
  multiplex. Nống trần chỉ là tạm thời, dễ đổi lỗi 1040 lấy **hết RAM**.

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. NoSQL là gì? So sánh NoSQL với SQL

### 📖 Giới thiệu kiến thức

**NoSQL** ("Not Only SQL" — không chỉ SQL) là nhóm các hệ CSDL **không theo mô hình quan hệ (bảng)** truyền
thống. Thay vì lưu dữ liệu thành **bảng với hàng/cột và schema cố định**, NoSQL lưu dữ liệu theo các mô hình
linh hoạt hơn: **document, key-value, column-family, graph**. Ra đời để giải quyết bài toán của web hiện đại:
**dữ liệu cực lớn, cấu trúc thay đổi liên tục, cần scale ngang (horizontal scaling) trên nhiều server**.

**Đặc điểm chính:**
- **Schema linh hoạt (schema-less):** mỗi bản ghi có thể có cấu trúc khác nhau, thêm/bớt field dễ dàng.
- **Scale ngang (horizontal scaling):** thêm nhiều server rẻ (sharding) thay vì nâng cấp 1 server mạnh
  (vertical scaling như SQL truyền thống). Chi tiết về scale ngang/dọc xem `Backend/devops.md` câu 2.
- **Ưu tiên hiệu năng & tính sẵn sàng (availability)** hơn tính nhất quán tuyệt đối — nhiều hệ NoSQL theo
  mô hình **BASE** (Basically Available, Soft state, Eventual consistency) thay vì **ACID** chặt như SQL.

**Các CSDL NoSQL phổ biến:** MongoDB, Cassandra, Redis, DynamoDB (AWS), Couchbase, Neo4j, Elasticsearch,
HBase, Firebase/Firestore.

**So sánh NoSQL vs SQL:**

| Tiêu chí | SQL (quan hệ) | NoSQL (phi quan hệ) |
|----------|---------------|---------------------|
| Mô hình dữ liệu | Bảng (hàng/cột) | Document / key-value / column / graph |
| Schema | **Cố định**, định nghĩa trước | **Linh hoạt**, có thể thay đổi |
| Ngôn ngữ | SQL chuẩn | API/query riêng theo từng hệ |
| Scale | Chủ yếu **dọc** (vertical) | Chủ yếu **ngang** (horizontal, sharding) |
| Tính nhất quán | **ACID** (mạnh) | Thường **BASE** / eventual consistency |
| Quan hệ / JOIN | Mạnh, hỗ trợ JOIN | Yếu, thường **không JOIN** (nhúng/denormalize) |
| Dữ liệu phù hợp | Có cấu trúc rõ, quan hệ chặt | Bán/không cấu trúc, thay đổi nhanh, khối lượng lớn |
| Ví dụ | MySQL, PostgreSQL, SQL Server | MongoDB, Redis, Cassandra, Neo4j |

> **Không phải "cái nào tốt hơn"** mà là **phù hợp bài toán nào**. SQL mạnh khi dữ liệu có cấu trúc và cần
> toàn vẹn (giao dịch, tài chính); NoSQL mạnh khi cần **linh hoạt, scale lớn, tốc độ cao** (real-time, big data).

### 🎯 Tóm tắt trả lời phỏng vấn

- **NoSQL** = "Not Only SQL", nhóm CSDL **phi quan hệ**, lưu dữ liệu theo document/key-value/column/graph
  thay vì bảng cố định. Sinh ra cho **dữ liệu lớn, linh hoạt, scale ngang**.
- Đặc điểm: **schema linh hoạt**, **scale ngang (sharding)**, thường theo **BASE / eventual consistency**
  (ưu tiên availability & tốc độ hơn ACID chặt).
- **CSDL tiêu biểu:** MongoDB, Redis, Cassandra, DynamoDB, Neo4j, Elasticsearch.
- **SQL vs NoSQL:** SQL = schema cố định, ACID, JOIN mạnh, scale dọc → hợp dữ liệu có cấu trúc/toàn vẹn cao.
  NoSQL = schema linh hoạt, scale ngang, tốc độ cao → hợp dữ liệu lớn/thay đổi nhanh. **Chọn theo bài toán,
  không có cái nào "tốt hơn" tuyệt đối.**

[↑ Mục lục](#mục-lục)

<a id="q3-1"></a>
### 3.1. Các loại NoSQL và cách chọn loại phù hợp

#### 📖 Giới thiệu kiến thức

Có **4 loại NoSQL chính**, khác nhau ở cách tổ chức dữ liệu:

**1. Key-Value (khóa–giá trị).**
- Lưu dữ liệu dạng cặp `key → value` đơn giản như một từ điển khổng lồ. Truy xuất bằng key **cực nhanh**.
- **Ví dụ:** **Redis**, DynamoDB, Memcached, Riak.
- **Dùng khi:** cache, session, đếm lượt, bảng xếp hạng real-time, dữ liệu tra cứu nhanh theo khóa.

**2. Document (tài liệu).**
- Lưu dữ liệu thành **document** dạng JSON/BSON, mỗi document tự chứa cấu trúc riêng (nested, mảng...).
- **Ví dụ:** **MongoDB**, CouchDB, Firestore.
- **Dùng khi:** dữ liệu bán cấu trúc, thay đổi thường xuyên — hồ sơ người dùng, sản phẩm, CMS, catalog.

**3. Column-Family (cột rộng / wide-column).**
- Lưu theo **họ cột (column family)**, tối ưu ghi & đọc khối lượng cực lớn, phân tán nhiều node.
- **Ví dụ:** **Cassandra**, HBase, ScyllaDB, Google Bigtable.
- **Dùng khi:** big data, time-series, log, IoT, ghi rất nhiều & scale khổng lồ (write-heavy).

**4. Graph (đồ thị).**
- Lưu dữ liệu thành **node** (đỉnh) và **edge** (quan hệ), tối ưu cho việc **duyệt quan hệ phức tạp**.
- **Ví dụ:** **Neo4j**, Amazon Neptune, ArangoDB.
- **Dùng khi:** mạng xã hội, hệ thống gợi ý (recommendation), phát hiện gian lận, quan hệ nhiều tầng.

**Dựa vào gì để chọn loại NoSQL?** — trả lời theo **mô hình truy cập dữ liệu (data access pattern)**:

| Nhu cầu / bài toán | Loại NoSQL nên chọn | Ví dụ CSDL |
|--------------------|---------------------|------------|
| Tra cứu siêu nhanh theo khóa, cache, session | **Key-Value** | Redis, DynamoDB |
| Dữ liệu bán cấu trúc, linh hoạt, lồng nhau | **Document** | MongoDB |
| Ghi cực nhiều, big data, time-series/log | **Column-Family** | Cassandra, HBase |
| Quan hệ phức tạp, mạng lưới, gợi ý | **Graph** | Neo4j |

**Các yếu tố cân nhắc khi quyết định:**
- **Cấu trúc dữ liệu:** đơn giản theo khóa? lồng nhau? nhiều quan hệ? → chọn loại tương ứng.
- **Data access pattern:** đọc nhiều hay ghi nhiều, truy vấn theo khóa hay duyệt quan hệ.
- **Yêu cầu scale & khối lượng:** cần phân tán khổng lồ → column-family; vừa phải → document.
- **Tính nhất quán:** cần mạnh (gần ACID) hay chấp nhận eventual consistency.
- **Độ trễ (latency):** cần real-time (µs–ms) → key-value in-memory (Redis).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **4 loại NoSQL:** **Key-Value** (Redis — cache/session), **Document** (MongoDB — dữ liệu linh hoạt/JSON),
  **Column-Family** (Cassandra — big data/write-heavy), **Graph** (Neo4j — quan hệ phức tạp).
- **Chọn loại nào dựa vào data access pattern & cấu trúc dữ liệu:** tra cứu theo khóa → Key-Value; document
  linh hoạt → Document; ghi khổng lồ/time-series → Column-Family; nhiều quan hệ/mạng lưới → Graph.
- Cân nhắc thêm: **yêu cầu scale, tính nhất quán (ACID vs eventual), độ trễ real-time**.

[↑ Mục lục](#mục-lục)

<a id="q3-2"></a>
### 3.2. Vì sao NoSQL "scale ngang" còn SQL "scale dọc"?

#### 📖 Giới thiệu kiến thức

Đây là **lý do lịch sử** NoSQL ra đời và là khác biệt cốt lõi giữa hai họ CSDL. Trước hết nhắc lại: **scale dọc
(vertical)** = nâng cấp **một máy** mạnh hơn; **scale ngang (horizontal)** = thêm **nhiều máy** song song (chi
tiết xem `Backend/devops.md` câu 2). Câu hỏi ở đây là **vì sao mỗi họ CSDL lại thiên về một hướng**.

**1. SQL quan hệ → khó scale ngang, nên thường scale dọc.**

SQL có hai đặc trưng khiến việc tách dữ liệu ra nhiều máy (gọi là **sharding**) rất khó:

- **JOIN chéo bảng:** một query thường cần dữ liệu từ **nhiều bảng**. Nếu các bảng/phần dữ liệu ấy nằm ở
  **nhiều máy khác nhau**, mỗi query phải **kết nối qua mạng** giữa các máy để ghép → chậm và phức tạp.
- **Transaction ACID chặt:** đảm bảo toàn vẹn trên **dữ liệu trải ra nhiều máy** (distributed transaction)
  cần cơ chế đồng bộ/chốt khóa tốn kém (2-phase commit...) → chậm, khó.

Vì sharding khó, hệ SQL thường chọn cách **"làm một máy thật to"** (scale dọc — thêm CPU/RAM/SSD). Nhưng máy
to **có trần** phần cứng và **giá đắt phi tuyến tính** (máy gấp đôi sức mạnh có thể đắt gấp 3–4 lần).

**2. NoSQL → sinh ra để scale ngang.**

NoSQL thiết kế **"bớt tính năng, đổi lấy dễ mở rộng"**:

- **Không (hoặc rất yếu) JOIN:** dữ liệu **tự chứa** (denormalize) — nhúng sẵn dữ liệu liên quan vào cùng một
  document/key (vd thông tin user và đơn hàng nằm chung trong 1 document) thay vì chia bảng rồi JOIN.
- **Schema linh hoạt + eventual consistency:** không bắt buộc đồng bộ gắt gao giữa các máy.
- **Gom theo key/hash:** dữ liệu dễ được **tách (shard) ra nhiều máy** sao cho mỗi query thường **chỉ chạm
  1 máy** → thêm server rẻ là tăng sức chứa ngay, **gần như không giới hạn**.

**So sánh trực tiếp:**

| Tiêu chí | SQL (quan hệ) | NoSQL |
|----------|---------------|-------|
| JOIN chéo bảng | **Có** → khó tách máy | **Không/yếu** (denormalize) → dễ tách |
| Transaction | **ACID chặt** → khó phân tán | Thường **eventual** → dễ phân tán |
| Hướng scale thiên về | **Dọc** (nâng cấp 1 máy) | **Ngang** (thêm nhiều máy, sharding) |
| Giới hạn | Trần phần cứng 1 máy | Gần như theo ngân sách |

> **Bản chất là một sự đánh đổi:** SQL giữ tính năng phong phú (JOIN, ACID, toàn vẹn) nhưng **khó mở rộng**;
> NoSQL **bớt** JOIN/ACID để đổi lấy **dễ scale ngang**. Không ai "tốt hơn" — chọn theo bài toán: dữ liệu có
> cấu trúc, cần toàn vẹn (tài chính, giao dịch) → SQL; dữ liệu lớn, thay đổi nhanh, cần mở rộng thoải mái → NoSQL.

**Hệ quả thực tế:** mở rộng **app server** theo chiều ngang thì dễ, nhưng **CSDL quan hệ mới là điểm nghẽn
thật** (sharding khó). Vì vậy nhiều hệ thống dùng **SQL cho dữ liệu quan trọng** + **NoSQL/Redis/cache** cho
phần cần mở rộng nhanh — kết hợp điểm mạnh của cả hai.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Scale dọc** = nâng cấp 1 máy mạnh hơn; **scale ngang** = thêm nhiều máy song song (xem `Backend/devops.md`
  câu 2).
- **SQL thường scale dọc** vì có **JOIN chéo bảng + transaction ACID** → tách dữ liệu ra nhiều máy (sharding)
  rất khó → đành nâng cấp 1 máy to (nhưng có trần, đắt).
- **NoSQL sinh ra để scale ngang** vì thiết kế **không JOIN** (dữ liệu tự chứa — denormalize), schema linh
  hoạt, thường eventual consistency → **dễ sharding**, thêm máy là tăng sức chứa, gần như không giới hạn.
- **Bản chất là đánh đổi:** SQL giữ JOIN/ACID đổi lấy khó mở rộng; NoSQL bớt JOIN/ACID đổi lấy dễ scale ngang.
- **Thực tế:** SQL cho dữ liệu cần toàn vẹn (tài chính), NoSQL/Redis cho phần cần mở rộng nhanh — hay dùng
  **kết hợp** vì CSDL quan hệ mới là điểm nghẽn thật khi scale.

[↑ Mục lục](#mục-lục)

<a id="q3-3"></a>
### 3.3. MongoDB là gì?

#### 📖 Giới thiệu kiến thức

**MongoDB** là hệ CSDL **NoSQL dạng Document** phổ biến nhất, mã nguồn mở, do **MongoDB Inc.** phát triển. Thay
vì lưu dữ liệu thành bảng/hàng/cột như SQL quan hệ, MongoDB lưu mỗi bản ghi dưới dạng một **document** có cấu
trúc linh hoạt giống **JSON** (thực tế là **BSON** — Binary JSON). Đây là đại diện tiêu biểu cho nhóm **Document
NoSQL** đã nhắc ở mục 3.1.

**Đặc điểm chính:**

- **Document model:** mỗi bản ghi là một "document" cấu trúc giống JSON, **có thể lồng nhau** (nested) và chứa
  **mảng**. Một document thường **tự chứa** toàn bộ dữ liệu liên quan (denormalize) thay vì chia nhiều bảng rồi JOIN.
- **BSON:** lưu document ở dạng **BSON (Binary JSON)** — nhị phân, hiệu quả và nhỏ hơn JSON thuần, hỗ trợ thêm
  nhiều kiểu dữ liệu (Date, ObjectId, Binary, Decimal128…).
- **Schema-less (linh hoạt):** các document trong cùng collection **không bắt buộc** giống cấu trúc — thêm/bớt
  field thoải mái (có thể bật **schema validation** nếu muốn ràng buộc).
- **No JOIN (gần như):** hỗ trợ `$lookup` (giống JOIN) nhưng dùng ít — thiết kế thiên về **denormalize** để đọc nhanh.
- **Scale ngang native:** hỗ trợ **sharding** sẵn → chia dữ liệu ra nhiều máy dễ (xem mục 3.2).
- **High availability:** **Replica Set** (1 primary + nhiều secondary) tự động **failover** khi primary chết.
- **Aggregation pipeline:** framework mạnh để xử lý/phân tích (lọc, gom nhóm, tính toán, sắp xếp) — mạnh hơn
  `GROUP BY` của SQL.
- **Index:** hỗ trợ nhiều loại index (B-tree, geospatial, full-text, TTL…).

**Ánh xạ thuật ngữ SQL ↔ MongoDB:**

| SQL (quan hệ) | MongoDB |
|---------------|---------|
| Database | Database |
| Table | **Collection** |
| Row | **Document** |
| Column | **Field** |
| Primary key | `_id` (mặc định là `ObjectId`) |
| JOIN | `$lookup` (ít dùng) |
| `SELECT … WHERE` | `db.collection.find({ … })` |

**Ví dụ một document** — toàn bộ thông tin user (kèm địa chỉ và đơn hàng) gói trong 1 document:

```
{
  "_id": ObjectId("65a1..."),
  "name": "Nguyen Van A",
  "email": "a@example.com",
  "age": 30,
  "address": { "city": "Hanoi", "zip": "100000" },   // object lồng
  "tags": ["vip", "active"],                          // mảng
  "orders": [                                         // mảng object lồng — tự chứa
    { "id": 1, "total": 500 },
    { "id": 2, "total": 1200 }
  ]
}
```

> Trong SQL, dữ liệu này cần **3 bảng** (`users`, `addresses`, `orders`) rồi **JOIN** để lấy ra. Trong MongoDB,
> nó nằm gọn trong **1 document** → đọc nhanh, không cần JOIN.

**Truy vấn cơ bản:**

```javascript
db.users.insertOne({ name: "An", age: 25 });                 // thêm
db.users.find({ age: { $gte: 18 } });                        // age >= 18  (≈ SELECT WHERE)
db.users.find({ "address.city": "Hanoi" });                  // query field lồng bằng dấu chấm
db.users.updateOne({ name: "An" }, { $set: { age: 26 } });   // sửa

// Aggregation pipeline (≈ GROUP BY + sắp xếp)
db.orders.aggregate([
  { $group: { _id: "$user_id", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } }
]);
```

**Khi nào dùng MongoDB?**

- Dữ liệu **bán cấu trúc / linh hoạt**, cấu trúc thay đổi liên tục (CMS, catalog, user profile, content).
- Cần **nhúng dữ liệu** (nested), **ít JOIN**, ưu tiên đọc nhanh.
- Cần **scale ngang** với khối lượng dữ liệu lớn.
- Prototyping nhanh nhờ schema linh hoạt.

**Khi nào KHÔNG nên?**

- Cần **transaction ACID chặt** trên nhiều thực thể (tài chính, giao dịch) — MongoDB có transaction (từ 4.0)
  nhưng SQL vẫn mạnh hơn cho bài toán này.
- Dữ liệu **quan hệ phức tạp**, cần JOIN nhiều.
- Đã có hệ sinh thái/tooling phân tích dựa trên **chuẩn SQL**.

> **MongoDB Atlas** là dịch vụ **MongoDB managed trên cloud** (DBaaS) của chính MongoDB Inc. — họ tự lo
> provisioning, backup, scaling, replica set; bạn chỉ cần kết nối và dùng.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **MongoDB** = CSDL **NoSQL dạng Document** phổ biến nhất, mã nguồn mở; lưu dữ liệu dạng **document giống JSON
  (BSON)** linh hoạt, có thể lồng/nhúng mảng, **không cần schema cố định**.
- **Thuật ngữ:** table → **collection**, row → **document**, column → **field**, PK → `_id`; ít dùng JOIN (thiên
  về denormalize), có `$lookup` khi cần.
- **Điểm mạnh:** schema linh hoạt, đọc nhanh (dữ liệu tự chứa), **scale ngang native (sharding)**, **replica
  set** cho high availability, **aggregation pipeline** mạnh, nhiều loại index.
- **Dùng khi:** dữ liệu linh hoạt/bán cấu trúc, ít JOIN, cần mở rộng lớn. **Không dùng khi:** cần ACID chặt
  (tài chính), quan hệ phức tạp/JOIN nhiều.
- **MongoDB Atlas:** bản MongoDB managed trên cloud (tự lo scaling, backup, HA).

[↑ Mục lục](#mục-lục)

<a id="q3-4"></a>
### 3.4. Elasticsearch là gì?

#### 📖 Giới thiệu kiến thức

**Elasticsearch (ES)** là **công cụ tìm kiếm & phân tích phân tán (distributed search & analytics engine)** mã
nguồn mở, xây trên thư viện **Apache Lucene**. Thường xếp vào nhóm **NoSQL dạng document** (lưu JSON — mục 3.1),
nhưng thế mạnh đặc trưng là **full-text search** cực nhanh và **phân tích/aggregation** near real-time trên khối
lượng dữ liệu lớn.

**Đặc điểm chính:**

- **Full-text search mạnh:** tìm kiếm văn bản có **xếp hạng độ liên quan (relevance score)**, **fuzzy /
  typo-tolerance**, phân tích ngôn ngữ (analyzer, tokenizer, stemming) — vượt xa `LIKE '%...%'` của SQL.
- **Inverted index (chỉ mục ngược) — cấu trúc cốt lõi:** thay vì map "document → từ", nó map "**từ → danh sách
  document chứa từ đó**" → tìm một từ trong hàng triệu tài liệu **cực nhanh**.
- **Distributed & scalable:** dữ liệu chia thành **shard**, nhân bản thành **replica** (liên hệ mục 16.1), chạy
  trên **cluster** nhiều node → scale ngang & high availability.
- **Near real-time:** document index xong là tìm được gần như ngay (~1 giây).
- **RESTful API + JSON:** giao tiếp qua HTTP/JSON; truy vấn bằng **Query DSL** (JSON). **Schema linh hoạt** (mapping).
- **Aggregation** mạnh cho phân tích, thống kê, gom nhóm.

**Ánh xạ thuật ngữ với SQL:**

| SQL | Elasticsearch |
|-----|---------------|
| Table | **Index** |
| Row | **Document** (JSON) |
| Column | **Field** |
| Schema | **Mapping** |

**ELK / Elastic Stack:** ES thường đi cùng **Kibana** (trực quan hoá/dashboard) + **Logstash/Beats** (thu thập &
đưa dữ liệu vào) → bộ **ELK Stack** nổi tiếng cho **log & observability**.

**Vì sao không dùng SQL `LIKE` mà cần ES?** (liên hệ câu 16 — full-text search)
- `LIKE '%keyword%'` → **full table scan**, không dùng index, chậm khi dữ liệu lớn; **không** có ranking, không
  xử lý ngôn ngữ (đồng nghĩa, số nhiều, gõ sai).
- ES với **inverted index** → tìm nhanh + **xếp hạng theo độ liên quan** + fuzzy/typo.

**Use case tiêu biểu:** full-text search cho web/app (sản phẩm, bài viết); **log & observability** (ELK);
analytics/dashboard near real-time; autocomplete, gợi ý, tìm kiếm địa lý (geo).

**Lưu ý quan trọng:**
- ES **KHÔNG thay thế** DB quan hệ chính — thường dùng **song song**: dữ liệu gốc (source of truth) ở
  MySQL/PostgreSQL, **đồng bộ (sync)** sang ES để tìm kiếm (qua CDC, message queue, hoặc reindex định kỳ).
- **Yếu** về transaction/ACID và JOIN phức tạp → không phải nơi lưu dữ liệu giao dịch.
- **Eventual consistency** (near real-time, không tức thì).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Elasticsearch** = công cụ **tìm kiếm & phân tích phân tán** mã nguồn mở trên **Lucene**, thuộc nhóm **NoSQL
  document (JSON)**; thế mạnh là **full-text search** + **analytics** near real-time.
- **Cốt lõi: inverted index** (từ → document) → tìm văn bản trong khối lượng lớn cực nhanh, có **ranking, fuzzy,
  typo-tolerance** — vượt xa SQL `LIKE`.
- **Distributed:** shard + replica trên cluster → scale ngang & HA; giao tiếp **REST/JSON** (Query DSL).
- **ELK Stack** (Elasticsearch + Kibana + Logstash/Beats) — chuẩn cho **log/observability**.
- **Dùng khi:** full-text search, log/analytics, autocomplete. **Lưu ý:** không thay DB chính (yếu ACID/JOIN);
  thường **dùng song song** với SQL và **đồng bộ dữ liệu** sang ES.

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Thứ tự thực thi (logical query processing order) của các mệnh đề trong câu SQL

### 📖 Giới thiệu kiến thức

Điều dễ nhầm nhất: **thứ tự bạn VIẾT câu SQL khác với thứ tự database THỰC THI nó.** Ta viết bắt đầu bằng
`SELECT`, nhưng engine lại xử lý `SELECT` gần **cuối cùng**. Chuỗi thực thi này gọi là **logical query
processing order** (thứ tự xử lý logic) — chuẩn ANSI SQL, áp dụng cho MySQL, PostgreSQL, SQL Server...

> Lưu ý: đây là thứ tự **logic** (để hiểu kết quả câu query ra sao). Còn optimizer thực tế có thể sắp xếp
> lại các bước để chạy nhanh hơn, miễn **kết quả cuối vẫn giống** thứ tự logic này.

**Thứ tự thực thi logic (đầy đủ):**

| # | Mệnh đề | Làm gì |
|---|---------|--------|
| 1 | **FROM** | Xác định (các) bảng nguồn. |
| 2 | **JOIN ... ON** | Ghép bảng, `ON` lọc điều kiện ghép → tạo tập dữ liệu trung gian. |
| 3 | **WHERE** | Lọc **từng dòng** (chưa gom nhóm). Không dùng được hàm tổng hợp (`SUM`, `COUNT`). |
| 4 | **GROUP BY** | Gom các dòng thành nhóm. |
| 5 | **HAVING** | Lọc **theo nhóm** (dùng được hàm tổng hợp, khác `WHERE`). |
| 6 | **SELECT** | Tính biểu thức, chọn cột, đặt **alias** (bí danh). |
| 7 | **DISTINCT** | Loại bỏ dòng trùng. |
| 8 | **ORDER BY** | Sắp xếp kết quả (dùng được alias vì `SELECT` đã chạy trước). |
| 9 | **LIMIT / OFFSET** | Giới hạn số dòng trả về (SQL Server: `TOP`/`FETCH`). |

**So sánh thứ tự VIẾT vs thứ tự THỰC THI:**

```sql
-- Thứ tự VIẾT (cách ta gõ):
SELECT ... FROM ... JOIN ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT ...

-- Thứ tự THỰC THI (engine chạy):
FROM → JOIN/ON → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
```

**Hệ quả quan trọng (rất hay hỏi tại sao):**

1. **Không dùng được alias của `SELECT` trong `WHERE`/`GROUP BY`/`HAVING`** — vì `SELECT` chạy *sau*. Nhưng
   **dùng được alias trong `ORDER BY`** — vì `ORDER BY` chạy *sau* `SELECT`.

```sql
SELECT price * quantity AS total
FROM orders
WHERE total > 100        -- ❌ LỖI: alias 'total' chưa tồn tại lúc WHERE chạy
ORDER BY total;          -- ✔ OK: SELECT đã chạy trước ORDER BY
```

**Cách viết đúng** cho câu trên — có 2 cách:

```sql
-- Cách 1 (phổ biến, hiệu quả nhất): lặp lại biểu thức trong WHERE
SELECT price * quantity AS total
FROM orders
WHERE price * quantity > 100   -- ✔ dùng biểu thức gốc, không dùng alias
ORDER BY total;                -- ✔ ORDER BY vẫn dùng alias được
```

```sql
-- Cách 2: bọc trong subquery/CTE để "vật chất hóa" alias rồi mới lọc
SELECT *
FROM (
    SELECT price * quantity AS total
    FROM orders
) AS t
WHERE total > 100              -- ✔ giờ 'total' đã là cột thật của bảng con t
ORDER BY total;
```

> Ưu tiên **Cách 1**: ngắn gọn và tối ưu hơn (optimizer dễ dùng index/biểu thức). Cách 2 dùng khi biểu
> thức phức tạp/lặp nhiều lần và muốn viết gọn, nhưng có thêm tầng subquery.

2. **`WHERE` vs `HAVING`:** `WHERE` lọc dòng *trước* khi gom nhóm (không dùng được `COUNT`/`SUM`); `HAVING`
   lọc *sau* khi đã `GROUP BY` (dùng được hàm tổng hợp).

```sql
SELECT user_id, COUNT(*) AS cnt
FROM orders
WHERE status = 'paid'    -- lọc dòng trước
GROUP BY user_id
HAVING COUNT(*) > 3;     -- lọc nhóm sau
```

### 🎯 Tóm tắt trả lời phỏng vấn

- Thứ tự **thực thi logic** (khác thứ tự viết): **`FROM` → `JOIN/ON` → `WHERE` → `GROUP BY` → `HAVING` →
  `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT/OFFSET`.**
- Ta viết `SELECT` đầu tiên nhưng nó chạy **gần cuối** (bước 6), sau `WHERE`/`GROUP BY`/`HAVING`.
- Vì vậy: **không dùng được alias trong `WHERE`/`HAVING`** (SELECT chưa chạy), nhưng **dùng được trong
  `ORDER BY`** (SELECT đã chạy).
- **`WHERE` lọc từng dòng trước `GROUP BY`** (không có hàm tổng hợp); **`HAVING` lọc nhóm sau `GROUP BY`**
  (có `COUNT`/`SUM`).
- Đây là thứ tự **logic**; optimizer có thể sắp xếp lại miễn kết quả không đổi.

[↑ Mục lục](#mục-lục)

<a id="q4-1"></a>
### 4.1. Phân biệt WHERE và HAVING trong SQL

#### 📖 Giới thiệu kiến thức

`WHERE` và `HAVING` đều là mệnh đề **lọc (filter)**, nhưng lọc ở **hai giai đoạn khác nhau** của quá trình xử lý
câu truy vấn (xem lại thứ tự thực thi ở câu 4). Đây chính là hệ quả quan trọng nhất của thứ tự thực thi: `WHERE`
chạy **trước** khi gom nhóm, `HAVING` chạy **sau** khi đã `GROUP BY`.

| Tiêu chí | `WHERE` | `HAVING` |
|----------|---------|----------|
| Lọc cái gì | **Từng dòng** (row) | **Từng nhóm** (group) |
| Khi nào chạy | **Trước** `GROUP BY` (bước 3) | **Sau** `GROUP BY` (bước 5) |
| Dùng hàm tổng hợp (`SUM`/`COUNT`/`AVG`…)? | **KHÔNG** được | **ĐƯỢC** |
| Dùng cột thường (không tổng hợp)? | Được (và **nên** dùng) | Được (nhưng **nên đẩy xuống** `WHERE`) |
| Có cần `GROUP BY`? | Không | Thường đi kèm `GROUP BY` |

> Nhớ nhanh: **`WHERE` lọc dòng, `HAVING` lọc nhóm.** Vì `WHERE` chạy trước khi gom nên **chưa** biết
> `COUNT`/`SUM` là bao nhiêu (chưa gom thì chưa tính tổng) → **không thể** dùng hàm tổng hợp. `HAVING` chạy **sau**
> khi gom → đã có kết quả tổng hợp → dùng được.

```sql
-- ❌ LỖI: dùng hàm tổng hợp trong WHERE (WHERE chạy trước GROUP BY, chưa có COUNT)
SELECT user_id FROM orders
WHERE COUNT(*) > 5
GROUP BY user_id;

-- ✔ ĐÚNG: điều kiện trên kết quả tổng hợp → phải dùng HAVING
SELECT user_id, COUNT(*) AS cnt
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

**Dùng cùng nhau (rất phổ biến):** `WHERE` **lọc dòng trước** (giảm lượng dữ liệu phải gom), rồi `GROUP BY`
gom, rồi `HAVING` **lọc nhóm** dựa trên kết quả tổng hợp.

```sql
-- Tìm user có > 5 đơn 'paid' VÀ tổng tiền > 1.000.000
SELECT user_id, COUNT(*) AS cnt, SUM(total) AS revenue
FROM orders
WHERE status = 'paid'                                    -- WHERE: lọc dòng (trước gom)
GROUP BY user_id
HAVING COUNT(*) > 5 AND SUM(total) > 1000000;            -- HAVING: lọc nhóm (sau gom, dùng aggregate)
```

**Mẹo hiệu năng:** nếu một điều kiện **không** dùng hàm tổng hợp (vd `status = 'paid'`, `dept != 'HR'`), hãy
đặt nó ở **`WHERE`** chứ đừng để ở `HAVING`. Lý do: `WHERE` lọc **trước** khi gom → ít dòng phải gom hơn →
`GROUP BY` nhẹ hơn. Nếu để ở `HAVING`, database **gom hết rồi mới loại** → phí công.

```sql
-- TỐT: đẩy điều kiện cột thường xuống WHERE
SELECT dept, AVG(salary) AS avg_sal
FROM employees
WHERE dept != 'HR'          -- lọc dòng trước → ít dòng gom hơn
GROUP BY dept
HAVING AVG(salary) > 1000;  -- chỉ điều kiện tổng hợp mới để HAVING

-- KÉM (gom tất cả phòng ban rồi mới loại HR):
SELECT dept, AVG(salary) AS avg_sal
FROM employees
GROUP BY dept
HAVING dept != 'HR' AND AVG(salary) > 1000;   -- chạy được nhưng kém hiệu quả
```

> **Ngoại lệ:** nếu dùng `HAVING` trên **cột đã `GROUP BY`** mà không có `WHERE`, một số optimizer có thể tự đẩy
> điều kiện xuống (predicate pushdown) nên thực tế có thể tương đương — nhưng viết đúng chỗ (`WHERE`) luôn rõ
> ràng và an toàn hơn, không nên ỷ lại vào optimizer.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`WHERE`** lọc **từng dòng** (chạy **trước** `GROUP BY`); **`HAVING`** lọc **từng nhóm** (chạy **sau** `GROUP BY`).
- **Khác biệt mấu chốt:** `WHERE` **không** dùng được hàm tổng hợp (`SUM`/`COUNT`/`AVG`), `HAVING` **dùng được**
  (vì nó chạy sau khi đã gom nhóm, đã có kết quả tổng hợp).
- **Dùng cùng nhau:** `WHERE` lọc dòng trước → `GROUP BY` gom → `HAVING` lọc nhóm theo kết quả tổng hợp.
- **Mẹo hiệu năng:** điều kiện **không** dùng hàm tổng hợp → luôn để ở **`WHERE`** (lọc trước khi gom = ít dòng
  phải gom); chỉ dùng `HAVING` cho điều kiện **gắn hàm tổng hợp**.
- Bản chất khác biệt chính là **thứ tự thực thi** (câu 4): `WHERE` ở bước 3, `HAVING` ở bước 5.

[↑ Mục lục](#mục-lục)

<a id="q4-2"></a>
### 4.2. Logic lọc theo điều kiện (vd kiểm tra range): đặt ở WHERE hay SELECT?

#### 📖 Giới thiệu kiến thức

**Tình huống:** có một logic kiểm tra **range** — chỉ trả về các point **nằm trong khoảng** đó. Nên đặt điều
kiện này ở **`WHERE`** hay ở **`SELECT`**?

**Câu trả lời: đặt ở `WHERE`.** Vì đây là **lọc dòng**, mà lọc dòng là việc của `WHERE`, không phải `SELECT`.
Cần phân biệt rõ vai trò hai mệnh đề (hệ quả của thứ tự thực thi — câu 4):

| Mệnh đề | Vai trò | Chạy khi (câu 4) |
|---------|---------|------------------|
| **`WHERE`** | **Lọc DÒNG (selection)** — quyết định dòng nào **được trả về** | Sớm — **bước 3** |
| **`SELECT`** | **Chọn/tính CỘT (projection)** — quyết định **giá trị/cột nào xuất ra** cho các dòng **đã qua lọc** | Muộn — **bước 6** |

**Vì sao `WHERE` là tốt nhất — 3 lý do:**

1. **Đúng ngữ nghĩa (quan trọng nhất):** đặt điều kiện ở `SELECT` **KHÔNG lọc gì cả** — nó chỉ tạo ra một **cột
   giá trị/cờ** (true/false) cho **MỌI dòng**; dòng ngoài range **vẫn được trả về**. Muốn thực sự **loại bỏ**
   dòng ngoài khoảng thì **bắt buộc** dùng `WHERE`.
2. **Hiệu năng — lọc sớm:** `WHERE` chạy **trước** (bước 3), loại bớt dòng ngay → các bước sau (`SELECT`,
   `ORDER BY`, trả về) xử lý **ít dữ liệu hơn**. Biểu thức ở `SELECT` chạy cho từng dòng nhưng **không giảm** số
   dòng kết quả.
3. **Dùng được index:** `WHERE` với range trên cột **có index** → **B-tree range scan** (`EXPLAIN` cho `type =
   range` — mục 13.1), tránh full scan. Biểu thức lọc nằm trong `SELECT` **không** tận dụng index để lọc.

```sql
-- ✔ ĐÚNG: lọc ở WHERE — chỉ trả về point trong khoảng [10, 20]
SELECT id, value FROM points
WHERE value BETWEEN 10 AND 20;          -- = value >= 10 AND value <= 20

-- ❌ Đặt điều kiện ở SELECT: KHÔNG lọc — trả về TẤT CẢ dòng, chỉ thêm 1 cột cờ
SELECT id, value, (value BETWEEN 10 AND 20) AS in_range
FROM points;                            -- mọi dòng vẫn xuất hiện, kèm true/false
```

**Vậy khi nào mới đặt logic ở `SELECT`?** — Khi bạn **muốn giữ mọi dòng** nhưng **gắn nhãn / tính giá trị**
(projection), *không* nhằm loại bỏ dòng:

```sql
-- Giữ mọi dòng, chỉ GẮN NHÃN in/out (đây là projection, không phải filter)
SELECT id, value,
       CASE WHEN value BETWEEN 10 AND 20 THEN 'in' ELSE 'out' END AS label
FROM points;
```

**Lưu ý mở rộng:**
- Nếu điều kiện dựa trên **hàm tổng hợp** (`SUM`/`COUNT` sau `GROUP BY`) thì không đặt ở `WHERE`/`SELECT` mà dùng
  **`HAVING`** (xem câu 4.1).
- Nếu "point trong range" là **toạ độ địa lý (spatial)** → vẫn đặt ở **`WHERE`**, dùng hàm & **index không gian**:
  PostGIS `WHERE ST_DWithin(geom, :point, :radius)` (mục 1.3), MySQL `ST_Contains`/`MBRContains`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Đặt ở `WHERE`.** Kiểm tra range để **quyết định trả về dòng nào** là **lọc dòng (selection)** → thuộc
  `WHERE`, không phải `SELECT`.
- **`SELECT` chỉ chọn/tính cột (projection)** cho các dòng **đã qua lọc**; đặt điều kiện ở `SELECT` chỉ tạo **cột
  cờ true/false**, **không loại bỏ dòng nào**.
- **Vì sao `WHERE` tốt nhất:** (1) **đúng ngữ nghĩa** (thực sự lọc); (2) **lọc sớm** (`WHERE` bước 3, trước
  `SELECT` — câu 4) → xử lý ít dữ liệu; (3) **dùng được index** (range scan, `EXPLAIN type=range` — mục 13.1).
- **Chỉ đặt ở `SELECT`** khi muốn **giữ mọi dòng và gắn nhãn/tính giá trị** (`CASE`...). Điều kiện trên **hàm
  tổng hợp** → **`HAVING`** (câu 4.1); range **không gian** → vẫn `WHERE` + spatial index (`ST_DWithin` — mục 1.3).

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. Phân biệt LIKE và ILIKE trong SQL

### 📖 Giới thiệu kiến thức

**LIKE** và **ILIKE** đều là toán tử **so khớp mẫu (pattern matching)** trên chuỗi, dùng hai ký tự đại diện
(wildcard):

- **`%`** — khớp **bất kỳ chuỗi nào** (kể cả chuỗi rỗng), 0 hoặc nhiều ký tự.
- **`_`** — khớp **đúng một** ký tự bất kỳ.

```sql
SELECT * FROM users WHERE name LIKE 'An%';   -- bắt đầu bằng 'An' (Andrea, Anna, Andrew)
SELECT * FROM users WHERE name LIKE '%son';  -- kết thúc bằng 'son' (Jackson, Mason)
SELECT * FROM users WHERE name LIKE '_an';   -- đúng 3 ký tự, đuôi 'an' (Dan, Ran khớp; Sam không)
```

**Khác biệt cốt lõi: `LIKE` phân biệt hoa–thường (case-sensitive), còn `ILIKE` thì không.**

| Toán tử | Phân biệt hoa thường? | Chuẩn SQL? | Có ở đâu |
|---------|----------------------|------------|----------|
| **LIKE** | **Có** (trong PostgreSQL) | **Chuẩn ANSI SQL** | Hầu hết DB: PostgreSQL, MySQL, SQL Server, SQLite, Oracle |
| **ILIKE** | **Không** (case-insensitive) | **KHÔNG** — phần mở rộng riêng của **PostgreSQL** | **Chỉ PostgreSQL** |

> Chữ **"I"** trong **ILIKE** = **Insensitive** (không nhạy cảm hoa/thường). Về cú pháp và wildcard, ILIKE giống
> hệt LIKE, chỉ thêm việc **bỏ qua phân biệt chữ hoa/thường**.

```sql
-- PostgreSQL
SELECT * FROM users WHERE name LIKE  'an';   -- chỉ khớp đúng 'an' (không khớp 'An', 'AN')
SELECT * FROM users WHERE name ILIKE 'an';   -- khớp cả 'an', 'An', 'AN', 'aN'...
```

**Tại sao có hai toán tử? — vì mỗi DB xử lý hoa/thường khác nhau:**

- **PostgreSQL:** so sánh chuỗi **luôn phân biệt hoa thường** theo mặc định → `LIKE` cũng phân biệt. Khi cần
  "không phân biệt", PostgreSQL cung cấp thêm **`ILIKE`** thay vì phải tự `LOWER()`.
- **MySQL / SQL Server:** `LIKE` **đã không phân biệt hoa thường** sẵn theo **collation** mặc định (vd MySQL
  dùng collation đuôi `_ci` = case-insensitive) → **không cần và không có** ILIKE.
- **SQLite:** `LIKE` không phân biệt hoa thường cho **ASCII** (a–z) mặc định; không có ILIKE.

> **Hệ quả dễ nhầm:** cùng câu `WHERE name LIKE 'an'`, ở **PostgreSQL** thì `LIKE` chỉ khớp `'an'` (phải dùng
> `ILIKE` để khớp cả `'An'`), còn ở **MySQL** thì `LIKE` đã khớp `'An'` rồi. Đây là nguồn bug khi chuyển code
> giữa hai DB.

**Các cách thay thế khi muốn so khớp không phân biệt hoa thường (không dùng ILIKE):**

```sql
-- (1) Dùng LOWER() cả hai vế — chạy được trên mọi DB (portable nhất)
SELECT * FROM users WHERE LOWER(name) LIKE LOWER('An%');

-- (2) PostgreSQL: regex ~* (case-insensitive); ~ là case-sensitive
SELECT * FROM users WHERE name ~* '^an';          -- tương đương ILIKE 'an%'

-- (3) PostgreSQL: kiểu citext (cài extension) — cả cột thành không phân biệt hoa thường
CREATE EXTENSION IF NOT EXISTS citext;
CREATE TABLE t (name CITEXT);                     -- LIKE / = đều tự không phân biệt hoa thường
```

**Lưu ý hiệu năng (rất hay hỏi kèm): Index có dùng được cho LIKE không?**

- **`LIKE 'tiền_tố%'`** (wildcard ở CUỐI): **dùng được index** (B-tree range scan) — cả MySQL và PostgreSQL đều
  tận dụng được.
- **`LIKE '%hậu_tố'`** hoặc **`LIKE '%giữa%'`** (wildcard ở ĐẦU/cả hai): **không** dùng được B-tree index →
  **quét toàn bảng** (full table scan), chậm khi bảng lớn.

```sql
-- PostgreSQL: tối ưu prefix search không phân biệt hoa thường bằng expression index
CREATE INDEX idx_users_name_lower ON users (LOWER(name) text_pattern_ops);
-- rồi query: WHERE LOWER(name) LIKE 'an%'
```

> Về mặt toán tử nội bộ trong PostgreSQL: `LIKE` ≡ `~~`, `ILIKE` ≡ `~~*`, `NOT LIKE` ≡ `!~~`, `NOT ILIKE` ≡
> `!~~*`.

### 🎯 Tóm tắt trả lời phỏng vấn

- **LIKE** và **ILIKE** đều là toán tử **so khớp mẫu chuỗi** với wildcard `%` (nhiều ký tự) và `_` (đúng 1 ký tự).
- **Khác biệt chính:** `LIKE` **phân biệt hoa–thường**, `ILIKE` **không phân biệt** (chữ "I" = Insensitive).
- **`ILIKE` là riêng của PostgreSQL** (không phải chuẩn SQL); MySQL/SQL Server/SQLite **không có** vì `LIKE` của
  chúng **đã không phân biệt hoa thường** sẵn theo collation mặc định.
- **Cách làm case-insensitive dùng chung mọi DB:** `LOWER(cột) LIKE LOWER('mẫu')`; PostgreSQL còn có regex `~*`
  và kiểu `citext`.
- **Hiệu năng:** `LIKE 'abc%'` (prefix) **dùng được index**; `LIKE '%abc'` / `'%abc%'` **không** → quét toàn
  bảng. Tối ưu prefix không phân biệt hoa thường bằng **expression index trên `LOWER()`**.
- **Cẩn thận bug khi chuyển code giữa DB:** cùng `LIKE`, PostgreSQL phân biệt hoa thường nhưng MySQL thì không.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Phân biệt UNION và UNION ALL trong SQL

### 📖 Giới thiệu kiến thức

**UNION** và **UNION ALL** đều là toán tử **tập hợp (set operator)** dùng để **ghép kết quả của hai (hoặc nhiều)
câu `SELECT` theo chiều dọc** — tức nối các bộ kết quả lại thành một (thêm dòng), khác với **JOIN** ghép theo
chiều ngang (thêm cột).

```sql
-- Ghép danh sách khách hàng và nhà cung cấp thành 1 danh sách
SELECT name FROM customers
UNION ALL
SELECT name FROM suppliers;
```

**Khác biệt cốt lõi: `UNION` loại bỏ dòng trùng, `UNION ALL` giữ nguyên.**

| Toán tử | Xử lý dòng trùng? | Tốc độ | Dùng khi |
|---------|-------------------|--------|----------|
| **UNION** | **Có** — gỡ bỏ các dòng trùng nhau (như `DISTINCT` ngầm) | **Chậm hơn** (phải sort/hash để tìm trùng) | Cần kết quả **không trùng lặp** |
| **UNION ALL** | **Không** — giữ **tất cả** dòng, kể cả trùng | **Nhanh hơn** (chỉ nối, không tìm trùng) | Biết chắc **không có trùng**, hoặc **muốn giữ** trùng |

> Nhớ nhanh: **`UNION` = `UNION ALL` + `DISTINCT`**. Vì phải gỡ trùng nên `UNION` tốn thêm bước **sort hoặc
> hash** trong execution plan → chậm và tốn bộ nhớ hơn khi dữ liệu lớn.

```sql
-- Giả sử 'Acme Corp' có cả trong customers VÀ suppliers
SELECT name FROM customers
UNION                                   -- 'Acme Corp' chỉ xuất hiện 1 LẦN (đã gỡ trùng)
SELECT name FROM suppliers;

SELECT name FROM customers
UNION ALL                               -- 'Acme Corp' xuất hiện 2 LẦN (giữ nguyên)
SELECT name FROM suppliers;
```

**Quy tắc bắt buộc cho mọi set operator (UNION/UNION ALL/INTERSECT/EXCEPT):**

1. **Số cột phải bằng nhau** ở mọi `SELECT`.
2. **Kiểu dữ liệu tương thích** ở các cột tương ứng.
3. **Thứ tự cột** phải khớp (không dựa vào tên cột).
4. **Tên cột kết quả** lấy theo **`SELECT` đầu tiên**.

```sql
-- ❌ LỖI: số cột khác nhau
SELECT id, name    FROM customers
UNION
SELECT id          FROM suppliers;

-- ✔ Sửa: thêm cột pad (NULL/giá trị mặc định) cho đủ số cột
SELECT id, name       FROM customers
UNION ALL
SELECT supplier_id, NULL  FROM suppliers;

-- ✔ Tên cột kết quả = của SELECT đầu ('id', 'name'), bất kể SELECT 2 đặt tên gì
SELECT id, name           FROM customers
UNION ALL
SELECT supplier_id, company_name FROM suppliers;
```

> Khi cần **sắp xếp kết quả cuối**, chỉ được đặt `ORDER BY` ở **cuối câu cuối cùng** (không `ORDER BY` giữa
> chừng), vì `ORDER BY` tác dụng lên toàn bộ kết quả ghép.

**Bộ set operator trong SQL:**

| Toán tử | Ý nghĩa |
|---------|---------|
| `UNION` | Hợp — tất cả dòng, **gỡ trùng** |
| `UNION ALL` | Hợp — tất cả dòng, **giữ trùng** |
| `INTERSECT` | Giao — chỉ dòng **có ở cả hai** kết quả |
| `EXCEPT` (`MINUS` trong Oracle) | Hiệu — dòng **chỉ ở kết quả đầu**, không ở kết quả sau |

> So với **JOIN**: set operator ghép **dọc** (thêm dòng, cùng cấu trúc cột), còn JOIN ghép **ngang** (thêm cột
> từ bảng khác theo điều kiện khớp).

**Lưu ý hiệu năng (mẹo rất hay hỏi):** Đừng lạm dụng `UNION`. Nếu **biết chắc hai nguồn không trùng** (hoặc
muốn giữ trùng), hãy dùng **`UNION ALL`** để tránh bước gỡ trùng tốn kém. Nhiều dev viết `UNION` theo thói quen
trong khi `UNION ALL` mới là điều họ cần → gây chậm không cần thiết trên tập dữ liệu lớn. Kiểm chứng bằng
`EXPLAIN`: câu có `UNION` sẽ thấy thêm node **Sort** (MySQL) / **HashAggregate** hoặc **Unique** (PostgreSQL).

### 🎯 Tóm tắt trả lời phỏng vấn

- **UNION** và **UNION ALL** đều là **set operator** — **ghép kết quả nhiều `SELECT` theo chiều dọc** (thêm dòng),
  khác JOIN ghép theo chiều ngang (thêm cột).
- **Khác biệt chính:** `UNION` **gỡ bỏ dòng trùng** (như `DISTINCT` ngầm, chậm hơn vì phải sort/hash); `UNION ALL`
  **giữ nguyên tất cả** dòng (nhanh hơn, chỉ nối).
- **Quy tắc bắt buộc:** các `SELECT` phải **cùng số cột**, **kiểu dữ liệu tương thích**, **cùng thứ tự cột**;
  tên cột kết quả lấy theo **`SELECT` đầu tiên**.
- **`ORDER BY`** (nếu có) đặt **cuối câu cuối**, áp dụng cho toàn bộ kết quả ghép.
- **Bộ set operator:** `UNION` (gỡ trùng) · `UNION ALL` (giữ trùng) · `INTERSECT` (giao) · `EXCEPT` (hiệu).
- **Hiệu năng:** ưu tiên **`UNION ALL`** khi không cần gỡ trùng; chỉ dùng `UNION` khi thực sự cần kết quả duy nhất.

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. JOIN và Subquery: cái nào nhanh hơn?

### 📖 Giới thiệu kiến thức

Câu trả lời chuẩn: **"Không có cái nào luôn nhanh hơn — tùy trường hợp, và phần lớn tình huống query
optimizer sẽ tối ưu chúng về cùng một kế hoạch thực thi."** Người phỏng vấn muốn nghe bạn hiểu *khi nào* cái
nào tốt hơn, chứ không phải một câu tuyệt đối.

**Nhắc lại khái niệm:**
- **JOIN:** ghép nhiều bảng theo điều kiện để lấy dữ liệu trong **một lần quét kết hợp**.
- **Subquery (truy vấn con):** một `SELECT` lồng bên trong câu khác — ở `WHERE`, `FROM` (derived table),
  hay `SELECT`.

**1. Trường hợp JOIN thường nhanh hơn: correlated subquery.**
**Correlated subquery** (subquery phụ thuộc từng dòng của query ngoài) thường **chạy lặp lại cho mỗi dòng** →
chậm khi bảng lớn. JOIN xử lý theo tập hợp (set-based) nên thường nhanh hơn nhiều.

```sql
-- CHẬM: correlated subquery chạy lại cho MỖI dòng của users
SELECT name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- NHANH hơn (thường): JOIN theo tập hợp
SELECT DISTINCT u.name
FROM users u JOIN orders o ON o.user_id = u.id;
```

**2. Trường hợp subquery ổn / rõ ràng hơn:**
- **`IN` với danh sách nhỏ, cố định** hoặc subquery **không tương quan** (chạy 1 lần): optimizer thường
  chuyển thành semi-join, tốc độ tương đương JOIN.
- Cần **giá trị tổng hợp** (max, count) để lọc → subquery/derived table đọc dễ hiểu hơn.
- Tránh **nhân bản dòng**: JOIN 1–nhiều có thể làm phồng số dòng (phải `DISTINCT`/`GROUP BY`); dùng
  `EXISTS`/`IN` thì không.

```sql
-- EXISTS thường tốt hơn IN khi subquery trả nhiều dòng / có NULL
SELECT name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id AND o.total > 100);
```

**3. `EXISTS` vs `IN` (hay hỏi kèm):**
- **`EXISTS`**: dừng ngay khi tìm thấy dòng đầu (short-circuit) → tốt khi subquery lớn; an toàn với `NULL`.
- **`IN`**: tốt khi danh sách con **nhỏ**; **cẩn thận `NOT IN` với NULL** → có thể trả kết quả sai (rỗng).

**4. Yếu tố quyết định thực sự vẫn là INDEX & optimizer.**
Cột dùng để JOIN / trong điều kiện subquery **có index** hay không mới là yếu tố lớn nhất — giống câu chuyện
index ở mọi truy vấn. Các optimizer hiện đại (MySQL 8+, PostgreSQL) thường **viết lại subquery thành join**
nội bộ, nên hai cách nhiều khi cho **cùng execution plan**.

**5. Đừng đoán — dùng `EXPLAIN`.**

```sql
EXPLAIN SELECT DISTINCT u.name FROM users u JOIN orders o ON o.user_id = u.id;
EXPLAIN SELECT name FROM users u WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
-- So sánh: type, key (index dùng), rows (số dòng ước tính). Chọn plan rẻ hơn.
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Không có cái nào luôn nhanh hơn** — tùy trường hợp; optimizer hiện đại thường viết lại subquery thành
  join nên nhiều khi **cùng một execution plan**.
- **JOIN thường nhanh hơn** khi thay cho **correlated subquery** (subquery chạy lặp cho mỗi dòng → chậm).
- **Subquery/`EXISTS`/`IN`** gọn & tốt khi: kiểm tra tồn tại, lọc theo giá trị tổng hợp, hoặc tránh nhân
  bản dòng (JOIN 1–nhiều phải `DISTINCT`).
- **`EXISTS`** (short-circuit, an toàn NULL) thường hơn **`IN`** khi subquery lớn; cẩn thận **`NOT IN` + NULL**.
- Yếu tố quyết định thật sự là **INDEX + optimizer**; luôn kiểm chứng bằng **`EXPLAIN`** thay vì phán đoán.

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. So sánh Subquery và CTE trong SQL

### 📖 Giới thiệu kiến thức

**Subquery (truy vấn con)** và **CTE (Common Table Expression, `WITH`)** là hai cách phổ biến để **chia nhỏ một
truy vấn phức tạp** thành các phần dễ quản lý. Cả hai đều tạo ra một "bộ kết quả trung gian" rồi dùng trong câu
chính — khác nhau ở **cách khai báo** và **khả năng tái sử dụng**.

```sql
-- SUBQUERY (derived table): lồng inline trong FROM, KHÔNG tên (chỉ có alias 't')
SELECT t.dept, t.avg_sal
FROM (
    SELECT dept, AVG(salary) AS avg_sal FROM employees GROUP BY dept
) AS t
WHERE t.avg_sal > 1000;

-- CTE: khai báo tách biệt bằng WITH, CÓ TÊN (dept_avg)
WITH dept_avg AS (
    SELECT dept, AVG(salary) AS avg_sal FROM employees GROUP BY dept
)
SELECT dept, avg_sal FROM dept_avg WHERE avg_sal > 1000;
```

> Hai câu trên cho **kết quả giống hệt nhau**. Có thể coi **CTE là một subquery được đặt tên và khai báo trước**.

**So sánh chi tiết:**

| Tiêu chí | Subquery | CTE (`WITH`) |
|----------|----------|--------------|
| Cách khai báo | **Lồng inline** ngay tại WHERE/FROM/SELECT | **Tách biệt**, khai báo `WITH ... AS (...)` ở đầu câu |
| Có tên không | Không (chỉ alias cho derived table) | **Có tên** — như một "biến" chứa kết quả trung gian |
| Tái sử dụng | Mỗi lần dùng phải **viết lại** | Tham chiếu **nhiều lần** trong cùng câu |
| Đệ quy | Không hỗ trợ trực tiếp | **Có** (`WITH RECURSIVE` — xem câu 9) |
| Tính dễ đọc | Kém khi lồng nhiều tầng | **Tốt hơn** — đọc từ trên xuống, cấu trúc rõ |
| Hiệu năng | Tương đương CTE trong hầu hết trường hợp | Tương đương (trừ PostgreSQL `MATERIALIZED`) |

**Khi nào CTE vượt trội? — khi cần tái dùng cùng một kết quả trung gian nhiều lần:**

```sql
-- CTE: tính thống kê theo phòng ban MỘT LẦN, dùng lại nhiều lần
WITH dept_stats AS (
    SELECT dept, AVG(salary) AS avg_sal, COUNT(*) AS cnt
    FROM employees GROUP BY dept
)
SELECT e.name, e.salary, d.avg_sal,
       e.salary - d.avg_sal AS diff        -- dùng dept_stats lần 1
FROM employees e
JOIN dept_stats d ON e.dept = d.dept
WHERE d.cnt > 5;                           -- dùng dept_stats lần 2
```

Nếu dùng subquery, phải **viết lại cả khối `GROUP BY` hai lần** → trùng lặp, dễ sai, khó bảo trì.

**Các vị trí dùng subquery mà CTE không tiện thay thế:**

```sql
-- (1) Subquery vô hướng (scalar) trong SELECT — trả đúng 1 giá trị
SELECT name, (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_cnt
FROM users u;

-- (2) Subquery trong WHERE với IN / EXISTS
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- (3) Subquery tương quan (correlated) — phụ thuộc từng dòng của câu ngoài
SELECT name FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

> Đây là các dạng **đặc thù** của subquery (scalar, `IN`/`EXISTS`, correlated) mà CTE không tiện dùng — CTE hợp
> nhất khi đóng vai trò một **bảng trung gian có tên**.

**Về hiệu năng:** phần lớn trường hợp, optimizer **viết lại CTE thành subquery nội tuyến** (hoặc ngược lại) nên
**cùng execution plan, tốc độ như nhau**. Ngoại lệ: PostgreSQL có `MATERIALIZED`/`NOT MATERIALIZED` để ép CTE
tính sẵn hay nội tuyến (xem câu 1.2) — khi đó mới khác biệt. **Vì vậy chọn subquery hay CTE chủ yếu là vấn đề
đọc–hiểu và bảo trì, không phải hiệu năng.**

### 🎯 Tóm tắt trả lời phỏng vấn

- **Subquery** = truy vấn lồng inline (WHERE/FROM/SELECT), không tên; **CTE** (`WITH`) = truy vấn con **đặt tên,
  khai báo tách biệt** ở đầu câu. Cả hai tạo kết quả trung gian giống nhau.
- **CTE vượt trội khi:** cần **tái dùng** cùng kết quả trung gian **nhiều lần** (subquery phải viết lại), cần **dễ
  đọc** với truy vấn phức tạp, hoặc cần **đệ quy** (`WITH RECURSIVE`).
- **Subquery hợp lý khi:** logic **ngắn, dùng 1 lần**, hoặc ở vị trí đặc thù (scalar trong `SELECT`, `IN`/`EXISTS`
  trong `WHERE`, correlated subquery).
- **Hiệu năng:** thường **như nhau** — optimizer nội tuyến cả hai → cùng plan. Ngoại lệ là PostgreSQL
  `MATERIALIZED`. Do đó **chọn theo độ rõ ràng/bảo trì**, không phải tốc độ.
- Nguyên tắc thực tế: query **đơn giản** → subquery cho gọn; query **phức tạp/nhiều tầng/tái dùng** → CTE cho dễ
  đọc, dễ sửa.

[↑ Mục lục](#mục-lục)

<a id="q9"></a>
## 9. CTE đệ quy (Recursive CTE) là gì?

### 📖 Giới thiệu kiến thức

**CTE (Common Table Expression)** là bảng tạm có tên khai báo bằng `WITH` (đã nhắc ở mục 1.2). **CTE đệ quy**
là CTE **tự tham chiếu chính nó** để lặp qua dữ liệu có **cấu trúc phân cấp / đệ quy**: cây danh mục, sơ đồ
tổ chức (nhân viên–quản lý), chuỗi bạn bè trong mạng xã hội, hoặc sinh dãy số. Trước khi có nó, việc duyệt
cây nhiều cấp phải dùng vòng lặp ở tầng ứng dụng hoặc nhiều câu query.

**Cú pháp — luôn gồm 2 phần nối bằng `UNION ALL`:**

```sql
WITH RECURSIVE ten_cte AS (
    -- (1) ANCHOR member: truy vấn nền, chạy MỘT lần, tạo dữ liệu khởi đầu
    SELECT ...
    UNION ALL
    -- (2) RECURSIVE member: tham chiếu lại ten_cte, lặp cho tới khi KHÔNG còn dòng mới
    SELECT ... FROM bang JOIN ten_cte ON ...
)
SELECT * FROM ten_cte;
```

**Cách nó chạy (rất hay hỏi):**
1. Chạy **anchor** một lần → tập kết quả ban đầu.
2. Chạy **recursive member**, dùng kết quả **bước trước** làm đầu vào → sinh dòng mới.
3. Lặp lại bước 2 cho tới khi recursive member **không trả thêm dòng nào** → dừng, gộp tất cả kết quả.

**Ví dụ 1 — duyệt cây danh mục cha–con (đi từ gốc xuống, kèm cấp độ):**

```sql
WITH RECURSIVE tree AS (
    SELECT id, parent_id, name, 1 AS level          -- anchor: các node gốc
    FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.parent_id, c.name, t.level + 1   -- recursive: con của node đã có
    FROM categories c
    JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree ORDER BY level;
```

**Ví dụ 2 — sinh dãy số 1..10 (không cần bảng):**

```sql
WITH RECURSIVE nums AS (
    SELECT 1 AS n                       -- anchor
    UNION ALL
    SELECT n + 1 FROM nums WHERE n < 10 -- recursive: dừng khi n = 10
)
SELECT n FROM nums;
```

**Chống lặp vô hạn (quan trọng):**
- Điều kiện dừng nằm ở **recursive member** (vd `WHERE n < 10`, hoặc dữ liệu cây hữu hạn). Sai điều kiện →
  chạy mãi.
- **MySQL:** có chặn cứng bằng biến `cte_max_recursion_depth` (mặc định 1000) → vượt sẽ báo lỗi thay vì treo.
- **PostgreSQL:** không giới hạn depth mặc định; với dữ liệu có **chu trình (cycle)** dùng mệnh đề `CYCLE`
  (PG 14+) hoặc tự lần vết đường đi để tránh lặp vô hạn.

**Hỗ trợ:** MySQL **8.0+** và PostgreSQL (từ lâu) đều hỗ trợ `WITH RECURSIVE`; MySQL 5.7 trở về trước **không có**.

### 🎯 Tóm tắt trả lời phỏng vấn

- **CTE đệ quy** = CTE **tự tham chiếu chính nó** (`WITH RECURSIVE`), dùng để duyệt dữ liệu **phân cấp/đệ quy**:
  cây danh mục, sơ đồ tổ chức, đồ thị, sinh dãy số.
- Cấu trúc 2 phần nối bằng `UNION ALL`: **anchor** (chạy 1 lần, dữ liệu nền) + **recursive member** (tham
  chiếu lại CTE, lặp tới khi không còn dòng mới).
- Cách chạy: anchor → lặp recursive dựa trên kết quả bước trước → dừng khi rỗng.
- **Phải có điều kiện dừng** để tránh lặp vô hạn; MySQL chặn bằng `cte_max_recursion_depth` (mặc định 1000),
  PostgreSQL dùng `CYCLE` cho dữ liệu có chu trình.
- Hỗ trợ: **MySQL 8.0+** và **PostgreSQL**; MySQL 5.7- không có.

[↑ Mục lục](#mục-lục)

<a id="q10"></a>
## 10. Transaction trong SQL là gì?

### 📖 Giới thiệu kiến thức

**Transaction (giao dịch)** là một **nhóm nhiều câu lệnh SQL** được gom lại và xử lý như **một đơn vị công
việc duy nhất (đơn vị logic)**: hoặc **tất cả thành công cùng nhau**, hoặc **tất cả bị hủy** — không có
trạng thái "làm nửa chừng".

**Tại sao cần transaction?** Nhiều thao tác nghiệp vụ phải xảy ra "cùng lúc" mới đúng. Ví dụ kinh điển là
**chuyển tiền**: trừ tiền tài khoản A **và** cộng tiền tài khoản B. Nếu trừ A xong mà server crash trước khi
cộng B → tiền "bốc hơi". Transaction đảm bảo cả hai cùng thành công hoặc cùng bị hủy (rollback).

**Các lệnh điều khiển transaction (TCL — Transaction Control Language):**

| Lệnh | Ý nghĩa |
|------|---------|
| `START TRANSACTION` / `BEGIN` | Mở một transaction mới. |
| `COMMIT` | **Xác nhận** — ghi vĩnh viễn mọi thay đổi trong transaction. |
| `ROLLBACK` | **Hủy** — trả DB về trạng thái lúc bắt đầu transaction. |
| `SAVEPOINT ten` | Đặt **điểm mốc** giữa transaction để rollback một phần. |
| `ROLLBACK TO ten` | Quay lui về savepoint (không hủy toàn bộ). |

```sql
START TRANSACTION;                                          -- mở transaction
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;   -- trừ A
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;   -- cộng B
COMMIT;                          -- cả 2 thành công → ghi vĩnh viễn
-- Nếu giữa chừng có lỗi → ROLLBACK; toàn bộ bị hủy, số dư A & B như chưa động vào
```

**SAVEPOINT — rollback một phần:**

```sql
START TRANSACTION;
  INSERT INTO orders (...) VALUES (...);
  SAVEPOINT after_order;              -- đặt mốc
  INSERT INTO order_items (...) VALUES (...);
  ROLLBACK TO after_order;            -- hủy phần order_items, GIỮ phần orders
COMMIT;
```

**Autocommit:** mặc định nhiều DB (MySQL, PostgreSQL) ở chế độ **autocommit = ON** — mỗi câu lệnh đơn lẻ là
một transaction tự động commit ngay. Khi gõ `START TRANSACTION`/`BEGIN`, ta **tạm tắt** autocommit cho tới
khi `COMMIT`/`ROLLBACK`.

**Điều làm transaction đáng tin cậy** chính là 4 tính chất **ACID** (xem mục 10.1).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Transaction** = nhóm câu lệnh SQL chạy như **một đơn vị logic**: all-or-nothing (tất cả thành công hoặc
  rollback toàn bộ).
- Dùng khi nhiều thao tác phải toàn vẹn cùng nhau (vd **chuyển tiền**: trừ A + cộng B).
- Điều khiển bằng: **`START TRANSACTION`/`BEGIN`** → **`COMMIT`** (ghi vĩnh viễn) hoặc **`ROLLBACK`** (hủy);
  **`SAVEPOINT`** để rollback một phần.
- Mặc định DB ở **autocommit** (mỗi lệnh tự commit); mở transaction để gom nhiều lệnh lại.
- Tính tin cậy của transaction đảm bảo bởi **ACID** (Atomicity, Consistency, Isolation, Durability).

[↑ Mục lục](#mục-lục)

<a id="q10-1"></a>
### 10.1. Tính chất ACID của transaction

#### 📖 Giới thiệu kiến thức

**ACID** là 4 tính chất bảo đảm transaction chạy tin cậy, viết tắt của **A**tomicity, **C**onsistency,
**I**solation, **D**urability.

**1. Atomicity — Tính nguyên tử ("tất cả hoặc không gì cả").**
Transaction là khối không thể chia nhỏ. Nếu **bất kỳ** câu lệnh nào lỗi → **ROLLBACK** toàn bộ, DB trở về
như chưa từng chạy. Không có trạng thái "làm nửa chừng" (trừ A xong nhưng chưa cộng B).

**2. Consistency — Tính nhất quán.**
Transaction đưa database từ **một trạng thái hợp lệ** sang **một trạng thái hợp lệ khác**, luôn tôn trọng
mọi **ràng buộc** (constraint, khóa ngoại, `UNIQUE`, `CHECK`, trigger...). Ví dụ: tổng tiền hệ thống trước
và sau khi chuyển phải bằng nhau; không được để số dư âm nếu có `CHECK (balance >= 0)`.

**3. Isolation — Tính cô lập.**
Nhiều transaction **chạy đồng thời** không được "giẫm chân" nhau; kết quả phải như thể chúng chạy **tuần
tự**. Mức độ cô lập điều chỉnh bằng **isolation level**, đánh đổi giữa tính đúng đắn và hiệu năng:

| Isolation level | Ngăn được | Cho phép (lỗi đồng thời) |
|-----------------|-----------|--------------------------|
| READ UNCOMMITTED | (thấp nhất) | Dirty read, Non-repeatable, Phantom |
| READ COMMITTED | Dirty read | Non-repeatable, Phantom |
| REPEATABLE READ | Dirty + Non-repeatable read | Phantom (MySQL InnoDB chặn luôn Phantom) |
| SERIALIZABLE | Tất cả (cô lập cao nhất) | — (chậm nhất) |

- **Dirty read:** đọc dữ liệu transaction khác **chưa commit**.
- **Non-repeatable read:** đọc cùng 1 dòng 2 lần ra **2 giá trị khác nhau** (do transaction khác đã sửa & commit).
- **Phantom read:** chạy lại cùng 1 truy vấn, **số dòng thay đổi** (do transaction khác thêm/xóa dòng).

> Mặc định: MySQL (InnoDB) = **REPEATABLE READ**; PostgreSQL/SQL Server = **READ COMMITTED**.

**4. Durability — Tính bền vững.**
Sau khi đã **COMMIT**, dữ liệu được ghi **vĩnh viễn** — dù ngay sau đó **mất điện / crash server** thì khi
khởi động lại dữ liệu vẫn còn. Đạt được nhờ ghi **transaction log / WAL (write-ahead log)** xuống đĩa trước
khi báo commit thành công.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **ACID** là 4 tính chất bảo đảm transaction tin cậy.
- **A — Atomicity (nguyên tử):** tất cả thành công hoặc rollback toàn bộ; không có trạng thái nửa chừng.
- **C — Consistency (nhất quán):** đưa DB từ trạng thái hợp lệ sang trạng thái hợp lệ, luôn giữ mọi ràng buộc.
- **I — Isolation (cô lập):** các transaction đồng thời không ảnh hưởng nhau; điều chỉnh bằng **isolation
  level** (READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ → SERIALIZABLE) để chặn dirty/non-repeatable/phantom read.
- **D — Durability (bền vững):** sau `COMMIT`, dữ liệu tồn tại vĩnh viễn kể cả khi crash (nhờ WAL/transaction log).
- Điều khiển bằng `START TRANSACTION` / `COMMIT` / `ROLLBACK`.

[↑ Mục lục](#mục-lục)

<a id="q10-2"></a>
### 10.2. Deadlock trong SQL là gì?

#### 📖 Giới thiệu kiến thức

**Deadlock (tắc nghẽn)** là tình trạng **hai (hoặc nhiều) transaction chờ lẫn nhau** giải phóng khóa (lock) mà
không bao giờ xảy ra → tất cả "đứng hình", không cái nào tiến được. Đây là hệ quả của **đồng thời (concurrency)**
trong transaction (liên quan chặt với tính **Isolation** ở mục 10.1). Database phải **phát hiện và giết** (rollback)
một transaction — gọi là **victim** — để phá vỡ vòng lặp.

**Ví dụ kinh điển — hai transaction chờ nhau:**

```
Transaction A                        Transaction B
----------------------------------   ----------------------------------
BEGIN;                               BEGIN;
UPDATE accounts SET bal=bal-100      -- A giữ khóa dòng id=1
  WHERE id=1;
                                    UPDATE accounts SET bal=bal+100
                                      WHERE id=2;   -- B giữ khóa dòng id=2
UPDATE accounts SET bal=bal+100      -- A cần khóa dòng id=2 → CHỜ B
  WHERE id=2;
                                    UPDATE accounts SET bal=bal-100
                                      WHERE id=1;   -- B cần khóa dòng id=1 → CHỜ A
-- 💀 DEADLOCK: A chờ B thả id=2, B chờ A thả id=1 → vòng lặp không lối thoát
```

Database phát hiện vòng lặp → **rollback 1 transaction** (vd A) và báo lỗi; transaction còn lại (B) tiếp tục
bình thường.

**Cách database xử lý deadlock:**

- **Phát hiện chủ động (deadlock detection):** DB duy trì **đồ thị chờ** (wait-for graph); khi thấy **chu trình**
  → chọn 1 transaction làm **victim** (thường là cái "rẻ nhất" để rollback) → rollback ngay → báo lỗi.
- **Lock wait timeout (dự phòng):** nếu một transaction chờ lock **quá lâu** (không hẳn là vòng lặp) → bị hủy
  khi hết thời gian chờ. Đây là cơ chế khác nhưng bổ sung cho detection.

**Nguyên nhân thường gặp:**

- Các transaction truy cập **cùng tài nguyên theo thứ tự khác nhau** (nguyên nhân số 1).
- Transaction **chạy dài**, giữ khóa lâu.
- **Isolation level cao** (càng cô lập → khóa nhiều → dễ deadlock).
- **Thiếu index** trên cột lọc của `UPDATE`/`DELETE` → DB quét và **khóa nhiều dòng** hơn, thậm chí cả bảng
  (lock escalation).

**Cách phòng tránh / giảm thiểu deadlock:**

1. **Truy cập tài nguyên theo cùng thứ tự** trong mọi transaction (luôn khóa `id=1` trước `id=2`) → triệt tiêu
   vòng lặp chờ.
2. **Giữ transaction ngắn** — làm xong việc rồi `COMMIT` nhanh, đừng giữ khóa lâu.
3. **Hạ isolation level** khi được (dùng `READ COMMITTED` thay vì `SERIALIZABLE`) → khóa ít hơn.
4. **Tạo index** cho cột trong `WHERE` của `UPDATE`/`DELETE` → khóa đúng từng dòng (row lock) thay vì quét nhiều.
5. **Bắt lỗi và retry** transaction khi gặp deadlock (vì DB đã rollback rồi, chạy lại an toàn).
6. Tránh chờ input của người dùng **bên trong** transaction (đừng để transaction "ngủ").

**Trong MySQL (InnoDB):**

- Phát hiện deadlock **tự động** mặc định; lỗi: `ERROR 1213 (40001): Deadlock found when trying to get lock;
  try restarting transaction`.
- Xem deadlock gần nhất: `SHOW ENGINE INNODB STATUS` (mục *LATEST DETECTED DEADLOCK*).
- `innodb_lock_wait_timeout` (mặc định 50s): thời gian chờ lock trước khi timeout (liên quan nhưng khác deadlock).
- Trên hệ thống rất tải, có thể tắt detection (`innodb_deadlock_detect = OFF`) và dựa vào timeout — nhưng **rủi ro**.

**Phân biệt Deadlock vs Lock Wait Timeout:**

|  | Deadlock | Lock Wait Timeout |
|---|----------|-------------------|
| Bản chất | **Vòng lặp chờ** (A chờ B, B chờ A) | Chờ **một** lock quá lâu (không vòng lặp) |
| Khi nào phát hiện | Ngay khi DB thấy chu trình | Khi hết thời gian chờ |
| Xử lý | DB chủ động giết victim | Transaction chờ bị hủy |

**Bắt và retry deadlock trong code (PHP/PDO):**

```php
$attempts = 0;
do {
    try {
        $pdo->beginTransaction();
        // ... các câu UPDATE ...
        $pdo->commit();
        break;
    } catch (\PDOException $e) {
        $pdo->rollBack();
        if ($e->errorInfo[1] === 1213 && $attempts < 3) {   // 1213 = deadlock MySQL
            $attempts++;
            usleep(100_000);                                 // chờ 100ms rồi thử lại
            continue;
        }
        throw $e;                                            // lỗi khác → ném lên
    }
} while (true);
```

> **Liên hệ ACID (mục 10.1):** deadlock là cái giá phải trả cho tính **Isolation** — càng cô lập cao thì càng
> khóa nhiều → càng dễ deadlock. Thiết kế tốt (thứ tự khóa, index, transaction ngắn) là cách cân bằng giữa
> **đúng đắn** và **ít xung đột**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Deadlock** = ≥2 transaction **chờ lẫn nhau** giải phóng lock → tất cả kẹt; DB **phát hiện và rollback 1
  victim** để phá vòng lặp.
- **Nguyên nhân chính:** các transaction truy cập cùng tài nguyên **theo thứ tự khác nhau**; transaction dài,
  isolation cao, thiếu index (khóa rộng).
- **Phòng tránh:** (1) **cùng thứ tự truy cập**; (2) **transaction ngắn**; (3) **hạ isolation** khi được;
  (4) **tạo index** cho cột UPDATE/DELETE; (5) **bắt lỗi + retry**.
- **MySQL/InnoDB:** phát hiện deadlock tự động, lỗi `1213`; xem qua `SHOW ENGINE INNODB STATUS`; khác
  **lock wait timeout** (chờ 1 lock quá lâu).
- **Thực tế:** app nên **bắt deadlock và retry** vài lần; bản chất là đánh đổi giữa Isolation và xung đột.

[↑ Mục lục](#mục-lục)

<a id="q10-3"></a>
### 10.3. Race condition là gì?

#### 📖 Giới thiệu kiến thức

**Race condition (điều kiện chạy đua)** là tình trạng **kết quả của hệ thống phụ thuộc vào thứ tự/thời điểm**
mà nhiều luồng/request thực thi — mà thứ tự đó **không thể kiểm soát** → dẫn đến **kết quả sai, không nhất
quán** hoặc **hỏng dữ liệu**. Đây là "anh em" với **deadlock** (mục 10.2): cùng là hazard của đồng thời, nhưng
bản chất khác nhau.

**Điều kiện để race condition xảy ra:**

1. **Đồng thời (concurrency):** ≥2 luồng/request cùng truy cập **chung một tài nguyên**.
2. **Thao tác không nguyên tử**, đặc biệt là mẫu **"đọc → sửa → ghi" (read–modify–write)** mà không có đồng bộ.

**Ví dụ kinh điển — "đếm lượt" gây mất cập nhật (lost update):**

```
Counter ban đầu = 0
Thread A: đọc counter (0) ──► +1 ──► ghi (1)
Thread B: đọc counter (0) ──► +1 ──► ghi (1)      // cùng lúc với A
→ Kết quả: counter = 1   ❌ SAI (đáng lẽ = 2)   → một lần tăng bị "mất"
```

Cả A và B đều **đọc giá trị 0** trước khi cái kia ghi → cùng ghi 1 → một cập nhật bị nuốt.

**Ví dụ thực tế:**

- **Oversell ghế/vé duy nhất:** 2 user cùng đặt chiếc ghế cuối → nếu không khóa, cả 2 đều "thành công".
- **Rút tiền:** đọc số dư → kiểm tra → trừ → ghi; 2 request rút cùng lúc có thể rút **vượt số dư**.
- **Like/view counter:** nhiều user like đồng thời → số đếm **sai**.
- **Khởi tạo (lazy singleton):** 2 thread cùng thấy "chưa tạo" → cùng tạo 2 instance.

**Phân biệt Race condition vs Deadlock (mục 10.2):**

|  | Race condition | Deadlock |
|---|---|---|
| Bản chất | Kết quả **sai** do thứ tự thực thi không kiểm soát | Các luồng **chờ lẫn nhau** → kẹt vĩnh viễn |
| Hệ quả | Dữ liệu **sai/mất** (silent bug) | Hệ thống **treo/đứng** |
| Có lỗi rõ ràng? | Thường **không** — chạy "thành công" nhưng sai | Có (hoặc DB giết victim, báo lỗi) |
| Khó chịu ở | **Rất khó tái hiện & debug** | Dễ nhận ra (kẹt) |

> Race condition **nguy hiểm** vì thường **im lặng**: không crash, không log lỗi, nhưng dữ liệu sai dần — và rất
> khó tái hiện ở môi trường test (chỉ nổ dưới tải thực tế).

**Cách phòng tránh / xử lý:**

1. **Khóa (locking):** mutex/semaphore ở ngôn ngữ; ở DB dùng **`SELECT ... FOR UPDATE`** (pessimistic lock)
   để chỉ 1 luồng sửa tài nguyên tại một thời điểm.
2. **Thao tác nguyên tử (atomic):** thay "đọc-sửa-ghi" bằng 1 câu nguyên tử — DB `UPDATE t SET x = x + 1`
   (DB tự khóa dòng); ngôn ngữ dùng `AtomicInteger`/`Interlocked`.
3. **Transaction + isolation phù hợp:** gói read–modify–write trong transaction; chọn isolation level đúng.
4. **Optimistic locking:** thêm cột `version`/`updated_at`; khi update kiểm tra `version` có bị đổi không →
   nếu đổi (ai đó đã sửa) thì **abort + retry** (phổ biến trong Eloquent, Hibernate).
5. **Unique/constraint ở tầng DB:** dùng `UNIQUE`/`CHECK` để DB chặn trùng/vượt ngay (vd chặn oversell).
6. **Queue serialize:** đẩy các thao tác xung đột qua **queue** (1 worker xử lý 1 key tại lúc).
7. **Tránh shared state:** thiết kế **stateless / bất biến (immutability)** → không còn gì để "đua".

**Fix "lost update" trong SQL — 3 cách:**

```sql
-- ❌ RACE CONDITION: đọc-sửa-ghi (giữa ĐỌC và GHI có thể bị request khác chen vào)
SELECT stock FROM products WHERE id = 1;            -- đọc stock = 1
-- (... request khác cũng đọc stock = 1 ở đây ...)
UPDATE products SET stock = 0 WHERE id = 1;         -- ghi → có thể OVERSELL

-- ✅ Cách 1: atomic UPDATE (DB tự khóa dòng, 1 câu)
UPDATE products SET stock = stock - 1
WHERE id = 1 AND stock > 0;                         -- trả affected rows; = 0 → hết hàng

-- ✅ Cách 2: pessimistic lock trong transaction
START TRANSACTION;
SELECT stock FROM products WHERE id = 1 FOR UPDATE; -- khóa dòng
-- kiểm tra & tính toán ở đây
UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT;

-- ✅ Cách 3: optimistic locking (cột version) — không khóa, phát hiện xung đột rồi retry
UPDATE products SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 5;                       -- nếu version đã đổi → 0 row → retry
```

> **Liên hệ Node.js:** dù **single-threaded** (event loop), Node **vẫn dính** race condition ở mức **async**:
> 2 thao tác bất đồng bộ cùng "đọc — chờ `await` — ghi" thì vẫn **interleave** được (vd 2 request cùng check
> "chưa có user" rồi cùng `create` → trùng). Nguyên tắc áp dụng **cùng** các fix trên.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Race condition** = kết quả **phụ thuộc thứ tự thực thi không kiểm soát** của nhiều luồng/request → dữ liệu
  **sai/mất**, thường do mẫu **"đọc–sửa–ghi" không nguyên tử** trên **tài nguyên dùng chung**.
- **Ví dụ kinh điển:** đếm lượt/`lost update` (2 thread cùng đọc giá trị cũ → cùng ghi → mất 1 cập nhật); thực
  tế: oversell vé, rút vượt số dư, like counter sai.
- **Khác deadlock (9.2):** race condition thường **im lặng** (không crash, dữ liệu silently sai, khó tái hiện);
  deadlock thì hệ thống **kẹt**.
- **Fix:** (1) **khóa** (mutex/`SELECT FOR UPDATE`); (2) **atomic op** (`UPDATE x=x+1`); (3) **transaction +
  isolation**; (4) **optimistic locking** (cột `version` + retry); (5) **unique/constraint**; (6) **queue**;
  (7) **tránh shared state**.
- **Lưu ý:** cả multi-thread lẫn **Node.js async** đều dính — bất cứ chỗ nào có "đọc-chờ-ghi" đều rủi ro.

[↑ Mục lục](#mục-lục)

<a id="q10-4"></a>
### 10.4. SELECT FOR UPDATE là gì?

#### 📖 Giới thiệu kiến thức

`SELECT ... FOR UPDATE` là câu lệnh **khóa bi quan (pessimistic locking)** trong SQL: ngoài việc đọc các dòng,
nó còn **đặt khóa (lock) trên các dòng đó** cho tới khi transaction kết thúc (`COMMIT`/`ROLLBACK`), ngăn các
transaction khác sửa/xóa (hoặc cũng `FOR UPDATE`) các dòng này. Đây là một trong các cách **fix race condition /
lost update** đã nhắc ở mục 10.3, dùng khi ta cần **"đọc → suy nghĩ/tính toán → ghi"** một cách an toàn.

**Cách hoạt động:**

- `SELECT ... FOR UPDATE` đọc dòng **và đặt exclusive lock** lên chúng.
- Transaction khác cố `UPDATE`/`DELETE`/`SELECT ... FOR UPDATE` trên cùng dòng sẽ **phải CHỜ** tới khi lock được
  nhả (`COMMIT`/`ROLLBACK`) hoặc hết `lock_wait_timeout`.
- → Đảm bảo **chỉ 1 transaction "nắm giữ" dòng tại một thời điểm**, loại bỏ cửa sổ race.

**Ví dụ — chống oversell khi đặt hàng:**

```sql
-- ❌ KHÔNG dùng FOR UPDATE: race condition (oversell)
START TRANSACTION;
SELECT stock FROM products WHERE id = 1;           -- đọc stock=1 (KHÔNG khóa)
-- (... request khác cũng đọc stock=1 ở đây → cùng kết luận "còn hàng" ...)
UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT;                                             -- → có thể BÁN THỪA

-- ✅ Dùng FOR UPDATE: khóa dòng, tránh race
START TRANSACTION;
SELECT stock FROM products WHERE id = 1 FOR UPDATE; -- KHÓA dòng id=1
-- (... request khác FOR UPDATE/UPDATE cùng dòng → CHỜ ...)
-- kiểm tra & tính toán yên tâm
UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT;                                             -- nhả lock → request khác tiếp tục
```

**Hai loại khóa hay đi cùng:**

| Lệnh | Loại khóa | Ý nghĩa |
|------|-----------|---------|
| `SELECT ... FOR UPDATE` | **Exclusive (X)** | Dòng bị khóa; tx khác **không** ghi/`FOR UPDATE` được (phải chờ). |
| `SELECT ... FOR SHARE` | **Shared (S)** | Tx khác cũng đọc được (`FOR SHARE`), nhưng **không ai** `UPDATE`/`DELETE` tới khi nhả. |

> **Lưu ý MVCC:** một `SELECT` thường (không lock) trên InnoDB/PostgreSQL vẫn **đọc được** dòng đang bị
> `FOR UPDATE` — nó đọc **bản cũ (snapshot)**. Chỉ các thao tác **ghi** hoặc cũng `FOR UPDATE`/`FOR SHARE` mới bị chờ.

**`NOWAIT` và `SKIP LOCKED` (PostgreSQL, MySQL 8.0+):**

- Mặc định `FOR UPDATE` sẽ **chờ** nếu dòng đã bị khóa.
- **`NOWAIT`**: dòng đang bị khóa → **báo lỗi ngay** thay vì chờ.
- **`SKIP LOCKED`**: **bỏ qua** các dòng đang khóa, chỉ trả về dòng còn lại — cực kỳ hữu ích cho **job queue**
  (lấy task chưa xử lý, bỏ qua task worker khác đang nắm).

```sql
-- Job queue: lấy 1 task 'pending', bỏ qua task đang bị worker khác lock
SELECT id FROM jobs
WHERE status = 'pending'
ORDER BY id
LIMIT 1
FOR UPDATE SKIP LOCKED;          -- nhiều worker cùng chạy, không bao giờ lấy trùng task
```

**Yêu cầu & lưu ý khi dùng:**

- **BẮT BUỘC nằm trong transaction** (`BEGIN ... COMMIT`). Ngoài transaction (autocommit), lock nhả ngay →
  `FOR UPDATE` **vô nghĩa**.
- Chỉ chạy trên storage engine có **row-level locking** (MySQL **InnoDB**; MyISAM không hỗ trợ).
- **Cần index** trên cột lọc (`WHERE`); nếu không, InnoDB có thể **khóa cả bảng** thay vì từng dòng → giảm
  đồng thời nghiêm trọng.
- **Giữ transaction ngắn** — đừng để lock tồn tại lâu (không chờ input người dùng, không gọi API/HTTP chậm
  bên trong transaction).
- **Dễ gây deadlock** nếu nhiều transaction `FOR UPDATE` theo **thứ tự khác nhau** (xem mục 10.2) → giữ thứ tự
  khóa nhất quán.

**Khi nào dùng `FOR UPDATE` (bi quan) vs Optimistic Locking (mục 10.3):**

|  | Pessimistic (`FOR UPDATE`) | Optimistic (cột `version`) |
|---|---|---|
| Triết lý | Khóa ngay, "biết chắc sẽ xung đột" | Không khóa, phát hiện xung đột **lúc ghi** |
| Phù hợp khi | Tranh chấp **cao** (tài nguyên nóng) | Tranh chấp **thấp** |
| Hiệu năng | **Giảm đồng thời** (chờ lock) | Không chờ, nhưng **retry** khi fail |
| Nguy cơ deadlock | **Cao hơn** | Thấp |

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`SELECT ... FOR UPDATE`** = đọc **+ khóa dòng (pessimistic lock)** trong transaction cho tới `COMMIT`/
  `ROLLBACK`, ngăn tx khác ghi/`FOR UPDATE` cùng dòng → **chống race condition / lost update** khi cần
  "đọc–tính–ghi".
- **Bắt buộc nằm trong transaction** (autocommit thì lock nhả ngay → vô dụng); chỉ có row-level lock trên
  **InnoDB**; **cần index** ở `WHERE` để không bị khóa cả bảng.
- **Biến thể:** `FOR SHARE` (shared lock); `NOWAIT` (lỗi thay vì chờ); `SKIP LOCKED` (bỏ qua dòng đang khóa —
  chuẩn cho **job queue**).
- **Cẩn thận:** **giảm throughput** (các tx chờ nhau) và **dễ deadlock** (giữ thứ tự khóa nhất quán, tx ngắn).
- **Chọn vs optimistic locking:** tranh chấp **cao** → `FOR UPDATE`; tranh chấp **thấp** → optimistic (cột
  `version` + retry).

[↑ Mục lục](#mục-lục)

<a id="q10-5"></a>
### 10.5. Optimistic Locking là gì?

#### 📖 Giới thiệu kiến thức

**Optimistic Locking (khóa lạc quan)** là chiến lược kiểm soát đồng thời dựa trên giả định **"xung đột hiếm
khi xảy ra"**: **không khóa** tài nguyên khi đọc, mà chỉ **phát hiện xung đột LÚC GHI**. Nếu thấy dữ liệu đã bị
ai khác sửa → **từ chối (abort) + retry**. Đây là **đối lập** của pessimistic locking (`SELECT FOR UPDATE`,
mục 10.4) và là một cách **fix race condition / lost update** (mục 10.3).

**Cơ chế — pattern "version":**

1. Thêm cột **`version`** (hoặc `updated_at`) vào bảng.
2. Khi **đọc**: lấy luôn `version` hiện tại.
3. Khi **ghi**: kiểm tra `version` có còn bằng giá trị đã đọc — trong **cùng 1 câu `UPDATE` nguyên tử**.
4. Nếu **affected rows = 1** → thành công; **= 0** → ai đó đã sửa giữa chừng → **abort + retry**.

```sql
-- Bước 1: ĐỌC (lấy cả version, KHÔNG giữ lock)
SELECT id, stock, version FROM products WHERE id = 1;
-- → stock = 10, version = 5

-- Bước 2: (có thể mất lâu — tính toán, user sửa form... không vấn đề vì không giữ lock)

-- Bước 3: GHI — bắt buộc check version TRONG CÙNG câu UPDATE (nguyên tử)
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 5;     -- version phải VẪN = 5 (chưa ai sửa)
-- affected rows = 1 → OK
-- affected rows = 0 → đã bị sửa giữa chừng → ĐỌC LẠI + TÍNH LẠI + GHI LẠI (retry)
```

> **Cực kỳ quan trọng:** việc check `version` **phải nằm trong cùng câu `UPDATE`** (dùng `WHERE ... AND version = ?`).
> Nếu tách thành "SELECT để check version rồi mới UPDATE" thì **chính khoảng đó lại là race** (giống lỗi
> read–modify–write). DB tự thực hiện check này một cách **nguyên tử**, nên optimistic locking **không cần
> transaction dài**.

**Ưu điểm:**

- **Không khóa** → không chờ, **không giảm đồng thời**, **không deadlock**.
- Hiệu năng cao khi xung đột **ít** (phần lớn request ghi thành công ngay).
- Có thể giữ "đọc–sửa–ghi" **lâu** (không giữ lock) → rất hợp cho **user sửa form lâu** trên web.

**Nhược điểm:**

- Xung đột **nhiều** → **retry liên tục** → còn tệ hơn pessimistic.
- Phải **viết retry logic** ở tầng ứng dụng.
- Cần thêm cột `version`/`updated_at`.

**Optimistic vs Pessimistic (`FOR UPDATE`, mục 10.4):**

|  | Optimistic (cột `version`) | Pessimistic (`FOR UPDATE`) |
|---|---|---|
| Giả định | Xung đột **ít** | Xung đột **nhiều** |
| Khóa tài nguyên? | **Không** | **Có** (giữ exclusive lock) |
| Khi xảy ra xung đột | Phát hiện lúc ghi → **retry** | **Chờ** lock ngay từ đầu |
| Throughput (xung đột ít) | **Cao** | Giảm (chờ nhau) |
| Throughput (xung đột nhiều) | Kém (retry liên tục) | **Ổn định hơn** |
| Deadlock risk | **Thấp** | Cao |
| Phù hợp | **Read nhiều / user-edit form dài** | **Tài nguyên nóng** (vé, tồn kho, ghế) |

**Khi nào chọn:**

- **Optimistic:** xung đột thấp, **read-heavy**, form edit lâu (phần lớn web app CRUD điển hình).
- **Pessimistic (`FOR UPDATE`):** tài nguyên **tranh chấp cao** (đặt vé/ghế, tồn kho nóng, số dư ví).

**Hiện thực ở tầng framework:**

- **Laravel/Eloquent:** không có native; dùng package `optimus/optimistic-locking`, hoặc **tự implement** (cột
  `version` + `UPDATE ... WHERE id=? AND version=?` + check affected rows), hoặc dùng cột `updated_at` làm "version".
- **Hibernate (Java):** có sẵn qua annotation **`@Version`** (tự động check + ném `OptimisticLockException`).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Optimistic Locking** = không khóa khi đọc, chỉ **phát hiện xung đột lúc ghi** qua cột **`version`** (hoặc
  `updated_at`); nếu bị sửa giữa chừng → **abort + retry**. Giả định **xung đột hiếm**.
- **Cơ chế:** `UPDATE ... SET version = version + 1 WHERE id = ? AND version = ?`; check **affected rows** (1 = OK,
  0 = retry). Bắt buộc check version **trong cùng câu UPDATE** (nguyên tử).
- **Ưu:** không khóa → **không giảm đồng thời, không deadlock**, hợp form edit lâu. **Nhược:** xung đột nhiều
  → retry liên tục; phải viết retry logic; cần cột version.
- **Chọn:** xung đột **ít / read-heavy** → optimistic; xung đột **nhiều / tài nguyên nóng** (vé, tồn kho) →
  **pessimistic `FOR UPDATE`** (mục 10.4).
- **Framework:** Laravel cần tự implement hoặc dùng package; Hibernate có `@Version` built-in.

[↑ Mục lục](#mục-lục)

<a id="q11"></a>
## 11. Xoá dữ liệu có relationship (foreign key) trong MySQL

### 📖 Giới thiệu kiến thức

Khi hai bảng có quan hệ qua **foreign key** (vd `users` 1–n `orders`, trong đó `orders.user_id` tham chiếu
`users.id`), việc xoá một record **cha** đang có record **con** tham chiếu sẽ gây **lỗi ràng buộc** nếu không
xử lý. Đây là cơ chế **toàn vẹn tham chiếu (referential integrity)** mà foreign key đảm bảo.

```sql
-- orders.user_id REFERENCES users(id)
DELETE FROM users WHERE id = 1;
-- ❌ Lỗi: Cannot delete or update a parent row: a foreign key constraint fails
```

**Có 3 cách chính** để xoá dữ liệu có relationship:

**Cách 1 — Dùng referential action khi định nghĩa foreign key (`ON DELETE ...`):** đây là cách đúng đắn nhất.
Chỉ định database tự làm gì với các record con khi cha bị xoá:

| `ON DELETE` | Hành vi khi xoá record cha |
|-------------|----------------------------|
| `RESTRICT` / `NO ACTION` | **(mặc định)** **Chặn** — báo lỗi nếu còn con tham chiếu |
| `CASCADE` | **Tự động xoá** luôn mọi record con tham chiếu |
| `SET NULL` | Giữ con, set cột FK = `NULL` (cột FK phải cho phép `NULL`) |
| `SET DEFAULT` | Set FK về giá trị mặc định (ít dùng, InnoDB hạn chế) |

```sql
-- Định nghĩa FK với ON DELETE CASCADE ngay khi tạo bảng
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
-- Giờ: DELETE FROM users WHERE id = 1;  → mọi orders của user 1 TỰ ĐỘNG bị xoá theo
```

```sql
-- Sửa FK trên bảng đã có: DROP FK cũ rồi ADD FK mới với hành động mong muốn
ALTER TABLE orders DROP FOREIGN KEY orders_ibfk_1;          -- xoá theo tên constraint
ALTER TABLE orders ADD CONSTRAINT fk_orders_user
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;
```

**Cách 2 — Xoá thủ công theo đúng thứ tự (con trước, cha sau):** khi không thể/không muốn đổi schema.

```sql
START TRANSACTION;
  DELETE FROM orders WHERE user_id = 1;   -- xoá CON trước
  DELETE FROM users  WHERE id = 1;        -- rồi mới xoá CHA
COMMIT;                                    -- bọc transaction để nguyên tử
```

**Cách 3 — Tạm tắt kiểm tra FK (`SET FOREIGN_KEY_CHECKS = 0`):** ⚠️ **NGUY HIỂM**, chỉ dùng cho
migration/import, **không** dùng trong app production.

```sql
SET FOREIGN_KEY_CHECKS = 0;               -- tắt kiểm tra FK
DELETE FROM users WHERE id = 1;           -- xoá thoải mái
SET FOREIGN_KEY_CHECKS = 1;               -- BẮT BUỘC bật lại
-- Hậu quả: orders của user 1 thành "mồ côi" (user_id trỏ tới user không còn) → hỏng toàn vẹn dữ liệu
```

**Khi nào chọn cách nào?**

- **`CASCADE`**: dữ liệu con **thuộc về hẳn** cha (orders thuộc user) → xoá cha thì xoá luôn con. ⚠️ Cẩn thận
  **xoá lan truyền** (cascading) có thể xoá rất nhiều dòng; nên có index trên cột FK.
- **`SET NULL`**: con vẫn có ý nghĩa độc lập khi mất cha (vd bài post mất author → set `author_id = NULL`,
  giữ lại bài viết).
- **`RESTRICT`**: muốn **chặn ép**, bắt ứng dụng xử lý tay để kiểm soát chặt.
- **Thủ công (Cách 2)**: logic phức tạp, cần xoá có điều kiện, hoặc không đổi được schema.
- **Soft delete (xoá mềm)**: thay vì `DELETE`, set `deleted_at` — giữ dữ liệu, không sinh rắc rối FK (phổ biến
  trong Laravel Eloquent `SoftDeletes`).

**Lưu ý riêng của MySQL:**

- Foreign key **chỉ thực sự hoạt động với storage engine InnoDB** (MyISAM không hỗ trợ FK).
- Cột FK và cột bị tham chiếu phải **cùng kiểu** (hoặc tương thích); cột bị tham chiếu phải là **key**
  (thường là `PRIMARY KEY`/`UNIQUE`).
- Nên **tạo index** cho cột FK để `CASCADE`/`SET NULL` nhanh (InnoDB tự tạo index cho FK, nhưng nên kiểm tra).
- Tránh **chuỗi CASCADE** dài nhiều tầng — xoá 1 dòng có thể lan xoá hàng loạt, khó kiểm soát.

> **Liên hệ Laravel (tầng ứng dụng):** trong migration dùng `->onDelete('cascade')` để sinh FK `ON DELETE
> CASCADE`; hoặc định nghĩa relationship rồi bắt sự kiện `deleting` để xoá con; hoặc dùng **SoftDeletes**
> (`$table->softDeletes()` + trait `SoftDeletes`) để xoá mềm. Nhưng bản chất vẫn dựa vào FK của MySQL ở tầng DB.

### 🎯 Tóm tắt trả lời phỏng vấn

- Khi 2 bảng có **foreign key**, xoá record **cha** đang có **con** tham chiếu sẽ **báo lỗi** (toàn vẹn tham chiếu).
- **3 cách xử lý:**
  1. **`ON DELETE ...`** khi tạo FK: **`CASCADE`** (tự xoá con), **`SET NULL`** (set FK NULL, giữ con),
     **`RESTRICT`** (chặn, mặc định) — cách đúng đắn nhất.
  2. **Xoá thủ công**: con trước, cha sau, bọc trong **transaction**.
  3. **`SET FOREIGN_KEY_CHECKS = 0`**: tạm tắt FK — **nguy hiểm**, để lại dữ liệu mồ côi, chỉ dùng cho migration.
- **Chọn:** con thuộc về cha → `CASCADE`; con độc lập → `SET NULL`; cần kiểm soát → `RESTRICT` + xoá tay; không
  muốn mất dữ liệu → **soft delete**.
- **Lưu ý MySQL:** FK chỉ chạy trên **InnoDB**; cột FK và cột tham chiếu **cùng kiểu**; nên **index** cột FK;
  cẩn thận chuỗi CASCADE lan truyền.

[↑ Mục lục](#mục-lục)

<a id="q12"></a>
## 12. Index trong DB: các loại, cơ chế lưu, cách query/EXPLAIN/debug

### 📖 Giới thiệu kiến thức

**Index (chỉ mục)** là **cấu trúc dữ liệu phụ** giúp DB **tìm dòng nhanh** mà không phải quét cả bảng (full
scan). Ví như **mục lục cuốn sách**: thay vì lật từng trang, tra mục lục để nhảy tới đúng trang. **Đánh đổi:**
đọc nhanh hơn nhưng **ghi chậm hơn** (mỗi `INSERT/UPDATE/DELETE` phải cập nhật index) và **tốn dung lượng**.

**1. Các loại index — phân theo MỤC ĐÍCH/ràng buộc:**

| Loại | Ý nghĩa |
|------|---------|
| **Primary index (`PRIMARY KEY`)** | Unique + not null, mỗi bảng 1 cái. Trong InnoDB **chính là clustered index**. |
| **Unique index** | Đảm bảo giá trị **không trùng**. |
| **Secondary (normal) index** | Index thường để tăng tốc tra cứu, **không** ràng buộc unique. |
| **Composite index** (nhiều cột) | Index trên nhiều cột, tuân **leftmost prefix**. |
| **Covering index** | Index chứa **đủ mọi cột** query cần → đọc thẳng từ index, không đọc bảng. |
| **Full-text index** | Tìm kiếm văn bản (`MATCH ... AGAINST`). |
| **Spatial index** | Dữ liệu không gian (R-tree; PostGIS dùng GiST — mục 1.3). |
| **Prefix index** | Index trên vài ký tự đầu của cột chuỗi dài: `INDEX(col(10))`. |

**2. Các loại index — phân theo CẤU TRÚC LƯU (data structure):**

- **B-Tree / B+Tree** (mặc định, phổ biến nhất): cân bằng, tốt cho **so sánh bằng (`=`), khoảng (range), sắp
  xếp (`ORDER BY`), prefix (`LIKE 'abc%'`)**. MySQL/InnoDB dùng **B+Tree**.
- **Hash index:** tra cứu **bằng (`=`)** cực nhanh ~O(1) nhưng **KHÔNG** hỗ trợ range/sort. MySQL MEMORY engine,
  PostgreSQL `USING hash`; InnoDB có **adaptive hash index** tự bật.
- **GIN / GiST (PostgreSQL):** **GIN** cho **JSONB/array/full-text** (nhiều giá trị trong 1 dòng), **GiST** cho
  **spatial/range** (xem mục 1.3).
- **BRIN / Bitmap:** cột **cardinality thấp** hoặc dữ liệu sắp xếp tự nhiên (BRIN của PostgreSQL rất nhỏ gọn).

**3. Clustered vs Non-clustered index (rất hay hỏi):**

- **Clustered index:** dữ liệu bảng được **sắp xếp vật lý theo index** — **node lá CHÍNH LÀ dòng dữ liệu**.
  InnoDB: **`PRIMARY KEY` là clustered index**; mỗi bảng **chỉ có 1** clustered.
- **Non-clustered (secondary):** node lá chứa **giá trị key + con trỏ tới dòng** (trong InnoDB là **giá trị
  PK**). Query qua secondary index → tìm ra PK → **tra lại clustered index** để lấy dòng đầy đủ — gọi là **"back
  to table" (bookmark lookup)**.

> Đây chính là lý do **covering index nhanh**: nếu index đã chứa đủ cột cần, **không phải back to table** (EXPLAIN
> hiện `Using index`).

**4. Cơ chế lưu — B+Tree hoạt động thế nào:**

- Cây **cân bằng, nhiều nhánh**: **node trong** chứa key dẫn đường; **node lá** chứa key + (dữ liệu hoặc con
  trỏ); **các lá nối nhau** thành danh sách liên kết → **duyệt range cực nhanh**.
- **Cây rất thấp** (vd 3–4 tầng cho hàng triệu dòng) → tìm 1 dòng chỉ mất **vài lần đọc trang** (page ~16KB) →
  nhanh hơn full scan hàng nghìn lần.
- Trong InnoDB: **clustered** B+Tree có lá = **dòng dữ liệu**; **secondary** B+Tree có lá = **giá trị PK**.

**5. Cách query / tạo & xem index:**

```sql
CREATE INDEX idx_email ON users(email);                   -- secondary index
CREATE UNIQUE INDEX idx_email ON users(email);            -- unique
CREATE INDEX idx_name_age ON users(last_name, age);       -- composite (leftmost prefix)
ALTER TABLE users ADD INDEX idx_created (created_at);
DROP INDEX idx_email ON users;
SHOW INDEX FROM users;                                     -- liệt kê index + cardinality
```

Query **tận dụng** index: đặt điều kiện ở **`WHERE`/`JOIN`/`ORDER BY`** trên cột có index, tuân **leftmost
prefix**, và **không bọc hàm** lên cột index (xem câu 13).

**6. Cách EXPLAIN (kiểm tra index CÓ được dùng không)** — chi tiết ở mục 13.1:

```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@x.com';
-- key: index thực dùng (NULL = KHÔNG dùng index) · type: ref/range (tốt) vs ALL (full scan)
-- Extra: 'Using index' = covering (rất tốt); 'Using filesort' = ORDER BY chưa có index
```

**7. Cách DEBUG index (khi index không được dùng / query chậm):**

1. **`EXPLAIN`**: `key = NULL` hoặc `type = ALL` → index không được dùng (mục 13.1).
2. **`SHOW INDEX FROM table`**: kiểm tra index tồn tại & **`Cardinality`** (số giá trị phân biệt — thấp thì
   index ít tác dụng).
3. **Vì sao index bị bỏ qua** (các nguyên nhân kinh điển):
   - **Bọc hàm** lên cột: `WHERE YEAR(created_at) = 2024` → viết lại theo range.
   - **Ép kiểu ngầm** (so cột số với chuỗi).
   - **`LIKE '%x'`** (wildcard ở đầu).
   - **Không tuân leftmost prefix** của composite index (vd chỉ lọc cột thứ 2).
   - **Cardinality thấp** → optimizer thấy full scan **rẻ hơn** dùng index.
   - **Statistics cũ** → chạy **`ANALYZE TABLE`** cập nhật.
4. **`Handler_read_*` status:** `SHOW STATUS LIKE 'Handler_read%'` — `Handler_read_rnd_next` cao = nhiều full scan.
5. **`FORCE INDEX(idx)` / `USE INDEX`**: ép dùng index khi optimizer chọn sai (hiếm khi cần).
6. **Slow query log** (mục 13) + `pt-query-digest` để tìm query cần thêm index.

**8. Khi nào KHÔNG nên index (đánh đổi):** bảng nhỏ (full scan đã nhanh), cột **cardinality thấp**, bảng **ghi
rất nhiều** (index làm chậm write), hoặc **quá nhiều index thừa** (tốn RAM/đĩa, chậm mọi thao tác ghi).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Index** = cấu trúc phụ giúp tìm dòng nhanh (như **mục lục sách**), tránh full scan; đổi lại **ghi chậm hơn +
  tốn dung lượng**.
- **Các loại — theo mục đích:** primary/unique/secondary/composite/covering/full-text/spatial/prefix. **Theo cấu
  trúc:** **B+Tree** (mặc định — tốt cho `=`, range, sort), **Hash** (chỉ `=`, ~O(1)), **GIN/GiST** (PostgreSQL:
  JSONB/array/spatial).
- **Clustered (InnoDB = `PRIMARY KEY`, lá là dữ liệu) vs Secondary (lá chứa PK → phải "back to table")** → đó là
  vì sao **covering index** nhanh.
- **Cơ chế lưu:** **B+Tree** cân bằng, **lá nối nhau** (range nhanh), **cây thấp** → vài lần đọc page là ra.
- **Query:** `CREATE INDEX`; tận dụng qua `WHERE/JOIN/ORDER BY`, tuân **leftmost prefix**, **không bọc hàm** lên cột.
- **EXPLAIN** (mục 13.1): xem `key`/`type`/`Extra` để biết index có được dùng. **Debug:** `SHOW INDEX`
  (cardinality), tránh làm index vô hiệu (hàm/ép kiểu/`LIKE '%x'`/sai leftmost), **`ANALYZE TABLE`** cập nhật
  statistics, `FORCE INDEX` khi cần.

[↑ Mục lục](#mục-lục)

<a id="q12-1"></a>
### 12.1. Phân tích B-Tree/B+Tree index và Hash index (kèm ví dụ)

#### 📖 Giới thiệu kiến thức

Đây là hai **cấu trúc dữ liệu index** phổ biến nhất (đã nhắc ở câu 12). Hiểu cấu trúc giúp biết mỗi loại tối ưu
cho kiểu query nào.

**1. B-Tree (Balanced Tree).**
- Cây tìm kiếm **cân bằng, nhiều nhánh (multi-way)**: mỗi node chứa **nhiều key đã sắp xếp** + con trỏ tới node
  con. Mọi node lá ở **cùng độ cao** → mọi tra cứu tốn số bước như nhau (**O(log n)**).
- Ở B-Tree gốc, **cả node trong lẫn lá** đều có thể chứa con trỏ tới dữ liệu.

**2. B+Tree (biến thể B-Tree — cái MySQL/InnoDB thực sự dùng).**
- **CHỈ node LÁ chứa dữ liệu / con trỏ dữ liệu**; node trong **chỉ chứa key dẫn đường (routing)** → node trong
  chứa được nhiều key hơn → **fanout cao, cây thấp hơn**.
- **Các node lá NỐI NHAU thành danh sách liên kết** → duyệt **range** (`BETWEEN`, `>`, `ORDER BY`) cực nhanh:
  tìm điểm đầu rồi **đi dọc chuỗi lá**.
- Nhờ vậy B+Tree tốt cho: **so sánh bằng (`=`), range, prefix (`LIKE 'abc%'`), `ORDER BY`** (dữ liệu đã sắp sẵn).

```
                 [ 30 | 60 ]                  ← node trong: CHỈ key dẫn đường
                /     |     \
        [10 20 30] [40 50 60] [70 80 90]      ← node lá: chứa data/con trỏ
           └───────────┴───────────┘           ← các lá NỐI NHAU (range scan nhanh)

Tìm 50: gốc (50<60 → nhánh giữa) → lá [40 50 60] → thấy 50.        (2–3 bước)
Range 40..80: tới lá chứa 40 → đi dọc chuỗi lá cho tới 80.
```

Vì fanout cao (mỗi node ~ hàng trăm key với page 16KB), cây chỉ **3–4 tầng** cho hàng triệu–tỷ dòng → tìm 1
dòng chỉ **3–4 lần đọc page**.

**3. Hash index.**
- Dùng **hàm băm**: `hash(key)` → vị trí **bucket** chứa con trỏ tới dòng. Tra cứu **bằng (`=`) ~O(1)** — cực
  nhanh, **không phụ thuộc** kích thước bảng.
- **Nhược điểm lớn:** **KHÔNG** hỗ trợ **range** (`>`, `<`, `BETWEEN`), **KHÔNG** `ORDER BY` (hash phá vỡ thứ
  tự), **KHÔNG** prefix (`LIKE 'abc%'`). Chỉ dùng cho **so sánh bằng chính xác** (và `IN`). Có thể bị **hash
  collision** (nhiều key cùng bucket → xử lý bằng chaining).
- MySQL: **MEMORY engine** dùng hash mặc định; **InnoDB** có **Adaptive Hash Index** — tự động tạo hash cho trang
  hay truy cập (không chỉ định tay). PostgreSQL: `CREATE INDEX ... USING hash`.

```
hash('an@x')   → bucket 3 → [con trỏ tới dòng an@x]
hash('binh@x') → bucket 7 → [con trỏ tới dòng binh@x]

WHERE email = 'an@x'  → hash('an@x')=3 → nhảy thẳng bucket 3 → 1 bước (O(1))
WHERE email > 'an@x'  → ❌ hash không biết thứ tự → KHÔNG dùng được index
```

**So sánh B+Tree vs Hash:**

| Tiêu chí | **B+Tree** | **Hash** |
|----------|-----------|----------|
| Tra cứu bằng (`=`) | Nhanh O(log n) | **Nhanh nhất ~O(1)** |
| Range (`>`, `<`, `BETWEEN`) | **Có** | **Không** |
| `ORDER BY` / sort | **Có** (lá đã sắp) | Không |
| Prefix (`LIKE 'abc%'`) | **Có** | Không |
| Mặc định ở | **InnoDB, hầu hết DB** | MEMORY engine, PG `USING hash` |

```sql
-- B+Tree (mặc định) — hợp cho range / sort / prefix
CREATE INDEX idx_created ON orders(created_at);   -- WHERE created_at BETWEEN ...; ORDER BY created_at

-- Hash — chỉ hợp tra cứu bằng chính xác (vd bảng cache key-value trong MEMORY)
CREATE TABLE cache (k VARCHAR(64), v TEXT, INDEX USING HASH (k)) ENGINE=MEMORY;
-- WHERE k = 'abc'  → O(1)
```

> **Kết luận:** gần như **luôn dùng B+Tree** (linh hoạt: `=`, range, sort, prefix). Chỉ chọn **Hash** khi query
> **chỉ tra cứu bằng chính xác** và cần O(1) (cache in-memory). InnoDB tự lo **Adaptive Hash** nên thường không
> cần chỉ định tay.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **B-Tree** = cây cân bằng nhiều nhánh, mọi lá cùng độ cao (**O(log n)**). **B+Tree** (cái InnoDB dùng) = biến
  thể: **chỉ lá chứa dữ liệu**, node trong chỉ key dẫn đường (**cây thấp, fanout cao**), **các lá nối nhau** →
  **range / sort / prefix cực nhanh**.
- **Hash index** = `hash(key)` → bucket, tra cứu **bằng (`=`) ~O(1)** nhanh nhất, nhưng **KHÔNG** range/sort/
  prefix (hash phá vỡ thứ tự); có thể collision. Dùng ở **MEMORY engine**, PG `USING hash`; InnoDB có **Adaptive
  Hash Index** tự động.
- **Chọn:** gần như luôn **B+Tree** (linh hoạt: `=`, range, `ORDER BY`, `LIKE 'abc%'`); **Hash** chỉ khi **chỉ
  tra cứu bằng chính xác** & cần O(1) (cache).
- B+Tree cây **3–4 tầng** cho hàng triệu dòng → tìm 1 dòng chỉ vài lần đọc page (16KB).

[↑ Mục lục](#mục-lục)

<a id="q12-2"></a>
### 12.2. Cơ chế SELECT có index nhanh hơn không index thế nào?

#### 📖 Giới thiệu kiến thức

Trả lời bằng cách **so sánh cơ chế** hai trường hợp — có index vs không index.

**Không có index → Full Table Scan (quét toàn bảng):**
- DB phải **đọc TỪNG dòng** của bảng từ đầu đến cuối, so khớp điều kiện `WHERE` với mỗi dòng.
- Độ phức tạp **O(n)** — n dòng thì đọc n dòng. Bảng 10 triệu dòng → **đọc 10 triệu dòng** dù chỉ cần 1.
- Tốn **I/O** (nạp mọi data page vào buffer pool) + **CPU** (so khớp từng dòng). EXPLAIN hiện **`type = ALL`**.

**Có index (B+Tree) → Index Seek/Scan (tra cứu có định hướng):**
- Thay vì quét, DB **duyệt cây B+Tree** (mục 12.1): gốc → node trong (theo key dẫn đường) → node lá chứa
  dòng/con trỏ. Chỉ **~3–4 lần đọc page** cho hàng triệu dòng.
- Độ phức tạp **O(log n)** — nhanh hơn O(n) hàng nghìn–triệu lần khi bảng lớn.
- Với **range**: tìm điểm đầu rồi **đi dọc chuỗi lá** (các lá nối nhau) → chỉ đọc đúng khoảng cần. EXPLAIN hiện
  **`type = ref/range/const`**.

**Vì sao nhanh hơn — 3 lý do cốt lõi:**
1. **Đọc ít dữ liệu hơn:** index dẫn **thẳng** tới dòng cần, không chạm mọi dòng → giảm **I/O** (thứ đắt nhất).
   Đây là yếu tố lớn nhất.
2. **Cấu trúc đã sắp xếp:** B+Tree lưu key theo thứ tự → tìm **định hướng (O(log n))** thay vì tuyến tính; range
   & `ORDER BY` tận dụng sẵn thứ tự (**không filesort**).
3. **Index nhỏ hơn bảng:** index chỉ chứa (các) cột được đánh + con trỏ → **nhiều entry/page** hơn, buffer pool
   hiệu quả hơn; **covering index** còn khỏi đọc bảng.

```sql
-- Bảng users 1 triệu dòng, tìm 1 email
EXPLAIN SELECT * FROM users WHERE email = 'a@x';
-- KHÔNG index: type=ALL,  rows≈1000000   (quét cả bảng — O(n))
-- CÓ index:    type=ref,  key=idx_email, rows=1   (duyệt B+Tree ~3–4 page — O(log n))
```

**Đánh đổi (vì sao không index MỌI thứ):**
- Index tăng tốc **đọc** nhưng **chậm ghi** (`INSERT/UPDATE/DELETE` phải cập nhật cả index — có thể page split,
  mục 12.6) và tốn **dung lượng/RAM** → chỉ đánh index cột thực sự cần (mục 12.4).

**Khi nào index KHÔNG nhanh hơn (optimizer bỏ qua index):**
- Query lấy **phần lớn dòng** (vd > 20–30% bảng) → full scan tuần tự **rẻ hơn** (đọc index + back-to-table nhiều
  dòng còn đắt hơn quét thẳng).
- Bảng **nhỏ** (vài trăm dòng) → full scan đã nhanh.
- Điều kiện **làm index vô hiệu** (bọc hàm, `LIKE '%x'`, ép kiểu — câu 13).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Không index → full table scan:** đọc **từng dòng** cả bảng, **O(n)** — bảng lớn thì rất chậm (EXPLAIN
  `type=ALL`).
- **Có index (B+Tree) → duyệt cây có định hướng:** ~3–4 lần đọc page, **O(log n)** — dẫn **thẳng** tới dòng cần
  (EXPLAIN `type=ref/range`).
- **Nhanh hơn vì:** (1) **đọc ít dữ liệu hơn** (giảm I/O — yếu tố lớn nhất); (2) **cấu trúc đã sắp xếp** (tìm
  định hướng + range/`ORDER BY` khỏi filesort); (3) **index nhỏ hơn bảng** (cache hiệu quả, covering khỏi đọc bảng).
- **Đánh đổi:** index nhanh đọc nhưng **chậm ghi + tốn dung lượng** → chỉ index cột cần (mục 12.4).
- **Không phải lúc nào cũng nhanh hơn:** lấy phần lớn dòng / bảng nhỏ / điều kiện làm index vô hiệu → optimizer
  chọn full scan.

[↑ Mục lục](#mục-lục)

<a id="q12-3"></a>
### 12.3. Phân biệt Clustered index vs Non-clustered index

#### 📖 Giới thiệu kiến thức

Đây là cặp khái niệm cốt lõi về **cách index lưu trữ dữ liệu** (đã nhắc ở câu 12) — hiểu nó mới giải thích được
vì sao covering index nhanh và vì sao PK nên nhỏ.

**Clustered index:**
- **Dữ liệu bảng được LƯU VẬT LÝ theo thứ tự của clustered index** — **node lá của B+Tree chính là toàn bộ dòng
  dữ liệu**.
- Vì dữ liệu chỉ có thể sắp xếp theo **một** cách → mỗi bảng chỉ có **1 clustered index**.
- **InnoDB: `PRIMARY KEY` tự động là clustered index.** Không có PK → InnoDB lấy **UNIQUE NOT NULL** đầu tiên;
  không có nữa thì tạo **hidden clustered index** (rowid 6 byte).
- Tra cứu theo clustered key → đi thẳng tới dòng (**1 lần duyệt cây**); **range** trên clustered key rất nhanh
  (dữ liệu nằm liền kề vật lý).

**Non-clustered (secondary) index:**
- Là B+Tree **riêng biệt**; node lá **KHÔNG chứa dòng dữ liệu** mà chứa **giá trị index key + con trỏ tới dòng**.
- Con trỏ đó: SQL Server dùng **RID (row locator)**; **InnoDB dùng chính giá trị `PRIMARY KEY`** (không phải địa
  chỉ vật lý).
- Mỗi bảng có thể có **nhiều** non-clustered index.

**"Back to table" (bookmark lookup) — điểm mấu chốt:**
Query qua secondary index → tìm ra **PK** ở lá → **tra tiếp clustered index bằng PK** để lấy các cột còn lại →
**2 lần duyệt cây**. Nếu index đã **covering** (chứa đủ cột `SELECT`) thì **bỏ được** bước này (xem mục 12.4).

```
Bảng users(id PK, email, name); có secondary index trên (email).

Clustered index (theo id):          Secondary index (theo email):
  id=1 → [1, 'a@x', 'An']             'a@x' → 1     ← lá chứa PK (id), KHÔNG có 'name'
  id=2 → [2, 'b@x', 'Bình']           'b@x' → 2

SELECT name FROM users WHERE email='a@x':
  (1) tra secondary(email) → id=1   (2) BACK TO TABLE: tra clustered(id=1) → 'name'   ← 2 bước
SELECT id   FROM users WHERE email='a@x':
  (1) tra secondary(email) → id=1 → XONG   ← covering, không cần back to table
```

**Bảng so sánh:**

| Tiêu chí | Clustered | Non-clustered (secondary) |
|----------|-----------|---------------------------|
| Lá B+Tree chứa | **Toàn bộ dòng dữ liệu** | **Key + con trỏ** (PK trong InnoDB) |
| Số lượng / bảng | **1** | **Nhiều** |
| Sắp xếp dữ liệu vật lý | **Có** | Không (cấu trúc riêng) |
| Tốc độ đọc điểm | Nhanh nhất (**1 lần** duyệt) | Chậm hơn (thường **2 lần** — back to table) |
| InnoDB mặc định | **`PRIMARY KEY`** | Các index còn lại |
| Tốn thêm dung lượng | Không (là chính bảng) | Có (cấu trúc riêng) |

**Hệ quả thực tế (rất hay hỏi):**
- **PK nên NHỎ & tăng dần** (vd `BIGINT AUTO_INCREMENT`): vì **mọi secondary index đều chứa giá trị PK** → PK
  lớn (UUID ngẫu nhiên, chuỗi dài) làm **mọi** secondary index **phình to & chậm**. PK ngẫu nhiên còn gây **page
  split** khi insert (không tăng dần).
- **Tránh đổi giá trị PK** — vì phải cập nhật lại toàn bộ secondary index.

**Khác biệt theo hệ CSDL:**
- **InnoDB (MySQL):** luôn có clustered index (PK); secondary trỏ bằng **PK value**.
- **MyISAM (MySQL):** **không** clustered — cả PK lẫn secondary đều non-clustered, trỏ bằng **địa chỉ vật lý**.
- **PostgreSQL:** **không** có clustered index thường trực — mọi index đều trỏ tới **heap** (`ctid`); lệnh
  `CLUSTER` chỉ sắp xếp **một lần**, không tự duy trì.
- **SQL Server:** có clustered index (mặc định theo PK); secondary dùng RID hoặc clustered key.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Clustered index:** dữ liệu bảng **lưu vật lý theo thứ tự index**, **lá = cả dòng**; mỗi bảng **1 cái**;
  InnoDB = **`PRIMARY KEY`**. Đọc theo key cực nhanh (**1 lần** duyệt cây).
- **Non-clustered (secondary):** B+Tree **riêng**, **lá = key + con trỏ** (InnoDB trỏ bằng **giá trị PK**); mỗi
  bảng **nhiều cái**.
- **Back to table:** query qua secondary → ra PK → **tra lại clustered** lấy cột còn lại (**2 lần duyệt**);
  **covering index** né được bước này → nhanh hơn (mục 12.4).
- **Hệ quả:** vì secondary chứa PK → **PK nên nhỏ & tăng dần** (tránh UUID ngẫu nhiên/chuỗi dài làm phình mọi
  secondary index & gây page split); **tránh đổi PK**.
- **Theo hệ:** InnoDB & SQL Server có clustered; **MyISAM & PostgreSQL không** có clustered thường trực (trỏ địa
  chỉ/heap).

[↑ Mục lục](#mục-lục)

<a id="q12-4"></a>
### 12.4. Cách chọn cột nào để đánh index

#### 📖 Giới thiệu kiến thức

Index **không miễn phí** (chậm ghi + tốn dung lượng — câu 12), nên chỉ đánh index cho cột **thực sự hưởng lợi**.
Nguyên tắc chọn:

**1. Cột hay xuất hiện trong `WHERE` (điều kiện lọc).** Ứng viên **số 1** — index giúp lọc nhanh, tránh full scan.

**2. Cột dùng trong `JOIN` (khoá ngoại).** JOIN trên cột có index cực nhanh; **foreign key nên có index** (InnoDB
thường tự tạo cho FK).

**3. Cột trong `ORDER BY` / `GROUP BY`.** Index đã sắp sẵn theo thứ tự → tránh **`Using filesort`/`Using
temporary`** (xem mục 13.1).

**4. Cột có CARDINALITY (độ chọn lọc) CAO.** *Cardinality* = số **giá trị phân biệt**. Cột như `email`,
`user_id` (nhiều giá trị unique) → index **rất hiệu quả**. Cột như `gender`, `boolean`, `status` (ít giá trị) →
index **ít tác dụng** (optimizer có thể vẫn full scan). Công thức *selectivity* = `distinct / total`, càng gần
**1** càng đáng index.

**5. Composite index — THỨ TỰ cột rất quan trọng (leftmost prefix).**

- Đặt cột lọc **bằng (`=`)** và **cardinality cao** lên **TRƯỚC**.
- Cột dùng **range** (`>`, `<`, `BETWEEN`) đặt **SAU** — vì sau một cột range, các cột tiếp theo trong index
  **không** được dùng để lọc tối ưu nữa.
- Quy tắc phổ biến **ESR: Equality → Sort → Range** (bằng → sắp xếp → khoảng).
- Thứ tự phải **khớp cách query lọc** (leftmost prefix — câu 13).

```sql
-- Query hay chạy:
SELECT id, name FROM users
WHERE status = 'active' AND created_at > '2024-01-01'
ORDER BY created_at;

-- ✔ Composite: equality (status) TRƯỚC, range/sort (created_at) SAU
CREATE INDEX idx_status_created ON users(status, created_at);
```

**6. Ưu tiên COVERING INDEX cho query "nóng".** Nếu một query chạy **rất nhiều**, gộp đủ cột nó cần (cột lọc +
cột `SELECT`) vào một index → đọc thẳng từ index, không **back to table** (câu 12).

**Khi nào KHÔNG nên đánh index (tránh index thừa):**

- Cột **cardinality thấp** (`gender`, boolean, `status` ít giá trị).
- Cột **hiếm khi** dùng trong query.
- Bảng **nhỏ** (full scan đã nhanh).
- Cột **update rất thường xuyên** (mỗi update phải sửa index).
- Bảng **ghi nhiều hơn đọc** (write-heavy) → cân nhắc kỹ.
- **Index trùng/thừa:** đã có composite `(a, b)` thì index `(a)` riêng là **thừa** (leftmost prefix đã cover).

**Cân nhắc thêm:**
- **Prefix index** cho cột chuỗi dài: `INDEX(col(20))` — tiết kiệm dung lượng, chọn độ dài đủ selectivity.
- Đừng đánh index **mọi cột** — mỗi index làm **chậm write** + tốn RAM (buffer pool).
- **Chọn theo query THỰC TẾ, không đoán:** dựa vào **slow query log + `EXPLAIN`** (câu 13, mục 13.1); xoá index
  không dùng (`sys.schema_unused_indexes` trong MySQL).

**Quy trình thực tế:** (1) tìm query **chạy nhiều & chậm** (slow log) → (2) `EXPLAIN` xem `type=ALL`/`key=NULL`
→ (3) đánh index cho cột `WHERE`/`JOIN`/`ORDER BY`, cardinality cao trước → (4) kiểm lại bằng `EXPLAIN`, dọn
index thừa.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Đánh index cho cột:** (1) hay trong **`WHERE`**, (2) dùng **`JOIN`/FK**, (3) trong **`ORDER BY`/`GROUP BY`**,
  (4) **cardinality cao** (nhiều giá trị phân biệt: `email`, `user_id`).
- **Composite index — thứ tự cột:** **equality (`=`) & cardinality cao TRƯỚC**, **range (`>`,`<`,`BETWEEN`) SAU**
  (quy tắc **ESR: Equality → Sort → Range**); phải khớp **leftmost prefix**.
- **Ưu tiên covering index** cho query nóng (gộp đủ cột cần → không back to table).
- **KHÔNG index:** cột **cardinality thấp** (gender/boolean/status), hiếm query, bảng nhỏ, cột update liên tục,
  hoặc **index trùng/thừa** (composite `(a,b)` đã cover `(a)`).
- **Nguyên tắc:** chọn index theo **query thực tế** (slow log + `EXPLAIN` — câu 13 & mục 13.1), **không đoán**;
  index nhiều = **chậm ghi + tốn RAM**.

[↑ Mục lục](#mục-lục)

<a id="q12-5"></a>
### 12.5. Tối ưu index cho câu query có nhiều điều kiện WHERE

#### 📖 Giới thiệu kiến thức

Khi query có **nhiều điều kiện `WHERE`** (nhiều `AND`, có `OR`, mix equality + range), thiết kế index đúng
quyết định query nhanh hay chậm. Đây là phần mở rộng của "cách chọn cột" (mục 12.4) cho tình huống nhiều điều kiện.

**1. Một COMPOSITE index thay vì nhiều index đơn.**
Query `WHERE a=? AND b=? AND c=?` → **một composite index `(a, b, c)`** thường tốt hơn 3 index đơn lẻ, vì lọc
được **cả 3 cột trong một lần duyệt cây**; index đơn chỉ lọc 1 cột rồi phải back-to-table/lọc phần còn lại.

**2. Thứ tự cột — quy tắc ESR (Equality → Sort → Range).**
- **Equality (`=`) TRƯỚC:** cột lọc bằng đặt đầu (thu hẹp nhanh nhất).
- **Sort (`ORDER BY`) GIỮA:** để index cấp sẵn thứ tự → tránh `Using filesort`.
- **Range (`>`, `<`, `BETWEEN`, `LIKE 'x%'`) SAU CÙNG:** vì **sau một cột range, các cột tiếp theo KHÔNG dùng
  được index để lọc** (chỉ để cover).

```sql
SELECT * FROM orders
WHERE status = 'paid' AND amount > 100
ORDER BY created_at;

-- ESR: Equality(status) → Sort(created_at) → Range(amount)
CREATE INDEX idx_orders ON orders(status, created_at, amount);
```
> Nếu đặt range trước — `(status, amount, created_at)` — thì sau `amount` (range), `created_at` **không** dùng
> index để sắp xếp được → vẫn **filesort**.

**3. Leftmost prefix — index `(a, b, c)` phục vụ được:**
- `WHERE a` ✓ · `WHERE a AND b` ✓ · `WHERE a AND b AND c` ✓
- `WHERE b` ✗ · `WHERE b AND c` ✗ · `WHERE a AND c` → chỉ dùng được `a` (bỏ `c` vì thiếu `b`).
→ Đặt cột **hay lọc nhất / cardinality cao** lên trước; thứ tự phải **khớp cách query lọc**.

**4. Điều kiện `OR` — khó dùng index hơn `AND`.**
`WHERE a=1 OR b=2` trên các cột khác nhau → composite `(a, b)` **không** giúp (leftmost chỉ dùng `a`). Xử lý:
- **Index riêng cho mỗi cột** → MySQL có thể **index merge (union)**; hoặc
- Viết lại thành **`UNION`**, mỗi nhánh dùng 1 index:

```sql
SELECT * FROM t WHERE a = 1
UNION
SELECT * FROM t WHERE b = 2;
```

**5. Index Merge (khi không có composite phù hợp):** optimizer có thể **gộp kết quả từ 2+ index đơn** —
**intersection** cho `AND`, **union** cho `OR` (EXPLAIN Extra hiện `Using intersect/union`). Tiện, nhưng thường
**kém hơn một composite index đúng**.

**6. Covering index cho query nóng:** thêm cả cột `SELECT` vào composite index → `Using index`, không
back-to-table (mục 12.3).

**7. Kiểm chứng bằng `EXPLAIN`** (mục 13.1): xem `key` (index nào được dùng), **`key_len`** (dùng bao nhiêu cột
của composite), `Extra` (`Using index` / `Using where` / `Using filesort`).

```sql
SELECT id, name FROM users
WHERE city = 'HN' AND age BETWEEN 20 AND 30
ORDER BY created_at
LIMIT 20;

-- ESR: Equality(city) → Sort(created_at) → Range(age)
CREATE INDEX idx_users ON users(city, created_at, age);
```

> **Lưu ý:** đừng tạo composite index cho **mọi** tổ hợp — nhiều index thừa làm **chậm ghi**. Thiết kế theo
> **query hay chạy nhất**; một composite index khéo (leftmost) phục vụ được nhiều query.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Query nhiều điều kiện `WHERE` → ưu tiên **một composite index** thay vì nhiều index đơn (lọc nhiều cột trong 1
  lần duyệt cây).
- **Thứ tự cột theo ESR: Equality (`=`) → Sort (`ORDER BY`) → Range (`>`,`<`,`BETWEEN`,`LIKE 'x%'`)**; range đặt
  cuối vì sau nó các cột không lọc được nữa. Tuân **leftmost prefix**; cột **selective/cardinality cao** lên trước.
- **`OR`** khó dùng index: tách **`UNION`** hoặc để optimizer **index merge (union)**; **`AND`** có thể **index
  merge (intersect)** khi thiếu composite.
- **Covering index** (thêm cột `SELECT`) → `Using index`, không back-to-table (mục 12.3).
- **Kiểm chứng bằng `EXPLAIN`** (`key`, `key_len`, `Extra` — mục 13.1); **đừng tạo index thừa** cho mọi tổ hợp,
  thiết kế theo query hay chạy.

[↑ Mục lục](#mục-lục)

<a id="q12-6"></a>
### 12.6. Page split trong index là gì?

#### 📖 Giới thiệu kiến thức

Index (B+Tree) lưu dữ liệu trong các **page (trang)** — đơn vị lưu trữ cơ bản của InnoDB (mặc định **16KB**).
Mỗi page chứa **nhiều entry đã sắp xếp**. **Page split** xảy ra khi một page đã **đầy** mà cần chèn thêm entry
vào **giữa** (để giữ đúng thứ tự).

**Page split là gì:**
- Khi page đầy và phải chèn entry vào **đúng vị trí giữa**, InnoDB **tách page đó thành 2**: tạo page mới,
  chuyển khoảng **nửa số entry** sang page mới, chèn entry vào page phù hợp, rồi **cập nhật con trỏ ở page cha**.
- Kết quả: 1 page đầy → **2 page mỗi cái ~nửa đầy**.

```
Page đầy [10, 20, 30, 40]  (giả sử tối đa 4 entry)

▶ Insert 25 (vào GIỮA) → PAGE SPLIT:
    Page A [10, 20]      Page B [25, 30, 40]     ← tách đôi, mỗi page ~nửa đầy

▶ Insert 50 (LỚN NHẤT, append cuối) → KHÔNG split:
    Page đầy [10,20,30,40]  →  tạo Page mới [50]  ← chỉ thêm page ở cuối
```

**Vì sao page split tốn kém:**
- Thêm **I/O** (ghi page mới + sửa page cũ + sửa page cha).
- **Phân mảnh (fragmentation):** các page giờ chỉ ~50% đầy → **tốn dung lượng**, đọc kém hiệu quả (nhiều page hơn
  để chứa cùng lượng dữ liệu), **buffer pool** chứa được ít dữ liệu hữu ích hơn.
- Có thể **lan lên tầng trên** (page cha đầy → cũng split).

**Khi nào xảy ra page split:**
- **Insert khoá KHÔNG tăng dần** (chèn vào giữa): vd **PK ngẫu nhiên (UUIDv4)** hoặc insert thứ tự lộn xộn →
  thường xuyên phải chèn vào page đã đầy ở giữa → **split nhiều**.
- **Update làm entry to ra** (vd cập nhật cột `VARCHAR` dài hơn) khiến page không đủ chỗ.

**Vì sao PK tăng dần (`AUTO_INCREMENT`) TRÁNH được page split:**
- Khoá insert **luôn lớn nhất** → luôn **append vào cuối** page cuối cùng. Page cuối đầy → **tạo page mới ở cuối**
  (không phải tách đôi page giữa) → gần như **không split**, page đầy ~100%.
- Đây chính là lý do **PK nên nhỏ & tăng dần** (mục 12.3) — dùng `BIGINT AUTO_INCREMENT` hoặc UUIDv7 sortable
  thay vì UUIDv4 ngẫu nhiên.

> Ngược lại với split là **merge:** khi delete nhiều làm page quá rỗng, InnoDB có thể **gộp** các page lại.

**Cách giảm page split:**
- Dùng **PK tăng dần**, tránh UUID ngẫu nhiên làm PK/clustered key.
- **Bulk load theo thứ tự khoá** (dữ liệu đã sort — mục 15).
- **`innodb_fill_factor`** (PostgreSQL: `fillfactor`): chừa sẵn chỗ trống trong page để insert sau ít split (đánh
  đổi dung lượng).
- Định kỳ **`OPTIMIZE TABLE`** (MySQL) / **`REINDEX`/`VACUUM`** (PostgreSQL) để gỡ phân mảnh, dồn page.
- Tránh update làm dòng phình to (thiết kế cột hợp lý; cân nhắc cột lớn để riêng).

**Theo dõi phân mảnh:** `SHOW TABLE STATUS` (cột **`Data_free`** lớn = nhiều khoảng trống), hoặc InnoDB metrics
trong `performance_schema`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Page** = đơn vị lưu của index/dữ liệu (InnoDB **16KB**), chứa entry đã sắp xếp. **Page split** = page **đầy**
  mà phải chèn entry vào **giữa** → InnoDB **tách page thành 2** (mỗi cái ~nửa đầy).
- **Tốn kém:** thêm I/O + **phân mảnh** (page nửa đầy → tốn dung lượng, đọc kém, buffer pool kém hiệu quả); có
  thể lan lên page cha.
- **Xảy ra khi:** insert khoá **không tăng dần** (PK ngẫu nhiên/UUIDv4), hoặc update làm entry to ra.
- **PK tăng dần (`AUTO_INCREMENT`) tránh split** vì luôn **append cuối** (page đầy → thêm page mới ở cuối, không
  tách giữa) → lý do **PK nên nhỏ & tăng dần** (mục 12.3).
- **Giảm split:** PK tăng dần, insert **theo thứ tự khoá** (mục 15), chỉnh `innodb_fill_factor`/`fillfactor`,
  định kỳ **`OPTIMIZE TABLE`/`REINDEX`**.

[↑ Mục lục](#mục-lục)

<a id="q12-7"></a>
### 12.7. Cách giảm phân mảnh (fragmentation) index trong MySQL

#### 📖 Giới thiệu kiến thức

**Phân mảnh (fragmentation)** là tình trạng dữ liệu/index nằm **rải rác**, các page **không đầy** hoặc **không
liền kề vật lý** → tốn dung lượng, đọc chậm (nhiều I/O). Nguyên nhân chính: **page split** (mục 12.6),
**`DELETE`/`UPDATE`** để lại khoảng trống, insert khoá ngẫu nhiên.

**Ba loại phân mảnh:**
- **Internal:** page chỉ đầy một phần (do split, delete) → phí không gian **trong** page.
- **External:** các page logic liền nhau nhưng nằm **rải rác vật lý** trên đĩa → đọc range phải nhảy nhiều chỗ.
- **Row:** dòng bị update to ra, phải tách/di chuyển.

**1. Phát hiện phân mảnh:**

```sql
-- Data_free = khoảng trống chưa dùng (dấu hiệu phân mảnh)
SHOW TABLE STATUS LIKE 'users';
SELECT table_name, data_length, index_length, data_free
FROM information_schema.tables WHERE table_schema = 'mydb';
```
`Data_free` **lớn** so với `data_length` → phân mảnh nhiều.

**2. Gỡ phân mảnh:**

```sql
-- (a) OPTIMIZE TABLE — cách chính
OPTIMIZE TABLE users;
-- (b) Tương đương: null-rebuild
ALTER TABLE users ENGINE=InnoDB;
```
- Với **InnoDB**, `OPTIMIZE TABLE` thực chất **rebuild bảng** (tạo lại bảng + index gọn gàng, dồn page) rồi
  `ANALYZE`. **Lưu ý:** tốn I/O, cần **~gấp đôi dung lượng** tạm thời, và có khoá (online DDL từ 5.6+ giảm ảnh
  hưởng) → **chạy lúc tải thấp**.
- Bảng **rất lớn / không được downtime** → dùng **`pt-online-schema-change`** (Percona) hoặc **`gh-ost`** để
  rebuild **không khoá**.

**3. Phòng ngừa (tốt hơn chữa):**
- **PK tăng dần** (`AUTO_INCREMENT`) → ít page split (mục 12.3, 12.6).
- **`innodb_fill_factor`**: chừa sẵn chỗ trống trong page nếu hay insert vào giữa.
- Tránh **`DELETE` hàng loạt** rồi để đó → với dữ liệu theo thời gian, dùng **partitioning + `DROP PARTITION`**
  (mục 14) thay vì `DELETE`.
- **Bulk insert theo thứ tự khoá** (mục 15).
- Giữ **`innodb_file_per_table = ON`** (mặc định) để `OPTIMIZE` **trả dung lượng về OS**.

> **Phân biệt `OPTIMIZE` vs `ANALYZE` (hay nhầm):** `OPTIMIZE TABLE` = **gỡ phân mảnh** (rebuild dữ liệu +
> index), tốn kém. **`ANALYZE TABLE`** = chỉ **cập nhật statistics** cho optimizer (nhanh, **KHÔNG** gỡ phân mảnh).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Phân mảnh** = page không đầy / rải rác (do **page split** mục 12.6, `DELETE`/`UPDATE` để lại khoảng trống) →
  tốn dung lượng, đọc chậm. Phát hiện qua **`Data_free`** (`SHOW TABLE STATUS`).
- **Gỡ:** **`OPTIMIZE TABLE`** (InnoDB = rebuild bảng+index, dồn page — nhưng **tốn I/O/khoá**, chạy lúc rảnh)
  hoặc `ALTER TABLE ... ENGINE=InnoDB`. Bảng lớn không downtime → **`pt-online-schema-change`/`gh-ost`**.
- **Phòng ngừa:** **PK tăng dần** (ít split), `innodb_fill_factor`, tránh `DELETE` hàng loạt (dùng **`DROP
  PARTITION`** — mục 14), bulk insert **theo thứ tự khoá** (mục 15), `innodb_file_per_table`.
- **Phân biệt:** `OPTIMIZE` **gỡ phân mảnh**; **`ANALYZE TABLE` chỉ cập nhật statistics** (không gỡ phân mảnh).

[↑ Mục lục](#mục-lục)

<a id="q12-8"></a>
### 12.8. B+Tree index được lưu ở đâu?

#### 📖 Giới thiệu kiến thức

B+Tree index (và cả dữ liệu bảng trong InnoDB) được lưu ở **2 nơi**: **trên đĩa** (bền vững) và **cache trong
RAM** (buffer pool).

**1. Trên đĩa — tablespace / file dữ liệu.**
- InnoDB lưu index & data trong **tablespace**. Với **`innodb_file_per_table = ON`** (mặc định), **mỗi bảng có
  1 file `.ibd`** riêng (vd `users.ibd`) chứa **cả clustered index (dữ liệu) lẫn các secondary index** của bảng
  đó. Nếu tắt → dồn vào **shared tablespace** `ibdata1`.
- Bên trong file: dữ liệu tổ chức thành các **page 16KB** (mục 12.6); B+Tree là các page liên kết nhau (page
  gốc → page trong → page lá).
- **Clustered index** lưu **cả dòng** ở lá; **secondary index** lưu **key + PK** ở lá (mục 12.3).

**2. Trong RAM — Buffer Pool.**
- Đĩa chậm → InnoDB **cache các page hay dùng** (data + index) vào **buffer pool** (vùng RAM, cấu hình
  **`innodb_buffer_pool_size`**).
- Query: page cần đã ở buffer pool → đọc từ **RAM** (nhanh); chưa có → đọc từ **đĩa** vào buffer pool rồi dùng.
- Buffer pool dùng **LRU** để giữ dữ liệu "nóng" trong RAM. Đây là lý do **RAM lớn giúp DB nhanh** (giữ được
  nhiều index/data → ít chạm đĩa) — liên hệ scale dọc (`Backend/devops.md` câu 2).

**3. Ghi bền vững (liên quan durability).**
Thay đổi index/data trước tiên vào buffer pool (page **"dirty"**) + ghi **redo log (WAL)** để không mất khi
crash; page dirty được **flush** xuống file `.ibd` sau (background) — liên hệ ACID **Durability** (mục 10.1).

```
[RAM]  Buffer Pool  ← cache page nóng (data + index B+Tree), quản theo LRU
   ↕ đọc / ghi
[Disk] users.ibd (file-per-table)
        └─ các page 16KB: clustered B+Tree (= dữ liệu) + secondary B+Tree
       + redo log (WAL) để đảm bảo durability
```

> **PostgreSQL (đối chiếu):** index lưu trong **file riêng** dưới thư mục data (`base/<db>/<relfilenode>`), cache
> trong **`shared_buffers`**; dữ liệu bảng ở **heap** riêng (không clustered — mục 12.3).

#### 🎯 Tóm tắt trả lời phỏng vấn

- B+Tree index lưu ở **2 nơi**: **trên đĩa** (bền vững) và **cache trong RAM (buffer pool)**.
- **Trên đĩa:** InnoDB lưu trong **tablespace** — mặc định mỗi bảng 1 file **`.ibd`** (file-per-table) chứa **cả
  clustered index (dữ liệu) + secondary index**; tổ chức thành **page 16KB**.
- **Trong RAM:** **buffer pool** (`innodb_buffer_pool_size`) cache page hay dùng theo **LRU** → đọc từ RAM thay
  vì đĩa. **RAM lớn = ít chạm đĩa = nhanh.**
- **Durability:** thay đổi vào buffer pool (dirty page) + **redo log (WAL)** trước, flush xuống `.ibd` sau (mục 10.1).
- **PostgreSQL:** index ở file riêng, cache trong `shared_buffers`; data ở **heap** (không clustered).

[↑ Mục lục](#mục-lục)

<a id="q12-9"></a>
### 12.9. So sánh PK BIGINT AUTO_INCREMENT vs PK UUID

#### 📖 Giới thiệu kiến thức

Chọn kiểu PK ảnh hưởng **lớn** tới hiệu năng — nhất là InnoDB, nơi **PK là clustered index** và **mọi secondary
index đều chứa giá trị PK** (mục 12.3). Vì vậy **kích thước & tính tuần tự của PK** tác động tới toàn bộ index.

**1. `BIGINT AUTO_INCREMENT`.**
- Số nguyên **8 byte**, tự **tăng tuần tự**.
- **Nhỏ gọn** → clustered index & mọi secondary index nhỏ. **Tăng dần** → insert luôn **append cuối** → **ít
  page split**, ít phân mảnh (mục 12.6).
- **Nhược:** **đoán được / lộ thông tin** (id tuần tự → dễ enumerate qua API, lộ số lượng record); khó **merge**
  dữ liệu từ nhiều nguồn/DB (trùng id); phải hỏi DB mới có id (không sinh trước ở client).

**2. UUID (128-bit).**
- **UUIDv4 (ngẫu nhiên):**
  - **Ưu:** sinh được ở **client** (không cần DB), **không đoán được** (an toàn), **duy nhất toàn cục** (dễ merge
    nhiều DB / sharding, không lộ số lượng).
  - **Nhược (nghiêm trọng khi làm clustered PK):** **ngẫu nhiên → insert vào giữa B+Tree → page split & phân
    mảnh nhiều** (mục 12.6); **to** (16 byte dạng `BINARY`, tới 36 byte nếu lưu `CHAR`) → **mọi secondary index
    phình to**, tốn RAM/đĩa, cache kém.
- **UUIDv7 / ULID (sortable, time-ordered):** phần đầu là timestamp → **tăng dần theo thời gian** → **ít page
  split** như auto_increment mà **vẫn giữ** ưu điểm UUID (sinh client, global unique). Là lựa chọn **hiện đại
  được khuyến nghị** nếu cần UUID.

> **Lưu UUID hiệu quả trong MySQL:** đừng dùng `CHAR(36)` (36 byte); dùng **`BINARY(16)`** qua
> `UUID_TO_BIN()`/`BIN_TO_UUID()` (với UUIDv1 thêm cờ swap để sortable).

**Bảng so sánh:**

| Tiêu chí | BIGINT AUTO_INCREMENT | UUIDv4 (random) | UUIDv7/ULID (sortable) |
|----------|-----------------------|-----------------|------------------------|
| Kích thước | **8 byte** | 16 (BINARY) / 36 (CHAR) | 16 byte |
| Tuần tự (ít page split) | **Có** | **Không** (split nhiều) | **Có** |
| Sinh ở client | Không | **Có** | **Có** |
| Global unique / merge | Kém | **Tốt** | **Tốt** |
| Đoán được / lộ số lượng | **Có** (rủi ro) | Không | Một phần (timestamp) |
| Ảnh hưởng secondary index | **Nhỏ nhất** | Lớn (phình) | Trung bình |

**Giải pháp kết hợp (rất hay dùng):** dùng **`BIGINT AUTO_INCREMENT` làm PK nội bộ (clustered)** + thêm cột
**UUID (public id) có `UNIQUE` index** để lộ ra API. Vừa **nhanh** (PK nhỏ, tuần tự) vừa **an toàn** (không lộ
id tuần tự ra ngoài).

**Khi nào chọn gì:**
- **BIGINT AUTO_INCREMENT:** mặc định, hệ thống 1 DB, ưu tiên hiệu năng, không cần sinh id ở client.
- **UUID:** phân tán/sharding/multi-master, cần sinh id offline/client, không muốn lộ số lượng/tuần tự. Nếu buộc
  dùng UUID → chọn **v7/ULID + `BINARY(16)`**, **tránh v4 làm clustered PK**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Trong InnoDB, **PK là clustered index và mọi secondary index đều chứa PK** (mục 12.3) → **kích thước & tính
  tuần tự của PK ảnh hưởng lớn** tới toàn bộ index.
- **BIGINT AUTO_INCREMENT:** **8 byte, tăng dần** → nhỏ gọn, **ít page split** (mục 12.6), secondary index nhỏ,
  nhanh. Nhược: **đoán được / lộ số lượng**, khó merge nhiều DB.
- **UUIDv4 (ngẫu nhiên):** sinh ở client, global unique, an toàn — nhưng **to** + **ngẫu nhiên → page split &
  phân mảnh nhiều**, phình secondary index → **hại hiệu năng khi làm clustered PK**.
- **UUIDv7/ULID (sortable):** khắc phục — tăng dần theo thời gian nên **ít page split** như auto_increment mà vẫn
  global unique/sinh client. Lưu **`BINARY(16)`** (`UUID_TO_BIN`), đừng `CHAR(36)`.
- **Hay dùng:** BIGINT auto_increment làm **PK nội bộ** + cột **UUID public** (`UNIQUE`) lộ ra API — vừa nhanh
  vừa an toàn.

[↑ Mục lục](#mục-lục)

<a id="q12-10"></a>
### 12.10. Xử lý khi index quá lớn không đủ RAM

#### 📖 Giới thiệu kiến thức

Buffer pool (mục 12.8) cache index/data trong RAM. Khi **tổng index + working set > RAM**, buffer pool **không
giữ hết** → nhiều truy vấn phải **đọc từ đĩa** (buffer pool *miss*) → **chậm hẳn** (disk I/O đắt). Đây là điểm
nghẽn phổ biến khi dữ liệu lớn dần.

**Dấu hiệu & cách kiểm tra:** query chậm dần khi dữ liệu lớn lên, disk I/O tăng, **buffer pool hit rate** giảm.

```sql
SHOW ENGINE INNODB STATUS;   -- xem 'Buffer pool hit rate'
-- hoặc so sánh: Innodb_buffer_pool_reads (đọc từ ĐĨA) / Innodb_buffer_pool_read_requests (tổng yêu cầu)
```
Hit rate thấp (< ~99%) = nhiều miss (phải đọc đĩa).

**Cách xử lý (xếp rẻ → đắt):**

**1. Giảm KÍCH THƯỚC index (thường hiệu quả nhất & rẻ nhất).**
- **Xoá index thừa/không dùng** (`sys.schema_unused_indexes`) → bớt index phải cache.
- **PK nhỏ** (`BIGINT` thay UUID/`CHAR` — mục 12.9): vì **mọi secondary index chứa PK** → PK nhỏ làm **mọi**
  index nhỏ.
- **Prefix index** cho cột chuỗi dài (`INDEX(col(20))`); dùng **kiểu dữ liệu gọn** (`INT` thay `BIGINT` nếu đủ).
- **Gỡ phân mảnh** (`OPTIMIZE TABLE` — mục 12.7) → index chặt hơn, chiếm ít page hơn.

**2. Tăng buffer pool / RAM (scale dọc — giải pháp trực tiếp).**
- Tăng **`innodb_buffer_pool_size`** (khuyến nghị ~**70–80% RAM** cho DB server chuyên dụng); nâng RAM máy. Có
  trần & đắt (scale dọc — `Backend/devops.md` câu 2).

**3. Chỉ cần "working set" nóng vừa RAM (không cần toàn bộ index).**
- Thực tế chỉ cần **dữ liệu/index hay truy cập (hot data)** nằm trong RAM; buffer pool **LRU** tự giữ phần nóng.
  Nếu hot set vẫn quá lớn → dùng các hướng dưới.

**4. Chia nhỏ dữ liệu — Partitioning / Sharding.**
- **Partitioning** (câu 14): chia bảng lớn thành partition → mỗi partition + index nhỏ hơn; truy vấn có
  **partition pruning** chỉ chạm phần liên quan → working set nhỏ hơn.
- **Sharding:** chia dữ liệu ra **nhiều server**, mỗi server giữ 1 phần + index của phần đó trong RAM của nó →
  tổng RAM = tổng nhiều máy (scale ngang — câu 16).

**5. Archive dữ liệu cũ.** Chuyển dữ liệu cũ ít dùng sang **bảng archive / cold storage** → bảng nóng + index
nhỏ lại, vừa RAM.

**6. Dùng SSD/NVMe.** Nếu buộc phải đọc đĩa (không đủ RAM), **SSD/NVMe** giảm mạnh chi phí *miss* so với HDD —
không thay được RAM nhưng giảm đau.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Khi **index + working set > RAM**, buffer pool (mục 12.8) không cache hết → nhiều **miss → đọc đĩa → chậm**.
  Kiểm tra bằng **buffer pool hit rate** (`SHOW ENGINE INNODB STATUS`).
- **Xử lý (rẻ → đắt):**
  1. **Giảm kích thước index:** xoá index thừa, **PK nhỏ** (BIGINT thay UUID — mục 12.9), prefix index, kiểu
     gọn, gỡ phân mảnh (mục 12.7).
  2. **Tăng buffer pool/RAM** (`innodb_buffer_pool_size` ~70–80% RAM; scale dọc).
  3. Chỉ cần **hot working set** vừa RAM, không cần toàn bộ index.
  4. **Partitioning** (câu 14) / **Sharding** ra nhiều server (câu 16) → mỗi phần + index nhỏ, vừa RAM từng máy.
  5. **Archive** dữ liệu cũ; **SSD/NVMe** giảm đau khi buộc đọc đĩa.

[↑ Mục lục](#mục-lục)

<a id="q12-11"></a>
### 12.11. Xử lý khi query dùng nhiều index cùng lúc (Index Merge)

#### 📖 Giới thiệu kiến thức

Thông thường MySQL chỉ dùng **1 index cho mỗi bảng** trong một query. Nhưng khi có điều kiện trên **nhiều cột mà
mỗi cột có index riêng**, optimizer có thể dùng **Index Merge** — kết hợp kết quả từ **2+ index của cùng một
bảng**.

**Ba kiểu Index Merge** (EXPLAIN `type = index_merge`, `Extra` hiện kiểu):
- **Intersection (giao):** `WHERE a=1 AND b=2` — tìm PK từ mỗi index rồi **giao** (AND) → dòng thoả cả hai.
- **Union (hợp):** `WHERE a=1 OR b=2` — **hợp** kết quả 2 index (OR).
- **Sort-Union:** như union nhưng cần **sort PK** trước khi hợp (dùng khi điều kiện là range).

```sql
-- users có index riêng: idx_last(last_name), idx_first(first_name)
SELECT * FROM users WHERE last_name = 'Nguyen' AND first_name = 'An';
-- EXPLAIN: type=index_merge, Extra="Using intersect(idx_last, idx_first)"
```

**Khi nào Index Merge là DẤU HIỆU cần tối ưu:**
Index Merge tiện (tận dụng index sẵn có) nhưng **thường KÉM hơn một composite index đúng** — vì phải **quét 2
index + merge danh sách PK** (thêm chi phí) và không tận dụng được thứ tự tối ưu. Thấy `index_merge` trên **query
nóng** → thường nên **tạo 1 composite index** phủ các cột đó.

**Cách xử lý:**
1. **Intersection (AND) → tạo composite index:** `WHERE a AND b` → `INDEX(a, b)` (leftmost + ESR — mục 12.5).
   Nhanh hơn merge (một lần duyệt cây).
2. **Union (OR):** composite thường **không** giúp (leftmost chỉ dùng cột đầu) → giữ **index riêng mỗi cột** để
   merge union, hoặc viết lại thành **`UNION`** 2 query (mục 12.5).
3. **Kiểm chứng bằng `EXPLAIN`** (mục 13.1): xem `type=index_merge`, `Extra` (intersect/union), `key` (các index
   được dùng); so sánh với plan khi đã có composite index.
4. Nếu optimizer chọn merge **kém** hơn plan khác → dùng **`FORCE INDEX`**, hoặc tắt merge bằng
   `optimizer_switch='index_merge=off'` (hiếm khi cần).

> **Phân biệt với JOIN nhiều bảng:** Index Merge = **nhiều index trên CÙNG một bảng** trong 1 lần truy cập. Còn
> **JOIN** nhiều bảng thì mỗi bảng dùng **1 index riêng** của nó — đó là bình thường, **không phải** index merge.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Bình thường MySQL dùng **1 index/bảng**; khi điều kiện trên nhiều cột có index riêng, optimizer có thể **Index
  Merge** — kết hợp **2+ index cùng bảng**: **intersection** (AND), **union** (OR), **sort-union** (EXPLAIN
  `type=index_merge`).
- Index Merge tiện nhưng **thường kém hơn một composite index đúng** (phải quét 2 index + merge PK).
- **Xử lý:** thấy `index_merge` trên query nóng → **tạo composite index** phủ các cột (ESR — mục 12.5) thay cho
  intersection (AND); với OR dùng index riêng để merge union hoặc viết lại **`UNION`**.
- **Kiểm chứng bằng `EXPLAIN`** (`type=index_merge`, `Extra`); có thể `FORCE INDEX` / tắt merge khi optimizer
  chọn sai.
- Khác **JOIN nhiều bảng** (mỗi bảng 1 index — bình thường): index merge là nhiều index trên **cùng 1 bảng**.

[↑ Mục lục](#mục-lục)

<a id="q13"></a>
## 13. Một số kỹ thuật tối ưu hoá SQL trong MySQL

### 📖 Giới thiệu kiến thức

Mục tiêu tối ưu hoá query luôn là: **giảm số dòng phải quét, tận dụng index, giảm I/O và CPU**. Nguyên tắc
vàng: **đừng đoán — dùng `EXPLAIN` để đo**, rồi sửa theo dữ liệu. Dưới đây là các nhóm kỹ thuật quan trọng nhất.

**1. Dùng index đúng cách (quan trọng nhất).**
- Tạo index cho cột hay xuất hiện trong **`WHERE`, `JOIN ... ON`, `ORDER BY`, `GROUP BY`**.
- **Composite index** (nhiều cột) tuân theo **leftmost prefix**: index `(a, b, c)` dùng được cho `WHERE a`,
  `WHERE a AND b`, `a AND b AND c` — **không** dùng được nếu chỉ lọc `b` hoặc `c`.
- **Covering index:** index chứa **đủ mọi cột** mà query cần → đọc thẳng từ index, **không phải đọc bảng**
  (Extra hiện `Using index`).
- **Đừng lạm dụng index:** mỗi index làm `INSERT/UPDATE/DELETE` chậm hơn (phải cập nhật index) và tốn dung lượng.

**2. Dùng `EXPLAIN` để phân tích query.**

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 5;
-- Đọc các cột: type, key, rows, Extra
```
- **`type`:** `ALL` = **full table scan (xấu)**; tốt dần: `index` → `range` → `ref` → `eq_ref` → `const`.
- **`key`:** index thực sự được dùng (NULL = không dùng index nào).
- **`rows`:** số dòng optimizer ước tính phải đọc — càng nhỏ càng tốt.
- **`Extra`:** cảnh báo `Using filesort`, `Using temporary` → cần tối ưu (thiếu index cho `ORDER BY`/`GROUP BY`).
- MySQL 8.0.18+ có **`EXPLAIN ANALYZE`** cho thời gian chạy thực tế.

**3. Tránh làm index "vô hiệu" (index bị bỏ qua).**

```sql
-- ❌ Bọc hàm quanh cột index → không dùng được index
SELECT * FROM orders WHERE YEAR(created_at) = 2024;
-- ✔ Viết lại theo khoảng để dùng index trên created_at
SELECT * FROM orders WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```
- **`LIKE '%abc'`** / `'%abc%'` (wildcard đầu) → full scan; **`LIKE 'abc%'`** thì dùng index (xem câu 5).
- **Ép kiểu ngầm:** so cột số với chuỗi (`WHERE phone = 123` khi `phone` là VARCHAR) làm hỏng index.
- **`!=`, `<>`, `NOT IN`, `OR`** trên các cột khác nhau thường không dùng index tốt → cân nhắc `UNION`/tách index.

**4. Chỉ `SELECT` cột cần thiết (tránh `SELECT *`).**
Giảm I/O và băng thông, và cho phép **covering index** phát huy tác dụng.

**5. Lọc sớm & đúng chỗ.**
- Đẩy điều kiện xuống **`WHERE`** thay vì `HAVING` (lọc trước khi gom nhóm — xem câu 4.1).
- Ưu tiên điều kiện **chọn lọc cao (selective)** để loại nhiều dòng nhất.

**6. Phân trang hiệu quả — tránh `OFFSET` lớn.**

```sql
-- ❌ CHẬM: phải quét & bỏ 1 triệu dòng trước khi lấy 10 dòng
SELECT * FROM posts ORDER BY id LIMIT 10 OFFSET 1000000;
-- ✔ Keyset / seek pagination: nhớ id cuối trang trước, dùng index
SELECT * FROM posts WHERE id > 1000000 ORDER BY id LIMIT 10;
```

**7. Tối ưu `JOIN`.**
- JOIN trên cột **có index ở cả hai bảng**.
- Tránh JOIN quá nhiều bảng; cân nhắc **correlated subquery → JOIN** khi cần (xem câu 7).
- **Tránh N+1 query:** đừng query trong vòng lặp — dùng **1 JOIN** hoặc eager loading (vd Eloquent `with()`).

**8. Batch thao tác ghi.** Multi-row `INSERT`, `UPDATE` theo lô thay vì từng dòng (xem câu 15).

**9. Chọn đúng kiểu dữ liệu.** Dùng kiểu nhỏ gọn (`INT` thay `BIGINT` nếu đủ, `TINYINT` cho cờ), cột index
càng nhỏ càng nhanh; đặt `NOT NULL` khi có thể để optimizer làm việc tốt hơn.

**10. Cache & bảo trì.**
- **Cache tầng ứng dụng** (Redis) cho query đọc nhiều, ít đổi (MySQL 8 đã **bỏ query cache** nội bộ).
- **`ANALYZE TABLE`** cập nhật statistics cho optimizer chọn plan đúng.
- Với bảng cực lớn: cân nhắc **partitioning** (xem câu 14).

**11. Tìm điểm nóng bằng Slow Query Log.**

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;      -- log query chạy > 1 giây
-- Sau đó phân tích các query chậm bằng EXPLAIN và tối ưu theo các bước trên
```

> **Quy trình tối ưu thực tế:** (1) bật slow query log tìm query chậm → (2) `EXPLAIN` xem nó quét bao nhiêu dòng
> / có dùng index không → (3) thêm index / viết lại query / tránh làm index vô hiệu → (4) đo lại. **Đừng tối ưu
> mò khi chưa đo.**

### 🎯 Tóm tắt trả lời phỏng vấn

- **Cốt lõi:** giảm số dòng phải quét → **dùng index đúng** + **`EXPLAIN`** để kiểm chứng (`type=ALL` là full
  scan xấu; để ý `rows`, `Using filesort/temporary`).
- **Index:** tạo cho cột `WHERE/JOIN/ORDER BY/GROUP BY`; **composite theo leftmost prefix**; **covering index**;
  đừng thừa index (chậm ghi).
- **Đừng làm index vô hiệu:** không bọc hàm lên cột (`YEAR(col)`), không `LIKE '%x'`, tránh **ép kiểu ngầm**.
- **Query:** `SELECT` đúng cột (tránh `SELECT *`); lọc ở **`WHERE`** (không `HAVING` — câu 4.1); điều kiện
  selective trước.
- **Phân trang:** tránh **`OFFSET` lớn** → **keyset pagination** (`WHERE id > last_id`).
- **JOIN** trên cột index, **tránh N+1**; **batch** insert/update (câu 15); đúng kiểu dữ liệu; **`ANALYZE TABLE`**;
  bảng lớn → **partitioning** (câu 14). Tìm điểm nóng bằng **slow query log**.

[↑ Mục lục](#mục-lục)

<a id="q13-1"></a>
### 13.1. EXPLAIN trong SQL — đọc execution plan

#### 📖 Giới thiệu kiến thức

**`EXPLAIN`** cho biết **optimizer dự định thực thi query như thế nào** (execution plan) — dùng index nào, quét
bao nhiêu dòng, ghép bảng theo thứ tự nào — mà (với `SELECT`) **không thực sự chạy** query. Đây là **công cụ số
1** để chẩn đoán query chậm (đã nhắc ở câu 13): thay vì đoán, ta *nhìn* xem MySQL làm gì.

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 5;
EXPLAIN FORMAT=JSON SELECT ...;   -- chi tiết hơn: cost ước tính từng bước
EXPLAIN ANALYZE SELECT ...;       -- MySQL 8.0.18+: CHẠY thật + thời gian/số dòng thực tế
```

**Các cột quan trọng của `EXPLAIN` (MySQL):**

| Cột | Ý nghĩa |
|-----|---------|
| `id` | Số thứ tự của mỗi `SELECT` (subquery, union) |
| `select_type` | Loại: `SIMPLE`, `PRIMARY`, `SUBQUERY`, `DERIVED`, `UNION`… |
| `table` | Bảng đang xét ở bước này |
| **`type`** | **Kiểu truy cập — cột quan trọng nhất** (xem dưới) |
| `possible_keys` | Các index **có thể** dùng |
| **`key`** | Index **thực sự** được dùng (`NULL` = không dùng index nào) |
| `key_len` | Độ dài phần index được dùng (biết dùng bao nhiêu cột của composite index) |
| `ref` | Cột/hằng số khớp với index |
| **`rows`** | Số dòng optimizer **ước tính** phải đọc (càng nhỏ càng tốt) |
| `filtered` | % dòng còn lại sau khi lọc |
| **`Extra`** | Thông tin thêm — nhiều cảnh báo quan trọng (xem dưới) |

**`type` — mức độ truy cập, từ TỆ → TỐT:**

- **`ALL`**: **full table scan** — quét cả bảng (tệ nhất với bảng lớn).
- **`index`**: full index scan — quét toàn bộ index.
- **`range`**: quét theo khoảng dùng index (`BETWEEN`, `>`, `IN`, `LIKE 'x%'`).
- **`ref`**: dùng index **không-unique** khớp giá trị (JOIN/WHERE thông thường).
- **`eq_ref`**: khớp **đúng 1 dòng** qua unique/PK trong JOIN.
- **`const` / `system`**: khớp **1 dòng** qua PK/unique với hằng số (nhanh nhất).

> Mục tiêu: **tránh `ALL`** trên bảng lớn; đạt được `range`/`ref`/`const` là tốt.

**`Extra` — các cờ hay gặp cần chú ý:**

- **`Using index`**: **covering index** — đọc thẳng từ index, không phải đọc bảng (**TỐT**).
- **`Using where`**: lọc thêm ở tầng server sau khi đọc dòng.
- **`Using filesort`**: phải **sắp xếp riêng** (không dùng được index cho `ORDER BY`) → cân nhắc thêm index.
- **`Using temporary`**: tạo **bảng tạm** (thường do `GROUP BY`/`DISTINCT` phức tạp) → tốn kém.
- **`Using index condition`**: index condition pushdown (tốt).

> Hai cờ **`Using filesort`** và **`Using temporary`** thường là dấu hiệu **cần tối ưu** (thêm index cho
> `ORDER BY`/`GROUP BY`).

**Đọc `EXPLAIN` để hành động:**

1. `type = ALL` + `rows` lớn → **thiếu index** → thêm index cho cột `WHERE`/`JOIN`.
2. `key = NULL` dù có `possible_keys` → index **không dùng được** (bọc hàm lên cột, ép kiểu ngầm — xem câu 13).
3. `Using filesort`/`Using temporary` → thêm index phù hợp cho `ORDER BY`/`GROUP BY`.
4. `rows` ước tính lệch xa thực tế → chạy **`ANALYZE TABLE`** cập nhật statistics cho optimizer.

**`EXPLAIN` vs `EXPLAIN ANALYZE`:**

- **`EXPLAIN`**: chỉ **ước tính** (không chạy query) — nhanh, an toàn, để xem plan.
- **`EXPLAIN ANALYZE`** (MySQL 8.0.18+): **chạy thật**, cho **thời gian & số dòng thực tế** từng bước → so
  **ước tính vs thực tế**. Cẩn thận với `UPDATE/DELETE` vì nó **thực thi thật**.

> **PostgreSQL:** cũng có `EXPLAIN` và `EXPLAIN ANALYZE` (chạy thật), `EXPLAIN (ANALYZE, BUFFERS)`. Plan hiển thị
> dạng **cây node** (`Seq Scan`, `Index Scan`, `Nested Loop`, `Hash Join`…) kèm `cost=khởi_đầu..tổng`.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **`EXPLAIN`** cho biết **execution plan**: optimizer dùng index nào, quét bao nhiêu dòng, thứ tự ghép bảng —
  (với `SELECT`) **không chạy** query. Là công cụ số 1 để chẩn đoán query chậm.
- **Cột quan trọng:** **`type`** (`ALL` = full scan xấu → `range`/`ref`/`const` tốt), **`key`** (index thực dùng;
  `NULL` = không dùng), **`rows`** (số dòng ước tính — càng nhỏ càng tốt), **`Extra`** (`Using filesort`/
  `Using temporary` = cần tối ưu; `Using index` = covering, tốt).
- **Dùng để:** phát hiện **thiếu index** (`type=ALL`), **index bị vô hiệu** (`key=NULL`), **`ORDER BY`/`GROUP BY`
  tốn kém** (`filesort`/`temporary`).
- **`EXPLAIN ANALYZE`** (MySQL 8.0.18+ / PostgreSQL): **chạy thật**, cho thời gian & số dòng **thực tế** → so
  với ước tính. Kết hợp **`ANALYZE TABLE`** khi ước tính lệch.

[↑ Mục lục](#mục-lục)

<a id="q13-2"></a>
### 13.2. Full flow một câu query thực thi trong MySQL

#### 📖 Giới thiệu kiến thức

Khi client gửi một câu SQL, nó đi qua **nhiều tầng** của MySQL server trước khi trả kết quả. Hiểu flow này giúp
biết **mỗi bước tối ưu ở đâu** (bổ trợ cho `EXPLAIN` — mục 13.1). MySQL chia làm **2 lớp lớn**: **Server layer**
(kết nối, phân tích, tối ưu, thực thi) và **Storage Engine layer** (InnoDB… lo lưu/đọc dữ liệu thật) — kiến trúc
**pluggable storage engine** (xem câu 1).

**Các bước của flow:**

1. **Connector (tầng kết nối):** client mở kết nối (TCP/socket), **xác thực** user/password, kiểm tra **quyền**.
   Kết nối được quản qua thread (liên hệ connection pool — mục 2.3).

2. **Query Cache (chỉ MySQL ≤ 5.7):** nếu câu `SELECT` **y hệt** đã có kết quả cache → trả ngay. **Đã BỎ hẳn từ
   MySQL 8.0** (vì invalidation kém hiệu quả và gây contention) → 8.0+ **bỏ qua** bước này.

3. **Parser (phân tích cú pháp):** **tokenize** câu SQL (tách token) rồi **syntax analysis** → dựng **parse
   tree**; sai cú pháp báo lỗi ngay.

4. **Preprocessor / Resolver:** kiểm tra **bảng/cột có tồn tại**, phân giải tên (alias, `*`), kiểm tra **quyền**
   trên đối tượng.

5. **Optimizer (bộ tối ưu — "bộ não"):** quyết định **execution plan** rẻ nhất — chọn **index** nào, **thứ tự
   JOIN** các bảng, full scan hay index scan, có cần **filesort/bảng tạm** không. Dựa trên **statistics** (số
   dòng ước tính, phân bố dữ liệu) để chọn plan **chi phí (cost) thấp nhất**. Đây chính là cái **`EXPLAIN` cho
   ta xem** (mục 13.1); **`ANALYZE TABLE`** cập nhật statistics cho optimizer.

6. **Executor (bộ thực thi):** kiểm tra quyền lần cuối rồi **chạy plan** — gọi **API của storage engine** để
   đọc/ghi từng dòng (mở bảng → đọc dòng đầu → lặp theo plan).

7. **Storage Engine (InnoDB):** thực sự **đọc/ghi dữ liệu**: đọc từ **buffer pool** (bộ nhớ) nếu có, không thì
   nạp từ đĩa; với ghi thì cập nhật buffer pool + ghi **redo log (WAL)**/undo log, dùng **B+tree index**. Trả
   từng dòng về executor.

8. **Trả kết quả về client:** executor gom & trả kết quả (thường **streaming** trả dần); nếu chậm thì ghi vào
   **slow query log** (mục 12).

**Sơ đồ tổng quát:**

```
Client ──SQL──►  [SERVER LAYER]
                   Connector      (auth, thread/kết nối)
                   → Query Cache  (đã bỏ ở MySQL 8.0)
                   → Parser       (tokenize → parse tree, check cú pháp)
                   → Preprocessor (check bảng/cột/quyền)
                   → Optimizer    (chọn plan rẻ nhất ◄─ statistics)   ← EXPLAIN xem plan này
                   → Executor     (chạy plan, gọi API engine)
                        │
                        ▼        [STORAGE ENGINE LAYER] – InnoDB
                   buffer pool / redo-undo log / B+tree index / disk
                        │
                        ▼
                   Kết quả ──► Client (streaming)
```

> **Liên hệ:** đây là flow **vật lý** qua các thành phần của MySQL — **khác** với **thứ tự xử lý logic** của các
> mệnh đề (`FROM → WHERE → … → SELECT`) ở câu 4. Câu 4 nói *"kết quả được tính theo trình tự nào"*; câu này nói
> *"query chạy qua những bộ phận nào của engine"*.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Một câu query đi qua **2 lớp**: **Server layer** (kết nối → parser → optimizer → executor) và **Storage
  engine** (InnoDB đọc/ghi thật) — kiến trúc **pluggable storage engine** (câu 1).
- **Flow:** **Connector** (auth/thread) → *(Query Cache — bỏ ở MySQL 8.0)* → **Parser** (tokenize + parse tree)
  → **Preprocessor** (check bảng/cột/quyền) → **Optimizer** (chọn execution plan rẻ nhất theo statistics — chính
  là cái **EXPLAIN** xem, mục 13.1) → **Executor** (chạy plan, gọi API engine) → **Storage Engine/InnoDB** (đọc/
  ghi qua **buffer pool**, **redo/undo log**, **B+tree**) → trả kết quả (streaming).
- **Optimizer** là bộ quyết định hiệu năng; `EXPLAIN` xem plan, `ANALYZE TABLE` cập nhật statistics.
- **Phân biệt:** đây là flow **vật lý qua các thành phần engine**, khác **thứ tự xử lý logic** của mệnh đề (câu 4).

[↑ Mục lục](#mục-lục)

<a id="q14"></a>
## 14. Partitioning trong SQL là gì?

### 📖 Giới thiệu kiến thức

**Partitioning (phân vùng)** là kỹ thuật **chia một bảng lớn thành nhiều phần nhỏ (partition)** dựa trên giá
trị của một hoặc vài cột. Về mặt logic vẫn là **một bảng duy nhất** (câu SQL không đổi), nhưng về mặt vật lý
dữ liệu được lưu tách thành nhiều mảnh riêng. Mục đích: **tăng hiệu năng và dễ quản lý** với bảng cực lớn
(hàng chục–trăm triệu dòng).

**Tại sao cần?** Khi bảng quá lớn, quét/đánh index toàn bảng chậm và bảo trì (xóa dữ liệu cũ, backup) tốn
kém. Partition giúp:
- **Partition pruning:** truy vấn có điều kiện trên cột phân vùng → engine chỉ đọc **partition liên quan**,
  bỏ qua phần còn lại → nhanh hơn nhiều.
- **Quản lý dễ:** xóa dữ liệu cũ bằng cách **drop nguyên một partition** (rất nhanh) thay vì `DELETE` hàng loạt.
- **Bảo trì độc lập:** index/backup/vacuum từng partition riêng.

**Các kiểu partition phổ biến:**

| Kiểu | Chia theo | Ví dụ dùng |
|------|-----------|-----------|
| **RANGE** | Khoảng giá trị | Theo ngày/tháng/năm (log, đơn hàng theo thời gian) |
| **LIST** | Danh sách giá trị rời rạc | Theo vùng miền, quốc gia, trạng thái |
| **HASH** | Hàm băm trên cột | Phân tán đều dữ liệu khi không có mốc tự nhiên |
| **KEY** (MySQL) | Giống HASH nhưng dùng hàm băm nội bộ | Phân tán đều theo khóa |

**Ví dụ — RANGE partition theo năm:**

```sql
-- MySQL
CREATE TABLE orders (
    id INT, created_at DATE, total DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026)
);

-- Truy vấn dưới đây chỉ đọc partition p2025 (partition pruning), bỏ qua p2023/p2024:
SELECT * FROM orders WHERE created_at >= '2025-01-01';

-- Xóa toàn bộ dữ liệu năm 2023 cực nhanh (không DELETE từng dòng):
ALTER TABLE orders DROP PARTITION p2023;
```

```sql
-- PostgreSQL (declarative partitioning, từ PG 10)
CREATE TABLE orders (id INT, created_at DATE, total NUMERIC)
PARTITION BY RANGE (created_at);

CREATE TABLE orders_2025 PARTITION OF orders
    FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

**Phân biệt với các khái niệm gần giống:**
- **Partitioning vs Index:** index tạo cấu trúc tra cứu phụ *trên* bảng; partition **chia nhỏ chính dữ liệu**
  của bảng. Hai kỹ thuật **bổ trợ nhau** (thường partition rồi vẫn index trong từng partition).
- **Partitioning vs Sharding:** partition chia bảng trong **cùng một server/instance**; **sharding** chia dữ
  liệu ra **nhiều server khác nhau** (scale ngang). Partition là "chia trong nhà", sharding là "chia ra nhiều nhà".

**Lưu ý:** partition chỉ hiệu quả khi truy vấn **lọc theo đúng cột phân vùng** (partition key). Chọn sai key
→ vẫn quét mọi partition, không lợi gì mà còn phức tạp thêm.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Partitioning** = chia **một bảng lớn** thành nhiều **partition** theo giá trị cột; logic vẫn là 1 bảng,
  vật lý lưu tách riêng. Mục đích: **hiệu năng + dễ quản lý** với bảng cực lớn.
- Lợi ích chính: **partition pruning** (chỉ đọc partition liên quan → nhanh), **xóa dữ liệu cũ bằng drop
  partition**, bảo trì/backup từng phần.
- **Các kiểu:** **RANGE** (theo khoảng, vd ngày tháng), **LIST** (danh sách giá trị), **HASH** (phân tán đều),
  **KEY** (MySQL).
- **Phân biệt:** khác **index** (index là cấu trúc tra cứu phụ, partition chia chính dữ liệu — bổ trợ nhau);
  khác **sharding** (partition trong 1 server, sharding trải ra nhiều server).
- Chỉ hiệu quả khi truy vấn **lọc theo partition key**; chọn sai key thì vô ích.

[↑ Mục lục](#mục-lục)

<a id="q15"></a>
## 15. Làm thế nào để insert hàng triệu record vào DB hiệu quả?

### 📖 Giới thiệu kiến thức

Insert ~5 triệu record theo **cách ngây thơ** — chạy 5 triệu câu `INSERT` đơn lẻ, mỗi câu **autocommit** — sẽ
**cực chậm** (có thể hàng giờ). Lý do: mỗi `INSERT` phải **round-trip mạng + parse SQL + cập nhật index + fsync
commit** riêng. Muốn nhanh, phải **giảm số round-trip, giảm số lần commit/fsync, và giảm chi phí cập nhật index**.
Dưới đây là các kỹ thuật, xếp theo mức tác động.

**1. Bulk load từ file — NHANH NHẤT (nếu dữ liệu ở dạng file CSV).**
Bỏ qua phần lớn overhead của SQL parsing, engine đọc thẳng file:

```sql
-- MySQL: LOAD DATA INFILE (nhanh nhất trong MySQL)
LOAD DATA INFILE '/tmp/data.csv' INTO TABLE users
FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n'
(name, email);

-- PostgreSQL: COPY (tương đương, cực nhanh)
COPY users(name, email) FROM '/tmp/data.csv' WITH (FORMAT csv);
-- Từ client không có quyền file server: dùng \copy trong psql
```

**2. Multi-row INSERT (gộp nhiều dòng vào 1 câu) — khi insert từ code.**
Giảm mạnh số round-trip và số lần parse:

```sql
INSERT INTO users (name, email) VALUES
  ('a','a@x.com'), ('b','b@x.com'), ('c','c@x.com'), ... ;  -- vài trăm–vài nghìn dòng/câu
```

Kích thước lô hợp lý ~**500–5000 dòng/câu** (quá lớn dễ vượt `max_allowed_packet` của MySQL).

**3. Bọc trong transaction (commit theo lô) thay vì autocommit từng dòng.**
Tránh **fsync mỗi dòng** — commit một lần cho cả nghìn dòng:

```sql
START TRANSACTION;
  -- nhiều câu multi-row INSERT...
COMMIT;                 -- commit mỗi ~10.000–50.000 dòng, không phải mỗi dòng
```

**4. Tắt/drop index & constraint trong lúc load, bật/rebuild lại sau.**
Cập nhật B-tree index cho **mỗi** dòng rất tốn; rẻ hơn nhiều nếu **build index một lần** sau khi đã có đủ dữ liệu:

```sql
-- MySQL (InnoDB): tạm tắt các kiểm tra tốn kém
SET foreign_key_checks = 0;
SET unique_checks = 0;
-- ... load dữ liệu ...
SET unique_checks = 1;
SET foreign_key_checks = 1;
-- Tốt nhất: tạo bảng KHÔNG index → load → CREATE INDEX sau

-- PostgreSQL: drop index trước, COPY, rồi tạo lại
DROP INDEX idx_users_email;
-- ... COPY ...
CREATE INDEX idx_users_email ON users(email);
```

**5. Prepared statement + bind (khi insert từ ứng dụng).**
Parse **một lần**, execute nhiều lần với tham số khác nhau; kết hợp với multi-row + transaction:

```php
// PHP/PDO — chèn 5tr record theo lô, mỗi lô 1000 dòng, bọc transaction
$pdo->beginTransaction();
$batchSize = 1000;
$values = str_repeat('(?,?),', $batchSize - 1) . '(?,?)';
$stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES $values");

foreach (array_chunk($rows, $batchSize) as $i => $chunk) {
    $params = [];
    foreach ($chunk as $r) { $params[] = $r['name']; $params[] = $r['email']; }
    $stmt->execute($params);
    if ($i % 50 === 0) { $pdo->commit(); $pdo->beginTransaction(); } // commit định kỳ
}
$pdo->commit();
```

**6. Tinh chỉnh cấu hình DB khi load lớn (tạm thời).**
- **MySQL/InnoDB:** tăng `innodb_buffer_pool_size`; tạm đặt `innodb_flush_log_at_trx_commit = 2` (hoặc `0`) để
  **giảm fsync** (đánh đổi durability — chấp nhận được cho lần import); tăng `bulk_insert_buffer_size`.
- **PostgreSQL:** tăng `maintenance_work_mem` (giúp rebuild index nhanh) và `max_wal_size`; cân nhắc bảng
  **`UNLOGGED`** (bỏ WAL) cho dữ liệu tạm; nới `checkpoint`.

**7. Song song hoá (parallel) & mẹo khác.**
- Chia file thành nhiều phần, **load song song** trên nhiều connection (nếu CPU/đĩa chịu được).
- **Chèn theo thứ tự khóa chính** (dữ liệu đã sort theo PK) → index B-tree ít bị **page split** → nhanh hơn.

**Công thức thực tế cho 5 triệu record:**
1. Data ở file → dùng **`LOAD DATA INFILE`/`COPY`** (thường chỉ vài giây–phút).
2. Insert từ code → **multi-row INSERT (vài nghìn dòng) + transaction theo lô + prepared statement**.
3. **Drop/tắt index & FK trước, rebuild sau.**
4. Tinh chỉnh config (flush log, buffer) trong lúc import.
5. Cần nhanh hơn → **chia nhỏ + parallel**.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Đừng** insert từng dòng với autocommit (mỗi dòng = round-trip + fsync + cập nhật index → cực chậm).
- **Nhanh nhất: bulk load từ file** — MySQL **`LOAD DATA INFILE`**, PostgreSQL **`COPY`**.
- **Nếu insert từ code:** gộp **multi-row INSERT** (vài nghìn dòng/câu) + **bọc transaction commit theo lô** +
  **prepared statement**.
- **Drop/tắt index & FK check trước khi load, rebuild sau** — cập nhật index mỗi dòng rất tốn.
- **Tinh chỉnh cấu hình:** InnoDB `innodb_flush_log_at_trx_commit=2`, buffer pool lớn; PostgreSQL
  `maintenance_work_mem`, bảng `UNLOGGED`/nới WAL.
- Nâng cao: **chia nhỏ + load song song**, chèn **theo thứ tự PK** để tránh page split. Chọn theo nguồn dữ
  liệu: file → LOAD/COPY; từ app → batch + transaction.

[↑ Mục lục](#mục-lục)

<a id="q15-1"></a>
### 15.1. Xử lý bản ghi trùng lặp khi insert dữ liệu vào bảng

#### 📖 Giới thiệu kiến thức

Khi truyền/insert dữ liệu vào bảng (nhất là import lớn — câu 15), rất hay gặp **bản ghi trùng** (trùng khóa,
trùng email/mã...). MySQL có nhiều cơ chế xử lý, nhưng tất cả đều dựa trên một **điều kiện tiên quyết**:

> **Phải có `UNIQUE` index hoặc `PRIMARY KEY`** trên cột (hoặc tổ hợp cột) xác định tính duy nhất — nếu không,
> MySQL **không biết** thế nào là "trùng".

**A. Xử lý trùng NGAY LÚC INSERT — 3 cách chính:**

**1. `INSERT IGNORE` — bỏ qua dòng trùng (không báo lỗi, chỉ warning).**

```sql
INSERT IGNORE INTO users (email, name) VALUES ('a@x.com', 'An');
-- email đã tồn tại (UNIQUE) → BỎ QUA dòng này, các dòng khác vẫn insert bình thường
```
> ⚠️ `IGNORE` cũng **nuốt luôn** các lỗi khác (sai kiểu, vượt độ dài…) thành warning → dễ **mất dữ liệu âm thầm**.

**2. `INSERT ... ON DUPLICATE KEY UPDATE` — trùng thì UPDATE (upsert). Hay dùng nhất.**

```sql
INSERT INTO users (email, name, login_count) VALUES ('a@x.com', 'An', 1)
ON DUPLICATE KEY UPDATE
    name = VALUES(name),                 -- MySQL 8.0.20+: nên dùng alias `AS new` rồi new.name
    login_count = login_count + 1;       -- có thể cộng dồn dựa trên giá trị cũ
```
Ý nghĩa: **chưa có thì thêm, đã có thì cập nhật** (xem thêm ví dụ upsert ở mục 1.1).

**3. `REPLACE INTO` — trùng thì XÓA dòng cũ rồi INSERT dòng mới.**

```sql
REPLACE INTO users (email, name) VALUES ('a@x.com', 'An');
```
> ⚠️ REPLACE = `DELETE` + `INSERT`: **mất các cột không truyền** (về default), **đổi giá trị `AUTO_INCREMENT id`**,
> và **kích hoạt `ON DELETE CASCADE`** (câu 11) → có thể xóa lan dữ liệu con. Thường **`ON DUPLICATE KEY UPDATE`
> an toàn hơn**.

| Cách | Khi gặp trùng | Dòng cũ |
|------|---------------|---------|
| `INSERT IGNORE` | Bỏ qua dòng mới | **Giữ nguyên** |
| `ON DUPLICATE KEY UPDATE` | Cập nhật dòng cũ | **Giữ + update** |
| `REPLACE` | Xóa cũ, chèn mới | **Mất hẳn** (id đổi, cascade) |

**4. Lọc trùng ngay trong `INSERT ... SELECT`** (khi nạp từ bảng staging):

```sql
INSERT INTO users (email, name)
SELECT DISTINCT s.email, s.name FROM staging s
WHERE NOT EXISTS (SELECT 1 FROM users u WHERE u.email = s.email);
```

**5. Với `LOAD DATA INFILE`** (câu 15): thêm `IGNORE` hoặc `REPLACE`:

```sql
LOAD DATA INFILE 'data.csv' IGNORE  INTO TABLE users ...;   -- bỏ qua dòng trùng
LOAD DATA INFILE 'data.csv' REPLACE INTO TABLE users ...;   -- thay dòng trùng
```

**B. Dedup dữ liệu ĐÃ trùng sẵn trong bảng:**

```sql
-- (1) Tìm bản trùng
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- (2) Xóa trùng, GIỮ lại 1 bản (id nhỏ nhất) bằng self-JOIN
DELETE u1 FROM users u1
JOIN users u2 ON u1.email = u2.email AND u1.id > u2.id;

-- (3) Ngăn tái diễn: thêm UNIQUE (phải dedup xong trước, nếu không sẽ lỗi)
ALTER TABLE users ADD UNIQUE (email);
```

**C. Ở tầng ứng dụng (Laravel/Eloquent):** `insertOrIgnore()` (≈ INSERT IGNORE), `upsert()` (≈ ON DUPLICATE KEY
UPDATE), `updateOrCreate()` / `firstOrCreate()` — bản chất vẫn dựa trên UNIQUE key của MySQL.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Tiên quyết:** phải có **`UNIQUE`/`PRIMARY KEY`** để MySQL nhận biết "trùng".
- **Xử lý lúc insert (3 cách):** **`INSERT IGNORE`** (bỏ qua dòng trùng), **`ON DUPLICATE KEY UPDATE`** (upsert —
  có thì update, chưa thì thêm, *hay dùng nhất*), **`REPLACE`** (xóa cũ + chèn mới — cẩn thận mất cột, đổi id,
  kích hoạt cascade).
- Nạp từ nguồn: **`INSERT ... SELECT DISTINCT / WHERE NOT EXISTS`**; import file: **`LOAD DATA ... IGNORE/REPLACE`**.
- **Dedup dữ liệu đã trùng:** tìm bằng `GROUP BY ... HAVING COUNT(*)>1`; xóa bằng **self-JOIN giữ id nhỏ nhất**;
  rồi **thêm `UNIQUE`** để ngăn tái diễn.
- **Laravel:** `insertOrIgnore`, `upsert`, `updateOrCreate` — vẫn dựa trên UNIQUE key.

[↑ Mục lục](#mục-lục)

<a id="q16"></a>
## 16. Tăng tốc truy vấn SQL ở tầng kiến trúc/hệ thống (cache, read replica, denormalization...)

### 📖 Giới thiệu kiến thức

Câu 12 tối ưu **bản thân query** (index, EXPLAIN, viết query đúng). Khi đã làm hết mà **vẫn chậm**, hoặc **tải
đọc quá lớn** cho một DB, ta phải tăng tốc ở **tầng kiến trúc/hệ thống** — giảm số lần chạm DB hoặc chia tải ra
nhiều nơi. Các hướng chính, xếp từ rẻ/đơn giản đến phức tạp:

**1. Caching (bộ nhớ đệm) — thường hiệu quả nhất cho đọc nhiều.**
Lưu kết quả query/đối tượng vào **in-memory store (Redis, Memcached)**; request sau đọc từ cache (µs) thay vì
đánh DB (ms).

- **Chiến lược phổ biến — cache-aside:** đọc cache trước; **miss** thì đọc DB rồi **ghi vào cache** (kèm `TTL`).
- Các tầng cache: application/object cache, query-result cache, HTTP/CDN cache.
- **Thách thức lớn nhất: cache invalidation** — làm mới/xoá cache khi dữ liệu gốc đổi, nếu không sẽ trả dữ liệu cũ.

**2. Read Replica (bản sao đọc) — tách đọc/ghi.**
DB **primary** nhận **ghi**, tự **replication** sang nhiều **replica** chỉ để **đọc**. App gửi `SELECT` tới
replica, `INSERT/UPDATE/DELETE` tới primary → **scale đọc ngang** ra nhiều máy.

- Lưu ý **replication lag:** replica trễ hơn primary chút (eventual consistency) → có thể đọc thấy dữ liệu **hơi
  cũ**. Query cần **"read-your-write"** (vừa ghi xong đọc lại) nên đọc từ **primary**.

**3. Denormalization (phi chuẩn hoá).**
Chấp nhận **lặp dữ liệu** để **tránh JOIN đắt**: nhúng sẵn dữ liệu liên quan, hoặc dùng **cột tính sẵn**
(counter/precomputed column). Đổi lại: **đọc nhanh hơn** nhưng **ghi phức tạp hơn** (phải cập nhật nhiều chỗ,
rủi ro lệch dữ liệu).

**4. Materialized View & Summary/Aggregate table.**
- **View thường** chạy lại query mỗi lần gọi; **Materialized View** **lưu sẵn kết quả** query phức tạp → đọc
  nhanh, **refresh** định kỳ. PostgreSQL có `CREATE MATERIALIZED VIEW ... REFRESH`; **MySQL không có native** →
  tự làm bằng **summary table**.
- **Summary/Aggregate table:** tính sẵn số liệu tổng hợp (doanh thu theo ngày, số lượt...) vào bảng riêng, cập
  nhật định kỳ hoặc theo sự kiện — thay vì `COUNT/SUM` trên bảng lớn mỗi request.

**5. Connection pooling.** Tái dùng connection để bỏ overhead mở kết nối ở mỗi query (xem mục 2.3) → giảm độ trễ.

**6. Partitioning → Sharding.**
- **Partitioning** (câu 14): chia bảng lớn **trong 1 server** → partition pruning giúp quét ít hơn.
- **Sharding:** chia dữ liệu ra **nhiều server** (scale cả ghi) — mạnh nhất nhưng **phức tạp nhất**, thường là bước cuối.

**7. Search engine chuyên dụng.** Full-text search / lọc văn bản phức tạp → dùng **Elasticsearch** thay vì
`LIKE '%...%'` (vốn full scan) trên SQL.

**8. Nâng cấu hình / phần cứng (scale dọc).** Tăng **RAM** để buffer pool giữ nhiều dữ liệu nóng trong bộ nhớ
(`innodb_buffer_pool_size`), dùng **SSD/NVMe** nhanh. Xem scale dọc trong `Backend/devops.md` câu 2.

**Thứ tự áp dụng thực tế (rẻ → đắt/phức tạp):**
1. **Tối ưu query + index** (câu 13) — luôn làm trước.
2. **Cache (Redis)** cho phần đọc nhiều.
3. **Read replica** tách đọc/ghi.
4. **Denormalization / materialized / summary table** cho query nặng lặp lại.
5. **Partitioning**, rồi **sharding** khi thật sự cần.

> **Nguyên tắc:** đừng nhảy ngay sang replica/sharding khi chưa tối ưu query và chưa cache — thường **80% vấn đề
> chậm** giải quyết được ở **index + query + cache**, rẻ hơn nhiều so với thay đổi kiến trúc.

### 🎯 Tóm tắt trả lời phỏng vấn

- Sau khi đã **tối ưu query + index (câu 13)** mà vẫn chậm / tải đọc quá lớn → tăng tốc ở **tầng hệ thống**.
- **Cache (Redis/Memcached):** hiệu quả nhất cho đọc nhiều — **cache-aside + TTL**; khó nhất là **cache
  invalidation**.
- **Read replica:** tách **đọc → replica**, **ghi → primary**, scale đọc ngang; lưu ý **replication lag**
  (eventual consistency, read-your-write phải đọc primary).
- **Denormalization / materialized view / summary table:** tính/nhúng sẵn để tránh JOIN & aggregate đắt (đổi:
  ghi phức tạp hơn). **MySQL không có materialized view native** → dùng summary table.
- Thêm: **connection pooling** (mục 2.3), **partitioning** (câu 14) → **sharding**, **Elasticsearch** cho
  full-text, **nâng RAM/SSD** (scale dọc).
- **Thứ tự ưu tiên:** query/index → cache → replica → denormalize → partition/shard. Đừng đổi kiến trúc trước khi
  cache & tối ưu query.

[↑ Mục lục](#mục-lục)

<a id="q16-1"></a>
### 16.1. Kỹ thuật Replication (nhân bản dữ liệu)

#### 📖 Giới thiệu kiến thức

**Replication (nhân bản)** = sao chép dữ liệu từ một DB server (**primary/master**) sang một hoặc nhiều server
khác (**replica/slave/standby**) và giữ chúng **đồng bộ liên tục**. Đây là cơ chế nền cho **read replica** đã
nhắc ở câu 16.

**Mục đích chính:**
- **Scale đọc:** đẩy `SELECT` sang các replica (read replica — câu 16).
- **High availability / Failover:** primary chết → **promote** một replica lên làm primary → hệ thống tiếp tục.
- **Backup không ảnh hưởng production:** chạy backup trên replica.
- **Geo-distribution:** đặt replica gần user để giảm latency.
- **Tách workload phân tích (OLAP)** khỏi giao dịch (OLTP).

**Cơ chế cơ bản (MySQL):** primary ghi mọi thay đổi vào **binary log (binlog)**; replica có **I/O thread** đọc
binlog của primary về **relay log**, rồi **SQL thread** apply lại các thay đổi → dữ liệu đồng bộ. (PostgreSQL
dùng **WAL streaming** tương tự.)

**1. Phân loại theo cách đồng bộ (đánh đổi consistency ↔ performance):**

| Kiểu | Cách hoạt động | Đánh đổi |
|------|----------------|----------|
| **Asynchronous** (mặc định) | Primary commit xong **trả về ngay**, không chờ replica | Nhanh nhất, nhưng có **lag** & rủi ro **mất dữ liệu** nếu primary chết trước khi replica nhận |
| **Synchronous** | Primary chờ **tất cả** replica xác nhận đã ghi mới commit | **An toàn tuyệt đối** (không mất data) nhưng **chậm** (chờ mạng), 1 replica chậm → cả hệ chậm |
| **Semi-synchronous** | Primary chờ **≥1** replica xác nhận **đã nhận** rồi mới commit | **Cân bằng** an toàn & tốc độ |

**2. Phân loại theo topology (kiến trúc):**
- **Master–Slave / Primary–Replica:** 1 primary ghi, nhiều replica đọc — **phổ biến nhất** cho read scaling.
- **Master–Master / Multi-Primary:** nhiều node đều ghi được → phức tạp vì **conflict** khi 2 nơi sửa cùng dòng.
- **Cascading:** replica lại nhân bản tiếp cho replica khác (giảm tải primary).
- **Group Replication (MySQL) / Patroni (PostgreSQL):** cụm **tự bầu primary & tự failover**.

**3. Phân loại theo định dạng binlog (MySQL):**
- **Statement-based (SBR):** nhân bản **câu SQL** — gọn, nhưng **không an toàn** với hàm non-deterministic
  (`NOW()`, `RAND()`).
- **Row-based (RBR):** nhân bản **thay đổi từng dòng** — chính xác, an toàn (**mặc định MySQL 8**), nhưng binlog to hơn.
- **Mixed:** tự chọn SBR/RBR tuỳ trường hợp.

**Replication lag (vấn đề then chốt):** replica trễ hơn primary → đọc thấy **dữ liệu cũ** (eventual
consistency). Query cần **read-your-write** (vừa ghi xong đọc lại) phải đọc từ **primary**. Theo dõi lag: MySQL
`SHOW REPLICA STATUS` (`Seconds_Behind_Source`); PostgreSQL `pg_stat_replication`.

**Phân biệt với khái niệm gần giống:**
- **vs Backup:** replication đồng bộ **real-time**; backup là **snapshot** tại 1 thời điểm. Replication **KHÔNG
  thay thế backup** — xoá nhầm dữ liệu thì replica cũng **xoá theo**.
- **vs Sharding:** replication = **nhân bản toàn bộ** dữ liệu (nhiều bản giống nhau, để đọc/HA); sharding =
  **chia** dữ liệu ra nhiều node (mỗi node giữ 1 phần, để scale ghi). Thực tế hay **kết hợp**: mỗi shard lại có
  replica riêng.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Replication** = nhân bản dữ liệu từ **primary** sang **replica**, giữ đồng bộ. Mục đích: **scale đọc,
  HA/failover, backup, geo-distribution**.
- **Cơ chế (MySQL):** primary ghi **binlog** → replica **I/O thread** đọc về relay log → **SQL thread** apply
  (PostgreSQL: **WAL streaming**).
- **Theo đồng bộ:** **async** (mặc định — nhanh, có lag, rủi ro mất data) · **sync** (an toàn, chậm) ·
  **semi-sync** (cân bằng, chờ ≥1 replica xác nhận nhận).
- **Theo topology:** **master–slave** (phổ biến, 1 ghi nhiều đọc) · **master–master** (nhiều nơi ghi, dễ
  conflict) · group replication (tự failover). **Theo binlog (MySQL):** statement / **row (mặc định)** / mixed.
- **Replication lag:** replica trễ → eventual consistency; **read-your-write phải đọc primary**; theo dõi
  `Seconds_Behind_Source`.
- **Lưu ý:** replication **≠ backup** (xoá nhầm → replica xoá theo); khác **sharding** (nhân bản vs chia dữ
  liệu), thường **kết hợp**.

[↑ Mục lục](#mục-lục)

<a id="q16-2"></a>
### 16.2. Hệ thống lớn check email đã dùng cực nhanh thế nào? (Bloom filter)

#### 📖 Giới thiệu kiến thức

**Bài toán:** hệ thống hàng trăm triệu user, mỗi lần đăng ký cần kiểm tra **email đã tồn tại chưa**. Query DB
(`SELECT 1 FROM users WHERE email=?`) mỗi lần — dù có **UNIQUE index** (mục 15.1) — vẫn tốn round-trip + đọc
index; với lượng request khổng lồ (nhất là bot/spam thử) sẽ tạo **tải lớn lên DB**. Cần cách check "email này
**chắc chắn chưa có**" cực nhanh **trong bộ nhớ** để tránh phần lớn query DB.

**Bloom filter là gì:**
- **Cấu trúc dữ liệu xác suất (probabilistic)** kiểm tra một phần tử **có thể** thuộc tập hợp hay không, dùng
  **rất ít bộ nhớ**.
- Gồm: một **mảng bit** (kích thước m, ban đầu toàn 0) + **k hàm băm**.
- **Thêm phần tử:** băm qua k hàm → k vị trí → set các bit đó = 1.
- **Kiểm tra:** băm qua k hàm → nếu **bất kỳ** bit nào = 0 → **CHẮC CHẮN KHÔNG** có; nếu **tất cả** = 1 → **CÓ
  THỂ** có (hoặc false positive).

**Tính chất then chốt:**
- **KHÔNG có false negative:** nói "không có" → **chắc chắn** không có (an toàn để bỏ qua DB).
- **CÓ false positive:** nói "có thể có" nhưng thực ra không → chỉ khi này mới cần query DB xác nhận.
- Không xoá phần tử dễ dàng (biến thể **Counting Bloom Filter** cho phép xoá).

**Áp dụng check email:**
```
Đăng ký email X:
1. Hỏi Bloom filter (RAM / RedisBloom): X đã có trong tập email đã dùng?
   - Bloom nói "KHÔNG"   → chắc chắn chưa dùng → cho đăng ký NGAY, KHÔNG query DB.
   - Bloom nói "CÓ THỂ"  → query DB xác nhận (vì có thể false positive):
        · DB có    → email đã dùng thật (báo lỗi).
        · DB không → false positive → cho đăng ký.
2. Tạo user mới → thêm email vào Bloom filter (và DB).
```
→ **Phần lớn** email mới (chưa từng tồn tại) bị Bloom loại ngay trong RAM (**O(k)**, vài phép băm) → **tránh hầu
hết query DB**, chỉ query khi Bloom báo "có thể" (số ít) → **giảm tải DB khổng lồ**.

**Vì sao nhanh & rẻ:**
- Chỉ vài phép băm + đọc bit trong RAM → **O(k)**, **không round-trip DB**.
- Bộ nhớ cực nhỏ: vd **100 triệu** email với false positive ~1% chỉ cần **~120MB** (lưu cả tập email thật thì tốn GB).

**Ví dụ code cụ thể:**

**a) Redis (RedisBloom) — cách dùng thực tế nhất trong production:**

```bash
# Redis CLI (module RedisBloom)
BF.RESERVE emails 0.01 100000000     # tạo filter: false positive 1%, sức chứa 100 triệu
BF.ADD     emails "an@example.com"   # thêm 1 email
BF.EXISTS  emails "an@example.com"   # → 1  (có thể có → cần query DB xác nhận)
BF.EXISTS  emails "new@example.com"  # → 0  (CHẮC CHẮN chưa có → cho đăng ký, bỏ qua DB)
BF.MADD    emails "a@x.com" "b@x.com"  # thêm nhiều cùng lúc
```

**b) PHP/Laravel — luồng đăng ký kết hợp Bloom + DB:**

```php
use Illuminate\Support\Facades\Redis;

function isEmailAvailable(string $email): bool {
    // BF.EXISTS = 0  → CHẮC CHẮN chưa có → available ngay, KHÔNG đụng DB
    if (Redis::rawCommand('BF.EXISTS', 'emails', $email) === 0) {
        return true;
    }
    // Bloom báo "có thể có" → xác nhận bằng DB (phòng false positive)
    return ! User::where('email', $email)->exists();
}

function register(string $email /* ... */): User {
    if (! isEmailAvailable($email)) {
        throw new \RuntimeException('Email đã được sử dụng');
    }
    // UNIQUE index vẫn là chốt cuối chống race condition (mục 15.1)
    $user = User::create(['email' => $email /* ... */]);   // có thể ném lỗi duplicate
    Redis::rawCommand('BF.ADD', 'emails', $email);          // cập nhật bloom filter
    return $user;
}
```

**c) Cơ chế bên trong (Python minh hoạ — để hiểu bản chất):**

```python
import mmh3                       # MurmurHash
from bitarray import bitarray

class BloomFilter:
    def __init__(self, size, k):
        self.size, self.k = size, k
        self.bits = bitarray(size); self.bits.setall(0)

    def add(self, item):
        for i in range(self.k):                       # k hàm băm (seed khác nhau)
            self.bits[mmh3.hash(item, i) % self.size] = 1

    def maybe_contains(self, item):                   # BẤT KỲ bit = 0 → chắc chắn KHÔNG có
        return all(self.bits[mmh3.hash(item, i) % self.size] for i in range(self.k))

bf = BloomFilter(1_000_000, 7)
bf.add("an@x.com")
bf.maybe_contains("an@x.com")    # True  (có thể có)
bf.maybe_contains("new@x.com")   # False (chắc chắn chưa có → bỏ qua DB)
```

**d) Góc "SQL" — PostgreSQL có extension `bloom` (lưu ý: KHÁC mục đích):**

```sql
CREATE EXTENSION bloom;
-- bloom INDEX cho truy vấn equality trên NHIỀU cột bất kỳ tổ hợp nào
CREATE INDEX idx_bloom ON t USING bloom (col1, col2, col3);
-- WHERE col2 = ? AND col3 = ?  → dùng được (index thường cần leftmost prefix thì không)
```
> Đây là **kiểu index** trong PostgreSQL (đánh đổi false positive → PG tự re-check dòng), **khác** với lớp
> **pre-filter chống query DB** đang bàn. MySQL **không** có bloom filter native cho existence check → dùng
> **Redis/RedisBloom** hoặc thư viện in-process.

**Đánh đổi & lưu ý quan trọng:**
- **False positive** → phải query DB xác nhận ở **số ít** trường hợp (kết quả cuối **không sai**, chỉ tốn thêm 1
  query). Tăng m/k → giảm false positive nhưng tốn RAM hơn.
- **KHÔNG thay thế UNIQUE constraint:** Bloom chỉ là **lớp lọc trước (pre-filter)** để giảm tải; **nguồn chân lý
  & đảm bảo duy nhất vẫn là `UNIQUE` index ở DB** (mục 15.1). Kể cả khi Bloom báo "chưa có", vẫn phải insert với
  `UNIQUE` để tránh **race condition**.
- **Khởi tạo/đồng bộ:** phải nạp sẵn email hiện có vào filter; hệ phân tán để trên **RedisBloom** cho nhiều app
  instance dùng chung.

**Nơi dùng thực tế:** chống trùng username/email, chống hiển thị lại bài đã đọc (news feed), web crawler (URL đã
thăm), **chống cache penetration** (check key tồn tại trước khi query DB), Cassandra/HBase dùng bloom filter để
**bỏ qua SSTable không chứa key**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Bloom filter** = cấu trúc dữ liệu **xác suất** (mảng bit + k hàm băm) kiểm tra "phần tử có thể thuộc tập" với
  **bộ nhớ cực nhỏ** & tốc độ **O(k)** trong RAM.
- **Tính chất:** **không false negative** (nói "không có" là chắc chắn không) nhưng **có false positive** (nói
  "có thể có").
- **Check email nhanh:** hỏi Bloom trước — "chưa có" → cho đăng ký ngay (**bỏ qua DB**); "có thể có" → mới query
  DB xác nhận → tránh **phần lớn** query DB cho email mới → **giảm tải khổng lồ**.
- **Đánh đổi:** false positive → số ít phải query DB xác nhận; **KHÔNG thay `UNIQUE` constraint** — Bloom chỉ là
  lớp lọc trước, nguồn chân lý vẫn là **UNIQUE index** (mục 15.1). Phân tán dùng **RedisBloom**.
- Ứng dụng khác: chống **cache penetration**, URL crawler, **Cassandra/HBase SSTable filter**.

[↑ Mục lục](#mục-lục)

<a id="q17"></a>
## 17. Tại sao phải lưu timestamp (created_at/updated_at)?

### 📖 Giới thiệu kiến thức

**Timestamp** là cột lưu **thời điểm** một sự kiện xảy ra (kiểu `TIMESTAMP`/`DATETIME`). Phổ biến nhất là cặp
**`created_at`** (lúc record được tạo) và **`updated_at`** (lần sửa gần nhất) — gần như **mọi bảng nghiệp vụ đều
nên có**. Lý do:

**1. Audit / truy vết (auditing).** Biết record được tạo/sửa **khi nào** — phục vụ kiểm toán, tuân thủ
(compliance), điều tra sự cố ("dữ liệu này đổi lúc nào?").

**2. Sắp xếp & lọc theo thời gian.** Hiển thị mới nhất (`ORDER BY created_at DESC`), lọc theo khoảng ("đơn hàng
tháng này"), báo cáo/thống kê theo thời gian — vô số tính năng dựa vào đây.

**3. Debug & giám sát.** Khi có bug/sự cố, timestamp giúp **dựng lại dòng thời gian (timeline)** và tương quan
với log để tìm nguyên nhân.

**4. Đồng bộ dữ liệu tăng dần (incremental sync / ETL).** Chỉ lấy record **thay đổi từ lần sync trước**
(`WHERE updated_at > :last_sync`) thay vì quét cả bảng → đồng bộ hiệu quả (nền tảng của change-data-capture).

**5. Optimistic locking.** Dùng `updated_at` làm "version" để phát hiện xung đột ghi:
`UPDATE ... WHERE id = ? AND updated_at = ?` (xem mục 10.5).

**6. Soft delete (xóa mềm).** Cột **`deleted_at`**: `NULL` = còn sống, có giá trị = đã xóa (giữ dữ liệu, khôi
phục được) — xem câu 11.

**7. Cache & HTTP điều kiện.** `updated_at` dùng cho `Last-Modified`/`ETag` và **invalidation cache** khi dữ
liệu đổi (liên hệ câu 16).

**8. Time-series & partitioning.** Dữ liệu theo thời gian (log, metric, đơn hàng) thường **partition RANGE theo
`created_at`** (xem câu 14); xóa dữ liệu cũ bằng drop partition.

**9. Business logic phụ thuộc thời gian.** Hết hạn (expiry/trial), tính tuổi record, SLA, thời hạn giao hàng...

**Thực hành tốt:**

```sql
CREATE TABLE orders (
  id INT PRIMARY KEY,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  -- tự cập nhật khi sửa
);
```

- **Lưu theo UTC**, convert sang giờ local khi hiển thị → tránh lệch múi giờ giữa các server/người dùng.
- **Index** cột timestamp nếu hay lọc/sắp xếp theo nó.
- Framework tự lo: **Laravel** quản `created_at`/`updated_at` (`$timestamps`) và `deleted_at` (trait `SoftDeletes`).

**Bonus — `TIMESTAMP` vs `DATETIME` (hay hỏi kèm):**

| | `TIMESTAMP` | `DATETIME` |
|---|-------------|------------|
| Kích thước | **4 byte** | 8 byte |
| Khoảng giá trị | 1970–**2038** (lỗi năm 2038) | 1000–9999 |
| Múi giờ | **Lưu UTC**, tự đổi theo timezone phiên | **Không** phụ thuộc timezone |
| Auto update | Có (`ON UPDATE CURRENT_TIMESTAMP`) | Có (từ MySQL 5.6) |

### 🎯 Tóm tắt trả lời phỏng vấn

- Timestamp (`created_at`/`updated_at`/`deleted_at`) lưu **thời điểm** sự kiện — gần như **mọi bảng nên có**.
- **Vì sao:** **audit/truy vết**, **sắp xếp & lọc theo thời gian**, **debug (dựng timeline)**, **sync tăng dần**
  (`WHERE updated_at > last_sync`), **optimistic locking** (updated_at làm version — mục 10.5), **soft delete**
  (`deleted_at` — câu 11), **cache/ETag** (câu 16), **time-series/partition** (câu 14), **business logic** (hết
  hạn, SLA).
- **Thực hành:** default `CURRENT_TIMESTAMP` + `ON UPDATE`, **lưu UTC**, **index** nếu hay query theo thời gian;
  Laravel tự quản.
- **Bonus:** **`TIMESTAMP`** (4B, lưu UTC, lỗi 2038, tự đổi timezone) vs **`DATETIME`** (8B, không timezone,
  range rộng).

[↑ Mục lục](#mục-lục)

<a id="q18"></a>
## 18. Redis — in-memory data store

> Gom toàn bộ chủ đề Redis: khái niệm & tốc độ, TTL, các ứng dụng (queue/pub-sub/stream/lock), bảo mật,
> persistence, data recovery và Cluster. Đọc từ trên xuống theo lộ trình cơ bản → nâng cao.

<a id="q18-1"></a>
### 18.1. Redis là gì? Cách sử dụng (data type, use case)

#### 📖 Giới thiệu kiến thức

**Redis (REmote DImctionary Server)** là **in-memory data store** dạng **key-value** mã nguồn mở — dữ liệu nằm
chủ yếu trong **RAM** nên đọc/ghi cực nhanh (**µs–ms**). Redis là đại diện tiêu biểu của nhóm **Key-Value NoSQL**
(mục 3.1), nhưng thực chất mạnh hơn key-value thuần vì hỗ trợ **nhiều kiểu dữ liệu có cấu trúc**.

**Đặc điểm cốt lõi:**
- **In-memory:** giữ toàn bộ dataset trong RAM → nhanh; có cơ chế **persistence** ghi ra đĩa để không mất khi
  restart (xem mục 18.14).
- **Single-threaded** (cho phần thực thi lệnh): mỗi lệnh xử lý tuần tự → **atomic** tự nhiên, không lo race
  condition ở cấp 1 lệnh (từ Redis 6 có I/O đa luồng cho network, nhưng lệnh vẫn tuần tự).
- **TTL & auto-expire:** mỗi key có thể đặt thời gian sống, hết hạn tự xóa — hợp cho cache/session.

**Các kiểu dữ liệu chính (đây là điểm mạnh so với key-value thuần):**

| Kiểu | Mô tả | Use case điển hình |
|------|-------|--------------------|
| **String** | Chuỗi/số/binary (tối đa 512MB) | Cache giá trị, counter (`INCR`), flag |
| **Hash** | Map field→value trong 1 key | Lưu object (user profile) |
| **List** | Danh sách có thứ tự (linked list) | Queue/stack, hàng đợi job, timeline |
| **Set** | Tập không trùng, không thứ tự | Tag, lọc trùng, quan hệ (bạn chung) |
| **Sorted Set (ZSet)** | Set có `score` để sắp xếp | **Leaderboard**, xếp hạng, delayed queue |
| **Bitmap / HyperLogLog** | Đếm/xấp xỉ tiết kiệm RAM | Daily active user, đếm unique gần đúng |
| **Stream** | Log append-only (như Kafka mini) | Event stream, message queue bền |
| **Geo** | Toạ độ địa lý | Tìm cửa hàng gần, ride-hailing |

**Lệnh cơ bản (Redis CLI):**

```bash
SET user:1:name "An" EX 3600     # set kèm TTL 3600s (1 giờ)
GET user:1:name
TTL user:1:name                  # còn bao nhiêu giây sống
INCR page:views                  # tăng counter atomic (rate limit, đếm view)

HSET user:1 name "An" age 25     # Hash: lưu object
HGETALL user:1

LPUSH jobs "job1"                # List làm queue
RPOP jobs

ZADD leaderboard 100 "An" 250 "Bình"   # Sorted Set: xếp hạng
ZREVRANGE leaderboard 0 9 WITHSCORES     # top 10 điểm cao nhất

SET lock:order:5 1 NX EX 10      # NX = chỉ set nếu chưa tồn tại → distributed lock
```

**Use case phổ biến nhất:**
- **Cache** (cache-aside): giảm tải DB cho dữ liệu đọc nhiều, ít đổi (mục 16).
- **Session store:** lưu session tập trung cho nhiều server (stateless app).
- **Rate limiting:** `INCR` + TTL để đếm số request/giây.
- **Leaderboard / ranking:** Sorted Set.
- **Distributed lock:** `SET NX PX` (hoặc Redlock cho phân tán).
- **Queue / pub/sub / stream:** List, `PUBLISH/SUBSCRIBE`, Stream.

**Pattern cache-aside (kinh điển) — PHP/Laravel:**

```php
use Illuminate\Support\Facades\Redis;

function getUser($id) {
    $key = "user:$id";
    $cached = Redis::get($key);
    if ($cached) return json_decode($cached, true);   // cache HIT

    $user = DB::table('users')->find($id);            // cache MISS → đọc DB
    Redis::setex($key, 3600, json_encode($user));     // ghi cache kèm TTL 1h
    return (array) $user;
}
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Redis** = **in-memory key-value store** mã nguồn mở, đọc/ghi **µs–ms** vì để dữ liệu trong RAM; **single-thread**
  nên mỗi lệnh **atomic**; hỗ trợ **TTL** tự hết hạn.
- Mạnh hơn key-value thuần nhờ **nhiều kiểu dữ liệu**: String, Hash, List, Set, **Sorted Set**, Bitmap/HyperLogLog,
  Stream, Geo.
- **Use case:** cache (cache-aside), **session store**, rate limiting, **leaderboard** (ZSet), distributed lock
  (`SET NX PX`), queue/pub-sub/stream.
- Lệnh nhớ: `SET key val EX ttl`, `GET`, `INCR`, `HSET/HGETALL`, `LPUSH/RPOP`, `ZADD/ZREVRANGE`, `SET ... NX`.

[↑ Mục lục](#mục-lục)

<a id="q18-2"></a>
### 18.2. Vì sao lưu/đọc Redis nhanh hơn MySQL?

#### 📖 Giới thiệu kiến thức

Redis nhanh hơn MySQL tới **hàng chục–hàng trăm lần** cho thao tác đọc/ghi đơn giản (Redis ~**sub-ms**, MySQL
thường **vài–vài chục ms**). Không phải do "Redis viết tối ưu hơn" mà do **kiến trúc khác hẳn về mục tiêu**. Có
**5 lý do gốc**:

**1. In-memory vs on-disk (lý do lớn nhất).**
- **Redis giữ toàn bộ dữ liệu trong RAM** → đọc/ghi thao tác thẳng trên bộ nhớ (**~100 ns**).
- **MySQL là on-disk**: dữ liệu và index nằm trên đĩa; dù có **buffer pool** cache trang nóng trong RAM, nó vẫn
  phải **đảm bảo bền vững** — ghi phải xuống đĩa (redo log/WAL, flush). Truy cập đĩa (kể cả SSD ~**100 µs**) chậm
  hơn RAM **~1000 lần**.

**2. Mô hình dữ liệu đơn giản (key → value) vs quan hệ.**
- Redis truy cập theo **key** qua **hash table → độ phức tạp O(1)**, không parse gì nhiều.
- MySQL mỗi query phải: **parse SQL → optimizer chọn plan → duyệt B+Tree index → đọc trang → (có thể) JOIN nhiều
  bảng → trả kết quả**. Nhiều bước, mỗi bước tốn CPU/IO. Redis **không có** những bước này.

**3. Không có chi phí ACID/transaction nặng.**
- MySQL (InnoDB) trả giá cho **độ bền & nhất quán**: ghi **redo log + undo log**, khóa hàng (row lock), MVCC, fsync
  khi commit → an toàn nhưng **tốn**.
- Redis **single-thread**, mỗi lệnh **atomic tự nhiên**, persistence **bất đồng bộ** (AOF/RDB ghi nền) → **không**
  chặn thao tác. Đổi lại: chấp nhận **có thể mất vài giây dữ liệu** khi sự cố (mục 18.14).

**4. Kiến trúc single-thread + I/O đa hợp (multiplexing).**
- Redis xử lý lệnh trên **một luồng** dùng **event loop (epoll)** → **không tốn chi phí khóa (lock), không context
  switching** giữa nhiều thread tranh dữ liệu. Với thao tác cực ngắn trên RAM, mô hình này *nhanh hơn* đa luồng.
- MySQL đa luồng (mỗi connection một thread) → có chi phí **lock, latch, context switch** để đảm bảo đồng thời an toàn.

**5. Cấu trúc dữ liệu tối ưu sẵn trong RAM.**
- Redis cài sẵn hash table, skip list (ZSet), ziplist/listpack… **tối ưu cho truy cập RAM**, thao tác đúng một
  bước. MySQL phải tổ chức dữ liệu theo **trang (page) trên đĩa** cho mục tiêu bền vững, không tối ưu cho tốc độ thuần.

**Cái giá của tốc độ (phải nói ra để không bị hỏi vặn):**
- Dữ liệu **giới hạn bởi RAM** (đắt hơn đĩa nhiều), khó chứa dataset khổng lồ.
- **Độ bền yếu hơn**: có thể mất dữ liệu gần nhất khi crash (async persistence).
- **Truy vấn hạn chế**: chủ yếu theo key, **không** JOIN/aggregate phức tạp/ràng buộc quan hệ như SQL.

> **Kết luận đúng bản chất:** MySQL và Redis **giải bài toán khác nhau**. MySQL = **source of truth** bền vững,
> quan hệ, truy vấn phức tạp. Redis = **tầng tốc độ** (cache/session/counter) đánh đổi độ bền & khả năng truy vấn
> lấy latency cực thấp. Vì vậy thực tế thường **dùng cả hai**: Redis cache trước MySQL (mục 16).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Redis nhanh hơn vì **kiến trúc khác mục tiêu**, không phải "code tối ưu hơn". 5 lý do:
- **(1) In-memory vs on-disk** — Redis thao tác thẳng trên **RAM (~ns)**, MySQL phải đảm bảo bền → chạm **đĩa
  (~µs, chậm ~1000 lần)**. Đây là lý do lớn nhất.
- **(2) Key-value O(1)** vs MySQL phải **parse SQL → optimizer → B+Tree → JOIN**.
- **(3) Không ACID nặng** — MySQL ghi redo/undo log, row lock, fsync commit; Redis atomic single-thread, persistence async.
- **(4) Single-thread + event loop (epoll)** — không lock/context-switch; **(5) cấu trúc dữ liệu tối ưu cho RAM**.
- **Cái giá:** giới hạn RAM, độ bền yếu hơn, không truy vấn quan hệ phức tạp → nên **dùng cả hai** (Redis cache
  trước MySQL), Redis **không thay** MySQL làm source of truth.

[↑ Mục lục](#mục-lục)

<a id="q18-3"></a>
### 18.3. Cơ chế lưu key kèm TTL (thời gian sống) hoạt động thế nào?

#### 📖 Giới thiệu kiến thức

Rất nhiều ứng dụng Redis (cache, session, rate limit, lock) dựa vào việc key **tự hết hạn** sau một khoảng thời
gian. Câu hỏi hay bị hỏi vặn: *"Đặt TTL 60s cho 1 triệu key — đến giây thứ 60 Redis có phải quét cả triệu key để
xóa không?"* Câu trả lời: **không** — nhờ cơ chế **lazy + active expiration**.

**1. TTL được lưu ở đâu?**
Redis giữ **một dictionary (hash table) riêng tên `expires`**, map **key → thời điểm hết hạn (Unix timestamp,
mili-giây)**. Khi bạn `SET k v EX 60` hay `EXPIRE k 60`, Redis ghi vào `expires[k] = now + 60000`. Bản thân dữ
liệu vẫn nằm trong dictionary chính; `expires` chỉ lưu **mốc chết** của những key có hạn.

```bash
SET session:1 "..." EX 3600   # đặt TTL ngay khi set
EXPIRE session:1 3600         # hoặc đặt TTL cho key đã có
TTL session:1                 # còn bao nhiêu giây (-1: không hạn, -2: đã hết/không tồn tại)
PERSIST session:1             # gỡ TTL → key sống vĩnh viễn
```

**2. Redis KHÔNG dùng timer cho từng key.** Nếu mỗi key có một hẹn giờ riêng thì 1 triệu key = 1 triệu timer →
quá tốn. Thay vào đó Redis xóa key hết hạn theo **2 chiến lược kết hợp**:

**a) Lazy expiration (xóa lười — khi truy cập).**
- Khi có ai **truy cập** một key (GET, SET…), Redis **kiểm tra TTL trước**: nếu đã quá hạn → **xóa ngay** rồi trả
  về "không tồn tại".
- Ưu: không tốn CPU cho key không ai đụng. Nhược: nếu key hết hạn mà **không ai truy cập**, nó **vẫn nằm trong
  RAM** → tốn bộ nhớ. Vì vậy cần thêm (b).

**b) Active expiration (xóa chủ động — quét định kỳ lấy mẫu).**
- Redis chạy một **background job ~10 lần/giây** (mỗi ~100ms): **lấy mẫu ngẫnhiên** ~20 key **có TTL** từ dict
  `expires`, xóa những cái đã hết hạn. Nếu **>25%** mẫu đã hết hạn → **lặp lại ngay** vòng nữa (vì có vẻ còn nhiều
  key chết). Cứ thế tới khi tỉ lệ hết hạn trong mẫu xuống thấp.
- Đây là thuật toán **xác suất (probabilistic)**: không quét toàn bộ, chỉ lấy mẫu → **không block**, mà vẫn dọn
  kịp phần lớn key chết. Đánh đổi: có thể có **độ trễ nhỏ** giữa lúc key hết hạn và lúc thực sự bị xóa khỏi RAM.

**3. Nhất quán trên replica.** Replica **không tự xóa** key hết hạn; nó **chờ lệnh `DEL` từ master** (master phát
`DEL`/`UNLINK` khi expire) để đảm bảo master–replica đồng nhất. (Đọc trên replica một key vừa hết hạn về mặt logic
được coi là đã hết.)

**4. Liên quan eviction (khác expiration).** Cần phân biệt:
- **Expiration:** xóa key vì **hết TTL** (do ta chủ động đặt hạn).
- **Eviction:** khi **RAM chạm `maxmemory`**, Redis xóa bớt key theo **policy** để có chỗ — kể cả key **chưa hết
  hạn**. Các policy: `noeviction` (mặc định — báo lỗi ghi), `allkeys-lru`/`allkeys-lfu` (xóa ít dùng nhất trong
  mọi key), `volatile-lru`/`volatile-ttl` (chỉ xóa trong nhóm key **có TTL**, ưu tiên sắp hết hạn)...

```conf
maxmemory 2gb
maxmemory-policy allkeys-lru   # khi đầy RAM: xóa key ít dùng gần đây nhất
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- TTL lưu trong **dict `expires` riêng** (key → timestamp hết hạn, ms); `SET ... EX`, `EXPIRE`, xem bằng `TTL`,
  gỡ bằng `PERSIST`.
- Redis **không dùng timer/từng key**, mà kết hợp **2 chiến lược**:
  - **Lazy:** khi **truy cập** key mới kiểm tra & xóa nếu hết hạn (rẻ, nhưng key không ai đụng còn tốn RAM).
  - **Active:** background **~10 lần/giây** **lấy mẫu ngẫnhiên ~20 key có TTL**, xóa cái hết hạn; nếu >25% mẫu chết
    thì lặp ngay. Là thuật toán **xác suất**, không quét toàn bộ → không block.
- **Replica** không tự expire, **chờ `DEL` từ master** để đồng nhất.
- Phân biệt với **eviction**: khi chạm **`maxmemory`** Redis xóa key theo **policy** (`allkeys-lru`, `volatile-ttl`…)
  kể cả key **chưa hết hạn** — khác với expiration theo TTL.

[↑ Mục lục](#mục-lục)

<a id="q18-4"></a>
### 18.4. Các ứng dụng thực tế của Redis

#### 📖 Giới thiệu kiến thức

Redis không chỉ là "cache". Nhờ **tốc độ µs–ms** + **nhiều kiểu dữ liệu** + **atomic** + **TTL** + **pub/sub**,
Redis được dùng cho rất nhiều bài toán. Điểm chung: những chỗ cần **truy cập cực nhanh**, **dữ liệu tạm/biến động
nhanh**, hoặc cần **điều phối giữa nhiều tiến trình/server**. Dưới đây là các ứng dụng phổ biến, kèm **kiểu dữ liệu
và cơ chế** đứng sau.

**1. Caching (ứng dụng số 1).**
Giảm tải DB cho dữ liệu **đọc nhiều, ít đổi**. Pattern **cache-aside**: đọc cache trước, miss thì đọc DB rồi ghi
cache kèm **TTL**. Dùng kiểu **String/Hash**.
- *Ví dụ:* cache trang sản phẩm, kết quả query nặng, response API.

**2. Session store (lưu phiên tập trung).**
App chạy nhiều server (load balancer) cần session **dùng chung** thay vì lưu trong RAM từng server → Redis làm
**session store** tập trung, có TTL tự hết hạn. Giúp app **stateless**, scale ngang dễ.

**3. Rate limiting (giới hạn tần suất).**
Chống spam/abuse API: đếm số request theo user/IP trong một cửa sổ thời gian bằng **`INCR` + TTL** (atomic).
```bash
# Cho phép tối đa 100 request/phút mỗi user
INCR rate:user:1        # tăng đếm (atomic)
EXPIRE rate:user:1 60   # đặt TTL 60s cho lần đầu → hết phút tự reset
# nếu giá trị > 100 → chặn
```

**4. Leaderboard / bảng xếp hạng.**
**Sorted Set (ZSet)** giữ phần tử kèm `score`, tự sắp xếp → lấy top-N và thứ hạng cực nhanh (game, thi đấu, trending).
```bash
ZADD leaderboard 250 "An"
ZREVRANGE leaderboard 0 9 WITHSCORES   # top 10
ZREVRANK leaderboard "An"              # hạng của "An"
```

**5. Distributed lock (khóa phân tán).**
Đảm bảo chỉ **một tiến trình** xử lý một tài nguyên tại một thời điểm giữa nhiều server (chống race condition):
`SET key val NX PX ttl` — chỉ set nếu chưa có, kèm hạn tự nhả. Bài toán mạnh hơn dùng **Redlock**.
- *Ví dụ:* chống double-submit đơn hàng, cron job chỉ chạy 1 nơi.

**6. Message queue & background job.**
- **List** làm hàng đợi đơn giản (`LPUSH`/`BRPOP` — blocking pop). Laravel/Bull (Node) dùng Redis làm **queue driver**.
- **Stream** (append-only log, có consumer group) cho hàng đợi **bền, nhiều consumer** — như Kafka thu nhỏ.

**7. Pub/Sub (real-time messaging).**
`PUBLISH`/`SUBSCRIBE` phát/nhận message theo kênh → **chat, notification, live update**, đồng bộ giữa các server
(vd broadcast WebSocket). Lưu ý: pub/sub cổ điển **không lưu lại** message (fire-and-forget) → cần bền thì dùng **Stream**.

**8. Counter & analytics thời gian thực.**
`INCR`/`DECR` atomic cho **đếm view, like, tồn kho**; **Bitmap** cho trạng thái nhị phân (điểm danh theo ngày);
**HyperLogLog** đếm **unique gần đúng** (số user duy nhất) tốn rất ít RAM.

**9. Geospatial (tìm theo vị trí).**
Lệnh **Geo** (`GEOADD`, `GEOSEARCH`) tìm đối tượng trong bán kính → **tài xế/cửa hàng gần bạn** (ride-hailing, giao đồ ăn).

**10. Full-page / fragment cache & feature flag.**
Cache HTML render sẵn; lưu **feature flag / config** đọc nhiều để bật/tắt tính năng tức thời không cần deploy.

> **Khi nào KHÔNG nên dùng Redis:** làm **nguồn chân lý (source of truth)** cho dữ liệu quan trọng cần quan hệ/ACID
> phức tạp (dùng SQL); dữ liệu **lớn hơn RAM** nhiều lần (Redis giữ trong RAM — tốn kém); cần **truy vấn phức tạp
> nhiều chiều** (Redis truy cập chủ yếu theo key).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Redis dùng cho mọi chỗ cần **nhanh / tạm / điều phối đa tiến trình**. Các ứng dụng chính:
- **Cache** (cache-aside + TTL) — phổ biến nhất; **Session store** tập trung cho app stateless; **Rate limiting**
  (`INCR`+TTL).
- **Leaderboard** (Sorted Set); **Distributed lock** (`SET NX PX`/Redlock); **Queue & job** (List/Stream, Laravel/Bull);
  **Pub/Sub** real-time (chat, notification).
- **Counter/analytics** (`INCR`, Bitmap, **HyperLogLog** đếm unique); **Geospatial** (tìm gần theo bán kính); feature flag.
- **Không dùng làm:** source-of-truth cho dữ liệu quan hệ/ACID phức tạp, dataset lớn hơn RAM, truy vấn nhiều chiều
  phức tạp → những việc đó để **SQL**.

[↑ Mục lục](#mục-lục)

<a id="q18-5"></a>
### 18.5. Thực tế hay dùng data type nào của Redis? Implement qua thư viện gì?

#### 📖 Giới thiệu kiến thức

Câu này thường hỏi để kiểm tra **kinh nghiệm thật**: Redis có ~9 data type (mục 18.1) nhưng trong dự án
đời thường **80% công việc chỉ dùng 3–4 loại**, và ứng viên có làm thật sẽ gọi được **tên thư viện client**
mình dùng cùng những cái bẫy của nó.

##### a) Những data type thực sự hay dùng (theo tần suất)

| Hạng | Data type | Dùng để làm gì | Lệnh chủ lực |
|------|-----------|----------------|--------------|
| 1 | **String** | Cache JSON/HTML, **counter** rate-limit, feature flag, **lock** `SET NX EX`, session blob | `SET/GET/SETEX`, `INCR`, `SET NX EX` |
| 2 | **Hash** | Object nhiều field cần **update lẻ từng field** (giỏ hàng, session, đếm theo field) | `HSET/HGETALL/HINCRBY` |
| 3 | **Sorted Set (ZSet)** | **Leaderboard**, **delayed queue** (score = timestamp), **sliding-window rate limit** | `ZADD`, `ZRANGEBYSCORE`, `ZREMRANGEBYSCORE` |
| 4 | **List** | **Job queue FIFO** đơn giản, activity feed gần nhất | `LPUSH` + `BRPOP` |
| 5 | **Set** | Chống trùng (idempotency), tag, user online, quan hệ (giao/hợp) | `SADD/SISMEMBER/SINTER` |
| 6 | **Stream** | Queue **bền + consumer group** (nhiều consumer, có ACK, replay được) | `XADD`, `XREADGROUP`, `XACK` |
| 7 | **Bitmap / HyperLogLog** | Đếm DAU / unique **cực tiết kiệm RAM** (HLL ~12KB cho hàng triệu phần tử, sai số ~0.81%) | `SETBIT/BITCOUNT`, `PFADD/PFCOUNT` |
| 8 | **Geo** | Tìm theo bán kính (tài xế/cửa hàng gần) | `GEOADD/GEOSEARCH` |

> **Pub/Sub không phải data type** mà là một *cơ chế* (`PUBLISH/SUBSCRIBE`, fire-and-forget — mục 18.10).
> Nhiều người nhầm khi kể "các loại Redis".
>
> Ngoài core, **Redis Stack** thêm module: **RedisJSON** (query field trong JSON), **RediSearch**
> (full-text + **vector search** cho RAG), **RedisTimeSeries**, **RedisBloom** (Bloom filter — xem
> mục 16.2). Ở dự án thường vẫn hay tự lưu JSON dạng String để tránh phụ thuộc module.

**Kinh nghiệm chọn type — 3 câu hỏi:** (1) cần **update lẻ 1 field** không → **Hash**, không thì **String**;
(2) cần **sắp thứ tự / lấy theo khoảng** không → **ZSet**; (3) cần **đảm bảo không mất message + nhiều
consumer** không → **Stream**, còn "mất cũng được" → List/Pub-Sub.

##### b) Thư viện implement — 2 tầng

Luôn phân biệt rõ khi trả lời: **client driver** (nói chuyện RESP với Redis) và **abstraction của
framework/thư viện chuyên dụng** (cache, queue, lock... xây trên client đó).

| Hệ sinh thái | Client driver (tầng thấp) | Thư viện/abstraction hay dùng kèm |
|---|---|---|
| **PHP / Laravel** | **phpredis** (C extension — nhanh, mặc định `REDIS_CLIENT=phpredis`) hoặc **predis/predis** (pure PHP, dễ cài, chậm hơn) | `Cache` / `Session` / `Queue` driver `redis`, facade `Redis`, **`Cache::lock()`** (atomic lock), **Horizon** (monitor queue) |
| **Node / NestJS** | **ioredis** (phổ biến nhất: cluster, sentinel, pipeline, Lua) hoặc **node-redis** (client chính chủ, RESP3, hỗ trợ module) | **BullMQ** (`@nestjs/bullmq`) cho queue/delayed/repeat job; `@nestjs/cache-manager` (+ Keyv/redis store); **connect-redis** cho express-session; **`@socket.io/redis-adapter`** scale WebSocket; **rate-limiter-flexible** / throttler storage; **redlock** cho lock phân tán |
| **Python** | **redis-py** (đã gộp `aioredis`, có async) | Celery (broker), `django-redis`, `fastapi-cache` |
| **Java / Spring** | **Lettuce** (mặc định của Spring Data Redis, netty/async), **Jedis** (sync) | **Redisson** — cung cấp `RLock`, Map/Queue phân tán sẵn; Spring Cache |
| **Go** | **go-redis**, hoặc **rueidis** (RESP3 + client-side caching) | asynq (job queue) |

> **Redis vs Valkey:** sau khi Redis đổi license (2024), nhiều nhà cung cấp (vd AWS ElastiCache) đẩy mạnh
> **Valkey** — fork tương thích giao thức, nên **client Redis hiện có vẫn dùng được**; ngoài ra có client
> riêng như `valkey-glide`. Nói được điểm này là điểm cộng.

**Laravel — dùng abstraction cho việc chuẩn, dùng raw client cho ZSet/atomic:**

```php
// .env:  REDIS_CLIENT=phpredis   CACHE_STORE=redis   QUEUE_CONNECTION=redis   SESSION_DRIVER=redis

// 1) Cache — dùng abstraction, không cần biết Redis
$user = Cache::remember("user:$id", 3600, fn () => User::find($id));

// 2) Distributed lock — Cache::lock() bọc sẵn SET NX PX + token an toàn khi release
Cache::lock("order:$id", 10)->block(3, function () use ($id) {
    // chỉ 1 tiến trình vào đây tại một thời điểm (xem mục 18.12)
});

// 3) Việc Redis-đặc-thù (ZSet leaderboard) → dùng thẳng facade Redis
Redis::zadd('leaderboard', 250, "user:$id");
$top10 = Redis::zrevrange('leaderboard', 0, 9, ['withscores' => true]);

// 4) Nhiều lệnh 1 vòng network → pipeline (giảm RTT, không phải transaction)
Redis::pipeline(function ($pipe) use ($ids) {
    foreach ($ids as $i) $pipe->hgetall("user:$i");
});
```

**NestJS — ioredis + BullMQ + Lua cho thao tác atomic nhiều bước:**

```ts
// redis.provider.ts — 1 client dùng chung cả app (KHÔNG tạo client mỗi request)
export const RedisProvider = {
  provide: 'REDIS',
  useFactory: () => new Redis({ host: process.env.REDIS_HOST, maxRetriesPerRequest: 3 }),
};

// Rate limit sliding-window bằng ZSet + Lua: nhiều lệnh nhưng chạy ATOMIC trong 1 script
const SLIDING_WINDOW = `
  redis.call('ZREMRANGEBYSCORE', KEYS[1], 0, ARGV[1] - ARGV[2])  -- bỏ record ngoài cửa sổ
  local count = redis.call('ZCARD', KEYS[1])
  if count >= tonumber(ARGV[3]) then return 0 end
  redis.call('ZADD', KEYS[1], ARGV[1], ARGV[4])
  redis.call('PEXPIRE', KEYS[1], ARGV[2])
  return 1`;

async allow(userId: string) {
  const now = Date.now();
  return await this.redis.eval(SLIDING_WINDOW, 1, `rl:${userId}`, now, 60_000, 100, `${now}-${randomUUID()}`);
}

// Queue: BullMQ (bên trong dùng List + ZSet + Hash + Stream của Redis)
@Processor('mail')
export class MailProcessor extends WorkerHost {
  async process(job: Job) { /* ... */ }
}
await this.mailQueue.add('welcome', { userId }, { delay: 60_000, attempts: 3 });
```

##### c) Bẫy thực tế khi dùng thư viện (hay được hỏi tiếp)

- **Tạo client mới mỗi request** → cạn connection/`max_clients`. Client Redis phải là **singleton**
  (Laravel/Nest đều quản lý qua container). Serverless/Lambda thì khai báo client **ngoài** handler.
- **Client đang `SUBSCRIBE` không dùng được cho lệnh thường** → Pub/Sub cần **connection riêng** (BullMQ,
  socket.io adapter vì thế tự tạo thêm connection — đừng ngạc nhiên khi thấy số connection gấp 2–3).
- **BullMQ + ioredis** yêu cầu `maxRetriesPerRequest: null` cho connection của worker (vì `BRPOPLPUSH`
  block lâu) — cấu hình sai sẽ lỗi ngay khi start.
- **Serializer khác nhau giữa thư viện**: phpredis + igbinary/Laravel prefix ghi ra format mà service
  Node đọc bằng `JSON.parse` **không hiểu**. Chia sẻ cache giữa nhiều ngôn ngữ thì **chốt JSON thuần** và
  thống nhất **prefix key**.
- **Đừng `KEYS *` trên production** (O(N), block single-thread) → dùng `SCAN`/`scanStream` của ioredis.
- **Redis Cluster**: lệnh multi-key (`MGET`, Lua nhiều KEYS, transaction) chỉ chạy khi các key **cùng
  slot** → phải dùng **hash tag** `user:{123}:profile`; client cluster của thư viện không tự gộp giúp
  (mục 18.16).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Thực tế chỉ dùng vài loại:** **String** (cache JSON, counter `INCR`, lock `SET NX EX`), **Hash**
  (object cần update lẻ field), **Sorted Set** (leaderboard, delayed queue, sliding-window rate limit),
  **List** (queue FIFO `LPUSH`/`BRPOP`). Thêm **Set** (chống trùng), **Stream** (queue bền có consumer
  group + ACK), **Bitmap/HyperLogLog** (đếm DAU/unique tiết kiệm RAM), **Geo** (tìm gần).
- **Cách chọn type:** cần update lẻ field → Hash; cần thứ tự/lấy theo khoảng → ZSet; cần không mất
  message + nhiều consumer → Stream; còn lại → String/List. *Pub/Sub là cơ chế, không phải data type.*
- **Thư viện chia 2 tầng:** client driver + abstraction.
  - **PHP/Laravel:** **phpredis** (C ext, nhanh — mặc định) hoặc **predis** (pure PHP); dùng qua
    `Cache`/`Session`/`Queue` driver `redis`, `Cache::lock()`, Horizon.
  - **Node/NestJS:** **ioredis** (hoặc **node-redis** chính chủ) + **BullMQ** (`@nestjs/bullmq`) cho job,
    `@nestjs/cache-manager` cho cache, `connect-redis` cho session, `@socket.io/redis-adapter` để scale
    WebSocket, `redlock` cho lock phân tán.
  - Khác: **redis-py** (Python), **Lettuce/Jedis/Redisson** (Java), **go-redis** (Go).
- **Nguyên tắc:** việc chuẩn (cache/session/queue) → dùng **abstraction của framework**; việc Redis-đặc-thù
  (ZSet, atomic nhiều bước) → **client thẳng + Lua `EVAL`** để giữ atomic.
- **Bẫy nên nói thêm:** client phải **singleton** (đừng tạo mỗi request), Pub/Sub cần **connection
  riêng**, BullMQ cần `maxRetriesPerRequest: null`, **serializer/prefix khác nhau** giữa PHP↔Node, tránh
  `KEYS *` (dùng `SCAN`), Cluster cần **hash tag** cho lệnh multi-key.

[↑ Mục lục](#mục-lục)

<a id="q18-6"></a>
### 18.6. Phân biệt 2 thư viện `redis` (node-redis) và `ioredis`

#### 📖 Giới thiệu kiến thức

Đây là 2 client Redis phổ biến nhất của **Node.js**. Cả hai đều nói **cùng giao thức RESP** với **cùng một
Redis server** — nên **không có "cái nào mạnh hơn" về mặt tính năng Redis**; khác biệt nằm ở **API, hành vi
kết nối và hệ sinh thái đi kèm**.

| | **`redis` (node-redis)** | **`ioredis`** |
|---|---|---|
| **Xuất thân** | Client **chính chủ**, do Redis maintain | Ban đầu là client cộng đồng, nay cũng nằm dưới tổ chức Redis; là **de-facto** trong ecosystem Node nhiều năm |
| **Kết nối** | **Phải gọi `await client.connect()`** — quên là dính `ClientClosedError` | **Tự connect ngay khi `new Redis()`** (muốn hoãn thì `lazyConnect: true`) |
| **Tên lệnh** | **camelCase**: `hGetAll`, `zAdd`, `expire` | **Giống hệt Redis (thường)**: `hgetall`, `zadd` — copy lệnh từ `redis-cli` sang dùng luôn |
| **Pipeline** | **Tự động gộp** các lệnh phát trong cùng một tick của event loop | Mặc định gửi từng lệnh; muốn tự động thì bật `enableAutoPipelining: true`, hoặc gom tay bằng `pipeline()` |
| **Cluster / Sentinel** | `createCluster()` | `new Redis.Cluster()` / cấu hình `sentinels` — **rất trưởng thành**, nhiều hệ thống lớn dùng |
| **Tiện ích "đời thường"** | Không có `keyPrefix` sẵn (tự nối prefix) | Có sẵn **`keyPrefix`**, **`retryStrategy`**, `maxRetriesPerRequest`, `enableReadyCheck`, **`scanStream`** (Node stream cho `SCAN`) |
| **Lua** | `EVAL`/`SCRIPT LOAD` trực tiếp | Có thêm **`defineCommand()`** — biến script Lua thành **method riêng** của client, gọi như lệnh Redis |
| **RESP3, client-side caching, Redis Stack** | Ưu tiên tính năng mới: **RESP3**, **client-side caching**, typed command cho **RedisJSON/RediSearch** (`client.json.set`, `client.ft.search`), **connection pool** ở v5 | Chạy tốt với lệnh cơ bản; tính năng mới thường về node-redis trước |
| **Hệ sinh thái** | `@keyv/redis` / `cache-manager` (dùng trong `@nestjs/cache-manager` bản mới), `connect-redis` | **BullMQ bắt buộc dùng ioredis**, `connect-redis`, `@socket.io/redis-adapter`, `rate-limiter-flexible` |
| **Kiểu dữ liệu TypeScript** | Typing chặt hơn, có type mapping | Typing lỏng hơn (hay trả `string \| null`, tự parse) |

**Cùng một việc, viết bằng 2 thư viện:**

```ts
// ---------- node-redis (package: redis) ----------
import { createClient } from 'redis';
const client = createClient({ url: 'redis://localhost:6379' });
client.on('error', (e) => logger.error(e));
await client.connect();                       // ⚠ BẮT BUỘC, khác ioredis

await client.set('user:1', JSON.stringify(u), { EX: 3600 });   // option kiểu object
const profile = await client.hGetAll('user:1:profile');        // camelCase
for await (const key of client.scanIterator({ MATCH: 'sess:*' })) { /* ... */ }

// ---------- ioredis ----------
import Redis from 'ioredis';
const redis = new Redis({ host: 'localhost', port: 6379, keyPrefix: 'app:' }); // connect ngay

await redis.set('user:1', JSON.stringify(u), 'EX', 3600);      // đúng thứ tự đối số như redis-cli
const profile = await redis.hgetall('user:1:profile');         // lowercase như lệnh Redis
redis.scanStream({ match: 'sess:*' }).on('data', (keys) => { /* ... */ });
```

**Điểm khác dễ gây bug nhất — thời điểm kết nối:**

```ts
// ioredis: dùng được ngay, lệnh phát trước khi connect xong được xếp vào offline queue rồi chạy sau
const redis = new Redis();
await redis.get('k');            // ✅ chạy được

// node-redis: quên connect() là lỗi runtime, không phải lỗi compile
const client = createClient();
await client.get('k');           // ❌ ClientClosedError: The client is closed
```

**Chọn cái nào?**

- **Chọn `ioredis` khi:** dự án dùng **BullMQ** (không có lựa chọn khác), cần **Cluster/Sentinel** nhiều,
  thích API giống hệt `redis-cli`, cần `keyPrefix`/`scanStream`/`defineCommand`, hoặc kế thừa codebase cũ.
- **Chọn `redis` (node-redis) khi:** dự án **mới**, muốn client **chính chủ** cập nhật lâu dài, cần
  **RESP3 / client-side caching**, dùng **Redis Stack** (JSON/Search/vector) với command có type, hoặc
  đi theo `@nestjs/cache-manager`/Keyv.
- **Đừng dùng cả hai trong một service**: 2 bộ connection pool, 2 kiểu cấu hình, 2 cách serialize → khó
  giám sát. Nếu đã có BullMQ thì thường thống nhất luôn **ioredis** cho toàn app.

> **Lưu ý chuyển dịch:** tính năng mới của Redis (RESP3, client-side caching, module) được ưu tiên đưa vào
> **node-redis** — nên với dự án mới, node-redis là lựa chọn "an toàn về lâu dài", còn **ioredis vẫn là bắt
> buộc nếu dùng BullMQ**. Nếu hạ tầng chuyển sang **Valkey**, cả hai vẫn chạy (tương thích RESP), ngoài ra
> có client riêng như `valkey-glide` / `iovalkey`.

#### Bẫy chung cho cả hai (hay bị hỏi tiếp)

- Client phải là **singleton** — đừng `new Redis()` trong mỗi request/handler (mục 18.5).
- Client đang **`SUBSCRIBE` không dùng cho lệnh thường** được → Pub/Sub cần **connection riêng** (BullMQ và
  socket.io adapter vì thế tự mở thêm connection).
- **BullMQ + ioredis:** connection của worker cần **`maxRetriesPerRequest: null`** (vì `BRPOPLPUSH` block lâu).
- Luôn gắn **`client.on('error', ...)`** — thiếu handler thì lỗi socket có thể làm **crash process**.
- **Cluster**: lệnh multi-key/Lua nhiều KEYS cần **hash tag** `{}` để các key cùng slot (mục 18.16).

#### 🎯 Tóm tắt trả lời phỏng vấn

- Cả hai là **client Node.js nói cùng giao thức RESP tới cùng Redis server** → khác nhau ở **API, hành vi
  kết nối và hệ sinh thái**, không phải ở "sức mạnh Redis".
- **`redis` (node-redis)** = client **chính chủ**: lệnh **camelCase** (`hGetAll`), **bắt buộc
  `await client.connect()`**, **tự động pipeline** trong cùng tick, ưu tiên tính năng mới (**RESP3,
  client-side caching**, typed command cho **RedisJSON/RediSearch**, connection pool).
- **`ioredis`** = client phổ biến nhất trong thực tế: **tự connect khi khởi tạo**, tên lệnh **giống hệt
  `redis-cli`** (`hgetall`), mạnh về **Cluster/Sentinel**, có sẵn **`keyPrefix`, `retryStrategy`,
  `scanStream`, `defineCommand`** cho Lua.
- **Chọn:** dùng **BullMQ** → **bắt buộc ioredis**; dự án mới/cần RESP3 + Redis Stack → **node-redis**;
  **không trộn cả hai** trong một service.
- **Khác biệt hay gây bug nhất:** node-redis quên `connect()` → **`ClientClosedError`**, còn ioredis kết nối
  ngay và **xếp lệnh vào offline queue** trong lúc chờ.

[↑ Mục lục](#mục-lục)

<a id="q18-7"></a>
### 18.7. Redis làm delayed job queue (hàng đợi job có trì hoãn) thế nào?

#### 📖 Giới thiệu kiến thức

Nhiều bài toán cần chạy job **ở tương lai**, không phải ngay: gửi email nhắc sau 1 giờ, hủy đơn chưa thanh toán
sau 15 phút, retry job lỗi sau 30s. Đây là **delayed (scheduled) job**. Redis không có "hẹn giờ job" sẵn, nhưng
dựng được rất gọn nhờ **Sorted Set (ZSet)**.

**1. Ý tưởng cốt lõi: ZSet với `score` = thời điểm chạy.**
Bỏ job vào một **Sorted Set**, dùng **score = timestamp lúc job cần chạy** (`now + delay`). Vì ZSet **luôn sắp
xếp theo score**, job nào tới hạn sớm nhất luôn nằm đầu → chỉ cần **lấy những job có score ≤ hiện tại**.

```bash
# Đẩy job "gửi mail đơn #5", chạy sau 900s (15 phút)
ZADD delayed_jobs 1720000900 "job:send_mail:order5"

# Worker chạy vòng lặp: lấy các job ĐÃ TỚI HẠN (score <= now)
ZRANGEBYSCORE delayed_jobs -inf 1720000000 LIMIT 0 10
# → xử lý xong thì xóa khỏi ZSet:
ZREM delayed_jobs "job:send_mail:order5"
```

**2. Worker (polling loop).** Một tiến trình nền lặp:
1. Lấy `now = thời gian hiện tại`.
2. `ZRANGEBYSCORE delayed_jobs -inf now` → các job đến hạn.
3. Với mỗi job: **chuyển sang hàng đợi "ready"** (List) để worker thường xử lý, rồi `ZREM` khỏi ZSet.
4. Ngủ ngắn (vd 1s) rồi lặp.

**3. Vấn đề race condition (nhiều worker cùng lấy 1 job).** Nếu nhiều worker cùng `ZRANGEBYSCORE` rồi `ZREM`, hai
worker có thể lấy **trùng** một job. Cách xử lý **atomic**:
- Dùng **Lua script** gói `ZRANGEBYSCORE` + `ZREM` trong **một lệnh nguyên tử** (Redis chạy Lua đơn luồng, không
  ai chen giữa).
- Hoặc **`ZPOPMIN`** (lấy & xóa phần tử score nhỏ nhất trong 1 lệnh atomic) rồi kiểm tra score có ≤ now không.

```lua
-- Lua: lấy tối đa N job đã tới hạn và xóa luôn, atomic
local jobs = redis.call('ZRANGEBYSCORE', KEYS[1], '-inf', ARGV[1], 'LIMIT', 0, ARGV[2])
if #jobs > 0 then redis.call('ZREM', KEYS[1], unpack(jobs)) end
return jobs
```

**4. Cách Laravel làm (thực tế).** Queue driver `redis` của Laravel dùng đúng mô hình này:
- `dispatch($job)->delay(now()->addMinutes(15))` → Laravel `ZADD` job vào ZSet **`queues:default:delayed`** với
  score = timestamp chạy.
- Worker (`queue:work`) mỗi vòng **migrate** các job đã tới hạn từ ZSet `:delayed` sang **List `queues:default`**
  (hàng đợi sẵn sàng) bằng **Lua script atomic**, rồi `BRPOP`/`LPOP` để xử lý.
- Có thêm ZSet **`:reserved`** để theo dõi job đang xử lý (retry nếu worker chết giữa chừng — visibility timeout).

**5. Lựa chọn khác — Redis Stream.** Với hàng đợi cần **bền, nhiều consumer, ack/retry** bài bản hơn, dùng
**Stream** (`XADD`/`XREADGROUP`/`XACK`) + consumer group; phần delay vẫn thường ghép ZSet để "hẹn giờ" rồi mới đẩy vào Stream.

> **Vì sao ZSet hợp cho delay:** thao tác thêm/lấy theo score đều **O(log N)**, luôn có sẵn phần tử tới hạn ở đầu,
> và Redis atomic giúp nhiều worker chia việc an toàn — không cần cron quét DB mỗi phút.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Delayed job = chạy job ở tương lai** (nhắc email, hủy đơn sau 15', retry). Redis dựng bằng **Sorted Set với
  `score = timestamp lúc chạy`** (`now + delay`).
- **Worker polling:** lặp lấy job **đã tới hạn** bằng `ZRANGEBYSCORE delayed -inf now` → chuyển sang List "ready"
  → `ZREM`. ZSet luôn sắp theo score nên job tới hạn nằm đầu.
- **Chống race** (nhiều worker lấy trùng): gói lấy+xóa trong **Lua script atomic** hoặc dùng **`ZPOPMIN`**.
- **Laravel** queue driver `redis` làm y hệt: `->delay()` `ZADD` vào ZSet `:delayed`, worker **migrate atomic**
  (Lua) sang List sẵn sàng rồi `BRPOP`; ZSet `:reserved` để retry job chết giữa chừng.
- Cần bền/nhiều consumer/ack → cân nhắc **Redis Stream** (`XREADGROUP` + consumer group).

[↑ Mục lục](#mục-lục)

<a id="q18-8"></a>
### 18.8. Delayed job queue khác job queue thường khi xử lý ở Redis thế nào?

#### 📖 Giới thiệu kiến thức

Hai loại queue khác nhau **cơ bản ở cấu trúc dữ liệu và cách worker lấy job**, kéo theo khác về hiệu năng và độ
phức tạp.

**1. Job queue thường (immediate) — dùng List, lấy theo FIFO.**
Job đẩy vào là **sẵn sàng chạy ngay**. Redis dùng **List**: `LPUSH` để đẩy, **`BRPOP`** (blocking pop) để worker
lấy. Worker **không cần biết thời gian** — cứ có job là xử lý.

```bash
LPUSH queue:emails "job:send:1"      # producer đẩy job
BRPOP queue:emails 5                  # worker CHỜ (blocking) tới khi có job, tối đa 5s
```

- **Lấy job: O(1)**, `BRPOP` **blocking** → worker **ngủ** cho tới khi có job, **không cần polling** → gần như
  không tốn CPU khi rảnh, độ trễ nhận job ~tức thì.
- Thứ tự: **FIFO** (hoặc LIFO tùy đầu pop). Không có khái niệm "khi nào chạy".

**2. Delayed job queue — dùng Sorted Set, lấy theo thời điểm (score).**
Job có **thời điểm được phép chạy** (`now + delay`) → **không** thể dùng List (List không sắp theo thời gian).
Phải dùng **Sorted Set** với **score = timestamp chạy**, worker chỉ lấy job **đã tới hạn** (`score ≤ now`).

```bash
ZADD queue:delayed 1720000900 "job:cancel_order:5"    # chạy lúc timestamp 1720000900
ZRANGEBYSCORE queue:delayed -inf <now> LIMIT 0 10       # lấy job đã tới hạn
```

**3. Khác biệt then chốt — vì sao delayed phức tạp hơn:**

| Tiêu chí | Queue thường (List) | Delayed queue (Sorted Set) |
|----------|--------------------|----------------------------|
| Cấu trúc | **List** | **Sorted Set** (score = thời điểm chạy) |
| Điều kiện lấy job | Có job là lấy | Chỉ job **đã tới hạn** (`score ≤ now`) |
| Cách worker chờ | **`BRPOP` blocking** — ngủ tới khi có job | **Polling** định kỳ (vd mỗi 1s) vì phải xét thời gian |
| Độ phức tạp thao tác | `LPUSH`/`BRPOP` **O(1)** | `ZADD`/`ZRANGEBYSCORE` **O(log N)** |
| Lấy + xóa atomic | `BRPOP` đã atomic sẵn | Phải tự lo: **Lua** hoặc `ZPOPMIN` (tránh 2 worker lấy trùng) |
| CPU khi rảnh | ~0 (blocking) | Tốn nhẹ do **poll lặp** |
| "Chạy đúng giờ" | Không có khái niệm | Có — nhưng độ chính xác phụ thuộc **chu kỳ poll** |

**4. Điểm quan trọng nhất về mặt xử lý:**
- **Blocking vs polling.** Queue thường `BRPOP` **chặn** → nhận job **tức thì**, không lãng phí CPU. Delayed queue
  **buộc phải poll** (kiểm tra định kỳ "đã có job nào tới hạn chưa?") vì điều kiện lấy phụ thuộc **đồng hồ**, không
  phải "có/không có phần tử". Chu kỳ poll quyết định **độ trễ tối đa** (poll 1s → job có thể chạy trễ tới ~1s).
- **Atomicity.** `BRPOP` tự nguyên tử (một worker lấy được thì worker khác không). Delayed queue `ZRANGEBYSCORE`
  rồi `ZREM` là **2 bước** → nhiều worker dễ lấy trùng → **phải gói Lua** hoặc dùng `ZPOPMIN`.

**5. Thực tế: hai loại thường ghép chung.** Delayed queue thường chỉ là **tầng "hẹn giờ"** phía trước. Một worker
điều phối **poll** Sorted Set, thấy job tới hạn thì **chuyển sang List "ready"** (`LPUSH`), rồi các worker xử lý
dùng **`BRPOP`** như queue thường. Laravel làm đúng vậy: ZSet `:delayed` → migrate (Lua atomic) → List `queues:default`
→ `BRPOP` (xem mục 18.7).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Queue thường:** job **chạy ngay** → dùng **List** (`LPUSH`/**`BRPOP`**). `BRPOP` **blocking**: worker ngủ tới
  khi có job, nhận **tức thì**, thao tác **O(1)**, atomic sẵn.
- **Delayed queue:** job có **thời điểm chạy** → dùng **Sorted Set** với **score = timestamp** (`now + delay`),
  chỉ lấy job **`score ≤ now`**. Thao tác **O(log N)**.
- **Khác cốt lõi khi xử lý:** queue thường **blocking (`BRPOP`)** nhận job tức thì; delayed **phải polling định kỳ**
  vì điều kiện lấy phụ thuộc **thời gian** → chu kỳ poll = độ trễ tối đa, và tốn CPU nhẹ khi rảnh.
- **Atomicity:** `BRPOP` nguyên tử sẵn; delayed phải tự chống lấy trùng bằng **Lua script** hoặc **`ZPOPMIN`**.
- **Thực tế ghép cả hai:** ZSet hẹn giờ → tới hạn thì migrate sang List → `BRPOP` xử lý (mô hình Laravel).

[↑ Mục lục](#mục-lục)

<a id="q18-9"></a>
### 18.9. BullMQ hoạt động thế nào bên trong Redis?

#### 📖 Giới thiệu kiến thức

**BullMQ** là thư viện job queue phổ biến nhất của Node (chạy trên **ioredis** — mục 18.6). Redis **không có
kiểu "queue" dựng sẵn**, nên BullMQ **tự dựng queue từ các data type cơ bản** đúng theo mô hình ở mục 18.7–18.8:
**List cho job sẵn sàng, Sorted Set cho job hẹn giờ, Hash cho nội dung job**, và **mọi chuyển trạng thái đều
là một Lua script atomic**. Hiểu phần này là hiểu luôn cách viết một queue nghiêm túc.

##### a) Bản đồ key trong Redis

Mọi key đều mang prefix `bull:<tên queue>:`. Với queue tên `mail`:

| Key | Data type | Vai trò |
|---|---|---|
| `bull:mail:id` | String | Bộ đếm `INCR` sinh **job id** tăng dần |
| `bull:mail:<jobId>` | **Hash** | **Nội dung job**: `name`, `data` (JSON), `opts`, `timestamp`, `attemptsMade`, `processedOn`, `finishedOn`, `returnvalue`, `failedReason`, `stacktrace` |
| `bull:mail:wait` | **List** | Job **sẵn sàng chạy** (chỉ chứa **jobId**, không chứa data) |
| `bull:mail:active` | **List** | Job **đang được worker xử lý** |
| `bull:mail:delayed` | **ZSet** | Job **hẹn giờ**, `score` ≈ timestamp tới hạn (kèm vài bit thứ tự để giữ FIFO khi trùng mili-giây) |
| `bull:mail:prioritized` | **ZSet** | Job có **priority** (score = mức ưu tiên) |
| `bull:mail:completed` / `:failed` | **ZSet** | Lịch sử job xong/lỗi (score = thời điểm kết thúc) — để `removeOnComplete/Fail` trim theo **số lượng hoặc tuổi** |
| `bull:mail:<jobId>:lock` | String + **TTL** | **Lock của worker** đang giữ job (mặc định `lockDuration` 30s) |
| `bull:mail:stalled` | Set | Job bị nghi **stalled** (worker chết giữa chừng) |
| `bull:mail:paused` | List | Nơi giữ job khi queue bị `pause()` |
| `bull:mail:meta` | Hash | Cấu hình queue (đang pause?, version...) |
| `bull:mail:events` | **Stream** | Sự kiện `added/active/completed/failed/progress` (`XADD` + `maxlen`) cho `QueueEvents` — chính là Redis Stream ở mục 18.11 |
| `bull:mail:limiter` | String + TTL | Đếm cho **rate limit** (giới hạn N job / khoảng thời gian) |

> **Job data nằm ở Hash, các List/ZSet chỉ giữ jobId.** Nhờ vậy job đi qua nhiều trạng thái mà **không phải
> copy payload**, và trạng thái nào cũng tra được chi tiết bằng 1 lệnh `HGETALL`.

##### b) Vòng đời một job

```text
add()                     ┌──────────── retry (backoff) ────────────┐
  │                       │                                          │
  ├─ có delay ──► [delayed ZSet] ──(tới hạn)──► [wait List] ──► [active List] ──► [completed ZSet]
  ├─ có priority ► [prioritized ZSet] ─────────►                  │
  └─ thường ─────────────────────────────────► [wait List]        └── hết attempts ──► [failed ZSet]

lock hết hạn (worker chết) ──► [stalled Set] ──► quay lại [wait]  (tối đa maxStalledCount lần)
```

- **`queue.add()`** → 1 Lua script: `INCR` lấy id → `HSET` job hash → `LPUSH` vào `wait` (hoặc `ZADD` vào
  `delayed`/`prioritized`) → `XADD` event `added`. **Một lệnh atomic**, không có trạng thái nửa vời.
- **Worker lấy job** → Lua `moveToActive`: chuyển jobId từ `wait` sang `active`, **`SET lock` kèm TTL**, tăng
  `attemptsMade`, ghi `processedOn`, `XADD` event `active` — tất cả trong một script nên **hai worker không
  thể cùng nhận một job** (Redis đơn luồng, xem mục 18.2).
- **Xong việc** → Lua `moveToFinished`: xoá khỏi `active`, xoá `lock`, `ZADD` vào `completed`, ghi
  `returnvalue`, trim lịch sử theo `removeOnComplete`, `XADD` event `completed`.
- **Lỗi** → còn `attempts` thì tính **backoff** rồi `ZADD` lại vào **`delayed`** (retry chính là một delayed
  job); hết attempts → vào **`failed`** kèm `failedReason` + `stacktrace` (đây chính là **DLQ**).

##### c) Ba cơ chế cốt lõi nên nói khi phỏng vấn

**1. Lua script = atomicity.** Mỗi chuyển trạng thái đụng **nhiều key** (list + hash + zset + stream). BullMQ
gói chúng vào Lua (`SCRIPT LOAD` → gọi bằng `EVALSHA`), Redis chạy script **nguyên khối** → không bao giờ có
cảnh "job vừa ở `wait` vừa ở `active`", cũng không cần transaction/lock ở tầng app.

**2. Lock + stalled detection = chống mất job khi worker chết.** Worker giữ `<jobId>:lock` có **TTL** và
**tự gia hạn** (renew) trong lúc xử lý. Nếu tiến trình chết/bị OOM kill, không ai renew → **lock hết hạn**;
bộ **stalled checker** phát hiện job nằm trong `active` mà **không còn lock** → đưa **về lại `wait`** để worker
khác chạy (tối đa `maxStalledCount`, mặc định 1, rồi mới cho `failed`).
→ Hệ quả quan trọng: BullMQ giao **at-least-once**, nên **job phải idempotent** (xem `Common/resolve-problem.md`).

**3. Đánh thức worker không cần polling nóng.** Bản cũ dùng `BRPOPLPUSH wait → active` (blocking, giống mô
hình ở mục 18.8). Từ **v5**, BullMQ dùng cơ chế **"marker"**: worker chặn trên một key báo hiệu
(`BZPOPMIN`) rồi mới chạy Lua `moveToActive` — nhờ vậy xử lý thống nhất được cả **wait + prioritized +
delayed** mà vẫn **ngủ khi rảnh** (không đốt CPU). Với job `delayed`, worker tính **thời điểm gần nhất** để
hẹn timer, tới hạn thì chạy script chuyển sang `wait` — không quét ZSet liên tục.

```lua
-- Tinh thần của moveToActive (rút gọn): nhiều key, một script → atomic
local jobId = redis.call('RPOPLPUSH', KEYS[1], KEYS[2])          -- wait -> active
if jobId then
  redis.call('SET', KEYS[3] .. jobId .. ':lock', ARGV[1], 'PX', ARGV[2])  -- lock + TTL
  redis.call('HINCRBY', KEYS[3] .. jobId, 'attemptsMade', 1)
  redis.call('XADD', KEYS[4], '*', 'event', 'active', 'jobId', jobId)
end
return jobId
```

```ts
// Nhìn thấy tận mắt: sau khi add 1 job delay 60s
await queue.add('welcome', { userId: 7 }, { delay: 60_000, attempts: 3 });

// redis-cli (dùng SCAN, đừng KEYS * trên prod):
//   bull:mail:id            -> "1"
//   bull:mail:1             -> Hash: name=welcome, data={"userId":7}, opts={"attempts":3,...}
//   bull:mail:delayed       -> ZSet: member "1", score ≈ (now+60000) << 12
//   bull:mail:events        -> Stream: event=added, jobId=1
```

##### d) Hệ quả vận hành (phần ăn điểm)

- **`maxmemory-policy` phải là `noeviction`.** Redis được cấu hình evict key (`allkeys-lru`) sẽ **xoá mất job
  hash/list** → **mất job âm thầm**. Đây là lỗi cấu hình kinh điển khi dùng chung Redis cache với queue →
  **nên tách instance/DB riêng cho queue**.
- **Queue nằm hoàn toàn trong RAM** → phải bật `removeOnComplete`/`removeOnFail` (theo count hoặc age), nếu
  không ZSet `completed` phình vô hạn. Muốn không mất job khi Redis restart thì cần **AOF** (mục 18.14).
- **Redis Cluster:** một queue đụng nhiều key → phải nằm **cùng slot** → đặt prefix có **hash tag**, vd
  `new Queue('mail', { prefix: '{bull}' })` (mục 18.16).
- **Số connection nhân lên:** mỗi Worker/QueueEvents mở connection riêng (blocking + subscriber), nên đừng
  ngạc nhiên khi thấy connection gấp 2–3 lần số instance; connection của worker cần
  **`maxRetriesPerRequest: null`**.
- **`lockDuration` phải lớn hơn thời gian xử lý thật** (hoặc gọi `job.updateProgress()`/renew), nếu không job
  đang chạy bình thường vẫn bị coi là **stalled** và **chạy lặp** — nguyên nhân phổ biến của "email gửi 2 lần".
- **Concurrency** của Worker là số job xử lý **đồng thời trong một tiến trình Node** (không phải số process);
  CPU-bound thì dùng **sandboxed processor** (chạy file riêng) để không chặn event loop.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Redis **không có kiểu queue sẵn** → BullMQ **tự dựng** bằng data type cơ bản, prefix `bull:<queue>:`:
  **Hash** giữ **nội dung job**, **List `wait`/`active`** giữ **jobId** sẵn sàng/đang chạy, **ZSet `delayed`**
  (score = thời điểm chạy) và **ZSet `prioritized`**, **ZSet `completed`/`failed`** làm lịch sử, **Stream
  `events`** phát sự kiện, cùng key **`<jobId>:lock` có TTL**.
- **Vòng đời:** `add` → `wait` (hoặc `delayed` nếu có delay) → `active` → `completed`; lỗi mà còn `attempts`
  thì **quay lại `delayed` theo backoff**, hết attempts thì vào **`failed` (DLQ)**.
- **3 cơ chế cốt lõi:**
  1. **Mọi chuyển trạng thái là một Lua script** → atomic trên Redis đơn luồng, không có job "ở hai nơi",
     không cần lock tầng app.
  2. **Lock có TTL + stalled checker:** worker renew lock khi đang chạy; worker chết → lock hết hạn → job
     **về lại `wait`**. Vì vậy BullMQ là **at-least-once** → **job phải idempotent**.
  3. **Worker ngủ khi rảnh:** bản cũ `BRPOPLPUSH`, từ v5 dùng **marker + `BZPOPMIN`** rồi chạy Lua; job delay
     được hẹn timer theo job gần nhất thay vì poll liên tục.
- **Vận hành phải nhớ:** Redis của queue để **`maxmemory-policy noeviction`** (evict = **mất job**) và nên
  **tách riêng khỏi Redis cache**; bật **`removeOnComplete/Fail`** kẻo phình RAM; **`lockDuration` > thời gian
  xử lý** kẻo job bị coi là stalled và **chạy lặp**; Cluster cần **hash tag** trong prefix.

[↑ Mục lục](#mục-lục)

<a id="q18-10"></a>
### 18.10. Redis Pub/Sub hoạt động thế nào?

#### 📖 Giới thiệu kiến thức

**Pub/Sub (Publish/Subscribe)** là mô hình nhắn tin **1 người gửi → nhiều người nhận** theo **kênh (channel)**,
trong đó **người gửi và người nhận không biết nhau** (decoupled). Redis cài sẵn Pub/Sub rất nhẹ để làm **real-time
messaging**: chat, notification, live update, đồng bộ giữa nhiều server.

**1. Ba khái niệm:**
- **Publisher:** gửi message vào một **channel** bằng `PUBLISH channel message`.
- **Subscriber:** đăng ký nghe một/nhiều channel bằng `SUBSCRIBE channel` (hoặc `PSUBSCRIBE news.*` theo pattern).
- **Channel:** "kênh" đặt tên; message chỉ đẩy tới **các subscriber đang nghe channel đó ngay lúc publish**.

```bash
# Terminal A — subscriber: nghe channel "news"
SUBSCRIBE news
# Terminal B — publisher: gửi vào "news"
PUBLISH news "Có bài viết mới"     # trả về SỐ subscriber đã nhận (vd 1)
# → mọi subscriber đang SUBSCRIBE "news" nhận ngay message
```

**2. Cơ chế hoạt động (điểm mấu chốt):**
- Khi `PUBLISH`, Redis **duyệt danh sách subscriber của channel đó** và **đẩy message ngay lập tức** tới từng
  người đang kết nối. Message **không được lưu lại** ở đâu cả.
- **Fire-and-forget:** subscriber nào **không online tại thời điểm publish** thì **mất message** — không có replay,
  không có hàng đợi tồn đọng. Đây là khác biệt lớn nhất so với queue/Stream.
- **Fan-out:** một message tới **tất cả** subscriber (broadcast), khác queue (mỗi job chỉ **một** worker lấy).

**3. Pattern subscribe.** `PSUBSCRIBE` cho phép nghe theo mẫu:
```bash
PSUBSCRIBE order.*      # nhận mọi channel bắt đầu bằng "order." (order.created, order.paid…)
```

**4. Hạn chế cần biết (hay bị hỏi vặn):**
- **Không bền (no persistence):** mất message nếu subscriber offline hoặc mạng chớp tắt → **không dùng** cho việc
  "không được mất" (thanh toán, job quan trọng).
- **At-most-once delivery:** không có ACK, không retry.
- **Không backpressure tốt:** subscriber chậm/nghẽn → Redis buffer đầy có thể **ngắt kết nối** subscriber đó.
- **Trong Redis Cluster:** `PUBLISH` cổ điển được **phát tới toàn cluster** (mọi node) để subscriber ở node bất
  kỳ đều nhận → tốn băng thông. Redis 7 thêm **Sharded Pub/Sub** (`SSUBSCRIBE`/`SPUBLISH`) giới hạn message theo
  shard để scale tốt hơn.

**5. Pub/Sub vs Stream vs Queue (List) — chọn cái nào:**

| | **Pub/Sub** | **Stream** (`XADD`/`XREADGROUP`) | **Queue** (List) |
|---|---|---|---|
| Kiểu giao | **Broadcast** (fan-out, nhiều nhận) | Broadcast + consumer group | **1 job → 1 worker** |
| Lưu message | **Không** (mất nếu offline) | **Có** (log bền, đọc lại được) | Có (tới khi bị pop) |
| ACK/retry | Không (at-most-once) | **Có** (`XACK`, pending list) | Cần tự làm |
| Dùng khi | Real-time, **chấp nhận mất** (chat, live, notify) | Event bền, nhiều consumer, cần replay | Xử lý job nền |

**6. Ứng dụng điển hình:** chat/room, notification real-time, **broadcast WebSocket** giữa nhiều instance server
(mỗi server subscribe kênh chung để đồng bộ sự kiện), invalidate cache đồng loạt, cập nhật dashboard live.

> **Chốt bản chất:** Pub/Sub = **nhắn tin real-time, nhẹ, broadcast, KHÔNG bền**. Cần **đảm bảo nhận** (bền, ACK,
> replay) thì dùng **Stream** chứ không phải Pub/Sub.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Pub/Sub** = messaging **1 gửi → nhiều nhận** theo **channel**, publisher/subscriber **decoupled**. Lệnh:
  `SUBSCRIBE`/`PSUBSCRIBE` (nghe), **`PUBLISH channel msg`** (gửi).
- **Cơ chế:** khi `PUBLISH`, Redis **đẩy ngay** tới các subscriber **đang online** của channel; **không lưu message**.
- **Đặc tính quan trọng:** **fire-and-forget / at-most-once** — subscriber **offline là mất message**, không ACK,
  không retry, không replay. Là **broadcast** (khác queue: mỗi job 1 worker).
- **Dùng khi:** real-time **chấp nhận mất** — chat, notification, live update, **broadcast WebSocket giữa nhiều
  server**, invalidate cache.
- **Cần bền/ACK/replay → dùng Redis Stream**, không phải Pub/Sub. Trong **Cluster** cân nhắc **Sharded Pub/Sub**
  (Redis 7) để đỡ tốn băng thông.

[↑ Mục lục](#mục-lục)

<a id="q18-11"></a>
### 18.11. Redis Stream hoạt động thế nào?

#### 📖 Giới thiệu kiến thức

**Redis Stream** (từ Redis 5.0) là kiểu dữ liệu **log append-only** — một chuỗi message **có thứ tự, được lưu bền,
đọc lại được** — như một **Kafka thu nhỏ** trong Redis. Sinh ra để bù đúng nhược điểm của **Pub/Sub** (mất message
nếu offline, không ACK): Stream **giữ lại message** và hỗ trợ **consumer group + ACK + retry**.

**1. Cấu trúc: mỗi entry là (ID, các cặp field-value).**
Mỗi message thêm vào Stream có một **ID tự tăng** dạng **`<millisecondTime>-<sequence>`** (vd `1720000000000-0`) →
đảm bảo **thứ tự** và **duy nhất**. Nội dung là các cặp key-value (giống Hash).

```bash
# Producer: thêm entry vào stream "orders" (* = Redis tự sinh ID theo thời gian)
XADD orders * order_id 5 amount 200
# → trả về ID, vd "1720000000000-0"

XLEN orders                 # số entry trong stream
XRANGE orders - +           # đọc mọi entry từ đầu (-) tới cuối (+)
```

**2. Hai cách đọc:**

**a) Đọc trực tiếp (không group) — `XREAD`.** Đọc như tail một file log; nhiều consumer đọc **đều nhận full**
(fan-out).
```bash
XREAD BLOCK 5000 STREAMS orders $   # $ = chỉ nhận entry MỚI, block tối đa 5s chờ
```

**b) Consumer Group — `XREADGROUP` (đây mới là điểm mạnh nhất).**
Một **group** gồm nhiều consumer **chia nhau** xử lý (mỗi entry chỉ **một** consumer trong group nhận) — giống
queue nhưng **bền + có ACK**.

```bash
XGROUP CREATE orders g1 0                       # tạo group "g1", đọc từ đầu (0)
# Consumer "c1" trong group lấy tối đa 1 entry chưa ai xử lý (>)
XREADGROUP GROUP g1 c1 COUNT 1 BLOCK 5000 STREAMS orders >
# Xử lý xong → báo đã hoàn tất (ACK) để Redis bỏ khỏi danh sách pending
XACK orders g1 1720000000000-0
```

**3. Cơ chế đảm bảo không mất job — PEL (Pending Entries List).**
- Khi consumer `XREADGROUP` lấy một entry, Redis đưa entry đó vào **PEL** (danh sách "đã giao, **chưa ACK**") của
  consumer đó.
- Nếu consumer **xử lý xong** → `XACK` → entry rời PEL.
- Nếu consumer **chết giữa chừng** (chưa ACK): entry **vẫn nằm trong PEL** → không mất. Consumer khác dùng
  **`XPENDING`** để thấy job treo, rồi **`XCLAIM`** (hoặc **`XAUTOCLAIM`**) để **giành lấy** job quá hạn mà xử lý
  lại. Đây là cơ chế **at-least-once delivery + retry** mà Pub/Sub không có.

**4. Giới hạn kích thước (vì Stream lưu bền → phải cắt bớt).**
Stream giữ message mãi → phình RAM. Dùng **`MAXLEN`**/**`MINID`** để cắt cũ:
```bash
XADD orders MAXLEN ~ 10000 * order_id 6 amount 50   # giữ ~10000 entry gần nhất (~ = xấp xỉ, nhanh hơn)
XTRIM orders MINID 1720000000000                     # xóa entry cũ hơn một mốc thời gian
```

**5. Stream vs Pub/Sub vs List (chọn khi nào):**

| Tiêu chí | **Stream** | Pub/Sub | List (queue) |
|----------|-----------|---------|--------------|
| Lưu message | **Có** (bền, đọc lại/replay) | Không (mất nếu offline) | Có (tới khi pop) |
| Nhiều consumer chia việc | **Có** (consumer group) | Broadcast (ai cũng nhận) | Có (nhiều worker `BRPOP`) |
| ACK / retry | **Có** (PEL, `XACK`, `XCLAIM`) | Không | Tự làm |
| Đọc lại lịch sử | **Có** (`XRANGE`) | Không | Không |
| Delivery | **at-least-once** | at-most-once | at-least-once (tùy) |
| Giống | Kafka mini | Fan-out realtime | Hàng đợi đơn giản |

**6. Ứng dụng:** event sourcing, message queue **bền** cần retry/ack, pipeline xử lý sự kiện (order → payment →
ship), thu thập log/metric, nhiều service tiêu thụ cùng nguồn sự kiện với tiến độ (offset) riêng.

> **Chốt bản chất:** Stream = **log bền + consumer group + ACK/retry**. Cần đảm bảo **không mất & xử lý lại được**
> → Stream. Chỉ cần realtime nhẹ, chấp nhận mất → Pub/Sub (mục 18.10). Hàng đợi job đơn giản → List (mục 18.8).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Redis Stream** (5.0+) = kiểu dữ liệu **log append-only** bền, mỗi entry có **ID `<time>-<seq>`** tự tăng —
  như **Kafka thu nhỏ**. Sinh ra để bù nhược điểm Pub/Sub (mất message, không ACK).
- **Ghi:** `XADD stream * field val`; **đọc:** `XREAD` (fan-out) hoặc **`XREADGROUP`** (consumer group — nhiều
  consumer **chia việc**, mỗi entry 1 consumer).
- **Không mất job nhờ PEL:** entry đã giao mà **chưa `XACK`** nằm trong **Pending Entries List**; consumer chết →
  người khác **`XCLAIM`/`XAUTOCLAIM`** xử lý lại → **at-least-once + retry**.
- **Cắt bớt** bằng `MAXLEN`/`XTRIM` để khỏi phình RAM (Stream lưu bền).
- **Chọn:** cần **bền + ACK + replay + chia việc** → Stream; realtime chấp nhận mất → **Pub/Sub**; queue job đơn
  giản → **List**.

[↑ Mục lục](#mục-lục)

<a id="q18-12"></a>
### 18.12. Redis distributed lock (khóa phân tán) hoạt động thế nào?

#### 📖 Giới thiệu kiến thức

Khi có **nhiều tiến trình/server** cùng muốn thao tác một tài nguyên (trừ tồn kho, xử lý một đơn, chạy một cron
job), ta cần đảm bảo **chỉ một** cái được vào tại một thời điểm → tránh **race condition**. Lock trong 1 process
(mutex) không đủ vì các server **khác nhau** không chia sẻ bộ nhớ. **Distributed lock** dùng một hệ chung (Redis)
làm "trọng tài" cấp khóa.

**1. Ý tưởng cơ bản: `SET key value NX PX ttl`.**
Redis dùng đúng một lệnh **atomic** để "giành khóa":
```bash
SET lock:order:5 <random_token> NX PX 30000
# NX  = chỉ set nếu key CHƯA tồn tại  → chỉ MỘT client thành công
# PX  = TTL 30000ms (30s)             → tự nhả khóa nếu client chết (tránh deadlock)
```
- Client nào `SET` trả `OK` → **giữ được khóa**; các client khác nhận `nil` → **thử lại hoặc bỏ**.
- Xong việc thì **xóa key** để nhả khóa.

**2. Vì sao cần TTL (`PX`)?** Nếu client giữ khóa **chết** trước khi xóa → khóa **kẹt vĩnh viễn** (deadlock). TTL
đảm bảo khóa **tự hết hạn**. Nhưng TTL sinh ra vấn đề ở (3).

**3. Vì sao cần `value` là token ngẫu nhiên (điểm quan trọng nhất)?**
Kịch bản lỗi kinh điển:
- Client A giành khóa, TTL 30s. A xử lý **chậm quá 30s** → khóa **tự hết hạn**.
- Client B giành được khóa (vì đã trống). Giờ **B đang giữ khóa**.
- A xử lý xong, gọi `DEL lock` → **xóa nhầm khóa của B**! → hai client cùng vào vùng tới hạn.

→ Khắc phục: mỗi client đặt **value = token ngẫu nhiên duy nhất**, và khi nhả khóa phải **kiểm tra token trước
khi xóa** — chỉ xóa nếu đúng khóa của mình. Việc "check rồi del" phải **atomic** bằng **Lua script**:

```lua
-- Nhả khóa an toàn: chỉ DEL nếu value đúng token của mình
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
```

**4. Vấn đề gia hạn (watchdog).** Nếu job có thể chạy lâu hơn TTL, cần một luồng nền **gia hạn TTL** (`PEXPIRE`)
định kỳ khi vẫn đang giữ khóa — gọi là **watchdog**. Thư viện **Redisson** (Java) làm sẵn; **Laravel** `Cache::lock`
cũng hỗ trợ block & auto-release.

```php
// Laravel: lock atomic, tự nhả sau 30s, chờ tối đa 10s để giành
Cache::lock('order:5', 30)->block(10, function () {
    // vùng tới hạn — chỉ 1 tiến trình vào
});
```

**5. Redlock — lock trên nhiều Redis master (khi cần chắc hơn).**
Lock trên **một** Redis có điểm yếu: nếu master chết ngay sau khi cấp khóa mà **chưa kịp nhân bản** sang replica,
failover → replica lên master **không biết** khóa đó → client khác giành lại → **hai client cùng giữ khóa**. Thuật
toán **Redlock** (do tác giả Redis đề xuất) giảm rủi ro: client giành khóa trên **N master độc lập** (vd 5), chỉ
coi là thành công nếu chiếm được **đa số (N/2+1)** trong thời gian ngắn.
> ⚠️ Redlock còn **gây tranh cãi** về tính đúng đắn (Martin Kleppmann phản biện). Với yêu cầu **tuyệt đối không
> được trùng**, nên dùng cơ chế mạnh hơn (fencing token, hoặc lock của Zookeeper/etcd). Đa số ứng dụng thực tế
> chỉ cần single-node lock + token + TTL là đủ.

**6. Fencing token (chống lỗi ở tầng tài nguyên).** Mỗi lần cấp khóa kèm một **số tăng dần** (token). Tài nguyên
(DB, file) chỉ chấp nhận thao tác có token **lớn hơn** token đã thấy → dù có 2 client tưởng mình giữ khóa, thao
tác của client "cũ" (token nhỏ) bị **từ chối**. Đây là cách bảo vệ đúng đắn nhất.

```php
// ===== Phía CLIENT: giành khóa + lấy fencing token tăng dần =====
$token = uniqid('', true);                                   // token ngẫu nhiên để nhả khóa an toàn
if (Redis::set('lock:order:5', $token, 'NX', 'PX', 30000)) { // giành được khóa
    // INCR là ATOMIC → mỗi lần cấp khóa sinh 1 số DUY NHẤT, luôn TĂNG DẦN
    $fence = Redis::incr('fence:order:5');                    // vd client A nhận 33

    // gửi kèm $fence xuống tài nguyên khi ghi
    writeToResource(orderId: 5, fencingToken: $fence, data: [...]);

    // ... xử lý xong thì nhả khóa bằng Lua (check token rồi mới DEL) ...
}

// ===== Phía TÀI NGUYÊN (DB/service): chỉ nhận token LỚN HƠN token đã thấy =====
function writeToResource(int $orderId, int $fencingToken, array $data): void
{
    // last_fence lưu bền cùng tài nguyên (cột trong DB, hoặc key riêng)
    $lastFence = DB::table('orders')->where('id', $orderId)->value('last_fence') ?? 0;

    if ($fencingToken <= $lastFence) {
        // token cũ → client này đã bị "qua mặt" (khóa hết hạn, client khác đã ghi)
        throw new RuntimeException("Fencing token $fencingToken bị từ chối (đã thấy $lastFence)");
    }

    // hợp lệ: cập nhật last_fence + ghi dữ liệu trong CÙNG 1 transaction (atomic)
    DB::transaction(function () use ($orderId, $fencingToken, $data) {
        DB::table('orders')->where('id', $orderId)
            ->update(['last_fence' => $fencingToken] + $data);
    });
}
```

> **Kịch bản token cứu bàn thua:** A giành khóa, nhận `fence=33`, rồi **kẹt** (GC pause/mạng) khiến khóa hết hạn.
> B giành khóa, nhận `fence=34`, ghi thành công → tài nguyên lưu `last_fence=34`. Khi A "tỉnh lại" ghi với
> `fence=33`, tài nguyên thấy `33 ≤ 34` → **từ chối**. Nhờ vậy dù cả hai *tưởng* mình giữ khóa, chỉ thao tác mới
> nhất được chấp nhận — an toàn ngay cả khi lock ở Redis bị lỗi.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Distributed lock** = đảm bảo **chỉ 1 tiến trình/server** vào vùng tới hạn giữa nhiều máy (chống race). Cốt lõi:
  **`SET key <token> NX PX ttl`** — `NX` để chỉ một client giành được, `PX` (TTL) để **tự nhả nếu client chết**
  (tránh deadlock).
- **Value phải là token ngẫu nhiên**, và nhả khóa phải **check token rồi mới DEL** bằng **Lua atomic** — nếu không
  sẽ **xóa nhầm khóa của client khác** khi job chạy quá TTL.
- Job lâu → cần **watchdog** gia hạn TTL (Redisson, Laravel `Cache::lock`).
- **Redlock** = giành khóa trên **đa số nhiều master** để chắc hơn khi single-node có thể mất khóa lúc failover —
  nhưng **gây tranh cãi**; tuyệt đối đúng thì dùng **fencing token** (số tăng dần, tài nguyên từ chối token cũ).
- Thực tế đa số: **single-node + random token + TTL + Lua release** là đủ.

[↑ Mục lục](#mục-lục)

<a id="q18-13"></a>
### 18.13. Redis — làm sao để tránh leak/rò rỉ dữ liệu (bảo mật)?

#### 📖 Giới thiệu kiến thức

Đây là câu **thực chiến** rất hay bị hỏi, vì Redis là một trong những dịch vụ **bị tấn công nhiều nhất** khi
cấu hình sai. **Mặc định Redis không mã hoá và (bản cũ) không yêu cầu mật khẩu** → nếu để cổng **6379** lộ ra
Internet, kẻ tấn công có thể **đọc toàn bộ dữ liệu**, thậm chí dùng `CONFIG SET dir` + `SAVE` để **ghi SSH key /
cron job** lên máy chủ → chiếm quyền. Vì vậy phòng chống "leak data" gồm nhiều lớp:

**1. Không phơi Redis ra Internet — network isolation (quan trọng nhất).**
- Đặt Redis trong **private subnet / VPC**, chỉ cho app trong mạng nội bộ truy cập; **không mở port 6379** ra ngoài.
- Dùng **firewall / security group** giới hạn IP nguồn.
- Cấu hình `bind` chỉ nghe interface cần thiết:

```conf
# redis.conf
bind 127.0.0.1 10.0.0.5      # chỉ nghe localhost + IP nội bộ, KHÔNG bind 0.0.0.0
protected-mode yes           # bật (mặc định): chặn kết nối ngoài khi chưa có password
```

> **Protected mode** (mặc định bật từ Redis 3.2): nếu Redis **không có password** mà lại bind ra interface ngoài
> loopback, nó **từ chối** kết nối từ xa — hàng rào cuối chống lộ ra ngoài do quên cấu hình.

**2. Bật xác thực (authentication).**
- **`requirepass`** — đặt mật khẩu mạnh, client phải `AUTH` trước khi dùng.
- **ACL (Redis 6+)** — phân quyền chi tiết theo **user**: giới hạn **lệnh** được chạy và **pattern key** truy cập
  (least privilege). Ví dụ user chỉ được đọc key `cache:*`.

```conf
requirepass "MatKhauRatManh_123!@#"
```
```bash
# ACL: tạo user chỉ được GET/SET trên key cache:*
ACL SETUSER app on >matkhau ~cache:* +get +set
```

**3. Mã hoá đường truyền (in transit) — TLS.**
- Redis 6+ hỗ trợ **TLS** native → chống nghe lén khi client–server qua mạng không tin cậy.

**4. Vô hiệu hoá / đổi tên lệnh nguy hiểm.**
- Chặn các lệnh có thể phá/leak dữ liệu hàng loạt: `FLUSHALL`, `FLUSHDB`, `CONFIG`, `KEYS`, `DEBUG`, `EVAL`.

```conf
rename-command FLUSHALL ""       # "" = vô hiệu hoá hẳn
rename-command CONFIG "CONFIG_9a3f"   # đổi tên khó đoán
```

> Vì sao chặn `KEYS`? `KEYS *` quét toàn bộ keyspace → chậm (blocking) và lộ cấu trúc dữ liệu; production dùng
> `SCAN` thay thế.

**5. Mã hoá lúc nghỉ (at rest) & bảo vệ file dump.**
- Redis **không mã hoá** file `dump.rdb` / `appendonly.aof` → ai đọc được file là đọc được dữ liệu. Phải **bảo vệ
  quyền file & mã hoá đĩa** (LUKS, hoặc dùng managed service như **AWS ElastiCache/Redis Enterprise** có encryption
  at rest).
- **Không lưu dữ liệu nhạy cảm dạng plaintext** (mật khẩu, thẻ) — nếu buộc phải lưu, **mã hoá ở tầng ứng dụng**.

**6. Vận hành an toàn.**
- **Không chạy Redis bằng user `root`** (chạy user riêng quyền thấp) → giảm thiệt hại nếu bị chiếm.
- Cập nhật phiên bản vá lỗi; giám sát log truy cập bất thường.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Redis mặc định **không mã hoá, (bản cũ) không password** → lộ port 6379 ra Internet là **rò rỉ toàn bộ dữ liệu**,
  thậm chí bị chiếm máy (`CONFIG SET dir`+`SAVE` ghi SSH key). Phòng chống **nhiều lớp**:
- **Network:** đặt trong **private VPC**, **không mở 6379 ra ngoài**, `bind` IP nội bộ, `protected-mode yes`, firewall.
- **Auth:** `requirepass` mật khẩu mạnh; **ACL (Redis 6+)** phân quyền theo user (giới hạn lệnh & pattern key).
- **TLS** mã hoá đường truyền; **rename/disable** lệnh nguy hiểm (`FLUSHALL`, `CONFIG`, `KEYS`, `EVAL`).
- **At rest:** file RDB/AOF **không mã hoá** → bảo vệ file + mã hoá đĩa (hoặc managed service); **không lưu dữ liệu
  nhạy cảm plaintext**, mã hoá ở tầng app.
- Chạy Redis bằng **user quyền thấp**, không phải root; cập nhật & giám sát.

[↑ Mục lục](#mục-lục)

<a id="q18-14"></a>
### 18.14. Redis — cơ chế lưu bền & khôi phục (restore) dữ liệu

#### 📖 Giới thiệu kiến thức

Redis để dữ liệu trong **RAM** → nếu chỉ vậy, **restart/crash là mất sạch**. Để không mất và **restore** được,
Redis cung cấp **persistence** ghi dữ liệu ra đĩa theo **2 cơ chế** (dùng riêng hoặc kết hợp):

**1. RDB (Redis Database) — snapshot điểm-thời-gian.**
- Chụp **ảnh toàn bộ dataset** tại một thời điểm, lưu vào file nhị phân **`dump.rdb`**.
- Cách chạy: `SAVE` (blocking) hoặc **`BGSAVE`** (Redis **fork** một process con ghi nền, không chặn); tự động
  theo cấu hình `save`:

```conf
save 900 1       # nếu ≥1 key đổi trong 900s → snapshot
save 300 10      # ≥10 key đổi trong 300s → snapshot
```

- **Ưu:** file **nhỏ gọn**, **restore nhanh** (nạp một phát), ít ảnh hưởng hiệu năng runtime → hợp làm **backup**.
- **Nhược:** **mất dữ liệu** giữa hai lần snapshot — crash ngay trước snapshot kế tiếp thì mất phần thay đổi từ
  snapshot gần nhất (có thể vài phút).

**2. AOF (Append Only File) — ghi log mọi lệnh ghi.**
- Ghi **tuần tự mọi lệnh write** (SET, LPUSH…) vào file **`appendonly.aof`**; khi khởi động, Redis **replay lại**
  các lệnh để dựng lại dữ liệu.
- Độ bền phụ thuộc `appendfsync`:

```conf
appendonly yes
appendfsync everysec   # fsync mỗi giây (khuyến nghị): tối đa mất ~1s dữ liệu
# appendfsync always   # fsync mỗi lệnh: an toàn nhất, chậm nhất
# appendfsync no       # để OS tự quyết: nhanh nhất, kém an toàn
```

- **AOF rewrite:** file cứ phình to → Redis định kỳ **nén lại** (`BGREWRITEAOF`) thành tập lệnh tối thiểu tái tạo
  cùng trạng thái.
- **Ưu:** **bền hơn** (mất tối đa ~1s với `everysec`). **Nhược:** file **to hơn**, **restore chậm hơn** RDB.

**3. Hybrid RDB + AOF (khuyến nghị, Redis 4.0+).**
- Bật cả hai: file AOF bắt đầu bằng **RDB preamble** (ảnh nền dạng RDB) rồi nối các lệnh sau đó → **nạp nhanh như
  RDB** + **bền như AOF**. Đây là cấu hình production khuyên dùng.

**Cơ chế RESTORE (khôi phục) hoạt động thế nào:**
- **Khi khởi động, Redis tự nạp dữ liệu từ đĩa:** nếu **AOF bật** → ưu tiên nạp **`appendonly.aof`**; nếu không →
  nạp **`dump.rdb`** trong thư mục `dir`.
- **Restore thủ công từ backup:** dừng Redis → **chép file `dump.rdb`** (hoặc AOF) vào đúng `dir` → khởi động lại
  → Redis tự nạp. RDB **an toàn để copy** (là snapshot nhất quán), nên quy trình backup thường là **`BGSAVE` rồi
  copy `dump.rdb`** ra nơi lưu trữ (S3…).

```bash
# Backup: ép snapshot rồi copy file ra ngoài
redis-cli BGSAVE
cp /var/lib/redis/dump.rdb /backup/dump-$(date +%F).rdb

# Restore: đặt file về đúng chỗ rồi start lại
cp /backup/dump-2026-07-14.rdb /var/lib/redis/dump.rdb
systemctl start redis     # Redis tự nạp dump.rdb khi khởi động
```

**4. Replication & HA (bổ trợ, không thay backup).**
- **Master–Replica:** replica nhận **full RDB** ban đầu rồi **stream lệnh** tiếp theo → có bản sao nóng, đọc scale
  và phục hồi nhanh nếu master chết.
- **Redis Sentinel** (tự động failover) hoặc **Redis Cluster** (sharding + HA) cho hệ thống lớn.
- ⚠️ Replication **không thay thế backup**: lệnh sai (vd `FLUSHALL`) sẽ **nhân bản sang replica** ngay → vẫn cần
  snapshot/AOF để khôi phục về mốc trước.

**Chọn cấu hình theo nhu cầu:**
- **Chỉ cache** (mất được): tắt persistence cho nhanh.
- **Cần bền vừa phải:** RDB định kỳ.
- **Không được mất dữ liệu (gần) real-time:** **AOF `everysec`** (hoặc hybrid) + replication + backup định kỳ.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Redis in-memory → cần **persistence** để không mất khi restart, có **2 cơ chế**:
- **RDB:** **snapshot** toàn bộ ra `dump.rdb` (`BGSAVE`, cấu hình `save`). Nhỏ gọn, **restore nhanh**, hợp **backup**;
  nhưng **mất dữ liệu** giữa 2 snapshot.
- **AOF:** ghi **log mọi lệnh write** vào `appendonly.aof`, replay khi khởi động; `appendfsync everysec` → **mất tối
  đa ~1s**. Bền hơn nhưng file to & restore chậm; có **AOF rewrite** để nén.
- **Hybrid RDB+AOF (Redis 4.0+):** nạp nhanh như RDB + bền như AOF → **khuyến nghị** cho production.
- **Restore:** khởi động Redis **tự nạp** AOF (nếu bật) hoặc RDB từ `dir`; backup thủ công = **`BGSAVE` → copy
  `dump.rdb`**, restore = đặt file về `dir` rồi start lại.
- **Replication (master-replica) + Sentinel/Cluster** cho HA & phục hồi nhanh — **không thay backup** (lỗi ghi
  nhân bản ngay sang replica).

[↑ Mục lục](#mục-lục)

<a id="q18-15"></a>
### 18.15. Giải pháp data recovery hoàn chỉnh cho Redis (production playbook)

#### 📖 Giới thiệu kiến thức

Mục 3.8 nói **cơ chế** (RDB/AOF/replication). Câu này ghép chúng thành một **chiến lược khôi phục end-to-end**:
không có một tính năng đơn lẻ nào lo được mọi kiểu sự cố — phải **phối nhiều lớp** theo mục tiêu **RPO/RTO**.

**Trước tiên — định nghĩa mục tiêu khôi phục:**
- **RPO (Recovery Point Objective):** chấp nhận **mất tối đa bao nhiêu dữ liệu** (tính bằng thời gian). AOF
  `everysec` → RPO ~1s; RDB `save 900` → RPO tới ~15 phút.
- **RTO (Recovery Time Objective):** chấp nhận **downtime tối đa bao lâu** để khôi phục. Sentinel failover → RTO
  vài giây; restore từ backup S3 → RTO vài phút.

**Bước 1 — Nhận diện các kịch bản sự cố (mỗi kịch bản cần lớp phòng thủ khác nhau):**

| Sự cố | Lớp cứu | Vì sao |
|-------|---------|--------|
| **Process crash / restart máy** | **Persistence (AOF/RDB)** tự nạp lại | Chỉ mất RAM, file trên đĩa còn |
| **Xóa nhầm dữ liệu** (`FLUSHALL`, xóa key sai) | **Backup point-in-time** (RDB cũ) | Replication **vô dụng** — lệnh xóa nhân bản ngay sang replica |
| **File RDB/AOF hỏng** (corrupt) | `redis-check-*` **repair** hoặc restore backup | AOF/RDB lỗi khiến Redis không start |
| **Node/ổ đĩa chết** | **Replica + Sentinel/Cluster failover** | Có bản sao nóng, promote replica |
| **Mất cả data center / AZ** | **Offsite / cross-region backup** | Backup nằm ngoài vùng bị sập |

> **Điểm mấu chốt phỏng vấn:** **replication ≠ backup**. Replica chống *hỏng phần cứng* nhưng **không** cứu được
> *lỗi logic/con người* (xóa nhầm, bug ghi sai) vì mọi lệnh được nhân bản tức thì. Backup point-in-time mới cứu được.

**Bước 2 — Giải pháp hoàn chỉnh (xếp lớp):**

1. **Persistence bền:** bật **hybrid RDB+AOF**, `appendfsync everysec` (`aof-use-rdb-preamble yes`) → RPO ~1s, nạp nhanh.
2. **Backup định kỳ + offsite:** cron **`BGSAVE`** rồi copy `dump.rdb` (có timestamp) lên **S3/cross-region**; đặt
   **retention/rotation** (giữ N ngày, bản tuần/tháng). Đây là lớp cứu "xóa nhầm" & "mất AZ".
3. **HA & auto-failover:** **master–replica** + **Sentinel** (hoặc **Redis Cluster**) → node chết thì promote replica,
   RTO vài giây. Client dùng driver **Sentinel-aware** để tự đổi endpoint.
4. **Kiểm chứng backup (bắt buộc):** định kỳ **restore thử** vào instance staging + `redis-check-rdb`/`redis-check-aof`.
   *Một backup chưa từng restore thử = coi như không có backup.*
5. **Runbook rõ ràng:** kịch bản → lệnh cụ thể (bên dưới), ai làm, RTO mục tiêu.

**Bước 3 — Quy trình khôi phục cụ thể (runbook):**

**a) Restore từ RDB backup (vd sau khi lỡ `FLUSHALL`):**
```bash
sudo systemctl stop redis
# đưa file backup về đúng thư mục dir của Redis
cp /backup/dump-2026-07-14.rdb /var/lib/redis/dump.rdb
sudo systemctl start redis        # Redis tự nạp dump.rdb khi khởi động
```
> ⚠️ **Bẫy quan trọng:** nếu **AOF đang bật**, khi khởi động Redis **ưu tiên nạp AOF**, **bỏ qua RDB** → restore
> RDB sẽ *không có tác dụng*. Cách xử lý đúng: tắt tạm `appendonly no` → start (nạp RDB) → rồi bật lại bằng
> `redis-cli CONFIG SET appendonly yes` (Redis tự **rewrite AOF** từ dataset hiện tại) → dữ liệu khôi phục thành AOF mới.

**b) Sửa file hỏng (corrupt):**
```bash
redis-check-aof --fix appendonly.aof     # cắt phần AOF lỗi ở cuối
redis-check-rdb dump.rdb                  # kiểm tra tính toàn vẹn RDB
```

**c) Node master chết:** Sentinel tự **promote** một replica thành master; nếu thủ công: `REPLICAOF NO ONE` trên
replica được chọn, rồi trỏ app sang.

**Bước 4 — Cạm bẫy vận hành cần tránh:**
- **Master không persistence + auto-restart:** master crash → khởi động lại với dataset **rỗng** → replica đồng bộ
  theo và **mất sạch dữ liệu**. → Master phải có persistence (hoặc tắt auto-restart).
- **Đầy đĩa khi `BGSAVE`/AOF rewrite:** fork cần chỗ ghi file → giám sát dung lượng đĩa.
- **Managed service** (AWS **ElastiCache**, GCP **Memorystore**): đã có **snapshot tự động + restore một click +
  Multi-AZ failover** → giảm gánh nặng tự vận hành; production nên ưu tiên nếu được.

#### 🎯 Tóm tắt trả lời phỏng vấn

- Không có tính năng đơn lẻ lo mọi sự cố → **xếp lớp theo RPO (mất bao nhiêu dữ liệu) / RTO (downtime bao lâu)**.
- **Map sự cố ↔ lớp cứu:** crash/restart → **persistence (AOF/RDB)**; **xóa nhầm** → **backup point-in-time**
  (replication vô dụng); file hỏng → **`redis-check-*`**; node chết → **replica + Sentinel failover**; mất AZ →
  **offsite/cross-region backup**.
- **Giải pháp hoàn chỉnh:** (1) hybrid **RDB+AOF everysec**; (2) **BGSAVE → copy offsite (S3)** có retention;
  (3) **replica + Sentinel/Cluster** auto-failover; (4) **restore thử định kỳ** + check integrity; (5) **runbook**.
- **Câu chốt:** **replication ≠ backup** (lỗi logic/xóa nhầm nhân bản ngay). Bẫy hay gặp: AOF bật thì restore RDB
  phải tắt tạm `appendonly`; master không persistence + auto-restart → replica mất sạch dữ liệu.
- Production nên cân nhắc **managed service** (ElastiCache/Memorystore) để có snapshot + failover sẵn.

[↑ Mục lục](#mục-lục)

<a id="q18-16"></a>
### 18.16. Redis Cluster hoạt động thế nào?

#### 📖 Giới thiệu kiến thức

Một Redis đơn (kể cả có replica) bị giới hạn: **toàn bộ dataset phải vừa RAM của 1 máy** và mọi ghi dồn vào 1
master. **Redis Cluster** giải quyết bằng **sharding tự động** — chia dữ liệu ra **nhiều master**, mỗi master
giữ một phần, cộng lại thành CSDL lớn hơn RAM một máy, đồng thời phân tải ghi. Đây là chế độ **scale ngang + HA**
tích hợp sẵn của Redis (khác **Sentinel** chỉ lo HA cho một dataset không sharding).

**1. Hash slot — cách chia dữ liệu (cốt lõi).**
Cluster chia keyspace thành cố định **16384 hash slot**. Mỗi key được ánh xạ về một slot bằng:

```
slot = CRC16(key) mod 16384
```

Mỗi **master** phụ trách một **dải slot**. Ví dụ cụm 3 master:
- Master A: slot 0–5460
- Master B: slot 5461–10922
- Master C: slot 10923–16383

→ Ghi/đọc key `user:1` sẽ được định tuyến tới master đang giữ slot của nó. **Không dùng consistent hashing** như
nhiều hệ khác, mà dùng **slot cố định** → resharding = **di chuyển slot** giữa các node, đơn giản và tường minh.

**2. Kiến trúc: mỗi shard có master + replica.**
Mỗi master thường kèm **1+ replica** (bản sao). Master chết → replica của **chính shard đó** được promote → không
mất phần dữ liệu của shard. Cụm production tối thiểu khuyến nghị **3 master + 3 replica**.

**3. Client định tuyến & redirection (MOVED / ASK).**
- Client Cluster-aware **cache bản đồ slot → node**, gửi lệnh thẳng tới đúng node.
- Nếu gửi nhầm node, node trả lỗi **`MOVED <slot> <ip:port>`** → client cập nhật map và gửi lại đúng chỗ.
- Trong lúc **đang di chuyển slot** (resharding), node trả **`ASK`** → client hỏi tạm node đích cho key đó (chuyển
  hướng một lần, chưa cập nhật map).

**4. Gossip protocol — các node tự biết trạng thái nhau.**
Các node nói chuyện với nhau qua **cluster bus** (cổng riêng, thường = cổng client + 10000) bằng **gossip**: trao
đổi thông tin ai sống/chết, ai giữ slot nào. Nhờ vậy cụm **không cần node điều phối trung tâm** (decentralized).

**5. Phát hiện lỗi & failover (bầu chọn).**
- Node nghi ngờ một master chết → đánh dấu **PFAIL** (possible fail); khi **đa số master** đồng ý → **FAIL**.
- **Replica** của master chết xin được promote; **các master bỏ phiếu** — cần **đa số master (quorum)** đồng ý →
  replica lên làm master mới, tiếp quản dải slot. Vì cần *đa số master* nên cụm phải có **số master lẻ ≥ 3** để
  tránh split-brain.

**6. Multi-key & hash tag.**
Lệnh đụng **nhiều key** (`MGET`, transaction, Lua) chỉ chạy được khi **các key cùng một slot**. Muốn ép nhiều key
về cùng slot, dùng **hash tag** — chỉ phần trong `{}` được hash:

```bash
# {user:1} là hash tag → cả hai key rơi vào CÙNG một slot → dùng chung được
SET {user:1}:profile "..."
SET {user:1}:cart    "..."
MGET {user:1}:profile {user:1}:cart      # OK vì cùng slot
```

**7. Tính nhất quán (cần biết để không mất điểm).**
Redis Cluster **không đảm bảo strong consistency**: replication là **bất đồng bộ** → khi failover có thể **mất
các ghi chưa kịp nhân bản**. Ngoài ra `cluster-require-full-coverage`: mặc định **yes** → nếu một dải slot **không
có node phục vụ** (mất cả master lẫn replica của shard đó), **cả cụm ngừng nhận lệnh** cho tới khi khôi phục (đánh
đổi availability lấy toàn vẹn).

**Cluster vs Sentinel — chọn cái nào?**

| | **Sentinel** | **Cluster** |
|---|---|---|
| Mục tiêu | Chỉ **HA** (auto-failover) | **HA + sharding** (scale ngang) |
| Dữ liệu | 1 dataset trên 1 master (vừa RAM 1 máy) | Chia **16384 slot** ra nhiều master |
| Khi dùng | Dataset vừa RAM, chỉ cần chống chết node | Dataset **lớn hơn RAM 1 máy** / cần phân tải ghi |
| Multi-key | Thoải mái | Chỉ khi **cùng slot** (hash tag) |

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Redis Cluster = sharding tự động + HA:** chia keyspace thành **16384 hash slot**, mỗi key về slot bằng
  **`CRC16(key) mod 16384`**; mỗi **master** giữ một dải slot → dataset lớn hơn RAM 1 máy & phân tải ghi.
- Mỗi shard có **master + replica**; master chết → **replica shard đó** được promote (giữ nguyên dải slot).
- **Client Cluster-aware** cache map slot→node, gửi thẳng; sai chỗ nhận **`MOVED`**, đang reshard nhận **`ASK`**.
- Các node dùng **gossip** (cluster bus) để biết trạng thái nhau — **phi tập trung**, không cần node điều phối.
- **Failover cần đa số master bỏ phiếu** → phải **≥3 master lẻ** để tránh split-brain.
- **Multi-key** chỉ chạy khi các key **cùng slot** → dùng **hash tag `{...}`**. Nhất quán **không mạnh** (replication
  async, có thể mất ghi khi failover).
- **Sentinel** = chỉ HA cho 1 dataset; **Cluster** = HA **+ scale ngang** khi dữ liệu vượt RAM một máy.

[↑ Mục lục](#mục-lục)

<!--
NHẮC FORMAT (xem /CLAUDE.md §4):

<a id="q1"></a>
## 1. <Câu hỏi>

### 📖 Giới thiệu kiến thức
...

### 🎯 Tóm tắt trả lời phỏng vấn
...

[↑ Mục lục](#mục-lục)
-->
