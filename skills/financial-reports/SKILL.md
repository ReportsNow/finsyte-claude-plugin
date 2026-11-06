---
name: financial-reports
description: "Use when the user is building, changing, or checking an Excel report or financial model that pulls NetSuite data through Finsyte (the Excel add-in for NetSuite): balance sheets, income statements, cash flow, trial balances, budget vs actual, departmental or trailing P&Ls, KPI or board summaries; a workbook with =FSN. formulas or a Finsyte report template; replacing hard-coded numbers in a spreadsheet or NetSuite export with live Finsyte formulas; Finsyte range functions or dropdowns; transaction lines, segment breakdowns, A/R or A/P aging, SuiteQL queries, saved searches or datasets pulled into a sheet with FSN functions; or #VALUE!, #NAME?, Invalid account number and other Finsyte messages, #GETTING_DATA, #SPILL!, zeros from a paused Finsyte, or sign errors in FSN formulas. A health check of a whole workbook belongs to the workbook-health-check skill, and a multi-page board book to the board-book skill. Not for financial statements or accounting work outside Finsyte and NetSuite."
---

# Finsyte for NetSuite

Finsyte is an Excel add-in for building financial reports and models on NetSuite data. Finance and FP&A teams, accountants, and board members work from these workbooks, and NetSuite stays the source of truth: every number on the sheet is a formula that Finsyte resolves against NetSuite, so a report is refreshed rather than re-keyed. Use this skill to build new reports and models and to change existing ones.

## Reference files

Load one only when the task needs it:

| File | Read it when |
|---|---|
| `references/templates.md` | A report starts from, or already is, a Finsyte template: the 11 templates, their parameters, layouts, and sign rules, and how to customize one |
| `references/model-building.md` | Building a model: a Drivers tab, dropdowns, department or subsidiary columns, budget vs actual, trailing twelve months, KPI and board summaries |
| `references/converting.md` | Replacing hard-coded numbers in a spreadsheet or NetSuite export with Finsyte formulas, including CTA and the tie-out |
| `references/functions.md` | You need a function's full argument list, the Range function arguments and the lists for dropdowns, the drilldown functions, the cause of a Finsyte message, worked period-range examples, the sign of each account type, or the full tree of `FSN_` codes |

---

## How Finsyte gets a number

Every value comes from an account plus parameters. You name the account, or accounts, and the slice of NetSuite data; Finsyte builds and runs the query:

```
=FSN.GLAccountBalance(Subsidiary, Fiscal Year, Period, Period Range, Account, [Book, Item, Class, Department, Location, Entity, Transaction Type, Posted, Currency, custom segments…])
```

What goes in the Account argument decides what comes back, so one function covers almost every line of a financial report:

| Account argument | Returns | Example |
|---|---|---|
| A posting account number | That account's balance | `4000` |
| A summary account (Summary checked in NetSuite) | Every account under it | `6500` |
| A parent account that takes postings itself | Its own postings only. `FSN.GLAccountSummaryBalance` returns it together with every account under it | `6000` |
| An account type code, `FSN_L3_*` | Every account of that NetSuite account type | `FSN_L3_BNK`, all Bank accounts |
| A group or category code, `FSN_L2_*` or `FSN_L1_*` | Every account in that group or category | `FSN_L2_CAS` Current Assets, `FSN_L1_AST` Assets |
| Several accounts of one statement | Their total, each account once. A summary account or a code in the list brings in the accounts under it | `4000^4010`, a cell range, or a named range |
| `FSN_L4_NIN`, `FSN_L4_RET`, `FSN_L4_CBP` | Net income, retained earnings, and cash at the beginning of the period, calculated by Finsyte | |
| A statistical account | Its amount: headcount, units, square feet | `9900` |

`FSN_L0_BAL` is the one code it doesn't accept (it returns "Not Supported"); use `FSN_L1_AST` and `FSN_L1_LEQ`. The full tree of codes is in `references/functions.md`. Never invent a code by analogy: there is no `FSN_L1_INC`.

A list takes one period span for all its accounts, from its first account: under `YTD`, a list that mixes balance sheet and income statement accounts adds part of itself over the wrong span. `FSN_L4_NIN`, `FSN_L4_RET` and `FSN_L4_CBP` add nothing inside a list, and no message says so. Keep each list to accounts of one statement, and give each `FSN_L4` code its own cell.

Budgets work the same way. `FSN.BudgetAccountBalance(Subsidiary, Fiscal Year, Period, Period Range, Account, Budget Category, …)` takes account numbers, summary accounts, lists, and the `FSN_L1` to `FSN_L3` codes, plus `FSN_L4_NIN` on its own for budgeted net income. It doesn't accept the other `FSN_L4` codes or the `FSN_L0` codes. A budget list adds the accounts under every parent in it, parents that take postings included. One parent that takes postings, on its own, returns its own budget only. Give an actual and its budget the same accounts (`references/functions.md`, Budget functions).

## How reports are organized

Finsyte reports follow NetSuite's own financial statement layout. Every GL account has an account type, set in NetSuite when the account is created: Bank, Accounts Receivable, Other Current Asset, Fixed Asset, Accounts Payable, Long Term Liability, Equity, Income, Cost of Goods Sold, Expense, Other Income, Other Expense, and so on. Accounts sit under their type, types roll up into groups, groups into categories, and categories into the statement:

```
Balance Sheet           FSN_L0_BAL
└─ Assets               FSN_L1_AST
   └─ Current Assets    FSN_L2_CAS
      └─ Bank           FSN_L3_BNK
         └─ 1000 Checking        NetSuite accounts; sub-accounts nest under their NetSuite parent
```

Templates lay reports out this way, and a report you build should too: account rows under their type heading, a subtotal for each type, types rolled into groups, and groups into categories. A summary report can skip the account rows and pull each type, group, or category directly with its code.

---

## Choose the starting point

| The user wants | Start with |
|---|---|
| A standard statement: balance sheet, income statement, cash flow, budget vs actual, trial balance | A Finsyte template: *Start from a template* |
| To change a report they already have | *Change an existing report* |
| A model: a department P&L, budget vs actual by department or location, trailing twelve months, a KPI or board summary, several subsidiaries side by side | *Build a model*, starting from a template where one fits |
| Live formulas in place of hard-coded numbers in a spreadsheet or NetSuite export | `references/converting.md` |
| The transaction lines behind a balance, a breakdown by segment, period or account, A/R or A/P aging, or the rows of a SuiteQL query, saved search or dataset, in a sheet that refreshes | The drilldown and data query functions in `references/functions.md` |

The user clicks Finsyte's ribbon commands: templates, Load Accounts, and dropdowns. When a step needs one, tell the user exactly which command to use, then work with the sheet it produces.

### Start from a template

Templates are starting points. They match the layout and groupings of the same reports in NetSuite, and the user customizes them in Excel afterward. The four most used are Balance Sheet, Income Statement, Cash Flow, and Budget vs Actual; there are also Comparative versions of the first three and four Trial Balance variants (`references/templates.md`).

1. **Insert it:** Finsyte ribbon › Templates and Actions › Standard Reports, then the statement's submenu (Balance Sheet, Income Statement, Cash Flow, or Trial Balance) and the template. Budget vs Actual is directly on the Standard Reports menu. The template opens as a new sheet with parameter cells at the top.
2. **Set the parameters:** Subsidiary, Fiscal Year, Period, Period Range, plus Budget Category for Budget vs Actual. Each parameter cell has a Finsyte dropdown.
3. **Load Accounts, once:** Finsyte ribbon › Templates and Actions › Load Accounts. It writes the accounts grouped the way the same NetSuite report groups them, a balance formula per account that reads the parameter cells, `AGGREGATE` subtotals, Excel outline groups, cell styles, and the sign reversals the statement needs.
4. **Customize it in Excel:** insert or delete rows, rename sections, add columns for prior year, budget, or variance, and restyle with the Finsyte cell styles. See *Change an existing report*.

**Load Accounts once per sheet, and never again on an existing report.** It clears and rebuilds the whole generated area, so every customization inside it is lost. To report a different period, change the parameter cells. To report another subsidiary, copy the sheet and change the copy's Subsidiary cell. GL accounts created in NetSuite later don't appear on their own; add the check in *Keep reports complete*.

### Change an existing report

1. **Read before writing.** Find the parameter cells the formulas read, which rows carry a leading `-`, and the range each `AGGREGATE` total covers.
2. **Leave Finsyte's own formulas alone.** Template header rows such as `=IF(SUBTOTAL(102,…)=0, AGGREGATE(…), "")` and name cells such as `=IF(SUBTOTAL(…)>0, FSN.GLAccountName(…, "Total - "), FSN.GLAccountName(…))` are written by Finsyte and correct as they are. The *Formula conventions* below are for formulas you write.
3. **Add an account row** inside the block its subtotal covers, so the `AGGREGATE` range grows with it. Copy the formula from a neighboring account row in the same section, which carries the right sign, and change the account cell.
4. **Add a column** by copying a value column and pointing it at its own parameter cell: a prior-year Fiscal Year, another Subsidiary, a Budget Category. Variance columns are plain arithmetic on the value cells.
5. **Change what the report shows** by changing parameter cells, never by editing formulas one at a time.
6. **Don't run Load Accounts** on it. After changing the Subsidiary, add the check in *Keep reports complete* if the sheet doesn't have one, because another subsidiary can have accounts the report doesn't list.

### Build a model

Models combine the same pieces. Worked patterns with formulas are in `references/model-building.md`.

- **One set of inputs.** Put Subsidiary, Fiscal Year, Period, Period Range, Budget Category, and Book in labeled parameter cells, on a Drivers tab when several sheets share them, so one change re-points the whole model.
- **Dropdowns on the inputs.** The user selects the input cells, then Finsyte ribbon › Data Retrieval › Dropdowns › the list: Account, Subsidiary, Fiscal Year, Period Numbers, Period Range, Budget Category, Accounting Book, Currency, Item, Class, Department, Location, Entity, Transaction Type, or a custom segment. A dropdown keeps an input to values NetSuite accepts. You can't click the ribbon yourself: ask the user to, or build the dropdown from a Finsyte list on a lists sheet (`references/model-building.md`, Dropdowns).
- **Rows and columns from NetSuite lists.** Range functions spill lists that grow with NetSuite: `FSN.Range.Departments` with Horizontal set to TRUE for a column per department, `FSN.Range.Subsidiaries` for a column per entity, `FSN.Range.AccountNumbers` for account rows. Value formulas reference the header or row cell.
- **Summary lines from codes.** Revenue, cost of goods sold, operating expense, and net income come straight from `FSN_L3_INC`, `FSN_L3_COG`, `FSN_L3_EXP`, and `FSN_L4_NIN`, so a summary or board page needs no account list.
- **Comparisons are parameters.** Actual, budget, and prior year are the same account read with a different function or a different Fiscal Year cell.
- **Rolling reports.** Relative periods such as `<Current>` move a report forward each month; see *Periods*.

---

## Formula conventions

These apply to every formula you write. They let a workbook be re-pointed by editing one cell, keep refreshes fast, and keep totals right.

### Parameters live in cells

Every argument that varies goes in a labeled cell that the formula references, locked with `$`: Subsidiary, Fiscal Year, Period, Period Range, the account or code, Budget Category, Book, segment values, `<None>`.

❌ `=FSN.GLAccountBalance("HQ", "FY 2026", 6, "YTD", "FSN_L4_NIN")`
✅ `=FSN.GLAccountBalance($B$2, $B$3, $B$4, $B$5, $B$6)`

Type the values as plain text without quote marks: `FY 2026`, `FSN_L0_INC`, `Headquarters (Consolidated)`. The only literals that belong inside a Finsyte call are display text, such as the `"Total - "` prefix of `FSN.GLAccountName`, and the `TRUE` or `FALSE` switches of Range functions, such as Horizontal and Posting Only.

**Always fill in Subsidiary.** A blank Subsidiary means the signed-in user's default subsidiary, not all subsidiaries, so the same workbook would show different numbers to different people.

### Inputs are never volatile

Every cell a Finsyte call reads, directly or through other formulas, holds a typed value, a relative period such as `<Current>`, or a formula with no volatile function in it. The volatile functions are `TODAY`, `NOW`, `OFFSET`, `INDIRECT`, `CELL`, `INFO`, `SHEET`, `SHEETS`, `RAND`, `RANDBETWEEN`, `RANDARRAY`, and a `SUMIF` whose sum range is a different size from its criteria range. Finsyte calls are asynchronous, so two or more calls that depend on a volatile cell can restart each other: with results that arrive at different times, such as a closed period served from the cache next to an open period fetched live, Excel recalculates every 2 seconds until the workbook is closed, and with caching off every restart queries NetSuite again.

| Instead of | Use |
|---|---|
| `YEAR(TODAY())`, `MONTH(TODAY())`, `EOMONTH(TODAY(),-1)`, or `FSN.DateToPeriod(TODAY(), …)` for the current or last period | `<Current>` or `<Current - 1>` typed in the Period cell; it sets the fiscal year too |
| The subsidiary from the tab name with `CELL("filename")` | The subsidiary typed in the tab's own Subsidiary cell, with a Subsidiary dropdown |
| `INDIRECT` to read a sheet or cell by name | A direct reference, or `INDEX` or `CHOOSE` over the possible cells |
| `OFFSET` for a moving range | `INDEX`, or one column per `<Current - n>` period |

When a workbook you change or convert already has a volatile input that Finsyte calls would read, such as a Drivers cell with `=YEAR(TODAY())`, replace it as above before you point Finsyte calls at it, and tell the user which cells you changed and why.

### One Finsyte call per cell, with nothing around it but a minus

Write one `FSN.` call per cell. Never nest it in another `FSN.` call or wrap it in `IF`, `IFERROR`, `SUM`, or any other function. A leading `-` for sign is the one exception; it's what Finsyte's Switch Sign writes. Finsyte caches each call, Store Values on Open saves the values of one-call cells only, and the right-click menu (Drill Down, Switch Sign, Go to Budget) only works on an unwrapped call. Put `IF` or `IFERROR` in a separate cell that references the call.

### Totals use `AGGREGATE(9,0,range)`, never `SUM`

Write subtotal and total rows as `=AGGREGATE(9,0,<range>)`. Function 9 sums, and option 0 skips any nested `AGGREGATE` or `SUBTOTAL` in the range, so a parent total can span a block that already contains child subtotals without counting them twice. This is how templates total. Two consequences:

- A line item is never an `AGGREGATE`, or its parent total skips it. A line that combines cells, such as CTA, uses `+`.
- A line that is a difference of totals, such as Gross Profit or Net Operating Income, is plain arithmetic: `=$D$20-$D$35`.

### Several values, or no value

The Account, Class, Department, Location, Item, Entity, and custom segment arguments accept several values: a `^` string (`4000^4010`), a cell range, or a named range, which is best for a list reused across sheets. `FSN_L4_NIN`, `FSN_L4_RET` and `FSN_L4_CBP` are calculations: in any of these forms they add nothing, with no message, so give each its own cell.

For a segment, blank and `<None>` mean different things:

| Segment argument | Returns |
|---|---|
| Blank | All values combined, including transactions with no value |
| `<None>` | Only transactions where the segment is not set |

`<None>` works for every segment. Finsyte also accepts `<No Department>`, `<No Class>`, and the like, which is what range functions return for the "no value" entry. Subsidiary is not a segment: see *Always fill in Subsidiary*.

---

## Signs

`FSN.GLAccountBalance` returns values as NetSuite stores them: debit-normal accounts positive, credit-normal accounts negative. Finsyte doesn't flip them.

| Stored | Account types | On a report |
|---|---|---|
| + | Assets; Cost of Goods Sold, Expense, Other Expense | Use as returned |
| − | Liabilities, Equity, Income, Other Income | Prepend `-` so they display positive |

- **Net income is negative when profitable**, because `FSN_L4_NIN` sums raw balances. Prepend `-` to show profit as positive.
- **Never negate a budget.** `FSN.BudgetAccountBalance` and `FSN.BudgetNetIncome` already return revenue and a profitable net income as positive.
- **CTA takes no sign flip** on any of its parts (`references/converting.md`).
- **Templates already reverse the sections their statement needs**, and which sections differs by template (`references/templates.md`). Don't negate a template row a second time.
- To choose the sign from the account itself, put `=FSN.GetListInfo($B$7, $A10, $B$8)` in a helper cell (with `GL Accounts` and `AccountType` in the parameter cells) and pick the raw or negated value in a separate display cell.

The sign of every account type is in `references/functions.md`.

---

## Periods

| Code | Covers |
|---|---|
| ITD | Inception through the selected period |
| YTD | The fiscal year through the selected period; for balance sheet accounts, the same as ITD |
| RTD | The fiscal year through the selected period, even for balance sheet accounts |
| QTD | The quarter through the selected period; use the quarter's last period for a full quarter |
| PTD | The selected period only |
| TTD | A trailing number of periods, 12 by default (set in Preferences) |

A balance sheet uses `YTD`; an income statement matches its span: `PTD` for a month, `QTD`, or `YTD`. Fiscal Year is in NetSuite's format (`FY 2026`), and Period is a number, where 1 is the first period of the fiscal year.

**Relative periods.** Fiscal Year and Period accept `<Current>`, `<Current - 1>`, `<Current + 2>`, and so on, typed into the parameter cell, so a rolling report moves forward each month by itself. A relative Period also sets the fiscal year, overriding the Fiscal Year cell. The Help › Preferences › Relative Periods setting, off by default, only adds these values to the Fiscal Year and Period Numbers dropdowns; typed values work either way. Use fixed values for a report tied to a particular close.

---

## Functions

Arguments are in order; `*` marks a required one. Full tables, defaults, and the Range function arguments are in `references/functions.md`.

- `FSN.GLAccountBalance(Subsidiary, FiscalYear*, Period*, PeriodRange*, Account*, Book, Item, Class, Department, Location, Entity, TransactionType, Posted, Currency, CustomSegments…)`: the main function; see *How Finsyte gets a number*. A blank Book is the primary book, Posted defaults to `P` (`NP` is non-posted), and Currency to the subsidiary's. A subsidiary's amounts convert only into its own currency or a parent subsidiary's currency, because NetSuite's consolidated exchange rates run from a subsidiary to its parents: any other currency returns `Invalid Currency`, or `Error` on older Finsyte versions.
- `FSN.GLAccountDebitBalance(…)`, `FSN.GLAccountCreditBalance(…)`: the same arguments, one side only. They refuse the `FSN_L0` and `FSN_L4` codes.
- `FSN.GLAccountSummaryBalance(…)`: the same arguments; an account together with every account under it, for a parent that takes postings (*How Finsyte gets a number*).
- `FSN.BudgetAccountBalance(Subsidiary, FiscalYear*, Period*, PeriodRange*, Account*, BudgetCategory, Book, Item, Class, Department, Location, Entity)`: budget amounts. A blank segment combines all of its values. A blank Budget Category reads the Legacy category, or the company's first category, so always fill it in.
- `FSN.BudgetNetIncome(Subsidiary, FiscalYear*, Period*, PeriodRange*, BudgetCategory, Book, Item, Class, Department, Location, Entity)`: budgeted net income. There is no `FSN.BudgetAccountNetIncome`; it returns `#NAME?`.
- `FSN.StatAccountAmount(Subsidiary, FiscalYear*, Period*, PeriodRange*, Account*, Reserved, Reserved, Class, Department, Location, Entity)`: statistical accounts. `FSN.GLAccountBalance` with a statistical account returns the same amount and ignores Book, Item, Transaction Type, Posted, Currency and the custom segments.
- `FSN.GLItemQuantity(…)`, `FSN.GLItemUnit(…)`: the same arguments as `GLAccountBalance` except Currency, so custom segments start at the 14th; item quantity and unit of measure. One posting account per cell: a list or an `FSN_` code returns a message.
- `FSN.GLAccountName(Account*, Prefix, Suffix)`
- `FSN.GetListInfo(ListName*, Identifier*, Property*, Prefix, Suffix)`: a column of a Finsyte list, such as `AccountType`, `AccountCategory`, or `Parent` from `GL Accounts`. `FSN.GetParentListInfo` returns the value one level up.
- `FSN.GetPeriodName(Period*, FiscalYear*, Subsidiary)`, `FSN.PeriodToDate(FiscalYear*, Period*, Side)` (format the cell as a date; it reads the top-level subsidiary's calendar), `FSN.DateToPeriod(Date*, PeriodType, Subsidiary)`.
- `FSN.SubHasAccess(Subsidiary*, ListName*, Identifier*)` (the text `True` or `False`), `FSN.CurrencyName(Subsidiary*, BudgetCategory)`, `FSN.LastDataRefresh(Source)`.
- `FSN.Range.*`: lists that spill, including `AccountNumbers`, `Accounts`, `Subsidiaries`, `Departments`, `Classifications`, `Locations`, `Items`, `Entities`, `Customers`, `Vendors`, `Employees`, `Projects`, `BudgetCategories`, `CustomSegments`, the other entity lists, and the lists behind Finsyte's dropdowns, such as `Periods.FY`, `PeriodNumbers`, `PeriodRanges`, `AccountingBooks` and `Currencies`. `AccountNumbers` and the segment lists take different argument orders (`references/functions.md`).
- `FSN.Query(SuiteQL*, OmitHeader, Skip, Take)`, `FSN.SavedSearch(SearchName*, OmitHeader, Skip, Take)`, `FSN.Dataset(DatasetId*, OmitHeader, Skip, Take)`: tables that spill. `FSN.Query` and `FSN.Dataset` write every cell as text, numbers included, so `SUM` over their output returns 0: convert a column with `VALUE` before you total it (`references/functions.md`). `FSN.SavedSearch` returns numbers and dates. Saved searches need the Finsyte SuiteApp, which an administrator deploys once from Help › Preferences › SuiteApp Deploy.
- `FSN.TransactionLineDrilldown(BalanceType, Subsidiary, FiscalYear*, Period*, PeriodRange*, Account*, …)`, `FSN.SegmentDrilldown(Segment*, BalanceType, Subsidiary, …)`, `FSN.AgingSummary(Subsidiary, …)`: the right-click drilldowns as formulas that spill and refresh with their parameter cells: the transaction lines behind a balance, the balance broken down by a segment, period or account, and the A/R or A/P aging by customer or vendor. After their own leading arguments they take the arguments of `GLAccountBalance` (`references/functions.md`, Drilldown functions).

Use `FSN.GLAccountBalance` with a code instead of `FSN.GLAccountNetIncome`, `FSN.GLAccountRetainedEarnings`, `FSN.GLAccountTypeBalance`, `FSN.GLAccountGroupBalance`, `FSN.GLAccountCategoryBalance`, and `FSN.CashAtBeginningOfPeriod`. The first five still calculate but are hidden from Excel's function list, and one function keeps every balance cell in the same shape.

**Registered saved searches.** A company can register a saved search as its own function, `FSN.SavedSearch.<Name>`, with filter arguments. Names and arguments are specific to each company: use one only if the user names it or it already appears in the workbook, and copy its argument order from there.

### Segment argument positions

One comma off silently filters on the wrong segment. Count against this table:

| Function | Book | Item | Class | Department | Location | Entity |
|---|---|---|---|---|---|---|
| `GLAccountBalance`, Debit/Credit, `GLItemQuantity`, `GLItemUnit`, `AgingSummary` | 6 | 7 | 8 | 9 | 10 | 11 |
| `TransactionLineDrilldown` | 7 | 8 | 9 | 10 | 11 | 12 |
| `SegmentDrilldown` | 8 | 9 | 10 | 11 | 12 | 13 |
| `BudgetAccountBalance` | 7 | 8 | 9 | 10 | 11 | 12 |
| `BudgetNetIncome` | 6 | 7 | 8 | 9 | 10 | 11 |
| `StatAccountAmount` | — | — | 8 | 9 | 10 | 11 |

---

## Keep reports complete

A report with a fixed list of account rows goes stale silently. A GL account added in NetSuite later is missing from the totals, the report still ties internally, and recalculating doesn't add the row. Templates don't include a check for this, so add one at the bottom of every report that lists accounts, templates included:

```
=FSN.Range.AccountNumbers($B$6, $B$21:$B$216, $B$2,,,,,,TRUE)
```

- **Include Filter:** the report's statement code in a parameter cell: `FSN_L0_BAL` or `FSN_L0_INC`. A report with code rows or summary account rows narrows it (below).
- **Exclude Filter:** the range that holds the report's account numbers. It removes only the accounts it names.
- **Subsidiary:** the report's Subsidiary cell.
- **Posting Only**, the last argument: TRUE for a report that lists posting accounts only, including the plain Trial Balance templates. Leave it blank when the report lists summary accounts too, as the other templates do, or every summary account shows as missing.

The check doesn't suit the Cash Flow templates, which leave out Bank accounts and Undeposited Funds by design.

**Rows that hold a code or a summary account.** A row that holds an `FSN_` code, or a summary account whose posting accounts have no rows of their own, counts every account under it, the accounts NetSuite adds later included, so those accounts need no check. A parent read with `FSN.GLAccountSummaryBalance` counts the same way. The Exclude Filter removes only the accounts it names, so a check that includes them lists them all. Put in the Include Filter cell only the parts of the chart the report lists account by account: the codes of those types, or the parent accounts of those accounts, joined with `^`. On a P&L whose revenue lists its accounts and whose cost of sales and operating expenses are code rows, the Include Filter cell holds `FSN_L3_INC`.

It returns one blank cell when nothing is missing and spills the missing account numbers otherwise. Put a flag beside it (here the check is in `$Z$220`):

```
=IF(COUNTA($Z$220#)=COUNTBLANK($Z$220#), "✓ All accounts present", "⚠ Missing accounts — see below")
```

`COUNTA` counts the empty result `""` as 1, so compare it with `COUNTBLANK` rather than with zero. Add each missing account to its type's group by copying a neighboring row's formula. An account that a code row or a summary account row counts gets no row of its own: the total would count the account twice. The check also reads complete while Finsyte is signed out, because `FSN.Range.AccountNumbers` then returns one blank cell: read it with values on the sheet.

---

## Gotchas

1. **`#VALUE!` after pointing a parameter at another cell.** A cell formatted as Text stores a formula as text (`=Drivers!$C$10` shows in the cell). A call that reads it returns a message, such as `Invalid period number` for the Period, and a call with a leading minus shows `#VALUE!` in place of the message. The Period cell is Text-formatted on every template except Balance Sheet, Cash Flow, and Comparative Cash Flow. Set the cell to General, then enter the formula. In Office.js, set `numberFormat` to General and sync before setting `formulas`; Period should read back as a number.
2. **Wait for `#GETTING_DATA` to clear.** Finsyte pulls asynchronously. Values read while any cell shows `#GETTING_DATA` are partial, and a total that looks wrong mid-refresh is almost never a wrong code. Don't swap codes to fix it.
3. **Never guess account numbers.** A number that doesn't exist returns `Invalid account number`. Take numbers from `FSN.Range.AccountNumbers` or from the user's sheet.
4. **`#SPILL!`.** Range functions, `Query`, `SavedSearch`, and `Dataset` spill, so leave room. An empty result is one blank cell.
5. **Summary accounts in a flat list.** `FSN.Range.AccountNumbers` returns summary accounts with the accounts under them, and a summary account's balance already includes those accounts. For a flat list totaled with `AGGREGATE`, set Posting Only (the 9th argument) to TRUE. A parent that takes postings stays in that list and returns its own postings only, so it counts nothing twice.
6. **Rollup entries in segment lists.** `FSN.Range.Departments`, `.Classifications`, and `.Locations` include a `Parent (Hierarchy)` entry for each parent by default, which overlaps its children. For one column per department, set Include Rollup (the 7th argument) to FALSE.
7. **Subsidiary access.** Not every account is available to every subsidiary; `FSN.SubHasAccess` checks. It returns the text `True` or `False`, so a test of it compares text, such as `=IF(D5="True",…)` in a separate cell.
8. **Custom segments** appear as function arguments and template parameter rows after they are added to Segments Used in Help › Preferences › Custom Segments and Excel restarts. The Dropdowns › Custom Segments menu lists every active segment.
9. **What the user can do from Finsyte's menus.** Finsyte ribbon › Refresh Data offers Entire Workbook, Current Sheet, List Data, Purge Function Cache, and Data Connections. Right-clicking a Finsyte value offers Refresh Selection, Drill Down (Transaction Lines, Journal Entry Lines, Base Accounts, each segment, Subsidiary, Transaction Type, Entity, Item, or Period), Switch Sign, Go to Budget on budget cells, Static Snapshot Selection, A/R Aging Summary or A/P Aging Summary on A/R or A/P balances, and Open in NetSuite on drilldown rows.
10. **Paused or signed out.** The Finsyte ribbon button in the Refresh group reads Pause Connection while Finsyte runs and Resume Connection while it is paused. A paused Finsyte shows 0 in every numeric cell it holds no value in memory for, closed periods from the Summary Cache included, and one empty cell for the query, saved search and drilldown functions; the account and segment lists keep working. Help › Preferences › General › Pause Connection on Open pauses every workbook as it opens. Signed out, numeric cells show empty text, and a cell with a leading minus shows `#VALUE!`. A sheet of zeros or empty text holds no real values: ask the user what the button reads, then Resume Connection or sign in, and Refresh Data › Entire Workbook.
11. **Finsyte messages.** `references/functions.md`, Messages, gives the cause and the fix of each message a Finsyte cell can show, such as `Invalid period number specified`, `Invalid Book Name` or `This account number is not supported for this function`.

---

## Workbook health check

A health check of a whole workbook, for what makes the workbook slow or gives wrong numbers, belongs to the `workbook-health-check` skill in this plugin. The health check skill points to the sections of this skill for its fixes.

## Board book

A multi-page board of directors package, with a cover, an index, and statements by month, quarter, year, subsidiary, segment and budget, belongs to the `board-book` skill in this plugin. A one-page board or KPI summary stays here (`references/model-building.md`).

---

## Report QA checklist

Run these before handing over any report or model. Each one is pass or fail.

1. **Inputs evaluate.** No `#VALUE!` from parameter cells, and Period reads as a number.
2. **Pulls have settled on live values.** No `#GETTING_DATA`, no sheet of zeros or empty text from a paused or signed-out Finsyte (Gotcha 10), and key totals tie to NetSuite in magnitude and sign for one period.
3. **Nothing made up.** No `#NAME?` (such as `BudgetAccountNetIncome`), no `Invalid account number` or other Finsyte message, no invented `FSN_` codes.
4. **Signs are right.** Credit-normal actuals negated once; budgets and CTA never negated; template rows not negated a second time.
5. **Formulas you wrote follow the conventions.** No literals inside calls, Subsidiary filled in, one call per cell with nothing around it but `-`, totals as `AGGREGATE(9,0,…)`, and no line item written as an `AGGREGATE`. Finsyte's own template formulas are exempt.
6. **No volatile inputs.** No Finsyte call reads a cell that depends on `TODAY`, `NOW`, `OFFSET`, `INDIRECT`, `CELL`, `INFO`, `SHEET`, `SHEETS`, `RAND`, `RANDBETWEEN`, `RANDARRAY`, or a `SUMIF` with ranges of different sizes.
7. **The report is complete.** The missing-accounts check is present and clean on every report that lists accounts.
8. **It re-points from one place.** Changing an input and refreshing updates every number, label, and title, and no `FY 20xx` or subsidiary name is typed anywhere but the input cells.
9. **A converted sheet ties** to its original (`references/converting.md`).
