# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project status

This is a fresh scaffold for a crypto portfolio/tax tracker ("koinly-clone"). No domain
models, auth, or business logic exist yet — the app currently renders the default Next.js
starter page. Treat the architecture notes below as the foundation new features get built on,
not a description of existing features.

## Commands

```bash
npm run dev              # start dev server (Turbopack) at localhost:3000
npm run build             # production build (also type-checks)
npm run start              # run the production build
npm run lint                 # eslint (flat config, eslint-config-next)
npx tsc --noEmit               # type-check only, no build

npm run prisma:generate          # regenerate the Prisma client after schema.prisma changes
npm run prisma:migrate            # create + apply a dev migration
npm run prisma:studio              # open Prisma Studio GUI against DATABASE_URL
```

There is no test runner configured yet. `postinstall` runs `prisma generate` automatically, so
the generated client exists right after `npm install`.

## Architecture

- **Framework**: Next.js 16 (App Router) with React 19, TypeScript, Tailwind CSS v4. Source
  lives under `src/`, routes under `src/app/`. The `@/*` path alias maps to `src/*`
  (`tsconfig.json`).
- **Next.js 16 is not the Next.js in your training data.** Breaking changes exist versus older
  versions — before writing routing/data-fetching/config code, check the bundled docs in
  `node_modules/next/dist/docs/01-app/` rather than assuming prior knowledge. This is enforced
  via the `@AGENTS.md` import above.
- **Database**: PostgreSQL via Prisma ORM 7, using the newer `prisma-client` generator (not the
  legacy `prisma-client-js`). Key consequences of this generator choice:
  - The schema (`prisma/schema.prisma`) generates into `src/generated/prisma/` (gitignored,
    regenerated via `postinstall`/`prisma:generate`) rather than into `node_modules`.
  - Import the client from the specific file, not a package-style root: `@/generated/prisma/client`.
  - Prisma Client is instantiated with an explicit driver adapter (`@prisma/adapter-pg` + `pg`),
    not a bare connection-string client. See `src/lib/prisma.ts` — always import `prisma` from
    there rather than constructing a new `PrismaClient`, to avoid exhausting connections via
    Next.js dev-mode hot reload (the module caches the instance on `globalThis` outside
    production).
  - `prisma.config.ts` (not the `datasource` block alone) is what wires `DATABASE_URL` from
    `.env` — see `prisma.config.ts` if connection config ever needs to change.
  - Reference docs for the Prisma CLI, client API, and Postgres/driver-adapter setup are checked
    into `.agents/skills/prisma-*/` — consult these before guessing at Prisma 7 APIs, since they
    differ from the widely-documented `prisma-client-js` generator.
- **Environment**: `DATABASE_URL` (Postgres connection string) must be set in `.env` (not
  committed). `GET /api/health` (`src/app/api/health/route.ts`) round-trips a `SELECT 1` through
  Prisma and is the fastest way to confirm DB connectivity end-to-end.
- **Styling**: Tailwind v4 via `@tailwindcss/postcss` (`postcss.config.mjs`); no `tailwind.config`
  file is needed for v4's CSS-first configuration. Dark mode is handled with Tailwind's `dark:`
  variant throughout.
