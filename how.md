Programmatic Insertion of Associate Time Card Data: Architectural Analysis of Bullhorn REST and Time & Expense APIs
I. Core API Access and Architectural Overview
The integration environment for Bullhorn is strictly governed by conventional RESTful architecture, specialized data center routing, and rigorous OAuth 2.0 security protocols. Understanding this foundation is paramount before attempting data transmission, particularly for transactional data like time cards.
A. Bullhorn REST API Conventions and Infrastructure
The Bullhorn REST API adheres to widely accepted standards, utilizing conventional HTTP practices regarding methods, character encodings, and URL case sensitivity. Data communication relies exclusively on the JavaScript Object Notation (JSON) format.
A critical architectural constraint is the mandatory use of data-center-specific URLs. Developers cannot rely on a single, global endpoint. The correct URL must be retrieved by executing a GET rest-services/loginInfo request using the API username. If the integration fails to use the correct data center URL, the Bullhorn system issues a 307 redirect. Consequently, the integration code must be explicitly engineered to handle this 307 redirect seamlessly to proceed with any subsequent API requests. Furthermore, all API URLs are logically partitioned by corporation, requiring a {corptoken} identifier as the first path element in all entity access paths (e.g., .../{corptoken}/entity/Candidate).
B. The Multi-Step OAuth 2.0 Authentication Flow
Bullhorn enforces OAuth 2.0 as the sole mechanism for REST API authorization, ensuring secure communication between the external application (client) and the Bullhorn resources. Establishing a valid session, represented by the BhRestToken, requires a sequential four-step process:
 * Data Center Determination: The appropriate data center URL ({DC}) is discovered by making a GET call to https://rest.bullhornstaffing.com/rest-services/loginInfo?username={API_Username}.
 * Authorization Code Retrieval: An authorization request is made to the specific data center’s authorize endpoint (/oauth/authorize), typically involving the provision of Bullhorn user credentials (manually or programmatically) to retrieve the temporary authCode.
 * Access Token Retrieval: The authCode is then exchanged for the access_token (valid for 10 minutes) and potentially a long-lived refresh_token via a POST request to the /oauth/token endpoint, requiring the application's client_id and client_secret.
 * REST Session Login: Finally, the access_token is used in a POST /rest-services/login request. The response returns the persistent BhRestToken (the session key) and the definitive restUrl required for all subsequent entity interactions.
OAuth 2.0 Authentication Flow Visualization
sequenceDiagram
    participant Client
    participant Bullhorn_Auth
    participant Bullhorn_REST
    Client->>Bullhorn_Auth: 1. GET /loginInfo?username={API_User}
    Bullhorn_Auth-->>Client: Data Center URL ({DC})
    Client->>Bullhorn_Auth: 2. GET /oauth/authorize (w/ Credentials) [span_18](start_span)[span_18](end_span)[span_30](start_span)[span_30](end_span)
    Bullhorn_Auth-->>Client: Authorization Code ({authCode}) [span_19](start_span)[span_19](end_span)[span_31](start_span)[span_31](end_span)
    Client->>Bullhorn_Auth: 3. POST /oauth/token (w/ {authCode}) [span_20](start_span)[span_20](end_span)[span_32](start_span)[span_32](end_span)
    Bullhorn_Auth-->>Client: Access Token / Refresh Token [span_21](start_span)[span_21](end_span)[span_33](start_span)[span_33](end_span)
    Client->>Bullhorn_REST: 4. POST /login (w/ Access Token) [span_22](start_span)[span_22](end_span)[span_34](start_span)[span_34](end_span)
    Bullhorn_REST-->>Client: BhRestToken + restUrl [span_23](start_span)[span_23](end_span)[span_35](start_span)[span_35](end_span)

C. API Governance, Rate Limits, and Performance Implications
Every developer application must obtain a unique partner API key from Bullhorn, which allows Bullhorn customers to manage and control application usage individually.
Bullhorn strictly enforces usage limits to ensure system stability. These limits vary dramatically based on the customer’s edition, a crucial factor when planning high-volume transactional integrations like payroll time card insertion.
| Edition | Max Concurrent Sessions | Total Daily Calls | Max Calls Per Minute |
|---|---|---|---|
| Corporate Edition | Up to 25 | 100,000 | 700 |
| Enterprise Edition | Up to 50 | 2,000,000 | 1,500 |
The disparity between the editions implies that any system designed for routine, high-volume batch processing of associate time entries must utilize the Enterprise Edition. Attempting to process large payroll batches using the Corporate Edition's limit of 100,000 calls per day would severely constrain operational throughput and increase the likelihood of encountering 429 "Too Many Requests" errors, necessitating sophisticated error handling and throttling mechanisms.
II. Mapping Core Entities for Payroll Integration
In the Bullhorn ecosystem, the individual whose time is being tracked—referred to in the query as the "associate"—is represented primarily by two linked entities: the Candidate (identity) and the Placement (the active assignment).
A. The Hierarchical Relationship: Candidate, Placement, and Associate Identity
Entities are the core data types within the Bullhorn system, capturing essential staffing concepts such as Candidate, JobOrder, and Placement.
The Candidate entity represents the fundamental identity of the potential employee or contractor, holding their personal details, work history, and primary employment information. Conversely, the Placement entity is the record of a successfully completed job assignment. It serves as the definitive anchor linking the Candidate to the specific job, contact, and, most critically for payroll, the financial terms and time tracking data. A successful time card insertion requires a valid ID from an active Placement.
B. The Candidate Entity: Identity and Employment Classification
The Candidate record houses the foundational information required for tax and core employment classification. These fields are essential for setting up the associate in any downstream payroll system .
Candidate Entity Key Payroll-Related Fields
| Field Name | Type | Payroll Relevance | Not Null (X) |
|---|---|---|---|
| id | Integer | Unique identifier for the Candidate (Associate). | X |
| employeeType | String (30) | Candidate’s employment classification (e.g., 1099 or W2). | X |
| payrollClientStartDate | Timestamp | Date the employee was first on payroll at the staffing company. |  |
| payrollStatus | String | Indicates if the Candidate is currently active on payroll. |  |
| ssn | String (18) | Candidate's Social Security Number . |  |
The employeeType field is a required identifier defining the contractual nature of the worker (e.g., 1099 or W2). The payrollClientStartDate provides the date of first payroll activity, an important marker for system integration. The payrollStatus field indicates the candidate's current eligibility for payroll processing. The architecture dictates that the ability to submit time card data is conditional not just on the Candidate's existence but also on their being linked to an active assignment. Therefore, verifying the payrollStatus and ensuring linkage to a Placement ID must precede any attempt to insert time data.
C. The Placement Entity: The Assignment and Financial Anchor
The Placement entity is the operational hub for payroll calculations and time tracking, acting as the assignment anchor for the Candidate. Bullhorn has evolved the Placement entity into a "Generic Entity," permitting extensibility comparable to the Candidate entity .
Placement Entity Key Payroll-Related Fields (Assignment Anchor)
| Field Name | Type | Payroll Relevance | Not Null (X) |
|---|---|---|---|
| id | Integer | Unique identifier for the Placement/Assignment. | X |
| payRate | BigDecimal | Standard compensation rate. | X |
| payRateUnit | String (20) | Unit of compensation (per hour/annum). | X |
| overtimeRate | BigDecimal | Rate for overtime work. |  |
| payGroup | String | Frequency with which the placement is paid. |  |
| dateBegin | Timestamp | Placement start date. | X |
| payrollSyncStatus | To-one association | Sync status with the payroll provider. |  |
| timeCard | TimeCard | Timecards associated with this Placement. | Not supported in this release  |
These fields define the compensation structure, including payRate, payRateUnit, and overtimeRate. The dateBegin and payGroup fields establish the payroll validity and cycle frequency.
A critical finding is that while the Placement schema logically includes fields like timeCard, timecardExpenses, and timecardTimes, the Bullhorn REST API documentation explicitly marks these as "Not supported in this release". This technical status confirms that transactional time data insertion cannot be performed using standard REST entity manipulation calls against the Placement entity.
Furthermore, the extensive customization options available on the Placement record, including up to 30 custom text fields and 30 custom dropdowns , are essential for integrating complex payroll requirements. These custom fields (assignmentCustomText1 - 30) can be mapped to external systems to capture specific payroll metadata, such as cost centers or internal client job codes, which are often non-standard but mandatory components of time card submissions.
D. Payroll Provider Entities
The Bullhorn structure also includes entities tailored for payroll provider data integration, such as the Deduction entity, which stores information like the provider’s identifier (id), deduction code, and description . These provider exports are organized around defined pay periods, using key timestamps such as periodStartDate and periodEndDate. The time data inserted must conform precisely to these period boundaries for successful batch processing .
III. Critical Architectural Assessment: Time Card Integration Pathway
Given the explicit status of time card entities within the primary REST API, the transactional integration pathway mandates a shift away from standard entity manipulation toward specialized services integrated with the Bullhorn Time & Expense (BTE) platform.
A. The Status of Native TimeCard Entities in the REST API
The explicit notation in the Placement entity reference that timeCard and associated time entry fields are "Not supported in this release" provides a definitive architectural constraint. This means that the standard Bullhorn REST API operations—such as POST {corptoken}/entity/TimeCard—cannot be used for creating or updating time entries. This forces developers to pursue alternative, specialized integration paths.
B. Requirement for Bullhorn Time & Expense (BTE) Integration
Programmatic insertion of time card data requires the use of the specialized Bullhorn Timesheet API, which interacts directly with the Bullhorn Time & Expense (BTE) platform . BTE, often integrated via PeopleNet technologies , is responsible for managing complex time and labor rules, approvals, and subsequent data translation into Payable and Billable charges within the Bullhorn ATS .
Crucially, setting up access to this specialized Timesheet API is not a self-service process; it demands mandatory engagement with Bullhorn Support . Developers must provide the name of the third party, along with necessary Redirect URIs and Terms of Service agreements, to receive the required API details and credentials .
C. Overview of BTE Time Capture Methods
The specialized API serves as a programmatic interface to BTE’s internal data ingestion methods, which handle administrative and bulk data entry . Bullhorn Time & Expense supports several methods for collecting time, including Web Time Entry (WTE) and Rapid Time Entry (RTE).
The most relevant internal BTE mechanism for high-volume programmatic integration is the Transaction Uploader. This tool allows administrators to upload candidate time, units, or hours in batch formats via spreadsheets. The existence of this batch import capability strongly suggests that the specialized Timesheet API endpoint is designed to accept similar high-volume, transactional payloads for time data insertion. By routing time through BTE, Bullhorn ensures that complex business logic—including pay rule validation and compliance checks—is applied consistently, preventing external integrations from bypassing critical payroll integrity measures.
IV. Programmatic Insertion Requirements and Data Structure
Successful time card data insertion via the specialized BTE API hinges on precise data formatting, association with an active Placement, and adherence to specific field requirements established in the BTE system.
A. Defining the Time Card Payload Requirements
The transactional payload must fundamentally link the time entry to the corresponding assignment in the system. The essential linkage is the active Placement ID (or Assignment Number), as time entries are tracked as events associated with the assignment, not the Candidate's static identity. The data payload must specify the transactional details, including the time granularity (hours, units, or dollars) and must align with the periodStartDate and periodEndDate of the pay cycle defined by the Placement’s payGroup .
B. The Role of Customer Required Fields (CRFs)
A critical factor for validation is the status of Customer Required Fields (CRFs). CRFs are configurable text fields or lists of values used to capture client-specific data, such as Purchase Orders or specific cost centers, on the timesheet.
If CRFs are enabled and designated as required in both Bullhorn ATS and Bullhorn Time & Expense , the programmatic insertion payload must include valid data for these mandatory fields. Failure to satisfy a required CRF will result in the rejection of the time entry by the BTE validation engine. Furthermore, for time capture mechanisms that are non-interactive, such as clock placements (which can be analogous to programmatic API insertion), a default CRF must be assigned to the Placement, confirming that CRF handling is integral to the integration design.
C. Prerequisites and Entitlements
The API user account must be provisioned with sufficient access rights. For general entity creation, the user requires the CREATE entitlement . For BTE operations, this translates into appropriate administrative or View/Edit access within the Bullhorn Time & Expense Role settings.
Prior to transmitting transactional time data, the system must perform comprehensive pre-validation. It is a best practice that all Placement data—including financial rates, assignment dates, and the payrollSyncStatus—be confirmed as correct and approved in the ATS, as this information flows into BTE to establish the necessary context for time processing.
The integration architecture suggests that time card processing is a critical, atomic transaction managed by BTE. The process, labeled internally as a "Closed Time Sync: BTE-->BH," creates a distinct batch event tied to the Placement for every submission . This means the insertion payload should be treated as a single, indivisible transaction; if any component (such as a missing CRF or an expired Placement date) is invalid, the entire time entry is likely to be rejected. Designing the integration with comprehensive error handling and validation logic is therefore essential to guarantee transactional reliability.
V. Implementation Blueprint and Advanced Recommendations
The implementation blueprint requires a continuous focus on security, token longevity, and robust error detection mechanisms, especially given the rate limits on API calls.
A. Session and Token Management Blueprint
The operational process starts with the OAuth 2.0 flow to secure the REST session token (BhRestToken) and culminates in the use of the specialized BTE Timesheet API endpoint.
| Step | Purpose | Request Type | Endpoint Segment | Key Output |
|---|---|---|---|---|
| 1 | Locate DC URL | GET | rest.bullhornstaffing.com/rest-services/loginInfo | Data Center Identifier ({DC})  |
| 2 | Obtain Auth Code | GET | auth-{DC}.bullhornstaffing.com/oauth/authorize | Authorization Code ({authCode})  |
| 3 | Obtain Access Token | POST | auth-{DC}.bullhornstaffing.com/oauth/token | Access Token, Refresh Token  |
| 4 | Establish REST Session | POST | rest-{DC}.bullhornstaffing.com/rest-services/login | BhRestToken, restUrl  |
| 5 | Time Card Insertion | POST | Specialized BTE Timesheet API endpoint | Transaction Status |
The refresh token management strategy is key for long-term integration stability. Since the refresh token expires only after it is used once successfully to generate a new access token, developers should leverage the latest refresh token for subsequent sessions to avoid the need to repeat the authorization code retrieval step (Step 2), which often requires manual user intervention or credential submission.
B. Time Card Insertion Architectural Flow
This diagram illustrates the mandatory separation between core ATS entity management and the transactional time data ingestion via Bullhorn Time & Expense (BTE).
graph TD
    A --> B{Placement ID, PayGroup, PayRate Confirmed};
    B --> C;
    C --> D{Programmatic POST to Specialized BTE API};
    D --> E;
    E -- Validation Success --> F;
    F --> G;
    G --> H
    E -- Validation Failure --> I

C. Monitoring and Troubleshooting
In high-volume payroll scenarios, efficient error handling is essential, particularly for rate limit violations. The integration should be prepared to handle HTTP 429 responses ("Too Many Requests") by implementing throttling mechanisms or dynamically adjusting the size and frequency of batch submissions to remain within the established call limits of the licensed edition.
For troubleshooting synchronization issues between the time entries and the payroll/billing systems, the BTE Placement Viewer is an invaluable administrative tool . Specifically, the "Closed Time Sync: BTE-->BH" tab provides a detailed record of every time the placement has been synced to Pay & Bill, allowing administrators to trace batch events, identify insertion failures, and verify that the transactional time card data successfully flowed from BTE back to the core Bullhorn billing system .
VI. Conclusions and Recommendations
The analysis of Bullhorn documentation reveals a distinct architectural separation between core entity management and transactional time processing, necessitating a two-pronged approach for programmatic time card insertion.
 * Mandatory Specialized API Use: Standard Bullhorn REST API entity operations are insufficient for inserting time card data because the required entities (TimeCard, TimecardTimes) are explicitly marked as unsupported. The sole validated pathway is through the specialized Bullhorn Timesheet API, which operates in conjunction with the Bullhorn Time & Expense (BTE) platform . Integration development must start with contacting Bullhorn Support to acquire the unique credentials and access details for this specialized endpoint .
 * Placement Entity as the Transactional Anchor: The success of time card insertion depends entirely on the Placement entity. The payload must be validated against the Placement’s ID, effective dates (dateBegin), financial terms (payRate, payRateUnit), and pay cycle (payGroup). Prior confirmation that the Placement has successfully synced to the payroll provider (payrollSyncStatus) is crucial to avoid downstream processing failures.
 * High-Volume Scaling Requires Enterprise Licensing: For agencies planning high-volume or frequent payroll integrations, the restrictive rate limits of the Corporate Edition (100,000 calls per day) are likely insufficient. Utilizing the Enterprise Edition, which allows up to 2,000,000 calls per day and higher per-minute throughput, is a prerequisite for maintaining payroll efficiency and meeting transactional demands .
 * Data Integrity through Pre-Validation: The transactional nature of BTE processing dictates that the insertion mechanism must implement robust checks for all mandatory fields, including any configured Customer Required Fields (CRFs). Any missing or invalid data, especially custom fields defined as required on the timesheet, will cause the entire time card batch to be rejected during BTE validation.
