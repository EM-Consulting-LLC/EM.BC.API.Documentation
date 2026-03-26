# Annual Leave Ledger Entries - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `AL_Ledgers`
> - `Entity name`: `AL_Ledger`

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/AL_Ledgers`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/AL_Ledgers({systemId})`

Power Automate notes:
- Use **Find records (V3)** for scheduled sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.

---

## 2) Data Model

### 2.1 Editable Fields
All exposed fields are **read-only** in this integration context.

- `PATCH`: Not supported for this contract.
- `POST`: Not part of this integration contract.

### 2.2 Annual Leave Ledger Entries API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| SystemId | SystemId | GUID | No | OData key |
| EmployeeId | "Employee Id" | Code | No | EmpId |
| AnnualLeaveTakenDays | "Annual Leave taken days" | Decimal | No | ЭА-г биеэр эдэлсэн хоног |
| AnnualPaiddays | "Annual Paid days" | Decimal | No | Ээлжийн амралтын цалин тооцсон хоног |
| AlStartDate | "Al Start Date" | Date | No | ЭА-г биеэр эдэлсэн эхлэх огноо |
| ALEndDate | "AL End Date" | Date | No | ЭА-г биеэр эдэлсэн дуусах огноо |
| SalaryCalculationtype | "Salary Calculation type" | Text/Enum | No | ЭА цалин тооцох хугацааны төрөл |
| RequestDate | RequestDate | Date | No | Хүсэлт гаргасан огноо |
| Month | Month | Integer | No | Сар |
| Year | Year | Integer | No | Жил |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/AL_Ledgers?$select=SystemId,EmployeeId,AnnualLeaveTakenDays,AnnualPaiddays,AlStartDate,ALEndDate,Year,Month&$filter=EmployeeId eq 'EMP001'&$top=1000

Response shape (example):
```json
{
  "value": [
    {
      "SystemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000701",
      "EmployeeId": "EMP001",
      "AnnualLeaveTakenDays": 10,
      "AnnualPaiddays": 10,
      "AlStartDate": "2026-07-01",
      "ALEndDate": "2026-07-12",
      "Year": 2026,
      "Month": 7
    }
  ],
  "@odata.nextLink": "..."
}
```

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/AL_Ledgers({systemId})

---

## 4) Business Rules & Validations

1. Key handling
   - Use `SystemId` as immutable technical key.
   - Use `EmployeeId` as business join key for employee-level grouping.

2. Date validation
   - Validate `AlStartDate` and `ALEndDate` formats before loading downstream.
   - Enforce `AlStartDate <= ALEndDate` in validation logic.
   - `RequestDate` is the date the leave request was submitted.

3. Days calculation
   - `AnnualLeaveTakenDays` = days physically taken off.
   - `AnnualPaiddays` = days for which salary was calculated.
   - These may differ depending on `SalaryCalculationtype`.

4. Period fields
   - `Year` and `Month` represent the payroll period associated with the leave entry.
   - Validate `Month` is between 1 and 12.

5. Read-only API behavior
   - Do not send `PATCH`, `POST`, or `DELETE` to this endpoint.
   - Treat as source-of-truth ledger from BC.

6. Pagination and filtering
   - Always process `@odata.nextLink` for full dataset retrieval.
   - Filter by `EmployeeId`, `Year`, or `Month` for targeted sync.

---

## 5) Integration Flow (Power Automate)

Recommended flow pattern (read-only pull):

1. Trigger
   - Scheduled cloud flow (for example every 1 to 6 hours).

2. Find records from BC
   - Call `AL_Ledgers` with `$select`, `$filter` (by `EmployeeId`, `Year`), and `$top`.
   - Iterate pages using `@odata.nextLink`.

3. Transform and validate
   - Validate date ordering and decimal precision.
   - Convert period fields to destination calendar format if needed.

4. Upsert to destination
   - Upsert by `SystemId`.
   - Optional business fallback key: `EmployeeId + AlStartDate + ALEndDate`.

5. Error handling
   - Log request URL, status code, and key fields.
   - Retry transient errors with exponential backoff.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or editing annual leave ledger entries through this endpoint.
- Implementing leave request or approval workflows.
- Calculating leave balances (see Annual Leave Calculation API for balance data).
- Guaranteeing schema stability across future API versions.
- Managing leave policy rules or entitlement logic.
