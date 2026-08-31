# Assets

Assets are static gamedata resources copied into the build output. Keep them separate from TypeScript, generated Lua,
generated configs, and generated UI XML.

The main source directory is `src/resources`. Project asset override roots can also be included by the build when asset
overrides are enabled for a language.

## What gets copied

The resources build step copies files and folders into `target/gamedata`. It skips repository metadata and unpacked
working folders such as `textures_unpacked` and `particles_unpacked`.

It also rejects resource folders that overlap with generated source areas such as `core`, `configs`, `lib`, and
`scripts`. Those names are reserved for generated or source-owned content.

## Filtering

Build filters are applied to resource paths. Use a filter when you only need to rebuild a focused asset group:

```powershell
npm run cli build -- --filter textures
npm run cli build -- --filter sounds
```

The filter behavior is path based. Check the source path when a filtered build appears to skip an expected file.

## Editing assets

Treat `src/resources` as base resource data. Avoid casual edits there unless the work is explicitly about resources. For
new generated game data, prefer the appropriate source folder:

- configs in `src/engine/configs`;
- UI forms in `src/engine/forms`;
- translations in `src/engine/translations`;
- scripts in `src/engine/scripts` and `src/engine/core`.

Do not patch packed output or files under `target/`.

## Validation workflow

After asset changes, run the narrowest build filter that covers the path you touched. Then inspect `target/gamedata` to
confirm the copied path matches the engine-facing layout.

Use format-specific tools for packed or structured assets:

- sprite sheet workflows through `sprites` commands;
- `particles.xr` workflows through `particles` commands;
- spawn workflows through `spawn` commands;
- archive inspection through the Tools CLI archive commands.

If an asset change depends on config references, validate the LTX/XML source that points at the asset as well.
