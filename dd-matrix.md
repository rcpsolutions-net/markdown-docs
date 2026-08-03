> **References:**
> * [Data Dictionary Table](https://docs.google.com/spreadsheets/d/1-BpdPYj2i_T5glycmst6PXGrwi9SEdC90YEX_szBrL8/edit?gid=1313502510#gid=1313502510)
> * [Bullhorn Data Replication Schema Layout](https://help.bullhorn.com/article/Data-Replication-schema-layout)

Based on the referenced documentation, below are the tables containing information regarding employee pay methods, pay information, and related audit logs.

---

### 1. Employee Pay Method

* **`DirectDepositAccount`** — Contains specific details regarding employee/candidate direct deposit configurations, including bank name, account number, routing numbers (`transitNumber`, `institutionNumber`), `allocationMethod`, and `paymentOrder`.
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
