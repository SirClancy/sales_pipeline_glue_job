# Documentation: AWS Environment and Data Pipeline Setup

## 1. Setup Instructions for the AWS Environment and Database

### 1.1 Create a Virtual Environment
1. **Create a Virtual Environment**:
   - Navigate to your project directory in the terminal.
   - Run the following command to create a virtual environment:
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
   - Ensure you have a `requirements.txt` file in your project directory.
   - Run the following command to install the necessary packages:
     ```bash
     pip3 install -r requirements.txt
     ```

### 1.2 AWS Account Setup
1. **Create an AWS Account**: If you don't have an account, go to [AWS](https://aws.amazon.com/) and create one.
2. **Sign in to the AWS Management Console**.

### 1.3 Securing the Root Account
1. **Enable Multi-Factor Authentication (MFA)**:
   - Sign in to the AWS Management Console using your root account.
   - Go to "My Security Credentials".
   - Under "Multi-Factor Authentication (MFA)", click "Activate MFA".
   - Follow the prompts to set up your MFA device (e.g., a smartphone app).

### 1.4 Creating an IAM User
1. **Navigate to IAM**:
   - In the AWS Management Console, search for "IAM" and select it.

2. **Create a User**:
   - Click on "Users" and then "Add user".
   - Enter a user name (e.g., `etl_user`).
   - Select "AWS Management Console access" and/or "Programmatic access" based on your needs.
   - Set a custom password and require the user to change it upon first sign-in.

3. **Set Permissions**:
   - Choose "Attach existing policies directly".
   - Create and attach a custom policy with only the necessary permissions for the ETL job:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": [
             "s3:GetObject",
             "s3:PutObject",
             "rds:DescribeDBInstances",
             "rds:Connect",
             "glue:*"
           ],
           "Resource": [
             "arn:aws:s3:::your-bucket-name/*",
             "arn:aws:rds:your-region:your-account-id:db:your-db-name",
             "arn:aws:glue:your-region:your-account-id:*"
           ]
         }
       ]
     }
     ```
   - Replace `your-bucket-name`, `your-region`, and `your-account-id` with your actual values.
   - Click "Next: Tags" to add tags if needed, then "Next: Review" and finally "Create user".

4. **Set Up MFA for the IAM User**:
   - After creating the user, select the user from the list.
   - Under "Security credentials", click "Manage" next to "Assigned MFA device".
   - Follow the prompts to set up MFA for this user.

### 1.5 Setting Up an RDS Database
1. **Navigate to RDS**:
   - In the AWS Management Console, search for "RDS" and select it.
   
2. **Create a Database**:
   - Click on "Create database".
   - Choose "Standard create".
   - Select the database engine (e.g., PostgreSQL, MySQL).
   - Configure the following settings:
     - **DB instance class**: Choose the instance size (e.g., db.t3.micro for free tier).
     - **Storage**: Specify the storage size (e.g., 20 GB).
     - **DB instance identifier**: Give your database a unique name.
     - **Master username**: Set a username (e.g., admin).
     - **Master password**: Set a strong password.
   - Click on "Create database".

3. **Configure Security Groups**:
   - Go to the "VPC" section and select "Security Groups".
   - Locate the security group associated with your RDS instance.
   - Edit inbound rules to allow access only from specific IP addresses or ranges, rather than `0.0.0.0/0`:
     - Type: `PostgreSQL` or `MySQL`
     - Protocol: `TCP`
     - Port: `5432` (PostgreSQL) or `3306` (MySQL)
     - Source: Your specific IP or CIDR block.

4. **Connect to the Database**:
   - Use a database client (e.g., pgAdmin for PostgreSQL) or a programming language (e.g., Python with psycopg2 for PostgreSQL) to connect using the endpoint provided in the RDS console.

### 1.6 Setting Up S3 for Data Storage
1. **Navigate to S3**:
   - In the AWS Management Console, search for "S3" and select it.
   
2. **Create a Bucket**:
   - Click on "Create bucket".
   - Enter a globally unique bucket name and select a region.
   - Configure settings (versioning, encryption) as needed.
   - **Set Bucket Policies**: Limit access to specific IAM roles or users to adhere to the principle of least privilege.
   - Click "Create bucket".

### 1.7 IAM Role for AWS Glue
1. **Create a Role**:
   - Click on "Roles" and then "Create role".
   - Choose "AWS Service" and select "Glue".
   - Attach only the necessary policies:
     - **Custom Policy for S3 Access**: Create a custom policy that grants only the required permissions for accessing specific S3 buckets.
     - **Custom Policy for RDS Access**: Create a custom policy that grants only the necessary permissions for your RDS instance.
   - Complete the role creation.

## 2. Details of the Data Pipeline Setup

### 2.1 AWS Glue Data Pipeline
1. **Navigate to AWS Glue**:
   - In the AWS Management Console, search for "Glue" and
