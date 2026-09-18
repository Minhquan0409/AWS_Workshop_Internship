---

title: "Workshop Overview"
date: 2026-08-24
weight: 1
chapter: false
pre: "<b>5.1. </b>"
-------------------

## Objective

This workshop aims to deploy the **SkyLink Airline** system on **Amazon Web Services (AWS)** while building a Cloud architecture that provides scalability, security, and monitoring capabilities.

The workshop moves the application from a local development environment to an AWS-based architecture using services such as **Amazon S3, Amazon CloudFront, Amazon EC2, Amazon RDS, Application Load Balancer, Amazon SQS, AWS Lambda, AWS WAF, Amazon CloudWatch, Amazon SNS, and AWS IAM**.

After completing the workshop, participants will be able to deploy SkyLink from scratch, validate the application, monitor logs and metrics, test security mechanisms, and clean up AWS resources.

---

## 1. Problem and Solution Overview

### 1.1. Problem

**SkyLink Airline** is an airline management and booking system designed to allow passengers to search for flights and perform flight-booking-related operations.

The system consists of several main components:

* **Frontend:** built with React and Vite, providing the user interface.
* **Backend:** built with Laravel and providing REST APIs and business logic.
* **Database:** MySQL for storing users, flights, bookings, and business data.
* **Queue:** Laravel Queue for background processing.

When running locally, the system depends on a development machine and does not take advantage of Cloud capabilities such as content distribution, scalability, monitoring, and security.

### 1.2. Solution

The workshop deploys SkyLink using the following Cloud architecture:

```text
Frontend
React + Vite
      |
      v
Amazon S3
      |
      v
Amazon CloudFront
      |
     WAF

Backend
Laravel
      |
      v
Application Load Balancer
      |
      v
Amazon EC2
      |
      v
Amazon RDS for MySQL
```

Additional services are introduced for asynchronous processing, security, and monitoring:

```text
Amazon SQS
     |
     v
AWS Lambda
     |
     +------> AWS WAF
     |
     +------> Amazon SNS

Amazon CloudWatch
     |
     +------> Logs
     +------> Metrics
     +------> Alarms
```

This architecture allows SkyLink to:

* Distribute the frontend through a CDN.
* Separate the frontend and backend.
* Use a managed database service.
* Process background tasks using an event-driven approach.
* Monitor system activity.
* Detect and notify administrators about abnormal events.
* Apply security and least-privilege principles.

---

## 2. System Architecture

The overall SkyLink AWS architecture is shown below:

![SkyLink Airline AWS Architecture](/images/5-Workshop/5.1-Workshop-overview/system_architecture.png?v=2)

### 2.1. Architecture Diagram

```text
                              INTERNET
                                  |
                 +----------------+----------------+
                 |                                 |
                 v                                 v
        +----------------+                 +----------------+
        |  CloudFront   |                 |      ALB       |
        +-------+--------+                 +-------+--------+
                |                                  |
                v                                  v
        +----------------+                 +----------------+
        |      WAF       |                 |      EC2       |
        +-------+--------+                 | Laravel API    |
                |                          +-------+--------+
                v                                  |
        +----------------+                          |
        |       S3       |                          v
        | React / Vite   |                  +---------------+
        +----------------+                  | RDS MySQL     |
                                            +---------------+
                                                    |
                                             +------+------+
                                             |             |
                                             v             v
                                           SQS           Database
                                             |
                                             v
                                          Lambda
                                             |
                              +--------------+--------------+
                              |                             |
                              v                             v
                           WAF IP Set                     SNS
                                                            |
                                                            v
                                                         Email


                     +--------------------------------+
                     |          CloudWatch             |
                     | Logs / Metrics / Alarms         |
                     +--------------------------------+
```

### 2.2. Frontend Flow

Users access SkyLink through the Internet.

Requests are delivered to **Amazon CloudFront**, where **AWS WAF** evaluates the request before CloudFront retrieves static assets from **Amazon S3**.

```text
User
  |
  v
CloudFront
  |
  v
AWS WAF
  |
  v
Amazon S3
  |
  v
React / Vite
```

### 2.3. Backend Flow

Frontend API requests are sent to the backend:

```text
React Frontend
      |
      v
Application Load Balancer
      |
      v
Amazon EC2
      |
      v
Laravel REST API
      |
      v
Amazon RDS
```

The Laravel backend handles SkyLink business operations such as:

* Authentication.
* Flight search.
* Booking.
* Ticket management.
* User management.
* Check-in.
* Other airline-related operations.

### 2.4. Asynchronous Processing

Background tasks can be placed into **Amazon SQS**:

```text
Laravel
   |
   v
Amazon SQS
   |
   v
AWS Lambda
```

This approach reduces the processing time of synchronous HTTP requests and provides a foundation for an event-driven architecture.

### 2.5. Monitoring and Security Flow

CloudWatch collects logs and metrics from the system.

```text
Application / WAF / Lambda
             |
             v
        CloudWatch
             |
             v
           Alarm
             |
             v
            SNS
             |
             v
           Email
```

AWS WAF controls abnormal HTTP requests.

When a security event requires automated processing, Lambda can update the WAF IP Set and send a notification through SNS.

---

## 3. System Workflow

The SkyLink AWS workflow consists of the following steps.

### Step 1 – User Accesses SkyLink

The user opens the SkyLink website through the domain or CloudFront URL.

```text
User
 ↓
CloudFront
```

### Step 2 – CloudFront and WAF Process the Request

CloudFront receives the request.

AWS WAF evaluates the request against the configured rules.

```text
CloudFront
     |
     v
AWS WAF
     |
     +---- Block → Request rejected
     |
     +---- Allow → S3
```

### Step 3 – Frontend is Loaded from S3

S3 serves React/Vite static assets:

```text
HTML
CSS
JavaScript
Images
Other static assets
```

### Step 4 – Frontend Calls the Backend API

When the user searches for a flight or creates a booking, the frontend sends an API request:

```text
React
  |
  v
ALB
  |
  v
EC2
  |
  v
Laravel API
```

### Step 5 – Backend Processes the Request

Laravel processes SkyLink business logic.

For example, a flight search follows:

```text
User
 ↓
Search Flight
 ↓
React
 ↓
Laravel API
 ↓
RDS MySQL
 ↓
Flight Data
 ↓
Laravel API
 ↓
React
 ↓
User
```

### Step 6 – Data is Stored in RDS

Business data is stored in Amazon RDS for MySQL.

Examples include:

* Users.
* Flights.
* Bookings.
* Tickets.
* Payment information.
* Check-in information.

### Step 7 – Background Processing

Asynchronous tasks can follow:

```text
Laravel
   |
   v
SQS
   |
   v
Lambda
```

This separates background processing from the main HTTP request.

### Step 8 – Monitoring

CloudWatch collects:

```text
Logs
Metrics
Errors
CPU Utilization
Lambda Invocations
WAF Events
```

### Step 9 – Alerting

When a configured threshold is exceeded:

```text
CloudWatch Alarm
       |
       v
      SNS
       |
       v
     Email
```

The administrator receives an alert and can investigate the system.

---

## 4. AWS Services Used

| Service                       | Role in SkyLink                         |
| ----------------------------- | --------------------------------------- |
| **Amazon S3**                 | Hosts React/Vite frontend               |
| **Amazon CloudFront**         | CDN and frontend distribution           |
| **AWS WAF**                   | Protects against abnormal HTTP requests |
| **Amazon EC2**                | Runs Laravel Backend                    |
| **Application Load Balancer** | Distributes backend traffic             |
| **Amazon RDS**                | Managed MySQL database                  |
| **Amazon SQS**                | Asynchronous message queue              |
| **AWS Lambda**                | Event processing and automation         |
| **Amazon CloudWatch**         | Logs, metrics, and monitoring           |
| **Amazon SNS**                | Monitoring and security notifications   |
| **AWS IAM**                   | Identity and access management          |
| **Amazon VPC**                | Private networking for AWS resources    |

### Service Selection Rationale

**Amazon S3:** suitable for React/Vite because the production build consists of static files.

**CloudFront:** distributes frontend content through a CDN and provides HTTPS.

**EC2:** provides control over the Laravel Backend runtime environment.

**RDS:** provides a managed MySQL database and reduces database administration effort.

**ALB:** provides a stable backend endpoint and prepares the architecture for multiple EC2 instances.

**SQS:** decouples background processing from the main HTTP request.

**Lambda:** processes events without requiring a dedicated server.

**WAF:** adds a security layer for HTTP/HTTPS requests.

**CloudWatch:** provides monitoring and troubleshooting capabilities.

**SNS:** sends notifications when important events occur.

**IAM:** applies least-privilege access and avoids hard-coded AWS credentials.

**VPC:** controls network access between AWS resources.

---

## 5. Results

After completing the workshop, SkyLink is deployed using a Cloud-based architecture instead of running only in a local environment.

### 5.1. Application Deployment

The React/Vite frontend is built and deployed to Amazon S3.

The Laravel backend is deployed on Amazon EC2.

The database is migrated to Amazon RDS for MySQL.

### 5.2. Cloud Architecture

The system contains clearly separated components:

```text
CloudFront
    ↓
WAF
    ↓
S3

ALB
    ↓
EC2
    ↓
RDS
```

### 5.3. Security

The workshop applies:

* IAM Roles.
* Least Privilege.
* AWS WAF.
* WAF IP Sets.
* HTTPS.
* No hard-coded AWS Access Keys.
* Restricted S3 public access.

### 5.4. Monitoring

CloudWatch monitors:

* Application logs.
* WAF logs.
* EC2 metrics.
* Lambda metrics.
* Error events.
* Alarm status.

### 5.5. Alerting

When an event exceeds the configured threshold:

```text
CloudWatch
     ↓
Alarm
     ↓
SNS
     ↓
Email
```

the administrator can receive an alert.

### 5.6. Scalability

The architecture can be extended in the future:

```text
                 ALB
                  |
          +-------+-------+
          |       |       |
         EC2     EC2     EC2
```

The backend can scale horizontally by adding additional EC2 instances.

SQS also reduces coupling between components and supports asynchronous processing.

### 5.7. Operations

The workshop provides a complete operational workflow:

```text
Deploy
  ↓
Test
  ↓
Monitor
  ↓
Alert
  ↓
Troubleshoot
  ↓
Cleanup
```

Therefore, SkyLink demonstrates not only application deployment on AWS but also important Cloud Computing concepts including **deployment, security, monitoring, scalability, and cost management**.
