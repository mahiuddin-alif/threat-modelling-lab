# Web Server Log Investigation Lab

## SOC Training - Apache Access Log Analysis

**Level:** Beginner to Intermediate  
**Environment:** Linux / Kali Linux / Ubuntu / WSL  
**Tools Used:** grep, awk, sed, sort, uniq, wc, cut, head, tail  
**Prerequisites:** Basic Linux command line knowledge  
**Objective:** Learn how to investigate Apache access logs and identify suspicious activities.

---

# Table of Contents

1. [Introduction](#introduction)
2. [Log Structure](#log-structure)
3. [Lab Setup](#lab-setup)
4. [Basic Analysis](#basic-analysis)
5. [Threat Hunting](#threat-hunting)
6. [Advanced Analysis](#advanced-analysis)
7. [Data Transformation](#data-transformation)
8. [Investigation Report](#investigation-report)
9. [Bonus Challenges](#bonus-challenges)
10. [Cheat Sheet](#cheat-sheet)

---

# Introduction

## Why Log Analysis Matters

Web servers generate logs that record every request sent by users, browsers, bots, and attackers.  
Security analysts use these logs to:

- Monitor website traffic patterns
- Detect attacks in real-time
- Investigate security incidents
- Identify malicious IP addresses
- Analyze suspicious request patterns
- Create threat intelligence
- Meet compliance requirements (PCI-DSS, HIPAA, etc.)

## Common Attack Types You'll Detect

| Attack Type | Log Indicators |
|---|---|
| SQL Injection | `union`, `select`, `--`, `'` characters in URLs |
| XSS | `<script>`, `alert()`, `javascript:` |
| Directory Traversal | `../`, `..%2f`, `..%252f` |
| Scanner Activity | `nikto`, `nmap`, `wpscan` user-agents |
| Brute Force | Multiple POST requests to login pages |
| DDoS | High request rate from single IP |

---

# Understanding Apache Log Structure

## Common Log Formats

### 1. Common Log Format (CLF)
```apache
LogFormat "%h %l %u %t \"%r\" %>s %b" common
```
**Example:**
```
192.168.1.100 - frank [10/Oct/2026:13:55:36 +0000] "GET /index.html HTTP/1.0" 200 2326
```

### 2. Combined Log Format (Most Common)
```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```
**Example:**
```
203.0.113.25 - - [12/Feb/2026:09:15:42 +0000] "GET /dashboard HTTP/1.1" 200 4521 "https://google.com" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

### 3. Extended Log Format (Custom)
```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D %I %O" extended
```
Adds: `%D` (request time μs), `%I` (bytes received), `%O` (bytes sent)

---

## Detailed Field Breakdown

| Position | Field | Variable | Description | Example |
|----------|-------|----------|-------------|---------|
| $1 | Client IP | `%h` | Source IP address | `203.0.113.25` |
| $2 | Identity (RFC 1413) | `%l` | Remote identity (rarely used) | `-` |
| $3 | Username | `%u` | Authenticated user | `admin` or `-` |
| $4 | Timestamp | `%t` | Date & time in [UTC] | `[12/Feb/2026:09:15:42 +0000]` |
| $5 | Request Line | `%r` | Full HTTP request | `"GET /dashboard HTTP/1.1"` |
| $5a | HTTP Method | - | GET, POST, PUT, DELETE | `GET` |
| $5b | Request URI | - | Resource path | `/dashboard` |
| $5c | HTTP Version | - | HTTP/1.0, HTTP/1.1, HTTP/2 | `HTTP/1.1` |
| $6 | Status Code | `%>s` | Server response code | `200` |
| $7 | Response Size | `%b` | Bytes transferred | `4521` |
| $8 | Referer | `%{Referer}i` | Where request came from | `"https://google.com"` |
| $9+ | User-Agent | `%{User-Agent}i` | Client software info | `"Mozilla/5.0..."` |

---

# Lab Setup

## Step 1: Create Workspace

```bash
mkdir -p ~/security-labs/apache-analysis
cd ~/security-labs/apache-analysis
```

## Step 2: Download Sample Log File

### Option A: Realistic Sample Log
```bash
wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs -O access.log
```

### Option B: Create Custom Test Log (If download fails)
```bash
cat > access.log << 'EOF'
192.168.1.10 - - [12/Feb/2026:10:15:22 +0000] "GET /index.html HTTP/1.1" 200 2048 "-" "Mozilla/5.0"
203.0.113.45 - - [12/Feb/2026:10:15:23 +0000] "GET /admin/login.php HTTP/1.1" 404 512 "-" "Mozilla/5.0"
45.33.22.11 - - [12/Feb/2026:10:15:24 +0000] "GET /wp-admin HTTP/1.1" 404 256 "-" "nikto/2.5"
10.0.0.5 - - [12/Feb/2026:10:15:25 +0000] "POST /login.php HTTP/1.1" 200 1024 "-" "Mozilla/5.0"
203.0.113.45 - - [12/Feb/2026:10:15:26 +0000] "GET /admin/index.php?user=1 UNION SELECT 1,2,3 HTTP/1.1" 500 128 "-" "Mozilla/5.0"
45.33.22.11 - - [12/Feb/2026:10:15:27 +0000] "GET /../../../../etc/passwd HTTP/1.1" 403 64 "-" "Python-urllib/3.9"
192.168.1.10 - - [12/Feb/2026:10:15:28 +0000] "GET /images/logo.png HTTP/1.1" 200 8192 "https://example.com" "Mozilla/5.0"
203.0.113.45 - - [12/Feb/2026:10:15:29 +0000] "GET /admin/index.php?id=1<script>alert(1)</script> HTTP/1.1" 200 256 "-" "Mozilla/5.0"
EOF
```

## Step 3: Verify File

```bash
ls -lh access.log
head -5 access.log
wc -l access.log
```

## Step 4: Create Analysis Directory Structure

```bash
mkdir -p {reports,outputs,scripts}
```

---

# Part 1 — Basic Log Analysis

---

## Exercise 1: Count Total Requests

### Objective
Find the total number of requests inside the log file.

### Command
```bash
wc -l access.log
```

### Expected Output
```
10000 access.log
```

### Explanation
- `wc` = word count utility
- `-l` = count lines (each line = one web request)
- Alternative: `cat access.log | wc -l`

### Try It Yourself
```bash
# Count lines with different methods
cat access.log | wc -l
grep -c "^" access.log
awk 'END {print NR}' access.log
```

---

## Exercise 2: Extract Client IP Addresses

### Show First 10 IPs
```bash
awk '{print $1}' access.log | head -10
```

### Show Last 10 IPs
```bash
awk '{print $1}' access.log | tail -10
```

### Count Unique IP Addresses
```bash
awk '{print $1}' access.log | sort -u | wc -l
```

### Explanation
- `awk '{print $1}'` extracts first column (IP addresses)
- `sort -u` sorts and removes duplicates
- `wc -l` counts unique entries

### Alternative Methods
```bash
# Using cut
cut -d' ' -f1 access.log | sort -u | wc -l

# Using sed
sed 's/ .*//' access.log | sort -u | wc -l
```

---

## Exercise 3: Detect Top Active IPs

### Objective
Identify the most active hosts communicating with the server.

### Command
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
```

### Sample Output
```
  1523 192.168.1.100
   892 203.0.113.45
   645 45.33.22.11
   234 10.0.0.5
   198 8.8.8.8
```

### Security Insight

| Request Count | Interpretation |
|---|---|
| < 100 | Normal user activity |
| 100-500 | Heavy user or light bot |
| 500-2000 | Suspicious - Possible scanner |
| > 2000 | High threat - Likely DDoS or aggressive scanner |

### Export Top IPs to File
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -20 > reports/top_ips.txt
```

---

## Exercise 4: Analyze HTTP Response Codes

### Display All Status Codes with Counts
```bash
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

### HTTP Status Code Categories

| Code Range | Category | Meaning |
|---|---|---|
| 1xx | Informational | Request received, continuing |
| 2xx | Success | Request successful |
| 3xx | Redirection | Further action needed |
| 4xx | Client Error | Request contains bad syntax |
| 5xx | Server Error | Server failed to fulfill request |

### Important Status Codes

| Code | Meaning | Security Implication |
|---|---|---|
| 200 | OK | Normal response |
| 301/302 | Redirect | May hide malicious redirects |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Access denied - possible bypass attempts |
| 404 | Not Found | Directory/file scanning |
| 500 | Internal Error | Possible exploit attempt |
| 503 | Service Unavailable | Possible DoS attack |

### Count Specific Status Codes
```bash
# Count 404 errors
awk '$9 == 404' access.log | wc -l

# Count 403 errors
awk '$9 == 403' access.log | wc -l

# Count 500 errors
awk '$9 >= 500' access.log | wc -l

# Count all errors (4xx and 5xx)
awk '$9 >= 400' access.log | wc -l

# Get percentage of errors
total=$(wc -l < access.log)
errors=$(awk '$9 >= 400' access.log | wc -l)
echo "Error rate: $(echo "scale=2; $errors * 100 / $total" | bc)%"
```

### Why 404 Errors Matter

Frequent 404 requests may indicate:
- Directory scanning (looking for admin panels)
- Vulnerability scanning (testing for vulnerable files)
- Attack reconnaissance (mapping the attack surface)
- Broken links (usually not security-related)

---

## Exercise 5: Most Requested Missing Pages

### Command
```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -nr | head -10
```

### Sample Output
```
   45 /admin
   32 /phpmyadmin
   28 /wp-login.php
   22 /.env
   18 /backup.zip
   15 /config.php
   12 /xmlrpc.php
   10 /wp-admin
    8 /robots.txt
    5 /shell.php
```

### Threat Indicator Analysis

| URL Pattern | Possible Threat | Risk Level |
|---|---|---|
| `/admin`, `/wp-admin` | Admin panel scanning | Medium |
| `/phpmyadmin` | Database access attempt | High |
| `/.env` | Environment file exposure | Critical |
| `/backup.zip` | Backup file hunting | Critical |
| `/*.php` with suspicious names | Shell upload attempts | High |
| `/xmlrpc.php` | WordPress XML-RPC attacks | Medium |

---

# Part 2 — Threat Hunting with grep

---

## Exercise 6: Detect Vulnerability Scanners

### Known Scanner User-Agents

| Scanner | User-Agent String |
|---|---|
| Nikto | `Nikto`, `nikto` |
| Nmap | `Nmap Scripting Engine` |
| SQLmap | `sqlmap` |
| Dirb | `dirb` |
| Gobuster | `gobuster` |
| WPScan | `wpscan` |
| Burp Suite | `Burp Suite` |
| ZAP | `ZAP` |

### Find Nikto Scanner Requests
```bash
grep -i "nikto" access.log
```

### Count Nikto Requests
```bash
grep -ci "nikto" access.log
```

### Extract Attacker IPs (with counts)
```bash
grep -i "nikto" access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

### Detect Multiple Scanners at Once
```bash
grep -iE "(nikto|nmap|sqlmap|dirb|gobuster|wpscan|burp|zap)" access.log | awk '{print $1, $7, $12, $13}' | head -20
```

### Create Suspicious IP List
```bash
grep -iE "(nikto|nmap|sqlmap|dirb|gobuster|wpscan)" access.log | awk '{print $1}' | sort -u > outputs/scanner_ips.txt
```

---

## Exercise 7: Search for SQL Injection Attempts

### SQL Injection Patterns

| Pattern | Description |
|---|---|
| `UNION SELECT` | Union-based SQL injection |
| `' OR '1'='1` | Authentication bypass |
| `DROP TABLE` | Destructive attack |
| `INSERT INTO` | Data modification |
| `--` | Comment injection |
| `;` | Query termination |

### Basic SQLi Detection
```bash
grep -iE "(union.*select|select.*from|drop table|insert into|--|';)" access.log
```

### Extended SQLi Patterns
```bash
grep -iE "(union|select|drop|insert|update|delete|create|alter|--|' or '1'='1|' or 1=1)" access.log
```

### Count SQL Injection Indicators
```bash
grep -ciE "(union|select|drop|insert|update|delete)" access.log
```

### Extract SQLi Attackers
```bash
grep -iE "(union|select|drop|insert|update|delete)" access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

### Detailed SQLi Request Analysis
```bash
grep -iE "(union|select|drop)" access.log | awk '{print $1, $7, $9, $12}' | column -t
```

---

## Exercise 8: Detect Cross-Site Scripting (XSS)

### XSS Patterns

| Pattern | Description |
|---|---|
| `<script>` | JavaScript injection |
| `alert(` | Alert box injection |
| `javascript:` | JavaScript pseudo-protocol |
| `onerror=` | Event handler injection |
| `onload=` | Page load event |
| `<img src=x onerror=` | Image-based XSS |

### Basic XSS Detection
```bash
grep -iE "(<script|javascript:|onerror=|alert\(|onload=|<img)" access.log
```

### Extended XSS Patterns
```bash
grep -iE "(<script|<\/script>|javascript:|alert\(|prompt\(|confirm\(|onerror|onload|onclick|onmouseover|<iframe|<body|<svg|expression\()" access.log
```

### Count XSS Attempts
```bash
grep -ciE "(<script|javascript:|onerror=|alert\()" access.log
```

### Extract XSS Attack Details
```bash
grep -iE "(<script|javascript:|alert\()" access.log | awk '{print $1, $7, length($0)}' | sort -k3 -nr | head -10
```

---

## Exercise 9: Detect Directory Traversal

### Traversal Patterns

| Pattern | Description |
|---|---|
| `../` | Standard traversal |
| `..%2f` | URL encoded slash |
| `..%252f` | Double encoded |
| `..\/` | Mixed slashes |
| `....//` | Double traversal |

### Command
```bash
grep -E "(\.\./|\.\.%2[fF]|\.\.%252[fF])" access.log
```

### Count Traversal Attempts
```bash
grep -cE "(\.\./|\.\.%2[fF]|\.\.%252[fF])" access.log
```

### Find Traversal Targets
```bash
grep -E "(\.\./|\.\.%2[fF])" access.log | awk '{print $7}' | sort | uniq -c | sort -nr
```

### Popular Traversal Targets
```bash
grep -E "(\.\./|\.\.%2[fF])" access.log | grep -E "(passwd|shadow|config|\.env|wp-config)" 
```

---

# Part 3 — Advanced Log Analysis

---

## Exercise 10: Traffic Analysis by Hour

### Extract Hour from Timestamp
```bash
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | sort -nr
```

### Alternative Method
```bash
grep -oP '\[.*?\]' access.log | cut -d: -f2 | sort | uniq -c | sort -nr
```

### Create Hourly Traffic Report
```bash
for hour in {00..23}; do
    count=$(grep -c ":${hour}:" access.log)
    echo "Hour $hour: $count requests"
done
```

### Identify Traffic Spikes
```bash
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | awk '$1 > 1000 {print "Spike at hour " $2 ": " $1 " requests"}'
```

### Purpose
- Detect unusual traffic spikes
- Identify off-hour activity (3 AM scans)
- Plan capacity management
- Investigate DDoS patterns

---

## Exercise 11: HTTP Method Distribution

### Standard Methods

| Method | Purpose | Security Concern |
|---|---|---|
| GET | Retrieve data | Normal, can be logged |
| POST | Submit data | Login forms, file uploads |
| PUT | Upload file | High - unauthorized uploads |
| DELETE | Remove file | High - data destruction |
| HEAD | Headers only | Reconnaissance |
| OPTIONS | List methods | Information disclosure |
| TRACE | Debug | Cross-site Tracing attacks |
| CONNECT | Proxy tunnel | Tunneling malware |

### Command
```bash
awk '{print $6}' access.log | tr -d '"' | sort | uniq -c | sort -nr
```

### Detect Unusual Methods
```bash
awk '{print $6}' access.log | tr -d '"' | sort -u | grep -vE "(GET|POST|HEAD|OPTIONS)"
```

### Analyze PUT Requests
```bash
grep '"PUT' access.log | awk '{print $1, $7, $9}'
```

### Security Considerations

Unexpected methods like:
- **PUT** - May indicate file upload attacks
- **DELETE** - Could be data destruction attempts  
- **TRACE** - Enables XST (Cross-Site Tracing) attacks
- **CONNECT** - Often used by proxy tools and malware

---

## Exercise 12: Analyze POST Requests

### POST Requests Analysis

POST requests are particularly important because they:
- Submit login credentials
- Upload files
- Send form data
- Are used in brute force attacks

### Count POST Requests
```bash
grep -c '"POST' access.log
```

### Extract POST Request IPs
```bash
awk '$6 == "\"POST"' access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
```

### Detailed POST Activity
```bash
awk '$6 == "\"POST"' access.log | awk '{print $1, $7, $9}' | head -20
```

### Find Login POST Attempts
```bash
grep '"POST' access.log | grep -E "(login|auth|signin)" | awk '{print $1, $7, $9}'
```

### Analyze POST Request Sizes
```bash
grep '"POST' access.log | awk '{print $1, $10}' | sort -k2 -nr | head -10
```

---

## Exercise 13: Measure Bandwidth Usage

### Total Data Transferred (All Requests)
```bash
awk '{sum += $10} END {printf "Total: %.2f MB\n", sum/1024/1024}' access.log
```

### Breakdown by Status Code
```bash
awk '{bytes[$9] += $10} END {for (code in bytes) printf "Status %s: %.2f MB\n", code, bytes[code]/1024/1024}' access.log
```

### Highest Bandwidth Consumers by IP
```bash
awk '{traffic[$1] += $10} END {for (ip in traffic) printf "%s: %.2f MB\n", ip, traffic[ip]/1024/1024}' access.log | sort -k2 -nr | head -10
```

### Largest Individual Requests
```bash
awk '{print $10, $1, $7}' access.log | sort -nr | head -10
```

### Download vs Upload Ratio
```bash
# Download (response size)
download=$(awk '{sum += $10} END {print sum}' access.log)
# Upload (request size - not in standard logs)
echo "Download total: $(echo "scale=2; $download/1024/1024" | bc) MB"
```

---

# Part 4 — Data Transformation

---

## Exercise 14: Export Unique IP Addresses

### Command
```bash
awk '{print $1}' access.log | sort -u > outputs/unique_ips.txt
```

### Verify Output
```bash
wc -l outputs/unique_ips.txt
head -10 outputs/unique_ips.txt
```

### Categorize IPs by Request Count
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | awk '$1 > 100 {print $2, $1}' > outputs/high_volume_ips.txt
```

---

## Exercise 15: Mask IP Addresses (Privacy)

### Basic IP Masking
```bash
sed 's/\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}/XXX.XXX.XXX.XXX/' access.log | head -5
```

### Preserve First Octet
```bash
sed 's/\([0-9]\{1,3\}\)\.\([0-9]\{1,3\}\)\.\([0-9]\{1,3\}\)\.\([0-9]\{1,3\}\)/\1.XXX.XXX.XXX/' access.log | head -5
```

### Preserve First Two Octets (Network ID)
```bash
sed 's/\([0-9]\{1,3\}\)\.\([0-9]\{1,3\}\)\.\([0-9]\{1,3\}\)\.\([0-9]\{1,3\}\)/\1.\2.XXX.XXX/' access.log | head -5
```

---

## Exercise 16: Extract Requested URLs

### Extract All URLs
```bash
awk '{print $7}' access.log | sort -u > outputs/urls.txt
```

### Extract Only Dynamic Pages
```bash
awk '{print $7}' access.log | grep -E "\.(php|jsp|asp|aspx|cgi|pl)$" > outputs/dynamic_urls.txt
```

### Extract Static Files
```bash
awk '{print $7}' access.log | grep -E "\.(css|js|png|jpg|jpeg|gif|ico|svg|woff|ttf)$" > outputs/static_urls.txt
```

### URL Length Analysis (Possible Buffer Overflow)
```bash
awk '{print length($7), $1, $7}' access.log | sort -nr | head -10
```

---

## Exercise 17: User-Agent Analysis

### Top User Agents
```bash
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -nr | head -10
```

### Identify Suspicious User-Agents
```bash
awk -F'"' '{print $6}' access.log | grep -iE "(python|curl|wget|nikto|nmap|sqlmap|burp|zap)" | sort | uniq -c | sort -nr
```

### Missing User-Agent (Possible Automated Tools)
```bash
grep '"-"' access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

---

# Part 5 — Automation Script

## Create Analysis Script

```bash
nano scripts/analyze_logs.sh
```

```bash
#!/bin/bash

# Apache Log Analysis Automation Script
# Usage: ./analyze_logs.sh access.log

LOG_FILE=$1
OUTPUT_DIR="reports/analysis_$(date +%Y%m%d_%H%M%S)"

if [ -z "$LOG_FILE" ]; then
    echo "Usage: $0 <access.log>"
    exit 1
fi

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: File $LOG_FILE not found!"
    exit 1
fi

mkdir -p $OUTPUT_DIR

echo "=== Apache Log Analysis Report ==="
echo "Analyzing: $LOG_FILE"
echo "Output directory: $OUTPUT_DIR"
echo ""

# 1. Basic Statistics
echo "[1] Generating basic statistics..."
echo "Total Requests: $(wc -l < $LOG_FILE)" > $OUTPUT_DIR/basic_stats.txt
echo "Unique IPs: $(awk '{print $1}' $LOG_FILE | sort -u | wc -l)" >> $OUTPUT_DIR/basic_stats.txt
echo "" >> $OUTPUT_DIR/basic_stats.txt

# 2. Status Code Distribution
echo "[2] Analyzing status codes..."
awk '{print $9}' $LOG_FILE | sort | uniq -c | sort -nr > $OUTPUT_DIR/status_codes.txt

# 3. Top IPs
echo "[3] Finding top IPs..."
awk '{print $1}' $LOG_FILE | sort | uniq -c | sort -nr | head -20 > $OUTPUT_DIR/top_ips.txt

# 4. Security Threats
echo "[4] Hunting threats..."

# SQL Injection
grep -ciE "(union|select|drop|insert|update|delete)" $LOG_FILE > $OUTPUT_DIR/sqli_count.txt
grep -iE "(union|select|drop)" $LOG_FILE | awk '{print $1}' | sort -u > $OUTPUT_DIR/sqli_ips.txt

# XSS
grep -ciE "(<script|javascript:|alert\(|onerror=)" $LOG_FILE > $OUTPUT_DIR/xss_count.txt
grep -iE "(<script|javascript:|alert\()" $LOG_FILE | awk '{print $1}' | sort -u > $OUTPUT_DIR/xss_ips.txt

# Directory Traversal
grep -cE "(\.\./|\.\.%2[fF])" $LOG_FILE > $OUTPUT_DIR/traversal_count.txt
grep -E "(\.\./|\.\.%2[fF])" $LOG_FILE | awk '{print $1}' | sort -u > $OUTPUT_DIR/traversal_ips.txt

# Scanners
grep -ciE "(nikto|nmap|sqlmap|dirb|gobuster)" $LOG_FILE > $OUTPUT_DIR/scanner_count.txt

# 5. Bandwidth
echo "[5] Calculating bandwidth..."
awk '{sum += $10} END {printf "%.2f MB\n", sum/1024/1024}' $LOG_FILE > $OUTPUT_DIR/bandwidth.txt

# 6. Create Summary
echo "[6] Creating summary..."
cat > $OUTPUT_DIR/summary.txt << EOF
=== ANALYSIS SUMMARY ===
Total Requests: $(wc -l < $LOG_FILE)
Unique IPs: $(awk '{print $1}' $LOG_FILE | sort -u | wc -l)
Total Bandwidth: $(awk '{sum += $10} END {printf "%.2f MB", sum/1024/1024}' $LOG_FILE)

Security Threats Detected:
- SQL Injection Attempts: $(cat $OUTPUT_DIR/sqli_count.txt)
- XSS Attempts: $(cat $OUTPUT_DIR/xss_count.txt)
- Directory Traversal: $(cat $OUTPUT_DIR/traversal_count.txt)
- Scanners Detected: $(cat $OUTPUT_DIR/scanner_count.txt)

Check individual files in $OUTPUT_DIR for details.
EOF

echo ""
echo "=== Analysis Complete ==="
echo "Reports saved to: $OUTPUT_DIR"
cat $OUTPUT_DIR/summary.txt
```

### Make Script Executable
```bash
chmod +x scripts/analyze_logs.sh
```

### Run the Script
```bash
./scripts/analyze_logs.sh access.log
```

---

# Investigation Report

## Report Template

Create file: `reports/Apache-Investigation-Report.md`

```md
# Apache Web Server Investigation Report

## Analyst Information

| Field | Value |
|-------|-------|
| Analyst Name | [Your Name] |
| Analysis Date | $(date) |
| Log Source | access.log |
| Investigation Scope | Full log analysis |
| Severity Level | [Low/Medium/High/Critical] |

---

## Executive Summary

[Brief overview of findings - 2-3 sentences]

---

## Traffic Statistics

| Metric | Value |
|--------|-------|
| Total Requests | `[paste wc -l output]` |
| Unique Visitors | `[paste unique IP count]` |
| Total Bandwidth | `[paste bandwidth total]` |
| Analysis Period | [First log date - Last log date] |

---

## Top 10 Active IP Addresses

| Rank | IP Address | Request Count | Threat Level |
|------|------------|---------------|--------------|
| 1 | [IP] | [count] | [Low/Med/High] |
| 2 | [IP] | [count] | [Low/Med/High] |
| ... | ... | ... | ... |

---

## HTTP Status Code Breakdown

| Status Code | Count | Percentage | Interpretation |
|-------------|-------|------------|----------------|
| 2xx | [count] | [%] | Success |
| 3xx | [count] | [%] | Redirect |
| 4xx | [count] | [%] | Client Error |
| 5xx | [count] | [%] | Server Error |

---

## HTTP Method Distribution

| Method | Count | Percentage | Risk |
|--------|-------|------------|------|
| GET | [count] | [%] | Low |
| POST | [count] | [%] | Medium |
| [Other] | [count] | [%] | [Risk] |

---

## Security Findings

### 1. Vulnerability Scanning

| Scanner | Detected | Source IPs |
|---------|----------|------------|
| Nikto | [Yes/No] | [IPs] |
| Nmap | [Yes/No] | [IPs] |
| SQLmap | [Yes/No] | [IPs] |
| Other | [Yes/No] | [IPs] |

### 2. SQL Injection Attempts

- **Total Attempts:** [count]
- **Unique Attacking IPs:** [count]
- **Most Targeted URLs:** [list]
- **Sample Attack:** `[paste example]`

### 3. Cross-Site Scripting (XSS) Attempts

- **Total Attempts:** [count]
- **Unique Attacking IPs:** [count]
- **Attack Vectors:** [list]
- **Sample Attack:** `[paste example]`

### 4. Directory Traversal Attempts

- **Total Attempts:** [count]
- **Unique Attacking IPs:** [count]
- **Targeted Files:** [list]
- **Sample Attack:** `[paste example]`

### 5. Suspicious User-Agents

| User-Agent | Count | Risk |
|------------|-------|------|
| [UA] | [count] | [Risk] |
| [UA] | [count] | [Risk] |

---

## Suspicious IP Addresses

| IP Address | Country* | Suspicious Activity | Priority |
|------------|----------|--------------------|----------|
| [IP] | [Country] | SQL Injection | High |
| [IP] | [Country] | Scanner | Medium |
| [IP] | [Country] | High Volume | Low |

*Optional - use geoiplookup

---

## Peak Traffic Hours

| Hour (UTC) | Request Count | Normal? |
|------------|---------------|---------|
| [00-23] | [count] | [Yes/No] |
| [00-23] | [count] | [Yes/No] |

---

## Most Requested Missing Pages (404)

| URL | Count | Potential Threat |
|-----|-------|------------------|
| [URL] | [count] | [Threat type] |
| [URL] | [count] | [Threat type] |

---

## Recommendations

### Immediate Actions
- [ ] Block malicious IPs: `[list IPs]`
- [ ] Review firewall rules
- [ ] Check for compromised accounts

### Short-term Improvements
- [ ] Implement WAF rules for SQLi/XSS
- [ ] Rate limiting for suspicious IPs
- [ ] Harden sensitive directories

### Long-term Recommendations
- [ ] Deploy IDS/IPS
- [ ] Implement centralized logging (SIEM)
- [ ] Regular security audits
- [ ] Security awareness training

---

## Conclusion

[Detailed summary of findings, impact assessment, and final recommendations]

---

## Appendix

### Commands Used
```bash
[paste key commands]
```

### Tools Version
- OS: `[OS info]`
- grep: `[version]`
- awk: `[version]`

### Raw Data Files
- `unique_ips.txt`
- `urls.txt`
- `suspicious_ips.txt`

---
```

---

# Deliverables

Submit the following files in your report:

| File | Description | Required |
|------|-------------|----------|
| `Apache-Investigation-Report.md` | Complete investigation report | Yes |
| `unique_ips.txt` | List of all unique IPs | Yes |
| `urls.txt` | All requested URLs | Yes |
| `suspicious_ips.txt` | IPs with malicious activity | Yes |
| `analysis_script.sh` | Automation script | Bonus |
| `threat_intel.txt` | Extracted threat indicators | Bonus |

---

# Bonus Challenges

## Challenge 1: Create Alerting Script

Create a script that alerts when:
- More than 10 SQLi attempts from same IP in 5 minutes
- Single IP requests > 500 times in 1 minute
- 5+ scanner signatures detected
- POST requests to /admin exceed threshold

```bash
# Example alerting script structure
#!/bin/bash
THRESHOLD=100
IP_COUNTS=$(awk '{print $1}' access.log | sort | uniq -c | sort -nr)

echo "$IP_COUNTS" | while read count ip; do
    if [ $count -gt $THRESHOLD ]; then
        echo "ALERT: $ip made $count requests"
    fi
done
```

## Challenge 2: Generate Timeline

Create chronological timeline of events:

```bash
awk '{print $4, $1,
