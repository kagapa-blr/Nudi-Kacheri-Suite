# Nudi Kacheri Suite Developer Guide

This workspace is a customized source tree based on [ONLYOFFICE Desktop Editors](https://github.com/ONLYOFFICE/DesktopEditors). The parent repository assembles the project through Git submodules. Source changes must be made inside the submodule that owns the behavior, then the parent repository must record the new submodule commit.

## Repository map

| Submodule | Owns | Make changes here when you need to change |
| --- | --- | --- |
| `desktop-apps` | Native Desktop Editors frontend and platform shell | Window behavior, application startup, tabs, menus outside the editor, file opening, printing integration, native dialogs, updates, themes at the shell level, Linux/Windows/macOS packaging and branding |
| `desktop-sdk` | Shared desktop SDK and embedded Chromium/editor integration | Desktop SDK interfaces, browser/editor embedding, communication between the native shell and editor pages, shared desktop services |
| `web-apps` | Main editor user interface | Toolbar, panels, menus, dialogs, editor layout, document/spreadsheet/presentation/PDF UI, UI strings and web-side interaction behavior |
| `sdkjs` | JavaScript editor SDK and document model/runtime | Client-side editor APIs, document model behavior, commands, collaboration-facing client logic, shared Word/Cell/Slide/PDF runtime behavior |
| `core` | Native document engines and converters (`x2t` and related components) | File parsing, conversion, rendering, import/export, OOXML/ODF/RTF/PDF and other document-format behavior |
| `dictionaries` | Spell-checking and hyphenation data | Adding or correcting a language dictionary, affix data, thesaurus/hyphenation data, or language package content |
| `build_tools` | Build orchestration, dependency setup, and product build scripts | Building Desktop Editors, preparing dependencies, configuring output, and generating distributable packages |

The upstream Desktop Editors repository also contains these same components as submodules. This checkout uses custom `kagapa-blr/*` remotes, so treat those forks as the source of truth for this project unless the team explicitly changes the policy.

## How to route a change

Start with the user-visible symptom and follow this order:

1. **Native application or operating-system behavior**: inspect `desktop-apps` first.
2. **Editor page or toolbar behavior**: inspect `web-apps` first, then `sdkjs` if the UI calls an SDK command or API.
3. **Document content, layout, import, export, or conversion**: inspect `core` first; check `sdkjs` only when the issue is client-side document logic.
4. **Desktop-to-editor bridge or embedded browser behavior**: inspect `desktop-sdk`, then the calling code in `desktop-apps` or `web-apps`.
5. **Spelling or hyphenation**: inspect `dictionaries`; do not change editor logic for a data-only correction.
6. **Packaging, product name, icons, installer, or platform resources**: inspect `desktop-apps/package`, `desktop-apps/common`, and the relevant platform directory.

When the same feature crosses layers, keep each change in its owning repository. For example, a new toolbar command normally needs a `web-apps` UI change, an `sdkjs` command/API change, and possibly a `core` change if it changes file representation. Record and test each submodule commit together in the parent repository.

## Important directories

### `desktop-apps`

- `win-linux/`: Qt/native application implementation and Linux/Windows build entry points.
- `win-linux/src/`: application startup, windows, tabs, editor tools, themes, events, printing, updates, and native integration.
- `macos/`: macOS application implementation.
- `common/`: shared desktop resources, branding, licenses, icons, help, and packaging inputs.
- `package/`: Linux package generation and product metadata.

### `desktop-sdk`

- `ChromiumBasedEditors/`: Chromium-based editor hosting and desktop integration.
- `HtmlFile/`: HTML/editor-related SDK support.

### `web-apps`

- `apps/documenteditor/`: document editor UI.
- `apps/spreadsheeteditor/`: spreadsheet editor UI.
- `apps/presentationeditor/`: presentation editor UI.
- `apps/pdfeditor/`: PDF editor UI.
- `apps/common/`: shared web UI and common editor functionality.
- `build/`: web-app build and bundling configuration.
- `translation/`: web-app localization resources.
- `test/`: web-app tests.

### `sdkjs`

- `word/`: document editor runtime and model.
- `cell/`: spreadsheet editor runtime and model.
- `slide/`: presentation editor runtime and model.
- `pdf/`: PDF runtime and UI support.
- `common/`: shared SDK/runtime code.
- `build/`: SDK build scripts and bundle configuration.
- `configs/`, `tools/`, `vendor/`: configuration, developer tooling, and third-party code.

### `core`

- `DesktopEditor/`: desktop-facing core integration.
- `OOXML/`, `MsBinaryFile/`, `OdfFile/`, `RtfFile/`: document format handling.
- `PdfFile/`, `HtmlFile/`, `XpsFile/`, `DjVuFile/`, `EpubFile/`: format-specific readers, writers, and converters.
- `DocxRenderer/`, `X2tConverter/`: rendering and conversion paths.
- `Common/`, `OfficeUtils/`: shared native utilities and format infrastructure.

### `dictionaries`

Each language is a separate locale directory such as `en_US` or `en_GB`. Use the locale directory already present for the language you are changing, and keep its dictionary data and accompanying README/license information together.

### `build_tools`

- `tools/linux/`: Linux automation scripts, including the Desktop Editors build entry point.
- `scripts/`: shared build and dependency scripts.
- `tools/`: platform and product-specific build tooling.
- `out/`: generated build output; do not commit it.

## Runtime relationship

```text
Desktop application shell
  desktop-apps
        |
        v
Desktop SDK / embedded Chromium bridge
  desktop-sdk
        |
        +--------------------+
        v                    v
Editor interface        Editor JavaScript runtime
  web-apps                 sdkjs
        |                    |
        +---------+----------+
                  v
        Native document/conversion core
                  core

Spell-check and hyphenation data: dictionaries
Build orchestration and generated output: build_tools
```

The exact build pipeline is orchestrated by ONLYOFFICE build tooling. The parent repository itself is not the place to compile individual editor components.

## Working with submodules

Clone or initialize the complete tree:

```bash
git clone --recurse-submodules <your-parent-repository-url>
cd Nudi-Kacheri-Suite
git submodule update --init --recursive
```

Before changing code, inspect both levels:

```bash
git status --short --branch
git submodule status --recursive
for dir in build_tools core desktop-apps desktop-sdk dictionaries sdkjs web-apps; do
  git -C "$dir" status --short --branch
done
```

A normal change has two commits:

1. Commit the source change in the affected submodule repository.
2. Commit the updated submodule pointer in this parent repository.

Example:

```bash
cd web-apps
git switch -c customize-toolbar
git add <changed-files>
git commit -m "Customize editor toolbar"
cd ..
git add web-apps
git commit -m "Update web-apps for toolbar customization"
```

Do not edit files inside a submodule and expect the parent commit alone to preserve them. The parent stores only the submodule commit ID. Push the submodule branch/commit to the configured custom remote before sharing the parent commit.

## Build and test

The source build is driven by the local [`build_tools`](build_tools) submodule. Follow its README and project setup documentation for supported operating-system packages, compiler versions, Qt/Chromium dependencies, and output paths. On Linux, the Desktop Editors build entry point is `build_tools/tools/linux/automate.py`:

```bash
cd build_tools/tools/linux
python3 ./automate.py desktop
```

The local `desktop-apps/package/Makefile` expects generated build output under a `build_tools/out/<platform>/<company>/` tree, so package creation is not a standalone operation.

Use the narrowest validation for the change:

- Native or packaging change: build the affected platform target and launch the resulting application.
- `web-apps` change: run its documented build and test commands, then exercise the affected editor in the desktop application.
- `sdkjs` change: run its documented build/test checks and exercise the affected command or document type.
- `core` change: run the relevant converter or format tests, then open, save, and round-trip representative files.
- `dictionaries` change: test spelling and hyphenation for the affected locale in the built application.

Always test the complete user path after a cross-repository change: launch the desktop shell, open a representative file, perform the customized action, save/export it, close it, and reopen the result.

## Customization rules

- Keep product-specific behavior and branding in the custom fork or a clearly named customization area; avoid modifying third-party vendor code unless required.
- Prefer existing APIs and extension points in `web-apps` and `sdkjs` before changing `core`.
- Treat generated bundles and build output as artifacts. Change source/configuration and regenerate them using the repository build process.
- Keep localization changes in the owning localization area: editor UI translations in `web-apps/translation` and language data in `dictionaries`.
- Preserve upstream compatibility where practical. For every customization, record the affected submodule, upstream version/base commit, reason, and validation performed.
- Check the applicable AGPL and third-party license files before redistributing a customized build.

## Useful upstream references

- [Desktop Editors](https://github.com/ONLYOFFICE/DesktopEditors)
- [Build tools](https://github.com/ONLYOFFICE/build_tools)
- [ONLYOFFICE API documentation](https://api.onlyoffice.com/)
- [Desktop Editors forum](https://community.onlyoffice.com/)
