# Excel mechanics

Read this before you touch the workbook: the checks before you start, how to read NetSuite's lists and chart of accounts through Finsyte formulas, the Office.js calls that write and format the book, and how to wait for the values.

## Contents

- Before you start
- Discovery
- Writing the book
- Formatting calls
- Waiting for values

---

## Before you start

Run these in order. The first three only read; nothing is written before all of them pass.

| Check | How | When it fails |
|---|---|---|
| Excel for Windows | `Office.context.diagnostics.platform` is `PC` | Stop. Finsyte runs only in Excel for Windows, and the book is built from values Finsyte returns. Tell the user to open Excel for Windows with Finsyte and ask again. |
| A workbook for the book | The active workbook has no sheet named Inputs, Cover, Index, Lists or a number, and none of the names in `references/front-pages.md`, "Workbook names" | Ask the user to open a new blank workbook (File › New › Blank workbook) and ask again there. Names and sheets with these names would clash with the book's formulas, and a write would overwrite their sheet. |
| Calculation mode | `application.calculationMode` | Record it. Under `Manual`, tell the user that the values appear only after F9, and ask them to press F9 when you wait for values. Don't change the setting: it applies to every open workbook. |
| Dynamic arrays | `Office.context.requirements.isSetSupported("ExcelApi","1.12")`; then, once Lists exists, a test spill `=SEQUENCE(2)` in a discovery cell returns two rows | Stop. The book uses spills, `FILTER`, `TEXTJOIN`, `TEXTSPLIT` and `LET`, which need Microsoft 365. |
| Finsyte loaded and running | `=FSN.Range.Subsidiaries()` in Lists A2, read after it settles | `#NAME?`: Finsyte isn't loaded. Empty text: Finsyte is signed out, or signed in with its lists not loaded; ask what the Finsyte tab's Connectivity button reads. `Login`: the user signs in. `Switch Company`: the user clicks Refresh Data › Entire Workbook. A paused Finsyte still returns these lists, and its value cells read 0 (SKILL.md, "Verify", step 1). While Finsyte runs, its button in the Refresh group reads Pause Connection, so send the user to Resume Connection only after the user says the button reads Resume Connection. Stop until it returns subsidiaries. |

Tell the user, before the first write, that you add a hidden Lists sheet to read their NetSuite lists, and that everything you add stays in this workbook. Then add Lists and run the last two checks there.

Every discovery formula reads its codes, list names and properties from heading cells in the discovery area, never typed inside the Finsyte call.

---

## Discovery

Read NetSuite through Finsyte formulas that you write on the Lists sheet, wait for them to settle ("Waiting for values"), then read their values. Never guess an account, subsidiary, fiscal year or member name: take each one from a value Finsyte returned.

### Before round 1

Write the Lists columns A to F (`references/front-pages.md`, "Lists"). From them:

- **Subsidiaries.** A OneWorld account lists its subsidiaries, and a parent with children also as `<name> (Consolidated)`. One entry and no `(Consolidated)` entries: a single company, so no consolidating pages.
- **Fiscal years** in `ListFiscalYears`, latest first.
- **Default period.** Put the last day of last month (from today's date, as a typed date, not `TODAY()`) in a scratch cell, and `=FSN.DateToPeriod(<cell>,<"Year" in a heading cell>,<subsidiary>)` beside it for the fiscal year. Its period number is the `n` whose start, `=FSN.PeriodToDate(<fiscal year>,<n>,"Start")`, is on or before that date and whose end, `=FSN.PeriodToDate(<fiscal year>,<n>)`, is on or after it. Period ends on a 4-4-5 or 13-period calendar don't fall on month-ends, so match the range, not the end date.

### After round 1

With the subsidiary, fiscal year and period on Inputs (C5, C11, C14), write the rest.

**Periods per year.** For `n` from 1 to 14 in the discovery area: `=FSN.GetPeriodName(<n>,CurFY,Subsidiary)`, `=FSN.PeriodToDate(CurFY,<n>,"Start")` and `=FSN.PeriodToDate(CurFY,<n>)`. A period is regular when it has a name and dates and doesn't start on the day the previous period ends. A period named `Adjust …`, or one whose start and end are the previous period's last day, is an adjustment period. The count of regular periods, usually 12, goes in Inputs C34 (`PeriodsPerYear`). When an adjustment period exists, compare `=-FSN.GLAccountBalance(Subsidiary,CurFY,<last regular period>,<"YTD">,<"FSN_L4_NIN">)` with the same call for the adjustment period's number, the period numbers typed and the range and code in heading cells; if they differ, the company posts year-end entries there (interview round 1).

**Subsidiary tree.** For each entry of `ListSubsidiaryMembers` (Lists B): `=FSN.GetListInfo("Subsidiaries",<name>,"Id")`, `"Parent"`, `"IsElimination"`, `"FiscalCalendar"`, `"LegalName"`, and `=FSN.CurrencyName(<name>)`. The members of a consolidated subsidiary `X (Consolidated)` are `X` and every subsidiary whose `Parent` chain reaches `X`'s `Id`. Note the elimination subsidiaries, the members whose fiscal calendar differs from the book subsidiary's, and the currencies.

**Segments.** Lists G to I. A list with one blank cell: the segment is unused for this subsidiary. Read the values, and which are `(Hierarchy)` rollups.

**Chart of accounts.** In the discovery area:

1. One list per account type code, as a spill down: `=FSN.Range.AccountNumbers(<code>,,Subsidiary)` for each of `FSN_L3_BNK`, `FSN_L3_ARE`, `FSN_L3_URE`, `FSN_L3_OCA`, `FSN_L3_DEX`, `FSN_L3_FAS`, `FSN_L3_LTO`, `FSN_L3_APL`, `FSN_L3_CCX`, `FSN_L3_OCL`, `FSN_L3_DRE`, `FSN_L3_LTL`, `FSN_L3_EQU`, `FSN_L3_INC`, `FSN_L3_COG`, `FSN_L3_EXP`, `FSN_L3_OIN`, `FSN_L3_OEX` (with the code typed in a heading cell, not in the formula). An account's type is the list it appears in. These lists include summary accounts, so the tree is complete.
2. Once they settle, write each account once, typed as text, down one column, and beside it: `=FSN.GLAccountName(<acct>)` and `=FSN.GetListInfo("GL Accounts",<acct>,<property>)` for `Id`, `Parent`, `IsSummary`, `SpecialAccountType` and `Eliminate`, with the list name and each property in heading cells.
3. Read them. `Parent` holds the parent's `Id`: build the tree from it. Keep NetSuite's order, the order of the lists.

From the table:

| Finding | How |
|---|---|
| Inventory accounts | `SpecialAccountType` `InvtAsset`, and the other posting accounts under the same parent |
| Undeposited Funds | `SpecialAccountType` `UndepFunds`, or the account named Undeposited Funds |
| Retained earnings and translation accounts | `SpecialAccountType` `RetEarnings`, `CumulTransAdj` or `CTA-E`: left out of Paid-in capital |
| Intercompany accounts | Balance sheet accounts only (the asset, liability and equity type lists) with `Eliminate` `T`, or a name with "intercompany", "due from" or "due to". Intercompany revenue, cost of sales and expense accounts stay on the operations lines: on Due from (to) affiliates their year-to-date activity would unbalance the balance sheet. Confirm in round 3. |
| Intercompany entity names | For each consolidating member, `=FSN.Range.Entities(<"*<member name>*" in a cell>,,Subsidiary,FALSE)`: the customers, vendors and other entities whose names hold the member's name, such as "IC - Canada". These are the candidates for `Affiliates`: the subsidiary names themselves often aren't the entity on intercompany lines, and then every balance lands in Other entities. Confirm in round 3. |
| Depreciation and amortization | Expense accounts, or a parent, named "depreciation" or "amortization" |
| Distributions | Equity accounts named "distribution", "dividend" or "drawing" |
| Prepaid expenses | An Other Current Asset parent named "prepaid" |
| Accrued groups | Other Current Liability parents |
| Accounts without a number | `FSN.Range.AccountNumbers` returns `id_<internal id>`: use it as the account everywhere |
| Account numbers with `-`, `^`, `*` or `?` | Finsyte reads these characters as filter syntax in lists. Use the account's `id_<Id>` in B instead, and tell the user why. |

---

## Writing the book

**Order.** Lists, then Inputs and the names, then Cover, then the report pages, then Index (it needs each page's Overall row). Write one page completely, sync, and move on. Tell the user which page you are on.

**Values and formulas.** Write each block as one 2-D array through `range.formulas`, with typed values in the same array. Keep each request well under 5 MB: about 2,000 rows by 25 columns. Call `context.application.suspendApiCalculationUntilNextSync()` before each write batch, so Excel calculates once per batch.

**Text cells first.** Set `numberFormat` to `@` on column B of the body and on any cell that holds an account number, and sync, before you write the values: an account such as `0100` keeps its zero, and `1000` stays text. Never write a formula into a cell formatted as text: Excel stores it as text. A cell that holds a formula gets `General` (or its number format) first.

**Spills.** Write a spilling formula in its anchor cell only, and keep the cells it spills into empty. A `#SPILL!` means something is in the way.

**Names.** `context.workbook.names.add("CurFY", inputs.getRange("C11"))`; for a spill, `context.workbook.names.add("ListFiscalYears", "=Lists!$E$2#")`. Check the result: a name that already exists throws.

**Dropdowns.** `range.dataValidation.rule = { list: { inCellDropDown: true, source: "=ListSubsidiaries" } }`, and `range.dataValidation.errorAlert = { showAlert: true, style: "Stop", message: "Choose a value from the list." }`; style `"Warning"` for the column lists. The Period cell: `{ wholeNumber: { formula1: 1, formula2: "=PeriodsPerYear", operator: "Between" } }`.

**Links.** `range.hyperlink = { documentReference: "'1'!A1", textToDisplay: "1" }`. Sheet names that are numbers need the quotes, in links and in formulas: `='1'!$C$1`.

**Outline.** For each group, `sheet.getRange("<first>:<last>").group(Excel.GroupOption.byRows)` over the rows that collapse (detail rows and the total), then group the inner rows of a nested group a second time for the second level. When all groups are made, `sheet.showOutlineLevels(1, 0)` collapses them, and the parameter block rows (6 to 19) get `rowHidden = true`. Office.js can't move the outline buttons above the groups: tell the user that Data › Outline › the dialog launcher › clear "Summary rows below detail" puts them on the header rows, if they want that.

---

## Formatting calls

**Format only the cells that need it.** Excel stores formatting cell by cell. Empty cells that carry formatting make the file larger and slower to open, save and calculate, and Excel's Check Performance flags them as unused formatting.

- **The font comes from the Normal style.** Before the first format call, set it once: `const normal = context.workbook.styles.getItem("Normal"); normal.font.name = "Times New Roman"; normal.font.size = 10;`. Every cell then has the book's font with no cell formatting. Format calls set only bold, italic, color or a size other than 10. Never set the font name on a range.
- **A row's content cells, never a block.** Format the cells that hold something, and the value and change cells a border, fill or number format must cover. Never format a whole page, a whole row or column, or a rectangle to the last column. Spacer columns take a width and nothing else; the list column and its spill area, and the rows below the checks block, take no formatting.
- **One conditional format per purpose per page**, on one range, or on several areas at once through `sheet.getRanges("I23:I182,N23:N182,S23:S182").conditionalFormats.add(...)`. Relative references cover every column: the future months are one rule over F to Q with `=F$8>Period`, never one rule per column.
- **Check each page.** `sheet.getUsedRange()` and `sheet.getUsedRange(true)` (values only) end on the same row and column. When the first is larger, formatting reached past the content: `range.clear(Excel.ClearApplyTo.formats)` on the extra rows and columns.

| Format | Call |
|---|---|
| Font | Name and size: the Normal style, above. On a range: `range.format.font.bold`, `.italic`, `.size` for another size, `.color = "#156082"` |
| Fill | `range.format.fill.color = "#CCEEFF"` |
| Borders | `range.format.borders.getItem("EdgeTop").style = "Continuous"`; `getItem("EdgeBottom").style = "Double"` |
| Indent | `range.format.indentLevel = 2` |
| Centered across columns | `range.format.horizontalAlignment = "CenterAcrossSelection"` on the whole span |
| Wrap | `range.format.wrapText = true`, and `range.format.rowHeight` for two lines |
| Column width | `range.format.columnWidth` is in points: points = (characters × 7 + 5) × 0.75, so 14.6 characters is about 80 points, 1.4 about 11, 3.6 about 23, 10 about 56 |
| Hidden column | `sheet.getRange("A:B").columnHidden = true` |
| Number format | `range.numberFormat = [[...]]`, a 2-D array the size of the range, on the amount cells of the rows that hold amounts |
| Conditional format | `const cf = range.conditionalFormats.add(Excel.ConditionalFormatType.custom); cf.custom.rule.formula = "=MOD(SUBTOTAL(103,$A$22:$A22),2)=1"; cf.custom.format.fill.color = "#CCEEFF";` Relative references are read from the range's top-left cell. For n/m: `cf.custom.format.numberFormat = '"n/m";"n/m";"n/m"'`. For statuses: `cf.custom.format.font.color` and `.bold`. |
| Freeze panes | `sheet.freezePanes.freezeRows(<first body row> - 1)` |
| Tab color, gridlines, hidden sheet | `sheet.tabColor = "#2E75B6"`, `sheet.showGridlines = false`, `sheet.visibility = Excel.SheetVisibility.hidden` |
| Page setup | `sheet.pageLayout.orientation = Excel.PageOrientation.landscape`; `.zoom = { horizontalFitToPages: 1, verticalFitToPages: 1 }` (a wide page: `{ horizontalFitToPages: 0, verticalFitToPages: 1 }`); `.centerHorizontally = true`; margins `leftMargin`, `rightMargin`, `topMargin`, `bottomMargin` = 36 points; `.setPrintArea("C1:T164")`; `.setPrintTitleColumns("$C:$E")`; `.setPrintTitleRows("$1:$4")` |

---

## Waiting for values

Finsyte fetches values in the background. A cell shows `#GETTING_DATA` (in Office.js, `values` returns the text `#GETTING_DATA` with `valueTypes` `Error`) until its value arrives, and any total over it is partial. A paused Finsyte returns 0 in place of the values (SKILL.md, "Verify", step 1).

1. Read each page's used range in blocks and count the `#GETTING_DATA` cells. Report the count to the user as it falls.
2. Read again every 30 to 60 seconds, and keep building the next page in between when you can.
3. When the count stops falling for five minutes, stop waiting: list the pages that still have pending cells, and ask the user to check that Finsyte is running and to use Finsyte › Refresh Data › Current Sheet on them. Never read checks or tie out while cells are pending.
4. Under Manual calculation, ask the user to press F9 before each read.

A large book can take several minutes on first calculation: Finsyte queries NetSuite for every value cell. Later opens are faster: Finsyte caches closed periods.
