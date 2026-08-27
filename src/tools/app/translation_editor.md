# Translation Editor

The translation editor opens XRF translation projects for read-only inspection.

## Screens

| Screen       | Route                          | Purpose                                 |
| ------------ | ------------------------------ | --------------------------------------- |
| Navigator    | `/translations_editor`         | Open a translation project.             |
| Project view | `/translations_editor/project` | Inspect the loaded translation project. |

## Backend commands

The Tauri `translations-editor` plugin exposes:

- `open_translations_project`;
- `read_translations_project`;
- `get_translations_project`;
- `close_translations_project`.

Both open/read paths use `TranslationProject::read_project`.

## Workflow

1. Open a translation project folder or file.
2. Inspect loaded translation ids and language values in the project view.
3. Close and reopen after editing translation sources outside the app.

When an XRF project path is configured, the app can prefill `src/engine/translations`.

## Limitations

The app keeps the parsed translation project in the Tauri plugin state and displays it in the project view. It does not
currently expose a save action, build XML output, or run translation verification from the UI.

Use the app for inspection while comparing ids across languages. Use the CLI when you need to initialize files, build
game XML, or validate translation structure.

## CLI equivalent

Use `xrf-cli initialize-translation`, `xrf-cli build-translation`, and `xrf-cli verify-translation` for write, build,
and verification workflows.
