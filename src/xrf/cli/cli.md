# CLI

The engine repository includes a local Node CLI. Run it from the repository root:

Use the npm wrapper:

```powershell
npm run cli -- <command>
```

Common package scripts wrap frequently used commands:

```powershell
npm run build
npm run verify
npm test
```

## Package scripts

Run package scripts from the repository root with `npm run <script>`:

- `setup`: initialize and update submodules.
- `verify`: run `verify project`.
- `build`: build scripts, configs, UI, translations, and resources.
- `pack:mod`: build a mod package.
- `pack:game`: build a game package.
- `watch:scripts`: rebuild scripts when TypeScript sources change.
- `typecheck`: run TypeScriptToLua type checking without emitting files.
- `typecheck:tests`: type-check test sources with TypeScript.
- `lint`: run ESLint with the full repository rule set.
- `test`: run Jest.
- `test:coverage`: run Jest coverage.
- `format`: rewrite Markdown, TypeScript, and LTX files using the configured formatters.
- `help`: print CLI help.

## Commands

The CLI builds and packages the project, manages resources and engine binaries, links a local game, and handles common
format, spawn, particle, translation, and verification tasks. Each command has its own page:

| Command                                      | Purpose                                                   |
| -------------------------------------------- | --------------------------------------------------------- |
| [`build`](./commands/build.md)               | Build `target/gamedata`.                                  |
| [`clone`](./commands/clone.md)               | Clone configured additional resource repositories.        |
| [`compress`](./commands/compress.md)         | Compress built gamedata into archives.                    |
| [`engine`](./commands/engine.md)             | Inspect, switch, list, or roll back bundled engines.      |
| [`format`](./commands/format.md)             | Format LTX files.                                         |
| [`icons`](./commands/icons.md)               | Pack and unpack equipment icons and texture descriptions. |
| [`link`](./commands/link.md)                 | Manage project links to the local game installation.      |
| [`lint`](./commands/lint.md)                 | Run repository lint checks.                               |
| [`logs`](./commands/logs.md)                 | Print the last lines from the linked game log.            |
| [`open`](./commands/open.md)                 | Open configured game and project folders.                 |
| [`pack`](./commands/pack.md)                 | Create mod or game packages.                              |
| [`particles`](./commands/particles.md)       | Pack or unpack `particles.xr`.                            |
| [`parse`](./commands/parse.md)               | Parse directory trees or game externals.                  |
| [`spawn`](./commands/spawn.md)               | Unpack ALife spawn files.                                 |
| [`start`](./commands/start.md)               | Start the configured game executable.                     |
| [`test`](./commands/test.md)                 | Run the project test suites.                              |
| [`translations`](./commands/translations.md) | Initialize, convert, and check translation files.         |
| [`verify`](./commands/verify.md)             | Run project, gamedata, LTX, and particles verification.   |

Use `npm run cli -- <command> --help` for current command-specific options.

## Paths and output

Run commands from the repository root unless a page says otherwise. Most defaults are repository-relative and come from
`cli/config.json`.

Generated output belongs under `target/`: built gamedata, parsed helper files, coverage, packed archives, and package
output. Source edits belong under `src/engine`, `src/resources`, `cli`, or the relevant external resource repository.

## Installed command

`package.json` exposes the binary name `xrf`, but local development should prefer `npm run cli -- ...` so the command
uses the repository version and local dependencies.

If the package is installed globally, the equivalent shape is:

```powershell
xrf build
xrf verify project
```

## Configuration

Most CLI defaults live in `cli/config.json`: locale, resource roots, build source paths, target paths, compression
tools, package roots, and game executable settings.

When a command cannot find the game, resources, or generated output, check the command page first and then inspect the
matching config key in `cli/config.json`. See [CLI Configuration](./configuration.md).
