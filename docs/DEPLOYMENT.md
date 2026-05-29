deployment_content = """# EKEX Intelligence -- Deployment Guide

## Overview

This document provides deployment instructions for EKEX Intelligence. The system is built on the Base44 platform with Deno Deploy Edge Functions, using Vite + React 18 for the frontend.

---

## Table of Contents

- [Infrastructure Overview](#infrastructure-overview)
- [Prerequisites](#prerequisites)
- [Frontend Deployment](#frontend-deployment)
- [Edge Functions Deployment](#edge-functions-deployment)
- [Environment Variables](#environment-variables)
- [Automations Setup](#automations-setup)
- [Domains](#domains)
- [Roles & Access](#roles--access)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)

---

## Infrastructure Overview

| Layer | Platform | Type |
|---|---|---|
| Frontend SPA | Base44 Platform | Managed hosting |
| Edge Functions | Deno Deploy | Serverless / edge |
| Database | Base44 Entity DB | Managed BaaS |
| Email | Resend | Transactional SaaS |
| Secondary DB | Supabase | Postgres mirror |
| Notifications | Command Center | Internal microservice |

---

## Prerequisites

### Required Software

| Software | Version | Purpose |
|---|---|---|
| Node.js | 18.x LTS | Frontend development |
| npm | 9+ | Package management |
| Deno | 1.40+ | Edge Function development |
| Git | 2.40+ | Source control |

### Base44 Platform Access

1. Create account at [base44.com](https://base44.com)
2. Create new project "ekex-intelligence"
3. Configure project settings

---

## Frontend Deployment

Built with **Vite + React 18**. Deployed automatically via Base44 platform.

### Local Development

```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

Frontend runs at http://localhost:5173

### Production Build

```bash
# Production build (handled by platform)
npm run build
```

Entry: `src/main.jsx` -> `src/App.jsx` (router) -> pages/

### Build Output

| File | Purpose |
|---|---|
| `src/main.jsx` | Application entry point |
| `src/App.jsx` | React Router v6 configuration |
| `src/pages/` | Route-level page components |
| `src/components/` | Reusable UI components |
| `src/hooks/` | Custom React hooks |
| `src/lib/` | Utility functions, SDK clients |

---

## Edge Functions Deployment

All 20 backend functions are deployed as Deno serverless handlers.

### Function Directory

```
functions/
|-- activateSupplierAccount.js
|-- adminGetAllProducts.js
|-- analyzeImage.js
|-- checkDuplicateAccount.js
|-- checkUserCredits.js
|-- consumeCredit.js
|-- createSupplierAccreditation.js
|-- deliverReport.js
|-- fetchProductGallery.js
|-- findSuppliersByProduct.js
|-- flagSupplierMisconduct.js
|-- notifyCommandCenter.js
|-- notifySupplierOnSignal.js
|-- onSupplierApplicationCreated.js
|-- processSupplierTag.js
|-- requestCreditPack.js
|-- routeSignalToSuppliers.js
|-- runProductMatching.js
|-- sendMatchEmail.js
|-- syncSignalToSupabase.js
```

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

### Deploy to Deno Deploy

```bash
# Install deployctl
npm install -g deployctl

# Deploy all functions
deployctl deploy --project=ekex-intelligence --include=functions/

# Deploy single function
deployctl deploy --project=ekex-intelligence --include=functions/analyzeImage.js
```

---

## Environment Variables

> Never commit secrets. All variables are set via platform dashboard -> Settings -> Environment Variables.

### Required Variables

| Variable | Used In | Description |
|---|---|---|
| `COMMAND_CENTER_URL` | notifyCommandCenter, flagSupplierMisconduct, deliverReport | Internal webhook endpoint |
| `RESEND_API_KEY` | sendMatchEmail | Resend email API key |

### Optional Variables

| Variable | Used In | Description |
|---|---|---|
| `SUPABASE_URL` | syncSignalToSupabase | Supabase project URL |
| `SUPABASE_KEY` | syncSignalToSupabase | Supabase service role key |
| `OPENAI_API_KEY` | analyzeImage, runProductMatching | LLM provider key |
| `TELEGRAM_BOT_TOKEN` | notifySupplierOnSignal | NEZA Telegram bot token |
| `WHATSAPP_API_KEY` | notifySupplierOnSignal | WhatsApp Business API key |

### Local Development

```bash
# Create local environment file
cp .env.example .env.local

# Edit with your values
nano .env.local
```

---

## Automations Setup

Automations are entity-triggered or scheduled workflows configured in the Base44 dashboard.

### Required Automations

| Name | Type | Trigger | Function | Status |
|---|---|---|---|---|
| Signal Router | Entity | ConsumerSignal created | routeSignalToSuppliers | Required |
| Supplier Notifier | Entity | ConsumerSignal created | notifySupplierOnSignal | Required |
| App Approved | Entity | SupplierApplication status=approved | notifyCommandCenter | Required |

### Automation Configuration

1. Navigate to Base44 Dashboard -> Automations
2. Create new automation
3. Select trigger type (Entity or Scheduled)
4. Configure trigger conditions
5. Select target Edge Function
6. Save and activate

---

## Domains

| Environment | URL | Status |
|---|---|---|
| Production | [EKEXintelligence.com](https://EKEXintelligence.com) | Live |
| Supplier Portal | [suppliers.ekexintelligence.com](https://suppliers.ekexintelligence.com) | Live |
| API | [api.ekexintelligence.com](https://api.ekexintelligence.com) | Live |

### Domain Configuration

1. Register domain with DNS provider
2. Add A/AAAA records pointing to Base44
3. Configure SSL certificate (auto-provisioned by Base44)
4. Set custom domain in Base44 Dashboard

---

## Roles & Access

| Role | Access Level | Description |
|---|---|---|
| super_admin | Full system access | Platform administrators |
| operations | Read/write most entities | Operations team |
| supplier | Own products, orders, analytics | Verified suppliers |
| producer | Same as supplier | Product manufacturers |
| logistics | Routes, delivery data | Logistics partners |
| user | Consumer features only | End consumers |

### Admin Portal

- URL: `/admin/*`
- Guard: `useAdminGuard` hook
- Additional lock: Identity-verified email
- Full access to all 23 entities

### Supplier Portal

- URL: `/supplier/*`
- Guard: Role check (supplier, producer)
- Access: Own products, orders, analytics

---

## Monitoring

### Base44 Dashboard

| Metric | Location | Alert |
|---|---|---|
| Entity operations | Dashboard -> Analytics | > 1000 ops/min |
| Edge Function invocations | Dashboard -> Functions | Error rate > 1% |
| Active users | Dashboard -> Analytics | Anomaly detection |

### Deno Deploy Dashboard

| Metric | Location | Alert |
|---|---|---|
| Function latency | Metrics | p95 > 500ms |
| Error rate | Logs | > 1% |
| Cold starts | Metrics | > 2s |

### Resend Dashboard

| Metric | Location | Alert |
|---|---|---|
| Delivery rate | Dashboard | < 95% |
| Bounce rate | Dashboard | > 5% |
| Open rate | Dashboard | Tracking |

### Health Check

```bash
GET /api/v1/health

Response:
{
  "status": "healthy",
  "version": "1.0.0",
  "timestamp": "2026-05-29T03:59:00Z",
  "services": {
    "base44": "connected",
    "deno_deploy": "connected",
    "resend": "connected"
  }
}
```

---

## Troubleshooting

### Common Issues

| Symptom | Cause | Solution |
|---|---|---|
| Build fails | Node version mismatch | Use Node 18.x LTS |
| Edge Function 401 | Missing auth | Verify base44.auth.me() |
| Email not sending | Invalid Resend key | Check RESEND_API_KEY |
| Automation not firing | Wrong trigger | Verify entity type and conditions |
| RLS denied | Role mismatch | Check user role and entity policies |
| Slow queries | Missing filters | Always include state/status filters |

### Log Locations

| Component | Location | Access |
|---|---|---|
| Frontend | Browser console | Client-side |
| Edge Functions | Deno Deploy logs | Dashboard |
| Automations | Base44 automation logs | Dashboard |
| Database | Base44 entity logs | Dashboard |

### Debug Mode

```bash
# Enable verbose logging
LOG_LEVEL=debug npm run dev

# Test Edge Function locally
deno run --allow-net --allow-env functions/analyzeImage.js

# Inspect Base44 entities
# Use Base44 Dashboard -> Data -> Entities
```

---

## Related Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) -- System architecture and component specifications
- [DATABASE.md](DATABASE.md) -- Entity schemas and RLS policies
- [API Reference](https://api.ekexintelligence.com) -- Interactive API documentation
- <div align="center">

&copy; 2026 Kimuntu Group &middot; EKEX Intelligence &middot; Licensed under GNU AGPLv3

[EKEXintelligence.com](https://EKEXintelligence.com) &middot; [Documentation](https://docs.ekexintelligence.com)

All Inquiries: [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

</div>
"""


---

<div align="center">

&copy; 2026 Kimuntu Group &middot; EKEX Intelligence &middot; Licensed under GNU AGPLv3

[EKEXintelligence.com](https://EKEXintelligence.com) &middot; [Documentation](https://docs.ekexintelligence.com)

All Inquiries: [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

</div>
"""

with open('/mnt/agents/output/docs/DATABASE.md', 'w') as f:
    f.write(database_content)

with open('/mnt/agents/output/docs/DEPLOYMENT.md', 'w') as f:
    f.write(deployment_content)

print("Written: docs/DATABASE.md")
print("Written: docs/DEPLOYMENT.md")
print("\n" + "="*60)
print("ALL 6 FILES GENERATED WITH ACTUAL BASE44/DENO ARCHITECTURE")
print("="*60)
print("\nRoot files:")
print("  - LICENSE")
print("  - README.md")
print("  - CONTRIBUTING.md")
print("\nDocs files:")
print("  - docs/ARCHITECTURE.md")
print("  - docs/DATABASE.md")
print("  - docs/DEPLOYMENT.md")
print("\nKey corrections from actual repo:")
print("  - Frontend: Vite + React 18 (not Next.js)")
print("  - Backend: Deno Deploy Edge Functions (not Node.js/Express)")
print("  - Database: Base44 Entity DB (not PostgreSQL primary)")
print("  - 20 Edge Functions documented with actual names")
print("  - AI: Vision LLM + LLM Agent (not custom PyTorch)")
print("  - Automations: entity-triggered workflows documented")
print("  - RLS policies: all 23 entities with role hierarchy")
print("  - No emojis, professional formatting throughout")
print("  - Unified contact: info@ekexintelligence.com")
print("  - Official ad
