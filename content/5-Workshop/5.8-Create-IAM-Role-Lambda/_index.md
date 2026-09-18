---
title: "Create IAM Role & AWS Lambda Function"
date: 2026-08-24
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### Objectives

Provision an IAM Role granting Least Privilege permissions and deploy an automated incident response AWS Lambda function. The Lambda function will parse WAF access log data from CloudWatch Logs to extract malicious IP addresses and automatically append them to the WAF IP Set, proactively mitigating DDoS/HTTP Flood attacks.

---

## 1. Overview

In this automated security response architecture, **AWS Lambda** serves as the central engine executing incident response workflows without requiring human intervention.

When the system detects traffic exceeding pre-configured thresholds, the response process executes through the following components:

- **Access Management (IAM Role):** The Lambda function must be assigned an **IAM Execution Role** attached to a fine-tuned IAM Policy. This role grants explicit permissions to access **CloudWatch Logs** (to read WAF log entries) and **AWS WAF** (to invoke operations appending offending IP addresses to `AutoBlockedIPSetV6`).
- **Automated Processing (Lambda Function):** A Python function extracts access log data from CloudWatch Logs, identifies IP addresses originating abnormal request volumes, and automatically invokes the AWS WAF `UpdateIPSet` API to place those IPs onto the edge blocking list.

---

## 2. Deployment Workflow

The configuration process for the IAM Role and AWS Lambda Function is divided into two separate practical modules:

- **Section 5.8.1:** Create an IAM Policy and IAM Execution Role (`LambdaWAFAutoBlockRole`) providing the necessary permissions for Lambda.
- **Section 5.8.2:** Create the AWS Lambda function (`WAFAutoBlockFunction`), configure Environment Variables, and deploy the automated Python source code.

---

## 3. Practical Exercises

Complete the following practical exercises in sequence:

- **[5.8.1. Create IAM Role for Lambda](5.8.1-Create-IAM-Role-for-Lambda)**
- **[5.8.2. Create and Deploy AWS Lambda Function](5.8.2-Create-and-Deploy-AWS-Lambda-Function)**

---

## 4. Expected Outcomes

Upon completing this chapter, you will achieve the following:

- An **IAM Execution Role** named `LambdaWAFAutoBlockRole` successfully created adhering to the principle of least privilege.
- An **AWS Lambda Function** named `WAFAutoBlockFunction` deployed with complete Python source code in the `us-east-1` Region.
- A functional Lambda function prepared to receive triggers from Amazon SNS or CloudWatch Alarms to automatically parse log streams and populate offending IPs into the WAF IP Set.
