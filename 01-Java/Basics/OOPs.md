# 🧬 OOPs - Lập trình Hướng đối tượng (Object-Oriented Programming)

> **Thuộc roadmap:** `1. Java → Basics → OOPs`

---

## 1. OOP là gì?

OOP (Object-Oriented Programming) là **triết lý lập trình** mà trong đó mọi thứ được mô hình hóa thành **Object (Đối tượng)**. Thay vì viết code theo kiểu "từ trên xuống dưới" (Procedural), bạn thiết kế hệ thống bằng cách chia nhỏ thành các đối tượng có **dữ liệu riêng** và **hành vi riêng**, rồi cho chúng tương tác với nhau.

Java là ngôn ngữ **OOP thuần túy** (trừ các kiểu Primitive). Toàn bộ Spring Framework, Hibernate, và mọi thư viện Java đều được xây dựng trên 4 trụ cột OOP.

### 4 Trụ cột của OOP:
1. **Encapsulation (Đóng gói):** Giấu dữ liệu bên trong, chỉ cho phép truy cập qua cổng kiểm soát.
2. **Inheritance (Kế thừa):** Class con thừa hưởng tài sản (thuộc tính + hành vi) từ Class cha.
3. **Polymorphism (Đa hình):** Cùng 1 hành vi nhưng mỗi đối tượng thể hiện theo cách riêng.
4. **Abstraction (Trừu tượng hóa):** Ẩn đi chi tiết phức tạp, chỉ lộ ra giao diện đơn giản.

---

## 2. Encapsulation (Đóng gói)

**Ý tưởng:** Bạn không bao giờ cho ai tự do sờ vào ruột gan dữ liệu của Object. Mọi truy cập đều phải đi qua "cổng kiểm soát" là Getter/Setter.

**Tại sao?** Vì nếu để thuộc tính `public`, bất kỳ ai cũng có thể gán giá trị bậy bạ (ví dụ: `nhanVien.tuoi = -5`). Encapsulation cho phép bạn **kiểm tra dữ liệu trước khi chấp nhận**.

```java
public class TaiKhoanNganHang {
    private double soDu;  // PRIVATE: Không ai được sờ trực tiếp!

    // Getter - Cổng đọc dữ liệu
    public double getSoDu() {
        return soDu;
    }

    // Setter - Cổng ghi dữ liệu (CÓ KIỂM TRA)
    public void napTien(double soTien) {
        if (soTien <= 0) {
            throw new IllegalArgumentException("Số tiền nạp phải lớn hơn 0!");
        }
        this.soDu += soTien;
    }

    public void rutTien(double soTien) {
        if (soTien > soDu) {
            throw new IllegalArgumentException("Số dư không đủ!");
        }
        this.soDu -= soTien;
    }
}
```
**Kết quả:** Không ai có thể gán `taiKhoan.soDu = -999_999_999` được nữa. Mọi thao tác đều phải qua `napTien()` / `rutTien()` và bị kiểm tra nghiêm ngặt.

---

## 3. Inheritance (Kế thừa)

**Ý tưởng:** Nếu nhiều Class có chung thuộc tính/hành vi, hãy gom phần chung đó vào 1 **Class cha (Parent/Super)**, rồi cho các **Class con (Child/Sub)** kế thừa lại.

```java
// Class cha: Chứa phần CHUNG
public class NhanVien {
    protected String hoTen;
    protected double luongCoBan;

    public NhanVien(String hoTen, double luongCoBan) {
        this.hoTen = hoTen;
        this.luongCoBan = luongCoBan;
    }

    public double tinhLuong() {
        return luongCoBan;
    }

    public void hienThiThongTin() {
        System.out.printf("%s | Lương: %,.0f VND%n", hoTen, tinhLuong());
    }
}

// Class con: Kế thừa + Bổ sung phần RIÊNG
public class NhanVienKinhDoanh extends NhanVien {
    private double doanhSo;
    private double tyLeHoaHong;  // VD: 5%

    public NhanVienKinhDoanh(String hoTen, double luongCoBan,
                              double doanhSo, double tyLeHoaHong) {
        super(hoTen, luongCoBan);  // Gọi Constructor cha
        this.doanhSo = doanhSo;
        this.tyLeHoaHong = tyLeHoaHong;
    }

    @Override  // Ghi đè hàm của cha (Polymorphism)
    public double tinhLuong() {
        return luongCoBan + (doanhSo * tyLeHoaHong / 100);
    }
}

// Class con khác
public class NhanVienQuanLy extends NhanVien {
    private double phuCapChucVu;

    public NhanVienQuanLy(String hoTen, double luongCoBan, double phuCapChucVu) {
        super(hoTen, luongCoBan);
        this.phuCapChucVu = phuCapChucVu;
    }

    @Override
    public double tinhLuong() {
        return luongCoBan + phuCapChucVu;
    }
}
```

### Từ khóa quan trọng:
- **`extends`:** Class con kế thừa Class cha.
- **`super`:** Gọi Constructor hoặc Method của Class cha.
- **`@Override`:** Đánh dấu rõ ràng rằng "tôi đang ghi đè hàm của cha".
- **`protected`:** Cho phép Class con truy cập, nhưng vẫn cấm bên ngoài.

---

## 4. Polymorphism (Đa hình)

**Ý tưởng:** Cùng 1 lời gọi hàm, nhưng tùy thuộc vào Object thực tế mà kết quả **khác nhau**. Đây là trụ cột mạnh nhất của OOP.

```java
public class Main {
    public static void main(String[] args) {
        // Khai báo kiểu CHA, nhưng tạo Object kiểu CON
        NhanVien nv1 = new NhanVienKinhDoanh("Anh", 10_000_000, 500_000_000, 5);
        NhanVien nv2 = new NhanVienQuanLy("Bình", 15_000_000, 5_000_000);
        NhanVien nv3 = new NhanVien("Cường", 8_000_000);

        // Gọi CÙNG 1 hàm tinhLuong() nhưng kết quả KHÁC NHAU
        NhanVien[] danhSach = { nv1, nv2, nv3 };
        for (NhanVien nv : danhSach) {
            nv.hienThiThongTin();
        }
    }
}
```

**Output:**
```
Anh | Lương: 35,000,000 VND      ← (10tr + 500tr * 5%)
Bình | Lương: 20,000,000 VND     ← (15tr + 5tr phụ cấp)
Cường | Lương: 8,000,000 VND     ← (Lương cơ bản)
```

**Magic:** Bạn chỉ gọi `nv.tinhLuong()` mà không cần biết `nv` là kiểu gì. JVM sẽ tự tìm đúng phiên bản `tinhLuong()` để chạy. Đây gọi là **Dynamic Dispatch (Phân phối động)**.

### 2 loại Polymorphism:
1. **Compile-time (Method Overloading):** Cùng tên hàm, khác tham số. JVM chọn hàm lúc biên dịch.
2. **Runtime (Method Overriding):** Class con ghi đè hàm Class cha. JVM chọn hàm lúc chạy.

```java
// Overloading: Cùng tên "tinh", nhưng số lượng/kiểu tham số khác nhau
public class MayTinh {
    public int tinh(int a, int b) { return a + b; }
    public double tinh(double a, double b) { return a + b; }
    public int tinh(int a, int b, int c) { return a + b + c; }
}
```

---

## 5. Abstraction (Trừu tượng hóa)

**Ý tưởng:** Ẩn đi phần ruột gan phức tạp, chỉ đưa ra "hợp đồng" (contract) gồm các hàm mà bạn cần gọi. Có 2 công cụ:

### 5.1. Abstract Class

```java
// Abstract Class = Bản thiết kế "chưa hoàn chỉnh"
// Không thể tạo Object trực tiếp bằng new
public abstract class HinhHoc {
    protected String tenHinh;

    public HinhHoc(String tenHinh) {
        this.tenHinh = tenHinh;
    }

    // Hàm trừu tượng: Chỉ khai báo, KHÔNG có thân hàm
    // Bắt buộc class con phải tự viết code hiện thực
    public abstract double tinhDienTich();

    // Hàm bình thường: Có thân hàm sẵn, class con được kế thừa luôn
    public void inThongTin() {
        System.out.printf("%s | Diện tích: %.2f%n", tenHinh, tinhDienTich());
    }
}

public class HinhTron extends HinhHoc {
    private double banKinh;

    public HinhTron(double banKinh) {
        super("Hình tròn");
        this.banKinh = banKinh;
    }

    @Override
    public double tinhDienTich() {
        return Math.PI * banKinh * banKinh;  // Bắt buộc phải viết!
    }
}
```

### 5.2. Interface

```java
// Interface = Hợp đồng thuần túy (100% trừu tượng)
public interface ThanhToan {
    boolean thanhToan(double soTien);    // Mọi class implement phải viết hàm này
    String getLoaiThanhToan();
}

// Class hiện thực hợp đồng
public class ThanhToanMomo implements ThanhToan {
    @Override
    public boolean thanhToan(double soTien) {
        System.out.println("Đang thanh toán " + soTien + " qua Momo...");
        return true;
    }

    @Override
    public String getLoaiThanhToan() { return "Momo"; }
}

public class ThanhToanVNPay implements ThanhToan {
    @Override
    public boolean thanhToan(double soTien) {
        System.out.println("Đang thanh toán " + soTien + " qua VNPay...");
        return true;
    }

    @Override
    public String getLoaiThanhToan() { return "VNPay"; }
}
```

### Khi nào dùng Abstract Class vs Interface?

| Tiêu chí | Abstract Class | Interface |
|----------|----------------|-----------|
| Dùng `extends` hay `implements` | `extends` | `implements` |
| Đa kế thừa | ❌ Chỉ kế thừa 1 | ✅ Implement nhiều |
| Có Constructor | ✅ Có | ❌ Không |
| Có thuộc tính thường | ✅ Có | ❌ Chỉ có `static final` |
| Khi nào dùng | Các class con **có chung bản chất** (IS-A mạnh) | Các class **có chung hành vi** nhưng bản chất khác nhau |

---

## 🔥 Ứng dụng thực tế (Thực tế dùng để làm gì? Giải quyết vấn đề gì?)

*Bài toán: Bạn xây dựng hệ thống Thanh toán đơn hàng. Khách hàng có thể chọn thanh toán bằng Momo, VNPay, hoặc thẻ tín dụng. Tương lai sẽ có thêm ZaloPay, Apple Pay...*

**Giải quyết vấn đề bằng OOP:**
- **Encapsulation:** Mỗi cổng thanh toán giữ kín thông tin API key, secret key bên trong.
- **Abstraction (Interface):** Tạo hợp đồng `ThanhToan` chung, hệ thống chính không cần biết ruột gan từng cổng.
- **Polymorphism:** Hệ thống chỉ gọi `thanhToan.thanhToan(soTien)` mà không cần if-else kiểm tra loại.
- **Inheritance:** Nếu các cổng có logic chung (logging, retry...), gom vào Abstract Class cha.

**Ví dụ thực tiễn:**

```java
// Service xử lý đơn hàng (Không hề biết cụ thể cổng nào)
public class DonHangService {
    private final ThanhToan congThanhToan;  // Chỉ biết Interface

    // Dependency Injection: Tiêm cổng thanh toán vào lúc runtime
    public DonHangService(ThanhToan congThanhToan) {
        this.congThanhToan = congThanhToan;
    }

    public void xuLyDonHang(double tongTien) {
        System.out.println("Đang xử lý đơn hàng...");
        
        // Polymorphism: Gọi CÙNG 1 hàm, nhưng kết quả tùy vào Object thật
        boolean ketQua = congThanhToan.thanhToan(tongTien);
        
        if (ketQua) {
            System.out.println("✅ Thanh toán thành công qua " 
                + congThanhToan.getLoaiThanhToan());
        }
    }
}

// Khi dùng:
public class Main {
    public static void main(String[] args) {
        // Khách chọn Momo
        DonHangService service1 = new DonHangService(new ThanhToanMomo());
        service1.xuLyDonHang(150_000);

        // Khách khác chọn VNPay (Không sửa 1 dòng code nào trong DonHangService!)
        DonHangService service2 = new DonHangService(new ThanhToanVNPay());
        service2.xuLyDonHang(300_000);
    }
}
```

**Giá trị mang lại:** Khi tháng sau thêm ZaloPay, bạn chỉ cần tạo class `ThanhToanZaloPay implements ThanhToan`. Không sửa bất kỳ code cũ nào. Đây chính là sức mạnh của OOP + nguyên tắc Open/Closed (SOLID). Và đây cũng chính xác là cách **Spring Framework Dependency Injection** hoạt động!

---

## 6. Tổng hợp các chủ đề OOP (Bản đồ tra cứu)

### 🌟 6.1. Nhóm ƯU TIÊN PHẢI BIẾT TRƯỚC (Bắt buộc)
1. **4 trụ cột OOP:** Encapsulation, Inheritance, Polymorphism, Abstraction. Phải giải thích được từng cái bằng ví dụ.
2. **Interface vs Abstract Class:** Khi nào dùng cái nào? (Câu phỏng vấn kinh điển).
3. **Method Overriding vs Overloading:** Phân biệt rõ ràng compile-time vs runtime polymorphism.
4. **Access Modifiers:** `private`, `default` (package-private), `protected`, `public`. Hiểu phạm vi truy cập của từng loại.

### 🟡 6.2. Nhóm Trung cấp
1. **`this` vs `super`:** Phân biệt: `this` tham chiếu Object hiện tại, `super` tham chiếu Class cha.
2. **`final` class/method:** `final class` không thể bị kế thừa (VD: `String`). `final method` không thể bị override.
3. **Interface `default` method (Java 8+):** Interface có thể có hàm có thân sẵn. Giúp thêm tính năng mới vào Interface mà không phá vỡ các class cũ đã implement.
4. **Composition over Inheritance:** Nguyên tắc "Ưu tiên chứa hơn kế thừa". Thay vì `class A extends B`, hãy cho `class A` chứa 1 biến kiểu `B`.

### 🔴 6.3. Nhóm Nâng cao
1. **SOLID Principles:** 5 nguyên tắc thiết kế hướng đối tượng cốt lõi. Nền tảng để viết code sạch và dễ bảo trì.
2. **Sealed Classes (Java 17+):** Giới hạn danh sách class con được phép kế thừa. Kết hợp với Pattern Matching cực mạnh.
3. **Covariant Return Types:** Hàm Override trong class con được phép trả về kiểu con (subtype) của kiểu trả về trong class cha.
4. **Diamond Problem & Interface Conflicts:** Khi 1 class implement 2 interface có cùng tên `default` method, Java giải quyết xung đột thế nào?

## 📚 Tài liệu tham khảo chính thống
| Tài liệu | Link |
|-----------|------|
| Oracle - Object-Oriented Concepts | [docs.oracle.com](https://docs.oracle.com/javase/tutorial/java/concepts/index.html) |
| Oracle - Interfaces and Inheritance | [docs.oracle.com](https://docs.oracle.com/javase/tutorial/java/IandI/index.html) |
| Baeldung - OOP in Java | [baeldung.com](https://www.baeldung.com/java-oop) |
| Dev.java - Classes & Objects | [dev.java](https://dev.java/learn/classes-objects/) |