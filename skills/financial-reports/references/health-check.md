# Workbook health check

A health check reads a whole workbook and reports what makes the workbook slow to calculate, open or save, or what can give wrong results. Every finding carries a rule ID, a severity, the cells, why it matters, and the fix. The check itself changes nothing. You make a fix only after the user says yes to the exact edits you listed.

## Ground rules

1. **The check writes nothing.** No edits, no temporary sheets or test cells, no Calculate, no Refresh Data. Read only.
2. **Every fix needs a yes to the listed edits.** List each edit with the cell, the old formula and the new formula, then wait. A request such as "fix all of it" asks for that list. The user's yes to the list starts the edits.
3. **Keep the old formulas.** Before an edit, record the old formula of every cell the edit touches, and restore the old formulas at the user's request. Excel's Undo does not always cover edits made by an add-in.
4. **Re-read before you edit.** The workbook can change after the check.
5. **Tie out every fix.** Record the values of the affected FSN cells before the edit, wait for `#GETTING_DATA` to clear, and compare. Report every difference. Cells that showed an error before the fix are expected to change.
6. **Hidden sheets stay hidden.** Report cells on hidden sheets with the sheet name. Unhide a sheet, or delete a sheet, name or style, only at the user's request.
7. **A workbook file gets a report only.** Without Excel, you read the file and report. You return no rewritten file: saving a workbook with a library such as openpyxl can drop charts, pivot tables, spill formulas and other parts, and nothing recalculates the FSN formulas. Give the user the steps to make in Excel.

| Thought | Reality |
|---|---|
| "The user said to fix all of it, so the confirmation can go." | "Fix all of it" asks for the list. List the edits and wait for the yes. |
| "The meeting is in an hour, so one write is fine." | A deadline changes nothing: list, yes, then edit. |
| "A temporary sheet to test the cause is harmless." | A test sheet is an edit. Read what you need, or ask the user. |
| "I'll clear the formatting past the data myself." | Excel's Check Performance does that cleanup, and Undo works there. Point the user to it. |
| "The skill says nothing about returning a fixed file." | Rule 7: a workbook file gets a report only. |

Red flags that mean stop and list the edits: "no confirmation round", "a temporary sheet", "I fixed what I could", a file name ending in `_fixed`.

## Read the workbook

**Finsyte loaded and settled.** Read the values of the FSN cells first. `#NAME?` in every FSN cell means Finsyte is not loaded: say so and skip the FSN checks. Cells that show `#GETTING_DATA` get a second read after about a minute. Cells still loading at the second read go in the report as `FSN_GETTING_DATA`.

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
| `VOLATILE_FUNCTION` | `NOW`, `TODAY`, `RAND`, `RANDBETWEEN`, `RANDARRAY`, `OFFSET`, `INDIRECT`, `CELL`, `INFO`, and `SUMIF` with a sum range of a different size from its range, in cells, conditional formatting and names | Info; Medium above 100 cells | Confirm |
| `WHOLE_COLUMN_REFERENCE` | References such as `A:A` and `1:1`. `SUMPRODUCT`, array formulas, spilling formulas and FSN calls process every cell of the column, about a million rows. `SUM`, `SUMIF` and similar functions stop at the last used row. | Medium inside `SUMPRODUCT`, array formulas, spilling formulas and FSN calls; Info elsewhere | Confirm |
| `ITERATIVE_CALCULATION` | Iterative calculation turned on, which hides circular references | Medium | Advise: Formulas, Error Checking, Circular References lists the circular references |
| `ERROR_CELL` | Other error cells, by error type and sheet | Medium | Advise |

**Workbook structure and settings**

| Rule ID | What the rule flags | Severity | Fix mode |
|---|---|---|---|
| `USED_RANGE_BLOAT` | A used range that ends more than 10,000 rows or 100 columns past the last cell with a value | Medium | Advise: Review, Check Performance |
| `CONDITIONAL_FORMATTING` | More than 100 conditional formatting rules on one sheet | Medium; High above 1,000 | Advise |
| `CELL_STYLES` | More than 1,000 custom cell styles | Medium; High above 10,000 | Advise: Review, Check Performance |
| `NAMES_BROKEN` | Defined names that refer to `#REF!` | Medium | Confirm |
| `EXTERNAL_LINKS` | Formulas and names that refer to other workbooks, such as `[Budget.xlsx]Sheet1!A1` | Medium | Advise: Data, Edit Links |
| `NAMES_HIDDEN` | Hidden defined names | Info | Advise |
| `NAMES_MANY` | More than 1,000 defined names | Info | Advise |
| `SHAPES` | More than 500 shapes on one sheet, or hidden or zero-size shapes | Medium for the count; Info for hidden or zero-size | Advise |
| `PRECISION_AS_DISPLAYED` | Precision as displayed turned on, which permanently rounds stored values | High | Advise |
| `CALCULATION_MANUAL` | Calculation set to Manual. FSN values may be stale until the next calculation. | Info | Advise |
| `HIDDEN_SHEETS` | Hidden and very hidden sheets | Info | Advise |

Check Performance (Review tab) is in Excel for Microsoft 365. For other Excel versions, Ctrl+End shows the last used cell, and the user clears the formatting past the data by hand.

## Volatile inputs

1. Find the volatile cells: formulas that call a function in the `VOLATILE_FUNCTION` row, plus `FSN.LastDataRefresh` and `FSN.Slicer`. Check names and conditional formatting too.
2. Collect the distinct cells and names that FSN calls reference, and read their formulas. A typed input has no precedents and needs no trace: 5,000 typed account cells cost one read and no trace.
3. Trace each input that holds a formula, once. Open workbook: `getPrecedents()` on that input cell returns every cell upstream, on any sheet. Workbook file: follow the references of the input's formula level by level, through cells and names. A name's own formula is checked directly.
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

1. For each FSN formula that shows `#VALUE!`, count the arguments from the start position to the end, empty ones included.
2. Ask the user for the Segments Used count in Help, Preferences, Custom Segments before you name the cause.
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

For a workbook file, the last line is: "I changed nothing and I am not returning a file. To make the fixes, open the workbook in Excel with Finsyte and follow the steps above."

## Fixes

| Rule ID | Fix |
|---|---|
| `FSN_VOLATILE_INPUT` | The SKILL.md replacement table, "Inputs are never volatile" |
| `VOLATILE_FUNCTION` | A direct reference, `INDEX` or `CHOOSE` in place of `INDIRECT`; `INDEX` in place of `OFFSET` |
| `WHOLE_COLUMN_REFERENCE` | A range that covers the data, such as `Data!$A$1:$A$2000`, or a table column. Compare the result before and after. |
| `NAMES_BROKEN` | Delete each name after confirming no formula, name, conditional format or data validation uses the name |
| `FSN_ERROR_MESSAGE`, `FSN_ERROR_VALUE` | The causes in SKILL.md, "Gotchas" |
| `FSN_SEGMENT_ARGUMENTS` | Trailing empty arguments removed (see "Custom segment arguments") |

Each fix follows the ground rules: the list, the yes, the old formulas kept, the edit, the tie-out.
