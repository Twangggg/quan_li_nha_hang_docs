# Hướng Dẫn Sử Dụng MediatR

## Mô tả

Thư viện triển khai pattern Mediator để tách biệt các thành phần và giảm coupling giữa chúng, thường dùng cho CQRS.

## Cách sử dụng trong dự án

- Sử dụng cho việc xử lý Commands và Queries trong tầng Application.
- Đăng ký trong `Application/DependencyInjection.cs`.

## Ví dụ sử dụng

```csharp
// Định nghĩa Query
public record GetEmployeeByIdQuery(Guid Id) : IRequest<Result<EmployeeDto>>;

// Handler
public class GetEmployeeByIdQueryHandler : IRequestHandler<GetEmployeeByIdQuery, Result<EmployeeDto>>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public GetEmployeeByIdQueryHandler(IUnitOfWork unitOfWork, IMapper mapper)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
    }

    public async Task<Result<EmployeeDto>> Handle(GetEmployeeByIdQuery request, CancellationToken cancellationToken)
    {
        var employee = await _unitOfWork.Repository<Employee>().GetByIdAsync(request.Id);
        if (employee == null) return Result<EmployeeDto>.NotFound("Employee not found");

        var dto = _mapper.Map<EmployeeDto>(employee);
        return Result<EmployeeDto>.Success(dto);
    }
}

// Sử dụng trong Controller
[HttpGet("{id}")]
public async Task<IActionResult> GetEmployee(Guid id)
{
    var result = await _mediator.Send(new GetEmployeeByIdQuery(id));
    return HandleResult(result);
}
```
