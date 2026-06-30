# Changelog

All notable changes to ADAM are documented here.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).  
Versions follow [Semantic Versioning](https://semver.org/).

---

## [0.9.1] - 2026-06

### Fixed
- The version pill could show pink (release) even when a build was published as
  a pre-release on GitHub, because ADAM was guessing pre-release status from the
  version number itself rather than checking how the release was actually
  published. ADAM now reads the real pre-release flag from the matching GitHub
  release, so the pill (and the update banner) always reflects what is shown on
  the Releases page.

---

## [0.9.0] — 2026-06

### Added
- A version pill now appears next to the ADAM title showing which build you are
  running. The pill is pink for release builds and yellow for pre-release builds,
  so you can see your release channel at a glance without opening the About window.
- The update notification banner is now channel-aware. If you are on a release
  build, ADAM only alerts you to new release builds (shown with a pink banner).
  If you are on a pre-release build, ADAM alerts you to new pre-release versions
  too (shown with a yellow banner). Release-channel users will no longer be prompted to
  install pre-release versions.
- Clicking Apply now opens a dedicated **Apply Layout** window before anything is
  written to your file. All decisions — resolving naming conflicts, renaming
  diagram tabs, queuing additional layouts, and choosing whether to close Power BI
  Desktop — are shown together on one screen. A single **Apply Now** button
  executes everything.
- Diagram names can now be customised directly in the Apply window before
  committing. Click the name field on any row to rename the diagram tab that will
  appear in Power BI Desktop.
- Naming conflicts are now resolved inline in the Apply window. Each conflicting
  diagram shows an **OVERWRITE** and **ADD AS NEW** toggle so you can see exactly
  what will happen before clicking Apply Now. When more than one conflict exists,
  a "Resolve all" shortcut lets you set all of them at once.
- When applying a Full Model layout and an existing "All Tables" diagram with a
  different style is already in the file (for example "All Tables - Waterfall"
  when you are about to apply Star Schema), the Apply window now offers to replace
  it in-place rather than silently adding a second diagram alongside it. A hint
  line beneath the diagram name explains which existing diagram will be replaced.
- Queued layouts now persist their conflict resolution choices across Apply window
  sessions. If you queued a layout and chose ADD AS NEW for a naming conflict, that
  choice is still shown and can be changed when you return to add a second layout.
- When applying to a **.pbip** file, the Apply window now explains that the layout
  is written to your project files on disk but will only appear in Power BI Desktop
  after the file is closed and reopened. A checkbox lets you choose whether ADAM
  should close and reopen Power BI Desktop immediately or leave it open so you can
  do so manually at a convenient time. Your preference is remembered for next time.
- The column headers **DIAGRAM NAME** and **LAYOUT** are now shown above the
  diagram list in the Apply window. The DIAGRAM NAME header includes a "— click to
  rename" hint so it is clear that diagram names are editable.
- Each row in the Apply window now has a trash icon on the far right. Clicking it
  removes that diagram from the list before anything is applied. If you remove
  everything, Apply Now disables automatically so you cannot confirm an empty apply.

### Fixed
- Pre-release detection was previously based on the version number major component
  being zero (v0.x.x = pre-release). ADAM now reads the full build tag instead,
  so a tagged release such as v1.0.0 correctly shows a pink release pill,
  and any build with a pre-release suffix in its tag (such as v0.9.0-preview.1)
  correctly shows yellow. Development builds between tags are also shown as
  pre-release.
- When ADAM was connected to one Power BI file and a second file launched ADAM
  from the External Tools ribbon, any in-progress Apply window for the first file
  was left open in the background. The Apply window is now dismissed automatically
  when ADAM switches to a new model, and the layout queue from the previous file
  is cleared so it cannot be accidentally applied to the wrong model.
- In dark mode, the text cursor was invisible when clicking into a diagram name
  field in the Apply window because it defaulted to the system colour. It now
  matches the text colour and is clearly visible in both themes.
- In light mode, the Apply Layout button on the main window and the Apply Now
  button in the Apply window appeared dark navy instead of purple. Both buttons
  now consistently use the brand purple colour in light and dark mode.
- The version shown in the About window now matches the version pill on the main
  window. Both display the full build string including any pre-release suffix, so
  you see the same version wherever you look.
- The "Resolve all" batch buttons in the Apply window were showing even when only
  one diagram had a naming conflict, which made no sense since there was nothing
  to batch. They now only appear once there are two or more conflicts to resolve
  at once.
- If you renamed a conflicting diagram to a name that didn't already exist, the
  OVERWRITE option stayed selectable even though there was nothing to overwrite at
  that name. ADAM now switches that row to ADD AS NEW automatically and greys out
  OVERWRITE (with a tooltip explaining why) whenever your typed name doesn't match
  an existing diagram. Toggling between the auto-generated names is unaffected —
  this only applies once you've typed something custom.

### Changed
- The intermediate dialogs that previously appeared during the Apply flow — "Close
  Power BI Desktop?", the .pbix unsupported format warning, "Add another layout
  before applying?", and the per-diagram conflict dialogs — have all been replaced
  by the new Apply window. There is now one confirmation step instead of up to
  five.
- For **.pbix** files, the Apply window clearly states upfront that Power BI
  Desktop must be closed to write the layout, rather than asking mid-flow after
  you have already committed to applying.
- The Apply button colour is now used consistently across both the main window and
  the Apply Now button in the Apply window.

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
- `docs/test-plans/test-plan-0.8.0.md` manual test plan.

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
