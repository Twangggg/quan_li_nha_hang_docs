# **1. Tổng quan & Bối cảnh (Overview & Context)**

## **Vấn đề (Problem):**

Nhà hàng cần đảm bảo mỗi ngày luôn có đủ nhân viên làm việc trong các ca:

- Ca sáng: 10:30 – 14:00
- Ca chiều: 17:00 – 23:00

Hiện tại chưa có hệ thống phân công lịch tập trung, dẫn đến:

- Dễ thiếu hoặc dư nhân sự trong ca
- Không kiểm soát được nhân viên làm ca nào
- Khó theo dõi lịch làm tổng thể

---

## **Pain Points hiện tại:**

- Phân ca thủ công → dễ nhầm lẫn
- Có thể gán 1 nhân viên nhiều ca trong 1 ngày
- Không có nơi xem lịch tổng thể

---

## **Giá trị Nghiệp vụ (Business Value):**

- Đảm bảo đủ nhân sự mỗi ca
- Dễ quản lý lịch làm theo ngày/tuần
- Giảm sai sót khi phân ca

---

## **Đối tượng (Actor):**

### **Primary Actor: Quản lý nhà hàng (Manager)**

---

## **User Story Statement**

Là một **Quản lý**, tôi muốn phân ca cho nhân viên theo từng ngày để đảm bảo nhà hàng luôn đủ nhân sự hoạt động.

---

# **2. Luồng Người dùng (User Flow)**

## **2.1. Xem và phân ca**

1. Manager vào trang **"Phân ca"** (/schedules)
2. Hệ thống hiển thị:
   - Danh sách nhân viên
   - Các ngày trong tuần
3. Manager chọn:
   - Nhân viên
   - Ngày
   - Ca (Sáng / Chiều)
4. Click **"Lưu"**

---

## **2.2. Validation**

- ❌ Không cho 1 nhân viên có &gt;1 ca trong 1 ngày
- ❌ Không cho gán ca đã bị Inactive

---

# **3. Acceptance Criteria**

- Gán ca thành công khi hợp lệ
- Không cho gán 2 ca cùng ngày
- Hiển thị lịch sau khi lưu
