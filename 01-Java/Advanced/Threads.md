# 🧵 Threads (Luồng xử lý)

> **Thuộc roadmap:** `1. Java → Advanced → Threads`

---

## 1. Thread là gì?

Mặc định, mọi chương trình Java khi chạy đều có 1 luồng chính gọi là **Main Thread**. Chương trình sẽ chạy từ trên xuống dưới, chạy xong lệnh 1 mới tới lệnh 2. 
**Thread (Luồng)** là cơ chế để Java có thể làm **nhiều việc cùng một lúc (Multi-threading)**. 

### Vòng đời của một Thread
1. **New:** Vừa gọi `new Thread()`, luồng mới ra đời nhưng chưa chạy.
2. **Runnable:** Khi gọi `thread.start()`, luồng sẵn sàng chạy và chờ CPU phân bổ thời gian.
3. **Running:** Đang thực sự chạy code bên trong hàm `run()`.
4. **Blocked / Waiting:** Bị tạm dừng (VD: Chờ đọc file từ ổ cứng, chờ tải mạng, hoặc gọi `Thread.sleep()`).
5. **Terminated:** Chạy xong hàm `run()`, luồng kết thúc và chết.

### 2 cách tạo Thread trong Java
- **Cách 1 (Kế thừa class `Thread`):** Ít dùng vì Java không hỗ trợ đa kế thừa.
- **Cách 2 (Hiện thực interface `Runnable`):** Hay dùng hơn, decoupling (tách rời) logic công việc ra khỏi cấu trúc Thread.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn làm chức năng "Đăng ký tài khoản". Khi user bấm nút Đăng ký, hệ thống phải làm 2 việc: (1) Lưu vào CSDL MySQL (mất 0.1s) và (2) Gửi Email xác nhận cho user (mất 3s).*

**Giải quyết vấn đề:**
Nếu bạn code tuần tự bình thường (chỉ dùng 1 Main Thread), User sẽ phải đợi **3.1 giây** màn hình cứ xoay tròn mòng mòng rồi mới thấy thông báo thành công. Trải nghiệm vô cùng tệ!

**Giải pháp: Tách việc gửi Email sang một Thread khác chạy ngầm.**
Main Thread chỉ lo lưu CSDL xong là báo "Thành công" ngay lập tức cho User (mất 0.1s). Còn việc gửi Email sẽ được quăng cho một Thread phụ khác (Worker Thread) từ từ xử lý ở chế độ nền (Background).

**Ví dụ thực tiễn:**

```java
public class RegistrationService {

    public void registerUser(String username, String email) {
        long start = System.currentTimeMillis();

        // 1. Việc chính: Lưu DB (Chạy trên Main Thread)
        saveToDatabase(username);

        // 2. Việc phụ: Gửi Email (Quăng sang Thread khác chạy ngầm)
        Thread emailThread = new Thread(new Runnable() {
            @Override
            public void run() {
                sendEmail(email);
            }
        });
        emailThread.start(); // Kích hoạt luồng mới, chạy song song luôn!

        long end = System.currentTimeMillis();
        System.out.println("✅ Phản hồi cho User chỉ mất: " + (end - start) + "ms");
    }

    private void saveToDatabase(String user) {
        try { Thread.sleep(100); } catch (Exception e){} // Giả lập mất 0.1s
        System.out.println("Đã lưu " + user + " vào CSDL.");
    }

    private void sendEmail(String email) {
        try { Thread.sleep(3000); } catch (Exception e){} // Giả lập mất 3s
        System.out.println("Đã gửi email thành công tới " + email);
    }
}
```

---

## 3. Tóm tắt
Thread là công cụ mạnh mẽ để tăng tốc độ phản hồi của ứng dụng và tận dụng tối đa CPU nhiều nhân. Trong Spring Framework hay Tomcat, cứ mỗi một Request từ user gửi tới, Server sẽ tự động cắt cử 1 Thread ra để phục vụ. Việc hiểu về vòng đời Thread là nền tảng để học tiếp chương "Concurrency" cực kỳ phức tạp phía sau.

---

## 4. Tổng hợp các chủ đề Threads (Bản đồ tra cứu)

### 🌟 4.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc khi đi làm)
1. **Vòng đời Thread:** Nắm vững các trạng thái New, Runnable, Blocked, Waiting, Terminated.
2. **Runnable vs Thread:** Phân biệt và hiểu tại sao luôn được khuyên dùng `Runnable` (hoặc `Callable`) thay vì kế thừa `Thread`.

### 🟡 4.2. Nhóm Trung cấp (Kiểm soát Luồng)
1. **Thread.sleep() vs Object.wait():** Phân biệt sự khác nhau (sleep giữ khóa, wait nhả khóa).
2. **Thread Interrupt:** Cách để ngắt một luồng đang chạy hoặc đang chờ một cách an toàn thay vì dùng `Thread.stop()` (đã bị deprecated).

### 🔴 4.3. Nhóm Nâng cao (Cấu trúc sâu)
1. **ThreadLocal:** Biến cục bộ cho từng luồng. Kỹ thuật này được dùng cực nhiều trong Spring để truyền User đang đăng nhập xuống các tầng dưới.
2. **Virtual Threads (Project Loom - Java 21):** Luồng ảo siêu nhẹ do JVM quản lý thay vì OS quản lý, cho phép khởi tạo hàng triệu luồng mà không tốn RAM.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Oracle - Concurrency in Java | [docs.oracle.com](https://docs.oracle.com/javase/tutorial/essential/concurrency/) |
| Baeldung - Java Thread Life Cycle | [baeldung.com](https://www.baeldung.com/java-thread-lifecycle) |