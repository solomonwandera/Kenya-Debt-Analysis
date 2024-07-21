## Kenya-Debt-Analysis

### Project Overview
This project aims to analyze and forecast debt levels using a dataset containing information on domestic debt, external debt, total debt, and GDP, spanning multiple years and months. By leveraging SQL for data processing and Power BI for visualization, the project seeks to provide insights into debt trends and their relationship with GDP.

### Project Objectives
1. Data Integration and Cleaning
2. Trend Analysis
3. Visualisation

### Data Sources
[World Bank](https://databank.worldbank.org/reports.aspx?source=2&country=KEN#)

### Tools
- [SQL](https://www.mysql.com/)
- [Power BI](https://www.microsoft.com/en-us/power-platform/products/power-bi/downloads)
- MS Excel

  
### 1.Data Integration and Cleaning
a) Database creation
- At this stage we define a foudnation of storing our data by establishinga dedicated database to store all the information and also define the structure of the debt and GDP tables.
```sql
CREATE DATABASE kenya_economy;

USE kenya_economy;

CREATE TABLE debt(
  year VARCHAR(4) NOT NULL,
  month VARCHAR(10) NOT NULL,
  domestic_debt DECIMAL(25,2),
  external_debt DECIMAL(25,2)
);

CREATE TABLE annual_gdp(
  year VARCHAR(13) NOT NULL,
  gdp_usd DECIMAL(25,2)
);
```
b) Data Enhancements
- We establish a total debt column with the sum of domestic and external debt  for each record, ensuring data completents and accuracy
```sql
ALTER TABLE debt 
ADD total_debt DECIMAL(25,2);

UPDATE debt
SET total_debt = domestic_debt + external_debt;
```
c) Data cleaning and transformation
- To facilitate consistency , we extract and create a new column for the actual year in the gdp table to facilitate consistentcy and easier joins with the debt table.
- We identify and remove any debt records with years that don't exist in the gdp table to prevent potential inconsistencies.
```sql
ALTER TABLE annual_gdp
ADD actual_year VARCHAR(4) NOT NULL;

UPDATE annual_gdp
SET actual_year = LEFT(year,4);

SELECT year 
FROM debt 
WHERE year NOT IN (SELECT actual_year FROM annual_gdp);

DELETE FROM debt
WHERE year = 2024;
```
d) Additional data integration
- We set up a new table 'fx_rates" to store foregin exchange rate data that can be integrated into the analysis to provided additional context and ennhancements.

``` sql
CREATE TABLE fx_rates (
year VARCHAR(50),
exchange_rate VARCHAR(50));
```

- To enable accurate sorting , filtering and tiem based analysis, we add a "calculated date" column in our debt table.
```sql

-- This column is for storing date values

ALTER TABLE debt
ADD COLUMN calculated_date VARCHAR(50);

-- This fills the calculated date column with date strings.
-- We set the day to "01" to create a valdu date string since only 'month' and 'year' are available in the original dataset.

UPDATE debt
SET calculated_date = CONCAT('01/', month, '/', year);

-- We then update the calculated date column by first creating a temporary table for convertion of the string dates into acttual date formats.

CREATE TEMPORARY TABLE new_dates AS
SELECT calculated_date, STR_TO_DATE(calculated_date, '%d/%m/%Y') AS converted_date
FROM debt;

UPDATE debt 
JOIN new_dates ON debt.calculated_date = new_dates.calculated_date
SET debt.calculated_date = new_dates.converted_date;
```
f) Data documentation
- For clear understanding and reference, we define the structure and purpose of each column in our tables using a data dictionary.

```sql
CREATE TABLE data_dictionary (
  Table_Name VARCHAR(50) NOT NULL,
  Column_Name VARCHAR(50) NOT NULL,
  data_type VARCHAR(50) NOT NULL,
  Description VARCHAR(250),
  Is_Nullable BOOLEAN NOT NULL
);

INSERT INTO data_dictionary (Table_Name, Column_Name, Data_Type, Description, IS_Nullable)
VALUES 
  ('debt', 'year', 'VARCHAR(4)', 'Year of recorded debt data', FALSE),
  ('debt', 'month', 'VARCHAR(10)', 'Month of recorded debt data', FALSE),
  ('debt', 'domestic_Debt', 'DECIMAL(25,2)', 'Amount of domestic debt in local currency- KES', FALSE),
  ('debt', 'external_Debt', 'DECIMAL(25,2)', 'Amount of external debt in local currency - KES', FALSE),
  ('debt', 'total_Debt', 'DECIMAL(25,2)','Sum of domestic and external debt', FALSE),
  ('annual_gdp', 'gdp_usd', 'DECIMAL(25,2)', 'GDP at purchasers prices measures the total value of goods and services produced within a country, excluding depreciation and some subsidies. ItS reported in current US dollars using exchange rates.', FALSE),
  ('annual_gdp', 'actual_year', 'VARCHAR(4)', 'Year of recorded gdp data', FALSE)
  ('fx_rates', 'year', 'VARCHAR(4)', 'Year of recorded exchange rate data', FALSE)
  ('fx_rates', 'exchange_rate', 'VARCHAR(4)', 'Recorded annual KES/USD exchange rate', FALSE);
```

### 2.Trend Analysis
- In this section we conduct explaratory data analysis with the aim of analysing the trends of debt over the years, with key questions around
  - How has the national debt evolved over time ?
  - Are there any siginificant increases or decreases on specific years or months
  - What patterns or cycles can be observed in the debt data
    - Trend of domestic,external and total debt over time
    - ![Trend of debt](https://github.com/user-attachments/assets/42ca78d3-db5d-4979-a7db-aace389d53c8)

      

  
