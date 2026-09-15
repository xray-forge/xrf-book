# Callbacks and events

Callbacks are the bridge from X-Ray and Lua config scripts into the TypeScript runtime. Events are XRF's internal
publish-subscribe layer for sharing lifecycle changes between managers, binders, and schemes.

## External callbacks

External callbacks live under `src/engine/declarations/callbacks` and are loaded by `registerExternals()`.

| Module                                                                                                | Examples                                                 |
| ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `on_actor_*.ts`, `travel_callbacks.ts`                                                                | Actor condition notifications and travel dialogs.        |
| `alife_storage_manager.ts`, `level_input.ts`, `visual_memory_manager.ts`                              | Save/load, input, and visual memory.                     |
| `loadscreen.ts`, `inventory_upgrades.ts`, `actor_menu*.ts`, `pda.ts`, `ui_wpn_params.ts`              | Engine-facing UI callbacks.                              |
| `on_*sleep*.ts`, `surge_survive_*.ts`, `check_achievement.ts`, `is_task_*.ts`, `effector_callback.ts` | Sleep, surge, achievement, task, and cutscene callbacks. |

The declarations use `extern(name, value)` to register global functions or modules. Config files and engine code call
those names from Lua.

Other callbacks include `trade_manager.ts`, `ai_stalker.ts` for loadout, `outro.ts`, and `on_unregister.ts`.

## Binder callbacks

Object binders register engine callbacks on online objects.

For example, `ActorBinder` registers callbacks for inventory info, item take/drop, trade, task state, use object, and
HUD animation end. It converts those callbacks to `EGameEvent` emissions.

`StalkerBinder` registers hit, death, use, sound, and patrol extrapolate callbacks. These callbacks forward work to
scheme events, managers, and global events such as `STALKER_HIT` or `STALKER_DEATH`.

## EventsManager

`EventsManager` is the internal event dispatcher. It stores a Lua table of subscribers for every declared `EGameEvent`.

Use:

```ts
getManager(EventsManager).registerCallback(EGameEvent.ACTOR_UPDATE, this.onActorUpdate, this);
EventsManager.emitEvent(EGameEvent.GAME_STARTED, isNewGame);
```

Callbacks can be registered with or without an explicit context. When a context is provided, the manager calls the
callback with that context.

## Timers

`EventsManager` extends `AbstractTimersManager`, so it also owns game-time intervals and timeouts.

- `registerGameInterval(callback, period)` repeats after at least `period` milliseconds.
- `registerGameTimeout(callback, delay)` runs once after `delay` milliseconds.
- intervals assert that the period is at least `50`.
- timers are processed from `ActorBinder.update()` through `eventsManager.tick()`.

## Event groups

`EGameEvent` includes events for:

- actor lifecycle and throttled actor update ticks;
- stalker, monster, helicopter, item, zone, smart terrain, and squad lifecycle;
- task, treasure, surge, notification, and hit events;
- save/load and level-change events;
- UI events such as main menu on/off;
- debug dump requests.

## Guidelines

- Use external callbacks only for names the engine or config files call directly.
- Use `EventsManager` for internal cross-system notifications.
- Unregister callbacks in `destroy()` or binder cleanup paths when the owner can be disposed.
- Do not put long-running work inside high-frequency actor update events unless it is explicitly throttled.
