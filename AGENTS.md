# AI Agent Instructions — CommerceOS

Before project work:
1. Read `PROJECT_CONTEXT.md`.
2. Consult `PROJECT_ARCHITECTURE.md` for subsystem navigation and known gaps.
3. Check `ROADMAP_NEXT.md` for current priorities; follow the user's explicit task if they choose another priority.

## Efficient context loading
- Do not scan or reread the whole monorepo for every task.
- Inspect only the requested feature's controller/route, service, DTO/types, schema, direct imports, related UI/API files and tests.
- Use the architecture map as a guide, not as a substitute for checking current code.
- If a file or status is not covered in the map, inspect it before making assumptions.
- Keep session handoffs concise and update project docs after meaningful discoveries/changes.

## Code structure and conventions
- Preserve the established feature-first Admin structure: `src/features/<feature>/{api,types,schemas,components}` and route entrypoints under `src/app/(dashboard)/dashboard`.
- Preserve the established NestJS structure: `src/modules/<feature>/{controllers,dto/requests,dto/responses,services}`.
- Follow neighboring modules for naming, RTK Query endpoint injection, DTO validation, permission checks, table/dialog patterns and import aliases. Do not introduce a competing architecture or move files broadly without approval.
- Keep Admin and backend changes in their respective app repositories.
- Complete one module at a time, report changed files and verification, then stop for the user's review.
- Do not start another module until the user explicitly asks. Never commit or push unless the user explicitly instructs it.

## Safety and implementation
- Check `git status --short` in the affected app before editing; preserve unrelated user changes.
- Never read, print, copy, commit or document actual `.env*` secret values. Use `.env.example` for variable names and placeholders only.
- Follow existing NestJS/Next.js patterns, DTO validation, response shapes, auth/permission conventions and error handling.
- Backend authorization must be enforced server-side; frontend route protection or cookie presence is not sufficient.
- Do not expose provider credentials or tokens in API responses, logs, settings JSON or docs.
- Do not run destructive commands, database resets, migrations, deployments or production actions without explicit user approval.
- Do not add unrelated integrations (e.g. Calendar) without product requirement.

## Verification
- Run the narrowest relevant tests/build/typecheck after changes.
- Report exact commands and whether they passed. Distinguish a successful build from meaningful test coverage.
- Recheck `git status` and review the diff before finishing.

## Persistent handoff
Update `PROJECT_CONTEXT.md` Task Log with changed files, decisions, verification and next step. Update `PROJECT_ARCHITECTURE.md` only when architecture/findings change, and `ROADMAP_NEXT.md` when priorities change.