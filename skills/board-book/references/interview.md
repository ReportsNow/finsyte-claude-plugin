# Interview

Read this before you ask the user anything. The interview runs in four rounds, then a build plan the user confirms. Every question offers the values Finsyte returned (`references/excel-mechanics.md`, "Discovery") and a recommended default, so the user can answer "defaults".

## Rules

1. **One round per message.** Number the questions, at most five a round, each with its options and the recommended default marked. Use a question tool if your environment has one; otherwise ask in the chat.
2. **Ask only what changes the book.** When discovery leaves one option, such as one accounting book, state it in the plan and don't ask.
3. **Offer only real values.** Every subsidiary, fiscal year, budget category, member and account you offer is one Finsyte returned. Show accounts as number and name.
4. **"Defaults"** answers a round with the recommended defaults: list what that set. Round 1 always needs the user's own answer for the subsidiary and the period: a wrong default there builds the wrong book.
5. **A changed answer reruns what depends on it.** A new subsidiary reruns the segment and chart of accounts discovery and the rounds after round 1.
6. **Nothing is built before the plan's yes.** Discovery writes the Lists sheet, and after round 1 the Inputs cells it needs (C5, C11, C14 and their names). No report page is written before the user replies to the plan with a yes. A request to "just build it" still gets round 1 and the plan.

---

## Round 1: entity and period

After Lists A to F settle.

1. **Subsidiary.** The list from `ListSubsidiaries`. Default: the top `(Consolidated)` entry, the one whose subsidiary has no parent; the only subsidiary when there is one.
2. **Fiscal year and period.** Default: the period that holds the last day of last month. Show its end date, such as "FY 2025, period 12 (December 31, 2025)". A board book is tied to a close, so the book holds these as fixed values: changing them on Inputs moves the whole book.
3. **Accounting book.** Only with more than one book. Default: the first in the list, the primary book.
4. **Reporting currency.** Only when the subsidiary tree holds more than one currency. Default: the subsidiary's own currency (`FSN.CurrencyName`). Every page reports in this currency.

After the answers, write Inputs C5, C11 and C14 with their names, then the rest of the discovery. If discovery finds an adjustment period with year-end entries (`references/excel-mechanics.md`, "Periods per year"), tell the user the amount and that the book reports regular periods, so period `<PeriodsPerYear>` leaves those entries out; ask whether to note that in the footer text.

---

## Round 2: contents

Show the page groups the account can have, each with what discovery found, then ask:

| Group | Offer it when | Questions |
|---|---|---|
| Financial statements, pages 1 to 3 | Always | None: always in the book |
| Trend analysis, pages 4 to 10 | Always | Include them? Default yes. Say how many fiscal years NetSuite has before the current one, and that page 8 shows five years, fewer when fewer exist, or is left out with fewer than two full prior years. |
| Consolidating, pages 11 to 13 | The subsidiary is `(Consolidated)` with two or more members | Which subsidiaries are columns? Default: every member, the parent's own books included, in NetSuite's order, with the elimination subsidiary last. Say that more than 8 columns print across several pages. Name any member with a different fiscal calendar: its periods don't line up with the book's, so leave it out by default. Name the currencies: every column is translated to the reporting currency. |
| By department, class, location: pages 14 to 16, 17 to 19, 20 to 22 | The segment has values for the subsidiary | One question per segment: which values are columns? Default: the top-level values, each parent as its `(Hierarchy)` entry. A "No department" (class, location) column is always added. Picking a `(Hierarchy)` entry and one of its children counts that child twice, and the sum check flags it. |
| Budget, pages 23 to 26 | `ListBudgetCategories` has entries | Include them, and which budget category? No default unless there is one category. Page 24 needs the consolidating columns. Budgets have no currency argument: when the reporting currency differs from a subsidiary's own, say that page 24 adds budgets in different currencies. |

---

## Round 3: lines

Draft the line plan (`references/statements.md`, "The line plan") and show it in short form: one line per statement line, with its accounts as number ranges and names, such as "Accounts receivable, net: 1100 to 1140 (5 accounts)". Show the balance sheet and operations lines; say the cash flow follows them. Then ask only what applies:

1. **Undeposited Funds** (when the account exists): count it as cash? Recommend yes: NetSuite's statement of cash flows and Finsyte's Cash Flow template treat it as cash. With no, it is an other current asset.
2. **Intercompany** (when discovery found balance sheet candidates): are these the intercompany accounts, and which entity names appear on intercompany transactions? Offer the entities discovery found for each member (`references/excel-mechanics.md`, "Intercompany entity names"). Explain that Due from (to) affiliates shows one line per entity and that "Other entities" holds whatever doesn't match a name. Without names, or if the user prefers, Due from (to) affiliates lists the intercompany accounts instead, one row per account. Intercompany revenue, cost and expense accounts stay on the operations lines.
3. **Depreciation and amortization**: are these the accounts? With none found, ask whether the company has any.
4. **Distributions** (when equity accounts look like distributions, dividends or drawings): show them as a financing line?
5. **Wording and grouping**: the caption of each operating expense group (default: the NetSuite parent's name in sentence case, such as "Selling and marketing" for "Selling Expenses"), and any lines to combine. A section with more than eight lines: propose keeping the largest and folding the rest into its Other line.

Apply edits and show again only the sections that changed.

---

## Round 4: cover and presentation

One message, with defaults:

1. Company name on the cover and every page. Default: the subsidiary's legal name.
2. Prepared by. Default: empty.
3. Distribution. Default: `Board of Directors`.
4. Footer text and unaudited label. Defaults: `Confidential` and `Unaudited`; the label empty for audited figures.
5. Materiality amount. Default: 50,000. The exchange rate check on the cash flow pages flags an effect larger than the amount. (Inputs also keeps a materiality percentage, default 10%, for the user's own review; no check reads it.)

---

## The build plan

Show the plan and wait for a yes:

```
Board book plan
Workbook: <workbook name>. New sheets: Inputs, Cover, Index, 1 to <N>, and Lists (hidden).
Entity: <subsidiary>. Book: <book>. Reporting currency: <currency>.
Period: <fiscal year> period <n>, ending <date>. Prior year: <prior fiscal year>. <PeriodsPerYear> periods a year.

Pages
  1 to 3    Financial statements
  4 to 10   Trend analysis<, five-year page with <k> years | , no five-year page>
  11 to 13  Consolidating: <members>
  ...       By department: <members> and No department
  ...       Budget: <category>

Lines
  Balance sheet: <n> lines. Operations: <n> lines. Cash flow: follows them.
  Undeposited Funds: <cash | other current asset>. Intercompany: <accounts> by <entities>.
  Depreciation and amortization: <accounts>. Distributions: <accounts | none>.

Cover: <company>. Prepared by <name>. Distribution: <...>. Footer: "<footer> — <company> — <label>".

About <rows> rows and <count> Finsyte formulas. The first calculation can take several minutes.

Reply "build" to start, or tell me what to change.
```

Any reply other than a yes to building is an edit: apply it and show the plan again.
