<div align="center">

# 🤝 Contributing to EKEX Intelligence

### Open Intent Intelligence Infrastructure for Local Commerce

[![AGPLv3 License](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Welcome](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg)]()
[![DCO Required](https://img.shields.io/badge/DCO-Required-orange.svg)]()
[![Region: Africa](https://img.shields.io/badge/Region-Africa-FFD700.svg)]()

**[🏠 EKEXintelligence.com](https://EKEXintelligence.com)** · **[📚 Documentation](https://docs.ekexintelligence.com)**

**📧 All Inquiries & Contributions:** [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

</div>

---

## 📑 Table of Contents

- [📜 Code of Conduct](#code-of-conduct)
- [🚀 Getting Started](#getting-started)
- [🔄 Contribution Workflow](#contribution-workflow)
- [🛠️ Development Setup](#development-setup)
- [📋 Coding Standards](#coding-standards)
- [🧪 Testing Requirements](#testing-requirements)
- [📝 Documentation](#documentation)
- [✍️ Developer Certificate of Origin (DCO)](#developer-certificate-of-origin-dco)
- [❓ Questions & Support](#questions--support)

---

## 📜 Code of Conduct

### 🤝 Our Pledge

We pledge to make participation in our project a **harassment-free experience** for everyone, regardless of age, body size, disability, ethnicity, sex characteristics, gender identity and expression, level of experience, education, socio-economic status, nationality, personal appearance, race, religion, or sexual identity and orientation.

### ✅ Positive Behavior

| ✅ DO | Description |
|---|---|
| 🗣️ **Inclusive language** | Use welcoming and inclusive language |
| 🤲 **Respect viewpoints** | Be respectful of differing viewpoints and experiences |
| 📥 **Accept criticism** | Gracefully accept constructive criticism |
| 🎯 **Community focus** | Focus on what is best for the community and the open infrastructure mission |
| 💚 **Show empathy** | Show empathy towards other community members |

### 🚫 Unacceptable Behavior

| 🚫 DON'T | Description |
|---|---|
| 🎭 **Trolling** | Trolling, insulting/derogatory comments, personal or political attacks |
| 😤 **Harassment** | Public or private harassment |
| 🔓 **Privacy violations** | Publishing others' private information without explicit permission |
| 🔒 **Proprietary proposals** | Proposing proprietary integrations or closed-source modules in the core repository |
| ❌ **Unprofessional conduct** | Other conduct which could reasonably be considered inappropriate |

---

## 🚀 Getting Started

### 📋 Before You Contribute

1. ✅ **Read this document in full** — especially the DCO section
2. ✅ **Review open issues** to find contribution opportunities
3. ✅ **Join our community** — [info@ekexintelligence.com](mailto:info@ekexintelligence.com)
4. ✅ **Understand the architecture** — read `/docs/architecture/`

### 🎯 Contribution Opportunities

#### 🔴 Tier 1 — Critical Path (Blocking Protocol Validation)

| # | Area | Impact | Skills Needed | Priority |
|---|---|---|---|---|
| 1.1 | 🤖 NEZA Signal Categorization | Fix 28% "Unknown" classification rate. Target: <5%. | NLP, ML, Python/Node.js | 🔴 P0 |
| 1.2 | 🧠 AI Inference Layer Verification | Diagnose why ProductMatch table has zero records. | Debugging, SQL, embedding models | 🔴 P0 |
| 1.3 | 📊 Confidence Scoring Model | Replace hardcoded `intent_score = 1` with real computation. | Statistics, ML, TypeScript | 🔴 P0 |

#### 🟡 Tier 2 — Protocol Maturity

| # | Area | Impact | Skills Needed | Priority |
|---|---|---|---|---|
| 2.1 | 🔄 Feedback Loop Implementation | Wire DissatisfactionSignal into confidence adjustment. | Backend, Event-driven | 🟡 P1 |
| 2.2 | 📊 Unmet Demand Detection | Build logic for market_gap_alert state promotion. | Data pipelines, Scheduling | 🟡 P1 |

#### 🟢 Tier 3 — Contributor Infrastructure

| # | Area | Impact | Skills Needed | Priority |
|---|---|---|---|---|
| 3.1 | 🐳 Local Dev Environment | Reproducible setup in <30 minutes. | Docker, DevOps, Documentation | 🟢 P2 |
| 3.2 | 📝 Protocol Documentation | RFC-style specification for all layers. | Technical writing, API design | 🟢 P2 |

---

## 🔄 Contribution Workflow

### 1️⃣ Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/ekex-core.git
cd ekex-core
git remote add upstream https://github.com/ekexintelligence/ekex-core.git
```

### 2️⃣ Create a Branch

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

### 3️⃣ Make Your Changes

- ✅ Follow our [Coding Standards](#coding-standards)
- ✅ Write or update tests
- ✅ Update documentation
- ✅ Ensure all tests pass

### 4️⃣ Commit Your Changes (DCO Required)

We require **Developer Certificate of Origin (DCO)** sign-offs on **all commits**:

```bash
git commit -s -m "feat: add signal categorization confidence scoring

- Implements intent_score computation based on source reliability
- Adds unit tests for confidence calculation
- Updates API documentation

Signed-off-by: Your Name <your.email@example.com>"
```

> 💡 The `-s` flag adds the required `Signed-off-by` line. This constitutes your electronic acceptance of the DCO.

### 5️⃣ Push and Open a Pull Request

```bash
git push origin feature/your-feature-name
```

Then open a Pull Request on GitHub against the **`dev`** branch.

### 6️⃣ PR Review Process

| Step | Check | Status |
|---|---|---|
| 1 | Automated CI checks (lint, test, build) | Must pass ✅ |
| 2 | DCO sign-off present on all commits | Must pass ✅ |
| 3 | Maintainers review code quality & architecture fit | Required 👥 |
| 4 | Address review feedback promptly | Required 🔄 |
| 5 | Maintainer merges approved PR | Final ✅ |

---

## 🛠️ Development Setup

### 🐳 Using Docker (Recommended)

```bash
# 1️⃣ Clone and enter the repository
git clone https://github.com/ekexintelligence/ekex-core.git
cd ekex-core

# 2️⃣ Copy environment template
cp .env.example .env.local

# 3️⃣ Start all services
docker-compose up -d

# 4️⃣ Install dependencies
npm install

# 5️⃣ Run migrations
npm run db:migrate

# 6️⃣ Seed sample data
npm run db:seed

# 7️⃣ Start development server
npm run dev
```

### 🔧 Manual Setup

See [README.md](README.md#getting-started) for detailed manual setup instructions.

---

## 📋 Coding Standards

### 🎯 General Principles

| Principle | Description |
|---|---|
| 📖 **Readability** | Readability over cleverness — code is read 10x more than written |
| 🔍 **Explicit** | Explicit over implicit — avoid magic numbers, hidden behaviors |
| 🔒 **Type safety** | All new code must be typed (TypeScript) |
| 🛡️ **Defensive** | Validate inputs, handle errors gracefully |
| ⚖️ **AGPL-3.0** | No proprietary dependencies in core; all network-deployed features must preserve source availability |

### 💻 Language-Specific Standards

#### TypeScript / JavaScript

| Rule | Detail |
|---|---|
| Linting | Use ESLint and Prettier configurations provided in repo |
| Line length | Maximum 100 characters |
| Async | Use `async/await` over raw Promises |
| Types | Explicit return types on all public functions |

#### Python (if applicable)

| Rule | Detail |
|---|---|
| Style | Follow PEP 8 |
| Types | Use type hints (Python 3.9+) |
| Docs | Docstrings for all public functions |

#### SQL / Database

| Rule | Detail |
|---|---|
| Queries | Use parameterized queries exclusively |
| Migrations | Migration files must be reversible |
| Performance | Index recommendations required for new queries |

### 📁 File Organization

```
src/
├── neza/
│   ├── __tests__/           🧪 Co-located tests
│   ├── types.ts             📋 Type definitions
│   ├── index.ts             🚀 Public API
│   └── README.md            📖 Layer-specific docs
├── inference/
│   ├── __tests__/
│   ├── types.ts
│   ├── index.ts
│   └── README.md
└── ekex/
    ├── __tests__/
    ├── types.ts
    ├── index.ts
    └── README.md
```

---

## 🧪 Testing Requirements

### 📊 Minimum Coverage

| Requirement | Target |
|---|---|
| New features | Unit tests required ✅ |
| Bug fixes | Regression tests required ✅ |
| API changes | Integration tests required ✅ |
| **Overall target** | 80% line coverage minimum |

### 🧪 Test Categories

| Type | Tool | Purpose | Command |
|---|---|---|---|
| **Unit** | Jest | Individual function/module testing | `npm run test:unit` |
| **Integration** | Jest + Supertest | API endpoint testing | `npm run test:integration` |
| **E2E** | Playwright | Full user flow testing | `npm run test:e2e` |
| **Coverage** | Jest | Coverage report | `npm run test:coverage` |

### 🏃 Running Tests

```bash
# All tests
npm test

# Unit tests only
npm run test:unit

# Integration tests
npm run test:integration

# With coverage
npm run test:coverage
```

---

## 📝 Documentation

### 📖 Code Documentation

| Requirement | Detail |
|---|---|
| Public functions | JSDoc comments required |
| Complex algorithms | Inline comments explaining the "why" |
| Configuration | All options documented |

### 📚 External Documentation

| Change Type | Update Location |
|---|---|
| API changes | `/docs/apis/` — OpenAPI spec updates |
| Architecture changes | `/docs/architecture/` — System diagrams |
| NEZA changes | `/docs/neza/` — Intent engine docs |
| New features | `README.md` — Project overview |

---

# ✍️ Developer Certificate of Origin (DCO)

## Version 1.1

By making a contribution to this project, I certify that:

> **(a)** The contribution was created in whole or in part by me and I have the right to submit it under the GNU Affero General Public License v3.0 (AGPL-3.0); or
>
> **(b)** The contribution is based upon previous work that, to the best of my knowledge, is covered under an appropriate open source license and I have the right under that license to submit that work with modifications, whether created in whole or in part by me, under the same license (unless I am permitted to submit under a different license), as indicated in the file; or
>
> **(c)** The contribution was provided directly to me by some other person who certified (a), (b) or (c) and I have not modified it.
>
> **(d)** I understand and agree that this project and the contribution are public and that a record of the contribution (including all personal information I submit with it, including my sign-off) is maintained indefinitely and may be redistributed consistent with this project or the open source license(s) involved.

---

## ⚖️ Inbound = Outbound License

All contributions to EKEX Intelligence are accepted under the **GNU Affero General Public License v3.0 (AGPL-3.0)**. By submitting a contribution, you agree that your contribution is licensed under AGPL-3.0 and that EKEX Intelligence may distribute your contribution under AGPL-3.0 to all users, including those interacting with the software over a network.

### 🚫 No Proprietary Sublicensing

You explicitly agree that your contribution will **not** be relicensed under proprietary or closed-source terms. Any distribution of your contribution, whether by EKEX Intelligence or third parties, must remain under AGPL-3.0 or a compatible free software license.

---

## 🏷️ Attribution

EKEX Intelligence maintains a `CONTRIBUTORS.md` file in the public repository acknowledging all contributors. This acknowledgment is a **community courtesy** and does not imply any special rights or obligations beyond the AGPL-3.0 license.

---

## 🎁 Contributor Recognition Program

EKEX Intelligence is committed to recognizing and rewarding contributors within the bounds of our open-source mission:

| Benefit | Description | Eligibility |
|---|---|---|
| 📜 **CONTRIBUTORS.md** | Public acknowledgment of all contributors | All contributors |
| 💰 **Contributor Grants** | Annual grants for significant contributions to the open core | Significant contributors |
| ✈️ **Conference Sponsorships** | Support for African tech conference attendance | Active contributors |
| 💵 **Revenue Sharing** | Revenue-sharing from AGPL-3.0 consulting, support, and deployment services | Top contributors |

> **🚫 Never from proprietary forks or closed-source modules.**

---

## ❓ Questions & Support

<div align="center">

**🏠 Official Website:** [EKEXintelligence.com](https://EKEXintelligence.com)  
**📧 All Inquiries & Contributions:** [info@ekexintelligence.com](mailto:info@ekexintelligence.com)

</div>

| Channel | Contact | Purpose |
|---|---|---|
| 📧 **General Questions** | [info@ekexintelligence.com](mailto:info@ekexintelligence.com) | All inquiries |
| 📧 **Legal & Licensing** | [info@ekexintelligence.com](mailto:info@ekexintelligence.com) | Legal questions |
| 📧 **Contributions** | [info@ekexintelligence.com](mailto:info@ekexintelligence.com) | Contribution support |
| 🐛 **GitHub Issues** | [github.com/ekexintelligence/ekex-core/issues](https://github.com/ekexintelligence/ekex-core/issues) | Bug reports & features |
| 💬 **Telegram** | @ekexcontributors | Community chat (coming soon) |

---

<div align="center">

**Thank you for helping build open intent intelligence infrastructure for Africa.** 🌍

*© 2026 Kimuntu Group · EKEX Intelligence · Licensed under GNU AGPLv3*

**[🏠 EKEXintelligence.com](https://EKEXintelligence.com)** · **[📚 Documentation](https://docs.ekexintelligence.com)**

</div>
