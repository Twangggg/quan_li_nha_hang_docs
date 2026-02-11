# Hướng Dẫn Sử Dụng BCrypt

## Mô tả

Thuật toán hash mật khẩu an toàn.

## Cách sử dụng trong dự án

- Sử dụng trong `PasswordService` để hash và verify mật khẩu.

## Ví dụ sử dụng

```csharp
public class PasswordService : IPasswordService
{
    public string HashPassword(string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password);
    }

    public bool VerifyPassword(string password, string hashedPassword)
    {
        return BCrypt.Net.BCrypt.Verify(password, hashedPassword);
    }
}
```
