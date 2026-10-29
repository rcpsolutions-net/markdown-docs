# Next-Gen Compliance Architecture: T&E Engine

*Prompted and edited by Lawrence Ham*

Move from a "data entry tool" to a **"Legal Compliance Engine."** In U.S. staffing, the software is the first line of defense against class-action labor lawsuits. Offer it as **Compliance-as-a-Service** against Bullhorn/BTE.

## Jurisdictional Trigger Matrix

State laws aren't uniform — toggle logic dynamically based on **work-site geolocation**.

| State | Daily OT (1.5x) | Double Time (2.0x) | Meal/Rest Penalties | Key Statute |
| :--- | :---: | :---: | :---: | :--- |
| **California** | > 8 hrs | > 12 hrs | **1 hr premium pay** | IWC Wage Orders |
| **Nevada** | > 8 hrs | N/A | Mandatory breaks | NRS 608.018 |
| **Alaska** | > 8 hrs | N/A | N/A | AS 23.10.060 |
| **Colorado** | > 12 hrs | N/A | 10m rest / 30m meal | COMPS Order #38 |
| **New York** | 40 hrs weekly | N/A | **Spread of Hours** | NYCRR 142-2.4 |

> **NY "Spread of Hours":** if a workday spans 8:00 AM–7:00 PM (11-hour spread), the employee is owed an **extra hour at minimum wage**, even if they only worked 8 hours.

## Smart-Punch Logic Flow

The system doesn't just record a timestamp — it evaluates it against a **versioned rule set**:

geo-fence check → identify local jurisdiction → load effective RuleSet → **OT logic** (daily CA 8/12, shift CO 12hr, weekly FLSA) + **meal check** (short/late → add 1hr penalty pay; on time → valid) → clean payroll export.

## Pay-Bucket Distribution

A CA 14-hour shift is **not** "14 hours" — it's fragmented into four line items stored separately for PeopleSoft/BTE:

- Regular (8 hrs)
- Overtime 1.5x (4 hrs)
- Double Time 2x (2 hrs)
- Meal Penalty (1 hr premium)

## Database Schema: Effective-Dated Architecture

Laws change mid-year (e.g., July 1st min-wage hikes), so use **versioned RuleSets**:

- `JURISDICTION` (name, code, local_admin e.g. "IWC") — governs →
- `RULE_SET` (`effective_start`, `effective_end`, `version` e.g. v2.4) — defines →
- `OVERTIME_LOGIC` (`ot_threshold` 8.0, `dt_threshold` 12.0, `consecutive_7th_day_rule`) and `PENALTY_LOGIC`
- `TIMECARD` is **calculated by** a `RULE_SET`.

## Compliance Lifecycle (Staying Current)

Automate law updates via legal data feeds (e.g., **Wolters Kluwer**):

Labor Dept announces a change → Compliance Feed pushes a JSON webhook (e.g. `{state: 'CA', new_rate: 16.50}`) → engine drafts a new RuleSet (v2025.2) → staffing admin approves → effective date: new RuleSet goes ACTIVE.

## Audit-Proof Attestation

BTE's strongest feature is **employee attestation** — the data that stops lawsuits. Before submitting, the employee digitally signs a statement. Store per timecard:

- `attestation_id` — unique hash of the legal text shown
- `ip_address` — where they signed from
- `meal_period_waived` — boolean (**critical for CA**)
- `injury_reported` — boolean

## Winning Strategy

1. **Never** let a user "build" a California rule from scratch.
2. **Always** provide a **Verified Template** vetted by legal researchers.
3. **Automatically** flag **at-risk** timecards in real-time (e.g., a worker with 4.9 hrs and no lunch) *before* the shift ends.
