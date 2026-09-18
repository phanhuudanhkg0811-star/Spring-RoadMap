# 🖥️ Nghệ thuật Triển khai Ứng dụng (Java Servers: Tomcat, Jetty, WildFly, WebLogic, WebSphere)

> **Thuộc roadmap:** `1. Java → Servers (Tomcat, Weblogic, JBOSS, WebSphere, Jetty)`

---

## 1. Java Server là gì? Tại sao cần nó?

Bạn viết xong code Java, bấm Run trên IntelliJ, thấy chương trình chạy đúng. Nhưng có một vấn đề: **Chương trình Java thuần chỉ chạy rồi tắt**. Nó chạy xong hàm `main()` là kết thúc. Vậy làm sao để code Java của bạn **lắng nghe liên tục 24/7** trên cổng 8080, chờ đợi hàng triệu request HTTP từ người dùng gửi đến?

Câu trả lời: **Bạn cần một Java Application Server** (hoặc gọi tắt là Server / Container).

Server đóng vai trò:
1. **Lắng nghe mạng (Network Listener):** Mở cổng TCP (ví dụ 8080), tiếp nhận kết nối HTTP từ trình duyệt / mobile app.
2. **Phân phối request (Request Dispatcher):** Khi nhận được request `GET /api/users`, Server sẽ tìm đúng Servlet/Controller của bạn để xử lý.
3. **Quản lý vòng đời (Lifecycle Management):** Server tạo ra các đối tượng Servlet, quản lý thread pool, session, và tự động dọn dẹp bộ nhớ.
4. **Cung cấp các dịch vụ Enterprise (Chỉ có ở Full Application Server):** Connection Pool, Transaction Manager, Message Queue (JMS), Security, JNDI...

### Hai loại Server trong Java

| Loại | Mô tả | Ví dụ |
|------|--------|-------|
| **Servlet Container** (Web Container) | Chỉ hỗ trợ **Servlet + JSP** (phần Web của Java EE). Nhẹ và nhanh. | Tomcat, Jetty, Undertow |
| **Full Application Server** | Hỗ trợ **toàn bộ Java EE / Jakarta EE** (EJB, JMS, JTA, JPA, CDI...). Nặng nhưng đầy đủ tính năng. | WildFly (JBoss), WebLogic, WebSphere |

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn được giao viết REST API cho ứng dụng thương mại điện tử. Bạn cần chọn Server để triển khai. CTO hỏi bạn: "Chọn Tomcat hay WebLogic?" — Bạn phải trả lời được.*

### Giải pháp: Hiểu rõ từng Server để chọn đúng công cụ

#### 🍅 Tomcat — Vua của thế giới Spring Boot

**Apache Tomcat** là Servlet Container phổ biến nhất thế giới Java. **Spring Boot tích hợp sẵn Tomcat bên trong** (Embedded Tomcat), nghĩa là khi bạn chạy `java -jar app.jar`, Tomcat tự động khởi động bên trong file JAR đó mà không cần cài đặt riêng.

```java
// Đây là toàn bộ code bạn cần để có một Web Server chạy trên cổng 8080!
// Spring Boot đã nhúng Tomcat sẵn rồi.
@SpringBootApplication
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
        // Tomcat tự động khởi động tại http://localhost:8080
    }
}
```

**Cấu hình Tomcat trong `application.yml`:**
```yaml
server:
  port: 8080                    # Đổi cổng
  tomcat:
    max-threads: 200            # Số luồng xử lý tối đa (Thread Pool)
    max-connections: 10000      # Số kết nối TCP tối đa
    connection-timeout: 20000   # Timeout kết nối (ms)
    accept-count: 100           # Hàng chờ khi thread pool đầy
```

#### ⚡ Jetty — Nhẹ nhàng, tốc độ cao

**Eclipse Jetty** là Servlet Container siêu nhẹ, nổi tiếng với khả năng xử lý kết nối **HTTP/2** và **WebSocket** cực tốt. Nếu dự án của bạn cần real-time (chat, live notification), Jetty là lựa chọn tuyệt vời.

**Thay Tomcat bằng Jetty trong Spring Boot (Maven):**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- Loại bỏ Tomcat mặc định -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- Thay bằng Jetty -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

> 💡 **Chỉ cần thay dependency trong POM, không cần sửa một dòng code Java nào!** Đây chính là sức mạnh của Spring Boot abstraction.

#### 🐺 WildFly (tên cũ: JBoss) — Full Application Server mã nguồn mở

**WildFly** (trước đây gọi là **JBoss Application Server**) là Full Application Server miễn phí của Red Hat. Nó hỗ trợ toàn bộ Jakarta EE (trước đây là Java EE), bao gồm:
- **EJB (Enterprise JavaBeans):** Quản lý nghiệp vụ phức tạp (Transaction, Security, Concurrency).
- **JMS (Java Message Service):** Hệ thống hàng đợi tin nhắn (Message Queue) tích hợp sẵn.
- **JTA (Java Transaction API):** Quản lý giao dịch phân tán (Distributed Transaction) giữa nhiều Database.
- **CDI (Contexts and Dependency Injection):** Dependency Injection chuẩn Jakarta EE.

**Khi nào dùng WildFly?**
- Dự án Enterprise yêu cầu tuân thủ đầy đủ chuẩn Jakarta EE.
- Cần EJB hoặc JMS tích hợp sẵn mà không muốn cài thêm Kafka hay RabbitMQ.
- Công ty dùng hệ sinh thái Red Hat (RHEL, OpenShift).

**Triển khai ứng dụng lên WildFly:**
```bash
# 1. Đóng gói thành file WAR (Web Archive)
mvn clean package -Pwar

# 2. Copy file WAR vào thư mục triển khai của WildFly
cp target/order-service.war $WILDFLY_HOME/standalone/deployments/

# 3. WildFly tự động phát hiện file WAR mới và triển khai (Hot Deploy)
```

#### 🏢 Oracle WebLogic — Ông lớn của Enterprise

**Oracle WebLogic Server** là Full Application Server thương mại hàng đầu, được các tập đoàn tài chính, ngân hàng, bảo hiểm tin dùng. Nó có giấy phép rất đắt đỏ nhưng đổi lại cung cấp:
- **Clustering:** Chạy nhiều instance trên nhiều Server, tự động cân bằng tải (Load Balancing) và chuyển đổi dự phòng (Failover).
- **Admin Console:** Giao diện Web quản trị cực kỳ mạnh mẽ để quản lý toàn bộ ứng dụng, datasource, JMS, security.
- **Hỗ trợ kỹ thuật Oracle 24/7:** Có đội ngũ Support chuyên nghiệp xử lý sự cố.

**Khi nào dùng WebLogic?**
- Các hệ thống ngân hàng Core Banking, bảo hiểm, viễn thông yêu cầu SLA 99.99%.
- Công ty đã mua license Oracle (Database, Middleware, Cloud).
- Cần Distributed Transaction giữa nhiều Database Oracle.

#### 🔵 IBM WebSphere — Đối thủ của WebLogic

**IBM WebSphere Application Server** là Full Application Server của IBM, cạnh tranh trực tiếp với Oracle WebLogic. Phổ biến trong các tập đoàn dùng hệ sinh thái IBM (IBM Cloud, DB2, MQ Series).

**WebSphere Liberty:** Phiên bản nhẹ hơn, hiện đại hơn của WebSphere truyền thống. Hỗ trợ MicroProfile (chuẩn Microservices cho Jakarta EE) và có thể khởi động trong vài giây (tương tự Spring Boot).

**Khi nào dùng WebSphere?**
- Tập đoàn đã đầu tư sâu vào hệ sinh thái IBM.
- Cần tích hợp với IBM MQ (Message Queue), DB2, hoặc IBM Cloud.

---

## 2. Embedded Server vs External Server — Trận chiến thế hệ

Đây là khái niệm **quan trọng bậc nhất** mà bạn phải hiểu:

| Tiêu chí | Embedded Server (Nhúng) | External Server (Cài riêng) |
|-----------|--------------------------|-------------------------------|
| **Cách hoạt động** | Server nằm **bên trong** file JAR. Chạy `java -jar app.jar` là xong. | Server cài riêng trên OS. Bạn deploy file `.war` vào thư mục của Server. |
| **Đại diện** | Spring Boot + Tomcat/Jetty/Undertow | WildFly, WebLogic, WebSphere |
| **Deploy** | `java -jar app.jar` | Copy `.war` vào `deployments/` |
| **Docker** | ✅ Siêu tiện (1 Dockerfile là xong) | ❌ Phức tạp (Phải cài Server trong Docker) |
| **Microservices** | ✅ Sinh ra để làm Microservices | ❌ Quá nặng nề cho Microservices |
| **Xu hướng** | 🚀 Đang thắng thế tuyệt đối | 📉 Đang dần thu hẹp (chỉ còn trong Legacy) |

**Dockerfile triển khai Spring Boot (Embedded Tomcat):**
```dockerfile
FROM eclipse-temurin:21-jre-alpine
COPY target/order-service.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
# Chỉ 4 dòng là có 1 container chạy được!
```

---

## 3. Bảng so sánh tổng hợp các Server

| Server | Loại | License | Jakarta EE | Embedded | Phổ biến |
|--------|------|---------|------------|----------|----------|
| **Tomcat** | Servlet Container | Miễn phí (Apache) | Chỉ Servlet + JSP | ✅ (Spring Boot mặc định) | ⭐⭐⭐⭐⭐ |
| **Jetty** | Servlet Container | Miễn phí (Eclipse) | Chỉ Servlet + JSP | ✅ | ⭐⭐⭐ |
| **Undertow** | Servlet Container | Miễn phí (Red Hat) | Chỉ Servlet + JSP | ✅ | ⭐⭐ |
| **WildFly** | Full App Server | Miễn phí (Red Hat) | ✅ Đầy đủ | ❌ | ⭐⭐⭐ |
| **WebLogic** | Full App Server | 💰 Thương mại (Oracle) | ✅ Đầy đủ | ❌ | ⭐⭐ (Enterprise) |
| **WebSphere** | Full App Server | 💰 Thương mại (IBM) | ✅ Đầy đủ | ❌ (Liberty có thể nhúng) | ⭐⭐ (Enterprise) |

---

## 4. Tổng hợp các chủ đề Servers (Bản đồ tra cứu)

### 🌟 4.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc)

1. **Servlet Container vs Application Server:** Phân biệt rõ 2 loại này. 90% dự án Spring Boot chỉ cần Servlet Container (Tomcat).
2. **Embedded Tomcat (Spring Boot):** Hiểu cách Spring Boot nhúng Tomcat vào trong file JAR. Cấu hình `server.port`, `server.tomcat.max-threads`.
3. **WAR vs JAR Deployment:** JAR = Embedded Server (Chạy `java -jar`). WAR = External Server (Deploy vào WildFly/WebLogic). Spring Boot mặc định dùng JAR.
4. **Thread Pool & Connection Pool:** Hiểu `max-threads` (số luồng xử lý request đồng thời) và `max-connections` (số kết nối TCP tối đa). Đây là 2 thông số quyết định Server chịu được bao nhiêu người dùng cùng lúc.
5. **Servlet API (Jakarta Servlet):** Hiểu Servlet là gì — nó là interface chuẩn mà mọi Java Web Framework (Spring MVC, JSF, Struts) đều phải implement. Tomcat/Jetty chính là "máy chạy" Servlet.

### 🟡 4.2. Nhóm Trung cấp

1. **Thay đổi Embedded Server:** Biết cách thay Tomcat bằng Jetty hoặc Undertow trong Spring Boot bằng cách exclude/include dependency.
2. **SSL/TLS (HTTPS):** Cấu hình Server để phục vụ HTTPS bằng certificate. Trong Spring Boot: `server.ssl.key-store`, `server.ssl.key-store-password`.
3. **Access Log & Request Logging:** Bật ghi log mọi request HTTP đến Server để phục vụ audit và debug.
4. **Graceful Shutdown:** Cấu hình `server.shutdown=graceful` để khi tắt Server, nó đợi tất cả request đang xử lý hoàn tất rồi mới tắt (thay vì cắt ngang giữa chừng).

### 🔴 4.3. Nhóm Nâng cao

1. **WildFly / WebLogic Administration:** Quản trị Full Application Server — tạo DataSource, cấu hình JMS Queue, Security Realm. Chỉ cần học khi làm việc với hệ thống Legacy Enterprise.
2. **Tomcat Tuning & Performance:** Tối ưu NIO Connector, AJP Connector, cấu hình `maxKeepAliveRequests`, `keepAliveTimeout` cho hệ thống chịu tải cao.
3. **Jakarta EE vs Spring:** Hiểu sâu cuộc chiến giữa 2 hệ sinh thái. Spring đã thắng thế trên thị trường, nhưng Jakarta EE (MicroProfile) vẫn là chuẩn chính thống được các Full Application Server tuân theo.
4. **Virtual Threads (Project Loom) + Tomcat:** Java 21 mang đến Virtual Threads. Tomcat và Jetty đã hỗ trợ, cho phép xử lý **hàng triệu request đồng thời** mà không cần Thread Pool truyền thống. Đây là tương lai của Java Server.

---

## 📚 Tài liệu tham khảo chính thống

| Tài liệu | Link |
|-----------|------|
| Apache Tomcat 11 Documentation | [tomcat.apache.org](https://tomcat.apache.org/tomcat-11.0-doc/index.html) |
| Eclipse Jetty Documentation | [jetty.org](https://eclipse.dev/jetty/documentation/jetty-12/index.html) |
| WildFly Documentation | [docs.wildfly.org](https://docs.wildfly.org/) |
| Spring Boot 4.1.0 — Embedded Web Servers | [docs.spring.io](https://docs.spring.io/spring-boot/reference/web/servlet.html#web.servlet.embedded-container) |
| Jakarta EE 11 Specification | [jakarta.ee](https://jakarta.ee/specifications/) |
| Oracle WebLogic Documentation | [docs.oracle.com](https://docs.oracle.com/en/middleware/standalone/weblogic-server/) |
| IBM WebSphere Liberty | [openliberty.io](https://openliberty.io/docs/latest/overview.html) |
