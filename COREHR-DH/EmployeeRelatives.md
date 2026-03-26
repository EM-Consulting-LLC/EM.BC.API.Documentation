# Employee Relatives - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `relatives`
> - `Entity name`: `relative`

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/relatives`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/relatives({systemId})`

Power Automate notes:
- Use **Find records (V3)** for list sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.
- Flow pattern: `BC Relatives created -> Гэр бүлийн мэдээлэл created`

---

## 2) Data Model

### 2.1 Editable Fields
All exposed fields are **read-only** in this integration context.

- `PATCH`: Not supported for this contract.
- `POST`: Not part of this integration contract.

### 2.2 Employee Relatives API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| SystemId | SystemId | GUID | No | OData key |
| EmployeeCode | "Employee Code" | Code | No | EMPID |
| Relatives | Relatives | Text/Enum | No | Таны хэн болох |
| Firstname | "Employee First Name" | Text | No | Ажилтны Нэр |
| RelativeFirstName | "Relative First Name" | Text | No | Нэр |
| RelativeLastName | "Relative Last Name" | Text | No | Эцэг/Эхийн нэр |
| RegistrationNo | "Registration No." | Text | No | РД |
| BirthDate | "Birth Date" | Date | No | Төрсөн огноо |
| Major | Major | Text | No | Мэргэжил, боловсрол |
| CurrentSchoolWorkplace | "Current Workplace/School" | Text | No | Одоо эрхэлж буй ажил, сургууль |
| Position | Position | Text | No | Албан тушаал |
| PhoneNumber | "Phone No." | Text | No | Утасны дугаар |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/relatives?$select=SystemId,EmployeeCode,Relatives,RelativeFirstName,RelativeLastName,BirthDate,PhoneNumber&$filter=EmployeeCode eq 'EMP001'&$top=1000

Response shape (example):
```json
{
  "value": [
    {
      "SystemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000601",
      "EmployeeCode": "EMP001",
      "Relatives": "Spouse",
      "RelativeFirstName": "Saran",
      "RelativeLastName": "Bat",
      "BirthDate": "1990-05-15",
      "PhoneNumber": "+97699887766"
    }
  ],
  "@odata.nextLink": "..."
}
```

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/relatives({systemId})

---

## 4) Business Rules & Validations

1. Key handling
   - Use `SystemId` as immutable technical key.
   - Use `EmployeeCode` as business join key for employee-level grouping.

2. Relationship field
   - `Relatives` indicates the relationship type (e.g., Spouse, Child, Parent).
   - Validate against expected values in downstream systems.

3. Personal data sensitivity
   - `RegistrationNo` (Registration Number) is a sensitive personal identifier.
   - Apply appropriate data protection and masking rules in downstream storage.
   - `BirthDate` and `PhoneNumber` are also PII — handle accordingly.

4. Employee name context
   - `Firstname` is the **employee's** first name (not the relative's).
   - `RelativeFirstName` and `RelativeLastName` are the relative's names.
   - Do not confuse these fields in downstream mapping.

5. Read-only API behavior
   - Do not send `PATCH`, `POST`, or `DELETE` to this endpoint.
   - Treat as source-of-truth read model from BC.

6. Pagination and filtering
   - Always process `@odata.nextLink` for full retrieval.
   - Filter by `EmployeeCode` for employee-specific sync.

---

## 5) Integration Flow (Power Automate)

Recommended flow pattern (read-only pull):

1. Trigger
   - Automated: `BC Relatives created -> Гэр бүлийн мэдээлэл created`
   - Or scheduled cloud flow (for example every 1 to 6 hours).

2. Find records from BC
   - Call `relatives` with `$select`, `$filter` (by `EmployeeCode`), and `$top`.
   - Iterate pages using `@odata.nextLink`.

3. Transform and validate
   - Validate date formats for `BirthDate`.
   - Normalize relationship type values.
   - Mask or encrypt sensitive fields before storing downstream.

4. Upsert to destination
   - Upsert by `SystemId`.
   - Optional business fallback key: `EmployeeCode + RelativeFirstName + Relatives`.

5. Error handling
   - Log request URL, status code, and key fields (excluding PII in logs).
   - Retry transient errors with exponential backoff.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or editing relative records through this endpoint.
- Defining benefit eligibility rules based on family data.
- Managing document attachments for family certificates.
- Guaranteeing schema stability across future API versions.
- Implementing data privacy compliance (GDPR, LPDP) — this is the integrator's responsibility.
