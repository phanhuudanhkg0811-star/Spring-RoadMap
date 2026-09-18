# 🗑️ Garbage Collection (Thu gom rác)

> **Thuộc roadmap:** `1. Java → Advanced → Garbage Collection`

---

## 1. Garbage Collection (GC) là gì?

Trong các ngôn ngữ cũ như C/C++, khi bạn tạo ra một Object (cấp phát bộ nhớ), bạn phải tự mình xóa nó đi (giải phóng bộ nhớ). Nếu quên xóa, máy tính sẽ bị tràn RAM (Memory Leak) và sập. 

Java giải quyết triệt để nỗi đau này bằng **Garbage Collector (GC)**. Đây là một chương trình chạy ngầm bên trong JVM, nó liên tục rình rập và tự động dọn dẹp các Object "không còn ai sử dụng" nằm trong bộ nhớ **Heap**, trả lại RAM cho hệ điều hành.

### Cơ chế hoạt động cốt lõi: GC Roots & Reachability
GC không xóa bừa bãi. Nó xuất phát từ các điểm gốc gọi là **GC Roots** (ví dụ: các biến cục bộ đang chạy trong Stack, các biến static). Nó sẽ lần theo các đường link tham chiếu (reference). 
- Object nào "còn đường dẫn" nối với GC Roots -> **Còn sống (Alive)**.
- Object nào "bị đứt liên kết", lơ lửng không ai trỏ tới -> **Là rác (Garbage)**, sẽ bị tiêu hủy.

### Cấu trúc vùng nhớ Heap (Generational GC)
Để dọn rác nhanh hơn, Heap được chia làm 2 thế hệ:
1. **Young Generation (Thế hệ trẻ):** Chứa các object vừa mới sinh ra. Đa số object trong Java có tuổi thọ rất ngắn (VD: Dữ liệu JSON vừa parse xong trả về cho user là bỏ). GC sẽ quét vùng này liên tục (gọi là *Minor GC*). Quét rất nhanh.
2. **Old Generation (Thế hệ già):** Những object nào sống sót qua nhiều đợt quét ở Young Generation sẽ được chuyển sang đây (VD: Các `@Service`, `@Repository` sống suốt đời). GC rất ít khi quét vùng này vì nó lớn, và khi quét (gọi là *Major GC* hoặc *Full GC*), nó có thể làm đứng toàn bộ ứng dụng của bạn trong chốc lát ("Stop-The-World").

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn code một app xử lý hàng triệu file Excel. Khi chạy thực tế, thi thoảng ứng dụng lại bị "đứng hình" (treo) khoảng 2-3 giây, user tưởng app bị lag.*

**Giải quyết vấn đề:**
Bằng kiến thức GC, bạn chuẩn đoán được ứng dụng đang bị hiệu ứng **"Stop-The-World" (STW)** do Full GC đang chạy. Khi bộ nhớ Old Generation bị đầy, JVM phải dừng toàn bộ mọi luồng xử lý của ứng dụng lại để tập trung đi nhặt rác. Rác càng nhiều, STW càng lâu.

**Giải pháp (Tuning GC):**
Bạn không sửa code logic, bạn chỉ cần thay đổi **Thuật toán dọn rác (GC Algorithm)** khi khởi động Server:
- `Serial GC`: Dành cho máy yếu 1 CPU (Dọn bằng 1 tay).
- `Parallel GC`: (Mặc định ở Java 8) Dọn rác bằng nhiều luồng, nhưng vẫn bị STW.
- **`G1 GC` (Garbage-First):** (Mặc định từ Java 9+) Chia bộ nhớ thành nhiều ô nhỏ, dọn cuốn chiếu từng ô, cam kết giữ thời gian treo máy dưới 200ms.
- **`ZGC` / `Shenandoah`:** (Các thuật toán hiện đại) Vừa dọn rác vừa chạy app đồng thời, thời gian STW chỉ còn dưới 1ms!

**Ví dụ thực tiễn: Cẩn thận với Memory Leak (Rò rỉ bộ nhớ) ở Java**
Dù có GC, Java vẫn bị rò rỉ bộ nhớ nếu bạn "vô tình" giữ reference (tham chiếu) của object mãi mãi mà không nhả ra!

```java
public class MemoryLeakExample {
    // 1. Một list static (sống mãi cùng chương trình)
    private static final List<byte[]> cache = new ArrayList<>();

    public void processData() {
        // 2. Mỗi lần gọi hàm, ta tạo ra 1MB dữ liệu
        byte[] megabyte = new byte[1024 * 1024]; 
        
        // 3. Và ta nhét nó vào danh sách static
        cache.add(megabyte); 
        
        // 4. LỖI CHẾT NGƯỜI: Do 'cache' là static (GC Root), nó không bao giờ chết.
        // Dẫn đến các cục byte[] bên trong nó cũng không bao giờ bị dọn.
        // Hậu quả: Sau vài ngàn lần gọi hàm, Heap đầy -> văng lỗi OutOfMemoryError.
    }
}
```
**Cách phòng tránh:** Dùng xong cấu trúc dữ liệu toàn cục, phải nhớ `remove()` nó đi, hoặc `cache.clear()` để bẻ gãy liên kết, giúp GC nhận diện nó là rác.

---
## 3. Tóm tắt
Hiểu GC không phải để viết ra một cái GC mới, mà để **Viết code thân thiện với GC**. Tạo object vô tội vạ trong vòng lặp sẽ bắt GC chạy liên tục, tiêu tốn CPU. Biết cách đọc Log GC và chọn đúng GC Algorithm sẽ cứu sống những ứng dụng High-Traffic (truy cập cao).

---

## 4. Tổng hợp các chủ đề Garbage Collection (Bản đồ tra cứu)

### 🌟 4.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc khi đi làm)
1. **GC Roots & Reachability:** Nguyên lý xác định rác cơ bản nhất, hiểu tại sao biến static lại dễ gây Memory Leak.
2. **Stop-The-World (STW):** Khái niệm hệ thống bị "treo" khi GC dọn rác và cách nhận biết.

### 🟡 4.2. Nhóm Trung cấp (Tùy chỉnh & Thuật toán)
1. **Generational Hypothesis:** Cấu trúc Young, Old/Tenured Generation và Eden, Survivor spaces. Phân biệt Minor GC và Major/Full GC.
2. **G1 GC (Garbage-First):** Thuật toán mặc định từ Java 9, cơ chế chia nhỏ Heap thành các Region để tránh STW kéo dài.

### 🔴 4.3. Nhóm Nâng cao (Tuning & Phân tích)
1. **GC Tuning Logs:** Cách kích hoạt cờ in log GC (`-Xlog:gc*`) và đọc log để chẩn đoán nguyên nhân treo hệ thống.
2. **ZGC & Shenandoah:** Thuật toán GC hiện đại (Ultra-low latency), thời gian STW dưới 1ms cho dù Heap có lên tới Terabytes.
3. **Phân tích Heap Dump:** Dùng tool (như Eclipse MAT, VisualVM) để mở file `.hprof` tìm ra object nào đang gây Memory Leak.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Oracle - Garbage Collection Tuning Guide | [docs.oracle.com](https://docs.oracle.com/en/java/javase/17/gctuning/introduction-garbage-collection-tuning.html) |
| Baeldung - JVM Garbage Collectors | [baeldung.com](https://www.baeldung.com/jvm-garbage-collectors) |