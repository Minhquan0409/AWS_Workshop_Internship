---
title: "Create and Configure CloudWatch Alarm"
date: 2026-08-24
weight: 2
chapter: false
pre: " <b> 5.9.2. </b> "
---

This article provides detailed instructions on creating an Amazon CloudWatch Alarm using the AWS Management Console. The CloudWatch Alarm will continuously monitor the Custom Metric `WAFRequestCount` from the Log Group and automatically send a notification to the SNS Topic `WAFAlertTopic` when request traffic exceeds the configured threshold.

---

## 1. Create a CloudWatch Alarm

### Step 1: Access the CloudWatch Alarms Interface

1. Log in to the **AWS Management Console**.
2. Search for and select the **CloudWatch** service.
3. In the left navigation menu, select **Alarms** -> Select **All alarms**.
4. Ensure your active AWS Region is set to **US East (N. Virginia) us-east-1**.
5. Click the **Create alarm** button.

![Access CloudWatch Alarms](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/access-alarms-menu.png)

---

### Step 2: Select Metric for the Alarm

1. Click the **Select metric** button.
2. Search for and select **WAFCustomMetrics** (the Namespace created in section 5.9.1).
3. Select **Metrics with no dimensions** -> Check the box for the **WAFRequestCount** metric.
4. Click **Select metric**.

![Select Custom Metric WAFRequestCount](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/select-metric.png)

---

### Step 3: Configure Metric and Alarm Conditions

1. Under the **Metric** section:
   - **Statistic:** Select **Sum** (the total aggregated count of requests).
   - **Period:** Select **5 minutes** (the metric evaluation period).
2. Under the **Conditions** section:
   - **Threshold type:** Select **Static**.
   - **Whenever WAFRequestCount is...:** Select **Greater/Equal** (`>=`).
   - **than...:** Enter `100` (Alarm threshold: 100 requests within 5 minutes).
3. Click **Next**.

![Configure Alarm Conditions](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/configure-conditions.png)

---

### Step 4: Configure Actions & Associate SNS Topic

1. **Alarm state trigger:** Select **In alarm** (triggered when the alarm state transitions to ALARM).
2. **Send a notification to the following SNS topic:**
   - Select **Select an existing SNS topic**.
   - **Send a notification to...:** Select the SNS Topic `skylink-waf-alerts` created in section 5.7.1.
3. Click **Next**.

![Associate SNS Topic](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/configure-actions-sns.png)

---

### Step 5: Name and Review Alarm

1. **Alarm name:** Enter `WAFHighRequestRateAlarm`.
2. **Alarm description:** Enter `Canh bao khi luong truy cap WAF vuot qua 100 requests trong 5 phut`.
3. Click **Next**.
4. Review all configuration settings and click **Create alarm**.

![Review and Create Alarm](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-alarm.png)
![Review and Create Alarm](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-alarm1.png)
![Review and Create Alarm](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-alarm2.png)

---

## 2. Verify CloudWatch Alarm Status

1. Return to the **All alarms** list.
2. Verify the status of the `WAFHighRequestRateAlarm`:
   - **State:** Initially, it will show `OK` or `Insufficient data` (it will stabilize to `OK` after a few minutes of data collection).
   - **Conditions:** `>= 100 for 1 datapoints within 5 minutes`.

![Verify Alarm Status](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/verify-alarm-status.png)

---

## 3. Expected Outcomes

Upon completing this practical exercise:

- A **CloudWatch Alarm** named `WAFHighRequestRateAlarm` is successfully created in the `us-east-1` Region.
- The Alarm is directly linked to the Custom Metric `WAFRequestCount` and configured to send notifications to the SNS Topic `skylink-waf-alerts`.
- The entire detection pipeline (Metric Filter -> CloudWatch Alarm -> SNS Topic) is fully prepared to integrate with AWS Lambda for automated IP blocking during testing.
