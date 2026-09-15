# ph_on_death

`ph_on_death` switches a physical object when it receives a death callback. Use it for scripted reactions to destroyed
physics objects.

## Parameters

`ph_on_death` has no scheme-specific fields.

The section supports common switch fields. They are evaluated from the death callback.

## Behavior

The manager subscribes to physical object death events. When the object dies and the object still has an active scheme,
it calls the common section-switching logic for the current `ph_on_death` state.

The death callback receives the dead object and optional killer object. The current manager does not inspect the killer;
conditions and effects in the switch fields define the response.

## Runtime sequence

1. `SchemePhysicalOnDeath.activate` parses common switch conditions with `getConfigSwitchConditions`.
2. `SchemePhysicalOnDeath.add` stores a `PhysicalDeathManager` action on the state and subscribes it.
3. `PhysicalDeathManager.onDeath` checks that the physical object still has an active scheme.
4. The manager calls `trySwitchToAnotherSection` for the current state.

The scheme is event-driven. It does not run a regular update loop and does not evaluate the killer object.

## Example

```ini
[logic]
active = ph_on_death@barrel

[ph_on_death@barrel]
on_info = ph_idle@dead %+barrel_destroyed%
```

## Notes

- The implementation comments note that `disable` does not unsubscribe from the death callback because death is expected
  to happen once.
- The scheme does not apply damage, spawn particles, or play sounds by itself. Put those effects in the switch condlist.
- Use another physical scheme, such as `ph_idle`, for the section that should exist after the destroyed-state switch.
