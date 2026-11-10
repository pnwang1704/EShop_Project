# 🛒 eShop – Shopping Cart (Spring Boot 3 + JPA + Thymeleaf + Spring Security 6 + JWT + MariaDB)

Dự án ứng dụng thương mại điện tử **eShop – Shopping Cart** được xây dựng hoàn chỉnh qua 3 phần (**Phần 1: CRUD & Storefront**, **Phần 2: Session Cart & Checkout/Orders**, **Phần 3: Spring Security 6 + JWT + Phân quyền 3 vai trò + REST API**) bằng **Spring Boot 3.2.12**, **Spring Data JPA (Hibernate)**, **Thymeleaf**, **Spring Security 6**, **JJWT (`0.12.6`)** và **MariaDB**.

---

## 🚀 Công nghệ sử dụng

| Thành phần | Phiên bản / Công nghệ |
| :--- | :--- |
| **Ngôn ngữ** | Java 17 (LTS) |
| **Framework** | Spring Boot 3.2.12 (Spring MVC, Spring Data JPA, Validation, Spring Security 6) |
| **Bảo mật & Xác thực** | Spring Security 6 + BCrypt + JWT (`io.jsonwebtoken:jjwt 0.12.6` – `HMAC-SHA`) |
| **Template Engine** | Thymeleaf 3.1 + `thymeleaf-extras-springsecurity6` + Bootstrap 5.3.3 |
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
    │   ├── entity/          Category, Product, Order, OrderDetail, OrderStatus,
    │   │                    PaymentMethod, User, Role
    │   ├── repository/      CategoryRepository, ProductRepository, OrderRepository,
    │   │                    UserRepository
    │   ├── exception/       NotFoundException
    │   ├── dto/             CheckoutForm, RegisterForm, UserForm
    │   ├── cart/            Cart, CartItem (@SessionScope)
    │   ├── security/        CustomUserDetailsService, JwtService,
    │   │                    JwtAuthenticationFilter, SecurityConfig
    │   ├── service/         CategoryService, ProductService, OrderService, UserService
    │   ├── controller/      ShopController, CartController, CheckoutController,
    │   │                    OrderController, AuthController, ErrorPageController,
    │   │                    GlobalModelAttributes
    │   ├── controller/admin/
    │   │                    AdminCategoryController, AdminProductController,
    │   │                    AdminOrderController, AdminUserController
    │   └── api/             AuthApiController, ProductApiController, ApiExceptionHandler
    └── resources/
        ├── application.properties
        ├── static/css/style.css
        └── templates/
            ├── fragments/layout.html
            ├── shop/            index.html, detail.html
            ├── auth/            login.html, register.html
            ├── error/           403.html
            ├── admin/category/  list.html, form.html
            ├── admin/product/   list.html, form.html
            ├── admin/order/     list.html, detail.html
            ├── admin/user/      list.html, form.html
            ├── cart/            view.html
            ├── checkout/        form.html
            └── order/           payment.html, detail.html, my-orders.html
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
  - Tự động điền thông tin người nhận (`fullName`, `phone`, `address`) từ hồ sơ khách hàng đang đăng nhập và xác thực dữ liệu (`CheckoutForm`).
  - Hỗ trợ 2 phương thức thanh toán:
    - **COD**: Thanh toán khi nhận hàng.
    - **BANKING**: Chuyển khoản ngân hàng giả lập qua QR Code (`/orders/{id}/payment`) và xác nhận thanh toán (`NEW` → `PAID`).
  - Xử lý đặt hàng toàn vẹn trong một giao dịch (`@Transactional`): kiểm tra tồn kho, trừ tồn kho và lưu `orders` + `order_detail` gắn với `user_id`.
- **Quản trị Đơn hàng (`/admin/orders`)**:
  - Xem danh sách và chi tiết đơn hàng.
  - Cập nhật trạng thái đơn hàng (`NEW`, `PAID`, `SHIPPING`, `DONE`, `CANCELLED`). Khi chuyển sang `CANCELLED`, hệ thống tự động hoàn lại số lượng tồn kho cho các sản phẩm trong đơn.

### Phần 3 – Spring Security 6 + JWT + Phân quyền 3 vai trò & REST API
- **Kiến trúc 2 chuỗi `SecurityFilterChain`**:
  - **`apiFilterChain` (`@Order(1)`, `/api/**`)**: Stateless hoàn toàn, tắt CSRF, xác thực qua header `Authorization: Bearer <token>`, trả mã `401 Unauthorized` dạng JSON khi chưa xác thực.
  - **`webFilterChain` (`@Order(2)`, `/**`)**: Xác thực qua `HttpOnly` cookie `ACCESS_TOKEN` kết hợp bảo vệ CSRF cho các biểu mẫu Thymeleaf, gán `ROLE_GUEST` cho người dùng ẩn danh (`anonymous`), tự động xoá cookie `ACCESS_TOKEN` và `JSESSIONID` khi đăng xuất (`POST /logout`).
- **Bảo vệ đa tầng**:
  - Phân quyền URL trên `SecurityFilterChain` + ẩn/hiện giao diện theo vai trò với `sec:authorize`.
  - Bảo vệ tầng nghiệp vụ với `@EnableMethodSecurity` (`@PreAuthorize` trên `CategoryService`, `ProductService`, `OrderService`, `UserService`).
  - Kiểm tra quyền sở hữu đơn hàng (`OrderController.getOwnedOrder`): khách hàng chỉ được xem/thanh toán đơn hàng của chính mình.
- **Quản trị Tài khoản (`/admin/users`)**:
  - Dành riêng cho `SELLER`: tìm kiếm, lọc theo vai trò (`ROLE_SELLER`, `ROLE_CUSTOMER`), thêm mới, chỉnh sửa, khoá/mở khoá và xoá tài khoản (chặn tự xoá hoặc tự khoá tài khoản đang đăng nhập).

---

## 🔐 Ma trận phân quyền & Tài khoản mẫu

### Ma trận phân quyền 3 vai trò

| Chức năng / Đường dẫn | `GUEST` (Chưa đăng nhập) | `CUSTOMER` (Khách hàng) | `SELLER` (Người bán / Quản trị) |
| :--- | :---: | :---: | :---: |
| Xem danh sách, tìm kiếm, lọc, chi tiết sản phẩm (`/`, `/products/**`) | ✅ | ✅ | ✅ |
| Đăng nhập, Đăng ký (`/login`, `/register`) | ✅ | ❌ | ❌ |
| Thêm vào giỏ & Quản lý giỏ hàng (`/cart/**`) | ✅ | ✅ | ❌ |
| Đặt hàng, Thanh toán & Đơn hàng của tôi (`/checkout/**`, `/orders/**`) | ❌ *(Chuyển tới `/login`)* | ✅ *(Chỉ đơn của mình)* | ❌ |
| Quản trị Danh mục, Sản phẩm, Đơn hàng, Tài khoản (`/admin/**`) | ❌ | ❌ *(403)* | ✅ |
| REST API đọc sản phẩm (`GET /api/products/**`) | ✅ | ✅ | ✅ |
| REST API thêm/sửa/xoá sản phẩm (`POST/PUT/DELETE /api/products/**`) | ❌ *(401)* | ❌ *(403)* | ✅ |

### Danh sách tài khoản mẫu (trong `database/eShop.sql`)

| Username | Mật khẩu | Vai trò (`role`) | Trạng thái (`enabled`) | Ghi chú |
| :--- | :--- | :--- | :---: | :--- |
| `seller1` | `123456` | `ROLE_SELLER` | `1` (Hoạt động) | Quản trị viên chính |
| `seller2` | `123456` | `ROLE_SELLER` | `1` (Hoạt động) | Nhân viên bán hàng |
| `customer1` | `123456` | `ROLE_CUSTOMER` | `1` (Hoạt động) | Có sẵn 2 đơn hàng mẫu (`#1`, `#2`) |
| `customer2` | `123456` | `ROLE_CUSTOMER` | `1` (Hoạt động) | Có sẵn 1 đơn hàng mẫu (`#3`) |
| `locked` | `123456` | `ROLE_CUSTOMER` | `0` (Đã khoá) | Dùng để kiểm thử chặn đăng nhập tài khoản bị khoá |

---

## 🌐 Hướng dẫn kiểm thử REST API (JWT Bearer Token)

### 1. Đăng nhập lấy JWT (`POST /api/auth/login`)
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"seller1\",\"password\":\"123456\"}"
```

### 2. Danh sách / Chi tiết sản phẩm (Public – `GET /api/products`)
```bash
curl "http://localhost:8080/api/products?q=ban%20phim&page=0&size=5"
curl "http://localhost:8080/api/products/1"
```

### 3. Thêm / Cập nhật / Xoá sản phẩm (`ROLE_SELLER` – Bearer Token)
```bash
curl -X POST http://localhost:8080/api/products \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d "{\"name\":\"Chuột Gaming Pro\",\"price\":550000,\"quantity\":25,\"description\":\"Chuột không dây\",\"imageFile\":\"mouse.jpg\",\"category\":{\"id\":2}}"
```

---

## ⚙️ Hướng dẫn cài đặt và chạy dự án

### 1. Khởi tạo cơ sở dữ liệu MariaDB
Chạy toàn bộ script SQL trong file `database/eShop.sql` trên MariaDB (`localhost:3306`):
- Tạo database `eshop` (`utf8mb4_unicode_ci`).
- Khởi tạo dữ liệu mẫu: **15 danh mục** (`category`), **50 sản phẩm** (`product`), **5 tài khoản mẫu** (`users`) và **3 đơn hàng mẫu** (`orders`, `order_detail`).

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
- **Đăng nhập / Đăng ký:** [http://localhost:8080/login](http://localhost:8080/login) | [http://localhost:8080/register](http://localhost:8080/register)
- **Đơn hàng của tôi (`CUSTOMER`):** [http://localhost:8080/orders/my](http://localhost:8080/orders/my)
- **Quản trị danh mục (`SELLER`):** [http://localhost:8080/admin/categories](http://localhost:8080/admin/categories)
- **Quản trị sản phẩm (`SELLER`):** [http://localhost:8080/admin/products](http://localhost:8080/admin/products)
- **Quản trị đơn hàng (`SELLER`):** [http://localhost:8080/admin/orders](http://localhost:8080/admin/orders)
- **Quản trị tài khoản (`SELLER`):** [http://localhost:8080/admin/users](http://localhost:8080/admin/users)
