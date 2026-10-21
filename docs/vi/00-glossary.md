# 00 — Glossary & quy ước dịch

Mục đích: giữ cách dịch **nhất quán** trong toàn bộ `docs/vi/` và `README.vi.md`.

## Quy tắc cốt lõi

> **Máy đọc = tiếng Anh. Người đọc = tiếng Việt.**

| Thành phần | Ngôn ngữ | Ghi chú |
|---|---|---|
| Tên class, package, biến, hàm | **EN** | Không bao giờ dịch |
| Tên bảng, cột, index | **EN** | Schema là contract |
| API path, field, enum value | **EN** | Client phụ thuộc |
| Log message | **EN** | grep / parse / alert |
| Comment, Javadoc trong code | **EN** | Giữ merge upstream |
| Commit message, branch, PR | **EN** | Conventional Commits |
| README, docs/ | **VI** | File này |
| Error message | **EN** mặc định + i18n `vi` | Không hardcode |
| UI label | i18n `vi.json` | Không hardcode |
| Seed/demo data | **VI** được | Chỉ dữ liệu demo |

## Thuật ngữ

| Term | Cách dùng trong docs | Giải thích |
|---|---|---|
| **module** | module | Đơn vị Maven/nghiệp vụ, ví dụ `yudao-module-system` |
| **starter** | starter | Spring Boot starter trong `yudao-framework` |
| **tenant** | tenant | Khách hàng/không gian dữ liệu độc lập trong SaaS |
| **multi-tenant** | multi-tenant | Một hệ thống phục vụ nhiều tenant |
| **tenant package** | gói tenant | Bộ menu/quyền gán cho tenant |
| **RBAC** | RBAC | Role-Based Access Control |
| **data permission** | phân quyền dữ liệu | Lọc dữ liệu theo phòng ban/người dùng (row-level) |
| **codegen** | sinh mã / codegen | Sinh code từ bảng DB |
| **job** | tác vụ định kỳ / job | Quartz job |
| **BPM** | quy trình nghiệp vụ (BPM) | Business Process Management |
| **process definition** | định nghĩa quy trình | Bản thiết kế quy trình |
| **process instance** | instance quy trình | Một lần chạy cụ thể |
| **task** | task / việc cần xử lý | Đơn vị công việc trong quy trình |
| **form** | form | Biểu mẫu động |
| **CC** | CC | Gửi bản sao kết quả duyệt |
| **sign-off** | ký duyệt | Hành động duyệt |
| **DO** | DO | Data Object — map 1-1 với bảng DB |
| **DTO** | DTO | Data Transfer Object — dữ liệu qua API/RPC |
| **VO** | VO | View Object — request/response của Controller |
| **BO** | BO | Business Object — object nghiệp vụ nội bộ |
| **Mapper** | Mapper | MyBatis Mapper |
| **Convert** | Convert | MapStruct mapper giữa DO/VO/DTO |
| **admin-api** | admin API | API cho backend/admin (`/admin-api/**`) |
| **app-api** | app API | API cho C-end (`/app-api/**`) |
| **upstream** | upstream | Repo gốc `YunaiV/ruoyi-vue-pro` |
| **fork** | fork | Bản sao trên account `chuanghiduoc` |

## Quy ước viết

1. Câu ngắn, chủ động, tránh dịch word-by-word.
2. Giữ nguyên tên riêng: Spring Boot, MySQL, Redis, Flowable, Quartz, MyBatis-Plus.
3. Với thuật ngữ chưa có từ Việt ổn: **giữ tiếng Anh**, lần đầu xuất hiện thêm giải thích trong ngoặc.
4. Không dịch tên module, tên package, tên bảng.
5. Số liệu phải ghi rõ **ngày lấy** nếu là star/release.
