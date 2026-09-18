# 🕸️ Servlets & JSPs (Công nghệ Web nền tảng)

> **Thuộc roadmap:** `1. Java → Advanced → Servlets and JSP’s`

---

## 1. Servlet & JSP là gì? (Lịch sử Java Web)

Trước khi có những Framework phép thuật xịn xò như Spring Boot hay Next.js, thế giới Java dùng **Servlet** để làm web. 

1. **Servlet:** Là một class Java đặc biệt, có khả năng lắng nghe và hứng các HTTP Request (GET, POST) từ trình duyệt gửi lên, xử lý logic, đọc database, rồi phản hồi về. Để chạy được Servlet, bạn phải cài đặt một phần mềm "Máy chủ Web" (Web Server / Servlet Container) ví dụ như **Apache Tomcat**.
2. **JSP (JavaServer Pages):** Servlet thì rất giỏi xử lý logic, nhưng khi nó phải in ra một cục HTML để trả về cho trình duyệt thì code trông cực kỳ kinh tởm (`out.println("<h1>Hello</h1>");`). Do đó, JSP ra đời. JSP cho phép bạn viết file HTML bình thường, nhưng nhúng các đoạn code Java `<% ... %>` vào giữa giao diện. (Tương tự như PHP).

**Tại sao phải học Servlet ngày nay?**
Spring Boot, Spring MVC về bản chất bên dưới lõi sâu xa nhất **chính là Servlet**. Spring sử dụng một con Servlet duy nhất tên là `DispatcherServlet` để hứng toàn bộ Request của hệ thống, rồi mới phân phát vào các hàm `@RestController` của bạn. Hiểu Servlet là hiểu bản chất của mạng HTTP trong Java.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Cần tạo một trang Web đơn giản có chức năng Đăng Nhập. Điền username/password, nếu đúng chuyển sang trang Chào mừng.*

**Giải quyết vấn đề:** 
Đây là mô hình nguyên thủy để xử lý một form HTML gửi lên Backend bằng Servlet.

**Ví dụ thực tiễn:**

```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;
import java.io.IOException;

// Cấu hình đường dẫn URL cho Servlet này
@WebServlet("/login") 
public class LoginServlet extends HttpServlet {

    // Hàm doGet sẽ hứng các request GET (Khi user gõ URL lên trình duyệt)
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        // Trả về một giao diện HTML Form
        response.setContentType("text/html");
        response.getWriter().println("<form method='POST' action='/login'>");
        response.getWriter().println("User: <input name='username'><br>");
        response.getWriter().println("Pass: <input name='password' type='password'><br>");
        response.getWriter().println("<button>Đăng Nhập</button>");
        response.getWriter().println("</form>");
    }

    // Hàm doPost sẽ hứng các request POST (Khi user bấm nút submit Form)
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        // 1. Rút trích dữ liệu gửi lên (giống @RequestParam của Spring)
        String user = request.getParameter("username");
        String pass = request.getParameter("password");

        // 2. Xử lý Logic
        if ("admin".equals(user) && "123".equals(pass)) {
            // Đăng nhập đúng, điều hướng (Redirect) về trang chủ
            response.getWriter().println("<h1>Dang nhap THANH CONG! Chào sếp.</h1>");
        } else {
            // Đăng nhập sai, báo lỗi
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // Mã 401
            response.getWriter().println("<h1>SAI MAT KHAU!</h1>");
        }
    }
}
```

---

## 3. Kiến trúc vòng đời (Servlet Life Cycle)
Khi Web Server (Tomcat) khởi động, vòng đời của một Servlet diễn ra như sau:
1. `init()`: Chạy 1 lần duy nhất khi Servlet được nạp vào bộ nhớ. (Dùng để nạp cấu hình, kết nối DB).
2. `service()`: Chạy vô số lần, mỗi khi có 1 user truy cập. Nó tự xem user gửi GET hay POST để gọi hàm `doGet` hoặc `doPost` tương ứng. (Mỗi user là một Thread riêng biệt, nên Servlet rất dễ dính Race Condition nếu dùng biến toàn cục).
3. `destroy()`: Chạy 1 lần duy nhất khi tắt Tomcat. (Dọn dẹp tài nguyên).

**Chuyển tiếp đến hiện tại:** Ngày nay, 99% các dự án không ai tự viết Servlet thuần nữa, mà chuyển sang dùng Spring Controller/RestController để code ngắn gọn, sạch sẽ hơn và test dễ dàng hơn. Nhưng kiến thức HTTP Request/Response (Header, Status Code, Parameter) từ Servlet thì vĩnh viễn không thay đổi.

---

## 4. Tổng hợp các chủ đề Servlets & JSP (Bản đồ tra cứu)

### 🌟 4.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc)
1. **HttpServletRequest & HttpServletResponse:** 2 object chứa mọi thông tin người dùng gửi lên và Server trả về (Header, Body, Status, Cookie).
2. **Servlet Life Cycle:** Vòng đời `init()`, `service()`, `destroy()`.

### 🟡 4.2. Nhóm Trung cấp (Bảo mật & Điều hướng)
1. **Session & Cookie:** Phân biệt cơ chế lưu thông tin ở trình duyệt (Cookie) và lưu ở Server (Session - `HttpSession`).
2. **Filter:** Chặn các request trước khi vào Servlet để kiểm tra quyền (Authentication) hoặc log dữ liệu.
3. **Forward vs Redirect:** Phân biệt sự khác nhau giữa việc server ngầm chuyển tiếp và ép trình duyệt tự chuyển hướng.

### 🔴 4.3. Nhóm Nâng cao (Mở rộng tính năng)
1. **Listeners (ServletContextListener):** Lắng nghe các sự kiện khi ứng dụng bật lên, tắt đi hoặc Session bị hủy.
2. **Asynchronous Servlets:** Xử lý bất đồng bộ để tránh bị nghẽn (Blocking) các luồng của Tomcat.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Oracle - Java EE Servlet Tutorial | [docs.oracle.com](https://docs.oracle.com/javaee/7/tutorial/servlets.htm) |
| Baeldung - Intro to Java Servlets | [baeldung.com](https://www.baeldung.com/intro-to-servlets) |