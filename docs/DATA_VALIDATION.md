# Data Validation Report

The data comes from a Scaler assignment. This report documents how the Tableau dashboard (`dashboard/amazon_insights.twbx`) was checked against the source data (`data/Amazon_Data.xlsx`).

## Method

1. Extracted the data stored inside the packaged workbook and compared it, cell by cell, with the Excel **Orders** sheet.
2. Recomputed every headline figure and chart value directly from the data using Python (pandas) and compared them with the values shown in the workbook.
3. Reviewed the workbook's calculated fields, filters, parameters and dashboard actions to confirm each chart measures what its title says.
4. Ran data-quality checks on the source data.

## 1. Source data vs the dashboard's data extract

| Check | Result |
|---|---|
| Row count | 51,290 in Excel, 51,290 in the extract: **match** |
| Row IDs | Identical sets: **match** |
| Column values | 23 of 24 columns identical in every row: **match** |
| Product Name | 16 cells differ only by a trailing space (Excel has it, Tableau trimmed it). No effect on any chart. |

## 2. Headline figures

| Metric | Recomputed from data | Shown in the workbook | Result |
|---|---|---|---|
| Sales | 12,642,501.91 | 12,642,502 | Match |
| Profit | 1,467,457.29 | 1,467,457 | Match |
| Quantity | 178,312 | 178,312 | Match |
| Segment share: Consumer | 51.48% | not legible in the saved preview | Computed from data |
| Segment share: Corporate | 30.25% | 30.25% | Match |
| Segment share: Home Office | 18.27% | 18.27% | Match |

The "shown" values were read from the preview images Tableau stores inside the workbook and from a screenshot of the published dashboard.

## 3. Chart values in the published dashboard

The values printed on the charts in the published dashboard were compared with values recomputed from the data.

| Chart | Values compared | Result |
|---|---|---|
| Sales by market | All 7 markets: APAC 3,585,744, EU 2,938,089, US 2,297,201, LATAM 2,164,605, EMEA 806,161, Africa 783,773, Canada 66,928 | All match exactly |
| Profit margin by sub-category | Labelled bars: Paper 24.24%, Accessories 17.30%, Art 15.58%, Fasteners 13.85%, Chairs 9.35%, Tables -8.46% | All match |
| Top profitable customers (Top 25) | 13 labelled bars from 8,673 down to 4,980, and the customer names shown at ranks 2, 4, 6, 8 and 10 | All match |

## 4. Chart logic review

| View | Logic | Assessment |
|---|---|---|
| Profit margin by sub-category | `SUM(Profit) / SUM(Sales)` | Correct weighted margin |
| Sales over the years | `SUM(Sales)` by month of Order Date | Correct |
| Sales by market | `SUM(Sales)` by Market | Correct |
| Sales distribution by segment | % of total `SUM(Sales)` | Correct |
| Top profitable customers | Top N by `SUM(Profit)`, grouped by Customer Name (795 names) | Correct; N is controlled by a parameter |
| Sales vs Profit | `SUM(Sales)` vs `SUM(Profit)` per Product Name (3,788 products), classified loss (<0), low (0 to 1,000), high (>1,000): 677 / 2,666 / 445 | Correct |

## 5. Data quality of the source

| Check | Result |
|---|---|
| Missing values | Only Postal Code (41,296 rows); all are non-US rows. Every US row has a postal code. |
| Duplicate Row IDs / duplicate rows | None |
| Zero or negative Sales / Quantity | None |
| Ship Date earlier than Order Date | None |
| Discount range | 0% to 85% |
| Loss-making order lines | 12,544 (24.5% of lines), totalling -920,646 in profit |

## 6. Observations and limitations

- **Returns are not applied.** The Returns sheet lists 1,172 returned orders (1,173 rows, one order listed twice). They represent 6.5% of sales and 8.0% of profit in the Orders sheet. The dashboard shows gross figures.
- **Two IDs per customer name.** All 795 customer names appear under two Customer IDs (1,590 IDs in total): one used for APAC / EU / LATAM / US orders and one for Africa / EMEA orders. Customer-level views should group by name, as the customer chart does.
- **Market labels in Returns.** The Returns sheet labels the US market "United States", while Orders uses "US". Two returned orders also carry a different market in the two sheets.
- **People sheet.** It lists the region "AMEA", while Orders uses "EMEA", so a join on Region would drop that region. The sheet is not used in the dashboard.
- **Supporting worksheets.** The workbook also contains three worksheets that are not placed on the dashboard: *Avg shipping cost by shipping mode*, *Avg transaction margin by sub category* and *Customer by Region over Time*. The last one applies each customer's first-ever order date to every region the customer has bought from (96.7% of customer IDs bought in more than one region), so its per-region acquisition counts should be interpreted with care. The average row-level margin also differs from the weighted margin used on the dashboard (for example Tables: -24.2% average against -8.5% weighted).
- **Currency:** the source file does not state a currency; the values are in US dollars (USD), as confirmed by the author.
