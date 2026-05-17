# Web Server Log Investigation Lab

## SOC Training - Apache Access Log Analysis

**Level:** Beginner to Intermediate  
**Environment:** Linux / Kali Linux / Ubuntu  
**Tools Used:** grep, awk, sed, sort, uniq, wc  
**Objective:** Learn how to investigate Apache access logs and identify suspicious activities.

---

# Introduction

Web servers generate logs that record every request sent by users, browsers, bots, and attackers.  
Security analysts use these logs to:

- Monitor website traffic
- Detect attacks
- Investigate incidents
- Identify malicious IP addresses
- Analyze suspicious requests

This lab demonstrates how to analyze Apache access logs using Linux command-line tools.

---

# Understanding Apache Log Structure

Apache commonly uses the **Combined Log Format**.

## Log Format

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

---

## Sample Log Entry

```text
203.0.113.25 - - [12/Feb/2026:09:15:42 +0000] "GET /dashboard HTTP/1.1" 200 4521 "-" "Mozilla/5.0"
```

---

# Log Field Explanation

| Field | Position | Description |
|---|---|---|
| Client IP | `$1` | Source IP address |
| Identity | `$2` | Remote identity |
| Username | `$3` | Authenticated user |
| Timestamp | `$4` | Date & time |
| Method | `$6` | HTTP request method |
| URL | `$7` | Requested resource |
| Protocol | `$8` | HTTP version |
| Status Code | `$9` | Server response code |
| Response Size | `$10` | Bytes transferred |
| User-Agent | `$12+` | Browser or tool |

---

# Lab Setup

## Create Workspace

```bash
mkdir -p ~/security-labs/apache-analysis
cd ~/security-labs/apache-analysis
```

---

## Download Sample Log File

```bash
wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs -O access.log
```

---

## Verify File

```bash
ls -lh access.log
head -5 access.log
```

---

# Part 1 — Basic Log Analysis

---

# Exercise 1: Count Total Requests

## Objective

Find the total number of requests inside the log file.

## Command

```bash
wc -l access.log
```

## Explanation

- `wc` = word count utility
- `-l` = count lines
- Every line represents one web request

---

# Exercise 2: Extract Client IP Addresses

## Show First 10 IPs

```bash
awk '{print $1}' access.log | head
```

---

## Count Unique IP Addresses

```bash
awk '{print $1}' access.log | sort -u | wc -l
```

---

## Explanation

- `awk '{print $1}'` extracts IP addresses
- `sort -u` sorts and removes duplicates
- `wc -l` counts unique entries

---

# Exercise 3: Detect Top Active IPs

## Objective

Identify the most active hosts communicating with the server.

## Command

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
```

---

## Security Insight

High request counts may indicate:

- Automated bots
- Vulnerability scanners
- Brute-force attempts
- DDoS activity

---

# Exercise 4: Analyze HTTP Response Codes

## Display All Status Codes

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

---

## Count 404 Errors

```bash
awk '$9 == 404' access.log | wc -l
```

---

## Why 404 Errors Matter

Frequent 404 requests may indicate:

- Directory scanning
- Attack reconnaissance
- Broken links
- Automated exploit attempts

---

# Exercise 5: Most Requested Missing Pages

## Command

```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -nr | head -5
```

---

## Possible Indicators

| URL Pattern | Possible Threat |
|---|---|
| `/admin/` | Admin panel scanning |
| `/phpmyadmin/` | Database access attempt |
| `/.env` | Environment file exposure |
| `/backup.zip` | Backup file hunting |

---

# Part 2 — Threat Hunting with grep

---

# Exercise 6: Detect Vulnerability Scanners

## Find Nikto Scanner Requests

```bash
grep -i "nikto" access.log
```

---

## Count Nikto Requests

```bash
grep -i "nikto" access.log | wc -l
```

---

## Extract Attacker IPs

```bash
grep -i "nikto" access.log | awk '{print $1}' | sort -u
```

---

# Exercise 7: Search for SQL Injection Attempts

## Command

```bash
grep -iE "(union|select|drop|insert|update)" access.log
```

---

## Count SQL Injection Indicators

```bash
grep -iE "(union|select|drop)" access.log | wc -l
```

---

## Extract Source IPs

```bash
grep -iE "(union|select|drop)" access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

---

# Exercise 8: Detect Cross-Site Scripting (XSS)

## Command

```bash
grep -iE "(<script|javascript:|onerror=|alert\()" access.log
```

---

## Count XSS Attempts

```bash
grep -iE "(<script|javascript:|onerror=)" access.log | wc -l
```

---

# Exercise 9: Detect Directory Traversal

## Command

```bash
grep -E "(\.\./|\.\.%2[Ff])" access.log
```

---

## Count Traversal Attempts

```bash
grep -E "(\.\./|\.\.%2[Ff])" access.log | wc -l
```

---

# Part 3 — Advanced Log Analysis

---

# Exercise 10: Traffic Analysis by Hour

## Command

```bash
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | sort -nr
```

---

## Purpose

Detect unusual traffic spikes or off-hour activity.

---

# Exercise 11: HTTP Method Distribution

## Command

```bash
awk '{print $6}' access.log | tr -d '"' | sort | uniq -c | sort -nr
```

---

## Security Considerations

Unexpected methods like:

- PUT
- DELETE
- TRACE

may indicate exploitation attempts.

---

# Exercise 12: Analyze POST Requests

## Extract POST Request IPs

```bash
awk '$6 == "\"POST"' access.log | awk '{print $1}' | sort -u
```

---

## Detailed POST Activity

```bash
awk '$6 == "\"POST"' access.log | awk '{print $1, $7, $9}' | head
```

---

# Exercise 13: Measure Bandwidth Usage

## Total Data Transferred

```bash
awk '{sum += $10} END {print sum/1024/1024 " MB"}' access.log
```

---

## Highest Bandwidth Consumers

```bash
awk '{traffic[$1] += $10} END {for (ip in traffic) print ip, traffic[ip]/1024/1024 " MB"}' access.log | sort -k2 -nr | head
```

---

# Part 4 — Data Transformation

---

# Exercise 14: Export Unique IP Addresses

## Command

```bash
awk '{print $1}' access.log | sort -u > unique_ips.txt
```

---

# Exercise 15: Mask IP Addresses

## Command

```bash
sed 's/\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}/\1XXX/' access.log | head
```

---

# Exercise 16: Extract Requested URLs

## Command

```bash
awk '{print $7}' access.log | sort -u > urls.txt
```

---

# Exercise 17: Extract Dynamic Pages

## Command

```bash
awk '{print $7}' access.log | grep -E "\.(php|jsp|asp)$"
```

---

# Final Investigation Report

Create a file named:

```text
Apache-Investigation-Report.md
```

---

# Report Template

```md
# Apache Investigation Report

## Analyst Information

- Analyst Name:
- Analysis Date:
- Log Source:
- Investigation Scope:

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

# HTTP Status Code Breakdown

[paste output]

---

# Security Findings

## Vulnerability Scanning

- Nikto requests:
- Scanner IPs:

## SQL Injection Attempts

- Total attempts:
- Attacking IPs:

## XSS Attempts

- Total attempts:

## Directory Traversal

- Total attempts:

---

# Suspicious IP Addresses

| IP Address | Suspicious Activity |
|---|---|
| x.x.x.x | SQL Injection |
| x.x.x.x | Scanner Activity |

---

# Recommendations

- Block malicious IPs
- Enable Web Application Firewall
- Monitor repeated failed requests
- Patch vulnerable services

---

# Conclusion

[Write your final findings]

```

---

# Deliverables

Submit the following files:

1. `Apache-Investigation-Report.md`
2. `unique_ips.txt`
3. `urls.txt`
4. `suspicious_ips.txt`

---

# Bonus Challenges

- Create a bash script to automate the analysis
- Generate charts using gnuplot
- Analyze multiple log files together
- Build SIEM detection rules
- Create custom alerting scripts

---

# Helpful Resources

- Apache Documentation
- OWASP Top 10
- GNU grep Manual
- GNU awk Manual
- Linux Command Reference

---

# Estimated Completion Time

**2 - 3 Hours**

---

# Learning Outcomes

After completing this lab, you will be able to:

- Analyze Apache access logs
- Detect common web attacks
- Identify suspicious IP addresses
- Use grep, awk, sed efficiently
- Create professional SOC investigation reports

---

# Author

Cyber Security Log Analysis Practice Lab

````

