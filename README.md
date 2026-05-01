# FabFlow — AI-Readable Reference

**What this file is:** a single self-contained reference describing every user-facing feature of the FabFlow add-in for Autodesk Inventor. It is written so you can paste the whole thing into Claude, ChatGPT, or any other chat assistant and then ask questions like *"How do I renumber only my frames?"* or *"Where do I change which drawings get exported as DXF?"* or *"Why did my export skip a part?"*

**How to use it:**
1. Open a new chat with Claude, ChatGPT, Gemini, or similar.
2. Paste this entire file as your first message.
3. Ask your question about how a FabFlow feature works.

The AI has been given no other context — everything it knows about FabFlow comes from this file. If you hit something that seems wrong, check the add-in directly (the behaviour in the product is always the source of truth) and please file a bug report via **FabFlow tab → Settings → General → Report a Problem**.

**Getting the latest version of this file from inside FabFlow:** open **Settings → General → Download AI Help**. That button fetches the freshest copy of this document for the FabFlow version you're running and tells you where it was saved on disk. Use that path next time you paste the reference into your AI of choice.

---

## What FabFlow is

FabFlow is an add-in for Autodesk Inventor that automates the repetitive parts of mechanical design for fabrication shops: numbering parts, exporting deliverables (PDFs / STEP / DXF / Excel BOM), placing drawing views, managing top-down derived layouts, controlling sketch visibility and colours, validating component Comments, deriving sheet metal thickness from a sketch line, and more.

It's installed as an Inventor add-in (a `.addin` manifest and a `.dll`) and surfaces its tools through ribbon tabs, three dockable panes, and a settings dialog. Nothing on the model itself is changed until you invoke a specific tool.

- **This reference describes:** FabFlow **1.0.8.8**
- **Inventor support:** 2025, 2026, 2027 (32-bit / older versions are not supported)
- **Platform:** Windows 10 / 11
- **Website:** https://fabflow.com.au
- **General enquiries:** fabflow@rmacdesign.org (also shown in **Settings → Contact**)
- **Technical support:** support@fabflow.com.au (used by the in-app bug report fallback)

Inside Inventor's Add-Ins Manager the add-in appears as **FabFlow_Addin**.

---

## Where to look in Inventor

FabFlow adds three kinds of UI:

1. **Ribbon buttons** on existing Inventor tabs (3D Model, Sheet Metal) and on a new **FabFlow** tab.
2. **Dockable panes** (Drawing Tools, Layouts, Recent Parts) that you open/close like any other Inventor dock.
3. **A Settings dialog** opened from the **FabFlow tab → Settings** button, available in every environment (no document open, Part, Assembly, Drawing, Presentation).

### Ribbon map

| Environment | Tab | Panel | Controls |
| --- | --- | --- | --- |
| All (incl. ZeroDoc / Part / Assembly / Drawing / Presentation) | **FabFlow** | Settings | **Settings** |
| Part (`.ipt`) | 3D Model | Dims | **Dimension Mode** combo (*Named Only* / *Hide All* / *Show All*), **Show Dims**, **Hide Dims** |
| Part (`.ipt`) | 3D Model | Visibility | **Hide Sketches**, **Show Sketches**, **Format All Sketches** |
| Part (`.ipt`) | 3D Model | Highlight | **Highlight**, **Clear H-Light** |
| Sheet Metal part (`.ipt`) | Sheet Metal | Dims | **Dimension Mode**, **Show Dims**, **Hide Dims** |
| Sheet Metal part (`.ipt`) | Sheet Metal | Visibility | **Hide Sketches**, **Show Sketches**, **Format All Sketches** |
| Sheet Metal part (`.ipt`) | Sheet Metal | Tools | **Thickness From Line** |
| Sheet Metal part (`.ipt`) | Sheet Metal | Highlight | **Highlight**, **Clear H-Light** |
| Drawing (`.idw`/`.dwg`) | — | — | *Drawing Tools dock — see below* |
| Assembly (`.iam`) | — | — | *Recent Parts dock — see below* |

If you don't see the FabFlow tab, the add-in is probably not loaded — check Inventor's **Add-Ins Manager** for **FabFlow_Addin**.

> **Beta features.** Some builds expose an additional **FabFlow → Beta Features** panel containing **Part Labeller** (Part environment) and **Bolt Insertion** (Assembly environment). These are off in customer builds and are only visible if you're running a development/beta build. If you don't see them, they aren't part of your version.

### Docks

| Dock title | Environment | What it's for |
| --- | --- | --- |
| **FabFlow Drawing Tools** | Drawing | Part renumbering, placing views, sheet automation, exporting |
| **FabFlow Layouts** | Part | Managing a top-down derive list for the active part |
| **Recent Parts** | Assembly | Quick-place the parts you've recently saved without re-browsing |

Open a dock via Inventor's **View → User Interface → Dock Windows** menu (the exact wording varies by locale) and tick the one you want. The dock will only show actual content when its target environment is active — for example, the Drawing Tools dock displays *"Open a Drawing document to use this tool"* when no drawing is current.

---

## Feature reference

### 1. Part Numbering

**What it does.** Walks an assembly's Bill of Materials and sets every part's and sub-assembly's **Part Number** iProperty according to a scheme you pick. Saves each file as it goes. Existing part numbers are overwritten for any component the scheme decides to renumber — there is no dry-run preview, so keep a backup or a PDM checkpoint if you're nervous.

**Where to find it.** Open a drawing of your assembly, then use the **FabFlow Drawing Tools** dock. The top group is titled **FABFLOW PART NUMBERING** and contains:

- **Selected assembly** field (read-only) — shows the Part Number of the currently loaded assembly, or `<none>` if nothing is loaded.
- **Load Assembly** — prompts you to click a drawing view that references the assembly you want to renumber. FabFlow then "locks" that drawing to that assembly and writes the lock into a custom iProperty on the drawing so it survives save/close.
- **Renumber** — runs the current scheme against the loaded assembly. A progress dialog appears while it works.
- **Check Comments** — validates the Comments iProperty on every component (covered separately below).

A status line at the bottom of the dock narrates what's happening.

#### Schemes

Choose one in **Settings → Part Numbering → Part numbering mode**:

- **Structured** *(default)* — reflects the assembly hierarchy. The top assembly stays at its existing Part Number unless you change the *Keep top-level fixed* behaviour (see below). Sub-assemblies are numbered **A-02**, **A-03**, **A-04**… in BOM-walk order. Parts inside each sub-assembly get a counter scoped to that assembly: **P-02-01**, **P-02-02** under A-02; **P-03-01**, **P-03-02** under A-03, etc. Nested sub-assemblies use their own A-NN number for their child parts: A-01 → A-02 → A-03 produces **P-03-01** for parts inside A-03. Frame members detected by the matcher are folded under their owning frame assembly; frame assemblies become **A-NN-FRAME** and frame skeletons become **A-NN-SKELETON FRAME**. Inseparable assemblies are numbered as if they were parts. "General" parts (one-offs flagged as PartSet=1 with a positive volume) get a global **GEN-P-001**, **GEN-P-002**… counter outside the per-assembly scoping.
- **Sequential** — flat counting across the whole tree. Top stays at its existing number, sub-assemblies get **A-02**, **A-03**, **A-04**… and every part gets a single running counter **P-001**, **P-002**, **P-003**… regardless of which assembly it sits in. The part counter does **not** reset per assembly.
- **FramesOnly** — only frame members and skeletons are renumbered. Each frame member becomes `{parent}-NN` (e.g. **A-02-01**, **A-02-02**) under its enclosing assembly. Skeletons become `{parent}-SKELETON FRAME`. **Everything else — normal parts, hardware, non-frame sub-assemblies — is left exactly as it was.** Use this when you want FabFlow to number your Frame Generator output but not touch hand-numbered parts.

#### Template tokens (prefix / suffix)

In **Settings → Part Numbering** you can set a **Prefix template** and a **Suffix template** that wrap around the auto-generated number. Click **Edit…** to open the token editor. The editor has:

- A **Scope** dropdown (Part vs Top Level Assembly) — picks whose iProperties the chosen token reads.
- A **Token** dropdown listing every available iProperty across Summary Information, Document Summary Information, and Design Tracking Properties (Part Number, Description, Material, Project, Revision Number, Stock Number, Cost, Mass, Designer, Authority, Title, Subject, Category, etc.), plus special tokens like **Qty**.
- A **Custom property** option for User Defined properties not in the registry.

Tokens use the syntax `<PropertyName>` or `<Top.PropertyName>` for top-level scope (e.g. `<Top.Part Number>`, `<Part.Material>`). Leave the templates blank for no prefix or suffix.

#### Skip Renumber gate

Under **Settings → Part Numbering**:

- **Enable Skip Renumber gate** *(checkbox; on by default)* — when ticked, FabFlow will skip any component whose chosen iProperty equals the **Match value**.
- **Skip property** — pick the property to gate on. Includes the standard iProperties plus a **Custom…** option for User Defined properties.
- **Custom property name** — enabled only when *Custom…* is chosen; enter the User Defined property name to read.
- **Match value** *(default: `TEST`)* — the string the property has to equal (case-insensitive) for the component to be skipped.

The default gate is `Project = TEST`, which is a safe no-op on most projects. Use it to keep standard/library parts out of the renumbering pass — for example, set Project = TEST on a vendor library part and FabFlow won't touch it.

If the **top-level assembly** matches the skip rule, the entire renumber run is aborted with the message *"Renumber skipped (top-level assembly matched skip rule)."* If a non-top component matches, that component and its subtree are skipped.

#### Top-level assembly

By default FabFlow keeps the top assembly's part number fixed (`Keep top-level assembly part number fixed` is on by default; not exposed as a separate UI control in 1.0.8.8 but the default is **don't rewrite the top number**). The initial prefix that FramesOnly uses comes from the top assembly's existing Part Number.

#### Frame detection

Frame members are matched with a multi-step heuristic that compares geometry and properties between candidate parts:

1. **Property match:** Material, Stock Number, the `G_L` (cut length) property, Mass (±0.01), Volume (±0.5), Surface Area (±1.0).
2. **Inertia match:** Principal inertias Ixx / Iyy / Izz must match within ±0.0001.
3. **Centre-of-gravity match:** X/Y/Z within tolerance, allowing for the six symmetry cases (mirroring, flipping along axes).

Members matching all three are treated as the same frame member and given the same Part Number, eliminating duplicate cut-length parts. Documents tagged `FrameMemberDoc` (set by Frame Generator) are recognised first; the geometric match is the fallback. Normal parts won't be picked up as frames.

#### What the renumberer skips

- **Reference BOM components** (Inventor's "reference" BOM structure) — always skipped.
- **Read-only / library / Content Centre / adoption-locked** files — anything Inventor considers not modifiable is skipped, and FabFlow does not recurse into it. Content Centre is recognised by the `ContentCenter` property set so it doesn't spam the log.
- **Components matching the Skip Renumber gate** — skipped along with their subtree.
- **Skeleton documents** — skipped during Comments validation but renumbered with the `-SKELETON FRAME` suffix during Structured / FramesOnly runs.
- **Phantom (FrameDoc) assemblies** — recursed into for their children; the assembly itself is labeled `A-NN-FRAME`.

#### What the renumberer writes

Only the **Part Number** iProperty (Design Tracking Properties) is changed. No other iProperties are modified. The base part number is passed through the prefix/suffix template before being written. **Each modified file is saved to disk as the renumber walk progresses** — if a save fails (file is read-only, locked by PDM, source-controlled), it is logged and the run continues.

#### Sheet ordering (related)

**Settings → Part Numbering → Sheet ordering** affects the **Rename Sheets** automation in the Drawing Tools dock, not the renumberer itself. Two modes are available:

- **Group by Assembly** *(default)* — each sub-assembly's parts are kept together on contiguous sheets.
- **All Assemblies, then Parts** — all assembly sheets come first, then all part sheets.

### 2. Drawing Tools dock — Part Insertion

**What it does.** Bulk-places base views (and flat patterns) of every component of an assembly onto a drawing, one click per part, with a **Next** button to walk through the list. Intended for parts-drawing packages where you need one sheet per piece.

**Where to find it.** The middle section of the **FabFlow Drawing Tools** dock is titled **PART INSERTION**.

#### Two modes (top toggle)

- **Standard Parts** — the full BOM walked from the drawing's first base view ("View 1").
- **General Parts** — only parts whose number matches `GEN-P-XXX` (the general-purpose one-offs created by the Structured numbering scheme).

#### Workflow

1. Pick a mode (the inactive one's label is shown but disabled).
2. Click the load button for that mode:
   - **Load BOM from current drawing (View 1)** *(Standard mode)*.
   - **Update General Parts List (GEN-P-XXX)** *(General mode)*.
   FabFlow scans the view, extracts the referenced assembly, builds a deduplicated sorted list of parts, and shows the first one.
3. The dock shows **Current Part Number** and **Comments** for the active part. (If a part has no Comment, you see *"No comment available"*.)
4. **Insert** places a base view (front orientation, hidden-line style) of that part on the active drawing sheet.
5. **Flat Pattern** does the same but places the unfolded flat pattern of a sheet metal part — FabFlow will generate the flat pattern automatically if one doesn't already exist. If the part isn't sheet metal, the button reports an error and skips.
6. Navigate with **Previous** / **Next**. At either end of the list you'll get a warning rather than wrapping.
7. **Next Sheet** creates a new drawing sheet named `{AssemblyPartNumber} Parts {SheetIndex}` with the **Part Title Block** definition from Settings, then advances to the next part.
8. The status box at the bottom narrates results and errors.

### 3. Drawing Tools dock — Drawing Automation

Three automation buttons sit in the **DRAWING AUTOMATION** group of the Drawing Tools dock.

- **Rename Sheets** — renames every sheet in the drawing based on the referenced assembly / part numbers and reorders them, respecting the **Sheet ordering** setting from Part Numbering.
- **Sort BOMs (Parts Lists)** — sorts every parts list on the current drawing by the Part Number column, then renumbers items.
- **Check Views Coverage** — scans the assembly BOM against the drawing's views and reports which components don't have a view of their own. By default, Check Views Coverage ignores comments listed in **Settings → Comments → No Drawing Required** (typical values: FASTENER, FITTING, HYD FITTING, HYD HOSE, PURCHASE, REFERENCE). The result list is also copied to the clipboard so you can paste it into a notes file.

### 4. Exporting

**What it does.** Turns a drawing-linked assembly into a complete output package: per-part STEP, DXF (for sheet metal), per-item PDFs, a bundled "construction drawings" PDF, and an Excel BOM. Files are routed into category folders according to each component's Comments iProperty, and DXFs are post-processed in place if you've enabled that for the category.

**Where to find it.** In the **FabFlow Drawing Tools** dock, the **EXPORT** group has a single **Export** button. You must have a drawing open and an assembly loaded (via **Load Assembly**, or the export will infer it from the first view of Sheet 1). Clicking **Export** prompts for an output root folder and then runs the whole pipeline, showing progress in the dock's status area.

#### Configuring exports

Open **Settings → Exporter**. The tab is divided into these sections:

##### Global options

Checkboxes for things that run once per export, not per component:

- **Export top-level assembly STEP** — writes a STEP of the top assembly at the output root.
- **Export BOM (Excel)** — writes a global `.xlsx` BOM at the output root.
- **Export construction drawings PDF (all sheets)** — produces a single multi-page PDF of every drawing sheet, "the drawing package".

##### Top-level naming templates

Read-only text fields with **Edit…** buttons that open the token editor. These name the top-level output files. Defaults:

- **STEP file name** — `<Top.Part Number>`
- **BOM file name** — `<Top.Part Number>-Rev<Top.Revision Number> BOM`
- **Drawing PDF name** — `<Top.Part Number>-Rev<Top.Revision Number> Construction Drawings`

##### Categories (master/detail)

Routing rules keyed off a component's **Comments** iProperty. The default category set in 1.0.8.8 is **MACHINE**, **PRESS**, **PROFILE**, and **TUBE LASER** — earlier versions used a different set, so don't be surprised if you see other names from your wizard run.

For each category you configure:

- **Category key** — the Comment value to match (e.g. `PROFILE`). Matching is case-insensitive.
- **Folder name** — subfolder under the output root where that category's files will be written. Usually the same as the key.
- **Actions** — tickboxes for what to export per item:
  - **Drawings (PDF package)**
  - **DXF**, with a nested **DXF post processing** tickbox to run FabFlow's internal post-processor on the DXF (see below)
  - **STEP**
  - **BOM**
- **File naming** — a per-format naming template:
  - **Drawings**, **DXF**, **STEP**, **BOM** (each with **Edit…**, defaulting to `<Part.Part Number>`)

Use **Add** / **Remove** at the top of the category list to manage categories. **Reset exporter defaults** at the bottom of the tab restores the factory category set and naming templates.

##### BOM Columns

A horizontal scroll of column cards defines what appears in the exported Excel BOM. Each card has:

- **Token** — dropdown of available iProperty/parameter tokens (or `(Custom…)` for a custom one). Special tokens include **Qty** (BOM quantity) and **THICKNESS** (the sheet metal thickness parameter).
- **Column name** — the header that appears in Excel.
- **Format** — checkbox to enable a number format (`0`, `0.0`, `0.00`, `0.000`) for numeric values. Nested under it is **Strip unit**, which drops the `mm` / `kg` etc. unit suffix.

Use **+ Column** / **− Column** to add and remove columns. A **COMMENTS** card is always shown at the end and is locked — Comments is appended automatically as the final column so the Excel file can be filtered by category.

The factory default column set is: PART NUMBER, ITEM QTY, STOCK NUMBER, DESCRIPTION, MATERIAL, THICKNESS (formatted to one decimal, unit stripped), LENGTH (G_L parameter, integer, unit stripped), then the locked COMMENTS column.

#### Token format syntax

Naming tokens accept an extended syntax for numeric formatting:

```
<Token|NumberFormat|Multiplier|StripUnit>
```

Pipe-separated trailing fields (all optional, in order):

1. **NumberFormat** — a C# numeric format string such as `0.0`, `0.00`, `0.000`, `G`. Applied to the resolved value if it is numeric.
2. **Multiplier** — a numeric factor applied to the value before formatting (e.g. `25.4` to convert mm to inches).
3. **StripUnit** — `N` to drop the unit suffix from the value (`mm`, `kg`, etc.); anything else (or omitted) leaves the unit in.

Example: `<Part.THICKNESS|0.0|1|N>` means "read the THICKNESS parameter, multiply by 1, format to one decimal, strip unit" — used in the default PRESS DXF naming template to produce `STEEL-3.0 mm-...`.

If a token resolves to a non-numeric value but formatting was requested, FabFlow shows a one-time warning and falls back to the raw text value.

#### Token scope

- `<Top.PropertyName>` — read from the top-level assembly's iProperties only.
- `<Part.PropertyName>` — read from the current per-item document.
- `<PropertyName>` *(no prefix)* — looks up context tokens first (Qty, Process, Category) and then falls back to the document's iProperties; provided for backwards compatibility with older templates.

If a `Part Number` token comes back empty, FabFlow falls back to the file name without extension so the output never lands at "blank.pdf".

#### What gets produced, typically

After a successful export the output folder contains:

- A top-level STEP of the assembly (if **Export top-level assembly STEP** is on).
- A top-level Excel BOM (if **Export BOM (Excel)** is on). The first row's columns match your BOM Columns settings; Comments is always the last column for filtering. The whole-assembly BOM is unfiltered.
- A top-level construction drawings PDF (if **Export construction drawings PDF (all sheets)** is on). Combines every sheet of the active drawing into one multi-page PDF named per the **Drawing PDF name** template.
- One subfolder per category, each containing:
  - Per-item PDFs (if the category has **Drawings (PDF package)** ticked) — one PDF per part with that category, named per the per-category **Drawings** template. Inside each, a multi-sheet PDF for items that span multiple drawing sheets.
  - Per-item DXFs (if **DXF** ticked) — flat patterns for sheet metal parts, named per the per-category **DXF** template.
  - Per-item STEPs (if **STEP** ticked).
  - A category-filtered Excel BOM (if **BOM** ticked) — same column layout as the global BOM but auto-filtered to that category.

#### DXF post-processing

If you tick **DXF post processing** on a category, FabFlow's internal post-processor edits each exported DXF in place. The post-processor:

1. **Trims bend lines** from sheet metal flat patterns. Long IV_BEND_DOWN lines (≥75mm) become two short etch segments at the ends rather than a full-length etch line; short bend lines collapse to a single short etch.
2. **Normalises layers** for downstream CAM tools:
   - `IV_OUTER`, `IV_INNER`, `IV_OUTER_PROFILE` → **CUT**
   - `IV_BEND_DOWN`, `IV_BEND_UP`, `IV_BEND`, `IV_MARK_SURFACE`, `IV_FEATURE_PROFILES` → **ETCH**
   - Anything else → deleted.
3. **Saves as DXF R12** for compatibility with older laser/plasma controllers.

There is no external post-processor or watch-folder integration in 1.0.8.8 — the work is done inside the add-in. A diagnostic log is written to `%TEMP%\RMAC_DxfPostProcess.log` if you need to see what the post-processor did.

#### Limitations

- Export overwrites files at the output path (`AlwaysOverwrite` is on by default in code) — pick a clean output root if you don't want to clobber anything.
- The drawing must reference the assembly you've loaded (or the export has nothing to walk).
- Parts whose Comments value doesn't match any configured category are skipped with a status line like *"No export profile configured for category 'XYZ' (PartName)"*. Run **Check Comments** first to catch these, or add the missing category to the exporter.

### 5. Comments Checker

**What it does.** Audits every component of an assembly and forces each one's **Comments** iProperty to be one of an allowed list. Catches unset/typo Comments that would otherwise end up mis-routed by the exporter or skipped silently.

**Where to find it.** **FabFlow Drawing Tools → FABFLOW PART NUMBERING → Check Comments** (after loading an assembly).

#### How it behaves

Walks the assembly BOM. For each component:

- **Skeleton documents** are skipped.
- **Inseparable assemblies** are validated against the **Part categories** list (treated as parts).
- **Normal assemblies** are validated against the **Assembly categories** list, plus the special action `INSEPERABLE` (note the deliberate spelling).
- **Parts** are validated against the **Part categories** list.

If a component has a Comment not in the allowed list (or no Comment at all), FabFlow opens that document and shows a modal picker titled **Select Part Category** or **Select Assembly Category**. You pick a valid value from the list and click OK to write it (or Cancel to skip the component).

#### Inseparable handling

If you pick `INSEPERABLE` for an assembly, a second picker appears asking for a **Part category**, and the assembly's `BOMStructure` is set to **Inseparable** so it's treated as a single line item from then on.

If you pick a normal assembly category that's listed in the **TreatAsInseparableWhenAssemblyIs** list (default: `HYD HOSE`, `INSEPERABLE`, `PURCHASE`), the same BOMStructure change is applied automatically. This list isn't surfaced as its own UI control in 1.0.8.8 — it lives in the settings JSON.

#### Completion

After processing every row, you see:

> *"Completed.*
> *Updated: {n}*
> *Skipped: {n}"*

(Title: **FabFlow Comment Checker**.) Each modified component is saved to disk.

#### Where to configure the allowed lists

**Settings → Comments** has three grids:

- **Part categories** *(default in 1.0.8.8: FASTENER, FITTING, HAND CUT, HYD FITTING, HYD HOSE, MACHINE, PRESS, PROFILE, PURCHASE, REFERENCE, ROLL, TUBE LASER)*.
- **Assembly categories** *(default: GENERAL ASSEMBLY, SUB-ASSEMBLY, SUB-WELDMENT, WELDMENT)*.
- **No Drawing Required** *(default: FASTENER, FITTING, HYD FITTING, HYD HOSE, PURCHASE, REFERENCE)*. Used by **Check Views Coverage** to ignore items that legitimately don't need their own drawing.

Each grid has **Add** / **Remove** buttons. The two main lists must each have at least one entry. The **Part categories** list is shared between the checker and the exporter — the categories you list here are the same ones that can have export profiles in **Settings → Exporter**.

### 6. Layouts dock (top-down derive helper)

**What it does.** Gives you a curated list of "layout parts" that can be derived into the active part on demand, so you can pull skeleton geometry / workplanes / sketches from several source parts at once without navigating the file system every time.

**Where to find it.** The **FabFlow Layouts** dock, in the Part environment.

#### UI layout (top to bottom)

- **Add…** / **Remove** — manage the list of derive sources.
- **DERIVE PARTS** list — the curated derive-source list. Stored per-project in a file named `{ProjectName} Efficiency Data.json` next to the Inventor `.ipj`, so the same list is shared across every part in that project.
- **FULL DERIVE** — derives every sketch and workplane from the selected source part into the active part.
- **CUSTOM DERIVE** — opens an interactive selection session on the source part: tick the specific Sketch2D / Sketch3D / WorkPlane / WorkAxis / WorkPoint entries you want and FabFlow derives only those.
- **DERIVED IN ACTIVE PART** list — populated from the active part's existing `ReferenceComponents.DerivedPartComponents` collection, refreshed on every document activation. Shows Part Number for each unique derive source.
- **EDIT DERIVE** — opens Inventor's derive-options dialog for the selected entry so you can change what's derived after the fact.

#### Limitations

- Only works in Part documents.
- Derived entities are references — editing the source updates the active part the next time you open it.
- The per-project JSON file is created on the first **Add…** action; if you don't see your old list it's likely because the active project changed.

### 7. Sketch tools (visibility, dimensions, colours, highlight)

FabFlow has several sketch-housekeeping tools that live on the ribbon. The core idea: sketches are noisy once you don't need them any more, but you also want to find a specific sketch quickly when you do need it.

#### Dimension visibility policy

**What it does.** Runs a rule across every sketch in the active part to decide which dimensions are visible.

**Where to find it.** The **Dimension Mode** combo on the **Dims** panel (3D Model and Sheet Metal ribbons), or **Settings → Sketch Tools → Dimension policy**.

**Options:**

- **Named Only** *(default)* — hide every sketch dimension whose Inventor parameter name is auto-generated (`d0`, `d1`, …). Dimensions you've named explicitly stay visible. This is FabFlow's recommended steady-state.
- **Hide All** — hide every sketch dimension, full stop.
- **Show All** — show everything, including auto dimensions.

Changing the combo immediately applies across the active part. The setting also **re-applies automatically** whenever you switch to a different part document — so if you set it to Named Only, every part you open will have that policy applied without clicking anything. The auto-application skips any document where you're currently editing a sketch or feature.

#### Per-sketch override (pinning)

Select a sketch in the browser or graphics area, then click:

- **Show Dims** — shows all dimensions in that sketch and **pins** it. Pinned sketches are always shown in full and are skipped by the global policy sweep, so the pin acts as a per-sketch override for "Named Only / Hide All".
- **Hide Dims** — unpins that sketch and re-applies the current global policy to it.

The pin is stored in the sketch's attribute set inside the part file, so it survives save/close/reopen.

#### Sketch visibility

- **Hide Sketches** / **Show Sketches** — sets the Visible property on every 2D and 3D sketch in the active part. Work features (planes, axes, points) are not affected.
- **Format All Sketches** — re-applies FabFlow's sketch colour palette (see below) to every sketch in the active part using the current mode (Disabled / Palette / Random). Use this after you've edited the palette or changed the mode in Settings.

#### Inactive sketch colouring (palette)

**What it does.** Applies colour overrides to *inactive* sketches — sketches you're not currently editing — so at a glance you can tell projected geometry apart from sketch geometry, construction lines apart from regular lines, and under-constrained / fully-constrained / over-constrained entities apart.

**Where to configure.** **Settings → Sketch Tools → Sketch coloring mode**:

- **No formatting** — disable FabFlow sketch colouring entirely; sketches use Inventor's default colours.
- **Muted palette** — apply the configured palette below. *(This is the recommended steady-state if you want colouring on.)*
- **Random colour per sketch** — gives every sketch its own hue (golden-angle distribution); projected geometry is always tan; construction is a lighter variant of the random hue.

Below the mode you can edit the **Inactive Sketch Colour Palette** swatches. Each swatch has a "Change…" button that opens a colour picker:

- **Projected** *(default tan)*
- **Standard Geometry**
  - **Under** *(default dark blue)* — under-constrained
  - **Full** *(default grey)* — fully-constrained
  - **Over** *(default red)* — over-constrained
- **Construction**
  - **Under** *(default steel blue)*
  - **Full** *(default dark grey)*
  - **Over** *(default dark red)*

Click **Reset defaults** to return to FabFlow's shipped palette.

#### Show Format / Hide Format lifecycle

FabFlow's colour overrides have to coexist with Inventor's native "Show Format" toggle, which Inventor uses to display the constraint-state colours during edit. FabFlow handles this automatically:

- **On enter sketch:** Inventor's "Show Format" is forced ON so you see the live constraint-state colours while you work. FabFlow's overrides are temporarily suppressed.
- **On exit sketch:** FabFlow re-applies its overrides and clears the Show Format toggle so the inactive sketch shows the configured palette.

You don't need to interact with this lifecycle directly. If something gets stuck, Format All Sketches and the Highlight buttons (below) will reset things.

#### Highlight mode

The **Highlight** and **Clear H-Light** buttons on the **Highlight** panel (3D Model and Sheet Metal ribbons) flip a sketch between FabFlow's muted/random colours and Inventor's bright native colours, useful when tracing a specific sketch on a complex part:

- **Highlight** — clears FabFlow's overrides on the selected sketch(es), revealing Inventor's native sketch colours (bright).
- **Clear H-Light** — re-applies FabFlow's configured colours so the sketch goes back to its muted/random state.

If nothing is preselected, FabFlow prompts you to pick a sketch in the canvas or browser first.

### 8. Sheet Metal — Thickness From Line

**What it does.** Sets the sheet metal thickness of the active part from the length of a sketch line you pick. Writes to a user parameter whose name is configurable, and updates the sheet metal flat-pattern thickness to match.

**Where to find it.** Ribbon → Sheet Metal tab → Tools panel → **Thickness From Line**.

#### How to use it

1. Open a sheet metal part. If the active document isn't a sheet metal part, FabFlow shows *"Active part is not a sheet metal part."* and exits.
2. Click **Thickness From Line**.
3. Pick a sketch line in any planar 2D sketch. The prompt is: *"Select a sketch line to drive sheet metal thickness."* Picking anything that isn't a SketchLine in a planar 2D sketch errors out with *"Selection must be a SketchLine"* or *"Selected line must be in a 2D planar sketch"*.
4. FabFlow:
   - Creates a driven dimension between the line's endpoints (named `RMAC_SM_THK_DIM` internally).
   - Writes its value to the configured thickness parameter (default name: **THICKNESS**), creating that parameter and exporting it as a user-visible iProperty if it doesn't exist already.
   - Sets the sheet metal `Thickness` property to reference that parameter, so the flat pattern is regenerated at the new thickness.

You can re-run the tool any time to re-bind to a different line — old internal sketches/parameters from a previous run are cleaned up automatically.

#### Configuring the parameter name

**Settings → Sheet Metal → Thickness property/parameter name** (default: `THICKNESS`). Change this if your shop standard already uses a different thickness parameter convention.

### 9. Recent Parts dock

**What it does.** Remembers the last few parts you've **saved** this session so you can re-place them in an assembly without re-browsing.

**Where to find it.** The **Recent Parts** dock, shown in the Assembly environment.

**UI:** a list titled **RECENT PARTS** with two buttons below it:

- **Place** — inserts the selected part into the active assembly at the cursor (normal interactive place flow; you can keep clicking to drop multiple instances).
- **Place - Grounded** — inserts a single occurrence at the assembly origin and applies three flush constraints (one per origin plane pair: YZ, XZ, XY) to lock the part rigid to the assembly's coordinate system. The component is fully constrained to origin but is **not actually grounded** in Inventor's "G" sense — it's a normal occurrence with three constraints, so it can be unconstrained and moved later.

The list holds up to **5** entries. Tracking is driven by save events: every time you save a part, it gets pushed onto the list. The list is **session-only** — it clears when Inventor restarts.

### 10. Settings dialog

Open from **FabFlow ribbon tab → Settings** (available in every environment, including no-document). The dialog is titled **Settings** and has **Apply**, **OK**, and **Cancel** at the bottom. Closing with unsaved changes prompts *"You have unsaved changes. Save before closing?"*. Each tab has its own **Reset defaults** button where applicable.

Tabs, in the order shown:

1. **General**
   - **Version** — read-only display of the running add-in version.
   - **Settings file** — read-only path to the file you're currently editing (User / Company / Custom).
   - **New** — start a new custom-location settings file.
   - **Open** — open an existing custom-location settings file.
   - **Reset Defaults** — wipe all settings back to factory defaults.
   - **Setup Wizard** — re-run the first-run wizard (see below).
   - **Check for Updates** — manual update check.
   - **Report a Problem** — open the bug report form.
   - **Download AI Help** — fetch the latest version of *this file* (the AI reference) for your installed FabFlow version. Tells you the local path it was saved to so you can paste it into your AI of choice.
   - **Enable debug logging** — checkbox; when on, FabFlow writes verbose log lines to Inventor's iLogic Log panel.

2. **Drawings**
   - **Assembly Title Block** — text box (read-only) plus **Load title blocks…** button that pulls the available Title Block definition names from the active drawing for you to pick from. Default: `RMAC_Title_Block`.
   - **Part Title Block** — same UI; default: `RMAC Parts Title`.
   - **Reset defaults** — restores the two default names above.
   These are the title block definition names used by Part Insertion's **Next Sheet** button and by the Rename Sheets automation.

3. **Part Numbering**
   - **Part numbering mode** dropdown (Structured / Sequential / Frames Only).
   - **Sheet ordering** dropdown (Group by Assembly / All Assemblies, then Parts).
   - **Prefix template** — read-only text + **Edit…** button (token editor).
   - **Suffix template** — same.
   - **Enable Skip Renumber gate** — checkbox.
   - **Skip property** dropdown — gated iProperty (or Custom…).
   - **Custom property name** — text box (enabled only when Custom… is selected).
   - **Match value** — text box (default `TEST`).
   - **Reset defaults**.

4. **Exporter**
   - Global options: **Export top-level assembly STEP**, **Export BOM (Excel)**, **Export construction drawings PDF (all sheets)** checkboxes.
   - Top-level naming: STEP / BOM / Drawing PDF read-only fields with **Edit…** buttons.
   - Categories: master/detail editor (left list with Add/Remove, right detail panel with Category key, Folder name, Actions tickboxes including DXF post processing nested under DXF, and per-format file naming templates).
   - BOM Columns: horizontal cards with Token, Column name, Format (with Strip unit nested under it), plus Add/Remove. Locked Comments column at the end.
   - **Reset exporter defaults**.

5. **Comments**
   - **Part categories** grid + Add/Remove.
   - **Assembly categories** grid + Add/Remove.
   - **No Drawing Required** grid + Add/Remove.

6. **Sheet Metal**
   - **Thickness property/parameter name** text box (default `THICKNESS`).
   - **Reset defaults**.

7. **Sketch Tools**
   - **Dimension policy** dropdown (Named Only / Hide All / Show All).
   - **Sketch coloring mode** dropdown (No formatting / Muted palette / Random colour per sketch).
   - **Inactive Sketch Colour Palette** — Projected swatch, Standard Geometry (Under / Full / Over), Construction (Under / Full / Over). Disabled when *No formatting* is selected.
   - **Reset defaults**.

8. **License** — see *Licensing & trial* below.

9. **Contact** — read-only **Email** (`fabflow@rmacdesign.org`) and **Website** (`https://fabflow.com.au`).

> **Beta builds only:** an additional **Part Labeller** tab is registered when the corresponding feature flag is on. It is not present in customer builds.

### 11. Setup Wizard

**What it does.** A first-run wizard that walks you through the settings most people want to change before their first real run. It opens automatically the first time FabFlow loads, and can be re-run any time from **Settings → General → Setup Wizard**.

**Title:** *FabFlow Setup Wizard*. Navigation buttons: **< Back**, **Next >**, **Cancel**. A sidebar shows the step list; the current step is highlighted.

**Steps in order (6 total):**

1. **Welcome** — intro blurb explaining what the wizard will cover (the four configuration steps you're about to walk through, plus where settings will live).
2. **Export Categories** — define your export category names, their folder names, and which actions (Drawings / DXF / DXF Post Processing / STEP / BOM) apply to each. Validates that at least one non-blank category is defined before letting you continue.
3. **Comment Lists** — populate the **Part Comments** and **Assembly Comments** lists. Both must have at least one entry to advance.
4. **Part Numbering** — pick a numbering mode (Structured / Sequential / Frames Only) and optionally set Prefix/Suffix templates with the same editor as the Settings dialog. Includes an explanation panel that describes each mode with examples.
5. **Settings Location** — choose whether to store settings in the default per-user location (`%APPDATA%\FabFlow Addin\settings.json`) or a custom path (useful for team sharing via a network drive). If custom, browse to the folder; FabFlow will create it if it doesn't exist.
6. **Finish** — summary screen showing the final settings file path. Click **Finish** to save.

You can re-run the wizard at any time; settings aren't reset until you actually click through.

> The wizard does **not** include a BOM-columns step. BOM columns are configured in **Settings → Exporter → BOM Columns**.

### 12. Licensing & trial

**What it does.** Gates the add-in behind a **14-day free trial** and then a paid licence. The licence is machine-bound and validated against FabFlow's commerce backend (Lemon Squeezy).

**Trial (first run).** The first time FabFlow loads on a machine, it starts a 14-day trial automatically — no licence key is required during the trial. The trial start is written to **two** places:

- The licence file at `%PROGRAMDATA%\FabFlow\license.dat` (encrypted with Windows DPAPI).
- The Windows registry under `HKEY_CURRENT_USER\Software\FabFlow\Internal`.

Deleting one doesn't reset the trial — FabFlow cross-checks both so re-installing or wiping AppData won't get you a fresh 14 days.

The License tab shows **Status: Trial — N days remaining** during the trial.

**Activation.** To activate a paid licence:

1. Open **Settings → License**.
2. Click **Manage** → **Activate License Key…** (or, if the trial has expired, the activation dialog opens automatically when Inventor starts).
3. Paste your licence key into the **License Key** field of the dialog (titled **FabFlow - License**).
4. Click **Activate**. FabFlow contacts Lemon Squeezy, records your machine fingerprint, and switches the state to **Activated**. Errors are shown in red text; common ones include "Invalid license key", "License key has been disabled", and a server outage message.

The activation dialog also has **Buy License** (opens the Lemon Squeezy checkout) and **Continue Without Addin** (skips loading FabFlow for this Inventor session).

**Machine binding.** Each licence activation is tied to a hardware fingerprint built from (in order of preference):

1. WMI motherboard serial number (`Win32_BaseBoard.SerialNumber`).
2. WMI processor ID (`Win32_Processor.ProcessorId`).
3. Windows MachineGuid (`HKLM\SOFTWARE\Microsoft\Cryptography\MachineGuid`).
4. Computer name as a last resort.

The inputs are SHA-256 hashed to a 64-char hex string. If a previously-activated licence file is loaded on a different machine, FabFlow shows *"This license was activated on a different machine"* and prompts for re-activation. (Trial / Deactivated / Expired states refresh the fingerprint silently — only Activated state is enforced.)

**Moving to a new machine.** On the old machine: **Settings → License → Manage → Deactivate License**. The deactivation call returns the seat to the server (best-effort; if the call fails, the local state is still marked deactivated). Then on the new machine: **Manage → Activate License Key…** with the same key. Restart Inventor between machines.

**Offline behaviour.** FabFlow caches the last online validation for **5 days**.

- During the 5-day window with no internet: status reads *"Running in offline mode. Internet connection required soon."* and the add-in works normally.
- Beyond 5 days offline: status switches to *"Unable to verify license. Please connect to the internet."* and FabFlow stops working until you reconnect. Reconnecting and re-opening Inventor revalidates automatically.

**Trial expiry.** When the 14 days are up: *"Your 14-day trial has ended. Please activate a license to continue using FabFlow."* The activation dialog opens at next Inventor start.

**Clock rollback protection.** If FabFlow sees that your system clock has gone backwards between runs (UTC now is earlier than the last validation, last online check, or trial start), it shows *"System clock appears to have been set back. Please correct your system time."* and refuses to load. Setting your clock forward to the correct time clears the block.

**License tab fields.**
- **Status** — current state (Trial / Activated / Expired / Deactivated / Needs Activation), colour-coded (green / amber / red).
- **License key** — last 4 characters of your key, the rest masked with asterisks.
- **Machine ID** — first 12 characters of the fingerprint hash.
- **Last validated** — local timestamp of the last successful server check.
- **Subscription** — raw subscription status from the server (e.g. `active`, `cancelled`, `paused`).
- **Manage** button — menu with *Activate License Key…* / *Deactivate License*.
- **Buy License** button — opens the Lemon Squeezy checkout.

**LicenseStatus values you might see in error messages:** `Licensed`, `Trial`, `Expired`, `NeedsActivation`, `Deactivated`.

### 13. Updates

**What it does.** Checks whether a newer version of FabFlow is available and, if so, downloads and runs the installer.

**Where the version comes from.** FabFlow reads `https://raw.githubusercontent.com/Roy-RMAC/ff-updates/main/version.json`. That JSON file holds the latest published version, the download URL for the installer, and release notes.

**Manual vs automatic.**
- **Manual check:** **Settings → General → Check for Updates**. Always runs.
- **Background check:** runs once per Inventor startup, but only if more than **24 hours** have passed since the last check. Errors are swallowed silently so a broken network never disrupts your work.

**Dialog flow when an update is available.**

The download happens first, in the background. Once the installer is on disk (`%LOCALAPPDATA%\FabFlow\Updates\FabFlowSetup.exe`), the **Update Available** dialog appears showing:

- The version that's available.
- The release notes for that version (read-only text box).
- **Install Now** — closes Inventor (after prompting you to close any open documents) and runs the installer in `/passive` mode. Inventor is automatically relaunched once the installer finishes.
- **Not Right Now** — closes the dialog; the next manual check or daily background check will surface the update again.

**Failure messages.**
- Manual check: a status line under the **Check for Updates** button reports the result. *"You're on the latest version."* if nothing newer; an error message if the network call fails.
- Background check: silent on failure.
- Download failure: the **Update Available** dialog is suppressed; the next 24h check will try again.

### 14. Bug reports

**What it does.** Submits a bug report directly from inside the add-in. Goes to FabFlow's issue tracker on GitHub (`Roy-RMAC/ff-support`) automatically; falls back to a clipboard-copy option you can email if the automatic submission fails.

**Where to find it.** **Settings → General → Report a Problem**. The form title is **Report a Problem**.

**Fields:**
- **Title \*** *(required)* — a short summary. The Submit button is disabled until you've typed something here.
- **What happened?** — describe the problem (multiline).
- **Steps to reproduce (optional)** — what you did before it broke (multiline).
- **System information (sent with report)** — auto-populated and read-only. Includes:
  - FabFlow add-in version.
  - OS description (e.g. *"Windows 11 Pro 10.0.26200"*).
  - .NET runtime version.
  - Inventor version and display name (or *"(unable to read version)"* if Inventor's reflection failed).
  No personal data, no licence key, no machine fingerprint is included.

Click **Submit**. On success: *"Thank you! Your report has been submitted successfully."*

If submission fails (network down, rate limited, server outage), FabFlow asks: *"Could not submit the report automatically: {error}.\n\nWould you like to copy the report to your clipboard so you can email it to support@fabflow.com.au manually?"*

- **Yes** copies a pre-formatted email (with `To:`, `Subject:`, and the markdown body) to your clipboard. You then paste it into a new email to **support@fabflow.com.au**.
- If even the clipboard copy fails, you'll be told: *"Could not copy to clipboard either. Please take a screenshot of this form and email it to support@fabflow.com.au."*

The report is never lost.

---

## Settings file locations and scope

FabFlow reads settings from one of three locations, in priority order. If a higher-priority file exists it wins; the others are ignored on load.

1. **Custom** — a path you choose via the wizard's *Settings Location* step or **Settings → General → Open / New**. The active custom path is remembered in `%APPDATA%\FabFlow Addin\active_settings_path.txt`.
2. **Company** — `%PROGRAMDATA%\FabFlow Addin\settings.json`. The default save location for new installs (machine-wide; readable by every Windows user on the machine).
3. **User** — `%APPDATA%\FabFlow Addin\settings.json`. Used as a fallback if Company can't be written (typically when the user lacks ProgramData write access). Also where the legacy "per-user" path the wizard offers points.

The active path is shown read-only at **Settings → General → Settings file**. If a save fails to its primary location, FabFlow silently falls back to User and tells you about the fallback in the save result.

A side file at `%APPDATA%\FabFlow Addin\active_settings_path.txt` holds the pointer to the active custom path. Delete that file if you want to drop back to the default Company / User chain.

---

## Troubleshooting / FAQ

### The FabFlow tab isn't showing up in Inventor.
Open **Inventor → Tools → Add-Ins Manager**, find **FabFlow_Addin**, and make sure it's loaded and set to **Load On Startup**. If it's not listed at all, the `.addin` manifest isn't where Inventor looks — re-run the FabFlow installer. If it's listed but greyed out / failing to load, FabFlow's licence check may have rejected the trial — open the License tab to see the status.

### Inventor says "FabFlow_Addin failed to load."
Most often this is a Visual C++ runtime / .NET 8 prerequisite issue. The FabFlow installer ships those prerequisites but enterprise-managed machines sometimes block them. Re-run the installer with admin rights. If the error persists, send a bug report (it auto-includes your .NET runtime version) so we can confirm.

### I just installed FabFlow and it says my trial has expired.
This can happen if FabFlow is being re-installed on a machine where the trial already ran some time ago. The trial start is remembered in the registry (`HKCU\Software\FabFlow\Internal`) even after you uninstall, specifically to stop trial resets. Contact `fabflow@rmacdesign.org` with your Machine ID (shown on **Settings → License → Machine ID**) — we can issue a short extension or a licence depending on your situation.

### "This license was activated on a different machine."
Your licence is bound to the hardware it was activated on. Two options:
- Go back to the original machine and **Manage → Deactivate License**, then activate on the new machine.
- If the old machine is unavailable (dead hard drive, replaced board), email `fabflow@rmacdesign.org` with your licence key and we'll reset the activation server-side.

### "Unable to verify license. Please connect to the internet."
You've been offline for more than 5 days since the last successful licence validation. Reconnect, restart Inventor, and FabFlow will revalidate silently. If you're definitely online and still see this, check that `api.lemonsqueezy.com` isn't being blocked by a corporate proxy or firewall.

### "System clock appears to have been set back."
FabFlow noticed the system clock is earlier than the last validation timestamp it recorded. Set your clock to correct local time (or correct UTC, if you're running on a server with no network time service) and restart Inventor.

### Why does the installer say "Unknown publisher"?
FabFlow is currently not code-signed. Windows SmartScreen shows an "Unknown publisher" warning on the first launch of an installer from a small publisher. Click **More Info → Run anyway** to proceed. This is safe if you downloaded the installer from https://fabflow.com.au — the installer has no network capabilities beyond the update checker. Code signing is on the roadmap; check https://fabflow.com.au for progress.

### Renumber didn't touch some of my parts.
Check these in order:
1. The **Skip Renumber gate** in Part Numbering — if enabled, parts whose property matches the Match value are skipped. Default is `Project = TEST`, so any part with Project property set to TEST is skipped.
2. **Reference BOM structure** — reference components are always skipped.
3. **Read-only / library / Content Centre / phantom** parts — anything Inventor considers not modifiable is skipped.
4. **FramesOnly** mode — this scheme deliberately skips normal parts and assemblies. Switch to **Structured** or **Sequential** if you want them numbered.
5. **Top-level assembly** — by default the top assembly's number is preserved (`Keep top-level fixed`). Sub-assemblies get A-02 onward.

### "Renumber skipped (top-level assembly matched skip rule)."
Your top assembly's iProperty value matches the Skip Renumber gate. Either change the property on the top assembly, or disable the gate, or change the Match value.

### Why doesn't my export produce STEP / DXF / PDF for a specific part?
Export is driven by the part's **Comments** iProperty. FabFlow picks the matching category in **Settings → Exporter** and uses that category's action tickboxes:
- If the part's Comment doesn't match any configured category → it's skipped with a status like *"No export profile configured for category 'XYZ'"*.
- If the matching category has **STEP** un-ticked → no STEP for that part.
- Run **Check Comments** first to make sure every component has an allowed comment. Unset/typo comments are a very common cause.

### "Comments category is blank (PartName)."
The part's Comments iProperty is empty. Run **Check Comments** to assign a valid category; you'll get a picker per part.

### My DXFs come out with extra layers / wrong line styles.
The internal DXF post-processor only runs if **DXF post processing** is ticked on the relevant category. With it on, FabFlow normalises layers (CUT / ETCH) and trims bend lines for laser/plasma compatibility, and saves as DXF R12. Without it, you get Inventor's raw DXF output with the original IV_* layers. Check `%TEMP%\RMAC_DxfPostProcess.log` if you suspect the post-processor mangled something — it logs each layer mapping.

### Bend lines are coming out as full-length etch lines.
The post-processor trims long IV_BEND_DOWN lines (≥75mm) to short etch segments at each end and short ones to a single segment. Make sure **DXF post processing** is ticked for the category. If you need different lengths, the thresholds aren't user-configurable in 1.0.8.8 — file a bug report.

### Excel BOM is missing a column / has the wrong order.
Configure columns in **Settings → Exporter → BOM Columns**. Use **+ Column** / **− Column** to add or remove. The COMMENTS column is locked to the end and can't be removed — that's deliberate, so the file can be filtered by category.

### Numeric BOM column shows "3.0 mm" instead of "3.0".
Tick **Strip unit** under the column's Format checkbox.

### Numeric BOM column shows raw float like "3.000000".
Tick **Format** for that column and pick a number format (`0`, `0.0`, `0.00`, `0.000`).

### Token in a naming template renders as the literal `<X>` string.
Either the property name doesn't exist on the document, or the template syntax has a typo. Open the **Edit…** dialog for the template and pick the token from the dropdown rather than typing it free-form.

### Construction Drawings PDF is huge / takes forever.
It combines every sheet of the active drawing into one multi-page PDF. If you've got 50+ sheets with heavy raster overlays, that's expected. Disable **Export construction drawings PDF (all sheets)** under **Settings → Exporter → Global options** if you don't need the bundle.

### Check Views Coverage says nothing is wrong but my fasteners have no drawings.
By design. FabFlow skips comments listed in **Settings → Comments → No Drawing Required**. FASTENER, FITTING, HYD FITTING, HYD HOSE, PURCHASE, REFERENCE are typical entries — they don't need their own drawings, so they don't trigger false alarms.

### The FabFlow Drawing Tools dock is empty or says "Open a Drawing document to use this tool."
The dock only activates when a drawing (`.idw` or `.dwg`) is the active document. Open or switch to one of your drawings and the dock will populate.

### The Drawing Tools dock buttons are greyed out.
You haven't loaded an assembly yet. Click **Load Assembly** and pick a view that references your top assembly first. Many of the part-numbering and export operations need that lock to know which BOM to walk.

### "Load BOM from current drawing (View 1)" fails or loads the wrong parts.
The BOM loader specifically reads the *first* drawing view on Sheet 1, extracts its referenced document, and walks that assembly's BOM. If your drawing's View 1 isn't what you expect, either re-arrange the views or use **Load Assembly** to explicitly lock the drawing to the correct assembly.

### Insert / Flat Pattern places the view at the wrong location.
Inventor places the base view at the cursor position. Click on the sheet area where you want the view *before* clicking Insert, or use Inventor's standard view-move handles afterwards.

### Flat Pattern doesn't work — "Part is not sheet metal."
Flat Pattern only places the unfolded development of a sheet metal `.ipt`. Standard parts and assemblies don't have a flat pattern. Use **Insert** instead.

### Part numbers weren't saved after Renumber.
Renumber saves as it goes. If a save fails (file is read-only, PDM-managed, source-controlled and locked), FabFlow logs it and keeps going. Tick **Settings → General → Enable debug logging**, re-run, and look at Inventor's iLogic Log panel (or temporary status messages) for SAVE failures.

### Recent Parts list is empty.
The list is populated by save events — every time you save a part, it gets pushed onto the list. If you haven't saved anything yet this session, the list is empty. The list is also cleared on Inventor restart by design.

### Recent Parts → Place Grounded — the part isn't actually grounded.
"Grounded" here is a UX shortcut: FabFlow places the part at origin and adds three flush constraints (one per origin plane pair) to lock it rigid to the assembly's coordinate system. The part still shows as a normal occurrence, not a grounded one — you can unconstrain and move it. If you want a true Inventor-grounded component, right-click the occurrence and choose Grounded.

### My Layouts dock list is empty when I open a different project.
The list is per-project, stored in `{ProjectName} Efficiency Data.json` next to the `.ipj`. Different project = different list. If the file isn't being created, check that the project workspace folder is writable and that an `.ipj` file actually exists (not all part documents have an active project).

### Dimension policy reverts every time I open a sketch.
Pinned sketches stay full-dim'd; unpinned ones follow the global policy. Click **Show Dims** with the sketch selected to pin it permanently. If the global policy is hiding things you want to keep, switch to **Show All** in the Dim Mode combo.

### Sketch colours look wrong / muted everywhere.
Open **Settings → Sketch Tools → Sketch coloring mode**. Three options:
- **No formatting** — disables FabFlow's overrides; you see Inventor's defaults.
- **Muted palette** — applies the configured palette (the default if you want colouring on).
- **Random colour per sketch** — every sketch gets its own hue.

If you've changed the palette and want to push the new colours out to existing sketches, click **Format All Sketches** on the ribbon.

### Highlight button doesn't work.
Highlight needs a sketch to act on. Either select sketches in the browser/canvas first, or click the button and FabFlow will prompt you to pick one. If preselection isn't working, exit any active sketch edit first — the buttons are blocked while you're inside a sketch.

### "Sheet metal thickness" tool says "Active part is not a sheet metal part."
The tool only runs in sheet metal `.ipt` files. Convert the part via Inventor's **Convert to Sheet Metal** tool before running it.

### Settings dialog says "You have unsaved changes" every time I cancel.
Settings tabs mark themselves changed when you edit any control. If you don't want to save the changes, click **Cancel** and then **No** at the prompt. If you want to keep them, click **Apply** before closing.

### My settings changes don't seem to stick.
Click **Apply** or **OK** in the Settings dialog to save. Closing with **Cancel** discards changes. Also check **Settings → General → Settings file** to confirm which file is being written — if you chose a **Custom location** via the wizard, it's written there and not to AppData.

### Multiple users on the same machine see different settings.
By default settings are written to `%PROGRAMDATA%\FabFlow Addin\settings.json` (machine-wide, all users). If your install is falling back to `%APPDATA%` (per-user) it's because the user account doesn't have ProgramData write permission. Check **Settings → General → Settings file** to see which path you're on, and re-run the installer as admin or grant write access if you need machine-wide.

### Update check never finds anything.
FabFlow checks once per startup and rate-limits to 24 hours. Use **Settings → General → Check for Updates** for a manual check that ignores the rate limit. If the manual check also reports no update, you're on the latest version. If it reports a network error, check that `raw.githubusercontent.com` isn't blocked by a corporate proxy.

### I clicked "Install Now" on the update dialog but Inventor didn't restart.
The installer runs in `/passive` mode and Inventor relaunches via a small batch process. If Inventor didn't come back up, the installer probably needed UAC elevation that the passive mode couldn't provide. Run the installer manually from `%LOCALAPPDATA%\FabFlow\Updates\FabFlowSetup.exe`.

### The Bug Report form says my submission failed.
GitHub Issues API has a small rate limit and the FabFlow PAT is limited to write-only on the `Roy-RMAC/ff-support` repo. If the auto-submit fails (rate-limited, GitHub down, network blocked), accept the clipboard fallback and email `support@fabflow.com.au` directly. Your report won't be lost.

### Inventor crashes / freezes while running FabFlow.
Use **Settings → General → Report a Problem** to send a bug report. Tick **Enable debug logging** first, reproduce the issue, and include any iLogic Log output in the report. If FabFlow itself crashes during load (so you can't open Settings), email `support@fabflow.com.au` with your Inventor version and Windows event log entry for the crash.

### "Edit Derive" doesn't seem to do anything.
EDIT DERIVE opens Inventor's standard Derived Component dialog for the selected entry in the **DERIVED IN ACTIVE PART** list. If nothing happens, check that you've selected an entry in that list (not in the upper DERIVE PARTS list, which is the source list).

### Setup Wizard keeps appearing every time I open Inventor.
The wizard sets a `WizardCompleted` flag in your settings file when you click Finish. If it's reappearing, your settings file isn't being saved — check **Settings → General → Settings file** for write errors, or try **Reset Defaults** to start fresh.

### How do I share my FabFlow settings with my team?
Run the wizard's **Settings Location** step and pick a path on your team network drive. Each team member picks the same path on their machine. Everyone now reads/writes the same `settings.json` and stays in sync. If the network is down, FabFlow falls back to `%APPDATA%` so users aren't blocked.

### Where is the AI reference (this file) kept on disk?
**Settings → General → Download AI Help** downloads the freshest version for your installed FabFlow version and tells you the local path. You can also use any older copy you've already saved — but features and defaults change between releases, so the AI may answer based on stale information. Re-download after every FabFlow update.

---

## Glossary

- **BOM** — Bill of Materials. Inventor's structured breakdown of an assembly's components.
- **Component** — an occurrence of a part or sub-assembly inside an assembly.
- **iProperty** — an Inventor document property. The **Part Number** and **Comments** iProperties are the two FabFlow cares most about.
- **Part Number** — the string FabFlow renumbers. Lives on every part and assembly.
- **Comments** — the iProperty FabFlow reads to route parts into export categories and to validate via Check Comments.
- **Category** — a Comments value that has an export profile attached to it (folder, naming templates, action tickboxes).
- **Frame member** — a part produced by Inventor's Frame Generator (cut-length structural steel, extrusion, etc.). FabFlow has special handling for these in FramesOnly mode and dedupes them by geometry.
- **Skeleton** — a part used as a shared geometry source for top-down design. Detected by FabFlow and treated specially in FramesOnly / Structured modes (`-SKELETON FRAME` suffix).
- **Inseparable** — Inventor's BOM Structure flag for an assembly that should be treated as a single line item. FabFlow can set this automatically via the Comments Checker.
- **Reference** — Inventor's BOM Structure flag for components that exist for context but aren't part of the assembly's actual content. Always skipped by FabFlow.
- **Phantom** — a non-physical assembly used by Frame Generator. FabFlow recurses into it for child parts but treats the assembly itself as `A-NN-FRAME`.
- **Flat pattern** — the unfolded 2D development of a sheet metal part.
- **Title block definition** — the reusable title block template in a drawing. FabFlow needs to know which one to apply when creating new parts-drawing sheets.
- **Derive** — Inventor's mechanism for pulling sketches, workplanes, axes, points, or bodies from one part into another as references. Used heavily in top-down design; FabFlow's Layouts dock wraps this.
- **Token** — a placeholder like `<Top.Part Number>` used in naming templates. Tokens are expanded per-item at export/numbering time.
- **Dimension Pin** — a per-sketch flag set by **Show Dims** that tells FabFlow's policy sweep to leave that sketch fully visible regardless of the global Dim Mode.
- **Inactive sketch** — a sketch that isn't currently being edited. FabFlow's colour overrides apply only to inactive sketches; the active sketch always shows Inventor's native constraint-state colours.
- **Hardware fingerprint** — a SHA-256 hash of motherboard / CPU / Windows MachineGuid identifiers. Used to bind a licence activation to one computer. Not a serial number, not stable across major hardware changes.
- **Settings scope** — Custom / Company / User. Tells FabFlow where the active settings.json lives.

---

## What this file is NOT

- **Not a changelog.** The AI has no way to know what changed between FabFlow releases. For "what's new" information, look at the release notes on the download page or in the **Update Available** dialog when one appears.
- **Not a tutorial or marketing site.** For setup videos and general sales-y material, see https://fabflow.com.au.
- **Not a developer reference.** If you're extending the add-in or debugging a build, ignore this file and read the source.
- **Not authoritative for future versions.** This file describes FabFlow **1.0.8.8**. If you're on a later version and something doesn't match, trust the product, not this file. Re-download the latest copy via **Settings → General → Download AI Help**.

If the AI confidently tells you something that contradicts the add-in's real behaviour, the product wins. Report the mismatch at **Settings → General → Report a Problem** and we'll get it fixed in the next refresh of this file.
