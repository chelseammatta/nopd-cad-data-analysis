# nopd-cad-data-analysis
Analysis of NOPD 911 CAD data from 2019-2022 from the 3rd and 4th districts.
## Overview

This project analyzes New Orleans Police Department (NOPD) 911 Computer-Aided Dispatch (CAD) data spanning 2019 to 2022, focusing on the 3rd and 4th police districts. The goal is to understand police resource allocation, call volumes, and response patterns, especially around major events such as Mardi Gras and the COVID-19 pandemic.

## Project Objectives

- Analyze how 911 call volume changed during major events such as COVID-19 lockdowns and Mardi Gras
- Determine which call types and priorities were most likely to go unserved or receive delayed responses
- Examine trends in domestic violence and violent/property crime over time
- Identify geographic concentrations of specific incident types within the 3rd and 4th districts
- Support data-informed public safety decision-making in resource allocation and emergency response 

## Personal Motivation

As a former EMT who worked in the neighborhoods covered by the 3rd and 4th police districts, I approached this project with firsthand insight into the urgency and complexity of emergency response. My goal is to use data to support safer, more efficient public safety systems. 

## Project Status

- Cleaned and standardized 458,255 CAD records across 4 years (after filtering to the 3rd and 4th districts)
- Removed 11,213 DUPLICATE rows, 16,521 VOID rows, and 230 TEST INCIDENT rows
- Categorized 73 unique call types into 16 major categories
- Created four master tables:
  - All Calls (with served/unserved and self-initiated flags)
  - Unserved calls
  - Served, Self-Initiated
  - Served, Public-Initiated
- Response time metrics calculated (served only)
- Next steps: developing visualizations and detailed analysis

## Preliminary Observations

Early exploration of the cleaned dataset revealed several important patterns that shaped the direction of the project:

- **Self-initiated calls accounted for a large portion of all records**, but differed significantly in nature and response dynamics from public-initiated calls. This led to the decision to analyze them separately when calculating response times and evaluating unserved incidents.
- **Certain call types had multiple redundant or inconsistent labels**, such as different entries for assault, burglary, or alarms (i.e. aggravated assault, burglary (residence), or alarm (residential)). This complexity made clear the need to group similar call types into standardized categories for meaningful aggregation and trend analysis.
- **A significant number of calls lacked arrival timestamps**, signaling either officer non-response or poor data entry. These calls were flagged as unserved, and this insight influenced the decision to exclude them from response time calculation and analyze them as a separate group.
- **Timestamps varied in formatting across years**, especially in the raw CSV files. This required manual preprocessing in LibreOffice Calc to ensure BigQuery compatibility and led to the development of a consistent pipeline for timestamp standardization.

These early insights not only shaped the project's categorization system, filtering logic, and summary table design, but also emphasized the importance of separating data types (e.g., served vs unserved, public vs self-initiated) for reliable and interpretable results.

## Key Challenges & Decisions

- Managed inconsistent timestamp formats across years using LibreOffice Calc for SQL compatibility
- Removed ~23,000 invalid records (VOID, DUPLICATE, TEST INCIDENT) to improve data quality
- Designed a Served/Unserved flag based on officer arrival data to reflect response coverage
- Separated Self-Initiated vs Public-Initiated calls to preserve analytical clarity in response time metrics
- Recategorized ~250+ call types into 16 higher-level categories to support meaningful aggregation
- Verified and corrected geolocation columns to enable accurate mapping

## Findings & Visualizations

### Domestic Violence Spike During COVID-19 Lockdown

Based on my personal experience working in emergency response during the COVID-19 pandemic and lockdown, I hypothesized that domestic violence (DV) calls would show a sharp increase at the onset of lockdown, followed by a slow but steady decline back to pre-pandemic levels over time. However, the visualization revealed a different pattern. There was a clear spike in DV calls in March 2020, but the numbers returned to baseline by April 2020, suggesting the increase was temporary.

This abrupt return to baseline could be due to a range of factors. One possibility is that DV incidents were still occurring but not being reported — potentially due to isolation, fear, or lack of access to resources. To better understand this pattern, a deeper analysis using electronic police report (EPR) data from the same districts and time frame would be valuable. Comparing EPR and CAD data could reveal discrepancies in how incidents were reported or classified, and help determine whether the apparent decline in DV calls reflects a true decrease or a shift in reporting behavior.

---

More findings and visualizations will be added as the project progresses.

## Tools & Technologies

- **Google BigQuery (SQL):** data querying, transformation, and analysis
- **LibreOffice Calc and Google Sheets:** Preprocessing timestamps and verifying formatting  
- **Looker Studio / Tableau (planned):** Visualization and dashboard creation
- **GitHub:** Version control and public-facing documentation

## Repository Structure

- `data-prep-sql/` — SQL scripts for data cleaning and categorization  
- `analysis/` — SQL queries for creating master tables and calculating response times  
- `visualizations/` — Placeholder for visualization files and dashboard exports  
- `docs/` — Project summaries, notes, and final reports

## Sample SQL Queries

Below are examples of key SQL queries used in the project. More detailed scripts are available in the `data-prep-sql/` and `analysis/` folders.

### 1. Creating Served/Unserved Flag

```sql
-- Example query to create served boolean flag based on arrival time (officer never arrived means call was unserved)
SELECT *,
  CASE
    WHEN CleanTimeArrival IS NULL THEN FALSE
    ELSE TRUE
  END AS Served
FROM `project.dataset.table_name`
```
### 2. Calculating Response Times
```sql
-- Example query calculating dispatch to arrival time difference in seconds
SELECT *,
  TIMESTAMP_DIFF(CleanTimeArrival, CleanTimeDispatch, SECOND) AS DispatchToArriveTime,
  TIMESTAMP_DIFF(CleanTimeArrival, CleanTimeCreate, SECOND) AS CreateToArriveTime
FROM `project.dataset.served_calls_table`
```
## Notes

- Self-initiated calls comprise a significant portion of the dataset and are analyzed separately due to differences in response metrics.
- Geolocation data was verified and corrected to ensure accuracy in spatial analysis.

## How to Reproduce

To replicate this project or build on it: 

1. Download 2019-2022 CAD datasets from the [NOLA Open Data Portal](https://data.nola.gov/Public-Safety-and-Preparedness/Calls-for-Service-Basic-View/6mc5-nn7g)
2. Preprocess timestamps using LibreOffice Calc or a paid desktop version of Excel (due to file sizes >100mb), or spreadsheet tool of choice to ensure BigQuery compatibility (convert to DATETIME yyyy-mm-dd hh:mm:ss)
3. Import cleaned CSVs into BigQuery.
4. Follow the SQL scripts in the `data-prep-sql/` folder to merge, clean, and categorize the data.
5. Use the `analysis/` queries to calculate response times and generate master tables.
6. Optional: Use Tableau or Looker Studio to build visualizations using the exported master tables.

> Note: This project focuses on the 3rd and 4th police districts and filters records accordingly during preprocessing.

## Future Work

- Develop and publish visualizations analyzing call volumes, response times, and category trends
- Perform geographic mapping of incidents by police district
- Finalize report summarizing key findings and recommendations

## Data Sources

- [NOLA Open Data Portal - 911 Calls for Service](https://data.nola.gov/Public-Safety-and-Preparedness/Calls-for-Service-Basic-View/6mc5-nn7g)
- [City of New Orleans - COVID-19 Response Orders](https://nola.gov/health/coronavirus/safe-reopening/phases/)
- [Mardi Gras Parade Schedules (2019–2022)](https://www.mardigrasneworleans.com/parades/schedule)

## Skills Demonstrated

- SQL, Data Cleaning, Categorization, Data Modeling, Response Time Analysis, Geospatial Prep, Data Documentation

## Notes on Data Tables

- During this project, I created multiple intermediate and cleaned tables.
- Some tables may have been archived, renamed, or deleted after completed analysis.
- Queries reference those tables as they existed at the time of analysis.
- Query structure remains intact; this project is reproducible if table names reflect your own accurate ones.

*This project is ongoing. Updates and improvements will be regularly added to this repository.*
