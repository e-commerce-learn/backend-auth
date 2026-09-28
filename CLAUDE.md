# backend-auth — Claude Instructions

> Read the global rules first: [`architecture/CLAUDE.md`](../../architecture/CLAUDE.md) (local path) /
> [GitHub](https://github.com/e-commerce-learn/architecture/blob/main/CLAUDE.md). This file only adds what is
> specific to this repo and can't contradict the global one. Setup and env vars: [`README.md`](README.md).

## What this service owns

Credentials, login, registration, issuing JWTs. User accounts/profiles belong to `backend-users`
([ADR 0012](https://github.com/e-commerce-learn/architecture/blob/main/adr/0012-auth-and-users-separate-services.md)).
JWTs are issued here but verified only at the gateway.

## Database layer

| File | Role |
|---|---|
| `src/database/database.module.ts` | `@Global()` module; builds the `pg` `Pool` from `DB_*` env vars via `ConfigService`, provided under the `PG_POOL` token |
| `src/database/database.service.ts` | `query<T>(text, params)` wrapper; `SELECT 1` on startup (logs `Connected to auth_db`), closes the pool on shutdown |
| `src/database/database.constants.ts` | `PG_POOL` injection token |

- **The user writes the DB layer** — schemas, migrations, `DatabaseModule`/`DatabaseService` changes. Don't
  pre-write it unless asked ([ADR 0002](https://github.com/e-commerce-learn/architecture/blob/main/adr/0002-reset-sqlite-to-postgres.md)).
- Raw SQL with parameters (`$1`, `$2`) only — no ORM, no query builder.
- Connect only as `auth_service` to `auth_db`; never put the Postgres superuser in `.env`.

## Gotchas

- The NestJS project name stays `auth` (`package.json`) — the repo name is `backend-auth`.
- `auth_service` / `auth_db` aren't recreated if the Postgres volume is deleted (until PLAT-3 init scripts).
- `package.json` scripts still reference a `test/` folder that doesn't exist (`test:e2e` fails) — left over from
  the Nest scaffold.
