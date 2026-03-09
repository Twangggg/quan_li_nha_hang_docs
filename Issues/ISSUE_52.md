**USER STORY  – BÁO CÁO (Analytics)**

**1. Tổng quan & Bối cảnh (Overview & Context)**

**Vấn đề (Problem)**

Hệ thống POS cần module Báo cáo để:

- Theo dõi **tổng doanh thu**
- Xem báo cáo theo **ca / ngày / tháng**
- Phân tích **món bán chạy (Best Sellers)**
- Hỗ trợ chốt ca và ra quyết định kinh doanh

**Pain Points**

- Khó đối soát tiền mặt khi chốt ca
- Thiếu báo cáo tổng hợp theo thời gian
- Chưa khai thác dữ liệu món bán chạy hiệu quả

**Giá trị Nghiệp vụ (Business Value)**

- Kiểm soát doanh thu minh bạch
- Hỗ trợ Manager ra quyết định chính xác
- Tối ưu menu và vận hành

**2. Đối tượng (Actors)**

**Primary Actors**

- **Thu ngân (Cashier)**
  - Xem báo cáo theo ca
  - Chốt ca
- **Quản lý (Manager)**
  - Xem toàn bộ báo cáo
  - Phân tích doanh thu & best sellers

**3. User Story Statement**

Là **Manager/Cashier**, tôi muốn xem các báo cáo doanh thu và món bán chạy theo thời gian, nhằm kiểm soát hoạt động kinh doanh và đưa ra quyết định phù hợp.

**4. Luồng Báo Cáo (Report Flows)**

**4.1. Báo cáo Doanh thu theo Ngày (Daily Report)**

- Tổng doanh thu ngày
- Tổng số đơn
- Doanh thu theo từng ca
- So sánh với mục tiêu ngày

**4.2. Báo cáo Doanh thu theo Tháng (Monthly Report)**

- Tổng doanh thu tháng
- So sánh tháng trước / cùng kỳ
- Tỷ lệ tăng trưởng

**4.3. Báo cáo Món Bán Chạy (Best Sellers)**

- Tên món
- Số lượng bán ra
- **Tổng doanh thu theo món**
- Tỷ trọng doanh thu (%)
- Lợi nhuận gộp (nếu có)
- Lọc theo ngày / tháng

**5. Tiêu chí Chấp nhận (Acceptance Criteria)**

- **AC-R01:** Cashier chỉ xem báo cáo ca
- **AC-R02:** Manager xem đầy đủ báo cáo
- **AC-R03:** Báo cáo ca/ngày/tháng hiển thị đúng doanh thu
- **AC-R04:** Best Sellers thống kê chính xác theo thời gian