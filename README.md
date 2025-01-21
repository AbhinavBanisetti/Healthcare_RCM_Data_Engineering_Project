# Revenue Cycle Management Pipeline

## Project Overview
This project aims to build a data pipeline solution for the revenue cycle management (RCM) processes of a hospital using Azure Databricks. This is an ELT pipeline built using Azure Data factory to ingest various raw source files into azure datalake to transform the data, improve the data quality at each stage of the pipeline workflow. The transformed data in the end with facts and dimension tables will be used as a source to generate appropriate KPIs to determine the status of the accounts receivable (AR) of the hospital. 

## Architecture Diagram
![Image](https://github.com/user-attachments/assets/44a7ad60-03a9-406f-acd3-d570d96f5bf5)

## How it works:
<h3>Source Files</h3>
We are referring to different open-source data for ICD (International Classfication of diseases) codes from the website WHO (World Health Organization) and NPI(National Provide Information) registry API. Custom data has also been generated using the faker module in python to emulate the EMR data of hospitals along with the patients and transaction information

| File Name  | File Type |
| ------------- | ------------- |
| Transactions | Custom Data Azure SQL |
| CPT codes | Custom Data CSV |
| ICD codes | CSV |
| Encounters  | Custom Data Azure SQL  |
| Patients | Custom Data Azure SQL |
| Claims  | CSV  |
| NPI codes  | CSV |
| Departments  | Custom Data Azure SQL |
| Providers | Custom Data Azure SQL | 

<h3>Execution Overview:</h3>
We are defining the flow of the pipeline execution with the config file which describes the order of execution of activities i.e metadata driven
- The raw source data obtained through the custom data generator script is placed either in the form of tables in Azure SQL DB, the claims data are stored in the **Landing Zone** and the ICD & NPI codes data obtained through the APIs is directly placed in the **Bronze Layer** (source of truth) using Azure Databricks
- Data from the **Landing Zone** is migrated and ingested into the **Bronze Layer**
- Once we have all the required data in the Bronze Layer, the data is migrated to the **Silver Layer** using Full Load or incremental load based on the kind of data being handled
- In the Silver Layer we clean and transform the data to fit into a common data model (CDM)
- The cleaned data from Silver Layer is filtered and ingested into the **Gold Layer**, wherein this data is used to create various facts and dimension tables for analytical purposed to calculate appropriate KPIs for measuring the accounts receivable status of the hospital

### ELT pipeline:
The entire flow of the ELT pipeline can be divided into 2 parts
- **Data Ingestion to Bronze**: Bringing all the required source data to the **Bronze Zone**
- **Data Transformation**: Migrating the data to the **Silver Layer** and the subsequent cleaning, transformations to the mobing of the data to the **Gold Layer**

<h3>Data Ingestion to Bronze</h3>

We create different **Linked Services** to interact with the various components of the pipeline namely the azure SQL DB, ADLS Gen2 Storage container, Databricks notebooks etc
For each data item listed in the config file, following is the logical flow of the pipeline to bring the source data to the **Bronze Layer**

![Image](https://github.com/user-attachments/assets/7814c103-49c6-4b62-a184-882789edc5d1)


For Datasets present in the Azure SQL DB
1. Check if the data file is present in the **Bronze Layer**, if YES then archive the existing file into a seperate folder based on the day of the execution of pipeline

2. In the other scenario of data file not being present or in the continuation step of step.no 1 we check if the pipeline needs to be run for the particular data element by looking at the **active_flag** parameter of the current item, if NO then data is not migrated to the Bronze Layer for the data file

3. if YES, then we check for another condition named **loadtype** wherein we verify if the data file needs to be migrated as whole at once i.e Full Load or Incrementally

4. For the Full Load scenario we directly upload all the data from Azure SQL DB to the Bronze Layer in parquet file format, and update the logs in an audit table which we create the databricks delta lake environment

5. For Incremental load scenario, first we check the last update date of the records in the data file in the Bronze layer from the logs in the audit schema, based on this only the records updated post the **last_fetched_date** will be uploaded into the files of the Bronze Layer, post this activity we update the logs in the audit schema

The claims data from the **Landing Zone** and the NPI & ICD data obtained through APIs are ingested to the Bronze Layer using Azure Databricks notebooks

In the first pipeline, data stored in JSON and CSV format is read using Apache Spark with minimal transformation saved into a delta table. The transformation includes dropping columns, renaming headers, applying schema, and adding audited columns (```ingestion_date``` and ```file_source```) and ```file_date``` as the notebook parameter. This serves as a dynamic expression in ADF.

In the second pipeline, Databricks SQL reads preprocessed delta files and transforms them into the final dimensional model tables in delta format. Transformations performed include dropping duplicates, joining tables using join, and aggregating using a window.

ADF is scheduled to run every Sunday at 10 PM and is designed to skip the execution if there is no race that week. We have another pipeline to execute the ingestion pipeline and transformation pipeline using file_date as the parameter for the tumbling window trigger.

![Screen Shot 2022-06-12 at 4 42 18 PM](https://user-images.githubusercontent.com/107358349/173252855-6a50be95-d7a7-481c-9438-8ae9fdc7df28.png)

## Azure Resources Used for this Project:
* Azure Data Lake Storage
* Azure Data Factory
* Azure Databricks
* Azure Key Vault

## Project Requirements:
The requirements for this project are broken down into six different parts which are

#### 1. Data Ingestion Requirements
* Ingest all 8 files into Azure data lake. 
* Ingested data must have the same schema applied.
* Ingested data must have audit columns.
* Ingested data must be stored in  columnar format (i.e. parquet).
* We must be able to analyze the ingested data via SQL.
* Ingestion Logic must be able to handle the incremental load.

#### 2. Data Transformation Requirements
* Join the key information required for reporting to create a new table.
* Join the key information required for analysis to create a new table.
* Transformed tables must have audit columns.
* We must be able to analyze the transformed data via SQL.
* Transformed data must be stored in columnar format (i.e. parquet).
* Transformation logic must be able to handle the incremental load.

#### 3. Data Reporting Requirements
* We want to be able to know Driver Standings.
* We should be able to know Constructor Standings as well.

#### 4. Data Analysis Requirements
* Find the Dominant drivers.
* Find the Dominant Teams. 
* Visualize the Outputs.
* Create Databricks dashboards.

#### 5. Scheduling Requirements
* Scheduled to run every Sunday at 10 pm.
* Ability to monitor pipelines.
* Ability to rerun failed pipelines.
* Ability to set up alerts on failures

#### 6. Other Non-Functional Requirements
* Ability to delete individual records
* Ability to see history and time travel
* Ability to roll back to a previous version

## Analysis Result:
![image](https://user-images.githubusercontent.com/64007718/235310453-95b6d253-aaab-454b-87f1-8fb722600014.png)
![image](https://user-images.githubusercontent.com/64007718/235310459-c9141816-2832-4be7-8902-3fce7096c88d.png)
![image](https://user-images.githubusercontent.com/64007718/235310466-4a83e4ce-00c3-444c-b22a-83ad42530321.png)
![image](https://user-images.githubusercontent.com/64007718/235310470-9c966e29-ba76-4c10-9554-f201d72ee636.png)
![image](https://user-images.githubusercontent.com/64007718/235310476-98db1649-0fb4-45f5-bfc4-8892afc8bc80.png)
![image](https://user-images.githubusercontent.com/64007718/235310486-98404d97-ed11-4be2-90c3-535f538cfdc9.png)

## Tasks performed:
•	Built a solution architecture for a data engineering solution using Azure Databricks, Azure Data Lake Gen2, Azure Data Factory, and Power BI.

•	Created and used Azure Databricks service and the architecture of Databricks within Azure.

•	Worked with Databricks notebooks and used Databricks utilities, magic commands, etc.

•	Passed parameters between notebooks as well as created notebook workflows.

•	Created, configured, and monitored Databricks clusters, cluster pools, and jobs.

•	Mounted Azure Storage in Databricks using secrets stored in Azure Key Vault.

•	Worked with Databricks Tables, Databricks File System (DBFS), etc.

•	Used Delta Lake to implement a solution using Lakehouse architecture.

•	Created dashboards to visualize the outputs.

•	Connected to the Azure Databricks tables from PowerBI.

## Spark (Only PySpark and SQL)
•	Spark architecture, Data Sources API, and Dataframe API.

•	PySpark - Ingested CSV, simple, and complex JSON files into the data lake as parquet files/ tables.

•	PySpark - Transformations such as Filter, Join, Simple Aggregations, GroupBy, Window functions etc.

•	PySpark - Created global and temporary views.

•	Spark SQL - Created databases, tables, and views.

•	Spark SQL - Transformations such as Filter, Join, Simple Aggregations, GroupBy, Window functions etc.

•	Spark SQL - Created local and temporary views.

•	Implemented full refresh and incremental load patterns using partitions.

## Delta Lake
•	Performed Read, Write, Update, Delete, and Merge to delta lake using both PySpark as well as SQL.

•	History, Time Travel, and Vacuum.

•	Converted Parquet files to Delta files.

•	Implemented incremental load pattern using delta lake.

## Azure Data Factory
•	Created pipelines to execute Databricks notebooks.

•	Designed robust pipelines to deal with unexpected scenarios such as missing files.

•	Created dependencies between activities as well as pipelines.

•	Scheduled the pipelines using data factory triggers to execute at regular intervals.

•	Monitored the triggers/ pipelines to check for errors/ outputs.

# About the Project:

<h3>Folders:</h3>

- 1-Authentication: The folder contains all notebooks to demonstrate different ways to access Azure Data Lake Gen2 containers into the Databricks file system.

- 2-includes: The folder contains notebooks with common functions and path configurations.

- 3-Data Ingestion: The folder contains all notebooks to ingest the data from raw to processed.

- 4-raw: The folder contains all notebooks to create raw tables in SQL.

- 5-Data Transformation: The folder contains all notebooks that transform the raw data into the processed layer.

- 6-Data Analysis: The folder contains all notebooks which include an analysis of the data.

- 7-demo: The folder contains notebooks with all the pre-requisite demos.

- 8-Power Bi reports: This folder contains all the reports created from the analyzed data.

<h3>Technologies/Tools Used:</h3>
<ul>
  <li>Pyspark</li> 
  <li>Spark SQL</li> 
  <li>Delta Lake</li> 
  <li>Azure Databricks </li> 
  <li>Azure Data Factory</li> 
  <li>Azure Date Lake Storage Gen2</li> 
  <li>Azure Key Fault</li> 
  <li>Power BI</li> 
</ul>  