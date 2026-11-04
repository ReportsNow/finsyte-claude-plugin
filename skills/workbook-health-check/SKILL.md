---
name: workbook-health-check
description: "Use when the user asks for a health check of a whole Excel workbook that uses Finsyte (=FSN. formulas that pull NetSuite data), or says a Finsyte workbook is slow to calculate, open or save, keeps recalculating, has grown too large, or may give wrong numbers across sheets: volatile functions and inputs, whole-column references, FSN errors across many cells, drilldown and query output, custom segment #VALUE!, report totals and missing accounts, used range, styles, conditional formatting, names, links, shapes and calculation settings; and for fixing what such a check finds. A question about one cell or one formula, or building or changing a report, belongs to the financial-reports skill."
---

# Workbook health check

A health check reads a whole workbook and reports what makes the workbook slow to calculate, open or save, and what can give wrong numbers. Every finding carries a rule ID, a severity, the cells, why the finding matters, and the fix. The check itself changes nothing. You make one fix at a time, each after the user says yes to the exact edits you listed for it.

## The Finsyte skill

This skill finds problems. The Finsyte skill in the same plugin, `financial-reports`, explains how Finsyte works, and the fixes below point to its sections: Formula conventions, Inputs are never volatile, Signs, Keep reports complete, Gotchas, and `references/functions.md`. A user who added the Finsyte skill on its own may have it under the name `finsyte-netsuite`, an older copy. With both installed, use `financial-reports`, and tell the user that `finsyte-netsuite` can be turned off. Load the Finsyte skill before the check.

Without the Finsyte skill, tell the user that the Finsyte skill is missing, run the check, and report each finding with its rule ID, severity and cells. For a fix that needs a Finsyte section, name the section in place of the fix. `FSN_SUBSIDIARY_BLANK`, `FSN_LITERAL_ARGUMENT` and `ACCOUNTS_MISSING` need the argument positions in the Finsyte skill's `references/functions.md`: put them on the Not checked line with that reason.

## Ground rules

1. **The check writes nothing.** No edits, no temporary sheets or test cells, no Calculate, no Refresh Data. Read only.
2. **Every fix needs its own yes.** Make one fix at a time. List the edits of one fix, with the cell, the old formula and the new formula of each edit, then wait. The user's yes starts the edits of that fix and no others. After the tie-out, report the result, then list the next fix and wait for its yes. A request such as "fix all of it", or a yes given before any list, starts with the first fix's list. Take the fixes in the order the user names, or in the report's order. The user sees each fix's tie-out before the next edit.
3. **Keep the old formulas and settings.** Before an edit, record the old formula of every cell the edit touches, the old number format for a format change, the name, formula, scope and visibility of a name the edit deletes, and the old value of a setting the edit changes. Restore them at the user's request. The restore covers formulas, number formats, names and settings: a deleted style stays deleted, and the listed edit says so. Excel's Undo does not always cover edits made by an add-in.
4. **Re-read before you edit.** The workbook can change after the check.
5. **Tie out every fix.** Before the edit, record the values of the cells the fix changes and of the FSN cells that read them. After the edit, wait for `#GETTING_DATA` to clear, and compare. Report every difference. Cells that showed an error before the fix, and a total that counted a subtotal twice, are expected to change. Under Manual calculation the values are stale: ask the user to press F9 before you record the values and again after the edit. A tie-out needs Finsyte running (see "Run the check", step 3): a paused Finsyte returns 0.
6. **Hidden sheets stay hidden.** Report cells on hidden sheets with the sheet name. Unhide a sheet, or delete a sheet, name or style, only at the user's request, through a listed edit and a yes. Finsyte's own `FinsyteNS.` sheets and names stay as they are (see "Finsyte's own items").
7. **A workbook file gets a report only.** Without Excel, you read the file and report. You return no rewritten file: saving a workbook with a library such as openpyxl can drop charts, pivot tables, spill formulas and other parts, and nothing recalculates the FSN formulas. Give the user the steps to make in Excel.

| Thought | Reality |
|---|---|
| "The user said to fix all of it, so the confirmation can go." | "Fix all of it" asks for the first fix's list. List its edits and wait for the yes. |
| "The user already said yes to everything, so one round covers every fix." | A yes covers the one fix whose edits you listed. Make that fix, tie it out, then list the next fix. |
| "The meeting is in an hour, so one write is fine." | A deadline changes nothing: list, yes, then edit, one fix at a time. |
| "A temporary sheet to test the cause is harmless." | A test sheet is an edit. Read what you need, or ask the user. |
| "I'll clear the formatting past the data myself." | Excel's Check Performance does that cleanup, and Undo works there. Point the user to it. |
| "The skill says nothing about returning a fixed file." | Rule 7: a workbook file gets a report only. |
| "The values match, so the fix tied out." | A paused or signed-out Finsyte shows 0 or empty text. Confirm Finsyte is running before any tie-out. |

Stop and list the edits of one fix at any of these words in your own plan: "no confirmation round", "all the fixes at once", "a temporary sheet", "I fixed what I could", a file name ending in `_fixed`.

## Run the check

1. **Load the Finsyte skill.**
2. **Read the reading method** for your case: `references/open-workbook.md` for an open workbook, `references/workbook-file.md` for a workbook file.
3. **Finsyte state.** Finsyte runs only in Excel for Windows. In Excel for the web, on a Mac or on a tablet, the FSN cells hold the values saved in the file: say so, run every rule, and make no fix that needs the FSN values to recalculate. The user makes those fixes in Excel for Windows with Finsyte loaded. In Excel for Windows, ask the user to confirm that Finsyte is signed in and that the Finsyte ribbon button reads Pause Connection. A button that reads Resume Connection means Finsyte is paused, and a paused Finsyte returns 0 for every FSN cell with no cached value.
4. **Settings,** before any value: calculation mode, iterative calculation, precision as displayed. Calculation mode and iterative calculation apply to every open workbook and come from the first workbook opened, so the setting can come from another open workbook.
5. **Structure:** sheets and their visibility and protection, names, styles, shapes, conditional formatting, tables, PivotTables, data tables, links.
6. **Formulas,** sheet by sheet, sheets with FSN formulas first: the FSN calls and their inputs (see "Find FSN calls").
7. **FSN values.** Read the values of the FSN cells. Cells that show `#GETTING_DATA` get a second read after about a minute. Under Manual calculation, ask the user to press F9 before the second read.
8. **Rules.** Apply the rules below.
9. **Report** (see "Report").

**Every FSN cell shows `#NAME?`.** Finsyte was not loaded at the last calculation, as in a file saved on a computer without Finsyte. Say so, skip the value rules, and run every rule that reads formulas, `FSN_VOLATILE_INPUT` included.

**Finsyte signed out or paused.** Empty text in the one-call cells of numeric FSN functions, with `#VALUE!` in the negated ones, means Finsyte is signed out. Zeros mean a pause only with the Finsyte ribbon button at Resume Connection. With Pause Connection showing and the user signed in, zeros are balances, and the value rules run. For a signed-out or paused Finsyte, report `FSN_CONNECTION`, skip the value rules and `FSN_RESULT_MESSAGE`, and run every rule that reads formulas. A fix and its tie-out wait until the user signs in or clicks Resume Connection.

**Active sheet only.** The user can limit the check to the active sheet. Run the rules on that sheet, follow its inputs onto other sheets for `FSN_VOLATILE_INPUT`, and still run the settings rules. The report heading says that the other sheets were not checked.

## Find FSN calls

Read each formula in this order:

1. **Links.** A bracketed workbook name followed by a sheet name and `!`, such as `[Budget.xlsx]Sheet1!A1` or `'C:\Reports\[Forecast.xlsx]Q4'!B5`, is a link to another workbook (`EXTERNAL_LINKS`). A workbook file stores the link as an index, such as `[1]Sheet1!A1`. A workbook name followed by `!` and a defined name, such as `Budget.xlsx!Total` or `'C:\Reports\Budget.xlsx'!Total`, is a link to a name in another workbook, stored in a file as `[1]!Total`. A table reference such as `Table1[Amount]` is no link.
2. **Prefixes, text and sheet names.** Remove the `_xll.`, `_xlfn.` and `_xlws.` prefixes first, in any formula text: a workbook file stores every FSN call as `_xll.FSN.…`. Then remove string literals (`="TODAY"`), quoted sheet names (`='Offset'!A1`) and unquoted sheet names (`=Info!B3`), so text and sheet names never count as function calls.
3. **Function names.** A function name counts only where `(` follows the name and no letter, digit, `_` or `.` comes before the name.

Every formula that contains `FSN.` holds an FSN call. A **one-call cell** holds one FSN call with nothing around the call but an optional leading minus, such as `=-FSN.GLAccountBalance(...)`. The value rules (`FSN_ERROR_NAME`, `FSN_ERROR_VALUE`, `FSN_ERROR_MESSAGE`, `FSN_GETTING_DATA`) read one-call cells only. Every other cell with a numeric FSN function, such as `=IFERROR(-FSN.GLAccountBalance(...),0)` or `="Revenue for "&TEXT(FSN.GLAccountBalance(...),"#,##0")`, goes under `FSN_WRAPPED_CALL`, and every other FSN rule still reads its calls.

**Numeric FSN functions** return a number on success: `GLAccountBalance`, `GLAccountDebitBalance`, `GLAccountCreditBalance`, `GLAccountSummaryBalance`, `BudgetAccountBalance`, `BudgetNetIncome`, `StatAccountAmount`, `GLItemQuantity`, `CashAtBeginningOfPeriod`, and the hidden `GLAccountGroupBalance`, `GLAccountCategoryBalance`, `GLAccountTypeBalance`, `GLAccountNetIncome` and `GLAccountRetainedEarnings`. Text in a one-call cell of a numeric function is a Finsyte message. Empty text in these cells means Finsyte is signed out.

**Table FSN functions** spill a range: `TransactionLineDrilldown`, `SegmentDrilldown`, `AgingSummary`, `Query`, `SavedSearch`, `Dataset`, and the registered `SavedSearch.<Name>` functions. One cell of text in place of the table is a failed result. The `Range.` list functions spill lists of values, and a list of one value is a normal result.

Finsyte messages follow Office's display language, so the message text changes with the language. Recognize a message by the cell's value type.

## Rules

Severity is High, Medium or Info. Fix mode **Confirm** means you make the fix after the user's yes to the listed edits. Fix mode **Advise** means you explain the fix and the user makes the fix in Excel or Finsyte. At the user's request, an Advise fix becomes a listed edit with a yes, like a Confirm fix. A threshold counts per sheet for a rule that says "on one sheet", and per workbook for the others.

**Finsyte**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `FSN_CONNECTION` | Finsyte signed out (empty text in numeric FSN cells, `#VALUE!` in the negated ones) or paused (the Finsyte ribbon button reads Resume Connection, and FSN cells show 0) | High | Advise: sign in, or click Resume Connection, then Refresh Data › Entire Workbook, and run the check again |
| `FSN_VOLATILE_INPUT` | FSN formulas that read a volatile cell, directly or through other cells, names and spills (see "Volatile inputs") | High | Confirm; Advise for `FSN.Slicer` and `FSN.LastDataRefresh` |
| `FSN_ERROR_NAME` | One-call FSN cells that show `#NAME?`. Causes: Finsyte not loaded; a misspelled or retired function name, such as `FSN.BudgetAccountNetIncome`; a registered `FSN.SavedSearch.<Name>` function that Finsyte did not register in this session (Finsyte registers these functions as Excel starts, for the company set as Use for Custom Features); a function newer than the installed Finsyte. | High | Advise: correct the name; for a saved search function, set the company that set up the function as Use for Custom Features (Switch Company, then Use for Custom Features) and restart Excel; or update Finsyte |
| `FSN_ERROR_VALUE` | One-call FSN cells that show `#VALUE!` (see "#VALUE! causes") | High | Confirm for a Text-formatted input and for trailing empty segment arguments; Advise for the other causes |
| `FSN_ERROR_MESSAGE` | One-call cells of numeric FSN functions that show a Finsyte message, such as "Invalid account number" or "Region 'East' not found" | High | Advise: the causes in the Finsyte skill's Gotchas |
| `FSN_RESULT_MESSAGE` | A table FSN function whose result is one cell of text, such as "Not loaded: the drilldown was cancelled or timed out…" or "Saved search not found" | High | Advise: fix the cause the message names, then right-click the cell › Refresh Selection |
| `FSN_GETTING_DATA` | FSN cells still showing `#GETTING_DATA` at the second read, and the totals over them | Info | Advise: confirm the connection is running, then Refresh Data › Current Sheet |
| `FSN_SEGMENT_ARGUMENTS` | FSN formulas that pass more custom segment arguments than the functions take, and show `#VALUE!` | High | Confirm for trailing empty arguments; Advise for arguments with values |
| `FSN_DRILLDOWN_LARGE` | More than 50,000 lines of drilldown, Query, SavedSearch or Dataset output on one sheet (see "Drilldown and table output") | Medium | Advise: filter the drilldown or query to fewer lines, or delete output no longer needed. `FSN.Query`, `FSN.SavedSearch` and `FSN.Dataset` also take a Take argument, which returns the first lines only: a total over that output leaves the other lines out. |
| `FSN_WRAPPED_CALL` | Numeric FSN functions with more in the formula than a leading minus: `IFERROR`, `IFNA`, `IF(ISERROR(…))`, `TEXT`, `SUM`, arithmetic, or another FSN call. `IFERROR` and `IFNA` show their fallback in place of a Finsyte message or error. Finsyte's Store Values on Open keeps only one-call numeric cells, so these cells query NetSuite again after every open. Text FSN functions inside a label, such as the `="As of "&TEXT(FSN.PeriodToDate(…),"mmmm d, yyyy")` title of Finsyte's templates, are left out. | Medium for `IFERROR`, `IFNA` and `ISERROR`; Info for the others | Advise: put the FSN call in its own cell and reference that cell (the Finsyte skill's Formula conventions) |
| `FSN_SUBSIDIARY_BLANK` | Numeric FSN functions whose Subsidiary is empty or `<None>`, directly or through the input cell. Each user then sees their own default subsidiary. The `Range.` list functions are left out: for a list, an empty Subsidiary means all subsidiaries. | Medium | Advise: type the subsidiary in the input cell (Formula conventions, "Always fill in Subsidiary") |
| `FSN_BUDGET_NEGATED` | A leading minus on `FSN.BudgetAccountBalance` or `FSN.BudgetNetIncome` | Medium | Advise: the Finsyte skill's Signs |
| `FSN_LITERAL_ARGUMENT` | Subsidiary, Fiscal Year, Period, Period Range or Account typed inside an FSN call | Info | Advise: move the value into a labeled input cell (Formula conventions) |
| `FSN_HARDCODED_VALUE` | A typed non-zero number inside a block of numeric FSN formulas with one pattern, such as a value left by Static Snapshot Selection or a typed override. The zeros that Load Accounts types for empty sections are left out. | Info | Advise: replace the number with the formula of its neighbors, or label the cell as an override |

**Report totals and completeness**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `TOTAL_SUM` | A `SUM` whose range holds an `AGGREGATE` or `SUBTOTAL` cell together with the cells that cell totals, so the subtotal counts twice | High | Confirm |
| `AGGREGATE_LINE_ITEM` | An `AGGREGATE` cell inside a parent `AGGREGATE(9,0,…)` range whose own range lies outside the parent's range, so the parent total leaves the line out | High | Confirm |
| `TOTAL_OMITS_ROWS` | A total whose range stops next to more numbers in the same block | Medium | Advise: extend the range to the whole block |
| `ACCOUNTS_MISSING` | A missing-accounts check (`FSN.Range.AccountNumbers` with the report's account range as Exclude Filter) that lists account numbers (see "Missing-accounts checks") | High; Medium for a report whose account range holds an `FSN_` code; Info on a board book | Advise: the Finsyte skill's Keep reports complete |
| `ACCOUNTS_CHECK_ABSENT` | A sheet whose FSN balance calls read a column of account numbers, with no missing-accounts check. Cash Flow sheets (an `FSN_L4_CBP` row) and the report pages of a board book (see "Board books") are left out. | Info | Confirm |
| `NUMBERS_AS_TEXT` | Numbers stored as text inside a range that a `SUM`, `AGGREGATE` or `SUBTOTAL` totals. The total skips them. | Medium | Advise: select the cells, then the warning icon › Convert to Number |
| `BOOK_CHECKS` | A board book's own checks that read Error or Review: Book checks on Inputs, column G of Index, and each page's Overall cell and check rows (see "Board books") | High for Error; Info for Review | Advise: the board-book skill's Verify |

**Volatile and slow formulas**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `VOLATILE_FUNCTION` | `NOW`, `TODAY`, `RAND`, `RANDBETWEEN`, `RANDARRAY`, `OFFSET`, `INDIRECT`, `CELL`, `INFO`, `SHEET`, `SHEETS`, and `SUMIF` with a sum range of a different size from its range, in cells, conditional formatting and names. A volatile function in a conditional formatting rule gets this finding as well as any `CONDITIONAL_FORMATTING` finding on that rule. Cells already in an `FSN_VOLATILE_INPUT` finding are left out. | Info; Medium above 100 cells | Confirm for `INDIRECT`, `OFFSET` and the `SUMIF` size mismatch; Advise for the others |
| `WHOLE_COLUMN_REFERENCE` | References such as `A:A` and `1:1`. `SUMPRODUCT`, array formulas, spilling formulas and FSN calls process every cell of the column, about a million rows. `SUM`, `SUMIF` and similar functions stop at the last used row. A whole column inside `INDEX`, `COUNTA` or `MATCH` that only finds where a range ends is left out. | Medium inside `SUMPRODUCT`, array formulas, spilling formulas and FSN calls; Info elsewhere | Confirm for Medium; Info gets "No change needed" |
| `RUNNING_TOTAL` | Running totals such as `=SUM($A$2:A2)` filled down. Each cell adds every row above it again. | Info; Medium above 2,000 rows | Confirm |
| `LARGE_RANGE_LOOKUP` | More than 1,000 `SUMIFS`, `COUNTIFS` or exact-match lookups over ranges of more than 50,000 rows, such as a GL export or drilldown output | Info | Advise: a PivotTable or a summary built once from the data, or FSN calls in place of lookups over an export |
| `DATA_TABLE` | Data tables (the Data Table command on the Data tab). A data table recalculates its formulas once for every table cell. In a workbook with data tables, the finding also names the Automatic except for data tables setting. | Info; Medium for a data table whose input cell feeds FSN calls | Advise: move the scenarios into separate columns, or keep Automatic except for data tables and press F9 to update the table |
| `ITERATIVE_CALCULATION` | Iterative calculation turned on, which hides circular references | Medium | Advise: turn iterative calculation off (File › Options › Formulas; on a Mac, Excel › Preferences › Calculation), then Formulas › Error Checking › Circular References lists the circular references |
| `ERROR_CELL` | Other error cells, by error type and sheet. A cell and the cells that show its error are one finding. An error in a cell that an FSN finding names, or passed on from such a cell, goes with that finding, and `#GETTING_DATA` in totals goes with `FSN_GETTING_DATA`. | Medium | Advise: Formulas › Error Checking steps through the error cells; fix the first cell of each chain |

**Workbook structure and settings**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `USED_RANGE_BLOAT` | A used range that ends more than 10,000 rows or 100 columns past the last cell with a value, on one sheet | Medium | Advise: Review › Check Performance (Microsoft 365 for Windows and Excel for the web). In other Excel versions, Ctrl+End shows the last used cell: delete the rows and columns past the data, then save. |
| `CONDITIONAL_FORMATTING` | More than 100 conditional formatting rules on one sheet, duplicate rules (one formula and format in several rules, over one range or over ranges that copying and inserting split apart), and rules applied to whole columns | Medium; High above 1,000 rules | Advise: Home › Conditional Formatting › Manage Rules |
| `CELL_STYLES` | More than 1,000 custom cell styles | Medium; High above 10,000 | Advise: delete unused custom styles (Home › Cell Styles, right-click a style › Delete) |
| `NAMES_BROKEN` | Defined names that refer to `#REF!`, apart from `FinsyteNS.` names | Medium | Confirm |
| `NAMES_HIDDEN` | Hidden defined names, apart from `FinsyteNS.` names and Excel's own `_xlnm.` and `_xlfn.` names. Name Manager does not show hidden names. | Info | Advise: name each hidden name and what it refers to, and delete unused ones at the user's request through a listed edit and a yes |
| `NAMES_MANY` | More than 1,000 defined names, apart from `FinsyteNS.` names | Info | Advise: Formulas › Name Manager |
| `EXTERNAL_LINKS` | Formulas, names and link parts that refer to other workbooks | Medium | Advise: Data › Workbook Links in Microsoft 365 (Data › Edit Links in older Excel) |
| `PIVOT_REFRESH_ON_OPEN` | PivotTables and connections that refresh as the workbook opens. PivotTables built from one source normally share one cache; in a workbook file, several pivot caches with one source hold copies of the same data. | Info | Advise: PivotTable Options › Data, clear `Refresh data when opening the file`. For copied caches, rebuild each extra PivotTable from a copy of the first one, which shares its cache. |
| `SHAPES` | More than 500 shapes on one sheet, or hidden or zero-size shapes | Medium for the count; Info for hidden or zero-size | Advise: Home › Find & Select › Selection Pane |
| `PRECISION_AS_DISPLAYED` | Precision as displayed turned on, which permanently rounds stored values | High | Advise: File › Options › Advanced, `When calculating this workbook`, clear `Set precision as displayed`. Values already rounded stay rounded. |
| `CALCULATION_MANUAL` | Calculation set to Manual. FSN values may be stale until the next calculation. | Info | Advise: Formulas › Calculation Options › Automatic |
| `HIDDEN_SHEETS` | Hidden and very hidden sheets, apart from Finsyte's own `FinsyteNS.` sheets | Info | Advise: right-click a sheet tab › Unhide shows a hidden sheet; a very hidden sheet needs the Visual Basic editor |

**Workbook files only**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `CELL_FORMATS` | More than 10,000 unique cell formats. Excel stops at about 64,000. | Medium; High above 50,000 | Advise: Review › Check Performance, or clear the formatting of cells with no data |
| `FORCE_FULL_CALCULATION` | Full calculation forced at every recalculation, which recalculates every FSN call each time. Info items: multi-threaded calculation turned off, and a full calculation set for the next open. | Medium for forced full calculation; Info for the Info items | Advise: in the Visual Basic editor, set ThisWorkbook's ForceFullCalculation to False; turn on multi-threaded calculation in File › Options › Advanced |
| `FILE_SIZE` | A file over 50 MB, with its largest parts, or a .xls file | Info | Advise: compress or remove the largest images, delete drilldown output and sheets no longer needed, and save a .xls file as .xlsx |

**Finsyte's own items.** Sheets and names that start with `FinsyteNS.` belong to Finsyte. They hold the lists that Finsyte dropdowns read, template parameter cells and report areas, Bulk Records ranges and Expense Report state, and Finsyte keeps the sheets hidden. Formulas on `FinsyteNS.` sheets are Finsyte's own and stay out of every rule. Report the sheets and names in one Info line as Finsyte's own and expected. Never list them for deletion or unhiding: deleting them breaks dropdowns, templates and other Finsyte features. Finsyte re-creates `FinsyteNS.DrilldownTotal`, so a `#REF!` in that name needs no fix. A user who asks to remove them gets that reason.

## Volatile inputs

1. Find the volatile cells and names: formulas that call a function in the `VOLATILE_FUNCTION` row, plus `FSN.LastDataRefresh` and `FSN.Slicer`. An FSN formula that calls a volatile function inside its own arguments, such as `MONTH(TODAY())` as the period, is an `FSN_VOLATILE_INPUT` finding by itself.
2. Collect the distinct cells and names that FSN calls reference, and read their formulas. A typed input has no precedents and needs no trace. A spilled cell has no formula of its own: before you treat an input as typed, check for a spill parent, and trace the parent cell.
3. Trace each input that holds a formula once, and each block of inputs with one formula once, such as `A5:A40`. Read the formula first, then follow its references level by level. Expand every name in an input or precedent formula and check the name's formula.
4. An input that is volatile, or that has a volatile precedent, flags every FSN formula that reads the input. Give one `FSN_VOLATILE_INPUT` finding per input cell, or per block of input cells with one formula, with the count of FSN formulas.

The fix is the replacement table in the Finsyte skill's "Inputs are never volatile": `<Current>` or `<Current - 1>` in the period cell, a typed subsidiary, and a direct reference, `INDEX` or `CHOOSE` in place of `INDIRECT` and `OFFSET`. A `SHEET` or `SHEETS` input gets a typed value. `FSN.Slicer` and `FSN.LastDataRefresh` have no replacement: each one does what the user built it to do. Their Fix line is advice only, in this form: "Advice only. `<function>` is volatile by design, so the FSN formulas that read it recalculate every time Excel calculates. It stays as you built it."

## #VALUE! causes

Finsyte functions return a number, a table or a message. A one-call FSN cell that shows `#VALUE!` has one of these causes. Check them in this order:

1. **An input cell that shows an error.** Finsyte returns a message that names the argument and the error, such as "Account is #N/A". A call without a minus shows the message (`FSN_ERROR_MESSAGE`), and the minus of a negated call turns the message into `#VALUE!`. Report the input cell. An input cell that still shows `#GETTING_DATA` keeps the FSN cell at `#GETTING_DATA` (`FSN_GETTING_DATA`).
2. **A leading minus over a Finsyte message.** Finsyte templates and Switch Sign write `=-FSN…`, and the minus turns a message into `#VALUE!`. The user sees the message with Formulas › Evaluate Formula on the cell. Ask the user for the message before you name the cause.
3. **A Text-formatted input that holds formula text,** such as `=Drivers!$C$10` showing in the cell. A call without a minus shows a message, such as "Invalid period number" for the period, and a call with a minus shows `#VALUE!`. Fix (Confirm): set the input's format to General, then enter the formula again, and record the old format with the old formula.
4. **More custom segment arguments than the functions take** (see "Custom segment arguments").
5. **A range of several cells in an argument that takes one value.**

## Custom segment arguments

Custom segments are the last arguments of 15 FSN functions: one argument per custom segment in Segments Used, in that order, for the company set as Use for Custom Features. Excel returns `#VALUE!` for a formula that passes more arguments than a function takes, before Finsyte runs, and Excel counts empty arguments (`,,` and `""`) as arguments.

| Function | Custom segments start at argument |
|---|---|
| `GLAccountBalance`, `GLAccountDebitBalance`, `GLAccountCreditBalance`, `GLAccountSummaryBalance`, `CashAtBeginningOfPeriod`, `AgingSummary` | 15 |
| `GLItemQuantity`, `GLItemUnit` | 14 |
| `TransactionLineDrilldown` | 16 |
| `SegmentDrilldown` | 17 |
| Hidden: `GLAccountGroupBalance`, `GLAccountCategoryBalance`, `GLAccountTypeBalance` | 15 |
| Hidden: `GLAccountNetIncome` | 13 |
| Hidden: `GLAccountRetainedEarnings` | 12 |

1. For each one-call FSN formula that shows `#VALUE!` after causes 1 to 3 are ruled out, count the arguments from the start position to the end, empty ones included. A formula with no argument at or past the start position passes no segment arguments. Finsyte's templates pass segment arguments as references to parameter cells, such as `$C$9`: each reference is one argument, a reference to an empty cell counts as an empty argument, and the listed edit that removes one names the parameter cell.
2. For the formulas that pass segment arguments, ask the user for the Segments Used count in Help › Preferences › Custom Segments, and whether a notice above the lists says that Finsyte functions use the custom segments of another company. The question goes in the finding's Fix line in the first report, next to any listed removal of trailing empty arguments. With that notice, the functions take the other company's segments, and the count shown in the pane does not apply.
3. Formulas that pass more segment arguments than the functions take are the cause: `FSN_SEGMENT_ARGUMENTS`.
4. Trailing empty segment arguments filter nothing, so removing them keeps every result, whatever the count turns out to be. List that edit with the old and new formula in the first report, before the count is known, and remove the arguments after the user's yes.
5. Segment arguments with values get advice. Segment setup is company-wide: a Finsyte administrator of the company sets up the segments in Help › Preferences › Custom Segments, and Excel restarts. For the segments of another company, the user clicks Switch Company, selects that company, and clicks Use for Custom Features; Finsyte's message then says whether Excel needs a restart. Both paths end with Refresh Data › Entire Workbook.
6. A cell that shows a message naming a segment value that the segment does not have is an `FSN_ERROR_MESSAGE` finding.

## Missing-accounts checks

The Exclude Filter of `FSN.Range.AccountNumbers` removes only the accounts it names. A row that holds an `FSN_` code, or a summary account whose posting accounts have no rows of their own, counts every account under it, and the check still lists those accounts.

- With an `FSN_` code in the report's account range, alone or in a `^` list, `ACCOUNTS_MISSING` is Medium. Name the code rows, and say that the check also lists the accounts under those codes, which the totals count. The Fix is the exclude list in the Finsyte skill's "Keep reports complete", after which the check lists only the accounts that no row counts.
- A summary account row works the same way: a listed account that the user says sits under a summary account row is counted too.
- A listed account that a code row or a summary account row counts gets no row of its own: the total would count the account twice.
- On a board book, see "Board books".

## Board books

A board book built with the board-book skill has sheets named Inputs, Cover and Index, numbered report pages, a hidden sheet named Lists, and the workbook name `BookChecks`. Each section of its pages ends in an Other line that subtracts the lines beside it from a NetSuite rollup, so an account that NetSuite adds later still lands in the totals. Below each page's footer, a block headed `CHECKS (not printed)` holds the page's checks, with their statuses in column E and the page's Overall cell in column F of the header row.

- `BOOK_CHECKS`: read `BookChecks` on Inputs, column G of Index, and the Overall cell of each page. Give one finding per page that reads Error or Review, with the check rows that read Error or Review and their values. Error is High, Review is Info. The Fix points to the board-book skill's Verify table. A page whose only Review is its missing-accounts row gets no `BOOK_CHECKS` finding: that Review goes in the page's `ACCOUNTS_MISSING` finding.
- `ACCOUNTS_CHECK_ABSENT` leaves out the report pages: the Other lines keep them complete, and only pages 1 and 2 carry a missing-accounts check.
- `ACCOUNTS_MISSING` on pages 1 and 2 is Info: the listed accounts sit in a group's plug row, an Other line or an Other group's `Accounts not listed` row, so the totals count them. The Fix is the board-book skill's "List a new account".
- The hidden Lists sheet is the board book's own: leave it out of `HIDDEN_SHEETS`, and never list it for unhiding or deletion.

## Drilldown and table output

Drilldown output comes in three forms:

- A spill from a table FSN function. Count the lines of the spill.
- A static drilldown sheet: the text "Source Formula:" in the first rows and an Excel table of values below it. Count the table's rows.
- Values left by Static Snapshot Selection, with no formula.

`FSN_DRILLDOWN_LARGE` counts lines, the unit of Finsyte's own drilldown warning, with 50,000 lines as the threshold.

## Report

Give the report in this shape, in the chat. Write a sheet with the findings only at the user's request.

```
Health check: <workbook>, <sheets checked>
High: <n>   Medium: <n>   Info: <n>

High
1. <RULE_ID>: <what was found>. <count> cells: <up to ten addresses> and <n> more.
   Why it matters: <one sentence>.
   Fix: <the fix, with the Finsyte skill section it follows>. <I can make this fix after you say yes to the edits I list. | You make this fix in Excel or Finsyte: <steps>. | No change needed.>

Medium
...

Info
...

Other
...

Not checked: <rule IDs> (<reason>).

I changed nothing. I can step through the flagged cells one at a time, and I can make the Confirm fixes one fix at a time, each after you say yes to its edits.
```

- One finding is one rule and one cause. Within a severity, findings follow the order of the rule tables.
- Number the findings straight through the report: the first Medium finding takes the number after the last High one, and so on through Other.
- Every finding has the three lines of the template: what was found, `Why it matters:` and `Fix:`.
- A problem that fits no rule goes under "Other" as Info, with advice.
- "Not checked" lists the rules that could not run and why, such as a protected sheet that could not be read, a rule that a workbook file cannot show, or FSN values from a session without Finsyte.
- The heading names the sheets checked. A check of the active sheet says in the heading that the other sheets were not checked.
- For a workbook file, every Fix line uses the form "You make this fix in Excel: <steps>.", and the last line is: "I changed nothing and I am not returning a file. To make the fixes, open the workbook in Excel with Finsyte and follow the steps above."
- Stepping through the flagged cells skips cells on hidden sheets, because Excel cannot select a cell on a hidden sheet. Name the hidden sheet and its cells instead, and leave the sheet hidden.

## Fixes

Each fix follows the ground rules: its own list and yes, the old formulas kept, the edit, the tie-out. Start each fix's list with the finding's number and rule ID from the report. After the tie-out, run the rule again on the edited cells, report the result, and then list the next fix.

| Rule ID | Fix |
|---|---|
| `FSN_VOLATILE_INPUT` | The replacement table in the Finsyte skill's "Inputs are never volatile" |
| `FSN_ERROR_VALUE` | For a Text-formatted input: General, then the formula entered again. The other causes: see "#VALUE! causes". |
| `FSN_SEGMENT_ARGUMENTS` | The trailing empty arguments removed (see "Custom segment arguments") |
| `VOLATILE_FUNCTION` | A direct reference, `INDEX` or `CHOOSE` in place of `INDIRECT`; `INDEX` in place of `OFFSET`; a sum range the same size as the range in the `SUMIF`. `TODAY` or `NOW` in a title or an "as of" date, `RAND`, `RANDBETWEEN` and `RANDARRAY` in a model, and `CELL`, `INFO`, `SHEET` and `SHEETS` get advice only: each one does what the user built it to do, and each one recalculates at every change. |
| `WHOLE_COLUMN_REFERENCE` | A range that grows with the data, in this order: a table column where the data is an Excel table; `A2:INDEX(A:A,COUNTA(A:A))` for a column with one header cell and no empty cells; a trim reference such as `A:.A`, after asking whether everyone who opens the workbook has Microsoft 365, because Excel 2021 and 2024 show `#NAME?` for a trim reference; a spill reference such as `A2#` for spilled data. A fixed range such as `$A$1:$A$2000` is no fix: it drops rows added later. For an FSN account list, use the range that holds the report's account numbers, as in the Finsyte skill's "Keep reports complete". The listed edit names the rows the new range covers. Compare the result before and after. |
| `TOTAL_SUM` | `AGGREGATE(9,0,<the same range>)` (the Finsyte skill's Formula conventions). `AGGREGATE` leaves out every `AGGREGATE` and `SUBTOTAL` cell in its range, so first fix a line item in the range that is itself an `AGGREGATE` (`AGGREGATE_LINE_ITEM`). The listed edit names the subtotals that the total stops counting twice. |
| `AGGREGATE_LINE_ITEM` | The line written with `+` over its cells, such as `=G5+G6+G7+G8` |
| `ACCOUNTS_CHECK_ABSENT` | The check formula and its flag from the Finsyte skill's "Keep reports complete", in empty cells below the report, with that section's exclude list for rows that hold a code or a summary account |
| `RUNNING_TOTAL` | The first total as `=A2`, and each later total as the previous total plus the current value with `SUM`, such as `=SUM(B2,A3)`. `SUM` skips a text cell, where `=B2+A3` returns `#VALUE!`. |
| `NAMES_BROKEN` | Delete each name after confirming that no formula, name, conditional format, data validation or chart uses the name |
