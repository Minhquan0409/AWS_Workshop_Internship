---

title: "Prerequisites"
weight: 2
chapter: false
pre: "<b>5.2. </b>"
-------------------

## Objective

Before deploying **SkyLink Airline** to AWS, the development environment, source code, AWS account, and required tools must be prepared.

The objective of this section is to ensure that participants can:

* Run SkyLink successfully in a local environment.
* Verify the React/Vite frontend.
* Verify the Laravel backend.
* Connect the backend to MySQL.
* Install and configure AWS CLI.
* Prepare the required deployment information.
* Follow basic IAM and least-privilege principles.

---

## 1. Required Tools and Resources

### 1.1. SkyLink Airline Source Code

This workshop uses the **SkyLink Airline System** as the application to be deployed to AWS.

The project consists mainly of:

```text
SkyLink-Airline-System
│
├── airline-frontend
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.*
│
└── airline-backend
    ├── app/
    ├── routes/
    ├── database/
    ├── public/
    ├── .env
    └── composer.json
```

> The actual directory structure may vary depending on the source-code version.

**Source code:**

`https://github.com/quangg199/SkyLink-Airline-System`

---

### 1.2. AWS Account

An active AWS account is required.

The account will be used to create and manage the AWS resources required by the workshop.

The planned AWS services are:

| Category       | AWS Service               |
| -------------- | ------------------------- |
| Networking     | Amazon VPC                |
| Frontend       | Amazon S3                 |
| CDN            | Amazon CloudFront         |
| Security       | AWS WAF                   |
| Backend        | Amazon EC2                |
| Load Balancing | Application Load Balancer |
| Database       | Amazon RDS for MySQL      |
| Messaging      | Amazon SQS                |
| Serverless     | AWS Lambda                |
| Monitoring     | Amazon CloudWatch         |
| Notification   | Amazon SNS                |
| Identity       | AWS IAM                   |

---

### 1.3. AWS Region

A single AWS Region should be used for most resources.

Example:

```text
Region: Asia Pacific (Singapore)
Region code: ap-southeast-1
```

Using a single Region makes the environment easier to manage and reduces unnecessary complexity.

> If another Region is used, replace `ap-southeast-1` with the actual Region throughout the workshop.

**Screenshot:**

```text
static/images/workshop/5-2/aws-region.png
```

![AWS Region](/images/5-Workshop/5.2-Prerequisite/aws-region.png?v=2)

---

### 1.4. Personal Computer

The computer used for this workshop should have a stable Internet connection.

For Windows:

```text
Operating System:
Windows 10 / Windows 11

Terminal:
Git Bash / PowerShell / CMD

Browser:
Google Chrome / Microsoft Edge
```

---

### 1.5. Node.js and npm

Node.js is required to install dependencies and build the React/Vite frontend.

Check the installed versions:

```bash
node -v
npm -v
```

Example:

```text
Node.js: v20.x
npm: 10.x
```

Navigate to the frontend:

```bash
cd airline-frontend
```

Install dependencies:

```bash
npm install
```

Build the frontend:

```bash
npm run build
```

A successful build normally creates:

```text
dist/
```

**Code snippet:**

```bash
node -v
npm -v

cd airline-frontend
npm install
npm run build
```
---

### 1.6. PHP, Composer and Laravel

The SkyLink backend is built with Laravel.

The following tools are required:

```text
PHP
Composer
Laravel
```

Check:

```bash
php -v
composer -V
```

Navigate to the backend:

```bash
cd airline-backend
```

Install dependencies:

```bash
composer install
```

Check Laravel:

```bash
php artisan --version
```
---

### 1.7. MySQL

SkyLink uses MySQL as its relational database.

For local development, MySQL can be provided through XAMPP or another MySQL Server installation.

Example configuration:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=airline_db
DB_USERNAME=root
DB_PASSWORD=
```

> Update the database port and credentials according to the actual local environment.

Check the database connection:

```bash
php artisan migrate:status
```

---

### 1.8. AWS CLI

AWS CLI is used to manage AWS resources from the terminal.

Check:

```bash
aws --version
```

Configure AWS CLI:

```bash
aws configure
```

Enter:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

Example:

```text
Default region name: ap-southeast-1
Default output format: json
```

> Never publish Access Keys or Secret Access Keys in GitHub, screenshots, Markdown files, or project `.env` files.

Verify the configured identity:

```bash
aws sts get-caller-identity
```

**Code snippet:**

```bash
aws --version
aws configure
aws sts get-caller-identity
```
---

### 1.9. Git

Git is used to manage the SkyLink source code.

Check:

```bash
git --version
```

Clone the project:

```bash
git clone https://github.com/quangg199/SkyLink-Airline-System.git
```

Then:

```bash
cd SkyLink-Airline-System
git status
```

---

### 1.10. IAM

The deployment account requires access to the AWS services used in this workshop.

IAM is used for:

* AWS CLI authentication.
* EC2 IAM Roles.
* Lambda permissions.
* S3 access.
* CloudWatch logging.
* SQS and SNS access.

The workshop follows:

> **Grant only the permissions required by each component.**

The AWS Root account should not be used for normal CLI operations.

---

## 2. Detailed Preparation Steps

### Step 1 – Clone the Source Code

Clone SkyLink:

```bash
git clone https://github.com/quangg199/SkyLink-Airline-System.git
```

Enter the project:

```bash
cd SkyLink-Airline-System
```

Check:

```bash
git status
```

Expected result:

```text
On branch ...
nothing to commit, working tree clean
```

---

### Step 2 – Verify the Frontend

Navigate to the frontend:

```bash
cd airline-frontend
```

Install dependencies:

```bash
npm install
```

Build:

```bash
npm run dev
```

Expected structure:

```text
airline-frontend/
└── dist/
    ├── assets/
    ├── index.html
    └── ...
```

The `dist` directory will later be uploaded to Amazon S3.

---

### Step 3 – Verify the Backend

Navigate to the backend:

```bash
cd ../airline-backend
```

Install dependencies:

```bash
composer install
```

Check Laravel:

```bash
php artisan --version
```

If `.env.example` exists, create `.env`:

```bash
cp .env.example .env
```

On Windows CMD:

```cmd
copy .env.example .env
```

Generate the application key:

```bash
php artisan key:generate
```

---

### Step 4 – Configure the Local Database

Update `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=airline_db
DB_USERNAME=root
DB_PASSWORD=
```

Check:

```bash
php artisan migrate:status
```

---

### Step 5 – Run the Backend Locally

Start Laravel:

```bash
php artisan serve
```

The backend will normally be available at:

```text
http://127.0.0.1:8000
```

**Screenshot:**

```text
static/images/workshop/5-2/backend-running.png
```

![Laravel Backend Running](/images/5-Workshop/5.2-Prerequisite/backend-running.png?v=2)

---

### Step 6 – Run the Frontend Locally

Open a new terminal.

Navigate to the frontend:

```bash
cd airline-frontend
```

Run:

```bash
npm run dev
```

Vite will provide a local URL such as:

```text
http://localhost:5173
```

**Screenshot:**

```text
static/images/workshop/5-2/frontend-running.png
```

![SkyLink Frontend Running](/images/5-Workshop/5.2-Prerequisite/frontend-build.png?v=2)

---

### Step 7 – Verify Frontend-to-Backend Communication

The frontend must be configured to call the correct Backend API.

Example:

```env
VITE_API_URL=http://127.0.0.1:8000
```

> Use the actual environment variable name defined by the SkyLink source code.

Test a basic feature such as:

```text
Login
Search Flight
View Flight
```

Expected flow:

```text
Frontend
    |
    v
Backend API
    |
    v
MySQL
```

---

### Step 8 – Verify AWS CLI

Run:

```bash
aws sts get-caller-identity
```

The command should return information about the configured AWS account.

Do not expose sensitive account information in screenshots published on a public workshop website.

---

### Step 9 – Configure the AWS Region

Set the Region:

```bash
aws configure set region ap-southeast-1
```

Check:

```bash
aws configure get region
```

Expected:

```text
ap-southeast-1
```

---

### Step 10 – Check AWS Resources

Open AWS Console and verify the selected Region.

The following services will be used:

```text
IAM
VPC
EC2
S3
CloudFront
RDS
Elastic Load Balancing
SQS
Lambda
WAF
CloudWatch
SNS
```

**Screenshot:**

```text
static/images/workshop/5-2/aws-console-services.png
```

![AWS Services](../../images/workshop/5-2/aws-console-services.png)

---

### Step 11 – Prepare Configuration Files

Important files include:

```text
SkyLink-Airline-System/
│
├── airline-frontend/
│   ├── package.json
│   ├── vite.config.*
│   └── .env
│
└── airline-backend/
    ├── composer.json
    ├── artisan
    ├── .env
    └── database/
```

### Security Notes

Do not commit:

```text
.env
*.pem
AWS Access Key
AWS Secret Access Key
Database Password
JWT Secret
Application Secret
```

Example `.gitignore`:

```gitignore
.env
.env.*
!.env.example

*.pem
*.key
```

---

## 3. Expected Results

After completing this section, the environment should satisfy the following conditions.

### 3.1. Source Code

The SkyLink project has been cloned and verified:

```text
SkyLink-Airline-System
        |
        +---- airline-frontend
        |
        +---- airline-backend
```

### 3.2. Frontend

The frontend can successfully execute:

```text
npm install
       ↓
npm run build
       ↓
dist/
```

and run locally.

### 3.3. Backend

The Laravel backend can:

```text
composer install
       ↓
.env configured
       ↓
Database connected
       ↓
php artisan serve
```

### 3.4. Database

MySQL is running and the backend can connect to `airline_db`.

### 3.5. AWS CLI

AWS CLI works correctly:

```bash
aws sts get-caller-identity
```

and returns the expected AWS account identity.

### 3.6. AWS Environment

The deployment Region has been selected:

```text
ap-southeast-1
```

The required AWS services have been identified and are ready for deployment.

### 3.7. Security

No AWS credentials or sensitive secrets are hard-coded into the source code.

The environment is now ready for the next stage:

```text
Local Environment
       ↓
Source Code Ready
       ↓
AWS CLI Ready
       ↓
AWS Account Ready
       ↓
      5.3
AWS Infrastructure Deployment
```
