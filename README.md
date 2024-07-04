# Loan-Disbursement-Reporting-Project
## Overview
The Bank Loan Report Project leverages SQL for data manipulation and Power BI for visualization, providing stakeholders with actionable insights into loan applications and approvals. It features optimized queries using CTEs for efficient processing and dynamic query handling for automatic data updates. The Loan Disbursement Reporting Project aims to analyze and visualize loan data efficiently and dynamically. This project involves creating optimized SQL queries to extract and transform data, followed by developing interactive and insightful Power BI dashboards to monitor Key Performance Indicators (KPIs) related to loan disbursements, statuses, and trends.

## Tools
### SQL: 
For data extraction, transformation, and loading (ETL).
### Power BI: 
For data visualization and dashboard creation.
### PostgreSQL: 
As the database management system.

## Dashboards
The Power BI report consists of three main pages:
### Summary: 
Provides an overview of key metrics and trends.
### Overview: 
Offers detailed insights into loan statuses, funding amounts, and amounts received.
### Detail: 
Allows for in-depth analysis with interactive filters and slicers.

## Implementation
### SQL Queries
### Good Loan Disbursement Overview

WITH Good_Loans AS (
    SELECT * FROM loans
    WHERE loan_status IN ('Fully Paid', 'Current')
), Total_Loans AS (
    SELECT COUNT(id) AS Total_Loans FROM loans
)
SELECT
    COUNT(GL.id) AS Good_Loan_Applications,
    (COUNT(GL.id) * 100.0) / TL.Total_Loans AS Good_Loan_Percentage,
    SUM(GL.loan_amount) AS Good_Loan_Funded_Amount,
    SUM(GL.total_payment) AS Good_Loan_Amount_Received
FROM Good_Loans GL, Total_Loans TL;

### Bad Loan Disbursement Overview

WITH Bad_Loans AS (
    SELECT * FROM loans
    WHERE loan_status = 'Charged Off'
), Total_Loans AS (
    SELECT COUNT(id) AS Total_Loans FROM loans
)
SELECT
    COUNT(BL.id) AS Bad_Loan_Applications,
    (COUNT(BL.id) * 100.0) / TL.Total_Loans AS Bad_Loan_Percentage,
    SUM(BL.loan_amount) AS Bad_Loan_Funded_Amount,
    SUM(BL.total_payment) AS Bad_Loan_Amount_Received
FROM Bad_Loans BL, Total_Loans TL;

### Loan Status Overview

SELECT
    loan_status,
    COUNT(id) AS LoanCount,
    SUM(total_payment) AS Total_Amount_Received,
    SUM(loan_amount) AS Total_Funded_Amount,
    AVG(int_rate * 100) AS Interest_Rate,
    AVG(dti * 100) AS DTI
FROM loans
GROUP BY loan_status;

WITH MaxDate AS (
    SELECT MAX(issue_date) AS max_issue_date
    FROM loans
)
SELECT 
    loan_status, 
    SUM(total_payment) AS MTD_Total_Amount_Received, 
    SUM(loan_amount) AS MTD_Total_Funded_Amount 
FROM loans
JOIN MaxDate ON EXTRACT(YEAR FROM issue_date) = EXTRACT(YEAR FROM max_issue_date)
AND EXTRACT(MONTH FROM issue_date) = EXTRACT(MONTH FROM max_issue_date)
GROUP BY loan_status;

### Monthly Loan Report Overview

WITH MaxYear AS (
    SELECT MAX(EXTRACT(YEAR FROM issue_date)) AS max_year
    FROM loans
)
SELECT 
    EXTRACT(MONTH FROM issue_date) AS Month_Number, 
    TO_CHAR(issue_date, 'Month') AS Month_Name, 
    COUNT(id) AS Total_Loan_Applications,
    SUM(loan_amount) AS Total_Funded_Amount,
    SUM(total_payment) AS Total_Amount_Received
FROM loans
JOIN MaxYear ON EXTRACT(YEAR FROM issue_date) = max_year
GROUP BY EXTRACT(MONTH FROM issue_date), TO_CHAR(issue_date, 'Month')
ORDER BY EXTRACT(MONTH FROM issue_date);

### Employee Length Report

SELECT 
    emp_length AS Employee_Length, 
    COUNT(id) AS Total_Loan_Applications,
    SUM(loan_amount) AS Total_Funded_Amount,
    SUM(total_payment) AS Total_Amount_Received
FROM loans
GROUP BY emp_length
ORDER BY ARRAY_POSITION(
    ARRAY['< 1 year', '1 year', '2 years', '3 years', '4 years', '5 years', '6 years', '7 years', '8 years', '9 years', '10+ years'],
    emp_length);
    
## Power BI Implementation
## Data Preparation

### Change Data Type: Change address_state data type to State or Province.

### Create Date Table:

DateTable = CALENDAR(MIN('public loans'[issue_date]), MAX('public loans'[issue_date]))
Extract Month:

Month = FORMAT('Date Table'[Date], "MMM")
MonthNumber = MONTH('Date Table'[Date])

## KPIs Measures

### Total Loan Applications:
Total Loan Applications = COUNT('public loans'[id])
MTD Loan Applications:

MTD Loan Applications = CALCULATE([Total Loan Applications], DATESMTD('Date Table'[Date]))
PMTD Loan Applications:

PMTD Loan Applications = CALCULATE([Total Loan Applications], DATESMTD(DATEADD('Date Table'[Date], -1, MONTH)))
MoM Loan Applications:

MoM Loan Applications = DIVIDE([MTD Loan Applications] - [PMTD Loan Applications], [PMTD Loan Applications])
Other Measures: Similar calculations for Funded Amount, Amount Received, Interest Rate, and DTI.

### Good and Bad Loan Analysis

### Good Loan Percentage:

Good Loan Percentage = DIVIDE(CALCULATE([Total Loan Applications], 'public loans'[loan_status] IN {"Fully Paid", "Current"}), [Total Loan Applications])

### Bad Loan Percentage:

Bad Loan Percentage = DIVIDE(CALCULATE([Total Loan Applications], 'public loans'[loan_status] = "Charged Off"), [Total Loan Applications])

## Visualization Elements

Donut Charts: Create donut charts for Good and Bad Loans.
Slicers: Use slicers for dynamic filtering and interaction.

## Data Validation
Data validation was performed using SQL queries to ensure accuracy and consistency of the data used in the Power BI reports. The queries were designed to dynamically fetch the latest data and handle data updates efficiently.

## Conclusion
This project demonstrates the power of combining SQL and Power BI to create dynamic, optimized, and insightful loan disbursement reports. By leveraging SQL for robust data extraction and transformation, and Power BI for interactive visualizations, the project provides a comprehensive solution for monitoring and analyzing loan data effectively.
