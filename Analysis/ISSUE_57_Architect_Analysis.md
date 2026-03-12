## Bước 1 — Phân tích Feature

- Module: Inventory (Settings + Opening Stock). Gồm Command (lưu cấu hình, nhập tồn đầu kỳ) và Query (tải cấu hình, tải danh sách NVL active cho grid).
- Ảnh hưởng: InventoryTransaction, Ingredient stock/cost, auto-deduct trong order fulfillment (flag bật/tắt), expiry warning logic.
- Transaction: Cần cho Opening Stock (multi-write update stock/cost + insert transaction). Settings update đơn ghi, không bắt buộc transaction riêng.
- Cache: Có thể cache InventorySettings (read-mostly); phải invalidation khi cập nhật.
- Domain event: Tùy kiến trúc, có thể phát InventorySettingsUpdated và OpeningStockImported cho audit/analytics hoặc warning jobs.
- Flow Settings Update: Controller `/settings/inventory` → Command `UpdateInventorySettingsCommand` → Handler → Domain (validate/set) → Repo InventorySettings → Response.
- Flow Settings Query: Controller → Query `GetInventorySettingsQuery` → Handler → Repo → Response.
- Flow Opening Stock Command: Controller `/settings/opening-stock` → Command `ImportOpeningStockCommand` (list items) → Handler → Domain (Ingredient.SetOpeningStock, InventoryTransaction.CreateOpeningStock) → Repos Ingredient/InventoryTransaction (+ UnitOfWork) → Response.
- Flow Opening Stock Query: Controller → Query `GetOpeningStockListQuery` → Handler → Repo (ingredients active, projection) → Response.

## Bước 2 — Đối chiếu kiến trúc

- Clean Architecture: Domain không biết Application/Infra; Handler chỉ orchestrate, không tự gán state entity (tuân FFA-CAG). Controller mỏng, không chứa logic.
- Boundaries: InventorySettings, Ingredient, InventoryTransaction tách biệt; không kéo logic giá vốn/cảnh báo vào Controller/Handler.
- Coupling/transaction: Giữ auto-deduct flag chỉ là cấu hình; không nhúng order logic vào flow này. Transaction chỉ bao DB writes, không bọc external calls.
- Cache invalidation: Cập nhật settings phải clear cache tương ứng.

## Bước 3 — Đề xuất kiến trúc triển khai

- Folder/Files (ví dụ):
  - `FoodHub.Application/InventorySettings/Queries/GetInventorySettings/{Query,Handler,Response}.cs`
  - `FoodHub.Application/InventorySettings/Commands/UpdateInventorySettings/{Command,Handler,Validator}.cs`
  - `FoodHub.Application/Inventory/OpeningStock/Queries/GetOpeningStockList/{Query,Handler,Response}.cs`
  - `FoodHub.Application/Inventory/OpeningStock/Commands/ImportOpeningStock/{Command,Handler,Validator}.cs`
  - Domain: `InventorySettings` (singleton guard, Update method), `Ingredient` (SetOpeningStock(quantity, costPrice?)), `InventoryTransaction` factory `CreateOpeningStock(...)`; optional domain service `OpeningStockService` nếu cần phối hợp nhiều entity.
- Transaction boundary: Trong `ImportOpeningStockHandler` dùng UnitOfWork transaction (Begin → writes → Commit; Rollback trong catch). Settings update không cần transaction riêng nếu chỉ một upsert.
- Logging (FFA-LOG): Log start/end info; warn/error khi business fail; tránh log dữ liệu nhạy cảm.
- Validation (FFA-ACV/ERR): FluentValidation
  - Settings: `ExpiryWarningDays >=1`, `DefaultLowStockThreshold >=0`, `MaxCostRecalcDays 1..365`, `AutoDeductOnCompleted` bool.
  - OpeningStock items: ingredientId required, quantity >=0, cost >=0 (nullable), list not empty.
- Exception strategy (FFA-ERR): Dùng `BusinessException`/`NotFoundException`; không throw raw `Exception`; không catch-swallow trong Handler, để middleware map HTTP.
- Controller (FFA-CTL/SEC): Kế thừa ApiControllerBase, inject Mediator, dùng `HandleResult`, `[Authorize(Roles="Manager")]` cho endpoints.
- Test strategy (FFA-TST):
  - Handler tests: UpdateInventorySettings (happy, validation fail); ImportOpeningStock (happy multi-items, ingredient not found, negative qty validation, existing stock warning path nếu có policy).
  - Domain tests: Ingredient.SetOpeningStock cập nhật stock/cost; InventoryTransaction.CreateOpeningStock.
  - Query tests: GetInventorySettings returns defaults when missing; GetOpeningStockList chỉ NVL active, projection đúng.

### Bảng feature cần có — Inventory Settings
| Feature | Mục đích | Ghi chú triển khai |
| --- | --- | --- |
| Xem cấu hình kho | Hiển thị ExpiryWarningDays, DefaultLowStockThreshold, AutoDeductOnCompleted, CostMethod (read-only), MaxCostRecalcDays | Query `GetInventorySettings`; tự tạo record mặc định nếu chưa có |
| Cập nhật cấu hình | Cho phép Manager chỉnh các trường trên, lưu singleton | Command `UpdateInventorySettings`; invalidate cache sau khi lưu |
| Validation cấu hình | Đảm bảo giá trị hợp lệ: ExpiryWarningDays ≥1; DefaultLowStockThreshold ≥0; MaxCostRecalcDays 1..365 | FluentValidation + Business/ValidationException mapping qua middleware |
| Auto-deduct toggle | Bật/tắt tự trừ kho khi Order Completed | Flag lưu trong settings; order handler đọc flag, không xử lý ở Controller |
| Cost method cố định | CostMethod = Bình quân gia quyền, không cho chỉnh | Field read-only trong UI/DTO; không cập nhật qua command |
| Giới hạn tính lại giá vốn | MaxCostRecalcDays điều khiển khung thời gian recalc giá vốn | Dùng trong domain/service tính cost; validator 1..365 |
| Trạng thái Opening Stock | Quản lý giai đoạn đầu kỳ: Pending/Completed + LockedAt | Lưu trong InventorySettings để kiểm soát nhập đầu kỳ, chặn ghi đè khi Completed |

## Bước 4 — Đánh giá rủi ro

- 🔴 Critical: Multi-write không có transaction; Handler tự gán `CurrentStock/CostPrice`; thiếu validation dẫn tới stock âm.
- 🟠 Architectural: Controller/Handler chứa logic giá vốn/cảnh báo; quên cache invalidation; coupling sang order module khi xử lý auto-deduct flag.
- 🟡 Maintainability: Thiếu domain method/factory → lặp code cập nhật stock/cost; thiếu test cho handler.
- 🔵 Performance: Query opening stock không cần pagination (danh sách NVL active), nhưng tránh N+1 và dùng projection + AsNoTracking.

## Bước 5 — Kết luận

- Triển khai được theo issue gốc, không cần refine nghiệp vụ.
- Giữ Clean Architecture: domain methods cho cập nhật stock/settings, transaction guard cho opening stock, validator & tests theo FFA-TST, controller mỏng dùng Mediator và authorize Manager.
