# Hướng Dẫn Sử Dụng AutoMapper

## Mô tả

Thư viện mapping tự động giữa các đối tượng, giúp chuyển đổi giữa Entities và DTOs.

## Cách sử dụng trong dự án

- Sử dụng interface `IMapFrom<T>` để tự động cấu hình mapping.
- Cấu hình trong `Application/Extensions/Mappings/MappingProfile.cs`.

## Ví dụ sử dụng

```csharp
// DTO với mapping tự động
public class EmployeeDto : IMapFrom<Employee>
{
    public Guid Id { get; set; }
    public string FullName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

// Mapping tùy chỉnh
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<Employee, EmployeeDto>()
            .ForMember(dest => dest.FullName, opt => opt.MapFrom(src => src.FullName.ToUpper()));
    }
}

// Sử dụng
var dto = _mapper.Map<EmployeeDto>(employee);
```
