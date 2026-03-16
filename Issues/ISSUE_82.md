========================================

US-INV-06: Tính giá vốn xuất kho & Cảnh báo kho (13 points)

========================================

User Story Statement:

Là Manager, tôi muốn hệ thống tính giá vốn xuất kho theo phương pháp Bình quân gia quyền và hiển thị cảnh báo NVL sắp hết/hết hạn, để kiểm soát chi phí và chủ động bổ sung hàng.

Tiền điều kiện: US-INV-01, US-INV-02, US-INV-03 hoàn thành.

Luồng Người dùng:

1\. Tính giá vốn xuất kho (COGS — Bình quân gia quyền)

Phương pháp: Weighted Average

Công thức:

Đơn giá BQ = (Giá trị tồn đầu kỳ + Giá trị nhập trong kỳ) / (SL tồn đầu kỳ + SL nhập trong kỳ)

Quy tắc:

\- Đơn giá BQ tính lại mỗi khi có Phiếu nhập kho mới

\- Xuất kho → đơn giá xuất = đơn giá BQ tại thời điểm xuất

\- Giá trị xuất = SL xuất x Đơn giá BQ

\- Chưa nhập lần nào → đơn giá = 0

Hiển thị:

\- Phiếu xuất kho: tự tính đơn giá xuất theo BQ

\- Báo cáo tồn kho: hiển thị đơn giá BQ + giá trị tồn

1.1. Tính lại giá vốn cho 1 kỳ

1\. Manager vào Quản lý Kho → Tính giá xuất kho (/inventory/cogs)

2\. Chọn kỳ: từ ngày — đến ngày (tối đa 31 ngày)

3\. Chọn NVL hoặc "Tất cả"

4\. Bấm "Tính giá"

5\. Hệ thống tính lại đơn giá BQ, cập nhật thành tiền trên phiếu xuất trong kỳ

6\. Toast "Tính giá vốn hoàn tất"

2\. Cảnh báo kho (Inventory Alerts)

1\. Manager vào Quản lý Kho → Cảnh báo (/inventory/alerts) hoặc thấy badge trên sidebar

2\. Tab "Sắp hết / Hết hàng":

\- NVL CurrentStock = 0 → lên trên cùng

\- NVL 0 &lt; CurrentStock &lt;= LowStockThreshold

\- Mỗi dòng: Tên NVL | Tồn hiện tại | Ngưỡng | Đơn vị

3\. Tab "Sắp hết hạn / Đã hết hạn":

\- Lô có ExpiryDate &lt;= ngày hiện tại + 7 ngày

\- Mỗi dòng: Tên NVL | Lô | Hạn SD | Số ngày còn lại

4\. Badge tổng cảnh báo hiển thị trên icon kho ở sidebar

Acceptance Criteria:

AC-01: Tính giá vốn BQ

Given: "Thịt bò" tồn đầu 5000g giá 200.000đ/kg, nhập thêm 3000g giá 180.000đ/kg

When: Hệ thống tính đơn giá BQ

Then:

\- Đơn giá BQ = (5000 x 200 + 3000 x 180) / (5000 + 3000) = 192.500đ/kg

\- Xuất 1000g → giá trị = 192.500đ

AC-02: Tính lại giá vốn theo kỳ

Given: Có phiếu xuất trong kỳ 01/03 — 10/03

When: Manager chọn kỳ, bấm "Tính giá"

Then:

\- Tính lại đơn giá BQ cho kỳ

\- Cập nhật thành tiền phiếu xuất

\- Toast "Tính giá vốn hoàn tất"

AC-03: Kỳ tính giá tối đa 31 ngày

Given: Manager chọn kỳ 01/01 — 15/02 (46 ngày)

When: Bấm "Tính giá"

Then: Lỗi "Kỳ tính giá tối đa 31 ngày"

AC-04: Cảnh báo NVL sắp hết

Given: "Hành tây" ngưỡng = 200g, tồn = 150g

When: Xem Cảnh báo

Then: "Hành tây" xuất hiện tab "Sắp hết"

AC-05: Cảnh báo NVL hết hàng

Given: "Ớt chuông" tồn = 0

When: Xem Cảnh báo

Then: "Ớt chuông" lên đầu tab "Hết hàng"

AC-06: Cảnh báo sắp hết hạn

Given: Lô "Thịt bò" hạn SD = ngày hiện tại + 3 ngày

When: Xem tab "Sắp hết hạn"

Then: Hiển thị "Thịt bò — Lô xxx — Còn 3 ngày"

AC-07: Badge cảnh báo

Given: Có 5 NVL sắp hết + 2 lô sắp hết hạn

When: Manager nhìn sidebar

Then: Badge "7" trên icon kho

AC-08: Phân quyền

Given: User role Waiter

When: Truy cập /inventory/\* hoặc gọi API

Then: Bị chặn 403 hoặc read-only
