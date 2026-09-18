# 🏭 BeanFactory và ApplicationContext

> **Thuộc roadmap:** `2. Spring Framework → Core → Bean Factory & Application Context`

---

## 1. Trái tim của IoC Container

Ở bài Inversion of Control, bạn đã biết IoC Container là một cái "Thùng chứa" ảo giúp khởi tạo và quản lý toàn bộ các Bean trong hệ thống.
Nhưng thực chất, để hiện thực hóa cái "Thùng chứa" đó, Spring Framework cung cấp cho bạn **2 cái máy bơm** khác nhau. Tùy vào việc bạn chọn máy bơm nào, hệ thống của bạn sẽ chạy theo cách khác biệt hoàn toàn.

Đó chính là: `BeanFactory` (Máy bơm chạy bằng cơm) và `ApplicationContext` (Máy bơm tự động hóa toàn phần).

### 1.1. BeanFactory (Máy bơm chạy bằng cơm)
- **Định nghĩa:** Là Interface cốt lõi và thô sơ nhất của Spring IoC.
- **Tính cách: Cực kỳ lười biếng (Lazy Initialization).** Khi Server vừa bật lên, BeanFactory sẽ... không làm gì cả. Nó chỉ đứng đó đọc cấu hình. Nó tuyệt đối không thèm tạo ra một Bean nào cho đến khi nào lập trình viên viết code gọi tên Bean đó ra xài (`factory.getBean()`).
- **Ưu điểm:** Khởi động Server nhanh như chớp. Tiết kiệm RAM vì chưa có Object nào được tạo ra.
- **Nhược điểm:** Chậm ở lần đầu tiên truy cập (vì lúc đó máy mới lạch cạch đi tạo Object). Thiếu hàng tá tính năng nâng cao. Không phù hợp với Web Server (Tomcat).

### 1.2. ApplicationContext (Máy bơm tự động hóa)
- **Định nghĩa:** Kế thừa trực tiếp từ BeanFactory. Mọi thứ BeanFactory làm được, ApplicationContext làm được, và còn cộng thêm hàng núi tính năng doanh nghiệp xịn xò.
- **Tính cách: Nhà giàu lo xa (Eager Initialization).** Ngay khoảnh khắc Server vừa bật lên, ApplicationContext sẽ cắm đầu chạy đi tạo ra toàn bộ 100% các Bean (Singleton) có trong dự án, chuẩn bị sẵn sàng mọi thứ.
- **Ưu điểm:** Lần đầu tiên truy cập nhanh thần tốc vì mọi Bean đã nằm sẵn trên RAM. Tích hợp AOP dễ dàng. Cung cấp chức năng đa ngôn ngữ, và quan trọng nhất là chức năng phát và nghe Sự kiện (Event). Nó chính là tiêu chuẩn vàng của thế giới Spring Boot hiện đại.
- **Nhược điểm:** Nếu dự án quá khổng lồ (vài ngàn class), khởi động Server có thể mất vài phút và ngốn rất nhiều RAM ngay từ đầu.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

Trong 99,99% các dự án Web hiện đại bằng Spring Boot, bạn sẽ **KHÔNG BAO GIỜ** xài BeanFactory. ApplicationContext là bá chủ độc quyền. Tuy nhiên, nếu bạn giữ nguyên cấu hình mặc định của ApplicationContext (tạo mọi thứ từ đầu), bạn sẽ gặp một rắc rối siêu to khổng lồ.

### Vấn đề: Trải nghiệm Lập trình viên (Developer Experience) tồi tệ
Dự án của ngân hàng bạn có 10.000 file Service. Mỗi lần bạn sửa đúng 1 dòng code, bạn nhấn nút Restart Server, bạn phải ngồi nhìn màn hình console chạy mất 5 phút mới load xong 10.000 cái Bean đó vào RAM. Quá trình phát triển bị kìm hãm nặng nề!

### Giải pháp thực tế: Ép ApplicationContext "Lười biếng" giống như BeanFactory
Thay vì phải đập bỏ ApplicationContext quay về xài BeanFactory (và mất đi các tính năng xịn xò khác), Spring Boot cung cấp một câu cấu hình ma thuật trong file `application.yml` để bắt thằng ApplicationContext tạm thời đổi tính cách thành Lười biếng (Lazy) ở môi trường máy cá nhân.

```yaml
spring:
  main:
    lazy-initialization: true # 👈 Câu thần chú ma thuật
```
**Giá trị mang lại:** 
Khi gắn lệnh này vào file `application-dev.yml` (chỉ chạy ở máy DEV), Server của bạn sẽ khởi động lên trong đúng 2 giây thay vì 5 phút! Bởi vì ApplicationContext không thèm khởi tạo 10.000 class kia nữa, nó sẽ đợi khi nào bạn dùng Postman bắn request vào Controller nào, thì nó mới tạo Bean cho cái Controller đó.
(Tuy nhiên, tuyệt đối KHÔNG ĐƯỢC bật tính năng này trên PROD, vì trên PROD chúng ta thà đợi 5 phút lúc khởi động còn hơn bắt khách hàng đầu tiên truy cập phải đợi 5 giây).

---

## 3. Tổng hợp các chủ đề về ApplicationContext (Bản đồ tra cứu)

Hãy tra cứu các từ vựng dưới đây khi bạn muốn trở thành Master (Bậc thầy) điều khiển hệ thống Spring từ gốc rễ:

### 🌟 3.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Rất hay dùng)
1. **Lý thuyết Eager vs Lazy Initialization:** Bắt buộc phải thuộc lòng sự khác biệt giữa "Tạo ngay lúc khởi động" và "Chờ đến lúc xài mới tạo" (Như đã giải thích ở trên).
2. **`ApplicationContext` là cái bao trùm tất cả:** Trong Spring Boot, bạn gần như không bao giờ phải viết code khởi tạo container này bằng tay. Spring Boot tự động làm ngầm bằng class `SpringApplication.run(...)`.

### 🟡 3.2. Nhóm Tính năng Doanh nghiệp (Trung cấp)
ApplicationContext mang lại 3 sức mạnh vĩ đại mà BeanFactory không có. Bạn cần biết để áp dụng:
1. **`ApplicationEventPublisher` (Event/Listener):** Chức năng tuyệt vời nhất! Cung cấp khả năng giao tiếp bất đồng bộ lỏng lẻo (decoupled). 
   - Thay vì `UserService` gọi trực tiếp `EmailService.send()`, `UserService` chỉ cần báo cho ApplicationContext: *"Có user đăng ký"*. Mọi thằng đang nghe (Listener) sẽ tự động chạy.
2. **`MessageSource` (Đa ngôn ngữ - i18n):** Khả năng đọc các file `.properties` chứa nhiều ngôn ngữ khác nhau để tự động dịch thông báo lỗi sang Tiếng Anh/Việt.
3. **Móc nối tự động AOP:** ApplicationContext tự động kích hoạt Proxy và kết nối các Aspect của bạn với các Bean, trong khi BeanFactory phải làm thủ công.

### 🔴 3.3. Nhóm Kỹ thuật Nâng cao (Khởi tạo hệ thống bằng tay)
Đây là kiến thức đồ cổ từ thời chưa có Spring Boot. Nếu bạn gặp các dự án Spring thuần (Legacy), bạn phải biết cách tự gọi hàm khởi tạo ApplicationContext:
1. **`AnnotationConfigApplicationContext`:** Máy bơm hiện đại nhất, nhận vào một class `@Configuration` để quét tìm Bean.
2. **`ClassPathXmlApplicationContext`:** Máy bơm cổ đại, nhận vào một file cấu hình định dạng XML đồ sộ rườm rà. (Rất nhiều bài test phỏng vấn Java vẫn còn hỏi cái này).

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Spring 7.0.8 — BeanFactory vs ApplicationContext | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html) |
| Spring Boot 4.1.0 — Lazy Initialization | [docs.spring.io](https://docs.spring.io/spring-boot/reference/core/spring-application.html#core.spring-application.lazy-initialization) |
