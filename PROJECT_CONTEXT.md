# CommerceOS — Persistent Project Context

Last audited: 2026-11-18
Workspace: `/home/tarikul3639/Desktop/CommerceOS`
Projectron workspace ID: `bb35b0d3-e720-4bbb-9252-05e93925788d`

## What this project is
CommerceOS is an e-commerce operations platform with three apps:
- `backend/`: NestJS API + Prisma 7 + PostgreSQL.
- `admin/`: Next.js admin dashboard.
- `storefront/`: Next.js customer-facing shop.

This is an e-commerce application. No OAuth or Google Calendar implementation was found in the tracked backend source on this audit. Do not assume Calendar is a product requirement or recommend it as the next milestone without the user asking for it.

## Read first in every new session
1. `AGENTS.md` — rules for working safely and efficiently.
2. `PROJECT_CONTEXT.md` — this short handoff.
3. `PROJECT_ARCHITECTURE.md` — architecture map and verified audit findings.
4. `ROADMAP_NEXT.md` — current priorities.

Do not reread the entire codebase by default. Use this map, then inspect only the requested feature's implementation, dependencies, schema, and tests. Revalidate relevant facts when code may have changed.

## Current architecture at a glance
- Backend has separate user/admin and customer authentication flows, cookie-based JWT access/refresh, role permissions, shared Prisma/mail/Cloudinary/audit infrastructure, catalog, inventory, purchasing, orders, carts, payments, gateway configuration, analytics, dashboard, settings, notifications and activity logs.
- Admin has dashboard routes and feature folders for catalog, customers, users, purchases, returns, settings, analytics, payments and other operations. It uses Redux Toolkit Query.
- Storefront route structure is present, but many tracked page and integration files are zero bytes. Its production build currently fails TypeScript because an account addresses page is not a module.
- The admin production build and backend build pass in this audit; admin typecheck passes. Backend has only a trivial placeholder unit test; meaningful domain tests are absent.

## Highest-priority verified issues
1. **API authorization is not consistently enforced.** The root module does not register a global authentication guard. The order controller has no guards; the cart add-item endpoint uses a hard-coded `CUSTOMER_ID`; admin product endpoints have no guards. Treat this as a security blocker before exposing the API.
2. **Storefront is not implemented yet.** Representative tracked routes and API/auth/store files are zero bytes; `pnpm build` fails on `src/app/(account)/account/addresses/page.tsx` because it is not a module.
3. **Payment gateway execution is incomplete.** Provider adapter/client/config source files for bKash and SSLCommerz are zero bytes; the payment factory always throws; payment creation does not start a provider session. Gateway configuration CRUD exists but does not complete payment processing.
4. **Admin has unfinished screens and route mismatches.** Orders, notifications, emails and employees components are basic heading placeholders; payments is only a heading/description; returns returns null. Navigation contains Discounts and Carts links but matching dashboard routes/features were not found in the tracked route inventory.
5. **Discount domain is incomplete.** `backend/prisma/models/discount.prisma` is zero bytes; discount permissions/enums and admin navigation references exist, but a working discount model/module/route was not found.
6. **Backend environment example is stale.** Validation and Prisma config expect `DATABASE_URL`; `backend/.env.example` currently documents `PRISMA_DATABASE_URL`. Never read actual `.env*` secrets to investigate this.
7. **Test coverage is inadequate.** Backend unit test currently contains an empty assertion; only one test suite exists. Add focused tests around auth/authorization, cart, orders, stock and payments.

## Current verified checks (2026-11-18)
- Backend: `pnpm build` passed.
- Backend: `pnpm test --runInBand` passed, but it ran only one trivial placeholder test.
- Admin: `pnpm typecheck` passed.
- Admin: `pnpm build` passed.
- Storefront: `pnpm build` failed during TypeScript checking because `src/app/(account)/account/addresses/page.tsx` is not a module.
- Git status after checks was clean in backend, admin and storefront.
- No source-code changes were made during the audit; only project documentation/context files are maintained at the monorepo root.

## Working agreement
- Check `git status --short` in the affected app before editing.
- Never read, print, copy, or commit actual `.env*` values. Use `.env.example` only.
- Preserve current architecture and user changes. Avoid broad rewrites.
- Do not run migrations, destructive commands, deployments, or production actions without explicit approval.
- Run targeted tests/builds and report what was actually verified.
- At the end of each meaningful task, update the Task Log below and relevant architecture/roadmap notes.

## Task Log
### 2026-11-18 — Full repository orientation audit
- Mapped tracked source files and major directory structure across backend, admin and storefront.
- Deep-read backend bootstrap/module wiring, environment validation, auth controllers, common infrastructure, Prisma models, order/cart/payment controllers and core services.
- Inspected admin routing/navigation, RTK Query base API, auth state, feature placeholders and package scripts.
- Inspected storefront route inventory, package manifest and representative files; verified representative route/API/auth/store files are zero bytes.
- Ran backend build, backend unit test, admin typecheck/build and storefront build.
- Identified API authorization gaps, incomplete storefront, incomplete payment adapters, incomplete discounts, admin placeholders and environment example mismatch.
- No OAuth/Calendar implementation found in tracked backend source; Calendar is not an evidence-based next feature.
- Next: address API authorization/security first, then finish a minimal customer storefront vertical slice and payment provider flow.
