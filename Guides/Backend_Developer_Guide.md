# Hướng Dẫn Phát Triển Backend (Backend Developer Guide)

Tài liệu này là hướng dẫn cho lập trình viên tham gia dự án FoodHub BE. Nó mô tả chi tiết tất cả các tiêu chuẩn từ kiến trúc đến việc triển khai code thực tế.

---

## 1. Kiến Trúc Hệ Thống (Architecture)

Hệ thống được xây dựng theo kiến trúc **Clean Architecture** kết hợp với **CQRS** (MediatR).

- **Domain**: Thực thể (Entities) kế thừa `BaseEntity`, Enum và Logic lõi.
- **Application**: Command/Query, Handlers, DTOs, Mappings, và Validation.
- **Infrastructure**: Database, Security, Redis, external services.
- **Presentation (WebAPI)**: Controllers và Middlewares.

---

## 2. Công Nghệ Sử Dụng (Tech Stack)

Chi tiết cấu hình và hướng dẫn sử dụng từng công nghệ:

### 🛠 Frameworks & Core

- [ASP.NET Core 9.0](../Technologies/Frameworks/ASP.NET_Core_9.0.md)
- [Entity Framework Core 9.0](../Technologies/Frameworks/Entity_Framework_Core_9.0.md)

### 📚 Thư viện & Chức năng

- [MediatR](../Technologies/Libraries/MediatR.md) | [AutoMapper](../Technologies/Libraries/AutoMapper.md) | [FluentValidation](../Technologies/Libraries/FluentValidation.md)
- [Serilog](../Technologies/Libraries/Serilog.md) | [xUnit](../Technologies/Libraries/xUnit.md)
- [Phân quyền (Policy-based Auth)](../Technologies/Features/Policy_based_Authorization.md)
- [Localization](../Technologies/Features/Localization.md) | [Background Jobs](../Technologies/Features/Background_Jobs.md)

---

## 3. Quy trình thêm tính năng mới (Step-by-Step)

### Bước 1: Domain

Tạo Entity mới trong `Domain/Entities/`. Luôn kế thừa từ `BaseEntity`.

### Bước 2: Database

1. Đăng ký Entity trong `AppDbContext`.
2. Chạy Migration: `dotnet ef migrations add [Name] -p Infrastructure -s WebAPI`.
3. Update DB: `dotnet ef database update`.

### Bước 3: Application (Trọng tâm)

1. **Command/Query**: Định nghĩa request class. Nếu là danh sách, phải có `PaginationParams`.
2. **DTO**: Tạo Response DTO.
   - Dùng `IMapFrom<Entity>` cho các DTO đơn dụng.
   - Nếu cần map thủ công hoặc cấu hình phức tạp, ghi đè method `Mapping(Profile profile)`.
3. **Permission**: Thêm hằng số vào `Permissions.cs` và ánh xạ trong `PermissionProvider.cs`.
4. **Validator**: Tạo class kế thừa `AbstractValidator<T>`. Tầng MediatR (`ValidationBehavior`) sẽ tự động chạy validate này.
5. **Handler**: Triển khai logic điều phối (Orchestration). **Lưu ý**: Logic nghiệp vụ quan trọng nên đẩy vào Domain Entity.

### Bước 4: Presentation (WebAPI)

1. **Controller**: Kế thừa `ApiControllerBase`.
2. **API Documentation**:
   - Viết XML Comments (`/// <summary>`, `<param>`, `<response>`).
   - Dùng `[ProducesResponseType]` để Swagger hiển thị rõ các schema trả về.
   - Dùng `[HasPermission]` cho các endpoint cần bảo mật.

---

## 4. "Giải phẫu" một Handler hoàn hảo

Một Handler đạt chuẩn trong FoodHub cần phối hợp nhiều service để đảm bảo tính an toàn và khả năng quan sát:

```csharp
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Result<Guid>>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;
    private readonly ILogger<CreateOrderHandler> _logger;
    private readonly IMessageService _messageService;
    private readonly ICurrentUserService _currentUserService;

    public CreateOrderHandler(...) { /* Inject all */ }

    public async Task<Result<Guid>> Handle(CreateOrderCommand request, CancellationToken token)
    {
        _logger.LogInformation("Creating order for table {TableId}", request.TableId);

        // 1. Kiểm tra User & Authentication
        var userId = _currentUserService.UserId;

        // 2. Map & Logic nghiệp vụ (Rich Domain Model)
        // Thay vì viết logic kiểm tra Table bận/rảnh ở đây, hãy để Entity/Service xử lý
        var order = _mapper.Map<Order>(request);

        // 3. Persist dữ liệu
        await _unitOfWork.Repository<Order>().AddAsync(order);
        await _unitOfWork.SaveChangeAsync(token);

        // 4. Invalidate Cache
        // await _cacheService.RemoveByPatternAsync(CacheKey.OrderList);

        return Result<Guid>.Success(order.OrderId);
    }
}
```

---

## 5. Domain Logic & Rich Domain Model

Tuyệt đối tránh **Anemic Domain Model** (Entity chỉ có getter/setter).

- **Entity**: Chứa logic thay đổi trạng thái và validate tính nhất quán của dữ liệu.
- **Handler**: Chỉ gọi các phương thức từ Entity và lưu xuống Database.

**Ví dụ trong `Order.cs`**:

```csharp
public DomainResult Cancel() {
    if (Status != OrderStatus.Serving) return DomainResult.Failure(DomainErrors.Order.InvalidStatus);
    Status = OrderStatus.Cancelled;
    CancelledAt = DateTime.UtcNow;
    return DomainResult.Success();
}
```

In Handler: `var result = order.Cancel(); if (!result.IsSuccess) return Result.Failure(...);`

---

## 5. Persistence - IUnitOfWork & Generic Repository

Dùng để quản lý dữ liệu và transaction một cách nhất quán.

### 5.1. Transaction Management

Khi luồng nghiệp vụ ảnh hưởng đến nhiều bảng hoặc cần tính toàn vẹn cao (vd: `CreateEmployee`), hãy sử dụng Transaction:

```csharp
await _unitOfWork.BeginTransactionAsync();
try {
    // 1. Thêm dữ liệu vào nhiều bảng
    await _unitOfWork.Repository<Employee>().AddAsync(employee);
    await _unitOfWork.Repository<AuditLog>().AddAsync(auditLog);

    // 2. Lưu thay đổi vào DB trước khi commit
    await _unitOfWork.SaveChangeAsync(ct);

    // 3. Commit
    await _unitOfWork.CommitTransactionAsync();
} catch (Exception) {
    // 4. Rollback nếu có lỗi
    await _unitOfWork.RollbackTransactionAsync();
    throw;
}
```

---

## 6. Caching - ICacheService (Redis)

Dự án sử dụng Redis để tăng tốc độ truy xuất dữ liệu danh sách hoặc dữ liệu ít thay đổi.

- **Quy tắc đặt Key**: `feature:sub-feature:id` (Ví dụ: `employee:list:p1`, `menu:detail:guid`).
- **Invalidation (Xóa cache)**: Khi dữ liệu bị thay đổi (Create/Update/Delete), phải xóa cache liên quan.
  - Sử dụng `RemoveByPatternAsync("feature:*")` để xóa hàng loạt key có chung tiền tố.
- **Serialization**: `ICacheService` tự động xử lý JSON Serialization cho bạn.

---

## 7. Security - Phân Quyền (Authorization)

Hệ thống sử dụng **Policy-based Authorization**.

- **Attribute**: `[HasPermission(Permissions.Orders.Create)]`.
- **Cơ chế**: `PermissionPolicyProvider` tự động tạo Policy -> `PermissionHandler` kiểm tra Claim "Permission" trong Token.
- **Tài liệu chi tiết**: [Policy-based Authorization](../Technologies/Features/Policy_based_Authorization.md).

---

## 8. Logging & Observability

Chuẩn hóa việc ghi log để dễ dàng truy vết lỗi:

- **LogInformation**: Cho các bước quan trọng trong luồng ("Starting process...", "Success...").
- **LogWarning**: Cho các vi phạm quy tắc nghiệp vụ (Dữ liệu không hợp lệ, không tìm thấy).
- **Audit Logging**: Luôn ghi lại "ai đã làm gì" vào bảng `AuditLog` cho các thao tác thay đổi dữ liệu nhạy cảm.

---

## 9. Localization & Messaging

- **IMessageService**: Dùng `GetMessage(key)` để lấy nội dung từ resource file.
- **MessageKeys**: Tuyệt đối không hardcode chuỗi thông báo. Sử dụng hằng số trong `MessageKeys`.

---

## 10. Hệ Thống Search, Filter, Sort & Phân Trang

Sử dụng `QueryableExtension.cs` để xử lý list linh hoạt:

```csharp
var query = _unitOfWork.Repository<Entity>().Query();

// 1. Search (Global)
query = query.ApplyGlobalSearch(request.Pagination.Search, new List<Expression<Func<Entity, string?>>> { x => x.Name });

// 2. Filter (Specific fields)
query = query.ApplyFilters(request.Pagination.Filters, new Dictionary<string, Expression<Func<Entity, object?>>> {
    { "status", x => x.Status }
});

// 3. Sort & Paginate
var pagedResult = await query
    .ProjectTo<ResponseDto>(_mapper.ConfigurationProvider) // Tối ưu SQL
    .ApplySorting(request.Pagination.OrderBy, sortMapping, defaultSort)
    .ToPagedResultAsync(request.Pagination);
```

---

## 11. Tài liệu API (API Documentation Standards)

Để có Swagger đẹp và dễ dùng cho Frontend:

1. **XML Comments**: Bắt buộc cho mọi Public Action.
2. **ProducesResponseType**: Khai báo rõ kiểu trả về cho từng mã lỗi.
3. **HandleResult**: Sử dụng helper trong `ApiControllerBase` để tự động hóa map mã lỗi.

```csharp
/// <summary>
/// Lấy thông tin chi tiết một món ăn.
/// </summary>
/// <param name="id">ID món ăn.</param>
/// <response code="200">Thành công.</response>
/// <response code="404">Không tìm thấy.</response>
[HttpGet("{id}")]
[ProducesResponseType(typeof(Result<MenuItemDto>), StatusCodes.Status200OK)]
public async Task<IActionResult> GetById(Guid id)
{
    var result = await _mediator.Send(new GetMenuItemByIdQuery(id));
    return HandleResult(result);
}
```

---

## 11. Checklist PR "Thần Thánh" (The Ultimate Checklist)

Một lập trình viên chuyên nghiệp tại FoodHub phải vượt qua checklist này trước khi gửi PR:

### 🛠 Thiết kế & Cấu trúc

- [ ] Entity đã kế thừa `BaseEntity` (Id, CreatedAt, UpdatedAt)?
- [ ] Đã định nghĩa Permission mới trong `Permissions.cs`?
- [ ] Permission mới đã được ánh xạ vào Role thích hợp trong `PermissionProvider` chưa?

### 💻 Triển khai (Handler)

- [ ] Đã Inject đúng các service cần thiết (`IUnitOfWork`, `ILogger`, `IMessageService`)?
- [ ] Các thông báo lỗi/thành công đã qua `IMessageService` (không hardcode)?
- [ ] Luồng có nhiều bảng đã được bọc trong `BeginTransactionAsync` chưa?
- [ ] Đã gọi `SaveChangeAsync` trước khi `CommitTransaction`?
- [ ] Nếu là Query danh sách, đã dùng `ToPagedResultAsync`?

### 🔒 Bảo mật & Dữ liệu

- [ ] Controller đã có `[HasPermission]` cho endpoint mới?
- [ ] Đã xử lý xóa Cache (`RemoveAsync` hoặc `RemoveByPatternAsync`) khi dữ liệu thay đổi?
- [ ] Đã ghi `AuditLog` cho các thao tác quan trọng?

### 👁️ Khả năng quan sát (Observability)

- [ ] Có `LogInformation` khi bắt đầu và kết thúc Handler?
- [ ] Có `LogWarning` khi trả về `Result.Failure` (kèm theo lý do và ID liên quan)?
- [ ] Tuyệt đối không log dữ liệu nhạy cảm (Password, Token).

### 🧪 Hoàn thiện

- [ ] Đã chạy `dotnet build` và không có lỗi/cảnh báo?
- [ ] Đã viết/cập nhật Unit Test phủ các trường hợp thành công và thất bại chính?

---

_Cập nhật bởi ToanTK (Tháng 02/2026)._
