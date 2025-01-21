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

- The raw source data obtained through the custom data generator script is placed either in the form of tables in Azure SQL DB is migrated to the **Bronze Layer** using Full Load or Incremental load based on the kind of data being handled
- The Claims data from the **Landing Zone** is migrated and ingested into the **Bronze Layer** and the ICD & NPI codes data obtained through the APIs is directly placed in the **Bronze Layer** (source of truth) using Azure Databricks
- Once we have all the required data in the Bronze Layer, we clean and transform the data to fit into a common data model (CDM). The cleaned data is migrated to the **Silver Layer** in the form of Delta tables
- The clean data from Silver Layer is filtered and ingested into the **Gold Layer**, wherein this data is used to create various facts and dimension tables for analytical purposed to calculate appropriate KPIs for measuring the accounts receivable status of the hospital

### ELT pipeline:
End to End pipeline
![Image](https://github.com/user-attachments/assets/7f3ace56-784a-487b-b0bc-646728fc9ea5)

The entire flow of the ELT pipeline can be divided into 2 parts
- **Data Ingestion to Bronze**: Bringing all the required source data to the **Bronze Zone**
- **Data Transformation**: Migrating the data to the **Silver Layer** and the subsequent cleaning, transformations to the mobing of the data to the **Gold Layer**

<h3>Data Ingestion to Bronze</h3>

We create different **Linked Services** to interact with the various components of the pipeline namely the azure SQL DB, ADLS Gen2 Storage container, Databricks notebooks etc. The secrets are stored in a key vault for all the components to connected through these different Linked Services

For each data item listed in the config file, following is the logical flow of the pipeline to bring the source data to the **Bronze Layer**

![Image](https://github.com/user-attachments/assets/56ff90e8-4401-4d2e-96ea-34f2ac39045b)


For Datasets present in the Azure SQL DB
1. Check if the data file is present in the **Bronze Layer**, if YES then archive the existing file into a seperate folder based on the day of the execution of pipeline

2. In the other scenario of data file not being present or in the continuation step of step.no 1 we check if the pipeline needs to be run for the particular data element by looking at the ```active_flag``` parameter of the current item, if NO then data is not migrated to the Bronze Layer for the data file

3. if YES, then we check for another condition named ```loadtype``` wherein we verify if the data file needs to be migrated as whole at once i.e Full Load or Incrementally

4. For the Full Load scenario we directly upload all the data from Azure SQL DB to the Bronze Layer in parquet file format, and update the logs in an audit table which we create the databricks delta lake environment

5. For Incremental load scenario, first we check the last update date of the records in the data file in the Bronze layer from the logs in the audit schema, based on this only the records updated post the ```last_fetched_date``` will be uploaded into the files of the Bronze Layer, post this activity we update the logs in the audit schema

The claims data from the **Landing Zone** and the NPI & ICD data obtained through APIs are ingested to the Bronze Layer using Azure Databricks notebooks.

All the files in the Bronze Layer are stored in a columnar format (i.e parquet)

<h3>Data Transformation</h3>

#### Bronze to Silver

1. Data is picked up from the Bronze Layer using Azure databricks notebooks

2. Since there are instances where we are dealing with the same kinds of data from two different hospitals with probably different schemas, we are implementing a common data model (CDM) by changing the column names, creating a new field by concatenating the ID and source columns in the combined data of the 2 hospitals

3. We implement data quality checks to identify records missing key information in different datasets, such records are flagged for data discrepancy

4. We also model certain datasets by implementing the Slowly Changing Dimension (SCD) type 2 technique to track the changes in data over time

This way the data in the Bronze layer is cleaned and modelled and migrated to the **Silver Layer** in the form of delta tables

#### Silver to Gold

In this step of the pipeline execution, the clean data in the Silver layer is picked up using the Azure Databricks notebooks and different facts and dimension delta tables are generated using only the latest and clean data i.e records that passed the data quality checks. This is done to ensure that we have the latest and correct data without any data discrepancies so that this could be used for analytical & reporting purposes. 

Schema for the facts and dimension tables
![Image](https://github.com/user-attachments/assets/9380541b-dd8d-42d9-9325-1e3500a197e7)

#### Gold
The latest clean and accurate data in the Gold Layer is being used to write analytical queries to calculate appropriate KPIs to determine the status of accounts receivables (AR)

## Azure Resources Used for this Project:
* Azure Data Lake Storage
* Azure Data Factory
* Azure Databricks
* Azure Key Vault

## Tasks performed:
•	Built a solution architecture for a data engineering solution using Azure Databricks, Azure Data Lake Gen2, Azure Data Factory, and Power BI.

•	Created and used Azure Databricks service and the architecture of Databricks within Azure.

•	Worked with Databricks notebooks and used Databricks utilities, magic commands, etc.

•	Created, configured Databricks clusters, cluster pools, and jobs.

•	Mounted Azure Storage in Databricks using secrets stored in Azure Key Vault.

•	Used Delta Lake to implement a solution using Lakehouse architecture.

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

•	Created dependencies between activities as well as pipelines.

•	Monitored the triggers/ pipelines to check for errors/ outputs.

# About the Project:

<h3>Folders and files:</h3>

- **API_data_extraction**: The folder contains the code to extract the ICD and NPI data using the public APIs to bring the data to te **Bronze Layer**

- **Bronze**: The folder contains notebooks to ingest the Claims and the CPT codes data from **Landing Zone** to the **Bronze Layer**

- **Gold**: The folder contains all notebooks to create facts and dimensions tables from the clean data present in the **Silver Layer**

- **setup**: The folder contains all notebooks to create the ```AUDIT``` table and define some global variables for the databricks workspace

- **silver**: The folder contains all notebooks that transform the raw data from the Bronze Layer into the **Silver Layer** by cleaning the data and creating a common data model between different versions of data

- **data_engineering_faker_module**: This is the script used to generate the custom data for EMR and COT records

- **README**: Markdown file describing the details and working of the project

<h3>Technologies/Tools Used:</h3>
<ul>
  <li>Pyspark</li> 
  <li>Spark SQL</li> 
  <li>Delta Lake</li> 
  <li>Azure Databricks</li> 
  <li>Azure Data Factory</li> 
  <li>Azure Date Lake Storage Gen2</li> 
  <li>Azure Key Vault</li> 
</ul>  