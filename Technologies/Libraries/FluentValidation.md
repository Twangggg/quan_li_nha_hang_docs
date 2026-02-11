# Hướng Dẫn Sử Dụng FluentValidation

## Mô tả

Thư viện validation mạnh mẽ cho các đối tượng .NET, hỗ trợ validation phức tạp và tùy chỉnh.

## Cách sử dụng trong dự án

- Sử dụng để validate các request DTOs.
- Tự động áp dụng qua MediatR pipeline.

## Ví dụ sử dụng

```csharp
public class CreateEmployeeCommandValidator : AbstractValidator<CreateEmployeeCommand>
{
    public CreateEmployeeCommandValidator()
    {
        RuleFor(x => x.FullName)
            .NotEmpty().WithMessage("Full name is required")
            .MaximumLength(100).WithMessage("Full name must not exceed 100 characters");

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format");

        RuleFor(x => x.Role)
            .IsInEnum().WithMessage("Invalid employee role");
    }
}
```
