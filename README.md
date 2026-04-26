# car-import-calculator

A Yarn 4 monorepo housing the **Auto Market Import Calculator** vehicle-marketplace ecosystem: a NestJS API, a React SPA, and the shared TypeScript contracts that bind them together.

---

## Project Overview

| Workspace               | Package name                          | Description                                                                                                                                      |
| ----------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `apps/api`              | `@car-import-calculator/api`          | NestJS 11 + Prisma 6 backend. Session-cookie auth backed by Redis, PostgreSQL persistence, Swagger docs at `/docs`.                              |
| `apps/web`              | `@car-import-calculator/web`          | Vite + React 18 SPA (Spanish-language UI). Tailwind styling, axios-based API client with `withCredentials`.                                      |
| `packages/shared-types` | `@car-import-calculator/shared-types` | Cross-workspace TypeScript interfaces, enums, and DTO contracts consumed by both apps via Yarn's `workspace:` protocol — never published to npm. |

The root orchestrates the three workspaces, owns the Docker Compose stack, the Yarn 4 configuration, and the CI/CD pipeline.

---

## Tech Stack

| Layer            | Technology                                |
| ---------------- | ----------------------------------------- |
| API framework    | NestJS 11                                 |
| ORM / database   | Prisma 6 · PostgreSQL 16                  |
| Session store    | Redis 7 (express-session + connect-redis) |
| Frontend         | React 18 · Vite · Tailwind CSS            |
| Language         | TypeScript 5                              |
| Package manager  | Yarn 4 Berry (`nodeLinker: node-modules`) |
| Containerization | Docker · Docker Compose                   |
| CI / CD          | GitHub Actions (path-filtered)            |
| Cloud target     | AWS ECR · ECS (Fargate) or Lambda         |

---

## Local Setup — Ubuntu Survival Guide

> Assumes Ubuntu 22.04+ with Node.js 20 (via nvm or the NodeSource repo), Docker, and Docker Compose v2. No global `yarn` install is required — Corepack ships with Node.

### 1. Enable Corepack and activate Yarn 4

Corepack pins the Yarn version declared in `package.json` (`packageManager: "yarn@4.5.0"`), so every contributor runs an identical toolchain.

```bash
corepack enable
corepack prepare yarn@4.5.0 --activate
yarn --version        # expect 4.5.0
```

### 2. Install dependencies

```bash
yarn install
```

This resolves every workspace, hoists shared packages into the root `node_modules`, symlinks the local `@car-import-calculator/*` packages, and runs the API's `postinstall` (`prisma generate`).

### 3. Initialize the Git repository

The repository currently has no `.git` directory. Initialize it and take your first snapshot:

```bash
git init
git add .
git commit -m "chore: initial monorepo import"
```

Commit `yarn.lock` and the `.yarn/releases/` binary so CI installs the exact same Yarn build.

### 4. Run each app individually

Every workspace script can be triggered from the repo root through `yarn workspace <name> <script>`. Shortcuts are also defined at the root.

| Task                    | Root shortcut             | Explicit form                                               |
| ----------------------- | ------------------------- | ----------------------------------------------------------- |
| API dev server (watch)  | `yarn dev:api`            | `yarn workspace @car-import-calculator/api start:dev`       |
| Web dev server (HMR)    | `yarn dev:web`            | `yarn workspace @car-import-calculator/web dev`             |
| API production build    | `yarn build:api`          | `yarn workspace @car-import-calculator/api build`           |
| Web production build    | `yarn build:web`          | `yarn workspace @car-import-calculator/web build`           |
| Shared types type-check | `yarn build:shared-types` | `yarn workspace @car-import-calculator/shared-types build`  |
| Prisma migrate (dev)    | `yarn prisma:migrate`     | `yarn workspace @car-import-calculator/api prisma:migrate`  |
| Prisma generate         | `yarn prisma:generate`    | `yarn workspace @car-import-calculator/api prisma:generate` |
| Lint everything         | `yarn lint`               | `yarn workspaces foreach -Apt run lint`                     |
| Test everything         | `yarn test`               | `yarn workspaces foreach -Apt run test`                     |

Running the API and Web directly on the host requires a reachable Postgres and Redis. The fastest path is to boot only the datastores in Docker and keep Node on the host:

```bash
docker compose up db redis   # terminal 1
yarn dev:api                 # terminal 2
yarn dev:web                 # terminal 3
```

---

## Environment Variables

Secrets never live in the repo. Each file below is git-ignored; copy the templates, fill in real values, and keep them local.

### Root — `./.env` (consumed by Docker Compose)

| Variable            | Purpose                                                                       |
| ------------------- | ----------------------------------------------------------------------------- |
| `APP_PORT`          | Host port published for the API container.                                    |
| `POSTGRES_USER`     | Postgres superuser created in the `db` container.                             |
| `POSTGRES_PASSWORD` | Postgres password.                                                            |
| `POSTGRES_DB`       | Initial database name.                                                        |
| `POSTGRES_PORT`     | Host port published for Postgres.                                             |
| `REDIS_PASSWORD`    | Password enforced via `redis-server --requirepass`.                           |
| `REDIS_PORT`        | Host port published for Redis.                                                |
| `REDIS_URI`         | Full connection URI used by the API container.                                |
| `SESSION_NAME`      | Session cookie name surfaced into the API container as `SESSION_COOKIE_NAME`. |

### API — `apps/api/.env.development.local`

Loaded by `CoreConfigModule`; `ConfigService.getOrThrow` means a missing variable crashes the app at boot.

| Variable                                                                                  | Purpose                                                                                                                                                      |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `APP_PORT`                                                                                | Port Nest binds to.                                                                                                                                          |
| `ALLOWED_ORIGIN`                                                                          | CORS origin (typically the web dev URL).                                                                                                                     |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_HOST` / `POSTGRES_PORT` / `POSTGRES_DB` | Postgres connection components.                                                                                                                              |
| `DATABASE_URL`                                                                            | Prisma connection string, usually interpolated from the Postgres pieces above. **Required by Prisma for `prisma migrate`, `generate`, and runtime queries.** |
| `COOKIE_SECRET`                                                                           | Key used to sign cookies.                                                                                                                                    |
| `SESSION_SECRET`                                                                          | Secret used to sign the session ID.                                                                                                                          |
| `SESSION_NAME`                                                                            | Session cookie name.                                                                                                                                         |
| `SESSION_DOMAIN`                                                                          | Cookie domain.                                                                                                                                               |
| `SESSION_MAX_AGE`                                                                         | Session lifetime in milliseconds.                                                                                                                            |
| `SESSION_HTTP_ONLY` / `SESSION_SECURE` / `SESSION_SAME_SITE`                              | Cookie security flags.                                                                                                                                       |
| `SESSION_FOLDER`                                                                          | Namespace prefix for Redis-backed session keys.                                                                                                              |
| `REDIS_PASSWORD` / `REDIS_HOST` / `REDIS_PORT` / `REDIS_URI`                              | Redis connection.                                                                                                                                            |
| `GOOGLE_CLIENT_ID`                                                                        | Google OAuth2 client ID.                                                                                                                                     |
| `GOOGLE_CLIENT_SECRET`                                                                    | Google OAuth2 client secret.                                                                                                                                 |
| `GOOGLE_CALLBACK_URL`                                                                     | Redirect URI registered with Google.                                                                                                                         |
| `FRONTEND_URL`                                                                            | Post-login redirect target for the OAuth callback.                                                                                                           |

### Web — `apps/web/.env` and `apps/web/.env.local`

Only `VITE_`-prefixed variables are exposed to the browser.

| Variable                     | Purpose                                                   |
| ---------------------------- | --------------------------------------------------------- |
| `VITE_API_URL`               | API base URL. Defaults to `http://localhost:3000/api/v1`. |
| `VITE_GOOGLE_MAPS_EMBED_KEY` | Google Maps Embed API key.                                |

---

## Docker & Databases

### Bring up the full stack

```bash
docker compose up --build
```

`--build` is needed the first time and after any dependency change (new `yarn.lock` → new install layer). Source-only changes are picked up automatically through bind mounts — no rebuild required.

| Service    | Image                              | Host port          | Notes                                                                   |
| ---------- | ---------------------------------- | ------------------ | ----------------------------------------------------------------------- |
| `db`       | `postgres:16`                      | `${POSTGRES_PORT}` | Data persists in the `db_data` named volume. Healthcheck gates the API. |
| `redis`    | `redis:7`                          | `${REDIS_PORT}`    | Password-protected, data in `redis_data`. Healthcheck gates the API.    |
| `backend`  | `apps/api/Dockerfile` target `dev` | `${APP_PORT}`      | Waits on `db` and `redis` healthchecks.                                 |
| `frontend` | `apps/web/Dockerfile` target `dev` | `5173`             | Vite dev server, HMR enabled.                                           |

### Working with the datastores

```bash
# Start only datastores, then run Node on the host
docker compose up -d db redis

# Inspect live logs
docker compose logs -f db
docker compose logs -f redis

# Psql into the running Postgres container
docker compose exec db psql -U "$POSTGRES_USER" "$POSTGRES_DB"

# Redis CLI with auth
docker compose exec redis redis-cli -a "$REDIS_PASSWORD"

# Wipe everything (including volumes — destructive)
docker compose down -v
```

### Why the build context is the repo root

Both Dockerfiles are invoked with `context: .` and `file: apps/<app>/Dockerfile`. That is mandatory in a Yarn workspaces monorepo: the build must see the root `yarn.lock`, `.yarnrc.yml`, `.yarn/`, and every workspace manifest so dependency resolution is deterministic and cross-workspace symlinks can be created inside the image.

---

## Shared Packages

### How `@car-import-calculator/shared-types` works

Each consumer declares the package with Yarn's workspace protocol:

```jsonc
// apps/api/package.json
"dependencies": {
  "@car-import-calculator/shared-types": "workspace:^"
}
```

At install time, Yarn creates a symlink from `apps/api/node_modules/@car-import-calculator/shared-types` to `packages/shared-types/`. Edits propagate instantly — TypeScript and Vite pick them up without a rebuild because the package exports source (`"main": "./src/index.ts"`). Nothing is ever published to npm; the package lives and dies with this repo.

### Adding a new shared type

1. Create or edit a file under `packages/shared-types/src/` and export from the barrel:

   ```ts
   // packages/shared-types/src/vehicle.ts
   export interface VehicleSummary {
     id: string;
     title: string;
     priceUsd: number;
   }
   ```

   ```ts
   // packages/shared-types/src/index.ts
   export * from "./vehicle";
   ```

2. Import from either app using the package name — never a relative path:
   ```ts
   import { VehicleSummary } from "@car-import-calculator/shared-types";
   ```
3. If you introduce a runtime dependency (rare — this package is type-only today), add it to `packages/shared-types/package.json` and re-run `yarn install` so the workspace hoists correctly.

No rebuild step is required for consumers when types change.

---

## CI / CD Pipeline

The workflow lives at `.github/workflows/deploy.yml` and runs on every push and pull request targeting `main`.

### Flow

1. **`changes` job** — runs `dorny/paths-filter` to decide which workspaces were affected. Edits to `packages/shared-types/**`, root `package.json`, `yarn.lock`, or `.yarnrc.yml` invalidate **both** apps; edits to `apps/api/**` or `apps/web/**` invalidate only that app. This detection job is needed because GitHub's top-level `on.push.paths` filter cannot express "rebuild both when a shared package changes."
2. **`api` job** (conditional) — install with `yarn install --immutable`, lint, test, assume an AWS role via OIDC, log in to ECR, build and push `apps/api/Dockerfile` (target `runtime`) with `context: .`, then force a new ECS deployment. A Lambda alternative is included in comments.
3. **`web` job** (conditional) — same pattern, builds the Vite bundle and pushes an nginx image to ECR. An S3 + CloudFront deployment variant is included in comments.

Both image builds use `cache-from`/`cache-to: type=gha` so layer caching survives across runs.

### Required GitHub secrets

| Secret                | Used by                  | Description                                                                                               |
| --------------------- | ------------------------ | --------------------------------------------------------------------------------------------------------- |
| `AWS_DEPLOY_ROLE_ARN` | Both jobs                | IAM role assumed via GitHub OIDC. Grant ECR push and ECS/Lambda update permissions.                       |
| `ECS_CLUSTER`         | `api` job                | Name of the ECS cluster hosting the API service.                                                          |
| `ECS_API_SERVICE`     | `api` job                | Name of the ECS service to redeploy.                                                                      |
| `LAMBDA_API_FUNCTION` | `api` job (if on Lambda) | Function name for `aws lambda update-function-code`. Only needed if you uncomment the Lambda deploy step. |
| `VITE_API_URL`        | `web` job                | Public API base URL baked into the Vite bundle at build time.                                             |
| `WEB_BUCKET`          | `web` job (if on S3)     | Destination S3 bucket when using the static-site deployment variant.                                      |
| `CF_DISTRIBUTION_ID`  | `web` job (if on S3)     | CloudFront distribution whose cache is invalidated after deployment.                                      |

Before the first successful run you also need:

- Two ECR repositories created: `car-import-calculator-api` and `car-import-calculator-web` (or update the `env:` block with your chosen names).
- A GitHub OIDC trust relationship configured on the deploy role (`token.actions.githubusercontent.com` as the principal).

---

## Repository Layout

```
car-import-calculator/
├── apps/
│   ├── api/                    @car-import-calculator/api
│   └── web/                    @car-import-calculator/web
├── packages/
│   └── shared-types/           @car-import-calculator/shared-types
├── .github/workflows/deploy.yml
├── docker-compose.yml
├── package.json                Yarn workspaces + root scripts
├── .yarnrc.yml                 Yarn 4 configuration
├── .dockerignore
├── .gitignore
└── CLAUDE.md                   Contributor / agent guidance
```

---

## License

Apache-2.0 license.
