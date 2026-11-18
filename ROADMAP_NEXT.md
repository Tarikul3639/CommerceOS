# CommerceOS — Development Roadmap

Updated: 2026-11-19. Product-owner priority: Admin Panel must be completed before Storefront work begins.

## Milestone A — Complete the Admin Panel (current)

Work module-by-module. Follow the established code structure in `AGENTS.md`; complete each module end-to-end across Admin UI and backend API where needed. After verification, report the work and stop for the user's review. Do not start the next module until the user explicitly asks. Never commit or push unless the user explicitly instructs it.

### Module order

1. **Orders** — Completed in this session
   - Admin list/search/pagination/status filters, detail view, fulfilment transitions and cancellation.
   - Backend order-route JWT/permission enforcement, sequential status transitions and cancellation constraints.
2. **Payments** — Admin transaction list/search/filters/details implemented; gateway execution and refund settlement remain deferred pending verified provider adapters/callbacks.
3. **Order Returns** — Completed for the agreed workflow: PENDING → APPROVED or REJECTED; APPROVED → COMPLETED. Rejection reason and decision actors/timestamps are audited. Approval validates cumulative returned quantities; completion restores tracked variant stock transactionally. Refund execution remains separate and deferred until verified gateway refund support exists. Apply the new Prisma migration before using the decision endpoints.
4. **Notifications**
   - Admin list/detail/read/unread and supported notification actions; align with backend API.
5. **Emails** — Completed
   - Paginated email delivery log list/detail and status/search filters.
   - Centralized send-attempt logging with PENDING/SENT/FAILED states and safe template summaries.
   - Staff JWT + EMAIL_READ permission; retry intentionally omitted because no safe replayable payload exists.
6. **Employees** — Completed
   - Added a staff-only directory for MANAGER and EMPLOYEE accounts, using the existing user domain rather than duplicating employee records.
   - Added employee creation with restricted MANAGER/EMPLOYEE role choices and a backend staff-only list filter.
   - Secured all user-management routes with staff JWT authentication and USER_CREATE/READ/UPDATE/DELETE permissions.
7. **Carts** — Implemented and user-verified
   - Read-only Admin overview of non-empty customer carts with pagination, customer/product search, line count, total quantity, estimated merchandise value, last activity and responsive item details.
   - Staff API `GET /admin/carts` is protected by UserJwtAuthGuard + PermissionsGuard + CART_READ. Customer cart endpoints require CustomerJwtAuthGuard and derive customer identity from JWT.
   - Added CART_READ to the backend permission enum and ADMIN seed; migrations have not been applied.
8. **Discounts** — Implemented; user verification pending
   - Added coupon-code CRUD for percentage and fixed discounts, minimum order amount, optional percentage cap, validity dates, optional total usage cap and active state.
   - Admin page includes search, status filter, create/edit and confirmed delete; backend endpoints enforce staff JWT and DISCOUNT_* permissions.
   - Added Prisma schema and migrations, plus ADMIN permission seed. Migrations have not been applied.
   - Coupon redemption, checkout application and usage-limit counting/enforcement are deferred; the configured cap is not yet enforced during order creation.
9. **Settings** — Next module
   - Finish and verify general, shipping, notification, security and payment-gateway settings; ensure secrets are never returned to the browser.
10. **Full Admin consistency and security audit**
    - Reconcile navigation/routes/permissions, backend guards, empty states, errors, responsive layouts and CRUD edge cases across all existing Admin modules.
    - Add meaningful tests for critical business workflows and verify complete production builds.

### Completion gate for every module

- Inspect the matching backend controller/service/DTO/schema and neighboring Admin patterns before editing.
- Use the established folder and naming conventions; no broad rewrites or new architectural style.
- Implement real API integration, permission-aware actions, validation, loading/error/empty states and responsive behavior.
- Run relevant typecheck/build/tests and review `git diff --check`.
- Review the diff and report changed files and verification. Leave changes uncommitted unless the user explicitly asks for a commit.

## Milestone B — Storefront (blocked until Milestone A is complete)

After the Admin Panel is declared complete:
1. Make all route files valid modules and restore a passing build.
2. Implement API client, typed endpoint contracts, cookie/session auth and refresh behavior.
3. Implement public home/catalog/category/product detail.
4. Implement customer registration/login/verification and account state.
5. Implement customer cart using secured cart endpoints.
6. Implement checkout/order creation and payment UI after verifying the order/payment contracts.
7. Implement account orders, profile and return workflows.

## Milestone C — Platform hardening and release readiness

- Complete bKash/SSLCommerz provider sessions, callback/webhook signature verification, idempotency and refund flows.
- Add domain and integration tests for auth, permissions, cart, orders, stock, payments, returns and mail.
- Reconcile backend `.env.example` mismatch (`PRISMA_DATABASE_URL` vs code's `DATABASE_URL`) without exposing real environment values.
- Add health/readiness checks, structured logs, rate limits, monitoring and deployment runbooks.

## Explicitly out of scope unless requested

Google OAuth / Google Calendar is not an evidenced core CommerceOS requirement. Do not prioritize it unless the product owner explicitly requests it.

### Discounts integration note
- Discounts navigation is grouped under Marketing. The standalone Admin CRUD and backend configuration API are implemented; coupon redemption in carts/orders and usage-limit enforcement remain deferred.
- User verification is still the gate before beginning Settings.
