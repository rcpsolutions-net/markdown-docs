# UKG Developer Hub — Complete Architecture Map

> A comprehensive guide to **developer.ukg.com**: APIs, tools, authentication, and ecosystem.

---

## Table of Contents

1. [High-Level Overview](#1-high-level-overview)
2. [Product Ecosystem](#2-product-ecosystem)
3. [UKG Pro HCM — Deep Dive](#3-ukg-pro-hcm--deep-dive)
4. [UKG Pro WFM — Deep Dive](#4-ukg-pro-wfm--deep-dive)
5. [Authentication Landscape](#5-authentication-landscape)
6. [Integration Tools](#6-integration-tools)
7. [Platform Services & Emerging APIs](#7-platform-services--emerging-apis)
8. [Specialty Solutions](#8-specialty-solutions)
9. [Documentation Structure](#9-documentation-structure)
10. [Quick-Start Decision Tree](#10-quick-start-decision-tree)

---

## 1. High-Level Overview

```mermaid
graph TB
    subgraph "UKG Developer Hub (developer.ukg.com)"
        direction TB

        GS[🚀 Getting Started<br/>Call Your First API<br/>Auth Guide<br/>API Overview]

        subgraph "UKG Pro — Enterprise"
            HCM[UKG Pro HCM<br/>Human Capital Management]
            WFM[UKG Pro WFM<br/>Workforce Management]
            PP[UKG Pro Platform<br/>Webhooks & Subscriptions]
        end

        subgraph "UKG Ready — SMB"
            RDY[UKG Ready REST API]
        end

        subgraph "Specialty Solutions"
            PD[People-Doc<br/>HR Service Delivery]
            TK[Talk<br/>Frontline Engagement]
            PF[People Fabric<br/>Suite-Wide Data (Beta)]
        end

        subgraph "Integration Tools"
            ID[Integrations Dashboard<br/>HCM]
            IH[Integrations Hub / iHub<br/>WFM]
            TIP[Turnkey Integration Platform<br/>WFM]
        end

        CH[📰 What's New<br/>Changelog]
        CM[🤝 Community & Support]
    end

    GS --> HCM
    GS --> WFM
    GS --> PP
    GS --> RDY
    GS --> PF

    HCM --> ID
    WFM --> IH
    WFM --> TIP

    PF --> HCM
    PF --> WFM
```

---

## 2. Product Ecosystem

```mermaid
graph LR
    subgraph "UKG Pro Suite"
        A[UKG Pro HCM]
        B[UKG Pro WFM]
        C[UKG Pro Platform]
    end

    subgraph "UKG Ready Suite"
        D[UKG Ready HCM]
        E[UKG Ready WFM]
    end

    subgraph "Specialty"
        F[People-Doc]
        G[Talk]
        H[People Fabric]
    end

    subgraph "Developer Hub"
        Docs[REST API Docs]
        SOAP[SOAP API Docs]
        GraphQL[GraphQL API Docs]
        Guides[Conceptual Guides]
        QuickStart[Quick Starts]
        Ref[API Reference]
    end

    A --> Docs
    A --> SOAP
    A --> Ref
    B --> Docs
    B --> Ref
    C --> Docs
    C --> Ref
    D --> Docs
    H --> GraphQL
    H --> Docs
    H --> Ref
    F -.-> Docs
    G -.-> Docs
```

---

## 3. UKG Pro HCM — Deep Dive

### 3.1 REST API Domains

```mermaid
graph TB
    subgraph "UKG Pro HCM REST API — 10 Domains"
        direction TB

        subgraph "Core Employee Data"
            EMP[PRO Employee Data<br/>Manage employee records]
            AUD[Audit<br/>Data auditing & compliance]
            CFG[Configuration Setup<br/>System configuration]
        end

        subgraph "Lifecycle"
            REC[Recruiting<br/>Prospective employees]
            ONB[Onboarding<br/>New hire workflows]
            SEC[PRO Security<br/>System security]
        end

        subgraph "Pay & Time"
            PAY[Payroll<br/>Payroll processing]
            UTA[UTA<br/>Time & attendance data]
        end

        subgraph "Platform"
            PRC[PRO Platform Config<br/>Platform configuration]
            IMP[PRO Import Tools<br/>Import tool management]
        end
    end
```

### 3.2 SOAP Services (20+ Services)

```mermaid
graph TB
    subgraph "UKG Pro HCM SOAP Services — 20+ Services"
        direction TB

        subgraph "Employee Lifecycle"
            EEI[Employee Employment<br/>Information Service]
            EJS[Employee Job Service]
            ENH[Employee New Hire Service<br/>US]
            ETS[Employee Termination Service]
            EPS[Employee Person Service]
            EAS[Employee Address Service]
            EPSN[Employee Phone Service]
            EUD[Employee User Defined<br/>Fields Service]
        end

        subgraph "Global & Regional"
            GENH[Global Employee<br/>New Hire Service]
            CNH[Canadian New Hire Service]
        end

        subgraph "Compensation & Development"
            ECS[Employee Compensation Service]
            DOS[Development Opportunity Service]
            DOPS[Development Opportunity<br/>Participation Service]
            DOSS[Development Opportunity<br/>Session Service]
        end

        subgraph "Time & Pay"
            TMS[Time Management<br/>Web Services]
            TAS[Time and Attendance Service]
            EPSST[Employee Pay Statement<br/>Service]
        end

        subgraph "Analytics & Auth"
            RaaS[Reports As A Service<br/>Cognos Integration]
            LS[Login Service<br/>Token retrieval]
            FSSO[Federated SSO User Service]
        end
    end
```

### 3.3 HCM Authentication Matrix

```mermaid
graph TB
    subgraph "UKG Pro HCM — Authentication Methods"
        direction TB

        subgraph "Core REST API"
            BA[Basic Authentication]
            BA --> BA1[Web Service Account<br/>Username + Password]
            BA --> BA2[US-Customer-Api-Key<br/>5-char header]
        end

        subgraph "Recruiting & Onboarding REST"
            OAUTH[OAuth 2.0]
            OAUTH --> OAUTH1[Client ID + Client Secret]
            OAUTH --> OAUTH2[Token with scope header]
            OAUTH --> OAUTH3[Identity Service endpoint]
        end

        subgraph "SOAP Services"
            XML[XML Body Authentication]
            XML --> XML1[Client Access Key]
            XML --> XML2[User Access Key]
            XML --> XML3[Username + Password]
        end

        subgraph "People Analytics (RaaS)"
            BIDS[BIDataService]
            BIDS --> BIDS1[Same XML auth as SOAP]
            BIDS --> BIDS2[Requires elevated permissions<br/>Open case with UKG Support]
        end
    end
```

---

## 4. UKG Pro WFM — Deep Dive

### 4.1 REST API Domains (22 Domains)

```mermaid
graph TB
    subgraph "UKG Pro WFM REST API — 22 Domains"
        direction TB

        subgraph "Core Workforce"
            PEOPLE[People Information<br/>Employee records, pay rules]
            PERSONASSIGN[Person Assignments<br/>System entity assignments]
            BUSSTRUCT[Business Structure<br/>Org hierarchy, locations, jobs]
        end

        subgraph "Scheduling"
            SCHSETUP[Scheduling Setup<br/>Configure scheduling entities]
            SCH[Scheduling<br/>Employee time management]
            ESS[Employee Self-Service<br/>Schedule requests]
        end

        subgraph "Timekeeping"
            TK[Timekeeping<br/>Hours, accruals, vacation]
            TKT[Timekeeping Timecards<br/>Timecard management]
            TKB[Timekeeping Bulk Operations<br/>Large-scale operations]
            UDM[Universal Device Manager<br/>Device data flow]
        end

        subgraph "Attendance & Leave"
            ATT[Attendance<br/>Policy enforcement]
            LEAVE[Leave<br/>Leave policy admin]
        end

        subgraph "Forecasting"
            FCS[Forecasting Setup<br/>Configure forecasts]
            FC[Forecasting<br/>Run volume & labor forecasts]
        end

        subgraph "Analytics & Data"
            HCP[Healthcare Productivity<br/>Payroll, volume, labor metrics]
            ACT[Activities<br/>Project work tracking]
            HCM_WFM[Human Capital Mgmt<br/>Attendance, scheduling, benefits]
            IA[Information Access<br/>Ad hoc queries, data views]
        end

        subgraph "Shared / Platform"
            CR1[Common Resources I<br/>Shared across domains]
            CR2[Common Resources II<br/>Shared across domains]
            PLAT[Platform<br/>WFM-neutral capabilities]
        end
    end
```

### 4.2 WFM Authentication Matrix

```mermaid
graph TB
    subgraph "UKG Pro WFM — Authentication Methods"
        direction TB

        subgraph "Option 1: Unified UKG Auth"
            UA[Modern unified authentication<br/>Full-suite session management]
            UA --> UA1[Seamless navigation]
            UA --> UA2[No mandatory password rotation]
            UA --> UA3[Self-service account unlock]
        end

        subgraph "Option 2: OAuth 2.0"
            O2[Standard OAuth 2.0]
            O2 --> O2A[HTTPS required]
            O2 --> O2B[Access token + refresh token]
            O2 --> O2C[Refresh token: 7-day expiry]
        end

        subgraph "Federated IDP"
            FED[Federated User Accounts]
            FED --> FED1[OAuth 2.0 with external IDP]
            FED --> FED2[Access token + refresh token]
            FED --> FED3[Refresh token: 8-hour expiry]
        end

        subgraph "UI Sessions"
            UIS[Manage UI Sessions<br/>API-controlled session management]
        end
    end
```

---

## 5. Authentication Landscape — Full Map

```mermaid
graph TB
    subgraph "UKG Authentication Landscape"
        direction TB

        subgraph "UKG Pro HCM"
            subgraph "Core REST"
                H1[Basic Auth]
                H1 --> H1a[Username + Password]
                H1 --> H1b[US-Customer-Api-Key header]
            end

            subgraph "Recruiting & Onboarding"
                H2[OAuth 2.0]
                H2 --> H2a[Client ID + Secret]
                H2 --> H2b[scope header]
                H2 --> H2c[Identity Service]
            end

            subgraph "SOAP Services"
                H3[XML Token Auth]
                H3 --> H3a[Client Access Key]
                H3 --> H3b[User Access Key]
                H3 --> H3c[Username + Password]
            end

            subgraph "People Analytics"
                H4[RaaS / Cognos]
                H4 --> H4a[BIDataService]
                H4 --> H4b[Elevated permissions needed]
            end
        end

        subgraph "UKG Pro WFM"
            subgraph "Unified Auth"
                W1[UKG Authentication]
                W1 --> W1a[Full-suite sessions]
                W1 --> W1b[Self-service unlock]
            end

            subgraph "OAuth 2.0"
                W2[Standard OAuth 2.0]
                W2 --> W2a[Access Token]
                W2 --> W2b[Refresh Token — 7 days]
                W2 --> W2c[HTTPS required]
            end

            subgraph "Federated"
                W3[Federated IDP]
                W3 --> W3a[External IDP token]
                W3 --> W3b[Refresh Token — 8 hours]
            end

            subgraph "UI Sessions"
                W4[UI Session Management]
            end
        end

        subgraph "UKG Pro Platform"
            P1[Webhooks]
            P1 --> P1a[HMAC signature verification]
            P1 --> P1b[Event subscriptions]
            P1 --> P1c[Client ID + Secret]
        end

        subgraph "People Fabric (Beta)"
            PF1[Token-based Auth]
            PF1 --> PF1a[GraphQL + REST]
        end

        subgraph "UKG Ready"
            R1[Cloud REST API]
            R1 --> R1a[Separate auth flow]
        end
    end
```

---

## 6. Integration Tools

```mermaid
graph TB
    subgraph "UKG Integration Tools"
        direction TB

        subgraph "No-Code / Low-Code"
            ID[📊 Integrations Dashboard]
            ID --> ID1[Product: UKG Pro HCM]
            ID --> ID2[URL: integrations.ultipro.com]
            ID --> ID3[Run & view integration results]
            ID --> ID4[Connect UKG Pro HCM ↔ business partners]
        end

        subgraph "Cloud Integration Platform"
            IH[🔄 Integrations Hub / iHub]
            IH --> IH1[Product: UKG Pro WFM]
            IH --> IH2[Cloud-based, no install needed]
            IH --> IH3[Multi-tenant support]
            IH --> IH4[On-demand or scheduled]
            IH --> IH5[Fetch → Map/Transform → Load]
            IH --> IH6[APIs or flat files]
        end

        subgraph "Pre-Built Integrations"
            TIP[📦 Turnkey Integration Platform]
            TIP --> TIP1[Product: UKG Pro WFM]
            TIP --> TIP2[Pre-built integration templates]
            TIP --> TIP3[Services/support users only]
            TIP --> TIP4[Configure & edit integration instances]
        end
    end
```

---

## 7. Platform Services & Emerging APIs

```mermaid
graph TB
    subgraph "Platform Services & Emerging APIs"
        direction TB

        subgraph "UKG Webhooks"
            WH[Webhooks]
            WH --> WH1[Event-driven notifications]
            WH --> WH2[Near real-time delivery]
            WH --> WH3[HMAC signature verification]
            WH --> WH4[Endpoint URL configuration]
            WH --> WH5[Subscriptions: create, test, deactivate, reactivate]
            WH --> WH6[Microsoft Teams channel support]
        end

        subgraph "Webhooks Premium"
            WHP[Webhooks Premium]
            WHP --> WHP1[Advanced capabilities]
            WHP --> WHP2[API-driven management]
        end

        subgraph "People Fabric — Beta"
            PF[People Fabric API]
            PF --> PF1[GraphQL query language]
            PF --> PF2[REST-based web services]
            PF --> PF3[Suite-wide data access]
            PF --> PF4[Specific data queries]
            PF --> PF5[More efficient than REST-only]
            PF --> PF6[Status: Beta → GA coming soon]
        end
    end
```

---

## 8. Specialty Solutions

```mermaid
graph TB
    subgraph "Specialty Solutions"
        direction TB

        subgraph "People-Doc"
            PD[📋 People-Doc]
            PD --> PD1[HR Service Delivery]
            PD --> PD2[Employee request management]
            PD --> PD3[Knowledge portal]
            PD --> PD4[Process automation]
            PD --> PD5[Document storage & compliance]
            PD --> PD6[Docs: doc.people-doc.com]
        end

        subgraph "Talk"
            TK[💬 Talk]
            TK --> TK1[Frontline engagement]
            TK --> TK2[Messaging]
            TK --> TK3[Interactive features]
            TK --> TK4[Engagement tools]
            TK --> TK5[API: /talk/reference/]
        end
    end
```

---

## 9. Documentation Structure

```mermaid
graph TB
    subgraph "UKG Developer Hub Documentation"
        direction TB

        subgraph "Conceptual Documentation"
            subgraph "Getting Started"
                GS[Getting Started]
                GS --> GS1[Call Your First Pro API]
                GS --> GS2[API & Developer Tools Overview]
                GS --> GS3[Authentication & Authorization]
                GS --> GS4[Community & Support]
            end

            subgraph "Foundations"
                F[Foundations]
                F --> F1[CRUD Operations]
                F --> F2[API Resource Structure]
                F --> F3[Authentication & Security]
                F --> F4[App Keys & Versioning]
                F --> F5[Errors & Error Handling]
                F --> F6[Async Requests]
                F --> F7[i18n & Localization]
                F --> F8[Tracking, Logging & Support]
            end

            subgraph "Guides"
                G[Guides / Tutorials]
                G --> G1[Step-by-step feature guides]
                G --> G2[Valid request construction]
                G --> G3[Example responses]
            end
        end

        subgraph "Reference Documentation"
            subgraph "API Specs"
                API[API Specifications]
                API --> API1[Domain-level overview]
                API --> API2[Subgroup organization]
                API --> API3[Resource-level detail]
                API --> API4[Operation-level detail<br/>HTTP method + URL]
                API --> API5[Request & response schemas]
            end

            subgraph "Quick Starts"
                QS[Quick Start Guides]
                QS --> QS1[UKG Pro WFM Quick Start]
                QS --> QS2[20+ SOAP Service Quick Starts]
                QS --> QS3[Login Service Quick Start]
                QS --> QS4[Reports as a Service Quick Start]
                QS --> QS5[Time & Attendance Quick Start]
            end
        end
    end
```

---

## 10. Quick-Start Decision Tree

```mermaid
graph LR
    A["🤔 What do you want to integrate?"]

    A --> B["Employee data,<br/>payroll, talent,<br/>recruiting?"]
    A --> C["Scheduling, timekeeping,<br/>attendance, leave?"]
    A --> D["Events & notifications<br/>across suite?"]
    A --> E["Suite-wide data<br/>across HCM + WFM?"]
    A --> F["SMB / small business?"]
    A --> G["HR service delivery<br/>(People-Doc)?"]
    A --> H["Frontline messaging<br/>(Talk)?"]

    B --> B1["📘 UKG Pro HCM"]
    B1 --> B1a["REST API<br/>Basic Auth"]
    B1 --> B1b["SOAP Services<br/>XML Token Auth"]
    B1 --> B1c["Recruiting/Onboarding<br/>OAuth 2.0"]

    C --> C1["📗 UKG Pro WFM"]
    C1 --> C1a["REST API<br/>OAuth 2.0"]
    C1 --> C1b["Unified Auth<br/>Modern experience"]
    C1 --> C1c["Federated IDP<br/>External identity"]

    D --> D1["🔔 UKG Pro Platform"]
    D1 --> D1a["Webhooks<br/>Event-driven"]
    D1 --> D1b["HMAC verification<br/>Security"]
    D1 --> D1c["Subscriptions<br/>API management"]

    E --> E1["🧬 People Fabric"]
    E1 --> E1a["GraphQL + REST"]
    E1 --> E1b["Suite-wide data"]
    E1 --> E1c["Status: Beta"]

    F --> F1["📙 UKG Ready"]
    F1 --> F1a["Cloud REST API"]
    F1 --> F1b["SMB-focused"]

    G --> G1["📋 People-Doc"]
    G --> G1a["HR Service Delivery"]

    H --> H1["💬 Talk"]
    H --> H1a["Frontline Engagement"]
```

---

## Appendix: Key URLs

| Resource | URL |
|---|---|
| **Developer Hub** | `developer.ukg.com` |
| **HCM API Reference** | `developer.ukg.com/hcm/reference` |
| **WFM API Reference** | `developer.ukg.com/wfm/reference` |
| **Pro Platform** | `developer.ukg.com/proplatform/reference` |
| **People Fabric** | `developer.ukg.com/peoplefabric/reference` |
| **Talk API** | `developer.ukg.com/talk/reference` |
| **Integrations Dashboard** | `integrations.ultipro.com` |
| **Community Forum** | `community.ukg.com` |
| **Feature Requests** | `ukg.ideas.aha.io` |
| **Marketplace** | `marketplace.ukg.com` |
| **People-Doc Docs** | `doc.people-doc.com` |

---

## Appendix: Domain Quick Reference

### HCM REST Domains (10)

`Audit` · `Configuration Setup` · `Onboarding` · `Payroll` · `PRO Platform Config` · `PRO Security` · `PRO Employee Data` · `PRO Import Tools` · `Recruiting` · `UTA`

### HCM SOAP Services (20+)

`Employee Employment Information` · `Employee Job` · `Employee New Hire (US)` · `Employee Contacts` · `Employee User Defined Fields` · `Federated SSO User` · `Development Opportunity` · `Employee Compensation` · `Global Employee New Hire`
 · `Employee Termination` · `Employee Phone` · `Development Opportunity Participation` · `Development Opportunity Session` · `Login` · `Employee Pay Statement` · `Time Management WS` · `Employee Person` · `Time and Attendance` · `Reports
As A Service` · `Employee Address` · `Canadian New Hire`

### WFM REST Domains (22)

`Activities` · `Attendance` · `Business Structure` · `Common Resources I` · `Common Resources II` · `Employee Self-Service` · `Forecasting Setup` · `Forecasting` · `Healthcare Productivity` · `Human Capital Management` · `Information Acce
ss` · `Leave` · `People` · `Person Assignments` · `Platform` · `Scheduling Setup` · `Scheduling` · `Timekeeping Bulk Operations` · `Timekeeping Timecards` · `Timekeeping` · `Universal Device Manager`

---

*Last reviewed: 2025 · Source: [developer.ukg.com](https://developer.ukg.com)*

