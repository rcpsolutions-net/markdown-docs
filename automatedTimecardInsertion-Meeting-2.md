# Timecard Project — Demo #2 Meeting Summary

*Prompted and edited by Lawrence Ham*

## Meeting Details

- **Time:** ~27 minutes (per transcript length).
- **Attendees:**
  - **Lawrence Ham** — presenter (dev lead)
  - **Daniel Russell** — facilitator / product vision
  - **Sandra Vargas**, **Elizabeth Garcia**, **Yvette Rodriguez** (limited participation) — payroll / user stakeholders
  - **David Bolde** — tech / API info provider
- **Purpose:** demonstrate the AI timesheet-processing system and gather feedback on workflow and Bullhorn/BTE integration.

## Key Topics

1. **Demo walkthrough.** Incoming timesheet PDFs/scans (11 pages demoed). Core process: AI + OCR + SQL matching for the users on the timecards. Page-by-page processing; as each page completes, more are dispatched. Demonstrated extraction of client names (e.g. "warehousing"), matching to Bullhorn placements via SQL (placement IDs, start/end dates). **AI reconciliation** fixes discrepancies (e.g. `lunchOut` after `timeOut`) *without altering* the original extracted numbers — focus is pure extraction + internal reconciliation. Pay-code logic correctly used "REG" for all entries under 40 hours.

2. **Current manual process.** Lawrence can't easily visualize the manual breakdown into Bullhorn (no front-end access). Sandra/Yvette: manual calculations or a timesheet calculator; for California clients, punches are entered manually into the VMS (e.g. HD Supply) or BTE. Those systems *should* compute hours, but manual sheets still require manual calculation.

3. **Workflow & queue.** Hours mostly arrive in bulk by client (client folders exist), with exceptions. **Proposed workflow:** automation pulls PDFs from the email inbox; users see a **work queue** of outstanding timecards. Queue: most-recent on top, select/review/apply rules, a "finalize"/"approve" button. **Multi-user / audit:** shared queue so users see what others are doing; audit log of who validated/uploaded; user-based views/filtering.

4. **Bullhorn/BTE integration.** BTE = a section within Bullhorn for manual entries, with its own API. **Decision:** approved/finalized time is pushed to **BTE via its API**, not to Bullhorn's time-card creation API (which would bypass BTE). The approved PDF is also pushed to client folders for reference.

5. **AI generates overtime entries (critical).** The software **generates OT entries** for employees over 40 hours, even if not on the timesheet (e.g. 50 hrs → 5-hr OT entry). Lawrence can pull payroll rules from Bullhorn but wants payroll to confirm.

6. **Sandra's feedback.** Page navigation must show client name, employee name, file name. **Must** be able to search/filter by **Client ID or Client Name** (responsibilities are client-list based — "I only want to see *my* APL timesheets"). Reiterated the need for data to flow into BTE after approval.

## Action Items

- **Lawrence:** obtain BTE API docs (from David Bolde); review/confirm OT/DT payroll rules with payroll; implement user-based queues/views with client/employee/filename display + search (Client ID, Client Name).
- **Daniel:** schedule a follow-up meeting.
- **All:** further define OT and DT business rules for AI generation.

## Open Questions

- Exact rules for **DT** calculation/generation (preamble mentions DT, but Step 10 covers only OT).
- How should generated OT/DT entries handle `timeIn`/`timeOut` in the JSON output? (`null` vs. copy the base entry's punches.)
- Allocation strategy for generated OT/DT hours across multiple days when unspecified (e.g. "latest day with hours").
