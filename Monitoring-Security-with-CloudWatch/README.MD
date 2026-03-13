# Monitoring Security with Amazon CloudWatch

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Well--Architected-blue)
![Tools](https://img.shields.io/badge/Tools-CloudWatch%2C%20CloudWatch%20Agent%2C%20SNS%2C%20EC2%2C%20NAT%20Gateway-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 9 Lab: Monitoring Security with Amazon CloudWatch — Log Ingestion, Metric Filters, Authentication Failure Alarms, and NAT Gateway Traffic Alerting

---

## **2. Purpose and Objectives**

The objective of this lab is to implement **security-focused monitoring** using Amazon CloudWatch by installing and configuring the CloudWatch agent to ingest security logs, creating metric filters to detect authentication failures, building alarms with SNS notifications, and configuring NAT gateway traffic alerts. This lab establishes a practical security observability stack aligned with AWS operational best practices.

**Key Learning Outcomes:**
- Install and configure the **Amazon CloudWatch Agent** on an EC2 Linux instance
- Ingest `/var/log/secure` into a CloudWatch **Log Group** for centralized security log management
- Create a **CloudWatch Metric Filter** to detect and count authentication failure events
- Configure a **CloudWatch Alarm** with SNS email notification for threshold-based alerting
- Generate test authentication failure events and verify alarm triggers and email delivery
- Create a second alarm monitoring **NAT gateway `BytesOutToDestination`** metric for data exfiltration detection
- Trigger the NAT gateway alarm by uploading a large file to S3

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Security pillar — detective controls, logging, and monitoring
- **Security Information and Event Management (SIEM) Concepts:** Log aggregation, pattern detection, and alert correlation
- **Defense in Depth:** Monitoring authentication events and network egress as complementary security controls
- **Threshold-Based Alerting:** Statistical alarm thresholds for both security events and network anomaly detection
- **SNS Notification Integration:** Multi-channel alert delivery as an incident response enablement pattern

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Monitoring Service:** Amazon CloudWatch (Logs, Metrics, Alarms, Dashboards)
- **Agent:** Amazon CloudWatch Agent
- **Notification Service:** Amazon SNS
- **Compute:** Amazon EC2 (Database Server — Linux)
- **Network:** AWS NAT Gateway
- **Log Source:** `/var/log/secure` (Linux authentication log)
- **Log Group:** `database_server_security_logs`
- **Metric Filter Pattern:** `"authentication failure"`
- **Metric Namespace:** `authentication failures`
- **Alarms Created:**
  - Authentication failure alarm: Sum > 2 over 5 minutes
  - NAT gateway alarm: BytesOutToDestination Sum > 2,000,000 over 15 minutes
- **SNS Topic:** `CloudWatch_alarm_notifications`
- **Test File:** `SampleData.zip` (~2.3MB)

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 1: Install and Configure CloudWatch Agent**
   - Connected to the **Database Server** EC2 instance via Session Manager
   - Installed the Amazon CloudWatch agent:
     ```bash
     sudo yum install amazon-cloudwatch-agent -y
     ```
   - Ran the CloudWatch agent configuration wizard:
     ```bash
     sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
     ```
   - Wizard configuration choices:
     - OS: Linux
     - Environment: EC2
     - Run as user: root
     - Enable StatsD: No
     - Enable CollectD: No
     - Monitor host metrics: No
     - Monitor log files: **Yes**
     - Log file path: `/var/log/secure`
     - Log group name: `database_server_security_logs`
     - Log stream name: default (instance ID)
   - Started the CloudWatch agent with the generated configuration:
     ```bash
     sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
       -a fetch-config -m ec2 -s \
       -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
     ```
   - Generated test authentication failure events by attempting 4 failed `su dbdev` logins:
     ```bash
     su dbdev  # Enter wrong password 4 times
     ```
   - Navigated to **CloudWatch → Log Groups → database_server_security_logs**
   - Confirmed log events appeared with authentication failure messages

2. **Task 2: Create Metric Filter and Authentication Failure Alarm**
   - Navigated to the `database_server_security_logs` log group
   - Created a **metric filter**:
     - Filter pattern: `"authentication failure"`
     - Filter name: `database server authentication failures`
     - Metric namespace: `authentication failures`
     - Metric name: `authentication failures`
     - Metric value: `1`
     - Default value: `0`
   - Created a **CloudWatch Alarm** from the metric filter:
     - Statistic: `Sum`
     - Period: 5 minutes
     - Threshold: `> 2` (more than 2 failures in 5 minutes)
     - Alarm name: `database server authentication failures alarm`
   - Created **SNS topic** `CloudWatch_alarm_notifications`
   - Added an email subscription with a registered email address
   - Confirmed the subscription via the email confirmation link
   - Generated additional failed su logins to trigger the alarm
   - Verified alarm transitioned to **In alarm** state in the CloudWatch console
   - Confirmed SNS notification email was received with alarm details

3. **Task 3: Create NAT Gateway Traffic Alarm**
   - Created a second CloudWatch alarm for NAT gateway monitoring:
     - Metric: `BytesOutToDestination`
     - Namespace: `AWS/NatGateway`
     - Statistic: `Sum`
     - Period: 15 minutes
     - Threshold: `> 2,000,000` (2MB)
   - Attached the existing `CloudWatch_alarm_notifications` SNS topic
   - Retrieved the S3 test bucket name from the lab environment
   - Uploaded `SampleData.zip` (~2.3MB) to trigger the NAT gateway threshold:
     ```bash
     aws s3 cp ~/SampleData.zip s3://$TestBucket/
     ```
   - Monitored the NAT gateway alarm and confirmed it transitioned to **In alarm** state
   - Verified SNS notification email was received for the NAT gateway alarm

</details>

---

## **6. Findings and Results**

- **Finding 1:** The CloudWatch agent wizard provided a guided, interactive configuration process that produced a valid JSON configuration file without requiring manual JSON editing — significantly reducing the barrier to agent setup
- **Finding 2:** `/var/log/secure` authentication failure events appeared in the `database_server_security_logs` CloudWatch Log Group within seconds of generation, confirming near-real-time log streaming from the agent
- **Finding 3:** The metric filter pattern `"authentication failure"` successfully matched the `su` authentication failure log entries, incrementing the custom metric counter for each matching event
- **Finding 4:** Setting the alarm threshold to `Sum > 2` over 5 minutes provided a meaningful signal threshold — 3 or more failures in a 5-minute window is a credible brute-force indicator, reducing false positives from single accidental failures
- **Finding 5:** The authentication failure alarm transitioned to **In alarm** state after the 4 failed `su dbdev` logins, and the SNS email notification was delivered within seconds containing alarm name, current value, threshold, and state change time
- **Finding 6:** Uploading `SampleData.zip` (~2.3MB) via `aws s3 cp` routed egress traffic through the NAT gateway, pushing `BytesOutToDestination` above the 2MB threshold and triggering the NAT gateway alarm
- **Finding 7:** Both alarms used the same `CloudWatch_alarm_notifications` SNS topic, demonstrating that a single notification channel can aggregate alerts from multiple, diverse alarm sources

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand alarm configuration and monitoring reference</summary>

### Alarm Configuration Summary

| Alarm | Metric | Namespace | Statistic | Period | Threshold |
|-------|--------|-----------|-----------|--------|-----------|
| Auth Failure Alarm | `authentication failures` | `authentication failures` | Sum | 5 min | > 2 |
| NAT Gateway Alarm | `BytesOutToDestination` | `AWS/NatGateway` | Sum | 15 min | > 2,000,000 |

### Metric Filter Configuration

```
Log Group:      database_server_security_logs
Filter Name:    database server authentication failures
Filter Pattern: "authentication failure"
Metric Namespace: authentication failures
Metric Name:    authentication failures
Metric Value:   1
Default Value:  0
```

### SNS Topic Configuration

| Property | Value |
|----------|-------|
| Topic Name | `CloudWatch_alarm_notifications` |
| Protocol | Email |
| Subscription Status | Confirmed |
| Alarms Attached | Auth Failure + NAT Gateway |

### End-to-End Security Monitoring Flow

```
EC2 Database Server
    │
    │ /var/log/secure (auth events)
    ▼
CloudWatch Agent
    │
    │ Log streaming
    ▼
CloudWatch Log Group: database_server_security_logs
    │
    │ Metric Filter: "authentication failure"
    ▼
Custom Metric: authentication failures (Sum)
    │
    │ Alarm: Sum > 2 / 5 min
    ▼
CloudWatch Alarm → SNS Topic → Email Notification
```

### Test Results

| Test | Action | Expected | Actual |
|------|--------|----------|--------|
| Auth failure log ingestion | 4 failed `su dbdev` logins | Logs in CloudWatch | ✅ Confirmed |
| Metric filter matching | Failed logins generating metric | Counter incremented | ✅ Confirmed |
| Auth failure alarm trigger | Alarm Sum > 2 | In alarm state | ✅ Confirmed |
| SNS email delivery | Alarm state change | Email received | ✅ Confirmed |
| NAT gateway alarm trigger | 2.3MB S3 upload | In alarm state | ✅ Confirmed |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| CloudWatch agent not streaming logs immediately after start | Verified agent status with `systemctl status amazon-cloudwatch-agent` and confirmed it was active | Logs appeared in CloudWatch Log Group within 60 seconds of agent start |
| SNS subscription not confirmed before alarm testing | Completed the email confirmation click before triggering alarms | Subscription was active when alarm fired; email delivered immediately |
| NAT gateway alarm not triggering with small test files | Identified that only files >2MB routed through NAT would breach the threshold; used `SampleData.zip` (~2.3MB) | Upload exceeded threshold and triggered the alarm |
| Metric filter not matching expected log entries | Tested the filter pattern against sample log events using the CloudWatch filter test feature | Confirmed pattern matched before creating the alarm |

---

## **9. Key Takeaways and Lessons Learned**

- **Log Centralization Enables Security Analysis:** Streaming `/var/log/secure` to CloudWatch enables security teams to query, alert on, and archive authentication events centrally — rather than SSH-ing into each server to review local logs.

- **Metric Filters Bridge Logs and Alarms:** CloudWatch's metric filter capability transforms unstructured log data into quantifiable metrics that can drive alarms — connecting the logging and monitoring pillars of observability.

- **Authentication Threshold Design:** Setting the alarm at `Sum > 2` balances sensitivity and specificity. A threshold of 1 would generate alerts for every accidental failed login; a threshold of 10 might miss a slow brute-force attempt. Context-appropriate threshold selection is a security design decision.

- **NAT Gateway Monitoring Detects Exfiltration:** Alerting on high `BytesOutToDestination` from NAT gateways provides an early warning for potential data exfiltration scenarios — a critical detective control for protecting sensitive workloads.

- **Reusable SNS Topics Simplify Alert Management:** A single SNS topic can receive notifications from multiple alarms across different services. Managing subscriptions centrally reduces operational overhead when alert routing or recipients need to change.

---

## **10. Artifacts and Deliverables**

- **CloudWatch Agent Configuration:** Wizard-generated config for `/var/log/secure` log streaming
- **Metric Filter Definition:** Pattern, namespace, and metric configuration for authentication failure detection
- **Authentication Failure Alarm:** CloudWatch alarm with Sum > 2 / 5 min threshold and SNS integration
- **NAT Gateway Alarm:** BytesOutToDestination alarm with 2MB threshold for egress traffic monitoring
- **SNS Topic and Subscription:** Confirmed email subscription for multi-alarm notification delivery
- **End-to-End Security Monitoring Flow Diagram:** Complete pipeline from log source to email notification

---

## **11. Conclusion**

This lab successfully implemented a **security-focused monitoring stack** using Amazon CloudWatch, combining log ingestion, metric filtering, threshold alarms, and SNS notifications. The CloudWatch agent configuration streamed `/var/log/secure` authentication events to a centralized log group; the metric filter converted those events into quantifiable signals; and the alarm triggered SNS email notifications when authentication failures exceeded the defined threshold. A second NAT gateway alarm demonstrated how network-level metrics complement host-level security monitoring for comprehensive detective coverage. Together, these controls establish a practical foundation for security observability in AWS environments.

---

## **12. References**

- [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [Amazon CloudWatch Agent User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [CloudWatch Metric Filters](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html)
- [CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [Amazon SNS Documentation](https://docs.aws.amazon.com/sns/)
- [NAT Gateway CloudWatch Metrics](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway-cloudwatch.html)
- [AWS Well-Architected Framework – Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Fundamentals
**Status:** Completed ✅
