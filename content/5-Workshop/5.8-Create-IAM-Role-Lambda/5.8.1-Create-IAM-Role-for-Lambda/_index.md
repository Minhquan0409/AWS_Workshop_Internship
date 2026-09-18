---
title: "Creating an IAM Role for Lambda"
date: 2026-08-24
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

This article walks through the detailed steps to create a properly scoped IAM Policy and provision an IAM Execution Role for an AWS Lambda function using the AWS Management Console. This IAM Role grants the least-privilege permissions Lambda needs to read log data from CloudWatch Logs and update offending IPs in a WAF IP Set.

---

## 1. Create the IAM Policy for Lambda

### Step 1: Access the IAM Service

1. Sign in to the **AWS Management Console**.
2. In the search bar, type `IAM` and select the **IAM** service.
3. In the left navigation menu, select **Policies**.
4. Click **Create policy**.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu.png)

---

### Step 2: Define the IAM Policy (JSON)

1. On the Create policy screen, select the **JSON** tab.
2. Paste the following detailed permissions JSON into the editor:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudWatchLogsPermissions",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:FilterLogEvents",
        "logs:GetLogEvents"
      ],
      "Resource": "*"
    },
    {
      "Sid": "WAFGetAndUpdateIPSetPermissions",
      "Effect": "Allow",
      "Action": ["wafv2:GetIPSet", "wafv2:UpdateIPSet"],
      "Resource": "*"
    }
  ]
}
```

3. Click **Next**.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu1.png)

---

### Step 3: Name and Save the IAM Policy

1. In the **Policy name** field, enter `AWSLambdaWAFAutoBlockPolicy`.
2. **Description**: Enter `Policy granting Lambda permissions to read CloudWatch Logs and update the WAF IP Set.`
3. Click **Create policy**.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu2.png)
![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu3.png)

---

## 2. Create the IAM Execution Role

### Step 1: Create a New IAM Role

1. In the left navigation menu of the IAM console, select **Roles**.
2. Click **Create role**.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu4.png)

---

### Step 2: Select the Trusted Entity

1. **Trusted entity type**: Select **AWS service**.
2. **Use case**: Select **Lambda**.
3. Click **Next**.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu5.png)

---

### Step 3: Attach the IAM Policy to the Role

1. In the policy search bar, type `AWSLambdaWAFAutoBlockPolicy`.
2. Check the box next to the **AWSLambdaWAFAutoBlockPolicy** policy created in Part 1.
3. Click **Next**.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu6.png)

---

### Step 4: Finish Creating the IAM Role

1. **Role name**: Enter `LambdaWAFAutoBlockRole`.
2. **Description**: Enter `Execution Role for the Lambda function that automatically analyzes logs and blocks IPs in AWS WAF.`
3. Review the attached policy information.
4. Click **Create role**.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu7.png)
![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu8.png)

---

## 3. Verify the Result

Once the role has been created, check the details of `LambdaWAFAutoBlockRole`:

- **Role Name**: `LambdaWAFAutoBlockRole`
- **ARN**: `arn:aws:iam::881081146390:role/LambdaWAFAutoBlockRole`
- **Permissions**: The `AWSLambdaWAFAutoBlockPolicy` policy has been successfully attached.

![Access IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu9.png)

---

## 4. Expected Outcome

After completing this exercise:

- The IAM Policy `AWSLambdaWAFAutoBlockPolicy` correctly defines the `wafv2:GetIPSet`, `wafv2:UpdateIPSet`, and CloudWatch Logs permissions.
- The IAM Role `LambdaWAFAutoBlockRole` is provisioned and ready to be assigned directly as the Execution Role for the AWS Lambda function in section 5.8.2.
