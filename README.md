# Redfin Data Pipeline Project

## Overview
This project demonstrates the process of extracting real estate data from the Redfin Data Center, transforming it using PySpark on Amazon EMR, and storing the final transformed data into an S3 bucket. We leverage Amazon EMR for scalable data processing and use Jupyter Notebook as our development environment. The workflow includes raw data extraction from the Redfin data source, cleaning and transforming the data, and finally storing the results in the S3 bucket.

**Data Source:** [Redfin Data Center](https://www.redfin.com/news/data-center/)

---

## Architecture Diagram

![](https://github.com/vighneshbuddhivant/Redfin-Emr-Data-Pipeline-Project/blob/efe70cea9b3158a237d9dd4c910681baae6e09dc/emrpipeline.png)

---

## Architecture Overview

The overall architecture follows these steps:
1. **User Account Creation**: A user account is created to grant appropriate permissions to work with Amazon EMR and S3.
2. **S3 Buckets**: We create two S3 buckets — one for storing raw data and another for storing transformed data.
3. **VPC Creation**: A dedicated Virtual Private Cloud (VPC) is created to run our EMR cluster.
4. **EMR Cluster Setup**: An EMR cluster is launched, ensuring proper configuration of VPC, logging, and EC2 key pairs.
5. **EMR Studio & Jupyter Notebook**: EMR Studio is used to create a Jupyter Notebook where we attach the EMR cluster and run our PySpark code.
6. **PySpark Transformation**: The data is extracted, cleaned, and transformed using PySpark in the Jupyter Notebook.
7. **Data Storage**: The final transformed data is written back to an S3 bucket in parquet format.

---

## Workflow

1. **Create User Account**: Create a user with administrative access and generate access keys for EMR usage.
2. **S3 Bucket Setup**: 
   - Create two S3 buckets: 
     - One for raw data.
     - One for transformed data.
   - Optionally, create another S3 bucket for EMR Studio logs.
3. **Create VPC**: Set up a VPC for running EMR in a specific network environment.
4. **Create EMR Cluster**: Configure and create an EMR cluster with necessary parameters like VPC, log bucket, EC2 key pair, etc.
5. **Set Up EMR Studio**: 
   - Launch EMR Studio.
   - Attach the correct VPC and S3 bucket.
   - Assign the necessary IAM roles (for S3 and EMR access).
6. **Create Jupyter Notebook**:
   - Create a workspace and launch a Jupyter Notebook inside EMR Studio.
   - Attach the EMR cluster to the notebook.
7. **PySpark Code**: 
   - Select the PySpark kernel.
   - Run the data extraction, transformation, and loading (ETL) code (explained in detail below).
8. **Store Data in S3**: Write the transformed data back to an S3 bucket as a parquet file.

---

## Tools and Technologies Used

- **Amazon EMR**: To manage and run Apache Spark clusters for large-scale data processing.
- **Amazon S3**: For storing raw and transformed data.
- **PySpark**: For data extraction and transformation.
- **Jupyter Notebook**: For interactive development within EMR Studio.
- **AWS IAM**: For managing permissions.
- **VPC**: To control the network environment for running EMR.

---

## PySpark Code Explanation

You can create a new file to explain the PySpark code step by step. Here's an example layout:

**File: `pyspark_code_explanation.md`**

### 1. Importing Libraries

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col
```

- **SparkSession**: Used to create a connection with the Spark framework.
- **col**: Helps refer to column names within the DataFrame.

---

### 2. Data Extraction from Redfin Source

```bash
%%bash
wget -O - https://redfin-public-data.s3.us-west-2.amazonaws.com/redfin_market_tracker/city_market_tracker.tsv000.gz | aws s3 cp - .tsv000.gz
```

- **wget**: Downloads the data file from the Redfin data source.
- **aws s3 cp**: Copies the downloaded file to the S3 bucket.

---

### 3. Create Spark Session

```python
spark = SparkSession.builder.appName("RedfinDataAnalysis").getOrCreate()
```

- This initializes the Spark session to process data.

---

### 4. Read the Data

```python
redfin_data = spark.read.csv("s3://redfin-project-staging-bucket/city_market_tracker.tsv000.gz", header=True, inferSchema=True, sep="\t")
```

- **read.csv**: Reads the data from the specified S3 path as a CSV file. 
- **inferSchema**: Automatically infers the data types of each column.
- **sep**: Defines the delimiter used in the TSV file.

---

### 5. Display Sample Data

```python
redfin_data.show(3)
```

- This shows the first three rows of the dataset to confirm successful data loading.

---

### 6. Check Schema

```python
redfin_data.printSchema()
```

- Displays the structure and types of the columns in the dataset.

---

### 7. Selecting Important Columns

```python
df_redfin = redfin_data.select(['period_end','period_duration', 'city', 'state', 'property_type', 'median_sale_price', 'median_ppsf', 'homes_sold', 'inventory', 'months_of_supply', 'median_dom', 'sold_above_list', 'last_updated'])
```

- We are selecting only the necessary columns for our analysis.

---

### 8. Handling Missing Values

```python
df_redfin = df_redfin.na.drop()
```

- **na.drop**: Removes rows with missing values.

---

### 9. Extracting Year and Month

```python
from pyspark.sql.functions import year, month
df_redfin = df_redfin.withColumn("period_end_yr", year(col("period_end")))
df_redfin = df_redfin.withColumn("period_end_month", month(col("period_end")))
```

- Extracts the year and month from the `period_end` column.

---

### 10. Mapping Month Numbers to Names

```python
from pyspark.sql.functions import when
df_redfin = df_redfin.withColumn("period_end_month", when(col("period_end_month") == 1, "January").when(...).otherwise("Unknown"))
```

- This replaces the numeric month values with their respective month names.

---

### 11. Writing Transformed Data to S3

```python
s3_bucket = "s3://redfin-transform-zone-yml/redfin_data.parquet"
df_redfin.write.mode("overwrite").parquet(s3_bucket)
```

- Writes the transformed DataFrame back to the S3 bucket in parquet format.

---

## Conclusion
This project showcases how to efficiently extract, transform, and load (ETL) data using Amazon EMR and PySpark. The scalability and flexibility of EMR combined with the power of PySpark allowed us to process and analyze large datasets effectively. The final transformed data is stored in an S3 bucket for further analysis or reporting.
