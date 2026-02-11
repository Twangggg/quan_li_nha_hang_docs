# Hướng Dẫn Sử Dụng Serilog

## Mô tả

Thư viện logging cấu trúc hóa, hỗ trợ ghi log ra nhiều đích khác nhau (file, console, database).

## Cách sử dụng trong dự án

- Cấu hình trong `Program.cs` để ghi log structured.
- Sử dụng `ILogger<T>` để ghi log.

## Ví dụ cấu hình

```csharp
// Program.cs
builder.Host.UseSerilog((context, config) =>
{
    config.WriteTo.Console()
          .WriteTo.File("logs/foodhub-.txt", rollingInterval: RollingInterval.Day)
          .Enrich.FromLogContext()
          .ReadFrom.Configuration(context.Configuration);
});
```

## Ví dụ sử dụng

```csharp
private readonly ILogger<MenuItemsController> _logger;

public MenuItemsController(ILogger<MenuItemsController> logger)
{
    _logger = logger;
}

[HttpPost]
public async Task<IActionResult> CreateMenuItem(CreateMenuItemCommand command)
{
    _logger.LogInformation("Creating menu item: {Name}", command.Name);

    var result = await _mediator.Send(command);

    if (result.IsSuccess)
        _logger.LogInformation("Menu item created successfully: {Id}", result.Data.Id);
    else
        _logger.LogError("Failed to create menu item: {Error}", result.Error);

    return HandleResult(result);
}
```
