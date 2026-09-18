# 🔨 Nghệ thuật Xây dựng Dự án (Build Tools: Maven, Gradle, Ant)

> **Thuộc roadmap:** `1. Java → Build Tools (Maven, Gradle, Ant)`

---

## 1. Build Tool là gì? Tại sao không thể thiếu?

Khi mới học Java, bạn chỉ cần mở IntelliJ, bấm nút Run là chạy. Nhưng thực tế đi làm, một dự án Backend có thể phụ thuộc vào **hàng trăm thư viện bên ngoài** (Spring Boot, Hibernate, Jackson, Logback, Lombok...). Mỗi thư viện lại phụ thuộc vào hàng chục thư viện khác nữa (gọi là **Transitive Dependencies** — phụ thuộc bắc cầu). 

Nếu bạn phải tải từng file `.jar` về bằng tay, copy vào thư mục `lib/`, rồi cấu hình classpath thủ công — thì chỉ riêng việc chuẩn bị thư viện đã mất cả tuần, chưa kể xung đột phiên bản giữa các thư viện sẽ khiến bạn phát điên.

**Build Tool** giải quyết TẤT CẢ những vấn đề trên:
1. **Quản lý thư viện tự động (Dependency Management):** Bạn chỉ cần khai báo tên + phiên bản thư viện, Build Tool tự động tải về từ kho trung tâm (Maven Central) và giải quyết mọi xung đột.
2. **Biên dịch mã nguồn (Compile):** Dịch toàn bộ `.java` thành `.class` theo đúng thứ tự phụ thuộc.
3. **Chạy kiểm thử tự động (Test):** Tự tìm và chạy tất cả các file test (JUnit, Mockito...) sau mỗi lần build.
4. **Đóng gói sản phẩm (Package):** Nén toàn bộ code + thư viện thành 1 file `.jar` hoặc `.war` duy nhất để triển khai lên Server.
5. **Vòng đời chuẩn hóa (Build Lifecycle):** Mỗi dự án đều chạy theo một quy trình chuẩn: `compile → test → package → install → deploy`. Bất kỳ ai ở bất kỳ máy nào cũng build giống nhau.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn vừa gia nhập công ty và được giao dự án Spring Boot với 150 thư viện phụ thuộc. Bạn cần clone code từ Git về máy, build thành file `.jar` và chạy lên Server trong vòng 15 phút.*

### Giải pháp 1: Dùng Maven (Phổ biến nhất)

Maven sử dụng file `pom.xml` (Project Object Model) để khai báo toàn bộ thư viện và cấu hình build.

**Bước 1: Khai báo thư viện trong `pom.xml`**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- Thông tin định danh dự án (GAV — Group, Artifact, Version) -->
    <groupId>com.company</groupId>     <!-- Tên tổ chức (thường dùng tên miền ngược) -->
    <artifactId>order-service</artifactId> <!-- Tên dự án -->
    <version>1.0.0</version>               <!-- Phiên bản -->
    <packaging>jar</packaging>             <!-- Kiểu đóng gói: jar hoặc war -->

    <!-- Kế thừa Spring Boot Parent (Tự quản lý phiên bản cho hàng trăm thư viện) -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.0</version>
    </parent>

    <!-- Java Version -->
    <properties>
        <java.version>21</java.version>
    </properties>

    <!-- Danh sách thư viện phụ thuộc -->
    <dependencies>
        <!-- Spring Boot Web: Khai báo 1 dòng, Maven tự tải hàng chục thư viện liên quan -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- Không cần khai version! Parent đã quản lý sẵn rồi -->
        </dependency>

        <!-- Thư viện Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope> <!-- Chỉ dùng khi chạy Test, KHÔNG đóng gói vào sản phẩm -->
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Plugin đóng gói thành JAR chạy được (Executable JAR) -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**Bước 2: Chạy các lệnh Maven phổ biến**
```bash
# Biên dịch mã nguồn
mvn compile

# Chạy toàn bộ test
mvn test

# Đóng gói thành file JAR (Bao gồm compile + test + package)
mvn package

# Cài vào kho Maven cục bộ (~/.m2/repository) để các dự án khác dùng chung
mvn install

# Xóa toàn bộ thư mục target/ (Dọn rác build cũ)
mvn clean

# Combo hay dùng nhất: Dọn rác → Biên dịch → Test → Đóng gói, bỏ qua test
mvn clean package -DskipTests
```

### Giải pháp 2: Dùng Gradle (Hiện đại, nhanh hơn Maven)

Gradle sử dụng file `build.gradle` (viết bằng Groovy) hoặc `build.gradle.kts` (viết bằng Kotlin DSL). Cú pháp gọn gàng hơn Maven rất nhiều.

**File `build.gradle.kts` (Kotlin DSL — Xu hướng mới nhất)**
```kotlin
plugins {
    java
    id("org.springframework.boot") version "4.1.0"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "com.company"
version = "1.0.0"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral() // Kho thư viện trung tâm trên Internet
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<Test> {
    useJUnitPlatform() // Dùng JUnit 5
}
```

**Các lệnh Gradle phổ biến:**
```bash
# Biên dịch
./gradlew compileJava

# Chạy test
./gradlew test

# Đóng gói
./gradlew build

# Dọn rác
./gradlew clean

# Combo hay dùng nhất
./gradlew clean build -x test
```

### So sánh nhanh: Maven vs Gradle vs Ant

| Tiêu chí | Maven | Gradle | Ant |
|-----------|-------|--------|-----|
| **File cấu hình** | `pom.xml` (XML) | `build.gradle` (Groovy/Kotlin) | `build.xml` (XML) |
| **Quản lý thư viện** | ✅ Tích hợp sẵn (Maven Central) | ✅ Tích hợp sẵn (Maven Central + Ivy) | ❌ Phải dùng thêm **Ivy** |
| **Tốc độ build** | 🐢 Trung bình (Không có cache) | 🚀 Rất nhanh (Incremental Build + Build Cache) | 🐢 Chậm (Không có tối ưu) |
| **Độ phổ biến** | 🥇 #1 (Đa số dự án Enterprise) | 🥈 #2 (Android, dự án mới) | ⚰️ Gần như tuyệt chủng |
| **Độ linh hoạt** | Thấp (Convention over Configuration) | Rất cao (Viết code logic tùy ý) | Rất cao (Nhưng quá phức tạp) |
| **Đường cong học tập** | Trung bình | Cao (Cần biết Groovy/Kotlin) | Thấp (Nhưng dài dòng) |

### Ant — Ông tổ nhưng đã lỗi thời

Apache Ant (Another Neat Tool) là Build Tool đầu tiên của Java, ra đời năm 2000. Nó sử dụng file `build.xml` và yêu cầu bạn phải viết tay **từng bước một** (imperative): tải thư viện ở đâu, copy vào đâu, dịch file nào trước, file nào sau...

```xml
<!-- build.xml (Ví dụ minh họa — ĐỌC HIỂU LÀ ĐỦ, KHÔNG CẦN HỌC SÂU) -->
<project name="MyApp" default="jar" basedir=".">
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="build"/>

    <target name="clean">
        <delete dir="${build.dir}"/>
    </target>

    <target name="compile" depends="clean">
        <mkdir dir="${build.dir}/classes"/>
        <javac srcdir="${src.dir}" destdir="${build.dir}/classes"/>
    </target>

    <target name="jar" depends="compile">
        <jar destfile="${build.dir}/MyApp.jar" basedir="${build.dir}/classes"/>
    </target>
</project>
```

> ⚠️ **Ant không có hệ thống quản lý thư viện.** Bạn phải tự tải `.jar` về và copy thủ công. Đây là lý do chính khiến Ant bị Maven thay thế hoàn toàn từ năm 2004.

---

## 3. Tổng hợp các chủ đề Build Tools (Bản đồ tra cứu)

### 🌟 3.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc)

1. **Maven POM (Project Object Model):** Hiểu cấu trúc file `pom.xml` — bao gồm `<parent>`, `<dependencies>`, `<plugins>`, `<properties>`.
2. **GAV (GroupId, ArtifactId, Version):** Bộ ba định danh duy nhất của mỗi thư viện. Không có nó, Maven không biết tải cái gì.
3. **Maven Lifecycle:** Thuộc lòng thứ tự `validate → compile → test → package → verify → install → deploy`. Mỗi phase gọi tự động tất cả các phase trước nó.
4. **Dependency Scope:** Hiểu `compile` (mặc định, dùng mọi lúc), `test` (chỉ khi chạy test), `provided` (Server đã có sẵn, ví dụ Servlet API trên Tomcat), `runtime` (không cần khi compile nhưng cần khi chạy, ví dụ JDBC driver).
5. **Maven Wrapper (`mvnw`):** File script đi kèm dự án, đảm bảo mọi người dùng cùng 1 phiên bản Maven. Chạy `./mvnw clean package` thay vì `mvn clean package`.

### 🟡 3.2. Nhóm Trung cấp

1. **Multi-Module Project:** Kỹ thuật chia 1 dự án lớn thành nhiều module con (ví dụ: `common`, `api`, `service`, `repository`). File `pom.xml` cha sẽ khai báo `<modules>` để quản lý tập trung.
2. **Maven Profiles:** Cấu hình nhiều môi trường (`dev`, `staging`, `production`) trong cùng 1 file POM. Chạy `mvn package -Pproduction` để build cho Production.
3. **Gradle Kotlin DSL:** Hiểu cú pháp `build.gradle.kts` — xu hướng mới thay thế Groovy DSL vì có auto-complete và type-safe.
4. **Gradle Wrapper (`gradlew`):** Tương tự Maven Wrapper. Mọi dự án Gradle đều nên có file `gradlew` đi kèm.

### 🔴 3.3. Nhóm Nâng cao

1. **Gradle Build Cache & Incremental Build:** Cơ chế khiến Gradle nhanh hơn Maven gấp 2-10 lần. Gradle chỉ biên dịch lại **những file đã thay đổi**, còn Maven biên dịch lại **tất cả** mỗi lần build.
2. **Maven BOM (Bill of Materials):** File POM đặc biệt chỉ chứa danh sách phiên bản thư viện, dùng để đồng bộ version giữa nhiều dự án microservice. Spring Boot Parent thực chất là 1 BOM khổng lồ.
3. **Custom Maven Plugin:** Viết Plugin riêng để tự động hóa các tác vụ đặc biệt (ví dụ: tự động generate code từ file `.proto` của gRPC).
4. **Gradle Convention Plugin:** Tạo plugin nội bộ để chuẩn hóa cách build cho toàn bộ team (Java version, code style, test coverage...).

---

## 📚 Tài liệu tham khảo chính thống

| Tài liệu | Link |
|-----------|------|
| Apache Maven Official Guide | [maven.apache.org](https://maven.apache.org/guides/getting-started/index.html) |
| Gradle User Manual | [docs.gradle.org](https://docs.gradle.org/current/userguide/userguide.html) |
| Spring Boot 4.1.0 — Build Tool Plugins | [docs.spring.io](https://docs.spring.io/spring-boot/reference/build-tool-plugins/index.html) |
| Maven Central Repository | [search.maven.org](https://search.maven.org/) |
| Baeldung — Maven Tutorial | [baeldung.com](https://www.baeldung.com/maven) |
| Baeldung — Gradle Tutorial | [baeldung.com](https://www.baeldung.com/gradle) |
