# ARCHITECT ANALYSIS — ISSUE 81: Inventory Check & Report

> **Mode**: FoodHub Internal Architect Mode
> **Date**: 2026-03-18
> **Goal**: Bám sát issue gốc để dev, QA, và reviewer có thể trace trực tiếp từ business flow sang thiết kế triển khai

---

## BƯỚC 1 — Phân tích Feature theo đúng Issue

### 1.1 Mục tiêu nghiệp vụ

Issue yêu cầu giải quyết 4 nhu cầu chính:

1. So sánh tồn sổ sách với tồn thực tế để phát hiện chênh lệch.
2. Tự điều chỉnh tồn kho về đúng thực tế bằng phiếu nhập hoặc xuất kiểm kê.
3. Cung cấp báo cáo tồn kho theo thời gian.
4. Cho phép truy vết lịch sử biến động kho của từng nguyên liệu.

### 1.2 Actor và trách nhiệm

| Actor | Trách nhiệm theo issue |
| --- | --- |
| `Manager` | Tạo phiếu kiểm kê, xử lý chênh lệch, xem báo cáo tồn kho, xem ledger nguyên liệu |
| `System` | Tự sinh `StockInReceipt` hoặc `StockOutReceipt`, ghi `InventoryTransaction`, cập nhật `Ingredient.CurrentStock` |

### 1.3 Entity liên quan

| Entity | Vai trò |
| --- | --- |
| `InventoryCheck` | Phiếu kiểm kê, có trạng thái `Draft` hoặc `Processed` |
| `InventoryCheckItem` | Dòng chi tiết từng nguyên liệu trong phiếu kiểm kê |
| `Ingredient` | Nguồn tồn sổ sách và nơi cập nhật `CurrentStock` sau xử lý |
| `StockInReceipt` | Sinh ra khi tồn thực tế lớn hơn tồn sổ |
| `StockOutReceipt` | Sinh ra khi tồn thực tế nhỏ hơn tồn sổ |
| `InventoryTransaction` | Log biến động kho để phục vụ ledger và audit |

### 1.4 Preconditions phải tôn trọng

| Điều kiện | Ý nghĩa kiến trúc |
| --- | --- |
| `US-INV-01` hoàn thành | `Ingredient` đã tồn tại và có `CurrentStock` |
| `US-INV-02` hoàn thành | Có luồng nhập kho và entity `StockInReceipt` |
| `US-INV-03` hoàn thành | Có luồng xuất kho và entity `StockOutReceipt` |

### 1.5 Feature decomposition

| Capability | Loại | Mục tiêu |
| --- | --- | --- |
| Xem danh sách phiếu kiểm kê | Query | Lọc theo trạng thái, khoảng ngày, phân trang |
| Tạo phiếu kiểm kê | Command | Lưu phiếu ở trạng thái `Draft` với tồn sổ và tồn thực tế |
| Xử lý chênh lệch kiểm kê | Command | Sinh phiếu nhập/xuất kiểm kê, cập nhật tồn, ghi transaction, đổi trạng thái |
| Xem báo cáo tồn kho | Query | Trả về tồn đầu kỳ, nhập, xuất, tồn cuối, đơn giá, giá trị tồn |
| Xem ledger nguyên liệu | Query | Trả về lịch sử biến động theo loại giao dịch và khoảng ngày |

---

## BƯỚC 2 — Trace trực tiếp từ User Flow sang Thiết kế

### 2.1 Flow 1: Danh sách phiếu kiểm kê

**Yêu cầu issue**

- Route: `/inventory/check`
- Hiển thị danh sách phiếu kiểm kê
- Có filter theo trạng thái
- Có filter theo khoảng ngày
- Có phân trang

**Thiết kế đề xuất**

- Query: `GetInventoryChecksQuery`
- Input:
  - `Status?`
  - `FromDate?`
  - `ToDate?`
  - `PageNumber`
  - `PageSize`
- Output tối thiểu:
  - `InventoryCheckId`
  - `CheckDate`
  - `Status`
  - `CreatedBy`
  - `TotalItems`

**Lưu ý**

- Issue chưa yêu cầu sort rule cụ thể, nên có thể mặc định sort `CheckDate desc, CreatedAt desc`.
- Không tự thêm business logic ngoài các filter nêu trong issue.

### 2.2 Flow 2: Tạo phiếu kiểm kê

**Yêu cầu issue**

- Manager bấm `+ Tạo phiếu kiểm kê`
- Hệ thống hiển thị tất cả nguyên liệu
- `Tồn theo sổ = Ingredient.CurrentStock`
- Manager nhập `Tồn thực tế`
- Hệ thống tự tính `Chênh lệch = Tồn thực tế - Tồn theo sổ`
- Manager nhập `Nguyên nhân` nếu có sai lệch
- Bấm `Lưu`
- Phiếu được lưu ở trạng thái `Nháp`

**Thiết kế đề xuất**

- Query để load grid tạo phiếu:
  - `GetInventoryCheckCreateFormQuery`
- Command để lưu phiếu:
  - `CreateInventoryCheckCommand`

**Command payload tối thiểu**

| Field | Bắt buộc | Ghi chú |
| --- | --- | --- |
| `CheckDate` | Có | Ngày kiểm kê hoặc ngày lập phiếu |
| `Items[].IngredientId` | Có | Mỗi nguyên liệu một dòng |
| `Items[].BookQuantity` | Có | Chụp snapshot từ `Ingredient.CurrentStock` tại thời điểm tạo |
| `Items[].PhysicalQuantity` | Có | Manager nhập |
| `Items[].DifferenceQuantity` | Có | Có thể tính server-side để tránh lệch client |
| `Items[].Reason` | Không | Nhưng nên cho phép nhập khi có chênh lệch |

**Quy tắc quan trọng**

- `BookQuantity` nên được snapshot khi tạo phiếu, không đọc lại từ `Ingredient.CurrentStock` lúc xử lý.
- `DifferenceQuantity` nên do server tính lại từ `PhysicalQuantity - BookQuantity`.
- Phiếu mới tạo luôn ở trạng thái `Draft`.

### 2.3 Flow 3: Xử lý chênh lệch tồn kho

**Yêu cầu issue**

- Khi `Thực tế > sổ sách`
  - Sinh `StockInReceipt`
  - `Type = INVENTORY_ADJUSTMENT`
  - `Quantity = chênh lệch dương`
- Khi `Thực tế < sổ sách`
  - Sinh `StockOutReceipt`
  - `Type = INVENTORY_ADJUSTMENT`
  - `Quantity = chênh lệch âm`
- Sau xử lý:
  - `Ingredient.CurrentStock = Tồn thực tế`
  - Tạo `InventoryTransaction`
  - `Type = INVENTORY_CHECK`
  - `Reference = InventoryCheckId`
  - Phiếu chuyển sang `Đã xử lý`

**Thiết kế đề xuất**

- Command: `ProcessInventoryCheckCommand`

**Handler phải làm đủ 4 việc trong một transaction**

1. Kiểm tra phiếu tồn tại và đang ở trạng thái `Draft`.
2. Duyệt từng `InventoryCheckItem` để xác định surplus hoặc deficit.
3. Tạo receipt tương ứng, cập nhật `Ingredient.CurrentStock`, ghi `InventoryTransaction`.
4. Đổi trạng thái phiếu sang `Processed`.

**Transaction boundary**

Toàn bộ flow xử lý chênh lệch là multi-write, nên bắt buộc cùng một transaction:

- insert `StockInReceipt` hoặc `StockOutReceipt`
- update `Ingredient.CurrentStock`
- insert `InventoryTransaction`
- update `InventoryCheck.Status`

Nếu một bước lỗi thì rollback toàn bộ.

**Không nên suy diễn thêm**

- Issue không nói đến approval step, partial processing, hay multi-stage review.
- Vì vậy thiết kế nên là xử lý một lần, atomically, từ `Draft` sang `Processed`.

### 2.4 Flow 4: Báo cáo tồn kho

**Yêu cầu issue**

- Route: `/inventory/report`
- Filter theo:
  - khoảng thời gian
  - nguyên liệu cụ thể
- Hiển thị:
  - tồn đầu kỳ
  - tổng nhập
  - tổng xuất
  - tồn cuối kỳ
  - đơn giá bình quân
  - giá trị tồn

**Công thức trong issue**

| Chỉ số | Rule theo issue |
| --- | --- |
| `Tồn đầu kỳ` | Tồn trước ngày `From` |
| `Tổng nhập trong kỳ` | `SUM(StockIn)` |
| `Tổng xuất trong kỳ` | `SUM(StockOut + SaleDeduction)` |
| `Tồn cuối kỳ` | `Tồn đầu + Nhập - Xuất` |
| `Giá trị tồn` | `Tồn cuối x CostPrice` |

**Điểm cần chốt để tránh lệch issue**

- Issue có đồng thời nhắc `CostPrice` và AC lại yêu cầu hiển thị `Đơn giá bình quân`.
- Vì vậy response báo cáo nên trả rõ cả:
  - `AverageUnitCost`
  - `ClosingStockValue`
- Nếu hệ thống hiện dùng `Ingredient.CostPrice` như đơn giá bình quân hiện hành, cần ghi rõ giả định này trong code comment hoặc tech note.

**Thiết kế đề xuất**

- Query: `GetInventoryReportQuery`
- Filter:
  - `FromDate`
  - `ToDate`
  - `IngredientId?`
  - pagination là optional, issue chưa bắt buộc

**Output tối thiểu**

| Field | Ý nghĩa |
| --- | --- |
| `IngredientId` | Khóa nguyên liệu |
| `IngredientName` | Tên hiển thị |
| `OpeningStock` | Tồn đầu kỳ |
| `TotalStockIn` | Tổng nhập trong kỳ |
| `TotalStockOut` | Tổng xuất kho trong kỳ |
| `TotalSaleDeduction` | Tổng xuất do bán hàng |
| `TotalOutbound` | `StockOut + SaleDeduction` |
| `ClosingStock` | Tồn cuối kỳ |
| `AverageUnitCost` | Đơn giá bình quân |
| `ClosingStockValue` | Giá trị tồn cuối kỳ |

### 2.5 Flow 5: Inventory Ledger

**Yêu cầu issue**

- Từ báo cáo, Manager bấm vào một nguyên liệu để xem log
- Có filter theo:
  - loại giao dịch
  - khoảng thời gian
- Ledger phải hiển thị:
  - thời điểm
  - loại giao dịch
  - số phiếu
  - biến động (+/-)
  - tồn sau giao dịch

**Loại giao dịch cần hỗ trợ**

- `INITIAL_STOCK`
- `STOCK_IN`
- `STOCK_OUT`
- `SALE_DEDUCTION`
- `INVENTORY_CHECK`
- `STOCK_OUT_REVERSE`

**Thiết kế đề xuất**

- Query: `GetInventoryLedgerQuery`

**Output tối thiểu**

| Field | Ý nghĩa |
| --- | --- |
| `OccurredAt` | Thời điểm phát sinh |
| `TransactionType` | Loại giao dịch |
| `ReferenceNo` | Số phiếu hoặc mã tham chiếu |
| `QuantityDelta` | Biến động `+/-` |
| `BalanceAfter` | Tồn sau giao dịch |
| `Note` | Ghi chú nếu có |

---

## BƯỚC 3 — Đối chiếu Acceptance Criteria

### AC-01: Tạo phiếu kiểm kê

**Issue yêu cầu**

- Load danh sách nguyên liệu
- `Tồn theo sổ = Ingredient.CurrentStock`
- Nhập `Tồn thực tế`
- Tự tính `Chênh lệch`

**Thiết kế phải đáp ứng**

- Có query load form create
- Có command create draft
- Server phải tính lại `DifferenceQuantity`
- Không cho client tự quyết định `BookQuantity` sai với snapshot lúc tạo

### AC-02: Kiểm kê khi thực tế lớn hơn sổ sách

**Issue yêu cầu**

- Tạo phiếu nhập kho kiểm kê
- Cập nhật `CurrentStock = 600g`
- Ghi `InventoryTransaction`
- `Type = INVENTORY_CHECK`

**Thiết kế phải đáp ứng**

- Receipt phải là `StockInReceipt`
- Receipt type phải là `INVENTORY_ADJUSTMENT`
- `InventoryTransaction.Reference` phải trỏ tới `InventoryCheckId`

### AC-03: Kiểm kê khi thực tế nhỏ hơn sổ sách

**Issue yêu cầu**

- Tạo phiếu xuất kho kiểm kê
- Cập nhật `CurrentStock = 800g`

**Thiết kế phải đáp ứng**

- Receipt phải là `StockOutReceipt`
- Receipt type phải là `INVENTORY_ADJUSTMENT`
- Không được bỏ qua bước cập nhật `Ingredient.CurrentStock`

### AC-04: Báo cáo tồn kho

**Issue yêu cầu**

- Hiển thị đúng:
  - `Tồn đầu kỳ`
  - `Tổng nhập`
  - `Tổng xuất`
  - `Tồn cuối kỳ`
  - `Đơn giá bình quân`
  - `Giá trị tồn`

**Thiết kế phải đáp ứng**

- Logic query phải tính riêng `StockOut` và `SaleDeduction`
- Response phải có trường đơn giá
- Test phải verify đúng công thức, không chỉ verify có dữ liệu trả về

### AC-05: Lịch sử biến động kho

**Issue yêu cầu**

- Hiển thị log gồm:
  - thời điểm
  - loại giao dịch
  - số phiếu
  - biến động
  - tồn sau giao dịch

**Thiết kế phải đáp ứng**

- `InventoryTransaction` hoặc projection ledger phải có đủ dữ liệu để render 5 cột này
- Không chỉ trả loại giao dịch và ngày; phải có cả `ReferenceNo` và `BalanceAfter`

---

## BƯỚC 4 — Đề xuất Kiến trúc Triển khai

### 4.1 Tách Command và Query

| Nhóm | Thành phần |
| --- | --- |
| Commands | `CreateInventoryCheckCommand`, `ProcessInventoryCheckCommand` |
| Queries | `GetInventoryChecksQuery`, `GetInventoryCheckCreateFormQuery`, `GetInventoryReportQuery`, `GetInventoryLedgerQuery` |

### 4.2 Cấu trúc file đề xuất

```text
FoodHub.Application/
  Inventory/
    InventoryChecks/
      Commands/
        CreateInventoryCheck/
        ProcessInventoryCheck/
      Queries/
        GetInventoryChecks/
        GetInventoryCheckCreateForm/
    Reports/
      Queries/
        GetInventoryReport/
        GetInventoryLedger/

FoodHub.Domain/
  Inventory/
    InventoryCheck.cs
    InventoryCheckItem.cs
    Services/
      InventoryAdjustmentService.cs
```

### 4.3 Phân chia trách nhiệm theo layer

| Layer | Trách nhiệm |
| --- | --- |
| Controller | Nhận request, dispatch Mediator, không chứa business logic |
| Application Handler | Orchestrate flow, validate, mở transaction, gọi domain/repository |
| Domain Entity/Service | Tính chênh lệch, enforce state `Draft -> Processed`, quyết định tạo adjustment |
| Infrastructure | Persist receipts, transactions, projections, query optimization |

### 4.4 Domain rules nên có

| Rule | Mục tiêu |
| --- | --- |
| `InventoryCheck` chỉ được process một lần | Chặn double-processing |
| `InventoryCheck` phải ở `Draft` trước khi process | Enforce state machine tối thiểu |
| `DifferenceQuantity = Physical - Book` | Tránh client gửi sai |
| `Ingredient.CurrentStock = PhysicalQuantity` sau xử lý | Bám đúng issue |

### 4.5 Validation bắt buộc

| Scenario | Validation |
| --- | --- |
| Tạo phiếu | danh sách item không rỗng |
| Tạo phiếu | `IngredientId` hợp lệ |
| Tạo phiếu | `PhysicalQuantity >= 0` |
| Xử lý phiếu | phiếu tồn tại |
| Xử lý phiếu | phiếu chưa `Processed` |
| Báo cáo/ledger | `FromDate <= ToDate` |

### 4.6 Exception mapping

| Tình huống | Exception |
| --- | --- |
| Không tìm thấy phiếu hoặc nguyên liệu | `NotFoundException` |
| Phiếu đã xử lý hoặc state không hợp lệ | `BusinessException` |
| Input không hợp lệ | `ValidationException` |

### 4.7 Logging

- Log bắt đầu và kết thúc ở command/query handler
- Log structured cho `InventoryCheckId`, `IngredientId`, số item, thời gian xử lý
- Không log payload thừa hoặc dữ liệu nhạy cảm

---

## BƯỚC 5 — Test Strategy bám đúng Issue

### 5.1 Command tests

| Test class | Case quan trọng |
| --- | --- |
| `CreateInventoryCheckHandlerTests` | tạo draft thành công với snapshot `BookQuantity` |
| `CreateInventoryCheckHandlerTests` | reject khi item rỗng hoặc `PhysicalQuantity < 0` |
| `ProcessInventoryCheckHandlerTests` | surplus tạo `StockInReceipt` loại `INVENTORY_ADJUSTMENT` |
| `ProcessInventoryCheckHandlerTests` | deficit tạo `StockOutReceipt` loại `INVENTORY_ADJUSTMENT` |
| `ProcessInventoryCheckHandlerTests` | cập nhật `Ingredient.CurrentStock = PhysicalQuantity` |
| `ProcessInventoryCheckHandlerTests` | ghi `InventoryTransaction` với `Type = INVENTORY_CHECK` và `Reference = InventoryCheckId` |
| `ProcessInventoryCheckHandlerTests` | rollback toàn bộ khi lỗi ở bước ghi receipt hoặc transaction |
| `ProcessInventoryCheckHandlerTests` | reject khi phiếu đã `Processed` |

### 5.2 Query tests

| Test class | Case quan trọng |
| --- | --- |
| `GetInventoryChecksQueryHandlerTests` | filter theo trạng thái, khoảng ngày, phân trang |
| `GetInventoryReportQueryHandlerTests` | tính đúng `OpeningStock` |
| `GetInventoryReportQueryHandlerTests` | tính đúng `TotalStockIn` |
| `GetInventoryReportQueryHandlerTests` | tính đúng `TotalOutbound = StockOut + SaleDeduction` |
| `GetInventoryReportQueryHandlerTests` | trả đúng `AverageUnitCost` và `ClosingStockValue` |
| `GetInventoryLedgerQueryHandlerTests` | filter theo loại giao dịch và khoảng ngày |
| `GetInventoryLedgerQueryHandlerTests` | projection đủ `OccurredAt`, `ReferenceNo`, `QuantityDelta`, `BalanceAfter` |

---

## BƯỚC 6 — Rủi ro nếu triển khai lệch Issue

### 🔴 Critical

| Risk | Hậu quả |
| --- | --- |
| Không dùng transaction khi process kiểm kê | Receipt, stock, transaction, status có thể lệch nhau |
| Dùng `CurrentStock` hiện tại thay vì snapshot `BookQuantity` của phiếu | Chênh lệch kiểm kê bị sai khi stock đã đổi sau lúc tạo phiếu |
| Không cập nhật `Ingredient.CurrentStock = PhysicalQuantity` | Sai trực tiếp so với issue và AC |

### 🟠 Architectural

| Risk | Hậu quả |
| --- | --- |
| Nhét logic xử lý chênh lệch vào controller | Khó test, khó reuse, sai clean architecture |
| Ledger projection không có `ReferenceNo` hoặc `BalanceAfter` | Không đáp ứng AC-05 |
| Báo cáo chỉ tính `StockOut`, bỏ `SaleDeduction` | Sai AC-04 |

### 🟡 Maintainability

| Risk | Hậu quả |
| --- | --- |
| Không tách rõ query create-form và query report | Response DTO lẫn lộn, khó mở rộng |
| Không có state rule `Draft -> Processed` | Dễ double-process |

---

## BƯỚC 7 — Kết luận

### Verdict

> **Triển khai được và đã bám sát issue**, với điều kiện thiết kế giữ đúng 3 trọng tâm sau:
>
> 1. Phiếu kiểm kê phải snapshot `BookQuantity` lúc tạo và chỉ process một lần.
> 2. Flow xử lý chênh lệch phải chạy trong một transaction và cập nhật đúng `CurrentStock`, receipt, transaction, status.
> 3. Query báo cáo và ledger phải trả đúng các cột mà acceptance criteria yêu cầu, đặc biệt là `SaleDeduction`, `AverageUnitCost`, `ReferenceNo`, và `BalanceAfter`.

### Checklist trước khi implement

- [ ] Chốt enum trạng thái `Draft` và `Processed` cho `InventoryCheck`
- [ ] Chốt field snapshot `BookQuantity`, `PhysicalQuantity`, `DifferenceQuantity`, `Reason` cho `InventoryCheckItem`
- [ ] Chốt receipt type `INVENTORY_ADJUSTMENT`
- [ ] Chốt `InventoryTransaction.Type = INVENTORY_CHECK` và `Reference = InventoryCheckId`
- [ ] Chốt cách xác định `AverageUnitCost` để không mâu thuẫn giữa AC và công thức `CostPrice`
- [ ] Viết test bám đủ `AC-01` đến `AC-05`
