# FigureSpec Protocol for originplot

## 0. Purpose

This document defines a declarative configuration protocol for building arbitrary OriginPro figures through the `originplot` skill.

The protocol is designed so an agent or script can translate one config into:

```text
clean data -> Origin objects -> GraphPage/GraphLayer/Plot construction -> export -> OPJU archive
```

The protocol answers two goals:

1. Make arbitrary-figure generation structured and reproducible.
2. Decouple figure intent from Python implementation details.

---

## 1. Core Object Tree

```text
FigureSpec
    |- RuntimeSpec
    |- DataSpec[]
    |- PageSpec
    |- LayerSpec[]
    |- PlotSpec[]
    |- AnnotationSpec[]
    |- StyleSpec
    |- ExportSpec
```

### Design Principle

- `FigureSpec` defines one complete figure.
- `LayerSpec` defines one panel / coordinate system.
- `PlotSpec` defines one plot primitive inside a layer.
- `AnnotationSpec` defines legend, panel tags, notes, arrows, colorbars, and labels.
- `StyleSpec` defines global or overridable style.
- `ExportSpec` defines artifacts and QA expectations.

---

## 2. Minimal Schema

```yaml
figure:
  id: fig1
  title: Optional human-readable name

runtime:
  show_origin: true
  new_project: true
  save_project: true
  windows_python: auto
  project_path: output/opju/fig1.opju
  log_path: output/logs/fig1.log

data:
  - id: ds_main
    source: data/processed/panel_a.csv
    format: csv
    object: worksheet
    roles:
      x: time
      y: response
      group: method
      err_low: lower
      err_high: upper
    preprocess:
      dropna: true
      sort_by: [time, method]
      normalize: null

page:
  id: page_main
  layout: single
  size_mm: [180, 130]
  margins_mm: [10, 10, 8, 8]
  panel_spacing_mm: [6, 6]

layers:
  - id: panel_a
    page: page_main
    position_mode: grid
    grid_cell: [0, 0]
    grid_span: [1, 1]
    title: null
    panel_tag: "(a)"
    data_ref: ds_main
    x:
      title: Time (s)
      scale: linear
      limits: auto
    y:
      title: Response (a.u.)
      scale: linear
      limits: auto
    frame:
      top: true
      right: true
      left: true
      bottom: true

plots:
  - id: plot_a1
    layer: panel_a
    type: line
    data_ref: ds_main
    map:
      x: time
      y: response
      group: method
    group_style:
      mode: color
      palette_keys: [blue_main, green_3, red_strong]
    uncertainty:
      type: band
      low: lower
      high: upper

annotations:
  - id: legend_main
    type: legend
    layer: panel_a
    location: upper_right
    frame: false

style:
  theme: nature
  font_family: Arial
  font_size_pt: 8
  line_width_pt: 2.0
  semantic_palette: nature
  template: null

export:
  dir_figures: output/figures
  dir_data: output/data
  dir_opju: output/opju
  png:
    enabled: true
    width_px: 2400
  pdf:
    enabled: true
  svg:
    enabled: false
  save_clean_data: true
  qa:
    require_opju: true
    require_vector: true
    require_nonempty_outputs: true
```

---

## 3. Spec Components

### 3.1 RuntimeSpec

Purpose: control execution environment.

Fields:

| Field | Type | Meaning |
|---|---|---|
| `show_origin` | bool | Whether Origin GUI is visible |
| `new_project` | bool | Start a fresh project |
| `save_project` | bool | Save OPJU at the end |
| `windows_python` | str | `auto`, path, or bridge mode |
| `project_path` | str | Output OPJU path |
| `log_path` | str | Log file path |
| `template_dir` | str | Optional template search path |

### 3.2 DataSpec

Purpose: define one logical data source.

Required fields:

| Field | Meaning |
|---|---|
| `id` | Internal reference name |
| `source` | File path |
| `format` | csv/dat/txt/xlsx/npy/npz |
| `object` | worksheet / matrix / xyz |
| `roles` | Semantic role mapping |

Optional fields:

- `preprocess.dropna`
- `preprocess.sort_by`
- `preprocess.rename`
- `preprocess.filter`
- `preprocess.normalize`
- `preprocess.pivot`
- `matrix_map.x_range`
- `matrix_map.y_range`

### 3.3 PageSpec

Purpose: define overall figure canvas.

Fields:

| Field | Meaning |
|---|---|
| `layout` | `single`, `grid`, `custom` |
| `size_mm` | Figure width/height |
| `margins_mm` | Outer margins |
| `panel_spacing_mm` | Horizontal / vertical spacing |
| `shared_legend` | Optional global legend |
| `shared_colorbar` | Optional global colorbar |

### 3.4 LayerSpec

Purpose: define one panel / coordinate system.

Fields:

| Field | Meaning |
|---|---|
| `id` | Layer id |
| `page` | Page reference |
| `position_mode` | `grid` or `absolute` |
| `grid_cell` | Row/column in grid |
| `grid_span` | Span in rows/columns |
| `position_abs` | Absolute [left, top, width, height] if custom |
| `title` | Optional panel title |
| `panel_tag` | `(a)` etc. |
| `data_ref` | Default dataset |
| `x` / `y` | Axis specs |
| `frame` | Border visibility |
| `share_with` | Optional shared-axis reference |

Axis spec example:

```yaml
x:
  title: Time (s)
  scale: linear
  limits: auto
  padding_fraction: 0.08
  ticks:
    major_count: 5
    minor_count: 0
```

### 3.5 PlotSpec

Purpose: define one plot primitive.

Required fields:

| Field | Meaning |
|---|---|
| `id` | Plot id |
| `layer` | Layer reference |
| `type` | line/scatter/bar/heatmap/contour/area/... |
| `data_ref` | Dataset reference |
| `map` | Channel mapping |

Common `type` values:

```text
line
scatter
bar
grouped_bar
errorbar
heatmap
contour
bubble
area
box
violin
surface
```

Common channel mappings:

```yaml
map:
  x: time
  y: response
  group: method
  color: condition
  size: magnitude
  error: sem
  z: value
  text: label
```

Optional style fields:

- `style.color`
- `style.line_width_pt`
- `style.symbol`
- `style.symbol_size_pt`
- `style.fill`
- `style.colormap`
- `style.opacity`

Optional structural fields:

- `group_style`
- `uncertainty`
- `baseline`
- `smoothing`
- `sort_points`

### 3.6 AnnotationSpec

Purpose: define non-plot visual elements.

Types:

```text
legend
panel_tag
text
arrow
callout
colorbar
reference_line
reference_band
```

Example:

```yaml
annotations:
  - id: panel_a_tag
    type: panel_tag
    layer: panel_a
    text: "(a)"
    position: upper_left_outside

  - id: ref_y0
    type: reference_line
    layer: panel_a
    orientation: horizontal
    value: 0
    style:
      color: neutral_mid
      line_style: dashed
```

### 3.7 StyleSpec

Purpose: define global visual policy.

Fields:

| Field | Meaning |
|---|---|
| `theme` | `nature`, `prl`, `custom`, `minimal` |
| `template` | Optional Origin template |
| `font_family` | Arial / Helvetica / ... |
| `font_size_pt` | Base size |
| `line_width_pt` | Default line width |
| `semantic_palette` | Palette name |
| `color_order` | Optional explicit order |
| `heatmap_diverging` | Default diverging palette |
| `legend_frame` | bool |
| `show_grid` | bool |

### 3.8 ExportSpec

Purpose: define artifacts and QA.

Fields:

| Field | Meaning |
|---|---|
| `dir_figures` | Figure export dir |
| `dir_data` | Clean data archive dir |
| `dir_opju` | OPJU dir |
| `png.enabled` | Export PNG |
| `png.width_px` | PNG width |
| `pdf.enabled` | Export PDF |
| `svg.enabled` | Export SVG |
| `tiff.enabled` | Export TIFF |
| `save_clean_data` | Archive normalized data |
| `qa.*` | Validation requirements |

---

## 4. Recommended Defaults

### 4.1 Theme Defaults

For `theme: nature`:

```yaml
font_family: Arial
font_size_pt: 8
line_width_pt: 2.0
legend_frame: false
show_grid: false
semantic_palette: nature
```

### 4.2 Layer Defaults

```yaml
x.limits: auto
y.limits: auto
x.padding_fraction: 0.08
y.padding_fraction: 0.08
frame.top: true
frame.right: true
frame.left: true
frame.bottom: true
```

### 4.3 Export Defaults

```yaml
png.enabled: true
png.width_px: 2400
pdf.enabled: true
save_clean_data: true
qa.require_opju: true
qa.require_nonempty_outputs: true
```

---

## 5. Canonical Examples

### 5.1 Single Line Figure

```yaml
figure:
  id: line_demo
runtime:
  show_origin: true
  new_project: true
  save_project: true
  project_path: output/opju/line_demo.opju

data:
  - id: ds_line
    source: data/processed/line.csv
    format: csv
    object: worksheet
    roles:
      x: time
      y: signal
      group: method
    preprocess:
      dropna: true
      sort_by: [time, method]

page:
  id: page_main
  layout: single
  size_mm: [160, 120]
  margins_mm: [10, 10, 8, 8]

layers:
  - id: panel_a
    page: page_main
    position_mode: grid
    grid_cell: [0, 0]
    panel_tag: "(a)"
    data_ref: ds_line
    x: {title: Time (s), scale: linear, limits: auto}
    y: {title: Signal (a.u.), scale: linear, limits: auto}
    frame: {top: true, right: true, left: true, bottom: true}

plots:
  - id: line_main
    layer: panel_a
    type: line
    data_ref: ds_line
    map:
      x: time
      y: signal
      group: method
    group_style:
      mode: color
      palette_keys: [blue_main, green_3, red_strong]

annotations:
  - id: legend_a
    type: legend
    layer: panel_a
    location: upper_right
    frame: false

style:
  theme: nature
  template: line_basic.otpu

export:
  dir_figures: output/figures
  dir_data: output/data
  dir_opju: output/opju
  png: {enabled: true, width_px: 2400}
  pdf: {enabled: true}
  save_clean_data: true
  qa: {require_opju: true, require_vector: true, require_nonempty_outputs: true}
```

### 5.2 2x2 Nature Multi-Panel Figure

```yaml
figure:
  id: nature_2x2
runtime:
  show_origin: true
  new_project: true
  save_project: true
  project_path: output/opju/nature_2x2.opju

data:
  - id: ds_line
    source: data/processed/panel_a_line_trends.csv
    format: csv
    object: worksheet
    roles: {x: time, y: response, group: method, err_low: lower, err_high: upper}
  - id: ds_bar
    source: data/processed/panel_b_grouped_bar_wide.csv
    format: csv
    object: worksheet
    roles: {x: group, y: value, group: method, error: sem}
  - id: ds_scatter
    source: data/processed/panel_c_scatter_correlation.csv
    format: csv
    object: worksheet
    roles: {x: x, y: y, group: group, size: size}
  - id: ds_heatmap
    source: data/processed/panel_d_heatmap_matrix.csv
    format: csv
    object: matrix
    roles: {z: value}
    matrix_map:
      x_range: [-1.0, 1.0]
      y_range: [-1.0, 1.0]

page:
  id: page_main
  layout: grid
  size_mm: [180, 160]
  margins_mm: [10, 10, 8, 8]
  panel_spacing_mm: [6, 6]

layers:
  - id: panel_a
    page: page_main
    position_mode: grid
    grid_cell: [0, 0]
    panel_tag: "(a)"
    data_ref: ds_line
    x: {title: Time (a.u.), scale: linear, limits: auto}
    y: {title: Response (a.u.), scale: linear, limits: auto}
    frame: {top: true, right: true, left: true, bottom: true}
  - id: panel_b
    page: page_main
    position_mode: grid
    grid_cell: [0, 1]
    panel_tag: "(b)"
    data_ref: ds_bar
    x: {title: Group, scale: categorical, limits: auto}
    y: {title: Score, scale: linear, limits: auto}
    frame: {top: true, right: true, left: true, bottom: true}
  - id: panel_c
    page: page_main
    position_mode: grid
    grid_cell: [1, 0]
    panel_tag: "(c)"
    data_ref: ds_scatter
    x: {title: X, scale: linear, limits: auto}
    y: {title: Y, scale: linear, limits: auto}
    frame: {top: true, right: true, left: true, bottom: true}
  - id: panel_d
    page: page_main
    position_mode: grid
    grid_cell: [1, 1]
    panel_tag: "(d)"
    data_ref: ds_heatmap
    x: {title: kx, scale: linear, limits: [-1.0, 1.0]}
    y: {title: ky, scale: linear, limits: [-1.0, 1.0]}
    frame: {top: true, right: true, left: true, bottom: true}

plots:
  - id: p1
    layer: panel_a
    type: line
    data_ref: ds_line
    map: {x: time, y: response, group: method}
    uncertainty: {type: band, low: lower, high: upper}
    group_style: {mode: color, palette_keys: [blue_main, green_3, red_strong]}
  - id: p2
    layer: panel_b
    type: grouped_bar
    data_ref: ds_bar
    map: {x: group, y: value, group: method, error: sem}
    group_style: {mode: fill, palette_keys: [blue_main, green_3, red_strong]}
  - id: p3
    layer: panel_c
    type: scatter
    data_ref: ds_scatter
    map: {x: x, y: y, group: group, size: size}
    group_style: {mode: color, palette_keys: [blue_main, teal, violet]}
  - id: p4
    layer: panel_d
    type: heatmap
    data_ref: ds_heatmap
    map: {z: value}
    style: {colormap: diverging_zero_center}

annotations:
  - {id: lg_a, type: legend, layer: panel_a, location: upper_right, frame: false}
  - {id: lg_b, type: legend, layer: panel_b, location: upper_right, frame: false}
  - {id: cb_d, type: colorbar, layer: panel_d, side: right, title: Signed value}

style:
  theme: nature
  template: nature_multpanel_2x2.otpu

export:
  dir_figures: output/figures
  dir_data: output/data
  dir_opju: output/opju
  png: {enabled: true, width_px: 2600}
  pdf: {enabled: true}
  svg: {enabled: true}
  save_clean_data: true
  qa: {require_opju: true, require_vector: true, require_nonempty_outputs: true}
```

---

## 6. Mapping Rules for Common Plot Types

### line

Required mapping:

```yaml
map:
  x: <field>
  y: <field>
```

Optional:

```yaml
group: <field>
error: <field>
err_low: <field>
err_high: <field>
```

### scatter

Required:

```yaml
x, y
```

Optional:

```yaml
group, size, color, text
```

### grouped_bar

Required:

```yaml
x, y, group
```

Optional:

```yaml
error
```

### heatmap

Required for matrix-backed data:

```yaml
z: value
```

### contour

Either matrix-backed or worksheet-XYZ-backed.

---

## 7. Executor Responsibilities

A compliant `figurespec_executor.py` should do the following:

1. Parse YAML.
2. Start Origin session from `RuntimeSpec`.
3. Load and normalize all `DataSpec` sources.
4. Create Origin worksheets/matrices.
5. Build one `GraphPage`.
6. Build all `GraphLayer`s from layout spec.
7. Create plots in each layer.
8. Apply axis settings.
9. Apply annotations.
10. Apply global and local style.
11. Export outputs.
12. Save OPJU.
13. Run QA.

---

## 8. QA Rules

Before success is reported, verify:

```text
1. OPJU exists.
2. At least one raster export exists.
3. If required, at least one vector export exists.
4. Output files are non-empty.
5. Panel labels exist when specified.
6. No panel is blank.
7. Clean data were archived when requested.
```

---

## 9. Practical Advice

- Prefer `FigureSpec` for reusable pipelines and agent execution.
- Prefer direct Python for tiny one-off plots.
- Keep style in Origin templates when publication quality matters.
- Use `FigureSpec` to describe figure intent, not every low-level Origin quirk.
