# Position - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**

> Notes:
> - `systemId` is a GUID and is the OData key.
> - `Environment`: use your target BC environment (Sandbox or Production).
> - `Company`: use the target company name and `companyId`.
> - `API category`: `EMC/digitalIntegration/v1.0`
> - `Table name (entitySet)`: `positions`
> - `Entity name`: `position`
> - This API is read-only (all fields have `Editable = false`).

Base URL pattern:
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/positions`
- `GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/positions({systemId})`

Power Automate notes:
- Use **Find records (V3)** for list sync.
- Use `$select`, `$filter`, and pagination (`@odata.nextLink`) to control payload size.

---

## 2) Data Model

### 2.1 Editable Fields
No editable fields.

- `PATCH`: Not supported.
- `POST`: Not supported.
- `DELETE`: Not supported.

### 2.2 Position API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| SystemId | SystemId | GUID | No | OData key |
| Code | Code | Code | No | Position Code |
| DescriptionMN | "Description (MN)" | Text | No | Description (MN) |
| DescriptionEN | "Description (EN)" | Text | No | Description (EN) |
| Grade | Grade | Code | No | Grade |
| PositionSegment | "Position Segment" | Text | No | Position Segment |
| Status | Status | Text/Enum | No | Status |

---

## 3) Request Examples

### 3.1 Find Records

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/positions?$select=SystemId,Code,DescriptionMN,DescriptionEN,Grade,PositionSegment,Status&$filter=Status eq 'Active'&$top=1000

Response shape (example):
```json
{
  "value": [
    {
      "SystemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000901",
      "Code": "POS001",
      "DescriptionMN": "Ахлах менежер",
      "DescriptionEN": "Senior Manager",
      "Grade": "G5",
      "PositionSegment": "Management",
      "Status": "Active"
    }
  ],
  "@odata.nextLink": "..."
}
```

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/positions({systemId})

---

## 4) Business Rules & Validations

1. Key handling
   - Use `SystemId` as immutable technical key.
   - Use `Code` as business reference key for position lookups and joins with Employee data.

2. Bilingual descriptions
   - `DescriptionMN` is the Mongolian position title.
   - `DescriptionEN` is the English position title.
   - Downstream systems should store both and display based on user locale.

3. Grade and segment
   - `Grade` represents the position's pay/level grade.
   - `PositionSegment` categorizes the position (e.g., Management, Technical, Administrative).
   - Use these for organizational reporting and filtering.

4. Status
   - Filter by `Status` to retrieve only active positions in most sync scenarios.
   - Inactive positions may still be referenced by historical employee records.

5. Read-only API behavior
   - Do not send `PATCH`, `POST`, or `DELETE` to this endpoint.
   - Position master data is managed exclusively in BC.

6. Pagination and filtering
   - Always process `@odata.nextLink` for full dataset retrieval.
   - Filter by `Status` for active-only sync.

---

## 5) Integration Flow (Power Automate)

Recommended flow pattern (read-only pull):

1. Trigger
   - Scheduled cloud flow (for example daily or weekly — positions change infrequently).

2. Find records from BC
   - Call `positions` with `$select`, `$filter` (by `Status`), and `$top`.
   - Iterate pages using `@odata.nextLink`.

3. Transform and validate
   - Validate required fields (`Code`, `DescriptionMN`).
   - Normalize grade and segment values for target schema.

4. Upsert to destination
   - Upsert by `SystemId`.
   - Optional business fallback key: `Code`.

5. Error handling
   - Log request URL, status code, and key fields.
   - Retry transient errors with exponential backoff.

---

## 6) Non-Goals

The following are out of scope for this API integration contract:

- Creating or editing position records through this endpoint.
- Defining organizational hierarchy or reporting lines.
- Managing position vacancy or headcount planning.
- Guaranteeing schema stability across future API versions.
- Linking positions to compensation or benefit structures.
