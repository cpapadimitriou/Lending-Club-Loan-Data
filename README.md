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


---

### Desktop Data Engineering Solution

In this section, I will implement an alternative data engineering solution using a Desktop approach instead of a Cloud approach. 

I will use python and the `SQLAlchemy` library to build an ETL data pipeline, and an `SQLite` database to store the data. 
The data engineering pipeline can be found here: [Loan Data - Data Pipeline.ipynb](https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/Loan%20Data%20-%20Data%20Pipeline.ipynb)

* **Step 1**. First, I created a database on SQLite called `loans.db`
* **Step 2**. Then, I defined a data model/schema using `SQLAlchemy`. I defined the data types based on my earlier exploratory analysis in 
[Loan Data - EDA.ipynb](https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/Loan%20Data%20-%20EDA.ipynb). 
My schema consists of only one table `tbl_loans`, but as I mention later in the future system improvements, 
if the `member_id` column was not empty I would be able to build a schema with two tables separating the loan level data from the member level data.
In this step, I also connected to my database via an SQL IDE on IntelliJ IDEA, which I will use later to easily run SQL queries.

![alt text][logo3]

[logo3]: https://github.com/cpapadimitriou/Lending-Club-Loan-Data/blob/master/images/SQL.png "SQL UI on IntelliJ IDEA"


* **Step 3**. The next step is to extract the data from its original source (i.e `loans.csv` file) and load it into my jupyter notebook using the `pandas` library.

* **Step 4**. Then I performed some minor transformations in python (but aimed to handle most of the transformation later via SQL):
    * dropped the empty `id` column and resetting the index as a new `loan_id` column to serve as the primary key in my table
 
* **Step 5**. After the transformations, I loaded the pandas data frame into a staging table in my SQLite database using the `to_sql` command. 

* **Step 6**. From this point onward, I used my SQL UI in IntelliJ IDEA to transform the staging table to the desired format run SQL queries on it. 
Now I will walk you through my SQL data transformation queries which are also included in the the `Loan Data - Data Pipeline.ipynb` notebook. 
    * create table (this step was already executed on python via SQLAlchemy)
    
    ```sql
    CREATE TABLE tbl_loans
    (
      loan_id                                    INT not null primary key,
      member_id                                  INT,
      term                                       VARCHAR(100),
      grade                                      VARCHAR(100),
      url                                        VARCHAR(100),
      sub_grade                                  VARCHAR(100),
      emp_title                                  VARCHAR(100),
      emp_length                                 VARCHAR(100),
      home_ownership                             VARCHAR(100),
      verification_status                        VARCHAR(100),
      loan_status                                VARCHAR(100),
      pymnt_plan                                 VARCHAR(100),
      desc                                       VARCHAR(100),
      purpose                                    VARCHAR(100),
      title                                      VARCHAR(100),
      zip_code                                   VARCHAR(100),
      addr_state                                 VARCHAR(100),
      earliest_cr_line                           VARCHAR(100),
      initial_list_status                        VARCHAR(100),
      application_type                           VARCHAR(100),
      verification_status_joint                  VARCHAR(100),
      sec_app_earliest_cr_line                   VARCHAR(100),
      hardship_flag                              VARCHAR(100),
      hardship_type                              VARCHAR(100),
      hardship_reason                            VARCHAR(100),
      hardship_status                            VARCHAR(100),
      hardship_loan_status                       VARCHAR(100),
      disbursement_method                        VARCHAR(100),
      debt_settlement_flag                       VARCHAR(100),
      settlement_status                          VARCHAR(100),
      policy_code                                INT,
      loan_amnt                                  FLOAT,
      funded_amnt                                FLOAT,
      revol_bal                                  FLOAT,
      funded_amnt_inv                            FLOAT,
      int_rate                                   FLOAT,
      installment                                FLOAT,
      annual_inc                                 FLOAT,
      dti                                        FLOAT,
      delinq_2yrs                                FLOAT,
      inq_last_6mths                             FLOAT,
      mths_since_last_delinq                     FLOAT,
      mths_since_last_record                     FLOAT,
      open_acc                                   FLOAT,
      pub_rec                                    FLOAT,
      revol_util                                 FLOAT,
      total_acc                                  FLOAT,
      out_prncp                                  FLOAT,
      out_prncp_inv                              FLOAT,
      total_pymnt                                FLOAT,
      total_pymnt_inv                            FLOAT,
      total_rec_prncp                            FLOAT,
      total_rec_int                              FLOAT,
      total_rec_late_fee                         FLOAT,
      recoveries                                 FLOAT,
      collection_recovery_fee                    FLOAT,
      last_pymnt_amnt                            FLOAT,
      collections_12_mths_ex_med                 FLOAT,
      mths_since_last_major_derog                FLOAT,
      annual_inc_joint                           FLOAT,
      dti_joint                                  FLOAT,
      acc_now_delinq                             FLOAT,
      tot_coll_amt                               FLOAT,
      tot_cur_bal                                FLOAT,
      open_acc_6m                                FLOAT,
      open_act_il                                FLOAT,
      open_il_12m                                FLOAT,
      open_il_24m                                FLOAT,
      mths_since_rcnt_il                         FLOAT,
      total_bal_il                               FLOAT,
      il_util                                    FLOAT,
      open_rv_12m                                FLOAT,
      open_rv_24m                                FLOAT,
      max_bal_bc                                 FLOAT,
      all_util                                   FLOAT,
      total_rev_hi_lim                           FLOAT,
      inq_fi                                     FLOAT,
      total_cu_tl                                FLOAT,
      inq_last_12m                               FLOAT,
      acc_open_past_24mths                       FLOAT,
      avg_cur_bal                                FLOAT,
      bc_open_to_buy                             FLOAT,
      bc_util                                    FLOAT,
      chargeoff_within_12_mths                   FLOAT,
      delinq_amnt                                FLOAT,
      mo_sin_old_il_acct                         FLOAT,
      mo_sin_old_rev_tl_op                       FLOAT,
      mo_sin_rcnt_rev_tl_op                      FLOAT,
      mo_sin_rcnt_tl                             FLOAT,
      mort_acc                                   FLOAT,
      mths_since_recent_bc                       FLOAT,
      mths_since_recent_bc_dlq                   FLOAT,
      mths_since_recent_inq                      FLOAT,
      mths_since_recent_revol_delinq             FLOAT,
      num_accts_ever_120_pd                      FLOAT,
      num_actv_bc_tl                             FLOAT,
      num_actv_rev_tl                            FLOAT,
      num_bc_sats                                FLOAT,
      num_bc_tl                                  FLOAT,
      num_il_tl                                  FLOAT,
      num_op_rev_tl                              FLOAT,
      num_rev_accts                              FLOAT,
      num_rev_tl_bal_gt_0                        FLOAT,
      num_sats                                   FLOAT,
      num_tl_120dpd_2m                           FLOAT,
      num_tl_30dpd                               FLOAT,
      num_tl_90g_dpd_24m                         FLOAT,
      num_tl_op_past_12m                         FLOAT,
      pct_tl_nvr_dlq                             FLOAT,
      percent_bc_gt_75                           FLOAT,
      pub_rec_bankruptcies                       FLOAT,
      tax_liens                                  FLOAT,
      tot_hi_cred_lim                            FLOAT,
      total_bal_ex_mort                          FLOAT,
      total_bc_limit                             FLOAT,
      total_il_high_credit_limit                 FLOAT,
      revol_bal_joint                            FLOAT,
      sec_app_inq_last_6mths                     FLOAT,
      sec_app_mort_acc                           FLOAT,
      sec_app_open_acc                           FLOAT,
      sec_app_revol_util                         FLOAT,
      sec_app_open_act_il                        FLOAT,
      sec_app_num_rev_accts                      FLOAT,
      sec_app_chargeoff_within_12_mths           FLOAT,
      sec_app_collections_12_mths_ex_med         FLOAT,
      sec_app_mths_since_last_major_derog        FLOAT,
      deferral_term                              FLOAT,
      hardship_amount                            FLOAT,
      hardship_length                            FLOAT,
      hardship_dpd                               FLOAT,
      orig_projected_additional_accrued_interest FLOAT,
      hardship_payoff_balance_amount             FLOAT,
      hardship_last_payment_amount               FLOAT,
      settlement_amount                          FLOAT,
      settlement_percentage                      FLOAT,
      settlement_term                            FLOAT,
      issue_d                                    DATETIME,
      last_pymnt_d                               DATETIME,
      next_pymnt_d                               DATETIME,
      last_credit_pull_d                         DATETIME,
      hardship_start_date                        DATETIME,
      hardship_end_date                          DATETIME,
      payment_plan_start_date                    DATETIME,
      debt_settlement_flag_date                  DATETIME,
      settlement_date                            DATETIME
    );
    
    ```

    *  insert the data of the staging table to the `tbl_loans` table that has the correct data types. 
    In this step I exclude the index column that was created when loading the data to the staging table, and the columns with 100% missing values (`member_id` and `url`).

    ```sql
    INSERT INTO tbl_loans 
    (loan_id, loan_amnt, funded_amnt, funded_amnt_inv, term, int_rate, installment, grade, sub_grade, emp_title, emp_length, home_ownership, annual_inc, verification_status, issue_d, loan_status, pymnt_plan, desc, purpose, title, zip_code, addr_state, dti, delinq_2yrs, earliest_cr_line, inq_last_6mths, mths_since_last_delinq, mths_since_last_record, open_acc, pub_rec, revol_bal, revol_util, total_acc, initial_list_status, out_prncp, out_prncp_inv, total_pymnt, total_pymnt_inv, total_rec_prncp, total_rec_int, total_rec_late_fee, recoveries, collection_recovery_fee, last_pymnt_d, last_pymnt_amnt, next_pymnt_d, last_credit_pull_d, collections_12_mths_ex_med, mths_since_last_major_derog, policy_code, application_type, annual_inc_joint, dti_joint, verification_status_joint, acc_now_delinq, tot_coll_amt, tot_cur_bal, open_acc_6m, open_act_il, open_il_12m, open_il_24m, mths_since_rcnt_il, total_bal_il, il_util, open_rv_12m, open_rv_24m, max_bal_bc, all_util, total_rev_hi_lim, inq_fi, total_cu_tl, inq_last_12m, acc_open_past_24mths, avg_cur_bal, bc_open_to_buy, bc_util, chargeoff_within_12_mths, delinq_amnt, mo_sin_old_il_acct, mo_sin_old_rev_tl_op, mo_sin_rcnt_rev_tl_op, mo_sin_rcnt_tl, mort_acc, mths_since_recent_bc, mths_since_recent_bc_dlq, mths_since_recent_inq, mths_since_recent_revol_delinq, num_accts_ever_120_pd, num_actv_bc_tl, num_actv_rev_tl, num_bc_sats, num_bc_tl, num_il_tl, num_op_rev_tl, num_rev_accts, num_rev_tl_bal_gt_0, num_sats, num_tl_120dpd_2m, num_tl_30dpd, num_tl_90g_dpd_24m, num_tl_op_past_12m, pct_tl_nvr_dlq, percent_bc_gt_75, pub_rec_bankruptcies, tax_liens, tot_hi_cred_lim, total_bal_ex_mort, total_bc_limit, total_il_high_credit_limit, revol_bal_joint, sec_app_earliest_cr_line, sec_app_inq_last_6mths, sec_app_mort_acc, sec_app_open_acc, sec_app_revol_util, sec_app_open_act_il, sec_app_num_rev_accts, sec_app_chargeoff_within_12_mths, sec_app_collections_12_mths_ex_med, sec_app_mths_since_last_major_derog, hardship_flag, hardship_type, hardship_reason, hardship_status, deferral_term, hardship_amount, hardship_start_date, hardship_end_date, payment_plan_start_date, hardship_length, hardship_dpd, hardship_loan_status, orig_projected_additional_accrued_interest, hardship_payoff_balance_amount, hardship_last_payment_amount, disbursement_method, debt_settlement_flag, debt_settlement_flag_date, settlement_status, settlement_date, settlement_amount, settlement_percentage, settlement_term)
    select loan_id, loan_amnt, funded_amnt, funded_amnt_inv, term, int_rate, installment, grade, sub_grade, emp_title, emp_length, home_ownership, annual_inc, verification_status, issue_d, loan_status, pymnt_plan, desc, purpose, title, zip_code, addr_state, dti, delinq_2yrs, earliest_cr_line, inq_last_6mths, mths_since_last_delinq, mths_since_last_record, open_acc, pub_rec, revol_bal, revol_util, total_acc, initial_list_status, out_prncp, out_prncp_inv, total_pymnt, total_pymnt_inv, total_rec_prncp, total_rec_int, total_rec_late_fee, recoveries, collection_recovery_fee, last_pymnt_d, last_pymnt_amnt, next_pymnt_d, last_credit_pull_d, collections_12_mths_ex_med, mths_since_last_major_derog, policy_code, application_type, annual_inc_joint, dti_joint, verification_status_joint, acc_now_delinq, tot_coll_amt, tot_cur_bal, open_acc_6m, open_act_il, open_il_12m, open_il_24m, mths_since_rcnt_il, total_bal_il, il_util, open_rv_12m, open_rv_24m, max_bal_bc, all_util, total_rev_hi_lim, inq_fi, total_cu_tl, inq_last_12m, acc_open_past_24mths, avg_cur_bal, bc_open_to_buy, bc_util, chargeoff_within_12_mths, delinq_amnt, mo_sin_old_il_acct, mo_sin_old_rev_tl_op, mo_sin_rcnt_rev_tl_op, mo_sin_rcnt_tl, mort_acc, mths_since_recent_bc, mths_since_recent_bc_dlq, mths_since_recent_inq, mths_since_recent_revol_delinq, num_accts_ever_120_pd, num_actv_bc_tl, num_actv_rev_tl, num_bc_sats, num_bc_tl, num_il_tl, num_op_rev_tl, num_rev_accts, num_rev_tl_bal_gt_0, num_sats, num_tl_120dpd_2m, num_tl_30dpd, num_tl_90g_dpd_24m, num_tl_op_past_12m, pct_tl_nvr_dlq, percent_bc_gt_75, pub_rec_bankruptcies, tax_liens, tot_hi_cred_lim, total_bal_ex_mort, total_bc_limit, total_il_high_credit_limit, revol_bal_joint, sec_app_earliest_cr_line, sec_app_inq_last_6mths, sec_app_mort_acc, sec_app_open_acc, sec_app_revol_util, sec_app_open_act_il, sec_app_num_rev_accts, sec_app_chargeoff_within_12_mths, sec_app_collections_12_mths_ex_med, sec_app_mths_since_last_major_derog, hardship_flag, hardship_type, hardship_reason, hardship_status, deferral_term, hardship_amount, hardship_start_date, hardship_end_date, payment_plan_start_date, hardship_length, hardship_dpd, hardship_loan_status, orig_projected_additional_accrued_interest, hardship_payoff_balance_amount, hardship_last_payment_amount, disbursement_method, debt_settlement_flag, debt_settlement_flag_date, settlement_status, settlement_date, settlement_amount, settlement_percentage, settlement_term
    
    FROM loans_staging_table;
    ```
    
    * drop the staging table
    ```sql 
    DROP TABLE loans_staging_table;
    ```

    * create aggregate views that will automatically update when the underlying data changes
    ```sql 
    
    CREATE VIEW vw_loan_amount_by_status
    AS
    select loan_status, sum(loan_amnt) as loan_amount, sum(loan_amnt)/(select sum(loan_amnt) from tbl_loans)*100 as percent_of_total
    from tbl_loans
    group by loan_status
    order by 2 desc
    
    
    CREATE VIEW vw_loan_amount_by_purpose
    AS
    select purpose, sum(loan_amnt) as loan_amount, sum(loan_amnt)/(select sum(loan_amnt) from tbl_loans)*100 as percent_of_total
    from tbl_loans
    group by purpose
    order by 2 desc
    
    
    CREATE VIEW vw_loan_amount_by_grade
    AS
    select grade, sub_grade, sum(loan_amnt) as loan_amount, sum(loan_amnt)/(select sum(loan_amnt) from tbl_loans)*100 as percent_of_total
    from tbl_loans
    group by grade, sub_grade
    order by 1, 2, 3 desc
    ```
    
    * include check constraints on the table for data validation (e.g. the loan amount can't be negative)
    ```sql 
    
    ALTER TABLE tbl_loans
    ADD CONSTRAINT CK_tbl_loans_loan_amnt
    CHECK  (loan_amnt >= 0)
    ```
    
    * inserting new data: here I created a stored procedure that can handle the ingestion of new data in the database (this would work with a more advanced data base like Postgres)
    ```sql   
    USE loans.db
    
    CREATE PROCEDURE [main].[upsertLoan]
      @member_id                                  INT,
      @term                                       VARCHAR(100),
      @grade                                      VARCHAR(100),
      @url                                        VARCHAR(100),
      @sub_grade                                  VARCHAR(100),
      @emp_title                                  VARCHAR(100),
      @emp_length                                 VARCHAR(100),
      @home_ownership                             VARCHAR(100),
      @verification_status                        VARCHAR(100),
      @loan_status                                VARCHAR(100),
      @pymnt_plan                                 VARCHAR(100),
      @desc                                       VARCHAR(100),
      @purpose                                    VARCHAR(100),
      @title                                      VARCHAR(100),
      @zip_code                                   VARCHAR(100),
      @addr_state                                 VARCHAR(100),
      @earliest_cr_line                           VARCHAR(100),
      @initial_list_status                        VARCHAR(100),
      @application_type                           VARCHAR(100),
      @verification_status_joint                  VARCHAR(100),
      @sec_app_earliest_cr_line                   VARCHAR(100),
      @hardship_flag                              VARCHAR(100),
      @hardship_type                              VARCHAR(100),
      @hardship_reason                            VARCHAR(100),
      @hardship_status                            VARCHAR(100),
      @hardship_loan_status                       VARCHAR(100),
      @disbursement_method                        VARCHAR(100),
      @debt_settlement_flag                       VARCHAR(100),
      @settlement_status                          VARCHAR(100),
      @policy_code                                INT,
      @loan_amnt                                  FLOAT,
      @funded_amnt                                FLOAT,
      @revol_bal                                  FLOAT,
      @funded_amnt_inv                            FLOAT,
      @int_rate                                   FLOAT,
      @installment                                FLOAT,
      @annual_inc                                 FLOAT,
      @dti                                        FLOAT,
      @delinq_2yrs                                FLOAT,
      @inq_last_6mths                             FLOAT,
      @mths_since_last_delinq                     FLOAT,
      @mths_since_last_record                     FLOAT,
      @open_acc                                   FLOAT,
      @pub_rec                                    FLOAT,
      @revol_util                                 FLOAT,
      @total_acc                                  FLOAT,
      @out_prncp                                  FLOAT,
      @out_prncp_inv                              FLOAT,
      @total_pymnt                                FLOAT,
      @total_pymnt_inv                            FLOAT,
      @total_rec_prncp                            FLOAT,
      @total_rec_int                              FLOAT,
      @total_rec_late_fee                         FLOAT,
      @recoveries                                 FLOAT,
      @collection_recovery_fee                    FLOAT,
      @last_pymnt_amnt                            FLOAT,
      @collections_12_mths_ex_med                 FLOAT,
      @mths_since_last_major_derog                FLOAT,
      @annual_inc_joint                           FLOAT,
      @dti_joint                                  FLOAT,
      @acc_now_delinq                             FLOAT,
      @tot_coll_amt                               FLOAT,
      @tot_cur_bal                                FLOAT,
      @open_acc_6m                                FLOAT,
      @open_act_il                                FLOAT,
      @open_il_12m                                FLOAT,
      @open_il_24m                                FLOAT,
      @mths_since_rcnt_il                         FLOAT,
      @total_bal_il                               FLOAT,
      @il_util                                    FLOAT,
      @open_rv_12m                                FLOAT,
      @open_rv_24m                                FLOAT,
      @max_bal_bc                                 FLOAT,
      @all_util                                   FLOAT,
      @total_rev_hi_lim                           FLOAT,
      @inq_fi                                     FLOAT,
      @total_cu_tl                                FLOAT,
      @inq_last_12m                               FLOAT,
      @acc_open_past_24mths                       FLOAT,
      @avg_cur_bal                                FLOAT,
      @bc_open_to_buy                             FLOAT,
      @bc_util                                    FLOAT,
      @chargeoff_within_12_mths                   FLOAT,
      @delinq_amnt                                FLOAT,
      @mo_sin_old_il_acct                         FLOAT,
      @mo_sin_old_rev_tl_op                       FLOAT,
      @mo_sin_rcnt_rev_tl_op                      FLOAT,
      @mo_sin_rcnt_tl                             FLOAT,
      @mort_acc                                   FLOAT,
      @mths_since_recent_bc                       FLOAT,
      @mths_since_recent_bc_dlq                   FLOAT,
      @mths_since_recent_inq                      FLOAT,
      @mths_since_recent_revol_delinq             FLOAT,
      @num_accts_ever_120_pd                      FLOAT,
      @num_actv_bc_tl                             FLOAT,
      @num_actv_rev_tl                            FLOAT,
      @num_bc_sats                                FLOAT,
      @num_bc_tl                                  FLOAT,
      @num_il_tl                                  FLOAT,
      @num_op_rev_tl                              FLOAT,
      @num_rev_accts                              FLOAT,
      @num_rev_tl_bal_gt_0                        FLOAT,
      @num_sats                                   FLOAT,
      @num_tl_120dpd_2m                           FLOAT,
      @num_tl_30dpd                               FLOAT,
      @num_tl_90g_dpd_24m                         FLOAT,
      @num_tl_op_past_12m                         FLOAT,
      @pct_tl_nvr_dlq                             FLOAT,
      @percent_bc_gt_75                           FLOAT,
      @pub_rec_bankruptcies                       FLOAT,
      @tax_liens                                  FLOAT,
      @tot_hi_cred_lim                            FLOAT,
      @total_bal_ex_mort                          FLOAT,
      @total_bc_limit                             FLOAT,
      @total_il_high_credit_limit                 FLOAT,
      @revol_bal_joint                            FLOAT,
      @sec_app_inq_last_6mths                     FLOAT,
      @sec_app_mort_acc                           FLOAT,
      @sec_app_open_acc                           FLOAT,
      @sec_app_revol_util                         FLOAT,
      @sec_app_open_act_il                        FLOAT,
      @sec_app_num_rev_accts                      FLOAT,
      @sec_app_chargeoff_within_12_mths           FLOAT,
      @sec_app_collections_12_mths_ex_med         FLOAT,
      @sec_app_mths_since_last_major_derog        FLOAT,
      @deferral_term                              FLOAT,
      @hardship_amount                            FLOAT,
      @hardship_length                            FLOAT,
      @hardship_dpd                               FLOAT,
      @orig_projected_additional_accrued_interest FLOAT,
      @hardship_payoff_balance_amount             FLOAT,
      @hardship_last_payment_amount               FLOAT,
      @settlement_amount                          FLOAT,
      @settlement_percentage                      FLOAT,
      @settlement_term                            FLOAT,
      @issue_d                                    DATETIME,
      @last_pymnt_d                               DATETIME,
      @next_pymnt_d                               DATETIME,
      @last_credit_pull_d                         DATETIME,
      @hardship_start_date                        DATETIME,
      @hardship_end_date                          DATETIME,
      @payment_plan_start_date                    DATETIME,
      @debt_settlement_flag_date                  DATETIME,
      @settlement_date                            DATETIME
    
    AS
      BEGIN
        DECLARE @loan_id_new as INT
        SET @loan_id_new = (select max(loan_id) from tbl_loans)+1
        BEGIN TRANSACTION
    
          INSERT INTO tbl_loans (loan_id, member_id, term, grade, url, sub_grade, emp_title, emp_length, home_ownership, verification_status, loan_status, pymnt_plan, desc, purpose, title, zip_code, addr_state, earliest_cr_line, initial_list_status, application_type, verification_status_joint, sec_app_earliest_cr_line, hardship_flag, hardship_type, hardship_reason, hardship_status, hardship_loan_status, disbursement_method, debt_settlement_flag, settlement_status, policy_code, loan_amnt, funded_amnt, revol_bal, funded_amnt_inv, int_rate, installment, annual_inc, dti, delinq_2yrs, inq_last_6mths, mths_since_last_delinq, mths_since_last_record, open_acc, pub_rec, revol_util, total_acc, out_prncp, out_prncp_inv, total_pymnt, total_pymnt_inv, total_rec_prncp, total_rec_int, total_rec_late_fee, recoveries, collection_recovery_fee, last_pymnt_amnt, collections_12_mths_ex_med, mths_since_last_major_derog, annual_inc_joint, dti_joint, acc_now_delinq, tot_coll_amt, tot_cur_bal, open_acc_6m, open_act_il, open_il_12m, open_il_24m, mths_since_rcnt_il, total_bal_il, il_util, open_rv_12m, open_rv_24m, max_bal_bc, all_util, total_rev_hi_lim, inq_fi, total_cu_tl, inq_last_12m, acc_open_past_24mths, avg_cur_bal, bc_open_to_buy, bc_util, chargeoff_within_12_mths, delinq_amnt, mo_sin_old_il_acct, mo_sin_old_rev_tl_op, mo_sin_rcnt_rev_tl_op, mo_sin_rcnt_tl, mort_acc, mths_since_recent_bc, mths_since_recent_bc_dlq, mths_since_recent_inq, mths_since_recent_revol_delinq, num_accts_ever_120_pd, num_actv_bc_tl, num_actv_rev_tl, num_bc_sats, num_bc_tl, num_il_tl, num_op_rev_tl, num_rev_accts, num_rev_tl_bal_gt_0, num_sats, num_tl_120dpd_2m, num_tl_30dpd, num_tl_90g_dpd_24m, num_tl_op_past_12m, pct_tl_nvr_dlq, percent_bc_gt_75, pub_rec_bankruptcies, tax_liens, tot_hi_cred_lim, total_bal_ex_mort, total_bc_limit, total_il_high_credit_limit, revol_bal_joint, sec_app_inq_last_6mths, sec_app_mort_acc, sec_app_open_acc, sec_app_revol_util, sec_app_open_act_il, sec_app_num_rev_accts, sec_app_chargeoff_within_12_mths, sec_app_collections_12_mths_ex_med, sec_app_mths_since_last_major_derog, deferral_term, hardship_amount, hardship_length, hardship_dpd, orig_projected_additional_accrued_interest, hardship_payoff_balance_amount, hardship_last_payment_amount, settlement_amount, settlement_percentage, settlement_term, issue_d, last_pymnt_d, next_pymnt_d, last_credit_pull_d, hardship_start_date, hardship_end_date, payment_plan_start_date, debt_settlement_flag_date, settlement_date)
                VALUES (@loan_id_new, @member_id, @term, @grade, @url, @sub_grade, @emp_title, @emp_length, @home_ownership, @verification_status, @loan_status, @pymnt_plan, @desc, @purpose, @title, @zip_code, @addr_state, @earliest_cr_line, @initial_list_status, @application_type, @verification_status_joint, @sec_app_earliest_cr_line, @hardship_flag, @hardship_type, @hardship_reason, @hardship_status, @hardship_loan_status, @disbursement_method, @debt_settlement_flag, @settlement_status, @policy_code, @loan_amnt, @funded_amnt, @revol_bal, @funded_amnt_inv, @int_rate, @installment, @annual_inc, @dti, @delinq_2yrs, @inq_last_6mths, @mths_since_last_delinq, @mths_since_last_record, @open_acc, @pub_rec, @revol_util, @total_acc, @out_prncp, @out_prncp_inv, @total_pymnt, @total_pymnt_inv, @total_rec_prncp, @total_rec_int, @total_rec_late_fee, @recoveries, @collection_recovery_fee, @last_pymnt_amnt, @collections_12_mths_ex_med, @mths_since_last_major_derog, @annual_inc_joint, @dti_joint, @acc_now_delinq, @tot_coll_amt, @tot_cur_bal, @open_acc_6m, @open_act_il, @open_il_12m, @open_il_24m, @mths_since_rcnt_il, @total_bal_il, @il_util, @open_rv_12m, @open_rv_24m, @max_bal_bc, @all_util, @total_rev_hi_lim, @inq_fi, @total_cu_tl, @inq_last_12m, @acc_open_past_24mths, @avg_cur_bal, @bc_open_to_buy, @bc_util, @chargeoff_within_12_mths, @delinq_amnt, @mo_sin_old_il_acct, @mo_sin_old_rev_tl_op, @mo_sin_rcnt_rev_tl_op, @mo_sin_rcnt_tl, @mort_acc, @mths_since_recent_bc, @mths_since_recent_bc_dlq, @mths_since_recent_inq, @mths_since_recent_revol_delinq, @num_accts_ever_120_pd, @num_actv_bc_tl, @num_actv_rev_tl, @num_bc_sats, @num_bc_tl, @num_il_tl, @num_op_rev_tl, @num_rev_accts, @num_rev_tl_bal_gt_0, @num_sats, @num_tl_120dpd_2m, @num_tl_30dpd, @num_tl_90g_dpd_24m, @num_tl_op_past_12m, @pct_tl_nvr_dlq, @percent_bc_gt_75, @pub_rec_bankruptcies, @tax_liens, @tot_hi_cred_lim, @total_bal_ex_mort, @total_bc_limit, @total_il_high_credit_limit, @revol_bal_joint, @sec_app_inq_last_6mths, @sec_app_mort_acc, @sec_app_open_acc, @sec_app_revol_util, @sec_app_open_act_il, @sec_app_num_rev_accts, @sec_app_chargeoff_within_12_mths, @sec_app_collections_12_mths_ex_med, @sec_app_mths_since_last_major_derog, @deferral_term, @hardship_amount, @hardship_length, @hardship_dpd, @orig_projected_additional_accrued_interest, @hardship_payoff_balance_amount, @hardship_last_payment_amount, @settlement_amount, @settlement_percentage, @settlement_term, @issue_d, @last_pymnt_d, @next_pymnt_d, @last_credit_pull_d, @hardship_start_date, @hardship_end_date, @payment_plan_start_date, @debt_settlement_flag_date, @settlement_date)
    
        COMMIT TRANSACTION
        end
    GO
    
    
    -- executing the procedure to add a new row 
    EXEC upsertLoan null, ' 36 months', 'D', null, 'D1', 'Administrative', '6 years', 'MORTGAGE', 'Source Verified', 'Current', 'n', null, 'debt_consolidation', 'Debt consolidation', '490xx', 'MI', 'Apr-2011', 'w', 'Individual', null, null, 'N', null, null, null, null, 'Cash', 'N', null, 1, 5000, 5000, 4599, 5000, 17.97, 180.69, 59280, 10.51, 0, 0, null, null, 8, 0, 19.1, 13, 4787.21, 4787.21, 353.89, 353.89, 212.79, 141.1, 0, 0, 0, 180.69, 0, null, null, null, 0, 0, 110299, 0, 1, 0, 2, 14, 7150, 72, 0, 2, 0, 35, 24100, 1, 5, 0, 4, 18383, 13800, 0, 0, 0, 87, 92, 15, 14, 2, 77, null, 14, null, 0, 0, 3, 3, 3, 4, 6, 7, 3, 8, 0, 0, 0, 0, 100, 0, 0, 0, 136927, 11749, 13800, 10000, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'Dec-2018', 'Feb-2019', 'Mar-2019', 'Feb-2019', null, null, null, null, null;
    ``` 

    * an alternative way to add new rows is to run insert statements like the below: 
    ```sql
    INSERT INTO tbl_loans (loan_id, member_id, term, grade, url, sub_grade, emp_title, emp_length, home_ownership, verification_status, loan_status, pymnt_plan, desc, purpose, title, zip_code, addr_state, earliest_cr_line, initial_list_status, application_type, verification_status_joint, sec_app_earliest_cr_line, hardship_flag, hardship_type, hardship_reason, hardship_status, hardship_loan_status, disbursement_method, debt_settlement_flag, settlement_status, policy_code, loan_amnt, funded_amnt, revol_bal, funded_amnt_inv, int_rate, installment, annual_inc, dti, delinq_2yrs, inq_last_6mths, mths_since_last_delinq, mths_since_last_record, open_acc, pub_rec, revol_util, total_acc, out_prncp, out_prncp_inv, total_pymnt, total_pymnt_inv, total_rec_prncp, total_rec_int, total_rec_late_fee, recoveries, collection_recovery_fee, last_pymnt_amnt, collections_12_mths_ex_med, mths_since_last_major_derog, annual_inc_joint, dti_joint, acc_now_delinq, tot_coll_amt, tot_cur_bal, open_acc_6m, open_act_il, open_il_12m, open_il_24m, mths_since_rcnt_il, total_bal_il, il_util, open_rv_12m, open_rv_24m, max_bal_bc, all_util, total_rev_hi_lim, inq_fi, total_cu_tl, inq_last_12m, acc_open_past_24mths, avg_cur_bal, bc_open_to_buy, bc_util, chargeoff_within_12_mths, delinq_amnt, mo_sin_old_il_acct, mo_sin_old_rev_tl_op, mo_sin_rcnt_rev_tl_op, mo_sin_rcnt_tl, mort_acc, mths_since_recent_bc, mths_since_recent_bc_dlq, mths_since_recent_inq, mths_since_recent_revol_delinq, num_accts_ever_120_pd, num_actv_bc_tl, num_actv_rev_tl, num_bc_sats, num_bc_tl, num_il_tl, num_op_rev_tl, num_rev_accts, num_rev_tl_bal_gt_0, num_sats, num_tl_120dpd_2m, num_tl_30dpd, num_tl_90g_dpd_24m, num_tl_op_past_12m, pct_tl_nvr_dlq, percent_bc_gt_75, pub_rec_bankruptcies, tax_liens, tot_hi_cred_lim, total_bal_ex_mort, total_bc_limit, total_il_high_credit_limit, revol_bal_joint, sec_app_inq_last_6mths, sec_app_mort_acc, sec_app_open_acc, sec_app_revol_util, sec_app_open_act_il, sec_app_num_rev_accts, sec_app_chargeoff_within_12_mths, sec_app_collections_12_mths_ex_med, sec_app_mths_since_last_major_derog, deferral_term, hardship_amount, hardship_length, hardship_dpd, orig_projected_additional_accrued_interest, hardship_payoff_balance_amount, hardship_last_payment_amount, settlement_amount, settlement_percentage, settlement_term, issue_d, last_pymnt_d, next_pymnt_d, last_credit_pull_d, hardship_start_date, hardship_end_date, payment_plan_start_date, debt_settlement_flag_date, settlement_date)
    VALUES ((select max(loan_id) from tbl_loans)+1, null, ' 36 months', 'D', null, 'D1', 'Administrative', '6 years', 'MORTGAGE', 'Source Verified', 'Current', 'n', null, 'debt_consolidation', 'Debt consolidation', '490xx', 'MI', 'Apr-2011', 'w', 'Individual', null, null, 'N', null, null, null, null, 'Cash', 'N', null, 1, 5000, 5000, 4599, 5000, 17.97, 180.69, 59280, 10.51, 0, 0, null, null, 8, 0, 19.1, 13, 4787.21, 4787.21, 353.89, 353.89, 212.79, 141.1, 0, 0, 0, 180.69, 0, null, null, null, 0, 0, 110299, 0, 1, 0, 2, 14, 7150, 72, 0, 2, 0, 35, 24100, 1, 5, 0, 4, 18383, 13800, 0, 0, 0, 87, 92, 15, 14, 2, 77, null, 14, null, 0, 0, 3, 3, 3, 4, 6, 7, 3, 8, 0, 0, 0, 0, 100, 0, 0, 0, 136927, 11749, 13800, 10000, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'Dec-2018', 'Feb-2019', 'Mar-2019', 'Feb-2019', null, null, null, null, null);
    ```
    
    * adding a new column to serve as a loan default indentifier 
    ```sql
    ALTER TABLE tbl_loans
    ADD is_default bit
    
    UPDATE tbl_loans SET is_default=1 where loan_status='Default'
    UPDATE tbl_loans SET is_default=0 where loan_status!='Default'
    ```



--- 

#### Future System Improvements

* Further data exploration to identify any potential data issues that have not been detected
* In the current schema the data is hosted in one table because the `id` and `member_id` columns are 100% null, which does not allow us to break down the data into loan-level and member-level. If those columns where populate, I would
modify the data schema by breaking down the tables into:
    * a `loan_data_table`: including the features at the loan level with a unique identifier the loan id (`id`)
    * a `member_data_table`: including the features at the member (borrower) level with a unique identifier the member id (`member_id`)
    * a `loan_to_member_data`: a mapping table that maps the loan id to the member id (`id` -> `member_id`)
