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
1. [Liability Disclaimer](#1-liability-disclaimer)
2. [Introduction](#2-introduction)
3. [Tools Used](#3-tools-used)
4. [Activities Performed (Milestones 1-3)](#4-activities-performed)
5. [Risk Analysis / Impact (Milestone 4)](#5-risk-analysis--impact)
6. [Recommendations & Remediation](#6-recommendations--remediation)
7. [Conclusion](#7-conclusion)

---

### 1. Liability Disclaimer
> These activities were performed only on systems and devices where written permission had been secured. All materials and techniques documented in this report are for educational and research purposes only. Do not use anything from here to break the law. The client, instructors, and supporting platforms are not responsible for what is done with this knowledge. Every action taken is the sole responsibility of the tester. Misuse can lead to criminal charges, heavy fines, loss of employment, and a permanent record. In most countries, unauthorized access is a crime even when nothing is damaged.

### 2. Introduction
This report documents a 5-day, black-box penetration test against Mediroza General Hospital's website. A black-box test implies starting with zero prior knowledge of the server infrastructure, simulating the approach of an external attacker. 

The engagement was divided into progressive phases: discovering exposed files (Milestone 1), cracking the encryption on those files (Milestone 2), and identifying a critical data leak containing employee salaries (Milestone 3). This report outlines the exact toolchain utilized, the methodologies applied, and the remediation steps required to secure the infrastructure.

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

---

### 4. Activities Performed

#### 📍 Milestone 1: Initial Access & Reconnaissance
The assessment commenced with footprinting to gather intelligence on the target:
*   **WHOIS & WhatWeb:** Domain registration details and web technologies were identified.
*   **Nslookup:** The target domain was resolved to its live server IP address starting with **198.x.x.x**.
*   **Curl & DNSRecon:** HTTP headers and DNS records were enumerated.
*   **Wafw00f:** The target was identified as being protected by a **LiteSpeed** Web Application Firewall.

Following reconnaissance, **Gobuster** was utilized to brute-force directories, which revealed hidden folders, specifically `/staff/` and `/old/`. 

Upon accessing the `/staff/` directory, a login portal was presented. Input manipulation was tested, leading to the discovery of an **SQL Injection (Authentication Bypass)** vulnerability. By inputting **`admin--`** into the login field, the password verification was bypassed. This unauthorized access permitted the download of three confidential patient PDF lab reports.

> **📸 Milestone 1 Evidence:**
> *Insert screenshots of WHOIS, WhatWeb, Nslookup, Wafw00f, and the `admin--` login bypass below.*

![Milestone 1](./images/milestone1.png)

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

### 5. Risk Analysis / Impact (Milestone 4)
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
PENETRATION TESTING REPORT
MEDIROZA GENERAL HOSPITAL - WEEK 4 PROJECT (MILESTONE 4)
| Project Detail | Information |
|---|---|
| Project Type | Penetration Testing & Vulnerability Assessment |
| Client Name | Mediroza General Hospital |
| Target | [https://medirozahospital.com](https://medirozahospital.com) |
| Scope | Full black-box penetration test across 4 comprehensive milestones |
| Duration / Timeline | 5 Days |
| Author / Pentester | Cybersecurity Professional |
| Authorization | Written permission secured from client |
1. Liability Disclaimer
I have performed these activities only on systems and devices where I had secured written permission. All materials and techniques documented in this report are for educational and research purposes only. Do not use anything from here to break the law. The client, instructors, and supporting platforms are not responsible for what is done with this knowledge. Every action taken is the sole responsibility of the tester. Misuse can lead to criminal charges, heavy fines, loss of employment, and a permanent record. In most countries, unauthorized access is a crime even when nothing is damaged.
2. Introduction
This report covers the comprehensive black-box penetration testing engagement conducted against Mediroza General Hospital ([https://medirozahospital.com](https://medirozahospital.com)) over a 5-day period. The assessment was divided into progressive milestones, moving from initial reconnaissance and footprinting (Milestone 1), file cracking and decryption using advanced tooling like John the Ripper (Milestone 2), critical data extraction and database backup analysis (Milestone 3), to professional reporting (Milestone 4).
Every phase was executed independently within a controlled lab environment. This report details the complete toolchain utilized, activities performed, technical findings, associated risks, and actionable remediation steps.
3. Tools Used
The table below lists each tool utilized during the engagement and its operational purpose:
| Tool | Purpose |
|---|---|
| Kali Linux | Operating system used for reconnaissance, scanning, and exploitation activities |
| WHOIS | Find domain registration details, owner information, and name servers |
| whatweb | Fingerprint web technologies, CMS, plugins, and server headers |
| nslookup | Resolve domain names to target IP addresses using DNS |
| curl / curl -I | Read HTTP response headers, inspect page sources, and fetch web endpoints |
| wafw00f | Detect whether a Web Application Firewall (WAF) protects the target web application |
| dnsrecon | Enumerate comprehensive DNS records (NS, MX, SPF, TXT, SRV) |
| Gobuster | Directory and file brute-forcing against web endpoints |
| John the Ripper | Password cracking utility used to recover encryption keys on restricted patient PDF documents |
| wget | Securely download remote files and backup archives from exposed directories |
| Mousepad / Text Editor | Inspect raw SQL database backups and extracted plain-text personnel/financial records |
4. Activities Performed
4.1 Milestone 1: Reconnaissance, Footprinting & Initial Access
The engagement began with black-box reconnaissance against [https://medirozahospital.com](https://medirozahospital.com) utilizing an array of standard footprinting tools:
 * WHOIS: Queried domain registration information to identify hosting infrastructure and administrative contacts.
 * WhatWeb: Fingerprinted the web application stack, identifying the underlying Content Management System (CMS) and server configuration.
 * Nslookup: Resolved the target domain to its live server IP address.
 * Curl & Curl -I: Inspected HTTP response headers to evaluate exposed server information and API endpoints (such as /wp-json/).
 * Wafw00f: Checked for active Web Application Firewall protections guarding the target infrastructure.
 * DNSRecon: Enumerate records including name servers, mail exchangers, and service entries.
 * Robots.txt & Directory Enumeration: Accessing robots.txt presented client-side JavaScript obfuscation and auto-refresh loading spinners. Bypassing this via direct browser inspection revealed Disallow rules for /patient/, /staff/, and /old/. Subsequent directory enumeration uncovered exposed endpoints including /staff/ and /wp-admin/.
 * Deliverable: Successful identification of restricted entry points and retrieval of three confidential patient PDF lab reports.
4.2 Milestone 2: File Cracking & Decryption
 * Analysis: The three retrieved patient lab reports were password-protected and encrypted, preventing immediate readability.
 * Execution: Utilizing John the Ripper alongside targeted wordlists, encryption hashes were extracted and cracked to recover the valid passphrases for all three files.
 * Deliverable: Fully decrypted patient lab reports verifying unauthorized data accessibility under black-box constraints.
4.3 Milestone 3: Critical Data Extraction & Database Backup
 * Wildcard Response Handling: Directory enumeration attempts initially failed due to server-side custom wildcard responses returning fake 200 OK status codes (1251 bytes) for invalid paths. This was mitigated by measuring response payload lengths and tuning scanning filters.
 * Directory Indexing Vulnerability: Manual navigation to the /old/ endpoint revealed that Directory Indexing was enabled on the server. Instead of a 404 Not Found error, the web server exposed a raw file index containing an unencrypted SQL database backup: mediroza_db_backup_2019.sql (~7 KB).
 * Extraction & Analysis: Using wget, the backup file was downloaded and inspected via Mousepad. The SQL dump exposed two highly sensitive internal tables:
   * staff table: 30 complete records containing full names, job titles, department emails, phone numbers, national IDs, and monthly salaries (monthly_salary_zar).
   * shareholders table: 10 records containing stakeholder names, exact share percentages, and share classes (e.g., Cedar Health Holdings, Reddy Family Trust).
 * Deliverable: Full documented evidence of database extraction and readable summaries of exposed PII and financial records.
5. Risk Analysis / Impact
| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
|---|---|---|---|---|
| 1 | Directory Indexing Enabled | /old/ directory displayed raw server file listings | Attackers can browse and download sensitive backup or legacy files directly | Critical |
| 2 | Unencrypted Database Backup Exposed | mediroza_db_backup_2019.sql publicly downloadable | Complete exposure of employee PII, national IDs, and internal salaries | Critical |
| 3 | Sensitive Shareholder Records Disclosed | Shareholder names and share percentages stored in plain backup | Financial and ownership intelligence leaked to unauthorized parties | High |
| 4 | Weak Patient File Encryption | Retrieved PDF lab reports susceptible to John the Ripper cracking | Compromise of confidential patient medical records | High |
| 5 | Server Custom Wildcard Responses | Non-existent paths returned 200 OK status headers | Obfuscates web reconnaissance and complicates automated scanning | Low |
| 6 | Exposed Technical Headers / DNS | Tools like WhatWeb and DNSRecon identified infrastructure details | Aids attackers in building a detailed target profile | Low |
Risk Level Key: ● Critical | ● High | ● Medium | ● Low
6. Recommendations & Remediation
Based on the critical findings identified throughout the engagement, the following remediation steps are strongly recommended:
 * Disable Directory Indexing: Configure web server settings (e.g., Options -Indexes in Apache) across all directories to prevent raw file listings.
 * Secure Backup Files: Immediately remove all legacy database backups, SQL dumps, and sensitive archives from web-accessible directories.
 * Enforce Strong Access Controls: Restrict administrative and legacy paths (/old/, /wp-admin/) using robust authentication mechanisms and IP whitelisting.
 * Strengthen Document Encryption: Apply robust, modern encryption standards with strong passphrases for all confidential patient or internal documents.
 * Review Server Error Handling: Correct custom error responses so that invalid requests return proper 404 Not Found codes rather than false 200 OK statuses.
7. Conclusion
During this 4-milestone penetration testing engagement against Mediroza General Hospital, critical vulnerabilities were successfully identified and exploited under controlled black-box conditions. Starting from initial reconnaissance with tools like WHOIS, WhatWeb, Nslookup, Curl, Wafw00f, and DNSRecon, through password cracking with John the Ripper, the assessment progressed to the discovery of enabled directory indexing on the /old/ endpoint. This led directly to the unauthenticated exfiltration of an internal SQL database backup containing sensitive staff salaries and shareholder records.
The findings underscore the importance of strict file hygiene, proper web server hardening, and robust access controls. Immediate implementation of the recommended remediation steps is vital to securing Mediroza General Hospital's web infrastructure and safeguarding sensitive patient and corporate data.
