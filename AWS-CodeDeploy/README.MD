# Introduction to AWS CodeDeploy

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20DevOps%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-AWS%20CodeDeploy%2C%20S3%2C%20EC2%2C%20AWS%20CLI-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Introduction to AWS CodeDeploy — Deploying a Windows Service Application to EC2 Instances Using AppSpec, Deployment Groups, and S3 Artifact Storage

---

## **2. Purpose and Objectives**

The objective of this lab is to gain hands-on experience with **AWS CodeDeploy** by deploying a Windows-based heartbeat service application to EC2 instances. The lab covers creating a CodeDeploy application and deployment group, reviewing an `appspec.yml` deployment specification, packaging and uploading artifacts to S3, creating a deployment, and verifying the deployed service. A redeployment with an updated application version validates the update workflow.

**Key Learning Outcomes:**
- Verify **CodeDeploy agent** status on target EC2 Windows instances
- Create a **CodeDeploy application** and **deployment group** with EC2 instance tag targeting
- Understand and review **`appspec.yml`** structure including `files` and `hooks` sections
- Package and push an application revision to S3 using `aws deploy push`
- Create and monitor a **CodeDeploy deployment** through the console
- Verify a deployed Windows service is `Running` and producing expected output
- Perform a **redeployment** with an updated application version

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence — automated application deployments via CodeDeploy
- **Continuous Delivery Practices:** Packaging, versioning, and deploying application revisions through a managed pipeline
- **AppSpec Deployment Specification:** Understanding the lifecycle event hook model for controlling deployment behavior
- **Blue/Green and In-Place Deployment Models:** Understanding `CodeDeployDefault.AllAtOnce` deployment configuration
- **IAM Service Roles:** `CodeDeployServiceRole` with `AWSCodeDeployRole` policy for EC2 fleet management

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Deployment Service:** AWS CodeDeploy
- **Target Compute:** 2x Amazon EC2 Windows instances (`HeartBeat-Instance` tagged)
- **Artifact Storage:** Amazon S3
- **Shell/CLI:** AWS CLI (`aws deploy push`, `aws deploy create-deployment`)
- **CodeDeploy Agent:** Verified running on Windows instances via PowerShell
- **IAM Role:** `CodeDeployServiceRole` (with `AWSCodeDeployRole` managed policy)
- **Application Name:** `HeartBeatProduction-App`
- **Deployment Group:** `HeartBeatProduction-App-Group`
- **Deployment Config:** `CodeDeployDefault.AllAtOnce`
- **AppSpec File:** `appspec.yml`
- **S3 Bucket:** `heartbeat-codedeploy-artifacts-{initials}-{postal}`

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 1: Verify Prerequisites**
   - Located the two `HeartBeat-Instance` EC2 Windows instances in the console
   - Confirmed both instances were in `Running` state
   - Connected to one instance and verified the **CodeDeploy agent** service was running:
     ```powershell
     Get-Service -Name codedeployagent
     ```
   - Confirmed status showed `Running`
   - Verified the `CodeDeployServiceRole` IAM role existed and had the `AWSCodeDeployRole` managed policy attached

2. **Task 2: Create CodeDeploy Application and Deployment Group**
   - Navigated to **CodeDeploy → Applications → Create Application**
   - Set:
     - **Application name:** `HeartBeatProduction-App`
     - **Compute platform:** EC2/On-premises
   - Created a **Deployment Group** named `HeartBeatProduction-App-Group`:
     - **Service role:** `CodeDeployServiceRole`
     - **Deployment type:** In-place
     - **Environment configuration:** Amazon EC2 instances
     - **Tag group:** Key=`Name`, Value=`HeartBeat-Instance`
     - **Deployment configuration:** `CodeDeployDefault.AllAtOnce`
     - **Load balancer:** Disabled (for this lab)

3. **Task 3: Review the AppSpec File**
   - Opened and reviewed `appspec.yml` from the provided application package:
     ```yaml
     version: 0.0
     os: windows
     files:
       - source: HeartbeatService
         destination: c:\HeartbeatService
     hooks:
       ApplicationStop:
         - location: scripts/uninstall.ps1
           timeout: 60
       AfterInstall:
         - location: scripts/install.ps1
           timeout: 60
         - location: scripts/copywintail.ps1
           timeout: 60
     ```
   - Confirmed the `files` section copies the service binary to `c:\HeartbeatService`
   - Confirmed the `hooks` section runs PowerShell scripts at `ApplicationStop` (uninstall) and `AfterInstall` (install + copy wintail) lifecycle events

4. **Task 4: Create S3 Bucket, Push Artifact, and Deploy**
   - Created the S3 artifact bucket:
     ```bash
     aws s3 mb s3://heartbeat-codedeploy-artifacts-{initials}-{postal}
     ```
   - Packaged and pushed the application revision to S3:
     ```bash
     aws deploy push \
       --application-name HeartBeatProduction-App \
       --s3-location s3://heartbeat-codedeploy-artifacts-{initials}-{postal}/HeartBeat.zip \
       --source .
     ```
   - Created a deployment:
     ```bash
     aws deploy create-deployment \
       --application-name HeartBeatProduction-App \
       --deployment-group-name HeartBeatProduction-App-Group \
       --s3-location bucket=heartbeat-codedeploy-artifacts-{initials}-{postal},key=HeartBeat.zip,bundleType=zip
     ```
   - Monitored the deployment in the CodeDeploy console until all instances showed **Succeeded**
   - Verified `AWSHeartbeat` Windows service was `Running`
   - Confirmed the service log showed expected output messages

5. **Task 5: Redeploy Updated Application**
   - Used `Updated-HeartBeat-App` revision package for the redeployment
   - Pushed the updated revision to S3 and created a new deployment
   - Monitored the redeployment and confirmed success
   - Verified the service log now showed **"Updated!!!"** messages confirming the new version was deployed

</details>

---

## **6. Findings and Results**

- **Finding 1:** Both `HeartBeat-Instance` EC2 Windows instances had the CodeDeploy agent installed and in `Running` state — a prerequisite that must be verified before any deployment attempt
- **Finding 2:** Tag-based instance targeting (`Name=HeartBeat-Instance`) in the deployment group allowed CodeDeploy to automatically discover and deploy to all matching instances without hardcoding instance IDs
- **Finding 3:** The `appspec.yml` lifecycle hook model provided precise control over deployment behavior: `ApplicationStop` ran the uninstall script first, then `AfterInstall` ran install and configuration scripts — ensuring clean installation on each deploy
- **Finding 4:** `aws deploy push` packaged the local application directory and uploaded it as a versioned ZIP artifact to S3 in a single command — abstracting the manual packaging and upload steps
- **Finding 5:** `aws deploy create-deployment` initiated the deployment using the S3-stored artifact; the CodeDeploy console's deployment timeline showed each lifecycle event progressing in real time across both instances
- **Finding 6:** The `AWSHeartbeat` Windows service was confirmed `Running` and producing expected log output after deployment, validating the end-to-end deployment pipeline
- **Finding 7:** The redeployment with the updated revision successfully replaced the running service, with log output changing to show "Updated!!!" messages — confirming CodeDeploy handles application updates cleanly via the hook lifecycle

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand AppSpec reference and deployment summary</summary>

### AppSpec.yml Structure

```yaml
version: 0.0
os: windows
files:
  - source: HeartbeatService
    destination: c:\HeartbeatService
hooks:
  ApplicationStop:
    - location: scripts/uninstall.ps1
      timeout: 60
  AfterInstall:
    - location: scripts/install.ps1
      timeout: 60
    - location: scripts/copywintail.ps1
      timeout: 60
```

### CodeDeploy Lifecycle Event Order

```
1. ApplicationStop     → Run uninstall.ps1 (stop/remove old service)
2. DownloadBundle      → CodeDeploy downloads artifact from S3
3. BeforeInstall       → (not used in this lab)
4. Install             → Copy files per appspec files section
5. AfterInstall        → Run install.ps1 + copywintail.ps1
6. ApplicationStart    → (not used in this lab)
7. ValidateService     → (not used in this lab)
```

### Deployment Configuration Summary

| Property | Value |
|----------|-------|
| Application Name | `HeartBeatProduction-App` |
| Deployment Group | `HeartBeatProduction-App-Group` |
| Target Instances | EC2 tag: `Name=HeartBeat-Instance` (2 instances) |
| Deployment Config | `CodeDeployDefault.AllAtOnce` |
| S3 Artifact Bucket | `heartbeat-codedeploy-artifacts-{initials}-{postal}` |
| Service Role | `CodeDeployServiceRole` |
| Initial Deploy | ✅ Succeeded |
| Redeployment | ✅ Succeeded — "Updated!!!" log confirmed |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Verifying CodeDeploy agent status on Windows | Used PowerShell `Get-Service -Name codedeployagent` to check service status | Confirmed `Running` state on both target instances |
| Understanding AppSpec hooks execution order | Reviewed the CodeDeploy lifecycle documentation to confirm `ApplicationStop` runs before `AfterInstall` | Understood why uninstall runs first — ensuring a clean state before reinstallation |
| `aws deploy push` requiring correct working directory | Ran the command from the directory containing `appspec.yml` and the application files | Artifact packaged correctly with all required files |
| Confirming deployment success beyond the console | Checked Windows Service Manager and the heartbeat log output directly on the instance | Service confirmed Running with correct log messages |

---

## **9. Key Takeaways and Lessons Learned**

- **CodeDeploy Abstracts Deployment Complexity:** Rather than SSH-ing into instances and running scripts manually, CodeDeploy orchestrates the entire deployment lifecycle across multiple instances simultaneously — a critical capability for fleet deployments at scale.

- **AppSpec is the Deployment Contract:** The `appspec.yml` file defines exactly what gets copied where, and which scripts run at which lifecycle events. Understanding its structure is essential for customizing deployment behavior for any application type.

- **Tag-Based Targeting Enables Dynamic Fleets:** By targeting instances with tags rather than IDs, the deployment group automatically includes any new instance tagged `Name=HeartBeat-Instance` — enabling fleet scaling without deployment configuration changes.

- **`aws deploy push` Simplifies Artifact Management:** The single command packages, uploads, and registers the revision with CodeDeploy — reducing the error surface compared to manual S3 upload and registration steps.

- **Redeployment Validates Update Workflow:** Testing redeployment with an updated revision confirmed that CodeDeploy handles version updates cleanly through the hook lifecycle — the old service was stopped before the new version was installed.

---

## **10. Artifacts and Deliverables**

- **`HeartBeatProduction-App` CodeDeploy Application:** Created with deployment group targeting HeartBeat-Instance tagged EC2 instances
- **`appspec.yml` Reference:** Documented files and hooks sections with lifecycle event explanations
- **S3 Artifact Bucket:** Created and populated with HeartBeat.zip application revision
- **Deployment Summary:** Initial and updated deployments confirmed successful with service verification
- **Lifecycle Event Order Reference:** Sequential deployment lifecycle phases with script assignments

---

## **11. Conclusion**

This lab successfully demonstrated the end-to-end **AWS CodeDeploy** workflow for deploying a Windows service application to a fleet of EC2 instances. From verifying agent prerequisites through creating the application and deployment group, reviewing the `appspec.yml` deployment specification, pushing artifacts to S3, creating deployments, and confirming service status, all components of a production-style deployment pipeline were exercised. The redeployment test validated that CodeDeploy handles application updates correctly through its lifecycle hook model. These skills form the foundation for building fully automated CI/CD pipelines where CodeCommit, CodeBuild, and CodeDeploy work together to deliver application changes from source code to running production services.

---

## **12. References**

- [AWS CodeDeploy Documentation](https://docs.aws.amazon.com/codedeploy/)
- [AppSpec File Reference for EC2](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)
- [CodeDeploy Lifecycle Events](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html)
- [AWS Deploy CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/deploy/)
- [CodeDeploy Agent Installation](https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-windows.html)
- [AWS Well-Architected Framework – Operational Excellence](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Operations
**Status:** Completed ✅
