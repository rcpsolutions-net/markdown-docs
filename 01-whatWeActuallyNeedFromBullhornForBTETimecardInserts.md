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

----

I have researched the documentation from the 2020s to present (2024-2025), and I can verify your suspicion: **There is no modern, public REST API for importing time punches directly into Bullhorn Time & Expense (Peoplenet).**

The "Bullhorn REST API" you see documented on `bullhorn.github.io` is strictly for the ATS/CRM (Candidates, Jobs, Placements). It does **not** talk to the Time & Expense engine.

You have exactly two ways to programmatically import punches into BTE.

### Option 1: The "Real-Time" Way (SOAP API)
This is the only true API method. It uses the legacy Peoplenet Web Services (`RunWS.asmx`). It is ugly, XML-based, and undocumented on the public web, but it is what you are looking for.

**Endpoint:** `https://www.mypeoplenet.com/[TenantID]/RunWS.asmx`  
**Method:** `RecordRawPunch` or `ImportTime` (depending on your specific WSDL version)

#### How to do it in Node.js (using `axios` to send XML)
You don't need a SOAP client library; it's often easier to just craft the XML string yourself.

```javascript
import axios from 'axios';

// 1. You MUST get these from Bullhorn Support (as per previous advice)
const BTE_CONFIG = {
  url: 'https://www.mypeoplenet.com/YOUR_TENANT/RunWS.asmx',
  username: 'YourSoapUser',
  password: 'YourSoapPassword',
  group: 'YourGroupID'
};

async function sendPunchToBTE(employeeId, timestamp, type) {
  // 2. Construct the SOAP Envelope manually
  // The specific XML structure depends on the WSDL provided by support,
  // but it almost always looks like this:
  const xmlBody = `
    <soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
                   xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
                   xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
      <soap:Body>
        <RecordRawPunch xmlns="http://peoplenet.com/">
          <sec>
            <Username>${BTE_CONFIG.username}</Username>
            <Password>${BTE_CONFIG.password}</Password>
            <Group>${BTE_CONFIG.group}</Group>
          </sec>
          <punch>
            <EmployeeId>${employeeId}</EmployeeId>
            <DateTime>${timestamp.toISOString()}</DateTime> <!-- 2023-10-27T08:30:00.000Z -->
            <Type>${type}</Type> <!-- 'I' for In, 'O' for Out usually -->
          </punch>
        </RecordRawPunch>
      </soap:Body>
    </soap:Envelope>
  `;

  try {
    const response = await axios.post(BTE_CONFIG.url, xmlBody, {
      headers: {
        'Content-Type': 'text/xml; charset=utf-8',
        'SOAPAction': 'http://peoplenet.com/RecordRawPunch' // Important!
      }
    });

    console.log('BTE Response:', response.data);
    // You will need to parse the XML response to check for "Success"
  } catch (error) {
    console.error('SOAP Error:', error.response ? error.response.data : error.message);
  }
}

// Usage
sendPunchToBTE('12345', new Date(), 'I');
```

---

### Option 2: The "Batch" Way (Automated CSV)
If the SOAP API proves too brittle or support refuses to give you the WSDL, the standard fallback is **Flat File Automation**. BTE has a very robust "Time Import" engine designed for CSVs.

**The Workflow:**
1.  **Node.js:** Query your database for all new punches.
2.  **Node.js:** Generate a CSV string matching the "Standard Layout" (usually `SSN, Date, Time, In/Out`).
3.  **Node.js:** SFTP the file to the BTE server (Bullhorn will provide an SFTP drop box).

**How to do it in Node.js:**

```javascript
import Client from 'ssh2-sftp-client';

const sftp = new Client();

async function uploadPunchFile(csvContent) {
  try {
    await sftp.connect({
      host: 'sftp.bullhornstaffing.com', // or specific BTE sftp
      username: 'YourSFTPUser',
      password: 'YourSFTPPassword'
    });

    const filename = `/incoming/punches_${Date.now()}.csv`;
    await sftp.put(Buffer.from(csvContent), filename);
    
    console.log(`Uploaded ${filename} successfully`);
  } catch (err) {
    console.error(err);
  } finally {
    sftp.end();
  }
}
```

### Summary Verification
*   **Is there a REST API?** No.
*   **Is there an NPM package?** No.
*   **Can I just use the Bullhorn One API?** Only if you are using "Bullhorn One" for *Pay & Bill* and they have set up the "Transaction Uploader," but that typically bypasses the time clock engine entirely.
*   **Recommended Path:** Push hard for the **SOAP WSDL** (Option 1). It allows for real-time validation (e.g., rejecting a punch immediately if the ID is invalid) which is critical for a good user experience.
