# Nguồn nên tham khảo:

# <https://helpv2.cukcuk.vn/vi/kb/ghep-order>

<https://helpv2.cukcuk.vn/vi/kb/tach_order>

<https://youtu.be/-HVrVSE05lo?si=KKbgpasQk9Z5yDup>

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Sau khi hệ thống FoodHub đã triển khai:

- Table Layout cơ bản

- Order Dine-in

- KDS

- Billing cơ bản (1 Order → 1 Payment)

Hệ thống cần mở rộng khả năng xử lý Order trong các tình huống thực tế như:

- Gộp nhiều Order thành một Order duy nhất

- Tách một phần Order sang bàn khác

- Chuyển món giữa các Order đang hoạt động

- Đồng bộ trạng thái bàn chính xác khi Order thay đổi

Phạm vi Sprint 3 chỉ xử lý ở mức Order.\
Không bao gồm tách hóa đơn thanh toán (Split Bill) hoặc xử lý nhiều Payment trên cùng một Order.

Hệ thống phải đảm bảo:

- Không làm sai tổng tiền

- Không tạo trùng Order

- Không ảnh hưởng báo cáo doanh thu

- Không cho thao tác nếu Order đã PAID

## **Pain Points hiện tại**

- Khi khách ghép bàn thực tế, hệ thống không hỗ trợ gộp Order linh hoạt.

- Khách đổi bàn giữa chừng nhưng không thể chuyển món sang Order khác.

- Nhập nhầm bàn khi Order → phải hủy và nhập lại.

- Nguy cơ sai tổng tiền khi điều chỉnh thủ công.

- Thiếu kiểm soát transaction khi gộp/tách Order.

## **Giá trị Nghiệp vụ (Business Value)**

- Linh hoạt xử lý tình huống vận hành thực tế.

- Giảm sai sót khi nhập nhầm bàn.

- Tăng tốc độ phục vụ khi khách đổi chỗ.

- Đảm bảo dữ liệu chính xác cho Billing và báo cáo.

# **2. Đối tượng (Actors)**

## **Primary Actor**

### **Cashier**

- Gộp Order

- Tách Order

- Chuyển món giữa các Order

- Đảm bảo tổng tiền chính xác

### **Waiter**

- Đề xuất điều chỉnh

- Thực hiện thao tác nếu được phân quyền

## **Secondary Actor**

### **Manager**

- Giám sát

- Override

# **3. User Story Statement**

Là Cashier, tôi muốn có thể gộp Order, tách Order và chuyển món giữa các Order đang hoạt động để xử lý linh hoạt các tình huống phục vụ mà vẫn đảm bảo dữ liệu chính xác và không ảnh hưởng đến doanh thu.

# **4. Luồng Người Dùng (User Flow)**

# **4.1. Luồng: Ghép Order (Merge Order)**

## **Mục tiêu**

Gộp nhiều Order đang ACTIVE thành một Order duy nhất.

## **Điều kiện trước**

- Các Order đều ở trạng thái ACTIVE

- Không Order nào đã PAID

- Tất cả đều là Dine-in

## **Các bước**

1. Cashier mở màn hình quản lý order.

2. Chọn chức năng “Ghép Order” trên order.

3. Màn hình hiển thị màn hình nhỏ để chọn order hiện đang hoạt động theo khu vực Ex:

4. Lựa chọn bàn và Bấm đồng ý (Nếu khách ghép order và muốn chuyển bàn: thao tác chọn order cần ghép, ấn Chọn bàn. Bỏ tích chọn các bàn cũ và tích chọn bàn khách yêu cầu chuyển đến. Nhấn Đồng ý.)

5. Order được chọn để ghép sẽ tự động biến mất trên màn hình order. Tất cả thông tin của order gồm danh sách món đã gọi, số lượng khách,… sẽ được cập nhật vào order yêu cầu ghép.

## **Hệ thống xử lý**

- Chuyển toàn bộ OrderItem từ Order phụ sang Order chính với số/mã bàn sẽ là số/mã bàn chinh, số/mã bàn phụ Ex:

- Tính lại Subtotal / Tax / Discount

- Order phụ chuyển trạng thái MERGED

- Nếu bàn phụ không còn Order ACTIVE → bàn chuyển AVAILABLE

- Ghi log thao tác

- Thực hiện trong một database transaction

# **4.2. Luồng: Tách Order (Split / Move OrderItem)**

---

## **🎯 Mục tiêu**

Cho phép chuyển một hoặc nhiều OrderItem từ Order nguồn sang:

- Một Order khác đang ACTIVE\
  hoặc

- Tạo một Order mới cho bàn khác

---

## **Điều kiện trước (Preconditions)**

- Order nguồn đang ở trạng thái ACTIVE

- Order chưa PAID

- Không ở trạng thái LOCKED

- OrderItem chưa COMPLETED hoặc đã thanh toán

---

## **Các bước thực hiện**

1. Cashier mở màn hình **Order Detail** của Order nguồn.

2. Nhấn biểu tượng dấu ba chấm \[…\] hoặc chức năng “Tách Order”.

3. Hệ thống hiển thị danh sách OrderItem của Order hiện tại.

4. Cashier chọn các món cần tách.

5. Nhấn biểu tượng mũi tên chuyển sang phải (→) hoặc nút “Chuyển”.

6. Hệ thống hiển thị popup lựa chọn:
   - Danh sách Order đang ACTIVE theo khu vực

   - Hoặc tùy chọn tạo Order mới cho bàn khác

7. Cashier chọn:
   - Order đích (nếu đã tồn tại)\
     hoặc

   - Bàn đích để tạo Order mới

8. Nhấn “Đồng ý” để xác nhận.

---

## **Hệ thống xử lý**

- Validate trạng thái hợp lệ của:
  - Order nguồn

  - Order đích

  - OrderItem

- Nếu chọn bàn chưa có Order:
  - Tạo Order mới cho bàn đích

- Chuyển OrderItem đã chọn sang Order đích

- Tính lại:
  - Subtotal của Order nguồn

  - Subtotal của Order đích

  - Thuế / Discount (nếu có)

- Nếu Order nguồn không còn OrderItem:
  - Order nguồn → CLOSED

- Cập nhật trạng thái bàn:
  - Nếu bàn đích chưa OCCUPIED → chuyển OCCUPIED

  - Nếu bàn nguồn không còn Order ACTIVE → chuyển AVAILABLE

- Không thay đổi trạng thái KDS của món:
  - Nếu đang COOKING → vẫn giữ COOKING

- Ghi log thao tác

- Toàn bộ xử lý thực hiện trong một database transaction

# **4.3. Luồng: Chuyển bàn (Change Table)**

🎯 **Mục tiêu**

Cho phép chuyển toàn bộ Order đang ACTIVE từ bàn hiện tại sang một bàn khác khi khách đổi chỗ.

---

## **Điều kiện trước (Preconditions)**

- Order đang ở trạng thái ACTIVE

- Order chưa PAID

- Bàn đích đang AVAILABLE

- Bàn đích không INACTIVE

---

## **Các bước thực hiện**

1. Cashier mở **Order**.

2. Nhấn biểu tượng ba chấm \[…\].

3. Chọn chức năng **“Chuyển bàn”**.

4. Hệ thống hiển thị popup danh sách:
   - Các bàn AVAILABLE

   - Hiển thị theo khu vực (Tầng 1, Tầng 2, VIP…)

5. Cashier chọn bàn đích.

6. Nhấn “Đồng ý” để xác nhận.

---

## **Hệ thống xử lý**

- Validate:
  - Order hợp lệ

  - Bàn đích hợp lệ

- Cập nhật:
  - Order.TableId = TableId mới

- Không thay đổi:
  - OrderItem

  - Tổng tiền

  - Trạng thái KDS

- Cập nhật trạng thái bàn:
  - Bàn cũ:
    - Nếu không còn Order ACTIVE → AVAILABLE

  - Bàn mới:
    - OCCUPIED

- Ghi log thao tác

- Thực hiện toàn bộ trong một database transaction

---

## **4.4. Đồng bộ trạng thái bàn**

- AVAILABLE → khi không còn Order ACTIVE

- OCCUPIED → khi có ít nhất một Order ACTIVE

- CLEANING → khi Order vừa PAID

- Không cho chỉnh trạng thái thủ công nếu còn Order ACTIVE

---

# **5. Business Rules (Sprint 3)**

- BR-O01: Không cho gộp/tách/chuyển nếu Order đã PAID.

- BR-O02: Không cho thao tác nếu Order LOCKED.

- BR-O03: Không cho chuyển món đã COMPLETED (theo policy).

- BR-O04: Merge/Split/Move phải thực hiện trong một transaction.

- BR-O05: Sau merge, Order phụ → MERGED.

- BR-O06: Nếu Order nguồn không còn món → tự động CLOSED.

- BR-O07: Không thao tác với bàn INACTIVE.

---

# **6. Tiêu chí Chấp nhận (Acceptance Criteria)**

---

### **AC-O31: Gộp Order thành công**

- OrderItem chuyển chính xác

- Order phụ → MERGED

- Tổng tiền tính lại đúng

---

### AC-O32: Tách / Chuyển OrderItem thành công

Given: Order đang ACTIVE\
When: Cashier chọn món và chuyển sang Order khác\
Then:

- OrderItem được chuyển chính xác

- Tổng tiền của cả hai Order được cập nhật đúng

- Không mất dữ liệu

- Trạng thái KDS không thay đổi

---

### AC-O33: Tạo Order mới khi bàn đích chưa có Order

Given: Bàn đích chưa có Order\
When: Cashier thực hiện tách\
Then:

- Hệ thống tạo Order mới

- OrderItem được gán đúng TableId

- Bàn đích chuyển OCCUPIED

---

### AC-O34: Đóng Order nguồn khi không còn món

Given: Order nguồn không còn OrderItem\
When: Hoàn tất thao tác\
Then:

- Order nguồn chuyển CLOSED

- Nếu không còn Order khác → bàn nguồn chuyển AVAILABLE

-
