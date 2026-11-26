# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

The imported `AGENTS.md` covers repo shape, run paths, runtime wiring, and file/image coupling. The notes below add what it does not.

## Project Context

- Access-control system (visitors, permissionários, military vehicles) for a military organization (OM), designed to run offline on an intranet. Code, docs, and UI are in Portuguese.
- Use the domain terms from `UBIQUITOUS_LANGUAGE.md` (e.g. **Visitante**, **Permissionario**, **Pessoa Nao Autorizada**, profiles **S2** and **Guarda**) and avoid the aliases it lists.
- `Task.md` is the phase checklist for the backend migration (based on `docs/backend-roadmap.md`). Update it when completing roadmap items.
- `CONTEXT.md` and `docs/deploy.md` describe the CI/CD model: `develop` → CI only; `develop`→`homolog` merge publishes `homolog` images and deploys to a self-hosted runner; `homolog`→`main` publishes `latest`/`sha-*` images and deploys to production only when `PROD_DEPLOY_ENABLED=true`. Images are built from the submodule gitlinks pinned in this root repo, so a submodule change only ships after the root repo bumps the gitlink.
- `back/` and root `prisma/schema.sql` are legacy leftovers; the real schema is `backend/prisma/schema.prisma`.

## Commands (beyond AGENTS.md)

- Backend single test file: `npx vitest run src/test/routes/vehicles-routes.test.ts` (from `backend/`); filter by name with `-t "<name>"`. Watch mode: `npm run test:watch`.
- Backend `npm run build` is a type-check only (`tsc --noEmit`). The backend runs TS directly through `tsx` and has no emitted build.
- Frontend build in CI runs with `CI=false` so lint warnings don't fail the build.
- If Docker builds fail with `SELF_SIGNED_CERT_IN_CHAIN` (TLS-inspecting network), set `NPM_CA_FILE` in `.env` and add `-f docker/compose.dev-ca.yaml`. Do not disable npm TLS validation.

## Backend Architecture

Fastify 4 + TypeScript (ESM, `NodeNext`, `strict: false`, `allowJs`). It is partway through a migration from JS to TS, so some `.js` controllers/routes/middleware remain. Rules from `Task.md` Fase 0: new features are TypeScript, every new route has a Zod schema, controllers contain no business logic, and Prisma access lives outside controllers.

Layering for migrated modules (follow `vehicles-*` as the reference):
- `routes/*-routes.ts`: register paths and attach `preHandler` auth guards (`verifyS2Role` etc. from `middleware/auth.js`).
- `controllers/*`: parse input with `parseBody`/`parseParams`/`parseQuery` (`lib/validation.ts`) and the Zod schemas in `schemas/`, call a service, and respond via `sendSuccess` (`lib/http.ts`, which returns raw data with no envelope).
- `services/*`: business rules. They throw `AppError` helpers (`badRequest`, `notFound`, `conflict`, … in `lib/errors.ts`) and map Prisma error codes through `hasPrismaCode`.
- `repositories/*`: the only layer that touches `prisma` (`lib/prisma.ts`). Each exports an interface plus a `prisma*Repository` implementation.
- `lib/error-handler.ts` converts `AppError`, `ZodError`, Fastify validation errors, and Prisma `P####` errors into standardized responses.
- Auth (`lib/auth.ts`): JWT in the `accessToken` cookie (8h). Roles are `S2` (admin) and `GUARDA_ROLES` (`Guarda`, `S2`, `Scmt`, `SFPC`). `JWT_SECRET` and `COOKIE_SECRET` env vars are required.
- `app.ts` exports `buildApp()` (no listen) so tests can use it; `server.ts` listens and starts the Prometheus gauge updater.

Testing pattern: route tests build the real app via `src/test/helpers/create-test-app.ts`, replace the repository module with an in-memory fake using `vi.mock("../../repositories/<x>-repository.js", ...)` plus `vi.hoisted` state, and call endpoints with `app.inject`. `src/test/helpers/auth.ts` logs in and returns the `accessToken` cookie header. `src/test/setup.ts` sets test secrets. Tests don't need a database.

## Frontend Architecture

CRA (`react-scripts` 5) with React 18, React Router 6, Chakra UI + Tailwind + styled-components. Page-level screens live in `src/routes/`, data and logic in `src/hooks/use*.js`, and auth state in `src/context/AuthContext.js`. All HTTP goes through the shared axios instance in `src/services/client.js`. `frontend/BACKEND_API_SETTINGS.md` documents the settings/system-image API contract the frontend expects.
