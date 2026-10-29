# Meeting Debrief: OCR Automation Tool with IT & Payroll

*Prompted and edited by Lawrence Ham*

**Date:** October 7, 2025 — a live demo of the new OCR/AI tool for digitizing manual timecard entry, plus payroll-team feedback to guide the next phase.

## The Tool in Action

Lawrence presented the core workflow: **PDF upload → OCR + AI extraction → automatic validation.**

- **Ingestion:** starts with a simple PDF upload. Future: automated ingestion from a dedicated email inbox or fax line.
- **Extraction:** scans the document and pulls employee names, dates, and individual time punches (in/out for lunch, in/out for the day).
- **Validation & flagging:** the system auto-calculates and highlights **in red** any field where the AI's calculated hours don't match the total written on the timecard. Payroll immediately got it: *"Is that why they're red, Lawrence? Is that like to flag your attention?"* This turns the tool from a data-entry assistant into a **proactive validation engine**.

## The "Missing Link": Bullhorn Context

The OCR data is incomplete without Bullhorn context. To stop users switching between systems, the tool must become a **unified validation dashboard**. Must-have fields to pull from Bullhorn alongside the OCR'd card:

- **Assignment ID** — the **absolute primary key**; all hours are submitted against a specific Assignment ID, so it's the most critical matching piece.
- **Client / Customer ID** — essential to identify the client, especially on national VMS accounts where names are generic or repetitive across locations.
- **Pay Rate** — crucial for complex OT rules. As Karla noted, **Nevada** rules depend on whether an employee earns more or less than **$18/hour**. Displaying it in-tool is non-negotiable.
- **Week start/end dates** — to validate the card's period against the assignment's active dates.

## Complex Business Rules

A simple hours calculator isn't enough — the tool eventually needs a **rules engine**:

- **Multi-layered hierarchy:** payroll rules are governed by **Client-specific → State → County → City** mandates.
- **Specialized pay codes:** Erin flagged the need to recognize and categorize pay codes (holiday, sick, vacation) captured on a card.

## The Path Forward: Direct SQL Integration

A pivotal technical decision: instead of the Bullhorn API's limitations, connect **directly to the mirrored SQL database**.

> **Lawrence:** "If I have access to... the mirror database, it's gonna be rather quick. I'm gonna shock you guys."

Direct SQL gives faster queries, more flexibility, and unrestricted access to the tables — accelerating integration and enabling more complex, reliable matching logic.

## Action Plan & Next Steps

| Action Item | Owner(s) | Details |
| :--- | :--- | :--- |
| **Provide UI feedback & mockups** | Payroll Team | Sketches/screenshots/descriptions of the ideal UI — layout of Bullhorn data, required buttons, overall workflow. |
| **Grant database access** | Yasser | Credentials + connection info for the Bullhorn SQL mirror database. |
| **Develop integration queries** | Lawrence | Once access is granted, build the SQL queries to match OCR'd data to Bullhorn records and pull the critical fields (Assignment ID, Pay Rate, etc.). |
| **Compile master business rules** | Maria (+ Jen) | Obtain the master document of all client/state/local payroll rules for a future development phase. |
| **Schedule follow-up meeting** | David | Next follow-up call to review Lawrence's integration progress and the payroll team's UI feedback. |
