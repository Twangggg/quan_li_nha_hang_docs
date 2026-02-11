# Cẩm Nang Tích Hợp Frontend (Frontend Integration Guide)

Tài liệu này cung cấp toàn bộ thông tin kỹ thuật cần thiết để đội ngũ Frontend tích hợp hiệu quả với Backend FoodHub.

---

## 1. Thông Tin Chung (General Info)

- **Base URL (Local):** `http://localhost:5000/api`
- **Port Docker:** Mặc định là `5000`.
- **Headers:** Luôn đính kèm các Header sau:
  - `Content-Type: application/json`
  - `Accept: application/json`

---

## 2. Danh Mục Hằng Số (Enums)

Frontend nên định nghĩa các Enum này để khớp với logic Backend:

### EmployeeRole (Vai trò nhân viên)

| Giá trị | Label     | Mô tả             |
| :------ | :-------- | :---------------- |
| `1`     | `Manager` | Quản lý           |
| `2`     | `Cashier` | Thu ngân          |
| `3`     | `Waiter`  | Phục vụ           |
| `4`     | `ChefBar` | Đầu bếp / Barista |

### EmployeeStatus (Trạng thái)

| Giá trị | Label      | Mô tả           |
| :------ | :--------- | :-------------- |
| `0`     | `Inactive` | Ngừng hoạt động |
| `1`     | `Active`   | Đang hoạt động  |

---

## 3. Xác Thực & Bảo Mật (Auth & Security)

> [!NOTE]
> Hệ thống Auth hiện đang được hoàn thiện. Dưới đây là cấu hình chuẩn bị sẵn:

- **JWT Auth:** Sử dụng Header `Authorization: Bearer <token>`.
- **Login API:** Dự kiến `POST /api/auth/login`.

---

## 4. Đặc Tả API Nhân Viên (Employee API)

### 4.1. Lấy danh sách nhân viên (Phân trang, Search, Filter, Sort)

`GET /api/employees`

**Tham số Query (Object `pagination`):**

- `pagination.pageIndex`: Trang số (mặc định 1).
- `pagination.pageSize`: Số bản ghi/trang (mặc định 10).
- `pagination.search`: Tìm kiếm theo Tên, Email, SĐT, Mã NV.
- `pagination.filters`: Lọc theo trường (Dạng `key:value`). Ví dụ: `status:1`, `role:3`.
- `pagination.orderBy`: Sắp xếp đa tầng. Dấu `-` cho giảm dần. Ví dụ: `role,-createdAt`.

**Dữ liệu trả về (`PagedResult`):**

```json
{
  "items": [],
  "pageIndex": 1,
  "pageSize": 10,
  "totalCount": 50,
  "totalPages": 5,
  "hasPreviousPage": false,
  "hasNextPage": true
}
```

### 4.2. Lấy chi tiết nhân viên

`GET /api/employees/{id}`

### 4.3. Thêm mới nhân viên

`POST /api/employees`

**Request Body:**

```json
{
  "employeeCode": "NV001",
  "fullName": "Nguyễn Văn A",
  "email": "a@example.com",
  "role": 2, // Sử dụng số từ Enum Role
  "status": 1 // Mặc định là 1 (Active)
}
```

### 4.4. Cập nhật nhân viên

`PUT /api/employees/{id}`

**Request Body:**

```json
{
  "employeeId": "uuid-here",
  "username": "vana",
  "fullName": "Nguyễn Văn A",
  "phone": "0988xxx",
  "address": "123 Đường ABC",
  "dateOfBirth": "1995-01-01" // Định dạng ISO Date
}
```

### 4.5. Xóa nhân viên

`DELETE /api/employees/{id}`

---

## 5. Quy Tắc Xử Lý Lỗi (Error Handling)

Hệ thống trả về các mã lỗi HTTP standard kèm theo thông báo chi tiết:

| Http Code | Ý nghĩa                  | Hành động Frontend                                        |
| :-------- | :----------------------- | :-------------------------------------------------------- |
| `400`     | Bad Request / Validation | Hiển thị thông báo trong `message` hoặc `errors`.         |
| `401`     | Unauthorized             | Redirect về trang Login.                                  |
| `404`     | Not Found                | Hiển thị trang 404 hoặc thông báo không tìm thấy bản ghi. |
| `500`     | Server Error             | Hiển thị thông báo "Hệ thống đang bảo trì".               |

**Cấu trúc Object Lỗi:**

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": ["Email không đúng định dạng", "Mã nhân viên đã tồn tại"]
}
```
