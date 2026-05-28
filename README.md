<div align="center">

# EKEX INTELLIGENCE

Scalable Digital Infrastructure for Demand-Driven African Supply Chains

[https://www.gnu.org/licenses/agpl-3.0](https://www.gnu.org/licenses/agpl-3.0)

[Documentation](https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE/tree/main/docs) · [API Reference](https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE/docs/API.md) · [Report Issue](https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE/issues)

</div>

## Table of Contents

- [Project Overview](#project-overview)
- [Repository Architecture](#repository-architecture)
- [Core Capabilities](#core-capabilities)
- [Technology Stack](#technology-stack)
- [Getting Started](#getting-started)
- [Defensive Licensing & Compliance Notice](#defensive-licensing--compliance-notice)
- [Contributing](#contributing)
- [Security](#security)
- [Support & Enterprise](#support--enterprise)
- [About Kimuntu Group](#about-kimuntu-group)

## Project Overview

EKEX Intelligence is a comprehensive digital infrastructure designed for market demand and B2B supply-chain management across Africa. Built from first principles by the Kimuntu Group, EKEX captures consumer demand signals, structures them into standardized intelligence.

### The Problem We Solve

Fragmented African markets suffer from a critical information asymmetry: consumers express demand through informal channels (WhatsApp, Telegram, word-of-mouth), while suppliers operate in silos without access to real-time signal data. This fragmentation leads to:

- **Inventory Mismatches**: Suppliers stock products nobody wants while demand goes unmet
- **Missed Opportunities**: Market gaps remain invisible to entrepreneurs who could fill them
- **Information Arbitrage**: Middlemen extract disproportionate value from information gaps
- **Capital Inefficiency**: B2B credit decisions rely on guesswork instead of demand data

### Our Mission

To build the foundational data infrastructure that enables African businesses to anticipate demand, optimize supply, and capture market opportunities that would otherwise remain invisible.

### Current Operational Scope

| Metric | Status |
|--------|--------|
| Primary Market | Kigali, Rwanda |
| Expansion Pipeline | Nairobi, Kenya · Kampala, Uganda |
| Active Consumers | 4+ real users submitting signals |
| Verified Suppliers | 6+ active across Rwanda & Uganda |
| Signal Volume | 14+ demand signals captured |
| Product Categories | Fashion/Apparel (primary) |
| Telegram Bot | NEZA (live, timeout fix pending) |

## Repository Architecture

This public repository contains the AGPLv3-licensed core infrastructure of EKEX Intelligence. Commercial extensions, proprietary integrations, and enterprise-grade modules are maintained in separate private repositories.

```
EKEX INTELLIGENCE — REPOSITORY MAP
═════════════════════════════════════════════════════════════════════

📁 PUBLIC REPOSITORY (This Repo) — GNU AGPLv3
│
├── 📁 src/
│   ├── 📁 api/                    Core REST/GraphQL API endpoints
│   ├── 📁 core/                   Protocol implementation layers
│   │   ├── 📁 layer-1-signal/     Signal capture & ingestion
│   │   ├── 📁 layer-2-normalize/  Data standardization & deduplication
│   │   ├── 📁 layer-3-match/      Matching engine (Base44 native AI)
│   │   ├── 📁 layer-4-gap/        Market gap detection schemas
│   │   └── 📁 layer-5-feedback/   Feedback loop & learning schemas
│   ├── 📁 models/                 Database entity definitions (15 schemas)
│   ├── 📁 services/               Core business services
│   └── 📁 config/                 Configuration templates & examples
│
├── 📁 docs/                       Public documentation
├── 📁 tests/                      Test suites & fixtures
├── 📁 scripts/                    Deployment & utility scripts
├── 📁 data/                       Sample data & schema definitions
│
├── 📄 .env.example                Environment variable template
├── 📄 docker-compose.example.yml  Local development orchestration
├── 📄 Dockerfile.example          Container build template
├── 📄 README.md                   This file
├── 📄 CONTRIBUTING.md             Contribution guidelines & CLA
├── 📄 LICENSE                     GNU AGPLv3 (with EKEX amendments)
└── 📄 .gitignore                  Privacy & security protections

═════════════════════════════════════════════════════════════════════
📁 PRIVATE REPOSITORIES — Commercial License Required
═════════════════════════════════════════════════════════════════════

🔒 ekex-commercial/mobile-money/
   ├── MTN MoMo Gateway Integration
   ├── Wave Payment Processing
   ├── Orange Money API Adapter
   └── Cross-telco reconciliation engine

🔒 ekex-commercial/ussd/
   ├── USSD Session Management
   ├── Menu Routing Engine
   ├── SMPP Protocol Adapter
   └── Offline-first sync layer

🔒 ekex-commercial/ai-analytics/
   ├── Demand Forecasting Models
   ├── Price Elasticity Engine
   ├── Supplier Performance Scoring
   └── Market Trend Prediction

🔒 ekex-commercial/kyc-compliance/
   ├── Identity Verification Pipelines
   ├── Document OCR & Validation
   ├── Regulatory Reporting (NIMC, Huduma, NIA)
   └── Biometric Encryption Layer

🔒 ekex-commercial/enterprise/
   ├── White-Label Customization
   ├── Advanced RBAC & SSO
   ├── Multi-tenant Orchestration
   └── SLA Monitoring & Alerting

═════════════════════════════════════════════════════════════════════
```

### Boundary Philosophy

The open-core boundary is designed around protocol vs. integration:

- **Public (AGPLv3)**: The demand-supply coordination protocol, data models, matching algorithms, and API infrastructure.
- **Private (Commercial)**: Financial integrations, telecommunications protocols, advanced analytics, compliance engines, and enterprise customization layers.

For Enterprise Licensing: Contact [info@ekexintelligence.com](mailto:info@ekexintelligence.com) for access to private commercial repositories and enterprise deployment options.

## Core Capabilities

### Layer 1: Signal Capture
- Multi-channel demand signal ingestion (Web, Telegram Bot NEZA, API)
- Location-enriched signal capture with city/country standardization
- 5-gate deduplication logic (technical duplicate, return-after-match, genuine re-submission, velocity flood, default)

### Layer 2: Normalization & Structuring
- Global location standardization (10 featured cities + free text, all countries including Congo DRC)
- DemandObject and SupplyObject formal structures
- Source reliability scoring framework

### Layer 3: Matching Engine
- Base44 native AI-powered demand-supply matching
- Location-weighted 2-tier fallback (city → country)
- Similarity scoring and probabilistic inference

### Layer 4: Gap Detection
- Market gap alert state machine
- Unmet demand classification framework
- Emerging demand identification schemas

### Layer 5: Feedback & Learning
- DissatisfactionSignal capture
- Temporal decay of demand relevance
- Confidence score adjustment over time
- Recursive system improvement loop

## Technology Stack

| Layer | Technology |
|-------|------------|
| Frontend | React / Next.js (Mobile-responsive, dark theme) |
| Backend API | Node.js / Express / GraphQL |
| Database | PostgreSQL + Redis (caching layer) |
| AI/ML | Base44 Native AI (Matching Engine) |
| Messaging | Telegram Bot API (NEZA) |
| Containerization | Docker + Docker Compose |
| Cloud | Base44 (Primary), Multi-cloud ready |

## Getting Started

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- Docker & Docker Compose (recommended)
- Git
- PostgreSQL 14+ (or use Docker)
- Redis 7+ (or use Docker)

### 1. Clone the Repository

```bash
git clone https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE.git
cd EKEXINTELLIGENCE
```

### 2. Configure Environment

```bash
# Copy the example environment file
cp .env.example .env.local

# Edit .env.local with your local configuration
# NEVER commit .env.local to git — it is protected by .gitignore
nano .env.local
```

### 3. Install Dependencies

```bash
npm install
```

### 4. Start Infrastructure Services

```bash
# Start PostgreSQL and Redis via Docker Compose
docker-compose -f docker-compose.example.yml up -d db redis
```

### 5. Run Database Migrations

```bash
npm run db:migrate
```

### 6. Seed Sample Data

```bash
npm run db:seed
```

### 7. Start Development Server

```bash
npm run dev
```

The API will be available at `http://localhost:3000` and the admin dashboard at `http://localhost:3000/admin`.

### 8. Run Tests

```bash
npm test
```

## Defensive Licensing & Compliance Notice

### GNU Affero General Public License v3.0 (AGPLv3)

This repository and all files within it are licensed under the GNU Affero General Public License v3.0 (AGPLv3), with EKEX-specific amendments.

#### ⚠️ CRITICAL: Network Service Obligations

Section 13 of AGPLv3 imposes specific obligations on anyone deploying this software as a network service (SaaS, API, web application):

> If you modify the Program, your modified version must prominently offer all users interacting with it remotely through a computer network an opportunity to receive the Corresponding Source of your modification.

### What This Means for EKEX Deployments

- **If you deploy EKEX Intelligence as a SaaS platform** (even unmodified), you MUST provide source code access to all users interacting with your service.
- **If you modify EKEX** (configuration changes, custom integrations, UI modifications, API wrappers), your modifications MUST be released under AGPLv3 with source code available.
- **API Wrappers & Shims**: Building a proprietary wrapper around EKEX APIs that modifies request/response data, authentication flows, or routing logic constitutes a "modified version" under this license.
- **Reverse Proxy Circumvention**: Deploying reverse proxies, CDNs, or gateways that remove or obscure the source code offer mechanism is a direct license violation.
- **White-Label & Reseller Restrictions**: Offering white-label or reseller versions of EKEX infrastructure without disclosing Corresponding Source to end users is prohibited.

### Enterprise Commercial Licensing Alternative

If your organization cannot comply with AGPLv3 obligations, or requires:

- Closed-source deployment rights
- Exemption from source disclosure obligations
- Access to proprietary commercial modules (Mobile Money, USSD, AI Analytics, KYC)
- Priority support and SLA guarantees
- Patent and trademark usage rights

**Contact us for licensing:**

📧 [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

Our commercial licensing program is designed for:
- Enterprise B2B platforms
- Telcos and Mobile Money operators
- Logistics and supply-chain companies
- Government and regulatory bodies
- Multi-national corporations operating in African markets

## Contributing

We welcome contributions from the African developer community and global open-source ecosystem. Please read our [CONTRIBUTING.md](./CONTRIBUTING.md) for detailed guidelines, our Contributor License Agreement, and community standards.

### Key Contribution Areas:

- Signal categorization engine (fix 28% "Unknown" rate)
- Matching engine end-to-end verification
- Confidence scoring model implementation
- Local development environment setup
- Protocol documentation and RFCs

## Security

### Reporting Vulnerabilities

If you discover a security vulnerability in EKEX Intelligence, please report it responsibly:

📧 [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

**Please DO NOT open public issues for security vulnerabilities.**

### Security Best Practices

- All environment files are protected by `.gitignore`
- Commercial module boundaries are strictly enforced
- API rate limiting is enabled by default
- Input validation and sanitization at all entry points
- Database queries use parameterized statements

## Support & Enterprise

| Channel | Contact |
|---------|----------|
| All Inquiries | [info@ekexintelligence.com](mailto:info@ekexintelligence.com) |

## About Kimuntu Group

EKEX Intelligence is a product of **Kimuntu Group**, founded on **16 March 2026**.

### Leadership:

- **Kayembe Ilunga Eddy-Grant** — Founder & Head of the Board
- **Goetz Kisoni** — Co-Founder & CEO

### Contributors & Advisors:

- **Stephane Bilambo** — First Angel Investor & Early Contributor (4.5% equity, no vesting) · Part-time Consultant

We are committed to building open, transparent infrastructure for African markets. This repository reflects that commitment — honest about current capabilities, clear about future direction, and transparent about the hard technical problems we're solving together.

> "We are not asking you to believe the white paper. We are asking you to help us make it true."
>
> — Kimuntu Group, 2026

---

<div align="center">

**EKEX Intelligence** · **Kimuntu Group** · [Documentation](https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE/tree/main/docs)

© 2026 Kimuntu Group. Licensed under [GNU AGPLv3](https://www.gnu.org/licenses/agpl-3.0).

</div>
