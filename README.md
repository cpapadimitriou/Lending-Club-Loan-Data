# Lending Club Loan Data: Exploratory Analysis & Data Pipeline

## Data

In this project I will be exploring Lending Club’s loan origination data from 2007-2015, available via Kaggle: https://www.kaggle.com/wendykan/lending-club-loan-data

---

## Part 1: Data Exploration and Evaluation

In this section, I will perform an exploratory data analysis on Lending Club's loan data using a Jupyter notebook.
The detailed analysis and code can be found here: [Loan Data - EDA.ipynb](https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/Loan%20Data%20-%20EDA.ipynb)

### Summary Findings: 
- After reading the data with python pandas, I carefully examined the features and wrote a data pipeline script to ensure the correct data types.
- There was a large number of columns with missing values. I wrote a script to remove columns with more than 80% missing values. This thershold can be easily adjusted.
- In the `1.4 Data Exploration` section of the `Loan Data - EDA.ipynb` notebook, I performed an exploratory analysis and gathered the following insights:
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


---

## Part 2: Data Pipeline Engineering 

In this section I will create a data model in a database engine, persist the dataset into this storage system in a fully automated way, and develop some data validation procedures.
I will implement this using two approaches:
1. A cloud-based approach using AWS Glue 
2. A desktop solution using SQL*Lite and python (this is solution is outlined but not implemented)


### AWS Glue

In this section I will use AWS Glue to build an end to end data engineering solution: https://aws.amazon.com/glue/ 

AWS Glue is a fully managed extract, transform, and load (ETL) service that makes it easy to prepare and load data for analytics. 
You can give Glue datasets, it reads them, infers a schema, loads them into object store, and then you can then run serverless SQL against them in Object Store (AWS S3). 
Glue can also to connect with the higher storage options. if you out grow serverless, Glue can load the data into RDS (relational data services), 
and if you out grow that, you can load it into RedShift (data warehouse). Glue can also interact with other AWS managed services, clusters, etc.

https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html

Below are the steps I followed to build a data engineering solution using AWS Glue:

- **Step 1**. First, I uploaded the `loan.csv` file with the lending club data to an AWS S3 bucket.
- **Step 2**. Then, I pointed AWS Glue to my data file in the S3 bucket.
- **Step 3**. Next, I created a data model/schema using an AWS crawler. The AWS crawler discovered a schema and then published the schema into the AWS data catalog.
The AWS crawler has built-in classifiers to recognize popular data types but it also allows manual creation of classifiers using regex.

![alt text][logo]

[logo]: https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/images/Schema%20Detected%20by%20AWS%20crawler.png "Schema detected by AWS Crawler"



- **Step 4**. The next step is the mapping. Here I mapped the sourced schema discovered by the AWS crawler to my target schema. I used the AWS Glue UI to make changes to the data types.

![alt text][logo1]

[logo1]: https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/images/Mapping%20Source%20to%20Target%20Schema.png "Mapping Source to Target Schema"


Here is the target data schema: 

```python
loandata = glueContext.create_dynamic_frame.from_catalog(database="loandata", table_name="lending_club_load_data")
print "Count: ", loandata.count()
loandata.printSchema()
```

```
Count:  2260668
root
|-- id: string
|-- member_id: string
|-- loan_amnt: double
|-- funded_amnt: double
|-- funded_amnt_inv: double
|-- term: string
|-- int_rate: double
|-- installment: double
|-- grade: string
|-- sub_grade: string
|-- emp_title: string
|-- emp_length: string
|-- home_ownership: string
|-- annual_inc: double
|-- verification_status: string
|-- issue_d: date
|-- loan_status: string
|-- pymnt_plan: string
|-- url: string
|-- desc: string
|-- purpose: string
|-- title: string
|-- zip_code: string
|-- addr_state: string
|-- dti: double
|-- delinq_2yrs: long
|-- earliest_cr_line: string
|-- inq_last_6mths: long
|-- mths_since_last_delinq: long
|-- open_acc: long
|-- pub_rec: long
|-- revol_bal: long
|-- revol_util: double
|-- total_acc: long
|-- initial_list_status: string
|-- out_prncp: double
|-- out_prncp_inv: double
|-- total_pymnt: double
|-- total_pymnt_inv: double
|-- total_rec_prncp: double
|-- total_rec_int: double
|-- total_rec_late_fee: double
|-- recoveries: double
|-- collection_recovery_fee: double
|-- last_pymnt_d: string
|-- last_pymnt_amnt: double
|-- next_pymnt_d: string
|-- last_credit_pull_d: string
|-- collections_12_mths_ex_med: long
|-- mths_since_last_major_derog: long
|-- policy_code: long
|-- application_type: string
|-- verification_status_joint: string
|-- acc_now_delinq: long
|-- tot_coll_amt: long
|-- tot_cur_bal: long
|-- open_acc_6m: long
|-- open_act_il: long
|-- open_il_12m: long
|-- open_il_24m: long
|-- mths_since_rcnt_il: long
|-- total_bal_il: long
|-- il_util: long
|-- open_rv_12m: long
|-- open_rv_24m: long
|-- max_bal_bc: long
|-- all_util: long
|-- total_rev_hi_lim: long
|-- inq_fi: long
|-- total_cu_tl: long
|-- inq_last_12m: long
|-- acc_open_past_24mths: long
|-- avg_cur_bal: long
|-- bc_open_to_buy: long
|-- bc_util: double
|-- chargeoff_within_12_mths: long
|-- delinq_amnt: long
|-- mo_sin_old_il_acct: long
|-- mo_sin_old_rev_tl_op: long
|-- mo_sin_rcnt_rev_tl_op: long
|-- mo_sin_rcnt_tl: long
|-- mort_acc: long
|-- mths_since_recent_bc: long
|-- mths_since_recent_inq: long
|-- num_accts_ever_120_pd: long
|-- num_actv_bc_tl: long
|-- num_actv_rev_tl: long
|-- num_bc_sats: long
|-- num_bc_tl: long
|-- num_il_tl: long
|-- num_op_rev_tl: long
|-- num_rev_accts: long
|-- num_rev_tl_bal_gt_0: long
|-- num_sats: long
|-- num_tl_120dpd_2m: long
|-- num_tl_30dpd: long
|-- num_tl_90g_dpd_24m: long
|-- num_tl_op_past_12m: long
|-- pct_tl_nvr_dlq: double
|-- percent_bc_gt_75: double
|-- pub_rec_bankruptcies: long
|-- tax_liens: long
|-- tot_hi_cred_lim: long
|-- total_bal_ex_mort: long
|-- total_bc_limit: long
|-- total_il_high_credit_limit: long
|-- sec_app_earliest_cr_line: string
|-- hardship_flag: string
|-- hardship_type: string
|-- hardship_reason: string
|-- hardship_status: string
|-- deferral_term: string
|-- hardship_amount: string
|-- hardship_start_date: date
|-- hardship_end_date: date
|-- payment_plan_start_date: date
|-- hardship_length: string
|-- hardship_dpd: string
|-- hardship_loan_status: string
|-- orig_projected_additional_accrued_interest: string
|-- hardship_payoff_balance_amount: string
|-- hardship_last_payment_amount: string
|-- disbursement_method: string
|-- debt_settlement_flag: string
|-- debt_settlement_flag_date: date
|-- settlement_status: string
|-- settlement_date: date
|-- settlement_amount: string
|-- settlement_percentage: string
|-- settlement_term: string
|-- annual_inc_joint: double
|-- dti_joint: double
|-- mths_since_recent_revol_delinq: long
|-- revol_bal_joint: long
|-- sec_app_inq_last_6mths: long
|-- sec_app_mort_acc: long
|-- sec_app_open_acc: long
|-- sec_app_revol_util: double
|-- sec_app_open_act_il: long
|-- sec_app_num_rev_accts: long
|-- sec_app_chargeoff_within_12_mths: long
|-- sec_app_collections_12_mths_ex_med: long
|-- mths_since_last_record: long
|-- mths_since_recent_bc_dlq: long
|-- sec_app_mths_since_last_major_derog: long
```

- **Step 5**. Next, I explored the data and run SQL queries against it using AWS Athena. The data can also be imported in a SageMaker jupyter notebook for analytics and Machine Learning model training and evaluation. 
Here I am able to create aggregate tables from queries or join different data tables together. In our case we only have one table so there is no need to join datasets.

![alt text][logo2]

[logo2]: https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/images/Query%20Data%20with%20AWS%20Athena.png "SQL Queries with AWS Athena"


- **Step 6**. Lastly, I schedule ETL jobs and defined triggers to run jobs based on a schedule or event. 
Specifically, I used AWS Glue to create ETL jobs in pySpark. AWS Glue automatically generates a diagram with the transformations that take place when the job runs. 
I added 2 triggers to run the job: (1) weekly (2) every time there is new data added to the S3 bucket. These jobs will ensure that the dataset persists into the storage system in a fully automated way. 
I monitored the jobs using AWS Glue. 

[ETL job with pySpark](https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/ETL_job_pySpark.py)

![alt text][logo2]

[logo2]: https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/images/ETL%20Job%20and%20Diagram.png "ETL job and diagram"



### Desktop Solution