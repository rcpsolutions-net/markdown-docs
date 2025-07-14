# Partner's Personnel: Project MS-SQL -> PowerBi -> RCP v4

## Overview
Partners Personnel: This document is a rough draft demonstrating the steps and softeware necessary to connect Partners' MS-SQL Logship to MS PowerBi using Azure and Microsoft Graph API. This solution enables automated data refresh and report generation based on client needs

## Architecture Overview

```mermaid
graph TD
    A[Node.js App] --> B[Microsoft Graph API]
    A --> C[SQL Server]
    B --> D[Power BI Service]
    C --> E[Database]
    D --> F[Power BI Reports]
    D --> G[Power BI Dashboards]
    
    style A fill:#90EE90
    style B fill:#ADD8E6
    style C fill:#FFB6C1
    style D fill:#DDA0DD
    style E fill:#F0E68C
    style F fill:#98FB98
    style G fill:#98FB98
```

## Authentication Flow

```mermaid
sequenceDiagram
    participant App as Node.js App
    participant Auth as Azure AD
    participant Graph as Microsoft Graph
    participant PBI as Power BI Service
    
    App->>Auth: Request Access Token
    Auth-->>App: Return Token
    App->>Graph: Authenticate with Token
    Graph-->>App: Confirm Authentication
    App->>PBI: Access Power BI API
    PBI-->>App: API Response
```

## System Components

```plantuml
@startuml
package "Node.js Application" {
    [Config Manager]
    [API Client]
    [SQL Connector]
    [Report Generator]
}

package "Microsoft Services" {
    [Azure AD]
    [Graph API]
    [Power BI Service]
}

package "Data Source" {
    [SQL Server]
    [Database]
}

[Config Manager] --> [API Client]
[API Client] --> [Graph API]
[SQL Connector] --> [SQL Server]
[Report Generator] --> [Power BI Service]
[Graph API] --> [Power BI Service]
[SQL Server] --> [Database]
@enduml
```

## Implementation Steps

### 1. Project Setup

```bash
# Install required dependencies
npm install @microsoft/microsoft-graph-client mssql @azure/identity dotenv
```

### 2. Environment Configuration

Create a `.env` file:

```env
AZURE_CLIENT_ID=your_client_id
AZURE_CLIENT_SECRET=your_client_secret
AZURE_TENANT_ID=your_tenant_id
SQL_SERVER=your_server
SQL_DATABASE=your_database
SQL_USER=your_username
SQL_PASSWORD=your_password
```

### 3. Database Connection

```javascript

const config = {
    user: process.env.SQL_USER,
    password: process.env.SQL_PASSWORD,
    server: process.env.SQL_SERVER,
    database: process.env.SQL_DATABASE,
    options: {
        encrypt: true
    }
};
```

### 4. Microsoft Graph Authentication

```javascript

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

module.exports = { client };
```

## Data Flow Diagram

```mermaid
flowchart LR
    subgraph Node.js App
        A[Config] --> B[Auth]
        B --> C[API Client]
        D[SQL Connector] --> E[Data Processor]
        E --> F[Report Generator]
    end
    
    subgraph External Services
        G[Azure AD] --> H[Graph API]
        I[SQL Server] --> J[Database]
        H --> K[Power BI]
    end
    
    B --> G
    C --> H
    D --> I
    F --> K
```

## Error Handling

```mermaid
graph TD
    A[Error Occurs] --> B{Error Type}
    B -->|Authentication| C[Token Error]
    B -->|SQL| D[Database Error]
    B -->|API| E[Graph API Error]
    
    C --> F[Refresh Token]
    D --> G[Retry Connection]
    E --> H[API Retry Logic]
    
    F --> I[Log Error]
    G --> I
    H --> I
    
    I --> J[Notify Admin]
```

## Best Practices

1. Security
   - Store credentials securely
   - Use managed identities when possible
   - Implement proper error handling
   - Regular token rotation

2. Performance
   - Implement connection pooling
   - Cache frequently used data
   - Use incremental refresh
   - Optimize SQL queries

3. Monitoring
   - Log all operations
   - Track API usage
   - Monitor refresh times
   - Set up alerts


## Deployment Architecture

```plantuml
@startuml
node "Azure Cloud" {
    [Azure App Service] as AppService
    [Azure Key Vault] as KeyVault
    [Azure AD] as AAD
}

node "Data Sources" {
    database "SQL Server" as SQL
}

node "Power BI" {
    [Power BI Service] as PBI
    [Reports] as Reports
    [Dashboards] as Dashboards
}

AppService --> KeyVault : Fetch Secrets
AppService --> AAD : Authenticate
AppService --> SQL : Query Data
AppService --> PBI : Update Reports
PBI --> Reports : Generate
PBI --> Dashboards : Update

@enduml
```

## Monitoring and Maintenance

```mermaid
graph TD
    A[Monitoring System] --> B[Log Analytics]
    A --> C[Performance Metrics]
    A --> D[Error Tracking]
    
    B --> E[Alert System]
    C --> E
    D --> E
    
    E --> F[Admin Notification]
    E --> G[Auto-Recovery]
    
    style A fill:#f9f,stroke:#333,stroke-width:4px
    style E fill:#bbf,stroke:#333,stroke-width:4px
```

document prompted and slightly edited by lham 05/02/2025
