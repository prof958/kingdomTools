# Tech Context — KingdomTools

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | Next.js (App Router) | 16.2.1 |
| Language | TypeScript | 5+ (strict mode) |
| UI Components | shadcn/ui (@base-ui/react — NOT Radix) | base-nova style |
| Styling | Tailwind CSS | v4 |
| Drag-and-Drop | @dnd-kit/core + @dnd-kit/sortable | latest |
| Canvas | react-konva (Konva.js) | latest |
| Client State | Zustand | latest (canvas only) |
| Database | PostgreSQL | 16 |
| ORM | Prisma | latest |
| Reverse Proxy | Caddy | 2 |
| Containerization | Docker + Docker Compose | latest |
| Testing | Vitest | latest |

## Development Setup
- Solo developer, Windows environment
- Repository: `kingdomTools` (Git)
- Package manager: npm
- Node.js 20+ LTS
- PostgreSQL 16.13 installed via `winget install PostgreSQL.PostgreSQL.16`
  - Service: `postgresql-x64-16` (Windows service, auto-start)
  - PG bin: `C:\Program Files\PostgreSQL\16\bin` (add to PATH each session)
  - Dev DB: `kingdomtools`, user: `kingdomtools`, password: `localdev`
  - Auth password: `kingmaker` (dev fallback hash hardcoded in `src/lib/auth.ts`; bcrypt `$` conflicts with dotenv-expand)
- PowerShell needs: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force`

## VPS (Production)
- Provider: Hetzner
- Specs: 4 vCPU, 8GB RAM, ~80GB SSD
- OS: Linux (Ubuntu/Debian)
- Domain: Custom domain via Spaceship DNS
- TLS: Auto-provisioned by Caddy (Let's Encrypt)

## Docker Compose Services
1. **app** — Next.js standalone build, port 3000
2. **db** — PostgreSQL 16 Alpine, port 5432 (internal only), named volume `pgdata`
3. **caddy** — Caddy 2 Alpine, ports 80/443, auto-TLS

## Environment Variables
| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | PostgreSQL connection string |
| `APP_PASSWORD_HASH` | bcrypt hash of the shared access password |
| `SESSION_SECRET` | Secret for signing session cookies |
| `NEXT_PUBLIC_APP_NAME` | Display name ("KingdomTools") |

## Key Technical Constraints
- `react-konva` requires `dynamic(() => import(...), { ssr: false })` in Next.js
- Prisma runs server-side only — never imported in client components
- Canvas components use Zustand for local state; everything else uses Server Components + API routes
- JSONB columns for flexible data (campsite layouts, kingdom logs)
- **shadcn/ui uses @base-ui/react** (NOT Radix): no `asChild` prop on DialogTrigger (use inline className), `Select.onValueChange` signature is `(value: string | null, eventDetails) => void` (wrap with `val ?? default`)
- Passing Prisma objects to client components requires `JSON.parse(JSON.stringify(...))` to strip non-serializable fields

## CI/CD Pipeline (Planned)
- GitHub Actions: lint → type-check → test → build → push Docker image
- Deploy: SSH into VPS → `docker compose pull && docker compose up -d`
- Prisma migrations: `prisma migrate deploy` runs on app container startup

## Backup Strategy
- Daily `pg_dump` via cron on VPS
- Optional: push to Hetzner Object Storage (S3-compatible)
