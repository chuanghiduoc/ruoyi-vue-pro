# 02 — Quickstart

> Áp dụng cho nhánh `master` (JDK 8 + Spring Boot 2.7).
> Nếu dùng JDK 17/21 hoặc 25, checkout `master-jdk17` / `master-jdk25` trước.

## 1. Yêu cầu môi trường

| Thành phần | Phiên bản | Ghi chú |
|---|---|---|
| JDK | **8** | `master` dùng Java 8; JDK 17/21 dùng nhánh `master-jdk17` |
| Maven | 3.8+ | |
| MySQL | 5.7 / 8.0+ | |
| Redis | 5 / 6 / 7 | |
| IDE | IntelliJ IDEA | Khuyến nghị |

Kiểm tra nhanh:

```bash
java -version
mvn -v
mysql --version
redis-cli ping
```

## 2. Tạo database và import SQL

```sql
CREATE DATABASE `ruoyi-vue-pro-jdk8`
  DEFAULT CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

Import theo thứ tự:

```bash
mysql -u root -p ruoyi-vue-pro-jdk8 < sql/mysql/ruoyi-vue-pro.sql
mysql -u root -p ruoyi-vue-pro-jdk8 < sql/mysql/quartz.sql
```

- `ruoyi-vue-pro.sql` — toàn bộ bảng nghiệp vụ + seed data.
- `quartz.sql` — **bắt buộc nếu bật module job**, thiếu sẽ lỗi `Table 'QRTZ_...' doesn't exist`.

> Repo hỗ trợ nhiều DB: xem `sql/postgresql`, `sql/oracle`, `sql/sqlserver`, `sql/dm`, `sql/kingbase`…

## 3. Cấu hình

File: `yudao-server/src/main/resources/application-local.yaml`

```yaml
spring:
  datasource:
    dynamic:
      datasource:
        master:
          url: jdbc:mysql://127.0.0.1:3306/ruoyi-vue-pro-jdk8?useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true&nullCatalogMeansCurrent=true&rewriteBatchedStatements=true
          username: root
          password: 123456

  redis:
    host: 127.0.0.1
    port: 6379
    database: 0
```

Lưu ý:

- `serverTimezone` nên đổi thành `Asia/Ho_Chi_Minh` nếu dữ liệu của bạn theo giờ Việt Nam.
- Production **phải** bật password Redis và không dùng `root/123456`.
- `spring.profiles.active` mặc định là `local` (xem `application.yaml`).

## 4. Chạy

### Cách 1 — IDE

Mở class:

```
yudao-server/src/main/java/cn/iocoder/yudao/server/YudaoServerApplication.java
```

Run → chờ log `Started YudaoServerApplication`.

### Cách 2 — Maven

```bash
mvn clean install -DskipTests
cd yudao-server
mvn spring-boot:run
```

### Cách 3 — Docker

```bash
cd script/docker
docker compose --env-file docker.env up -d
```

Ports:

| Dịch vụ | URL |
|---|---|
| API server | <http://localhost:48080> |
| Admin UI | <http://localhost:8080> |
| MySQL | `localhost:3306` (root/123456) |
| Redis | `localhost:6379` |

## 5. Kiểm tra

| Mục | URL |
|---|---|
| Health | <http://localhost:48080/actuator/health> |
| Swagger UI | <http://localhost:48080/swagger-ui> |
| Knife4j | <http://localhost:48080/doc.html> |

Tài khoản seed:

```
username: admin
password: admin123
```

> Đã kiểm tra hash BCrypt trong `sql/mysql/ruoyi-vue-pro.sql` khớp với `admin123`.
> Biến `system.user.init-password` = `123456` chỉ là mật khẩu khởi tạo cho **user mới tạo**, không phải của `admin`.

Nếu bật multi-tenant, request phải kèm header:

```
tenant-id: 1
```

## 6. Bật thêm module

Ví dụ bật `bpm`:

**Bước 1** — `pom.xml` gốc:

```xml
<module>yudao-module-bpm</module>
```

**Bước 2** — `yudao-server/pom.xml`:

```xml
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-bpm</artifactId>
    <version>${revision}</version>
</dependency>
```

**Bước 3** — import SQL của module (nếu có) trong `sql/mysql/`.

**Bước 4**:

```bash
mvn clean install -DskipTests
```

> Thiếu bước 1 hoặc 2 thì module **không được compile** hoặc **không được nạp vào Spring context**, dù source vẫn nằm trong repo.

## 7. Lỗi thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| `Table 'xxx.system_users' doesn't exist` | Chưa import SQL | Import `ruoyi-vue-pro.sql` |
| `Table 'QRTZ_...' doesn't exist` | Chưa import `quartz.sql` | Import `quartz.sql` hoặc tắt job |
| `Access denied for user` | Sai user/pass MySQL | Sửa `application-local.yaml` |
| `Unable to connect to Redis` | Redis chưa chạy | `redis-server` / `docker run redis` |
| `Port 48080 was already in use` | Trùng port | Đổi `server.port` hoặc kill process |
| `Unsupported class file major version` | Sai JDK | `master` cần JDK 8; dùng JDK 17 thì checkout `master-jdk17` |
| `Invalid bound statement (not found)` | Mapper chưa scan / sai package | Kiểm tra `@MapperScan` và `mybatis-plus.type-aliases-package` |
| Request trả `tenant-id` null | Thiếu header tenant | Thêm `tenant-id: 1` hoặc tắt tenant khi dev |
| Module không xuất hiện trong Swagger | Chưa bật ở 2 file pom | Làm lại mục 6 |

## 8. Sau khi chạy

- Đọc `01-kien-truc.md` để hiểu luồng request và multi-tenant.
- Đọc `03-module-guide.md` để biết module nào cần bật.
- Đọc `04-quy-uoc.md` trước khi viết module mới.
