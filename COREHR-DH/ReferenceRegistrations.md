# Reference Registrations - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**
- **Create Record**
- **Update Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.1`
> - `Table name (entitySet)`: `ReferenceRegistrations`
> - `Entity name`: `ReferenceRegistration`

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.1/companies({companyId})/ReferenceRegistrations`
- `GET /api/EMC/digitalIntegration/v1.1/companies({companyId})/ReferenceRegistrations({systemId})`

Power Automate notes:
- Use **Find records (V3)** for list sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.

---

## 2) Data Model

### 2.1 Editable Fields

| API Field | Caption |
|---|---|
| No | Системийн дугаар |

All other fields are editable by default (no explicit `Editable = false` on them), but check BC permissions for write access.

### 2.2 Reference Registration API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| SystemId | SystemId | GUID | No | OData key |
| No | "No." | Code | Yes | Системийн дугаар |
| ReferenceNo | "Reference No." | Code/Text | Yes | Тодорхойлолын дугаар |
| ReferenceDate | "Reference Date" | Date | Yes | Тодорхойлолын огноо |
| EmployeeNo | "Employee No." | Code | Yes | Ажилтны дугаар |
| FirstName | "First Name" | Text | Yes | Нэр |
| LastName | "Last Name" | Text | Yes | Овог |
| EmployeeStatus | "Employee Status" | Text/Enum | Yes | Ажилтны төлөв |
| Division | "Division" | Text | Yes | Газар |
| Department | "Department" | Text | Yes | Алба |
| Position | "Position" | Text | Yes | Албан тушаал |
| CompanyType | "Company Type" | Text/Enum | Yes | Байгууллагын төрөл |
| CompanyName | "Company Name" | Text | Yes | Байгууллагын нэр |
| Type | "Type" | Text/Enum | Yes | Төрөл |
| ReasonofReference | "Reason of Reference" | Text | Yes | Шалтгаан |
| ReferenceRecievedDate | "Reference Recieved Date" | Date | Yes | Хүсэлт ирсэн огноо |
| ReferenceStatus | "Reference Status" | Text/Enum | Yes | Тодорхойлолтын статус |
| ReferenceLayoutCode | "Reference Layout Code" | Code | Yes | Загварын код |
| ReportCode | "Report Code" | Code | Yes | Тайлан |
| ReportDescription | "Report Description" | Text | Yes | Тайлангийн тайлбар |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.1/companies({companyId})/ReferenceRegistrations?$select=SystemId,No,ReferenceNo,EmployeeNo,FirstName,LastName,ReferenceStatus&$filter=EmployeeNo eq 'EMP001'&$top=1000

Response shape (example):
```json
{
  "value": [
    {
      "SystemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000401",
      "No": "REF001",
      "ReferenceNo": "R-2025-0001",
      "EmployeeNo": "EMP001",
      "FirstName": "Bat",
      "LastName": "Erdene",
      "ReferenceStatus": "Approved"
    }
  ],
  "@odata.nextLink": "..."
}
```

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.1/companies({companyId})/ReferenceRegistrations({systemId})

### 3.3 Create Record

Request:
POST /api/EMC/digitalIntegration/v1.1/companies({companyId})/ReferenceRegistrations

Body example:
```json
{
  "EmployeeNo": "EMP001",
  "Type": "Employment",
  "ReasonofReference": "Bank loan application"
}
```

### 3.4 Update Record

Request:
PATCH /api/EMC/digitalIntegration/v1.1/companies({companyId})/ReferenceRegistrations({systemId})

Body example:
```json
{
  "ReferenceStatus": "Issued",
  "ReferenceDate": "2026-03-26"
}
```

---

## 4) Business Rules & Validations

1. Key handling
   - Treat `SystemId` as the immutable technical key.
   - `No` is the business-facing reference number; store it for traceability.
   - `EmployeeNo` links back to the Employee entity.

2. Date handling
   - Validate `ReferenceDate` and `ReferenceRecievedDate` formats before loading downstream.
   - `ReferenceRecievedDate` should be ≤ `ReferenceDate` in logical business terms.

3. Status workflow
   - Track `ReferenceStatus` transitions in downstream systems.
   - Only update status through approved workflow steps.

4. API version
   - This endpoint uses `v1.1` (not `v1.0`). Ensure Power Automate connector references the correct version.

5. Pagination and filtering
   - Always process `@odata.nextLink` to avoid partial sync.
   - Prefer `$filter` by `EmployeeNo` or `ReferenceStatus` for targeted retrieval.

---

## 5) Integration Flow (Power Automate)

Recommended flow patterns:

### 5.1 Read/Sync (pull)

1. Trigger
   - Scheduled cloud flow (for example every 1 to 6 hours).

2. Find records from BC
   - Call `ReferenceRegistrations` with `$select`, `$filter`, and `$top`.
   - Iterate pages using `@odata.nextLink`.

3. Transform and validate
   - Validate required keys (`SystemId`, `EmployeeNo`).
   - Normalize date formats for destination.

4. Upsert to destination
   - Upsert by `SystemId`.
   - Optional fallback key: `No` or `ReferenceNo`.

5. Error handling
   - Log failed records with request URL and error body.
   - Retry transient failures with exponential backoff.

### 5.2 Create (push from Digital HR)

1. Trigger
   - When a reference request is submitted in the external system.

2. POST to BC
   - Create a new reference registration record with employee and type details.

3. Error handling
   - Log response status and created `SystemId`.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Generating or printing reference documents/PDFs through this endpoint.
- Defining the reference approval workflow logic.
- Managing reference layout templates or report designs.
- Guaranteeing schema stability across future API versions.
- Implementing downstream deduplication beyond recommended keys.
