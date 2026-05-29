<div align="center">

# EKEX INTELLIGENCE

### Open Intent Intelligence Infrastructure for Local Commerce

[![AGPLv3 License](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-green.svg)]()
[![Platform](https://img.shields.io/badge/Platform-Web%20%7C%20API%20%7C%20Telegram%20%7C%20WhatsApp-orange.svg)]()
[![Region](https://img.shields.io/badge/Region-Africa-FFD700.svg)]()

**[Documentation](https://docs.ekexintelligence.com)** · **[API Reference](https://api.ekexintelligence.com)** · **[Report Issue](https://github.com/ekexintelligence/ekex-core/issues)**

</div>

---

## Table of Contents

- [What is EKEX Intelligence?](#what-is-ekex-intelligence)
- [NEZA: The Visual Intent Engine](#neza-the-visual-intent-engine)
- [The Core Loop](#the-core-loop)
- [Repository Architecture](#repository-architecture)
- [Core Capabilities](#core-capabilities)
- [Technology Stack](#technology-stack)
- [Getting Started](#getting-started)
- [AGPL-3.0 Compliance Notice](#agpl-3.0-compliance-notice)
- [Contributing](#contributing)
- [Security](#security)
- [Support](#support)
- [About](#about)

---

## What is EKEX Intelligence?

**EKEX Intelligence** is a self-hostable, open-source commerce intelligence infrastructure. It transforms how local markets match demand to supply by understanding *what people want* — especially when they show, not just tell.

### The Problem

Informal African markets run on intent expressed through scattered channels: WhatsApp forwards, Telegram messages, voice notes, and photos. Suppliers operate blind to aggregate demand. EKEX bridges this gap not by building another marketplace, but by making intent *understandable* and *actionable* as open infrastructure.

### Our Mission

> *To build the foundational data infrastructure that enables African businesses to anticipate demand, optimize supply, and capture market opportunities that would otherwise remain invisible.*

### Current Operational Scope

| Metric | Status |
|---|---|
| **Primary Market** | Kigali, Rwanda |
| **Expansion Pipeline** | Nairobi, Kenya · Kampala, Uganda |
| **Active Consumers** | 4+ real users submitting signals |
| **Verified Suppliers** | 6+ active across Rwanda & Uganda |
| **Signal Volume** | 14+ demand signals captured |
| **Product Categories** | Fashion/Apparel (primary) |
| **Telegram Bot** | NEZA (live, timeout fix pending) |

---

## NEZA: The Visual Intent Engine

**NEZA** is the multimodal intent intelligence layer at the heart of EKEX. It processes a single input — an image, a voice note, a WhatsApp message, or a Telegram text — and extracts structured intent: product, attributes, quality, style, and urgency. When an image is present, NEZA treats it as the highest-confidence signal.

### How NEZA Works

1. **Ingest** — Captures input from any channel (image, text, voice, WhatsApp, Telegram)
2. **Understand** — Extracts intent: product recognition, attribute parsing (material, quality, category, style)
3. **Structure** — Formalizes intent into a `DemandObject` with confidence scoring
4. **Route** — Hands structured intent to the AI inference layer for matching

NEZA is not a chatbot. It is an intent engine that happens to speak through messaging channels.

---

## The Core Loop

```
Intent → Visual/Language Understanding → Product Match → Local Availability
    ↓
Fulfillment OR Unmet Demand → EKEX Network Routing
```

1. **Intent Capture** — NEZA ingests multimodal signals (image, text, voice, WhatsApp, Telegram).
2. **Visual/Language Understanding** — NEZA extracts intent: product recognition, attribute parsing (material, quality, category, style).
3. **Product Match** — AI inference layer matches intent against structured supply data.
4. **Local Availability** — Location-weighted matching (city → country fallback) verifies stock proximity.
5. **Fulfillment OR Unmet Demand** — If matched, route to supplier. If unmatched, flag as unmet demand for network intelligence.
6. **EKEX Network Routing** — Aggregate unmet demand, route signals to supplier networks, enable regional interoperability.

---

## Repository Architecture

```
EKEX INTELLIGENCE — REPOSITORY MAP
═══════════════════════════════════════════════════════════════════════════════

📁 PUBLIC REPOSITORY — GNU AGPLv3
│
├── 📁 src/
│   ├── 📁 neza/                   NEZA intent engine (multimodal input processing)
│   │   ├── 📁 signal-ingestion/   Voice, image, text, WhatsApp, Telegram capture
│   │   ├── 📁 visual-intent/      Image-based product recognition & attribute extraction
│   │   └── 📁 intent-structurer/  DemandObject formalization
│   │
│   ├── 📁 inference/              AI inference layer
│   │   ├── 📁 embedding/          Product + intent vectorization
│   │   ├── 📁 matching/           Similarity scoring & probabilistic match
│   │   └── 📁 ranking/            Price, quality, similarity, preference fit ranking
│   │
│   ├── 📁 ekex/                   EKEX intelligence infrastructure
│   │   ├── 📁 network-routing/    Supplier network signal distribution
│   │   ├── 📁 demand-aggregation/ Unmet demand pooling & regional interoperability
│   │   ├── 📁 availability/       Local stock verification & fallback logic
│   │   └── 📁 feedback/           Fulfillment outcome learning loop
│   │
│   ├── 📁 api/                    REST/GraphQL endpoints
│   ├── 📁 models/                 Database schemas (DemandObject, SupplyObject, etc.)
│   └── 📁 config/                 Environment & deployment templates
│
├── 📁 docs/                       Public documentation
│   ├── 📁 architecture/
│   ├── 📁 neza/
│   ├── 📁 apis/
│   ├── 📁 deployment/
│   ├── 📁 intelligence-systems/
│   ├── 📁 supplier-network/
│   ├── 📁 onboarding/
│   └── 📁 legal/
│
├── 📁 tests/                      Test suites & fixtures
├── 📁 scripts/                    Deployment & utility scripts
├── 📁 data/                       Sample data & schema definitions
│
├── 📄 .env.example                Environment variable template
├── 📄 docker-compose.yml          Local development orchestration
├── 📄 Dockerfile                  Container build template
├── 📄 README.md                   This file
├── 📄 CONTRIBUTING.md             Contribution guidelines
├── 📄 LICENSE                     GNU AGPLv3 + EKEX Network Service Amendments
└── 📄 .gitignore                  Privacy & security protections

═══════════════════════════════════════════════════════════════════════════════
```

---

## Core Capabilities

### NEZA Intent Engine
- **Multimodal ingestion**: Image, text, voice, WhatsApp, Telegram
- **Visual intent extraction**: Product recognition, material detection, quality assessment, style classification
- **Language understanding**: Attribute parsing, urgency detection, preference inference
- **Signal deduplication**: 5-gate logic (technical duplicate, return-after-match, genuine re-submission, velocity flood, default)

### AI Inference Layer
- **Embedding & vectorization**: Product + intent semantic representation
- **Probabilistic matching**: Similarity scoring with confidence thresholds
- **Ranking engine**: Price, quality, similarity, and preference-fit ranking
- **Location-weighted fallback**: City → country availability tiers

### EKEX Intelligence Layer
- **Local availability matching**: Real-time stock proximity verification
- **Unmet demand detection**: Classification of unmatched intent for network intelligence
- **Supplier network routing**: Signal distribution to verified supplier pools
- **Regional interoperability**: Cross-city, cross-country demand aggregation
- **Feedback learning**: Temporal decay, confidence adjustment, recursive improvement

---

## Technology Stack

| Layer | Technology |
|---|---|
| **Frontend** | React / Next.js (Mobile-responsive, dark theme) |
| **Backend API** | Node.js / Express / GraphQL |
| **Database** | PostgreSQL + Redis (caching layer) |
| **AI/ML** | AI inference layer (embedding + matching engine) |
| **Messaging** | Telegram Bot API, WhatsApp Business API |
| **Containerization** | Docker + Docker Compose |
| **Cloud** | Multi-cloud ready, self-hostable by default |

---

## Getting Started

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- Docker & Docker Compose (recommended)
- Git
- PostgreSQL 14+ (or use Docker)
- Redis 7+ (or use Docker)

### 1. Clone the Repository

```bash
git clone https://github.com/ekexintelligence/ekex-core.git
cd ekex-core
```

### 2. Configure Environment

```bash
cp .env.example .env.local
# Edit .env.local — NEVER commit this file
```

### 3. Install & Start

```bash
npm install
docker-compose up -d db redis
npm run db:migrate
npm run db:seed
npm run dev
```

API: `http://localhost:3000` | Admin: `http://localhost:3000/admin`

### 4. Run Tests

```bash
npm test
```

---

## AGPL-3.0 Compliance Notice

This repository is free software licensed under the **GNU Affero General Public License v3.0**.

### Network Service Obligations

If you deploy EKEX Intelligence as a network service (SaaS, API, bot, or any remote user interaction), you **must** offer all users the **Corresponding Source** of your deployed version, including all modifications.

### What This Means

- **Unmodified deployments**: Still require source offer mechanism.
- **Modified deployments**: All modifications must be released under AGPL-3.0.
- **API wrappers & shims**: Proprietary wrappers around EKEX APIs do NOT exempt you from source disclosure for the underlying EKEX deployment.
- **White-label deployments**: Every end user interface must contain the source offer.

### No Proprietary Sublicensing

EKEX Intelligence is licensed under AGPL-3.0 **only**. No proprietary sublicensing, commercial licenses, or closed-source distribution rights are granted for any code in this repository. Revenue may be generated through AGPL-3.0-compliant services: consulting, support, hosting, and deployment assistance.

For the full license text, see [LICENSE](LICENSE).

---

## Contributing

We welcome contributions from the African developer community and global open-source ecosystem. Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines and our Developer Certificate of Origin (DCO) process.

**Priority Areas:**
- Visual intent engine accuracy (reduce 28% "Unknown" classification rate)
- End-to-end matching engine verification
- Confidence scoring model calibration
- Local development environment simplification
- Protocol RFCs and architecture documentation

---

## Security

Report vulnerabilities responsibly: **security@ekexintelligence.com**
Do NOT open public issues for security vulnerabilities.

- Environment files protected by `.gitignore`
- API rate limiting enabled by default
- Parameterized database queries
- Input validation at all entry points

---

## Support

| Channel | Contact |
|---|---|
| **General Inquiries** | info@ekexintelligence.com |
| **Security Issues** | security@ekexintelligence.com |
| **Legal Matters** | legal@ekexintelligence.com |
| **Contributors** | contributors@ekexintelligence.com |

---

## About

**EKEX Intelligence** is maintained by **Kimuntu Group**, founded 16 March 2026 by **Kayembe Ilunga Eddy-Grant**, with co-founders **Stephane Bilambo** (Finance & Operations) and **Goetz Kisoni**.

> *"We are not asking you to believe the white paper. We are asking you to help us make it true."*
> — Kimuntu Group, 2026

---

<div align="center">

**[EKEX Intelligence](https://ekexintelligence.com)** · **[Documentation](https://docs.ekexintelligence.com)** · **[Kimuntu Group](https://kimuntugroup.com)**

*© 2026 Kimuntu Group. Licensed under GNU AGPLv3.*

</div>
