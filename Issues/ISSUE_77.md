# **US-INV-02: Phiếu Nhập Kho (Stock In Receipt) – 13 Points**

---

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Trong quá trình vận hành, nhà hàng/cửa hàng cần nhập thêm nguyên vật liệu (NVL) vào kho từ nhiều nguồn như:

- Nhà cung cấp

- Mua ngoài

- Bổ sung nội bộ

Nếu hệ thống không ghi nhận các lần nhập kho:

- Số lượng tồn kho sẽ không phản ánh đúng thực tế

- Không có lịch sử biến động kho

- Không thể kiểm soát giá vốn NVL

Do đó hệ thống cần chức năng **Phiếu Nhập Kho** để ghi nhận các lần nhập NVL.

---

## **Pain Points hiện tại**

- Không có màn hình ghi nhận việc nhập nguyên vật liệu.

- Không có lịch sử biến động kho.

- Tồn kho không được cập nhật khi NVL được nhập thêm.

- Không thể kiểm tra lại các lần nhập kho trước đó.

---

## **Giá trị Nghiệp vụ (Business Value)**

### **1. Cập nhật tồn kho chính xác**

Mỗi lần nhập NVL sẽ tự động cộng tồn kho của nguyên liệu.

### **2. Lưu lịch sử giao dịch kho**

Hệ thống ghi lại toàn bộ lịch sử nhập kho để phục vụ kiểm tra, audit và báo cáo.

### **3. Hỗ trợ vận hành menu**

Khi NVL được nhập thêm, hệ thống có thể tự động **mở lại các món ăn bị hết hàng**.

### **4. Quản lý chi phí nguyên vật liệu**

Đơn giá nhập giúp phục vụ tính toán **giá vốn NVL**.

---

## **Đối tượng (Actor)**

### **Primary Actor**

**Manager**

- Tạo phiếu nhập kho

- Xem danh sách phiếu nhập

- Kiểm tra chi tiết phiếu

- Xóa phiếu nhập khi cần

### **Secondary Actors**

**System**

- Cập nhật tồn kho

- Ghi InventoryTransaction

- Kiểm tra trạng thái IsOutOfStock của MenuItem

---

## **User Story Statement**

Là **Manager**, tôi muốn **tạo Phiếu Nhập kho để ghi nhận nguyên vật liệu nhập vào kho**, để **hệ thống tự động cộng tồn và lưu lịch sử giao dịch kho**.

---

## **Tiền điều kiện (Preconditions)**

- US-INV-01 hoàn thành (đã có danh mục NVL).

- Các **Ingredient** đã tồn tại trong hệ thống.

---

## **Entity liên quan**

- StockInReceipt

- StockInReceiptItem

- Ingredient

- InventoryTransaction

- MenuItemIngredient

---

# **2. Luồng Người dùng (User Flow)**

---

# **2.1 Luồng: Xem danh sách phiếu nhập kho**

1. Manager đăng nhập vào hệ thống

2. Điều hướng đến:

Quản lý Kho → Nhập kho

/inventory/stock-in

3. Hệ thống hiển thị danh sách phiếu nhập gồm các cột:

- Số phiếu (NK-yyyymmdd-0001)

- Ngày nhập

- Tổng số dòng NVL

- Tổng giá trị nhập

- Người tạo

- Ghi chú

4. Manager có thể:

- Tìm kiếm theo **số phiếu**

- Lọc theo **khoảng ngày nhập**

- Phân trang danh sách

---

# **2.2 Luồng: Tạo phiếu nhập kho**

1. Manager bấm:

\+ Tạo phiếu nhập kho

2. Hệ thống hiển thị form tạo phiếu gồm:

## **Thông tin chung**

- Số phiếu (tự sinh)

- Ngày nhập (mặc định hôm nay)

- Ghi chú chung (tùy chọn)

---

## **Bảng chi tiết NVL**

Mỗi dòng gồm:

- Nguyên liệu (dropdown NVL Active)

- Số lượng nhập (bắt buộc, &gt; 0)

- Đơn vị (tự động hiển thị)

- Đơn giá nhập (tùy chọn, ≥ 0)

- Thành tiền

Thành tiền = Số lượng × Đơn giá

(read-only)

- Hạn sử dụng (tùy chọn)

- Số lô / Batch code (tùy chọn)

---

3. Manager thêm các dòng NVL cần nhập.

4. Manager bấm **"Lưu"**

5. Hệ thống validate:

- Phiếu phải có **ít nhất 1 dòng NVL**

- Mỗi NVL **chỉ xuất hiện 1 lần trong phiếu**

- Số lượng nhập **&gt; 0**

6. Nếu hợp lệ, hệ thống thực hiện:

- Tạo:

StockInReceipt

StockInReceiptItems

- Cập nhật tồn kho:

Ingredient.CurrentStock += Số lượng nhập

- Ghi giao dịch kho:

InventoryTransaction

Type = STOCK_IN

Reference = mã phiếu

- Kiểm tra lại trạng thái **IsOutOfStock của MenuItem**

7. Hiển thị thông báo:

"Tạo phiếu nhập kho thành công"

---

# **2.3 Luồng: Xem chi tiết phiếu nhập kho**

1. Manager bấm vào một phiếu nhập trong danh sách

2. Hệ thống hiển thị trang chi tiết gồm:

### **Thông tin chung**

- Số phiếu

- Ngày nhập

- Người tạo

- Ghi chú

### **Bảng chi tiết NVL**

- Nguyên liệu

- Số lượng nhập

- Đơn vị

- Đơn giá

- Thành tiền

- Hạn sử dụng

- Batch code

3. Trang chi tiết có nút:

Xóa phiếu nhập

---

# **2.4 Luồng: Xóa phiếu nhập kho**

1. Manager bấm **"Xóa phiếu nhập"**

2. Hệ thống hiển thị xác nhận:

Bạn có chắc muốn xóa phiếu nhập này?

3. Nếu xác nhận:

Hệ thống thực hiện:

- Trả ngược tồn kho:

Ingredient.CurrentStock -= Số lượng đã nhập

- Ghi giao dịch:

InventoryTransaction

Type = STOCK_IN_REVERSE

Reference = mã phiếu

4. Phiếu nhập bị xóa khỏi hệ thống.

---

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

---

## **AC-01: Hiển thị danh sách phiếu nhập**

**Given**

Manager đã đăng nhập

**When**

Truy cập:

/inventory/stock-in

**Then**

- Hệ thống hiển thị danh sách phiếu nhập

- Có chức năng:
  - Tìm kiếm

  - Lọc theo ngày

  - Phân trang

---

## **AC-02: Tạo phiếu nhập kho thành công**

**Given**

NVL **Thịt bò** hiện có:

CurrentStock = 5000g

**When**

Manager tạo phiếu nhập:

Thịt bò — 2000g — 150.000đ/kg

và bấm **Lưu**

**Then**

- Hệ thống tạo phiếu nhập với số phiếu dạng:

NK-yyyymmdd-xxxx

- Tồn kho được cập nhật:

CurrentStock = 7000g

- Ghi:

InventoryTransaction

Type = STOCK_IN

---

## **AC-03: Thất bại — không có dòng NVL**

**Given**

Form phiếu nhập kho đang mở

**When**

Manager bấm **Lưu** khi chưa thêm dòng NVL nào

**Then**

Hệ thống hiển thị lỗi:

"Phải có ít nhất 1 nguyên liệu"

---

## **AC-04: Thất bại — NVL trùng dòng**

**Given**

Manager đã thêm **Thịt bò** vào phiếu

**When**

Manager thêm **Thịt bò** lần nữa

**Then**

Hệ thống hiển thị lỗi:

"Nguyên liệu đã có trong phiếu"

---

## **AC-05: Tự động mở lại món hết hàng**

**Given**

MenuItem **Bò Lúc Lắc**

IsOutOfStock = true

do NVL **Thịt bò** đã hết.

**When**

Manager nhập kho:

Thịt bò = 500g

**Then**

Nếu tất cả NVL của món đều còn hàng:

IsOutOfStock = false

---

## **AC-06: Xóa phiếu nhập kho**

**Given**

Phiếu nhập:

NK-20260311-0001

đã nhập:

Thịt bò +2000g

**When**

Manager bấm **Xóa phiếu**

**Then**

- Tồn kho cập nhật:

CurrentStock giảm 2000g

- Hệ thống ghi:

InventoryTransaction

Type = STOCK_IN_REVERSE
