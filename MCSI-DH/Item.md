# DH - Item API Documentation

## 1) Endpoint

API metadata:
- Publisher/Group/Version: `EMC/digitalIntegration/v1.0`
- API page: `81624 "Item API"`
- Entity: `item`
- Entity set: `items`
- Key field: `id` (`SystemId`, GUID, read-only)

Supported operations:
- Find records (GET `.../items`)
- Get record by id (GET `.../items({id})`)
- Create record (POST `.../items`)
- Update record (PATCH `.../items({id})`)

### 1.1 Integration Parameters (Power Automate)

Use these parameters in BC connector actions (Find/Create/Get/Update):

| Parameter | Example value | Notes |
|---|---|---|
| Tenant | `<your-tenant-id-guid>` | Azure AD tenant ID used by connection. |
| Environment | `Sandbox20260127` (test) or `Production` | BC environment name. |
| Company | `MCS International LLC (MCSI_LIVE)` | Company display name in BC. |
| Company Id | `058eaa30-e178-f011-8eef-000d3aa3bbd3` | Company GUID for API actions requiring company ID. |
| API category (dataset) | `EMC/digitalIntegration/v1.0` | From API page metadata. |
| Table (entity set) | `items` | Item API entity set name. |
| Entity | `item` | Singular entity name. |

Example parameter object:

```json
{
  "bcenvironment": "Sandbox20260127",
  "company": "058eaa30-e178-f011-8eef-000d3aa3bbd3",
  "dataset": "EMC/digitalIntegration/v1.0",
  "table": "items"
}
```

---

## 2) Data Model

### 2.1 Core Fields

| API field | Type | Required | Notes |
|---|---|---|---|
| id | GUID | No | Read-only OData key (`SystemId`). |
| no | Code[20] | Yes | Item No. Cannot be empty. |
| no2 | Code[20] | No | Secondary item number. |
| description | Text[100] | Yes | Cannot be empty. |
| description2 | Text[50] | No | Additional description. |
| baseUnitOfMeasure | Code[10] | Yes | Cannot be empty. |
| itemType | Enum | No | Item type (`Inventory`, `Service`, etc.). |
| itemCategoryCode | Code[20] | Conditional | Required when `itemType = Inventory`. |
| costingMethod | Enum | No | BC costing method enum value. |
| itemTrackingCode | Code[10] | No | Item tracking code. |
| genProdPostingGroup | Code[20] | Yes | Cannot be empty. |
| vatProdPostingGroup | Code[20] | No | VAT product posting group. |
| inventoryPostingGroup | Code[20] | Conditional | Required when `itemType = Inventory`. |
| commonItemNo | Code[20] | No | Common item number. |
| lastDirectCost | Decimal | No | Informational/cost value. |
| blocked | Boolean | No | Blocked status. |
| variantMandatoryIfExists | Boolean | No | Variant behavior in BC. |

### 2.2 Custom Fields From Item Extension

| API field | BC Item field | Type |
|---|---|---|
| long | Long | Integer |
| width | Width (cm) | Integer |
| high | High | Integer |
| thick | Thick | Integer |
| diameter | Diameter | Integer |
| valueType | Value Type | Text[100] |
| material | Material | Text[100] |
| brand | Brand | Text[100] |
| partTag | Part Tag | Text[100] |
| spec | Spec | Text[250] |
| storageCondition | Storage Condition | Text[250] |
| packageInstrucsion | Package Instrucsion | Text[250] |
| msds | MSDS | Boolean |
| energyEfficient | Energy Efficient | Boolean |
| sustainablePurchasing | Sustainable Purchasing | Boolean |

---

## 3) Request Examples

### 3.1 Create/Update Payload Example

```json
{
  "no": "ITEM-1001",
  "description": "Safety helmet",
  "baseUnitOfMeasure": "PC",
  "itemType": "Inventory",
  "itemCategoryCode": "PPE",
  "costingMethod": "Average",
  "genProdPostingGroup": "MATERIAL",
  "vatProdPostingGroup": "VAT10",
  "inventoryPostingGroup": "GENERAL",
  "blocked": false,
  "long": 30,
  "width": 22,
  "high": 18,
  "thick": 3,
  "diameter": 0,
  "valueType": "Equipment",
  "material": "ABS",
  "brand": "3M",
  "partTag": "H-700",
  "spec": "Industrial safety standard",
  "storageCondition": "Store in dry and cool place",
  "packageInstrucsion": "Handle with care, keep upright",
  "msds": true,
  "energyEfficient": false,
  "sustainablePurchasing": true
}
```

### 3.2 Power Automate Mapping Pattern

```json
{
  "item/no": "@triggerBody()?['field_1']",
  "item/description": "@triggerBody()?['Title']",
  "item/baseUnitOfMeasure": "@if(equals(triggerBody()?['field_3'], 'Бараа'), triggerBody()?['field_25'], 'EA')",
  "item/itemType": "@if(equals(triggerBody()?['field_3'], 'Бараа'), 'Inventory', 'Service')",
  "item/width": "@int(triggerBody()?['width'])",
  "item/high": "@int(triggerBody()?['high'])",
  "item/msds": "@if(equals(triggerBody()?['msds'], 'yes'), true, false)"
}
```

---

## 4) Business Rules & Validations

- `no` must not be empty.
- `description` must not be empty.
- `baseUnitOfMeasure` must not be empty.
- `genProdPostingGroup` must not be empty.
- If `itemType = Inventory`, then `itemCategoryCode` is required.
- If `itemType = Inventory`, then `inventoryPostingGroup` is required.
- For `itemType = Inventory`, category validation checks that selected item category has an inventory posting group configured.

Boolean mapping guidance for integrations:
- `yes` -> `true`
- `no` -> `false`

Localized example:
- `Тийм` -> `true`
- `Үгүй` -> `false`

---

## 5) Integration Flow

Recommended Power Automate flow:
1. Trigger on source item created/modified.
2. Normalize text lengths and numeric conversions.
3. Map source fields to BC Item API payload.
4. Upsert logic:
   - Find existing item by `no`.
   - If found, PATCH by `id`.
   - If not found, POST to create.
5. Handle API validation errors and log failed records.

---

## 6) Non-Goals

- This API does not include attachment/document upload.
- This API does not manage item variants, BOMs, or prices beyond exposed item fields.
- This API does not perform custom transformation business logic beyond field validation already defined in the API page.
