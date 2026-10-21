# 04 — Quy ước code

> Đọc trước khi thêm module, thêm API hoặc sửa hành vi nghiệp vụ.
> Đối chiếu trực tiếp source `yudao-module-system` trong repo này.

Mục tiêu: code mới **nhất quán**, **dễ merge upstream**, **không rò rỉ dữ liệu tenant**.

## 1. Chính sách ngôn ngữ

Nguyên tắc: **máy đọc = tiếng Anh, người đọc = tiếng Việt.**

| Thành phần | Ngôn ngữ | Khi viết code mới |
|---|---|---|
| Class, interface, package | **EN** | Không dịch, không viết tắt tuỳ tiện |
| Biến, tham số, method | **EN** | `userList`, không `danhSachUser` |
| Bảng, cột, index | **EN** | `system_users`, `dept_id` |
| API path, field JSON, enum value | **EN** | Client đang phụ thuộc |
| Log message | **EN** | Dễ grep và alert |
| Comment, Javadoc | **EN** | File mới viết EN |
| Error message (`ErrorCode`) | **EN** mặc định (code mới) | Xem mục 8 |
| Commit, branch, PR title | **EN** | Conventional Commits |
| `README.vi.md`, `docs/vi/*` | **VI** | File bạn đang đọc |
| UI label | i18n `vi` | Không hardcode trong Vue |
| Seed/demo data | **VI** được | Chỉ dữ liệu demo |

Ba lưu ý thực dụng:

1. **Không dịch comment tiếng Trung của upstream.** Sửa nhỏ thì giữ nguyên để merge không conflict.
2. **Không đổi tên class/field upstream** chỉ để "cho đẹp". Đó là breaking change.
3. **Không hardcode tiếng Việt trong code Java.** Chỗ cần đa ngôn ngữ thì đi qua i18n.

## 2. Cấu trúc package của một module

```
yudao-module-xxx/
└── src/main/java/cn/iocoder/yudao/module/xxx/
    ├── api/                  ← hợp đồng cho module khác
    │   └── user/
    │       ├── AdminUserApi.java
    │       ├── AdminUserApiImpl.java
    │       └── dto/AdminUserRespDTO.java
    ├── controller/
    │   ├── admin/            ← /admin-api/**
    │   └── app/              ← /app-api/**
    ├── convert/              ← MapStruct DO ↔ VO ↔ DTO
    ├── dal/
    │   ├── dataobject/       ← DO
    │   ├── mysql/            ← Mapper
    │   └── redis/            ← (tuỳ chọn) DAO Redis
    ├── enums/                ← enum + ErrorCodeConstants
    ├── framework/            ← cấu hình riêng của module
    ├── job/                  ← Quartz JobHandler
    ├── mq/                   ← producer / consumer / message
    ├── service/              ← business logic
    └── util/                 ← helper nội bộ module
```

| Package | Chứa gì | Bắt buộc? |
|---|---|---|
| `api` | Interface + DTO cho module khác gọi | Chỉ khi module được module khác dùng |
| `controller/admin` | REST cho backend/admin | Bắt buộc nếu có API admin |
| `controller/app` | REST cho C-end | Chỉ khi có app API |
| `convert` | MapStruct mapper | Bắt buộc nếu DO khác VO |
| `dal/dataobject` | DO map 1-1 với bảng | Bắt buộc nếu có DB |
| `dal/mysql` | Mapper MyBatis-Plus | Bắt buộc nếu có DB |
| `enums` | Enum nghiệp vụ + `ErrorCodeConstants` | Bắt buộc nếu có lỗi nghiệp vụ (hầu như module nào cũng có) |
| `framework` | Config, interceptor, custom bean của module | Tuỳ chọn |
| `dal/redis` | DAO Redis của module | Tuỳ chọn |
| `job` | Tác vụ định kỳ | Tuỳ chọn |
| `mq` | Producer/consumer/message | Tuỳ chọn |
| `service` | Interface + Impl | Bắt buộc |
| `util` | Helper chỉ module này dùng | Tuỳ chọn |

Quy tắc phụ:

- Package con đặt **số ít**, theo domain: `dal/dataobject/user`, `service/user`, không `users`.
- `controller/admin` và `controller/app` **không** hardcode `/admin-api` trong `@RequestMapping`. Prefix do `WebProperties` gắn theo package.
- `util/` không được chứa business logic. Nếu bắt đầu có `if` nghiệp vụ, chuyển vào `service/`.
- Không tạo `common/` trong module. Thứ dùng chung toàn hệ thống nằm ở `yudao-framework/yudao-common`.

## 3. Quy tắc đặt tên

### 3.1. DO / VO / DTO / BO

| Loại | Hậu tố | Vị trí | Ví dụ |
|---|---|---|---|
| Data Object | `DO` | `dal/dataobject/<domain>` | `AdminUserDO` |
| Request VO | `ReqVO` | `controller/{admin,app}/<domain>/vo` | `UserSaveReqVO` |
| Response VO | `RespVO` | cùng chỗ | `UserRespVO` |
| Page Request VO | `PageReqVO` | cùng chỗ, `extends PageParam` | `UserPageReqVO` |
| Save Request VO | `SaveReqVO` | dùng chung create + update | `UserSaveReqVO` |
| API DTO | `DTO` | `api/<domain>/dto` | `AdminUserRespDTO` |
| Business Object | `BO` | `service/<domain>/bo` (hiếm) | `CrmCustomerCreateReqBO` |

Quy tắc:

- `ReqVO` là input, `RespVO` là output. Không dùng một class cho cả hai.
- `SaveReqVO` có `id` nullable: có `id` = update, không có = create.
- `PageReqVO` luôn `extends PageParam`, không tự khai báo `pageNo/pageSize`.
- VO/DTO **không** extend DO và không chứa field `deleted`, `creator`, `updater`.
- DTO trong `api/` chỉ chứa dữ liệu module khác cần. Không trả `DO` ra ngoài.
- BO chỉ dùng khi cần gom nhiều DO/VO cho một nghiệp vụ. Không tạo BO cho có.

### 3.2. Service

```java
public interface AdminUserService {
    Long createUser(@Valid UserSaveReqVO reqVO);
    PageResult<AdminUserDO> getUserPage(UserPageReqVO reqVO);
}

@Service
public class AdminUserServiceImpl implements AdminUserService {
    @Resource
    private AdminUserMapper userMapper;
}
```

- Interface: `XxxService`. Impl: `XxxServiceImpl`. **Không** viết `IXxxService`.
- Method theo hành động: `createXxx`, `updateXxx`, `deleteXxx`, `getXxx`, `getXxxList`, `getXxxPage`.
- Method trả `void`/`Boolean`/`Long`/DO/`PageResult`, **không** trả `CommonResult`. `CommonResult` chỉ ở Controller.
- Service không nhận `HttpServletRequest`, không đọc header trực tiếp.

### 3.3. Controller, Mapper, Convert

| Thành phần | Tên | Ghi chú |
|---|---|---|
| Controller | `XxxController` | `controller/admin/<domain>/XxxController.java` |
| Mapper | `XxxMapper` | `dal/mysql/<domain>/XxxMapper.java`, `extends BaseMapperX<XxxDO>` |
| Convert | `XxxConvert` | `convert/<domain>/XxxConvert.java`, interface + `INSTANCE` |
| Error code | `ErrorCodeConstants` | Mỗi module một file trong `enums/` |

```java
@Mapper
public interface AdminUserMapper extends BaseMapperX<AdminUserDO> {

    default AdminUserDO selectByUsername(String username) {
        return selectOne(AdminUserDO::getUsername, username);
    }
}
```

- Mapper method: `selectById`, `selectByXxx`, `selectListByXxx`, `selectPage`.
- Convert: `@Mapper` (MapStruct) và `XxxConvert INSTANCE = Mappers.getMapper(XxxConvert.class);`.
- Case mapping đơn giản có thể dùng `BeanUtils.toBean(source, Target.class)` thay vì viết Convert.
- Không đặt tên `UserDao`, `UserRepository`, `UserManager` — repo dùng `Mapper` và `Service`.

## 4. Luồng code chuẩn

```
HTTP request
  → Controller        @Valid VO, @PreAuthorize
  → Service           business logic, @Transactional
  → Mapper            MyBatis-Plus
  → MySQL

DO → Convert → RespVO → CommonResult
```

### 4.1. Controller

```java
@Tag(name = "管理后台 - 用户")
@RestController
@RequestMapping("/system/user")
@Validated
public class UserController {

    @Resource
    private AdminUserService userService;

    @PostMapping("/create")
    @Operation(summary = "新增用户")
    @PreAuthorize("@ss.hasPermission('system:user:create')")
    public CommonResult<Long> createUser(@Valid @RequestBody UserSaveReqVO reqVO) {
        return success(userService.createUser(reqVO));
    }
}
```

- Controller **không** chứa business logic, không inject Mapper.
- `@Valid` đặt trước `@RequestBody`; class có `@Validated` để validate `@RequestParam`.
- Response luôn `CommonResult<T>`; dùng `success(...)` static import.
- Danh sách phân trang trả `CommonResult<PageResult<XxxRespVO>>`.

### 4.2. Service

```java
@Service
@Validated
public class UserServiceImpl implements UserService {

    @Resource
    private UserMapper userMapper;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long createUser(@Valid UserSaveReqVO reqVO) {
        // validate nghiệp vụ: username trùng, dept tồn tại...
        UserDO user = UserConvert.INSTANCE.convert(reqVO);
        userMapper.insert(user);
        return user.getId();
    }
}
```

- Ghi nhiều bảng hoặc có side effect → `@Transactional(rollbackFor = Exception.class)`.
- Validate nghiệp vụ đặt trong Service, không đặt trong Controller.
- Không nuốt exception. Không `catch (Exception e) { return null; }`.
- Không gọi API module khác trong vòng lặp lớn. Gom ID rồi gọi batch.

### 4.3. Convert

```java
@Mapper
public interface UserConvert {

    UserConvert INSTANCE = Mappers.getMapper(UserConvert.class);

    UserDO convert(UserSaveReqVO reqVO);

    UserRespVO convert(UserDO user);
}
```

- Convert chỉ map field. Không query DB trong Convert.
- Field cần ghép từ nhiều nguồn → viết `default` method trong Convert.
- Không dùng reflection tự viết để copy field.

## 5. Gọi chéo module

**Chỉ** qua package `api`. Không inject `Service`, `Mapper`, `DO` của module khác.

Interface đặt tên `XxxApi` — ví dụ `UserApi`; trong module system là `AdminUserApi`.

```java
// module-system: api/user/AdminUserApi.java
public interface AdminUserApi {
    AdminUserRespDTO getUser(Long id);
}

// module-system: api/user/AdminUserApiImpl.java
@Service
public class AdminUserApiImpl implements AdminUserApi {

    @Resource
    private AdminUserService userService;

    @Override
    public AdminUserRespDTO getUser(Long id) {
        return BeanUtils.toBean(userService.getUser(id), AdminUserRespDTO.class);
    }
}
```

```java
// module khác: chỉ biết interface + DTO
@Resource
private AdminUserApi adminUserApi;
```

Quy tắc:

1. `api/` chỉ chứa interface, impl và DTO. Không trả `DO`, không trả `PageResult<DO>`.
2. Impl đặt trong `api/<domain>/`, annotate `@Service`, được gọi Service nội bộ của chính module.
3. API là hợp đồng ổn định. Đổi signature = đổi hợp đồng, phải rà soát caller.
4. API dùng cho nội bộ thường thêm `@DataPermission(enable = false)` để không bị lọc mất dữ liệu.
5. Không gọi chéo module bằng HTTP trong monolith. Đó là dấu hiệu tách sai ranh giới.

## 6. Phân quyền

### 6.1. RBAC — `@PreAuthorize`

```java
@GetMapping("/page")
@PreAuthorize("@ss.hasPermission('system:user:query')")
public CommonResult<PageResult<UserRespVO>> getUserPage(@Valid UserPageReqVO reqVO) {
    return success(userService.getUserPage(reqVO));
}
```

Mã permission: `<module>:<entity>:<action>`.

| Action | Dùng cho |
|---|---|
| `query` | Xem danh sách / chi tiết |
| `create` | Tạo mới |
| `update` | Cập nhật |
| `delete` | Xoá |
| `export` | Export Excel |
| `import` | Import Excel |

- Đặt `@PreAuthorize` ở **method Controller**. Thống nhất một chỗ, không rải xuống Service.
- Chuỗi permission phải khớp menu/button trong DB (`system_menu.permission`). Sai một ký tự = 403.
- Không tự kiểm tra `if (loginUser.isAdmin())` trong Service. Dùng permission hoặc `@DataPermission`.
- API công khai (login, captcha, callback) không gắn `@PreAuthorize`, nhưng phải nằm trong danh sách permit của Security.

### 6.2. Row-level — `@DataPermission`

`@DataPermission` mặc định **bật**. Rule `DeptDataPermissionRule` lọc theo `dept_id` / `user_id`.

```java
// Tắt cho API nội bộ: tránh lọc mất dữ liệu khi module khác cần đọc
@DataPermission(enable = false)
public AdminUserRespDTO getUser(Long id) {
    ...
}
```

- Chỉ tắt khi thật sự hiểu hậu quả. Ghi comment lý do ngay trên annotation.
- Muốn giới hạn rule cụ thể: dùng `includeRules` / `excludeRules`.
- Bảng nghiệp vụ cần lọc phải có cột `dept_id` hoặc `user_id`.
- Test cả hai chiều: trưởng phòng A **thấy** dữ liệu A, **không thấy** dữ liệu B.

## 7. Multi-tenant

### 7.1. DO có tenant

```java
@Data
@EqualsAndHashCode(callSuper = true)
@TableName("system_users")
public class AdminUserDO extends TenantBaseDO {
    // tenant_id nằm trong TenantBaseDO
}
```

- DO thuộc tenant → `extends TenantBaseDO`.
- DO global (bảng dùng chung) → `extends BaseDO`.
- Không tự khai báo `private Long tenantId` trong DO. Nó đã có ở base class.
- `TenantDatabaseInterceptor` tự thêm `tenant_id = ?` cho câu query đi qua MyBatis-Plus.

### 7.2. Bỏ qua tenant

```java
@TenantIgnore // chỉ dùng khi thật sự cần
@GetMapping("/callback")
public CommonResult<Boolean> callback(...) { ... }
```

Các case hợp lý:

- Webhook/callback từ bên thứ ba (SMS, payment).
- API captcha, login trước khi biết tenant.
- Danh sách tenant toàn cục.
- Consumer MQ xử lý message toàn cục.

Không dùng `@TenantIgnore` để "cho query chạy được". Nếu query thiếu `tenant_id`, sửa thiết kế bảng.

### 7.3. Context

```java
// Đúng
TenantContextHolder.setTenantId(tenantId);
Long current = TenantContextHolder.getTenantId();

// Chạy logic dưới một tenant khác, tự restore trong finally
TenantUtils.execute(tenantId, () -> doSomething());
```

- **Không** tạo `ThreadLocal<Long>` riêng. Dùng `TenantContextHolder`.
- `TenantContextHolder` dùng `TransmittableThreadLocal`, nhưng chỉ phát huy tác dụng khi task được wrap.
- Ưu tiên `TenantUtils.execute(...)` vì nó restore cả `tenantId` và `ignore` trong `finally`.

### 7.4. Điểm dễ rò rỉ dữ liệu

| Tình huống | Vấn đề | Cách xử lý |
|---|---|---|
| `@Async`, `CompletableFuture` | Mất context nếu không wrap | Wrap bằng `TtlRunnable` / `TtlExecutors`, hoặc dùng async config sẵn có |
| Thread pool tự tạo | Thread tái sử dụng, context cũ | Không tự tạo pool; nếu buộc phải tạo, truyền `tenantId` tường minh |
| Cache key | Key trùng giữa tenant | Dùng `@Cacheable` qua `TenantRedisCacheManager`, hoặc tự prefix `tenantId` |
| `RedisTemplate` thủ công | Không tự prefix | Tự thêm `tenantId` vào key |
| `@Select` / XML mapper | Interceptor không sửa SQL viết tay | Tự thêm `tenant_id = #{tenantId}` |
| Native query / store procedure | Không đi qua MyBatis-Plus | Tự lọc tenant trong SQL |
| Export Excel / báo cáo | Query thẳng, dễ quên | Kiểm tra lại query nguồn |
| Job Quartz | Không có context | Để `TenantJobAspect` set, hoặc `TenantUtils.execute` |
| MQ consumer | Message không mang tenant | Gắn tenant vào header; consumer `@TenantIgnore` nếu global |

**Test bắt buộc:** tạo dữ liệu ở tenant A và B, query dưới tenant A, assert **không** thấy bản ghi của B.

## 8. Error handling

### 8.1. Khai báo error code

```java
public interface ErrorCodeConstants {

    // ========== User module 1-002-003-000 ==========
    ErrorCode USER_NOT_EXISTS = new ErrorCode(1_002_003_003, "User does not exist");
    ErrorCode USER_USERNAME_EXISTS = new ErrorCode(1_002_003_000, "Username already exists");
}
```

- Mỗi module một `ErrorCodeConstants` trong `enums/`.
- Mã lỗi 10 chữ số, theo range của module. System dùng `1_002_xxx_xxx`; xem comment trong `ServiceErrorCodeRange`.
- Tên theo `ENTITY_REASON`: `USER_NOT_EXISTS`, `ROLE_NAME_DUPLICATE`.
- Không tái sử dụng mã của module khác. Không dùng số tuỳ tiện ngoài range.

### 8.2. Ném lỗi

```java
import static cn.iocoder.yudao.framework.common.exception.util.ServiceExceptionUtil.exception;

if (user == null) {
    throw exception(USER_NOT_EXISTS);
}
```

- Chỉ ném `ServiceException` cho lỗi nghiệp vụ. Không ném `RuntimeException("...")`.
- Không catch `ServiceException` rồi bọc lại thành lỗi khác, trừ khi cần đổi mã.
- `GlobalExceptionHandler` đã map `ServiceException` → `CommonResult` với `code` + `msg`.
- Lỗi không xác định → handler trả `500` với message chung. Stack trace chỉ vào log, **không** ra client.
- Không đưa `ex.getMessage()`, SQL, tên bảng, đường dẫn file vào response.

### 8.3. Ngôn ngữ message

- Message trong `ErrorCode` viết **EN** cho code mới.
- Repo hiện tại còn nhiều message tiếng Trung của upstream. Không dịch hàng loạt.
- Repo **chưa có** message bundle i18n (`ErrorCode` mới chỉ có TODO). Cần tiếng Việt thì thêm `messages_vi.properties` + `MessageSource`, đừng hardcode chuỗi VI trong `ErrorCode`.
- Log message viết EN, kèm ID/tenant, không log password/token/CCCD.

## 9. Test

Dependency có sẵn trong module:

```xml
<dependency>
    <groupId>cn.iocoder.boot</groupId>
    <artifactId>yudao-spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

Vị trí: `src/test/java/...`, **cùng package** với source. Tên class `XxxTest`.

| Base class | Dùng khi |
|---|---|
| `BaseDbUnitTest` | Service + Mapper với H2 |
| `BaseMockitoUnitTest` | Chỉ mock, không DB |
| `BaseRedisUnitTest` | Có Redis |
| `BaseDbAndRedisUnitTest` | Có cả DB và Redis |

```java
@Import(PostServiceImpl.class)
public class PostServiceImplTest extends BaseDbUnitTest {

    @Resource
    private PostServiceImpl postService;
    @Resource
    private PostMapper postMapper;

    @Test
    public void testCreatePost_success() {
        PostSaveReqVO reqVO = randomPojo(PostSaveReqVO.class).setId(null);

        Long id = postService.createPost(reqVO);

        assertNotNull(id);
        assertPojoEquals(reqVO, postMapper.selectById(id), "id");
    }

    @Test
    public void testCreatePost_nameDuplicate() {
        PostDO exists = randomPojo(PostDO.class);
        postMapper.insert(exists);

        PostSaveReqVO reqVO = randomPojo(PostSaveReqVO.class)
                .setId(null).setName(exists.getName());

        assertServiceException(() -> postService.createPost(reqVO), POST_NAME_DUPLICATE);
    }
}
```

- Method test: `test<Method>_<case>` — `testCreateUser_success`, `testCreateUser_usernameExists`.
- Assert error bằng `assertServiceException`, không assert message string.
- Không gọi API thật, không gửi mail/SMS trong unit test.
- Tối thiểu trước khi mở PR: **service logic** + **tenant isolation**.

Tenant isolation cần `TenantDatabaseInterceptor`, nên **không** chạy được với `BaseDbUnitTest`. Đặt test ở nơi boot full context (`yudao-server/src/test`), dùng profile `local`:

```java
// yudao-server/src/test/java/.../UserTenantIsolationTest.java
// Cần yudao-spring-boot-starter-test (scope test) trong yudao-server/pom.xml
@SpringBootTest
@ActiveProfiles("integration")
class UserTenantIsolationTest {

    @Resource
    private AdminUserService userService;

    @Test
    void pageUser_onlyReturnsCurrentTenant() {
        Long idA = TenantUtils.execute(1L, () -> createUser("tenant-a"));
        Long idB = TenantUtils.execute(2L, () -> createUser("tenant-b"));

        PageResult<AdminUserDO> page = TenantUtils.execute(1L,
                () -> userService.getUserPage(new UserPageReqVO()));

        assertTrue(page.getList().stream().anyMatch(u -> idA.equals(u.getId())));
        assertTrue(page.getList().stream().noneMatch(u -> idB.equals(u.getId())));
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

Test này boot cả app nên chậm. Tạo profile `integration` trỏ DB/Redis test riêng; **không** chạy trên DB local thật.

## 10. Git workflow giữ fork sync upstream

Mô hình:

| Remote | Repo | Vai trò |
|---|---|---|
| `origin` | `chuanghiduoc/ruoyi-vue-pro` | Fork của bạn, nơi push |
| `upstream` | `YunaiV/ruoyi-vue-pro` | Repo gốc, chỉ fetch |

### 10.1. Cài đặt lần đầu

```bash
# kiểm tra remote hiện có
git remote -v

# chỉ thêm nếu chưa có upstream
git remote add upstream https://github.com/YunaiV/ruoyi-vue-pro.git
git fetch upstream
```

Nếu đã có `upstream` nhưng sai URL:

```bash
git remote set-url upstream https://github.com/YunaiV/ruoyi-vue-pro.git
```

### 10.2. Đồng bộ định kỳ

```bash
git checkout master
git fetch upstream
git merge upstream/master
git push origin master
```

- Dùng `merge`, không `rebase` cho nhánh `master` đã push. Rebase buộc phải force push.
- Nếu conflict: xử lý, `git add`, `git commit`, rồi push.
- Không bao giờ `git push --force` lên `master`.

### 10.3. Sync bằng GitHub UI

Trên trang fork, bấm **Sync fork → Update branch**.

- GitHub merge `upstream/master` vào `master` của fork.
- Commit docs của bạn **vẫn giữ**, vì nằm trên các commit của upstream.
- Chỉ conflict khi bạn sửa cùng file/dòng với upstream.
- Sau khi sync UI, máy local cần `git pull origin master`.

### 10.4. Quy tắc giữ fork sạch

1. **Chỉ thêm file mới**: `README.vi.md`, `docs/vi/*`. Đây là các file upstream không có → không conflict.
2. **Không sửa code upstream** trong nhánh tài liệu. Không format lại, không đổi version, không sửa typo.
3. **Không commit** file IDE, `.env`, log, `target/`.
4. Nếu buộc phải sửa file upstream:
   - Tách commit riêng, không trộn với docs.
   - Ghi rõ lý do trong commit body.
   - Giữ diff nhỏ nhất có thể.
   - Cân nhắc mở PR cho upstream thay vì giữ patch riêng.

```bash
# commit docs
git add docs/vi/04-quy-uoc.md
git commit -m "docs(vi): add coding conventions guide"

# commit sửa upstream (nếu buộc phải có)
git add path/to/upstream-file
git commit -m "chore(fork): keep local patch for ..." -m "Reason: ..."
```

## 11. Commit convention

Dùng **Conventional Commits**, viết **EN**, thể mệnh lệnh, không dấu chấm cuối.

```
<type>(<scope>): <subject>
```

| Type | Dùng cho |
|---|---|
| `feat` | Tính năng mới |
| `fix` | Sửa bug |
| `docs` | Tài liệu |
| `refactor` | Đổi cấu trúc, không đổi hành vi |
| `test` | Thêm/sửa test |
| `chore` | Build, config, việc lặt vặt |
| `perf` | Tối ưu hiệu năng |

Ví dụ:

```bash
docs(vi): add Vietnamese architecture guide
docs(vi): translate README
docs(vi): add coding conventions guide
fix(module-system): prevent duplicate username on create
feat(module-infra): add file upload limit
refactor(module-system): extract user converter
test(module-system): cover tenant isolation for user page
```

Quy tắc:

- Scope là module hoặc khu vực: `vi`, `module-system`, `module-infra`, `server`.
- Subject ≤ 72 ký tự, imperative: `add`, `fix`, `remove` — không `added`, `fixes`.
- Mỗi commit một việc. Docs và code không trộn.
- Body giải thích **why**, không lặp lại **what**.

## 12. Checklist trước khi mở PR

- [ ] Chỉ thêm file mới trong `docs/vi/` (và `README.vi.md`), không đụng code upstream.
- [ ] Thuật ngữ khớp `00-glossary.md`; identifier, path, field giữ EN.
- [ ] Link chéo giữa các file docs đúng, không có link chết.
- [ ] Code mẫu trong docs khớp API thật (`CommonResult`, `PageParam`, `TenantBaseDO`...).
- [ ] Không có secret, token, mật khẩu, dữ liệu khách hàng trong diff.
- [ ] `mvn -pl yudao-module-xxx -am test` pass (nếu có sửa code).
- [ ] Có test cho service logic; có test/kiểm tra tenant isolation nếu đụng dữ liệu tenant.
- [ ] Commit message theo Conventional Commits; PR nhỏ, mô tả rõ why.
