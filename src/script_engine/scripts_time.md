# Time

Time helpers live in `xrf-xray16-sdk/src/lib/utils/time.ts`. They format game time, convert weather periods, advance
game time, and serialize X-Ray `CTime` values for save data.

## Formatting

### `toTimeDigit`

Pads values below `10` with a leading zero.

### `gameTimeToString`

Formats a `CTime` value as:

```text
hh:mm MM/DD/YYYY
```

### `globalTimeToString`

Formats a duration in milliseconds as:

```text
h:mm:ss
```

This is used by visible timers such as `sr_timer`.

### `hoursToWeatherPeriod`

Converts an hour to a weather period section label:

```text
6 -> 06:00:00
12 -> 12:00:00
```

## Time checks

`isInTimeInterval(fromHours, toHours)` checks whether the current level hour is inside a range. Ranges that cross
midnight are supported.

For example, `22` to `4` means late night through early morning.

## Changing time

`setCurrentTime(hour, min, sec)` advances game time to the requested time of day. If the current day has already passed
that time, it advances to the next day.

The helper temporarily sets the level time factor to `10000`, waits until game time reaches the target, then restores
the previous time factor.

Use this carefully. It is not a passive setter; it advances the simulation while waiting.

## Save and load

`writeTimeToPacket(packet, time)` writes a nullable time value to a net packet. `null` is stored as `MAX_U8`.

`readTimeFromPacket(reader)` restores a `CTime` value or returns `null` for the null marker.

`serializeTime(time)` and `deserializeTime(data)` use `marshal` to store and restore a `CTime` tuple as a string.

## Guidelines

- Use `globalTimeToString` for millisecond durations.
- Use `gameTimeToString` for calendar game time.
- Use packet helpers for net save data, not ad hoc string formatting.
- Avoid `setCurrentTime` in ordinary update paths because it waits while time advances.
