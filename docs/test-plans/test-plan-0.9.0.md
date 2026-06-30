# ADAM v0.9.0 — Manual Test Plan

Run through this plan after each significant development round for the v0.9.0 release.  
Mark each scenario **Pass / Fail / Skip** (not applicable to your setup).

Sections 1–8 cover new v0.9.0 features.  
Sections 9–16 are full regression of all existing behaviour.

---

## Environment setup checklist

Before starting, confirm you have access to:

- [ ] PBI Desktop **Store App** (Microsoft Store version)
- [ ] PBI Desktop **Standalone installer** version (if available)
- [ ] `test/pbip/Retail Sales.pbip` — primary test fixture (2 facts, 5 dims)
- [ ] `test/pbip/Space Exploration Agency.pbip` — large model fixture (~25 tables, 6 facts)
- [ ] `test/pbix/Retail Sales.pbix`
- [ ] A thin report `.pbip` (connects to Power BI Service)
- [ ] ADAM **debug build**: `src/ADAM/bin/Debug/net8.0-windows/ADAM.exe`
- [ ] ADAM **pre-release build** (untagged commit ahead of last tag, or a `-preview.x` tagged build)
- [ ] ADAM **release build** (built from a `v1.x.x` or later tag — or simulate by temporarily tagging)
- [ ] Log file open in a viewer: `%LOCALAPPDATA%\ADAM\adam.log`
- [ ] Settings file accessible: `%LOCALAPPDATA%\ADAM\settings.json`

---

## 1. Version pill and release channel indicator

### 1.1 Pre-release build — yellow pill
1. Run a build where the HEAD is ahead of the last git tag (MinVer generates `x.y.z-alpha.0.N`).
2. Launch ADAM.
3. **Expected:** A yellow pill appears next to the "ADAM" title. It shows the full version string including the pre-release suffix (e.g. `v0.9.0-alpha.0.3`).

### 1.2 Release build — pink pill
1. Build from a commit exactly on a `vX.Y.Z` tag with no pre-release suffix.
2. Launch ADAM.
3. **Expected:** A pink pill (`#F1497A`) appears showing the version (e.g. `v0.9.0`). No suffix.

### 1.3 Pill colour is distinct from background in both themes
1. Toggle between Dark and Light mode.
2. **Expected:** Yellow pill and pink pill are both clearly readable against the window background in each theme.

### 1.4 Version matches assembly / MinVer
1. Note the version shown in the pill.
2. Run `src/ADAM/bin/Debug/net8.0-windows/ADAM.exe --cli --file non-existent.pbip` and check the User-Agent header in the log, or compare against `git describe --tags`.
3. **Expected:** Versions match.

### 1.5 About window version matches pill
1. Note the version shown in the main window pill.
2. Click **About**.
3. **Expected:** The version shown in the About window is identical to the pill — same string, same suffix (e.g. both show `0.9.0-alpha.0.3` on a dev build, or both show `0.9.0` on a tagged release build). No "- Public Preview" suffix appended separately.

---

## 2. Update banner — release channel awareness

### 2.1 Pre-release build notified of newer pre-release
1. Run a pre-release build that is older than the newest published pre-release on GitHub.
2. Launch ADAM and wait up to ~10 seconds.
3. **Expected:** A **yellow** banner appears at the top of the window. Text mentions the newer version. "Download now" button present.

### 2.2 Release build notified of newer stable only
1. Run a release build older than the newest release build.
2. **Expected:** A **pink** banner appears. Text mentions the newer version.

### 2.3 Release build not notified of pre-release-only updates
1. Run a release build. The only newer release on GitHub is marked as pre-release.
2. **Expected:** No update banner appears.

### 2.4 Banner dismiss persists for the session
1. When the banner appears, click the × dismiss button.
2. **Expected:** Banner disappears. Does not reappear if ADAM is left open.

### 2.5 "Download now" opens the release page
1. Click "Download now" in the banner.
2. **Expected:** The GitHub release page opens in the default browser. ADAM closes (as it always has).

### 2.6 Both banners readable in both themes
1. With a yellow update banner visible, toggle to Dark mode and then Light mode.
2. **Expected:** Yellow banner with dark text and pink banner with white text both remain clearly legible.

---

## 3. Apply window — structure and layout

### 3.1 Apply window opens on clicking Apply
1. Open `Retail Sales.pbip` in PBI Desktop, connect ADAM.
2. Click **Apply Waterfall Layout – Full Model**.
3. **Expected:** The Apply window opens. Main ADAM window remains open behind it.

### 3.2 Layout summary not shown at top
1. Inspect the Apply window header.
2. **Expected:** Only "APPLY LAYOUT" title visible in the header. No "Waterfall — Full Model" subtitle at the top.

### 3.3 LAYOUT column shows style and mode per row
1. Check each diagram row.
2. **Expected:** A third column shows the style and mode (e.g. `Waterfall  ·  Full Model`) in subtle text.

### 3.4 DIAGRAM NAME column header visible
1. **Expected:** Column headers "DIAGRAM NAME" and "LAYOUT" appear above the diagram list, separated by a thin divider.

### 3.5 DIAGRAM NAME field is editable
1. Click into the name field of a diagram row.
2. **Expected:** Cursor appears. You can type to change the name. The field shows a subtle bottom border on focus.

### 3.6 Apply Now button matches main Apply button colour — dark mode
1. Ensure ADAM is in dark mode.
2. Compare the **Apply Now** button in the Apply window with the main **Apply Layout** button on the main window.
3. **Expected:** Both buttons are brand purple (`#5F00C5`).

### 3.7 Apply Now button matches main Apply button colour — light mode
1. Switch to light mode.
2. Compare the same two buttons.
3. **Expected:** Both buttons remain brand purple. Neither appears dark navy or any other colour.

### 3.8 Apply Now disabled state is visually obvious
1. Open the Apply window and delete all rows using the trash icons.
2. **Expected:** The **Apply Now** button becomes clearly muted/faded (approximately 30% opacity) so it is obvious the button cannot be clicked. The **+ Queue another layout** button remains visually active.

### 3.9 Cursor visible in rename field — dark mode
1. Switch to dark mode.
2. Open the Apply window and click into a DIAGRAM NAME field.
3. **Expected:** The text cursor (caret) is clearly visible — it should be a light colour matching the text, not an invisible black bar.

### 3.10 Cancel closes without applying
1. Open the Apply window and click **Cancel**.
2. **Expected:** Window closes. No diagrams written. `diagramLayout.json` unchanged. Status bar in main window shows "Cancelled."

---

## 4. Apply window — conflict resolution

### 4.1 First apply — no conflict → [NEW] badge
1. Use a `.pbip` that has no existing model view diagrams.
2. Click Apply.
3. **Expected:** Each diagram row shows a green **[NEW]** badge. No conflict controls.

### 4.2 Exact name conflict → [OVERWRITE] / [ADD AS NEW] toggle
1. Apply a Waterfall Full Model layout (creates "All Tables - Waterfall").
2. Click Apply again with the same style.
3. In the Apply window:
   **Expected:** "All Tables - Waterfall" row shows **[OVERWRITE]** (orange, active) and **[ADD AS NEW]** (blue, outlined/inactive) pill buttons.

### 4.3 OVERWRITE keeps the same name
1. In the conflict row from 4.2, ensure OVERWRITE is selected (default).
2. **Expected:** Name field shows "All Tables - Waterfall". Clicking Apply Now overwrites the existing diagram.

### 4.4 ADD AS NEW generates a unique name
1. Click **ADD AS NEW** on the conflict row.
2. **Expected:** Name field updates to an auto-incremented name (e.g. "All Tables - Waterfall 2"). Apply Now creates a new diagram alongside the original.

### 4.5 Inactive pill is clearly outlined (not disabled)
1. Inspect the inactive pill (e.g. [ADD AS NEW] when OVERWRITE is active).
2. **Expected:** The inactive pill has a visible coloured border with matching text — it looks like a pressable outlined button, not a disabled badge.

### 4.6 All Tables variant conflict — overwrite different style
1. Apply Waterfall Full Model (creates "All Tables - Waterfall").
2. Switch to Star Schema Full Model and click Apply.
3. **Expected:** Apply window shows "All Tables - Star Schema" row with **[OVERWRITE]** / **[ADD AS NEW]** toggle and a hint text beneath the name: *replaces existing "All Tables - Waterfall"*.
4. Select OVERWRITE. Click Apply Now.
5. **Expected:** "All Tables - Waterfall" diagram is replaced in-place; "All Tables - Waterfall" name no longer exists; "All Tables - Star Schema" name now appears.

### 4.7 Variant conflict — Add alongside
1. Repeat 4.6 but select **ADD AS NEW**.
2. **Expected:** Both "All Tables - Waterfall" and "All Tables - Star Schema" exist after apply.

### 4.8 Batch resolve — Overwrite all (multiple per-fact conflicts)
1. Apply Per-Fact Waterfall (creates "Store Sales", "Store Items").
2. Click Apply again with the same style.
3. **Expected:** Both rows show the toggle. "Resolve all:" buttons visible.
4. Click **Overwrite** batch button.
5. **Expected:** Both rows switch to OVERWRITE active.

### 4.9 Batch resolve — Add all as new
1. In the same scenario, click **Add as new** batch button.
2. **Expected:** Both rows switch to ADD AS NEW active, names show auto-incremented versions.

### 4.10 Batch resolve visible only with 2+ conflicts
1. Apply Full Model (1 diagram) when a conflict exists.
2. **Expected:** "Resolve all:" buttons are **not** visible (only one conflict).

---

## 5. Apply window — custom diagram naming

### 5.1 Typing a custom name
1. In the Apply window, click into the DIAGRAM NAME field and type a custom name (e.g. "My Custom Diagram").
2. Click Apply Now.
3. **Expected:** The diagram tab in Power BI Desktop is named "My Custom Diagram".

### 5.2 Toggle does not overwrite custom name
1. Type a custom name into a conflict row's name field.
2. Toggle between OVERWRITE and ADD AS NEW.
3. **Expected:** The name field retains your custom text and is not replaced by the auto-generated name.

### 5.3 Clearing custom name restores auto name on toggle
1. After typing a custom name, clear the field completely.
2. Toggle between OVERWRITE and ADD AS NEW.
3. **Expected:** The name field updates to the auto-generated name (either original or unique) on each toggle.

### 5.4 Custom name preserved through queue
1. Type a custom name into a diagram row.
2. Click **+ Queue another layout** instead of Apply Now.
3. Change style/mode and click Apply again.
4. **Expected:** The queued row shows the custom name you typed in the first session.

### 5.5 Empty or whitespace name falls back to auto name
1. Clear the name field completely and click Apply Now.
2. **Expected:** ADAM uses the auto-generated `FinalName` (not an empty string). No error.

### 5.6 Trash icon removes a diagram row
1. Open the Apply window with at least two diagram rows.
2. Click the trash icon on the far right of one row.
3. **Expected:** That row disappears. Remaining rows are unaffected. The row count decreases by one.

### 5.7 Apply Now disables when all rows are removed
1. Open the Apply window with exactly one diagram row.
2. Click the trash icon to remove it.
3. **Expected:** The **Apply Now** button becomes visibly disabled (faded). The list is empty.

### 5.8 Queue button stays enabled when list is empty
1. In the empty state from 5.7, inspect the **+ Queue another layout** button.
2. Click it.
3. **Expected:** The button is still enabled and clickable. Clicking it closes the Apply window without applying anything (equivalent to cancel — the queue is cleared and the Apply button label on the main window reflects zero queued items).

### 5.9 Typed name with no match disables OVERWRITE
1. In a conflict row (from 4.2), type a custom name that does not match any existing diagram (e.g. "Totally New Name").
2. **Expected:** The **OVERWRITE** pill greys out further (lower opacity, arrow cursor, tooltip explaining there's nothing to overwrite at that name) and **ADD AS NEW** becomes active. Clicking OVERWRITE has no effect.

### 5.10 Auto-generated names can still be toggled freely
1. In a conflict row (from 4.2), without typing anything, click **ADD AS NEW**, then click **OVERWRITE** again.
2. **Expected:** Toggling works normally both ways — the auto-generated "add as new" name never disables OVERWRITE, since the restriction only applies to hand-typed custom names (per 5.9).

---

## 6. Apply window — queuing multiple layouts

### 6.1 Queue another layout button present
1. Open the Apply window.
2. **Expected:** A **+ Queue another layout** button is visible at the bottom left.

### 6.2 Clicking Queue stores diagrams and returns to main window
1. Click **+ Queue another layout**.
2. **Expected:** Apply window closes. Main window Apply button shows queued count (e.g. `Apply Star Schema Layout – Full Model  [1 queued]`). Status bar shows queued confirmation.

### 6.3 Second Apply opens window with queued items
1. After queueing one layout, change style/mode and click Apply again.
2. **Expected:** Apply window reopens. The queued diagram(s) are listed above the new ones. They show their style/mode in the LAYOUT column. No [QUEUED] badge.

### 6.4 Queued items have no special badge
1. Inspect the queued rows in the Apply window opened in 6.3.
2. **Expected:** Queued rows that had no conflict show no badge in the action column. Queued rows that had a conflict still show the OVERWRITE/ADD AS NEW toggle.

### 6.5 Queued conflict state persists
1. Queue a layout where you chose ADD AS NEW for a conflict.
2. Reopen the Apply window (step 6.3).
3. **Expected:** The queued row still shows ADD AS NEW as active.

### 6.6 User can change queued conflict resolution
1. In the Apply window from 6.3, click OVERWRITE on the previously-queued ADD AS NEW row.
2. Click Apply Now.
3. **Expected:** The final diagram uses the overwrite name, not the "Add as New" unique name.

### 6.7 Apply Now writes all queued + new diagrams
1. With 1 queued layout and 1 new layout, click Apply Now.
2. **Expected:** Both diagrams are written to the file. PBI Desktop reopens with both visible.

### 6.8 Queue is cleared after successful Apply
1. After Apply Now completes, click Apply again.
2. **Expected:** Apply window opens with only new diagrams — no residual queued items.

### 6.9 Queue cleared on model switch
1. Queue at least one layout for model A.
2. Open model B in PBI Desktop and click External Tools → ADAM.
3. **Expected:** ADAM switches to model B. Queue count gone from Apply button label. No queued items visible when Apply is clicked for model B.

### 6.10 Apply window dismissed on model switch
1. With the Apply window open (do not click Apply Now or Queue), open a second PBI Desktop file and click External Tools → ADAM.
2. **Expected:** The Apply window closes automatically. ADAM switches to the new model. No stale state.

---

## 7. PBIP — close/reopen option

### 7.1 Blue info banner shown for .pbip
1. With `Retail Sales.pbip` connected, click Apply.
2. **Expected:** Apply window shows a blue left-accented banner reading "ADAM will write the layout to your .pbip files on disk. The new diagram won't be visible in Power BI Desktop until the file is closed and reopened."

### 7.2 Checkbox present and checked by default (first use)
1. On first use (no `closeAfterApplyPbip` setting saved), open the Apply window.
2. **Expected:** "Close and reopen Power BI Desktop now" checkbox is **checked** by default.

### 7.3 No mandatory close warning shown for .pbip
1. **Expected:** The blue banner for .pbix ("must be closed to write") is **not** shown for .pbip.

### 7.4 Checkbox checked — PBI closes and reopens
1. Ensure the checkbox is checked. Click Apply Now.
2. **Expected:** PBI Desktop closes, layout is written, PBI Desktop reopens. Status bar: "Done! Power BI Desktop is reopening with the new ADAM diagram applied."

### 7.5 Checkbox unchecked — PBI stays open
1. Uncheck the checkbox. Click Apply Now.
2. **Expected:** PBI Desktop remains open throughout. Status bar: "Done! Close and reopen Power BI Desktop to see the new diagram."

### 7.6 Layout is actually written when unchecked
1. After step 7.5, manually close and reopen the `.pbip` file in PBI Desktop.
2. **Expected:** The new diagram(s) appear in the Model View.

### 7.7 TMDL annotations written in both paths
1. After 7.4 and after 7.5 (two separate runs), inspect `Retail Sales.SemanticModel/definition/tables/*.tmdl`.
2. **Expected:** `ADAM_Classification` annotations are present in both cases.

### 7.8 Setting is saved and restored
1. Uncheck the checkbox and click Apply Now.
2. Close ADAM, relaunch, and click Apply again.
3. **Expected:** The checkbox is **unchecked** (the saved preference is honoured).

### 7.9 Changing and saving the preference
1. Check the checkbox and click Apply Now.
2. Relaunch ADAM and click Apply.
3. **Expected:** Checkbox is **checked**.

---

## 8. PBIX — mandatory close/reopen messaging

### 8.1 Two warnings shown for .pbix
1. Connect to `Retail Sales.pbix` and click Apply.
2. **Expected:** Apply window shows two banners:
   - Blue: "Power BI Desktop must be closed to write to this .pbix file. ADAM will close it automatically, apply the layout, then reopen it. Save any unsaved changes before clicking Apply Now."
   - Orange: the .pbix unsupported format warning with backup info.

### 8.2 No checkbox for .pbix
1. **Expected:** No "Close and reopen" checkbox visible in the Apply window for .pbix.

### 8.3 Close/reopen is automatic (no user choice)
1. Click Apply Now on a .pbix file.
2. **Expected:** PBI Desktop closes and reopens automatically — the user is not asked.

### 8.4 Backup created before write
1. After Apply Now on a .pbix, check the folder.
2. **Expected:** A timestamped backup exists (e.g. `Retail Sales_ADAM_backup_2026-xx-xx.pbix`).

### 8.5 .pbix warns upfront — user can cancel before closing PBI
1. Click Apply, review the warnings, click **Cancel**.
2. **Expected:** PBI Desktop is still open. No changes made. Backup not created.

---

## 9. Regression — file detection

### 9.1 .pbip opened via External Tools ribbon (Store App)
1. Open `Retail Sales.pbip` in PBI Desktop Store App.
2. Click External Tools → ADAM.
3. **Expected:** ADAM connects, file path shown in footer, no picker.

### 9.2 .pbip opened via External Tools ribbon (Standalone)
1. Repeat 9.1 using the standalone installer.
2. **Expected:** Same outcome.

### 9.3 .pbix opened via double-click (command line path)
1. Double-click `Retail Sales.pbix` to open it.
2. Click External Tools → ADAM.
3. **Expected:** File path detected automatically via command-line args.

### 9.4 .pbix opened via Open Recent (Store App)
1. Close and reopen via File → Open Recent (Store App).
2. Click External Tools → ADAM.
3. **Expected:** File path shown. Log shows `FindFileByWindowTitle`.

### 9.5 .pbix opened via Open Recent (Standalone)
1. Repeat 9.4 using standalone installer.
2. **Expected:** Same outcome. Log may show `FindOpenFileByPid` or handle-based discovery.

### 9.6 New unsaved file
1. Open PBI Desktop, create a blank report, do not save.
2. Click External Tools → ADAM.
3. **Expected:** Model loads, file path unknown, Apply prompts with clear message.

### 9.7 Multiple files open — External Tools from each
1. Open `Retail Sales.pbip` and `Space Exploration Agency.pbip` simultaneously.
2. Click External Tools → ADAM from each.
3. **Expected:** ADAM correctly switches between models. Log shows `Pipe: received new args`.

### 9.8 Two files with the same name in different folders
1. Copy `Retail Sales.pbix` to two folders.
2. Open the one in the less-obvious folder via Open Recent.
3. **Expected:** ADAM finds the correct file (verified by path shown in footer).

### 9.9 File path cached after first successful detection (.pbip)
1. Open a `.pbip` via Open Recent, connect ADAM, apply successfully.
2. Close PBI Desktop and reopen the file via Open Recent.
3. Launch ADAM.
4. **Expected:** File path shown immediately from settings cache.

### 9.10 File path cached after first successful detection (.pbix)
1. Repeat 9.9 with a `.pbix`.
2. **Expected:** Same — path retrieved from settings, no picker.

---

## 10. Regression — layout generation

For each row, apply the layout and verify the diagram in PBI Desktop Model View.

| # | Model | Format | Style | Mode | Expected diagrams |
|---|---|---|---|---|---|
| 10.1 | Retail Sales | .pbip | Waterfall | Full Model | 1 × "All Tables - Waterfall" |
| 10.2 | Retail Sales | .pbip | Waterfall | Per Fact | 2 × fact-named diagrams |
| 10.3 | Retail Sales | .pbip | Star Schema | Full Model | 1 × "All Tables - Star Schema" |
| 10.4 | Retail Sales | .pbip | Star Schema | Per Fact | 2 × fact-named diagrams |
| 10.5 | Retail Sales | .pbip | Inverted L | Full Model | 1 × "All Tables - Inverted L" |
| 10.6 | Retail Sales | .pbip | Inverted L | Per Fact | 2 × fact-named diagrams |
| 10.7 | Retail Sales | .pbix | Waterfall | Full Model | 1 × "All Tables - Waterfall" |
| 10.8 | Retail Sales | .pbix | Star Schema | Per Fact | 2 × fact-named diagrams |
| 10.9 | Space Exploration Agency | .pbip | Waterfall | Full Model | 1 × all ~25 tables |
| 10.10 | Space Exploration Agency | .pbip | Waterfall | Per Fact | 6 × fact-named diagrams |
| 10.11 | Space Exploration Agency | .pbip | Star Schema | Full Model | 1 × all tables radial |
| 10.12 | Space Exploration Agency | .pbip | Inverted L | Per Fact | 6 × fact-named diagrams |

**For each:** verify in the Model View:
- Tables are placed without overlap.
- Relationships are visible and correctly connected.
- Fact tables are in expected positions relative to dimensions.
- OutriggerBridge (Time Period / Outrigger Astronaut Quals etc.) does not overlap its parent dimension.
- Disconnected tables appear in a column (Waterfall/Inverted L) or outside the star (Star Schema).

---

## 11. Regression — table classification

### 11.1 Auto-classification accuracy (Retail Sales)
1. Connect ADAM to `Retail Sales.pbip` or `.pbix`.
2. Review the table list before applying.
3. **Expected:** Store Sales = FACT, Store Items = FACT, Calendar / Channel / Item / Store = DIM, Time Period = OUTRIGGER, @Measures = DISCONNECTED.

### 11.2 Auto-classification accuracy (Space Exploration Agency)
1. Connect ADAM to `Space Exploration Agency.pbip`.
2. **Expected:** All 6 Fact_* tables classified FACT, all Dim_* tables classified DIM, Bridge Mission Astronaut = BRIDGE, Outrigger tables = OUTRIGGER, _Measures / _Scenario Parameter = DISCONNECTED.

### 11.3 Manual override — single table
1. Override one classification using the dropdown.
2. Apply.
3. Relaunch ADAM.
4. **Expected:** Override remembered (TMDL annotation for .pbip, settings for .pbix).

### 11.4 Override persists across PBI restarts (.pbip)
1. Apply a manual override for .pbip.
2. Close PBI Desktop completely and reopen.
3. Relaunch ADAM.
4. **Expected:** Override still shown.

### 11.5 Override persists across PBI restarts (.pbix)
1. Repeat 11.4 for .pbix.
2. **Expected:** Override stored in settings, not TMDL.

### 11.6 System tables excluded
1. Open a model that uses PBI's Auto date/time feature (creates `DateTableTemplate_*` and `LocalDateTable_*` tables).
2. **Expected:** These system tables do not appear in ADAM's table list.

### 11.7 CLI — list classifications
```
ADAM.exe --cli --file "test/pbip/Retail Sales.pbip" --list-classifications
```
**Expected:** Table list printed with correct classifications. Exit code 0.

### 11.8 CLI — set classification
```
ADAM.exe --cli --file "test/pbip/Retail Sales.pbip" --set-classification "Calendar=Fact"
```
Then reopen ADAM on the file.  
**Expected:** Calendar shows as FACT (stored override).

---

## 12. Regression — .pbip write cycle (with new close/reopen option)

### 12.1 Close/reopen path — layout visible immediately
1. Apply with checkbox checked.
2. **Expected:** PBI Desktop reopens to the model; new diagram visible in Model View.

### 12.2 No-close path — layout in file but not yet visible
1. Apply with checkbox unchecked.
2. Inspect `diagramLayout.json` directly — confirm the new diagram JSON is present.
3. Keep PBI Desktop open; do NOT close it.
4. **Expected:** PBI Desktop Model View does not yet show the new diagram.
5. Close and reopen PBI Desktop.
6. **Expected:** Diagram now visible.

### 12.3 Multiple diagrams in one session
1. Queue a Waterfall Full Model and a Per-Fact Inverted L.
2. Apply both in one go (Apply Now from second Apply window).
3. **Expected:** Both diagram sets written in one close/reopen cycle.

---

## 13. Regression — .pbix write cycle

### 13.1 Backup created automatically
1. Apply to a `.pbix`.
2. **Expected:** Timestamped backup exists in the same folder.

### 13.2 PBI Desktop reopens automatically
1. **Expected:** PBI Desktop closes, writes, reopens. No manual step needed.

### 13.3 User cancels PBI save dialog — ADAM recovers
1. Apply to a `.pbix` with unsaved changes.
2. When PBI's save prompt appears, click **Cancel** (keep PBI open).
3. **Expected:** After ~10 seconds ADAM shows "Power BI Desktop hasn't closed yet — if you cancelled the save dialog, save your changes first and click Apply again." ADAM re-enables the Apply button.

### 13.4 Two .pbix files open — ADAM closes correct one
1. Open two different `.pbix` files.
2. Launch ADAM from the second one.
3. Apply.
4. **Expected:** Only the second file's PBI Desktop instance closes. The first remains open.

---

## 14. Regression — CLI mode

### 14.1 Basic CLI apply
```
ADAM.exe --cli --file "test/pbip/Retail Sales.pbip"
```
**Expected:** Layout written, exit 0, success output to stdout.

### 14.2 Style flags
```
ADAM.exe --cli --file "..." --style star-schema
ADAM.exe --cli --file "..." --style inverted-l
ADAM.exe --cli --file "..." --style waterfall
```
**Expected:** Correct layout for each.

### 14.3 Per-fact mode
```
ADAM.exe --cli --file "..." --mode per-fact
```
**Expected:** One diagram per fact table written.

### 14.4 Overwrite flag
```
ADAM.exe --cli --file "..." --overwrite
```
**Expected:** Existing diagrams of the same name are overwritten without error.

### 14.5 File not found — clear error
```
ADAM.exe --cli --file "does-not-exist.pbip"
```
**Expected:** Clear error to stderr, exit code 1.

### 14.6 .pbix rejected
```
ADAM.exe --cli --file "test/pbix/Retail Sales.pbix"
```
**Expected:** "CLI mode only supports .pbip files." error, exit 1.

---

## 15. Regression — single-instance and model switching

### 15.1 Second External Tools launch forwards to existing ADAM
1. With ADAM open on model A, click External Tools → ADAM from model B.
2. **Expected:** No second window. Existing window switches to model B. Log shows `Pipe: received new args`.

### 15.2 Switching back
1. After 15.1, click External Tools → ADAM from model A.
2. **Expected:** ADAM switches back to model A correctly.

### 15.3 Apply window dismissed on switch
1. Open the Apply window for model A. Do not click Apply Now.
2. Click External Tools → ADAM from model B.
3. **Expected:** Apply window closes. Main window shows model B's tables. Queue count cleared.

### 15.4 Queue cleared on switch
1. Queue one layout for model A.
2. Switch to model B via External Tools.
3. Click Apply.
4. **Expected:** Apply window shows only new items for model B — no residual queued items from model A.

---

## 16. Regression — edge cases and error handling

### 16.1 File deleted between detection and Apply
1. ADAM detects file path.
2. Delete or move the file.
3. Click Apply.
4. **Expected:** Helpful error message. No crash.

### 16.2 Model with no relationships
1. Open a model with tables but no relationships.
2. Apply each layout style.
3. **Expected:** Tables laid out without error.

### 16.3 Model with only one table
1. Apply each style to a single-table model.
2. **Expected:** No crash. Single table placed.

### 16.4 Large model — Space Exploration Agency
1. Apply Waterfall Full Model.
2. **Expected:** All ~25 tables placed without overlap. Completes in under 10 seconds.

### 16.5 Special characters in model/file name
1. Open a file with spaces, parentheses, or hyphens in its name.
2. Apply.
3. **Expected:** No errors.

### 16.6 ADAM launched standalone (no PBI open)
1. Launch `ADAM.exe` directly with no arguments.
2. **Expected:** Manual connect UI shown. No crash.

### 16.7 Thin report rejected gracefully
1. Open a thin report `.pbip` (Power BI Service-hosted model).
2. Click External Tools → ADAM.
3. **Expected:** Clear "Thin Reports Not Supported" message. No crash.

---

## 17. Regression — theme and settings persistence

### 17.1 Theme persists across restarts
1. Switch to Dark mode. Close ADAM. Relaunch.
2. **Expected:** Dark mode is still active.

### 17.2 Layout style selection persists
1. Select Star Schema. Close ADAM. Relaunch.
2. **Expected:** Star Schema is still selected.

### 17.3 Close/reopen preference persists (.pbip)
1. Uncheck "Close and reopen PBI Desktop now". Apply.
2. Close ADAM. Relaunch. Click Apply.
3. **Expected:** Checkbox is still unchecked.

### 17.4 Settings survive a version upgrade
1. Note current settings.
2. Install/replace with a newer build.
3. **Expected:** Saved classifications, file paths, theme, and close/reopen preference are all preserved.

---

## 18. Combined multi-layout workflow (end-to-end)

This section tests the complete new UX flow as a user would experience it.

### 18.1 Full workflow — .pbip, queue two layouts, custom names, no-close path

1. Open `Space Exploration Agency.pbip` in PBI Desktop.
2. Launch ADAM from External Tools.
3. Select **Star Schema**, **Full Model**.
4. Click Apply. Apply window opens.
   - Rename "All Tables - Star Schema" to "Overview".
   - Uncheck "Close and reopen Power BI Desktop now".
   - Click **+ Queue another layout**.
5. Select **Waterfall**, **Per Fact Table**.
6. Click Apply. Apply window opens again.
   - Queued "Overview" row shown, no badge, style = Star Schema · Full Model.
   - 6 new Waterfall Per Fact rows shown, each with [NEW] badge.
   - Click Apply Now.
7. **Expected:** Both layouts written without closing PBI. Status: "Done! Close and reopen Power BI Desktop to see the new diagram."
8. Close and reopen the file manually.
9. **Expected:** "Overview" diagram and all 6 per-fact diagrams visible in Model View.

### 18.2 Full workflow — .pbix, overwrite existing, close/reopen mandatory

1. Open `Retail Sales.pbix` in PBI Desktop via double-click.
2. Apply a Waterfall Full Model layout (first time).
3. Click Apply again with Inverted L Full Model.
4. Apply window opens:
   - Blue mandatory-close banner and orange .pbix warning both visible.
   - No checkbox.
   - "All Tables - Waterfall" shows variant conflict → OVERWRITE active.
5. Click Apply Now.
6. **Expected:** PBI Desktop closes. Backup created. Layout written. PBI reopens. "All Tables - Waterfall" diagram replaced with Inverted L layout.

### 18.3 Model switch mid-queue

1. Queue a layout for `Retail Sales.pbip`.
2. Open `Space Exploration Agency.pbip` and click External Tools → ADAM.
3. **Expected:** Queue cleared, Apply window dismissed if open. ADAM shows Space Exploration Agency tables.
4. Click Apply for Space Exploration Agency.
5. **Expected:** No residual Retail Sales diagrams queued.

---

## Automation notes

**What is covered by the automated test suite (`ADAM.Tests`):**
- Layout algorithm — node positions, no-overlap, OutriggerBridge placement, coordinate shifting.
- Table classifier — all classification rules, stored annotation handling, `HasStoredClassification` copy.
- `SettingsManager` round-trips.
- `ModelReader` offline parsing.
- `LayoutWriter` read/write round-trips (.pbip only).
- **`DiagramActionItem`** — conflict detection (exact and variant), `FinalName`/`EffectiveName` computation, custom name override, `IsPending` suppression, copy constructor fidelity. (28 tests added in v0.9.0.)
- **CLI integration** — `CliIntegrationTests` spawns `ADAM.exe --cli` against a temp copy of `Retail Sales.pbip` and a programmatically-generated large model (6 facts), validating output diagram counts, no-overlap, and correct fact isolation.

**What must remain manual for this release:**
- All Apply window UI interactions (WPF dialog flow).
- Close/reopen option and PBI Desktop lifecycle.
- Update banner appearance.
- File detection across SA/SI/Open Recent/command-line variants.
- `.pbix` write cycle (requires an actual .pbix file and PBI Desktop).
- Single-instance pipe forwarding.
