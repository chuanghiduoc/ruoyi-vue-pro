<p align="center">
 <img src="https://img.shields.io/badge/Spring%20Boot-2.7.18-blue.svg" alt="Spring Boot">
 <img src="https://img.shields.io/badge/Vue-3.2-blue.svg" alt="Vue">
 <img src="https://img.shields.io/github/license/YunaiV/ruoyi-vue-pro"/>
</p>

> 🇻🇳 **Bản dịch tiếng Việt** của README gốc.
> Bản gốc: <https://github.com/YunaiV/ruoyi-vue-pro> · Dịch từ commit `ac022b1` (2026-10-05), nhánh `master` (JDK 8 + Spring Boot 2.7).
> Quy ước dịch thuật: xem [`docs/vi/00-glossary.md`](docs/vi/00-glossary.md). Tên class, package, API, bảng, cột **giữ nguyên tiếng Anh**.

**Tuyên bố nghiêm túc: hiện tại và tương lai đều không có bản thương mại, toàn bộ mã nguồn đều mở!**

**「Tôi thích viết code, làm hoài không chán」**
**「Tôi thích làm open source, đó là niềm vui của tôi」**

Tôi 🐶 đang bươn chải ở Thượng Hải, ngày đêm cày cuốc ở một trong top 3 big tech, ban đêm đóng góp cho open source.

Nếu dự án này giúp ích cho bạn, nhớ Star theo dõi nhé — đó là động lực rất lớn với tôi.

## 🐶 Người mới nên đọc

* Bản demo【Vue3 + element-plus】: <http://dashboard-vue3.yudao.iocoder.cn>
* Bản demo【Vue3 + vben (ant-design-vue)】: <http://dashboard-vben.yudao.iocoder.cn>
* Bản demo【Vue2 + element-ui】: <http://dashboard.yudao.iocoder.cn>
* Tài liệu khởi động: <https://doc.iocoder.cn/quick-start/>
* Video hướng dẫn: <https://doc.iocoder.cn/video/>

## 🐰 Phiên bản

| Phiên bản | JDK 8 + Spring Boot 2.7 | JDK 17/21 + Spring Boot 3.5 | JDK 25 + Spring Boot 4.x |
|---|---|---|---|
| 【Bản đầy đủ】[ruoyi-vue-pro](https://gitee.com/zhijiantianya/ruoyi-vue-pro) | nhánh [`master`](https://gitee.com/zhijiantianya/ruoyi-vue-pro/tree/master/) | nhánh [`master-jdk17`](https://gitee.com/zhijiantianya/ruoyi-vue-pro/tree/master-jdk17/) | nhánh [`master-jdk25`](https://gitee.com/zhijiantianya/ruoyi-vue-pro/tree/master-jdk25/) |
| 【Bản rút gọn】[yudao-boot-mini](https://gitee.com/yudaocode/yudao-boot-mini) | nhánh [`master`](https://gitee.com/yudaocode/yudao-boot-mini/tree/master/) | nhánh [`master-jdk17`](https://gitee.com/yudaocode/yudao-boot-mini/tree/master-jdk17/) | nhánh [`master-jdk25`](https://gitee.com/yudaocode/yudao-boot-mini/tree/master-jdk25/) |

* 【Bản đầy đủ】: gồm chức năng hệ thống, hạ tầng, quy trình nghiệp vụ (BPM), thanh toán, báo cáo dữ liệu, WeChat Official Account, thương mại điện tử, trung tâm hội viên, ERP, WMS, CRM, MES, HRM, FMS, PMS, OA, AI/LLM, IoT, IM.
* 【Bản rút gọn】: chỉ gồm chức năng hệ thống và hạ tầng; **không** gồm BPM, thanh toán, báo cáo, WeChat, thương mại điện tử, hội viên, ERP, WMS, CRM, MES, HRM, FMS, PMS, OA, AI, IoT, IM.

Có thể tham khảo [《tài liệu migration》](https://doc.iocoder.cn/migrate-module/): chỉ mất 5–10 phút để chuyển từ **bản đầy đủ** sang **bản rút gọn** theo nhu cầu.

## 🐯 Giới thiệu nền tảng

**Yudao (芋道)** — lấy lập trình viên làm trung tâm, xây dựng nền tảng phát triển nhanh hàng đầu Trung Quốc, **toàn bộ mã nguồn mở**, cá nhân và doanh nghiệp dùng miễn phí 100%.

> Có câu hỏi hoặc muốn thêm tính năng, hãy tạo _Issues_ gửi cho NaiNai (艿艿).
>
> 😜 Nhớ Star dự án nhé, điều đó rất quan trọng với chúng tôi!

![Sơ đồ kiến trúc](/.image/common/ruoyi-vue-pro-architecture.png)

* Backend Java: nhánh `master` dùng JDK 8 + Spring Boot 2.7; `master-jdk17` dùng JDK 17/21 + Spring Boot 3.5; `master-jdk25` dùng JDK 25 + Spring Boot 4.x.
* Admin desktop: Vue 3 có 2 phiên bản `element-plus` và `vben (ant-design-vue)`; Vue 2 có bản `element-ui`.
* Admin mobile: dùng `uni-app`, một codebase chạy nhiều nền tảng — APP, mini program, H5.
* Backend dùng kiến trúc **Spring Boot multi-module**, MySQL + MyBatis-Plus, Redis + Redisson.
* Database hỗ trợ: MySQL, Oracle, PostgreSQL, SQL Server, MariaDB, DM (Đạt Mộng), TiDB…
* Message queue hỗ trợ: Event, Redis, RabbitMQ, Kafka, RocketMQ…
* Xác thực & phân quyền: Spring Security + Token + Redis; hỗ trợ nhiều loại thiết bị/người dùng, SSO.
* Menu phân quyền động, phân quyền tới cấp button, cache Redis để tăng hiệu năng.
* Hỗ trợ **SaaS multi-tenant**, tuỳ biến quyền theo từng tenant, đóng gói tầng multi-tenant trong suốt với business code.
* Workflow dùng Flowable: form động, thiết kế quy trình online, ký đồng thời / ký bất kỳ, nhiều cách gán người xử lý.
* **Code generator**: sinh một lần ra code Java + Vue + SQL + tài liệu API; hỗ trợ bảng đơn, bảng cây, master–detail.
* Real-time: Spring WebSocket, tích hợp xác thực Token, hỗ trợ WebSocket cluster.
* Tích hợp đăng nhập WeChat mini program / Official Account / WeCom / DingTalk; tích hợp thanh toán & hoàn tiền Alipay, WeChat.
* Tích hợp SMS: Alibaba Cloud, Tencent Cloud…; lưu trữ: MinIO, Alibaba Cloud OSS, Tencent COS, Qiniu…
* Tích hợp report designer và dashboard designer: kéo–thả để tạo báo cáo và màn hình lớn.

## 🐳 Quan hệ dự án

![Lộ trình kiến trúc](/.image/common/yudao-roadmap.png)

So sánh tính năng của 3 dự án: xem bảng [so sánh các dự án open source nội địa](https://www.yuque.com/xiatian-bsgny/lm0ec1/wqf8mn) do cộng đồng tổng hợp.

### Dự án backend

| Dự án | Star | Mô tả |
|---|---|---|
| [ruoyi-vue-pro](https://gitee.com/zhijiantianya/ruoyi-vue-pro) | [![Gitee star](https://gitee.com/zhijiantianya/ruoyi-vue-pro/badge/star.svg?theme=white)](https://gitee.com/zhijiantianya/ruoyi-vue-pro) [![GitHub stars](https://img.shields.io/github/stars/YunaiV/ruoyi-vue-pro.svg?style=social&label=Stars)](https://github.com/YunaiV/ruoyi-vue-pro) | Kiến trúc Spring Boot multi-module |
| [yudao-cloud](https://gitee.com/zhijiantianya/yudao-cloud) | [![Gitee star](https://gitee.com/zhijiantianya/yudao-cloud/badge/star.svg?theme=white)](https://gitee.com/zhijiantianya/yudao-cloud) [![GitHub stars](https://img.shields.io/github/stars/YunaiV/yudao-cloud.svg?style=social&label=Stars)](https://github.com/YunaiV/yudao-cloud) | Kiến trúc microservice Spring Cloud |
| [Spring-Boot-Labs](https://gitee.com/yudaocode/SpringBoot-Labs) | [![Gitee star](https://gitee.com/yudaocode/SpringBoot-Labs/badge/star.svg?theme=white)](https://gitee.com/yudaocode/SpringBoot-Labs) [![GitHub stars](https://img.shields.io/github/stars/yudaocode/SpringBoot-Labs.svg?style=social&label=Stars)](https://github.com/yudaocode/SpringBoot-Labs) | Chuyên mục học Spring Boot & Cloud |

### Dự án frontend

| Dự án | Star | Mô tả |
|---|---|---|
| [yudao-ui-admin-vue3](https://gitee.com/yudaocode/yudao-ui-admin-vue3) | [![Gitee star](https://gitee.com/yudaocode/yudao-ui-admin-vue3/badge/star.svg?theme=white)](https://gitee.com/yudaocode/yudao-ui-admin-vue3) [![GitHub stars](https://img.shields.io/github/stars/yudaocode/yudao-ui-admin-vue3.svg?style=social&label=Stars)](https://github.com/yudaocode/yudao-ui-admin-vue3) | Admin xây bằng Vue3 + element-plus |
| [yudao-ui-admin-vben](https://gitee.com/yudaocode/yudao-ui-admin-vben) | [![Gitee star](https://gitee.com/yudaocode/yudao-ui-admin-vben/badge/star.svg?theme=white)](https://gitee.com/yudaocode/yudao-ui-admin-vben) [![GitHub stars](https://img.shields.io/github/stars/yudaocode/yudao-ui-admin-vben.svg?style=social&label=Stars)](https://github.com/yudaocode/yudao-ui-admin-vben) | Admin xây bằng Vue3 + vben (ant-design-vue) |
| [yudao-mall-uniapp](https://gitee.com/yudaocode/yudao-mall-uniapp) | [![Gitee star](https://gitee.com/yudaocode/yudao-mall-uniapp/badge/star.svg?theme=white)](https://gitee.com/yudaocode/yudao-mall-uniapp) [![GitHub stars](https://img.shields.io/github/stars/yudaocode/yudao-mall-uniapp.svg?style=social&label=Stars)](https://github.com/yudaocode/yudao-mall-uniapp) | Mini program thương mại điện tử xây bằng uni-app |
| [yudao-ui-admin-vue2](https://gitee.com/yudaocode/yudao-ui-admin-vue2) | [![Gitee star](https://gitee.com/yudaocode/yudao-ui-admin-vue2/badge/star.svg?theme=white)](https://gitee.com/yudaocode/yudao-ui-admin-vue2) [![GitHub stars](https://img.shields.io/github/stars/yudaocode/yudao-ui-admin-vue2.svg?style=social&label=Stars)](https://github.com/yudaocode/yudao-ui-admin-vue2) | Admin xây bằng Vue2 + element-ui |
| [yudao-ui-admin-uniapp](https://gitee.com/yudaocode/yudao-ui-admin-uniapp) | [![Gitee star](https://gitee.com/yudaocode/yudao-ui-admin-uniapp/badge/star.svg?theme=white)](https://gitee.com/yudaocode/yudao-ui-admin-uniapp) [![GitHub stars](https://img.shields.io/github/stars/yudaocode/yudao-ui-admin-uniapp.svg?style=social&label=Stars)](https://github.com/yudaocode/yudao-ui-admin-uniapp) | Admin xây bằng Vue2 + element-ui |
| [yudao-ui-go-view](https://gitee.com/yudaocode/yudao-ui-go-view) | [![Gitee star](https://gitee.com/yudaocode/yudao-ui-go-view/badge/star.svg?theme=white)](https://gitee.com/yudaocode/yudao-ui-go-view) [![GitHub stars](https://img.shields.io/github/stars/yudaocode/yudao-ui-go-view.svg?style=social&label=Stars)](https://github.com/yudaocode/yudao-ui-go-view) | Dashboard/report lớn xây bằng Vue3 + naive-ui |

## 😎 Giấy phép

**Vì sao nên dùng dự án này?**

① Dự án dùng [MIT License](https://gitee.com/zhijiantianya/ruoyi-vue-pro/blob/master/LICENSE) — thoáng hơn cả Apache 2.0. Cá nhân và doanh nghiệp dùng miễn phí 100%, **không cần giữ tên tác giả hay copyright**.

② Toàn bộ code đều mở, không kiểu chỉ mở một phần rồi giấu phần lõi. Xem [bảng so sánh các dự án open source nội địa](https://www.yuque.com/xiatian-bsgny/lm0ec1/wqf8mn).

![So sánh dự án open source](/.image/common/project-vs.png)

③ Code gọn, kiến trúc gọn, tuân theo《Alibaba Java Development Manual》, comment chi tiết: **113.770 dòng Java**, trong đó **42.462 dòng comment**.

## 🤝 Outsourcing

Chúng tôi cũng nhận outsourcing. Nếu bạn có dự án cần làm, liên hệ WeChat【**Aix9975**】.

Team gồm PM, architect, frontend, backend, tester, DevOps — có thể cung cấp dịch vụ trọn gói.

Loại dự án: thương mại điện tử, SCRM, OA, logistics, ERP, HRM, FMS, PMS, CMS, HIS, thanh toán, IM, WeChat Official Account, mini program…

## 🐼 Tính năng có sẵn

Hệ thống có nhiều chức năng nghiệp vụ đóng gói sẵn, dùng để dựng nhanh hệ thống của bạn:

![Phân tầng chức năng](/.image/common/ruoyi-vue-pro-biz.png)

* **Module phổ thông (bắt buộc)**: chức năng hệ thống, hạ tầng.
* **Module phổ thông (tuỳ chọn)**: BPM, thanh toán, báo cáo dữ liệu, trung tâm hội viên.
* **Hệ thống nghiệp vụ (dùng theo nhu cầu)**: Mall, ERP, WMS, CRM, CMS, MES, HRM, FMS, PMS, OA, AI/LLM, IoT, IM, Mobile, Report dashboard.

> Lưu ý: dự án dựa trên RuoYi-Vue, **refactor** lại backend và **làm đẹp** frontend.
>
> * Tính năng **thêm mới** được đánh dấu 🚀.
> * Tính năng **viết lại** được đánh dấu ⭐️.

🙂 Mọi tính năng đều được bảo đảm bằng **unit test**.

### Chức năng hệ thống

| | Chức năng | Mô tả |
|---|---|---|
| | Quản lý người dùng | Người dùng là chủ thể thao tác hệ thống; cấu hình tài khoản người dùng |
| ⭐️ | Người dùng online | Giám sát trạng thái người dùng đang hoạt động; có thể kick thủ công |
| | Quản lý vai trò | Gán quyền menu cho vai trò; cấu hình phạm vi dữ liệu theo đơn vị |
| | Quản lý menu | Cấu hình menu, quyền thao tác, mã quyền button; cache local để tăng hiệu năng |
| | Quản lý phòng ban | Cấu hình cơ cấu tổ chức (công ty, phòng, nhóm); hiển thị dạng cây, hỗ trợ phân quyền dữ liệu |
| | Quản lý chức danh | Cấu hình chức danh/chức vụ của người dùng |
| 🚀 | Quản lý tenant | Cấu hình tenant, hỗ trợ multi-tenant cho SaaS |
| 🚀 | Gói tenant | Cấu hình gói cho tenant: menu, thao tác, quyền button riêng cho từng tenant |
| | Quản lý từ điển (dict) | Bảo trì các tập dữ liệu cố định dùng thường xuyên |
| 🚀 | Quản lý SMS | Kênh SMS, template SMS, log SMS; tích hợp Alibaba Cloud, Tencent Cloud… |
| 🚀 | Quản lý email | Tài khoản email, template email, log gửi mail; hỗ trợ mọi nền tảng mail |
| 🚀 | Thông báo nội bộ | Thông báo trong hệ thống; có template và message |
| 🚀 | Log thao tác | Ghi và tra cứu log thao tác bình thường; tích hợp Swagger để sinh nội dung log |
| ⭐️ | Log đăng nhập | Ghi và tra cứu log đăng nhập, bao gồm đăng nhập lỗi |
| 🚀 | Quản lý mã lỗi | Quản lý toàn bộ error code; sửa message online, không cần restart |
| | Thông báo & announcement | Đăng và bảo trì thông báo hệ thống |
| 🚀 | Từ nhạy cảm | Cấu hình từ nhạy cảm, hỗ trợ gắn tag/nhóm |
| 🚀 | Quản lý ứng dụng | Quản lý ứng dụng SSO, hỗ trợ nhiều kiểu cấp quyền OAuth2 |
| 🚀 | Quản lý khu vực | Hiển thị tỉnh/thành/quận; hỗ trợ map IP → thành phố |

![Sơ đồ chức năng](/.image/common/system-feature.png)

### Hạ tầng

| | Chức năng | Mô tả |
|---|---|---|
| 🚀 | Sinh mã (codegen) | Sinh code frontend + backend (Java, Vue, SQL, unit test); hỗ trợ tải CRUD |
| 🚀 | API hệ thống | Tự sinh tài liệu RESTful API từ Swagger |
| 🚀 | Tài liệu database | Tự sinh tài liệu DB bằng Screw; export Word, HTML, MD |
| | Form builder | Kéo–thả phần tử form để sinh HTML; export JSON, file Vue |
| 🚀 | Quản lý cấu hình | Cấu hình tham số động, Spring Boot load được |
| ⭐️ | Tác vụ định kỳ (job) | Thêm/sửa/xoá lịch chạy online, có log kết quả |
| 🚀 | Dịch vụ file | Lưu file lên S3 (MinIO, Alibaba, Tencent, Qiniu), local, FTP, DB… |
| 🚀 | WebSocket | Ví dụ tích hợp WebSocket, gửi 1–1 và 1–n |
| 🚀 | Log API | Log truy cập RESTful API + log exception, phục vụ debug |
| | Giám sát MySQL | Theo dõi connection pool, phân tích SQL tìm điểm nghẽn |
| | Giám sát Redis | Theo dõi Redis, quản lý Redis key |
| 🚀 | Message queue | MQ trên Redis: Stream cho cluster consumer, Pub/Sub cho broadcast |
| 🚀 | Giám sát Java | Giám sát ứng dụng qua Spring Boot Admin |
| 🚀 | Tracing | Tích hợp SkyWalking để trace request |
| 🚀 | Log center | Tích hợp SkyWalking để xem log tập trung |
| 🚀 | Bảo vệ dịch vụ | Distributed lock, idempotency, rate limit trên Redis cho tình huống tải cao |
| 🚀 | Dịch vụ log | Log center nhẹ, xem log server từ xa |
| 🚀 | Unit test | JUnit + Mockito, bảo đảm đúng chức năng và chất lượng code |

![Sơ đồ chức năng](/.image/common/infra-feature.png)

### Quy trình nghiệp vụ (BPM)

![Sơ đồ chức năng](/.image/common/bpm-feature.png)

Xây trên Flowable, hỗ trợ database nội địa (信创), đáp ứng đặc thù quy trình Trung Quốc:

| Trình thiết kế BPMN | Trình thiết kế kiểu DingTalk/Feishu |
|---|---|
| ![](/.image/工作流设计器-bpmn.jpg) | ![](/.image/工作流设计器-simple.jpg) |

> Đã được kiểm chứng trên môi trường production của các doanh nghiệp đầu ngành: workflow engine **bắt buộc phải có cả 2 trình thiết kế** — kiểu DingTalk/Feishu và BPMN.
>
> Trình đầu để cấu hình quy trình đơn giản, trình sau để dàn dựng kịch bản phức tạp.

| Danh sách chức năng | Mô tả | Hoàn thành |
|---|---|---|
| Trình thiết kế SIMPLE | Giống DingTalk/Feishu, kéo–thả để dựng form và quy trình, cấu hình luồng duyệt trong 10 phút | ✅ |
| Trình thiết kế BPMN | Theo chuẩn BPMN, phù hợp kịch bản phức tạp, nhiều tầng duyệt và tự động hoá | ✅ |
| Ký đồng thời (会签) | Một node duyệt gán nhiều người (A, B, C); tất cả phải đồng ý mới qua node tiếp | ✅ |
| Ký bất kỳ (或签) | Một node gán nhiều người; chỉ cần một người xử lý là qua node tiếp | ✅ |
| Duyệt tuần tự | Một node gán nhiều người; lần lượt A → B → C, tất cả đồng ý mới qua node tiếp | ✅ |
| CC (抄送) | Gửi kết quả duyệt cho người nhận CC; mặc định chống trùng | ✅ |
| Trả về (驳回) | Đưa quy trình về một node để duyệt lại: về người khởi tạo, node trước, hoặc node bất kỳ | ✅ |
| Chuyển xử lý (转办) | A chuyển cho B duyệt; B duyệt xong thì sang node tiếp | ✅ |
| Uỷ quyền (委派) | A chuyển cho B duyệt; B duyệt xong trả lại A; A duyệt tiếp rồi sang node sau | ✅ |
| Thêm người duyệt (加签) | Người duyệt hiện tại tự thêm người duyệt cho node, thêm trước hoặc sau | ✅ |
| Bớt người duyệt (减签) | Huỷ thêm người duyệt trước khi người duyệt hiện tại thao tác | ✅ |
| Thu hồi (撤销) | Người khởi tạo có thể thu hồi quy trình | ✅ |
| Kết thúc (终止) | Admin dừng instance quy trình tại bất kỳ node nào | ✅ |
| Quyền trên form | Kéo–thả cấu hình form; mỗi node cấu hình quyền chỉ đọc / sửa / ẩn | ✅ |
| Duyệt quá hạn | Cấu hình thời gian; quá hạn tự động duyệt / không duyệt / trả về | ✅ |
| Nhắc nhở tự động | Cấu hình thời điểm nhắc; tự gửi SMS, email, thông báo nội bộ; hỗ trợ lặp | ✅ |
| Quy trình cha–con | Node cha tự kích hoạt quy trình con; con xong thì cha mới chạy tiếp; hỗ trợ sync/async | ✅ |
| Nhánh điều kiện | Nhánh exclusive: chọn một nhánh theo điều kiện | ✅ |
| Nhánh song song | Chia nhiều nhánh, không xét điều kiện, tất cả đều chạy | ✅ |
| Nhánh bao hàm (inclusive) | Kết hợp điều kiện + song song: chọn nhiều nhánh theo điều kiện, có nhánh mặc định | ✅ |
| Nhánh định tuyến | Chọn một nhánh theo điều kiện (redirect tới node chỉ định) hoặc nhánh mặc định | ✅ |
| Node kích hoạt | Đến node này thì gọi HTTP request / callback / update data / delete data… | ✅ |
| Node trì hoãn | Đến node này thì chờ một khoảng thời gian; hỗ trợ thời lượng cố định, ngày cố định | ✅ |
| Cấu hình mở rộng | Thông báo trước/sau quy trình, trước/sau node; báo cáo quy trình; chống trùng auto-approve; tự đặt mã/title/summary | ✅ |

### Hệ thống thanh toán

| | Chức năng | Mô tả |
|---|---|---|
| 🚀 | Thông tin ứng dụng | Cấu hình thông tin merchant/app, kết nối Alipay, WeChat… |
| 🚀 | Đơn thanh toán | Xem đơn **thanh toán** Alipay/WeChat do người dùng tạo |
| 🚀 | Đơn hoàn tiền | Xem đơn **hoàn tiền** Alipay/WeChat |
| 🚀 | Callback | Xem kết quả callback **thanh toán** / **hoàn tiền** |
| 🚀 | Ví dụ tích hợp | Ví dụ thực chiến tích hợp **thanh toán** / **hoàn tiền** |

### Báo cáo dữ liệu

| | Chức năng | Mô tả |
|---|---|---|
| 🚀 | Report designer | Báo cáo dữ liệu, báo cáo đồ hoạ, thiết kế bản in |
| 🚀 | Dashboard designer | Kéo–thả tạo dashboard, hàng chục loại chart có sẵn |

### WeChat Official Account

| | Chức năng | Mô tả |
|---|---|---|
| 🚀 | Quản lý tài khoản | Cấu hình Official Account, hỗ trợ nhiều tài khoản |
| 🚀 | Thống kê | Thống kê tăng/giảm user, tổng user, message, phân tích API |
| 🚀 | Quản lý follower | Xem danh sách follower/following; đồng bộ, gắn tag |
| 🚀 | Quản lý tin nhắn | Xem tin nhắn follower; chủ động trả lời |
| 🚀 | Template message | Cấu hình và gửi template message để push thông báo |
| 🚀 | Auto-reply | Tự trả lời: khi follow, khi nhận tin, theo keyword |
| 🚀 | Quản lý tag | CRUD tag của Official Account |
| 🚀 | Quản lý menu | Tuỳ biến menu; đồng bộ menu từ Official Account |
| 🚀 | Quản lý material | Quản lý ảnh, audio, video; phát online |
| 🚀 | Draft bài viết | Lưu bài viết thường dùng vào draft, có thể publish |
| 🚀 | Lịch sử publish | Xem bài đã publish, có thể xoá |

### Hệ thống thương mại điện tử (Mall)

Demo: <https://doc.iocoder.cn/mall-preview/>

![Sơ đồ chức năng](/.image/common/mall-feature.png)

![Preview](/.image/common/mall-preview.png)

### Trung tâm hội viên

| | Chức năng | Mô tả |
|---|---|---|
| 🚀 | Quản lý hội viên | Hội viên là người tiêu dùng phía C; tìm kiếm và quản lý |
| 🚀 | Tag hội viên | CRUD tag hội viên |
| 🚀 | Cấp hội viên | Quản lý cấp và điểm成长 (growth); dùng cho chiết khấu đơn hàng |
| 🚀 | Nhóm hội viên | Phân nhóm để vẽ chân dung người dùng, push nội dung |
| 🚀 | Điểm danh & tích điểm | Tích điểm theo hành vi; dùng điểm để trừ tiền hoặc đổi quà |

### ERP

Demo: <https://doc.iocoder.cn/erp-preview/>

![Sơ đồ chức năng](/.image/common/erp-feature.png)

### WMS

Demo: <https://doc.iocoder.cn/wms-preview/>

![Sơ đồ chức năng](/.image/common/wms-feature.png)

![Preview](/.image/common/wms-preview.png)

### CRM

Demo: <https://doc.iocoder.cn/crm-preview/>

![Sơ đồ chức năng](/.image/common/crm-feature.png)

### MES

Demo: <https://doc.iocoder.cn/mes-preview/>

![Sơ đồ chức năng](/.image/common/mes-feature.png)

![Preview](/.image/common/mes-preview.png)

### HRM — Quản lý nhân sự

Demo: <https://doc.iocoder.cn/hrm-preview/>

![Sơ đồ chức năng](/.image/common/hrm-feature.png)

![Preview](/.image/common/hrm-preview.png)

### FMS — Quản lý tài chính

Demo: <https://doc.iocoder.cn/fms-preview/>

![Sơ đồ chức năng](/.image/common/fms-feature.png)

![Preview](/.image/common/fms-preview.png)

### PMS — Quản lý dự án

Demo: <https://doc.iocoder.cn/pms-preview/>

![Sơ đồ chức năng](/.image/common/pms-feature.png)

![Preview](/.image/common/pms-preview.png)

### OA — Cộng tác văn phòng

Demo: <https://doc.iocoder.cn/oa-preview/>

![Sơ đồ chức năng](/.image/common/oa-feature.png)

![Preview](/.image/common/oa-preview.png)

### AI / LLM

Demo: <https://doc.iocoder.cn/ai-preview/>

![Sơ đồ chức năng](/.image/common/ai-feature.png)

![Preview](/.image/common/ai-preview.gif)

### IoT

Demo: <https://doc.iocoder.cn/iot/build>

![Sơ đồ chức năng](/.image/common/iot-feature.png)

![Preview](/.image/common/iot-preview.png)

### IM — Chat

Demo (Boot): <https://doc.iocoder.cn/im-preview/>

Demo (Vue3 + Element Plus): <http://dashboard-vue3.yudao.iocoder.cn>

![Sơ đồ chức năng](/.image/common/im-feature.png)

| Giao diện chat | Quản lý chat |
|---|---|
| ![Giao diện chat](/.image/common/im-preview-home.png) | ![Quản lý chat](/.image/common/im-preview-manager.png) |

## 🐨 Tech stack

### Module

| Dự án | Mô tả |
|---|---|
| `yudao-dependencies` | Quản lý phiên bản dependency Maven |
| `yudao-framework` | Phần mở rộng framework Java |
| `yudao-server` | Server cho admin + app người dùng |
| `yudao-module-system` | Module chức năng hệ thống |
| `yudao-module-infra` | Module hạ tầng |
| `yudao-module-bpm` | Module quy trình nghiệp vụ |
| `yudao-module-pay` | Module thanh toán |
| `yudao-module-report` | Module báo cáo dashboard |
| `yudao-module-mp` | Module WeChat Official Account |
| `yudao-module-mall` | Module thương mại điện tử |
| `yudao-module-member` | Module trung tâm hội viên |
| `yudao-module-erp` | Module ERP |
| `yudao-module-wms` | Module WMS |
| `yudao-module-crm` | Module CRM |
| `yudao-module-mes` | Module MES |
| `yudao-module-hrm` | Module HRM |
| `yudao-module-fms` | Module FMS |
| `yudao-module-pms` | Module PMS |
| `yudao-module-oa` | Module OA |
| `yudao-module-ai` | Module AI/LLM |
| `yudao-module-iot` | Module IoT |
| `yudao-module-im` | Module IM |

### Framework

| Framework | Mô tả | Phiên bản | Tài liệu |
|---|---|---|---|
| [Spring Boot](https://spring.io/projects/spring-boot) | Framework phát triển ứng dụng | 2.7.18 | [Tài liệu](https://github.com/YunaiV/SpringBoot-Labs) |
| [MySQL](https://www.mysql.com/cn/) | Database server | 5.7 / 8.0+ | |
| [Druid](https://github.com/alibaba/druid) | Connection pool JDBC, monitoring | 1.2.28 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/datasource-pool/?yudao) |
| [MyBatis Plus](https://mp.baomidou.com/) | Bộ công cụ tăng cường MyBatis | 3.5.16 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/MyBatis/?yudao) |
| [Dynamic Datasource](https://dynamic-datasource.com/) | Data source động | 4.5.0 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/datasource-pool/?yudao) |
| [Redis](https://redis.io/) | Database key-value | 5.0 / 6.0 / 7.0 | |
| [Redisson](https://github.com/redisson/redisson) | Redis client | 4.6.1 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/Redis/?yudao) |
| [Spring MVC](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc) | Framework MVC | 5.3.39 | [Tài liệu](http://www.iocoder.cn/SpringMVC/MVC/?yudao) |
| [Spring Security](https://github.com/spring-projects/spring-security) | Framework bảo mật | 5.8.16 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/Spring-Security/?yudao) |
| [Hibernate Validator](https://github.com/hibernate/hibernate-validator) | Validate tham số | 6.2.5 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/Validation/?yudao) |
| [Flowable](https://github.com/flowable/flowable-engine) | Workflow engine | 6.8.1 | [Tài liệu](https://doc.iocoder.cn/bpm/) |
| [Quartz](https://github.com/quartz-scheduler) | Job scheduler | 2.3.2 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/Job/?yudao) |
| [Springdoc](https://springdoc.org/) | Tài liệu Swagger | 1.8.0 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/Swagger/?yudao) |
| [SkyWalking](https://skywalking.apache.org/) | Hệ thống tracing phân tán | 8.12.0 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/SkyWalking/?yudao) |
| [Spring Boot Admin](https://github.com/codecentric/spring-boot-admin) | Nền tảng monitoring Spring Boot | 2.7.15 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/Admin/?yudao) |
| [Jackson](https://github.com/FasterXML/jackson) | Thư viện JSON | 2.13.5 | |
| [MapStruct](https://mapstruct.org/) | Chuyển đổi Java Bean | 1.6.3 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/MapStruct/?yudao) |
| [Lombok](https://projectlombok.org/) | Giảm code Java dài dòng | 1.18.46 | [Tài liệu](http://www.iocoder.cn/Spring-Boot/Lombok/?yudao) |
| [JUnit](https://junit.org/junit5/) | Framework unit test | 5.8.2 | - |
| [Mockito](https://github.com/mockito/mockito) | Framework mock | 4.11.0 | - |

## 🐷 Ảnh demo

### Chức năng hệ thống

| Module | Ảnh | Ảnh | Ảnh |
|---|---|---|---|
| Đăng nhập & trang chủ | ![Đăng nhập](/.image/登录.jpg) | ![Trang chủ](/.image/首页.jpg) | ![Trang cá nhân](/.image/个人中心.jpg) |
| Người dùng & ứng dụng | ![Quản lý người dùng](/.image/用户管理.jpg) | ![Quản lý token](/.image/令牌管理.jpg) | ![Quản lý ứng dụng](/.image/应用管理.jpg) |
| Tenant & gói | ![Quản lý tenant](/.image/租户管理.jpg) | ![Gói tenant](/.image/租户套餐.png) | - |
| Phòng ban & chức danh | ![Phòng ban](/.image/部门管理.jpg) | ![Chức danh](/.image/岗位管理.jpg) | - |
| Menu & vai trò | ![Menu](/.image/菜单管理.jpg) | ![Vai trò](/.image/角色管理.jpg) | - |
| Audit log | ![Log thao tác](/.image/操作日志.jpg) | ![Log đăng nhập](/.image/登录日志.jpg) | - |
| SMS | ![Kênh SMS](/.image/短信渠道.jpg) | ![Template SMS](/.image/短信模板.jpg) | ![Log SMS](/.image/短信日志.jpg) |
| Dict & từ nhạy cảm | ![Loại dict](/.image/字典类型.jpg) | ![Dữ liệu dict](/.image/字典数据.jpg) | ![Từ nhạy cảm](/.image/敏感词.jpg) |
| Mã lỗi & thông báo | ![Mã lỗi](/.image/错误码管理.jpg) | ![Thông báo](/.image/通知公告.jpg) | - |

### Quy trình nghiệp vụ

| Module | Ảnh | Ảnh | Ảnh |
|---|---|---|---|
| Process model | ![Danh sách](/.image/流程模型-列表.jpg) | ![Thiết kế](/.image/流程模型-设计.jpg) | ![Định nghĩa](/.image/流程模型-定义.jpg) |
| Form & nhóm | ![Form](/.image/流程表单.jpg) | ![Nhóm người dùng](/.image/用户分组.jpg) | - |
| Quy trình của tôi | ![Danh sách](/.image/我的流程-列表.jpg) | ![Khởi tạo](/.image/我的流程-发起.jpg) | ![Chi tiết](/.image/我的流程-详情.jpg) |
| Việc cần làm & đã làm | ![Duyệt](/.image/任务列表-审批.jpg) | ![Cần làm](/.image/任务列表-待办.jpg) | ![Đã làm](/.image/任务列表-已办.jpg) |
| OA nghỉ phép | ![Danh sách](/.image/OA请假-列表.jpg) | ![Khởi tạo](/.image/OA请假-发起.jpg) | ![Chi tiết](/.image/OA请假-详情.jpg) |

### Hạ tầng

| Module | Ảnh | Ảnh | Ảnh |
|---|---|---|---|
| Codegen | ![Codegen](/.image/代码生成.jpg) | ![Kết quả](/.image/生成效果.jpg) | - |
| Tài liệu | ![API](/.image/系统接口.jpg) | ![Tài liệu DB](/.image/数据库文档.jpg) | - |
| File & cấu hình | ![Cấu hình file](/.image/文件配置.jpg) | ![Quản lý file](/.image/文件管理2.jpg) | ![Cấu hình](/.image/配置管理.jpg) |
| Job | ![Job](/.image/定时任务.jpg) | ![Log job](/.image/任务日志.jpg) | - |
| Log API | ![Log truy cập](/.image/访问日志.jpg) | ![Log lỗi](/.image/错误日志.jpg) | - |
| MySQL & Redis | ![MySQL](/.image/MySQL.jpg) | ![Redis](/.image/Redis.jpg) | - |
| Monitoring | ![Giám sát Java](/.image/Java监控.jpg) | ![Tracing](/.image/链路追踪.jpg) | ![Log center](/.image/日志中心.jpg) |

### Thanh toán

| Module | Ảnh | Ảnh | Ảnh |
|---|---|---|---|
| Merchant & ứng dụng | ![Merchant](/.image/商户信息.jpg) | ![Danh sách app](/.image/应用信息-列表.jpg) | ![Sửa app](/.image/应用信息-编辑.jpg) |
| Thanh toán & hoàn tiền | ![Đơn thanh toán](/.image/支付订单.jpg) | ![Đơn hoàn tiền](/.image/退款订单.jpg) | --- |

### Báo cáo dữ liệu

| Module | Ảnh | Ảnh | Ảnh |
|---|---|---|---|
| Report designer | ![Báo cáo](/.image/报表设计器-数据报表.jpg) | ![Đồ hoạ](/.image/报表设计器-图形报表.jpg) | ![Thiết kế in](/.image/报表设计器-打印设计.jpg) |
| Dashboard designer | ![Danh sách](/.image/大屏设计器-列表.jpg) | ![Preview](/.image/大屏设计器-预览.jpg) | ![Sửa](/.image/大屏设计器-编辑.jpg) |

### Mobile (admin)

| Ảnh | Ảnh | Ảnh |
|---|---|---|
| ![](/.image/admin-uniapp/01.png) | ![](/.image/admin-uniapp/02.png) | ![](/.image/admin-uniapp/03.png) |
| ![](/.image/admin-uniapp/04.png) | ![](/.image/admin-uniapp/05.png) | ![](/.image/admin-uniapp/06.png) |
| ![](/.image/admin-uniapp/07.png) | ![](/.image/admin-uniapp/08.png) | ![](/.image/admin-uniapp/09.png) |

Hiện đã có: đăng nhập, trang cá nhân, workbench, sửa hồ sơ, đổi avatar, đổi mật khẩu, câu hỏi thường gặp, about us…

---

## 📌 Ghi chú của bản dịch

1. **Nhánh `master` chỉ build `yudao-module-system` + `yudao-module-infra`.** Trong `pom.xml` gốc và `yudao-server/pom.xml`, 18 module còn lại đang bị **comment out**. Muốn dùng module nào phải mở comment ở **cả hai** file. Chi tiết: [`docs/vi/01-kien-truc.md`](docs/vi/01-kien-truc.md).
2. Tên class, package, bảng, cột, API, enum **giữ nguyên tiếng Anh** — không dịch.
3. Các đường dẫn ảnh `/.image/...` giữ nguyên như bản gốc.
4. Bản dịch này chỉ thêm file mới, **không sửa code** — để vẫn merge được từ upstream.
5. Giấy phép gốc: **MIT**. Bản dịch là tác phẩm phái sinh, vẫn giữ nguyên MIT và ghi công tác giả gốc.
