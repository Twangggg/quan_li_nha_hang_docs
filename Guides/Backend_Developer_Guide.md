# Hướng Dẫn Phát Triển Backend (Backend Developer Guide)

Tài liệu này hướng dẫn cách sử dụng, cấu trúc code và cách phát triển các tính năng mới cho hệ thống Backend của FoodHub.

## 1. Kiến Trúc Hệ Thống (Architecture)

Hệ thống được xây dựng theo kiến trúc **Clean Architecture** kết hợp với các pattern hiện đại như **CQRS** (Command Query Responsibility Segregation).

### Các Lớp Trong Hệ Thống:

- **Domain**: Chứa các thực thể (Entities), Enums, và các quy tắc nghiệp vụ cốt lõi. Không phụ thuộc vào bất kỳ thư viện ngoài nào ngoại trừ các thư viện hệ thống.
- **Application**: Chứa logic nghiệp vụ (Services, MediatR Handlers), DTOs, Mappings, và Interfaces cho các service bên ngoài. Đây là lớp điều phối chính.
- **Infrastructure**: Chứa các triển khai chi tiết cho việc lưu trữ (Persistence - EF Core), Security (JWT), Email, Redis Cache, v.v.
- **Presentation (API)**: Chứa các Controllers, Middlewares để giao tiếp với bên ngoài.

---

## 2. Công Nghệ Sử Dụng (Tech Stack)

- **Language**: C# 13 / .NET 9
- **Database**: PostgreSQL (Entity Framework Core 9)
- **Mapping**: AutoMapper
- **Messaging**: MediatR (CQRS Pattern)
- **Validation**: FluentValidation
- **Caching**: Redis (IDistributedCache)
- **Logging**: Serilog
- **Documentation**: Swagger/OpenAPI

---

## 3. Cấu Trúc Folder & Quy Tắc Đặt Tên

### Folder Structure

- `Domain/Entities/`: Tên file PascalCase, số ít (ví dụ: `Employee.cs`).
- `Application/Features/[FeatureName]/Commands/`: Chứa các yêu cầu thay đổi dữ liệu.
- `Application/Features/[FeatureName]/Queries/`: Chứa các yêu cầu đọc dữ liệu.
- `Infrastructure/Persistence/Configurations/`: Cấu hình Fluent API cho EF Core.

### Coding Rules

- Sử dụng **File-scoped namespaces** để giảm indentation.
- Luôn sử dụng `async/await` cho các thao tác IO (DB, Network).
- Tuân thủ quy tắc đặt tên: Class/Method/Property: PascalCase, Parameter/Variable: camelCase.

---

## 4. Cách Thêm Tính Năng Mới (Step-by-Step)

Giả sử bạn muốn thêm tính năng "Lấy danh sách sản phẩm":

### Bước 1: Tạo Response DTO

Tạo file `GetMenuItemsResponse.cs` trong `Application/Features/MenuItems/Queries/GetMenuItems/`:
Sử dụng `IMapFrom<MenuItem>` để tự động mapping.

```csharp
public class GetMenuItemsResponse : IMapFrom<MenuItem>
{
    public Guid MenuItemId { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal PriceDineIn { get; set; }
    // ... các trường khác
}
```

### Bước 2: Tạo Query & Handler

Tạo file `GetMenuItemsQuery.cs` cùng thư mục:

```csharp
public record GetMenuItemsQuery(PaginationParams Pagination) : IRequest<Result<PagedResult<GetMenuItemsResponse>>>;

public class GetMenuItemsQueryHandler : IRequestHandler<GetMenuItemsQuery, Result<PagedResult<GetMenuItemsResponse>>>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public GetMenuItemsQueryHandler(IUnitOfWork unitOfWork, IMapper mapper)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    public async Task<Result<PagedResult<GetMenuItemsResponse>>> Handle(GetMenuItemsQuery request, CancellationToken cancellationToken)
    {
        var query = _unitOfWork.Repository<MenuItem>().Query();

        // Áp dụng search, filter, sort nếu cần (xem phần 8)

        var result = await query
            .ProjectTo<GetMenuItemsResponse>(_mapper.ConfigurationProvider)
            .ToPagedResultAsync(request.Pagination);

        return Result<PagedResult<GetMenuItemsResponse>>.Success(result);
    }
}
```

### Bước 3: Tạo Controller

Controllers trong project kế thừa trực tiếp từ `ControllerBase` và inject `IMediator`. Sử dụng helper method `HandleResult` để chuẩn hóa response.

```csharp
[ApiController]
[Route("api/[controller]")]
public class MenuItemsController : ControllerBase
{
    private readonly IMediator _mediator;
    public MenuItemsController(IMediator mediator) => _mediator = mediator;

    private IActionResult HandleResult<T>(Result<T> result)
    {
        if (result.IsSuccess)
        {
            if (result.HasWarning) return Ok(new { data = result.Data, warning = result.Warning });
            return Ok(result.Data);
        }

        return result.ErrorType switch
        {
            ResultErrorType.NotFound => NotFound(new { message = result.Error }),
            ResultErrorType.Unauthorized => Unauthorized(new { message = result.Error }),
            ResultErrorType.Conflict => Conflict(new { message = result.Error }),
            _ => BadRequest(new { message = result.Error })
        };
    }

    [HttpGet]
    public async Task<IActionResult> GetMenuItems([FromQuery] PaginationParams pagination)
    {
        var result = await _mediator.Send(new GetMenuItemsQuery(pagination));
        return HandleResult(result);
    }
}
```

---

## 5. Các Patterns Quan Trọng

### Result Pattern

Sử dụng `Result<T>` để trả về kết quả thành công hoặc lỗi từ tầng Application.

- **Thành công**: `Result<T>.Success(data)` hoặc `Result<T>.SuccessWithWarning(data, "Lưu ý...")`
- **Thất bại**: `Result<T>.Failure("Thông báo lỗi", ResultErrorType.BadRequest)` hoặc các method shortcut như `Result<T>.NotFound("Không tìm thấy")`.

### Validation Behavior

Mọi Command được gửi qua MediatR sẽ tự động được kiểm tra bởi các lớp kế thừa `AbstractValidator<T>`. Nếu có lỗi, hệ thống sẽ throw `ValidationException` và trả về mã lỗi 400.

### AutoMapper (IMapFrom)

Interface `IMapFrom<T>` giúp tự động cấu hình Mapping. Mặc định nó sẽ thực hiện `CreateMap<T, GetType>().ReverseMap()`.

```csharp
// Trong DTO
public class MenuItemDto : IMapFrom<MenuItem> { }

// Nếu cần custom mapping:
public void Mapping(Profile profile)
{
    profile.CreateMap<MenuItem, MenuItemDto>()
        .ForMember(d => d.CategoryName, opt => opt.MapFrom(s => s.Category.Name));
}
```

### Unit of Work & Generic Repository

Dùng để quản lý dữ liệu và transaction.

- `_unitOfWork.Repository<T>().Query()`: Lấy IQueryable.
- `_unitOfWork.Repository<T>().AddAsync(entity)`: Thêm mới.
- `_unitOfWork.SaveChangeAsync()`: Thực thi lưu xuống DB.

---

## 6. EF Core Migrations

Chạy lệnh Migration tại thư mục gốc `FoodHub_BE`:

1. **Thêm Migration:** `dotnet ef migrations add [Name] --project Infrastructure --startup-project .`
2. **Cập nhật Database:** `dotnet ef database update --project Infrastructure --startup-project .`
3. **Xóa Migration cuối:** `dotnet ef migrations remove --project Infrastructure --startup-project .`

---

## 7. Hướng Dẫn Chạy Project

1. Cấu hình Connection String trong `appsettings.json` hoặc `.env`.
2. Đảm bảo Docker Desktop đã chạy để khởi động Redis và (tùy chọn) PostgreSQL.
3. Chạy lệnh: `dotnet run`.

---

## 8. Hệ Thống Search, Filter & Sort Đa Năng

Sử dụng `QueryableExtension.cs` để xử lý tập trung việc tìm kiếm, lọc và phân trang.

```csharp
var searchableFields = new List<Expression<Func<Employee, string?>>> {
    u => u.FullName, u => u.Email
};
query = query.ApplyGlobalSearch(request.Pagination.Search, searchableFields);

var filterMapping = new Dictionary<string, Expression<Func<Employee, object?>>> {
    { "status", u => u.Status },
    { "role", u => u.Role }
};
query = query.ApplyFilters(request.Pagination.Filters, filterMapping);

var sortMapping = new Dictionary<string, Expression<Func<Employee, object?>>> {
    { "fullname", u => u.FullName },
    { "createdAt", u => u.CreatedAt }
};
query = query.ApplySorting(request.Pagination.OrderBy, sortMapping, u => u.Id);
```

---

## 9. Caching với Redis

Project sử dụng `IDistributedCache` để tương tác với Redis. Container được đặt tên là `foodhub_redis`.

---

## 10. Quản lý Message & Đa ngôn ngữ (Localization)

Resources nằm tại `Application/Resources/`:

- `ErrorMessages.resx`: Chứa các key báo lỗi.
- `Messages.resx`: Chứa các key thông báo thành công.

Inject `IStringLocalizer<ErrorMessages>` để lấy chuỗi thông báo theo ngôn ngữ (dựa vào header `Accept-Language`).

---

## 11. Hướng Dẫn Chi Tiết Sử Dụng Các Công Nghệ Chính

Phần này cung cấp hướng dẫn chi tiết cách sử dụng các công nghệ chính trong dự án FoodHub Backend. Mỗi công nghệ có file hướng dẫn riêng trong thư mục `FoodHub_Docs/`:

- [ASP.NET Core 9.0](ASP.NET_Core_9.0.md) - Framework web chính
- [Entity Framework Core 9.0](Entity_Framework_Core_9.0.md) - ORM cho database
- [MediatR](MediatR.md) - CQRS pattern implementation
- [FluentValidation](FluentValidation.md) - Validation framework
- [AutoMapper](AutoMapper.md) - Object mapping
- [JWT Bearer Authentication](JWT_Bearer_Authentication.md) - Authentication
- [Serilog](Serilog.md) - Structured logging
- [xUnit](xUnit.md) - Unit testing
- [API Versioning](API_Versioning.md) - API versioning
- [Response Compression](Response_Compression.md) - HTTP compression
- [CORS](CORS.md) - Cross-origin resource sharing
- [Localization](Localization.md) - Multi-language support
- [Background Jobs](Background_Jobs.md) - Asynchronous tasks
- [BCrypt](BCrypt.md) - Password hashing
- [Rate Limiting](Rate_Limiting.md) - API protection
- [SMTP](SMTP.md) - Email sending

---

_Tài liệu này được cập nhật với hướng dẫn chi tiết các công nghệ (Tháng 02/2026)._
