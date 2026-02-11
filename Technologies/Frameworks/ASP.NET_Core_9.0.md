# Hướng Dẫn Sử Dụng ASP.NET Core 9.0

## Mô tả

ASP.NET Core là framework web đa nền tảng, hiệu suất cao để xây dựng các ứng dụng web và API hiện đại.

## Cách sử dụng trong dự án

- Dự án sử dụng ASP.NET Core 9.0 cho việc xây dựng RESTful API.
- Cấu hình trong `Program.cs` với các service như authentication, authorization, CORS, v.v.

## Ví dụ cấu hình

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Thêm services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Cấu hình JWT
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* cấu hình JWT */ });

// Cấu hình CORS
builder.Services.AddCors(options => {
    options.AddPolicy("AllowSpecificOrigins", policy => {
        policy.WithOrigins("http://localhost:3000")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

var app = builder.Build();

// Sử dụng middleware
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.UseCors("AllowSpecificOrigins");
app.MapControllers();
app.Run();
```

## Lưu ý

Luôn cập nhật .NET SDK để sử dụng các tính năng mới nhất của ASP.NET Core 9.0.
