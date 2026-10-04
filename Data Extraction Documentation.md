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