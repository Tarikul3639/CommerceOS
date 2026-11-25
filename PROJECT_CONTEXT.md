# CommerceOS — Persistent Project Context

Last audited: 2026-11-19
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
- Admin has dashboard routes and feature folders for catalog, customers, users, purchases, returns, carts, settings, analytics, payments and other operations. It uses Redux Toolkit Query.
- Storefront route structure is present, but many tracked page and integration files are zero bytes. Its production build currently fails TypeScript because an account addresses page is not a module.
- The admin production build and backend build pass in this audit; admin typecheck passes. Backend has only a trivial placeholder unit test; meaningful domain tests are absent.

## Highest-priority verified issues
1. **Admin API authorization audit is in progress.** The backend does not use a global authentication guard, so every protected controller must declare guards explicitly. The current audit added staff JWT + permission checks to dashboard, analytics, customers, stock, size guides, suppliers, Admin products/images/options/variants, notifications, purchases and purchase returns; catalog brand/category writes are guarded while their read endpoints remain public for storefront catalog use. Staff logout/logout-all/change-password now require staff JWT. Only store/products remains intentionally public among the controller files that lack guards. Continue reviewing every route's method-level decorators and public/customer boundaries before declaring authorization complete.
2. **Storefront is not implemented yet.** Representative tracked routes and API/auth/store files are zero bytes; `pnpm build` fails on `src/app/(account)/account/addresses/page.tsx` because it is not a module.
3. **Payment gateway execution is incomplete.** Provider adapter/client/config source files for bKash and SSLCommerz are zero bytes; the payment factory always throws; payment creation does not start a provider session. Gateway configuration CRUD exists but does not complete payment processing.
4. **Admin completion is in progress.** Orders, Payments, Returns, Notifications, Emails, Employees, Carts and Discounts have implemented screens. Settings has General, Payment Gateway, Shipping, Notification Preferences and Security surfaces; user verification and the full Admin consistency/security audit remain.
5. **Discount redemption is deferred.** Coupon configuration CRUD is implemented and user-verified. Cart/order checkout application, usage counting/enforcement and per-customer limits are not implemented.
6. **Backend environment example is stale.** Validation and Prisma config expect `DATABASE_URL`; `backend/.env.example` currently documents `PRISMA_DATABASE_URL`. Never read actual `.env*` secrets to investigate this.
7. **Test coverage is inadequate.** Backend unit test currently contains an empty assertion; only one test suite exists. Add focused tests around auth/authorization, cart, orders, stock and payments.

## Current verified checks (baseline: 2026-11-18; later feature checks are recorded in the Task Log)
- Backend: `pnpm build` passed.
- Backend: `pnpm test --runInBand` passed, but it ran only one trivial placeholder test.
- Admin: `pnpm typecheck` passed.
- Admin: `pnpm build` passed.
- Storefront: `pnpm build` failed during TypeScript checking because `src/app/(account)/account/addresses/page.tsx` is not a module.
- Git status after checks was clean in backend, admin and storefront.
- No source-code changes were made during the audit; only project documentation/context files are maintained at the monorepo root.

## Current Storefront milestone
- The user has explicitly directed that Storefront work should begin. The former Admin-first sequencing constraint is superseded by this newer instruction.
- Before Storefront implementation, read `storefront/PROJECT_REQUIREMENTS.md`. It is the persistent source of truth for the approved stack and requirements: Next.js App Router, Native Fetch for server-side API access, TanStack Query for client-side server state, Zustand for client-only UI state, SEO, social sharing, advertising/analytics, multi-store isolation, themes and implementation sequence.
- Storefront must not use Redux Toolkit/RTK Query unless the user explicitly changes the decision. Do not duplicate backend-owned cart/wishlist/customer/order data in Zustand.
- Preserve the existing root-level Storefront structure (`app/`, `components/`, `hooks/`, `lib/`) unless inspection justifies a change. Read the Storefront-specific `AGENTS.md` and installed Next.js 16.4 documentation before implementing Next.js APIs.
- The Admin code/static permission audit is complete; authenticated role-by-role Settings E2E remains unverified against a migrated test database. This is a recorded verification gap, not a reason to disregard the user's explicit Storefront priority.
- Work one Storefront vertical slice at a time, verify it, report changed files and exact checks, and preserve unrelated changes.
- Never commit or push unless the user explicitly instructs it.

## Working agreement
- **Component documentation rule (global):** Every React/UI component created or modified in CommerceOS must include clear, concise explanatory comments documenting its purpose, responsibility and any non-obvious behavior or important logic. Comments must remain accurate and useful; do not add redundant comments that merely restate obvious code. Apply this rule across Backend, Admin and Storefront component work wherever components are present.
- **Storefront protected files (read-only):** `storefront/package.json`, `storefront/providers/theme-provider.tsx`, and `storefront/app/globals.css` may be read/inspected but must not be edited, modified, overwritten, reformatted, renamed, moved or deleted unless the user explicitly grants permission for the specific change. If a task needs one of these files changed, explain why and ask first.
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

### 2026-11-18 — Admin Orders module
- Replaced the Orders heading placeholder with a feature-first RTK Query implementation: paginated/searchable order table, order/payment status filters, order detail dialog, fulfilment status progression and cancellation confirmation.
- Added typed order contracts and endpoints under `src/features/orders/{api,types,components}`.
- Protected backend order routes with user JWT authentication and ORDER_CREATE/READ/UPDATE permissions.
- Enforced sequential order status transitions; disallowed cancellation after shipment or delivery; stock restoration remains transactional on allowed cancellation.
- Verification: Admin `pnpm typecheck` passed; Admin `pnpm build` passed; backend `pnpm build` passed; `git diff --check` passed in both repos. Admin ESLint command is currently blocked by existing ESLint 10 / eslint-plugin-react incompatibility (`contextOrFilename.getFilename is not a function`).
- Next: Payments module, then order returns/returns, notifications, emails, employees, carts/discounts and remaining Admin gap audit. Storefront remains deferred.


### 2026-11-18 — Admin Payments module
- Replaced the Payments placeholder with a permission-aware, paginated transaction table and lazy-loaded transaction details.
- Added search by invoice number, order ID and transaction reference; filters for payment method, status and transaction type (PAYMENT/REFUND).
- Added typed RTK Query endpoints and payment contracts in the existing feature-first structure.
- Extended the backend payment query DTO/service with transaction type filtering and order-ID search. Existing staff JWT and ORDER_READ guards remain in force; no gateway or refund-settlement actions were added because provider verification/settlement is not implemented.
- Verification: Admin `pnpm typecheck` passed; Admin `pnpm build` passed; backend `pnpm build` passed; backend `pnpm test --runInBand` passed (one trivial placeholder test only); `git diff --check` passed in both app repos.
- No commits created. Working trees contain only the Payments changes; user review required before the next module.


### 2026-11-18 — Admin Order Returns module
- Replaced the empty ReturnsContent placeholder with a permission-aware, paginated/searchable order-return table, status filters and lazy-loaded return detail dialog.
- Added typed RTK Query endpoints and return contracts under the existing feature-first structure; registered the OrderReturn cache tag.
- Extended backend order-return search across return number, order invoice and customer name.
- Secured order-return routes with UserJwtAuthGuard + PermissionsGuard: ORDER_CREATE for creating a return; ORDER_READ for list/detail.
- No approve/reject/complete mutation was invented: current backend exposes no such lifecycle endpoint or decision business rules. Admin return management is read-only until those are specified and implemented.
- Verification: Admin `pnpm typecheck` passed; Admin `pnpm build` passed; backend `pnpm build` passed; `git diff --check` passed in both app repos.
- No commits or pushes created. Next planned Admin module: Notifications, but wait for the user's explicit instruction.


### 2026-11-18 — Order Returns lifecycle decisions
- Implemented staff-only approve, reject and complete endpoints guarded by UserJwtAuthGuard + PermissionsGuard + ORDER_UPDATE.
- Approval is restricted to PENDING returns and validates cumulative APPROVED/COMPLETED quantities against each ordered item inside a serializable transaction; concurrent status changes are guarded by conditional updates.
- Rejection is restricted to PENDING returns, requires a 3–500 character reason, and records rejection actor/time/reason.
- Completion is restricted to APPROVED returns, records completion actor/time and restores tracked ProductVariant stock in the same transaction. Products without a variant have no stock counter to restore.
- Added rejection/completion audit fields and relations to the Prisma schema plus migration `20261118220000_add_order_return_decisions`. Prisma client regenerated. Migration was not applied to any database.
- Admin detail dialog now exposes permission-aware approve/reject/complete actions, rejection reason input and decision audit information. Completion explicitly does not issue a refund.
- Verification: Prisma validate/generate passed; backend build passed on retry (first attempt hit a transient ENOTEMPTY while cleaning dist); Admin typecheck/build passed; backend test command passed only the existing trivial placeholder test; git diff --check passed in both apps.
- No commits, pushes or database migrations were run. Existing uncommitted Order Returns changes were preserved.


### 2026-11-18 — Order Returns test seed integration
- Added `backend/prisma/seed/order-returns.seed.ts` as an exported, repeatable development fixture seed and wired it into the existing `prisma/seed/index.ts` runner.
- The main seed runner now runs the existing user and customer seeds first (required fixture relations), then settings and Order Returns test fixtures. Other previously disabled seed steps remain disabled.
- Creates/reuses a dedicated test category, product and tracked variant, then creates three delivered/paid test orders with return cases for PENDING approval, PENDING rejection and APPROVED completion.
- Existing return fixtures are left untouched on reruns so a tested decision is not silently reset.
- Verification: backend `pnpm exec tsc --noEmit` passed; `git diff --check` passed.
- The seed was not executed and no database was changed. Apply the Order Returns migration before running the main seed command.


### 2026-11-18 — Prisma environment loading priority
- Updated `backend/prisma.config.ts` to load `.env.local` before `.env`; because dotenv does not override existing process variables by default, values in `.env.local` take precedence and `.env` fills only missing variables.
- Verification: backend `pnpm exec prisma validate` passed and `git diff --check` passed.
- No environment file contents were inspected or changed; no migration or database operation was run.


### 2026-11-18 — Add dotenv package for Prisma config
- Added `dotenv` as a direct backend dependency so TypeScript can resolve the import used by `prisma.config.ts`; updated `backend/package.json` and `backend/pnpm-lock.yaml`.
- Verification: backend `pnpm exec prisma validate`, `pnpm exec tsc --noEmit` and `git diff --check` passed. Prisma validation confirms `.env.local` loaded before `.env`.
- No migrations or database writes were run; no commits or pushes were created.

### 2026-11-18 — Order Returns navigation and responsive dialog refinement
- Added the Order Returns entry under the Admin Sales navigation group, linking to `/dashboard/returns` and gated by `ORDER_READ`.
- Extracted return details/actions from the table column definition into `src/features/returns/components/return-details-dialog.tsx`; the columns file now focuses on table presentation.
- Reworked the dialog into a responsive, viewport-bounded layout with a separately scrollable content area, compact responsive overview grid, stacked mobile item rows, and a footer that keeps decision controls accessible.
- Kept approve/reject permission-gated; rejection still requires a reason. Added an explicit confirmation before completing a return and restoring tracked variant stock.
- Verification: Admin `pnpm typecheck`, `pnpm build`, and `git diff --check` passed.
- No database migration, seed, commit or push was run. Existing unrelated/previous Order Returns changes in the Admin working tree were preserved; review the full git status before committing.

### 2026-11-18 — Nest Order navigation
- Grouped Sales navigation into an expandable **Orders** parent with **All Orders** (`/dashboard/orders`) and **Returns** (`/dashboard/returns`) children.
- Reused the existing nested-navigation renderer and existing `ORDER_READ` permission; no sidebar component changes were necessary.
- Verification: Admin `pnpm typecheck`, `pnpm build`, and `git diff --check` passed.
- No commit or push created.

### 2026-11-18 — Nest Purchasing navigation
- Grouped Purchasing navigation into an expandable **Purchases** parent with **All Purchases** (`/dashboard/purchases`) and **Returns** (`/dashboard/purchase-returns`) children.
- Reused the existing nested-navigation renderer and `PURCHASE_READ` permission; no sidebar component changes were needed.
- Verification: pending.
- No commit or push created.

### 2026-11-18 — Admin Notifications module
- Replaced the Notifications placeholder with a permission-aware inbox using the existing notification API: paginated list, notification type filter, read/unread filter, unread count, mark-one-read, mark-all-read, detail view and confirmed delete.
- Added typed notification contracts and RTK Query endpoints under `admin/src/features/notifications/{api,types}`; registered the Notification cache tag.
- Added a responsive detail dialog with message, status, timestamps and related entity metadata. Internal notification links are rendered as navigation links only when they are root-relative; external/arbitrary links are displayed as text.
- Protected all backend notification routes with `UserJwtAuthGuard`. Existing service queries/mutations scope records to the authenticated user's ID, preventing access to another user's notifications.
- Backend currently has no mark-as-unread endpoint; Admin supports unread filtering and marking notifications as read, but does not pretend to support a reverse state transition.
- Verification: Admin `pnpm typecheck` passed; Admin `pnpm build` passed and includes `/dashboard/notifications`; backend `pnpm build` passed; `git diff --check` passed in both app repositories.
- No migrations, database writes, commits or pushes were run. Existing Order Returns and other pre-existing working-tree changes were preserved.
- Next Admin module: Emails. Wait for explicit user instruction after their review.


### 2026-11-18 — Notifications toolbar alignment
- Moved the Notifications page's **Mark all** action into the shared DataTable toolbar via `toolbarActions`, matching the Orders/Payments table pattern.
- Moved notification type and read-status filters into the same table toolbar via `toolbarFilters`; removed the standalone summary/filter card and page-header action.
- Kept the unread count in the table description so the table has one consistent control area.
- Verification: Admin `pnpm typecheck` and `git diff --check` passed.
- No unrelated files, commits or pushes.

### 2026-11-18 — Wire header notification control
- Replaced the decorative, non-interactive header bell with a permission-aware notification popover backed by the existing Notifications RTK Query endpoints.
- Header now fetches the latest five notifications, refreshes the list every 60 seconds, displays the live unread count badge, supports marking one/all as read, and navigates to a safe internal notification link or the Notifications inbox.
- Added loading/empty states and a responsive, scrollable popover. Users without `NOTIFICATION_READ` do not trigger the notifications query or see the control.
- Verification: Admin `pnpm typecheck`, `pnpm build`, and `git diff --check` passed.
- No backend/API changes, database operations, commits or pushes.

### 2026-11-18 — Admin Emails module
- Replaced the Emails placeholder with a paginated, searchable email delivery log table, status filtering and a responsive lazy-loaded details dialog.
- Added backend `GET /emails` and `GET /emails/:id`, protected by staff JWT authentication, PermissionsGuard and the new `EMAIL_READ` permission. List supports recipient/subject/related-entity search, status and date filters.
- Connected email logging centrally in `MailService.sendMail`: every actual mail send creates a PENDING log, increments the attempt count, and records SENT or FAILED after the transport result. The stored preview is only a safe template identifier; rendered content, reset links and authentication tokens are never persisted or returned.
- Added Prisma permission migration and a follow-up migration granting `EMAIL_READ` to existing ADMIN and SUPER_ADMIN roles. Added the permission to the role-permission seed definition.
- Retry is intentionally not exposed because the system does not retain a safe replayable template payload.
- Verification: Prisma format/generate passed; backend TypeScript check/build passed; Admin typecheck/build passed; `git diff --check` passed in both apps.
- Migrations were not applied, role permissions were not seeded in the database, and no commit/push was created. Apply the two migrations before using the Emails screen.
- Added `backend/prisma/seed/emails.seed.ts` with six repeatable EmailLog fixtures covering SENT, FAILED and PENDING states; fixture IDs use the `commerceos-email-test` prefix and use safe template summaries rather than rendered email content. The main seed runner invokes it after the existing user/customer/return fixture steps.
- Verification for the seed addition: backend `pnpm exec tsc --noEmit`, `pnpm build`, and `git diff --check` passed.
- Seed was not executed and no database was changed. Existing pending migrations must be applied before the main seed runner can successfully create the EmailLog fixtures.
- Next Admin module: Employees. Wait for user verification before starting it.


### 2026-11-18 — Admin Employees module
- Replaced the Employees placeholder with a paginated staff directory backed by the existing Users domain; only MANAGER and EMPLOYEE records are returned by the dedicated `staffOnly` query filter, avoiding a duplicate employee model.
- Added an employee creation dialog with MANAGER/EMPLOYEE-only role choices and a dedicated `POST /users/employees` DTO/endpoint that enforces the same restriction server-side while reusing the existing account creation and welcome-email behavior.
- Added the Employees navigation link under Management, gated by USER_READ.
- Extended the user query DTO/service with role and staff-only filters.
- Applied UserJwtAuthGuard and PermissionsGuard to the Users controller; create/list/detail/update/status/restore/delete routes now require the corresponding USER_CREATE/READ/UPDATE/DELETE permission. Existing SUPER_ADMIN-only status and restore role checks remain.
- Verification: backend `pnpm exec tsc --noEmit` and `pnpm build` passed; Admin `pnpm typecheck` and `pnpm build` passed; `git diff --check` passed in both app repos.
- No migrations, seed execution, database writes, commits or pushes. Existing unrelated dirty work in Admin/Backend was preserved.
- Next Admin module: Discounts. Carts is implemented; wait for user verification before starting another module.

### 2026-11-19 — Admin Carts module
- Implemented the read-only Admin Carts overview at `/dashboard/carts`: paginated active-cart list, search by customer name/email/phone or product name, product-line count, total quantity, estimated merchandise value, last activity and a responsive details dialog with cart items and variant details.
- Added staff-only `GET /admin/carts` with `UserJwtAuthGuard`, `PermissionsGuard` and `CART_READ`; results are limited to non-empty carts and expose only customer contact basics and cart/product data needed by the screen.
- Added the backend `CART_READ` permission, ADMIN role seed assignment, and two migrations: add the enum value and grant it to existing ADMIN accounts. SUPER_ADMIN continues to bypass explicit permission rows.
- Secured existing customer cart endpoints with `CustomerJwtAuthGuard` and replaced the hard-coded `CUSTOMER_ID` in add-item with `@CurrentCustomer('id')`.
- Estimated values use Prisma Decimal arithmetic and current selling prices; the UI clearly states that the value excludes discounts, shipping and taxes and does not reserve stock. No Admin cart mutation actions were added.
- Verification: Prisma format/generate passed; backend `pnpm exec tsc --noEmit` and `pnpm build` passed; Admin `pnpm typecheck` and `pnpm build` passed (route `/dashboard/carts` generated); backend/Admin `git diff --check` passed.
- No migrations were applied, no seed was run, and no database was changed. Apply the two new CART_READ migrations before using the Admin endpoint. No commit or push was created.
- Next Admin module: Discounts. Wait for user verification before starting it.

### 2026-11-19 — Admin Carts development seed fixtures
- Added `backend/prisma/seed/carts.seed.ts` and wired it into the main seed runner after Order Returns and Email fixtures.
- Adds three deterministic development cart fixtures for the seeded customers Rahim, Karim and Nusrat, using the dedicated Order Returns test product/variant.
- The seed is additive and rerunnable: if a customer already has a cart, it leaves that cart and its contents untouched. It never clears or overwrites existing customer carts.
- The seed requires the existing customer seed and `seedOrderReturns()` to have created the shared test product/variant first.
- Verification: pending.
- The seed was not executed; no database was modified. Run the main seed only after confirming the intended development database and applying required migrations.

### 2026-11-19 — Admin Discounts module
- Added the Prisma Discount model and DiscountType enum for normalized unique coupon codes, percentage/fixed values, minimum order amount, optional percentage cap, start/end validity, optional total usage cap, active state and staff audit relations.
- Added staff-protected CRUD endpoints at /discounts, guarded by UserJwtAuthGuard + PermissionsGuard and DISCOUNT_CREATE/READ/UPDATE/DELETE. Codes are normalized to uppercase; duplicate codes conflict; percentage values are capped at 100; invalid date ranges and fixed-discount caps are rejected.
- Added the Discounts Admin page with paginated/searchable list, active filter, create/edit dialog, validation, status display and confirmed deletion.
- Updated ADMIN permission seeding and added a migration to grant existing ADMIN roles the four discount permissions. Existing MANAGER read permission remains.
- Important limitation: coupon redemption/checkout application, usage counting/enforcement and per-customer limits are not implemented. The configured total usage limit is stored for future checkout integration; it is not yet enforced by order creation. No automatic discounts.
- Verification: Prisma format/generate/validate passed; backend pnpm exec tsc --noEmit and pnpm build passed; Admin pnpm exec tsc --noEmit and pnpm build passed (route /dashboard/discounts generated); git diff --check passed in both app repos.
- No migrations or seeds were executed; no database changed. No commit or push. The unrelated pre-existing Admin change to src/components/media/app-image.tsx was preserved.
- Next Admin module: Settings; wait for Discounts verification before starting it.


### 2026-11-19 — Discounts form validation alignment
- Reviewed existing Admin feature patterns in Customers, Brands and Stock. These place Zod schemas under `features/<feature>/schemas` and use `react-hook-form` with `zodResolver`, displaying field-level errors.
- Refactored `admin/src/features/discounts/components/discount-form-dialog.tsx` to use `useForm`, `zodResolver`, `register`/`Controller`, reset-on-open edit/create values and inline field errors.
- Added `admin/src/features/discounts/schemas/discount.schema.ts` with coupon code, positive amount, percentage cap, minimum order, optional cap, date ordering and positive integer usage limit validation.
- Payload conversion remains explicit at submit time; values are normalized and converted to API number/ISO date/null fields. Existing create/update mutation behavior and loading/error toasts are retained.
- Verification: Admin `pnpm exec tsc --noEmit`, `pnpm build` (including `/dashboard/discounts`) and `git diff --check` passed.
- No migrations, seeds, commits or pushes run. Existing unrelated Admin working-tree changes were preserved.

### 2026-11-19 — Discounts integration review and commit
- Reviewed the Discounts backend module wiring, Prisma/User audit relations, DISCOUNT_* permission seeding, Admin RTK Query feature, route entrypoint and navigation placement.
- Moved the Discounts navigation entry from Catalog to Marketing beside Banners, matching its promotional/coupon-management responsibility. The route remains `/dashboard/discounts` and is gated by `DISCOUNT_READ`.
- Confirmed Discounts is integrated as its own backend feature module and Admin feature; it is not yet integrated into cart/order checkout or coupon redemption. Usage-limit configuration is stored but not enforced. Those are known domain gaps, not silently implied functionality.
- Preserved and included all existing working-tree changes requested for this commit, including the pending Carts formatting/response-description changes and Admin image styling/base API newline adjustments.
- Verification: backend `pnpm exec tsc --noEmit`, `pnpm build`, `pnpm exec prisma validate`; Admin `pnpm exec tsc --noEmit`, `pnpm build`; `git diff --check` passed after whitespace cleanup.
- No database migrations or seeds were executed. Discounts verification by the user remains pending; Settings must not start until the user confirms and explicitly asks.
- Commits: pending at time of this log entry.

### 2026-11-19 — Fix product image tab for products without images
- Root cause: the update route parameter can be a product slug, while the Product Images API requires a UUID (`ParseUUIDPipe`). The update form passed the route parameter into `ProductImages`, causing its image query to fail and the component to render an error state before exposing the upload controls.
- Changed `admin/src/features/products/components/update/update-product-form.tsx` to pass the loaded product's canonical UUID (`product.id`) to `ProductImages`, consistent with the Options and Variants tabs.
- Verification: Admin `pnpm exec tsc --noEmit`, `pnpm build` and `git diff --check` passed. No commit or push created. User verification pending.

### 2026-11-19 — Standardize Admin edit/update actions
- Standardized visible Admin edit/update action controls on `SquarePen` with `className="size-4"` and the label `Update`.
- Updated Discounts, Users, Product Variants, Product Details, Payment Gateway cards and Banner list/detail actions. Removed the now-unused Pencil icon import from the permissions update dialog. Existing Customers, Brands, Size Guides, Products list, Categories, Purchases, Stock, Suppliers and Permissions actions already used the requested pattern.
- Verification: Admin `pnpm exec tsc --noEmit`, `pnpm build` and `git diff --check` passed. Changes remain uncommitted; user review pending.

### 2026-11-19 — Keep product image upload available when image fetch fails
- Confirmed backend `GetProductImagesService` returns an empty array for an existing product with zero images; it throws `Product not found` only when the product itself is missing. The Admin previously replaced the entire image editor with a blocking `ErrorState` whenever the GET query errored.
- Updated `admin/src/features/products/components/form/product-images.tsx`: query errors now show a non-blocking inline warning and Retry button while preserving the file picker, drag/drop and Save Images controls. Empty image arrays continue to render the normal empty upload state.
- Verification: Admin `pnpm exec tsc --noEmit`, `pnpm build` and `git diff --check` passed. No commit or push. User should retest the Images tab and add/save flow.

### 2026-11-19 — Accept legacy product identifiers in product-image routes
- User reported `POST /api/v1/admin/product-images/commerceos-return-test-product` failing with `Validation failed (uuid is expected)`. Confirmed the Admin detail route and product response can supply this legacy slug-like product identifier, while the product-image controller incorrectly enforced UUID parsing on `productId` for both GET and POST.
- Removed `ParseUUIDPipe` only from product-image `productId` route parameters; retained UUID validation for `imageId` on PATCH/DELETE because image IDs are UUIDs. Product-image services continue using the supplied product identifier to find the product.
- Verification: Backend `pnpm exec tsc --noEmit`, `pnpm build`, and `git diff --check` passed. No database changes, commit, or push. User should retry loading and adding images.

### 2026-11-19 — Settings module foundation and access hardening
- Inspected the existing Settings implementation. General company/branding/regional settings and payment-gateway configuration UI/API already existed; Shipping, Notifications and Security routes were placeholders, with no corresponding persisted policy models or enforcement services.
- Secured `GET/PATCH /settings` with `UserJwtAuthGuard`, `PermissionsGuard` and dedicated `SETTING_READ/SETTING_UPDATE` permissions. Added Prisma enum values, an ADMIN-only permission grant migration and matching seed entries. SUPER_ADMIN receives the permissions through the existing all-permissions seed behavior.
- Changed payment-gateway configuration endpoints to use settings permissions instead of unrelated ORDER_READ/ORDER_UPDATE permissions.
- Added recursive gateway-settings input validation that rejects credential-like keys and response sanitization that strips any legacy secret-like keys. Gateway credentials remain server-side environment/secret-manager configuration only.
- Added a shared Settings navigation layout for General, Payment gateways, Shipping, Notifications and Security. Replaced misleading Shipping/Notifications/Security placeholder text with explicit notices explaining that these settings are not yet backed by persisted policy models or runtime enforcement.
- Verification: Admin `pnpm exec tsc --noEmit` and `pnpm build` passed; Backend `pnpm exec tsc --noEmit`, `pnpm build`, `pnpm exec prisma validate` and `git diff --check` passed.
- Added migrations `20261119120000_add_settings_permissions` and `20261119120100_grant_settings_permissions`. They have NOT been applied. No seed, database mutation, commit or push was performed.
- Remaining scope before Settings can be called complete: define shipping-rate/free-shipping policy and order calculation behavior; define store-wide notification preference model and delivery integration; define security policy requirements and wire them into authentication/session enforcement. These were not invented because no product rules or backend support exist yet.
- Removed the Settings sidebar after user feedback; the Settings content now uses the full available width. Admin TypeScript check and `git diff --check` passed after this adjustment.
- Next: user review of Settings foundation and confirmation of the missing shipping/notification/security policy requirements. Do not begin another module until explicit instruction.


### 2026-11-19 — Settings: Shipping Settings implementation
- Implemented district/city-based shipping zones with a configurable flat delivery rate per zone, active/inactive state, case-insensitive normalized uniqueness, and optional global free-shipping threshold (null disables it).
- Backend: added ShippingSettings and ShippingZone Prisma models, migration 20261119130000_add_shipping_settings, DTOs, staff-JWT/SETTING_READ or SETTING_UPDATE-protected API at /shipping-settings and /shipping-settings/zones, and CRUD service validation.
- Admin: replaced the Shipping placeholder with a responsive configuration screen for the free-shipping threshold and district/city zone create/update/delete, status and empty/loading/error states. API contracts and RTK Query endpoints added.
- Important boundary: these values are persisted configuration only. They are not yet used to calculate order totals or checkout shipping; that integration is deferred until the customer checkout/address flow is implemented. No default Bangladesh zones or delivery prices were invented.
- Verification: backend Prisma format/generate/validate passed; backend pnpm exec tsc --noEmit and pnpm build passed; Admin pnpm exec tsc --noEmit and pnpm build passed; git diff --check passed in both apps.
- No migration was applied, no seed/database operation was run, and no commit or push was created. The unrelated pre-existing Admin/Backend working-tree changes were preserved.
- Next: stop for user verification of Shipping Settings before beginning Notification Preferences.

### 2026-11-19 — Shipping Settings card and dialog layout
- Corrected the target page after user clarification: the requested card layout and Create/Update dialogs apply to `/dashboard/settings/shipping`, not Discounts.
- Reverted the accidental Discounts page layout change from the immediately preceding turn; no Discounts page change remains from that request.
- Updated `admin/src/features/settings/components/shipping/shipping-settings-content.tsx`: each district/city shipping zone is now shown as a responsive card with rate and active status; Create shipping zone and Update open the same responsive dialog form. Delete remains a separate confirmation dialog. The free-shipping threshold remains a dedicated inline setting.
- Verification: Admin Prettier, `pnpm exec tsc --noEmit`, `pnpm build`, and `git diff --check` passed.
- No backend/API/schema/database changes, migration, seed, commit or push. User verification pending.

### 2026-11-19 — Refine Shipping Settings UI and feature structure
- Refined the district/city shipping-zone cards with a restrained premium visual treatment: subtle primary gradient accent, elevated hover state, clear status badge, emphasized rate panel, and cleaner action footer. Updated the free-shipping threshold panel to use the shared Card primitives.
- Refactored the former single large Shipping Settings component into focused components under the existing feature-first path `admin/src/features/settings/components/shipping/`:
  - `shipping-settings-content.tsx` — page composition, query/mutations, permissions and notifications.
  - `shipping-threshold-card.tsx` — free-shipping threshold form.
  - `shipping-zone-list.tsx` — zone grid and empty state.
  - `shipping-zone-card.tsx` — individual zone presentation/actions.
  - `shipping-zone-form-dialog.tsx` — reusable create/update dialog and form state.
- Kept API contracts and business behavior unchanged; no checkout calculation integration was added.
- Verification: Admin Prettier, `pnpm exec tsc --noEmit`, `pnpm build`, and `git diff --check` passed.
- No backend/database changes, migration, seed, commit or push. Other pre-existing working-tree changes were preserved. User verification pending.

### 2026-11-19 — Settings Notification Preferences implementation and Security Settings completion
- Notification Preferences: added the singleton `NotificationPreferences` Prisma model and migration `20261119131500_add_notification_preferences`; added Settings GET/PATCH API, request/response DTOs, Admin types/API and category-based in-app/email toggles.
- Notification creation consults the matching in-app preference; operational email sending consults email preferences. Event producer call sites have not been comprehensively wired, so delivery integration is partial. Migration has NOT been applied.
- Security Settings: added singleton `SecuritySettings` model, migration `20261119160000_add_security_settings`, DTOs, response DTO, Settings controller/service and Admin form/schema/types/API.
- Security defaults: access token 15 minutes, standard refresh session 7 days, remember-me session 30 days, minimum password length 12, maximum failed login attempts 5, lockout 15 minutes. Staff token/refresh generation, remember-me refresh lifetime, failed-login lockout, password change and reset flows enforce the configured policies.
- Admin Security form follows feature conventions with Zod schema in `features/settings/schemas`; Save appears only while values are dirty and hides again when values are reverted.
- Verification: Prisma format/generate/validate and backend `npm run build` passed; Admin `npm run build` passed after moving schema to the shared `schemas/` folder.
- Both migrations remain unapplied; no database migration or seed was run.
- Security implementation was committed at user request: Admin `b68018f`, Backend `c4113ee`, root submodule update `9f78fa6`. No remote push.
- Next: continue the Full Admin consistency/security audit per the product owner's explicit priority; Settings end-to-end verification remains pending.

### 2026-11-19 — Full Admin consistency/security audit (batch 1: route authorization and permission catalog)
- Added staff JWT authentication and DASHBOARD_READ / ANALYTICS_READ permission checks to all Dashboard and Analytics endpoints.
- Reconciled the Prisma Permission enum with the Admin permission catalog, adding missing Customer, Category, Brand, Product Restore, Supplier, Cart and Notification permissions. Added ordered enum/grant migrations: 20261119170000_add_dashboard_analytics_permissions, 20261119170100_grant_dashboard_analytics_permissions, 20261119170200_sync_admin_permission_catalog, 20261119170300_grant_admin_permission_catalog.
- Updated permission seed assignments for ADMIN/MANAGER/EMPLOYEE and completed the Admin permission-group matrix (Dashboard, Analytics, Brands, Carts, Activity Logs, Emails, Notifications, Settings, Product Restore). Size Guides navigation now requires PRODUCT_READ.
- Added route-level staff JWT + permission enforcement to Customer, Stock, Size Guides, Supplier, Admin Product, Product Image, Product Option, Product Variant and Notification controllers. Brand/Category create/update/delete now require staff JWT and matching permissions; GET catalog endpoints intentionally remain public for storefront use.
- Secured all Purchase and Purchase Return list/detail/mutation endpoints with staff JWT and the matching PURCHASE_* permissions.
- Secured staff logout, logout-all and change-password routes with UserJwtAuthGuard. Customer auth already had guards on authenticated customer operations.
- Permissions role path params now use ParseEnumPipe; attempts to mutate the SUPER_ADMIN role permission set are rejected.
- Verification: Prisma format/validate/generate passed; backend npm run build passed; Admin npm run build passed; backend and Admin git diff --check passed. No tests or live HTTP authorization probes have been run yet.
- New Prisma migrations have NOT been applied; role grants require applying migrations in timestamp order. No seed or database command was run.
- Next: audit payment/refund boundaries, product/catalog service ownership and remaining Admin permission coverage; then address purchase inventory modeling with an explicit design.
### 2026-11-19 — Full Admin consistency/security audit (batch 4: purchasing and purchase-return invariants)
- Purchase creation now rejects empty line items, negative unit prices, negative discount/tax and negative totals. Existing duplicate-product, supplier and product-reference checks remain.
- Purchase updates now use a conditional PENDING-only updateMany inside the existing transaction, so a purchase that changes state concurrently cannot be updated after leaving PENDING. Cancellation now also performs a conditional PENDING-only update to prevent stale-read state overwrites.
- Purchase-return approval now performs its cumulative approved/completed quantity check and status transition in a Serializable transaction to prevent concurrent approvals from oversubscribing a purchase item's returnable quantity.
- Verification: backend production build passed; backend Jest suite passed 14 tests across 7 suites. No database migration was needed for this batch.
- Critical pre-existing workflow blocker discovered: ReceivePurchaseService intentionally always throws because PurchaseItem references Product, not ProductVariant, and cannot safely increment variant inventory. Consequently purchases cannot reach RECEIVED through the normal API, so CreatePurchaseReturnService's RECEIVED-only prerequisite makes purchase-return creation unreachable in the current workflow. Purchase-return completion also lacks a safe variant-level stock adjustment. This requires a product/inventory data-model decision before end-to-end purchase returns can be declared complete; no unsafe stock behavior was invented.
- Payment audit: customer idempotency responses now verify order ownership; customer IDs are no longer written into the staff User foreign key. Manual settlement is CASH-only and conditional from PENDING; refunds link to original payments and use Serializable balance checks. Backend Jest: 17 tests passing. Gateway session/callback verification and refund settlement remain deferred pending verified provider workflows.
### 2026-11-19 — Full Admin consistency/security audit (batch 6: product inventory boundaries and product-page permissions)
- Removed stock mutation from product variant create/update DTOs, services and Admin variant editor. New variants start at the Prisma default of zero stock; stock quantities are changed only through the STOCK_UPDATE-protected Stock module, closing a PRODUCT_UPDATE-to-stock permission bypass.
- Stock adjustment now uses an atomic increment with a stock floor inside a Serializable transaction, rejects zero adjustments, returns a conflict for serialization races and records the acting staff user with before/after quantities.
- Product variant and product deletion now reject removal while any variant has positive stock; inventory must be reduced to zero through the Stock module before deletion, preventing hidden/trapped stock. Product option variant generation is capped at 500 combinations to prevent unbounded record creation from a single request.
- Added permission-aware read/create/update gates and query skipping for Products list/details/create/update pages. Removed broken links from product rows to category/brand/size-guide detail routes that do not exist; these cells now render as plain labels.
- Added stock adjustment tests. Backend Jest now passes 21 tests across 9 suites; backend production build and Admin production build both pass.

### 2026-11-19 — Full Admin consistency/security audit (batch 7: catalog audit attribution and route correctness)
- Product create/update/delete/restore, product-image add/update/delete, product-option create/update/delete, product-variant create/update/delete and generated-variant actions now record the authenticated staff actor in Activity Logs.
- Product image update requires imageUrl and Cloudinary publicId to be supplied together, preventing mismatched asset references. Image update/delete lookups are constrained to non-deleted parent products.
- Removed ParseUUIDPipe from Product Image IDs because the resource IDs are not UUIDs; the prior pipe rejected valid persisted identifiers.
- Product create/update now reject soft-deleted category and brand references. Product and variant deletion reject positive stock; stock must first be reduced to zero through the dedicated Stock module, avoiding inaccessible inventory.
- Admin Products list/details/create/update and Purchase Create/Permissions pages now show permission-denied states and skip protected queries when the user lacks the required permission. Removed the non-existent Storefront product link from Admin details until Storefront routes exist.
- Verification: backend production build passed; backend Jest 21/21 passed across 9 suites. Admin production build had passed after product permission gating; re-run once final Admin permission changes are complete. No migrations or database operations were run.
- Next: continue with category/brand/supplier/customer service invariants, navigation/route coverage and remaining action-level permission gaps.
- Next: audit payment/refund boundaries, product/catalog service ownership and remaining Admin permission coverage; then address purchase inventory modeling with an explicit design.

### 2026-11-19 — Full Admin consistency/security audit (batch 3: staff account privilege boundaries)
- Staff account creation now allows ADMIN-role creation only by SUPER_ADMIN. ADMIN users can create MANAGER/EMPLOYEE accounts through the standard and employee workflows.
- Staff account updates now prevent non-SUPER_ADMIN users from editing ADMIN accounts or assigning the ADMIN role; self-role changes are rejected. SUPER_ADMIN records remain immutable through the normal user update service.
- Staff account reads no longer expose SUPER_ADMIN details to lower roles, and the staff list excludes SUPER_ADMIN rows for non-SUPER_ADMIN requesters.
- Staff deletion now prevents self-deletion and prevents non-SUPER_ADMIN users from deleting ADMIN accounts. Both soft and permanent deletion paths record the acting user; soft deletion also revokes all active refresh sessions.
- Staff status changes revoke active refresh sessions when deactivating/suspending accounts. The status DTO now permits only ACTIVE, INACTIVE and SUSPENDED; deletion must use the dedicated delete workflow.
- Staff refresh-token rotation now reloads the active account and uses current database identity/role rather than stale refresh-token claims. Password change/reset already revoke active refresh tokens; JWT strategies also reject inactive accounts.
- Admin UI role choices now restrict ADMIN creation/assignment to SUPER_ADMIN; user actions hide status/restore actions from non-SUPER_ADMIN users and protect privileged account rows. Create/update/delete/read actions are permission-aware.
- Added focused staff-account policy tests. Backend Jest now passes 14 tests across 7 suites; backend build passes. Admin production build passed before this last backend-only batch; re-run final Admin build after remaining UI edits.
- No migrations applied, no seed/database mutation run, and no commits/pushes created.
- Next: continue auditing purchase/return state invariants, payment/refund boundaries, product/catalog service ownership and remaining UI permissions; add tests for account session revocation and critical transitions.

### 2026-11-19 — Full Admin consistency/security audit (batch 2: account validation and permission-aware Admin UI)
- Staff JWT strategy now reloads the current staff record on every authenticated request, rejects missing/inactive/suspended/deleted accounts, and uses the current database role/name/email instead of trusting stale role claims.
- Customer JWT strategy now reloads the current customer record and rejects missing/inactive/suspended/deleted accounts. CurrentCustomer decorator now reads Passport's request.user payload; it previously read request.customer, which the guard/strategy never populated.
- Added focused tests for both JWT strategies and PermissionsGuard. Backend Jest suite now passes 10 tests across 4 suites; backend build passes.
- Added a reusable PermissionDenied state and permission-aware read/create/action gating for Brands, Categories, Customers, Users, Employees, Size Guides, Dashboard and Analytics. Restricted pages skip their protected queries when the user lacks read permission. Brand/category catalog GET endpoints remain public for storefront use, while write actions are hidden unless authorized.
- User actions now hide update/status/delete/restore operations based on USER_* grants; customer actions follow CUSTOMER_* grants; size guide actions follow PRODUCT_* grants. Role permission editor hides update actions unless PERMISSION_UPDATE is present and never offers editing SUPER_ADMIN.
- Verification: Admin production build passed; backend Jest 10/10 passed and backend production build passed. Both app git diff --check checks passed after the batch.
- No migrations applied, no seed or database mutation run, and no commits/pushes created.
- Next: finish category/brand/supplier/customer service invariants, remaining navigation/action permission checks and a full Admin route/API consistency sweep.

### 2026-11-19 — Full Admin consistency/security audit (batch 8: customer, category, brand and supplier lifecycle)
- Category, Brand, Supplier and Customer create/update/delete/restore mutations now attribute the authenticated staff actor in Activity Logs. Customer updates now record before/after profile data; customer soft and hard deletes are attributed.
- Customer list filtering now supports DELETED customers using deletedAt, so the existing restore action can actually find deleted records. Customer detail lookup can retrieve soft-deleted records for authorized Admin review.
- Customer soft deletion revokes active customer refresh sessions and removes outstanding password-reset/email-verification tokens. Customer email changes invalidate existing verification and password-reset tokens atomically.
- Customer, Brand and User avatar/image updates now require the URL and Cloudinary public ID to be supplied or removed together. Replaced customer/brand/user assets are deleted only after the database update and activity record succeed.
- Verification: backend build passed; backend Jest 26/26 passed across 11 suites. Admin build remains passing from the previous UI batch. No migrations or database operations were run.

### 2026-11-19 — Full Admin consistency/security audit (batch 9: staff/customer authentication lifecycle)
- Admin-created staff accounts now receive a password-setup invitation link backed by a one-time password-reset token. The welcome email clearly explains setup and expiry; completing setup sets isVerified=true, so newly created employees can actually sign in.
- Staff/customer forgot-password flows no longer reveal whether an email exists; only active accounts receive reset messages. Previous unused reset tokens are invalidated when issuing a new token.
- Staff/customer password-reset and email-verification tokens are consumed conditionally and transactionally, preventing concurrent replay. Reset flows require active accounts and revoke all existing refresh sessions; staff reset also verifies the email and clears lockout counters.
- Staff/customer email-verification flows reject inactive/deleted accounts and consume tokens once. Staff/customer account email changes invalidate prior verification/reset tokens.
- Staff and customer refresh-token consumption is now atomic, preventing two concurrent requests from reusing the same refresh token. Fixed CustomerRefreshTokenService's reversed bcrypt compare arguments, which previously made customer logout token validation fail.
- Settings General and Payment Gateway pages now gate reads by SETTING_READ, skip protected queries without permission, and disable/hide mutations unless SETTING_UPDATE is granted. Payment Gateway cards, toggles, empty-state add actions and dialogs are read-only for users without update permission.
- Admin production build passed after these settings permission changes.
- Staff refresh rotation now consumes the old session and creates the replacement in one Serializable transaction after rechecking active/verified account state. Initial staff/customer login token persistence also rechecks the current password hash inside a Serializable transaction, preventing a login verified against a stale password from creating a session after password reset. UserJwtStrategy now rejects unverified staff sessions.
- Backend Jest remains 26/26 across 11 suites; backend production build passed.

### 2026-11-19 — Full Admin consistency/security audit (batch 12: role-permission escalation boundary)
- Role permission assignment is now SUPER_ADMIN-only at the service boundary; a non-SUPER_ADMIN actor receives Forbidden even if they hold PERMISSION_UPDATE. The Admin edit action is shown only to SUPER_ADMIN, while role permission viewing remains available to PERMISSION_READ users.
- `/permissions/me` now requires staff authentication but not PERMISSION_READ, so every authenticated staff account can load its own effective permissions without needing the permission-management read privilege. Role listing/details remain PERMISSION_READ-protected.
- Added tests for non-SUPER_ADMIN rejection and SUPER_ADMIN updates. Backend Jest: 28/28 across 12 suites; backend production build and Admin production build pass.

### 2026-11-19 — Full Admin consistency/security audit (batch 13: transactional workflow audit trails)
- CreateActivityLogService now accepts an optional Prisma transaction client, allowing critical state changes and their audit records to commit or roll back together.
- Order status transitions now use conditional Serializable transactions and record the acting staff user. Order cancellation atomically changes status, restores tracked variant stock and writes its audit record.
- Purchase update/cancel/delete and purchase-return create/approve/reject/complete/delete now record the acting staff user; status changes and delete audit records are transactional. Payment status changes and refund creation also write audit records inside their Serializable transactions.
- Product deletion now requires zero stock and no cart/order references; variant deletion requires zero stock and no cart/order references. Product options cannot be deleted while variants use them, and option type cannot change while variants depend on it. These rules protect later cancellation/return stock restoration and historical variant attributes.
- Purchase receipt remains intentionally blocked because PurchaseItem has no product-variant reference; do not claim purchase returns change inventory until variant-linked receiving is implemented.
- Verification: backend build passed; backend Jest 28/28 across 12 suites. No migrations or database operations were run.

### 2026-11-19 — Full Admin consistency/security audit (batch 14: payment/order settlement consistency)
- Paid orders can no longer be cancelled while refund settlement is unsupported; the Admin cancellation action is hidden for PAID orders and the backend independently rejects cancellation if either order.paymentStatus or a paid payment transaction indicates settlement.
- Manual payment status updates now prevent multiple PAID payment transactions for one order and recalculate order.paymentStatus across all PAYMENT transactions, so failing/cancelling another pending attempt cannot overwrite an existing PAID state.
- Added tests for duplicate paid transactions and preserving paid order state. Backend Jest: 30/30 across 12 suites; backend and Admin production builds pass. No migrations or database operations were run.
- Staff login now uses a generic invalid-credentials response for unknown emails. Staff forgot-password also uses a generic response for unknown/inactive accounts.
- Added invitation and refresh-token consumption tests. Backend Jest: 26/26 passed across 11 suites; backend production build passed. No database migrations or live mail delivery were run.

### 2026-11-19 — Full Admin consistency/security audit (batch 15: remaining actor-attributed settings and size-guide mutations)
- Completed actor-attributed audit logging for General Settings, Notification Preferences, Security Settings, Payment Gateway and Shipping Settings. Shipping settings and zone create/update/delete now persist audit entries in the same Prisma transaction as each mutation.
- Payment Gateway create/update now persist safe metadata audit entries in the same Prisma transaction as the mutation. Gateway credential-like settings are intentionally excluded from audit values.
- Size Guide create/update/delete now couple mutation and actor-attributed audit entry in one Prisma transaction; deletion still rejects guides assigned to products.
- Discount deletion is transactional. Banner deletion now commits the database delete and actor audit record together before attempting irreversible Cloudinary cleanup, avoiding remote asset removal before a failed database delete.
- Verification: backend `npm run build` passed; backend `npm test -- --run` passed 29 tests across 11 files; Admin `npm run build` passed with 43 generated pages; backend `git diff --check` passed. No migrations, seed, database mutation, commit or push was run.
- Remaining audit work: authenticated role-based Settings E2E against a migrated test database is not yet verified; unauthenticated runtime enforcement and static permission coverage are verified. Purchase receiving still needs a product-variant data-model decision; payment settlement/refund still needs verified provider adapters and callbacks.

### 2026-11-19 — Full Admin consistency/security audit (batch 16: permission and mutation audit closure)
- Added transactional actor audit records for Discount create/update (delete was already transactional), Banner create/update (delete was already transactional), Category create/update/delete, Brand create/update/delete, Supplier create/update/delete, and Customer create/update/delete/restore. External Cloudinary cleanup for Banner, Brand and Customer deletes/asset replacement now runs after the database transaction commits.
- Secured Cloudinary upload-signature generation with staff JWT authentication and a folder-specific authorization guard. Products, categories, brands, banners, staff, customers and settings folders require the matching create/update permission; unsupported folders such as reports/documents are denied. Signature responses are marked no-store/private.
- Fixed Shipping zone whitespace normalization. Fixed sidebar parent visibility so users with a permitted child action can see its parent group; added a dedicated Create Purchase navigation entry.
- Added focused tests for Settings/Size Guide audit transactions, Discount create/update audit transactions and Cloudinary folder authorization. Backend verification: production build PASS; Vitest 14 files / 43 tests PASS; ESLint on all changed backend files PASS. Admin verification: production build PASS (43 routes), TypeScript typecheck PASS; ESLint on changed Admin files PASS. Full Admin lint now runs after explicitly setting React 19.2.8 in ESLint config, but reports 20 existing errors and 61 warnings elsewhere in the Admin codebase; those unrelated lint issues were not mass-modified.
- Runtime unauthenticated checks: Admin direct navigation to /dashboard/settings/general redirects to login; General Settings, Payment Gateway, Shipping, Notification Preferences, Security and Cloudinary upload-signature APIs all return 401 without staff authentication. Authenticated role-by-role Settings E2E remains unverified because no test login/session was supplied and no database migration or data change was authorized.
- No migrations, seed, DB writes, commit or push were run. Existing root .pnpm-store deletions were preserved.

### 2026-11-19 — Prisma generated client location
- Updated `prisma/schema.prisma` so Prisma 7 generates the client into `src/infrastructure/database/generated/prisma` rather than `src/lib/prisma`.
- Regenerated Prisma Client with `npx prisma generate`; this changes generated source files only and does not run migrations or write to the database.
- Kept a temporary `src/lib/prisma` symlink to the new generated-client directory so existing relative imports and Prisma seed imports remain compatible pending a dedicated import-normalization pass.
- Updated `.gitignore` for the new generated output path and updated the architecture map.
- Verification: backend production build PASS; Vitest 14 files / 44 tests PASS. No migrations, seed, database writes, commits or pushes.

### 2026-11-19 — Backend infrastructure cleanup and final verification
- Normalized Prisma generated-client imports throughout backend source, tests and seeds to `src/infrastructure/database/generated/prisma`; removed `src/lib/` and all `src/common/prisma/` references.
- Infrastructure ownership is now explicit: database/Prisma under `src/infrastructure/database`, email delivery under `src/infrastructure/messaging/email`, Cloudinary under `src/infrastructure/storage/cloudinary`, and provider-specific payment adapters under `src/infrastructure/payment-gateways`.
- The remaining 88-file working diff consisted of formatting/import-layout and newline normalization from the infrastructure migration; no schema or migration changes were introduced in this final cleanup batch.
- Verification: backend build PASS; Vitest 14 files / 44 tests PASS; ESLint PASS; `git diff --check` PASS. No migrations, seeds, database writes or provider calls were run.

### 2026-11-20 — Admin Profile page
- Added `/dashboard/profile` to the Admin app; the existing sidebar profile dropdown already linked to this route, so no navigation redesign was required.
- Profile page includes editable name, phone and avatar; read-only email, role and verification state; account creation/last-login details; and a change-password form.
- Added staff-authenticated `PATCH /auth/user/profile`, which only updates the authenticated user's own name, phone and paired avatar/publicId fields. The update and actor-attributed Activity Log are committed in one Prisma transaction; replaced/removed Cloudinary assets are deleted after the DB commit.
- Extended `GET /auth/user/me` with `publicId` and `createdAt`. Profile RTK Query mutation invalidates the Auth tag and updates the Redux user state.
- Scoped signed uploads in the `users` Cloudinary folder to the uploading staff user's folder. Every authenticated staff user may upload their own avatar; the profile endpoint rejects a newly assigned public ID belonging to another staff user's folder. Existing unchanged avatar IDs remain valid.
- Added four focused Vitest cases for empty updates, paired avatar fields, transactional update/audit behavior and cross-user avatar rejection.
- Verification: backend build PASS; backend Vitest 17 files / 53 tests PASS; changed backend ESLint PASS. Admin typecheck PASS; Admin production build PASS (includes `/dashboard/profile`); changed Admin ESLint PASS; both app `git diff --check` PASS. No migrations, database writes, commits or pushes.


### 2026-11-20 — Storefront requirements and architecture decisions
- Created `storefront/PROJECT_REQUIREMENTS.md` as the persistent Storefront requirements/architecture baseline for future sessions.
- Confirmed API/state architecture: Next.js Server Components + Native Fetch for server-side/SEO-critical data; TanStack Query for client-side server state; Zustand only for client-owned UI state. Explicitly ruled out Redux Toolkit/RTK Query unless the user changes the decision.
- Documented SEO, Open Graph/social sharing, JSON-LD, sitemap/robots, Meta Pixel + server-side CAPI, GA4/GTM, Google Ads conversion tracking, optional TikTok Pixel, consent, event deduplication, secure secret handling and per-store tracking isolation.
- Documented multi-store tenant isolation, independent Light/Dark/System theme, responsive/accessibility/performance requirements and a phased implementation plan.
- Updated `SESSION_HANDOFF.md` and the current milestone section here so a new session starts with Storefront requirements. Admin's authenticated role-by-role Settings E2E remains a known unverified gap.
- No dependencies installed, application source changed, build/tests run, commit created or push performed.
- Next: inspect Storefront app status and existing Next.js 16.4 docs, verify shadcn preset setup, then implement the foundation/provider layer in a separate task.


### 2026-11-20 — Storefront home-page foundation
- Added the public route-group layout at `storefront/app/(public)/layout.tsx` and moved the homepage route into `storefront/app/(public)/page.tsx`; removed the duplicate root `app/page.tsx` placeholder so `/` has one route owner.
- Added the initial feature-first home module under `storefront/features/home/components`, with a server-rendered page composition and a small Motion-powered hero client component that respects reduced-motion preferences.
- Implemented shared storefront header/footer components and connected the public layout to them. Their `Your Store` branding is temporary until tenant/store configuration is integrated.
- Implemented the TanStack Query v5 QueryProvider and integrated it into the root layout. Motion was installed by the user; package manifest and lockfile changes from that install were preserved.
- Kept protected files `storefront/package.json`, `storefront/providers/theme-provider.tsx` and `storefront/app/globals.css` untouched by the assistant. The theme provider still defaults to `system`; changing it to the approved Light-first default requires explicit permission to edit the protected provider.
- Verification: ESLint on changed TS/TSX files PASS; Prettier on changed TS/TSX files PASS; `git diff --check` PASS. `pnpm typecheck` remains blocked by pre-existing empty route `page.tsx` files across account/auth/public routes, which generate Next.js route-validator TypeScript errors; unrelated placeholder routes were not modified.
- No commit or push created. Next: inspect backend public store/catalog contracts, then replace temporary home copy/branding and static fallback metadata with tenant-aware server-fetched store data before building dynamic home sections.


### 2026-11-20 — Global component documentation rule
- Added a global working-agreement rule: every React/UI component created or modified across CommerceOS must include clear, concise explanatory comments describing its purpose, responsibility and non-obvious behavior/important logic.
- Comments must remain accurate and useful; avoid redundant comments that only restate self-evident code.
- This is a project-wide coding convention for Backend, Admin and Storefront component work wherever applicable.


### 2026-11-20 — Backend + Admin Wishlist module
- Added a persistent one-wishlist-per-customer data model (Wishlist + WishlistItem) with cascading customer/wishlist ownership, restrictive product references and a unique (wishlistId, productId) constraint to prevent duplicates.
- Added customer-authenticated GET /wishlist, POST /wishlist/items, DELETE /wishlist/items/:productId and DELETE /wishlist endpoints. Customer identity is always derived from CurrentCustomer; add operations accept only published, non-deleted products and use a unique-key upsert for idempotency.
- Added staff-only GET /admin/wishlists, guarded by the new WISHLIST_READ permission, with pagination and search by customer name/email/phone or product name. Admin view is intentionally read-only.
- Added Prisma migrations 20261120220000_add_wishlists and 20261120220100_grant_wishlist_permission. The enum change and permission grant are separate migrations to avoid PostgreSQL enum transaction ordering issues. Migrations have NOT been applied to any database.
- Added Admin RTK Query API, typed contracts, permission-gated /dashboard/wishlists page, Sales navigation item and responsive read-only wishlist details dialog.
- Verification: Prisma format, validate and generate passed; backend pnpm build passed; backend pnpm test passed (18 files / 57 tests, including 2 Wishlist add-service tests); Admin pnpm typecheck and pnpm build passed; git diff --check passed in both app repos. Backend pnpm exec tsc --noEmit still reports an unrelated pre-existing type error in src/modules/settings/services/settings-audit.spec.ts (emailOrders is not a member of UpdateNotificationPreferencesDto); the Nest production build succeeds.
- No migration was applied, no database was modified, and no commit or push was created. Next: connect Storefront wishlist UI to these customer API contracts after the backend migration is applied.


### 2026-11-21 — Storefront Banner feature
- Added a dedicated `storefront/features/banner/` feature with typed campaign data, three homepage campaign slides, editorial image backdrops, CTA links, slide pagination, previous/next controls, swipe/keyboard navigation and six-second autoplay.
- Autoplay pauses while the carousel is hovered or focused and respects reduced-motion preferences. The first campaign heading is the homepage H1; subsequent slide headings use H2.
- Integrated the Banner feature into the existing public homepage composition. Existing HomeHero source remains untouched but is no longer rendered by the homepage.
- Campaign content and Unsplash image URLs are temporary static demo data; no new backend endpoint or admin integration was assumed. Existing user edits in Storefront header and cart sheet were preserved.
- Verification: targeted ESLint PASS; Prettier check PASS; `git diff --check` PASS. Storefront typecheck and production build both remain blocked by pre-existing empty route page files (e.g. account/addresses and other auth/account/public placeholders); Next.js compilation itself succeeded before TypeScript validation stopped the build.
- No package manifest/protected file changes, dependency installs, commits, pushes or backend/database changes.


### Banner visual correction — 2026-11-21
- Removed hard-coded slate palette classes from the Banner feature. Interactive UI now uses the Storefront shadcn semantic tokens (`primary`, `background`, `ring`, `muted`) so its controls follow the configured CSS preset. White copy and a neutral dark overlay remain only for legibility over campaign photography.
- Replaced custom white CTA/outline button styling with the initialized shadcn Button default variant, which inherits the configured primary and primary-foreground colors.
- Reduced the Banner width to the existing storefront content width and adjusted its responsive height.
- Re-ran targeted ESLint and Prettier checks successfully. Production build still compiles Next.js successfully but is blocked during TypeScript validation by the existing empty route page files listed above.


### 2026-11-21 — Storefront Banner backend integration and ratio layout
- Added a public read-only `GET /api/v1/banners/storefront` feed. It returns only active `HOME_TOP` banners whose start/end schedule currently allows display, ordered by `sortOrder` then newest creation; internal actor details and Cloudinary IDs are excluded from the public DTO. Existing staff CRUD/list routes retain their authentication and permission guards.
- Replaced the three static demo slides with server-side Native Fetch data from the new backend feed. Storefront reads the server-only `API_URL` environment variable and falls back to the local development API base `http://localhost:5000/api/v1`; failed/unavailable banner requests degrade to an empty banner area rather than breaking the homepage. Responses are revalidated every 60 seconds.
- Storefront uses the backend's single `imageUrl` field; deprecated `mobileImageUrl` is not used. The same image is rendered with `cover` cropping and responsive aspect ratios (4:3 mobile, 16:9 tablet/desktop); no fixed pixel height is applied. Optional title, link, button text and new-tab behavior come from backend data.
- Updated the Banner carousel to receive server-fetched slides while retaining autoplay, pause-on-hover/focus, reduced-motion support and controls. Existing user edits in Storefront header/cart sheet were preserved.
- Verification: backend `pnpm build` PASS; Storefront targeted ESLint PASS; Prettier write applied to two files; `git diff --check` PASS in backend and Storefront. Storefront `pnpm typecheck` remains blocked by pre-existing empty route page modules across account/auth/public routes; no Banner-specific TypeScript error appeared in the output. No migrations or database changes were needed.
- No commit or push created. Storefront `API_URL` should be configured to the backend API base in non-local environments.

### 2026-11-23 — Storefront New Arrivals vertical slice
- Added the New Arrivals endpoint to the existing public NestJS `StoreProductsController` at `GET /store/products/new-arrivals`, a validated limit DTO, and a standalone `GetStoreNewArrivalsService` registered in ProductModule. It returns published, non-deleted, non-featured products ordered newest-first; excluding featured products prevents duplication with the Featured Products homepage section.
- Extracted shared storefront product/discount response mapping so the existing store product listing and New Arrivals service return consistent effective prices and response fields.
- Added the server-fetched Storefront New Arrivals API function and homepage section after the Promotional Banner. It reuses the existing ProductGrid/ProductCard and fetches in parallel with the other homepage sections; no client-only fetch or migration is required.
- Verification: backend `pnpm build` PASS; backend Vitest 18 files / 57 tests PASS; changed backend ESLint PASS; changed Storefront ESLint PASS; backend `git diff --check` PASS. Storefront `pnpm typecheck` remains blocked by pre-existing zero-byte/non-module account, auth and public route page files under `.next/dev/types/validator.ts`; no errors from the changed homepage files were reported.
- No migrations, seed execution, commits or pushes. Next homepage section in the approved order: Best Sellers.


### 2026-11-25 — Storefront wishlist live refresh
- Fixed the Storefront product wishlist toggle so a successful add/remove invalidates the separate `['wishlist-items']` React Query cache used by the wishlist page and header sheet. The list and item count now refetch after the mutation instead of remaining stale until a browser refresh; the existing `['customer-wishlist']` cache and Zustand guest IDs continue to update as before.
- Changed only `storefront/features/wishlist/hooks/use-product-wishlist.ts`; no backend, database, migration, or dependency changes.
- Verification: targeted ESLint PASS; Prettier check PASS; `git diff --check` PASS. No commit or push.


### 2026-11-25 — Storefront breadcrumbs across route groups
- Added a reusable pathname-aware Storefront breadcrumb component using the existing shadcn Breadcrumb primitives. It renders Home plus linked ancestor segments and a non-linked current page, with readable labels for product, order and return detail routes; the homepage intentionally hides breadcrumbs.
- Integrated it into the public layout and added lightweight account/auth layouts so breadcrumb navigation appears at the top of all non-home Storefront routes across those route groups.
- Changed/added: `storefront/components/layout/store-breadcrumbs.tsx`, `storefront/app/(public)/layout.tsx`, `storefront/app/(account)/layout.tsx`, `storefront/app/(auth)/layout.tsx`.
- Verification: targeted ESLint PASS; Prettier check PASS; Storefront `pnpm typecheck` PASS; `git diff --check` PASS. Existing unrelated working-tree changes were preserved. No commit or push.

### 2026-11-25 — Storefront Products listing and filter drawer
- Replaced the /products placeholder with a server-rendered catalog page using the existing public product, category and brand APIs. Added pagination, product search, sorting (newest/oldest, price ascending/descending, name A–Z/Z–A), and a left-side shadcn Sheet filter drawer.
- Filter drawer supports category, brand, minimum/maximum price, in-stock-only and featured-only filters. Applied search/filter/sort state is encoded in URL query parameters so filtered catalog views can be shared and refreshed.
- Extended the public backend product query contract with price bounds, stock availability and sort field/direction; category, brand, search and featured filtering reuse existing query support. Public products remain restricted to published, non-deleted records.
- Added a small Storefront brand API adapter for active filter options. Existing category product fetching now reuses the shared product listing API function.
- Verification: Storefront targeted ESLint PASS; Storefront typecheck PASS; Storefront production build PASS (31 static/dynamic routes); backend build PASS; backend Vitest 18 files / 57 tests PASS; backend changed-file ESLint PASS; git diff --check PASS in both repositories.
- Price bounds currently query the product's stored sellingPrice field; stock filtering checks for at least one variant with positive stock. No migrations, database writes, commits or pushes.
### 2026-11-25 — Refined Products filter Sheet
- Updated only the storefront ProductFiltersSheet to use shadcn Accordion, Slider and ScrollArea components.
- Category accordion is open by default; Brand starts closed. Availability/highlights and Price are collapsible groups; Price is placed last and uses a two-thumb range slider (৳0–৳100,000, step ৳100) instead of number inputs.
- Removed the custom SheetContent width override so the left Sheet uses the shadcn default width. Kept the filter body scrollable and the Reset/Apply footer fixed.
- Verification: targeted ESLint PASS; Storefront typecheck PASS; Storefront production build PASS (31 routes); git diff --check PASS.

### 2026-11-25 — Products filter drawer uses available height
- Updated `storefront/features/product/components/product-listing/product-filters-sheet.tsx` so the category/brand ScrollArea sits in a `flex min-h-0 flex-1 flex-col` wrapper and expands to all remaining vertical space. Removed the fixed `max-h-60` limits and nested list scrolling; the shared ScrollArea now handles overflow. Availability/price remain in the fixed bottom area and the action footer remains fixed.
- Verification: targeted ESLint PASS; Storefront `pnpm typecheck` PASS; Prettier PASS; `git diff --check` PASS. No commit or push.
