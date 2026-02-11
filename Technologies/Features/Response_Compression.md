# Hướng Dẫn Sử Dụng Response Compression

## Mô tả

Nén response HTTP để giảm kích thước dữ liệu truyền tải và cải thiện hiệu suất.

## Cách sử dụng trong dự án

- Cấu hình trong `Program.cs` để hỗ trợ Gzip/Brotli.

## Ví dụ cấu hình

```csharp
// Program.cs
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<GzipCompressionProvider>();
    options.Providers.Add<BrotliCompressionProvider>();
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Optimal;
});

var app = builder.Build();
app.UseResponseCompression();
```
