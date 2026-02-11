## **1. Tổng quan & Bối cảnh (Overview & Context)**

### **Vấn đề (Problem):**

Phần mềm cần cơ chế đăng nhập và quản lý tài khoản tập trung để:

- Cho phép người dùng truy cập đúng chức năng theo vai trò (Manager / Cashier / Waiter / Chef-Bar). Đảm bảo bảo mật, hạn chế truy cập trái phép.

- Cho phép Login và sử dụng các chức năng cho phép vai trò của mình sử dụng.

- Nhân viên (Cashier/Waiter/Chef-Bar/Manager) có thể logout khi hết ca.

- Cho phép nhân viên tự đổi mật khẩu, thay đổi thông tin cá nhân.

- Có thể thay đổi mật khẩu qua email nếu quên.

### **Pain Points hiện tại**

- Chưa có nơi quản lý tài khoản tập trung, sử dụng chung tài khoản khó truy vết được ai đã thao tác/hoạt động trong nhà hàng.

- Quên mật khẩu gây gián đoạn ca làm, chưa có luồng reset nhanh.

- Chưa có màn hình profile chuẩn, thông tin liên hệ không đồng bộ.

### **Giá trị Nghiệp vụ (Business Value)**

- Kiểm soát truy cập theo vai trò: đúng người đúng quyền, giảm sai sót nghiệp vụ.

- Giảm gián đoạn vận hành: nhân viên quên mật khẩu có thể reset nhanh qua email hoặc qua Manager reset khi cần.

- Quản lý thông tin cá nhân: Nhân viên có thể tự thay đổi thông tin cá nhân trừ các mục đã khóa.

### Note

Nhân viên bao gồm Manager / Cashier / Waiter / Chef-Bar.

## Đối tượng (Actor)

### Primary Actors

- Người dùng phần mềm (Cashier/Waiter/Chef-Bar/Manager): đăng nhập/đăng xuất và tự đổi mật khẩu, cập nhật thông tin cá nhân.

- Manager: có thể reset mật khẩu cho tất cả role khi cần.

### Secondary Actors

- None

## **User Story Statement**

Là người dùng phần mềm (Cashier/Waiter/Chef-Bar/Manager), tôi muốn đăng nhập/đăng xuất, quản lý mật khẩu và thông tin cá nhân, và (với Manager) reset mật khẩu khi cần để đảm bảo vận hành nhà hàng an toàn, đúng quyền và không gián đoạn ca làm.

# **2. Luồng Người dùng (User Flow)**

## **2.1. Luồng chính: Đăng nhập**

1. Người dùng truy cập vào phần mềm, phần mềm yêu cầu Login trước khi sử dụng.

2. Hệ thống hiển thị form gồm:
   - Employee Code (format: W/C/B/M + 6 digits)

   - Password

   - (Optional) “Remember me”

3. Người dùng nhập thông tin và bấm Đăng nhập.

4. Hệ thống validate:
   - Không để trống username(Employee code)/password

   - (Optional) rate limit/captcha sau nhiều lần sai.

5. Nếu hợp lệ:
   - Hệ thống xác thực thành công.

   - Tạo phiên đăng nhập:

   - Điều hướng về Dashboard theo role:

6. Nếu không hợp lệ:
   - Hiển thị lỗi chung: “Sai tài khoản hoặc mật khẩu”

## **2.2. Luồng: Đăng xuất (Logout)**

1. Người dùng bấm Logout ở trang Dashboard

2. Hệ thống xóa token (hoặc revoke token) theo cơ chế xác thực.

3. Điều hướng về trang Login.

## **2.3. Luồng: Người dùng tự đổi mật khẩu**

1. Người dùng đăng nhập vào phần mềm, bấm Account Settings ở sidebar, Đổi mật khẩu.

2. Form gồm:
   - Mật khẩu hiện tại (Current password)

   - Mật khẩu mới (New password)

   - Nhập lại mật khẩu mới (Confirm password)

3. Người dùng bấm Lưu.

4. Hệ thống validate:
   - Mật khẩu hiện tại đúng

   - New password đạt yêu cầu policy (độ dài, ký tự…): 8 chữ cái bao gồm chữ và số + 1 ký tự đặc biệt

   - Confirm trùng new password

5. Nếu hợp lệ:
   - Hệ thống cập nhật mật khẩu.

   - Hiển thị thông báo: “Đổi mật khẩu thành công”

   - Yêu cầu đăng nhập lại sau khi đổi pass

6. Nếu không hợp lệ:
   - Hiển thị lỗi cụ thể dưới từng field.

## **2.5. Luồng: Người dùng chỉnh sửa thông tin cá nhân (Profile)**

1. Người dùng đăng nhập thành công và trên menu sidebar bấm Account Settings.

2. Hệ thống hiển thị thông tin hiện tại (read + editable fields):
   - Họ và tên (editable)

   - Số điện thoại (editable)

   - Email (editable)

   - Avatar (optional)

   - Employee code(read-only)

   - Vai trò (Role) (read-only)

   - Trạng thái tài khoản (read-only)

   - Trường khác: địa chỉ, ngày sinh…

3. Người dùng chỉnh sửa và bấm **Lưu**.

4. Hệ thống validate:
   - Họ tên không rỗng (required)

   - SĐT đúng định dạng (VN)

   - Email đúng định dạng

   - SĐT cần unique\*\*

5. Nếu hợp lệ:
   - Hệ thống cập nhật thông tin

   - Hiển thị toast: **“Cập nhật thông tin thành công”**

   - Refresh dữ liệu (không reload toàn trang)

6. Nếu không hợp lệ:
   - Hiển thị lỗi dưới từng field

   - Giữ lại dữ liệu đã nhập để sửa tiếp

**Rule/giới hạn (để tránh lạm quyền)**

- Người dùng chỉ được sửa profile của chính mình.

- Không được sửa: role, Employee code, trạng thái.

## **2.6. Luồng: Quên mật khẩu / Reset mật khẩu qua Email Link**

### **2.6.1. Luồng yêu cầu reset (Request reset link)**

1. Tại màn Login, người dùng bấm “Quên mật khẩu?”.

2. Hệ thống hiển thị form:
   - EmployeeCode

3. Người dùng nhập thông tin và bấm “Gửi link đặt lại mật khẩu”.

4. Hệ thống validate:
   - Trường nhập không được trống

5. Nếu hợp lệ:
   - Hệ thống tạo Reset Token (one-time token) + TTL

   - Tạo Reset Link dạng:\
     https://\[domain\]/reset-password?token=...

   - Gửi email tới địa chỉ email đã lưu của tài khoản

   - Hiển thị thông báo:
     - Khuyến nghị dùng message chung để bảo mật:\
       “Nếu tài khoản hợp lệ, hệ thống đã gửi link đặt lại mật khẩu qua email.”

Trường hợp đặc biệt

- Nếu tài khoản Inactive:
  - Hệ thống hiển thị message chung hoặc chặn rõ ràng

### **2.6.2. Luồng mở link & đặt mật khẩu mới (Open link + Set new password)**

1. Người dùng mở email và click Reset Link.

2. Hệ thống mở trang Reset Password (public page) gồm:
   - Mật khẩu mới

   - Nhập lại mật khẩu mới

3. Hệ thống validate token trước khi cho submit:
   - Token đúng\
     Token chưa hết hạn (TTL)

   - Token chưa bị sử dụng (one-time)

4. Người dùng nhập mật khẩu mới và bấm “Xác nhận”.

5. Hệ thống validate:
   - Mật khẩu mới đúng policy: 8 ký tự gồm chữ + số + 1 ký tự đặc biệt

   - Confirm trùng mật khẩu mới

6. Nếu hợp lệ:
   - Hệ thống cập nhật mật khẩu mới\
     Invalidate token (đánh dấu đã dùng)

   - Hủy tất cả token đang còn hiệu lực

   - Điều hướng về Login + toast: “Đặt lại mật khẩu thành công”

7. Nếu không hợp lệ:
   - Hiển thị lỗi cụ thể:
     - “Link không hợp lệ hoặc đã hết hạn”

     - “Mật khẩu không đạt policy”

     - “Xác nhận mật khẩu không khớp”

**Rule/giới hạn đề xuất (Security)**

- Giới hạn số lần yêu cầu gửi reset link theo thời gian: \[TBD: ví dụ 3 lần / 10 phút\]

- Token one-time: link chỉ dùng 1 lần.

- Không lộ user tồn tại: luôn hiển thị message chung “Nếu tài khoản hợp lệ…”

- DB có **Unique Index** cho EmployeeCode.

- Tạo mã dùng **sequence/atomic increment** để tránh trùng khi tạo đồng thời.

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

## **AC-LOGIN-01: Đăng nhập thành công**

**Given**: Người dùng có tài khoản Active do Manager tạo

**When**: Nhập đúng \[employeeCode\] và password, bấm “Đăng nhập”

**Then**:

- Hệ thống đăng nhập thành công

- Tạo phiên đăng nhập (JWT) hợp lệ

- Điều hướng vào màn hình phù hợp với role

## **AC-LOGIN-02: Đăng nhập thất bại**

**Given**: Người dùng ở màn Login

**When**: Nhập sai username (Employee code) hoặc password và bấm “Đăng nhập”

**Then**:

- Hệ thống không đăng nhập

- Hiển thị lỗi: “Sai tài khoản hoặc mật khẩu”

## **AC-LOGIN-03: Không cho truy cập khi chưa login**

**Given**: Người dùng chưa đăng nhập

**When**: Truy cập URL nội bộ (vd /students, /orders, /kds…)\
**Then**:

- Bị redirect về /login

AC-AUTH-04: Người dùng đổi mật khẩu thành công

**Given**: Người dùng đã đăng nhập

**When**: Nhập đúng mật khẩu hiện tại + mật khẩu mới hợp lệ và bấm “Lưu”

**Then**:

- Mật khẩu được cập nhật

- Hiển thị “Đổi mật khẩu thành công”

- Yêu cầu đăng nhập lại

## **AC-AUTH-05: Đổi mật khẩu thất bại do mật khẩu hiện tại sai**

**Given**: Người dùng đang đổi mật khẩu

**When**: Nhập sai mật khẩu hiện tại

**Then**:

- Không cập nhật mật khẩu

- Hiển thị lỗi: “Mật khẩu hiện tại không đúng”

## **AC-AUTH-06: Manager reset mật khẩu thành công**

**Given**: Manager đã đăng nhập

**When**: Thực hiện reset mật khẩu một tài khoản bất kỳ

**Then**:

- Mật khẩu user được reset

- Ghi nhận audit resetBy/resetAt

- (Optional) buộc user đổi mật khẩu ở lần đăng nhập tiếp theo

## **AC-AUTH-07: Phân quyền theo vai trò**

**Given**: Có user với role khác nhau

**When**: Truy cập chức năng không thuộc quyền

**Then**:

- Hệ thống trả 403 hoặc hiển thị “Không có quyền truy cập”

## **AC-AUTH-08: Logout thành công**

**Given**: user đã đăng nhập

**When**: bấm Logout

**Then**: token bị xóa (client), redirect /login, không truy cập được trang nội bộ (401/redirect)

## **AC-AUTH-09: Cập nhật thông tin cá nhân thành công**

**Given**: Người dùng đang ở trang Profile

**When**: Chỉnh sửa thông tin hợp lệ và bấm “Lưu”

**Then**:

- Hệ thống cập nhật dữ liệu thành công

- Hiển thị “Cập nhật thông tin thành công”

- Dữ liệu hiển thị sau lưu đúng với dữ liệu mới

## **AC-AUTH-10: Cập nhật thất bại do dữ liệu không hợp lệ**

**Given**: Người dùng đang chỉnh profile

**When**: Nhập email sai định dạng hoặc SĐT sai định dạng rồi bấm “Lưu”

**Then**:

- Không cập nhật dữ liệu

- Hiển thị lỗi cụ thể tại field sai

- Giữ dữ liệu đã nhập

## **AC-AUTH-11: Không cho sửa thông tin của người khác**

**Given**: Người dùng A đã đăng nhập

**When**: Cố truy cập /profile/{userId khác} hoặc gọi API update user khác

**Then**:

- Trả về thông báo lỗi 404, tùy thiết kế FE, không để thông báo lỗi mặc định

- Không thay đổi dữ liệu

**AC-AUTH-12: Yêu cầu reset link thành công (message chung)**\
Given: user ở màn Login\
When: nhập EmployeeCode hợp lệ và bấm “Gửi link đặt lại mật khẩu”\
Then:

- Hiển thị message chung “Nếu tài khoản hợp lệ…”

- Phần mềm tạo token + gửi email

## **AC-AUTH-13: Reset mật khẩu thành công qua link**

**Given**: user mở link reset còn hạn, chưa dùng

**When**: nhập mật khẩu mới hợp lệ + confirm đúng và submit

**Then**:

- Mật khẩu cập nhật

- Token bị invalidate

- Redirect login

- Toast thành công

## **AC-AUTH-14: Reset thất bại do link hết hạn/đã dùng/không hợp lệ**

**Given**: link sai/hết hạn/đã dùng

**When**: user mở link hoặc submit

**Then**:

- Báo “Link không hợp lệ hoặc đã hết hạn”

- Không đổi mật khẩu
