# Scripts

The XRF scripting layer is the TypeScript runtime that is compiled to Lua and loaded by the X-Ray engine. It owns the
Lua extern modules, object binders, scheme registry, global managers, server object classes, and shared helpers used by
configs and gameplay logic.

For object lifecycle, manager startup, events, and save/load, start with [Runtime lifecycle](runtime_lifecycle.md). This
page is the lower-level source map for script modules.

The main source roots are:

| Area                   | Source                      |
| ---------------------- | --------------------------- |
| Lua entry points       | `src/engine/scripts`        |
| Runtime binders        | `src/engine/core/binders`   |
| Global managers        | `src/engine/core/managers`  |
| Runtime registry       | `src/engine/core/database`  |
| Scheme implementations | `src/engine/core/schemes`   |
| Server object classes  | `src/engine/core/objects`   |
| Shared helpers         | `src/engine/core/utils`     |
| UI classes             | `src/engine/core/ui`        |
| Animation tables       | `src/engine/core/animation` |

## Entry points

The engine-facing entry points are registered with `extern(...)`.

### `_g`

Loads global runtime declarations. This is the global bridge used before other modules are available.

### `register`

Exposes class-id and class registration helpers:

- `register.registerGameClasses`
- `register.getGameClassId`
- `register.getUiClassId`

The X-Ray engine calls these functions while linking script classes to engine class ids.

### `bind`

Exposes binder factories such as `bind.actor`, `bind.stalker`, `bind.restrictor`, `bind.weapon`, and
`bind.smart_terrain`. Each function receives a game object and attaches the matching `object_binder` implementation.

Some binders are conditional. For example, helicopters and physical objects are bound only when their spawn ini or
object kind needs script logic.

### `start`

Runs the game-start callback. It updates class ids, registers the simulator and ranks, unlocks system ini overriding,
registers managers, registers schemes, registers extensions, and emits `GAME_STARTED`.

## Runtime shape

Most gameplay code is not called directly from config files. The usual path is:

1. X-Ray loads script entry modules.
2. `start.callback` initializes shared runtime systems.
3. X-Ray creates online objects and calls a `bind.*` function.
4. The binder registers the object in `registry`.
5. The binder initializes object logic from spawn ini or generated config.
6. Schemes, managers, and events update behavior on object or actor ticks.

Server-side classes such as squads and smart terrains participate through `on_register`, `on_unregister`, `STATE_Write`,
and `STATE_Read`.

## What to edit

- Add new engine callbacks under `src/engine/scripts/declarations`.
- Add new object lifecycle behavior under `src/engine/core/binders`.
- Add cross-object systems under `src/engine/core/managers`.
- Add runtime object registries under `src/engine/core/database` only when the state is shared across modules.
- Add shared helpers under `src/engine/core/utils` when they are stateless or narrowly scoped.

## Validation

For script runtime changes, start with focused tests near the changed module. Use broader checks when touching shared
runtime contracts:

```powershell
npm test -- src/engine/scripts
npm test -- src/engine/core/database
npm run typecheck
```
