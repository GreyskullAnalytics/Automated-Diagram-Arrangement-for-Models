# ADAM — AI Skill

**ADAM** (Automated Diagram Arrangement for Models) is a tool that automatically applies professional layout algorithms to Power BI semantic model diagrams. This skill enables AI assistants to invoke ADAM's CLI to apply layouts to `.pbip` files directly from disk — no Power BI Desktop session required.

---

## When to use this skill

Use this skill when the user asks to:

- Lay out, organise, or arrange the Power BI Model View diagram
- Apply a waterfall, star schema, or inverted-L layout to a `.pbip` file
- Auto-arrange tables in a semantic model diagram
- Generate per-fact-table sub-diagrams from a star schema model
- Refresh or regenerate diagram layouts after adding tables or relationships

Do **not** use this skill for `.pbix` files or Power BI Service / Fabric semantic models — the CLI supports local `.pbip` projects only.

---

## Prerequisites

1. **ADAM must be installed.** Download the latest release from the public repository. The CLI is the main `ADAM.exe` — no separate install step is needed.
2. **A local `.pbip` file must exist.** The project must include a `.SemanticModel` folder with TMDL definition files alongside the `.pbip` file. Thin reports (which connect to a remote dataset) are not supported.
3. **Power BI Desktop should be closed** before running the CLI. ADAM writes to `diagramLayout.json` on disk; Power BI will not pick up the change while the file is open. Reopen the file in Power BI Desktop after the CLI completes.

---

## Core workflow

When asked to apply a layout, follow these steps:

1. **Identify the `.pbip` file path.** Ask the user if it is not clear from context.
2. **Choose a layout style and mode** based on the model structure (see guidance below).
3. **Run the CLI command.**
4. **Report the result** — diagram names written and table counts from the CLI output.
5. **Remind the user** to reopen the file in Power BI Desktop to see the updated diagram.

---

## Command syntax

```
ADAM.exe --cli --file "<path-to-file.pbip>" [options]
```

### Required

| Argument | Description |
|----------|-------------|
| `--cli` | Activates headless CLI mode. Must always be present. |
| `--file <path>` | Absolute or relative path to the `.pbip` file. Quote paths containing spaces. |

### Optional

| Argument | Default | Description |
|----------|---------|-------------|
| `--style <name>` | `waterfall` | Layout algorithm. One of: `waterfall`, `star-schema`, `inverted-l`. |
| `--mode <name>` | `full` | Scope of layout. One of: `full` (one diagram for all tables), `per-fact` (one diagram per fact table). |
| `--diagram <name>` | Auto-generated | Custom name for the diagram (full-model mode only). |
| `--overwrite` | *(not set)* | Overwrite an existing diagram with the same name. Without this flag, a numeric suffix is added instead (e.g. `"All Tables - Waterfall 2"`). |
| `--list-classifications` | *(not set)* | Report all table classifications without applying a layout. Writes annotations to TMDL as a side effect. Cannot be combined with `--style`. |
| `--set-classification "Name=Type"` | *(not set)* | Override the classification for a named table. Can be repeated for multiple tables. Valid types: `Fact`, `Dimension`, `Bridge`, `OutriggerBridge`, `Disconnected`. Can be combined with `--style` to override and then apply a layout in one step. |

---

## Layout styles

### `waterfall` *(default — use when unsure)*

Arranges tables in descending tiers:
- Fact tables occupy the top tier.
- Dimensions connected to facts sit in the tier below.
- Outrigger/bridge tables cascade further down from their parent dimensions.

Best for: models where you want to see the flow of grain from facts down through lookup chains. Works well for any model structure.

### `star-schema`

Places fact tables in the centre and radiates dimensions around them in a ring.

Best for: classic star schema models where the primary concern is showing which dimensions surround each fact. Models with multiple fact tables or complex bridge structures can become crowded — prefer `per-fact` mode in that case.

### `inverted-l`

Fact tables sit on the left side of the canvas; dimensions extend to the right and downward in an L-shaped arrangement.

Best for: wide models with many dimensions where the waterfall becomes too tall, or when a horizontal-primary layout is preferred for screen real estate reasons.

---

## Layout modes

### `full` — Full model diagram *(default)*

Generates a single diagram containing all tables in the model. Suitable for an overview of the entire schema.

The diagram is named `"All Tables - {Style}"` by default (e.g. `"All Tables - Waterfall"`). Use `--diagram` to override.

### `per-fact` — Per-fact-table diagrams

Generates one diagram per fact table. Each diagram contains:
- The fact table itself
- All dimensions and bridge tables with a direct relationship to that fact (1-hop)
- Outrigger/bridge tables connected to those dimensions (2-hop)

Sibling fact tables are excluded from each other's diagrams — shared dimensions appear independently in each.

Diagram names match the fact table name (e.g. `"Sales"`, `"Inventory"`).

Best for: models with multiple fact tables where a single all-tables diagram is too cluttered.

---

## Table classification

ADAM automatically classifies each table from the relationship structure and table name:

| Classification | How detected |
|----------------|-------------|
| **Fact** | Table name starts with `Fact` or `Fct`; or has multiple foreign-key (many-side) relationships; or has exactly one single-direction relationship on the many side; or is bidirectional with multiple key columns |
| **Dimension** | Primarily on the one (primary-key) side of relationships; single foreign key if also on the many side |
| **Bridge** | Bidirectional filter, single key column, connects a fact to a dimension |
| **Outrigger Bridge** | Bidirectional, single key, connects a dimension to another dimension |
| **Disconnected** | No relationships (typically a parameters or slicer table) |

After each run, ADAM writes an `ADAM_Classification` annotation to each table's TMDL file. On subsequent runs these stored annotations take precedence over auto-detection, so manual overrides persist. Use `--list-classifications` to inspect the current state and `--set-classification` to override individual tables from the CLI.

---

## Examples

### Apply a waterfall full-model layout

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip"
```

Expected output:
```
Reading model from SalesModel.SemanticModel...
  12 table(s), 14 relationship(s)
Classifying tables...
  3 fact(s), 7 dimension(s)
Reading existing node sizes...
Calculating Waterfall layout (FullModel)...
Writing layout...
Done. 1 diagram(s) written:
  "All Tables - Waterfall"  (12 tables)
```

---

### Apply a star-schema layout, per fact table

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --style star-schema --mode per-fact
```

Expected output:
```
Reading model from SalesModel.SemanticModel...
  12 table(s), 14 relationship(s)
Classifying tables...
  3 fact(s), 7 dimension(s)
Reading existing node sizes...
Calculating Star Schema layout (PerFact)...
Writing layout...
Done. 3 diagram(s) written:
  "Sales"  (8 tables)
  "Inventory"  (6 tables)
  "Budget"  (5 tables)
```

---

### Add a second layout without overwriting the first

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --style star-schema --mode full
```

If `"All Tables - Star Schema"` already exists in the file, ADAM writes it as `"All Tables - Star Schema 2"` and notes this in stdout.

To replace the existing diagram instead:

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --style star-schema --mode full --overwrite
```

---

### Use a custom diagram name

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --style waterfall --mode full --diagram "Overview Layout"
```

---

### List current table classifications

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --list-classifications
```

Expected output:
```
Reading model from SalesModel.SemanticModel...
  12 table(s), 14 relationship(s)
Classifying tables...

  Table                    Classification
  ───────────────────────  ────────────────
  Sales                    Fact
  Inventory                Fact
  Budget                   Fact
  Customer                 Dimension
  Product                  Dimension
  Date                     Dimension (stored)
  ...

3 facts, 7 dimensions, 0 bridges, 0 outriggers, 2 disconnecteds
Annotations written.
```

`(stored)` indicates classifications that came from a previously saved `ADAM_Classification` annotation rather than auto-detection.

---

### Override a table classification

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --set-classification "Budget=Fact"
```

Multiple overrides:

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --set-classification "Budget=Fact" --set-classification "FxRates=Bridge"
```

Override and immediately apply a layout:

```
ADAM.exe --cli --file "C:\Projects\SalesModel\SalesModel.pbip" --set-classification "Budget=Fact" --style waterfall --mode per-fact
```

---

## Choosing a layout

Use this decision guide when the user has not specified a preference:

| Scenario | Recommended style | Recommended mode |
|----------|------------------|-----------------|
| First time laying out any model | `waterfall` | `full` |
| Classic star schema, single fact | `star-schema` | `full` |
| Multiple fact tables, complex model | `waterfall` or `star-schema` | `per-fact` |
| Model is wide with many dimensions | `inverted-l` | `full` |
| User wants one diagram per subject area | any | `per-fact` |
| User wants a single overview diagram | any | `full` |

When in doubt, `waterfall` + `full` is the safe default. The user can always rerun with a different style — each run adds a new diagram or overwrites the chosen one; it does not remove other existing diagrams.

---

## Interpreting CLI output

| Output | Meaning |
|--------|---------|
| `Done. N diagram(s) written:` followed by a list | Success. Report the diagram names and table counts to the user. |
| `"X" already exists — writing as "X 2"` | A naming conflict was resolved automatically. Let the user know. |
| `Overwriting existing diagram "X"` | `--overwrite` was used and an existing diagram was replaced. |
| Anything on stderr | An error occurred. Report it to the user and suggest checking the file path and model structure. |

**Exit codes:** `0` = success, `1` = error. Check the exit code when running programmatically.

---

## Limitations

- **Local `.pbip` only.** The CLI reads TMDL files from disk. It does not connect to Power BI Desktop, Power BI Service, or Fabric workspaces.
- **No `.pbix` support.** Use the ADAM desktop application for `.pbix` files.
- **No classification overrides.** Classification is algorithm-only in CLI mode. To manually reclassify tables, use the desktop application and then rerun the CLI.
- **PBI Desktop must be closed.** The CLI writes directly to `diagramLayout.json`. Power BI Desktop does not hot-reload this file while it is open; close and reopen the file to see the result.
- **TMDL format only.** Models saved in legacy `.bim` format are not supported. Ensure the semantic model uses the TMDL definition format (the default for new `.pbip` projects in Power BI Desktop since mid-2023).

---

## Troubleshooting

| Error | Likely cause | Fix |
|-------|-------------|-----|
| `File not found` | Wrong path or file doesn't exist | Check the path; quote paths with spaces |
| `Semantic model folder not found` | Thin report, or model not in the expected sibling folder | Ensure the `.SemanticModel` folder exists next to the `.pbip` file |
| `No tables found` | TMDL `definition/tables/` folder is missing or empty | Confirm the model has been saved locally in TMDL format |
| `Per-fact mode requires at least one fact table` | No tables classified as Fact | Use full-model mode, or reclassify tables in the ADAM desktop app |
| Layout written but not visible in Power BI | PBI Desktop was open when CLI ran | Close and reopen the `.pbip` file in Power BI Desktop |
