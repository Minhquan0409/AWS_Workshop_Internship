---
title: "System Testing"
date: 2026-08-24
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Objective

After completing the configuration of the entire infrastructure (CloudFront, AWS WAF, CloudWatch Logs, Metric Filter, CloudWatch Alarm, AWS Lambda, and SNS Topic), this testing phase aims to verify the correctness and automatic response capability of the system against an HTTP Flood attack.

---

## Project Links

- **Website Demo:** [https://d2n9euazwledbh.cloudfront.net/](https://d2n9euazwledbh.cloudfront.net/)
- **GitHub Repository:** [https://github.com/quangg199/SkyLink-Airline-System](https://github.com/quangg199/SkyLink-Airline-System)

---

### Test Scenario

- **Objective:** Simulate a burst attack of over **100 requests within 5 minutes** from the test machine to the CloudFront Distribution address to trigger the automated processing chain.
- **Expected outcomes:**
  1. The CloudWatch Log Group `aws-waf-logs-cloudfront` fully records the access logs.
  2. The CloudWatch Alarm `WAFHighRequestRateAlarm` switches to the **`IN ALARM`** state.
  3. The AWS Lambda function `WAFAutoBlockFunction` is automatically triggered and correctly extracts the offending IP address (IPv4/IPv6).
  4. The offending IP address is pushed into the WAF IP Set (`AutoBlockedIPSetV6` or `AutoBlockedIPSet`).
  5. AWS WAF applies the blocking rule, returning a **`403 Forbidden`** error when that IP continues to access.
  6. The system sends an automatic email notification via SNS with the subject `[AWS WAF AUTO-BLOCK] Successfully blocked offending IP` containing the IP's detailed information.

---

### Test Execution Steps

#### Step 1: Simulate High Traffic Volume (HTTP Flood Simulation)

Open **Command Prompt (CMD)** or **PowerShell** on the test machine and run a loop to send 200 HTTP requests to CloudFront:

```cmd
for /L %i in (1,1,200) do @curl -s -o nul -w "Request %i - Status: %{http_code}\n" https://d2n9euazwledbh.cloudfront.net/
```

> **Note:** Observe the output on the screen — initially, the results will show successful HTTP `200` status codes.

![Result of running the loop to send 200 HTTP requests via CMD/PowerShell](/images/5-Workshop/5.10-Testing/buoc1-http-flood-simulation.png)

#### Step 2: Check Logs and CloudWatch Alarm

**CloudWatch Logs:**

- Navigate to **CloudWatch → Log groups → aws-waf-logs-cloudfront**.
- Check the most recent Log Streams to confirm that the requests sent from `curl` have been recorded by WAF.

![Log Group aws-waf-logs-cloudfront with the most recent Log Streams](/images/5-Workshop/5.10-Testing/buoc2-cloudwatch-log-group.png)

**CloudWatch Alarm:**

- Navigate to **CloudWatch → Alarms → All alarms**.
- Observe the `WAFHighRequestRateAlarm` alarm. After 1–3 minutes, the `WAFRequestCount` metric will exceed the threshold of 100, and the alarm will switch to the **In alarm** state.

![CloudWatch Alarm WAFHighRequestRateAlarm switching to the In alarm state](/images/5-Workshop/5.10-Testing/buoc2-cloudwatch-alarm-in-alarm.png)

#### Step 3: Check AWS Lambda Execution Logs (CloudWatch Logs / Execution Logs)

- Navigate to **AWS Lambda → select the `WAFAutoBlockFunction` function → select the Monitor tab → select View CloudWatch logs**.
- Open the most recent Log Stream and check the log content:

```text
Starting log analysis from Log Group: aws-waf-logs-cloudfront with threshold: 100
Total offending IPs found: ['2405:4802:1d5e:a560:782e:f496:9ce7:d360/128']
IPs compatible with the IP Set (IPV6): ['2405:4802:1d5e:a560:782e:f496:9ce7:d360/128']
Successfully updated WAF IP Set AutoBlockedIPSetV6. New IP list: ['2405:4802:1d5e:a560:782e:f496:9ce7:d360/128']
Successfully sent detailed blocked-IP notification email via SNS.
```

![Execution Logs of the Lambda function WAFAutoBlockFunction](/images/5-Workshop/5.10-Testing/buoc3-lambda-execution-logs.png)

#### Step 4: Check the WAF IP Set and Blocking Response (Blocking Verification)

**Check the WAF IP Set in the AWS Console:**

- Navigate to **AWS WAF → IP sets** (select the **Global (CloudFront)** Region).
- Open `AutoBlockedIPSetV6` (or `AutoBlockedIPSet`) and confirm that the test machine's IP address has been automatically added to the list.

![WAF IP Set AutoBlockedIPSetV6 containing the offending IP address](/images/5-Workshop/5.10-Testing/buoc4-waf-ip-set.png)

**Verify the actual blocking response:**

Open CMD and send a single test request:

```dos
curl -I https://d2n9euazwledbh.cloudfront.net/
```

**Result:** Returns the `HTTP/1.1 403 Forbidden` error (confirming the block was successful).

![HTTP 403 Forbidden response when the blocked IP continues to access](/images/5-Workshop/5.10-Testing/buoc4-http-403-forbidden.png)

#### Step 5: Check the Email Notification from SNS

Check the inbox of the Gmail account registered with the SNS Topic; you will receive 2 emails:

1. **Threshold alert email from CloudWatch Alarm:** Notifies that the system has exceeded the request threshold.

   ![Threshold alert email for the request count from CloudWatch Alarm](/images/5-Workshop/5.10-Testing/buoc5-email-cloudwatch-alarm.png)

2. **Detailed notification email from Lambda:**
   - **Subject:** `[AWS WAF AUTO-BLOCK] Successfully blocked offending IP`
   - **Body:**

     ```text
     The system automatically detected and blocked an abnormal access IP:

     - List of blocked IP(s): 2405:4802:1d5e:a560:782e:f496:9ce7:d360/128
     - WAF IP Set name: AutoBlockedIPSetV6
     - IP version: IPV6
     - Execution time: 2026-09-10 17:39:23 UTC

     The above IP(s) have been automatically added to the WAF IP Set and denied access.
     ```

   ![Detailed notification email about the blocked IP sent from Lambda via SNS](/images/5-Workshop/5.10-Testing/buoc5-email-lambda-auto-block.png)

---

### Test Conclusion

| Test Item          | Expected Status                                        | Actual Status                                          | Conclusion |
| ------------------ | ------------------------------------------------------ | ------------------------------------------------------ | ---------- |
| WAF Log Recording  | Access logs are pushed to CloudWatch Logs              | Logs appear in the Log Group `aws-waf-logs-cloudfront` | Pass       |
| Alarm Triggering   | Switches to `IN ALARM` when > 100 requests/5 min       | Alarm triggers correctly once the threshold is reached | Pass       |
| Lambda Automation  | Correctly extracts the offending IP (`clientIp`)       | Successfully extracts the offending IPv6/IPv4 address  | Pass       |
| WAF IP Set Update  | Adds the offending IP to the corresponding IP Set      | IP successfully merged into `AutoBlockedIPSetV6`       | Pass       |
| Denial Response    | Returns a `403 Forbidden` error for the blocked IP     | Successfully blocked; returns HTTP 403 on curl         | Pass       |
| Email Notification | Sends a detailed notification email for the blocked IP | SNS email delivered to Gmail with full IP details      | Pass       |

**Overall assessment:** The system responds fully automatically, successfully detecting and blocking access that exceeds the threshold, in accordance with the intended architecture design.
