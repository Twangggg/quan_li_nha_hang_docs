# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem)**

Hiện tại, hệ thống FoodHub cần cho phép khách hàng đặt bàn trực tiếp từ trang chủ (không cần đăng nhập) để:

- Chọn ngày và giờ nhận bàn.

- Lựa chọn loại tiệc (Tiệc, Hẹn hò ăn tối, Cầu hôn, Khác).

- Chọn số lượng người và thông tin trẻ em đi cùng.

- Chọn khu vực (Tầng 1, Tầng 2, VIP).

- Chọn menu combo/set hoặc gọi món tại nhà hàng.

- Gửi ghi chú yêu cầu setup (ví dụ: trang trí cầu hôn, phòng riêng, sinh nhật…).

Hệ thống cần đảm bảo:

- Không cho đặt ngày quá khứ.

- Không cho đặt hôm nay nếu chưa đủ 45 phút trước giờ hiện tại.

- Chỉ cho đặt bàn ACTIVE.

- Một bàn tại một thời điểm chỉ có 1 reservation active.

---

## **Pain Points hiện tại**

- Không có hệ thống đặt bàn online → phải gọi điện thủ công.

- Dễ xảy ra trùng lịch đặt.

- Không kiểm soát được thời gian tối thiểu trước khi nhận bàn.

- Không có dữ liệu chuẩn về loại tiệc và yêu cầu setup.

- Không kiểm soát được bàn INACTIVE vẫn bị đặt.

---

## **Giá trị Nghiệp vụ (Business Value)**

- Tăng trải nghiệm khách hàng.

- Giảm tải nhân viên nhận đặt bàn qua điện thoại.

- Quản lý lịch đặt tập trung và minh bạch.

- Tăng khả năng bán combo/set trước.

- Thu thập dữ liệu nhu cầu theo khu vực và khung giờ.

---

## **Đối tượng (Actor)**

Primary Actor: Khách hàng (Public – chưa đăng nhập)\
Secondary Actor: Hệ thống FoodHub

---

## **User Story Statement**

Là khách hàng, tôi muốn đặt bàn trực tuyến bằng cách chọn ngày, giờ, khu vực và loại tiệc để đảm bảo có chỗ khi đến nhà hàng.

---

# **2. Luồng Người dùng (User Flow)**

---

## **2.1. Luồng chính: Nhập thông tin đặt bàn**

1. Khách truy cập trang chủ.

2. Click nút “Đặt bàn”.

3. Hệ thống hiển thị form:

### **Thông tin đặt bàn**

- Ngày đặt (date picker):

  - Không cho chọn ngày quá khứ.

- Giờ nhận bàn:

  - Danh sách từ 09:00 đến 20:00.

  - Nếu đặt hôm nay → phải ≥ thời điểm hiện tại + 45 phút.

- Loại tiệc:

  - Tiệc

  - Hẹn hò ăn tối

  - Cầu hôn

  - Khác

- Số lượng người.

- Checkbox “Có trẻ em đi cùng”.

- Ghi chú yêu cầu setup (optional).

---

## **2.2. Chọn khu vực**

- Tầng 1

- Tầng 2

- VIP

Hệ thống:

- Chỉ hiển thị bàn ACTIVE.

- Chỉ hiển thị bàn còn trống trong khung giờ đã chọn.

- Nếu chọn VIP → hiểu là đặt phòng riêng.

---

## **2.3. Chọn Menu**

- Chọn combo/set có sẵn.

- Hoặc chọn “Gọi món tại nhà hàng”.

- Có filter theo khoảng giá.

---

## **2.4. Xác nhận thông tin**

Khách nhập:

- Họ và tên.

- Số điện thoại.

Click “Xác nhận đặt bàn”.

Hệ thống:

- Validate dữ liệu.

- Tạo Reservation với status = BOOKED.

- Hiển thị thông báo thành công.

---

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

AC-PR-01: Không cho chọn ngày quá khứ\
AC-PR-02: Nếu đặt hôm nay thì giờ nhận bàn ≥ hiện tại + 45 phút\
AC-PR-03: Không cho đặt bàn INACTIVE\
AC-PR-04: Không cho đặt trùng thời gian trên cùng bàn\
AC-PR-05: Reservation được tạo với status BOOKED\
AC-PR-06: Nếu chọn VIP thì Reservation phải gắn khu VIP