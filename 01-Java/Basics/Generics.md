# 🔷 Generics (Kiểu tổng quát)

> **Thuộc roadmap:** `1. Java → Basics → Generics`

---

## 1. Generics là gì?

Trước Java 5, Collections không có cơ chế kiểm tra kiểu dữ liệu. Bạn có thể bỏ bất cứ thứ gì vào một `ArrayList` — cả `String`, `Integer`, `NhanVien` lẫn lộn — rồi khi lấy ra phải ép kiểu thủ công (`casting`). Điều này dẫn đến lỗi `ClassCastException` nổ tung lúc runtime mà không ai phát hiện lúc compile.

**Generics** ra đời để giải quyết vấn đề này: cho phép bạn **khai báo kiểu dữ liệu từ trước**, để **trình biên dịch (Compiler) kiểm tra kiểu ngay lúc viết code**, không cần đợi tới lúc chạy mới nổ lỗi.

### Trước và sau khi có Generics:

```java
// ❌ TRƯỚC (Java 4 trở về trước) - Không có Generics
List danhSach = new ArrayList();
danhSach.add("Java");
danhSach.add(123);        // Bỏ Integer lẫn vào, Compiler KHÔNG cảnh báo!
danhSach.add(new Object());

String s = (String) danhSach.get(1);  // 💥 ClassCastException lúc RUNTIME!
// Phải ép kiểu thủ công, rất nguy hiểm!

// ✅ SAU (Java 5+) - Có Generics
List<String> danhSach = new ArrayList<>();
danhSach.add("Java");
danhSach.add(123);        // ❌ LỖI COMPILE! Compiler cấm ngay tại đây!

String s = danhSach.get(0);  // Không cần ép kiểu, Compiler biết đây là String
```

**Kết luận:** Generics biến **lỗi Runtime** (nguy hiểm, khó debug) thành **lỗi Compile-time** (phát hiện ngay, sửa liền). Đây là một bước tiến cực lớn về **Type Safety (An toàn kiểu)**.

---

## 2. Cú pháp Generics cơ bản

### 2.1. Generic Class (Class tổng quát)

Bạn tạo một Class có thể làm việc với **bất kỳ kiểu dữ liệu nào**, nhưng vẫn đảm bảo an toàn kiểu.

```java
// T = Type Parameter (Tham số kiểu) - Đặt tên gì cũng được, nhưng theo quy ước:
// T = Type, E = Element, K = Key, V = Value, N = Number

public class HopQua<T> {
    private T monQua;   // Kiểu dữ liệu được quyết định khi tạo Object

    public HopQua(T monQua) {
        this.monQua = monQua;
    }

    public T moHop() {
        System.out.println("🎁 Mở hộp ra được: " + monQua);
        return monQua;
    }
}

// Khi dùng: T được thay thế bằng kiểu cụ thể
HopQua<String> hopChu = new HopQua<>("Xin chào!");
String chuoi = hopChu.moHop();  // Trả về String, không cần ép kiểu

HopQua<Integer> hopSo = new HopQua<>(42);
Integer so = hopSo.moHop();     // Trả về Integer

HopQua<NhanVien> hopNV = new HopQua<>(new NhanVien("Anh", 15_000_000));
NhanVien nv = hopNV.moHop();    // Trả về NhanVien
```

### 2.2. Generic Method (Hàm tổng quát)

Bạn có thể viết 1 hàm duy nhất hoạt động cho nhiều kiểu dữ liệu khác nhau:

```java
public class TienIch {

    // <T> trước kiểu trả về = khai báo Type Parameter cho hàm
    public static <T> void inDanhSach(List<T> danhSach) {
        for (T item : danhSach) {
            System.out.println(item);
        }
    }

    // Generic Method trả về giá trị
    public static <T> T layPhanTuDau(List<T> danhSach) {
        if (danhSach == null || danhSach.isEmpty()) {
            return null;
        }
        return danhSach.get(0);  // Trả về đúng kiểu T
    }
}

// Khi dùng:
List<String> tenList = List.of("Java", "Spring", "Docker");
TienIch.inDanhSach(tenList);             // Compiler suy ra T = String

List<Integer> soList = List.of(1, 2, 3);
Integer dau = TienIch.layPhanTuDau(soList);  // Compiler suy ra T = Integer
```

### 2.3. Generic Interface

```java
// Interface tổng quát
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    T save(T entity);
    void deleteById(ID id);
}

// Class hiện thực với kiểu cụ thể
public class NhanVienRepository implements Repository<NhanVien, Long> {
    @Override
    public NhanVien findById(Long id) { /* ... */ return null; }

    @Override
    public List<NhanVien> findAll() { /* ... */ return List.of(); }

    @Override
    public NhanVien save(NhanVien entity) { /* ... */ return entity; }

    @Override
    public void deleteById(Long id) { /* ... */ }
}
```

> 💡 **Đây chính xác là cách `JpaRepository<T, ID>` của Spring Data JPA được thiết kế!**

---

## 3. Bounded Type Parameters (Giới hạn kiểu)

Đôi khi bạn không muốn `T` là **bất kỳ kiểu nào**, mà chỉ muốn nó là **một nhóm kiểu cụ thể**. Dùng từ khóa `extends` để đặt giới hạn.

### 3.1. Upper Bounded (`extends`)

```java
// T phải là Number hoặc class con của Number (Integer, Double, Long...)
public class MayTinh<T extends Number> {
    private T giaTriA;
    private T giaTriB;

    public MayTinh(T a, T b) {
        this.giaTriA = a;
        this.giaTriB = b;
    }

    public double tong() {
        // Vì T extends Number, ta chắc chắn có hàm doubleValue()
        return giaTriA.doubleValue() + giaTriB.doubleValue();
    }
}

// ✅ Hợp lệ
MayTinh<Integer> mt1 = new MayTinh<>(10, 20);
MayTinh<Double> mt2 = new MayTinh<>(3.14, 2.71);

// ❌ LỖI COMPILE! String không phải là Number
// MayTinh<String> mt3 = new MayTinh<>("a", "b");
```

### 3.2. Multiple Bounds (Nhiều giới hạn)

```java
// T phải vừa kế thừa Comparable VÀ Serializable
public <T extends Comparable<T> & Serializable> T timMax(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
```

---

## 4. Wildcards (Ký tự đại diện `?`)

Wildcard `?` nghĩa là "kiểu không xác định". Dùng khi bạn **không quan tâm** kiểu cụ thể là gì, hoặc muốn hàm **linh hoạt hơn**.

### 4.1. Ba loại Wildcard

```java
// 1. Unbounded Wildcard: ? = bất kỳ kiểu nào
public void inDanhSach(List<?> danhSach) {
    for (Object item : danhSach) {
        System.out.println(item);
    }
}
// Có thể truyền List<String>, List<Integer>, List<NhanVien>... đều được!

// 2. Upper Bounded Wildcard: ? extends Number = Number hoặc class con
public double tinhTong(List<? extends Number> soLieu) {
    double tong = 0;
    for (Number n : soLieu) {
        tong += n.doubleValue();
    }
    return tong;
}
// Có thể truyền List<Integer>, List<Double>, List<Long>...

// 3. Lower Bounded Wildcard: ? super Integer = Integer hoặc class CHA
public void themSo(List<? super Integer> danhSach) {
    danhSach.add(1);
    danhSach.add(2);
    danhSach.add(3);
}
// Có thể truyền List<Integer>, List<Number>, List<Object>...
```

### 4.2. Nguyên tắc PECS (Producer Extends, Consumer Super)

Đây là quy tắc vàng khi làm việc với Wildcard, do Joshua Bloch đặt ra trong cuốn **Effective Java**:

| Vai trò | Wildcard | Giải thích |
|---------|----------|------------|
| **Producer** (Lấy dữ liệu ra) | `? extends T` | Collection **cung cấp** dữ liệu cho bạn → dùng `extends` |
| **Consumer** (Nhét dữ liệu vào) | `? super T` | Collection **nhận** dữ liệu từ bạn → dùng `super` |

```java
// Producer: Chỉ ĐỌC từ source (extends)
public static <T> void copy(List<? extends T> source,   // Đọc từ đây
                              List<? super T> dest) {     // Ghi vào đây
    for (T item : source) {
        dest.add(item);
    }
}
```

---

## 5. Type Erasure (Xóa kiểu lúc Runtime)

Một khái niệm quan trọng: **Generics chỉ tồn tại lúc Compile-time.** Khi biên dịch xong, JVM sẽ **xóa toàn bộ thông tin kiểu** (Type Erasure) và thay bằng `Object` (hoặc bound type). Điều này nghĩa là:

```java
// Code bạn viết:
List<String> list = new ArrayList<>();
list.add("Java");
String s = list.get(0);

// Sau khi biên dịch (bytecode), JVM thấy:
List list = new ArrayList();       // Mất hết <String>
list.add("Java");
String s = (String) list.get(0);   // Compiler tự chèn ép kiểu

// Hậu quả: Lúc runtime, bạn KHÔNG THỂ kiểm tra kiểu Generic
if (list instanceof List<String>) {}  // ❌ LỖI COMPILE!
```

**Tại sao Java làm vậy?** Để đảm bảo **backward compatibility** (tương thích ngược) với code Java cũ viết trước khi có Generics.

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn xây dựng tầng Repository cho ứng dụng quản lý. Có nhiều entity: NhanVien, SanPham, DonHang. Mỗi entity đều cần các hàm CRUD giống hệt nhau (findById, findAll, save, delete). Nếu viết 3 class Repository riêng biệt, code bị copy-paste 80%.*

**Giải quyết vấn đề bằng: Generic Base Repository**

**Ví dụ thực tiễn:**

```java
// 1. Base Repository tổng quát (Viết 1 lần, dùng cho MỌI entity)
public abstract class BaseRepository<T, ID> {
    private final Map<ID, T> storage = new HashMap<>();  // Giả lập Database

    public T save(T entity, ID id) {
        storage.put(id, entity);
        System.out.println("✅ Đã lưu: " + entity);
        return entity;
    }

    public T findById(ID id) {
        T result = storage.get(id);
        if (result == null) {
            throw new RuntimeException("Không tìm thấy entity với ID: " + id);
        }
        return result;
    }

    public List<T> findAll() {
        return new ArrayList<>(storage.values());
    }

    public void deleteById(ID id) {
        storage.remove(id);
        System.out.println("🗑️ Đã xóa entity ID: " + id);
    }

    public long count() {
        return storage.size();
    }
}

// 2. Repository cụ thể: Chỉ cần extends là XONG!
// T = NhanVien, ID = Long
public class NhanVienRepository extends BaseRepository<NhanVien, Long> {
    // Có thể thêm các hàm riêng biệt cho NhanVien
    public List<NhanVien> findByLuongGreaterThan(double minLuong) {
        return findAll().stream()
            .filter(nv -> nv.getLuong() > minLuong)
            .toList();
    }
}

// T = SanPham, ID = String (mã sản phẩm)
public class SanPhamRepository extends BaseRepository<SanPham, String> {
    // Có thể thêm hàm riêng cho SanPham
}

// 3. Service sử dụng
public class NhanVienService {
    private final NhanVienRepository repo = new NhanVienRepository();

    public void demo() {
        // Type-safe: Compiler biết save() nhận NhanVien, findById() trả NhanVien
        repo.save(new NhanVien("Anh", 15_000_000), 1L);
        repo.save(new NhanVien("Bình", 25_000_000), 2L);
        repo.save(new NhanVien("Cường", 12_000_000), 3L);

        NhanVien nv = repo.findById(2L);  // Trả về NhanVien, không cần ép kiểu!
        System.out.println("Tìm thấy: " + nv.getHoTen());

        List<NhanVien> luongCao = repo.findByLuongGreaterThan(14_000_000);
        System.out.println("Lương > 14 triệu: " + luongCao.size() + " người");
    }
}
```

**Giá trị mang lại:**
- Viết `BaseRepository` **1 lần**, tái sử dụng cho **mọi entity** trong dự án.
- **An toàn kiểu hoàn toàn:** `NhanVienRepository.findById()` luôn trả về `NhanVien`, không bao giờ trả về `SanPham` hay `Object`.
- Đây chính xác là cách **Spring Data JPA** thiết kế `JpaRepository<T, ID>`. Khi bạn viết `interface UserRepo extends JpaRepository<User, Long>`, bạn đang sử dụng Generics!

---

## 6. Tổng hợp các chủ đề Generics (Bản đồ tra cứu)

### 🌟 6.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc)
1. **Cú pháp `<T>`:** Hiểu cách khai báo và sử dụng Generic Class, Generic Method, Generic Interface.
2. **Quy ước đặt tên:** `T` (Type), `E` (Element), `K` (Key), `V` (Value), `N` (Number).
3. **Diamond Operator `<>`:** Cú pháp tắt `new ArrayList<>()` (Java 7+), Compiler tự suy ra kiểu.
4. **`List<String>` vs `List<Object>`:** Hiểu rằng `List<String>` **KHÔNG phải** subtype của `List<Object>` (Invariance). Đây là nguồn gốc của nhiều bug khó hiểu.

### 🟡 6.2. Nhóm Trung cấp
1. **Bounded Types (`extends`, `super`):** Giới hạn kiểu để truy cập các hàm đặc thù (VD: `T extends Comparable<T>`).
2. **Wildcards (`?`, `? extends`, `? super`):** Viết hàm linh hoạt cho nhiều kiểu. Nắm vững nguyên tắc PECS.
3. **Type Erasure:** Hiểu rằng Generic bị xóa lúc runtime. Không thể `new T()`, không thể `instanceof List<String>`.
4. **Generic + Comparable/Comparator:** Viết hàm sắp xếp tổng quát cho bất kỳ kiểu nào implement `Comparable`.

### 🔴 6.3. Nhóm Nâng cao
1. **Recursive Type Bound:** `<T extends Comparable<T>>` — Kiểu T so sánh được với chính nó. Pattern cốt lõi cho Sorting.
2. **Type Token Pattern:** Truyền `Class<T>` để giữ lại thông tin kiểu lúc runtime (bypass Type Erasure).
3. **Generic DAO/Repository Pattern:** Thiết kế base repository tổng quát cho toàn bộ tầng persistence. Nền tảng của Spring Data.
4. **Heterogeneous Container (Typesafe):** Container chứa nhiều kiểu khác nhau nhưng vẫn type-safe. Dùng `Map<Class<?>, Object>` kết hợp Type Token.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Oracle - Generics Tutorial | [docs.oracle.com](https://docs.oracle.com/javase/tutorial/java/generics/index.html) |
| Oracle - Generics FAQ (Angelika Langer) | [angelikalanger.com](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html) |
| Baeldung - Java Generics | [baeldung.com](https://www.baeldung.com/java-generics) |
| Dev.java - Generics | [dev.java](https://dev.java/learn/generics/) |
| Effective Java (Joshua Bloch) - Chapter 5 | Item 26-33: Best practices cho Generics |
