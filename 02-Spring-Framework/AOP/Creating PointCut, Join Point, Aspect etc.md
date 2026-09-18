# 🛠 Thực hành AOP: Giải phẫu Aspect, PointCut, Advice & Join Point

> **Thuộc roadmap:** `2. Spring Framework → AOP → Creating PointCut, Join Point, Aspect etc`

---

## 1. Dịch thuật các thuật ngữ "Dị" của AOP (Ví dụ Y khoa)

Để hiểu các thuật ngữ AOP một cách dễ nhất, hãy tưởng tượng bạn là một Bác sĩ đang chuẩn bị phẫu thuật cho một bệnh nhân (Bệnh nhân ở đây chính là các class Service, Controller trong dự án của bạn).

1. **Join Point (Điểm nối - Chỗ CÓ THỂ mổ):** Trên cơ thể bệnh nhân, có rất nhiều chỗ bạn CÓ THỂ đụng dao mổ vào (Tay, chân, bụng, ngực). Trong Spring, Join Point chính là **Tất cả các hàm** đang tồn tại trong hệ thống (vì hàm nào cũng có thể bị chặn bởi Proxy).
2. **PointCut (Điểm cắt - Tọa độ CHÍNH XÁC để mổ):** Bạn không thể cầm dao rạch tùm lum. Bạn dùng bút lông khoanh tròn đúng 1 vị trí trên ruột thừa. PointCut chính là **câu lệnh lọc/định vị** (Ví dụ: "Chỉ chọn những hàm nào có dán nhãn @AuditLog").
3. **Advice (Hành động kẹp vào - Nhát dao mổ):** Tại cái tọa độ đó, bạn quyết định sẽ làm gì? Rạch trước (`@Before`), khâu lại sau (`@After`), hay làm trọn bộ từ đầu đến cuối (`@Around`)?
4. **Aspect (Khía cạnh - Ông Bác sĩ):** Là một file class bao trùm, gom tất cả 3 thứ trên (Tọa độ mổ + Hành động mổ) lại thành một gói hoàn chỉnh để tiêm vào IoC Container.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán kinh điển: **Xây dựng hệ thống Nhật ký kiểm toán (Audit Log)***. 
Sếp yêu cầu: *"Bất cứ khi nào có một user thực hiện hành động quan trọng (Sửa, Xóa), hệ thống phải tự động ghi vào Database xem user đó tên gì, gọi hàm nào, truyền vào tham số gì"*.

Thay vì đi vào hàng chục file Service và gõ lệnh `log.info(user, action, params)` rải rác khắp nơi, chúng ta sẽ giải quyết triệt để bằng AOP + Custom Annotation (đúng như tư duy cực kỳ hiện đại mà bạn vừa ngộ ra ở phần trước).

### Bước 1: Khai báo vũ khí (Maven)
Dù dùng Spring Boot, tính năng AOP mặc định vẫn tắt để tiết kiệm bộ nhớ. Bạn phải nhúng thư viện để kích hoạt nó:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### Bước 2: Tạo "Cái nhãn dán" (Custom Annotation)
Chúng ta tự chế ra một Annotation tên là `@AuditLog`. Bất cứ hàm nào gắn nhãn này sẽ bị AOP "bắt cóc".
```java
import java.lang.annotation.*;

@Target(ElementType.METHOD) // Cấm dán bậy, nhãn này chỉ được phép dán lên trên đầu CÁC HÀM
@Retention(RetentionPolicy.RUNTIME) // Sống dai tới tận lúc chương trình chạy
public @interface AuditLog {
    String action() default "Unknown Action"; // Một tham số để truyền vào mô tả hành động
}
```

### Bước 3: Tạo Ông Bác Sĩ (Aspect)
Đây là nơi chứa toàn bộ "Phép thuật" (Nơi kết hợp PointCut và Advice).

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;
import java.util.Arrays;

@Aspect // Khai báo tôi là Bác sĩ AOP
@Component // Khai báo tôi là một Bean để Spring nhét vào IoC Container
public class AuditLogAspect {

    // 1. CHỈ ĐỊNH TỌA ĐỘ MỔ (POINTCUT) kết hợp HÀNH ĐỘNG MỔ (ADVICE = @Around)
    // Dịch câu thần chú: "Hãy bao bọc (Around) TẤT CẢ những hàm nào có dán nhãn @AuditLog"
    @Around("@annotation(auditLogAnnotation)")
    public Object logAudit(ProceedingJoinPoint joinPoint, AuditLog auditLogAnnotation) throws Throwable {
        
        // --- ĐÂY LÀ PHẦN @BEFORE (Trước khi hàm gốc bắt đầu chạy) ---
        String functionName = joinPoint.getSignature().getName(); // Rút trích tên hàm
        Object[] args = joinPoint.getArgs(); // Rút trích mảng các tham số người dùng truyền vào
        
        System.out.println("⚠️ [AUDIT] Bắt đầu hành động: " + auditLogAnnotation.action());
        System.out.println("⚠️ [AUDIT] Đang gọi hàm: " + functionName);
        System.out.println("⚠️ [AUDIT] Tham số truyền vào: " + Arrays.toString(args));
        
        // --- CHO PHÉP HÀM GỐC CHẠY ---
        // (Thằng cò Proxy chính thức mở cửa cho code nghiệp vụ gốc bên trong chạy)
        Object result = joinPoint.proceed(); 
        
        // --- ĐÂY LÀ PHẦN @AFTER (Sau khi hàm gốc chạy xong thành công và có kết quả) ---
        System.out.println("✅ [AUDIT] Hành động " + auditLogAnnotation.action() + " đã hoàn tất thành công.");
        
        return result; // Trả kết quả cuối cùng về cho Client
    }
}
```

### Bước 4: Tận hưởng thành quả (Gắn nhãn vào Service)
Bây giờ, lập trình viên code Service cực kỳ nhàn hạ, sạch sẽ.
```java
@Service
public class ProductService {

    // Phép màu: Chỉ cần dán 1 cái nhãn, thằng Proxy AOP sẽ tự động làm mọi trò ghi Log rườm rà ở trên!
    @AuditLog(action = "Xóa Sản Phẩm")
    public void deleteProduct(Long productId) {
        System.out.println("Đang xóa sản phẩm " + productId + " khỏi CSDL...");
        // Logic xóa thật nằm ở đây, không hề bị rác code log lẫn lộn vào.
    }
}
```

---

## 2. Tổng hợp các chủ đề AOP (Bản đồ tra cứu khi viết code)

Dưới đây là cẩm nang để bạn tra cứu khi thực hành viết code AOP trong dự án:

### 🌟 2.1. Nhóm Annotation & Object ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc dùng thường xuyên)
1. **`@Aspect` & `@Component`:** Luôn đi thành 1 cặp trên đầu class. Phải có `@Component` thì Spring mới lôi cái Aspect của bạn vào hoạt động được.
2. **`@Around`:** Là loại Advice quyền lực nhất (bọc cả đầu và đuôi hàm). 90% các trường hợp bạn chỉ cần dùng `@Around` là đủ giải quyết mọi việc (đỡ phải viết lắt nhắt 1 hàm `@Before` rồi 1 hàm `@After` riêng rẽ).
3. **`ProceedingJoinPoint`:** Object chứa mọi thông tin về cái hàm đang bị chặn. Chứa luôn cả nút bấm `proceed()` để quyết định có cho hàm gốc chạy tiếp hay không. (Ứng dụng đỉnh cao: Nếu code AOP của bạn là Check Quyền, nếu check thất bại, bạn bỏ qua lệnh `proceed()`, báo lỗi luôn, hàm gốc vĩnh viễn không được chạy).
4. **PointCut bằng Annotation `@annotation(...)`:** Cách cấu hình tọa độ đỉnh cao và chuẩn xác nhất hiện nay (y chang cách chúng ta làm ở ví dụ trên).

### 🟡 2.2. Nhóm Advice phân mảnh & Cú pháp Regex (Trung cấp)
1. **`@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`:** Các loại Advice bị băm nhỏ. Đặc biệt `@AfterThrowing` rất hay dùng độc lập để chặn bắt các lỗi Exception và bắn thông báo cảnh báo qua Telegram/Slack cho team dev.
2. **PointCut bằng Regex `execution(...)`:** Cú pháp cổ điển để quét hàng loạt class mà không cần dán nhãn thủ công từng hàm. 
   - Ví dụ: `execution(* com.congty.service.*.*(..))` (Quét mọi hàm, mọi file trong package service). Nhược điểm là rất bạo lực và dễ lỡ tay quét trúng những hàm không mong muốn.

### 🔴 2.3. Nhóm Kỹ thuật Nâng cao (Tùy biến luồng AOP)
1. **`@Order`:** Xử lý đụng độ. Khi bạn có 2 thằng Cò (2 Aspect) cùng chặn vào một hàm (Ví dụ: Thằng Cò Security chặn check quyền, Thằng Cò Audit chặn ghi Log). Bạn BẮT BUỘC phải dùng `@Order(1)`, `@Order(2)` để phân định xem thằng cò nào được đứng lớp ngoài cùng bao bọc thằng nào bên trong.
2. **Pointcut Designators phức tạp (`@within`, `@args`, `bean()`):** Các câu thần chú nâng cao để chặn dựa trên kiểu dữ liệu của tham số truyền vào, hoặc chặn mọi class có chung một tên hậu tố là `*ServiceImpl` hoặc dựa vào tên Bean trong IoC Container.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Spring Boot 4.1.0 — AOP Auto-configuration | [docs.spring.io](https://docs.spring.io/spring-boot/reference/core/spring-application.html) |
| Spring 7.0.8 — Declaring an Aspect | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/aspects.html) |
| Spring 7.0.8 — Pointcut Expressions | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/pointcuts.html) |

---

> **Tiếp theo trong roadmap:** Chúc mừng bạn đã hoàn thành trụ cột thứ 2 của Spring Framework! Chúng ta sẽ chuyển sang một chương vĩ đại mới: `Spring Boot` - Khám phá tại sao cấu hình Spring thuần lại đau khổ, và Spring Boot đã sinh ra để "Cứu rỗi" thế giới như thế nào.
