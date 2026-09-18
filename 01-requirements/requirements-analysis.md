# Requirements Analysis — automationexercise.com

There is no formal PRD for this site (it's a public QA practice target), so this document simulates what a QA engineer does when joining a project without one: read the real, live behavior, infer the functional requirements it implies, and write down the questions a stakeholder/PM would need to answer before those requirements could be signed off. Every ambiguity below was found by actually exercising the site (reading real form markup, submitting real requests, or reading the site's own documentation pages) on 2026-09-19 — none are guessed.

---

## 1. Home Page

**Inferred functional requirements**
- FR: The home page shall display a rotating promotional banner.
- FR: The home page shall list product categories (Women, Men, Kids) and subcategories in a sidebar, each linking to a filtered product list.
- FR: The home page shall list brands with a product count per brand, each linking to a filtered product list.
- FR: The home page shall display a grid of featured products, each with an "Add to cart" and "View Product" action.
- FR: The home page shall display a "recommended items" carousel with its own add-to-cart action.
- FR: The home page shall offer a newsletter signup field in the footer.

**Ambiguities / Questions for stakeholder**
- The brand product counts shown in the sidebar (e.g. "Polo (6)") — are these live counts or cached? Worth checking if adding/removing a product elsewhere updates them.
- "Add to cart" is available directly from the Home grid and the Recommended Items carousel without ever visiting the product detail page — does that bypass the quantity selector, i.e., does it always add quantity 1? No visible way to choose quantity from here.
- No pagination is visible on Home's featured grid — is there a defined maximum number of "featured" products, or does the section grow unbounded as the catalog grows?

---

## 2. Product Listing (All Products) & Search

**Inferred functional requirements**
- FR: `/products` shall display every product in the catalog with name, price, and image.
- FR: Users shall be able to filter the list by category or brand via the sidebar.
- FR: Users shall be able to search products by typing a term into `#search_product` and submitting; results replace the grid under a "Searched Products" heading (per the site's own Test Case 9).
- FR: The same search capability is exposed as a public API (`POST /api/searchProduct`, required param `search_product`).

**Ambiguities / Questions for stakeholder**
- No pagination or product count is shown on `/products` — with a larger catalog, what's the intended behavior (infinite scroll, paging, "load more")? Currently everything renders in one page load, which is a scalability question worth flagging.
- The UI search box submits via a form, but the documented API expects a JSON/form param named `search_product`, not `search` (the UI field's actual HTML `name` attribute). Are the UI and the public API guaranteed to apply identical matching logic (substring vs. exact vs. case-insensitive), or could they diverge? Worth a cross-check test (UI result set vs. API result set for the same term).
- What happens when a search returns zero results — is there a defined "no results" message, or does it just show an empty grid? Needs to be observed directly rather than assumed.
- What characters are sanitized/rejected in the search field? The field has no client-side pattern restriction, so it accepts anything including SQL-metacharacters and script tags — worth a negative test (see Phase 3).

---

## 3. Category Products & Brand Products

**Inferred functional requirements**
- FR: Selecting a category or subcategory shall show only products in that category, with the category name as a page heading.
- FR: Selecting a brand shall show only products from that brand.
- FR: Each of these filtered views shall still allow "Add to cart" per product.

**Ambiguities / Questions for stakeholder**
- If a category or brand has zero products, is there a defined empty-state message? Not directly observed (all categories currently listed have items).
- Brand URLs embed the brand name directly in the path (e.g. `/brand_products/Mast & Harbour` — a name containing a space and an ampersand). Is the server expected to URL-encode/decode this correctly in all cases, and is this a stable enough contract to hardcode brand names in automated tests, or should brand slugs/IDs be used instead? This is a reasonable target for a negative/boundary test.

---

## 4. Product Detail & Review

**Inferred functional requirements**
- FR: The product detail page shall display name, category, price, availability, condition, and brand.
- FR: The quantity field shall default to `1` and shall not allow values below `1` (`min="1"` confirmed in markup).
- FR: "Add to cart" shall add the selected quantity of the product to the cart and confirm with a message + "View Cart" link.
- FR: Users shall be able to submit a review (Name, Email, Review text — all required) without needing to be logged in.

**Ambiguities / Questions for stakeholder**
- The quantity input has **no `max` attribute** and no visible stock-count display. What is the intended maximum? Can a user add, say, 99999 units of an item whose "Availability" says "In Stock" with no quantity given? This is a concrete, real boundary-value question (not assumed) — the field's own markup simply doesn't constrain it.
- The quantity field is `type="number"` but has no `step` restriction — does the server reject non-integer or negative values submitted directly (bypassing the UI), or only the client-side widget?
- Reviews are submitted with just Name + Email + free-text Review, with no CAPTCHA, moderation notice, or rate limit visible. Are reviews published immediately and publicly, and is there any spam/abuse protection? Worth a negative test (script/HTML payload in the review text) to see how it's rendered back.
- No login is required to leave a review — is that intentional (anonymous reviews are allowed) or a gap?

---

## 5. Shopping Cart

**Inferred functional requirements**
- FR: The cart shall list each added product with description, unit price, quantity, and line total.
- FR: An empty cart shall display "Cart is empty!" with a link back to `/products`.
- FR: The cart page shall offer the same newsletter subscribe widget as other pages.
- FR: Users shall be able to proceed from the cart to Checkout.

**Ambiguities / Questions for stakeholder**
- Is cart quantity editable directly from the cart page, or only from the product detail page before adding? The empty-cart state gives no evidence either way — this needs to be verified with an actual item in the cart during manual testing (flagged for Phase 4, since Test Case 13 "Verify Product quantity in Cart" exists on the site's own test-cases page, implying it is editable).
- Is the cart persisted server-side per logged-in account, or session/cookie-based, or both? This matters for the test case "Search Products and Verify Cart After Login" (Test Case 20 on the site) — does adding items while logged out and then logging in preserve the cart, merge it, or discard it? Needs to be observed manually since it depends on session state Claude Code cannot exercise from static HTML fetches alone.

---

## 6. Signup / Account Registration (2-step) & Login / Logout

**Inferred functional requirements**
- FR: Step 1 (`/login` page, POST to `/signup`) shall collect Name and Email only, both required.
- FR: Step 2 (dynamic "Enter Account Information" page) shall collect: title, password, date of birth (day/month/year), first/last name, company, address1/2, country, state, city, zip code, mobile number (field set per the site's own `createAccount` API docs).
- FR: Login shall accept Email + Password, both required, and shall reject invalid credentials with an error.
- FR: Logout shall end the session and return the user to a logged-out state.

**Ambiguities / Questions for stakeholder**
- No password field exists at all in Step 1 — password is only ever set in Step 2. What happens if a user abandons the flow after Step 1 (name+email submitted) — is a partial/passwordless account created server-side, or is nothing persisted until Step 2 completes? This is directly testable and worth a dedicated test case.
- No visible password strength requirement anywhere in the markup (no `minlength`, `pattern`, or helper text on the password field). Is any non-empty string accepted, including a 1-character password? A real, concrete question — not assumed.
- **Observed directly while probing this flow:** submitting a POST to `/signup` that omits an expected hidden field (`form_type`) returns an unhandled Django error page with `DEBUG=True` — i.e., a full stack trace is exposed to the client on a malformed request, rather than a generic 400/500 error page. This was surfaced by a normal (non-malicious) request during exploration, not by exploitation. Flagging as an ambiguity/risk: should malformed requests to this endpoint return a generic error instead of an internal debug trace? (Relevant again in Phase 9 — Security.)
- Registering with an email that already exists is documented on the site's own Test Cases page (Test Case 5) — implying a defined "email already exists" error path exists. Exact wording needs to be captured during manual execution (Phase 4).
- Is there any limit on failed login attempts (lockout/throttling)? Not observable from static markup — flagged for manual negative testing (wrong password N times, per the spec's negative-case guidance).

---

## 7. Checkout & Payment (fake gateway)

**Inferred functional requirements**
- FR: Checkout shall display the logged-in user's delivery and billing addresses and a review of cart contents with a total.
- FR: Users shall be able to add an optional comment to their order before placing it.
- FR: "Place Order" shall proceed to the Payment page.
- FR: Payment shall collect card holder name, card number, CVC, expiry month, and expiry year, all required, and shall simulate a charge without contacting a real payment processor.

**Ambiguities / Questions for stakeholder**
- **Confirmed directly:** both `/checkout` and `/payment` return HTTP 200 and render (with blank addresses and "Rs. 0" total) even when requested with **no login session and an empty cart** — there is no server-side redirect to `/login` or `/view_cart`. Is this the intended behavior for a demo site, or should these routes guard against invalid state? Concrete, testable ambiguity — not assumed.
- The payment fields (`card_number`, `cvc`, `expiry_month`, `expiry_year`) are all plain `type="text"` with **no `maxlength`, `pattern`, or numeric constraint** in the markup. Since this is a fake gateway, is *any* string meant to be accepted (e.g., a 40-character "card number", or letters in the CVC), or is validation expected to happen server-side only? A clear negative/BVA test target.
- No indication of what "Download Invoice after purchase order" (Test Case 24, per the site's own test-cases page) actually produces (PDF? HTML page?) — needs manual verification in Phase 4.

---

## 8. Contact Us

**Inferred functional requirements**
- FR: The contact form shall collect Name, Email, Subject, Message, and an optional file attachment.
- FR: Only Email is enforced as required by the browser (HTML5 `required` attribute is present **only** on the `email` field — confirmed in markup); Name, Subject, and Message carry no `required` attribute despite the form implying they're all part of "the" contact message.
- FR: Submitting the form shall show a confirmation (the site's own Test Case 6 confirms a defined "Contact Us Form" success flow exists).

**Ambiguities / Questions for stakeholder**
- Since Message has no `required` attribute, can a message be submitted with an empty body (just an email)? Is that intended, or a gap versus the obvious intent of a "Contact Us" form?
- The file upload field (`upload_file`) has **no `accept` attribute and no visible size constraint** in the markup. What file types are actually accepted server-side, and what's the maximum size? This is exactly the "oversized file upload" negative case the project spec calls out — needs a real boundary test (Phase 3) since the client gives no hint of the limit.
- Is the "share your feedback" email address (obfuscated via Cloudflare email-protection on the page) actually where messages are delivered, or is that unrelated to what the form submits? Worth clarifying during manual testing.

---

## 9. Newsletter Subscription

**Inferred functional requirements**
- FR: A single email field, present in the footer of every page, shall accept an email address and show "You have been successfully subscribed!" on success.

**Ambiguities / Questions for stakeholder**
- **Confirmed directly in markup on every page fetched (Home, Products, Cart, Contact, Checkout, Payment):** the subscribe field's HTML `id` is `susbscribe_email` — a misspelling of "subscribe" baked into the live site's code, and the surrounding `<form>` element reuses `class="searchform"` (the same class as the unrelated product-search form). This isn't a functional bug users would notice, but it's a real code-quality signal, and automation locators must target the actual (misspelled) id, not the "correct" spelling.
- Is duplicate-email subscription handled (does re-subscribing the same address show success again, an error, or silently no-op)? Not observable from static HTML — flagged for manual testing, and specifically called out because the project brief mentions this as a known site quirk worth verifying.
- Is there email format validation beyond the browser's native `type="email"` check? Needs a manual negative-case pass (invalid formats).

---

## 10. Test Cases (reference) & API Testing (reference) Pages

**Inferred functional requirements**
- FR: `/test_cases` shall list the 26 scenarios the site is designed to support, as a checklist for QA learners (confirmed: read directly from the live page — full list below).
- FR: `/api_list` shall document all public REST endpoints, their methods, required parameters, and expected response codes (confirmed: read directly from the live page — full list below).

**Ambiguities / Questions for stakeholder**
- These two pages are themselves the closest thing this project has to a "spec." Should our own Phase 3 test cases be designed to at least cover all 26 listed scenarios 1:1, or go beyond them? (Decision: Phase 3 will use these as a baseline and add EP/BVA/negative cases the list doesn't cover.)
- The API docs specify exact expected error codes/messages for negative cases (e.g., 405 for wrong method, 400 for missing params) — these become direct, verifiable assertions for Phase 5, not just documentation to read.

**26 Test Cases listed on `/test_cases` (read directly from the live page):**
1. Register User
2. Login User with correct email and password
3. Login User with incorrect email and password
4. Logout User
5. Register User with existing email
6. Contact Us Form
7. Verify Test Cases Page
8. Verify All Products and product detail page
9. Search Product
10. Verify Subscription in home page
11. Verify Subscription in Cart page
12. Add Products in Cart
13. Verify Product quantity in Cart
14. Place Order: Register while Checkout
15. Place Order: Register before Checkout
16. Place Order: Login before Checkout
17. Remove Products From Cart
18. View Category Products
19. View & Cart Brand Products
20. Search Products and Verify Cart After Login
21. Add review on product
22. Add to cart from Recommended items
23. Verify address details in checkout page
24. Download Invoice after purchase order
25. Verify Scroll Up using 'Arrow' button and Scroll Down functionality
26. Verify Scroll Up without 'Arrow' button and Scroll Down functionality

**14 API endpoints listed on `/api_list` (read directly from the live page):**

| # | Name | Method | Endpoint | Notes |
|---|---|---|---|---|
| 1 | Get All Products List | GET | `/api/productsList` | 200 |
| 2 | POST To All Products List | POST | `/api/productsList` | 405, "This request method is not supported" |
| 3 | Get All Brands List | GET | `/api/brandsList` | 200 |
| 4 | PUT To All Brands List | PUT | `/api/brandsList` | 405, "This request method is not supported" |
| 5 | POST To Search Product | POST | `/api/searchProduct` | requires `search_product`; 200 |
| 6 | POST To Search Product without param | POST | `/api/searchProduct` | 400, "Bad request, search_product parameter is missing" |
| 7 | Verify Login with valid details | POST | `/api/verifyLogin` | requires `email`, `password`; 200, "User exists!" |
| 8 | Verify Login without email param | POST | `/api/verifyLogin` | 400, "Bad request, email or password parameter is missing" |
| 9 | DELETE To Verify Login | DELETE | `/api/verifyLogin` | 405, "This request method is not supported" |
| 10 | Verify Login with invalid details | POST | `/api/verifyLogin` | 404, "User not found!" |
| 11 | Create/Register User Account | POST | `/api/createAccount` | requires name, email, password, title, birth_date, birth_month, birth_year, firstname, lastname, company, address1, address2, country, zipcode, state, city, mobile_number; 201, "User created!" |
| 12 | Delete User Account | DELETE | `/api/deleteAccount` | requires `email`, `password`; 200, "Account deleted!" |
| 13 | Update User Account | PUT | `/api/updateAccount` | same fields as create; 200, "User updated!" |
| 14 | Get User Detail By Email | GET | `/api/getUserDetailByEmail` | requires `email`; 200 |

---

## Cross-cutting ambiguities (apply to multiple features)

- **No pagination anywhere observed** (Products, Category, Brand listings) — a scale/performance question worth carrying into Phase 8.
- **`DEBUG=True` Django error pages are reachable from malformed requests** (observed once on `/signup`, see Section 6) — worth a light, non-destructive check on 2-3 other POST endpoints during Phase 9's *passive* scan, since exposing stack traces is itself an informational security finding, not something that requires active/intrusive testing to confirm.
- **No visible rate limiting / CAPTCHA** on login, signup, review, or contact forms — relevant to both negative testing (Phase 3) and the security baseline scan (Phase 9), but out of scope to actively brute-force given this is a shared public site.
