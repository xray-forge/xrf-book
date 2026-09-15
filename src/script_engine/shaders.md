# Shaders

Shader files are static game resources. XRF does not compile or generate shader source from TypeScript. The build copies
shader files from the resource roots into `target/gamedata/shaders` as part of the `resources` build target.

Use this page when you need to find where shader files live, how they reach the game package, or where configs and UI
forms reference shader names.

## Source layout

| Source                                        | Purpose                                         |
| --------------------------------------------- | ----------------------------------------------- |
| `src/resources/shaders`                       | Base shader files copied to `gamedata/shaders`. |
| `src/resources/shaders/r1`, `r2`, `r3`, `gl`  | Renderer-specific shader directories.           |
| `src/resources/shaders/shared`                | Shared include files used by shader source.     |
| `src/engine/configs/**/*.ltx`                 | Config references to shader names.              |
| `src/engine/forms/**/*.tsx` and static UI XML | UI texture nodes can set a `shader` attribute.  |
| `src/engine/constants/roots.ts`               | Defines the `$game_shaders$` root alias.        |

The base resource directory also contains other static asset folders such as `anims`, `levels`, `sounds`, `spawns`, and
`textures`. Shader files follow the same static-resource build path as those folders.

## Build behavior

The CLI `resources` target copies static assets from configured resource roots:

```powershell
npm run cli build -- --include resources
```

The default root is `src/resources`. When asset overrides are enabled, the CLI also checks locale and override resource
roots from `cli/config.json` before the base root.

The static resource copier intentionally skips development-only folders such as `textures_unpacked` and
`particles_unpacked`. It also rejects resource roots that overlap generated engine folders such as `configs`, `scripts`,
`core`, or `lib`.

## References from configs

Shader names are usually referenced by engine configs rather than by scripts. Examples include:

```ltx
shader = font
tracer_shader = effects\bullet_tracer
sun_shader = effects\sun
```

UI forms can also set shader attributes on texture nodes:

```tsx
<texture shader={"hud\\p3d"}>ui_inGame2_Detector_icon_acid_big</texture>
```

Keep these references aligned with files under `gamedata/shaders`. The game resolves shader names through the normal
X-Ray filesystem aliases, including `$game_shaders$`.

## Guidelines

- Edit shader source under resource roots, not under `target/`.
- Use the `resources` build target for shader-only changes.
- Search configs and forms before renaming a shader path.
- Keep renderer-specific variants together when a shader exists in more than one renderer directory.
- Treat `src/resources` as resource-owned content. Avoid broad edits unless the task is about assets.
