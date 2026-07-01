# cc_fishing docs — working notes

Resource-specific notes for editing this folder. The folder-wide editorial rules
(audience, voice, page shapes, components) live in [../CLAUDE.md](../CLAUDE.md) —
read that first. This file only records facts about *this* resource that the docs
depend on, so an edit doesn't drift from the shipped code.

Source of truth: [resources/[cc]/cc_fishing/](../../resources/[cc]/cc_fishing/).
Before publishing any config value, item, or command, open the source and confirm
it — earlier versions of these pages shipped claims the code never had (see traps
below).

## Pages

- `index.mdx` — what it is, feature overview, NPCs, dependencies.
- `installation.mdx` — place, items (ox + qb tabs), ensure order, verify.
- `configuration.mdx` — module toggles, NPC placement, payment sources, tunables, editable files.
- `exports.mdx` — read-only server exports (level/xp/zone).

## Facts the docs rely on

- **Module toggles exist.** `cfg.modules` gates `pawnshop_ped`, `anchor`,
  `boat_rentals`, `nets`, `treasure`, `logbook`, and `tournaments.{weekly,daily}`.
  Core fishing (fishing NPC, casting, selling fish) is always on and has no toggle;
  neither does bait digging (the shovel is not under the `treasure` toggle).
  Disabling `treasure` also stops new map drops but still lets players sell treasure
  loot they already hold.
- **Database auto-installs.** `cfg.autoInstallDatabase = true` + `server/db_init.lua`
  create the four tables on first start. The `sql/install.sql` import is the
  fallback for users whose DB account can't `CREATE TABLE` at runtime — not a
  mandatory step.
- **Tables (4):** `cc_fishing_player`, `cc_fishing_leaderboard`,
  `cc_fishing_tournaments` (weekly history + queued prizes), `cc_fishing_nets`.
- **The treasure minigame is swappable** in `client/minigames.lua` (escrow-ignored,
  fleeca pattern: returns a table of named wrappers, each returning a bool). The
  `treasure_safe` entry now **defaults to `return true`** (no minigame — safe opens
  on interact) because ox_lib `lib.skillCheck` cancels itself underwater, where
  safes are opened. Both `exports.cc_minigames:Safe` (works underwater) and
  `lib.skillCheck` (land-only) ship commented as opt-ins.
  `cfg.treasure.safe_difficulty` (`'easy'|'medium'|'hard'`) feeds either one.
- **`cc_minigames` is OPTIONAL**, not a default dependency. It's only needed if the
  owner switches `treasure_safe` to the commented `exports.cc_minigames:Safe`
  alternative in `client/minigames.lua`. It's not in `fxmanifest.lua` dependencies
  either way, so if switched it must be `ensure`d before `cc_fishing` manually.
- **Inventory images** go in the inventory's own images folder
  (`ox_inventory/web/images/`, `qb-inventory/html/images/`). No image pack ships
  inside cc_fishing. Item ids reuse legacy fishing names so old packs plug in.
- **Self-registered useable items:** `fishingrod*`, `treasure_map`, `logbook`
  register their own use-handlers server-side at boot — no `items.lua` wiring
  beyond the definition.
- **`cfg.debug`** only enables verbose server logging (`utils.dprint`). It does
  **not** bypass cooldowns. Don't claim otherwise.
- **`cfg.rod`** only has `low_durability` (a warning threshold). There is no
  `random_break` field.
- **Offline tournament prizes:** weekly winners are paid on next login (stored on
  the tournament's entries); daily winners forfeit if offline (live in-zone event).
- **`shared/*.lua` and `locales/*.json` are escrow-ignored** — customers can edit
  fish, zones, equipment, challenges, treasures, and locale freely.

## Traps that bit a previous revision

- Documented a `cfg.modules` block with 10 toggles — the real block (added later)
  has 6 flags + a nested `tournaments.{weekly,daily}`. Match the shipped shape, not
  the old invented one.
- Documented `cfg.rod.random_break` — never existed.
- Pointed at `web/inventory_images/` for an image pack — no such folder.
- Claimed `cfg.debug = true` bypasses cooldowns — it only logs.
- Listed the SQL import as a required step — it auto-installs.

If you add or rename a tunable, fish, item, or command, update the matching page
*and* this list in the same change.
