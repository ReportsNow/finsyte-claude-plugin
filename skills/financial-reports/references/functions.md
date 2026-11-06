# Finsyte Function Reference

Read this when you need the exact position, requirement, or default of an argument that SKILL.md's function list doesn't settle, the Range function arguments and the lists for dropdowns, the drilldown functions, the cause of a Finsyte message, worked period-range examples, the sign of an account type, or the full tree of `FSN_` codes. Section names in *italics* refer to SKILL.md.

## Contents

- GL account functions
- Budget functions
- Statistical and inventory functions
- Period functions
- Utility functions
- Data query functions
- Drilldown functions
- Range functions
- Older functions to avoid
- Period range examples
- Account type sign table
- System-generated account tree
- Messages

---

## GL account functions

### FSN.GLAccountBalance

Returns the balance of an account, or of a set of accounts, for the slice of data the other arguments select.

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | Subsidiary | No | Subsidiary name. Blank means the user's default subsidiary, so always pass one |
| 2 | Fiscal Year | **Yes** | NetSuite format, e.g. `FY 2026`, or a relative value such as `<Current>` |
| 3 | Period | **Yes** | Period number, 1 = first period of the fiscal year, or a relative value |
| 4 | Period Range | **Yes** | ITD, YTD, RTD, QTD, PTD, or TTD |
| 5 | Account | **Yes** | See below |
| 6 | Book | No | Accounting book name or internal ID; blank is the primary book |
| 7 | Item | No | Item |
| 8 | Class | No | Class name |
| 9 | Department | No | Department name |
| 10 | Location | No | Location name |
| 11 | Entity | No | Customer, vendor, employee, or other entity name |
| 12 | Transaction Type | No | Transaction type |
| 13 | Posted | No | `P` posted (default) or `NP` non-posted |
| 14 | Currency | No | Currency name (default: the subsidiary's currency). A subsidiary converts only into its own currency or a parent subsidiary's currency: NetSuite's consolidated exchange rates run from a subsidiary to its parents. Any other currency returns `Invalid Currency`, or `Error` on older Finsyte versions |
| 15+ | Custom Segments | No | One argument per custom segment configured in Preferences |

**What the Account argument accepts** (*How Finsyte gets a number*):

- A posting account number.
- A summary account (Summary checked in NetSuite): returns every account under it, so don't also add those accounts in the same total.
- A posting parent, an account with children that takes postings itself: returns its own postings only. Its children need rows of their own, or `FSN.GLAccountSummaryBalance` returns the parent with every account under it.
- `FSN_L1_*`, `FSN_L2_*`, `FSN_L3_*`: every account in that category, group, or account type, inactive accounts included.
- `FSN_L0_INC`: the whole income statement. `FSN_L0_BAL` returns "Not Supported".
- `FSN_L4_NIN` net income, `FSN_L4_RET` retained earnings (non-zero only under `YTD`, typed in capitals: any other Period Range text returns 0), `FSN_L4_CBP` cash at the beginning of the period (0 under `ITD`).
- A statistical account: returns its amount, the same as `FSN.StatAccountAmount`, and ignores Book, Item, Transaction Type, Posted, Currency and the custom segments.
- Several accounts or codes as a `^` string, a range, or a named range: returns their total, each account once. A code or a summary account in the list brings in the accounts under it. A posting parent brings in its own postings only. The list takes one period span from its first account, so under `YTD` a list that mixes balance sheet and income statement accounts adds part of itself over the wrong span: keep a list to accounts of one statement. `FSN_L4_NIN`, `FSN_L4_RET` and `FSN_L4_CBP` add nothing in a list, with no message: give each its own cell. `FSN_L0_BAL` in a list brings in every balance sheet account.

**Notes:**

- Values come back exactly as stored: credit-normal accounts (equity, income, other income, all liabilities) are negative. See *Signs*.
- Some accounts are available only to certain subsidiaries; check with `FSN.SubHasAccess` or the SubsidiaryHasAccess column of the GL Accounts list.
- Right-click a result to drill down into its transactions.

### FSN.GLAccountDebitBalance, FSN.GLAccountCreditBalance

The debit side or the credit side only. **Same arguments as `GLAccountBalance`.** Useful for isolating debit-side activity (such as cost analysis) or credit-side activity (such as revenue analysis). The `FSN_L0` and `FSN_L4` codes return `This account number is not supported for this function`.

### FSN.GLAccountName

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | Account | **Yes** | Account number or `FSN_` code |
| 2 | Prefix | No | Text added before the name |
| 3 | Suffix | No | Text added after the name |

```
=FSN.GLAccountName($A10)
=FSN.GLAccountName($A10, "Total - ")
```

### FSN.GLAccountSummaryBalance

An account together with every account under it, at any depth. For a summary account, `FSN.GLAccountBalance` returns the same, so prefer it there. For a posting parent, `FSN.GLAccountBalance` returns the parent's own postings and `FSN.GLAccountSummaryBalance` the parent with every account under it, the accounts NetSuite adds later included. The arguments are identical to `GLAccountBalance`'s.

---

## Budget functions

> **Never negate a budget figure.** The budget functions use no debit/credit logic: budget revenue and a profitable budget net income both come back positive. Negating a budget revenue line turns +$24M into −$24M on a chart. The negation rules in *Signs* apply to actuals only.

### FSN.BudgetAccountBalance

Budget amounts from NetSuite budgets.

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | Subsidiary | No | Subsidiary name |
| 2 | Fiscal Year | **Yes** | NetSuite format |
| 3 | Period | **Yes** | Period number |
| 4 | Period Range | **Yes** | ITD, YTD, RTD, QTD, PTD, TTD |
| 5 | Account | **Yes** | Account number, summary account, `FSN_L1`–`L3` code, `FSN_L4_NIN` on its own, or a list |
| 6 | Budget Category | No | Budget category name. Blank reads the Legacy category, or the company's first category: always fill it in |
| 7 | Book | No | Accounting book (blank is the primary book) |
| 8 | Item | No | Item |
| 9 | Class | No | Class name |
| 10 | Department | No | Department name |
| 11 | Location | No | Location name |
| 12 | Entity | No | Entity name |
| 13 | Reserved | No | Leave blank |

A blank segment returns all values of that segment combined; omitting Location, for example, returns the budget across all locations. With `FSN_L4_NIN` as the account it returns budgeted net income. Right-click a result › Go to Budget opens the budget in NetSuite.

**Lists in a budget.** A `^` list or a range adds the accounts under every parent in it, a posting parent included, each account once. A posting parent on its own returns its own budget only. To budget a posting parent with its children, pass the parent and its children as a list, which also picks up the accounts NetSuite adds under the parent later; the actual beside it is `FSN.GLAccountSummaryBalance` on the parent. The first entry of a list decides how the list is read: `FSN_L4_NIN` first returns budgeted net income and ignores the rest, and `FSN_L0_INC` later in a list adds every income statement budget as stored. Keep `FSN_L4_NIN` and the `FSN_L0` codes out of budget lists.

### FSN.BudgetNetIncome

Budgeted net income: Income and Other Income minus Cost of Goods Sold, Expense, and Other Expense. The function is `FSN.BudgetNetIncome`; `FSN.BudgetAccountNetIncome` doesn't exist and returns `#NAME?`. A blank Budget Category reads the Legacy category, as in `FSN.BudgetAccountBalance`.

| # | Argument | Required |
|---|----------|----------|
| 1 | Subsidiary | No |
| 2 | Fiscal Year | **Yes** |
| 3 | Period | **Yes** |
| 4 | Period Range | **Yes** |
| 5 | Budget Category | No |
| 6 | Book | No |
| 7 | Item | No |
| 8 | Class | No |
| 9 | Department | No |
| 10 | Location | No |
| 11 | Entity | No |
| 12 | Reserved | No |

---

## Statistical and inventory functions

### FSN.StatAccountAmount

The amount of a statistical account in its default unit of measure.

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | Subsidiary | No | Subsidiary name |
| 2 | Fiscal Year | **Yes** | NetSuite format |
| 3 | Period | **Yes** | Period number |
| 4 | Period Range | **Yes** | ITD, YTD, RTD, QTD, PTD, TTD |
| 5 | Account | **Yes** | A statistical account |
| 6 | Reserved | No | Leave blank |
| 7 | Reserved | No | Leave blank |
| 8 | Class | No | Class name |
| 9 | Department | No | Department name |
| 10 | Location | No | Location name |
| 11 | Entity | No | Entity name |

### FSN.GLItemQuantity, FSN.GLItemUnit

The quantity of an item, and its unit of measure, from transaction activity. **Same arguments as `GLAccountBalance` except Currency:** arguments 1–13 match, and custom segments start at the 14th. The unit can differ from the item's default unit if transactions used other units. Drill-down works on the results. One posting account per cell: a list returns `Multi-account not supported for this function` and an `FSN_` code `This account number is not supported for this function`. `FSN.GLItemUnit` returns text.

---

## Period functions

### FSN.DateToPeriod

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | Date | **Yes** | A cell holding a date |
| 2 | Period Type | No | `Year`, `Quarter`, or `Posting` (default) |
| 3 | Subsidiary | No | For subsidiaries with different period structures |

Returns the period's name, or `Not Found`. The Subsidiary is matched by name, with ` (Consolidated)` removed.

### FSN.GetPeriodName

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | Period | **Yes** | Period number |
| 2 | Fiscal Year | **Yes** | e.g. `FY 2026` |
| 3 | Subsidiary | No | For subsidiaries with different period names |

Useful for column headers on period-by-period reports. Past the year's last period it returns `Invalid period number specified`.

### FSN.PeriodToDate

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | Fiscal Year | **Yes** | e.g. `FY 2026` |
| 2 | Period | **Yes** | Period number |
| 3 | Side | No | `Start` or `End` (default) |

Returns a date serial number; format the cell as a date. It has no Subsidiary argument and reads the top-level subsidiary's calendar. Past the year's last period it returns `Incomplete parameters`.

---

## Utility functions

### FSN.GetListInfo, FSN.GetParentListInfo

| # | Argument | Required | Description |
|---|----------|----------|-------------|
| 1 | List Name | **Yes** | A Finsyte list, e.g. `GL Accounts`, as named under Data Retrieval › From List |
| 2 | Identifier | **Yes** | The name or number to look up |
| 3 | Property | **Yes** | The column to return, e.g. `AccountType`, `AccountCategory`, `AccountGroup`, `Parent` |
| 4 | Prefix | No | Text added before the value |
| 5 | Suffix | No | Text added after the value |

`FSN.GetParentListInfo` returns the value from one level up the hierarchy, such as a parent subsidiary or parent account. Its arguments are List Name, Identifier, Property, Level, Prefix, Suffix: Level, currently unused, sits 4th, so Prefix and Suffix are 5th and 6th.

### FSN.SubHasAccess(Subsidiary*, ListName*, Identifier*)

The text `True` or `False` (`False` also for an item it can't find): whether the subsidiary can use the list item, typically a GL account. Compare it as text in a separate cell, such as `=IF(D5="True",…)`.

### FSN.CurrencyName(Subsidiary*, BudgetCategory)

The subsidiary's currency. With a Budget Category that is global, it returns the currency of the top parent subsidiary.

### FSN.LastDataRefresh(Source)

When data was last refreshed: `NetSuite` (default) or `Cache`. Format the cell as a date and time. It shows `Never Refreshed` before the first refresh. It is volatile: it recalculates at every calculation, so keep it out of the inputs Finsyte calls read.

### FSN.Slicer(SlicerName)

The items selected in the workbook's slicer of that name, joined with `^`, for a segment or account argument; every item while nothing is filtered. Excel's function list hides it. It is volatile by design: a slicer click changes no cell, so every Finsyte call that reads it recalculates at each calculation.

---

## Data query functions

All three return tables that spill.

| Function | Arguments |
|---|---|
| `FSN.Query` | SuiteQL query*, Omit Header, Skip, Take. Skip must be a multiple of Take |
| `FSN.SavedSearch` | Search*, Omit Header, Skip, Take. The search's ID (`customsearch_…`) or unique name |
| `FSN.Dataset` | Dataset*, Omit Header, Skip, Take. The dataset's name, or its ID when the name isn't unique |

- `FSN.Query` and `FSN.Dataset` write every cell as text, numbers and dates included, so `SUM` over their output returns 0. Convert a column first: `=SUM(IFERROR(VALUE(INDEX($F$2#,0,3)),0))` totals the third column of a result anchored in F2. `FSN.SavedSearch` returns numbers and dates.
- SuiteQL is Oracle-flavored: use `FETCH FIRST n ROWS ONLY`, not `LIMIT`.
- `FSN.SavedSearch` needs the Finsyte SuiteApp, deployed once by an administrator from Help › Preferences › SuiteApp Deploy. If the function reports it's unavailable, that deployment is the first thing to check. Search IDs are listed under Data Retrieval › From List › Saved Searches, or in NetSuite under Reports › Saved Searches.
- **Registered saved searches.** In Help › Preferences › Saved Search Filters, a company administrator picks a search, chooses fields to filter on, and names the function; Finsyte registers it as `FSN.SavedSearch.<Name>` with one argument per filter. Names and argument orders are specific to each company, so use one only if the user names it or it already appears in the workbook. A blank argument skips that filter; `^` gives several values; a date filter takes a start and an optional end argument, with `-1` as the start meaning since inception and a start alone filtering that one day; a between filter takes two arguments; list values (Subsidiary, Location, Department, Class, Item, Entity, Transaction Type, Currency) are matched by name, and Subsidiary, Location, Department and Class also accept `mine`, `mine and descendants`, `mine and unassigned`, and `mine, descendants and unassigned`. A value from Excel replaces the search's own criterion for that run; the saved search in NetSuite isn't changed. New or edited registrations need an Excel restart, and renaming one breaks formulas that use the old name.

---

## Drilldown functions

The drilldowns of the right-click menu as formulas. Each returns a header row and the detail rows as a table that spills below the cell, refreshes with its parameter cells, and has the columns of the right-click drilldown sheets. After their own leading arguments they take the arguments of `GLAccountBalance`, Subsidiary to Currency, so a balance formula becomes its breakdown by adding the leading arguments at the front.

| Function | Leading arguments | Returns | Custom segments start at |
|---|---|---|---|
| `FSN.TransactionLineDrilldown` | Balance Type: `Balance` (also blank), `Debit` or `Credit` | The transaction lines behind the balance; journal entry lines for a statistical account | 16 |
| `FSN.SegmentDrilldown` | Segment*, then Balance Type: `Balance` (also blank), `Debit`, `Credit` or `Quantity` | One row per value of the segment: `Department`, `Class`, `Location`, `Subsidiary`, `Transaction Type`, `Entity`, `Item`, `Period`, `Account` (the base accounts behind a summary account, `FSN_L4_NIN`, `FSN_L4_RET` or a `^` list), or a custom segment's name | 17 |
| `FSN.AgingSummary` | None | The A/R or A/P aging as of the period's end, one row per customer or vendor and a column per aging bucket; the Period Range is ignored | 15 |

With Subsidiary, Fiscal Year, Period and Period Range in `$B$2` to `$B$5` and the account in `$B$6`:

```
=FSN.TransactionLineDrilldown(,$B$2,$B$3,$B$4,$B$5,$B$6)
=FSN.SegmentDrilldown($B$7,,$B$2,$B$3,$B$4,$B$5,$B$6)      ← $B$7 = Department
=FSN.AgingSummary($B$2,$B$3,$B$4,$B$5,$B$8)                ← $B$8 = an A/R or A/P account
```

- Amounts are unsigned, as on the right-click drilldown sheets, and Switch Sign doesn't apply.
- `FSN.AgingSummary` needs Accounts Receivable or Accounts Payable accounts, all of one type; any other account returns `FSN.AgingSummary needs an accounts receivable or accounts payable account.` `FSN.SegmentDrilldown` by `Account` with one base account returns `Account needs a summary account or a list of accounts.`
- Leave room below each formula: the spill grows with the data, and a `#SPILL!` means something is in the way. Only the first drilldown result on a sheet gets filter buttons.
- An argument that shows an Excel error returns that error. A drilldown cancelled or timed out shows `Not loaded: …`; right-click the formula cell › Refresh Selection loads it again. Finsyte warns above 50,000 lines (Help › Preferences › Drilldown Options) and still loads every line.
- Pass a range as a reference: a workbook name typed as text isn't expanded.
- Help › Preferences › Drilldown Options sets the columns of `FSN.TransactionLineDrilldown`, and Aging Options the aging buckets.

---

## Range functions

Range functions return lists that spill down, or across when Horizontal is TRUE. Use them for account rows, column headers, dropdown sources, and checks. An empty result is one blank cell. Their structural switches (Horizontal, Posting Only, Include Rollup, and so on) can be typed as `TRUE` or `FALSE` in the formula; filters and Subsidiary go in parameter cells as usual.

### Filter syntax

The Include and Exclude filters of every Range function accept:

| Pattern | Example | Meaning |
|---|---|---|
| `^` | `1000^1010` | Several values |
| `-` | `1000-2000` | A range of values |
| `?` | `10?0` | Any single character |
| `*` | `1*` | Anything starting with 1 |
| An `FSN_` code | `FSN_L3_BNK` | In the Include Filter, that rollup, plus the accounts under it with Include Children TRUE. In the Exclude Filter, the rollup entry only |

A cell range or named range also works, and each cell in it can hold any of these patterns.

The Exclude Filter removes only the values it names. Include Children adds the accounts under an Include Filter match only, so a code or a summary account in the Exclude Filter leaves the accounts under it in the result. To leave those accounts out, keep them out of the Include Filter (*Keep reports complete*).

### FSN.Range.AccountNumbers, FSN.Range.Accounts

`AccountNumbers` returns account numbers; `Accounts` returns accounts as they appear in Finsyte's account dropdown. Same arguments:

| # | Argument | Default | Description |
|---|---|---|---|
| 1 | Include Filter | All | Accounts to include |
| 2 | Exclude Filter | None | Accounts to leave out |
| 3 | Subsidiary | All | Limit to one subsidiary's accounts |
| 4 | Level | All | Deepest level to return. Rollups are levels 0–3; top-level GL accounts are level 4, their children 5, 6, and so on |
| 5 | Include Children | **TRUE** | Also return every account beneath each match |
| 6 | Include System Generated | FALSE | Include the `FSN_` rollups in the result |
| 7 | Include Inactive | FALSE | Include inactive posting accounts (summary accounts are always kept) |
| 8 | Horizontal | FALSE | Spill across instead of down |
| 9 | Posting Only | FALSE | Return only posting accounts, dropping summary (parent) accounts |

**How a rollup filter expands.** `FSN_L0_BAL` on its own matches just the rollup. Include Children, on by default, brings in everything beneath it, and Include System Generated, off by default, then removes the `FSN_` rollups from the result. The defaults together return all balance sheet accounts.

**Summary accounts double-count in a flat list.** The default result contains summary accounts and the accounts under them, and a summary account's balance already includes those accounts. For a flat list totaled with `AGGREGATE`, set Posting Only to TRUE. Posting parents stay in the result and return their own postings only. The numbers come sorted as text.

```
=FSN.Range.AccountNumbers($B$6)                    ← $B$6 = FSN_L3_BNK: all Bank accounts, summary accounts included
=FSN.Range.AccountNumbers($B$6,,$B$2,,,,,,TRUE)    ← $B$6 = FSN_L0_BAL: balance sheet posting accounts for one subsidiary
=FSN.Range.AccountNumbers(,,,0,,TRUE)              ← only the level-0 rollups (FSN_L0_BAL, FSN_L0_INC, …)
```

### Segment and entity lists

`FSN.Range.Departments`, `.Classifications`, `.Locations`, `.Items`, `.Entities`, `.Customers`, `.Vendors`, `.Employees`, `.Projects`, `.ProjectTemplates`, `.Companies`, `.Contacts`, `.Partners`, `.Groups`, `.GenericResources`, and `.OtherNames` share one argument order, different from `AccountNumbers`:

| # | Argument | Default | Description |
|---|---|---|---|
| 1 | Include Filter | All | Values to include |
| 2 | Exclude Filter | None | Values to leave out |
| 3 | Subsidiary | All | Limit to one subsidiary's values |
| 4 | Include Blank | **TRUE** | Add the "no value" entry, such as `<No Department>`, which selects transactions with no value set |
| 5 | Level | All | Deepest hierarchy level to return |
| 6 | Include Inactive | Preference | Include inactive values. Blank follows the General preference that shows inactive items in dropdowns, off by default; type `FALSE` to keep inactive values out for every user |
| 7 | Include Rollup | **TRUE** | Add a `Parent (Hierarchy)` entry for each parent, which selects the parent and all its children |
| 8 | Rollup Only | FALSE | Return only the `(Hierarchy)` entries |
| 9 | Horizontal | FALSE | Spill across instead of down |
| 10 | Rollup First | **TRUE** | List each `(Hierarchy)` entry before its children |

For one column or row per value, set Include Rollup to FALSE, or the `(Hierarchy)` entries overlap their children and the total counts them twice. The class list is `FSN.Range.Classifications`. The Include Filter of an entity list matches the entity's ID and name, and a matching parent also adds its `(Hierarchy)` entry: set Include Rollup to FALSE for the entities alone.

**Other lists:**

- `FSN.Range.Subsidiaries` has no Subsidiary or Include Blank argument: Include, Exclude, Level, Include Inactive, Include Rollup, Rollup Only, Horizontal, Rollup First. It includes consolidated entries such as `Headquarters (Consolidated)`. Its Include Inactive works as in the segment lists.
- `FSN.Range.CustomSegments` takes the custom segment's name first, then the same ten arguments as the segment lists.
- `FSN.Range.BudgetCategories(Horizontal)` lists the active budget categories.

### Lists for dropdowns

These lists feed Finsyte's own dropdowns. Excel's function list hides them, and they work in any formula, so a workbook can build its own dropdowns from them (`references/model-building.md`, Dropdowns). Each takes Horizontal first.

| Function | Returns |
|---|---|
| `FSN.Range.Periods.FY(Horizontal, Ascending)` | The fiscal year names; `FSN.Range.Periods.FY(FALSE,FALSE)` lists the latest first, as Finsyte's Fiscal Year dropdown does |
| `FSN.Range.PeriodNumbers(Horizontal, Ascending)` | The period numbers |
| `FSN.Range.PeriodRanges(Horizontal)` | `ITD`, `YTD`, `RTD`, `QTD`, `PTD`, `TTD` |
| `FSN.Range.AccountingBooks(Horizontal)` | The accounting book names, the primary book first |
| `FSN.Range.Currencies(Horizontal)` | The active currency names; one blank cell for a NetSuite account with one currency |
| `FSN.Range.TransactionTypes(Horizontal, Ascending)` | The transaction type names |

The period lists follow the top-level subsidiary's calendar. With the Relative Periods preference on, `<Current>` and the other relative values follow the fiscal years and the period numbers in these lists.

---

## Older functions to avoid

These still calculate. All but `FSN.CashAtBeginningOfPeriod` are hidden from Excel's function list. Write the `FSN.GLAccountBalance` form instead, which keeps every balance cell in the same shape:

| Older function | Use instead |
|---|---|
| `FSN.GLAccountNetIncome(…)` | `FSN.GLAccountBalance(…, FSN_L4_NIN)` |
| `FSN.GLAccountRetainedEarnings(…)` | `FSN.GLAccountBalance(…, FSN_L4_RET)` with Period Range `YTD` |
| `FSN.GLAccountTypeBalance(…, "Bank")` | `FSN.GLAccountBalance(…, FSN_L3_BNK)` |
| `FSN.GLAccountGroupBalance(…, "Current Assets")` | `FSN.GLAccountBalance(…, FSN_L2_CAS)` |
| `FSN.GLAccountCategoryBalance(…, "Assets")` | `FSN.GLAccountBalance(…, FSN_L1_AST)` |
| `FSN.CashAtBeginningOfPeriod(…)` | `FSN.GLAccountBalance(…, FSN_L4_CBP)` |

One of them that already appears in a user's workbook works and needs no rewrite; rewrite it at the user's request. `FSN.GLAccountNetIncome` and `FSN.GLAccountRetainedEarnings` take fewer arguments than `GLAccountBalance`, so their custom segment arguments start at the 13th argument (`GLAccountNetIncome`) and the 12th (`GLAccountRetainedEarnings`), against the 15th for `GLAccountBalance`.

---

## Period range examples

| Range | Setup | Start | End |
|-------|-------|-------|-----|
| ITD | FY 2024 period 7, company inception Jan 2000 | Jan 2000 | Jul 2024 |
| YTD | FY 2023 period 2, calendar fiscal year | Income statement: Jan 2023; balance sheet: Jan 2000 | Feb 2023 |
| RTD | FY 2024 period 6, calendar fiscal year | Jan 2024 | Jun 2024 |
| QTD | FY 2022 period 11, calendar fiscal year | Oct 2022 | Nov 2022 |
| PTD | FY 2024 period 8, calendar fiscal year | Aug 2024 | Aug 2024 |
| TTD | FY 2024 period 10, 12 trailing periods | Nov 2023 | Oct 2024 |

---

## Account type sign table

Every NetSuite account type, its category, its normal balance, and the sign Finsyte returns (values come back exactly as stored).

| Account Type | Category | Normal balance | Sign returned |
|---|---|---|---|
| Accounts Receivable | Asset | Debit | + |
| Bank | Asset | Debit | + |
| Deferred Expense | Asset | Debit | + |
| Fixed Asset | Asset | Debit | + |
| Other Asset | Asset | Debit | + |
| Other Current Asset | Asset | Debit | + |
| Unbilled Receivable | Asset | Debit | + |
| Equity | Equity | Credit | − |
| Cost of Goods Sold | Expense | Debit | + |
| Expense | Expense | Debit | + |
| Other Expense | Expense | Debit | + |
| Income | Income | Credit | − |
| Other Income | Income | Credit | − |
| Accounts Payable | Liability | Credit | − |
| Credit Card | Liability | Credit | − |
| Deferred Revenue | Liability | Credit | − |
| Long Term Liability | Liability | Credit | − |
| Other Current Liability | Liability | Credit | − |

The Statistical account type is non-monetary and isn't in this table.

---

## System-generated account tree

Finsyte adds these rollup accounts to the GL Accounts list. Codes are `FSN_L<level>_<suffix>`: level 0 is a whole statement and level 3 is a NetSuite account type. Every code here is real; don't build others by analogy (there is no `FSN_L1_INC` or `FSN_L1_EXP`).

| Code | Name | Level | Parent |
|---|---|---|---|
| `FSN_L0_BAL` | Balance Sheet | 0 | — |
| `FSN_L1_AST` | Assets | 1 | `FSN_L0_BAL` |
| `FSN_L2_CAS` | Current Assets | 2 | `FSN_L1_AST` |
| `FSN_L3_BNK` | Bank | 3 | `FSN_L2_CAS` |
| `FSN_L3_ARE` | Accounts Receivable | 3 | `FSN_L2_CAS` |
| `FSN_L3_URE` | Unbilled Receivable | 3 | `FSN_L2_CAS` |
| `FSN_L3_OCA` | Other Current Asset | 3 | `FSN_L2_CAS` |
| `FSN_L3_DEX` | Deferred Expense | 3 | `FSN_L2_CAS` |
| `FSN_L2_LTA` | Long Term Assets | 2 | `FSN_L1_AST` |
| `FSN_L3_FAS` | Fixed Asset | 3 | `FSN_L2_LTA` |
| `FSN_L3_LTO` | Other Asset | 3 | `FSN_L2_LTA` |
| `FSN_L1_LEQ` | Liabilities & Equity | 1 | `FSN_L0_BAL` |
| `FSN_L2_CLS` | Current Liabilities | 2 | `FSN_L1_LEQ` |
| `FSN_L3_APL` | Accounts Payable | 3 | `FSN_L2_CLS` |
| `FSN_L3_CCX` | Credit Card | 3 | `FSN_L2_CLS` |
| `FSN_L3_OCL` | Other Current Liability | 3 | `FSN_L2_CLS` |
| `FSN_L3_DRE` | Deferred Revenue | 3 | `FSN_L2_CLS` |
| `FSN_L2_LTL` | Long Term Liabilities | 2 | `FSN_L1_LEQ` |
| `FSN_L3_LTL` | Long Term Liability | 3 | `FSN_L2_LTL` |
| `FSN_L2_EQU` | Equity | 2 | `FSN_L1_LEQ` |
| `FSN_L3_EQU` | Equity | 3 | `FSN_L2_EQU` |
| `FSN_L0_INC` | Income Statement | 0 | — |
| `FSN_L1_ORD` | Ordinary Income/Expense | 1 | `FSN_L0_INC` |
| `FSN_L2_REV` | Revenue | 2 | `FSN_L1_ORD` |
| `FSN_L3_INC` | Income | 3 | `FSN_L2_REV` |
| `FSN_L2_EXP` | Expense | 2 | `FSN_L1_ORD` |
| `FSN_L3_COG` | Cost of Goods Sold | 3 | `FSN_L2_EXP` |
| `FSN_L3_EXP` | Expense | 3 | `FSN_L2_EXP` |
| `FSN_L1_OTH` | Other Income/Expense | 1 | `FSN_L0_INC` |
| `FSN_L2_OIN` | Other Income | 2 | `FSN_L1_OTH` |
| `FSN_L3_OIN` | Other Income | 3 | `FSN_L2_OIN` |
| `FSN_L2_OEX` | Other Expense | 2 | `FSN_L1_OTH` |
| `FSN_L3_OEX` | Other Expense | 3 | `FSN_L2_OEX` |
| `FSN_L0_STA` → `FSN_L1_STA` → `FSN_L2_STA` → `FSN_L3_STA` | Stat | 0–3 | chain |
| `FSN_L0_NPG` → `FSN_L1_NPG` → `FSN_L2_NPG` → `FSN_L3_NPG` | Non Posting | 0–3 | chain |

**Calculated accounts (level 4, no parent).** These are calculations: in a multi-value argument they add nothing, with no message, so each gets its own cell:

| Code | Name |
|---|---|
| `FSN_L4_NIN` | Net Income |
| `FSN_L4_RET` | Retained Earnings |
| `FSN_L4_CBP` | Cash at Beginning of Period |

---

## Messages

A Finsyte cell shows a message in place of a value, and the minus of a negated cell turns the message into `#VALUE!`: Formulas › Evaluate Formula on the cell shows the message. Messages follow Office's display language; the English text is below.

| Message | Cause | Fix |
|---|---|---|
| `Invalid account number`, `Invalid account number <n> specified` | An account that doesn't exist, often a typo or an account of another company | Take the number from `FSN.Range.AccountNumbers` or the Account dropdown |
| `Invalid period number` | The Period is neither a number nor a relative period such as `<Current>` | Type the period number |
| `Invalid period number specified` | Period 0, a period past the year's last one, or a Period cell that shows an Excel error | Fix the Period cell |
| `Invalid period range` | A Period Range other than `ITD`, `YTD`, `RTD`, `QTD`, `PTD` and `TTD` | Pick it from the Period Range dropdown |
| `Invalid Book Name` | No accounting book has that name | Pick it from the Accounting Book dropdown |
| `Invalid budget category '<name>'` | No budget category has that name | Pick it from the Budget Category dropdown |
| `Invalid Currency` | A currency the subsidiary's amounts can't convert into: a child's or a sibling's currency | The subsidiary's own currency or a parent subsidiary's currency |
| `Subsidiary <name> not found`; `Department <name> not found`, and the same for Class, Location, Transaction Type and a custom segment; `Entity '<name>' not found`; `Item '<name>' not found` | A value the list doesn't have: a typo, an inactive value, or a value of another subsidiary | Pick the value from its dropdown |
| `<Argument> is <error>`, such as `Account is #N/A`; `<Argument> range contains <error>` | An input cell, or a cell of an input range, shows an Excel error | Fix the input cell |
| `Missing account number` | A blank Account; the budget, debit and credit functions also show it for an Account input that shows an Excel error | Fill in or fix the Account cell |
| `This account number is not supported for this function` | A code the function refuses, such as `FSN_L4_RET` in a budget or an `FSN_L0` code in a debit or credit call | An account or a code the function accepts |
| `Multi-account not supported for this function` | A list in `FSN.GLItemQuantity` or `FSN.GLItemUnit` | One account per cell |
| `Not Supported` | `FSN_L0_BAL` as the whole Account | `FSN_L1_AST` and `FSN_L1_LEQ` |
| `No default subsidiary was found…` | A blank Subsidiary, and Finsyte holds no default subsidiary for the user | Fill in Subsidiary, then Refresh Data › Entire Workbook |
| `Failed to Initialize`, `Initialization Error` | Finsyte isn't connected to NetSuite: signed out, or the connection failed | Sign in, then Refresh Data › Entire Workbook |
| `Error` | The call couldn't run: a failed connection, or on older Finsyte versions a currency the amounts can't convert into | Check the sign-in and the Currency, then Refresh Data › Entire Workbook |
| `Not loaded: the drilldown was cancelled or timed out…` | A drilldown function stopped before all its lines loaded | Right-click the formula cell › Refresh Selection |
| `Saved search not found`, `Dataset not found` | No saved search or dataset has that ID or name | Take the ID from Data Retrieval › From List › Saved Searches or Datasets |
| `This custom function isn't available for the connected company.` | A registered saved search function of another company | Switch Company to the company set as Use for Custom Features |

Empty text in a numeric cell means Finsyte is signed out, and zeros everywhere can mean a paused Finsyte (SKILL.md, Gotchas).
