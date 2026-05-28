# EKEX Intelligence — Database Schema Documentation

**Version:** 1.0  
**Date:** 2026-05-28  
**Author:** Kimuntu Group  
**Status:** Reference Implementation

---

## Table of Contents

1. [Overview](#overview)
2. [Database Architecture](#database-architecture)
3. [Entity Schemas](#entity-schemas)
4. [Relationships](#relationships)
5. [Indexing Strategy](#indexing-strategy)
6. [Migrations](#migrations)
7. [Data Types & Constraints](#data-types--constraints)

---

## Overview

EKEX Intelligence uses **PostgreSQL 14+** as the primary database with **15 entity schemas** defined from first principles. All schemas follow these principles:

- **Schema-First Design**: Database structure drives system design
- **Type Safety**: Explicit column types, constraints, and validation
- **ACID Transactions**: Full transaction support for critical workflows
- **Audit Trail**: All entities include `created_at` and `updated_at` timestamps
- **Soft Deletes**: Entities use `deleted_at` for data retention compliance

---

## Database Architecture

### Storage Layers

```
┌────────────────────────────────────────────────────┐
│           PostgreSQL Primary Database               │
│  (EKEX Intelligence Core — Public Schema)           │
├────────────────────────────────────────────────────┤
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │          PUBLIC SCHEMA (AGPLv3)              │  │
│  │                                              │  │
│  │  • Entity tables (Consumer, Supplier, etc.)  │  │
│  │  • Signal tables (ConsumerSignal, etc.)      │  │
│  │  • Lookup tables (Location, Category)        │  │
│  │  • Match & Feedback tables                   │  │
│  │  • Admin audit tables                        │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │    INDICES & FULL-TEXT SEARCH                │  │
│  │  (Optimized for matching & analytics)        │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │    JSONB COLUMNS (Flexible Metadata)         │  │
│  │  (For signal attributes, custom fields)      │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
└────────────────────────────────────────────────────┘
```

### Connection Details

| Parameter | Value | Notes |
|-----------|-------|-------|
| Host | `localhost` (dev) / RDS endpoint (prod) | Set via `.env` |
| Port | `5432` | Standard PostgreSQL port |
| Database | `ekex_intelligence` | Main application DB |
| Pool Size | 20 (dev) / 50 (prod) | Connection pooling |
| SSL Mode | `disable` (dev) / `require` (prod) | Production enforces TLS |

---

## Entity Schemas

### Core User Entities

#### **Consumer**

Represents an end-user who submits demand signals.

```sql
CREATE TABLE "Consumer" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone VARCHAR(20) NOT NULL UNIQUE,
  email VARCHAR(255) UNIQUE,
  location_id UUID NOT NULL REFERENCES "Location"(id),
  status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
  verified_at TIMESTAMP,
  last_signal_at TIMESTAMP,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP,
  
  CONSTRAINT valid_phone CHECK (phone ~ '^\+?[0-9]{10,15}$')
);

CREATE INDEX idx_consumer_phone ON "Consumer"(phone);
CREATE INDEX idx_consumer_location ON "Consumer"(location_id);
CREATE INDEX idx_consumer_status ON "Consumer"(status) WHERE deleted_at IS NULL;
```

#### **Supplier**

Represents a business/organization that provides products/services.

```sql
CREATE TABLE "Supplier" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255),
  phone VARCHAR(20),
  location_id UUID NOT NULL REFERENCES "Location"(id),
  status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
  tier ENUM('bronze', 'silver', 'gold', 'platinum') DEFAULT 'bronze',
  verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP,
  registration_source ENUM('web', 'telegram', 'api') DEFAULT 'web',
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_supplier_location ON "Supplier"(location_id);
CREATE INDEX idx_supplier_status ON "Supplier"(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_supplier_tier ON "Supplier"(tier);
CREATE INDEX idx_supplier_verified ON "Supplier"(verified);
```

---

### Signal & Demand Entities

#### **ConsumerSignal** (Layer 1)

Raw demand signal submitted by consumer through any channel (web, Telegram, API).

```sql
CREATE TABLE "ConsumerSignal" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  consumer_id UUID NOT NULL REFERENCES "Consumer"(id),
  description TEXT NOT NULL,
  category_id UUID REFERENCES "Category"(id),
  location_id UUID NOT NULL REFERENCES "Location"(id),
  signal_state ENUM(
    'submitted',
    'duplicate_pending',
    'deduplicated',
    'valid',
    'flooded',
    'processed'
  ) DEFAULT 'submitted',
  source ENUM('web', 'telegram', 'api', 'ussd') DEFAULT 'web',
  source_metadata JSONB DEFAULT '{}',
  
  -- Deduplication tracking (Layer 1, Gates A-E)
  gate_result ENUM(
    'gate_a_technical_duplicate',
    'gate_b_return_after_match',
    'gate_c_genuine_submission',
    'gate_d_velocity_flood',
    'gate_e_default_pass'
  ),
  duplicate_of_signal_id UUID REFERENCES "ConsumerSignal"(id),
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_consumer_signal_consumer ON "ConsumerSignal"(consumer_id);
CREATE INDEX idx_consumer_signal_category ON "ConsumerSignal"(category_id);
CREATE INDEX idx_consumer_signal_location ON "ConsumerSignal"(location_id);
CREATE INDEX idx_consumer_signal_state ON "ConsumerSignal"(signal_state);
CREATE INDEX idx_consumer_signal_source ON "ConsumerSignal"(source);
CREATE INDEX idx_consumer_signal_created ON "ConsumerSignal"(created_at DESC);
```

#### **DemandObject** (Layer 2)

Structured, normalized representation of consumer demand after processing.

```sql
CREATE TABLE "DemandObject" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  consumer_signal_id UUID NOT NULL UNIQUE REFERENCES "ConsumerSignal"(id),
  category_id UUID NOT NULL REFERENCES "Category"(id),
  location_id UUID NOT NULL REFERENCES "Location"(id),
  
  -- Normalized fields
  description_normalized TEXT NOT NULL,
  quantity_min INT,
  quantity_max INT,
  price_min DECIMAL(12, 2),
  price_max DECIMAL(12, 2),
  
  -- Scoring & confidence
  intent_score DECIMAL(3, 2) DEFAULT 0.5,  -- 0-1 confidence (TODO: implement real scoring)
  reliability_source DECIMAL(3, 2) DEFAULT 0.5,  -- Source trust metric
  
  -- Metadata
  quality_indicators JSONB DEFAULT '{}',  -- Custom quality flags
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_demand_category ON "DemandObject"(category_id);
CREATE INDEX idx_demand_location ON "DemandObject"(location_id);
CREATE INDEX idx_demand_intent_score ON "DemandObject"(intent_score DESC);
```

#### **SupplyObject** (Layer 2)

Structured representation of supplier product/service capability.

```sql
CREATE TABLE "SupplyObject" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  supplier_product_id UUID NOT NULL UNIQUE REFERENCES "SupplierProduct"(id),
  category_id UUID NOT NULL REFERENCES "Category"(id),
  location_id UUID NOT NULL REFERENCES "Location"(id),
  
  -- Product details (normalized)
  name_normalized VARCHAR(255) NOT NULL,
  quantity_available INT,
  price DECIMAL(12, 2),
  quality_tier ENUM('basic', 'standard', 'premium', 'luxury') DEFAULT 'standard',
  availability ENUM('in_stock', 'limited', 'backorder', 'discontinued') DEFAULT 'in_stock',
  lead_time_days INT,
  
  -- Supplier reliability
  supplier_reliability_score DECIMAL(3, 2) DEFAULT 0.5,  -- 0-1 score
  fulfillment_rate DECIMAL(3, 2),  -- Based on historical matching
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_supply_category ON "SupplyObject"(category_id);
CREATE INDEX idx_supply_location ON "SupplyObject"(location_id);
CREATE INDEX idx_supply_availability ON "SupplyObject"(availability);
```

---

### Matching Entities

#### **ProductMatch** (Layer 3)

Result of matching a DemandObject to a SupplyObject. **⚠️ Currently: 0 records (matching engine not yet firing successfully)**

```sql
CREATE TABLE "ProductMatch" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  demand_object_id UUID NOT NULL REFERENCES "DemandObject"(id),
  supply_object_id UUID NOT NULL REFERENCES "SupplyObject"(id),
  
  -- Match classification
  match_type ENUM('direct', 'partial', 'inferred') NOT NULL,
  similarity_score DECIMAL(3, 2) NOT NULL,  -- 0-1 semantic match
  location_score DECIMAL(3, 2) NOT NULL,    -- 0-1 geographic match
  temporal_score DECIMAL(3, 2) NOT NULL,    -- 0-1 time relevance
  reliability_score DECIMAL(3, 2) NOT NULL, -- 0-1 supplier history
  
  -- Final ranking
  overall_score DECIMAL(3, 2) GENERATED ALWAYS AS (
    (similarity_score * 0.4 + location_score * 0.2 + 
     temporal_score * 0.2 + reliability_score * 0.2)
  ) STORED,
  
  -- Match metadata
  matching_engine_version VARCHAR(20),
  matching_algorithm ENUM('base44_ai', 'semantic', 'hybrid'),
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_product_match_demand ON "ProductMatch"(demand_object_id);
CREATE INDEX idx_product_match_supply ON "ProductMatch"(supply_object_id);
CREATE INDEX idx_product_match_overall_score ON "ProductMatch"(overall_score DESC);
CREATE INDEX idx_product_match_created ON "ProductMatch"(created_at DESC);
```

#### **MatchHistory**

Audit log for all match state changes.

```sql
CREATE TABLE "MatchHistory" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_match_id UUID NOT NULL REFERENCES "ProductMatch"(id),
  
  action ENUM(
    'created',
    'score_updated',
    'status_changed',
    'consumer_notified',
    'supplier_notified',
    'closed'
  ) NOT NULL,
  
  actor_type ENUM('system', 'admin', 'consumer', 'supplier') DEFAULT 'system',
  actor_id UUID,
  
  old_values JSONB,
  new_values JSONB,
  
  timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_match_history_match ON "MatchHistory"(product_match_id);
CREATE INDEX idx_match_history_action ON "MatchHistory"(action);
CREATE INDEX idx_match_history_timestamp ON "MatchHistory"(timestamp DESC);
```

---

### Gap Detection Entities

#### **MarketGapAlert** (Layer 4)

Alert generated when unmet demand is detected.

```sql
CREATE TABLE "MarketGapAlert" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  consumer_signal_id UUID NOT NULL REFERENCES "ConsumerSignal"(id),
  
  gap_type ENUM('unmet_demand', 'underserved', 'emerging') NOT NULL,
  severity ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
  
  -- Geographic scope
  location_id UUID NOT NULL REFERENCES "Location"(id),
  
  -- Assignment & tracking
  assigned_to UUID REFERENCES "AdminUser"(id),
  status ENUM('open', 'investigating', 'resolved', 'wontfix') DEFAULT 'open',
  
  -- Incentivization
  bounty_amount DECIMAL(10, 2),
  contributor_recruited BOOLEAN DEFAULT FALSE,
  
  metadata JSONB DEFAULT '{}',
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  resolved_at TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_gap_alert_location ON "MarketGapAlert"(location_id);
CREATE INDEX idx_gap_alert_gap_type ON "MarketGapAlert"(gap_type);
CREATE INDEX idx_gap_alert_status ON "MarketGapAlert"(status);
CREATE INDEX idx_gap_alert_created ON "MarketGapAlert"(created_at DESC);
```

---

### Feedback Entities

#### **DissatisfactionSignal** (Layer 5)

Signal indicating consumer dissatisfaction with a match.

```sql
CREATE TABLE "DissatisfactionSignal" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_match_id UUID NOT NULL REFERENCES "ProductMatch"(id),
  consumer_id UUID NOT NULL REFERENCES "Consumer"(id),
  
  reason ENUM(
    'return_quality_issue',
    'return_wrong_product',
    'return_damaged',
    'undelivered',
    'delayed_delivery',
    'price_mismatch',
    'other'
  ) NOT NULL,
  
  severity ENUM('minor', 'moderate', 'severe') DEFAULT 'moderate',
  details TEXT,
  
  -- Learning impact
  impact_on_supplier BOOLEAN DEFAULT FALSE,
  impact_on_category BOOLEAN DEFAULT FALSE,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_dissatisfaction_match ON "DissatisfactionSignal"(product_match_id);
CREATE INDEX idx_dissatisfaction_consumer ON "DissatisfactionSignal"(consumer_id);
CREATE INDEX idx_dissatisfaction_reason ON "DissatisfactionSignal"(reason);
```

#### **FeedbackLoop**

Tracks confidence score adjustments and system learning.

```sql
CREATE TABLE "FeedbackLoop" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  entity_type ENUM('supplier', 'consumer', 'category', 'location') NOT NULL,
  entity_id UUID NOT NULL,
  
  feedback_type ENUM(
    'temporal_decay',
    'confidence_adjustment',
    'reliability_update',
    'fulfillment_rate_change'
  ) NOT NULL,
  
  score_delta DECIMAL(3, 2),  -- Change in score
  old_score DECIMAL(3, 2),
  new_score DECIMAL(3, 2),
  
  reason_dissatisfaction_signal_id UUID REFERENCES "DissatisfactionSignal"(id),
  
  applied_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_feedback_loop_entity ON "FeedbackLoop"(entity_type, entity_id);
CREATE INDEX idx_feedback_loop_feedback_type ON "FeedbackLoop"(feedback_type);
```

---

### Lookup Tables

#### **Category**

Product/service categories with hierarchical support.

```sql
CREATE TABLE "Category" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL UNIQUE,
  description TEXT,
  parent_id UUID REFERENCES "Category"(id),
  
  -- Metadata
  is_featured BOOLEAN DEFAULT FALSE,
  unknown_rate DECIMAL(3, 2) DEFAULT 0.28,  -- TODO: Reduce from 28% to <5%
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_category_parent ON "Category"(parent_id);
CREATE INDEX idx_category_featured ON "Category"(is_featured) WHERE is_featured = true;
```

#### **Location**

Geographic locations with standardization.

```sql
CREATE TABLE "Location" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  city VARCHAR(100) NOT NULL,
  country VARCHAR(100) NOT NULL,
  
  -- Coordinates
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  
  -- Metadata
  is_featured BOOLEAN DEFAULT FALSE,
  region VARCHAR(100),
  timezone VARCHAR(50),
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_location UNIQUE(city, country)
);

CREATE INDEX idx_location_city ON "Location"(city);
CREATE INDEX idx_location_country ON "Location"(country);
CREATE INDEX idx_location_featured ON "Location"(is_featured) WHERE is_featured = true;
CREATE INDEX idx_location_geo ON "Location" USING GIST(
  ll_to_earth(latitude, longitude)
) WHERE latitude IS NOT NULL AND longitude IS NOT NULL;
```

---

### Admin Entities

#### **AdminUser**

Platform administrators with role-based access.

```sql
CREATE TABLE "AdminUser" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  
  role ENUM('superadmin', 'admin', 'moderator', 'analyst') DEFAULT 'admin',
  permissions JSONB DEFAULT '{}',
  
  last_login TIMESTAMP,
  last_login_ip INET,
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_admin_email ON "AdminUser"(email);
CREATE INDEX idx_admin_role ON "AdminUser"(role);
```

#### **SupplierProduct**

Products offered by suppliers (intermediate table for many-to-many).

```sql
CREATE TABLE "SupplierProduct" (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  supplier_id UUID NOT NULL REFERENCES "Supplier"(id),
  category_id UUID NOT NULL REFERENCES "Category"(id),
  
  name VARCHAR(255) NOT NULL,
  description TEXT,
  price_range_min DECIMAL(12, 2),
  price_range_max DECIMAL(12, 2),
  
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP,
  
  CONSTRAINT unique_supplier_product UNIQUE(supplier_id, category_id, name)
);

CREATE INDEX idx_supplier_product_supplier ON "SupplierProduct"(supplier_id);
CREATE INDEX idx_supplier_product_category ON "SupplierProduct"(category_id);
```

---

## Relationships

### Core Flow

```
Consumer (1) ──→ (M) ConsumerSignal
              ──→ (M) DemandObject (via Signal)
              
Supplier (1) ──→ (M) SupplierProduct
              ──→ (M) SupplyObject (via Product)

DemandObject (M) ──→ (M) SupplyObject : ProductMatch (Junction)

ProductMatch (1) ──→ (M) MatchHistory
               ──→ (M) DissatisfactionSignal
               
DissatisfactionSignal (M) ──→ (1) FeedbackLoop
```

---

## Indexing Strategy

### Primary Performance Indexes

| Table | Index | Columns | Purpose |
|-------|-------|---------|---------|
| ConsumerSignal | idx_consumer_signal_state | signal_state | Find signals by status |
| DemandObject | idx_demand_intent_score | intent_score DESC | Rank by confidence |
| SupplyObject | idx_supply_location | location_id | Geographic queries |
| ProductMatch | idx_product_match_overall_score | overall_score DESC | Best matches first |
| MarketGapAlert | idx_gap_alert_location + gap_type | location_id, gap_type | Gap detection analysis |

### Full-Text Search

For signal descriptions and product names:

```sql
-- Add GIN indexes for full-text search
CREATE INDEX idx_consumer_signal_search ON "ConsumerSignal" 
  USING GIN(to_tsvector('english', description));

CREATE INDEX idx_supplier_product_search ON "SupplierProduct"
  USING GIN(to_tsvector('english', name || ' ' || COALESCE(description, '')));
```

---

## Migrations

### Migration Workflow

```bash
# Generate new migration
npm run db:create-migration -- --name <migration_name>

# Run pending migrations
npm run db:migrate

# Rollback last migration
npm run db:migrate:rollback

# Reset database (dev only)
npm run db:reset
```

### Schema Versioning

Migrations follow semantic versioning:
- Format: `YYYYMMDD_HHmmss_<description>.sql`
- Example: `20260528_143000_add_consumer_signal_gates.sql`

---

## Data Types & Constraints

### Enums

All enum columns use PostgreSQL native ENUM types for consistency:

```sql
-- Signal states
CREATE TYPE signal_state AS ENUM(
  'submitted', 'duplicate_pending', 'deduplicated', 'valid', 'flooded', 'processed'
);

-- Match types
CREATE TYPE match_type AS ENUM('direct', 'partial', 'inferred');

-- Gap types
CREATE TYPE gap_type AS ENUM('unmet_demand', 'underserved', 'emerging');
```

### Constraints

| Constraint | Purpose |
|-----------|---------|
| NOT NULL | Enforce data completeness |
| UNIQUE | Prevent duplicates (phone, email, etc.) |
| FOREIGN KEY | Maintain referential integrity |
| CHECK | Validate domain constraints (phone format, scores 0-1) |
| DEFAULT | Set sensible defaults for new records |

### Decimal Precision

Scores use `DECIMAL(3, 2)` for accuracy:
- Range: 0.00 to 1.00
- Precision: 2 decimal places
- Example: `0.85` = 85% confidence

---

## Appendix: Quick Reference

### Connection String

```
postgresql://user:password@localhost:5432/ekex_intelligence
```

### Total Tables

- **Core Entities**: Consumer, Supplier, AdminUser
- **Signals**: ConsumerSignal, DemandObject, SupplyObject, SupplierProduct
- **Matching**: ProductMatch, MatchHistory
- **Gap Detection**: MarketGapAlert
- **Feedback**: DissatisfactionSignal, FeedbackLoop
- **Lookup**: Category, Location

**Total: 15 entity schemas** ✓

---

<div align="center">

**[← Back to README](../README.md)** · **[Architecture Documentation](./ARCHITECTURE.md)**

*© 2026 Kimuntu Group · EKEX Intelligence*

</div>
