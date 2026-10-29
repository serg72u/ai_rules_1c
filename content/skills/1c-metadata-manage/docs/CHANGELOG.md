# 1c-metadata-manage — sync history of the vendored scripts

The PowerShell tools under `tools/` are vendored from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills) (MIT). This file holds the per-document sync notes that used to close each `docs/*.md` (what each upstream sync changed, what was cross-checked and what stays deliberately local). It is maintainer history — nothing loads it during a task; the operational description of every tool lives in the domain doc named by each heading.

## bsp-manage.md

### Upstream sync `2026-07-30`

Re-checked against upstream `epf-bsp-init` / `epf-bsp-add-command` at the `2026-07-30` sync: **no drift** — kinds, default command types, the `СведенияОВнешнейОбработке` skeleton, `Назначение` / `Модификатор` sections, `НСтр` literals for additional commands, and every handler template (`ВыполнитьКоманду` branches, `Печать` with `УправлениеПечатью.СведенияОПечатнойФорме`) still match Sections 1–2 below. Nothing to port; both skills remain no-script upstream.

### Earlier Additions (upstream `w-2026-05-17`)

The upstream `cc-1c-skills` skills `epf-bsp-init` and `epf-bsp-add-command` are no-script (the agent does the work directly via Read / Edit / Glob / Grep). Their content is already covered by Sections 1–2 of this document, in English. Cross-checked against upstream `w-2026-05-17`:

- Kind mapping (six kinds: `ДополнительнаяОбработка`, `ДополнительныйОтчет`, `ЗаполнениеОбъекта`, `Отчет`, `ПечатнаяФорма`, `СозданиеСвязанныхОбъектов`) — aligned.
- Default command type per kind — aligned.
- Free-form command types (open form, client method, server method, form filling, safe-mode script) — aligned.
- `СведенияОВнешнейОбработке` skeleton, `Назначение` section for assignable kinds, `Модификатор` for `ПечатнаяФорма` — aligned.
- Server handlers (`ВыполнитьКоманду` for `ЗаполнениеОбъекта` / `СозданиеСвязанныхОбъектов` / global processors, `Печать` for `ПечатнаяФорма`) — aligned.

No script files were brought into `tools/` — the operations are pure module-text edits performed by the agent, which is how upstream ships them as well.

## cf-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `cf-edit` v1.12 → **v1.31**, `cf-init` v1.3 → **v1.17**, `cf-validate` v1.4 → **v1.13**, `cf-info` v1.4 → **v1.10**.

Deltas kept: the `.dev.env SUPPORT_GUARD` gate ahead of `.v8-project.json` `editingAllowedCheck`; `NEW_OBJECT_POSITION` from `.dev.env` inside upstream's own `Get-NewObjectPosition` (a set key decides by itself, an unrecognised value means `end`); the auto-validation call to the sibling `cf-validate.ps1` in the same `scripts` directory, with the validator's exit code propagated; processing instructions kept literal by the shared `Finalize-XmlText`.

Delta **dropped**: the fork's own `$byName` / `Test-OrderSensitiveType` ordering block. Upstream implements the same `newObjectPosition` ordering, including the order-sensitive kinds, and the two implementations cannot coexist in one file.


### Local fix `2026-10-19` — `add-childObject` follows `NEW_OBJECT_POSITION`

`cf-edit` v1.11 → **v1.12** (support ticket 4e6f4e67, item 4): `add-childObject` always inserted a new entry by name inside its kind group (culture-aware `-gt`), whatever `NEW_OBJECT_POSITION` said. It now resolves the setting as `meta-compile` does — `end` (default) after the last object of the kind, as Configurator; `byName` with the 1C-tree comparator; order-sensitive kinds never sorted. Pinned by `tools/tests/metadata-tools-regression.ps1`.

### Local fix `2026-10-01` — sibling validators are found

The auto-validation of `cf-edit`, `interface-edit`, `subsystem-compile` and `subsystem-edit` looked for the validator at upstream's `..\..\<validator>\scripts\` path, which does not exist in this layout, so it was skipped silently; `xdto-edit` looked for `xdto-decompile` / `xdto-compile` / `xdto-validate` the same way and refused every structural operation. All five now take the sibling script from their own `scripts` directory.

### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `cf-edit` v1.4 → **v1.11**, `cf-info` v1.2 → **v1.4**, `cf-init` v1.2 → **v1.3**, `cf-validate` v1.3 → **v1.4**.

- **`cf-edit` now enforces the vendor support gate** — it refuses to edit a configuration whose "possibility of modification" is off, and points at `support-edit`. See [support-manage.md](support-manage.md).
- **`cf-info` reports the support state** of the configuration (read from `Ext/ParentConfigurations.bin`) — whether it is on vendor support and whether modification is allowed.

### Earlier Additions (upstream `w-2026-05-17`)

The PowerShell scripts under `tools/1c-cf-manage/scripts/` were refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights:

#### `cf-init`

- Generates a default panel layout aligned with stock ERP/БП ≥ 8.3.24: open windows on top, sections on the left. Without this layout the web client renders sections as icons only — `web-test` could not see them. Modern interface mode + 8.2-style backward compatibility is the default.

#### `cf-edit` — new operations

- **`set-panels`** — configure the configuration panels via JSON. Aliases by name (`sections`, `open`, `favorites`, `history`, `functions`), panel stacking via `{group: [...]}`, multiple panels side-by-side. Russian aliases (`Открытых`, `Разделов`, `Избранного`, `История`, `Функций`) are accepted and silently mapped to canonical English aliases.
- **`set-home-page`** — rewrite the home page: one or two columns, list of forms with height / visibility / role overrides.
- **`add-childObject`** — now requires the object file to exist on disk; otherwise the script fails with a hint to call `/meta-compile`, `/role-compile`, `/subsystem-compile` first. Previously `Configuration.xml` could end up referencing a missing file and the platform would silently refuse to load. The legacy mode (root XML lost while files survive) is preserved for the rare rollback scenario.

#### `cf-info`

- Configuration overview shows the panel layout (panel stacks within one side and side-by-side panels). With `-Section home-page` — full home page contents with layout and role overrides.

#### `cf-validate`

- Cross-references `Form` references in the home page and in default-form properties of configuration objects — broken links are now caught at validation time, not at load time.
- Platform 8.5 support — new compatibility-mode and interface-mode values plus the new XML header format. (Same upgrade in `cfe-validate`, `epf-validate`, `skd-validate`.)

## cfe-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `cfe-borrow` v1.12 → **v1.58**, `cfe-validate` v1.5 → **v1.31**, `cfe-init` v1.2 → **v1.13**, `cfe-diff` v1.0 → **v1.8**, `cfe-patch-method` v2.5 → **v2.14**.

**`cfe-borrow` is upstream v1.58 taken whole, then the fork's contracts re-implemented on top.** v1.13…v1.58 is a rework of the very code the fork had patched (BaseForm indenting, choice-parameter link rewriting, adopted tabular sections, module handling, insertion order), so a hunk-wise merge of two implementations of one behaviour would have produced a file neither side had ever tested. What the fork contributed conceptually is now upstream's: the byte-vs-character `Configuration.xml` fix, BaseForm indentation, by-name section selection, `newObjectPosition` ordering, tabular-section column merging, dropped-link reporting.

Re-implemented deltas, each marked `Local:` in the file and covered by `tools/tests/cfe-borrow-regression.ps1` (**13/13**):

- `-BorrowMainAttribute Only` — the form is borrowed with its main attribute and **no cascade**: the borrowed attribute set is empty and every data path is stripped from both parts of the form (an empty binding root is passed to `Strip-FormBindings`). The validation accepts `Only` next to `Form` / `All`.
- `<Type>` blocks of borrowed attributes and tabular-section columns are written the way the Configurator dump has them — one child element per line, one indent level deeper (`Format-TypeXml`); the form's own main attribute keeps its `<Type>` on a single line.
- An element-level `CommandSet` (with its `ExcludedCommand` list) is kept in the form and in `BaseForm`; upstream drops every nested `CommandSet`.
- No empty `Module.bsl` is created for a borrowed form, and no empty `Ext/Form` directory is left behind — the Configurator creates neither, and an empty module makes later `form-edit` / `remove-form` runs treat the module as existing.
- A borrowed `Document` carries an empty `<RegisterRecords/>` after its properties.
- References to styles that stayed in the source configuration are stripped from `AutoCommandBar`, `ChildItems` and the form properties (platform style items such as `ImportantColor` stay), so the extension does not fail to load with «элемент стиля не найден».
- An unknown sub-object is refused with a single-line `[ERROR] Source attribute not found: <name>` on stderr — `Write-Error` wrapped the line and prefixed the script path, which broke the operator-facing text.

Delta dropped: the internal `-BorrowMainMode` / `-KeepObjektBindings` switches (undocumented, superseded by upstream's `$mainAttrName` model).

The upstream harness for this tool reports **18/65**: the other 47 cases encode upstream's own variants of the behaviours above (multi-line form-attribute `<Type>`, mandatory `Module.bsl`, `CommandSet` stripping, different `RegisterRecords` handling). Our contract suite is the authority here — it encodes what our projects load and what the Configurator writes.

`cfe-validate` / `cfe-init` / `cfe-diff` keep the `.dev.env` platform chain and the sibling-validator dispatch; the new checks of v1.31 (form path roots, `AdditionalColumns`) are in.


### Upstream sync `2026-10-28` — `cfe-patch-method` v2.5 → **v2.14**

Source `662e752e` (same commit as the tier-1 database tools). **No local deltas**: the vendored copy was byte-identical to the pinned script, so this is a plain refresh plus `-Check`/`-Actualize` behaviour that the docs already promised but the old script did not deliver.

- `-Check` now compares the **parameter list** and the return type of every controlled method against the current original, not only the unmarked body — a vendor parameter rename is caught while the body still matches.
- The comparison key follows the platform's measured rule (case-insensitive marker counter; the marker line is returned unchanged), so a re-run no longer reports phantom drift.
- BSL keywords are recognized in both spellings (`And`/`Or`/`Not` beside `И`/`Или`/`НЕ`), including `Или` inside the keyword table.
- Interception of a **common form's own module** is supported: `-ModulePath "CommonForm.<Name>"`.
- Borrowed-object modules and the extended-property mark are handled by `cfe-borrow`/`cfe-validate` in the same family.

### Local fix `2026-10-28` — choice-parameter and type-link paths

`cfe-borrow` v1.11 → **v1.12** (GitHub issue 33, platform 8.3.27): a borrowed form kept readable `ChoiceParameterLinks` / `TypeLink` paths (`Объект.<Attribute>`, `Items.<Table>.CurrentData.<Field>`). The platform rejects the load with «Неверный путь к полю» when that attribute is not borrowed. The link encoder from upstream `cfe-borrow` is now applied to both the form and `BaseForm`. A borrowed attribute stays readable (`All`, or `Form` when the form references it). Anything else is written as the source form's link code: `<source main id>/0:<uuid>`, a standard-field code, or the source main-attribute id alone when the main attribute is borrowed but the target is not. The extension's own main-attribute id `1000001` is not used. A path that cannot be coded is removed, with a warning. The rest of the local borrow behaviour is unchanged. Pinned by `tools/tests/cfe-borrow-regression.ps1`.

### Local fix `2026-10-19` — an element `CommandSet` keeps its `ExcludedCommand` list

`cfe-borrow` v1.10 → **v1.11** (support ticket 2b392369, an addition to 4e6f4e67): `Borrow-Form` removed every `ExcludedCommand` inside `ChildItems`, so an element whose base form excludes standard commands (a tabular-section table without `Add` / `Copy`) got an empty `<CommandSet></CommandSet>` in the form and in `BaseForm`. Configurator keeps the element's `CommandSet` with its `ExcludedCommand` list in both parts and drops an empty one on load, so the check dump after a partial load differed from the source directory. The element `CommandSet` is now copied as is. Pinned by `tools/tests/cfe-borrow-regression.ps1`.

### Local fix `2026-10-18` — borrowing follows Configurator 2.20

`cfe-borrow` v1.9 → **v1.10**, `cfe-validate` v1.4 → **v1.5** (support ticket 4e6f4e67):

- A borrowed form's `Forms/<Form>.xml` gets the `InternalInfo` `PropertyState` (`Form` / `Extended`) instead of an empty `<InternalInfo/>`.
- `CommandSet` no longer ends the form-property copy: it, its `ExcludedCommand` list and every property after it up to `AutoCommandBar` reach the form and `BaseForm`.
- One attribute (`Type.Name.Attribute.A`) or one tabular-section column (`Type.Name.TabularSection.T.Attribute.A`) can be borrowed: source uuid, source `<Type>`, type objects as shells, no cascade. `-BorrowMainAttribute Only` adds the form attribute `Объект` without the cascade and strips the data paths in both parts.
- A new extension `ChildObjects` entry is appended to the end of its type group instead of being sorted by name. `cf-edit` / `meta-compile` keep the by-name order (APK:1108) for the main configuration.
- No empty `Ext/Form/Module.bsl` for a borrowed form; an existing module is kept.
- A borrowed `Document` gets an empty `<RegisterRecords/>`.
- `cfe-validate` check 12 reports an unborrowed `CommonPicture` or a `StyleItem` outside the platform style list as an error (the form will not load with `Configuration.xml`); check 11 no longer warns about a borrowed form without `Module.bsl`. `cfe-borrow` borrows a referenced style item when the source file exists, and strips a `style:` reference that is neither borrowed nor a platform style item.
- `Detect-FormatVersion` takes the string length after reading. A short UTF-8 `Configuration.xml` with Cyrillic no longer throws on `Substring` (file length is bytes).

Pinned by `tools/tests/cfe-borrow-regression.ps1`.

### Local fix `2026-10-01` — `BaseForm` keeps multi-line texts

`cfe-borrow` shifted every line of the borrowed `AutoCommandBar` / `ChildItems` one tab deeper for `BaseForm`, including the continuation line of a multi-line `v8:content`, so the copy of the base form no longer matched the configuration (support ticket bc42b89c). Only line breaks between two tags are indentation now; the texts keep their bytes.

### Recent Additions (upstream sync `2026-07-30`)

The PowerShell scripts under `tools/1c-cfe-manage/scripts/` were refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights of this sync (previous base: late May 2026):

#### `cfe-patch-method` — v1.1 → v2.5, the biggest change in this group

- **Source-aware generation**: the original method is read from the source configuration (`-ConfigPath`, auto-detectable via `.dev.env` `EXPORT_PATH`), so the interceptor gets the real signature, context directive and — for `ModificationAndControl` — the full original body. `ModulePath` may now be a path to the source `.bsl` instead of a logical name.
- **`Instead` (`&Вместо`)** interceptor type with a `ПродолжитьВызов(...)` scaffold; works for functions too.
- **Drift control** — `-Check` / `-Actualize` over controlled (`&ИзменениеИКонтроль`) methods, extension-wide or narrowed to a module / method. Re-applies your `#Вставка` / `#Удаление` edits onto a changed vendor original, reports what moved, what conflicted and what the vendor has already adopted. Conflicts land in a merge workspace with anchored `conflict.md` files instead of being lost. **Run `-Check` after every vendor update** — the platform never reports this drift itself.
- Repeated interception of an already patched method is skipped instead of duplicated.

Full description in section 4 above.

#### `cfe-borrow` — borrowing forms now matches Configurator output

- The borrowed form gets the **complete `ChildItems` tree** (not the previously empty skeleton). Loadable into the target base without manual XML fixes for complex ERP forms.
- **Dependencies are auto-borrowed**: shared pictures, style elements, enums, and enum values used by the borrowed form. No more cascade of "object not found" errors after the first load.
- `DataPath`, `Events`, `TitleDataPath`, `TypeLink`, `CommandName` are stripped from the borrowed form (these caused "invalid data path" / "event not loaded" errors on real ERP forms, also for command-bar buttons).
- Form-level properties (`AutoTitle`, `WindowOpeningMode`, …) are preserved both in the main section and in `BaseForm`.
- **`-BorrowMainAttribute`** (`Form` or `All`) — borrows the form's main attribute (`Object`) and transitively all its attributes, tabular sections, and dependent types. Closes the manual-collection workflow when adding an attribute to a borrowed form.

#### `cfe-validate`

- New checks for borrowed-form structure, their dependencies (shared pictures, style elements, enums), and the extension's own subobjects (attributes, tabular sections, enum values, forms).
- False positives removed: `DataPath` / `TitleDataPath` inside `BaseForm` are correct (Configurator emits them); the extension's own subobjects (own attributes, own enum values) are no longer validated as borrowed.

#### `cfe-init`

- Interface mode and compatibility mode are inherited from the base configuration (resolved via `-ConfigPath`). The extension matches the base's behaviour by default.

#### Vendor support

An extension is the **default answer** when a typical object on vendor support needs a change: support state stays untouched and vendor updates keep flowing. The mutating tools of this skill now enforce that — they refuse to edit a locked object directly and point here. See [support-manage.md](support-manage.md).

## db-manage.md

### Upstream sync `2026-10-28` — the rest of the family

Source `662e752e`. `db-create` v1.10 → **v1.21**, `db-dump-cf` v1.12 → **v1.23**, `db-dump-dt` v1.11 → **v1.22**, `db-load-cf` v1.13 → **v1.25**, `db-load-dt` v1.12 → **v1.23**, `db-run` v1.7 → **v1.16** (the four tools of the first pass are described above).

The same two deltas as the first pass: `.dev.env` (`PLATFORM_ARGS` / `IBCMD_ARGS` / `PLATFORM_PATH`) ahead of `.v8-project.json`, and `PLATFORM_PATH` accepted in the install-directory shape with the `bin\1cv8.exe` fallback. Upstream's own change of this release — the database record's `v8path` wins over the root one, which is why six scripts now consult `Find-ProjectDatabase` first — is kept.


### Upstream sync `2026-10-28` — the four tier-1 database tools

Source: `Nikolay-Shirokov/cc-1c-skills` at `662e752e` (2026-10-28), three months of commits past the `ecd289fe` pin. Ported by three-way merge (`git merge-file`, base = the pinned script), one tool at a time, with every local delta re-applied on top. Versions: `db-dump-xml` v1.14 → **v1.29**, `db-load-xml` v1.19 → **v1.38**, `db-load-git` v1.18 → **v1.36**, `db-update` v1.14 → **v1.28**.

Why these four first: they are on the hot path of every project, EDT included — `/loadfrom1cbase`, `/getconfigfiles`, `/update1cbase` and `/deploy-and-test` drive them regardless of source format, unlike the XML metadata tools.

Behaviour that changed for our workflows:

- **Platform launch** — the platform is awaited until it exits, not until its pipes close; inherited descriptors are passed explicitly; a failure *before* the launch is no longer reported as success; a batch command embedded in `AdditionalV8Arguments` is rejected instead of silently swallowing the real one.
- **`db-dump-xml -Mode Changes`** — no `-force` any more: a dump-format-version mismatch surfaces as a platform refusal instead of a silent full re-dump that the caller reads as an incremental export. An object list (`-Objects` / `-ObjectsFile`) makes the run partial by itself.
- **`db-load-xml`** — `-Files` / `-ListFile` imply `Partial`; `-AllExtensions` with a partial load is refused on both engines; after an extension load the platform is asked whether the extension is applicable (`/CheckCanApplyConfigurationExtensions`, separate launch — one command line executes only its last batch command).
- **`db-load-git -DryRun`** — warns about deletions a partial load cannot apply (and exits non-zero when they are the only changes), and warns that a `Configuration.xml` in the list means a full load.
- **Registry/`.v8-project.json` (not used here)** — `-RepositoryPath/User/Password`, `-ObjectsFile`, per-database `v8path`, `v8args`; new `-StrictLog` / `-NoApplyCheck` switches. `.dev.env` still wins over the registry (below).

Local deltas re-applied (unchanged in intent from `NOTICE.md`): `.dev.env` first in the extra-args lookup (`PLATFORM_ARGS` / `IBCMD_ARGS`) and in the v8path lookup (`PLATFORM_PATH`); `PLATFORM_PATH` accepting the install directory with a `bin\1cv8.exe` fallback; `/DumpResult` appended to every batch launch and read as the batch verdict next to the exit code.

Local deltas re-worked:

- **`db-update` silent rejections** — upstream replaced the fork's own pattern scan with `Find-SilentRejections` (a wider list, minus the generic `Ошибка обновления конфигурации базы данных` header, and gated on `-StrictLog`). The wider list and the new extension apply-check are taken; the generic header is re-added, and the fork's **hard** escalation is kept — a log rejection fails the run regardless of `-StrictLog`. Verified: the doc and `designer-batch-checks.md` promise a non-zero verdict, so a warning-only path would have been a regression.
- **`db-update -Dynamic auto|disable`** — dropped. Upstream's own fix accepts `on` / `off` / `yes` / `no`, so the `-File`-safe spelling exists upstream (`-Dynamic off` → `-UpdateDBCfg -Dynamic-`, checked on 8.3.27). Two spellings for one thing is worse than one.

Verification (real platform, 8.3.27.2130, throwaway file base in the scratch directory): PowerShell parse of all four scripts; `db-create` → `db-dump-xml -Mode Full` → `db-load-xml -Mode Full -UpdateDB` → `db-update -Dynamic off`, all exit 0 with the `/DumpResult` verdict read; `db-load-git -DryRun` on a fixture git repo exercised the `Configuration.xml` full-load warning and the deletion warning + refusal. `.dev.env` resolution was exercised implicitly (no `-V8Path` passed, `PLATFORM_PATH` came from the project). No db tool has a Python port, so `tools/tests/python-ports-regression.py` is unaffected.

The `NOTICE.md` pin stays at `ecd289fe`: the other 64 scripts were not re-synced, and a pin is a property of the tree, not of four files. Their source commit is recorded in `NOTICE.md` per file.

### Local fix `2026-10-20` — `-Dynamic` takes `auto` / `disable`

`db-update` v1.13 → **v1.14** (support ticket 7557c7cf): through `powershell.exe -File` a lone `-Dynamic "-"` never reaches the script — Windows PowerShell 5.1 exits with a non-zero code and no output, so the agent saw a silent failure instead of `/UpdateDBCfg -Dynamic-`. `-Dynamic` now also accepts `auto` and `disable` (the ibcmd spellings), mapped to `+` / `-`; `+` / `-` keep working in-process, and `-Dynamic:-` works through `-File` as well. Checked on a throwaway 8.3.27.2130 file base: `-File … -Dynamic disable` → `-Dynamic-`, `auto` → `-Dynamic+`, both exit 0.

### Recent Additions (upstream sync `2026-07-30`)

The PowerShell scripts under `tools/1c-db-ops/scripts/` were refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights of this sync (previous base: late May 2026):

- **`db-dump-dt` / `db-load-dt`** — new: full infobase snapshot (configuration + data). See sections 9 and 10.
- **All `db-*` / `epf-*`** — `-AdditionalV8Arguments` / `-AdditionalIbcmdArguments` with per-engine validation and secret masking (see *Common Parameters*); project-wide defaults via `v8args` / `ibcmdargs` in `.v8-project.json`.
- **Platform resolution** — unified across the scripts: explicit `-V8Path` → project config → auto-detect the newest `C:\Program Files\1cv8\*\bin\1cv8.exe`. Locally patched so the project config is read from **`.dev.env`** (`PLATFORM_PATH`, `PLATFORM_ARGS`, `IBCMD_ARGS`, `SUPPORT_GUARD`) before upstream's `.v8-project.json`, and so a **version install directory** — the shape `PLATFORM_PATH` uses — resolves via `bin\1cv8.exe`.
- **`db-load-xml`** — strict log parsing. Catches "Неверное свойство объекта метаданных", "Неизвестное имя типа" and similar messages that the platform writes to the log despite a formal "success" exit. Previously a partial silent metadata loss was reported as a green run.
- **`db-load-xml` / `db-load-git`** — `-UpdateDB` flag combines load + database update in a single Configurator launch (was two separate calls).
- **`db-load-git`** — picks up changes to HTML help (`ru.html` and similar) via partial load even without the accompanying `Help.xml` in the commit. Previously such edits were silently dropped and the help text in the base stayed stale. Fixed search for changed files when sources live in a nested folder of the repo (`src/cf` etc.); path normalisation for the configuration directory is corrected.
- **db-list** — already fully described in Part 1 of this doc (registry of `.v8-project.json`). It is a no-script skill in upstream — the agent reads / writes the JSON directly. No script files were added under `tools/`.

## epf-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `epf-build` v1.12 → **v1.26**, `epf-dump` v1.11 → **v1.22**, `epf-validate` v1.2 → **v1.11**, `epf-init` v1.1 → **v1.9**, `erf-init` v1.1 → **v1.9**.

Deltas kept: the `.dev.env` platform chain in the three `epf-*` tools; the PowerShell-7-safe stub launcher (`$psExe` — `pwsh` under Core) in `epf-build`, on top of upstream's reworked wait (the stub itself is awaited, not `-Wait` on a process tree that a leftover crashpad child would hold); `-FormatVersion` and the fork's `-BspForms` switch side by side in `erf-init`.

Upstream absorbed the fork's save pipeline in `epf-init` / `erf-init` (`write_xml_file` / `write-bom` helpers).


### Local fix `2026-10-07` — common forms in external reports

External reports and data processors may name configuration common forms in `DefaultForm` / `DefaultSettingsForm` / `DefaultVariantForm` (БСП reports: `CommonForm.ФормаОтчета`). Checked on 8.3.27.2130: such an ERF builds in an infobase that has the form, and the reference survives a dump in another infobase with a same-named form (it is stored by name); without the form the build fails with «Неизвестный объект метаданных».

- `epf-validate` check 5 accepts `CommonForm.<Name>` in `DefaultForm` instead of warning about an unexpected prefix.
- `stub-db-create` creates an empty managed common form for every `CommonForm.<Name>` named by those properties, so a build without an explicit infobase passes.
- `erf-init` gained `-BspForms`: the three main forms are the БСП common forms.

### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `epf-build` v1.0 → **v1.12**, `epf-dump` v1.0 → **v1.11**, `stub-db-create` v1.0 → **v1.7**, `erf-init` → **v1.1** (full ERF scaffold — the local copy was a truncated variant).

- **Build / dump via `ibcmd`** — full EPF *and* ERF assembly and disassembly without launching the Designer, when the platform path points at `ibcmd`.
- **`-AdditionalV8Arguments` / `-AdditionalIbcmdArguments`** with per-engine validation and secret masking — same contract as the `db-*` tools ([db-manage.md](db-manage.md)).
- Platform resolution unified with the `db-*` tools (explicit path → `.dev.env` → `.v8-project.json` → auto-detect), locally patched to accept the version install directory shape used by `.dev.env` `PLATFORM_PATH`.
- Autonomous external objects (EPF/ERF) are explicitly exempt from the vendor support gate — they are never part of a configuration on support.

### Earlier Additions (upstream `w-2026-05-17`)

The PowerShell script `tools/1c-epf-validate/scripts/epf-validate.ps1` was refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights:

- **Format version auto-detection** from the nearest `Configuration.xml` (8.3.27+, 8.5).
- **Platform 8.5** support across `epf-validate` and `erf-validate` (new compatibility-mode and interface-mode values, new XML header format).
- **Universal validator improvements** — one-liner output by default (`-Detailed` for the full per-check trace); accepts both an XML file and a folder path as the primary argument; universal `-Path` parameter alongside legacy `-ObjectPath`.
- The same script handles `erf-validate` — upstream `erf-validate` is a thin pass-through to `epf-validate.ps1`, the script auto-detects `ExternalReport` vs `ExternalDataProcessor` from the root XML element. No separate `erf-validate.ps1` is shipped.

## form-manage.md

### Upstream sync `2026-10-28` — `form-info`

Source `662e752e`. `form-info` v1.5 → **v1.14**. No local delta beyond the processing-instruction branch of the shared compact-tag writer.


### Upstream sync `2026-10-28` — `form-edit` v1.6 → **v1.39**, `form-validate` v1.10 → **v1.36**

Source `662e752e`. Both tools grew far past the pin (form-edit 1 549 → 9 535 lines: the element emitter is now shared with `form-compile`, a per-element event catalogue with handler stubs, `set`/`rename`/`remove` operations, form properties, conditional appearance, command interface).

Local deltas dropped as **superseded** by upstream — each one is now upstream behaviour, verified against the new code:

- the event-suffix map and `knownEvents` additions (`OnEditEnd` → `ПриОкончанииРедактирования`, `Selection` → `Выбор`, `DragEnd`, `Tuning`, `StartListChoice`): upstream ships a per-element event catalogue with the Russian names, signatures and stub generation;
- button/command child order and `Command.Representation = TextPicture`: upstream has `childTagOrder` / `formRootTagOrder` tables and per-element enumeration maps with the `PictureAndText ↔ TextPicture` aliases;
- the `Module.bsl` handler check (local 13): upstream **Check 22** does it better — events, command actions, borrowed-form scope, dead-code warnings.

Local deltas kept:

- `.dev.env SUPPORT_GUARD` before `.v8-project.json` `editingAllowedCheck` (vendor support gate) — verbatim from the other mutators.
- form-edit's **DynamicList source refusal**: a `DynamicList` attribute with neither `settings.mainTable` nor `settings.query` is still rejected before writing. Upstream fills a `MainTable` heuristically but never refuses the empty case, and the form does not open.
- form-validate **Check 12**: `cfg:Catalogs.X` — an export folder name where a type belongs — is an error, not an "unrecognized prefix" warning.
- form-validate **Check 12b**: the main attribute of a default object / record form belongs to the owner; a zero-length `Description` / `Code` is not bound.
- form-validate **Check 26** (was 13): duplicate declaration (error), missing compilation directive, wrong context and handler arity. The "handler not found" branch was **removed** so Check 22's finding is not reported twice.
- form-validate **Check 27** (was 14): `РеквизитФормыВЗначение` / `ЗначениеВРеквизитФормы` require the form context on the server.
- form-edit **compact empty tags in literal XML text** — upstream's writer compresses ` />` → `/>` and protects CDATA and comments but **not** processing instructions, so a `<?…<Tag />…?>` was rewritten. The fork's alternation keeps `<?…?>` as well; without it the port silently reintroduced the defect our own regression case pins (it caught this one).

Verification: upstream's own snapshot harness (`tests/skills/runner.mjs`, PowerShell runtime) run against the ported files in a hybrid skills tree — `form-edit` **117/117**, `form-validate` **58/58** (2 skipped by fixture prerequisites). Five extra cases authored for the kept deltas (DynamicList without source, duplicate handler, client-context conversion, export-folder type, client event on a server procedure) pass alongside them: **63/63**.

`tools/tests/metadata-tools-regression.ps1` needed its `form-validate` case updated to the new contract — the handler checks are now numbered 26 / 27, a missing handler is **Check 22's error** rather than a warning (the platform raises "method not found" when the form opens), and the format-version finding names the form descriptor instead of `Configuration.xml`. The compact-tag case (`form-edit` inside a processing instruction) stayed as it was and is what exposed the missing delta above.

### Local fix `2026-10-14` — input-field events and pass-through keys

`form-compile` refused the input-field events `Tuning` (Регулирование), `Opening` and `StartListChoice`, which vendor forms use (ZUP dump: 333, 195 and 2 input fields; support ticket 0225a24c, ERP 2.5 `Документ.НачислениеЗарплаты.ФормаДокумента`). They are allowed for `input` in the PowerShell and Python compilers and in `form-edit`, with the handler suffixes `Регулирование` / `НачалоВыбораИзСписка`. The pass-through scalars of `genericScalars` (`markNegatives`, `threeState`, `choiceListHeight` and 30 more) were written to `Form.xml` yet reported as «unknown key … ignored»; they now count as known keys. `form-compile-dsl.md` lists the `input` and `picField` events separately, as the compiler checks them.

### Local fix `2026-10-01` — button and command order, command `Representation`

`form-edit` wrote a button's `CommandName` before its `Representation`, a command's `Action` before its `Picture` and a command's picture-and-text `Representation` as `PictureAndText`; the platform refused such a borrowed form with «Исключение XDTO произошло при чтении файла» (support ticket bc42b89c, 8.3.27.2170). Button children now follow the platform order (`Type`, `Visible`, `Representation`, `DefaultButton`, `Enabled`, `CommandName`, `Picture`, `Title`, `LocationInCommandBar`) and command children `Title`, `Shortcut`, `Picture`, `Action`, `Representation`, taken from every `Form.xml` of the ZUP dump; a command's `PictureAndText` is written as `TextPicture` (and a button's `TextPicture` as `PictureAndText`). `form-validate` check 15 reports a `Representation` outside the element's own enumeration. `form-compile` still writes a command's `Action` before `Shortcut` / `Picture` and a button's `Representation` after `CommandName` — not changed here.

### Local fix `2026-09-25` — vendor-shaped defaults, closed enum values, Form.xml against Module.bsl

Rules taken from the static checks of [AzeevAN/mcp-1c](https://github.com/AzeevAN/mcp-1c) (Apache-2.0; rules re-implemented, no code copied) and calibrated on the ERP (8.3.27) and ZUP Designer dumps so that vendor forms raise no errors.

- **`form-compile` v1.176** (both runtimes): the from-object document choice form sets `ChoiceMode` on its list table like the catalog one; the `document.item` preset (hard-coded default and `presets/erp-standard.json`) writes `AutoTime=CurrentOrLast`, `UsePostingMode=Auto`, `RepostOnWrite=true` in Configurator order; `Description` is bound only when `DescriptionLength > 0`. `windowOpeningMode` / `autoTime` / `usePostingMode` accept only the platform enum values — `form-dsl-spec.md` listed `Modeless`, `Current`, `Postings`, `Movements`, which do not exist. A missing `Configuration.xml` is a `[WARN]` naming the assumed 2.17 instead of a silent guess.
- **`form-add` v1.12** (both runtimes): `-FormName` must be a 1C identifier (exit 2 before any path or XPath is built from it); the object form of an information register names its `RecordManager` main attribute `Запись`, as the Configurator does.
- **`form-validate` v1.10**: version equal to `Configuration.xml` (check 1); `cfg:Catalogs.X` is an error (12); owner type of the main attribute in default object / record forms and no zero-length `Description` / `Code` binding (12b); handlers against `Module.bsl` (13 — duplicate declaration is an error, the rest are warnings because ERP ships them); `РеквизитФормыВЗначение` / `ЗначениеВРеквизитФормы` only in `&НаСервере` (14).

### Local fix `2026-09-14` — compact XML across the reported writers

`form-edit.ps1` normalizes empty tags to Configurator's `<Tag/>` spelling and retains the input EOL. The same save-time behavior covers `form-add`, `remove-form`, parent registration by `form-compile`, `cf-edit`, object registration/merge by `cfe-borrow`, `subsystem-edit`, parent registration by `subsystem-compile`, `interface-edit`, `add-template` and `add-help`. `skd-edit` already normalized tags; its normalization now protects CDATA, comments and processing instructions. No shared runtime dependency was added.

Focused regressions exercise each reported writer on temporary fixtures. Command-add checks compare unrelated form text byte-for-byte for LF and CRLF; template checks cover absolute/relative XML paths and the existing name/EPF lookup. The remove-form quarantine and rollback remain intact.

### Local fix `2026-09-10` — DynamicList settings in `form-edit`

`form-edit` v1.5 wrote a `DynamicList` attribute as Type only. The platform then opened the form with «не задан ни текст запроса, ни основная таблица». `form-edit` v1.6 requires `settings.mainTable` or `settings.query` and emits `<Settings xsi:type="DynamicList">` (`ManualQuery`, `DynamicDataRead`, optional `QueryText`, `MainTable`). `form-validate` v1.9 reports the same gap on an existing form. Full DCS list settings remain `form-compile`.

### Python runtime for `form-remove` (`2026-09-04`)

`remove-form.py` is vendored from the same upstream repository, pinned at commit `ecd289fe11733028d87b55284ea9fb5feff8f513` — the state the PowerShell family below was synced from, so both runtimes are the same tool generation. It exists so a Linux / macOS install is not left with a script it cannot run.

The upstream port needed the same local hardening the PowerShell script carries, and one ordering fix on top: upstream rejects `-DryRun` as an unknown argument, deletes without `-Force`, and deletes the form files **before** parsing the root XML, so a parse failure leaves a half-removed tree. The vendored copy follows the shipped contract instead — parse → plan → gate → atomic root-XML write → delete. Both runtimes are pinned against each other by `tools/tests/python-ports-regression.py`.

The safety hardening is applied to **both** runtimes, so a Windows user is not left with the weaker tool: identifier validation, path containment and the transactional mutation path live in `remove-form.ps1` as well.

The Python port keeps upstream's round-trip style preservation (BOM, EOL, `encoding` case, `<Tag/>`). Since the local 2026-09-14 fix, `remove-form.ps1` also keeps the input EOL and compact empty tags when it stages the parent XML; the quarantine and rollback sequence is unchanged. The `ChildObjects` indentation is identical in both.

Licence: the vendored upstream code is MIT; the full notice ships with the skill as [`NOTICE.md`](../NOTICE.md) and is installed alongside it.

### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `form-compile` v1.23 → **v1.175**, `form-add` v1.5 → **v1.11**, `form-edit` v1.0 → **v1.5**, `form-info` v1.3 → **v1.5**, `form-validate` v1.6 → **v1.8**, `form-remove` → **v1.4**.

- **`form-compile` is the big one** — 150+ upstream releases since the previous base. The DSL grew well past what [form-compile-dsl.md](form-compile-dsl.md) describes: dynamic-list settings (filters, order, conditional appearance, groupings, calculated fields, typed parameter values), forgiving platform types (`StandardPeriod`, `StandardBeginningDate`, `UUID`), roles by GUID for borrowed / extension forms, `SettingsStorage`. **The complete grammar is vendored as [`form-dsl-spec.md`](../tools/1c-form-compile/form-dsl-spec.md)** — read it when a key is missing from the local reference.
- **Support gate** — `form-compile`, `form-edit`, `form-add` refuse to touch a form of a locked object of a typical configuration. See [support-manage.md](support-manage.md).
- **`form-add`** — `DocumentJournal` support.
- **`form-validate`** — dangling-binding checks; paired with the `cfe-borrow` re-borrow idempotency fix.
- **`form-info`** — prints the object's support state.
- **`form-remove`** — clears **every** `Default*Form` slot pointing at the removed form (previously only the generic `DefaultForm`, which left a dangling reference to a deleted form). Local `-DryRun` / `-Force` safety gate is preserved on top.

### Earlier Additions (upstream `w-2026-05-17`)

In addition to the form-compile / form-info / form-add / form-edit / form-remove changes already documented in sections 2–5, **`form-validate`** got the following improvements (script `tools/1c-form-validate/scripts/form-validate.ps1`):

- Stops false-flagging real ERP and БП forms — `Items.<Table>.CurrentData.<Field>` and `~<DynamicList>.<Field>` paths are now correctly resolved through the table's data attribute. Missing table → error; third segment ≠ `CurrentData` → warning.
- Opaque platform paths (`"10"`, `"1000003"`, `"N/M: "`) are skipped without an error. Previously Check 5 reported "attribute not found" on these.
- New attribute-type check in `data`: error on intentionally invalid types, warning on unrecognised ones. Context is honoured — `ExternalDataProcessorObject` / `ExternalReportObject` are valid only inside an external data processor / report; in regular configuration object forms it is an error with a hint to use the inner object type.
- Platform 8.5 support — new compatibility / interface mode values and the new XML header format.
- Brief output by default; full per-check trace via `-Detailed`. The `-Path` parameter accepts both a `Form.xml` file and a `Forms/<Name>` folder (auto-resolves to `Forms/<Name>/Ext/Form.xml`).

### What's New in `form-compile` (vs the previous local snapshot)

- **`-FromObject` mode** — produces a typical form from object metadata; purpose (`Object`/`List`/`Choice`/`Folder`/`Record`) is inferred from `OutputPath`. Document list forms get the standard `Number` and `Date` columns automatically; ChartOfAccounts pulls accounting flags / sub-account kinds correctly.
- **New element types** — `radio` (RadioButtonField with `radioButtonType`: `Auto` / `RadioButtons` / `Tumbler`, `choiceList`); `autoCmdBar` (fills the form's main AutoCommandBar id=-1); `columnGroup` (column grouping inside table `columns` — `horizontal` / `vertical` / `inCell`, nestable).
- **New input keys** — `textEdit: false` (disable free text editing on reference fields), `maxWidth` / `maxHeight` (hard caps, usually with `autoMaxWidth: false`).
- **New group key** — `collapsed: true` for `"group": "collapsible"` (group starts collapsed).
- **Multilingual strings** — any title / presentation may be `{ "ru": "...", "en": "..." }`.
- **Auto-titles** — attributes, commands, pages, popups and decorations without explicit `title` get a humanised title from the name (`НомерСчёта` → "Номер счёта").
- **Format version** — auto-detected from the nearest `Configuration.xml` (8.3.27+, 8.5).
- **Presets** — `tools/1c-form-compile/presets/erp-standard.json` is shipped; project-level override at `<projectRoot>/presets/skills/form/<name>.json`.
- **Defaults aligned with real ERP/БП forms** — multi-line inputs are not auto-width-bounded by default; checkbox title is on the right; `autoTitle` is suppressed when `title` is set; objects with editable state save the input state (`Esc → confirm`).

### Upstream sync `2026-10-28` — `form-compile`

`form-compile` v1.176 → **v1.217**. This one needed its own pass: upstream replaced the fork's hand-written event tables with a **generated event catalog** (`Get-FormEventCatalog`, built from 28.5k forms of the typical configurations and checked by loading into 8.3.24 / 8.3.27 / 8.5), added `callType` (`Before` / `After` / `Override`) for extension handlers, form-module handler stubs, and reworked the from-object presets. Fifteen conflicts, eleven of them semantic.

Deltas kept (each marked `Local:` in the file):

- the support-guard target is normalised for `-FromObject` (the real edit target is `Ext/Form.xml` inside the form directory) and resolved with `ProviderPath`;
- an event name that is missing from the catalog is **refused** (`[ERROR] Unknown event … Known: …`, exit 1) instead of upstream's warning — a typo used to land in `Form.xml` and only surfaced when the Configurator opened the form;
- the closed value sets of `windowOpeningMode` / `autoTime` / `usePostingMode` stay (upstream accepts any string and its spec listed values that do not exist — `Modeless`, `Current`, `Postings`, `Movements` — and the Configurator rejects such a `Form.xml` on load);
- the canonical root-tag order (`$script:formRootTagOrder`) and the value table are both present.

Verification: upstream harness **64/68** in isolation. The four remaining cases encode upstream's variants of our deltas (unknown event as a warning, enum value forgiveness) plus one preset property order and one catalog-warning case; our own `python-ports-regression` pins the parity that survived and records `form-compile.py` as lagging (see `docs/runtime-selection.md`).

### Upstream sync `2026-10-28` — the scaffolds and `form-compile`'s neighbours

Source `662e752e`. `form-add` v1.12 → **v1.34**, `add-template` v1.10 → **v1.30**, `add-help` v1.9 → **v1.26**.

Deltas kept in `form-add`: the `-FormName` identifier gate (Phase 0 — the name becomes a file name, a directory name and an XPath literal), the `.dev.env SUPPORT_GUARD` gate, and **XML escaping of `-FormName` / `-Synonym`** in the generated descriptor. The escaping is not cosmetic: upstream interpolates the values verbatim, so an ordinary synonym `A & B` produced a descriptor no parser accepts — our own regression case caught it after the port.

Delta **dropped** in `form-add`: the fork's "an information register's object form names its main attribute `Запись`". Upstream now carries a per-kind purpose table where the register purposes are `Custom` / `List` / `Record` / `RecordSet` — the record form is `Record` with `AttrName = "Запись"`, and `-Purpose Object` is refused for a register naming the allowed values. The fork's rule became wrong, not redundant.

`add-template` / `add-help`: upstream absorbed the fork's whole save pipeline (MemoryStream post-processing, BOM strip, uppercase `UTF-8`, compact tags, target-EOL detection); only the processing-instruction branch had to be re-added.

**`form-remove` (v1.4) is deliberately not re-synced.** The fork rewrote the tool — identifier validation, path containment, `-DryRun` / `-Force`, and a bounded transaction whose deletions are reversible renames into a quarantine — and that rewrite is pinned by `tools/tests/python-ports-regression.py` and by the metadata regression. Upstream's v1.5–v1.11 fixes (reference check before deletion, unified XML finalisation) are present in the rewrite or superseded by it.

Verification: `form-add` **30/30** in the upstream harness, the compact-tag suite **11/11** (covers `form-add`, `remove-form`, `form-compile` registration, `add-template`, `add-help`, `skd-edit`), metadata regression **62/62**.

### Cross-cutting fix `2026-10-28` — a JSON definition under PowerShell 7

Six scripts read their JSON definition through `Write-Output -NoEnumerate $parsed`. Under PowerShell 7 that wraps a **non-array** in a `List<object>`, so the caller's `$def.PSObject.Properties` enumerated `List` members (`Capacity`, `Count`, …) instead of the JSON keys and every operation was reported as `Unknown operation: Capacity`. Windows PowerShell 5.1 returns the object unchanged, which is why the idiom survived. Fixed in `meta-edit`, `meta-compile`, `form-edit`, `role-edit`, `skd-compile`, `skd-decompile`: only arrays take the `-NoEnumerate` path. The harness runs `pwsh` first, so it is what surfaced here; `docs/runtime-selection.md` now states that PowerShell 7 is supported.

## help-manage.md

### Upstream sync `2026-07-30`

Script refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `help-add` v1.4 → **v1.9**. It now enforces the vendor support gate — adding help to a locked object of a typical configuration is refused with a pointer to `support-edit` ([support-manage.md](support-manage.md)).

## interface-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `interface-edit` v1.8 → **v1.24**, `interface-validate` v1.1 → **v1.6**.

Deltas kept: the `.dev.env SUPPORT_GUARD` gate, the sibling validator in `$PSScriptRoot` with its exit code propagated, processing instructions kept literal. Upstream's target-EOL detection — an edit inherits the destination's line endings — replaces the fork's own: same rule, one implementation.


### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `interface-edit` v1.3 → **v1.8**, `interface-validate` → **v1.1**. `interface-edit` now enforces the vendor support gate — it refuses to edit the command interface of a locked typical configuration ([support-manage.md](support-manage.md)).

### Earlier Additions (upstream `w-2026-05-17`)

The PowerShell scripts under `tools/1c-interface-manage/scripts/` were refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights:

- **`interface-edit`** — operations `place` / `order` accept the value as an object (not only as a string). Command names in `hide` / `show` / `place` / `order` are normalised: `Catalogs.X` and `Справочник.X` map to canonical `Catalog.X`.
- **`interface-validate`** — universal validator improvements (one-liner output by default, `-Detailed`, folder path auto-resolution) — see `role-manage.md` → "Recent Additions".


## meta-manage.md

### Upstream sync `2026-10-28` — `meta-remove`

Source `662e752e`. `meta-remove` v1.5 → **v1.15**, keeping the `.dev.env SUPPORT_GUARD` gate.


### Upstream sync `2026-10-28` — the `meta-*` family

Source `662e752e`. Versions: `meta-compile` v1.69 → **v1.124**, `meta-edit` v1.24 → **v1.70**, `meta-validate` v1.13 → **v1.42**, `meta-info` v1.4 → **v1.16** (no local deltas — plain refresh).

Deltas kept, re-applied on the new bodies:

- `.dev.env SUPPORT_GUARD` before `.v8-project.json` `editingAllowedCheck` in `meta-compile` and `meta-edit`.
- `.dev.env NEW_OBJECT_POSITION` before the registry's `newObjectPosition` in `meta-compile`, with the fork's semantics restored: a value that is **set** decides on its own, and an unrecognized value resolves to the documented `end` rather than silently handing the ordering to the fallback file.
- `meta-edit` auto-validation: the sibling validator is resolved under the fork's directory names (`1c-meta-validate`, `1c-epf-validate` for external objects), a **missing** validator is a refusal raised before the write, and the validator's exit code propagates as the run's verdict (with an explicit stderr line naming the code).
- `meta-edit` compact empty tags: processing instructions are kept literal (`<?…?>`), as in `form-edit` — upstream protects CDATA and comments only.
- `meta-validate`: **6a** (an inline `<Form><Properties>` descriptor where a scalar reference belongs), **6c** (descriptor does not parse), **6d** (descriptor name differs from its registration), **16a** (export folder name where a type belongs).

Deltas **dropped** because the ported fixtures showed them to be wrong, not merely redundant. Each was written against v1.12 and fired on dumps the platform accepts:

- **1e / 6e** (format version must equal `Configuration.xml`) — a 2.18 object inside a 2.20 dump is a routine merge-branch artefact and the platform loads it; upstream's check 24 reports it as the warning it is. Ours turned it into a hard error.
- **6f** (`Default*Form` must name a declared form) — upstream's check 20 already resolves those references and distinguishes "in the composition, not in this dump" (warning) from "absent altogether" (error); it also accepts a UUID reference, which ours rejected.
- **6b as an error** — kept, but **downgraded to a warning**: a registered form whose descriptor is missing is the normal shape of a partial dump (`/getconfigfiles` exports a selection, the platform merges it), so a hard error blocked the workflow the ruleset itself prescribes.
- **2's "name exactly `<Type><Category>.<Name>`"** — external data source tables carry the source name (`ExternalDataSourceTableManager.PG.prices`), so the exact-name rule rejected valid objects.
- **8 / 12 / 6g / 6h** — superseded: upstream's case-insensitive shared namespace for attributes, tabular sections, dimensions and resources, its per-kind enum tables and its `Description` / `Code` handling cover the same ground with the platform's own rules.

Verification against upstream's own snapshot harness (hybrid skills tree, PowerShell runtime): `meta-compile` **122/123**, `meta-edit` **59/62**, `meta-validate` **77/80** (3 skipped) plus five authored cases for the kept deltas (**5/5**), `meta-info` **29/39** (10 skipped by fixture prerequisites). The failures are accounted for: `meta-compile` / `meta-edit` each have one **LF-preservation** case that cannot pass in a checkout with `core.autocrlf=true` (the LF fixture arrives as CRLF; the fork's own LF handling was verified directly — an LF object keeps LF, 0 CRLF, BOM intact, with the edit applied), and `meta-edit` has two snapshot diffs where the fork deliberately writes `<Name>` and `<Synonym>` on separate lines (the whitespace fix of 2026-09); those snapshots are upstream's, not ours.

### Local fix `2026-09-25` — versions, generated types, default forms, name clashes

`meta-validate` v1.13, both runtimes with identical messages: object, form descriptor and `Form.xml` versions equal `Configuration.xml` (1e, 6e); every GeneratedType category is required and named exactly (2); `Default*Form` / `Auxiliary*Form` resolve to a declared form of the right role (6f); no `Description` / `Code` binding when the length is 0 (6g) and a warning when the default object form hides a mandatory `Description` nobody assigns (6h); case-insensitive name uniqueness shared by attributes, tabular sections, dimensions and resources (8); `Periodicity` and `DefaultRecordSetForm` forbidden in an information register (12); `cfg:Catalogs.X` is an error (16a). Over all 10 965 ERP objects none of the new error checks fires; 6h warns on 6 attached-file / exchange catalogs.

### Local fix `2026-09-13` — refuse false-success template additions

Both `meta-edit` runtimes refuse `add-template` before any write, including aliases and mixed JSON. The error points to the existing PowerShell template scaffold and requires an explicit template type. No partial registration is presented as a completed template.

### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `meta-compile` v1.12 → **v1.68**, `meta-edit` v1.6 → **v1.23**, `meta-validate` v1.3 → **v1.12**, `meta-info` v1.2 → **v1.4**, `meta-remove` v1.1 → **v1.5**.

- **Support gate** — `meta-edit` / `meta-compile` refuse to touch an object of a typical configuration that is locked ("на замке"); `meta-remove` refuses to delete an object still on support. See [support-manage.md](support-manage.md).
- **`meta-compile`** — many more object types authorable from the DSL (`CommonPicture`, `CommonTemplate`, `SessionParameter`, `CommonCommand`, `CommandGroup`, `CommonAttribute`, `FunctionalOptionsParameter`, `WSReference`), format 2.20 properties (platform 8.3.27), command-group validation, better auto-synonym derivation. Full grammar: [`meta-dsl-spec.md`](../tools/1c-meta-compile/meta-dsl-spec.md).
- **`meta-edit`** — structural attribute properties are now editable point-wise: `Format` / `EditFormat` / `ToolTip` / `ChoiceForm`, `MinValue` / `MaxValue`, `LinkByType` / `ChoiceParameterLinks`, `ChoiceParameters`, `FillValue`; list properties `DataLockFields` / `RegisteredDocuments`; `add-predefined` for predefined items; create-if-missing for properties with a type guard.
- **`meta-validate`** — checks that referenced types exist, validates object commands (group + parameter rule), version-dependent properties and the `LineNumberLength` range, `MDObjectRef` reference shape, and rejects reserved attribute names in a type-aware way.
- **`meta-info`** — prints the object's support state and the type presentation for reference objects.

### Earlier Additions (upstream `w-2026-05-17`)

The PowerShell scripts under `tools/1c-meta-{compile,edit,info,remove,validate}/scripts/` were refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights:

#### `meta-compile` — new properties and stricter type rules

- **Catalog properties** are now driven by JSON (no more hard-coded values): `limitLevelCount`, `levelCount`, `foldersOnTop`, `subordinationUse`, `codeSeries`, `quickChoice`, `choiceMode`. Hand-edit of XML is no longer required for non-default settings.
- **`owners`** — array of catalog owners with shorthand syntax.
- **`multiLine: true`** (or flag `| multiline`) on an attribute marks it as multiline.
- **`choiceHistoryOnInput`** on attributes — controls history-based auto-completion when entering a reference value.
- **Default for `quickChoice`** aligned with real configurations: catalogs / chart-of-characteristic-types / chart-of-accounts / chart-of-calculation-types / exchange plans default to `false`; enums default to `true` (≈95% / ≈99% match across real configs).
- **Manager modules** are now created alongside the object module for **reports and data processors** — required for reports that override `НастроитьВариантыОтчета`. Constants get manager and value-manager modules; enums get a manager module.
- **Empty `Ext/` folders** no longer created for constants, enums and document journals — they previously caused the platform to wipe extension modules on load.
- **Register-attribute properties** are filtered by register kind: AccumulationRegister / AccountingRegister / CalculationRegister attributes no longer get attribute-only properties the platform silently dropped. InformationRegister keeps the full set.
- **System enum values** in properties (`RegisterType`, `WriteMode`, `Periodicity`, …≈20 more) now accept synonyms and are case-insensitive — typical model errors like `Balances` → `Balance` or Russian variants no longer break the build.
- **Strict validation of enum values**: an unknown value for a known property gives a clear error instead of leaking into XML.
- **Format version** auto-detected from the nearest `Configuration.xml` (8.3.27+, 8.5).

#### `meta-edit`

- Same synonym dictionary and case-insensitivity for system enum values, applied in `modify-attribute` / `modify-property` and when parsing `fillChecking` / `indexing`.

#### `meta-validate`

- Empty register check (no dimensions, no resources, no attributes — platform refuses to load).
- Document-movements pointing to a non-existent register are reported.

#### `meta-remove`

- Returns exit code 1 when the object is not found (was silently 0).

## mxl-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `mxl-compile` v1.4 → **v1.59**, `mxl-decompile` v1.0 → **v1.32**, `mxl-info` v1.2 → **v1.7**, `mxl-validate` v1.1 → **v1.10**.

`mxl-compile` keeps the `.dev.env SUPPORT_GUARD` gate; the other three had no local delta and were replaced outright. The refresh brings `mxl-decompile`'s newer lossless model (cell formats, merged and named areas) that `mxl-compile` reads back.


### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `mxl-compile` v1.1 → **v1.4**, `mxl-info` v1.0 → **v1.2**; `mxl-decompile` / `mxl-validate` unchanged. `mxl-compile` enforces the vendor support gate ([support-manage.md](support-manage.md)); `mxl-info` prints the support state of the layout's owner.

### Earlier Additions (upstream `w-2026-05-17`)

The PowerShell script `tools/1c-mxl-validate/scripts/mxl-validate.ps1` was refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights:

- **Universal validator improvements** — one-liner output by default (`-Detailed` for the full per-check trace); accepts both `Template.xml` and a folder path; universal `-Path` parameter alongside legacy `-TemplatePath`.

The compile/decompile/info scripts (`mxl-compile`, `mxl-decompile`, `mxl-info`) were not refreshed — no significant upstream changes for them in this period.

## role-manage.md

### Upstream sync `2026-10-28` — the rest of the family

Source `662e752e`. `role-compile` v1.8 → **v1.47**, `role-info` v1.2 → **v1.7**, `role-validate` v1.1 → **v1.9**, plus the newly vendored `role-edit` described above.

Deltas kept: `.dev.env SUPPORT_GUARD` in `role-compile`; in `role-edit` the validator resolved from `1c-role-validate` with its exit code propagated.


### New tool `2026-10-28` — `role-edit` v1.9 vendored

`role-edit` is new to the fork: our `role-compile` only ever creates a role with a new UUID, so changing an existing role was the documented exception "edit `Ext/Rights.xml` by hand against the templates". The upstream tool does that in place — rights, RLS, templates, role flags, synonym and comment — keeping the UUID and every entry the operation does not name.

Local deltas, on top of the upstream script:

- `.dev.env SUPPORT_GUARD` before `.v8-project.json` `editingAllowedCheck`, the same block as the other mutators (a role on vendor support is refused).
- The auto-validation call resolves the fork's sibling directory (`../../1c-role-validate`) and its **exit code propagates** — a role that was written but fails `role-validate` exits non-zero instead of silently reporting success.

Verification: upstream's 28 `role-edit` cases pass against the vendored script (hybrid skills tree, PowerShell runtime).

### Local fix `2026-10-14` — `role-validate` parameters, rights-only edits

The validate section described `-MetadataPath`, which `role-validate.ps1` does not have (the call failed with a PowerShell parameter error; support ticket d3372592): the metadata file is located from the `Rights.xml` path. The section now lists the real parameters (`-RightsPath` / `-Path`, `-OutFile`, `-Detailed`, `-MaxErrors`). The manual fallback for rights-only edits of an existing role contradicted `SKILL.md → Hard rule`; it is now one of its documented exceptions, because no tool edits a role in place.

### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `role-compile` v1.5 → **v1.8**, `role-info` v1.0 → **v1.2**, `role-validate` → **v1.1**. `role-compile` enforces the vendor support gate ([support-manage.md](support-manage.md)); `role-info` prints the support state.

### Earlier Additions (upstream `w-2026-05-17`)

The PowerShell scripts under `tools/1c-role-{compile,info,validate}/scripts/` were refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights:

- **`role-compile`** — `OutputDir` now accepts the source root of the configuration; `Roles/` is created automatically. Legacy behaviour (path already ending in `Roles`) is preserved. The DSL key `rights` is accepted as a synonym of `objects`.
- **`role-validate`** — auto-discovers metadata from the path to `Rights.xml`; `-MetadataPath` is no longer required, metadata-driven checks always run when the file is present.
- **All 10 validators** (`role-validate`, `meta-validate`, `epf-validate`, `skd-validate`, `cf-validate`, `cfe-validate`, `form-validate`, `mxl-validate`, `subsystem-validate`, `interface-validate`) now emit a single one-liner by default; the full per-check trace is available via `-Detailed`. Each accepts a folder path as the primary file argument and resolves to the canonical XML file (e.g. `Roles/MyRole` → `Roles/MyRole/Ext/Rights.xml`). The universal `-Path` parameter is supported in addition to the legacy named parameters (`-RolePath`, `-FormPath`, `-TemplatePath`, `-ObjectPath`, …).

## skd-manage.md

### Upstream sync `2026-10-28` — the `skd-*` family

Source `662e752e`. `skd-compile` v1.109 → **v1.139**, `skd-decompile` v0.91 → **v0.106**, `skd-edit` v1.30 → **v1.50**, `skd-info` v1.8 → **v1.14**, `skd-validate` v1.2 → **v1.8** (`skd-info` / `skd-validate` / `skd-decompile` had no local delta — plain refresh).

Deltas kept: `.dev.env SUPPORT_GUARD` before `.v8-project.json` in `skd-compile` / `skd-edit`; processing instructions kept literal in the compact-tag writer of `skd-edit` (upstream protects CDATA and comments only).

What the refresh buys: a lossless `decompile → compile` round-trip for filter and parameter **values** (references, empty values, calculated fields, `availableValues`, parameter data types and their short form), query-dataset flags and source binding, and `ProviderPath` for network paths.

Verification (upstream snapshot harness against the ported files): `skd-compile` **54/54**, `skd-decompile` **19/19**, `skd-edit` **59/59**, `skd-info` **8/8** (3 skipped by fixture prerequisites), `skd-validate` **19/19** (7 skipped).


### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `skd-compile` v1.104 → **v1.109**, `skd-edit` v1.24 → **v1.30**, `skd-info` v1.5 → **v1.8**, `skd-decompile` v0.90 → **v0.91**, `skd-validate` → **v1.2**.

- **Support gate** — `skd-compile` / `skd-edit` refuse to modify a schema of a locked typical configuration ([support-manage.md](support-manage.md)).
- **`skd-edit`** — parameter value lists in shorthand (paired with `skd-compile`).
- **`skd-info`** — `-Raw` flag for lossless round-trip extraction of the query text; prints the support state.
- Full DSL grammar vendored as [`skd-dsl-spec.md`](../tools/1c-skd-compile/skd-dsl-spec.md).

### Earlier Additions (upstream sync `w-2026-05-31`)

The PowerShell scripts under `tools/1c-skd-{compile,edit,info,validate}/scripts/` were refreshed again from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills) (`skd-compile` → v1.104, `skd-edit` → v1.24, `skd-info` → v1.5, `skd-validate` → v1.2), and the new `1c-skd-decompile` tool (draft, v0.90 — see section 5) was added. Highlights from the prior `w-2026-05-17` batch still apply:

#### `skd-edit` — new operations and flags

- `set-field-role` — replace a field role in one line with flag tokens (`@balance`, `@dimension`, `@account`, `@period`, `@required`, `@autoOrder`, `@ignoreNullValues`) and kv params (`balanceGroupName`, `balanceType`, `accountTypeExpression`, …). Closes the "temporarily disable role to debug" pattern.
- `modify-structure` — change groupings by name while preserving selection, order, filters, and conditional appearance.
- `clear-conditionalAppearance` — drop conditional appearance for a field.
- `add-drilldown` — wire detail processing to DCS resources across all named templates.
- `rename-parameter` — atomically rename a parameter and update `&Name` references in other parameters and in all variants.
- `reorder-parameters` — partial list (named ones go first in the given order, the rest keep relative order).
- `modify-parameter` — `use`, `denyIncompleteValues`, `availableValues` (single shot replaces the full list).
- `patch-query` `@once` flag — fail when the substring is missing or appears more than once. Multiline replacements are now correct; empty replacement (`old =>`) deletes the substring.
- `add-total` shortcut: `Func`, `Func(expr)`, or just a field name (non-aggregate functions become an identity expression).
- `set-structure`: comma in shorthand for several fields on one level; quotes in `@name=` are stripped on write; reference parameter types serialise with the right `xsi:type`; batch with `;;` is trimmed.
- `add-selection` accepts `@group=Name` and nested `Folder(...)`.
- `modify-filter` / `modify-dataParameter` preserve `Use=false` when the `@off` / `@on` flag is omitted (no longer silently flipped).
- `conditionalAppearance` — `OrGroup`, `DesignTimeValue`, `Format`. `#noFilter` / `#noOrder` / `#noGroup` flags now also work in `add-calculated-field`.

#### `skd-compile` — DSL upgrades

- `@autoDates` emits the canonical БСП pattern (`НачалоПериода` / `КонецПериода`, two date pickers, `useRestriction`). Shortcut requires the period filled (`use=Always` + `denyIncompleteValues=true`).
- Composite reference types: `"type": ["CatalogRef.A", "CatalogRef.B"]`.
- Multilingual titles / presentations: `{ "ru": "...", "en": "..." }`.
- Horizontal cell merge in templates via `>` (mirror of `|` for vertical) — two-level headers without hand-editing XML.
- Parameters: shorthand `[Title]`, array `availableValues`, `denyIncompleteValues`, flag `@hidden` (auto-`useRestriction`), `@valueList` / `valueListAllowed`, `dataParameters: "auto"` (preserves defaults across all parameter kinds).
- Templates: `drilldown` via `DetailsAreaTemplateParameter`, `groupHeaderTemplate`, `groupName`.
- Structure: `"GroupName[Field] > details"` for named groupings; object form `{ "items": [...] }`; `useRestriction` as an object `{ field: true }`.
- Reference values (chart of accounts, catalog, enum, document) in filters and conditional appearance auto-receive predefined presentations. `Format` parameter as multilingual string. `OrGroup` in conditional appearance via `or`.
- `Folder(Title: f1, f2)` in selection — for `SelectedItemFolder` with nested fields.
- Standard period parameter always serialised in canonical form — no diff after first re-save in Configurator.
- External SQL files via `"@path/to/file.sql"`.
- Compact area templates DSL: `rows`, `widths`, `style` with built-in presets (`header` / `data` / `subheader` / `total`), vertical merging, parameters in cells.
- Project-level user style preset at `<projectRoot>/presets/skills/skd/skd-styles.json` (auto-discovered by `skd-compile`).
- `dataSetLinks` accepts both DSL keys (`sourceExpr`, `destExpr`, `source`, `dest`) and XML keys (`sourceExpression`, `destinationExpression`, ...).
- `--from-object`-like discovery: pass a path to a report or processor folder to `skd-info`, the script finds the embedded DCS template by itself; if multiple — it lists them.
- 8.5 platform support across `cf-validate`, `cfe-validate`, `epf-validate`, `skd-validate`.

#### `skd-info`

- Field detail view (`-Mode fields -Name <Field>`) prints kv role parameters (`balanceGroupName`, `balanceType`, `accountTypeExpression`, …) on the Role line.
- Section `query` of `-Mode full` prints external dataset names when no queries exist (no more anonymous "(no query datasets)").

## subsystem-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `subsystem-compile` v1.9 → **v1.35**, `subsystem-edit` v1.7 → **v1.27**, `subsystem-info` v1.2 → **v1.7**, `subsystem-validate` v1.2 → **v1.7**.

Deltas kept: `.dev.env SUPPORT_GUARD`; the sibling validator in `$PSScriptRoot`; processing instructions kept literal. Upstream's anchor-based `ChildObjects` insert — it patches the parent text instead of re-serialising the document, which is what keeps the parent's formatting intact — replaces the fork's `XmlWriter` save.


### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `subsystem-compile` v1.5 → **v1.9**, `subsystem-edit` v1.2 → **v1.7**, `subsystem-info` v1.0 → **v1.2**, `subsystem-validate` → **v1.2**.

- **Support gate** — `subsystem-compile` / `subsystem-edit` refuse to modify subsystems of a locked typical configuration; see [support-manage.md](support-manage.md).
- **`subsystem-info`** — prints the support state of the subsystem.

### Earlier Additions (upstream `w-2026-05-17`)

The PowerShell scripts under `tools/1c-subsystem-manage/scripts/` were refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills). Highlights:

- **`subsystem-compile` / `subsystem-edit`** — content of a subsystem accepts Russian and plural prefixes (`Справочник`, `Справочники`, `Catalogs`) and normalises them to the canonical `Catalog`. `subsystem-validate` flags surviving plural forms as an error.
- **Stub-files for child subsystems** are created automatically when the parent declares a child. Previously the `<Subsystems>` reference existed but the file did not — the platform silently ignored it, and stricter loaders started failing.
- Subsystem `objects` accepts `content` as a synonym (and vice versa).
- Validators got the universal improvements described in `role-manage.md` → "Recent Additions" (one-liner output by default, `-Detailed`, folder path auto-resolution).

## template-manage.md

### Upstream sync `2026-10-28` — `template-remove`

Source `662e752e`. `template-remove` v1.3 → **v1.9** (`add-template` is described above). Upstream absorbed the fork's save pipeline here too; the processing-instruction branch had to be re-added.


### Local fix `2026-09-13` — object XML paths

`add-template.ps1 -ObjectName` accepts absolute and relative object XML paths and places the template beside the resolved object. Name-only configuration lookup and the EPF `ProcessorName` alias remain supported.

### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `template-add` v1.5 → **v1.10**, `template-remove` → **v1.3**. `template-add` enforces the vendor support gate — it refuses to add a template to a locked object of a typical configuration ([support-manage.md](support-manage.md)). `template-remove` keeps the local hardening (preflight parse, atomic root-XML write, `-DryRun` / `-Force`).

## web-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `web-publish` v1.4 → **v1.16**, `web-info` v1.0 → **v1.7**, `web-stop` v1.0 → **v1.4**, `web-unpublish` v1.0 → **v1.4**.

Deltas kept: the `.dev.env` platform chain in `web-publish` (install-directory `PLATFORM_PATH`, `bin\` resolution for `wsap24.dll`); the `-DryRun` / `-Force` gate in `web-unpublish`. Upstream's newer `default.vrd` defaults (`publishExtensionsByDefault`, `httpServices`) are in.


### Local fix `2026-10-01` — extension HTTP services are published

`web-publish` (both runtimes) writes `publishExtensionsByDefault="true"` on `httpServices` in `default.vrd`, as the platform's own publication does; without it the HTTP services of the infobase's extensions were not published.

### Upstream sync `2026-07-30`

Scripts refreshed from [Nikolay-Shirokov/cc-1c-skills](https://github.com/Nikolay-Shirokov/cc-1c-skills): `web-publish` v1.2 → **v1.4**; `web-info` / `web-stop` unchanged; `web-unpublish` keeps the local `-DryRun` / `-Force` safety gate.

- **OData is enabled correctly in `default.vrd`** — via the `<standardOdata enable="true"/>` child element instead of the old `enableStandardOdata` attribute, which the platform ignored. Publications that need the standard OData interface actually get it now.
- Platform path resolution follows the `db-*` chain (explicit → `.dev.env` → `.v8-project.json` → auto-detect), locally patched to also accept the version install directory shape used by `.dev.env` `PLATFORM_PATH` (resolves `bin\` for `wsap24.dll`).

## support-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `support-edit` v1.0 → **v1.3** — the toggle itself is unchanged; the refresh brings the shared support-state specification and the `Ext/ParentConfigurations.bin` parsing that the other tools' guard rails now use.

## xdto-manage.md

### Upstream sync `2026-10-28`

Source `662e752e`. `xdto-compile` v1.1 → **v1.18**, `xdto-decompile` v1.0 → **v1.3**, `xdto-edit` v1.0 → **v1.9**, `xdto-info` v1.0 → **v1.3**, `xdto-validate` v1.1 → **v1.6**.

Deltas kept: the `.dev.env SUPPORT_GUARD` gate in `xdto-compile` / `xdto-edit`; the sibling `xdto-validate` / `xdto-decompile` dispatch from `$PSScriptRoot` (upstream's `..\..\xdto-<name>\scripts` path does not exist in this layout). Upstream's `Assert-EditAllowed` — resolve the element uuid, walk up to the configuration root, treat an autonomous external object as always editable — replaces the fork's simpler walk.
