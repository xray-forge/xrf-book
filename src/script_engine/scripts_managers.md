# Script managers

Managers are singleton runtime services stored in `registry.managers`. They own cross-object systems such as events,
save/load, sound, simulation, trade, tasks, map spots, weather, upgrades, and debug state.

## Registration

`registerManagers()` initializes the manager list during `start.callback`.

The current startup list includes:

- actor input and actor inventory menu;
- database, debug, and events;
- dialogs, load screen, loadout, map display, music, notifications, PDA;
- phantom, save, simulation, sleep, sound, statistics;
- tasks, trade, travel, treasures, upgrades, weather;
- body release handling.

Each manager is initialized through `initializeManager(ManagerClass)`.

Other managers can be initialized lazily through `getManager`. For the startup list and lifecycle rules, see the Runtime
managers page.

## Manager registry

Manager instances are stored in two maps:

- `registry.managers`, keyed by class reference;
- `registry.managersByName`, keyed by class name.

Use `getManager(ManagerClass)` for normal access. It initializes the manager if needed.

Use `getWeakManager(ManagerClass)` only when missing manager state is acceptable.

Use `getManagerByName(name)` only for circular-reference cases where the class reference is not available.

## Lifecycle

Managers extend `AbstractManager`. The base class defines:

- `initialize()`;
- `destroy()`;
- `update(delta)`;
- `save(packet)`;
- `load(reader)`.

`update`, `save`, and `load` abort by default. A manager should implement only the lifecycle methods it actually uses.

`disposeManager` calls `destroy()`, marks the instance as destroyed, and removes it from both registry maps.

## Common patterns

Managers often subscribe to `EventsManager` in `initialize()` and unsubscribe in `destroy()`.

Examples:

- `SoundManager` listens for actor update/offline and debug dump events.
- `SimulationManager` listens for actor registration and actor offline events.
- `SaveManager` coordinates client/server save callbacks exposed through `alife_storage_manager`.

Managers with persistent state write to net packets or dynamic save data. When editing save/load logic, keep marker
order and read/write order synchronized.

## Guidelines

- Put shared system state in a manager, not in a binder.
- Keep object-local state in the object's registry state or binder.
- Register manager callbacks in `initialize()` and unregister them in `destroy()`.
- Avoid constructing managers directly; use `getManager` unless a test is isolating the class.
