# DH - Vendor Bank Account API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Create Record**
- **Get Record**
- **Update Record**

> Notes
> - `systemId`: is a GUID. Can be stored in String/Text field.
> - `Environment`: Sandbox20260119 for test.
> - `Company`: "*MCS International LLC (MCSI_LIVE)*"  *(058eaa30-e178-f011-8eef-000d3aa3bbd3)*
> - `API category`: *EMC/digitalIntegration/v1.0*
> - `Table name / Entity`: *vendorBankAccounts*
> - `Source Table`: *Vendor Bank Account (288)*
>
> - `Flow Trigger`: *Created* or *modified*

---

## 2) Data Model

### 2.1 Vendor Bank Account Fields

The **Vendor Bank Account** stores the bank details used to pay a vendor. A vendor can have multiple bank accounts.

#### Core Identification

| API field | BC field | Type | Required | Editable | Notes |
|---|---|---:|:---:|:---:|---|
| `id` | `SystemId` | GUID | System | No | OData key; uniquely identifies the bank account record |
| `vendorNo` | `"Vendor No."` | Code[20] | **Yes** | Yes | Links to Vendor table |
| `code` | `Code` | Code[20] | **Yes** | Yes | Bank account code; unique per vendor |
| `bankAccountName` | `Name` | Text[100] | No | Yes | Bank account display name |

#### Bank Details

| API field | BC field | Type | Required | Editable | Notes |
|---|---|---:|:---:|:---:|---|
| `bankAccountNo` | `"Bank Account No."` | Text[50] | No | Yes | Account number provided by the bank |
| `ibanAccount` | `IBAN` | Code[50] | No | Yes | IBAN; API field caption is `IBAN&Account` |
| `bankCodeName` | `"Bank Branch No."` | Text[20] | No | Yes | Branch number / routing identifier; API field caption is `bankBranchNo` |
| `emBankCode` | `"EM Bank Code"` | Code[20] | No | Yes | Custom DH bank code |
| `emBankAccountName` | `"EM Bank Account Name"` | Text[100] | No | Yes | Custom DH bank account name |

#### System & Audit Fields

| API field | BC field | Type | Editable | Notes |
|---|---|---:|:---:|---|
| `systemCreatedAt` | `SystemCreatedAt` | DateTime | No | Set automatically on creation |
| `systemModifiedAt` | `SystemModifiedAt` | DateTime | No | Auto-updated on modification |

---

## 3) Request Examples

### 3.1 Create Vendor Bank Account

**Request**
```json
{
  "vendorNo": "VENDOR001",
  "code": "USD-PRIMARY",
  "bankAccountName": "Primary USD Account",
  "bankAccountNo": "001234567890",
  "ibanAccount": "GB82WEST12345698765432",
  "bankCodeName": "1101",
  "emBankCode": "GBL",
  "emBankAccountName": "Global Bank - USD"
}
```

**Response (example)**
```json
{
  "id": "5d60f9cc-1f60-ef11-a3f7-6045bd000002",
  "vendorNo": "VENDOR001",
  "code": "USD-PRIMARY",
  "bankAccountName": "Primary USD Account",
  "bankAccountNo": "001234567890",
  "ibanAccount": "GB82WEST12345698765432",
  "bankCodeName": "1101",
  "emBankCode": "GBL",
  "emBankAccountName": "Global Bank - USD",
  "systemCreatedAt": "2026-02-27T09:10:00.000Z",
  "systemModifiedAt": "2026-02-27T09:10:00.000Z"
}
```

### 3.2 Update Vendor Bank Account

**Request** (PATCH/PUT to existing bank account)
```json
{
  "bankAccountName": "Primary USD Account - Updated",
  "emBankAccountName": "Global Bank - USD - Updated"
}
```

**Response**
```json
{
  "id": "5d60f9cc-1f60-ef11-a3f7-6045bd000002",
  "vendorNo": "VENDOR001",
  "code": "USD-PRIMARY",
  "bankAccountName": "Primary USD Account - Updated",
  "emBankAccountName": "Global Bank - USD - Updated",
  "systemModifiedAt": "2026-02-27T11:05:00.000Z"
}
```

### 3.3 Get Vendor Bank Account

**Request**
```
GET /vendorBankAccounts('{systemId}')
```

**Response**
```json
{
  "id": "5d60f9cc-1f60-ef11-a3f7-6045bd000002",
  "vendorNo": "VENDOR001",
  "code": "USD-PRIMARY",
  "bankAccountNo": "001234567890",
  "ibanAccount": "GB82WEST12345698765432",
  "bankAccountName": "Primary USD Account",
  "bankCodeName": "1101",
  "emBankCode": "GBL",
  "emBankAccountName": "Global Bank - USD",
  "systemModifiedAt": "2026-02-27T09:10:00.000Z"
}
```

### 3.4 List Vendor Bank Accounts with Filtering

**Request**
```
GET /vendorBankAccounts?$filter=vendorNo eq 'VENDOR001' and emBankCode eq 'GBL'&$orderby=code asc&$top=50
```

**Response** (array of bank accounts)
```json
{
  "value": [
    {
      "id": "5d60f9cc-1f60-ef11-a3f7-6045bd000002",
      "vendorNo": "VENDOR001",
      "code": "USD-PRIMARY",
      "bankAccountName": "Primary USD Account",
      "emBankCode": "GBL",
      "emBankAccountName": "Global Bank - USD"
    },
    {
      "id": "5d60f9cc-1f60-ef11-a3f7-6045bd000003",
      "vendorNo": "VENDOR001",
      "code": "USD-SECONDARY",
      "bankAccountName": "Secondary USD Account",
      "emBankCode": "GBL",
      "emBankAccountName": "Global Bank - USD"
    }
  ]
}
```

---

## 4) Business Rules & Validations (DH requirements)

### 4.1 Required Field Validations (API must enforce)

1) **Vendor No. (Required)**
   - Must not be empty.
   - Must exist on a Vendor record.
   - Vendor must not be blocked for purchases.

2) **Bank Account Code (Required)**
   - `code` must not be empty.
   - Must be unique per `vendorNo`.

3) **Bank Account No. / IBAN (Recommended)**
  - Optional, but recommended for payment files.

### 4.2 Bank Account Integration Rules

- **Vendor Validation**:
  - The Vendor No. must exist and must not be blocked.

- **Code Uniqueness**:
  - `code` is unique within a vendor; same code can be reused by different vendors.

- **Payment Usage**:
  - The API stores data only; selection of bank account during payment is handled by BC payment logic.

- **Data Normalization**:
  - API does not reformat `bankAccountNo` or `ibanAccount`; values are stored as provided.

### 4.3 Data Integrity

- **System Fields**:
  - `id`, `systemCreatedAt`, and `systemModifiedAt` are system-managed and not editable.

- **Uniqueness**:
  - (`vendorNo`, `code`) is unique by design.

---

## 5) Integration Flow

### 5.1 Typical Integration Flow (DH -> Business Central)

1. **DH system prepares vendor bank account data**
   - Collects vendor number and bank account details.

2. **DH calls Create Vendor Bank Account API**
   - `POST /vendorBankAccounts`
   - Required fields: `vendorNo`, `code`.

3. **API validation**
   - Verifies vendor exists and is not blocked.
   - Ensures `code` is unique for that vendor.

4. **Vendor Bank Account record is created**
   - System assigns `systemId`.
   - `lastDateModified` is set.

5. **DH receives response**
   - API returns created record with `systemId`.
   - DH stores `systemId` for updates.

6. **Master Data Sync (optional)**
   - DH periodically queries vendor bank accounts to reconcile data.

---

## 6) Non-Goals & Explicit Exclusions

The following behaviors are **explicitly out of scope** for this API:

- This API **does not validate or create Vendor records**; vendors must pre-exist in BC.
- This API **does not choose default or preferred bank accounts** for payment; payment logic is handled by BC.
- This API **does not validate payment formats** or country-specific IBAN rules.
- This API **does not verify bank account ownership** or anti-fraud checks.
- This API **does not create or update vendor ledger entries**.
- This API **does not manage payment approval or workflow**.
- This API **does not support attachments** (bank letters, documents).
- This API **does not expose internal system fields** for editing.
- This API **does not support partial updates (PATCH)**; clients must provide full record state for updates.

> Any behavior not explicitly documented above should be considered **unsupported** and must not be relied upon by external systems.

---

## 7) Error Handling

### Common Error Responses

| HTTP Status | Error Code | Message | Cause |
|---|---|---|---|
| 400 | BadRequest | "Vendor No. is required" | Missing required vendor |
| 400 | BadRequest | "Vendor VENDOR999 does not exist" | Vendor lookup failed |
| 400 | BadRequest | "Vendor is blocked for purchases" | Blocked vendor cannot be used |
| 400 | BadRequest | "Bank account code already exists" | Duplicate `code` for vendor |
| 400 | BadRequest | "Invalid IBAN format" | IBAN format validation failed |
| 401 | Unauthorized | "Authentication required" | Missing auth token |
| 403 | Forbidden | "Insufficient permissions to create vendor bank account" | User lacks permissions |
| 404 | NotFound | "Vendor Bank Account not found" | Invalid `systemId` |
| 500 | InternalServerError | "Database error occurred" | Transient DB issue |

---

## 8) Pagination & Filtering

### Find / List Operations

**Request**
```
GET /vendorBankAccounts?$filter=vendorNo eq 'VENDOR001' and emBankCode eq 'GBL'&$orderby=code asc&$top=20&$skip=0
```

**Supported Filters**
- `vendorNo`, `code`, `bankAccountNo`, `emBankCode`, `emBankAccountName`, `systemModifiedAt`

**Supported Orderby**
- `vendorNo`, `code`, `bankAccountName`, `emBankCode`, `systemModifiedAt`

**Pagination Parameters**
- `$top`: Max records to return (default: 100, max: 1000)
- `$skip`: Number of records to skip for pagination