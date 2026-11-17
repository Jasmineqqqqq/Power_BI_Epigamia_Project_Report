# Epigamia Sales & Returns Analytics

A six-page Power BI project analysing sales, returns, products, channels, and regional performance for Epigamia. Power Query prepares the data, a star-schema model connects business dimensions, and DAX measures support interactive analysis.

## Project objectives

- Track sales value, return value, net sales, and return rates.
- Identify leading products, categories, channels, and regions.
- Investigate return reasons and areas with elevated return exposure.
- Provide transaction-level detail for further investigation.

## Tools

- Microsoft Power BI Desktop
- Power Query and M
- DAX
- CSV source data

## Report pages

| Report | Purpose | Main analysis |
| --- | --- | --- |
| 1. Overview | Business performance summary | Sales and returns KPIs, daily trend, leading categories and channels, performance snapshot |
| 2. Product & Variant | Product portfolio analysis | Category and SKU performance, contribution, product hierarchy, product comparison |
| 3. Channel | Channel performance | Net sales, return rates, contribution, Fresh/Ambient mix, daily trend |
| 4. Region & City | Geographic performance | Zone and city sales, geographic contribution, returns, return-rate comparison |
| 5. Sales Return | Return analysis | Daily returns, reasons, category and product exposure, return rates |
| 6. Transaction Detail | Detailed investigation | Transaction records, invoice and return references, outlet and product filtering |

## Dataset

The reviewed dataset covers **1–31 March 2024**.

| Attribute | Value |
| --- | ---: |
| Transaction rows | 337,066 |
| Invoice rows (INV) | 254,238 |
| Return rows (RETR) | 82,828 |
| Products | 64 |
| Product categories | 20 |
| Channels | 14 |
| Outlets | 11,250 |
| Zones | 4 |

A transaction row represents a source line, not necessarily a unique invoice. SourceRowID identifies imported rows; it is not a business document number.

## Data preparation and model

StagingSales supplies the cleaned source data. FactSales holds transaction fields and numeric amounts. Dimension tables provide descriptive attributes for filtering and grouping.

| Dimension | Relationship key in FactSales |
| --- | --- |
| DimDate | Date |
| DimProduct | Sku_Code |
| DimCustomer | Outlet_ID |
| DimChannel | Outlet_Type |
| DimGeography | GeographyKey |
| DimDepot | Depo |
| DimReason | ReturnReason |

The model uses active, single-direction, many-to-one relationships from FactSales to these dimensions. DataCoverage and SnapshotMetrics support coverage information and the overview snapshot.

DimDate covers the April–March financial year. A full date table does not imply that transactions exist for every month.

## KPI definitions

| KPI | Definition |
| --- | --- |
| Sales Value | Amount on INV records |
| Return Value | Negative of Amount on RETR records, displayed as a positive return amount |
| Net Sales | Sum of signed Amount across transactions |
| Sales Quantity | Quantity on INV records |
| Return Quantity | Negative of Quantity on RETR records |
| Return Value Rate | Return Value / Sales Value |
| Return Quantity Rate | Return Quantity / Sales Quantity |
| Net Sales Lakh | Net Sales / 100,000 |
| Net Sales Crore | Net Sales / 10,000,000 |

### Core DAX measures

These definitions reflect the reviewed model:

```dax
Sales Value =
CALCULATE(
    SUM(FactSales[Amount]),
    KEEPFILTERS(FactSales[Document_Type] = "INV"),
    REMOVEFILTERS(DimReason)
)

Return Value =
-CALCULATE(
    SUM(FactSales[Amount]),
    KEEPFILTERS(FactSales[Document_Type] = "RETR")
)

Net Sales =
SUM(FactSales[Amount])

Sales Quantity =
CALCULATE(
    SUM(FactSales[Quantity]),
    KEEPFILTERS(FactSales[Document_Type] = "INV"),
    REMOVEFILTERS(DimReason)
)

Return Quantity =
-CALCULATE(
    SUM(FactSales[Quantity]),
    KEEPFILTERS(FactSales[Document_Type] = "RETR")
)

Return Value Rate =
DIVIDE([Return Value], [Sales Value])

Return Quantity Rate =
DIVIDE([Return Quantity], [Sales Quantity])

Net Sales Lakh =
DIVIDE([Net Sales], 100000)

Net Sales Crore =
DIVIDE([Net Sales], 10000000)
```

Format rates as percentages, quantities as whole numbers, and financial measures as currency. DIVIDE returns blank when the denominator is zero.

Sales measures remove return-reason filtering so that selecting a reason can compare those returns against the relevant sales base. Net Sales remains responsive to the current filter context and should not be interpreted as a complete sales-minus-returns result when only return reasons are selected.

## Reconciliation totals

For all March 2024 records, with report filters cleared:

| Metric | Expected value |
| --- | ---: |
| Sales Value | ₹184,707,884.37 |
| Return Value | ₹20,636,785.97 |
| Net Sales | ₹164,071,098.40 |
| Sales Quantity | 4,249,541 |
| Return Quantity | 475,266 |
| Net Quantity | 3,774,275 |
| Return Value Rate | 11.17% |
| Return Quantity Rate | 11.18% |

INR is the reporting assumption; the source has no separate currency field.

## Key findings

These findings describe the complete March 2024 dataset, not every possible slicer selection.

- QCOM contributed approximately **40.46% of net sales**, with a **2.20% return value rate**.
- Greek Yogurt 85 contributed approximately **44.57% of net sales**.
- South contributed approximately **34.90% of net sales**, the largest zone share.
- North and West together accounted for approximately **77.83% of return value**, compared with **64.66% of sales value**.
- Market Return represented approximately **58.38% of return value**; Order Cancellation represented **41.62%**.
- WGT had the highest channel return value rate at approximately **43.03%**, while GT-T had the highest absolute channel return value.

High return rates identify areas to investigate; they do not establish a root cause.

## Open and refresh the project

1. Open **EPIGAMIA_ASSIGNMENT_NOV26.pbix** in Power BI Desktop.
2. Extract the source CSV from the supplied data package.
3. Select **Home → Transform data**.
4. Open **Manage Parameters** and update **pSourceFile** to the complete CSV file path, including its filename.
5. Example path: `C:\Epigamia_Assignment\Data\Sales_Returns.csv`.
6. Confirm that StagingSales previews the expected columns and rows.
7. Select **Close & Apply**, then **Refresh**.
8. Clear report filters and compare the results with the reconciliation totals above.
9. Test page navigation, slicers, chart selections, and transaction drillthrough.
10. Save a working copy after validating the results.

The original supporting package, **Epigamia_PowerBI_Assignment.zip**, includes the CSV, Power Query scripts, DAX definitions, a theme file, documentation, and a PBIP project. The current PBIX is the report file to open for this version.

## Using the reports

- Start with Overview, then use the other pages to investigate products, channels, regions, and returns.
- Use date and dimension slicers to narrow the analysis.
- Hover over chart marks for additional tooltip information.
- Clear slicer selections to return to the complete dataset.
- Use Transaction Detail to inspect records behind an analytical finding.
- Product drillthrough is configured in the reviewed file; confirm its behaviour in Power BI Desktop.

## Design

The report follows an Epigamia-inspired style with a dark header, light page background, white visual panels, and restrained accent colours.

Suggested consistent styling:

- Font: Segoe UI.
- Page size: 1400 × 1100 pixels.
- Page margin: 24 pixels.
- Visual spacing: 16 pixels.
- Navy: `#1C3177`.
- Pale blue: `#C7DCF9`.
- Rose: `#CA7077`.
- Peach: `#FEA691`.
- Cream background: `#FAF4F2`.
- White panels: `#FFFFFF`.
- Use dark, readable chart titles and consistent currency units.
- Use thin KPI accent strips and minimal borders or shadows.

These are design recommendations, not a claim that every visual in the current PBIX already uses them.

## Validation status and outstanding checks

Source totals, core measure logic, and model relationships were reviewed. Power BI Desktop interactions and refresh behaviour require final testing.

Before submission:

- Correct Return Value formatting to currency and use ₹ consistently instead of $.
- Format contribution and return-rate measures as percentages.
- Remove the inherited Top 5 SKU filter from the Channel daily trend.
- Use a shared scale for comparable sales and net-sales lines.
- Set transaction-level SourceRowID, Quantity, and Amount to **Don't summarize** in the detail table.
- Clear the saved blank selection in the return-number slicer so returns remain visible.
- Remove the erroneous duplicate City Zone Label calculated column from DataCoverage; retain the valid DimGeography column.
- Replace Net Sales in the return-reason matrix with an appropriate return metric.
- Correct copied visual titles and unrelated tooltip fields.
- Confirm slicer search, drillthrough, navigation, and reset behaviour in Desktop.

This README documents the reviewed project; it does not certify that the above corrections have been applied.

## Assumptions and limitations

- Only March 2024 transactions are available. Year-over-year growth, month-over-month comparisons, and seasonality cannot be established from this extract.
- Return rates compare activity within the selected period; they do not match each return to its original invoice cohort.
- Return rates can legitimately exceed 100% when period returns exceed period sales. Do not cap them.
- The exact tax and discount basis of Amount is not established. Net Sales is not profit.
- Geographic labels are preserved from the source; apparent mapping inconsistencies require business confirmation.
- Static insight text describes the full dataset unless explicitly implemented as a dynamic measure.

## Author

Jasmine Fernandes

