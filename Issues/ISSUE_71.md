# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Khi khách đến nhà hàng, Reservation cần được chuyển thành Order DineIn để bắt đầu quy trình phục vụ.

Hệ thống cần đảm bảo:

- Không tạo trùng Order.

- Đồng bộ trạng thái bàn.

- Tự động ưu tiên nếu là VIP.

---

## **Pain Points hiện tại**

- Tạo Order thủ công khi khách đến.

- Dễ nhầm bàn.

- Không đồng bộ trạng thái bàn.

- Không liên kết giữa Reservation và Order.

---

## **Giá trị Nghiệp vụ (Business Value)**

- Check-in nhanh chóng.

- Đồng bộ Reservation → Order.

- Giảm thao tác cho nhân viên.

- Quản lý chính xác luồng khách.

---

## **Đối tượng (Actor)**

Primary Actor: Cashier / Manager\
Secondary Actor: Waiter (tuỳ quyền)

---

## **User Story Statement**

Là Cashier/Manager, tôi muốn check-in Reservation và hệ thống tự động tạo Order DineIn tương ứng để bắt đầu phục vụ khách.

---

# **2. Luồng Người dùng (User Flow)**

---

## **2.1. Check-in khách**

1. Mở Reservation trạng thái BOOKED.

2. Click “Check-in”.

3. Hệ thống:
   - Tạo Order:
     - OrderType = DineIn

     - TableId tương ứng

     - Status = DRAFT

     - IsPriority = true nếu VIP

   - Reservation → CHECKED_IN

   - Bàn → OCCUPIED

4. Redirect sang màn Order Detail.

---

## **2.2. Trường hợp đặc biệt**

- Không cho check-in nếu:
  - Reservation đã CANCELLED.

  - Reservation đã CHECKED_IN.

- Không cho check-in nếu bàn đang OCCUPIED bởi Order khác.

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

AC-RC-01: Check-in tạo Order thành công\
AC-RC-02: Reservation chuyển CHECKED_IN\
AC-RC-03: Bàn chuyển OCCUPIED\
AC-RC-04: VIP auto set IsPriority = true\
AC-RC-05: Không cho check-in nếu trạng thái không hợp lệ
