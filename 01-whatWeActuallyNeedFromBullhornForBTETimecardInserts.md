# 📋 Directive: Acquisition of BTE (Peoplenet) API Access

**To:** Executive Assistant  
**From:** Development Lead  
**Subject:** Requesting Developer Access for Bullhorn Time & Expense (Peoplenet)  
**Priority:** High

### 🛑 Context for You
We are integrating a client who uses **Bullhorn Time & Expense** (also known as **Peoplenet**). This is *not* the standard Bullhorn CRM. It is a separate, legacy system that requires a specific set of "Web Services" credentials. The standard "REST API" keys we usually ask for will not work here.

Please open a support ticket with **Bullhorn Support** immediately using the script below.

---

### 📨 The Request Script
*Please submit the following ticket via the **Bullhorn Resource Center** (or email `support@bullhorn.com` if we do not have portal access).*

**Subject:** Developer Access Request: Peoplenet Web Services (BTE API)

**Body:**
> Hello Support Team,
>
> We are currently implementing a custom integration for Partners Personnel and require programmatic access to the Bullhorn Time & Expense (Peoplenet) backend.
>
> We specifically need access to the **Peoplenet Web Services (SOAP API)** to upload time and attendance data.
>
> **Please provide the following:**
> 1. **Integration Credentials:** A dedicated API Username and Password for the Peoplenet environment.
> 2. **WSDL / Documentation:** The most recent PDF or WSDL link for the Peoplenet Web Services.
> 3. **Environment Details:**
>    - The specific **Group ID** or **Client ID** for this tenant.
>    - The correct **Service URL** (e.g., `https://www.mypeoplenet.com/.../RunWS.asmx`).
>
> *Note: We are NOT looking for the standard Bullhorn REST API (BhRestToken) credentials. We need the legacy Peoplenet connection details.*
>
> Thank you.

---

### 💡 If They Push Back (Troubleshooting)
If the support agent seems confused or sends you a link to `bullhorn.github.io` (the wrong API), please reply with this clarification:

> "To clarify, we are not trying to access the ATS/CRM data. We need to push/pull time clock data directly from the **Time & Expense engine (Peoplenet)**. We believe this requires the **'RunWS.asmx'** SOAP endpoints. Please escalate to the Time & Expense integration team if necessary."

### ✅ Definition of Success
We are successful when I have a **PDF document** (usually 100+ pages) titled "Peoplenet Web Services" and a **SOAP Endpoint URL** that looks like `.../RunWS.asmx`.
