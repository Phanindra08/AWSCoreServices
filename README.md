# AWS Data Processing Pipeline

This repository details the end-to-end serverless data processing pipeline created on AWS. The process involved ingesting raw data into S3, using a Lambda function for automated processing, cataloging the processed data with AWS Glue, running analytical queries with Amazon Athena, and visualizing the results on a dynamic webpage hosted on an EC2 instance.
Each section below includes an explanation of the approach and a screenshot.

## 1. Amazon S3 Bucket Structure 🪣

### Approach
Created a central Amazon S3 bucket to serve as the foundation for the entire data pipeline. To organize the workflow, established three distinct folders within this bucket:

* **`raw/`**: The folder acts as the ingestion point. All input CSV files are uploaded here.
* **`processed/`**: The folder stores the clean, filtered data output by the Lambda function after it processes the files from the `raw/` folder.
* **`enriched/`**: The folder is the designated output location for Amazon Athena. It stores the `.csv` files generated as results from the SQL queries.

### Screenshot: 
<img width="780" height="620" alt="S3BucketStructure" src="https://github.com/user-attachments/assets/1c25c723-296d-413a-8b51-38394415cae7" />

---
## 2. IAM Roles and Permissions 🔐

### Approach
To ensure secure, permission-based interaction between AWS services, created three separate IAM roles:
1.  **Lambda Execution Role (`Lambda-S3-Processing-Role`)**: The role was attached to the Lambda function. It grants permissions for Lambda to read objects from the `raw/` S3 folder, write new objects to the `processed/` S3 folder, and write logs to CloudWatch. `AWSLambdaBasicExecutionRole` and `AmazonS3FullAccess` are the policies granted for this role.
2.  **Glue Service Role (`Glue-S3-Crawler-Role`)**: This role was used by the AWS Glue Crawler. It provides permissions to access the `processed/` data in S3 and the `AWSGlueServiceRole` policy to manage the Glue Data Catalog (creating databases and tables).
3.  **EC2 Instance Profile (`EC2-Athena-Dashboard-Role`)**: This role was attached to the EC2 instance. It grants the instance permissions to run queries with `AmazonAthenaFullAccess` and to read/write query results from S3 using `AmazonS3FullAccess`.

### Screenshot: IAM Roles



---

## 3. AWS Lambda Function ⚙️

### Approach

I created a serverless compute function using AWS Lambda to automate the data cleaning process. The function, named `FilterAndProcessOrders`, uses the Python 3.9 runtime. It is configured to use the existing `Lambda-S3-Processing-Role` created in the previous step. The function's code is designed to be triggered by an S3 event, read the uploaded CSV file, perform data filtering and transformations, and then save the cleaned data back to the `processed/` S3 folder.

### Screenshot: Lambda Function



---

## 4. S3 Trigger Configuration ⚡

### Approach

To make the pipeline event-driven, I added an S3 trigger to the `FilterAndProcessOrders` Lambda function. This trigger is configured to monitor the S3 bucket for **All object create events**. To ensure the function only runs on relevant files, I specified a **prefix** of `raw/` and a **suffix** of `.csv`. This configuration automatically invokes the Lambda function *only* when a new CSV file is uploaded to the `raw/` folder.

### Screenshot: Configured Trigger



---

## 5. Processed Data in S3

### Approach

After configuring the Lambda function and its S3 trigger, I tested the pipeline by uploading the `Orders.csv` file to the `raw/` folder of the S3 bucket. This action successfully triggered the Lambda function, which executed its processing logic. As a result, a new, cleaned CSV file appeared in the `processed/` folder, confirming the automated workflow was successful.

### Screenshot: Processed CSV File in S3



---

## 6. AWS Glue Crawler 🕸️

### Approach

To make the processed data queryable by Athena, I used AWS Glue to create a data catalog. I created a new crawler named `orders_processed_crawler` and pointed it to the `processed/` S3 folder. I assigned the `Glue-S3-Crawler-Role` to it. The crawler was configured to output its findings to a new database named `orders_db`. Upon running the crawler, it successfully scanned the processed CSV, inferred its schema, and created a new table within the `orders_db` database.

### Screenshot: Crawler CloudWatch (Run Details)



---

## 7. Amazon Athena Query Results in S3

### Approach

With the data cataloged, I navigated to the Amazon Athena service. I set my query results location to the `enriched/` S3 folder. Using the Trino SQL editor, I successfully ran analytical queries against the `orders_db` database and its table. The web application hosted on EC2 also runs these queries, and the results of each query are automatically saved as new CSV and metadata files in the `enriched/` S3 folder.

### Screenshot: Athena Query CSV Files in S3



---

## 8. Final EC2 Webpage Visualization 🖥️

### Approach

Finally, to display the query results, I launched a `t2.micro` EC2 instance. I attached the `EC2-Athena-Dashboard-Role` and configured the security group to allow SSH (port 22) and HTTP access on port 5000. After connecting via SSH, I installed Python, Flask, and Boto3. I then configured and ran the `app.py` script, which starts a Flask web server. This server executes the Athena queries in real-time and renders the results in a web browser.

### Screenshot: Final Webpage Result
