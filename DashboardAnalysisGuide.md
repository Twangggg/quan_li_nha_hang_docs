# FoodHub Dashboard Analysis Guide

Tài liệu này chi tiết hóa các tính năng và chỉ số cần thiết cho Dashboard Manager dành cho phân tích (Analysis), từ vận hành hàng ngày đến chiến lược trung hạn.

## 1. Phân tầng ưu tiên triển khai (Implementation Roadmap)

### Giai đoạn 1: Sức khỏe vận hành & KPI cơ bản (MVP)

_Tập trung vào tính ổn định của hệ thống và các con số "nhìn là biết ngay" tình hình kinh doanh._

- **KPI Hiệu suất (Nhanh):**
  - **Doanh thu/Số đơn:** So sánh tức thời với ngày hôm qua/tuần trước (YoY/WoW).
  - **AOV (Average Order Value):** Theo dõi giá trị trung bình trên mỗi đơn để đánh giá hiệu quả up-sell.
  - **Tỉ lệ hủy/hoàn (Cực kì quan trọng):** Cần drill-down để biết lý do (Hết món, khách đổi ý, hay lỗi KDS).
- **Sức khỏe vận hành (Real-time Monitoring):**
  - **Thiết bị (Hardware Status):** Trạng thái kết nối POS, KDS, Máy in bill. Cảnh báo ngay khi một thiết bị mất kết nối.
  - **Đồng bộ dữ liệu:** Trạng thái đồng bộ của Kho và Doanh thu. Phát hiện sớm các lỗi API/Thanh toán (PayOS, Momo).
  - **Backlog KDS:** Tổng số món đang chờ xử lý. Nếu vượt ngưỡng SLA (ví dụ: > 20 món/bếp), Dashboard cần đổi màu cảnh báo.

---

### Giai đoạn 2: Quản trị trung hạn & Tối ưu (Core Analytics)

_Phân tích sâu hơn sau khi đã có dữ liệu tích lũy ổn định._

- **Phân tích Menu (Menu Engineering):**
  - **Stars/Plowhorses/Dogs:** Phân loại món theo độ phổ biến và lợi nhuận.
  - **Tỷ lệ bị hoàn/hủy theo món:** Phát hiện các món có vấn đề về chất lượng (quá mặn/lâu).
  - **Biên lợi nhuận ước tính:** Dựa trên định lượng (Recipe) và giá nhập kho thực tế.
- **Quản trị Kho & Cost:**
  - **Food Cost %:** Tỷ lệ chi phí nguyên liệu trên doanh thu theo ngày/tuần.
  - **Cảnh báo tồn kho đỏ:** Tự động dự báo các nguyên liệu sẽ hết dựa trên vận tốc bán (Velocity).
  - **Chênh lệch (Variance):** So sánh tồn kho thực tế (stock take) và tồn kho hệ thống để phát hiện thất thoát.
- **Bàn & Khu vực (Space Optimization):**
  - **Heatmap lấp đầy:** Khu vực nào khách thích ngồi nhất theo giờ.
  - **Thời lượng chiếm bàn (Table Turn):** Thời gian trung bình một lượt khách. Nếu quá thấp => Phục vụ nhanh nhưng khách chưa chill; Nếu quá cao => Khách ngồi lâu gây nghẽn lượt tiếp theo.

---

### Giai đoạn 3: Chiến lược & Trải nghiệm khách (Advanced)

_Sử dụng AI và các luồng dữ liệu bên ngoài._

- **Khách hàng & Kênh (Omnichannel):**
  - **Kênh mix:** Cân đối giữa tại chỗ (Dine-in) và giao hàng (Delivery). Biết được kênh nào đang gánh chi phí chiết khấu nhiều nhất.
  - **Khách quay lại (Retention):** Nếu có hệ thống Loyalty, đo lường tỉ lệ khách ghé thăm > 2 lần/tháng.
- **Nhân sự & Hiệu suất ca:**
  - **Nhân viên trực:** Danh sách nhân viên đang active.
  - **Productivity:** Doanh thu trung bình trên mỗi nhân viên phục vụ/bếp. Cảnh báo thiếu nhân sự nếu dự báo đơn hàng cao.
- **Dự báo (Forecasting):** Dự báo số đơn hàng cho ngày mai dựa trên dữ liệu lịch sử và các yếu tố ngoại cảnh (cuối tuần, lễ tết).

---

## 2. Thiết kế Visual & Trải nghiệm Dashboard (UX/UI)

### Bố cục 3 Vùng (Three-Zone Principle)

1.  **Vùng Header (Control Center):** Đặt các bộ lọc nhanh (Ca, Khu vực, Khoảng thời gian) và nút "Auto-refresh".
2.  **Vùng Tiêu điểm (The Hook):** Các Widget thẻ (Card) có màu sắc tương ứng với trạng thái (Xanh - Bình thường, Vàng - Cảnh báo, Đỏ - Nguy cấp).
3.  **Vùng Chi tiết (Drill-down Area):** Các bảng và biểu đồ tương tác. Click vào một con số ở Vùng Tiêu điểm sẽ filter dữ liệu ở Vùng Chi tiết này.

### Chế độ tương tác

- **Drill-down:** Nhấp vào số lượng đơn hủy sẽ mở danh sách chi tiết các đơn đó để xem lý do.
- **Quick Action:** Cho phép Manager thực hiện thao tác nhanh (như đổi trạng thái bàn, tạm ngưng món) ngay khi phát hiện vấn đề trên Dashboard.
- **SLA Thresholds:** Manager có thể cấu hình các ngưỡng cảnh báo (Ví dụ: Thời gian chờ món trung bình > 15 phút sẽ báo động).

---

## 3. Kỹ thuật triển khai (Technical Checklist)

| Tính năng           | Backend Requirement              | Frontend Requirement              |
| :------------------ | :------------------------------- | :-------------------------------- |
| **Real-time Alert** | SignalR / WebSockets             | Toast Notification / Badge update |
| **Forecasting**     | Aggregated Query / AI Service    | Recharts / Chart.js Area chart    |
| **Inventory Sync**  | Background Job (Hangfire/Quartz) | Sync status indicator             |
| **Menu Analysis**   | Profit calculation procedures    | Treemap / Quadrant chart          |
| **Device Status**   | Heartbeat monitoring             | Monitoring status grid            |

---

_Tài liệu này được biên soạn bởi Antigravity AI để hỗ trợ quản trị vận hành FoodHub._
