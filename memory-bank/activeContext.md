# Active Context — KingdomTools

## Current Phase
**Phase 4 — Polish** ✅ COMPLETE

## Current Focus
Phases 0–4 complete. Phase 5 (Kingdom Management) is **on hold** until explicitly requested.

## Recent Decisions
- **Player helper, not GM tool** — no encounter rollers, NPC trackers, or hex logs
- **shadcn/ui uses @base-ui/react** (NOT Radix) — no `asChild` prop, Select.onValueChange is `(value: string | null, eventDetails) => void`
- **bcrypt dev fallback** — hash hardcoded in auth.ts because `$` conflicts with dotenv-expand
- **Phase 5 on hold** — Kingdom Management deferred indefinitely; only start when user explicitly asks
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

## Open Questions
- None currently
