# Script configs

Script configs are LTX files that drive object logic. They live under `src/engine/configs/scripts` and are copied to
`target/gamedata/configs/scripts`.

These files are active gameplay data. They select schemes, configure object binders, define section switches, call
conditions and effects, and describe smart terrain jobs.

## Common structure

Most logic files start from `[logic]`:

```ltx
[logic]
active = sr_idle

[sr_idle]
on_info = {+some_info} sr_idle@done %=some_effect%

[sr_idle@done]
```

The active section name selects the scheme. Suffixes such as `@done` let one scheme have multiple named states.

## Common fields

Common script logic fields include:

- `active` in `[logic]`, selecting the first section;
- `on_info`, `on_signal`, `on_timer`, and related switch fields;
- `on_actor_inside`, `on_actor_outside`, and zone-related switch fields;
- `on_death` and `on_hit` for event-driven switches;
- `suitable` and `prior` for smart terrain job selection;
- `spawn` for item sections spawned on activation;
- scheme-specific fields such as paths, animations, sounds, dialogs, and combat flags.

Field support depends on the active scheme. Check the scheme implementation before adding a field.

## Extern calls

Condlists call short names, but the registered functions live under global namespaces:

- `{=actor_has_item(wpn_pm)}` calls `xr_conditions.actor_has_item`;
- `%=give_task(task_id)%` calls `xr_effects.give_task`.

Search both the short config name and the full extern name before renaming a condition or effect.

## Validation

Use LTX validation after editing script configs:

```powershell
npm run cli verify ltx
```

For behavior changes, test the scheme, condition, effect, or manager that reads the field.

## Editing workflow

Start from the object, smart terrain job, or restrictor that owns the logic file. Follow `[logic] active` to the active
section, then identify the scheme from the section prefix before `@`.

When a switch does not fire, check the pieces in this order:

1. the active section name;
2. the switch field supported by that scheme;
3. the condlist conditions and effects;
4. the target section existence;
5. any runtime event needed to trigger the manager, such as hit, death, use, signal, or actor-zone update.
