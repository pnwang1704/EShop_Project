# 🛒 eShop – Shopping Cart (Spring Boot 3 + JPA + Thymeleaf + MariaDB)

Dự án ứng dụng thương mại điện tử **eShop – Shopping Cart** được xây dựng bằng **Spring Boot 3.2.12**, **Spring Data JPA (Hibernate)**, **Thymeleaf**, **Bean Validation** và **MariaDB**.

---

## 🚀 Công nghệ sử dụng

| Thành phần | Phiên bản / Công nghệ |
| :--- | :--- |
| **Ngôn ngữ** | Java 17 (LTS) |
| **Framework** | Spring Boot 3.2.12 (Spring MVC, Spring Data JPA, Validation) |
| **Template Engine** | Thymeleaf 3.1 + Bootstrap 5.3.3 |
| **Cơ sở dữ liệu** | MariaDB 10.6+ / 11.x |
| **Build Tool** | Apache Maven 3.9+ |
| **Tiện ích** | Project Lombok, Spring Boot DevTools |

---

## 📂 Cấu trúc dự án

```text
eshop/
├── pom.xml
├── database/
│   └── eShop.sql
└── src/main/
    ├── java/com/example/eshop/
    │   ├── EshopApplication.java
    │   ├── entity/          Category, Product, Order, OrderDetail, OrderStatus, PaymentMethod
    │   ├── repository/      CategoryRepository, ProductRepository, OrderRepository
    │   ├── exception/       NotFoundException
    │   ├── dto/             CheckoutForm
    │   ├── cart/            Cart, CartItem (@SessionScope)
    │   ├── service/         CategoryService, ProductService, OrderService
    │   ├── controller/      ShopController, CartController, CheckoutController,
    │   │                    OrderController, GlobalModelAttributes
    │   └── controller/admin/
    │                        AdminCategoryController, AdminProductController,
    │                        AdminOrderController
    └── resources/
        ├── application.properties
        ├── static/css/style.css
        └── templates/
            ├── fragments/layout.html
            ├── shop/            index.html, detail.html
            ├── admin/category/  list.html, form.html
            ├── admin/product/   list.html, form.html
            ├── admin/order/     list.html, detail.html
            ├── cart/            view.html
            ├── checkout/        form.html
            └── order/           payment.html, detail.html
```

---

## ✨ Tính năng chính

### Phần 1 – CRUD với Thymeleaf & Trang Cửa hàng
- **Trang cửa hàng (`/`, `/products`)**:
  - Hiển thị danh sách sản phẩm phân trang (12 sản phẩm/trang).
  - Tìm kiếm sản phẩm theo từ khoá (không phân biệt hoa thường).
  - Lọc sản phẩm theo danh mục kết hợp tìm kiếm.
  - Xem chi tiết sản phẩm (`/products/{id}`).
- **Quản trị Danh mục (`/admin/categories`)**:
  - Xem danh sách, thêm mới, cập nhật, xoá danh mục.
  - Kiểm tra trùng tên danh mục và chặn xoá danh mục đang có sản phẩm.
- **Quản trị Sản phẩm (`/admin/products`)**:
  - Xem danh sách có tìm kiếm, lọc theo danh mục và phân trang (10 sản phẩm/trang).
  - Thêm mới, chỉnh sửa, xoá sản phẩm kèm kiểm tra dữ liệu đầu vào (`@Valid`) và ràng buộc khoá ngoại với đơn hàng.

### Phần 2 – Giỏ hàng (`@SessionScope`) & Thanh toán Đơn hàng
- **Giỏ hàng trong Session (`/cart`)**:
  - Quản lý giỏ hàng độc lập cho từng phiên trình duyệt (`@SessionScope`).
  - Thêm sản phẩm vào giỏ, cập nhật số lượng (kiểm tra tồn kho thực tế), xoá từng mặt hàng hoặc xoá toàn bộ giỏ.
  - Hiển thị tổng số lượng sản phẩm trực tiếp trên thanh điều hướng (`@ControllerAdvice` – `GlobalModelAttributes`).
- **Đặt hàng & Thanh toán (`/checkout`, `/orders`)**:
  - Xác thực thông tin người nhận (`CheckoutForm`: họ tên, số điện thoại 10 số bắt đầu bằng `0`, địa chỉ, phương thức thanh toán).
  - Hỗ trợ 2 phương thức thanh toán:
    - **COD**: Thanh toán khi nhận hàng.
    - **BANKING**: Chuyển khoản ngân hàng giả lập qua QR Code (`/orders/{id}/payment`) và xác nhận thanh toán (`NEW` → `PAID`).
  - Xử lý đặt hàng toàn vẹn trong một giao dịch (`@Transactional`): kiểm tra tồn kho, trừ tồn kho và lưu `orders` + `order_detail` (rollback toàn bộ nếu bất kỳ sản phẩm nào không đủ tồn kho).
- **Quản trị Đơn hàng (`/admin/orders`)**:
  - Xem danh sách và chi tiết đơn hàng.
  - Cập nhật trạng thái đơn hàng (`NEW`, `PAID`, `SHIPPING`, `DONE`, `CANCELLED`). Khi chuyển sang `CANCELLED`, hệ thống tự động hoàn lại số lượng tồn kho cho các sản phẩm trong đơn.

---

## ⚙️ Hướng dẫn cài đặt và chạy dự án

### 1. Khởi tạo cơ sở dữ liệu MariaDB
Chạy toàn bộ script SQL trong file `database/eShop.sql` (hoặc `eShop.sql` ở thư mục gốc) trên MariaDB (`localhost:3306`):
- Tạo database `eshop` (`utf8mb4_unicode_ci`).
- Khởi tạo dữ liệu mẫu: **15 danh mục** (`category`) và **50 sản phẩm** (`product`).

### 2. Cấu hình kết nối trong `application.properties`
Mở file `src/main/resources/application.properties` và cập nhật mật khẩu MariaDB của bạn:

```properties
spring.datasource.url=jdbc:mariadb://localhost:3306/eshop
spring.datasource.username=root
spring.datasource.password=<điền_mật_khẩu_mariadb_của_bạn>
```

### 3. Biên dịch và chạy ứng dụng
```bash
# Kiểm tra biên dịch
mvn clean compile

# Chạy ứng dụng
mvn spring-boot:run
```

Sau khi ứng dụng khởi động, truy cập:
- **Trang cửa hàng:** [http://localhost:8080](http://localhost:8080)
- **Quản trị danh mục:** [http://localhost:8080/admin/categories](http://localhost:8080/admin/categories)
- **Quản trị sản phẩm:** [http://localhost:8080/admin/products](http://localhost:8080/admin/products)
- **Quản trị đơn hàng:** [http://localhost:8080/admin/orders](http://localhost:8080/admin/orders)
