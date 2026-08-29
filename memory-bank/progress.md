# Progress — KingdomTools

## Implementation Phases

| Phase | Scope | Status |
|-------|-------|--------|
| 0 — Scaffolding | Project setup, schema, Docker, auth, nav | ✅ Complete |
| 1 — Inventory | Group inventory, bulk calc, wallets, loot splitter | ✅ Complete |
| 2 — Dashboard | Quests, wealth summary, quick links | ✅ Complete |
| 3 — Campsite | Canvas editor, activities, watches, recipes | ✅ Complete |
| 4 — Polish | Wish list UI, UX refinements, mobile, seed data | ✅ Complete |
| 5 — Kingdom | Hex grid, kingdom stats, settlements, turns | 🚧 In Progress |

## What Works

### Phase 0 — Scaffolding
- ✅ Next.js 16.2.1 + TypeScript 5 (strict mode)
- ✅ Tailwind v4 + shadcn/ui (base-nova style, @base-ui/react — NOT Radix)
- ✅ Prisma 7.6 schema (19 models, 4 enums) with @prisma/adapter-pg
- ✅ PF2e rule engine: bulk.ts, currency.ts, investment.ts, camping.ts
- ✅ Password gate: JWT sessions via jose + bcryptjs, dev fallback hash
- ✅ Tab navigation: Dashboard, Inventory, Campsite, Kingdom
- ✅ Docker Compose (app + db + caddy), Dockerfile (multi-stage), Caddyfile
- ✅ Local dev: PostgreSQL 16, database migrated, dev server at localhost:3000

### Phase 1 — Inventory Management
- ✅ Campaign bootstrap (`getOrCreateCampaign()` auto-creates campaign + treasury)
- ✅ Character CRUD API + UI (name + STR modifier)
- ✅ Item catalog seed data (54 PF2e items via prisma/seed.ts)
- ✅ Custom item creation form (add-item-dialog.tsx custom tab)
- ✅ Inventory CRUD: add from catalog, add custom, assign, quantity, delete
- ✅ Bulk calculation display per character (bulk-tracker.tsx)
- ✅ Wallet management: party treasury + individual wallets (wallet-manager.tsx)
- ✅ Loot splitting calculator with preview and remainder handling
- ✅ Invested item tracking + worn/equipped toggles
- ✅ Wish list API routes (/api/wishlist, /api/wishlist/[id]) — UI deferred to Phase 4

### Phase 2 — Dashboard
- ✅ Objectives API: GET/POST /api/objectives, PATCH/DELETE /api/objectives/[id]
- ✅ Quick Links API: GET/POST /api/quick-links, PATCH/DELETE /api/quick-links/[id]
- ✅ Objective tracker UI: add, edit, complete, fail, reactivate, priority levels, filtering
- ✅ Quick links manager UI: add, edit, delete, categorize, suggested PF2e defaults
- ✅ Party wealth summary widget: aggregate all wallets, treasury + per-character breakdown
- ✅ Dashboard page wired as server component with real data
- ✅ Kingdom turn reminder placeholder ("Coming Soon" card)
- ✅ 15 API routes total, clean production build

### Phase 3 — Campsite Planner
- ✅ Campsite layout API: GET/POST /api/campsite, PATCH/DELETE /api/campsite/[id] (JSONB elements, watch shifts, activities)
- ✅ Recipe API: GET/POST /api/recipes, PATCH/DELETE /api/recipes/[id]
- ✅ Zustand canvas store (src/stores/campsite-store.ts): elements, selection, zoom/pan, dirty tracking
- ✅ react-konva interactive canvas (dynamic SSR-disabled): drag elements, zoom/pan, grid, selection ring
- ✅ Element palette: tent, campfire, bedroll, trap, marker + character tokens
- ✅ Activity picker UI: character → activity assignment from 10 PF2e activities, result tracking
- ✅ Watch order manager UI: numbered shifts, toggle characters per shift
- ✅ Recipe book UI: add/delete recipes, toggle discovered/undiscovered, expandable effects
- ✅ Campsite shell: tabbed layout (Layout/Activities/Watch/Recipes), layout switching, save/delete
- ✅ Campsite page wired as server component with real data
- ✅ 21 API routes total, clean production build

## What's Left (Phase 4 — Polish)
- [✓] Wish list UI (catalog search + custom items, assign to character, price tracking, acquired toggle)
- [✓] Wish list tab added to Inventory page (3rd tab: Inventory | Wallets | Wish List)
- [✓] Mobile responsiveness: campsite canvas auto-resizes to container, button text hidden on small screens
- [✓] Responsive table columns (hidden on small screens with mobile inline alternatives)

### Phase 4 — Polish
- ✅ Wish list UI: catalog search, custom items, assign to character/party, price in GP, notes, acquired toggle
- ✅ Wish list table: pending/acquired filtering, total pending cost, level badges, responsive columns
- ✅ Wish list integrated as 3rd tab on Inventory page (Inventory | Wallets | Wish List)
- ✅ Campsite canvas auto-resizes to container width (responsive, min 300px height)
- ✅ Campsite shell: flex-wrap button bar, hidden button labels on mobile
- ✅ 21 API routes, clean production build

## What's Left (Phase 5 — Kingdom) 🚧 IN PROGRESS
- [x] Kingdom tab design and plan (rules analysis, schema, UI architecture).
- [x] Rules engine `src/lib/pf2e/kingdom.ts` + 35 Vitest cases (RAW & VK rulesets).
- [x] Schema redesign + migration `20260828120000_redesign_kingdom` (skills, feats,
      founding choices, ruin tracks, fame, size; JSONB settlement grid; structure catalog).
      Applied in prod via `prisma migrate deploy` — now wired into the Dockerfile CMD
      (previously nothing ran migrations on deploy; that was the cause of the first
      P2022 "kingdoms.ruleset does not exist" runtime error on Coolify).
- [x] `getOrCreateKingdom()` bootstrap + `/api/kingdom`, `/api/kingdom/leadership`,
      `/api/kingdom/skills` (24 API routes total, clean build).
- [x] Kingdom Dashboard page: Overview (abilities, derived stats, unrest, ruin, resources),
      Skills (16 skills w/ computed modifiers + rank picker), Leadership (assign + invest),
      Founding (charter/heartland/government + boost preview & apply).
- [x] Art assets extracted from the Player's Guide into `public/kingdom/`: four blank
      Stolen Lands map sheets (with measured pointy-top hex geometry, 175x152px pitch) and
      69 top-down structure tiles with a slug/lots manifest.
      `scripts/extract-kingdom-assets.py`.
- [x] Activity catalog — 43 activities (41 RAW + V&K's Take Charge and Reconnoiter Hex),
      each with phase, eligible skills and ranks, requirements, and the four degree
      outcomes. Generated by `scripts/extract-kingdom-activities.py`; 14 integrity tests
      guard against extraction regressions.
- [x] Dice module `src/lib/dice.ts` — rolls plus PF2e degree of success with the natural
      20/1 shift, injectable rng so results are testable. 9 cases.
- [x] V&K starting trained skills — charter and heartland each grant a set skill plus a
      free pick; a duplicate grant converts to another free pick. `startingSkills()`.
- [x] XP removed from the engine and the Overview UI (milestone levelling).
- [x] Migration `20260828170000_kingdom_founding` — adds `kingdoms.skill_picks` and
      `kingdoms.founded`, drops the unused `xp` column. **Not yet applied** (no local DB
      and no Docker in the session where it was written); `prisma migrate deploy` runs it
      on the next Coolify deploy. Existing kingdoms with a government are marked founded so
      they land on the dashboard rather than back in onboarding.
- [x] Founding + leadership onboarding wizard — an 8-step character-creation-style flow
      (name/ruleset → charter → heartland → government → ability finalization → trained
      skills → leadership draft → review) with heraldic ability crests that update live as
      choices are made, a step rail, and per-step gating. `POST /api/kingdom/found` commits
      it in one transaction, recomputing scores and skills server-side so stored state can
      never drift from the choices. The Kingdom tab shows the wizard while `founded` is
      false and the dashboard afterwards.
- [x] Hex Map — a Konva canvas over the four printed sheets with a hex overlay that
      sits exactly on the painted grid. Sheet picker, scroll-zoom, drag-pan, click to
      select; a hex inspector sets state / terrain / work site / terrain features / roads /
      fortification / label. Claimed and reconnoitered hexes are tinted, untouched hexes
      draw nothing so the printed map stays readable. The Claim Hex adjacency rule is
      surfaced in the inspector. `GET`/`PATCH /api/kingdom/hexes` store hexes lazily and
      recount the kingdom's Size from claimed hexes on every write.
- [x] Migration `20260828180000_hex_sheet` — `hexes.sheet` plus a widened uniqueness
      constraint, since (q, r) only identifies a hex within one sheet. **Not yet applied.**
- [x] `src/lib/hex.ts` — pointy-top hex maths (centres, corners, hit-testing, neighbours,
      distance) with 21 tests, including a round-trip over every hex centre on a sheet.
- [x] Grid origin solved properly: fitting the whole hexagon outline instead of a
      one-dimensional comb of grid lines. A hex lattice has vertical edges every *half*
      column pitch, so the comb scored two phases equally and picked between them by noise
      — it had produced origins disagreeing by 90px between identical sheets.
- [x] Settlements & Urban Grid. `src/lib/urban-grid.ts` (pure state-transition logic, 35
      tests): 3x3 blocks of 2x2 lots per Urban Grid instance, flattened to one 6x6 lot
      space rather than nested per-block grids — that's what lets a 2-lot structure span
      two lots in different blocks (RAW doesn't require multi-lot structures to stay
      within one block). Handles placement/removal, rubble (a failed-Demolish/event state
      distinct from empty), block activation with RAW's contiguity rule (required for
      village/town, dropped for city/metropolis, which may build in any of the 9 blocks),
      settlement level and Overcrowded derivation. `GET/POST /api/kingdom/settlements` +
      `PATCH/DELETE /api/kingdom/settlements/[id]` (one `action`-discriminated endpoint for
      every grid mutation); level/overcrowded are recomputed server-side on every mutation
      and at creation, never trusted from the client — same discipline as kingdom Size.
      UI: `SettlementsTab` (switcher, founding flow, borders, inspector) +
      `UrbanGridEditor` (the flat 6x6 grid, one visual item per placement/inactive-block/
      empty-lot, each with its own `gridColumn`/`gridRow` span) + `StructurePicker`
      (searches the 74-structure catalog, shows the extracted tile art, filters by fit and
      kingdom level). Cost is shown but NOT deducted — no resource economy exists yet
      outside manual Overview edits, and deducting only here would be a worse
      inconsistency than deferring it to the turn tracker, which is where a real Build
      Structure check belongs anyway.
      NOTE (real bug, fixed): tile art used `next/image`, whose `/_next/image` optimizer
      route does a server-side loopback fetch with no session cookie — the auth proxy
      redirected it to `/login`, so every tile "loaded" as an HTML page instead of a PNG.
      Same root cause class as the hex map's asset-gating issue earlier, different code
      path (a browser-issued `<img>` carries the real cookie; the optimizer's server-side
      fetch does not). Switched to plain `<img>`, matching the campsite canvas's existing
      convention.
      Verified live end-to-end against the real dev DB: founded a village, placed/selected/
      demolished structures, cycled a border, toggled capital — all persisted correctly,
      including `level`/`overcrowded` recomputing right (0 → 1 after the first structure).
- [ ] Turn Tracker (roll, then confirm).
- [x] Structure catalog — 74 structures with level, lot count, build cost, construction
      check, upgrade chains, item bonuses, effects and Ruin, each joined to its top-down
      tile. Generated by `scripts/extract-kingdom-structures.py` into
      `src/lib/pf2e/kingdom-structures.ts` (a TS module like the activities, not a DB
      seed), with the V&K item-bonus additions layered on and flagged. 21 tests.
      NOTE: this makes the `kingdom_structures` table redundant; it is still in the
      schema, unused.
- [x] Army activities added to the activity catalog (51 total). The catalog now covers
      the warfare chapter too, and Recruit Army correctly lands as a Leadership activity
      as the guide specifies.
- [x] `dotenv` moved to runtime dependencies. `prisma.config.ts` imports it, so any
      future prune of dev dependencies would have broken `prisma migrate deploy` in the
      Dockerfile CMD — a deploy-time failure, not a build-time one.
- [x] Kingdom Size is now read-only in the Overview and rejected by `PATCH /api/kingdom`;
      the map is its only source.
- [x] Delete kingdom — type-to-confirm `DangerZone` on the Founding tab, `DELETE
      /api/kingdom`. Name match enforced server-side; cascade enforced by Postgres FK
      constraints, not app code. Recreates as a fresh unfounded kingdom on next load.
- [x] Local dev environment — `docker-compose.dev.yml` (isolated Postgres, fixed
      credentials, separate volume/filename from prod), `.env.example` recreated,
      `db:up`/`db:down`/`db:reset`/`db:migrate`/`db:seed`/`db:studio` npm scripts,
      `dev:setup` for one-command first run. Fixed three real bugs found while building
      it: a `.gitignore` line silently re-blocking `.env.example`, Docker Compose
      mangling the bcrypt hash in `.env` via its `$`-as-variable parsing, and
      `prisma/seed.ts` never loading `.env` itself. Verified end-to-end: fresh volume →
      all 13 migrations → seed → typecheck/tests/build all clean. See README.md.
- [ ] Theme pass — the app is still the default neutral shadcn greyscale.

## Known Issues
- Deploy (Coolify/Docker) previously ran **no** DB migrations — Dockerfile CMD now does
  `npx prisma migrate deploy && npm start`. Requires `_prisma_migrations` history to be
  intact on prod; check with `npx prisma migrate status` if a deploy migration fails.
- `next start` warns it "does not work with output: standalone" — app still serves; proper
  fix is `node .next/standalone/server.js` + copying `.next/static` and `public`, deferred.
- Next.js 16 warns that `middleware` is deprecated in favor of `proxy` — functional, no migration needed yet
- PowerShell requires `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force` each new terminal session
- bcrypt hashes contain `$` which conflicts with dotenv-expand — dev fallback hash hardcoded in auth.ts; production uses real env var
