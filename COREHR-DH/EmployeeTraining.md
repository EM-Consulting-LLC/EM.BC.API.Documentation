# Employee Training - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `employeeTrainings`
> - `Entity name`: `employeeTraining`
> - This API is read-only (`InsertAllowed = false`, `ModifyAllowed = false`, `DeleteAllowed = false`).

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/employeeTrainings`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/employeeTrainings({systemId})`

Power Automate notes:
- Use **Find records (V3)** for scheduled sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to reduce payload size.

---

## 2) Data Model

### 2.1 Editable Fields
No editable fields.

- `PATCH`: Not supported.
- `POST`: Not supported.
- `DELETE`: Not supported.

### 2.2 Employee Training API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| systemId | SystemId | GUID | No | OData key |
| ID | ID | Integer | No | ID |
| EmployeeCode | Employee Code | Code[50] | No | Ажилтны код |
| Lastname | Lastname | Text[50] | No | Ажилтны овог |
| Firstname | Firstname | Text[50] | No | Ажилтны нэр |
| Department | Department | Text[100] | No | Хэлтэс |
| Position | Position | Text[100] | No | Албан тушаал |
| TrainingName | Training Name | Text[100] | No | Сургалтын нэр |
| Location | Location | Text[300] | No | Байршил |
| TrainingType | Training Type | Text[100] | No | Сургалтын төрөл |
| TrainingOrganizer | Training Organizer | Text[100] | No | Сургалт зохион байгуулагч |
| TrainingFee | Training Fee | Decimal | No | Сургалтын төлбөр |
| TotalTrainingTime | Total Training Time | Text[100] | No | Нийт суралцсан хугацаа |
| TrainingDate | Training Date | Date | No | Сургалтанд суусан огноо |
| BusinessTripExpense | Business Trip Expense | Decimal | No | Томилолтын зардал |
| Explanation | Explanation | Text[200] | No | Тайлбар |
| Certificate | Certificate | Text[100] | No | Certificate |
| Report | Report | Text[200] | No | Тайлан |
| Contracted | Contracted | Boolean | No | Гэрээтэй эсэх |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/employeeTrainings?$select=systemId,EmployeeCode,TrainingName,TrainingType,TrainingDate,TrainingFee,Contracted&$filter=EmployeeCode eq 'EMP001'&$top=1000

Response shape (example):
{
  "value": [
    {
      "systemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000301",
      "EmployeeCode": "EMP001",
      "TrainingName": "Safety Training",
      "TrainingType": "Internal",
      "TrainingDate": "2025-04-15",
      "TrainingFee": 250000,
      "Contracted": true
    }
  ],
  "@odata.nextLink": "..."
}

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/employeeTrainings({systemId})

---

## 4) Business Rules & Validations

1. Key handling
- Use `systemId` as immutable technical key.
- Keep `EmployeeCode` as business reference for reconciliation.

2. Date and amount checks
- Validate `TrainingDate` format before loading downstream.
- Validate decimal parsing for `TrainingFee` and `BusinessTripExpense`.

3. Read-only contract
- Do not send `PATCH`, `POST`, or `DELETE` to this endpoint.
- Treat this API as source-of-truth read model from BC.

4. Data quality checks
- Flag records with missing `TrainingName` or `TrainingType`.
- If `Contracted = true`, enforce downstream rule that contract evidence exists.

5. Pagination
- Always follow `@odata.nextLink` for full extraction.

---

## 5) Integration Flow

Recommended Power Automate flow (read-only pull):

1. Trigger
- Scheduled cloud flow (for example every 1 to 6 hours).

2. Read data from BC
- Call `employeeTrainings` with `$select`, `$filter`, and `$top`.
- Iterate pages using `@odata.nextLink`.

3. Validate and transform
- Validate key fields and convert dates/decimals to destination format.

4. Upsert in destination
- Upsert by `systemId`.
- Optional business fallback key: `EmployeeCode + TrainingName + TrainingDate`.

5. Error handling
- Log request URL, status code, and key fields.
- Retry transient errors with exponential backoff.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or editing training records through API.
- Managing attachments/document binaries for certificates or reports.
- Implementing approval workflow for training enrollment or completion.
- Guaranteeing schema stability across future API versions.
- Defining downstream deduplication beyond recommended keys.
