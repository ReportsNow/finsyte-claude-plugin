---
name: board-book
description: "Use when the user wants a board book, board package, board pack, board reporting package or executive summary book built in Excel from NetSuite data through Finsyte (=FSN. formulas): a cover, an index and up to 26 landscape pages of balance sheets, statements of operations and cash flows by month, quarter and year, monthly and cumulative trends, a five-year history, consolidating statements by subsidiary, statements by department, class or location, and budget vs actual, with KPIs and tie-out checks. Also for rolling a board book built this way to a new period, adding or changing its pages, lines or columns, and explaining its checks. A single report or a one-page board summary belongs to the financial-reports skill; a health check of a whole workbook to the workbook-health-check skill."
---

# Board book

A board book is a reporting package for a board of directors: an Inputs sheet, a cover, an index, and up to 26 report pages, each built from the company's own chart of accounts with live Finsyte formulas. Every number re-points from the Inputs sheet, every total ties to a NetSuite rollup, and every page carries hidden checks that say whether it ties. You ask the user a short series of questions first, show the build plan, and build only after the user says yes.

## Reference files

| File | Read it |
|---|---|
| `references/excel-mechanics.md` | First: the checks before you start, discovery of NetSuite's lists and chart of accounts, the Office.js calls, and waiting for values |
| `references/interview.md` | Before you ask the user anything |
| `references/front-pages.md` | Before you write Lists, Inputs, Cover or Index, and for every workbook name |
| `references/pages.md` | Before you write a report page: the page frame, each page's columns and parameters |
| `references/statements.md` | Before you write a page's rows: the line plan, the three statement trees, signs, plug lines, checks and row formats |

## The Finsyte skill

The `financial-reports` skill in the same plugin explains how Finsyte works: functions and their arguments, signs, `FSN_` codes, formula conventions and gotchas. Load it before you start. A user who added the Finsyte skill on its own may have it as `finsyte-netsuite`, an older copy: with both installed, use `financial-reports`. Without either, tell the user the Finsyte skill is missing and continue: this skill spells out every formula the book needs.

## Where it runs

The book is built in Excel for Windows with the Finsyte add-in loaded and signed in, through Office.js. Discovery reads NetSuite through Finsyte formulas in the workbook, so in Excel for the web, on a Mac, or from a workbook file without Excel, there is nothing to read the chart of accounts with: say so, and offer to walk through the questions and the plan so the user can build in Excel for Windows.

## Ground rules

1. **Questions, then a plan, then the build.** Run the interview (`references/interview.md`): one question per message, then wait for the answer. Never send a round of numbered questions. No report page is written before the user answers the plan with a yes.
2. **Every NetSuite value comes from Finsyte.** Each subsidiary, fiscal year, account, member and budget category in the book is one that a Finsyte formula returned in discovery. Never type an account number from memory or by analogy.
3. **A new workbook.** Build in a blank workbook. Never build over a board book, and never into a workbook whose sheet or workbook names clash (`references/excel-mechanics.md`, "Before you start").
4. **The Finsyte formula conventions hold everywhere.** One Finsyte call per cell, with nothing around it but a leading minus; every parameter in a cell, read from Inputs; Subsidiary always filled; totals as `AGGREGATE(9,0,…)`; no volatile function in any cell a Finsyte call reads (no `TODAY`, `NOW`, `OFFSET`, `INDIRECT`, `CELL`, `INFO`, `SHEET`, `RAND`). The front pages' own label formulas are the one place `TEXT` and `CHOOSE` wrap dates.
5. **Complete by construction.** Every statement section ends in a plug line against a NetSuite rollup, and every group of a whole type or parent ends in a plug row (`references/statements.md`, "Completeness"). Never drop a plug, a rollup or a check to make a page pass.
6. **Values before checks.** Read checks only when no cell shows `#GETTING_DATA`. A check read from partial values says nothing.
7. **Report what the checks find; never force them.** Fix your own build mistakes: a wrong row in a formula, a wrong sign, a group range one row short. A difference that comes from NetSuite, such as an account in an Other line or an exchange rate effect over materiality, goes in the report with its cause. Never type a number, change a check's formula or widen its tolerance to make it pass.
8. **Build without stopping.** The plan's yes covers the whole book: go from page to page without stopping to report or ask, until the book is built, verified and reported. Keep the user posted with one status line between pages, such as "Page 4 of 26: Balance Sheets by month", and go straight on. When a reply ends before the book is done, its last line is `Built: Lists, Inputs, Cover and pages 1 to <n> of <N>. Reply "continue" to build from page <n+1>.`, with what was actually built. If the user stops you, or the build fails partway, leave what is built, say what is missing, and offer to delete the book's sheets and names.

| Thought | Reality |
|---|---|
| "The user wants it fast, so I'll skip the questions." | Round 1 and the plan always run. "Defaults for the rest" answers every question after round 1 in one reply. |
| "Asking the whole round in one message is quicker." | Users miss and mix up answers in a list of five. One question, then wait. |
| "This segment has too many values; I'll suggest leaving it out." | Show the checklist anyway, and say how wide the checked columns print. The user unchecks values to narrow it; unchecked values go to All Other. |
| "Cash is usually 1000 and revenue 4000." | Read the chart of accounts. Numbers differ by company, and a wrong one returns "Invalid account number". |
| "`TODAY()` gives the default period." | Discovery types last month's end date in a scratch cell. Inputs holds typed values. |
| "`CELL("filename")` gives the page number." | `CELL` is volatile. Type the page number. |
| "`FSN_L4_NIN` is net income everywhere." | It includes a parent's children and ignores segments. Net income on balance sheet and cash flow pages comes from the income statement accounts; `FSN_L4_NIN` checks the consolidated columns only. |
| "The check is off by 3; I'll adjust the plug." | Find the cause and report it. A plug that hides a difference hides it from the board. |
| "I'll put the book in the user's report workbook." | Workbook names such as `Period` and `Subsidiary` clash with theirs. Use a new workbook. |
| "One request writes the whole book." | Write page by page in batches under 5 MB, and sync after each. |
| "Inputs and Cover are done; I'll stop and report." | The plan's yes covers the whole book. Post a status line and build the next page. |
| "The first read looks fine." | Wait until no cell shows `#GETTING_DATA`, then read. |
| "Every check reads OK, so the book ties." | A paused Finsyte returns 0, and 0 less 0 passes every check. Check that Total assets holds a value first. |

## Build the book

1. **Load** `financial-reports`, and read `references/excel-mechanics.md`.
2. **Check before you start** (`references/excel-mechanics.md`, "Before you start"). Stop at the first failed check and tell the user what to do.
3. **Discovery, part 1:** the Lists sheet, columns A to F.
4. **Round 1** of the interview. Write Inputs C5, C11 and C14 and their names, then **discovery, part 2:** periods, the subsidiary tree, segments, the chart of accounts.
5. **Rounds 2 to 4.** Draft the line plan for round 3 from the chart of accounts.
6. **The build plan.** Wait for a yes.
7. **Build**, in this order: the rest of Lists; Inputs (with the line plan in its notes); Cover; each report page, in page order; Index; then `BookChecks` on Inputs. Before the first page, set the Normal style's font (`references/excel-mechanics.md`, "Formatting calls"). For each page: frame and titles, parameter block, headings, body rows, checks block, number formats, bold, sizes and colors on the content cells only (the font name comes from the Normal style), outline, conditional formats, page setup, then the used-range check.
8. **Wait for values** (`references/excel-mechanics.md`, "Waiting for values").
9. **Verify** (below), fix your own mistakes, and wait and verify again after each fix.
10. **Clean up:** clear the discovery area on Lists, hide Lists, collapse the outlines, and select Cover A1.
11. **Report** (below).

**Resume a stopped build.** When the user says "continue", "resume" or "keep going" after a build stopped, pick up where it stopped; never rerun the interview or ask for the plan again. List the book's sheets and names. The last sheet written may be half done:

- A report page: delete that sheet and build it again.
- Lists, Inputs or Cover: other sheets read their names, so never delete them. Check each cell and name `references/front-pages.md` gives them and write the missing ones.

Then go on in the order of step 7 from the first thing missing: the next page, or with every page there, the Index and `BookChecks`. Then steps 8 to 11.

## Verify

1. **Finsyte returned values.** A paused Finsyte returns 0 in every value cell it holds no cached value for, and 0 less 0 passes every check. Total assets of 0 on page 1, or a page of zeros, stops the verify: ask the user what the button in Finsyte's Refresh group reads. Resume Connection means Finsyte is paused: the user clicks it, then Refresh Data › Entire Workbook, and you wait for values and verify again.
2. **Errors.** Search every page for `#VALUE!`, `#NAME?`, `#REF!`, `#SPILL!` and Finsyte messages such as "Invalid account number". Each one is a build mistake or a gotcha in the Finsyte skill: fix its cause.
3. **Checks.** Read each page's Overall cell and every check row. For each Error, find the row the check points to and decide:

| Check | Usually a build mistake when | Usually NetSuite when |
|---|---|---|
| Total assets less total liabilities and equity | A group's range misses a row; a liability row lacks its minus | Never: the book balances by construction |
| Total assets less ERP assets; revenue, cost of sales, expense rollups | A detail row sits outside its group's range; a plug subtracts a line twice | An account on two lines; the intercompany accounts are wrong |
| Net income less ERP net income | A sign; the net income list reads the wrong range | A posting in an account type outside the operations rollups |
| Net change in cash less change in bank balances | A cash flow line outside the subtotal ranges | Never: it ties by construction |
| Effect of exchange rates within materiality | A balance sheet line missing from the cash flow tree; a wrong sign | Real translation effects; accounts the line plan left out of the cash flow. On segment pages it reads the Total columns only. |
| Sum of months less total | A month column with the wrong period or range | Never |
| Sum of subsidiaries or segment columns less total (Review) | A column with the wrong parameter | Overlapping `(Hierarchy)` members; subsidiaries left out of the consolidating columns, such as one on another fiscal calendar; inactive segment values that still hold balances |
| Accounts not listed on this page (Review) | A body account missing from the exclude list | New or unlisted accounts, now in a group's plug row or an Other line |

4. **Tie across pages.** Net income on page 2 (year column) equals net income on pages 1 and 3 (year column), and cash at end of period on page 3 equals cash on page 1. The month and cumulative pages agree with page 2 and page 3 at the book's period.
5. **Against NetSuite.** Ask the user to compare total assets and net income for the period with NetSuite's own balance sheet and income statement: the checks prove the book ties to Finsyte's rollups, and this proves the parameters are the ones the user meant.

## Report

```
Board book built: <workbook>, <N> pages, <fiscal year> period <n> (<end date>).
Checks: <n> OK, <n> Review, <n> Error. Inputs › Book checks shows the book; Index column G shows each page.

Error
- Page <n>, <title>: <check>. <What it found, the cause, and what to do>.

Review
- Page <n>: <accounts> are in <Other line>. Say "list <account> under <line>" and I'll add the rows on every page of that statement.

Not in this book: <page groups left out, and why>.

To roll forward: pick Current fiscal year and Period from their dropdowns on Inputs, wait for the values, and check Book checks. The other dropdowns on Inputs (subsidiary, accounting book, currency, budget category and the column lists) re-point every page the same way.
```

Then the one optional manual step: Office.js puts each outline button below its group; Data › Outline › clear "Summary rows below detail" moves the buttons to the header rows.

## Change a board book

Read the pages before you change them: the line plan is on Inputs (E3 down) and in the rows of any page of the statement.

- **Roll forward.** Pick Current fiscal year and Period from their dropdowns on Inputs, and change nothing else. Wait for values, then read Book checks. Never rebuild for a new period.
- **List a new account.** Insert a row inside the group, between its first and last detail rows, so the group's ranges grow; copy the formulas of the detail row above; type the account in B. Do it on every page of that statement and on the matching cash flow line. A group plug row's `SUM` range grows with the insert; in a group with sub-groups, where the plug row subtracts rows by name, add the new row to its subtraction. The section's Other line subtracts the group's total, so the account leaves the Other line by itself. An account in an Other group's `Accounts not listed` row is listed the same way, as a detail row of that Other group or of another group.
- **Change a column's member.** Change the entry on Inputs (H, J, L or N). Adding or removing a column: copy a member column, or delete one, on every page of that group, resize the list's name on Inputs, and update the sum checks.
- **Add a page or a page group.** Build it from `references/pages.md` with the existing Inputs and line plan, insert it in page order, renumber the sheets after it, and update the Index, its links and `BookChecks`.
- **Rename a caption.** Type over it on every page of the statement and in the Inputs notes.

## Edge cases

| Situation | What to do |
|---|---|
| A single company, or a subsidiary with no children | No consolidating pages and no page 24. Scope reads Subsidiary, and the titles drop "Consolidated". |
| Elimination subsidiary | The last consolidating column. Left out of page 24 and of Affiliates. |
| An inactive subsidiary | Not in `ListSubsidiaryMembers`, so never a column. When the user names one, look for it in `=FSN.Range.Subsidiaries(,,,TRUE,FALSE)`: if it's there, say it is inactive, not missing. Its balances can still count in the `(Consolidated)` column; then the sum check reads Review. |
| A member with another fiscal calendar | Left out of the consolidating columns by default (round 2): its period numbers fall on other months. The sum check then reads Review, and the report says why. |
| More than one currency | Every value column passes `ReportCurrency`. Budgets have no currency argument: say so when budgets of different currencies meet on page 24. |
| A segment unused, or no budget categories | Leave out its pages. |
| Departments, classes or locations | A single member doesn't balance on its own, so on the segment pages the translation adjustment and the exchange rate lines carry what isn't assigned to a segment, and their captions say so. |
| Intercompany accounts | Balance sheet accounts only. Intercompany revenue, cost and expense stay on the operations lines. |
| A mid-year book | The actual month columns after the book's period show blank until the book rolls forward. |
| More than 8 member columns | The page runs several pages wide with C:E repeated. Suggest 8 or fewer. |
| No Cost of Goods Sold accounts | No Cost of sales line; Gross profit equals Total revenues. |
| Fewer than five fiscal years | Page 8 shows the years that exist, or is left out with fewer than two full prior years (`references/pages.md`). |
| No prior fiscal year in NetSuite | The prior-year columns can't calculate. Tell the user in round 2: this layout compares with the prior year, so build only if they accept those columns showing a Finsyte message. |
| Period 1 | The prior month and prior quarter-end columns read the prior fiscal year's last period. Inputs handles it. |
| 13 periods, or an adjustment period | `PeriodsPerYear` counts regular periods. For 13 periods, type the quarter cells (`references/front-pages.md`). For adjustment entries, see round 1. |
| A fiscal year that isn't the calendar year, or not named `FY yyyy` | Month headings come from `FSN.PeriodToDate`, earlier fiscal years from the fiscal year list, and the headings that name a fiscal year use its NetSuite name, so all of them follow NetSuite. |
| Accounts without numbers, or with `-`, `^`, `*` or `?` in the number | Use `id_<internal id>` in B. |
| Account numbers with leading zeros | Column B is text, set before the values are written. |
| Summary accounts | Never detail rows: their posting children are. A posting parent is a detail row of its own. |
| Inactive accounts with balances | Not listed as detail; their balances land in the group plug rows through the rollup lists, which include inactive accounts. |
| Finsyte paused or signed out during the build | Paused: value cells read 0, and the button in Finsyte's Refresh group reads Resume Connection; the user clicks it. Signed out: value cells show empty text, the cells with a leading minus show `#VALUE!`, and the Finsyte tab's Connectivity button reads Login; the user signs in. Then Refresh Data › Entire Workbook, and verify only after the values return. |
| The user stops the build | Leave what is built, list what is missing, and offer to finish or to delete the book's sheets and names. |
