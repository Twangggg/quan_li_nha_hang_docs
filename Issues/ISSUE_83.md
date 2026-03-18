**1. Tổng quan & Bối cảnh (Overview & Context)**

**Vấn đề (Problem):**

Hiện tại, hệ thống FoodHub chỉ hỗ trợ thanh toán tiền mặt đơn giản (MVP). Nhà hàng cần mở rộng thanh toán để:

\- Hỗ trợ thanh toán chuyển khoản ngân hàng qua QR Code (VietQR), giảm thao tác thủ công cho thu ngân và khách.

\- Cho phép thanh toán 1 hóa đơn bằng nhiều phương thức cùng lúc (Split Payment), ví dụ: tiền mặt + chuyển khoản.

\- Quản lý danh sách phương thức thanh toán (thêm/sửa/bật/tắt, cấu hình tài khoản ngân hàng).

\- Xem báo cáo doanh thu theo từng phương thức để đối soát cuối ca.

**Pain Points hiện tại:**

\- Chỉ có 1 phương thức thanh toán (tiền mặt) — không đáp ứng nhu cầu thực tế.

\- Khi khách muốn chuyển khoản, thu ngân phải đưa QR in sẵn → khách tự nhập số tiền → dễ sai.

\- Không ghi nhận được khách thanh toán bằng phương thức nào → cuối ca không đối soát được.

\- Khách muốn chia tiền (1 phần tiền mặt, 1 phần CK) → hệ thống không hỗ trợ, thu ngân phải ghi sổ tay.

**Giá trị Nghiệp vụ (Business Value)**:

1\. Trải nghiệm khách hàng tốt hơn: khách quét QR → số tiền + nội dung đã điền sẵn → chỉ cần xác nhận.

2\. Giảm sai sót: hệ thống tự sinh QR với số tiền chính xác, không phụ thuộc khách nhập tay.

3\. Linh hoạt thanh toán: cho phép chia tiền bằng nhiều phương thức trong 1 hóa đơn.

4\. Kiểm soát tài chính: báo cáo doanh thu theo phương thức giúp đối soát cuối ca nhanh chóng.

5\. Dễ mở rộng: sau này thêm VNPay, MoMo, ZaloPay chỉ cần thêm phương thức mới.

**Đối tượng (Actor):**

\- Primary Actor: Cashier / Waiter (Thu ngân / Nhân viên phục vụ)

\- Thao tác thanh toán hóa đơn, chọn phương thức, hiển thị QR cho khách, xác nhận đã nhận tiền.

\- Secondary Actors:

\- Manager (Quản lý nhà hàng):

\- Cấu hình phương thức thanh toán (thêm/sửa/bật/tắt).

\- Xem báo cáo doanh thu theo phương thức.

\- Khách hàng:

\- Quét QR Code để thanh toán chuyển khoản (không tương tác trực tiếp với hệ thống).

**User Story Statement:**

Là Cashier/Waiter, tôi muốn thanh toán hóa đơn bằng nhiều phương thức (tiền mặt, chuyển khoản qua QR Code), có thể chia tiền thanh toán nhiều cách trong 1 hóa đơn, để phục vụ khách nhanh hơn và ghi nhận chính xác phương thức thanh toán.

**2. Luồng Người dùng (User Flow)**

**2.1. Luồng: Quản lý Phương thức Thanh toán (Manager)**

1\. Manager vào Cài đặt/Quản lý hệ thống → Phương thức thanh toán (/settings/payment-methods)

2\. Hệ thống hiển thị danh sách phương thức đã có:

\- Tên phương thức

\- Loại (CASH / BANK_TRANSFER / E_WALLET / OTHER)

\- Trạng thái (Bật / Tắt)

\- Thông tin tài khoản (nếu loại BANK_TRANSFER): Ngân hàng, Số TK, Tên chủ TK

3\. Manager có thể:

\- Thêm phương thức mới

\- Sửa thông tin phương thức

\- Bật/Tắt phương thức (Tắt → phương thức không hiện lên khi thanh toán)

Lưu ý:

\- Phương thức "Tiền mặt" được tạo sẵn mặc định, không cho xóa, luôn bật.

\- Khi thêm phương thức loại BANK_TRANSFER, bắt buộc nhập: Tên ngân hàng, Số tài khoản, Tên chủ tài khoản.

\- BIN ngân hàng (Bank Identification Number) cần nhập để sinh QR VietQR đúng chuẩn.

**2.2. Luồng: Thêm phương thức thanh toán mới**

1\. Manager bấm "Thêm phương thức"

2\. Hệ thống hiển thị form:

\- Tên phương thức (bắt buộc) — VD: "Chuyển khoản Vietcombank"

\- Loại (bắt buộc) — chọn: Tiền mặt / Chuyển khoản / Ví điện tử / Khác

\- Trạng thái: Bật (mặc định)

\- Nếu loại = Chuyển khoản:

\- Tên ngân hàng (bắt buộc) — VD: "Vietcombank"

\- BIN ngân hàng (bắt buộc) — VD: "970436"

\- Số tài khoản (bắt buộc) — VD: "0123456789"

\- Tên chủ tài khoản (bắt buộc) — VD: "NGUYEN VAN A"

3\. Manager điền thông tin và bấm "Lưu"

4\. Hệ thống validate:

\- Tên phương thức không được trống

\- Nếu Chuyển khoản: phải có đủ thông tin ngân hàng

5\. Nếu hợp lệ:

\- Tạo phương thức mới

\- Hiển thị thông báo "Thêm phương thức thanh toán thành công"

\- Refresh danh sách

6\. Nếu không hợp lệ:

\- Hiển thị lỗi cụ thể dưới từng trường

\- Form vẫn mở

**2.3. Luồng: Thanh toán Tiền mặt (giữ nguyên flow hiện tại)**

1\. Cashier/Waiter mở hóa đơn của Order đang SERVING

2\. Bấm "Thanh toán"

3\. Hệ thống hiển thị màn thanh toán:

\- Tổng tiền hóa đơn

\- Phương thức: mặc định "Tiền mặt"

\- Số tiền khách đưa (nhập tay)

\- Tiền thừa (hệ thống tự tính = Khách đưa - Tổng tiền)

4\. Bấm "Xác nhận thanh toán"

5\. Hệ thống:

\- Ghi nhận thanh toán với phương thức = Tiền mặt

\- Đổi trạng thái Order → COMPLETED

\- Toast "Thanh toán thành công"

**2.4. Luồng: Thanh toán Chuyển khoản (QR Code VietQR)**

1\. Cashier/Waiter mở hóa đơn, bấm "Thanh toán"

2\. Chọn phương thức "Chuyển khoản" (chọn từ danh sách phương thức đang Bật, loại BANK_TRANSFER)

3\. Hệ thống tự sinh QR Code theo chuẩn VietQR chứa:

\- BIN ngân hàng (từ cấu hình phương thức)

\- Số tài khoản (từ cấu hình)

\- Số tiền = tổng hóa đơn (hoặc số tiền được gán cho phương thức này nếu split)

\- Nội dung chuyển khoản = mã hóa đơn (VD: "HD-20260311-0005")

4\. Cashier đưa màn hình QR cho khách quét

5\. Khách mở app ngân hàng → quét QR → thông tin điền sẵn → xác nhận chuyển khoản

6\. Cashier kiểm tra tiền đã về tài khoản (kiểm tra thủ công qua app NH)

7\. Bấm "Xác nhận đã nhận tiền"

8\. Hệ thống:

\- Ghi nhận thanh toán với phương thức = Chuyển khoản

\- Đổi trạng thái Order → COMPLETED

\- Toast "Thanh toán thành công"

Lưu ý:

\- QR Code sinh theo chuẩn VietQR (EMVCo), không cần tích hợp API ngân hàng.

\- Xác nhận là thủ công (Cashier bấm "Đã nhận tiền"). Tích hợp webhook tự động nằm ngoài scope v1.

**2.5. Luồng: Thanh toán Nhiều phương thức (Split Payment)**

1\. Cashier/Waiter mở hóa đơn (tổng 500.000đ), bấm "Thanh toán"

2\. Hệ thống hiển thị màn thanh toán:

\- Tổng tiền hóa đơn: 500.000đ

\- Mặc định 1 dòng phương thức: Tiền mặt — 500.000đ

3\. Cashier bấm "Thêm phương thức" để chia tiền:

\- Dòng 1: Tiền mặt — nhập 200.000đ

\- Dòng 2: Chuyển khoản — nhập 300.000đ (hệ thống sinh QR cho 300.000đ)

4\. Hệ thống kiểm tra:

\- Tổng các dòng phải = Tổng hóa đơn (200k + 300k = 500k → hợp lệ)

\- Nếu chưa khớp → hiển thị "Còn thiếu X đồng" hoặc "Vượt X đồng"

\- Không cho xác nhận khi chưa khớp

5\. Khi khớp → Cashier bấm "Xác nhận thanh toán"

6\. Đối với dòng Chuyển khoản: Cashier cần bấm "Đã nhận tiền" riêng cho dòng đó (sau khi kiểm tra)

7\. Hệ thống:

\- Ghi nhận chi tiết từng dòng thanh toán (phương thức + số tiền)

\- Order → COMPLETED

\- Toast "Thanh toán thành công"

Lưu ý:

\- Được phép thêm tối đa N dòng (N = số phương thức đang Bật).

\- Mỗi phương thức chỉ xuất hiện 1 lần.

\- Số tiền mỗi dòng phải &gt; 0.

**2.6. Luồng: Xem Báo cáo Doanh thu theo Phương thức (Manager)**

1\. Manager vào Báo cáo → Doanh thu theo phương thức (/reports/revenue-by-payment)

2\. Lọc theo:

\- Khoảng thời gian (từ ngày — đến ngày, mặc định = hôm nay)

\- Phương thức cụ thể (tùy chọn)

3\. Hệ thống hiển thị bảng:

\- Phương thức | Số giao dịch | Tổng tiền | Tỷ lệ %

\- Dòng tổng cộng ở cuối

4\. Có thể xem chi tiết từng phương thức → danh sách hóa đơn đã thanh toán bằng phương thức đó

**3. Tiêu chí Chấp nhận (Acceptance Criteria)**

AC-01: Hiển thị danh sách phương thức thanh toán

Given: Manager đã đăng nhập

When: Truy cập /settings/payment-methods

Then:

\- Hiển thị danh sách phương thức: Tên, Loại, Trạng thái, Thông tin TK (nếu có)

\- Phương thức "Tiền mặt" luôn tồn tại và không cho xóa

AC-02: Thêm phương thức Chuyển khoản thành công

Given: Manager ở trang Phương thức thanh toán

When: Bấm "Thêm", chọn loại "Chuyển khoản", nhập đầy đủ thông tin ngân hàng (tên NH, BIN, số TK, tên chủ TK), bấm "Lưu"

Then:

\- Phương thức được tạo, trạng thái Bật

\- Toast "Thêm phương thức thanh toán thành công"

\- Phương thức xuất hiện trong dropdown khi thanh toán

AC-03: Thêm thất bại — thiếu thông tin ngân hàng

Given: Manager chọn loại "Chuyển khoản" nhưng bỏ trống số tài khoản

When: Bấm "Lưu"

Then:

\- Lỗi "Số tài khoản là bắt buộc"

\- Không tạo, form vẫn mở

AC-04: Bật/Tắt phương thức

Given: Phương thức "MoMo" đang Bật

When: Manager bấm "Tắt"

Then:

\- Phương thức chuyển sang Tắt

\- Khi Cashier thanh toán, "MoMo" không xuất hiện trong danh sách chọn

AC-05: Thanh toán tiền mặt

Given: Order đang SERVING, tổng 350.000đ

When: Cashier chọn "Tiền mặt", nhập khách đưa 400.000đ, bấm "Xác nhận"

Then:

\- Hiển thị tiền thừa: 50.000đ

\- Ghi nhận thanh toán Tiền mặt 350.000đ

\- Order → COMPLETED

\- Toast "Thanh toán thành công"

AC-06: Thanh toán chuyển khoản — hiển thị QR

Given: Đã có phương thức "CK Vietcombank" (BIN 970436, STK 0123456789)

When: Cashier chọn "CK Vietcombank", tổng 350.000đ

Then:

\- Hệ thống sinh QR Code VietQR chứa: BIN, STK, số tiền 350000, nội dung = mã hóa đơn

\- Hiển thị QR trên màn hình để khách quét

\- Có nút "Xác nhận đã nhận tiền"

AC-07: Xác nhận chuyển khoản

Given: QR đã hiển thị, khách đã chuyển khoản

When: Cashier bấm "Xác nhận đã nhận tiền"

Then:

\- Ghi nhận thanh toán Chuyển khoản

\- Order → COMPLETED

AC-08: Split Payment — thanh toán 2 phương thức

Given: Order tổng 500.000đ

When: Cashier thêm 2 dòng: Tiền mặt 200.000đ + Chuyển khoản 300.000đ

Then:

\- Hệ thống kiểm tra 200k + 300k = 500k → hợp lệ

\- Sinh QR cho dòng Chuyển khoản (số tiền 300.000đ)

\- Sau khi xác nhận cả 2 dòng → Order COMPLETED

\- Ghi nhận 2 bản ghi thanh toán riêng

AC-09: Split Payment — tổng chưa khớp

Given: Order tổng 500.000đ

When: Cashier nhập Tiền mặt 200.000đ + Chuyển khoản 100.000đ (tổng = 300k)

Then:

\- Hiển thị "Còn thiếu 200.000đ"

\- Nút "Xác nhận" bị disable

AC-10: Báo cáo doanh thu theo phương thức

Given: Có các hóa đơn đã thanh toán trong ngày

When: Manager xem /reports/revenue-by-payment, lọc hôm nay

Then:

\- Hiển thị bảng: Phương thức | Số giao dịch | Tổng tiền | Tỷ lệ %

\- Có dòng tổng cộng

\- Bấm vào phương thức → xem danh sách hóa đơn chi tiết

AC-11: Phân quyền

Given: User role Waiter

When: Truy cập /settings/payment-methods

Then:

\- Bị chặn 403 hoặc không hiển thị menu Cài đặt

\- Waiter chỉ được thao tác thanh toán, không được cấu hình phương thức

AC-12: QR Code sinh đúng chuẩn VietQR

Given: Phương thức CK có BIN = 970436, STK = 0123456789

When: Cashier thanh toán CK 350.000đ cho hóa đơn HD-20260311-0005

Then:

\- QR Code khi quét bằng app ngân hàng hiển thị đúng:

\- Ngân hàng: Vietcombank

\- Số TK: 0123456789

\- Số tiền: 350.000đ

\- Nội dung: HD-20260311-0005
