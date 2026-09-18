# 07 — Automation (Playwright)

**Status:** Placeholder — populated in Phase 7.

## What will go here
A Playwright + TypeScript automation suite using the Page Object Model,
automating the ~15-20 Smoke-tagged test cases from Phase 3:

- `playwright/pages/` — Page Object classes (LoginPage, SignupPage,
  ProductsPage, ProductDetailPage, CartPage, CheckoutPage, ContactPage).
- `playwright/tests/ui/` — UI end-to-end specs, cross-referenced to Phase 3
  Test Case IDs.
- `playwright/tests/api/` — Playwright `request`-based API tests (mirrors the
  Postman collection from Phase 5, runs in CI).
- `playwright/tests/accessibility/` — `@axe-core/playwright` WCAG scans of
  key pages.
- `playwright/playwright.config.ts` — Chromium/Firefox/WebKit config, retries,
  trace/video/screenshot on first retry, HTML reporter.

Run instructions and current pass/fail status will be documented here once
built.
