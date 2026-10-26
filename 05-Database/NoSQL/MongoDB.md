# 🍃 Database NoSQL: MongoDB — Hệ quản trị CSDL Document-Oriented phổ biến nhất thế giới

> **Nguồn chính thống:** [MongoDB 8.0 Manual](https://www.mongodb.com/docs/manual/) | [Spring Data MongoDB Reference](https://docs.spring.io/spring-data/mongodb/reference/index.html)
>
> **Thuộc roadmap:** `5. Database → No SQL → Mongo Db`

---

## 1. Giới thiệu MongoDB

> *"MongoDB is a document database with the scalability and flexibility that you want with the querying and indexing that you need."*
>
> — **mongodb.com**

### Dịch & giải thích đơn giản:

**MongoDB** là hệ quản trị cơ sở dữ liệu **NoSQL hướng tài liệu (Document-Oriented)** mã nguồn mở phổ biến nhất hiện nay. Thay vì lưu dữ liệu vào các hàng (rows) và cột (columns) cứng nhắc như MySQL hay PostgreSQL, MongoDB lưu dữ liệu dưới dạng các **Documents** linh hoạt (định dạng **BSON** — Binary JSON).

Các document tương tự nhau được gom thành một **Collection** (tương đương với Table trong RDBMS).

```
RDBMS (MySQL/PostgreSQL)          MongoDB (NoSQL)
─────────────────────────────────────────────────
Database                 ───►    Database
Table                    ───►    Collection
Row (Bản ghi)            ───►    Document (BSON)
Column (Cột)             ───►    Field (Trường)
Index (Chỉ mục)          ───►    Index
JOIN                     ───►    Embedding (Lồng nhau) hoặc $lookup
Primary Key (id)         ───►    Primary Key (_id / ObjectId)
```

**Tại sao MongoDB được sử dụng rộng rãi?**
- **Schema-less (Linh hoạt):** Các document trong cùng collection không bắt buộc phải có cùng số lượng hoặc kiểu trường dữ liệu, rất thuận tiện khi phát triển tính năng mới nhanh chóng (Agile/Iterative).
- **Mô hình hướng đối tượng tự nhiên:** Document BSON ánh xạ 1-1 trực tiếp với các Object/Class trong Java mà không cần mapping phức tạp qua tầng trung gian.
- **Tốc độ đọc/ghi cực nhanh:** Tối ưu hóa bộ nhớ với WiredTiger Cache và ghi đĩa tuần tự.
- **Dễ dàng mở rộng ngang (Horizontal Scalability):** Tích hợp sẵn cơ chế Replica Sets (sẵn sàng cao) và Sharding (phân mảnh dữ liệu tự động ra nhiều server).

> [!NOTE]
> **JSON vs BSON:** JSON là text thuần túy. BSON (Binary JSON) là định dạng nhị phân mở rộng của JSON, bổ sung thêm các kiểu dữ liệu phong phú như `Date`, `Long (64-bit int)`, `Decimal128` (dùng cho tiền tệ), và `ObjectId`, đồng thời tối ưu tốc độ phân tích cú pháp (parsing) của máy tính.

---

## 2. Kiến trúc MongoDB

```mermaid
flowchart TB
    subgraph Client["Client / Application Layer"]
        A[Spring Boot 4.1.0 App]
        B[MongoDB Compass GUI]
        C[mongosh CLI]
    end

    subgraph Cluster["MongoDB High-Availability Cluster"]
        subgraph ReplicaSet["Replica Set (Automatic Failover)"]
            P["Primary Node ⭐<br>(Nhận mọi lệnh GHI & ĐỌC mặc định)"]
            S1["Secondary Node 1<br>(Đồng bộ qua Oplog, Đọc dữ liệu phụ)"]
            S2["Secondary Node 2<br>(Đồng bộ qua Oplog, Dự phòng)"]
            Arb["Arbiter Node<br>(Tham gia bầu chọn Primary, không lưu data)"]
        end

        subgraph Sharding["Sharding Layer (Horizontal Scale)"]
            Router["Mongos Router"]
            ConfigServer["Config Servers (Metadata)"]
        end
    end

    subgraph StorageEngine["WiredTiger Storage Engine"]
        WT_Cache["WiredTiger In-Memory Cache"]
        Journal["Journal WAL (Ghi nhật ký đĩa phục hồi)"]
        DataFiles["Data Files (.wt on Disk)"]
    end

    A --> Router
    B --> Router
    C --> Router
    Router --> ConfigServer
    Router --> P
    P -. "Replicate (Oplog)" .-> S1
    P -. "Replicate (Oplog)" .-> S2
    P --- Arb
    P --> WT_Cache
    WT_Cache --> Journal
    WT_Cache --> DataFiles

    style P fill:#22c55e,color:white
    style WT_Cache fill:#0284c7,color:white
```

### Các thành phần cốt lõi:
1. **WiredTiger Storage Engine:** Engine lưu trữ mặc định từ bản MongoDB 3.2+. Sử dụng cơ chế nén dữ liệu (Snappy/zlib), quản lý bộ nhớ đệm (Cache) và hỗ trợ Concurrency Control mức Document (tương đương row-level lock).
2. **Journal (Write-Ahead Log):** Đảm bảo tính bền vững (Durability). Dữ liệu được ghi vào Journal trước khi flush ra đĩa cứng định kỳ (mặc định mỗi 60 giây checkpoint 1 lần).
3. **Replica Set:** Một nhóm gồm tối thiểu 3 node (Primary, Secondary, Secondary/Arbiter). Nếu Primary sập, thuật toán bầu chọn (Raft-like consensus) sẽ tự động đưa 1 Secondary lên làm Primary trong vài giây mà không làm gián đoạn ứng dụng.

---

## 3. Cài đặt & Cấu hình MongoDB

### 3.1 Cài đặt với Docker (Standalone cho Development)

```bash
# Tạo và chạy MongoDB 8.0 container kèm volume
docker run -d \
  --name mongodb-dev \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=admin123 \
  -e MONGO_INITDB_DATABASE=ecommerce_db \
  -p 27017:27017 \
  -v mongo_data:/data/db \
  mongo:8.0

# Truy cập vào MongoDB Shell (mongosh)
docker exec -it mongodb-dev mongosh -u admin -p admin123 --authenticationDatabase admin
```

### 3.2 Docker Compose chuẩn Replica Set (Môi trường thực tế)

```yaml
# docker-compose.yml
services:
  mongodb:
    image: mongo:8.0
    container_name: mongodb-server
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: app_admin
      MONGO_INITDB_ROOT_PASSWORD: securePassword123
      MONGO_INITDB_DATABASE: ecommerce_db
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
      - ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
    command: ["--wiredTigerCacheSizeGB", "1.5"]
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  mongo_data:
```

---

## 4. Kiểu dữ liệu BSON & Thiết kế dữ liệu (Schema Design)

### 4.1 Bảng ánh xạ kiểu dữ liệu BSON sang Java

| BSON Type | Ý nghĩa / Kích thước | Java Type | Ghi chú Use Case |
|-----------|----------------------|-----------|------------------|
| `ObjectId` | 12 bytes nhị phân | `org.bson.types.ObjectId` / `String` | Khóa chính `_id` tự sinh (Timestamp + Random) ⭐ |
| `String` | Chuỗi UTF-8 | `java.lang.String` | Tên, email, mã sản phẩm |
| `Int32` | Số nguyên 4 bytes | `java.lang.Integer` / `int` | Số lượng tồn kho, thứ tự |
| `Int64` | Số nguyên 8 bytes | `java.lang.Long` / `long` | Timestamp epoch millis, số tiền lớn |
| `Double` | Số thực 8 bytes | `java.lang.Double` / `double` | Tọa độ GPS, điểm đánh giá sao (rating) |
| `Decimal128` | Số thập phân 128-bit | `java.math.BigDecimal` | **Tiền tệ, số dư tài khoản** (Tránh làm tròn) ⭐ |
| `Boolean` | Đúng / Sai | `java.lang.Boolean` / `boolean` | Trạng thái kích hoạt (`isActive`) |
| `Date` | 8 bytes UTC datetime | `java.time.Instant` / `LocalDateTime` | Thời điểm tạo/cập nhật bản ghi |
| `Array` | Danh sách các phần tử | `java.util.List<T>` / `Set<T>` | Tags, biến thể sản phẩm, danh sách quyền |
| `Object (Embedded)` | Sub-document lồng nhau | Class Java tùy biến | Địa chỉ, cấu hình thuộc tính |

---

### 4.2 Nghệ thuật thiết kế Schema: Embedding vs Referencing

Trong NoSQL, quy tắc thiết kế không tuân theo 3 dạng chuẩn (Normal Forms) của RDBMS mà phụ thuộc hoàn toàn vào **tần suất truy vấn**:

```mermaid
graph TD
    subgraph Embedding["1. Embedding (Lồng đối tượng)"]
        Doc["Document User"]
        Addr["Sub-document: Address"]
        Doc --> Addr
    end

    subgraph Referencing["2. Referencing (Tham chiếu qua ID)"]
        DocO["Document Order"]
        DocU["Document User"]
        DocO -. "userId (Foreign Key)" .-> DocU
    end
```

| Chiến lược | Khi nào nên dùng? | Ví dụ thực tế |
|------------|-------------------|---------------|
| **Embedding (Denormalization)** | Quan hệ 1 - 1 hoặc 1 - ít (1-to-Few); Dữ liệu con luôn được đọc cùng dữ liệu cha; Dữ liệu con ít khi bị cập nhật độc lập. | User có 2-3 `Address`; Order có danh sách `OrderItems` (snapshot lúc mua). |
| **Referencing (Normalization)** | Quan hệ 1 - Nhiều (1-to-Many lớn); Dữ liệu con phát triển không giới hạn (Unbounded Growth); Dữ liệu được tái sử dụng ở nhiều nơi. | Một `Author` có hàng nghìn `Posts`; `Customer` liên kết với lịch sử hàng trăm `Orders`. |

> [!CAUTION]
> **Giới hạn 16MB:** Một Document BSON trong MongoDB có kích thước tối đa là **16 Megabytes**. Tuyệt đối không bao giờ dùng Embedding cho danh sách có thể tăng vô tận (ví dụ: Log của 1 thiết bị, bình luận của một video viral triệu view). Khi danh sách lớn, bắt buộc phải dùng **Referencing** hoặc **Bucket Pattern**.

---

## 5. Thao tác CRUD & Aggregation Pipeline trong MongoDB

### 5.1 Cú pháp MQL (MongoDB Query Language) cơ bản

```javascript
// 1. Chuyển sang database
use ecommerce_db;

// 2. Thêm mới bản ghi (INSERT)
db.products.insertOne({
    name: "MacBook Pro M4",
    brand: "Apple",
    category: "Laptops",
    price: NumberDecimal("42990000"),
    stock: 50,
    tags: ["laptop", "apple", "m4"],
    specifications: {
        cpu: "Apple M4 Pro 12-core",
        ram: "24GB Unified",
        storage: "512GB SSD"
    },
    isActive: true,
    createdAt: new Date()
});

// 3. Truy vấn dữ liệu (SELECT)
db.products.find({
    category: "Laptops",
    price: { $gte: NumberDecimal("30000000"), $lte: NumberDecimal("50000000") },
    tags: { $in: ["apple"] }
}).sort({ price: -1 }).limit(10);

// 4. Cập nhật dữ liệu (UPDATE - Sử dụng toán tử $set, $inc, $push)
db.products.updateOne(
    { name: "MacBook Pro M4" },
    {
        $inc: { stock: -1 },                        // Trừ tồn kho đi 1
        $set: { updatedAt: new Date() },             // Đặt lại thời gian
        $push: { tags: "bestseller" }                // Thêm tag mới vào array
    }
);

// 5. Xóa dữ liệu (DELETE)
db.products.deleteOne({ _id: ObjectId("665a1b2c3d4e5f6a7b8c9d0e") });
```

### 5.2 Aggregation Framework (Đỉnh cao phân tích dữ liệu)

Aggregation Pipeline xử lý dữ liệu qua nhiều tầng tuần tự (Stage), tương đương với `GROUP BY`, `JOIN`, `HAVING` trong SQL nhưng mạnh mẽ hơn nhiều:

```javascript
// Thống kê doanh thu và sản phẩm bán chạy nhất theo từng thương hiệu
db.orders.aggregate([
    // Stage 1: Lọc các đơn hàng đã thanh toán thành công
    { $match: { status: "PAID" } },

    // Stage 2: Tách mảng items ra từng dòng riêng biệt
    { $unwind: "$items" },

    // Stage 3: Nhóm theo thương hiệu sản phẩm và tính tổng
    {
        $group: {
            _id: "$items.brand",
            totalRevenue: { $sum: { $multiply: ["$items.price", "$items.quantity"] } },
            totalQuantitySold: { $sum: "$items.quantity" },
            orderCount: { $sum: 1 }
        }
    },

    // Stage 4: Lọc các thương hiệu có tổng doanh thu trên 100 triệu
    { $match: { totalRevenue: { $gte: 100000000 } } },

    // Stage 5: Sắp xếp doanh thu cao nhất lên đầu
    { $sort: { totalRevenue: -1 } }
]);
```

---

## 6. Tích hợp MongoDB với Spring Boot 4.1.0 & Spring Framework 7.0.8

### 6.1 Khai báo Dependencies (`pom.xml`)

```xml
<dependencies>
    <!-- Spring Data MongoDB Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

    <!-- Hỗ trợ Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
</dependencies>
```

### 6.2 Cấu hình kết nối (`application.yml`)

```yaml
spring:
  data:
    mongodb:
      # Connection URI chuẩn hỗ trợ Replica Set và Auth
      uri: mongodb://app_admin:securePassword123@localhost:27017/ecommerce_db?authSource=admin&replicaSet=rs0
      auto-index-creation: false # Production: TẮT auto-create index để kiểm soát schema

logging:
  level:
    org.springframework.data.mongodb.core.MongoTemplate: DEBUG # Log chi tiết câu MQL sinh ra
```

### 6.3 Định nghĩa Document & Sub-document

```java
package com.example.ecommerce.document;

import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.annotation.Version;
import org.springframework.data.mongodb.core.index.CompoundIndex;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;
import org.springframework.data.mongodb.core.mapping.FieldType;

import java.math.BigDecimal;
import java.time.Instant;
import java.util.List;
import java.util.Map;

@Document(collection = "products")
@CompoundIndex(name = "idx_category_brand_price", def = "{'category': 1, 'brand': 1, 'price': -1}")
public class Product {

    @Id
    private String id; // Tự động map với _id ObjectId của MongoDB

    @Indexed(unique = true)
    private String sku;

    private String name;
    private String category;
    private String brand;

    @Field(targetType = FieldType.DECIMAL128)
    private BigDecimal price;

    private Integer stock;

    private List<String> tags;

    // Sub-document lồng nhau linh hoạt
    private Map<String, Object> specifications;

    @Version // Optimistic Locking ngăn chặn Lost Update trong môi trường Concurrency
    private Long version;

    @CreatedDate
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;

    // constructors, getters, setters...
}
```

### 6.4 Triển khai Repository & Dynamic Criteria với `MongoTemplate`

```java
package com.example.ecommerce.repository;

import com.example.ecommerce.document.Product;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;

@Repository
public interface ProductRepository extends MongoRepository<Product, String> {

    Optional<Product> findBySku(String sku);

    // Derived Query Method
    List<Product> findByCategoryAndBrand(String category, String brand);

    // Custom JSON Query
    @Query("{ 'category': ?0, 'price': { $gte: ?1, $lte: ?2 } }")
    Page<Product> findByPriceRange(String category, BigDecimal minPrice, BigDecimal maxPrice, Pageable pageable);

    // Full-Text Search trong MongoDB
    @Query("{ $text: { $search: ?0 } }")
    List<Product> searchByText(String keyword);
}
```

```java
package com.example.ecommerce.service;

import com.example.ecommerce.document.Product;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ProductSearchService {

    private final MongoTemplate mongoTemplate;

    public ProductSearchService(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    // Dynamic Query Builder - Xây dựng bộ lọc tìm kiếm đa điều kiện
    public List<Product> filterProducts(String category, String brand, List<String> tags) {
        Query query = new Query();

        if (category != null && !category.isBlank()) {
            query.addCriteria(Criteria.where("category").is(category));
        }
        if (brand != null && !brand.isBlank()) {
            query.addCriteria(Criteria.where("brand").is(brand));
        }
        if (tags != null && !tags.isEmpty()) {
            query.addCriteria(Criteria.where("tags").all(tags));
        }

        return mongoTemplate.find(query, Product.class);
    }

    // Atomic Update - Cập nhật tồn kho an toàn mà không cần đọc nguyên document ra
    public boolean updateStockAtomic(String productId, int quantityToDeduct) {
        Query query = new Query(Criteria.where("id").is(productId)
                                        .and("stock").gte(quantityToDeduct));

        Update update = new Update()
                .inc("stock", -quantityToDeduct)
                .currentDate("updatedAt");

        var result = mongoTemplate.updateFirst(query, update, Product.class);
        return result.getModifiedCount() > 0;
    }
}
```

---

## 7. Index (Chỉ mục) & Tối ưu hóa hiệu năng

```mermaid
flowchart LR
    subgraph NoIndex["❌ Không có Index (COLLSCAN)"]
        A["db.products.find({sku: 'PRO-1001'})"]
        B["Collection Scan: Duyệt tuần tự 5.000.000 documents<br>⏱️ ~3.5 giây | Tốn 100% CPU"]
    end

    subgraph WithIndex["✅ Có Index trên sku (IXSCAN)"]
        C["db.products.find({sku: 'PRO-1001'})"]
        D["B-Tree Index Lookup: Nhảy thẳng đến Document bằng R-Tree / B-Tree<br>⏱️ ~2ms | Không tốn CPU"]
    end

    A --> B
    C --> D

    style B fill:#ef4444,color:white
    style D fill:#22c55e,color:white
```

### Quy tắc vàng thiết kế Compound Index: **ESR (Equality, Sort, Range)**
Khi câu query chứa nhiều điều kiện, thứ tự đặt trường trong Index quyết định hiệu năng:
1. **E (Equality):** Các trường so sánh bằng (`=` hoặc `$eq`) phải đặt đầu tiên.
2. **S (Sort):** Các trường dùng để sắp xếp thứ tự (`sort()`) đặt tiếp theo.
3. **R (Range):** Các trường so sánh khoảng (`$gt`, `$lt`, `$gte`, `$in`) đặt ở cuối cùng.

```javascript
// Ví dụ Query:
db.orders.find({ customerId: "C123", orderDate: { $gte: ISODate("2026-01-01") } }).sort({ totalAmount: -1 });

// Tạo Index chuẩn theo quy tắc ESR:
// 1. customerId (Equality) -> 2. totalAmount (Sort) -> 3. orderDate (Range)
db.orders.createIndex({ customerId: 1, totalAmount: -1, orderDate: 1 });
```

---

## 8. Transaction & Tính nhất quán (ACID trong MongoDB)

Từ MongoDB 4.0+, hệ thống hỗ trợ **Multi-Document ACID Transactions** trên Replica Set và Sharded Clusters:

```java
@Configuration
public class MongoTransactionConfig {
    @Bean
    MongoTransactionManager transactionManager(MongoDatabaseFactory dbFactory) {
        return new MongoTransactionManager(dbFactory);
    }
}
```

```java
@Service
public class CheckoutService {

    private final ProductSearchService productService;
    private final OrderRepository orderRepository;

    public CheckoutService(ProductSearchService productService, OrderRepository orderRepository) {
        this.productService = productService;
        this.orderRepository = orderRepository;
    }

    // Đảm bảo ACID giao dịch trên MongoDB
    @Transactional
    public Order placeOrder(Order order) {
        boolean stockDeducted = productService.updateStockAtomic(order.getProductId(), order.getQuantity());
        if (!stockDeducted) {
            throw new InsufficientStockException("Sản phẩm không đủ tồn kho!");
        }
        return orderRepository.save(order);
    }
}
```

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

### MongoDB được sử dụng ở đâu trong thực tế:

| Doanh nghiệp | Use Case thực tế | Giải quyết bài toán gì |
|---|---|---|
| **eBay / Amazon** | Product Catalog & Inventory | Mỗi mặt hàng có hàng trăm thuộc tính khác nhau; schema thay đổi liên tục mà không cần chạy `ALTER TABLE`. |
| **Forbes / The New York Times** | Content Management System (CMS) | Bài viết dạng block-content, đa ngôn ngữ, lưu trữ siêu dữ liệu phong phú, tốc độ hiển thị cực nhanh. |
| **Uber** | Telemetry & Dispatch Driver Profiles | Lưu thông tin động của tài xế, tọa độ di chuyển gần nhất với Geospatial Index (`2dsphere`). |
| **EA Sports** | Gaming Profiles & Leaderboards | Lưu trữ kho đồ nhân vật (Inventory items), chỉ số game thủ dạng lồng nhau phức tạp. |

### Khi nào nên chọn MongoDB vs Khi nào KHÔNG nên:

| Tình huống | Nên chọn MongoDB? | Lý do kỹ thuật |
|---|:---:|---|
| Dữ liệu có cấu trúc linh hoạt, đa hình (Polymorphic data) | ✅ | Document BSON tự do mở rộng, không cần migrate schema nặng nề. |
| Yêu cầu độ trễ cực thấp (Low Latency), đọc nhiều | ✅ | In-Memory WiredTiger Cache và Embedding triệt tiêu hoàn toàn chi phí JOIN bảng. |
| Hệ thống phân tán mở rộng ngang (Sharding) | ✅ | Auto-Sharding tích hợp sẵn trong nhân MongoDB. |
| Báo cáo tài chính kế toán phức tạp với 10+ bảng quan hệ chặt | ❌ | Quan hệ nhiều-nhiều phức tạp nên dùng RDBMS (PostgreSQL) để đảm bảo ràng buộc khóa ngoại (Foreign Keys). |
| Ứng dụng ghi chép dữ liệu khổng lồ hàng triệu event/giây liên tục | ⚠️ | Nên cân nhắc **Apache Cassandra** (Xem file tiếp theo). |

---

## 9. Tổng hợp các chủ đề (Bản đồ tra cứu)

### 🟢 Ưu tiên phải biết trước (Bắt buộc)
- Phân biệt RDBMS vs Document Database (Database, Collection, Document, Field).
- Định dạng BSON, `_id` và cấu trúc `ObjectId`.
- Cài đặt MongoDB với Docker.
- MQL CRUD cơ bản: `find()`, `insertOne()`, `updateOne()` với `$set`, `$inc`, `deleteOne()`.
- Tích hợp Spring Boot: `spring-boot-starter-data-mongodb`, `@Document`, `@Id`, `MongoRepository`.
- Chiến lược Embedding vs Referencing cơ bản.

### 🟡 Nhóm trung cấp
- Nghệ thuật thiết kế Index: Single, Compound Index và **quy tắc ESR**.
- Phân tích hiệu năng truy vấn với `explain("executionStats")` (Phân biệt COLLSCAN vs IXSCAN).
- Aggregation Pipeline cơ bản: `$match`, `$project`, `$group`, `$sort`, `$unwind`.
- Xây dựng Dynamic Query với `MongoTemplate` và `Criteria`.
- Chỉ mục chuyên biệt: TTL Index (xóa session tự động), Text Index.
- Quản lý giao dịch ACID với `MongoTransactionManager`.

### 🔴 Nhóm nâng cao
- Giải quyết giới hạn 16MB Document: Subset Pattern, Bucket Pattern, Outlier Pattern.
- Kiến trúc Replica Set: Bầu chọn Primary (Raft consensus), Heartbeat, Oplog replication.
- Thiết lập Read & Write Concerns (`w: "majority"`, `j: true`, `readPreference`).
- Kiến trúc Sharding: Mongos Router, Config Server, lựa chọn Shard Key chống Hotspotting.
- Lắng nghe sự kiện dữ liệu thời gian thực: **MongoDB Change Streams**.
- Tối ưu hóa bộ nhớ WiredTiger Storage Engine và giải phóng RAM.

---

## 📚 Tài liệu tham khảo chính thống

| Tài liệu | Đường dẫn liên kết |
|---|---|
| MongoDB 8.0 Official Documentation | [mongodb.com/docs/manual](https://www.mongodb.com/docs/manual/) |
| MongoDB University (Học miễn phí có chứng chỉ) | [learn.mongodb.com](https://learn.mongodb.com/) |
| Spring Data MongoDB Reference Guide | [docs.spring.io/spring-data/mongodb](https://docs.spring.io/spring-data/mongodb/reference/index.html) |
| MongoDB Schema Design Patterns (Building with Patterns) | [mongodb.com/blog/post/building-with-patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary) |
| MongoDB Docker Official Image | [hub.docker.com/_/mongo](https://hub.docker.com/_/mongo) |

---

> **Tiếp theo trong roadmap:** [Cassandra.md](file:///d:/WebSpace/Java-Backend/05-Database/NoSQL/Cassandra.md) — Hệ quản trị CSDL phân tán Wide-Column hiệu năng ghi cực đại (Write-heavy), thiết kế theo kiến trúc Peer-to-Peer masterless không có điểm chết đơn độc (Zero SPOF).
