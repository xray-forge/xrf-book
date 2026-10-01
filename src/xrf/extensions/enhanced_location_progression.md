# Enhanced Location Progression

`Enhanced location progression` requires a smart terrain to be visited before it appears on the map or can be selected
as a same-level travel target.

The source lives under `src/engine/extensions/enhanced_location_progression`.

## Default state

The extension exports:

```ts
export const name = "Enhanced location progression";
export const enabled = true;
```

It is enabled by default unless saved extension state overrides it.

## Behavior

On registration, the extension sets:

```ts
mapDisplayConfig.REQUIRE_SMART_TERRAIN_VISIT = true;
```

This flag is checked by map display and travel code.

## Map Spots

`updateTerrainsMapSpotDisplay` shows global terrain spots only when:

- `REQUIRE_SMART_TERRAIN_VISIT` is disabled; or
- the actor has the `"<terrain>_visited"` info portion.

Restrictor lifecycle code gives visited info portions when the actor reaches the matching restrictor.

## Travel

`isTravelRouteAvailable` from the travel manager utils rejects smart terrain travel targets on the current level when
`REQUIRE_SMART_TERRAIN_VISIT` is enabled and the terrain has not been visited.

## Saved data

This extension does not export `save` or `load`. It changes runtime config during extension registration.

## Editing Notes

- Keep the progression toggle in `main.ts`.
- Check map and travel behavior together when changing this flag.
- Preserve visited info portion naming because other code checks `"<terrain>_visited"`.
