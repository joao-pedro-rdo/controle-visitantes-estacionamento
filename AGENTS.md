# AGENTS

## Repo Shape
- Root repo is an orchestrator with Git submodules: `backend/` and `frontend/` track separate repos on their `develop` branches (`.gitmodules`). Check each submodule for changes before editing.
- Treat `backend/` and `frontend/` as independent Node apps. There is no root workspace manifest or root package script layer.

## Preferred Run Paths
- Full stack without Nginx: `docker compose --env-file .env -f docker/compose.yaml -f docker/compose.dev.yaml up --build` from repo root.
- Full stack with Nginx: add `-f docker/compose.nginx.yaml`; local TLS certificates are required.
- Compose waits for PostgreSQL health before starting the backend; the backend image runs Prisma migrations on startup.
- Local frontend-only dev: run the backend first, then run `npm start` in `frontend/`.
- Local backend-only dev: run `npm run dev` in `backend/`, but you still need a Postgres database matching `backend/prisma/schema.prisma`.

## Commands
- Backend (`backend/package.json`): `npm run dev`, `npm start`, `npm run prisma:generate`, `npm run prisma:migrate`, `npm run prisma:studio`
- Frontend (`frontend/package.json`): `npm start`, `npm run build`, `npm test`
- The repo contains both `package-lock.json` and `pnpm-lock.yaml`, but the checked-in scripts and Dockerfiles use `npm`/`npx`. Default to `npm` unless you are intentionally fixing package manager drift.

## Runtime Wiring
- Nginx is an optional Compose overlay at `docker/compose.nginx.yaml`. It terminates HTTPS and proxies `/api/` to the backend while stripping the `/api` prefix (`docker/nginx.conf`).
- Frontend API code is written around that proxy: `frontend/src/services/api-config.js` falls back to `/api`, and `frontend/src/services/client.js` sends `withCredentials: true`.
- Auth is cookie-based in practice, not localStorage-based: login sets an `accessToken` cookie, frontend auth checks `GET /auth/check`, and most route protection depends on cookies surviving the proxy.

## Files And Data Coupling
- In Docker, backend system images share the persistent host `frontend/public` bind mount at `/app/public`; deployment sync merges default frontend assets into that directory so uploaded system images are preserved.
- Backend serves that directory at `/public/*` and also exposes `/system-images/*`; frontend hooks still assume default files like `/img/logo.png` and `/img/bg-cover.jpg`. If you change image handling, check both backend controllers and frontend hooks/components.
- User-uploaded visitor/permissionario images are served from backend routes under `/images/...`, not directly from the frontend build.

## Codebase Entry Points
- Backend entrypoints are `backend/src/app.ts` (Fastify app/plugins/routes) and `backend/src/server.ts` (listen and metrics updater).
- Frontend entrypoints are `frontend/src/index.js` and `frontend/src/App.js`. Despite `tsconfig.json`, the app root is still plain JS/CRA (`react-scripts`), with some mixed TS files deeper in `src/`.
- In backend route files, route declaration order matters. `backend/src/routes/entries-routes.ts` explicitly keeps specific routes before generic `:id` routes.

## Verification Notes
- CI and the homolog deploy workflow live under `.github/workflows/`; production deploy is intentionally disabled in `deploy-prod.yml.disabled`.
- Backend tests use Vitest (`npm test`) and type/build checks use `npm run build`.
- Frontend test runner is the default CRA `react-scripts test`; use focused checks when possible instead of assuming a broader test harness exists.

## Useful Seed Context
- `backend/SEED_INSTRUCTIONS.md` documents a non-scripted seed flow using `node prisma/seed-faker.js` and test users `s2`, `guarda`, `scmt` with password `teste123`. Verify the seed file exists before relying on it in automation.
