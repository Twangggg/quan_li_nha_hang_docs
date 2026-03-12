# **US-INV-04: Định lượng Nguyên liệu & Auto trừ kho khi bán hàng (13 Points)**

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Trong hệ thống bán hàng nhà hàng/cafe, mỗi món ăn thường sử dụng nhiều nguyên liệu với **định lượng cố định**.

Ví dụ:

\[table\]

Nếu không có cơ chế định lượng:

- Hệ thống **không biết phải trừ bao nhiêu nguyên liệu khi bán món**

- Tồn kho **không phản ánh đúng thực tế**

- Nhân viên phải **trừ kho thủ công**

Vì vậy cần **Recipe/BOM (Bill Of Materials)** cho từng món ăn và **tự động trừ kho khi Order hoàn thành**.

---

## **Giá trị nghiệp vụ (Business Value)**

### **1. Tự động hóa quản lý kho**

Mỗi khi món được bán → hệ thống tự động trừ NVL.

---

### **2. Tồn kho chính xác theo thời gian thực**

Không cần nhập phiếu xuất thủ công khi bán hàng.

---

### **3. Kiểm soát nguyên liệu tiêu thụ**

Có thể thống kê:

- NVL tiêu thụ theo món

- NVL tiêu thụ theo ngày

---

### **4. Tự động phát hiện món hết hàng**

Nếu NVL hết → món sử dụng NVL đó sẽ tự động **IsOutOfStock**.

---

# **Actor**

### **Primary Actor**

**Manager**

- Thiết lập định lượng nguyên liệu cho món ăn

- Chỉnh sửa / xóa định lượng

---

### **Secondary Actor**

**System**

- Tự động trừ kho khi OrderItem hoàn thành

- Ghi InventoryTransaction

- Kiểm tra trạng thái món ăn

---

# **User Story Statement**

Là **Manager**, tôi muốn **thiết lập định lượng nguyên liệu cho từng món ăn**, để **hệ thống tự động trừ kho khi món được bán** và đảm bảo tồn kho chính xác.

---

# **Tiền điều kiện (Preconditions)**

- US-INV-01 hoàn thành (đã có Ingredient)

- US-INV-02 hoàn thành (đã có tồn kho)

- MenuItem đã tồn tại

---

# **Entity liên quan**

- MenuItem

- MenuItemIngredient (Recipe/BOM)

- Ingredient

- OrderItem

- StockOutReceipt

- InventoryTransaction

---

# **2. Luồng Người dùng (User Flow)**

---

# **2.1 Xem và thiết lập Recipe**

1️⃣ Manager vào:

Quản lý Thực đơn → Chọn món ăn

2️⃣ Mở tab:

Định lượng nguyên liệu

3️⃣ Hệ thống hiển thị danh sách NVL đã thiết lập:

\[table\]

Ví dụ:

| Thịt bò | 200 | g | 3500 |\
| Ớt chuông | 50 | g | 1200 |

---

4️⃣ Nếu món chưa có recipe:

Hiển thị badge:

Chưa thiết lập định lượng

---

# **2.2 Thêm nguyên liệu vào recipe**

1️⃣ Manager bấm:

Thêm nguyên liệu

2️⃣ Form hiển thị:

- Nguyên liệu (dropdown Ingredient Active)

- Số lượng / phần (QuantityPerServing)

Rule:

QuantityPerServing &gt; 0

3️⃣ Manager bấm **Lưu**

4️⃣ Hệ thống validate:

- NVL không được trùng trong recipe

- QuantityPerServing &gt; 0

5️⃣ Nếu hợp lệ:

Recipe được lưu vào bảng:

MenuItemIngredient

---

# **2.3 Sửa hoặc xóa dòng recipe**

Manager có thể:

### **Sửa**

- QuantityPerServing

---

### **Xóa**

- Remove NVL khỏi recipe

---

### **Rule**

Một nguyên liệu chỉ được xuất hiện **1 lần** trong recipe.

---

# **2.4 Set Menu / Combo**

Nếu món là **Set Menu (Combo)**:

- Recipe **không đặt ở combo**

- Recipe đặt **ở từng món thành phần**

Ví dụ:

Combo A gồm:

- Bò lúc lắc

- Coca

Recipe đặt tại:

Bò lúc lắc

---

# **2.5 Auto trừ kho khi bán hàng**

Trigger:

OrderItem.Status → Completed

---

## **Flow hệ thống**

1️⃣ Hệ thống nhận event:

OrderItemCompleted

---

2️⃣ Lấy recipe:

MenuItem → MenuItemIngredient

---

3️⃣ Nếu **không có recipe**

Bỏ qua

Không lỗi

---

4️⃣ Với mỗi dòng recipe:

Công thức trừ kho:

QuantityToDeduct =

Recipe.QuantityPerServing × OrderItem.Quantity

---

Ví dụ:

Recipe: Thịt bò 200g/phần

Order: 2 phần

→ Trừ:

200 × 2 = 400g

---

5️⃣ Cập nhật tồn kho

Ingredient.CurrentStock -= QuantityToDeduct

Rule:

CurrentStock &gt;= 0

---

6️⃣ Ghi giao dịch kho

Tạo:

InventoryTransaction

Type:

SALE_DEDUCTION

Reference:

OrderItemId

---

7️⃣ Hệ thống tạo phiếu xuất kho tự động

StockOutReceipt

Type = SALE

Reference = OrderId

---

8️⃣ Re-check trạng thái món ăn

Nếu NVL:

CurrentStock = 0

→ Các món dùng NVL đó:

MenuItem.IsOutOfStock = true

---

# **Lưu ý kỹ thuật**

### **Không chặn thanh toán**

Auto trừ kho chạy:

Background job / async event

---

### **Không trừ kho khi order bị hủy**

Nếu:

OrderItem.Status = Cancelled

OrderItem.Status = Rejected

→ Không trigger deduction.

---

# **3. Acceptance Criteria**

---

# **AC-01: Thiết lập recipe**

**Given**

Món:

Bò Lúc Lắc

Nguyên liệu:

Thịt bò

Ớt chuông

đã tồn tại.

---

**When**

Manager mở tab **Định lượng nguyên liệu** và thêm:

Thịt bò — 200g

Ớt chuông — 50g

---

**Then**

Recipe được lưu với **2 dòng nguyên liệu**.

Hệ thống hiển thị tồn kho hiện tại của mỗi NVL.

---

# **AC-02: Không cho trùng NVL**

**Given**

Recipe **Bò Lúc Lắc** đã có:

Thịt bò

---

**When**

Manager thêm lại:

Thịt bò

---

**Then**

Hệ thống hiển thị lỗi:

"Nguyên liệu này đã có trong định lượng"

---

# **AC-03: Hiển thị badge chưa có recipe**

**Given**

Món:

Nước suối

chưa thiết lập recipe.

---

**When**

Manager mở tab:

Định lượng nguyên liệu

---

**Then**

Hệ thống hiển thị badge:

Chưa thiết lập định lượng

---

# **AC-04: Auto trừ kho khi bán**

**Given**

Recipe:

Bò Lúc Lắc

Thịt bò — 200g/phần

Tồn kho:

Thịt bò = 1000g

---

**When**

OrderItem:

Bò Lúc Lắc

Quantity = 2

Status = Completed

---

**Then**

Hệ thống trừ kho:

200 × 2 = 400g

Tồn mới:

600g

Hệ thống ghi:

InventoryTransaction

Type = SALE_DEDUCTION

Ref = OrderItemId

Và tạo:

StockOutReceipt

Type = SALE

---

# **AC-05: Không trừ kho khi chưa có recipe**

**Given**

Món:

Nước suối

không có recipe.

---

**When**

OrderItem:

Nước suối → Completed

---

**Then**

- Không trừ kho

- Không phát sinh lỗi

---

# **AC-06: Tự động bật IsOutOfStock**

**Given**

Nguyên liệu:

Thịt bò

bị trừ tồn về:

CurrentStock = 0

---

**When**

Hệ thống re-check món ăn.

---

**Then**

Tất cả MenuItem sử dụng **Thịt bò** được cập nhật:

IsOutOfStock = true

---

# **AC-07: Không trừ kho khi order bị hủy**

**Given**

OrderItem:

Bò Lúc Lắc

Status = Cancelled

---

**When**

Order bị hủy.

---

**Then**

- Không trừ kho

- Không ghi InventoryTransaction
