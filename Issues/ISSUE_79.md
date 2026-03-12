# **US-INV-03: Phiếu Xuất Kho Thủ Công (Manual Stock Out) – 13 Points**

---

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Trong quá trình vận hành kho, có nhiều trường hợp nguyên vật liệu cần **xuất ra khỏi kho nhưng không phải do bán hàng**, ví dụ:

- NVL bị hư hỏng hoặc hết hạn

- NVL dùng cho mục đích nội bộ

- Điều chỉnh tồn kho khi kiểm kê

- Các lý do vận hành khác

Nếu không có cơ chế ghi nhận xuất kho:

- Tồn kho trong hệ thống sẽ không khớp với thực tế

- Không có lịch sử biến động kho

- Không thể truy vết nguyên nhân giảm tồn kho

Do đó cần có **Phiếu Xuất Kho Thủ Công** để ghi nhận các lần xuất NVL.

---

## **Pain Points hiện tại**

- Không có màn hình ghi nhận xuất kho ngoài bán hàng.

- Không có lịch sử cho các trường hợp NVL bị hủy hoặc sử dụng nội bộ.

- Không thể kiểm soát lý do giảm tồn kho.

- Không có cách điều chỉnh tồn kho khi phát sinh sai lệch.

---

## **Giá trị Nghiệp vụ (Business Value)**

### **1. Kiểm soát tồn kho chính xác**

Mọi lần xuất NVL đều được ghi nhận và cập nhật tồn kho.

### **2. Lưu lịch sử biến động kho**

Hệ thống lưu lại các giao dịch xuất kho để phục vụ kiểm tra và báo cáo.

### **3. Hỗ trợ kiểm soát vận hành**

Manager có thể theo dõi lý do xuất NVL như:

- Hủy NVL

- Xuất nội bộ

- Điều chỉnh kho

### **4. Cập nhật trạng thái món ăn**

Nếu NVL bị xuất hết, hệ thống có thể tự động **đánh dấu món ăn hết hàng**.

---

## **Đối tượng (Actor)**

### **Primary Actor**

**Manager**

- Tạo phiếu xuất kho

- Xem danh sách phiếu xuất

- Xem chi tiết phiếu

- Xóa phiếu xuất

### **Secondary Actors**

**System**

- Cập nhật tồn kho

- Ghi InventoryTransaction

- Cập nhật trạng thái IsOutOfStock của MenuItem

---

## **User Story Statement**

Là **Manager**, tôi muốn **tạo Phiếu Xuất kho để ghi nhận nguyên vật liệu xuất ra khỏi kho**, để **hệ thống tự động trừ tồn và lưu lịch sử giao dịch kho**.

---

## **Tiền điều kiện (Preconditions)**

- US-INV-01 đã hoàn thành.

- Danh mục **Ingredient** đã tồn tại.

---

## **Entity liên quan**

- StockOutReceipt

- StockOutReceiptItem

- Ingredient

- InventoryTransaction

- MenuItemIngredient

---

# **2. Luồng Người dùng (User Flow)**

---

# **2.1 Luồng: Xem danh sách phiếu xuất kho**

1. Manager đăng nhập hệ thống

2. Điều hướng đến:

Quản lý Kho → Xuất kho

/inventory/stock-out

3. Hệ thống hiển thị danh sách phiếu xuất với các cột:

- Số phiếu (XK-yyyymmdd-0001)

- Ngày xuất

- Lý do xuất

- Tổng số dòng NVL

- Tổng giá trị xuất

- Người tạo

4. Manager có thể:

- Lọc theo **lý do xuất**

- Lọc theo **khoảng ngày**

- Phân trang danh sách

---

# **2.2 Luồng: Tạo phiếu xuất kho thủ công**

1. Manager bấm:

\+ Tạo phiếu xuất kho

2. Hệ thống hiển thị form tạo phiếu.

---

## **Thông tin chung**

- Số phiếu (tự sinh)

- Ngày xuất (mặc định hôm nay)

- Lý do xuất (bắt buộc)

Các giá trị lý do:

Hủy NVL

Xuất nội bộ

Kiểm kê

Khác

- Ghi chú (tùy chọn)

---

## **Bảng chi tiết NVL**

Mỗi dòng gồm:

- Nguyên liệu (dropdown NVL Active)

- Số lượng xuất (bắt buộc, &gt; 0)

- Đơn vị (tự động)

- Đơn giá xuất

Có thể:

\- tự tính theo giá vốn bình quân

\- hoặc nhập tay

- Thành tiền

Thành tiền = Số lượng × Đơn giá

(read-only)

---

3. Manager thêm các dòng NVL cần xuất.

4. Manager bấm **"Lưu"**

5. Hệ thống validate:

- Phiếu phải có **ít nhất 1 dòng NVL**

- Mỗi NVL **chỉ xuất hiện 1 lần trong phiếu**

- Số lượng xuất **&gt; 0**

---

# **2.3 Xử lý tồn kho không đủ**

Nếu số lượng xuất lớn hơn tồn kho:

Ví dụ

CurrentStock = 100g

Xuất = 300g

Hệ thống:

- Hiển thị cảnh báo

"Tồn kho không đủ"

- Vẫn cho phép lưu phiếu

- Tồn kho sau khi xuất:

CurrentStock = 0

(không cho phép âm)

---

# **2.4 Xử lý khi lưu phiếu xuất**

Nếu dữ liệu hợp lệ, hệ thống thực hiện:

Tạo:

StockOutReceipt

StockOutReceiptItems

Cập nhật tồn kho:

Ingredient.CurrentStock -= Số lượng xuất

(CurrentStock tối thiểu = 0)

Ghi giao dịch kho:

InventoryTransaction

Type = STOCK_OUT

Reference = mã phiếu

Reason = lý do xuất

Kiểm tra lại trạng thái món ăn:

MenuItem.IsOutOfStock

Hiển thị thông báo:

"Tạo phiếu xuất kho thành công"

---

# **2.5 Luồng: Xem chi tiết phiếu xuất**

1. Manager bấm vào một phiếu xuất trong danh sách

2. Hệ thống hiển thị:

### **Thông tin chung**

- Số phiếu

- Ngày xuất

- Lý do xuất

- Người tạo

- Ghi chú

### **Bảng chi tiết NVL**

- Nguyên liệu

- Số lượng xuất

- Đơn vị

- Đơn giá

- Thành tiền

---

# **2.6 Luồng: Xóa phiếu xuất**

1. Manager bấm **"Xóa phiếu xuất"**

2. Hệ thống hiển thị xác nhận:

Bạn có chắc muốn xóa phiếu xuất này?

3. Nếu xác nhận:

Hệ thống:

Cập nhật tồn kho:

Ingredient.CurrentStock += Số lượng đã xuất

Ghi giao dịch kho:

InventoryTransaction

Type = STOCK_OUT_REVERSE

Reference = mã phiếu

4. Phiếu xuất bị xóa khỏi hệ thống.

---

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

---

# **AC-01: Hiển thị danh sách phiếu xuất**

**Given**

Manager đã đăng nhập

**When**

Truy cập:

/inventory/stock-out

**Then**

- Hệ thống hiển thị danh sách phiếu xuất

- Có chức năng:
  - Lọc theo lý do xuất

  - Lọc theo khoảng ngày

  - Phân trang

---

# **AC-02: Tạo phiếu xuất thành công**

**Given**

Nguyên liệu **Thịt bò** hiện có:

CurrentStock = 3000g

**When**

Manager tạo phiếu xuất:

Thịt bò — 500g — Hủy NVL

và bấm **Lưu**

**Then**

Tồn kho được cập nhật:

CurrentStock = 2500g

Hệ thống ghi giao dịch:

InventoryTransaction

Type = STOCK_OUT

---

# **AC-03: Xuất kho khi tồn không đủ**

**Given**

Nguyên liệu **Hành tây** hiện có:

CurrentStock = 100g

**When**

Manager tạo phiếu xuất:

300g

**Then**

Hệ thống hiển thị cảnh báo:

"Tồn kho không đủ"

Nhưng vẫn cho phép lưu phiếu.

Sau khi lưu:

CurrentStock = 0

---

# **AC-04: Tự động đánh dấu món ăn hết hàng**

**Given**

Nguyên liệu:

Thịt bò

CurrentStock = 200g

Món ăn:

Bò Lúc Lắc

sử dụng **Thịt bò**.

**When**

Manager xuất:

200g Thịt bò

**Then**

CurrentStock = 0

Hệ thống cập nhật:

MenuItem.IsOutOfStock = true

cho món **Bò Lúc Lắc**.

---

# **AC-05: Xóa phiếu xuất kho**

**Given**

Phiếu xuất:

XK-20260311-0001

đã xuất:

Thịt bò 500g

**When**

Manager xóa phiếu xuất

**Then**

Tồn kho được cập nhật lại:

CurrentStock + 500g

Hệ thống ghi:

InventoryTransaction

Type = STOCK_OUT_REVERSE
