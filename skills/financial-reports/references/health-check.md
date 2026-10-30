# Workbook health check

A health check reads a whole workbook and reports what makes the workbook slow to calculate, open or save, or what can give wrong results. Every finding carries a rule ID, a severity, the cells, why it matters, and the fix. The check itself changes nothing. You make a fix only after the user says yes to the exact edits you listed.

## Ground rules

1. **The check writes nothing.** No edits, no temporary sheets or test cells, no Calculate, no Refresh Data. Read only.
2. **Every fix needs a yes to the listed edits.** List each edit with the cell, the old formula and the new formula, then wait. A request such as "fix all of it" asks for that list. The user's yes to the list starts the edits.
3. **Keep the old formulas.** Before an edit, record the old formula of every cell the edit touches, and restore the old formulas at the user's request. Excel's Undo does not always cover edits made by an add-in.
4. **Re-read before you edit.** The workbook can change after the check.
5. **Tie out every fix.** Before the edit, record the values of the cells the fix changes and of the FSN cells that read them. After the edit, wait for `#GETTING_DATA` to clear, and compare. Report every difference. Cells that showed an error before the fix are expected to change. Under Manual calculation the values are stale: ask the user to press F9 before you record the values and again after the edit.
6. **Hidden sheets stay hidden.** Report cells on hidden sheets with the sheet name. Unhide a sheet, or delete a sheet, name or style, only at the user's request. Finsyte's own `FinsyteNS.` sheets and names stay as they are (see "Checks").
7. **A workbook file gets a report only.** Without Excel, you read the file and report. You return no rewritten file: saving a workbook with a library such as openpyxl can drop charts, pivot tables, spill formulas and other parts, and nothing recalculates the FSN formulas. Give the user the steps to make in Excel.

| Thought | Reality |
|---|---|
| "The user said to fix all of it, so the confirmation can go." | "Fix all of it" asks for the list. List the edits and wait for the yes. |
| "The meeting is in an hour, so one write is fine." | A deadline changes nothing: list, yes, then edit. |
| "A temporary sheet to test the cause is harmless." | A test sheet is an edit. Read what you need, or ask the user. |
| "I'll clear the formatting past the data myself." | Excel's Check Performance does that cleanup, and Undo works there. Point the user to it. |
| "The skill says nothing about returning a fixed file." | Rule 7: a workbook file gets a report only. |

Stop and list the edits at any of these words in your own plan: "no confirmation round", "a temporary sheet", "I fixed what I could", a file name ending in `_fixed`.

## Read the workbook

**Finsyte loaded and settled.** Read the values of the FSN cells first. `#NAME?` in every FSN cell means Finsyte was not loaded at the last calculation, as in a file saved on a computer without Finsyte. Say so, skip the rules that read FSN values (`FSN_ERROR_NAME`, `FSN_ERROR_VALUE`, `FSN_ERROR_MESSAGE`, `FSN_GETTING_DATA`, `FSN_SEGMENT_ARGUMENTS`), and run every rule that reads formulas, `FSN_VOLATILE_INPUT` included. Cells that show `#GETTING_DATA` get a second read after about a minute. Cells still loading at the second read go in the report as `FSN_GETTING_DATA`.

**Open workbook (Office.js).**

| Item | Call |
|---|---|
| Formula cells, per sheet | `getUsedRange().getSpecialCellsOrNullObject("Formulas")`, then `formulas`, `values` and `valueTypes` for each area |
| Used range against data | `getUsedRange()` and `getUsedRange(true)`, which counts values only |
| Conditional formatting | `getUsedRange().conditionalFormats.getCount()` |
| Styles | `workbook.styles`, counting styles with `builtIn` false |
| Names | `workbook.names` and each sheet's `names`: `formula`, `visible`, `type` |
| Shapes | `worksheet.shapes`: count, `visible`, `width`, `height` |
| Sheet visibility | `worksheet.visibility` |
| Settings | `application.calculationMode`, `application.iterativeCalculation`, `workbook.usePrecisionAsDisplayed` |
| Spill size | `getSpillingToRange()` on cells with a drilldown function |
| Precedents | `getPrecedents()` on the input cells of FSN calls (see "Volatile inputs") |

Read each sheet in blocks that stay under the 5 MB request limit of Excel for the web, sheets with FSN formulas first. The user can limit the check to the active sheet.

**Workbook file (no Excel).** Load the file twice with openpyxl: once for formulas, once with `data_only=True` for the values saved at the last save. Read names, conditional formatting, styles, the sheet state (`hidden`, `veryHidden`), and `calcPr` in `xl/workbook.xml`: `calcMode="manual"`, `iterate="1"`, and `fullPrecision="0"`, which means precision as displayed.

## Find FSN calls

Every formula that contains `FSN.` holds an FSN call, including calls inside other functions, such as `=IFERROR(-FSN.GLAccountBalance(...),0)`. Before you match function names in any formula, remove string literals (`="TODAY"`) and quoted sheet names (`='Offset'!A1`), so text and sheet names never count as function calls.

The numeric FSN functions return a number on success: `GLAccountBalance`, `GLAccountDebitBalance`, `GLAccountCreditBalance`, `GLAccountSummaryBalance`, `BudgetAccountBalance`, `BudgetNetIncome`, `StatAccountAmount`, `GLItemQuantity`, `CashAtBeginningOfPeriod`, and the older balance functions in `references/functions.md`. Text in the cell of a numeric FSN function is an error message.

## Checks

Severity is High, Medium or Info. Fix mode **Confirm** means you make the fix after the user's yes to the listed edits. Fix mode **Advise** means you explain the fix and the user makes the fix in Excel or Finsyte.

**Finsyte**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `FSN_VOLATILE_INPUT` | FSN formulas that read a volatile cell, directly or through other cells and names, including `FSN.LastDataRefresh` and `FSN.Slicer` | High | Confirm |
| `FSN_ERROR_NAME` | FSN cells that show `#NAME?` | High | Advise |
| `FSN_ERROR_VALUE` | FSN cells that show `#VALUE!` | High | Confirm |
| `FSN_ERROR_MESSAGE` | Numeric FSN functions that return message text, such as "Invalid account number 1234 specified" or "Region 'East' not found" | High | Confirm |
| `FSN_GETTING_DATA` | FSN cells still showing `#GETTING_DATA` at the second read | Info | Advise |
| `FSN_SEGMENT_ARGUMENTS` | FSN formulas that pass more custom segment arguments than the functions take, and show `#VALUE!` | High | Confirm for trailing empty arguments, Advise otherwise |
| `FSN_DRILLDOWN_LARGE` | More than 50,000 cells of drilldown output on one sheet | Medium | Advise |

**Volatile and slow formulas**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `VOLATILE_FUNCTION` | `NOW`, `TODAY`, `RAND`, `RANDBETWEEN`, `RANDARRAY`, `OFFSET`, `INDIRECT`, `CELL`, `INFO`, and `SUMIF` with a sum range of a different size from its range, in cells, conditional formatting and names. Cells already in an `FSN_VOLATILE_INPUT` finding are left out. | Info; Medium above 100 cells | Confirm for `INDIRECT`, `OFFSET` and the `SUMIF` size mismatch; Advise for `TODAY`, `NOW`, `RAND`, `RANDBETWEEN`, `RANDARRAY`, `CELL` and `INFO` |
| `WHOLE_COLUMN_REFERENCE` | References such as `A:A` and `1:1`. `SUMPRODUCT`, array formulas, spilling formulas and FSN calls process every cell of the column, about a million rows. `SUM`, `SUMIF` and similar functions stop at the last used row. | Medium inside `SUMPRODUCT`, array formulas, spilling formulas and FSN calls; Info elsewhere | Confirm for Medium; Advise for Info, with no change, because whole columns in `SUM` and `SUMIF` cost little |
| `ITERATIVE_CALCULATION` | Iterative calculation turned on, which hides circular references | Medium | Advise: turn iterative calculation off first (File, Options, Formulas), then Formulas, Error Checking, Circular References lists the circular references |
| `ERROR_CELL` | Other error cells, by error type and sheet | Medium | Advise |

**Workbook structure and settings**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `USED_RANGE_BLOAT` | A used range that ends more than 10,000 rows or 100 columns past the last cell with a value | Medium | Advise: Review, Check Performance |
| `CONDITIONAL_FORMATTING` | More than 100 conditional formatting rules on one sheet | Medium; High above 1,000 | Advise |
| `CELL_STYLES` | More than 1,000 custom cell styles | Medium; High above 10,000 | Advise: Review, Check Performance |
| `NAMES_BROKEN` | Defined names that refer to `#REF!` | Medium | Confirm |
| `EXTERNAL_LINKS` | Formulas and names that refer to other workbooks: `[Budget.xlsx]Sheet1!A1` in an open workbook. A workbook file stores the same link as an index, such as `[1]Sheet1!A1`, with the path in `xl/externalLinks`. | Medium | Advise: Data, Workbook Links in Microsoft 365 (Data, Edit Links in older Excel) |
| `NAMES_HIDDEN` | Hidden defined names, apart from Finsyte's own `FinsyteNS.` names | Info | Advise |
| `NAMES_MANY` | More than 1,000 defined names | Info | Advise |
| `SHAPES` | More than 500 shapes on one sheet, or hidden or zero-size shapes | Medium for the count; Info for hidden or zero-size | Advise |
| `PRECISION_AS_DISPLAYED` | Precision as displayed turned on, which permanently rounds stored values | High | Advise |
| `CALCULATION_MANUAL` | Calculation set to Manual. FSN values may be stale until the next calculation. | Info | Advise |
| `HIDDEN_SHEETS` | Hidden and very hidden sheets, apart from Finsyte's own `FinsyteNS.` sheets | Info | Advise |

Check Performance (Review tab) is in Excel for Microsoft 365. For other Excel versions, Ctrl+End shows the last used cell, and the user clears the formatting past the data by hand.

**Finsyte's own items.** Sheets and names that start with `FinsyteNS.` hold the lists behind Finsyte dropdowns and templates, and Finsyte keeps them hidden. Report them in one Info line as Finsyte's own and expected. Never list them for deletion or unhiding: deleting them breaks the dropdowns. A user who asks to remove them gets that reason.

## Volatile inputs

1. Find the volatile cells and names: formulas that call a function in the `VOLATILE_FUNCTION` row, plus `FSN.LastDataRefresh` and `FSN.Slicer`. Check conditional formatting too. An FSN formula that calls a volatile function inside its own arguments, such as `MONTH(TODAY())` as the period, is an `FSN_VOLATILE_INPUT` finding by itself.
2. Collect the distinct cells and names that FSN calls reference, and read their formulas. A typed input has no precedents and needs no trace, so 5,000 typed account cells need one read and no trace.
3. Trace each input that holds a formula, once. Open workbook: read the input's formula first, then call `getPrecedents()` only on an input whose formula references other cells, one input per sync. `getPrecedents()` returns every cell upstream, on any sheet, and throws ItemNotFound for a formula without cell references: treat that error as no precedents. Workbook file: follow the references of the input's formula level by level. In both cases, expand every name in an input or precedent formula and check the name's formula, because precedents are cells only.
4. An input that is volatile, or that has a volatile precedent, flags every FSN formula that reads the input. Give one `FSN_VOLATILE_INPUT` finding per input cell, or per block of input cells with the same formula, such as `A5:A40`, with the count of FSN formulas.

The fix is the replacement table in SKILL.md, "Inputs are never volatile": `<Current>` or `<Current - 1>` in the period cell, a typed subsidiary, and a direct reference, `INDEX` or `CHOOSE` in place of `INDIRECT` and `OFFSET`.

## Custom segment arguments

Custom segments are the last arguments of 15 FSN functions: one argument per custom segment set up in Help, Preferences, Custom Segments, in that order. Excel returns `#VALUE!` for a formula that passes more arguments than a function takes, before Finsyte runs, and Excel counts empty arguments (`,,`) as arguments.

| Function | Custom segments start at argument |
|---|---|
| `GLAccountBalance`, `GLAccountDebitBalance`, `GLAccountCreditBalance`, `GLAccountSummaryBalance`, `CashAtBeginningOfPeriod`, `AgingSummary` | 15 |
| `GLItemQuantity`, `GLItemUnit` | 14 |
| `TransactionLineDrilldown` | 16 |
| `SegmentDrilldown` | 17 |
| Hidden: `GLAccountGroupBalance`, `GLAccountCategoryBalance`, `GLAccountTypeBalance` | 15 |
| Hidden: `GLAccountNetIncome` | 13 |
| Hidden: `GLAccountRetainedEarnings` | 12 |

1. For each FSN formula that shows `#VALUE!`, count the arguments from the start position to the end, empty ones included. A formula with no argument at or past the start position passes no segment arguments, so its cause is something else (see SKILL.md, "Gotchas").
2. For the formulas that pass segment arguments, ask the user for the Segments Used count in Help, Preferences, Custom Segments before you name the cause.
3. Formulas that pass more segment arguments than that count are the cause: `FSN_SEGMENT_ARGUMENTS`.
4. Trailing empty segment arguments filter nothing, so removing them keeps every result. List the edit with the old and new formula, and remove the arguments after the user's yes.
5. Segment arguments with values get advice: the segments are not set up on this computer. The user or an administrator sets them up in Help, Preferences, Custom Segments, then restarts Excel. For a workbook of another company, the Company Mismatch dialog at open leads to Use for Custom Features.
6. A cell that shows "<segment> '<value>' not found" holds a value the segment at that position does not have: `FSN_ERROR_MESSAGE`.

## Report

Give the report in this shape, in the chat. Write a sheet with the findings only at the user's request.

```
Health check: <workbook>, <sheets checked>
High: <n>   Medium: <n>   Info: <n>

High
1. <RULE_ID>: <what was found>. <count> cells: <up to ten addresses> and <n> more.
   Why it matters: <one sentence>.
   Fix: <the fix>. <I can make this fix after you say yes to the edits I list. | You make this fix in Excel or Finsyte: <steps>.>

Medium
...

Info
...

I changed nothing. I can step through the flagged cells one at a time, and I can make the Confirm fixes after you say yes to the edits I list.
```

For a workbook file, every Fix line uses the second form, "You make this fix in Excel: <steps>.", and the last line is: "I changed nothing and I am not returning a file. To make the fixes, open the workbook in Excel with Finsyte and follow the steps above."

Stepping through the flagged cells skips cells on hidden sheets, because Excel cannot select a cell on a hidden sheet. Name the hidden sheet and its cells instead, and leave the sheet hidden.

## Fixes

| Rule ID | Fix |
|---|---|
| `FSN_VOLATILE_INPUT` | The SKILL.md replacement table, "Inputs are never volatile" |
| `VOLATILE_FUNCTION` | A direct reference, `INDEX` or `CHOOSE` in place of `INDIRECT`; `INDEX` in place of `OFFSET`; a sum range the same size as the range in the `SUMIF`. `TODAY` or `NOW` in a title or an "as of" date, `RAND` in a model, `CELL` and `INFO` get advice only: each one does what the user built it to do, and the cost is a recalculation at every change. |
| `WHOLE_COLUMN_REFERENCE` | A range that grows with the data: a table column, a spill reference such as `A2#`, or `A2:INDEX(A:A,COUNTA(A:A))`. A fixed range such as `$A$1:$A$2000` drops rows added later. For an FSN account list, use the account range from SKILL.md, "Keep reports complete". The listed edit names the rows the new range covers. Compare the result before and after. |
| `NAMES_BROKEN` | Delete each name after confirming no formula, name, conditional format or data validation uses the name |
| `FSN_ERROR_MESSAGE`, `FSN_ERROR_VALUE` | The causes in SKILL.md, "Gotchas" |
| `FSN_SEGMENT_ARGUMENTS` | Trailing empty arguments removed (see "Custom segment arguments") |

Each fix follows the ground rules: the list, the yes, the old formulas kept, the edit, the tie-out.
