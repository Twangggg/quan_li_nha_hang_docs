## **1. Tổng quan & Bối cảnh (Overview & Context)**

### **Vấn đề (Problem)**

Hiện tại, hệ thống FoodHub cần một màn hình quản lý sơ đồ bàn trực quan để:

- Hiển thị bố cục bàn theo từng tầng (Tầng 1, Tầng 2, VIP,..).

- Cho phép Manager tạo và cấu hình bàn theo mặt bằng thực tế.

- Cho phép Waiter/Cashier xem bố trí bàn để phục vụ vận hành.

Hệ thống cần đảm bảo mỗi bàn được quản lý tập trung, có mã chuẩn hóa và dùng thống nhất trong các luồng Order và Billing.

### **Pain Points hiện tại**

- Không có màn hình layout chuẩn → nhân viên dễ nhầm vị trí bàn.

- Bàn không được chuẩn hóa mã → khó tracking theo tầng.

- Việc tạo Order phải chọn bàn thủ công → mất thao tác.

- Không kiểm soát được bàn đang tạm ngưng hoạt động.

### **Giá trị Nghiệp vụ (Business Value)**

- Chuẩn hóa quản lý bàn theo tầng → giảm sai sót khi phục vụ.

- Tạo Order trực tiếp từ bàn → tăng tốc độ thao tác.

- Manager chủ động thay đổi mặt bằng ngay trên hệ thống.

- Đồng bộ dữ liệu bàn với Order & Billing.

## **2. Đối tượng (Actor)**

### **Primary Actor**

**Manager**

- Tạo, chỉnh sửa, di chuyển, kích hoạt/vô hiệu hóa bàn.

### **Secondary Actors**

**Cashier / Waiter**

- Xem layout.

- Tạo Order từ bàn (nếu bàn ACTIVE).

**Chef-Bar**

- Chỉ xem layout (read-only).

## **3. User Story Statement**

Là Manager, tôi muốn quản lý sơ đồ bàn theo từng tầng và cho phép tạo Order trực tiếp từ bàn, để phản ánh đúng mặt bằng nhà hàng và hỗ trợ vận hành nhanh chóng.

## **4. Luồng Người dùng (User Flow)**

### **4.1. Luồng chính: Xem sơ đồ bàn**

1. User đăng nhập vào hệ thống.

2. Truy cập trang

3. Hệ thống hiển thị:
   - Tabs chọn tầng (Tầng 1, Tầng 2, VIP).

   - Layout dạng grid đơn giản.

   - Mỗi bàn hiển thị:
     - Mã bàn

     - Số ghế

     - Hình dạng (tròn/chữ nhật)

     - Trạng thái: ACTIVE / INACTIVE (có thể là có màu là active, màu xám mờ là inactive tùy người làm UI)

4. Cashier/Waiter chỉ được xem, không có nút chỉnh sửa.

### **4.2. Luồng: Manager tạo bàn mới**

1. Manager bật chế độ “Edit Layout”.

2. Chọn một ô trống trong grid.

3. Nhập:
   - Mã bàn(Ví dụ: A01, A02)

   - Số ghế (&gt;0)

   - Hình dạng (tròn/chữ nhật)

4. Lưu thành công → bàn xuất hiện tại ô đã chọn.

### **4.3. Luồng: Chỉnh sửa bàn**

1. Manager bật Edit Layout.

2. Chọn bàn cần chỉnh sửa.

3. Có thể sửa:
   - Số ghế

   - Hình dạng

4. Không được sửa mã bàn.

5. Lưu → cập nhật ngay trên layout.

### **4.4. Luồng: Vô hiệu hóa bàn (Inactive)**

1. Manager bật Edit Layout.

2. Chọn bàn.

3. Bấm “Dừng hoạt động”.

4. Bàn chuyển trạng thái INACTIVE:
   - Tô xám

   - Không thể clicker

   - Không xuất hiện trong dropdown chọn bàn (nếu có).

### **4.5. Luồng: Tạo Order từ bàn (thuộc order)**

1. Waiter/Cashier click vào bàn ACTIVE.

2. Hệ thống hiển thị panel thông tin bàn.

3. Click “Tạo Order”.

4. Hệ thống tạo:
   - OrderType = DineIn

   - TableId = selected

   - Status = DRAFT

   - CreatedBy = current user

   - OrderCode unique theo ngày

5. Redirect sang màn Order Detail.

Nếu bàn thuộc khu VIP:

- Order.IsPriority = true.

## **4.6. Luồng: Quản lý Khu vực (Floor / Area Management)**

### **Mục tiêu**

Cho phép Manager tạo, chỉnh sửa và vô hiệu hóa khu vực (Tầng 1, Tầng 2, VIP…) để hệ thống linh hoạt khi mở rộng mặt bằng.

### **4.6.1. Luồng: Xem danh sách khu vực**

1. Manager truy cập /table-layout.

2. Hệ thống hiển thị:
   - Tabs hoặc Dropdown chọn khu vực.

   - Danh sách khu vực hiện có.

Mỗi khu vực hiển thị:

- Tên khu vực

- Mã khu vực

- Trạng thái (ACTIVE / INACTIVE)

Cashier/Waiter:

- Chỉ xem.

- Không có quyền chỉnh sửa.

### **4.6.2. Luồng: Thêm khu vực mới**

1. Manager click “Quản lý khu vực”.

2. Click “Thêm khu vực”.

3. Nhập:
   - Tên khu vực (required)

   - Mã khu vực (required, unique)

   - Loại khu vực:
     - NORMAL

     - VIP

   - Mô tả (optional)

4. Validate:
   - Tên không được trống.

   - Mã khu vực unique.

5. Lưu thành công:
   - Khu vực xuất hiện trong danh sách.

   - Có thể chọn khi tạo bàn.

### **4.6.3. Luồng: Chỉnh sửa khu vực**

1. Manager chọn khu vực.

2. Click “Chỉnh sửa”.

3. Có thể sửa:
   - Tên

   - Mô tả

4. Không cho sửa:
   - Mã khu vực (readonly).

5. Lưu → cập nhật ngay.

### **4.6.4. Luồng: Vô hiệu hóa khu vực**

1. Manager chọn khu vực.

2. Click “Dừng hoạt động”.

3. Confirm.

Hệ thống:

- Khu vực chuyển INACTIVE.

- Không cho tạo bàn mới trong khu vực này.

- Các bàn thuộc khu vực đó:
  - Giữ nguyên dữ liệu.

  - Không cho tạo Order nếu khu vực INACTIVE.

- Không cho chọn khu vực này khi đặt bàn (Reservation).

## **5. Tiêu chí Chấp nhận (Acceptance Criteria)**

### **AC-L01: Hiển thị layout theo tầng**

Given: User đăng nhập\
When: Truy cập /table-layout\
Then: Hiển thị grid bàn theo từng tầng.

### **AC-L02: Manager tạo bàn thành công**

Given: Manager bật Edit Layout\
When: Nhập dữ liệu hợp lệ và lưu\
Then: Bàn được tạo với mã đúng format và unique.

### **AC-L03: Không cho tạo bàn trùng mã**

Given: A01 đã tồn tại\
When: Tạo bàn cùng number tại Tầng 1\
Then: Hiển thị lỗi “Mã bàn đã tồn tại”.

### **AC-L04: Chỉnh sửa bàn thành công**

Given: Manager chỉnh sửa số ghế\
When: Lưu\
Then: Layout cập nhật ngay.

### **AC-L05: Vô hiệu hóa bàn**

Given: Manager chọn bàn ACTIVE\
When: Bấm “Dừng hoạt động”\
Then: Bàn chuyển INACTIVE và không dùng được cho Order.

### **AC-L06: Tạo Order từ bàn ACTIVE**

Given: Bàn T1_101 đang ACTIVE\
When: Click “Tạo Order”\
Then: Order DRAFT được tạo và chuyển sang màn Order Detail.

### **AC-L07: Không cho tạo Order từ bàn INACTIVE**

Given: Bàn INACTIVE\
When: Click tạo Order\
Then: Nút bị disable hoặc API trả lỗi 409.
