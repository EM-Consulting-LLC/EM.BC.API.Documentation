# Annual Leave Calculation - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `annualLeaves`
> - `Entity name`: `annualLeave`
> - All fields are **read-only**. Calculated fields use the `Annual Leave Calculate` codeunit.

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/annualLeaves`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/annualLeaves({systemId})`

Power Automate notes:
- Use **Find records (V3)** for scheduled sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.
- Flow patterns:
  - `Annual Leave Calculation Created -> Digital ХНМС Modified`
  - `Annual Leave calculation modified -> Digital ХНМС modified`

---

## 2) Data Model

### 2.1 Editable Fields
All exposed fields are **read-only**.

- `PATCH`: Not supported for this contract.
- `POST`: Not part of this integration contract.

### 2.2 Annual Leave Calculation API fields

| API field | BC field / Calculation | Type | Editable | Caption |
|---|---|---|:---:|---|
| SystemId | SystemId | GUID | No | OData key |
| EmployeeCode | "Employee Code" | Code | No | EMPID |
| Start_Date | "Start Date" | Date | No | Ажлын жилийн цикл эхлэх огноо |
| End_Date | "End Date" | Date | No | Ажлын жилийн цикл дуусах огноо |
| FromDate | "Earliest WE Date" | Date | No | Улсад шимтгэл төлж ажиллаж эхэлсэн огноо |
| ALEntitlementDays | Entitlement_Days(EmployeeCode) | Decimal | No | Ээлжийн амралтын үндсэн хоног |
| Additional_days__Normal_ | addition_days(EmployeeCode, true) | Decimal | No | Ээлжийн амралтын нэмэгдэл хоног - Хэвийн |
| Additional_days__Abnormal_ | addition_days(EmployeeCode, false) | Decimal | No | Ээлжийн амралтын нэмэгдэл хоног - Хэвийн бус |
| ALEntitlementTotal | Entitlement_Total(EmployeeCode) | Decimal | No | Ээлжийн амралтын нийт хоног |
| ALRemainingDaysInPerson | Entitlement_Total - CalculateALTakenDays | Decimal | No | ЭА-г биеэр эдлэх үлдэгдэл хоног |
| AL_Remaining_Days__Salary_ | Entitlement_Total - CalculateALPaidDays | Decimal | No | Ээлжийн амралтын цалин тооцох үлдэгдэл хоног |
| SHIPeriodHistory_Normal | "SHI Period Total (Normal)" | Decimal | No | Улсад шимтгэл төлж ажилласан жил - Хэвийн |
| SHIPeriodHistoryAbnormal | "SHI Period Total (Abnormal)" | Decimal | No | Улсад шимтгэл төлж ажилласан жил - Хэвийн бус |
| EmploymentDate | "Employment Date" | Date | No | Анх ажилд орсон огноо |
| LastEmploymentDate | "Last Employment Date" | Date | No | Сүүлд ажилд орсон огноо |
| ALPaidDays | CalculateALPaidDays | Decimal | No | Ээлжийн амралтын цалин тооцсон хоног |
| ALTakenDays | CalculateALTakenDays | Decimal | No | Ээлжийн амралтын биеэр эдэлсэн хоног |

### 2.3 Calculated Fields

The following fields are computed at read-time by the `Annual Leave Calculate` codeunit (not stored in the table):

| Field | Calculation |
|---|---|
| ALEntitlementDays | Base entitlement days per employee |
| Additional_days__Normal_ | Additional days for normal working conditions |
| Additional_days__Abnormal_ | Additional days for abnormal working conditions |
| ALEntitlementTotal | Total entitlement (base + additional normal + additional abnormal) |
| ALRemainingDaysInPerson | Total entitlement minus days physically taken |
| AL_Remaining_Days__Salary_ | Total entitlement minus days paid in salary |
| ALPaidDays | Total days for which annual leave salary was calculated |
| ALTakenDays | Total days of annual leave physically taken |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/annualLeaves?$select=SystemId,EmployeeCode,ALEntitlementTotal,ALRemainingDaysInPerson,AL_Remaining_Days__Salary_,ALPaidDays,ALTakenDays&$filter=EmployeeCode eq 'EMP001'&$top=1000

Response shape (example):
```json
{
  "value": [
    {
      "SystemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000801",
      "EmployeeCode": "EMP001",
      "ALEntitlementTotal": 25,
      "ALRemainingDaysInPerson": 15,
      "AL_Remaining_Days__Salary_": 15,
      "ALPaidDays": 10,
      "ALTakenDays": 10
    }
  ],
  "@odata.nextLink": "..."
}
```

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/annualLeaves({systemId})

---

## 4) Business Rules & Validations

1. Key handling
   - Use `SystemId` as immutable technical key.
   - Use `EmployeeCode` as business join key.

2. Cycle dates
   - `Start_Date` and `End_Date` define the annual leave work-year cycle.
   - Each employee has one active cycle record.
   - `FromDate` (`Earliest WE Date`) is the earliest date the employee started contributing to social insurance.

3. Entitlement logic
   - `ALEntitlementDays` is the base annual leave entitlement.
   - Additional days are split by working condition: Normal vs. Abnormal.
   - `ALEntitlementTotal` = `ALEntitlementDays` + `Additional_days__Normal_` + `Additional_days__Abnormal_`.

4. Balance fields
   - `ALRemainingDaysInPerson` = remaining days the employee can physically take off.
   - `AL_Remaining_Days__Salary_` = remaining days that can be compensated as salary.
   - These are calculated values — do not attempt to write back.

5. SHI period
   - `SHIPeriodHistory_Normal` and `SHIPeriodHistoryAbnormal` represent total years of social health insurance contributions under normal/abnormal conditions.
   - These drive the additional days calculation.

6. Read-only API behavior
   - Do not send `PATCH`, `POST`, or `DELETE` to this endpoint.
   - Treat calculated fields as point-in-time snapshots.

7. Pagination and filtering
   - Always process `@odata.nextLink` for full dataset retrieval.
   - Filter by `EmployeeCode` for employee-specific queries.

---

## 5) Integration Flow (Power Automate)

Recommended flow patterns:

### 5.1 Automated sync (event-driven)

1. Trigger
   - `Annual Leave Calculation Created -> Digital ХНМС Modified`
   - `Annual Leave calculation modified -> Digital ХНМС modified`

2. Get record from BC
   - Call `annualLeaves` by `SystemId` or `EmployeeCode`.

3. Map leave balance to Digital HR
   - Map `ALEntitlementTotal`, `ALRemainingDaysInPerson`, `AL_Remaining_Days__Salary_` to target fields.

4. Update destination
   - Update employee record in Digital HR system with current leave balance.

### 5.2 Scheduled full sync (pull)

1. Trigger
   - Scheduled cloud flow (for example daily or weekly).

2. Find records from BC
   - Call `annualLeaves` with `$select`, `$filter`, and `$top`.
   - Iterate pages using `@odata.nextLink`.

3. Validate and transform
   - Verify entitlement math: `ALEntitlementTotal = ALEntitlementDays + Additional Normal + Additional Abnormal`.
   - Flag discrepancies for review.

4. Upsert to destination
   - Upsert by `SystemId`.
   - Optional business fallback key: `EmployeeCode + Start_Date`.

5. Error handling
   - Log request URL, status code, and key fields.
   - Retry transient errors with exponential backoff.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or editing annual leave calculation records through this endpoint.
- Implementing leave request or approval workflows (see AL Ledger Entries for transaction data).
- Modifying entitlement rules, SHI period calculations, or working condition classifications.
- Guaranteeing schema stability across future API versions.
- Providing real-time leave balance (values are snapshots at read time).
