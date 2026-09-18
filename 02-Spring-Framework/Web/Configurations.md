# ⚙️ Cấu hình (Configurations) trong Spring Web

> **Nguồn chính thống:** [Spring Framework 7.0.8 — MVC Config](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-config.html)
>
> **Thuộc roadmap:** `2. Spring Framework → Web → Configurations`

---

## 1. Cấu hình (Configuration) là gì?

Nếu coi ứng dụng Web của bạn là một chiếc xe ô tô, thì **Controllers**, **Services**, **Models** chính là các bộ phận (bánh xe, động cơ, vô lăng). Còn **Configuration (Cấu hình)** chính là những sợi dây điện, ốc vít và bảng mạch trung tâm giúp kết nối, tinh chỉnh tất cả các bộ phận đó lại với nhau để chiếc xe chạy mượt mà và đúng ý người lái.

Trước đây, cấu hình trong Spring được viết bằng các file **XML** khổng lồ và rất khó bảo trì. Kể từ Spring 4 trở đi (đặc biệt là trong Spring 7.0.8 / Spring Boot 4.1.0), **Java-based Configuration (Cấu hình bằng code Java)** đã trở thành tiêu chuẩn vàng.

---

## 2. Nền tảng: `@Configuration` và `@Bean`

Để tạo một file cấu hình trong Spring, bạn chỉ cần tạo một class bình thường và gắn annotation `@Configuration`.

- `@Configuration`: Khai báo với IoC Container rằng "Đây là một bản thiết kế, hãy đọc nó!".
- `@Bean`: Thường được đặt trên các hàm bên trong class cấu hình. Nó chỉ định rằng hàm này sẽ tạo ra một object (Bean) và Spring hãy lấy object đó bỏ vào IoC Container dùng chung.

```java
@Configuration
public class AppConfig {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // Tạo một Bean mã hóa mật khẩu dùng chung cho toàn hệ thống
    }
}
```

---

## 3. WebMvcConfigurer: "Chìa khóa vàng" cấu hình Spring Web

Trong phần lớn các dự án thực tế, cấu hình mặc định của Spring Boot (Auto-configuration) đã quá đủ để ứng dụng chạy. Tuy nhiên, khi bạn muốn **can thiệp sâu vào luồng xử lý Web MVC** (như chặn request, mở khóa kết nối liên miền, đổi định dạng ngày tháng), bạn sẽ cần dùng đến interface `WebMvcConfigurer`.

Khi một class của bạn kế thừa (implements) `WebMvcConfigurer`, bạn có quyền ghi đè (override) các hàm để "độ" lại Spring Web theo ý muốn.

### Các tùy chỉnh Web MVC phổ biến nhất:

1. **CorsRegistry:** Mở khóa bảo mật CORS (để Frontend React/Vue gọi API mà không bị chặn).
2. **InterceptorRegistry:** Tạo bộ lọc (Interceptor) chặn các request đi vào để kiểm tra quyền, hoặc log thông tin.
3. **ResourceHandlerRegistry:** Cấu hình đường dẫn chứa file tĩnh (ảnh, css, js, thư mục chứa file upload).
4. **FormatterRegistry:** Tùy chỉnh cách ép kiểu dữ liệu từ tham số URL (ví dụ biến chuỗi "2023-10-15" thành object `LocalDate`).

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Chúng ta hãy xem một vấn đề cực kỳ phổ biến trong mọi dự án: **Lỗi CORS (Cross-Origin Resource Sharing)**.*

### Vấn đề thực tế 1 (Nỗi đau của Frontend):
Giả sử bạn đang viết backend API tại `http://localhost:8080`.
Đội Frontend dùng React chạy ở `http://localhost:3000`.
Khi React gọi API sang 8080, trình duyệt sẽ **CHẶN ĐỨNG** request này lại và ném ra một lỗi màu đỏ lòm: `CORS policy: No 'Access-Control-Allow-Origin' header is present`. 
*(Vì lý do bảo mật, trình duyệt cấm 2 server/domain khác nhau gọi nhau tự do).*

**Giải quyết bằng Web Configuration:**
Chúng ta cần cấu hình Spring Web "mở cửa" cho domain `http://localhost:3000`. Thay vì đi sửa từng cái Controller một, ta gom vào một file Configuration trung tâm.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    // Mở khóa CORS cho phép React/Vue gọi API thoải mái
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**") // Áp dụng cho mọi API bắt đầu bằng /api/
                .allowedOrigins("http://localhost:3000") // Chỉ cho phép domain của React
                .allowedMethods("GET", "POST", "PUT", "DELETE") // Cho phép các HTTP method này
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```

### Ứng dụng thực tế 2: Tạo "Trạm thu phí" (Interceptor)
Sếp yêu cầu: *"Mọi API gọi vào server phải được ghi lại (log) thời gian xử lý xem có API nào bị chậm không"*.
Thay vì đi copy-paste đoạn code tính thời gian vào 100 cái Controller, ta cấu hình một **Interceptor (Bộ đánh chặn)**:

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

// BƯỚC 1: Tạo cái "Trạm thu phí"
@Component
public class TimeLoggingInterceptor implements HandlerInterceptor {
    
    // Hàm này chạy TRƯỚC KHI request đi vào Controller
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        request.setAttribute("startTime", System.currentTimeMillis());
        return true; // true = Cho phép đi tiếp vào Controller
    }

    // Hàm này chạy SAU KHI Controller đã xử lý xong và view đã render
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        long startTime = (Long) request.getAttribute("startTime");
        System.out.println("URL: " + request.getRequestURI() + " tốn " + (System.currentTimeMillis() - startTime) + "ms");
    }
}
```

```java
// BƯỚC 2: Mang "Trạm thu phí" ra đặt vào cao tốc trong file Configuration
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final TimeLoggingInterceptor timeInterceptor;

    public WebConfig(TimeLoggingInterceptor timeInterceptor) {
        this.timeInterceptor = timeInterceptor;
    }

    @Override
    public void addInterceptors(org.springframework.web.servlet.config.annotation.InterceptorRegistry registry) {
        // Chặn tất cả các request có đường dẫn bắt đầu bằng /api/
        registry.addInterceptor(timeInterceptor)
                .addPathPatterns("/api/**"); 
    }
}
```

**Giá trị thực tiễn mang lại:**
Cấu hình tập trung giúp hệ thống **cực kỳ dễ bảo trì**. Bạn có thể linh hoạt thay đổi chiến lược chung (như bảo mật, hiệu năng, phân quyền) tại một nơi duy nhất mà **không làm rác code** ở phía các tầng xử lý nghiệp vụ (Controller/Service).

---

## 4. Các cấu hình Web nâng cao (Advanced Web Configurations)

Bên cạnh `WebMvcConfigurer`, Spring Web còn rất nhiều cơ chế cấu hình "hạng nặng" khác mà bạn sẽ dùng thường xuyên trong các dự án doanh nghiệp:

### 4.1. Bắt lỗi toàn cục với `@RestControllerAdvice`
Thay vì dùng `try-catch` lặp đi lặp lại trong mọi hàm của Controller, bạn tạo một class `@RestControllerAdvice`. Nó đóng vai trò như một "Bác sĩ trực ban", hễ bất kỳ Controller nào ném ra Lỗi (Exception), nó sẽ nhảy ra hứng lấy và trả về một cục JSON chuẩn mực cho Frontend.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND) // Ép HTTP Status Code về 404
    public Map<String, String> handleUserNotFound(UserNotFoundException ex) {
        // Luôn trả về form JSON đồng nhất cho mọi lỗi
        return Map.of("errorCode", "ERR_404", "message", ex.getMessage());
    }
}
```

### 4.2. Bộ lọc (Servlet Filter) vs Interceptor
Spring Web chạy trên nền tảng Servlet, do đó bạn có thể cấu hình các **Filter** (thường kế thừa `OncePerRequestFilter`). 
- **Filter:** Đứng ở "vòng ngoài cùng", chặn request trước cả khi nó chạm mặt `DispatcherServlet`. Thường dùng cho các tác vụ mang tính hệ thống thấp: Đọc/giải mã token JWT, nén luồng dữ liệu.
- **Interceptor:** Đứng ở "vòng trong", chạy sau khi `DispatcherServlet` đã biết rõ request này thuộc về Controller nào. Thường dùng cho logic nghiệp vụ: Ghi log, kiểm tra quyền xem có được vào hàm abc không.

### 4.3. Cấu hình định dạng JSON (Jackson HttpMessageConverters)
Khi Controller trả về Object, thằng "đệ" Jackson sẽ tự động chuyển nó thành JSON. Đôi khi bạn muốn ép Jackson làm theo ý mình (ví dụ: tự động bỏ qua các trường `null` cho bớt rác, chuẩn hóa lại toàn bộ ngày tháng).
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.annotation.JsonInclude;

@Configuration
public class JacksonConfig {
    @Bean
    public ObjectMapper customJson() {
        return new Jackson2ObjectMapperBuilder()
                .serializationInclusion(JsonInclude.Include.NON_NULL) // Xóa sổ mọi field bị null khỏi JSON
                .simpleDateFormat("yyyy-MM-dd HH:mm:ss") // Chuẩn hóa toàn bộ ngày tháng
                .build();
    }
}
```

---

## 5. Tóm lược về `application.properties` / `yml`
Bên cạnh file Java `@Configuration`, Spring Boot còn cung cấp một kiểu cấu hình dạng file text (key-value) để định nghĩa các thông số môi trường (Environment properties), thường nằm trong file `application.yml` hoặc `application.properties`. 

Mặc dù phần này sẽ được học sâu hơn ở module Spring Boot, nhưng đối với Web, bạn thường hay cấu hình các thông số sau ở file này:
```yaml
server:
  port: 8080 # Đổi cổng chạy web
  servlet:
    context-path: /my-app # Đổi đường dẫn gốc (ví dụ: localhost:8080/my-app/api/...)
```

---

## 6. Tổng hợp các chủ đề Cấu hình Web (Bản đồ tra cứu)

Dưới đây là bức tranh toàn cảnh về hệ sinh thái cấu hình trong Spring Web. Thay vì học hết một lượt, bạn hãy dùng danh sách này như một bản đồ tra cứu khi đi làm thực tế.

### 🌟 6.1. Những cấu hình ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc khi đi làm)
Đây là những kỹ thuật xuất hiện trong 99% các dự án thực tế. Bạn nên **ưu tiên học và thực hành thật kỹ** 4 món này trước tiên:
1. **CORS Configuration:** Bắt buộc phải rành để Frontend và Backend nói chuyện được với nhau mà không bị trình duyệt block.
2. **Global Exception Handling (`@RestControllerAdvice`):** Bắt buộc phải biết để chuẩn hóa cục JSON trả về khi hệ thống văng lỗi.
3. **HttpMessageConverters (Jackson):** Bắt buộc phải biết để làm chủ cách dữ liệu Java Object biến thành JSON (chuẩn hóa ngày tháng, giấu dữ liệu nhạy cảm/null).
4. **Interceptors:** Rất hay dùng để xử lý logic cắt ngang (cross-cutting) cho mọi API như: Kiểm tra Token, đo lường thời gian chạy, ghi Log hệ thống.

### 🟡 6.2. Nhóm cấu hình Trung cấp (Cần biết để tùy biến hệ thống)
1. **Filters (`OncePerRequestFilter`):** Cần phân biệt được Filter (của tầng Servlet) khác với Interceptor (của tầng Spring) ra sao và khi nào dùng cái nào (ví dụ Filter hay dùng cho Security).
2. **Multipart Configuration:** Phục vụ chức năng Upload file (cấu hình dung lượng file tối đa, thư mục lưu tạm).
3. **Formatters & Converters:** Dùng khi muốn tự động ép kiểu một tham số trên URL (ví dụ ép chuỗi `"20-10-2023"` thành kiểu `LocalDate`).
4. **Content Negotiation:** Giúp một API có thể linh hoạt trả về cả XML lẫn JSON tùy theo yêu cầu (Header) của Client.

### 🔴 6.3. Nhóm cấu hình Nâng cao (Kiến trúc & Ứng dụng đặc thù)
1. **Validation Configuration:** Can thiệp sâu vào bộ kiểm tra dữ liệu đầu vào (tự viết ra các Annotation `@Valid` của riêng mình, đọc file ngôn ngữ i18n để trả lỗi tiếng Việt/Anh).
2. **Async Request Processing:** Cấu hình xử lý luồng bất đồng bộ (`WebAsyncTask`, `DeferredResult`) để không làm "treo" Tomcat khi gặp tác vụ tốn thời gian.
3. **WebSockets / STOMP Configuration:** Bắt buộc khi làm các tính năng thời gian thực (Real-time) như Chat, bắn Notification.
4. **Security Web Configuration:** Tích hợp Spring Security (cấu hình `SecurityFilterChain`) - Đây là một mảng khổng lồ riêng biệt nhưng gắn chặt với Web.
5. **Reactive Web Configuration (Spring WebFlux):** Khi kiến trúc dự án chuyển sang mô hình Non-blocking (xử lý hàng chục ngàn request cùng lúc), bạn sẽ phải học lại cấu hình bằng `RouterFunctions` mới tinh.

## 📚 Tài liệu tham khảo chính thống

| Tài liệu | Link |
|-----------|------|
| Spring 7.0.8 — MVC Config | [docs.spring.io](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-config.html) |
| Spring 7.0.8 — Interceptors | [docs.spring.io](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-handlermapping/interceptors.html) |
| Spring 7.0.8 — CORS | [docs.spring.io](https://docs.spring.io/spring-framework/reference/web/webmvc/cors.html) |

---

> **Tiếp theo trong roadmap:** `Integrating different Libraries/Frameworks` (Cách nhúng và cấu hình các thư viện bên ngoài như Swagger, Validation... vào kiến trúc Spring Web).

# 5. Giải quyết xung đột quy ước đặt tên (Naming Strategy)

**Vấn đề:** Java dùng chuẩn camelCase (createdAt, phoneNumber), nhưng nhiều đối tác, database hoặc quy chuẩn REST API lại dùng snake_case (created_at, phone_number).

Nếu không có Jackson: Bạn phải đặt tên biến trong DTO là phone_number (sai chuẩn Java) hoặc viết hàm map thủ công.

Với Jackson:
    Cấu hình toàn cục trong application.properties:

        Properties
            spring.jackson.property-naming-strategy=SNAKE_CASE

    Ngay lập tức toàn bộ API tự động biến createdAt thành created_at mà không cần sửa từng DTO.

    Hoặc cấu hình riêng lẻ bằng @JsonProperty("user_phone").