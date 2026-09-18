# 🌐 Spring Web Annotations — (Spring Boot 4.1.0 / Spring 7.0.8)

> **Thuộc roadmap:** `2. Spring Framework → Web → Annotations`

Trong Spring Web, **Annotation** đóng vai trò như các "biển báo giao thông". Khi có một Request từ phía người dùng (từ trình duyệt, ứng dụng mobile) gửi tới server, các biển báo này sẽ hướng dẫn Spring Container biết: *Request này sẽ đi vào Class nào, Hàm nào, và lấy dữ liệu ra sao?*

Dưới đây là các Annotation "sống còn" mà bạn sẽ sử dụng hằng ngày khi làm backend.

---

## 1. Xác định Controller (Người tiếp tân)

Controller chính là điểm đến đầu tiên khi có một HTTP Request bay vào server của bạn.

### `@Controller`
- Dùng cho kiến trúc MVC truyền thống (thường trả về giao diện HTML như file Thymeleaf, JSP).
- Nếu muốn trả về JSON thay vì HTML, bạn phải gắn thêm `@ResponseBody` vào từng hàm.

### `@RestController` (Khuyên dùng 99% hiện nay)
- Được dùng để xây dựng các API (RESTful API).
- Bản chất `@RestController` = `@Controller` + `@ResponseBody`.
- Nghĩa là mọi dữ liệu (Object, List...) bạn trả về từ class này sẽ tự động được Spring biến thành chuỗi **JSON**.

---

## 2. Ánh xạ Request (Định tuyến - Routing)

Để chỉ định đường dẫn (URL) cụ thể cho một hàm, chúngẽ dùng các Annotation thuộc nhóm Mapping.

### `@RequestMapping`
- Là annotation gốc, có thể dùng ở cả cấp độ Class và Method.
- Có thể nhận mọi loại HTTP Method (GET, POST, PUT, DELETE) nếu không chỉ định rõ.

### Các biến thể chuyên biệt (Method-specific)
Kể từ Spring 4.3 trở đi, bạn nên dùng các biến thể chuyên biệt cho từng loại hành động để code rõ ràng hơn:
- `@GetMapping("/users")`: Lấy dữ liệu (Read).
- `@PostMapping("/users")`: Tạo mới dữ liệu (Create).
- `@PutMapping("/users/{id}")`: Cập nhật toàn bộ dữ liệu (Update).
- `@DeleteMapping("/users/{id}")`: Xóa dữ liệu (Delete).

---

## 3. Bắt dữ liệu từ Client (Nhận thông tin đầu vào)

Làm sao để lấy những gì user gửi lên (như ID, Form data, JSON payload)?

### `@PathVariable` (Biến trên đường dẫn)
- Dùng để lấy giá trị động nằm ngay trong cấu trúc của URL.
- **Ví dụ URL:** `GET /users/123`

### `@RequestParam` (Tham số truy vấn)
- Dùng để lấy các query string nằm sau dấu hỏi chấm `?` trên URL, hoặc dữ liệu từ HTML Form.
- **Ví dụ URL:** `GET /users?page=2&size=10`

### `@RequestBody` (Phần thân Request)
- Cực kỳ quan trọng! Dùng để lấy dữ liệu JSON (hoặc XML) mà client gửi lên ở phần Body (thường dùng trong POST, PUT).
- Spring sẽ tự động "ép kiểu" (deserialize) chuỗi JSON đó thành một Object Java tương ứng cho bạn.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Đây là phần được thiết kế theo Quy tắc (Rule) học tập của bạn!*

Thay vì lý thuyết suông, hãy xem một tình huống thực tế: **Xây dựng API Quản lý Người dùng (User Management)**.

**Tình huống:** Frontend (React/Vue/Mobile App) cần giao tiếp với Backend để thực hiện các thao tác: Lấy danh sách user, lấy chi tiết 1 user, tạo user mới.

```java
import org.springframework.web.bind.annotation.*;
import java.util.List;

// 1. Khai báo đây là REST API, luôn trả về JSON thay vì giao diện HTML
@RestController 
// 2. Định nghĩa "Tiền tố" chung cho toàn bộ Class này. Mọi hàm bên dưới đều bắt đầu bằng /api/v1/users
@RequestMapping("/api/v1/users") 
public class UserController {

    /* =========================================
     * CASE 1: Ứng dụng @RequestParam
     * Thực tế: Phục vụ chức năng TÌM KIẾM và PHÂN TRANG (Pagination & Filtering).
     * Frontend gọi: GET /api/v1/users?status=active&page=1
     * ========================================= */
    @GetMapping
    public List<User> searchUsers(
            // Nếu Frontend không gửi lên, mặc định status là 'all'
            @RequestParam(defaultValue = "all") String status, 
            @RequestParam(defaultValue = "1") int page) {
        
        System.out.println("Đang tìm users với status: " + status + ", trang: " + page);
        return List.of(new User("Alice"), new User("Bob")); // Trả về dạng JSON
    }


    /* =========================================
     * CASE 2: Ứng dụng @PathVariable
     * Thực tế: Xem chi tiết Profile (Hồ sơ) của MỘT người dùng cụ thể.
     * Frontend gọi: GET /api/v1/users/99
     * ========================================= */
    @GetMapping("/{id}")
    public User getUserDetail(@PathVariable("id") Long userId) {
        // userId sẽ tự động lấy số 99 từ URL trên
        System.out.println("Lấy thông tin của User ID: " + userId);
        return new User("Alice"); 
    }


    /* =========================================
     * CASE 3: Ứng dụng @RequestBody
     * Thực tế: Xử lý chức năng ĐĂNG KÝ TÀI KHOẢN (Register).
     * Frontend gửi lên 1 cục JSON bự chứa: { "username": "admin", "email": "a@a.com", "password": "123" }
     * ========================================= */
    @PostMapping
    public String createUser(@RequestBody UserDTO userDto) {
        // Cục JSON từ Frontend đã được Spring "nhào nặn" thành Object UserDTO.
        // Bạn không cần phải tự parse chuỗi JSON lằng nhằng nữa!
        System.out.println("Đang lưu user mới vào database: " + userDto.getEmail());
        return "Tạo thành công!"; 
    }
}
```

### 🎯 Tổng kết giá trị ứng dụng:
Nhờ bộ Annotations này, việc viết API của bạn trở nên **"Declarative" (Mang tính khai báo)**:
1. Bạn không phải viết code loằng ngoằng để mò mẫm bóc tách URL, tìm tham số (như thời dùng Servlet cổ điển). 
2. Bạn không cần viết code chuyển đổi JSON thành Java Object (GSON, Jackson thủ công).
3. Code rất gọn gàng, tự làm tài liệu (Self-documenting), nhìn vào Annotation là biết ngay hàm này phục vụ cho thao tác gì (GET/POST) trên đường dẫn nào.

---

## 4. Tổng hợp các chủ đề Web Annotations (Bản đồ tra cứu)

Spring Web có hàng tá Annotation khác nhau để hỗ trợ mọi nhu cầu bóc tách dữ liệu từ HTTP Request. Dưới đây là danh sách phân loại để bạn dễ dàng định hướng:

### 🌟 4.1. Những Annotation ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc khi đi làm)
1. **`@RequestHeader`:** Dùng để lấy các giá trị từ HTTP Header. Thực tế 99% dùng để trích xuất chuỗi `Authorization: Bearer <token>` để kiểm tra user đăng nhập.
2. **`@ResponseStatus`:** Dùng để ép kiểu HTTP Status Code trả về. Ví dụ: theo chuẩn RESTful, khi tạo user mới thành công, thay vì trả mã `200 OK` mặc định, bạn phải gắn annotation này để ép nó trả mã `201 Created`.

### 🟡 4.2. Nhóm Annotation Trung cấp (Xử lý Form & Cấu hình cục bộ)
1. **`@ModelAttribute`:** Thường dùng trong mô hình MVC cũ (render HTML) để bóc tách toàn bộ dữ liệu từ một Form HTML (có chứa cả text và file upload) gán thẳng vào một Java Object.
2. **`@CookieValue`:** Lấy nhanh một giá trị được lưu trong Cookie của trình duyệt (ví dụ: lấy `session_id`).
3. **`@CrossOrigin`:** Mở khóa bảo mật CORS cục bộ cho riêng một Controller hoặc một hàm API cụ thể (thay vì phải mở toàn cục bằng file Config).

### 🔴 4.3. Nhóm Kỹ thuật Nâng cao (Tự chế Annotation)
1. **Custom `HandlerMethodArgumentResolver` (Kỹ thuật cực xịn):** Kỹ thuật này cho phép bạn tự định nghĩa ra một Annotation của riêng mình (ví dụ: `@CurrentUser`). Khi bạn gắn `@CurrentUser User u` vào tham số hàm Controller, Spring sẽ tự động giải mã token, chui xuống Database, tìm đúng ông User đang đăng nhập và nhét vào biến `u` sẵn cho bạn. Code Controller lúc này sẽ sạch sẽ tuyệt đối!
2. **`@MatrixVariable`:** Xử lý các chuỗi tham số URL siêu phức tạp (ví dụ: `/cars;color=red;year=2023`).

## 📚 Tài liệu tham khảo chính thống

| Tài liệu | Link |
|-----------|------|
| Spring 7.0.8 — Annotated Controllers | [docs.spring.io](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html) |
| Spring 7.0.8 — Handler Methods Arguments | [docs.spring.io](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html) |
