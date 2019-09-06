# Lending Club Loan Data Exploratory Analysis & Data Pipeline

## Data

In this project I will be exploring Lending Club’s loan origination data from 2007-2015, available via Kaggle: https://www.kaggle.com/wendykan/lending-club-loan-data

## Part 1: Data Exploration and Evaluation

In this section I will perform an exploratory data analysis on Lending Club's loan data using a Jupyter notebook.
The detailed analysis and code can be found here: [Loan Data - EDA.ipynb](https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/Loan%20Data%20-%20EDA.ipynb)

### Summary Findings: 
- After reading the data with python pandas, I carefully examined the features and wrote a data pipeline script to ensure the correct data types.
- There was a large number of columns with missing values. I wrote a script to remove columns with more than 80% missing values. This thershold can be easily adjusted.
- In the `1.4 Data Exploration` section of the `Loan Data - EDA,ipynb` notebook, I performed an exploratory analysis and gathered the following insights:
    * The summary statistics of the numerical variables showed a big variation in ranges among variables, indicating that a normalization of the features would be necessary before using this data for modeling purposes.
    * Certain variables (e.g. `annual income`) have high standard deviation values.
    * The average loan amount is around $15,000 but the median is $12,900, showing a negative skewness.
    * The correlation matrix indicated that certain key variables are very highly correlated with each other. This finding has to be taken into consideration when using this dataset of modeling purposes, since for example multicollinearity violates the OLS regression assumptions. 
    * Histograms of the numeric variables showed that most variables in the dataset are negatively skewed, with very few resembling a normal distribution.
    * Distribution plots on the main categorical variables showed several insights into understanding the data: 
        * Most of the loans in the dataset are fully paid, and the second largest category is current loans. The third is charged off loans (never paid), and it is a quite large number in terms of loan counts.
        * Only 31 out of 2,260,668 loans in our dataset have defaulted - that is 0.0014%.
        * The most common purpose for a loan origination in our dataset is consolidation of debt, followed by credit card debt.
        * The most common job titles of borrowers are Teacher, Manager, and Owner and the majority of borrowers leave this field blank.
        * B and C are the most common credit rating categories in the dataset.
        * Loan originations have been growing every year, with a larger growth rate in 2007-2015, than in 2015-2018.


## Part 2: Data Pipeline Engineering 

In this section I will create a data model in a database engine, persist the dataset into this storage system in a fully automated way, and develop some data validation procedures.
I will implement this using two approaches (1) a cloud based approach using AWS Glue and (2) a desktop solution using SQL*Lite and python.





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
I used AWS Glue to create an ETL job in pySpark. AWS Glue automatically generates a diagram with the transformations that take place when the job runs. 
I then added 2 triggers to run the job: (1) weekly (2) every time there is new data added to the S3 bucket. These jobs will ensure that the dataset persists into the storage system in a fully automated way. 



### Desktop Solution