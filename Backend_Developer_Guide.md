# Hướng Dẫn Phát Triển Backend (Backend Developer Guide)

Tài liệu này hướng dẫn cách sử dụng, cấu trúc code và cách phát triển các tính năng mới cho hệ thống Backend của FoodHub.

## 1. Kiến Trúc Hệ Thống (Architecture)

Hệ thống được xây dựng theo kiến trúc **Clean Architecture** (hoặc Onion Architecture) kết hợp với các pattern hiện đại như **CQRS** (Command Query Responsibility Segregation).

### Các Lớp Trong Hệ Thống:

- **Domain**: Chứa các thực thể (Entities), Enums, và các quy tắc nghiệp vụ cốt lõi. Không phụ thuộc vào bất kỳ thư viện ngoài nào ngoại trừ các thư viện hệ thống.
- **Application**: Chứa logic nghiệp vụ (Services, MediatR Handlers), DTOs, Mappings, và Interfaces cho các service bên ngoài. Đây là lớp điều phối chính.
- **Infrastructure**: Chứa các triển khai chi tiết cho việc lưu trữ (Persistence - EF Core), Security (JWT), Email, v.v.
- **Presentation (API)**: Chứa các Controllers, Middlewares để giao tiếp với bên ngoài.

---

## 2. Công Nghệ Sử Dụng (Tech Stack)

- **Language**: C# 13 / .NET 9
- **Database**: PostgreSQL (Entity Framework Core 9)
- **Mapping**: AutoMapper
- **Messaging**: MediatR (CQRS Pattern)
- **Validation**: FluentValidation
- **Logging**: Serilog (Nếu có)
- **Documentation**: OpenAPI (Swagger)

---

## 3. Cấu Trúc Folder & Quy Tắc Đặt Tên

### Folder Structure

- `Domain/Entities/`: Tên file PascalCase, số ít (ví dụ: `Employee.cs`).
- `Application/Features/[FeatureName]/Commands/`: Chứa các yêu cầu thay đổi dữ liệu.
- `Application/Features/[FeatureName]/Queries/`: Chứa các yêu cầu đọc dữ liệu.
- `Infrastructure/Persistence/Configurations/`: Cấu hình Fluent API cho EF Core.

### Coding Rules

- Sử dụng **File-scoped namespaces** để giảm indentation.
- Tuân thủ **Coding_Convention_v1.0.md** đã đề ra.
- Luôn sử dụng `async/await` cho các thao tác IO (DB, Network).

---

## 4. Cách Thêm Tính Năng Mới (Step-by-Step)

Giả sử bạn muốn thêm tính năng "Lấy danh sách nhân viên":

### Bước 1: Tạo DTO

Tạo file `EmployeeDto.cs` trong `Application/DTOs/Employees/` (nếu chưa có).
Sử dụng `IMapFrom<Employee>` để tự động mapping.

### Bước 2: Tạo Query & Handler

Tạo file `GetEmployeesQuery.cs` trong `Application/Features/Employees/Queries/`:

```csharp
public record GetEmployeesQuery : IRequest<List<EmployeeDto>>;

public class GetEmployeesQueryHandler : IRequestHandler<GetEmployeesQuery, List<EmployeeDto>>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public GetEmployeesQueryHandler(IUnitOfWork unitOfWork, IMapper mapper)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    public async Task<List<EmployeeDto>> Handle(GetEmployeesQuery request, CancellationToken cancellationToken)
    {
        var employees = await _unitOfWork.Repository<Employee>()
            .Query()
            .ToListAsync(cancellationToken);

        return _mapper.Map<List<EmployeeDto>>(employees);
    }
}
```

### Bước 3: Tạo Controller

Nên tạo một `ApiControllerBase` để dùng chung `ISender` (Mediator):

```csharp
[ApiController]
[Route("api/[controller]")]
public abstract class ApiControllerBase : ControllerBase
{
    private ISender _mediator = null!;
    protected ISender Mediator => _mediator ??= HttpContext.RequestServices.GetRequiredService<ISender>();
}
```

Sau đó tạo `EmployeesController.cs`:

```csharp
public class EmployeesController : ApiControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetEmployees()
    {
        var result = await Mediator.Send(new GetEmployeesQuery());
        return Ok(result);
    }
}
```

---

## 5. Các Patterns Quan Trọng

### Result Pattern (Khuyến nghị)

Thay vì throw exception cho các lỗi logic, hãy dùng `Result<T>`:

- `Result.Success(data)`
- `Result.Failure(errors)`

### Validation Behavior

Hệ thống đã cấu hình `ValidationBehavior` trong MediatR Pipeline. Khi bạn gửi một Request (Command/Query):

1. Pipeline sẽ tìm tất cả các class kế thừa `AbstractValidator<TRequest>`.
2. Nếu có lỗi validation, nó sẽ throw `ValidationException`.
3. `ExceptionMiddleware` sẽ bắt được và trả về lỗi 400 kèm chi tiết các field bị lỗi.

**Cách dùng:** Chỉ cần tạo file validator cùng thư mục với Command.

```csharp
public class CreateEmployeeCommandValidator : AbstractValidator<CreateEmployeeCommand>
{
    public CreateEmployeeCommandValidator()
    {
        RuleFor(v => v.Email).NotEmpty().EmailAddress();
        RuleFor(v => v.Username).MinimumLength(5);
    }
}
```

### AutoMapper (IMapFrom Pattern)

Để tránh việc phải cấu hình Mapping thủ công cho từng DTO, hệ thống sử dụng interface `IMapFrom<T>`.

**Cách dùng:**

- Cho mapping đơn giản (tên field giống nhau):

```csharp
public class EmployeeDto : IMapFrom<Employee> { }
```

- Cho mapping phức tạp (cần custom logic):

```csharp
public class EmployeeDto : IMapFrom<Employee>
{
    public string FullName { get; set; }
    public string RoleName { get; set; }

    public void Mapping(Profile profile)
    {
        profile.CreateMap<Employee, EmployeeDto>()
            .ForMember(d => d.RoleName, opt => opt.MapFrom(s => s.Role.ToString()));
    }
}
```

### Unit of Work & Generic Repository

Hệ thống sử dụng Unit of Work để quản lý transaction và đảm bảo tính nhất quán của dữ liệu.

**Cách dùng trong Handler:**

```csharp
// Thêm mới
await _unitOfWork.Repository<Employee>().AddAsync(employee);
// Lưu thay đổi (Đây là lúc DbContext.SaveChangesAsync được gọi)
await _unitOfWork.SaveChangeAsync(cancellationToken);
```

### Xử lý lỗi (Exception Handling)

Sử dụng các Exception tùy chỉnh để `ExceptionMiddleware` có thể trả về đúng mã lỗi HTTP:

- `NotFoundException("Message")` -> Trả về 404.
- `BusinessException("Message")` -> Trả về 400.
- Các lỗi khác -> Trả về 500.

---

## 6. EF Core Migrations

Vì dự án chia theo nhiều lớp (Domain, Infrastructure, API), việc chạy lệnh Migration cần chỉ định rõ project.

### Các lệnh cơ bản (Chạy tại thư mục gốc FoodHub_BE):

1. **Thêm Migration mới:**

   ```bash
   dotnet ef migrations add [MigrationName] --project Infrastructure --startup-project .
   ```

   _(Infrastructure là nơi chứa DbContext, . là thư mục hiện tại chứa file Program.cs)_

2. **Cập nhật Database:**

   ```bash
   dotnet ef database update --project Infrastructure --startup-project .
   ```

3. **Xóa Migration cuối cùng (chưa update DB):**

   ```bash
   dotnet ef migrations remove --project Infrastructure --startup-project .
   ```

4. **Tạo file Script SQL (để deploy tay):**
   ```bash
   dotnet ef migrations script --project Infrastructure --startup-project .
   ```

### Lưu ý quan trọng:

- **Snake Case:** Database đã được cấu hình tự động chuyển từ `PascalCase` sang `snake_case`. Không cần đặt tên bảng/cột thủ công trong code.
- **Data Seeding:** Có thể thực hiện seeding trong file `AppDbContext.cs` hoặc thông qua các class `Configuration`.
- **Migration History:** Bảng `__EFMigrationsHistory` sẽ lưu lại các migration đã chạy, đừng xóa bảng này.

---

## 7. Hướng Dẫn Chạy Project

1. Cài đặt PostgreSQL và tạo database.
2. Cập nhật `ConnectionStrings:DefaultConnection` trong `appsettings.json`.
3. Chạy lệnh migration:
   ```bash
   dotnet ef database update
   ```
4. Run project:
   ```bash
   dotnet run --project FoodHub.csproj
   ```

---

_Tài liệu này sẽ được cập nhật liên tục khi hệ thống phát triển._
