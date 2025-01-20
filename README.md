# Healthcare_RCM_Data_Engineering_Project

## Project Overview
This project aims to build a data pipeline solution for the revenue cycle management 9RCM) processess of a hospital using Azure Databricks. This is an ELT pipeline built using Azure Data factory to ingest various source files containing data about the claims, transactions, provider and patient data ,load it into azure datalake to transform the data, improve the data quality at each stage of the pipeline workflow. The transformed data in the end with facts and dimesnion tables will be used as a source to generate appropriate KPIs to determine the status of the accounts receivable (AR) of the hospital. 

## Architecture Diagram
![Image](https://github.com/user-attachments/assets/44a7ad60-03a9-406f-acd3-d570d96f5bf5)

