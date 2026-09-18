# Accessibility Tests

**Status:** Placeholder — populated in Phase 7.

Will contain `a11y-key-pages.spec.ts` using `@axe-core/playwright` to scan
Home, Login, Product Detail, Cart, Checkout, and Contact pages for WCAG
violations. Critical/serious violations fail the build; moderate/minor ones
are logged to a report file.
