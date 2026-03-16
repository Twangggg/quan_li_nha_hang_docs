# **US-INV-05: Kiểm kê kho & Báo cáo tồn kho (13 Points)**

---

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Trong vận hành thực tế, số lượng nguyên vật liệu trong hệ thống có thể **không khớp với tồn kho thực tế** do:

- Hao hụt trong quá trình chế biến

- Sai sót khi nhập/xuất kho

- Hư hỏng, mất mát

- Lỗi thao tác hệ thống

Nếu không có quy trình **kiểm kê kho**, doanh nghiệp sẽ:

- Không phát hiện được chênh lệch tồn kho

- Không biết nguyên nhân sai lệch

- Không thể điều chỉnh tồn kho đúng thực tế

Ngoài ra, Manager cũng cần **báo cáo tổng hợp tồn kho theo thời gian** để theo dõi:

- Tổng nhập

- Tổng xuất

- Giá trị tồn kho

---

# **Giá trị nghiệp vụ (Business Value)**

### **1. Phát hiện sai lệch tồn kho**

So sánh **tồn sổ sách vs tồn thực tế**.

---

### **2. Điều chỉnh tồn kho chính xác**

Hệ thống tự sinh phiếu:

- Nhập kho kiểm kê

- Xuất kho kiểm kê

---

### **3. Báo cáo tồn kho theo thời gian**

Manager có thể xem:

- Tồn đầu kỳ

- Tổng nhập

- Tổng xuất

- Tồn cuối kỳ

---

### **4. Truy vết lịch sử biến động kho**

Có thể xem **ledger của từng nguyên liệu**.

---

# **Actor**

### **Primary Actor**

**Manager**

Có thể:

- Tạo phiếu kiểm kê

- Xử lý chênh lệch tồn kho

- Xem báo cáo tồn kho

- Xem lịch sử biến động NVL

---

### **Secondary Actor**

**System**

- Tự sinh phiếu nhập/xuất khi kiểm kê

- Ghi InventoryTransaction

- Cập nhật CurrentStock

---

# **Tiền điều kiện (Preconditions)**

- US-INV-01 hoàn thành (Ingredient tồn tại)

- US-INV-02 hoàn thành (Stock In)

- US-INV-03 hoàn thành (Stock Out)

---

# **Entity liên quan**

- InventoryCheck

- InventoryCheckItem

- Ingredient

- StockInReceipt

- StockOutReceipt

- InventoryTransaction

---

# **2. Luồng Người dùng (User Flow)**

---

# **2.1 Kiểm kê kho (Inventory Check / Stock Take)**

---

# **2.1.1 Xem danh sách phiếu kiểm kê**

Manager vào:

Quản lý Kho → Kiểm kê

/inventory/check

Hệ thống hiển thị danh sách:

\[table\]

Trạng thái gồm:

Nháp

Đã xử lý

Manager có thể:

- Lọc theo trạng thái

- Lọc theo khoảng ngày

- Phân trang

---

# **2.1.2 Tạo phiếu kiểm kê**

Manager bấm:

\+ Tạo phiếu kiểm kê

Hệ thống hiển thị bảng tất cả nguyên liệu:

\[table\]

---

### **Các trường**

**Tồn theo sổ**

Ingredient.CurrentStock

Hệ thống tự điền.

---

**Tồn thực tế**

Manager nhập tay sau khi kiểm tra kho.

---

**Chênh lệch**

Hệ thống tự tính:

Chênh lệch = Tồn thực tế − Tồn theo sổ

---

**Nguyên nhân**

Manager nhập nếu có sai lệch.

---

Manager bấm:

Lưu

Phiếu được lưu với trạng thái:

Nháp

---

# **2.1.3 Xử lý chênh lệch tồn kho**

Manager mở phiếu kiểm kê và bấm:

Xử lý chênh lệch

---

## **Trường hợp 1**

### **Thực tế &gt; sổ sách**

Ví dụ:

Sổ sách = 500g

Thực tế = 600g

Chênh lệch:

+100g

Hệ thống tự tạo:

StockInReceipt

Type = INVENTORY_ADJUSTMENT

Quantity = +100g

---

## **Trường hợp 2**

### **Thực tế &lt; sổ sách**

Ví dụ:

Sổ sách = 1000g

Thực tế = 800g

Chênh lệch:

\-200g

Hệ thống tự tạo:

StockOutReceipt

Type = INVENTORY_ADJUSTMENT

Quantity = -200g

---

## **Cập nhật tồn kho**

Sau khi xử lý:

Ingredient.CurrentStock = Tồn thực tế

---

## **Ghi giao dịch kho**

Hệ thống tạo:

InventoryTransaction

Type = INVENTORY_CHECK

Reference = InventoryCheckId

---

## **Trạng thái phiếu**

Phiếu chuyển sang:

Đã xử lý

---

# **2.2 Báo cáo tồn kho (Inventory Report)**

Manager vào:

Quản lý Kho → Báo cáo tồn kho

/inventory/report

---

Manager có thể lọc:

Khoảng thời gian (From — To)

Nguyên liệu cụ thể

---

Hệ thống hiển thị bảng:

\[table\]

---

### **Công thức tính**

**Tồn đầu kỳ**

Tồn trước ngày From

---

**Tổng nhập trong kỳ**

SUM(StockIn)

---

**Tổng xuất trong kỳ**

SUM(StockOut + Sale Deduction)

---

**Tồn cuối kỳ**

Tồn cuối = Tồn đầu + Nhập − Xuất

---

**Giá trị tồn**

Tồn cuối × CostPrice

---

# **2.3 Lịch sử biến động kho (Inventory Ledger)**

Manager bấm vào **một nguyên liệu** trong báo cáo.

---

Hệ thống hiển thị bảng log:

\[table\]

---

### **Loại giao dịch**

INITIAL_STOCK

STOCK_IN

STOCK_OUT

SALE_DEDUCTION

INVENTORY_CHECK

STOCK_OUT_REVERSE

---

Manager có thể lọc theo:

- loại giao dịch

- khoảng thời gian

---

# **3. Acceptance Criteria**

---

# **AC-01: Tạo phiếu kiểm kê**

**Given**

Manager ở trang:

/inventory/check

---

**When**

Manager bấm:

\+ Tạo phiếu kiểm kê

---

**Then**

Hệ thống hiển thị bảng NVL với:

Tồn theo sổ = Ingredient.CurrentStock

Manager nhập tồn thực tế.

Hệ thống tự tính:

Chênh lệch

---

# **AC-02: Kiểm kê — thực tế &gt; sổ sách**

**Given**

Nguyên liệu:

Bột mì

Tồn sổ:

500g

Tồn thực tế:

600g

---

**When**

Manager bấm:

Xử lý chênh lệch

---

**Then**

Hệ thống:

- Tạo phiếu nhập kho kiểm kê:

+100g

- Cập nhật:

CurrentStock = 600g

- Ghi:

InventoryTransaction

Type = INVENTORY_CHECK

---

# **AC-03: Kiểm kê — thực tế &lt; sổ sách**

**Given**

Nguyên liệu:

Đường

Tồn sổ:

1000g

Tồn thực tế:

800g

---

**When**

Manager xử lý chênh lệch.

---

**Then**

Hệ thống:

- Tạo phiếu xuất kho kiểm kê:

\-200g

- Cập nhật:

CurrentStock = 800g

---

# **AC-04: Báo cáo tồn kho**

**Given**

Khoảng thời gian:

01/03 — 10/03

Có giao dịch nhập và xuất.

---

**When**

Manager xem báo cáo.

---

**Then**

Hệ thống hiển thị đúng:

- Tồn đầu kỳ

- Tổng nhập

- Tổng xuất

- Tồn cuối kỳ

- Đơn giá bình quân

- Giá trị tồn

---

# **AC-05: Lịch sử biến động kho**

**Given**

Nguyên liệu:

Thịt bò

có nhiều giao dịch.

---

**When**

Manager bấm xem chi tiết.

---

**Then**

Hệ thống hiển thị log:

- Thời điểm

- Loại giao dịch

- Số phiếu

- Biến động (+/-)

- Tồn sau giao dịch
