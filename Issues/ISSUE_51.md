# **USER STORY — KDS (Kitchen Display System) (Sprint II)**

## **1. Tổng quan & Bối cảnh (Overview & Context)**

### **Vấn đề (Problem)**

Nhà hàng cần KDS để:

- Nhận ticket/order từ Ordering System sau khi submit.

- Hiển thị danh sách **order** theo 2 station chính:
  - **Kitchen (Hot + Cold chung một màn)**

  - **Bar (màn riêng)**

- Xử lý theo cơ chế hàng đợi ưu tiên (priority queue) và giới hạn số order đang **COOKING** để tránh quá tải.

- Cập nhật trạng thái **order** realtime về FOH.

### **Pain Points hiện tại**

- Bếp nhận thông tin chậm/không nhất quán.

- Không có thứ tự ưu tiên rõ ràng (VIP/đặt trước/món lâu).

- Dễ quá tải nếu nhận quá nhiều order đang nấu.

- Thiếu log khi reject do thiếu nguyên liệu/sự cố.

### **Giá trị Nghiệp vụ (Business Value)**

- Bếp vận hành mượt: queue ưu tiên + WIP limit.

- Giảm sót món: Kitchen/Bar nhìn đúng phần món thuộc station của mình.

- Minh bạch xử lý sự cố: reject bắt buộc lý do + có log.

- Đồng bộ tốt: FOH thấy trạng thái order realtime.

## **2. Đối tượng (Actor)**

### **Primary Actors**

- **Chef (Kitchen):** xem queue Kitchen, xử lý và cập nhật trạng thái.

- **Bartender (Bar):** xem queue Bar, xử lý và cập nhật trạng thái.

- **Chef-Bar:** có quyền **REJECTED** khi đang COOKING (bắt buộc lý do).

- **Manager:** can thiệp REJECTED → PREPARING, xem log vận hành.

### **Secondary Actor**

- **Ordering System (FOH):** gửi ticket xuống KDS và nhận trạng thái trả về để hiển thị FOH.

## **3. User Story Statement**

Là Chef/Bar/Manager, tôi muốn KDS nhận ticket theo station (Kitchen/Bar), sắp hàng đợi ưu tiên, giới hạn số order đang COOKING, và cập nhật trạng thái order (COOKING/READY/REJECTED) realtime để tối ưu vận hành bếp/bar và đồng bộ phục vụ.

## **4. Trạng thái Order (Order Status) — góc nhìn BOH (Order-level)**

### **Flow chính**

- **PREPARING → COOKING → READY**

### **Ngoại lệ**

- **REJECTED:** từ chối khi COOKING do thiếu nguyên liệu/sự cố (bắt buộc lý do).

- **Return:** Manager chuyển **REJECTED → PREPARING** để quay lại hàng đợi.

Note: DRAFT/CANCELLED/COMPLETED thuộc vòng đời tổng thể, nhưng KDS chủ yếu xử lý từ **PREPARING** trở đi.

## **5. Luồng Người dùng (User Flow)**

### **5.1. Nhận ticket từ FOH (PREPARING)**

- Khi FOH submit, order xuất hiện trên KDS theo station:
  - **Kitchen Screen:** hiển thị order có ít nhất 1 món thuộc Kitchen

  - **Bar Screen:** hiển thị order có ít nhất 1 món thuộc Bar

- Mặc định hiển thị theo thứ tự ưu tiên (priority); nếu score bằng nhau → FIFO theo sentAt.

### **5.2. Station routing (Kitchen/Bar)**

- KDS có 2 màn hình/tabs:
  - **Kitchen (Hot + Cold chung):** hiển thị **chỉ các món kitchen** của order.

  - **Bar:** hiển thị **chỉ các món bar** của order.

- **Order-level status** là chung, nhưng mỗi station nhìn “phần món thuộc station”.

### **5.3. Hàng đợi ưu tiên + realtime cập nhật**

- KDS tính **Priority Score** để sắp thứ tự order trong PREPARING dựa trên:
  - Thời gian chờ

  - Độ phức tạp (món lâu &gt; món nhanh; số lượng món/qty)

  - VIP

  - Khách đặt trước

- Khi FOH sửa order trong PREPARING (thêm món/đổi qty/note/huỷ món):
  - KDS update realtime nội dung order

  - Queue **recalc/reorder** nếu score thay đổi

Rule quan trọng: FOH **chỉ được update nội dung** khi order còn **PREPARING**. Nếu order đã COOKING → KDS từ chối/ignore cập nhật.

### **5.4. Giới hạn COOKING (WIP limit)**

- Giới hạn tối đa **4 order COOKING cùng lúc (tổng hệ thống)**.

- Khi đủ 4 COOKING → các order khác tiếp tục nằm ở PREPARING.

### **5.5. Auto-pull PREPARING → COOKING**

- Khi có slot trống (COOKING &lt; 4):
  - Hệ thống tự động chọn order **đầu hàng đợi** (sau khi sort theo Priority Score, tie-break FIFO)

  - Chuyển order từ **PREPARING → COOKING**

  - Đồng bộ trạng thái COOKING về FOH realtime

### **5.6. COOKING → READY (hoàn tất chế biến)**

- Kitchen/Bar cập nhật “Done” phần station của mình.

- **READY (order-level)** xảy ra khi:
  - Nếu order chỉ có món Kitchen → Kitchen done thì READY

  - Nếu order có cả Kitchen và Bar → chỉ READY khi **cả Kitchen và Bar đều done**

- Trạng thái READY được đồng bộ realtime về FOH.

### **5.7. REJECTED khi đang COOKING & quay lại PREPARING**

- Khi COOKING gặp sự cố/thiếu nguyên liệu:
  - Chỉ **Manager hoặc Chef-Bar** được chuyển order sang **REJECTED**

  - Bắt buộc nhập lý do (required)

  - Khi REJECTED → **giải phóng slot COOKING**

- Xử lý sau REJECTED:
  - Trao đổi khách

  - Nếu khách OK tiếp tục:
    - **Manager chuyển REJECTED → PREPARING**

    - Order quay lại queue và được recalc priority score

### **5.8. Audit log BOH**

- KDS lưu log: nhận ticket, chuyển trạng thái, auto-pull, ready-by-station, reject (reason), return-to-preparing…

## **6. Business Rules**

- **BR-KDS-01:** KDS nhận ticket khi FOH submit → order vào PREPARING.

- **BR-KDS-02:** Có 2 màn: **Kitchen (Hot+Cold chung)** và **Bar**.

- **BR-KDS-03:** Mỗi màn chỉ hiển thị **món thuộc station** của mình, nhưng status là **order-level**.

- **BR-KDS-04:** Priority score = thời gian chờ + độ phức tạp + VIP + đặt trước.

- **BR-KDS-04a:** Nếu score bằng nhau → FIFO theo sentAt.

- **BR-KDS-05:** Queue recalc/reorder realtime khi order PREPARING thay đổi.

- **BR-KDS-05a:** Nếu order đã COOKING → không nhận update nội dung từ FOH.

- **BR-KDS-06:** WIP limit: tối đa **4 order COOKING** đồng thời (tổng hệ thống).

- **BR-KDS-07:** Có slot trống → auto-pull order đầu queue vào COOKING.

- **BR-KDS-08:** REJECTED chỉ do Manager hoặc Chef-Bar, bắt buộc lý do.

- **BR-KDS-08a:** REJECTED giải phóng slot COOKING.

- **BR-KDS-09:** Manager có quyền REJECTED → PREPARING.

- **BR-KDS-10 (Audit):** Log status changes + actor + timestamp + reason (nếu có).

## **7. Tiêu chí Chấp nhận (Acceptance Criteria)**

**AC-KDS-01: Nhận ticket và hiển thị đúng station**

- Given: FOH submit order hợp lệ (PREPARING)

- When: mở KDS Kitchen/Bar

- Then: order xuất hiện realtime ở đúng màn
  - Kitchen thấy phần món kitchen

  - Bar thấy phần món bar

**AC-KDS-02: Priority + tie-break FIFO**

- Given: nhiều order PREPARING

- When: KDS sắp xếp hàng đợi

- Then: sắp theo priority score; nếu bằng nhau thì theo sentAt (FIFO)

**AC-KDS-03: FOH sửa PREPARING → KDS update & reorder**

- Given: order PREPARING đang hiển thị

- When: FOH cập nhật món/qty/note thành công

- Then: KDS cập nhật realtime; nếu score đổi thì reorder, không tạo order mới

**AC-KDS-04: Không nhận update khi COOKING**

- Given: order đã COOKING

- When: FOH gửi update nội dung

- Then: KDS từ chối/ignore; dữ liệu trên KDS không thay đổi

**AC-KDS-05: WIP limit 4 COOKING**

- Given: đang có 4 order COOKING

- When: có order đến lượt

- Then: order vẫn ở PREPARING, không chuyển COOKING

**AC-KDS-06: Auto-pull khi có slot**

- Given: COOKING &lt; 4 và có queue PREPARING

- When: slot trống xuất hiện

- Then: auto-pull order đầu queue → COOKING, đồng bộ về FOH

**AC-KDS-07: READY order-level theo điều kiện station**

- Given: order COOKING và có món thuộc Kitchen/Bar

- When: Kitchen done (và Bar done nếu có)

- Then: order chuyển READY, đồng bộ FOH

**AC-KDS-08: REJECTED bắt buộc reason và đúng quyền**

- Given: order COOKING

- When: chuyển REJECTED

- Then: chỉ Manager/Chef-Bar được làm, phải có reason; REJECTED giải phóng slot; đồng bộ FOH

**AC-KDS-09: Manager return REJECTED → PREPARING**

- Given: order REJECTED

- When: Manager return

- Then: order quay về PREPARING, vào queue, recalc score, đồng bộ FOH

**AC-KDS-10: Audit log**

- Given: có thao tác status/auto-pull/reject/return

- When: Manager xem log

- Then: thấy actor/timestamp/action/reason (nếu có) đúng lịch sử
