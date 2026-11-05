# Report pages

Read this before you write a report page. It gives the frame every page shares, then each page of the page list: title, the statement it shows, and its columns with their parameters and headings. The rows of each statement are in `references/statements.md`. Names such as `CurFY` and `CovPEmd` are the workbook names in `references/front-pages.md`.

## Contents

- Page list
- Page frame
- Value formulas
- Pages 1 to 3: financial statements
- Pages 4 to 10: trend analysis
- Pages 11 to 13: consolidating
- Pages 14 to 22: by department, class and location
- Pages 23 to 26: budget

---

## Page list

The page number is the book's page order. A page left out of the book closes the gap: sheet names are consecutive numbers in the order below, and the Index lists the sheets that exist.

| Page | ID | Title (C2) | Layout (C3) | Statement | Index section | Tab color |
|---|---|---|---|---|---|---|
| 1 | `BS-MQY` | Balance Sheets | `Month, Quarter and Year End` | Balance sheet | FINANCIAL STATEMENTS | `#2E75B6` |
| 2 | `IS-MQY` | Statements of Operations | `=CovMQYLayout` | Operations | FINANCIAL STATEMENTS | `#2E75B6` |
| 3 | `CF-MQY` | Statements of Cash Flows | `=CovMQYLayout` | Cash flow | FINANCIAL STATEMENTS | `#2E75B6` |
| 4 | `BS-MONTH` | Balance Sheets | `Current Year by Month` | Balance sheet | TREND ANALYSIS | `#548235` |
| 5 | `BS-ACTIVITY` | Balance Sheets | `Current Year by Month, Activity` | Balance sheet | TREND ANALYSIS | `#548235` |
| 6 | `IS-MONTH` | Statements of Operations | `Current Year by Month` | Operations | TREND ANALYSIS | `#548235` |
| 7 | `IS-CUM` | Statements of Operations | `Current Year by Month, Cumulative` | Operations | TREND ANALYSIS | `#548235` |
| 8 | `IS-5YR` | Statements of Operations | `=IF(Period=PeriodsPerYear,"Five Years Ended","Prior Four Years and Year to Date")` | Operations | TREND ANALYSIS | `#548235` |
| 9 | `CF-MONTH` | Statements of Cash Flows | `Current Year by Month` | Cash flow | TREND ANALYSIS | `#548235` |
| 10 | `CF-CUM` | Statements of Cash Flows | `Current Year by Month, Cumulative` | Cash flow | TREND ANALYSIS | `#548235` |
| 11 | `BS-CONSOL` | Consolidating Balance Sheets | `Current and Prior Year End` | Balance sheet | CONSOLIDATING | `#C55A11` |
| 12 | `IS-CONSOL` | Consolidating Statements of Operations | `=CovYTDLayout` | Operations | CONSOLIDATING | `#C55A11` |
| 13 | `CF-CONSOL` | Consolidating Statements of Cash Flows | `=CovYTDLayout` | Cash flow | CONSOLIDATING | `#C55A11` |
| 14 | `BS-DEPT` | Balance Sheets by Department | `Current and Prior Year End` | Balance sheet | BY DEPARTMENT, CLASS AND LOCATION | `#7F6000` |
| 15 | `IS-DEPT` | Statements of Operations by Department | `=CovYTDLayout` | Operations | same | `#7F6000` |
| 16 | `CF-DEPT` | Statements of Cash Flows by Department | `=CovYTDLayout` | Cash flow | same | `#7F6000` |
| 17 to 19 | `BS-CLASS`, `IS-CLASS`, `CF-CLASS` | as 14 to 16, "by Class" | as 14 to 16 | as 14 to 16 | same | `#7F6000` |
| 20 to 22 | `BS-LOC`, `IS-LOC`, `CF-LOC` | as 14 to 16, "by Location" | as 14 to 16 | as 14 to 16 | same | `#7F6000` |
| 23 | `IS-BUD-MQY` | Statements of Operations | `="Budget: "&CovMQYLayout` | Operations | BUDGET | `#7030A0` |
| 24 | `IS-BUD-CONSOL` | Consolidating Statements of Operations | `="Budget: "&CovYTDLayout` | Operations | BUDGET | `#7030A0` |
| 25 | `IS-BUD-MONTH` | Statements of Operations | `Budget: Current Year by Month` | Operations | BUDGET | `#7030A0` |
| 26 | `IS-BUD-CUM` | Statements of Operations | `Budget: Current Year by Month, Cumulative` | Operations | BUDGET | `#7030A0` |

**Title.** Pages other than the Consolidating ones start the title with the scope, so a single subsidiary reads "Balance Sheets" and a consolidated one "Consolidated Balance Sheets": `=IF(Scope="Consolidated","Consolidated ","")&"Balance Sheets"`. The Consolidating pages keep their title as written.

---

## Page frame

Every report page has this frame. Page 1 has one more heading row, so its body starts one row lower.

**Columns**

| Column | Holds | Shown |
|---|---|---|
| A | Level: `=FSN.GetListInfo($B$20,$B<r>,$A$20)` on rows with an account or code in B, `1` on every other body row. Every body row has a value in A: the band and the collapsed-group formulas count the visible cells of A. | Hidden |
| B | The row's account number, `id_<internal id>` for an account without a number, an `FSN_` code, or several codes and accounts joined with `^`. Text format (`@`). | Hidden |
| C | Caption. D and E stay empty in the body so captions run across them. | Shown |
| E | Below the footer: each check's status. | Not printed |
| F onward | Value columns, change columns and narrow spacer columns, as each page lists. | Shown |

The print area runs from C1 to the page's last spacer column and down to the footer row. The column after the print area holds "Return to Index" in row 1, a link to `Index!A1`.

Widths: A 3.6, B 10, C 14.6, D 14.6, E 10.3, value and change columns 14.6, spacer columns 1.4.

**Rows**

| Row | Holds |
|---|---|
| 1 | C1 `=CompanyName` (13 pt bold) |
| 2 | C2 title (12 pt bold, color `#156082`) |
| 3 | C3 layout (12 pt bold, color `#156082`) |
| 4 | C4 `=CovPElong` (14 pt bold) |
| 5 | C5 `=CovRefresh` on the left, and `Page <n>` typed right-aligned in the last value or change column before the final spacer (9 pt italic, color `#156082`, thin bottom border across the print width, row height 24). Type the page number: a formula that reads the sheet name uses `CELL`, which is volatile. |
| 6 to 19 | The parameter block, hidden. C6 "Parameters (not printed)". The labels in E, one per row: 6 Subsidiary, 7 Fiscal year, 8 Period, 9 Period range, 10 Book, 11 Item, 12 Class, 13 Department, 14 Location, 15 Entity, 16 Transaction type, 17 Posted, 18 Currency, 19 Balance range (balance sheet and cash flow pages) or Budget category (budget pages; empty on other operations pages). |
| 20 | A20 `Level`, B20 `GL Accounts` (the list name and property the Level formulas read). From F: the first heading row. |
| 21 | Second heading row. |
| 22 | Page 1: third heading row. Every other page: the first body row. |
| Footer | Two rows below the last body row (the last KPI row): C `=FooterText&" — "&CompanyName&" — "&UnauditedText`, 9 pt italic, color `#404040`, centered across the print width. The print area ends here; the checks block starts two rows lower (`references/statements.md`, "Checks"). |

Titles C1 to C4 are centered across the print width (Center Across Selection, not merged cells). Heading cells are 10 pt bold with a thin bottom border; a heading over several columns is centered across them; single-column headings are right-aligned. Member names in headings wrap, with row 21 tall enough for two lines.

**Parameter cells.** In every value column: row 6 Subsidiary, 7 Fiscal year, 8 Period, 9 Period range and 19 as the page lists; row 10 `=Book`; row 18 `=ReportCurrency`. Rows 11 to 17 stay empty unless the page sets a segment in them. Change and spacer columns have no parameters.

**Freeze panes** at the first body row. Gridlines off. Landscape, margins 0.5 inch, centered horizontally, fit to one page wide and one tall. A page with more than 20 columns from C to the last print column fits one page tall and runs as many pages wide as it needs, with columns C:E repeated on each printed page.

---

## Value formulas

`<c>` is the value column and `<r>` the row. Every value cell holds one Finsyte call, with nothing around it but the row's sign (a leading minus, `references/statements.md`, "Signs").

| Column kind | Formula |
|---|---|
| Actual | `=FSN.GLAccountBalance(<c>$6,<c>$7,<c>$8,<c>$9,$B<r>,<c>$10,<c>$11,<c>$12,<c>$13,<c>$14,<c>$15,<c>$16,<c>$17,<c>$18)` |
| Actual, balance range | The Actual formula with `<c>$19` in place of `<c>$9`. Used where `references/statements.md` says "balance range". |
| Budget | `=FSN.BudgetAccountBalance(<c>$6,<c>$7,<c>$8,<c>$9,$B<r>,<c>$19,<c>$10,<c>$11,<c>$12,<c>$13,<c>$14,<c>$15)`, never negated |

Change columns are arithmetic on the value cells of the same row. `% Change` and `% Variance` cells read `=IFERROR(<difference>/ABS(<base>),0)`.

**Number formats**

| Use | Format |
|---|---|
| Amount | `_(* #,##0_);_(* \(#,##0\);_(* "–"_);_(@_)` |
| Amount with $ | `_("$"* #,##0_);_("$"* \(#,##0\);_("$"* "–"_);_(@_)` |
| Percent | `0.0%;\(0.0%\);"–"` |
| Ratio | `0.00"x";\(0.00"x"\);"–"` |
| Heading year | `0` |

A change column takes the format of its row (`$ Change` the amount format, `% Change` the percent format).

---

## Pages 1 to 3: financial statements

### Page 1, `BS-MQY`

Three heading rows (20 to 22); the body starts at row 23 and the panes freeze there. Row 19 is Balance range, `=RangeITD`, in every value column.

| Col | Kind | Sub (6) | Fiscal year (7) | Period (8) | Range (9) | Row 20 | Row 21 | Row 22 |
|---|---|---|---|---|---|---|---|---|
| F | Actual | `=Subsidiary` | `=CurFY` | `=Period` | `=RangeYTD` | `Month over Month` across F:I | `=CovPEmd` | `=CovCY` |
| G | Actual | `=Subsidiary` | `=IF(Period=1,PriorFY,CurFY)` | `=IF(Period=1,PeriodsPerYear,Period-1)` | `=RangeYTD` | | `=INDEX(MonthNames,MONTH(FSN.PeriodToDate(G$7,G$8)))&" "&DAY(FSN.PeriodToDate(G$7,G$8))&","` | `=YEAR(FSN.PeriodToDate(G$7,G$8))` |
| H | `=F<r>-G<r>` | | | | | | | `$ Change` |
| I | `=IFERROR(H<r>/ABS(G<r>),0)` | | | | | | | `% Change` |
| J | Spacer | | | | | | | |
| K | Actual | as F | as F | as F | as F | `Quarter over Quarter` across K:N | `=CovPEmd` | `=CovCY` |
| L | Actual | `=Subsidiary` | `=IF(PriorQtrPeriod>=Period,PriorFY,CurFY)` | `=PriorQtrPeriod` | `=RangeYTD` | | `=CovPQEmd` | `=YEAR(CovPQE)` |
| M, N | `=K-L`, `=IFERROR(M/ABS(L),0)` | | | | | | | `$ Change`, `% Change` |
| O | Spacer | | | | | | | |
| P | Actual | as F | as F | as F | as F | `=CovYTDBasis` across P:S | `=CovPEmd` | `=CovCY` |
| Q | Actual | `=Subsidiary` | `=PriorFY` | `=PeriodsPerYear` | `=RangeYTD` | | `=INDEX(MonthNames,MONTH(FSN.PeriodToDate(Q$7,Q$8)))&" "&DAY(FSN.PeriodToDate(Q$7,Q$8))&","` | `=YEAR(FSN.PeriodToDate(Q$7,Q$8))` |
| R, S | `=P-Q`, `=IFERROR(R/ABS(Q),0)` | | | | | | | `$ Change`, `% Change` |
| T | Spacer | | | | | | | |

### Page 2, `IS-MQY`

| Col | Kind | Fiscal year | Period | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|---|
| F | Actual | `=CurFY` | `=Period` | `=RangeMTD` | `="Month Ended "&CovPEmd` across F:I | `=CovCY` |
| G | Actual | `=PriorFY` | `=Period` | `=RangeMTD` | | `=CovPY` |
| H, I | `=F-G`, `=IFERROR(H/ABS(G),0)` | | | | | `$ Change`, `% Change` |
| J | Spacer | | | | | |
| K, L | Actual, as F and G with `=RangeQTD` | | | | `=CovQTDBasis&" "&CovPEmd` across K:N | `=CovCY`, `=CovPY` |
| M, N | `=K-L`, `=IFERROR(M/ABS(L),0)` | | | | | `$ Change`, `% Change` |
| O | Spacer | | | | | |
| P, Q | Actual, as F and G with `=RangeYTD` | | | | `=CovYTDBasis&" "&CovPEmd` across P:S | `=CovCY`, `=CovPY` |
| R, S | `=P-Q`, `=IFERROR(R/ABS(Q),0)` | | | | | `$ Change`, `% Change` |
| T | Spacer | | | | | |

Subsidiary is `=Subsidiary` in every value column.

### Page 3, `CF-MQY`

Page 2's columns and headings, with Range `=RangeMTD` in F and G, `=RangeQTD` in K and L, and `=RangeBSMove` in P and Q. Row 19, Balance range, is `=RangeYTD` in every value column.

---

## Pages 4 to 10: trend analysis

`N` is `PeriodsPerYear`. The month columns run from F for N columns (F to Q for 12 periods). Each month column's Period cell holds its period number, typed. Every month heading in row 21 is `=INDEX(MonthNames,MONTH(FSN.PeriodToDate(<c>$7,<c>$8)))`, the name of the month the column's fiscal period ends in, so the heading follows the company's fiscal calendar. `DATE(<year>,<period>,1)` gives the calendar month instead: in a July to June year it puts January over July. Below, the column letters are those of a 12-period year; with 13 periods every column after the months moves one to the right.

`INDEX($F<r>:$Q<r>,Period)` is the month column at the book's period.

**Months after the book's period.** On pages 4 to 7, 9 and 10, the actual month columns after the book's period would show open periods to the board. Give the body of the month columns, F to Q, one conditional format `=F$8>Period` with the number format `;;;`, so those months show blank and reappear as the book rolls forward. The budget pages (25, 26) show every month: the full-year budget is the point.

### Page 4, `BS-MONTH`

| Col | Kind | Fiscal year | Period | Range | Row 19 | Row 20 | Row 21 |
|---|---|---|---|---|---|---|---|
| F to Q | Actual | `=CurFY` | 1 to N | `=RangeYTD` | `=RangeITD` | F: `=CurFY&", Month-End Balances"` across the months | month |
| R | Actual | `=PriorFY` | `=Period` | `=RangeYTD` | `=RangeITD` | `=CovPEmd` | `=CovPY` |
| S | Spacer | | | | | | |
| T | `=INDEX($F<r>:$Q<r>,Period)-R<r>` | | | | | | `$ Change` |
| U | `=IFERROR(T<r>/ABS(R<r>),0)` | | | | | | `% Change` |
| V | Spacer | | | | | | |

### Page 5, `BS-ACTIVITY`

| Col | Kind | Fiscal year | Period | Range | Row 19 | Row 20 | Row 21 |
|---|---|---|---|---|---|---|---|
| F to Q | Actual | `=CurFY` | 1 to N | `=RangeMTD` | `=RangeMTD` | F: `=CurFY&", Change During Month"` | month |
| R | Actual | `=CurFY` | `=Period` | `=RangeBSMove` | `=RangeBSMove` | | `Total` |
| S | Spacer | | | | | | |
| T | Actual | `=PriorFY` | `=Period` | `=RangeBSMove` | `=RangeBSMove` | `=CovYTDBasis&" "&CovPEmd` | `=CovPY` |
| U | `=R<r>-T<r>` | | | | | | `$ Change` |
| V | `=IFERROR(U<r>/ABS(T<r>),0)` | | | | | | `% Change` |
| W | Spacer | | | | | | |

### Page 6, `IS-MONTH`

| Col | Kind | Fiscal year | Period | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|---|
| F to Q | Actual | `=CurFY` | 1 to N | `=RangeMTD` | F: `=CurFY` | month |
| R | Actual | `=CurFY` | `=Period` | `=RangeYTD` | | `Total` |
| S | Spacer | | | | | |
| T | Actual | `=PriorFY` | `=Period` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd` | `=CovPY` |
| U, V | `=R-T`, `=IFERROR(U/ABS(T),0)` | | | | | `$ Change`, `% Change` |
| W | Spacer | | | | | |

### Page 7, `IS-CUM`

| Col | Kind | Fiscal year | Period | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|---|
| F to Q | Actual | `=CurFY` | 1 to N | `=RangeYTD` | F: `=CurFY&", Cumulative by Month"` | month |
| R | Actual | `=PriorFY` | `=Period` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd` | `=CovPY` |
| S | Spacer | | | | | |
| T | `=INDEX($F<r>:$Q<r>,Period)-R<r>` | | | | | `$ Change` |
| U | `=IFERROR(T<r>/ABS(R<r>),0)` | | | | | `% Change` |
| V | Spacer | | | | | |

### Page 8, `IS-5YR`

| Col | Kind | Fiscal year | Period | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|---|
| F | Actual | `=PriorFY4` | `=PeriodsPerYear` | `=RangeYTD` | `Fiscal Years` across F:I | `=F$7` |
| G | Actual | `=PriorFY3` | `=PeriodsPerYear` | `=RangeYTD` | | `=G$7` |
| H | Actual | `=PriorFY2` | `=PeriodsPerYear` | `=RangeYTD` | | `=H$7` |
| I | Actual | `=PriorFY` | `=PeriodsPerYear` | `=RangeYTD` | | `=I$7` |
| J | Spacer | | | | | |
| K | Actual | `=CurFY` | `=Period` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd` | `=K$7` |
| L | `=K<r>-I<r>` | | | | | `$ Change` |
| M | `=IFERROR(L<r>/ABS(I<r>),0)` | | | | | `% Change` |
| N | `=IFERROR(IF(Period=PeriodsPerYear,(K<r>/F<r>)^(1/4),(I<r>/F<r>)^(1/3))-1,0)` | | | | | `="CAGR "&F$7&"–"&IF(Period=PeriodsPerYear,K$7,I$7)` |
| O | Spacer | | | | | |
| P | `=IFERROR(K<r>/K$<TR>,0)` | | | | `% of Total Revenues` across P:S | `=K$7` |
| Q | `=IFERROR(I<r>/I$<TR>,0)` | | | | | `=I$7` |
| R | `=IFERROR(H<r>/H$<TR>,0)` | | | | | `=H$7` |
| S | `=IFERROR(G<r>/G$<TR>,0)` | | | | | `=G$7` |
| T | Spacer | | | | | |

`<TR>` is the Total revenues row. P to S use the percent format. The year headings are the fiscal year names, so a fiscal year that ends mid-calendar-year is labeled by NetSuite's own name.

**Fewer than five fiscal years.** Before you build the page, check that `PriorFY2`, `PriorFY3` and `PriorFY4` on Inputs hold fiscal years (they read the fiscal year list and are empty past the first year in NetSuite). Leave out the columns of the years that don't exist, from the left, together with their `% of Total Revenues` column, and set the CAGR to the earliest year shown: `(K/<earliest>)^(1/<years between>)` at year end, `(I/<earliest>)^(1/<years between>)` otherwise. With fewer than two full prior years, leave page 8 out and say why.

### Page 9, `CF-MONTH`

| Col | Kind | Fiscal year | Period | Range | Row 19 | Row 20 | Row 21 |
|---|---|---|---|---|---|---|---|
| F to Q | Actual | `=CurFY` | 1 to N | `=RangeMTD` | `=RangeYTD` | F: `=CurFY` | month |
| R | Actual | `=CurFY` | `=Period` | `=RangeBSMove` | `=RangeYTD` | | `Total` |
| S | Spacer | | | | | | |
| T | Actual | `=PriorFY` | `=Period` | `=RangeBSMove` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd` | `=CovPY` |
| U, V | `=R-T`, `=IFERROR(U/ABS(T),0)` | | | | | | `$ Change`, `% Change` |
| W | Spacer | | | | | | |

### Page 10, `CF-CUM`

| Col | Kind | Fiscal year | Period | Range | Row 19 | Row 20 | Row 21 |
|---|---|---|---|---|---|---|---|
| F to Q | Actual | `=CurFY` | 1 to N | `=RangeBSMove` | `=RangeYTD` | F: `=CurFY&", Cumulative by Month"` | month |
| R | Actual | `=PriorFY` | `=Period` | `=RangeBSMove` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd` | `=CovPY` |
| S | Spacer | | | | | | |
| T | `=INDEX($F<r>:$Q<r>,Period)-R<r>` | | | | | | `$ Change` |
| U | `=IFERROR(T<r>/ABS(R<r>),0)` | | | | | | `% Change` |
| V | Spacer | | | | | | |

---

## Pages 11 to 13: consolidating

One member column per entry of `SubsidiaryColumns` on Inputs, from F, in list order. The elimination subsidiary, when the user keeps it, is the list's last entry.

| Column | Kind | Subsidiary (6) | Fiscal year | Period | Range (9) | Row 19 | Row 20 | Row 21 |
|---|---|---|---|---|---|---|---|---|
| Each member | Actual | `=INDEX(SubsidiaryColumns,<k>)` | `=CurFY` | `=Period` | statement range | statement row 19 | first member: the member heading, across the members | `=<c>$6` |
| Spacer | | | | | | | | |
| Consolidated | Actual | `=Subsidiary` | `=CurFY` | `=Period` | statement range | statement row 19 | `Consolidated` across this and the next column | `=CovCY` |
| Prior year | Actual | `=Subsidiary` | `=PriorFY` | `=Period` | statement range | statement row 19 | | `=CovPY` |
| % Change | `=IFERROR((<Consolidated><r>-<Prior year><r>)/ABS(<Prior year><r>),0)` | | | | | | | `% Change` |
| Spacer | | | | | | | | |

| Page | Range (9) | Row 19 | Member heading (row 20) |
|---|---|---|---|
| 11, balance sheet | `=RangeYTD` | `=RangeITD` | `=CovPEmd&" "&CovCY` |
| 12, operations | `=RangeYTD` | empty | `=CovYTDBasis&" "&CovPEmd&" "&CovCY` |
| 13, cash flow | `=RangeBSMove` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd&" "&CovCY` |

Row 18, Currency, is `=ReportCurrency` in every value column, so each subsidiary is translated to the reporting currency and the columns add up.

---

## Pages 14 to 22: by department, class and location

One member column per entry of the page's list on Inputs (`DepartmentColumns`, `ClassColumns` or `LocationColumns`), from F. The member goes in the segment's parameter row: 13 Department, 12 Class, 14 Location.

| Column | Kind | Subsidiary | Segment row | Fiscal year | Range, row 19 | Row 20 | Row 21 |
|---|---|---|---|---|---|---|---|
| Each member | Actual | `=Subsidiary` | `=INDEX(<list>,<k>)` | `=CurFY` | as the statement (table above) | first member: the member heading (table above), across the members | `=<c>$<segment row>` |
| No segment | Actual | `=Subsidiary` | `<None>` | `=CurFY` | same | `No` | `Department`, `Class` or `Location` |
| All other | Only when the columns leave out a top-level value. On every row that holds a Finsyte formula, body and checks block: `=<Total><r>-SUM(<first member><r>:<No segment><r>)`. Every other row: as in a value column. | empty | empty | empty | empty | `All Other` | `Departments`, `Classes` or `Locations` |
| Spacer | | | | | | | |
| Total | Actual | `=Subsidiary` | empty | `=CurFY` | same | `Total` across Total and Prior year | `=CovCY` |
| Prior year | Actual | `=Subsidiary` | empty | `=PriorFY` | same | | `=CovPY` |
| % Change | `=IFERROR((<Total><r>-<Prior year><r>)/ABS(<Prior year><r>),0)` | | | | | | `% Change` |
| Spacer | | | | | | | |

Period is `=Period` and Currency `=ReportCurrency` in every value column. The No column passes `<None>`, the transactions with no value in the segment, so the members and the No column add up to the Total when the members don't overlap. When the user picks only some top-level values, the All other column holds the values left out, so the columns still add up to the Total.

---

## Pages 23 to 26: budget

Row 19, Budget category, is `=BudgetCat` in every budget column. Actual columns leave row 19 empty.

### Page 23, `IS-BUD-MQY`

| Col | Kind | Fiscal year | Period | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|---|
| F | Actual | `=CurFY` | `=Period` | `=RangeMTD` | `="Month Ended "&CovPEmd&" "&CovCY` across F:I | `Actual` |
| G | Budget | `=CurFY` | `=Period` | `=RangeMTD` | | `Budget` |
| H | `=F<r>-G<r>` | | | | | `$ Variance` |
| I | `=IFERROR(H<r>/ABS(G<r>),0)` | | | | | `% Variance` |
| J | Spacer | | | | | |
| K to N | as F to I with `=RangeQTD` | | | | `=CovQTDBasis&" "&CovPEmd&" "&CovCY` across K:N | as F to I |
| O | Spacer | | | | | |
| P to S | as F to I with `=RangeYTD` | | | | `=CovYTDBasis&" "&CovPEmd&" "&CovCY` across P:S | as F to I |
| T | Spacer | | | | | |

### Page 24, `IS-BUD-CONSOL`

One budget column per entry of `SubsidiaryColumns`, leaving out the elimination subsidiary.

| Column | Kind | Subsidiary | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|
| Each member | Budget | `=INDEX(SubsidiaryColumns,<k>)` | `=RangeYTD` | first member: `="Budget, "&CovYTDBasis&" "&CovPEmd&" "&CovCY` across the members | `=<c>$6` |
| Spacer | | | | | |
| Consolidated budget | Budget | `=Subsidiary` | `=RangeYTD` | `Consolidated` across this and the next column | `Budget` |
| Consolidated actual | Actual | `=Subsidiary` | `=RangeYTD` | | `Actual` |
| $ Variance | `=<actual><r>-<budget><r>` | | | | `$ Variance` |
| % Variance | `=IFERROR(<$ Variance><r>/ABS(<budget><r>),0)` | | | | `% Variance` |
| Spacer | | | | | |

Fiscal year `=CurFY` and Period `=Period` throughout.

### Page 25, `IS-BUD-MONTH`

| Col | Kind | Fiscal year | Period | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|---|
| F to Q | Budget | `=CurFY` | 1 to N | `=RangeMTD` | F: `=CurFY&" Budget"` | month |
| R | Budget | `=CurFY` | `=Period` | `=RangeYTD` | | `Total` |
| S | Spacer | | | | | |
| T | Actual | `=CurFY` | `=Period` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd&" "&CovCY` | `Actual` |
| U | `=T<r>-R<r>` | | | | | `$ Variance` |
| V | `=IFERROR(U<r>/ABS(R<r>),0)` | | | | | `% Variance` |
| W | Spacer | | | | | |

### Page 26, `IS-BUD-CUM`

| Col | Kind | Fiscal year | Period | Range | Row 20 | Row 21 |
|---|---|---|---|---|---|---|
| F to Q | Budget | `=CurFY` | 1 to N | `=RangeYTD` | F: `=CurFY&" Budget, Cumulative by Month"` | month |
| R | Actual | `=CurFY` | `=Period` | `=RangeYTD` | `=CovYTDBasis&" "&CovPEmd&" "&CovCY` | `Actual` |
| S | Spacer | | | | | |
| T | `=R<r>-INDEX($F<r>:$Q<r>,Period)` | | | | | `$ Variance` |
| U | `=IFERROR(T<r>/ABS(INDEX($F<r>:$Q<r>,Period)),0)` | | | | | `% Variance` |
| V | Spacer | | | | | |
