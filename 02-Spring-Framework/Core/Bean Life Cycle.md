# ♻️ Vòng đời của một Bean (Bean Life Cycle)

> **Thuộc roadmap:** `2. Spring Framework → Core → Bean Life Cycle`

---

## 1. Vòng đời của Bean là gì?

Cũng giống như con người sinh ra, lớn lên, làm việc và qua đời, một Bean (bất kể là `@Service`, `@Controller` hay `@Component`) khi được Spring IoC Container quản lý cũng trải qua một vòng đời vô cùng khắt khe. 

Quy trình chuẩn của một vòng đời Bean diễn ra như sau:
1. **Sinh ra (Instantiation):** Spring gọi từ khóa `new` để tạo Object.
2. **Nhồi nhét (Populate Properties / DI):** Spring chích (inject) các phụ thuộc (các Bean khác) vào Object này bằng `@Autowired` hoặc Constructor.
3. **Thức tỉnh (`@PostConstruct`):** Spring cho phép Bean chạy một hàm chuẩn bị cuối cùng trước khi chính thức đưa vào phục vụ.
4. **Cống hiến (Ready for use):** Nhận request, xử lý logic, trả kết quả. 
5. **Hấp hối (`@PreDestroy`):** Khi tắt Server, Spring cho phép Bean chạy một đoạn code để trăn trối (ví dụ: dọn dẹp bộ nhớ).
6. **Chết (Destroyed):** Bị thu hồi bộ nhớ.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Hãy cùng xem xét bài toán: **Tự động tải dữ liệu và Đóng kết nối an toàn***.

### Vấn đề thực tế
1. **Lúc khởi động:** Bạn có một `CurrencyService` chuyên gọi API ngân hàng lấy tỷ giá ngoại tệ. Bạn không muốn đợi user đầu tiên truy cập mới bắt đầu gọi API (làm user đó phải đợi rất lâu). Bạn muốn **ngay khoảnh khắc Server vừa bật lên thành công**, tỷ giá đã phải được tải sẵn về RAM.
2. **Lúc tắt máy:** Bạn có một `WebSocketService` đang giữ 1000 kết nối tới người dùng. Nếu tắt Server đột ngột, người dùng sẽ bị văng lỗi. Bạn muốn Server báo cho user biết trước khi tắt.

### Giải pháp: Sử dụng `@PostConstruct` và `@PreDestroy`
Thay vì tự phải viết một cái còi báo động để canh me lúc server bật/tắt, bạn chỉ cần dùng 2 Annotation này dán lên các hàm thông thường. IoC Container sẽ tự động gọi chúng đúng thời điểm.

```java
@Service
public class CurrencyService {

    // 1. IoC khởi tạo Object này, chích các dependency vào xong xuôi.

    // 2. NGAY LẬP TỨC tự động gọi hàm này (Thức tỉnh)
    @PostConstruct
    public void init() {
        System.out.println("Máy chủ vừa bật xong! Đang gọi API lấy tỷ giá...");
        // Code load dữ liệu từ API ngân hàng bỏ vào RAM
    }

    // 3. Hàm phục vụ bình thường (Cống hiến)
    public double getExchangeRate(String currency) {
        return 25000.0;
    }

    // 4. Khi sếp bấm nút Shut Down Server (Hấp hối)
    @PreDestroy
    public void cleanup() {
        System.out.println("Máy chủ đang tắt... Tiến hành đóng sạch kết nối Database và giải phóng RAM...");
    }
}
```

**Giá trị mang lại:** Bạn nắm quyền kiểm soát tuyệt đối trạng thái của mọi class trong hệ thống. Rất hữu ích để thực hiện **Warm-up** (làm nóng hệ thống trước khi chạy thật) và **Graceful Shutdown** (tắt hệ thống một cách êm ái).

---

## 3. Tổng hợp các chủ đề Vòng đời Bean (Bản đồ tra cứu)

Nếu bạn có nhu cầu can thiệp sâu vào Vòng đời của hệ thống, hãy tra cứu các từ khóa sau:

### 🌟 3.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Rất hay dùng)
1. **`@PostConstruct`:** Hàm khởi động chạy ngay sau khi Spring tạo Bean và Inject hoàn tất. Bắt buộc hàm này không được có tham số truyền vào, và trả về `void`.
2. **`@PreDestroy`:** Hàm trăn trối dọn dẹp chạy ngay trước khi Bean bị tiêu hủy. (Lưu ý: Nếu chương trình bị crash/kill đột ngột bằng lệnh `kill -9` trong Linux, hàm này sẽ không kịp chạy).

### 🟡 3.2. Nhóm Trung cấp (Cú pháp đồ cổ của Spring)
Trước khi Java đẻ ra `@PostConstruct`, Spring ép lập trình viên phải xài các Interface sau để làm việc tương tự (Bạn cần biết để bảo trì các dự án cũ từ 10 năm trước):
1. **`InitializingBean` (Interface):** Ép class của bạn phải cài đặt hàm `afterPropertiesSet()`.
2. **`DisposableBean` (Interface):** Ép class của bạn phải cài đặt hàm `destroy()`.
*(Ngày nay chúng ta không xài cách này nữa vì nó ép code của bạn phải dính chặt với Interface của Spring).*

### 🔴 3.3. Nhóm Nâng cao (Thay đổi lõi của Spring)
1. **`BeanPostProcessor` (Interface):** Đây là cảnh giới tối cao của việc thao túng vòng đời Bean. Nếu `@PostConstruct` chỉ tác động lên 1 class của bạn, thì `BeanPostProcessor` tác động lên **TẤT CẢ** mọi class trong toàn bộ hệ thống! 
   - Nó cho phép bạn chui vào giữa quá trình Spring tạo Bean, bạn có quyền kiểm tra xem Bean đó là gì, chỉnh sửa nó, bọc nó bằng Proxy (AOP), rồi mới trả nó lại cho hệ thống hoạt động.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Spring 7.0.8 — Bean Lifecycle | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html) |
| Spring 7.0.8 — @PostConstruct | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/postconstruct-and-predestroy.html) |
