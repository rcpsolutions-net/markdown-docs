# RCP GreenShades Sync Service
> AI agent? Load [README.AI.md](./README.AI.md) for a token-efficient version of this file.

This service keeps our internal database automatically up to date with data from GreenShades (our payroll system). Instead of someone manually running a report and importing it every week, this service listens for activity from GreenShades and updates our records on its own.

---

## What Problem Does This Solve?

Previously, keeping our database current with GreenShades required:

1. Manually running a report in GreenShades
2. Downloading the result as a JSON file
3. Running an import script by hand
4. Hoping nothing was missed between runs

This service replaces all of that. It reacts to GreenShades events the moment they happen, and runs scheduled checks to catch anything that slipped through.

---

## How It Works — Big Picture

```mermaid
flowchart TD
    GS[GreenShades Payroll System]
    WR[Webhook Receiver Service]
    DB[(MongoDB Database)]
    SYNC[GreenShades Sync Service\nThis Service]
    GSAPI[GreenShades API]

    GS -->|"Sends a notification when\nsomething changes"| WR
    WR -->|"Saves the notification\nto the database"| DB
    DB -->|"Sync service notices\nthe new notification"| SYNC
    SYNC -->|"Calls GreenShades API\nfor full record data"| GSAPI
    GSAPI -->|"Returns the requested data"| SYNC
    SYNC -->|"Saves records\nto the database"| DB
```

---

## When the Service Starts Up

Every time this service starts (or restarts), it checks whether any notifications were missed while it was offline, and processes them before doing anything else.

```mermaid
flowchart TD
    A([Service Starts]) --> B[Connect to MongoDB]
    B --> C{Any unprocessed\nnotifications in\nthe database?}
    C -->|Yes| D[Process each one\nin order]
    D --> E[Start listening for\nnew notifications]
    C -->|No| E
    E --> F[Start scheduled jobs]
    F --> G([Service is running])
```

---

## Real-Time Event Handling

This is the core of the service. GreenShades sends a small notification — called a **webhook** — when key payroll events occur. The service handles three event types:

| Event | What it does |
|---|---|
| `payRun.payRecords.completed` | Fetches all paystubs for the completed pay run and upserts them |
| `payRun.payRecord.voided` | Marks the individual pay record as `voided: true` in the database |
| `employee.updated` | Fetches the updated employee record and upserts it |

```mermaid
flowchart TD
    A([GreenShades fires\nan event]) --> B[Webhook Receiver saves the\nnotification to MongoDB]
    B --> C[Sync service detects\nthe new notification\nvia Change Stream]
    C --> D{Event type?}
    D -->|payRun.payRecords.completed| E[Fetch all paystubs for\nthe pay run from API\nand bulk upsert]
    D -->|payRun.payRecord.voided| F[Mark the specific\npay record voided]
    D -->|employee.updated| G[Fetch updated employee\nfrom API and upsert]
    D -->|Unknown| H[Skip — log and move on]
    E --> I[Mark notification\nas processed ✓]
    F --> I
    G --> I
```

> **What is a Change Stream?** It's a feature of MongoDB that lets the service be instantly notified when a new notification document is saved — without having to check the database on a timer.

> **What is a Resume Token?** If the service crashes mid-stream, it saves a bookmark (resume token) in the database so it knows exactly where it left off when it restarts — no events are missed or replayed.

---

## Scheduled Jobs

Two jobs run automatically on a schedule (configurable via environment variables).

### 1. Time-Off Balance Sync

GreenShades doesn't send webhooks for sick leave balance changes, so we pull those on a schedule. With ~55k records returned by the API (including past employees), the sync is done efficiently: one bulk read of all existing balances, delta computation in memory, then batched bulk writes — no per-record transactions.

```mermaid
flowchart TD
    A([Scheduled]) --> B[Fetch all employee\ntime-off balances from API]
    B --> C[Bulk load all existing\nbalance docs from MongoDB]
    C --> D[Compute deltas in memory\nfor all records]
    D --> E{Delta = 0?}
    E -->|No change| F[Skip]
    E -->|Balance went up| G[Queue as Accrual]
    E -->|Balance went down| H[Queue as Usage]
    G --> I[Bulk upsert balances\nin batches of 500]
    H --> I
    I --> J[Bulk insert audit logs\nin batches of 500]
    J --> K([Done])
```

Every change is written to an audit log (`gs_accrual_logs`) so there's a full history of when hours were earned and used.

### 2. Reconciliation

This job acts as a safety net. Rather than comparing pay run lists, it directly fetches all pay records from GreenShades for a rolling date window (5 days back through 3 days ahead) and bulk upserts them — catching any gaps or stale records in one pass.

```mermaid
flowchart TD
    A([Scheduled]) --> B["Fetch all pay records from API\nfor date window:\n(today − 5 days) → (today + 3 days)"]
    B --> C[Bulk upsert all records\nin batches of 500]
    C --> D[Log progress per batch]
    D --> E([Done])
```

---

## Health Check

The service exposes two simple URLs to confirm it's running:

| URL | What it shows |
|-----|--------------|
| `/health` | Is the service alive? When did it last process something? |
| `/status` | How many notifications are waiting to be processed? What are the cron schedules? |

---

## Error Handling

```mermaid
flowchart TD
    A[Processing a notification] --> B{Error occurs}
    B --> C[Record the error message\non the notification document]
    C --> D[Leave it marked as\nnot processed]
    D --> E[Continue to next notification]
    E --> F[On next service restart:\nBackfill job retries\nall unprocessed notifications]
```

Failed notifications are never silently dropped. They stay in the database with their error message attached, and will be retried automatically the next time the service restarts.

---

## Data Flow Summary

```mermaid
flowchart LR
    subgraph GreenShades
        PR[Pay Run Completed]
        PV[Pay Record Voided]
        EU[Employee Updated]
        API[REST API]
    end

    subgraph MongoDB
        WH[(incoming-webhooks)]
        PD[(gs_pay_details)]
        AB[(gs_accrual_balances)]
        AL[(gs_accrual_logs)]
        EMP[(gs_employees)]
        DD[(gs_direct_deposits)]
        DDL[(gs_direct_deposit_change_log)]
        SS[(gs_sync_state)]
    end

    subgraph This Service
        WW[Webhook Watcher]
        BF[Backfill Job]
        TOS[Time-Off Sync]
        REC[Reconciliation]
        DDD[Direct Deposit Discovery]
    end

    PR -->|webhook| WH
    PV -->|webhook| WH
    EU -->|webhook| WH
    WH -->|change stream| WW
    WW -->|fetch pay records| API
    WW -->|fetch employee + custom fields| API
    WW -->|fetch direct deposits| API
    API --> PD
    API --> EMP
    API --> DD
    WW -->|changelog on change| DDL
    WW -->|mark voided| PD

    BF -->|on startup| WH
    BF -->|fetch details| API

    TOS -->|scheduled| API
    TOS --> AB
    TOS --> AL

    REC -->|scheduled, by date range| API
    REC --> PD

    DDD -->|nightly, all employees| API
    DDD --> DD
    DDD -->|changelog on change| DDL
    DDD -->|checkpoint| SS
    WW --> SS
```

---

## Webhook Event Processing — Detailed Flow

Both the real-time change stream watcher and the startup backfill job run the same event handlers. The difference is how they discover events to process.

```mermaid
flowchart TD
    GS([GreenShades fires event]) -->|HTTP POST| WHREC[rcp-webhook-receiver\nsaves to incoming-webhooks]
    SVC([Service starts]) --> BKQL[runBackfill — query all\nunprocessed incoming-webhooks]

    WHREC -->|MongoDB change stream\ninsert on incoming-webhooks| WATCHER[webhookWatcher]

    WATCHER --> DISP{eventName?}
    BKQL -->|for each webhook\noldest first| DISP

    DISP -->|payRun.payRecord.voided| VOID["Set voided=true on pay record\nin gs_pay_details\n→ mark webhook processed ✓"]
    DISP -->|unknown| SKIP[Log warning and skip]
    DISP -->|payRun.payRecords.completed\nor payRun.completed| WAIT

    WAIT["Wait 1s — let GS finalize pay run data"] --> APIFETCH
    APIFETCH["GS API: getPayRunPayRecords\n(retry ×3, 5s between attempts)"] --> NOREC

    NOREC{Records\nreturned?}
    NOREC -->|No after 3 tries\nor 400 error| MARK1([Mark webhook processed ✓])
    NOREC -->|Yes| SPLIT

    SPLIT["Split into batches of 100"] --> BW1
    BW1["bulkWrite → gs_pay_details\n(upsert by record id)"] --> PERAS

    PERAS["Per employee in batch\nPromise.allSettled(batch.map(…))"]
    PERAS --> EE
    PERAS --> EDD

    subgraph EE [ensureEmployee]
        EE_CHK{Employee in\ngs_employees?}
        EE_CHK -->|Yes| CFONLY["GS API: getEmployeeCustomFields\n→ update gs_employees"]
        EE_CHK -->|No| EEFULL["GS API: getEmployee + getEmployeeCustomFields\nin parallel → upsert gs_employees"]
    end

    subgraph EDD [ensureDirectDeposit]
        EDDAPI["GS API: getDirectDeposit"] --> EDDDIFF
        EDDDIFF["Load existing records from gs_direct_deposits\nDiff each by ordinal"] --> EDDBW
        EDDBW["bulkWrite → gs_direct_deposits"] --> CHANGED
        CHANGED{Any field\nchanged?}
        CHANGED -->|Yes| CHGLOG["insertMany → gs_direct_deposit_change_log\n(previous + current snapshots\nprevious=null for new records)"]
        CHANGED -->|No| NOCHANGE[No changelog entry]
    end

    SPLIT -->|all batches complete| MARK2([Mark webhook processed ✓])
```

---

## Nightly Direct Deposit Discovery — Detailed Flow

Runs every night against all 80,000+ employees to catch direct deposit changes that were not triggered by a webhook. Rate-limited to 20 requests/second, resumable via checkpoint, and protected by a circuit breaker.

```mermaid
flowchart TD
    CRON([Nightly cron ~noon]) --> LOAD
    LOAD["Load all employee IDs from gs_employees\nSort for stable ordering"] --> CKP

    CKP{SyncState checkpoint\nkey: directDeposit:discovery\nexists?}
    CKP -->|Yes — previous run was interrupted| RESUME["Resume: indexOf(lastId) + 1"]
    CKP -->|No| FRESH[Start from index 0]

    RESUME --> SLICE
    FRESH --> SLICE
    SLICE["Take next batch of 20 employee IDs"] --> STARTTIME

    STARTTIME[Record batch start time] --> CALLS
    CALLS["Promise.allSettled × 20\nensureDirectDeposit for each employee\n(GS API call per employee)"] --> SLEEP

    SLEEP["Sleep remaining ms of 1-second window\nmath.max(0, 1000 − elapsed)"] --> CB

    CB{10 consecutive\nfailures?}
    CB -->|Yes — API likely down| ABORT["Stop job\nLeave checkpoint at previous batch\n→ failing batch retried on next run"]
    CB -->|No| SAVECKP

    SAVECKP["Save checkpoint\n(last employee ID in this batch)\n→ gs_sync_state"] --> MORE

    MORE{More employees\nremaining?}
    MORE -->|Yes| SLICE
    MORE -->|No — clean finish| CLEARCKP

    CLEARCKP["Delete SyncState checkpoint\nNext nightly run starts fresh"] --> DONE([Done])
```
