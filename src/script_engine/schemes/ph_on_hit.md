# ph_on_hit

`ph_on_hit` switches a physical object when it receives a hit callback. Use it for breakable or reactive props where the
next section should be chosen only after damage is applied.

## Parameters

`ph_on_hit` has no scheme-specific fields.

The section supports common switch fields such as `on_info`, `on_timer`, and zone or distance checks. They are evaluated
from the hit callback, not from a normal per-frame update.

## Behavior

The manager subscribes to physical object hit events. When the object is hit and the object still has an active scheme,
it calls the common section-switching logic for the current `ph_on_hit` state.

Hit amount, direction, attacker, and bone index are received by the callback, but the current implementation only logs
the object name, bone index, and hit amount. The switch conditions decide what happens next.

## Runtime sequence

1. `SchemePhysicalOnHit.activate` parses common switch conditions.
2. `SchemePhysicalOnHit.add` stores a `PhysicalOnHitManager` action and subscribes it.
3. `PhysicalOnHitManager.onHit` logs object name, bone index, and hit amount.
4. If the object still has an active scheme, the manager calls `trySwitchToAnotherSection`.
5. `SchemePhysicalOnHit.disable` unsubscribes the stored action when the state exists.

The callback receives hit direction and attacker, but those values are not written to scheme state by the current
implementation.

## Example

```ini
[logic]
active = ph_on_hit@crate

[ph_on_hit@crate]
on_info = ph_idle@damaged %+crate_was_hit%
```

## Notes

- The scheme is event-driven. Without a hit callback, its switch fields are not checked by this manager.
- `disable` unsubscribes the stored manager action when the scheme state exists.
- Use `ph_idle` bone-hit condlists when the response depends on a specific physical bone; `ph_on_hit` treats all hits
  the same.
