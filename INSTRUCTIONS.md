# Setting up event-utils locally

A step-by-step guide to get the full platform running on your machine.

> **Repo status:** we're in the docs phase (v0.1) — the Phase 1 skeleton (apps, packages, compose file) is being scaffolded right now. The commands below describe the Phase 1 layout and will match `main` as it lands. If something doesn't exist yet, check the [roadmap](docs/architecture.md#12-roadmap) or ping the CIG.

---

## Table of contents

1. [Prerequisites](#1-prerequisites)
2. [Clone & install](#2-clone--install)
3. [Environment variables](#3-environment-variables)
4. [Start the stack](#4-start-the-stack)
5. [Database setup](#5-database-setup)
6. [Dev URLs](#6-dev-urls)
7. [Everyday commands](#7-everyday-commands)
8. [Troubleshooting](#8-troubleshooting)
9. [Useful references](#9-useful-references)

---

## 1. Prerequisites

| Tool | Version | Why | Check |
|---|---|---|---|
| **Node.js** | 20 LTS or newer | Runs the web + API apps | `node -v` |
| **pnpm** | 9 or newer | Package manager for the monorepo | `pnpm -v` |
| **Docker Desktop** (or Engine + Compose v2) | recent | Runs Postgres, Mailhog, Caddy; can run the whole stack | `docker compose version` |
| **Git** | any recent | Cloning the repo | `git -v` |

Install pnpm if you don't have it:

```bash
corepack enable
corepack prepare pnpm@latest --activate
```

> **Don't have Docker?** You can run Postgres natively instead — see [Troubleshooting](#8-troubleshooting). Docker is still the recommended path because it mirrors the production VPS setup.

## 2. Clone & install

```bash
git clone https://github.com/recursekmit/event-utils.git
cd event-utils
pnpm install
```

This installs dependencies for all workspaces (`apps/*`, `packages/*`, `modules/*`) in one go.

## 3. Environment variables

Each app keeps its own env file. Start from the committed examples:

```bash
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env
```

Key variables you'll touch (full list is documented inside the example files):

| Variable | Where | Default for local dev | What it is |
|---|---|---|---|
| `DATABASE_URL` | `apps/api/.env` | `postgresql://postgres:postgres@localhost:5432/event_utils` | Postgres connection string |
| `APP_DOMAIN` | `apps/api/.env` | `localhost` | Base domain used for event subdomains (`*.localhost` in dev) |
| `MAILER_*` (host, port, from) | `apps/api/.env` | points at Mailhog | SMTP the mailer interface sends through |
| `NEXT_PUBLIC_API_URL` | `apps/web/.env` | `http://api.localhost:4000` | Where the web app finds the API |

No secrets are needed locally: magic-link emails are captured by **Mailhog** instead of being sent, so you can open any link it catches without a real mailbox.

## 4. Start the stack

**Option A — everything in Docker (closest to production):**

```bash
docker compose up
```

This starts all four services: `web` (Next.js), `api` (NestJS), `postgres`, and `caddy`. Mailhog joins in dev. Caddy handles the wildcard `*.localhost` routing, so you don't manage ports per subdomain. Add `-d` to run it in the background, and `docker compose down` to stop.

**Option B — infra in Docker, apps on your host (faster for development):**

```bash
docker compose up postgres mailhog caddy -d   # infra only
pnpm dev                                      # runs web + api with hot reload
```

## 5. Database setup

Once Postgres is up, push the schema and (optionally) seed demo data:

```bash
pnpm --filter api prisma migrate dev     # apply migrations
pnpm --filter api prisma db seed         # demo org + event + users (if a seed exists)
```

If you're exploring the data model, Prisma Studio is handy:

```bash
pnpm --filter api prisma studio          # opens a DB browser at localhost:5555
```

## 6. Dev URLs

`*.localhost` resolves automatically on modern OSes — no `/etc/hosts` edits needed.

| URL | What you'll see |
|---|---|
| `http://localhost:3000` | Web app (apex: platform landing + event index) |
| `http://myevent.localhost:3000` | A specific event's themed site, e.g. `myevent` from the seed |
| `http://admin.localhost:3000` | Admin shell (organizer dashboard) |
| `http://localhost:4000` | API (NestJS) |
| `http://localhost:8025` | **Mailhog** — every magic-link email lands here; click the link inside to log in |

**First login:** open Mailhog, find the magic-link email, click it — you're in. The first account to claim `superadmin` is typically granted during seeding or via a bootstrap step documented in the seed output.

## 7. Everyday commands

All commands run from the repo root:

```bash
pnpm dev                # run web + api with hot reload (workspaces)
pnpm build              # build all workspaces
pnpm lint               # lint everything
pnpm typecheck          # tsc --noEmit across the monorepo
pnpm test               # run all tests

pnpm --filter web dev   # a single app only, e.g. the web app
pnpm --filter api test  # or the API's tests
```

Module development (once Phase 2 lands): copy `modules/_template` to `modules/<your-module-id>`, edit its manifest, and enable it on an event from the admin shell. The full guide (`CREATE-MODULE.md`) ships with the Phase 3 freeze.

## 8. Troubleshooting

**`pnpm install` fails on Node version** — check `node -v`; you need 20+. Use `nvm install 20 && nvm use 20` if you're on an older one.

**Port already in use (3000/4000/5432/8025)** — something else is listening. Stop it, or run `docker compose down` first; stale containers from another project are the usual culprit.

**Event subdomain doesn't resolve** — make sure you're using `http://<slug>.localhost:3000` (with the port) in dev. Older browsers/OS setups that don't resolve `*.localhost` need a hosts-file entry: `127.0.0.1 myevent.localhost`.

**"Database does not exist" / connection refused** — Postgres isn't up yet. Run `docker compose up postgres -d`, wait a few seconds, then re-run migrations. The first `migrate dev` also creates the database for you.

**No magic-link email arrives** — check Mailhog at `http://localhost:8025`. If Mailhog wasn't running when you requested the link, just request it again; tokens are short-lived by design.

**Docker running out of space** — `docker system prune` (careful: removes stopped containers and unused images).

**Running without Docker at all** — install Postgres locally, create a database named `event_utils` (or match `DATABASE_URL`), and skip the infra containers; you'll lose Mailhog, so point `MAILER_*` at any SMTP you have and read the mailer docs in `apps/api`.

## 9. Useful references

- [docs/architecture.md](docs/architecture.md) — the source of truth: stack rationale, data model, module system, ADR log
- [Feature-request template](.github/ISSUE_TEMPLATE/feature_request.yml) — propose a feature/module with the context maintainers need
- [README](README.md) — what the project is and where it's headed

Questions? Open an issue or ask in the CIG channel — someone has hit the same wall before.
