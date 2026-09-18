---
title: "Proposal"
date: 2026-09-10
weight: 2
chapter: false
pre: "<b> 2. </b>"
---

During my learning and practice in the First Cloud AI Journey (FCAJ) program, I realized that combining AWS knowledge with a real-world project makes learning more intuitive and effective. Therefore, I propose building a Workshop to deploy the **SkyLink Airline** system on the AWS platform.

SkyLink Airline is a web system supporting flight booking-related functions. In this Workshop, I focus on deploying and protecting the Frontend of the system on AWS, while also building monitoring and alerting mechanisms when abnormal requests are detected.

## 2.1. Proposal objectives

The proposed Workshop has the following main objectives:

- Apply the AWS knowledge learned to a real web project.
- Deploy the SkyLink Airline Frontend on Amazon S3.
- Use Amazon CloudFront to distribute content and support HTTPS access.
- Use Origin Access Control (OAC) to control access between CloudFront and Amazon S3.
- Use AWS WAF to protect the application from unwanted requests.
- Use IP Sets and Rate-based Rules to control abnormal traffic sources.
- Use Amazon CloudWatch to collect logs, create metrics, and configure alerts.
- Use Amazon SNS to send email notifications when important events are detected.
- Apply IAM according to the principle of least privilege.
- Help learners understand how AWS services work together in a real-world web architecture.

## 2.2. Problem to be solved

When deploying a web application on the Internet, the system may have to handle many different types of requests. Some requests may come from unwanted sources or have unusually high access frequency, affecting resources and system availability.

If the Frontend is only deployed on a storage service without a protection and monitoring layer, detecting and handling abnormal requests becomes difficult.

Therefore, this Workshop focuses on solving three main issues:

1. **Application deployment:** Put the SkyLink Airline Frontend into the AWS environment and provide stable access through CloudFront.
2. **Security:** Use AWS WAF to inspect requests and limit unwanted traffic sources.
3. **Monitoring and alerts:** Collect logs, create metrics, and use CloudWatch Alarm in combination with SNS to detect and notify abnormal events.

## 2.3. Proposed solution

The proposed architecture uses AWS services in the following model:

```text
                         Internet
                            │
                            ▼
                       CloudFront
                            │
                            ▼
                         AWS WAF
                       ┌────┴────┐
                       │         │
                    Allow       Block
                       │         │
                       ▼         ▼
                    Amazon S3   WAF Logs
                                   │
                                   ▼
                           CloudWatch Logs
                                   │
                                   ▼
                            Metric Filter
                                   │
                                   ▼
                           CloudWatch Alarm
                                   │
                                   ▼
                              SNS Topic
                                   │
                                   ▼
                              Email Alert
```

In this architecture:

* Amazon S3: Stores Frontend files after build.
* Amazon CloudFront: Distributes content from S3 to users and supports HTTPS.
* AWS WAF: Inspects and controls requests before they are forwarded to the Origin.
* CloudWatch Logs: Stores WAF logs for monitoring and analysis.
* CloudWatch Metric Filter: Converts matching events in the logs into metrics.
* CloudWatch Alarm: Monitors metrics and detects when the number of blocked requests exceeds the configured threshold.
* Amazon SNS: Sends alert notifications to subscribers.
* Email: Receives notifications when the system detects events that require attention.

## 2.4. Scope of implementation

Within the scope of the Workshop, I focus on the Frontend and the AWS components used for deployment, security, and monitoring.

The main content includes:

* Preparing and building the SkyLink Airline Frontend.
* Creating and configuring Amazon S3.
* Deploying the Frontend to S3.
* Configuring Amazon CloudFront.
* Configuring Origin Access Control.
* Creating and configuring the AWS WAF Web ACL.
* Configuring IP Set and Rate-based Rule.
* Setting up WAF Logging.
* Creating CloudWatch Log Metric Filter.
* Configuring CloudWatch Alarm.
* Creating Amazon SNS Topic and Email Subscription.
* Testing access, protection, and alerting capabilities of the system.
* Cleaning up AWS resources after completing the Workshop.

## 2.5. Expected results

After completing the Workshop, learners will be able to:

* Successfully deploy a web Frontend on AWS.
* Understand how CloudFront distributes content from S3.
* Understand how to use OAC to protect the S3 Origin.
* Know how to use AWS WAF to control requests.
* Know how to monitor WAF behavior through CloudWatch Logs.
* Know how to create Metric Filters and CloudWatch Alarms.
* Be able to configure SNS to receive alerts by email.
* Understand how to build a basic security and monitoring workflow for a web application on AWS.

Through this proposal, the SkyLink Airline Workshop aims to combine theoretical knowledge with practical AWS experience while creating a foundation for continued expansion toward Cloud, DevOps, and Cloud Security.