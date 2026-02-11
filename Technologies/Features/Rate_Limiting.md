# Hướng Dẫn Sử Dụng Rate Limiting

## Mô tả

Giới hạn số lượng request để bảo vệ API khỏi abuse.

## Cách sử dụng trong dự án

- Cấu hình trong `Program.cs` sử dụng AspNetCoreRateLimit.

## Ví dụ cấu hình

```csharp
// Program.cs
builder.Services.AddMemoryCache();
builder.Services.Configure<IpRateLimitOptions>(builder.Configuration.GetSection("IpRateLimiting"));
builder.Services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
builder.Services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();
builder.Services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
builder.Services.AddSingleton<IProcessingStrategy, AsyncKeyLockProcessingStrategy>();

var app = builder.Build();
app.UseIpRateLimiting();
```
