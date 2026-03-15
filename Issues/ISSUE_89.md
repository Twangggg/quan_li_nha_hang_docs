User Story: Xem và Xuất PDF Phiếu Tạm Tính (Pre-check Bill)

1. Tổng quan & Bối cảnh (Overview & Context)

Vấn đề (Problem):\
Hiện tại, khi khách hàng muốn thanh toán, thu ngân hoặc phục vụ cần một phương thức để khách hàng kiểm tra lại danh sách món trước khi xuất hóa đơn chính thức (Invoice). Tuy nhiên, dự án không triển khai thiết bị máy in nhiệt phần cứng.

Pain Points hiện tại:

- Thu ngân/phục vụ thiếu công cụ để khách kiểm tra đối chiếu món (kiểm đồ) một cách chuyên nghiệp.

- Không có bước đệm xác nhận "Pre-check" an toàn cho luồng thanh toán, dễ dẫn đến phải hủy/hoàn hóa đơn chính thức nếu có sai sót.

- Không hỗ trợ xuất file mềm (PDF) để lưu trữ tạm hoặc gửi cho khách (qua Zalo/Email) nếu khách yêu cầu.

Giá trị Nghiệp vụ (Business Value):

1. Minh bạch thông tin: Khách hàng thấy được bản xem trước (Preview) chi tiết phiếu tạm tính ngay trên màn hình thu ngân, rõ ràng và trực quan.

2. Lưu trữ linh hoạt: Khả năng xuất ra file PDF giúp nhân viên có thể gửi file mềm cho khách đối chiếu mà không cần in giấy.

3. Giảm thiểu sai sót & tranh chấp: Thu ngân và khách đồng kiểm thông qua phiếu thể hiện đầy đủ giá, số lượng, tránh việc tính nhầm món hoặc quên hủy món, loại bỏ rủi ro sai sót chứng từ kế toán.

Đối tượng (Actor):

- Primary Actor: Thu ngân (Cashier), Nhân viên phục vụ (Waiter)

User Story Statement:\
Là Thu ngân (hoặc Phục vụ), tôi muốn xem bản xem trước của phiếu tạm tính trên màn hình và có thể xuất ra file PDF cho một đơn hàng đang phục vụ để khách hàng có thể kiểm tra danh sách món, số lượng, tổng tiền trước khi tiến hành thanh toán chính thức.

---

2. Luồng Người dùng (User Flow)

2.1. Luồng chính: Xem trước và Xuất PDF phiếu tạm tính

1. Cashier/Waiter đăng nhập vào hệ thống.

2. Tại màn hình "Order", chọn một bàn đang ở trạng thái "Đang phục vụ".

3. Tại khu vực chứa nút chức năng (cạnh nút Thanh toán), click vào nút "Xem Phiếu Tạm Tính" (hoặc icon xem trước).

4. Hệ thống kiểm tra: Đơn hàng phải có ít nhất 1 món ở trạng thái hợp lệ (chưa bị hủy).

5. Modal/Dialog "Phiếu Tạm Tính" hiện lên giữa màn hình, hiển thị đầy đủ thiết kế của một tờ phiếu (giống hóa đơn giấy) với tiêu đề "PHIẾU TẠM TÍNH". Lưu ý: Không gọi API tạo Invoice mới ở Backend.

6. Trên Modal này có một nút chức năng: "Xuất PDF" (hoặc "Tải xuống PDF").

7. Cashier/Waiter có thể đưa màn hình cho khách xem, hoặc nhấn nút "Xuất PDF".

8. Trình duyệt tự động tạo và tải xuống file PDF (ví dụ tên file: Phieu_Tam_Tinh_Ban5_14032026.pdf).

9. Cashier/Waiter đóng Modal để quay lại màn hình đơn hàng bình thường. Trạng thái bàn và đơn hàng không thay đổi.

2.2. Luồng ngăn chặn: Đơn hàng trống không thể xem phiếu

1. Cashier/Waiter chọn một bàn vừa mở (chưa gọi món) hoặc tất cả các món đều mang trạng thái "Đã hủy".

2. Hệ thống làm mờ (disable) nút "Xem Phiếu Tạm Tính"; hoặc nếu click được, hệ thống hiển thị thông báo lỗi (Toast message): "Đơn hàng trống, không thể tạo phiếu tạm tính".

### 2.3. Luồng điều chỉnh món sau khi kiểm đồ

1. Cashier/Waiter mở Modal "Phiếu Tạm Tính" để khách kiểm tra món.

2. Trong danh sách món hiển thị, mỗi dòng món có thêm các nút:
   - \[-\] Giảm số lượng

   - \[+\] Tăng số lượng

3. Khi khách phát hiện:
   - Dư món: Cashier nhấn \[-\] để giảm số lượng.

   - Thiếu món: Cashier nhấn \[+\] để tăng số lượng.

4. Hệ thống cập nhật lại tổng tiền ngay lập tức (real-time) trên UI.

5. Các thay đổi này được áp dụng trực tiếp vào Order hiện tại (không tạo Invoice).

6. Sau khi chỉnh sửa xong, Cashier có thể:
   - Đóng Modal

   - Hoặc Xuất lại PDF phiếu tạm tính đã cập nhật.

---

3. Tiêu chí Chấp nhận (Acceptance Criteria)

AC-01: Hiển thị chức năng "Xem Phiếu Tạm Tính" trên UI

Given: Cashier đang ở màn hình Chi tiết đơn hàng của một bàn đang hoạt động.\
When: Nhìn vào các nút thao tác gần khu vực Tổng tiền/Thanh toán.\
Then: Phải hiển thị rõ nút bấm hoặc icon \[Xem Phiếu Tạm Tính\].

AC-02: Hiển thị Modal Preview chính xác thông tin (Happy Case)

Given: Đơn hàng hiện tại có tối thiểu 1 món hợp lệ.\
When: Cashier click nút \[Xem Phiếu Tạm Tính\].\
Then:

- Một UI Modal xuất hiện đè lên màn hình hiện tại.

- Về mặt DB: Không có bản ghi Invoice nào được sinh ra.

- Trong Modal, giao diện phiếu phải hiển thị chuẩn xác:
  - Header: Dòng tiêu đề nổi bật "PHIẾU TẠM TÍNH" (không được dùng chữ Hóa đơn).

  - Thông tin chung: Số bàn, Tên nhân viên thu ngân, Giờ in phiếu.

  - Danh sách món: Tên món, Số lượng, Đơn giá, Thành tiền (chỉ tính những món không bị cancel).

  - Footer: Tổng tiền, Giảm giá (nếu có), Phụ phí/VAT (nếu có), và Số tiền khách cần trả.

  - Ghi chú: Có dòng chữ "Đây không phải là hóa đơn thanh toán".

AC-03: Tính năng xuất/tải file PDF hoạt động đúng chuẩn

Given: Cashier đang mở Modal Phiếu Tạm Tính.\
When: Cashier click vào nút \[Xuất PDF\] (hoặc icon Download).\
Then:

- Hệ thống tự động chuyển đổi giao diện phiếu trên Modal thành một file định dạng .pdf.

- Trình duyệt tự động tải file xuống (hoặc mở sang tab mới dạng PDF để xem).

- Tên file sinh ra tự động phải có ý nghĩa (ví dụ: TamTinh_Ban05_14h30.pdf).

- File PDF tải về phải giữ nguyên bố cục chữ, font, format y hệt như những gì nhìn thấy trên Modal.

AC-04: Xử lý ngoại lệ với đơn hàng không hợp lệ

Given: Bàn được chọn chưa có món nào hoặc tất cả món đã hủy.\
When: Nhìn vào nút \[Xem Phiếu Tạm Tính\].\
Then: Nút bấm bị vô hiệu hóa (disabled) hoặc khi click sẽ hiển thị cảnh báo đỏ "Không thể in, đơn hàng đang trống!", Modal không xuất hiện.

AC-05: Cập nhật real-time không giới hạn

Given: Bàn X đã xem phiếu tạm tính 1 lần, sau đó khách gọi thêm món mới.\
When: Cashier mở lại modal \[Xem Phiếu Tạm Tính\].\
Then: Modal bắt buộc phải hiển thị dữ liệu (số lượng món, tổng tiền) mới nhất, tức thời theo giỏ hàng hiện tại.
