# Active Context — KingdomTools

## Current Phase
**Phase 5 — Kingdom** 🚧 IN PROGRESS

## Current Focus
Phase 5 Kingdom — building toward a **game-feel** Kingdom section (Travian-like), not a
spreadsheet. Rules engine, schema, bootstrap API and the Dashboard page are in. Art assets
are extracted from the Player's Guide. The activity catalog, dice module and V&K trained
skills just landed. Next: schema migration for the trained-skill fields, then the founding
wizard → hex map → settlements/Urban Grid → turn tracker → theme pass.

## Phase 5 Direction (decided with the user)
- **Game feel over forms** — the Kingdom section should read like a browser kingdom-builder.
- **Milestone levelling, no XP** — the GM sets the kingdom level. All XP machinery removed
  from the engine, the schema field is still there but unused by the UI.
- **Turn engine: roll, then confirm** — the app rolls and shows the full modifier breakdown
  and proposed outcome, but nothing is written until someone applies it (and it stays
  editable first).
- **Onboarding: founding + leadership** — a character-creation-style wizard covering
  Kingdom Creation steps 1-5 plus the leadership-role draft.
- **Art comes from the Player's Guide** — top-down structure tiles and the Stolen Lands map
  sheets, extracted by `scripts/extract-kingdom-assets.py`. Paizo assets marked "personal
  use only": fine behind the password gate, do not make the app publicly readable.

## Recent Decisions
- **Player helper, not GM tool** — no encounter rollers, NPC trackers, or hex logs
- **shadcn/ui uses @base-ui/react** (NOT Radix) — no `asChild` prop, Select.onValueChange is `(value: string | null, eventDetails) => void`
- **bcrypt dev fallback** — hash hardcoded in auth.ts because `$` conflicts with dotenv-expand
- **Kingdom ruleset** — support both "RAW" and "VK" (Vance & Kerenshara house rules) behind
  a per-kingdom `ruleset` field, **default VK**. They differ only in advancement (skill
  increase cadence, ability-boost count, Untrained Improvisation) and the finalize-boost
  count (RAW 2 / VK 3). `src/lib/pf2e/kingdom.ts` is the single source of truth.
- **Kingdom schema redesigned** from the Phase-0 placeholder models. New migration
  `20260828120000_redesign_kingdom` drops the old kingdom tables (incl. blocks/lots/
  structures) and rebuilds. `Block`/`Lot` gone; settlements use a JSONB `grid`;
  `KingdomStructure` is a seedable catalog. **Migration not yet applied** (no local DB in
  the dev session where it was written) — run `npx prisma migrate dev`.
- **Vitest** — 65 tests across `kingdom.test.ts`, `kingdom-activities.test.ts` and
  `dice.test.ts`. Scripts: `npm test`, `npm run test:watch`. No vitest.config (v4 zero-config).
- **The activity catalog is generated, not hand-written** — `kingdom-activities.ts` comes
  from `scripts/extract-kingdom-activities.py`. Re-run the script rather than editing the
  data. The V&K additions/amendments live in `VK_NEW` / `VK_PATCH` inside that script.
- **The map sheets were NOT stitched.** The guide calls them "blank maps" (plural) and each
  carries its own compass rose. Two independent correlation searches each produced a
  confident-looking arrangement (a 2x2, then a horizontal strip) that turned out to be the
  repeating hex lattice matching itself. Do not re-stitch without outside evidence of the
  intended layout.
- **Hex geometry is measured** — pointy-top, 175px column pitch, 152px row pitch, ~101px
  circumradius, per-sheet grid origins in `public/kingdom/map/manifest.json`. The ratio
  175/152 = 1.151 against the 1.1547 a pointy-top lattice predicts.
- **react-konva** — requires `dynamic(() => import(...), { ssr: false })` for Next.js compatibility
- **Zustand** — used only for canvas state management (positions, selections, zoom/pan)
- **JSONB elements** — campsite layout positions stored as JSON in CampsiteLayout.elements column
- **Prisma objects → client** — requires `JSON.parse(JSON.stringify(...))` to strip non-serializable fields

## Completed Phases

### Phase 0 — Scaffolding ✅
- Next.js 16 + TS scaffold, Tailwind v4 + shadcn/ui (12 components)
- Prisma schema (19 models, 4 enums), PF2e rule modules
- Password gate (JWT middleware), tab navigation, Docker config
- Local dev env (PostgreSQL 16, migration applied)

### Phase 1 — Inventory ✅
- Campaign bootstrap, Character CRUD, Item catalog (54 items)
- Inventory CRUD, Bulk tracker, Wallet manager, Loot splitter
- Invested/worn toggles, Wish list API (UI deferred)

### Phase 2 — Dashboard ✅
- Objectives API + tracker UI (add/edit/complete/fail/reactivate/priority/filter)
- Quick Links API + manager UI (add/edit/delete/categorize/suggested defaults)
- Wealth summary widget (aggregate wallets, treasury + per-character)
- Dashboard wired as server component with real data
- 15 API routes total, clean production build

## Completed (Phase 3 — Campsite) ✅
- Campsite API routes: /api/campsite, /api/campsite/[id], /api/recipes, /api/recipes/[id]
- Zustand canvas store: src/stores/campsite-store.ts (elements, selection, zoom/pan)
- react-konva canvas: dynamic SSR-disabled, draggable elements, zoom/pan, grid, selection
- Activity picker: 10 PF2e activities, character assignment, result tracking
- Watch order: numbered shifts with character toggle badges
- Recipe book: CRUD, discovered/undiscovered toggle, expandable effects
- Campsite shell: tabbed layout, layout switching/create/delete, save with dirty tracking
- Page wired as server component, 21 API routes total, clean build

## Key Phase 3 Files
- `src/stores/campsite-store.ts` — Zustand store (CampElement, ELEMENT_PALETTE)
- `src/components/campsite/campsite-canvas.tsx` — react-konva canvas (responsive, SSR-disabled)
- `src/components/campsite/campsite-shell.tsx` — client wrapper with tabs
- `src/components/campsite/activity-picker.tsx` — character → activity assignment
- `src/components/campsite/watch-order.tsx` — shift manager with badge toggles
- `src/components/campsite/recipe-book.tsx` — recipe CRUD + discover toggle

## Key Phase 4 Files
- `src/components/inventory/wish-list.tsx` — wish list UI (catalog + custom, assign, price, acquired)
- `src/components/inventory/inventory-shell.tsx` — updated with Wish List tab
- `src/app/(app)/inventory/page.tsx` — updated to fetch wish list data

## Key Phase 5 Files
- `src/lib/pf2e/kingdom.ts` — rules engine: charters, heartlands, governments, 16 skills,
  8 leadership roles, Size table, Control DC, proficiency/skill modifiers, ruin,
  RAW vs VK advancement tables, `computeAbilityScores`, `startingSkills`, turn phases
- `src/lib/pf2e/kingdom-activities.ts` — 43 activities (41 RAW + 2 V&K), GENERATED
- `src/lib/dice.ts` — dice + PF2e degree of success, injectable rng
- `scripts/extract-kingdom-assets.py` — map sheets + 69 structure tiles → `public/kingdom/`
- `scripts/extract-kingdom-activities.py` — the activity catalog generator
- `src/lib/pf2e/kingdom.test.ts` / `kingdom-activities.test.ts` / `src/lib/dice.test.ts`
- `src/lib/kingdom.ts` — `getOrCreateKingdom()` (seeds 16 skills + 8 roles)
- `src/app/api/kingdom/route.ts` — GET / PATCH scalar fields (whitelisted)
- `src/app/api/kingdom/leadership/route.ts` — PATCH one role (assign char/NPC, invest)
- `src/app/api/kingdom/skills/route.ts` — PATCH one skill's rank (0–4)
- `src/components/kingdom/*` — shell + overview / skills / leadership-roster / founding-choices
- `src/app/(app)/kingdom/page.tsx` — server component, replaces the placeholder

## Open Questions
- Do the four Stolen Lands map sheets tile into one map, and in what arrangement? Could not
  be determined from the PDF. If the physical copy or a Foundry module settles it, the
  offsets drop straight into `public/kingdom/map/manifest.json`.
