# 05 — API Testing

**Status:** Placeholder — populated in Phase 5.

## What will go here
- `postman/ShopSureQA.postman_collection.json` — Postman collection covering
  the documented API at `automationexercise.com/api_list`, with tests for
  status code, response time, schema, and data types, including negative
  cases (wrong HTTP method, missing params, invalid credentials).
- `postman/ShopSureQA.postman_environment.json` — environment file with the
  base URL as a variable.
- Instructions for running the collection with Newman and generating an HTML
  report.

The equivalent code-based checks used in CI live in
[`../07-automation/playwright/tests/api/`](../07-automation/playwright/tests/api/).
