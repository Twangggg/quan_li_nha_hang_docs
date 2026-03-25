# 📋 FoodHub API Requirements Spec - Manager Dashboard & Core Features

## Overview
This document specifies the backend (BE) API requirements based on the current frontend (FE) UI implementation. It aims to replace all mock/hardcoded data on the Manager Dashboard, KDS Widget, Inventory Alerting, and Audit Log screens.

---

## 1. Manager Dashboard KPI Stats
**Endpoint:** `GET /api/v1/salesanalytics/summary` (Enhanced)
**Purpose:** Powering the main 4 status cards on the dashboard.

### Request Parameters
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `date` | `DateOnly` | No | Target date (default: today) |
| `movingAverageDays` | `int` | No | Period to calculate growth/target trends (default: 30) |

### Response Schema (`data`)
```json
{
  "totalRevenue": 12500000.0,
  "revenueGrowth": 12.5,        // % trend compared to average
  "totalOrders": 48,
  "orderGrowth": 5.2,           // % trend compared to average (currently hardcoded in FE)
  "avgOrderValue": 260416.7,    // totalRevenue / totalOrders
  "cancelledOrders": 3,
  "revenueAchievement": 85.5    // % against dailyTarget
}
```

---

## 2. Real-time Operational Stats
**Endpoint:** `GET /api/v1/dashboard/operational-stats`
**Purpose:** Real-time monitoring of restaurant floor and staffing status.

### Response Schema (`data`)
```json
{
  "occupiedTables": 12,
  "totalTables": 20,
  "tableOccupancyRate": 60.0,
  "tableTrend": -2.4,           // % change in occupancy vs last hour
  "activeStaffCount": 6,        // Staff currently on status 'OnShift'
  "totalStaffOnShift": 8,       // Total staff assigned to current shift
  "staffTrend": 2               // Net staff change in last 30 mins
}
```

---

## 3. KDS Backlog Summary (Kitchen/Bar)
**Endpoint:** `GET /api/v1/kds/backlog-summary`
**Purpose:** Feeding the KDS Status Widget on the dashboard.

### Response Schema (`data`)
```json
{
  "totalProcessingItems": 23,
  "waitingCount": 15,           // Items in 'Pending' queue
  "preparingCount": 8,          // Items marked 'Preparing'
  "delayedCount": 3,            // Items exceeding preparation threshold (e.g. > 20 mins)
  "preparingPercentage": 34.8   // (preparingCount / totalProcessingItems) * 100
}
```

---

## 4. Audit Log (Advanced List & Filtering)
**Endpoint:** `GET /api/v1/auditlogs`
**Purpose:** Providing detailed change history with server-side filtering.

### Query Parameters
| Parameter | Type | Description |
| :--- | :--- | :--- |
| `pageNumber` | `int` | Current page (default: 1) |
| `pageSize` | `int` | Items per page (default: 15) |
| `actionFilter` | `string` | e.g. "Create", "Update", "Delete", "StatusChange" |
| `entityNameFilter` | `string` | e.g. "Table", "Reservation", "MenuItem", "Ingredient" |
| `entityIdFilter` | `string` | GUID or partial string for specific entity search |
| `fromDate` | `DateTime` | Start of audit window (ISO format) |
| `toDate` | `DateTime` | End of audit window (ISO format) |

### Response Schema (`PagedResult<GetAuditLogsResponse>`)
```json
{
  "items": [
    {
      "logId": "f47b-...",
      "createdAt": "2026-03-25T14:30:00Z",
      "action": "StatusChange",
      "entityName": "Table",
      "entityId": "T-01",
      "summary": "Changed status to Occupied", // Brief human-readable description
      "actorInfo": "Nguyễn Văn A (Manager)",  // Display name and role
      "oldValues": "{ \"status\": \"Available\" }",
      "newValues": "{ \"status\": \"Occupied\" }"
    }
  ],
  "totalCount": 125,
  "totalPages": 9
}
```

---

## 5. Promotion (Voucher) Quick Status Toggle
**Endpoint:** `PATCH /api/v1/promotions/{id}/status`
**Purpose:** Instant activation/deactivation of vouchers from the list view.

### Request Body / Query
*   `isActive`: `boolean`

### Response
*   `Result<bool>`: Returns `true` if operation succeeded.

---

## 🏗️ Technical Notes for Backend Implementation
1.  **Response Wrapper:** All responses must follow the `ApiControllerBase` logic, wrapping the payload in a `data` field: `{ "data": { ... } }`.
2.  **Permissions:** Each endpoint should be protected by appropriate permission checks (`HasPermission` attribute):
    *   `Permissions.SalesAnalytics.View`
    *   `Permissions.Employees.ViewAuditLogs`
    *   `Permissions.Vouchers.UpdateStatus`
3.  **Localization:** Human-readable fields like `summary` or `actorInfo` (sub-labels) should utilize the `IMessageService` or database-linked display names.
4.  **SignalR (Optional Recommendation):** For `operational-stats`, consider pushing updates via Hub when Table/Staff statuses change to avoid frequent polling.
