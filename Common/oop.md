<a id="mục-lục"></a>
# Common — OOP (Lập trình hướng đối tượng)

> 4 tính chất OOP (Encapsulation · Inheritance · Polymorphism · Abstraction) · class/object ·
> interface · abstract class · composition vs inheritance · coupling/cohesion.

## 📑 Mục lục

1. [Lập trình hướng đối tượng (OOP) là gì?](#q1)
   - 1.1. [Ưu điểm của lập trình hướng đối tượng](#q1-1)

---

<a id="q1"></a>
## 1. Lập trình hướng đối tượng (OOP) là gì?

### 📖 Giới thiệu kiến thức

**Lập trình hướng đối tượng (Object-Oriented Programming — OOP)** là một **mô hình lập trình
(paradigm)** tổ chức code xoay quanh các **đối tượng (object)**, thay vì xoay quanh hàm và luồng
logic tuần tự như lập trình thủ tục (procedural).

Mỗi **object** đóng gói 2 thứ vào cùng một chỗ:
- **Thuộc tính (property/attribute)** — *dữ liệu*, mô tả trạng thái của đối tượng (vd: `name`, `balance`).
- **Phương thức (method)** — *hành vi*, những việc đối tượng có thể làm (vd: `deposit()`, `login()`).

#### Class và Object

- **Class** là **bản thiết kế (blueprint)** — định nghĩa một loại đối tượng có những thuộc tính và
  hành vi gì.
- **Object** là **thực thể cụ thể (instance)** được tạo ra từ class.

> Ví von: `Class` như **bản vẽ ngôi nhà**, còn `Object` là **những căn nhà thật** xây từ bản vẽ đó —
> cùng thiết kế nhưng mỗi căn có dữ liệu riêng (màu sơn, chủ nhà...).

```php
class BankAccount
{
    private float $balance = 0;          // thuộc tính (dữ liệu, được đóng gói)

    public function deposit(float $amount): void  // phương thức (hành vi)
    {
        $this->balance += $amount;
    }

    public function getBalance(): float
    {
        return $this->balance;
    }
}

$acc = new BankAccount();   // tạo object (instance) từ class
$acc->deposit(100);
echo $acc->getBalance();    // 100
```

#### 4 tính chất nền tảng của OOP

OOP dựa trên 4 tính chất cốt lõi (sẽ đi sâu ở các câu sau):

1. **Encapsulation (Đóng gói)** — gói dữ liệu + hành vi vào object, ẩn chi tiết bên trong, chỉ phơi
   ra giao diện cần thiết (qua `public`/`private`/`protected`).
2. **Inheritance (Kế thừa)** — class con kế thừa thuộc tính/hành vi từ class cha, tái sử dụng code.
3. **Polymorphism (Đa hình)** — cùng một lời gọi method nhưng mỗi object phản hồi theo cách riêng.
4. **Abstraction (Trừu tượng)** — chỉ tập trung vào *cái gì* đối tượng làm, ẩn đi *làm thế nào*
   (qua abstract class / interface).

#### So với lập trình thủ tục

Lập trình thủ tục tách rời **dữ liệu** (biến) và **hành vi** (hàm); khi chương trình lớn dần, dữ liệu
bị nhiều hàm sửa khắp nơi → khó kiểm soát. OOP gom dữ liệu và hành vi liên quan vào cùng object → code
phản ánh đúng **mô hình thế giới thực**, dễ quản lý hơn ở quy mô lớn.

### 🎯 Tóm tắt trả lời phỏng vấn
- OOP là **mô hình lập trình tổ chức code quanh các *object*** — mỗi object **đóng gói dữ liệu
  (thuộc tính)** và **hành vi (method)** liên quan vào cùng một chỗ.
- **Class** là bản thiết kế, **object** là thực thể cụ thể (instance) tạo từ class.
- OOP dựa trên **4 tính chất**: **Encapsulation** (đóng gói), **Inheritance** (kế thừa),
  **Polymorphism** (đa hình), **Abstraction** (trừu tượng).
- Khác lập trình thủ tục ở chỗ: thủ tục tách rời dữ liệu và hàm, còn OOP **gom dữ liệu + hành vi vào
  object**, giúp mô hình hóa thế giới thực và dễ quản lý khi dự án lớn.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Ưu điểm của lập trình hướng đối tượng

#### 📖 Giới thiệu kiến thức

Ưu điểm của OOP đến trực tiếp từ 4 tính chất nền tảng:

1. **Tái sử dụng code (Reusability)** — nhờ **inheritance** và **composition**, code viết một lần
   dùng lại nhiều nơi, không phải lặp lại (đúng tinh thần **DRY**).

2. **Dễ bảo trì & mở rộng (Maintainability / Extensibility)** — code chia thành các object độc lập,
   sửa một chỗ ít ảnh hưởng nơi khác; thêm tính năng thường là **thêm class mới** thay vì sửa code cũ
   (gần với nguyên lý Open/Closed trong SOLID).

3. **Đóng gói & bảo vệ dữ liệu (Encapsulation)** — ẩn dữ liệu bên trong, chỉ cho truy cập qua method
   công khai → giảm việc dữ liệu bị sửa lung tung, **giảm side-effect**, dễ kiểm soát tính hợp lệ.

4. **Mô hình hóa thế giới thực** — object ánh xạ tự nhiên với khái niệm nghiệp vụ (User, Order,
   Product...) → code **dễ đọc, dễ hiểu**, dễ trao đổi với người khác.

5. **Linh hoạt nhờ đa hình (Polymorphism)** — có thể xử lý nhiều loại object qua cùng một giao diện,
   giúp code mềm dẻo, dễ thay thế/mở rộng mà không sửa nơi gọi.

6. **Dễ chia nhỏ & làm việc nhóm** — mỗi người/đội phụ trách một nhóm class với giao diện rõ ràng →
   phát triển song song thuận lợi.

7. **Dễ kiểm thử (Testability)** — các đơn vị tách bạch, giao diện rõ ràng nên dễ viết unit test và
   dễ **mock** phụ thuộc.

> **Cân bằng (nên nói thêm để ghi điểm):** OOP không phải "viên đạn bạc". Với script nhỏ/đơn giản, nó
> có thể **rườm rà hơn** (nhiều class/abstraction không cần thiết) và **học khó hơn** lập trình thủ
> tục. Lợi ích của OOP thể hiện rõ nhất ở **dự án lớn, cần bảo trì lâu dài**.

#### 🎯 Tóm tắt trả lời phỏng vấn
- **Tái sử dụng code** (inheritance/composition → DRY), giảm trùng lặp.
- **Dễ bảo trì & mở rộng** — code tách thành object độc lập; thêm tính năng bằng cách thêm class, ít
  đụng code cũ.
- **Đóng gói bảo vệ dữ liệu** — ẩn chi tiết, truy cập qua method → giảm side-effect, dễ kiểm soát.
- **Mô hình hóa thế giới thực** → code dễ đọc, dễ hiểu; **đa hình** giúp linh hoạt; **dễ làm việc
  nhóm** và **dễ test/mock**.
- Chốt cân bằng: OOP phát huy ở **dự án lớn cần bảo trì lâu dài**; với task nhỏ có thể **rườm rà hơn**
  so với lập trình thủ tục.

[↑ Mục lục](#mục-lục)
