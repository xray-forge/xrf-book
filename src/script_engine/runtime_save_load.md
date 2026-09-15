# Runtime save and load

XRF has two save paths:

- engine net-packet save/load for binders, server objects, and selected managers;
- dynamic save data stored beside the game save through marshal-backed files.

Use net-packet save/load for compact state that is part of the engine lifecycle. Use dynamic save data for flexible
extension or event state that should not be constrained by packet layout.

## Save manager

`SaveManager` coordinates core manager save/load and engine save callbacks.

Client manager state is saved and loaded through:

- `WeatherManager`;
- `ReleaseBodyManager`;
- `SurgeManager`;
- `PsyAntennaManager`;
- `SoundManager`;
- `StatisticsManager`;
- `TreasureManager`;
- `TaskManager`;
- `ActorInputManager`;
- `GameSettingsManager`;
- `DeimosManager`.

Server manager state currently goes through `SimulationManager`.

`SaveManager` also handles `alife_storage_manager` callbacks exposed from
`src/engine/declarations/callbacks/alife_storage_manager.ts`.

## Dynamic save data

Before a game save, `SaveManager.onBeforeGameSave(saveName)`:

1. emits `GAME_SAVE`;
2. saves extension state;
3. writes `registry.dynamicData` with `saveDynamicGameSave`.

When loading starts, `SaveManager.onGameLoad(saveName)`:

1. loads `registry.dynamicData` with `loadDynamicGameSave`;
2. loads extension state;
3. emits `GAME_LOAD`.

After the engine reports successful load, `SaveManager.onAfterGameLoad(saveName)` emits `GAME_LOADED`.

## Binder save and load

Save-relevant binders write their own state from `save(packet)` and read it in `load(reader)`.

Common binder pattern:

```ts
openSaveMarker(packet, BinderClass.__name);
super.save(packet);
saveObjectLogic(this.object, packet);
closeSaveMarker(packet, BinderClass.__name);
```

The load path must read fields in the same order:

```ts
openLoadMarker(reader, BinderClass.__name);
super.load(reader);
loadObjectLogic(this.object, reader);
closeLoadMarker(reader, BinderClass.__name);
```

`saveObjectLogic` persists logic file names, active section, smart terrain name, activation time, active scheme save
event, and portable store data. `loadObjectLogic` restores the matching loaded fields and portable store.

## Save markers

Save markers protect net-packet layout.

- `openSaveMarker` records the current packet offset.
- `closeSaveMarker` writes the saved block size.
- `openLoadMarker` records the current reader offset.
- `closeLoadMarker` checks that the loaded block size matches the saved block size.

The marker helpers assert when the read/write sizes drift. If a save format changes, update save and load together and
adjust tests for the saved data list.

## Guidelines

- Keep net-packet data compact.
- Never reorder saved fields without updating the load path.
- Save manager state through `SaveManager` when it is part of global runtime state.
- Save object logic through binder save/load when it belongs to one online object.
- Use dynamic save data for extension data or flexible event state.
