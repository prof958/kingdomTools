# Active Context — KingdomTools

## Current Phase
**Phase 5 — Kingdom** 🚧 IN PROGRESS

## Current Focus
Phase 5 Kingdom — building toward a **game-feel** Kingdom section (Travian-like), not a
spreadsheet. Rules engine, schema, bootstrap API and the Dashboard page are in. Art assets
are extracted from the Player's Guide. The activity catalog, dice module and V&K trained
skills landed, the founding wizard is in, the hex map works, and the structure catalog is
complete. Delete-kingdom (type-to-confirm) is in. A proper local dev environment exists.
The Settlement Urban Grid is in — the app's Select primitive was also fixed (see below,
it was silently showing raw enum values instead of labels app-wide). Next: turn tracker →
theme pass.

## Settlement Urban Grid (new)
- `src/lib/urban-grid.ts` — pure logic, no DB/React, 35 tests. One Urban Grid instance is
  3x3 blocks of 2x2 lots, but the module models it as one flat 6x6 lot grid rather than
  nested per-block grids, because a 2-lot structure can span two lots in *different*
  blocks (RAW never restricts multi-lot structures to one block) and a flat grid makes
  that just plain 2D arithmetic instead of a cross-container spanning problem.
  `whyCannotPlace`/`whyCannotActivateBlock` return a reason string (or null), which the
  API forwards straight to the UI as the error message — one source of truth for both.
- Village/Town block activation requires contiguity (RAW, KPG 46); City/Metropolis drop
  that and can use any of the 9 blocks. `settlementLevel` and `isOvercrowded` are derived
  from the grid every time, recomputed server-side on every mutation (and at creation —
  the schema's `level` default of 1 doesn't match an empty grid's derived 0, so the POST
  route sets it explicitly rather than trusting the default).
- Structure build cost is shown, not deducted — this app has no resource economy loop yet
  outside manual Overview edits; auto-deducting only in one place would be a worse
  inconsistency than waiting for the turn tracker, which is also where a real rolled
  Build Structure check belongs.
- Tile art must use a plain `<img>`, not `next/image` — its `/_next/image` route does a
  server-side loopback fetch with no session cookie, so the auth proxy redirects it to
  `/login` and the "image" that loads is an HTML page. Same failure class as the hex map
  sheets earlier in the session, different code path (`<img>` is a real browser request
  and carries the cookie; the optimizer's fetch does not).

## Select primitive fix (base-ui) (new)
`<Select.Value>` in `@base-ui/react` does NOT read the `label` prop on `<Select.Item>` —
that only feeds keyboard typeahead. The closed trigger's text comes from a separate
`items` map that must be passed to `<Select.Root>` explicitly; without it the trigger
shows the raw `value`. Every `<Select>` in the app (11 usages) was passing `label` on each
item expecting it to cover the trigger too, and none of them did — this is why Kingdom's
Founding tab showed `conquest`/`forest_swamp`/`__vacant__` instead of proper labels, and
it was affecting every other Select in the app too (e.g. a priority filter would have
shown `"0"` instead of "Low Priority"). Fixed once, centrally, in
`src/components/ui/select.tsx`: `Select` now walks its own children to auto-derive
`items` from the `label`/`value` pairs every call site already provides — no call site
had to change. Had to stay a *generic* function component (matching
`SelectPrimitive.Root`'s own `<Value, Multiple>` signature) rather than being typed via
`React.ComponentProps<typeof SelectPrimitive.Root>`, which collapses the generic and
breaks `onValueChange` typing at every call site.

## Local dev environment (new)
- **`docker-compose.dev.yml`** — an isolated local-only Postgres, separate from prod in
  every way that matters: different compose filename (prod's `docker compose up -d` on
  the VPS only ever reads `docker-compose.yml`), a named Docker volume
  (`kingdomtools_dev_pgdata`) instead of prod's bind mount, bound to `127.0.0.1` only.
  Fixed literal credentials (`kingdomtools` / `localdev`) rather than reading `.env` —
  see next point for why.
- **Real bug found**: Docker Compose always auto-loads `.env` from the cwd for `${VAR}`
  interpolation, *regardless of which compose file you asked for*. This repo's `.env` has
  a bcrypt `APP_PASSWORD_HASH` full of `$` characters, which Compose's parser reads as
  more variable references and mangles — the exact landmine `docs/DEPLOY.md` already
  warns about for the prod compose file, just not previously hit locally. Fixed by passing
  `--env-file docker-compose.dev.env` (a deliberately empty tracked file) in every `db:*`
  npm script, so the dev compose file never looks at the app's `.env` at all.
- **`.gitignore` had a real bug**: a stray second `.env.example` line at the bottom
  silently re-ignored the file an earlier `!.env.example` was explicitly un-ignoring —
  last-match-wins in gitignore. `.env.example` had been deleted from the repo at some
  point and could never be re-added while that line stood. Removed; `.env.example`
  recreated and now tracked.
- **`prisma/seed.ts` had a real bug**: it read `process.env.DATABASE_URL` directly but
  never loaded `.env` itself, so running it via its own documented command
  (`npx tsx prisma/seed.ts`) failed with an opaque pg SASL error unless `DATABASE_URL`
  happened to already be exported some other way. Fixed with `import "dotenv/config"`.
- **npm scripts**: `db:up` / `db:down` / `db:reset` / `db:migrate` / `db:seed` /
  `db:studio`, plus `dev:setup` (up + migrate) for a one-command first run. All verified
  end-to-end this session: `db:reset` from a torn-down volume through all 13 migrations
  applying cleanly, then `db:seed` loading 54 items.
- See `README.md` for the full local-dev walkthrough and the "why this can't touch
  production" explanation.
- Local `.env`'s `APP_PASSWORD_HASH` was cleared (it held a leftover custom hash with no
  known plaintext — bcrypt can't be reversed). Login now uses the documented dev fallback:
  password **`kingmaker`**.

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
  circumradius. The ratio 175/152 = 1.151 against the 1.1547 a pointy-top lattice predicts.
  Both pitches are kept rather than derived from one circumradius: a lattice with rowPitch
  152 wants colPitch 175.5, and deriving one from the other drifts ~4px across a sheet.
  Grid origins are found by fitting the full hexagon outline — a 1-D comb of grid lines is
  ambiguous, because a hex lattice has vertical edges every *half* column pitch.
- **Map assets are behind the password gate.** `/kingdom/map/*` is matched by the proxy
  like any other route, so the sheets 302 to /login when signed out. That is the right
  default for art licensed for personal use; it does mean an unauthenticated preview shows
  an empty canvas.
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
- `src/lib/pf2e/kingdom-activities.ts` — 51 activities (49 RAW incl. Army + 2 V&K), GENERATED
- `src/lib/pf2e/kingdom-structures.ts` — 74 structures, GENERATED, joined to their tiles
- `src/lib/dice.ts` — dice + PF2e degree of success, injectable rng
- `scripts/extract-kingdom-assets.py` — map sheets + 69 structure tiles → `public/kingdom/`
- `scripts/extract-kingdom-activities.py` / `extract-kingdom-structures.py` — catalog generators
- `src/lib/pf2e/kingdom.test.ts` / `kingdom-activities.test.ts` / `kingdom-structures.test.ts`
  / `src/lib/dice.test.ts` / `src/lib/hex.test.ts`
- `src/components/kingdom/founding-wizard.tsx` — the 8-step onboarding flow
- `src/components/kingdom/founding-parts.tsx` — ability crests, choice cards, step rail
- `src/app/api/kingdom/found/route.ts` — transactional commit of the founding
- `src/lib/hex.ts` + `src/lib/map-sheets.ts` (generated) — hex maths and sheet geometry
- `src/components/kingdom/kingdom-map.tsx` / `hex-map-canvas.tsx` / `hex-inspector.tsx`
- `src/app/api/kingdom/hexes/route.ts` — hex upsert; recounts kingdom Size
- `src/lib/urban-grid.ts` + `urban-grid.test.ts` — Urban Grid pure logic (block/lot
  placement, rubble, activation, level/overcrowded derivation)
- `src/components/kingdom/settlements-tab.tsx` / `urban-grid-editor.tsx` /
  `structure-picker.tsx`
- `src/app/api/kingdom/settlements/route.ts` + `[id]/route.ts` — found/list, grid mutations
- `src/lib/kingdom.ts` — `getOrCreateKingdom()` (seeds 16 skills + 8 roles)
- `src/app/api/kingdom/route.ts` — GET / PATCH scalar fields (whitelisted)
- `src/app/api/kingdom/leadership/route.ts` — PATCH one role (assign char/NPC, invest)
- `src/app/api/kingdom/skills/route.ts` — PATCH one skill's rank (0–4)
- `src/components/kingdom/*` — shell + overview / skills / leadership-roster / founding-choices
- `src/app/(app)/kingdom/page.tsx` — server component, replaces the placeholder
- **Delete kingdom** — `DELETE /api/kingdom` (type-to-confirm `DangerZone` in
  `founding-choices.tsx`). The name match is enforced server-side, not just in the UI; the
  cascade is enforced by Postgres FK constraints (`onDelete: Cascade` on all six
  Kingdom-owned models), not application code. Deleting just empties the Kingdom row —
  `getOrCreateKingdom()` recreates a fresh unfounded one on the next load, which is what
  sends the player back through the founding wizard.

## Notes for the next slice
- **All 13 migrations are applied in production** (confirmed via `npx prisma migrate status`
  in the container: "Database schema is up to date"). `prisma` is not on PATH there — it is
  a local package, so use `npx prisma ...` or `./node_modules/.bin/prisma ...`.
- **Kingdom Size is derived from the map.** `PATCH /api/kingdom/hexes` recounts claimed
  hexes and writes Size; the Overview shows it read-only and `PATCH /api/kingdom` refuses
  the field.
- **Both generated catalogs hide the same trap.** Structure and activity names contain
  commas and the word "and" ("tavern, popular", "Rest and Relax"), so upgrade paths and
  item-bonus targets cannot be split on punctuation — they are resolved by matching against
  the catalogs' own names. The guide also labels some structures "BUILDING n" rather than
  "STRUCTURE n", and Rubble's level is an em-dash; matching only the common form silently
  dropped six entries and merged their text into their neighbours.
- The wizard lets one character hold several leadership roles. The Player's Guide expects
  one role per PC, but this is a player helper rather than a rules enforcer — tighten it
  only if the table wants that.

## Open Questions
- Do the four Stolen Lands map sheets tile into one map, and in what arrangement? Could not
  be determined from the PDF. If the physical copy or a Foundry module settles it, the
  offsets drop straight into `public/kingdom/map/manifest.json`.
