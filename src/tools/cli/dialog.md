# Dialog CLI

`dialog info` checks dialog XML structure and summarizes the dialog graph. Use it after changing phrases or links, or to
inspect an imported dialog set. See [dialog configuration](../../script_engine/configs_dialogs.md) for authoring rules.

## Inspect a dialog set

From a project containing an assembled `gamedata` directory:

```powershell
xrf-cli dialog info --path ./gamedata --source directory --strict --report ./dialog-report.json
```

`--source directory` selects the loose tree explicitly. The default, `containing-installation`, can discover the game
installation around the supplied path. Repeat `--path` for layered roots, with the highest-priority root first;
`--prefix` narrows the virtual path scope.

The summary covers files, dialogs, phrases, and links, including empty dialogs, final phrases, missing text, and phrases
outside a phrase list. Use `--verbose` for individual findings or inspect `result` in the saved report.

## Interpret the result

Without `--strict`, completed inspection can return exit 0 even when it finds invalid dialogs. With `--strict`, those
findings produce exit 3. An error, incomplete inspection, or skipped input produces exit 1, including when no dialog
files were selected.

These checks establish structural consistency. Review the conversation in game to verify its conditions, scripting, and
intended flow.

## Command reference

{{#include reference/dialog.md:commands}}
