# AGENTS

## Repo Shape
- Root repo is an orchestrator with Git submodules: `backend/` and `frontend/` track separate repos on their `develop` branches (`.gitmodules`). Check each submodule for changes before editing.
- Treat `backend/` and `frontend/` as independent Node apps. There is no root workspace manifest or root package script layer.

## Preferred Run Paths
- Full stack: `docker compose up --build` from repo root. This is the only verified root-level run flow (`compose.yaml`).
- First boot can fail because `backend` may start before Postgres is ready; `backend/README.md` explicitly says to restart `docker compose` once.
- Local frontend-only dev: run the backend first, then run `npm start` in `frontend/`.
- Local backend-only dev: run `npm run dev` in `backend/`, but you still need a Postgres database matching `backend/prisma/schema.prisma`.

## Commands
- Backend (`backend/package.json`): `npm run dev`, `npm start`, `npm run prisma:generate`, `npm run prisma:migrate`, `npm run prisma:studio`
- Frontend (`frontend/package.json`): `npm start`, `npm run build`, `npm test`
- The repo contains both `package-lock.json` and `pnpm-lock.yaml`, but the checked-in scripts and Dockerfiles use `npm`/`npx`. Default to `npm` unless you are intentionally fixing package manager drift.

## Runtime Wiring
- Nginx is part of the expected full-stack setup. It terminates HTTPS and proxies `/api/` to the backend while stripping the `/api` prefix (`nginx.conf`).
- Frontend API code is written around that proxy: `frontend/src/services/api-config.js` falls back to `/api`, and `frontend/src/services/client.js` sends `withCredentials: true`.
- Auth is cookie-based in practice, not localStorage-based: login sets an `accessToken` cookie, frontend auth checks `GET /auth/check`, and most route protection depends on cookies surviving the proxy.

## Files And Data Coupling
- In Docker, backend system images are coupled to frontend static assets through `compose.yaml`: `./frontend/public` is mounted into `/app/public` inside the backend container.
- Backend serves that directory at `/public/*` and also exposes `/system-images/*`; frontend hooks still assume default files like `/img/logo.png` and `/img/bg-cover.jpg`. If you change image handling, check both backend controllers and frontend hooks/components.
- User-uploaded visitor/permissionario images are served from backend routes under `/images/...`, not directly from the frontend build.

## Codebase Entry Points
- Backend entrypoint is `backend/src/server.js`: Fastify app, CORS/cookies/multipart/static setup, then route registration.
- Frontend entrypoints are `frontend/src/index.js` and `frontend/src/App.js`. Despite `tsconfig.json`, the app root is still plain JS/CRA (`react-scripts`), with some mixed TS files deeper in `src/`.
- In backend route files, route declaration order matters. `backend/src/routes/entries-routes.js` explicitly keeps specific routes before generic `:id` routes.

## Verification Notes
- There is no repo-level CI workflow checked in under `.github/workflows/`.
- There are no backend automated tests configured in `backend/package.json`; backend verification is mainly targeted manual/API testing plus Prisma commands.
- Frontend test runner is the default CRA `react-scripts test`; use focused checks when possible instead of assuming a broader test harness exists.

## Useful Seed Context
- `backend/SEED_INSTRUCTIONS.md` documents a non-scripted seed flow using `node prisma/seed-faker.js` and test users `s2`, `guarda`, `scmt` with password `teste123`. Verify the seed file exists before relying on it in automation.
