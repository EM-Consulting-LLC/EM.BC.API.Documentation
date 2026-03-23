# Employee - Power Automate API Documentation

## 1) Endpoint
Common operations:
- **Find Records**
- **Get Record**
- **Update Record**
- **Create Record**

>Notes:
>- `systemId` is a GUID and is the OData key. Can be stored in String/Text field.
>- `Environment`: use your target BC environment (Sandbox or Production).
>- `Company`: use the target company name and companyId.
>- `API category`: EMC/digitalIntegration/v1.0
>- `Table name(entitySet)`: employees
>- `Entity name`: employee

Power Automate notes:
- Flow Trigger : *Modified*.
- Flow Filter : `if(equals(formatDateTime(outputs('Get_record_(V3)')?['body/EmploymentDate'], 'yyyy-MM-dd'), '0001-01-01'), true, false)`. Only employees with non null  employment dates.

---

## 2) Data Model

### 2.1 Editable Fields

**Personal Information** — PATCH: Update Record (Personal Info)

| API Field | Caption |
|---|---|
| FamilyName | Ургийн овог |
| LastName | Эцэг Эхийн нэр |
| FirstName | Ажилтны нэр |
| BirthDate | Төрсөн огноо |
| BloodType | Цусны бүлэг |
| CompanyEmail | Ажлын имэйл-1 |
| E_Mail | Хувийн имэйл |
| Phone2 | Утасны дугаар-1 |
| EmergencyContactName | Яаралтай үед холбоо барих хүний нэр-1 |
| MobilePhoneNo | Яаралтай үед холбоо барих хүний утас-1 |
| Relationship | Яаралтай үед холбоо барих хүн хэн болох-1 |
| EmergencyContactName2 | Яаралтай үед холбоо барих хүний нэр-2 |
| Phone3 | Яаралтай үед холбоо барих хүний утас-2 |
| Relationship_2 | Яаралтай үед холбоо барих хүн хэн болох-2 |
| Social_Media | Facebook |
| MaritalStatus | Гэрлэлтийн байдал |

**Address** — PATCH: Update Record (Address)

| API Field | Caption |
|---|---|
| AddressKhorooBagUpdate | Хороо шинэчлэх |
| AddressCityProvinceDesc | Хот/Аймаг код |
| AddressDistrSoumDesc | Сум/Дүүрэг код |
| CitizenshipCityProvinceDesc | Харьяалал Хот/Аймаг Код |
| CitizenshipDistrSoumDesc | Харьяалал Сум/Дүүрэг Код |
| BirthshipCityProvinceDesc | Төрсөн хот/аймаг Код |
| BirthshipDistrSoumDesc | Төрсөн дүүрэг/сум Код |
| Neighborhood | Хороолол/Хотхон |
| Apartment | Байр |
| ApartmentNo | Тоот |
| BirthCountryCode | Төрсөн улс код |

All other fields are read-only.

### 2.2 Employee API fields

| API field | BC field | Type | Editable | Caption |
|---|---|---|:---:|---|
| systemId | SystemId | GUID | No | *(OData key)* |
| Status | Status | Enum | No | Статус |
| No | "No." | Code[20] | No | EMPID |
| FamilyName | "Family Name" | Text | Yes | Ургийн овог |
| LastName | "Last Name" | Text | Yes | Эцэг Эхийн нэр |
| FirstName | "First Name" | Text | Yes | Ажилтны нэр |
| PositionCode | Position_Cd | Code[10] | No | PositionCode |
| Position | "Position Description (MN)" | Text | No | Албан тушаал |
| Grade | Grade | Code | No | Зэрэг |
| DivisionMN | "Division Description (MN)" | Text | No | Газар |
| DepartmentCode | "Department Description (MN)" | Text | No | Алба |
| LastNameEng | "Last Name (EN)" | Text | No | Last Name |
| FirstNameEng | "First Name (EN)" | Text | No | First Name |
| PositionEng | "Position Description (EN)" | Text | No | Official Position |
| DivisionDescriptionENG | "Division Description (EN)" | Text | No | Department I |
| DepartmentDescriptionENG | "Department Description (EN)" | Text | No | Department II |
| EmploymentDate | "Employment Date" | Date | No | Анх компанид ажилд орсон огноо |
| LastEmploymentDate | "Last Employment Date" | Date | No | Сүүлд компанид ажилд орсон огноо |
| SHIPeriodHistory_Normal | "SHI Period History (Normal)" | Decimal | No | Компанид ажиллахаас өмнө хэвийн нөхцөлд ажилласан сар |
| SHIPeriodHistoryAbnormal | "SHI Period History (Abnormal)" | Decimal | No | Компанид ажиллахаас өмнө хэвийн бус нөхцөлд ажилласан сар |
| AnnualLeaveStartDate | "AL Start Date" | Date | No | Ээлжийн амралтын эрх үүсэх огноо |
| MCSGEmploymentDate | "MCSG Employment date" | Date | No | Группт ажилд орсон огноо |
| BirthDate | "Birth Date" | Date | Yes | Төрсөн огноо |
| Gender | Gender | Enum | No | Хүйс |
| BloodType | "Blood Type" | Enum | Yes | Цусны бүлэг |
| RegistrationNumber | "Registration No." | Text | No | Регистрийн дугаар |
| BirthCountry | getBirthCountryCode() | Text | No | Төрсөн улс |
| BirthCityProvince | getBirthCityCode() | Text | No | Төрсөн хот/аймаг |
| BirthDistrictSoum | getBirthSoumCode() | Text | No | Төрсөн дүүрэг/сум |
| CompanyEmail | "Company E-Mail" | Text | Yes | Ажлын имэйл-1 |
| E_Mail | "E-Mail" | Text | Yes | Хувийн имэйл |
| Phone | "Phone No." | Text | No | Утасны дугаар /Deprecated/ |
| Phone2 | "Mobile Phone No." | Text | Yes | Утасны дугаар-1 |
| EmergencyContactName | "Emergency contact name" | Text | Yes | Яаралтай үед холбоо барих хүний нэр-1 |
| MobilePhoneNo | Phone | Text | Yes | Яаралтай үед холбоо барих хүний утас-1 |
| Relationship | "Emergency Contact Relationship" | Text | Yes | Яаралтай үед холбоо барих хүн хэн болох-1 |
| EmergencyContactName2 | "Emergency contact name 2" | Text | Yes | Яаралтай үед холбоо барих хүний нэр-2 |
| Phone3 | "Phone 2" | Text | Yes | Яаралтай үед холбоо барих хүний утас-2 |
| Relationship_2 | "EmergencyContactRelationship 2" | Text | Yes | Яаралтай үед холбоо барих хүн хэн болох-2 |
| Social_Media | "Social Media" | Text | Yes | Facebook |
| Social_media_2 | "Social media 2" | Text | No | LinkedIn |
| CitizenshipCityProvince | getCitizenshipCode() | Text | No | Харьяалал Хот/Аймаг |
| CitizenshipDistrictSoum | getCitizenshipSoumCode() | Text | No | Харьяалал сум дүүрэг |
| AddressCityProvince | getAddressCityCode() | Text | No | Хот/Аймаг |
| AddressDistrictSoum | getAddressSoumCode() | Text | No | Сум/Дүүрэг |
| AddressKhorooBag | Format("Address: Khoroo, Bag") | Text | No | Хороо |
| AddressKhorooBagUpdate | "Address: Khoroo, Bag" | Code | Yes | Хороо шинэчлэх |
| AddressKhorooBagDesc | GetKhoroo() | Text | No | Хороо description |
| AddressNeighborApartment | "Address: Neighbor, apartment" | Text | No | хороолол/Хотхон/Байр/Тоот |
| AddressGoogleMapLink | "Address: Google map link" | Text | No | Гэрийн хаяг link |
| MaritalStatus | "Marital Status" | Enum | Yes | Гэрлэлтийн байдал |
| TINCode | "TIN Code" | Text | No | Иргэний бүртгэлийн дугаар/ТИН дугаар/ |
| KPIFrequency | "KPI Frequency" | Enum | No | KPI Frequency |
| WorkingCondition | "Working Condition" | Enum | No | Ажлын байрны нөхцөл |
| EmployeeType | "Employee Type" | Enum | No | Ажлын байрны төрөл |
| Assistant | Assistant | Boolean | No | Дагалдан эсэх |
| ProbStartDate | "Prob Start Date" | Date | No | Туршилтын хугацаа эхлэх огноо |
| ProbEndDate | "Prob End Date" | Date | No | Туршилтын хугацаа дуусах огноо |
| AddressCityProvinceDesc | "Address: City, Province" | Code | Yes | Хот/Аймаг код |
| AddressDistrSoumDesc | "Address: District, Soum" | Code | Yes | Сум/Дүүрэг код |
| CitizenshipCityProvinceDesc | "Citizenship: City, Province" | Code | Yes | Харьяалал Хот/Аймаг Код |
| CitizenshipDistrSoumDesc | "Citizenship: District, Soum" | Code | Yes | Харьяалал Сум/Дүүрэг Код |
| BirthshipCityProvinceDesc | "Birth City, province" | Code | Yes | Төрсөн хот/аймаг Код |
| BirthshipDistrSoumDesc | "Birth District, Soum" | Code | Yes | Төрсөн дүүрэг/сум Код |
| BirthCountryDesc | "BirthCountry Desc" | Text | No | - |
| Neighborhood | "Address: Neighbor, apartment" | Text | Yes | Хороолол/Хотхон |
| Apartment | Apartment | Text | Yes | Байр |
| ApartmentNo | "Apartment No." | Text | Yes | Тоот |
| BankCode | "PR Bank Code" | Code[20] | No | Банкны BC код |
| bankDesc | GetBankDescription("PR Bank Code") | Text | No | Банкны нэр |
| bankDesc2 | GetBankDescription("Bank Code2") | Text | No | Банкны нэр |
| bankDesc3 | GetBankDescription("Bank Code3") | Text | No | Банкны нэр |
| BankAccountNo | "Bank Account No." | Text | No | Дансны дугаар |
| BankCode2 | "Bank Code2" | Code[20] | No | Банкны BC код 1 |
| BankCode3 | "Bank Code3" | Code[20] | No | Банкны BC код 2 |
| BankAccountNo2 | "Bank Account No. 2" | Text | No | Дансны дугаар 2 |
| BankAccountNo3 | "Bank Account No. 3" | Text | No | Дансны дугаар 3 /benefit/ |
| BirthCountryCode | "Birth Country" | Code | Yes | Төрсөн улс код |
| PositionSegment | GetPosSegment(Position_Cd) | Text[20] | No | Position Segment |
| TerminationDate | GetTerminationDate() | Date | No | Ажлааас гарсан огноо |

---

## 3) Request Examples

### 3.1 Find Records (Power Automate HTTP)

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/employees?$select=SystemId,No,FirstName,LastName,Status,CompanyEmail,Phone2&$filter=Status eq 'Active'&$top=1000

Response shape (example):
{
  "value": [
    {
      "systemId": "8d0f6e61-0c4c-ef11-a3f7-6045bd000001",
      "No": "EMP001",
      "FirstName": "Bat",
      "LastName": "Erdene",
      "Status": "Active",
      "CompanyEmail": "bat.erdene@company.mn",
      "Phone2": "+97699112233"
    }
  ],
  "@odata.nextLink": "..."
}

### 3.2 Get Record

Request:
GET /api/EMC/digitalIntegration/v1.0/companies({companyId})/employees({systemId})

### 3.3 Update Record

Request:
PATCH /api/EMC/digitalIntegration/v1.0/companies({companyId})/employees({systemId})

Body example:
{
  "FirstName": "Bat",
  "LastName": "Erdene",
  "CompanyEmail": "bat.erdene@company.mn",
  "Phone2": "+97699112233",
  "AddressKhorooBagUpdate": "1101"
}

### 3.4 Create Record (optional policy)

Request:
POST /api/EMC/digitalIntegration/v1.0/companies({companyId})/employees

Body example:
{
  "No": "EMP999",
  "FirstName": "New",
  "LastName": "Employee"
}

Use create only if your BC process explicitly allows employee creation via API.

---

## 4) Business Rules and Validations (Power Automate requirements)

### 4.1 Flow-level validations to enforce

1. systemId for updates
- Must be present for PATCH.

2. Identity fields
- No should be present for cross-system matching when available.

3. Date fields
- Validate date format before calling API.
- Avoid sending future dates unless explicitly allowed by business policy.

4. Contact fields
- Validate email format for CompanyEmail and E_Mail.
- Validate phone format for Phone2.

5. Read-only/computed fields
- Do not send system-managed and computed fields in PATCH or POST payloads.

### 4.2 Sync behavior rules

- If destination uses upsert, key priority should be:
  - systemId (technical)
  - No (business fallback)
- Use $select to reduce payload size and API pressure.
- Use @odata.nextLink loop for complete dataset reads.

### 4.3 Avoiding Infinite Loops — Create-Triggered Writeback Pattern

#### Problem
The outbound flow (BC → external system) is triggered on **Modified**. If the writeback flow (external system → BC) also runs on **Modified**, updating BC triggers the outbound flow again — creating an infinite loop:

```
BC modified → outbound flow → update external system
→ external system modified → writeback flow → PATCH BC
→ BC modified → outbound flow → ... (loop)
```

#### Solution
Split writeback into **two separate flows**, each triggered on **Created** (not Modified) on the external system list. A newly created row indicates a deliberate user submission, not a system sync-write. This breaks the cycle.

---

#### Flow A — Personal Info Writeback (Created trigger)

**Trigger:** When an item is **created** in the Personal Info submission list

**Fields written to BC via PATCH:**

| External List Column | API Field | Caption |
|---|---|---|
| field_2 | FamilyName | Ургийн овог |
| field_3 | LastName | Эцэг Эхийн нэр |
| field_4 | FirstName | Ажилтны нэр |
| field_6 | BirthDate | Төрсөн огноо |
| field_7 | BloodType | Цусны бүлэг |
| field_18 | CompanyEmail | Ажлын имэйл-1 |
| field_20 | E_Mail | Хувийн имэйл |
| field_21 | Phone2 | Утасны дугаар-1 |
| field_24 | EmergencyContactName | Яаралтай үед холбоо барих хүний нэр-1 |
| field_25 | MobilePhoneNo | Яаралтай үед холбоо барих хүний утас-1 |
| field_23 | Relationship | Яаралтай үед холбоо барих хүн хэн болох-1 |
| field_27 | EmergencyContactName2 | Яаралтай үед холбоо барих хүний нэр-2 |
| field_28 | Phone3 | Яаралтай үед холбоо барих хүний утас-2 |
| field_26 | Relationship_2 | Яаралтай үед холбоо барих хүн хэн болох-2 |
| field_29 | Social_Media | Facebook |
| field_41 | MaritalStatus | Гэрлэлтийн байдал |

**Flow steps:**
1. Trigger: When an item is **created** in Personal Info list
2. Get Record (V3) from BC using `Title` (systemId) to read current BC values
3. PATCH BC employee — for each field: if incoming value is empty, keep existing BC value
4. Terminate — do NOT write back to the submission list (no Modified event fires on it)

**PATCH field expression pattern:**
```
@if(empty(triggerBody()?['field_2']), outputs('Get_record_(V3)')?['body/FamilyName'], triggerBody()?['field_2'])
```

---

#### Flow B — Address Writeback (Created trigger)

**Trigger:** When an item is **created** in the Address submission list

**Fields written to BC via PATCH:**

| External List Column | API Field | Caption |
|---|---|---|
| KhorooNum (variable) | AddressKhorooBagUpdate | Хороо шинэчлэх |
| field_31 | AddressCityProvinceDesc | Хот/Аймаг код |
| field_33 | AddressDistrSoumDesc | Сум/Дүүрэг код |
| field_9 | CitizenshipCityProvinceDesc | Харьяалал Хот/Аймаг Код |
| field_11 | CitizenshipDistrSoumDesc | Харьяалал Сум/Дүүрэг Код |
| field_15 | BirthshipCityProvinceDesc | Төрсөн хот/аймаг Код |
| field_16 | BirthshipDistrSoumDesc | Төрсөн дүүрэг/сум Код |
| field_36 | Neighborhood | Хороолол/Хотхон |
| field_37 | Apartment | Байр |
| field_38 | ApartmentNo | Тоот |
| field_13 | BirthCountryCode | Төрсөн улс код |

**Flow steps:**
1. Trigger: When an item is **created** in Address list
2. Get Record (V3)_1 from BC using `Title` (systemId) to read current BC values
3. Set variable `KhorooNum` from incoming khoroo input
4. PATCH BC employee — for each field: if incoming value is empty, keep existing BC value
5. Terminate — do NOT write back to the submission list

**PATCH field expression pattern:**
```
@if(empty(triggerBody()?['field_31']), outputs('Get_record_(V3)_1')?['body/AddressCityProvinceDesc'], triggerBody()?['field_31'])
```

---

#### Why this is safe

| Flow | Trigger | Writes to | Modifies BC | Fires outbound? |
|---|---|---|:---:|:---:|
| Outbound (BC → External) | BC Modified | External list (Created row) | No | No |
| Flow A — Personal Info | External Created | BC | Yes | Only if outbound filter field changes |
| Flow B — Address | External Created | BC | Yes | Only if outbound filter field changes |

> The outbound flow filter `if(equals(formatDateTime(...EmploymentDate...), '0001-01-01'), true, false)` targets Employment Date — a read-only field never changed by writebacks. This means writeback PATCHes will not re-trigger the outbound flow, fully closing the loop.


---
## 5) Permission Error Handling (Request to EM)

Use this section when a flow fails because the integration account does not have enough Business Central API permission.

### 5.1 Error patterns

- `401 Unauthorized`: token/connection issue (invalid or expired auth)
- `403 Forbidden`: authenticated, but permission is missing for company/dataset/table/action
- `404 Not Found` with correct URL and ID can also indicate missing company/table access policy

### 5.2 Immediate checks in Power Automate

1. Confirm the connection account used by the failed action (`shared_dynamicssmbsaas`).
2. Confirm `bcenvironment`, `company`, `dataset`, and `table` values are correct.
3. Re-run with a simple GET on one known employee to isolate permission vs payload issues.
4. Capture error evidence:
  - Flow run URL
  - Action name (for example `Get_record_(V3)` or `PatchItemV3`)
  - HTTP status code
  - Error message/body
  - Timestamp (UTC)
  - Employee `systemId` / `No` used in request

### 5.3 Request new permission from EM

When checks confirm access issue, submit a permission request to EM support/BC admin.

Required request template:

```
Subject: BC API Permission Request - Employee Integration (Power Automate)

Environment: PRODUCTION
Company ID: 5fc9e579-d069-ef11-a673-000d3ac98ed3
API category (dataset): EMC/digitalIntegration/v1.0
Table (entitySet): employees
Connection account: <service-account-upn>

Required operations:
- Read (GET): Yes
- Update (PATCH): Yes
- Create (POST): <Yes/No>

Failure details:
- Flow name: <flow-name>
- Flow run URL: <url>
- Action: <action-name>
- Status code: <401/403/404>
- Error message: <message>
- Time (UTC): <timestamp>

Business reason:
- Personal Info and Address writeback from Power Automate list to BC Employee API.
```

### 5.4 While waiting for EM approval

- Keep failed items in retry queue (do not drop payload).
- Suspend auto-retry if it can cause throttling/noise.
- Notify business owner that updates are pending permission approval.

### 5.5 After permission is granted

1. Re-authenticate or refresh the Power Automate connection.
2. Test GET on one employee.
3. Test PATCH for one Personal Info field and one Address field.
4. Replay queued failed updates.
5. Close the incident with run ID evidence.

---
## 6) Non-Goals and Explicit Exclusions

The following are out of scope for this API integration contract:

- This document does not guarantee that all 80 fields are editable.
- This integration does not bypass BC permissions or business rules.
- This integration does not guarantee idempotency by default.
- This integration does not define payroll, posting, or approval logic.
- This integration does not define custom conflict resolution for concurrent updates.
- This integration does not guarantee schema stability across future API versions.

Any behavior not explicitly documented should be considered unsupported by external consumers.

---

## 7) Operational Checklist

- Confirm Power Automate connection owner and service account.
- Confirm base URL, environment name, and companyId.
- Validate Find Records with $top=1.
- Validate pagination with @odata.nextLink.
- Validate one PATCH with editable fields only.
- Configure retry policy and error alerting.
- Record mapping for systemId and No in downstream system documentation.
