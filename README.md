# Is the Regional Minimum Wage Keeping Up with the Cost of Living in the Philippines?

## Problem Statement and Hypotheses
The Philippines has no single national minimum wage. Each of the 17 Regional Tripartite Wages and Productivity Boards (RTWPBs) issues wage orders for its own region, and these orders are reviewed by the National Wages and Productivity Commission (NWPC). 

Because wage orders are issued infrequently while prices rise every month, a nominal increase can still leave workers with less purchasing power than before. This project asks whether regional minimum wages have kept pace with regional price increases and the cost of a basic standard of living, and whether the answer relates to regional poverty.

### Problem statement:
Between January 2018 and December 2025, did regional daily minimum wages keep up with regional price levels and with the family poverty threshold, and are regions where the wage covers more of the threshold the ones with lower poverty?

### Hypotheses:
**H1 (real wage erosion)** In most regions, the real minimum wage (nominal wage deflated by the regional Consumer Price Index, in 2018 pesos) in December 2025 is not significantly higher than in January 2018, and it declines in the months between wage orders.

**H2 (threshold coverage)** The wage coverage ratio (estimated monthly minimum wage income divided by the monthly poverty threshold of a family of five) is below 1.0 in most regions, and it declined between the 2018, 2021 and 2023 poverty estimates.

**H3 (poverty link)** Regions with a higher coverage ratio have lower poverty incidence among families, after controlling for the underemployment rate.

### Objectives
1. Build a region-by-month panel of nominal and real minimum wages from public data.
2. Measure real wage growth and erosion between wage orders for each region.
3. Compare wages against the regional poverty threshold and test its relationship with poverty incidence.

## Datasets Used
All datasets are real-world public data. Access date for all links: October 27, 2026.

### Dataset 1: Regional daily minimum wage rates and wage orders
• Source: National Wages and Productivity Commission (NWPC), Department of Labor and Employment. Regional wage rate summaries and RTWPB wage orders.
• Link: https://nwpc.dole.gov.ph/ncr/
• Contribution: the main variable. Nominal daily minimum wage by region and sector (non-agriculture, with 10 or more workers; non-agriculture, with fewer than 10 workers and retail/service; agriculture), with each wage order's effectivity date.
• Note: the data is published as wage orders and summary tables, not as one clean file, so it must be compiled by hand and coded into a table. Third-party sites that mirror wage orders are used only to cross-check, not as the cited source.

### Dataset 2: Consumer Price Index by region (2018=100)
• Source: Philippine Statistics Authority (PSA), OpenSTAT, Retail Price Survey for the generation of the CPI.
• Link: https://openstat.psa.gov.ph (Prices, Consumer Price Index, 2018=100 database: https://openstat.psa.gov.ph/PXWeb/pxweb/en/DB/DB__2M__PI__CPI__2018/?tablelist=true)
https://data.worldbank.org/indicator/FP.CPI.TOTL.ZG?locations=PH

• Contribution: the deflator. Monthly regional CPI, all items, used to convert nominal wages into real wages and to estimate the poverty threshold between survey years.

### Dataset 3: Poverty threshold and poverty incidence by region (2018, 2021, 2023)
• Source: PSA, 2023 Full Year Official Poverty Statistics (based on the Family Income and Expenditure Survey).
•Links:https://psa.gov.ph/sites/default/files/phdsd/Press%20Release_2023%20Full%20Year%20Official%20Poverty%20Statistics.pdf (Table 1: monthly poverty threshold per family of five and poverty incidence among families, by region) and https://psa.gov.ph/sites/default/files/phdsd/2023%20FY%20Official%20Poverty%20Statistics%20Publication_15August2024.pdf  (full publication)
• Contribution: the benchmark for what a basic living costs, and the outcome variable for H3.

### Dataset 4: Labor Force Survey, underemployment rate by region (control)
• Source: PSA, Labor Force Survey (LFS), regional statistical tables, available through https://openstat.psa.gov.ph/PXWeb/pxweb/en/DB/DB__3S__C11/0153K2BURP0.px (Labor and Employment)
• Example regional table: https://rssomimaropa.psa.gov.ph/system/files/attachment-dir/2024%20July%20LFS%20Preliminary_SR%20Stat%20Table.pdf
• Contribution: control variable for H3. Regions with higher underemployment may have higher poverty rates regardless of wage levels.

### Final dataset design
• Unit of observation: region × wage category × month, January 2018 to December 2025 (96 months).
• Expected size: 17 regions × up to 3 wage categories × 96 months, about 4,900 rows, above the 2,000-row requirement.
• Poverty variables exist only for 2018, 2021 and 2023, so poverty incidence (H3) is analyzed at the region-year level, while real wage analysis (H1) uses the full monthly panel.
Key assumptions
• A minimum wage earner works 26 days per month (sensitivity checks at 22 and 30 days).
• The poverty threshold is for a family of five, so the coverage ratio shows how much of that threshold one minimum wage earner can cover.
• Thresholds for months outside survey years are estimated by scaling the nearest survey-year threshold by the regional CPI, and are labeled as estimates.
• PSA's 2023 poverty tables list the Negros Island Region separately; regions are harmonized to the 17 NWPC regions using a documented crosswalk.

## Pipeline
The four raw sources are loaded into pandas, pass through five transform steps in order, and are exported as one CSV. The whole pipeline is implemented in Python (pandas) in one notebook, with a Markdown cell before each step:

### Extract
1. Wage orders and NWPC summary tables are read and coded into a CSV with one row per region, wage category and wage order (order number, effectivity date, daily rate, tranche if any).
2. CPI by region is downloaded from OpenSTAT as CSV.
3. Poverty threshold and incidence by region (2018, 2021, 2023) are read from the PSA Table 1.
4. LFS regional underemployment rates are read from the PSA statistical tables.

### Load
Each raw file is loaded into its own pandas DataFrame, with raw copies kept unchanged. Column names, region names and dates are standardized, and every region is mapped to a common region code using a region lookup table.

### Transform
• Filtering: keep January 2018 to December 2025; remove national or aggregate rows (e.g., "Philippines"); remove domestic worker (kasambahay) wage orders; keep private sector establishment rates.
• Aggregation: where a region has several rates by area or class, use the highest-tier rate as the headline rate (the lowest tier is used as a robustness check); aggregate quarterly LFS data to the periods needed; compute annual average wages and CPI to match the poverty survey years.
• Imputation: expand each wage order into monthly rows and carry the rate forward until the next order, using a day-weighted average in the month of change; carry quarterly LFS values across the months of the quarter; estimate monthly poverty thresholds for non-survey months using CPI scaling.
• Merging: merge wages and CPI on region code and year-month; merge LFS on region code and quarter; merge poverty data on region code and year.
• Feature engineering: real wage (nominal wage ÷ CPI × 100), real wage index (January 2018 = 100), estimated monthly wage income (daily wage × 26), estimated poverty threshold, coverage ratio (monthly wage ÷ threshold), months since last wage order, and an erosion flag (real wage below its level at the last wage order).

### Output
The cleaned and merged table is exported as final_dataset.csv, then used for the descriptive analysis (trends, rankings by region) and diagnostic analysis (correlation and regression of poverty incidence on coverage ratio, with underemployment as a control).

## References
Consumer Price Index for All Income Households by Commodity Group (2018=100): January 2018 - December 2025. (n.d.). Retrieved October 27, 2026, from https://openstat.psa.gov.ph/PXWeb/pxweb/en/DB/DB__2M__PI__CPI__2018/0012M4ACP09.px/?rxid=9f96a5b0-cea8-4e13-98c2-3028c5944551 

NCR – National Wages and Productivity Commission. (2026, October 5). https://nwpc.dole.gov.ph/ncr/ 

Official Poverty Statistics of the Philippines. (2023). Philippine Statistics Authority. Retrieved October 27, 2026, from https://psa.gov.ph/sites/default/files/phdsd/2023%20FY%20Official%20Poverty%20Statistics%20Publication_15August2024.pdf 

TABLE 11.14 Underemployed Persons and Underemployment Rate by Region, Philippines: 2017 to 2023 (Based on Past Week Reference Period; In Thousands Except Rates). (n.d.). Philippine Statistics Authority. Retrieved October 27, 2026, from https://openstat.psa.gov.ph/PXWeb/pxweb/en/DB/DB__2M__PI__CPI__2018/?tablelist=true 

