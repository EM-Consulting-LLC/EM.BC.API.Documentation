# Education - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `educations`
> - `Entity name`: `education`

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/educations`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/educations({systemId})`

Power Automate notes:
- Use **Find records (V3)** for list sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.

---

## 2) Data Model

### 2.1 Editable Fields
All exposed fields are **read-only** in API page `52114 EmployeeEducationAPI`.

- `PATCH`: Not supported for this contract.
- `POST`: Not part of this integration contract.

### 2.2 Education API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| systemId | SystemId | GUID | No | OData key |
| EmployeeCode | Employee Code | Code | No | EMPID |
| CountryCityProvince | Country, City, Province | Text | No | Country where education was obtained |
| EducationDegree | Education Degree | Enum/Text | No | Education level |
| EducationCenter | Education Center | Text | No | School/University name |
| Major | Major | Text | No | Major |
| Result | GPA | Decimal/Text | No | GPA/Result |
| StartDate | Start Date | Date | No | Education start date |
| EndDate | End Date | Date | No | Graduation date |
| DiplomaNo | Diploma No. | Code/Text | No | Diploma number |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/educations?$select=systemId,EmployeeCode,EducationDegree,EducationCenter,Major,StartDate,EndDate&$filter=EmployeeCode eq 'EMP001'&$top=1000

Response shape (example):
{
  "value": [
    {
      "systemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000101",
      "EmployeeCode": "EMP001",
      "EducationDegree": "Bachelor",
      "EducationCenter": "National University",
      "Major": "Software Engineering",
      "StartDate": "2016-09-01",
      "EndDate": "2020-06-30"
    }
  ],
  "@odata.nextLink": "..."
}

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/educations({systemId})

---

## 4) Business Rules and Validations

1. Key handling
- Treat `systemId` as the immutable technical key.
- Store `EmployeeCode` as business join key in downstream systems.

2. Date handling
- Validate `StartDate` and `EndDate` date formats before loading into external systems.
- If both dates exist, enforce `StartDate <= EndDate` in downstream validation logic.

3. Read-only API behavior
- Do not send `PATCH`, `POST`, or `DELETE` to this endpoint in production flows.
- Keep this endpoint in pull/sync mode only.

4. Pagination and filtering
- Always process `@odata.nextLink` to avoid partial sync.
- Prefer targeted `$select` and `$filter` to reduce API load.

5. Data quality checks
- Flag records with missing `EducationCenter` or `EducationDegree` for business review.

---

## 5) Integration Flow (Power Automate)

Recommended flow pattern (read-only pull):

1. Trigger
- Scheduled cloud flow (for example every 1 to 6 hours).

2. Find records from BC
- Call `educations` with `$select`, `$filter`, and `$top`.
- Loop through pages using `@odata.nextLink`.

3. Transform and validate
- Normalize date formats.
- Validate required business keys (`systemId`, `EmployeeCode`).

4. Upsert to destination
- Upsert by `systemId`.
- Optional fallback match by `EmployeeCode + EducationCenter + StartDate` if needed by target schema.

5. Error handling
- Log failed records with request URL, employee code, and error body.
- Retry transient failures with exponential backoff.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or updating education rows through this endpoint.
- Defining recruitment or HR approval workflows.
- Guaranteeing historical backfill completeness if source data is missing.
- Guaranteeing schema stability across future API versions.
- Implementing conflict resolution between multiple downstream systems.
