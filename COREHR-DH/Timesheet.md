# Timesheet - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `timesheets`
> - `Entity name`: `timesheet`

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/timesheets`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/timesheets({systemId})`

Power Automate notes:
- Use **Find records (V3)** for scheduled sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.

---

## 2) Data Model

### 2.1 Editable Fields
All exposed fields are **read-only** in this integration context.

- `PATCH`: Not supported for this contract.
- `POST`: Not part of this integration contract.

### 2.2 Timesheet API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| SystemId | SystemId | GUID | No | OData key |
| EmployeeCode | "Employee Code" | Code | No | Ажилтны код |
| Year | "Year" | Integer | No | Жил |
| Month | "Month" | Integer | No | Сар |
| CalculationType | "Calculation Type" | Text/Enum | No | Тооцооллын төрөл |
| WorkingHourTotal | "Working Hour-Total" | Decimal | No | Нийт ажиллах цаг |
| WorkingHourHalf | "Working Hour-Half" | Decimal | No | Хагас ажиллах цаг |
| WorkedHour | "Worked Hour" | Decimal | No | Ажилласан цаг |
| Overtime | "Overtime" | Decimal | No | Илүү цаг |
| HolidayOvertime | "Holiday Overtime" | Decimal | No | Баярын илүү цаг |
| PaternityLeave | "Paternity Leave" | Decimal | No | Эцгийн чөлөө |
| KPIScheduled | "KPI-Scheduled" | Decimal | No | KPI-Төлөвлөгөөт |
| KPIUnscheduled | "KPI-Unscheduled" | Decimal | No | KPI-Төлөвлөгөөт бус |
| Status | Status | Text/Enum | No | Статус |
| AnnualLeave | "Annual Leave" | Decimal | No | Ээлжийн амралт |
| LeaveUnpaid | "Leave-Unpaid" | Decimal | No | Цалингүй чөлөө |
| SickLeaveUnpaid | "Sick Leave-Unpaid" | Decimal | No | Цалингүй өвчний чөлөө |
| LateArrival | "Late Arrival" | Decimal | No | Хоцролт |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/timesheets?$select=SystemId,EmployeeCode,Year,Month,WorkedHour,Overtime,Status&$filter=EmployeeCode eq 'EMP001' and Year eq 2026&$top=1000

Response shape (example):
```json
{
  "value": [
    {
      "SystemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000501",
      "EmployeeCode": "EMP001",
      "Year": 2026,
      "Month": 3,
      "WorkedHour": 168.0,
      "Overtime": 12.5,
      "Status": "Approved"
    }
  ],
  "@odata.nextLink": "..."
}
```

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/timesheets({systemId})

---

## 4) Business Rules & Validations

1. Key handling
   - Use `SystemId` as immutable technical key.
   - Use `EmployeeCode` + `Year` + `Month` + `CalculationType` as composite business key for reconciliation.

2. Period validation
   - `Year` and `Month` define the timesheet period.
   - Validate that `Month` is between 1 and 12.
   - Do not assume records exist for every employee/month combination.

3. Hour calculations
   - `WorkedHour` should be ≤ `WorkingHourTotal` in normal cases.
   - `Overtime` and `HolidayOvertime` are tracked separately from regular hours.
   - `WorkingHourHalf` represents half-day working hour schedules.

4. Leave tracking
   - `AnnualLeave`, `LeaveUnpaid`, and `SickLeaveUnpaid` are period-level aggregates.
   - These values reflect BC-calculated totals, not user-editable fields.

5. Read-only API behavior
   - Do not send `PATCH`, `POST`, or `DELETE` to this endpoint.
   - Treat this as a source-of-truth read model from BC.

6. Pagination and filtering
   - Always process `@odata.nextLink` for full dataset retrieval.
   - Filter by `Year` and `Month` for period-specific sync.

---

## 5) Integration Flow (Power Automate)

Recommended flow pattern (read-only pull):

1. Trigger
   - Scheduled cloud flow (for example every 1 to 6 hours, or monthly after payroll close).

2. Find records from BC
   - Call `timesheets` with `$select`, `$filter` (by Year, Month), and `$top`.
   - Iterate pages using `@odata.nextLink`.

3. Transform and validate
   - Validate numeric fields and period values.
   - Convert decimal hours to target system format if needed.

4. Upsert to destination
   - Upsert by `SystemId`.
   - Optional business fallback key: `EmployeeCode + Year + Month + CalculationType`.

5. Error handling
   - Log request URL, status code, and key fields.
   - Retry transient errors with exponential backoff.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or editing timesheet records through this endpoint.
- Implementing timesheet approval workflows.
- Calculating overtime pay or salary from timesheet hours.
- Guaranteeing schema stability across future API versions.
- Managing attendance device integration or raw clock-in/clock-out data.
