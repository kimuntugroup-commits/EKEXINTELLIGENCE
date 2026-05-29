
import os
os.makedirs('/mnt/agents/output/docs', exist_ok=True)

# ARCHITECTURE.md with actual Base44/Deno stack
architecture_content = """# EKEX Intelligence -- System Architecture

## Overview

This document describes the architecture of EKEX Intelligence, an open-source intent intelligence infrastructure for local commerce. The system is built on the Base44 platform with Deno Deploy Edge Functions, using a modern React frontend and AI-powered signal matching.

---

## High-Level Architecture

```
+-----------------------------------------------------------------------------+
|                              CLIENT LAYER                                    |
|  React 18 (Vite SPA) | Tailwind CSS | shadcn/ui | Framer Motion            |
|  React Router v6 | TanStack Query v5                                           |
+-----------------------------------------------------------------------------+
                                      |
                                      | HTTPS
                                      v
+-----------------------------------------------------------------------------+
|                         PLATFORM LAYER (Base44)                              |
|                                                                              |
|  Auth | Entity DB | RLS | File Storage | Realtime | Automations              |
|  (entity-triggered + scheduled workflows)                                    |
+-----------------------------------------------------------------------------+
                                      |
                                      v
+-----------------------------------------------------------------------------+
|                      EDGE FUNCTIONS LAYER (Deno Deploy)                      |
|                                                                              |
|  20 serverless functions | @base44/sdk v0.8.25                               |
|                                                                              |
|  +----------+  +----------+  +----------+                                   |
|  |  Resend  |  | Supabase |  | Command  |                                   |
|  |  Email   |  | Mirror   |  | Center   |                                   |
|  |          |  |   DB     |  |   API    |                                   |
|  +----------+  +----------+  +----------+                                   |
+-----------------------------------------------------------------------------+
```

---

## Frontend Modules

| Module | Route | Description | Access |
|---|---|---|---|
| Home | `/` | Landing page, NEZA chat widget | Public |
| Consumer Dashboard | `/dashboard` | Demand pins, signal tracking | User |
| NEZA Chat | `/neza` | AI demand chat interface | User |
| Discover | `/discover` | Product discovery gallery | Public |
| Supplier Portal | `/supplier/*` | Dashboard, products, orders, analytics | Supplier |
| Intelligence Reports | `/reports` | Credit-gated market reports | User (credits) |
| Demand Board | `/demand-board` | Public demand marketplace | Public |
| Admin Portal | `/admin/*` | Applications, suppliers, verification | Admin |

---

## AI Intelligence Layer

```
Consumer Query
    |
    v
NEZA Chat (LLM Agent)
    |
    +---> Image Analysis (Vision LLM)
    |
    +---> ConsumerSignal saved to Base44 Entity DB
    |
    v
runProductMatching (Edge Function)
    |
    +---> Keyword extraction
    +---> Location-aware filtering (city -> country -> global)
    +---> AI similarity scoring (0-100 per product)
    |
    v
Top 3 results -> Email via Resend
```

### AI Components

| Component | Technology | Input | Output |
|---|---|---|---|
| NEZA Chat | LLM Agent | Natural language query | Structured intent |
| Image Analysis | Vision LLM | Product image | Product category, attributes |
| Product Matching | AI Scoring | ConsumerSignal + SupplierProduct | Ranked matches (0-100) |
| Keyword Extraction | NLP | Free-text description | Structured attributes |

---

## Edge Functions

All backend logic is implemented as 20 Deno Deploy serverless functions.

### Function Inventory

| # | Function | Purpose | Trigger |
|---|---|---|---|
| 1 | `activateSupplierAccount.js` | Activate verified supplier accounts | Admin action |
| 2 | `adminGetAllProducts.js` | Admin product listing | Admin request |
| 3 | `analyzeImage.js` | Vision LLM image analysis | User upload |
| 4 | `checkDuplicateAccount.js` | Prevent duplicate registrations | Registration |
| 5 | `checkUserCredits.js` | Verify user credit balance | Report request |
| 6 | `consumeCredit.js` | Deduct credit on report generation | Report delivery |
| 7 | `createSupplierAccreditation.js` | Create verified credentials | KYC completion |
| 8 | `deliverReport.js` | Send intelligence reports | Credit purchase |
| 9 | `fetchProductGallery.js` | Public product discovery | Gallery request |
| 10 | `findSuppliersByProduct.js` | Supplier search by product | User query |
| 11 | `flagSupplierMisconduct.js` | Report supplier issues | User complaint |
| 12 | `notifyCommandCenter.js` | Internal webhook routing | System events |
| 13 | `notifySupplierOnSignal.js` | Alert suppliers of demand | Signal creation |
| 14 | `onSupplierApplicationCreated.js` | Process new applications | Application submit |
| 15 | `processSupplierTag.js` | Tag processing for categorization | Product upload |
| 16 | `requestCreditPack.js` | Handle credit purchases | Purchase intent |
| 17 | `routeSignalToSuppliers.js` | Distribute signals to relevant suppliers | Automation |
| 18 | `runProductMatching.js` | AI similarity scoring | Signal creation |
| 19 | `sendMatchEmail.js` | Deliver match results via Resend | Match completion |
| 20 | `syncSignalToSupabase.js` | Mirror data to Supabase | Signal creation |

### Function Pattern

```javascript
import { createClientFromRequest } from 'npm:@base44/sdk@0.8.25';

Deno.serve(async (req) => {
  const base44 = createClientFromRequest(req);
  const user = await base44.auth.me();
  if (!user) return Response.json({ error: 'Unauthorized' }, { status: 401 });
  // ... business logic
});
```

---

## Security Model

| Layer | Measure | Implementation |
|---|---|---|
| Admin access | Identity-locked | Single verified email |
| Role hierarchy | Escalation pattern | super_admin -> operations -> supplier/producer/logistics -> user |
| Frontend guards | Route protection | useAdminGuard hook on all `/admin/*` routes |
| Backend guards | Authorization | Email + role check on every sensitive function |
| Entity RLS | Data access control | Per-entity create/read/update/delete policies |
| Secrets management | Environment isolation | All API keys in Deno environment only |
| No client secrets | Frontend security | No API keys exposed in browser bundle |

### RLS Policy Pattern

```
Entity: SupplierProduct (example)

CREATE: role IN [supplier, producer, super_admin, operations]
READ:   created_by == user.email OR role IN [super_admin, operations]
UPDATE: created_by == user.email OR role IN [super_admin, operations]
DELETE: created_by == user.email OR role == super_admin
```

All 23 entities follow this ownership + role-escalation pattern.

---

## Notification Channels

| Channel | Provider | Used For |
|---|---|---|
| Email | Resend | Match results, report delivery, approvals |
| WhatsApp | NEZA Bot | Supplier demand alerts |
| Telegram | NEZA Bot | Supplier demand alerts |
| In-App | Base44 Realtime | Real-time signal mentions |
| Internal | Command Center | Webhook routing, system events |

---

## Data Flow: Intent to Fulfillment

```
Step 1: Consumer submits query (image/text) via NEZA Chat
    |
Step 2: analyzeImage.js (if image) or LLM Agent (if text)
    |       -> Extracts structured intent
    |
Step 3: ConsumerSignal entity created in Base44 Entity DB
    |       -> Automation triggered
    |
Step 4: routeSignalToSuppliers.js (automation)
    |       -> Location-aware filtering
    |       -> Category matching
    |
Step 5: runProductMatching.js (automation)
    |       -> Keyword extraction
    |       -> AI similarity scoring (0-100)
    |       -> Top 3 ranking
    |
Step 6: sendMatchEmail.js
    |       -> Resend API delivers results to consumer
    |
Step 7: notifySupplierOnSignal.js (automation)
    |       -> WhatsApp/Telegram alert to matched suppliers
    |
Step 8: Supplier responds via Supplier Portal
    |
Step 9: Fulfillment tracked via SupplierOrder entity
    |
Step 10: MatchFeedback captured for learning loop
```

---

## Technology Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | Vite + React 18 | 18.x | SPA build system |
| Styling | Tailwind CSS + shadcn/ui | 3.x | Utility-first CSS, components |
| Routing | React Router v6 | 6.x | Client-side navigation |
| State | TanStack Query v5 | 5.x | Server state management |
| Animation | Framer Motion | 11.x | UI transitions |
| Backend | Deno Deploy | 1.40+ | Serverless Edge Functions |
| SDK | @base44/sdk | 0.8.25 | Platform integration |
| Database | Base44 Entity DB | Latest | Document-store primary DB |
| Mirror DB | Supabase (PostgreSQL) | 15+ | Analytics and reporting |
| Auth | Base44 Auth | Latest | Identity management |
| Email | Resend | Latest | Transactional email |
| Notifications | Command Center | Internal | Webhook routing |
| AI | Vision LLM + LLM Agent | Latest | Image analysis, NLP |
| Storage | Base44 File Storage | Latest | Image uploads, documents |
| Realtime | Base44 Realtime | Latest | Live updates |
| Automations | Base44 Automations | Latest | Entity-triggered workflows |

---

## Scalability Considerations

| Component | Scaling Strategy | Current Limit |
|---|---|---|
| Frontend | CDN + static hosting | Base44 managed |
| Edge Functions | Deno Deploy auto-scale | 20 functions |
| Entity DB | Base44 managed | Document-store |
| Supabase Mirror | Read replicas | Single project |
| File Storage | Base44 managed | Unlimited |
| Realtime | WebSocket connections | Base44 managed |

---

## Monitoring & Observability

| Metric | Tool | Alert Threshold |
|---|---|---|
| Edge Function latency | Deno Deploy metrics | p95 > 500ms |
| Error rate | Deno Deploy logs | > 1% |
| Entity DB operations | Base44 dashboard | > 1000 ops/min |
| Email delivery | Resend dashboard | Bounce rate > 5% |
| Match accuracy | Custom dashboard | < 75% precision |

---

## Related Documentation

- [DATABASE.md](DATABASE.md) -- Entity schemas, RLS policies, and data model
- [DEPLOYMENT.md](DEPLOYMENT.md) -- Deployment guides and environment setup
- [API Reference](https://api.ekexintelligence.com) -- Interactive API documentation

---

<div align="center">

&copy; 2026 Kimuntu Group &middot; EKEX Intelligence &middot; Licensed under GNU AGPLv3

[EKEXintelligence.com](https://EKEXintelligence.com) &middot; [Documentation](https://docs.ekexintelligence.com)

All Inquiries: [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

</div>
"""

with open('/mnt/agents/output/docs/ARCHITECTURE.md', 'w') as f:
    f.write(architecture_content)

print("Written: docs/ARCHITECTURE.md")
