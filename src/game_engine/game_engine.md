# X-Ray engine

XRF runs on top of the X-Ray engine used by S.T.A.L.K.E.R. Call of Pripyat style games. The engine owns the executable,
renderer, file system, console, configuration loading, Lua VM, luabind exports, server objects, client objects, ALife
simulation, and the low-level update loop.

XRF replaces the game script layer. TypeScript sources are compiled to Lua scripts, then loaded by the engine through
the same script entry points that vanilla game logic uses.

## What the engine owns

The engine starts from the executable. During startup it initializes core services, resolves the file system layout,
loads configuration files, creates the console, initializes the Lua script engine, and opens engine bindings for Lua.

At runtime it also owns:

- server-side ALife objects and the object factory;
- client-side game objects and their `object_binder` instances;
- save and load packets;
- console command execution;
- render, sound, input, and UI infrastructure;
- frame updates and scheduled object processing.

The source references for these behaviors are the local `xray-16` engine tree and the XRF X-Ray 16 SDK declarations.

## What XRF adds

XRF provides the Lua scripts that the engine calls into:

- `_g.script` preloads `register`, `bind`, and `start`, then registers global script externals.
- `register.script` registers game classes, UI classes, server object classes, and callback functions.
- `start.script` initializes managers, schemes, simulation helpers, extensions, and emits the XRF `GAME_STARTED` event.
- `bind.script` maps engine object sections and script class names to XRF binder classes.

After that point, most game behavior goes through XRF managers, schemes, binders, and event callbacks.

## Where to go next

- Use [Command line arguments](./command_line_arguments.md) when changing engine startup behavior.
- Use [Console commands](./console_commands.md) for commands available after the console is initialized.
- Use [Execution flow](./game_engine_flow.md) to understand the order from executable startup to active gameplay.
- Use [Lifecycle](./game_engine_lifecycle.md) when editing binders, managers, save/load code, or online/offline logic.
- Use [Luabind](./luabind.md) when a TypeScript class needs to be visible to the Lua engine runtime.
