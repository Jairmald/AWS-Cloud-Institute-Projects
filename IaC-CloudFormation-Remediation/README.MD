# Infrastructure as Code – CloudFormation Drift Detection and Remediation

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Well--Architected-blue)
![Tools](https://img.shields.io/badge/Tools-CloudFormation%2C%20Lambda%2C%20EventBridge%2C%20EC2-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Infrastructure as Code Practice Lab: CloudFormation Drift Detection and Automated Remediation — Detecting, Identifying, and Correcting Configuration Drift using Lambda and EventBridge

---

## **2. Purpose and Objectives**

The objective of this lab is to gain hands-on experience with **AWS CloudFormation drift detection**, understand how manual out-of-band changes cause infrastructure drift, and implement automated drift detection and remediation using **AWS Lambda** and **Amazon EventBridge**. This lab reinforces the principle that infrastructure managed by IaC must not be modified manually.

**Key Learning Outcomes:**
- Provision AWS resources using a **CloudFormation template** as the baseline configuration
- Simulate configuration **drift** by manually modifying a resource outside of CloudFormation
- Use CloudFormation **Detect Drift** to identify which resources have drifted from the template definition
- Implement an **AWS Lambda function** to automatically remediate detected drift
- Create an **EventBridge scheduled rule** to run the Lambda function on a regular cadence
- Understand the operational risk of configuration drift in IaC-managed environments

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence — detecting and correcting drift to maintain desired state
- **Infrastructure as Code Principles:** Treating infrastructure definitions as the single source of truth
- **GitOps Philosophy:** Any change to infrastructure must go through the IaC pipeline, never applied manually
- **Automated Remediation Patterns:** Using Lambda + EventBridge for scheduled drift detection and correction
- **Defense in Depth:** Automated compliance enforcement as a layer in the security and operations posture

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **IaC Service:** AWS CloudFormation
- **Remediation Compute:** AWS Lambda (Python runtime)
- **Scheduling Service:** Amazon EventBridge (scheduled rule)
- **Target Resource:** Amazon EC2 Security Group (simulated drift target)
- **Drift Detection:** CloudFormation Drift Detection (console + CLI)
- **AWS Services Used:**
  - AWS CloudFormation (stack management, drift detection)
  - AWS Lambda (drift remediation function)
  - Amazon EventBridge (scheduled invocation)
  - Amazon EC2 (security groups as drift target)
  - AWS IAM (Lambda execution role with CloudFormation and EC2 permissions)

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Provision Resources via CloudFormation**
   - Deployed a CloudFormation template that created an EC2 instance with an associated security group
   - The template defined specific security group rules as the desired state
   - Verified the stack reached `CREATE_COMPLETE` and all resources were provisioned as defined
   - Noted the security group ID from the stack's Resources tab for later drift simulation

2. **Simulate Configuration Drift**
   - Navigated to **EC2 → Security Groups** in the console
   - Located the security group created by the CloudFormation stack
   - Manually added an additional **inbound rule** to the security group:
     - Protocol: TCP
     - Port: 8080
     - Source: 0.0.0.0/0
   - Saved the rule change directly in the console — bypassing CloudFormation
   - This manual change represented a configuration drift from the template-defined desired state

3. **Detect Drift with CloudFormation**
   - Navigated to **CloudFormation → Stacks → [Stack Name]**
   - Selected **Stack actions → Detect drift**
   - Waited for the drift detection to complete
   - Reviewed the **Drift status**: showed `DRIFTED`
   - Clicked into the drift results to see which resources were affected
   - Confirmed `InstanceSecurityGroup` showed status `MODIFIED`
   - Reviewed the drift details showing the added inbound rule not present in the template

4. **Implement Lambda Remediation Function**
   - Created a Lambda function with Python runtime and an execution role with permissions for CloudFormation DescribeStackResourceDrifts and EC2 AuthorizeSecurityGroupIngress/RevokeSecurityGroupIngress
   - Lambda function logic:
     - Called `describe_stack_resource_drifts()` to identify drifted resources
     - For security group drifts: compared current rules against the expected rules from the template
     - Removed the unauthorized inbound rule by calling `revoke_security_group_ingress()`
     - Logged remediation actions to CloudWatch Logs
   - Tested the Lambda function manually and confirmed the unauthorized rule was removed
   - Re-ran CloudFormation drift detection to confirm the stack returned to `IN_SYNC` state

5. **Create EventBridge Scheduled Rule**
   - Navigated to **Amazon EventBridge → Rules → Create rule**
   - Set the rule type to **Schedule**
   - Configured a rate-based schedule (e.g., every 1 hour or using a cron expression)
   - Set the target as the Lambda remediation function
   - Confirmed the rule was enabled and would invoke Lambda on the configured schedule
   - Verified Lambda was invoked on the next scheduled trigger

</details>

---

## **6. Findings and Results**

- **Finding 1:** The manual addition of port 8080 inbound rule to the security group was immediately reflected in the EC2 console but CloudFormation was unaware of the change until drift detection was explicitly triggered
- **Finding 2:** CloudFormation's drift detection accurately identified the `InstanceSecurityGroup` resource as `MODIFIED` and showed the exact delta — the added inbound rule — between the current state and the template definition
- **Finding 3:** The drift details view provided a property-level comparison between expected (template) and actual (current) resource configuration, making it clear exactly what changed and how
- **Finding 4:** The Lambda remediation function successfully identified the drift, isolated the unauthorized security group rule, and revoked it using the EC2 API — restoring the resource to its template-defined state
- **Finding 5:** After Lambda remediation, re-running CloudFormation drift detection confirmed the stack returned to `IN_SYNC` status — validating the automated remediation was effective
- **Finding 6:** The EventBridge scheduled rule successfully triggered the Lambda function on schedule, demonstrating a fully automated, recurring drift detection and remediation cycle
- **Finding 7:** The entire workflow — drift introduction → detection → remediation → verification — demonstrated that manual changes to IaC-managed resources can be automatically detected and corrected without human intervention

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand architecture and drift detection reference</summary>

### Drift Detection and Remediation Architecture

```
Manual Change (Console)
        │
        ▼
EC2 Security Group (Modified — port 8080 added)
        │
        │ CloudFormation sees mismatch
        ▼
CloudFormation Drift Detection
→ Stack Status: DRIFTED
→ Resource: InstanceSecurityGroup — MODIFIED
        │
        │ EventBridge Schedule triggers
        ▼
AWS Lambda (Remediation Function)
→ Calls describe_stack_resource_drifts()
→ Identifies unauthorized rule
→ Calls revoke_security_group_ingress()
        │
        ▼
Security Group Restored to Template State
→ CloudFormation Drift Status: IN_SYNC
```

### Drift Detection Results

| Resource | Drift Status | Change Description |
|----------|-------------|-------------------|
| `InstanceSecurityGroup` | `MODIFIED` | Inbound rule for TCP:8080 added manually |

### EventBridge Rule Configuration

| Setting | Value |
|---------|-------|
| Rule Type | Schedule |
| Schedule Expression | `rate(1 hour)` |
| Target | Lambda Remediation Function |
| State | Enabled |

### Remediation Verification

| Step | Status |
|------|--------|
| Drift introduced (port 8080 added) | ✅ Confirmed |
| Drift detected by CloudFormation | ✅ `DRIFTED` status |
| Lambda remediation executed | ✅ Unauthorized rule revoked |
| Drift re-detection post-remediation | ✅ `IN_SYNC` status confirmed |
| EventBridge scheduled invocation | ✅ Lambda triggered on schedule |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Understanding drift detection doesn't run automatically | CloudFormation drift detection must be explicitly triggered (manually or via automation) — it does not poll continuously | Implemented EventBridge + Lambda to automate periodic detection |
| Lambda needing correct IAM permissions for EC2 and CloudFormation | Created execution role with `cloudformation:DescribeStackResourceDrifts` and `ec2:RevokeSecurityGroupIngress` permissions | Lambda successfully called both APIs without permission errors |
| Confirming remediation was effective | Re-ran CloudFormation drift detection after Lambda executed and verified return to `IN_SYNC` | Drift status confirmed cleared after remediation |
| Identifying the specific rule to revoke programmatically | Parsed the drift details to identify the exact IP permission object added and passed it to `revoke_security_group_ingress()` | Correct rule removed; no other rules affected |

---

## **9. Key Takeaways and Lessons Learned**

- **Drift is Silent Without Detection:** CloudFormation does not monitor for drift continuously. Manual changes to managed resources go undetected until drift detection is explicitly run — making automated periodic detection essential in production environments.

- **IaC is the Source of Truth:** Any change to infrastructure managed by CloudFormation should be made by updating the template and redeploying — never manually in the console. Manual changes create drift that undermines the reliability of the IaC baseline.

- **Lambda + EventBridge Enables Automated Governance:** Combining EventBridge schedules with Lambda functions creates a continuous compliance loop — detecting drift on a schedule and remediating it automatically without human intervention.

- **Drift Details Enable Precise Remediation:** CloudFormation's drift detail view provides property-level diffs between expected and actual state, enabling Lambda functions to make targeted, surgical corrections rather than wholesale resource recreation.

- **Audit Trail is Critical:** Every remediation action should be logged (CloudWatch Logs, AWS CloudTrail) to maintain an audit trail of what was changed, when, and by which automation — supporting compliance and incident investigation requirements.

---

## **10. Artifacts and Deliverables**

- **CloudFormation Stack:** Baseline infrastructure provisioned with security group as drift target
- **Drift Detection Results:** Documentation of `MODIFIED` status on `InstanceSecurityGroup` resource
- **Lambda Remediation Function:** Python function for automated security group rule enforcement
- **EventBridge Scheduled Rule:** Hourly invocation of Lambda for continuous drift monitoring
- **Drift Architecture Diagram:** End-to-end flow from manual change through detection to remediation

---

## **11. Conclusion**

This practice lab successfully demonstrated the complete **CloudFormation drift detection and automated remediation** workflow. By provisioning baseline infrastructure, simulating drift through a manual security group modification, detecting the drift via CloudFormation, and remediating it automatically with a Lambda function triggered by EventBridge, a fully automated infrastructure compliance loop was implemented. The return to `IN_SYNC` status after Lambda remediation validated the effectiveness of the approach. This pattern is directly applicable to production environments where security group rules, IAM policies, and other sensitive configurations must be continuously enforced to match their IaC-defined desired state.

---

## **12. References**

- [AWS CloudFormation Drift Detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Amazon EventBridge Scheduled Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html)
- [CloudFormation describe-stack-resource-drifts CLI](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/describe-stack-resource-drifts.html)
- [AWS Well-Architected Framework – Operational Excellence](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)
- [EC2 Security Groups API Reference](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_RevokeSecurityGroupIngress.html)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Operations
**Status:** Completed ✅
