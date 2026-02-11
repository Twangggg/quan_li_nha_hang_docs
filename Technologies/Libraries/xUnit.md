# Hướng Dẫn Sử Dụng xUnit

## Mô tả

Framework testing cho .NET, hỗ trợ unit testing, integration testing.

## Cách sử dụng trong dự án

- Viết tests trong project `FoodHub.Tests`.
- Chạy tests bằng `dotnet test`.

## Ví dụ test

```csharp
public class EmployeeServiceTests
{
    private readonly Mock<IUnitOfWork> _unitOfWorkMock;
    private readonly EmployeeService _service;

    public EmployeeServiceTests()
    {
        _unitOfWorkMock = new Mock<IUnitOfWork>();
        _service = new EmployeeService(_unitOfWorkMock.Object);
    }

    [Fact]
    public async Task GetEmployeeById_ShouldReturnEmployee_WhenExists()
    {
        // Arrange
        var employeeId = Guid.NewGuid();
        var employee = new Employee { Id = employeeId, FullName = "Test Employee" };
        _unitOfWorkMock.Setup(u => u.Repository<Employee>().GetByIdAsync(employeeId))
            .ReturnsAsync(employee);

        // Act
        var result = await _service.GetEmployeeByIdAsync(employeeId);

        // Assert
        Assert.True(result.IsSuccess);
        Assert.Equal(employeeId, result.Data.Id);
    }
}
```
