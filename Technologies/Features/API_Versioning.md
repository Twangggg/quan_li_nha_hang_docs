# Hướng Dẫn Sử Dụng API Versioning

## Mô tả

Cho phép versioning API để duy trì tương thích ngược khi có thay đổi.

## Cách sử dụng trong dự án

- Cấu hình trong `Program.cs`.
- Sử dụng attribute `[ApiVersion]` trên controllers.

## Ví dụ cấu hình

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});
```

## Ví dụ sử dụng

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class EmployeesController : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetEmployees() { /* ... */ }
}

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class EmployeesController : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetEmployeesV2() { /* ... */ }
}
```
