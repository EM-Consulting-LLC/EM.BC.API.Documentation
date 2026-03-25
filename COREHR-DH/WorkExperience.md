# Work Experience - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `workExperiences`
> - `Entity name`: `workExperience`

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/workExperiences`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/workExperiences({systemId})`

Power Automate notes:
- Use **Find records (V3)** for list sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.

---

## 2) Data Model

### 2.1 Editable Fields
All exposed fields are **read-only** in API page `52115 WorkExperience`.

- `PATCH`: Not supported for this contract.
- `POST`: Not part of this integration contract.

### 2.2 Work Experience API fields

| API field | BC field | Type | Editable | Description |
|---|---|---|:---:|---|
| systemId | SystemId | GUID | No | OData key |
| EmployeeCode | Employee Code | Code | No | Employee ID |
| Position | Position | Text | No | Job position/title |
| FromDate | Start Date | Date | No | Work experience start date |
| EndDate | End Date | Date | No | Work experience end date |
| CompanyNo | Company No. | Code | No | Company reference code |
| CompanyName | Company Name | Text | No | Company name |
| Firstname | Derived from Employee.First Name | Text | No | Employee first name |
| Lastname | Derived from Employee.Last Name | Text | No | Employee last name |
| email1 | Derived from Employee.Company E-Mail | Text | No | Employee company email |
| IsMCS1 | IsMCS | Boolean | No | MCS indicator |
| MCSStartdate1 | Derived from Company Registration.Start Date | Date | No | Company registration start date |
| MCSEnddate1 | Derived from Company Registration.End Date | Date | No | Company registration end date |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/workExperiences?$select=systemId,EmployeeCode,CompanyName,Position,FromDate,EndDate,IsMCS1&$filter=EmployeeCode eq 'EMP001'&$top=1000

Response shape (example):
{
  "value": [
    {
      "systemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000201",
      "EmployeeCode": "EMP001",
      "CompanyName": "Example LLC",
      "Position": "Analyst",
      "FromDate": "2019-02-01",
      "EndDate": "2021-03-31",
      "IsMCS1": true
    }
  ],
  "@odata.nextLink": "..."
}

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/workExperiences({systemId})

---

## 4) Business Rules and Validations

1. Key handling
- Treat `systemId` as immutable integration key.
- Keep `EmployeeCode` for business-side traceability and reconciliation.

2. Date rules
- Validate date format for `FromDate`, `EndDate`, `MCSStartdate1`, and `MCSEnddate1`.
- If both values are present, enforce `FromDate <= EndDate`.

3. Derived fields behavior
- `Firstname`, `Lastname`, and `email1` are calculated from Employee data.
- `MCSStartdate1` and `MCSEnddate1` are calculated from Company Registration.
- Downstream systems should treat derived values as snapshots from BC at read time.

4. Read-only API behavior
- Do not send `PATCH`, `POST`, or `DELETE` in production integrations.
- Use pull-only synchronization.

5. Pagination and filtering
- Process `@odata.nextLink` for full dataset retrieval.
- Use `$filter` by `EmployeeCode` for targeted sync when possible.

---

## 5) Integration Flow (Power Automate)

Recommended flow pattern (read-only pull):

1. Trigger
- Scheduled cloud flow (for example every 1 to 6 hours).

2. Find records from BC
- Call `workExperiences` endpoint with `$select`, `$filter`, and `$top`.
- Iterate through all pages via `@odata.nextLink`.

3. Transform and validate
- Validate date ordering and required keys.
- Normalize booleans and null values for target schema.

4. Upsert to destination
- Upsert by `systemId`.
- Optional fallback match by `EmployeeCode + CompanyNo + FromDate` if needed.

5. Error handling
- Log response status, request URL, and record key values.
- Retry transient errors with backoff and alert on repeated failures.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or editing work experience in BC through this endpoint.
- Defining employee lifecycle business processes.
- Backfilling missing historical records outside BC source data.
- Guaranteeing schema stability across future API versions.
- Managing downstream deduplication strategy beyond recommended keys.
