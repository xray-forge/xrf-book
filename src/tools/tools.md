# XRF Tools

`xrf-tools` is the XRF companion workspace. It contains reusable Rust format crates, a CLI, and a Tauri desktop
application.

Use it for tasks that are awkward to do by hand:

- reading and unpacking X-Ray archives;
- verifying and formatting LTX configs;
- converting and checking translations;
- inspecting script exports;
- packing and unpacking equipment icons and texture descriptions;
- inspecting or converting spawn, particles, OGF, and OMF data.

## Repository layout

- `crates/`: reusable Rust crates for X-Ray formats and project validation.
- `bin/xrf-cli`: command-line tool.
- `bin/xrf-app`: Tauri backend for the desktop application.
- `bin/xrf-ui`: React frontend for the desktop application.

The engine repository uses a bundled tools binary from `cli/bin` for some build and asset operations.

## Choose an interface

Use the CLI when the command must be repeatable, run in CI, or become part of an engine build step. Examples include LTX
verification, translation builds, archive unpacking, spawn conversion, and texture packing.

Use the desktop app when you need to inspect structured project data with navigation: archives, configs, dialogs,
exports, icons, spawns, and translations. Some app routes are read-only or prototype workflows; the application itself
is the authority on what each tool currently supports.

## Implementation source

Tool behavior comes from the tools workspace, not from the book text:

- CLI commands: `xrf-tools/bin/xrf-cli/src/commands`;
- desktop backend commands: `xrf-tools/bin/xrf-app/src`;
- desktop frontend routes: `xrf-tools/bin/xrf-ui/src/applications`;
- reusable format logic: `xrf-tools/crates`.
