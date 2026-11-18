# CommerceOS — Development Roadmap

Updated: 2026-11-18 after repository audit.

## Priority 0 — API security and authorization hardening
This should be the immediate next engineering task, before adding new product features.

Observed reasons:
- Root NestJS module does not register a global authentication guard.
- The inspected Orders controller has no authentication/permission guards.
- Cart `POST /cart/items` uses hard-coded `CUSTOMER_ID` instead of `@CurrentCustomer('id')` and a customer guard.
- The inspected admin product controller has no user JWT or permission guards.
- Admin proxy checks cookie presence only; backend must be the security boundary.

Work plan:
1. Inventory every controller and classify routes: public store reads, customer-only, authenticated staff, staff permission-gated, webhook.
2. Apply correct JWT guards and permissions to each protected route; do not blanket-protect public catalog/auth/webhook endpoints.
3. Replace hard-coded customer identity with the authenticated customer decorator and guard.
4. Check ownership on every customer resource (cart, order, payment, profile).
5. Add tests proving unauthenticated, wrong-role and cross-customer requests are rejected.
6. Review audit context so authenticated actor identity is captured after guards execute.

## Priority 1 — Build the customer storefront vertical slice
Storefront is currently a scaffold: representative route/API/auth/store files are zero bytes and production build fails on the empty account addresses page.

Build in this order:
1. Make every route file a valid module and restore a passing build.
2. Implement API client, typed endpoint contracts, cookie/session auth and refresh behavior.
3. Implement public home/catalog/category/product detail using existing `/store/products` endpoints.
4. Implement customer register/login/verification and account state using `/auth/customer`.
5. Implement cart using secured customer cart endpoints (after Priority 0).
6. Implement checkout/order creation and payment UI only after order/customer contracts and payment workflow are verified.
7. Implement customer account orders, profile and return workflows.

## Priority 2 — Complete payment provider integration
Current payment CRUD/configuration is not equivalent to a working gateway:
- bKash and SSLCommerz adapter/client/config source files are empty.
- PaymentProviderFactory always throws.
- CreatePaymentService persists PENDING payments but does not call a gateway.
- Provider callback/webhook verification is not complete.

Before implementation: define provider credentials and secret management, session creation, callback signature verification, amount/currency matching, replay protection, idempotency, status transitions, refund behavior and tests. Never expose credentials in API responses or JSON settings.

## Priority 3 — Finish admin operations
- Implement order list/detail/status/cancel workflows.
- Implement payment list/details/refund workflows and distinguish verified gateway payments from manual cash collection.
- Implement notifications, email log and employee pages.
- Replace ReturnsContent null placeholder with a defined order-return view or remove the route if out of scope.
- Reconcile Discounts and Carts nav entries with actual API/routes.
- Confirm permission names are consistent across Prisma seed, backend enums and admin config.

## Priority 4 — Discounts and business rules
- Decide discount requirements and lifecycle.
- Add Prisma model/migration/DTO/service/controller/permission seeds/admin UI only after business rules are agreed.
- Ensure price calculation is server-authoritative and applied consistently in cart/order/payment.

## Priority 5 — Quality, observability and deployment readiness
- Add meaningful unit/integration/e2e tests for auth, permissions, cart, order, stock, payments, refunds and mail failure behavior.
- Fix backend `.env.example` variable mismatch (`PRISMA_DATABASE_URL` vs code's `DATABASE_URL`).
- Add health/readiness checks, structured logs, rate limits, webhook monitoring and backup/migration runbooks.
- Run full CI checks after each milestone.

## Explicitly not on the current roadmap
Google OAuth / Google Calendar was not found in the tracked backend source and is not evidenced as a core CommerceOS requirement. Do not prioritize it unless the product owner explicitly requests an external calendar integration.
