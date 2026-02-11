# Policy-based Authorization (Phân quyền dựa trên Policy)

Hệ thống này thay thế cho phân quyền dựa trên Role truyền thống, cung cấp khả năng kiểm soát truy cập linh hoạt đến từng hành động (Action) cụ thể.

## 1. Thành phần chính

### A. Permission Constants (`Permissions.cs`)

Định nghĩa tập hợp các quyền hạn dưới dạng hằng số chuỗi.

- **Vị trí**: `FoodHub.Application/Constants/Permissions.cs`
- **Ví dụ**: `Permissions.Orders.Create`, `Permissions.Employees.Delete`.

### B. IPermissionProvider & PermissionProvider

Ánh xạ các `EmployeeRole` (Manager, Waiter, Cashier...) sang tập hợp các `Permission` tương ứng.

- **Vị trí**: `FoodHub.Infrastructure/Security/PermissionProvider.cs`

### C. PermissionPolicyProvider

Bộ xử lý động tự động tạo ra các `AuthorizationPolicy` dựa trên tên Permission. Bạn không cần khai báo Policy thủ công trong `Program.cs`.

### D. PermissionHandler

Lớp gác cổng kiểm tra xem `Claims` của người dùng hiện tại (lấy từ JWT) có chứa Permission được yêu cầu hay không.

## 2. Cách sử dụng

### Tại Controller

Sử dụng attribute `[HasPermission]` đi kèm với hằng số từ `Permissions` class.

```csharp
[HttpPost]
[HasPermission(Permissions.Orders.Create)]
public async Task<IActionResult> CreateOrder(...) { ... }
```

### Tại Frontend (Token)

Khi đăng nhập thành công, các Permission sẽ được nhúng vào Token dưới dạng Claim:

- **Type**: `Permission`
- **Value**: `Permissions.Orders.Create`

---

## 3. Lợi ích

1. **Linh hoạt**: Thay đổi quyền của một Role chỉ tại 1 nơi (`PermissionProvider`), không cần sửa Controller hay Database.
2. **Bảo mật**: Kiểm soát quyền ở mức độ hạt nhân (Granular).
3. **Dễ đọc**: `[HasPermission]` tường minh hơn nhiều so với việc liệt kê danh sách Role.
