# 3-Minute Design — HubSpot Multi-Account Dashboard

*Prompted and edited by Lawrence Ham*

Blueprint for a single dashboard that aggregates marketing/sales/service metrics across **multiple HubSpot portals** into one database and one UI.

## Stack

Node.js + TypeScript, TypeORM (or Prisma), PostgreSQL, `@hubspot/api-client`.

## Auth: OAuth 2.0 (required)

API keys are per-portal and deprecating — use OAuth.

1. HubSpot Developer Portal → create a **Private App** → get Client ID + Secret.
2. Set a redirect URL (e.g. `https://example.com/oauth/callback`).
3. Minimum scopes: `crm.objects.contacts.read`, `crm.objects.deals.read`, `crm.objects.companies.read`, plus tickets/communications as needed.

OAuth flow:

1. Build the auth URL with `client_id`, `redirect_uri`, `scope`.
2. On callback, receive `code`.
3. Exchange `code` → `access_token` + `refresh_token`.
4. **Encrypt and store per-portal tokens** (Node `crypto`, e.g. AES-256-GCM). Never store plaintext.

## KPIs to pull

- **Marketing**: new leads, MQLs, CAC, site traffic, email open/CTR, form submissions.
- **Sales**: new deals by pipeline, deal velocity, win rate, pipeline value, activities.
- **Service**: tickets created/closed, average response time, CSAT.

## Aggregation

Two options: (a) pull into your own **PostgreSQL/BigQuery**, or (b) BI data blending (Databox / Looker Studio). This design pulls into your own PostgreSQL.

## Architecture

```
N HubSpot portals → Backend (OAuth handler, ETL, REST API) → PostgreSQL → UI
```

- **ETL**: scheduled per-portal sync with token refresh, upsert by HubSpot ID.
- **Scheduling**: `node-cron` in dev; Lambda + EventBridge or system cron in prod.

## Visualization

Recommended **hybrid**: point a BI tool (Looker / Tableau / Power BI) at the PostgreSQL DB, or build a custom React/Vue/Angular + charts UI.
