
# DATABASE.md and DEPLOYMENT.md with actual Base44 entity model

database_content = """# EKEX Intelligence -- Database Architecture

## Overview

This document describes the database architecture for EKEX Intelligence. The system uses Base44 Entity Database (document-store, JSON Schema, MongoDB-style queries) as the primary data store, with Row-Level Security (RLS) enforced at the platform level per entity. A Supabase PostgreSQL mirror serves secondary analytics and reporting needs.

---

## Platform

| Component | Technology | Type |
|---|---|---|
| Primary Database | Base44 Entity DB | Document-store, JSON Schema |
| Mirror Database | Supabase | PostgreSQL |
| Security | Base44 RLS | Per-entity policies |

---

## Entity Map

### Supplier Domain

| Entity | Key Fields | Notes |
|---|---|---|
| `Supplier` | name, slug, email, city, country, status, primary_sector | Core supplier profile |
| `SupplierProduct` | product_name, category, price, image_url, verified_by_admin | Inventory listings |
| `ProductVariant` | product_id, variant_name, color, size, stock_quantity | SKU-level variants |
| `SupplierOrder` | product_id, quantity, order_status, payment_status | Order management |
| `SupplierAnalytics` | analytic_type, supplier_id, product_name, city | Signal/revenue events |
| `SupplierAccreditation` | supplier_id, account_type, status | Verified credentials |
| `SupplierMention` | supplier_id, signal_id, category, city, status | Demand mentions |

### Application Domain

| Entity | Key Fields | Notes |
|---|---|---|
| `SupplierApplication` | account_type, email, business_name, status | Onboarding requests |
| `SuperSupplierApplication` | supplier_id, requested_level, status | Tier upgrade requests |

### Consumer Domain

| Entity | Key Fields | Notes |
|---|---|---|
| `ConsumerSignal` | product_description, category, location, signal_state | Raw demand signals |
| `DissatisfactionSignal` | original_signal_id, reason, supplier_id | Return/gap signals |
| `DemandPin` | product_name, consumer_id, intent_type, status, visibility | Pinned demand |
| `DemandCoRequest` | original_pin_id, intent_type, co_requester_name | Group demand |
| `DemandAlert` | -- | Market gap alerts |

### Intelligence Domain

| Entity | Key Fields | Notes |
|---|---|---|
| `ProductMatch` | consumer_signal_id, match_count, top_3_shown_json | AI match results |
| `IntelligenceDelivery` | consumer_email, delivery_status, resend_message_id | Email delivery log |
| `IntelligenceReport` | user_email, product_query, full_content, credit_type | Saved reports |
| `DemandConnectionRequest` | demand_pin_id, supplier_id, consumer_id, status | Supplier <-> consumer |

### Credits & Payments

| Entity | Key Fields | Notes |
|---|---|---|
| `UserCredit` | user_email, free_reports_used, paid_credits_remaining | Credit ledger |
| `PurchaseRequest` | user_email, pack_name, price_usd, status | Payment intent log |
| `CreditPack` | name, report_count, price_usd, is_active | Pack catalog |

### Logistics

| Entity | Key Fields | Notes |
|---|---|---|
| `DeliveryRoute` | -- | Logistics route data |
| `ProcurementRequest` | -- | B2B procurement |
| `SupplierLogisticsLink` | -- | Supplier <-> logistics mapping |

---

## Entity Specifications

### ConsumerSignal

Primary demand capture entity. Stores raw consumer intent.

| Field | Type | Required | Description |
|---|---|---|---|
| id | string (auto) | Yes | Unique identifier |
| product_description | string | Yes | Free-text demand description |
| category | string | Yes | Product category classification |
| location | string | Yes | City/country of demand |
| signal_state | string | Yes | pending, matched, unmet, fulfilled, expired |
| image_url | string | No | Uploaded product image reference |
| urgency | string | No | low, medium, high, critical |
| created_by | string (auto) | Yes | Email of creating user |
| created_date | datetime (auto) | Yes | ISO timestamp |
| updated_date | datetime (auto) | Yes | Last modification |

**Automations:**
- On create: `routeSignalToSuppliers.js`
- On create: `notifySupplierOnSignal.js`
- On create: `runProductMatching.js`
- On create: `syncSignalToSupabase.js`

### Supplier

Core supplier profile entity.

| Field | Type | Required | Description |
|---|---|---|---|
| id | string (auto) | Yes | Unique identifier |
| name | string | Yes | Business name |
| slug | string | Yes | URL-friendly identifier |
| email | string | Yes | Primary contact email |
| city | string | Yes | Business city |
| country | string | Yes | ISO country code |
| status | string | Yes | pending, active, suspended, rejected |
| primary_sector | string | Yes | Main product category |
| performance_score | number | No | Historical fulfillment score |
| created_by | string (auto) | Yes | Email of creating user |
| created_date | datetime (auto) | Yes | ISO timestamp |
| updated_date | datetime (auto) | Yes | Last modification |

### SupplierProduct

Inventory listing entity.

| Field | Type | Required | Description |
|---|---|---|---|
| id | string (auto) | Yes | Unique identifier |
| product_name | string | Yes | Product name |
| category | string | Yes | Product category |
| price | number | Yes | Unit price |
| image_url | string | No | Product image reference |
| verified_by_admin | boolean | No | Admin verification status |
| supplier_id | reference | Yes | Parent Supplier |
| created_by | string (auto) | Yes | Email of creating user |
| created_date | datetime (auto) | Yes | ISO timestamp |
| updated_date | datetime (auto) | Yes | Last modification |

### ProductMatch

AI-generated match results.

| Field | Type | Required | Description |
|---|---|---|---|
| id | string (auto) | Yes | Unique identifier |
| consumer_signal_id | reference | Yes | Source ConsumerSignal |
| match_count | number | Yes | Total matches found |
| top_3_shown_json | json | Yes | JSON array of top 3 matches |
| similarity_scores | json | No | Per-match scores (0-100) |
| created_by | string (auto) | Yes | Email of creating user |
| created_date | datetime (auto) | Yes | ISO timestamp |
| updated_date | datetime (auto) | Yes | Last modification |

**Note:** As of current deployment, this entity has zero records due to an unresolved matching engine issue (see CONTRIBUTING.md Tier 1.2).

### UserCredit

Credit ledger for report access.

| Field | Type | Required | Description |
|---|---|---|---|
| id | string (auto) | Yes | Unique identifier |
| user_email | string | Yes | User identifier |
| free_reports_used | number | Yes | Count of used free reports |
| paid_credits_remaining | number | Yes | Remaining paid credits |
| created_by | string (auto) | Yes | Email of creating user |
| created_date | datetime (auto) | Yes | ISO timestamp |
| updated_date | datetime (auto) | Yes | Last modification |

---

## RLS Policy Pattern

All 23 entities follow this ownership + role-escalation pattern.

### Example: SupplierProduct

```
CREATE: role IN [supplier, producer, super_admin, operations]
READ:   created_by == user.email OR role IN [super_admin, operations]
UPDATE: created_by == user.email OR role IN [super_admin, operations]
DELETE: created_by == user.email OR role == super_admin
```

### Role Hierarchy

```
super_admin
    |
    v
operations
    |
    v
supplier / producer / logistics
    |
    v
user
```

### Role Permissions Matrix

| Role | Create | Read | Update | Delete | Admin |
|---|---|---|---|---|---|
| super_admin | All | All | All | All | Yes |
| operations | Most | All | Most | Limited | No |
| supplier | Own | Own + Public | Own | Own | No |
| producer | Own | Own + Public | Own | Own | No |
| logistics | Routes | Routes + Public | Routes | Routes | No |
| user | Consumer entities | Public + Own | Own | Own | No |

---

## Built-in Fields (All Entities)

| Field | Type | Auto-generated | Description |
|---|---|---|---|
| `id` | string | Yes | Unique identifier |
| `created_date` | ISO datetime | Yes | Creation timestamp |
| `updated_date` | ISO datetime | Yes | Last modification timestamp |
| `created_by` | string (email) | Yes | Email of creating user |

---

## Supabase Mirror Schema

The Supabase mirror replicates select entities for analytics and reporting.

| Mirrored Entity | Sync Trigger | Purpose |
|---|---|---|
| ConsumerSignal | On create | Analytics, trend detection |
| Supplier | On update | Directory search |
| SupplierProduct | On update | Inventory analytics |
| ProductMatch | On create | Match quality reporting |
| SupplierAnalytics | On create | Revenue reporting |

### Sync Function

```javascript
// syncSignalToSupabase.js
Deno.serve(async (req) => {
  const base44 = createClientFromRequest(req);
  const { signal } = await req.json();
  
  // Sync to Supabase
  const supabase = createClient(Deno.env.get('SUPABASE_URL'), Deno.env.get('SUPABASE_KEY'));
  await supabase.from('consumer_signals').insert(signal);
  
  return Response.json({ synced: true });
});
```

---

## Data Retention

| Data Type | Retention Period | Action |
|---|---|---|
| ConsumerSignal raw payloads | 90 days | Archive to file storage |
| ProductMatch results | 2 years | Aggregate after 1 year |
| SupplierAnalytics events | 2 years | Aggregate after 1 year |
| IntelligenceDelivery logs | 1 year | Delete after retention |
| PurchaseRequest records | 7 years | Legal compliance |
| Session data | 24 hours | Auto-expire (client-side) |

---

## Performance Notes

### Known Issues

| Issue | Entity | Impact | Status |
|---|---|---|---|
| ProductMatch zero records | ProductMatch | Matching engine non-functional | Investigating |
| Missing composite queries | ConsumerSignal | Slow category+location filters | Planned |
| Unmet demand clustering | DemandAlert | Slow gap detection | Planned |

### Query Optimization

1. Always include `signal_state` filter in ConsumerSignal queries (high cardinality)
2. Use `category` + `location` composite filters where possible
3. Prefer exact `city` matches over partial `location` text search
4. Index `supplier_id` references for join performance
5. Use `created_date` DESC for recent-first listings

---

## Related Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) -- System architecture and component specifications
- [DEPLOYMENT.md](DEPLOYMENT.md) -- Deployment and environment setup
- [API Reference](https://api.ekexintelligence.com) -- Interactive API documentation

---

<div align="center">

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
print("  - Official address: EKEXintelligence.com")
