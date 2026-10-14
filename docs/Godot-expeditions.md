# Godot native expedition port

`GameExpedition` in `scripts/game_expedition.gd` is a native GDScript port of the untouched `legacy-v03/src/expedition.ts`. It uses the exported route, encounter, boon, battle-version, and resource-cap data in `data/game-data.json`. It does not load JavaScript, use a WebView, or write save files.

## Transaction contract

`GameExpedition.command(state, name, args = {}, now = -1)` mutates the supplied draft and returns `{ ok: bool, error: String, data: Variant }`. The economy/store owns draft creation, whole-state validation, disk commit, and rollback; callers must not run mutation commands against an authoritative state before a successful transaction.

| Command | Arguments | Successful `data` |
|---|---|---|
| `startExpedition` | `{ routeId, options?: {id?, seed?, now?} }` | Deep copy of the locked run |
| `chooseBoon` | `{ id }` | Selected boon ID |
| `prepareExpeditionBattle` | `{}` | `{battleId, stageId, team, options}` |
| `settleExpeditionBattle` | `{ battle }` | `{duplicate, won, node}` |
| `closeExpedition` | `{runId?, reason?}` | `{id, routeId, complete, first, reward, reason, duplicate}` |
| `isRouteUnlocked` | `{routeId}` | Boolean |
| `expeditionCompatibility` | `{}` | `{compatible, message}` |
| `expeditionCandidates` | `{}` | Candidate ID array |

The equivalent mutation helpers are `start_expedition`, `choose_boon`, `prepare_expedition_battle`, `settle_expedition_battle`, and `close_expedition`; they return the same result envelope. Read helpers are `routes`, `route`, `is_route_unlocked`, `legal_boons`, `expedition_compatibility`, and `expedition_candidates`. `validate_expedition_ledger(value, state)` returns an empty string on success or a readable error on failure.

`startExpedition` also accepts `id` and `seed` directly in `args`; values inside `options` take precedence. An explicit command `now` populates `options.now` only if it is absent. Run IDs otherwise use operating-system randomness, independent of gameplay streams.

## Rules and saved schema

- All persisted keys retain their camelCase TypeScript names, including `selectedBoons`, `secondByFirst`, `startedBattles`, `completedBattles`, `nodeRewards`, `firstCleared`, and `bossClears`. Numeric node keys are JSON-compatible strings (`"1"`, `"3"`, `"5"`).
- 丹火遗境 unlocks after mainline node 8; 雷泽残境 unlocks after node 12. Neither requires finishing the thunder project. The project-specific `thunderward` method still requires the teaching direction.
- Each route contains battle → choice → battle → choice → boss, followed by node 6 / `complete` pending final settlement.
- Teams, levels, branches, branch-unlock flags, treasures, method, pill, enemies, rule version, node rewards, completion reward, first reward, and all seeds/candidate lists are snapshotted at entry.
- Boons carry across later nodes; enemy and ally HP, status, and cooldown state do not carry across nodes. Each preparation request creates a fresh `GameCombat` battle from the locked team. Rebuilding the same request after a reload/retry also starts that battle fresh, with the same seed and loadout.
- An explicit seed reproduces the reference 32-bit xorshift/Fisher–Yates order exactly without touching saved random streams. Normal starts advance the expedition stream once and the battle stream three times; summon randomness is unaffected.
- Both first and second three-option choices are saved at entry. Every possible second set excludes the chosen first boon. `clarity` requires a cleansing method or a genuinely cleansing team member (太乙, 比干, or 玄都 with a restored treasure).
- The first preparation of each battle ID spends one snapshotted pill. Resume/retry does not spend another. A new node requires another pill of that same type. Spent pills are never refunded on exit.
- A mainline battle and expedition cannot coexist. Started tickets must be settled against the current route/node. Settlement validates stage, seed, rule version, non-trial origin, exact ordered ally loadout, method/pill/boons, exact ordered enemy IDs, completed result, and a defeated enemy group for a win.
- Node settlement accumulates pending reward and advances daily battle progress once. The node-5 victory records the boss clear. No resources are credited until `closeExpedition` succeeds in the owning transaction.
- Full closure adds completion rewards, plus first-clear rewards only once per route, and obeys resource caps. Node and run receipts prevent duplicate credit. Partial/failure/old-rule closure credits only completed node rewards; it never adds unearned boss, completion, or first-clear progress.
- Structurally valid old-rule ledgers remain loadable/exportable and may close earned rewards. New boon selections or battles are blocked. Legacy runs without optional `nodeRewards` use the reference node-reward fallback.

### Exact complete-route rewards

| Route | First full clear (stone / xp / material / herb) | Later full clears |
|---|---|---|
| 丹火遗境 | 390 / 315 / 29 / 18 | 310 / 235 / 24 / 8 |
| 雷泽残境 | 500 / 390 / 38 / 23 | 400 / 290 / 32 / 11 |

Ticket and fate rewards are always zero. Returned nominal rewards can exceed the actual credit only when a resource was already near its cap, matching the reference.

## Validation and tests

`tests/test_expedition.gd` is a headless `SceneTree` test runner. It covers both full routes, exact rewards, progression, deferred credit, duplicate node/run receipts, saved-choice stability, saved-stream determinism, exact TypeScript golden seed parity (0, 1, 91, 20261006, and 4294967295), JSON replay, locked external loadout changes, per-node pill use, failure/retry, early exit, forged tickets, legal-boon requirements, old-version closure, malformed ledgers, legacy reward fallback, caps, and repeated clears.

Settlement unit tests deliberately use battle-ticket fixtures; those assertions validate expedition rules. The same runner also executes 18 real `GameCombat` victories across both routes with three different level-11 gift teams (杨戬、赵公明、云霄), reloading JSON between nodes and verifying fresh HP/cooldowns, boon carryover, and final settlement. Two actual guard-only defeats verify identical retries and no repeated pill charge. These are native headless logic tests, not UI, browser, mobile-device, or performance acceptance.

Run from the project directory:

```sh
HOME="$PWD/../runtime-home" \
XDG_CONFIG_HOME="$PWD/../runtime-home/config" \
XDG_CACHE_HOME="$PWD/../runtime-home/cache" \
XDG_DATA_HOME="$PWD/../runtime-home/data" \
/opt/godot/4.6.3/Godot_v4.6.3-stable_linux.x86_64 \
  --headless --path . --script tests/test_expedition.gd
```

Godot parses JSON numbers as floats; numeric membership tests explicitly normalize them. That matters for both fresh native state and restored/imported JSON state.
