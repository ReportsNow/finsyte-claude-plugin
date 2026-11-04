# Statements

Read this before you write the rows of a report page. Every balance sheet page shows one balance sheet tree, every operations page one operations tree, and every cash flow page one cash flow tree, built once from the line plan and repeated on each page of that statement. The columns are in `references/pages.md`.

## Contents

- The line plan
- Row kinds
- Signs
- Completeness: lines, plug lines and rollup rows
- Balance sheet
- Operations
- Cash flow
- Checks
- Row formats

---

## The line plan

The line plan says which accounts each line of a statement holds. You draft it from the chart of accounts (`references/excel-mechanics.md`, "Discovery") and the user confirms it in the interview (`references/interview.md`, round 3). Write it down as a list before you build: every page of a statement uses the same plan.

A line is one of:

| Line | What it holds | On the page |
|---|---|---|
| Single line | One account, or one `FSN_` code, with no account rows below it | One row with the account in B |
| Group | Several accounts: a NetSuite parent and the accounts under it, the accounts of a type, or a set the user names | A header row, then one hidden detail row per posting account, then a hidden "Total ..." row. Collapsed, the header shows the total. A group of a whole type also has a hidden plug row, `Other <caption in lowercase>`, above its total (see "Completeness"). |
| Group of lines | Lines that roll up under one caption, such as Cost of sales or an operating expense parent | A header row, then the lines (single lines, groups, a plug line) one outline level down, then a hidden "Total ..." row |
| Plug line | "Other ..." : a rollup minus the lines beside it | One row of arithmetic while no account sits in it. When the line plan leaves accounts in it, it is an Other group (see "Completeness"). |
| Other group | A plug line that holds accounts when the book is built, such as Other current assets | A header row, one hidden detail row per account it holds, a hidden `Accounts not listed` plug row, then a hidden "Total ..." row |
| Calculated line | Arithmetic on other rows, such as Gross profit or Retained earnings | One row of arithmetic |

**Detail rows** are the posting accounts of the group in NetSuite's order: a summary account is never a detail row, its posting children are. A posting parent (an account with children that is not a summary account) is a detail row of its own, ahead of its children. Two outline levels below a visible line is the deepest: accounts deeper than that are listed at the second level.

**An account appears on one line of a statement only.** An account that two lines could claim, such as a depreciation account under a general and administrative parent, belongs to the line the user picked, and the other line leaves it out (see "Completeness").

**Captions** of groups and lines are typed: the user's wording, or the NetSuite name in sentence case. Detail rows take the NetSuite name with `=FSN.GLAccountName($B<r>)`, so a renamed account updates itself.

---

## Row kinds

`<c>` is any amount column, value or change. Columns A and B are as in `references/pages.md`, "Page frame".

| Kind | A | B | C | Amount columns |
|---|---|---|---|---|
| Caption | `1` | | Text, bold | Empty |
| Blank | `1` | | | Empty |
| Single line | Level formula | Account or code | Typed caption | Value columns: the value formula with the row's sign. Change columns: the change formula. |
| Group header | Level formula, or `1` with no account | The parent account, the type's `FSN_` code, or empty | Typed caption | Every column: `=IF(SUBTOTAL(103,$A<first>:$A<last>)=0,<c><total>,"")`, where `<first>` to `<last>` are the group's hidden rows, its total row included, and `<total>` is the group's total row |
| Detail | Level formula | Account | `=FSN.GLAccountName($B<r>)` | As a single line |
| Group total | `1` | | `Total ` and the caption in lowercase, bold | Value columns: `=AGGREGATE(9,0,<c><first detail>:<c><last row above>)`. Change columns: the change formula. |
| Subtotal | `1` | | Typed, bold | Value columns: `=AGGREGATE(9,0,<c><first row of the section>:<c><row above>)`. Change columns: the change formula. |
| Plug line, calculated line | `1` | | Typed | Value columns: the arithmetic. Change columns: the change formula. |
| Ratio (KPI) | `1` | | Typed | Value columns: `=IFERROR(<c><numerator>/<c><denominator>,0)`. Change columns: empty. |
| Amount KPI | `1` | | Typed | Value columns: the arithmetic. Change columns: the change formula. |

The Level formula is `=FSN.GetListInfo($B$20,$B<r>,$A$20)`, on a row whose B holds one account or one code. A row whose B holds several accounts joined with `^` takes `1`.

**Why the totals add up.** `AGGREGATE(9,0,…)` leaves out every cell whose formula holds `SUBTOTAL` or `AGGREGATE`. A group header holds `SUBTOTAL` and a total holds `AGGREGATE`, so a subtotal over a whole section counts each detail row, single line, plug line and calculated line once. A line is never an `AGGREGATE` itself, or the totals above it leave it out.

**Dynamic captions.** A row the user reads as income or loss takes a caption from the signs of its value cells: `=IF(MIN(<value cells>)>=0,"<positive>",IF(MAX(<value cells>)<=0,"<negative>","<mixed>"))`, with every value column of the row listed, such as `F143,G143,K143,L143,P143,Q143`. The captions are in the trees below, written `positive / negative / mixed`.

**Codes in column B.** A rollup row, and any line whose accounts are a type, holds its `FSN_` code in B and reads it with the Actual formula, as Finsyte's templates do, or the "Actual, balance range" formula where the row says so. A code returns every account of its rollup, inactive accounts included. Several codes or accounts joined with `^` return their total, each account once, and a summary account returns its children. A posting parent (an account with children that is not a summary account) returns its own postings only, so its children are rows of their own. No row reads an account list.

---

## Signs

Finsyte returns balances as NetSuite stores them: debit-normal accounts positive, credit-normal accounts negative. A leading minus on the value formula reverses a row. Budgets are never reversed.

| Statement | Reversed (leading `-`) on actual columns | As returned |
|---|---|---|
| Balance sheet | Liability and equity accounts; the net income line; the `FSN_L0_INC` and `FSN_L4_RET` rollups; the `FSN_L3_OCL` and `FSN_L3_EQU` rollups | Asset accounts; the affiliate lines; the `FSN_L2_CAS`, `FSN_L1_AST` and `FSN_L1_LEQ` rollups and the intercompany rollups |
| Operations | Income and Other Income accounts, contra-revenue accounts included; the `FSN_L3_INC`, `FSN_L3_OIN` and `FSN_L4_NIN` rollups | Cost of Goods Sold, Expense and Other Expense accounts and their rollups |
| Cash flow | Every line of balance sheet accounts (an increase in an asset is cash used, an increase in a liability cash provided); the net income line; the change rollups of non-cash current assets, intercompany accounts, property and equipment and posted equity; the revenue rollup | Depreciation and amortization; the bank rollup; Cash at end of period |

---

## Completeness: lines, plug lines and rollup rows

The book stays complete as NetSuite changes: every total ties to a NetSuite rollup, and an account added in NetSuite later lands in an "Other" plug line without any change to the book.

- **Plug line.** `Other <x> = <rollup row> − (<the lines beside it>)`. Subtract each line by its total row, or by its own row for a single line. When a line beside the plug holds accounts outside the plug's rollup, such as the affiliate group, subtract only the part inside the rollup: add a rollup row that holds those accounts in B and subtract it.
- **Other group.** A plug line that holds accounts when the book is built (accounts of its rollup that no other line lists, such as the Other Current Asset accounts outside the named lines) lists them: a header row with the plug's caption and no account in B, one hidden detail row per account in NetSuite's order, then a hidden plug row `Accounts not listed` = `<rollup row> − (<the lines beside it>) − SUM(<first detail>:<last detail>)`, then the hidden total. The plug row reads 0 until NetSuite adds an account to the rollup. This applies to every Other line of the trees except Other equity on the balance sheet and Other income (expense), net, which name their accounts in B. Other equity on the cash flow is an Other group: it lists the retained earnings and translation adjustment accounts. With every account on a row, the missing-accounts check passes until NetSuite adds an account.
- **Group plug row.** A group of a whole type (such as Bank, Fixed Asset or Accounts Payable) ends with a hidden plug row above its total: `Other <caption in lowercase> = <the group's rollup row> − SUM(<first detail>:<last detail>)`. It reads 0 until NetSuite adds an account to the type, and then it carries that account until the user asks for it to be listed. In a group with sub-groups, subtract each sub-group's total row and each detail row by name in place of the `SUM`: a `SUM` over a header row would count it. The group's rollup row has the header's B, the type's code, and uses the same formula as the group's detail rows (the balance range for Cash at end of period). Accounts of the type that sit on another line, such as a lease liability shown on its own line or the type's intercompany accounts on Due from (to) affiliates, leave the plug row through a subtracted rollup row that holds them in B.
- **Groups of a parent** list the parent and its children as rows, as Finsyte's templates do, and have no plug row; neither does a group of accounts the user picked one by one. The section's plug line catches what they leave out, an account that NetSuite adds under the parent later included, and the missing-accounts check lists that account until the user lists it.
- **A whole type is always a group.** Credit card, Deferred revenue and any other line that holds a whole type is a group with its plug row, even with one account today, so an account NetSuite adds later still lands on the line.
- **Intercompany accounts and Undeposited Funds** leave a rollup the same way, through a subtracted rollup row: the intercompany accounts of the rollup's types, joined with `^` from discovery, and the Undeposited Funds account counted as cash. The Inputs cells `IntercompanyAccounts` and `CashExtraAccounts` hold all of them (`references/front-pages.md`).
- **Rollup rows** sit in the checks block below the footer and are never printed. Each one has its code or accounts in B and a caption that names them. In a budget column a rollup row uses the Budget formula, never reversed, with the same B; `FSN_L4_NIN` returns budgeted net income there.
- **Net income and retained earnings** read Finsyte's special accounts and the income statement accounts as codes. Net income on the balance sheet and cash flow pages reads `FSN_L4_NIN` in every column: Finsyte applies the column's subsidiary, department, class and location to it as to any account. Retained earnings is computed from `FSN_L0_INC` ("Balance sheet"). Check rows tie net income to `FSN_L0_INC` on the balance sheet and cash flow pages and to `FSN_L4_NIN` on the operations pages, and retained earnings to `FSN_L4_RET` ("Checks").

---

## Balance sheet

Pages 1, 4, 5, 11, 14, 17 and 20. Indents: captions 0, lines 1, a line's detail or sub-lines 2, detail under a sub-line 3.

Intercompany accounts sit on Due from (to) affiliates only, whatever their type: every other line and group leaves them out.

| Caption | Kind | Accounts or formula | Notes |
|---|---|---|---|
| `Current assets:` | Caption | | |
| Cash and cash equivalents | Group | Bank accounts, plus Undeposited Funds when the user counts it as cash. B of the header: `FSN_L3_BNK`, or `FSN_L3_BNK^<Undeposited Funds account>` when it counts as cash. | $ format |
| Accounts receivable, net | Group | Accounts Receivable and Unbilled Receivable accounts, intercompany accounts left out. B: `FSN_L3_ARE^FSN_L3_URE`. | |
| Inventories | Group | The inventory accounts the user confirms | Left out when there are none |
| Prepaid expenses | Group | The accounts the user confirms | Left out when the user names none |
| Due from (to) affiliates | Group of affiliate lines | One hidden row per entry of `Affiliates`: B the intercompany accounts joined with `^`; C `=INDEX(Affiliates,<k>)`; value cells the Actual formula on B with `$C<r>` as the Entity argument (the 11th) in place of `<c>$15`. Then `Other entities` = [intercompany accounts, all entities] − the affiliate rows. Then the total. Without entity names (round 3), a group of the intercompany accounts instead, one detail row per account. | Only with intercompany accounts. Holds the receivable and the payable side, net, as returned. |
| Other named current-asset lines | Group or single line | As the user names them | Optional |
| Other current assets | Plug | [`FSN_L2_CAS`] − (cash, receivables, inventories, prepaid, named lines) − [intercompany accounts in current assets] | |
| Total current assets | Subtotal | From the first line of the section | Bold, top border, $ |
| Property and equipment, net | Group | Fixed Asset accounts. B: `FSN_L3_FAS`. | |
| Other assets | Group | Other Asset accounts. B: `FSN_L3_LTO`. | |
| Total assets | Subtotal | From the first line under Current assets | Bold, top and double bottom border, $ |
| `Current liabilities:` | Caption | | |
| Accounts payable | Group | Accounts Payable accounts, intercompany accounts left out. B: `FSN_L3_APL`. | $ format |
| Named liability lines | Single line or group | As the user names them, such as lease liabilities | Optional |
| Credit card | Group | Credit Card accounts. B: `FSN_L3_CCX`. | Left out when there are none |
| Deferred revenue | Group | Deferred Revenue accounts. B: `FSN_L3_DRE`. | Left out when there are none |
| Accrued and other current liabilities | Group of lines | One group per Other Current Liability parent the user keeps (such as accrued payroll, accrued liabilities, short-term debt), then `Other current liabilities` = [`FSN_L3_OCL`, reversed] − (those groups, the named lines of this type) − [intercompany accounts in other current liabilities, reversed], then the total | |
| Total current liabilities | Subtotal | | Bold, top border, $ |
| Long-term liabilities | Group | Long Term Liability accounts, intercompany accounts left out. B: `FSN_L3_LTL`. | |
| Total liabilities | Subtotal | From Accounts payable | Bold, top border, $ |
| `Equity:` | Caption | | |
| Paid-in capital and owners' accounts | Group | The Equity accounts the user keeps (default: every posting Equity account except the retained earnings and translation adjustment special accounts) | |
| Retained earnings | Calculated | [`FSN_L0_INC` at the balance range, as a code, reversed] − Net income | |
| Net income | Single line, dynamic | B `FSN_L4_NIN`; the Actual formula, reversed | `Net income / Net loss / Net income (loss)` |
| Cumulative translation adjustment | Calculated | [`FSN_L1_AST`] + [`FSN_L1_LEQ`] − Retained earnings − Net income | Both rollups as returned. On pages 14 to 22 the caption reads `Translation adjustment and balances not assigned to a segment`: a single department, class or location doesn't balance, and the difference lands here. |
| Other equity | Plug | [`FSN_L3_EQU`, reversed] − Paid-in capital. B: the retained earnings and translation adjustment special accounts, joined with `^`, so the missing-accounts check counts them as placed. | |
| Total equity | Subtotal | From Paid-in capital | Bold, top border |
| Total liabilities and equity | Subtotal | From Accounts payable | Bold, top and double bottom border, $ |
| (blank) | Blank | | |
| `Key ratios:` | Caption | | |
| Working capital | Amount KPI | Total current assets − Total current liabilities | $ format |
| Current ratio | Ratio | Total current assets / Total current liabilities | Ratio format |
| Liabilities-to-equity | Ratio | Total liabilities / Total equity | Ratio format |
| Equity ratio | Ratio | Total equity / Total assets | Percent |
| Cash as % of total assets | Ratio | Cash total / Total assets | Percent |

The KPI rows are hidden at outline level 1 under `Key ratios:`.

**Rollup rows (balance sheet):** one per group with a plug row; for each such group whose type holds intercompany accounts (receivables, payables, long-term liabilities), those accounts, with the group's sign; `FSN_L2_CAS`; intercompany accounts, all entities; intercompany accounts in current assets; intercompany accounts in other current liabilities (reversed); intercompany liability accounts (as returned); `FSN_L3_OCL` (reversed); `FSN_L3_EQU` (reversed); `FSN_L1_AST`; `FSN_L1_LEQ`; `FSN_L0_INC` at the balance range, as a code (reversed), for Retained earnings; `FSN_L0_INC` as a code (reversed), for the net income check; `FSN_L4_RET` (reversed), for the retained earnings check. Leave out the intercompany rows when the book has no intercompany accounts.

**Where Retained earnings reads the balance range.** Its rollup row reads `FSN_L0_INC` with the "Actual, balance range" formula: on the month-end and year-end columns row 19 holds `ITD`, so the income statement accounts since inception, less this year's net income, give retained earnings. On page 5 row 19 holds the period's own range, and retained earnings shows the change.

---

## Operations

Pages 2, 6, 7, 8, 12, 15, 18, 21 and 23 to 26. Indents: captions 0, lines 1, sub-lines 2, detail 3.

| Caption | Kind | Accounts or formula | Notes |
|---|---|---|---|
| `Revenues:` | Caption | | |
| Revenue lines | Single lines and groups | Each top-level Income account or parent the user keeps; a parent is a group | The first one $ |
| Other revenue | Plug | [`FSN_L3_INC`, reversed] − (the revenue lines) | |
| Total revenues | Subtotal | From the first revenue line | Bold, top border, $ |
| Cost of sales | Group of lines | The Cost of Goods Sold lines the user keeps, then `Other cost of sales` = [`FSN_L3_COG`] − (those lines), then the total | |
| Gross profit | Calculated | Total revenues − Total cost of sales | Bold, top border, $ |
| `Operating expenses:` | Caption | | |
| Expense groups | Group of lines, one per top-level Expense parent the user keeps | The parent's children as lines (a child parent is a group of its own), then `Other <caption in lowercase>`: a group with the parent's own row (a posting parent only) and the children the user folds, one detail row each, then the total. Depreciation and amortization accounts under the parent stay on their own line. | Caption: the user's, such as "Selling and marketing" |
| Single expense lines | Single line or group | Top-level Expense accounts the user keeps | Optional |
| Depreciation and amortization | Group | The accounts the user confirms | Left out when there are none |
| Other operating expenses | Plug | [`FSN_L3_EXP`] − (the expense groups, the single expense lines, Depreciation and amortization) | |
| Total operating expenses | Subtotal | From the first expense line | Bold, top border |
| Operating income | Calculated, dynamic | Gross profit − Total operating expenses | `Operating income / Operating loss / Operating income (loss)`. Bold, $ |
| Other income (expense), net | Calculated, dynamic | [`FSN_L3_OIN`, reversed] − [`FSN_L3_OEX`]. B: the Other Income and Other Expense accounts, joined with `^` from discovery, so the missing-accounts check counts them as placed ("Missing accounts"). | `Other income, net / Other expense, net / Other income (expense), net` |
| Net income | Dynamic | `=AGGREGATE(9,0,<Operating income>:<Other income (expense), net>)` | `Net income / Net loss / Net income (loss)`. Bold, top and double bottom border, $ |
| (blank) | Blank | | |
| `Key performance indicators:` | Caption | | |
| Gross margin | Ratio | Gross profit / Total revenues | Percent |
| Operating margin | Ratio | Operating income / Total revenues | Percent |
| Net margin | Ratio | Net income / Total revenues | Percent |
| Cost of sales % of revenue | Ratio | Total cost of sales / Total revenues | Percent |
| `<expense group>` % of revenue | Ratio | The group's total / Total revenues, one row per expense group | Percent |
| Total operating expenses % of revenue | Ratio | Total operating expenses / Total revenues | Percent |

A line with no accounts for this subsidiary is left out. With no Cost of Goods Sold accounts, leave out Cost of sales and its KPI, and Gross profit equals Total revenues.

**Rollup rows (operations):** one per group with a plug row; `FSN_L3_INC` (reversed); `FSN_L3_COG`; `FSN_L3_EXP`; `FSN_L3_OIN` (reversed); `FSN_L3_OEX`; `FSN_L4_NIN` (reversed on actual columns; the Budget formula in budget columns).

---

## Cash flow

Pages 3, 9, 10, 13, 16, 19 and 22. Indirect method. Every line of balance sheet accounts shows the change in the column's range, reversed. Indents: captions 0, lines 1, the lines under a group of lines 2, detail 3.

| Caption | Kind | Accounts or formula | Notes |
|---|---|---|---|
| `Cash flows from operating activities:` | Caption | | |
| Net income | Single line, dynamic | B `FSN_L4_NIN`, the Actual formula, reversed | `Net income / Net loss / Net income (loss)`. $ |
| `Adjustments to reconcile net income to net cash:` | Caption | | Indent 1, bold |
| Depreciation and amortization | Group | The operations tree's depreciation and amortization accounts, as returned | Indent 2. Left out when there are none. |
| Changes in operating assets and liabilities | Group of lines | Accounts receivable; Inventories; Prepaid expenses; Due from (to) affiliates (the intercompany accounts, one detail row per account); the other named current-asset lines; `Other current assets` = [non-cash current assets] − (Accounts receivable, Inventories, Prepaid expenses, the other named current-asset lines) − [intercompany accounts in current assets] − [Undeposited Funds counted as cash], leaving Due from (to) affiliates out of the subtraction because it also holds liabilities; Accounts payable; the named liability lines; Credit card; Deferred revenue; Accrued and other current liabilities (the balance sheet's accrued groups, then `Other current liabilities` = [`FSN_L3_OCL` change, reversed] − (those groups, the named lines of this type) − [intercompany accounts in other current liabilities]); then the total | The same accounts, groups and plug rows as the balance sheet lines of the same names. Lines left out of the balance sheet are left out here. |
| Net cash from operating activities | Subtotal, dynamic | From Net income | `Net cash provided by operating activities / Net cash used in operating activities / Net cash provided by (used in) operating activities`. Bold, top border, $ |
| `Cash flows from investing activities:` | Caption | | |
| Purchases of property and equipment | Calculated | [property and equipment change] − Depreciation and amortization total | |
| Other assets | Group | Other Asset accounts | |
| Net cash from investing activities | Subtotal, dynamic | From Purchases of property and equipment | As operating, with "investing". Bold, top border. |
| `Cash flows from financing activities:` | Caption | | |
| Long-term liabilities | Group | Long Term Liability accounts, intercompany accounts left out | |
| Common stock and owners' capital | Group | The Paid-in capital accounts, the distribution accounts left out | |
| Distributions | Group | The distribution accounts the user names | Optional |
| Other equity | Plug | [posted equity change] − (Common stock and owners' capital, Distributions) | |
| Net cash from financing activities | Subtotal, dynamic | From Long-term liabilities | As operating, with "financing". Bold, top border. |
| Effect of exchange rate changes on cash | Plug | [bank change] − (net cash from operating, investing and financing) | On pages 14 to 22 the caption reads `Exchange rate effect and cash not assigned to a segment`. |
| Net increase in cash | Subtotal, dynamic | `=AGGREGATE(9,0,<Net income>:<Effect of exchange rate changes>)` | `Net increase in cash / Net decrease in cash / Net increase (decrease) in cash`. Bold, top border, $ |
| Cash at beginning of period | Calculated | Cash at end of period − Net increase in cash | $ |
| Cash at end of period | Group | The Cash and cash equivalents accounts, with the "Actual, balance range" formula. B: as on the balance sheet. Its plug row reads its own rollup row at the balance range. | Bold, top and double bottom border, $. Detail rows hidden as usual. |
| (blank) | Blank | | |
| `Cash flow measures:` | Caption | | |
| Free cash flow | Amount KPI | Net cash from operating activities + Purchases of property and equipment | $ |
| Cash conversion (operating cash flow to net income) | Ratio | Net cash from operating activities / Net income | Ratio format |
| Operating cash flow % of revenue | Ratio | Net cash from operating activities / [revenue rollup] | Percent |
| Distributions % of net income | Amount KPI | `=IFERROR(-<Distributions total>/<Net income>,0)` | Percent. Only with a Distributions line. |

**Undeposited Funds.** When the user counts Undeposited Funds as cash, it is a detail row of Cash at end of period, its account joins the bank rollup in B (`FSN_L3_BNK^<account>`), and Other current assets subtracts it through a rollup row with the account in B. Otherwise it is an Other Current Asset like any other.

**Purchases of property and equipment** assume that depreciation and amortization is credited to accumulated depreciation and amortization accounts of the Fixed Asset type. When the user says amortization is credited to Other Asset accounts, subtract the amortization accounts' total from Other assets instead, and confirm with the exchange rate check.

**Rollup rows (cash flow):** one per group with a plug row; non-cash current assets, B `FSN_L3_ARE^FSN_L3_URE^FSN_L3_OCA^FSN_L3_DEX` (reversed); intercompany accounts in current assets (reversed); `FSN_L3_OCL` (reversed); intercompany accounts in other current liabilities (reversed); property and equipment, `FSN_L3_FAS` (reversed); posted equity, `FSN_L3_EQU` (reversed); bank change, B as the cash group's (as returned); cash at end of period, the same B at the balance range (as returned), for the group's plug row; revenue, `FSN_L3_INC` (reversed); `FSN_L0_INC` as a code (reversed), for the net income check; Undeposited Funds counted as cash, B the account (reversed); for each group with a plug row whose type holds intercompany accounts, those accounts, with the group's sign.

---

## Checks

Below the footer, two rows down: the checks block. Nothing in it prints.

| Row | C | E | F onward |
|---|---|---|---|
| Header | `CHECKS (not printed)` | `Overall` | F: `=IFERROR(IF(COUNTIF(<status cells>,"OK")+COUNTIF(<status cells>,"Review")<<n>,"Error",IF(COUNTIF(<status cells>,"Review")>0,"Review","OK")),"Error")`, where `<status cells>` is the E range of the check rows and `<n>` the number of check rows |
| Caption | `ERP rollups used by plug lines and checks` | | |
| Rollup rows | Their captions | | As "Completeness" says |
| Blank, then caption | `Checks` | | |
| Check rows | Their captions | `=IFERROR(IF(SUMPRODUCT(ABS(F<r>:<last column><r>))<1,"OK","<fail>"),"Error")`, where `<fail>` is `Error`, or `Review` for the checks marked Review below | `=ROUND(<a>-<b>,0)` in the value columns the check covers; every other cell empty, not `""` |

Every row of the block below the header is hidden at outline level 1. `<last column>` is the page's last print column. A check cell that shows an error or `#GETTING_DATA` makes its status, and the page's Overall, read Error: the Overall counts the statuses that are OK or Review, so nothing unread passes.

**Every page of the statement:**

| Statement | Check | Value |
|---|---|---|
| Balance sheet | Total assets less total liabilities and equity | Total assets − Total liabilities and equity |
| Balance sheet | Total assets less ERP assets | Total assets − ([`FSN_L1_AST`] + [intercompany liability accounts]). The affiliate group carries the intercompany liabilities on the asset side, net. |
| Operations | Net income less ERP net income | Net income − [`FSN_L4_NIN`] in every value column, budget columns included |
| Balance sheet, cash flow | Net income less income statement accounts | Net income − [`FSN_L0_INC`] in every value column |
| Balance sheet, except `BS-ACTIVITY` | Retained earnings less ERP retained earnings | Retained earnings − [`FSN_L4_RET`] in every value column |
| Operations | Total revenues less revenue rollup | Total revenues − [`FSN_L3_INC`] |
| Operations | Total cost of sales less COGS rollup | Total cost of sales − [`FSN_L3_COG`] |
| Operations | Total operating expenses less expense rollup | Total operating expenses − [`FSN_L3_EXP`] |
| Cash flow | Net change in cash less change in bank balances | Net increase in cash − [bank] |
| Cash flow | Effect of exchange rates within materiality | `=ROUND(IF(ABS(<FX>)>MatAmt,<FX>,0),0)`. On the segment pages (14 to 22), in the Total and Prior year columns only: a single department, class or location doesn't balance, so its column carries cash not assigned to the segment. A large exchange rate effect means a balance sheet account sits on no cash flow line, or a line holds the wrong accounts. |

**One page only.** Pages are named by ID (`references/pages.md`, "Page list"), since the sheet numbers change when pages are left out.

| Page | Check | Value | On failure |
|---|---|---|---|
| `BS-MQY` | Balance sheet accounts not listed on this page | See "Missing accounts" | Review |
| `IS-MQY` | Income statement accounts not listed on this page | See "Missing accounts" | Review |
| `IS-MQY` | Revenue lines add up to Total revenues | Total revenues − the sum of the revenue lines and Other revenue, each by its own row or total row | Error |
| `CF-MQY` | Year-end cash less balance sheet cash | In F: Cash at end of period in P − the cash total in P on `BS-MQY` | Error |
| `CF-MQY` | Beginning cash less prior year-end cash | In F: Cash at beginning of period in P − the cash total in Q on `BS-MQY` | Error |
| `CF-MQY` | Month and quarter cash equal year cash | In F: `=ROUND(ABS(F<end>-P<end>)+ABS(K<end>-P<end>),0)`, with `<end>` the Cash at end of period row | Error |
| `BS-ACTIVITY`, `IS-MONTH`, `CF-MONTH`, `IS-BUD-MONTH` | Sum of months less total | In the Total column only: `=ROUND(SUM($F<x>:INDEX($F<x>:$Q<x>,Period))-<Total><x>,0)` for Total assets (`BS-ACTIVITY`), Net income and Total revenues (`IS-MONTH`, `IS-BUD-MONTH`), Net increase in cash (`CF-MONTH`) | Error |
| `BS-CONSOL`, `IS-CONSOL`, `CF-CONSOL` | Sum of subsidiaries less consolidated | In the Consolidated column: the member columns' sum − Consolidated, for Total assets and Total liabilities and equity (balance sheet), Total revenues and Net income (operations), Net increase in cash (cash flow) | Review: members left out, such as one on another fiscal calendar, make it differ |
| Segment pages without an All other column | Sum of segment columns less total | In the Total column: the member columns and the No column − Total, for the same rows | Review: overlapping `(Hierarchy)` members and inactive values make it differ |
| `IS-BUD-CONSOL` | Sum of subsidiary budgets less consolidated budget | In the Consolidated budget column, for Total revenues and Net income | Review |

**Missing accounts** (`BS-MQY` and `IS-MQY`). One row in the checks block, below the check rows:

| Row | B | C | E | F |
|---|---|---|---|---|
| Missing | `FSN_L0_BAL` (`BS-MQY`) or `FSN_L0_INC` (`IS-MQY`) | the check caption | `=IFERROR(IF(COUNTA(F<r>#)=COUNTBLANK(F<r>#),"OK","Review"),"Error")` | `=FSN.Range.AccountNumbers($B<r>,$B$<first>:$B$<last>,Subsidiary,,TRUE,FALSE,FALSE,TRUE,TRUE)`, with `<first>` to `<last>` the body rows |

It spills across, so leave the rest of the row empty. Finsyte's Exclude Filter matches account numbers, ranges, wildcards and `id_` values, and splits each cell at `^`; a code such as `FSN_L3_BNK` matches no account there. So column B names the accounts of the lines that have no detail rows: Other income (expense), net and Other equity hold theirs joined with `^`. What the check lists are accounts no row names: accounts NetSuite added after the build, which sit in a group's plug row, a plug line or an Other group's `Accounts not listed` row, and accounts the line plan missed. A new build passes: every account is a detail row, in Other income (expense), net, or named in Other equity's B. The totals are right either way, so the status is Review. Tell the user which accounts are listed, and offer to add them to a line.

---

## Row formats

- **Font:** Times New Roman 10 pt, from the Normal style (`references/excel-mechanics.md`, "Formatting calls"). The checks block's content cells: 8 pt, color `#595959`, set row by row on the cells that hold a caption, a code or a formula, never in one call over the block; its header bold, color `#808080`, fill `#F2F2F2`.
- **Bold:** captions, group totals, subtotals, Gross profit, Operating income, Net income, the net cash lines, Net increase in cash, Cash at end of period.
- **Borders**, on the value and change cells of the row: a thin top border on group totals, subtotals and the bold calculated lines; a thin top and a double bottom border on Total assets, Total liabilities and equity, Net income (operations) and Cash at end of period.
- **$ format** (`references/pages.md`, "Number formats"): the first line under each caption that starts a section, with its first detail row and its total; every row the trees above mark $. Other rows take the amount format; ratio and percent KPIs their own.
- **Outline:** detail rows, a group of lines' lines and its total, KPI rows and the checks block are hidden. Group them with Excel outline groups, one level per depth.
- **Band:** a conditional format over C, from the first body row to the last KPI row and across to the last print column: `=MOD(SUBTOTAL(103,$A$<first>:$A<first>),2)=1`, fill `#CCEEFF`. It shades every other visible row.
- **n/m:** one conditional format over the body of every `% Change` and `% Variance` column of the page, as one multi-area range: `=AND(ISNUMBER(<top cell of the first area>),ABS(<top cell of the first area>)>9.99)` with the number format `"n/m";"n/m";"n/m"`.
- **Status colors:** on the E status cells and the Overall cell: `OK` bold green `#008000`, `Error` bold red `#C00000`, `Review` bold orange `#C65911`.
