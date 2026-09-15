# Dialog CLI

`dialog info` checks dialog XML structure and summarizes the dialog graph. Use it after changing phrases or links, or to
inspect an imported dialog set. See [dialog configuration](../../script_engine/configs_dialogs.md) for authoring rules.

## Inspect a dialog set

From a project containing an assembled `gamedata` directory:

```powershell
xrf-cli dialog info --path ./gamedata --source directory --strict --report ./dialog-report.json
```

Example output excerpt — inspect dialogs:

```text
Reading dialogs in ./gamedata (Directory)
Swept 2 files in 7 ms, 0 unreadable, 0 archived
Dialogs: 4 total, 1 with no phrases, 1 with a priority
Phrases: 8 total, 5 links, 3 final, 2 without text, 0 outside a phrase list
Largest dialog: zat_test_trader_start with 4 phrases
Encodings: UTF-8: 1, windows-1251: 1
Dialog elements: dont_has_info: 1, has_info: 1, init_func: 1, precondition: 1
```

Exit code: `0`. The summary ends with "Read 2 files, status: failed". Exit 0 records a completed inspection; add
--strict to fail on its finding.

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
