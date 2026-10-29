# Scalability Blueprint: AI Timecard Processing

*Prompted and edited by Lawrence Ham*

**Volume: 10,000 timecards / week.** Processing 10k handwritten timecards weekly at the branch level is a massive manual bottleneck. Using **Gemini 2.5 Flash** shifts the model from labor-intensive manual entry to high-speed automated validation.

## Financial Impact (Weekly)

Manual entry ≈ **$1.50/card** (labor, errors, corrections).

| Metric | Manual | AI Real-time | AI Batch (Optimized) |
| :--- | :--- | :--- | :--- |
| **Processing Time** | ~500 hrs | ~2.5 hrs compute | ~2.5 hrs compute |
| **Cost / Card** | $1.50 | $0.0019 | **$0.00095** |
| **Weekly Total** | **$15,000** | **$19.00** | **$9.50** |
| **Annual Savings** | — | **$779,012** | **$779,506** |

## Scalable Processing Architecture

At this volume, move from single-threaded to a **parallel batch architecture** to avoid the Friday-afternoon bottleneck when branches upload.

- **Volume routing:** queue `> 1000` → trigger **Batch API** (50% discount); `< 100` → real-time API.
- Flow: branch uploads (PDF/JPG) → inbound queue → Gemini 2.5 Flash → structured JSON → Pinpoint matching engine → match Associate ID + Client ID → update MongoDB → ready for payroll export.

## Cost Breakdown: Tokens & Caching

To hit the **$9.50/week** target, use **Context Caching** — store the prompt "Instructions" in Google's cache so those tokens aren't re-billed per card.

- Weekly token consumption ≈ **15M tokens**: ~40% cached instructions, ~45% new input (timecard images), ~15% output (JSON).
- **Caching** cuts per-request input cost ~40%; **Batch** cuts the remaining ~50%.

## Technical Strategy for 10k/Week

**A. Parallel consumer.** Scale the RabbitMQ consumer to multiple workers — recommend **5–10 concurrent Node.js workers**. At ~2s/card, 5 workers process all 10k cards in **~1.1 hours**.

**B. Matching (Bullhorn/PeopleSoft).** At 10k records, fuzzy matching gets risky. Use a **tiered confidence score**:
- `> 95%` → auto-match to Bullhorn
- `70–94%` → flag for human review
- `< 70%` → reject / re-scan

**C. Database optimization.** The `timesheets` collection grows fast (40k+ records/month).
- **Retention:** move images to cold storage (S3/GCS) after 60 days.
- **Indexing:** index `clientId`, `associateId`, `weekEnding` to keep the Angular UI snappy.

## Roadmap to Scale

| Phase | Volume | Target | Focus |
| :--- | :--- | :--- | :--- |
| **1** | 1,000/wk | Prototype | Logic accuracy & webhook reliability |
| **2** | 5,000/wk | Pilot | Context caching + RabbitMQ scaling |
| **3** | 10,000/wk | Production | Batch API integration + auto-matching to Bullhorn |

## Final Verdict

At 10k cards/week, the **AI cost ($9.50)** is noise next to the **500 labor hours saved**. Engineering focus should go to **matching-engine accuracy** (linking names → Bullhorn IDs), not AI cost.
