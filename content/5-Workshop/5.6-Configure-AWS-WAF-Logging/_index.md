---
title: "Configure AWS WAF & Logging"
date: 2026-08-24
weight: 6
chapter: false
pre: "<b>5.6. </b>"
---

## Objectives

In this section, we will deploy **AWS WAF (Web Application Firewall)** to protect the **SkyLink Airline** website distributed through Amazon CloudFront.

In addition, we will configure **AWS WAF Logging** to record requests passing through the Web ACL, helping with monitoring, inspection, and detection of blocked requests.

After completion, the system architecture will be expanded as follows:

```text
User
  │
  │ HTTPS
  ▼
Amazon CloudFront
  │
  │ AWS WAF
  ▼
Web ACL
  │
  │ Allowed requests
  ▼
Amazon S3
  │
  └── React Frontend
```

### Main objectives
* Create a Web ACL in AWS WAF.
* Associate the Web ACL with the SkyLink Airline CloudFront Distribution.
* Use AWS Managed Rules to strengthen website protection.
* Check whether AWS WAF allows or blocks requests.
* Configure logging to record AWS WAF activity.
* Review logs and recorded requests.
* Understand how CloudFront works together with AWS WAF to protect the website.
* Clean up unnecessary resources to minimize costs.

---

## 1. Overview
### 1.1. Role of AWS WAF

AWS WAF (Web Application Firewall) is an AWS application-layer firewall service that helps control HTTP/HTTPS requests sent to supported resources such as Amazon CloudFront.

In the SkyLink Airline project, AWS WAF is placed in front of CloudFront to inspect requests before forwarding them to S3.

Architecture:
```
                    Internet
                        │
                        ▼
                ┌───────────────┐
                │     User      │
                └───────┬───────┘
                        │ HTTPS
                        ▼
                ┌───────────────┐
                │  CloudFront   │
                │      CDN      │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │    AWS WAF    │
                │    Web ACL    │
                └───────┬───────┘
                        │
                 Allowed Request
                        │
                        ▼
                ┌───────────────┐
                │      S3       │
                │    Private    │
                └───────┬───────┘
                        │
                        ▼
                React Frontend
```
AWS WAF can inspect requests based on conditions such as:
```
IP Address
HTTP Method
URI
Headers
Query String
Request Rate
Managed Rules
```
In this workshop, AWS WAF is used at a basic level to demonstrate how to protect the CloudFront website.

---

### 1.2. Role of Web ACL

Web ACL (Web Access Control List) is a set of rules used to determine how AWS WAF handles requests.

A Web ACL can contain multiple rules.

Example:
```
Incoming Request
       │
       ▼
    Web ACL
       │
       ├── AWS Managed Rule
       │
       ├── IP Rule
       │
       └── Rate-based Rule
       │
       ▼
   Allow / Block
```
In the SkyLink Airline project, the Web ACL is associated with the CloudFront Distribution.

---

### 1.3. AWS Managed Rules

AWS provides AWS Managed Rules to help protect applications against common groups of malicious requests.

In this workshop, we can use: AWSManagedRulesCommonRuleSet

This rule set provides common rules to inspect HTTP/HTTPS requests.

Using Managed Rules reduces the number of custom rules that must be created.

---

## 2. Lab Content
### 2.1 Create WAF IP Set
### 2.1.1. Initialize WAF IP Set

### Step 1: Access the AWS WAF service

1. Sign in to the **AWS Management Console**.
2. In the search bar, enter `WAF` and select the **AWS WAF & Shield** service.
3. In the left navigation menu, select **IP sets**.

![Access WAF IP Sets](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/waf-ip-sets-menu.png)

---

### Step 2: Configure IP Set parameters

1. In the **Region** section, select **Global (CloudFront)**.
2. Click the **Create IP set** button.
3. Enter the detailed information:
   - **IP set name:** `AutoBlockedIPSetV6`
   - **Description:** `IP set containing the list of IPs automatically blocked by Lambda`
   - **Region:** Keep default `Global (CloudFront)`
   - **IP version:** Select **IPv6**.
   - **IP addresses:** Leave empty (do not enter any IP) because this list will be updated automatically by AWS Lambda when violations are detected.

![Configure IP Set Name and Region](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/configure-ip-set.png)

---

### Step 3: Complete creating the IP Set

1. Scroll to the bottom of the page and click **Create IP set**.

![Click Create IP Set](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/finish-create-ip-set.png)

---

### 2.1.2. Check the WAF IP Set list

After successful creation, the IP sets list will display `AutoBlockedIPSetV6` with the following details:

- **Region:** Global (CloudFront)
- **IP version:** IPv6
- **Capacity:** 1

---

### 2.1.3. Expected result

After completing this lab:

- The **WAF IP Set** named `AutoBlockedIPSetV6` is successfully created in the `Global (CloudFront)` scope.
- The initial IP list is left empty successfully, ready to be assigned to the Web ACL in section 2.2 and for the Lambda function to overwrite/add offending IPs in Workshop 5.8.

---

### 2.2. Create and Configure Web ACL

This section provides detailed steps for creating a new Web Access Control List (Web ACL) in AWS WAF, configuring protection rules (Rate-based rule and IP Set rule), and directly associating it with the Amazon CloudFront Distribution to protect the system.

---

### 2.2.1. Initialize Web ACL

### Step 1: Access the Web ACL creation page

1. Sign in to the **AWS Management Console**.
2. Go to the **AWS WAF & Shield** service.
3. In the left navigation menu, select **Web ACLs**.
4. In the **Region** section, select **Global (CloudFront)**.
5. Click the **Create web ACL** button.

![Access Web ACLs](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/web-acl-menu.png)

---

### Step 2: Configure general information (Describe web ACL)

1. **Name:** `skylink-airline-waf`
2. **Description:** `AWS WAF protection for the SkyLink Airline CloudFront distribution.`
3. **Resource type:** Keep the default **CloudFront distributions**.
4. **Associated AWS resources:**
   - Click the **Add AWS resources** button.
   - Select **Amazon CloudFront distributions**.
   - Tick the CloudFront Distribution name created in the previous lab (5.5).
   - Click **Add**.
5. Click **Next**.

![Configure Web ACL Information](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/describe-web-acl.png)
![Configure Web ACL Information](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/describe-web-acl2.png)

---

### 2.2.2. Configure Rules and Actions

### Step 1: Add IP Set Rule (Block offending IPs)

1. At the **Add rules and rule groups** step, click **Add rules** -> select **Add my own rules and rule groups**.
2. Configure the IP block rule:
   - **Rule type:** Select **IP set**.
   - **Name:** `BlockAutoIPSetRule`
   - **IP set:** Select the IP Set `AutoBlockedIPSetV6` created in section 2.1.
   - **Source IP location:** Select **Source IP address**.
   - **Action:** Select **Block**.
3. Click **Add rule**.

![Configure IP Set Rule](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/add-ip-set-rule.png)

---

### Step 2: Add a Rate-based Rule (Limit request volume)

1. Continue by clicking **Add rules** -> select **Add my own rules and rule groups**.
2. Configure the Rate limit rule:
   - **Rule type:** Select **Rate-based rule**.
   - **Name:** `HTTPRateLimitRule`
   - **Rate limit:** Enter a threshold, for example: `100` (or `100` - `2000` depending on the lab requirement).
   - **Evaluation window:** Select **5 minutes** (or the default time).
   - **Criteria to aggregate requests:** Select **IP address** -> **Source IP address**.
   - **Action:** Select **Block**.
3. Click **Add rule**.

![Configure Rate-based Rule](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/add-rate-rule.png)

---

### Step 3: Configure the Default Web ACL Action

1. In the **Default action** section, select **Allow** (allow all normal requests that do not violate the rules).
2. Click **Next**.

---

### 2.2.3. Complete Web ACL Installation

### Step 1: Set rule priority

1. Keep the rule order unchanged (ensure `BlockAutoIPSetRule` is placed above or evaluated appropriately compared to `HTTPRateLimitRule`).
2. Click **Next**.

---

### Step 2: Review and create the Web ACL

1. Review all configured settings.
2. Click **Create web ACL** at the end of the page.

![Complete Web ACL Creation](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/finish-web-acl.png)

---

### 2.2.4. Expected result

After completing this lab:

- The **Web ACL** named `skylink-airline-waf` is successfully created in the `Global (CloudFront)` scope.
- The **IP Set Rule** (`BlockAutoIPSetRule`) is successfully integrated with `AutoBlockedIPSetV6`.
- The **Rate-based Rule** (`HTTPRateLimitRule`) is successfully integrated to monitor and limit request volume from a single IP.
- The Web ACL is successfully attached to the **CloudFront Distribution**, ready to filter and process network traffic at the Edge layer.

---

### 2.3 Configure WAF Access Logging to CloudWatch Logs

This section walks through the detailed steps to create an Amazon CloudWatch Log Group with the required naming convention and enable access logging on AWS WAF so it automatically forwards all access logs to CloudWatch.

---

### 2.3.1. Create a CloudWatch Log Group

### Step 1: Access the CloudWatch Logs service

1. Sign in to the **AWS Management Console**.
2. In the search bar, enter `CloudWatch` and select the **CloudWatch** service.
3. In the left navigation menu, open **Logs** and select **Log groups**.
4. Ensure the current Region is **US East (N. Virginia) us-east-1** (because WAF for CloudFront must deliver logs to this Region).

![Access CloudWatch Log Groups](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/cloudwatch-log-groups-menu.png)

---

### Step 2: Create the Log Group using the required AWS WAF naming convention

1. Click the **Create log group** button.
2. **Log group name:** Enter the required AWS WAF prefix exactly: `aws-waf-logs-cloudfront` (or `aws-waf-logs-website-protection`).
   > **Important note:** The AWS WAF Log Group name must begin with `aws-waf-logs-` or the WAF Console will not recognize it and will not allow the association.
3. **Retention setting:** Choose the log retention period (for example: **1 day** or **7 days** to save on costs in the lab).
4. Click **Create**.

![Create CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/create-log-group.png)
![Create CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/create-log-group2.png)

---

### 2.3.2. Enable WAF Logging on the Web ACL

### Step 1: Open the AWS WAF Logging configuration page

1. Return to the **AWS WAF & Shield** service.
2. Select **Web ACLs** in the left menu -> choose **Global (CloudFront)**.
3. Select the Web ACL `skylink-airline-waf` created in section 2.2.
4. Switch to the **Logging and metrics** tab.
5. In the **Logging** section, click the **Enable** button.

![Enable Logging on the Web ACL](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/enable-waf-logging.png)

---

### Step 2: Link it to the CloudWatch Log Group

1. **Logging destination:** Select **CloudWatch Logs log group**.
2. **CloudWatch Logs log group:** Select the correct log group `aws-waf-logs-cloudfront` created earlier.
3. **Redacted fields (Optional):** Keep the default settings (do not hide any fields) or select sensitive fields to redact if needed.
4. **Filter logs (Optional):** Keep the default setting to record all traffic (All traffic).
5. Click **Save**.

![Link WAF to CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/link-log-destination.png)
![Link WAF to CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/link-log-destination2.png)

---

### 2.3.3. Check the recorded log data

### Step 1: Generate traffic

1. Access the CloudFront Distribution domain name (obtained in the previous lab 5.5) in the browser or send several requests to the website to generate real traffic.

---

### Step 2: Check the Log Streams in CloudWatch

1. Return to the **CloudWatch** service -> **Log groups** -> select `aws-waf-logs-cloudfront`.
2. In the **Log streams** tab, check whether new log streams appear containing AWS WAF JSON log data.

![Check WAF Log Streams](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/verify-log-streams.png)

---

### 2.3.4. Expected result

After completing this lab:

- The **CloudWatch Log Group** named `aws-waf-logs-cloudfront` is successfully created in the `us-east-1` Region.
- **WAF Access Logging** is successfully enabled on the `skylink-airline-waf` Web ACL.
- All HTTP/HTTPS traffic sent to CloudFront is recorded in detail as JSON Log Streams in CloudWatch, making it ready as a data source for CloudWatch alarms and Lambda automation in labs 5.7 and 5.8.
