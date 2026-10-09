# Finsyte Report Templates

Read this when a report starts from a Finsyte template, or when the sheet in front of you was made from one: the parameter block at the top, a column of account numbers, `FSN.GLAccountBalance` formulas, and Excel outline groups down the left are the signs. Section names in *italics* refer to SKILL.md.

## Contents

- The templates
- What Load Accounts writes
- Layouts
- Sign reversals by template
- Customizing a template

---

## The templates

All are on the Finsyte ribbon › Templates and Actions › Standard Reports. Balance Sheet, Income Statement, Cash Flow, and Trial Balance are submenus holding each statement's versions; Budget vs Actual is directly on the menu. Each template opens as a new sheet. Templates match the layout and groupings of the same reports in NetSuite, and they're starting points: the user customizes them in Excel.

| Template | Contents |
|---|---|
| **Balance Sheet** | Assets, and Liabilities & Equity, grouped by account type, with Retained Earnings, Net Income, and a Cumulative Translation Adjustment block |
| **Income Statement** | Income, Cost of Sales, Gross Profit, Expense, Net Ordinary Income, Other Income and Expenses, Net Income |
| **Cash Flow** | Indirect method: Operating, Investing, and Financing Activities, then Net Change in Cash, Cash at Beginning and End of Period |
| **Budget vs Actual** | The Income Statement layout with Actual Amount, Budget Amount, Amount Over Budget (actual − budget), and % of Budget (actual ÷ budget) columns |
| **Comparative** Balance Sheet, Income Statement, Cash Flow | The same layouts with a second value column, a difference column, and a % column |
| **Trial Balance** | Every active posting account, balance sheet and income statement, plus Retained Earnings, as one flat list that totals to zero |
| **Grouped Trial Balance** | The same accounts, with each NetSuite parent account as a header over its sub-accounts and a subtotal |
| **Comparative Trial Balance**, **Comparative Grouped Trial Balance** | The two above with a second value column, difference, and % |

**Parameters.** Every template has Subsidiary, Fiscal Year, Period, and Period Range cells at the top, plus Book; Budget vs Actual adds Budget Category. Each parameter cell has a Finsyte dropdown. In a Comparative template, each value column has its own parameter cells, so it can compare two years, two periods, or two subsidiaries. Rows for Item, Class, Department, Location, Entity, Transaction Type, Posted, Currency, and each custom segment are added by default, skipping features the NetSuite account doesn't use. The user can turn any of them off in Help › Preferences › Template Parameters, which affects only templates created afterward.

**Period Range.** A balance sheet uses `YTD`. The Cash Flow templates accept only `PTD` and `RTD`, because a cash flow statement has no opening balance.

---

## What Load Accounts writes

After the parameters are set, the user runs Finsyte ribbon › Templates and Actions › Load Accounts. It reads the chart of accounts for the selected subsidiary and writes:

- **Account rows**, grouped the way the same NetSuite report groups them (see *Layouts*), with sub-accounts under their NetSuite parent.
- **A value formula per account** that reads the parameter cells in its own column: `=FSN.GLAccountBalance(D$5,D$6,D$7,D$8,$B12,D$9,…)`, with a leading `-` on rows the statement reverses. Parameter references are locked by row and the account reference by column, so a copy of the column one to the right reads the parameter cells above the new column.
- **Subtotal rows** as `=AGGREGATE(9,0,<first>:<last>)` over each group's block.
- **Header rows** that show the group's total only while the group is collapsed: `=IF(SUBTOTAL(102,…)=0, AGGREGATE(…), "")`.
- **Name cells** from `FSN.GLAccountName`, with a `"Total - "` prefix on subtotal rows while the group is expanded.
- **Calculated rows** such as Gross Profit, Net Ordinary Income, Net Income, Net Change in Cash, and Cash at End of Period, as arithmetic on the subtotals.
- **Excel outline groups** for expanding and collapsing each level, and the Finsyte cell styles: `FS L1` to `FS L4`, their `Total` versions, `FS Posting`, `FS Sum`, and `FS Input`.

**Run it once.** Load Accounts clears and rebuilds the whole generated area each time, so rows, formulas, and formatting added there are lost. Run it when the template is first created and never on an existing report. To change the period or subsidiary, change the parameter cells; to report another subsidiary, copy the sheet. Add the check in *Keep reports complete* (SKILL.md) for accounts created in NetSuite later.

Template sheets carry hidden workbook names that begin `FinsyteNS.`, which Finsyte uses to find the parameter cells and the generated area. Leave them as they are.

---

## Layouts

**Balance Sheet.** Assets, and Liabilities & Equity, each broken down by group (Current Assets, Long Term Assets, and so on) and then account type. Retained Earnings (`FSN_L4_RET`) and Net Income (`FSN_L4_NIN`) appear under Equity. Cumulative Translation Adjustment is a group inside Equity: four rows pulling `FSN_L1_AST`, `FSN_L4_RET`, `FSN_L4_NIN`, and `FSN_L1_LEQ`, none reversed, with an `AGGREGATE` subtotal. See `references/converting.md` for why CTA is calculated this way.

**Income Statement and Budget vs Actual.** Ordinary Income/Expense holds Income, then Cost of Goods Sold labeled "Cost of Sales", then Gross Profit, then Expense, then Net Ordinary Income. Other Income and Expenses holds Other Income and Other Expense, then Net Other Income. The statement ends with Net Income and a "Net Income (Check)" row that pulls `FSN_L4_NIN` directly and should equal it. Budget vs Actual's budget column uses `FSN.BudgetAccountBalance`.

**Cash Flow.** Indirect method, built from account types and NetSuite's special account types:

- **Operating Activities:** Net Income, then Adjustments to Net Income: Accounts Receivable, Unbilled Receivable, Inventory Asset, Other Current Asset, Accounts Payable, payroll liabilities, Sales Tax Payable, and Other Current Liabilities.
- **Investing Activities:** Fixed Asset and Other Asset.
- **Financing Activities:** Long Term Liabilities, Opening Balance Equity, and Other Equity.
- Then Net Change in Cash for Period, Cash at Beginning of Period (`FSN_L4_CBP`), Effect of Exchange Rate on Cash, and Cash at End of Period.

Undeposited Funds is treated as cash by default, so it stays out of Other Current Asset.

**Trial Balance.** Every active posting account plus `FSN_L4_RET`, so the list nets to zero. A CTA block is included by default (a Preferences setting). The plain versions are one flat list; the Grouped versions make each NetSuite parent account a header over its sub-accounts, with a subtotal.

---

## Sign reversals by template

Templates write a leading `-` on the rows their statement shows as positive. Don't add a second one.

| Template | Rows reversed |
|---|---|
| Balance Sheet, Comparative Balance Sheet | Liability and equity accounts, Retained Earnings, Net Income; the CTA rows are not reversed |
| Income Statement, Comparative Income Statement, Budget vs Actual | Income and Other Income accounts, and Net Income (Check). Budget columns are never reversed |
| Cash Flow, Comparative Cash Flow | Net Income, every line under Adjustments, Investing, and Financing, and the accounts under Effect of Exchange Rate on Cash, so an increase in an asset shows as cash used. Cash at Beginning of Period is not reversed |
| All four Trial Balances | None, apart from the CTA rows |

The user can add or remove a reversal on any value with right-click › Switch Sign, which adds or removes the leading `-`.

---

## Customizing a template

The generated sheet is ordinary Excel. Customize it directly:

- **Add an account row:** insert the row inside the group, above its last account row, so the group's `AGGREGATE` range expands to include it. Copy the value formula from a neighboring account row, which carries the right sign, and put the new account number in the account column.
- **Remove a row:** delete it; the subtotal ranges shrink with it.
- **Rename a section:** overwrite its label. The name cells are formulas; replacing one with text is fine.
- **Add a column** for prior year, another subsidiary, or a budget: copy a value column one column to the right and fill in the parameter cells above the new column. Its formulas read those cells. Variance and % columns are plain arithmetic on the value columns.
- **Restyle** through Home › Cell Styles, by changing the `FS` styles, so every row of that level changes together.
- **Hide detail** by collapsing outline levels; header rows show each collapsed group's total.
- **Point parameters at a Drivers tab** to run several sheets from one set of inputs (`references/model-building.md`). The Period cell is Text-formatted on every template except Balance Sheet, Cash Flow, and Comparative Cash Flow; set it to General before entering a formula in it.

Leave Finsyte's own formulas as they are, including the `IF(SUBTOTAL(…))` header rows and name cells. Write new formulas by the *Formula conventions* in SKILL.md.
