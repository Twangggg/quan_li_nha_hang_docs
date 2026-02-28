# ARCHITECT ANALYSIS — ISSUE 51: KDS (Kitchen Display System)

> **Mode**: FoodHub Internal Architect Mode  
> **Date**: 2026-02-28  
> **Skills Applied**: FFA-FLW, FFA-TXG, FFA-ERR, FFA-TST

---

## BƯỚC 1 — Phân tích Feature

### 1.1 Module Classification

| Dimension            | Kết luận                                             |
| -------------------- | ---------------------------------------------------- |
| **Module**           | `KDS` (Kitchen Display System) — BOH module          |
| **Primary Entity**   | `OrderItem` (BOH Owned)                              |
| **Secondary Entity** | `Order` (FOH Owned — read-only từ KDS)               |
| **Audit Entity**     | `OrderAuditLog` (ghi tất cả thay đổi trạng thái KDS) |

### 1.2 Command vs Query phân tách

#### Commands (Write Operations — cần transaction)

| Command                  | Trigger                 | Transition             |
| ------------------------ | ----------------------- | ---------------------- |
| `StartCookingCommand`    | Chef bấm hoặc auto-pull | `PREPARING → COOKING`  |
| `MarkReadyCommand`       | Chef bấm Done           | `COOKING → READY`      |
| `RejectOrderItemCommand` | Manager/Chef-Bar reject | `COOKING → REJECTED`   |
| `ReturnOrderItemCommand` | Manager return          | `REJECTED → PREPARING` |

#### Queries (Read Operations — không cần transaction)

| Query              | Dữ liệu trả về                                       |
| ------------------ | ---------------------------------------------------- |
| `GetKdsItemsQuery` | Danh sách OrderItem theo station, sorted by priority |
| `GetKdsQueueQuery` | Queue PREPARING items chờ xử lý                      |

### 1.3 Module Dependency Analysis

```
KDS Module
  ├── READS   → Order (FOH Owned) — chỉ đọc context
  ├── WRITES  → OrderItem (BOH Owned) — điều khiển state
  ├── WRITES  → OrderAuditLog — ghi mọi state transition
  └── NOTIFIES → FOH via SignalR (real-time sync)
```

**Cross-module impact:**

- `OrderItems` module: KDS tái sử dụng entity nhưng KHÔNG reuse handler. KDS có handler riêng.
- `Orders` module: KDS chỉ đọc `Order.Status`, không write.
- `SignalR Hub`: KDS push realtime update về FOH sau mỗi state change.

### 1.4 Key Business Requirements Mapping

| Requirement     | Cần gì                                                                      |
| --------------- | --------------------------------------------------------------------------- |
| Station Routing | `OrderItem.StationSnapshot` (đã có)                                         |
| Priority Queue  | Computed từ `CreatedAt`, `IsPriority`, `Quantity` — không cần DB column mới |
| WIP Limit       | Count query theo station + status COOKING                                   |
| Auto-pull       | Trigger trong `StartCookingCommand` hoặc Background Job                     |
| Audit Log       | `OrderAuditLog` (đã có entity)                                              |
| Realtime sync   | `ISignalRService` (cần thêm interface)                                      |

### 1.5 Flow Diagram

```
FOH submits Order
       │
       ▼
OrderItem.Status = PREPARING (BOH nhận)
       │
       ▼
[GetKdsItemsQuery] — Hiển thị theo station + priority order
       │
       ▼
[StartCookingCommand] — Auto-pull hoặc Chef bấm
  ┌─ Check WIP limit < Threshold ─┐
  │  Pick top PREPARING item      │
  └─ Transition → COOKING ────────┘
  Audit: PREPARING→COOKING | SignalR: notify FOH
       │
       ├──────────────────────────────────────┐
       ▼                                      ▼
[MarkReadyCommand]                 [RejectOrderItemCommand]
  COOKING → READY                    COOKING → REJECTED
  Audit + SignalR                    Reason bắt buộc (BR-KDS-06)
                                     Giải phóng WIP slot
                                     Audit + SignalR
                                           │
                                           ▼
                                  [ReturnOrderItemCommand]
                                    Manager only
                                    REJECTED → PREPARING
                                    Recalculate priority
                                    Audit + SignalR
```

---

## BƯỚC 2 — Đối chiếu Kiến trúc

### 2.1 Domain Model Assessment

**OrderItem** đã có:

- ✅ `StationSnapshot` — dùng cho station routing
- ✅ `Status` enum với đủ `Preparing`, `Cooking`, `Ready`, `Rejected`
- ✅ `CreatedAt` — dùng cho FIFO fallback
- ❌ `RejectionReason` — thiếu (xem phân tích §2.4 bên dưới)
- ⚠️ `ChangeStatus()` hiện tại không enforce KDS-specific transitions

**OrderAuditLog đã có:**

- ✅ `Action`, `OldValue`, `NewValue`, `ChangeReason` — đủ cho KDS audit

### 2.2 Clean Architecture Compliance

| Layer          | Nhận xét                                                                     |
| -------------- | ---------------------------------------------------------------------------- |
| Domain         | Cần thêm KDS-specific domain methods (xem §3.1)                              |
| Application    | WIP limit check đúng chỗ: Handler, KHÔNG phải Domain                         |
| Application    | Role check đúng chỗ: Handler + `ICurrentUserService`                         |
| WebAPI         | Controller chỉ delegate command — không có business logic                    |
| Infrastructure | `ISignalRService` cần được tạo tại Application, implement tại Infrastructure |

### 2.3 Dependency Direction

```
WebAPI → Application → Domain
WebAPI → Application → Infrastructure (via interface)
```

✅ Không có vi phạm nếu tuân thủ pattern hiện tại.

### 2.4 RejectionReason — OrderAuditLog vs OrderItem

> **Câu hỏi thiết kế**: `RejectionReason` nên lưu ở `OrderAuditLog.ChangeReason` hay `OrderItem.RejectionReason`?

|                          | `OrderAuditLog.ChangeReason` | `OrderItem.RejectionReason` |
| ------------------------ | ---------------------------- | --------------------------- |
| Audit trail              | ✅ Source of truth           | ❌ Không phải mục đích      |
| Hiển thị trên KDS        | ⚠️ Cần JOIN mỗi lần query    | ✅ Single query             |
| Lịch sử nhiều lần reject | ✅ Giữ hết                   | ❌ Bị overwrite             |

**Kết luận: Dùng cả hai, mỗi nơi một vai trò:**

```
OrderAuditLog.ChangeReason  → Source of truth cho AUDIT (lịch sử đầy đủ)
OrderItem.RejectionReason   → Denormalized field cho DISPLAY hiện tại
```

**Rule rõ ràng:**

```csharp
// Khi REJECT:
item.RejectionReason = reason;   // display field — đọc nhanh trên KDS list
auditLog.ChangeReason = reason;  // audit field — lịch sử đầy đủ

// Khi RETURN (REJECTED → PREPARING):
item.RejectionReason = null;     // item đã quay lại queue
// auditLog giữ nguyên lịch sử reject cũ
```

> **Quyết định phụ thuộc UX**: Nếu KDS list view cần show reason → cần denormalized field. Nếu chỉ show ở detail popup → `OrderAuditLog` là đủ, không cần field thêm.

---

## BƯỚC 3 — Đề xuất Kiến trúc Triển khai

### 3.1 Domain Changes Cần thiết

**[MODIFY] `OrderItem` entity — Thêm KDS domain methods:**

```csharp
// Nếu UX cần display reason → thêm field này
public string? RejectionReason { get; set; }
public DateTime? RejectedAt { get; set; }

public DomainResult StartCooking()
{
    if (Status != OrderItemStatus.Preparing)
        return DomainResult.Failure(DomainErrors.OrderItem.MustBePreparingToStartCooking);
    Status = OrderItemStatus.Cooking;
    UpdatedAt = DateTime.UtcNow;
    return DomainResult.Success();
}

public DomainResult MarkReady()
{
    if (Status != OrderItemStatus.Cooking)
        return DomainResult.Failure(DomainErrors.OrderItem.MustBeCookingToMarkReady);
    Status = OrderItemStatus.Ready;
    UpdatedAt = DateTime.UtcNow;
    return DomainResult.Success();
}

public DomainResult Reject(string reason)
{
    if (Status != OrderItemStatus.Cooking)
        return DomainResult.Failure(DomainErrors.OrderItem.MustBeCookingToReject);
    if (string.IsNullOrWhiteSpace(reason))
        return DomainResult.Failure(DomainErrors.OrderItem.RejectionReasonRequired);
    Status = OrderItemStatus.Rejected;
    RejectionReason = reason;  // display field (nếu cần)
    RejectedAt = DateTime.UtcNow;
    UpdatedAt = DateTime.UtcNow;
    return DomainResult.Success();
}

public DomainResult ReturnToQueue()
{
    if (Status != OrderItemStatus.Rejected)
        return DomainResult.Failure(DomainErrors.OrderItem.MustBeRejectedToReturn);
    Status = OrderItemStatus.Preparing;
    RejectionReason = null;
    UpdatedAt = DateTime.UtcNow;
    return DomainResult.Success();
}
```

### 3.2 Cấu trúc Folder Application

```
Features/KDS/
  ├── Commands/
  │   ├── StartCooking/
  │   │   ├── StartCookingCommand.cs
  │   │   ├── StartCookingCommandValidator.cs
  │   │   └── StartCookingHandler.cs
  │   ├── MarkReady/
  │   │   ├── MarkReadyCommand.cs
  │   │   ├── MarkReadyCommandValidator.cs
  │   │   └── MarkReadyHandler.cs
  │   ├── RejectOrderItem/
  │   │   ├── RejectOrderItemCommand.cs
  │   │   ├── RejectOrderItemCommandValidator.cs
  │   │   └── RejectOrderItemHandler.cs
  │   └── ReturnOrderItem/
  │       ├── ReturnOrderItemCommand.cs
  │       ├── ReturnOrderItemCommandValidator.cs
  │       └── ReturnOrderItemHandler.cs
  ├── Queries/
  │   ├── GetKdsItems/
  │   │   ├── GetKdsItemsQuery.cs
  │   │   ├── GetKdsItemsHandler.cs
  │   │   └── KdsItemResponse.cs
  │   └── GetKdsQueue/
  │       ├── GetKdsQueueQuery.cs
  │       ├── GetKdsQueueHandler.cs
  │       └── KdsQueueResponse.cs
  └── Common/
      └── KdsPriorityCalculator.cs
```

### 3.3 Handler Pipeline (theo FFA-FLW)

**StartCookingHandler** (ví dụ đầy đủ pipeline):

```
[R1] FluentValidation (orderItemId not empty)
[R2] Log.Information("StartCooking started — ItemId: {id}")
[R3] Load currentUser → verify role (Chef/Bartender/Manager)
[R4] BeginTransactionAsync
[R5a] Load OrderItem + verify station ownership
[R5b] WIP limit check: COUNT(COOKING) theo station < threshold
[R5c] orderItem.StartCooking()  ← Domain method
[R5d] Add OrderAuditLog (action: PREPARING→COOKING)
[R6] SaveChangeAsync
[R7] CommitTransactionAsync
[R8] ISignalRService.NotifyKdsUpdate()  ← NGOÀI transaction
[R9] Log.Information("StartCooking completed") + Return result
```

### 3.4 Transaction Boundary (theo FFA-TXG)

| Handler                  | Write count              | Transaction?  |
| ------------------------ | ------------------------ | ------------- |
| `StartCookingHandler`    | 2 (OrderItem + AuditLog) | ✅ Required   |
| `MarkReadyHandler`       | 2 (OrderItem + AuditLog) | ✅ Required   |
| `RejectOrderItemHandler` | 2 (OrderItem + AuditLog) | ✅ Required   |
| `ReturnOrderItemHandler` | 2 (OrderItem + AuditLog) | ✅ Required   |
| `GetKdsItemsHandler`     | 0 (read only)            | ❌ Not needed |

> ⚠️ **SignalR call PHẢI nằm NGOÀI transaction** (theo FFA-TXG TX-5)

### 3.5 Exception Strategy (theo FFA-ERR)

| Scenario                                   | Exception Type        | HTTP |
| ------------------------------------------ | --------------------- | ---- |
| OrderItem không tồn tại                    | `NotFoundException`   | 404  |
| Sai state transition                       | `BusinessException`   | 400  |
| WIP limit đã đầy                           | `BusinessException`   | 400  |
| Sai station (Chef Kitchen reject Bar item) | `ForbiddenException`  | 403  |
| Thiếu rejection reason                     | `ValidationException` | 400  |
| Role không đủ (non-Manager return)         | `ForbiddenException`  | 403  |

### 3.6 Priority Scoring

```csharp
// KdsPriorityCalculator.cs — Application/Features/KDS/Common/
public class KdsPriorityCalculator
{
    public int Calculate(OrderItem item, Order order)
    {
        var waitMinutes = (DateTime.UtcNow - item.CreatedAt).TotalMinutes;
        var score = 0;
        score += (int)(waitMinutes * 2);    // Thời gian chờ
        score += order.IsPriority ? 50 : 0; // VIP / đặt trước
        score += item.Quantity * 3;         // Độ phức tạp (Qty)
        return score;
    }
}
// Nếu score bằng nhau → FIFO theo CreatedAt (BR-KDS)
```

### 3.7 Test Strategy (theo FFA-TST)

```
FoodHub.Tests/Features/KDS/
  ├── StartCookingHandlerTests.cs
  ├── MarkReadyHandlerTests.cs
  ├── RejectOrderItemHandlerTests.cs
  ├── ReturnOrderItemHandlerTests.cs
  └── GetKdsItemsHandlerTests.cs
```

**Test Matrix:**

| Handler                  | Happy Path                        | Error Paths                                      |
| ------------------------ | --------------------------------- | ------------------------------------------------ |
| `StartCookingHandler`    | PREPARING → COOKING (WIP < limit) | NotFound, WIP full, wrong station, invalid state |
| `MarkReadyHandler`       | COOKING → READY                   | NotFound, not COOKING                            |
| `RejectOrderItemHandler` | COOKING → REJECTED với reason     | NotFound, empty reason, wrong role, not COOKING  |
| `ReturnOrderItemHandler` | REJECTED → PREPARING (Manager)    | NotFound, non-Manager, not REJECTED              |

**Domain Unit Tests:**

- `OrderItem.StartCooking()` — valid / invalid state
- `OrderItem.Reject(reason)` — with / without reason
- `OrderItem.ReturnToQueue()` — valid / invalid state

---

## BƯỚC 4 — Đánh giá Rủi ro

### 🔴 Critical Risk

| Risk                                          | Giải pháp                                                          |
| --------------------------------------------- | ------------------------------------------------------------------ |
| `OrderItem` thiếu domain methods KDS-specific | Tạo `StartCooking()`, `MarkReady()`, `Reject()`, `ReturnToQueue()` |
| SignalR call bị bọc vào transaction → DB lock | Enforce: SignalR NGOÀI `CommitTransactionAsync`                    |

### 🟠 Architectural Risk

| Risk                                          | Giải pháp                                                          |
| --------------------------------------------- | ------------------------------------------------------------------ |
| WIP limit hardcode "4"                        | Dùng `IKdsConfigService` hoặc constant có tên rõ ràng              |
| Auto-pull: inline Handler hay Background Job? | Tách rõ: Handler = manual; Background = auto-pull trigger          |
| Station ownership check thiếu                 | Handler phải check `item.StationSnapshot` vs `currentUser.Station` |

### 🟡 Maintainability Risk

| Risk                                    | Giải pháp                                                           |
| --------------------------------------- | ------------------------------------------------------------------- |
| Priority formula thay đổi theo business | Tách vào `KdsPriorityCalculator` — dễ test và thay đổi              |
| Audit log format không chuẩn            | Định nghĩa `KdsAuditPayload` DTO để serialize `OldValue`/`NewValue` |

### 🔵 Performance Risk

| Risk                                  | Giải pháp                                                  |
| ------------------------------------- | ---------------------------------------------------------- |
| WIP limit query N+1                   | Single `COUNT WHERE Station=X AND Status=Cooking`          |
| Priority sort in-memory khi queue lớn | Composite DB index: `(StationSnapshot, Status, CreatedAt)` |
| SignalR broadcast all                 | Group-based SignalR theo station                           |

---

## BƯỚC 5 — Kết luận

### ✅ Verdict

> **Triển khai được** — nhưng cần hoàn thiện Domain trước khi implement Application layer.

### Pre-Implementation Checklist

- [ ] **[CRITICAL]** Domain: Thêm `StartCooking()`, `MarkReady()`, `Reject(reason)`, `ReturnToQueue()` vào `OrderItem`
- [ ] **[CRITICAL]** Quyết định UX về `RejectionReason`: cần hiển thị trong list hay chỉ detail? → quyết định có thêm field vào `OrderItem` hay không
- [ ] **[HIGH]** Application: Tạo `ISignalRService` interface tại `Application/Interfaces/`
- [ ] **[HIGH]** Domain: Thêm `DomainErrors` cho KDS transitions
- [ ] **[MEDIUM]** Config: WIP limit là hardcode hay `IKdsConfigService`?
- [ ] **[MEDIUM]** Auto-pull: inline trong Command hay `IHostedService` riêng?

### Recommended Implementation Order

```
Phase 1 — Domain
  1. Thêm domain methods vào OrderItem
  2. (Nếu cần) EF Core migration thêm RejectionReason + RejectedAt

Phase 2 — Application Commands (priority order)
  3. RejectOrderItemCommand → Handler → Tests
  4. MarkReadyCommand → Handler → Tests
  5. StartCookingCommand → Handler (WIP check + auto-pull) → Tests
  6. ReturnOrderItemCommand → Handler → Tests

Phase 3 — Application Queries
  7. GetKdsItemsQuery → Handler (priority sorted)
  8. GetKdsQueueQuery → Handler

Phase 4 — WebAPI
  9. KdsController (thin — chỉ delegate Mediator)

Phase 5 — Realtime
  10. ISignalRService + Infrastructure implementation + integrate vào Handlers
```
