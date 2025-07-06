# Data Ingestion from S3 to RDS with Fallback to AWS Glue using Dockerized Python Application

## ✨ Project Title:

**Data Ingestion from S3 to RDS with Fallback to AWS Glue using Dockerized Python Application**

---

## 🚀 Objective:

Build a Docker-based Python solution that:

- Downloads a CSV file from an AWS S3 bucket
- Inserts the data into a MySQL RDS table
- If RDS upload fails, automatically falls back to store data into AWS Glue Table

---

## 🛌 AWS Services Used:

- Amazon S3
- Amazon RDS (MySQL)
- AWS Glue
- IAM Roles (For credentials)
- EC2 (Ubuntu - For Docker app execution)

---

## ⚙️ Tech Stack:

- Python (with `boto3`, `sqlalchemy`, `pymysql`)
- Docker
- AWS CLI

---

## 📂 Folder Structure:

```
project-root/
|
|-- Dockerfile
|-- requirements.txt
|-- app/
|   |-- ingest.py
|
|-- README.md
```

---

## 🌐 Step-by-Step Setup & Execution:

### ✅ STEP 1: Prepare CSV File

Create a CSV file `sample.csv` with user data. Sample format:

```
id,name,email,command,contacts
1,Prashant,prashantgharate555@gmail.com,HI I am Prashant & you,9837473930
...
```

### ✅ STEP 2: Upload CSV to S3 Bucket

- Bucket Name: `s3-to-rds-fallback-demo`
- File: `sample.csv`

### ✅ STEP 3: RDS Setup

- Created MySQL RDS instance
- DB Name: `datadb`
- Table Name: `users`

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    command VARCHAR(100),
    contacts VARCHAR(100)
);
```

- Users created:

```sql
CREATE USER 'raj'@'%' IDENTIFIED BY 'Word$123';
GRANT ALL PRIVILEGES ON datadb.* TO 'raj'@'%';
FLUSH PRIVILEGES;
```

- Security Group updated to allow EC2 access on port 3306 (MySQL)

### ✅ STEP 4: Docker App Setup

#### Dockerfile

```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ ./app/
CMD ["python", "./app/ingest.py"]
```

#### requirements.txt

```
boto3
pymysql
sqlalchemy
```

#### app/ingest.py (Main Logic)

- Downloads CSV from S3
- Tries to insert into RDS
- On failure, falls back to AWS Glue

### ✅ STEP 5: Build Docker Image

```bash
sudo docker build -t s3-to-rds-app .
```

### ✅ STEP 6: Run Docker Container with ENV Vars

```bash
sudo docker run \
  -e "AWS_ACCESS_KEY_ID=..." \
  -e "AWS_SECRET_ACCESS_KEY=..." \
  -e "AWS_DEFAULT_REGION=ap-south-1" \
  -e "S3_BUCKET=s3-to-rds-fallback-demo" \
  -e "S3_KEY=sample.csv" \
  -e "RDS_HOST=datadb.c3qqgwcsofyl.ap-south-1.rds.amazonaws.com" \
  -e "RDS_USER=raj" \
  -e "RDS_PASS=Word\$123" \
  -e "RDS_DB=datadb" \
  -e "GLUE_DB=fallbackdb" \
  -e "GLUE_TABLE=users_fallback" \
  -e "S3_LOCATION=s3://s3-to-rds-fallback-demo/" \
  s3-to-rds-app
```

---

## 🚫 Error Handling

- **RDS Access Denied / User issue**: Solved by creating new MySQL user `raj`
- **Duplicate Key (1062)**: If CSV file contains existing primary key `id`, insert fails
- **S3 File Not Found (404)**: Fixed by checking correct filename and location
- **Glue Table Exists**: If table already exists, Glue throws AlreadyExistsException

---

## ✈️ Final Output

- Docker logs:

```
CSV downloaded from S3
Data uploaded to RDS successfully
```

- MySQL query confirms inserted data:

```sql
SELECT * FROM users;
```

