# Contributing to EKEX Intelligence

Thank you for your interest in contributing to **EKEX Intelligence**, the open intent intelligence infrastructure for local commerce across Africa. This document outlines our contribution workflow, coding standards, and the **Developer Certificate of Origin (DCO)** process that governs all contributions.

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

### Our Standards

**Positive behavior:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community and the open infrastructure mission
- Showing empathy towards other community members

**Unacceptable behavior:**
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Proposing proprietary integrations or closed-source modules in the core repository
- Other conduct which could reasonably be considered inappropriate in a professional setting

---

## Getting Started

### Before You Contribute

1. **Read this document in full** — especially the DCO section
2. **Review open issues** to find contribution opportunities
3. **Join our community** — contact contributors@ekexintelligence.com
4. **Understand the architecture** — read `/docs/architecture/`

### Contribution Opportunities

We have identified the following high-impact contribution surfaces:

#### Tier 1 — Critical Path (Blocking Protocol Validation)

| # | Area | Impact | Skills Needed |
|---|---|---|---|
| 1.1 | NEZA Signal Categorization | Fix 28% "Unknown" classification rate. Target: <5%. | NLP, ML, Python/Node.js |
| 1.2 | AI Inference Layer Verification | Diagnose why ProductMatch table has zero records. | Debugging, SQL, embedding models |
| 1.3 | Confidence Scoring Model | Replace hardcoded `intent_score = 1` with real computation. | Statistics, ML, TypeScript |

#### Tier 2 — Protocol Maturity

| # | Area | Impact | Skills Needed |
|---|---|---|---|
| 2.1 | Feedback Loop Implementation | Wire DissatisfactionSignal into confidence adjustment. | Backend, Event-driven |
| 2.2 | Unmet Demand Detection | Build logic for market_gap_alert state promotion. | Data pipelines, Scheduling |

#### Tier 3 — Contributor Infrastructure

| # | Area | Impact | Skills Needed |
|---|---|---|---|
| 3.1 | Local Dev Environment | Reproducible setup in <30 minutes. | Docker, DevOps, Documentation |
| 3.2 | Protocol Documentation | RFC-style specification for all layers. | Technical writing, API design |

---

## Contribution Workflow

### 1. Fork and Clone

```bash
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
- `feature/` — New features or enhancements
- `fix/` — Bug fixes
- `docs/` — Documentation updates
- `refactor/` — Code refactoring
- `test/` — Test additions or improvements

### 3. Make Your Changes

- Follow our [Coding Standards](#coding-standards)
- Write or update tests
- Update documentation
- Ensure all tests pass

### 4. Commit Your Changes

We require **Developer Certificate of Origin (DCO)** sign-offs on all commits:

```bash
git commit -s -m "feat: add signal categorization confidence scoring

- Implements intent_score computation based on source reliability
- Adds unit tests for confidence calculation
- Updates API documentation

Signed-off-by: Your Name <your.email@example.com>"
```

The `-s` flag adds the required `Signed-off-by` line. This constitutes your electronic acceptance of the DCO (see [DCO section below](#developer-certificate-of-origin-dco)).

### 5. Push and Open a Pull Request

```bash
git push origin feature/your-feature-name
```

Then open a Pull Request on GitHub against the `dev` branch.

### 6. PR Review Process

1. Automated CI checks must pass (lint, test, build)
2. DCO sign-off must be present on all commits
3. Maintainers will review code quality and architecture fit
4. Address review feedback promptly
5. Once approved, a maintainer will merge your PR

---

## Development Setup

### Using Docker (Recommended)

```bash
# Clone and enter the repository
git clone https://github.com/ekexintelligence/ekex-core.git
cd ekex-core

# Copy environment template
cp .env.example .env.local

# Start all services
docker-compose up -d

# Install dependencies
npm install

# Run migrations
npm run db:migrate

# Seed sample data
npm run db:seed

# Start development server
npm run dev
```

### Manual Setup

See [README.md](README.md#getting-started) for detailed manual setup instructions.

---

## Coding Standards

### General Principles

- **Readability over cleverness** — Code is read 10x more than written
- **Explicit over implicit** — Avoid magic numbers, hidden behaviors
- **Type safety** — All new code must be typed (TypeScript)
- **Defensive programming** — Validate inputs, handle errors gracefully
- **AGPL-3.0 compliance** — No proprietary dependencies in core; all network-deployed features must preserve source availability

### Language-Specific Standards

#### TypeScript / JavaScript
- Use ESLint and Prettier configurations provided in repo
- Maximum line length: 100 characters
- Use `async/await` over raw Promises
- Explicit return types on all public functions

#### Python (if applicable)
- Follow PEP 8
- Use type hints (Python 3.9+)
- Docstrings for all public functions

#### SQL / Database
- Use parameterized queries exclusively
- Migration files must be reversible
- Index recommendations required for new queries

### File Organization

```
src/
├── neza/
│   ├── __tests__/           # Co-located tests
│   ├── types.ts             # Type definitions
│   ├── index.ts             # Public API
│   └── README.md            # Layer-specific docs
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

## Testing Requirements

### Minimum Coverage

- All new features must include unit tests
- Bug fixes must include regression tests
- API changes must include integration tests
- Target: 80% line coverage minimum

### Test Categories

| Type | Tool | Purpose |
|---|---|---|
| Unit | Jest | Individual function/module testing |
| Integration | Jest + Supertest | API endpoint testing |
| E2E | Playwright | Full user flow testing |

### Running Tests

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

## Documentation

### Code Documentation

- All public functions must have JSDoc comments
- Complex algorithms must include inline comments explaining the "why"
- Configuration options must be documented

### External Documentation

- API changes require OpenAPI spec updates in `/docs/apis/`
- Architecture changes require updates in `/docs/architecture/`
- NEZA changes require updates in `/docs/neza/`
- New features require README updates

---

# DEVELOPER CERTIFICATE OF ORIGIN (DCO)

## Version 1.1

By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I have the right to submit it under the GNU Affero General Public License v3.0 (AGPL-3.0); or

(b) The contribution is based upon previous work that, to the best of my knowledge, is covered under an appropriate open source license and I have the right under that license to submit that work with modifications, whether created in whole or in part by me, under the same license (unless I am permitted to submit under a different license), as indicated in the file; or

(c) The contribution was provided directly to me by some other person who certified (a), (b) or (c) and I have not modified it.

(d) I understand and agree that this project and the contribution are public and that a record of the contribution (including all personal information I submit with it, including my sign-off) is maintained indefinitely and may be redistributed consistent with this project or the open source license(s) involved.

## Inbound = Outbound License

All contributions to EKEX Intelligence are accepted under the **GNU Affero General Public License v3.0 (AGPL-3.0)**. By submitting a contribution, you agree that your contribution is licensed under AGPL-3.0 and that EKEX Intelligence may distribute your contribution under AGPL-3.0 to all users, including those interacting with the software over a network.

**No proprietary sublicensing:** You explicitly agree that your contribution will not be relicensed under proprietary or closed-source terms. Any distribution of your contribution, whether by EKEX Intelligence or third parties, must remain under AGPL-3.0 or a compatible free software license.

## Attribution

EKEX Intelligence maintains a `CONTRIBUTORS.md` file in the public repository acknowledging all contributors. This acknowledgment is a community courtesy and does not imply any special rights or obligations beyond the AGPL-3.0 license.

---

## Contributor Recognition Program

EKEX Intelligence is committed to recognizing and rewarding contributors within the bounds of our open-source mission:

- **CONTRIBUTORS.md**: Public acknowledgment of all contributors
- **Contributor Grants**: Annual grants for significant contributions to the open core
- **Conference Sponsorships**: Support for African tech conference attendance
- **Revenue Sharing**: Top contributors eligible for revenue-sharing from **AGPL-3.0 consulting, support, and deployment services** — never from proprietary forks or closed-source modules

---

## Questions & Support

| Channel | Purpose |
|---|---|
| **contributors@ekexintelligence.com** | General contributor questions |
| **legal@ekexintelligence.com** | Licensing and legal inquiries |
| **GitHub Issues** | Technical questions and bug reports |
| **Telegram: @ekexcontributors** | Community chat (coming soon) |

---

<div align="center">

**Thank you for helping build open intent intelligence infrastructure for Africa.**

*© 2026 Kimuntu Group · EKEX Intelligence · Licensed under GNU AGPLv3*

</div>
