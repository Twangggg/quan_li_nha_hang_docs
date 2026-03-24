# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem):**

Nhà hàng hoạt động theo khung giờ cố định từ **10:30 đến 23:00**, với thời gian nghỉ giữa ca từ **14:00 đến 17:00**.

Hệ thống cần quản lý các ca làm việc để:

- Chuẩn hóa thời gian làm việc của nhân viên theo giờ hoạt động thực tế
- Phục vụ cho việc phân công lịch làm hàng ngày
- Đảm bảo nhân viên chỉ được gán vào các ca hợp lệ trong khung giờ hoạt động

---

## **Pain Points hiện tại:**

- Không có định nghĩa ca làm rõ ràng → dễ phân sai giờ
- Có thể tạo ca vượt ngoài giờ hoạt động của nhà hàng
- Có thể tạo ca nằm trong thời gian nghỉ (14:00 – 17:00)
- Dữ liệu ca làm không nhất quán → ảnh hưởng đến phân ca và chấm công

---

## **Giá trị Nghiệp vụ (Business Value):**

- **Chuẩn hóa vận hành:** Ca làm bám sát giờ hoạt động thực tế
- **Giảm sai sót:** Không cho tạo ca sai giờ hoặc chồng chéo
- **Đơn giản hóa hệ thống:** Chỉ sử dụng 2 ca cố định mỗi ngày
- **Hỗ trợ phân ca chính xác:** Là nền tảng cho toàn bộ module lịch làm và chấm công

---

## **Đối tượng (Actor):**

### **Primary Actor: Quản lý nhà hàng (Manager)**

- Tạo và quản lý các ca làm việc
- Đảm bảo ca làm đúng với giờ hoạt động của nhà hàng

---

## **User Story Statement**

Là một **Quản lý nhà hàng (Manager)**, tôi muốn tạo và quản lý các ca làm việc phù hợp với giờ hoạt động của nhà hàng (10:30–14:00 và 17:00–23:00) để đảm bảo việc phân công nhân viên luôn chính xác và nhất quán.

---

# **2. Luồng Người dùng (User Flow)**

---

## **2.1. Luồng chính: Xem danh sách ca làm**

1. Manager đăng nhập hệ thống
2. Truy cập trang **"Quản lý ca làm"** (/shifts)
3. Hệ thống hiển thị danh sách ca:
   - Ca sáng: 10:30 – 14:00
   - Ca chiều: 17:00 – 23:00
4. Hiển thị:
   - Tên ca
   - Giờ bắt đầu
   - Giờ kết thúc
   - Trạng thái

---

## **2.2. Luồng: Tạo ca làm**

1. Manager click **"Tạo ca"**
2. Form gồm:
   - Tên ca
   - Giờ bắt đầu
   - Giờ kết thúc
3. Click **"Lưu"**

---

## **2.3. Validation (QUAN TRỌNG – theo giờ hoạt động)**

Hệ thống validate:

- Giờ bắt đầu &lt; giờ kết thúc
- ❌ Không được tạo ca ngoài khung giờ:
  - Trước 10:30
  - Sau 23:00
- ❌ Không được tạo ca nằm trong giờ nghỉ:
  - 14:00 – 17:00
- ❌ Không được tạo ca **overlap** với ca khác

---

## **2.4. Luồng: Chỉnh sửa ca**

- Tương tự tạo
- Áp dụng toàn bộ validation trên

---

## **2.5. Luồng: Vô hiệu hóa ca**

- Manager click **"Vô hiệu hóa"**
- Hệ thống:
  - Đổi trạng thái → Inactive
  - Không xóa dữ liệu

---

# **3. Tiêu chí Chấp nhận (Acceptance Criteria)**

---

### **AC-01: Hiển thị ca hợp lệ**

**Given:** Manager đăng nhập\
**When:** Truy cập /shifts\
**Then:**

- Hiển thị các ca:
  - 10:30–14:00
  - 17:00–23:00

---

### **AC-02: Tạo ca thành công**

**Given:** Manager nhập:

- 10:30 – 14:00 hoặc 17:00 – 23:00\
  **When:** Click "Lưu"\
  **Then:**
- Tạo thành công
- Hiển thị thông báo

---

### **AC-03: Tạo ca thất bại – ngoài giờ hoạt động**

**Given:**\
Nhập ca: 09:00 – 12:00

**When:** Click "Lưu"\
**Then:**

- Hiển thị lỗi:\
  _"Ca làm phải nằm trong khung giờ hoạt động (10:30 – 23:00)"_

---

### **AC-04: Tạo ca thất bại – trong giờ nghỉ**

**Given:**\
Nhập ca: 14:30 – 16:00

**When:** Click "Lưu"\
**Then:**

- Hiển thị lỗi:\
  _"Không được tạo ca trong thời gian nghỉ (14:00 – 17:00)"_

---

### **AC-05: Tạo ca thất bại – trùng giờ**

**Given:**\
Đã có ca 10:30 – 14:00

**When:**\
Tạo ca 11:00 – 13:00

**Then:**

- Hiển thị lỗi:\
  _"Ca làm bị trùng giờ với ca đã tồn tại"_

---

### **AC-06: Vô hiệu hóa ca**

**Given:** Ca đang Active\
**When:** Click "Vô hiệu hóa"\
**Then:**

- Ca chuyển sang Inactive
- Không bị xóa
