# 📦 Collections Framework (Bộ sưu tập dữ liệu)

> **Thuộc roadmap:** `1. Java → Basics → Collections`

---

## 1. Collections là gì?

Khi lập trình, bạn thường xuyên phải xử lý **một nhóm nhiều Object** (danh sách sản phẩm, tập hợp user, hàng đợi đơn hàng...). Mảng (`Array`) truyền thống có nhiều hạn chế: kích thước cố định, không thể thêm/xóa linh hoạt, thiếu các hàm tiện ích.

**Java Collections Framework** là bộ thư viện chuẩn cung cấp các **cấu trúc dữ liệu** (Data Structures) mạnh mẽ, linh hoạt, sẵn sàng xài. Nó nằm trong package `java.util`.

### Hệ thống phân cấp Collections:

```
                     Iterable<E>
                         │
                    Collection<E>
                    ┌────┼────────┐
                    │    │        │
                 List<E> Set<E>  Queue<E>
                    │    │        │
                    │    │        ├── Deque<E>
                    │    │        │
              ┌─────┤    ├────┐  │
              │     │    │    │  │
         ArrayList  │  HashSet│  LinkedList (vừa List vừa Deque)
              │     │    │    │  │
         LinkedList │ TreeSet │  PriorityQueue
                    │    │    │
                    │ LinkedHashSet
                    │
                    ├── Vector (Legacy, ít dùng)

                    Map<K,V>  (KHÔNG thuộc Collection interface)
                    ┌───┼────────┐
                    │   │        │
               HashMap TreeMap  LinkedHashMap
                    │
                  Hashtable (Legacy, ít dùng)
```

---

## 2. List (Danh sách có thứ tự)

**Đặc điểm:** Có thứ tự, cho phép phần tử trùng lặp, truy cập bằng chỉ số (index).

### 2.1. ArrayList (Dùng nhiều nhất)

Bên trong là một **mảng động** (Dynamic Array). Khi hết chỗ, nó tự tạo mảng mới lớn hơn 50% và copy dữ liệu sang.

```java
import java.util.ArrayList;
import java.util.List;

List<String> danhSach = new ArrayList<>();

// Thêm phần tử
danhSach.add("Java");
danhSach.add("Spring");
danhSach.add("Docker");
danhSach.add("Java");  // Cho phép trùng lặp!

// Truy cập theo index
String first = danhSach.get(0);  // "Java"

// Xóa phần tử
danhSach.remove("Docker");        // Xóa theo giá trị
danhSach.remove(0);               // Xóa theo index

// Duyệt danh sách
for (String item : danhSach) {
    System.out.println(item);
}

// Kiểm tra
boolean coSpring = danhSach.contains("Spring");  // true
int kichThuoc = danhSach.size();                  // Kích thước hiện tại
```

### 2.2. LinkedList

Bên trong là **danh sách liên kết kép** (Doubly Linked List). Mỗi phần tử (node) chứa giá trị + con trỏ tới node trước và node sau.

| Thao tác | ArrayList | LinkedList |
|----------|-----------|------------|
| Truy cập index `get(i)` | ⚡ O(1) - Cực nhanh | 🐌 O(n) - Phải duyệt |
| Thêm/xóa ở đầu/cuối | 🐌 O(n) - Phải dịch mảng | ⚡ O(1) - Cực nhanh |
| Thêm/xóa ở giữa | 🐌 O(n) | 🐌 O(n) |
| Bộ nhớ | Ít hơn | Tốn hơn (lưu thêm con trỏ) |

**Quy tắc ngón tay cái:** 95% trường hợp dùng `ArrayList`. Chỉ dùng `LinkedList` khi bạn cần thêm/xóa ở **đầu danh sách** liên tục (VD: implement Stack, Queue).

---

## 3. Set (Tập hợp không trùng lặp)

**Đặc điểm:** **Không cho phép phần tử trùng lặp.** Mỗi phần tử chỉ xuất hiện đúng 1 lần.

### 3.1. HashSet (Dùng nhiều nhất)

```java
import java.util.HashSet;
import java.util.Set;

Set<String> tags = new HashSet<>();
tags.add("java");
tags.add("spring");
tags.add("java");     // Bị bỏ qua! Không thêm trùng
tags.add("docker");

System.out.println(tags.size());  // 3 (không phải 4)
System.out.println(tags);        // [java, spring, docker] (THỨ TỰ KHÔNG ĐẢM BẢO!)
```

### 3.2. So sánh 3 loại Set

| Loại Set | Thứ tự | Tốc độ | Khi nào dùng |
|----------|--------|--------|-------------|
| `HashSet` | ❌ Không có thứ tự | ⚡ Nhanh nhất O(1) | Mặc định, chỉ cần "không trùng" |
| `LinkedHashSet` | ✅ Giữ thứ tự chèn | ⚡ Gần bằng HashSet | Cần không trùng VÀ giữ thứ tự |
| `TreeSet` | ✅ Tự sắp xếp (Sorted) | 🐌 O(log n) | Cần sắp xếp tự động |

### Lưu ý quan trọng: `equals()` và `hashCode()`

Khi bạn bỏ **Object tự tạo** vào `HashSet` hoặc dùng làm key trong `HashMap`, bạn **BẮT BUỘC** phải override 2 hàm `equals()` và `hashCode()`. Nếu không, Set sẽ không nhận ra 2 Object giống nhau!

```java
public class SanPham {
    private String maSP;
    private String tenSP;

    // PHẢI override để HashSet biết khi nào 2 SanPham là "giống nhau"
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        SanPham sp = (SanPham) o;
        return Objects.equals(maSP, sp.maSP);  // So sánh theo mã SP
    }

    @Override
    public int hashCode() {
        return Objects.hash(maSP);
    }
}
```

---

## 4. Map (Bản đồ Key-Value)

**Đặc điểm:** Lưu dữ liệu theo cặp **Key → Value**. Key không được trùng, Value được phép trùng.

### 4.1. HashMap (Dùng nhiều nhất)

```java
import java.util.HashMap;
import java.util.Map;

Map<String, Double> bangLuong = new HashMap<>();

// Thêm cặp key-value
bangLuong.put("Anh", 15_000_000.0);
bangLuong.put("Bình", 20_000_000.0);
bangLuong.put("Cường", 12_000_000.0);

// Truy cập theo key
double luongAnh = bangLuong.get("Anh");  // 15,000,000

// Kiểm tra key tồn tại
if (bangLuong.containsKey("Bình")) {
    System.out.println("Bình có trong danh sách!");
}

// Giá trị mặc định nếu key không tồn tại
double luong = bangLuong.getOrDefault("Đức", 0.0);  // 0.0

// Duyệt Map
for (Map.Entry<String, Double> entry : bangLuong.entrySet()) {
    System.out.printf("%s: %,.0f VND%n", entry.getKey(), entry.getValue());
}

// Duyệt chỉ key hoặc chỉ value
for (String ten : bangLuong.keySet()) { /* ... */ }
for (Double luongNV : bangLuong.values()) { /* ... */ }
```

### 4.2. So sánh 3 loại Map

| Loại Map | Thứ tự Key | Tốc độ | Khi nào dùng |
|----------|-----------|--------|-------------|
| `HashMap` | ❌ Không có thứ tự | ⚡ O(1) | Mặc định, dùng hầu hết mọi nơi |
| `LinkedHashMap` | ✅ Giữ thứ tự chèn | ⚡ Gần bằng HashMap | Cần giữ thứ tự chèn (VD: JSON output) |
| `TreeMap` | ✅ Tự sắp xếp theo Key | 🐌 O(log n) | Cần key luôn được sắp xếp |

---

## 5. Queue & Deque (Hàng đợi)

**Queue:** FIFO (First In, First Out) - Ai vào trước thì ra trước (giống xếp hàng mua vé).

**Deque:** Double-Ended Queue - Có thể thêm/xóa ở CẢ HAI ĐẦU.

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.ArrayDeque;
import java.util.Deque;

// Queue: Hàng đợi xử lý đơn hàng
Queue<String> hangDoi = new LinkedList<>();
hangDoi.offer("Đơn hàng #001");  // Thêm vào cuối
hangDoi.offer("Đơn hàng #002");
hangDoi.offer("Đơn hàng #003");

String donTiepTheo = hangDoi.poll();  // Lấy ra từ đầu: "Đơn hàng #001"
String xemTruoc = hangDoi.peek();     // Xem đầu hàng mà KHÔNG lấy ra: "#002"

// Deque làm Stack (LIFO: Last In, First Out)
Deque<String> stack = new ArrayDeque<>();
stack.push("Trang chủ");       // Đẩy vào đỉnh
stack.push("Danh mục");
stack.push("Chi tiết SP");

String trangHienTai = stack.pop();  // Lấy từ đỉnh: "Chi tiết SP" (quay lại)
```

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn xây dựng API thống kê bán hàng. Dữ liệu gồm danh sách hàng ngàn đơn hàng. Bạn cần: (1) Lọc đơn trùng, (2) Đếm số lượng bán theo từng sản phẩm, (3) Sắp xếp top sản phẩm bán chạy.*

**Ví dụ thực tiễn:**

```java
import java.util.*;
import java.util.stream.*;

public class ThongKeBanHang {

    public static void main(String[] args) {
        // Dữ liệu giả lập: Danh sách sản phẩm đã bán
        List<String> donHangRaw = List.of(
            "iPhone 15", "Galaxy S24", "iPhone 15", "Pixel 9",
            "Galaxy S24", "iPhone 15", "Pixel 9", "Galaxy S24",
            "iPhone 15", "Pixel 9", "Galaxy S24", "Galaxy S24"
        );

        // 1. Set: Lọc ra danh sách sản phẩm KHÔNG TRÙNG LẶP
        Set<String> sanPhamUnique = new HashSet<>(donHangRaw);
        System.out.println("Sản phẩm unique: " + sanPhamUnique);
        // Output: [Pixel 9, iPhone 15, Galaxy S24]

        // 2. Map: Đếm số lượng bán theo từng sản phẩm
        Map<String, Integer> thongKe = new HashMap<>();
        for (String sp : donHangRaw) {
            thongKe.put(sp, thongKe.getOrDefault(sp, 0) + 1);
        }
        System.out.println("Thống kê: " + thongKe);
        // Output: {Pixel 9=3, iPhone 15=4, Galaxy S24=5}

        // 3. List + Sort: Sắp xếp top bán chạy (dùng Map.Entry)
        List<Map.Entry<String, Integer>> topBanChay = new ArrayList<>(thongKe.entrySet());
        topBanChay.sort((a, b) -> b.getValue() - a.getValue()); // Giảm dần

        System.out.println("\n🏆 TOP SẢN PHẨM BÁN CHẠY:");
        int hang = 1;
        for (Map.Entry<String, Integer> entry : topBanChay) {
            System.out.printf("  %d. %s → %d chiếc%n", hang++,
                entry.getKey(), entry.getValue());
        }
    }
}
```

**Output:**
```
Sản phẩm unique: [Pixel 9, iPhone 15, Galaxy S24]
Thống kê: {Pixel 9=3, iPhone 15=4, Galaxy S24=5}

🏆 TOP SẢN PHẨM BÁN CHẠY:
  1. Galaxy S24 → 5 chiếc
  2. iPhone 15 → 4 chiếc
  3. Pixel 9 → 3 chiếc
```

**Giá trị mang lại:** `Set` xử lý "không trùng", `Map` xử lý "đếm theo nhóm", `List` xử lý "sắp xếp". Ba cấu trúc dữ liệu phối hợp nhịp nhàng. Đây chính xác là những gì bạn làm hàng ngày khi viết API backend.

---

## 6. Collections Utility Class

Java cung cấp class tiện ích `Collections` (có chữ "s") chứa các hàm static cực hữu ích:

```java
List<Integer> soLieu = new ArrayList<>(List.of(5, 2, 8, 1, 9, 3));

Collections.sort(soLieu);                    // Sắp xếp tăng dần: [1, 2, 3, 5, 8, 9]
Collections.reverse(soLieu);                 // Đảo ngược: [9, 8, 5, 3, 2, 1]
Collections.shuffle(soLieu);                 // Xáo trộn ngẫu nhiên
int max = Collections.max(soLieu);           // Phần tử lớn nhất
int min = Collections.min(soLieu);           // Phần tử nhỏ nhất

// Tạo danh sách bất biến (Không ai được add/remove)
List<String> immutable = Collections.unmodifiableList(soLieu);

// Java 9+: Cách tạo collection bất biến gọn hơn
List<String> list = List.of("A", "B", "C");
Set<String> set = Set.of("X", "Y", "Z");
Map<String, Integer> map = Map.of("key1", 1, "key2", 2);
```

---

## 7. Tổng hợp các chủ đề Collections (Bản đồ tra cứu)

### 🌟 7.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc)
1. **ArrayList:** Danh sách động dùng nhiều nhất. Biết `add`, `get`, `remove`, `size`, `contains`.
2. **HashMap:** Cấu trúc Key-Value dùng nhiều nhất. Biết `put`, `get`, `containsKey`, `entrySet`.
3. **HashSet:** Tập hợp không trùng lặp. Hiểu tại sao phải override `equals()` + `hashCode()`.
4. **For-each loop & Iterator:** 2 cách duyệt Collection chuẩn.

### 🟡 7.2. Nhóm Trung cấp
1. **LinkedHashMap / LinkedHashSet:** Giữ thứ tự chèn. Hay dùng khi cần JSON output có thứ tự cố định.
2. **TreeMap / TreeSet:** Tự sắp xếp. Hiểu `Comparable` vs `Comparator` interface.
3. **Queue / Deque / PriorityQueue:** Hàng đợi FIFO, Stack LIFO, và hàng đợi ưu tiên.
4. **`Collections.unmodifiableList()` / `List.of()`:** Tạo collection bất biến (immutable). Cực kỳ quan trọng trong thiết kế API an toàn.

### 🔴 7.3. Nhóm Nâng cao
1. **ConcurrentHashMap:** Phiên bản thread-safe của HashMap. Dùng trong môi trường đa luồng (Multi-threading).
2. **CopyOnWriteArrayList:** List thread-safe, tối ưu cho trường hợp đọc nhiều ghi ít.
3. **EnumMap / EnumSet:** Collection tối ưu hiệu năng khi Key là Enum.
4. **SequencedCollection (Java 21+):** Interface mới thống nhất thao tác truy cập phần tử đầu/cuối cho List, Deque, SortedSet...

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Oracle - Collections Framework | [docs.oracle.com](https://docs.oracle.com/javase/tutorial/collections/index.html) |
| Oracle - Collections API Design FAQ | [docs.oracle.com](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/doc-files/coll-overview.html) |
| Baeldung - Java Collections | [baeldung.com](https://www.baeldung.com/java-collections) |
| Dev.java - Collections Framework | [dev.java](https://dev.java/learn/api/collections-framework/) |
