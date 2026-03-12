# CLAUDE.md

## Project

- **Name:** opengame-api
- **Description:** Push notification relay API for the Open Game System (OGS). Game servers send notifications through this API, which looks up device push tokens and dispatches via APNs (iOS) or FCM (Android).
- **Tech stack:** Hono, Cloudflare Workers, D1 (SQLite), TypeScript, Vitest
- **Build:** Node 20+, wrangler v4, npm

## Knowledge Base

Start here. Load deeper docs **only when working on the relevant domain.**

| Topic | Location |
|---|---|
| System architecture | [docs/architecture.md](docs/architecture.md) |
| Lessons learned | [docs/lessons.md](docs/lessons.md) |
| Database schema | [schema.sql](schema.sql) |

> **Progressive disclosure:** Do NOT load all docs upfront. Read this file,
> then load the specific doc relevant to your current task.

## Core Principles

- **Error consistency**: Every error response uses `{ error: { code, message, status } }` format
- **Minimal surface**: Only the endpoints needed for push notification delivery
- **Stub-first providers**: Push providers (APNs, FCM) are stubs until real credentials and integration are wired up
- **Test with mocks**: D1 is mocked in tests via `vi.fn()` chains matching the D1 prepare/bind/run API

## Key Conventions

- **Router pattern**: Each route group is a separate `Hono` instance in `src/routes/`, mounted in `src/index.ts`
- **Middleware**: Auth middleware applied via `app.use()` before the route group it protects (see `index.ts`)
- **D1 queries**: Always use parameterized `prepare().bind().run()`/`.first()` -- never string interpolation
- **D1 upsert**: Use `INSERT ... ON CONFLICT(...) DO UPDATE SET` for idempotent registration
- **Type definitions**: Request/response types and DB row types in `src/types.ts`
- **Push providers**: `src/providers/push.ts` exports a `PushProvider` interface; `getProviderForPlatform()` factory selects APNs or FCM
- **Auth**: Bearer token in `Authorization` header, validated against `api_keys` table; game info set on Hono context variables
- **Error responses**: Always `{ error: { code: string, message: string, status: number } }` with matching HTTP status
- **No ORM**: Raw D1 SQL via Hono's `c.env.DB`

## Build & Test

```bash
# Install dependencies
npm ci

# Run tests (vitest, no Cloudflare bindings needed)
npm test

# Run tests in watch mode
npm run test:watch

# Type check
npm run typecheck

# Local dev server
npm run dev           # wrangler dev

# Deploy to Cloudflare Workers
npm run deploy        # wrangler deploy

# Run D1 migration
npm run db:migrate    # wrangler d1 execute opengame-api-db --file=schema.sql
```

## Git

- **Main branch:** `main`
- **CI:** GitHub Actions runs typecheck + tests on push/PR to main
- **CD:** GitHub Actions deploys to Cloudflare on push to main
- Never use `--no-verify` on git hooks
