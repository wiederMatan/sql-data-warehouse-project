# ETL Process Using Change Data Capture (CDC) with SQL Server

## Overview
This project demonstrates how to implement an **ETL (Extract, Transform, Load) process** using **Change Data Capture (CDC)** in **SQL Server**. CDC enables tracking changes in source tables and efficiently loading incremental updates into a data warehouse or another target system.

## Prerequisites
- **SQL Server 2016 or later**
- **SQL Server Management Studio (SSMS)**
- **Basic knowledge of SQL and ETL concepts**

## Steps to Implement ETL Using CDC in SQL Server

### 1. Enable CDC on the Database
Before tracking changes, **CDC must be enabled** at the database level.
```sql
USE YourDatabase;
EXEC sys.sp_cdc_enable_db;
```

### 2. Enable CDC on a Table
Enable CDC on the specific table you want to track.
```sql
EXEC sys.sp_cdc_enable_table
    @source_schema = 'dbo',
    @source_name = 'YourTable',
    @role_name = NULL,  -- (Optional) Specify a role for access
    @supports_net_changes = 1; -- Allows net changes
```
This will create a new **CDC capture instance** for the table.

### 3. View CDC Tracked Changes
SQL Server creates system tables to store change data. Use this query to check the changes:
```sql
SELECT * FROM cdc.dbo_YourTable_CT;
```

### 4. Extract Changes for ETL Processing
To retrieve only new changes since the last ETL run:
```sql
DECLARE @from_lsn BINARY(10);
DECLARE @to_lsn BINARY(10);

-- Get the latest LSN
SELECT @to_lsn = sys.fn_cdc_get_max_lsn();

-- Get the last processed LSN (should be stored in your ETL system)
SELECT @from_lsn = LastProcessedLSN FROM ETL_MetadataTable;

-- Fetch changed data
SELECT *
FROM cdc.fn_cdc_get_all_changes_dbo_YourTable(@from_lsn, @to_lsn, 'all');

-- Update the last processed LSN
UPDATE ETL_MetadataTable
SET LastProcessedLSN = @to_lsn;
```

### 5. Transform the Data
Transformation can be done using **SQL Views, Stored Procedures, or external ETL tools** like **Apache Airflow, SSIS, or DBT**.
```sql
SELECT ID, UPPER(Name) AS Name_Upper, GETDATE() AS ProcessedDate
FROM cdc.fn_cdc_get_all_changes_dbo_YourTable(@from_lsn, @to_lsn, 'all');
```

### 6. Load Data into the Target System
After transformation, load data into the target data warehouse:
```sql
INSERT INTO DataWarehouse.dbo.YourTable (ID, Name, ProcessedDate)
SELECT ID, Name_Upper, ProcessedDate
FROM TransformedTable;
```

### 7. Automate the Process
- **Schedule Jobs**: Use **SQL Server Agent** to run CDC-based ETL jobs at regular intervals.
- **Use Airflow or SSIS**: Integrate with ETL orchestration tools.

### 8. Monitor CDC Cleanup
SQL Server **automatically cleans up old CDC data**. Adjust the retention period if needed:
```sql
EXEC sys.sp_cdc_change_job @job_type='cleanup', @retention=1440; -- Retains for 1 day
```

## Benefits of CDC for ETL
- **Efficient Incremental Data Extraction**
- **No Need for Triggers (Low Overhead)**
- **Supports Historical Change Tracking**

## Conclusion
This guide demonstrates a **CDC-driven ETL process** with **SQL Server**. This method is useful for efficient **incremental data processing** while maintaining a historical record of changes.
