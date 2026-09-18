# 🧪 Nghệ thuật Kiểm thử (Unit Test & Integration Test với JUnit & Mockito)

> **Thuộc roadmap:** `1. Java → Testing`

---

## 1. Kiểm thử phần mềm là gì?

Lập trình viên viết xong code, bấm nút Run thấy không có lỗi, dùng Postman gọi API thấy trả về kết quả đúng. Vậy là xong việc đúng không? 
**Sai hoàn toàn.**
Ngày hôm nay bạn code đúng, nhưng 1 tháng sau, đồng nghiệp của bạn vào sửa một hàm khác và vô tình làm chết cái API của bạn. Mọi thứ nổ tung khi đẩy lên Production. Đó là lý do chúng ta phải viết **Code tự động kiểm tra Code (Automated Testing)**.

Có 2 cấp độ kiểm thử cốt lõi mà lập trình viên Backend phải làm:
1. **Unit Testing (Kiểm thử đơn vị):** Kiểm tra **1 hàm duy nhất** chạy trong môi trường hoàn toàn bị cô lập. Tốc độ cực nhanh (vài milli-giây).
2. **Integration Testing (Kiểm thử tích hợp):** Kiểm tra **nhiều thành phần liên kết với nhau** (Ví dụ: Controller gọi Service, Service gọi Database). Chậm hơn vì phải kết nối mạng, Database thật.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn có một `OrderService` chuyên tính toán giảm giá, sau đó gọi xuống `OrderRepository` để lưu vào CSDL MySQL.*
Bạn muốn viết **Unit Test** cho hàm tính giảm giá. Nhưng khổ nỗi, cứ chạy test là nó lại đòi kết nối MySQL, mà lúc đó máy bạn chưa cài MySQL thì Test báo lỗi (mặc dù logic tính toán của bạn đúng).

**Giải pháp: Dùng "Mô hình nộm" (Mocking) bằng Mockito.**
Trong Unit Test, bạn không được phép kết nối mạng, không được đụng vào CSDL. Thay vào đó, bạn sẽ tạo ra một cái `OrderRepository` giả mạo (Mock Object). Cái kho giả mạo này sẽ luôn luôn báo "Lưu thành công" bất kể bạn đưa cho nó cái gì. Nhờ vậy, bạn chỉ tập trung 100% vào việc kiểm tra **thuật toán tính toán giảm giá** của mình.

### Code Thực hành (JUnit 5 + Mockito)

```java
// Logic nghiệp vụ cần test (Class thật)
@Service
public class OrderService {
    private final OrderRepository repository; // Phụ thuộc vào Database
    
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }

    public double calculateDiscount(double price) {
        if (price > 1000) return price * 0.9; // Giảm 10%
        return price;
    }

    public boolean placeOrder(double price) {
        double finalPrice = calculateDiscount(price);
        return repository.saveOrder(finalPrice); // Chỗ này sẽ đụng Database!
    }
}
```

```java
// File viết Test (JUnit + Mockito)
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

@ExtendWith(MockitoExtension.class) // Kích hoạt Mockito
public class OrderServiceTest {

    @Mock
    private OrderRepository mockRepository; // 1. Chế tạo Kho giả mạo (Không đụng DB thật)

    @InjectMocks
    private OrderService orderService; // 2. Bơm cái Kho giả mạo đó vào Service thật

    @Test
    public void testCalculateDiscount() {
        // Kiểm tra logic tính toán (Dùng JUnit Assertions)
        double result = orderService.calculateDiscount(1500);
        assertEquals(1350.0, result, "Nếu mua trên 1000, phải giảm 10%");
    }

    @Test
    public void testPlaceOrder() {
        // Dạy cho Kho giả mạo cách nói dối: "Hễ ai gọi hàm saveOrder, cứ trả về TRUE"
        Mockito.when(mockRepository.saveOrder(Mockito.anyDouble())).thenReturn(true);

        // Chạy hàm nghiệp vụ thật
        boolean isSuccess = orderService.placeOrder(1500);

        // Kiểm tra kết quả
        assertTrue(isSuccess);
        
        // Kiểm tra xem Kho giả mạo CÓ BỊ GỌI ĐÚNG 1 LẦN hay không
        Mockito.verify(mockRepository, Mockito.times(1)).saveOrder(1350.0);
    }
}
```

**Giá trị mang lại:**
- Test chạy siêu nhanh (Không tốn mili giây nào để kết nối DB).
- Nếu test `testCalculateDiscount` báo đỏ (Failed), bạn chắc chắn 100% thuật toán của bạn sai, chứ không phải do rớt mạng hay DB chết. Đó là sự cô lập hoàn hảo của Unit Test.

---

## 3. Tổng hợp chủ đề Testing (Bản đồ tra cứu)

Khi đi làm, Test là tiêu chuẩn bắt buộc (Nhiều công ty ép phải có đủ 80% code coverage thì mới cho đẩy code). Dưới đây là các từ khóa bạn cần master:

### 🌟 3.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bộ khung bắt buộc)
1. **JUnit 5 Annotations:** Khung xương của mọi bài Test. 
   - `@Test` (Đánh dấu hàm này là hàm chạy test).
   - `@BeforeEach` / `@AfterEach` (Chạy trước/sau mỗi hàm test để dọn dẹp dữ liệu).
   - `@BeforeAll` / `@AfterAll` (Chạy 1 lần duy nhất trước khi cả class bắt đầu test).
2. **Assertions (Xác nhận):** Các hàm kiểm tra kết quả `assertEquals()`, `assertTrue()`, `assertThrows()` (kiểm tra xem hàm có ném ra đúng lỗi Exception như mong đợi không).
3. **Cú pháp BDD (Behavior-Driven Development):** Cấu trúc viết 1 hàm test cực kỳ dễ đọc gồm 3 bước chuẩn quốc tế: `Given` (Giả sử có dữ liệu này) - `When` (Khi chạy hàm này) - `Then` (Thì kết quả phải là thế này).

### 🟡 3.2. Nhóm Mockito (Mô phỏng Trung cấp)
1. **`@Mock` vs `@InjectMocks`:** Phân biệt rõ đâu là con rối giả mạo, đâu là nạn nhân bị tiêm con rối vào.
2. **`Mockito.when(...).thenReturn(...)`:** Dạy con rối cách trả lời.
3. **`Mockito.verify(...)`:** Xác minh xem con rối có bị gọi đúng cách không (Rất hay dùng khi test các hàm không trả về kết quả `void` mà chỉ lưu DB).

### 🔴 3.3. Nhóm Integration Testing (Nâng cao)
1. **`@SpringBootTest`:** Annotation vô cùng nặng nề. Nó sẽ khởi động TOÀN BỘ Spring IoC Container của bạn (tốn mấy chục giây y như chạy app thật) để test thử xem mọi thứ từ Web xuống DB có chạy thông suốt hay không.
2. **Testcontainers:** Kỹ thuật đỉnh cao. Khi chạy Integration Test, thư viện này sẽ tự động bật 1 cái Docker ảo chứa MySQL thật lên, nhồi dữ liệu test vào, chạy xong thì xóa Docker đó đi. Đảm bảo môi trường sạch 100%.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| JUnit 5 User Guide | [junit.org](https://junit.org/junit5/docs/current/user-guide/) |
| Mockito Documentation | [javadoc.io](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html) |
