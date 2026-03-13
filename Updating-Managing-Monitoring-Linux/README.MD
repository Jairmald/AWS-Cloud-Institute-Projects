# Updating, Managing, and Monitoring Linux

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-Linux%20Administration-blue)
![Tools](https://img.shields.io/badge/Tools-dnf%2C%20systemctl%2C%20top%2C%20CloudWatch%2C%20httpd-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 2 Lab 4: Updating, Managing, and Monitoring Linux — Package Management with dnf, Web Server Deployment, System Resource Monitoring, and Package History

---

## **2. Purpose and Objectives**

The objective of this lab is to develop proficiency with **Linux package management** using `dnf`, deploy and manage a web server (`httpd`), monitor system resource utilization using `top` and **Amazon CloudWatch**, and use `dnf history` to audit and roll back package changes. This lab bridges local Linux administration with cloud-native monitoring integration.

**Key Learning Outcomes:**
- Use `dnf` to **check for, apply security updates**, and upgrade packages
- **Install and start `httpd`** (Apache Web Server) and verify it is running
- Deploy a static web page to the Apache document root and access it via public IP
- Use the **`top`** command to observe CPU and memory utilization in real time
- Simulate CPU load with a stress script and observe the spike in both `top` and **CloudWatch**
- Use `dnf history` to **audit package installation events** and perform a package undo/rollback

---

## **3. Frameworks, Standards, and Methodologies**

- **Linux Package Management:** `dnf` as the standard package manager for RHEL-based distributions
- **Security Patch Management:** Applying security-only updates as a best practice before full upgrades
- **Service Lifecycle Management:** Using `systemctl` to start, stop, enable, and check service status
- **System Performance Monitoring:** Using `top` for real-time process and resource monitoring
- **Cloud-Native Monitoring Integration:** Leveraging Amazon CloudWatch for EC2 CPU metrics alongside local tools
- **Package Rollback and Audit:** Using `dnf history` for change tracking and reversibility

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Compute:** Amazon EC2 (Linux — RHEL-based)
- **Package Manager:** `dnf` (Dandified YUM)
- **Web Server:** Apache HTTP Server (`httpd`)
- **Monitoring Tools:**
  - `top` (local real-time process monitor)
  - Amazon CloudWatch (EC2 CPU metrics dashboard)
- **Shell:** Bash
- **Stress Tool:** `stress.sh` (provided lab script for CPU load simulation)
- **Key Commands:**
  - `dnf -y check-update`
  - `sudo dnf update --security`
  - `sudo dnf upgrade -y`
  - `sudo dnf install httpd`
  - `rpm -q httpd`
  - `systemctl start/status httpd.service`
  - `top`
  - `sudo dnf history list`
  - `sudo dnf history info <ID>`
  - `sudo dnf -y history undo <ID>`

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 2: Check and Apply Updates**
   - Ran `dnf -y check-update` to list all available package updates without installing them
   - Applied security-only updates with:
     ```bash
     sudo dnf update --security
     ```
   - Confirmed security patches were applied (or system was already up to date)
   - Tested network connectivity with `curl https://aws.amazon.com` to confirm outbound access
   - Performed a full system upgrade with:
     ```bash
     sudo dnf upgrade -y
     ```
   - Verified upgrade completed without errors

2. **Task 3: Install and Configure Apache Web Server**
   - Installed the `httpd` package:
     ```bash
     sudo dnf install httpd
     ```
   - Verified installation with:
     ```bash
     rpm -q httpd
     ```
   - Started the Apache service:
     ```bash
     systemctl start httpd.service
     ```
   - Checked service status:
     ```bash
     systemctl status httpd.service
     ```
   - Confirmed status showed `Active: active (running)`
   - Copied the provided `index.html` to the Apache document root:
     ```bash
     sudo cp index.html /var/www/html/
     ```
   - Retrieved the EC2 instance's Public IPv4 address from the console
   - Opened a browser and navigated to `http://<Public-IPv4>` to verify the web page loaded

3. **Task 4: Monitor System Resources with top and CloudWatch**
   - Launched `top` in the terminal to observe live process and resource data
   - Noted CPU, memory, and load average metrics for the idle system
   - Ran the CPU stress script in the background:
     ```bash
     sh stress.sh &
     ```
   - Observed the CPU utilization spike in `top` — idle % dropped significantly
   - Switched to the **Amazon CloudWatch EC2 dashboard**:
     - Navigated to CloudWatch → Metrics → EC2 → Per-Instance Metrics
     - Selected the CPUUtilization metric for the lab instance
     - Enabled **Live data** override and set period to **30 seconds**
     - Observed the CPU spike reflected in the CloudWatch graph
   - Allowed the stress script to complete and observed CPU return to baseline

4. **Task 5: Audit and Roll Back Package Changes with dnf history**
   - Listed the dnf transaction history:
     ```bash
     sudo dnf history list
     ```
   - Identified the transaction ID for the `httpd` installation
   - Viewed detailed information for that transaction:
     ```bash
     sudo dnf history info <ID>
     ```
   - Confirmed the transaction showed `httpd` as the installed package
   - Rolled back (undid) the httpd installation:
     ```bash
     sudo dnf -y history undo <ID>
     ```
   - Verified `httpd` was removed by checking `rpm -q httpd` — returned "package not installed"

</details>

---

## **6. Findings and Results**

- **Finding 1:** `dnf check-update` provided a complete list of available updates without making changes — a safe first step for auditing the update state of any Linux system before applying patches
- **Finding 2:** `dnf update --security` applied only security-classified patches, demonstrating a targeted update strategy that reduces risk compared to a full unfiltered upgrade
- **Finding 3:** `rpm -q httpd` confirmed successful package installation by returning the exact installed package version — a reliable post-install verification method
- **Finding 4:** `systemctl status httpd.service` showed `Active: active (running)` immediately after starting the service, confirming Apache was correctly installed and ready to serve requests
- **Finding 5:** The web page loaded successfully at the instance's public IPv4 address, confirming Apache was correctly bound to port 80 and the `index.html` was in the correct document root
- **Finding 6:** The CPU stress script caused a dramatic spike visible in both `top` (immediate, sub-second refresh) and CloudWatch (30-second metric period with live data enabled) — demonstrating complementary monitoring at different granularities
- **Finding 7:** `dnf history undo <ID>` cleanly removed the `httpd` package installation, confirming that dnf maintains a reversible transaction log — a critical capability for change management and incident response

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand command reference and monitoring summary</summary>

### dnf Package Management Commands

| Command | Purpose |
|---------|---------|
| `dnf -y check-update` | List available updates without installing |
| `sudo dnf update --security` | Apply security patches only |
| `sudo dnf upgrade -y` | Full system package upgrade |
| `sudo dnf install httpd` | Install Apache web server |
| `rpm -q httpd` | Verify package installation |
| `sudo dnf history list` | List all dnf transaction history |
| `sudo dnf history info <ID>` | Show details for a specific transaction |
| `sudo dnf -y history undo <ID>` | Undo/rollback a specific transaction |

### systemctl Service Management

| Command | Purpose |
|---------|---------|
| `systemctl start httpd.service` | Start the Apache service |
| `systemctl status httpd.service` | Check current service status |
| `systemctl stop httpd.service` | Stop the service |
| `systemctl enable httpd.service` | Enable service to start on boot |

### top Output Key Fields

| Field | Meaning |
|-------|---------|
| `%us` | User-space CPU usage |
| `%sy` | Kernel/system CPU usage |
| `%id` | CPU idle percentage |
| `MiB Mem` | Total and used memory |
| `load average` | 1, 5, 15 minute load averages |

### CloudWatch Monitoring Configuration

| Setting | Value |
|---------|-------|
| Metric | CPUUtilization |
| Namespace | AWS/EC2 |
| Period | 30 seconds |
| Live Data | Enabled |
| Dashboard | EC2 per-instance metrics |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Web page not loading after httpd install | Confirmed the security group allowed inbound HTTP (port 80) from the internet | Page loaded correctly after verifying the EC2 security group inbound rules |
| CPU spike not immediately visible in CloudWatch | Enabled **Live data** override and set the period to 30 seconds for near-real-time visibility | Observed CPU spike within one CloudWatch data point after stress script execution |
| Identifying the correct dnf history ID for undo | Used `dnf history list` to find the transaction ID, then `dnf history info <ID>` to confirm it was the httpd install before undoing | Correctly identified and rolled back only the httpd transaction |
| `stress.sh` running in foreground blocking the terminal | Appended `&` to run the script in the background (`sh stress.sh &`) | Terminal remained usable while the stress script executed in background |

---

## **9. Key Takeaways and Lessons Learned**

- **Security Updates First:** Applying `dnf update --security` before a full upgrade is a disciplined approach — it addresses known vulnerabilities immediately without the broader risk surface of a full package upgrade.

- **`systemctl` is the Service Management Standard:** On modern systemd-based Linux distributions, `systemctl` manages all service lifecycle operations. Understanding `start`, `stop`, `status`, and `enable` is essential for any server administration role.

- **`top` and CloudWatch are Complementary:** `top` provides sub-second local visibility into individual processes; CloudWatch provides historical, aggregated, cross-instance visibility with alarming capability. Production monitoring requires both layers.

- **`dnf history` Enables Rollback:** The dnf transaction log provides a full audit trail of every package change and supports undo operations. This capability is critical for change management processes where installations must be reversible.

- **CPU Load Testing Validates Monitoring:** Running a stress script and observing the spike in both local and cloud monitoring tools confirms the end-to-end observability pipeline works correctly — a validation step that should be standard practice when setting up monitoring.

---

## **10. Artifacts and Deliverables**

- **dnf Command Reference:** Complete package management command table with purposes
- **systemctl Service Management Reference:** Key service lifecycle commands
- **top Field Reference:** Guide to reading the top output for CPU and memory monitoring
- **CloudWatch Configuration Details:** Settings used for near-real-time CPU monitoring
- **Package Rollback Workflow:** Step-by-step dnf history undo process

---

## **11. Conclusion**

This lab successfully demonstrated a complete Linux system administration workflow: applying security patches, installing and deploying a web server, monitoring CPU utilization through both local (`top`) and cloud-native (CloudWatch) tools, and rolling back package changes using `dnf history`. The end-to-end flow from security patching through service deployment, load testing, and rollback represents the core operational skills required for maintaining Linux servers in an AWS environment. The integration of CloudWatch with local monitoring tools demonstrated how cloud-native observability extends and complements traditional system administration capabilities.

---

## **12. References**

- [DNF Package Manager Documentation](https://dnf.readthedocs.io/)
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
- [Linux `top` Command](https://man7.org/linux/man-pages/man1/top.1.html)
- [systemd systemctl Reference](https://man7.org/linux/man-pages/man1/systemctl.1.html)
- [Amazon CloudWatch EC2 Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/viewing_metrics_with_cloudwatch.html)
- [RPM Package Manager](https://rpm.org/documentation.html)

---

**Lab Completed:** 2026
**Course:** AWS Cloud Operations
**Status:** Completed ✅
