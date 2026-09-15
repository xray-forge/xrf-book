# Dialog configs

Dialog configs define conversation XML and the script predicates/actions used by dialog phrases. XRF keeps dialog data
under `src/engine/configs/gameplay` and dialog externs under `src/engine/declarations/dialogs`.

Use this page when you need to add a phrase, wire a phrase to script, or check why a dialog option is not visible.

## Source files

Core dialog XML files:

- `src/engine/configs/gameplay/dialogs.xml`
- `src/engine/configs/gameplay/dialogs_zaton.xml`
- `src/engine/configs/gameplay/dialogs_jupiter.xml`
- `src/engine/configs/gameplay/dialogs_pripyat.xml`

Dialog text lives in translation files such as:

- `src/engine/translations/st_dialogs.json`
- `src/engine/translations/st_dialogs_zaton.json`
- `src/engine/translations/st_dialogs_jupiter.json`
- `src/engine/translations/st_dialogs_pripyat.json`

Script callbacks live in:

- `src/engine/declarations/dialogs/generic.ts`, `object.ts`, and `world.ts` for shared dialog callbacks;
- `src/engine/declarations/dialogs/zaton`, `jupiter`, `pripyat`, and `quests` for quest callbacks;
- `src/engine/declarations/dialogs/dialog_manager` for generic dialog categories and phrase state.

Generic dialog state is handled by `src/engine/core/managers/dialogs/DialogManager.ts`. The manager tracks phrase
priority tables, disabled phrases, and generic phrase categories such as hello, job, anomalies, and information.

## Runtime hooks

Dialog XML can call script functions through registered dialog externs. XRF registers these from
`src/engine/declarations/dialogs`. The global script entry point loads `externals_registrator`, and the registrator
exposes `dialogs`, `dialogs_zaton`, `dialogs_jupiter`, `dialogs_pripyat`, and `dialog_manager`.

Files under `dialogs/dialog_manager` also register callbacks used by generated generic dialogs. Examples include:

- `dialog_manager.init_new_dialog`;
- `dialog_manager.fill_priority_hello_table`;
- `dialog_manager.precondition_job_dialogs`;
- `dialog_manager.action_information_dialogs`;
- `dialog_manager.action_disable_phrase`.

Keep XML callback names in sync with the extern name. A typo in XML does not create a TypeScript error at the call site.

When changing a dialog:

1. update the XML phrase flow;
2. update translation ids used by the phrases;
3. update or add dialog predicate/action externs when XML calls script;
4. test the dialog declaration code when behavior changes.

## Editing workflow

Start from the dialog id and search across gameplay XML, translation JSON, and dialog declarations. Most quest dialogs
touch at least two of those areas: XML controls phrase flow, translations hold the text, and declarations decide whether
a phrase is visible or what action runs after selection.

For a script-backed phrase:

1. add the XML phrase node and translation id;
2. reference an existing callback or add a new `extern(...)` in the matching declaration file;
3. update tests beside the declaration when the callback has logic;
4. run a focused test for the declaration file, then run config verification if XML changed.

Use location-specific declaration folders when the condition belongs to a level quest. Use `dialogs/dialog_manager` for
generic dialog category behavior only.

## Generated XML

Some gameplay XML is generated from `.tsx` sources. Dynamic XML sources export `create()` and are rendered with
`renderJsxToXmlText`.

Do not edit generated XML under `target/`. Change the XML source or TSX generator.

## Validation notes

- `externals_registrator.test.ts` confirms the dialog extern namespaces are registered.
- Dialog declaration tests check individual callback bindings and branch behavior.
- Translation ids must exist in the matching `st_dialogs*.json` file, otherwise the game can show raw ids or missing
  text.
- If a dialog changes quest state, check the related task, info portion, and effect declarations in the same pass.
