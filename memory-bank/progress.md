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
- [ ] Migration for the trained-skill fields (charterSkill / heartlandSkill / picks).
- [ ] Founding + leadership onboarding wizard.
- [ ] Hex Map implementation.
- [ ] Settlements & Urban Grid.
- [ ] Turn Tracker (roll, then confirm).
- [ ] Seed `kingdom_structures` catalog (RAW + V&K item-bonus additions).
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
