---
title: "Create CloudWatch Log Metric Filter"
date: 2026-08-24
weight: 1
chapter: false
pre: " <b> 5.9.1. </b> "
---

This article provides detailed instructions on creating a CloudWatch Log Metric Filter using the AWS Management Console. The Metric Filter scans all log data from the AWS WAF Log Group to count request entries and convert them into a Custom Metric, allowing CloudWatch Alarm to monitor traffic in real time.

---

## 1. Create a Log Metric Filter

### Step 1: Access the AWS WAF Log Group

1. Log in to the **AWS Management Console**.
2. Search for and select the **CloudWatch** service.
3. In the left navigation menu, expand **Logs** -> Select **Log groups**.
4. Ensure your active AWS Region is set to **US East (N. Virginia) us-east-1**.
5. Select the `aws-waf-logs-cloudfront` Log Group created in section 5.6.3.

![Select AWS WAF Log Group](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/select-log-group.png)

---

### Step 2: Create a Metric Filter

1. On the `aws-waf-logs-cloudfront` details page, switch to the **Metric filters** tab.
2. Click the **Create metric filter** button.

![Click Create Metric Filter](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/click-create-metric-filter.png)

---

### Step 3: Define Filter Pattern

1. **Filter pattern:** Enter `{ $.httpRequest.clientIP = "*" }` (this pattern matches all WAF log records containing a client IP address).
2. Open the **Test pattern** section and select an active Log Stream to verify that the pattern matches log events accurately.
3. Click **Next**.

![Define Filter Pattern](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/define-filter-pattern.png)

---

### Step 4: Configure Metric Details

1. **Filter name:** Enter `WAFRequestCountFilter`.
2. **Metric namespace:** Enter `WAFCustomMetrics` (or specify a custom namespace).
3. **Metric name:** Enter `WAFRequestCount`.
4. **Metric value:** Enter `1` (each log record matching the pattern counts as 1 request unit).
5. **Default value:** Leave blank or enter `0`.
6. **Unit:** Select **Count**.
7. Click **Next**.

![Configure Metric Details](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/configure-metric-details.png)

---

### Step 5: Review and Complete

1. Review all configuration settings.
2. Click **Create metric filter**.

![Complete Metric Filter Creation](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-metric-filter.png)
![Complete Metric Filter Creation](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-metric-filter1.png)

---

## 2. Verify the Created Custom Metric

1. Return to the **Metric filters** tab inside the `aws-waf-logs-cloudfront` Log Group.
2. Verify that the `WAFRequestCountFilter` filter is successfully displayed.
3. Click on the Custom Metric name `WAFRequestCount` (or navigate to **Metrics** -> **All metrics** -> **WAFCustomMetrics**) to view the real-time request volume metrics graph.

![Verify Custom Metric Display](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/verify-custom-metric.png)

---

## 3. Expected Outcomes

Upon completing this practical exercise:

- A **CloudWatch Log Metric Filter** named `WAFRequestCountFilter` is successfully attached to the `aws-waf-logs-cloudfront` Log Group.
- The Custom Metric `WAFRequestCount` under the `WAFCustomMetrics` Namespace is created, continuously capturing request count data from WAF Log Streams and ready to serve as the evaluation data source for CloudWatch Alarm in section 5.9.2.
