# Documentation: AWS Environment and Data Pipeline Setup

## Table of Contents
- [1. Setup Instructions for the AWS Environment and Database](#1-setup-instructions-for-the-aws-environment-and-database)
  - [1.1 Create a Virtual Environment](#11-create-a-virtual-environment)
  - [1.2 AWS Account Setup](#12-aws-account-setup)
  - [1.3 Securing the Root Account](#13-securing-the-root-account)
  - [1.4 Creating an IAM User](#14-creating-an-iam-user)
  - [1.5 Setting Up an RDS Database](#15-setting-up-an-rds-database)
  - [1.6 Setting Up S3 for Data Storage](#16-setting-up-s3-for-data-storage)
  - [1.7 IAM Role for AWS Glue](#17-iam-role-for-aws-glue)
- [2. Details of the Data Pipeline Setup](#2-details-of-the-data-pipeline-setup)
  - [2.1 AWS Glue Data Pipeline](#21-aws-glue-data-pipeline)
  - [2.2 SQL Scripts for Data Transformation](#22-sql-scripts-for-data-transformation)
- [3. Running the Pipeline](#3-running-the-pipeline)
- [4. Additional Considerations](#4-additional-considerations)

## 1. Setup Instructions for the AWS Environment and Database

### 1.1 Create a Virtual Environment
1. **Create a Virtual Environment**:
   ```bash
   python3 -m venv venv
   ```

2. **Activate the Virtual Environment**:
   - On macOS/Linux:
     ```bash
     source venv/bin/activate
     ```
   - On Windows:
     ```bash
     venv\Scripts\activate
     ```

3. **Install Dependencies**:
   Ensure you have a `requirements.txt` file in your project directory.
   ```bash
   pip3 install -r requirements.txt
   ```

### 1.2 AWS Account Setup
1. **Create an AWS Account**: Go to [AWS](https://aws.amazon.com/) and create one.
2. **Sign in to the AWS Management Console**.

### 1.3 Securing the Root Account
1. **Enable Multi-Factor Authentication (MFA)**:
   - Sign in to the AWS Management Console.
   - Go to "My Security Credentials".
   - Click "Activate MFA" and follow the prompts.

### 1.4 Creating an IAM User
1. **Navigate to IAM**: In the AWS Management Console, search for "IAM" and select it.
2. **Create a User**:
   - Click on "Users" and "Add user".
   - Enter a user name (e.g., `etl_user`).
   - Select access types (Console and/or Programmatic).
   - Set a custom password and require a change upon first sign-in.
3. **Set Permissions**: Attach a custom policy with the necessary permissions for the ETL job. Follow the principle of least privilege.

### 1.5 Setting Up an RDS Database
1. **Navigate to RDS**: Search for "RDS" in the AWS Management Console.
2. **Create a Database**: Configure settings such as the instance type, database engine (PostgreSQL or MySQL), and storage.
3. **Configure Security Groups**: Edit inbound rules to limit access to your specific IP or trusted resources.

### 1.6 Setting Up S3 for Data Storage
1. **Navigate to S3**: Search for "S3" in the AWS Management Console.
2. **Create a Bucket**: Name your bucket, choose a region, and configure permissions as required for data storage in your ETL process.

### 1.7 IAM Role for AWS Glue
1. **Create a Role for AWS Glue**:
   - Go to "IAM" in AWS Management Console.
   - Create a new role with the `AWSGlueServiceRole` and attach policies such as `AmazonS3FullAccess` (or restricted S3 access) and `AmazonRDSFullAccess` (or restricted DB access).

## 2. Details of the Data Pipeline Setup

### 2.1 AWS Glue Data Pipeline
1. **Navigate to AWS Glue**: In the AWS Management Console, go to "AWS Glue".
2. **Create a Crawler**:
   - Define your data source (S3 bucket).
   - Set the target data store to RDS or another service.
   - Run the crawler to populate the Glue Data Catalog.
   
3. **Create a Glue Job**: Create an ETL job that uses the data from the Glue Data Catalog and transforms it based on your business logic.

### 2.2 SQL Scripts for Data Transformation
#### Example SQL Scripts for Normalizing Data
- **Create Tables**:
```sql
CREATE TABLE IF NOT EXISTS Distributor (
    DistributorID SERIAL PRIMARY KEY,
    DistributorName VARCHAR(255) NOT NULL
);

CREATE TABLE IF NOT EXISTS Product (
    ProductID SERIAL PRIMARY KEY,
    ShellSKUNumber VARCHAR(255) NOT NULL,
    ProductDescription VARCHAR(255)
);

CREATE TABLE IF NOT EXISTS Sales (
    SalesID SERIAL PRIMARY KEY,
    DistributorID INT,
    ProductID INT,
    SalesMonth INT,
    SalesYear INT,
    DFOAQuantity INT,
    NonDFOAQuantity INT,
    FOREIGN KEY (DistributorID) REFERENCES Distributor(DistributorID),
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID)
);
```

- **Insert Data**:
```sql
INSERT INTO Distributor (DistributorName) VALUES ('X Petroleum');

INSERT INTO Product (ShellSKUNumber, ProductDescription) VALUES ('ABC123-GL', 'Product A Description');

INSERT INTO Sales (DistributorID, ProductID, SalesMonth, SalesYear, DFOAQuantity, NonDFOAQuantity)
VALUES (1, 1, 8, 2024, 1000, 0);
```

## 3. Running the Pipeline
1. **Run the Glue Job**: Execute the Glue job to process and transform the data.
2. **Verify Data in RDS**: Check the RDS database to ensure the data has been correctly loaded.

## 4. Additional Considerations
- **Monitoring and Logging**: Use AWS CloudWatch to monitor your Glue jobs and track errors or performance issues.
- **Cost Management**: Monitor costs using AWS Cost Explorer, especially for Glue and RDS usage.
- **Security**: Follow the principle of least privilege for IAM roles and policies. Regularly review and update your security settings.