# Google-DataProc-and-Bigquery-retail-sales-Project

In this project we I have worked on GCP DataProc and BigQuery to load, analyse and save the results of Retail Sales data.

Google Dataproc is a fast, easy-to-use, fully managed cloud service for running Apache Spark and Apache Hadoop clusters in a simpler, more cost-efficient way.

Google BigQuery is a Serverless, highly scalable, cost-effective data warehouse designed for business agility.

So we will use BigQuery to load our dataset. We can perform SQL queries to analyse the data in BigQuery aw well. But if we want to write and run jobs in pyspark in multi node cluster DataProc provides us fully managed environment to perform this.

Steps to load and analyse our data:

### [I] LOADING THE DATA
-----------------------------------------------------------------------
# open Google CLI

# 1) creating a dataset in bigquery
bq mk retail_sales_dataset

# 2) storage bucket using gsutil mb (make bucket)
gsutil mb gs://retailsales124


# Note: if running the transformations directly in SSH, open pyspark with .jar file. If running job from Google CLI, specify jar file while running the dataproc job
pyspark --jars=gs://spark-lib/bigquery/spark-bigquery-with-dependencies_2.12-0.23.2.jar 

# 3) performing the following transformation code
from pyspark.sql import SparkSession

spark = SparkSession.builder.master('yarn').appName('spark-bigquery-demo').getOrCreate()

bucket="retailsales124"
spark.conf.set('temporaryGcsBucket', bucket)


# Load data from BigQuery.
sales= spark.read.format('bigquery').option('table', 'satish-practice-project.retail_sales_dataset.sales_table').load()
sales.createOrReplaceTempView('sales_view')



[II] GROUPING THE DATA IN DATAPROC
-------------------------------------------------------------------------------------------------

# (i) Perform word grouping based on total sales by product category
total_sales_group1 = spark.sql('SELECT Product_Category, SUM(Total_amount) as product_sales FROM sales_view GROUP BY Product_Category;')
total_sales_group1.show()

# (ii) Perform word grouping based on number of transactions
no_of_transact_group2 = spark.sql('SELECT Product_Category, COUNT(Transaction_ID) as transaction_count FROM sales_view GROUP BY Product_Category;')
no_of_transact_group2.show()

# (iii) Perform word grouping based on avg transaction value per category
avg_transact_group3 = spark.sql('SELECT Product_Category, ROUND(AVG(Total_Amount),2) as avg_transact_value FROM sales_view GROUP BY Product_Category;')
avg_transact_group3.show()



[III] LOADING THE RESULTS IN BIGQUERY
---------------------------------------------------------------------------------------------------
# Saving the data to BigQuery
total_sales_group1.write.format('bigquery').option('table', 'satish-practice-project.retail_sales_dataset.total_sales_group1').save()

# Saving the data to BigQuery
no_of_transact_group2.write.format('bigquery').option('table', 'satish-practice-project.retail_sales_dataset.no_of_transact_group2').save()

# Saving the data to BigQuery
avg_transact_group3.write.format('bigquery').option('table', 'satish-practice-project.retail_sales_dataset.avg_transact_group3').save()




[IV] PROFITABILITY PER EACH PRODUCT CATEGORY
------------------------------------------------------------------------------------------------
-- Google BigQuery query

SELECT Product_Category, SUM(Total_Amount) as Total_Sales ,ROUND(AVG(Price_per_Unit),2) as avg_price_per_unit_catg
FROM `satish-practice-project.retail_sales_dataset.sales_table` GROUP BY Product_Category;


-- result set is attached as CSV and screenshot


[V] VISUALIZATION
--------------------------------------------------

## Visualization is attached as screenshot

Most profitable: Electronics products

Least profitable: Beauty products


## If we want to run the entire code from google cli, we can save the above pyspark code in retail.py using vi command in terminal and run the below code with updated details

gcloud dataproc jobs submit pyspark retail.py \
    --cluster=cluster-eea5 \
    --region=us-central1 \
    --jars=gs://spark-lib/bigquery/spark-bigquery-latest.jar
