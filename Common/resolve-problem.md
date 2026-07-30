<a id="mục-lục"></a>
# Project — Bài toán thực tế & Cách giải quyết (Problem Solving)

> Các bài toán/tình huống kỹ thuật thực tế đã gặp và cách xử lý: tối ưu hiệu năng, xử lý
> bug khó, scale hệ thống, đồng bộ dữ liệu, race condition, xử lý lượng lớn dữ liệu...
> Trả lời nên theo hướng **STAR** (Situation – Task – Action – Result), nêu rõ trade-off & kết quả.
> (Câu hỏi tổng quan về dự án — bối cảnh, vai trò, kiến trúc, điểm nhấn kỹ thuật: xem thư mục [ProjectHighlights/](../ProjectHighlights/).)

## 📑 Mục lục

1. [Khi gặp 1 bug thì debug ở backend thế nào? Các step và kiểm tra những gì?](#q1)
2. [Queue job / cron job lỗi thì xử lý thế nào? Làm sao biết lỗi để đối ứng ngay?](#q2)
   - 2.1. [Riêng cron job: chống chạy chồng (overlap) & phát hiện cron không chạy](#q2-1)

---

<a id="q1"></a>
## 1. Khi gặp 1 bug thì debug ở backend thế nào? Các step và kiểm tra những gì?

### 📖 Giới thiệu kiến thức

**Bối cảnh câu hỏi:** người phỏng vấn không muốn nghe "em đặt breakpoint / em `console.log`". Họ muốn xem
bạn có **quy trình** để **thu hẹp không gian giả thuyết một cách có hệ thống**, và có phân biệt được
**triệu chứng** với **root cause**. Cốt lõi của debug backend: backend **không có UI để nhìn**, nên mọi
kết luận phải dựa vào **bằng chứng** (log, metric, trace, dữ liệu) — **đoán rồi sửa thử là phản pattern**.

#### Step 0 — Triage trước khi debug: "đang chảy máu" hay không?

Việc đầu tiên **không phải** tìm nguyên nhân mà là **đánh giá mức độ**:

- **Ảnh hưởng:** bao nhiêu user / endpoint nào / có **mất hoặc sai dữ liệu** không (nặng nhất) / có rủi ro
  bảo mật không?
- **Thời điểm bắt đầu:** trùng với **deploy / migration / thay config / traffic tăng** nào không? Đây là
  câu hỏi cho ra manh mối nhanh nhất trong thực tế.
- **Nếu là sự cố production:** **cầm máu trước, điều tra sau** — rollback / tắt **feature flag** / scale
  tạm / chuyển traffic. Nhưng phải **giữ bằng chứng trước khi restart** (dump log, snapshot, danh sách
  process/connection), vì restart thường **xoá sạch dấu vết**.

#### Step 1 — Tái hiện (reproduce) & viết rõ "expected vs actual"

- Xác định **input tối thiểu** gây lỗi: request nào, payload nào, user/role nào, dữ liệu nào.
- Ghi rõ **mong đợi gì / thực tế ra gì** — nhiều "bug" hoá ra là **hiểu sai requirement**, không phải lỗi code.
- **Không tái hiện được?** Đừng ngồi đoán: **tăng khả năng quan sát** rồi bắt lần sau —
  thêm **correlation/request ID**, log input + nhánh rẽ, bật sampling trace cho endpoint đó, ghi lại
  full context khi exception (Sentry).

#### Step 2 — Thu hẹp phạm vi theo tầng (chia đôi vấn đề)

Backend là một chuỗi: **client → LB/Nginx → app → cache/DB → external API**. Hãy kiểm tra **ở từng biên**
để biết lỗi nằm ở đoạn nào, thay vì đọc code từ đầu:

| Kiểm tra ở biên | Cách xác định | Nếu OK ở đây thì loại trừ |
|---|---|---|
| Request có **tới app** không? | access log của Nginx/LB, log middleware đầu vào | vấn đề DNS/LB/CORS/firewall |
| App có **nhận đúng dữ liệu** không? | log payload/header đã parse (không log secret) | lỗi client gửi sai, middleware chặn |
| App có **gọi DB/cache** không, câu gì? | query log, `SHOW PROCESSLIST`, log của ORM | logic nhánh rẽ sai trước đó |
| **External API** trả gì? | log request/response + status + thời gian | lỗi phía mình |
| **Response ra khỏi app** có đúng không? | so response app (curl trực tiếp container) vs qua proxy | nếu khác → lỗi **proxy/cache/header** |

> **Kỹ thuật xương sống:** `curl` thẳng vào app (bỏ qua LB/CDN) — nếu **thẳng thì đúng, qua proxy thì
> sai** thì thủ phạm là proxy/cache/header, khỏi cần đọc code. Đây là bisect theo *hạ tầng*; còn nếu
> "trước đây chạy được" thì bisect theo *thời gian* bằng **`git bisect`**.

#### Step 3 — Kiểm tra những gì (checklist bằng chứng)

**1. Log & stack trace**
- Đọc stack trace **từ dưới lên**, tìm **dòng code của mình đầu tiên** (frame trên cùng thường là code
  thư viện, không phải nguyên nhân).
- Đọc **exception gốc** — chú ý `caused by` / inner exception; try/catch chôn lỗi (`catch {}`) là kẻ thù
  số 1 của việc debug.
- Dùng **correlation ID** để nối log của **cùng một request qua nhiều service** (thiếu nó thì log
  microservice gần như vô dụng).

**2. Metrics (trả lời "chậm hay lỗi, một chỗ hay toàn hệ thống")**
- **Error rate & status code** (4xx vs 5xx: 4xx là client/validation, 5xx là phía server).
- **Latency p95/p99** (không xem average — average che mất đuôi chậm).
- **CPU / RAM / GC**, **connection pool** đã cạn chưa, **queue depth**, số connection DB so
  `max_connections`.
- Đối chiếu **timeline metric với thời điểm deploy** để xác nhận nghi vấn ở Step 0.

**3. Distributed tracing (OpenTelemetry/Jaeger)** — xem **span nào ăn thời gian**; **N+1 query** hiện ra
rất rõ (một span cha kèm hàng trăm span DB giống nhau).

**4. Tầng database** (nguồn của phần lớn bug "chậm" và "sai dữ liệu")
- **Slow query log** + **`EXPLAIN`** (xem `Backend/database-system.md` — index & execution plan).
- **Lock/blocking:** `SHOW PROCESSLIST`, `information_schema.innodb_trx` (MySQL),
  `pg_stat_activity` + `pg_locks` (PostgreSQL); **deadlock log** (`SHOW ENGINE INNODB STATUS`).
- **Transaction & isolation:** thiếu transaction hoặc commit sai chỗ → dữ liệu nửa vời; đọc bẩn/lost
  update → xem **race condition & locking**.
- Dữ liệu **thật sự** trong DB là gì? (nhiều lúc "API trả sai" thực ra là **DB đã sai từ trước**.)

**5. Cache & dữ liệu tạm** — bug "sai nhưng chỉ với một số user / F5 thì lại đúng" rất hay là **cache
stale**, sai **cache key** (thiếu userId/locale/tenant), hoặc **cache invalidation** không chạy.

**6. Config & môi trường** (thủ phạm số 1 của "local chạy được, prod thì không")
- Biến env/secret thiếu hoặc sai; **timezone & locale**; version runtime/thư viện khác nhau; giới hạn
  `upload_max_filesize`/body size/timeout của Nginx-PHP-FPM; quyền file; **migration chưa chạy**.

**7. Dữ liệu biên & scale** — chỉ lỗi với dữ liệu thật: `NULL`, chuỗi unicode/emoji, số âm, tập rỗng,
timezone lệch ngày, bản ghi cũ thiếu field, hoặc chỉ lỗi khi **dữ liệu đủ lớn / đồng thời đủ cao**.

#### Step 4 — Đặt giả thuyết & kiểm chứng: mỗi lần đúng **một** biến

Đây là bước phân biệt người có phương pháp: viết ra giả thuyết ("nghi connection pool cạn"), **dự đoán
trước** bằng chứng sẽ thấy nếu đúng ("pool usage 100% + hàng loạt timeout xếp hàng"), rồi mới đi kiểm tra.
Trùng khớp → tiếp tục đào; không khớp → **loại bỏ giả thuyết** và đi nhánh khác. **Không đổi 3–4 thứ cùng
lúc**, vì khi hết lỗi bạn sẽ **không biết cái nào đã sửa** (và bug sẽ quay lại).

#### Step 5 — Công cụ đào sâu (chọn theo loại bug)

| Loại bug | Cách bắt |
|---|---|
| Logic phức tạp, tái hiện được ở local | **Debugger** step-by-step + **breakpoint có điều kiện** (Xdebug với PHP, `node --inspect` với Node) — hiệu quả hơn rắc `log` khắp nơi |
| Chỉ xảy ra trên production | So sánh **config/data/scale**; **structured log** + sampling trace; nhân bản dữ liệu prod đã **ẩn PII** về staging |
| **Flaky / không ổn định (heisenbug)** | Nghi **race condition**, thứ tự message, cache, thời gian/timezone → log kèm **timestamp + pid/thread + request ID**, chạy lặp lại nhiều lần, **tăng concurrency** để lỗi dễ nổ |
| Rò rỉ bộ nhớ / chậm dần theo thời gian | RSS/heap tăng đơn điệu → **heap snapshot 2 thời điểm rồi so sánh**; kiểm tra listener/timer/connection không đóng |
| Lỗi mạng, treo không rõ lý do | `curl -v`, `dig`, kiểm tra **SSL/timeout/keep-alive**; nặng hơn thì `netstat`/`ss`, `strace`, `tcpdump` |

#### Step 6 → 8 — Fix cho đúng cách

1. **Viết test tái hiện bug TRƯỚC khi sửa** (test phải **fail**) → sửa → test **pass**. Test này thành
   **regression test**, đảm bảo bug không sống lại (xem `Common/testing.md`).
2. **Sửa root cause, không sửa triệu chứng.** Thêm `try/catch` cho hết lỗi, tăng timeout, retry cho qua…
   chỉ **đẩy vấn đề đi xa hơn**. Dùng **5 Whys** để đi tới nguyên nhân thật.
3. **Quét chỗ khác cùng pattern** — bug thường không đơn độc (cùng một hàm dùng ở 5 nơi khác).
4. **Verify & theo dõi sau deploy:** kiểm trên staging, deploy, rồi **canh metric/log 15–30 phút** —
   chưa xem metric thì chưa gọi là xong.
5. **Postmortem blameless:** vì sao **không phát hiện sớm hơn**? → bổ sung **alert/log/test** còn thiếu.
   Một bug được xử lý tốt phải để lại **guardrail**, không chỉ một commit fix.

```bash
# Bộ lệnh "kiểm tra nhanh" khi có sự cố backend
kubectl logs -f deploy/api --since=15m | grep -i 'error\|exception'   # log gần nhất
curl -w '\ntime_total=%{time_total}\n' -o /dev/null -s http://app:3000/health  # app còn sống & nhanh?
curl -v http://localhost:3000/api/orders/1        # gọi thẳng app, bỏ qua LB/CDN
```

```sql
-- DB: ai đang chạy lâu / đang bị chặn?
SHOW FULL PROCESSLIST;                                    -- MySQL: query đang chạy
SELECT * FROM information_schema.innodb_trx ORDER BY trx_started;  -- transaction treo
SHOW ENGINE INNODB STATUS;                                -- LATEST DETECTED DEADLOCK
-- PostgreSQL
SELECT pid, state, wait_event_type, now()-query_start AS dur, query
FROM pg_stat_activity WHERE state <> 'idle' ORDER BY dur DESC;
```

```ts
// Correlation ID: điều kiện tiên quyết để debug được hệ nhiều service
app.use((req, res, next) => {
  req.reqId = req.headers['x-request-id'] ?? randomUUID();
  res.setHeader('x-request-id', req.reqId);          // trả về để user báo lỗi kèm ID
  logger.info({ reqId: req.reqId, method: req.method, url: req.url, userId: req.user?.id });
  next();                                            // mọi log/HTTP call sau đều mang reqId này
});
```

#### Bảng tra nhanh: triệu chứng → nghi ngờ gì → xem ở đâu

| Triệu chứng | Nghi ngờ đầu tiên | Xem ở đâu |
|---|---|---|
| **5xx rải rác** | exception chưa xử lý, dependency chết | error tracking + stack trace, health của downstream |
| **504/timeout** | query chậm/lock, worker cạn, external treo | slow query log, pool usage, trace span |
| **Chậm dần theo giờ/ngày** | memory leak, connection không đóng, thiếu index khi data lớn | metric RSS/pool, `EXPLAIN` |
| **Dữ liệu sai/trùng** | thiếu transaction, **race condition**, job at-least-once không idempotent | log theo `reqId`, isolation level, mục 2 |
| **Chỉ một số user bị** | dữ liệu biên, permission, feature flag, **cache key thiếu scope** | so sánh 2 user, đọc thẳng DB/cache |
| **Ngay sau deploy** | migration, config/env, breaking change API | diff commit, `git bisect`, log lúc boot |
| **Chỉ prod, local OK** | khác config/dữ liệu/scale, timezone | so sánh env, thử với dữ liệu prod đã ẩn PII |

### 🎯 Tóm tắt trả lời phỏng vấn (STAR)

- **Nguyên tắc:** debug là **thu hẹp giả thuyết bằng bằng chứng**, không phải đoán rồi sửa thử. Mục tiêu là
  **root cause**, không phải làm hết triệu chứng.
- **Quy trình 6 bước em thường theo:**
  1. **Triage:** ảnh hưởng bao nhiêu user, có sai/mất dữ liệu không, **bắt đầu sau deploy/thay đổi nào**.
     Nếu là sự cố prod → **cầm máu trước** (rollback/feature flag) nhưng **giữ log làm bằng chứng trước khi
     restart**.
  2. **Tái hiện:** tìm input tối thiểu, viết rõ **expected vs actual**. Không tái hiện được thì **tăng
     observability** (correlation ID, log context, trace) để bắt lần sau.
  3. **Thu hẹp theo tầng:** client → LB/Nginx → app → cache/DB → external. `curl` thẳng vào app để loại
     trừ proxy; "trước chạy được" thì **`git bisect`**.
  4. **Kiểm tra bằng chứng:** stack trace (đọc từ dưới lên, tìm dòng code của mình) → **metrics** (error
     rate, **p95/p99**, CPU/RAM, **connection pool**, queue depth) → **trace** (span nào chậm, N+1) →
     **DB** (slow query + `EXPLAIN`, `PROCESSLIST`, lock/deadlock, transaction) → **cache** (stale, sai
     cache key) → **config/env** (timezone, secret, migration chưa chạy) → **dữ liệu biên**.
  5. **Giả thuyết & kiểm chứng:** mỗi lần đổi **đúng 1 biến**, dự đoán trước bằng chứng rồi mới kiểm tra.
  6. **Fix đúng cách:** **viết test tái hiện bug trước khi sửa** → sửa **root cause** → quét chỗ khác cùng
     pattern → verify staging/prod và **canh metric sau deploy** → **postmortem blameless**: bổ sung
     alert/log/test để lần sau **phát hiện sớm hơn**.
- **Bug khó nhất là bug không ổn định (flaky):** nghi **race condition**/cache/timezone/thứ tự message →
  log kèm timestamp + pid + request ID, chạy lặp và **tăng concurrency** cho lỗi dễ nổ.
- **Phản pattern nên tránh:** đổi nhiều thứ một lúc; **restart cho hết lỗi** (mất bằng chứng); `try/catch`
  chôn lỗi hoặc tăng timeout để "cho qua"; sửa xong **không xem metric**; kết luận khi chưa có bằng chứng.
- **Kể theo STAR (khung để bạn điền case thật):** *S* — API bị 504 rải rác giờ cao điểm; *T* — tìm nguyên
  nhân, khôi phục dịch vụ, chống tái diễn; *A* — theo p99 + trace thấy một span DB chiếm ~90% thời gian →
  slow query log + `EXPLAIN` cho thấy **thiếu index** nên full scan, làm **connection pool cạn** → thêm
  index + sửa **N+1**, thêm alert p99 & pool usage, viết test regression; *R* — p99 từ … còn …, hết 504
  (_điền số liệu thật của bạn_).

[↑ Mục lục](#mục-lục)

<a id="q2"></a>
## 2. Queue job / cron job lỗi thì xử lý thế nào? Làm sao biết lỗi để đối ứng ngay?

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

<a id="q2-1"></a>
### 2.1. Riêng cron job: chống chạy chồng (overlap) & phát hiện cron không chạy

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
