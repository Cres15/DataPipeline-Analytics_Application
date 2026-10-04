**Data Extraction Documentation**

**1\. Data Source and Extraction Specification**

1.  **Source System**  
   **Name:** BlockFlow: Design and Development of an Integrated Inventory, and Sales Management System for Magalin Hollow Blocks Trading  
   **Purpose:** To monitor and record the daily operation of the business in Magalin Hollow Blocks Trading, including a sales transaction, stocks updates, and operational expenses. The Owner and Staff can record daily and the data is exported as raw CSV into the entry directory, where it serves as the source input for the BlockFlow ETL pipeline which prepares it for output in analytics.  
2. **Source Database or File**  
   **Database management system:** SQLite is the pipeline storage like [blockflow.db], there is a serverless, and self-contained relational database that stores data in a local file. But, it is not the extraction source, it is the destination where extracted data is loaded into the users, inventory, expenses, and sales tables.  
   **File format:** The file is CSV for daily raw batch files (stocks updates, sales receipts, and expenses logs) in an entry directory serving as the extraction source.  
3.  **Extraction Method**  
   **Incremental:** The pipeline performs a daily incremental extraction, pulling only to the most recent days batch files rather than re-extracting the entire dataset in every run.   
   **Extraction:** The purpose of an extraction window is to determine the transaction date fields in the raw batch files, including in updated\_at (last stocks updates), sales_date (sales receipts), and expenses_date (expenses logs). Each schedule extracts only in records belonging to the previous calendar day and any already processed records are protected from duplication by the INSERT/UPSERT logic used at load time.  
   **How data will be extracted:** The extracted process is on each scheduled run, the Prefect flow triggers the extraction task, which scans the entry directory for the target days raw CSV batch files. Then the Python reads these files into in memory pandas DataFrames, which passed in the Transform stage of the pipeline.  
   **Extraction tool:** The Python extraction script using pandas, csv, and json libraries, wrapped as a Prefect @task inside a Prefect @flow. The Prefect handles the scheduling, task tracking, retries and execution logging.  
   **Extraction schedule:** 12:00 AM daily with a maximum of one active run at a time and only 30 minutes execution timeout.
4. **Extraction Scope**
   
Data to be extracted: The pipeline extracts the business daily operation data. There are three domains: the stock updates (current stock levels of hollow blocks), sales transactions (hollow block sales receipts), and operational expenses (raw materials, maintenance, and other costs). This pipeline matches to define the data source of daily raw CSV batch operational logs.
Files: Per batch domain there is one daily entry in the entry directory, including the stock updates, sales receipts, and expense logs as (CSV).
Target tables: There are three tables including the Stocks, Sales, and Expenses in the SQLite database. This is extracted fields follow the documented schema, included: Stocks (size, quantity, unit, unit price, updated at, recorded by), Sales (customer name, phone number, shop name, quantity, unit price, sale date, recorded by), and Expenses (category, description, amount, expense date, recorded by).
Excluded data: The user account credentials are not extracted, since the users table exists only for system login and access control, not for operational analytics.

5. **Source Limitations and Assumptions**

The data extraction process is limited by the availability and quality of the daily raw CSV files. Some source data may contain missing values, duplicate records, or inconsistent date and number formats, which may require cleaning and checking before loading the data into the database. The extraction is also limited to the available operational data, including sales, stocks, and expense records. Some historical records may be incomplete because the business previously relied on manual logs and spreadsheets. The extraction process also depends on the defined database schema and relationships. Changes in field names, data types, or required fields may require adjustments to the extraction and transformation process. External information, such as competitor pricing and market costs, is outside the scope of the extraction.
The extraction process assumes that the required daily raw CSV files are available in the designated folder and that the system has the necessary read and write permissions. It also assumes that the local SQLite database is accessible and that the required tables and relationships are already available. The process also assumes that the extracted data follows the required format and contains valid references for fields such as recorded_by and stocks_id. It is also assumed that the extracted data represents the actual operational records of Magalin Hollow Blocks Trading and is processed only for the intended BlockFlow system and reporting purposes. Only authorized business data related to the BlockFlow system is included in the extraction.
