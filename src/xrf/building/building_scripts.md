# Building Scripts

The scripts target compiles entry points in `src/engine/scripts` and their imports to Lua with TypeScriptToLua.

The output is written into `target/gamedata` as X-Ray script files. The generated bundle also includes
`lualib_bundle.script`, which provides helper functions emitted by TypeScriptToLua.

## Build scripts

```powershell
npm run cli -- build --include scripts
npm run cli -- build -i scripts
```

Use `--no-lua-logs` when you want compiled scripts without Lua logger calls:

```powershell
npm run cli -- build -i scripts --no-lua-logs
```

Use `--inject-tracy-zones` when profiling with Tracy support:

```powershell
npm run cli -- build -i scripts --inject-tracy-zones
```

## Watch mode

Use watch mode during script development:

```powershell
npm run watch:scripts
```

Additional script watch commands are available for optimized and Tracy-instrumented builds:

```powershell
npm run watch:scripts-optimized
npm run watch:scripts-tracy
npm run watch:scripts-tracy-optimized
```

## Type checking

Run TypeScriptToLua type checking without emitting files:

```powershell
npm run typecheck
```

When script compilation reports diagnostics, use `typecheck` for a focused failure report.

## Type Definitions

X-Ray engine APIs and supporting Lua typedefs come from the `xray16` package. Supporting typedefs are installed under
`node_modules/xray16/typedefs`. Use the generated type documentation when checking engine class, method, and enum names:

- [XRF X-Ray 16 SDK source](https://github.com/xray-forge/xrf-xray16-sdk)
- [XRF X-Ray 16 SDK documentation](https://xray-forge.github.io/xrf-xray16-sdk/index.html)

## Luabind classes

XRF uses custom TypeScriptToLua transforms for luabind-style classes. Classes that need engine-compatible luabind
registration use project decorators and generated Lua class shapes instead of plain TypeScriptToLua metatable classes.
