# 🧩 Design Patterns (Mẫu thiết kế)

> **Thuộc roadmap:** `1. Java → Advanced → Design Patterns`

---

## 1. Design Patterns là gì?

Code chạy được là một chuyện, nhưng code để **dễ bảo trì, dễ mở rộng, và cho team khác dễ đọc** lại là chuyện khác.

Trong quá trình phát triển phần mềm hàng chục năm qua, các kỹ sư vĩ đại đã gặp đi gặp lại những bài toán thiết kế giống hệt nhau. Thay vì mỗi người tự chế ra một cách viết code (thường là rối rắm), họ đã đúc kết lại thành **23 Design Patterns (Mẫu thiết kế chuẩn)** của nhóm Gang of Four (GoF). 

Design Pattern giống như "Bản vẽ thiết kế nhà mẫu". Bạn không cần tự nghĩ cách bố trí cột bê tông, chỉ cần lấy mẫu chuẩn ra dùng là đảm bảo nhà không sập.

### 3 Nhóm chính:
1. **Creational (Khởi tạo):** Mẫu chuyên lo việc tạo ra Object (thay thế cho từ khóa `new`). Nổi bật: *Singleton, Factory, Builder.*
2. **Structural (Cấu trúc):** Mẫu chuyên lo việc lắp ghép các class lại với nhau. Nổi bật: *Adapter, Proxy, Facade.*
3. **Behavioral (Hành vi):** Mẫu quản lý sự giao tiếp và phân chia nhiệm vụ giữa các Object. Nổi bật: *Observer, Strategy.*

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn xây dựng tính năng Xuất báo cáo (Export Report). Khách hàng yêu cầu xuất ra 3 loại: PDF, Excel, và Word.*

**Cách code "Gà mờ" (Dùng If-Else):**
```java
public void export(String type) {
    if(type.equals("PDF")) { /* 100 dòng code PDF */ }
    else if(type.equals("EXCEL")) { /* 100 dòng code Excel */ }
    else if(type.equals("WORD")) { /* 100 dòng code Word */ }
}
```
**Vấn đề:** Class này dài 300 dòng. Tháng sau sếp đòi thêm tính năng xuất ra HTML, CSV, JSON... cái hàm này sẽ phình to thành 1000 dòng. Vi phạm quy tắc Open/Closed (O trong SOLID).

**Giải quyết vấn đề bằng: Strategy Pattern (Mẫu Chiến Lược)**
Ứng dụng Strategy, ta tách mỗi loại báo cáo thành một class (chiến lược) riêng biệt. Lúc nào cần xuất loại nào, ta nhét chiến lược đó vào.

**Ví dụ thực tiễn:**

```java
// 1. Tạo 1 cái khung chuẩn (Interface)
public interface ExportStrategy {
    void exportReport();
}

// 2. Viết từng chiến lược rành mạch vào từng File riêng
public class PdfExport implements ExportStrategy {
    public void exportReport() { System.out.println("Đang vẽ PDF..."); }
}
public class ExcelExport implements ExportStrategy {
    public void exportReport() { System.out.println("Đang kẻ ô Excel..."); }
}

// 3. Class chạy chính, nó không cần quan tâm (không cần if-else)
// Bạn đưa cho nó chiến lược nào, nó chạy chiến lược đó!
public class ReportService {
    private ExportStrategy strategy;

    // Thay đổi chiến lược vào lúc Runtime (Lúc đang chạy)
    public void setStrategy(ExportStrategy strategy) {
        this.strategy = strategy;
    }

    public void runExport() {
        // Đẩy trách nhiệm cho chiến lược tự làm
        strategy.exportReport(); 
    }
}

// Khi dùng:
public class Main {
    public static void main(String[] args) {
        ReportService service = new ReportService();
        
        // Khách đòi PDF
        service.setStrategy(new PdfExport());
        service.runExport();

        // Khách đổi ý đòi Excel (Không hề phải sửa một dòng if-else nào)
        service.setStrategy(new ExcelExport());
        service.runExport();
    }
}
```
**Hậu quả tốt:** Tháng sau có thêm báo cáo HTML, bạn chỉ việc tạo class `HtmlExport` mới, hệ thống cũ không suy suyển gì!

---

## 3. Các Pattern "Bắt buộc phải biết" trong Spring
Nếu bạn học Spring Framework, bạn đang sống trong thế giới của Design Patterns:
- **Singleton Pattern:** Mặc định toàn bộ các `@Service`, `@Repository` trong Spring đều là Singleton (Chỉ có 1 Object duy nhất được sinh ra để tiết kiệm RAM).
- **Proxy Pattern:** Xương sống của AOP và `@Transactional`.
- **Template Method Pattern:** Có trong `JdbcTemplate`, `RestTemplate`. 

---

## 4. Tổng hợp các Design Patterns (Bản đồ tra cứu)

### 🌟 4.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Rất hay gặp)
1. **Singleton:** Đảm bảo chỉ có 1 instance tồn tại (Spring Bean mặc định).
2. **Strategy:** Thay thế if-else chằng chịt bằng các class chiến lược tiêm vào lúc runtime.
3. **Factory Method:** Đưa việc khởi tạo object phức tạp vào một xưởng (Factory) thay vì dùng lệnh `new` lặp lại nhiều nơi.

### 🟡 4.2. Nhóm Trung cấp (Xây dựng Framework)
1. **Builder:** Rất hay dùng để tạo các Object có quá nhiều tham số.
2. **Observer:** Mô hình Pub/Sub, khi 1 đối tượng thay đổi trạng thái, nó báo cho các đối tượng đang đăng ký theo dõi.
3. **Proxy:** Bọc một object để chặn trước và sau khi gọi hàm của nó (Cốt lõi của `@Transactional` và AOP).

### 🔴 4.3. Nhóm Nâng cao (Kiến trúc phức tạp)
1. **Facade:** Gộp nhiều Service phức tạp lại thành 1 giao diện đơn giản duy nhất để client gọi.
2. **Adapter:** Biến đổi interface của một class thành interface mà client mong muốn.
3. **Decorator:** Thêm tính năng mới cho một object lúc chạy mà không cần sửa code class gốc.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Refactoring.Guru | [refactoring.guru](https://refactoring.guru/design-patterns/java) |
| Baeldung - Java Design Patterns | [baeldung.com](https://www.baeldung.com/java-patterns) |