# Lộ trình tự học 2 tuần: chuẩn bị cho dự án Hotel Booking (Java/Spring Boot)

## Context

Mentor giao tự nghiên cứu trước stack: Java 21-25, Spring Boot, Gradle, Flyway, RESTful API, Unit Test, IAM/SSO (PKCE), Multi-tenant, Mobile push notification, nghiệp vụ du lịch/khách sạn, Sequence Diagram. Người học chỉ học Java core sơ bộ 2 năm trước và gần như quên hết, repo `hotel-booking-system` hiện đang trống (chưa có code). Có ~2 tuần, 3-5 giờ/ngày (~42-70 giờ), nên mục tiêu không phải "thành thạo mọi thứ" mà là: **đủ nền tảng Java/Spring Boot để code được, và đủ hiểu biết khái niệm ở các phần khó (IAM/SSO, multi-tenant, push notification) để không bị choáng khi dự án thực sự bắt đầu.**

Chiến lược: ưu tiên chắc Java + Spring Boot core trước (vì đây là nền làm việc hàng ngày), các phần còn lại học ở mức "hiểu đúng khái niệm + đọc được sequence diagram liên quan", không cần tự implement hoàn chỉnh trong 2 tuần.

## Lộ trình theo ngày

### Ngày 1-3: Java core refresher (nhanh, vì đã học qua)
- Ôn: OOP (class/interface/abstract, inheritance, polymorphism), Collections (List/Map/Set), Generics, Lambda & Stream API, Exception handling, Optional.
- Mới (nếu chưa biết): `record`, `sealed class`, `switch` pattern matching, `var` — các feature Java 17-21 hay dùng trong code Spring Boot hiện đại.
- Tài nguyên: Java official tutorial (docs.oracle.com/javase/tutorial), hoặc khóa "Java OOP crash course" bất kỳ trên YouTube. Baeldung.com có bài riêng cho từng feature Java mới.
- Thực hành: viết vài class nhỏ (ví dụ model `Room`, `Booking`, `Guest`) dùng record + stream để lọc/group dữ liệu mẫu, không cần Spring.

### Ngày 4-8: Spring Boot + Gradle + REST API + Flyway + Unit Test (trọng tâm, học qua thực hành)
- Setup: tạo project Spring Boot bằng Gradle (start.spring.io), hiểu cấu trúc `build.gradle`, dependency injection, annotation cơ bản (`@Component`, `@Service`, `@RestController`, `@Repository`).
- REST API: dựng CRUD đơn giản cho 2 entity thuộc domain khách sạn — ví dụ `Room` (phòng) và `Booking` (đặt phòng). Học `@RestController`, `@RequestMapping`, DTO vs Entity, validation (`@Valid`), xử lý lỗi (`@ExceptionHandler`).
- Database + Flyway: dùng Postgres/H2, viết migration SQL đầu tiên (`V1__init.sql`) tạo bảng `room`, `booking`; hiểu versioning của Flyway (vì sao không sửa migration cũ mà luôn thêm file mới).
- Unit Test: viết test cho service layer bằng JUnit 5 + Mockito (mock repository), và 1-2 test tích hợp cho REST endpoint bằng `@SpringBootTest`/`MockMvc`.
- Tài nguyên: Spring.io official guides (spring.io/guides — "Building a RESTful Web Service", "Accessing Data with JPA"), Baeldung cho Flyway + Spring Boot, Flyway docs chính thức.
- Sản phẩm cuối giai đoạn: 1 REST API nhỏ chạy được — tạo phòng, đặt phòng, xem danh sách, có test pass, có migration Flyway. Đây là "hotel booking mini" dùng để học các phần sau luôn trên chính project này.

### Ngày 9-10: Sequence Diagram + nghiệp vụ hotel booking
- Học cách đọc/vẽ sequence diagram: actor, lifeline, message (sync/async), activation box, alt/opt block.
- Công cụ: PlantUML (text-based, dễ version control cùng code) hoặc draw.io.
- Thực hành: vẽ 3 sequence diagram cho chính nghiệp vụ sẽ làm — (1) tìm phòng trống theo ngày, (2) đặt phòng (check inventory → tạo booking → trả kết quả), (3) hủy booking. Việc vẽ cũng chính là lúc học nghiệp vụ: trạng thái phòng (available/booked/blocked), overbooking, chính sách hủy/hoàn tiền.
- Nếu có thời gian: tìm hiểu thêm luồng mua vé (ticket) nếu dự án có đụng tới, theo cùng cách.

### Ngày 11-12: IAM/SSO, OAuth2 + PKCE (học khái niệm, không cần tự dựng IAM server)
- Hiểu vai trò: Identity Provider (IdP) vs client app vs resource server. OAuth2 Authorization Code Flow, vì sao cần PKCE (chống bị đánh cắp authorization code, đặc biệt với mobile/SPA không giữ được client secret an toàn).
- V-ID: đọc tài liệu/docs mà công ty cung cấp về V-ID nếu có (hỏi mentor nếu chưa có) — vì đây là SSO provider cụ thể sẽ dùng, khác với OAuth2 lý thuyết chung.
- Thực hành tối thiểu: thử 1 demo Spring Boot dùng Spring Security OAuth2 Client để login qua 1 provider test (Google/Okta free tier) — mục đích là thấy luồng redirect, callback, token thật chạy, không cần hiểu sâu toàn bộ Spring Security.
- Tài nguyên: oauth.net/2 (giải thích PKCE rất rõ), Spring Security OAuth2 reference docs.

### Ngày 13: Multi-tenant (khái niệm + trade-off, chưa cần implement)
- 3 chiến lược chính: DB riêng mỗi tenant, schema riêng mỗi tenant (cùng DB), hoặc 1 schema dùng chung + cột `tenant_id` phân biệt (discriminator). Đọc ưu/nhược điểm từng cách (isolation vs chi phí vận hành vs độ phức tạp migration).
- Liên hệ với Flyway đã học: multi-tenant schema-per-tenant ảnh hưởng thế nào đến việc chạy migration (phải chạy cho từng schema).
- Không cần code, chỉ cần hiểu đủ để thảo luận với mentor khi dự án quyết định chọn hướng nào.

### Ngày 14: Mobile push notification (khái niệm, góc nhìn notification service) + tổng ôn
- Hiểu ở mức kiến trúc: app mobile đăng ký device token với FCM (Android)/APNs (iOS) → backend lưu token theo user → khi cần gửi, backend gọi FCM/APNs API với token đó → không cần biết code mobile, chỉ cần biết backend/notification service đóng vai trò gì (lưu token, gọi gửi, xử lý token hết hạn/invalid).
- Buffer: quay lại phần nào còn yếu trong 13 ngày trước, chạy lại mini project REST API để chắc tay.

## Nguyên tắc học
- Ưu tiên "học bằng làm": phần Java/Spring Boot phải có code chạy thật, không chỉ đọc.
- Các phần khó (IAM/SSO, multi-tenant, push notification) trong 2 tuần chỉ cần **hiểu đúng khái niệm và trả lời được "vì sao"**, không cần tự dựng hệ thống hoàn chỉnh — những phần này thường do dự án có sẵn hạ tầng/thư viện, việc của người mới là hiểu để tích hợp đúng, không phải xây từ đầu.
- Ghi chú lại câu hỏi phát sinh trong lúc học để hỏi mentor, đặc biệt về V-ID (SSO cụ thể của công ty) và multi-tenant strategy thực tế dự án dùng — hai phần này sách/docs chung sẽ không nói chính xác.

## Kiểm tra kết quả (cuối 2 tuần)
- Chạy được REST API mini (CRUD Room/Booking) từ máy local, có Flyway migration, có ít nhất 5 unit test + 1 integration test pass.
- Vẽ được và giải thích được 3 sequence diagram nghiệp vụ booking.
- Giải thích được bằng lời: OAuth2 Authorization Code + PKCE hoạt động thế nào và vì sao cần PKCE; 3 chiến lược multi-tenant khác nhau ra sao; luồng push notification đi qua những thành phần nào.
