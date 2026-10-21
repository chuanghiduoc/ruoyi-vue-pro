# 03 — Hướng dẫn module

> Nguồn: đọc trực tiếp source tại `D:\best-struct\ruoyi-vue-pro`, nhánh `master` (JDK 8), đối chiếu `pom.xml` và các package `controller/admin` ngày 2026-10-21.
> Đọc kèm: `00-glossary.md` (quy ước dịch) và `01-kien-truc.md` (kiến trúc, trạng thái build).

## 1. Quy ước đọc

- **Module cấp 1**: thư mục `yudao-module-<tên>`, tương ứng một Maven artifact.
- **Package gốc**: `cn.iocoder.yudao.module.<tên>`. Module con của `mall` và `iot` dùng package riêng, ghi ở từng mục.
- **Phụ thuộc**: lấy từ `<dependency>` trong `pom.xml`. Đây là phụ thuộc Maven, không phải lời gọi HTTP — mọi module chạy trong cùng JVM.
- **Nhóm**: theo `01-kien-truc.md` mục 12: bắt buộc / phổ biến / theo domain.

### 1.1 Vì sao bảng có 20 dòng

Source có **19 module cấp 1**. Bảng dưới có 20 dòng vì tách riêng `yudao-module-trade-api` — Maven module con của `mall`, chỉ chứa API/DTO/enum. Nếu chỉ đếm module cấp 1 thì là 19.

### 1.2 Mặc định `master` build gì

Trên `master`, `pom.xml` gốc chỉ mở 2 module nghiệp vụ:

```xml
<module>yudao-module-system</module>
<module>yudao-module-infra</module>
```

17 module còn lại đang bị comment (19 module cấp 1 trừ 2 module mặc định). `yudao-server/pom.xml` cũng comment tương ứng. Hệ quả:

- Source vẫn nằm trong repo, nhưng **không được compile** và **không nạp vào Spring context**.
- Muốn bật phải sửa **cả hai** file pom. Chi tiết ở mục 6.
- Bật pom chỉ giải quyết build và nạp bean; API còn cần bảng DB (xem mục 7).

## 2. Bảng tổng quan 20 module

| Module | Nhóm | Chức năng chính | Phụ thuộc module khác | Khi nào bật |
|---|---|---|---|---|
| `yudao-module-system` | Bắt buộc | Người dùng admin, role/menu/quyền, phòng ban, tenant, dict, thông báo, mail/SMS, OAuth2/social | `infra` | Luôn bật (mặc định) |
| `yudao-module-infra` | Bắt buộc | Codegen, config, datasource, file, job, log API, Redis, demo | Không | Luôn bật (mặc định) |
| `yudao-module-bpm` | Phổ biến | Thiết kế quy trình Flowable, form, task, CC, comment | `system` | Cần duyệt nhiều bước, form động |
| `yudao-module-pay` | Phổ biến | Kênh thanh toán, order, refund, transfer, ví | `system` | Cần thu tiền, hoàn tiền, ví |
| `yudao-module-report` | Phổ biến | JimuReport, GoView, báo cáo và dashboard | `system` | Cần báo cáo, biểu đồ |
| `yudao-module-member` | Phổ biến | Người dùng C-end, hạng, điểm, đăng ký, tag, địa chỉ | `infra`, `system` | Có người dùng C-end |
| `yudao-module-mall` | Theo domain | TMĐT: product, promotion, trade, statistics, trade-api | `product`, `promotion`, `statistics`, `trade`, `trade-api`; `member`, `pay` | Bán hàng online |
| `yudao-module-trade-api` | Theo domain | Hợp đồng API/DTO/enum của `trade` | Không | Không bật riêng |
| `yudao-module-erp` | Theo domain | Mua, bán, kho, tài chính, sản phẩm | `system` | Doanh nghiệp thương mại |
| `yudao-module-crm` | Theo domain | Khách hàng, lead, cơ hội, hợp đồng, công nợ, hiệu suất | `bpm`, `infra`, `system` | Bán hàng B2B, chăm sóc khách |
| `yudao-module-mes` | Theo domain | Lịch sản xuất, thiết bị, master data, sản xuất, QC, kho nhà máy | `system` | Nhà máy sản xuất |
| `yudao-module-wms` | Theo domain | Tồn kho, nhập/xuất/chuyển/kiểm kê, master data kho | `system` | Kho độc lập |
| `yudao-module-hrm` | Theo domain | Nhân viên, chấm công, bảo hiểm, tuyển dụng, lương, portal | `bpm`, `system` | Quản trị nhân sự |
| `yudao-module-fms` | Theo domain | Kế toán: chứng từ, sổ, báo cáo tài chính, kết chuyển | `infra`, `system` | Kế toán tài chính |
| `yudao-module-pms` | Theo domain | Dự án, work item, knowledge base | `infra`, `system` | Quản lý dự án, tri thức |
| `yudao-module-oa` | Theo domain | Công văn, đơn từ, lịch, phòng họp, task, công tác, xe | `bpm`, `infra`, `system` | Văn phòng, quy trình nội bộ |
| `yudao-module-ai` | Theo domain | Chat, ảnh, RAG, mindmap, model, nhạc, workflow, viết | `infra`, `system` | Tích hợp LLM |
| `yudao-module-iot` | Theo domain | Sản phẩm, thing model, thiết bị, cảnh báo, OTA, rule | `iot-biz` → `system`, `iot-core` | Thiết bị IoT |
| `yudao-module-im` | Theo domain | Bạn bè, nhóm, kênh, tin nhắn, RTC, face | `infra`, `system` | Chat, gọi thoại/video |
| `yudao-module-mp` | Theo domain | Quản lý WeChat Official Account: account, menu, material, news, message | `infra`, `system` | Vận hành WeChat OA |

## 3. Nhóm bắt buộc

### 3.1 `yudao-module-system`

**Mục đích:** Xương sống của hệ thống: tài khoản quản trị, phân quyền, tenant, danh mục và các kênh thông báo.

**Chức năng chính:**

- `auth`, `captcha`: đăng nhập, token, captcha.
- `user`, `user/profile`, `dept`, `permission` (menu/role/permission), `post`: người dùng và RBAC.
- `tenant`, `tenant/packages`: multi-tenant và gói tenant.
- `dict`, `notice`, `notify`: danh mục, thông báo nội bộ.
- `mail`: tài khoản, template, log. `sms`: channel, template, log, callback.
- `oauth2`, `social`: OAuth2 server và đăng nhập mạng xã hội.
- `ip`: tra cứu khu vực theo IP.
- `logger`: log đăng nhập và log thao tác.

**Package gốc:** `cn.iocoder.yudao.module.system`

**Phụ thuộc:** `yudao-module-infra`.

**Lưu ý:** Hầu hết module khác gọi `system` qua package `api` (`AdminUserApi`, `DeptApi`, `PermissionApi`...). `system_users` là tài khoản admin; không nhầm với `member_user` của module `member`.

### 3.2 `yudao-module-infra`

**Mục đích:** Hạ tầng kỹ thuật dùng chung: sinh mã, cấu hình, file, job, log và tiện ích vận hành.

**Chức năng chính:**

- `codegen`: đọc metadata bảng/cột, sinh code CRUD.
- `config`: tham số cấu hình động.
- `db`: cấu hình datasource động.
- `file`: quản lý file và cấu hình lưu trữ.
- `job`: Quartz job và log chạy job.
- `logger`: API access log, API error log.
- `redis`: xem thông tin monitor Redis.
- `demo`: CRUD mẫu (demo01/02/03).

**Package gốc:** `cn.iocoder.yudao.module.infra`

**Phụ thuộc:** Không.

**Lưu ý:** Là module nền, được `system` và nhiều module khác phụ thuộc. Chứa `WebSocketSenderApi` và starter WebSocket; `im` dùng để đẩy tin nhắn. Job cần `sql/mysql/quartz.sql`. Codegen chỉ phù hợp CRUD; code sinh ra sửa tay sẽ mất khả năng regenerate.

## 4. Nhóm phổ biến

### 4.1 `yudao-module-bpm`

**Mục đích:** Quy trình nghiệp vụ (BPM) trên Flowable: thiết kế, chạy và theo dõi quy trình.

**Chức năng chính:**

- `definition`: category, model, process definition, form, expression, listener, user group.
- `task`: process instance, task, activity, CC.
- `comment`: bình luận trên task.
- `base`: VO dùng chung cho dept/user trong node.
- `oa`: ví dụ nghỉ phép (`BpmOALeaveController`).

**Package gốc:** `cn.iocoder.yudao.module.bpm`

**Phụ thuộc:** `yudao-module-system`.

**Lưu ý:** Flowable tự tạo bảng `ACT_*` khi `flowable.database-schema-update=true`; bảng `bpm_*` vẫn cần schema. `bpm/oa` chỉ là demo, không thay module `oa`. `crm`, `hrm`, `oa` phụ thuộc `bpm` để duyệt.

### 4.2 `yudao-module-pay`

**Mục đích:** Thu tiền, hoàn tiền, chuyển khoản và ví.

**Chức năng chính:**

- `app`, `channel`: ứng dụng thanh toán và kênh (Alipay, WeChat).
- `order`, `refund`, `transfer`: đơn thanh toán, hoàn tiền, chuyển khoản.
- `notify`: nhận callback từ kênh thanh toán.
- `wallet`: ví, gói nạp, giao dịch.
- `demo`: đơn hàng mẫu.

**Package gốc:** `cn.iocoder.yudao.module.pay`

**Phụ thuộc:** `yudao-module-system`.

**Lưu ý:** Cần cấu hình kênh và URL callback. `pay` expose `PayOrderApi`, `PayRefundApi`, `PayTransferApi`, `PayWalletApi` cho module khác; không chứa logic đơn hàng.

### 4.3 `yudao-module-report`

**Mục đích:** Báo cáo và dashboard.

**Chức năng chính:**

- `ajreport`: nhúng JimuReport.
- `goview`: quản lý project GoView và cấp dữ liệu cho GoView qua SQL/HTTP.

**Package gốc:** `cn.iocoder.yudao.module.report`

**Phụ thuộc:** `yudao-module-system`.

**Lưu ý:** Dùng `jimureport-spring-boot-starter` và `jimubi-spring-boot-starter`. Controller của JimuReport do starter quét (`@ComponentScan("org.jeecg.modules.jmreport")`); module chỉ thêm token/permission và service ngoài. `GoViewDataService` chạy SQL do client gửi lên bằng `JdbcTemplate` — chỉ cấp cho admin tin cậy và cân nhắc chặn quyền SQL.

### 4.4 `yudao-module-member`

**Mục đích:** Tài khoản người dùng C-end và các cơ chế loyalty.

**Chức năng chính:**

- `user`, `address`, `config`: tài khoản, địa chỉ, cấu hình.
- `level`: cấp hội viên, kinh nghiệm, lịch sử.
- `point`, `signin`: điểm và điểm danh.
- `tag`, `group`: tag và nhóm hội viên.

**Package gốc:** `cn.iocoder.yudao.module.member`

**Phụ thuộc:** `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** Được `mall` dùng qua `MemberUserApi`, `MemberLevelApi`, `MemberPointApi`. Đây không phải tài khoản admin.

## 5. Nhóm theo domain

### 5.1 `yudao-module-mall`

**Mục đích:** Module aggregator cho thương mại điện tử. Bản thân `mall` là `packaging=pom`, không có `src/`.

**Cấu trúc 5 module con:**

| Module con | Chức năng chính |
|---|---|
| `yudao-module-product` | SPU/SKU, brand, category, property, comment, favorite, history |
| `yudao-module-promotion` | Coupon, seckill, bargain, combination, discount, point, reward, DIY, article, banner, kefu |
| `yudao-module-trade` | Order, aftersale, delivery, brokerage, config; expose `TradeOrderApi` |
| `yudao-module-statistics` | Thống kê trade, product, member, pay |
| `yudao-module-trade-api` | Hợp đồng API/DTO/enum của `trade` |

**Package gốc:** `cn.iocoder.yudao.module.product`, `.promotion`, `.statistics`, `.trade`, `.trade.api`

**Phụ thuộc:** aggregator gồm 5 module con. Ở tầng Maven, `trade` và `statistics` khai báo phụ thuộc trực tiếp `member` và `pay`.

**Lưu ý:** `pom.xml` gốc bật `yudao-module-mall`; `yudao-server/pom.xml` bật 4 artifact `product`, `promotion`, `trade`, `statistics` (không phải aggregator). `trade-api` đi kèm `trade`. Phải bật cả `member` và `pay`, nếu không build sẽ lỗi.

### 5.2 `yudao-module-trade-api`

**Mục đích:** Hợp đồng API của `trade` để module khác dùng mà không phụ thuộc implementation.

**Chức năng chính:**

- `TradeOrderApi`.
- DTO trong `api/order/dto`.
- Enum `aftersale`, `brokerage`, `delivery`, `notify`, `order`.

**Package gốc:** `cn.iocoder.yudao.module.trade.api`; enum ở `cn.iocoder.yudao.module.trade.enums`.

**Phụ thuộc:** Không.

**Lưu ý:** Không có controller/service. Không khai báo riêng trong `yudao-server/pom.xml`; được `trade` kéo vào qua Maven. Không bật riêng.

### 5.3 `yudao-module-erp`

**Mục đích:** ERP cho doanh nghiệp thương mại: mua, bán, kho và dòng tiền.

**Chức năng chính:**

- `product`: category, product, unit.
- `purchase`: supplier, order, nhập kho, trả hàng.
- `sale`: customer, order, xuất kho, trả hàng.
- `stock`: warehouse, stock, nhập/xuất/chuyển/kiểm kê, stock record.
- `finance`: tài khoản, phiếu thu, phiếu chi.
- `statistics`: thống kê mua và bán.

**Package gốc:** `cn.iocoder.yudao.module.erp`

**Phụ thuộc:** `yudao-module-system`.

**Lưu ý:** Không dùng BPM. `finance` chỉ có phiếu thu/chi (`ErpFinancePayment`, `ErpFinanceReceipt`), không phải sổ kế toán; cần kế toán đầy đủ thì dùng `fms`.

### 5.4 `yudao-module-crm`

**Mục đích:** Quản lý khách hàng và quy trình bán hàng B2B.

**Chức năng chính:**

- `customer`: customer, limit config, pool config.
- `clue`, `contact`: lead và người liên hệ.
- `business`: cơ hội bán hàng và trạng thái.
- `contract`: hợp đồng và cấu hình hợp đồng.
- `followup`: lịch sử chăm sóc.
- `receivable`: kế hoạch thu và khoản phải thu.
- `product`, `permission`, `performance`, `statistics`, `operatelog`.

**Package gốc:** `cn.iocoder.yudao.module.crm`

**Phụ thuộc:** `yudao-module-bpm`, `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** Duyệt hợp đồng và khoản phải thu qua `BpmProcessInstanceApi`; bắt buộc bật `bpm`. Phân quyền dữ liệu theo team nằm trong package `permission`.

### 5.5 `yudao-module-mes`

**Mục đích:** Hệ thống điều hành sản xuất (MES) cho nhà máy.

**Chức năng chính:**

- `cal`: lịch, ngày nghỉ, kế hoạch, nhóm và ca.
- `dv`: thiết bị, kế hoạch kiểm tra, bảo trì, sửa chữa.
- `md`: master data: item, BOM, SIP/SOP, đơn vị, khách hàng, nhà cung cấp, workstation.
- `pro`: work order, route, process, card, feedback, andon, work record.
- `qc`: IQC/IPQC/OQC/RQC, defect, indicator, template.
- `tm`: dụng cụ.
- `wm`: kho nhà máy: nhập/xuất/chuyển/kiểm kê, barcode, SN.
- `home`: thống kê trang chủ.

**Package gốc:** `cn.iocoder.yudao.module.mes`

**Phụ thuộc:** `yudao-module-system`.

**Lưu ý:** Tên package dùng viết tắt; mô tả ở mức nhóm. `wm` là kho trong nhà máy, khác module `wms`. Module không có package `api` cho module khác.

### 5.6 `yudao-module-wms`

**Mục đích:** Quản lý kho độc lập.

**Chức năng chính:**

- `md`: item/SKU, brand, category, merchant, warehouse.
- `order`: receipt, shipment, movement, check.
- `inventory`: tồn kho và lịch sử.
- `home`: thống kê.

**Package gốc:** `cn.iocoder.yudao.module.wms`

**Phụ thuộc:** `yudao-module-system`.

**Lưu ý:** Ít nghiệp vụ hơn `mes/wm`; không có sản xuất hay QC. Phù hợp kho phân phối.

### 5.7 `yudao-module-hrm`

**Mục đích:** Quản trị nhân sự.

**Chức năng chính:**

- `employee`: hồ sơ, hợp đồng, chứng chỉ, học vấn/kinh nghiệm, file, biến động, nghỉ việc, thẻ lương.
- `attendance`: chấm công, nhóm, ngày nghỉ, nghỉ phép, thống kê.
- `insurance`: phương án, định mức, nhân viên, bản ghi tháng.
- `salary`: cấu hình, nhóm, option, thuế, template phiếu lương, bảng lương tháng.
- `recruit`: vị trí, kênh, ứng viên, phỏng vấn.
- `performance`: kế hoạch, template, đánh giá.
- `portal`: cổng tự phục vụ của nhân viên.
- `operatelog`, `home`.

**Package gốc:** `cn.iocoder.yudao.module.hrm`

**Phụ thuộc:** `yudao-module-bpm`, `yudao-module-system`.

**Lưu ý:** Nghỉ phép dùng `BpmProcessInstanceApi`; bắt buộc bật `bpm`. `portal` là góc nhân viên tự phục vụ (chấm công, phiếu lương, đánh giá).

### 5.8 `yudao-module-fms`

**Mục đích:** Kế toán tài chính.

**Chức năng chính:**

- `config`: account set, subject, currency, auxiliary, voucher template/word, initial balance, parameter/indicator, digest.
- `voucher`: chứng từ kế toán.
- `ledger`: sổ cái, sổ chi tiết, số dư, sổ phụ, nhiều cột, số lượng.
- `report`: bảng cân đối, báo cáo kết quả kinh doanh, lưu chuyển tiền tệ.
- `closing`: kỳ kết chuyển, scheme, template, voucher.
- `home`: thống kê.

**Package gốc:** `cn.iocoder.yudao.module.fms`

**Phụ thuộc:** `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** Dùng Aviator để tính công thức báo cáo lưu chuyển tiền tệ. Chứng từ đính kèm qua `FileApi` của `infra`.

### 5.9 `yudao-module-pms`

**Mục đích:** Quản lý dự án và cơ sở tri thức.

**Chức năng chính:**

- `pm`: project, group, member, template, announcement, favorite.
- `pm/iteration`, `pm/workitem`: iteration, work item, status, label, comment, activity, worklog.
- `pm/workbench`: bảng làm việc.
- `kb`: library, group, member, template, folder, document, label, permission.
- `kb/interaction`: comment, like, favorite, share, view record.
- `kb/recycle`: thùng rác.

**Package gốc:** `cn.iocoder.yudao.module.pms`

**Phụ thuộc:** `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** `kb` dùng `FileApi` của `infra` để lưu file chia sẻ. `pms` không phải module BPM; đừng nhầm với `bpm`.

### 5.10 `yudao-module-oa`

**Mục đích:** Văn phòng điện tử: công văn, đơn từ, lịch và cộng tác nội bộ.

**Chức năng chính:**

- Công văn, văn bản: `officialdoc` (send/receive/template), `seal`, `announcement`.
- Đơn từ và phê duyệt: `leave`, `overtime`, `regular`, `resign`, `reimbursement`, `travel`, `vehicle`, `supply`, `meetingroom`.
- Công việc: `task`, `plan`, `schedule`, `workreport`.
- Cộng tác: `discussion`, `note`, `contact`, `file`.
- Khác: `attendance`, `mail`.

**Package gốc:** `cn.iocoder.yudao.module.oa`

**Phụ thuộc:** `yudao-module-bpm`, `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** Nhiều đơn từ gọi `BpmProcessInstanceApi` (nghỉ phép, tăng ca, hoàn ứng, công tác, xe, con dấu, công văn, đặt phòng họp). `file` dùng `FileApi`; `mail` có account/provider riêng của OA.

### 5.11 `yudao-module-ai`

**Mục đích:** Tích hợp mô hình AI/LLM vào hệ thống.

**Chức năng chính:**

- `chat`: hội thoại và tin nhắn.
- `image`: tạo ảnh, có nhánh Midjourney.
- `knowledge`: knowledge, document, segment (RAG).
- `mindmap`, `write`, `music`.
- `model`: model, API key, chat role, tool.
- `workflow`: luồng xử lý AI.

**Package gốc:** `cn.iocoder.yudao.module.ai`

**Phụ thuộc:** `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** Dùng Spring AI 1.1.8, hỗ trợ OpenAI, Azure OpenAI, Anthropic, DeepSeek, Ollama, Stability AI, DashScope và vector store Qdrant/Redis/Milvus. Cần API key. Có `tool/` (function/method) và `util/` riêng; không có package `api` cho module khác.

### 5.12 `yudao-module-iot`

**Mục đích:** Nền tảng IoT: quản lý sản phẩm, thiết bị, dữ liệu và rule.

**Cấu trúc 3 module con:**

| Module con | Chức năng chính |
|---|---|
| `yudao-module-iot-core` | DTO, topic, protocol dùng chung |
| `yudao-module-iot-biz` | Nghiệp vụ và admin API: product, thing model, device, alert, OTA, rule, statistics |
| `yudao-module-iot-gateway` | Gateway kết nối thiết bị; Spring Boot app riêng |

**Package gốc:** `cn.iocoder.yudao.module.iot` (biz), `cn.iocoder.yudao.module.iot.core`, `cn.iocoder.yudao.module.iot.gateway`.

**Phụ thuộc:** `iot-biz` → `system`, `iot-core`; `iot-gateway` → `iot-core`; aggregator không phụ thuộc module khác.

**Lưu ý:** `iot-gateway` có main class `IotGatewayServerApplication`, chạy ngoài `yudao-server`. Hỗ trợ HTTP, TCP, UDP, WebSocket, CoAP, MQTT và EMQX hook; các protocol mặc định `enabled: false` trong `application.yaml` của gateway, phải bật protocol cần dùng.

### 5.13 `yudao-module-im`

**Mục đích:** IM: bạn bè, nhóm, kênh, tin nhắn và gọi thoại/video.

**Chức năng chính:**

- `friend`: bạn bè và lời mời kết bạn.
- `group`: nhóm, thành viên, yêu cầu vào nhóm.
- `channel`: kênh và material.
- `conversation`, `message`: hội thoại, tin nhắn riêng/nhóm/kênh.
- `face`: bộ biểu cảm.
- `rtc`: cuộc gọi và LiveKit.
- `manager`: quản trị channel, face, friend, group, message, rtc, sensitive word, statistics.

**Package gốc:** `cn.iocoder.yudao.module.im`

**Phụ thuộc:** `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** Đẩy tin nhắn qua `WebSocketSenderApi` của `infra`. RTC dùng LiveKit ngoài, cấu hình `yudao.im.livekit-url` (mặc định `ws://127.0.0.1:7880`). Lọc từ nhạy cảm bằng thư viện `sensitive-word`.

### 5.14 `yudao-module-mp`

**Mục đích:** Quản lý WeChat Official Account.

**Chức năng chính:**

- `account`, `menu`, `material`.
- `news`: draft và đăng bài.
- `message`: auto-reply, message, template.
- `user`, `tag`, `statistics`.
- `open`: nhận callback/open API từ WeChat.

**Package gốc:** `cn.iocoder.yudao.module.mp`

**Phụ thuộc:** `yudao-module-infra`, `yudao-module-system`.

**Lưu ý:** Dùng `wx-java-mp-spring-boot-starter`. Cần AppID/AppSecret và callback URL public. Hỗ trợ nhiều account.

## 6. Bật một module: sửa 2 file pom

Đây là điều kiện bắt buộc. Thiếu một trong hai bước thì module không được build hoặc không được nạp vào Spring context.

### 6.1 Bước 1 — `pom.xml` gốc

Mở comment `<module>` tương ứng trong `<modules>`:

```xml
<modules>
    <module>yudao-dependencies</module>
    <module>yudao-framework</module>
    <module>yudao-server</module>
    <module>yudao-module-system</module>
    <module>yudao-module-infra</module>

    <!-- mở comment module cần dùng -->
    <module>yudao-module-bpm</module>
</modules>
```

### 6.2 Bước 2 — `yudao-server/pom.xml`

Mở comment `<dependency>` tương ứng:

```xml
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-bpm</artifactId>
    <version>${revision}</version>
</dependency>
```

### 6.3 Trường hợp đặc biệt

| Module | `pom.xml` gốc | `yudao-server/pom.xml` |
|---|---|---|
| `mall` | `<module>yudao-module-mall</module>` | 4 dependency: `yudao-module-product`, `yudao-module-promotion`, `yudao-module-trade`, `yudao-module-statistics`; đồng thời bật `member`, `pay` |
| `iot` | `<module>yudao-module-iot</module>` | 1 dependency: `yudao-module-iot-biz`. Gateway chạy riêng |
| `trade-api` | Không cần | Không cần; `trade` kéo theo |
| `report` | `<module>yudao-module-report</module>` | `yudao-module-report` |

### 6.4 Build và chạy

```bash
mvn clean install -DskipTests
```

Nếu build riêng server, dùng `-am` để Maven build kèm module phụ thuộc:

```bash
mvn -pl yudao-server -am clean package -DskipTests
```

Sau đó chạy `YudaoServerApplication`. Kiểm tra:

1. Log khởi động có bean/controller của module.
2. Gọi thử một endpoint admin của module; 404 nghĩa là chưa nạp.
3. Với module có job/MQ, kiểm tra bảng và cấu hình tương ứng.

## 7. SQL: điểm dễ bỏ sót

Trong repo này, `sql/mysql/` chỉ có:

| File | Nội dung |
|---|---|
| `ruoyi-vue-pro.sql` | 48 bảng: `system_*`, `infra_*`, `yudao_demo*` và seed data |
| `quartz.sql` | Bảng Quartz cho job |

**Không có** bảng nghiệp vụ của các module tuỳ chọn trong `sql/mysql/`: `bpm_*`, `crm_*`, `erp_*`, `product_*`, `promotion_*`, `trade_*`, `mes_*`, `wms_*`, `hrm_*`, `fms_*`, `pms_*`, `oa_*`, `ai_*`, `iot_*`, `im_*`, `member_*`, `mp_*` (kiểm tra ngày 2026-10-21).

Hệ quả: bật pom thành công không có nghĩa API chạy được. Phải chuẩn bị schema của module từ nguồn triển khai tương ứng, rồi import trước khi gọi API. Lỗi thường gặp là `Table 'xxx' doesn't exist`, xuất hiện lúc runtime chứ không phải lúc build.

Ngoại lệ:

- Flowable tự tạo bảng `ACT_*` khi `flowable.database-schema-update=true`.
- Quartz cần `quartz.sql` nếu bật job.

## 8. Ba quy tắc phụ thuộc cần nhớ

1. **`system` + `infra` là gốc.** `infra` không phụ thuộc module nào; `system` phụ thuộc `infra`. Hầu hết module còn lại phụ thuộc một hoặc cả hai.
2. **`bpm` là gốc của nhóm có workflow.** `crm`, `hrm`, `oa` khai báo phụ thuộc `bpm` vì dùng `BpmProcessInstanceApi`; bật các module này mà thiếu `bpm` sẽ lỗi build.
3. **`mall` cần `member` + `pay`.** Ở tầng Maven, `trade` và `statistics` khai báo trực tiếp hai module này. Bật `mall` mà thiếu một trong hai sẽ lỗi build, dù `pom.xml` gốc chỉ nhắc `mall`.

Ngoài ra:

- `report` chỉ phụ thuộc `system`; không cần `infra`.
- `iot` không phụ thuộc module nghiệp vụ nào ở aggregator; `iot-biz` phụ thuộc `system` và `iot-core`.
- `trade-api` không có dependency; chỉ là hợp đồng API.
