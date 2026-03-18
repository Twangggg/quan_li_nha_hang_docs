# US-INV-05 — Inventory Check & Report (Architect Analysis)

## Bước 1 — Phân tích Feature
- Module: Inventory; Command (tạo/ xử lý phiếu kiểm kê, sinh StockIn/OutReceipt, cập nhật CurrentStock, ghi InventoryTransaction) và Query (danh sách phiếu, báo cáo tồn kho, ledger).
- Ảnh hưởng: liên quan Ingredient, StockIn/Out receipts; sale deduction đọc từ POS/Order; giá trị tồn phụ thuộc CostPrice/Pricing.
- Transaction: bắt buộc cho xử lý chênh lệch (multi-write). Query không cần transaction.
- Cache: tránh cache dữ liệu tồn thời gian thực; báo cáo có thể cache ngắn hạn với invalidation sau giao dịch mới (optional).
- Domain event: cân nhắc `InventoryAdjusted`/`InventoryChecked` sau commit cho audit/alert; không có external integration.
- Flow: Controller → Command/Query → Handler → Domain Service/Entity → Repository (UoW) → Response DTO.

## Bước 2 — Đối chiếu kiến trúc
- Clean Architecture: Controller mỏng, chỉ dispatch; Handler orchestration; tính chênh lệch, quyết định nhập/xuất, cập nhật stock nằm trong Domain service/entity.
- Boundary: Inventory không gọi trực tiếp POS/Order; chỉ đọc dữ liệu qua interface. Domain không biết Application/Infra (tuân FFA-CAG).
- Transaction guard: multi-write phải có Begin/Commit/Rollback, không bọc external call (FFA-TXG).
- Logging: log start/end, structured, không log secret (FFA-LOG). Validation ở Application; exception theo FFA-ERR.

## Bước 3 — Đề xuất kiến trúc triển khai
- Folder/File (ví dụ .NET + MediatR):
  - Controllers/Inventory/InventoryCheckController.cs (routes list/create/process)
  - Application/Inventory/InventoryChecks/Commands/CreateInventoryCheckCommand.cs + Handler
  - Application/Inventory/InventoryChecks/Commands/ProcessInventoryCheckCommand.cs + Handler
  - Application/Inventory/InventoryChecks/Queries/GetInventoryChecksQuery.cs + Handler
  - Application/Inventory/Reports/Queries/GetInventoryReportQuery.cs + Handler
  - Application/Inventory/Reports/Queries/GetInventoryLedgerQuery.cs + Handler
  - Domain/Inventory/InventoryCheck.cs, InventoryCheckItem.cs (factory + methods: CreateDraft, ApplyPhysicalCount, MarkProcessed)
  - Domain service: InventoryAdjustmentService quyết định sinh StockIn/OutReceipt, cập nhật CurrentStock, tạo InventoryTransaction.
- Transaction boundary: bao phủ xử lý chênh lệch (tạo receipts, update Ingredient.CurrentStock, ghi InventoryTransaction, đổi trạng thái InventoryCheck). Queries không cần transaction.
- Logging level: Information start/end; Warning/Error khi fail; structured placeholders.
- Validation: FluentValidation cho Command/Query (ID bắt buộc, physical count ≥ 0, date range hợp lệ, state chưa processed).
- Exception: NotFoundException khi không thấy phiếu/nguyên liệu; BusinessException khi phiếu đã xử lý hoặc state sai; middleware map HTTP.
- Tests (FFA-TST):
  - CreateInventoryCheckHandlerTests: happy tạo draft với stock sổ; validation fail (items rỗng/ID thiếu).
  - ProcessInventoryCheckHandlerTests: surplus → StockInReceipt + update stock + transaction; deficit → StockOutReceipt; already processed → BusinessException; not found → NotFoundException; rollback khi write thứ 2 lỗi.
  - GetInventoryReportQueryHandlerTests: đúng opening/in/out/closing/value theo date window; GetInventoryLedgerQueryHandlerTests: filter by type/date.

## Bước 4 — Đánh giá rủi ro
- 🔴 Critical: Bỏ transaction khi multi-write; Handler tự mutate entity (anemic); thiếu rollback.
- 🟠 Architectural: Đặt business logic ở Controller/Handler; coupling trực tiếp POS/Order để trừ tồn; bypass domain service.
- 🟡 Maintainability: Thiếu domain service cho điều chỉnh tồn; thiếu validation/state machine; lặp tính toán.
- 🔵 Performance: Báo cáo/ledger không index (IngredientId, Date); cache sai thời điểm.

## Bước 5 — Kết luận
- Có thể triển khai theo issue gốc; cần refine: tách Command/Query rõ, logic trong Domain service/entity, bảo đảm transaction + logging + validation + exception chuẩn FFA-ERR/FFA-TXG/FFA-LOG.
- Đề xuất thêm: index ledger/report (IngredientId, Date); domain event InventoryAdjusted; mapper/DTO chuẩn; cache ngắn hạn cho báo cáo kèm invalidation sau giao dịch.
