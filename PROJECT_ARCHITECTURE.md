# CommerceOS — Architecture & Codebase Map

Audit date: 2026-11-19
This document is a navigation index plus a record of verified findings. It is not a claim that every source line has been exhaustively reviewed. Use targeted file inspection before modifying any feature.

## 1. Repository layout

```text
CommerceOS/
├── backend/       NestJS API, Prisma schema/migrations/seeds
├── admin/         Next.js operations dashboard
├── storefront/    Next.js customer shop
├── AGENTS.md
├── PROJECT_CONTEXT.md
├── PROJECT_ARCHITECTURE.md
└── ROADMAP_NEXT.md
```

Each app is a separate package/repository with its own `.git` directory. Working-tree status is checked at the end of each implementation batch.

## 2. Backend

### Stack and bootstrap
- NestJS 11, TypeScript, Prisma 7, PostgreSQL using `@prisma/adapter-pg`.
- JWT + Passport, bcrypt, class-validator/class-transformer, Joi config validation.
- Nodemailer via `@nestjs-modules/mailer` and Handlebars templates.
- Cloudinary uploads, Swagger, cookie-parser.
- `src/main.ts`: global validation pipe (`whitelist`, `transform`, `forbidNonWhitelisted`), `/api` prefix and URI versioning, credentialed CORS, optional Swagger.
- `src/app.module.ts`: registers feature modules and a global audit-context interceptor. No global APP_GUARD is registered in the inspected module.

### Shared infrastructure
- `src/infrastructure/database`: `DatabaseModule` owns the Prisma-backed `PrismaService`; the Prisma 7 generated client is stored at `src/infrastructure/database/generated/prisma` (configured in `prisma/schema.prisma`). All application, test-mock and seed imports target the infrastructure-generated client directly; `src/lib/` has been removed.
- `src/infrastructure/messaging/email`: `EmailModule`/`EmailService`, Nodemailer transport, delivery templates and email-log persistence.
- `src/infrastructure/storage/cloudinary`: upload signature/controller/service.
- `src/common/guards`, `strategies`, `decorators`: separate user/customer JWT and permission/role primitives. `common/` is reserved for cross-cutting application concerns, not external integrations.
- `src/common/audit`: AsyncLocalStorage-style request audit context and interceptor.
- `src/config`: app/auth/bcrypt/cloudinary/CORS/database/logger/mail/Swagger configs and Joi env schema.

### Feature modules
`src/modules` includes:
- `auth`: separate user and customer login/register/refresh/logout/password reset/email verification flows.
- `users`, `customers`, `permissions`: staff/customer management and role permissions.
- `catalog`: products, product images/options/variants, brands and categories; separate admin/store product controllers.
- `inventory/stocks`: stock reads and manual adjustments.
- `carts`: customer cart CRUD plus a staff-only, read-only Admin cart overview (`GET /admin/carts`) protected by `UserJwtAuthGuard`, `PermissionsGuard` and `CART_READ`.
- Product variant creation/update no longer accepts stock quantities; stock mutations are isolated to STOCK_UPDATE-protected stock adjustment APIs. Stock adjustments use atomic increments and Serializable transactions; variants/products with positive stock cannot be deleted until stock is adjusted to zero through the Stock module. Variant combination generation is capped at 500. Catalog product/image/option/variant mutations now record the authenticated staff actor in Activity Logs. Product Image endpoints do not UUID-parse their identifiers; image URL and Cloudinary public ID replacements are paired.
- `orders`: order lifecycle and order returns.
- `purchases`, `purchase-return`, `suppliers`: supplier purchasing and return lifecycle. Purchase create/update/cancel now validate financial inputs and use conditional PENDING-only state writes. Purchase receiving is deliberately blocked because PurchaseItem references Product rather than ProductVariant; it cannot safely update variant stock. Since purchase returns require RECEIVED purchases, the return-create workflow is currently unreachable through normal APIs. Purchase-return approval now uses a Serializable transaction for cumulative return-quantity checks; completion still lacks safe variant-level stock decrement semantics.
- `payments`: payment records/refunds and provider integration scaffolding. Payment Gateway configuration is consolidated inside the existing `settings` module; Admin/backend naming is `payment-gateway`.
- `banners`, `size-guides`, `settings`, `notifications`, `activity-logs`, `emails`, `dashboard`, `analytics`.

### Database model organization
- `prisma/schema.prisma` configures the generated client and PostgreSQL datasource.
- Models are split into `prisma/models/*.prisma`: user/access, customer, catalog, product, variant, order/cart/returns, purchase/supplier/returns, payment/gateway config, system/audit/notifications/settings/banners.
- Migrations live in `prisma/migrations`; seeds live in `prisma/seed`.
- `discount.prisma` defines coupon-code discounts (PERCENTAGE/FIXED), unique codes, minimum order threshold, optional percentage cap, validity dates, optional total usage cap and staff audit relations. Redemption/usage enforcement is not yet connected to order creation.

### Important backend observations
- Authentication is split between `UserJwtAuthGuard` and `CustomerJwtAuthGuard`; permission checks are an additional `PermissionsGuard`.
- The root module does not apply authentication globally. Protected Admin controllers explicitly apply UserJwtAuthGuard + PermissionsGuard with method-level permission metadata; public catalog GETs and store/products remain intentionally public. Cloudinary upload-signature generation is staff-authenticated and guarded by folder-specific create/update grants (including an allow-list; unsupported folders are denied). Permission checks are backed by current database role grants; SUPER_ADMIN is the only unconditional bypass. Authenticated role-by-role Settings E2E remains pending against a migrated test database.
- Product store read controller is public by design; admin writes must not be.
- Order creation uses a Prisma transaction, Decimal arithmetic, duplicate line validation, product/variant validation and conditional variant stock decrement. Stock is represented on ProductVariant.
- Customer cart add/get/update/remove/clear endpoints are now protected by `CustomerJwtAuthGuard`; customer identity is taken from `@CurrentCustomer('id')`. Admin cart listing is isolated in a separate staff-only controller to avoid mixing customer and staff authentication.
- Payment persistence has idempotency key support and gateway configuration records. Customer payment creation verifies order ownership before returning idempotent responses and leaves staff userId null. Manual settlement permits CASH only; gateway methods cannot be marked paid through an unverified Admin status update. Refund requests link to their original payment and use serializable refundable-balance checks, but remain PENDING until a verified settlement workflow exists. Execution remains incomplete:
  - bKash and SSLCommerz adapter/client/config implementation files inspected are zero bytes.
  - `PaymentProviderFactory.get()` always throws “No verified provider adapter is configured”.
  - `CreatePaymentService` validates order ownership, amount, payment status and enabled gateway, then creates a PENDING row; it does not create a provider session.
  - status update is an administrative/manual status operation; webhook/session verification flow is not completed.
- `discount.prisma` defines the coupon configuration model. Discount CRUD is implemented, but coupon redemption/checkout application and usage-limit enforcement remain deferred.
- Backend `.env.example` names `PRISMA_DATABASE_URL`, while `src/config/env.validation.ts`, `src/config/database.config.ts`, `PrismaService` and `prisma.config.ts` use `DATABASE_URL`. Reconcile the example without inspecting real secret files.

## 3. Admin dashboard

### Stack
- Next.js 16.3, React 19, TypeScript, Tailwind 4, Redux Toolkit + RTK Query, React Hook Form, Zod, TanStack Table, Recharts, shadcn-style UI components.
- `src/lib/api/base-api.ts`: RTK Query base API, cookie credentials, mutex-protected refresh flow, API tags.
- `src/store`: Redux store, auth/notification/UI slices and providers.
- `src/proxy.ts`: protects `/dashboard/:path*` by checking whether `user_access_token` cookie exists; this is a presence check, not token validation. API must enforce auth independently.
- `src/config/navigation.config.ts` and `permissions.config.ts`: dashboard navigation and permission names.
- `src/features/*`: feature-oriented API, types, schemas and UI components.
- `src/app/(dashboard)/dashboard/*`: route entrypoints.

### Feature coverage in route inventory
Routes exist for activity logs, analytics, banners, brands, carts, categories, customers, emails, employees, notifications, orders, payments, permissions, products, purchase returns, purchases, returns, settings, size guides, stock, suppliers and users.

### Important admin observations
- `OrdersContent` has a paginated/searchable/filterable order table, detail dialog, status progression and cancellation workflow. `NotificationsContent` has a permission-aware inbox. `EmailsContent` now has a paginated delivery-log table, status filter and details dialog; backend logs actual mail attempts and exposes read-only endpoints behind `EMAIL_READ`. Retry is intentionally unsupported because no safe replay payload is stored. `EmployeesContent` now reuses the staff user contracts for a paginated manager/employee directory and employee creation via a dedicated `POST /users/employees` contract that restricts roles to MANAGER/EMPLOYEE. The backend user list supports a staff-only filter and all user-management routes enforce staff JWT plus USER_* permissions.
- `PaymentsContent` now provides a paginated/searchable payment and refund transaction table with method/status/type filters and a lazy-loaded detail dialog. The backend payment list query supports payment type filtering and searches transaction reference, invoice number and order ID.
- Gateway session creation, callback verification and refund settlement remain unimplemented; the Admin screen is intentionally read-only and does not imply gateway verification.
- `ReturnsContent` provides a permission-aware, paginated/searchable order-return table with status filters, lazy-loaded details, and ORDER_UPDATE-gated approve/reject/complete actions. Backend decisions are staff-JWT protected: approve/reject only PENDING returns; complete only APPROVED returns. Approval checks cumulative approved/completed quantities against the ordered quantity using a serializable transaction. Rejection requires a reason and records actor/time. Completion records actor/time and restores stock only for tracked product variants in the same transaction. Refunds are not issued automatically; gateway refund settlement remains separate and unimplemented. A migration adds rejection/completion audit fields; it must be applied in the target database before using these endpoints.
- The Carts dashboard is implemented as a read-only active-cart overview backed by a staff-protected paginated API. `CART_READ` exists in Admin and backend permission configuration; its migration must be applied before use. Discounts now has a staff-protected CRUD API and Admin coupon management screen; the new discounts migrations must be applied before use. Coupon redemption, checkout application and usage-limit enforcement remain deferred.
- Settings includes General, Payment Gateway, Shipping, Notification Preferences and Security UI/API surfaces. Mutations in all five settings areas persist actor-attributed Activity Logs in the same Prisma transaction; Size Guide create/update/delete do the same. Banner, Discount, Category, Brand, Supplier and Customer mutations also couple actor audit entries to their database mutations. `GET/PATCH /settings`, `/settings/payment-gateway`, shipping settings and security settings use staff JWT plus dedicated `SETTING_READ/SETTING_UPDATE`; Notification Preferences are also exposed through the Settings module. Gateway settings reject credential-like keys on writes and strip legacy secret-like keys from responses. Payment Gateway is not a separate backend module: its controller, service, DTOs and utility are registered directly in `SettingsModule`. The Admin feature path is `features/settings/components/payment-gateway/` and API/type files use `payment-gateway` naming. Shipping supports district/city flat-rate zones and an optional global free-shipping threshold; checkout/order-total integration is deferred. Notification Preferences persist category toggles for in-app/email; notification creation and operational email sending consult them, but event producer call sites are not comprehensively wired. Security Settings persist and enforce staff access/refresh token lifetimes, remember-me session lifetime, minimum password length, failed-login threshold and lockout duration. Save appears only when the Security form is dirty. Relevant Settings migrations have been created but have NOT been applied.
- The backend has no global staff-authentication guard. The Admin security audit therefore applies UserJwtAuthGuard + PermissionsGuard at controller/method level. Dashboard/Analytics, customers, stock, size guides, suppliers, Admin product CRUD/images/options/variants, notifications, purchases and purchase returns are now protected. Brand/category write methods are protected while GET catalog reads remain public for storefront use; store/products is intentionally public. Staff logout/logout-all/change-password now require UserJwtAuthGuard.
- The Prisma Permission enum is being reconciled with Admin permissions.config.ts. Dashboard/Analytics, Customer, Category, Brand, Product Restore, Supplier, Cart and Notification permissions were added with ordered enum/grant migrations. The Admin permission group UI now includes the missing feature groups and Product Restore. Do not use new guarded routes with an existing database until the corresponding migrations and permission grants have been applied.
- Staff/customer JWT strategies now re-check the current database account on each protected request and reject inactive/deleted accounts; the staff strategy also refreshes role/name/email from the database so stale JWT role claims cannot retain old privileges. CurrentCustomer now reads Passport's request.user payload.
- Refresh/logout token consumption is conditional and atomic for staff and customers, preventing concurrent reuse; staff refresh rotation consumes the old token and creates the replacement in one Serializable transaction. Initial login token persistence rechecks active account and expected password hash inside a Serializable transaction, preventing stale-password login races. CustomerRefreshTokenService uses bcrypt compare with plaintext first. Password reset and email-verification tokens are single-use, require active accounts and are invalidated when email changes or accounts are deactivated/deleted. Forgot-password responses do not reveal whether an account exists.
- Staff invitation creates a one-time password-setup token and sends a welcome email with a setup link; completing password setup also verifies the staff email and revokes any existing sessions. Customer soft deletion revokes refresh sessions and removes outstanding reset/verification tokens.
- Customer list queries can explicitly retrieve soft-deleted customers for restore; default listing excludes them. Customer profile changes and catalog/customer/supplier mutations record the acting staff ID in Activity Logs. Avatar/image URL and Cloudinary public ID updates are paired to prevent mismatched asset references. Admin General Settings and Payment Gateway pages gate reads by SETTING_READ and mutations by SETTING_UPDATE; read-only users can inspect but cannot change gateway configuration. Role permission assignments are SUPER_ADMIN-only at the service boundary to prevent a user with PERMISSION_UPDATE from granting privilege-escalating permissions to other roles. Authenticated users can read their own effective permissions without requiring PERMISSION_READ. CreateActivityLogService supports the caller's Prisma transaction; order, purchase, purchase-return and payment status mutations now persist audit records atomically. Product/variant deletion and option deletion/type changes are blocked when active cart/order/variant references could break historical data or later stock restoration. Purchase receiving remains blocked until PurchaseItem is linked to a product variant. Paid orders cannot be cancelled until refund settlement exists; payment status updates prevent duplicate paid transactions and derive order paymentStatus from all payment attempts.
- Staff account management is least-privilege: only SUPER_ADMIN can create/assign/manage ADMIN accounts; ADMIN cannot modify or delete other ADMIN accounts, cannot change its own role, and cannot delete itself. Non-SUPER_ADMIN user lists/details do not expose SUPER_ADMIN accounts. Deactivation/deletion revokes active refresh sessions; refresh rotation revalidates the current active user and role.
- The Admin UI uses the shared PermissionDenied state and gates read/create/update/delete actions for audited catalog, customer, user, employee, size-guide, dashboard, analytics and Settings pages. Sidebar parent groups remain visible when at least one child action is allowed, so create-only users can reach their permitted create routes. Role permission editing is hidden without PERMISSION_UPDATE and cannot target SUPER_ADMIN.
- This is an in-progress audit, not a completed authorization certification: staff role escalation, remaining UI page/action coverage, purchase/return invariants, service ownership rules and additional focused authorization tests still require review.
- `src/proxy.ts` only checks cookie presence. Do not treat this as API authorization.
- The backend Orders controller now uses `UserJwtAuthGuard` + `PermissionsGuard` with per-route ORDER_CREATE/READ/UPDATE permissions. Order status changes follow PENDING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED; shipped/delivered orders cannot be cancelled.

## 4. Storefront

### Intended route map
- Public: home, product listing/details, categories, brands, search, cart, checkout, contact/about/legal.
- Auth: register, login, forgot/reset password, email verification.
- Account: profile, addresses, orders, returns, notifications, settings.
- Shared layout: header, footer, mobile navigation.
- Planned client modules: API client/endpoints/interceptors, auth/session/token helpers, auth/cart/notification/UI stores and hooks.

### Current observed state
- Storefront package is a minimal Next.js 16.2 + React 19 package.
- Its README and root layout are still create-next-app boilerplate.
- Representative tracked page files for home, product listing, cart, checkout, account pages, auth login, header, API client, auth helpers, cart store and hooks are zero bytes.
- `pnpm build` currently fails TypeScript: `src/app/(account)/account/addresses/page.tsx` is not a module.
- Treat storefront as a scaffold, not a finished customer application.

## 5. Build/test baseline (2026-11-18; historical baseline, not a current full-repository audit)

| App | Check | Result |
|---|---|---|
| backend | `pnpm build` | Pass |
| backend | `pnpm test --runInBand` | Pass, but only one trivial placeholder test |
| admin | `pnpm typecheck` | Pass |
| admin | `pnpm build` | Pass |
| storefront | `pnpm build` | Fail: zero-byte addresses page is not a module |

Current backend verification (2026-11-19): production build passes; Vitest passes 14 files / 44 tests; ESLint passes; `git diff --check` passes. Authenticated role-by-role Settings E2E remains pending because no test session/migrated test database was provided.

## 6. Suggested architectural priorities
1. Enforce authentication and authorization at every protected API boundary; add regression tests.
2. Implement storefront as an end-to-end customer journey using existing public product, customer auth, cart, order and payment API contracts.
3. Complete payment provider integration only after defining secure credential storage, session creation, callback/webhook verification and idempotent status transitions.
4. Complete missing admin order/payment/notification/return workflows and reconcile nav routes.
5. Define discount domain/schema/API/UI or remove unsupported references until it is planned.
6. Expand domain tests and update configuration examples.

## 7. Session workflow
Use `PROJECT_CONTEXT.md` as the fast handoff and this file for orientation. When working on a feature, inspect its exact controller/service/DTO/model/API/UI/tests and direct dependencies only. Update these docs after discoveries or material changes.

### Discounts integration status (2026-11-19)
- Discounts is registered in AppModule, backed by the Prisma Discount model and User creator/updater relations, with staff JWT + DISCOUNT_* permission checks and Admin RTK Query CRUD.
- Admin navigation places Discounts under Marketing beside Banners, not Catalog.
- The module is currently configuration/CRUD only: cart/order checkout does not yet apply coupon codes, and usageLimit is not counted or enforced. Do not describe the module as a complete redemption workflow.


## 8. Infrastructure and payment architecture update (2026-11-19)
- New infrastructure ownership: `src/infrastructure/database/` contains the Prisma module/service; `src/infrastructure/storage/cloudinary/` contains Cloudinary module, service, controller, upload guard, types and DTOs. `src/common/` remains for application-wide guards, strategies, decorators, interceptors, filters, pipes, interfaces, constants and utilities.
- No compatibility facades remain under `src/common/prisma/` or `src/common/cloudinary/`; application imports use `src/infrastructure/...` directly.
- Payment domain remains `src/modules/payments/` (controllers, DTOs, services, strategies, interfaces, enums, security, webhooks and tests). Gateway-specific implementations live in `src/infrastructure/payment-gateways/{bkash,sslcommerz}/`; shared gateway webhook/HTTP utilities live alongside them.
- Strategy Pattern boundary: `modules/payments/strategies` will own the application-facing strategy contract/factory; concrete gateway strategies and provider clients/config/mappers live under `infrastructure/payment-gateways`. The current newly added payment files are empty scaffolding only; no gateway is operational yet.
- `PaymentModule` no longer registers the removed legacy `PaymentProviderFactory`. Payment creation remains persistence-only until the strategy contract and gateway adapters are implemented and wired.


### 2026-11-19 — Direct Prisma infrastructure imports
- Removed `src/common/prisma/` completely. All backend application imports and Vitest mocks now reference `src/infrastructure/database/prisma.service.js` or `prisma.module.js` directly; common guards/mail/strategies also use the infrastructure path.
- No compatibility facade remains under `common/prisma`; `common/` is reserved for cross-cutting application code.
- Cloudinary imports were also migrated to `src/infrastructure/storage/cloudinary` and its nested `services/`, `controllers/`, `guards/` and `types/` folders as part of keeping the infrastructure move buildable.
- Verification after the direct-import migration: backend `npm run build` PASS; `npm test -- --run` PASS (14 files / 44 tests); `git diff --check` PASS. No migration, seed, database write, commit or push.

### 2026-11-19 — Consolidated backend cleanup and verification
- Normalized generated Prisma Client imports across backend source, Vitest mocks and seed modules to `src/infrastructure/database/generated/prisma`; removed `src/lib/` and the former `src/common/prisma/` implementation.
- Completed infrastructure relocation for Prisma/database, email delivery and Cloudinary storage. `DatabaseModule` exposes `PrismaService`; email delivery is provided by `EmailModule`/`EmailService`.
- Payment files under `modules/payments` and `infrastructure/payment-gateways` are architectural scaffolding only; provider sessions, verified callbacks, settlement and refunds are not operational.
- Latest verification: `npm run build` PASS; `npm test -- --run` PASS (14 files / 44 tests); `npm run lint` PASS; `git diff --check` PASS. No migrations, seed execution or database writes were run.
