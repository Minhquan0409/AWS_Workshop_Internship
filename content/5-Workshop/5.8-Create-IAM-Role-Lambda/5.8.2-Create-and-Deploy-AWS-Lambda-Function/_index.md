---
title: "Creating and Deploying the AWS Lambda Function"
date: 2026-08-24
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---

This article walks through the detailed steps to create a new AWS Lambda function in Python, configure its Execution Role, set up Environment Variables, and deploy source code that automatically extracts offending IPs from CloudWatch Logs and updates them into a WAF IP Set.

---

## 1. Create the AWS Lambda Function

### Step 1: Access the AWS Lambda Service

1. Sign in to the **AWS Management Console**.
2. In the search bar, type `Lambda` and select the **Lambda** service.
3. Make sure the selected Region is **US East (N. Virginia) us-east-1**.
4. Click **Create function**.

![Access the AWS Lambda service](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/lambda-dashboard.png)

---

### Step 2: Configure Basic Settings

1. Select **Author from scratch**.
2. **Function name**: Enter `WAFAutoBlockFunction`.
3. **Runtime**: Select **Python 3.12** (or the latest Python 3.x version).
4. **Architecture**: Keep the default **x86_64**.
5. Expand **Change default execution role**:
   - Select **Use an existing role**.
   - **Existing role**: Select the `LambdaWAFAutoBlockRole` role created in section 5.8.1.
6. Click **Create function**.

![Configure Lambda creation settings](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/create-function-config.png)
![Configure Lambda creation settings](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/create-function-config1.png)

---

## 2. Configure Environment Variables & General Configuration

### Step 1: Set Up Environment Variables

1. On the `WAFAutoBlockFunction` details page, select the **Configuration** tab, then select **Environment variables**.
2. Click **Edit** and add the following key-value pairs:

| Key               | Value                     | Description                                         |
| :---------------- | :------------------------ | :-------------------------------------------------- |
| `LOG_GROUP_NAME`  | `aws-waf-logs-cloudfront` | Name of the WAF CloudWatch Log Group                |
| `IPSET_NAME`      | `AutoBlockedIPSetV6`      | Name of the WAF IP Set storing the block list       |
| `IPSET_SCOPE`     | `CLOUDFRONT`              | Scope of the IP Set (CLOUDFRONT)                    |
| `LIMIT_THRESHOLD` | `100`                     | Maximum number of requests allowed from a single IP |

3. Click **Save**.

![Configure Environment Variables](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/environment-variables.png)

---

### Step 2: Adjust the Lambda Timeout

1. Still in the **Configuration** tab, select **General configuration**.
2. Click **Edit**.
3. **Timeout**: Increase the processing time from the default `3 sec` to **1 min 0 sec** (to ensure enough time for CloudWatch Logs Insights queries and the WAF API update).
4. Click **Save**.

![Adjust the Lambda timeout](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/general-configuration.png)

---

## 3. Deploy the Python Source Code

### Step 1: Add the Lambda Source Code

1. Switch to the **Code** tab.
2. In the `lambda_function.py` editor, paste the following automation source code:

```python
import os
import time
import boto3
from datetime import datetime, timedelta

# Initialize AWS SDK clients
logs_client = boto3.client('logs', region_name='us-east-1')
waf_client = boto3.client('wafv2', region_name='us-east-1')

def lambda_handler(event, context):
    log_group_name = os.environ.get('LOG_GROUP_NAME', 'aws-waf-logs-cloudfront')
    ipset_name = os.environ.get('IPSET_NAME', 'AutoBlockedIPSetV6')
    ipset_scope = os.environ.get('IPSET_SCOPE', 'CLOUDFRONT')
    threshold = int(os.environ.get('LIMIT_THRESHOLD', '100'))

    print(f"Starting log analysis for Log Group: {log_group_name} with threshold: {threshold}")

    # Query to find IPs exceeding the request threshold over the last 5 minutes
    query = f"fields httpRequest.clientIP | stats count(*) as requestCount by httpRequest.clientIP | filter requestCount > {threshold}"

    end_time = datetime.utcnow()
    start_time = end_time - timedelta(minutes=5)

    start_query_response = logs_client.start_query(
        logGroupName=log_group_name,
        startTime=int(start_time.timestamp()),
        endTime=int(end_time.timestamp()),
        queryString=query
    )

    query_id = start_query_response['queryId']
    response = None

    # Wait for CloudWatch Insights to finish the query
    while response is None or response['status'] == 'Running':
        time.sleep(1)
        response = logs_client.get_query_results(queryId=query_id)

    offending_ips = []
    for row in response['results']:
        for field in row:
            if field['field'] == 'httpRequest.clientIP':
                ip = field['value']
                # Format as IPv6 /128 or IPv4 /32
                ip_cidr = f"{ip}/128" if ":" in ip else f"{ip}/32"
                offending_ips.append(ip_cidr)

    print(f"Found offending IPs: {offending_ips}")

    if not offending_ips:
        return {'statusCode': 200, 'body': 'No IPs exceeded the threshold.'}

    # Retrieve the current IP Set from WAF
    ip_sets = waf_client.list_ip_sets(Scope=ipset_scope)
    ipset_id = None
    for ipset in ip_sets.get('IPSets', []):
        if ipset['Name'] == ipset_name:
            ipset_id = ipset['Id']
            break

    if not ipset_id:
        raise Exception(f"IP Set named {ipset_name} not found")

    get_ipset_res = waf_client.get_ip_set(Name=ipset_name, Scope=ipset_scope, Id=ipset_id)
    lock_token = get_ipset_res['LockToken']
    current_addresses = get_ipset_res['IPSet']['Addresses']

    # Merge the new IPs into the existing list (removing duplicates)
    updated_addresses = list(set(current_addresses + offending_ips))

    # Update the WAF IP Set
    waf_client.update_ip_set(
        Name=ipset_name,
        Scope=ipset_scope,
        Id=ipset_id,
        Addresses=updated_addresses,
        LockToken=lock_token
    )

    print(f"Successfully updated WAF IP Set {ipset_name}. New IP list: {updated_addresses}")

    return {
        'statusCode': 200,
        'body': f"Successfully blocked IPs: {offending_ips}"
    }
```

---

### Step 2: Deploy the Source Code

1. After pasting the code above into the editor, click **Deploy** to save and apply the new source code.

![Deploy the Lambda source code](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/general-configuration2.png)

---

## 4. Expected Outcome

After completing this exercise:

- The AWS Lambda function `WAFAutoBlockFunction` is successfully created with the Python 3.12 runtime and correctly assigned the `LambdaWAFAutoBlockRole` Execution Role.
- The environment variables `LOG_GROUP_NAME`, `IPSET_NAME`, `IPSET_SCOPE`, and `LIMIT_THRESHOLD` are configured correctly.
- The source code that automatically analyzes CloudWatch Logs and calls the WAF `UpdateIPSet` API is deployed and ready to receive an automated trigger signal.
