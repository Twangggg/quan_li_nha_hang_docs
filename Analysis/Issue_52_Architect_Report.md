# 🏛️ Kiến Trúc Khởi tạo: Issue 52 Module Báo Cáo (Analytics)

Dựa trên yêu cầu của **Issue 52** và bộ tiêu chuẩn **FoodHub Architecture (FFA Rules)**, dưới đây là bản phân tích kiến trúc triển khai để đảm bảo tính chuẩn xác, bảo mật và hiệu năng cho hệ thống, phục vụ việc đối soát và phân tích doanh thu.

---

## BƯỚC 1 — Phân tích Feature

- **Thuộc module**: `Analytics / Reports`.
- **Loại tác vụ**: Thuần **Query**. Feature này chủ yếu đọc dữ liệu từ DB (Read-only), không thay đổi state của các Entity (Order, Table).
- **Phụ thuộc module khác**: Truyền thông tin từ module Orders (để lấy danh sách đơn, thanh toán) và Shifts (thông tin ca làm việc).
- **Yêu cầu kỹ thuật cốt lõi**:
  - Không cần Transaction (vì thuần Query).
  - Có thể tích hợp **Cache** (cho Monthly/Daily Report) vì dữ liệu quá khứ ít biến động.
  - Không cần Domain Event hay External Integration.
- **Workflow (Pipeline chuẩn theo FFA-FLW)**:
  `ReportController` → `Query Validator` → `Log Start` → `Auth Check` → `Query Handler (Query Logic)` → `Repository (Dùng AsNoTracking + Select)` → `Mapping (nếu chưa map)` → `Log End` → `HandleResult`.

---

## BƯỚC 2 — Đối chiếu kiến trúc (Theo chuẩn FFA)

1. **Architecture (FFA-CAG & FFA-CTL)**:
   - Các Query Handler nằm trong `Application/Features/Reports/Queries/`.
   - `ReportController` phải kế thừa `ApiControllerBase`, không được chứa if/else nghiệp vụ, trả kết quả bằng `HandleResult()`.
2. **Performance (FFA-PERF)**:
   - **Đặc biệt lưu ý**: Báo cáo tổng doanh thu sẽ tổng hợp lượng lớn Order. BẮT BUỘC dùng aggregation query ở DB level (`SUM`, `COUNT` qua `.GroupBy()`), KHÔNG load hàng triệu Entity bằng `.ToListAsync()` xuống memory rồi tính tổng bằng LINQ to Objects.
   - BẮT BUỘC có `AsNoTracking()`.
   - Danh sách "Món bán chạy" (Best Sellers) nếu quá lớn phải có cơ chế Skip/Take (Cursor).
3. **Security (FFA-SEC)**:
   - Tất cả Endpoints cần `[Authorize]`.
   - Cần phân quyền: `Manager` xem toàn bộ. `Cashier` **chỉ được xem báo cáo Ca** và có thể cần ownership check (chỉ xem ca của chính họ).
4. **Error Handling & Validation (FFA-ERR & FFA-ACV)**:
   - Các filter param như `StartDate`, `EndDate` phải được validate qua `FluentValidation` (VD: StartDate phải nhỏ hơn EndDate). Thường sẽ throw `ValidationException`.
   - Response DTO phải strictly định nghĩa các trường (decimal cho doanh thu), không rò rỉ ID nội bộ không cần thiết.

---

## BƯỚC 3 — Đề xuất kiến trúc triển khai chi tiết

### Cấu trúc Folder & Files

Tạo trong `FoodHub.WebAPI`:

- `Controllers/ReportController.cs` (Tiếp nhận endpoints `GET /api/reports/shift/{id}`, `GET /api/reports/daily`, `GET /api/reports/monthly`, `GET /api/reports/best-sellers`).

Tạo trong `FoodHub.Application/Features/Reports/`:

- `Queries/GetShiftReport/`
  - `GetShiftReportQuery.cs` (Param: ShiftId)
  - `GetShiftReportQueryValidator.cs`
  - `GetShiftReportQueryHandler.cs`
  - `ShiftReportResponse.cs` (DTO)
- `Queries/GetDailyReport/`
  - `GetDailyReportQuery.cs` (Param: Date)
  - _...các file tương tự_
- `Queries/GetBestSellers/`
  - `GetBestSellersQuery.cs` (Param: StartDate, EndDate, Limit)
  - _...các file tương tự_

### Phân ranh giới & Logging

- **Logging Level**: Ghi log `Information` khi nhận query, và hoàn tất. Ghi log parameters nhưng TUYỆT ĐỐI KHÔNG chứa dữ liệu nhạy cảm. (FFA-LOG)
- **Validation Layer**: Sử dụng kĩ thuật DateRange Validator tiêu chuẩn để lọc request lỗi từ sớm. (FFA-ACV)

### Test Strategy (FFA-TST)

Cần bổ sung các Test Class trong `FoodHub.Tests/Features/Reports/`:

- `GetShiftReportQueryHandlerTests.cs`
- `GetBestSellersQueryHandlerTests.cs`
- Phạm vi Test:
  - **Happy Path:** Đảm bảo Repository Mock trả về số liệu, Handler tính đúng aggregation và mapping đúng Response.
  - **Error Path:** Dữ liệu trống (trả về 0 hoặc List rỗng), hoặc ShiftId truyền vào bị Null/Not Found.

---

## BƯỚC 4 — Đánh giá Rủi ro (Risk Assessment)

- 🔴 **CRITICAL RISK (Security - FFA-SEC):** Rò rỉ báo cáo tổng quan cho nhân viên thông thường.
  - _Phòng ngừa_: Bắt buộc route `Daily / Monthly / Best-Sellers` chỉ định `[Authorize(Roles = "Manager")]`. Route `Shift` dùng `[Authorize(Roles = "Manager, Cashier")]` kèm Handler logic để chặn Cashier xem ca khác.
- 🟠 **ARCHITECTURAL RISK (Anemic Domain - FFA-CAG):**
  - _Nguy cơ_: Handler tự đi gán doanh thu bằng tay hoặc parse dữ liệu bẩn.
  - _Phòng ngừa_: Trả về báo cáo dưới dạng **Query / View**, không cố nhồi nhét vào logic thay đổi state Entity.
- 🟡 **MAINTAINABILITY RISK (Validation - FFA-ACV):**
  - _Nguy cơ_: Không giới hạn mức độ truy xuất ngày của Best Sellers (VD query suốt 10 năm liền).
  - _Phòng ngừa_: Thêm `RuleFor` giới hạn (EndDate - StartDate) <= 365 ngày để hệ thống không bị ngẽn.
- 🔵 **PERFORMANCE RISK (DB Overload - FFA-PERF):**
  - _Nguy cơ_: Load tất cả Order items bằng LINQ, gây N+1 hoặc OutOfMemory.
  - _Phòng ngừa_: Repository phải có các custom query (`Select new BestSellerResponse { ... }`) dùng `GroupBy` SQL-side execution.

---

## BƯỚC 5 — Kết luận & Hành động

- **Nhận định**: Issue gốc 52 đủ cơ sở để đi vào cài đặt kỹ thuật, nhưng **cần Refine** thêm ở phần bảo mật phân quyền (phải strict theo Role) và mức độ truy xuất giới hạn của Best Seller (thêm limit pagination hoặc dải ngày max).
- **Quyết định Triển khai**: Mẫu thiết kế đã tuân thủ FFA và đạt chuẩn Clean Architecture. Tiến trình kế tiếp sẽ là cài đặt code.
