# AWS-Snowflake-Pipeline-Code

This project demonstrates how to build a data pipeline from AWS S3 to Snowflake. It includes creating the necessary storage integration, uploading data to S3, and loading it into Snowflake tables using external stages.

---

## 🗂️ Overview

This guide covers:
- Creating an S3 bucket and uploading files
- Setting up an external stage in Snowflake
- Creating a storage integration between AWS and Snowflake
- Loading data from S3 into Snowflake tables

---

## 🔧 Prerequisites

- AWS Account with access to S3 and IAM
- Snowflake account with appropriate roles and privileges
- Basic SQL and cloud knowledge

---

## 📥 Step 1: Upload Data to S3

1. Create an S3 bucket.
2. Ensure **public access is not blocked** (required for Snowflake access).
3. Create folders inside the bucket and upload your local `.csv` files.

---

## 🏗️ Step 2: Create External Stage in Snowflake

Use the following SQL:

```sql
CREATE STAGE AWS_STG
  URL = 's3://your-bucket-name/folder/'
  STORAGE_INTEGRATION = AWS_INTG;
```

- Replace the URL with your S3 path.
- `AWS_INTG` is the storage integration name.

---

## 🔐 Step 3: Create Storage Integration

```sql
CREATE STORAGE INTEGRATION AWS_INTG
  TYPE = EXTERNAL_STAGE
  ENABLED = TRUE
  STORAGE_PROVIDER = S3
  STORAGE_ALLOWED_LOCATIONS = ('s3://your-bucket-name/folder/')
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::account-id:role/role-name';
```

### Then:

1. Go to AWS → IAM → Roles.
2. Create a new role with **AmazonS3FullAccess**.
3. Copy the **Role ARN** and paste it into the Snowflake integration statement.
4. Paste the **External ID** from Snowflake into the AWS in Trust Relationships,but put temporary external id '0000'.

After running this in Snowflake:
✅ **Integration is created successfully.**

---

## 🔗 Step 4: Link AWS & Snowflake

1. Copy the storage AWS IAM User ARN and paste it into the AWS role trust policy.
2. Update the **External ID** in the trust relationship (replace temporary placeholder like `0000` with the value from Snowflake).

✅ Snowflake and AWS are now linked.

---

## 📤 Step 5: Load Data into Snowflake Table

### Create a table:
```sql

CREATE TABLE T1
(INDEX INT, CUSTOMER_ID VARCHAR, FIRST_NAME VARCHAR, LAST_NAME VARCHAR, COMPANY VARCHAR, CITY VARCHAR, COUNTRY VARCHAR, PHONE_1 VARCHAR, PHONE_2 VARCHAR, EMAIL VARCHAR, SUBSCRIPTION_DATE VARCHAR, WEBSITE VARCHAR);
```

### Create a file format:
```sql
CREATE FILE FORMAT ISV
  TYPE = CSV
  FIELD_DELIMITER = ','
  RECORD_DELIMITER = '\n'
  SKIP_HEADER = 1;
```

### Load data from stage:
```sql
COPY INTO T1
FROM @AWS_STG
FILE_FORMAT = CSV
ON_ERROR = 'CONTINUE';
```

✅ Data successfully loaded into Snowflake from S3.

---

## 👁️ View Data in Stage or Table

To view staged files:
```sql
LIST @AWS_STG;
```

To select data:
```sql
SELECT $1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13, $14
FROM @AWS_STG;
```

To view data in the table:
```sql
SELECT * FROM T1;
```

---

## 📌 Notes

- Use **external stage** for cloud sources like AWS S3.
- Use **internal stage** if you're uploading from a local system.
- Ensure correct permissions and policies are assigned in AWS IAM.

---

## 📅 Date Logged

Notebook Documentation Date: **17-Apr-2025**

---
