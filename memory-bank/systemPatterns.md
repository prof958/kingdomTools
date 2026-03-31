# System Patterns — KingdomTools

## Architecture
Single Next.js 14+ monolith with PostgreSQL backend, deployed via Docker Compose.

```
[Browser] → [Caddy :443 (TLS)] → [Next.js :3000]
                                        ↓
                                 [Server Components]  ←→  [Prisma ORM]  ←→  [PostgreSQL :5432]
                                 [API Routes /api/*]  ←→  [Prisma ORM]  ←→  [PostgreSQL :5432]
                                 [Client Components]  ←→  [Zustand (canvas state only)]
```

## Route Structure (App Router)
```
app/
├── (auth)/
│   └── login/page.tsx          # Password gate
├── (app)/
│   ├── layout.tsx              # Tab navigation shell
│   ├── dashboard/page.tsx      # Landing: quests, wealth, links
│   ├── inventory/page.tsx      # Group inventory + player assignment
│   ├── campsite/page.tsx       # Canvas editor + activities + watches
│   └── kingdom/page.tsx        # Placeholder → Phase 5
├── api/
│   ├── inventory/route.ts      # CRUD for inventory items
│   ├── objectives/route.ts     # CRUD for quests
│   ├── campsite/route.ts       # Save/load campsite layouts
│   ├── wallet/route.ts         # Wallet operations
│   └── auth/route.ts           # Login/logout
└── layout.tsx                  # Root layout
```

## Key Technical Patterns

### Server vs Client Components
- **Server Components** (default): All data-fetching pages (Dashboard, Inventory list, Kingdom overview)
- **Client Components** (`'use client'`): Only where interactivity requires it — campsite canvas, drag-and-drop, inline editing, forms
- Prisma runs **server-side only** (in Server Components and API routes)

### PF2e Rule Engine (`lib/pf2e/`)
- Pure TypeScript functions, **stateless and tested**
- Takes data in, returns computed results — decoupled from UI and DB
- Modules: `bulk.ts`, `currency.ts`, `investment.ts`, `camping.ts`, `kingdom.ts` (Phase 5)

### Data Flow
1. Server Component fetches data via Prisma
2. Renders initial HTML (fast first paint)
3. Client Components hydrate for interactivity
4. Mutations go through API routes → Prisma → DB
5. `router.refresh()` or `revalidatePath()` to update server data after mutations

### Canvas Components (Campsite, future Hex Grid)
- `react-konva` for 2D canvas rendering
- Must use `dynamic(() => import(...), { ssr: false })` — canvas is client-only
- Canvas state managed via Zustand store (positions, selections, zoom)
- Saved to DB as JSONB (flexible schema for evolving layouts)

### Inventory Model
- `InventoryItem.character_id` nullable: `null` = shared party loot, set = assigned to player
- Self-referencing FK (`container_inventory_item_id`) for items inside containers
- Bulk calculation runs client-side via `lib/pf2e/bulk.ts` using fetched data

### Authentication
- Simple password gate via Next.js middleware
- Hashed password stored in env var
- Session cookie set on successful login
- Middleware checks cookie on all `/(app)` routes

### JSONB Usage
- Campsite layout element positions
- Kingdom turn phase logs (Phase 5)
- Structure effects (Phase 5)
- Flexible enough for schema evolution, structured enough for queries
