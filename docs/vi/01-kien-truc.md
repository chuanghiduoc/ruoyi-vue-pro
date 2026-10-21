# 01 — Kiến trúc

> Nguồn: đọc trực tiếp source tại commit `ac022b1` (2026-10-05), nhánh `master`.

## 1. Tổng quan

`ruoyi-vue-pro` là **modular monolith**: nhiều Maven module, nhưng **một process duy nhất** (`yudao-server`, port `48080`).

- Các module **không** gọi nhau qua HTTP. Chúng là Spring Bean trong cùng JVM.
- Ranh giới module được giữ bằng **package `api`**, không phải bằng network.
- Có thể tách thành microservice sau này (xem `05-monolith-vs-microservices.md`).

```
┌──────────────────────────────────────────────┐
│                yudao-server                  │
│  (1 process, 1 JAR, port 48080)              │
│                                              │
│  ┌─────────────┐  ┌─────────────┐            │
│  │  module-    │  │  module-    │   ...      │
│  │  system     │  │  infra      │            │
│  └──────┬──────┘  └──────┬──────┘            │
│         └────────┬───────┘                   │
│                  ▼                           │
│           yudao-framework                    │
│      (15 starter kỹ thuật + common)          │
└──────────────────┬───────────────────────────┘
                   ▼
        MySQL · Redis · (MQ / S3 / ...)
```

## 2. Sơ đồ phụ thuộc

```
yudao-dependencies        ← BOM: quản lý version toàn bộ dependency
        ▲
yudao-framework           ← hạ tầng kỹ thuật, KHÔNG chứa nghiệp vụ
  ├── yudao-common
  └── 15 starter
        ▲
yudao-module-*            ← nghiệp vụ, mỗi module 1 Maven artifact
        ▲
yudao-server              ← entrypoint, chỉ phụ thuộc các module được bật
```

Quy tắc: **module không được phụ thuộc ngược vào `yudao-server`**, và **framework không được biết gì về module nghiệp vụ**.

## 3. Vai trò từng tầng

| Tầng | Vai trò | Ví dụ |
|---|---|---|
| `yudao-dependencies` | BOM, khoá version | Spring Boot 2.7.18, MyBatis-Plus 3.5.16 |
| `yudao-common` | Util dùng chung, base class, exception | `CommonResult`, `PageResult`, `ServiceException` |
| `yudao-framework` | Starter kỹ thuật | tenant, security, mybatis, redis, job, mq… |
| `yudao-module-*` | Nghiệp vụ | system, infra, bpm, mall… |
| `yudao-server` | Ghép module + cấu hình + main class | `YudaoServerApplication` |
| `yudao-ui` | Frontend (repo riêng) | Vue3 / Vue2 / uni-app |

### 15 starter trong `yudao-framework`

| Starter | Nhiệm vụ |
|---|---|
| `yudao-spring-boot-starter-web` | MVC, global exception, `CommonResult` |
| `yudao-spring-boot-starter-security` | Token, `@PreAuthorize`, login user |
| `yudao-spring-boot-starter-mybatis` | MyBatis-Plus, base mapper, phân trang |
| `yudao-spring-boot-starter-redis` | Redis, Redisson, cache |
| `yudao-spring-boot-starter-biz-tenant` | **Multi-tenant** |
| `yudao-spring-boot-starter-biz-data-permission` | **Phân quyền dữ liệu (row-level)** |
| `yudao-spring-boot-starter-biz-ip` | Map IP → khu vực |
| `yudao-spring-boot-starter-job` | Quartz, `JobHandler` |
| `yudao-spring-boot-starter-mq` | Event / Redis / RabbitMQ / Kafka / RocketMQ |
| `yudao-spring-boot-starter-monitor` | Spring Boot Admin, SkyWalking |
| `yudao-spring-boot-starter-protection` | Idempotency, rate limit, distributed lock |
| `yudao-spring-boot-starter-excel` | Import/export Excel |
| `yudao-spring-boot-starter-test` | Base test, Redis/MySQL test |
| `yudao-spring-boot-starter-websocket` | WebSocket + auth token |
| `yudao-spring-boot-starter-biz-ip` | (trùng nhóm tiện ích) |

## 4. Cấu trúc chuẩn của một module

Tất cả module đều theo cùng một layout (đã kiểm tra thực tế):

```
yudao-module-system/
├── pom.xml
└── src/main/java/cn/iocoder/yudao/module/system/
    ├── api/            ← interface + DTO cho module KHÁC gọi
    ├── controller/
    │   ├── admin/      ← API cho admin  (/admin-api/**)
    │   └── app/        ← API cho C-end  (/app-api/**)
    ├── convert/        ← MapStruct: DO ↔ VO ↔ DTO
    ├── dal/
    │   ├── dataobject/ ← DO, map 1-1 với bảng
    │   └── mysql/      ← Mapper (MyBatis-Plus)
    ├── enums/          ← enum + error code
    ├── framework/      ← cấu hình riêng của module
    ├── job/            ← Quartz job
    ├── mq/             ← producer/consumer
    ├── service/        ← business logic
    └── util/
```

Không phải module nào cũng có đủ: module đơn giản (erp, mes, wms) không có `api/` hay `mq/`.

## 5. Luồng xử lý một request

```
HTTP
  │
  ▼
TenantSecurityWebFilter      ← kiểm tra tenant (nếu bật)
  │
  ▼
TokenAuthenticationFilter    ← parse token → LoginUser
  │
  ▼
Controller (admin/app)
  │  @Valid VO
  ▼
Service
  │  @PreAuthorize("@ss.hasPermission('system:user:query')")
  │  @DataPermission(...)
  ▼
Mapper (MyBatis-Plus)
  │  TenantDatabaseInterceptor tự thêm tenant_id
  ▼
MySQL
```

Response luôn bọc trong `CommonResult<T>`:

```json
{ "code": 0, "data": {...}, "msg": "" }
```

## 6. Quy tắc gọi chéo module

**Được:** gọi qua package `api`.

```java
// module-system expose
public interface UserApi {
    UserRespDTO getUser(Long id);
}

// module-khác inject
@Resource
private UserApi userApi;
```

**Không được:** inject `UserService` hoặc `UserMapper` của module khác.

Lý do: `api` là hợp đồng ổn định. Khi tách microservice, chỉ cần đổi implementation của `UserApi` sang Feign client, business code không đổi.

## 7. Multi-tenant

Bật/tắt trong `application.yaml`:

```yaml
yudao:
  tenant:
    enable: true
    ignore-urls: [...]        # URL không cần tenant
    ignore-tables: [...]      # bảng global, không có tenant_id
    ignore-caches: [...]      # cache key không prefix tenant
```

### Cơ chế

| Thành phần | Nhiệm vụ |
|---|---|
| `TenantContextHolder` | Giữ `tenantId` trong ThreadLocal |
| `TenantSecurityWebFilter` | Lấy tenant từ header `tenant-id` / domain, set vào context |
| `TenantDatabaseInterceptor` | Chặn MyBatis-Plus, tự thêm `tenant_id = ?` |
| `TenantBaseDO` | Base class cho DO có tenant |
| `@TenantIgnore` | Bỏ qua filter cho method cụ thể |
| `TenantJobAspect` | Set tenant context khi job chạy |
| `TenantRedisCacheManager` | Prefix cache key theo tenant |
| `TenantRedisMessageInterceptor` | Prefix channel Redis pub/sub |
| `TenantRabbitMQMessagePostProcessor` | Gắn tenant vào message header |

### Điểm dễ rò rỉ dữ liệu

Interceptor **chỉ** chặn query đi qua MyBatis-Plus. Các trường hợp sau phải tự xử lý:

1. `@Select` / XML mapper viết tay → tự thêm `tenant_id`.
2. Native query, store procedure.
3. Cache key không đi qua `TenantRedisCacheManager`.
4. `@Async`, thread pool, `CompletableFuture` → mất ThreadLocal.
5. Export Excel / báo cáo query thẳng.
6. Job không có `TenantJobAspect`.

**Test bắt buộc:** login tenant A, query danh sách, xác nhận **không** thấy bản ghi của tenant B.

### Hai kiểu cô lập

| Kiểu | Cách làm | Phù hợp |
|---|---|---|
| Shared DB + `tenant_id` | Mặc định của repo | Nhiều tenant nhỏ, chi phí thấp |
| Separate DB / schema | Phải tự cấu hình dynamic datasource | Tenant enterprise, yêu cầu compliance |

Repo chỉ hỗ trợ sẵn kiểu **shared DB**. Muốn DB riêng phải làm thêm (tham khảo `yudao-cloud` hoặc `lamp-cloud`).

## 8. Phân quyền dữ liệu (row-level)

Ngoài RBAC (menu/button), còn có lọc dữ liệu theo phòng ban:

```java
@DataPermission({
    @DataPermission.Column(columnName = "dept_id", javaType = Long.class)
})
public List<UserDO> getUserList(...) { ... }
```

| Thành phần | Nhiệm vụ |
|---|---|
| `@DataPermission` | Đánh dấu method/class cần lọc |
| `DataPermissionRuleHandler` | Áp rule vào SQL |
| `DeptDataPermissionRule` | Rule theo phòng ban |
| `DataPermissionUtils` | Bật/tắt tạm thời trong code |

Đây là thứ RBAC thông thường **không** có: cùng một role, nhưng trưởng phòng A chỉ thấy dữ liệu phòng A.

## 9. Job (Quartz)

- `JobHandler` — interface business job implement.
- `JobHandlerInvoker` — cầu nối Quartz → Spring Bean.
- `SchedulerManager` — add/update/pause/resume job runtime.
- `CronUtils` — validate cron.
- Lưu trữ: **JDBC JobStore, cluster mode** (`isClustered: true`).
- Bảng: `infra_job`, `infra_job_log`.
- Cần import `sql/mysql/quartz.sql` trước khi bật job.

```java
@Component
public class MyJob implements JobHandler {
    @Override
    public String execute(String param) {
        // ...
        return "ok";
    }
}
```

## 10. Codegen

| Thành phần | Nhiệm vụ |
|---|---|
| `CodegenBuilder` | Đọc metadata bảng/cột từ DB |
| `CodegenEngine` | Render template → file |
| `CodegenProperties` | Cấu hình (base package, VO type, có sinh test không) |
| `CodegenTableDO` / `CodegenColumnDO` | Lưu cấu hình sinh mã |

Flow:

```
Chọn bảng DB
  → CodegenBuilder đọc schema
  → Cấu hình (module, package, loại bảng: single/tree/master-detail)
  → CodegenEngine render
  → Preview / Download ZIP
  → Copy vào module tương ứng
```

Sinh ra: DO, Mapper, Service, Controller, VO, Convert, SQL menu, (tuỳ chọn) unit test, Vue view.

**Lưu ý:** codegen chỉ đáng dùng cho CRUD/admin. Domain logic phức tạp nên viết tay — code sinh ra sửa tay sẽ mất khả năng regenerate.

## 11. Trạng thái build mặc định (rất quan trọng)

Trên nhánh `master`, **chỉ 2 module được bật**:

```xml
<!-- pom.xml gốc -->
<modules>
    <module>yudao-dependencies</module>
    <module>yudao-framework</module>
    <module>yudao-server</module>
    <module>yudao-module-system</module>
    <module>yudao-module-infra</module>

    <!-- 18 module còn lại ĐANG COMMENT OUT -->
    <!-- <module>yudao-module-bpm</module> -->
    <!-- <module>yudao-module-mall</module> -->
    ...
</modules>
```

`yudao-server/pom.xml` cũng comment tương ứng.

### Bật thêm một module (ví dụ BPM)

1. Mở comment trong `pom.xml` gốc:
   ```xml
   <module>yudao-module-bpm</module>
   ```
2. Mở comment dependency trong `yudao-server/pom.xml`:
   ```xml
   <dependency>
       <groupId>cn.iocoder.boot</groupId>
       <artifactId>yudao-module-bpm</artifactId>
       <version>${revision}</version>
   </dependency>
   ```
3. Import SQL của module (nếu có) — kiểm tra `sql/mysql/`.
4. `mvn clean install -DskipTests`
5. Chạy lại `YudaoServerApplication`.

> Đây là lý do README ghi "chuyển full → mini trong 5–10 phút": thực chất là comment/uncomment, không phải xoá code.

## 12. Phân loại module

| Nhóm | Module | Ghi chú |
|---|---|---|
| **Bắt buộc** | `system`, `infra` | Nền tảng: user, role, tenant, codegen, job, file |
| **Tuỳ chọn phổ biến** | `bpm`, `pay`, `report`, `member` | Dùng cho phần lớn hệ thống |
| **Theo domain** | `mall`, `erp`, `wms`, `crm`, `mes`, `hrm`, `fms`, `pms`, `oa`, `ai`, `iot`, `im`, `mp` | Chỉ bật khi đúng nghiệp vụ |

Xem chi tiết từng module ở `03-module-guide.md`.
