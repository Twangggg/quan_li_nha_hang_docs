# Hướng Dẫn Sử Dụng CORS (Cross-Origin Resource Sharing)

## Mô tả

Cho phép hoặc chặn các request từ domain khác.

## Cách sử dụng trong dự án

- Cấu hình trong `Program.cs` để cho phép frontend truy cập API.

## Ví dụ cấu hình

```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:3000", "https://foodhub.com")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();
    });
});

var app = builder.Build();
app.UseCors("AllowFrontend");
```
