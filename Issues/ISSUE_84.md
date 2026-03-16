# **US-INV-BONUS: Điều chỉnh Tồn kho (Inventory Adjustment) – 8 Points**

---

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Trong quá trình vận hành kho, số lượng tồn kho thực tế có thể **không khớp với số liệu trong hệ thống** do nhiều nguyên nhân:

- Nhập kho sai số lượng

- Hư hỏng nguyên vật liệu

- Mất mát trong quá trình bảo quản

- Sai sót khi cân đo

- Sai khi nhập phiếu kho trước đó

Nếu không có cơ chế điều chỉnh:

- Tồn kho hệ thống sẽ **không phản ánh đúng thực tế**

- Các món ăn có thể bị **báo hết hàng sai**

- Báo cáo kho sẽ **không chính xác**

Do đó hệ thống cần chức năng **Điều chỉnh tồn kho (Inventory Adjustment)**.

---

## **Pain Points**

- Không thể sửa sai khi nhập kho nhầm số lượng

- Không có cách xử lý NVL bị hư hỏng

- Không có log giải thích vì sao tồn kho thay đổi

- Khó kiểm kê kho thực tế

---

## **Giá trị Nghiệp vụ (Business Value)**

### **1. Đồng bộ tồn kho với thực tế**

Cho phép Manager cập nhật lại số lượng NVL khi kiểm kê kho.

---

### **2. Lưu lịch sử thay đổi**

Mỗi điều chỉnh đều được ghi lại trong **InventoryTransaction**.

---

### **3. Hỗ trợ kiểm kê định kỳ**

Nhà hàng có thể kiểm kê kho cuối ngày hoặc cuối tuần.

---

### **4. Đảm bảo vận hành menu chính xác**

Khi tồn kho thay đổi, hệ thống có thể **đóng/mở món ăn tự động**.

---

## **Actors**

### **Primary Actor**

**Manager**

- Tạo phiếu điều chỉnh tồn kho

- Xem danh sách phiếu điều chỉnh

- Xem chi tiết phiếu

---

### **Secondary Actor**

**System**

- Cập nhật tồn kho

- Ghi InventoryTransaction

- Kiểm tra trạng thái MenuItem

---

# **User Story Statement**

Là **Manager**, tôi muốn **điều chỉnh tồn kho nguyên vật liệu**, để **đảm bảo số lượng tồn kho trong hệ thống khớp với thực tế**.

---

# **Tiền điều kiện (Preconditions)**

- Danh mục **Ingredient** đã tồn tại

- Hệ thống đã có bảng **InventoryTransaction**

---

# **Entity liên quan**

- InventoryAdjustment

- InventoryAdjustmentItem

- Ingredient

- InventoryTransaction

- MenuItemIngredient

---

# **2. Luồng Người dùng (User Flow)**

---

# **2.1 Xem danh sách phiếu điều chỉnh**

Manager truy cập:

/inventory/adjustments

Hệ thống hiển thị danh sách:

\[table\]

Chức năng:

- Tìm kiếm theo mã phiếu

- Lọc theo ngày

- Phân trang

---

# **2.2 Tạo phiếu điều chỉnh**

Manager bấm:

\+ Điều chỉnh tồn kho

---

## **Thông tin chung**

- Mã phiếu (tự sinh)

ADJ-yyyymmdd-0001

- Ngày điều chỉnh

- Lý do điều chỉnh

- Ghi chú

---

## **Danh sách NVL**

Mỗi dòng gồm:

\[table\]

---

### **Ví dụ**

\[table\]

---

Manager bấm **Lưu**

---

# **2.3 Hệ thống xử lý**

Hệ thống:

### **1️⃣ Tạo phiếu**

InventoryAdjustment

InventoryAdjustmentItems

---

### **2️⃣ Cập nhật tồn kho**

Ingredient.CurrentStock += AdjustmentQuantity

---

### **3️⃣ Ghi transaction**

InventoryTransaction

Type:

ADJUSTMENT

Ví dụ:

\[table\]

---

### **4️⃣ Kiểm tra lại menu**

Nếu NVL tăng đủ:

MenuItem.IsOutOfStock = false

---

# **3. Ví dụ thực tế**

Kho hiện tại:

Beef = 7000g

Kiểm kê thực tế:

6000g

Manager tạo:

ADJ-001

Beef -1000g

Reason: Inventory count

Sau khi lưu:

CurrentStock = 6000g

---

# **4. Acceptance Criteria**

---

## **AC-01: Tạo phiếu điều chỉnh thành công**

**Given**

Thịt bò tồn kho:

7000g

**When**

Manager điều chỉnh:

\-1000g

**Then**

CurrentStock = 6000g

và ghi:

InventoryTransaction

Type = ADJUSTMENT

---

## **AC-02: Điều chỉnh không được làm tồn kho âm**

**Given**

Thịt bò:

CurrentStock = 500g

**When**

Manager điều chỉnh:

\-600g

**Then**

Hệ thống báo lỗi:

"Tồn kho không đủ để điều chỉnh"

---

## **AC-03: Phiếu phải có ít nhất 1 NVL**

Nếu Manager bấm **Lưu** mà chưa thêm dòng:

"Phải có ít nhất 1 nguyên liệu"

# **5. Transaction Log ví dụ**

\[table\]
