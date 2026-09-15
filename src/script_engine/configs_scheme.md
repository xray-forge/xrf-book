# LTX scheme

`src/engine/configs/$scheme` contains validation schemas for LTX configs. The `xrf-cli ltx verify` command uses these
files to check includes, inheritance, field names, and field types.

The root file is `scheme.ltx`. It includes category schemas such as `base.scheme.ltx`, `script.scheme.ltx`,
`environment.scheme.ltx`, `items.scheme.ltx`, `weapons.scheme.ltx`, and `zone.scheme.ltx`.

## Defining a schema section

A schema section starts with a section name. Use inheritance when several config sections share fields:

```ltx
[$item_weapon]:$item,$item_weapon_sounds,$item_weapon_params
$strict = true
ammo_class = ?string[]
ammo_mag_size = ?u32
weapon_class = ?enum:assault_rifle,shotgun,sniper_rifle,heavy_weapon,pistol,grenade,misc
```

Schema section names must start with `$`. A game config section selects its schema with `$scheme = $item_weapon`.

## Field syntax

| Syntax                      | Meaning                                      |
| --------------------------- | -------------------------------------------- |
| `name = string`             | Required string field.                       |
| `name = ?string`            | Optional string field.                       |
| `name = string[]`           | Array of strings.                            |
| `name = tuple:f32,f32,f32`  | Tuple with fixed element types.              |
| `name = enum:on,off`        | Value must be one of the listed enum values. |
| `name = section`            | Value should reference a section.            |
| `* = tuple:f32,f32,f32,f32` | Wildcard rule for arbitrary field names.     |

Optional marker `?` can be combined with arrays, enums, tuples, and section references.

## Available types

Common schema types:

- `string`
- `section`
- `tuple`
- `condlist`
- `f32`
- `u32`
- `i32`
- `u16`
- `i16`
- `u8`
- `i8`
- `bool`
- `vector`
- `enum`
- `rgb`
- `rgba`
- `const:<value>`

Prefer the narrowest type that matches the engine behavior. The parser rejects unsupported types, including `unknown`
and `any`.

## Arrays, enums, and tuples

Arrays use `[]`:

```ltx
levels = ?string[]
installed_upgrades = ?section[]
```

Enums list allowed values:

```ltx
scope_status = ?enum:0,1,2
flares = enum:on,off
```

Tuples define ordered values:

```ltx
fire_point = ?tuple:f32,f32,f32
hit_power = ?tuple:f32,f32,f32,f32
```

## Strict sections

`$strict = true` means fields not described by the schema are validation errors unless a wildcard rule covers them. Use
strict schemas for stable config formats such as weapons, weather, and script definitions.

Leave strict mode off only when the config format is intentionally open-ended or not fully modeled yet.

## Verification

Run:

```powershell
npm run cli verify ltx
```

When a valid config fails validation, update the matching schema section instead of weakening unrelated schemas. When a
schema accepts invalid data, add a narrower field type or enable strict mode for that schema if the format is stable.
