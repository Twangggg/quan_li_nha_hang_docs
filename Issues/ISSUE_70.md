# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Hệ thống FoodHub cần cho phép Manager và Cashier quản lý danh sách đặt bàn để:

- Xem danh sách reservation theo ngày.

- Lọc theo khu vực.

- Hủy hoặc chỉnh sửa đặt bàn.

- Theo dõi tình trạng khách đến hoặc không đến.

---

## **Pain Points hiện tại**

- Không có màn hình quản lý tập trung đặt bàn.

- Khó theo dõi bàn nào đã được đặt trước.

- Không phân biệt rõ trạng thái BOOKED, CHECKED_IN, CANCELLED.

- Không có lịch sử đặt bàn để đối chiếu.

---

## **Giá trị Nghiệp vụ (Business Value)**

- Kiểm soát toàn bộ lịch đặt bàn trong ngày.

- Tránh overbooking.

- Hỗ trợ vận hành giờ cao điểm.

- Dễ dàng thống kê số lượng khách đặt.

---

## **Đối tượng (Actor)**

Primary Actor: Manager / Cashier\
Secondary Actor: Waiter (view-only)

---

## **User Story Statement**

Là Manager/Cashier, tôi muốn xem và quản lý danh sách đặt bàn để kiểm soát lịch và xử lý khi khách đến hoặc hủy đặt.

---

# **2. Luồng Người dùng (User Flow)**

---

## **2.1. Xem danh sách Reservation**

1. User đăng nhập.

2. Truy cập trang /reservations.

3. Hệ thống hiển thị:

   - Tên khách

   - SĐT

   - Ngày

   - Giờ

   - Khu vực

   - Loại tiệc

   - Số người

   - Trạng thái (BOOKED / CHECKED_IN / CANCELLED / NO_SHOW)

Có thể:

- Lọc theo ngày.

- Lọc theo khu vực.

- Tìm kiếm theo tên/SĐT.

---

## **2.2. Hủy Reservation**

1. Chọn Reservation có trạng thái BOOKED.

2. Bấm “Hủy”.

3. Confirm.

4. Reservation → CANCELLED.

---

## **2.3. Chỉnh sửa Reservation (trước check-in)**

- Cho phép sửa:

  - Giờ

  - Số khách

  - Ghi chú

- Không cho sửa khi đã CHECKED_IN.

---

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

AC-RM-01: Hiển thị danh sách theo ngày\
AC-RM-02: Có thể lọc theo khu vực\
AC-RM-03: Hủy Reservation thành công nếu chưa CHECKED_IN\
AC-RM-04: Không cho chỉnh sửa khi đã CHECKED_IN\
AC-RM-05: Phân quyền – Waiter chỉ xem