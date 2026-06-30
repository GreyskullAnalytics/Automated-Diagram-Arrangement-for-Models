# ADAM — Manual Test Plan

Run through this plan after each significant round of development.  
Mark each scenario Pass / Fail / Skip (not applicable to your setup).

---

## Environment setup checklist

Before starting, confirm you have access to:

- [ ] PBI Desktop **Store App** (Microsoft Store version)
- [ ] PBI Desktop **Standalone installer** version (if available)
- [ ] A `.pbip` project file (use `test/pbip/Retail Sales.pbip`)
- [ ] A `.pbix` file (use `test/pbix/Retail Sales.pbix`)
- [ ] A thin report `.pbip` (connects to Power BI Service — use `test/pbip/Thin Report.pbip` if available)
- [ ] ADAM debug build at `src/ADAM/bin/Debug/net8.0-windows/ADAM.exe`
- [ ] Log file open in a viewer: `%LOCALAPPDATA%\ADAM\adam.log`

---

## 1. File detection

### 1.1 `.pbip` — opened via External Tools ribbon
1. Open `Retail Sales.pbip` in PBI Desktop.
2. Click External Tools → ADAM.
3. **Expected:** ADAM opens, file path shown in footer, no file picker prompt.

### 1.2 `.pbix` — opened via External Tools ribbon (file double-clicked / command line)
1. Open `Retail Sales.pbix` by double-clicking it (not Open Recent).
2. Click External Tools → ADAM.
3. **Expected:** File path shown in footer, no file picker prompt.

### 1.3 `.pbix` — opened via Open Recent (Store App)
1. Close and reopen the `.pbix` file using **File → Open Recent** in PBI Desktop Store App.
2. Click External Tools → ADAM.
3. **Expected:** File path shown in footer. Log should show `FindFileByWindowTitle: found …`.

### 1.4 `.pbix` — opened via Open Recent (standalone installer)
1. Repeat 1.3 using the standalone installer version.
2. **Expected:** Same as 1.3. Log may show `FindOpenFileByPid` finding the path instead if the installer version keeps the file open.

### 1.5 New unsaved file
1. Open PBI Desktop and create a blank report — do **not** save it.
2. Click External Tools → ADAM.
3. **Expected:** ADAM loads the model, file path area is blank or hidden, Apply falls back to the manual file picker with a clear status message explaining why.

### 1.6 Multiple files open — all via command line
1. Open `cascade test.pbip` **and** `cascade test pbix.pbix` simultaneously.
2. Click External Tools → ADAM from **each** file in turn.
3. **Expected:** ADAM correctly identifies each file independently. Switching between them via the External Tools ribbon updates ADAM to show the correct file.

### 1.7 Multiple files open — mix of command-line and Open Recent
1. Open one `.pbix` via double-click and a second `.pbix` via Open Recent.
2. Launch ADAM from each.
3. **Expected:** Correct file shown for each. Log should show the discovery method used for each (`FindOpenFileByPid` vs `FindFileByWindowTitle`).

### 1.8 Two files with the same name in different folders
1. Create two copies of the same `.pbix` in two different folders (e.g. `Downloads` and `Desktop`).
2. Open one via Open Recent.
3. Launch ADAM.
4. **Expected:** ADAM finds a file (note which one). Verify it is the correct one. After clicking Apply once, relaunch ADAM and confirm the correct path is now cached.

### 1.9 File path remembers after PBI restart (`.pbix`)
1. Open a `.pbix` via Open Recent, launch ADAM, click Apply successfully.
2. Close PBI Desktop and reopen the same file via Open Recent.
3. Launch ADAM again.
4. **Expected:** File path shown immediately — retrieved from settings (`db.Name` key). Log should show `Using saved file path` or no file picker prompt.

### 1.10 File path remembers after PBI restart (`.pbip`)
1. Repeat 1.9 with a `.pbip` file.
2. **Expected:** Same — GUID-keyed settings lookup succeeds because `.pbip` GUIDs are stable.

---

## 2. Thin report detection

### 2.1 Thin report rejected gracefully
1. Open a thin report `.pbip` (one that connects to the Power BI Service).
2. Click External Tools → ADAM.
3. **Expected:** ADAM shows a clear message explaining thin reports are not supported. No crash, no misleading error.

### 2.2 XMLA endpoint in manual connect dropdown
1. Launch ADAM standalone (without PBI open or with no External Tools pass-through).
2. Open the instance dropdown — if a thin report is running it should appear.
3. Try to connect to it.
4. **Expected:** Same rejection message as 2.1.

---

## 3. Layout generation

For each of the following, verify the diagram appears correctly in PBI Desktop after Apply.

| # | File format | Layout style | Mode | Notes |
|---|---|---|---|---|
| 3.1 | `.pbip` | Waterfall | Per-Fact | One diagram per fact table |
| 3.2 | `.pbip` | Waterfall | Full Model | Single diagram, all tables |
| 3.3 | `.pbip` | Star Schema | Per-Fact | |
| 3.4 | `.pbip` | Star Schema | Full Model | |
| 3.5 | `.pbip` | Inverted L | Per-Fact | |
| 3.6 | `.pbip` | Inverted L | Full Model | |
| 3.7 | `.pbix` | Waterfall | Per-Fact | Requires close/reopen cycle |
| 3.8 | `.pbix` | Waterfall | Full Model | |
| 3.9 | `.pbix` | Star Schema | Full Model | |
| 3.10 | `.pbix` | Inverted L | Full Model | |

**For each:** open the resulting diagram in PBI Desktop and check:
- Tables are positioned without overlap.
- Relationships are visible and connected.
- Fact tables appear in expected positions relative to dimensions.
- Diagram name matches the expected naming convention.

---

## 4. Table classification

### 4.1 Auto-classification accuracy
1. Open a model with a known structure (e.g. `cascade test`).
2. Launch ADAM — review the table list before clicking Apply.
3. **Expected:** Fact, Dimension, Date, and Calculated tables are correctly identified. Compare against the known expected classifications.

### 4.2 Manual override — single table
1. Change one table's classification using the dropdown.
2. Click Apply.
3. Relaunch ADAM.
4. **Expected:** The overridden classification is remembered (stored as a TMDL annotation for `.pbip`, or in the PBIX layout for `.pbix`).

### 4.3 Manual override — persists across PBI restarts
1. Apply a manual override classification.
2. Close PBI Desktop completely and reopen the file.
3. Relaunch ADAM.
4. **Expected:** Classification override still shown.

### 4.4 Classification CLI — list
1. Run: `adam.exe --cli --file "path/to/model.pbip" --list-classifications`
2. **Expected:** Table names and their classifications printed to stdout. Exit code 0.

### 4.5 Classification CLI — set
1. Run: `adam.exe --cli --file "path/to/model.pbip" --set-classification "TableName=Fact"`
2. Reopen the file in PBI Desktop and launch ADAM.
3. **Expected:** Classification shows the overridden value.

---

## 5. Diagram naming and conflicts

### 5.1 First Apply — no existing diagrams
1. Use a fresh `.pbip` with no existing model view diagrams.
2. Apply a layout.
3. **Expected:** Diagram created with default name. No conflict prompt.

### 5.2 Apply again — existing diagram with same name
1. Apply a layout to a file that already has a diagram of the same name from a previous Apply.
2. **Expected:** ADAM prompts for what to do (overwrite / rename / cancel). Whichever is chosen, the outcome is correct.

### 5.3 Per-Fact — multiple diagrams
1. Apply a Per-Fact layout to a model with 3 fact tables.
2. **Expected:** 3 diagrams created, one per fact, named after each fact table.

### 5.4 Special characters in model name
1. Open a `.pbix` whose filename contains spaces, parentheses, or hyphens.
2. Apply a layout.
3. **Expected:** No errors, diagram created correctly.

---

## 6. `.pbix` write cycle

### 6.1 Backup created
1. Apply a layout to a `.pbix` file.
2. Check the folder containing the `.pbix`.
3. **Expected:** A timestamped backup file (e.g. `MyFile_ADAM_backup_2026-06-05_10-30-00.pbix`) exists.

### 6.2 PBI Desktop reopens automatically
1. Apply a layout to a `.pbix` file.
2. **Expected:** PBI Desktop closes, layout is written, PBI Desktop reopens to the same file automatically.

### 6.3 File locked — user hasn't saved
1. Open a `.pbix` and make unsaved changes.
2. Click Apply in ADAM.
3. When ADAM asks to close PBI, click OK.
4. **Expected:** PBI Desktop prompts to save before closing (its own dialog). ADAM waits. If the user saves and closes, Apply continues. If the user cancels, ADAM detects PBI didn't close and shows a clear message.

### 6.4 Manual file picker fallback
1. Force ADAM into a state where it can't detect the file path (e.g. unsaved new file).
2. Click Apply.
3. **Expected:** File picker dialog opens. Selecting the correct `.pbix` allows Apply to proceed successfully.

---

## 7. ADAM single-instance and model switching

### 7.1 Second External Tools launch forwards to existing ADAM
1. With ADAM open and connected to file A, click External Tools → ADAM from file B (a different open file).
2. **Expected:** No second ADAM window opens. The existing ADAM window switches to file B's model. Log shows `Pipe: received new args`.

### 7.2 Switching back
1. After 7.1, click External Tools → ADAM from file A again.
2. **Expected:** ADAM switches back to file A correctly.

---

## 8. Edge cases and error handling

### 8.1 File deleted between detection and Apply
1. ADAM detects the file path correctly.
2. Delete or move the `.pbip` / `.pbix` file before clicking Apply.
3. **Expected:** Helpful error message. No crash.

### 8.2 Very long file path
1. Open a `.pbix` stored in a deeply nested folder (path > 200 characters).
2. Apply a layout.
3. **Expected:** No failure from path length limitations.

### 8.3 File on a network drive
1. Open a `.pbix` stored on a mapped network drive.
2. Apply a layout.
3. **Expected:** Works correctly, or fails with a clear message if the network write fails.

### 8.4 Model with no relationships
1. Open a model that has tables but no relationships defined.
2. Apply each layout style.
3. **Expected:** Tables are laid out without error. No crash from empty relationship list.

### 8.5 Model with only one table
1. Open a model with a single table, no relationships.
2. Apply each layout style.
3. **Expected:** Sensible single-table layout. No crash.

### 8.6 Large model (20+ tables)
1. Open a model with 20 or more tables.
2. Apply a Full Model layout.
3. **Expected:** All tables placed. Performance is acceptable (completes within ~10 seconds).

### 8.7 ADAM launched with PBI Desktop not running
1. Launch `ADAM.exe` directly (no `-server`/`-database` args).
2. **Expected:** Manual connect UI shown. No crash.

---

## 9. Theme and settings persistence

### 9.1 Theme switches and persists
1. Change the theme (Light/Dark) in ADAM.
2. Close and relaunch ADAM.
3. **Expected:** Theme preference is remembered.

### 9.2 Layout style selection persists
1. Select Waterfall layout, close ADAM, relaunch.
2. **Expected:** Waterfall is still selected. *(Note: verify whether this is intended behaviour.)*

---

## 10. CLI mode

### 10.1 Basic CLI Apply
```
adam.exe --cli --file "path/to/model.pbip"
```
**Expected:** Layout written, exit code 0, success message to stdout.

### 10.2 CLI with style flag
```
adam.exe --cli --file "path/to/model.pbip" --style waterfall
adam.exe --cli --file "path/to/model.pbip" --style star-schema
adam.exe --cli --file "path/to/model.pbip" --style inverted-l
```
**Expected:** Correct layout applied for each.

### 10.3 CLI — file not found
```
adam.exe --cli --file "nonexistent.pbip"
```
**Expected:** Clear error message to stderr, non-zero exit code.

### 10.4 CLI — wrong file type
```
adam.exe --cli --file "somefile.txt"
```
**Expected:** Clear error message, non-zero exit code.

---

## Automation notes

Most of ADAM's test surface is UI and integration-level, which makes full automation hard. Realistic options:

**What can be automated today (unit tests already exist in `ADAM.Tests`):**
- Layout algorithm correctness — given a set of tables and relationships, assert node positions meet the layout rules.
- Table classification logic — given table names / column patterns, assert correct classification.
- `SettingsManager` read/write round-trips.
- `ModelReader` parsing of layout JSON files (offline, no PBI needed).

**What could be automated with more work:**
- **CLI tests** — the CLI mode (`--cli --file`) is automatable with a simple shell script or xUnit test that shells out to `adam.exe` and inspects the output `.pbip` file. This is the highest-value automation opportunity since it covers layout writing end-to-end without a UI.
- **WinAppDriver / FlaUI** — WPF UI automation is possible but brittle. Realistic only for smoke tests (does ADAM open? does the Apply button become enabled?).
- **PBI Desktop mock** — the hardest part to automate is that ADAM needs a live PBI Desktop AS port. A lightweight mock AS server (e.g. using the TOM SDK in server mode with a test model) could allow `ConnectWithTokenAsync` to be tested without a real PBI Desktop instance.

**Recommended priority:**
1. Expand the existing unit tests to cover layout rules and classification edge cases.
2. Write CLI integration tests as shell scripts or xUnit process-launch tests — these cover the `.pbip` write path end-to-end.
3. Manual testing for everything involving `.pbix` write, PBI Desktop interaction, and file detection.
