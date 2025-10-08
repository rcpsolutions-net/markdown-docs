---

# Meeting Summary: OCR Timecard Automation Demo & Feedback

**Objective:** To review a new OCR/AI tool for automating manual timecard entry and gather feedback from the payroll team to guide the next development steps.

### Key Takeaways

1.  **Successful OCR Demonstration:**
    *   Lawrence presented a tool that successfully ingests PDF timecards, uses OCR/AI to extract punch data, and visually flags discrepancies (e.g., calculated hours vs. reported hours) for user review.

2.  **Crucial Bullhorn Data Needed for Validation:**
    *   The payroll team confirmed that to process timecards, they must cross-reference data from Bullhorn.
    *   The tool must pull and display the following for the relevant associate:
        *   **Assignment ID** (Most critical identifier)
        *   **Client / Customer ID Number**
        *   Pay Rate (especially for state-specific OT rules)
        *   Week Start/End Dates

3.  **Complex Business Logic is Required:**
    *   Payroll rules are not one-size-fits-all. The system must eventually account for rules based on **Client, State, County, and City**.
    *   The tool also needs to handle special pay codes (e.g., holiday, sick time) that may appear on timecards.

4.  **Technical Path Forward:**
    *   Instead of relying on the potentially restrictive Bullhorn API, Lawrence will integrate directly with the **mirrored SQL database**. This provides greater flexibility and speed for development.
    *   The primary technical challenge is creating a robust query that uses the OCR'd data (associate name, client name/ID) to reliably find the correct **Assignment ID** in the database.

---

### ✅ Action Items

| Action Item | Owner(s) | Details |
| :--- | :--- | :--- |
| **1. Provide UI Feedback & Mockups** | Payroll Team | Send sketches, screenshots, or descriptions of the ideal user interface, including required fields and buttons. |
| **2. Grant Database Access** | Yasser | Provide Lawrence with access to the Bullhorn SQL mirror database. |
| **3. Begin Bullhorn Integration** | Lawrence Ham | Once access is granted, start developing the queries to match timecards and pull required data (Assignment ID, Pay Rate, etc.). |
| **4. Compile Business Rules** | Maria Tinoco | Obtain the master document of all client, state, and local payroll rules from Jen Rivas for future implementation. |
| **5. Schedule Follow-up Meeting** | David Bolde | Organize a follow-up call for the next week to review progress. |
