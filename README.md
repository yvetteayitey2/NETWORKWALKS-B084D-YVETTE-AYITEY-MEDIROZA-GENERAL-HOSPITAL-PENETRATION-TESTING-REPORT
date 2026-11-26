# 🏥 Mediroza General Hospital - Penetration Testing Project

![Security Assessment](https://img.shields.io/badge/Security-Penetration%20Test-red)
![Status](https://img.shields.io/badge/Status-Completed-success)
![Client](https://img.shields.io/badge/Client-MediRoza%20Hospital-blue)

| Project Detail | Information |
| :--- | :--- |
| **Project Type** | Penetration Testing & Vulnerability Assessment |
| **Client Name** | Mediroza General Hospital |
| **Target** | [https://medirozahospital.com](https://medirozahospital.com) |
| **Scope** | Full black-box penetration test across 4 comprehensive milestones |
| **Duration / Timeline** | 5 Days |
| **Author / Pentester** | Cybersecurity Intern |
| **Authorization** | Written permission secured from client |

---

## 📑 Table of Contents
1. [Executive Summary](#1-executive-summary)
2. [Project Overview, Scope & Methodology](#2-project-overview-scope--methodology)
3. [Tools Used](#3-tools-used)
4. [Project Execution & Milestones](#4-project-execution--milestones)
5. [Risk Findings & Evidence](#5-risk-findings--evidence)
6. [Recommendations & Remediation](#6-recommendations--remediation)
7. [Conclusion](#7-conclusion)

---

### 1. Executive Summary
The assessment of the Mediroza General Hospital web application revealed multiple severe security flaws. The most critical findings include an SQL Injection vulnerability in the authentication portal, an exposed unencrypted database backup, and directory indexing enabled on a sensitive directory. 

By exploiting a basic input manipulation flaw (`admin--`), unauthorized access was gained to restricted patient files, which were subsequently decrypted. Furthermore, a misconfigured legacy directory exposed a full database backup containing highly sensitive data, including the salaries of 30 employees and the financial details of 10 shareholders. Overall, the system requires immediate remediation to prevent a catastrophic data breach.

### 2. Project Overview, Scope & Methodology


**Liability Disclaimer**
> These activities were performed only on systems and devices where written permission had been secured. All materials and techniques documented in this report are for educational and research purposes only. Do not use anything from here to break the law. The client, instructors, and supporting platforms are not responsible for what is done with this knowledge. Every action taken is the sole responsibility of the tester. Misuse can lead to criminal charges, heavy fines, loss of employment, and a permanent record.


**Scope & Methodology**

This report documents a 5-day, black-box penetration test against Mediroza General Hospital's website. A black-box test means starting with zero prior knowledge of the server infrastructure, simulating the approach of an external attacker. 

The engagement followed a standard penetration testing methodology divided into three phases: discovering exposed files (Milestone 1), cracking the encryption on those files (Milestone 2), and identifying a critical data leak containing employee salaries (Milestone 3). This report outlines the exact toolchain utilized, the execution steps, the vulnerabilities found, and the remediation steps required to secure the infrastructure.


### 3. Tools Used
The following tools were utilized during the engagement to achieve the assessment objectives:

| Tool | Purpose |
| :--- | :--- |
| **Kali Linux** | The operating system used for all testing and terminal environments. |
| **WHOIS** | Used to gather public domain registration details and ownership information. |
| **whatweb** | Used to fingerprint the web application software and CMS. |
| **nslookup** | Utilized to resolve the target domain to its live IP address. |
| **curl / curl -I** | Used to inspect raw HTTP response headers and page source code. |
| **wafw00f** | Deployed to identify the presence of a Web Application Firewall (WAF). |
| **dnsrecon** | Used to enumerate comprehensive DNS records (MX, TXT, SRV) for the domain. |
| **Gobuster** | Utilized for directory and file brute-forcing to find hidden paths. |
| **Firefox Browser** | Used to view the website, bypass JavaScript obfuscation, and access the web extraction tool. |
| **Web Hash Extraction Tool** | Used to extract the `$pdf$` security hashes from the locked PDF documents. |
| **Mousepad** | Used to save extracted hashes and read the downloaded SQL database backup. |
| **John the Ripper (CLI & GUI)** | Deployed to crack the PDF encryption using the RockYou wordlist. |
| **wget** | Used to download the exposed SQL database backup from the server. |
|**Nmap / Zenmap** | Used for network discovery, port scanning, and service version detection. |
---

### 4. Project Execution & Milestones

#### 📍 Milestone 1: Initial Access & Reconnaissance
The assessment commenced with footprinting to gather intelligence on the target:
*   **WHOIS & WhatWeb:** Domain registration details and web technologies were identified.
*   **Nslookup:** The target domain was resolved to its live server IP address 
*   **Curl & DNSRecon:** HTTP headers and DNS records were enumerated.
*   **Wafw00f:** The target was identified as being protected by a **LiteSpeed** Web Application Firewall.
*   **Nmap / Zenmap:** Used for active network scanning, identifying open ports, and detecting service versions. 
Following reconnaissance, **Gobuster** was utilized to brute-force directories, which revealed hidden folders, specifically `/staff/` and `/old/`. 

Upon accessing the `/staff/` directory, a login portal was presented. Input manipulation was tested, leading to the discovery of an **SQL Injection (Authentication Bypass)** vulnerability. By inputting **`admin--`** into the login field, the password verification was bypassed. This unauthorized access permitted the download of three confidential patient PDF lab reports.

 **📸 Milestone 1 Evidence:**
 Milestone 1 Evidence:

![Admin Login](Milestone%201/Admin--%20login.png)

![Curl Headers](Milestone%201/Curl%20-I.png)

![DNSRecon](Milestone%201/Dnsrecon.png)

![Nslookup](Milestone%201/Nslookup.png)

![Wafw00f](Milestone%201/Wafw00f.png)

![Whois](Milestone%201/Whois.png)

![Gobuster](Milestone%201/gobuster%20file.png)

![Nmap Port Scan](Milestone%201/nmap_port_scan_results.png)

#### 📍 Milestone 2: Data Extraction & File Cracking
The retrieved patient PDFs were encrypted. The following methodology was applied to recover the contents:
1.  The **Web Hash Extraction Tool** was accessed via Firefox to extract the `$pdf$` security hashes.
2.  The hashes were saved into a text file (`mediroza_hashes.txt`) using **Mousepad**.
3.  The file was loaded into **John the Ripper (Johnny GUI)**. An initial failure occurred due to a known hash permissions bug (the number `4294967292`).
4.  The terminal was used to execute a `sed` command, replacing the buggy number with `-4`.
5.  **John the Ripper** was executed via the terminal using the `rockyou.txt` wordlist. The tool successfully cracked the passwords, granting access to all three medical PDFs.

> **📸 Milestone 2 Evidence:**
> *Insert screenshots of the Web Tool, Mousepad hash file, and Johnny GUI showing the cracked passwords below.*

![Milestone 2](./images/milestone2.png)

#### 📍 Milestone 3: Attack (Cracking) & Critical Data Extraction
The objective of this phase was to uncover staff salaries and shareholder details.
1.  **Obstacle:** Initial attempts to brute-force the `/old/` directory using **Gobuster** were hindered by custom wildcard responses. The server returned fake `200 OK` status pages (1251 bytes) for invalid paths, causing the tool to fail.
2.  **Bypass:** To circumvent this, the `robots.txt` file was inspected. The server attempted to obfuscate the text using a JavaScript auto-reload script. This was bypassed by navigating directly to `https://medirozahospital.com/robots.txt` in Firefox.
3.  **Discovery:** The legitimate `robots.txt` confirmed the existence of the `/old/` path. Manual navigation to `https://medirozahospital.com/old/` revealed that **Directory Indexing** was enabled on the server, displaying a raw list of all files.
4.  **Exploit:** An unencrypted database backup named `mediroza_db_backup_2019.sql` was downloaded using **wget**. Analysis in **Mousepad** revealed two highly sensitive data tables:
    *   **Staff Table:** 30 records exposing full names, emails, National IDs, and monthly salaries (`monthly_salary_zar`).
    *   **Shareholders Table:** 10 records exposing shareholder names, exact share percentages, and share classes.

> **📸 Milestone 3 Evidence:**
> *Insert screenshots of the Firefox directory listing, the `robots.txt` file, and the Mousepad SQL database file below.*

![Milestone 3](./images/milestone3.png)

---

### 5. Risk Findings & Evidence
The vulnerabilities identified during the assessment are categorized below based on their severity and potential business impact:

| # | Risk / Finding | Evidence | Potential Impact | Risk Level |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **SQL Injection (Login Bypass)** | Inputting `admin--` bypassed authentication. | Attackers can access restricted areas without credentials. | 🔴 Critical |
| 2 | **Unencrypted DB Backup Exposed** | `mediroza_db_backup_2019.sql` downloaded via `wget`. | Massive data breach exposing PII, salaries, and National IDs. | 🔴 Critical |
| 3 | **Directory Indexing Enabled** | `/old/` directory publicly listed its contents. | Allows attackers to easily discover and download sensitive backups. | 🔴 Critical |
| 4 | **Weak PDF Encryption** | Hashes cracked by John the Ripper via `rockyou.txt`. | Compromise of confidential patient medical records. | 🟠 High |
| 5 | **Information Leakage** | `robots.txt` disclosed hidden directories. | Aids attackers in mapping the application architecture. | 🟡 Medium |

### 6. Recommendations & Remediation
To mitigate the identified risks, Mediroza General Hospital should implement the following security improvements:
1.  **Fix SQL Injection:** Implement prepared statements and parameterized queries on the login portal to prevent input manipulation.
2.  **Disable Directory Indexing:** Configure the web server (e.g., `Options -Indexes` in Apache) to prevent the listing of files in a directory.
3.  **Secure Backup Files:** Remove all database backups (`.sql` files) from web-accessible directories. Store them in a secure, offline environment.
4.  **Enforce Strong Passwords:** Require complex passwords for all patient PDF documents to defend against dictionary attacks.
5.  **Review `robots.txt`:** Do not use `robots.txt` to hide sensitive directories; use proper authentication and authorization controls instead.

### 7. Conclusion
This engagement successfully demonstrated the severe impact of multiple critical vulnerabilities. Reconnaissance using WHOIS and WhatWeb bypassed the LiteSpeed firewall to uncover hidden directories. An SQL Injection flaw allowed authentication to be bypassed via `admin--`, leading to the retrieval of confidential patient files, which were subsequently decrypted using John the Ripper. 

Most critically, during Milestone 3, server wildcard errors were bypassed to reveal Directory Indexing on the `/old/` folder. This directly resulted in the unauthenticated download of an unencrypted SQL database containing sensitive employee salaries and shareholder records. This project highlights how a single misconfiguration (Directory Indexing) and a basic coding error (SQL Injection) can culminate in a catastrophic data breach.

---
