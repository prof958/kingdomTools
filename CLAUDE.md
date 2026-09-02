# CLAUDE.md — KingdomTools

## Core Principles
- **Player-first design**: Every feature must serve players directly. If it's only useful to a GM, it doesn't belong here.
- **Group-first inventory**: Default view is shared party loot. Player assignment is secondary.
- **PF2e correctness**: Bulk, encumbrance, coins, and investment rules must match RAW (Rules As Written).
- **Foundry complement**: This tool handles campaign logistics; Foundry handles tactical play. No overlap.

## Technical Conventions
- TypeScript strict mode everywhere
- Server Components by default; `'use client'` only when interactivity requires it
- Prisma for all DB access — never raw SQL unless absolutely necessary
- PF2e rules live in `lib/pf2e/` as pure functions — no side effects, no DB calls
- Canvas components (react-konva) must use `dynamic(() => import(...), { ssr: false })`
- API routes handle mutations; Server Components handle reads
- Use `router.refresh()` after mutations (see `patchKingdom` in `kingdom-shell.tsx` for the pattern: PATCH, then refresh, with a toast on failure)

## Naming Conventions
- Database tables: PascalCase (Prisma convention)
- API routes: kebab-case paths (`/api/inventory`, `/api/campsite`)
- Components: PascalCase files (`InventoryTable.tsx`)
- Utilities: camelCase files (`bulk.ts`, `currency.ts`)
- Types: PascalCase, suffixed with purpose (`InventoryItemWithRelations`, `BulkCalculation`)

## Data Patterns
- `character_id` nullable = shared party resource (inventory items, wallets)
- JSONB for flexible/evolving data (campsite layouts, kingdom logs)
- Self-referencing FK for container relationships (`container_inventory_item_id`)
- Axial coordinates (q, r) for hex grid, with a `sheet` field since (q, r) only identifies a hex within one map sheet

## Deployment
- Docker Compose: 3 services (app, db, caddy) — see `docs/DEPLOY.md` for the full VPS runbook
- Caddy for auto-TLS — never manually manage certificates
- `prisma migrate deploy` runs in the Dockerfile CMD on every container start
- Daily `pg_dump` backups
- Local dev is fully isolated from prod (separate compose file, separate volume) — see `README.md`

## Project History
Ongoing project state, phase-by-phase build history, and open questions live in
`memory-bank/` (`projectbrief.md`, `productContext.md`, `activeContext.md`,
`systemPatterns.md`, `techContext.md`, `progress.md`) — read those for what's
been built, what's in progress, and past decisions before starting new work.
