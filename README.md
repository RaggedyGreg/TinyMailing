# Educa CRM — Implementation Blueprint

A multi-tenant SaaS CRM, email marketing, automation, and billing platform for kindergartens, schools, academies, childcare centers, and child-focused businesses, initially targeting Argentina.

## Technology baseline

- Frontend: Angular **22.2.x** (latest stable line verified 2026-10-28), TypeScript, standalone components, Signals where appropriate, RxJS for streams.
- Backend: Node.js + NestJS + TypeScript, REST API, OpenAPI.
- Data: PostgreSQL in production; SQLite may be used only for local experiments, not as the default multi-tenant production database.
- ORM: Prisma.
- Background jobs: PostgreSQL-backed `pg-boss` initially, avoiding a separate Redis service. If volume grows, evaluate BullMQ + Redis.
- Email: provider adapter; initial candidates include Amazon SES, Brevo, or Resend. Do not send bulk mail synchronously from API requests.
- Payments: provider adapter; Mercado Pago is the first Argentina-oriented candidate. Keep manual bank transfer/cash recording possible.
- Local orchestration: Docker Compose.
- CI: GitHub Actions.
- UI language: Spanish (Argentina) initially; architecture should allow future localization.
- Currency: ARS as default; use integer minor units for money (centavos) and ISO currency codes.
- Time zone: `America/Argentina/Buenos_Aires` as the default organization time zone.

## Documents

- `docs/PRD.md` — product requirements, user stories, acceptance criteria, and release scope.
- `docs/ARCHITECTURE.md` — modules, boundaries, security, jobs, integrations, and deployment.
- `docs/DATABASE.md` — schema decisions, relationships, and data rules.
- `docs/API.md` — API conventions and endpoint catalog.
- `openapi.yaml` — starter OpenAPI 3.1 contract.
- `prisma/schema.prisma` — initial Prisma schema to refine with migrations.
- `docs/PROJECT-STRUCTURE.md` — monorepo layout and development conventions.
- `.github/copilot-instructions.md` — repository-wide instructions for GitHub Copilot.
- `compose.yaml` and `.env.example` — local infrastructure baseline.

## Quick start

Prerequisites: Node.js version compatible with Angular 22 and NestJS (use a supported Node LTS release accepted by Angular's compatibility table), npm, Docker Desktop, and Git.

1. Copy `.env.example` to `.env`; set local secrets and provider settings.
2. Start PostgreSQL: `docker compose up -d postgres`.
3. Scaffold the API in `apps/api` with NestJS and install Prisma, `pg-boss`, validation, security, and test dependencies.
4. Scaffold the frontend in `apps/web` with Angular CLI 22.
5. Install workspace dependencies and configure npm workspaces.
6. Generate Prisma client and run migrations.
7. Run API and frontend tests before implementing integrations.

This blueprint is implementation-ready as a product and architecture specification, but it is not a finished runnable application. External provider credentials, exact package versions, deployment targets, and legal review must be configured before production.

## Core design decisions

1. Multi-tenant SaaS from the start. Every tenant-owned record carries `organizationId`; tenant context comes from authenticated membership, never a client-supplied organization ID alone.
2. Modular monolith, not microservices.
3. PostgreSQL for production correctness, concurrency, transactions, and background jobs.
4. All email sending and payment-provider calls happen asynchronously or through explicit integration services.
5. Marketing consent, suppression, and unsubscribe are first-class data, not campaign UI options.
6. Children are not marketing recipients by default. Communicate with verified adult guardians and minimize sensitive child data.
7. Payment events must be idempotent; never trust browser redirects as proof of payment.
8. Money is stored as integer minor units plus ISO currency; never use floating-point arithmetic for monetary values.
9. API responses use a stable envelope for errors and cursor/offset pagination conventions documented in `docs/API.md`.
10. No production secrets in source control or frontend environment files.
