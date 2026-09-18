# ⚡ Concurrency & Multithreading (Đồng thời)

> **Thuộc roadmap:** `1. Java → Advanced → Concurrency`

---

## 1. Concurrency (Xử lý đồng thời) là gì?

Nếu *Thread* chỉ là khái niệm về 1 luồng xử lý riêng lẻ, thì **Concurrency (Đồng thời)** là nghệ thuật quản lý **HÀNG TRĂM, HÀNG NGÀN Thread** cùng chạy một lúc, cùng tranh giành CPU, và cùng tranh giành dữ liệu.

Concurrency sinh ra các vấn đề kinh điển khiến lập trình viên mất ăn mất ngủ:
1. **Race Condition (Tình trạng tương tranh):** Khi 2 luồng cùng nhào vô thay đổi chung 1 biến, dữ liệu sẽ bị sai lệch.
2. **Deadlock (Bế tắc):** Luồng A khóa dữ liệu 1 chờ dữ liệu 2, Luồng B khóa dữ liệu 2 chờ dữ liệu 1. Cả 2 chờ nhau vĩnh viễn, ứng dụng treo cứng.
3. **Overhead (Quá tải):** Việc tạo mới 1 Thread tốn rất nhiều RAM (khoảng 1MB/thread). Nếu có 10.000 user, tạo 10.000 thread máy chủ sẽ sập ngay.

### Giải pháp cơ bản của Java
- Từ khóa `synchronized`: Khóa cửa phòng lại. Hàm nào gắn từ này, tại 1 thời điểm chỉ cho phép 1 Thread được chui vào chạy, các Thread khác phải xếp hàng chờ ở ngoài.
- `ThreadPool` (ExecutorService): Thay vì tạo Thread mới liên tục, ta tạo sẵn 1 cái "hồ" chứa 10 Thread tái sử dụng. Xong việc thì Thread quay về hồ đợi việc mới, không bị chết đi.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán kinh điển: **Bán vé xem phim / Bán hàng Flash Sale (Chống bán lố - Overselling)***. 

**Vấn đề:** Trong kho chỉ còn đúng 1 cái iPhone. 
Lúc 12:00:00, cả Anh A (Thread 1) và Chị B (Thread 2) cùng bấm nút mua.
1. Luồng A chạy hàm: `if (kho > 0)` -> Thấy kho còn 1 -> Pass.
2. Vừa lúc đó CPU chuyển qua Luồng B chạy hàm: `if (kho > 0)` -> Vẫn thấy kho còn 1 (vì luồng A chưa kịp trừ) -> Pass.
3. Luồng A trừ kho: `kho = 0`, báo anh A mua thành công.
4. Luồng B cũng trừ kho: `kho = -1`, báo chị B mua thành công.
**Hậu quả:** 1 cái điện thoại bán cho 2 người (Race Condition). Bạn phải đền tiền.

**Giải quyết vấn đề bằng Synchronized / ReentrantLock**
Chúng ta phải "khóa" đoạn code kiểm tra và trừ kho lại. Chỉ cho phép 1 Thread xử lý trọn vẹn logic đó xong, rồi mới tới Thread khác.

**Ví dụ thực tiễn:**

```java
public class InventoryService {
    private int stock = 1; // Chỉ còn 1 sản phẩm

    // Từ khóa synchronized đảm bảo tính Đồng bộ hóa (Thread-safe)
    // Dù có 1 triệu người bấm Mua cùng lúc, họ cũng phải xếp hàng đi qua hàm này từng người một.
    public synchronized void buyProduct(String user) {
        if (stock > 0) {
            System.out.println(user + " đã kiểm tra kho, thấy còn hàng.");
            
            // Giả lập hệ thống xử lý mất một chút thời gian (gọi API ngân hàng...)
            try { Thread.sleep(100); } catch (Exception e){}
            
            stock = stock - 1;
            System.out.println("✅ " + user + " đã mua thành công! Kho còn: " + stock);
        } else {
            System.out.println("❌ " + user + " mua thất bại! Hết hàng.");
        }
    }
}
```

*Nếu không có `synchronized`, cả A và B đều mua thành công và kho sẽ về `-1`.*

---

## 3. Bản đồ tra cứu nâng cao (Concurrent Utilities)
Java cung cấp sẵn các bộ đồ nghề tối tân trong gói `java.util.concurrent` để bạn khỏi phải xài `synchronized` thủ công rườm rà:
1. **Concurrent Collections:** `ConcurrentHashMap`, `CopyOnWriteArrayList` (Các danh sách chống xung đột dữ liệu).
2. **Atomic Variables:** `AtomicInteger`, `AtomicLong` (Biến đếm an toàn, dùng để đếm số lượt xem bài viết siêu tốc độ mà không cần khóa).
3. **Locks:** `ReentrantLock` (Khóa xịn hơn, cho phép hẹn giờ chờ mở khóa, hết giờ không chờ nữa để tránh Deadlock).

---

## 4. Tổng hợp các chủ đề Concurrency (Bản đồ tra cứu)

### 🌟 4.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc khi đi làm)
1. **Race Condition & Deadlock:** Hiểu bản chất lỗi tương tranh dữ liệu và tình trạng bế tắc khi chờ khóa của nhau.
2. **Synchronized:** Cách dùng cơ bản nhất để khóa 1 phương thức hoặc 1 block code.
3. **ThreadPool (ExecutorService):** Tuyệt đối không `new Thread()` thủ công trong môi trường Production, luôn dùng ThreadPool.

### 🟡 4.2. Nhóm Trung cấp (Công cụ Concurrency)
1. **Volatile:** Từ khóa giúp các biến không bị cache ở CPU, đảm bảo mọi luồng đều đọc được giá trị mới nhất.
2. **Atomic Classes:** `AtomicInteger`, `AtomicReference`... dùng cơ chế CAS (Compare-And-Swap) thay vì khóa block.
3. **Concurrent Collections:** `ConcurrentHashMap`, `CopyOnWriteArrayList` chống lỗi ConcurrentModificationException.

### 🔴 4.3. Nhóm Nâng cao (Tối ưu chuyên sâu)
1. **Locks (ReentrantLock, ReadWriteLock):** Cơ chế khóa tinh vi hơn `synchronized`, hỗ trợ fair-lock, try-lock.
2. **CountDownLatch & CyclicBarrier:** Các class giúp đồng bộ hóa các luồng.
3. **CompletableFuture:** Viết code bất đồng bộ (Asynchronous) theo phong cách Functional/Chaining cực kỳ mạnh mẽ.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Java Concurrency in Practice | Tác giả: Brian Goetz (Sách kinh điển về Concurrency) |
| Baeldung - Java Concurrency | [baeldung.com](https://www.baeldung.com/java-concurrency) |