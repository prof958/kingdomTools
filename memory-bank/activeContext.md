# Active Context — KingdomTools

## Current Phase
**Phase 0 — Scaffolding**

## Current Focus
Setting up the foundational project structure:
- Next.js + TypeScript project initialization
- Prisma schema definition (all phases, including kingdom tables)
- Docker Compose configuration (app + db + caddy)
- Password gate middleware
- Tab-based navigation shell
- PF2e rule engine stubs (`lib/pf2e/`)
- Memory Bank files (this file and siblings)

## Recent Decisions
- **Player helper, not GM tool** — no encounter rollers, NPC trackers, or hex logs
- **Group-first inventory** — `InventoryItem.character_id` nullable (null = shared)
- **Minimal character model** — name + STR modifier only
- **No session notes** — handled externally
- **Inventory (Phase 1) before Dashboard (Phase 2)** — highest player value first
- **Caddy over nginx** — auto-TLS, minimal config
- **Enhancement features**: loot splitter, shopping wish list, recipe book

## Next Steps
1. ~~Create Memory Bank files~~ ✓
2. Scaffold Next.js project with TypeScript
3. Install and configure Tailwind + shadcn/ui
4. Define Prisma schema
5. Create PF2e rule modules
6. Build password gate middleware
7. Create app layout with tab navigation
8. Set up Docker Compose + Caddy
9. Verify everything compiles and runs

## Open Questions
- None currently — plan is approved and implementation is underway
