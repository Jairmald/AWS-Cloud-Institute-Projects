# Using AWS CloudFormation

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Well--Architected-blue)
![Tools](https://img.shields.io/badge/Tools-CloudFormation%2C%20AWS%20CLI%2C%20Code%20Editor-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Using AWS CloudFormation — Authoring Templates, Deploying Stacks, Detecting Drift, and Applying Change Sets via AWS CLI and Code Editor

---

## **2. Purpose and Objectives**

The objective of this lab is to develop hands-on proficiency with **AWS CloudFormation** by authoring a template with parameters, resources, and outputs; deploying it via the AWS CLI; detecting and investigating drift; and applying a change set to remediate drift and update the stack. This lab covers the complete CloudFormation lifecycle from template authoring to change management.

**Key Learning Outcomes:**
- Set up and use the **AWS Code Editor IDE** for CloudFormation template authoring
- Add a **Parameters section** with `InstanceType` parameter and allowed values
- Add a **Resources section** with an `AWS::EC2::Route` resource and `DependsOn` reference
- Add an **Outputs section** with a dynamic `AppURL` using `!Sub`
- Deploy a CloudFormation stack using **`aws cloudformation create-stack`** via CLI
- Detect and describe stack drift using `aws cloudformation describe-stack-resource-drifts`
- Create and execute a **change set** to update the stack and remediate drift

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence — IaC lifecycle management and change control
- **Infrastructure as Code Best Practices:** Template parameterization, resource dependencies, and output exports
- **Change Management:** Using CloudFormation change sets as a safe, reviewable update mechanism
- **Drift Detection and Remediation:** Identifying manual changes and restoring desired state through IaC updates
- **CLI-Driven Deployments:** Using `aws cloudformation` commands for scriptable, repeatable stack operations

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **IaC Service:** AWS CloudFormation
- **IDE:** AWS Code Editor (LabWorkspaceURL)
- **CLI:** AWS CLI (`aws cloudformation` commands)
- **Template:** `lab1.yaml` (YAML-format CloudFormation template)
- **Stack Name:** `Lab1`
- **Key Template Sections Added:**
  - `Parameters`: `InstanceType` (allowed: t3.nano, t3.micro, t3.small)
  - `Resources`: `AWS::EC2::Route` with `DependsOn`
  - `Outputs`: `AppURL` using `!Sub http://${WebServerInstance.PublicIp}`
- **Change Set:** `Lab1ChangeSet` — updates security group CidrIP to 0.0.0.0/0

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 1: Set Up Code Editor and Review Template**
   - Opened the Code Editor IDE using the `LabWorkspaceURL`
   - Verified the AWS CLI was available: `aws --version`
   - Opened `lab1.yaml` and reviewed its existing structure:
     - VPC, subnets, internet gateway, security group, and EC2 instance resources
     - Noted which sections were missing (Parameters, Route resource, Outputs)

2. **Task 2: Author Template Additions**
   - **Added Parameters section:**
     ```yaml
     Parameters:
       InstanceType:
         Type: String
         Default: t3.nano
         AllowedValues:
           - t3.nano
           - t3.micro
           - t3.small
         Description: EC2 instance type for the web server
     ```
   - **Added AWS::EC2::Route resource:**
     ```yaml
     PublicRoute:
       Type: AWS::EC2::Route
       DependsOn:
         - VPC
         - AttachGateway
       Properties:
         RouteTableId: !Ref PublicRouteTable
         DestinationCidrBlock: 0.0.0.0/0
         GatewayId: !Ref InternetGateway
     ```
   - **Added Outputs section:**
     ```yaml
     Outputs:
       AppURL:
         Description: URL to access the web application
         Value: !Sub http://${WebServerInstance.PublicIp}
     ```
   - Saved the updated `lab1.yaml`

3. **Task 3: Deploy the Stack via CLI**
   - Deployed the stack specifying the InstanceType parameter:
     ```bash
     aws cloudformation create-stack \
       --stack-name Lab1 \
       --template-body file://lab1.yaml \
       --parameters ParameterKey=InstanceType,ParameterValue=t3.micro \
       --capabilities CAPABILITY_IAM
     ```
   - Monitored stack creation:
     ```bash
     aws cloudformation describe-stacks --stack-name Lab1
     ```
   - Waited for `StackStatus: CREATE_COMPLETE`
   - Viewed the Resources tab — confirmed 18 resources created
   - Retrieved the `AppURL` output — navigated to it and observed a connection timeout (security group blocked inbound traffic)

4. **Task 4: Detect and Investigate Drift**
   - Manually changed the security group's inbound rule **CidrIP to "My IP"** directly in the EC2 console
   - Confirmed `AppURL` was now accessible after the manual change
   - Triggered drift detection from the CloudFormation console
   - Described the drift results:
     ```bash
     aws cloudformation describe-stack-resource-drifts \
       --stack-name Lab1 \
       --stack-resource-drift-status-filters MODIFIED DELETED
     ```
   - Confirmed `InstanceSecurityGroup` showed `MODIFIED` status — the CidrIP change was detected

5. **Task 5: Create and Execute a Change Set**
   - Created `lab1-CS.yaml` as a copy of `lab1.yaml` with `CidrIP: 0.0.0.0/0` set in the security group
   - Created a change set:
     ```bash
     aws cloudformation create-change-set \
       --stack-name Lab1 \
       --change-set-name Lab1ChangeSet \
       --template-body file://lab1-CS.yaml \
       --parameters ParameterKey=InstanceType,ParameterValue=t3.micro
     ```
   - Reviewed the change set in the console — confirmed only `InstanceSecurityGroup` would be modified
   - Executed the change set:
     ```bash
     aws cloudformation execute-change-set \
       --stack-name Lab1 \
       --change-set-name Lab1ChangeSet
     ```
   - Monitored the stack update until `UPDATE_COMPLETE`
   - Verified `AppURL` remained accessible with the security group now managed by CloudFormation at `0.0.0.0/0`

</details>

---

## **6. Findings and Results**

- **Finding 1:** Adding `AllowedValues` to the `InstanceType` parameter enforced valid instance type selection at deploy time, preventing invalid values from being passed to the stack
- **Finding 2:** `DependsOn: [VPC, AttachGateway]` on the Route resource ensured the route was not created before the VPC and gateway attachment existed — resolving a dependency ordering requirement not automatically inferred by CloudFormation
- **Finding 3:** `!Sub http://${WebServerInstance.PublicIp}` dynamically constructed the AppURL output by substituting the instance's public IP at runtime — demonstrating CloudFormation's intrinsic function capability
- **Finding 4:** The initial AppURL showed a timeout because the security group's CidrIP in the template was not set to allow public access — intentional for the drift simulation exercise
- **Finding 5:** `describe-stack-resource-drifts` with `MODIFIED` and `DELETED` status filters precisely returned only the drifted `InstanceSecurityGroup` resource, confirming accurate drift detection
- **Finding 6:** The change set preview clearly showed that only the `InstanceSecurityGroup` would be modified — providing confidence that the update would not affect any other resources
- **Finding 7:** After change set execution, the stack returned to `IN_SYNC` drift status and the AppURL remained accessible — confirming that the change set both remediated drift and enforced the new security group configuration

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand template additions and CLI commands</summary>

### Parameters Section Added

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t3.nano
    AllowedValues:
      - t3.nano
      - t3.micro
      - t3.small
    Description: EC2 instance type for the web server
```

### Route Resource Added

```yaml
PublicRoute:
  Type: AWS::EC2::Route
  DependsOn:
    - VPC
    - AttachGateway
  Properties:
    RouteTableId: !Ref PublicRouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway
```

### Outputs Section Added

```yaml
Outputs:
  AppURL:
    Description: Web application access URL
    Value: !Sub http://${WebServerInstance.PublicIp}
```

### Key CLI Commands

```bash
# Deploy stack
aws cloudformation create-stack \
  --stack-name Lab1 \
  --template-body file://lab1.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t3.micro

# Check stack status
aws cloudformation describe-stacks --stack-name Lab1

# Describe drift results
aws cloudformation describe-stack-resource-drifts \
  --stack-name Lab1 \
  --stack-resource-drift-status-filters MODIFIED DELETED

# Create change set
aws cloudformation create-change-set \
  --stack-name Lab1 \
  --change-set-name Lab1ChangeSet \
  --template-body file://lab1-CS.yaml

# Execute change set
aws cloudformation execute-change-set \
  --stack-name Lab1 \
  --change-set-name Lab1ChangeSet
```

### Stack Summary

| Property | Value |
|----------|-------|
| Stack Name | `Lab1` |
| Template | `lab1.yaml` |
| Instance Type (parameter) | `t3.micro` |
| Total Resources | 18 |
| Drift Detected | `InstanceSecurityGroup — MODIFIED` |
| Change Set | `Lab1ChangeSet` |
| Final Status | `UPDATE_COMPLETE`, `IN_SYNC` |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| AppURL timing out after initial deployment | Recognized the security group intentionally blocked access for the drift simulation — confirmed after manual CidrIP change restored access | Understood the deliberate access restriction as a setup for the drift detection exercise |
| `DependsOn` requirement for the Route resource | Identified that the Route depends on both VPC creation and gateway attachment — added both to `DependsOn` | Route created successfully after dependencies were satisfied |
| Change set modifying unintended resources | Reviewed the change set preview carefully before executing — confirmed only `InstanceSecurityGroup` was in scope | Executed with confidence that no unintended resources would be affected |
| Verifying drift detection CLI output | Used `--stack-resource-drift-status-filters MODIFIED DELETED` to limit output to only drifted resources | Received a clean, actionable output showing only the drifted security group |

---

## **9. Key Takeaways and Lessons Learned**

- **Parameters Enable Reusable Templates:** Adding `AllowedValues` to parameters transforms a single-use template into a reusable artifact that can be deployed with different validated configurations across environments.

- **`DependsOn` for Explicit Dependencies:** CloudFormation automatically infers dependencies through `!Ref` and `!GetAtt`, but some dependencies (like ensuring an internet gateway attachment completes before creating a route) must be declared explicitly with `DependsOn`.

- **Change Sets as Safe Updates:** Creating and reviewing a change set before executing it provides a safety review step that prevents accidental resource recreation or deletion. Change sets should be standard practice for all production stack updates.

- **CLI Deployments Enable Automation:** Using `aws cloudformation create-stack` and `create-change-set` from the CLI enables CloudFormation operations to be scripted into CI/CD pipelines — no console interaction required.

- **Outputs Provide Discoverable Endpoints:** The `Outputs` section makes the AppURL discoverable from the CloudFormation console and retrievable via CLI (`describe-stacks`), enabling downstream automation to consume stack outputs without hardcoding values.

---

## **10. Artifacts and Deliverables**

- **`lab1.yaml`:** Complete CloudFormation template with Parameters, Route resource, and AppURL output
- **`lab1-CS.yaml`:** Updated template for change set execution with corrected security group CidrIP
- **CLI Command Reference:** All `aws cloudformation` commands used throughout the lab
- **Stack Summary:** Resource count, drift findings, and final stack state
- **Change Set Documentation:** Change set name, scope, and execution results

---

## **11. Conclusion**

This lab successfully demonstrated the complete **AWS CloudFormation lifecycle** from template authoring through deployment, drift detection, and change set-based remediation. The addition of a parameterized `InstanceType`, an explicit-dependency Route resource, and a dynamic AppURL output enriched the template with production-standard IaC patterns. CLI-based stack deployment, drift detection using `describe-stack-resource-drifts`, and a reviewed-then-executed change set validated all major CloudFormation operational workflows. These skills form the core competency for infrastructure engineers managing AWS environments through code — ensuring infrastructure is repeatable, auditable, and maintainable across development, staging, and production environments.

---

## **12. References**

- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [CloudFormation Template Anatomy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html)
- [CloudFormation Intrinsic Functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
- [CloudFormation Change Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- [CloudFormation Drift Detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
- [AWS CLI CloudFormation Reference](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Operations
**Status:** Completed ✅
