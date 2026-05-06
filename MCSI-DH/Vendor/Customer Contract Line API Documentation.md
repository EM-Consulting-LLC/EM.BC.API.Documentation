# DH – Vendor/Customer Contract Line API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Create Record**
- **Get Record**
- **Update Record**
- **Delete Record**

> Notes
> - `rowId` (SystemId) is a GUID and can be stored in Text/String.
> - `Environment`: DH_INT for test. Production for production.
> - `Company`: *MCS International LLC (MCSI_LIVE)* *(058eaa30-e178-f011-8eef-000d3aa3bbd3)*
> - `API category`: *EMC/digitalIntegration/v1.0*
> - `Entity Name`: *vendorCustomerContractLine*
> - `Entity Set Name`: *vendorCustomerContractLines*
> - `Table name`: *Vendor/Customer Contract Line* (ID 81636)
> - `Parent Table`: *Vendor/Customer Contract* (ID 81604)
> - `Flow Trigger`: *Created* or *modified*

---

## 2) Data Model

### 2.1 Contract Line API Fields

| API Field | BC Field | Type | Required | Editable | Notes | DH Field Name (MN) |
|---|---|---|---|---|---|---|
| `rowId` | `SystemId` | GUID | System | No | OData key field | |
| `contractNumber` | `Contract Number` | Code[50] | **Yes** | Yes | Must not be empty; must exist in parent contract | Гэрээний дугаар |
| `lineNo` | `Line No.` | Integer | System | No | Auto-assigned| |
| `contractActNumber` | `Contract Act Number` | Code[50] | No | Yes | Optional | Гэрээний актны дугаар |
| `contractActName` | `Contract Act Name` | Text[100] | No | Yes | Optional | Гэрээний актны нэр |
| `contractCode` | `Contract Code` | Code[50] | No | Yes | Optional | Гэрээний код |
| `purchaseOrderNo` | `Purchase Order No.` | Code[20] | No | Yes | Synced from parent contract during insert/contract validate | PO дугаар |
| `totalAmount` | `Total Amount` | Decimal | No | Yes | Synced from parent contract during insert/contract validate | Гэрээний үнийн дүн |
| `workPerformancePercentage` | `Work Performance Percentage` | Decimal | No | Yes | DecimalPlaces 0:2 | Ажлын гүйцэтгэлийн хувь |
| `workPerformanceAmount` | `Work Performance Amount` | Decimal | No | Yes | Optional amount | Ажлын гүйцэтгэлийн дүн |
| `vendorNo` | `Vendor No.` | Code[20] | No | Yes | TableRelation Vendor.No. | Гэрээлэгчийн регистр |
| `vendorName` | `Vendor Name` | Text[100] | No | Yes | Auto-filled from Vendor when `vendorNo` validates | Гэрээлэгчийн нэр |
| `contractType` | `Contract Type` | Text[50] | No | Yes | Synced from parent contract | Гэрээний төрөл |
| `contractName` | `Contract Name` | Text[100] | No | Yes | Synced from parent contract | Гэрээний нэр |
| `status` | `Status` | Text[100] | No | Yes | Synced from parent contract | Статус |

---

### 2.2 Parent Sync Logic

When `contractNumber` is provided (or on insert), the line syncs these fields from parent table `Vendor/Customer Contract`:
- `purchaseOrderNo`
- `totalAmount`
- `contractType`
- `contractName`
- `status`

Additional behavior:
- If parent `contractType = 'Vendor'`, then:
  - `vendorNo` is copied from parent `No.`
  - `vendorName` is copied from parent `Name`

---

## 3) Request Examples

### 3.1 Create Contract Line

**Request**
```json
{
  "contractNumber": "CNT-2026-001",
  "contractActNumber": "ACT-2026-001",
  "contractActName": "Contract closing act - phase 1",
  "contractCode": "CONT-CODE-001",
  "workPerformancePercentage": 35.50,
  "workPerformanceAmount": 1800000,
  "vendorNo": "V00010"
}
```


```
---

## 4) Business Rules & Validations

### 4.1 Required Validation
1) **Contract Number**
- Must not be empty.
- Must exist in `Vendor/Customer Contract`.

### 4.2 System-generated and Auto-sync Values

1) **Header-to-line sync**
- On insert and on contract number validation, line gets parent values:
  - PO no, total amount, contract type, contract name, status

2) **Vendor Name auto-fill**
- On `vendorNo` validate, `vendorName` is read from Vendor master.

### 4.3 Data Type Notes
- `workPerformancePercentage` supports up to 2 decimal places (0:2).
- No custom max/min rule is currently implemented in API/page triggers.

---

## 5) Configuration Reference

| Setting | Value |
|---|---|
| API Publisher | EMC |
| API Group | digitalIntegration |
| API Version | v1.0 |
| Entity Name | vendorCustomerContractLine |
| Entity Set Name | vendorCustomerContractLines |

**Last Updated:** May 6, 2026
