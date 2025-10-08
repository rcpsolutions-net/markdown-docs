## 🔎 Finding a Specific Placement from Timesheet Data
### Prompted and edited by Lawrence Ham

When a timesheet only provides an associate's name and a client's name, you must perform a series of queries to pinpoint the exact, active `Placement`. This process acts like a funnel, narrowing down the possibilities at each step.

**The logical flow is:**
1.  **Fast Path (If Possible):** If a unique identifier like a `placementId` is available, use it for a direct lookup. This is the most efficient method.
2.  **Search for Candidates:** Use the associate's name to find all potential `Candidate` records.
3.  **Search for Clients:** Use the client name to find all potential `ClientCorporation` records.
4.  **Correlate and Find Placements:** Use the IDs from the previous steps to find `Placement` records that connect one of the found candidates to a job at one of the found clients.
5.  **Validate and Select:** Analyze the results to find a single, active placement. Handle cases where zero or multiple matches are found.

### 📊 Diagram: The Placement Search Funnel

```mermaid
graph TD
    A[📄 Timesheet Info Associate: Jane Doe, Client: Global Tech Inc] --> B{Has placementId?};
    B -- Yes --> C[🎯 Direct Lookup: GET /entity/Placement/{id?}];
    B -- No --> D[🔍 Step 1: Search Candidates query=name:'Jane Doe'];
    A --> D;
    D --> E[Found Candidate IDs e.g., [123, 456]];
    A --> F[🔍 Step 2: Search Clients query=name:'Global Tech Inc.'];
    F --> G[Found Client IDs [88, 99]];
    E --> H[🧩 Step 3: Correlate Placements query=candidate.id IN [123,456] AND jobOrder.clientCorporation.id IN [88,99] AND status:'Approved'];
    G --> H;
    H --> I{Analyze Results};
    I -- 0 Found --> J[❌ No Match Found];
    I -- 1 Found --> K[✅ Success! Return Placement];
    I -- 2+ Found --> L[⚠️ Ambiguous Match Return all possibilities for user to select];
    C --> K;
```

### 👨‍💻 Node.js 22 Example: `findPlacementFromTimesheet`

This code assumes you already have a valid `session` object (containing `restUrl` and `BhRestToken`) from the authentication flow in the previous document.

```javascript
// Node.js 22 - Using built-in fetch

/**
 * A helper for making authenticated Bullhorn API calls.
 * Assumes a valid session object is passed.
 */
async function bullhornFetch(session, endpoint, options = {}) {
    const url = `${session.restUrl}${endpoint}`;
    const defaultOptions = {
        headers: {
            'BhRestToken': session.BhRestToken,
            'Content-Type': 'application/json'
        }
    };
    const response = await fetch(url, { ...defaultOptions, ...options });
    if (!response.ok) {
        const errorBody = await response.text();
        throw new Error(`❌ Bullhorn API Error on ${endpoint}: ${response.statusText} - ${errorBody}`);
    }
    return response.json();
}

/**
 * Parses a full name string into firstName and lastName components.
 * Handles simple "First Last" and "First Middle Last" formats.
 * @param {string} fullName The full name of the associate.
 * @returns {{firstName: string, lastName: string}}
 */
function parseName(fullName) {
    const parts = fullName.trim().split(/\s+/);
    if (parts.length === 1) {
        // Cannot reliably determine, but can search both fields
        return { firstName: parts[0], lastName: parts[0] };
    }
    const lastName = parts.pop();
    const firstName = parts.join(' ');
    return { firstName, lastName };
}


/**
 * Finds a specific, active Placement using information typically found on a timesheet.
 * @param {object} session - The Bullhorn session object { restUrl, BhRestToken }.
 * @param {object} timesheetInfo - Information from the timesheet.
 * @param {string} [timesheetInfo.placementId] - The placement ID (if available, for a fast path).
 * @param {string} [timesheetInfo.associateName] - The full name of the candidate.
 * @param {string} [timesheetInfo.clientName] - The name of the client corporation.
 * @returns {Promise<{status: 'SUCCESS'|'NOT_FOUND'|'AMBIGUOUS', data: object|array}>}
 */
async function findPlacementFromTimesheet(session, timesheetInfo) {
    // ---------------------------------
    // 🎯 FAST PATH: If Placement ID is provided
    // ---------------------------------
    if (timesheetInfo.placementId) {
        console.log(`🚀 Fast path: Searching directly for Placement ID: ${timesheetInfo.placementId}`);
        try {
            const result = await bullhornFetch(session, `entity/Placement/${timesheetInfo.placementId}?fields=id,status,candidate,jobOrder,dateBegin`);
            if (result.data && result.data.status === 'Approved') {
                 return { status: 'SUCCESS', data: result.data };
            }
        } catch (error) {
            console.warn(`Fast path failed for Placement ID ${timesheetInfo.placementId}, proceeding with name search.`);
        }
    }

    if (!timesheetInfo.associateName || !timesheetInfo.clientName) {
        throw new Error("associateName and clientName are required when placementId is not provided.");
    }
    
    console.log(`🔍 Starting search for Associate: "${timesheetInfo.associateName}" at Client: "${timesheetInfo.clientName}"`);

    try {
        // ---------------------------------
        // 👤 STEP 1: Find matching Candidates
        // ---------------------------------
        const { firstName, lastName } = parseName(timesheetInfo.associateName);
        const candidateQuery = `firstName:'${firstName}' AND lastName:'${lastName}' AND isDeleted:false`;
        console.log(`  -> Querying Candidates with: ${candidateQuery}`);
        const candidateResults = await bullhornFetch(session, `search/Candidate?query=${candidateQuery}&fields=id,firstName,lastName`);

        if (candidateResults.total === 0) {
            return { status: 'NOT_FOUND', data: { message: `No candidate found matching name '${timesheetInfo.associateName}'.` } };
        }
        const candidateIds = candidateResults.data.map(c => c.id);
        console.log(`  -> Found ${candidateResults.total} candidate(s): IDs [${candidateIds.join(', ')}]`);

        // ---------------------------------
        // 🏢 STEP 2: Find matching Client Corporations
        // ---------------------------------
        const clientQuery = `name:'${timesheetInfo.clientName}' AND isDeleted:false`;
        console.log(`  -> Querying Clients with: ${clientQuery}`);
        const clientResults = await bullhornFetch(session, `search/ClientCorporation?query=${clientQuery}&fields=id,name`);
        
        if (clientResults.total === 0) {
            return { status: 'NOT_FOUND', data: { message: `No client found matching name '${timesheetInfo.clientName}'.` } };
        }
        const clientCorpIds = clientResults.data.map(c => c.id);
        console.log(`  -> Found ${clientResults.total} client(s): IDs [${clientCorpIds.join(', ')}]`);

        // ---------------------------------
        // 🧩 STEP 3: Correlate to find Placements
        // ---------------------------------
        const placementQuery = `candidate.id IN (${candidateIds.join(',')}) AND jobOrder.clientCorporation.id IN (${clientCorpIds.join(',')}) AND status:'Approved'`;
        const placementFields = 'id,status,dateBegin,dateEnd,candidate(id,firstName,lastName),jobOrder(id,title,clientCorporation(id,name))';
        console.log(`  -> Querying Placements with: ${placementQuery}`);
        const placementResults = await bullhornFetch(session, `search/Placement?query=${placementQuery}&fields=${placementFields}`);

        // ---------------------------------
        // ✅ STEP 4: Analyze and return result
        // ---------------------------------
        if (placementResults.total === 0) {
            return { status: 'NOT_FOUND', data: { message: 'An active placement could not be found for the specified candidate and client.' } };
        }
        
        if (placementResults.total === 1) {
            console.log('✅ Success! Found a single matching placement.');
            return { status: 'SUCCESS', data: placementResults.data[0] };
        }
        
        // If more than one, it's ambiguous. The user/system needs to decide.
        console.log(`⚠️ Ambiguous result. Found ${placementResults.total} possible placements.`);
        return {
            status: 'AMBIGUOUS',
            data: {
                message: `Multiple active placements found. Please select the correct one.`,
                placements: placementResults.data
            }
        };

    } catch (error) {
        console.error("An error occurred during the placement search:", error);
        throw error; // Re-throw the error for higher-level handling
    }
}

// --- Example Usage ---
async function main() {
    // Mock the getBullhornSession() for this example
    const mockSession = {
        restUrl: 'https://restXX.bullhornstaffing.com/rest-services/ABCDE/',
        BhRestToken: 'YOUR_FAKE_TOKEN'
    };

    console.log("\n--- Scenario 1: Successful Match ---");
    const timesheet1 = {
        associateName: 'Jane Doe',
        clientName: 'Global Tech Inc.'
    };
    // const result1 = await findPlacementFromTimesheet(mockSession, timesheet1);
    // console.log('Result:', JSON.stringify(result1, null, 2));


    console.log("\n--- Scenario 2: No Match Found (Bad Name) ---");
    const timesheet2 = {
        associateName: 'John Smithson', // Non-existent name
        clientName: 'Global Tech Inc.'
    };
    // const result2 = await findPlacementFromTimesheet(mockSession, timesheet2);
    // console.log('Result:', JSON.stringify(result2, null, 2));


    console.log("\n--- Scenario 3: Ambiguous Match (Multiple Placements) ---");
    const timesheet3 = {
        associateName: 'Peter Jones', // This person has two jobs at the same client
        clientName: 'Innovate Solutions'
    };
    // const result3 = await findPlacementFromTimesheet(mockSession, timesheet3);
    // console.log('Result:', JSON.stringify(result3, null, 2));
    
    console.log("\n--- Scenario 4: Fast Path with Placement ID ---");
    const timesheet4 = {
        placementId: '12345'
    };
    // const result4 = await findPlacementFromTimesheet(mockSession, timesheet4);
    // console.log('Result:', JSON.stringify(result4, null, 2));
    
    console.log("\nNOTE: The above function calls are commented out as they require a live Bullhorn session.");
    console.log("Uncomment them and replace mockSession with a real session object to run.");
}

main();
```

### Key Takeaways and Best Practices

1.  **Status is Crucial:** Always include `status:'Approved'` in your `Placement` search. You don't want to process timecards for placements that are pending, terminated, or on hold.
2.  **Handle Ambiguity:** It is very possible for a contractor to have multiple, concurrent, active placements with the same client (e.g., a developer on two different projects). Your application logic must gracefully handle the `'AMBIGUOUS'` response, perhaps by presenting the options to the user for manual selection.
3.  **Use `isDeleted:false`:** When searching for primary records like `Candidate` and `ClientCorporation`, including `isDeleted:false` ensures you don't find records that have been removed from the system.
4.  **Error Handling:** The function returns a structured object `{status, data}` to make it easy for the calling code to handle the different outcomes (success, failure, ambiguity).
5.  **Field Selection:** Use the `fields` parameter in your API calls to request only the data you need. This reduces payload size and improves performance. Notice how the final placement search requests nested data like `candidate(id,firstName)` to enrich the final output without needing extra API calls.
