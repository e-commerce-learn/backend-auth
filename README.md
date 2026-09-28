# backend-auth

Auth service of the [`e-commerce-learn`](https://github.com/e-commerce-learn) org: credentials, login,
registration and issuing JWTs. NestJS 11 + raw SQL on PostgreSQL (no ORM).

User accounts and profiles are **not** here — they belong to `backend-users`
([ADR 0012](https://github.com/e-commerce-learn/architecture/blob/main/adr/0012-auth-and-users-separate-services.md)).
The NestJS project is still named `auth`.

| | |
|---|---|
| Owner team | Identity (Jira space `IDN`) |
| Port | `3000` (override with `PORT`) |
| Database | `auth_db`, role `auth_service` |

## Prerequisites

- Node.js + npm (developed on Node 26)
- Docker, with the shared PostgreSQL 16 running on port `5432`
  - Postgres lives in the `infra-postgres` repo (PLAT-12). Until that's done it runs from the old monorepo copy:
    `docker compose up -d` in `projects/ecommerce/`.
  - The role `auth_service` and the database `auth_db` must already exist. They are **not** created
    automatically yet (init scripts: PLAT-3) — never delete the Postgres volume (`docker compose down -v`).

## Setup

```bash
npm ci
cp .env.example .env    # then set the real DB_PASSWORD
npm run start:dev
```

It works when the log shows:

```
[DatabaseService] Connected to auth_db
```

On startup the service runs `SELECT 1`; if the database can't be reached, the app doesn't start and prints the
error instead.

| Error | Cause |
|---|---|
| `ECONNREFUSED 127.0.0.1:5432` | Postgres isn't running |
| `password authentication failed for user "auth_service"` | Wrong `DB_PASSWORD` in `.env` |
| `database "auth_db" does not exist` | Postgres started on an empty volume — the role and database are missing |

## Environment variables

Set in `.env` (git-ignored; never commit it). `.env.example` has the keys with placeholder values.

| Variable | Example | Meaning |
|---|---|---|
| `DB_HOST` | `localhost` | Postgres host |
| `DB_PORT` | `5432` | Postgres port (default `5432`) |
| `DB_NAME` | `auth_db` | This service's database |
| `DB_USER` | `auth_service` | This service's role — never a superuser |
| `DB_PASSWORD` | `changeme` | Password of `DB_USER` |
| `PORT` | `3000` | HTTP port of the service (optional, default `3000`) |

## Scripts

| Command | What it does |
|---|---|
| `npm run start:dev` | Run with auto-restart on file changes |
| `npm run start` | Run once |
| `npm run build` | Compile to `dist/` |
| `npm run start:prod` | Run the compiled build (`node dist/main`) |
| `npm run lint` | ESLint + Prettier check, auto-fixes what it can |
| `npm run format` | Format `src/` with Prettier |
| `npm test` | Unit tests (Jest) — none written yet |

## Docs

Company-wide rules, ADRs and the service catalog live in the
[`architecture`](https://github.com/e-commerce-learn/architecture) repo.
