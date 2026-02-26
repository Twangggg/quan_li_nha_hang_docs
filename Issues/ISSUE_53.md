## **1. Tổng quan & Bối cảnh (Overview & Context)**

### **Vấn đề (Problem)**

Hệ thống FoodHub cần module Thanh toán để:

- Hoàn tất giao dịch của Order.

- Ghi nhận phương thức thanh toán.

- Cập nhật trạng thái Order chính xác.

- Đồng bộ trạng thái bàn sau thanh toán.

### **Pain Points hiện tại**

- Không có luồng thanh toán chuẩn hóa.

- Dễ thanh toán nhầm hoặc thanh toán nhiều lần.

- Không khóa Order sau khi thanh toán.

- Bàn không tự động chuyển trạng thái sau khi trả tiền.

### **Giá trị Nghiệp vụ (Business Value)**

- Chuẩn hóa quy trình thanh toán.

- Giảm sai sót và trùng giao dịch.

- Đồng bộ dữ liệu cho báo cáo doanh thu.

- Tự động cập nhật trạng thái bàn.

## **2. Đối tượng (Actor)**

### **Primary Actor**

**Cashier**

- Thực hiện thanh toán.

### **Secondary Actors**

**Manager**

- Xem giao dịch.

Waiter/Chef-Bar: không có quyền thanh toán.

## **3. User Story Statement**

Là Cashier, tôi muốn thanh toán hóa đơn bằng nhiều phương thức và hệ thống cập nhật trạng thái Order chính xác, để đảm bảo giao dịch hoàn tất và dữ liệu doanh thu đúng.

## **4. Luồng Người dùng (User Flow)**

### **4.1. Luồng chính: Thanh toán tiền mặt**

1. Cashier mở Order đang ACTIVE.

2. Click “Thanh toán”.

3. Hệ thống hiển thị:
   - Tổng tiền

   - Phương thức thanh toán

4. Cashier chọn “Tiền mặt”.

5. Nhập số tiền khách đưa.

6. Hệ thống tính tiền thối.

7. Confirm.

8. Hệ thống:
   - Lưu giao dịch

   - Cập nhật Order → PAID

   - Lock Order (không chỉnh sửa)

   - Cập nhật bàn → Cleaning

### **4.2. Luồng: Thanh toán QR**

1. Cashier chọn phương thức QR.

2. Hệ thống đánh dấu paymentMethod = QR

3. Order → PAID

4. Không tích hợp cổng thanh toán thật

(Integration gateway thật  làm ở Sprint sau.)

## **5. Tiêu chí Chấp nhận (Acceptance Criteria)**

### **AC-B01: Thanh toán tiền mặt thành công**

Given: Order đang ACTIVE\
When: Cashier chọn tiền mặt và xác nhận\
Then:

- Order chuyển trạng thái PAID

- Ghi nhận paymentMethod = CASH

- Lock Order

- Bàn chuyển sang Cleaning.

### **AC-B02: Không cho thanh toán 2 lần**

Given: Order đã PAID\
When: Cashier mở lại và bấm Thanh toán\
Then: Hệ thống không cho thanh toán lại.

### **AC-B03: Thanh toán QR thành công (QR mock)**

Given: Order đang ACTIVE\
When: Cashier chọn QR\
Then: Order chuyển PAID và lưu paymentMethod = QR.

### **AC-B04: Phân quyền**

Given: User là Waiter\
When: Truy cập chức năng thanh toán\
Then: Bị chặn 403 hoặc không hiển thị nút.
