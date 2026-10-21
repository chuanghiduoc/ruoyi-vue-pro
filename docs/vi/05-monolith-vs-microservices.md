# 05 — Monolith vs Microservices: `ruoyi-vue-pro` và `yudao-cloud`

> Số liệu repo lấy ngày **21/10/2026**. Star/fork thay đổi theo thời gian.
> Kết luận kiến trúc dựa trên source `ruoyi-vue-pro` và cấu trúc repo `yudao-cloud`.

## 1. TL;DR

- `ruoyi-vue-pro` là **modular monolith**: 1 process `yudao-server`, 1 JAR, port `48080`, module gọi nhau bằng Spring Bean trong cùng JVM.
- `yudao-cloud` là **microservices** trên cùng bộ nghiệp vụ: có `yudao-gateway`, mỗi module tách `-api`/`-server`, gọi nhau qua OpenFeign/RPC.
- Team nhỏ hoặc backend nội bộ đa dụng → chọn `ruoyi-vue-pro`; chỉ lên cloud khi nhiều team cần deploy độc lập và đã có hạ tầng vận hành.

## 2. Hai repo, cùng một gốc

Cùng tác giả **YunaiV**, cùng bộ module nghiệp vụ, cùng framework và frontend. Đây là hai repo độc lập, không phải bản fork trực tiếp của nhau.

| Repo | Star | Fork | Size | Push cuối | License |
|---|---:|---:|---:|---|---|
| `ruoyi-vue-pro` | 39.941 | 8.603 | 196 MB | 2026-10-09 | MIT |
| `yudao-cloud` | 19.653 | 4.902 | 59 MB | 2026-10-07 | MIT |

- Star/fork chỉ đo mức quan tâm, **không** đo độ phù hợp với bài toán của bạn.
- Size là dung lượng repo source, không phải artifact build và không phải RAM.

## 3. Bảng so sánh

| Tiêu chí | `ruoyi-vue-pro` (modular monolith) | `yudao-cloud` (microservices) |
|---|---|---|
| **Kiến trúc** | Nhiều Maven module, **một JVM**, một process. | Mỗi module nghiệp vụ là service riêng, chạy process riêng. |
| **Entrypoint** | `YudaoServerApplication`, port `48080`. | `GatewayServerApplication` + mỗi service một `XxxServerApplication`. Vẫn có `yudao-server/` để chạy gộp local. |
| **Cấu trúc module** | `yudao-module-xxx/` gồm controller, service, dal, api trong cùng artifact. | `yudao-module-xxx-api/` (DTO, interface) + `yudao-module-xxx-server/` (implement). Consumer chỉ phụ thuộc `-api`. |
| **Giao tiếp** | Spring Bean trong cùng JVM, qua interface ở package `api`. Không network. | OpenFeign/RPC qua network. Gateway định tuyến request từ ngoài vào service. |
| **Deploy** | 1 JAR. Build một lần, deploy một lần. | N image/JAR. Deploy, rollback, version từng service độc lập. |
| **Hạ tầng tối thiểu** | JDK + MySQL + Redis. MQ/S3 tuỳ module. | JDK + MySQL + Redis + **Nacos** + **gateway**. Production cần thêm tracing, log tập trung, monitoring. |
| **RAM ước tính** | **Ước tính** ~1 GB cho 1 process (heap + metaspace + overhead). | **Ước tính** ~8–16 GB cho stack đủ Nacos + gateway + N service, tuỳ số service và heap. |
| **Transaction** | `@Transactional` local. Một lỗi → rollback các thao tác trong cùng JVM và DB chính. | Không có transaction xuyên service. Cần Saga/TCC/outbox, idempotency, chấp nhận eventual consistency. |
| **Debug/tracing** | Một log, một stack trace. Breakpoint xuyên module. | Phải có traceId xuyên gateway → service → MQ, log tập trung, SkyWalking/OTel. Thiếu thì gần như không debug được. |
| **Scale** | Scale cả process: thêm replica của cùng JAR. Không scale riêng một module. | Scale riêng từng service. Đổi lại: nhiều thành phần phải vận hành. |
| **Tốc độ dev** | Nhanh: 1 build, 1 run config, refactor xuyên module dễ. Build toàn repo chậm dần khi bật nhiều module. | Chậm hơn lúc đầu: contract, Feign client, Nacos và nhiều process khi dev. Team lớn có thể song song theo service. |
| **Team size** | 1 team, quy ước ~1–10 dev, sở hữu toàn bộ backend. | Nhiều team, mỗi team sở hữu 1–vài service; cần platform/DevOps. |
| **Phù hợp** | Backend admin, hệ thống nội bộ, SaaS giai đoạn đầu, team ops mỏng. | Tổ chức lớn, release độc lập theo domain, tải lệch rõ giữa các module, đã có K8s/Nacos. |

> Các ngưỡng team size và RAM là **quy ước/ước tính**, không phải số liệu đo trên một workload cụ thể.

## 4. Điểm chung (dễ nhầm)

| Điểm chung | Hệ quả |
|---|---|
| Cùng tác giả YunaiV, cùng triết lý module | Đọc hiểu repo này chuyển sang repo kia gần như ngay. |
| Cùng `yudao-framework` (bộ starter kỹ thuật) | Tenant, security, mybatis, job, mq, excel… giống nhau. |
| Cùng frontend `yudao-ui-*` | Không phải viết lại admin/app khi đổi cách deploy backend. |
| Cùng bộ module nghiệp vụ `yudao-module-*` | Cùng domain, cùng bảng, cùng luồng nghiệp vụ. |
| Cùng multi-tenant, RBAC, data permission, codegen, job | Không phải "lên cloud mới có". |
| Cùng `CommonResult`, `/admin-api/**`, `/app-api/**` | Client giữ nguyên nếu gateway giữ đúng path. |
| Cùng license MIT | Dùng thương mại được, phải giữ copyright. |
| Cả hai đều có `yudao-server/` | Thấy `yudao-server` **không** đủ kết luận là monolith. |

Khác biệt thật nằm ở **cách module nói chuyện với nhau** và **cách deploy**, không nằm ở nghiệp vụ.

`yudao-cloud` vẫn giữ `yudao-server/` để chạy gộp khi local. Đây là điểm dễ nhầm nhất: nhìn cây thư mục tưởng là monolith, nhưng production có thể chạy tách service.

Hai starter chỉ có ở `yudao-cloud`:

| Starter | Vai trò |
|---|---|
| `yudao-spring-boot-starter-rpc` | Cấu hình chung cho lớp gọi RPC/OpenFeign (client, interceptor). |
| `yudao-spring-boot-starter-env` | Hỗ trợ tag môi trường khi nhiều bản deploy dùng chung hạ tầng/registry. |

## 5. Khi nào chọn monolith

Chọn `ruoyi-vue-pro` khi nỗi đau chính là **làm nhanh, ít người, ít hạ tầng**, không phải **nhiều team release độc lập**.

- Một team sở hữu toàn bộ backend, không có platform team.
- Backend riêng đa dụng: admin, ERP nội bộ, CRM, SaaS giai đoạn đầu.
- Domain còn thay đổi, cần refactor xuyên module.
- Traffic vừa, scale ngang cả ứng dụng là đủ.
- Muốn transaction local và debug trong một process.
- Hạ tầng hiện có chỉ gồm MySQL + Redis, chưa có K8s/Nacos.
- Ưu tiên time-to-market hơn tối ưu hạ tầng.

Với bối cảnh này, microservices thêm chi phí mà không giải quyết vấn đề nào đang có.

## 6. Khi nào chọn cloud

Chọn `yudao-cloud` khi lý do đến từ **tổ chức** hoặc **tải**, không phải từ cảm giác "microservices nghe hiện đại".

- Nhiều team, ownership theo domain, cần release độc lập.
- Một module có tải hoặc độ phức tạp vượt trội (AI, IM, report) cần scale riêng.
- Cần cô lập lỗi: một module chết không kéo toàn hệ thống.
- Đã có K8s, Nacos, CI/CD, monitoring, log tập trung và người trực on-call.
- Tổ chức/yêu cầu compliance buộc tách service theo team topology.
- Cần deploy/rollback từng phần mà không dừng cả hệ thống.

Nếu chưa trả lời được "team nào sở hữu service nào" và "ai trực khi Nacos chết", chưa nên chọn cloud.

## 7. Cảnh báo thực tế

### 7.1 Microservices không miễn phí

- Thêm network hop: latency, timeout, retry, lỗi một phần.
- Thêm contract: DTO, version, compatibility giữa consumer và provider.
- Thêm pipeline, image, config, secret cho từng service.
- Số cặp tích hợp tăng nhanh khi thêm service.
- Monolith có 1 thứ để deploy; cloud có N thứ để deploy đúng thứ tự.

### 7.2 Transaction phân tán

Ví dụ: tạo đơn ở `trade`, trừ kho ở `wms`, tạo thanh toán ở `pay`.

- `@Transactional` chỉ rollback DB của service đang chạy. Không rollback service khác.
- Cần Saga (orchestration hoặc choreography), TCC, hoặc transactional outbox + event.
- Cần idempotency cho retry, compensation cho bước đã thành công, và reconciliation định kỳ.
- UX phải chấp nhận trạng thái "đang xử lý", không phải thành công ngay.

Bẫy lớn: tách process nhưng **giữ chung một DB**. Khi đó vẫn có transaction local, nhưng mất luôn lợi ích cô lập dữ liệu. Đây là "distributed monolith".

### 7.3 Trace

- Request đi qua gateway → service A → Feign → service B → MQ → consumer.
- Phải propagate traceId qua HTTP header và message header.
- Cần log tập trung (ELK/Loki), tracing (SkyWalking/OTel), metric theo service.
- Thiếu trace thì câu "request này chậm/lỗi ở đâu?" không có câu trả lời.

### 7.4 Versioning API

- `-api` là contract đã publish, không còn là interface nội bộ.
- Đổi tên/xoá field DTO có thể làm consumer lỗi **runtime**, không phải compile error.
- Quy tắc: chỉ thêm field theo hướng tương thích, không đổi nghĩa field cũ.
- Thay đổi breaking phải có version mới hoặc DTO mới, kèm giai đoạn chạy song song.
- Cần contract test giữa consumer và provider trước khi deploy.

Trong monolith, đổi signature sai thường lộ ngay khi compile. Trong cloud, lỗi tương tự chỉ lộ khi chạy đủ service.

### 7.5 Chi phí vận hành

- Nacos HA, gateway HA, N service, monitoring, log storage, CI runner, K8s.
- RAM **ước tính** 8–16 GB chỉ là khởi điểm; production cần headroom và replica.
- Mỗi service cần dashboard, alert, runbook, người chịu trách nhiệm.
- Chi phí con người thường lớn hơn chi phí server.

### 7.6 "Cloud" không phải hosting

- `yudao-cloud` là một **cách tổ chức code**, không phải dịch vụ được host sẵn.
- Bạn vẫn phải tự chạy server/K8s, MySQL, Redis, Nacos, domain, TLS, backup, WAF.
- Nó không tự động scale, không tự động HA, không tự động multi-region.
- "Cloud" trong tên chỉ kiến trúc cloud-native, không đồng nghĩa SaaS hay multi-tenant. Cả hai repo đều có multi-tenant.

### 7.7 Token và tenant khi qua network

- Feign call là một request mới. `ThreadLocal` của `LoginUser` và `TenantContextHolder` không tự truyền.
- Phải copy `Authorization` và `tenant-id` sang header, rồi set lại context ở provider.
- Thiếu token → `401/403`. Thiếu tenant → sai tenant hoặc rò rỉ dữ liệu.
- Đây là lỗi bảo mật, không chỉ là lỗi chức năng.
- Test bắt buộc: user tenant A gọi một API tổng hợp qua 2 service, xác nhận chỉ thấy dữ liệu tenant A.

### 7.8 Dev gộp, deploy tách

`yudao-cloud` vẫn có `yudao-server/` để chạy gộp local. Nếu chỉ test chế độ gộp, lỗi Feign, serialization, token, tenant chỉ xuất hiện khi tách thật.

CI nên có ít nhất một môi trường chạy split, dù chỉ 2–3 service.

## 8. Lộ trình tách monolith → cloud

Vì `ruoyi-vue-pro` đã chia module và có package `api`, việc tách không phải viết lại từ đầu. Mục tiêu là giữ nguyên nghiệp vụ, chỉ đổi **cách gọi** và **cách deploy**.

```
Trước:  client → [yudao-server] ──Bean──> [module A] [module B]

Sau:    client → [gateway] → [module A server] ──Feign──> [module B server]
                                   │                         │
                              Nacos registry + config
```

| Bước | Việc làm | Điều kiện hoàn thành |
|---|---|---|
| **0. Chốt ranh giới** | Chỉ gọi chéo module qua `api`. Cấm inject `Service`/`Mapper` của module khác. Thêm ArchUnit test để chặn vi phạm. | Không còn phụ thuộc chéo ngoài `api`. |
| **1. Tách `-api`** | Tách interface + DTO + enum sang `yudao-module-xxx-api`. `-server` implement. Consumer chỉ depend `-api`. | Build `-api` độc lập, không kéo theo DB/controller. |
| **2. Đổi sang Feign** | `@FeignClient(name = "yudao-module-xxx")` extends chính interface cũ. Thêm error decoder, timeout, retry hợp lý. | Consumer gọi qua network, không còn Bean trong JVM. |
| **3. Propagate token** | Feign `RequestInterceptor` copy `Authorization`, `tenant-id`. Provider set lại `LoginUser`/`TenantContextHolder`, clear sau request. | API tổng hợp qua nhiều service trả đúng tenant và quyền. |
| **4. Xử lý transaction** | Liệt kê ghi chéo module. Chọn local transaction + outbox/event hoặc Saga. Thêm idempotency key và compensation. | Không còn chỗ nào giả định rollback xuyên service. |
| **5. Gateway + Nacos** | Thêm `yudao-gateway` định tuyến `/admin-api/**`, `/app-api/**`. Đăng ký service và external config qua Nacos. | Client vào một cổng, service resolve được qua tên. |
| **6. Tách deploy** | Image/JAR riêng, pipeline riêng, config ngoài artifact. Quyết định DB riêng hay chấp nhận dùng chung có kiểm soát. | Deploy/rollback từng service độc lập. |
| **7. Dọn monolith** | Bỏ `yudao-server` khỏi production, chỉ giữ cho local/test tích hợp. | Không còn traffic production đi vào process gộp. |

Lưu ý thứ tự thực tế:

- **Nacos** nên bật từ bước 2 để Feign resolve tên service. Gateway có thể thêm sau, trước khi mở traffic ngoài.
- Tách **một module trước** (ví dụ `infra` hoặc `system`) để đo chi phí thật. Không big-bang.
- Giai đoạn đầu có thể dùng chung DB để giảm rủi ro, nhưng phải ghi nhận là nợ kỹ thuật. Không tuyên bố "đã microservices" khi vẫn chung schema.
- Giữ `-api` ổn định. Mọi thay đổi breaking cần version và kế hoạch chạy song song.

Sai lầm thường gặp:

- Tách theo layer (controller/service/dal) thay vì theo domain.
- Tách service nhưng vẫn join bảng trực tiếp qua DB.
- Không có tracing, không có log tập trung.
- Tách tất cả cùng lúc, không có đường lùi.

## 9. Kết luận

- **1 team nhỏ hoặc backend riêng đa dụng → chọn `ruoyi-vue-pro`.**
- Chọn `yudao-cloud` khi có nhiều team cần release độc lập, tải lệch rõ giữa các module, và đã có nền tảng vận hành.
- Vì cả hai dùng chung `yudao-framework`, `yudao-ui-*` và cùng bộ module, chọn monolith trước **không phải ngõ cụt**. Có thể tách sau theo lộ trình ở mục 8.

Câu hỏi quyết định:

| Câu hỏi | Nếu "có" |
|---|---|
| Có hơn một team cần release độc lập? | Cân nhắc cloud. |
| Có module cần scale vượt xa phần còn lại? | Cân nhắc cloud. |
| Đã có người trực Nacos/K8s/monitoring? | Cloud khả thi. |
| Domain còn đổi liên tục, team còn nhỏ? | Chọn monolith. |
| Hạ tầng chỉ có MySQL + Redis? | Chọn monolith. |

Kiến trúc đúng là kiến trúc mà team của bạn **vận hành được**, không phải kiến trúc nhiều thành phần nhất.
