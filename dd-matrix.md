***What's on the data mirror?***

> **References:**
> * [Data Dictionary Table](https://docs.google.com/spreadsheets/d/1-BpdPYj2i_T5glycmst6PXGrwi9SEdC90YEX_szBrL8/edit?gid=1313502510#gid=1313502510)
> * [Bullhorn Data Replication Schema Layout](https://help.bullhorn.com/article/Data-Replication-schema-layout)

Based on the referenced documentation, below are the tables containing information regarding employee pay methods, pay information, and related audit logs. 
 
Note: Table names are what Bullhorn API call Entities (and are usually aliased into something shorter). 
      *We need to be sure we can access that entity through the API. We cannot access them all (by design)*

---

### 1. Employee Pay Method

* **`DirectDepositAccount`** — Contains specific details regarding employee/candidate direct deposit configurations, including bank name, account number, routing numbers (`transitNumber`, `institutionNumber`), `allocationMethod`, and `paymentOrder`.
  
* #### dbo.DirectDepositAccount - Schema Reference
* #### Columns (18 total)

| # | Column | Type | Nullable | Default | Notes |
|---|--------|------|----------|---------|-------|
| 1 | `amount` | `money(19,4)` | YES | — | Fixed dollar amount for this deposit split |
| 2 | `transitNumber` | `nvarchar(50)` | YES | — | Bank routing/transit number |
| 3 | `bankName` | `nvarchar(200)` | YES | — | Human-readable bank name |
| 4 | `directDepositAccountTypeLookupID` | `int` | YES | — | FK → `DirectDepositAccountTypeLookup` (e.g. Checking/Savings) |
| 5 | `accountNumber` | `nvarchar(MAX)` | YES | — | Bank account number (MAX — likely encrypted/long) |
| 6 | `dateAdded` | `datetime2` | YES | — | Record creation timestamp |
| 7 | `candidateID` | `int` | YES | — | FK → `Candidate.candidateID` (the employee/worker) |
| 8 | `percentValue` | `money(19,4)` | YES | — | Percentage-based allocation (used when `allocationMethod` = percent) |
| 9 | `dateLastModified` | `datetime2` | YES | — | Last update timestamp |
| 10 | `institutionNumber` | `nvarchar(50)` | YES | — | Canadian bank institution number (3-digit) |
| 11 | `paymentOrder` | `int` | YES | — | Priority order when multiple accounts exist |
| 12 | `directDepositAccountID` | `int` | **NOT NULL** | — | **Primary Key** — clustered |
| 13 | `remainder` | `bit` | YES | — | If `1`, receives whatever is left after other splits |
| 14 | `currencyUnitID` | `int` | YES | — | FK → `CurrencyUnit` (USD, CAD, etc.) |
| 15 | `isDeleted` | `bit` | YES | `0` | **Soft-delete flag** — set to `1` instead of physical delete |
| 16 | `dateLastSync` | `datetime2` | YES | — | Last sync timestamp from Bullhorn source system |
| 17 | `allocationMethod` | `nvarchar(MAX)` | YES | — | Split calculation method (`"Percent"`, `"Amount"`, `"Remainder"`) |
| 18 | `deletedByUserID` | `int` | YES | — | FK → `CorporateUser.corporateUserID` — who soft-deleted this record |
 ---

 #### Primary Key

| Name | Type | Column |
|------|------|--------|
| `PK_DirectDepositAccount` | CLUSTERED | `directDepositAccountID` |

---

#### - Indexes

| Index Name | Columns | Unique |
|------------|---------|--------|
| `IX_DirectDepositAccount_candidateID` | `candidateID` | No |
| `IX_DirectDepositAccount_currencyUnitID` | `currencyUnitID` | No |
| `IX_DirectDepositAccount_dateAdded` | `dateAdded` | No |
| `IX_DirectDepositAccount_dateLastSync` | `dateLastSync` | No |
| `IX_DirectDepositAccount_deletedByUserID` | `deletedByUserID` | No |
| `IX_DirectDepositAccount_directDepositAccountTypeLookupID` | `directDepositAccountTypeLookupID` | No |
| `IX_DirectDepositAccount_isDeleted` | `isDeleted` | No |
| `IX_DirectDepositAccount_Merge_candidateID` | `directDepositAccountID, candidateID` | No |

---

#### - Foreign Keys

| FK Name | Column | References |
|---------|--------|------------|
| `FK_DirectDepositAccount_candidateID` | `candidateID` | `Candidate.candidateID` |
| `FK_DirectDepositAccount_currencyUnitID` | `currencyUnitID` | `CurrencyUnit.currencyUnitID` |
| `FK_DirectDepositAccount_directDepositAccountTypeLookupID` | `directDepositAccountTypeLookupID` | `DirectDepositAccountTypeLookup.directDepositAccountTypeLookupID` |
| `FK_DirectDepositAccount_deletedByUserID` | `deletedByUserID` | `CorporateUser.corporateUserID` |

---

#### - Triggers

**None.** No triggers are defined on this table.

---

#### - Soft-Delete Pattern

Records are never physically deleted. Instead:

- **`isDeleted`** (`bit`, default `0`) — flip to `1` to logically delete; always filter `WHERE isDeleted = 0` for active records
- **`deletedByUserID`** (`int`) — audit trail recording which `CorporateUser` performed the logical delete
- No dedicated `deletedAt` timestamp column; infer deletion time from `dateLastModified` (the last write before `isDeleted` flipped to `1`)
  
* **`DirectDepositAccountTypeLookup`** — A lookup table defining the types of direct deposit accounts, including a flag indicating whether the method is a pay card (`isPayCard`).

### 2. Pay Information

* **`EmployeePay`** — Tracks detailed pay information per entry, including `amount`, `unitRate`, `hoursWorked`, `hoursUnits`, and the associated `earnCodeName`.
* **`PayCheck`** — Contains summary information for issued paychecks, including `grossPay`, `netPay`, `taxAmount`, `employeeTotalDeduction`, `payDate`, and `payPeriod` details.
* **`PayMaster` & `PayMasterTransaction`** — Manage the core financial transactions for payroll, recording details such as `rate`, `quantity`, `amount`, `payPeriodEndDate`, and transaction status.
* **`PayableCharge`** — Tracks charges that are ready to be paid out, including fields for `payGroup` and payroll readiness overrides.
* **`PayBillSetting`** — Stores configuration rules for payroll and billing, such as `overtimePayMultiplier` and `doubletimePayMultiplier`.
* **`PayBillCycle`** — Defines the processing schedules and cycles for evaluating pay and bill data.
* **`PayExportBatch` / `PayExportBatchExternal` / `PayExportBatchPayableCharge`** — Manage groups of payroll records bundled together for export to external payroll providers.

### 3. Logs and History

* **`PayServiceExportAuditLog`** — Logs the results of payroll exports to external systems, capturing fields like `isSuccess`, `responseStatusCode`, and `rawResponseMessage`.
* **`EditHistoryPayMasterTransaction` & `EditHistoryFieldChangePayMasterTransaction`** — Audit logs tracking any changes, modifications, or updates made to pay master transactions.
* **`EditHistoryPayBillSetting` & `EditHistoryFieldChangePayBillSetting`** — Audit logs tracking historical changes to organizational pay and bill settings.
* **`EditHistoryPayrollExportConfig` & `EditHistoryFieldChangePayrollExportConfig`** — Logs changes made to the configurations used for exporting payroll data.
* **`TimesheetEntryApprovalStatusLogEntry`** — Tracks the approval history and status changes of timesheet entries, which serve as the primary source log for generating pay.
