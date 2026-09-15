# Effects and conditions

Effects and conditions are Lua externals called from config condlists. They connect LTX logic to XRF TypeScript
behavior.

Effects are registered under `xr_effects`. Conditions are registered under `xr_conditions`.

## Source layout

| Source area                                            | Purpose                                                          |
| ------------------------------------------------------ | ---------------------------------------------------------------- |
| `src/engine/declarations/effects`                      | Effect functions called from `%...%` condlist actions.           |
| `src/engine/declarations/conditions`                   | Boolean condition functions called from `{...}` condlist checks. |
| `src/engine/scripts/register/externals_registrator.ts` | Loads declaration modules and prevents duplicate registration.   |
| `xrf-xray16-sdk/src/lib/utils/binding.ts`              | Implements `extern(...)`, imported from `xray16/lib`.            |
| `src/engine/core/ini`                                  | Runtime condlist parsing and execution.                          |

## Config names

Configs call short names:

```ltx
on_info = {=actor_has_item(af_oasis_heart)} %=give_task(jup_b16_task)%
```

The registered globals include the namespace:

- `actor_has_item` resolves to `xr_conditions.actor_has_item`;
- `give_task` resolves to `xr_effects.give_task`.

Search both names before changing an effect or condition.

## Function shape

Effect and condition declarations commonly receive the actor object, the current object, and a parameter array parsed
from the condlist:

```ltx
%=play_sound(story_sound_id)%
{=dist_to_actor_le(30)}
```

Parameters in configs are colon-separated. Keep parsing simple and update parser tests before adding syntax that needs
nested values or escaping.

## Side effects

Conditions should answer a question. Effects may mutate game state: give or remove info portions, start tasks, play
sounds, set weather, spawn objects, save the game, or switch object state.

`pickSectionFromCondList` can run effects while choosing a section. A field that looks like a value read may still
change state if its matching condlist entry contains `%...%`.

## Testing

Effect and condition files have focused Jest tests beside the declarations. When changing behavior, update the matching
test:

```powershell
npm test -- src/engine/declarations/effects
npm test -- src/engine/declarations/conditions
```

For config changes that call the function, also run:

```powershell
npm run cli verify ltx
```
