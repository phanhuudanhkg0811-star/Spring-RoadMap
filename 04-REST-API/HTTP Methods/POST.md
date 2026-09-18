# 📤 HTTP Method: POST — REST API

> **Nguồn chính thống:** [RFC 9110 — HTTP Semantics: POST](https://www.rfc-editor.org/rfc/rfc9110.html#name-post) | [MDN Web Docs — POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)
>
> **Thuộc roadmap:** `4. REST API → HTTP Methods → POST`

---

## 1. Định nghĩa chính thức từ RFC 9110

> *"The **POST** method requests that the target resource **process the representation** enclosed in the request according to the resource's own specific semantics."*
>
> — **RFC 9110, Section 9.3.3**

RFC liệt kê các use case chính của POST:

> *"1. Providing a block of data... to a data-handling process;*
> *2. Posting a message to a bulletin board, newsgroup, mailing list...;*
> *3. **Creating a new resource** that has yet to be identified by the origin server; and*
> *4. Appending data to a resource's existing representation(s)."*

### Dịch & giải thích đơn giản:

**POST** là phương thức HTTP dùng để **gửi dữ liệu lên server** để xử lý. Trong REST API, POST **chủ yếu** được dùng để **tạo mới** tài nguyên (resource).

Đặc tính quan trọng:
- **NOT Safe:** POST **thay đổi** dữ liệu trên server (tạo bản ghi mới trong database).
- **NOT Idempotent:** Gọi POST 2 lần = tạo **2 bản ghi khác nhau** (không giống GET hay PUT).
- **NOT Cacheable:** Kết quả POST thường không được cache.
- **Có Request Body:** Dữ liệu gửi trong **body** của request (thường là JSON).

> [!IMPORTANT]
> **POST KHÔNG phải idempotent!** Đây là điểm khác biệt cốt lõi với PUT. Nếu client gửi POST 2 lần (ví dụ: do mạng lag, user click nút 2 lần), server sẽ tạo **2 bản ghi trùng lặp**. Đây là vấn đề thực tế mà bạn phải xử lý (xem phần Idempotency Key bên dưới).

---

## 2. POST trong REST API — Convention chuẩn

### URL Convention:

```
POST /api/products         ← Tạo 1 sản phẩm mới
POST /api/users            ← Tạo 1 user mới
POST /api/orders           ← Tạo 1 đơn hàng mới
POST /api/users/42/orders  ← Tạo đơn hàng cho user 42
```

### Request & Response chuẩn:

**Request:**
```http
POST /api/products HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

{
  "name": "iPhone 16 Pro",
  "price": 29990000,
  "category": "electronics",
  "description": "Smartphone cao cấp Apple",
  "stock": 100
}
```

**Response (201 Created):**
```http
HTTP/1.1 201 Created
Location: /api/products/42
Content-Type: application/json

{
  "id": 42,
  "name": "iPhone 16 Pro",
  "price": 29990000,
  "category": "electronics",
  "description": "Smartphone cao cấp Apple",
  "stock": 100,
  "createdAt": "2026-09-16T10:30:00"
}
```

> [!NOTE]
> **Hai điểm quan trọng trong response:**
> 1. Status code phải là **201 Created** (KHÔNG phải 200 OK)
> 2. Header **`Location`** chứa URL đến resource vừa tạo — đây là best practice theo REST convention

---

## 3. Implementation trong Spring Boot 4.1.0

### 3.1 Controller Layer

```java
@RestController
@RequestMapping("/api/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @PostMapping
    public ResponseEntity<ProductResponse> create(
            @Valid @RequestBody ProductCreateRequest request) {

        ProductResponse created = productService.create(request);

        // Tạo URI cho header Location: /api/products/42
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();

        return ResponseEntity
                .created(location)     // 201 Created + Location header
                .body(created);
    }
}
```

### 3.2 Request DTO với Validation

```java
@Data
public class ProductCreateRequest {

    @NotBlank(message = "Tên sản phẩm không được để trống")
    @Size(min = 2, max = 200, message = "Tên sản phẩm phải từ 2-200 ký tự")
    private String name;

    @NotNull(message = "Giá không được để trống")
    @DecimalMin(value = "0.01", message = "Giá phải lớn hơn 0")
    private BigDecimal price;

    @NotBlank(message = "Danh mục không được để trống")
    private String category;

    @Size(max = 1000, message = "Mô tả tối đa 1000 ký tự")
    private String description;

    @Min(value = 0, message = "Tồn kho không được âm")
    private int stock;
}
```

### 3.3 Service Layer

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {

    private final ProductRepository productRepository;
    private final ProductMapper productMapper;

    @Transactional
    public ProductResponse create(ProductCreateRequest request) {
        // 1. Kiểm tra business rule
        if (productRepository.existsByName(request.getName())) {
            throw new DuplicateResourceException("Product", "name", request.getName());
        }

        // 2. Chuyển DTO → Entity
        Product product = productMapper.toEntity(request);
        product.setCreatedAt(LocalDateTime.now());

        // 3. Lưu vào database
        Product saved = productRepository.save(product);
        log.info("Created product: id={}, name={}", saved.getId(), saved.getName());

        // 4. Chuyển Entity → Response DTO
        return productMapper.toResponse(saved);
    }
}
```

### 3.4 Global Exception Handling cho Validation Errors

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Xử lý lỗi validation (@Valid)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );

        ErrorResponse response = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Dữ liệu không hợp lệ",
            errors
        );

        return ResponseEntity.badRequest().body(response);
    }

    // Xử lý resource trùng lặp
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleDuplicate(
            DuplicateResourceException ex) {

        ErrorResponse response = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            ex.getMessage(),
            null
        );

        return ResponseEntity.status(HttpStatus.CONFLICT).body(response);
    }
}
```

**Response khi validation fail (400 Bad Request):**
```json
{
  "status": 400,
  "message": "Dữ liệu không hợp lệ",
  "errors": {
    "name": "Tên sản phẩm không được để trống",
    "price": "Giá phải lớn hơn 0"
  }
}
```

---

## 4. Response Status Codes cho POST

| Status Code | Khi nào dùng | Ví dụ |
|------------|-------------|-------|
| **201 Created** ⭐ | Tạo tài nguyên thành công | POST /api/products → sản phẩm mới được tạo |
| **200 OK** | Xử lý thành công nhưng không tạo resource mới | POST /api/auth/login → trả về token |
| **202 Accepted** | Request đã nhận, đang xử lý (async) | POST /api/reports → báo cáo sẽ được tạo sau |
| **400 Bad Request** | Dữ liệu đầu vào sai | Thiếu field bắt buộc, sai format |
| **401 Unauthorized** | Chưa đăng nhập | Thiếu/sai JWT token |
| **403 Forbidden** | Không có quyền tạo | User thường tạo admin resource |
| **409 Conflict** | Xung đột dữ liệu | Tạo user với email đã tồn tại |
| **422 Unprocessable Entity** | Dữ liệu hợp lệ về format nhưng sai logic | Đặt hàng sản phẩm hết tồn kho |

---

## 5. Xử lý vấn đề Duplicate POST (Idempotency Key)

### Vấn đề: User click nút "Đặt hàng" 2 lần → 2 đơn hàng!

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant DB

    Client->>Server: POST /api/orders (lần 1)
    Server->>DB: INSERT order → id=101
    Note right of Server: Mạng lag...
    Client->>Server: POST /api/orders (lần 2 — user click lại!)
    Server->>DB: INSERT order → id=102
    Note right of DB: 2 đơn hàng trùng lặp! ❌
```

### Giải pháp: Idempotency Key

```java
@PostMapping("/api/orders")
public ResponseEntity<OrderResponse> createOrder(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @Valid @RequestBody OrderCreateRequest request) {

    // Kiểm tra key đã được xử lý chưa
    Optional<OrderResponse> existing = orderService.findByIdempotencyKey(idempotencyKey);
    if (existing.isPresent()) {
        return ResponseEntity.ok(existing.get());  // Trả về kết quả cũ, KHÔNG tạo mới
    }

    OrderResponse order = orderService.create(request, idempotencyKey);
    return ResponseEntity.status(HttpStatus.CREATED).body(order);
}
```

```
// Client gửi header đặc biệt
POST /api/orders
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000   ← UUID duy nhất cho mỗi action
```

> [!TIP]
> **Idempotency Key** là pattern rất phổ biến trong payment API (VNPay, Stripe, PayPal). Mỗi giao dịch thanh toán có 1 key duy nhất — nếu client gửi lại, server sẽ trả kết quả cũ thay vì trừ tiền 2 lần.

---

## 6. POST cho các use case đặc biệt (không phải CRUD)

Ngoài việc tạo resource, POST còn được dùng cho các tác vụ **không thuộc CRUD chuẩn**:

```java
// 1. Authentication — Đăng nhập
@PostMapping("/api/auth/login")
public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest request) {
    // Trả 200 OK (không tạo resource mới)
}

// 2. Batch Operation — Xử lý hàng loạt
@PostMapping("/api/products/batch-import")
public ResponseEntity<BatchResult> batchImport(
        @RequestBody List<ProductCreateRequest> products) {
    // Tạo nhiều sản phẩm cùng lúc
}

// 3. Search phức tạp — Khi query string quá dài
@PostMapping("/api/products/search")
public ResponseEntity<Page<ProductResponse>> advancedSearch(
        @RequestBody ProductSearchCriteria criteria,
        Pageable pageable) {
    // POST search khi filter quá phức tạp cho GET
}

// 4. File Upload
@PostMapping("/api/files/upload")
public ResponseEntity<FileResponse> upload(
        @RequestParam("file") MultipartFile file) {
    // Upload file
}

// 5. Trigger Action — Kích hoạt hành động
@PostMapping("/api/orders/42/cancel")
public ResponseEntity<OrderResponse> cancelOrder(@PathVariable Long orderId) {
    // Hành động "cancel" không phải CRUD chuẩn
}
```

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

### POST trong ứng dụng E-commerce thực tế:

| Hành động | POST Request | Body chính |
|----------|-------------|-----------|
| Đăng ký tài khoản | `POST /api/auth/register` | `{username, email, password}` |
| Đăng nhập | `POST /api/auth/login` | `{email, password}` |
| Thêm vào giỏ hàng | `POST /api/cart/items` | `{productId, quantity}` |
| Đặt hàng | `POST /api/orders` | `{items, shippingAddress, paymentMethod}` |
| Thanh toán | `POST /api/payments` | `{orderId, amount, method}` |
| Viết đánh giá | `POST /api/products/42/reviews` | `{rating, comment}` |
| Upload ảnh đại diện | `POST /api/users/avatar` | `multipart/form-data` |

### Luồng đặt hàng hoàn chỉnh:

```mermaid
sequenceDiagram
    participant Client as Frontend
    participant API as Spring Boot API
    participant DB as Database
    participant Payment as Payment Service

    Client->>API: POST /api/orders<br/>{items, address, payment}
    API->>API: @Valid → Validate input
    API->>DB: Kiểm tra tồn kho
    DB-->>API: Còn hàng ✅
    API->>DB: INSERT order (status=PENDING)
    API->>Payment: POST /api/payments<br/>{orderId, amount}
    Payment-->>API: Payment success ✅
    API->>DB: UPDATE order (status=PAID)
    API-->>Client: 201 Created<br/>Location: /api/orders/101
```

---

## 7. Tóm tắt POST

```mermaid
mindmap
  root((POST))
    Đặc tính
      NOT Safe - Thay đổi dữ liệu
      NOT Idempotent ⭐ - Gọi 2 lần = 2 records
      NOT Cacheable
      CÓ Request Body - JSON
    Use cases
      Tạo mới resource ⭐ - CRUD
      Authentication - Login
      File Upload
      Batch Operations
      Complex Search
    Response chuẩn
      201 Created ⭐ + Location header
      200 OK - Non-CRUD actions
      400 Bad Request - Validation fail
      409 Conflict - Duplicate
    Xử lý duplicate
      Idempotency Key
      Unique constraints
      Client-side debounce
```

---

## 📚 Tài liệu tham khảo chính thống

| Tài liệu | Link |
|-----------|------|
| RFC 9110 — POST | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc9110.html#name-post) |
| MDN — POST | [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST) |
| Spring MVC — @PostMapping | [docs.spring.io](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-requestmapping.html) |
| Spring Boot — Validation | [docs.spring.io](https://docs.spring.io/spring-boot/reference/io/validation.html) |

---

> **Tiếp theo trong roadmap:** `PUT` (Phương thức cập nhật toàn bộ tài nguyên — so sánh với PATCH, idempotent, và implementation chuẩn).
