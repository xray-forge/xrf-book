# UI

Runtime UI scripts load XML forms, initialize X-Ray CUI controls, bind callbacks, and implement menus, debug screens,
and in-game dialogs.

This page covers runtime scripts. Form source generation is covered in the Forms page.

## Source layout

| Source                              | Purpose                                        |
| ----------------------------------- | ---------------------------------------------- |
| `src/engine/core/ui`                | Runtime UI classes                             |
| `src/engine/core/utils/ui`          | XML loading and element initialization helpers |
| `src/engine/forms`                  | TSX/static XML form sources                    |
| `src/engine/declarations/callbacks` | Engine-facing UI callbacks                     |

## XML loading

`resolveXmlFormPath(path, hasWideScreenSupport)` normalizes form paths and optionally selects a `.16.xml` variant when
the current screen is wide and the file exists.

`resolveXmlFile(path, xml?, hasWideScreenSupport?)` creates or reuses `CScriptXmlInit`, resolves the form path, and
calls `ParseFile`.

In debug mode, XML paths containing `/` abort because X-Ray expects Windows-style UI paths.

## Element initialization

`initializeElement(xml, type, selector, base, descriptor?)` initializes CUI controls from XML and can register callbacks
on the owning `CUIScriptWnd`.

Supported element types include:

- buttons, check buttons, combo boxes, tabs, track bars, edit boxes, list boxes, scroll views;
- static text and text windows;
- frames and frame lines;
- map list and map info controls;
- message boxes;
- generic windows.

`initializeStatic` and `initializeStatics` are shortcuts for static controls.

## Runtime UI classes

The runtime UI tree includes:

- main menu and options dialogs;
- save/load dialogs;
- extensions dialog;
- debug dialog and debug sections;
- sleep, numpad, and freeplay dialogs;
- inventory menu integration through actor menu callbacks.

These classes load XML by selector names. If an XML element name changes, update the runtime class and tests together.

## Interface externals

Separate callback files register engine-facing modules:

- `loadscreen.ts` registers `loadscreen`;
- `inventory_upgrades.ts` registers `inventory_upgrades`;
- `actor_menu.ts` registers `actor_menu`;
- `actor_menu_inventory.ts` registers `actor_menu_inventory`;
- `pda.ts` registers `pda`;
- `ui_wpn_params.ts` registers `ui_wpn_params`.

These callbacks connect UI XML or C++ UI code to managers such as `LoadScreenManager`, `UpgradesManager`,
`ActorInventoryMenuManager`, `TradeManager`, and `PdaManager`.

## Guidelines

- Keep generated XML and runtime selectors in sync.
- Check paired 16:9 forms when changing layout.
- Register UI callbacks through `initializeElement` when the control needs script events.
- Use `resolveXmlFile` instead of constructing `CScriptXmlInit` paths by hand.
