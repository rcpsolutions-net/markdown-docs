# Client: Project MS-SQL → PowerBI → RCP v4

*Markdown document prompted and edited by LHam.*

## Overview

Rough draft of the steps and software needed to connect **Partners' MS-SQL Logship** to **MS PowerBI** using **Azure** and the **Microsoft Graph API**. The solution enables automated data refresh and report generation based on client needs.

Architecture: Node.js app talks to both the **SQL Server** (queries) and the **Microsoft Graph API** (which drives the **Power BI Service** → reports + dashboards).

## Implementation Steps

**1. Project setup**

```bash
npm install @microsoft/microsoft-graph-client mssql @azure/identity dotenv
```

**2. Environment config** (`.env`)

```env
AZURE_CLIENT_ID=your_client_id
AZURE_CLIENT_SECRET=your_client_secret
AZURE_TENANT_ID=your_tenant_id
SQL_SERVER=your_server
SQL_DATABASE=your_database
SQL_USER=your_username
SQL_PASSWORD=your_password
```

**3. Database connection** — `mssql` config with `options: { encrypt: true }`.

**4. Microsoft Graph authentication**

```js
const credential = new ClientSecretCredential(
  process.env.AZURE_TENANT_ID,
  process.env.AZURE_CLIENT_ID,
  process.env.AZURE_CLIENT_SECRET
);

const client = Client.init({
  authProvider: async (done) => {
    try {
      const token = await credential.getToken(['https://graph.microsoft.com/.default']);
      done(null, token.token);
    } catch (err) {
      done(err, null);
    }
  }
});
```

## Error Handling

- **Token errors** → refresh the token
- **SQL errors** → retry the connection
- **Graph API errors** → apply API retry logic
- All errors → **log** and **notify admin**

## Best Practices

- **Security:** store credentials securely, use managed identities where possible, proper error handling, regular token rotation.
- **Performance:** connection pooling, cache frequently-used data, incremental refresh, optimized SQL.
- **Monitoring:** log all operations, track API usage, monitor refresh times, set up alerts.

## Deployment

**Azure App Service** + **Azure Key Vault** (secrets) + **Azure AD** (auth) → queries SQL Server → updates Power BI reports/dashboards.
