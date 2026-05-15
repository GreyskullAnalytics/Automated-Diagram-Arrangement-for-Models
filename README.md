<img align="right" src="docs/icon.png" width="80" height="80"/>

# ADAM - Automated Diagram Arrangement for Models

> **A free Power BI External Tool by [Greyskull Analytics](https://www.greyskullanalytics.com)**  
> *Data Solutions that make businesses better*

ADAM automatically arranges the tables in your Power BI model view into clean, readable diagrams - eliminating the hours spent manually dragging tables around every time your model changes.

![ADAM Screenshot](docs/screenshot.png)

---

## Why use ADAM?

Anyone who has opened the **Model view** in Power BI Desktop on a complex semantic model knows the pain: tables scattered randomly, relationship lines crossing each other, impossible to read. ADAM fixes this in one click.

- Supports **Waterfall**, **Star Schema**, and **Inverted L** layout styles
- Automatically classifies tables as facts, dimensions, bridges, outriggers, and disconnected
- **Per-fact diagrams** - one focused diagram per fact table for large, complex models
- Minimises crossing relationship lines using barycenter ordering
- Works with both `.pbip` (recommended) and `.pbix` files
- Launched from the **External Tools** ribbon (installer) or directly as a standalone app (portable)
- Dark mode support, system-default theme detection

---

## Requirements

- **Power BI Desktop** (any recent version)
- **Windows 10 / 11** (x64)
- .NET 8 runtime *(included in the self-contained download - no separate install needed)*

---

## Installation

### Option A - Installer (recommended)

1. Download `ADAM-setup-x.x.x.exe` from [Releases](../../releases)
2. Run the installer — it handles everything including External Tools registration
3. Restart Power BI Desktop

> Requires local admin rights. Ask your IT team if needed.

### Option B - Portable (no admin required)

1. Download `ADAM-x.x.x.exe` from [Releases](../../releases)
2. Place it anywhere on your machine (Desktop, a shared folder, etc.)
3. Launch `ADAM.exe` directly

The portable version does **not** appear in the External Tools ribbon. Instead, open your Power BI file first, then launch ADAM and select the file from the dropdown — ADAM will detect all currently open Power BI Desktop instances and let you connect to one.

> **Windows SmartScreen warning**
>
> When running the installer or the portable `.exe` you may see a "Windows protected your PC" message listing the publisher as unknown. This is expected - ADAM is not yet code-signed with a paid certificate. It is safe to proceed:
>
> 1. Click **More info**
> 2. Click **Run anyway**
>
> Greyskull Analytics is working towards a signed release in a future version. In the meantime, the portable `.exe` (Option B above) can be used as an alternative if your organisation does not allow unsigned installers.

---

## How to use

1. Open your `.pbip` or `.pbix` file in Power BI Desktop
2. Launch ADAM:
   - **Installer**: click **ADAM** in the External Tools ribbon — ADAM opens connected to the active file automatically
   - **Portable**: launch `ADAM.exe` directly, then select your open Power BI file from the dropdown and click **Connect**
3. Review the table classifications in the list - override any you disagree with using the dropdown on each row
4. Choose a **Layout Style** and **Mode**
5. Click **Apply [Style] Layout**

### Layout Styles

| Style | Description |
|-------|-------------|
| **Waterfall** | Dimensions at the top, facts at the bottom. Positions are optimised to minimise crossing relationship lines. Best for most models. |
| **Star Schema** | Facts at the centre, dimensions radiating outward in a circle. Mirrors the classic star schema structure. |
| **Inverted L** | Facts in a left column, dimensions in a top row. Works well for models with many fact tables. |

### Modes

| Mode | Description |
|------|-------------|
| **Full Model** | One diagram containing all tables, named `All Tables - [Style]`. |
| **Per Fact Table** | One diagram per fact table, containing only that fact's related dimensions and ancestor tables. Named after the fact table. Ideal for large models. |

### Table Classification

ADAM automatically classifies each table based on its relationships:

| Badge | Classification | How detected |
|-------|---------------|--------------|
| 🟣 FACT | Fact table | Many-side of relationships; relationships span multiple columns |
| 🟡 DIM | Dimension | One-side of relationships; single key column |
| 🔵 BRIDGE | Bridge table | Bidirectional relationship; sits between a fact and a dimension |
| 🔴 OUTRIGGER | Outrigger bridge | Connects a dimension to another dimension |
| 🟢 DISCONNECTED | No relationships | Not connected to any other table |

You can override any classification using the dropdown on each row before applying.

---

## Notes on .pbix vs .pbip

ADAM is designed primarily for the `.pbip` (Power BI Project) format. When used with `.pbix` files, ADAM will warn you that this is an unsupported operation and will create a timestamped backup before making changes.

Microsoft recommends migrating to `.pbip` for all new development. You can convert by using **File → Save As → Power BI Project** in Power BI Desktop.

---

## Thin reports (not supported)

Thin reports — Power BI files whose semantic model is hosted in the Power BI Service rather than stored locally — are **not supported by ADAM**.

ADAM reads model metadata (tables, relationships, column counts) directly from the local Analysis Services instance that Power BI Desktop runs when a file is open. Thin reports do not have a local model for ADAM to read, so automatic table classification is not possible.

If you open a thin report and launch ADAM from the External Tools ribbon, you will see a "Thin Reports Not Supported" message. After dismissing it, ADAM will show the file picker so you can connect to a different open Power BI file instead.

### What to do instead

- Open the **semantic model** (`.pbip` or `.pbix`) directly in Power BI Desktop rather than a thin report that references it.
- If you only have the thin report, consider downloading the semantic model from the Power BI Service and working with it locally.
- If you have other Power BI files already open in Desktop, select one from the dropdown after dismissing the message.

---

## Support ADAM

ADAM is free and always will be. If it saves you time, Greyskull Analytics would really appreciate your support - it helps keep tools like this coming.

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-%23F1497A?style=for-the-badge&logo=buy-me-a-coffee&logoColor=white)](https://buymeacoffee.com/greyskullanalytics)

---

## Licence

ADAM is free to use for personal and commercial purposes. See [LICENSE](LICENSE) for full terms.

---

## About Greyskull Analytics

[Greyskull Analytics](https://www.greyskullanalytics.com) builds data solutions that make businesses better. ADAM is a free tool shared with the Power BI community.

*By the Power of Greyskull!*
