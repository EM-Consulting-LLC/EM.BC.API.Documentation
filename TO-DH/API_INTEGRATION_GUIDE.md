# Payment Request Status API - Integration Guide
**For DH System Developers**

---

## Overview

API integration to automatically sync Payment Request status changes from Business Central (BC) to DH system.

**Architecture:**
```
Power Apps (DH) → SharePoint List → Power Automate → Business Central API → Update DH System
```

---

## 1. Business Central API Endpoint

### Base Information
- **API Publisher:** `emconsulting`
- **API Group:** `paymentrequest`
- **API Version:** `v1.0`
- **Entity Name:** `paymentRequestStatus`
- **Entity Set Name:** `paymentRequestStatuses`

### Endpoint URL Format
```
https://[BC-SERVER-URL]/[BC-INSTANCE]/api/emconsulting/paymentrequest/v1.0/companies([COMPANY-ID])/paymentRequestStatuses
```

**Example:**
```
https://api.businesscentral.dynamics.com/v2.0/[TENANT-ID]/Production/api/emconsulting/paymentrequest/v1.0/companies(12345678-1234-1234-1234-123456789012)/paymentRequestStatuses
```

**How to get Company ID:**
```
GET https://[BC-SERVER-URL]/[BC-INSTANCE]/api/v2.0/companies
```

---

## 2. API Response Schema

### Fields Returned

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| `id` | GUID | Unique system identifier | `a1b2c3d4-...` |
| `documentNo` | String | Payment Request number | `PR-00001` |
| `dhPaymentStatementNo` | String | DH system reference number | `DH-2026-001` |
| `status` | Integer | Status option value | `4` |
| `statusText` | String | Status text | `Approved` |
| `currentApproverUserID` | String | Current approver user ID | `BCADMIN` |
| `lastModifiedDateTime` | DateTime | Last modified date time | `2026-02-03T14:30:00Z` |

### Status Values Mapping

| Value | statusText | Description |
|-------|-----------|-------------|
| 0 | Open | Open |
| 1 | Pending Approval | Pending approval |
| 2 | Budget Checked | Budget checked |
| 3 | Accountant Checked | Accountant checked |
| 4 | Approved | Approved |
| 5 | Cancelled | Cancelled |

### Sample Response

```json
{
  "@odata.context": "https://api.../api/.../v1.0/$metadata#companies(12345678-1234-1234-1234-123456789012)/paymentRequestStatuses",
  "value": [
    {
      "@odata.etag": "W/\"JzQ0O0VBQUFBQUFBQUFBPSc=\"",
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "documentNo": "PR-00001",
      "dhPaymentStatementNo": "DH-2026-001",
      "status": 4,
      "statusText": "Approved",
      "currentApproverUserID": "BCADMIN",
      "lastModifiedDateTime": "2026-02-03T14:30:00Z"
    }
  ]
}
```

---

## 3. OData Query Options

### Filter by Last Modified DateTime
```
GET .../paymentRequestStatuses?$filter=lastModifiedDateTime gt 2026-02-03T10:00:00Z
```

### Filter by DH Payment Statement No
```
GET .../paymentRequestStatuses?$filter=dhPaymentStatementNo eq 'DH-2026-001'
```

### Filter by Status
```
GET .../paymentRequestStatuses?$filter=status eq 4
```

### Select Specific Fields
```
GET .../paymentRequestStatuses?$select=documentNo,dhPaymentStatementNo,statusText,lastModifiedDateTime
```

### Order By
```
GET .../paymentRequestStatuses?$orderby=lastModifiedDateTime desc
```

---

## 4. Power Automate Setup (When a record is modified V3)

### Connection Settings - Sandbox/Test Environment

**Trigger:** When a record is modified (V3)

**Business Central Connection Parameters:**
- **Environment:** `Sand20260127` *(Sandbox/Test Environment)*
- **Company:** `MCS International LLC (MCSI_LIVE)`
- **API Category:** `paymentrequest`
- **Table Name:** `paymentRequestStatuses`

### Step-by-Step Setup

#### Step 1: Create New Flow
- Power Automate → Create → Automated cloud flow
- Flow name: `BC Payment Request Status Sync`

#### Step 2: Add Trigger
- Search: "Dynamics 365 Business Central"
- Select: **"When a record is modified (V3)"**

#### Step 3: Configure Trigger
Click "Show advanced options" and fill in:
```
Environment: Sand20260127
Company: MCS International LLC (MCSI_LIVE)
API Category: paymentrequest
Table Name: paymentRequestStatuses
```

#### Step 4: Add Actions
1. **Condition:** Check if `dhPaymentStatementNo` is not empty
2. **Get items:** From DH System (filter by `dhPaymentStatementNo`)
3. **Update item:** Update DH record with `statusText` and `currentApproverUserID`
4. **Log** (optional): Save to SharePoint log

### Trigger Output Fields

- `id` - System ID
- `documentNo` - Payment Request number
- `dhPaymentStatementNo` - DH reference number
- `status` - Status code (0-5)
- `statusText` - Status text
- `currentApproverUserID` - Approver user ID

### Example Flow Structure

```
Trigger: When a record is modified (V3)
  └─ Condition: dhPaymentStatementNo is not empty
      ├─ Yes:
      │   ├─ Get items from DH System (filter by dhPaymentStatementNo)
      │   └─ Update DH record
      │       • Payment Transfer Status = statusText
      │       • Comment = "Approver: currentApproverUserID"
      └─ No: Skip
```

**Benefits:**
- ✅ Real-time updates (no polling delay)
- ✅ Automatic webhook management
- ✅ Lower API call usage
- ✅ Immediate sync when status changes

---

## 5. Error Handling

### Common Errors

#### Error 1: 401 Unauthorized
**Cause:** Authentication failed
**Solution:** Check API permissions and Power Automate connection

#### Error 2: 404 Not Found
**Cause:** API endpoint not found or company ID incorrect
**Solution:** Verify extension is published and Company ID is correct

#### Error 3: 500 Internal Server Error
**Cause:** BC server error
**Solution:** Check BC Event Log and verify DataAuditFields property exists

#### Error 4: Empty Response
**Cause:** No records match filter
**Solution:** Test without filter: `?$top=10`

### Flow Error Handling

Add **Scope** actions:
```
Scope: Try
  └─ All actions

Scope: Catch (Run after Try fails)
  └─ Send email notification
  └─ Log error to SharePoint
```

---

## 6. Testing Guide

### Step 1: Test BC API
**Tool:** Postman or Browser
```
GET https://[BC-URL]/api/emconsulting/paymentrequest/v1.0/companies([ID])/paymentRequestStatuses?$top=5
Authorization: Bearer [TOKEN]
```

### Step 2: Test Power Automate Flow
1. Create test flow with manual trigger
2. Add HTTP action to BC API
3. Run and check output

### Step 3: End-to-End Test
1. Create Payment Request in BC
2. Change Status (Open → Approved)
3. Verify status updated in DH system

---

## 7. Monitoring

### Power Automate Dashboard
- Flow runs history
- Success/Failure rate
- Average execution time

### Recommended Alerts
- Email notification when flow fails 3+ times
- Daily summary report

---

## 8. Quick Reference

### Key URLs
```
# Get Companies
GET https://[BC-URL]/api/v2.0/companies

# Get Payment Request Statuses
GET https://[BC-URL]/api/emconsulting/paymentrequest/v1.0/companies([ID])/paymentRequestStatuses

# Filter by Date
?$filter=lastModifiedDateTime gt 2026-02-03T10:00:00Z

# Filter by DH No
?$filter=dhPaymentStatementNo eq 'DH-2026-001'
```

### Status Mapping
```
0 = Open
1 = Pending Approval
2 = Budget Checked
3 = Accountant Checked
4 = Approved
5 = Cancelled
```

---

**Document Version:** 1.0
**Last Updated:** 2026-02-03
**Author:** EM Consulting BC Team
