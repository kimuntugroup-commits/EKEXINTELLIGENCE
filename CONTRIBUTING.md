
# Write CONTRIBUTING.md and docs/ files with ACTUAL architecture

contributing_content = """# Contributing to EKEX Intelligence

Thank you for your interest in contributing to **EKEX Intelligence**, the open intent intelligence infrastructure for local commerce across Africa. This document outlines our contribution workflow, coding standards, and the Developer Certificate of Origin (DCO) process that governs all contributions.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Contribution Workflow](#contribution-workflow)
- [Development Setup](#development-setup)
- [Coding Standards](#coding-standards)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)
- [Developer Certificate of Origin (DCO)](#developer-certificate-of-origin-dco)
- [Questions & Support](#questions--support)

---

## Code of Conduct

### Our Pledge

We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, sex characteristics, gender identity and expression, level of experience, education, socio-economic status, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Positive Behavior

- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community and the open infrastructure mission
- Showing empathy towards other community members

### Unacceptable Behavior

- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Proposing proprietary integrations or closed-source modules in the core repository
- Other conduct which could reasonably be considered inappropriate in a professional setting

---

## Getting Started

### Before You Contribute

1. Read this document in full -- especially the DCO section
2. Review open issues to find contribution opportunities
3. Join our community -- [info@ekexintelligence.com](mailto:info@ekexintelligence.com)
4. Understand the architecture -- read `/docs/ARCHITECTURE.md` and `/docs/DATABASE.md`

### Contribution Opportunities

#### Tier 1 -- Critical Path (Blocking Protocol Validation)

| # | Area | Impact | Skills Needed | Priority |
|---|---|---|---|---|
| 1.1 | NEZA Signal Categorization | Fix 28% "Unknown" classification rate. Target: <5%. | NLP, ML, Deno | P0 |
| 1.2 | AI Inference Layer Verification | Diagnose why ProductMatch table has zero records. | Debugging, Base44 SDK, Deno | P0 |
| 1.3 | Confidence Scoring Model | Replace hardcoded intent_score with real computation. | Statistics, ML, TypeScript | P0 |

#### Tier 2 -- Protocol Maturity

| # | Area | Impact | Skills Needed | Priority |
|---|---|---|---|---|
| 2.1 | Feedback Loop Implementation | Wire DissatisfactionSignal into confidence adjustment. | Backend, Event-driven, Deno | P1 |
| 2.2 | Unmet Demand Detection | Build logic for market_gap_alert state promotion. | Data pipelines, Base44 Automations | P1 |

#### Tier 3 -- Contributor Infrastructure

| # | Area | Impact | Skills Needed | Priority |
|---|---|---|---|---|
| 3.1 | Local Dev Environment | Reproducible setup in <30 minutes. | Vite, Deno, Base44 | P2 |
| 3.2 | Protocol Documentation | RFC-style specification for all layers. | Technical writing, API design | P2 |

---

## Contribution Workflow

### 1. Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/ekex-core.git
cd ekex-core
git remote add upstream https://github.com/ekexintelligence/ekex-core.git
```

### 2. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/issue-description
```

**Branch naming conventions:**

| Prefix | Purpose | Example |
|---|---|---|
| `feature/` | New features or enhancements | `feature/neza-visual-intent` |
| `fix/` | Bug fixes | `fix/matching-zero-records` |
| `docs/` | Documentation updates | `docs/api-reference-v2` |
| `refactor/` | Code refactoring | `refactor/signal-ingestion` |
| `test/` | Test additions or improvements | `test/confidence-scoring` |

### 3. Make Your Changes

- Follow our [Coding Standards](#coding-standards)
- Write or update tests
- Update documentation
- Ensure all tests pass

### 4. Commit Your Changes (DCO Required)

We require Developer Certificate of Origin (DCO) sign-offs on all commits:

```bash
git commit -s -m "feat: add signal categorization confidence scoring

- Implements intent_score computation based on source reliability
- Adds unit tests for confidence calculation
- Updates API documentation

Signed-off-by: Your Name <your.email@example.com>"
```

The `-s` flag adds the required `Signed-off-by` line. This constitutes your electronic acceptance of the DCO.

### 5. Push and Open a Pull Request

```bash
git push origin feature/your-feature-name
```

Then open a Pull Request on GitHub against the `dev` branch.

### 6. PR Review Process

| Step | Check | Status |
|---|---|---|
| 1 | Automated CI checks (lint, test, build) | Must pass |
| 2 | DCO sign-off present on all commits | Must pass |
| 3 | Maintainers review code quality & architecture fit | Required |
| 4 | Address review feedback promptly | Required |
| 5 | Maintainer merges approved PR | Final |

---

## Development Setup

### Prerequisites

| Software | Version | Purpose |
|---|---|---|
| Node.js | 18.x LTS | Frontend development |
| npm | 9+ | Package management |
| Deno | 1.40+ | Edge Function development |
| Git | 2.40+ | Source control |

### Frontend Setup

```bash
# 1. Clone and enter the repository
git clone https://github.com/ekexintelligence/ekex-core.git
cd ekex-core

# 2. Install dependencies
npm install

# 3. Configure environment
cp .env.example .env.local
# Edit .env.local -- NEVER commit this file

# 4. Start development server
npm run dev
```

Frontend runs at http://localhost:5173

### Edge Function Development

```bash
# Install Deno
curl -fsSL https://deno.land/install.sh | sh

# Test a function locally
deno run --allow-net functions/analyzeImage.js

# Deploy to Deno Deploy (requires API token)
deployctl deploy --project=ekex-intelligence --include=functions/
```

### Base44 Platform Setup

1. Create account at [base44.com](https://base44.com)
2. Create new project "ekex-intelligence"
3. Configure environment variables in dashboard
4. Link local repository to project

---

## Coding Standards

### General Principles

| Principle | Description |
|---|---|
| Readability | Readability over cleverness -- code is read 10x more than written |
| Explicit | Explicit over implicit -- avoid magic numbers, hidden behaviors |
| Type safety | All new code must be typed (TypeScript for frontend, JSDoc for Deno) |
| Defensive | Validate inputs, handle errors gracefully |
| AGPL-3.0 | No proprietary dependencies in core; all network-deployed features must preserve source availability |

### Frontend (React + Vite)

| Rule | Detail |
|---|---|
| Framework | React 18 with Vite build system |
| Styling | Tailwind CSS + shadcn/ui components |
| Routing | React Router v6 |
| State | TanStack Query v5 for server state |
| Animation | Framer Motion for transitions |
| Linting | ESLint and Prettier configurations provided |
| Line length | Maximum 100 characters |
| Async | Use async/await over raw Promises |
| Types | Explicit prop types on all components |

### Edge Functions (Deno)

| Rule | Detail |
|---|---|
| Runtime | Deno Deploy serverless |
| SDK | @base44/sdk v0.8.25 |
| Auth | Always verify user via base44.auth.me() |
| Secrets | Never hardcode; use environment variables |
| Error handling | Return Response.json with appropriate status codes |
| Logging | Use console.error for errors, never expose secrets |

### File Organization

```
src/
|-- pages/              Route-level page components
|   |-- Home.jsx
|   |-- Dashboard.jsx
|   |-- NezaChat.jsx
|   |-- Discover.jsx
|   |-- SupplierPortal.jsx
|   |-- Reports.jsx
|   |-- DemandBoard.jsx
|   |-- AdminPortal.jsx
|-- components/         Reusable UI components
|-- hooks/              Custom React hooks
|   |-- useAdminGuard.js
|-- lib/                Utility functions, SDK clients
|-- main.jsx            Application entry
|-- App.jsx             Router configuration

functions/              Deno Edge Functions
|-- analyzeImage.js
|-- runProductMatching.js
|-- routeSignalToSuppliers.js
|-- sendMatchEmail.js
|-- [16 other functions]
```

---

## Testing Requirements

### Minimum Coverage

| Requirement | Target |
|---|---|
| New features | Unit tests required |
| Bug fixes | Regression tests required |
| API changes | Integration tests required |
| Overall target | 80% line coverage minimum |

### Test Categories

| Type | Tool | Purpose | Command |
|---|---|---|---|
| Unit | Vitest | Individual function/component testing | `npm run test:unit` |
| Integration | Playwright | Full user flow testing | `npm run test:e2e` |
| Coverage | Vitest | Coverage report | `npm run test:coverage` |

### Running Tests

```bash
# All tests
npm test

# Unit tests only
npm run test:unit

# E2E tests
npm run test:e2e

# With coverage
npm run test:coverage
```

---

## Documentation

### Code Documentation

| Requirement | Detail |
|---|---|
| Public functions | JSDoc comments required |
| React components | PropTypes or TypeScript interfaces |
| Edge Functions | Request/response schema documentation |
| Complex algorithms | Inline comments explaining the "why" |
| Configuration | All options documented |

### External Documentation

| Change Type | Update Location |
|---|---|
| API changes | `/docs/apis/` -- OpenAPI spec updates |
| Architecture changes | `/docs/ARCHITECTURE.md` -- System diagrams |
| Database changes | `/docs/DATABASE.md` -- Entity schemas |
| New features | `README.md` -- Project overview |

---

# Developer Certificate of Origin (DCO)

## Version 1.1

By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I have the right to submit it under the GNU Affero General Public License v3.0 (AGPL-3.0); or

(b) The contribution is based upon previous work that, to the best of my knowledge, is covered under an appropriate open source license and I have the right under that license to submit that work with modifications, whether created in whole or in part by me, under the same license (unless I am permitted to submit under a different license), as indicated in the file; or

(c) The contribution was provided directly to me by some other person who certified (a), (b) or (c) and I have not modified it.

(d) I understand and agree that this project and the contribution are public and that a record of the contribution (including all personal information I submit with it, including my sign-off) is maintained indefinitely and may be redistributed consistent with this project or the open source license(s) involved.

---

## Inbound = Outbound License

All contributions to EKEX Intelligence are accepted under the **GNU Affero General Public License v3.0 (AGPL-3.0)**. By submitting a contribution, you agree that your contribution is licensed under AGPL-3.0 and that EKEX Intelligence may distribute your contribution under AGPL-3.0 to all users, including those interacting with the software over a network.

### No Proprietary Sublicensing

You explicitly agree that your contribution will not be relicensed under proprietary or closed-source terms. Any distribution of your contribution, whether by EKEX Intelligence or third parties, must remain under AGPL-3.0 or a compatible free software license.

---

## Attribution

EKEX Intelligence maintains a `CONTRIBUTORS.md` file in the public repository acknowledging all contributors. This acknowledgment is a community courtesy and does not imply any special rights or obligations beyond the AGPL-3.0 license.

---

## Contributor Recognition Program

EKEX Intelligence is committed to recognizing and rewarding contributors within the bounds of our open-source mission:

| Benefit | Description | Eligibility |
|---|---|---|
| CONTRIBUTORS.md | Public acknowledgment of all contributors | All contributors |
| Contributor Grants | Annual grants for significant contributions to the open core | Significant contributors |
| Conference Sponsorships | Support for African tech conference attendance | Active contributors |
| Revenue Sharing | Revenue-sharing from AGPL-3.0 consulting, support, and deployment services | Top contributors |

**Never from proprietary forks or closed-source modules.**

---

## Questions & Support

<div align="center">

**Official Website:** [EKEXintelligence.com](https://EKEXintelligence.com)  
**All Inquiries & Contributions:** [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

</div>

| Channel | Contact | Purpose |
|---|---|---|
| General Questions | [info@ekexintelligence.com](mailto:info@ekexintelligence.com) | All inquiries |
| Legal & Licensing | [info@ekexintelligence.com](mailto:info@ekexintelligence.com) | Legal questions |
| Contributions | [info@ekexintelligence.com](mailto:info@ekexintelligence.com) | Contribution support |
| GitHub Issues | [github.com/ekexintelligence/ekex-core/issues](https://github.com/ekexintelligence/ekex-core/issues) | Bug reports & features |
| Telegram | @ekexcontributors | Community chat (coming soon) |

---

<div align="center">

Thank you for helping build open intent intelligence infrastructure for Africa.

&copy; 2026 Kimuntu Group &middot; EKEX Intelligence &middot; Licensed under GNU AGPLv3

[EKEXintelligence.com](https://EKEXintelligence.com) &middot; [Documentation](https://docs.ekexintelligence.com)

</div>
"""

with open('/mnt/agents/output/CONTRIBUTING.md', 'w') as f:
    f.write(contributing_content)

print("Written: CONTRIBUTING.md")
