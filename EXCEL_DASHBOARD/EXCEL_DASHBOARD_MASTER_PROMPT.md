# EXCEL DASHBOARD MASTER PROMPT
# Copy everything below this line and paste at the start of any new dashboard conversation.
# Replace [TOPIC], [FILE], and [DATA DESCRIPTION] with your specifics.
---

You are building a professional Excel dashboard about **[TOPIC]**.

Source file: `[FILE PATH]`
Data: `[DATA DESCRIPTION — e.g. "48 rows of monthly payroll data, columns: Year, Month, Dept1..."]`
Output: `[OUTPUT FILE PATH]`

---

## MANDATORY ARCHITECTURE — 6 sheets, exact names

| Sheet | Visibility | Purpose |
|---|---|---|
| DASHBOARD | Visible | Dark theme canvas |
| DASHBOARD Light | Visible | Light theme twin |
| Processing | Hidden | All aggregation formulas (SUMIFS/AVERAGEIFS/COUNTIFS) |
| Control | Hidden | Slicer binding: B1=Year, B2=Month |
| Data | Hidden | Flat normalized table, named range `mydata` |
| Resources | Hidden | Reserve for assets |

---

## DESIGN SYSTEM — Never deviate

### Font
- **All cells**: Aptos Narrow
- **Unicode icons**: Arial Unicode MS
- **Hero KPI numbers**: Aptos Narrow 24pt regular
- **Dashboard title**: Aptos Narrow 18pt bold
- **Section labels**: Aptos Narrow 10pt bold
- **Card labels**: Aptos Narrow 9pt bold
- **Delta/sub-labels**: Aptos Narrow 8pt

### Dark Theme Colors
```
Canvas background : #000B27
Card background   : #111829
Section bar       : #1F2F57
Accent 1 (blue)   : #00B0F0
Accent 2 (purple) : #A555F6
Accent 3 (teal)   : #34DEE9
Alert (orange)    : #FF6600
Positive (green)  : #00C74D
Negative (red)    : #FF0077
White text        : #FFFFFF
Muted text        : #A2ADBE
```

### Light Theme Colors
```
Canvas background : #EFF0F3
Card background   : #E8ECF4
Section bar       : #D4DCF0
All text          : #000000
```

---

## CANVAS LAYOUT — Fixed row structure

```
Row 1        : height 2pt   → hairline spacer (ALWAYS)
Rows 2–4     : height 28pt  → Header bar (title + subtitle)
Row 5        : height 6pt   → spacer
Row 6        : height 22pt  → KPI icon row
Row 7        : height 38pt  → KPI hero value row
Row 8        : height 20pt  → KPI label row
Row 9        : height 18pt  → KPI delta row
Rows 10–13   : height 10pt  → spacers
Row 14       : height 20pt  → Section label bar
Rows 15–39   : height 15pt  → PRIMARY chart zone (≈13.2cm)
Row 40       : height 20pt  → Section label bar
Rows 41–60   : height 15pt  → SECONDARY chart zone (≈10.6cm)
Rows 61–62   : height 14pt  → Footer (merged, credit line)
```

### Column layout
- Column A: 1.5pt (left margin hairline)
- Separator columns between card groups: 1.5pt
- Card body columns: 9.8pt each, 6 columns per card
- 5 KPI cards starting at columns: 2, 10, 18, 26, 34

---

## KPI CARD TEMPLATE — 5 cards mandatory

Each card spans columns sc to sc+6, rows 5–11:
- **Column sc** (accent strip): fill = card accent color, rows 5–11
- **Columns sc+1 to sc+6**: fill = #111829 (dark) or #E8ECF4 (light)
- **Row 6, col sc+1**: Unicode icon, Arial Unicode MS 11pt, accent color
- **Row 7, cols sc+1:sc+6 MERGED**: hero value formula, 24pt, white; `number_format` set on cell
- **Row 8, cols sc+1:sc+6 MERGED**: label, 9pt bold, white
- **Row 9, cols sc+1:sc+6 MERGED**: delta formula, 8pt, #00C74D

Card accent colors in order: #00B0F0, #A555F6, #34DEE9, #FF6600, #00C74D

---

## CHART RULES — All 5 mandatory

| Chart | Type | Position | Size |
|---|---|---|---|
| Trend (multi-series line) | LineChart | C15 | 34w × 13h cm |
| Category breakdown (column) | BarChart type="col" | Z15 | 18w × 13h cm |
| Cost split (doughnut) | DoughnutChart | C41 | 14w × 10h cm |
| Distribution (horizontal bar) | BarChart type="bar" | N41 | 16w × 10h cm |
| Comparison (clustered horiz bar) | BarChart type="bar" | AB41 | 17w × 10h cm |

### Chart coding rules
- `chart.title = "Title text"` + `chart.autoTitleDeleted = False` — title INSIDE chart
- Line chart: `series.smooth = True`, `series.marker.symbol = "none"`, line width 22000 EMU
- Doughnut: `holeSize = 65`
- Legend position: `'b'` (bottom) on multi-series charts; `None` on single-series
- Remove major gridlines: `chart.plot_area.valAx.majorGridlines = None`
- **Data references for bar/column/horizontal bar charts**: use `min_row` pointing to FIRST DATA ROW, NOT the header row. Do NOT use `titles_from_data=True` on single-series charts.
- For multi-series employee/comparison charts: use `titles_from_data=True` only when the header row is included in the reference range.

---

## POST-PROCESSING — MANDATORY XML STEP

After `wb.save(OUT)`, apply dark styling to the first 5 chart XML files inside the xlsx (which belong to the dark DASHBOARD sheet). This is required because openpyxl cannot set chart backgrounds or axis text colors natively.

```python
import zipfile, os, re
from lxml import etree

NS_C = "http://schemas.openxmlformats.org/drawingml/2006/chart"
NS_A = "http://schemas.openxmlformats.org/drawingml/2006/main"

def _sub(p, tag):   return etree.SubElement(p, tag)
def _rm(p, tag):
    for e in list(p.findall(tag)): p.remove(e)

def _solid(parent, hex_val):
    sf = _sub(parent, f"{{{NS_A}}}solidFill")
    etree.SubElement(sf, f"{{{NS_A}}}srgbClr", val=hex_val)

def _set_bg(elem, hex_val):
    _rm(elem, f"{{{NS_C}}}spPr")
    spPr = _sub(elem, f"{{{NS_C}}}spPr")
    _solid(spPr, hex_val)
    ln = _sub(spPr, f"{{{NS_A}}}ln")
    _sub(ln, f"{{{NS_A}}}noFill")

def _make_txPr(parent, sz, color, bold=False):
    txPr = _sub(parent, f"{{{NS_C}}}txPr")
    _sub(txPr, f"{{{NS_A}}}bodyPr")
    _sub(txPr, f"{{{NS_A}}}lstStyle")
    p = _sub(txPr, f"{{{NS_A}}}p")
    pPr = _sub(p, f"{{{NS_A}}}pPr")
    defRPr = _sub(pPr, f"{{{NS_A}}}defRPr")
    defRPr.set("sz", str(sz))
    if bold: defRPr.set("b", "1")
    _solid(defRPr, color)
    return txPr

def _add_axis_txPr(ax, sz, color):
    """
    CRITICAL: txPr MUST be inserted BEFORE c:crossAx in the element sequence.
    Appending at the end breaks the CT_CatAx / CT_ValAx OOXML schema order
    and Excel silently ignores the text properties — axis labels stay invisible.
    """
    _rm(ax, f"{{{NS_C}}}txPr")
    txPr = _make_txPr(ax, sz, color)          # appended at end first
    crossAx = ax.find(f"{{{NS_C}}}crossAx")
    if crossAx is not None:
        idx = list(ax).index(crossAx)
        ax.remove(txPr)
        ax.insert(idx, txPr)                  # moved to correct position

def _style_title(title_node, sz, color):
    for tag in [f"{{{NS_A}}}defRPr", f"{{{NS_A}}}rPr"]:
        for el in title_node.findall(f".//{tag}"):
            el.set("sz", str(sz)); el.set("b", "1")
            _rm(el, f"{{{NS_A}}}solidFill")
            _solid(el, color)

def apply_dark_chart_styles(filepath, num_dark=5):
    """
    Opens the saved xlsx as a zip archive and rewrites chart XML files to:
      - Set chart area and plot area background to #111829
      - Set chart title to white 13pt bold
      - Set axis labels (catAx + valAx) to white 10pt — BEFORE crossAx
      - Set legend text to white 12pt
    Only the first num_dark chart files are modified (dark DASHBOARD charts).
    Light DASHBOARD charts get title-only styling (12pt dark navy).
    Always call this AFTER wb.save().
    """
    tmp = filepath + ".tmp"
    with zipfile.ZipFile(filepath, 'r') as zin, \
         zipfile.ZipFile(tmp, 'w', zipfile.ZIP_DEFLATED) as zout:

        for item in zin.infolist():
            data = zin.read(item.filename)
            m = re.match(r'xl/charts/chart(\d+)\.xml', item.filename)

            if m:
                n    = int(m.group(1))
                dark = (n <= num_dark)
                root = etree.fromstring(data)

                # Title styling for ALL charts
                title_node = root.find(f".//{{{NS_C}}}title")
                if title_node is not None:
                    _style_title(title_node,
                                 sz=1300 if dark else 1200,
                                 color="FFFFFF" if dark else "1F2F57")

                if dark:
                    BG = "111829"
                    _set_bg(root, BG)

                    pa = root.find(f".//{{{NS_C}}}plotArea")
                    if pa is not None:
                        _set_bg(pa, BG)

                    leg = root.find(f".//{{{NS_C}}}legend")
                    if leg is not None:
                        _set_bg(leg, BG)
                        _rm(leg, f"{{{NS_C}}}txPr")
                        _make_txPr(leg, sz=1200, color="FFFFFF")

                    for ax_tag in [f"{{{NS_C}}}catAx", f"{{{NS_C}}}valAx"]:
                        for ax in root.findall(f".//{ax_tag}"):
                            _add_axis_txPr(ax, sz=1000, color="FFFFFF")

                data = etree.tostring(root, xml_declaration=True,
                                      encoding="UTF-8", standalone=True)
            zout.writestr(item, data)

    os.replace(tmp, filepath)

# Call immediately after wb.save(OUT):
apply_dark_chart_styles(OUT, num_dark=5)
```

---

## FORMULAS — Processing sheet rules

- **ALWAYS** wrap every formula in `IFERROR(..., 0)` or `IFERROR(..., "-")`
- Use `SUMIFS`, `AVERAGEIFS`, `COUNTIFS` — never hardcode aggregated values
- Year selector: `Control!B1`; Month selector: `Control!B2`
- YoY growth formula: `=IFERROR((SUMIFS(...)current_year - SUMIFS(...)prior_year) / SUMIFS(...)prior_year, 0)`
- Display YoY on KPI card: `=IFERROR(TEXT(Processing!Bxx,"+0.0%;-0.0%"),"-")`
  - **NEVER use CHAR(8212)** — Excel CHAR() only accepts 1–255; CHAR(8212) causes #VALUE!

---

## FINAL CHECKLIST — Verify before delivering

- [ ] Zero formula errors (#REF! #VALUE! #DIV/0! #N/A #NAME?)
- [ ] All 5 KPI cards render with values (no blanks)
- [ ] All 5 charts have visible titles inside the chart area
- [ ] All bar/column/horizontal bar charts show category axis labels (department names, locations, etc.)
- [ ] All multi-series charts show a legend at the bottom
- [ ] Legend text is 12pt and readable on the dark background
- [ ] Axis labels are 10pt white and readable
- [ ] Dark theme: no white chart boxes visible — backgrounds match #111829
- [ ] Light theme: charts have white backgrounds and dark titles
- [ ] `freeze_panes = "A5"` on both dashboard sheets
- [ ] `showGridLines = False` on both dashboard sheets
- [ ] Processing, Control, Data, Resources sheets are hidden
- [ ] `wb.save(OUT)` called BEFORE `apply_dark_chart_styles(OUT)`
- [ ] Post-processing runs without errors and replaces the file
