# 🏥 Mediroza General Hospital - Penetration Testing Project

![Security Assessment](https://img.shields.io/badge/Security-Penetration%20Test-red)
![Status](https://img.shields.io/badge/Status-Completed-success)
![Client](https://img.shields.io/badge/Client-MediRoza%20Hospital-blue)
---
⚖️ Liability Disclaimer

These activities were performed only on systems and devices where written permission had been secured. All materials and techniques documented in this report are for educational and research purposes only. Do not use anything from here to break the law. The client, instructors, and supporting platforms are not responsible for what is done with this knowledge. Every action taken is the sole responsibility of the tester. Misuse can lead to criminal charges, heavy fines, loss of employment, and a permanent record. In most countries, unauthorized access is a crime even when nothing is damaged.

---

📋 Project Overview

Project Detail Information
Project Type Penetration Testing & Vulnerability Assessment
Client Name Mediroza General Hospital
Target https://medirozahospital.com
Scope Full black-box penetration test across 4 comprehensive milestones
Duration / Timeline 5 Days
Author / Pentester Cybersecurity Intern
Authorization Written permission secured from client

---

📑 Table of Contents

1. Executive Summary
2. Scope and Methodology
3. Project Execution & Milestones
4. Risk Analysis & Impact
5. Recommendations & Remediation
6. Conclusion
7. Liability Disclaimer

---

Executive Summary

Over a 5-day period, a comprehensive black-box penetration testing engagement was conducted against Mediroza General Hospital (https://medirozahospital.com). Simulating an external attacker with zero prior knowledge of the internal infrastructure, the assessment evaluated web application defenses, access controls, data confidentiality, and system hardening across four structured milestones.

The engagement revealed critical vulnerabilities—specifically:

· ★ Authentication bypass via SQL injection
· ★ Directory indexing disclosures
· ★ An unencrypted database backup containing sensitive employee salaries and shareholder details

This report documents the complete assessment, risk ratings, and immediate remediation steps required to secure the hospital's digital assets.

---

 Scope and Methodology

Scope

The assessment encompassed the publicly facing web application infrastructure of Mediroza General Hospital, focusing on:

· ★ Identifying exploitable entry points
· ★ Bypassing authentication mechanisms
· ★ Extracting restricted patient data
· ★ Uncovering administrative misconfigurations

Methodology & Toolchain

The testing methodology followed a systematic four-milestone structure utilizing the following toolchain:

Tool Purpose
Kali Linux Operating system used for all reconnaissance, scanning, and exploitation activities
WHOIS Gather domain registration details, owner information, and name servers
WhatWeb Fingerprint web technologies, CMS, plugins, and server headers
Nslookup Resolve domain names to target IP addresses using DNS
Curl / Curl -I Read HTTP response headers, inspect page sources, and fetch web endpoints
Wafw00f Detect whether a Web Application Firewall (WAF) protects the target web application
DNSRecon Enumerate comprehensive DNS records (NS, MX, SPF, TXT, SRV)
Gobuster Directory and file brute-forcing against web endpoints
John the Ripper (CLI & GUI) Password cracking utility used to recover encryption keys on restricted patient PDF documents
wget Securely download remote files and backup archives from exposed directories
Mousepad / Text Editor Inspect raw SQL database backups and extracted plain-text personnel/financial records

---

 Project Execution & Milestones

Milestone 1: Reconnaissance, Footprinting & Initial Access

The engagement began with black-box reconnaissance against the target:

· ★ WHOIS & WhatWeb: Queried domain registration information and fingerprinted the web application stack
· ★ Nslookup, Curl, & Wafw00f: Resolved live server IPs, inspected HTTP response headers, and identified LiteSpeed WAF protection
· ★ DNSRecon & Enumeration: Inspected DNS records and navigated robots.txt (bypassing client-side JavaScript obfuscation) to discover restricted paths like /patient/, /staff/, and /old/
· ★ Initial Access: Accessing /staff/ revealed a login portal vulnerable to an SQL Injection (Authentication Bypass). Inputting admin'-- successfully bypassed verification, permitting the download of three confidential patient PDF lab reports

Milestone 2: File Cracking & Decryption

· ★ Analysis: The retrieved patient lab reports were password-protected and encrypted
· ★ Execution: Using John the Ripper (via both CLI and the Johnny GUI), hashes were extracted and cracked using the rockyou.txt wordlist, recovering valid passphrases for all three files
· ★ Deliverable: Fully decrypted patient lab reports confirming unauthorized data accessibility under black-box constraints

Milestone 3: Critical Data Extraction & Database Backup

· ★ Wildcard Response Handling: Initial directory enumeration failed due to server wildcard responses returning fake 200 OK status codes for invalid paths. This was resolved by tuning scanning filters
· ★ Directory Indexing Vulnerability: Manual navigation to the /old/ endpoint revealed that Directory Indexing was enabled, exposing a raw file index containing an unencrypted SQL database backup: mediroza_db_backup_2019.sql
· ★ Extraction & Analysis: Using wget, the backup file was downloaded and inspected via Mousepad. The SQL dump exposed two sensitive internal tables:
  · Staff Table: 30 records containing full names, department emails, phone numbers, National IDs, and monthly salaries (monthly_salary_zar)
  · Shareholders Table: 10 records containing stakeholder names, exact share percentages, and share classes

Milestone 4: Reporting & Risk Evaluation

· ★ Consolidation of findings, risk scoring, and delivery of professional remediation guidance

---

⚠️ Risk Analysis & Impact

# Risk / Finding Evidence / Observation Potential Impact Risk Level
1 SQL Injection (Login Bypass) Inputting admin'-- bypassed authentication Attackers can access restricted areas without credentials 🔴 Critical
2 Directory Indexing Enabled /old/ directory displayed raw server file listings Attackers can browse and download sensitive backup files directly 🔴 Critical
3 Unencrypted Database Backup Exposed mediroza_db_backup_2019.sql publicly downloadable via wget Complete exposure of employee PII, national IDs, and internal salaries 🔴 Critical
4 Sensitive Shareholder Records Disclosed Shareholder names and share percentages stored in plain backup Financial and ownership intelligence leaked to unauthorized parties 🟠 High
5 Weak Patient File Encryption Retrieved PDF lab reports susceptible to John the Ripper Compromise of confidential patient medical records 🟠 High
6 Server Custom Wildcard Responses Non-existent paths returned 200 OK status headers Obfuscates web reconnaissance and complicates automated scanning 🟡 Medium

---

🛠️ Recommendations & Remediation

To mitigate identified risks, Mediroza General Hospital should immediately implement:

1. Fix SQL Injection: Implement prepared statements and parameterized queries on all login portals
2. Disable Directory Indexing: Configure web server settings (e.g., Options -Indexes in Apache) to prevent raw file listings
3. Secure Backup Files: Remove all legacy database backups and sensitive archives from web-accessible directories
4. Enforce Strong Access Controls: Restrict administrative paths (/old/, /wp-admin/) using robust authentication and IP whitelisting
5. Strengthen Document Encryption: Apply robust encryption standards with strong passphrases for all confidential documents

---

 Conclusion

During this 4-milestone penetration testing engagement, critical vulnerabilities were successfully identified and exploited under controlled black-box conditions. From reconnaissance and password cracking to exploiting directory indexing on the /old/ endpoint, the assessment demonstrated how configuration oversights and coding flaws lead to catastrophic data breaches.

Immediate implementation of remediation steps is vital to securing Mediroza General Hospital's infrastructure.

---

