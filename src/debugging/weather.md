# Debug weather

XRF weather is managed by `WeatherManager`. It reads level weather settings, dynamic weather graphs, AtmosFear-style
configuration, and weather FX state, then updates weather on actor spawn and on hourly game-time changes.

## Runtime weather flow

On actor network spawn, the manager:

1. reads the current level's `weathers` setting from `game.ltx`;
2. parses it as a condlist;
3. initializes the current weather period and graph state;
4. applies the first weather immediately.

During actor updates, it:

- advances graph state when the game hour changes;
- switches between good and bad weather periods;
- marks transition and pre-blowout weather states;
- updates depth-of-field settings for AtmosFear weather;
- resumes weather FX from saved state when an FX is active.

Use the debug panel `general` section to dump Lua state when you need to inspect the live `WeatherManager` fields. The
dump is written to `_appdata_\\dumps\\lua_data.json`.

## Change weather from scripts

Script effects can force weather through `xr_effects.set_weather`, which calls `level.set_weather`:

```ini
on_info = %+some_info =set_weather(default_clear:true)%
```

The first argument is the weather section. The optional second argument controls whether the change is forced.

From Lua/TypeScript runtime code, the underlying engine call is:

```ts
level.set_weather(weatherName, isForced);
```

For weather effects, the engine exposes:

```ts
level.set_weather_fx("fx_surge_day_3");
level.start_weather_fx_from_time("fx_surge_day_3", time);
```

## Weather console settings

`WeatherManager` applies commands from the `weather_console_settings` section in
`environment\\dynamic_weather_graphs.ltx` during initialization. Use that section for console settings that must be
applied with the dynamic weather system.

## Debug weather editor issues

OpenXRay includes a weather editor project and editor documentation. Use it for visual tuning of weather sections and
effects, then confirm the resulting section names and graph entries in XRF configs.

If a weather change does not appear:

- verify the level `weathers` field resolves to the expected section or condlist branch;
- verify the weather graph exists in `dynamic_weather_graphs.ltx`;
- check whether a weather FX is currently playing;
- check whether the level is treated as underground;
- dump Lua state and inspect `currentWeatherSection`, `nextWeatherSection`, `weatherFx`, and `weatherState`.

## References

- [OpenXRay weather editor article](https://github.com/OpenXRay/xray-16/wiki/%5BEN%5D-Game-Editor#weather-editor)
