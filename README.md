# Finsyte for Claude

[Finsyte](https://www.finsyte.com) is an Excel add-in for building financial reports and models on NetSuite data. Finance and FP&A teams, accountants, and board members work from Finsyte workbooks, and NetSuite stays the source of truth: every number is a live `=FSN.` formula that refreshes from NetSuite instead of being re-keyed.

This plugin adds the **financial-reports** skill, which teaches Claude how Finsyte works so it can help you build new reports and models and change the ones you have. Claude starts from Finsyte's report templates, pulls any account, account type, or rollup with the right parameters, lays reports out by NetSuite account type the way NetSuite does, and keeps signs, totals, and segment arguments right. The result is a workbook that ties out to NetSuite and re-points to a new period or subsidiary by changing one cell.

Claude uses the skill on its own whenever a request involves Finsyte, FSN formulas, or NetSuite data in Excel. In Claude Code you can also call it directly with `/finsyte:financial-reports`.

## What it helps with

- Starting a balance sheet, income statement, cash flow statement, budget vs actual, or trial balance from Finsyte's templates, then customizing it in Excel
- Changing existing Finsyte reports: adding rows, prior-year and budget columns, variances, or another subsidiary
- Building models such as department and location P&Ls, budget vs actual, trailing twelve months, and KPI or board summaries, driven from one inputs tab with Finsyte dropdowns
- Laying out rows and columns from NetSuite lists, so new departments, subsidiaries, and accounts show up on refresh
- Replacing hard-coded numbers in a spreadsheet or NetSuite export with live Finsyte formulas, then tying every converted cell back to the original
- Diagnosing `#VALUE!`, `#NAME?`, `Invalid account number`, `#GETTING_DATA`, `#SPILL!`, and sign errors in FSN formulas
- Adding a check that flags GL accounts created in NetSuite after a report was built, so totals don't go stale silently

## Requirements

- Microsoft Excel with the [Finsyte add-in](https://www.finsyte.com/trial) installed and connected to NetSuite
- A NetSuite login whose role can view the data the report uses

Claude writes the formulas. The Finsyte add-in evaluates them in Excel, against your NetSuite account and under your own NetSuite permissions.

## Example prompts

1. "I just ran Load Accounts on Finsyte's Income Statement template. Add a prior-year column and a variance column, and set it up so the board sees one line per account type."
2. "Build a budget vs actual income statement for Headquarters (Consolidated), FY 2026 year to date through period 6, using the Board Approved budget category, with one column per department."
3. "Build a one-page board summary with revenue, gross margin, operating expenses, and net income for this quarter, compared with budget and with the same quarter last year."
4. "I exported our December 2025 balance sheet from NetSuite into this workbook. Convert it to live Finsyte formulas and show me any rows that don't tie to the original."
5. "Every FSN.GLAccountBalance cell on my P&L shows #VALUE! since I pointed its parameter cells at a Drivers sheet. What's wrong?"

## What this plugin does and doesn't do

This plugin contains instructions and reference material only: Markdown files that Claude reads. It contains no executable code or scripts, makes no network requests, and does not collect, store, or send any data. When Claude works in your workbook it writes Excel formulas, which the Finsyte add-in evaluates. The plugin covers reporting with Finsyte's functions and does not create, change, or delete records in NetSuite.

All NetSuite access happens in the Finsyte add-in on your computer, under your NetSuite login and role. How Finsyte handles your data is described in the [Finsyte privacy policy](https://www.finsyte.com/privacy-policy).

AI-generated output can contain errors. Review every report before relying on it; the skill's tie-out and QA steps are there to help with that.

## Documentation and support

- Finsyte documentation: <https://www.finsyte.com/docs/add-in/docs/getting-started>
- Support and contact: <https://www.finsyte.com/contact>
- Privacy policy: <https://www.finsyte.com/privacy-policy>
- Terms and conditions: <https://www.finsyte.com/terms-and-conditions>

## License

Copyright 2026 Finsyte.com LLC. The contents of this repository are free to use, copy, modify, and distribute under the terms in [LICENSE](LICENSE).

The license covers this plugin's files only. It grants no rights to the Finsyte add-in, which is licensed under its own [software license agreement](https://www.finsyte.com/software-license-agreement), or to the Finsyte name and logo.

Claude is a product and registered trademark of Anthropic, PBC; Finsyte is not affiliated with, endorsed by, or sponsored by Anthropic. NetSuite is a trademark of Oracle Corporation. Microsoft Excel is a trademark of Microsoft Corporation.
