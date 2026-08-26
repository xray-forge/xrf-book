# Game scripting

This chapter documents the XRF gameplay layer: TypeScript scripts, generated Lua, LTX/XML configs, forms, translations,
schemes, managers, binders, and gameplay data.

The engine starts Lua and calls XRF entry points. XRF then owns most gameplay behavior above the native X-Ray runtime.

## Source layout

| Source area               | Purpose                                                               |
| ------------------------- | --------------------------------------------------------------------- |
| `src/engine/scripts`      | Lua script entry points and externally callable script declarations.  |
| `src/engine/core`         | Runtime managers, binders, schemes, objects, utils, and domain logic. |
| `src/engine/configs`      | Static and generated LTX/XML game configuration source.               |
| `src/engine/forms`        | JSX source for generated UI XML forms.                                |
| `src/engine/translations` | Translation JSON and XML source.                                      |
| `src/resources`           | Static resource data copied into gamedata output.                     |

Generated Lua, generated configs, copied resources, coverage, and packed outputs are written under `target/`. Do not
edit `target/` by hand.

## Runtime entry points

The script layer starts from `_g.script`, then loads `register`, `bind`, and `start`.

- `register` exposes game classes, UI classes, tasks, dialogs, conditions, effects, and callbacks.
- `bind` selects object binder classes for online engine objects.
- `start` initializes managers, schemes, extensions, simulation state, and emits `GAME_STARTED`.

Most gameplay work is then handled by managers, schemes, object binders, and event callbacks.

## Config-driven behavior

XRF keeps vanilla-style config behavior where possible. LTX logic sections still drive object logic, condlists still
call conditions and effects, and XML files still define dialogs, tasks, character descriptions, UI forms, and gameplay
data.

When changing behavior, trace the full path:

1. config field or XML entry;
2. parser or build helper;
3. runtime manager, scheme, binder, or extern;
4. test or validation command.

For config changes, compare against original gamedata when baseline behavior matters.

## Validation

Use focused validation first:

```powershell
npm test -- <path-or-pattern>
npm run typecheck
npm run cli verify ltx
npm run cli -- build --include configs --filter <pattern>
```

Use broader `npm run verify` or `npm run build` when a change touches shared config, generated output, or multiple
runtime systems.
