# 🎭 AOP Hoạt động như thế nào? (How AOP Works)

> **Thuộc roadmap:** `2. Spring Framework → AOP → How AOP Works`

---

## 1. Mảnh ghép còn thiếu của lập trình hướng đối tượng (OOP)

Spring Framework nổi danh giang hồ nhờ 2 trụ cột sức mạnh (2 khái niệm đã làm thay đổi hoàn toàn cách chúng ta thiết kế hệ thống Java):
1. **IoC (Inversion of Control):** Dùng để quản lý vòng đời và tiêm phụ thuộc (DI).
2. **AOP (Aspect-Oriented Programming):** Lập trình hướng khía cạnh. Nó được sinh ra để khắc phục "điểm mù" chí mạng của Lập trình Hướng Đối Tượng (OOP).

Vậy điểm mù của OOP là gì? OOP chia nhỏ chương trình thành các Đối tượng rời rạc (Ví dụ: `UserService`, `OrderService`, `ProductService`). 
Tuy nhiên, trong một hệ thống thực tế, có những đoạn logic **KHÔNG thuộc về riêng một đối tượng nào**, nhưng đối tượng nào cũng cần phải gọi nó. Các đoạn logic vắt ngang này được gọi là **Cross-cutting concerns (Mối bận tâm cắt ngang)**.
*Ví dụ phổ biến:* Ghi Log (nhật ký hệ thống), Kiểm tra quyền (Security), Mở/Đóng giao dịch Database (Transaction Management).

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Hãy cùng xem xét bài toán thực tế: **Đo lường thời gian chạy của các hàm API để tìm ra hàm nào làm ứng dụng bị chậm***.

### Nỗi đau 1: Mớ bòng bong (Code Tangling) & Rải rác khắp nơi (Code Scattering)
Giả sử sếp yêu cầu bạn in ra thời gian chạy của mọi hàm. Nếu chỉ dùng OOP thuần túy, bạn sẽ giải bài toán này như sau:
```java
@Service
public class OrderService {
    public void createOrder() {
        // --- LOGIC XEN NGANG (Cross-cutting) ---
        long start = System.currentTimeMillis(); 
        System.out.println("Bắt đầu gọi hàm tạo đơn hàng...");
        
        // --- LOGIC NGHIỆP VỤ CHÍNH CỦA CHÚNG TA ---
        System.out.println("Đang lưu đơn hàng vào Database...");
        
        // --- LOGIC XEN NGANG (Cross-cutting) ---
        long end = System.currentTimeMillis();
        System.out.println("Tạo đơn hàng mất: " + (end - start) + "ms");
    }
}
```
**Nhìn thì dễ, nhưng hệ lụy là khổng lồ:**
- **Code Tangling (Mớ bòng bong):** Logic nghiệp vụ chính (lưu đơn hàng) bị che lấp bởi hàng đống code phụ trợ (đo thời gian, in log).
- **Code Scattering (Code bị rải rác):** Nếu dự án của bạn có 100 cái Service, bạn phải copy-paste cái đoạn đo thời gian (start, end) kia vào 100 file khác nhau. Sếp đổi ý muốn ghi log ra file thay vì in ra màn hình? Chúc mừng, bạn phải mở 100 file ra sửa!

### Giải pháp của AOP: Dao mổ tách lớp
Ý tưởng của AOP rất kỳ diệu: **Rút toàn bộ những đoạn code rác phụ trợ ra khỏi hàm, nhét vào một file duy nhất. Sau đó "ám" (kẹp) cái file đó vào bất kỳ hàm nào bạn muốn mà tuyệt đối không cần đụng vào code gốc.**

**1. File logic nghiệp vụ gốc: (Sạch bóng code rác)**
```java
@Service
public class OrderService {
    // Code trở nên cực kỳ thuần khiết, chỉ tập trung vào nghiệp vụ bán hàng
    public void createOrder() {
        System.out.println("Đang lưu đơn hàng vào Database...");
    }
}
```

**2. File AOP (Nơi chứa logic cắt ngang):**
```java
@Aspect
@Component
public class TimeLoggingAspect {
    // Câu thần chú: "Hãy chạy hàm đo thời gian này bao bọc LẤY TẤT CẢ các hàm đang nằm trong thư mục Service của dự án"
    @Around("execution(* com.myapp.service.*.*(..))")
    public Object measureTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis(); 
        
        // Mượn đường cho chạy hàm gốc (ví dụ hàm tạo đơn hàng)
        Object result = joinPoint.proceed(); 
        
        long end = System.currentTimeMillis();
        System.out.println("Hàm " + joinPoint.getSignature().getName() + " chạy mất: " + (end - start) + "ms");
        
        return result;
    }
}
```
**Giá trị mang lại cực lớn:** Bây giờ, dự án của bạn có thêm 1000 file Service đi chăng nữa, lập trình viên viết logic cứ việc viết, không cần bận tâm đến việc đo thời gian. Cái "Bóng ma" (Aspect) kia sẽ tự động bao bọc tất cả và làm việc đó âm thầm đằng sau cánh gà.

---

## 2. AOP thực chất hoạt động như thế nào? (Bí mật đằng sau)

Khi bạn gắn `@Aspect` hoặc gọi hàm từ một Bean, bằng phép thuật nào Spring có thể nhảy vào chen ngang đoạn log trước khi hàm thực thi? 
Câu trả lời không có ma thuật nào cả, nó dựa trên một kỹ thuật đỉnh cao của Java: **Proxy Pattern (Người đại diện / Thằng cò)**.

Khi bạn yêu cầu IoC Container lấy cái `OrderService` ra xài, Spring **KHÔNG** đưa cho bạn `OrderService` thật. 
Nó lén tạo ra một bản sao giả mạo (gọi là Proxy). Bản sao giả mạo này có tên các hàm y chang bản thật.

**Luồng chạy thực tế:**
1. Khi Controller gọi `orderService.createOrder()`, thực chất nó đang gọi vào cái **Proxy (Thằng cò)**.
2. Thằng cò bắt đầu bấm đồng hồ (`start = System.currentTimeMillis()`).
3. Thằng cò chạy xuống gọi `createOrder()` của thằng thật.
4. Thằng thật làm xong báo lên cho thằng cò.
5. Thằng cò bấm đồng hồ dừng, in log ra màn hình, rồi mới trả kết quả cuối cùng về cho Controller.

Vì cả Thằng cò và Thằng thật đều bị giam chung trong một nhà tù IoC Container, nên sự đánh tráo này là **trong suốt** đối với bạn. Bạn cứ ngỡ mình đang gọi hàm gốc, nhưng thực ra mọi thứ đã bị chặn lại kiểm duyệt ở cửa. 
Kỹ thuật đình đám nhất ứng dụng cơ chế Thằng Cò này chính là annotation `@Transactional` (dùng để quản lý tính toàn vẹn của Database).

---

## 3. Tổng hợp các chủ đề cốt lõi của AOP (Bản đồ tra cứu)

AOP là một thuật ngữ vô cùng trừu tượng. Để thành thạo, bạn hãy bám vào lộ trình tra cứu sau:

### 🌟 3.1. Nhóm lý thuyết ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc hiểu bản chất)
1. **Lý thuyết Proxy Pattern:** Bắt buộc phải hiểu khái niệm "Thằng đại diện" (Proxy) chèn vào giữa. Lý thuyết này sẽ giúp bạn giải thích bài toán phỏng vấn kinh điển: *"Tại sao khi gọi một hàm A, hàm A gọi tiếp hàm B trong CÙNG MỘT CLASS thì annotation `@Transactional` hay `@Async` gắn trên hàm B lại không hoạt động?"* (Đáp án: Vì trong cùng 1 class thì hàm gọi trực tiếp lẫn nhau, không qua thằng Proxy, nên luồng AOP bị vô hiệu hóa).
2. **Khái niệm Aspect:** Hiểu rằng Aspect chính là cái mảng logic cắt ngang (ví dụ: Security, Caching, Logging) được tách ra thành một file độc lập.

### 🟡 3.2. Nhóm thuật ngữ chuyên ngành AOP (Trung cấp)
AOP sử dụng một bộ từ vựng rất "dị", bạn bắt buộc phải làm quen (Bài học tiếp theo sẽ hướng dẫn cách code thực tế các từ vựng này):
1. **Join Point (Điểm nối):** Một điểm dừng chân cụ thể trong luồng chạy ứng dụng (Ví dụ: Ngay trước khi gọi hàm A, hoặc ngay sau khi hàm B quăng lỗi Exception).
2. **Advice (Hành động kẹp vào):** Đoạn code bạn muốn nhét thêm vào (Ví dụ: Code in log). Có các loại Advice cơ bản: `@Before` (chạy trước), `@After` (chạy sau), `@Around` (bọc cả đầu lẫn đuôi).
3. **Pointcut (Điểm cắt):** Câu thần chú dạng cú pháp Regex để chỉ định chính xác cái *Advice* sẽ được nhét vào *những* *Join Point* nào (Ví dụ: Câu lệnh `execution(* com.myapp.service..*)` để lọc ra toàn bộ các hàm thuộc thư mục Service).

### 🔴 3.3. Nhóm kiến trúc Nâng cao (Lõi Spring)
1. **JDK Dynamic Proxy vs CGLIB Proxy:** Tìm hiểu sâu về 2 thuật toán mà Spring sử dụng để chế tạo ra "Thằng đại diện". Nếu class của bạn có kế thừa Interface thì nó dùng JDK, không có Interface thì nó tự ép dùng CGLIB (trực tiếp can thiệp vào bytecode class).
2. **Spring AOP vs AspectJ:** Cần biết rằng Spring AOP chỉ là phiên bản "rút gọn" (chỉ cho phép kẹp code vào thời điểm 1 hàm được gọi - Method Execution). AspectJ mới là framework full phép thuật của Java, có thể kẹp code vào tận lúc khởi tạo biến, thay đổi giá trị biến, tạo constructor...

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Spring 7.0.8 — AOP Concepts | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html) |
| Spring 7.0.8 — Spring AOP vs AspectJ | [docs.spring.io](https://docs.spring.io/spring-framework/reference/core/aop/introduction-proxies.html) |

---

> **Tiếp theo trong roadmap:** `Creating PointCut, Join Point, Aspect etc` (Chúng ta sẽ chuyển từ phần lý thuyết khái niệm sang cách gõ code thực tế, giải phẫu 100% các thành phần để tạo nên một Aspect hoàn chỉnh trong Spring).
