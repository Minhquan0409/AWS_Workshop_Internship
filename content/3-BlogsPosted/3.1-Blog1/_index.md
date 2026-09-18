---

title: "Blog 1"
date: 2026-08-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "

---

# AMAZON EVENTBRIDGE — THE HEART OF EVENT-DRIVEN SECURITY ARCHITECTURE ON AWS

Amazon EventBridge is a fully managed serverless event bus service provided by AWS that allows you to connect applications through real-time events. In Security Detection & Response systems, EventBridge acts as the "heart" that orchestrates the entire automation workflow.

### How does EventBridge work?

When a user performs an action on AWS (for example, creating a new IAM User or changing an S3 Bucket policy), CloudTrail records the event. EventBridge continuously listens for these events and matches them against predefined Event Rules. If a match is found, EventBridge immediately triggers a Target, which can be a Lambda Function, SQS Queue, SNS Topic, etc.

Processing flow: CloudTrail API Call -> EventBridge Event Bus -> Event Rule Match -> Lambda Function (Target)

![EventBridge event pattern bus architecture](/images/3-BlogsPosted/event_pattern_bus_eventbridge_architecture.svg)

### Why is EventBridge suitable for Security Automation?

1. **Near Real-time Detection (Almost Instant Detection):** The delay between an event occurring and a Lambda Function being triggered is typically only a few seconds, which is much faster than periodically polling CloudTrail logs from S3.

2. **Event Pattern Filtering (Precise Event Filtering):** You can precisely define which events should be captured using JSON pattern syntax.

3. **Multi-Region Support:** One particularly important point is that AWS generates global IAM and Console Sign-in events in the `us-east-1` Region, regardless of where your actual resources are located. The solution is to create a separate Event Rule in `us-east-1` and point it to a Lambda Function located in the primary Region (for example, `ap-southeast-2`).

4. **Zero-Polling Architecture (No Polling Required):** Instead of scheduling periodic log checks (which consume DynamoDB RCUs and Lambda invocations), EventBridge only triggers the Lambda Function when an actual event occurs, resulting in significant cost savings.

### Key Highlights to Note

* An Event Rule can have multiple Targets (up to 5), for example, invoking a Lambda Function and sending an event to SQS at the same time.
* It is recommended to enable a Dead Letter Queue (DLQ) for the Lambda Target to prevent events from being lost if the Lambda Function fails.
* EventBridge provides at-least-once delivery — a Lambda Function may be invoked more than once for the same event, so your handler should be designed to be idempotent.
* **Custom Events Pricing:** AWS charges $1.00 per million events starting from the first event — there is no Free Tier for custom events. However, AWS Management Events (including CloudTrail API calls) ingested into the default event bus are completely free, which is exactly what our team's project uses. EventBridge Scheduler has a Free Tier of 14 million invocations per month, but this is a different feature.

With EventBridge, you can build a completely serverless AWS threat detection system that responds instantly without requiring server management — this is the foundation of an AWS-Native SOAR (Security Orchestration, Automation and Response) model.

---

## References

* [Amazon EventBridge Documentation](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
* [EventBridge Event Patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)