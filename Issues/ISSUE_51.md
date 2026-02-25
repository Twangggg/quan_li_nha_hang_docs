# **USER STORY — KDS (Kitchen Display System)**

## **Sprint II**

# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **1.1 Vấn đề (Problem)**

Khi FOH (Ordering System) gửi món xuống bếp, hệ thống cần một cơ chế hiển thị và xử lý tập trung để:

- Nhận **OrderItem** khi Order được submit

- Hiển thị theo đúng station (Kitchen / Bar)

- Sắp xếp queue theo priority

- Giới hạn số lượng **OrderItem đang COOKING**

- Đồng bộ trạng thái realtime về FOH

- Ghi audit log cho các thao tác BOH

## **1.2 Pain Points**

- Bếp nhận thông tin chậm hoặc thiếu nhất quán

- Không có cơ chế ưu tiên rõ ràng (VIP / đặt trước / món lâu)

- Dễ quá tải khi quá nhiều món cùng lúc

- Thiếu log khi từ chối món do sự cố

## **1.3 Business Value**

- Vận hành mượt nhờ queue ưu tiên + WIP limit

- Giảm sót món nhờ station routing rõ ràng

- Minh bạch khi reject (bắt buộc reason)

- FOH theo dõi trạng thái món realtime

# **2. Đối tượng (Actors)**

## **Primary Actors**

- **Chef (Kitchen)**: xử lý món thuộc Kitchen

- **Bartender (Bar)**: xử lý món thuộc Bar

- **Chef-Bar**: có quyền REJECTED

- **Manager**: can thiệp REJECTED → PREPARING, xem log

## **Secondary Actor**

- **Ordering System (FOH)**: gửi OrderItem và nhận trạng thái cập nhật

# **3. User Story Statement**

Là Chef/Bartender/Manager, tôi muốn KDS nhận OrderItem theo station, sắp xếp hàng đợi ưu tiên, giới hạn số lượng OrderItem đang COOKING, và cập nhật trạng thái (COOKING / READY / REJECTED) realtime để tối ưu vận hành và đồng bộ với FOH.

# **4. Phạm vi trạng thái (State Ownership)**

## **4.1 Order-level (FOH Owned)**

KDS **không điều khiển Order-level**.

Order-level chỉ có:

SERVING → COMPLETED

↘

CANCELLED

KDS chỉ:

- Đọc Order-level để hiển thị context

- Biết Order còn SERVING hay đã CANCELLED

## **4.2 OrderItem-level (BOH Owned)**

KDS điều khiển:

PREPARING → COOKING → READY

↘

REJECTED

### **Ý nghĩa:**

- PREPARING: đã gửi bếp, đang chờ xử lý

- COOKING: đang chế biến

- READY: chế biến xong

- REJECTED: từ chối do sự cố (bắt buộc lý do)

# **5. Luồng Người dùng (User Flow)**

## **5.1 Nhận ticket từ FOH**

### **Trigger**

FOH submit Order (Order.Status = SERVING)

### **Backend tạo:**

OrderItem với Status = PREPARING

### **KDS hiển thị:**

- Chỉ OrderItem PREPARING

- Theo station:
  - Kitchen: món thuộc Kitchen

  - Bar: món thuộc Bar

## **5.2 Station Routing**

KDS có 2 màn hình:

### **Kitchen Screen**

Hiển thị:

- OrderItem thuộc Kitchen

- Status = PREPARING hoặc COOKING

### **Bar Screen**

Hiển thị:

- OrderItem thuộc Bar

- Status = PREPARING hoặc COOKING

## **5.3 Priority Queue (Item-level)**

Queue chỉ áp dụng cho OrderItem PREPARING.

Priority Score dựa trên:

- Thời gian chờ

- Độ phức tạp món

- Qty

- VIP

- Đặt trước

Nếu score bằng nhau → FIFO theo createdAt.

## **5.4 FOH Update Rule (Corrected)**

FOH chỉ được update:

- Order.Status = SERVING

- OrderItem.Status = PREPARING

Nếu OrderItem đã COOKING:\
→ Backend từ chối update.

## **5.5 WIP Limit (Item-level, theo station)**

Ví dụ:

- Kitchen: tối đa 4 OrderItem COOKING

- Bar: tối đa 4 OrderItem COOKING

Không áp dụng cho Order-level.

## **5.6 Auto-pull PREPARING → COOKING**

Khi:

- COOKING &lt; WIP limit

- Có OrderItem PREPARING

Hệ thống tự động:

OrderItem: PREPARING → COOKING

Đồng bộ realtime về FOH.

## **5.7 COOKING → READY**

Chef/Bartender bấm Done:

OrderItem: COOKING → READY

FOH thấy món READY.

Order-level vẫn = SERVING.

## **5.8 REJECTED**

Điều kiện:

- OrderItem.Status = COOKING

- Actor = Manager hoặc Chef-Bar

- Reason bắt buộc

Transition:

OrderItem: COOKING → REJECTED

Giải phóng slot WIP.

## **5.9 Return REJECTED → PREPARING**

Chỉ Manager được làm:

OrderItem: REJECTED → PREPARING

Item quay lại queue và được tính lại priority.

## **5.10 Audit Log**

Log các action:

- Nhận ticket

- PREPARING → COOKING

- COOKING → READY

- COOKING → REJECTED

- REJECTED → PREPARING

# **6. Business Rules**

BR-KDS-01: KDS chỉ xử lý OrderItem, không thay đổi Order-level\
BR-KDS-02: Station routing theo item.station\
BR-KDS-03: Queue áp dụng cho PREPARING item\
BR-KDS-04: WIP limit áp dụng cho OrderItem COOKING theo station\
BR-KDS-05: COOKING item không được sửa\
BR-KDS-06: REJECTED bắt buộc reason\
BR-KDS-07: KDS không tự động COMPLETE Order\
BR-KDS-08: Tất cả thay đổi phải có audit log

# **7. Acceptance Criteria**

### **AC-01: Nhận ticket đúng station**

Given Order SERVING và có OrderItem PREPARING\
When mở KDS\
Then item hiển thị đúng station

### **AC-02: Priority + FIFO**

Given nhiều OrderItem PREPARING\
When sắp xếp\
Then theo priority; nếu bằng nhau → FIFO

### **AC-03: WIP limit**

Given Kitchen có 4 item COOKING\
When có PREPARING\
Then không auto-pull thêm

### **AC-04: Auto-pull**

Given slot trống\
When có item PREPARING\
Then item đầu queue → COOKING

### **AC-05: READY**

Given item COOKING\
When bấm Done\
Then item → READY\
And Order vẫn SERVING

### **AC-06: Reject**

Given item COOKING\
When reject\
Then item → REJECTED\
And reason được lưu

### **AC-07: Return**

Given item REJECTED\
When Manager return\
Then item → PREPARING\
And recalc queue

### **AC-08: Audit log**

Given có thao tác trạng thái\
When Manager xem log\
Then hiển thị đúng actor / timestamp / action / reason
