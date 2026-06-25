# Changelog

All notable changes to ADAM are documented here.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).  
Versions follow [Semantic Versioning](https://semver.org/).

---

## [0.8.2] — 2026-06

### Added
- `FindFileByWindowTitle` wired into the `.pbix` file detection chain — resolves
  the file path from the PBI Desktop window title when the file was opened via
  Open Recent (no command-line path available).
- `FindPbixByFileHandle` — enumerates process file handles as an additional
  detection strategy (useful when PBI keeps the file locked).
- `db.Name` used as a stable settings key for `.pbix` file path caching, so the
  path survives PBI restarts (the session GUID changes each time, but the model
  name is stable).
- Store App window title support — PBI Desktop Store App shows just the model
  name without the " - Power BI Desktop" suffix; both formats are now handled.
- `--overwrite` flag for CLI mode.
- `--mode per-fact` flag for CLI mode.
- `--set-classification` flag for CLI mode.
- Unsupported format warning before writing to `.pbix` files.
- Timestamped backup created automatically before every `.pbix` write.
- Test suite: `TableClassifierTests`, `DiagramBuilderTests`, `CliIntegrationTests`,
  `SettingsManagerTests`, extended `LayoutEngineTests`.
- GitHub Actions: PR test gate (`tests.yml`), CodeQL security scan (`codeql.yml`).
- `Directory.Build.props` for shared project properties.
- MinVer for automatic version resolution from git tags.
- Coverlet for code coverage collection.
- `.editorconfig` enforcing project code style.
- `dependabot.yml` for automated dependency updates.
- PR template and issue templates.
- `CLAUDE.md` project context for AI-assisted development.
- `docs/test-plan.md` manual test plan.

### Fixed
- When a `.pbip` file was open and a `.pbix` file with the same model name existed
  anywhere on your machine, ADAM would sometimes detect the `.pbix` instead of the
  open `.pbip`. Several issues contributed: (1) a format filter on the command-line
  scan blocked the `.pbip` path because Power BI Desktop uses a session identifier
  for both formats — the filter has been removed since the cmdline path is definitive;
  (2) a file-handle scan was producing false-positive `.pbix` results because Power
  BI keeps internal AS workspace files open that happen to share the model name,
  regardless of whether the user's file is `.pbip` or `.pbix` — the scan has been
  removed and the more targeted per-file lock check (already used inside the
  window-title search) is used instead; (3) for the Standalone installer, ADAM now
  uses the fact that it keeps `.pbix` files locked while open — if no file-lock is
  confirmed, a `.pbip` found on disk is preferred; (4) the `.pbix` and `.pbip`
  filesystem scans now run in separate error handlers so a scan error on one can
  never suppress the other.
- When two Power BI files were open at the same time — one opened by double-clicking
  and one via Open Recent — ADAM launched from the Open Recent window would sometimes
  show the double-click file's path instead. This happened because ADAM assumed the
  only process it could see with a file path in its command line must be the one that
  launched it, even when a second Power BI window was running without a command-line
  path. ADAM now requires a name match before relying on that shortcut.
- When a `.pbix` file was opened via Open Recent, ADAM would sometimes detect the
  `.pbip` file of the same name in the same folder instead. This happened because
  ADAM searched for `.pbip` files before `.pbix` files, and picked up the `.pbip`
  as the first match before it could confirm which file Power BI actually had open.
  ADAM now tries to confirm each `.pbix` candidate via the OS file-lock first, and
  uses the database name format (Power BI always uses a session ID for `.pbix` files)
  to resolve the tie if confirmation is not possible — so the right file is found
  even when both formats share the same name in the same folder.
- OutriggerBridge tables (such as a Time Period table linked to a Calendar
  dimension) were appearing on top of their parent Dimension in the Star Schema
  layout even after the previous fix, because Power BI clips tables with
  negative canvas coordinates to the canvas edge. ADAM now shifts the entire
  diagram right/down after placement so all tables have positive coordinates.
- ADAM was forgetting the last-used layout style (Waterfall, Star Schema, or
  Inverted L) on every restart, always reverting to Waterfall. The radio button
  events fired during startup and overwrote the saved value before the restore
  code ran. ADAM now suppresses those startup events so the saved style is
  correctly restored.
- When you saved an unsaved file in Power BI Desktop and then clicked Apply,
  ADAM couldn't find the newly saved file because it only re-ran discovery when
  it had never found a file at all, not when the stored path was empty after a
  re-connect. Discovery now re-runs whenever the file path is missing.
- When searching for your Power BI file by window title, ADAM checked the
  Downloads folder first. If you had a stray downloaded copy of the same file
  there, ADAM would open the wrong file. ADAM now checks your development
  folders (Repos, Git, Projects, etc.) first and falls back to Downloads last.
  Project files (.pbip) are also tried before report archives (.pbix).
- A settings-cache shortcut in the file-detection chain could return a stale
  path from a previous session (for example, a .pbix path when you had since
  switched to .pbip), causing ADAM to write the layout to the wrong file and
  skip writing TMDL annotations. The cache is now only consulted after the
  live scan finds nothing.
- Manual table classification changes were not being remembered across Power BI
  restarts for .pbix files. The classifications were saved under the AS session
  GUID, which changes every time Power BI restarts, so they could never be
  found again. ADAM now saves them under the stable file name instead.
- ADAM now remembers which layout style you last used (Waterfall, Star Schema,
  or Inverted L) and restores it when you reopen the app.
- Manual table classification changes are now saved and remembered when working
  with `.pbix` files, so you no longer have to re-classify tables every time
  you relaunch ADAM.
- OutriggerBridge tables (for example, a Time Period table linked to a Calendar
  dimension) were appearing on top of their parent Dimension in the Star Schema
  layout. They are now placed further out so they no longer overlap.
- When ADAM can't detect your file path — usually because the report hasn't
  been saved yet — it now shows a clear message explaining that you should save
  the file in Power BI Desktop first, rather than opening a file browser with
  no explanation.
- When you cancelled Power BI's "save before closing" prompt during a `.pbix`
  layout apply, ADAM would stay hidden and appear frozen for up to 60 seconds.
  ADAM now reappears within about 10 seconds and shows a clear message
  explaining what happened.
- Improved how ADAM picks the right file when you have multiple Power BI files
  with the same name in different folders. ADAM now checks in a smarter order
  — including files that are currently open in Power BI — before resorting to a
  general folder scan that could pick the wrong one.
- `TableClassifier.Classify` was dropping `HasStoredClassification` from its
  output, preventing the `(stored)` label from ever appearing in
  `--list-classifications`.
- `ApplyToPbix` was not saving the file path to settings after a successful
  write, so ADAM couldn't remember the path across sessions.
- File detection would hang indefinitely when scanning process handles
  containing pipe or console handles — now runs on a background thread with a
  2-second timeout.
- Classification overrides you set in ADAM were not surviving across Power BI
  sessions in `.pbip` projects. The override annotation was being written inside
  a column block in the table's definition file rather than at the table level,
  and Power BI strips unrecognised column-level annotations whenever it saves —
  so the override was silently lost on the next Power BI restart. ADAM now
  writes the annotation in the correct place and it persists reliably.
- Power BI's Auto date/time setting automatically creates hidden system tables
  for every date column in your model (named with prefixes like
  `DateTableTemplate_` and `LocalDateTable_` followed by a unique ID). These
  tables were previously visible to ADAM, causing them to appear in layouts and
  inflating the table count. ADAM now ignores these system tables entirely —
  they are excluded from diagrams, classification output, and relationship
  calculations.

### Changed
- Release and doc-sync workflows consolidated into a single `release.yml`.
- Test fixture renamed: `cascade test` → `Retail Sales`, `test thin report` →
  `Thin Report`; `.pbix` files moved to `test/pbix/`.

---

## [0.7.0] — 2026-05

### Added
- CLI mode (`adam.exe --cli --file <path.pbip>`).
- Per-fact layout mode — generates one diagram per fact table.
- Table classification persistence via TMDL annotations.
- Inverted-L layout style.
- Thin report detection and rejection with a clear error message.
- About window.
- Dark / Light theme support.

### Changed
- Layout engine refactored; disconnected tables placed in a left-hand column
  rather than being omitted.

---

## [0.6.x and earlier]

Initial development — waterfall and star-schema layout styles, basic PBI Desktop
integration via the External Tools ribbon, `.pbip` and `.pbix` write support.

[0.8.2]: https://github.com/GreyskullAnalytics/pbi-modelling-layouts/compare/v0.7.0...v0.8.2
[0.7.0]: https://github.com/GreyskullAnalytics/pbi-modelling-layouts/releases/tag/v0.7.0
