# Reading a workbook file

**Formats.** openpyxl reads .xlsx, .xlsm, .xltx and .xltm files. For a .xlsb or .xls file, ask the user to save a copy as .xlsx (File › Save As › Excel Workbook) or to run the check in Excel with Finsyte loaded, and report nothing you could not read.

**Cells.** Load the file twice with `read_only=True`: once for formulas, once with `data_only=True` for the values saved at the last save. openpyxl needs about 50 times the file size in memory, and read-only mode keeps that down. A formula cell can hold an `ArrayFormula` object (read its `text` and its `ref`) or a `DataTableFormula` object in place of a string. Remove the `_xll.`, `_xlfn.` and `_xlws.` prefixes before you match function names: the file stores every FSN call as `_xll.FSN.…`.

**Settings.** Read the `calcPr` element of `xl/workbook.xml` from the XML. openpyxl's `wb.calculation` fills in defaults of its own, such as `fullCalcOnLoad` set to true, so its values do not show what the file holds. An absent attribute takes Excel's default: calculation automatic, iteration off, full precision on. True is written `1` or `true`.

| Attribute | Meaning |
|---|---|
| `calcMode="manual"` | `CALCULATION_MANUAL` |
| `calcMode="autoNoTable"` | Automatic except for data tables (`DATA_TABLE`) |
| `iterate="1"` | `ITERATIVE_CALCULATION` |
| `fullPrecision="0"` | `PRECISION_AS_DISPLAYED` |
| `forceFullCalc="1"` | `FORCE_FULL_CALCULATION` |
| `concurrentCalc="0"`, `fullCalcOnLoad="1"` | `FORCE_FULL_CALCULATION`, Info items |

**Parts that openpyxl drops or skips.** Read these from the zip:

| Item | Where |
|---|---|
| Shapes | `xl/drawings/drawingN.xml`, the drawing named in the sheet's relationships: count the anchors. `hidden="1"` on `xdr:cNvPr` marks a hidden shape, and `a:ext cx="0" cy="0"` a zero-size shape. |
| Conditional formatting | The `conditionalFormatting` and `x14:conditionalFormatting` elements of the sheet XML. A read-only worksheet has no `conditional_formatting`, and openpyxl drops the `x14` rules with the warning "Conditional Formatting extension is not supported and will be removed". |
| Unique cell formats | `cellXfs count` in `xl/styles.xml`. Named styles are in `cellStyles`. |
| External links | Every part in `xl/externalLinks`, including links that no formula uses. A formula stores a link as an index, such as `[1]Sheet1!A1`; the index follows the order of `externalReferences` in `xl/workbook.xml`, and the part's relationship holds the path. |
| PivotTables and connections | `refreshOnLoad` in `xl/pivotCache/pivotCacheDefinitionN.xml` and in `xl/connections.xml` |
| Data tables | `DataTableFormula` cells |
| Static drilldown sheets | The table's `ref` in `xl/tables/tableN.xml` |
| File size | Each part's size in the zip |

**Names and sheets.** `wb.defined_names` and each sheet's names: the formula text and `hidden`. Sheet state `hidden` or `veryHidden`. Leave out Excel's own `_xlnm.` names.

**Saved values.**
- A `#VALUE!` cell with a `vm` attribute holds a newer error, such as `#SPILL!` or `#CALC!`, saved as `#VALUE!`. Report the error type it most likely holds, and leave the cell out of `FSN_ERROR_VALUE`.
- FSN cells saved as `#GETTING_DATA` were saved while Finsyte was still loading them: report the values as partial. A file gets no second read.
- `#NAME?` in every FSN cell means the file was saved without Finsyte loaded.

**Drilldown size.** The `ref` of the array formula in a table FSN function's cell, or the table's `ref` on a static drilldown sheet. The hidden name `FinsyteNS.DrilldownTotal` on a drilldown sheet runs from the anchor to the total row.

**Volatile inputs.** A file has no `getPrecedents()`. Follow the references of each input's formula level by level, through cells, names and the array formulas whose `ref` covers a cell.

**Not checked in a file.** A file cannot show the second read of `#GETTING_DATA`, the connection state (`FSN_CONNECTION`), or values newer than the last save. Name these on the report's "Not checked" line.
