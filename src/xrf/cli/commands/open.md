# Open

Open commands launch the system file explorer for common project and game folders.

```powershell
npm run cli -- open_game_folder
npm run cli -- open_project_folder
```

## Commands

| Command               | Opens                                                |
| --------------------- | ---------------------------------------------------- |
| `open_game_folder`    | The configured or detected S.T.A.L.K.E.R. game root. |
| `open_project_folder` | The `xrf-engine` repository root.                    |

`open_game_folder` uses the same game path resolution as `link`, `logs`, `start_game`, and `verify project`.
`open_project_folder` uses the repository root detected from the CLI process location.

## Examples

```powershell
npm run cli -- open_game_folder
npm run cli -- open_project_folder
```

## When to use it

Use `open_game_folder` when checking linked `gamedata`, engine logs, or installed game binaries after `build` and
`link`. Use `open_project_folder` when a script or tool printed a project-relative path and you want to inspect the
source tree from Explorer.

These commands do not build, link, or verify files. They only open the resolved folders.

## Failure notes

If the game folder does not open, check `cli/config.json` under `targets` or run `npm run cli -- verify project` to see
which path is being resolved.
