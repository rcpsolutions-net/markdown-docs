# How Magnit & Bullhorn Handle Time-Keeping (Succint Version)

## The Big Picture

Think of it like this: **Magnit** is where your workers clock in/out. **Bullhorn** is where your recruiters and finance team see everything. The two systems talk to each other automatically so nobody has to re-enter data.

---

## Visual Flow

```mermaid
flowchart LR
    A[👷 Worker] -->|Clocks in/out| B[📱 Magnit VMS]
    B -->|Sends time data| C[🔄 Magnit API]
    C -->|Pushes to Bullhorn| D[🐂 Bullhorn ATS]
    D -->|Imports time| E[💰 Bullhorn Time & Expense]
    E -->|Approves & pays| F[✅ Done!]

    style A fill:#4CAF50,stroke:#2E7D32,color:#fff
    style B fill:#2196F3,stroke:#1565C0,color:#fff
    style C fill:#FF9800,stroke:#E65100,color:#fff
    style D fill:#9C27B0,stroke:#6A1B9A,color:#fff
    style E fill:#F44336,stroke:#C62828,color:#fff
    style F fill:#00BCD4,stroke:#00838F,color:#fff
```

---

## Step-by-Step (Plain English)

| Step | What Happens | Who Does It |
|------|-------------|-------------|
| **1. Clock In** | Worker logs their hours in Magnit (app or web) | 👷 Worker |
| **2. Send** | Magnit packages the time data and sends it through its API | 📱 Magnit |
| **3. Receive** | Bullhorn gets the data automatically — no manual entry | 🐂 Bullhorn |
| **4. Import** | Bullhorn Time & Expense pulls in the hours for payroll | 💰 T&E |
| **5. Approve** | Manager reviews, approves, and the worker gets paid | ✅ Manager |

---

## Key Things to Know

### 🔑 No Double Entry
Workers enter time **once** in Magnit. It shows up in Bullhorn automatically.

### 🔑 Real-Time Updates
New job requests and status changes flow from Magnit to Bullhorn instantly (as of Oct 2026).

### 🔑 Secure
All data moves through authenticated API calls (OAuth 2.0) — no spreadsheets or emails.

### 🔑 What Gets Synced
- Worker profiles & engagement details
- Timecards (hours, shift differentials)
- Job requisitions & status changes
- Position delivery

---

## The Tech Stuff (If You're Curious)

| Piece | What It Is |
|-------|-----------|
| **Magnit API** | REST-based, JSON/XML, async with 24h correlation IDs |
| **Bullhorn VMS Sync** | The bridge that connects Magnit ↔ Bullhorn |
| **BTE Report Settings** | How Bullhorn Time & Expense imports VMS time data |
| **Time Source** | Must be `WAND` for timecard API to work |
| **TEI** | Time Entry Interface — Hourly 8 (no cost allocation) |

---

## Need Help?

- **Magnit API access**: Contact your Program Representative
- **Convert to API**: Contact Bullhorn VMS Support
- **Time issues**: Check BTE Report Settings in Bullhorn

---

*Last updated: October 2026*
