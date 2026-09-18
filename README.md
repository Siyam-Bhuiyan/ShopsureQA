# ShopSure QA

A complete, portfolio-grade Software QA project demonstrating manual testing,
test design, bug tracking, API testing, UI automation, performance testing,
accessibility testing, security scanning, and CI/CD — built end-to-end
against a real, public e-commerce practice site,
[automationexercise.com](https://automationexercise.com).

**Author:** Md. Siyam Bhuiyan

---

## Table of Contents

| Phase | Folder | Contents |
|---|---|---|
| 0 | [`00-setup/`](./00-setup/) | Project setup notes |
| 1 | [`01-requirements/`](./01-requirements/) | Feature inventory, requirements analysis, traceability matrix |
| 2 | [`02-test-plan/`](./02-test-plan/) | Master test plan |
| 3 | [`03-test-cases/`](./03-test-cases/) | Test case design (EP/BVA, negative cases, smoke set) |
| 4 | [`04-bug-reports/`](./04-bug-reports/) | Manual execution log & bug summary |
| 5 | [`05-api-testing/`](./05-api-testing/) | Postman collection + Playwright API tests |
| 6 | [`06-ui-verification/`](./06-ui-verification/) | Figma design spec & UI comparison checklist |
| 7 | [`07-automation/`](./07-automation/) | Playwright UI, API & accessibility automation suite |
| 8 | [`08-performance/`](./08-performance/) | JMeter baseline load test |
| 9 | [`09-security/`](./09-security/) | OWASP ZAP baseline scan report |
| 10 | [`10-final-report/`](./10-final-report/) | Final test summary report |
| 11 | [`.github/workflows/`](./.github/workflows/) | CI/CD pipeline |

---

## Tech Stack

| Area | Tool |
|---|---|
| UI Automation | Playwright + TypeScript |
| API Testing | Postman (+ Newman) and Playwright `request` |
| Performance | JMeter |
| Accessibility | axe-core (`@axe-core/playwright`) |
| Security | OWASP ZAP (baseline scan) |
| CI/CD | GitHub Actions |
| Bug Tracking | Jira (free tier) |
| Test Case Management | Markdown / CSV in-repo |

---

## Status

- [ ] Phase 0 — Setup
- [ ] Phase 1 — Requirement Analysis
- [ ] Phase 2 — Test Plan
- [ ] Phase 3 — Test Case Design
- [ ] Phase 4 — Manual Testing & Bug Reporting
- [ ] Phase 5 — API Testing
- [ ] Phase 6 — UI/UX Verification
- [ ] Phase 7 — Automation (Playwright)
- [ ] Phase 8 — Performance Testing
- [ ] Phase 9 — Security Baseline Scan
- [ ] Phase 10 — Final Report & Repo Polish
- [ ] Phase 11 — CI/CD

---

## Links

- **Jira board:** _to be added once created (Phase 1)_
- **Figma design spec:** _to be added (Phase 6)_
- **Live automation/API reports:** _to be added (Phase 10/11)_

---

## System Under Test

[automationexercise.com](https://automationexercise.com) — a free site built
specifically for QA practice, with signup/login, product catalog, search,
cart, checkout, a contact form, newsletter subscription, and a documented
public API at `/api_list`.
