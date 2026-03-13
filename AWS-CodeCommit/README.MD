# Working with AWS CodeCommit

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20DevOps%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-AWS%20CodeCommit%2C%20Git%2C%20AWS%20CLI-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Working with AWS CodeCommit — Creating a Git Repository, Configuring Credentials, Committing Files, and Pushing to a Managed Source Control Service

---

## **2. Purpose and Objectives**

The objective of this lab is to gain hands-on experience with **AWS CodeCommit**, Amazon's fully managed source control service, by creating a repository, configuring Git with the HTTPS credential helper, cloning the repository to an EC2 instance, committing files, and pushing changes to the remote. This lab establishes foundational DevOps version control skills within the AWS ecosystem.

**Key Learning Outcomes:**
- Create a **CodeCommit repository** from the AWS Management Console
- Configure **Git credential helper** and `UseHttpPath` for HTTPS authentication to CodeCommit
- **Clone a CodeCommit repository** to an EC2 Linux instance using HTTPS
- Create files, stage them with `git add`, commit with `git commit`, and verify with `git log`
- **Push commits** to the remote CodeCommit repository with `git push -u origin main`
- Verify pushed files appear in the **CodeCommit console**

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS DevOps Best Practices:** Managed source control as the foundation of CI/CD pipelines
- **Git Workflow Standards:** Stage → Commit → Push workflow for version-controlled development
- **AWS IAM-Based Git Authentication:** Using the AWS credential helper instead of username/password for CodeCommit
- **HTTPS Repository Access:** Configuring Git for secure HTTPS-based CodeCommit connectivity
- **Repository Hygiene:** Meaningful commit messages and verified push confirmation

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Source Control Service:** AWS CodeCommit
- **Version Control Tool:** Git
- **Package Manager:** `yum` (for Git installation)
- **Shell:** Bash (on EC2 Linux instance via Session Manager)
- **Authentication:** AWS credential helper (`aws codecommit credential-helper`)
- **Repository Name:** `My-First-Repo`
- **Branch:** `main`
- **Files Committed:** `cat.txt`, `dog.txt`

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 1: Create the CodeCommit Repository**
   - Navigated to **AWS CodeCommit** in the AWS Management Console
   - Selected **Create repository**
   - Named the repository: `My-First-Repo`
   - Added an optional description
   - Clicked **Create** and confirmed the repository was created
   - Noted the HTTPS clone URL from the repository's connection details

2. **Task 2: Install Git and Configure Credentials**
   - Connected to the EC2 instance via Session Manager
   - Installed Git using yum:
     ```bash
     sudo yum install -y git
     ```
   - Verified installation with `git --version`
   - Configured the AWS credential helper for CodeCommit HTTPS authentication:
     ```bash
     git config --global credential.helper '!aws codecommit credential-helper $@'
     git config --global credential.UseHttpPath true
     ```
   - Verified configuration was saved:
     ```bash
     git config --list
     ```
   - Confirmed `credential.helper` and `credential.UseHttpPath=true` appeared in the output

3. **Task 3: Clone and Create Files**
   - Cloned the CodeCommit repository using the HTTPS URL:
     ```bash
     git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/My-First-Repo
     ```
   - Navigated into the cloned directory:
     ```bash
     cd My-First-Repo
     ```
   - Created two new files:
     ```bash
     echo "Information about cats" > cat.txt
     echo "Information about dogs" > dog.txt
     ```
   - Staged both files:
     ```bash
     git add cat.txt dog.txt
     ```
   - Checked staging status:
     ```bash
     git status
     ```
   - Confirmed both files showed as "Changes to be committed"

4. **Task 4: Commit and Push**
   - Committed the staged files with a descriptive message:
     ```bash
     git commit -m "Added cat.txt and dog.txt"
     ```
   - Verified the commit was recorded:
     ```bash
     git log
     ```
   - Confirmed the commit showed the correct message, author, and timestamp
   - Pushed the commit to the remote CodeCommit repository:
     ```bash
     git push -u origin main
     ```
   - Verified the push succeeded (no error messages)
   - Navigated to the **CodeCommit console → My-First-Repo → Code** tab
   - Confirmed `cat.txt` and `dog.txt` appeared in the repository browser

</details>

---

## **6. Findings and Results**

- **Finding 1:** The CodeCommit repository was created instantly from the console with no infrastructure provisioning required — the service is fully managed and immediately available for use
- **Finding 2:** The AWS credential helper configuration (`credential.helper` + `UseHttpPath`) enabled seamless HTTPS authentication using the EC2 instance's IAM role credentials — no passwords or access keys required
- **Finding 3:** `git clone` successfully retrieved the (initially empty) repository, creating a local working copy on the EC2 instance with the remote origin pre-configured
- **Finding 4:** `git status` after `git add` showed both files in the "Changes to be committed" state, confirming correct staging before commit
- **Finding 5:** `git log` displayed the commit with the correct message "Added cat.txt and dog.txt", author identity, and ISO timestamp — confirming the commit was recorded in the local repository history
- **Finding 6:** `git push -u origin main` successfully uploaded the commit to CodeCommit; the `-u` flag set the upstream tracking reference, enabling future `git push` without specifying the remote
- **Finding 7:** Both `cat.txt` and `dog.txt` appeared in the CodeCommit console repository browser after the push, confirming end-to-end Git workflow completion

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand Git workflow reference</summary>

### Complete Git Workflow

```bash
# 1. Install Git
sudo yum install -y git

# 2. Configure credential helper
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true

# 3. Clone repository
git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/My-First-Repo

# 4. Create files
cd My-First-Repo
echo "Information about cats" > cat.txt
echo "Information about dogs" > dog.txt

# 5. Stage files
git add cat.txt dog.txt
git status

# 6. Commit
git commit -m "Added cat.txt and dog.txt"
git log

# 7. Push
git push -u origin main
```

### Git Stage → Commit → Push Flow

```
Working Directory     Staging Area      Local Repository      Remote (CodeCommit)
       │                   │                   │                      │
  [cat.txt]                │                   │                      │
  [dog.txt]                │                   │                      │
       │                   │                   │                      │
       ├── git add ───────►│                   │                      │
       │                   │                   │                      │
       │               [cat.txt]               │                      │
       │               [dog.txt]               │                      │
       │                   │                   │                      │
       │                   ├── git commit ─────►│                      │
       │                   │                   │                      │
       │                   │              [commit: "Added             │
       │                   │               cat.txt and dog.txt"]     │
       │                   │                   │                      │
       │                   │                   ├── git push ─────────►│
```

### Repository Summary

| Item | Value |
|------|-------|
| Repository Name | `My-First-Repo` |
| HTTPS Clone URL | `https://git-codecommit.<region>.amazonaws.com/v1/repos/My-First-Repo` |
| Branch | `main` |
| Files Committed | `cat.txt`, `dog.txt` |
| Commit Message | "Added cat.txt and dog.txt" |
| Push Status | ✅ Success |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Authentication error when cloning without credential helper | Configured the AWS credential helper with `UseHttpPath=true` before attempting the clone | HTTPS authentication succeeded using IAM role credentials transparently |
| `git push` failing without upstream set | Used `git push -u origin main` on the first push to set the upstream tracking branch | Subsequent pushes can use bare `git push` without specifying remote and branch |
| Verifying push was successful beyond the terminal | Navigated to the CodeCommit console and confirmed files appeared in the Code browser | Visual confirmation that remote repository contains the pushed files |
| Empty initial clone with no default branch | Made the first commit locally and pushed to `main` to establish the branch | Remote repository initialized with `main` branch and first commit |

---

## **9. Key Takeaways and Lessons Learned**

- **Credential Helper Simplifies AWS Git Auth:** The AWS CodeCommit credential helper eliminates manual token management by transparently using IAM credentials. This is the correct production pattern — no passwords stored in Git config.

- **`UseHttpPath=true` is Required for CodeCommit:** Without this setting, the credential helper may not correctly distinguish between multiple CodeCommit repositories. It should always be set when configuring the CodeCommit credential helper.

- **Stage → Commit → Push is a Mental Model:** The three-step Git workflow (add to staging, commit to local repo, push to remote) represents distinct checkpoints. Each step is deliberate — staging allows review before committing, and committing allows local history before sharing.

- **CodeCommit is Git-Compatible:** Any standard Git workflow, tooling, or IDE integration works with CodeCommit. Switching from GitHub or GitLab to CodeCommit requires only a URL change and credential helper configuration.

- **`git log` for Commit Verification:** Always verifying commits with `git log` before pushing ensures the commit message, author, and file set are correct — a professional habit that prevents pushing incorrect or incomplete work.

---

## **10. Artifacts and Deliverables**

- **`My-First-Repo` CodeCommit Repository:** Created with `main` branch containing `cat.txt` and `dog.txt`
- **Git Configuration Reference:** Credential helper setup commands with `UseHttpPath` setting
- **Complete Git Workflow Script:** All commands from installation through push in sequence
- **Stage → Commit → Push Diagram:** Visual representation of Git's three-stage model

---

## **11. Conclusion**

This lab successfully demonstrated the complete workflow for using **AWS CodeCommit** as a managed Git source control service. From repository creation through credential helper configuration, cloning, file creation, staging, committing, and pushing, every step of the standard Git workflow was validated in an AWS-native context. The credential helper configuration eliminated manual authentication overhead, and the CodeCommit console provided immediate visual confirmation of the pushed content. These skills directly support all DevOps and CI/CD pipeline workflows where CodeCommit serves as the source of truth for application and infrastructure code.

---

## **12. References**

- [AWS CodeCommit Documentation](https://docs.aws.amazon.com/codecommit/)
- [CodeCommit HTTPS Git Credentials Setup](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-https-unixes.html)
- [Git Official Documentation](https://git-scm.com/doc)
- [AWS CLI CodeCommit Reference](https://docs.aws.amazon.com/cli/latest/reference/codecommit/)
- [Git Credential Helper for CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-https-unixes.html#setting-up-https-unixes-credential-helper)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Operations
**Status:** Completed ✅
