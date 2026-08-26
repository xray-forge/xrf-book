# Weather configs

Weather configs define environment cycles, ambient sounds, fog, suns, thunderbolts, weather effects, and manager-level
weather selection. Runtime weather behavior is handled by `WeatherManager`.

## Source layout

Important source areas:

- `src/engine/configs/environment/environment.ltx`;
- `src/engine/configs/environment/weathers/*.ltx`;
- `src/engine/configs/environment/weather_effects/*.ltx`;
- `src/engine/configs/environment/ambients/*.ltx`;
- `src/engine/configs/environment/ambient_channels/*.ltx`;
- `src/engine/configs/environment/fog/*.ltx`;
- `src/engine/configs/environment/dynamic_weather_graphs.ltx`;
- `src/engine/configs/managers/weather_manager.ltx`;
- `src/engine/configs/managers/weather/weather_manager_levels.ltx`.

## Weather sections

The `$weather` schema is strict and includes fields for sky, fog, rain, sun, clouds, wind, water, ambient, thunderbolt,
and sun shafts. A weather cycle is built from time sections that point the engine at these values.

Weather effect sections describe temporary events such as surge, blowout, and psi storm effects. They reference
particles, sound, wind, and lifetime fields.

## Runtime manager

`WeatherManager` subscribes to actor update events and actor online events. It applies configured weather and exposes
debug information through the XRF debug panel.

On actor network spawn, the manager reads the current level's `weathers` value from `game.ltx`. If no level-specific
value is configured, it uses the AtmosFear-style dynamic weather section. The selected value is parsed as a condition
list and becomes the source for later weather section selection.

During actor updates, the manager:

- checks hourly changes and advances weather graph state;
- changes good/bad weather periods when the configured period boundary is reached;
- marks pre-blowout weather when a surge or weather FX is close;
- updates DOF every five game seconds for active AtmosFear weather;
- saves and loads weather section, period, graph state, and active weather FX data.

When changing weather selection logic, update manager tests. When changing only LTX values, validate the configs and
check the result in game.

## Validation

Run:

```powershell
npm run cli verify ltx
```

Use the [debugging weather page](../debugging/weather.md) for in-game inspection and debug panel workflow.

## Editing notes

- Edit weather cycle values under `environment/weathers` when changing sky, fog, rain, sun, or ambient output.
- Edit `dynamic_weather_graphs.ltx` when changing transitions between clear, cloudy, rainy, or related graph states.
- Edit `weather_manager_levels.ltx` and `game.ltx` links when changing which weather set a level uses.
- Use a save/load check after manager logic changes because weather state is serialized.
