<a id="mục-lục"></a>
# Backend — DevOps & Cloud

> AWS · Cloud server · Docker · Kubernetes · CI/CD · Nginx · Linux · Deploy · Monitoring.

## 📑 Mục lục

1. [Proxy là gì?](#q1)
   - 1.1. [Forward Proxy vs Reverse Proxy khác nhau thế nào?](#q1-1)
   - 1.2. [Reverse Proxy vs Load Balancer vs API Gateway](#q1-2)
2. [Phân biệt Scale ngang (Horizontal) và Scale dọc (Vertical)](#q2)
3. [Docker là gì?](#q3)
   - 3.1. [Container khác Virtual Machine (VM) thế nào?](#q3-1)
   - 3.2. [Phân biệt Image, Container, Dockerfile, Registry](#q3-2)
   - 3.3. [Dockerfile là gì? Các chỉ thị (instruction) chính](#q3-3)
   - 3.4. [Tối ưu dung lượng Docker image như thế nào?](#q3-4)
   - 3.5. [Multi-stage build là gì?](#q3-5)
   - 3.6. [Docker Volume là gì?](#q3-6)
   - 3.7. [Docker Network là gì?](#q3-7)
   - 3.8. [Docker Compose là gì? Không dùng nó được không?](#q3-8)
   - 3.9. [Build docker-compose cho Laravel / NestJS / Vue.js cần những gì?](#q3-9)
   - 3.10. [Tại sao tạo file config tên `default.conf` khi build Docker/Nginx với docker-compose?](#q3-10)
4. [Kubernetes (K8s) là gì?](#q4)
   - 4.1. [Các thành phần chính của Kubernetes (Pod, Node, Deployment, Service...)](#q4-1)
   - 4.2. [Docker và Kubernetes khác nhau thế nào?](#q4-2)
5. [CI/CD là gì?](#q5)
   - 5.1. [Phân biệt Continuous Integration / Delivery / Deployment](#q5-1)
   - 5.2. [GitHub Actions / GitLab CI: workflow file, job, step, trigger](#q5-2)
   - 5.3. [Triển khai CI/CD cho NestJS cần những gì?](#q5-3)
   - 5.4. [Deploy CI/CD cho Laravel cần những gì?](#q5-4)
6. [Kể tên một số dịch vụ của AWS và Google Cloud](#q6)
7. [Serverless là gì?](#q7)
8. [Cron job và queue job ở backend chạy nền ở đâu?](#q8)

---

<a id="q1"></a>
## 1. Proxy là gì?

### 📖 Giới thiệu kiến thức

**Proxy** (máy chủ ủy quyền / trung gian) là một thành phần đứng **ở giữa** client và server,
nhận request rồi **chuyển tiếp** (forward) thay cho một bên. Thay vì client nói chuyện trực tiếp
với server, mọi traffic đi qua proxy — nhờ đó proxy có thể **kiểm soát, sửa đổi, lọc, cache hoặc
giấu** thông tin của request/response.

Mô hình cơ bản:

```
[Client]  →  [Proxy]  →  [Server]
          ←           ←
```

Có **2 loại chính**, phân biệt theo việc proxy "đại diện cho ai":

1. **Forward Proxy** — đại diện cho **client**. Client cấu hình đi qua proxy để ra Internet.
   Server đích chỉ thấy IP của proxy, không thấy client thật.
   - Dùng để: ẩn danh client, vượt chặn địa lý, lọc/giám sát traffic trong mạng công ty,
     cache nội dung dùng chung. Ví dụ: proxy của công ty, Squid, VPN (gần giống).

2. **Reverse Proxy** — đại diện cho **server**. Client gửi request tới proxy như thể đó là server,
   proxy mới chuyển tiếp tới các backend phía sau. Client **không biết** có bao nhiêu server thật.
   - Dùng để: **load balancing**, **SSL/TLS termination** (giải mã HTTPS tập trung),
     **caching**, **nén (gzip)**, che giấu kiến trúc nội bộ, chặn tấn công (WAF), serve static file.
   - Công cụ phổ biến: **Nginx**, HAProxy, Apache (mod_proxy), Traefik, Envoy, AWS ELB/ALB.

**Lợi ích chung của proxy:** thêm một lớp **trung gian để kiểm soát** → bảo mật (ẩn IP, lọc),
hiệu năng (cache, nén), khả năng mở rộng (cân tải), và linh hoạt (định tuyến, A/B testing).

> Liên hệ: về mặt thiết kế phần mềm, Proxy cũng là một **Design Pattern** (một object đại diện
> cho object khác để kiểm soát truy cập) — xem `Common/solid-design-pattern.md` mục 2.6.
> Ý tưởng giống nhau: một lớp trung gian đứng trước "đối tượng thật".

**Ví dụ cấu hình Reverse Proxy với Nginx:**

```nginx
server {
    listen 80;
    server_name myapp.com;

    location / {
        # chuyển tiếp request tới app Node/PHP chạy nội bộ cổng 3000
        proxy_pass http://127.0.0.1:3000;

        # giữ lại thông tin client thật cho backend
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 🎯 Tóm tắt trả lời phỏng vấn

- Proxy là **máy chủ trung gian** đứng giữa client và server, nhận và **chuyển tiếp** request thay cho một bên.
- **Forward Proxy** đại diện cho *client* (ẩn danh, lọc, cache, vượt chặn) — server không thấy client thật.
- **Reverse Proxy** đại diện cho *server* (Nginx, HAProxy): làm **load balancing, SSL termination, cache,
  nén, che giấu backend, chống tấn công** — client không thấy server thật.
- Lợi ích: thêm lớp **kiểm soát** giúp tăng **bảo mật, hiệu năng và khả năng mở rộng**.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Forward Proxy vs Reverse Proxy khác nhau thế nào?

#### 📖 Giới thiệu kiến thức

Cả hai cùng là proxy, khác nhau ở **vị trí trong luồng và đại diện cho bên nào**:

| Tiêu chí | Forward Proxy | Reverse Proxy |
|----------|---------------|---------------|
| Đại diện cho | **Client** | **Server** |
| Ai cấu hình | Client trỏ traffic qua proxy | Đặt trước server, client không biết |
| Giấu ai | Giấu **client** khỏi server | Giấu **server** khỏi client |
| Đặt ở đâu | Gần phía client (mạng nội bộ, ISP) | Gần phía server (data center) |
| Mục đích chính | Ẩn danh, lọc, vượt chặn, cache outbound | Load balance, SSL termination, cache, bảo vệ backend |
| Ví dụ | Squid, proxy công ty | Nginx, HAProxy, Cloudflare, AWS ALB |

Cách nhớ nhanh: **Forward** = bảo vệ/đại diện cho người *đi ra* (client); **Reverse** = bảo vệ/đại diện
cho người *nhận vào* (server). Cùng một phần mềm (như Nginx) có thể đóng cả hai vai tùy cấu hình.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Forward** đứng trước *client*, giấu client khỏi server (ẩn danh, lọc, vượt chặn).
- **Reverse** đứng trước *server*, giấu server khỏi client (load balance, SSL, cache, bảo vệ).
- Mẹo nhớ: Forward đại diện bên *gửi*, Reverse đại diện bên *nhận*.

[↑ Mục lục](#mục-lục)

<a id="q1-2"></a>
### 1.2. Reverse Proxy vs Load Balancer vs API Gateway

#### 📖 Giới thiệu kiến thức

Ba khái niệm này chồng lấn nhau và thường gây nhầm — thực tế một sản phẩm (Nginx, Envoy) có thể
làm cả ba. Phân biệt theo **mục đích cốt lõi**:

- **Reverse Proxy**: khái niệm **rộng nhất** — bất kỳ proxy nào đứng trước server để chuyển tiếp request.
  Load Balancer và API Gateway về bản chất *đều là* một dạng reverse proxy chuyên biệt.

- **Load Balancer**: reverse proxy tập trung vào việc **phân phối tải** đều cho nhiều backend giống nhau
  (round-robin, least-connections, IP-hash...), kèm **health check** để loại server chết.
  Mục tiêu: tăng throughput, độ sẵn sàng (high availability). Hoạt động ở **L4 (TCP)** hoặc **L7 (HTTP)**.

- **API Gateway**: reverse proxy ở **tầng ứng dụng (L7)** cho kiến trúc **microservice**, làm điểm vào
  duy nhất và xử lý các tác vụ **cross-cutting**: định tuyến theo path tới đúng service, **xác thực/phân
  quyền**, **rate limiting**, transform request/response, gộp API, logging/metrics. Ví dụ: Kong, AWS API
  Gateway, Spring Cloud Gateway.

Tóm lại: Reverse Proxy là *khái niệm gốc*; Load Balancer nhấn mạnh **cân tải**; API Gateway nhấn mạnh
**logic ứng dụng + quản trị API** cho microservice.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Reverse Proxy**: khái niệm rộng — proxy đứng trước server, chuyển tiếp request.
- **Load Balancer**: reverse proxy chuyên **phân phối tải** + health check cho nhiều backend giống nhau (L4/L7).
- **API Gateway**: reverse proxy L7 cho **microservice** — điểm vào duy nhất, lo auth, rate limit, routing, transform.
- Một công cụ (Nginx, Envoy) có thể đảm nhận cả ba vai trò.

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Phân biệt Scale ngang (Horizontal) và Scale dọc (Vertical)

### 📖 Giới thiệu kiến thức

Khi hệ thống tải tăng, ta cần **mở rộng (scale)** để chịu được nhiều request/dữ liệu hơn. Có **hai hướng**
mở rộng, khác nhau ở "tăng sức mạnh của một máy" hay "tăng số lượng máy":

- **Scale dọc (Vertical Scaling / Scale Up):** nâng cấp **cùng một máy** mạnh hơn — thêm **CPU, RAM, SSD,
  băng thông**. Giống việc đổi chiếc laptop 8GB RAM sang 64GB RAM.
- **Scale ngang (Horizontal Scaling / Scale Out):** thêm **nhiều máy** chạy song song, chia sẻ tải. Giống
  việc dùng 4 chiếc laptop 16GB thay vì 1 chiếc 64GB.

```
Scale dọc (Scale Up):           Scale ngang (Scale Out):
  ┌──────────────┐                ┌──┐ ┌──┐ ┌──┐ ┌──┐
  │   1 máy      │  (máy to hơn)  │n1│ │n2│ │n3│ │n4│  (thêm node)
  │   siêu to    │                └──┘ └──┘ └──┘ └──┘
  └──────────────┘                       ↑ Load Balancer chia tải ↑
```

**So sánh chi tiết:**

| Tiêu chí | Scale dọc (Vertical) | Scale ngang (Horizontal) |
|----------|----------------------|--------------------------|
| Cách làm | Nâng cấp **1 máy** (CPU/RAM/SSD) | Thêm **nhiều máy** song song |
| Giới hạn | **Có trần** — giới hạn phần cứng của 1 máy | **Gần như không giới hạn** (theo ngân sách) |
| Downtime | Thường **cần** (reboot/resize máy) | **Không cần** (thêm/bớt node tận gốc) |
| Độ phức tạp | **Đơn giản** (vẫn 1 máy) | **Phức tạp** — cần load balancer, chia trạng thái |
| Điểm lỗi đơn (SPOF) | **Có** (máy đó chết = sập toàn bộ) | **Không** (còn node khác gánh) |
| Cần đổi code? | Thường **không** | Thường **có** — app phải **stateless** |
| Chi phí | Máy càng to **đắt phi tuyến tính** | **Rẻ tuyến tính** (nhiều máy nhỏ) |
| Phù hợp | **CSDL quan hệ**, app đơn giản, dữ liệu nặng | **Web app**, **microservice**, traffic lớn |

**Tại sao scale ngang khó hơn? — vì bài toán "trạng thái" (state):**

- Để nhiều máy cùng phục vụ, app phải **stateless** — không lưu session/data người dùng **trong RAM của một
  máy** (nếu lưu, request tiếp theo rơi vào máy khác sẽ "mất" session).
- Giải pháp: đẩy trạng thái ra **ngoài** — session vào **Redis/DB**, file vào **S3/shared storage**.
- Cần một **Load Balancer** đứng trước chia request đều cho các node (xem câu 1.2).
- **CSDL là điểm nghẽn:** mở rộng app server dễ, nhưng **CSDL quan hệ rất khó scale ngang** (phải sharding,
  read replica) → nhiều hệ thống scale ngang app rồi bị "kẹt" ở DB.

**Khi nào chọn cái nào?**

- **Scale dọc:** bài toán **tạm thời**, app đơn giản, hoặc **CSDL quan hệ** (sharding phức tạp), cần nhanh
  mà không muốn sửa code.
- **Scale ngang:** cần **high availability**, **traffic lớn**, **microservice**, hoặc khi đã **chạm trần** 1 máy.
- **Thực tế:** đa số hệ thống dùng **cả hai** — scale dọc tới mức hợp lý, rồi scale ngang ra.

**Ví dụ trên AWS / Kubernetes:**

```bash
# Scale dọc: đổi loại instance EC2 mạnh hơn (t3.medium → t3.large)
aws ec2 modify-instance-attribute --instance-id i-xxx --instance-type t3.large

# Scale ngang: Auto Scaling Group tăng số EC2 giống nhau
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg --desired-capacity 5
```

```yaml
# Kubernetes: scale ngang bằng cách tăng replica (HPA có thể tự động theo tải)
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 5          # chạy 5 pod giống nhau; Service + kube-proxy làm load balancer
```

> Cũng có thể **scale dọc trong K8s** bằng cách tăng `resources.requests/limits` (CPU/RAM) của từng pod — nhưng
> bị giới hạn bởi node lớn nhất trong cluster.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Scale dọc (Vertical/Scale Up):** nâng cấp **một máy** mạnh hơn (thêm CPU/RAM). Đơn giản, không phải sửa
  code, nhưng **có trần** phần cứng, **có downtime**, và **1 điểm lỗi (SPOF)**.
- **Scale ngang (Horizontal/Scale Out):** thêm **nhiều máy** song song. **Không giới hạn**, **high availability**,
  **không downtime**, nhưng **phức tạp** — cần app **stateless**, **load balancer**, và xử lý trạng thái chia sẻ.
- **Khó khăn lớn nhất của scale ngang là CSDL** — app server mở rộng dễ, còn CSDL quan hệ rất khó scale ngang
  (phải sharding/replica).
- **Chọn:** CSDL/app đơn giản/thiếu thời gian → **scale dọc**; cần HA/traffic lớn/microservice → **scale ngang**.
  Thực tế thường dùng **cả hai**.
- Ví dụ: đổi instance type EC2 (dọc) vs Auto Scaling Group / tăng replica K8s (ngang).

[↑ Mục lục](#mục-lục)

<a id="q3"></a>
## 3. Docker là gì?

### 📖 Giới thiệu kiến thức

**Docker** là nền tảng **containerization** (đóng gói theo container) phổ biến nhất, cho phép **đóng gói
một ứng dụng cùng toàn bộ thứ nó cần để chạy** — mã nguồn, runtime, thư viện, dependency, biến môi trường,
cấu hình — vào **một đơn vị chuẩn hóa gọi là container**. Container này chạy **y hệt nhau** trên mọi môi
trường có Docker: máy dev, server test, production, cloud. Nó giải quyết dứt điểm câu kinh điển
**"trên máy tôi chạy được mà"** (*"it works on my machine"*), vì môi trường chạy được đóng gói theo app.

**Container là gì?** — là một **tiến trình (process) bị cô lập** trên hệ điều hành, có "ảo giác" rằng nó
đang sở hữu riêng một hệ thống (filesystem, network, process tree). Sự cô lập này do **nhân Linux (kernel)**
cung cấp qua hai cơ chế:

- **Namespaces**: cô lập *tầm nhìn* — mỗi container chỉ thấy process, network, mount, hostname... của riêng nó.
- **Cgroups (control groups)**: giới hạn *tài nguyên* — CPU, RAM, I/O mà container được dùng.

Khác với máy ảo (VM), container **KHÔNG chứa cả một hệ điều hành riêng**. Tất cả container **dùng chung nhân
(kernel) của máy host**, chỉ đóng gói phần user-space (thư viện, binary) → nên **rất nhẹ, khởi động trong
mili-giây và tốn ít RAM** (xem so sánh chi tiết ở mục 3.1).

**Kiến trúc & thành phần chính của Docker:**

```
Dockerfile  --build-->  Image  --run-->  Container
 (công thức)          (bản đóng gói,     (instance đang
                       chỉ đọc)           chạy, ghi được)
                          ↑↓ push/pull
                     Registry (Docker Hub)
```

- **Docker Engine**: dịch vụ chạy nền (daemon `dockerd`) thực sự tạo và quản lý container; ta điều khiển
  qua **Docker CLI** (`docker ...`).
- **Image**: "bản đóng gói" **chỉ đọc**, gồm nhiều **layer** xếp chồng (cache được, tái dùng được). Là
  *khuôn* để tạo container.
- **Container**: một **instance đang chạy** của image (thêm một lớp ghi được ở trên). Có thể tạo nhiều
  container từ cùng một image.
- **Dockerfile**: file text chứa **các bước "công thức"** để build ra image.
- **Registry**: kho lưu trữ image để chia sẻ (**Docker Hub** công khai, hoặc private như AWS ECR, GitLab Registry).

> Docker chỉ đóng gói & chạy **từng container**. Khi cần chạy **nhiều container phối hợp** (app + DB + cache)
> dùng **Docker Compose**; khi cần vận hành **hàng loạt container trên nhiều máy** (auto-scale, self-heal,
> load balance) thì dùng **orchestrator** như **Kubernetes** (K8s).

**Ví dụ Dockerfile cho một app Node.js:**

```dockerfile
# 1. Ảnh nền (base image) — đã có sẵn Node 20 gọn nhẹ
FROM node:20-alpine

# 2. Thư mục làm việc bên trong container
WORKDIR /app

# 3. Copy trước file khai báo dependency để tận dụng cache layer
COPY package*.json ./
RUN npm ci --production

# 4. Copy phần code còn lại
COPY . .

# 5. Cổng app lắng nghe (chỉ mang tính tài liệu)
EXPOSE 3000

# 6. Lệnh chạy khi container khởi động
CMD ["node", "server.js"]
```

```bash
docker build -t my-app:1.0 .        # build image từ Dockerfile
docker run -d -p 8080:3000 my-app:1.0   # chạy container, map cổng 8080(host)→3000(container)
docker ps                            # liệt kê container đang chạy
docker push my-registry/my-app:1.0   # đẩy image lên registry
```

**Lợi ích chính:**

- **Nhất quán môi trường** (dev = test = prod) → hết cảnh "máy tôi chạy được".
- **Portable**: chạy ở đâu có Docker cũng được (on-prem, mọi cloud).
- **Nhẹ & nhanh**: chia sẻ kernel host → khởi động mili-giây, tốn ít tài nguyên hơn VM nhiều.
- **Cô lập**: mỗi app + dependency riêng, không xung đột phiên bản thư viện.
- **Hợp CI/CD & microservice**: mỗi service đóng gói riêng, deploy/scale độc lập; build image một lần chạy mọi nơi.

### 🎯 Tóm tắt trả lời phỏng vấn

- **Docker** là nền tảng **containerization**: đóng gói app **cùng mọi dependency** (runtime, thư viện, config)
  vào **container** chạy **y hệt nhau ở mọi môi trường** → giải quyết *"trên máy tôi chạy được mà"*.
- **Container** là **process bị cô lập** nhờ **namespaces** (cô lập tầm nhìn) + **cgroups** (giới hạn tài
  nguyên) của kernel; **dùng chung kernel host** nên **nhẹ, khởi động mili-giây**, khác với VM có OS riêng.
- **4 khái niệm lõi:** **Dockerfile** (công thức) → build ra **Image** (bản đóng gói chỉ đọc, nhiều layer) →
  chạy thành **Container** (instance đang chạy) → chia sẻ qua **Registry** (Docker Hub).
- **Lợi ích:** nhất quán môi trường, portable, nhẹ/nhanh, cô lập, hợp **CI/CD & microservice**.
- Nhiều container phối hợp → **Docker Compose**; vận hành quy mô lớn nhiều máy → **Kubernetes**.

[↑ Mục lục](#mục-lục)

<a id="q3-1"></a>
### 3.1. Container khác Virtual Machine (VM) thế nào?

#### 📖 Giới thiệu kiến thức

Cả hai đều tạo môi trường **cô lập** để chạy app, nhưng **cô lập ở tầng khác nhau**:

- **VM** ảo hóa ở **tầng phần cứng**: một phần mềm **Hypervisor** (VMware, VirtualBox, KVM) mô phỏng phần
  cứng ảo, rồi **mỗi VM cài trọn một Guest OS riêng** (cả một kernel). Rất cô lập nhưng **nặng** — mỗi VM
  tốn hàng GB đĩa/RAM và khởi động cỡ **phút**.
- **Container** ảo hóa ở **tầng hệ điều hành**: **dùng chung kernel** của host, chỉ đóng gói phần user-space.
  → **nhẹ** (MB), khởi động cỡ **mili-giây**, chạy được nhiều container hơn trên cùng phần cứng.

```
   Virtual Machine                     Container
┌────────────────────┐        ┌────────────────────┐
│  App A  │  App B    │        │  App A  │  App B    │
│  Bins/Libs (x2)     │        │  Bins/Libs (x2)     │
│  Guest OS │ Guest OS│ ← nặng │   (KHÔNG có OS riêng)│ ← nhẹ
│     Hypervisor      │        │   Docker Engine     │
│      Host OS        │        │      Host OS        │
│      Hardware       │        │      Hardware       │
└────────────────────┘        └────────────────────┘
```

| Tiêu chí | Virtual Machine | Container |
|----------|-----------------|-----------|
| Ảo hóa tầng | Phần cứng (Hypervisor) | Hệ điều hành (kernel dùng chung) |
| Có OS riêng? | **Có** — mỗi VM 1 Guest OS | **Không** — chung kernel host |
| Kích thước | **GB** | **MB** |
| Khởi động | **Giây → phút** | **Mili-giây** |
| Mức cô lập | **Mạnh hơn** (tách cả kernel) | Nhẹ hơn (chung kernel) |
| Overhead | Cao | Rất thấp |

**Đánh đổi:** VM cô lập **mạnh và an toàn hơn** (kernel riêng, thoát container rủi ro hơn), hợp khi cần chạy
**OS khác nhau** hoặc yêu cầu bảo mật cao. Container **nhẹ, nhanh, mật độ cao**, hợp microservice & CI/CD.
Thực tế thường **kết hợp**: chạy container **bên trong** các VM trên cloud.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **VM** ảo hóa **phần cứng** qua Hypervisor, **mỗi VM có OS riêng** → nặng (GB), khởi động chậm (phút), cô lập mạnh.
- **Container** ảo hóa **hệ điều hành**, **dùng chung kernel host**, chỉ đóng gói app + thư viện → nhẹ (MB),
  khởi động mili-giây, mật độ cao.
- **Đánh đổi:** VM an toàn/cô lập hơn; container nhanh/nhẹ/tiết kiệm hơn. Thực tế hay chạy container **trong** VM.

[↑ Mục lục](#mục-lục)

<a id="q3-2"></a>
### 3.2. Phân biệt Image, Container, Dockerfile, Registry

#### 📖 Giới thiệu kiến thức

Bốn khái niệm này thường bị lẫn — hiểu bằng **phép ẩn dụ lập trình / nấu ăn**:

| Khái niệm | Là gì | Ẩn dụ |
|-----------|-------|-------|
| **Dockerfile** | File text chứa **các bước công thức** để build image (`FROM`, `COPY`, `RUN`, `CMD`...) | **Công thức nấu ăn** |
| **Image** | Bản đóng gói **chỉ đọc**, bất biến, gồm nhiều **layer**; là *khuôn* tạo container | **Class** (khuôn) / món ăn đã đóng hộp |
| **Container** | **Instance đang chạy** của image (thêm lớp ghi được); tạo/xóa/nhân bản được | **Object** (thể hiện) / món ăn dọn ra |
| **Registry** | Kho **lưu trữ & chia sẻ image** (Docker Hub, ECR, GitLab Registry) | **GitHub cho image** |

Luồng vòng đời điển hình:

```
viết Dockerfile → docker build → Image → docker push → Registry
                                   │                        │
                          docker run                 docker pull (máy khác)
                                   ↓
                              Container (đang chạy)
```

Điểm mấu chốt:

- **Một image → nhiều container**: giống *class → nhiều object*. Chạy `docker run` 3 lần từ một image được 3
  container độc lập.
- **Image bất biến (immutable)**; thay đổi lúc chạy nằm ở **lớp ghi** của container, container xóa đi là mất
  (muốn giữ dữ liệu phải dùng **volume**).
- **Layer & cache**: mỗi lệnh trong Dockerfile tạo một layer; build lại chỉ chạy layer thay đổi → **nhanh**.
  Vì vậy nên `COPY package.json` + cài dependency **trước** rồi mới `COPY` code (như ví dụ mục 3).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Dockerfile** = **công thức** (các bước build). → build ra **Image** = **bản đóng gói chỉ đọc, bất biến,
  nhiều layer** (như *class*). → chạy thành **Container** = **instance đang chạy** (như *object*). → chia sẻ
  qua **Registry** (Docker Hub — "GitHub cho image").
- **Một image tạo được nhiều container** độc lập.
- Image **bất biến**; dữ liệu ghi lúc chạy mất khi xóa container → cần **volume** để lưu bền.

[↑ Mục lục](#mục-lục)

<a id="q3-3"></a>
### 3.3. Dockerfile là gì? Các chỉ thị (instruction) chính

#### 📖 Giới thiệu kiến thức

**Dockerfile** là một **file text chứa "công thức" (recipe)** — một danh sách **chỉ thị (instruction)** mà
Docker đọc **từ trên xuống** để **build ra một image** (xem Image ở mục 3.2). Mỗi dòng mô tả một bước: chọn ảnh
nền, copy code, cài dependency, đặt lệnh chạy... Từ Dockerfile, chạy `docker build` sẽ ra một image; từ image
chạy `docker run` sẽ ra container.

**Mỗi chỉ thị tạo một LAYER (lớp):** image gồm nhiều layer xếp chồng, **cache lại được**. Khi build lại, Docker
**tái dùng cache** cho các layer không đổi và chỉ chạy lại từ layer đầu tiên bị thay đổi trở đi. Đây là lý do
**thứ tự các dòng rất quan trọng** (xem tối ưu bên dưới).

**Các chỉ thị chính:**

| Chỉ thị | Ý nghĩa |
|---------|---------|
| `FROM` | **Ảnh nền (base image)** để bắt đầu, vd `FROM node:20-alpine`. **Bắt buộc**, luôn ở đầu. |
| `WORKDIR` | Đặt **thư mục làm việc** trong image (các lệnh sau chạy tại đây). |
| `COPY` | Copy file/thư mục từ **host** vào image. |
| `ADD` | Như `COPY` nhưng thêm: tự giải nén `.tar`, tải được URL. **Khuyến nghị dùng `COPY`** trừ khi cần các tính năng đó. |
| `RUN` | **Chạy lệnh khi BUILD** (cài package, biên dịch...), tạo layer mới. Vd `RUN npm ci`. |
| `ENV` | Đặt **biến môi trường** (tồn tại cả lúc build lẫn lúc chạy). |
| `ARG` | Biến **chỉ có lúc build** (truyền qua `--build-arg`). |
| `EXPOSE` | **Khai báo tài liệu** cổng container lắng nghe (không tự mở cổng — vẫn cần `-p` khi run). |
| `VOLUME` | Đánh dấu một thư mục là volume (dữ liệu nên tách ra — xem mục 3.4). |
| `USER` | Chạy các lệnh sau (và container) dưới **user không phải root** (bảo mật). |
| `CMD` | **Lệnh mặc định chạy khi container KHỞI ĐỘNG**; bị `docker run ... <cmd>` ghi đè. Chỉ 1 cái có hiệu lực. |
| `ENTRYPOINT` | Lệnh **cố định** khi khởi động; không dễ bị ghi đè — thường kết hợp với `CMD` làm tham số. |

**`CMD` vs `ENTRYPOINT` (câu hay hỏi):** `ENTRYPOINT` định nghĩa **"chạy cái gì"** (cố định), còn `CMD` cung cấp
**tham số mặc định** có thể ghi đè. Ví dụ `ENTRYPOINT ["php"]` + `CMD ["artisan","serve"]` → mặc định chạy
`php artisan serve`, nhưng `docker run img artisan migrate` sẽ thành `php artisan migrate`.

**Ví dụ Dockerfile cơ bản (Laravel PHP-FPM):**

```dockerfile
# Lưu ý: comment trong Dockerfile phải nằm ở ĐẦU DÒNG, không viết sau lệnh trên cùng dòng
FROM php:8.3-fpm-alpine
WORKDIR /var/www

# Cài extension PHP cần cho Laravel
RUN docker-php-ext-install pdo_mysql

# Copy TRƯỚC file khai báo dependency → layer cài đặt được cache nếu deps không đổi
COPY composer.json composer.lock ./
RUN composer install --no-dev --no-scripts

# Rồi mới copy toàn bộ code
COPY . .
# Phân quyền thư mục ghi
RUN chown -R www-data:www-data storage
# Chạy dưới user không phải root (bảo mật)
USER www-data
EXPOSE 9000
CMD ["php-fpm"]
```

**Tối ưu layer cache — nguyên tắc "cái ít đổi để trên, cái hay đổi để dưới":** copy file dependency
(`composer.json` / `package.json`) và cài đặt **trước**, rồi mới `COPY . .` phần code. Vì code thay đổi liên tục
còn dependency ít đổi → giữ được cache của bước cài đặt (chậm nhất), build lại **nhanh hơn nhiều**.

**Multi-stage build (nâng cao) — cho image production nhỏ, sạch:** dùng **nhiều `FROM`**, một stage để **build**
(cần compiler, dev-dependency), stage cuối chỉ **copy sản phẩm** sang ảnh nền gọn → image cuối **không chứa
công cụ build**, nhẹ và an toàn hơn. Rất hợp Vue/NestJS:

```dockerfile
# --- Stage 1: build ra file tĩnh của Vue ---
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# Sinh thư mục dist/
RUN npm run build

# --- Stage 2: chỉ lấy dist/ cho Nginx serve ---
FROM nginx:alpine
# Copy sản phẩm build từ stage 'build' sang ảnh Nginx
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Best practices thường được hỏi:**

- **Ảnh nền nhỏ** (`-alpine`, `-slim`) → image gọn, ít lỗ hổng.
- **`.dockerignore`** để loại `node_modules`, `.git`, `.env`... khỏi build context → build nhanh, tránh lộ secret.
- **Gộp lệnh `RUN`** liên quan bằng `&&` để bớt layer; dọn cache trong cùng layer (`apt-get clean`).
- **Pin phiên bản** ảnh nền (`node:20-alpine`, không `node:latest`) để build **ổn định, lặp lại được**.
- **Chạy non-root** (`USER`) để giảm rủi ro bảo mật.
- **Không copy secret** vào image; dùng `ARG`/biến môi trường lúc chạy.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Dockerfile** = **file text chứa "công thức"** gồm các **chỉ thị** Docker đọc từ trên xuống để **build ra
  image** (`docker build`). Mỗi chỉ thị tạo một **layer** được **cache**.
- **Chỉ thị chính:** `FROM` (ảnh nền), `WORKDIR`, `COPY`/`ADD`, `RUN` (chạy lúc build), `ENV`/`ARG`, `EXPOSE`
  (khai báo cổng), `USER`, và **`CMD`/`ENTRYPOINT`** (lệnh chạy khi container khởi động).
- **`CMD` vs `ENTRYPOINT`:** `ENTRYPOINT` = "chạy cái gì" (cố định), `CMD` = tham số mặc định (dễ ghi đè).
- **Tối ưu cache:** copy file dependency + cài đặt **trước**, copy code **sau** → tận dụng cache, build lại nhanh.
- **Multi-stage build:** stage build (có compiler) + stage runtime chỉ copy sản phẩm → image production **nhỏ,
  sạch** (rất hợp Vue/NestJS).
- **Best practices:** base image nhỏ (`alpine`), `.dockerignore`, gộp `RUN`, **pin version**, chạy **non-root**,
  không nhét secret vào image.

[↑ Mục lục](#mục-lục)

<a id="q3-4"></a>
### 3.4. Tối ưu dung lượng Docker image như thế nào?

#### 📖 Giới thiệu kiến thức

Image nhỏ mang lại nhiều lợi ích: **build & push/pull nhanh hơn**, tốn ít băng thông/đĩa, **khởi động nhanh**,
và **ít lỗ hổng bảo mật hơn** (ít gói = ít bề mặt tấn công). Một image Node dựng ẩu có thể **hơn 1GB**, trong
khi làm đúng chỉ còn **~100–150MB**. Muốn tối ưu, phải hiểu **cách image được tạo** (từ Dockerfile — mục 3.3).

**Nguyên tắc cốt lõi cần nhớ: layer là bất biến và cộng dồn.** Mỗi chỉ thị tạo một layer **chồng thêm**; **xóa
file ở một layer sau KHÔNG làm image nhỏ đi** — file vẫn nằm trong layer cũ, layer sau chỉ "đánh dấu đã xóa".
→ Muốn giảm size thật, phải **xóa ngay trong cùng layer (cùng lệnh `RUN`) đã tạo ra file đó**.

**Các kỹ thuật tối ưu (từ hiệu quả nhất):**

**1) Chọn base image nhỏ.** Đây là đòn bẩy lớn nhất:

| Base image | Kích thước xấp xỉ |
|------------|-------------------|
| `node:20` (Debian đầy đủ) | ~1.1 GB |
| `node:20-slim` | ~200 MB |
| `node:20-alpine` | ~130 MB |
| `distroless` / `scratch` | vài MB (chỉ chứa app) |

`alpine` (Linux siêu gọn) và `slim` là lựa chọn phổ biến; `distroless`/`scratch` hợp binary tĩnh (Go).

**2) Dùng multi-stage build** (mục 3.3) — kỹ thuật quan trọng nhất cho app phải compile: stage build chứa
compiler + dev-dependency, **stage cuối chỉ copy sản phẩm** → image production **không mang theo công cụ build**.

```dockerfile
# Stage build: có đủ toolchain
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage runtime: chỉ giữ dist + production deps
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=build /app/dist ./dist
CMD ["node", "dist/main.js"]
```

**3) Chỉ cài dependency production.** Bỏ dev-dependency: `npm ci --omit=dev`, `composer install --no-dev`,
`pip install --no-cache-dir`.

**4) Gộp lệnh `RUN` và dọn cache TRONG CÙNG layer.** Cài rồi xóa cache ở **một** `RUN`:

```dockerfile
# ĐÚNG: cài và dọn trong cùng 1 layer → không để lại rác
RUN apt-get update \
 && apt-get install -y --no-install-recommends some-lib \
 && rm -rf /var/lib/apt/lists/*
```

```dockerfile
# SAI: xóa ở layer sau KHÔNG giảm dung lượng (rác vẫn nằm ở layer trước)
RUN apt-get update && apt-get install -y some-lib
RUN rm -rf /var/lib/apt/lists/*
```

**5) Dùng `.dockerignore`.** Loại `node_modules`, `.git`, `vendor`, file test, `.env`... khỏi **build context**
→ không bị copy vào image, build nhanh hơn, tránh lộ secret:

```
node_modules
.git
vendor
*.log
.env
tests
```

**6) Giảm số lệnh tạo layer & bỏ package không cần** (`--no-install-recommends`, không cài công cụ debug vào
image production).

**7) Kiểm tra image nặng ở đâu.** Dùng `docker history <image>` xem size từng layer, hoặc công cụ **`dive`** để
soi chi tiết layer nào phình to và vì sao.

```bash
docker images                 # xem tổng dung lượng các image
docker history my-app:1.0     # size từng layer → biết layer nào nặng
```

> Lưu ý phân biệt: **`.dockerignore`** giảm *build context* (không giảm trực tiếp image nhưng tránh copy rác);
> còn **multi-stage + base nhỏ + dọn cache cùng layer** mới là thứ **giảm dung lượng image cuối** nhiều nhất.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Vì sao:** image nhỏ → build/push/pull nhanh, tốn ít đĩa/băng thông, **ít lỗ hổng bảo mật**.
- **Nguyên tắc lõi:** layer **bất biến & cộng dồn** → **xóa file phải ở cùng `RUN` đã tạo ra nó**, xóa ở layer
  sau không giảm size.
- **Kỹ thuật (mạnh nhất trước):**
  1. **Base image nhỏ** (`alpine`/`slim`/`distroless`) — đòn bẩy lớn nhất.
  2. **Multi-stage build** — stage build có toolchain, stage cuối chỉ copy sản phẩm.
  3. **Chỉ cài production deps** (`--omit=dev`, `--no-dev`).
  4. **Gộp `RUN` + dọn cache trong cùng layer** (`&& rm -rf /var/lib/apt/lists/*`).
  5. **`.dockerignore`** loại `node_modules`/`.git`/`.env`.
  6. Bớt package thừa (`--no-install-recommends`).
- **Chẩn đoán:** `docker history` / công cụ `dive` để tìm layer nặng.

[↑ Mục lục](#mục-lục)

<a id="q3-5"></a>
### 3.5. Multi-stage build là gì?

#### 📖 Giới thiệu kiến thức

**Multi-stage build** là kỹ thuật viết **một Dockerfile có NHIỀU `FROM`** — mỗi `FROM` mở một **stage (giai
đoạn)** riêng — trong đó các stage sau có thể **copy sản phẩm từ stage trước** bằng `COPY --from=`. Image cuối
cùng **chỉ giữ lại stage cuối**; mọi thứ ở các stage trước (compiler, dev-dependency, mã nguồn, file trung
gian) **bị vứt bỏ**. Đây là cách chuẩn để có **image production nhỏ và sạch** (liên hệ tối ưu image ở mục 3.4).

**Vấn đề nó giải quyết:** nhiều app cần **công cụ build** (compiler, `node_modules` dev, SDK) để tạo ra sản
phẩm, nhưng lúc **chạy** thì **không cần** những thứ đó. Trước khi có multi-stage, người ta phải:

- hoặc nhét cả toolchain vào image → image **phình to, nhiều lỗ hổng**;
- hoặc dùng **2 Dockerfile + script** (build ở image này, copy artifact sang image kia) → **phức tạp, khó bảo trì**.

Multi-stage gộp tất cả vào **một Dockerfile** mà vẫn cho image cuối gọn.

**Cú pháp cốt lõi:**

- `FROM <image> AS <tên-stage>` — đặt tên cho stage để tham chiếu.
- `COPY --from=<tên-stage> <nguồn> <đích>` — copy file **từ stage khác** (chỉ lấy sản phẩm, không lấy toolchain).
- `COPY --from=<image-ngoài>` — thậm chí copy thẳng từ **một image khác** (vd `COPY --from=nginx:latest ...`).
- `docker build --target <tên-stage>` — build **dừng ở một stage** cụ thể (hữu ích để tách stage `dev`/`test`).

**Ví dụ 1 — Vue.js (build ra file tĩnh → Nginx serve):**

```dockerfile
# Stage 'build': cần Node + toolchain để chạy npm run build
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage cuối: chỉ Nginx + thư mục dist/, KHÔNG có Node/node_modules
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Image cuối chỉ là `nginx:alpine` + vài file tĩnh (~vài chục MB), không mang theo Node và `node_modules` nặng nề.

**Ví dụ 2 — NestJS (build TypeScript → runtime Node gọn):**

```dockerfile
# Stage build: có devDependency để compile TS → JS
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage runtime: chỉ production deps + thư mục dist đã compile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=build /app/dist ./dist
CMD ["node", "dist/main.js"]
```

**Lợi ích:**

- **Image production nhỏ, sạch:** không chứa compiler, dev-dependency, mã nguồn gốc → nhẹ hơn nhiều.
- **Bảo mật hơn:** ít gói/công cụ = **ít bề mặt tấn công**; không lộ source/toolchain trong image chạy thật.
- **Một Dockerfile duy nhất:** tách bạch **build** vs **runtime** mà không cần script/nhiều file.
- **Linh hoạt:** đặt tên stage, dùng `--target` để build riêng stage `dev`/`test`/`prod`.

> Multi-stage chính là **kỹ thuật quan trọng nhất** trong danh sách tối ưu image (mục 3.4) — đặc biệt với các
> ngôn ngữ/framework cần compile (Vue, NestJS, Go, Java). Với Go/binary tĩnh, stage cuối thậm chí có thể là
> `scratch` (rỗng hoàn toàn) → image chỉ vài MB.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Multi-stage build** = một Dockerfile có **nhiều `FROM`** (nhiều **stage**); stage sau **`COPY --from=`** sản
  phẩm của stage trước, image cuối **chỉ giữ stage cuối** → **vứt bỏ toolchain/dev-deps/source**.
- **Giải quyết:** app cần công cụ build lúc **build** nhưng không cần lúc **chạy** → tránh image phình to hoặc
  phải dùng 2 Dockerfile + script.
- **Cú pháp:** `FROM x AS build` → `COPY --from=build /app/dist ...`; có thể `COPY --from=<image-ngoài>`;
  `docker build --target <stage>` để dừng ở stage cụ thể.
- **Lợi ích:** image production **nhỏ, sạch, an toàn hơn**; **một** Dockerfile tách bạch build vs runtime.
- Là **kỹ thuật tối ưu image quan trọng nhất** cho app cần compile (Vue/NestJS/Go — Go có thể kết thúc ở `scratch`).

[↑ Mục lục](#mục-lục)

<a id="q3-6"></a>
### 3.6. Docker Volume là gì?

#### 📖 Giới thiệu kiến thức

Mặc định, mọi thay đổi ghi bên trong một container nằm ở **lớp ghi (writable layer)** của riêng container đó —
và lớp này **biến mất khi container bị xóa** (container **phù du/ephemeral**, xem mục 3.2). Nghĩa là: xóa
container MySQL → **mất sạch dữ liệu**. **Docker Volume** ra đời để giải quyết đúng vấn đề này: nó là **cơ chế
lưu trữ dữ liệu bền (persistent), tồn tại độc lập với vòng đời container** — container chết/tạo lại, dữ liệu vẫn
còn. Volume còn dùng để **chia sẻ dữ liệu** giữa host ↔ container hoặc giữa nhiều container.

**Docker có 3 kiểu "mount" dữ liệu — cần phân biệt rõ:**

| Kiểu | Là gì | Docker quản lý? | Dùng khi |
|------|-------|-----------------|----------|
| **Named volume** | Vùng lưu trữ Docker tự quản (trong `/var/lib/docker/volumes/`) | **Có** | **Lưu dữ liệu bền** (DB, uploads) |
| **Bind mount** | Map **một thư mục trên host** vào container (`./code:/app`) | Không (đường dẫn host) | **Dev**: mount source code để hot reload |
| **tmpfs mount** | Lưu trong **RAM**, không xuống đĩa | — | Dữ liệu tạm/nhạy cảm, không cần bền |

- **Named volume** là "volume" đúng nghĩa: **Docker sở hữu và quản lý**, không phụ thuộc cấu trúc thư mục của
  host → **dễ backup, di chuyển, chia sẻ**, hiệu năng tốt (nhất là trên Mac/Windows). Đây là lựa chọn chuẩn để
  giữ **dữ liệu DB**.
- **Bind mount** trỏ thẳng vào **một đường dẫn cụ thể trên máy host** → tiện cho **dev** (sửa code trên host,
  container thấy ngay), nhưng phụ thuộc máy và dễ dính vấn đề quyền/OS.
- **Anonymous volume**: giống named volume nhưng **không đặt tên** (Docker sinh tên ngẫu nhiên). Mẹo hay dùng:
  **"che" một thư mục** khỏi bị bind mount đè — ví dụ mount code `./app:/app` nhưng thêm `/app/node_modules` để
  giữ `node_modules` **được cài trong image**, không bị thư mục host ghi đè (đã dùng ở mục 3.5).

**Các lệnh thao tác volume:**

```bash
docker volume create db_data          # tạo named volume
docker volume ls                      # liệt kê
docker volume inspect db_data         # xem chi tiết (nơi lưu thật trên host)
docker volume rm db_data              # xóa
docker volume prune                   # dọn các volume không còn container nào dùng
```

**Gắn volume khi chạy container:**

```bash
# Named volume: dữ liệu MySQL sống sót kể cả khi xóa & tạo lại container
docker run -d --name mysql -v db_data:/var/lib/mysql mysql:8

# Bind mount: map code host vào container để dev
docker run -d --name app -v ./src:/var/www/html my-php-app

# Cú pháp --mount (dài, tường minh hơn -v)
docker run -d --name mysql \
  --mount source=db_data,target=/var/lib/mysql mysql:8
```

**Trong Docker Compose** (xem mục 3.4), khai báo named volume ở khối `volumes:` cấp cao rồi tham chiếu trong
service:

```yaml
services:
  mysql:
    image: mysql:8
    volumes:
      - db_data:/var/lib/mysql      # named volume → dữ liệu bền
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro   # bind mount file (read-only)
volumes:
  db_data:                          # phải khai báo named volume ở đây
```

> Điểm hay hỏi: **volume vs lớp ghi của container** — dữ liệu trong volume **không** nằm trong image, **không**
> bị mất khi xóa container, và **không** làm image phình to. Vì thế **không bao giờ** lưu dữ liệu DB trực tiếp
> trong container mà luôn tách ra volume.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Docker Volume** = cơ chế **lưu trữ dữ liệu bền (persistent)**, tồn tại **độc lập với vòng đời container**:
  container bị xóa/tạo lại thì dữ liệu vẫn còn. Cũng dùng để **chia sẻ dữ liệu** host ↔ container.
- **Vì sao cần:** lớp ghi của container **phù du**, xóa container là **mất dữ liệu** → DB, uploads phải để trên volume.
- **3 kiểu mount:** **Named volume** (Docker tự quản, chuẩn cho dữ liệu DB — dễ backup/di chuyển); **Bind mount**
  (map thư mục host, hợp **dev** hot reload); **tmpfs** (trong RAM, tạm). Thêm **anonymous volume** để "che"
  `node_modules`/`vendor` khỏi bị bind mount đè.
- **Lệnh:** `docker volume create/ls/inspect/rm/prune`; gắn bằng `-v db_data:/var/lib/mysql` hoặc `--mount`.
- Trong **Compose**: khai báo ở `volumes:` cấp cao rồi tham chiếu trong service.

[↑ Mục lục](#mục-lục)

<a id="q3-7"></a>
### 3.7. Docker Network là gì?

#### 📖 Giới thiệu kiến thức

Mỗi container mặc định có **network namespace riêng** — tức là một "ngăn mạng" cô lập với IP riêng, tách khỏi
host và các container khác (xem namespaces ở mục 3). Vậy làm sao container **nói chuyện** được với nhau, với
host, và ra Internet? → nhờ **Docker Network**: một **mạng ảo** do Docker tạo, nối các container lại để chúng
**giao tiếp**, đồng thời **kiểm soát** container nào thấy/gọi được container nào.

**Ý tưởng cốt lõi — user-defined bridge & DNS nội bộ:** khi ta tạo một mạng riêng và cắm nhiều container vào,
Docker cung cấp **DNS nội bộ** để các container **gọi nhau bằng TÊN container/service** thay vì IP (IP hay đổi).
Đây chính là lý do trong Compose, Laravel nối MySQL bằng host `mysql` (tên service) chứ không phải `localhost`
hay IP (xem mục 3.6).

**Các network driver chính:**

| Driver | Là gì | Dùng khi |
|--------|-------|----------|
| **bridge** | Mạng ảo mặc định trên **một máy**; container cùng bridge nói chuyện với nhau | **Mặc định & phổ biến nhất** — app nhiều container trên 1 host |
| **host** | Container **dùng chung network stack của host** (không cô lập, không map cổng) | Cần hiệu năng mạng tối đa, chấp nhận mất cô lập |
| **none** | **Tắt mạng** hoàn toàn cho container | Tác vụ không cần mạng, cô lập tuyệt đối |
| **overlay** | Mạng trải **nhiều máy (multi-host)** | **Docker Swarm / cluster** — container ở các host khác nhau |
| **macvlan** | Gán **MAC/IP riêng**, container xuất hiện như thiết bị vật lý trong LAN | Tích hợp hệ thống mạng truyền thống |

**Phân biệt quan trọng — default bridge vs user-defined bridge:**

- **Default bridge** (`bridge` có sẵn khi cài Docker): các container cắm vào đây **KHÔNG tự resolve tên nhau**
  (phải dùng cờ `--link` cũ kỹ). Không nên dùng cho app thật.
- **User-defined bridge** (ta tự tạo `docker network create`): **có DNS nội bộ** → container **gọi nhau bằng
  tên** tự động. Docker Compose **tự tạo sẵn một user-defined bridge** cho mỗi project → vì thế các service
  trong compose gọi nhau bằng tên "miễn phí".

**Cổng nội bộ vs publish cổng ra host (`-p`):**

- Container **cùng một network** nói chuyện trực tiếp qua **cổng nội bộ** — **không cần** `-p`. Ví dụ Laravel
  gọi `mysql:3306`, php-fpm nghe `9000` cho Nginx — đều là traffic **nội bộ**.
- Chỉ khi muốn **truy cập từ ngoài host** (browser, Postman) mới cần **publish** bằng `-p 8080:80`. → Nguyên tắc
  bảo mật: **chỉ mở cổng ra host cho những gì thật sự cần** (Nginx), còn DB/Redis để nội bộ, **không `-p`**.

**Các lệnh thao tác network:**

```bash
docker network create app-net           # tạo user-defined bridge
docker network ls                        # liệt kê các network
docker network inspect app-net           # xem chi tiết + container đang cắm
docker network connect app-net mysql     # cắm container 'mysql' vào network
docker network disconnect app-net mysql  # rút ra
docker network rm app-net                # xóa
docker network prune                     # dọn network không dùng
```

```bash
# Chạy 2 container cùng network → gọi nhau bằng TÊN
docker network create app-net
docker run -d --name mysql --network app-net mysql:8
docker run -d --name laravel --network app-net my-laravel   # trong app: host = "mysql"
```

**Trong Docker Compose** (mục 3.5): Compose tự tạo network mặc định, nhưng có thể khai báo network riêng để
**phân tách** (vd tách tầng `frontend` và `backend` để DB không lộ ra tầng ngoài):

```yaml
services:
  nginx:   { image: nginx:alpine, networks: [frontend] }
  laravel: { build: ./laravel, networks: [frontend, backend] }
  mysql:   { image: mysql:8, networks: [backend] }     # chỉ ở backend → nginx KHÔNG với tới
networks:
  frontend:
  backend:
```

> Điểm hay hỏi: **vì sao trong container không dùng `localhost` để gọi service khác?** Vì `localhost` trỏ về
> **chính container đó**, không phải container khác. Phải dùng **tên service/container** (Docker DNS phân giải)
> hoặc IP của network.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Docker Network** = **mạng ảo** Docker tạo để các container **giao tiếp** với nhau, với host và Internet,
  đồng thời **kiểm soát/cô lập** kết nối (mỗi container mặc định có network namespace riêng).
- **DNS nội bộ:** trên **user-defined bridge**, container **gọi nhau bằng TÊN** (không dùng IP hay `localhost`)
  — chính là cơ chế để service trong Compose nối nhau bằng tên (Laravel → host `mysql`).
- **Driver chính:** **bridge** (mặc định, 1 máy), **host** (chung stack host, không cô lập), **none** (tắt mạng),
  **overlay** (đa máy — Swarm/cluster), **macvlan**.
- **Cổng:** container cùng network nói chuyện qua **cổng nội bộ, không cần `-p`**; `-p` chỉ để **mở ra host** →
  bảo mật: chỉ publish cái cần (Nginx), DB/Redis để nội bộ.
- **Lệnh:** `docker network create/ls/inspect/connect/rm/prune`; trong Compose có thể tách `frontend`/`backend`
  để cô lập.

[↑ Mục lục](#mục-lục)

<a id="q3-8"></a>
### 3.8. Docker Compose là gì? Không dùng nó được không?

#### 📖 Giới thiệu kiến thức

**Docker Compose** là công cụ để **định nghĩa và chạy một ứng dụng gồm nhiều container** bằng **một file YAML**
(`compose.yaml`, trước đây `docker-compose.yml`) và điều khiển bằng **một lệnh** (`docker compose up`). Thay vì
gõ `docker run` cho từng container kèm cả rừng cờ (`-p`, `-v`, `-e`, `--network`...) rồi phải nhớ thứ tự và cách
nối chúng, ta **khai báo toàn bộ stack** — các service, network, volume — vào một file, rồi Compose dựng tất cả.

**Trước khi có Compose, ta phải làm thủ công:**

```bash
docker network create app-net
docker run -d --name mysql --network app-net -e MYSQL_ROOT_PASSWORD=root -v db:/var/lib/mysql mysql:8
docker run -d --name redis --network app-net redis:7-alpine
docker run -d --name laravel --network app-net -v ./laravel:/var/www -e DB_HOST=mysql my-laravel
docker run -d --name nginx  --network app-net -p 8080:80 -v ./laravel:/var/www nginx:alpine
# ... phải nhớ đúng thứ tự, đúng tên, đúng network cho mọi container
```

Cồng kềnh, dễ sai, khó lặp lại và khó chia sẻ cho người khác.

**Với Compose, tất cả gói gọn trong `compose.yaml`:**

```yaml
services:
  mysql:   { image: mysql:8, environment: { MYSQL_ROOT_PASSWORD: root }, volumes: [db:/var/lib/mysql] }
  redis:   { image: redis:7-alpine }
  laravel: { build: ./laravel, environment: { DB_HOST: mysql }, depends_on: [mysql, redis] }
  nginx:   { image: nginx:alpine, ports: ["8080:80"], depends_on: [laravel] }
volumes: { db: {} }
```

```bash
docker compose up -d     # dựng CẢ stack bằng 1 lệnh
docker compose down      # gỡ toàn bộ
```

**Lợi ích chính:**

- **Một lệnh, một file** thay cho hàng chục lệnh `docker run`.
- **Khai báo (declarative) & lặp lại được (reproducible):** commit `compose.yaml` vào Git → ai clone về cũng
  dựng được **môi trường y hệt** (hết cảnh "máy tôi cấu hình khác").
- **Tự tạo network mặc định** cho các service **gọi nhau bằng tên** (Laravel nối `mysql`, không cần `--link`).
- **Quản lý vòng đời cả nhóm:** `up`, `down`, `logs`, `ps`, `exec`, `restart` cho toàn stack.

**Phạm vi & giới hạn:** Compose chạy trên **MỘT máy (single host)** — tuyệt cho **dev, test, CI, demo, staging
nhỏ**. Nó **không phải orchestrator đa máy**: không auto-scale nhiều node, không self-heal khi máy chết. Việc đó
là của **Kubernetes** (câu 4) hay Docker Swarm.

> Ghi chú phiên bản: bản cũ là **`docker-compose`** (viết bằng Python, dấu gạch ngang — Compose **v1**). Bản nay
> là **`docker compose`** (plugin viết bằng Go, tích hợp trong Docker — Compose **v2**). Trường `version:` ở đầu
> file nay đã **lỗi thời**, không cần khai báo nữa.

**"Không dùng Docker Compose được không?" — ĐƯỢC, hoàn toàn.**

Compose **không phải bắt buộc** và **không thêm khả năng gì mới** so với Docker CLI — nó chỉ **đóng gói nhiều
lệnh `docker` vào một file khai báo cho tiện**. Không dùng Compose thì ta thay bằng:

- **`docker run` thủ công + `docker network create`** như ví dụ trên — làm được nhưng cồng kềnh, dễ sai.
- **Script bash / Makefile** bọc các lệnh `docker run` lại — đỡ hơn nhưng vẫn là "chế lại" Compose.
- **Chỉ 1 container duy nhất** thì thậm chí **không cần Compose** — một lệnh `docker run` là đủ.
- **Ở production đa máy**, người ta **không dùng Compose** mà dùng **Kubernetes / Docker Swarm / Nomad** (hoặc
  PaaS) vì cần scale, self-heal, rolling update trên nhiều node.

Nói cách khác: Compose là **tiện ích giúp đời dev dễ thở**, không phải thứ bắt buộc. Bỏ nó thì bạn vẫn chạy được
mọi thứ — chỉ là phải **tự tay làm lại** những gì Compose tự động hóa.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Docker Compose** = công cụ **định nghĩa & chạy app nhiều container** bằng **một file YAML** (`compose.yaml`)
  + **một lệnh** (`docker compose up`), thay cho việc gõ `docker run` từng container thủ công.
- **Lợi ích:** một lệnh dựng cả stack; **khai báo & reproducible** (commit vào Git, ai cũng dựng môi trường y
  hệt); **tự tạo network** cho service gọi nhau bằng tên; quản vòng đời cả nhóm (`up/down/logs/exec`).
- **Phạm vi:** chỉ **một máy** — hợp **dev/test/CI/staging nhỏ**; **không phải orchestrator đa máy** (việc đó là
  **Kubernetes**).
- **Không dùng được không?** **Được** — Compose không bắt buộc, không thêm khả năng mới, chỉ gói các lệnh
  `docker` cho tiện. Có thể thay bằng **`docker run` thủ công + network**, **script/Makefile**, hoặc ở production
  đa máy thì dùng **K8s/Swarm**. Bỏ Compose vẫn chạy được, chỉ là phải tự làm thủ công.

[↑ Mục lục](#mục-lục)

<a id="q3-9"></a>
### 3.9. Build docker-compose cho Laravel / NestJS / Vue.js cần những gì?

#### 📖 Giới thiệu kiến thức

**Docker Compose** là công cụ chạy **nhiều container phối hợp** bằng **một file YAML** (`compose.yaml`) và
**một lệnh** (`docker compose up`). Thay vì `docker run` thủ công từng container với cả rừng cờ, ta **khai báo**
toàn bộ stack — app, database, cache, web server — cùng cách chúng nối với nhau. Rất hợp môi trường **dev** và
các stack **nhỏ/vừa** trên **một máy** (khác Kubernetes lo nhiều máy — câu 4).

Với một hệ **Laravel (API) + NestJS (API/service) + Vue.js (frontend)**, ta cần suy nghĩ theo **các service**
và **các khối cấu hình** sau.

**1) Xác định các service (container) cần có:**

| Service | Vai trò | Base image gợi ý |
|---------|---------|------------------|
| `laravel` (php-fpm) | Chạy code PHP Laravel | `php:8.3-fpm` (+ extension: pdo_mysql, redis...) |
| `nginx` | Web server đứng trước Laravel (php-fpm không tự serve HTTP) | `nginx:alpine` |
| `nestjs` | API/service Node | `node:20-alpine` |
| `vue` | Frontend (dev: Vite server; prod: build tĩnh rồi Nginx serve) | `node:20-alpine` |
| `mysql` (hoặc `postgres`) | Cơ sở dữ liệu | `mysql:8` |
| `redis` | Cache / queue / session | `redis:7-alpine` |
| `queue` (tùy chọn) | Laravel worker chạy `queue:work` | *dùng lại image `laravel`* |

> Lưu ý đặc thù: **Laravel chạy PHP-FPM cần thêm Nginx** để nhận HTTP rồi chuyển sang FPM qua cổng 9000 — đây
> là điểm khác NestJS/Vue (Node tự lắng nghe HTTP). Vue thường **build ra file tĩnh** khi lên production và cho
> Nginx serve, còn khi dev thì chạy Vite dev server (hot reload).

**2) Các khối cấu hình cốt lõi trong `compose.yaml`:**

- **`services`** — khai báo từng container.
- **`build` vs `image`** — `build` (trỏ tới Dockerfile) cho service tự viết (Laravel, NestJS, Vue);
  `image` (kéo sẵn) cho hạ tầng (MySQL, Redis, Nginx).
- **`ports`** — map cổng host→container để truy cập từ ngoài (`"8080:80"`). Service nội bộ (MySQL, php-fpm)
  **không cần** mở cổng ra host.
- **`volumes`** — (a) **named volume** để **giữ dữ liệu bền** cho DB (không mất khi xóa container);
  (b) **bind mount** code (`./laravel:/var/www`) để **hot reload** lúc dev.
- **`environment` / `env_file`** — truyền config & secret (DB credentials, `REDIS_HOST`...).
- **`depends_on` + `healthcheck`** — thứ tự khởi động; **quan trọng:** `depends_on` chỉ đợi container *bật*,
  **chưa chắc DB đã sẵn sàng nhận kết nối** → cần `healthcheck` + `condition: service_healthy`.
- **`networks`** — cho các service **gọi nhau bằng tên service** (Laravel nối DB qua host `mysql`, không phải
  `localhost`). Compose tự tạo 1 network mặc định, đủ dùng cho đa số trường hợp.
- **`restart`** — chính sách tự khởi động lại (`unless-stopped`).

**3) Ví dụ `compose.yaml` (rút gọn cho dev):**

```yaml
services:
  # --- Laravel: PHP-FPM + Nginx ---
  laravel:
    build: ./laravel                 # Dockerfile riêng (php:8.3-fpm + extension)
    volumes:
      - ./laravel:/var/www           # bind mount code để hot reload
    environment:
      DB_HOST: mysql                 # gọi DB bằng TÊN SERVICE, không phải localhost
      REDIS_HOST: redis
    depends_on:
      mysql:
        condition: service_healthy   # đợi DB thật sự sẵn sàng
      redis:
        condition: service_started

  nginx:
    image: nginx:alpine
    ports: ["8080:80"]               # vào Laravel qua http://localhost:8080
    volumes:
      - ./laravel:/var/www
      - ./docker/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on: [laravel]

  # --- Laravel queue worker (tái dùng image laravel) ---
  queue:
    build: ./laravel
    command: php artisan queue:work --tries=3
    volumes: ["./laravel:/var/www"]
    depends_on:
      redis: { condition: service_started }

  # --- NestJS API ---
  nestjs:
    build: ./nestjs                  # node:20-alpine, CMD ["node","dist/main.js"]
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: mysql://app:secret@mysql:3306/app
    depends_on:
      mysql: { condition: service_healthy }

  # --- Vue.js (dev: Vite server) ---
  vue:
    build: ./vue                     # node:20-alpine, CMD ["npm","run","dev","--","--host"]
    ports: ["5173:5173"]
    volumes:
      - ./vue:/app
      - /app/node_modules            # anonymous volume: giữ node_modules trong container

  # --- Hạ tầng: kéo image sẵn ---
  mysql:
    image: mysql:8
    environment:
      MYSQL_DATABASE: app
      MYSQL_USER: app
      MYSQL_PASSWORD: secret
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - db_data:/var/lib/mysql       # named volume: dữ liệu bền
    healthcheck:                     # để depends_on biết khi nào DB "khỏe"
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      retries: 10
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    restart: unless-stopped

volumes:
  db_data:                           # khai báo named volume để MySQL dùng
```

**4) Những điểm hay bị sai / hay bị hỏi:**

- **Gọi nhau bằng tên service, không `localhost`**: trong container, `localhost` là chính nó → Laravel nối MySQL
  phải dùng host `mysql`. Đây là lỗi phổ biến nhất.
- **`depends_on` không đợi "sẵn sàng"** mà chỉ đợi "đã bật" → dùng **healthcheck** để tránh app kết nối DB khi
  DB chưa lên (xem `condition: service_healthy`).
- **`node_modules` / `vendor`**: khi bind mount code, cần **anonymous/named volume** đè lên `node_modules` để
  không bị thư mục rỗng của host ghi đè, và để không đồng bộ thư mục nặng này.
- **Dev vs Prod khác nhau**: dev thì **bind mount code + hot reload**; prod thì **build code vào image**
  (multi-stage build), Vue **build ra tĩnh** cho Nginx serve, **không mount code**, không mở cổng DB ra ngoài.
- **Secret**: dev để tạm trong `environment`/`.env`; prod nên dùng **secret manager**, không commit `.env`.
- **Lệnh vận hành**: `docker compose up -d` (chạy nền), `docker compose logs -f <svc>`, `docker compose exec
  laravel php artisan migrate`, `docker compose down -v` (xóa cả volume).

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Docker Compose** = khai báo **nhiều container** trong **một file YAML** + chạy bằng **một lệnh** — hợp
  **dev / stack nhỏ trên 1 máy** (khác K8s lo nhiều máy).
- **Các service cần cho Laravel + NestJS + Vue:** `laravel` (**php-fpm**) **+ `nginx`** (vì php-fpm không tự
  serve HTTP), `nestjs` (node), `vue` (Vite dev / build tĩnh cho Nginx ở prod), `mysql`/`postgres`, `redis`,
  và tùy chọn `queue` worker của Laravel.
- **Các khối cấu hình lõi:** `services`, **`build` vs `image`**, `ports`, **`volumes`** (named volume cho DB bền
  + bind mount code hot reload), `environment`/`env_file`, **`depends_on` + `healthcheck`**, `networks`, `restart`.
- **Điểm hay hỏi/sai:** gọi service **bằng tên** không phải `localhost`; `depends_on` **không đợi DB sẵn sàng**
  (phải healthcheck); volume đè **`node_modules`/`vendor`**; **dev bind mount vs prod build vào image**
  (multi-stage), không mở cổng DB ra ngoài ở prod.

[↑ Mục lục](#mục-lục)

<a id="q3-10"></a>
### 3.10. Tại sao tạo file config tên `default.conf` khi build Docker/Nginx với docker-compose?

#### 📖 Giới thiệu kiến thức

Câu hỏi này xuất phát từ dòng hay gặp trong `compose.yaml` (mục 3.9):

```yaml
nginx:
  image: nginx:alpine
  volumes:
    - ./docker/nginx.conf:/etc/nginx/conf.d/default.conf:ro   # ← vì sao lại là default.conf?
```

Muốn hiểu, phải nắm **cách image `nginx` chính thức nạp cấu hình**.

**1) Nginx nạp config theo kiểu "include cả thư mục".** File config gốc của Nginx là
`/etc/nginx/nginx.conf`. Bên trong khối `http { ... }` của nó có sẵn một dòng:

```nginx
http {
    # ... các thiết lập chung ...
    include /etc/nginx/conf.d/*.conf;   # nạp MỌI file .conf trong thư mục này
}
```

Nghĩa là Nginx **tự động gộp tất cả file `*.conf`** trong thư mục `/etc/nginx/conf.d/` vào cấu hình.
Mỗi file thường chứa một khối `server { ... }` (một virtual host). Ta **không sửa `nginx.conf` gốc**,
mà chỉ cần **thả file `.conf` của mình vào `conf.d/`** là Nginx nhận.

**2) Image `nginx` xuất xưởng đã có sẵn `default.conf`.** Khi build/chạy container từ `nginx:alpine`,
trong image **đã có** file `/etc/nginx/conf.d/default.conf`. Chính file này định nghĩa trang
"**Welcome to nginx!**" mặc định — nó khai báo `listen 80;`, `root /usr/share/nginx/html;` và serve
trang tĩnh mẫu. Đó là lý do container Nginx "trần" chạy lên đã có trang chào.

**3) Đặt tên `default.conf` để GHI ĐÈ (override) đúng file mặc định đó.** Khi ta mount (hoặc `COPY`)
file cấu hình của mình **vào đúng đường dẫn** `/etc/nginx/conf.d/default.conf`, nó **thay thế** file
mặc định của image → trang Welcome biến mất, thay bằng cấu hình thật của ta (reverse proxy sang
php-fpm cho Laravel, hoặc serve `dist/` cho Vue...). Đây là lý do cốt lõi: **trùng tên + trùng đường
dẫn = đè lên file gốc**.

> Nếu đặt tên **khác** (vd `app.conf`) và mount vào `/etc/nginx/conf.d/app.conf`, thì **cả hai** file
> đều được `include`: `default.conf` (trang Welcome) **vẫn còn**. Hậu quả: hai khối `server` cùng
> `listen 80` → Nginx lấy server mặc định theo thứ tự/`default_server`, dễ khiến request rơi vào trang
> Welcome thay vì app. Vì vậy cách gọn nhất là **đặt tên đúng `default.conf`** để đè phăng file gốc
> (hoặc phải xóa/đổi `default.conf` gốc đi).

**4) File `default.conf` chứa gì?** Chính là khối `server` định tuyến cho app. Ví dụ cho **Laravel
(PHP-FPM)** — Nginx nhận HTTP rồi đẩy PHP sang service `laravel` qua cổng FastCGI 9000:

```nginx
# ./docker/nginx.conf  →  mount tới /etc/nginx/conf.d/default.conf
server {
    listen 80;
    root /var/www/public;              # thư mục public của Laravel
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        # 'laravel' = TÊN SERVICE trong compose (Docker DNS phân giải — xem mục 3.7)
        fastcgi_pass laravel:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Ví dụ cho **Vue.js (SPA build tĩnh)** — serve file trong `dist/` và fallback `index.html` cho router:

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;        # nơi COPY dist/ vào (xem multi-stage mục 3.5)
    location / {
        try_files $uri $uri/ /index.html;   # SPA: mọi route → index.html
    }
}
```

**5) Hai cách nạp file này (đều dẫn tới `conf.d/default.conf`):**

- **Bind mount lúc dev** (như compose ở trên): `- ./docker/nginx.conf:/etc/nginx/conf.d/default.conf:ro`
  → sửa file trên host, chỉ cần `docker compose restart nginx` là áp dụng (`:ro` = read-only cho an toàn).
- **`COPY` vào lúc build (prod):** nhét thẳng config vào image để image tự đủ, không phụ thuộc host:

  ```dockerfile
  FROM nginx:alpine
  COPY docker/nginx.conf /etc/nginx/conf.d/default.conf   # đè file mặc định ngay khi build
  ```

> Tóm lại: cái tên `default.conf` **không phải quy ước bắt buộc của riêng ta**, mà là **tên file mặc
> định có sẵn trong image `nginx`**. Ta cố tình đặt trùng tên để **ghi đè** nó bằng cấu hình app của
> mình. Thư mục `conf.d/` được `include *.conf` nên bất kỳ file `.conf` nào cũng được nạp — nhưng dùng
> đúng `default.conf` là cách sạch nhất để thay thế hành vi mặc định.

#### 🎯 Tóm tắt trả lời phỏng vấn

- File `nginx.conf` gốc có dòng `include /etc/nginx/conf.d/*.conf;` → Nginx **tự nạp mọi file `.conf`**
  trong thư mục `conf.d/` (mỗi file thường là một khối `server`).
- Image `nginx` **xuất xưởng đã có sẵn** `/etc/nginx/conf.d/default.conf` — chính là trang **"Welcome to
  nginx!"** mặc định.
- Ta đặt tên file của mình là **`default.conf`** và mount/`COPY` vào **đúng đường dẫn đó** để **ghi đè**
  file mặc định → thay trang Welcome bằng cấu hình app thật (reverse proxy sang php-fpm cho Laravel,
  serve `dist/` cho Vue...).
- Nếu đặt tên khác (`app.conf`) thì `default.conf` gốc **vẫn còn** → hai `server` cùng `listen 80` dễ
  xung đột, request có thể rơi vào trang Welcome. Nên trùng tên `default.conf` để đè cho gọn.
- **Dev:** bind mount `:ro`; **Prod:** `COPY` vào image lúc build.

[↑ Mục lục](#mục-lục)

<a id="q4"></a>
## 4. Kubernetes (K8s) là gì?

### 📖 Giới thiệu kiến thức

**Kubernetes** (viết tắt **K8s** — "K" + 8 chữ + "s") là nền tảng **điều phối container (container
orchestration)** mã nguồn mở, ban đầu do Google phát triển, nay thuộc **CNCF**. Nếu **Docker** lo việc
**đóng gói và chạy từng container** (câu 3), thì **Kubernetes lo việc vận hành hàng loạt container trên
nhiều máy (cluster)** một cách **tự động**: triển khai, scale, tự hồi phục, cân tải, cập nhật không gián đoạn.

**Vì sao cần K8s?** — Khi chạy microservice ở quy mô thật, ta có **hàng trăm–nghìn container** trải trên
nhiều server. Làm thủ công thì không kham nổi các câu hỏi: container nào chạy ở máy nào? một container chết
thì ai bật lại? traffic chia thế nào? deploy version mới ra sao mà không sập? Kubernetes **tự động hóa toàn
bộ** những việc đó.

**Các khả năng cốt lõi:**

- **Tự hồi phục (self-healing):** container chết → K8s tự khởi động lại; node chết → chuyển pod sang node khác.
- **Auto-scaling:** tự tăng/giảm số bản sao (replica) theo tải (CPU/RAM/metric) bằng **HPA**.
- **Load balancing & service discovery:** tự chia traffic đều cho các pod qua **Service**.
- **Rollout & rollback:** cập nhật phiên bản **từ từ, không downtime** (rolling update); lỗi thì **quay lui**.
- **Khai báo (declarative):** ta mô tả **trạng thái mong muốn** bằng file YAML, K8s **liên tục điều chỉnh
  thực tế cho khớp** (reconciliation loop) — không cần ra lệnh từng bước.

**Mô hình khai báo — trái tim của K8s:**

```yaml
# Ta khai báo "muốn 3 bản chạy image my-app:1.0"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3                # trạng thái mong muốn: luôn có 3 pod
  selector:
    matchLabels: { app: my-app }
  template:
    metadata:
      labels: { app: my-app }
    spec:
      containers:
        - name: my-app
          image: my-app:1.0
          ports:
            - containerPort: 3000
```

Nếu 1 pod chết còn 2, K8s **tự tạo lại pod thứ 3** để khớp `replicas: 3` — đây là ý nghĩa "declarative +
self-healing". (Chi tiết Pod, Node, Service... xem mục 4.1.)

**Hệ sinh thái liên quan:** K8s không thay thế Docker mà **dùng container runtime** (containerd, CRI-O) để
chạy container. Các bản phân phối/dịch vụ phổ biến: **EKS** (AWS), **GKE** (Google), **AKS** (Azure),
**Minikube/k3s** (chạy local/nhẹ).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Kubernetes (K8s)** là nền tảng **điều phối container (orchestration)**: vận hành **hàng loạt container
  trên nhiều máy (cluster)** một cách **tự động**.
- Trong khi **Docker đóng gói & chạy từng container**, **K8s quản lý chúng ở quy mô lớn**: **self-healing,
  auto-scaling (HPA), load balancing, service discovery, rolling update & rollback**.
- Cốt lõi là mô hình **khai báo (declarative)**: ta mô tả **trạng thái mong muốn** bằng YAML, K8s **liên tục
  điều chỉnh thực tế cho khớp** (reconciliation loop).
- **Vì sao cần:** microservice quy mô lớn có hàng trăm container → không thể quản thủ công.
- Dịch vụ managed: **EKS/GKE/AKS**; local: **Minikube/k3s**.

[↑ Mục lục](#mục-lục)

<a id="q4-1"></a>
### 4.1. Các thành phần chính của Kubernetes (Pod, Node, Deployment, Service...)

#### 📖 Giới thiệu kiến thức

Một **cluster** K8s gồm **Control Plane** (bộ não ra quyết định) và các **Worker Node** (nơi chạy app).

**Phía workload (thứ ta khai báo hằng ngày), từ nhỏ tới lớn:**

- **Container**: app đã đóng gói (thường là Docker image).
- **Pod**: **đơn vị nhỏ nhất** K8s quản lý — bọc **1 (hoặc vài) container** dùng chung network & storage.
  K8s scale/schedule theo **pod**, không theo container lẻ. Pod **phù du (ephemeral)**: chết là tạo pod mới
  (IP đổi).
- **ReplicaSet**: đảm bảo **luôn đủ số pod** mong muốn (thường không dùng trực tiếp, để Deployment lo).
- **Deployment**: quản lý ReplicaSet, lo **rollout/rollback** và số **replica** — đối tượng ta khai báo nhiều nhất cho app stateless.
- **Service**: **địa chỉ mạng cố định** đứng trước một nhóm pod (pod đổi IP liên tục), làm **load balance +
  service discovery**. Loại: `ClusterIP` (nội bộ), `NodePort`, `LoadBalancer` (ra ngoài).
- **Ingress**: định tuyến **HTTP/HTTPS từ ngoài** vào các Service theo host/path (như reverse proxy L7 — câu 1.2).
- **ConfigMap / Secret**: tách **cấu hình / thông tin nhạy cảm** ra khỏi image.
- **Namespace**: chia cluster thành nhiều "không gian" logic (dev/staging/prod, team...).

**Phía hạ tầng:**

- **Node**: một **máy** (VM/vật lý) trong cluster, nơi pod thực sự chạy; trên node có **kubelet** (agent) và
  **container runtime** (containerd).
- **Control Plane**: **API Server** (cổng giao tiếp duy nhất), **etcd** (CSDL key-value lưu trạng thái
  cluster), **Scheduler** (chọn node cho pod mới), **Controller Manager** (chạy các vòng lặp reconcile).

```
Cluster
├── Control Plane:  API Server · etcd · Scheduler · Controller Manager
└── Worker Nodes:
    ├── Node 1:  [Pod(app) Pod(app)]  kubelet + runtime
    └── Node 2:  [Pod(app)         ]  kubelet + runtime
        ▲ Deployment giữ đủ replica │ Service gom & cân tải các pod
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Pod**: đơn vị nhỏ nhất, bọc 1+ container dùng chung network/storage; **phù du** (chết là thay mới, đổi IP).
- **Deployment**: giữ đủ **replica**, lo **rollout/rollback** (qua ReplicaSet) — dùng nhiều nhất cho app stateless.
- **Service**: **địa chỉ cố định** trước nhóm pod → **load balance + service discovery**; **Ingress** định tuyến
  HTTP từ ngoài theo host/path.
- **Node**: máy chạy pod (có kubelet + runtime). **Control Plane**: API Server, **etcd** (lưu trạng thái),
  Scheduler, Controller Manager.
- **ConfigMap/Secret** tách cấu hình; **Namespace** chia môi trường/team.

[↑ Mục lục](#mục-lục)

<a id="q4-2"></a>
### 4.2. Docker và Kubernetes khác nhau thế nào?

#### 📖 Giới thiệu kiến thức

Đây là câu hay bị hỏi vì dễ nhầm. Hai thứ **không cạnh tranh mà bổ sung cho nhau** — ở **hai tầng khác nhau**:

- **Docker** = công cụ **containerization**: **đóng gói** app thành image và **chạy container** trên **một máy**.
- **Kubernetes** = công cụ **orchestration**: **điều phối nhiều container trên nhiều máy** (cluster) — scale,
  self-heal, load balance, rollout. K8s **không tự tạo container** mà gọi một **runtime** (containerd) để chạy.

Cách nói gọn: **Docker tạo ra con thuyền (container); Kubernetes là thuyền trưởng điều cả hạm đội.**

| Tiêu chí | Docker | Kubernetes |
|----------|--------|------------|
| Vai trò | Đóng gói & chạy container | Điều phối container ở quy mô lớn |
| Phạm vi | Thường **1 máy** | **Cluster nhiều máy** |
| Lo việc | Build image, chạy container | Scale, self-heal, load balance, rollout/rollback |
| Auto-scale / tự hồi phục | Không (tự thân) | **Có** |
| Quan hệ | Tạo *đơn vị* để chạy | *Vận hành* các đơn vị đó |

**Điểm hay nhầm:** "Docker vs Kubernetes" không đối lập kiểu "chọn 1". Thực tế: **dùng Docker để build image**,
rồi **K8s để vận hành** các container đó ở production. Với **1 máy, ít container** thì **Docker Compose** là đủ;
khi cần **nhiều máy, HA, auto-scale** mới cần **Kubernetes**.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Không đối lập, mà bổ sung ở hai tầng:** **Docker = đóng gói + chạy container (1 máy)**; **Kubernetes =
  điều phối nhiều container trên nhiều máy (cluster)**.
- K8s lo cái Docker đơn lẻ **không lo**: **auto-scale, self-heal, load balance, rollout/rollback** — và **không
  tự chạy container** mà gọi runtime (containerd).
- Ẩn dụ: **Docker tạo container, Kubernetes là "thuyền trưởng" điều cả hạm đội.**
- Ít container/1 máy → **Docker Compose**; nhiều máy/HA/auto-scale → **Kubernetes**.

[↑ Mục lục](#mục-lục)

<a id="q5"></a>
## 5. CI/CD là gì?

### 📖 Giới thiệu kiến thức

**CI/CD** là tập hợp **thực hành (practice)** giúp **tự động hóa việc build, test và đưa phần mềm tới người
dùng**, biến quy trình từ *"code xong build/deploy thủ công, dễ sai"* thành một **pipeline (dây chuyền) tự
động** chạy mỗi khi có thay đổi code. CI/CD là xương sống của văn hóa **DevOps**, giúp **giao phần mềm nhanh,
thường xuyên và đáng tin cậy**.

Viết tắt gồm hai (thực ra ba) phần:

- **CI — Continuous Integration (Tích hợp liên tục):** lập trình viên **merge code thường xuyên** vào nhánh
  chung; **mỗi lần push** kích hoạt pipeline **tự động build + chạy test**. Mục tiêu: **phát hiện lỗi tích hợp
  sớm**, tránh cảnh dồn code hàng tuần rồi "merge hell".
- **CD — Continuous Delivery (Chuyển giao liên tục):** mọi thay đổi qua được CI sẽ được **tự động đóng gói và
  đưa tới trạng thái SẴN SÀNG deploy** (lên staging / registry). Việc **lên production vẫn cần một cú bấm nút
  (phê duyệt thủ công)**.
- **CD — Continuous Deployment (Triển khai liên tục):** đi xa hơn — mọi thay đổi qua hết pipeline được **tự động
  deploy thẳng lên production**, **không cần bước duyệt tay**.

**Một pipeline CI/CD điển hình (các stage):**

```
[Push code]                         (CI)                         (CD)
   Git  →  Build  →  Test  →  Security scan  →  Deliver/Deploy  →  Monitor
          (compile,   (unit,     (quét lỗ hổng)   (push image lên
           build image) integration)               registry, deploy
                                                    lên K8s/server)
```

- **Source:** trigger khi `git push` / mở Pull Request.
- **Build:** compile code, **build Docker image** (mục 3.3).
- **Test:** chạy **unit / integration / e2e test**; fail thì dừng pipeline, báo dev.
- **(Scan):** quét bảo mật (SAST, quét lỗ hổng image), kiểm tra chất lượng code.
- **Deliver / Deploy:** **push image lên registry** rồi **deploy** (lên K8s, server, hoặc cloud) — tự động
  (Continuous Deployment) hoặc chờ duyệt (Continuous Delivery).
- **Monitor:** theo dõi sau khi deploy; lỗi thì **rollback**.

**Lợi ích:**

- **Giao hàng nhanh & thường xuyên** (release nhiều lần/ngày thay vì vài tuần/lần).
- **Ít bug lọt production** nhờ test tự động ở mọi thay đổi.
- **Nhất quán & lặp lại được:** cùng một pipeline cho mọi lần deploy → giảm lỗi do làm tay.
- **Phản hồi nhanh** cho dev; **rollback nhanh** khi có sự cố.

**Công cụ phổ biến:** **GitHub Actions**, **GitLab CI/CD**, **Jenkins**, CircleCI, Travis CI; phía CD cho
Kubernetes có **Argo CD**, Flux (GitOps).

**Ví dụ pipeline đơn giản (GitHub Actions) — build & test khi push:**

```yaml
name: CI
on: [push]                        # trigger mỗi lần push
jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4        # lấy code
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci                       # cài dependency
      - run: npm test                     # chạy test — fail thì dừng pipeline
      - run: docker build -t my-app:${{ github.sha }} .   # build image
```

> Liên hệ: CI/CD thường **build Docker image** (mục 3) rồi **deploy lên Kubernetes** (câu 4) — bộ ba
> Docker + K8s + CI/CD là nền tảng triển khai hiện đại. Với K8s còn có mô hình **GitOps** (Argo CD): trạng thái
> mong muốn khai báo trong Git, công cụ CD **tự đồng bộ** cluster cho khớp.

### 🎯 Tóm tắt trả lời phỏng vấn

- **CI/CD** = thực hành **tự động hóa build → test → giao/triển khai** phần mềm bằng một **pipeline** chạy mỗi
  khi có thay đổi code; là xương sống của **DevOps**.
- **CI (Continuous Integration):** merge code thường xuyên, **mỗi push tự động build + test** → bắt lỗi tích hợp sớm.
- **CD:** **Continuous Delivery** = tự động đưa tới trạng thái **sẵn sàng deploy**, lên prod cần **bấm nút duyệt**;
  **Continuous Deployment** = **tự động deploy thẳng prod**, không cần duyệt tay.
- **Các stage:** source → build (build image) → test → scan → deploy → monitor.
- **Lợi ích:** giao hàng nhanh/thường xuyên, ít bug, nhất quán, rollback nhanh.
- **Công cụ:** GitHub Actions, GitLab CI, Jenkins; CD cho K8s: Argo CD (GitOps). Thường **build Docker image →
  deploy lên K8s**.

[↑ Mục lục](#mục-lục)

<a id="q5-1"></a>
### 5.1. Phân biệt Continuous Integration / Delivery / Deployment

#### 📖 Giới thiệu kiến thức

Ba khái niệm là **các bậc kế tiếp nhau** trên cùng một hành trình tự động hóa — mỗi bậc **tự động hóa thêm một
đoạn** so với bậc trước:

| Khái niệm | Tự động hóa tới đâu | Lên production |
|-----------|---------------------|----------------|
| **Continuous Integration** | Merge code → **build + test** tự động | *Chưa đụng tới deploy* |
| **Continuous Delivery** | Thêm: tự động **đóng gói, đưa lên staging / sẵn sàng release** | **Thủ công** — bấm nút duyệt |
| **Continuous Deployment** | Thêm: **tự động deploy thẳng production** | **Tự động** — không cần người |

```
CI            :  push → build → test
Continuous Delivery   :  push → build → test → (staging) → [DUYỆT TAY] → prod
Continuous Deployment :  push → build → test → (staging) → prod (tự động)
```

Điểm mấu chốt phân biệt hai chữ "CD":

- **Continuous Delivery** luôn giữ code **ở trạng thái deploy-được bất cứ lúc nào**, nhưng **con người quyết định
  khi nào bấm nút** lên prod (hợp môi trường cần kiểm soát/phê duyệt).
- **Continuous Deployment** **bỏ luôn cái nút đó** — qua hết test là ra prod. Đòi hỏi **bộ test rất mạnh & giám
  sát tốt** vì không còn "chốt chặn" con người.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **CI:** tự động **build + test** mỗi lần push (chưa deploy).
- **Continuous Delivery:** tự động đưa tới trạng thái **sẵn sàng release**, lên prod **cần bấm nút duyệt tay**.
- **Continuous Deployment:** tự động **deploy thẳng prod**, **không cần duyệt** — cần test & monitoring cực mạnh.
- Mẹo nhớ: cả ba là **bậc thang nối tiếp**, mỗi bậc tự động thêm một đoạn; khác biệt lớn nhất giữa hai "CD" là
  **có hay không cú bấm nút thủ công trước production**.

[↑ Mục lục](#mục-lục)

<a id="q5-2"></a>
### 5.2. GitHub Actions / GitLab CI: workflow file, job, step, trigger

#### 📖 Giới thiệu kiến thức

**GitHub Actions** và **GitLab CI/CD** là hai công cụ CI/CD phổ biến, đều hoạt động theo cùng một ý tưởng: ta
đặt **một file YAML mô tả pipeline** vào repo, mỗi khi có **sự kiện (trigger)** thì công cụ đọc file đó và chạy
các bước tự động trên một **runner** (máy chạy job). Nắm 4 khái niệm cốt lõi là hiểu được cả hai:

**1) Workflow file (file mô tả pipeline)** — nơi khai báo toàn bộ pipeline bằng YAML:

- **GitHub Actions:** đặt trong `.github/workflows/*.yml` (được **nhiều file**, mỗi file một workflow).
- **GitLab CI:** mặc định **một file** `.gitlab-ci.yml` ở gốc repo.

**2) Trigger (sự kiện kích hoạt)** — quyết định **khi nào** pipeline chạy:

- Ví dụ: **push**, **pull/merge request**, tạo **tag**, theo **lịch (cron)**, hoặc **chạy tay (manual)**.
- **GitHub Actions:** khai báo ở khóa **`on:`** (`on: [push]`, hoặc `on: { pull_request: { branches: [main] } }`).
- **GitLab CI:** dùng **`rules:`** (hoặc `only/except` kiểu cũ) ở từng job để lọc theo nhánh/sự kiện; sự kiện có
  sẵn qua các biến như `$CI_PIPELINE_SOURCE`.

**3) Job (công việc)** — một **nhóm bước** chạy trên **một runner**; các job có thể **chạy song song** hoặc
**phụ thuộc nhau**:

- **GitHub Actions:** khóa **`jobs:`**, mỗi job có **`runs-on:`** (chọn runner) và có thể **`needs:`** để đợi job
  khác xong.
- **GitLab CI:** mỗi mục cấp cao (không phải từ khóa) là **một job**; job thuộc một **`stage:`**; các stage chạy
  **tuần tự**, job **cùng stage** chạy **song song**; **`needs:`** cho phép chạy chéo stage.

**4) Step (bước)** — **hành động nhỏ nhất** trong một job (chạy lệnh hoặc gọi action dựng sẵn):

- **GitHub Actions:** khóa **`steps:`**; mỗi step là **`run:`** (chạy lệnh shell) hoặc **`uses:`** (dùng lại một
  **Action** có sẵn, vd `actions/checkout@v4`).
- **GitLab CI:** không có khái niệm "step" riêng — mỗi job có khối **`script:`** liệt kê các lệnh chạy tuần tự.

**Bảng ánh xạ thuật ngữ:**

| Khái niệm | GitHub Actions | GitLab CI |
|-----------|----------------|-----------|
| File | `.github/workflows/*.yml` (nhiều) | `.gitlab-ci.yml` (một) |
| Cả pipeline | **Workflow** | **Pipeline** |
| Trigger | `on:` | `rules:` / `only` / `workflow` |
| Nhóm bước | **Job** (`jobs:`, `runs-on`) | **Job** (thuộc `stage:`) |
| Bước nhỏ | **Step** (`run` / `uses`) | dòng trong `script:` |
| Giai đoạn tuần tự | (dùng `needs:`) | **Stage** (`stages:`) |
| Máy chạy | **Runner** (`runs-on`) | **Runner** (`tags:`) |

**Ví dụ GitHub Actions** (`.github/workflows/ci.yml`):

```yaml
name: CI
on:                                  # TRIGGER
  push:
    branches: [main]
  pull_request:

jobs:                                # các JOB
  test:                              # job 'test'
    runs-on: ubuntu-latest           # RUNNER
    steps:                           # các STEP
      - uses: actions/checkout@v4    # step gọi Action dựng sẵn
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci                  # step chạy lệnh
      - run: npm test

  build:                             # job 'build'
    needs: test                      # đợi job 'test' pass mới chạy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t my-app:${{ github.sha }} .
```

**Ví dụ GitLab CI** (`.gitlab-ci.yml`) tương đương:

```yaml
stages: [test, build]                # các STAGE chạy tuần tự

test:                                # JOB thuộc stage 'test'
  stage: test
  image: node:20                     # image runner dùng
  script:                            # các lệnh (tương đương step)
    - npm ci
    - npm test
  rules:                             # TRIGGER: chỉ chạy trên main / merge request
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"

build:                               # JOB thuộc stage 'build'
  stage: build                       # stage sau → chạy sau khi 'test' xong
  script:
    - docker build -t my-app:$CI_COMMIT_SHA .
```

> Mấu chốt để nhớ: **Trigger** (khi nào chạy) → **Workflow/Pipeline** (cả dây chuyền) gồm nhiều **Job** (chạy
> trên runner, song song hoặc `needs` nhau) → mỗi job gồm nhiều **Step/`script`** (lệnh cụ thể). GitLab thêm lớp
> **Stage** để nhóm job theo thứ tự tuần tự.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Workflow file:** file YAML mô tả pipeline trong repo — GitHub Actions: `.github/workflows/*.yml` (nhiều
  file); GitLab CI: `.gitlab-ci.yml` (một file).
- **Trigger:** sự kiện kích hoạt (push, pull/merge request, tag, cron, manual) — GHA khai báo ở **`on:`**, GitLab
  ở **`rules:`**.
- **Job:** nhóm bước chạy trên **một runner**; có thể **song song** hoặc phụ thuộc bằng **`needs:`**. GHA: `jobs:`
  + `runs-on`; GitLab: mỗi job thuộc một **`stage:`** (stage tuần tự, job cùng stage chạy song song).
- **Step:** đơn vị nhỏ nhất trong job — GHA dùng **`run:`** (chạy lệnh) hoặc **`uses:`** (Action dựng sẵn); GitLab
  gộp trong **`script:`**.
- **Mạch:** Trigger → Workflow/Pipeline → Job (runner) → Step/`script`; GitLab có thêm lớp **Stage**.

[↑ Mục lục](#mục-lục)

<a id="q5-3"></a>
### 5.3. Triển khai CI/CD cho NestJS cần những gì?

#### 📖 Giới thiệu kiến thức

Câu này gộp mọi thứ đã học ở trên thành một **pipeline hoàn chỉnh cho một app NestJS**. Cần suy nghĩ theo **hai
pha CI → CD** và các mảnh ghép sau.

**1) Chuẩn bị (điều kiện cần):**

- **Repo + trigger:** code trên Git; trigger pipeline khi **push / mở PR** (chạy CI) và khi **merge vào main /
  tạo tag** (chạy CD) — xem `on:` / `rules:` ở mục 5.2.
- **Dockerfile (multi-stage)** cho NestJS: stage build compile **TypeScript → `dist/`**, stage runtime chỉ giữ
  **production deps + `dist/`** (xem mục 3.5). Chạy bằng `node dist/main.js`.
- **Registry** để lưu image (Docker Hub, **GHCR**, GitLab Registry, AWS ECR).
- **Secrets** khai báo trong CI (không commit): thông tin đăng nhập registry, biến môi trường app (DB URL, JWT
  secret...), khóa SSH/kubeconfig để deploy.

**2) Pha CI (chạy mỗi push/PR) — đảm bảo code đúng:**

- **Cài dependency:** `npm ci` (cài đúng theo `package-lock.json`, sạch & lặp lại được).
- **Lint & format check:** `npm run lint` (ESLint) để giữ chất lượng.
- **Build:** `npm run build` — build được thì mới chắc TS không lỗi biên dịch.
- **Test:** `npm run test` (unit — Jest) và `npm run test:e2e` (e2e). Test thường **cần DB/Redis** → khai báo
  **service** trong CI (Postgres/MySQL/Redis) để test chạy được.
- **(Tùy chọn):** coverage, quét bảo mật (`npm audit`, Snyk).

**3) Pha CD (sau khi CI pass, thường trên nhánh main/tag) — đóng gói & deploy:**

- **Build Docker image** và **tag** theo `commit SHA` hoặc version (`my-app:1.2.0`, `my-app:<sha>`).
- **Push image lên registry.**
- **Chạy migration** CSDL (TypeORM `migration:run` / Prisma `migrate deploy`) — bước **rất quan trọng và hay bị
  quên**; thường chạy **trước khi** cho traffic vào bản mới.
- **Deploy** lên đích: SSH vào server chạy `docker compose pull && up -d` (nhỏ), hoặc `kubectl set image` /
  Helm / **Argo CD** (K8s — câu 4), hoặc PaaS (Render/Railway/Cloud Run).
- **Health check + rollback:** kiểm tra endpoint health sau deploy; lỗi thì **quay lui** bản trước.

**Ví dụ pipeline GitHub Actions (CI test + build & push image):**

```yaml
name: CI-CD
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:                          # DB tạm cho e2e test
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: test, POSTGRES_DB: app_test }
        ports: ["5432:5432"]
    env:
      DATABASE_URL: postgres://postgres:test@localhost:5432/app_test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint
      - run: npm run build
      - run: npm run test
      - run: npm run test:e2e

  build-deploy:
    needs: test                        # chỉ chạy khi test pass
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Login registry
        run: echo "${{ secrets.REGISTRY_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Build & push
        run: |
          docker build -t ghcr.io/myorg/nest-app:${{ github.sha }} .
          docker push ghcr.io/myorg/nest-app:${{ github.sha }}
      # - name: Deploy  → kubectl set image / helm upgrade / ssh docker compose up
```

**Những điểm NestJS hay bị hỏi / dễ sai:**

- **Migration khi deploy:** phải có bước chạy migration; đừng dựa vào `synchronize: true` của TypeORM ở
  production (nguy hiểm, dễ mất dữ liệu).
- **Config theo môi trường:** dùng `@nestjs/config` (ConfigModule) + biến môi trường; secret lấy từ CI/secret
  manager, **không** hard-code hay commit `.env`.
- **`npm ci` chứ không `npm install`** trong CI để build lặp lại được.
- **Test cần hạ tầng:** e2e cần DB/Redis → dùng `services:` (GHA) hoặc `services:` trong GitLab CI.
- **Image gọn:** multi-stage + chỉ production deps (mục 3.4, 3.5) để deploy nhanh.
- **Zero-downtime:** deploy kiểu rolling (K8s Deployment tự làm — câu 4.1) để không rớt request khi lên bản mới.

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Cần chuẩn bị:** repo + **trigger** (push/PR → CI; main/tag → CD), **Dockerfile multi-stage** (build TS →
  `dist`, runtime gọn), **registry**, và **secrets** trong CI (registry, DB, JWT, kubeconfig).
- **Pha CI:** `npm ci` → **lint** → **build** (`npm run build`) → **test** unit + e2e (e2e cần khai báo
  **service** DB/Redis trong CI).
- **Pha CD:** **build & tag image** (theo commit SHA) → **push lên registry** → **chạy migration** (TypeORM/
  Prisma) → **deploy** (docker compose / `kubectl`/Helm/Argo CD / PaaS) → **health check + rollback**.
- **Điểm hay sai:** quên **migration**; dùng `synchronize:true` ở prod; hard-code secret; dùng `npm install` thay
  `npm ci`; e2e thiếu DB service; image không multi-stage.
- Liên hệ: dùng **Dockerfile multi-stage** (3.5), có thể deploy lên **Kubernetes** (câu 4) với rolling update.

[↑ Mục lục](#mục-lục)

<a id="q5-4"></a>
### 5.4. Deploy CI/CD cho Laravel cần những gì?

#### 📖 Giới thiệu kiến thức

Giống câu NestJS (5.3) nhưng có **những bước đặc thù của Laravel/PHP**. Vẫn chia **CI → CD**, nhưng chú ý các
điểm riêng: **`APP_KEY`**, **migration `--force`**, các lệnh **cache tối ưu**, **`queue:restart`**, và quyền
thư mục `storage`.

**1) Chuẩn bị (điều kiện cần):**

- **Repo + trigger:** push/PR → CI; merge main / tag → CD (xem `on:` / `rules:` mục 5.2).
- **Cách đóng gói:** hoặc **Docker** (php-fpm + Nginx, multi-stage — xem mục 3.5, 3.9), hoặc **deploy truyền
  thống** không container (SSH + Deployer/Envoyer/Forge). Laravel có cả hệ sinh thái riêng: **Envoyer** (deploy
  zero-downtime), **Laravel Forge** (quản server), **Vapor** (serverless trên AWS Lambda — xem câu 6).
- **Registry** (nếu dùng Docker) + **Secrets** trong CI: `APP_KEY`, DB credentials, thông tin registry, khóa SSH.

**2) Pha CI (mỗi push/PR):**

- **Cài dependency:** `composer install --prefer-dist --no-interaction` (bản test cần cả dev-dependency để chạy
  PHPUnit).
- **Chuẩn bị env:** `cp .env.example .env` rồi **`php artisan key:generate`** (Laravel bắt buộc có `APP_KEY`, nếu
  không sẽ lỗi mã hóa/session).
- **Static analysis & lint:** **Laravel Pint** (`./vendor/bin/pint --test`) và **PHPStan/Larastan**.
- **Test:** **`php artisan test`** (hoặc `phpunit`). Test cần **DB + Redis** → khai báo **service** trong CI
  (MySQL/PostgreSQL/Redis); hoặc dùng **SQLite in-memory** cho nhanh.

**3) Pha CD (sau khi CI pass, trên main/tag):**

- **Build & push image** (nếu Docker): tag theo `commit SHA`/version, đẩy lên registry.
- **Chạy migration:** **`php artisan migrate --force`** — cờ **`--force` bắt buộc ở production** (vì chạy
  non-interactive, không có bước hỏi xác nhận).
- **Tối ưu cache (rất quan trọng cho hiệu năng prod):**
  ```
  php artisan config:cache
  php artisan route:cache
  php artisan view:cache
  php artisan event:cache
  ```
  (gộp lại: `php artisan optimize`). **Lưu ý:** sau khi `config:cache`, hàm `env()` **chỉ đọc được trong file
  config** — code dùng `env()` trực tiếp ngoài config sẽ nhận `null`.
- **`php artisan queue:restart`:** báo các **queue worker** nạp lại code mới (worker là tiến trình chạy dài, nếu
  không restart sẽ **chạy code cũ**).
- **`php artisan storage:link`** (nếu cần symlink storage) và **phân quyền** `storage/`, `bootstrap/cache` cho
  user web (`www-data`).
- **Deploy:** SSH `docker compose pull && up -d` / `kubectl set image` (K8s — câu 4) / Envoyer / Forge.
- **Zero-downtime + rollback:** Envoyer/Deployer dùng **symlink nguyên tử** để đổi bản không downtime; K8s dùng
  rolling update. Health check xong, lỗi thì rollback.

**Ví dụ pipeline GitHub Actions (CI test + build image):**

```yaml
name: CI-CD
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:                          # DB tạm cho test
      mysql:
        image: mysql:8
        env: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: app_test }
        ports: ["3306:3306"]
    env:
      DB_CONNECTION: mysql
      DB_HOST: 127.0.0.1
      DB_DATABASE: app_test
      DB_USERNAME: root
      DB_PASSWORD: root
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2      # cài PHP + extension
        with: { php-version: '8.3' }
      - run: composer install --prefer-dist --no-interaction
      - run: cp .env.example .env && php artisan key:generate   # APP_KEY bắt buộc
      - run: ./vendor/bin/pint --test        # lint
      - run: php artisan migrate --force      # migrate DB test
      - run: php artisan test                 # chạy test

  build-deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          docker build -t ghcr.io/myorg/laravel-app:${{ github.sha }} .
          echo "${{ secrets.REGISTRY_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker push ghcr.io/myorg/laravel-app:${{ github.sha }}
      # - name: Deploy → ssh docker compose up / kubectl / Envoyer
      #   sau deploy nhớ: php artisan migrate --force && php artisan optimize && php artisan queue:restart
```

**So với NestJS — điểm khác cốt lõi:**

| Bước | NestJS | Laravel |
|------|--------|---------|
| Cài deps | `npm ci` | `composer install` |
| Build | `npm run build` (TS→`dist`) | *Không compile*; nhưng cần `artisan optimize` cache |
| Env | biến môi trường / ConfigModule | **`APP_KEY` bắt buộc** + `.env` |
| Migration | TypeORM/Prisma `migrate` | **`php artisan migrate --force`** |
| Sau deploy | restart process (PM2/K8s) | **`queue:restart`** + cache config/route/view |

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Chuẩn bị:** repo + **trigger**; đóng gói bằng **Docker (php-fpm + Nginx, multi-stage)** hoặc deploy truyền
  thống (**Envoyer/Forge/Vapor**); **secrets** (`APP_KEY`, DB, registry).
- **Pha CI:** `composer install` → **`key:generate`** (APP_KEY bắt buộc) → **Pint/PHPStan** → **`php artisan
  test`** (cần **service** DB/Redis hoặc SQLite in-memory).
- **Pha CD:** build & push image → **`php artisan migrate --force`** → **cache** (`config/route/view:cache` hay
  `optimize`) → **`queue:restart`** → phân quyền `storage` → deploy (compose/K8s/Envoyer) → health check + rollback.
- **Điểm Laravel hay sai:** quên **`--force`** khi migrate prod; quên **`queue:restart`** (worker chạy code cũ);
  dùng **`env()` ngoài config** sau khi `config:cache` (nhận `null`); thiếu **`APP_KEY`**; sai quyền `storage/`.
- Khác NestJS: Laravel **không compile** nhưng cần **cache artisan**; luôn có **`APP_KEY`** + **`migrate --force`**.

[↑ Mục lục](#mục-lục)

<a id="q6"></a>
## 6. Kể tên một số dịch vụ của AWS và Google Cloud

### 📖 Giới thiệu kiến thức

**AWS (Amazon Web Services)** và **Google Cloud Platform (GCP)** là hai nhà cung cấp **cloud computing** lớn (cùng với Azure). Ý tưởng chung: thay vì tự mua/vận hành server vật lý, ta **thuê hạ tầng theo nhu cầu** (pay-as-you-go) và nhà cung cấp lo phần vận hành, mở rộng, bảo mật ở lớp dưới.

Các dịch vụ được nhóm theo **chức năng**. Nắm theo nhóm sẽ dễ nhớ hơn là học thuộc tên rời rạc — và quan trọng: **AWS và GCP gần như tương đương từng cặp**, chỉ khác tên. Nhiều dịch vụ ở đây chính là hiện thân cụ thể của các khái niệm đã học: Serverless = Lambda / Cloud Functions ([q7](#q7)), Kubernetes managed = EKS / GKE ([q4](#q4)), Load Balancer/CDN = ELB / Cloud Load Balancing ([q1](#q1)).

**Bảng ánh xạ theo nhóm (AWS ↔ GCP):**

| Nhóm | AWS | Google Cloud (GCP) | Dùng để làm gì |
|------|-----|--------------------|----------------|
| **Compute** (máy chủ ảo) | **EC2** | **Compute Engine** | Chạy VM, tự quản OS |
| Compute — PaaS | Elastic Beanstalk | App Engine | Deploy app không lo hạ tầng |
| Compute — Container | ECS / **EKS**, Fargate | Cloud Run, **GKE** | Chạy container / Kubernetes managed |
| **Serverless** (hàm) | **Lambda** | **Cloud Functions** | Chạy code theo sự kiện, không giữ server |
| **Storage** (object) | **S3** | **Cloud Storage** | Lưu file/ảnh/backup, tĩnh, rẻ |
| Storage — block/disk | EBS | Persistent Disk | Ổ đĩa gắn vào VM |
| **Database** — SQL | **RDS**, Aurora | **Cloud SQL**, Spanner | MySQL/PostgreSQL managed |
| Database — NoSQL | DynamoDB | Firestore, Bigtable | Key-value / document, scale lớn |
| Database — Cache | ElastiCache (Redis) | Memorystore (Redis) | Cache in-memory |
| Database — Data warehouse | Redshift | **BigQuery** | Phân tích dữ liệu lớn (OLAP) |
| **Networking** — DNS | Route 53 | Cloud DNS | Quản lý tên miền |
| Networking — CDN | **CloudFront** | Cloud CDN | Phân phối nội dung gần user |
| Networking — Load Balancer | ELB / ALB | Cloud Load Balancing | Cân tải traffic |
| Networking — mạng riêng | VPC | VPC | Mạng ảo cô lập |
| **Messaging / Queue** | **SQS**, SNS, EventBridge | **Pub/Sub** | Hàng đợi, message giữa service |
| **CI/CD & IaC** | CodePipeline, CloudFormation | Cloud Build, Deployment Manager | Pipeline, hạ tầng dạng code |
| **Monitoring** | CloudWatch | Cloud Monitoring / Logging | Log, metric, cảnh báo |
| **Security / Identity** | IAM, Cognito, KMS | Cloud IAM, Identity Platform, Cloud KMS | Phân quyền, auth user, mã hoá |
| **AI / ML** | SageMaker, Bedrock | Vertex AI | Train/deploy mô hình ML |

> Ghi nhớ nhanh 3 dịch vụ "quốc dân" hay bị hỏi nhất của AWS: **EC2** (server), **S3** (lưu file), **RDS** (database) — cộng **Lambda** (serverless). Bên GCP tương ứng: **Compute Engine, Cloud Storage, Cloud SQL, Cloud Functions**.

### 🎯 Tóm tắt trả lời phỏng vấn
- AWS và GCP là 2 nền tảng **cloud** lớn, dịch vụ chia theo nhóm và gần như **tương đương từng cặp**, chỉ khác tên.
- **Compute:** EC2 ↔ Compute Engine; container EKS ↔ GKE; serverless **Lambda ↔ Cloud Functions**.
- **Storage:** **S3 ↔ Cloud Storage** (object storage).
- **Database:** RDS ↔ Cloud SQL (SQL); DynamoDB ↔ Firestore (NoSQL); Redshift ↔ **BigQuery** (data warehouse); cache ElastiCache ↔ Memorystore.
- **Khác:** CDN CloudFront ↔ Cloud CDN; queue **SQS/SNS ↔ Pub/Sub**; monitoring CloudWatch ↔ Cloud Monitoring; quyền IAM (cả hai đều gọi IAM).
- Nhớ tối thiểu: **EC2, S3, RDS, Lambda** (AWS) là bộ hay hỏi nhất.

[↑ Mục lục](#mục-lục)

<a id="q7"></a>
## 7. Serverless là gì?

### 📖 Giới thiệu kiến thức

**Serverless** là mô hình điện toán đám mây (cloud) trong đó **nhà cung cấp cloud lo toàn bộ việc quản lý server**
— cấp phát (provisioning), scale, vá lỗi, bảo trì hạ tầng — còn **lập trình viên chỉ cần viết code**. Tên gọi
dễ gây hiểu lầm: **"serverless" KHÔNG phải "không có server"** — server vẫn tồn tại, chỉ là **bạn không phải
nhìn thấy hay quản lý** chúng. Đây có thể xem là bước tiếp theo của câu chuyện scaling (câu 2): thay vì tự cấu
hình auto-scaling, **cloud tự scale hộ bạn**.

**Hai dạng chính:**

- **FaaS (Function as a Service):** viết các **hàm nhỏ**, mỗi hàm chạy khi có **sự kiện** kích hoạt (HTTP
  request, upload file, message vào queue, cron...). Cloud tự chạy và tự scale số instance. Ví dụ: **AWS
  Lambda**, Google Cloud Functions, Azure Functions, Cloudflare Workers.
- **BaaS (Backend as a Service):** dùng các **dịch vụ backend quản lý sẵn** thay vì tự viết/vận hành — auth,
  database, storage, realtime. Ví dụ: **Firebase**, Supabase, AWS (DynamoDB, S3, Cognito).

**Đặc điểm cốt lõi:**

- **Event-driven:** code chỉ chạy khi có sự kiện, không chạy thường trực.
- **Auto-scaling & scale-to-zero:** tự scale từ **0** lên hàng nghìn instance theo tải; **không có request thì
  không chạy** (và không tính tiền).
- **Pay-per-use:** chỉ trả tiền theo **số lần gọi + thời gian thực thi** (tính bằng ms) + bộ nhớ dùng — **không**
  trả cho server nằm chờ (idle) như VM truyền thống.
- **Stateless & ephemeral:** mỗi lần gọi độc lập, **không giữ trạng thái**; function chạy xong bị hủy → phải đẩy
  state ra ngoài (DB, cache, S3).

**So với các mô hình khác:**

| Mô hình | Ai quản server | Scaling | Trả tiền | Ví dụ |
|---------|----------------|---------|----------|-------|
| **VM / máy chủ** (IaaS) | **Bạn** (OS, scaling) | Thủ công / auto-scaling tự cấu hình | Theo giờ máy chạy (kể cả idle) | EC2, VPS |
| **Container** | Bạn quản orchestration | Bạn cấu hình (K8s HPA) | Theo tài nguyên cấp phát | Docker + K8s |
| **Serverless** | **Cloud lo hết** | **Tự động, scale-to-zero** | **Theo lần chạy (ms)** | Lambda, Cloud Functions |

**Ưu điểm:**

- **No-ops:** không phải quản OS/hạ tầng → tập trung vào business logic.
- **Tự scale**, chịu tải đột biến tốt mà không cần cấu hình.
- **Tiết kiệm chi phí** cho tải **biến động / thấp** (không trả tiền lúc rảnh).
- Deploy nhanh, hợp kiến trúc microservice / event-driven.

**Nhược điểm:**

- **Cold start:** lần gọi đầu (hoặc sau thời gian rảnh) phải khởi tạo môi trường → **độ trễ** tăng.
- **Giới hạn thời gian chạy** (AWS Lambda tối đa **15 phút**) → **không hợp task chạy dài**.
- **Vendor lock-in:** phụ thuộc nền tảng cloud cụ thể, khó chuyển.
- **Khó debug/monitor**; môi trường local khác production.
- **Bùng nổ kết nối DB:** rất nhiều instance chạy song song → mỗi cái mở connection riêng → dễ **vượt
  `max_connections`** → cần dùng **connection pooler** (RDS Proxy, PgBouncer) — xem `Backend/database-system.md`
  mục 2.4.

**Khi nào dùng / không dùng:**

- **Hợp:** API backend tải **biến động**, xử lý sự kiện (resize ảnh khi upload S3), **cron job**, webhook, xử
  lý stream/message, chatbot.
- **Không hợp:** task **chạy dài**, ứng dụng **latency-critical** (cold start hại), hoặc **tải cao & đều liên
  tục** (khi đó VM/container thường rẻ và ổn định hơn).

### 🎯 Tóm tắt trả lời phỏng vấn

- **Serverless** = mô hình cloud nơi **provider quản toàn bộ server**, dev **chỉ viết code**. "Serverless" nghĩa
  là **không phải quản server**, chứ **không phải không có server**.
- **Hai dạng:** **FaaS** (AWS Lambda, Cloud Functions — chạy hàm theo **sự kiện**) và **BaaS** (Firebase,
  Supabase — dùng dịch vụ backend quản lý sẵn: auth/DB/storage).
- **Đặc điểm:** event-driven, **auto-scale + scale-to-zero**, **pay-per-use** (trả theo ms chạy), **stateless &
  ephemeral**.
- **Ưu:** no-ops, tự scale, tiết kiệm cho tải biến động, deploy nhanh. **Nhược:** **cold start**, **giới hạn thời
  gian chạy** (Lambda 15'), **vendor lock-in**, khó debug, **bùng nổ connection DB** (cần pooler — xem
  `database-system.md` mục 2.4).
- **Dùng khi:** API biến động, xử lý sự kiện, cron, webhook. **Tránh:** task dài, latency-critical, tải cao
  liên tục (dùng VM/container).

[↑ Mục lục](#mục-lục)

<a id="q8"></a>
## 8. Cron job và queue job ở backend chạy nền ở đâu?

### 📖 Giới thiệu kiến thức

Điểm mấu chốt (hay bị hỏi để kiểm tra hiểu bản chất): **background job KHÔNG chạy trong tiến trình web
server**, tức **không nằm trong vòng đời request HTTP**. Chúng chạy trong **tiến trình riêng (worker
process)** — thường là **container/pod/dyno riêng** dùng chung codebase nhưng **lệnh khởi động khác**. Web
process trả response ngay, việc nặng để tiến trình khác lo.

#### Queue job chạy ở đâu?

1. **Web process** chỉ **đẩy job vào hàng đợi** rồi trả về ngay (không thực thi).
2. Job nằm ở **queue backend / broker** — nơi lưu trữ: **Redis**, **database**, **RabbitMQ**, **AWS SQS**,
   **Beanstalkd**...
3. Một **worker process (daemon) chạy trường tồn (long-running)** liên tục lắng nghe queue, **lấy job ra
   và thực thi**:
   - **Laravel:** `php artisan queue:work` (hoặc **Horizon** cho Redis).
   - **Node/NestJS:** **BullMQ `Worker`** (tiến trình worker riêng).
   - Hệ khác: **Sidekiq** (Ruby), **Celery** (Python).
4. **Scale** = chạy **nhiều worker song song** (nhiều container/replica) khi backlog lớn.

#### Cron job chạy ở đâu? (cần bộ lập lịch kích hoạt theo giờ)

- **System cron (crontab)** của OS/Linux — dòng `* * * * * command`.
- **Framework scheduler:**
  - **Laravel Scheduler:** chỉ cần **1 dòng crontab** gọi `php artisan schedule:run` **mỗi phút**, rồi
    **framework tự quyết** task nào tới hạn (định nghĩa trong code) → dễ quản, versioned theo code.
  - **NestJS:** `@nestjs/schedule` với decorator `@Cron()` — chạy **in-process** (ngay trong app Node).
    Lưu ý: nếu chạy **nhiều instance** thì cron in-process sẽ **kích hoạt trùng** ở mỗi instance (xem lock
    chống overlap ở `Common/resolve-problem.md`).
- **Cloud / hạ tầng:** **Kubernetes `CronJob`** (tạo Pod theo lịch), **AWS EventBridge (CloudWatch Events)
  + Lambda**, **GitHub Actions** schedule.

#### Giữ worker sống — process manager

Worker/daemon phải được **tự khởi động lại khi chết**: **Supervisor**, **systemd**, **pm2** (Node), hoặc
**Laravel Horizon** (quản + giám sát queue worker Redis).

#### Triển khai (deploy) ở đâu?

Thường là **tiến trình/service riêng** cùng image, khác **command**:
- **Docker Compose:** thêm **service `worker`** dùng lại image app, `command: php artisan queue:work` (đã
  thấy ở [câu 3.9](#q3-9)).
- **Kubernetes:** **Deployment riêng** cho queue worker + **CronJob** cho cron ([câu 4](#q4)).
- **PaaS (Heroku):** **worker dyno** riêng, tách khỏi **web dyno**.

**Vì sao phải tách khỏi web process:** (1) **không block request** (trả response nhanh); (2) **scale độc
lập** (thêm worker khi hàng đợi dồn mà không đụng web); (3) **cô lập tài nguyên & lỗi**; (4) **deploy/restart
riêng**.

```
   [Web process] --push job-->  [Broker: Redis/DB/RabbitMQ/SQS]  <--poll--  [Worker process(es)]  --> thực thi
   [System cron / K8s CronJob] --mỗi phút--> [scheduler (schedule:run)] --> chạy task tới hạn
   (worker & scheduler = container/pod/dyno RIÊNG, được Supervisor/systemd/Horizon giữ sống)
```

### 🎯 Tóm tắt trả lời phỏng vấn

- **Không chạy trong web server** — background job chạy trong **tiến trình worker riêng**, thường là
  **container/pod/dyno riêng** cùng code nhưng command khác.
- **Queue job:** web **đẩy job vào broker** (**Redis/DB/RabbitMQ/SQS**); **worker daemon trường tồn**
  (`queue:work`/Horizon, **BullMQ Worker**, Sidekiq, Celery) lấy ra thực thi; **scale = nhiều worker**.
- **Cron job:** cần **scheduler** theo giờ — **system crontab**, **Laravel Scheduler** (1 dòng cron gọi
  `schedule:run` mỗi phút), **`@nestjs/schedule`**, hoặc **K8s CronJob / AWS EventBridge**.
- **Giữ sống:** process manager — **Supervisor / systemd / pm2 / Horizon**.
- **Tách khỏi web để:** không block request, **scale độc lập**, cô lập lỗi/tài nguyên, deploy riêng.
  (Chống cron chạy chồng & xử lý job lỗi: xem `Common/resolve-problem.md`.)

[↑ Mục lục](#mục-lục)

[↑ Mục lục](#mục-lục)