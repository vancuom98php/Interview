<a id="mục-lục"></a>
# Project — Bài toán thực tế & Cách giải quyết (Problem Solving)

> Các bài toán/tình huống kỹ thuật thực tế đã gặp và cách xử lý: tối ưu hiệu năng, xử lý
> bug khó, scale hệ thống, đồng bộ dữ liệu, race condition, xử lý lượng lớn dữ liệu...
> Trả lời nên theo hướng **STAR** (Situation – Task – Action – Result), nêu rõ trade-off & kết quả.
> (Câu hỏi tổng quan về dự án — bối cảnh, vai trò, kiến trúc, điểm nhấn kỹ thuật: xem thư mục [ProjectHighlights/](../ProjectHighlights/).)

## 📑 Mục lục

1. [Queue job / cron job lỗi thì xử lý thế nào? Làm sao biết lỗi để đối ứng ngay?](#q1)
   - 1.1. [Riêng cron job: chống chạy chồng (overlap) & phát hiện cron không chạy](#q1-1)

---

<a id="q1"></a>
## 1. Queue job / cron job lỗi thì xử lý thế nào? Làm sao biết lỗi để đối ứng ngay?

### 📖 Giới thiệu kiến thức

**Bối cảnh bài toán:** hệ thống có nhiều **job chạy nền** (queue worker + cron) — vd pipeline đồng bộ lịch
`sync-meeting` (BullMQ) trong Virtual Office: sync initial, backfill, subscribe webhook, refresh token.
Khác request HTTP có user thấy lỗi ngay, **job hỏng thường âm thầm** — không ai biết cho tới khi dữ liệu
sai/thiếu. Bài toán chia **2 vế**: (A) **làm job chịu lỗi tốt** (resilient) và (B) **quan sát để biết &
phản ứng ngay** (observability + alerting).

#### A. Xử lý khi job lỗi (làm job resilient)

- **Retry + exponential backoff + jitter:** lỗi tạm thời (mạng chập chờn, DB busy, rate-limit) thường
  tự khỏi → thử lại. **Không retry ngay lập tức** (dồn tải, thundering herd) mà giãn cấp số nhân
  `min(base·2^n, max)` + thêm **jitter** ngẫu nhiên để nhiều job không cùng retry một lúc. Giới hạn số
  lần (vd 3–5).
- **Idempotency (quan trọng):** hầu hết queue giao **at-least-once** → một job **có thể chạy lại/lặp**.
  Job phải **idempotent**: dùng **idempotency key** / cờ đã-xử-lý / `UPSERT` để chạy 2 lần vẫn ra 1 kết quả
  (vd trừ tiền, gửi mail phải chống trùng). Liên hệ tính *idempotent* của HTTP method — `Common/common.md`.
- **Dead Letter Queue (DLQ):** job **hết số lần retry** → **không vứt đi** mà đẩy sang **hàng đợi lỗi**
  (DLQ / bảng `failed_jobs`) kèm payload + lỗi → để **điều tra, sửa, rồi replay** thay vì mất dữ liệu.
- **Poison message:** message **luôn** gây lỗi (dữ liệu hỏng) sẽ retry vô tận, **kẹt cả queue** → tách
  nhanh vào DLQ, không để chặn job khác.
- **Timeout cho job:** job treo (gọi API bên thứ 3 không phản hồi) → đặt **timeout** để kill & retry, tránh
  worker bị giữ mãi.
- **Circuit breaker:** nếu **downstream chết** (provider 5xx liên tục) → **mở circuit** dừng gọi một lúc
  thay vì để hàng loạt job cùng fail dồn.
- **Graceful shutdown:** khi deploy/restart worker, xử lý `SIGTERM` để **job đang chạy hoàn tất hoặc được
  requeue**, không bỏ dở giữa chừng (liên hệ `onApplicationShutdown` — `Backend/nestjs.md`).
- **Lưu trạng thái + payload lỗi:** ghi `pending/processing/failed/done` và **response lỗi** để replay &
  audit — thay vì xoá dấu vết (giống pattern đánh dấu `isValid=false` ở `ProjectHighlights/virtual-office-backend.md`).

```ts
// BullMQ (NestJS/Node) — retry + backoff + bắt sự kiện thất bại → DLQ
new Worker('sync', handler, {
  // mỗi job: thử 5 lần, backoff luỹ thừa (1s,2s,4s...) — BullMQ tự thêm jitter
});
await queue.add('sync', payload, {
  attempts: 5,
  backoff: { type: 'exponential', delay: 1000 },
  removeOnComplete: 1000,        // giữ lịch sử vừa đủ
});

worker.on('failed', async (job, err) => {
  if (job.attemptsMade >= job.opts.attempts) {
    await deadLetterQueue.add('sync-dlq', { data: job.data, err: err.message }); // vào DLQ
    metrics.increment('job.failed', { queue: 'sync' });                         // bắn metric để alert
  }
});
```

```php
// Laravel Queue — khai báo retry/backoff ngay trong Job; job fail vào bảng failed_jobs
class SyncMeetingJob implements ShouldQueue {
    public int $tries = 5;
    public array $backoff = [10, 30, 60];          // giãn cách giữa các lần retry (giây)
    public int $timeout = 120;                     // job treo quá 120s → kill & retry
    public function failed(\Throwable $e): void {  // gọi khi hết retry
        Log::error('sync failed', ['e' => $e->getMessage()]);
        // notify Slack / đánh dấu trạng thái để replay
    }
}
// Replay job lỗi: php artisan queue:retry all   (đọc từ failed_jobs)
```

#### B. Làm sao biết lỗi để **đối ứng ngay** (observability + alerting)

Nguyên tắc: **detect → alert → notify → respond**. Ba trụ observability + kênh cảnh báo:

- **Logging tập trung (structured):** đẩy log JSON về **ELK / Loki / CloudWatch** để **search & correlate**
  (theo `jobId`, `userId`). Log rời rạc trên từng worker gần như vô dụng khi điều tra.
- **Metrics + ngưỡng cảnh báo (Prometheus + Grafana):** theo dõi **queue depth (backlog)**, **failed rate**,
  **retry rate**, **thời gian xử lý**, **số worker sống**. **Alert khi vượt ngưỡng** — vd *backlog tăng
  liên tục* = worker chết/tắc, *failed rate > X%* = downstream hỏng.
- **Error tracking (Sentry / Bugsnag):** gom **exception + stack trace**, **dedupe**, gắn context và
  **thông báo tức thì** khi có lỗi mới / lỗi tăng đột biến.
- **Alerting & on-call:** **Alertmanager / PagerDuty / Opsgenie** đẩy cảnh báo ra **Slack / email / gọi
  điện**, có **escalation** (không ai xử lý → báo cấp cao hơn). Alert phải **actionable** (kèm link
  runbook), tránh spam gây "alert fatigue".
- **Dashboard queue realtime:** **Laravel Horizon** (`Backend/laravel.md`) / **Bull Board** — nhìn thấy job
  failed, throughput, retry, worker để **xử lý/replay** ngay. **Telescope** để soi lịch sử job (laravel.md).
- **Distributed tracing (OpenTelemetry):** lần theo một job xuyên nhiều service khi hệ vi dịch vụ.

> **Chốt:** *xử lý lỗi* (retry/DLQ/idempotency) và *biết lỗi* (metrics/alert/heartbeat) là **hai việc tách
> nhau** — job có DLQ mà không có alert thì lỗi vẫn nằm im. Phải có **cả hai**.

### 🎯 Tóm tắt trả lời phỏng vấn (STAR)

- **S (Situation):** hệ thống có nhiều **background job** (queue + cron) — vd pipeline đồng bộ lịch
  `sync-meeting`. Job **hỏng âm thầm**: không có user nhìn thấy lỗi như request HTTP.
- **T (Task):** đảm bảo job **không mất dữ liệu khi lỗi** và team **biết lỗi sớm để xử lý ngay**, chịu được
  lỗi tạm thời của provider bên thứ 3.
- **A (Action):**
  - *Chịu lỗi:* **retry + exponential backoff + jitter**; **idempotency** (queue *at-least-once* → job chạy
    lại an toàn); **DLQ / `failed_jobs`** cho job hết retry để **replay**; thêm **timeout**, **circuit
    breaker**, **graceful shutdown**, và **lưu trạng thái + payload lỗi** (kiểu `isValid=false`).
  - *Biết lỗi:* **detect → alert → notify → respond** — **log tập trung** (ELK/Loki), **metrics + ngưỡng**
    (Prometheus/Grafana, nhất là **queue depth** & **failed rate**), **error tracking** (Sentry); cảnh báo
    qua **Alertmanager/PagerDuty/Slack** (có escalation, kèm runbook); **dashboard** Horizon/Bull Board để
    replay từ DLQ.
- **R (Result):** job lỗi **không còn mất dữ liệu** (nằm ở DLQ, replay được); lỗi/backlog **được phát hiện
  trong vài phút** thay vì đợi user báo. _(→ điền số liệu thật của bạn: giảm X% job fail, MTTR từ … còn …,
  backlog giảm …)._
- **Nhấn khi trả lời:** DLQ để **không mất dữ liệu**, alert theo **backlog/failed-rate**, và **heartbeat cho
  cron** — vì background job **hỏng âm thầm** là rủi ro lớn nhất.

[↑ Mục lục](#mục-lục)

<a id="q1-1"></a>
### 1.1. Riêng cron job: chống chạy chồng (overlap) & phát hiện cron không chạy

#### 📖 Giới thiệu kiến thức

Cron có **2 lỗi đặc thù** mà queue thường không có:

- **Chạy chồng (overlap):** cron 5 phút/lần nhưng một lần chạy **lâu hơn 5 phút** → lần sau **khởi động
  khi lần trước chưa xong** → chạy song song gây trùng dữ liệu / tranh chấp. **Khắc phục:** giành **lock**
  trước khi chạy, đang chạy thì **bỏ qua** lần này.
  - **Laravel:** `->withoutOverlapping()` trên schedule (dùng cache lock nội bộ).
  - **Tự làm:** Redis `SET lockKey val NX EX ttl` (giống distributed lock ở
    `Backend/database-system.md` / OAuth refresh lock) — chỉ 1 tiến trình giữ lock mới chạy.
- **Cron bị bỏ lỡ (missed run) & lỗi im lặng:** server down đúng giờ hẹn → **cron skip mất một nhịp**, hoặc
  cron chạy nhưng **fail mà không ai hay**. Cron hỏng **không có ai nhìn thấy** như request HTTP → nguy hiểm.
  - **Dead man's switch / heartbeat:** mỗi lần chạy xong, cron **"ping"** một dịch vụ giám sát
    (**Healthchecks.io, Cronitor, Dead Man's Snitch**). Nếu **quá hạn không thấy ping** → dịch vụ **báo
    động** (đảo ngược logic: không phải "báo khi lỗi" mà "báo khi **im lặng bất thường**").
  - Ngoài ra: ghi **`last_run_at`** để phát hiện trễ; với công việc quan trọng, thiết kế **catch-up** (bù
    nhịp bị lỡ) thay vì chỉ chạy đúng giờ.

```php
// Laravel scheduler: chống overlap + báo động khi cron KHÔNG chạy
$schedule->command('sync:meetings')
    ->everyFiveMinutes()
    ->withoutOverlapping()            // lock: bỏ qua nếu lần trước chưa xong
    ->onFailure(fn () => Log::error('sync cron failed'))
    ->pingOnSuccess(env('HEALTHCHECK_URL'));  // heartbeat → thiếu ping thì bị báo động
```

#### 🎯 Tóm tắt trả lời phỏng vấn

- **Chống overlap:** cron chạy lâu hơn chu kỳ dễ **chồng lần** → dùng **lock** (`withoutOverlapping` /
  Redis `SET NX EX`) để mỗi lúc chỉ 1 lần chạy.
- **Biết cron không chạy:** cron hỏng **im lặng** → dùng **heartbeat / dead-man's-switch** (Healthchecks.io,
  Cronitor): cron **ping mỗi lần chạy**, **thiếu ping thì báo động**. Kèm ghi `last_run_at` và cân nhắc
  **catch-up** cho nhịp bị lỡ.

[↑ Mục lục](#mục-lục)

<!--
NHẮC FORMAT (xem /CLAUDE.md §4):

Với folder Project, phần "📖 Giới thiệu kiến thức" mô tả bối cảnh bài toán / vấn đề kỹ thuật,
phần "🎯 Tóm tắt trả lời phỏng vấn" trình bày theo STAR, ngắn gọn, có số liệu/kết quả nếu có.

<a id="qN"></a>
## N. <Bài toán>

### 📖 Giới thiệu kiến thức
<bối cảnh bài toán / ràng buộc / vì sao khó>

### 🎯 Tóm tắt trả lời phỏng vấn
- **S/T:** ...
- **A:** ...
- **R:** ...

[↑ Mục lục](#mục-lục)
-->
