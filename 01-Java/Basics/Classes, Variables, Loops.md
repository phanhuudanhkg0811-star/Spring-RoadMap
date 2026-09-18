# 🏗️ Classes, Variables, Loops (Lớp, Biến, Vòng lặp)

> **Thuộc roadmap:** `1. Java → Basics → Classes, Variables, Loops etc`

---

## 1. Tổng quan về nền tảng Java

Java là ngôn ngữ **hướng đối tượng (OOP)**, tức là mọi thứ đều xoay quanh **Object (Đối tượng)** và **Class (Bản thiết kế)**. Trước khi bay vào các framework phức tạp như Spring Boot, bạn phải nắm chắc 3 viên gạch nền tảng này:

1. **Class & Object:** Bản thiết kế và sản phẩm thực tế được tạo ra từ bản thiết kế đó.
2. **Variables (Biến):** Nơi lưu trữ dữ liệu tạm thời trong bộ nhớ (RAM).
3. **Loops (Vòng lặp):** Cơ chế để lặp đi lặp lại một hành động mà không cần copy-paste code.

---

## 2. Class & Object (Lớp & Đối tượng)

**Class** giống như một bản vẽ kỹ thuật (blueprint). **Object** là sản phẩm thật được đúc ra từ bản vẽ đó.

```java
// Bản thiết kế (Class)
public class NhanVien {
    // Thuộc tính (Properties / Fields)
    String hoTen;
    int tuoi;
    double luong;

    // Hành vi (Method / Behavior)
    public void lamViec() {
        System.out.println(hoTen + " đang làm việc...");
    }
}

// Đúc ra sản phẩm thật (Object) từ bản thiết kế
public class Main {
    public static void main(String[] args) {
        NhanVien nv1 = new NhanVien();  // Object thứ nhất
        nv1.hoTen = "Nguyễn Văn A";
        nv1.tuoi = 25;
        nv1.luong = 15_000_000;
        nv1.lamViec(); // Output: Nguyễn Văn A đang làm việc...

        NhanVien nv2 = new NhanVien();  // Object thứ hai (khác nv1 hoàn toàn)
        nv2.hoTen = "Trần Thị B";
        nv2.lamViec(); // Output: Trần Thị B đang làm việc...
    }
}
```

### Constructor (Hàm khởi tạo)

Constructor là một hàm **đặc biệt** được gọi tự động khi bạn dùng từ khóa `new`. Nó dùng để "nhồi" dữ liệu ban đầu vào Object ngay lúc tạo ra.

```java
public class NhanVien {
    String hoTen;
    double luong;

    // Constructor: Trùng tên với Class, KHÔNG có kiểu trả về
    public NhanVien(String hoTen, double luong) {
        this.hoTen = hoTen;   // this.hoTen = thuộc tính của Object
        this.luong = luong;   // hoTen (không có this) = tham số truyền vào
    }
}

// Khi dùng:
NhanVien nv = new NhanVien("Nguyễn Văn A", 15_000_000);
// Không cần gán từng thuộc tính nữa!
```

---

## 3. Variables (Biến)

Biến là "cái hộp" chứa dữ liệu trong RAM. Java là ngôn ngữ **kiểu tĩnh (Statically Typed)**, tức là bạn phải khai báo kiểu dữ liệu từ trước.

### 3.1. Kiểu dữ liệu nguyên thủy (Primitive Types) - 8 kiểu

| Kiểu | Kích thước | Phạm vi giá trị | Dùng khi |
|------|-----------|------------------|----------|
| `byte` | 1 byte | -128 → 127 | Tiết kiệm RAM, dữ liệu nhỏ |
| `short` | 2 bytes | -32,768 → 32,767 | Ít dùng |
| `int` | 4 bytes | ±2.1 tỷ | **Mặc định cho số nguyên** |
| `long` | 8 bytes | Cực lớn | ID từ Database, timestamp |
| `float` | 4 bytes | 7 chữ số thập phân | Ít dùng (kém chính xác) |
| `double` | 8 bytes | 15 chữ số thập phân | **Mặc định cho số thực** |
| `char` | 2 bytes | 1 ký tự Unicode | Ký tự đơn lẻ |
| `boolean` | 1 bit | `true` / `false` | Điều kiện, cờ (flag) |

### 3.2. Kiểu tham chiếu (Reference Types)

Tất cả những gì **không phải Primitive** đều là Reference Type. Biến tham chiếu không chứa giá trị thật, mà chứa **địa chỉ bộ nhớ** trỏ tới Object trên Heap.

```java
// Primitive: Biến chứa giá trị thật
int a = 10;
int b = a;    // Copy giá trị → b = 10, a và b độc lập nhau
b = 99;       // a vẫn = 10 (không bị ảnh hưởng)

// Reference: Biến chứa địa chỉ
NhanVien nv1 = new NhanVien("Anh", 10_000_000);
NhanVien nv2 = nv1;  // Copy địa chỉ → nv2 trỏ tới cùng Object với nv1
nv2.luong = 50_000_000;  // nv1.luong CŨNG bị đổi thành 50 triệu! (Cùng Object)
```

### 3.3. Phạm vi biến (Variable Scope)

```java
public class Demo {
    static int bienStatic = 0;  // Biến của Class (dùng chung cho mọi Object)
    int bienInstance = 0;        // Biến của Object (mỗi Object có bản copy riêng)

    public void method() {
        int bienLocal = 0;       // Biến cục bộ (chỉ sống trong method này)
        // Ra khỏi method → biến bị xóa khỏi RAM
    }
}
```

---

## 4. Loops (Vòng lặp)

Vòng lặp giúp bạn thực hiện một hành động **lặp đi lặp lại** mà không cần viết code trùng lặp.

### 4.1. Các loại vòng lặp

```java
// 1. FOR loop: Biết trước số lần lặp
for (int i = 0; i < 5; i++) {
    System.out.println("Lần lặp thứ " + i);
}

// 2. WHILE loop: Không biết trước, lặp cho tới khi điều kiện sai
int soLanThu = 0;
while (soLanThu < 3) {
    System.out.println("Đang thử lần " + soLanThu);
    soLanThu++;
}

// 3. DO-WHILE loop: Luôn chạy ít nhất 1 lần rồi mới kiểm tra điều kiện
int pin = -1;
do {
    pin = nhapPinTuBanPhim();
} while (pin != 1234);  // Hỏi mật khẩu cho đến khi đúng

// 4. FOR-EACH loop (Enhanced for): Duyệt qua mảng/collection cực gọn
String[] danhSach = {"Java", "Spring", "Docker"};
for (String item : danhSach) {
    System.out.println(item);
}
```

### 4.2. Từ khóa điều khiển

```java
// break: Thoát khỏi vòng lặp ngay lập tức
for (int i = 0; i < 100; i++) {
    if (i == 5) break;  // Dừng lại khi i = 5
    System.out.println(i);
}

// continue: Bỏ qua lần lặp hiện tại, nhảy sang lần lặp kế tiếp
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;  // Bỏ qua số chẵn
    System.out.println(i);     // Chỉ in số lẻ
}
```

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn đang xây dựng module quản lý Đơn hàng cho một sàn thương mại điện tử. Mỗi đơn hàng gồm nhiều sản phẩm, và bạn cần tính tổng tiền, áp dụng mã giảm giá, rồi in hóa đơn.*

**Ví dụ thực tiễn:**

```java
// 1. Class = Bản thiết kế cho 1 sản phẩm trong giỏ hàng
public class SanPham {
    private String tenSP;      // Biến instance (thuộc tính)
    private double donGia;
    private int soLuong;

    // Constructor
    public SanPham(String tenSP, double donGia, int soLuong) {
        this.tenSP = tenSP;
        this.donGia = donGia;
        this.soLuong = soLuong;
    }

    public double thanhTien() {
        return donGia * soLuong;
    }

    // Getter
    public String getTenSP() { return tenSP; }
    public double getDonGia() { return donGia; }
    public int getSoLuong() { return soLuong; }
}

// 2. Class = Bản thiết kế cho 1 Đơn hàng
public class DonHang {
    private SanPham[] danhSachSP;    // Mảng Object (Reference Type)
    private double phanTramGiam;     // Biến primitive

    public DonHang(SanPham[] danhSachSP, double phanTramGiam) {
        this.danhSachSP = danhSachSP;
        this.phanTramGiam = phanTramGiam;
    }

    public double tinhTongTien() {
        double tong = 0;    // Biến local

        // Loop: Duyệt qua từng sản phẩm để cộng dồn
        for (SanPham sp : danhSachSP) {
            tong += sp.thanhTien();
        }

        // Áp dụng giảm giá
        return tong * (1 - phanTramGiam / 100);
    }

    public void inHoaDon() {
        System.out.println("========= HÓA ĐƠN =========");
        for (int i = 0; i < danhSachSP.length; i++) {
            SanPham sp = danhSachSP[i];
            System.out.printf("%d. %s | SL: %d | Đơn giá: %,.0f | Thành tiền: %,.0f%n",
                i + 1, sp.getTenSP(), sp.getSoLuong(), sp.getDonGia(), sp.thanhTien());
        }
        System.out.printf("Giảm giá: %.0f%%%n", phanTramGiam);
        System.out.printf("💰 TỔNG THANH TOÁN: %,.0f VND%n", tinhTongTien());
    }
}

// 3. Chạy thử
public class Main {
    public static void main(String[] args) {
        SanPham[] gioHang = {
            new SanPham("Laptop Dell", 25_000_000, 1),
            new SanPham("Chuột Logitech", 500_000, 2),
            new SanPham("Bàn phím cơ", 1_200_000, 1)
        };

        DonHang don = new DonHang(gioHang, 10);  // Giảm 10%
        don.inHoaDon();
    }
}
```

**Output:**
```
========= HÓA ĐƠN =========
1. Laptop Dell | SL: 1 | Đơn giá: 25,000,000 | Thành tiền: 25,000,000
2. Chuột Logitech | SL: 2 | Đơn giá: 500,000 | Thành tiền: 1,000,000
3. Bàn phím cơ | SL: 1 | Đơn giá: 1,200,000 | Thành tiền: 1,200,000
Giảm giá: 10%
💰 TỔNG THANH TOÁN: 24,480,000 VND
```

**Giá trị mang lại:** Mọi kiến thức nền tảng (Class, Variable, Loop) đều hội tụ trong 1 bài toán thực tế. Class `SanPham` là bản thiết kế, `new SanPham(...)` tạo Object, các biến `tong`, `i` là biến local, và vòng lặp `for-each` duyệt qua danh sách sản phẩm.

---

## 5. Tổng hợp các chủ đề (Bản đồ tra cứu)

### 🌟 5.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc)
1. **Class & Object:** Phân biệt được Class (bản thiết kế) vs Object (sản phẩm). Biết viết Constructor, Method.
2. **Primitive vs Reference Type:** Hiểu rõ sự khác biệt giữa "chứa giá trị" và "chứa địa chỉ". Đây là nguồn gốc của rất nhiều bug khó hiểu.
3. **Vòng lặp `for` và `for-each`:** Kỹ năng duyệt mảng/collection cơ bản nhất, dùng mọi lúc mọi nơi.
4. **`if-else`, `switch`:** Cấu trúc rẽ nhánh, điều kiện. Nền tảng của mọi logic nghiệp vụ.

### 🟡 5.2. Nhóm Trung cấp
1. **Access Modifiers (`private`, `public`, `protected`, default):** Kiểm soát ai được phép nhìn thấy/sử dụng thuộc tính và hàm. Quan trọng khi thiết kế API.
2. **`static` keyword:** Phân biệt rõ thành viên thuộc về Class vs thuộc về Object. Ảnh hưởng trực tiếp tới cách Spring Bean hoạt động.
3. **`final` keyword:** Đánh dấu biến/hàm/class không được phép thay đổi. Cốt lõi của Immutability.
4. **String Pool & Immutable String:** Hiểu tại sao `String` là bất biến, khi nào dùng `StringBuilder`.

### 🔴 5.3. Nhóm Nâng cao
1. **`var` (Java 10+):** Suy luận kiểu tự động ở biến local. Code gọn hơn nhưng cần cân nhắc khả năng đọc code.
2. **Record (Java 16+):** Class bất biến tự sinh Constructor, Getter, `equals()`, `hashCode()`, `toString()`. Cực kỳ tiện cho DTO.
3. **Sealed Classes (Java 17+):** Giới hạn danh sách class con được phép kế thừa. Hữu ích cho domain modeling.
4. **Pattern Matching `switch` (Java 21+):** Switch cải tiến, kết hợp kiểm tra kiểu và destructuring trong 1 câu lệnh.

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Oracle - Java Tutorials (Classes & Objects) | [docs.oracle.com](https://docs.oracle.com/javase/tutorial/java/javaOO/index.html) |
| Oracle - Language Basics | [docs.oracle.com](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html) |
| Baeldung - Java Basics | [baeldung.com](https://www.baeldung.com/java-tutorial) |
| Dev.java (Tài liệu chính thức mới) | [dev.java](https://dev.java/learn/) |
