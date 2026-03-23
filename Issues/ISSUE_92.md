# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem):**

Trong hệ thống quản lý nhà hàng, hiện tại chưa có module quản lý **discount / voucher** tập trung để:

- Áp dụng giảm giá vào đơn hàng một cách nhất quán

- Kiểm soát điều kiện áp dụng (giá trị đơn tối thiểu, thời gian hiệu lực)

- Tránh sai sót khi tính tiền (giảm quá mức, áp sai rule)

- Hỗ trợ triển khai các chương trình khuyến mãi

---

## **Pain Points hiện tại:**

- Logic giảm giá có thể bị viết rải rác trong nhiều nơi (Order, Payment, UI)

- Không kiểm soát được voucher còn hạn hay không

- Không có cơ chế validate chuẩn (min order, max discount)

- Dễ gây sai lệch tổng tiền → ảnh hưởng doanh thu

---

## **Giá trị Nghiệp vụ (Business Value):**

- **Tăng doanh thu**: hỗ trợ chương trình khuyến mãi linh hoạt

- **Tính toán chính xác**: đảm bảo tiền thanh toán đúng

- **Quản lý tập trung**: tất cả voucher/discount nằm trong một module

- **Dễ mở rộng**: có thể thêm rule nâng cao sau này (happy hour, combo)

---

## **Đối tượng (Actor):**

### **Primary Actor:**

- Nhân viên thu ngân / phục vụ (Staff)
  - Áp dụng voucher vào đơn hàng

### **Secondary Actor:**

- Quản trị viên (Admin)
  - Tạo và quản lý voucher

---

## **User Story Statement**

**Là nhân viên thu ngân**, tôi muốn nhập và áp dụng voucher vào đơn hàng để hệ thống tự động tính toán giảm giá chính xác và khách hàng được hưởng đúng ưu đãi.

---

# **2. Luồng Người dùng (User Flow)**

---

## **2.1. Luồng chính: Áp dụng voucher**

1. Staff mở hoặc tạo Order

2. Nhập mã voucher vào ô "Mã giảm giá" hoặc lựa chọn các voucher có thể sử dụng khi bấm vào nút Quà tặng (icon GIFT)

3. Click "Áp dụng"

4. Hệ thống:
   - Kiểm tra voucher tồn tại

   - Kiểm tra is_active

   - Kiểm tra thời gian (start_date, end_date), thời gian trong ngày (StartTime, EndTime)

   - Kiểm tra min_order_value

   -

5. Nếu voucher có free item

Tự động thêm món vào order

Set giá = 0

Đánh dấu: is_free_item = true

6. Nếu hợp lệ:
   - Tính discount

   - Cập nhật order:
     - discount_amount

     - final_price

   - Hiển thị kết quả

7. Nếu không hợp lệ:
   - Hiển thị lỗi tương ứng

   - Không nằm trong khung giờ

→ Hiển thị: **"Voucher không áp dụng trong khung giờ này"**

---

## **2.2. Luồng: Tính toán discount**

- Nếu type = percent:
  - discount = total \* value / 100

  - áp dụng max_discount nếu có

- Nếu type = fixed:
  - discount = value

- Đảm bảo:
  - final_price ≥ 0

---

## **2.3. Luồng: Gỡ voucher**

1. Staff click "Xóa voucher"

2. Hệ thống:
   - Remove discount

   - Tính lại tổng tiền

---

## **2.4. Luồng: Tạo voucher**

1. Admin vào màn "Voucher"

2. Click "Tạo voucher"

3. Nhập:

- Code

- Type

- Value

- Max discount

- Min order value

- Start date / End date

4. Click "Lưu"

5. Hệ thống:

- Validate

- Lưu DB

- Thông báo thành công

---

## **2.5. Luồng: Tìm kiếm & quản lý voucher**

- Tìm theo code

- Lọc theo:
  - type

  - trạng thái

- Phân trang

---

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

---

### **AC-01: Áp dụng voucher thành công**

**Given:** Order hợp lệ\
**When:** Nhập voucher đúng\
**Then:**

- Discount được tính đúng

- Tổng tiền cập nhật chính xác

---

### **AC-02: Voucher không tồn tại**

**Then:**

- Hiển thị "Voucher không hợp lệ"

---

### **AC-03: Voucher hết hạn**

**Then:**

- Hiển thị "Voucher đã hết hạn"

---

### **AC-04: Không đạt min order**

**Then:**

- Hiển thị "Không đủ điều kiện áp dụng"

---

### **AC-05: Percent có max discount**

**Then:**

- Không vượt quá max_discount

---

### **AC-06: Không âm tiền**

**Then:**

- final_price ≥ 0

---

### **AC-07: Gỡ voucher**

**Then:**

- Tổng tiền trở lại ban đầu

---

### **AC-08: Tạo voucher thành công**

**Then:**

- Code unique

- Lưu thành công

---

### **AC-09: Code trùng**

**Then:**

- Báo lỗi "Voucher đã tồn tại"

---

### **AC-10: Performance**

**Then:**

- Validate &lt; 1s

---

### **AC-11: Voucher chỉ áp dụng trong khung giờ**

**Given:** Voucher có StartTime = 14:00, EndTime = 17:00\
**When:** Nhân viên áp dụng lúc 13:00\
**Then:**

- Hiển thị: "Voucher không áp dụng trong khung giờ này"

- Không áp dụng discount

---

### **AC-12: Voucher hợp lệ trong khung giờ**

**Given:** Voucher có khung giờ hợp lệ\
**When:** Áp dụng trong khoảng thời gian đó\
**Then:**

- Voucher được áp dụng bình thường

### **AC-13: Free item**

- Món được thêm

- Giá = 0

### **AC-14: Remove voucher**

- Free item bị xóa

# **4. Gợi ý Entity (chuẩn Clean Architecture)**

---

## **🎯 4.1. Promotion (Aggregate Root)**

public class Promotion : BaseEntity

{

public string Code { get; set; } = default!;

public PromotionType Type { get; set; }  // Percent, Fixed, FreeItem

public decimal Value { get; set; }

public decimal? MaxDiscount { get; set; }

public decimal? MinOrderValue { get; set; }

public Guid? ItemtId { get; set; }

public int FreeQuantity { get; set; }

public DateTime StartDate { get; set; }

public DateTime EndDate { get; set; }

public TimeSpan? StartTime { get; set; } // ví dụ: 14:00

public TimeSpan? EndTime { get; set; } // ví dụ: 17:00

public bool IsActive { get; set; }

public int UsageLimit { get; set; }

public int UsedCount { get; set; }

}

---

## **🎯 4.2. Enum**

public enum PromotionType

{

Percent = 1,

Fixed = 2,

FreeItem = 3

}

---

## **🎯 4.3. Order (update thêm)**

public class Order : BaseEntity

{

public decimal SubTotal { get; set; }

public decimal DiscountAmount { get; set; }

public decimal FinalPrice { get; set; }

public Guid? PromotionId { get; set; }

public Promotion? Promotion { get; set; }

}

public class OrderItem

{

public Guid ItemtId { get; set; }

public decimal Price { get; set; }

public int Quantity { get; set; }

public bool IsFreeItem { get; set; } //Quan trọng

}

---

## **🎯 4.4. Domain Service (quan trọng)**

public class PromotionService

{

public decimal CalculateDiscount(Order order, Promotion promo)

{

if (order.SubTotal &lt; promo.MinOrderValue)

return 0;

decimal discount = 0;

if (promo.Type == PromotionType.Percent)

{

discount = order.SubTotal \* promo.Value / 100;

if (promo.MaxDiscount.HasValue)

discount = Math.Min(discount, promo.MaxDiscount.Value);

}

else

{

discount = promo.Value;

}

return Math.Min(discount, order.SubTotal);

}

}

public bool IsValidTime(Promotion promo)

{

if (!promo.StartTime.HasValue || !promo.EndTime.HasValue)

return true; // không có giới hạn giờ

var now = [DateTime.Now](http://DateTime.Now).TimeOfDay;

return now &gt;= promo.StartTime && now &lt;= promo.EndTime;

}
