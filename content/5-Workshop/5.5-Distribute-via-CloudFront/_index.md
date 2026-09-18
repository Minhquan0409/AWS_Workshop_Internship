---
title: "Distributing the Website via Amazon CloudFront"
date: 2026-08-24
weight: 5
chapter: false
pre: "<b>5.5. </b>"
---

## Objectives

In this section, we will deploy **Amazon CloudFront** to distribute the **SkyLink Airline** website, which is built with React and hosted on Amazon S3.

After completing this section, the system will move from direct access to S3 to the following architecture:

```text
User
  │
  │ HTTPS
  ▼
Amazon CloudFront
  │
  │ Origin Access Control (OAC)
  ▼
Amazon S3
  │
  └── React Frontend
  ```

## Main Objectives

* Use Amazon CloudFront as a CDN for the SkyLink Airline Frontend.
* Connect CloudFront to the S3 Bucket storing the website.
* Use Origin Access Control (OAC) to allow CloudFront to access S3.
* Keep the S3 Bucket private.
* Prevent users from accessing S3 directly.
* Configure HTTPS for the website.
* Test the website through the CloudFront Domain.
* Verify the accessibility and operation of the system.
* Perform security and resource clean-up steps.

## 1. Overview
### 1.1. Role of Amazon CloudFront

Amazon CloudFront is an AWS Content Delivery Network (CDN) service used to distribute website content to users through a network of Edge Locations.

In the SkyLink Airline project, the React Frontend is built into static files such as:
```
HTML
CSS
JavaScript
Images
Fonts
SVG
```

These files are stored in Amazon S3.

CloudFront sits between users and S3 to receive requests, cache content, and distribute the content to users.

Deployment architecture:
```
                    Internet
                        │
                        ▼
                ┌───────────────┐
                │   CloudFront  │
                │      CDN      │
                └───────┬───────┘
                        │
                        │ OAC
                        ▼
                ┌───────────────┐
                │      S3       │
                │    Private    │
                └───────┬───────┘
                        │
                        ▼
                React Frontend
```

### 1.2. Why Use CloudFront?

If users access the S3 Website Endpoint directly:
```
User
  │
  ▼
S3 Website Endpoint
  │
  ▼
React Frontend
```
S3 must provide the website content directly to the Internet.

In the completed SkyLink architecture, S3 is kept private and CloudFront is used as the distribution layer.
```
User
  │
  ▼
CloudFront
  │
  ▼
Private S3
```

### 1.3. Architecture Before Deploying CloudFront

In Section 5.4, the SkyLink Frontend was stored on Amazon S3.
```
User
  │
  ▼
Amazon S3
  │
  ├── index.html
  ├── assets/
  └── images/
```

During the Static Website Hosting testing process, the S3 Website Endpoint can be used to verify that the website is working correctly.

After the testing is completed, the Bucket is switched back to private mode.

When directly accessing the S3 Website Endpoint:
```
HTTP 403 Forbidden
AccessDenied
```
this is the expected result when the Bucket is no longer public.

## 2. Practical Implementation
### 2.1. Create an Amazon CloudFront Distribution

### Step 1: Access the Amazon CloudFront Service
1. Sign in to the AWS Management Console.
2. In the search bar, enter CloudFront and select the CloudFront service.
3. In the CloudFront Dashboard, click Create distribution.

![Truy cập CloudFront Console](/images/5-Workshop/5.5-Distribute-via-CloudFront/cloudfront-console.png)

---

### Step 2: Configure Origin Settings & Origin Access Control (OAC)
1. Origin domain: Select the S3 Bucket created in Section 5.4 (for example: skylink-airline-frontend-2026.s3.ap-southeast-1.amazonaws.com).
2. Name: Keep the default suggested name.
3. Origin access: Select Origin access control settings (recommended).
4. Click Create new OAC (if an OAC is not already available) → Keep the default name → Click Create.

![Cấu hình Origin và OAC](/images/5-Workshop/5.5-Distribute-via-CloudFront/origin-oac-config.png)

---

### Step 3: Configure Default Cache Behavior
1. Viewer protocol policy: Select Redirect HTTP to HTTPS to enforce encryption for all incoming traffic.
2. Allowed HTTP methods: Keep the default GET, HEAD.
3. Cache key and origin requests: Keep the default CachingOptimized.

![Cấu hình Default Cache Behavior](/images/5-Workshop/5.5-Distribute-via-CloudFront/cache-behavior.png)

---

### Step 4: Configure Web Application Firewall (WAF) & Settings
1. Web Application Firewall (WAF): Temporarily select Do not enable security protections. AWS WAF will be integrated in detail in Section 5.6.
2. Default root object: Enter index.html.
3. Scroll down to the bottom of the page and click Create distribution.

![Cấu hình Root Object và nhấn Create](/images/5-Workshop/5.5-Distribute-via-CloudFront/create-distribution.png)
![Cấu hình Root Object và nhấn Create](/images/5-Workshop/5.5-Distribute-via-CloudFront/create-distribution2.png)

---

### 2.2. Update the S3 Bucket Policy with OAC

After the CloudFront Distribution is successfully created, a notification will appear requesting that the S3 Bucket Policy be updated to grant access to the OAC.

### Step 1: Copy the Bucket Policy from CloudFront
On the details page of the newly created CloudFront Distribution, click the Copy policy button in the blue notification box.

![Copy S3 Bucket Policy](/images/5-Workshop/5.5-Distribute-via-CloudFront/copy-bucket-policy.png)

---

### Step 2: Paste the Policy into the Amazon S3 Bucket
1. Return to the Amazon S3 service and select your S3 Bucket.
Switch to the Permissions tab.
2. In the Bucket policy section, click Edit.
3. Paste the Policy copied from CloudFront into the JSON editor.
Click Save changes.

![Cập nhật S3 Bucket Policy](/images/5-Workshop/5.5-Distribute-via-CloudFront/update-s3-policy.png)

![Cập nhật S3 Bucket Policy](/images/5-Workshop/5.5-Distribute-via-CloudFront/update-s3-policy2.png)

---

### 2.3. Verify the Distribution Result (Distribution Domain Name)

### Step 1: Get the CloudFront Domain Name
1. Return to the CloudFront Distributions interface.
2. Find the Domain name column or copy the Distribution domain name from the distribution details page (for example: d2n9euazwledbh.cloudfront.net).

![Lấy CloudFront Domain Name](/images/5-Workshop/5.5-Distribute-via-CloudFront/get-domain-name.png)

---

### Step 2: Access the Website for Testing
1. Wait for the Distribution status to finish deploying.
2. Open a new browser tab and access the following URL: : `https://d2n9euazwledbh.cloudfront.net/`

![Kiểm tra truy cập CloudFront Domain](/images/5-Workshop/5.5-Distribute-via-CloudFront/test-website-access.png)

---

### 2.4. Expected Results

After completing this practical exercise:

* The CloudFront Distribution is successfully created and correctly connected to the S3 Origin.
* The S3 Bucket Policy is successfully updated to grant access through OAC, preventing direct access through the S3 URL and allowing requests from CloudFront.
* The SkyLink website is successfully accessible through HTTPS using the CloudFront Domain Name.
* The S3 Bucket remains private while CloudFront acts as the public distribution layer.


