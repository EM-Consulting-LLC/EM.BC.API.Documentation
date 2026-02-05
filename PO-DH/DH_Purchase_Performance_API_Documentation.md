# DH – Purchase Performance API Documentation


## 1) Endpoint
Common operations:
- **Find Records** 
- **Create Record** 
- **Get Record** 
> Notes  
> - `systemId`: is a GUID. Can be stored in String/Text field.
> - `Environment`: Sandbox20260119 for test.
> - `Company`: "*MCS International LLC (MCSI_LIVE)*"  *(058eaa30-e178-f011-8eef-000d3aa3bbd3)*
> - `API category` : *EMC/digitalIntegration/v1.0*
> - `Table name`: *contractPerformances*

> - `Flow Trigger` : *Created* or *modified*
> - `Flow Filter` : *if(OData__x0421__x0442__x0430__x0442__x04 eq 'Батлав.')*
---

## 2) Data Model

### 2.1 Purchase Header fields

These fields live on **Purchase Header** (via table extension) and are **NOT directly editable** from the API. They are updated by triggers/logic when Purchase Performance records are created/updated.

- **performanceTotalPercent** 
- **performanceTotalAmount** 
- **performanceStatus**: Done or InProgress
- **performanceCompletionDate** 
- **SystemId**
- **no**
- **dhDocumentNo**

Expected behavior:
- Totals are derived from Purchase Performance lines.
- Status is controlled by business rules (see section 4).
- Completion date is set when status becomes **Done**.

### 2.2 Purchase Performance fields (API)

| API field | BC field | Type | Required | Editable | Notes | DH FIELD |
|---|---|---:|:---:|:---:|---| --- |
| `systemId` | `SystemId` | GUID | System | No | OData key | Title |
| `dhDocumentNo` | `"DH Document No."` | Code[50] | **Yes** | Yes | Must exist in Purchase Header (see validations) | PO дугаар|
| `percentage` | `Percentage` | Decimal | **Yes** | Yes | 0–100; must be > 0 | Ажлын гүйцэтгэлийн хувь |
| `amount` | `Amount` | Decimal | **Yes** | Yes | must be non-zero | Ажлын гүйцэтгэлийн дүн |
| `date` | `Date` | Date | **Yes** | Yes | must be today or earlier | Гүйцэтгэл биелсэн огноо |
| `documentNo` | `"Document No."` | Code[20] | **Yes** | Yes | links to Purchase Header `"No."` | optional |
| `description` | `Description` | Text[200] | No | Yes | optional |Гэрээний нэр |
| `isDone` | `IsDone` | Boolean | No | Yes | may set Purchase Header status Done | if(OData__x0413__x044d__x0440__x044d__x046 eq 'Гэрээ хаах акт') |
| `createdDate` | `CreatedDate` | Date | System | No | set automatically on insert |  |
| `headerRowID` | `"Header RowID"` | GUID | System | No | internal linkage (read-only) |  |

---

## 3) Request Examples

### 3.1 Create Record

**Request**
```json
{
  "dhDocumentNo": "PO-25123615-003", //must be existing DH PO number
  "date": "2026-01-29", // must not be future
  "percentage": 20.5,
  "amount": 150000.00,
  "description": "Performance payment #1",
  "isDone": false // if true, no more insert allowed on ph
}
```

**Response (example)**
```json
{
  "systemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000001",
  "documentNo": "BC123",
  "dhDocumentNo": "PO-25123615-003",
  "date": "2026-01-29",
  "percentage": 20.5,
  "amount": 150000.0,
  "description": "Performance payment #1",
  "isDone": false,
  "createdDate": "2026-01-29",
  "headerRowID": "8d0f6e61-0c4c-ef11-a3f7-6045bd000001" //PO systemId
}
```
---

## 4) Business Rules & Validations (DH requirements)

### 4.1 Required field validations (API must enforce)

1) **DH No. (Required)**
- Must not be empty.
- Must exist on a Purchase Header record.

2) **Date (Required)**
- Must not be empty.
- Must **NOT** be a future date. Allowed: **today and earlier**.

3) **Percentage (Required)**
- Must be provided.
- Must be **> 0**.
- Should be within **0..100** (table field already has Min/Max 0..100).

4) **Amount (Required)**
- Must be provided.
- Must be **non-zero**. (Optional: allow negative?.)

5) **Description**
- Optional.

6) **IsDone**
- Optional.
- If `true`, Purchase Header status must become **Done** (see status rules).

### 4.2 Purchase Header status rules

- If **Purchase Header."Performance Status" = Done**, then **no new Purchase Performance line can be inserted**. 

- On insert/update of Purchase Performance, Purchase Header extension fields are updated:
  - Performance Total Percent
  - Performance Total Amount
  - Performance Status ("In Progress" / "Done")
  - Performance Completion Date (when Done)


## 5) Integration Flow

### 5.1 Typical Integration Flow (DH → Business Central)

1. **DH system prepares Purchase Performance data**
   - `dhDocumentNo` must already exist on a Purchase Header.
   - Purchase Header **must not** be in `Performance Status = Done`.

2. **DH calls Create Purchase Performance API**
   - `Create Record /contractPerformances`
   - Required fields are validated (`dhDocumentNo`, `date`, `percentage`, `amount`).

3. **API validation & linkage**
   - Purchase Header is resolved using `dhDocumentNo` (and `documentNo` if provided).
   - API rejects the request if:
     - Purchase Header does not exist
     - Purchase Header status is already **Done**
     - Any required field validation fails

4. **Purchase Performance record is inserted**
   - `CreatedDate` and internal linkage fields are set automatically.
   - Record becomes part of the Purchase Header aggregation.

5. **Purchase Header recalculation**
   - System recalculates:
     - Performance Total Percent
     - Performance Total Amount
     - Performance Status
   - If `isDone = true`, header status is set to **Done** and completion date is populated.

6. **Response returned to DH**
   - API returns the created record including `systemId`.
   - Further inserts are blocked if header status is **Done**.

---

## 6) Non-Goals & Explicit Exclusions

The following behaviors are **explicitly out of scope** for this API:

- This API **does not create or modify Purchase Header records** directly.
- This API **does not validate or create DH documents**.
- This API **does not auto-balance performance percentages** to reach 100%.
- This API **does not enforce currency conversions or exchange rates**.
- This API **does not support partial updates (PATCH)**; full create/update logic applies.
- This API **does not allow inserts or updates** once Purchase Header status is **Done**.
- This API **does not guarantee idempotency**; duplicate requests may create duplicate records.
- This API **does not handle approval workflows or posting logic**.
- This API **does not expose internal linkage or system-managed fields for editing**.

> Any behavior not explicitly documented above should be considered **unsupported** and must not be relied upon by external systems.
---