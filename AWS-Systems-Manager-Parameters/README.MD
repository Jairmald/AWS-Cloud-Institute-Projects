# Creating AWS Systems Manager Parameters

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-AWS%20Systems%20Manager%2C%20Parameter%20Store%2C%20CloudFormation-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Creating AWS Systems Manager Parameters — Storing EC2 Configuration Parameters and Integrating with CloudFormation for Dynamic Infrastructure Provisioning

---

## **2. Purpose and Objectives**

The objective of this lab is to gain hands-on experience with **AWS Systems Manager Parameter Store** by creating configuration parameters and consuming them dynamically in a **CloudFormation** stack. This lab demonstrates how centralized parameter management decouples configuration from infrastructure code, enabling reusable, environment-agnostic templates.

**Key Learning Outcomes:**
- Create **String** and **aws:ec2:image** type parameters in AWS Systems Manager Parameter Store
- Understand the **`aws:ec2:image`** parameter type for dynamic AMI ID resolution
- Reference SSM parameters in a **CloudFormation template** using `AWS::SSM::Parameter::Value<>` types
- Deploy a CloudFormation stack that consumes SSM parameters at runtime
- Verify the provisioned EC2 instance matches the parameter-defined instance type

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence — separating configuration from code using Parameter Store
- **Infrastructure as Code (IaC):** Dynamic parameter injection into CloudFormation templates
- **Configuration Management Best Practices:** Centralizing environment-specific values in Parameter Store for reuse across stacks
- **Immutable Infrastructure Patterns:** Using SSM-resolved AMI IDs to ensure consistent, current AMI selection
- **Least Privilege IAM:** CloudFormation service role with scoped SSM read permissions

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Services Used:**
  - AWS Systems Manager — Parameter Store
  - AWS CloudFormation
  - Amazon EC2
- **Parameter Types Created:**
  - `String` — for instance type configuration
  - `aws:ec2:image` — for AMI ID with built-in validation
- **Parameters Created:**
  - `/ec2/instance-type` → `t3.micro`
  - `/ec2/ami-id` → Latest Amazon Linux 2 AMI ID (LatestAmiId)
- **CloudFormation Template:** `lab-ec2.yaml`
- **Resources Deployed:**
  - EC2 instance: `Test-Instance` (t3.micro, Lab VPC, Private Subnet 1)

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 1: Create Parameter Store Parameters**
   - Navigated to **AWS Systems Manager → Parameter Store** in the AWS Console
   - Created the first parameter:
     - **Name:** `/ec2/instance-type`
     - **Type:** `String`
     - **Value:** `t3.micro`
     - **Description:** EC2 instance type for lab deployments
   - Created the second parameter:
     - **Name:** `/ec2/ami-id`
     - **Type:** `aws:ec2:image`
     - **Value:** The `LatestAmiId` value (current Amazon Linux 2 AMI for the region)
     - **Description:** Latest Amazon Linux 2 AMI ID
   - Verified both parameters appeared in the Parameter Store console with correct types and values

2. **Task 2: Deploy CloudFormation Stack Using SSM Parameters**
   - Downloaded the provided `lab-ec2.yaml` CloudFormation template
   - Reviewed the template structure — parameters were declared with type `AWS::SSM::Parameter::Value<String>` and `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` to pull values from Parameter Store at deploy time
   - Navigated to **CloudFormation → Create Stack**
   - Uploaded `lab-ec2.yaml` as the template source
   - In the Parameters section, specified:
     - SSM parameter path: `/ec2/instance-type` for the InstanceType parameter
     - SSM parameter path: `/ec2/ami-id` for the AmiId parameter
   - Selected **Lab VPC** as the target VPC
   - Selected **Private Subnet 1** as the deployment subnet
   - Submitted the stack creation
   - Monitored the **Events** tab until the stack reached `CREATE_COMPLETE`

3. **Verification**
   - Navigated to **EC2 → Instances** and located `Test-Instance`
   - Confirmed the instance type showed **t3.micro** — matching the `/ec2/instance-type` parameter value
   - Confirmed the AMI ID matched the value stored in `/ec2/ami-id`
   - Verified the instance was deployed in the correct VPC and Private Subnet 1

</details>

---

## **6. Findings and Results**

- **Finding 1:** Parameter Store accepted both `String` and `aws:ec2:image` parameter types — the `aws:ec2:image` type provides built-in AMI ID format validation, preventing invalid values from being stored
- **Finding 2:** The `/ec2/instance-type` parameter successfully stored `t3.micro` as a reusable configuration value that could be referenced by multiple CloudFormation stacks without hardcoding
- **Finding 3:** CloudFormation's `AWS::SSM::Parameter::Value<>` type resolved the SSM parameter values at stack creation time — the template itself contained no hardcoded instance type or AMI ID values
- **Finding 4:** The `Test-Instance` EC2 instance was provisioned as `t3.micro` as expected, confirming that the CloudFormation → SSM parameter resolution chain worked correctly end-to-end
- **Finding 5:** Deploying the stack to **Lab VPC** and **Private Subnet 1** demonstrated that VPC and subnet selection at deploy time, combined with SSM parameter injection, makes the template fully environment-agnostic
- **Finding 6:** The `aws:ec2:image` parameter type with `LatestAmiId` ensures that any stack using `/ec2/ami-id` always receives a current, valid AMI — eliminating the risk of deprecated AMI IDs causing stack failures over time

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand configuration and template reference</summary>

### Parameter Store Configuration

| Parameter Name | Type | Value | Purpose |
|---------------|------|-------|---------|
| `/ec2/instance-type` | `String` | `t3.micro` | EC2 instance type |
| `/ec2/ami-id` | `aws:ec2:image` | `ami-xxxxxxxxx` (LatestAmiId) | Amazon Linux 2 AMI |

### CloudFormation Template Parameter Declaration (Excerpt)

```yaml
Parameters:
  InstanceType:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /ec2/instance-type

  AmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /ec2/ami-id

Resources:
  TestInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !Ref AmiId
      SubnetId: !Ref PrivateSubnet1
      Tags:
        - Key: Name
          Value: Test-Instance
```

### Stack Deployment Summary

| Property | Value |
|----------|-------|
| Stack Name | `lab-ec2` |
| Template | `lab-ec2.yaml` |
| VPC | Lab VPC |
| Subnet | Private Subnet 1 |
| Instance Type (resolved) | `t3.micro` |
| Stack Status | `CREATE_COMPLETE` |

### Verification Checklist

| Check | Result |
|-------|--------|
| `/ec2/instance-type` parameter created | ✅ Verified |
| `/ec2/ami-id` parameter created | ✅ Verified |
| CloudFormation stack `CREATE_COMPLETE` | ✅ Verified |
| Test-Instance type = t3.micro | ✅ Verified |
| Instance in correct VPC/Subnet | ✅ Verified |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Understanding `aws:ec2:image` vs. `String` parameter types | The `aws:ec2:image` type validates that the stored value is a valid AMI ID format — `String` accepts any text; confirmed via Parameter Store console tooltips | Used correct type for each parameter; AMI ID validated on creation |
| Locating the LatestAmiId value for the current region | Found the current Amazon Linux 2 AMI ID from the EC2 console's AMI catalog filtered by Amazon Linux 2 | Stored the correct region-specific AMI ID in the parameter |
| CloudFormation parameter referencing SSM paths | Template used `AWS::SSM::Parameter::Value<String>` type with the SSM path as the Default value — CloudFormation resolves it at deploy time | Parameters resolved correctly during stack creation |

---

## **9. Key Takeaways and Lessons Learned**

- **Parameter Store Decouples Config from Code:** Storing instance types and AMI IDs in Parameter Store means the CloudFormation template never needs to be modified when these values change — only the parameter value is updated, and all stacks referencing it benefit automatically.

- **`aws:ec2:image` Type Adds Validation:** Using the specialized `aws:ec2:image` parameter type rather than a plain `String` for AMI IDs adds a layer of validation that prevents storing malformed or invalid AMI references.

- **SSM Parameters as Infrastructure Contracts:** Parameter Store paths like `/ec2/instance-type` act as contracts between teams — the infrastructure team defines acceptable values, and deployment templates consume them without needing to know the actual values.

- **Dynamic Parameter Resolution at Deploy Time:** CloudFormation resolves SSM parameter values at stack creation time, not at template creation time. This means updating a Parameter Store value and redeploying the stack automatically picks up the new value.

---

## **10. Artifacts and Deliverables**

- **Parameter Store Configuration:** Two parameters created (`/ec2/instance-type`, `/ec2/ami-id`) with correct types and values
- **`lab-ec2.yaml`:** CloudFormation template consuming SSM parameters for dynamic EC2 provisioning
- **Deployment Summary:** Stack creation details including VPC, subnet, and resolved parameter values
- **Verification Checklist:** Confirmed instance type and placement matching parameter-defined values

---

## **11. Conclusion**

This lab successfully demonstrated the integration of **AWS Systems Manager Parameter Store** with **CloudFormation** for dynamic, configuration-driven infrastructure provisioning. By creating String and `aws:ec2:image` parameters and referencing them in a CloudFormation template, the EC2 instance was provisioned with the correct type and AMI without any hardcoded values in the template. This pattern is foundational for production IaC workflows where environment-specific configuration (instance types, AMI IDs, VPC IDs) must be managed centrally and consumed consistently across multiple stacks and environments.

---

## **12. References**

- [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [CloudFormation SSM Parameter Types](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html#aws-ssm-parameter-types)
- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [Amazon EC2 AMI Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [AWS Well-Architected Framework – Operational Excellence](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Operations
**Status:** Completed ✅
