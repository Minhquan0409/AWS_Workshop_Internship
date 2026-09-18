---
title: "Configure CloudWatch Alarm"
date: 2026-08-24
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

### Objectives

Provision and configure an Amazon CloudWatch Alarm to continuously monitor incoming traffic requests. When the request volume exceeds specified safety thresholds, the CloudWatch Alarm will automatically transition to the `ALARM` state and trigger a notification to an Amazon SNS Topic to initiate the automated incident response pipeline.

---

## 1. Overview

In this automated security model, **Amazon CloudWatch Alarm** serves as the primary detection mechanism. It aggregates metric data from AWS WAF or CloudWatch Logs to evaluate real-time network traffic patterns.

The CloudWatch Alarm operational workflow includes:

- **Metric Monitoring:** Extracts log data from WAF Access Logs or the CloudWatch Log Group `aws-waf-logs-cloudfront` to measure total request volume within a given timeframe (e.g., 1-minute or 5-minute intervals).
- **Alarm Triggering:** When the request count crosses the configured threshold (e.g., > 100 requests per 5 minutes), the alarm instantly shifts to the `ALARM` state.
- **Notification Dispatch (SNS Notification):** The CloudWatch Alarm automatically sends an alert payload to the SNS Topic `WAFAlertTopic`, which simultaneously dispatches email notifications to administrators and triggers the AWS Lambda function `WAFAutoBlockFunction` to execute automated IP blocking.

---

## 2. Deployment Workflow

The configuration process for CloudWatch Alarm is divided into two detailed practical modules:

- **Section 5.9.1:** Create a CloudWatch Log Metric Filter to extract request counts from the WAF Log Group.
- **Section 5.9.2:** Create the CloudWatch Alarm (`WAFHighRequestRateAlarm`), define threshold settings, and associate notification actions with the SNS Topic `WAFAlertTopic`.

---

## 3. Practical Exercises

Complete the following practical exercises in sequence:

- **[5.9.1. Create CloudWatch Log Metric Filter](5.9.1-Create-CloudWatch-Log-Metric-Filter)**
- **[5.9.2. Create and Configure CloudWatch Alarm](5.9.2-Create-and-Configure-CloudWatch-Alarm)**

---

## 4. Expected Outcomes

Upon completing this chapter, you will achieve the following:

- A **CloudWatch Metric Filter** successfully established on the `aws-waf-logs-cloudfront` Log Group.
- A **CloudWatch Alarm** named `WAFHighRequestRateAlarm` created in the `us-east-1` Region.
- A fully prepared detection mechanism capable of identifying HTTP Flood/DDoS attacks and dispatching signals to trigger the automated incident response chain.
