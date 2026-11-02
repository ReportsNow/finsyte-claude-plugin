# Reading an open workbook (Office.js)

**Platform.** `Office.context.diagnostics.platform` returns `PC` for Excel for Windows. Finsyte runs only there. Any other value, such as `OfficeOnline` or `Mac`, means the FSN cells hold the values saved in the file (see SKILL.md, "Run the check", step 3).

**Order.** Settings, then structure, then formulas, then FSN values. Read each sheet in blocks of rows that stay under the 5 MB request limit of Excel for the web and the 5,000,000-cell limit of one read, sheets with FSN formulas first.

| Item | Call |
|---|---|
| Settings | `application.calculationMode` (`Automatic`, `AutomaticExceptTables` or `Manual`), `application.iterativeCalculation.enabled`, `workbook.usePrecisionAsDisplayed` |
| Sheets | `worksheet.visibility`, `worksheet.protection.protected` |
| Formula cells, per sheet | `getUsedRange().getSpecialCellsOrNullObject("Formulas")`, then `formulas`, `values`, `valueTypes` and `numberFormat` for each area |
| Array and spilling formulas | `savedAsArray` and `hasSpill` on the formula cells |
| FSN inputs | `formulas`, `values`, `valueTypes` and `numberFormat` of each input cell, and `getSpillParentOrNullObject()` on an input with no formula of its own |
| Spill size | `getSpillingToRangeOrNullObject()` on each cell with a table FSN function. A cell that does not spill returns a null object. |
| Used range against data | `getUsedRange()` and `getUsedRange(true)`, which counts values only |
| Conditional formatting | `worksheet.getRange().conditionalFormats` on the whole sheet: `getCount()`, then each format's `type`, `customOrNullObject.rule.formula` or `cellValueOrNullObject.rule.formula1` and `formula2`, and `getRanges()` for the cells |
| Data validation | `getSpecialCellsOrNullObject("DataValidations")`, then `dataValidation.rule` for each area, for the `NAMES_BROKEN` fix |
| Styles | `workbook.styles`, loading `items/name,items/builtIn`, and counting the styles with `builtIn` false |
| Names | `workbook.names` and each sheet's `names`: `name`, `formula`, `visible`, `type` |
| Shapes | `worksheet.shapes`: the count, and `visible`, `width` and `height` of each shape |
| Tables | `worksheet.tables`: each table's range, which sizes a static drilldown sheet |
| PivotTables | `worksheet.pivotTables`: `refreshOnOpen`, and `getDataSourceString()` to group PivotTables by source |
| Links | Formula and name text (SKILL.md, "Find FSN calls"). Excel for the web also lists them in `workbook.linkedWorkbooks`. |
| Precedents | `getPrecedents()` on input cells, as below |

**Protected sheets.** `getSpecialCellsOrNullObject("Formulas")` can fail on a protected sheet. Read the used range's `formulas` in blocks of rows, and keep the cells whose formula starts with `=`. A sheet that cannot be read goes on the report's "Not checked" line with the reason.

**Precedents.** Read an input's formula first. Call `getPrecedents()` only on an input whose formula references other cells, once per block of inputs with one formula, one call per `context.sync()`. `getPrecedents()` returns every cell upstream, on any sheet, and throws ItemNotFound for a formula without cell references: treat that error as no precedents. Precedents are cells only, so expand every name in a formula yourself.

**Second read.** Cells that show `#GETTING_DATA` get a second read after about a minute. Under Manual calculation, the user presses F9 first.

**Fixes.** Set `numberFormat` and sync before you set `formulas` on a cell whose format changes. Keep the old `formulas` and `numberFormat` of every edited cell for the restore.
