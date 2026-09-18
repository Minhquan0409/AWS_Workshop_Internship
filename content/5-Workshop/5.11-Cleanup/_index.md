---
title: "Resource Cleanup"
date: 2026-08-24
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Objective

Upon completing the system testing and acceptance phases, cleaning up the provisioned AWS resources is crucial to avoid incurring unwanted charges (Cost Optimization) and maintain account security.

---

## Resource Cleanup Steps

To prevent dependency errors during deletion, complete the steps in the following order:

### Step 1: Delete CloudFront Distribution

1. Navigate to the **AWS CloudFront Console** -> select **Distributions**.
2. Select the distribution created for this project (`https://d2n9euazwledbh.cloudfront.net/`).
3. Click **Disable** and wait for the status to change to _Disabled_.
4. Once _Disabled_, select the distribution and click **Delete**.

![Disable and delete CloudFront Distribution](/images/5-Workshop/5.11-Cleanup/buoc1-delete-cloudfront-distribution.png)
![Disable and delete CloudFront Distribution](/images/5-Workshop/5.11-Cleanup/buoc1-delete-cloudfront-distribution2.png)
![Disable and delete CloudFront Distribution](/images/5-Workshop/5.11-Cleanup/buoc1-delete-cloudfront-distribution3.png)

---

### Step 2: Disassociate Web ACL from WAF & Delete WAF Rules / IP Sets

1. Navigate to **AWS WAF** -> select **Web ACLs** (switch the Region to **Global (CloudFront)**).
2. Select the project's Web ACL -> switch to the **Associated AWS resources** tab -> click **Disassociate** to unlink it from the CloudFront Distribution (if associated).
   ![Disassociate Web ACL and delete WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets.png)
3. Go to the **Rules** tab -> delete the IP blocking rules created earlier.
   ![Disassociate Web ACL and delete WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets2.png)
4. Click **Delete** to delete the Web ACL.
   ![Disassociate Web ACL and delete WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets3.png)
   ![Disassociate Web ACL and delete WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets4.png)
5. Navigate to **IP sets** (Region **Global (CloudFront)**) -> select and delete both IP sets:
   - `AutoBlockedIPSetV6`
   - `AutoBlockedIPSet`

![Disassociate Web ACL and delete WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets5.png)
![Disassociate Web ACL and delete WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets6.png)

---

### Step 3: Delete AWS Lambda Function & IAM Role/Policy

1. Navigate to the **AWS Lambda Console** -> select **Functions**.
2. Select the function `WAFAutoBlockFunction` -> click **Actions** -> select **Delete**.

![Delete Lambda Function WAFAutoBlockFunction](/images/5-Workshop/5.11-Cleanup/buoc3a-delete-lambda-function1.png)
![Delete Lambda Function WAFAutoBlockFunction](/images/5-Workshop/5.11-Cleanup/buoc3a-delete-lambda-function2.png)

3. Navigate to the **IAM Console**:
   - Go to **Roles** -> locate and delete the role `LambdaWAFAutoBlockRole`.
   - Go to **Policies** -> locate and delete the policy `LambdaWAFAutoBlockPolicy`.

![Delete IAM Role and IAM Policy](/images/5-Workshop/5.11-Cleanup/buoc3b-delete-iam-role-policy1.png)
![Delete IAM Role and IAM Policy](/images/5-Workshop/5.11-Cleanup/buoc3b-delete-iam-role-policy2.png)

---

### Step 4: Delete CloudWatch Alarms & Log Groups

1. Navigate to the **CloudWatch Console** -> select **Alarms** -> **All alarms**.
2. Select `WAFHighRequestRateAlarm` -> click **Actions** -> select **Delete**.

![Delete CloudWatch Alarm](/images/5-Workshop/5.11-Cleanup/buoc4a-delete-cloudwatch-alarm.png)

3. Go to **Logs** -> select **Log groups**.
4. Select and delete the following log groups:
   - `aws-waf-logs-cloudfront`
   - `/aws/lambda/WAFAutoBlockFunction`

![Delete CloudWatch Log Groups](/images/5-Workshop/5.11-Cleanup/buoc4b-delete-log-groups.png)
![Delete CloudWatch Log Groups](/images/5-Workshop/5.11-Cleanup/buoc4b-delete-log-groups1.png)

---

### Step 5: Delete Amazon SNS Topic & Subscriptions

1. Navigate to the **Amazon SNS Console** -> select **Topics**.
2. Select `skylink-waf-alerts` -> click **Delete**.
3. Go to **Subscriptions** -> select all subscriptions associated with Email and Lambda -> click **Delete**.

![Delete Amazon SNS Topic and Subscriptions](/images/5-Workshop/5.11-Cleanup/buoc5-delete-sns-topic.png)
![Delete Amazon SNS Topic and Subscriptions](/images/5-Workshop/5.11-Cleanup/buoc5-delete-sns-topic1.png)

---

### Step 6: Delete S3 Bucket / Origin Server (If Applicable)

1. Navigate to the **Amazon S3 Console**.
2. Select the S3 bucket storing the website/portfolio source code.
3. Click **Empty** to remove all objects inside the bucket.
4. Click **Delete** to permanently delete the S3 bucket.

![Clean up and delete Amazon S3 Bucket](/images/5-Workshop/5.11-Cleanup/buoc6-delete-s3-bucket1.png)
![Clean up and delete Amazon S3 Bucket](/images/5-Workshop/5.11-Cleanup/buoc6-delete-s3-bucket2.png)
![Clean up and delete Amazon S3 Bucket](/images/5-Workshop/5.11-Cleanup/buoc6-delete-s3-bucket3.png)

---

## Cleanup Verification Table

| AWS Service    | Target Resource                                                  | Status      |
| :------------- | :--------------------------------------------------------------- | :---------- |
| **CloudFront** | Distribution (`https://d2n9euazwledbh.cloudfront.net/`)                    | **Deleted** |
| **AWS WAF**    | Web ACL, `AutoBlockedIPSetV6`, `AutoBlockedIPSet`                | **Deleted** |
| **AWS Lambda** | Function `WAFAutoBlockFunction`                                  | **Deleted** |
| **IAM**        | Role `LambdaWAFAutoBlockRole`, Policy `LambdaWAFAutoBlockPolicy` | **Deleted** |
| **CloudWatch** | Alarm `WAFHighRequestRateAlarm`, WAF & Lambda Log Groups         | **Deleted** |
| **Amazon SNS** | Topic `skylink-waf-alerts` & Email Subscriptions                      | **Deleted** |

> **Note:** Resource deletion is permanent and cannot be undone. All system configurations and project source code are safely backed up in the [GitHub Repository](https://github.com/quangg199/SkyLink-Airline-System).
