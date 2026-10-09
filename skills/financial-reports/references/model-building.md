# Building Models with Finsyte

Read this when building a model rather than a single standard statement: a workbook driven from one inputs tab, columns per department or subsidiary, budget vs actual, a trailing-twelve-month trend, or a KPI or board summary. Section names in *italics* refer to SKILL.md. In the examples, `$B$2` to `$B$5` hold Subsidiary, Fiscal Year, Period, and Period Range, and the other parameter cells are named where they're used.

## Contents

- Drivers tab
- Dropdowns
- Columns from a NetSuite list
- Budget vs actual
- Monthly trends and trailing twelve months
- KPI and board summaries

---

## Drivers tab

To make a whole workbook recompute from one place, drive every sheet from one inputs tab rather than setting parameters statement by statement.

1. **Create a Drivers sheet** as the first tab. Put the shared inputs in labeled cells: Subsidiary, Current Fiscal Year, Prior Fiscal Year, Period, Period Range, Budget Category, and Book if it isn't the primary book. Give the input cells Finsyte dropdowns (*Dropdowns* below).
2. **Name the inputs** (`CurrentFY`, `PriorFY`, `Subsidiary`) so formulas read clearly. Derive year labels from the input, for example `=VALUE(RIGHT(TRIM(CurrentFY),4))` for the year as a number, then `="FY "&(year-1)` for the prior year.
3. **Point every sheet's parameter cells at the Drivers cells.** Template parameter cells can be Text-formatted (the Period cell is, on every template except Balance Sheet, Cash Flow, and Comparative Cash Flow); set each to General before entering the reference, or it stores the formula as text and its dependents return `#VALUE!`.
4. **Drive every label from the inputs too.** Titles, subtitles, column headers, chart titles, and axis labels are easy to hard-code and forget:
   - Titles: `=Drivers!$C$4&" Board Summary | "&Drivers!$C$5`
   - Column headers: `=CurrentFY`, `=PriorFY`
   - Period labels: `=FSN.GetPeriodName(Drivers!$C$6, Drivers!$C$5, Drivers!$C$4)` in its own cell
   - Chart titles: link the title to a helper cell that builds the text from the inputs.
5. **Test the cascade.** Change one Drivers input, wait until no cell shows `#GETTING_DATA`, and check that every statement, header, and label moved with it; then restore the input. Finally, search the workbook for literal `FY 20` and subsidiary names: the only ones left should be the Drivers input cells.

---

## Dropdowns

A dropdown keeps an input to values NetSuite will accept. The user adds them:

1. Select the input cell, or several cells at once.
2. Finsyte ribbon › Data Retrieval › Dropdowns, then the list: Account, Subsidiary, Fiscal Year, Period Numbers, Period Range, Posting Period, Budget Category, Accounting Book, Currency, Item, Class, Department, Location, Entity (All, Customer, Vendor, Employee, Project, and other entity types), Transaction Type, or a custom segment.
3. To remove one, Finsyte ribbon › Productivity › Remove Dropdown.

Finsyte keeps each list on a hidden sheet whose name begins `FinsyteNS.` and points Excel data validation at it. Leave those sheets and names as they are. Template parameter cells come with dropdowns already. The Department, Class, and Location lists include `Parent (Hierarchy)` entries, which select a parent and all of its children.

If you are building the workbook yourself and can't use the ribbon, you can make an equivalent list: put a Range function on a lists sheet, for example `=FSN.Range.Departments(,,$B$2)` in `Lists!A2`, and set the input cell's data validation to List with the source `=Lists!$A$2#`. Prefer the ribbon when the user can click it.

---

## Columns from a NetSuite list

A Range function in the header row spills one column per department, class, location, or subsidiary, and picks up new ones on refresh. Each value formula reads its column's header cell.

**Department columns** (header in `C9`, accounts down column `A` from row 12):

```
C9:   =FSN.Range.Departments(,,$B$2,,,,FALSE,,TRUE)
C12:  =FSN.GLAccountBalance($B$2,$B$3,$B$4,$B$5,$A12,,,,C$9)
```

- The `FSN.Range.Departments` arguments are Include, Exclude, Subsidiary, Include Blank, Level, Include Inactive, Include Rollup, Rollup Only, Horizontal, Rollup First. Here Include Rollup (7th) is FALSE, so the list has no `Parent (Hierarchy)` entries that would overlap their children, and Horizontal (9th) is TRUE, so it spills across.
- Include Blank defaults to TRUE and adds a `<No Department>` column: transactions with no department. Keep it, so the department columns add up to the total.
- Department is the 9th argument of `FSN.GLAccountBalance` (*Segment argument positions*). Class is the 8th and Location the 10th; the Range functions for them are `FSN.Range.Classifications` and `FSN.Range.Locations`.
- Add a **Total** column with the Department argument left blank, which combines all departments: `=FSN.GLAccountBalance($B$2,$B$3,$B$4,$B$5,$A12)`. The department columns, including `<No Department>`, should add up to it.
- Apply the row's sign as usual: a leading `-` on Income and Other Income rows.

**The value grid doesn't grow by itself.** When a department is added in NetSuite, the header spills one more column, but no value formulas sit under it. Don't pre-fill spare columns: a blank header passes a blank Department, which returns the all-department total. Instead, add a check cell, for example `=IF(COUNTA($C$9#)=<number of department columns>, "✓ Columns complete", "⚠ New department: copy the last column across")`, and tell the user what it means.

**Subsidiary columns:** `=FSN.Range.Subsidiaries(,,,,,,TRUE)` spills across (Horizontal is the 7th argument; this function has no Subsidiary or Include Blank argument), and each value formula takes its Subsidiary from the header: `=FSN.GLAccountBalance(C$9,$B$3,$B$4,$B$5,$A12)`. The list adds a consolidated entry, such as `Headquarters (Consolidated)`, for each parent subsidiary; use the Include and Exclude filters to keep the ones the report needs. Each subsidiary returns amounts in its own currency, so columns in different currencies don't add up; use the consolidated subsidiary for a total.

---

## Budget vs actual

Start from the Budget vs Actual template when the layout fits. To build one, or to add budget columns to another report, use the same account in both functions. Here Actual is column C, Budget D, Variance E, and % Variance F:

| Column | Formula (row 12, account in `$A12`, Budget Category in `$B$6`) |
|---|---|
| Actual | `=-FSN.GLAccountBalance($B$2,$B$3,$B$4,$B$5,$A12)` on Income rows; no `-` on expense rows |
| Budget | `=FSN.BudgetAccountBalance($B$2,$B$3,$B$4,$B$5,$A12,$B$6)`, never negated |
| Variance | `=C12-D12` on income rows and `=D12-C12` on expense rows, so favorable is positive |
| % Variance | `=IF(D12<>0,E12/ABS(D12),0)` |

- **Net income:** with `FSN_L4_NIN` in `$B$7`, actual is `=-FSN.GLAccountBalance($B$2,$B$3,$B$4,$B$5,$B$7)` and budget is `=FSN.BudgetAccountBalance($B$2,$B$3,$B$4,$B$5,$B$7,$B$6)`. `=FSN.BudgetNetIncome($B$2,$B$3,$B$4,$B$5,$B$6)` returns the same budget figure. There is no `FSN.BudgetAccountNetIncome`.
- **By department or location:** the segment positions differ between the two functions. Department is the 9th argument of `GLAccountBalance` and the 10th of `BudgetAccountBalance`, because Budget Category sits in the 6th slot.
- **Several budgets:** each Budget Category (for example Board Approved and Reforecast) is its own column with its own parameter cell.
- Right-click a budget value › Go to Budget opens that budget in NetSuite.

---

## Monthly trends and trailing twelve months

**One trailing-twelve-month figure:** Period Range `TTD`. The number of trailing periods is 12 unless changed in Preferences.

**A column per month:** each column is `PTD` and reads its own Period and Fiscal Year from two helper rows above the headers.

- **For a rolling trend**, put `<Current - 11>`, `<Current - 10>`, … `<Current>` in the Period row. A relative Period sets its own fiscal year and wraps at year end, so the Fiscal Year row can simply reference the Drivers input.
- **For a trend that ends at a fixed period**, compute each column from the end period and year. With the end period in `$B$4`, the end year as a number in `$B$8`, and `k` the column's position from 1 to 12 (`COLUMNS($C$7:C7)` in the first month's column): Period `=IF($B$4-12+k<=0, $B$4+k, $B$4-12+k)` and Fiscal Year `="FY "&IF($B$4-12+k<=0, $B$8-1, $B$8)`. This assumes twelve periods a year; adjust it if the company uses a thirteenth adjustment period.
- **Headers:** `=FSN.GetPeriodName(C$7, C$6, $B$2)` in the header row, its own cell per column.
- **Balance sheet trends** use `YTD` in each column, which gives the closing balance for that month.

---

## KPI and board summaries

A summary page doesn't need an account list. Pull each line with a code, keep every code in a helper cell, and compute the rest with plain arithmetic:

| Line | Account argument | Sign |
|---|---|---|
| Revenue | `FSN_L3_INC` | `-` |
| Cost of goods sold | `FSN_L3_COG` | none |
| Gross profit | Revenue − cost of goods sold | arithmetic |
| Gross margin % | `=IF(Revenue<>0, GrossProfit/Revenue, 0)` | arithmetic |
| Operating expenses | `FSN_L3_EXP` | none |
| Net other income | `FSN_L1_OTH` | `-` |
| Net income | `FSN_L4_NIN` | `-` |
| Cash | `FSN_L3_BNK`, Period Range `YTD` | none |
| Accounts receivable | `FSN_L3_ARE`, `YTD` | none |
| Accounts payable | `FSN_L3_APL`, `YTD` | `-` |
| Headcount, units, other statistics | the statistical account number | none |

- Gross profit minus operating expenses plus net other income should equal net income. Add that as a check row.
- Put Actual, Budget, and Prior Year side by side: the budget column uses `FSN.BudgetAccountBalance` with the same code (never negated), and the prior-year column reads the Prior Fiscal Year input.
- Ratios such as revenue per employee or days sales outstanding are arithmetic on these cells, never `FSN.` calls wrapped in a formula.
- For a pack tied to a closed period, use fixed Fiscal Year and Period values rather than relative periods, so it doesn't change after it's sent.
- Before it goes out, run the *Report QA checklist* in SKILL.md and tie net income to NetSuite for the period.
