# Hướng Dẫn Sử Dụng SMTP

## Mô tả

Giao thức gửi email qua server SMTP.

## Cách sử dụng trong dự án

- Cấu hình trong `EmailService` để gửi email thông báo.

## Ví dụ cấu hình

```csharp
// appsettings.json
{
  "Email": {
    "SmtpHost": "smtp.gmail.com",
    "SmtpPort": 587,
    "SenderEmail": "your-email@gmail.com",
    "SenderName": "FoodHub",
    "AppPassword": "your-app-password"
  }
}

// EmailService.cs
public class EmailService : IEmailService
{
    private readonly SmtpClient _smtpClient;
    private readonly string _senderEmail;
    private readonly string _senderName;

    public EmailService(IConfiguration configuration)
    {
        _smtpClient = new SmtpClient(configuration["Email:SmtpHost"])
        {
            Port = int.Parse(configuration["Email:SmtpPort"]!),
            Credentials = new NetworkCredential(
                configuration["Email:SenderEmail"],
                configuration["Email:AppPassword"]),
            EnableSsl = true
        };
        _senderEmail = configuration["Email:SenderEmail"]!;
        _senderName = configuration["Email:SenderName"]!;
    }

    public async Task SendEmailAsync(EmailMessage message)
    {
        var mailMessage = new MailMessage
        {
            From = new MailAddress(_senderEmail, _senderName),
            Subject = message.Subject,
            Body = message.Body,
            IsBodyHtml = true
        };
        mailMessage.To.Add(message.To);

        await _smtpClient.SendMailAsync(mailMessage);
    }
}
```
