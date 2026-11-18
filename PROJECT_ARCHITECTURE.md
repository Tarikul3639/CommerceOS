# CommerceOS — Architecture & Codebase Map

Audit date: 2026-11-18
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

Each app is a separate package/repository with its own `.git` directory. The three app working trees were clean after the audit.

## 2. Backend

### Stack and bootstrap
- NestJS 11, TypeScript, Prisma 7, PostgreSQL using `@prisma/adapter-pg`.
- JWT + Passport, bcrypt, class-validator/class-transformer, Joi config validation.
- Nodemailer via `@nestjs-modules/mailer` and Handlebars templates.
- Cloudinary uploads, Swagger, cookie-parser.
- `src/main.ts`: global validation pipe (`whitelist`, `transform`, `forbidNonWhitelisted`), `/api` prefix and URI versioning, credentialed CORS, optional Swagger.
- `src/app.module.ts`: registers feature modules and a global audit-context interceptor. No global APP_GUARD is registered in the inspected module.

### Shared infrastructure
- `src/common/prisma`: Prisma module/service; Prisma client generated under `src/lib/prisma`.
- `src/common/mail`: mail module/service and Handlebars templates for auth, orders and notifications.
- `src/common/cloudinary`: upload signature/controller/service.
- `src/common/guards`, `strategies`, `decorators`: separate user/customer JWT and permission/role primitives.
- `src/common/audit`: AsyncLocalStorage-style request audit context and interceptor.
- `src/config`: app/auth/bcrypt/cloudinary/CORS/database/logger/mail/Swagger configs and Joi env schema.

### Feature modules
`src/modules` includes:
- `auth`: separate user and customer login/register/refresh/logout/password reset/email verification flows.
- `users`, `customers`, `permissions`: staff/customer management and role permissions.
- `catalog`: products, product images/options/variants, brands and categories; separate admin/store product controllers.
- `inventory/stocks`: stock reads and manual adjustments.
- `carts`: customer cart CRUD.
- `orders`: order lifecycle and order returns.
- `purchases`, `purchase-return`, `suppliers`: supplier purchasing and return lifecycle.
- `payments`, `payment-gateway-config`: payment records/refunds and gateway settings.
- `banners`, `size-guides`, `settings`, `notifications`, `activity-logs`, `emails`, `dashboard`, `analytics`.

### Database model organization
- `prisma/schema.prisma` configures the generated client and PostgreSQL datasource.
- Models are split into `prisma/models/*.prisma`: user/access, customer, catalog, product, variant, order/cart/returns, purchase/supplier/returns, payment/gateway config, system/audit/notifications/settings/banners.
- Migrations live in `prisma/migrations`; seeds live in `prisma/seed`.
- `discount.prisma` is currently empty (0 bytes).

### Important backend observations
- Authentication is split between `UserJwtAuthGuard` and `CustomerJwtAuthGuard`; permission checks are an additional `PermissionsGuard`.
- The inspected root module does not apply authentication globally. The inspected order controller has no guards; cart `POST /cart/items` uses the literal `CUSTOMER_ID` instead of the authenticated customer; the inspected admin product controller has no guards. Review every controller and apply correct route-level or global policies before public deployment.
- Product store read controller is public by design; admin writes must not be.
- Order creation uses a Prisma transaction, Decimal arithmetic, duplicate line validation, product/variant validation and conditional variant stock decrement. Stock is represented on ProductVariant.
- Cart controller's add endpoint contains a Bengali TODO comment indicating customer ID should later come from JWT.
- Payment persistence has idempotency key support and gateway configuration records, but execution is incomplete:
  - bKash and SSLCommerz adapter/client/config implementation files inspected are zero bytes.
  - `PaymentProviderFactory.get()` always throws “No verified provider adapter is configured”.
  - `CreatePaymentService` validates order ownership, amount, payment status and enabled gateway, then creates a PENDING row; it does not create a provider session.
  - status update is an administrative/manual status operation; webhook/session verification flow is not completed.
- The `discount.prisma` model file is empty despite discount permissions/references in frontend configuration.
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
Routes exist for activity logs, analytics, banners, brands, categories, customers, emails, employees, notifications, orders, payments, permissions, products, purchase returns, purchases, returns, settings, size guides, stock, suppliers and users.

### Important admin observations
- `OrdersContent` now has a paginated/searchable/filterable order table, detail dialog, status progression and cancellation workflow (implemented 2026-11-18). `NotificationsContent`, `EmailsContent`, `EmployeesContent` remain basic heading placeholders.
- `PaymentsContent` now provides a paginated/searchable payment and refund transaction table with method/status/type filters and a lazy-loaded detail dialog. The backend payment list query supports payment type filtering and searches transaction reference, invoice number and order ID.
- Gateway session creation, callback verification and refund settlement remain unimplemented; the Admin screen is intentionally read-only and does not imply gateway verification.
- `ReturnsContent` provides a permission-aware, paginated/searchable order-return table with status filters, lazy-loaded details, and ORDER_UPDATE-gated approve/reject/complete actions. Backend decisions are staff-JWT protected: approve/reject only PENDING returns; complete only APPROVED returns. Approval checks cumulative approved/completed quantities against the ordered quantity using a serializable transaction. Rejection requires a reason and records actor/time. Completion records actor/time and restores stock only for tracked product variants in the same transaction. Refunds are not issued automatically; gateway refund settlement remains separate and unimplemented. A migration adds rejection/completion audit fields; it must be applied in the target database before using these endpoints.
- Navigation links for Discounts and Carts were present, but corresponding dashboard routes/features were not found in the tracked file list.
- Admin product management appears much further along than order/payment/notifications/returns screens.
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

## 5. Build/test baseline (2026-11-18)

| App | Check | Result |
|---|---|---|
| backend | `pnpm build` | Pass |
| backend | `pnpm test --runInBand` | Pass, but only one trivial placeholder test |
| admin | `pnpm typecheck` | Pass |
| admin | `pnpm build` | Pass |
| storefront | `pnpm build` | Fail: zero-byte addresses page is not a module |

The backend test inventory contains only `src/app.controller.spec.ts`; its assertion body is empty. Do not interpret the green test command as meaningful business-flow coverage.

## 6. Suggested architectural priorities
1. Enforce authentication and authorization at every protected API boundary; add regression tests.
2. Implement storefront as an end-to-end customer journey using existing public product, customer auth, cart, order and payment API contracts.
3. Complete payment provider integration only after defining secure credential storage, session creation, callback/webhook verification and idempotent status transitions.
4. Complete missing admin order/payment/notification/return workflows and reconcile nav routes.
5. Define discount domain/schema/API/UI or remove unsupported references until it is planned.
6. Expand domain tests and update configuration examples.

## 7. Session workflow
Use `PROJECT_CONTEXT.md` as the fast handoff and this file for orientation. When working on a feature, inspect its exact controller/service/DTO/model/API/UI/tests and direct dependencies only. Update these docs after discoveries or material changes.