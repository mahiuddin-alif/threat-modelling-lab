# Advanced Apache Access Log Analysis Lab

<div align="center">

## SOC Analyst Practical Lab

### Web Server Monitoring & Threat Detection

![Linux](https://img.shields.io/badge/Platform-Linux-blue)
![Apache](https://img.shields.io/badge/Web%20Server-Apache-red)
![SOC](https://img.shields.io/badge/SOC-Log%20Analysis-green)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner%20to%20Intermediate-orange)

</div>

---

# Table of Contents

1. Introduction
2. Lab Objectives
3. Apache Log Fundamentals
4. Understanding Log Fields
5. Lab Environment Setup
6. Basic Log Analysis
7. Intermediate Security Analysis
8. Advanced Threat Hunting
9. Data Extraction & Transformation
10. Incident Investigation
11. Reporting & Documentation
12. Deliverables
13. Bonus Tasks
14. Learning Outcomes
15. References

---

# 1. Introduction

Web servers continuously generate logs containing information about:

- Website visitors
- Browsers and devices
- Requested resources
- Server responses
- Errors and failures
- Suspicious activities
- Automated scanners
- Attack attempts

Security analysts use these logs to investigate incidents, identify attackers, monitor traffic patterns, and improve organizational security posture.

This practical lab demonstrates how to investigate Apache access logs using standard Linux command-line tools.

---

# 2. Lab Objectives

After completing this lab, you will be able to:

- Understand Apache log structures
- Extract useful information from logs
- Detect suspicious web traffic
- Identify reconnaissance activities
- Detect common web attacks
- Analyze HTTP response codes
- Investigate attacker behavior
- Generate professional investigation reports
- Use Linux text-processing utilities efficiently

---

# 3. Apache Access Log Fundamentals

Apache HTTP Server stores client requests in access logs.

The default Apache combined log format is:

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

---

# Example Log Entry

```text
203.0.113.25 - - [15/May/2026:14:23:51 +0000] "GET /login.php HTTP/1.1" 200 5321 "https://example.com" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

---

# 4. Understanding Apache Log Fields

| Position | Field Name | Description | Example |
|---|---|---|---|
| `$1` | IP Address | Client source IP | `203.0.113.25` |
| `$2` | Identity | RFC 1413 identity | `-` |
| `$3` | Username | Authenticated user | `admin` |
| `$4` | Timestamp | Request time | `[15/May/2026:14:23:51 +0000]` |
| `$5-$7` | Request | Method + URL + Protocol | `"GET /login.php HTTP/1.1"` |
| `$8` | Protocol | HTTP version | `HTTP/1.1"` |
| `$9` | Status Code | Server response | `200` |
| `$10` | Response Size | Bytes sent | `5321` |
| `$11` | Referer | Referring URL | `"https://example.com"` |
| `$12+` | User-Agent | Browser/tool info | `"Mozilla/5.0"` |

---

# 5. Lab Environment Setup

---

# Step 1 — Create Working Directory

```bash
mkdir -p ~/soc-training/apache-lab
cd ~/soc-training/apache-lab
```

---

# Step 2 — Download Sample Access Log

```bash
wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs -O access.log
```

---

# Step 3 — Verify File

```bash
ls -lh access.log
```

---

# Step 4 — Preview Log Entries

```bash
head -5 access.log
```

---

# Step 5 — Count Total Lines

```bash
wc -l access.log
```

Each line represents one HTTP request.

---

# 6. Basic Log Analysis

---

# Exercise 1 — Count Total Requests

## Objective

Determine how many requests were recorded.

## Command

```bash
wc -l access.log
```

---

# Explanation

| Command | Purpose |
|---|---|
| `wc` | Word count utility |
| `-l` | Count lines only |

---

# Why This Matters

High traffic volume may indicate:

- Legitimate user activity
- DDoS attacks
- Automated scraping
- Bot traffic

---

# Exercise 2 — Extract Client IP Addresses

## Display First 10 IP Addresses

```bash
awk '{print $1}' access.log | head
```

---

# Explanation

| Component | Purpose |
|---|---|
| `awk` | Text processing utility |
| `{print $1}` | Print first column |
| `head` | Show first lines |

---

# Count Unique Visitors

```bash
awk '{print $1}' access.log | sort -u | wc -l
```

---

# Security Importance

Unique IP analysis helps identify:

- Number of visitors
- Distributed attacks
- Botnets
- Scanning infrastructure

---

# Exercise 3 — Identify Top Active IPs

## Command

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
```

---

# Detailed Breakdown

| Command | Function |
|---|---|
| `awk '{print $1}'` | Extract IP addresses |
| `sort` | Sort entries |
| `uniq -c` | Count duplicates |
| `sort -nr` | Numerical reverse sorting |
| `head -10` | Top 10 results |

---

# Example Output

```text
523 192.168.1.50
487 10.0.0.22
412 172.16.5.14
```

---

# Analyst Notes

IPs with excessive requests may indicate:

- Web crawlers
- Penetration testing tools
- Brute-force attacks
- Malicious bots

---

# Exercise 4 — Analyze HTTP Status Codes

## Command

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

---

# Common Status Codes

| Code | Meaning |
|---|---|
| 200 | Successful request |
| 301 | Redirect |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |

---

# Security Relevance

Large numbers of:

- `404` → scanning attempts
- `403` → blocked access attempts
- `500` → server issues/exploitation

---

# Exercise 5 — Investigate 404 Errors

## Count 404 Responses

```bash
awk '$9 == 404' access.log | wc -l
```

---

# Most Requested Missing Resources

```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -nr | head -10
```

---

# Example Output

```text
45 /admin/
32 /.env
29 /backup.zip
24 /phpmyadmin/
```

---

# Threat Analysis

These paths often indicate attackers searching for:

| URL | Purpose |
|---|---|
| `/admin/` | Admin panels |
| `/.env` | Environment credentials |
| `/backup.zip` | Backup leaks |
| `/phpmyadmin/` | Database access |

---

# 7. Intermediate Security Analysis

---

# Exercise 6 — Detect Vulnerability Scanners

## Nikto Detection

```bash
grep -i "nikto" access.log
```

---

# Count Scanner Requests

```bash
grep -i "nikto" access.log | wc -l
```

---

# Extract Scanner IPs

```bash
grep -i "nikto" access.log | awk '{print $1}' | sort -u
```

---

# Why This Matters

Nikto is a vulnerability scanner commonly used for:

- Web reconnaissance
- Vulnerability discovery
- Security assessments

Unrecognized scanning may indicate hostile activity.

---

# Exercise 7 — Detect SQL Injection Attempts

---

# Common SQL Injection Keywords

| Keyword | Purpose |
|---|---|
| UNION | Extract database data |
| SELECT | Query database |
| DROP | Delete tables |
| INSERT | Inject records |

---

# Search for SQL Injection Patterns

```bash
grep -iE "(union|select|insert|update|drop|exec)" access.log
```

---

# Count SQL Injection Attempts

```bash
grep -iE "(union|select|drop)" access.log | wc -l
```

---

# Extract Attacker IPs

```bash
grep -iE "(union|select|drop)" access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

---

# Example Malicious Request

```text
GET /index.php?id=1 UNION SELECT username,password FROM users
```

---

# Security Impact

Successful SQL injection may lead to:

- Database compromise
- Credential theft
- Data leakage
- Remote code execution

---

# Exercise 8 — Detect Cross-Site Scripting (XSS)

---

# Common XSS Patterns

| Pattern | Meaning |
|---|---|
| `<script>` | Script injection |
| `javascript:` | JS URI scheme |
| `onerror=` | Event injection |
| `alert()` | Test payload |

---

# Detect XSS Attempts

```bash
grep -iE "(<script|javascript:|onerror=|alert\()" access.log
```

---

# Count XSS Attempts

```bash
grep -iE "(<script|javascript:|onerror=)" access.log | wc -l
```

---

# Identify Source IPs

```bash
grep -iE "(<script|javascript:|onerror=)" access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

---

# Exercise 9 — Detect Directory Traversal

---

# Common Traversal Patterns

| Pattern | Example |
|---|---|
| `../` | Linux traversal |
| `..%2F` | URL encoded traversal |

---

# Detection Command

```bash
grep -E "(\.\./|\.\.%2[Ff])" access.log
```

---

# Count Attempts

```bash
grep -E "(\.\./|\.\.%2[Ff])" access.log | wc -l
```

---

# Potential Targets

Attackers use traversal to access:

- `/etc/passwd`
- Configuration files
- Application secrets
- Backup files

---

# 8. Advanced Threat Hunting

---

# Exercise 10 — Analyze Traffic by Hour

## Command

```bash
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | sort -nr
```

---

# Purpose

Identify unusual activity spikes.

---

# Investigation Clues

Traffic spikes during:

- Midnight
- Early morning
- Weekends

may indicate automated attacks.

---

# Exercise 11 — Analyze HTTP Methods

## Command

```bash
awk '{print $6}' access.log | tr -d '"' | sort | uniq -c | sort -nr
```

---

# Common Methods

| Method | Purpose |
|---|---|
| GET | Retrieve resource |
| POST | Submit data |
| PUT | Upload content |
| DELETE | Remove content |

---

# Security Concern

Unexpected methods may indicate:

- File uploads
- Web shell deployment
- API abuse

---

# Exercise 12 — Investigate POST Requests

---

# Extract POST Request IPs

```bash
awk '$6 == "\"POST"' access.log | awk '{print $1}' | sort -u
```

---

# Detailed POST Activity

```bash
awk '$6 == "\"POST"' access.log | awk '{print $1, $7, $9}' | head -20
```

---

# Why POST Matters

POST requests often involve:

- Authentication
- Form submissions
- File uploads
- API activity

Attackers frequently abuse POST requests.

---

# Exercise 13 — Bandwidth Usage Analysis

---

# Calculate Total Bandwidth

```bash
awk '{sum += $10} END {print sum/1024/1024 " MB"}' access.log
```

---

# Top Bandwidth Consumers

```bash
awk '{bytes[$1] += $10} END {for (ip in bytes) print ip, bytes[ip]/1024/1024 " MB"}' access.log | sort -k2 -nr | head
```

---

# Investigation Importance

High bandwidth consumption may indicate:

- File downloads
- Data exfiltration
- Media streaming
- DoS attacks

---

# 9. Data Extraction & Transformation

---

# Exercise 14 — Export Unique IPs

```bash
awk '{print $1}' access.log | sort -u > unique_ips.txt
```

---

# Exercise 15 — Anonymize IP Addresses

## Privacy Protection

```bash
sed 's/\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}/\1XXX/' access.log
```

---

# Exercise 16 — Export Requested URLs

```bash
awk '{print $7}' access.log | sort -u > urls.txt
```

---

# Exercise 17 — Extract Dynamic Web Pages

```bash
awk '{print $7}' access.log | grep -E "\.(php|asp|jsp)$"
```

---

# 10. Incident Investigation Scenario

---

# Scenario

The organization suspects:

- Web scanning activity
- SQL injection attempts
- Unauthorized access attempts

Your task is to:

1. Identify suspicious IP addresses
2. Detect malicious payloads
3. Determine attack types
4. Recommend defensive actions

---

# Investigation Questions

- Which IP generated the highest traffic?
- Were SQL injection attempts successful?
- Are there signs of automated scanning?
- What URLs were targeted most?
- Are attackers using encoded payloads?

---

# 11. Final Investigation Report

Create:

```text
Advanced-Apache-Investigation-Report.md
```

---

# Suggested Report Structure

```md
# Apache Access Log Investigation Report

## Analyst Information

- Analyst Name:
- Investigation Date:
- Target Server:
- Log Source:
- Analysis Scope:

---

# Executive Summary

[Brief overview of findings]

---

# Traffic Statistics

## Total Requests

```bash
wc -l access.log
```

## Unique Visitors

```bash
awk '{print $1}' access.log | sort -u | wc -l
```

---

# Top Active IP Addresses

[paste output]

---

# HTTP Status Code Distribution

[paste output]

---

# Security Findings

## Vulnerability Scanning

- Total Nikto requests:
- Source IPs:
- Targeted resources:

---

## SQL Injection Attempts

- Total attempts:
- Attacker IPs:
- Sample payloads:

---

## XSS Attempts

- Total attempts:
- Affected URLs:

---

## Directory Traversal Attempts

- Total attempts:
- Source IPs:

---

# High-Risk IP Addresses

| IP Address | Suspicious Behavior |
|---|---|
| x.x.x.x | SQL Injection |
| x.x.x.x | Directory Traversal |

---

# Recommendations

- Block malicious IPs
- Deploy WAF rules
- Enable rate limiting
- Patch vulnerable applications
- Improve logging visibility

---

# Conclusion

[Final assessment]

```

---

# 12. Deliverables

Submit the following:

| File | Description |
|---|---|
| `Advanced-Apache-Investigation-Report.md` | Complete investigation report |
| `unique_ips.txt` | Unique visitors |
| `urls.txt` | Requested URLs |
| `suspicious_ips.txt` | Malicious IP list |

---

# 13. Bonus Challenges

- Automate analysis using Bash scripting
- Create SIEM detection rules
- Visualize traffic patterns
- Correlate logs from multiple servers
- Detect brute-force authentication attempts
- Build custom IOC lists

---

# 14. Learning Outcomes

By completing this lab, you learned how to:

- Investigate Apache logs
- Detect common web attacks
- Analyze malicious traffic
- Use Linux forensic commands
- Produce SOC investigation reports
- Identify Indicators of Compromise (IOCs)

---

# 15. Recommended References

- Apache HTTP Documentation
- OWASP Top 10
- MITRE ATT&CK Framework
- GNU grep Manual
- GNU awk Documentation
- Linux Command Line Guide

---

# Estimated Completion Time

**3 — 4 Hours**

---

# Difficulty Level

**Beginner → Intermediate**

---

# Author

Cyber Security & SOC Training Lab
