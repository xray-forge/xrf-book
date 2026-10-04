# Dialog CLI

`dialog info` checks dialog XML structure and summarizes the dialog graph. Use it after changing phrases or links, or to
inspect an imported dialog set. `dialog list` lists dialogs, or the ones an NPC offers, and `dialog inspect` explains
one dialog phrase by phrase. See [dialog configuration](../../script_engine/configs_dialogs.md) for authoring rules.

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

## List the dialogs an NPC offers

```powershell
xrf-cli dialog list --path ./gamedata --source directory --profile zat_b33_stalker_snag --language eng
```

Example output — Snag, in the xrf build:

```text
zat_b33_stalker_snag_about_cache_dialog (configs\gameplay\dialogs_zaton.xml, 17 phrases, start of zat_b33_stalker_snag): -
zat_b33_stalker_snag_share_package_dialog (configs\gameplay\dialogs_zaton.xml, 45 phrases, actor dialog of zat_b33_stalker_snag): Here's the container.
zat_b33_stalker_snag_refuse_task (configs\gameplay\dialogs_zaton.xml, 9 phrases, actor dialog of zat_b33_stalker_snag): About that container...
actor_break_dialog (configs\gameplay\dialogs.xml, 1 phrases, actor dialog of zat_b33_stalker_snag, with info global_dialogs): Laters.
Profile 'zat_b33_stalker_snag' (zat_b33_stalker_snag) offers 4 dialogs
```

`--profile` takes the id `profile_name()` answers for an NPC. The profile resolves to its pinned specific character, or
to every character it may pick by class, and the list holds what the engine gathers when the actor talks to one: the
character's start dialog, which the NPC opens itself, its actor dialogs, and the dialogs info portions offer to
everyone. Character descriptions are read with their `#include` lines spliced in, as the engine reads them. Each dialog
lists its own conditions under `elements` in the report; whether it is offered right now is for the running game to
judge.

Without `--profile`, the command lists every dialog in the tree. An unknown profile, or one that resolves to no
character, exits 1 and names any character files that could not be read.

## Explain one dialog

```powershell
xrf-cli dialog inspect zat_b33_stalker_snag_about_cache_dialog --path ./gamedata --source directory --language eng
```

Each phrase prints as `[id] line -> next ids | effects`, the effects being its info portions, preconditions and actions.
A dialog declared by more than one file is read from the first and the others are named; `--file` picks one by its
logical path. The report carries every element of every phrase in document order.

## Command reference

{{#include reference/dialog.md:commands}}
