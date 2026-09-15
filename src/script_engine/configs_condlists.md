# Condlists

Condlists are conditional config expressions used throughout script configs, task configs, dialogs, and scheme switches.
A condlist can check info portions, call conditions, apply side effects, and select a value.

Basic shape:

```ltx
{conditions} value %effects%
```

Multiple entries are separated by commas and checked in order. The first matching entry wins:

```ltx
on_info = {+quest_started} walker@active %=play_sound(quest_start)%, sr_idle
```

## Conditions

| Syntax            | Meaning                                                  |
| ----------------- | -------------------------------------------------------- |
| `+info_name`      | Require an info portion.                                 |
| `-info_name`      | Require a missing info portion.                          |
| `=condition_name` | Call `xr_conditions.condition_name` and require `true`.  |
| `!condition_name` | Call `xr_conditions.condition_name` and require `false`. |
| `~50`             | Pass with a random chance from 1 to 100.                 |

Function parameters use colon separators:

```ltx
{=actor_has_item(af_oasis_heart)}
{!npc_in_actor_frustum =dist_to_actor_le(30)}
```

## Effects

Effects live inside `%...%`:

```ltx
%=give_task(jup_b1_task) +jup_b1_started -jup_b1_waiting%
```

Inside effects:

- `=effect_name` calls `xr_effects.effect_name`;
- `+info_name` gives an info portion;
- `-info_name` disables an info portion.

This means reading a config value can change game state. Check the caller before assuming a condlist is pure.

## Generated condlists

Generated LTX sources should use helpers from `cli/utils/ltx/condlist.ts`:

- `checkCondition(...)`
- `checkNoCondition(...)`
- `checkChance(...)`
- `checkHasInfo(...)`
- `checkNoInfo(...)`
- `callEffect(...)`
- `addInfo(...)`
- `removeInfo(...)`
- `createCondlist(...)`
- `joinCondlists(...)`

These helpers keep generated syntax consistent with the runtime parser.

## Parser limits

The parser is pattern-based. Avoid nested commas, nested parentheses, and quoted strings that require custom escaping.
If new syntax is needed, update parser tests before relying on it in configs.

## Debugging workflow

When a condlist does not behave as expected:

1. identify the caller, such as a scheme switch, task field, dialog phrase, or smart terrain job;
2. check whether the caller expects a returned section/value or only side effects;
3. search for the condition under `src/engine/declarations/conditions`;
4. search for the effect under `src/engine/declarations/effects`;
5. add or update tests for parser helpers when generated syntax changes.

Keep side effects guarded with info portions when the same condlist can be evaluated repeatedly.
