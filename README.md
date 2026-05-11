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
- Launched directly from the **External Tools** ribbon in Power BI Desktop
- Dark mode support, system-default theme detection

---

## Requirements

- **Power BI Desktop** (any recent version)
- **Windows 10 / 11** (x64)
- .NET 8 runtime *(included in the self-contained download - no separate install needed)*

---

## Installation

### Option A - Standalone (no admin required)

1. Download `ADAM-standalone-x.x.x.exe` from [Releases](../../releases)
2. Place it anywhere on your machine
3. Copy `adam.pbitool.json` (included in the release) to:
   ```
   C:\Program Files (x86)\Common Files\Microsoft Shared\Power BI Desktop\External Tools\
   ```
   *(This step requires admin rights - ask your IT team if needed)*
4. Restart Power BI Desktop

### Option B - Installer (admin required, recommended)

1. Download `ADAM-setup-x.x.x.exe` from [Releases](../../releases)
2. Run the installer - it handles everything including External Tools registration
3. Restart Power BI Desktop

ADAM will appear in the **External Tools** ribbon in Power BI Desktop.

> **Windows SmartScreen warning**
>
> When running the installer you may see a "Windows protected your PC" message listing the publisher as unknown. This is expected - ADAM is not yet code-signed with a paid certificate. It is safe to proceed:
>
> 1. Click **More info**
> 2. Click **Run anyway**
>
> Greyskull Analytics is working towards a signed release in a future version. In the meantime, the standalone `.exe` (Option A above) can be used as an alternative if your organisation does not allow unsigned installers.

---

## How to use

1. Open your `.pbip` or `.pbix` file in Power BI Desktop
2. Click **ADAM** in the External Tools ribbon
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

## Thin reports (live-connected to Power BI Service)

ADAM supports thin reports - `.pbip` files that connect to a published dataset in the Power BI Service rather than containing a local model definition.

### What works in all cases

Layout diagrams are written to the local `.pbip` file (`semanticModelDiagramLayout.json`) regardless of workspace capacity. Layouts always apply.

### Automatic table classification

Automatic classification (fact, dimension, bridge, etc.) requires reading the model's relationship metadata via the XMLA endpoint. The behaviour depends on your workspace capacity:

| Workspace type | Auto-classification | How it connects |
|---|---|---|
| **Fabric capacity (F SKU)** | ✅ Full | Automatic via PBI Desktop session (ClaimsToken) |
| **Premium P SKU** | ✅ Full | Automatic via PBI Desktop session (ClaimsToken) |
| **PPU (Premium Per User)** | ⚠️ Requires sign-in | PPU uses a legacy endpoint - see below |
| **Pro** | ⚠️ Manual only | No XMLA access - table names read from existing diagram |

When automatic classification is not available, ADAM shows all tables as **DISCONNECTED**. Use the dropdown on each row to classify them manually before applying a layout.

### PPU workspaces - Microsoft sign-in required

PPU (Premium Per User) workspaces use a legacy XMLA endpoint format (`pbiazure://`) that Power BI Desktop does not expose to External Tools in the same way as Fabric or Premium P capacities. As a result, ADAM cannot reuse the existing PBI Desktop session and instead uses **Microsoft Authentication Library (MSAL)** to authenticate independently.

#### What this means for users

The first time a user launches ADAM against a PPU thin report, their default browser will open to a Microsoft sign-in page. After signing in and accepting the permissions prompt, the token is cached - all future connections are silent with no browser prompt.

The sign-in page will request the following permission on behalf of ADAM:

- **Dataset.Read.All** - read-only access to Power BI datasets (ADAM never writes to the dataset)

ADAM's Azure AD app registration is named **ADAM - MSAL** (Client ID: `019dbda8-33ea-4bd2-9930-e156baa68702`).

#### Admin consent for organisations

Some organisations configure their Microsoft 365 tenant to require administrator approval before users can grant permissions to third-party applications. If users see a "Need admin approval" page rather than an Accept button, a Power BI or Azure AD administrator needs to grant consent once on behalf of the organisation.

**Admin consent URL:**
```
https://login.microsoftonline.com/common/adminconsent?client_id=019dbda8-33ea-4bd2-9930-e156baa68702
```

An administrator visiting this URL and clicking **Accept** will approve ADAM for all users in the tenant. Users will then connect silently without any browser prompt.

> **Note:** Admin consent is only required if your organisation has disabled user consent for third-party apps. Many organisations allow users to consent individually, in which case no admin action is needed.

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
