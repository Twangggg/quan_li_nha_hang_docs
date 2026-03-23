**USER STORY – AUDIT LOG**

**1. Tổng quan & Bối cảnh (Overview & Context)**

**Vấn đề (Problem)**\
Hiện tại hệ thống FoodHub chưa có cơ chế ghi nhận lịch sử thao tác (Audit Log), dẫn đến:

- Không theo dõi được **ai đã thực hiện hành động gì** (tạo/sửa/xóa reservation).
- Khó kiểm tra khi xảy ra **trùng lịch, sai dữ liệu hoặc khiếu nại**.
- Không có dữ liệu để **truy vết (trace)** và kiểm soát thay đổi hệ thống.

Hệ thống cần Audit Log để:

- Ghi lại các hành động quan trọng:
  - Tạo / cập nhật / hủy reservation
  - Thay đổi trạng thái bàn
  - Thay đổi thông tin khách hàng
- Lưu thông tin:
  - Thời gian (timestamp)
  - Hành động (action)
  - Đối tượng (entity: Reservation, Table…)
  - Dữ liệu trước và sau khi thay đổi (old/new value)
  - Người thực hiện (user hoặc system)

**Business Rules**

- Mỗi hành động quan trọng phải tạo **1 log record**.
- Log **không được chỉnh sửa hoặc xóa**.
- Log phải lưu theo thứ tự thời gian.
- Các hành động từ Public user (đặt bàn không đăng nhập) vẫn phải được log (gắn theo phone/name).

---

**Pain Points hiện tại**

- Không truy vết được lỗi hoặc tranh chấp đặt bàn.
- Không xác định được ai đã thay đổi dữ liệu.
- Thiếu minh bạch trong vận hành.
- Khó debug và kiểm tra hệ thống.

---

**Giá trị Nghiệp vụ (Business Value)**

- Tăng khả năng **kiểm soát và minh bạch hệ thống**.
- Hỗ trợ **debug và xử lý sự cố nhanh**.
- Cung cấp dữ liệu cho **kiểm toán (audit)**.
- Giảm rủi ro sai lệch dữ liệu.

---

**Đối tượng (Actor)**

- **Primary Actor:** Hệ thống FoodHub (tự động ghi log)
- **Secondary Actor:** manager / Staff (xem log)

---

**User Story Statement**

Là manager, tôi muốn ghi lại tất cả các hành động quan trọng để có thể theo dõi, kiểm tra và truy vết khi cần thiết.

---

**2. Luồng Hệ thống (System Flow)**

**2.1. Ghi log tự động**

Khi có hành động xảy ra:

- Tạo Reservation
- Cập nhật Reservation
- Hủy Reservation
- Thay đổi trạng thái bàn

Hệ thống sẽ:

- Capture thông tin:
  - Action (CREATE / UPDATE / DELETE)
  - Entity (Reservation, Table…)
  - EntityId
  - Timestamp
  - User (nếu có) hoặc thông tin khách (phone/name)
  - OldData (nếu có)
  - NewData
- Lưu vào bảng **AuditLog**

---

**2.2. Xem Audit Log (Manager)**

Admin truy cập màn hình Audit Log:

- Filter theo:
  - Thời gian
  - Loại hành động
  - Entity (Reservation, Table…)
- Hiển thị:
  - Danh sách log theo timeline
  - Chi tiết thay đổi (before/after)

---

**3. Tiêu chí Chấp nhận (Acceptance Criteria)**

- **AC-AL-01:** Mỗi hành động CREATE/UPDATE/DELETE phải được ghi log.
- **AC-AL-02:** Log phải chứa: action, entity, entityId, timestamp, user/actor.
- **AC-AL-03:** Với UPDATE phải lưu cả old value và new value.
- **AC-AL-04:** Log không được phép chỉnh sửa hoặc xóa.
- **AC-AL-05:** Log phải được lưu theo thứ tự thời gian.
- **AC-AL-06:** Hành động từ khách không đăng nhập vẫn phải được ghi log.
