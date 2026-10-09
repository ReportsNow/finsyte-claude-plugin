# Converting a Static Sheet to Finsyte Formulas

Read this when the user has a spreadsheet of hard-coded numbers, often a report exported from NetSuite or an old workbook of pasted values, and wants the numbers replaced with live Finsyte formulas while keeping their layout. Section names in *italics* refer to SKILL.md.

If the user doesn't need to keep the layout and the sheet is a standard statement, a Finsyte template is quicker (SKILL.md, *Start from a template*). Otherwise, convert it in place.

An export is its own answer key: every number you replace can be checked against what was there. Use that instead of asking the user questions this skill already answers, such as how to represent "no department", how to calculate CTA, how to total, or which sign to use.

## Contents

- Steps 1–8
- Cumulative Translation Adjustment (CTA)

---

## 1. Keep the answer key

Before changing anything, duplicate the sheet, for example as `Balance Sheet (Original)`, and leave the copy untouched. Step 7 ties every converted cell back to it.

## 2. Survey the layout

Identify, without editing:

- **The header block:** company or subsidiary, report title, the date or date range, and the accounting book if shown.
- **Row types:** section headers, account rows (NetSuite usually writes number and name together, such as `1000 - Checking`), subtotal and total rows, and calculated rows such as Net Income, Retained Earnings, Cumulative Translation Adjustment, and Gross Profit.
- **What each column means:** a single amount, one column per period, one per subsidiary, one per department, class, or location, or comparative periods.

## 3. Build the parameter block

Add labeled parameter cells above the report or in a small Params area: Subsidiary, Fiscal Year (`FY 2026` format), Period (a number), Period Range, and Book if it isn't the primary book. When each column means something different, add a helper row above the column headers that holds that column's parameter: its period, subsidiary, or segment value.

Choose the Period Range from the report type: a balance sheet is `YTD`, which acts as inception to date for balance sheet accounts; an income statement matches the header's span, `PTD` for one month, `QTD` for a quarter, or `YTD`. If the fiscal calendar isn't clear from the header, state the Fiscal Year and Period you inferred and let the user correct it rather than asking an open question.

## 4. Isolate the account numbers

Put each account row's number in a helper column as text, so leading zeros survive, either typed as values or parsed with a formula such as `=TRIM(LEFT($A12, FIND(" - ", $A12) - 1))`. Leave the visible labels as they are. A row with no recognizable number isn't an account row; never invent a number for it.

## 5. Replace the account values only

In each account row's value cells, write one call that reads the parameter cells and that row's account-number helper:

```
=FSN.GLAccountBalance($B$2, $B$3, $B$4, $B$5, $H12)
=-FSN.GLAccountBalance($B$2, $B$3, $B$4, $B$5, $H12)        ← credit-normal sections
```

- **Sign:** NetSuite reports show revenue, liabilities, and equity as positive, so rows in those sections get the leading `-` (SKILL.md, *Signs*).
- **Segment columns:** pass the column's helper cell in the correct argument position (SKILL.md, *Segment argument positions*). The unassigned column, labeled something like `- No Department -`, `- No Class -`, `- None -`, or `Unassigned`, maps to `<None>`: keep NetSuite's visible label and put `<None>` in the helper cell the formula reads. A **Total** column across segments is the opposite case: leave the segment argument blank, because blank means all values combined. Using blank for the unassigned column would overstate it and break the cross-foot to Total.
- Large sheets generate many calls. Write them, then wait for `#GETTING_DATA` to clear before reading anything back.

## 6. Rebuild totals and calculated rows

- **Subtotals and totals** become `=AGGREGATE(9,0,<range>)` over the block they cover, never `SUM` and never the exported constant. A constant total stops tying the moment the data changes.
- **Differences of totals,** such as Gross Profit or Operating Income, are plain arithmetic on the total cells.
- **Net Income** on a balance sheet is `FSN.GLAccountBalance` with `FSN_L4_NIN` in a parameter cell, and **Retained Earnings** is the same with `FSN_L4_RET`. Each is its own call; neither can be combined with other accounts.
- **Cumulative Translation Adjustment** is four calls added with `+` and no sign flip; see below.

## 7. Tie out against the original

Once the pulls have settled, add a temporary check column: the converted value minus the value on the untouched copy. Every row should be zero, allowing for rounding. Read the differences before changing anything:

| The difference looks like | Likely cause |
|---|---|
| Exactly −2× the original (converted = −original) | Sign: a `-` is missing, or shouldn't be there |
| Far too large or too small, consistently | Period Range (`YTD` vs `PTD`), the wrong Period, or a consolidated vs non-consolidated Subsidiary |
| One segment column off while the Total column ties | The segment value is in the wrong argument position, or blank was used where `<None>` was meant |
| A total off while its rows tie | The `AGGREGATE` range doesn't match the block, or a line item was written as `AGGREGATE` |
| Small differences on a few rows | Transactions posted since the export; report these to the user, don't force them to match |

Remove or hide the check column only after the user has seen the result.

## 8. Finish

Add the missing-accounts check (SKILL.md, *Keep reports complete*) so the converted report doesn't go stale, then run the *Report QA checklist*.

---

## Cumulative Translation Adjustment (CTA)

NetSuite's consolidated balance sheet shows Cumulative Translation Adjustment as one row, but CTA isn't a GL account with postings: NetSuite calculates it as the amount that makes the translated balance sheet balance. There's no account number to pass to `FSN.GLAccountBalance`, so calculate it from four system accounts, each pulled by its own call. The Balance Sheet template uses the same four codes as four rows inside Equity with an `AGGREGATE` subtotal; when the calls sit outside the report, as below, add them with `+` in one CTA row.

1. **Put the four codes in parameter cells** in a small block beside the report or on a helper sheet: `FSN_L1_AST` (total assets), `FSN_L1_LEQ` (total liabilities and equity), `FSN_L4_RET` (retained earnings), and `FSN_L4_NIN` (net income).
2. **Make four separate calls, one per cell,** with the same Subsidiary, Fiscal Year, Period, and Period Range cells as the rest of the balance sheet:
   ```
   H40: =FSN.GLAccountBalance($B$2, $B$3, $B$4, $B$5, $G40)     ← $G40 = FSN_L1_AST
   H41: =FSN.GLAccountBalance($B$2, $B$3, $B$4, $B$5, $G41)     ← $G41 = FSN_L1_LEQ
   H42: =FSN.GLAccountBalance($B$2, $B$3, $B$4, $B$5, $G42)     ← $G42 = FSN_L4_RET
   H43: =FSN.GLAccountBalance($B$2, $B$3, $B$4, $B$5, $G43)     ← $G43 = FSN_L4_NIN
   ```
3. **Add them in the CTA row with `+`:** `=$H$40+$H$41+$H$42+$H$43`. CTA is a line item inside Equity, so it must not be an `AGGREGATE`, or the Total Equity `AGGREGATE(9,0,…)` would skip it.

- **No sign flip anywhere.** Add each value exactly as returned, and don't negate the result. This is an exception to negating equity lines: the calculation only works on the stored signs.
- **Why four calls instead of one:** `FSN_L4_RET` and `FSN_L4_NIN` are calculations, not account rollups, so they can't be combined in a `^` list or a range. Separate cells also leave each part visible for the tie-out.
- **Period Range:** use the balance sheet's own cell, `YTD`. Retained earnings is non-zero only under `YTD`, so any other range drops prior-year earnings and inflates CTA.
- **Several columns:** build one set of four helper cells per column, each reading that column's parameters.
- **Sanity check:** for a single-currency, non-consolidated subsidiary the four values net to zero. A non-zero result there usually means the pulls haven't settled or the four calls don't use identical parameters.
