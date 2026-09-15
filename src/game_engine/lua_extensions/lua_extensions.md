# Lua extensions

The engine Lua environment is not plain standalone Lua. OpenXRay-style builds initialize LuaJIT, luabind, engine
exports, standard libraries, and a few engine-specific libraries before game scripts run.

The inspected script engine opens the usual Lua libraries such as `base`, `package`, `table`, `io`, `os`, `math`,
`string`, `bit`, and `ffi`. Debug builds can also expose the Lua `debug` library. LuaJIT is opened unless the executable
starts with `-nojit`.

## Runtime availability

XRF can have TypeScript declarations for a library even when a specific engine executable does not load that library.
Always separate:

- compile-time declarations shipped in `xray16/typedefs`;
- runtime modules actually opened by the engine;
- modules shipped in the selected gamedata or Lua environment.

This matters for `marshal` and `lfs`: XRF has typings for them, but availability depends on the chosen engine/runtime
package.

## Script path

The engine appends gamedata script paths to `package.path`, allowing scripts to be required from the game script
directory. Keep runtime `require(...)` names aligned with the emitted Lua script layout.

## Practical checks

When adding a dependency on a Lua module:

1. check the TypeScript declaration under `xray16/typedefs` (source: `xrf-xray16-sdk/typedefs`);
2. check whether the target executable opens or ships the module;
3. run the game with the same executable that will ship to users;
4. keep fallback behavior for optional modules.

For engine-bound code, prefer X-Ray APIs and XRF helpers over standalone Lua assumptions. The engine can change module
availability, package paths, and debug library access depending on build flags.
