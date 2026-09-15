# hit

`hit` switches a stalker section when the NPC receives a hit callback. It also records hit metadata that conditions can
use indirectly through the scheme state.

## Parameters

`hit` has no scheme-specific fields.

The section supports common switch fields. They are evaluated from the hit callback.

## Behavior

On activation, `hit` verifies that the configured section exists and parses common switch conditions.

When the NPC is hit, the manager stores the hit bone index and attacker id. Missing attackers are stored as `-1`. A
zero-damage hit is ignored when the object is not invulnerable. If the object has an active scheme, the manager marks
`isDeadlyHit` when the hit amount is greater than or equal to current health times `100`, tries to switch section, then
clears the flag.

## Runtime sequence

1. `SchemeHit.activate` aborts if the referenced section does not exist.
2. `SchemeHit.add` subscribes a `HitManager` action.
3. `HitManager.onHit` stores `boneIndex` in the `hit` scheme state.
4. Zero-damage hits are ignored unless the object is invulnerable.
5. The manager stores `who.id()` or `-1`.
6. While switching, `isDeadlyHit` is true only for hits whose amount is at least `health * 100`.
7. The flag is cleared after the switch attempt.

## Example

```ini
[logic]
active = hit@guard

[hit@guard]
on_info = walker@angry %+guard_was_hit%
```

## Notes

- This scheme is event-driven. Its switch fields are checked when the object is hit.
- Disabling the scheme unsubscribes the stored hit manager.
- Use condition/effect code that reads the scheme state if behavior depends on attacker, bone, or deadly-hit status.
