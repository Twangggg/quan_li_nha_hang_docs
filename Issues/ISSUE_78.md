# **US-INV-01: Quản lý Danh mục Nguyên liệu (Ingredient Management) – 13 Points**

---

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Để hệ thống quản lý kho hoạt động, cần có **danh mục nguyên vật liệu (NVL)** làm dữ liệu nền tảng cho các nghiệp vụ như:

- Nhập kho

- Xuất kho

- Tính giá vốn

- Cảnh báo tồn kho

- Kiểm soát nguyên liệu trong công thức món ăn (recipe)

Nếu không có danh mục nguyên liệu chuẩn:

- Không thể thực hiện nhập/xuất kho

- Không thể tính tồn kho

- Không thể kiểm soát tình trạng thiếu nguyên liệu cho món ăn.

---

## **Pain Points hiện tại**

- Không có màn hình quản lý danh mục nguyên liệu tập trung.

- Không thể thiết lập ngưỡng cảnh báo tồn kho cho từng NVL.

- Không có cơ chế vô hiệu hóa nguyên liệu không còn sử dụng.

- Dữ liệu nguyên liệu có thể bị trùng lặp hoặc không đồng nhất.

---

## **Giá trị Nghiệp vụ (Business Value)**

### **1. Chuẩn hóa dữ liệu nguyên liệu**

Tất cả NVL được quản lý tập trung trong một danh mục thống nhất.

### **2. Là nền tảng cho toàn bộ nghiệp vụ kho**

Danh mục NVL là dữ liệu đầu vào cho:

- Phiếu nhập kho

- Phiếu xuất kho

- Tính tồn kho

- Cảnh báo tồn kho.

### **3. Kiểm soát trạng thái nguyên liệu**

Cho phép vô hiệu hóa NVL không còn sử dụng để tránh sai sót khi tạo phiếu nhập/xuất.

### **4. Hỗ trợ cảnh báo tồn kho**

Thiết lập **ngưỡng cảnh báo sắp hết** giúp hệ thống phát hiện NVL sắp hết hàng.

---

## **Đối tượng (Actor)**

### **Primary Actor**

**Manager**

- Tạo nguyên liệu mới

- Xem danh sách nguyên liệu

- Cập nhật thông tin nguyên liệu

- Vô hiệu hóa nguyên liệu

### **Secondary Actors**

**System**

- Lấy giá trị mặc định từ InventorySettings.DefaultLowStockThreshold

- Tính trạng thái tồn kho của NVL

- Kiểm tra NVL có đang được sử dụng trong recipe hay không

---

## **User Story Statement**

Là **Manager**, tôi muốn **khai báo và quản lý danh mục nguyên liệu (tên, đơn vị, ngưỡng cảnh báo)**, để **có dữ liệu nền tảng cho toàn bộ nghiệp vụ quản lý kho**.

---

## **Entity liên quan**

- **Ingredient**

- **InventorySettings**

---

# **2. Luồng Người dùng (User Flow)**

---

# **2.1 Luồng: Xem danh sách nguyên liệu**

1. Manager đăng nhập hệ thống

2. Điều hướng đến:

Quản lý Kho → Danh mục Nguyên liệu

/inventory/ingredients

3. Hệ thống hiển thị bảng danh sách nguyên liệu với các cột:

- Mã NVL

- Tên nguyên liệu

- Đơn vị tính (gram, kg, ml, lít, cái,...)

- Tồn kho hiện tại

- Ngưỡng cảnh báo

- Trạng thái tồn kho

- Active / Inactive

---

### **Trạng thái tồn kho được tính như sau**

Đủ hàng: CurrentStock &gt; LowStockThreshold

Sắp hết: 0 &lt; CurrentStock ≤ LowStockThreshold

Hết hàng: CurrentStock = 0

---

4. Manager có thể sử dụng các chức năng:

### **Tìm kiếm**

- Theo **Tên NVL**

- Theo **Mã NVL**

- Debounce 300ms

### **Lọc**

- Theo trạng thái tồn kho

  - Đủ hàng

  - Sắp hết

  - Hết hàng

### **Phân trang**

- Mặc định: 20 dòng

- Tùy chọn: 20 / 50 / 100

---

# **2.2 Luồng: Tạo nguyên liệu mới**

1. Manager bấm:

\+ Thêm nguyên liệu

2. Hệ thống hiển thị form tạo nguyên liệu gồm:

### **Thông tin nguyên liệu**

- **Mã NVL**

  - Có thể tự sinh hoặc nhập tay

  - Phải unique

- **Tên nguyên liệu**

  - Bắt buộc

  - Unique (không phân biệt hoa thường)

- **Đơn vị tính**

  - Bắt buộc

  - Ví dụ: gram, kg, ml, lít, cái

- **Ngưỡng cảnh báo sắp hết**

  - Bắt buộc

  - ≥ 0

  - Default = InventorySettings.DefaultLowStockThreshold

- **Mô tả / Ghi chú**

  - Tùy chọn

---

3. Manager bấm **"Lưu"**

4. Hệ thống validate dữ liệu.

5. Nếu hợp lệ:

Hệ thống tạo Ingredient mới:

CurrentStock = 0

CostPrice = 0

6. Hiển thị thông báo:

"Tạo nguyên liệu thành công"

7. Danh sách nguyên liệu được refresh.

---

# **2.3 Luồng: Cập nhật nguyên liệu**

1. Manager bấm **"Chỉnh sửa"** trên dòng NVL.

2. Hệ thống hiển thị form chỉnh sửa.

3. Các trường có thể chỉnh sửa:

- Tên nguyên liệu

- Đơn vị tính

- Ngưỡng cảnh báo

- Mô tả

- Trạng thái Active / Inactive

---

4. Các trường **không cho chỉnh sửa trực tiếp**:

CurrentStock

CostPrice

Hai trường này chỉ thay đổi thông qua:

- Phiếu nhập kho

- Phiếu xuất kho

---

5. Khi lưu thành công:

- Cập nhật Ingredient

- Ghi:

updatedBy

updatedAt

---

# **2.4 Luồng: Vô hiệu hóa nguyên liệu**

1. Manager bấm **"Vô hiệu"** trên một nguyên liệu.

2. Hệ thống kiểm tra nguyên liệu có đang được sử dụng trong recipe hay không.

---

### **Trường hợp 1: NVL đang được sử dụng**

Ví dụ:

Bột mì → dùng trong recipe "Bánh mì"

Hệ thống hiển thị cảnh báo:

"Nguyên liệu đang được sử dụng trong định lượng của món: Bánh mì"

→ Chặn thao tác vô hiệu hóa.

---

### **Trường hợp 2: NVL không được sử dụng**

Hệ thống:

Active = false

Nguyên liệu sẽ:

- Không hiển thị trong dropdown chọn NVL

- Không thể dùng cho phiếu nhập/xuất mới.

---

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

---

# **AC-01: Hiển thị danh sách nguyên liệu**

**Given**

Manager đã đăng nhập hệ thống.

**When**

Truy cập:

/inventory/ingredients

**Then**

- Hiển thị danh sách nguyên liệu với các cột:

Mã NVL

Tên NVL

Đơn vị

Tồn kho

Ngưỡng cảnh báo

Trạng thái

- Có chức năng:

  - Tìm kiếm

  - Lọc

  - Phân trang

---

# **AC-02: Tạo nguyên liệu thành công**

**Given**

Manager đang ở trang **Danh mục nguyên liệu**

**When**

Bấm **Thêm nguyên liệu**, nhập:

- Tên nguyên liệu unique

- Đơn vị hợp lệ

- Ngưỡng cảnh báo hợp lệ

và bấm **Lưu**

**Then**

- Ingredient mới được tạo

- Giá trị mặc định:

CurrentStock = 0

CostPrice = 0

- Ngưỡng cảnh báo mặc định lấy từ:

InventorySettings.DefaultLowStockThreshold

- Hiển thị toast:

"Tạo nguyên liệu thành công"

---

# **AC-03: Tạo nguyên liệu thất bại — tên trùng**

**Given**

Hệ thống đã có nguyên liệu:

Thịt bò

**When**

Manager tạo nguyên liệu mới:

thịt bò

**Then**

Hệ thống hiển thị lỗi:

"Tên nguyên liệu đã tồn tại"

và không tạo bản ghi mới.

---

# **AC-04: Cập nhật nguyên liệu**

**Given**

Manager bấm **Chỉnh sửa** một nguyên liệu.

**When**

Manager thay đổi:

Ngưỡng cảnh báo

và bấm **Lưu**

**Then**

- Ingredient được cập nhật

- Hệ thống ghi:

updatedBy

updatedAt

- Không cho phép chỉnh sửa:

CurrentStock

CostPrice

---

# **AC-05: Không cho vô hiệu nguyên liệu đang sử dụng**

**Given**

Nguyên liệu:

Bột mì

đang được sử dụng trong recipe:

Bánh mì

**When**

Manager bấm **Vô hiệu**

**Then**

Hệ thống hiển thị cảnh báo:

"Nguyên liệu đang được sử dụng trong định lượng của món: Bánh mì"

và **chặn thao tác vô hiệu hóa**.