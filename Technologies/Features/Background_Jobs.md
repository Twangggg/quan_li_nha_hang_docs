# Hướng Dẫn Sử Dụng Background Jobs

## Mô tả

Chạy các tác vụ nền không đồng bộ, như gửi email.

## Cách sử dụng trong dự án

- Sử dụng `IBackgroundEmailSender` và `EmailBackgroundWorker`.
- Đăng ký trong `Infrastructure/DependencyInjection.cs`.

## Ví dụ sử dụng

```csharp
public class EmailService : IEmailService
{
    private readonly IBackgroundEmailSender _backgroundEmailSender;

    public EmailService(IBackgroundEmailSender backgroundEmailSender)
    {
        _backgroundEmailSender = backgroundEmailSender;
    }

    public async Task SendWelcomeEmailAsync(string email, string name)
    {
        var message = new EmailMessage
        {
            To = email,
            Subject = "Welcome to FoodHub",
            Body = $"Hello {name}, welcome to our restaurant management system!"
        };

        await _backgroundEmailSender.SendEmailAsync(message);
    }
}
```
