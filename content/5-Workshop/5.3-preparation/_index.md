---

title: "Project Preparation"
date: 2026-08-24
weight: 3
chapter: false
pre: "<b>5.3. </b>"
-------------------

### Objective

Prepare the **SkyLink Airline** source code for deploying the Frontend and Backend to AWS.

This section covers the project structure, dependencies, environment variables, database configuration, and Frontend build.

---

## 1. Check the Project Structure

After cloning the source code, verify the structure:

```text
SkyLink-Airline-System/
├── airline-frontend/
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.*
│
└── airline-backend/
    ├── app/
    ├── routes/
    ├── database/
    ├── public/
    ├── artisan
    ├── composer.json
    └── .env
```

**Source code:**

`https://github.com/quangg199/SkyLink-Airline-System`

---

## 2. Prepare the Frontend

### Step 1: Install Dependencies

Navigate to the Frontend directory:

```bash
cd airline-frontend
npm install
```

### Step 2: Configure the API

Create or update `.env`:

```env
VITE_API_URL=http://127.0.0.1:8000
```

> Adjust the environment variable according to the actual SkyLink source-code configuration.

### Step 3: Build the Frontend

Run:

```bash
npm run build
```

After a successful build:

```text
airline-frontend/
└── dist/
    ├── assets/
    └── index.html
```

The `dist/` directory will later be deployed to **Amazon S3**.

**Code snippet:**

```bash
cd airline-frontend
npm install
npm run build
```
---

## 3. Prepare the Backend

### Step 1: Install Dependencies

Navigate to the Backend:

```bash
cd ../airline-backend
```

Install Composer dependencies:

```bash
composer install
```

### Step 2: Configure the Environment

If `.env` does not exist:

```bash
cp .env.example .env
```

Generate the Laravel application key:

```bash
php artisan key:generate
```

### Step 3: Configure the Database

Update the database configuration:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=airline_db
DB_USERNAME=root
DB_PASSWORD=
```

> These settings are for the local environment. During AWS deployment, `DB_HOST` will be replaced with the Amazon RDS endpoint.

Check the database connection:

```bash
php artisan migrate:status
```

---

## 4. Test the Project Locally

Before deploying to AWS, verify that the application works correctly in the local environment.

### Run the Backend

```bash
php artisan serve
```

Backend:

```text
http://127.0.0.1:8000
```

### Run the Frontend

Open a new terminal:

```bash
cd airline-frontend
npm run dev
```

Frontend:

```text
http://localhost:5173
```

### Test the Main Functions

Perform several basic operations:

```text
Login
   ↓
Search Flight
   ↓
View Flight
   ↓
Booking
```

---

## 5. Prepare for AWS Deployment

After successful local testing, prepare the deployment components:

```text
Frontend
   ↓
npm run build
   ↓
dist/
   ↓
Amazon S3
```

```text
Backend
   ↓
Laravel Application
   ↓
Docker / EC2
   ↓
Application Load Balancer
```

```text
Database
   ↓
MySQL
   ↓
Amazon RDS
```

Optional deployment files can be organized as:

```text
deployment/
├── Dockerfile
├── docker-compose.yml
├── scripts/
└── cloudformation/
```

If CloudFormation templates or deployment scripts are used, they will be provided in the corresponding deployment section.

**Download files:**

```text
static/downloads/workshop/
```

---

## 6. Expected Results

After completing this section:

* The SkyLink source code has been verified.
* React/Vite dependencies are installed successfully.
* The Frontend builds successfully and generates `dist/`.
* Laravel dependencies are installed successfully.
* The `.env` file is configured correctly.
* The Backend can connect to MySQL.
* Frontend and Backend run successfully in the local environment.
* The project is ready for AWS deployment.

```text
SkyLink Source Code
        ↓
Frontend Ready
        ↓
Backend Ready
        ↓
Database Ready
        ↓
Local Test Passed
        ↓
   AWS Deployment
```
