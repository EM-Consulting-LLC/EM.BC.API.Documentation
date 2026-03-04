# DH – Contract Document Attachment API Documentation

## 1) Endpoint
Common operations:
- **Find Records** 
- **Create Record** 
- **Get Record** 
- **Update Record**
- **Delete Record**

> Notes  
> - `systemId`: is a GUID. Can be stored in String/Text field.
> - `Environment`: Sandbox20260212 for test.
> - `Company`: "*MCS International LLC (MCSI_LIVE)*"  *(058eaa30-e178-f011-8eef-000d3aa3bbd3)*
> - `API category` : *emconsulting/digitalIntegration/v1.0*
> - `Table name`: *contractDocumentAttachments*
> - `Entity Set Name`: *contractDocumentAttachments*
> - `Scope`: Document attachments for **Vendor/Customer Contracts only**

> - `Flow Trigger` : *Created* or *modified*

---

---

## 2) Vendor/Customer Contract API Fields

| API Field | BC Field | Type | Description | Required |
|---|---|---|---|---|
| `rowId` | SystemId | GUID | System ID - unique identifier | Yes (Auto) |
| `contractNumber` | Contract Number | Code[20] | Unique contract identifier | Yes |
| `contractName` | Contract Name | Text[100] | Contract name/title | Yes |
| `contractType` | Contract Type | Code[20] | Vendor or Customer | Yes |
| `no` | No. | Code[20] | Vendor/Customer number | Yes |
| `name` | Name | Text[100] | Vendor/Customer name | Yes |
| `address` | Address | Text[100] | Vendor/Customer address | No |
| `purchaseOrderNo` | Purchase Order No. | Code[20] | Related PO number | No |
| `analyticAccount` | Analytic Account | Code[20] | Analytic account code | No |
| `division` | Division | Code[20] | Division code | No |
| `currencyCode` | Currency Code | Code[10] | ISO currency code | No |
| `totalAmount` | Total Amount | Decimal | Contract total value | No |
| `remainingAmount` | Remaining Amount | Decimal | Remaining contract value | No |
| `totalNumberOfAmendments` | Total Number of Amendments | Integer | Amendment count | No |
| `paymentReference` | Payment Reference | Text[100] | Payment reference | No |
| `shipmentMethodsCode` | Shipment Methods Code | Code[10] | Shipment method | No |
| `generalDescription` | General Description | Text[2000] | Contract description | No |
| `deliveryLocation` | Delivery Location | Text[100] | Delivery location | No |
| `packingRequirement` | Packing Requirement | Text[500] | Packing requirements | No |
| `specialRequirement` | Special Requirement | Text[500] | Special requirements | No |
| `status` | Status | Code[20] | pending/approved/canceled | No |
| `startDate` | Start Date | Date | Contract start date | No |
| `endDate` | End Date | Date | Contract end date | No |
| `requiredDate` | Required Date | Date | Required delivery date | No |
| `warrantyDate` | Warranty Date | Date | Warranty expiration date | No |
| `supplyDay` | Supply Day | Integer | Supply days | No |
| `assignedEmployee` | Assigned Employee | Code[20] | Assigned employee | No |
| `createdDate` | Created Date | Date | Creation date (read-only) | No |
| `createdBy` | Created By | Code[50] | Created by user (read-only) | No |
| `modified` | Modified | DateTime | Last modification (read-only) | No |

---

## 3) Document Attachment API Fields

| API Field | BC Field | Type | Description | Required |
|---|---|---|---|---|
| `rowId` | SystemId | GUID | System ID - unique identifier | Yes (Auto) |
| `tableId` | Table ID | Integer | Always 81604 (Vendor/Customer Contract) | Yes |
| `contractNumber` | No. | Code[20] | Contract number the attachment is linked to | Yes |
| `fileName` | File Name | Text[250] | File name without extension | Yes |
| `fileExtension` | File Extension | Text[30] | File extension (pdf, docx, xlsx, etc.) | Yes |
| `base64Content` | Document Reference ID | Media | Base64-encoded file content | Yes (Insert only) |

---


## 4) Request/Response Examples

### Create Contract Example

**Request:**
```json

{
  "contractNumber": "CNT-2024-001",
  "contractName": "Annual Supply Agreement",
  "contractType": "Vendor",
  "no": "VENDOR-001",
  "name": "Acme Corporation",
  "status": "pending",
  "totalAmount": 50000,
  "currencyCode": "USD"
}
```

### Create Attachment Example

**Request:**
```json

{
  "tableId": 81604,
  "contractNumber": "CNT-2024-001",
  "fileName": "SignedAgreement",
  "fileExtension": "pdf",
  "base64Content": "JVBERi0xLjQK..."
}
```

---

## 5) Best Practices

1. **Contract Numbering**: Use unique, meaningful contract numbers
2. **File Management**: Keep file names descriptive without extension
3. **Base64 Encoding**: Properly encode file content before transmission
4. **Status Workflow**: Follow consistent progression (pending → approved → canceled)
5. **Error Handling**: Implement retry logic in Power Automate
6. **Permissions**: Ensure users have appropriate BC roles
7. **Validation**: Validate all required fields before API calls
8. **Filtering**: Use OData $filter for efficient data retrieval

---

## 9) Configuration Reference

| Setting | Value |
|---|---|
| **Environment** | Sandbox20260212 |
| **Company** | MCS International LLC (MCSI_LIVE) |
| **Publisher** | EMC |
| **API Group** | digitalIntegration |
| **API Version** | v1.0 |
| **Contract Table ID** | 81604 |
| **Attachment Table ID** | 1173 |

**Power Automate Flow:** [View Contract Management Flow](https://make.powerautomate.com/environments/default-b68b6df0-24c3-40c7-8935-e80a48efc7c1/flows/c24dad81-e4d9-deea-c262-97523e02d016/details)

**Last Updated:** March 4, 2026