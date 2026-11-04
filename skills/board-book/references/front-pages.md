# Front pages and workbook names

Read this before you write the Lists, Inputs, Cover and Index sheets. The book's sheets, in order: Inputs, Cover, Index, the report pages 1 to N, and Lists (hidden). Every report page reads its parameters from Inputs through the workbook names below, so a new period or subsidiary is one change on Inputs.

## Contents

- Lists
- Inputs
- Cover
- Index
- Workbook names

---

## Lists

A hidden sheet named `Lists`. Row 1 holds headings; each list spills down from row 2. The Inputs dropdowns read these lists, so they grow with NetSuite.

| Column | Heading | Row 2 formula | Name |
|---|---|---|---|
| A | Subsidiaries | `=FSN.Range.Subsidiaries()` | `ListSubsidiaries` |
| B | Subsidiaries, not consolidated | `=FSN.Range.Subsidiaries(,,,,FALSE)` | `ListSubsidiaryMembers` |
| C | Accounting books | `=FSN.Range.AccountingBooks()` | `ListBooks` |
| D | Currencies | `=FSN.Range.Currencies()` | `ListCurrencies` |
| E | Fiscal years, latest first | `=FSN.Range.Periods.FY(FALSE,FALSE)` | `ListFiscalYears` |
| F | Budget categories | `=FSN.Range.BudgetCategories()` | `ListBudgetCategories` |
| G | Departments | `=FSN.Range.Departments(,,Subsidiary,FALSE)` | `ListDepartments` |
| H | Classes | `=FSN.Range.Classifications(,,Subsidiary,FALSE)` | `ListClasses` |
| I | Locations | `=FSN.Range.Locations(,,Subsidiary,FALSE)` | `ListLocations` |
| J | Periods | `=SEQUENCE(PeriodsPerYear)` | `ListPeriods` |

Each name refers to the spill, such as `=Lists!$A$2#`. Columns G to I read the book's subsidiary, so write them after Inputs C5 holds it; column J reads the periods per year, so write it after Inputs C34 holds them. Leave out a segment column when the NetSuite account doesn't use the segment. The department, class and location lists include the `Parent (Hierarchy)` entries, so the user can pick a parent and its children as one column.

Columns from K are the discovery area while you build (`references/excel-mechanics.md`, "Discovery"). Clear it when the book is built.

---

## Inputs

Font Times New Roman 10 pt; B1 14 pt bold; section labels bold. A typed input is blue (`#0000FF`) on light yellow (`#FFFFCC`); a formula is black on white. Column widths: A 2.7, B 30.9, C 30.9, D 3.6, E 76.4, F 25.4, G 3, H 30.8, I 3.8, J 30.8, K 3.8, L 30.8, M 3.8, N 30.8. Gridlines off. Print area B1 to N and the last used row, landscape, one page wide.

E1 holds "Return to Index", a link to `Index!A1`.

| Row | B | C | Name | Dropdown |
|---|---|---|---|---|
| 3 | **Entity** | | | |
| 4 | Company name | Typed | `CompanyName` | |
| 5 | Subsidiary | Typed | `Subsidiary` | `ListSubsidiaries` |
| 6 | Scope | `=IF(ISNUMBER(SEARCH("(Consolidated)",C5)),"Consolidated","Subsidiary")` | `Scope` | |
| 7 | Accounting book | Typed | `Book` | `ListBooks` |
| 8 | Reporting currency | Typed | `ReportCurrency` | `ListCurrencies` |
| 9 | **Periods** | | | |
| 10 | Current year | `=YEAR(C17)`, format `0` | `CurYear` | |
| 11 | Current fiscal year | Typed, such as `FY 2025` | `CurFY` | `ListFiscalYears` |
| 12 | Prior year | `=C10-1`, format `0` | `PriorYear` | |
| 13 | Prior fiscal year | `=IFERROR(INDEX(ListFiscalYears,MATCH(C11,ListFiscalYears,0)+1),"")` | `PriorFY` | |
| 14 | Period | Typed number | `Period` | `ListPeriods` |
| 15 | Quarter | `=ROUNDUP(C14/3,0)` | `Quarter` | |
| 16 | Prior quarter-end period | `=IF(C15=1,C34,(C15-1)*3)` | `PriorQtrPeriod` | |
| 17 | Period end date | `=FSN.PeriodToDate(C11,C14)`, format `mmmm d, yyyy` | `PeriodEnd` | |
| 18 | Prior-year period end date | `=FSN.PeriodToDate(C13,C14)`, same format | `PriorPeriodEnd` | |
| 19 | Prior quarter-end date | `=FSN.PeriodToDate(IF(C15=1,C13,C11),C16)`, same format | `PriorQtrEnd` | |
| 20 | Month range | `PTD` | `RangeMTD` | |
| 21 | Quarter range | `QTD` | `RangeQTD` | |
| 22 | Year range | `YTD` | `RangeYTD` | |
| 23 | Balance sheet movement range | `RTD` | `RangeBSMove` | |
| 24 | Budget category | Typed; empty when the book has no budget pages | `BudgetCat` | `ListBudgetCategories` |
| 25 | Period range: inception to date | `ITD` | `RangeITD` | |
| 26 | **Materiality** | | | |
| 27 | Materiality amount | Typed, default `50000`, format `"$"#,##0` | `MatAmt` | |
| 28 | Materiality percentage | Typed, default `10%`, format `0.0%` | `MatPct` | |
| 29 | Book checks | `=IFERROR(IF(COUNTIF(<G>,"OK")+COUNTIF(<G>,"Review")<<pages>,"Error",IF(COUNTIF(<G>,"Review")>0,"Review","OK")),"Error")`, where `<G>` is `Index!$G$5:$G$<last page row>` and `<pages>` the number of report pages, so a page whose status can't be read counts as Error; with the status colors of the checks | `BookChecks` | |
| 30 | **Refresh** | | | |
| 31 | Last data refresh | `=FSN.LastDataRefresh()`, format `mmmm d, yyyy h:mm AM/PM` | `RefreshTime` | |
| 32 | Footer text | Typed, default `Confidential` | `FooterText` | |
| 33 | Unaudited text | Typed, default `Unaudited`; empty for audited figures | `UnauditedText` | |
| 34 | Periods per fiscal year | Typed, from discovery | `PeriodsPerYear` | |
| 36 | **Earlier fiscal years** | | | |
| 37 | Two years prior | `=IFERROR(INDEX(ListFiscalYears,MATCH(C11,ListFiscalYears,0)+2),"")` | `PriorFY2` | |
| 38 | Three years prior | the same with `+3` | `PriorFY3` | |
| 39 | Four years prior | the same with `+4` | `PriorFY4` | |
| 41 | **Accounts** | | | |
| 42 | Intercompany accounts | Typed, accounts joined with `^`; empty without intercompany accounts | `IntercompanyAccounts` | |
| 43 | Undeposited Funds counted as cash | Typed account, or empty | `CashExtraAccounts` | |
| 45 | **Affiliates (intercompany entity names)** | | | |
| 46 on | Affiliate 1, 2, … | Typed entity names, one per row | `Affiliates`, over all the rows; with no names, C46 alone, empty, so the Cover's formula still reads it | |

`FSN.LastDataRefresh` recalculates every time Excel calculates, by design. No Finsyte call reads row 31, so it slows nothing.

**Quarter.** Rows 15, 16 and 19 assume three periods per quarter. For a year of 13 periods, ask the user which period ends each quarter, type Quarter and Prior quarter-end period in C15 and C16 as values, and tell the user to update them with the period.

**Column lists**, from row 3 down in H, J, L and N:

| Column | Row 3 (bold) | Rows 4 on | Name | Dropdown |
|---|---|---|---|---|
| H | Consolidating columns | One subsidiary per row, the elimination subsidiary last | `SubsidiaryColumns` | `ListSubsidiaryMembers`, warning style |
| J | Department columns | One department per row | `DepartmentColumns` | `ListDepartments`, warning style |
| L | Class columns | One class per row | `ClassColumns` | `ListClasses`, warning style |
| N | Location columns | One location per row | `LocationColumns` | `ListLocations`, warning style |

Each name covers exactly the rows filled. Leave out the column of a list whose pages the book doesn't have. The warning style lets the user type a value the list doesn't show yet. Changing an entry re-points that page column; adding or removing an entry needs the page rebuilt, so a blank entry is never a column (a blank segment would return every value, and a blank Subsidiary the user's default subsidiary).

**How the pages are built**, in E3 down, bold headings and indented lines, one line per statement section: what each line holds, which lines are plug lines and what they are measured against, and which pages show each statement. Write it from the confirmed line plan, in words a controller reads, such as "Selling and marketing: accounts under 6050 Selling Expenses; Other selling and marketing is the rest of 6050". F holds the pages, such as `Pages 2, 6, 7, 8`.

---

## Cover

Font Times New Roman. Gridlines off. Print area C1:H21, landscape, fit to one page, centered horizontally. Widths: A 2.7, C 8, D 26, E 48, G 26, H 8, I 14.6, R 16, S 20. Rows centered across C:H use Center Across Selection.

| Cell | Content | Format |
|---|---|---|
| C3 | `=CompanyName` | 24 pt bold, white on `#1F3864`, across C:H, row height 40 |
| C5 | `=IF(OR(COUNTA(Affiliates)=0,COUNTA(Affiliates)>8),Subsidiary,TEXTJOIN("  ·  ",TRUE,Affiliates))` | 12 pt bold italic, `#1F3864` on `#D9D9D9`, across C:H, row height 24 |
| C7 | `Board of Directors — Executive Summary` | 20 pt bold, `#1F3864`, across C:H |
| C8 | `Financial Reporting Package` | 13 pt italic, `#1F3864`, across C:H |
| C9 | `=IF(Period=PeriodsPerYear,"Quarter and Year Ended ",IF(Period=1,"Month Ended ",IF(Period=3,"Quarter Ended ",IF(MOD(Period,3)=0,"Three and "&CovMonthsWord&" Months Ended ","Month and "&CovMonthsWord&" Months Ended "))))&CovPElong` | 13 pt bold, `#2E75B6`, across C:H |
| D12, E12 | `Prepared by`, typed | D bold; D12:E16 12 pt on `#F2F2F2`; typed values blue |
| D13, E13 | `Distribution`, typed (default `Board of Directors`) | |
| D14, E14 | `Reporting period`, `=CovPElong` | |
| D15, E15 | `Accounting book`, `=Book` | |
| D16, E16 | `Last data refresh`, `=TEXT(RefreshTime,"mmmm d, yyyy")&" at "&TEXT(RefreshTime,"h:mm AM/PM")` | |
| C19 | `=FooterText&" — "&CompanyName&" — "&UnauditedText` | 10 pt italic, `#404040` on `#D9D9D9`, across C:H |
| R1 | `Go to Index`, a link to `Index!A1` | |

**Heading dates**, not printed: S2 `Heading dates (linked to Inputs; not printed)` (bold, `#808080`), labels in S (hidden column), formulas in T. Each T cell has the workbook name shown.

| Row | S | T | Name |
|---|---|---|---|
| 3 | Current year | `=CurYear`, format `0` | `CovCY` |
| 4 | Prior year | `=PriorYear`, format `0` | `CovPY` |
| 5 | Current period end | `=PeriodEnd`, format `mmmm d, yyyy` | `CovPE` |
| 6 | Prior-year period end | `=PriorPeriodEnd`, same | `CovPPE` |
| 7 | Prior quarter end | `=PriorQtrEnd`, same | `CovPQE` |
| 8 | Period end (month day,) | `=TEXT(PeriodEnd,"mmmm d,")` | `CovPEmd` |
| 9 | Prior quarter end (month day,) | `=TEXT(PriorQtrEnd,"mmmm d,")` | `CovPQEmd` |
| 10 | Month name | `=TEXT(PeriodEnd,"mmmm")` | `CovMonth` |
| 11 | Quarter name | `=CHOOSE(Quarter,"First","Second","Third","Fourth")` | `CovQtrName` |
| 12 | Months in year to date | `=Period` | |
| 13 | Months word | `=CHOOSE(Period,"One","Two","Three","Four","Five","Six","Seven","Eight","Nine","Ten","Eleven","Twelve","Thirteen")` | `CovMonthsWord` |
| 14 | Period end (long) | `=TEXT(PeriodEnd,"mmmm d, yyyy")` | `CovPElong` |
| 15 | Year-to-date basis | `=IF(Period=PeriodsPerYear,"Year Ended",IF(Period=1,"Month Ended",CovMonthsWord&" Months Ended"))` | `CovYTDBasis` |
| 16 | Quarter-to-date basis | `=CHOOSE(MOD(Period-1,3)+1,"Month Ended","Two Months Ended","Three Months Ended")` | `CovQTDBasis` |
| 17 | Year-to-date layout label | `=IF(Period=PeriodsPerYear,"Year Ended","Year to Date")` | `CovYTDLayout` |
| 18 | Year-to-date phrase (current) | `="the "&LOWER(CovYTDBasis)&" "&TEXT(PeriodEnd,"mmmm d, yyyy")` | `CovYTDPhrase` |
| 19 | Year-to-date phrase (prior) | `="the "&LOWER(CovYTDBasis)&" "&TEXT(PriorPeriodEnd,"mmmm d, yyyy")` | `CovYTDPhrasePY` |
| 20 | Quarter phrase (current) | `="the "&LOWER(CovQTDBasis)&" "&TEXT(PeriodEnd,"mmmm d, yyyy")` | `CovQTDPhrase` |
| 21 | Quarter phrase (prior) | `="the "&LOWER(CovQTDBasis)&" "&TEXT(PriorPeriodEnd,"mmmm d, yyyy")` | `CovQTDPhrasePY` |
| 22 | Month, quarter and year layout label | `=IF(Period=PeriodsPerYear,"Month, Quarter and Year Ended","Month, Quarter and Year to Date")` | `CovMQYLayout` |

---

## Index

Font Times New Roman. Gridlines off. Column A hidden; widths B 10, C 34, D 52, E 44, F 52, G 10. Freeze panes at row 5. Print area B1:F and the footer row, rows 1 to 4 repeated, landscape, one page wide.

| Row | Content |
|---|---|
| 1 | B1 `=CompanyName`, 13 pt bold, across B:F |
| 2 | B2 `Index of Reports`, 12 pt bold, across B:F |
| 3 | B3 `=CovPElong`, across B:F |
| 4 | B `Page`, C `Entity`, D `Report Type (Y Axis)`, E `Layout (X Axis)`, F `Information Source`, G `Checks`: bold, thin top and bottom border |
| From 5 | For each section that has pages: a section row (B the section name from `references/pages.md`, bold, `#1F3864`, medium bottom border), then one row per page of the section |
| Page row | B the sheet name, as a link to `'<sheet>'!A1`, centered on `#CCEEFF`; C `='<sheet>'!$C$1`; D `='<sheet>'!$C$2`; E `='<sheet>'!$C$3`; F `Finsyte Functions`; G `='<sheet>'!$F$<the page's Overall row>` with the status colors. C to F on `#CCEEFF`. |
| Then | An `OTHER` section row and an Inputs row: B `Inputs` linked to `Inputs!A1`, C `=CompanyName`, D `Inputs`, E `Parameters, dates and column lists`, F `Management inputs; FSN period and refresh functions` |
| Footer | Two rows below: B `=FooterText&" — "&CompanyName&" — "&UnauditedText`, 9 pt italic, `#404040`, across B:F |

Column G is outside the print area: it shows the preparer each page's checks, and `BookChecks` on Inputs counts it.

---

## Workbook names

The pages and the front sheets use these workbook-level names. Before you build, check that the workbook has none of them and no sheet named Inputs, Cover, Index, Lists or a number (`references/excel-mechanics.md`, "Before you start").

`Affiliates`, `Book`, `BookChecks`, `BudgetCat`, `CashExtraAccounts`, `ClassColumns`, `CompanyName`, `CovCY`, `CovMonth`, `CovMonthsWord`, `CovMQYLayout`, `CovPE`, `CovPElong`, `CovPEmd`, `CovPPE`, `CovPQE`, `CovPQEmd`, `CovPY`, `CovQTDBasis`, `CovQTDPhrase`, `CovQTDPhrasePY`, `CovQtrName`, `CovYTDBasis`, `CovYTDLayout`, `CovYTDPhrase`, `CovYTDPhrasePY`, `CurFY`, `CurYear`, `DepartmentColumns`, `FooterText`, `IntercompanyAccounts`, `ListBooks`, `ListBudgetCategories`, `ListClasses`, `ListCurrencies`, `ListDepartments`, `ListFiscalYears`, `ListLocations`, `ListSubsidiaries`, `ListSubsidiaryMembers`, `LocationColumns`, `MatAmt`, `MatPct`, `Period`, `PeriodEnd`, `PeriodsPerYear`, `PriorFY`, `PriorFY2`, `PriorFY3`, `PriorFY4`, `PriorPeriodEnd`, `PriorQtrEnd`, `PriorQtrPeriod`, `PriorYear`, `Quarter`, `RangeBSMove`, `RangeITD`, `RangeMTD`, `RangeQTD`, `RangeYTD`, `RefreshTime`, `ReportCurrency`, `Scope`, `Subsidiary`, `SubsidiaryColumns`, `UnauditedText`.
