**User Story – Quản lý Thực đơn**

**1. Tổng quan & Bối cảnh (Overview & Context)**

**Vấn đề (Problem)**

Hệ thống POS/Order hiện tại cần một cơ chế **quản lý thực đơn tập trung, rõ ràng và linh hoạt** để:

- Quản lý danh sách món ăn theo danh mục (Khai vị, Món chính, Đồ uống, Tráng miệng)
- Quản lý các nhóm đặc biệt như **Combo, Set Morning, Set Lunch** theo luồng riêng
- Đảm bảo **mỗi món chỉ thuộc một danh mục**
- Hỗ trợ tìm kiếm nhanh món ăn theo **mã món**
- Lọc món theo **khoảng giá**
- Kiểm soát trạng thái **Out of Stock** để tránh đặt nhầm món đã hết
- Đảm bảo thông tin món ăn đầy đủ, chính xác phục vụ cho order, vận hành bếp và báo cáo

**Pain Points hiện tại**

- Chưa có màn hình quản lý thực đơn tập trung và thống nhất
- Không kiểm soát tốt trạng thái món đã hết (Out of Stock)
- Thiếu phân quyền rõ ràng trong việc thêm/sửa/xóa món
- Giá bán và giá vốn chưa được tách bạch theo vai trò người dùng
- Chưa chuẩn hóa option món (topping, đường đá, độ cay, ghi chú)

**Giá trị Nghiệp vụ (Business Value)**

- **Quản lý menu tập trung**: Giúp Manager dễ dàng kiểm soát toàn bộ thực đơn trên hệ thống.
- **Giảm sai sót khi order**: Tránh việc nhân viên bán món đã hết hoặc sai giá.
- **Hỗ trợ vận hành bếp**: Thông tin station (bếp nóng/lạnh/bar) và thời gian dự kiến giúp điều phối món hiệu quả.
- **Kiểm soát tài chính**: Giá vốn chỉ hiển thị cho đúng vai trò (Manager, Cashier), chuẩn bị cho việc tính lợi nhuận sau này.
- **Mở rộng linh hoạt**: Sẵn sàng bổ sung báo cáo lợi nhuận, log lịch sử chỉnh sửa trong tương lai.

**Đối tượng (Actors)**

**Primary Actor**

- **Manager**
  - Thêm / sửa / xóa món ăn
  - Quản lý trạng thái Out of Stock
  - Chỉnh sửa giá bán (trước hoặc sau order, không chỉnh tại thời điểm order)

**Secondary Actors**

- **Cashier**
  - Xem danh sách món
  - Xem giá bán
  - Xem giá vốn (chỉ xem, không chỉnh)
- **Staff / Order Taker**
  - Xem menu
  - Thực hiện order theo menu hiện có

**User Story Statement**

Là **Manager**, tôi muốn có một màn hình **quản lý thực đơn tập trung** (tạo, xem, cập nhật, quản lý trạng thái Out of Stock) để đảm bảo menu luôn chính xác, phục vụ hiệu quả cho quy trình order, vận hành bếp và quản lý doanh thu.

**2. Luồng Người dùng (User Flow)**

**2.1. Luồng chính: Quản lý danh sách món ăn**

1. Người dùng đăng nhập hệ thống
2. Điều hướng đến trang **Quản lý Thực đơn** (/menu)
3. Hệ thống hiển thị danh sách món ăn với các thông tin:
   - Hình ảnh món
   - Tên món
   - Mã món
   - Danh mục (Khai vị, Món chính, Đồ uống, Tráng miệng)
   - Giá bán (ăn tại chỗ / mang đi nếu có)
   - Station (Bếp nóng / Bếp lạnh / Bar)
   - Trạng thái (Available / Out of Stock)
4. Người dùng có thể:
   - **Search theo mã món**
   - **Filter theo khoảng giá**
   - Filter theo danh mục
   - Phân trang danh sách
5. Mỗi dòng món có các hành động (theo quyền):
   - Xem chi tiết
   - Chỉnh sửa (Manager)
   - Bật/Tắt Out of Stock (Manager)

**2.2. Luồng: Tạo món ăn mới**

1. Manager click **“Thêm món”**
2. Hệ thống hiển thị form tạo món với các trường bắt buộc:
   - Mã món (bắt buộc, unique)
   - Tên món (bắt buộc)
   - Hình ảnh (bắt buộc)
   - Mô tả
   - Danh mục (chỉ chọn 1)
   - Giá bán:
     - Ăn tại chỗ
     - Mang đi (nếu áp dụng phí bao bì)
   - Giá vốn (chỉ Manager/Cashier thấy)
   - Station (Bếp nóng / Lạnh / Bar)
   - Thời gian dự kiến
3. Click **“Lưu”**
4. Hệ thống validate:
   - Trường bắt buộc không được để trống
   - Mã món không trùng
5. Nếu hợp lệ:
   - Tạo món thành công
   - Hiển thị thông báo “Thêm món thành công”
   - Cập nhật danh sách
6. Nếu không hợp lệ:
   - Hiển thị lỗi chi tiết
   - Giữ nguyên dữ liệu đã nhập

**2.3. Luồng: Cập nhật món ăn**

1. Manager chọn món cần chỉnh sửa
2. Click **“Chỉnh sửa”**
3. Cập nhật thông tin (trừ các rule bị khóa nếu có)
4. Click **“Lưu”**
5. Hệ thống:
   - Validate dữ liệu
   - Không cho sửa giá tại **thời điểm order**
   - Chỉ Manager được sửa giá ngoài thời điểm order
6. Cập nhật thành công và hiển thị thông báo

**2.4. Luồng: Tùy chọn món khi order**

Khi order một món, hệ thống hiển thị các option:

- Topping thêm
- Mức đường đá: 0% / 30% / 50%
- Độ cay: 0 – 3
- Ghi chú tự do
- **Không bắt buộc chọn option**

**2.5. Luồng: Quản lý Out of Stock**

1. Khi món hết, Manager bật trạng thái **Out of Stock**
2. Món vẫn hiển thị trong menu nhưng có nhãn “Out of Stock”
3. Món không thể được chọn để order
4. Chỉ Manager có quyền bật/tắt trạng thái này

**3. Tiêu chí Chấp nhận (Acceptance Criteria)**

**AC-01: Hiển thị danh sách thực đơn**\
Danh sách hiển thị đầy đủ thông tin món, có search theo mã món, filter theo giá và danh mục.

**AC-02: Thêm món mới thành công**\
Món được tạo với đầy đủ field bắt buộc, hiển thị ngay trên danh sách.

**AC-03: Không cho tạo món trùng mã**\
Hệ thống báo lỗi nếu mã món đã tồn tại.

**AC-04: Phân quyền giá vốn**\
Chỉ Manager và Cashier nhìn thấy giá vốn.

**AC-05: Quản lý Out of Stock**\
Món Out of Stock không thể order nhưng vẫn hiển thị.

**AC-06: Option món không bắt buộc**\
Order có thể hoàn tất mà không cần chọn option.

**AC-07: Không sửa giá tại thời điểm order**\
Hệ thống chặn chỉnh giá khi đang order.
