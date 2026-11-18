# CommerceOS — Development Roadmap

Updated: 2026-11-18. Product-owner priority: Admin Panel must be completed before Storefront work begins.

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
5. **Emails**
   - Email log list/detail, status filters and retry only if backend exposes a safe retry operation.
6. **Employees**
   - Complete employee/staff management using existing users/roles/permissions contracts; avoid duplicating user management without a domain distinction.
7. **Carts**
   - Decide whether this is an admin operational screen or remove unsupported navigation. Build only against a defined, secured backend contract.
8. **Discounts**
   - Define discount rules first; implement schema/API/permissions/Admin UI as a coherent feature. Current Prisma discount model is empty.
9. **Settings**
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