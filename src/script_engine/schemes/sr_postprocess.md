# sr_postprocess

`sr_postprocess` applies a gray/noise postprocess effect while the actor is inside a restrictor and applies periodic
radiation and shock hits.

Use it for hazardous visual zones where the actor should see an effect and take damage while inside.

## Parameters

### `intensity`

Type: number. Required. Default: none.

Target postprocess intensity. The value is multiplied by `0.01`.

### `intensity_speed`

Type: number. Required. Default: none.

Ramp speed for entering and leaving the zone. The value is multiplied by `0.01`.

### `hit_intensity`

Type: number. Required. Default: none.

Damage accumulation rate while the actor is inside.

Common switch fields are parsed, but switching away calls `PostProcessController.deactivate()`, which currently aborts.
Keep this section active; leaving the zone already ramps the effect down and stops damage accumulation.

## Behavior

On activation, the manager starts a postprocess effector with id `object.id() + 2000`. Each update:

- checks common switch conditions first;
- tests whether the actor is inside the restrictor;
- ramps intensity toward the target when inside and back toward zero when outside;
- updates gray color and noise parameters;
- accumulates hit power while inside;
- once per second, applies radiation and shock hits to the actor.

`intensity` and `intensity_speed` are converted from percent-style values by multiplying by `0.01`. `hit_intensity` is
used directly as the per-second accumulation rate.

## Example

```ini
[logic]
active = sr_postprocess@hazard

[sr_postprocess@hazard]
intensity = 40
intensity_speed = 8
hit_intensity = 0.02
```

## Notes

- `intensity` and `intensity_speed` are percent-style config values.
- The hit direction is zero and impulse is `0`.
