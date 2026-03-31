# Product Context — KingdomTools

## Why This Project Exists
Running the Kingmaker Adventure Path in PF2e generates significant logistical overhead for players:
- Tracking shared loot across sessions is error-prone with spreadsheets or paper
- PF2e's bulk/encumbrance system requires constant math that slows gameplay
- Kingmaker's unique campsite subsystem (activities, watches, cooking) has many moving parts
- Kingdom building (future) involves complex resource management and spatial planning

Foundry VTT handles the tactical side of play, but doesn't solve the **campaign logistics** problem. Players need a persistent, always-accessible tool between sessions.

## Problems It Solves
1. **"Who has what?"** — Centralized inventory with clear ownership (shared vs. assigned)
2. **"Are we encumbered?"** — Automatic bulk calculation per player based on STR modifier
3. **"What do we do at camp?"** — Visual campsite planner with activity/watch management
4. **"How much gold do we have?"** — Party wealth summary across all wallets
5. **"What were we doing?"** — Active quest tracker keeps objectives visible
6. **"What should we buy in town?"** — Shopping wish list with price tracking

## How It Should Work
- Open browser → enter shared password → see Dashboard
- Navigate via tabs: Dashboard | Inventory | Campsite | Kingdom
- Any player can add/edit data — it's a shared tool, no role-based permissions
- Changes persist immediately (server-side saves)
- Works on desktop and tablet (mobile is secondary but functional)

## User Experience Goals
- **Fast** — page loads under 2s, interactions feel instant
- **Simple** — no learning curve; if you can use a spreadsheet, you can use this
- **Reliable** — data never lost; daily backups
- **Accessible** — works on any modern browser, no install required
- **PF2e-native** — bulk, coins, and item mechanics work correctly out of the box
