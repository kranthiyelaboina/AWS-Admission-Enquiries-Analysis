
# Admission Inquiry Analysis

This repository contains a detailed project that analyzes admission inquiries using AWS services. The project has been updated to use the `students_data.csv` file, which contains synthetic data for students’ admission inquiries. All instructions, AWS configurations, and SQL commands have been updated to suit the new data structure.

![Project Workflow](https://github.com/kranthiyelaboina/AWS-Admission-Enquiries-Analysis/blob/c85ebe2c285b9ff3330033a61f8a3a932e8f0f92/Project_PNG/Admission-project.drawio.png)

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Data Preparation](#data-preparation)
3. [AWS S3 Bucket Setup](#aws-s3-bucket-setup)
4. [AWS Glue Setup](#aws-glue-setup)
5. [AWS Redshift Cluster Setup](#aws-redshift-cluster-setup)
6. [SQL Commands & Data Analysis](#sql-commands--data-analysis)
7. [Amazon QuickSight Visualization](#amazon-quicksight-visualization)
8. [Pushing to GitHub](#pushing-to-github)

---

## Project Overview

This project demonstrates a comprehensive workflow to analyze student admission inquiries using AWS. The project now uses the updated `students_data.csv` dataset, which contains the following columns:

- **Student_ID**: A unique identifier for each student.
- **Name**: Full name of the student.
- **Email**: College email address.
- **Preferred_Domain**: The domain or engineering specialization the student is interested in (e.g., Aeronautical_Engineering, Data_Science, Mechanical, IT, etc.).
- **JEE_Score**: The student’s entrance exam score.
- **Admission_Enquiry**: The type of admission enquiry (e.g., BTech, MTech).

This guide details the end-to-end process—from uploading data to AWS S3 to setting up Glue, Redshift, and visualizations in Amazon QuickSight.

---

## Data Preparation

1. **Dataset Overview:**
   - **Filename:** `students_data.csv`
   - **Structure:**
     - **Student_ID:** Unique identifier (e.g., STU00001)
     - **Name:** Full name of the student
     - **Email:** Generated college email (e.g., student@college.edu)
     - **Preferred_Domain:** Domain of interest (e.g., Aeronautical_Engineering, Data_Science)
     - **JEE_Score:** Numeric value representing the entrance exam score
     - **Admission_Enquiry:** Type of enquiry (e.g., BTech, MTech)

2. **Generating/Validating Data:**
   - If you are creating your own synthetic dataset, ensure it has at least 8000 records with the above attributes.
   - Use tools like Python, Excel, or any data generation utility to build the dataset.

---

## AWS S3 Bucket Setup

1. **Create an S3 Bucket:**
   - Log in to the AWS Management Console.
   - Navigate to S3 and create a new bucket, for example: `admission-enquiries-data`.

2. **Upload the Dataset:**
   - Upload the `students_data.csv` file into the newly created S3 bucket.
   - Verify the upload via the AWS S3 console.

![Bucket Uploaded Successfully](https://github.com/kranthiyelaboina/AWS-Admission-Enquiries-Analysis/blob/369f209674fb27dc906f07e5e8c05321eecf010f/Project_PNG/Screenshot%20(44).png)

---

## AWS Glue Setup

1. **Create an AWS Glue Crawler:**

   - Navigate to AWS Glue → Crawlers → Click *Add Crawler*.
   - Name the crawler (e.g., `admission_crawler`).
   - Set the data store to S3 and provide the path:
     ```
     s3://admission-enquiries-data/
     ```
   - Click Next and choose *Create IAM Role* (e.g., `AWSGlueServiceRole-admission`).
   - Select/Create an AWS Glue database (e.g., `admission_db`).

2. **Run the Crawler & Verify Metadata:**
   - Run the crawler to catalog the schema.
   - Confirm that a table (e.g., `students_data`) is created in the `admission_db` database with the proper schema reflecting the columns in `students_data.csv`.

![Glue Crawler](https://github.com/kranthiyelaboina/AWS-Admission-Enquiries-Analysis/blob/369f209674fb27dc906f07e5e8c05321eecf010f/Project_PNG/Screenshot%20(45).png)

---

## AWS Redshift Cluster Setup

1. **Create a Redshift Cluster:**

   - Navigate to Amazon Redshift and create a new cluster. For free-tier eligibility, use `dc2.large` with 1 node.
   - **Cluster Configuration:**
     - **Cluster Name:** admission-cluster
     - **Node Type:** dc2.large
     - **Database Name:** admission_db
     - **Master Username:** admin
     - **Master Password:** YourSecurePassword

2. **Set Up IAM Role for Redshift Access:**

   - In AWS IAM, create a role for Redshift.
   - Attach policies:
     - AmazonS3ReadOnlyAccess
     - AWSGlueConsoleFullAccess
   - Name the role (e.g., `RedshiftGlueS3Role`) and copy the IAM Role ARN.

3. **Connect Redshift to the AWS Glue Data Catalog:**

   - In the Redshift console, go to Clusters → Select your cluster → Query Editor v2.
   - Under *Manage Settings*, enable connection to the AWS Glue Data Catalog.
   - Select the database `admission_db`.

4. **Create Table in Redshift:**

   Run the following SQL command in the Redshift Query Editor to create a table that matches the structure of `students_data.csv`:

   ```sql
   CREATE TABLE students_data (
       Student_ID VARCHAR(20),
       Name VARCHAR(100),
       Email VARCHAR(100),
       Preferred_Domain VARCHAR(100),
       JEE_Score INT,
       Admission_Enquiry VARCHAR(20)
   );
   ```

   ![Table Creation](https://github.com/kranthiyelaboina/AWS-Admission-Enquiries-Analysis/blob/369f209674fb27dc906f07e5e8c05321eecf010f/Project_PNG/Screenshot%20(46).png)

5. **Load Data from S3 into Redshift:**

   Use the following COPY command (replace `<your-account-id>` and `<your-role-arn>` accordingly):

   ```sql
   COPY students_data
   FROM 's3://admission-enquiries-data/students_data.csv'
   IAM_ROLE 'arn:aws:iam::<your-account-id>:role/RedshiftGlueS3Role'
   FORMAT AS CSV IGNOREHEADER 1;
   ```

   ![Data Loading](https://github.com/kranthiyelaboina/AWS-Admission-Enquiries-Analysis/blob/369f209674fb27dc906f07e5e8c05321eecf010f/Project_PNG/Screenshot%20(47).png)

6. **Verify Data Load:**

   Run the following query to verify that the data has been loaded correctly:

   ```sql
   SELECT * FROM students_data WHERE Preferred_Domain = 'Data_Science' AND JEE_Score > 100;
   ```

---

## SQL Commands & Data Analysis

Below are some example SQL queries to perform basic analysis on the `students_data` table:

1. **List All Admission Inquiries:**

   ```sql
   SELECT * FROM students_data;
   ```

2. **Filter by Admission Enquiry Type (e.g., BTech):**

   ```sql
   SELECT * FROM students_data WHERE Admission_Enquiry = 'BTech';
   ```

3. **Students Interested in a Specific Domain (e.g., Aeronautical_Engineering):**

   ```sql
   SELECT * FROM students_data WHERE Preferred_Domain = 'Aeronautical_Engineering';
   ```

4. **Students with High JEE Scores (e.g., greater than 150):**

   ```sql
   SELECT * FROM students_data WHERE JEE_Score > 150;
   ```

5. **Combined Query for Specific Analysis:**

   For example, to retrieve students who enquired for BTech admission in Data Science with a JEE score above 120:

   ```sql
   SELECT * FROM students_data 
   WHERE Admission_Enquiry = 'BTech' 
     AND Preferred_Domain = 'Data_Science'
     AND JEE_Score > 120;
   ```

Feel free to modify these queries further to suit more specific analysis requirements.

---

## Amazon QuickSight Visualization

1. **Sign Up and Configure QuickSight:**
   - Navigate to Amazon QuickSight in the AWS Console.
   - Complete the signup process (Standard Edition is free-tier eligible).
   - Choose data sources including S3, Athena, and Redshift.

2. **Connect QuickSight to Redshift:**
   - In QuickSight, add a new dataset.
   - Select Redshift and enter:
     - **Cluster Name:** admission-cluster
     - **Database:** admission_db
     - **Username:** admin
     - **Password:** YourSecurePassword
   - Validate the connection and choose the `students_data` table.

3. **Create Visualizations:**
   - **Admission Trends:** Use a bar chart showing counts per Preferred_Domain.
   - **JEE Score Analysis:** Create a histogram to display score distribution.
   - **Enquiry Type Analysis:** Use a pie chart to depict the proportion of BTech vs. MTech enquiries.

   After selecting and preparing the dataset, click **Save & Visualize** to begin creating interactive dashboards.

![Bar Plot](https://github.com/kranthiyelaboina/AWS-Admission-Enquiries-Analysis/blob/369f209674fb27dc906f07e5e8c05321eecf010f/Project_PNG/Screenshot%20(48).png)
![Stacked Bar Chart](https://github.com/kranthiyelaboina/AWS-Admission-Enquiries-Analysis/blob/369f209674fb27dc906f07e5e8c05321eecf010f/Project_PNG/Screenshot%20(49).png)

---

## Pushing to GitHub

To push your project to GitHub, follow these steps:

```bash
git init
git add .
git commit -m "Initial commit of Admission Inquiry Analysis project with updated students_data.csv"
git remote add origin <repository-url>
git push -u origin main
```

Replace `<repository-url>` with the URL of your GitHub repository.

---
