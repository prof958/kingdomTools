# Project Brief — KingdomTools

## Overview
KingdomTools is a web-based **player-focused** TTRPG campaign helper for the Pathfinder 2nd Edition (PF2e) Kingmaker Adventure Path. It serves as a centralized, interactive hub to reduce logistics friction during and between sessions.

## Core Identity
- **Player Helper**, not a GM tool — all features serve players directly
- Shared group tool behind a simple password gate (no individual accounts)
- Complements Foundry VTT (which handles tactical/GM-side play)

## Core Feature Requirements (Tab-Based Interface)

### Dashboard
Central landing page with:
- Active quests / objectives tracker (player-managed)
- Party wealth summary (total across shared + individual wallets)
- Customizable quick-reference links (rules, AP resources, house rules)
- Kingdom turn reminder (Phase 5 future-proofing)

### Inventory Management
- **Group-first** shared inventory as the default view
- Items can optionally be assigned to individual players
- PF2e bulk/encumbrance calculation per player (using STR modifier)
- Container support (items inside backpacks, etc.)
- Party treasury + individual wallets (CP/SP/GP/PP)
- Loot splitting calculator
- Invested item tracking

### Campsite Planner
- Interactive drag-and-drop canvas for campsite layout (tents, fire, PCs, traps)
- Camping activity picker per character
- Watch order / shift assignment
- Recipe book for discovered campsite meals

### Kingdom Management (Phase 5 — Future)
- Hex grid, kingdom stats, settlements, structures
- Kingdom turn tracker with phase management
- Leadership roles linked to party characters
- Resource and commodity tracking

## Characters (Lightweight)
- Name + STR modifier only
- No detailed stats — just enough for bulk calculation and campsite/inventory assignment

## Constraints
- Single play group (4-6 players), single campaign
- Hosted on Hetzner VPS (4 vCPU, 8GB RAM, ~80GB SSD)
- Custom domain via Spaceship DNS
- Solo developer, JS/TS ecosystem
- Session notes handled externally (not in-app)

## Enhancement Features
- Shopping wish list (want-to-buy tracker with price awareness)
- Recipe book (Kingmaker campsite cooking subsystem)
- Loot splitting calculator
