# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Postiz is a social media scheduling tool supporting 28+ platforms. Posts are added to a calendar, queued via Temporal workflows, and published at the scheduled time.

## Monorepo Structure

```
apps/
  backend/       # NestJS REST API (port 3000)
  frontend/      # Vite + React (port 4200)
  orchestrator/  # Temporal worker for background jobs (NestJS)
  commands/      # CLI commands (NestJS)
  extension/     # Chrome browser extension
  cli/           # CLI tool for API usage
  sdk/           # Public SDK

libraries/
  nestjs-libraries/   # Shared NestJS: DB (Prisma), integrations, DTOs, services
  helpers/            # Shared utilities and React fetch hooks
  react-shared-libraries/  # Shared React components and i18n translations
```

**Package manager:** pnpm only. Node >= 22.12.0 < 23.

## Commands

```bash
# Development (all services in parallel)
pnpm dev

# Individual services
pnpm dev:backend
pnpm dev:frontend
pnpm dev:orchestrator

# Builds
pnpm build                  # all apps
pnpm build:backend
pnpm build:frontend

# Database
pnpm prisma-generate        # regenerate Prisma client
pnpm prisma-db-push         # push schema changes (accepts data loss)

# Linting (must run from root)
pnpm eslint .

# Tests
pnpm test                   # jest with coverage

# Docker dev environment
pnpm dev:docker             # starts postgres + redis via docker-compose.dev.yaml
```

## Backend Architecture

All backend logic must flow through **3 layers** (no shortcuts):

```
Controller → Service → Repository
Controller → Manager → Service → Repository  (for complex flows)
```

- **Controllers** live in `apps/backend/src/api/routes/` — thin layer, delegates to services
- **Business logic** lives in `libraries/nestjs-libraries/src/` — this is `@gitroom/nestjs-libraries`
- **Repositories** are in `libraries/nestjs-libraries/src/database/prisma/*/` — one per domain
- **Prisma schema** is at `libraries/nestjs-libraries/src/database/prisma/schema.prisma`

The package alias `@gitroom/nestjs-libraries` maps to `libraries/nestjs-libraries`. Backend controllers import from there, not from `apps/`.

## Adding a Social Media Integration

Each platform is a provider class in `libraries/nestjs-libraries/src/integrations/social/`. To add a new one:

1. Create `<platform>.provider.ts` extending `SocialAbstract` and implementing `IAuthenticator`
2. Register it in `libraries/nestjs-libraries/src/integrations/integration.manager.ts`

Key interfaces: `IAuthenticator` (OAuth flow) and `SocialAbstract` (post publishing) in `social.abstract.ts` and `social.integrations.interface.ts`.

## Temporal Workflows (Orchestrator)

Workflows are in `apps/orchestrator/src/workflows/`, activities in `apps/orchestrator/src/activities/`. The main post publishing flow is `post.workflow.v1.0.1.ts`. Temporal handles scheduling, retries, and token refresh.

## Frontend Architecture

- **Routing**: `apps/frontend/src/app/` (Next.js-style file routing via Vite)
- **UI components**: `apps/frontend/src/components/ui/`
- **Feature components**: `apps/frontend/src/components/`
- **Data fetching**: Always use SWR via the `useFetch` hook from `@gitroom/helpers/utils/custom.fetch.tsx`

### SWR Rules

Each SWR call must be in its own named hook. Never inline hooks or return hook-calling functions:

```tsx
// ✅ Correct
const useIntegrations = () => useSWR('/integrations', fetch);

// ❌ Wrong — violates react-hooks/rules-of-hooks
const useData = () => ({
  integrations: () => useSWR('/integrations', fetch),
});
```

### Styling

- Tailwind 3 — config at `apps/frontend/tailwind.config.js`
- CSS variables defined in `apps/frontend/src/app/colors.scss` and `global.scss`
- Use semantic color tokens: `primary`, `secondary`, `third`–`seventh`, `newBgColor`, `btnPrimary`, etc.
- **Do not use** `--color-custom*` variables (deprecated)
- Custom breakpoints: `mobile` (≤1025px), `tablet` (≤1300px), `iconBreak` (≤1560px)
- Do not install npm UI component libraries — write native components

## Environment Variables

Copy `.env.example` to `.env`. Required variables:
- `DATABASE_URL` — PostgreSQL connection string
- `REDIS_URL` — Redis connection string
- `JWT_SECRET`
- `FRONTEND_URL`, `NEXT_PUBLIC_BACKEND_URL`, `BACKEND_INTERNAL_URL`
- `CLOUDFLARE_*` — for media/avatar storage (or set `STORAGE_PROVIDER=local`)

Each social platform requires its own `CLIENT_ID`/`CLIENT_SECRET` pair (see `.env.example`).

## TypeScript Path Aliases

- `@gitroom/nestjs-libraries` → `libraries/nestjs-libraries/src`
- `@gitroom/helpers` → `libraries/helpers/src`
- `@gitroom/react` → `libraries/react-shared-libraries/src`
- `@gitroom/backend` → `apps/backend/src`
