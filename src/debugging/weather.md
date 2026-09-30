# Debug weather

XRF weather is managed by `WeatherManager`. A level's `weathers` field in `game_maps_single.ltx` names either a weather
cycle, played as it is, or `dynamic`, which plays the hourly weather graphs in
`environment\\dynamic_weather_graphs.ltx`.

## Runtime weather flow

On actor network spawn, the manager:

1. reads the current level's `weathers` field, defaulting to `dynamic`, and parses it as a condlist;
2. picks the weather section: the named cycle, or the `dynamic_<period>` graph of the current good or bad period;
3. plays a `w_<state>` cycle of that graph, or the named cycle, immediately;
4. resumes the weather FX the game was saved during, after its cycle is set, so the level returns to that cycle once the
   effect ends.

Once every game hour, it:

- switches between the good and bad periods when the current one has lasted its duration, playing the
  `dynamic_transition` graph for that hour;
- plays the `dynamic_pre_blowout` graph and holds the period for the two hours before a surge and while one plays;
- picks a new state from the graph and blends into its cycle.

A playing weather FX is never cut into: the engine records the new cycle and returns to it when the effect ends.

Use the debug panel `general` section to dump Lua state when you need to inspect the live `WeatherManager` fields. The
dump is written to `_appdata_\\dumps\\lua_data.json`.

## Change weather from scripts

Script effects can set a weather cycle through `xr_effects.set_weather`, which calls `level.set_weather`:

```ini
on_info = %+some_info =set_weather(w_clear:true)%
```

The first argument is the weather cycle. The optional second argument controls whether the change is forced. On levels
with dynamic weather, the manager plays its own cycle again on the next game hour.

From Lua/TypeScript runtime code, the underlying engine call is:

```ts
level.set_weather(weatherName, isForced);
```

For weather effects, the engine exposes:

```ts
level.set_weather_fx("fx_surge_day_3");
level.start_weather_fx_from_time("fx_surge_day_3", time);
```

## Debug weather editor issues

OpenXRay includes a weather editor project and editor documentation. Use it for visual tuning of weather sections and
effects, then confirm the resulting section names and graph entries in XRF configs.

If a weather change does not appear:

- verify the level `weathers` field resolves to the expected section or condlist branch;
- verify the weather graph exists in `dynamic_weather_graphs.ltx` and each of its states has a `w_<state>` cycle;
- check the engine log for `! Invalid weather name`, printed for a cycle that does not exist;
- check whether a weather FX is currently playing;
- dump Lua state and inspect `weatherSection`, `weatherState`, `weatherPeriod`, and `savedWeatherFx`.

## References

- [OpenXRay weather editor article](https://github.com/OpenXRay/xray-16/wiki/%5BEN%5D-Game-Editor#weather-editor)
