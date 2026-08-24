# Nudi Kacheri Suite

Nudi Kacheri Suite is a customized [ONLYOFFICE Desktop Editors](https://github.com/ONLYOFFICE/DesktopEditors) source workspace. It combines the Desktop Editors shell, editor frontends, JavaScript runtime, document-conversion core, dictionaries, and build automation as Git submodules.

## Included repositories

| Repository | Purpose |
| --- | --- |
| `desktop-apps` | Native Desktop Editors application shell and platform packaging |
| `desktop-sdk` | Desktop SDK and embedded editor integration |
| `web-apps` | Document, spreadsheet, presentation, and PDF editor interface |
| `sdkjs` | JavaScript editor SDK and document runtime |
| `core` | Native document readers, writers, renderers, and converters |
| `dictionaries` | Spell-checking and hyphenation data |
| `build_tools` | Dependency setup, build orchestration, and package generation |

The submodules are configured to use the customized repositories under the `kagapa-blr` GitHub account. The parent repository records the exact commit of each component.

## Initialize the complete workspace

```bash
git clone --recurse-submodules <your-parent-repository-url>
cd Nudi-Kacheri-Suite
git submodule update --init --recursive
```

If the workspace was already cloned, synchronize and initialize all configured submodules:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

## Build Desktop Editors on Linux

Build requirements and supported dependencies are documented in [`build_tools/README.md`](build_tools/README.md) and [`build_tools/PROJECT_SETUP.md`](build_tools/PROJECT_SETUP.md). The usual Desktop Editors build command is:

```bash
cd build_tools/tools/linux
python3 ./automate.py desktop
```

Build output is generated below `build_tools/out/`. The exact output path depends on the platform and build configuration.

## Where to customize

- Native windows, startup, file handling, themes, and packaging: `desktop-apps`
- Editor toolbar, menus, panels, dialogs, and web UI: `web-apps`
- Editor commands, APIs, and document runtime behavior: `sdkjs`
- File formats, rendering, import/export, and conversion: `core`
- Spell-checking and hyphenation: `dictionaries`
- Build dependencies, automation, and packaging pipeline: `build_tools`

For the detailed change-routing guide and submodule workflow, see [`DEVELOPER.md`](DEVELOPER.md).

## Submodule change workflow

Commit source changes inside the owning submodule first. Then update and commit that submodule’s Git pointer in this parent repository:

```bash
cd web-apps
git add <changed-files>
git commit -m "Describe the customization"
cd ..
git add web-apps
git commit -m "Update web-apps customization"
```

Do not commit generated output from `build_tools/out/`. Validate the complete user flow after cross-repository changes: build, launch the desktop application, exercise the customization, save/export a representative file, and reopen it.

## License

This workspace contains components licensed under the GNU AGPL v3.0 and separate third-party licenses. Review the license files in the parent repository and each submodule before distributing a customized build.