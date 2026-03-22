# ISSUE 82 - US-INV-06: Detailed Architect Implementation Plan

## 1. Muc tieu cua plan

Plan nay mo ta cach hoan thanh `US-INV-06` trong [ISSUE_82.md](h:\quan_li_nha_hang\FoodHub_Docs\Issues\ISSUE_82.md) theo dung nghiep vu da thong nhat:

- Gia von xuat kho su dung phuong phap `Weighted Average`.
- He thong da co co che cap nhat `Ingredient.CostPrice` moi khi nhap kho, vi vay ISSUE 82 khong lam lai engine tinh gia von co ban.
- ISSUE 82 can bo sung:
  - man hinh va command `recalculate COGS theo ky`;
  - cap nhat lai thanh tien phieu xuat trong ky;
  - inventory alerts cho `low stock`, `out of stock`, `near expiry`, `expired`;
  - badge tong canh bao;
  - batch/lot management de quan ly han su dung chat che va xuat theo FEFO.

Muc tieu cuoi cung la dat duoc dung nghiep vu trong issue, an toan transaction, khong roi logic vao Controller/Handler, va co test du de merge an toan.

## 2. Nguyen tac thiet ke bat buoc theo skill

Plan nay phai bam sat cac skill/guard sau:

- `aspnet-core`: ton trong cau truc ASP.NET Core hien co, su dung DI, middleware, controller/API theo convention cua solution.
- `ffa-cag`: Handler chi orchestration, logic doi state va tinh toan nam trong Domain service/Entity, Domain khong phu thuoc Application/Infrastructure.
- `ffa-ctl`: Controller mong, inject `ISender`/`IMediator`, khong co business if/else, response di qua `HandleResult`.
- `ffa-err`: dung `BusinessException`, `NotFoundException`, `ForbiddenException`; khong throw raw `Exception` cho nghiep vu.
- `ffa-flw`: pipeline Command phai co validation -> log start -> auth -> begin transaction -> business logic -> save -> commit -> mapping -> return.
- `ffa-log`: log start/end, structured logging, warning/error o business fail, khong log secret/token/PII.
- `ffa-txg`: multi-write bat buoc co transaction begin/commit/rollback, external call khong nam trong transaction.
- `ffa-acv`: DTO/Validator ro rang, required fields co validator, string co max length, dung dung type.
- `ffa-tst`: moi Command/Query/Domain logic quan trong phai co test happy path, error path, edge case.

## 3. Refine scope cua ISSUE 82

### 3.1. Scope nam trong ISSUE 82

1. COGS weighted average theo nghiep vu issue:

- hien thi trang `/inventory/cogs`;
- cho phep Manager chon tu ngay - den ngay toi da 31 ngay;
- filter theo 1 NVL hoac tat ca;
- recalculate gia von cho cac phieu xuat trong ky;
- cap nhat thanh tien tren phieu xuat trong ky;
- toast thanh cong va log audit.

2. Inventory alerts:

- low stock;
- out of stock;
- near expiry;
- expired;
- badge tong o sidebar;
- trang `/inventory/alerts`.

3. Batch/Lot management de phuc vu han su dung:

- tao lot khi nhap kho;
- ton kho theo lot;
- uu tien xuat theo FEFO;
- chan xuat lot het han;
- theo doi lot sap het han/da het han;
- dispose/adjust lot het han.

### 3.2. Scope khong nen gom vao ISSUE 82

- khong doi phuong phap gia von sang FIFO/LIFO;
- khong viet lai toan bo inventory ledger;
- khong thay doi nghiep vu recipe/menu cost ngoai pham vi can de doc gia von moi;
- khong them external integration;
- khong refactor lon cac module inventory khac neu khong phuc vu truc tiep issue.

## 4. Hien trang code va tac dong den thiet ke

### 4.1. Hien trang da co

- `Ingredient.ReceiveStock()` da cap nhat `CostPrice` theo weighted average khi nhap kho.
- `Ingredient.ReverseReceivedStock()` da ho tro hoan nhap theo cost da luu.
- `StockInReceiptItem` da luu `ExpiryDate` va `BatchCode`.
- `InventorySettings` da co `ExpiryWarningDays` va `MaxCostRecalcDays`.
- `GetInventoryReport` da doc `ingredient.CostPrice` de hien thi `AverageUnitCost` va `ClosingStockValue`.

### 4.2. Khoang trong hien tai

- chua co use case recalculate COGS theo ky;
- chua co domain service rieng de replay movement va restate COGS;
- chua co entity `InventoryLot` de quan ly ton thuc te theo lo;
- chua co lot allocation khi xuat kho;
- chua co alert query tong hop low stock + expiry + badge;
- chua co policy chan xuat lot het han;
- chua co write-off/dispose cho lot het han;
- chua co test matrix cho lot + recalc COGS.

## 5. Dinh nghia nghiep vu can chot truoc khi code

### 5.1. COGS

- `Weighted Average` la nguon su that cho cost gia von.
- Moi lan co nhap kho moi thi `CostPrice` hien hanh thay doi.
- Khi xuat kho, gia von xuat phai bang average cost tai thoi diem xuat.
- Recalculate theo ky la nghiep vu `restate`:
  - replay giao dich trong ky;
  - cap nhat don gia xuat va thanh tien phieu xuat trong ky;
  - khong duoc lam sai ton kho.

### 5.2. Batch/Lot

- Moi dong nhap kho co expiry thi sinh ra 1 lot.
- Mot ingredient co the co nhieu lot dang ton tai song song.
- Xuat kho phai uu tien lot co `ExpiryDate` som nhat, neu cung ngay thi den `ReceivedAt` som nhat.
- Lot het han khong duoc su dung de xuat ban hang / xuat kho thuong.
- Lot het han co the duoc:
  - dispose;
  - inventory check adjustment;
  - reverse neu nghiep vu cho phep.

### 5.3. Alerts

- `Out of stock`: `CurrentStock = 0`.
- `Low stock`: `0 < CurrentStock <= LowStockThreshold`.
- `Near expiry`: lot co `ExpiryDate <= today + ExpiryWarningDays` va chua depleted/disposed.
- `Expired`: lot co `ExpiryDate < today` va chua depleted/disposed.
- Badge tong = tong so ingredient canh bao ton + tong so lot canh bao expiry.

## 6. Domain model can them/chinh sua

### 6.1. Entity moi

#### `InventoryLot`

Muc dich: dai dien ton thuc te theo lo.

Field de xuat:

- `InventoryLotId : Guid`
- `IngredientId : Guid`
- `StockInReceiptItemId : Guid?`
- `LotCode : string`
- `ReceivedAt : DateTime`
- `ExpiryDate : DateTime?`
- `UnitCost : decimal`
- `OriginalQuantity : decimal`
- `RemainingQuantity : decimal`
- `ReservedQuantity : decimal` (optional, de room mo rong)
- `Status : InventoryLotStatus`
- `Notes : string?`
- audit fields

Behavior de xuat:

- `Create(...)`
- `Consume(quantity, occurredAt, updatedBy)`
- `ReverseConsume(quantity, occurredAt, updatedBy)`
- `MarkExpired(currentDate, updatedBy)`
- `MarkDisposed(quantity, reason, updatedBy)`
- `AdjustQuantity(delta, reason, updatedBy)`
- `CanConsume(onDate)`
- `GetAvailableQuantity(onDate)`

#### `InventoryLotMovement`

Muc dich: audit trail cho thay doi ton theo lo.

Field de xuat:

- `InventoryLotMovementId : Guid`
- `InventoryLotId : Guid`
- `TransactionType : InventoryLotTransactionType`
- `QuantityDelta : decimal`
- `BalanceAfter : decimal`
- `ReferenceType : InventoryReferenceType`
- `ReferenceId : Guid?`
- `ReferenceCode : string?`
- `OccurredAt : DateTime`
- `UnitCost : decimal?`
- `Note : string?`
- audit fields

### 6.2. Enum moi

- `InventoryLotStatus`
  - `Active`
  - `NearExpiry`
  - `Expired`
  - `Depleted`
  - `Disposed`

- `InventoryLotTransactionType`
  - `StockIn`
  - `StockInReverse`
  - `StockOut`
  - `StockOutReverse`
  - `SaleDeduction`
  - `SaleDeductionReverse`
  - `InventoryCheck`
  - `Dispose`
  - `Adjustment`

### 6.3. Domain service moi

#### `InventoryCostService`

Trach nhiem:

- replay movement theo ingredient;
- tinh average cost tai moi diem xuat;
- restate `StockOutReceiptItem` / lot allocation cost neu can;
- tra ve ket qua recalc de handler save.

Khong nen:

- query DB truc tiep;
- log truc tiep;
- tu commit transaction.

#### `InventoryLotAllocationService`

Trach nhiem:

- tim lot co the xuat;
- sap xep theo FEFO;
- chia quantity can xuat qua nhieu lot;
- chan lot het han;
- tra ve allocation plan.

#### `InventoryAlertService`

Trach nhiem:

- tong hop low stock/out of stock;
- tong hop near expiry/expired;
- tinh badge count;
- format model trung gian cho query layer.

## 7. Persistence va migration plan

### 7.1. Bang moi

1. `inventory_lots`

- PK `inventory_lot_id`
- FK `ingredient_id`
- FK `stock_in_receipt_item_id` nullable
- unique index de xet `ingredient_id + lot_code` neu nghiep vu yeu cau lot code unique trong 1 NVL
- index `expiry_date`
- index `(ingredient_id, status, expiry_date)`
- index `(ingredient_id, remaining_quantity)`

2. `inventory_lot_movements`

- PK `inventory_lot_movement_id`
- FK `inventory_lot_id`
- index `(inventory_lot_id, occurred_at)`
- index `(reference_id, reference_type)`

3. `stock_out_receipt_item_lots` hoac `inventory_outbound_allocations`

Muc dich:

- luu phieu xuat da tru vao lot nao;
- giup reverse chinh xac;
- giup audit/trace.

Field de xuat:

- `Id`
- `StockOutReceiptItemId`
- `InventoryLotId`
- `Quantity`
- `UnitCost`
- `LineCost`
- `OccurredAt`

### 7.2. Chinh sua bang hien co

- `stock_out_receipt_items`:
  - xem xet doi ten `UnitPrice` neu hien dang dang dung cho cost noi bo; neu khong doi ten duoc thi can document ro la field nay dang duoc dung lam don gia xuat/gia von.
  - them field `CalculatedUnitCost`
  - them field `CalculatedLineCost`
  - them field `CostCalculatedAt`
  - them field `CostCalculationSource` (`Realtime`, `PeriodRecalculation`)

- `inventory_transactions`:
  - giu nguyen de lam ledger tong cap ingredient;
  - tiep tuc luu `UnitCost`, nhung lot-level detail se nam o bang allocation/movement moi.

### 7.3. Backfill du lieu cu

Bat buoc vi issue yeu cau near expiry theo lot:

- Backfill `InventoryLot` tu `StockInReceiptItem` da ton tai:
  - moi `StockInReceiptItem` -> 1 lot;
  - `OriginalQuantity = item.Quantity`;
  - `RemainingQuantity` can tinh dua tren giao dich xuat da co.

Hai cach:

1. Cach an toan de lam trong sprint:

- backfill lot cho du lieu moi;
- du lieu cu tao lot voi `RemainingQuantity` bang `max(0, so luong ton uoc tinh)` theo heuristic;
- chi bat FEFO cứng voi giao dich moi.

2. Cach chuan hon:

- replay tat ca stock in/stock out/sale deduction/reverse tu dau de tinh `RemainingQuantity` chuan cho tung lot.

De xuat:

- neu data san xuat chua nhieu, chon cach 2.
- neu deadline gap, chon cach 1 + ghi chu migration risk.

## 8. Command, Query, API va UI plan

### 8.1. Command/Query cho COGS

#### `RecalculateCogsCommand`

Input:

- `FromDate`
- `ToDate`
- `IngredientId?`

Validator:

- `FromDate <= ToDate`
- period <= `MaxCostRecalcDays`
- yeu cau Manager

Handler flow:

1. log start
2. load settings
3. validate auth + max days
4. load ingredient filter
5. begin transaction
6. lay movement theo thu tu thoi gian
7. dung `InventoryCostService` de recalc
8. update `StockOutReceiptItem` / allocation cost
9. save
10. commit
11. clear cache lien quan
12. log success
13. return summary

Response:

- tong so ingredient xu ly
- tong so phieu xuat cap nhat
- tong so dong xuat cap nhat
- from/to
- message

#### `GetInventoryCogsPreviewQuery` (optional nhung nen co)

Muc dich:

- preview so dong se cap nhat truoc khi bam recalc.

### 8.2. Query cho Alerts

#### `GetInventoryAlertsQuery`

Tra ve:

- `LowStockItems`
- `OutOfStockItems`
- `NearExpiryLots`
- `ExpiredLots`
- `BadgeCount`

#### `GetInventoryAlertBadgeQuery`

Muc dich:

- sidebar badge nhe va de cache rieng.

### 8.3. Command/Query cho lot

#### `GetInventoryLotsQuery`

- loc theo ingredient, status, expiry range
- phuc vu UI xem ton theo lo

#### `GetInventoryLotByIdQuery`

- chi tiet lot + movement history

#### `DisposeInventoryLotCommand`

- dung cho lo het han/hu hong

#### `AdjustInventoryLotCommand`

- dung cho inventory check / hao hut / bo sung dieu chinh

### 8.4. Controllers theo `ffa-ctl`

De xuat tach theo controller hien co thay vi tao 1 controller lon:

- `InventoryReportsController`
  - `POST /inventory/cogs/recalculate`
  - `GET /inventory/cogs/preview` (optional)

- `InventoryAlertsController` (controller moi neu chua co)
  - `GET /inventory/alerts`
  - `GET /inventory/alerts/badge`

- `InventoryLotsController` (controller moi)
  - `GET /inventory/lots`
  - `GET /inventory/lots/{id}`
  - `POST /inventory/lots/{id}/dispose`
  - `POST /inventory/lots/{id}/adjust`

Controller chi:

- nhan DTO;
- gui `ISender.Send`;
- `return HandleResult(result);`

### 8.5. UI/UX can co

#### `/inventory/cogs`

- form chon `FromDate`, `ToDate`, `IngredientId | All`
- validate 31 ngay ngay tai client
- nut `Tinh gia`
- bang ket qua sau khi xong:
  - so phieu xuat duoc cap nhat
  - so dong duoc cap nhat
  - tong gia tri thay doi

#### `/inventory/alerts`

- tab 1: `Sap het / Het hang`
- tab 2: `Sap het han / Da het han`
- sort:
  - out of stock tren cung
  - low stock theo stock asc
  - expired tren cung
  - near expiry theo so ngay con lai asc

#### Sidebar badge

- goi endpoint badge rieng
- cache ngan
- invalidate sau stock in/out/reverse/dispose/adjust

#### UI lot details

- trong detail ingredient hoac stock in receipt can co link xem lot
- lot detail hien:
  - lot code
  - expiry
  - original quantity
  - remaining quantity
  - status
  - movement history

## 9. Rule nghiep vu chi tiet can implement

### 9.1. Rule cho weighted average

- opening stock co `CostPrice` -> duoc xem la opening value.
- stock in co `UnitCost` -> cap nhat average cost.
- stock out/sale deduction khong doi average cost, chi tieu ton.
- stock in reverse phai hoan lai average cost hop le.
- neu chua tung nhap va khong co opening cost -> don gia = 0.

### 9.2. Rule cho recalculate theo ky

- ky toi da `31 ngay`.
- sort replay theo:
  - `OccurredAt`
  - tie-break `CreatedAt`
  - tie-break `Id` neu can
- chi Manager/Cac role co `Permissions.Inventory.Update` moi duoc recalc.
- recalc chi cap nhat cost fields, khong tao ton kho moi.
- khong goi external service trong transaction.

### 9.3. Rule cho lot va expiry

- lot co `ExpiryDate = null`:
  - khong nam trong near expiry/expired;
  - dung sau cac lot co expiry khi FEFO, hoac can mot rule ro rang trong code.

De xuat:

- FEFO sap xep:
  1. lot co expiry som nhat
  2. neu cung expiry -> `ReceivedAt` som nhat
  3. lot khong expiry xep cuoi

- lot `Expired`:
  - khong duoc xuat
  - van xuat hien trong alerts
  - can dispose hoac adjust

- lot `Depleted`:
  - khong xuat hien trong alerts expiry
  - van giu trong history

- lot `Disposed`:
  - khong duoc reverse xuat thuong
  - can luu reason/audit

### 9.4. Rule cho reverse

- reverse stock in:
  - chi cho phep neu movement cua lot/phieu nhap la latest hop le hoac co co che reverse an toan;
  - phai rollback lot quantity va lot movement;
  - neu lot da bi consume/dispose mot phan thi can chan hoac buoc reverse phuc tap hon.

De xuat sprint nay:

- giu rule hien co: chi reverse stock in neu la latest movement cua ingredient va lot chua bi consume.

- reverse stock out:
  - phai hoan quantity vao dung lot da bi tru;
  - dung bang allocation de biet hoan vao lot nao.

### 9.5. Rule cho inventory check

- inventory check future-proof:
  - phase 1 co the van check cap ingredient;
  - phase 2 nen mo rong check cap lot cho nguyen lieu co expiry.

Neu khong lam check cap lot ngay:

- phai ghi ro la inventory check se adjust tong stock, sau do can command phu de phan bo lai vao cac lot.

## 10. Validation va exception map theo `ffa-acv` + `ffa-err`

### 10.1. Validation

- `RecalculateCogsCommandValidator`
  - `FromDate` required
  - `ToDate` required
  - `ToDate >= FromDate`
  - `period <= MaxCostRecalcDays`

- `DisposeInventoryLotCommandValidator`
  - `LotId` required
  - `Quantity > 0`
  - `Reason` max length

- `AdjustInventoryLotCommandValidator`
  - `LotId` required
  - `Delta != 0`
  - `Reason` max length

### 10.2. Exception cases

- ingredient khong ton tai -> `NotFoundException`
- lot khong ton tai -> `NotFoundException`
- user khong du quyen -> `ForbiddenException`
- period > 31 ngay -> `BusinessException`
- lot het han ma van xuat -> `BusinessException`
- khong du ton theo lot -> `BusinessException`
- lot code duplicate -> `BusinessException`
- reverse khong hop le -> `BusinessException`

## 11. Transaction design theo `ffa-txg`

### 11.1. Command bat buoc transaction

- `CreateStockInReceipt`
- `ReverseStockInReceipt`
- `CreateStockOutReceipt`
- `ReverseStockOutReceipt`
- `RecalculateCogsCommand`
- `DisposeInventoryLotCommand`
- `AdjustInventoryLotCommand`
- `ProcessInventoryCheck` neu co dong vao lot

### 11.2. Transaction boundary

#### `CreateStockInReceipt`

- write receipt
- write receipt items
- update ingredient stock/cost
- create inventory transaction
- create inventory lot
- create inventory lot movement
- commit
- sau commit moi invalidate cache / sync availability

#### `CreateStockOutReceipt`

- write receipt
- write receipt items
- allocate lots qua `InventoryLotAllocationService`
- update remaining quantity tung lot
- write lot movements
- write outbound allocation records
- update ingredient stock
- write inventory transaction
- commit

#### `RecalculateCogsCommand`

- load range va target items
- replay cost
- update cost fields item xuat
- update allocation `UnitCost`, `LineCost` neu dung lot allocation
- save once
- commit

## 12. Logging plan theo `ffa-log`

Moi handler/query quan trong can co:

- `LogInformation` start:
  - handler name
  - from/to
  - ingredientId/all
  - lotId neu co

- `LogInformation` success:
  - so ban ghi xu ly
  - so lot canh bao
  - so phieu xuat cap nhat

- `LogWarning`:
  - lot khong du ton
  - lot het han
  - skip ingredient do data khong hop le

- `LogError`:
  - rollback transaction
  - exception khong phai business flow expected

Khong log:

- token
- password
- raw payload co PII neu khong can

## 13. Test strategy theo `ffa-tst`

### 13.1. Domain tests

#### `InventoryCostServiceTests`

- tinh average cost voi opening + stock in
- xuat kho dung average cost tai thoi diem xuat
- recalc voi nhieu stock in/out xen ke
- chua co stock in -> cost = 0
- reverse stock in lam average cost thay doi dung

#### `InventoryLotTests`

- tao lot thanh cong
- consume lot giam remaining quantity
- khong consume qua available
- mark expired / depleted / disposed dung status

#### `InventoryLotAllocationServiceTests`

- FEFO chon lot gan het han nhat
- skip lot expired
- lot null expiry xep sau cung
- xuat qua nhieu lot thi split dung
- khong du ton lot -> throw business rule

### 13.2. Application handler tests

#### `RecalculateCogsCommandHandlerTests`

- happy path all ingredients
- happy path 1 ingredient
- period > 31 ngay -> throw
- ingredient khong ton tai -> throw
- khong co giao dich trong ky -> success voi 0 update
- cache invalidation duoc goi
- transaction begin/commit/rollback dung flow

#### `GetInventoryAlertsQueryHandlerTests`

- out of stock len tren
- low stock hien thi dung
- near expiry trong 7 ngay hien thi dung
- expired hien thi dung
- badge = low stock + expiry lots

#### `CreateStockInReceiptHandlerTests`

- tao lot sau khi nhap kho
- lot movement duoc tao
- lot code trim/max length hop le

#### `CreateStockOutReceiptHandlerTests`

- xuat kho tao allocation theo FEFO
- lot expired bi chan
- reverse xuat kho hoan ve dung lot

### 13.3. Integration/API tests neu co kha nang

- endpoint `/inventory/cogs/recalculate`
- endpoint `/inventory/alerts`
- endpoint `/inventory/alerts/badge`
- endpoint `/inventory/lots/{id}/dispose`
- phan quyen Waiter -> 403

## 14. Acceptance Criteria map -> implementation tasks

### AC-01 Weighted Average

- xac nhan `Ingredient.ReceiveStock()` va `InventoryCostService` thong nhat cong thuc.
- test domain voi data issue.

### AC-02 Recalculate theo ky

- tao `RecalculateCogsCommand`
- update cost fields tren phieu xuat
- toast thanh cong tai UI

### AC-03 Gioi han 31 ngay

- validator + settings check
- test error path + message

### AC-04 / AC-05 Low stock va out of stock

- query low/out stock
- sort out of stock len dau

### AC-06 Near expiry

- can `InventoryLot`
- query near expiry theo `ExpiryWarningDays`

### AC-07 Badge

- aggregate low stock + expiry lots
- endpoint badge + cache

### AC-08 Authorization

- Manager/Cashier/role hop le moi duoc vao inventory update features
- Waiter bi 403 hoac read-only o FE tuy route

## 15. Delivery roadmap theo phase

### Phase 1 - Nen tang COGS + alert co ban

Deliverables:

- `RecalculateCogsCommand` + handler + validator
- cost service co kha nang replay/restate
- endpoint recalc
- query alerts cho low stock/out of stock
- query badge co ban
- tests cho recalc + low stock

Definition of Done:

- AC-01, AC-02, AC-03, AC-04, AC-05 pass

### Phase 2 - Batch/Lot + expiry control

Deliverables:

- migration bang lot + movement + allocation
- create lot khi stock in
- FEFO allocation khi stock out
- query near expiry/expired
- chan xuat lot expired
- UI alerts expiry
- tests lot/allocation/expiry

Definition of Done:

- AC-06, AC-07 pass o nghiep vu expiry

### Phase 3 - Reverse / dispose / hardening

Deliverables:

- reverse xuat kho theo lot
- dispose/adjust lot
- audit/report lot detail
- backfill/migration hardening
- integration tests, perf tuning, indexes

Definition of Done:

- issue san sang production, co path xu ly lot expired va reverse an toan

## 16. File structure de xuat

### WebAPI

- `FoodHub_BE/FoodHub.WebAPI/Presentation/Controllers/Inventory/InventoryAlertsController.cs`
- `FoodHub_BE/FoodHub.WebAPI/Presentation/Controllers/Inventory/InventoryLotsController.cs`
- mo rong `InventoryReportsController.cs` cho COGS recalc

### Application

- `FoodHub_BE/FoodHub.Application/Features/Inventory/Costing/Commands/RecalculateCogs/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/Costing/Queries/GetInventoryCogsPreview/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/Alerts/Queries/GetInventoryAlerts/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/Alerts/Queries/GetInventoryAlertBadge/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/Lots/Queries/GetInventoryLots/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/Lots/Queries/GetInventoryLotById/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/Lots/Commands/DisposeInventoryLot/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/Lots/Commands/AdjustInventoryLot/...`
- `FoodHub_BE/FoodHub.Application/Features/Inventory/DTOs/...`

### Domain

- `FoodHub_BE/FoodHub.Domain/Entities/InventoryLot.cs`
- `FoodHub_BE/FoodHub.Domain/Entities/InventoryLotMovement.cs`
- `FoodHub_BE/FoodHub.Domain/Enums/InventoryLotStatus.cs`
- `FoodHub_BE/FoodHub.Domain/Enums/InventoryLotTransactionType.cs`
- `FoodHub_BE/FoodHub.Domain/Services/InventoryCostService.cs`
- `FoodHub_BE/FoodHub.Domain/Services/InventoryLotAllocationService.cs`
- `FoodHub_BE/FoodHub.Domain/Services/InventoryAlertService.cs`

### Infrastructure

- configurations cho lot/movement/allocation
- migration tao bang
- repository query toi uu neu can

### Tests

- `FoodHub_BE/FoodHub.Tests/Features/Inventory/RecalculateCogsCommandHandlerTests.cs`
- `FoodHub_BE/FoodHub.Tests/Features/Inventory/GetInventoryAlertsQueryHandlerTests.cs`
- `FoodHub_BE/FoodHub.Tests/Features/Inventory/CreateStockOutReceiptLotAllocationTests.cs`
- `FoodHub_BE/FoodHub.Tests/Features/Inventory/DisposeInventoryLotCommandHandlerTests.cs`
- `FoodHub_BE/FoodHub.Tests/Features/Inventory/InventoryLotEntityTests.cs`
- `FoodHub_BE/FoodHub.Tests/Features/Inventory/InventoryLotAllocationServiceTests.cs`
- `FoodHub_BE/FoodHub.Tests/Features/Inventory/InventoryCostServiceTests.cs`

## 17. Rủi ro va cach giam thieu

### Rui ro 1 - Doi y nghia field cost tren phieu xuat

- van de: `StockOutReceiptItem.UnitPrice` co the dang bi dung lam gia von noi bo.
- giam thieu: khong doi ten field trong sprint neu rui ro cao; them field cost moi va migrate ro rang.

### Rui ro 2 - Backfill lot cho du lieu cu sai lech

- van de: data cu co the khong du de phan bo ton ve tung lot chuan 100%.
- giam thieu: viet script replay movement; ghi nhan exceptions; cho phep manual adjustment sau migration.

### Rui ro 3 - Reverse phuc tap hon hien tai

- van de: reverse stock in/out khi da co lot allocation phuc tap.
- giam thieu: phase rollout voi rule reverse chat hon, chan cac truong hop ambiguous.

### Rui ro 4 - Performance query alerts va recalc

- van de: replay transaction va query lots co the cham.
- giam thieu:
  - index theo `ingredient_id`, `occurred_at`, `expiry_date`, `status`;
  - cache badge/alerts ngan han;
  - recalc theo ingredient filter va period <= 31 ngay.

## 18. Definition of Done cho ISSUE 82

ISSUE 82 duoc xem la hoan thanh tot khi:

- AC-01 den AC-08 deu co implementation ro rang va test cover.
- Controller moi tuan thu thin controller theo `ffa-ctl`.
- Handler moi tuan thu flow va transaction theo `ffa-flw` + `ffa-txg`.
- Domain logic khong roi vao handler theo `ffa-cag`.
- Validation/exception dung theo `ffa-acv` + `ffa-err`.
- Log day du va an toan theo `ffa-log`.
- Test happy path, error path, edge case day du theo `ffa-tst`.
- Lot-based expiry control hoat dong du de:
  - xem lot sap het han / da het han;
  - chan xuat lot het han;
  - xuat theo FEFO;
  - trace lot da duoc xuat/hoan/dispose.

## 19. Khuyen nghi chot truoc khi implement

De tranh rework, can chot 4 quyet dinh nghiep vu sau truoc khi code:

1. `StockOutReceiptItem` se dung field cost nao:

- tai su dung `UnitPrice`;
- hay them `CalculatedUnitCost` / `CalculatedLineCost`.

2. Lot `ExpiryDate = null` co duoc xuat truoc hay xep cuoi trong FEFO.

3. Reverse stock in co tiep tuc bi gioi han "latest movement only" hay cho phep reverse theo lot allocation phuc tap.

4. Backfill du lieu cu chon replay full history hay heuristic migration.

Neu chot 4 diem nay som, implementation se di rat thang va giam manh rui ro doi schema giua sprint.
