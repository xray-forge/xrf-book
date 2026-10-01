# sr_psy_antenna

`sr_psy_antenna` applies psy-zone effects while the actor is inside a restrictor. It adjusts the shared
`PsyAntennaManager`, enables fake HUD indicators, optionally starts a postprocess effector, and restores the manager
values when the actor leaves.

## Parameters

### `eff_intensity`

Type: number. Required. Default: none.

Sound/postprocess intensity. Multiplied by `0.01`.

### `postprocess`

Type: string. Optional. Default: `psy_antenna`.

Postprocess effector name. Use `nil` to skip adding an effector.

### `hit_intensity`

Type: number. Required. Default: none.

Hit intensity added to the psy antenna manager. Multiplied by `0.01`.

### `phantom_prob`

Type: number. Optional. Default: `0`.

Phantom spawn probability. Multiplied by `0.01`.

### `mute_sound_threshold`

Type: number. Optional. Default: `0`.

Added to the manager mute threshold while inside.

### `no_static`

Type: boolean. Optional. Default: `false`.

Sets the manager `noStatic` flag.

### `no_mumble`

Type: boolean. Optional. Default: `false`.

Sets the manager `noMumble` flag.

### `hit_type`

Type: string. Optional. Default: `wound`.

`chemical` hits with a chemical burn, any other value with a telepathic hit.

### `hit_freq`

Type: number. Optional. Default: `5000`.

Hit frequency used by the manager.

The section also supports common switch fields.

## Behavior

When the actor enters the zone, the manager:

- enables fake HUD indicators;
- adds intensity, hit intensity, mute threshold, and phantom probability to the shared psy manager;
- copies `no_static`, `no_mumble`, `hit_type`, and `hit_freq` to the shared manager;
- starts the configured postprocess if it is not `nil`.

When the actor leaves or the scheme deactivates, those additive values are subtracted. Fake indicators turn off once the
actor is in no psy antenna zone.

The manager saves its inside/outside state in portable storage under key `inside`.

## Example

```ini
[logic]
active = sr_psy_antenna@lab

[sr_psy_antenna@lab]
eff_intensity = 60
hit_intensity = 10
phantom_prob = 5
mute_sound_threshold = 0.2
postprocess = psy_antenna
hit_freq = 3000
on_actor_outside = sr_idle@outside
```

## Notes

- Percent-style fields are multiplied by `0.01`.
- Multiple active psy antenna zones add to the shared manager values.
- `postprocess = nil` disables postprocess creation for this zone.
