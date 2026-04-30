---
name: originplot
description: Use external Windows Python and the originpro package to automate Origin/OriginPro plotting, control figure hierarchy from runtime to export, map arbitrary data to Worksheet/Matrix/Layer/Plot objects, apply publication-quality templates and Nature-style rules, batch export figures, and save editable OPJU projects. For publication-style figures, follow the embedded palette/style and hierarchy-control rules.
---

# originplot Skill

## 0. Skill Identity

**Skill name:** `originplot`  
**Alias:** `originpro-external-plotting`  
**Chinese name:** `OriginPro 外部 Python 自动绘图 Skill`  
**Primary target:** Anyone who wants to use external Python to control Origin / OriginPro for scientific, engineering, experimental, or publication-quality plotting.  
**Core idea:** Treat OriginPro as a publication-quality rendering engine controlled by Python, not as the main numerical-computation engine.

Typical pipeline:

```text
Numerical computation / experiment
    -> standardized data file
    -> Python data adapter
    -> originpro automation layer
    -> Origin workbook / worksheet / matrix
    -> Origin graph template
    -> exported figure and saved OPJU project
```

---

## 1. When to Use This Skill

Use this skill when the user wants to:

- Use `originpro` from an external Python environment.
- Control Origin / OriginPro programmatically from Python.
- Import CSV, DAT, TXT, Excel, or NumPy data into Origin automatically.
- Batch-generate Origin figures from many data files.
- Apply Origin graph templates automatically.
- Export PNG, TIFF, PDF, EPS, SVG, or other publication figures from Origin.
- Save Origin projects `.opju` automatically.
- Convert a manual Origin plotting workflow into a reusable script.
- Debug external Python + OriginPro automation issues.
- Design a reusable plotting pipeline around OriginPro.
- Produce Nature-style or high-impact-journal scientific figures through OriginPro.
- Design a universal “any figure” control architecture around GraphPage / GraphLayer / Plot.
- Drive plotting from a declarative config such as `FigureSpec`.

---

## 2. Scope and Non-Scope

### This Skill Does

- Design an external Python + OriginPro plotting architecture.
- Provide reusable Python code skeletons for Origin automation.
- Help choose between Worksheet, Matrix, GraphPage, GraphLayer, and template-based plotting.
- Generate line plots, scatter plots, grouped plots, heatmaps, matrix plots, contour plots, and batch plots.
- Use Origin templates as the preferred styling mechanism.
- Require the embedded palette/style rules for Nature-style, paper-ready, or high-impact-journal figures.
- Help export figures and save `.opju` projects.
- Help users keep computation, data preparation, plotting, and export cleanly separated.
- Help debug common `originpro` automation errors.
- Define a universal hierarchy-control model for arbitrary figures.
- Define a declarative `FigureSpec` / `LayerSpec` / `PlotSpec` protocol so agents can generate figures consistently.

### This Skill Does Not

- Use Origin as the main engine for large-scale numerical simulation.
- Replace Fortran / C++ / Python / MATLAB / HPC workflows for heavy computation.
- Encourage WSL/Linux to directly call `originpro` when the official external package requires Windows and local Origin installation.
- Hard-code every visual style into Python when an Origin template is a better abstraction.
- Treat `originpro` as a matplotlib replacement. It is an automation bridge to Origin.

---

## 3. External Facts to Remember

Stable principles:

- `originpro` is a high-level Python API for interacting with Origin software.
- External `originpro` controls Origin through Origin Automation Server / COM.
- It can read, write, and modify Origin data objects, create graphs, and export graphs.
- External `originpro` is intended for Windows Python controlling a local Origin installation.
- Origin should be treated as the rendering and template engine; Python should be the orchestration layer.

Official reference links to include when useful:

```text
https://www.originlab.com/doc/ExternalPython
https://www.originlab.com/doc/ExternalPython/External-Python-Code-Samples
https://www.originlab.com/doc/python
https://www.originlab.com/doc/python/Examples/Graphing
https://www.originlab.com/doc/python/Examples/Origin-Python-Data-Exchange
https://pypi.org/project/originpro/
```

---

## 4. Top-Level Architecture

Design every OriginPro automation workflow using the following layers:

```text
1. Computation / acquisition layer
   Fortran, C++, Python, MATLAB, lab instruments, simulation, experiment

2. File/data layer
   CSV, DAT, TXT, XLSX, NPY, NPZ, HDF5, raw instrument output

3. Python data adapter layer
   pandas, numpy, unit conversion, column naming, reshaping, validation

4. Origin data object layer
   Workbook, Worksheet, Matrixbook, Matrixsheet

5. Origin graph object layer
   GraphPage, GraphLayer, Plot, Axis, Legend, Colormap

6. Template and styling layer
   .otpu, .otp, custom graph themes, journal styles

7. Export and archive layer
   PNG, TIFF, PDF, EPS, SVG, OPJU, logs
```

Architectural principle:

```text
Computation belongs outside Origin.
Data normalization belongs in Python.
Visual style belongs mostly in Origin templates.
Automation and batching belong in Python.
Final figure quality belongs to Origin.
```

---

## 5. Universal Figure Hierarchy Control Model

This section is mandatory for arbitrary-figure control. Any figure should be decomposed into the following control layers:

```text
L0 Runtime / Session
L1 Data
L2 Origin Data Object
L3 GraphPage
L4 GraphLayer
L5 Plot / Primitive
L6 Axis / Scale / Tick
L7 Legend / Label / Annotation
L8 Theme / Template / Style
L9 Export / Archive / QA
```

### L0 Runtime / Session
Controls:
- Whether Origin is launched.
- Visible or hidden mode.
- New project or attach to existing project.
- Exception-safe shutdown.
- Output, template, and log directories.
- Windows Python vs WSL-to-Windows bridge.

### L1 Data
Controls:
- Data source, schema, units, and semantics.
- Missing/invalid values.
- Sorting, grouping, normalization, reshaping, pivoting.
- Semantic roles such as `x`, `y`, `group`, `size`, `error`, `z`.

### L2 Origin Data Object
Controls:
- `Worksheet` for XY, multi-Y, grouped tables, bar/scatter data, raw XYZ.
- `Matrix` for regular-grid heatmaps, contour, and image-like data.
- Worksheet-XYZ when data are irregular or need triangulation-style contouring.

Decision rule:

```text
Regular 2D field -> Matrix preferred
Tabular XY / grouped data -> Worksheet preferred
Irregular XYZ -> Worksheet XYZ
```

### L3 GraphPage
Controls:
- Whole figure canvas.
- Single-panel vs multi-panel layout.
- Page size, margins, inter-panel spacing.
- Shared legend or shared colorbar strategy.

### L4 GraphLayer
Controls:
- Each panel / coordinate system.
- Layer position and size.
- Which plots live inside the layer.
- Layer-specific axis titles, ranges, panel tag, and shared-axis behavior.

### L5 Plot / Primitive
Controls:
- Line, scatter, bar, error bar, heatmap, contour, area, surface, box, violin, bubble.
- Mapping from data fields to visual channels:

```text
x, y, z, color, size, group, error, text
```

### L6 Axis / Scale / Tick
Controls:
- Axis titles and units.
- Linear / log scale.
- Range, padding, ticks, minor ticks, tick label formatting.
- Shared axes.
- Top/right frame visibility.

### L7 Legend / Label / Annotation
Controls:
- Legends.
- Panel labels `(a)`, `(b)`, `(c)`.
- Callouts, arrows, text notes, fit equations, statistical markers.
- Colorbar labels.

### L8 Theme / Template / Style
Controls:
- Global typography and line widths.
- Semantic palette.
- Bar fill, marker shapes, colormap, frame style.
- Journal style such as Nature or PRL.
- Prefer Origin templates for final styling.

### L9 Export / Archive / QA
Controls:
- PNG/PDF/SVG/TIFF/EPS export.
- OPJU save.
- Clean-data archiving.
- Visual QA and file-existence checks.

### Universal Control Order
For any figure, use this order:

```text
Data -> Object -> Layout -> Plot -> Axis -> Annotation -> Style -> Export
```

Or expanded:

```text
1. Define data semantics.
2. Choose the Origin data object.
3. Decide page layout.
4. Decide each layer's role.
5. Map data to plot primitives.
6. Configure axes.
7. Add legend/labels/annotations.
8. Apply theme/template.
9. Export and run QA.
```

---

## 6. Control Planes for Any Figure

When users ask how to control arbitrary figures, answer using these four planes:

### 6.1 Structure Control
Questions answered:
- How many panels?
- What page layout?
- Which Origin objects?
- Single page or multiple pages?

Maps to: `L2 + L3 + L4`

### 6.2 Semantic Control
Questions answered:
- What do columns mean?
- What maps to x/y/color/size/error?
- Which variable is the comparison dimension?

Maps to: `L1 + L5`

### 6.3 Visual Control
Questions answered:
- Fonts, line width, colors, borders, legend style, color maps.

Maps to: `L6 + L7 + L8`

### 6.4 Delivery Control
Questions answered:
- What gets exported?
- Is OPJU saved?
- Is clean data archived?
- Is QA performed?

Maps to: `L9`

---

## 7. FigureSpec Protocol Overview

To make arbitrary-figure generation agent-friendly, use a declarative config protocol.

```text
FigureSpec
    |- RuntimeSpec
    |- DataSpec[]
    |- PageSpec
    |- LayerSpec[]
    |- PlotSpec[]
    |- StyleSpec
    |- ExportSpec
```

Design principle:

```text
FigureSpec describes WHAT to build.
Python/originpro executor decides HOW to build it.
```

See the companion document `FIGURESPEC_PROTOCOL.md` for the full protocol.

---

## 8. Recommended Project Structure

```text
originplot_project/
├── data/
│   ├── raw/
│   ├── processed/
│   └── README.md
├── specs/
│   ├── figure_2x2_nature.yaml
│   └── figure_single_heatmap.yaml
├── templates/
│   ├── line_basic.otpu
│   ├── line_multi_component.otpu
│   ├── scatter_basic.otpu
│   ├── heatmap_diverging.otpu
│   ├── contour_basic.otpu
│   ├── journal_style.otpu
│   └── nature_style.otpu
├── scripts/
│   ├── origin_session.py
│   ├── data_loader.py
│   ├── data_validator.py
│   ├── style_nature.py
│   ├── plot_line.py
│   ├── plot_scatter.py
│   ├── plot_heatmap.py
│   ├── plot_multipanel.py
│   ├── figurespec_executor.py
│   ├── batch_export.py
│   └── config.py
├── output/
│   ├── figures/
│   ├── opju/
│   ├── data/
│   └── logs/
└── README.md
```

---

## 9. Data-Type Decision Tree

### 9.1 XY or Multi-Y Curves
Use `Worksheet`.

### 9.2 XYZ Data
If regular grid, convert to `Matrix`; otherwise use `Worksheet XYZ`.

### 9.3 Matrix Data
Use `Matrixsheet`.

### 9.4 Batch Plotting
Use file loops + one common template + one OPJU archive per run.

---

## 10. Template Strategy

Preferred final workflow:

```text
1. Manually create one ideal graph in Origin.
2. Save it as .otpu.
3. Let Python inject data and export.
```

Keep this separation:

```text
Origin template controls:
    fonts, line widths, colors, legends, axis style, page size
Python controls:
    data, mapping, graph names, variable labels, output paths, batching
```

---

## 11. Mandatory Nature-Style Rules

Use the following semantic palette for Nature-style outputs unless the user explicitly overrides it:

```python
NATURE_PALETTE = {
    "blue_main":      "#0F4D92",
    "blue_secondary": "#3775BA",
    "green_1": "#DDF3DE",
    "green_2": "#AADCA9",
    "green_3": "#8BCF8B",
    "red_1":   "#F6CFCB",
    "red_2":   "#E9A6A1",
    "red_strong": "#B64342",
    "neutral_light": "#CFCECE",
    "neutral_mid":   "#767676",
    "neutral_dark":  "#4D4D4D",
    "neutral_black": "#272727",
    "gold":   "#FFD700",
    "teal":   "#42949E",
    "violet": "#9A4D8E",
    "magenta":"#EA84DD",
}
```

Core rules:
- Prefer Arial or Helvetica.
- Keep panel borders visible.
- Use frameless legends.
- Use semantic colors consistently.
- Use sparse ticks and no grid by default.
- Use data-aware axis limits.
- Save OPJU plus at least one vector export.
- Add `(a)`, `(b)`, `(c)` style panel labels only after layout is finalized.

---

## 12. Installation Guidance

```bash
pip install originpro pandas numpy pyyaml
```

External mode must run in Windows Python.

---

## 13. Standard Session Wrapper

```python
import sys
import originpro as op


def setup_origin(show=True, new_project=True):
    def origin_shutdown_exception_hook(exctype, value, traceback):
        try:
            op.exit()
        finally:
            sys.__excepthook__(exctype, value, traceback)

    if op and getattr(op, "oext", False):
        sys.excepthook = origin_shutdown_exception_hook
        op.set_show(show)

    if new_project:
        op.new()
    return op


def close_origin(save_path=None):
    if save_path:
        op.save(save_path)
    if getattr(op, "oext", False):
        op.exit()
```

---

## 14. Response Pattern for This Skill

When answering any plotting request, structure the response as:

```text
1. Data semantics
2. Origin data object choice
3. GraphPage / GraphLayer structure
4. Plot mappings
5. Axis strategy
6. Annotation strategy
7. Style / template strategy
8. Export / OPJU / QA strategy
9. Minimal working code or FigureSpec
```

---

## 15. Checklist Before Providing Code

Determine:

```text
1. Windows Python or WSL/Linux bridge?
2. Local Origin installed?
3. Data format?
4. Data type: XY, multi-Y, XYZ, matrix, batch?
5. Desired output: PNG, PDF, SVG, TIFF, OPJU?
6. Built-in template or custom .otpu?
7. Single figure or batch?
8. Needs Nature style or not?
9. For arbitrary figures: what are page, layer, plot, and annotation responsibilities?
10. Would FigureSpec be more reusable than hard-coded Python?
```

---

## 16. Final Design Principle

```text
Use Python to automate.
Use OriginPro to render.
Use templates to standardize.
Use FigureSpec to formalize arbitrary figures.
Use OPJU to preserve editability.
```
