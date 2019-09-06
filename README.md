# Lending Club Loan Data Exploratory Analysis & Data Pipeline

## Data

We will be exploring Lending Club’s loan origination data from 2007-2015, available via Kaggle: https://www.kaggle.com/wendykan/lending-club-loan-data

## Part 1: Data Exploration and Evaluation

In this section I will perform an exploratory data analysis on Lending Club's loan data.
The detailed analysis and code can be found in this notebook: [Loan Data - EDA.ipynb](https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/Loan%20Data%20-%20EDA.ipynb)

### Summary Findings: 


## Part 2: Data Pipeline Engineering 

In this section I will create a data model in a database engine, persist the dataset into this storage system in a fully automated way, and develop some data validation procedures.
I will implement this using two approaches (1) a cloud based approach using AWS Glue and (2) a desktop solution using SQL*Lite and python.

### Desktop Solution



### AWS Glue

In this section I will use AWS Glue to build an end to end data engineering solution: https://aws.amazon.com/glue/ 

AWS Glue is a fully managed extract, transform, and load (ETL) service that makes it easy to prepare and load data for analytics. You can give Glue datasets, it reads them, infers schema, loads into object store, and you can then run serverless SQL against them in Object Store (AWS S3). Glue can also to connect with the higher storage options. if you out grow serverless, Glue can load the data into RDS (relational data services), and if you out grow that, load it into RedShift (data warehouse). Glue can also interact with other AWS managed services, clusters, etc.

https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html

- Step 1. Upload the `loan.csv` file with the lending club data to an AWS S3 bucket 
- Step 2. Pointing AWS Glue to my data file in the S3 bucket
- Step 3. Create the data model/schema. In this step I use the AWS crawler to discover a schema and then publish the schema into the data catalog.
The AWS crawler has built-in classifiers to recognize popular data types.

- Step 4. The next step is the mapping. Here I map the sourced schema discovered by the AWS crawler to my target schema.
- Step 5. Edit and Explore - running queries against the data (using SQL queries on AWS Athena)
- Step 6. Here I am able to create aggregate tables from queries or join different data tables together. In our case we only have one table so there is no need to join datasets.
- Step 6. Schedule ETL jobs and define triggers to run jobs based on a schedule or event. Monitor the jobs using AWS Glue.


run an ETL job to go from the sourced dataset to 

use a notebook within AWS 