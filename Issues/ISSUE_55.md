## **1. Tổng quan & Bối cảnh (Overview & Context)**

### **Vấn đề (Problem)**

Nhà hàng cần một màn hình quản lý nhân viên tập trung để:

- Manager tạo tài khoản nhân viên mới và gán đúng vai trò (Cashier/Waiter/Chef-Bar/Manager).

- Xem danh sách nhân viên và chi tiết thông tin từng nhân viên.

- Cập nhật thông tin nhân viên (hồ sơ, liên hệ, vai trò theo rule).

- Quản lý trạng thái hoạt động (Active/Inactive) để kiểm soát truy cập hệ thống.

- Hỗ trợ vận hành nhanh khi có nhân viên mới/nghỉ việc.

- Cho phép Manager xem lịch sử thao tác (audit log) cho các hành động quan trọng: tạo/cập nhật/vô hiệu hóa, reset mật khẩu, đổi role để truy vết.

### **Pain Points hiện tại**

- Không có nơi quản lý nhân viên tập trung → khó kiểm soát ai có quyền truy cập hệ thống.

- Dễ dùng chung tài khoản hoặc quên thu hồi quyền khi nhân viên nghỉ → rủi ro bảo mật.

- Thông tin liên hệ nhân viên rời rạc, cập nhật không đồng bộ.

- Tạo tài khoản mới thủ công mất thời gian, dễ sai role.

### **Giá trị Nghiệp vụ (Business Value)**

- Quản trị tập trung: Manager nắm toàn bộ danh sách nhân sự và quyền truy cập.

- Giảm rủi ro: khóa/vô hiệu hóa tài khoản nhanh khi cần.

- Vận hành mượt: onboard nhân viên mới nhanh, đúng quyền.

- Truy vết rõ ràng: phân quyền đúng giúp giảm sai thao tác nghiệp vụ.

- Audit minh bạch: Manager xem được ai thao tác gì, khi nào, lý do → dễ kiểm soát và xử lý sự cố.

## **2. Đối tượng (Actor)**

### **Primary Actor**

- **Manager**: có toàn quyền quản lý nhân viên (CRUD + phân quyền + trạng thái).

### **Secondary Actors (tùy scope)**

- **Cashier/Waiter/Chef-Bar**: không truy cập module này (403). *(TBD: có cần “xem danh sách liên hệ” read-only không?)*

## **3. User Story Statement**

Là Manager, tôi muốn quản lý danh sách nhân viên (tạo mới, xem danh sách, xem chi tiết, cập nhật, vô hiệu hóa) để kiểm soát quyền truy cập và vận hành nhà hàng hiệu quả.

## **4. Luồng Người dùng (User Flow)**

### **4.1. Luồng chính: Xem danh sách nhân viên**

1. Manager đăng nhập.

2. Vào màn **Staff Management** (ví dụ: /staff).

3. Hệ thống hiển thị danh sách nhân viên với các cột tối thiểu:

   - Mã nhân viên/EmployeeCode (read-only)

   - Họ tên

   - Role

   - SĐT

   - Email

   - Trạng thái (Active/Inactive)

   - Ngày tạo / cập nhật gần nhất *(TBD)*

4. Hỗ trợ thao tác:

   - Search: theo tên / mã / SĐT / email (debounce 300ms)

   - Filter: theo role, theo trạng thái

   - Sort: theo tên A-Z, ngày tạo

   - Pagination: 10/20/50

### **4.2. Luồng: Xem chi tiết nhân viên**

1. Từ danh sách, Manager bấm **Xem chi tiết**.

2. Hiển thị trang/modal gồm:

   - Thông tin cơ bản: họ tên, mã NV, role, trạng thái

   - Thông tin liên hệ: SĐT, email, địa chỉ, ngày sinh…

   - Audit: createdAt/createdBy, updatedAt/updatedBy

### **4.2.1. Luồng: Xem Audit Log (Activity History)**

- Từ trang chi tiết nhân viên, Manager mở tab/section **Audit Log / Activity**.

- Hệ thống hiển thị danh sách log theo thời gian (mới nhất trước), gồm tối thiểu:

  - Action (Create / Update / Deactivate / ResetPassword / ChangeRole)

  - Actor (ai thực hiện)

  - Time (timestamp)

  - Reason (nếu có: reset/change role)

  - Metadata (oldRole/newRole, oldEmployeeCode/newEmployeeCode… nếu có)

- Hỗ trợ (optional theo UI bạn muốn): filter theo action, pagination.

### **4.3. Luồng: Manager tạo tài khoản (Account Provisioning)**

1. Manager vào màn “Quản lý tài khoản nhân viên”.

2. Bấm + Tạo tài khoản.

3. Hệ thống hiển thị form gồm:

   - Họ tên nhân viên

   - Employee Code.

   - Email (required)

   - SĐT (required)

   - Địa chỉ/Ngày sinh

   - Vai trò: Manager/Cashier/Waiter/Chef-Bar

   - Trạng thái: Active/Inactive (default là Active)

   - Mật khẩu:

     - Manager tự đặt password ban đầu

     - Hệ thống auto-generate password và hiển thị Manager bấm Lưu.

4. Hệ thống validate:

   - Employee Code theo định dạng quy định

   - Email định dạng đúng

   - SDT unique, theo format Việt Nam

   - Role hợp lệ

5. Nếu thành công:

   - Tạo User account

   - Thông báo: “Tạo tài khoản thành công”

   - Gửi thông tin đăng nhập cho nhân viên qua email

6. Nếu thất bại:

   - Thông báo lỗi cụ thể.

**Rule:**

- Employee Code theo định dạng: **W: waiter / C: Cashier / B: Chef&Bar / M: Manager + 6 số tăng dần theo role. Ex:** Nhà hàng có 2 Manager M000001, M000002, 10 Waiter W000001, W000002,...\*

### **4.4. Luồng: Cập nhật thông tin nhân viên (Edit Staff)**

Manager mở chi tiết nhân viên → bấm Chỉnh sửa.

**Cho phép sửa:**

- Họ tên, SĐT, email, địa chỉ, ngày sinh…

- Trạng thái Active/Inactive

**Không cho sửa:**

- EmployeeCode (read-only)

- **Role (đổi role làm bằng luồng riêng 4.8)**

Validate tương tự tạo mới.\
Lưu thành công → cập nhật dữ liệu + audit updatedAt/updatedBy.

### **4.5. Luồng: Vô hiệu hóa/Khóa nhân viên (Deactivate Staff)**

1. Manager chọn nhân viên → bấm **Deactivate/Inactive**.

2. Confirm.

3. Hệ thống chuyển trạng thái sang Inactive.

4. Rule:

   - Nhân viên Inactive **không đăng nhập được**.

   - Revoke token/session hiện tại nếu đang đăng nhập

### **4.6. Luồng: Xóa nhân viên (Delete Staff) — chỉ “*soft delete*”**

- Tại trang Quản lý tài khoản nhân viên/chi tiết nhân viên

- Bấm nút vô hiệu hóa tài khoản

**Khuyến nghị**: không xóa cứng, dùng Inactive/Archived để giữ lịch sử.

### **4.7. Luồng: Manager reset mật khẩu tài khoản bất kỳ**

1. Manager vào danh sách tài khoản.

2. Chọn một tài khoản → bấm Reset mật khẩu.\
   Hệ thống yêu cầu xác nhận (confirm).\
   Cách reset:

   - Manager đặt mật khẩu mới thủ công

   - Hệ thống generate mật khẩu mới

3. Hệ thống thực hiện reset, ghi nhận audit:

   - resetBy, resetAt

   - lý do reset (required)

4. Thông báo kết quả:

   - “Reset mật khẩu thành công”

   - Buộc user đổi mật khẩu khi đăng nhập lần đầu sau reset

### **4.8. Luồng: Đổi Role (Change Role) — tài khoản mới**

Manager vào chi tiết nhân viên → bấm **Change Role**.\
Hệ thống hiển thị form:

- Role hiện tại (read-only)

- Role mới (dropdown: Manager/Cashier/Waiter/Chef-Bar)

- Xác nhận thao tác (confirm)

Manager chọn Role mới và xác nhận.

Hệ thống thực hiện:

- **Tạo một tài khoản mới** với **Role mới**

- **Copy toàn bộ thông tin** từ tài khoản cũ sang tài khoản mới (họ tên, SĐT, email, địa chỉ, ngày sinh, avatar, …)

- **Generate EmployeeCode mới** theo role mới (W/C/B/M + 6 số tăng dần theo role)

- **Chuyển tài khoản cũ sang Inactive** để giữ lịch sử (soft)

- **Revoke token hiện tại** của tài khoản cũ nếu đang đăng nhập

- Ghi nhận audit: changedRoleBy, changedRoleAt, oldEmployeeCode, newEmployeeCode, oldRole, newRole

Sau đó hệ thống:

- **Gửi email confirm lại thông tin tài khoản** tới email của nhân viên (bao gồm: EmployeeCode mới, Role mới, thông tin liên hệ đã được copy)

Thông báo kết quả:

- “Đổi role thành công”

## **5. Business Rules**

- **BR-STAFF-01:** Chỉ **Manager** có quyền truy cập Staff Management.

- **BR-STAFF-02:** EmployeeCode/Username **readonly sau khi tạo**.

- **BR-STAFF-03:** Email và SĐT **unique** trong hệ thống.

- **BR-STAFF-04:** Nhân viên **Inactive** không thể đăng nhập và không xuất hiện ở các luồng gán ca/assign (nếu có).

- **BR-STAFF-05:** Hệ thống ghi log/audit cho CRUD: create/update/deactivate (actor + timestamp + reason nếu có).

- **BR-STAFF-06:** Đổi role là **chức năng riêng** (Change Role), không đổi trực tiếp trong Edit Staff.

- **BR-STAFF-07:** Khi đổi role, hệ thống **tạo tài khoản mới** + copy thông tin + generate EmployeeCode mới; tài khoản cũ chuyển **Inactive/Archived** để giữ lịch sử.

- **BR-STAFF-08:** EmployeeCode chỉ thay đổi thông qua luồng **Change Role** (không update trực tiếp).

- **BR-STAFF-09:** Manager có thể xem Audit Log của từng nhân viên (read-only).

- **BR-STAFF-10:** Audit Log ghi nhận các hành động: create/update/deactivate/reset password/change role (actor + timestamp + reason nếu có).

**6. Tiêu chí Chấp nhận (Acceptance Criteria)**

- **AC-STAFF-01: Xem danh sách nhân viên**

  - Given: Manager đã đăng nhập

  - When: truy cập /staff

  - Then: thấy danh sách + search/filter/sort/pagination.

- **AC-STAFF-02: Tạo nhân viên thành công**

  - Given: Manager ở màn /staff

  - When: nhập dữ liệu hợp lệ và lưu

  - Then: tạo staff + account, hiển thị thông báo thành công, record xuất hiện trong danh sách.

- **AC-STAFF-03: Tạo nhân viên thất bại do trùng email/SĐT**

  - Given: đã tồn tại email hoặc SĐT

  - When: tạo mới với email/SĐT trùng

  - Then: báo lỗi dưới field, không tạo record.

- **AC-STAFF-04: Xem chi tiết nhân viên**

  - Given: Manager xem danh sách

  - When: bấm “Xem chi tiết”

  - Then: hiển thị đầy đủ thông tin + audit (nếu có).

- **AC-STAFF-05: Cập nhật nhân viên thành công**

  - Given: Manager mở form edit

  - When: sửa thông tin hợp lệ và lưu

  - Then: dữ liệu cập nhật, updatedAt/updatedBy ghi nhận.

- **AC-STAFF-06: Không cho sửa EmployeeCode**

  - Given: Manager edit staff

  - When: cố sửa EmployeeCode/Username

  - Then: field readonly và API không cho update.

- **AC-STAFF-07: Vô hiệu hóa nhân viên**

  - Given: staff đang Active

  - When: Manager chuyển Inactive

  - Then: staff không login được; trạng thái hiển thị Inactive.

- **AC-STAFF-08: Phân quyền**

  - Given: user là Cashier/Waiter/Chef-Bar

  - When: truy cập /staff hoặc gọi API staff CRUD

  - Then: trả 403/“Không có quyền truy cập”.

- **AC-STAFF-09: Manager reset mật khẩu tài khoản bất kỳ**

Given: Manager đã đăng nhập và đang ở màn Staff Management (/staff)\
When: Manager chọn một tài khoản → bấm “Reset mật khẩu” → nhập **lý do reset (required)** → xác nhận (confirm) và thực hiện reset theo 1 trong 2 cách:

- Manager đặt mật khẩu mới thủ công, hoặc

- Hệ thống generate mật khẩu mới\
  Then:

- Hệ thống reset mật khẩu của user thành công

- Ghi nhận audit: **resetBy, resetAt, lý do reset**

- Hiển thị thông báo: “Reset mật khẩu thành công”

- Buộc user đổi mật khẩu khi đăng nhập lần đầu sau reset

**AC-STAFF-10: Đổi role thành công (tạo tài khoản mới)**\
Given: Manager đã đăng nhập và đang ở chi tiết nhân viên\
When: chọn Role mới và xác nhận Change Role\
Then:

- Hệ thống tạo **tài khoản mới** với role mới

- Copy thông tin từ tài khoản cũ sang tài khoản mới

- Tạo EmployeeCode mới theo đúng rule

- Tài khoản cũ bị chuyển Inactive/Archived

- Revoke session/token tài khoản cũ nếu đang đăng nhập

- Gửi email confirm thông tin tài khoản mới

**AC-STAFF-11: Không cho đổi role khi role mới trùng role cũ**\
Given: Manager đang đổi role\
When: chọn role mới = role hiện tại\
Then: không tạo tài khoản mới, hiển thị lỗi phù hợp

**AC-STAFF-12: EmployeeCode phải đúng format khi tạo mới/đổi role**\
When: hệ thống tạo EmployeeCode\
Then: EmployeeCode đúng định dạng (W/C/B/M + 6 digits)

**AC-STAFF-13: EmployeeCode unique + tăng dần theo role**\
When: tạo tài khoản mới (tạo mới hoặc đổi role)\
Then: EmployeeCode không trùng và tăng dần theo role (không duplicate khi tạo đồng thời)

**AC-STAFF-14: Xem Audit Log thành công**\
Given: Manager đã đăng nhập và đang ở trang chi tiết nhân viên\
When: mở tab/section “Audit Log / Activity”\
Then: hệ thống hiển thị danh sách log đúng theo thứ tự thời gian, có action/actor/time và reason/metadata (nếu có)

**AC-STAFF-15: Phân quyền Audit Log**\
Given: user là Cashier/Waiter/Chef-Bar\
When: truy cập API/UI Audit Log\
Then: trả 403 / “Không có quyền truy cập”