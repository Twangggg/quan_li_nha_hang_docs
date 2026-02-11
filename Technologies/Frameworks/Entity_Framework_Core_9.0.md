# Hướng Dẫn Sử Dụng Entity Framework Core 9.0

## Mô tả

ORM (Object-Relational Mapper) cho phép làm việc với database bằng cách sử dụng các đối tượng .NET thay vì viết SQL trực tiếp.

## Cách sử dụng trong dự án

- Sử dụng cho việc truy vấn và thao tác dữ liệu PostgreSQL.
- Cấu hình trong `Infrastructure/Persistence/AppDbContext.cs`.

## Ví dụ sử dụng

```csharp
// Truy vấn dữ liệu
var employees = await _unitOfWork.Repository<Employee>()
    .Query()
    .Where(e => e.Status == EmployeeStatus.Active)
    .ToListAsync();

// Thêm mới
var newEmployee = new Employee { FullName = "John Doe", Email = "john@example.com" };
await _unitOfWork.Repository<Employee>().AddAsync(newEmployee);
await _unitOfWork.SaveChangesAsync();

// Cập nhật
employee.FullName = "Jane Doe";
_unitOfWork.Repository<Employee>().Update(employee);
await _unitOfWork.SaveChangesAsync();
```

## Migration

```bash
# Tạo migration
dotnet ef migrations add InitialCreate --project FoodHub.Infrastructure --startup-project FoodHub.WebAPI

# Áp dụng migration
dotnet ef database update --project FoodHub.Infrastructure --startup-project FoodHub.WebAPI
```
