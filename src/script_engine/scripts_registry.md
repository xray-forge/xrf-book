# Registry

The registry is the shared runtime state table for the XRF scripting layer. It is defined in
`src/engine/core/database/registry.ts` and re-exported through `src/engine/core/database`.

Use it for state that must be visible across binders, schemes, managers, and utility modules.

## Main state groups

| Field                         | Purpose                                    |
| ----------------------------- | ------------------------------------------ |
| `simulator`                   | Current ALife simulator                    |
| `actor`                       | Online actor game object                   |
| `actorServer`                 | Actor server object                        |
| `managers` / `managersByName` | Manager singletons                         |
| `schemes`                     | Registered scheme constructors             |
| `objects`                     | Online object registry states              |
| `offlineObjects`              | Saved state for offline objects            |
| `simulationObjects`           | Objects that can participate in simulation |
| `storyLink`                   | Story id to object id mappings             |
| `stalkers`                    | Online stalker id set                      |
| `smartTerrains`               | Registered smart terrains                  |
| `smartCovers`                 | Registered smart covers                    |
| `zones`                       | Active zones by name                       |
| `dynamicData`                 | Marshal-backed dynamic save data           |

There are also focused registries for wounded objects, doors, helicopters, crows, anomaly zones, light zones, silence
zones, no-weapon zones, trade state, camp managers, ranks, goodwill, and save markers.

## Object state

`registry.objects` is the central per-object store. Binders reset and register object state when objects come online.
Schemes attach their state into the same object descriptor by scheme id.

Common object state includes:

- object reference;
- spawn ini and logic section;
- active scheme and active section;
- scheme states;
- overrides;
- state manager and patrol manager for stalkers.

## Story links

Story links are stored both ways:

- `storyLink.sidById`: object id to story id;
- `storyLink.idBySid`: story id to object id.

Use the database helpers to register or unregister story links. Do not update these tables by hand unless the helper
does not cover the case.

## Manager access

Managers should be accessed through:

```ts
getManager(SoundManager);
getWeakManager(SoundManager);
getManagerByName("SoundManager");
```

Direct reads from `registry.managers` are reserved for low-level registry helpers and exceptional circular-reference
cases.

## Guidelines

- Prefer a focused database helper over direct table mutation.
- Store transient object-specific state under `registry.objects`.
- Store persistent cross-system state in a manager or `registry.dynamicData`.
- Clear registry entries in offline, unregister, or destroy paths.
- Keep registry fields narrow. A broad table without a lifecycle owner becomes difficult to save and clean up.
