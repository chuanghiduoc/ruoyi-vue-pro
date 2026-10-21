# 06 — Lộ trình: từ fork `ruoyi-vue-pro` đến backend platform

> Mục tiêu: biến fork này thành **backend platform riêng**, đa dụng, vẫn sync được upstream.
> Phạm vi: backend (`yudao-server` + các `yudao-module-*`), không gồm frontend.
> Trạng thái tham chiếu: nhánh `master`, JDK 8, commit `ac022b1` (2026-10-05) theo `01-kien-truc.md`.
> Đọc kèm: `03-module-guide.md` (chọn module) và `04-quy-uoc.md` (quy ước code, test, git).

## 1. Cách dùng tài liệu

- Làm **tuần tự** Phase 0 → Phase 4. Phase 5 chạy song song, ngay từ ngày đầu.
- Mỗi phase có 3 phần: **Mục tiêu**, **Việc làm**, **Tiêu chí done**.
- Không chuyển phase khi checklist còn ô chưa tick.
- Mọi lệnh chạy từ thư mục gốc repo, trừ khi ghi khác.
- Identifier (class, package, bảng, biến) giữ tiếng Anh. Ghi chú tiếng Việt.

## 2. Tổng quan

| Phase | Thời gian | Kết quả |
|---|---|---|
| 0 — Chạy được | 1 ngày | App chạy port `48080`, login `admin/admin123` |
| 1 — Thu gọn platform lõi | 1–2 ngày | Chỉ còn `system` + `infra`, Swagger sạch |
| 2 — Hardening multi-tenant | 2–3 ngày | Có test cách ly tenant, audit điểm rò rỉ |
| 3 — Module nghiệp vụ đầu tiên | 2–3 ngày | 1 module mới đủ entity/CRUD/quyền/menu/test |
| 4 — Chuẩn hoá vận hành | 3–5 ngày | CI, Docker, log, backup, deploy/rollback |
| 5 — Sync upstream | Định kỳ | Merge upstream không conflict |

Tổng thời gian ước tính: **9–14 ngày** cho Phase 0–4, chưa tính thời gian học codebase.

---

## Phase 0 — Chạy được (1 ngày)

### Mục tiêu

Có một môi trường local chạy được đúng nhánh `master`, đăng nhập được, trước khi sửa bất cứ thứ gì.

### Việc làm

#### 0.1. Kiểm tra môi trường

| Thành phần | Phiên bản | Ghi chú |
|---|---|---|
| JDK | **8** | `master` chỉ hỗ trợ Java 8 |
| Maven | 3.8+ | |
| MySQL | 5.7 / 8.0+ | Khuyến nghị 8.0 |
| Redis | 5 / 6 / 7 | |

```bash
java -version
mvn -v
mysql --version
redis-cli ping
```

`java -version` phải ra `1.8.0_...`. Nếu ra 17/21, đổi `JAVA_HOME` hoặc checkout nhánh `master-jdk17` / `master-jdk25` tương ứng.

#### 0.2. Tạo database và import SQL

```sql
CREATE DATABASE `ruoyi-vue-pro-jdk8`
  DEFAULT CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

```bash
mysql -u root -p ruoyi-vue-pro-jdk8 < sql/mysql/ruoyi-vue-pro.sql
mysql -u root -p ruoyi-vue-pro-jdk8 < sql/mysql/quartz.sql
```

- `ruoyi-vue-pro.sql`: toàn bộ bảng + seed data.
- `quartz.sql`: bắt buộc nếu bật job. Thiếu sẽ lỗi `Table 'QRTZ_...' doesn't exist`.
- Trên PowerShell, `<` không hoạt động. Dùng Git Bash, hoặc `Get-Content ... | mysql ...`.

#### 0.3. Cấu hình `application-local.yaml`

File: `yudao-server/src/main/resources/application-local.yaml`

Sửa tối thiểu 3 nhóm:

```yaml
spring:
  datasource:
    dynamic:
      datasource:
        master:
          url: jdbc:mysql://127.0.0.1:3306/ruoyi-vue-pro-jdk8?useSSL=false&serverTimezone=Asia/Ho_Chi_Minh&allowPublicKeyRetrieval=true&nullCatalogMeansCurrent=true&rewriteBatchedStatements=true
          username: root
          password: 123456

  redis:
    host: 127.0.0.1
    port: 6379
    database: 0
```

Lưu ý:

- `application-local.yaml` có cả datasource `slave` (lazy). Nếu không dùng đọc/ghi tách biệt, trỏ `slave` về cùng DB hoặc để nguyên vì `lazy: true`.
- `serverTimezone=Asia/Ho_Chi_Minh` nếu dữ liệu theo giờ Việt Nam.
- Không commit mật khẩu thật. Đây là file dev.
- Để sync upstream dễ hơn, có thể đặt override vào `application-fork.yaml` và chạy profile `local,fork` ngay từ đầu. Phase 0 dùng `application-local.yaml` cho nhanh; Phase 5 chuyển dần sang file fork.

#### 0.4. Chạy

```bash
mvn clean install -DskipTests
mvn -pl yudao-server spring-boot:run
```

Hoặc mở `yudao-server/src/main/java/cn/iocoder/yudao/server/YudaoServerApplication.java` và Run trong IDE.

Chờ log:

```text
Started YudaoServerApplication
```

#### 0.5. Kiểm tra

```bash
curl -s http://localhost:48080/actuator/health
```

Kỳ vọng: `{"status":"UP"}`.

```bash
curl -s -X POST http://localhost:48080/admin-api/system/auth/login \
  -H "Content-Type: application/json" \
  -H "tenant-id: 1" \
  -d '{"username":"admin","password":"admin123"}'
```

Kỳ vọng: `"code":0` và có `accessToken`. Local đã tắt captcha (`yudao.captcha.enable: false`) nên không cần mã xác nhận.

Mở Swagger:

- <http://localhost:48080/swagger-ui>
- <http://localhost:48080/doc.html>

### Tiêu chí done

- [ ] `java -version` ra Java 8.
- [ ] Import đủ 2 file SQL, không lỗi.
- [ ] `mvn clean install -DskipTests` thành công.
- [ ] App khởi động, không lỗi `QRTZ_`.
- [ ] `/actuator/health` trả `UP`.
- [ ] Login `admin/admin123` trả `accessToken`.
- [ ] Swagger mở được.

---

## Phase 1 — Thu gọn về platform lõi (1–2 ngày)

### Mục tiêu

Chỉ giữ `yudao-module-system` + `yudao-module-infra`. Bỏ mọi module nghiệp vụ khỏi build, menu và seed data. Không xoá source, để còn sync upstream.

### Việc làm

#### 1.1. Chốt danh sách module

```bash
grep -n "<module>" pom.xml | grep -v "<!--"
grep -n "yudao-module-" yudao-server/pom.xml | grep -v "<!--"
ls -d yudao-module-* | wc -l
```

Tại commit tham chiếu:

- Có **19 thư mục** `yudao-module-*`.
- Giữ **2**: `yudao-module-system`, `yudao-module-infra`.
- Còn **17 thư mục** cần tắt.
- `pom.xml` gốc đang có **17 dòng** `<module>` bị comment.

> Một số tài liệu ghi 18 module. Con số thay đổi theo upstream và cách đếm module con (`yudao-module-mall`, `yudao-module-iot`). Luôn đếm lại bằng lệnh trên; đừng tin số trong tài liệu.

#### 1.2. Giữ `system` + `infra`, tắt phần còn lại

Trên nhánh này, cả 2 file **đã đúng trạng thái**. Việc cần làm là xác nhận, không phải xoá:

- `pom.xml` gốc: chỉ mở `<module>` cho `yudao-dependencies`, `yudao-framework`, `yudao-server`, `yudao-module-system`, `yudao-module-infra`.
- `yudao-server/pom.xml`: chỉ giữ dependency `yudao-module-system` và `yudao-module-infra`.

Nếu fork đã bật thêm module, **comment lại**, không xoá thư mục. Xoá thư mục sẽ làm conflict khi sync upstream.

`yudao-module-system` đã phụ thuộc `yudao-module-infra`, nên giữ cả 2 là bắt buộc.

#### 1.3. Dọn menu và permission seed

Không sửa `sql/mysql/ruoyi-vue-pro.sql` (file upstream). Tạo script riêng, ví dụ `sql/fork/cleanup-core.sql`, chạy sau khi import.

Bước 1 — xem menu gốc:

```sql
SELECT id, name, path
FROM system_menu
WHERE parent_id = 0
ORDER BY sort;
```

Tại commit tham chiếu, các ID gốc của module nghiệp vụ là:

```text
114 pay, 148 bpm, 207 report, 272 mp, 373 member,
449 mall, 480 crm, 597 erp, 791 ai, 860 iot,
959 mes, 1348 wms, 1418 im, 1476 hrm, 1637 cms,
1894 fms, 8000 pms, 8200 oa, 12940 eam
```

Ngoài ra có 3 menu link tài liệu ngoài: `194`, `347`, `348`. Giữ hay bỏ đều được.

> Seed SQL có cả menu của module không nằm trong repo (`cms`, `eam`). Vì vậy phải dọn theo **menu ID**, không dọn theo danh sách thư mục module.

Bước 2 — gom cả cây menu con (MySQL 5.7 và 8.0):

```sql
DROP TABLE IF EXISTS tmp_menu_del;
CREATE TABLE tmp_menu_del (id BIGINT PRIMARY KEY);

INSERT INTO tmp_menu_del (id) VALUES
  (114), (148), (207), (272), (373),
  (449), (480), (597), (791), (860),
  (959), (1348), (1418), (1476), (1637),
  (1894), (8000), (8200), (12940);

-- Lặp câu này đến khi "affected rows = 0"
INSERT IGNORE INTO tmp_menu_del (id)
SELECT m.id
FROM system_menu m
JOIN tmp_menu_del t ON m.parent_id = t.id;

SELECT COUNT(*) FROM tmp_menu_del;  -- kiểm tra số menu sẽ xoá
```

Bước 3 — xoá link role trước, rồi xoá menu:

```sql
DELETE FROM system_role_menu
WHERE menu_id IN (SELECT id FROM tmp_menu_del);

DELETE FROM system_menu
WHERE id IN (SELECT id FROM tmp_menu_del);

DROP TABLE tmp_menu_del;
```

Kiểm tra không còn link mồ côi:

```sql
SELECT COUNT(*) AS orphan
FROM system_role_menu rm
LEFT JOIN system_menu m ON m.id = rm.menu_id
WHERE m.id IS NULL;
```

Kỳ vọng: `0`.

Kiểm tra thêm `system_tenant_package.menu_ids`. Đây là cột `text` chứa JSON array, không có FK, nên vẫn giữ ID menu đã xoá:

```sql
SELECT id, name, menu_ids
FROM system_tenant_package
WHERE deleted = 0;
```

Mở UI **Tenant Package**, chọn lại menu `system` + `infra`, lưu. Nếu không làm, gói tenant vẫn tham chiếu menu không còn tồn tại.

#### 1.4. Dọn dict seed

Liệt kê toàn bộ dict type:

```sql
SELECT type, COUNT(*) AS n
FROM system_dict_data
GROUP BY type
ORDER BY type;
```

Tại commit tham chiếu có **385** dict type. Giữ tối thiểu:

- `system_*`: user, role, login, sms, mail, notice…
- `infra_*`: job, config, codegen, file, api log…
- `common_status`, `user_type`.

Các type còn lại chỉ xoá khi chắc chắn không còn dùng:

```bash
grep -rn "bpm_process_instance_status" yudao-module-system yudao-module-infra
```

Nếu không có kết quả, và frontend cũng đã bỏ trang tương ứng, mới xoá:

```sql
DELETE FROM system_dict_data WHERE dict_type = 'bpm_process_instance_status';
DELETE FROM system_dict_type WHERE type = 'bpm_process_instance_status';
```

Không xoá hàng loạt theo tiền tố nếu chưa grep. Một số type dùng chung (`date_interval`, `terminal`, `merchant_type`).

#### 1.5. Xoá cache và build lại

Sau khi sửa menu trực tiếp trong DB, cache quyền có thể còn dữ liệu cũ:

```bash
redis-cli -n 0 --scan --pattern '*permission_menu_ids*'
```

Cách nhanh trên môi trường dev:

```bash
redis-cli -n 0 FLUSHDB
```

> `FLUSHDB` xoá toàn bộ Redis DB 0. Không chạy trên production.

Khởi động lại app, sau đó kiểm tra Swagger:

```bash
curl -s http://localhost:48080/v3/api-docs \
  | grep -o '"/admin-api/[^"]*"' \
  | cut -d/ -f3 \
  | sort -u
```

Kỳ vọng chỉ còn:

```text
infra
system
```

### Tiêu chí done

- [ ] `pom.xml` gốc và `yudao-server/pom.xml` chỉ bật `system` + `infra`.
- [ ] `mvn clean install -DskipTests` thành công.
- [ ] App khởi động, login được.
- [ ] Swagger chỉ còn API `/admin-api/system/**` và `/admin-api/infra/**`.
- [ ] Menu admin chỉ còn `system` + `infra`.
- [ ] `system_role_menu` không còn link mồ côi.
- [ ] Script dọn menu/dict nằm ở `sql/fork/`, không sửa file SQL upstream.
- [ ] Không xoá thư mục module nào.

---

## Phase 2 — Hardening multi-tenant (2–3 ngày)

### Mục tiêu

Chứng minh tenant A **không** đọc/ghi được dữ liệu tenant B. Tìm và đóng các đường vòng qua interceptor.

### Việc làm

#### 2.1. Xác nhận tenant đang bật

Trên nhánh này, `yudao.tenant.enable` **đã là `true`** trong `application.yaml`. Không cần bật lại.

```bash
grep -n "tenant:" -A 12 yudao-server/src/main/resources/application.yaml
```

Kỳ vọng:

```yaml
yudao:
  tenant:
    enable: true
```

Nếu fork đã tắt, bật lại và khởi động lại app.

#### 2.2. Viết integration test cách ly tenant

Dùng **full-context test**, không dùng `BaseDbUnitTest`: `TenantDatabaseInterceptor` chỉ hoạt động khi MyBatis-Plus và tenant auto-configuration được nạp đầy đủ. Đặt test trong `yudao-server/src/test`, dùng profile `integration` trỏ DB/Redis riêng.

Thêm dependency test vào `yudao-server/pom.xml`:

```xml
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

Test mẫu:

```java
@SpringBootTest
@ActiveProfiles("integration")
@Transactional
@EnabledIfSystemProperty(named = "integration", matches = "true")
class TenantIsolationIntegrationTest {

    @Resource
    private AdminUserService userService;

    @Test
    void tenantA_khongThayUserTenantB() {
        String suffix = String.valueOf(System.nanoTime());
        Long idA = TenantUtils.execute(1L, () -> createUser("tenant-a-" + suffix));
        Long idB = TenantUtils.execute(121L, () -> createUser("tenant-b-" + suffix));

        PageResult<AdminUserDO> pageA = TenantUtils.execute(1L,
                () -> userService.getUserPage(new UserPageReqVO()));

        assertTrue(pageA.getList().stream().anyMatch(u -> idA.equals(u.getId())));
        assertTrue(pageA.getList().stream().noneMatch(u -> idB.equals(u.getId())));
    }

    private Long createUser(String username) {
        UserSaveReqVO reqVO = new UserSaveReqVO();
        reqVO.setUsername(username);
        reqVO.setNickname(username);
        reqVO.setPassword("123456");
        return userService.createUser(reqVO);
    }
}
```

Thêm import cho `@SpringBootTest`, `@ActiveProfiles`, `@Transactional`, `@EnabledIfSystemProperty`, `TenantUtils`, `AdminUserService`, `AdminUserDO`, `UserPageReqVO`, `UserSaveReqVO`, `PageResult`.

Tạo `yudao-server/src/test/resources/application-integration.yaml` trỏ MySQL/Redis test riêng. Không trỏ vào DB local thật.

Test này boot cả app nên chậm. Chấp nhận được: đây là test cách ly, không phải unit test. `@EnabledIfSystemProperty` giúp `mvn verify` mặc định bỏ qua; chạy riêng bằng:

```bash
mvn -pl yudao-server test -Dintegration=true \
  -Dtest=TenantIsolationIntegrationTest
```

#### 2.3. Chứng minh test bắt được lỗi

Đây là bước quan trọng nhất, đừng bỏ:

1. Đặt `yudao.tenant.enable: false` trong `application-integration.yaml`.
2. Chạy lại test.
3. Test **phải đỏ** ở `assertTrue(...noneMatch...)` vì `pageA` nhìn thấy user của tenant B.
4. Bật lại `true`, test xanh.

Nếu tắt tenant mà test vẫn xanh, test đang không kiểm tra gì cả.

#### 2.4. Kiểm tra qua HTTP

Lấy token của tenant 1, rồi cố gắng gọi API với `tenant-id: 2`:

```bash
curl -s http://localhost:48080/admin-api/system/dept/list?pageNo=1&pageSize=10 \
  -H "Authorization: Bearer $TOKEN" \
  -H "tenant-id: 2"
```

Kỳ vọng: body có `"code":403` và message `您无权访问该租户的数据`.

Lưu ý: filter ghi JSON trực tiếp, HTTP status có thể vẫn là `200`. Kiểm tra `code` trong body, đừng chỉ nhìn HTTP status.

#### 2.5. Audit điểm rò rỉ

| Điểm | Rủi ro | Cách kiểm tra |
|---|---|---|
| `@Select`, XML mapper | SQL viết tay không được thêm `tenant_id` | `grep -rn "@Select" yudao-module-system/src/main/java yudao-module-infra/src/main/java` |
| `JdbcTemplate`, native query | Đi vòng qua MyBatis-Plus | `grep -rn "JdbcTemplate\|SqlSession" ...` |
| Cache key | `RedisTemplate` tự set key không có prefix tenant | `grep -rn "opsForValue\|opsForHash\|RedisTemplate" ...` |
| `@Async`, thread pool | Thread pool tái sử dụng có thể giữ tenant cũ | Test một tác vụ async; set/clear hoặc dùng `TenantUtils.execute` |
| Job | Job không có `@TenantJob` chỉ chạy một context | `grep -rn "implements JobHandler" ...` |
| Export Excel | Export query thẳng, không qua mapper | Export ở 2 tenant, kiểm tra file |
| MQ | Message thiếu tenant header | Kiểm tra producer/consumer, test 2 tenant |
| `@TenantIgnore` | Cố ý bỏ qua tenant | `grep -rn "@TenantIgnore" ...`; review từng chỗ |
| `ignore-tables` | Thêm bảng tenant vào đây = tắt cách ly | Review file config |

Cơ chế cần nhớ:

| DO | Interceptor xử lý |
|---|---|
| `extends TenantBaseDO` | Tự thêm `tenant_id` |
| `extends BaseDO` + `@TenantIgnore` | Bỏ qua tenant (bảng global) |
| `extends BaseDO` **không** có `@TenantIgnore` | Vẫn bị thêm `tenant_id` → dễ lỗi SQL nếu bảng không có cột |
| Không map DO | Bỏ qua |

Vì vậy: bảng global phải có `@TenantIgnore`; bảng tenant phải `extends TenantBaseDO`.

#### 2.6. Chốt `ignore-tables` và `ignore-caches`

```yaml
yudao:
  tenant:
    enable: true
    ignore-tables: []          # chỉ bảng global thật sự
    ignore-caches:
      - user_role_ids
      - permission_menu_ids
      - oauth_client
```

- `ignore-tables` đang rỗng vì các bảng global dùng `@TenantIgnore`. Không thêm bảng tenant vào đây để "cho hết lỗi".
- `ignore-caches` chỉ dành cho cache global. Cache của module mới mặc định đã được prefix tenant; không thêm nếu không có lý do.
- Danh sách hiện tại còn cache của module đã tắt (`iot:device`, `iot:thing_model_list`, mail/sms). Có thể bỏ để gọn.

### Tiêu chí done

- [ ] Test `tenantA_khongThayDuLieuTenantB` xanh.
- [ ] Tắt `yudao.tenant.enable` → test đỏ (đã kiểm tra thủ công).
- [ ] Gọi API với token tenant A + `tenant-id` tenant B → `code:403`.
- [ ] Đã grep và review toàn bộ `@Select`, `JdbcTemplate`, `RedisTemplate`, `@TenantIgnore`, `JobHandler`.
- [ ] `ignore-tables` không chứa bảng nghiệp vụ.
- [ ] `ignore-caches` chỉ chứa cache global.
- [ ] Test chạy trong CI (Phase 4), không chỉ chạy local.

---

## Phase 3 — Tạo module nghiệp vụ đầu tiên (2–3 ngày)

### Mục tiêu

Tạo một module mới đúng chuẩn, chứng minh quy trình lặp lại được. Chọn một domain nhỏ, ít quan hệ.

Ví dụ xuyên suốt: module `demo`, entity `Project`, bảng `demo_project`.

### Việc làm

#### 3.1. Chọn đường

| Cách | Dùng khi | Ghi chú |
|---|---|---|
| Codegen | CRUD chuẩn, 1 bảng, ít logic | Nhanh, nhưng phải sửa tay phần tenant |
| Copy skeleton | Cần kiểm soát cấu trúc, nhiều logic | Copy layout từ `system`, pom tối giản từ `erp` |

Không copy nguyên `yudao-module-system` nếu không cần: module này rất lớn và kéo nhiều dependency.

#### 3.2. Tạo module

```bash
mkdir -p yudao-module-demo/src/main/java/cn/iocoder/yudao/module/demo
mkdir -p yudao-module-demo/src/test/java/cn/iocoder/yudao/module/demo
```

`yudao-module-demo/pom.xml` tối giản:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>cn.iocoder.boot</groupId>
        <artifactId>yudao</artifactId>
        <version>${revision}</version>
    </parent>

    <artifactId>yudao-module-demo</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>cn.iocoder.boot</groupId>
            <artifactId>yudao-module-system</artifactId>
            <version>${revision}</version>
        </dependency>
        <dependency>
            <groupId>cn.iocoder.boot</groupId>
            <artifactId>yudao-spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.iocoder.boot</groupId>
            <artifactId>yudao-spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.iocoder.boot</groupId>
            <artifactId>yudao-spring-boot-starter-mybatis</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.iocoder.boot</groupId>
            <artifactId>yudao-spring-boot-starter-biz-tenant</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.iocoder.boot</groupId>
            <artifactId>yudao-spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

Thêm `redis` nếu dùng cache, `excel` nếu export, `job` nếu có job.

#### 3.3. Đăng ký module trong 2 pom

`pom.xml` gốc, thêm vào cuối `<modules>`:

```xml
<module>yudao-module-demo</module>
```

`yudao-server/pom.xml`, thêm dependency:

```xml
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-module-demo</artifactId>
    <version>${revision}</version>
</dependency>
```

Chỉ thêm dòng, không format lại file. Đây là 2 file sẽ conflict khi sync upstream; càng ít thay đổi càng tốt.

#### 3.4. Tạo bảng và code

Bảng phải có `tenant_id` nếu là dữ liệu tenant:

```sql
CREATE TABLE `demo_project` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `status` tinyint NOT NULL DEFAULT 0,
  `tenant_id` bigint NOT NULL DEFAULT 0,
  `creator` varchar(64) DEFAULT '',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updater` varchar(64) DEFAULT '',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `deleted` bit(1) NOT NULL DEFAULT b'0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='项目';
```

Quy ước: `<ten>` là tên module, lowercase, không dấu, không gạch dưới. Package gốc luôn là `cn.iocoder.yudao.module.<ten>`.

Cấu trúc package:

```text
cn.iocoder.yudao.module.demo/
├── api/                    # DTO/interface cho module khác gọi
├── controller/admin/project/
├── convert/
├── dal/dataobject/project/
├── dal/mysql/project/
├── enums/
├── framework/              # cấu hình riêng của module, nếu có
├── job/                    # nếu có job
├── mq/                     # nếu có MQ
└── service/
```

Không cần tạo đủ thư mục ngay. Chỉ tạo khi có code.

**Lưu ý codegen:** template `do.vm` sinh `extends BaseDO`. Nếu bảng có `tenant_id`, sửa thành:

```java
public class DemoProjectDO extends TenantBaseDO {
```

Codegen đã loại field `tenantId` khỏi DO, nên không bị trùng.

Nếu bảng global (không có `tenant_id`), giữ `BaseDO` và thêm `@TenantIgnore`.

#### 3.5. Menu và permission

Codegen sinh sẵn:

- 1 menu cha: `demo/project/index`
- 5 button: `demo:project:query`, `:create`, `:update`, `:delete`, `:export`

Chạy SQL menu do codegen sinh, hoặc tạo menu qua UI. Giữ bản fork trong `sql/fork/demo-menu.sql`.

Controller dùng permission:

```java
@PreAuthorize("@ss.hasPermission('demo:project:query')")
```

Kiểm tra menu hiển thị cho tenant admin, và user không có quyền bị chặn.

#### 3.6. Test

Copy pattern từ `yudao-module-system`:

```text
yudao-module-demo/src/test/resources/
├── application-unit-test.yaml
└── sql/
    ├── create_tables.sql
    └── clean.sql
```

- `create_tables.sql`: lấy DDL H2 do codegen sinh (`codegen/sql/h2.vm`).
- `clean.sql`: `DELETE FROM "demo_project";`
- Test service: `DemoProjectServiceImplTest extends BaseDbUnitTest`.

Chạy:

```bash
mvn -pl yudao-module-demo -am clean install -DskipTests
mvn -pl yudao-module-demo test
```

Sau đó chạy server và kiểm tra Swagger có tag `demo`.

### Tiêu chí done

- [ ] Có 1 bảng `demo_project` với `tenant_id`.
- [ ] DO `extends TenantBaseDO` (hoặc `@TenantIgnore` nếu bảng global).
- [ ] Có 1 CRUD: create/update/delete/get/page.
- [ ] Có ít nhất 1 permission, gắn `@PreAuthorize`.
- [ ] Có 1 menu và 5 button, hiển thị đúng.
- [ ] Có 1 test service chạy xanh.
- [ ] Module có trong 2 pom; server khởi động; Swagger có tag mới.
- [ ] `system`/`infra` không phụ thuộc ngược vào `demo`.

---

## Phase 4 — Chuẩn hoá vận hành (3–5 ngày)

### Mục tiêu

Đưa platform lên môi trường thật: build tự động, image cố định, log truy vết được, backup có restore, deploy/rollback một lệnh.

### Việc làm

#### 4.1. CI: build + test + lint

Repo hiện có `.github/workflows/maven.yml`:

- Matrix Java `8`, `11`, `17`.
- Chạy `mvn -B package -Dmaven.test.skip=true`.

Với fork JDK 8, sửa thành:

- Chỉ Java `8`.
- Chạy `mvn -B -ntp verify` (có test).
- Thêm lint.

Ví dụ workflow fork (file mới `.github/workflows/fork-ci.yml` để tránh conflict):

```yaml
name: fork-ci

on:
  push:
    branches: [master]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '8'
          cache: maven
      - run: mvn -B -ntp verify
```

Lint: repo chưa có sẵn. Chọn **một**:

- Spotless + google-java-format.
- Checkstyle.

Nếu dùng Spotless, bật `ratchetFrom` để chỉ check file thay đổi, tránh format toàn bộ code upstream.

`mvn verify` mặc định bỏ qua integration test. Thêm job riêng, có MySQL/Redis:

```yaml
  integration:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8
        env:
          MYSQL_ROOT_PASSWORD: 123456
          MYSQL_DATABASE: ruoyi-vue-pro-test
        ports: ['3306:3306']
        options: >-
          --health-cmd="mysqladmin ping -h 127.0.0.1 -p123456"
          --health-interval=10s --health-timeout=5s --health-retries=10
      redis:
        image: redis:6-alpine
        ports: ['6379:6379']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '8'
          cache: maven
      - run: sudo apt-get update && sudo apt-get install -y mysql-client
      - run: mysql -h 127.0.0.1 -u root -p123456 ruoyi-vue-pro-test < sql/mysql/ruoyi-vue-pro.sql
      - run: mysql -h 127.0.0.1 -u root -p123456 ruoyi-vue-pro-test < sql/mysql/quartz.sql
      - run: mvn -B -ntp test -Dintegration=true -Dtest=TenantIsolationIntegrationTest
```

Job này dùng `application-integration.yaml` trỏ `127.0.0.1:3306/ruoyi-vue-pro-test` và Redis `127.0.0.1:6379`.

Tiêu chí CI:

- PR đỏ khi test fail.
- PR đỏ khi lint fail.
- Job integration xanh, test tenant thật sự chạy (không bị skip).
- Không dùng `-DskipTests` trên CI.

#### 4.2. Docker image cho `yudao-server`

Dockerfile có sẵn: `yudao-server/Dockerfile`, dùng `eclipse-temurin:8-jre`, copy `target/yudao-server.jar`.

```bash
mvn -B clean package -DskipTests

TAG=2026.10.21-$(git rev-parse --short HEAD)
docker build -f yudao-server/Dockerfile \
  -t registry.example.com/platform/yudao-server:$TAG \
  yudao-server
docker push registry.example.com/platform/yudao-server:$TAG
```

Thêm `.dockerignore` trong `yudao-server` để giảm context:

```text
src
target/classes
target/test-classes
```

Chạy thử:

```bash
docker run -d --name yudao-server -p 48080:48080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_URL='jdbc:mysql://mysql:3306/platform?useSSL=false&serverTimezone=Asia/Ho_Chi_Minh' \
  -e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_USERNAME=platform \
  -e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_PASSWORD='***' \
  -e SPRING_REDIS_HOST=redis \
  -e SPRING_REDIS_PASSWORD='***' \
  registry.example.com/platform/yudao-server:$TAG
```

Không truyền secret qua lịch sử shell. Dùng Docker secret, file env được bảo vệ, hoặc secret manager.

`script/docker/docker-compose.yml` có sẵn nhưng `build.context` trỏ `./yudao-server/` tương đối với `script/docker`; kiểm tra lại trước khi dùng. Nếu thiếu Dockerfile, dùng lệnh `docker build` ở trên.

#### 4.3. Logging + correlation ID + actuator

Hiện tại `logback-spring.xml` chỉ có console + file, chưa có request ID.

Thêm filter (file mới, ví dụ `cn.iocoder.yudao.fork.web.RequestIdFilter`):

```java
String requestId = request.getHeader("X-Request-Id");
if (StrUtil.isBlank(requestId)) {
    requestId = UUID.randomUUID().toString();
}
MDC.put("traceId", requestId);
response.setHeader("X-Request-Id", requestId);
try {
    chain.doFilter(request, response);
} finally {
    MDC.remove("traceId");
}
```

Sửa pattern trong `logback-spring.xml`:

```text
%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{traceId}] %-5level %logger{50}:%L - %msg%n
```

Actuator:

- Local đang mở `*` (`management.endpoints.web.exposure.include: '*'`).
- Production chỉ mở `health,info,metrics`.
- Không mở `env`, `heapdump`, `threaddump` ra internet.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

Kiểm tra:

```bash
curl -fsS http://localhost:48080/actuator/health
```

Tiêu chí: một request có thể lần theo `X-Request-Id` trong toàn bộ log; `/actuator/health` trả `UP`; endpoint nhạy cảm không truy cập được từ ngoài.

#### 4.4. Backup MySQL và Redis

MySQL:

```bash
mysqldump --single-transaction --quick --routines --triggers \
  -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" \
  | gzip > "/backup/mysql-$(date +%F-%H%M).sql.gz"
```

Redis:

```bash
redis-cli -h "$REDIS_HOST" -a "$REDIS_PASS" \
  --rdb "/backup/redis-$(date +%F-%H%M).rdb"
```

Hoặc `BGSAVE` rồi copy `dump.rdb` từ volume.

Restore thử:

```bash
gunzip -c /backup/mysql-2026-10-21-0200.sql.gz \
  | mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME_SCRATCH"
```

- `quartz.sql` và bảng `QRTZ_*` nằm trong MySQL, đã được backup cùng.
- Redis restore: dừng Redis, thay `dump.rdb`, khởi động lại. Chỉ restore khi chấp nhận mất dữ liệu sau thời điểm backup.
- Chưa restore thử = chưa có backup.

Tiêu chí: restore vào DB scratch thành công, app trỏ vào DB scratch khởi động được; ghi rõ RPO/RTO mục tiêu.

#### 4.5. Cấu hình production

Tạo `application-prod.yaml` (file mới, không sửa file upstream):

```yaml
server:
  port: 48080

springdoc:
  api-docs:
    enabled: false
  swagger-ui:
    enabled: false

knife4j:
  enable: false

yudao:
  demo: false
  security:
    mock-enable: false
  tenant:
    enable: true

spring:
  redis:
    password: ${REDIS_PASSWORD}

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

Checklist bắt buộc:

| Cấu hình | Giá trị production | Lý do |
|---|---|---|
| `springdoc.api-docs.enabled` | `false` | Không lộ API docs |
| `springdoc.swagger-ui.enabled` | `false` | Không lộ Swagger UI |
| `knife4j.enable` | `false` | Không lộ doc.html |
| `yudao.demo` | `false` | Tắt DemoFilter |
| `yudao.security.mock-enable` | `false` | Chặn mock login |
| `yudao.tenant.enable` | `true` | Giữ cách ly tenant |
| `spring.redis.password` | `${REDIS_PASSWORD}` | Redis không mở public |
| DB user/password | user riêng, không `root/123456` | Giảm thiệt hại khi lộ |
| `mybatis-plus.encryptor.password` | `${...}` | Giá trị đang hardcode trong repo coi như đã lộ |
| `spring.datasource.druid.stat-view-servlet.enabled` | `false` | Không lộ trang SQL/Druid |
| `management.endpoints.web.exposure.include` | `health,info,metrics` | Giảm bề mặt tấn công |
| `yudao.access-log.enable` | `true` | Có audit trail |
| Admin password | Đổi khỏi `admin123` | Seed công khai |
| Tenant/user demo | Xoá hoặc disable | Seed có tenant `121`, `122` |

Hai lỗi production hay gặp:

- `server.port` chỉ có trong `application-local.yaml`. Không set trong prod thì app bind `8080`, không phải `48080`. Giữ `server.port: 48080` trong `application-prod.yaml` hoặc đặt `SERVER_PORT=48080`.
- Nếu bật job, copy khối `spring.quartz` từ `application-dev.yaml` (JDBC JobStore, clustered) sang `application-prod.yaml`. Không copy thì Quartz có thể chạy memory store, mất job khi restart.

Đổi mật khẩu admin qua UI, hoặc API:

```http
PUT /admin-api/system/user/update-password
{ "id": 1, "password": "<mật-khẩu-mới>" }
```

Lưu ý `mybatis-plus.encryptor.password`: đổi trước khi lưu dữ liệu mã hoá. Nếu đã có dữ liệu, phải re-encrypt, không đổi ngẫu nhiên.

#### 4.6. Deploy và rollback một lệnh

Tạo `deploy/docker-compose.prod.yml` (fork-only), dùng image tag cố định:

```yaml
services:
  server:
    image: registry.example.com/platform/yudao-server:${TAG}
    ports:
      - "48080:48080"
    env_file:
      - .env
    restart: unless-stopped
```

Deploy:

```bash
TAG=2026.10.21-abc123 \
  docker compose -f deploy/docker-compose.prod.yml up -d
```

Rollback:

```bash
TAG=2026.10.20-def456 \
  docker compose -f deploy/docker-compose.prod.yml up -d --no-deps server
```

Quy trình:

1. Backup DB trước khi deploy.
2. Chạy migration (nếu có) theo hướng **backward-compatible**.
3. Deploy image mới.
4. Health check:
   ```bash
   curl -fsS http://localhost:48080/actuator/health | grep '"status":"UP"'
   ```
5. Nếu health fail, rollback image cũ.
6. Không rollback DB tự động; dùng backup hoặc migration `down` đã chuẩn bị.

Tiêu chí: deploy 1 lệnh, rollback 1 lệnh, đã diễn tập trên staging; image tag là commit SHA, không dùng `latest`.

### Tiêu chí done

- [ ] CI chạy `mvn verify`, có lint, PR fail khi test/lint fail.
- [ ] Có image `yudao-server` tag theo commit SHA, push lên registry.
- [ ] Log có `X-Request-Id` xuyên suốt; actuator chỉ mở endpoint cần thiết.
- [ ] Backup MySQL + Redis chạy tự động; đã restore thử.
- [ ] Production tắt Swagger, demo, mock login; đổi admin password; bật Redis password.
- [ ] Deploy 1 lệnh, rollback 1 lệnh, health check tự động.
- [ ] Không còn secret hardcode trong file commit.

---

## Phase 5 — Giữ sync upstream (định kỳ)

### Mục tiêu

Nhận bugfix/feature từ `YunaiV/ruoyi-vue-pro` mà không phá phần fork.

### Việc làm

#### 5.1. Thêm remote

```bash
git remote -v

git remote add upstream https://github.com/YunaiV/ruoyi-vue-pro.git
# nếu đã có:
# git remote set-url upstream https://github.com/YunaiV/ruoyi-vue-pro.git
```

Kỳ vọng:

```text
origin    https://github.com/chuanghiduoc/ruoyi-vue-pro.git
upstream  https://github.com/YunaiV/ruoyi-vue-pro.git
```

#### 5.2. Sync

Cách CLI (khuyến nghị khi fork có commit riêng):

```bash
git status --short
git tag fork-before-sync-$(date +%Y%m%d-%H%M)
git switch -c sync/upstream-$(date +%Y%m%d)

git fetch upstream --prune --tags
git merge --no-commit --no-ff upstream/master
```

Kiểm tra, resolve conflict, rồi:

```bash
git commit -m "merge: sync upstream $(date +%Y-%m-%d)"
mvn -B -ntp verify
```

Hủy khi conflict quá lớn:

```bash
git merge --abort
git switch master
```

Cách GitHub UI:

1. Mở fork trên GitHub.
2. Bấm **Sync fork** → **Update branch**.
3. Không bấm **Discard commits** nếu fork đang có commit riêng.

Tần suất: 2 tuần/lần, và trước khi bắt đầu feature lớn.

#### 5.3. Quy tắc file

Nguyên tắc: **chỉ thêm file mới, không sửa code upstream**. Áp dụng thực tế như sau:

| Đường dẫn | Được sửa? | Cách làm |
|---|---|---|
| `yudao-module-*` của upstream | Không | Thêm class mới, hoặc tạo module mới |
| `yudao-framework` | Hạn chế | Thêm class mới trong package riêng, hoặc gửi PR upstream |
| `pom.xml` gốc | Chỉ thêm dòng | Thêm module mới, không format lại |
| `yudao-server/pom.xml` | Chỉ thêm dòng | Thêm dependency module mới |
| `application.yaml` | Không | Tạo `application-fork.yaml`, chạy profile `local,fork` |
| `application-local.yaml` | Chỉ setup local | Override lâu dài chuyển sang `application-fork.yaml` |
| `sql/mysql/*.sql` | Không | Tạo `sql/fork/*.sql` |
| `docs/vi`, `deploy`, `script/fork` | Có | File fork-only, không conflict |
| `.github/workflows` | Hạn chế | Thêm workflow mới thay vì sửa workflow upstream |

Ví dụ chạy với config fork:

```bash
mvn -pl yudao-server spring-boot:run \
  -Dspring-boot.run.profiles=local,fork
```

`application-fork.yaml` đè lên `application-local.yaml`, không cần sửa file upstream.

#### 5.4. Xử lý conflict

- Conflict ở `pom.xml`: giữ cả module upstream và module fork; không đổi thứ tự, không format.
- Conflict ở `application*.yaml`: lấy upstream, chuyển phần fork sang `application-fork.yaml`.
- Conflict ở SQL: lấy upstream, chuyển phần fork sang `sql/fork/`.
- Conflict ở code module upstream: lấy upstream, chuyển logic fork sang class/module mới.
- Bật `git rerere` để ghi nhớ cách resolve:
  ```bash
  git config rerere.enabled true
  ```

### Tiêu chí done

- [ ] `git merge upstream/master` không conflict, hoặc chỉ conflict ở 2 file `pom.xml` và đã resolve.
- [ ] Sau merge, `mvn -B -ntp verify` xanh.
- [ ] App khởi động, login được, Swagger đúng.
- [ ] `git diff --stat upstream/master` chỉ hiện file fork + dòng module thêm vào pom.
- [ ] Có tag trước mỗi lần sync để rollback.

---

## 6. Những gì KHÔNG nên làm

| Không nên | Vì sao | Làm thay thế |
|---|---|---|
| Sửa trực tiếp code trong module của upstream | Mỗi lần sync đều conflict; không phân biệt được bug fork hay bug upstream | Thêm class/module mới, hoặc gửi PR upstream |
| Đổi tên package gốc `cn.iocoder.yudao` khi còn sync | Codegen, `@MapperScan`, config, SQL, tài liệu đều phụ thuộc; sync gần như không thể | Giữ package gốc, đặt tên module mới |
| Bật tất cả module ngay từ đầu | Build chậm, nhiều bảng/menu, bề mặt tấn công lớn, khó debug | Giữ `system` + `infra`, bật theo nhu cầu |
| Dùng codegen cho domain logic phức tạp | Code sinh ra sửa tay sẽ mất khả năng regenerate; invariant/luồng tiền không được kiểm soát | Codegen cho CRUD, viết tay domain logic |
| Chạy production với `admin/admin123` | Mật khẩu seed công khai, ai cũng biết | Đổi ngay, xoá user/tenant demo |
| Thêm bảng vào `ignore-tables` để hết lỗi | Tắt cách ly tenant một cách âm thầm | Sửa DO sang `TenantBaseDO`, thêm `tenant_id` |
| Commit secret (DB password, Redis password, encryptor key) | Fork có thể public; secret nằm trong lịch sử git | Dùng env/secret manager, rotate |
| Deploy tag `latest` | Không biết đang chạy bản nào, không rollback được | Tag commit SHA, giữ N-1 image |
| Sửa `sql/mysql/ruoyi-vue-pro.sql` | Conflict mỗi lần upstream đổi schema/seed | `sql/fork/*.sql` chạy sau import |
| Chạy CI với `-DskipTests` | Test đỏ không chặn merge | `mvn verify`, tách job nếu cần |

---

## 7. Rủi ro và việc cần chốt

- **Menu ID thay đổi theo upstream.** Script dọn menu phải chạy lại `SELECT` trước khi `DELETE`; đừng dùng ID mù.
- **Test tenant chỉ chứng minh đường MyBatis.** Raw SQL, cache tự set, MQ vẫn phải audit tay.
- **`@TenantJob` chạy job cho mọi tenant.** Job phải idempotent; retry có thể chạy lại tenant đã thành công.
- **CI chưa có lint.** Chọn Spotless hoặc Checkstyle trước khi mở PR cho người khác.
- **Backup chưa restore thử thì chưa tính là backup.** Đặt lịch diễn tập restore.
- **Chốt trước Phase 3:** domain đầu tiên, mô hình tenant (shared DB hay DB riêng), nhu cầu MQ, nơi lưu file, có cần app-api cho C-end không.

---

## 8. Thứ tự kiểm tra nhanh sau mỗi phase

```bash
# 1. Build + test
mvn -B -ntp verify

# 2. Chạy app
mvn -pl yudao-server spring-boot:run

# 3. Health
curl -fsS http://localhost:48080/actuator/health

# 4. Login
curl -s -X POST http://localhost:48080/admin-api/system/auth/login \
  -H "Content-Type: application/json" \
  -H "tenant-id: 1" \
  -d '{"username":"admin","password":"admin123"}'

# 5. Swagger chỉ còn system + infra
curl -s http://localhost:48080/v3/api-docs \
  | grep -o '"/admin-api/[^"]*"' \
  | cut -d/ -f3 \
  | sort -u
```

Nếu 5 bước trên xanh, phase hiện tại đã đủ điều kiện chuyển tiếp.
