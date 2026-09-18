# 🔁 HTTP Method: TRACE — REST API

> **Nguồn chính thống:** [RFC 9110 — HTTP Semantics: TRACE](https://www.rfc-editor.org/rfc/rfc9110.html#name-trace) | [MDN Web Docs — TRACE](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/TRACE)
>
> **Thuộc roadmap:** `4. REST API → HTTP Methods → TRACE`

---

## 1. Định nghĩa chính thức từ RFC 9110

> *"The **TRACE** method requests a **remote, application-level loop-back** of the request message. The final recipient of the request SHOULD reflect the message received, excluding some fields described below, back to the client as the message body of a **200 (OK)** response with a **Content-Type of `message/http`**."*
>
> — **RFC 9110, Section 9.3.8**

### Dịch & giải thích đơn giản:

**TRACE** là phương thức HTTP dùng để **debug mạng**. Khi client gửi TRACE, server sẽ **gửi lại (echo) nguyên văn request** mà nó nhận được. Điều này giúp kiểm tra xem request có bị **thay đổi** khi đi qua các proxy, load balancer hay middleware trung gian hay không.

Hãy tưởng tượng TRACE giống như **gửi thư bảo đảm kèm yêu cầu "gửi lại bản sao cho tôi"**. Bạn muốn xem tờ thư đến tay người nhận có bị ai đó mở ra đọc/sửa gì không.

Đặc tính:
- **Safe:** TRACE **KHÔNG** thay đổi dữ liệu trên server.
- **Idempotent:** Gọi lại bao nhiêu lần cũng cho kết quả giống nhau.
- **KHÔNG có Request Body:** RFC quy định: *"A client MUST NOT send content in a TRACE request."*
- **Response Body = Echo của Request:** Server trả lại chính xác request nhận được.

> [!CAUTION]
> **TRACE nên bị TẮT trong production!** Đây là một attack vector bảo mật quan trọng. Xem chi tiết ở phần "Vấn đề bảo mật" bên dưới.

---

## 2. Cách TRACE hoạt động

### Request:
```http
TRACE /api/products HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
X-Custom-Header: some-value
```

### Response (Server echo lại request):
```http
HTTP/1.1 200 OK
Content-Type: message/http
Content-Length: 172

TRACE /api/products HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
X-Custom-Header: some-value
X-Forwarded-For: 192.168.1.100          ← Proxy thêm header này!
X-Forwarded-Proto: https                 ← Proxy thêm header này!
```

### Khi nào hữu ích?

```mermaid
sequenceDiagram
    participant Client
    participant Proxy as Reverse Proxy<br/>(Nginx)
    participant LB as Load Balancer
    participant Server as Spring Boot

    Client->>Proxy: TRACE /api/products<br/>Headers: A, B, C
    Proxy->>LB: TRACE /api/products<br/>Headers: A, B, C, X-Forwarded-For
    LB->>Server: TRACE /api/products<br/>Headers: A, B, C, X-Forwarded-For, X-Request-ID
    Server-->>Client: 200 OK<br/>Body = Echo request nhận được<br/>→ Bạn thấy Proxy và LB đã THÊM headers gì!
```

→ Giúp bạn biết request **bị thay đổi như thế nào** khi đi qua các intermediary.

---

## 3. Vấn đề bảo mật — Cross-Site Tracing (XST) Attack ⚠️

### 3.1 Tại sao TRACE nguy hiểm?

Nếu TRACE được bật, hacker có thể khai thác nó kết hợp với **Cross-Site Scripting (XSS)** để **đánh cắp cookie HttpOnly** hoặc **Authorization header**:

```mermaid
flowchart TD
    A["1. Hacker inject XSS vào trang web"] --> B["2. JavaScript độc hại chạy\ntrong trình duyệt nạn nhân"]
    B --> C["3. Script gửi TRACE request\nđến server (kèm cookie/token tự động)"]
    C --> D["4. Server echo lại request\n→ Response body chứa cookie & token!"]
    D --> E["5. JavaScript đọc response body\n→ ĐÁnh CẮP được cookie HttpOnly\nvà Authorization header!"]
```

**Vấn đề cốt lõi:** Cookie `HttpOnly` được thiết kế để JavaScript **không thể đọc trực tiếp**. Nhưng nếu TRACE được bật, JavaScript có thể gửi TRACE → server echo lại request (kèm cookie trong header) → JavaScript đọc **response body** → có cookie!

### 3.2 Tại sao phải tắt TRACE?

| Lý do | Chi tiết |
|-------|---------|
| **Cross-Site Tracing (XST)** | Cho phép đánh cắp HttpOnly cookies thông qua XSS |
| **Information Disclosure** | Lộ internal headers (X-Forwarded-For, X-Real-IP) |
| **Security Compliance** | PCI DSS, OWASP đều khuyến nghị tắt TRACE |
| **Không cần thiết** | Debugging network có nhiều tool tốt hơn (Wireshark, Chrome DevTools) |

---

## 4. Tắt TRACE trong Spring Boot 4.1.0

### 4.1 Spring Boot mặc định

> [!NOTE]
> **Tin tốt:** Spring Boot (thông qua Spring MVC `DispatcherServlet`) **mặc định KHÔNG xử lý TRACE request**. Nếu client gửi TRACE, Spring MVC sẽ trả về **405 Method Not Allowed**. Bạn **không cần làm gì thêm** ở tầng ứng dụng.

Tuy nhiên, bạn vẫn nên tắt TRACE ở tầng **web server/reverse proxy** để phòng ngừa:

### 4.2 Tắt ở Embedded Tomcat (Belt & Suspenders)

```java
@Configuration
public class TomcatConfig {

    @Bean
    public WebServerFactoryCustomizer<TomcatServletWebServerFactory> tomcatCustomizer() {
        return factory -> factory.addConnectorCustomizers(connector -> {
            connector.setAllowTrace(false);   // Tắt TRACE ở Tomcat level
        });
    }
}
```

### 4.3 Tắt ở Nginx (Reverse Proxy)

```nginx
server {
    listen 80;
    server_name api.example.com;

    # Chặn TRACE và TRACK methods
    if ($request_method ~ ^(TRACE|TRACK)$) {
        return 405;
    }

    location / {
        proxy_pass http://localhost:8080;
    }
}
```

### 4.4 Tắt qua Spring Security

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http
        // Chỉ cho phép các methods cần thiết
        .authorizeHttpRequests(auth -> auth
            .anyRequest().authenticated()
        )
        // Spring Security mặc định đã chặn TRACE
        .build();
}
```

---

## 5. TRACE vs các công cụ debug hiện đại

TRACE ra đời từ HTTP/1.1 (1997) khi công cụ debug mạng còn hạn chế. Ngày nay, có nhiều cách debug tốt hơn:

| Công cụ | Mục đích | Ưu điểm so với TRACE |
|---------|---------|----------------------|
| **Chrome DevTools → Network tab** | Xem request/response chi tiết | Trực quan, an toàn, không cần bật TRACE |
| **Postman / Insomnia** | Test API, xem headers | Hiển thị đầy đủ request/response |
| **Wireshark** | Bắt packet mạng cấp thấp | Xem toàn bộ traffic, không chỉ HTTP |
| **curl -v** | Debug từ terminal | `curl -v http://localhost:8080/api/products` → xem headers chi tiết |
| **Spring Boot Actuator** | Giám sát ứng dụng | `/actuator/httptrace` — xem lịch sử requests |
| **Access Log** | Ghi log mọi request đến server | Cấu hình `server.tomcat.accesslog.enabled=true` |

```bash
# Debug bằng curl — thay thế TRACE an toàn hơn
curl -v -X GET http://localhost:8080/api/products \
  -H "Authorization: Bearer eyJ..." \
  2>&1 | head -30

# Output hiển thị:
# > GET /api/products HTTP/1.1
# > Host: localhost:8080
# > Authorization: Bearer eyJ...
# < HTTP/1.1 200 OK
# < Content-Type: application/json
```

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

### Thực tế: TRACE hầu như KHÔNG ĐƯỢC SỬ DỤNG

Trong 99.9% các dự án thực tế, bạn sẽ **không bao giờ cần dùng TRACE**. Lý do:
1. **Bảo mật:** Mọi security audit đều yêu cầu tắt TRACE
2. **Công cụ tốt hơn:** Chrome DevTools, Postman, Wireshark đều mạnh hơn TRACE
3. **Spring Boot đã tắt sẵn:** Mặc định trả 405 cho TRACE

### Checklist bảo mật HTTP Methods cho Production:

| HTTP Method | Nên bật? | Ghi chú |
|------------|---------|---------|
| GET | ✅ Bật | Đọc dữ liệu |
| POST | ✅ Bật | Tạo mới |
| PUT | ✅ Bật | Cập nhật toàn bộ |
| PATCH | ✅ Bật | Cập nhật một phần |
| DELETE | ✅ Bật | Xoá |
| OPTIONS | ✅ Bật | CORS preflight (trình duyệt cần) |
| HEAD | ✅ Bật | Giống GET nhưng không body (health check) |
| **TRACE** | **❌ TẮT** | **Rủi ro bảo mật XST** |
| **TRACK** | **❌ TẮT** | **Biến thể của TRACE (Microsoft IIS)** |

---

## 6. Tóm tắt TRACE

```mermaid
mindmap
  root((TRACE))
    Đặc tính
      Safe - Không thay đổi dữ liệu
      Idempotent
      KHÔNG có Request Body
      Response = Echo request
    Mục đích gốc
      Debug mạng
      Kiểm tra proxy thay đổi gì
      Năm 1997 - công cụ debug ít
    Bảo mật ⚠️
      Cross-Site Tracing - XST
      Đánh cắp HttpOnly cookies
      Information Disclosure
      PHẢI TẮT trong Production ⭐
    Thay thế bằng
      Chrome DevTools
      curl -v
      Postman
      Wireshark
      Spring Actuator
```

---

## 7. Tổng kết toàn bộ HTTP Methods (Bản đồ tra cứu)

| Method | Mục đích | Safe? | Idempotent? | Body? | Status thường gặp |
|--------|---------|-------|------------|-------|-------------------|
| **GET** | Đọc dữ liệu | ✅ | ✅ | ❌ | 200, 404 |
| **POST** | Tạo mới | ❌ | ❌ | ✅ | 201, 400, 409 |
| **PUT** | Cập nhật toàn bộ | ❌ | ✅ | ✅ | 200, 404 |
| **PATCH** | Cập nhật một phần | ❌ | ⚠️ | ✅ | 200, 404 |
| **DELETE** | Xoá | ❌ | ✅ | ❌ | 204, 404, 409 |
| **OPTIONS** | CORS preflight / Khám phá | ✅ | ✅ | ❌ | 204, 200 |
| **HEAD** | Giống GET, không body | ✅ | ✅ | ❌ | 200 |
| **TRACE** | Debug (⚠️ TẮT) | ✅ | ✅ | ❌ | 200 (405 nếu tắt) |

## 📚 Tài liệu tham khảo chính thống

| Tài liệu | Link |
|-----------|------|
| RFC 9110 — TRACE | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc9110.html#name-trace) |
| MDN — TRACE | [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/TRACE) |
| OWASP — Cross-Site Tracing | [owasp.org](https://owasp.org/www-community/attacks/Cross_Site_Tracing) |
| OWASP — Test HTTP Methods | [owasp.org](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/06-Test_HTTP_Methods) |

---

> **Tiếp theo trong roadmap:** `HTTP Status Codes` (Phân loại chi tiết 1xx → 5xx — khi nào dùng status code nào, best practices cho REST API).
