# 📜 Nghệ thuật Ghi Nhật Ký (Logging trong Java)

> **Thuộc roadmap:** `1. Java → Logging (Log4j, Log4j2, Logback)`

---

## 1. Tại sao phải dùng Logging thay vì `System.out.println`?

Khi mới học Java, công cụ gỡ lỗi yêu thích nhất của mọi người là `System.out.println("Chạy tới đây rồi!")`. Nhưng khi đi làm thực tế, nếu bạn viết dòng lệnh này vào dự án, bạn sẽ bị phạt ngay lập tức. 

Tại sao?
1. **Không thể tắt bật:** Khi đẩy lên Production, hàng ngàn lệnh `println` sẽ liên tục in ra làm tràn màn hình, tốn CPU vô ích và không có cách nào tắt chúng đi ngoài việc xóa code.
2. **Không lưu trữ được:** `println` chỉ in ra màn hình Console tạm thời. Nếu Server bị sập lúc 2 giờ sáng, sáng hôm sau bạn mở máy lên thì màn hình Console đen thui, không có vết tích gì để tìm hiểu nguyên nhân sập.
3. **Không có phân loại mức độ:** Bạn không phân biệt được dòng nào là thông báo bình thường, dòng nào là lỗi nghiêm trọng.

Đó là lý do các **Logging Frameworks** ra đời. Chúng cho phép in ra màu sắc, tự động lưu vào file `.log` (mỗi ngày 1 file), tự động nén file, và lọc hiển thị theo các Cấp độ (Log Levels).

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn viết chức năng chuyển tiền. Có lỗi xảy ra, bạn cần tìm dấu vết lúc 2h sáng.*

### Giải pháp: Viết Log chuẩn mực với Logback
Spring Boot tích hợp sẵn thư viện **Logback** (xịn hơn Log4j). Bạn sẽ khai báo đối tượng `Logger` (hoặc dùng `@Slf4j` của Lombok) và in log theo các cấp độ chuẩn.

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j // Lombok tự động tạo ra một biến tên là 'log' để bạn xài
@Service
public class TransferService {

    public void transferMoney(String fromAccount, String toAccount, double amount) {
        
        // CẤP ĐỘ INFO: Dùng để ghi lại các mốc sự kiện quan trọng của hệ thống (Để theo dõi)
        log.info("Bắt đầu thực hiện giao dịch chuyển {} từ {} sang {}", amount, fromAccount, toAccount);

        // CẤP ĐỘ DEBUG: Dùng để ghi lại các giá trị chi tiết khi lập trình viên cần soi lỗi (Rất nhiều rác)
        log.debug("Số dư hiện tại của {}: 1000", fromAccount); 
        
        try {
            if (amount < 0) {
                // CẤP ĐỘ WARN: Dùng để cảnh báo hiện tượng bất thường nhưng hệ thống CHƯA chết.
                log.warn("Cảnh báo: Có người cố tình chuyển số tiền ÂM: {}", amount);
                return;
            }
            
            // Lỗi giả lập: Bị mất mạng kết nối ngân hàng
            throw new RuntimeException("Mất kết nối tới máy chủ ngân hàng đích!");
            
        } catch (Exception e) {
            // CẤP ĐỘ ERROR: Dùng khi tính năng bị sập. BẮT BUỘC phải truyền biến 'e' (Exception) vào cuối cùng
            // Để hệ thống log in ra toàn bộ lịch sử dấu vết (Stack Trace)
            log.error("Giao dịch thất bại! Lỗi nghiêm trọng: {}", e.getMessage(), e);
        }
    }
}
```

**Giá trị mang lại (Nhờ file cấu hình `application.yml`):**
```yaml
logging:
  level:
    root: INFO # Dòng cấu hình quyền lực nhất!
```
Chỉ với 1 cấu hình trên, toàn bộ các lệnh `log.debug` trong hệ thống sẽ **bị câm hoàn toàn**. Chúng bị tắt đi để tiết kiệm CPU cho máy chủ. Hệ thống chỉ in ra những log nào có cấp độ từ `INFO` trở lên (`INFO`, `WARN`, `ERROR`).
Nếu một ngày đẹp trời hệ thống bị lỗi bí ẩn, bạn chỉ cần mở file này sửa lại thành `DEBUG` rồi restart, toàn bộ rác sẽ hiện ra để bạn điều tra. Tuyệt đối không cần sửa code Java!

---

## 3. Tổng hợp các chủ đề Logging (Bản đồ tra cứu)

Thế giới Logging trong Java hơi rối rắm vì nó có "lịch sử cạnh tranh" rất dài giữa các thư viện. Dưới đây là những gì bạn cần nắm chắc:

### 🌟 3.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc thuộc lòng)
1. **SLF4J (Simple Logging Facade for Java):** Đây KHÔNG PHẢI là thư viện in ra log. Đây là một bộ giao diện (Facade). Nó đóng vai trò "cò mồi". Khi viết code, bạn KHÔNG gọi trực tiếp thư viện Log4j hay Logback. Bạn gọi qua SLF4J (bằng `@Slf4j`). Nếu sau này công ty bạn cấm dùng Logback bắt chuyển sang Log4j2, bạn chỉ cần thay thư viện trong `pom.xml`, code Java không phải sửa một chữ nào!
2. **6 Cấp độ Log (Log Levels):** Cần thuộc thứ tự độ nghiêm trọng từ thấp lên cao: 
   - `TRACE` (Siêu chi tiết, cực rác) -> `DEBUG` (Dùng để fix bug) -> `INFO` (Thông báo mốc sự kiện bình thường) -> `WARN` (Cảnh báo bất thường) -> `ERROR` (Lỗi sập luồng) -> `FATAL` (Lỗi sập toàn bộ Server).
3. **Logback:** Là thư viện in log được cấu hình làm mặc định trong Spring Boot. Tốc độ rất nhanh và chiếm thị phần số 1 hiện tại.

### 🟡 3.2. Nhóm Cấu hình file (Trung cấp)
1. **Cấu hình `logback-spring.xml`:** Thay vì chỉ in ra console, bạn sẽ học cách viết file XML cấu hình cho Logback. 
2. **Rolling File Appender (File cuộn):** Cấu hình đỉnh cao nhất mà mọi công ty đều làm: *"Hãy lưu log vào file `app.log`. Nếu file này phình to quá 10MB, hãy nén nó thành file ZIP, đổi tên theo ngày (ví dụ `app-2023-10-15.zip`) và cất đi. Chỉ giữ lại tối đa log của 30 ngày gần nhất để không làm đầy ổ cứng Server"*.

### 🔴 3.3. Nhóm Nâng cao (Thay thế và Tối ưu hóa)
1. **Log4j2 (Thư viện mạnh nhất hiện nay):** Logback rất tốt, nhưng Log4j2 (của Apache) sử dụng kỹ thuật Asynchronous (Bất đồng bộ - Disruptor) khiến tốc độ in log của nó đè bẹp cả Logback. Trong các dự án chứng khoán, siêu chịu tải, người ta sẽ dùng lệnh `exclude` để xóa Logback ra khỏi Spring Boot và thay bằng Log4j2.
2. **Tích hợp ELK Stack (Elasticsearch, Logstash, Kibana):** Khi công ty có 100 cái Server, bạn không thể remote vào từng Server mở file `.log` để đọc. Bạn sẽ cấu hình để 100 Server đó bắn log qua mạng TCP về một Server Trung Tâm (ELK) để dễ dàng search lỗi trên trình duyệt.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Spring Boot 4.1.0 — Logging | [docs.spring.io](https://docs.spring.io/spring-boot/reference/core/features.html#boot-features-logging) |
| SLF4J Manual | [slf4j.org](https://www.slf4j.org/manual.html) |
