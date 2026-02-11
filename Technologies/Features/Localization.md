# Hướng Dẫn Sử Dụng Localization

## Mô tả

Hỗ trợ đa ngôn ngữ cho ứng dụng.

## Cách sử dụng trong dự án

- Sử dụng resource files trong `Application/Resources/`.
- Inject `IStringLocalizer` để lấy chuỗi theo ngôn ngữ.

## Ví dụ sử dụng

```csharp
// Resources/Messages.resx
// Key: EmployeeCreated, Value: Employee created successfully

private readonly IStringLocalizer<Messages> _localizer;

public EmployeesController(IStringLocalizer<Messages> localizer)
{
    _localizer = localizer;
}

[HttpPost]
public async Task<IActionResult> CreateEmployee(CreateEmployeeCommand command)
{
    var result = await _mediator.Send(command);
    if (result.IsSuccess)
    {
        return Ok(new { message = _localizer["EmployeeCreated"] });
    }
    return BadRequest(new { message = result.Error });
}
```
