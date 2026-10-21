# Tài liệu tiếng Việt — `ruoyi-vue-pro`

Đây là bản **Việt hóa phần docs** của [`YunaiV/ruoyi-vue-pro`](https://github.com/YunaiV/ruoyi-vue-pro), thực hiện trên fork [`chuanghiduoc/ruoyi-vue-pro`](https://github.com/chuanghiduoc/ruoyi-vue-pro).

- **Không sửa code upstream.** Chỉ thêm file mới → fork vẫn `Sync fork` được trên GitHub UI.
- **Mã nguồn, tên class, API, bảng, cột giữ nguyên tiếng Anh.**
- Bản dịch dựa trên commit `ac022b1` (2026-10-05), nhánh `master` (JDK 8 + Spring Boot 2.7).

## Mục lục

| File | Nội dung |
|---|---|
| [`00-glossary.md`](00-glossary.md) | Thuật ngữ EN–VI và quy ước dịch |
| [`01-kien-truc.md`](01-kien-truc.md) | Kiến trúc modular monolith, 15 starter, multi-tenant, data permission, job, codegen |
| [`02-quickstart.md`](02-quickstart.md) | Cài đặt, import SQL, cấu hình, chạy, lỗi thường gặp |
| [`03-module-guide.md`](03-module-guide.md) | 19 module cấp 1 (20 artifact), phụ thuộc, khi nào bật |
| [`04-quy-uoc.md`](04-quy-uoc.md) | Quy ước code, package, RBAC, tenant, test, git workflow |
| [`05-monolith-vs-microservices.md`](05-monolith-vs-microservices.md) | So sánh `ruoyi-vue-pro` và `yudao-cloud` |
| [`06-lo-trinh.md`](06-lo-trinh.md) | Lộ trình 6 phase để dựng backend platform riêng |
| [`../README.vi.md`](../../README.vi.md) | Bản dịch README gốc |

## Đọc theo thứ tự

1. Người mới: `02-quickstart.md` → `01-kien-truc.md`
2. Sắp viết module: `03-module-guide.md` → `04-quy-uoc.md`
3. Cân nhắc kiến trúc: `05-monolith-vs-microservices.md`
4. Bắt tay dựng platform: `06-lo-trinh.md`

## Đồng bộ upstream

```bash
git remote add upstream https://github.com/YunaiV/ruoyi-vue-pro.git
git fetch upstream
git merge upstream/master
```

Hoặc trên GitHub: mở fork → **Sync fork** → **Update branch**.

Vì toàn bộ thay đổi nằm ở file mới (`README.vi.md`, `docs/vi/*`), merge upstream không bị conflict.

## Giấy phép

Repo gốc dùng **MIT**. Bản dịch là tác phẩm phái sinh, giữ nguyên MIT và ghi công tác giả gốc.
