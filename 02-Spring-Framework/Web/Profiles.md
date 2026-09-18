# 🌍 Môi trường triển khai (Spring Profiles)

> **Thuộc roadmap:** `2. Spring Framework → Web → Profiles`

---

## 1. Spring Profiles là gì?

Trong vòng đời phát triển phần mềm, ứng dụng của bạn không chỉ chạy trên máy tính cá nhân của bạn. Nó sẽ di chuyển qua nhiều môi trường (Environment) khác nhau trước khi đến tay người dùng cuối. Thường có 3 môi trường chính:
- **DEV (Development):** Môi trường lập trình trên máy cá nhân của bạn (cần kết nối DB ảo chạy trên máy tính, in ra nhiều log lỗi để dễ tìm bug).
- **STG (Staging/Test):** Môi trường kiểm thử trên server nội bộ để đội Tester (QA) vào test (cần kết nối DB test dùng chung của cả team).
- **PROD (Production):** Môi trường chạy thật, nơi khách hàng đang sử dụng (cần kết nối DB thật, tối ưu hiệu năng tối đa, tắt log lỗi chi tiết để tránh lộ thông tin bảo mật).

**Vấn đề:** Không lẽ mỗi lần đưa code lên một môi trường mới, bạn lại phải mở file cấu hình ra, xóa dòng chứa mật khẩu DB cũ rồi gõ mật khẩu DB mới bằng tay? Điều này cực kỳ rủi ro (lỡ tay gõ sai chữ) và vô cùng phiền toái.

**Giải pháp của Spring:** Đó chính là khái niệm **Profiles**. Spring cho phép bạn gom nhóm các cấu hình (hoặc các Bean) lại và gắn mác cho chúng (ví dụ: mác `dev`, mác `prod`). Khi chạy ứng dụng, bạn chỉ cần hô to với server: *"Hãy chạy ở chế độ PROD!"*, Spring sẽ tự động kích hoạt tất cả các cấu hình thuộc mác `prod` và tự động làm ngơ mác `dev`.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Hãy cùng xem xét 2 cách áp dụng Profiles đỉnh cao nhất mà mọi công ty đều làm.*

### Thực tế 1: Tách biệt file cấu hình YAML (application.yml)
Thay vì nhét mọi mật khẩu của cả 3 môi trường vào chung một file `application.yml` duy nhất, bạn sẽ chia nó ra làm nhiều file tương ứng với từng Profile. Spring Boot tự động nhận diện quy tắc đặt tên file là `application-{tên-profile}.yml`.

**1. File gốc: `application.yml` (Chứa cấu hình chung không thay đổi)**
```yaml
spring:
  application:
    name: "My Awesome App"
  # Dòng lệnh quyết định Profile nào đang được kích hoạt làm Vua!
  profiles:
    active: dev # Mặc định đang ở chế độ DEV (lập trình viên)
```

**2. File môi trường DEV: `application-dev.yml`**
```yaml
# Kết nối DB trên máy tính cá nhân, log ra mọi câu lệnh SQL
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/local_db
    username: root
    password: 123
  jpa:
    show-sql: true # Bật tính năng in SQL ra màn hình để DEV dễ tìm lỗi
```

**3. File môi trường PROD: `application-prod.yml`**
```yaml
# Kết nối DB của công ty, cấm tuyệt đối việc in log SQL để bảo mật
spring:
  datasource:
    url: jdbc:mysql://192.168.1.100:3306/company_real_db
    username: admin_secret
    password: siêu-bảo-mật-6969
  jpa:
    show-sql: false # Tắt để tối ưu tốc độ CPU và bảo mật
```
**Quy trình thực tế:** Trên máy tính cá nhân, file gốc đang chĩa về `dev`, bạn cứ nhấn nút Run bình thường. Khi bàn giao ứng dụng cho đội DevOps đẩy lên Server thật, họ chỉ cần chạy lệnh khởi động file `.jar` kèm theo cái cờ (flag) trên Terminal:
```bash
java -jar my-app.jar --spring.profiles.active=prod
```
Ứng dụng sẽ ngay lập tức tự lấy file `prod` để chạy mà **bạn không cần phải mở IDE ra sửa một dòng code Java nào**. Rất an toàn!

### Thực tế 2: Bật/Tắt các Bean bằng `@Profile`
Đôi khi, không chỉ là đổi mật khẩu, mà bạn muốn **đổi cả class xử lý logic** tùy theo môi trường.
Ví dụ: Ở môi trường thật, bạn gọi API gửi tin nhắn SMS tốn tiền. Ở môi trường code, bạn chỉ muốn in ra màn hình console giả vờ là đã gửi để tiết kiệm tiền.

```java
// Class này CHỈ được IoC Container tạo ra nếu đang chạy ở chế độ 'prod'
@Service
@Profile("prod")
public class RealSmsService implements SmsService {
    public void send(String phone) {
        System.out.println("Đang trừ tiền và gọi API mạng Viettel gửi tới " + phone);
    }
}

// Class này CHỈ được tạo ra nếu đang chạy ở chế độ 'dev'
@Service
@Profile("dev")
public class FakeSmsService implements SmsService {
    public void send(String phone) {
        System.out.println("Môi trường DEV: Giả vờ đã gửi SMS tới " + phone + " để khỏi tốn tiền!");
    }
}
```
Nhờ `@Profile`, IoC Container sẽ tự động chọn đúng class tương ứng với môi trường hiện tại để `Inject` vào Controller. Tính năng này cứu sống các công ty hàng ngàn đô la chi phí gọi API ngoài (SMS, Cổng thanh toán, Gửi Email) khi test.

---

## 3. Tổng hợp các chủ đề Profiles (Bản đồ tra cứu)

### 🌟 3.1. Những khái niệm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc khi đi làm)
1. **Quy tắc nạp chồng file (Property Override):** Bắt buộc phải hiểu cách Spring đọc cấu hình. Nó sẽ đọc file `application.yml` gốc trước, rồi lấy các giá trị trong file profile cụ thể (ví dụ `application-dev.yml`) đè lên các giá trị gốc.
2. **Kích hoạt Profile qua biến môi trường (Environment Variable):** Bắt buộc phải biết cách bật profile `prod` khi chạy file `.jar` ngoài Terminal bằng lệnh `java -jar ... --spring.profiles.active=prod`.
3. **Annotation `@Profile`:** Dùng để bật/tắt động toàn bộ một class `@Configuration` hoặc một `@Service` cụ thể (như ví dụ Fake SMS ở trên).

### 🟡 3.2. Nhóm kiến trúc Trung cấp (Tùy biến hành vi hệ thống)
1. **Profile Groups:** Kỹ thuật gom nhiều profile nhỏ lẻ thành một profile lớn. Ví dụ: khi bạn hô to kích hoạt profile tên là `production`, Spring sẽ tự động hiểu ngầm là kích hoạt kèm theo một chùm các profile con `proddb`, `prodsecurity`, `prodlogging`.
2. **`!profile` (Toán tử NOT):** Kích hoạt một Bean ở mọi môi trường *NGOẠI TRỪ* một môi trường cụ thể. (Ví dụ: `@Profile("!prod")` nghĩa là Bean FakeSms này sẽ được phép chạy ở môi trường dev, test nhưng tuyệt đối cấm không được sinh ra ở prod).

### 🔴 3.3. Nhóm kiến trúc Nâng cao (Tùy biến sâu & Kiến trúc Đám mây)
1. **`@ActiveProfiles` trong Unit Testing:** Khi viết code tự động Test (JUnit/Mockito), bạn cần phải biết dùng annotation này gắn trên đầu class Test để ép Spring giả lập đang chạy ở môi trường `dev` hoặc `test`.
2. **Đọc cấu hình từ Spring Cloud Config Server:** Ở kỷ nguyên Microservices (hàng chục service phân tán), người ta không thèm lưu file YAML trong từng ổ cứng nữa. Người ta xây dựng hẳn một Server Trung Tâm chuyên chứa file cấu hình, và cấp phát Profiles qua mạng nội bộ cho các service khác.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Spring Boot 4.1.0 — Profiles | [docs.spring.io](https://docs.spring.io/spring-boot/reference/features/profiles.html) |
| Spring 7.0.8 — Bean Definition Profiles | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/beans/environment.html#beans-definition-profiles) |

---

> **Tiếp theo trong roadmap:** `AOP (Aspect-Oriented Programming)` - Chúng ta sẽ chính thức rời khỏi phần Web và đi vào học **trụ cột sức mạnh thứ 2** của Spring Framework (bên cạnh IoC), đó là nghệ thuật phân tách logic cắt ngang.
