# EKEX Intelligence — Architecture Documentation

**Version:** 1.0  
**Date:** 2026-05-28  
**Author:** Kimuntu Group  
**Status:** Production (Partial Implementation)

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Architectural Principles](#2-architectural-principles)
3. [The Five Protocol Layers](#3-the-five-protocol-layers)
4. [System Architecture Diagram](#4-system-architecture-diagram)
5. [Data Flow Diagrams](#5-data-flow-diagrams)
6. [Technology Stack](#6-technology-stack)
7. [Security Architecture](#7-security-architecture)
8. [Scalability & Performance](#8-scalability--performance)
9. [Commercial Module Boundaries](#9-commercial-module-boundaries)
10. [Deployment Architecture](#10-deployment-architecture)

---

## 1. System Overview

EKEX Intelligence is a **five-layer demand-supply coordination protocol** built as an open-core digital infrastructure for African B2B markets. The system captures unstructured consumer demand signals, transforms them into structured intelligence, matches them against verified supplier inventories, detects market gaps, and learns from fulfillment outcomes to continuously improve.

### Core Design Philosophy

- **Protocol-First:** The system is designed as a protocol specification first, implementation second
- **Open-Core:** Core protocol layers are AGPLv3; commercial integrations are proprietary
- **Africa-Optimized:** Built for low-bandwidth, mobile-first, multi-language African markets
- **Closed-Loop:** Every signal flows through capture → structure → match → fulfill → learn

---

## 2. Architectural Principles

| Principle | Description | Implementation |
|---|---|---|
| **Layer Isolation** | Each protocol layer operates independently with defined interfaces | Separate modules with strict API contracts |
| **Event-Driven** | State changes propagate via events, not direct coupling | Redis Pub/Sub + WebSocket |
| **Schema-First** | Database schemas define the domain model before code | 15 entity schemas with migrations |
| **Mobile-First** | All interfaces optimized for low-bandwidth mobile | Responsive UI, compressed payloads |
| **Multi-Channel** | Signals accepted from web, Telegram, USSD, API | Unified ingestion pipeline |
| **Privacy-By-Design** | Data protection compliant with Rwanda Law 058/2021 | Encryption, anonymization, audit trails |

---

## 3. The Five Protocol Layers

### Layer 1: Signal Capture
**Status:** 20% Implemented | **Priority:** CRITICAL

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAYER 1: SIGNAL CAPTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐ │
│   │   Web    │   │ Telegram │   │   API    │   │  USSD*   │ │
│   │  Portal  │   │   NEZA   │   │  Client  │   │ (future) │ │
│   └────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘ │
│        │              │              │              │         │
│        └──────────────┴──────────────┴──────────────┘         │
│                         │                                     │
│              ┌──────────┴──────────┐                          │
│              │   Signal Ingestion    │                          │
│              │      Pipeline         │                          │
│              │  (Validation + Dedupl)│                          │
│              └──────────┬──────────┘                          │
│                         │                                     │
│              ┌──────────┴──────────┐                          │
│              │   ConsumerSignal      │                          │
│              │   (Raw Signal Entity) │                          │
│              └───────────────────────┘                          │
│                                                                 │
│   Gates: A(duplicate) → B(return) → C(genuine) → D(flood) → E │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**
- Web consumer portal (React/Next.js)
- Telegram Bot "NEZA" (Node.js + Telegram API)
- REST/GraphQL API for third-party integrations
- 5-gate deduplication engine
- Signal state machine

**Key Entities:** `ConsumerSignal`, `SignalState`

---

### Layer 2: Normalization & Structuring
**Status:** 25% Implemented | **Priority:** CRITICAL

```
┌─────────────────────────────────────────────────────────────────┐
│              LAYER 2: NORMALIZATION & STRUCTURING               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────┐                                          │
│   │ ConsumerSignal  │                                          │
│   │  (Raw Input)    │                                          │
│   └────────┬────────┘                                          │
│            │                                                    │
│   ┌────────┴────────┐                                          │
│   │  Location Parser │  → Standardize city/country              │
│   │  Category Parser │  → Classify product category             │
│   │  Deduplicator    │  → Remove duplicates (Gates A-E)         │
│   │  Confidence      │  → Compute intent_score                  │
│   │   Calculator     │                                          │
│   └────────┬────────┘                                          │
│            │                                                    │
│   ┌────────┴──────────────────────────┐                        │
│   │                                   │                        │
│   ▼                                   ▼                        │
│ ┌──────────────┐              ┌──────────────┐               │
│ │ DemandObject │              │ SupplyObject │               │
│ │ (Structured  │              │ (Supplier    │               │
│ │   Demand)     │              │   Product)   │               │
│ └──────────────┘              └──────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**
- Location standardization (10 featured cities + global)
- Category classification (currently 28% "Unknown" — needs fix)
- Deduplication engine (Gates A-E)
- Confidence scoring (currently hardcoded to 1 — needs implementation)

**Key Entities:** `DemandObject`, `SupplyObject`, `Location`, `Category`

---

### Layer 3: Matching Engine
**Status:** 10% Implemented | **Priority:** CRITICAL

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAYER 3: MATCHING ENGINE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────────┐      ┌──────────────┐                     │
│   │ DemandObject │      │ SupplyObject │                     │
│   │  (Consumer   │      │  (Supplier   │                     │
│   │   Request)   │      │   Product)   │                     │
│   └──────┬───────┘      └──────┬───────┘                     │
│          │                     │                               │
│          └──────────┬──────────┘                               │
│                     │                                          │
│          ┌──────────┴──────────┐                              │
│          │   Matching Engine     │                              │
│          │   (Base44 Native AI)  │                              │
│          │                       │                              │
│          │  ┌─────────────────┐  │                              │
│          │  │ Similarity Score│  │  → Text/semantic matching   │
│          │  │ Location Weight │  │  → City → Country fallback  │
│          │  │ Temporal Prox.  │  │  → Time sensitivity           │
│          │  │ Reliability     │  │  → Supplier history           │
│          │  └─────────────────┘  │                              │
│          └──────────┬──────────┘                              │
│                     │                                          │
│          ┌──────────┴──────────┐                              │
│          │    ProductMatch       │                              │
│          │  (Match Result Entity)│                              │
│          │                       │                              │
│          │  match_type: DIRECT | │                              │
│          │            PARTIAL |  │                              │
│          │            INFERRED   │                              │
│          └───────────────────────┘                              │
│                                                                 │
│   ⚠️  CURRENT STATUS: ProductMatch table has ZERO records        │
│       Engine deployed but never fired successfully               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**
- Base44 Native AI matching engine
- Similarity scoring algorithm
- Location-weighted fallback (2-tier: city → country)
- Match type classifier (Direct, Partial, Inferred)

**Key Entities:** `ProductMatch`, `MatchHistory`

---

### Layer 4: Gap Detection
**Status:** 5% Implemented | **Priority:** MEDIUM

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAYER 4: GAP DETECTION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────┐                   │
│   │  Unmatched Demand Signals              │                   │
│   │  (Signals with no ProductMatch)        │                   │
│   └──────────────────┬────────────────────┘                   │
│                      │                                          │
│           ┌──────────┴──────────┐                             │
│           │   Gap Analyzer        │                             │
│           │                       │                             │
│           │  ┌───────────────┐   │                             │
│           │  │ Unmet Demand  │   │ → No supplier exists        │
│           │  │ Underserved   │   │ → Few suppliers, high demand│
│           │  │ Emerging      │   │ → New category/location     │
│           │  └───────────────┘   │                             │
│           └──────────┬──────────┘                             │
│                      │                                          │
│           ┌──────────┴──────────┐                             │
│           │  MarketGapAlert      │                             │
│           │  (Alert Entity)       │                             │
│           │                       │                             │
│           │  → Routes to admin    │                             │
│           │  → Triggers data      │                             │
│           │    collection         │                             │
│           │  → Incentivizes       │                             │
│           │    contributors       │                             │
│           └───────────────────────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**
- Unmatched demand analyzer
- Market gap classifier (Unmet, Underserved, Emerging)
- Alert routing engine
- Contributor incentive system

**Key Entities:** `MarketGapAlert`

---

### Layer 5: Feedback & Learning
**Status:** 3% Implemented | **Priority:** MEDIUM

```
┌─────────────────────────────────────────────────────────────────┐
│                 LAYER 5: FEEDBACK & LEARNING                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────┐                   │
│   │  Post-Match Interactions               │                   │
│   │  (Delivery, Satisfaction, Returns)   │                   │
│   └──────────────────┬────────────────────┘                   │
│                      │                                          │
│           ┌──────────┴──────────┐                             │
│           │   Feedback Capture    │                             │
│           │                       │                             │
│           │  ┌───────────────┐   │                             │
│           │  │Dissatisfaction│   │ → Return after match        │
│           │  │   Signal      │   │ → Quality issues            │
│           │  ├───────────────┤   │                             │
│           │  │Fulfillment    │   │ → Success/failure           │
│           │  │   Signal      │   │ → Timing, quality           │
│           │  ├───────────────┤   │                             │
│           │  │Validator     │   │ → Community verification    │
│           │  │Confirmation   │   │                             │
│           │  └───────────────┘   │                             │
│           └──────────┬──────────┘                             │
│                      │                                          │
│           ┌──────────┴──────────┐                             │
│           │   Learning Engine     │                             │
│           │                       │                             │
│           │  ┌───────────────┐   │                             │
│           │  │Temporal Decay │   │ → Reduce old signal weight  │
│           │  ├───────────────┤   │                             │
│           │  │Confidence     │   │ → Adjust supplier scores    │
│           │  │Adjustment     │   │ → Improve matching          │
│           │  ├───────────────┤   │                             │
│           │  │Recursive      │   │ → System self-improvement   │
│           │  │Improvement    │   │                             │
│           │  └───────────────┘   │                             │
│           └───────────────────────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**
- DissatisfactionSignal capture
- Fulfillment tracking
- Temporal decay calculator
- Confidence score adjustment
- Recursive improvement loop

**Key Entities:** `DissatisfactionSignal`, `PurchaseRequest`, `FeedbackLoop`

---

## 4. System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │  Consumer   │  │   Admin     │  │  Supplier   │  │  Third-Party    │  │
│  │   Portal    │  │  Dashboard  │  │  Dashboard  │  │    API Clients  │  │
│  │  (React)    │  │  (React)    │  │  (React)    │  │                 │  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘  │
│         │                │                │                  │           │
│         └────────────────┴────────────────┴──────────────────┘           │
│                                   │                                      │
│                          ┌────────┴────────┐                           │
│                          │  Telegram Bot   │                           │
│                          │     NEZA        │                           │
│                          └─────────────────┘                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY LAYER                                │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                    Express.js / GraphQL API                             │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │ │
│  │  │   Auth      │  │   Rate      │  │  Validation │  │   Logging   │  │ │
│  │  │ Middleware  │  │   Limit     │  │ Middleware  │  │ Middleware  │  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                    │                                      │
│                          ┌─────────┴─────────┐                            │
│                          │   WebSocket Hub   │                            │
│                          │  (Real-time)      │                            │
│                          └───────────────────┘                            │
└─────────────��───────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CORE PROTOCOL LAYERS                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   │
│  │  Layer 1    │ → │  Layer 2    │ → │  Layer 3    │ → │  Layer 4    │   │
│  │   Signal    │   │  Normalize  │   │   Match     │   │    Gap      │   │
│  │   Capture   │   ���  & Structure│   │   Engine    │   │  Detection  │   │
│  └─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘   │
│         │                                                          │       │
│         └──────────────────────────────────────────────────────────┘       │
│                                    │                                        │
│                           ┌────────┴────────┐                             │
│                           │   Layer 5       │                             │
│                           │ Feedback &      │                             │
│                           │   Learning      │                             │
│                           └─────────────────┘                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SERVICE LAYER                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │  Signal     │  │  Matching   │  │   Gap       │  │  Feedback   │      │
│  │  Service    │  │  Service    │  │  Service    │  │  Service    │      │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │
│                                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │  Supplier   │  │  Consumer   │  │  Location   │  │  Category   │      │
│  │  Service    │  │  Service    │  │  Service    │  │  Service    │      │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                         │
├──���──────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────┐      ┌─────────────────────────┐             │
│  │      PostgreSQL         │      │         Redis           │             │
│  │   (Primary Database)    │      │    (Cache + Pub/Sub)    │             │
│  │                         │      │                         │             │
│  │  · 15 entity schemas    │      │  · Session cache        │             │
│  │  · ACID transactions    │      │  · Rate limiting        │             │
│  │  · Full-text search     │      │  · Real-time events     │             │
│  │  · JSONB for flexibility│      │  · Matching queue       │             │
│  └─────────────────────────┘      └─────────────────────────┘             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMMERCIAL MODULES (PRIVATE)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ Mobile Money│  │    USSD     │  │  AI Analytics│  │    KYC      │      │
│  │  Gateways   │  │  Routing    │  │   Engine     │  │ Compliance  │      │
│  │  (Private)  │  │  (Private)  │  │  (Private)   │  │  (Private)  │      │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │
│                                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                       │
│  │  Enterprise │  │ White-Label │  │   SSO/RBAC  │                           │
│  │  Features   │  │  Custom     │  │  (Private)  │                           │
│  │  (Private)  │  │  (Private)  │  │             │                           │
│  └─────────────┘  └─────────────┘  └─────────────┘                       │
│                                                                             │
│  🔒 Licensed separately — contact licensing@ekexintelligence.com           │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Data Flow Diagrams

### 5.1 Consumer Signal → Match Flow

```
Consumer submits demand signal
         │
         ▼
┌─────────────────┐
│  Layer 1:       │
│  Signal Capture │ → Validate, deduplicate (Gates A-E)
│                 │ → Store as ConsumerSignal
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Layer 2:       │
│  Normalize      │ → Parse location, category
│                 │ → Compute confidence score
│                 │ → Create DemandObject
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Layer 3:       │
│  Match Engine   │ → Query SupplyObjects
│                 │ → Compute similarity scores
│                 │ → Apply location weights
│                 │ → Create ProductMatch
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐ ┌───────────┐
│ Match │ │ No Match  │
│Found  │ │           │
└───┬───┘ └─────┬─────┘
    │           │
    ▼           ▼
┌─────────┐ ┌─────────────┐
│Notify   │ │ Layer 4:    │
│Consumer │ │ Gap Detect  │ → Create MarketGapAlert
│&       │ │             │ → Route to admin
│Supplier │ │             │ → Trigger data collection
└─────────┘ └─────────────┘
```

### 5.2 Feedback Loop Flow

```
Match delivered to consumer
         │
         ▼
┌─────────────────┐
│ Consumer rates  │
│ experience      │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐ ┌───────────┐
│Satisfy│ │Dissatisfy │
│  ed   │ │           │
└───┬───┘ └─────┬─────┘
    │           │
    ▼           ▼
┌─────────┐ ┌─────────────┐
│Update   │ │ Create      │
│supplier │ │Dissatisfaction│
│score    │ │Signal        │
│+1       │ │             │
└─────────┘ └──────┬──────┘
                     │
                     ▼
            ┌─────────────┐
            │ Layer 5:    │
            │ Learning    │ → Adjust confidence scores
            │             │ → Update temporal weights
            │             │ → Improve future matching
            └─────────────┘
```

---

## 6. Technology Stack

### 6.1 Core Infrastructure

| Component | Technology | Version | Purpose |
|---|---|---|---|
| Runtime | Node.js | 18+ | Server execution |
| Framework | Express.js | 4.x | HTTP server |
| API | GraphQL | 16+ | Query language |
| Database | PostgreSQL | 14+ | Primary data store |
| Cache | Redis | 7+ | Caching, Pub/Sub, sessions |
| ORM | Prisma | 5+ | Database abstraction |
| AI/ML | Base44 Native | Latest | Matching engine |
| Frontend | React/Next.js | 14+ | UI framework |

### 6.2 DevOps & Infrastructure

| Component | Technology | Purpose |
|---|---|---|
| Containerization | Docker | Application packaging |
| Orchestration | Docker Compose | Local development |
| CI/CD | GitHub Actions | Automated testing/deployment |
| Monitoring | Base44 Native | Application metrics |
| Logging | Winston | Structured logging |

### 6.3 External Integrations

| Service | Status | Module |
|---|---|---|
| Telegram Bot API | ✅ Live | Core (NEZA) |
| MTN MoMo API | 🔒 Private | Commercial |
| Wave API | 🔒 Private | Commercial |
| Orange Money API | 🔒 Private | Commercial |
| USSD Gateways | 🔒 Private | Commercial |
| KYC Providers | 🔒 Private | Commercial |

---

## 7. Security Architecture

### 7.1 Authentication & Authorization

```
┌─────────────────────────────────────────┐
│         AUTHENTICATION FLOW             │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────┐    ┌─────────────┐        │
│  │  User   │───→│   Login     │        │
│  │         │    │  (JWT)      │        │
│  └─────────┘    └──────┬──────┘        │
│                        │                │
│                        ▼                │
│               ┌─────────────┐         │
│               │  JWT Token    │         │
│               │  (15 min)     │         │
│               └──────┬──────┘         │
│                      │                 │
│                      ▼                 │
│               ┌─────────────┐         │
│               │  Refresh     │         │
│               │  Token       │         │
│               │  (7 days)    │         │
│               └──────┬──────┘         │
│                      │                 │
│                      ▼                 │
│               ┌─────────────┐         │
│               │  RBAC Check │         │
│               │  (Role-based│         │
│               │   access)   │         │
│               └─────────────┘         │
│                                         │
└─────────────────────────────────────────┘
```

### 7.2 Data Protection

| Measure | Implementation | Compliance |
|---|---|---|
| Encryption at Rest | AES-256 (PostgreSQL) | Rwanda Law 058/2021 |
| Encryption in Transit | TLS 1.3 | Rwanda Law 058/2021 |
| PII Anonymization | Hash + salt for analytics | GDPR-inspired |
| Audit Logging | All data access logged | Regulatory |
| Access Control | Principle of least privilege | Security best practice |

### 7.3 API Security

- Rate limiting: 100 req/min per IP, 1000 req/min per API key
- Input validation: JSON Schema + custom validators
- SQL injection prevention: Parameterized queries (Prisma)
- XSS prevention: Content Security Policy + output encoding
- CSRF protection: Double-submit cookie pattern

---

## 8. Scalability & Performance

### 8.1 Horizontal Scaling Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCALING ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐                                                   │
│   │  CDN    │  → Static assets, API responses                   │
│   │(CloudFlare)│                                                │
│   └────┬────┘                                                   │
│        │                                                        │
│   ┌────┴────┐                                                   │
│   │ Load    │  → Round-robin / least-connections                │
│   │Balancer │                                                   │
│   └────┬────┘                                                   │
│        │                                                        │
│   ┌────┴────┐    ┌─────────┐    ┌─────────┐                    │
│   │ API     │    │ API     │    │ API     │                    │
│   │Server 1 │    │Server 2 │    │Server N │  → Auto-scaling    │
│   │(Docker) │    │(Docker) │    │(Docker) │                    │
│   └────┬────┘    └────┬────┘    └────┬────┘                    │
│        │              │              │                          │
│        └──────────────┴──────────────┘                          │
│                      │                                            │
│   ┌──────────────────┴──────────────────┐                       │
│   │         PostgreSQL Cluster          │                       │
│   │  (Primary + Read Replicas)          │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   ┌─────────────────────────────────────┐                       │
│   │         Redis Cluster               │                       │
│   │  (Cache + Session + Pub/Sub)          │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Performance Targets

| Metric | Target | Current |
|---|---|---|
| API Response Time | < 200ms p95 | ~150ms |
| Signal Processing | < 5 seconds | ~3s (without AI) |
| Matching Engine | < 2 seconds | Not firing |
| Database Queries | < 100ms | ~50ms |
| Concurrent Users | 10,000 | ~10 |
| Uptime | 99.9% | 99.5% |

### 8.3 Caching Strategy

| Cache Level | Technology | TTL | Purpose |
|---|---|---|---|
| L1 (In-Memory) | Node.js Map | 60s | Hot data |
| L2 (Redis) | Redis | 5min | API responses |
| L3 (CDN) | CloudFlare | 1hr | Static assets |
| L4 (Database) | PostgreSQL | Permanent | Persistent data |

---

## 9. Commercial Module Boundaries

### 9.1 Open-Core vs. Commercial Separation

```
┌─────────────────────────────────────────────────────────────────┐
│                    MODULE BOUNDARY MAP                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              PUBLIC (AGPLv3) — This Repository            │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │   │
│  │  │ Signal  │ │Normalize│ │  Match  │ │   Gap   │      │   │
│  │  │ Capture │ │ & Struct│ │ Engine  │ │ Detect  │      │   │
│  │  │ (Core)  │ │ (Core)  │ │ (Core)  │ │ (Core)  │      │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘      │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │   │
│  │  │  API    │ │  Auth   │ │  Admin  │ │ Consumer│      │   │
│  │  │ Routes  │ │ (Basic) │ │ Dashboard│ │  Portal │      │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘      │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐                   │   │
│  │  │ Database│ │  Cache  │ │  Queue  │                   │   │
│  │  │ Schemas │ │ (Redis) │ │ (Redis) │                   │   │
│  │  └─────────┘ └─────────┘ └─────────┘                   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              │ API Interface                   │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           PRIVATE (Commercial License)                  │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │   │
│  │  │  MTN    │ │  Wave   │ │ Orange  │ │ Cross-  │      │   │
│  │  │  MoMo   │ │  Pay    │ │  Money  │ │  Telco  │      │   │
│  │  │Gateway  │ │Gateway  │ │Gateway  │ │Reconcile│      │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘      │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │   │
│  │  │  USSD   │ │  SMPP   │ │  Menu   │ │ Offline │      │   │
│  │  │ Session │ │ Adapter │ │ Router  │ │  Sync   │      │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘      │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │   │
│  │  │   AI    │ │  Price  │ │ Supplier│ │ Market  │      │   │
│  │  │ Forecast│ │Elasticity│ │ Score  │ │  Trend  │      │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘      │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │   │
│  │  │   KYC   │ │ Document│ │Biometric│ │Regulatory│      │   │
│  │  │ Verify  │ │   OCR   │ │ Encrypt │ │  Report  │      │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘      │   │
│  │                                                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐                   │   │
│  │  │ White-  │ │  SSO    │ │ Multi-  │                   │   │
│  │  │ Label   │ │  RBAC   │ │ Tenant  │                   │   │
│  │  └─────────┘ └─────────┘ └─────────┘                   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  🔒 Access: licensing@ekexintelligence.com                     │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Integration Contract

Commercial modules integrate with the core through:
- **Stable Public APIs:** Versioned REST/GraphQL endpoints
- **Webhook Events:** Async notifications for state changes
- **Plugin SDK:** TypeScript SDK for building extensions (AGPLv3)
- **Database Views:** Read-only views for analytics

---

## 10. Deployment Architecture

### 10.1 Local Development

```bash
# Single-machine development
docker-compose -f docker-compose.dev.yml up

# Services:
# - app: Node.js API server (port 3000)
# - db: PostgreSQL (port 5432)
# - cache: Redis (port 6379)
# - bot: Telegram NEZA (port 3001)
```

### 10.2 Staging Environment

```
┌─────────────────────────────────────────┐
│           STAGING CLUSTER               │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │  App    │  │  App    │  │  Worker │ │
│  │ Server  │  │ Server  │  │  Queue  │ │
│  │  (t2)   │  │  (t2)   │  │  (t2)   │ │
│  └────┬────┘  └────┬────┘  └────┬────┘ │
│       │            │            │      │
│       └────────────┴────────────┘      │
│                    │                    │
│         ┌──────────┴──────────┐        │
│         │   RDS PostgreSQL     │        │
│         │   (db.t3.micro)      │        │
│         └──────────────────────┘        │
│                                         │
│         ┌──────────────────────┐       │
│         │   ElastiCache Redis   │       │
│         │   (cache.t3.micro)    │       │
│         └──────────────────────┘       │
│                                         │
└─────────────────────────────────────────┘
```

### 10.3 Production Environment

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCTION ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐                                                    │
│  │   CDN   │  CloudFlare / AWS CloudFront                       │
│  │         │  → Static assets, DDoS protection                    │
│  └────┬────┘                                                    │
│       │                                                         │
│  ┌────┴────┐                                                    │
│  │   WAF   │  Web Application Firewall                           │
│  │         │  → Rate limiting, bot detection                      │
│  └────┬────┘                                                    │
│       │                                                         │
│  ┌────┴────────────────────────────────────┐                    │
│  │         Load Balancer (ALB/NLB)        │                    │
│  │         SSL termination                │                    │
│  └────┬───────────────────────────────────┘                    │
│       │                                                         │
│  ┌────┴────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│  │  App    │    │  App    │    │  App    │    │  Worker │    │
│  │Server 1 │    │Server 2 │    │Server 3 │    │  Pods   │    │
│  │(c5.xl)  │    │(c5.xl)  │    │(c5.xl)  │    │(t3.lg)  │    │
│  └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘    │
│       │              │              │              │          │
│       └──────────────┴──────────────┴──────────────┘          │
│                      │                                          │
│  ┌───────────────────┴───────────────────┐                    │
│  │      PostgreSQL Primary + Replicas     │                    │
│  │      (db.r5.xlarge primary)            │                    │
│  │      (2x db.r5.large replicas)         │                    │
│  └────────────────────────────────────────┘                    │
│                                                                 │
│  ┌────────────────────────────────────────┐                    │
│  │      Redis Cluster (ElastiCache)        │                    │
│  │      (3x cache.r5.large nodes)          │                    │
│  │      → Cache + Session + Pub/Sub        │                    │
│  └────────────────────────────────────────┘                    │
│                                                                 │
│  ┌────────────────────────────────────────┐                    │
│  │      S3 / Object Storage                │                    │
│  │      → File uploads, exports, backups   │                    │
│  └────────────────────────────────────────┘                    │
│                                                                 │
│  ┌────────────────────────────────────────┐                    │
│  │      Monitoring & Alerting              │                    │
│  │      → CloudWatch / Datadog / Grafana   │                    │
│  └────────────────────────────────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Appendix A: Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│   Consumer      │       │  ConsumerSignal │       │   Category      │
│                 │1    * │                 │*     1│                 │
│  id (PK)        │◄──────►│  id (PK)        │◄──────►│  id (PK)        │
│  phone          │       │  consumer_id    │       │  name           │
│  location_id    │       │  category_id    │       │  description    │
│  created_at     │       │  location_id    │       │  parent_id      │
└─────────────────┘       │  description    │       └─────────────────┘
                          │  intent_score   │
                          │  signal_state   │       ┌─────────────────┐
                          │  created_at     │       │    Location     │
                          └─────────────────┘       │                 │
                                   │                │  id (PK)        │
                                   │                │  city           │
                                   │                │  country        │
                                   │                │  lat/lng        │
                                   └───────────────►│  featured       │
                                                    └─────────────────┘

┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│    Supplier     │       │  SupplierProduct│       │   ProductMatch  │
│                 │1    * │                 │1    * │                 │
│  id (PK)        │◄──────►│  id (PK)        │◄──────►│  id (PK)        │
│  name           │       │  supplier_id    │       │  demand_id      │
│  status         │       │  category_id    │       │  supply_id      │
│  location_id    │       │  name           │       │  match_type     │
│  tier           │       │  description    │       │  similarity     │
│  verified       │       │  price_range    │       │  location_score │
└─────────────────┘       │  quality_tier   │       │  temporal_score │
                          │  availability   │       │  created_at     │
                          │  created_at     │       └─────────────────┘
                          └─────────────────┘

┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│  MarketGapAlert │       │Dissatisfaction  │       │ PurchaseRequest │
│                 │       │    Signal       │       │                 │
│  id (PK)        │       │                 │       │  id (PK)        │
│  signal_id      │       │  id (PK)        │       │  match_id       │
│  gap_type       │       │  match_id       │       │  consumer_id    │
│  severity       │       │  reason         │       │  supplier_id    │
│  assigned_to    │       │  severity       │       │  status         │
│  status         │       │  created_at     │       │  created_at     │
│  created_at     │       └─────────────────┘       └─────────────────┘
└─────────────────┘

┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│    AdminUser    │       │   MatchHistory  │       │  FeedbackLoop   │
│                 │       │                 │       │                 │
│  id (PK)        │       │  id (PK)        │       │  id (PK)        │
│  email          │       │  match_id       │       │  entity_type    │
│  role           │       │  action         │       │  entity_id      │
│  permissions    │       │  actor_type     │       │  feedback_type  │
│  last_login     │       │  actor_id       │       │  score_delta    │
│  created_at     │       │  timestamp      │       │  applied_at     │
└─────────────────┘       └─────────────────┘       └─────────────────┘
```

---

## Appendix B: Glossary

| Term | Definition |
|---|---|
| **ConsumerSignal** | Raw demand signal submitted by a consumer |
| **DemandObject** | Structured, normalized representation of consumer demand |
| **SupplyObject** | Structured representation of supplier product/capability |
| **ProductMatch** | Result of matching a DemandObject to a SupplyObject |
| **Signal State** | Current state of a signal in the processing pipeline |
| **Gate** | Deduplication checkpoint (A through E) |
| **Gap Type** | Classification of unmet demand (Unmet, Underserved, Emerging) |
| **Intent Score** | Confidence metric for signal quality (0-1) |
| **Quality Tier** | Supplier classification (Bronze, Silver, Gold, Platinum) |
| **NEZA** | EKEX Telegram bot for signal capture |

---

<div align="center">

**[← Back to README](../README.md)** · **[Database Schema →](DATABASE.md)**

*© 2026 Kimuntu Group · EKEX Intelligence*

</div>
