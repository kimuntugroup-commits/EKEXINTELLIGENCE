# Contributing to EKEX Intelligence

Thank you for your interest in contributing to EKEX Intelligence, the demand-supply coordination protocol for African B2B markets. This document outlines our contribution workflow, coding standards, and legal requirements.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Contribution Workflow](#contribution-workflow)
- [Development Setup](#development-setup)
- [Coding Standards](#coding-standards)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)
- [Contributor License Agreement (CLA)](#contributor-license-agreement-cla)
- [CLA Acceptance Process](#cla-acceptance-process)
- [Moral Rights & Jurisdictional Compliance](#moral-rights--jurisdictional-compliance)
- [Questions & Support](#questions--support)

## Code of Conduct

### Our Pledge

We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, sex characteristics, gender identity and expression, level of experience, education, socio-economic status, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

**Positive behavior:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community and the African B2B ecosystem
- Showing empathy towards other community members

**Unacceptable behavior:**
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Other conduct which could reasonably be considered inappropriate in a professional setting

## Getting Started

### Before You Contribute

- Read this document in full — especially the CLA section
- Review open issues to find contribution opportunities
- Join our community — contact info@ekexintelligence.com
- Understand the architecture — read `/docs/architecture/`

### Contribution Opportunities

We have identified the following high-impact contribution surfaces:

#### Tier 1 — Critical Path (Blocking Protocol Validation)

| # | Area | Impact | Skills Needed |
|---|------|--------|---------------|
| 1.1 | Signal Categorization Engine | Fix 28% "Unknown" classification rate. Target: <5%. | NLP, ML, Python/Node.js |
| 1.2 | Matching Engine Verification | Diagnose why ProductMatch table has zero records. | Debugging, Base44 AI, SQL |
| 1.3 | Confidence Scoring Model | Replace hardcoded intent_score = 1 with real computation. | Statistics, ML, TypeScript |

#### Tier 2 — Protocol Maturity

| # | Area | Impact | Skills Needed |
|---|------|--------|---------------|
| 2.1 | Feedback Loop Implementation | Wire DissatisfactionSignal into confidence adjustment. | Backend, Event-driven |
| 2.2 | Gap Detection Automation | Build logic for market_gap_alert state promotion. | Data pipelines, Scheduling |

#### Tier 3 — Contributor Infrastructure

| # | Area | Impact | Skills Needed |
|---|------|--------|---------------|
| 3.1 | Local Dev Environment | Reproducible setup in <30 minutes. | Docker, DevOps, Documentation |
| 3.2 | Protocol Documentation | RFC-style specification for all 5 layers. | Technical writing, API design |

## Contribution Workflow

### 1. Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/EKEXINTELLIGENCE.git
cd EKEXINTELLIGENCE
git remote add upstream https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE.git
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

- Follow our Coding Standards
- Write or update tests
- Update documentation
- Ensure all tests pass

### 4. Commit Your Changes

We require Developer Certificate of Origin (DCO) sign-offs on all commits:

```bash
git commit -s -m "feat: add signal categorization confidence scoring

- Implements intent_score computation based on source reliability
- Adds unit tests for confidence calculation
- Updates API documentation

Signed-off-by: Your Name <your.email@example.com>"
```

The `-s` flag adds the required Signed-off-by line. This constitutes your electronic acceptance of the CLA (see [CLA Acceptance Process](#cla-acceptance-process)).

### 5. Push and Open a Pull Request

```bash
git push origin feature/your-feature-name
```

Then open a Pull Request on GitHub against the main branch.

### 6. PR Review Process

- Automated CI checks must pass (lint, test, build)
- CLA Assistant will verify your CLA acceptance
- Maintainers will review code quality and architecture fit
- Address review feedback promptly
- Once approved, a maintainer will merge your PR

## Development Setup

### Using Docker (Recommended)

```bash
# Clone and enter the repository
git clone https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE.git
cd EKEXINTELLIGENCE

# Copy environment template
cp .env.example .env.local

# Start all services
docker-compose -f docker-compose.example.yml up -d

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

See [README.md](./README.md) for detailed manual setup instructions.

## Coding Standards

### General Principles

- **Readability over cleverness** — Code is read 10x more than written
- **Explicit over implicit** — Avoid magic numbers, hidden behaviors
- **Type safety** — All new code must be typed (TypeScript)
- **Defensive programming** — Validate inputs, handle errors gracefully

### Language-Specific Standards

#### TypeScript / JavaScript

- Use ESLint and Prettier configurations provided in repo
- Maximum line length: 100 characters
- Use async/await over raw Promises
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
├── layer-1-signal/
│   ├── __tests__/           # Co-located tests
│   ├── types.ts             # Type definitions
│   ├── index.ts             # Public API
│   └── README.md            # Layer-specific docs
```

## Testing Requirements

### Minimum Coverage

- All new features must include unit tests
- Bug fixes must include regression tests
- API changes must include integration tests
- Target: 80% line coverage minimum

### Test Categories

| Type | Tool | Purpose |
|------|------|---------|
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

## Documentation

### Code Documentation

- All public functions must have JSDoc comments
- Complex algorithms must include inline comments explaining the "why"
- Configuration options must be documented

### External Documentation

- API changes require OpenAPI spec updates in `/docs/api/`
- Architecture changes require updates in `/docs/architecture/`
- New features require README updates

---

## Contributor License Agreement (CLA)

### IMPORTANT: READ BEFORE CONTRIBUTING

By submitting a Contribution to EKEX Intelligence ("the Project"), you ("Contributor") agree to the following terms. If you do not agree, DO NOT submit any Contribution.

**This Agreement is legally binding.** It grants Kimuntu Group and EKEX Intelligence Ltd. ("EKEX") broad rights to use your Contribution in both open-source and commercial contexts, enabling our business model while ensuring your work is recognized and valued.

### 1. DEFINITIONS

**1.1** "Contribution" means any source code, object code, patch, documentation, design, algorithm, data model, configuration, or other material submitted to the Project, including through GitHub pull requests, issues, or direct submission.

**1.2** "You" or "Your" means the individual or legal entity submitting the Contribution. If submitting on behalf of your employer, you represent that you have authority to bind your employer.

**1.3** "EKEX Intelligence Core" means the software licensed under GNU AGPLv3 as published in the public repository.

**1.4** "Commercial Extensions" means proprietary modules including but not limited to: Mobile Money gateway integrations (MTN MoMo, Wave, Orange Money), USSD connectivity modules, AI demand analytics, KYC/compliance engines, and enterprise customization layers.

### 2. COPYRIGHT LICENSE GRANT

**2.1 Grant of Rights**

Subject to the terms of this Agreement, You hereby grant to EKEX Intelligence Ltd. a perpetual, irrevocable, worldwide, royalty-free, fully paid-up, non-exclusive license, with the right to grant sublicenses, to:

(a) Reproduce, prepare derivative works of, display, perform, distribute, and otherwise use Your Contribution;

(b) Incorporate Your Contribution into the EKEX Intelligence Core or any Commercial Extension;

(c) License Your Contribution under any license, including but not limited to: GNU AGPLv3, proprietary commercial licenses, or any future license chosen by EKEX;

(d) Enforce intellectual property rights in Your Contribution against third parties;

(e) Use Your Contribution to develop, train, and deploy AI/ML models and related services.

**2.2 Retention of Ownership**

You retain ownership of the copyright in Your Contribution. This Agreement does NOT transfer ownership of Your copyright to EKEX. You retain the right to use Your Contribution for any purpose, subject only to the license granted herein.

**2.3 Scope of License**

The license granted in Section 2.1 applies to:

(a) The Contribution as submitted;

(b) Any modifications or derivative works created by EKEX;

(c) Any combination of the Contribution with other EKEX software;

(d) Any future versions or iterations of the EKEX platform.

### 3. PATENT LICENSE GRANT

**3.1 Patent Grant**

You hereby grant to EKEX a perpetual, irrevocable, worldwide, royalty-free, fully paid-up, non-exclusive license under any patent claims owned or controlled by You that are necessarily infringed by the use of Your Contribution, to:

(a) To make, have made, use, sell, offer for sale, import, and otherwise exploit the EKEX platform;

(b) To sublicense these rights to third parties;

(c) To enforce these patent rights against infringers.

**3.2 Patent Retaliation**

If You initiate patent litigation against EKEX or any user of the EKEX platform alleging that the EKEX platform infringes Your patents, Your patent license granted under Section 3.1 shall automatically terminate as of the date of such claim.

### 4. MORAL RIGHTS AND AUTHORSHIP

**4.1 Moral Rights Waiver**

To the fullest extent permitted by applicable law, You hereby waive and agree not to assert any moral rights, droit moral, or similar rights in Your Contribution, including but not limited to:

(a) The right of paternity (right to be identified as author);

(b) The right of integrity (right to object to derogatory treatment);

(c) The right of disclosure (right to control first publication);

(d) The right of withdrawal (right to retract the work);

(e) Any other moral rights recognized under the Berne Convention, OHADA, OAPI, or any national law.

**4.2 Attribution**

Notwithstanding Section 4.1, EKEX agrees to maintain a CONTRIBUTORS.md file in the public repository acknowledging significant contributors. This acknowledgment is a courtesy, not a legal obligation, and does not constitute a waiver of any intellectual property rights.

**4.3 Jurisdictional Variance Acknowledgment**

You acknowledge that moral rights laws vary across African and international jurisdictions:

(a) **OHADA/OAPI Civil Law Jurisdictions** (Benin, Burkina Faso, Cameroon, Central African Republic, Comoros, Congo, Côte d'Ivoire, Equatorial Guinea, Gabon, Guinea, Guinea-Bissau, Mali, Niger, Rwanda, Senegal, Togo): Moral rights may be inalienable under the Bangui Agreement. You agree to a binding covenant not to assert moral rights against EKEX;

(b) **Common Law Jurisdictions** (Nigeria, Kenya, Ghana, South Africa, Tanzania, Uganda, Rwanda): Moral rights may be limited or waivable. You agree to full waiver where permitted;

(c) **Hybrid Jurisdictions**: You agree to the maximum waiver permitted by local law, and where waiver is impossible, to a binding covenant not to sue.

### 5. REPRESENTATIONS AND WARRANTIES

**5.1 Originality**

You represent and warrant that:

(a) Your Contribution is Your original creation;

(b) You have the right to submit the Contribution under this Agreement;

(c) Your Contribution does not infringe any third-party intellectual property rights;

(d) Your Contribution does not violate any applicable law or regulation, including data protection laws (e.g., Rwanda Law No. 058/2021 on Data Protection).

**5.2 Employer Consent**

If You are an employee, You represent that Your employer has approved this Agreement or that Your Contribution is outside the scope of Your employment.

**5.3 Third-Party Code**

If Your Contribution includes third-party code, You represent that:

(a) The third-party code is compatible with GNU AGPLv3;

(b) You have obtained all necessary permissions to include the third-party code under this Agreement;

(c) You have clearly identified all third-party code and its license terms in your Pull Request description.

### 6. INDEMNIFICATION

**6.1 Indemnity Obligation**

You agree to indemnify, defend, and hold harmless EKEX Intelligence Ltd., its affiliates, officers, directors, employees, agents, and licensors from and against any and all claims, damages, losses, liabilities, costs, and expenses (including reasonable attorneys' fees) arising from:

(a) Your Contribution;

(b) Any breach of Your representations or warranties;

(c) Any claim that Your Contribution infringes third-party rights;

(d) Any violation of applicable law by Your Contribution.

**6.2 Scope of Indemnification**

This indemnification obligation survives termination of this Agreement and applies regardless of whether the claim arises in contract, tort, or under any theory of liability.

### 7. AUTOMATED CONTRIBUTION ACCEPTANCE

**7.1 Pull Request Signing**

By creating a GitHub Pull Request ("PR") to the EKEX repository, You AUTOMATICALLY agree to the terms of this Agreement. The act of creating a PR constitutes Your electronic signature and acceptance of this CLA.

**7.2 DCO Alternative**

If You prefer, You may sign-off Your commits using the Developer Certificate of Origin (DCO) format:

```
Signed-off-by: Your Name <email@example.com>
```

This DCO sign-off constitutes acceptance of this Agreement.

**7.3 CLA Assistant Integration**

EKEX uses CLA Assistant or similar automated tools. If the CLA check fails, Your PR will not be merged until You explicitly agree to this Agreement.

**7.4 Contribution Rejection Rights**

EKEX reserves the right to reject any Contribution for any reason, including but not limited to:

(a) Failure to agree to this CLA;

(b) Incompatibility with project goals;

(c) Quality or security concerns;

(d) Potential legal risks.

### 8. TERM AND TERMINATION

**8.1 Perpetual License**

The licenses granted in Sections 2 and 3 are perpetual and irrevocable. They survive any termination of this Agreement.

**8.2 Termination for Breach**

EKEX may terminate this Agreement if You materially breach its terms. Termination does not affect licenses already granted or Contributions already incorporated into the EKEX platform.

**8.3 Survival**

Sections 2, 3, 4, 6, and 9 survive termination of this Agreement.

### 9. MISCELLANEOUS

**9.1 Governing Law**

This Agreement is governed by the laws of Rwanda, without regard to conflict of law principles. The African Union's Model Law on Electronic Commerce (2019) applies to electronic signatures and transactions.

**9.2 Dispute Resolution**

Any dispute arising from this Agreement shall be resolved through binding arbitration under the rules of the Kigali International Arbitration Centre (KIAC). The arbitration shall take place in Kigali, Rwanda.

**9.3 Severability**

If any provision of this Agreement is found unenforceable, the remaining provisions shall continue in full force and effect.

**9.4 Entire Agreement**

This Agreement constitutes the entire agreement between You and EKEX regarding Contributions and supersedes all prior agreements.

**9.5 Modifications**

EKEX may modify this Agreement with 30 days' notice. Continued contributions after such notice constitute acceptance of the modified terms.

---

## CLA ACCEPTANCE PROCESS

### How to Accept This Agreement

#### Option 1: GitHub Pull Request (Recommended)

1. Read this document in full
2. Create a GitHub account (if you don't have one)
3. Submit a Pull Request to any EKEX repository
4. The CLA Assistant bot will prompt you to agree
5. Click "I agree" to electronically sign

#### Option 2: Manual Email Acceptance

1. Read this document in full
2. Send an email to [info@ekexintelligence.com](mailto:info@ekexintelligence.com) with subject: "CLA Acceptance — [Your GitHub Username]"
3. Include the following text: "I have read and agree to the EKEX Intelligence Contributor License Agreement v2.0."
4. Include your full name, GitHub username, and email address

#### Option 3: DCO Sign-Off

1. Configure Git: `git config --global user.name "Your Name"` and `git config --global user.email "your.email@example.com"`
2. Use `git commit -s` for all commits
3. The Signed-off-by line constitutes CLA acceptance

---

## Moral Rights & Jurisdictional Compliance

### Special Provisions for African Contributors

EKEX Intelligence is built by Africans, for Africans. We recognize the diversity of legal traditions across the continent:

#### Francophone Africa (OHADA/OAPI)

We acknowledge that moral rights (droits moraux) may be inalienable under the Bangui Agreement.

Our CLA includes a binding covenant not to sue on moral rights grounds, achieving practical commercial certainty while respecting civil law traditions.

Contributors from these jurisdictions retain their moral rights in form but agree not to enforce them against EKEX.

#### Anglophone Africa (Common Law)

Moral rights are generally waivable or limited in scope.

Our CLA includes a full waiver where permitted by local law.

Contributors from these jurisdictions grant maximum flexibility to EKEX.

#### Lusophone & Other Jurisdictions

We apply the most permissive interpretation permitted by local law.

Where ambiguity exists, the binding arbitration clause in Kigali provides neutral resolution.

### Contributor Recognition Program

Despite the broad license grant, EKEX is committed to recognizing and rewarding contributors:

- **CONTRIBUTORS.md**: Public acknowledgment of all contributors
- **Contributor Grants**: Annual grants for significant contributions
- **Conference Sponsorships**: Support for African tech conference attendance
- **Revenue Sharing**: Top contributors eligible for revenue-sharing from commercial extensions

---

## Questions & Support

| Channel | Purpose |
|---------|---------|
| 📧 [info@ekexintelligence.com](mailto:info@ekexintelligence.com) | General contributor questions, CLA, licensing, and legal inquiries |
| 🐙 [GitHub Issues](https://github.com/kimuntugroup-commits/EKEXINTELLIGENCE/issues) | Technical questions and bug reports |

---

<div align="center">

Thank you for helping build the future of African B2B infrastructure.

© 2026 Kimuntu Group · EKEX Intelligence

</div>
