# Working with the Linux Filesystem

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-Linux%20Administration-blue)
![Tools](https://img.shields.io/badge/Tools-Bash%2C%20Linux%20CLI%2C%20mkdir%2C%20mv%2C%20cp%2C%20rm-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 2 Lab: Working with the Linux Filesystem — Directory Creation, File Management, and Filesystem Organization for AnyCompany

---

## **2. Purpose and Objectives**

The objective of this lab is to develop hands-on proficiency with **Linux filesystem operations** by creating a directory structure, populating it with files, and reorganizing the structure through copy, move, and delete operations. Using a realistic AnyCompany organizational scenario, this lab reinforces the core CLI commands required for all Linux file management tasks.

**Key Learning Outcomes:**
- Create **nested directory structures** using `mkdir` with path specification
- Create empty files using the `touch` command
- Navigate the filesystem using `cd` and verify contents with `ls`
- **Copy directories recursively** with `cp -r`
- **Remove files and empty directories** with `rm` and `rmdir`
- **Move and rename** files and directories with `mv`
- Organize files into a logical department-based directory hierarchy

---

## **3. Frameworks, Standards, and Methodologies**

- **Linux Filesystem Hierarchy Standard (FHS):** Understanding standard Linux directory conventions and best practices for file organization
- **Organizational Directory Modeling:** Mapping business department structures to Linux directory hierarchies
- **Destructive Command Safety:** Understanding the permanence of `rm` and `rmdir` in Linux (no Recycle Bin)
- **Recursive Operations:** Using `-r` flags to apply operations to directories and all contents
- **Idiomatic CLI Patterns:** Combining `mkdir`, `touch`, `cp`, `mv`, `rm`, and `rmdir` in workflows

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Compute:** Amazon EC2 (Linux instance via Session Manager)
- **Shell:** Bash
- **Core Commands Practiced:**
  - `mkdir` — create directories
  - `touch` — create empty files
  - `cd` — change directory
  - `ls` — list directory contents
  - `cp -r` — recursively copy directories
  - `rm` — remove files
  - `rmdir` — remove empty directories
  - `mv` — move or rename files and directories
- **Working Directory:** `/home/ssm-user/AnyCompany/`

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 2: Create the AnyCompany Directory Structure**
   - Created the top-level `AnyCompany` directory and three department subdirectories:
     ```bash
     mkdir -p /home/ssm-user/AnyCompany/Finance
     mkdir /home/ssm-user/AnyCompany/HR
     mkdir /home/ssm-user/AnyCompany/Management
     ```
   - Navigated into each directory and created CSV files using `touch`:
     - `Finance/`: `Budgets.csv`, `Forecasts.csv`, `Invoices.csv`
     - `HR/`: `Employees.csv`, `Recruitment.csv`
     - `Management/`: `Assessments.csv`, `TrialPeriod.csv`, `Projects.csv`
   - Verified the structure with `ls -R /home/ssm-user/AnyCompany/`

2. **Task 3: Reorganize the Directory Structure**
   - **Copied Finance and HR into a new structure:**
     - Used `cp -r Finance HR` to recursively copy the Finance directory into HR
     - Verified Finance contents now existed under `HR/Finance/`
   - **Removed original files from Finance:**
     - Used `rm Finance/Budgets.csv Finance/Forecasts.csv Finance/Invoices.csv`
     - Removed the now-empty Finance directory with `rmdir Finance`
   - **Moved Management into HR:**
     - Used `mv Management HR/Management` to relocate the Management department
   - **Created an Employees subdirectory and moved files:**
     - Created `HR/Employees/` with `mkdir HR/Employees`
     - Moved `Assessments.csv` and `TrialPeriod.csv` into `HR/Employees/`:
       ```bash
       mv HR/Management/Assessments.csv HR/Employees/
       mv HR/Management/TrialPeriod.csv HR/Employees/
       ```
   - **Verified final directory structure:**
     - Ran `ls -R /home/ssm-user/AnyCompany/` to confirm the reorganized layout

</details>

---

## **6. Findings and Results**

- **Finding 1:** `mkdir -p` with a full path created nested directories in a single command, more efficient than creating each level individually
- **Finding 2:** `touch` created empty files instantly without opening a text editor — ideal for scaffolding a file structure before populating content
- **Finding 3:** `cp -r` successfully copied the entire Finance directory tree (including all CSV files) into HR, preserving the subdirectory structure
- **Finding 4:** `rm` permanently deleted the specified files with no confirmation prompt and no recovery option — confirming that destructive operations in Linux require deliberate intent
- **Finding 5:** `rmdir` successfully removed the now-empty Finance directory; it would have failed if any files had remained — confirming `rmdir` is a safe non-recursive deletion method
- **Finding 6:** `mv` relocated the entire Management directory into HR in a single command, renaming/repositioning the path simultaneously
- **Finding 7:** The final AnyCompany directory structure was correctly organized with all files in their intended locations as verified by `ls -R`

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand directory structure and command reference</summary>

### Initial Directory Structure

```
AnyCompany/
├── Finance/
│   ├── Budgets.csv
│   ├── Forecasts.csv
│   └── Invoices.csv
├── HR/
│   ├── Employees.csv
│   └── Recruitment.csv
└── Management/
    ├── Assessments.csv
    ├── TrialPeriod.csv
    └── Projects.csv
```

### Final Directory Structure (After Reorganization)

```
AnyCompany/
└── HR/
    ├── Employees.csv
    ├── Recruitment.csv
    ├── Finance/
    │   ├── Budgets.csv
    │   ├── Forecasts.csv
    │   └── Invoices.csv
    ├── Management/
    │   └── Projects.csv
    └── Employees/
        ├── Assessments.csv
        └── TrialPeriod.csv
```

### Key Commands Reference

| Command | Example | Action |
|---------|---------|--------|
| `mkdir -p` | `mkdir -p AnyCompany/Finance` | Create directory with parent paths |
| `touch` | `touch Finance/Budgets.csv` | Create empty file |
| `cp -r` | `cp -r Finance HR` | Recursively copy directory |
| `rm` | `rm Finance/Budgets.csv` | Delete file permanently |
| `rmdir` | `rmdir Finance` | Remove empty directory |
| `mv` | `mv Management HR/Management` | Move/rename directory |
| `ls -R` | `ls -R AnyCompany/` | List all contents recursively |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| `rmdir` failing because directory still contained files | Used `rm` to delete all files inside first, then ran `rmdir` | Directory removed cleanly without using `rm -r` |
| Ensuring `cp -r` preserved the full directory structure | Verified with `ls -R HR/` after the copy to confirm Finance subdirectory appeared | Confirmed recursive copy preserved all files and structure |
| Moving multiple files to a new directory efficiently | Used separate `mv` commands for each file — could also use wildcard `mv *.csv destination/` | All files successfully moved to the Employees directory |
| Verifying the final structure was correct | Used `ls -R /home/ssm-user/AnyCompany/` to recursively list all contents | Confirmed all files were in their intended locations |

---

## **9. Key Takeaways and Lessons Learned**

- **`rm` is Permanent:** Linux does not have a Recycle Bin. `rm` immediately and irrecoverably deletes files. Developing the habit of double-checking `rm` commands — especially with wildcards — is a critical safety practice.

- **`cp -r` vs. `mv` for Reorganization:** `cp -r` duplicates content while preserving the original; `mv` relocates it. The choice between them depends on whether the original location should retain a copy — understanding this distinction prevents data loss.

- **`rmdir` as a Safety Net:** Preferring `rmdir` over `rm -r` for directory deletion is safer because `rmdir` fails if the directory contains files, preventing accidental bulk deletion.

- **`mkdir -p` for Deep Structures:** Creating multi-level directory paths with `mkdir -p` in a single command is more efficient and readable than sequential `mkdir` calls.

- **Filesystem Organization Matters:** Organizing files into logical department-based directories mirrors real-world server administration where configuration files, logs, and data must be structured for maintainability and access control.

---

## **10. Artifacts and Deliverables**

- **Initial Directory Structure Diagram:** AnyCompany filesystem before reorganization
- **Final Directory Structure Diagram:** AnyCompany filesystem after all reorganization operations
- **Key Commands Reference:** Complete table of filesystem commands with examples
- **Reorganization Workflow:** Step-by-step operations: copy, remove, move, and verify

---

## **11. Conclusion**

This lab successfully demonstrated core Linux filesystem management skills through a realistic organizational directory reorganization scenario. By creating the AnyCompany department structure, populating it with files, and systematically reorganizing it using `cp -r`, `rm`, `rmdir`, and `mv`, all fundamental file management operations were practiced in context. The permanent nature of `rm` was reinforced as a critical operational awareness point. The final verified directory structure confirmed that all reorganization operations executed correctly. These filesystem skills are directly applicable to real-world Linux administration tasks including log management, configuration file organization, deployment directory setup, and server maintenance.

---

## **12. References**

- [Linux Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)
- [GNU Coreutils – File Management Commands](https://www.gnu.org/software/coreutils/manual/)
- [Linux `cp` Command](https://man7.org/linux/man-pages/man1/cp.1.html)
- [Linux `mv` Command](https://man7.org/linux/man-pages/man1/mv.1.html)
- [Linux `rm` Command](https://man7.org/linux/man-pages/man1/rm.1.html)
- [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Operations
**Status:** Completed ✅
